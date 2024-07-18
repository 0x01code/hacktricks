{% hint style="success" %}
AWS 해킹 배우고 실습하기:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP 해킹 배우고 실습하기: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks 지원하기</summary>

* [**구독 요금제**](https://github.com/sponsors/carlospolop)를 확인하세요!
* 💬 [**Discord 그룹**](https://discord.gg/hRep4RUj7f)에 가입하거나 [**텔레그램 그룹**](https://t.me/peass)에 참여하거나 **트위터** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**를 팔로우**하세요.
* [**HackTricks**](https://github.com/carlospolop/hacktricks) 및 [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) 깃헙 저장소에 PR을 제출하여 해킹 트릭을 공유하세요.

</details>
{% endhint %}

# CBC - Cipher Block Chaining

CBC 모드에서는 **이전에 암호화된 블록이 IV로 사용**되어 다음 블록과 XOR 연산을 수행합니다:

![https://defuse.ca/images/cbc\_encryption.png](https://defuse.ca/images/cbc\_encryption.png)

CBC를 복호화하려면 **반대로** **작업**을 수행합니다:

![https://defuse.ca/images/cbc\_decryption.png](https://defuse.ca/images/cbc\_decryption.png)

**암호화 키**와 **IV**를 사용해야 함에 유의하세요.

# 메시지 패딩

암호화는 **고정된 크기의 블록**에서 수행되므로 **마지막 블록**을 완성하기 위해 일반적으로 **패딩**이 필요합니다.\
일반적으로 **PKCS7**이 사용되며, 패딩은 블록을 **완성**하기 위해 필요한 **바이트 수**를 **반복**하는 패딩을 생성합니다. 예를 들어, 마지막 블록이 3바이트 부족하면 패딩은 `\x03\x03\x03`이 됩니다.

**8바이트 길이의 2개 블록**에 대한 더 많은 예제를 살펴봅시다:

| 바이트 #0 | 바이트 #1 | 바이트 #2 | 바이트 #3 | 바이트 #4 | 바이트 #5 | 바이트 #6 | 바이트 #7 | 바이트 #0  | 바이트 #1  | 바이트 #2  | 바이트 #3  | 바이트 #4  | 바이트 #5  | 바이트 #6  | 바이트 #7  |
| ------- | ------- | ------- | ------- | ------- | ------- | ------- | ------- | -------- | -------- | -------- | -------- | -------- | -------- | -------- | -------- |
| P       | A       | S       | S       | W       | O       | R       | D       | 1        | 2        | 3        | 4        | 5        | 6        | **0x02** | **0x02** |
| P       | A       | S       | S       | W       | O       | R       | D       | 1        | 2        | 3        | 4        | 5        | **0x03** | **0x03** | **0x03** |
| P       | A       | S       | S       | W       | O       | R       | D       | 1        | 2        | 3        | **0x05** | **0x05** | **0x05** | **0x05** | **0x05** |
| P       | A       | S       | S       | W       | O       | R       | D       | **0x08** | **0x08** | **0x08** | **0x08** | **0x08** | **0x08** | **0x08** | **0x08** |

마지막 예제에서 **마지막 블록이 가득 찼으므로 패딩만 있는 다른 블록이 생성**되었음을 주목하세요.

# 패딩 오라클

응용 프로그램이 암호화된 데이터를 복호화하면 먼저 데이터를 복호화한 다음 패딩을 제거합니다. 패딩을 정리하는 동안 **잘못된 패딩이 감지 가능한 동작을 유발**하면 **패딩 오라클 취약점**이 있습니다. 감지 가능한 동작은 **오류**, **결과 부족**, 또는 **응답이 느림**일 수 있습니다.

이 동작을 감지하면 **암호화된 데이터를 복호화**하고 심지어 **임의의 평문을 암호화**할 수 있습니다.

## 악용 방법

[https://github.com/AonCyberLabs/PadBuster](https://github.com/AonCyberLabs/PadBuster)를 사용하여 이러한 종류의 취약점을 악용하거나 단순히 수행할 수 있습니다.
```
sudo apt-get install padbuster
```
사이트의 쿠키가 취약한지 테스트하려면 다음을 시도할 수 있습니다:
```bash
perl ./padBuster.pl http://10.10.10.10/index.php "RVJDQrwUdTRWJUVUeBKkEA==" 8 -encoding 0 -cookies "login=RVJDQrwUdTRWJUVUeBKkEA=="
```
**인코딩 0**은 **base64**가 사용된다는 것을 의미합니다 (그러나 다른 것들도 사용할 수 있습니다, 도움 메뉴를 확인하세요).

또한 이 취약점을 악용하여 새 데이터를 암호화할 수 있습니다. 예를 들어, 쿠키의 내용이 "**_**user=MyUsername**_**"인 경우, 이를 "\_user=administrator\_"로 변경하여 응용 프로그램 내에서 권한을 상승시킬 수 있습니다. 또한 `-plaintext` 매개변수를 지정하여 `paduster`를 사용하여 이를 수행할 수도 있습니다:
```bash
perl ./padBuster.pl http://10.10.10.10/index.php "RVJDQrwUdTRWJUVUeBKkEA==" 8 -encoding 0 -cookies "login=RVJDQrwUdTRWJUVUeBKkEA==" -plaintext "user=administrator"
```
만약 사이트가 취약하다면 `padbuster`는 자동으로 패딩 오류가 발생할 때 찾으려고 시도할 것이지만, **-error** 매개변수를 사용하여 오류 메시지를 지정할 수도 있습니다.
```bash
perl ./padBuster.pl http://10.10.10.10/index.php "" 8 -encoding 0 -cookies "hcon=RVJDQrwUdTRWJUVUeBKkEA==" -error "Invalid padding"
```
## 이론

**요약**하면, 모든 **다른 패딩**을 만들 수 있는 올바른 값을 추측하여 암호화된 데이터를 복호화하기 시작할 수 있습니다. 그런 다음, 패딩 오라클 공격은 **1, 2, 3 등의 패딩을 만드는 올바른 값**을 추측하여 시작하여 끝에서 시작하여 바이트를 복호화하기 시작합니다.

![](<../.gitbook/assets/image (629) (1) (1).png>)

**E0에서 E15**까지의 바이트로 구성된 **2개 블록**을 차지하는 암호화된 텍스트가 있다고 상상해보십시오.\
**마지막 블록(E8에서 E15)**을 **복호화**하기 위해 전체 블록은 "블록 암호 복호화"를 통해 **중간 바이트 I0에서 I15**를 생성합니다.\
마지막으로, 각 중간 바이트는 이전 암호화된 바이트(E0에서 E7)와 **XOR**됩니다. 그래서:

* `C15 = D(E15) ^ E7 = I15 ^ E7`
* `C14 = I14 ^ E6`
* `C13 = I13 ^ E5`
* `C12 = I12 ^ E4`
* ...

이제, `C15`가 `0x01`이 될 때까지 `E7`을 수정할 수 있습니다. 이것도 올바른 패딩이 될 것입니다. 따라서, 이 경우에는: `\x01 = I15 ^ E'7`

그래서, `E'7`을 찾으면 **I15를 계산할 수 있습니다**: `I15 = 0x01 ^ E'7`

이로써 **C15를 계산할 수 있습니다**: `C15 = E7 ^ I15 = E7 ^ \x01 ^ E'7`

**C15**를 알게 되면, 이제 **C14를 계산할 수 있습니다**. 이번에는 패딩 `\x02\x02`를 브루트 포싱하여 계산합니다.

이 BF는 이전 것만큼 복잡합니다. `E''15`의 값이 0x02인 `E''7 = \x02 ^ I15`를 계산할 수 있으므로 **`C14`가 `0x02`와 같은 `E'14`를 찾기만 하면 됩니다**.\
그런 다음, C14를 복호화하기 위해 동일한 단계를 수행합니다: **`C14 = E6 ^ I14 = E6 ^ \x02 ^ E''6`**

**전체 암호화된 텍스트를 복호화할 때까지 이 체인을 따릅니다.**

## 취약점 탐지

계정을 등록하고 이 계정으로 로그인합니다.\
**여러 번 로그인**하고 항상 **동일한 쿠키**를 받으면 응용 프로그램에 **문제가 있을 수 있습니다**. 로그인할 때마다 **반드시 고유한 쿠키**가 반환되어야 합니다. 쿠키가 **항상** **동일하면**, 아마도 항상 유효하고 **무효화할 수 있는 방법이 없을 것입니다**.

이제, **쿠키를 수정**하려고 하면 응용 프로그램에서 **오류**가 발생하는 것을 볼 수 있습니다.\
그러나 패딩을 브루트 포싱하여 (예: 패드버스터 사용) 다른 사용자에 대해 유효한 다른 쿠키를 얻을 수 있습니다. 이 시나리오는 패드버스터에 취약할 가능성이 높습니다.

## 참고 자료

* [https://en.wikipedia.org/wiki/Block\_cipher\_mode\_of\_operation](https://en.wikipedia.org/wiki/Block\_cipher\_mode\_of\_operation)
