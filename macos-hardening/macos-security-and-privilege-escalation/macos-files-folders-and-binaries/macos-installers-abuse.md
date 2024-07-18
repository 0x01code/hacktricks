# macOSインストーラーの乱用

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

## Pkgの基本情報

macOSの**インストーラーパッケージ**（または`.pkg`ファイルとしても知られる）は、ソフトウェアを**配布**するためにmacOSが使用するファイル形式です。これらのファイルは、インストールと正常な実行に必要なすべてのものを含む**ボックスのようなもの**です。

パッケージファイル自体は、**ターゲットコンピューターにインストールされるファイルとディレクトリの階層**を保持するアーカイブです。また、**設定ファイルのセットアップや古いバージョンのソフトウェアのクリーンアップなど**を行うためのスクリプトも含めることができます。

### 階層

<figure><img src="../../../.gitbook/assets/Pasted Graphic.png" alt="https://www.youtube.com/watch?v=iASSG0_zobQ"><figcaption></figcaption></figure>

* **Distribution (xml)**: カスタマイズ（タイトル、ウェルカムテキストなど）およびスクリプト/インストールチェック
* **PackageInfo (xml)**: 情報、インストール要件、インストール場所、実行するスクリプトへのパス
* **Bill of materials (bom)**: ファイルのリスト（ファイルの権限を含む）をインストール、更新、削除する
* **Payload (CPIOアーカイブgzip圧縮)**: PackageInfoから`install-location`にインストールするファイル
* **Scripts (CPIOアーカイブgzip圧縮)**: インストール前およびインストール後のスクリプトおよび実行用に一時ディレクトリに展開されるその他のリソース

### 解凍
```bash
# Tool to directly get the files inside a package
pkgutil —expand "/path/to/package.pkg" "/path/to/out/dir"

# Get the files ina. more manual way
mkdir -p "/path/to/out/dir"
cd "/path/to/out/dir"
xar -xf "/path/to/package.pkg"

# Decompress also the CPIO gzip compressed ones
cat Scripts | gzip -dc | cpio -i
cpio -i < Scripts
```
## DMGの基本情報

DMGファイル、またはApple Disk Imagesは、AppleのmacOSで使用されるディスクイメージのファイル形式です。DMGファイルは基本的には**マウント可能なディスクイメージ**（独自のファイルシステムを含む）であり、通常は圧縮され、時には暗号化された生のブロックデータを含んでいます。DMGファイルを開くと、macOSはそれを物理ディスクのように**マウント**し、その内容にアクセスできるようにします。

{% hint style="danger" %}
**`.dmg`**インストーラは**多くの形式**をサポートしているため、過去には脆弱性を含むものが悪用され、**カーネルコードの実行**を取得するために使用されたことがあります。
{% endhint %}

### 階層

<figure><img src="../../../.gitbook/assets/image (225).png" alt=""><figcaption></figcaption></figure>

DMGファイルの階層は、内容に基づいて異なる場合があります。ただし、アプリケーションのDMGの場合、通常はこの構造に従います：

- トップレベル：これはディスクイメージのルートです。通常、アプリケーションとおそらくApplicationsフォルダへのリンクが含まれています。
- アプリケーション（.app）：これが実際のアプリケーションです。macOSでは、アプリケーションは通常、アプリケーションを構成する多くの個々のファイルとフォルダを含むパッケージです。
- Applicationsリンク：これはmacOSのApplicationsフォルダへのショートカットです。これは、アプリケーションをインストールしやすくするためのものです。.appファイルをこのショートカットにドラッグしてアプリをインストールできます。

## pkgの悪用による特権昇格

### 公開ディレクトリからの実行

たとえば、事前または事後のインストールスクリプトが**`/var/tmp/Installerutil`**から実行されている場合、攻撃者はそのスクリプトを制御できるため、実行されるたびに特権を昇格させることができます。また、別の類似の例：

<figure><img src="../../../.gitbook/assets/Pasted Graphic 5.png" alt="https://www.youtube.com/watch?v=iASSG0_zobQ"><figcaption><p><a href="https://www.youtube.com/watch?v=kCXhIYtODBg">https://www.youtube.com/watch?v=kCXhIYtODBg</a></p></figcaption></figure>

### AuthorizationExecuteWithPrivileges

これは、いくつかのインストーラやアップデータが**rootとして何かを実行**するために呼び出す[公開関数](https://developer.apple.com/documentation/security/1540038-authorizationexecutewithprivileg)です。この関数は、**実行するファイル**の**パス**をパラメータとして受け入れますが、攻撃者がこのファイルを**変更**できれば、特権を昇格させるためにその実行を**悪用**することができます。
```bash
# Breakpoint in the function to check wich file is loaded
(lldb) b AuthorizationExecuteWithPrivileges
# You could also check FS events to find this missconfig
```
### マウントによる実行

インストーラが`/tmp/fixedname/bla/bla`に書き込む場合、`/tmp/fixedname`に所有者がいないマウントを作成して、インストール中に**任意のファイルを変更**してインストールプロセスを悪用することが可能です。

これの例として、**CVE-2021-26089**があり、**定期スクリプトを上書き**してルートとして実行することに成功しました。詳細については、以下のトークをご覧ください: [**OBTS v4.0: "Mount(ain) of Bugs" - Csaba Fitzl**](https://www.youtube.com/watch?v=jSYPazD4VcE)

## マルウェアとしてのpkg

### 空のペイロード

**`.pkg`**ファイルに**事前および事後インストールスクリプト**を生成するだけでも可能です。

### Distribution xml内のJS

パッケージの**distribution xml**ファイルに**`<script>`**タグを追加することができ、そのコードが実行され、**`system.run`**を使用して**コマンドを実行**することができます:

<figure><img src="../../../.gitbook/assets/image (1043).png" alt=""><figcaption></figcaption></figure>

## 参考文献

* [**DEF CON 27 - Unpacking Pkgs A Look Inside Macos Installer Packages And Common Security Flaws**](https://www.youtube.com/watch?v=iASSG0\_zobQ)
* [**OBTS v4.0: "The Wild World of macOS Installers" - Tony Lambert**](https://www.youtube.com/watch?v=Eow5uNHtmIg)
* [**DEF CON 27 - Unpacking Pkgs A Look Inside MacOS Installer Packages**](https://www.youtube.com/watch?v=kCXhIYtODBg)
