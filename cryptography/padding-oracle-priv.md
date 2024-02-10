<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert)</strong>를 통해 AWS 해킹을 처음부터 전문가까지 배워보세요<strong>!</strong></summary>

HackTricks를 지원하는 다른 방법:

* **회사를 HackTricks에서 광고하거나 HackTricks를 PDF로 다운로드**하려면 [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)를 확인하세요!
* [**공식 PEASS & HackTricks 스웨그**](https://peass.creator-spring.com)를 얻으세요.
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)를 발견하세요. 독점적인 [**NFT**](https://opensea.io/collection/the-peass-family) 컬렉션입니다.
* 💬 [**Discord 그룹**](https://discord.gg/hRep4RUj7f) 또는 [**텔레그램 그룹**](https://t.me/peass)에 **참여**하거나 **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)를 **팔로우**하세요.
* **Hacking 트릭을 공유하려면** [**HackTricks**](https://github.com/carlospolop/hacktricks) 및 [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github 저장소에 PR을 제출하세요.

</details>


# CBC - Cipher Block Chaining

CBC 모드에서는 **이전 암호화된 블록이 IV로 사용**되어 다음 블록과 XOR 연산을 수행합니다:

![https://defuse.ca/images/cbc\_encryption.png](https://defuse.ca/images/cbc\_encryption.png)

CBC를 복호화하려면 **반대로** **연산**을 수행합니다:

![https://defuse.ca/images/cbc\_decryption.png](https://defuse.ca/images/cbc\_decryption.png)

암호화에는 **암호화 키**와 **IV**를 사용해야 함에 유의하세요.

# 메시지 패딩

암호화는 **고정된 크기의 블록**으로 수행되므로, **마지막 블록**의 길이를 완성하기 위해 일반적으로 **패딩**이 필요합니다.\
일반적으로는 **PKCS7**이 사용되며, 패딩은 블록을 **완성**하기 위해 필요한 **바이트 수**를 **반복**하여 생성됩니다. 예를 들어, 마지막 블록이 3바이트가 부족한 경우 패딩은 `\x03\x03\x03`이 됩니다.

**8바이트 길이의 2개 블록**을 가진 더 많은 예제를 살펴보겠습니다:

| byte #0 | byte #1 | byte #2 | byte #3 | byte #4 | byte #5 | byte #6 | byte #7 | byte #0  | byte #1  | byte #2  | byte #3  | byte #4  | byte #5  | byte #6  | byte #7  |
| ------- | ------- | ------- | ------- | ------- | ------- | ------- | ------- | -------- | -------- | -------- | -------- | -------- | -------- | -------- | -------- |
| P       | A       | S       | S       | W       | O       | R       | D       | 1        | 2        | 3        | 4        | 5        | 6        | **0x02** | **0x02** |
| P       | A       | S       | S       | W       | O       | R       | D       | 1        | 2        | 3        | 4        | 5        | **0x03** | **0x03** | **0x03** |
| P       | A       | S       | S       | W       | O       | R       | D       | 1        | 2        | 3        | **0x05** | **0x05** | **0x05** | **0x05** | **0x05** |
| P       | A       | S       | S       | W       | O       | R       | D       | **0x08** | **0x08** | **0x08** | **0x08** | **0x08** | **0x08** | **0x08** | **0x08** |

마지막 예제에서는 **마지막 블록이 가득 차 있어 패딩만 있는 블록이 추가로 생성**되었음을 알 수 있습니다.

# 패딩 오라클

응용 프로그램이 암호화된 데이터를 복호화할 때, 먼저 데이터를 복호화한 다음 패딩을 제거합니다. 패딩을 정리하는 동안, **잘못된 패딩이 감지 가능한 동작을 트리거**하면 패딩 오라클 취약점이 있습니다. 감지 가능한 동작은 **오류**, **결과 부족** 또는 **응답 속도가 느려짐**일 수 있습니다.

이러한 동작을 감지하면, 암호화된 데이터를 **복호화**하고 심지어 **임의의 평문을 암호화**할 수 있습니다.

## 악용 방법

[https://github.com/AonCyberLabs/PadBuster](https://github.com/AonCyberLabs/PadBuster)를 사용하여 이러한 유형의 취약점을 악용하거나, 단순히 다음을 수행할 수 있습니다.
```
sudo apt-get install padbuster
```
사이트의 쿠키가 취약한지 테스트하기 위해 다음을 시도할 수 있습니다:
```bash
perl ./padBuster.pl http://10.10.10.10/index.php "RVJDQrwUdTRWJUVUeBKkEA==" 8 -encoding 0 -cookies "login=RVJDQrwUdTRWJUVUeBKkEA=="
```
**인코딩 0**은 **base64**가 사용된다는 것을 의미합니다 (하지만 다른 것들도 사용 가능하니 도움말 메뉴를 확인하세요).

또한 이 취약점을 **새로운 데이터를 암호화하기 위해 악용**할 수도 있습니다. 예를 들어, 쿠키의 내용이 "\_user=MyUsername\_"인 경우, 이를 "\_user=administrator\_"로 변경하여 응용 프로그램 내에서 권한을 상승시킬 수 있습니다. 또한 `padbuster`를 사용하여 `-plaintext` 매개변수를 지정하여 이 작업을 수행할 수도 있습니다:
```bash
perl ./padBuster.pl http://10.10.10.10/index.php "RVJDQrwUdTRWJUVUeBKkEA==" 8 -encoding 0 -cookies "login=RVJDQrwUdTRWJUVUeBKkEA==" -plaintext "user=administrator"
```
만약 사이트가 취약하다면 `padbuster`는 자동으로 패딩 오류가 발생하는 시점을 찾으려고 시도할 것입니다. 그러나 **-error** 매개변수를 사용하여 오류 메시지를 지정할 수도 있습니다.
```bash
perl ./padBuster.pl http://10.10.10.10/index.php "" 8 -encoding 0 -cookies "hcon=RVJDQrwUdTRWJUVUeBKkEA==" -error "Invalid padding"
```
## 이론

요약하자면, 모든 **다른 패딩**을 생성하는 데 사용될 수 있는 올바른 값을 추측하여 암호화된 데이터의 복호화를 시작할 수 있습니다. 그런 다음 패딩 오라클 공격은 1, 2, 3 등의 패딩을 생성하는 올바른 값을 추측하여 시작하여 끝에서 시작하여 바이트를 복호화합니다.

![](<../.gitbook/assets/image (629) (1) (1).png>)

E0에서 E15까지의 바이트로 구성된 **2개의 블록**으로 이루어진 암호화된 텍스트가 있다고 상상해보십시오.\
마지막 블록(E8에서 E15)을 **복호화**하기 위해 전체 블록은 "블록 암호 복호화"를 통해 중간 바이트 I0에서 I15을 생성합니다.\
마지막으로, 각 중간 바이트는 이전의 암호화된 바이트(E0에서 E7)와 **XOR**됩니다. 그래서:

* `C15 = D(E15) ^ E7 = I15 ^ E7`
* `C14 = I14 ^ E6`
* `C13 = I13 ^ E5`
* `C12 = I12 ^ E4`
* ...

이제 `C15`이 `0x01`이 될 때까지 `E7`을 수정할 수 있습니다. 이는 올바른 패딩이기도 합니다. 따라서 이 경우에는: `\x01 = I15 ^ E'7`

따라서, `E'7`을 찾으면 **I15을 계산**할 수 있습니다: `I15 = 0x01 ^ E'7`

이로써 **C15을 계산**할 수 있습니다: `C15 = E7 ^ I15 = E7 ^ \x01 ^ E'7`

**C15**을 알게 되면, 이제 **C14를 계산**할 수 있지만, 이번에는 패딩 `\x02\x02`를 무차별 대입(brute-force)합니다.

이 무차별 대입은 이전과 같이 복잡합니다. 0x02인 **`E''15`**를 계산할 수 있습니다: `E''7 = \x02 ^ I15` 그래서 **`C14`**가 **`0x02`**와 같은 **`E'14`**를 찾기만 하면 됩니다.\
그런 다음, C14를 복호화하기 위해 동일한 단계를 수행합니다: **`C14 = E6 ^ I14 = E6 ^ \x02 ^ E''6`**

**전체 암호화된 텍스트를 복호화**하기 위해 이러한 과정을 따르십시오.

## 취약점 탐지

계정을 등록하고이 계정으로 로그인하십시오.\
여러 번 로그인하고 항상 **동일한 쿠키**를 받으면 응용 프로그램에 문제가 있을 수 있습니다. 쿠키는 로그인할 때마다 **고유해야**합니다. 쿠키가 **항상** **동일**하면 항상 유효하고 **무효화할 수 있는 방법이 없을 것**입니다.

이제 쿠키를 **수정**하려고하면 응용 프로그램에서 **오류**가 발생하는 것을 볼 수 있습니다.\
그러나 패딩을 무차별 대입(padbuster를 사용하여 예를 들면)하면 다른 사용자에 대한 유효한 다른 쿠키를 얻을 수 있습니다. 이 시나리오는 padbuster에 취약할 가능성이 매우 높습니다.

## 참고 자료

* [https://en.wikipedia.org/wiki/Block\_cipher\_mode\_of\_operation](https://en.wikipedia.org/wiki/Block\_cipher\_mode\_of\_operation)


<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert)</strong>를 통해 AWS 해킹을 처음부터 전문가까지 배워보세요<strong>!</strong></summary>

HackTricks를 지원하는 다른 방법:

* 회사를 HackTricks에서 **광고하거나 HackTricks를 PDF로 다운로드**하려면 [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)를 확인하세요!
* [**공식 PEASS & HackTricks 스웨그**](https://peass.creator-spring.com)를 얻으세요.
* 독점적인 [**NFTs**](https://opensea.io/collection/the-peass-family)인 [**The PEASS Family**](https://opensea.io/collection/the-peass-family) 컬렉션을 발견하세요.
* 💬 [**Discord 그룹**](https://discord.gg/hRep4RUj7f) 또는 [**텔레그램 그룹**](https://t.me/peass)에 **참여**하거나 **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**를** 팔로우하세요.
* **HackTricks**와 **HackTricks Cloud** github 저장소에 PR을 제출하여 **자신의 해킹 기법을 공유**하세요.

</details>
