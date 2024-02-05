# macOS .Net Applications Injection

<details>

<summary><strong>htARTE（HackTricks AWS Red Team Expert）</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>でゼロからヒーローまでAWSハッキングを学びましょう</strong></a><strong>！</strong></summary>

HackTricks をサポートする他の方法:

- **HackTricks で企業を宣伝したい**、または **HackTricks をPDFでダウンロードしたい**場合は、[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください！
- [**公式PEASS＆HackTricksスウォッグ**](https://peass.creator-spring.com)を入手する
- [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見し、独占的な[**NFTs**](https://opensea.io/collection/the-peass-family)コレクションを見つける
- **💬 [Discordグループ](https://discord.gg/hRep4RUj7f)**に参加するか、[telegramグループ](https://t.me/peass)に参加するか、**Twitter** 🐦で私をフォローする [**@carlospolopm**](https://twitter.com/carlospolopm)**。**
- **ハッキングトリックを共有するために、[HackTricks](https://github.com/carlospolop/hacktricks)と[HackTricks Cloud](https://github.com/carlospolop/hacktricks-cloud)のGitHubリポジトリにPRを提出してください。**

</details>

**これは[https://blog.xpnsec.com/macos-injection-via-third-party-frameworks/](https://blog.xpnsec.com/macos-injection-via-third-party-frameworks/)の投稿の要約です。詳細についてはそちらをご確認ください！**

## .NET Core デバッグ <a href="#net-core-debugging" id="net-core-debugging"></a>

### **デバッグセッションの確立** <a href="#net-core-debugging" id="net-core-debugging"></a>

.NETにおけるデバッガーとデバッガ対象間の通信は、[**dbgtransportsession.cpp**](https://github.com/dotnet/runtime/blob/0633ecfb79a3b2f1e4c098d1dd0166bc1ae41739/src/coreclr/debug/shared/dbgtransportsession.cpp)によって管理されます。このコンポーネントは、[dbgtransportsession.cpp#L127](https://github.com/dotnet/runtime/blob/0633ecfb79a3b2f1e4c098d1dd0166bc1ae41739/src/coreclr/debug/shared/dbgtransportsession.cpp#L127)で確認できるように、.NETプロセスごとに2つの名前付きパイプを設定します。これらは[**`-in`**]と[**`-out`**]で接尾辞が付けられています。

ユーザーの**`$TMPDIR`**を訪れることで、.Netアプリケーションのデバッグ用FIFOを見つけることができます。

[**DbgTransportSession::TransportWorker**](https://github.com/dotnet/runtime/blob/0633ecfb79a3b2f1e4c098d1dd0166bc1ae41739/src/coreclr/debug/shared/dbgtransportsession.cpp#L1259)は、デバッガーからの通信を管理する責任があります。新しいデバッグセッションを開始するには、デバッガーは、.NETソースコードで詳細に説明されている`MessageHeader`構造体で始まる`out`パイプ経由でメッセージを送信する必要があります。
```c
struct MessageHeader {
MessageType   m_eType;        // Message type
DWORD         m_cbDataBlock;  // Size of following data block (can be zero)
DWORD         m_dwId;         // Message ID from sender
DWORD         m_dwReplyId;    // Reply-to Message ID
DWORD         m_dwLastSeenId; // Last seen Message ID by sender
DWORD         m_dwReserved;   // Reserved for future (initialize to zero)
union {
struct {
DWORD         m_dwMajorVersion;   // Requested/accepted protocol version
DWORD         m_dwMinorVersion;
} VersionInfo;
...
} TypeSpecificData;
BYTE          m_sMustBeZero[8];
}
```
新しいセッションをリクエストするには、この構造体を次のように設定してポピュレートします。メッセージタイプを `MT_SessionRequest` に、プロトコルバージョンを現在のバージョンに設定します。
```c
static const DWORD kCurrentMajorVersion = 2;
static const DWORD kCurrentMinorVersion = 0;

// Configure the message type and version
sSendHeader.m_eType = MT_SessionRequest;
sSendHeader.TypeSpecificData.VersionInfo.m_dwMajorVersion = kCurrentMajorVersion;
sSendHeader.TypeSpecificData.VersionInfo.m_dwMinorVersion = kCurrentMinorVersion;
sSendHeader.m_cbDataBlock = sizeof(SessionRequestData);
```
このヘッダーは、`write` シスコールを使用してターゲットに送信され、その後にセッションの GUID を含む `sessionRequestData` 構造体が続きます：
```c
write(wr, &sSendHeader, sizeof(MessageHeader));
memset(&sDataBlock.m_sSessionID, 9, sizeof(SessionRequestData));
write(wr, &sDataBlock, sizeof(SessionRequestData));
```
`out` パイプ上の読み取り操作は、デバッグセッションの確立の成功または失敗を確認します。
```c
read(rd, &sReceiveHeader, sizeof(MessageHeader));
```
## メモリの読み取り
デバッグセッションが確立されると、[`MT_ReadMemory`](https://github.com/dotnet/runtime/blob/f3a45a91441cf938765bafc795cbf4885cad8800/src/coreclr/src/debug/shared/dbgtransportsession.cpp#L1896) メッセージタイプを使用してメモリを読み取ることができます。readMemory 関数は詳細に記載されており、読み取りリクエストを送信し、レスポンスを取得するために必要な手順を実行します。
```c
bool readMemory(void *addr, int len, unsigned char **output) {
// Allocation and initialization
...
// Write header and read response
...
// Read the memory from the debuggee
...
return true;
}
```
完全な概念の証明（POC）は[こちら](https://gist.github.com/xpn/95eefc14918998853f6e0ab48d9f7b0b)で入手できます。

## メモリの書き込み

同様に、`writeMemory` 関数を使用してメモリに書き込むことができます。このプロセスでは、メッセージタイプを `MT_WriteMemory` に設定し、データのアドレスと長さを指定してからデータを送信します。
```c
bool writeMemory(void *addr, int len, unsigned char *input) {
// Increment IDs, set message type, and specify memory location
...
// Write header and data, then read the response
...
// Confirm memory write was successful
...
return true;
}
```
関連するPOCは[こちら](https://gist.github.com/xpn/7c3040a7398808747e158a25745380a5)で入手できます。

## .NET Coreコード実行 <a href="#net-core-code-execution" id="net-core-code-execution"></a>

コードを実行するには、vmmap -pagesを使用してrwx権限を持つメモリ領域を特定する必要があります。
```bash
vmmap -pages [pid]
vmmap -pages 35829 | grep "rwx/rwx"
```
関数ポインタを上書きする場所を見つけることは必要であり、.NET Coreでは**Dynamic Function Table (DFT)** をターゲットにすることができます。このテーブルは、[`jithelpers.h`](https://github.com/dotnet/runtime/blob/6072e4d3a7a2a1493f514cdf4be75a3d56580e84/src/coreclr/src/inc/jithelpers.h) で詳細に説明されており、ランタイムがJITコンパイルヘルパー関数に使用します。

x64システムでは、シグネチャハンティングを使用して、`libcorclr.dll`内のシンボル `_hlpDynamicFuncTable` への参照を見つけることができます。

`MT_GetDCB` デバッガー関数は、`libcorclr.dll` のプロセスメモリ内の場所を示すヘルパー関数 `m_helperRemoteStartAddr` のアドレスを含む有用な情報を提供します。このアドレスは、DFTを検索してシェルコードのアドレスで関数ポインタを上書きするために使用されます。

PowerShellへのインジェクションのための完全なPOCコードは、[こちら](https://gist.github.com/xpn/b427998c8b3924ab1d63c89d273734b6) でアクセスできます。

## 参考文献

* [https://blog.xpnsec.com/macos-injection-via-third-party-frameworks/](https://blog.xpnsec.com/macos-injection-via-third-party-frameworks/)
