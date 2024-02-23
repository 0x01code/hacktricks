# macOS 프로세스 남용

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert)</strong>를 통해 **제로부터 영웅까지 AWS 해킹 배우기**!</summary>

HackTricks를 지원하는 다른 방법:

* **회사를 HackTricks에서 광고**하거나 **PDF로 HackTricks 다운로드**하려면 [**구독 요금제**](https://github.com/sponsors/carlospolop)를 확인하세요!
* [**공식 PEASS & HackTricks 스왜그**](https://peass.creator-spring.com)를 얻으세요
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)를 발견하세요, 당사의 독점 [**NFTs**](https://opensea.io/collection/the-peass-family) 컬렉션
* **💬 [디스코드 그룹](https://discord.gg/hRep4RUj7f)** 또는 [텔레그램 그룹](https://t.me/peass)에 **가입**하거나 **트위터** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**를 팔로우**하세요.
* **HackTricks** 및 **HackTricks Cloud** github 저장소에 PR을 제출하여 **해킹 트릭을 공유**하세요.

</details>

## macOS 프로세스 남용

맥OS는 다른 운영 체제와 마찬가지로 **프로세스가 상호 작용하고 통신하며 데이터를 공유하는 다양한 방법과 메커니즘**을 제공합니다. 이러한 기술은 효율적인 시스템 작동에 필수적이지만, 위협 요소들이 **악의적인 활동을 수행**하는 데 남용될 수도 있습니다.

### 라이브러리 주입

라이브러리 주입은 공격자가 **프로세스에 악성 라이브러리를 로드하도록 강제하는 기술**입니다. 주입된 라이브러리는 대상 프로세스의 컨텍스트에서 실행되어 공격자에게 프로세스와 동일한 권한과 액세스를 제공합니다.

{% content-ref url="macos-library-injection/" %}
[macos-library-injection](macos-library-injection/)
{% endcontent-ref %}

### 함수 후킹

함수 후킹은 소프트웨어 코드 내에서 **함수 호출이나 메시지를 가로채는** 기술입니다. 함수 후킹을 통해 공격자는 프로세스의 동작을 **수정하거나 민감한 데이터를 관찰하거나 실행 흐름을 제어**할 수 있습니다.

{% content-ref url="../mac-os-architecture/macos-function-hooking.md" %}
[macos-function-hooking.md](../mac-os-architecture/macos-function-hooking.md)
{% endcontent-ref %}

### 프로세스 간 통신

프로세스 간 통신 (IPC)은 별도의 프로세스가 **데이터를 공유하고 교환하는 다양한 방법**을 가리킵니다. IPC는 많은 합법적인 응용 프로그램에 필수적이지만, 프로세스 격리를 무력화하거나 민감한 정보를 유출하거나 무단 조치를 취할 수 있도록 남용될 수도 있습니다.

{% content-ref url="../mac-os-architecture/macos-ipc-inter-process-communication/" %}
[macos-ipc-inter-process-communication](../mac-os-architecture/macos-ipc-inter-process-communication/)
{% endcontent-ref %}

### 일렉트론 애플리케이션 주입

특정 환경 변수로 실행된 일렉트론 애플리케이션은 프로세스 주입에 취약할 수 있습니다:

{% content-ref url="macos-electron-applications-injection.md" %}
[macos-electron-applications-injection.md](macos-electron-applications-injection.md)
{% endcontent-ref %}

### 크로미움 주입

`--load-extension` 및 `--use-fake-ui-for-media-stream` 플래그를 사용하여 **브라우저 중간자 공격**을 수행하여 키 입력, 트래픽, 쿠키를 도용하거나 페이지에 스크립트를 주입할 수 있습니다:

{% content-ref url="macos-chromium-injection.md" %}
[macos-chromium-injection.md](macos-chromium-injection.md)
{% endcontent-ref %}

### 더러운 NIB

NIB 파일은 응용 프로그램 내에서 **사용자 인터페이스 (UI) 요소와 상호 작용을 정의**합니다. 그러나 NIB 파일은 **임의의 명령을 실행**할 수 있으며 **게이트키퍼는** NIB 파일이 수정되면 이미 실행된 응용 프로그램이 다시 실행되는 것을 막지 않습니다. 따라서 임의의 프로그램이 임의의 명령을 실행하도록 사용할 수 있습니다:

{% content-ref url="macos-dirty-nib.md" %}
[macos-dirty-nib.md](macos-dirty-nib.md)
{% endcontent-ref %}

### 자바 애플리케이션 주입

일부 자바 기능(예: **`_JAVA_OPTS`** 환경 변수)을 남용하여 자바 애플리케이션이 **임의의 코드/명령을 실행**하도록 할 수 있습니다.

{% content-ref url="macos-java-apps-injection.md" %}
[macos-java-apps-injection.md](macos-java-apps-injection.md)
{% endcontent-ref %}

### .Net 애플리케이션 주입

.Net 애플리케이션에 코드를 주입하는 것은 **.Net 디버깅 기능을 남용**하여 가능합니다(런타임 강화와 같은 macOS 보호 기능으로 보호되지 않음).

{% content-ref url="macos-.net-applications-injection.md" %}
[macos-.net-applications-injection.md](macos-.net-applications-injection.md)
{% endcontent-ref %}

### 펄 주입

펄 스크립트가 임의의 코드를 실행하도록 만드는 다양한 옵션을 확인하세요:

{% content-ref url="macos-perl-applications-injection.md" %}
[macos-perl-applications-injection.md](macos-perl-applications-injection.md)
{% endcontent-ref %}

### 루비 주입

루비 환경 변수를 남용하여 임의의 스크립트가 임의의 코드를 실행하도록 할 수도 있습니다:

{% content-ref url="macos-ruby-applications-injection.md" %}
[macos-ruby-applications-injection.md](macos-ruby-applications-injection.md)
{% endcontent-ref %}

### 파이썬 주입

환경 변수 **`PYTHONINSPECT`**가 설정되어 있으면 파이썬 프로세스가 완료되면 파이썬 CLI로 이동합니다. 대화형 세션의 시작 시 파이썬 스크립트를 실행하도록 **`PYTHONSTARTUP`**을 사용할 수도 있습니다.\
그러나 **`PYTHONSTARTUP`** 스크립트는 **`PYTHONINSPECT`**가 대화형 세션을 생성할 때 실행되지 않습니다.

**`PYTHONPATH`** 및 **`PYTHONHOME`**과 같은 다른 환경 변수도 파이썬 명령이 임의의 코드를 실행하도록 하는 데 유용할 수 있습니다.

**`pyinstaller`**로 컴파일된 실행 파일은 내장된 파이썬을 사용하더라도 이러한 환경 변수를 사용하지 않습니다.

{% hint style="danger" %}
전반적으로 환경 변수를 남용하여 파이썬이 임의의 코드를 실행하도록 하는 방법을 찾지 못했습니다.\
그러나 대부분의 사람들은 **Hombrew**를 사용하여 파이썬을 설치하므로 기본 관리자 사용자를 위한 **쓰기 가능한 위치**에 파이썬을 설치합니다. 다음과 같이 이를 탈취할 수 있습니다:
```bash
mv /opt/homebrew/bin/python3 /opt/homebrew/bin/python3.old
cat > /opt/homebrew/bin/python3 <<EOF
#!/bin/bash
# Extra hijack code
/opt/homebrew/bin/python3.old "$@"
EOF
chmod +x /opt/homebrew/bin/python3
```
## 탐지

### Shield

[**Shield**](https://theevilbit.github.io/shield/) ([**Github**](https://github.com/theevilbit/Shield))은 다음과 같은 프로세스 주입 작업을 **탐지하고 차단**할 수 있는 오픈 소스 애플리케이션입니다:

* **환경 변수 사용**: **`DYLD_INSERT_LIBRARIES`**, **`CFNETWORK_LIBRARY_PATH`**, **`RAWCAMERA_BUNDLE_PATH`**, **`ELECTRON_RUN_AS_NODE`** 환경 변수의 존재를 모니터링합니다.
* **`task_for_pid`** 호출 사용: 한 프로세스가 다른 프로세스의 **작업 포트를 가져오려고 할 때** 코드를 주입할 수 있게 해줍니다.
* **Electron 앱 매개변수**: 누군가는 **`--inspect`**, **`--inspect-brk`**, **`--remote-debugging-port`** 명령행 인수를 사용하여 Electron 앱을 디버깅 모드로 시작하고 이에 코드를 주입할 수 있습니다.
* **심볼릭 링크** 또는 **하드 링크** 사용: 일반적으로 가장 흔한 남용은 **사용자 권한으로 링크를 만들고**, **보다 높은 권한** 위치를 가리키는 것입니다. 하드 링크와 심볼릭 링크 모두에 대한 감지는 매우 간단합니다. 링크를 만드는 프로세스가 대상 파일보다 **다른 권한 수준**을 가지고 있으면 **경고**를 생성합니다. 불행히도 심볼릭 링크의 경우 차단이 불가능합니다. 링크의 대상에 대한 정보를 사전에 알 수 없기 때문에 이는 Apple의 EndpointSecuriy 프레임워크의 제한입니다.

### 다른 프로세스에 의한 호출

[**이 블로그 게시물**](https://knight.sc/reverse%20engineering/2019/04/15/detecting-task-modifications.html)에서는 **`task_name_for_pid`** 함수를 사용하여 다른 **프로세스가 프로세스에 코드를 주입**하는 것에 대한 정보를 얻고 그 다른 프로세스에 대한 정보를 얻는 방법에 대해 알 수 있습니다.

해당 함수를 호출하려면 해당 프로세스를 실행하는 사용자와 **동일한 uid**이거나 **root** 여야 합니다 (그리고 이는 프로세스에 대한 정보를 반환하며 코드를 주입하는 방법은 아닙니다).

## 참고 자료

* [https://theevilbit.github.io/shield/](https://theevilbit.github.io/shield/)
* [https://medium.com/@metnew/why-electron-apps-cant-store-your-secrets-confidentially-inspect-option-a49950d6d51f](https://medium.com/@metnew/why-electron-apps-cant-store-your-secrets-confidentially-inspect-option-a49950d6d51f)
