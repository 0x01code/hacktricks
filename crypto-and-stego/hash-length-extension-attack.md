# Atak na długość skrótu

<details>

<summary><strong>Nauka hakowania AWS od zera do bohatera z</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Inne sposoby wsparcia HackTricks:

* Jeśli chcesz zobaczyć swoją **firmę reklamowaną w HackTricks** lub **pobrać HackTricks w formacie PDF**, sprawdź [**PLANY SUBSKRYPCYJNE**](https://github.com/sponsors/carlospolop)!
* Zdobądź [**oficjalne gadżety PEASS & HackTricks**](https://peass.creator-spring.com)
* Odkryj [**Rodzinę PEASS**](https://opensea.io/collection/the-peass-family), naszą kolekcję ekskluzywnych [**NFT**](https://opensea.io/collection/the-peass-family)
* **Dołącz do** 💬 [**grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegramowej**](https://t.me/peass) lub **śledź** nas na **Twitterze** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podziel się swoimi sztuczkami hakowania, przesyłając PR-y do** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repozytoriów na GitHubie.

</details>

#### [WhiteIntel](https://whiteintel.io)

<figure><img src="../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) to wyszukiwarka zasilana przez **dark web**, która oferuje **darmowe** funkcje do sprawdzania, czy firma lub jej klienci zostali **skompromitowani** przez **złośliwe oprogramowanie kradnące informacje**.

Ich głównym celem WhiteIntel jest zwalczanie przejęć kont i ataków ransomware wynikających z złośliwego oprogramowania kradnącego informacje.

Możesz sprawdzić ich stronę internetową i wypróbować ich silnik **za darmo** pod adresem:

{% embed url="https://whiteintel.io" %}

***

## Podsumowanie ataku

Wyobraź sobie serwer, który **podpisuje** pewne **dane**, **dodając** do nich **tajny klucz** do pewnych znanych danych tekstowych i następnie haszując te dane. Jeśli znasz:

* **Długość tajnego klucza** (można to również przeprowadzić metodą brutalnej siły w określonym zakresie długości)
* **Dane tekstowe**
* **Algorytm (podatny na ten atak)**
* **Padding jest znany**
* Zazwyczaj używany jest domyślny, więc jeśli spełnione są pozostałe 3 wymagania, to również jest znany
* Padding różni się w zależności od długości tajnego klucza+danych, dlatego potrzebna jest długość tajnego klucza

W takim przypadku **atakujący** może **dodać** **dane** i **wygenerować** poprawny **podpis** dla **poprzednich danych + dodanych danych**.

### Jak?

W podatnych algorytmach haszowanie odbywa się poprzez **najpierw haszowanie bloku danych**, a następnie, **z** **wcześniej** utworzonego **skrótu** (stanu), **dodają następny blok danych** i **haszują go**.

Wyobraź sobie, że tajny klucz to "tajne" a dane to "dane", MD5 z "tajnedane" to 6036708eba0d11f6ef52ad44e8b74d5b.\
Jeśli atakujący chce dodać ciąg "dodaj" może:

* Wygenerować MD5 z 64 "A"
* Zmienić stan wcześniej zainicjowanego skrótu na 6036708eba0d11f6ef52ad44e8b74d5b
* Dodać ciąg "dodaj"
* Zakończyć haszowanie, a wynikowy skrót będzie **poprawny dla "tajne" + "dane" + "padding" + "dodaj"**

### **Narzędzie**

{% embed url="https://github.com/iagox86/hash_extender" %}

### Referencje

Możesz znaleźć ten atak dobrze wyjaśniony na stronie [https://blog.skullsecurity.org/2012/everything-you-need-to-know-about-hash-length-extension-attacks](https://blog.skullsecurity.org/2012/everything-you-need-to-know-about-hash-length-extension-attacks)

#### [WhiteIntel](https://whiteintel.io)

<figure><img src="../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) to wyszukiwarka zasilana przez **dark web**, która oferuje **darmowe** funkcje do sprawdzania, czy firma lub jej klienci zostali **skompromitowani** przez **złośliwe oprogramowanie kradnące informacje**.

Ich głównym celem WhiteIntel jest zwalczanie przejęć kont i ataków ransomware wynikających z złośliwego oprogramowania kradnącego informacje.

Możesz sprawdzić ich stronę internetową i wypróbować ich silnik **za darmo** pod adresem:

{% embed url="https://whiteintel.io" %}

<details>

<summary><strong>Nauka hakowania AWS od zera do bohatera z</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Inne sposoby wsparcia HackTricks:

* Jeśli chcesz zobaczyć swoją **firmę reklamowaną w HackTricks** lub **pobrać HackTricks w formacie PDF**, sprawdź [**PLANY SUBSKRYPCYJNE**](https://github.com/sponsors/carlospolop)!
* Zdobądź [**oficjalne gadżety PEASS & HackTricks**](https://peass.creator-spring.com)
* Odkryj [**Rodzinę PEASS**](https://opensea.io/collection/the-peass-family), naszą kolekcję ekskluzywnych [**NFT**](https://opensea.io/collection/the-peass-family)
* **Dołącz do** 💬 [**grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegramowej**](https://t.me/peass) lub **śledź** nas na **Twitterze** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podziel się swoimi sztuczkami hakowania, przesyłając PR-y do** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repozytoriów na GitHubie.

</details>
