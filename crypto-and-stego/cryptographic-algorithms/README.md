# 암호화/압축 알고리즘

## 암호화/압축 알고리즘

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert)</strong>를 통해 AWS 해킹을 처음부터 전문가까지 배워보세요<strong>!</strong></summary>

HackTricks를 지원하는 다른 방법:

* **회사를 HackTricks에서 광고하거나 HackTricks를 PDF로 다운로드**하려면 [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)를 확인하세요!
* [**공식 PEASS & HackTricks 스웨그**](https://peass.creator-spring.com)를 얻으세요.
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)를 발견하세요. 독점적인 [**NFTs**](https://opensea.io/collection/the-peass-family) 컬렉션입니다.
* 💬 [**Discord 그룹**](https://discord.gg/hRep4RUj7f) 또는 [**텔레그램 그룹**](https://t.me/peass)에 **참여**하거나 **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**를** **팔로우**하세요.
* **Hacking 트릭을 공유하려면** [**HackTricks**](https://github.com/carlospolop/hacktricks) 및 [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github 저장소에 PR을 제출하세요.

</details>

## 알고리즘 식별

만약 코드에서 **시프트 연산, XOR 및 여러 산술 연산을 사용하는 경우**, 그것은 **암호화 알고리즘의 구현**일 가능성이 높습니다. 여기에서는 **각 단계를 반전시키지 않고 사용된 알고리즘을 식별하는 방법**을 소개합니다.

### API 함수

**CryptDeriveKey**

이 함수가 사용된 경우, 두 번째 매개변수의 값을 확인하여 **사용된 알고리즘을 찾을 수** 있습니다:

![](<../../.gitbook/assets/image (375) (1) (1) (1) (1).png>)

가능한 알고리즘과 해당 값에 대한 테이블은 여기에서 확인할 수 있습니다: [https://docs.microsoft.com/en-us/windows/win32/seccrypto/alg-id](https://docs.microsoft.com/en-us/windows/win32/seccrypto/alg-id)

**RtlCompressBuffer/RtlDecompressBuffer**

주어진 데이터 버퍼를 압축하거나 압축 해제합니다.

**CryptAcquireContext**

[문서](https://learn.microsoft.com/en-us/windows/win32/api/wincrypt/nf-wincrypt-cryptacquirecontexta)에서: **CryptAcquireContext** 함수는 특정 암호화 서비스 제공자(CSP) 내의 특정 키 컨테이너에 대한 핸들을 획득하는 데 사용됩니다. **이 반환된 핸들은 선택한 CSP를 사용하는 CryptoAPI 함수 호출에 사용됩니다.**

**CryptCreateHash**

데이터 스트림의 해싱을 시작합니다. 이 함수가 사용된 경우, 두 번째 매개변수의 값을 확인하여 **사용된 알고리즘을 찾을 수** 있습니다:

![](<../../.gitbook/assets/image (376).png>)

가능한 알고리즘과 해당 값에 대한 테이블은 여기에서 확인할 수 있습니다: [https://docs.microsoft.com/en-us/windows/win32/seccrypto/alg-id](https://docs.microsoft.com/en-us/windows/win32/seccrypto/alg-id)

### 코드 상수

때로는 특정하고 고유한 값을 사용해야 하는 알고리즘을 식별하는 것이 매우 쉽습니다.

![](<../../.gitbook/assets/image (370).png>)

첫 번째 상수를 Google에서 검색하면 다음과 같은 결과가 나옵니다:

![](<../../.gitbook/assets/image (371).png>)

따라서, 디컴파일된 함수가 **sha256 계산기**임을 가정할 수 있습니다. 다른 상수 중 하나를 검색하면 (아마도) 동일한 결과를 얻을 수 있습니다.

### 데이터 정보

코드에 유의미한 상수가 없는 경우, 코드가 **.data 섹션에서 정보를 로드**할 수 있습니다.\
해당 데이터에 액세스하여 **첫 번째 dword를 그룹화**하고 앞에서 한 것과 같이 Google에서 검색할 수 있습니다:

![](<../../.gitbook/assets/image (372).png>)

이 경우, **0xA56363C6**을 검색하면 **AES 알고리즘의 테이블**과 관련된 것임을 알 수 있습니다.

## RC4 **(대칭 암호)**

### 특징

RC4는 3가지 주요 부분으로 구성됩니다:

* **초기화 단계/**: 0x00에서 0xFF(총 256바이트, 0x100)까지의 값으로 구성된 **값 테이블** 또는 **치환 상자**(SBox)를 생성합니다.
* **혼란 단계**: 이전에 생성된 테이블을 반복하여 (다시 0x100번 반복) 각 값을 **반 랜덤** 바이트로 수정합니다. 이 반 랜덤 바이트를 생성하기 위해 RC4 **키**가 사용됩니다. RC4 **키**는 **1바이트에서 256바이트**까지 가능하지만, 일반적으로 5바이트 이상을 권장합니다. 일반적으로 RC4 키는 16바이트입니다.
* **XOR 단계**: 마지막으로, 평문 또는 암호문은 이전에 생성된 값들과 **XOR 연산**됩니다. 암호화 및 복호화를 위해 동일한 함수가 사용됩니다. 이를 위해 생성된 256바이트를 필요한 만큼 반복하여 수행합니다. 이는 일반적으로 디컴파일된 코드에서 **%256 (mod 256)**로 인식됩니다.

{% hint style="info" %}
**디어셈블리/디컴파일된 코드에서 RC4를 식별하려면 키를 사용한 크기가 0x100인 2개의 루프와 256개의 값으로 생성된 입력 데이터의 XOR를 확인할 수 있습니다. 이 때, 256바이트로 생성된 2개의 루프에서 %256 (mod 256)를 사용하는 것을 확인할 수 있습니다.**
{% endhint %}

### **초기화 단계/치환 상자:** (256이라는 숫자가 카운터로 사용되고 256개 문자의 각 위치에 0이 쓰여져 있는 것에 유의하세요)

![](<../../.gitbook/assets/image (377).png>)

### **혼란 단계:**

![](<../../.gitbook/assets/image (378).png>)

### **XOR 단계:**

![](<../../.gitbook/assets/image (379).png>)

## **AES (대칭 암호)**

### **특징**

* **치환 상자 및 룩업 테이블**의 사용
* 특정 룩업 테이블 값(상수)의 사용으로 **AES를 구별**할 수 있습니다. _상수는 이진 파일에 **저장**되거나 **동적으로 생성**될 수 있습니다._
* **암호화 키**는 16으로 **나눌 수 있어야** 하며(일반적으로 32B), 일반적으로 16B의 **IV**가 사용됩니다.

### SBox 상수

![](<../../.gitbook/assets/image (380).png>)

## Serpent **(대칭 암호)**

### 특징

* 악성 코드에서 사용되는 것은 드물지만 예시(Ursnif)가 있습니다.
* 알고리즘이 Serpent인지 아닌지를 판단하는 것은 **길이**(
## RSA **(비대칭 암호화)**

### 특징

* 대칭 알고리즘보다 복잡함
* 상수가 없음! (사용자 정의 구현이 어려움)
* RSA는 상수에 의존하므로 KANAL (암호 분석기)은 RSA에 대한 힌트를 제공하지 않음.

### 비교를 통한 식별

![](<../../.gitbook/assets/image (383).png>)

* 11번 줄 (왼쪽)에는 `+7) >> 3`이 있으며, 35번 줄 (오른쪽)에는 `+7) / 8`과 동일함.
* 12번 줄 (왼쪽)에서는 `modulus_len < 0x040`을 확인하고, 36번 줄 (오른쪽)에서는 `inputLen+11 > modulusLen`을 확인함.

## MD5 & SHA (해시)

### 특징

* Init, Update, Final 3개의 함수
* 유사한 초기화 함수

### 식별

**Init**

상수를 확인하여 두 알고리즘을 식별할 수 있습니다. MD5에는 sha\_init에 없는 1개의 상수가 있다는 점에 유의하세요:

![](<../../.gitbook/assets/image (385).png>)

**MD5 Transform**

더 많은 상수의 사용에 주목하세요.

![](<../../.gitbook/assets/image (253) (1) (1) (1).png>)

## CRC (해시)

* 데이터의 우연한 변경을 찾기 위한 함수로서 더 작고 효율적임
* 상수를 사용하는 조회 테이블을 사용함

### 식별

**조회 테이블 상수**를 확인하세요:

![](<../../.gitbook/assets/image (387).png>)

CRC 해시 알고리즘은 다음과 같습니다:

![](<../../.gitbook/assets/image (386).png>)

## APLib (압축)

### 특징

* 식별 가능한 상수가 없음
* 파이썬으로 알고리즘을 작성하고 온라인에서 유사한 것을 검색해볼 수 있음

### 식별

그래프가 상당히 큽니다:

![](<../../.gitbook/assets/image (207) (2) (1).png>)

**식별하기 위해 3개의 비교**를 확인하세요:

![](<../../.gitbook/assets/image (384).png>)

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert)</strong>를 통해 제로에서 영웅까지 AWS 해킹을 배워보세요<strong>!</strong></summary>

HackTricks를 지원하는 다른 방법:

* **회사를 HackTricks에서 광고하거나 HackTricks를 PDF로 다운로드**하려면 [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)를 확인하세요!
* [**공식 PEASS & HackTricks 상품**](https://peass.creator-spring.com)을 구매하세요.
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)를 발견하세요. 독점적인 [**NFT**](https://opensea.io/collection/the-peass-family) 컬렉션입니다.
* 💬 [**Discord 그룹**](https://discord.gg/hRep4RUj7f) 또는 [**텔레그램 그룹**](https://t.me/peass)에 **참여**하거나 **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)을 **팔로우**하세요.
* **HackTricks**와 **HackTricks Cloud** github 저장소에 PR을 제출하여 **당신의 해킹 기교를 공유**하세요.

</details>
