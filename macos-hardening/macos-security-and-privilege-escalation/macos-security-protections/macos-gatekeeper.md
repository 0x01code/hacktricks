# macOS Gatekeeper / Quarantine / XProtect

<details>

<summary><strong>htARTE（HackTricks AWS Red Team Expert）</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>を通じてゼロからヒーローまでAWSハッキングを学ぶ</strong></a><strong>！</strong></summary>

* **サイバーセキュリティ企業**で働いていますか？ **HackTricksで会社を宣伝**してみたいですか？または**PEASSの最新バージョンにアクセスしたり、HackTricksをPDFでダウンロード**したいですか？[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見し、独占的な[NFTs](https://opensea.io/collection/the-peass-family)コレクションをご覧ください
* [**公式PEASS＆HackTricks swag**](https://peass.creator-spring.com)を手に入れましょう
* **[💬](https://emojipedia.org/speech-balloon/) Discordグループ**に**参加**するか、[**telegramグループ**](https://t.me/peass)に参加するか、**Twitter**で私をフォローする🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**。**
* **ハッキングトリックを共有するために** [**hacktricksリポジトリ**](https://github.com/carlospolop/hacktricks) **と** [**hacktricks-cloudリポジトリ**](https://github.com/carlospolop/hacktricks-cloud) **にPRを提出してください**

</details>

<figure><img src="https://pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}

## Gatekeeper

**Gatekeeper**は、Macオペレーティングシステム向けに開発されたセキュリティ機能で、ユーザーがシステム上で**信頼されたソフトウェアのみを実行**することを保証するために設計されています。ユーザーがApp Store以外のソースから（アプリ、プラグイン、またはインストーラーパッケージなど）ダウンロードして開こうとするソフトウェアを**検証**することで機能します。

Gatekeeperの主要なメカニズムは、その**検証**プロセスにあります。ダウンロードしたソフトウェアが**認識された開発者によって署名されているかどうか**をチェックし、ソフトウェアの信頼性を確認します。さらに、Appleによって**ノータライズ**されているかどうかを確認し、既知の悪意のあるコンテンツが含まれていないこと、およびノータライズ後に改ざんされていないことを確認します。

さらに、Gatekeeperは、ユーザーがダウンロードしたソフトウェアの初めての実行を承認するよう**ユーザーに促す**ことで、ユーザーの誤って害のある実行可能コードを実行するのを防ぐのに役立ちます。

### アプリケーション署名

アプリケーション署名、またはコード署名としても知られるものは、Appleのセキュリティインフラストラクチャの重要な部分です。これらは、ソフトウェアの著者（開発者）の**身元を確認**し、コードが最後に署名されてから改ざんされていないことを確認するために使用されます。

動作方法は次のとおりです：

1. **アプリケーションの署名:** 開発者がアプリケーションを配布する準備が整ったとき、**開発者はプライベートキーを使用してアプリケーションに署名**します。このプライベートキーは、AppleがApple Developer Programに登録したときに開発者に発行する**証明書**と関連付けられています。署名プロセスには、アプリのすべての部分の暗号ハッシュを作成し、このハッシュを開発者のプライベートキーで暗号化することが含まれます。
2. **アプリケーションの配布:** 署名されたアプリケーションは、開発者の証明書とともにユーザーに配布されます。この証明書には、対応する公開キーが含まれています。
3. **アプリケーションの検証:** ユーザーがアプリケーションをダウンロードして実行しようとすると、Macオペレーティングシステムは、開発者の証明書から公開キーを使用してハッシュを復号化します。その後、アプリケーションの現在の状態に基づいてハッシュを再計算し、これを復号化されたハッシュと比較します。一致する場合、それは開発者が署名したときから**アプリケーションが変更されていない**ことを意味し、システムはアプリケーションの実行を許可します。

アプリケーション署名は、AppleのGatekeeperテクノロジーの重要な部分です。ユーザーがインターネットから**ダウンロードしたアプリケーションを開こうとする**と、Gatekeeperはアプリケーションの署名を検証します。Appleが既知の開発者に発行した証明書で署名されており、コードが改ざんされていない場合、Gatekeeperはアプリケーションの実行を許可します。そうでない場合、アプリケーションをブロックしてユーザーに警告します。

macOS Catalinaからは、**GatekeeperはアプリケーションがAppleによってノータライズされているかどうかも確認**し、追加のセキュリティレイヤーを追加します。ノータライズプロセスは、アプリケーションを既知のセキュリティ問題や悪意のあるコードに対してチェックし、これらのチェックに合格した場合、AppleはGatekeeperが検証できるアプリケーションにチケットを追加します。

#### 署名の確認

いくつかの**マルウェアサンプル**をチェックするときは、常にバイナリの署名を**確認**する必要があります。署名した**開発者**がすでに**マルウェア**と**関連付けられている**可能性があるためです。
```bash
# Get signer
codesign -vv -d /bin/ls 2>&1 | grep -E "Authority|TeamIdentifier"

# Check if the app’s contents have been modified
codesign --verify --verbose /Applications/Safari.app

# Get entitlements from the binary
codesign -d --entitlements :- /System/Applications/Automator.app # Check the TCC perms

# Check if the signature is valid
spctl --assess --verbose /Applications/Safari.app

# Sign a binary
codesign -s <cert-name-keychain> toolsdemo
```
### Notarization

Appleのノータリゼーションプロセスは、ユーザーを潜在的に有害なソフトウェアから保護する追加の保護手段として機能します。これには、開発者が自分のアプリケーションをAppleのノータリサービスによる審査に提出することが含まれます。このサービスは、App Reviewとは異なる自動化システムであり、提出されたソフトウェアを悪意のあるコンテンツやコードサイニングに関する潜在的な問題の有無を調査します。

ソフトウェアがこの検査をパスし、懸念事項がない場合、ノータリサービスはノータリゼーションチケットを生成します。その後、開発者はこのチケットを自分のソフトウェアに添付する必要があります。このプロセスは「ステープリング」として知られています。さらに、ノータリゼーションチケットはオンラインでも公開され、Gatekeeper（Appleのセキュリティテクノロジー）がアクセスできます。

ユーザーがソフトウェアを初めてインストールまたは実行する際、実行可能ファイルにステープリングされているかオンラインで見つかるかにかかわらず、ノータリゼーションチケットの存在により、GatekeeperにそのソフトウェアがAppleによってノータリゼーションされたことが通知されます。その結果、Gatekeeperは初回起動ダイアログに記述的なメッセージを表示し、Appleによる悪意のあるコンテンツのチェックを受けたことを示します。このプロセスにより、ユーザーは自分のシステムにインストールまたは実行するソフトウェアのセキュリティに対する信頼を高めることができます。

### Enumerating GateKeeper

GateKeeperは、信頼されていないアプリケーションの実行を防止するいくつかのセキュリティコンポーネントであり、またその1つでもあります。

GateKeeperのステータスを確認することができます:
```bash
# Check the status
spctl --status
```
{% hint style="danger" %}
GateKeeperの署名チェックは、**Quarantine属性を持つファイル**にのみ実行されることに注意してください。
{% endhint %}

GateKeeperは、**設定と署名**に従ってバイナリが実行できるかどうかをチェックします：

<figure><img src="../../../.gitbook/assets/image (1150).png" alt=""><figcaption></figcaption></figure>

この構成を保持するデータベースは**`/var/db/SystemPolicy`**にあります。これをルートとして次のコマンドで確認できます：
```bash
# Open database
sqlite3 /var/db/SystemPolicy

# Get allowed rules
SELECT requirement,allow,disabled,label from authority where label != 'GKE' and disabled=0;
requirement|allow|disabled|label
anchor apple generic and certificate 1[subject.CN] = "Apple Software Update Certification Authority"|1|0|Apple Installer
anchor apple|1|0|Apple System
anchor apple generic and certificate leaf[field.1.2.840.113635.100.6.1.9] exists|1|0|Mac App Store
anchor apple generic and certificate 1[field.1.2.840.113635.100.6.2.6] exists and (certificate leaf[field.1.2.840.113635.100.6.1.14] or certificate leaf[field.1.2.840.113635.100.6.1.13]) and notarized|1|0|Notarized Developer ID
[...]
```
注意してください、最初のルールは "**App Store**" で終わり、2番目のルールは "**Developer ID**" で終わり、前の画像では**App Storeおよび識別された開発者からのアプリの実行が有効になっていました**。\
その設定をApp Storeに変更すると、**"Notarized Developer ID" ルールが消えます**。

**GKEタイプ**のルールも何千もあります:
```bash
SELECT requirement,allow,disabled,label from authority where label = 'GKE' limit 5;
cdhash H"b40281d347dc574ae0850682f0fd1173aa2d0a39"|1|0|GKE
cdhash H"5fd63f5342ac0c7c0774ebcbecaf8787367c480f"|1|0|GKE
cdhash H"4317047eefac8125ce4d44cab0eb7b1dff29d19a"|1|0|GKE
cdhash H"0a71962e7a32f0c2b41ddb1fb8403f3420e1d861"|1|0|GKE
cdhash H"8d0d90ff23c3071211646c4c9c607cdb601cb18f"|1|0|GKE
```
これらは、**`/var/db/SystemPolicyConfiguration/gke.bundle/Contents/Resources/gke.auth`、`/var/db/gke.bundle/Contents/Resources/gk.db`**、および**`/var/db/gkopaque.bundle/Contents/Resources/gkopaque.db`**から取得されるハッシュです。

または、前述の情報をリストすることもできます:
```bash
sudo spctl --list
```
オプション**`--master-disable`**と**`--global-disable`**は、**`spctl`**の署名チェックを完全に**無効**にします。
```bash
# Disable GateKeeper
spctl --global-disable
spctl --master-disable

# Enable it
spctl --global-enable
spctl --master-enable
```
完全に有効にすると、新しいオプションが表示されます：

<figure><img src="../../../.gitbook/assets/image (1151).png" alt=""><figcaption></figcaption></figure>

GateKeeper によってアプリが許可されるかどうかを**チェックする**ことが可能です。
```bash
spctl --assess -v /Applications/App.app
```
GateKeeperに新しいルールを追加して、特定のアプリケーションの実行を許可することが可能です:
```bash
# Check if allowed - nop
spctl --assess -v /Applications/App.app
/Applications/App.app: rejected
source=no usable signature

# Add a label and allow this label in GateKeeper
sudo spctl --add --label "whitelist" /Applications/App.app
sudo spctl --enable --label "whitelist"

# Check again - yep
spctl --assess -v /Applications/App.app
/Applications/App.app: accepted
```
### ファイルの隔離

特定のmacOS **アプリケーション**（Webブラウザやメールクライアントなど）が、アプリケーションやファイルを**ダウンロード**する際に、**拡張ファイル属性**である一般的に知られている "**隔離フラグ**" をダウンロードされたファイルに**添付**します。この属性は、ファイルを信頼できないソース（インターネット）から取得したものとしてマークし、潜在的なリスクを持つ可能性があることを示すセキュリティ対策として機能します。ただし、すべてのアプリケーションがこの属性を添付するわけではなく、たとえば一般的なBitTorrentクライアントソフトウェアは通常、このプロセスをバイパスします。

**隔離フラグが存在する場合、ユーザーがファイルを実行しようとすると、macOSのGatekeeperセキュリティ機能が作動します**。

**隔離フラグが存在しない場合**（一部のBitTorrentクライアントを介してダウンロードされたファイルなど）、Gatekeeperの**チェックが実行されない可能性**があります。したがって、ユーザーは安全でないまたは不明なソースからダウンロードしたファイルを開く際には注意を払う必要があります。

{% hint style="info" %}
**コード署名の有効性**を**チェック**することは、コードとそのバンドルされたリソースの暗号ハッシュを生成するなど、**リソースを多く消費する**プロセスです。さらに、証明書の有効性をチェックするには、発行後に取り消されたかどうかをAppleのサーバーに**オンラインで確認**する必要があります。これらの理由から、アプリケーションを起動するたびに完全なコード署名と公証チェックを実行するのは**現実的ではありません**。

したがって、これらのチェックは**隔離属性を持つアプリを実行するときにのみ実行**されます。
{% endhint %}

{% hint style="warning" %}
この属性は、ファイルを作成/ダウンロードする**アプリケーションによって設定**する必要があります。

ただし、サンドボックス化されたファイルは、作成されるすべてのファイルにこの属性が設定されます。サンドボックス化されていないアプリケーションは、自分で設定するか、**Info.plist**で[**LSFileQuarantineEnabled**](https://developer.apple.com/documentation/bundleresources/information\_property\_list/lsfilequarantineenabled?language=objc)キーを指定することで、システムに`com.apple.quarantine`拡張属性をファイルに設定させることができます。
{% endhint %}

さらに、**`qtn_proc_apply_to_self`**を呼び出すプロセスによって作成されたすべてのファイルは隔離されます。また、API **`qtn_file_apply_to_path`**は指定されたファイルパスに隔離属性を追加します。

状態を**チェックして有効/無効にする**（ルート権限が必要）ことが可能です。
```bash
spctl --status
assessments enabled

spctl --enable
spctl --disable
#You can also allow nee identifies to execute code using the binary "spctl"
```
あるファイルがクォータンティン拡張属性を持っているかどうかを次のようにして見つけることもできます:
```bash
xattr file.png
com.apple.macl
com.apple.quarantine
```
**拡張属性**の**値**をチェックし、quarantine属性を書き込んだアプリを特定します。
```bash
xattr -l portada.png
com.apple.macl:
00000000  03 00 53 DA 55 1B AE 4C 4E 88 9D CA B7 5C 50 F3  |..S.U..LN.....P.|
00000010  16 94 03 00 27 63 64 97 98 FB 4F 02 84 F3 D0 DB  |....'cd...O.....|
00000020  89 53 C3 FC 03 00 27 63 64 97 98 FB 4F 02 84 F3  |.S....'cd...O...|
00000030  D0 DB 89 53 C3 FC 00 00 00 00 00 00 00 00 00 00  |...S............|
00000040  00 00 00 00 00 00 00 00                          |........|
00000048
com.apple.quarantine: 00C1;607842eb;Brave;F643CD5F-6071-46AB-83AB-390BA944DEC5
# 00c1 -- It has been allowed to eexcute this file (QTN_FLAG_USER_APPROVED = 0x0040)
# 607842eb -- Timestamp
# Brave -- App
# F643CD5F-6071-46AB-83AB-390BA944DEC5 -- UID assigned to the file downloaded
```
実際にプロセスは「作成するファイルに隔離フラグを設定できる可能性があります」（作成したファイルにUSER\_APPROVEDフラグを適用しようとしましたが、適用されませんでした）:

<details>

<summary>ソースコードに隔離フラグを適用</summary>
```c
#include <stdio.h>
#include <stdlib.h>

enum qtn_flags {
QTN_FLAG_DOWNLOAD = 0x0001,
QTN_FLAG_SANDBOX = 0x0002,
QTN_FLAG_HARD = 0x0004,
QTN_FLAG_USER_APPROVED = 0x0040,
};

#define qtn_proc_alloc _qtn_proc_alloc
#define qtn_proc_apply_to_self _qtn_proc_apply_to_self
#define qtn_proc_free _qtn_proc_free
#define qtn_proc_init _qtn_proc_init
#define qtn_proc_init_with_self _qtn_proc_init_with_self
#define qtn_proc_set_flags _qtn_proc_set_flags
#define qtn_file_alloc _qtn_file_alloc
#define qtn_file_init_with_path _qtn_file_init_with_path
#define qtn_file_free _qtn_file_free
#define qtn_file_apply_to_path _qtn_file_apply_to_path
#define qtn_file_set_flags _qtn_file_set_flags
#define qtn_file_get_flags _qtn_file_get_flags
#define qtn_proc_set_identifier _qtn_proc_set_identifier

typedef struct _qtn_proc *qtn_proc_t;
typedef struct _qtn_file *qtn_file_t;

int qtn_proc_apply_to_self(qtn_proc_t);
void qtn_proc_init(qtn_proc_t);
int qtn_proc_init_with_self(qtn_proc_t);
int qtn_proc_set_flags(qtn_proc_t, uint32_t flags);
qtn_proc_t qtn_proc_alloc();
void qtn_proc_free(qtn_proc_t);
qtn_file_t qtn_file_alloc(void);
void qtn_file_free(qtn_file_t qf);
int qtn_file_set_flags(qtn_file_t qf, uint32_t flags);
uint32_t qtn_file_get_flags(qtn_file_t qf);
int qtn_file_apply_to_path(qtn_file_t qf, const char *path);
int qtn_file_init_with_path(qtn_file_t qf, const char *path);
int qtn_proc_set_identifier(qtn_proc_t qp, const char* bundleid);

int main() {

qtn_proc_t qp = qtn_proc_alloc();
qtn_proc_set_identifier(qp, "xyz.hacktricks.qa");
qtn_proc_set_flags(qp, QTN_FLAG_DOWNLOAD | QTN_FLAG_USER_APPROVED);
qtn_proc_apply_to_self(qp);
qtn_proc_free(qp);

FILE *fp;
fp = fopen("thisisquarantined.txt", "w+");
fprintf(fp, "Hello Quarantine\n");
fclose(fp);

return 0;

}
```
</details>

そして、次のようにその属性を**削除**します:
```bash
xattr -d com.apple.quarantine portada.png
#You can also remove this attribute from every file with
find . -iname '*' -print0 | xargs -0 xattr -d com.apple.quarantine
```
そして、次のコマンドで隔離されたすべてのファイルを検索します：

{% code overflow="wrap" %}
```bash
find / -exec ls -ld {} \; 2>/dev/null | grep -E "[x\-]@ " | awk '{printf $9; printf "\n"}' | xargs -I {} xattr -lv {} | grep "com.apple.quarantine"
```
{% endcode %}

**Quarantine情報**は、LaunchServicesによって管理される中央データベースに保存されます。**`~/Library/Preferences/com.apple.LaunchServices.QuarantineEventsV2`**。

#### **Quarantine.kext**

このカーネル拡張機能は、システムの**カーネルキャッシュ**を介してのみ利用可能です。ただし、**https://developer.apple.com/** から**Kernel Debug Kit**をダウンロードすることができ、このキットには拡張機能のシンボル化されたバージョンが含まれています。

### XProtect

XProtectはmacOSに組み込まれた**対マルウェア**機能です。XProtectは、**アプリケーションが初めて起動されるか変更される際に、既知のマルウェアや安全でないファイルタイプのデータベース**と照合します。Safari、Mail、Messagesなどの特定のアプリを介してファイルをダウンロードすると、XProtectが自動的にファイルをスキャンします。データベース内の既知のマルウェアに一致する場合、XProtectは**ファイルの実行を防止**し、脅威を警告します。

XProtectデータベースは、Appleによって**定期的に更新**され、これらのアップデートは自動的にMacにダウンロードおよびインストールされます。これにより、XProtectが常に最新の既知の脅威に対応していることが保証されます。

ただし、**XProtectは完全なアンチウイルスソリューションではない**ことに注意する価値があります。XProtectは特定の既知の脅威のリストをチェックするだけであり、ほとんどのアンチウイルスソフトウェアのようにアクセス時スキャンを実行しません。

最新のXProtectアップデートに関する情報を取得するには、次のコマンドを実行します：

{% code overflow="wrap" %}
```bash
system_profiler SPInstallHistoryDataType 2>/dev/null | grep -A 4 "XProtectPlistConfigData" | tail -n 5
```
{% endcode %}

XProtectは、**/Library/Apple/System/Library/CoreServices/XProtect.bundle**にあるSIP保護された場所にあり、バンドル内にはXProtectが使用する情報が含まれています：

- **`XProtect.bundle/Contents/Resources/LegacyEntitlementAllowlist.plist`**：これらのcdhashを持つコードがレガシー権限を使用できるようにします。
- **`XProtect.bundle/Contents/Resources/XProtect.meta.plist`**：BundleIDとTeamIDを介してロードを禁止されているプラグインと拡張機能のリスト、または最小バージョンを示すもの。
- **`XProtect.bundle/Contents/Resources/XProtect.yara`**：マルウェアを検出するためのYaraルール。
- **`XProtect.bundle/Contents/Resources/gk.db`**：ブロックされたアプリケーションとTeamIDのハッシュを持つSQLite3データベース。

XProtectに関連する**`/Library/Apple/System/Library/CoreServices/XProtect.app`**に別のアプリがあることに注意してくださいが、これはGatekeeperプロセスとは関係ありません。

### Gatekeeperではない

{% hint style="danger" %}
Gatekeeperは、アプリケーションを実行するたびに実行されるわけではないことに注意してください。_**AppleMobileFileIntegrity**_（AMFI）は、Gatekeeperによってすでに実行および検証されたアプリケーションを実行するときにのみ、**実行可能コードの署名を検証**します。
{% endhint %}

したがって、以前はアプリを実行してGatekeeperでキャッシュすることが可能で、その後、アプリケーションの実行ファイル以外（Electron asarやNIBファイルなど）を**変更**し、他に保護がない場合、アプリケーションは**悪意のある**追加を含めて**実行**されていました。

しかし、現在はmacOSがアプリケーションバンドル内のファイルを**変更できないように防止**しています。そのため、[Dirty NIB](../macos-proces-abuse/macos-dirty-nib.md)攻撃を試みると、Gatekeeperでキャッシュするためにアプリを実行した後、バンドルを変更できなくなるため、悪用することができなくなりました。たとえば、Contentsディレクトリの名前をExploitで示されているようにNotConに変更し、その後、Gatekeeperでキャッシュするためにアプリのメインバイナリを実行すると、エラーが発生して実行されません。

## Gatekeeperのバイパス

Gatekeeperをバイパスする方法（ユーザーに何かをダウンロードさせ、Gatekeeperが拒否すべきときに実行させる方法）は、macOSの脆弱性と見なされます。これまでにGatekeeperをバイパスするための技術に割り当てられたいくつかのCVEは次のとおりです：

### [CVE-2021-1810](https://labs.withsecure.com/publications/the-discovery-of-cve-2021-1810)

**Archive Utility**を使用してファイルを抽出すると、**886文字を超えるパス**を持つファイルにはcom.apple.quarantine拡張属性が付与されません。この状況により、これらのファイルがGatekeeperのセキュリティチェックを**回避**することが誤って許可されます。

詳細については、[**元のレポート**](https://labs.withsecure.com/publications/the-discovery-of-cve-2021-1810)を参照してください。

### [CVE-2021-30990](https://ronmasas.com/posts/bypass-macos-gatekeeper)

**Automator**で作成されたアプリケーションでは、実行に必要な情報が`application.app/Contents/document.wflow`にあり、実行可能ファイルにはありません。実行可能ファイルは、**Automator Application Stub**と呼ばれる一般的なAutomatorバイナリです。

したがって、`application.app/Contents/MacOS/Automator\ Application\ Stub`を**シンボリックリンクでシステム内の別のAutomator Application Stubを指すように**すると、実際の実行可能ファイルに隔離xattrがないため、Gatekeeperをトリガーせずに`document.wflow`（スクリプト）内の内容を実行できます。

期待される場所の例：`/System/Library/CoreServices/Automator\ Application\ Stub.app/Contents/MacOS/Automator\ Application\ Stub`

詳細については、[**元のレポート**](https://ronmasas.com/posts/bypass-macos-gatekeeper)を参照してください。

### [CVE-2022-22616](https://www.jamf.com/blog/jamf-threat-labs-safari-vuln-gatekeeper-bypass/)

このバイパスでは、zipファイルが`application.app`ではなく`application.app/Contents`から圧縮を開始するアプリケーションが作成されました。したがって、**quarantine属性**が`application.app/Contents`のすべての**ファイルに適用**されましたが、Gatekeeperがチェックしていた`application.app`には適用されず、`application.app`がトリガーされたときには**隔離属性がなかった**ため、Gatekeeperがバイパスされました。
```bash
zip -r test.app/Contents test.zip
```
チェックして、詳細情報は[**元のレポート**](https://www.jamf.com/blog/jamf-threat-labs-safari-vuln-gatekeeper-bypass/)を参照してください。

### [CVE-2022-32910](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2022-32910)

コンポーネントが異なる場合でも、この脆弱性の悪用は前のものと非常に似ています。この場合、**`application.app/Contents`** からApple Archiveを生成しますので、**Archive Utility** によって展開される際に **`application.app` には隔離属性が付与されなくなります**。
```bash
aa archive -d test.app/Contents -o test.app.aar
```
チェックして、より詳しい情報を入手してください。

### [CVE-2022-42821](https://www.microsoft.com/en-us/security/blog/2022/12/19/gatekeepers-achilles-heel-unearthing-a-macos-vulnerability/)

ACL **`writeextattr`** は、ファイル内の属性の書き込みを誰もが防ぐために使用できます。
```bash
touch /tmp/no-attr
chmod +a "everyone deny writeextattr" /tmp/no-attr
xattr -w attrname vale /tmp/no-attr
xattr: [Errno 13] Permission denied: '/tmp/no-attr'
```
さらに、**AppleDouble**ファイル形式は、そのACEを含むファイルをコピーします。

[**ソースコード**](https://opensource.apple.com/source/Libc/Libc-391/darwin/copyfile.c.auto.html)では、**`com.apple.acl.text`**というxattrに格納されたACLテキスト表現が、展開されたファイルのACLとして設定されることがわかります。したがって、ACLを設定して他のxattrの書き込みを防止するACLを持つzipファイルにアプリケーションを圧縮した場合... クアランティンxattrはアプリケーションに設定されませんでした:
```bash
chmod +a "everyone deny write,writeattr,writeextattr" /tmp/test
ditto -c -k test test.zip
python3 -m http.server
# Download the zip from the browser and decompress it, the file should be without a quarantine xattr
```
{% endcode %}

詳細については、[**元のレポート**](https://www.microsoft.com/en-us/security/blog/2022/12/19/gatekeepers-achilles-heel-unearthing-a-macos-vulnerability/)をチェックしてください。

AppleArchivesでもこの脆弱性を悪用することができる可能性があることに注意してください：
```bash
mkdir app
touch app/test
chmod +a "everyone deny write,writeattr,writeextattr" app/test
aa archive -d app -o test.aar
```
### [CVE-2023-27943](https://blog.f-secure.com/discovery-of-gatekeeper-bypass-cve-2023-27943/)

**Google Chromeがダウンロードしたファイルにquarantine属性を設定していなかった**ことが、macOSの内部問題によるものであることが発見されました。

### [CVE-2023-27951](https://redcanary.com/blog/gatekeeper-bypass-vulnerabilities/)

AppleDoubleファイル形式は、ファイルの属性を`._`で始まる別のファイルに保存し、これによりmacOSマシン間でファイル属性をコピーするのに役立ちます。しかし、AppleDoubleファイルを展開した後、`._`で始まるファイルには**quarantine属性が付与されていない**ことが気付かれました。
```bash
mkdir test
echo a > test/a
echo b > test/b
echo ._a > test/._a
aa archive -d test/ -o test.aar

# If you downloaded the resulting test.aar and decompress it, the file test/._a won't have a quarantitne attribute
```
{% endcode %}

Gatekeeperをバイパスすることが可能でした。トリックは、AppleDouble名前規則を使用して（`._`で始める）、quarantine属性が設定されていない隠しファイルにシンボリックリンクとして**見えるファイルを作成**し、**DMGファイルアプリケーションを作成**することでした。**dmgファイルが実行される**と、quarantine属性がないため、Gatekeeperを**バイパス**します。
```bash
# Create an app bundle with the backdoor an call it app.app

echo "[+] creating disk image with app"
hdiutil create -srcfolder app.app app.dmg

echo "[+] creating directory and files"
mkdir
mkdir -p s/app
cp app.dmg s/app/._app.dmg
ln -s ._app.dmg s/app/app.dmg

echo "[+] compressing files"
aa archive -d s/ -o app.aar
```
### uchg (from this [talk](https://codeblue.jp/2023/result/pdf/cb23-bypassing-macos-security-and-privacy-mechanisms-from-gatekeeper-to-system-integrity-protection-by-koh-nakagawa.pdf))

* アプリを含むディレクトリを作成します。
* アプリに uchg を追加します。
* アプリを tar.gz ファイルに圧縮します。
* tar.gz ファイルを被害者に送信します。
* 被害者が tar.gz ファイルを開き、アプリを実行します。
* Gatekeeper はアプリをチェックしません。

### Prevent Quarantine xattr

".app" バンドル内に quarantine xattr が追加されていない場合、それを実行すると **Gatekeeper はトリガーされません**。
