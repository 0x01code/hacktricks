# macOS 커널 및 시스템 확장

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert)</strong>를 통해 AWS 해킹을 처음부터 전문가까지 배워보세요<strong>!</strong></summary>

HackTricks를 지원하는 다른 방법:

* **회사를 HackTricks에서 광고하거나 HackTricks를 PDF로 다운로드**하려면 [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)를 확인하세요!
* [**공식 PEASS & HackTricks 스웨그**](https://peass.creator-spring.com)를 얻으세요.
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)를 발견하세요. 독점적인 [**NFT**](https://opensea.io/collection/the-peass-family) 컬렉션입니다.
* 💬 [**Discord 그룹**](https://discord.gg/hRep4RUj7f) 또는 [**텔레그램 그룹**](https://t.me/peass)에 **참여**하거나 **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)을 **팔로우**하세요.
* **Hacking 트릭을 공유하려면** [**HackTricks**](https://github.com/carlospolop/hacktricks) 및 [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github 저장소에 PR을 제출하세요.

</details>

## XNU 커널

**macOS의 핵심은 XNU**로, "X is Not Unix"의 약자입니다. 이 커널은 기본적으로 **Mach 마이크로커널**(나중에 설명)과 **Berkeley Software Distribution (BSD)**의 요소로 구성됩니다. XNU는 또한 **I/O Kit이라는 시스템을 통해 커널 드라이버를 제공**합니다. XNU 커널은 Darwin 오픈 소스 프로젝트의 일부이며, **소스 코드가 자유롭게 접근 가능**합니다.

보안 연구원이나 Unix 개발자의 관점에서 macOS는 우아한 GUI와 다양한 사용자 정의 애플리케이션을 갖춘 **FreeBSD** 시스템과 매우 **유사**할 수 있습니다. BSD에 개발된 대부분의 애플리케이션은 수정 없이 macOS에서 컴파일 및 실행될 수 있습니다. Unix 사용자에게 익숙한 명령 줄 도구들이 모두 macOS에 포함되어 있기 때문입니다. 그러나 XNU 커널은 Mach를 통합하기 때문에 전통적인 Unix와 macOS 간에 중요한 차이점이 있으며, 이러한 차이점은 잠재적인 문제를 일으킬 수 있거나 독특한 이점을 제공할 수 있습니다.

XNU의 오픈 소스 버전: [https://opensource.apple.com/source/xnu/](https://opensource.apple.com/source/xnu/)

### Mach

Mach는 **UNIX 호환성**을 갖춘 **마이크로커널**입니다. 그 중요한 설계 원칙 중 하나는 **커널 공간에서 실행되는 코드의 양을 최소화**하고 파일 시스템, 네트워킹, I/O와 같은 일반적인 커널 기능을 **사용자 수준의 작업으로 실행**할 수 있도록 하는 것입니다.

XNU에서 Mach는 프로세서 스케줄링, 멀티태스킹, 가상 메모리 관리와 같은 커널이 일반적으로 처리하는 많은 중요한 저수준 작업을 담당합니다.

### BSD

XNU 커널은 또한 **FreeBSD** 프로젝트에서 파생된 많은 코드를 **통합**합니다. 이 코드는 Mach와 함께 커널의 일부로 **동일한 주소 공간에서 실행**됩니다. 그러나 XNU 내의 FreeBSD 코드는 Mach와의 호환성을 보장하기 위해 수정이 필요하여 원래 FreeBSD 코드와 크게 다를 수 있습니다. FreeBSD는 다음과 같은 많은 커널 작업에 기여합니다:

* 프로세스 관리
* 신호 처리
* 사용자 및 그룹 관리를 포함한 기본 보안 메커니즘
* 시스템 호출 인프라
* TCP/IP 스택 및 소켓
* 방화벽 및 패킷 필터링

BSD와 Mach 간의 상호 작용을 이해하는 것은 개념적인 차이로 인해 복잡할 수 있습니다. 예를 들어, BSD는 프로세스를 기본 실행 단위로 사용하고, Mach는 스레드를 기반으로 동작합니다. 이러한 불일치는 XNU에서 **각 BSD 프로세스를 정확히 하나의 Mach 스레드를 포함하는 Mach 작업과 연관**시켜 해결됩니다. BSD의 fork() 시스템 호출이 사용될 때, 커널 내의 BSD 코드는 Mach 함수를 사용하여 작업과 스레드 구조를 생성합니다.

또한, **Mach와 BSD는 각각 다른 보안 모델을 유지**합니다. **Mach의** 보안 모델은 **포트 권한**을 기반으로 하며, BSD의 보안 모델은 **프로세스 소유권**을 기반으로 합니다. 이러한 두 모델 간의 차이로 인해 로컬 권한 상승 취약점이 가끔 발생할 수 있습니다. 일반적인 시스템 호출 외에도 **Mach 트랩을 통해 사용자 공간 프로그램이 커널과 상호 작용**할 수 있습니다. 이러한 다른 요소들이 macOS 커널의 다양한 면을 형성합니다.

### I/O Kit - 드라이버

I/O Kit은 XNU 커널의 오픈 소스, 객체 지향 **장치 드라이버 프레임워크**로, **동적으로 로드되는 장치 드라이버**를 처리합니다. 이를 통해 다양한 하드웨어를 지원하기 위해 모듈식 코드를 커널에 동적으로 추가할 수 있습니다.

{% content-ref url="macos-iokit.md" %}
[macos-iokit.md](macos-iokit.md)
{% endcontent-ref %}

### IPC - 프로세스 간 통신

{% content-ref url="macos-ipc-inter-process-communication/" %}
[macos-ipc-inter-process-communication](macos-ipc-inter-process-communication/)
{% endcontent-ref %}

### 커널캐시

**커널캐시**는 XNU 커널의 **미리 컴파일된 및 미리 링크된 버전**으로, 필수 장치 **드라이버** 및 **커널 확장**과 함께 저장됩니다. 이는 **압축된** 형식으로 저장되며 부팅 프로세스 중에 메모리로 압축 해제됩니다. 커널캐시는 준비된 상태의 커널과 중요한 드라이버를 사용할 수 있도록 함으로써 부팅 시 동적으로 이러한 구성 요소를 로드하고 링크하는 데 소요되는 시간과 리소스를 줄여 **빠른 부팅 시간**을 제공합니다.

iOS에서는 **`/System/Library/Caches/com.apple.kernelcaches/kernelcache`**에 위치하며 macOS에서는 **`find / -name kernelcache 2>/dev/null`**로 찾을 수 있습니다.

#### IMG4

IMG4 파일 형식은 Apple의 iOS 및 macOS 장치에서 **펌웨어를 안전하게 저장하고 검증**하기 위해 사용되는 컨테이너 형식입니다(예: **kernelcache**). IMG4 형식은 헤더와 실제 페이로드(커널 또는 부트로더와 같은)를 캡슐화하는 여러 태그를 포함하고 있습니다. 또한 서명과 일련의 매니페스트 속성을 포함합니다. 이 형식은 암호화된 페이로드를 지원하며 장치가 펌웨어 구성 요소를 실행하기 전에 인증 및 무결성을 확인할 수 있도록 암호학적 검증을 지원합니다.

일반적으로 다음 구성 요소로 구성됩니다:

* **페이로드 (IM4P)**:
* 종종 압축됨 (LZFSE4, LZSS, ...)
* 선택적으로 암호화됨
* **매니페스트 (IM4M)**:
* 서명 포함
* 추가 키/값 사전
* **복원 정보 (IM4R)**:
* APNonce로도 알
```bash
# pyimg4 (https://github.com/m1stadev/PyIMG4)
pyimg4 im4p extract -i kernelcache.release.iphone14 -o kernelcache.release.iphone14.e

# img4tool (https://github.com/tihmstar/img4tool
img4tool -e kernelcache.release.iphone14 -o kernelcache.release.iphone14.e
```
#### 커널캐시 심볼

때때로 Apple은 **심볼**이 포함된 **커널캐시**를 공개합니다. [https://theapplewiki.com](https://theapplewiki.com/)의 링크를 따라가면 일부 펌웨어에 심볼이 포함된 것을 다운로드할 수 있습니다.

### IPSW

이는 Apple의 **펌웨어**로 [**https://ipsw.me/**](https://ipsw.me/)에서 다운로드할 수 있습니다. 다른 파일들과 함께 **커널캐시**가 포함되어 있습니다.\
파일을 **추출**하기 위해 그냥 압축을 풀면 됩니다.

펌웨어를 추출한 후에는 **`kernelcache.release.iphone14`**와 같은 파일을 얻게 됩니다. 이 파일은 **IMG4** 형식이며, 다음과 같은 명령으로 관련 정보를 추출할 수 있습니다:

* [**pyimg4**](https://github.com/m1stadev/PyIMG4)

{% code overflow="wrap" %}
```bash
pyimg4 im4p extract -i kernelcache.release.iphone14 -o kernelcache.release.iphone14.e
```
{% endcode %}

* [**img4tool**](https://github.com/tihmstar/img4tool)
```bash
img4tool -e kernelcache.release.iphone14 -o kernelcache.release.iphone14.e
```
다음 명령을 사용하여 추출된 커널캐시에서 심볼을 확인할 수 있습니다: **`nm -a kernelcache.release.iphone14.e | wc -l`**

이제 우리는 **모든 확장자** 또는 **관심 있는 확장자를 추출**할 수 있습니다:
```bash
# List all extensions
kextex -l kernelcache.release.iphone14.e
## Extract com.apple.security.sandbox
kextex -e com.apple.security.sandbox kernelcache.release.iphone14.e

# Extract all
kextex_all kernelcache.release.iphone14.e

# Check the extension for symbols
nm -a binaries/com.apple.security.sandbox | wc -l
```
## macOS 커널 확장자

맥OS는 코드가 실행될 때의 높은 권한 때문에 커널 확장자(.kext)를 로드하는 것이 매우 제한적입니다. 사실, 기본적으로는 거의 불가능합니다(우회 방법을 찾지 않는 한).

{% content-ref url="macos-kernel-extensions.md" %}
[macos-kernel-extensions.md](macos-kernel-extensions.md)
{% endcontent-ref %}

### macOS 시스템 확장자

맥OS는 커널 확장자 대신 시스템 확장자를 만들어, 사용자 수준의 API를 통해 커널과 상호 작용할 수 있도록 제공합니다. 이렇게 함으로써 개발자는 커널 확장자를 사용하지 않을 수 있습니다.

{% content-ref url="macos-system-extensions.md" %}
[macos-system-extensions.md](macos-system-extensions.md)
{% endcontent-ref %}

## 참고 자료

* [**The Mac Hacker's Handbook**](https://www.amazon.com/-/es/Charlie-Miller-ebook-dp-B004U7MUMU/dp/B004U7MUMU/ref=mt\_other?\_encoding=UTF8\&me=\&qid=)
* [**https://taomm.org/vol1/analysis.html**](https://taomm.org/vol1/analysis.html)

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert)</strong>를 통해 제로에서 영웅까지 AWS 해킹 배우기<strong>!</strong></summary>

HackTricks를 지원하는 다른 방법:

* HackTricks에서 **회사 광고를 보거나 PDF로 HackTricks를 다운로드**하려면 [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)를 확인하세요!
* [**공식 PEASS & HackTricks 스웨그**](https://peass.creator-spring.com)를 얻으세요.
* 독점적인 [**NFT**](https://opensea.io/collection/the-peass-family) 컬렉션인 [**The PEASS Family**](https://opensea.io/collection/the-peass-family)를 발견하세요.
* 💬 [**Discord 그룹**](https://discord.gg/hRep4RUj7f) 또는 [**텔레그램 그룹**](https://t.me/peass)에 **참여**하거나 **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)을 **팔로우**하세요.
* **HackTricks**와 **HackTricks Cloud** github 저장소에 PR을 제출하여 여러분의 해킹 기법을 공유하세요.

</details>
