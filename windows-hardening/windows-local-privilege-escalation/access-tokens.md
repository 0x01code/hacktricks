# 액세스 토큰

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert)를 통해 제로부터 영웅까지 AWS 해킹을 배우세요</strong></summary>

* **사이버 보안 회사**에서 일하시나요? **회사가 HackTricks에 광고**되길 원하시나요? 아니면 **PEASS의 최신 버전에 액세스하거나 HackTricks를 PDF로 다운로드**하고 싶으신가요? [**구독 요금제**](https://github.com/sponsors/carlospolop)를 확인해보세요!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)를 발견해보세요, 저희의 독점 [**NFTs**](https://opensea.io/collection/the-peass-family) 컬렉션
* [**공식 PEASS & HackTricks 스왹**](https://peass.creator-spring.com)을 받아보세요
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord 그룹**](https://discord.gg/hRep4RUj7f)이나 [**텔레그램 그룹**](https://t.me/peass)에 **가입**하거나 **트위터** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**를 팔로우**하세요.
* **해킹 트릭을 공유하고 싶다면** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **및** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **로 PR을 제출**하세요.

</details>

## WhiteIntel

<figure><img src=".gitbook/assets/image (1224).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io)은 **다크 웹**을 기반으로 한 검색 엔진으로, **회사 또는 그 고객이 **스틸러 악성 소프트웨어**에 의해 **침해**당했는지 무료로 확인할 수 있는 기능을 제공합니다.

WhiteIntel의 주요 목표는 정보를 도난당한 악성 소프트웨어로 인한 계정 탈취 및 랜섬웨어 공격을 막는 것입니다.

그들의 웹사이트를 방문하여 엔진을 **무료로** 시도해볼 수 있습니다:

{% embed url="https://whiteintel.io" %}

---

## 액세스 토큰

시스템에 **로그인한 각 사용자는 해당 로그온 세션에 대한 보안 정보를 포함한 액세스 토큰을 보유**합니다. 사용자가 로그온할 때 시스템은 액세스 토큰을 생성합니다. 사용자를 대신하여 실행되는 **모든 프로세스에는 액세스 토큰의 사본**이 있습니다. 토큰은 사용자, 사용자의 그룹 및 사용자의 권한을 식별합니다. 또한 토큰에는 현재 로그온 세션을 식별하는 로그온 SID (보안 식별자)도 포함되어 있습니다.

`whoami /all`을 실행하여 이 정보를 확인할 수 있습니다.
```
whoami /all

USER INFORMATION
----------------

User Name             SID
===================== ============================================
desktop-rgfrdxl\cpolo S-1-5-21-3359511372-53430657-2078432294-1001


GROUP INFORMATION
-----------------

Group Name                                                    Type             SID                                                                                                           Attributes
============================================================= ================ ============================================================================================================= ==================================================
Mandatory Label\Medium Mandatory Level                        Label            S-1-16-8192
Everyone                                                      Well-known group S-1-1-0                                                                                                       Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Local account and member of Administrators group Well-known group S-1-5-114                                                                                                     Group used for deny only
BUILTIN\Administrators                                        Alias            S-1-5-32-544                                                                                                  Group used for deny only
BUILTIN\Users                                                 Alias            S-1-5-32-545                                                                                                  Mandatory group, Enabled by default, Enabled group
BUILTIN\Performance Log Users                                 Alias            S-1-5-32-559                                                                                                  Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\INTERACTIVE                                      Well-known group S-1-5-4                                                                                                       Mandatory group, Enabled by default, Enabled group
CONSOLE LOGON                                                 Well-known group S-1-2-1                                                                                                       Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users                              Well-known group S-1-5-11                                                                                                      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization                                Well-known group S-1-5-15                                                                                                      Mandatory group, Enabled by default, Enabled group
MicrosoftAccount\cpolop@outlook.com                           User             S-1-11-96-3623454863-58364-18864-2661722203-1597581903-3158937479-2778085403-3651782251-2842230462-2314292098 Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Local account                                    Well-known group S-1-5-113                                                                                                     Mandatory group, Enabled by default, Enabled group
LOCAL                                                         Well-known group S-1-2-0                                                                                                       Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Cloud Account Authentication                     Well-known group S-1-5-64-36                                                                                                   Mandatory group, Enabled by default, Enabled group


PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                          State
============================= ==================================== ========
SeShutdownPrivilege           Shut down the system                 Disabled
SeChangeNotifyPrivilege       Bypass traverse checking             Enabled
SeUndockPrivilege             Remove computer from docking station Disabled
SeIncreaseWorkingSetPrivilege Increase a process working set       Disabled
SeTimeZonePrivilege           Change the time zone                 Disabled
```
또는 Sysinternals의 _Process Explorer_를 사용하여 (프로세스를 선택하고 "보안" 탭에 액세스):

![](<../../.gitbook/assets/image (769).png>)

### 로컬 관리자

로컬 관리자가 로그인하면 **두 개의 액세스 토큰이 생성**됩니다: 하나는 관리자 권한이 있고 다른 하나는 일반 권한이 있습니다. **기본적으로**이 사용자가 프로세스를 실행할 때는 **일반**(관리자가 아닌) **권한이 사용**됩니다. 이 사용자가 관리자로 **실행**하려고 할 때 ("관리자로 실행" 예를 들어) **UAC**가 권한을 요청하기 위해 사용됩니다.\
[**UAC에 대해 자세히 알아보려면 이 페이지를 읽어보세요**](../authentication-credentials-uac-and-efs/#uac)**.**

### 자격 증명 사용자 위장

다른 사용자의 **유효한 자격 증명**이 있다면 해당 자격 증명으로 **새로운 로그온 세션을 생성**할 수 있습니다:
```
runas /user:domain\username cmd.exe
```
**액세스 토큰**에는 **LSASS** 내의 로그온 세션에 대한 **참조**도 있습니다. 이는 프로세스가 네트워크의 일부 객체에 액세스해야 할 때 유용합니다.\
네트워크 서비스에 액세스하는 데 **다른 자격 증명을 사용하는 프로세스를 실행**할 수 있습니다:
```
runas /user:domain\username /netonly cmd.exe
```
이것은 네트워크 내의 객체에 액세스하기 위한 유용한 자격 증명이 있지만 해당 자격 증명이 현재 호스트 내에서 유효하지 않은 경우 유용합니다(현재 호스트에서는 현재 사용자 권한이 사용됩니다).

### 토큰 유형

사용 가능한 두 가지 유형의 토큰이 있습니다:

* **기본 토큰(Primary Token)**: 프로세스의 보안 자격 증명을 나타냅니다. 기본 토큰의 생성 및 프로세스와의 연결은 권한 상승이 필요한 작업으로, 권한 분리의 원칙을 강조합니다. 일반적으로 인증 서비스가 토큰 생성을 담당하며, 로그온 서비스가 사용자의 운영 체제 셸과의 연결을 처리합니다. 프로세스는 생성 시 부모 프로세스의 기본 토큰을 상속합니다.
* **임시 토큰(Impersonation Token)**: 서버 응용 프로그램이 안전한 객체에 액세스하기 위해 일시적으로 클라이언트의 신원을 취할 수 있게 합니다. 이 메커니즘은 네 가지 수준의 작업으로 계층화됩니다:
  * **익명(Anonymous)**: 식별되지 않은 사용자와 유사한 서버 액세스를 부여합니다.
  * **식별(Identification)**: 객체 액세스에 사용하지 않고 클라이언트의 신원을 확인할 수 있게 합니다.
  * **임시(Impersonation)**: 서버가 클라이언트의 신원으로 작동할 수 있게 합니다.
  * **위임(Delegation)**: 임시와 유사하지만, 서버가 상호 작용하는 원격 시스템에 이 신원 가정을 확장할 수 있는 능력을 포함하여 자격 증명 보존을 보장합니다.

#### 토큰 변장

메타스플로잇의 _**incognito**_ 모듈을 사용하면 충분한 권한이 있다면 다른 **토큰**을 쉽게 **목록화**하고 **변장**할 수 있습니다. 이 기술을 사용하면 **다른 사용자인 것처럼 작업**을 수행하는 데 유용할 수 있습니다. 또한 이 기술로 **권한 상승**도 가능합니다.

### 토큰 권한

**권한 상승에 악용될 수 있는 토큰 권한을 학습하세요:**

{% content-ref url="privilege-escalation-abusing-tokens.md" %}
[privilege-escalation-abusing-tokens.md](privilege-escalation-abusing-tokens.md)
{% endcontent-ref %}

[**모든 가능한 토큰 권한 및 이 외부 페이지에서의 일부 정의를 확인하세요**](https://github.com/gtworek/Priv2Admin).

## 참고 자료

이 튜토리얼에서 토큰에 대해 더 알아보세요: [https://medium.com/@seemant.bisht24/understanding-and-abusing-process-tokens-part-i-ee51671f2cfa](https://medium.com/@seemant.bisht24/understanding-and-abusing-process-tokens-part-i-ee51671f2cfa) 및 [https://medium.com/@seemant.bisht24/understanding-and-abusing-access-tokens-part-ii-b9069f432962](https://medium.com/@seemant.bisht24/understanding-and-abusing-access-tokens-part-ii-b9069f432962)

## WhiteIntel

<figure><img src=".gitbook/assets/image (1224).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io)은 **다크 웹**을 활용한 검색 엔진으로, 회사나 그 고객이 **스틸러 악성 소프트웨어**에 의해 **침해**당했는지 무료로 확인할 수 있는 기능을 제공합니다.

WhiteIntel의 주요 목표는 정보 도난 악성 소프트웨어로 인한 계정 탈취 및 랜섬웨어 공격에 대항하는 것입니다.

그들의 웹사이트를 방문하여 엔진을 **무료로** 사용해보세요:

{% embed url="https://whiteintel.io" %}

<details>

<summary><strong>**htARTE (HackTricks AWS Red Team Expert)**로부터 AWS 해킹을 처음부터 전문가까지 배우세요!</strong></summary>

* **사이버 보안 회사에서 일하시나요? 귀하의 회사를 HackTricks에서 광고하고 싶으신가요? 또는 최신 버전의 PEASS에 액세스하거나 HackTricks를 PDF로 다운로드하고 싶으신가요? [**구독 요금제**](https://github.com/sponsors/carlospolop)를 확인하세요!**
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)를 발견하세요, 당사의 독점 [**NFTs**](https://opensea.io/collection/the-peass-family) 컬렉션
* [**공식 PEASS & HackTricks 스왜그**](https://peass.creator-spring.com)를 얻으세요
* **[💬](https://emojipedia.org/speech-balloon/) [Discord 그룹](https://discord.gg/hRep4RUj7f)** 또는 [텔레그램 그룹](https://t.me/peass)에 가입하거나 **트위터** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**를 팔로우하세요.**
* **해킹 트릭을 공유하려면 PR을 제출하여** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **및** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **에 참여하세요.**

</details>
