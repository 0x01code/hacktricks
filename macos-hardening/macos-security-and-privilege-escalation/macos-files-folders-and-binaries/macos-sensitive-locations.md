# macOSのセンシティブな場所と興味深いデーモン

{% hint style="success" %}
AWSハッキングの学習と実践：<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCPハッキングの学習と実践：<img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricksのサポート</summary>

* [**サブスクリプションプラン**](https://github.com/sponsors/carlospolop)をチェック！
* 💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)に参加するか、[**telegramグループ**](https://t.me/peass)に参加するか、**Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**をフォロー**してください。
* ハッキングトリックを共有するために、[**HackTricks**](https://github.com/carlospolop/hacktricks)と[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud)のGitHubリポジトリにPRを提出してください。

</details>
{% endhint %}

## パスワード

### シャドウパスワード

シャドウパスワードは、ユーザーの構成と共に**`/var/db/dslocal/nodes/Default/users/`**にあるplistに格納されています。\
次のワンライナーを使用して、**ユーザーに関するすべての情報**（ハッシュ情報を含む）をダンプできます：

{% code overflow="wrap" %}
```bash
for l in /var/db/dslocal/nodes/Default/users/*; do if [ -r "$l" ];then echo "$l"; defaults read "$l"; fi; done
```
{% endcode %}

[**このようなスクリプト**](https://gist.github.com/teddziuba/3ff08bdda120d1f7822f3baf52e606c2)または[**このようなもの**](https://github.com/octomagon/davegrohl.git)は、ハッシュを**hashcat** **形式**に変換するために使用できます。

すべての非サービスアカウントのクレデンシャルをmacOS PBKDF2-SHA512形式のhashcat形式でダンプする代替のワンライナー：

{% code overflow="wrap" %}
```bash
sudo bash -c 'for i in $(find /var/db/dslocal/nodes/Default/users -type f -regex "[^_]*"); do plutil -extract name.0 raw $i | awk "{printf \$0\":\$ml\$\"}"; for j in {iterations,salt,entropy}; do l=$(k=$(plutil -extract ShadowHashData.0 raw $i) && base64 -d <<< $k | plutil -extract SALTED-SHA512-PBKDF2.$j raw -); if [[ $j == iterations ]]; then echo -n $l; else base64 -d <<< $l | xxd -p -c 0 | awk "{printf \"$\"\$0}"; fi; done; echo ""; done'
```
{% endcode %}

### キーチェーンのダンプ

セキュリティバイナリを使用してパスワードを復号化してダンプする際には、ユーザーにこの操作を許可するように求めるプロンプトが複数表示されることに注意してください。
```bash
#security
secuirty dump-trust-settings [-s] [-d] #List certificates
security list-keychains #List keychain dbs
security list-smartcards #List smartcards
security dump-keychain | grep -A 5 "keychain" | grep -v "version" #List keychains entries
security dump-keychain -d #Dump all the info, included secrets (the user will be asked for his password, even if root)
```
### [Keychaindump](https://github.com/juuso/keychaindump)

{% hint style="danger" %}
[このコメント](https://github.com/juuso/keychaindump/issues/10#issuecomment-751218760)に基づくと、これらのツールはBig Surではもはや機能しないようです。
{% endhint %}

### Keychaindump 概要

**keychaindump**というツールは、macOSのキーチェーンからパスワードを抽出するために開発されましたが、Big Surなどの新しいmacOSバージョンでは制限があります。[議論](https://github.com/juuso/keychaindump/issues/10#issuecomment-751218760)で示されているように、**keychaindump**の使用には攻撃者がアクセス権を取得して特権を昇格させる必要があります。このツールは、キーチェーンがユーザーのログイン時にデフォルトでロック解除されていることを悪用しており、ユーザーのパスワードを繰り返し入力することなくアプリケーションがアクセスできるようにしています。ただし、ユーザーが各使用後にキーチェーンをロックすることを選択した場合、**keychaindump**は効果がありません。

**Keychaindump**は、Appleによって認証と暗号操作のためのデーモンとして説明される**securityd**という特定のプロセスを対象として動作します。抽出プロセスには、ユーザーのログインパスワードから派生した**Master Key**を特定することが含まれます。このキーは、キーチェーンファイルを読み取るために不可欠です。**keychaindump**は、`vmmap`コマンドを使用して**securityd**のメモリヒープをスキャンし、`MALLOC_TINY`としてフラグ付けされた領域内の潜在的なキーを探します。これらのメモリ位置を調査するためには、次のコマンドが使用されます：
```bash
sudo vmmap <securityd PID> | grep MALLOC_TINY
```
潜在的なマスターキーを特定した後、**keychaindump** はヒープを検索し、マスターキーの候補を示す特定のパターン (`0x0000000000000018`) を探します。このキーを利用するには、**keychaindump** のソースコードで詳細に説明されているように、さらなるステップが必要です。この領域に焦点を当てるアナリストは、キーチェーンを復号化するための重要なデータが **securityd** プロセスのメモリに格納されていることに注意する必要があります。**keychaindump** を実行するための例として、次のコマンドがあります：
```bash
sudo ./keychaindump
```
### chainbreaker

[**Chainbreaker**](https://github.com/n0fate/chainbreaker)は、次の種類の情報をOSXキーチェーンから法的に適切な方法で抽出するために使用できます：

* ハッシュ化されたキーチェーンパスワード、[hashcat](https://hashcat.net/hashcat/)や[John the Ripper](https://www.openwall.com/john/)でクラック可能
* インターネットパスワード
* 一般的なパスワード
* プライベートキー
* パブリックキー
* X509証明書
* セキュアノート
* Appleshareパスワード

キーチェーンのアンロックパスワード、[volafox](https://github.com/n0fate/volafox)や[volatility](https://github.com/volatilityfoundation/volatility)を使用して取得したマスターキー、またはSystemKeyなどのアンロックファイルを使用すると、Chainbreakerは平文パスワードも提供します。

これらのいずれかの方法でキーチェーンをアンロックしない場合、Chainbreakerは他の利用可能な情報をすべて表示します。

#### **キーチェーンキーをダンプ**
```bash
#Dump all keys of the keychain (without the passwords)
python2.7 chainbreaker.py --dump-all /Library/Keychains/System.keychain
```
#### **SystemKeyを使用してキーチェーンキー（パスワード付き）をダンプする**
```bash
# First, get the keychain decryption key
# To get this decryption key you need to be root and SIP must be disabled
hexdump -s 8 -n 24 -e '1/1 "%.2x"' /var/db/SystemKey && echo
## Use the previous key to decrypt the passwords
python2.7 chainbreaker.py --dump-all --key 0293847570022761234562947e0bcd5bc04d196ad2345697 /Library/Keychains/System.keychain
```
#### **ハッシュをクラックしてキーチェーンのキー（パスワード付き）をダンプする**
```bash
# Get the keychain hash
python2.7 chainbreaker.py --dump-keychain-password-hash /Library/Keychains/System.keychain
# Crack it with hashcat
hashcat.exe -m 23100 --keep-guessing hashes.txt dictionary.txt
# Use the key to decrypt the passwords
python2.7 chainbreaker.py --dump-all --key 0293847570022761234562947e0bcd5bc04d196ad2345697 /Library/Keychains/System.keychain
```
#### **メモリーダンプを使用してキーチェーンキー（パスワード付き）をダンプする**

[これらの手順に従います](../#dumping-memory-with-osxpmem) **メモリーダンプ**を実行します
```bash
#Use volafox (https://github.com/n0fate/volafox) to extract possible keychain passwords
# Unformtunately volafox isn't working with the latest versions of MacOS
python vol.py -i ~/Desktop/show/macosxml.mem -o keychaindump

#Try to extract the passwords using the extracted keychain passwords
python2.7 chainbreaker.py --dump-all --key 0293847570022761234562947e0bcd5bc04d196ad2345697 /Library/Keychains/System.keychain
```
#### **ユーザーパスワードを使用してキーチェーンキー（パスワード付き）をダンプする**

ユーザーパスワードを知っている場合、それを使用してユーザーに属するキーチェーンをダンプおよび復号化できます。
```bash
#Prompt to ask for the password
python2.7 chainbreaker.py --dump-all --password-prompt /Users/<username>/Library/Keychains/login.keychain-db
```
### kcpassword

**kcpassword**ファイルは、**ユーザーのログインパスワード**を保持するファイルですが、システム所有者が**自動ログインを有効にしている場合**にのみ該当します。したがって、ユーザーはパスワードを求められることなく自動的にログインされます（これはあまり安全ではありません）。

パスワードは、ファイル**`/etc/kcpassword`**に**`0x7D 0x89 0x52 0x23 0xD2 0xBC 0xDD 0xEA 0xA3 0xB9 0x1F`**というキーでXOR演算されて格納されます。ユーザーのパスワードがキーよりも長い場合、キーは再利用されます。\
これにより、パスワードはかなり簡単に回復できます。たとえば、[**このようなスクリプト**](https://gist.github.com/opshope/32f65875d45215c3677d)を使用することができます。

## データベース内の興味深い情報

### メッセージ
```bash
sqlite3 $HOME/Library/Messages/chat.db .tables
sqlite3 $HOME/Library/Messages/chat.db 'select * from message'
sqlite3 $HOME/Library/Messages/chat.db 'select * from attachment'
sqlite3 $HOME/Library/Messages/chat.db 'select * from deleted_messages'
sqlite3 $HOME/Suggestions/snippets.db 'select * from emailSnippets'
```
### 通知

通知データは`$(getconf DARWIN_USER_DIR)/com.apple.notificationcenter/`にあります。

興味深い情報のほとんどは**blob**に含まれています。したがって、そのコンテンツを**抽出**して**人間が読める形式**に**変換**するか、**`strings`**を使用する必要があります。アクセスするには、次のようにします：

{% code overflow="wrap" %}
```bash
cd $(getconf DARWIN_USER_DIR)/com.apple.notificationcenter/
strings $(getconf DARWIN_USER_DIR)/com.apple.notificationcenter/db2/db | grep -i -A4 slack
```
{% endcode %}

### ノート

ユーザーの**ノート**は`~/Library/Group Containers/group.com.apple.notes/NoteStore.sqlite`に保存されています。 

{% code overflow="wrap" %}
```bash
sqlite3 ~/Library/Group\ Containers/group.com.apple.notes/NoteStore.sqlite .tables

#To dump it in a readable format:
for i in $(sqlite3 ~/Library/Group\ Containers/group.com.apple.notes/NoteStore.sqlite "select Z_PK from ZICNOTEDATA;"); do sqlite3 ~/Library/Group\ Containers/group.com.apple.notes/NoteStore.sqlite "select writefile('body1.gz.z', ZDATA) from ZICNOTEDATA where Z_PK = '$i';"; zcat body1.gz.Z ; done
```
{% endcode %}

## Preferences

macOSアプリケーションの設定は**`$HOME/Library/Preferences`**にあり、iOSの場合は`/var/mobile/Containers/Data/Application/<UUID>/Library/Preferences`にあります。&#x20;

macOSでは、**`defaults`**というCLIツールを使用して**設定ファイルを変更**できます。

**`/usr/sbin/cfprefsd`**はXPCサービス`com.apple.cfprefsd.daemon`と`com.apple.cfprefsd.agent`を所有し、設定を変更するなどのアクションを実行するために呼び出すことができます。

## システム通知

### Darwin通知

通知のためのメインデーモンは**`/usr/sbin/notifyd`**です。通知を受信するために、クライアントは`com.apple.system.notification_center` Machポートを介して登録する必要があります（`sudo lsmp -p <pid notifyd>`で確認できます）。デーモンはファイル`/etc/notify.conf`で構成可能です。

通知に使用される名前は一意の逆DNS表記であり、通知が送信されると、それを処理できると示したクライアントがそれを受信します。

現在の状態をダンプして（すべての名前を表示することができます）、notifydプロセスにSIGUSR2シグナルを送信し、生成されたファイル`/var/run/notifyd_<pid>.status`を読み取ることができます：
```bash
ps -ef | grep -i notifyd
0   376     1   0 15Mar24 ??        27:40.97 /usr/sbin/notifyd

sudo kill -USR2 376

cat /var/run/notifyd_376.status
[...]
pid: 94379   memory 5   plain 0   port 0   file 0   signal 0   event 0   common 10
memory: com.apple.system.timezone
common: com.apple.analyticsd.running
common: com.apple.CFPreferences._domainsChangedExternally
common: com.apple.security.octagon.joined-with-bottle
[...]
```
### 分散通知センター

**分散通知センター**は、メインバイナリが**`/usr/sbin/distnoted`**であるもう一つの通知の送信方法です。いくつかのXPCサービスを公開し、クライアントを検証しようとするいくつかのチェックを実行します。

### Apple Push Notifications (APN)

この場合、アプリケーションは**トピック**に登録できます。クライアントは**`apsd`**を介してAppleのサーバーに接触してトークンを生成します。\
その後、プロバイダーはトークンを生成し、Appleのサーバーに接続してクライアントにメッセージを送信できます。これらのメッセージは**`apsd`**によってローカルで受信され、それを待っているアプリケーションに通知が中継されます。

設定は`/Library/Preferences/com.apple.apsd.plist`にあります。

macOSにはメッセージのローカルデータベースがあり、`/Library/Application\ Support/ApplePushService/aps.db`に、iOSには`/var/mobile/Library/ApplePushService`にあります。3つのテーブルがあります: `incoming_messages`、`outgoing_messages`、`channel`。
```bash
sudo sqlite3 /Library/Application\ Support/ApplePushService/aps.db
```
次の方法でも、デーモンと接続に関する情報を取得することができます：
```bash
/System/Library/PrivateFrameworks/ApplePushService.framework/apsctl status
```
## ユーザー通知

これらは、ユーザーが画面で見る必要がある通知です：

- **`CFUserNotification`**: このAPIは、画面にメッセージ付きのポップアップを表示する方法を提供します。
- **掲示板**: これはiOSでバナーを表示し、消えて通知センターに保存されます。
- **`NSUserNotificationCenter`**: これはMacOSのiOS掲示板です。通知のデータベースは`/var/folders/<user temp>/0/com.apple.notificationcenter/db2/db`にあります。
