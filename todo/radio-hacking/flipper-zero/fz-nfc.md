# FZ - NFC

<details>

<summary><strong>Naucz się hakować AWS od zera do bohatera z</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Pracujesz w **firmie zajmującej się cyberbezpieczeństwem**? Chcesz zobaczyć swoją **firmę reklamowaną w HackTricks**? A może chcesz mieć dostęp do **najnowszej wersji PEASS lub pobrać HackTricks w formacie PDF**? Sprawdź [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Odkryj [**Rodzinę PEASS**](https://opensea.io/collection/the-peass-family), naszą kolekcję ekskluzywnych [**NFT**](https://opensea.io/collection/the-peass-family)
* Zdobądź [**oficjalne gadżety PEASS & HackTricks**](https://peass.creator-spring.com)
* **Dołącz do** [**💬**](https://emojipedia.org/speech-balloon/) [**grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegramowej**](https://t.me/peass) lub **śledź** mnie na **Twitterze** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Podziel się swoimi sztuczkami hakerskimi, przesyłając PR-y do** [**repozytorium hacktricks**](https://github.com/carlospolop/hacktricks) **i** [**repozytorium hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>

<figure><img src="/.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

Znajdź najważniejsze podatności, aby móc je szybko naprawić. Intruder śledzi powierzchnię ataku, wykonuje proaktywne skanowanie zagrożeń, znajduje problemy w całym stosie technologicznym, od interfejsów API po aplikacje internetowe i systemy chmurowe. [**Wypróbuj go za darmo**](https://www.intruder.io/?utm\_source=referral\&utm\_campaign=hacktricks) już dziś.

{% embed url="https://www.intruder.io/?utm_campaign=hacktricks&utm_source=referral" %}

***

## Wprowadzenie <a href="#9wrzi" id="9wrzi"></a>

Aby uzyskać informacje na temat RFID i NFC, sprawdź następującą stronę:

{% content-ref url="../../../radio-hacking/pentesting-rfid.md" %}
[pentesting-rfid.md](../../../radio-hacking/pentesting-rfid.md)
{% endcontent-ref %}

## Obsługiwane karty NFC <a href="#9wrzi" id="9wrzi"></a>

{% hint style="danger" %}
Oprócz kart NFC Flipper Zero obsługuje **inne rodzaje kart o wysokiej częstotliwości**, takie jak kilka kart **Mifare** Classic i Ultralight oraz **NTAG**.
{% endhint %}

Nowe rodzaje kart NFC zostaną dodane do listy obsługiwanych kart. Flipper Zero obsługuje następujące **rodzaje kart NFC typu A** (ISO 14443A):

* ﻿**Karty bankowe (EMV)** — tylko odczytaj UID, SAK i ATQA bez zapisywania.
* ﻿**Nieznane karty** — odczytaj (UID, SAK, ATQA) i emuluj UID.

Dla **kart NFC typu B, typu F i typu V**, Flipper Zero jest w stanie odczytać UID bez zapisywania go.

### Karty NFC typu A <a href="#uvusf" id="uvusf"></a>

#### Karta bankowa (EMV) <a href="#kzmrp" id="kzmrp"></a>

Flipper Zero może tylko odczytać UID, SAK, ATQA i zapisane dane na kartach bankowych **bez zapisywania**.

Ekran odczytu karty bankowejDla kart bankowych Flipper Zero może tylko odczytywać dane **bez zapisywania i emulowania**.

<figure><img src="https://cdn.flipperzero.one/Monosnap_Miro_2022-08-17_12-26-31.png?auto=format&#x26;ixlib=react-9.1.1&#x26;h=916&#x26;w=2662" alt=""><figcaption></figcaption></figure>

#### Nieznane karty <a href="#37eo8" id="37eo8"></a>

Kiedy Flipper Zero jest **niezdolny do określenia typu karty NFC**, można odczytać i zapisać tylko **UID, SAK i ATQA**.

Ekran odczytu nieznanej kartyDla nieznanych kart NFC Flipper Zero może tylko emulować UID.

<figure><img src="https://cdn.flipperzero.one/Monosnap_Miro_2022-08-17_12-27-53.png?auto=format&#x26;ixlib=react-9.1.1&#x26;h=932&#x26;w=2634" alt=""><figcaption></figcaption></figure>

### Karty NFC typu B, F i V <a href="#wyg51" id="wyg51"></a>

Dla **kart NFC typu B, F i V**, Flipper Zero może tylko **odczytać i wyświetlić UID** bez zapisywania go.

<figure><img src="https://archbee.imgix.net/3StCFqarJkJQZV-7N79yY/zBU55Fyj50TFO4U7S-OXH_screenshot-2022-08-12-at-182540.png?auto=format&#x26;ixlib=react-9.1.1&#x26;h=1080&#x26;w=2704" alt=""><figcaption></figcaption></figure>

## Działania

Aby uzyskać wprowadzenie do NFC [**przeczytaj tę stronę**](../../../radio-hacking/pentesting-rfid.md#high-frequency-rfid-tags-13.56-mhz).

### Odczytaj

Flipper Zero może **odczytywać karty NFC**, jednak **nie rozumie wszystkich protokołów**, które opierają się na ISO 14443. Jednak ponieważ **UID jest atrybutem niskiego poziomu**, możesz znaleźć się w sytuacji, gdy **UID jest już odczytane, ale protokół transferu danych na wysokim poziomie jest nadal nieznany**. Możesz odczytywać, emulować i ręcznie wprowadzać UID za pomocą Flippera dla prymitywnych czytników, które używają UID do autoryzacji.

#### Odczytanie UID vs Odczytanie danych wewnątrz <a href="#reading-the-uid-vs-reading-the-data-inside" id="reading-the-uid-vs-reading-the-data-inside"></a>

<figure><img src="../../../.gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>

W Flipperze odczytywanie tagów 13,56 MHz można podzielić na dwie części:

* **Odczyt na niskim poziomie** — odczytuje tylko UID, SAK i ATQA. Flipper próbuje zgadnąć protokół na wysokim poziomie na podstawie tych danych odczytanych z karty. Nie można być w 100% pewnym, ponieważ jest to tylko założenie oparte na pewnych czynnikach.
* **Odczyt na wysokim poziomie** — odczytuje dane z pamięci karty za pomocą określonego protokołu na wysokim poziomie. Może to być odczytanie danych na Mifare Ultralight, odczytanie sektorów z Mifare Classic lub odczytanie atrybutów karty z PayPass/Apple Pay.

### Odczytaj konkretny

W przypadku, gdy Flipper Zero nie jest w stanie znaleźć typu karty na podstawie danych na niskim poziomie, w sekcji `Extra Actions` możesz wybrać `Read Specific Card Type` i **ręcznie** **określić typ karty, który chcesz odczytać**.
#### Karty bankowe EMV (PayPass, payWave, Apple Pay, Google Pay) <a href="#emv-bank-cards-paypass-paywave-apple-pay-google-pay" id="emv-bank-cards-paypass-paywave-apple-pay-google-pay"></a>

Oprócz prostego odczytywania UID, można wyciągnąć znacznie więcej danych z karty bankowej. Możliwe jest **uzyskanie pełnego numeru karty** (16 cyfr na przedniej stronie karty), **daty ważności** oraz w niektórych przypadkach nawet **imię i nazwisko właściciela** wraz z listą **najnowszych transakcji**.\
Jednak **nie można odczytać CVV w ten sposób** (3 cyfry na tyle karty). Ponadto **karty bankowe są chronione przed atakami typu replay**, więc skopiowanie jej za pomocą Flippera i próba emulacji do dokonania płatności nie zadziała.

## Referencje

* [https://blog.flipperzero.one/rfid/](https://blog.flipperzero.one/rfid/)

<figure><img src="/.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

Znajdź najważniejsze podatności, aby szybko je naprawić. Intruder śledzi powierzchnię ataku, wykonuje proaktywne skanowanie zagrożeń, znajduje problemy w całym stosie technologicznym, od interfejsów API po aplikacje internetowe i systemy chmurowe. [**Wypróbuj go za darmo**](https://www.intruder.io/?utm\_source=referral\&utm\_campaign=hacktricks) już dziś.

{% embed url="https://www.intruder.io/?utm\_campaign=hacktricks&utm\_source=referral" %}


<details>

<summary><strong>Naucz się hakować AWS od zera do bohatera z</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Pracujesz w **firmie zajmującej się cyberbezpieczeństwem**? Chcesz zobaczyć **reklamę swojej firmy w HackTricks**? A może chcesz mieć dostęp do **najnowszej wersji PEASS lub pobrać HackTricks w formacie PDF**? Sprawdź [**PLAN SUBSKRYPCYJNY**](https://github.com/sponsors/carlospolop)!
* Odkryj [**Rodzinę PEASS**](https://opensea.io/collection/the-peass-family), naszą kolekcję ekskluzywnych [**NFT**](https://opensea.io/collection/the-peass-family)
* Zdobądź [**oficjalne gadżety PEASS & HackTricks**](https://peass.creator-spring.com)
* **Dołącz do** [**💬**](https://emojipedia.org/speech-balloon/) [**grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegramowej**](https://t.me/peass) lub **śledź** mnie na **Twitterze** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Podziel się swoimi trikami hakerskimi, przesyłając PR do repozytorium** [**hacktricks**](https://github.com/carlospolop/hacktricks) **i** [**hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
