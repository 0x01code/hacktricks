<details>

<summary><strong>Naucz się hakować AWS od zera do bohatera z</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Inne sposoby wsparcia HackTricks:

* Jeśli chcesz zobaczyć swoją **firmę reklamowaną w HackTricks** lub **pobrać HackTricks w formacie PDF**, sprawdź [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Zdobądź [**oficjalne gadżety PEASS & HackTricks**](https://peass.creator-spring.com)
* Odkryj [**Rodzinę PEASS**](https://opensea.io/collection/the-peass-family), naszą kolekcję ekskluzywnych [**NFT**](https://opensea.io/collection/the-peass-family)
* **Dołącz do** 💬 [**grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegramowej**](https://t.me/peass) lub **śledź** nas na **Twitterze** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Podziel się swoimi sztuczkami hakerskimi, przesyłając PR-y do** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repozytoriów github.

</details>


# CBC - Cipher Block Chaining

W trybie CBC **poprzedni zaszyfrowany blok jest używany jako IV** do operacji XOR z następnym blokiem:

![https://defuse.ca/images/cbc\_encryption.png](https://defuse.ca/images/cbc\_encryption.png)

Aby odszyfrować CBC, wykonuje się **odwrotne** **operacje**:

![https://defuse.ca/images/cbc\_decryption.png](https://defuse.ca/images/cbc\_decryption.png)

Zauważ, że potrzebne jest użycie **klucza szyfrowania** i **IV**.

# Padding wiadomości

Ponieważ szyfrowanie jest wykonywane w **blokach o stałej wielkości**, zazwyczaj potrzebne jest **uzupełnienie** w **ostatnim** **bloku**, aby uzupełnić jego długość.\
Zazwyczaj używany jest **PKCS7**, który generuje uzupełnienie **powtarzając** **liczbę** **bajtów**, **które są potrzebne**, aby **uzupełnić** blok. Na przykład, jeśli ostatni blok brakuje 3 bajtów, uzupełnienie będzie `\x03\x03\x03`.

Przyjrzyjmy się więcej przykładom z **2 blokami o długości 8 bajtów**:

| bajt #0 | bajt #1 | bajt #2 | bajt #3 | bajt #4 | bajt #5 | bajt #6 | bajt #7 | bajt #0  | bajt #1  | bajt #2  | bajt #3  | bajt #4  | bajt #5  | bajt #6  | bajt #7  |
| ------- | ------- | ------- | ------- | ------- | ------- | ------- | ------- | -------- | -------- | -------- | -------- | -------- | -------- | -------- | -------- |
| P       | A       | S       | S       | W       | O       | R       | D       | 1        | 2        | 3        | 4        | 5        | 6        | **0x02** | **0x02** |
| P       | A       | S       | S       | W       | O       | R       | D       | 1        | 2        | 3        | 4        | 5        | **0x03** | **0x03** | **0x03** |
| P       | A       | S       | S       | W       | O       | R       | D       | 1        | 2        | 3        | **0x05** | **0x05** | **0x05** | **0x05** | **0x05** |
| P       | A       | S       | S       | W       | O       | R       | D       | **0x08** | **0x08** | **0x08** | **0x08** | **0x08** | **0x08** | **0x08** | **0x08** |

Zauważ, że w ostatnim przykładzie **ostatni blok był pełny, więc wygenerowano kolejny tylko z uzupełnieniem**.

# Padding Oracle

Gdy aplikacja odszyfrowuje dane, najpierw odszyfrowuje dane, a następnie usuwa uzupełnienie. Podczas usuwania uzupełnienia, jeśli **nieprawidłowe uzupełnienie wywołuje wykrywalne zachowanie**, mamy do czynienia z **podatnością na padding oracle**. Wykrywalne zachowanie może być **błędem**, **brakiem wyników** lub **wolniejszą odpowiedzią**.

Jeśli wykryjesz takie zachowanie, możesz **odszyfrować zaszyfrowane dane** i nawet **zaszyfrować dowolny tekst jawnie**.

## Jak wykorzystać

Możesz użyć [https://github.com/AonCyberLabs/PadBuster](https://github.com/AonCyberLabs/PadBuster), aby wykorzystać tego rodzaju podatność lub po prostu wykonać
```
sudo apt-get install padbuster
```
Aby sprawdzić, czy ciasteczko strony jest podatne, można spróbować:
```bash
perl ./padBuster.pl http://10.10.10.10/index.php "RVJDQrwUdTRWJUVUeBKkEA==" 8 -encoding 0 -cookies "login=RVJDQrwUdTRWJUVUeBKkEA=="
```
**Kodowanie 0** oznacza, że używane jest **base64** (ale dostępne są inne, sprawdź menu pomocy).

Możesz również **wykorzystać tę podatność do szyfrowania nowych danych. Na przykład, wyobraź sobie, że zawartość ciasteczka to "**_**user=MyUsername**_**", wtedy możesz ją zmienić na "\_user=administrator\_" i podnieść uprawnienia wewnątrz aplikacji. Możesz to również zrobić, używając `paduster` i określając parametr -plaintext**.
```bash
perl ./padBuster.pl http://10.10.10.10/index.php "RVJDQrwUdTRWJUVUeBKkEA==" 8 -encoding 0 -cookies "login=RVJDQrwUdTRWJUVUeBKkEA==" -plaintext "user=administrator"
```
Jeśli strona jest podatna, `padbuster` automatycznie spróbuje znaleźć moment wystąpienia błędu w dopełnianiu, ale można również wskazać komunikat o błędzie za pomocą parametru **-error**.
```bash
perl ./padBuster.pl http://10.10.10.10/index.php "" 8 -encoding 0 -cookies "hcon=RVJDQrwUdTRWJUVUeBKkEA==" -error "Invalid padding"
```
## Teoria

Podsumowując, można rozpocząć deszyfrowanie zaszyfrowanych danych, zgadując poprawne wartości, które mogą być użyte do utworzenia wszystkich różnych wypełnień. Następnie atak na orakulum wypełnienia rozpocznie deszyfrowanie bajtów od końca do początku, zgadując, jaka będzie poprawna wartość, która tworzy wypełnienie o wartości 1, 2, 3, itd.

![](<../.gitbook/assets/image (629) (1) (1).png>)

Wyobraź sobie, że masz zaszyfrowany tekst, który zajmuje 2 bloki utworzone przez bajty od E0 do E15. Aby odszyfrować ostatni blok (od E8 do E15), cały blok przechodzi przez "deszyfrowanie szyfru blokowego", generując pośrednie bajty I0 do I15. Ostatecznie każdy pośredni bajt jest XORowany z poprzednimi zaszyfrowanymi bajtami (E0 do E7). Więc:

* `C15 = D(E15) ^ E7 = I15 ^ E7`
* `C14 = I14 ^ E6`
* `C13 = I13 ^ E5`
* `C12 = I12 ^ E4`
* ...

Teraz można **zmodyfikować `E7` aż `C15` będzie `0x01`**, co również będzie poprawnym wypełnieniem. Więc w tym przypadku: `\x01 = I15 ^ E'7`

Znalezienie E'7 pozwala **obliczyć I15**: `I15 = 0x01 ^ E'7`

Co pozwala nam **obliczyć C15**: `C15 = E7 ^ I15 = E7 ^ \x01 ^ E'7`

Znając **C15**, teraz można **obliczyć C14**, ale tym razem brute-force'ując wypełnienie `\x02\x02`.

Ten BF jest tak samo skomplikowany jak poprzedni, ponieważ można obliczyć `E''15`, którego wartość wynosi 0x02: `E''7 = \x02 ^ I15`, więc wystarczy znaleźć **`E'14`**, który generuje **`C14` równy `0x02`**.\
Następnie wykonaj te same kroki, aby odszyfrować C14: **`C14 = E6 ^ I14 = E6 ^ \x02 ^ E''6`**

**Postępuj zgodnie z tą sekwencją, aż odszyfrujesz cały zaszyfrowany tekst.**

## Wykrywanie podatności

Zarejestruj konto i zaloguj się na to konto.\
Jeśli **zalogujesz się wiele razy** i za każdym razem otrzymasz **ten sam ciasteczko**, prawdopodobnie **coś jest nie tak** w aplikacji. Ciasteczko wysyłane z powrotem powinno być unikalne za każdym razem, gdy się logujesz. Jeśli ciasteczko jest **zawsze** takie **same**, prawdopodobnie zawsze będzie ważne i nie będzie możliwości jego unieważnienia.

Teraz, jeśli spróbujesz **zmodyfikować** ciasteczko, zobaczysz, że otrzymujesz **błąd** od aplikacji.\
Ale jeśli brute-force'ujesz wypełnienie (używając np. padbuster), uda ci się uzyskać inne ciasteczko ważne dla innego użytkownika. Ten scenariusz jest bardzo prawdopodobnie podatny na padbuster.

## Odwołania

* [https://en.wikipedia.org/wiki/Block\_cipher\_mode\_of\_operation](https://en.wikipedia.org/wiki/Block\_cipher\_mode\_of\_operation)


<details>

<summary><strong>Naucz się hakować AWS od zera do bohatera z</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Inne sposoby wsparcia HackTricks:

* Jeśli chcesz zobaczyć swoją **firmę reklamowaną w HackTricks** lub **pobrać HackTricks w formacie PDF**, sprawdź [**PLAN SUBSKRYPCJI**](https://github.com/sponsors/carlospolop)!
* Zdobądź [**oficjalne gadżety PEASS & HackTricks**](https://peass.creator-spring.com)
* Odkryj [**Rodzinę PEASS**](https://opensea.io/collection/the-peass-family), naszą kolekcję ekskluzywnych [**NFT**](https://opensea.io/collection/the-peass-family)
* **Dołącz do** 💬 [**grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegramowej**](https://t.me/peass) lub **śledź** nas na **Twitterze** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Podziel się swoimi sztuczkami hakerskimi, przesyłając PR-y do** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
