# ブラウザのアーティファクト

<details>

<summary><strong>htARTE（HackTricks AWS Red Team Expert）</strong>を使って、ゼロからヒーローまでAWSハッキングを学びましょう！</summary>

HackTricksをサポートする他の方法：

- **HackTricksで企業を宣伝したい**または**HackTricksをPDFでダウンロードしたい**場合は、[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください！
- [**公式PEASS＆HackTricksスワッグ**](https://peass.creator-spring.com)を手に入れる
- [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見し、独占的な[NFTs](https://opensea.io/collection/the-peass-family)のコレクションを見つける
- 💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)または[**telegramグループ**](https://t.me/peass)に**参加**するか、**Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)を**フォロー**する
- **HackTricks**と**HackTricks Cloud**のgithubリポジトリにPRを提出して、あなたのハッキングテクニックを共有する

</details>

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks)を使用して、世界で最も高度なコミュニティツールによって強化された**ワークフローを簡単に構築**および**自動化**します。\
今すぐアクセスしてください：

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## ブラウザのアーティファクト <a href="#id-3def" id="id-3def"></a>

ブラウザのアーティファクトには、ナビゲーション履歴、ブックマーク、キャッシュデータなど、Webブラウザによって保存されるさまざまな種類のデータが含まれます。これらのアーティファクトは、オペレーティングシステム内の特定のフォルダに保管されており、ブラウザごとに場所と名前が異なりますが、一般的には類似したデータ型を保存しています。

以下は、最も一般的なブラウザのアーティファクトの要約です：

- **ナビゲーション履歴**：ユーザーがWebサイトを訪れた履歴で、悪意のあるサイトへの訪問を特定するのに役立ちます。
- **オートコンプリートデータ**：頻繁な検索に基づいた提案で、ナビゲーション履歴と組み合わせると洞察を提供します。
- **ブックマーク**：ユーザーが保存したサイトで、迅速にアクセスできます。
- **拡張機能とアドオン**：ユーザーがインストールしたブラウザの拡張機能やアドオン。
- **キャッシュ**：Webコンテンツ（画像、JavaScriptファイルなど）を保存してWebサイトの読み込み時間を短縮するためのもので、法的解析に貴重です。
- **ログイン情報**：保存されたログイン資格情報。
- **Favicons**：Webサイトに関連付けられたアイコンで、タブやブックマークに表示され、ユーザーの訪問に関する追加情報に役立ちます。
- **ブラウザセッション**：オープンされたブラウザセッションに関連するデータ。
- **ダウンロード**：ブラウザを介してダウンロードされたファイルの記録。
- **フォームデータ**：Webフォームに入力された情報で、将来の自動入力提案のために保存されます。
- **サムネイル**：Webサイトのプレビュー画像。
- **Custom Dictionary.txt**：ユーザーがブラウザの辞書に追加した単語。

## Firefox

Firefoxは、プロファイル内のユーザーデータをオペレーティングシステムに基づいて特定の場所に保存します：

- **Linux**：`~/.mozilla/firefox/`
- **MacOS**：`/Users/$USER/Library/Application Support/Firefox/Profiles/`
- **Windows**：`%userprofile%\AppData\Roaming\Mozilla\Firefox\Profiles\`

これらのディレクトリ内には、`profiles.ini`というファイルがユーザープロファイルのリストを示しています。各プロファイルのデータは、`profiles.ini`と同じディレクトリにある`profiles.ini`内の`Path`変数で名前が付けられたフォルダに保存されます。プロファイルのフォルダが見つからない場合は、削除されている可能性があります。

各プロファイルフォルダ内には、いくつかの重要なファイルがあります：

- **places.sqlite**：履歴、ブックマーク、ダウンロードを保存します。Windows上の[BrowsingHistoryView](https://www.nirsoft.net/utils/browsing\_history\_view.html)などのツールを使用して履歴データにアクセスできます。
- 履歴とダウンロード情報を抽出するために特定のSQLクエリを使用します。
- **bookmarkbackups**：ブックマークのバックアップが含まれています。
- **formhistory.sqlite**：Webフォームデータを保存します。
- **handlers.json**：プロトコルハンドラを管理します。
- **persdict.dat**：カスタム辞書の単語。
- **addons.json**および**extensions.sqlite**：インストールされたアドオンと拡張機能に関する情報。
- **cookies.sqlite**：Cookieの保存先で、Windows上で[MZCookiesView](https://www.nirsoft.net/utils/mzcv.html)を使用して検査できます。
- **cache2/entries**または**startupCache**：キャッシュデータで、[MozillaCacheView](https://www.nirsoft.net/utils/mozilla\_cache\_viewer.html)などのツールを使用してアクセスできます。
- **favicons.sqlite**：faviconsを保存します。
- **prefs.js**：ユーザー設定と環境設定。
- **downloads.sqlite**：古いダウンロードデータベースで、現在はplaces.sqliteに統合されています。
- **thumbnails**：Webサイトのサムネイル。
- **logins.json**：暗号化されたログイン情報。
- **key4.db**または**key3.db**：機密情報を保護するための暗号化キーを保存します。

さらに、ブラウザのフィッシング対策設定を確認するには、`prefs.js`内で`browser.safebrowsing`エントリを検索して、安全なブラウジング機能が有効になっているか無効になっているかを確認できます。

マスターパスワードを復号化しようとする場合は、[https://github.com/unode/firefox\_decrypt](https://github.com/unode/firefox\_decrypt)を使用できます。\
次のスクリプトと呼び出しを使用して、ブルートフォースするパスワードファイルを指定できます：

{% code title="brute.sh" %}
```bash
#!/bin/bash

#./brute.sh top-passwords.txt 2>/dev/null | grep -A2 -B2 "chrome:"
passfile=$1
while read pass; do
echo "Trying $pass"
echo "$pass" | python firefox_decrypt.py
done < $passfile
```
## Google Chrome

Google Chromeは、オペレーティングシステムに基づいて特定の場所にユーザープロファイルを保存します：

- **Linux**: `~/.config/google-chrome/`
- **Windows**: `C:\Users\XXX\AppData\Local\Google\Chrome\User Data\`
- **MacOS**: `/Users/$USER/Library/Application Support/Google/Chrome/`

これらのディレクトリ内で、ほとんどのユーザーデータは **Default/** や **ChromeDefaultData/** フォルダにあります。重要なデータを保持する以下のファイルがあります：

- **History**: URL、ダウンロード、検索キーワードを含む。Windowsでは、[ChromeHistoryView](https://www.nirsoft.net/utils/chrome_history_view.html)を使用して履歴を読むことができます。"Transition Type"列には、リンクのクリック、入力されたURL、フォームの送信、ページの再読み込みなど、さまざまな意味があります。
- **Cookies**: クッキーを保存します。検査には、[ChromeCookiesView](https://www.nirsoft.net/utils/chrome_cookies_view.html)が利用できます。
- **Cache**: キャッシュされたデータを保持します。検査するために、Windowsユーザーは[ChromeCacheView](https://www.nirsoft.net/utils/chrome_cache_view.html)を利用できます。
- **Bookmarks**: ユーザーのブックマーク。
- **Web Data**: フォーム履歴を含む。
- **Favicons**: ウェブサイトのファビコンを保存します。
- **Login Data**: ユーザー名やパスワードなどのログイン資格情報を含みます。
- **Current Session**/**Current Tabs**: 現在のブラウジングセッションとオープンされているタブに関するデータ。
- **Last Session**/**Last Tabs**: Chromeが閉じられる前の最後のセッション中にアクティブだったサイトに関する情報。
- **Extensions**: ブラウザの拡張機能やアドオンのためのディレクトリ。
- **Thumbnails**: ウェブサイトのサムネイルを保存します。
- **Preferences**: プラグイン、拡張機能、ポップアップ、通知などの設定を含む情報が豊富なファイル。
- **ブラウザの組み込みのフィッシング対策**: フィッシング対策やマルウェア保護が有効になっているかどうかを確認するには、`grep 'safebrowsing' ~/Library/Application Support/Google/Chrome/Default/Preferences`を実行します。出力で `{"enabled: true,"}` を探します。

## **SQLite DBデータの回復**

前のセクションで確認できるように、ChromeとFirefoxはデータを保存するために **SQLite** データベースを使用しています。ツール [**sqlparse**](https://github.com/padfoot999/sqlparse) または [**sqlparse\_gui**](https://github.com/mdegrazia/SQLite-Deleted-Records-Parser/releases) を使用して、削除されたエントリを回復することが可能です。

## **Internet Explorer 11**

Internet Explorer 11は、異なる場所にデータとメタデータを管理しており、格納された情報とそれに対応する詳細を簡単にアクセスおよび管理するのに役立ちます。

### メタデータの保存

Internet Explorerのメタデータは `%userprofile%\Appdata\Local\Microsoft\Windows\WebCache\WebcacheVX.data`（VXはV01、V16、またはV24）に保存されます。これに加えて、`V01.log`ファイルは `WebcacheVX.data` との修正時間の不一致を示す場合があり、`esentutl /r V01 /d` を使用して修復が必要となります。このESEデータベースに格納されたメタデータは、photorec や [ESEDatabaseView](https://www.nirsoft.net/utils/ese_database_view.html) などのツールを使用して回復および検査することができます。**Containers** テーブル内では、各データセグメントが格納されている特定のテーブルやコンテナを識別することができ、Skypeなどの他のMicrosoftツールのキャッシュの詳細も含まれます。

### キャッシュの検査

[IECacheView](https://www.nirsoft.net/utils/ie_cache_viewer.html) ツールを使用すると、キャッシュデータの抽出フォルダの場所が必要となり、キャッシュのメタデータにはファイル名、ディレクトリ、アクセス回数、URLの起源、キャッシュの作成、アクセス、修正、有効期限の時間が示されます。

### クッキーの管理

クッキーは [IECookiesView](https://www.nirsoft.net/utils/iecookies.html) を使用して調査することができ、メタデータには名前、URL、アクセス回数、さまざまな時間関連の詳細が含まれます。永続的なクッキーは `%userprofile%\Appdata\Roaming\Microsoft\Windows\Cookies` に保存され、セッションクッキーはメモリに保存されます。

### ダウンロードの詳細

ダウンロードのメタデータは [ESEDatabaseView](https://www.nirsoft.net/utils/ese_database_view.html) を介してアクセスでき、特定のコンテナにはURL、ファイルタイプ、ダウンロード場所などのデータが保持されます。物理ファイルは `%userprofile%\Appdata\Roaming\Microsoft\Windows\IEDownloadHistory` にあります。

### 閲覧履歴

閲覧履歴を確認するには、[BrowsingHistoryView](https://www.nirsoft.net/utils/browsing_history_view.html) を使用し、抽出された履歴ファイルの場所とInternet Explorerの構成が必要です。ここでのメタデータには、修正時間、アクセス回数などが含まれます。履歴ファイルは `%userprofile%\Appdata\Local\Microsoft\Windows\History` にあります。

### 入力されたURL

入力されたURLとその使用時刻は、`NTUSER.DAT` 内の `Software\Microsoft\InternetExplorer\TypedURLs` および `Software\Microsoft\InternetExplorer\TypedURLsTime` に格納されており、ユーザーが入力した最後の50のURLとその最終入力時刻を追跡しています。

## Microsoft Edge

Microsoft Edgeは、ユーザーデータを `%userprofile%\Appdata\Local\Packages` に保存します。さまざまなデータタイプのパスは次のとおりです：

- **プロファイルパス**: `C:\Users\XX\AppData\Local\Packages\Microsoft.MicrosoftEdge_XXX\AC`
- **履歴、クッキー、ダウンロード**: `C:\Users\XX\AppData\Local\Microsoft\Windows\WebCache\WebCacheV01.dat`
- **設定、ブックマーク、読書リスト**: `C:\Users\XX\AppData\Local\Packages\Microsoft.MicrosoftEdge_XXX\AC\MicrosoftEdge\User\Default\DataStore\Data\nouser1\XXX\DBStore\spartan.edb`
- **キャッシュ**: `C:\Users\XXX\AppData\Local\Packages\Microsoft.MicrosoftEdge_XXX\AC#!XXX\MicrosoftEdge\Cache`
- **最後のアクティブセッション**: `C:\Users\XX\AppData\Local\Packages\Microsoft.MicrosoftEdge_XXX\AC\MicrosoftEdge\User\Default\Recovery\Active`

## Safari

Safariのデータは `/Users/$User/Library/Safari` に保存されます。主要なファイルは次のとおりです：

- **History.db**: URLと訪問タイムスタンプを含む `history_visits` および `history_items` テーブルが含まれています。クエリを実行するには `sqlite3` を使用します。
- **Downloads.plist**: ダウンロードしたファイルに関する情報。
- **Bookmarks.plist**: ブックマークされたURLを保存します。
- **TopSites.plist**: 最も頻繁に訪れたサイト。
- **Extensions.plist**: Safariブラウザの拡張機能のリスト。取得するには `plutil` または `pluginkit` を使用します。
- **UserNotificationPermissions.plist**: 通知をプッシュすることが許可されたドメイン。解析するには `plutil` を使用します。
- **LastSession.plist**: 最後のセッションからのタブ。解析するには `plutil` を使用します。
- **ブラウザの組み込みのフィッシング対策**: `defaults read com.apple.Safari WarnAboutFraudulentWebsites` を使用して確認します。1が返されると、その機能が有効になっていることを示します。

## Opera

Operaのデータは `/Users/$USER/Library/Application Support/com.operasoftware.Opera` にあり、履歴やダウンロードに関してはChromeと同じ形式を共有しています。

- **ブラウザの組み込みのフィッシング対策**: `fraud_protection_enabled` が `true` に設定されているかどうかを確認するには、`grep` を使用してください。

これらのパスとコマンドは、異なるウェブブラウザによって保存されるブラウジングデータにアクセスして理解するために重要です。
* **HackTricks** に掲載されたい場合や **HackTricks の PDF をダウンロード** したい場合は、[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) をチェックしてください！
* [**公式 PEASS & HackTricks スワッグ**](https://peass.creator-spring.com) を入手してください
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) を発見し、独占的な [**NFTs**](https://opensea.io/collection/the-peass-family) のコレクションをご覧ください
* **💬 [**Discord グループ**](https://discord.gg/hRep4RUj7f) に参加するか、[**telegram グループ**](https://t.me/peass) に参加するか、**Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** をフォローしてください**
* **ハッキングテクニックを共有するには、[**HackTricks**](https://github.com/carlospolop/hacktricks) と [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) の github リポジトリに PR を提出してください**.
