# Zrzucanie pamięci w macOS

{% hint style="success" %}
Dowiedz się i ćwicz Hacking AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Dowiedz się i ćwicz Hacking GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Wesprzyj HackTricks</summary>

* Sprawdź [**plany subskrypcyjne**](https://github.com/sponsors/carlospolop)!
* **Dołącz do** 💬 [**grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegramowej**](https://t.me/peass) lub **śledź** nas na **Twitterze** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Udostępnij sztuczki hakerskie, przesyłając PR-y do** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) na githubie.

</details>
{% endhint %}

### [WhiteIntel](https://whiteintel.io)

<figure><img src="../../../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) to wyszukiwarka zasilana przez **dark web**, która oferuje **darmowe** funkcje do sprawdzania, czy firma lub jej klienci zostali **skompromitowani** przez **złośliwe oprogramowanie kradnące informacje**.

Ich głównym celem WhiteIntel jest zwalczanie przejęć kont i ataków ransomware wynikających z złośliwego oprogramowania kradnącego informacje.

Możesz sprawdzić ich stronę internetową i wypróbować ich silnik **za darmo** pod adresem:

{% embed url="https://whiteintel.io" %}

***

## Artefakty pamięci

### Pliki wymiany

Pliki wymiany, takie jak `/private/var/vm/swapfile0`, służą jako **bufory, gdy pamięć fizyczna jest pełna**. Gdy nie ma już miejsca w pamięci fizycznej, jej dane są przenoszone do pliku wymiany, a następnie przywracane do pamięci fizycznej w razie potrzeby. Może być obecnych wiele plików wymiany o nazwach takich jak swapfile0, swapfile1, itd.

### Obraz hibernacji

Plik znajdujący się w `/private/var/vm/sleepimage` jest kluczowy podczas trybu **hibernacji**. **Dane z pamięci są przechowywane w tym pliku podczas hibernacji systemu OS X**. Po obudzeniu komputera system odzyskuje dane pamięci z tego pliku, pozwalając użytkownikowi kontynuować tam, gdzie skończył.

Warto zauważyć, że w nowoczesnych systemach MacOS ten plik jest zazwyczaj szyfrowany ze względów bezpieczeństwa, co utrudnia odzyskanie danych.

* Aby sprawdzić, czy szyfrowanie jest włączone dla sleepimage, można uruchomić polecenie `sysctl vm.swapusage`. Pokaże to, czy plik jest zaszyfrowany.

### Dzienniki ciśnienia pamięci

Innym ważnym plikiem związanym z pamięcią w systemach MacOS są **dzienniki ciśnienia pamięci**. Te dzienniki znajdują się w `/var/log` i zawierają szczegółowe informacje o użyciu pamięci systemu i zdarzeniach związanych z ciśnieniem pamięci. Mogą być szczególnie przydatne do diagnozowania problemów związanych z pamięcią lub zrozumienia sposobu zarządzania pamięcią przez system w czasie.

## Zrzucanie pamięci za pomocą osxpmem

Aby zrzucić pamięć w maszynie z systemem MacOS, można użyć [**osxpmem**](https://github.com/google/rekall/releases/download/v1.5.1/osxpmem-2.1.post4.zip).

**Uwaga**: Poniższe instrukcje będą działać tylko dla komputerów Mac z architekturą Intel. To narzędzie jest teraz zarchiwizowane, a ostatnie wydanie miało miejsce w 2017 roku. Pobrany binarny plik za pomocą poniższych instrukcji jest przeznaczony dla chipów Intel, ponieważ Apple Silicon nie istniał w 2017 roku. Możliwe jest skompilowanie binarnego pliku dla architektury arm64, ale będziesz musiał spróbować samodzielnie.
```bash
#Dump raw format
sudo osxpmem.app/osxpmem --format raw -o /tmp/dump_mem

#Dump aff4 format
sudo osxpmem.app/osxpmem -o /tmp/dump_mem.aff4
```
Jeśli napotkasz ten błąd: `osxpmem.app/MacPmem.kext nie udało się załadować - (libkern/kext) błąd uwierzytelniania (własność pliku/uprawnienia); sprawdź dzienniki systemowe/jądra w poszukiwaniu błędów lub spróbuj kextutil(8)` Możesz to naprawić wykonując:
```bash
sudo cp -r osxpmem.app/MacPmem.kext "/tmp/"
sudo kextutil "/tmp/MacPmem.kext"
#Allow the kext in "Security & Privacy --> General"
sudo osxpmem.app/osxpmem --format raw -o /tmp/dump_mem
```
**Inne błędy** można naprawić, **zezwalając na załadowanie kext** w "Bezpieczeństwo i prywatność --> Ogólne", po prostu **zezwól** na to.

Możesz także użyć tego **onelinera** do pobrania aplikacji, załadowania kext i zrzucenia pamięci:

{% code overflow="wrap" %}
```bash
sudo su
cd /tmp; wget https://github.com/google/rekall/releases/download/v1.5.1/osxpmem-2.1.post4.zip; unzip osxpmem-2.1.post4.zip; chown -R root:wheel osxpmem.app/MacPmem.kext; kextload osxpmem.app/MacPmem.kext; osxpmem.app/osxpmem --format raw -o /tmp/dump_mem
```
{% endcode %}

### [WhiteIntel](https://whiteintel.io)

<figure><img src="../../../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) to wyszukiwarka zasilana przez **dark web**, która oferuje **darmowe** funkcje do sprawdzenia, czy firma lub jej klienci zostali **skompromitowani** przez **złośliwe oprogramowanie kradnące informacje**.

Ich głównym celem WhiteIntel jest zwalczanie przejęć kont i ataków ransomware wynikających z złośliwego oprogramowania kradnącego informacje.

Możesz sprawdzić ich stronę internetową i wypróbować ich silnik za **darmo** pod adresem:

{% embed url="https://whiteintel.io" %}

{% hint style="success" %}
Dowiedz się i ćwicz Hacking AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Dowiedz się i ćwicz Hacking GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Wesprzyj HackTricks</summary>

* Sprawdź [**plany subskrypcyjne**](https://github.com/sponsors/carlospolop)!
* **Dołącz do** 💬 [**Grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegramowej**](https://t.me/peass) lub **śledź** nas na **Twitterze** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Udostępniaj sztuczki hakerskie, przesyłając PR-y do** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repozytoriów github.

</details>
{% endhint %}
