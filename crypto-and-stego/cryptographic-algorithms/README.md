# Algorytmy kryptograficzne/kompresji

## Algorytmy kryptograficzne/kompresji

<details>

<summary><strong>Nauka hakowania AWS od zera do bohatera z</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Inne sposoby wsparcia HackTricks:

* Jeśli chcesz zobaczyć swoją **firmę reklamowaną w HackTricks** lub **pobrać HackTricks w formacie PDF**, sprawdź [**PLANY SUBSKRYPCYJNE**](https://github.com/sponsors/carlospolop)!
* Zdobądź [**oficjalne gadżety PEASS & HackTricks**](https://peass.creator-spring.com)
* Odkryj [**Rodzinę PEASS**](https://opensea.io/collection/the-peass-family), naszą kolekcję ekskluzywnych [**NFT**](https://opensea.io/collection/the-peass-family)
* **Dołącz do** 💬 [**grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegramowej**](https://t.me/peass) lub **śledź** nas na **Twitterze** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podziel się swoimi sztuczkami hakowania, przesyłając PR-y do** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) na GitHubie.

</details>

## Identyfikacja algorytmów

Jeśli natrafisz na kod **używający przesunięć bitowych w prawo i w lewo, operacji XOR oraz kilku operacji arytmetycznych**, istnieje wysokie prawdopodobieństwo, że jest to implementacja **algorytmu kryptograficznego**. Tutaj zostaną przedstawione sposoby **identyfikacji użytego algorytmu bez konieczności odwracania każdego kroku**.

### Funkcje API

**CryptDeriveKey**

Jeśli jest używana ta funkcja, można znaleźć, który **algorytm jest używany**, sprawdzając wartość drugiego parametru:

![](<../../.gitbook/assets/image (156).png>)

Sprawdź tutaj tabelę możliwych algorytmów i ich przypisane wartości: [https://docs.microsoft.com/en-us/windows/win32/seccrypto/alg-id](https://docs.microsoft.com/en-us/windows/win32/seccrypto/alg-id)

**RtlCompressBuffer/RtlDecompressBuffer**

Kompresuje i dekompresuje dany bufor danych.

**CryptAcquireContext**

Z [dokumentacji](https://learn.microsoft.com/en-us/windows/win32/api/wincrypt/nf-wincrypt-cryptacquirecontexta): Funkcja **CryptAcquireContext** służy do uzyskania uchwytu do określonego kontenera kluczy w określonym dostawcy usług kryptograficznych (CSP). **Ten zwrócony uchwyt jest używany w wywołaniach funkcji CryptoAPI**, które korzystają z wybranego CSP.

**CryptCreateHash**

Inicjuje haszowanie strumienia danych. Jeśli jest używana ta funkcja, można znaleźć, który **algorytm jest używany**, sprawdzając wartość drugiego parametru:

![](<../../.gitbook/assets/image (549).png>)

Sprawdź tutaj tabelę możliwych algorytmów i ich przypisane wartości: [https://docs.microsoft.com/en-us/windows/win32/seccrypto/alg-id](https://docs.microsoft.com/en-us/windows/win32/seccrypto/alg-id)

### Stałe kodu

Czasami bardzo łatwo jest zidentyfikować algorytm dzięki konieczności użycia specjalnej i unikalnej wartości.

![](<../../.gitbook/assets/image (833).png>)

Jeśli wyszukasz pierwszą stałą w Google, otrzymasz:

![](<../../.gitbook/assets/image (529).png>)

Dlatego można założyć, że zdekompilowana funkcja to **kalkulator sha256**.\
Możesz wyszukać dowolną inną stałą i otrzymasz (prawdopodobnie) ten sam wynik.

### Informacje o danych

Jeśli kod nie zawiera istotnej stałej, może **wczytywać informacje z sekcji .data**.\
Możesz uzyskać dostęp do tych danych, **zgrupować pierwsze słowo dwubajtowe** i wyszukać je w Google, tak jak zrobiliśmy wcześniej w sekcji:

![](<../../.gitbook/assets/image (531).png>)

W tym przypadku, jeśli wyszukasz **0xA56363C6**, dowiesz się, że jest to związane z **tabelami algorytmu AES**.

## RC4 **(Szyfrowanie symetryczne)**

### Charakterystyka

Składa się z 3 głównych części:

* **Etap inicjalizacji/**: Tworzy **tabelę wartości od 0x00 do 0xFF** (łącznie 256 bajtów, 0x100). Ta tabela jest zwykle nazywana **Substitution Box** (lub SBox).
* **Etap mieszania**: Przejdzie **pętlą przez tabelę** utworzoną wcześniej (pętla 0x100 iteracji, ponownie) modyfikując każdą wartość za pomocą **półlosowych** bajtów. Aby utworzyć te półlosowe bajty, używany jest klucz RC4. Klucze RC4 mogą mieć długość **od 1 do 256 bajtów**, jednak zazwyczaj zaleca się, aby były one powyżej 5 bajtów. Zazwyczaj klucze RC4 mają długość 16 bajtów.
* **Etap XOR**: Wreszcie, tekst jawnie lub zaszyfrowany jest **XORowany z utworzonymi wcześniej wartościami**. Funkcja do szyfrowania i deszyfrowania jest taka sama. W tym celu **przejdzie się pętlą przez utworzone 256 bajtów** tak wiele razy, ile jest to konieczne. Zazwyczaj jest to rozpoznawane w zdekompilowanym kodzie za pomocą **%256 (mod 256)**.

{% hint style="info" %}
**Aby zidentyfikować RC4 w kodzie disassembly/zdekompilowanym, można sprawdzić 2 pętle o rozmiarze 0x100 (z użyciem klucza) oraz XOR danych wejściowych z 256 wartościami utworzonymi wcześniej w 2 pętlach, prawdopodobnie z użyciem %256 (mod 256)**
{% endhint %}

### **Etap inicjalizacji/Substitution Box:** (Zauważ liczbę 256 używaną jako licznik i jak 0 jest zapisywane na każdym miejscu spośród 256 znaków)

![](<../../.gitbook/assets/image (584).png>)

### **Etap mieszania:**

![](<../../.gitbook/assets/image (835).png>)

### **Etap XOR:**

![](<../../.gitbook/assets/image (904).png>)

## **AES (Szyfrowanie symetryczne)**

### **Charakterystyka**

* Użycie **skrzynek substytucji i tablic poszukiwań**
* Możliwe jest **rozróżnienie AES dzięki użyciu określonych wartości tablic poszukiwań** (stałych). _Zauważ, że **stała** może być **przechowywana** w pliku binarnym **lub tworzona**_ _**dynamicznie**._
* **Klucz szyfrowania** musi być **podzielny** przez **16** (zwykle 32B), a zazwyczaj używany jest **IV** o długości 16B.

### Stałe SBox

![](<../../.gitbook/assets/image (208).png>)

## Serpent **(Szyfrowanie symetryczne)**

### Charakterystyka

* Rzadko można znaleźć złośliwe oprogramowanie używające tego algorytmu, ale istnieją przykłady (Ursnif)
* Łatwo określić, czy algorytm to Serpent, na podstawie jego długości (bardzo długa funkcja)

### Identyfikacja

Na poniższym obrazku zauważ, jak używana jest stała **0x9E3779B9** (zauważ, że ta stała jest również używana przez inne algorytmy kryptograficzne, takie jak **TEA** -Tiny Encryption Algorithm).\
Zauważ również **rozmiar pętli** (**132**) i **liczbę operacji XOR** w instrukcjach **disassembly** oraz w przykładzie **kodu**:

![](<../../.gitbook/assets/image (547).png>)

Jak wspomniano wcześniej, ten kod może być zobrazowany w dowolnym dekompilatorze jako **bardzo długa funkcja**, ponieważ wewnątrz niej **nie ma skoków**. Zdekompilowany kod może wyglądać tak jak poniżej:

![](<../../.gitbook/assets/image (513).png>)

Dlatego możliwe jest zidentyfikowanie tego algorytmu, sprawdzając **numer magiczny** i **początkowe XORy**, widząc **bardzo długą funkcję** i **porównując** niektóre **instrukcje** z długiej funkcji **z implementacją** (taką jak przesunięcie w lewo o 7 i obrót w lewo o 22).
## RSA **(Szyfrowanie asymetryczne)**

### Charakterystyka

* Bardziej złożony niż algorytmy symetryczne
* Brak stałych! (trudno określić niestandardowe implementacje)
* KANAL (analizator krypto) nie pokazuje wskazówek dotyczących RSA, ponieważ polega na stałych.

### Identyfikacja poprzez porównania

![](<../../.gitbook/assets/image (1113).png>)

* W linii 11 (lewa) jest `+7) >> 3`, co jest takie samo jak w linii 35 (prawa): `+7) / 8`
* Linia 12 (lewa) sprawdza, czy `modulus_len < 0x040`, a w linii 36 (prawa) sprawdza, czy `inputLen+11 > modulusLen`

## MD5 & SHA (funkcje skrótu)

### Charakterystyka

* 3 funkcje: Init, Update, Final
* Podobne funkcje inicjalizacji

### Identyfikacja

**Init**

Możesz je zidentyfikować, sprawdzając stałe. Zauważ, że sha\_init ma 1 stałą, której MD5 nie ma:

![](<../../.gitbook/assets/image (406).png>)

**Transformacja MD5**

Zauważ użycie większej liczby stałych

![](<../../.gitbook/assets/image (253) (1) (1).png>)

## CRC (funkcja skrótu)

* Mniejsza i bardziej wydajna, ponieważ jej funkcją jest znajdowanie przypadkowych zmian w danych
* Wykorzystuje tablice poszukiwań (dzięki czemu można zidentyfikować stałe)

### Identyfikacja

Sprawdź **stałe tablic poszukiwań**:

![](<../../.gitbook/assets/image (508).png>)

Algorytm funkcji skrótu CRC wygląda tak:

![](<../../.gitbook/assets/image (391).png>)

## APLib (Kompresja)

### Charakterystyka

* Stałe nie są rozpoznawalne
* Możesz spróbować napisać algorytm w języku Python i szukać podobnych rzeczy online

### Identyfikacja

Graf jest dość duży:

![](<../../.gitbook/assets/image (207) (2) (1).png>)

Sprawdź **3 porównania, aby go rozpoznać**:

![](<../../.gitbook/assets/image (430).png>)
