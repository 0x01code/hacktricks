# Narzędzia do wycinania i odzyskiwania danych

<details>

<summary><strong>Naucz się hakować AWS od zera do bohatera z</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Inne sposoby wsparcia HackTricks:

* Jeśli chcesz zobaczyć swoją **firmę reklamowaną w HackTricks** lub **pobrać HackTricks w formacie PDF**, sprawdź [**PLANY SUBSKRYPCYJNE**](https://github.com/sponsors/carlospolop)!
* Kup [**oficjalne gadżety PEASS & HackTricks**](https://peass.creator-spring.com)
* Odkryj [**Rodzinę PEASS**](https://opensea.io/collection/the-peass-family), naszą kolekcję ekskluzywnych [**NFT**](https://opensea.io/collection/the-peass-family)
* **Dołącz do** 💬 [**grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegram**](https://t.me/peass) lub **śledź** nas na **Twitterze** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podziel się swoimi sztuczkami hakerskimi, przesyłając PR-y do** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) na GitHubie.

</details>

## Narzędzia do wycinania i odzyskiwania danych

Więcej narzędzi znajdziesz na [https://github.com/Claudio-C/awesome-datarecovery](https://github.com/Claudio-C/awesome-datarecovery)

### Autopsy

Najczęściej używanym narzędziem w dziedzinie kryminalistyki do wyodrębniania plików z obrazów jest [**Autopsy**](https://www.autopsy.com/download/). Pobierz go, zainstaluj i spraw, aby przetworzył plik w celu znalezienia "ukrytych" plików. Zauważ, że Autopsy jest przeznaczony do obsługi obrazów dysków i innych rodzajów obrazów, ale nie prostych plików.

### Binwalk <a href="#binwalk" id="binwalk"></a>

**Binwalk** to narzędzie do analizy plików binarnych w celu znalezienia osadzonej zawartości. Można je zainstalować za pomocą `apt`, a jego źródło znajduje się na [GitHub](https://github.com/ReFirmLabs/binwalk).

**Przydatne polecenia**:
```bash
sudo apt install binwalk #Insllation
binwalk file #Displays the embedded data in the given file
binwalk -e file #Displays and extracts some files from the given file
binwalk --dd ".*" file #Displays and extracts all files from the given file
```
### Foremost

Innym powszechnie używanym narzędziem do znajdowania ukrytych plików jest **foremost**. Konfigurację foremost można znaleźć w `/etc/foremost.conf`. Jeśli chcesz wyszukać określone pliki, odkomentuj je. Jeśli nic nie odkomentujesz, foremost będzie przeszukiwał domyślnie skonfigurowane typy plików.
```bash
sudo apt-get install foremost
foremost -v -i file.img -o output
#Discovered files will appear inside the folder "output"
```
### **Scalpel**

**Scalpel** to kolejne narzędzie, które można użyć do znalezienia i wyodrębnienia **plików osadzonych w pliku**. W tym przypadku będziesz musiał odkomentować z pliku konfiguracyjnego (_/etc/scalpel/scalpel.conf_) typy plików, które chcesz wyodrębnić.
```bash
sudo apt-get install scalpel
scalpel file.img -o output
```
### Bulk Extractor

To narzędzie znajduje się w Kali, ale można je znaleźć tutaj: [https://github.com/simsong/bulk\_extractor](https://github.com/simsong/bulk\_extractor)

To narzędzie może przeskanować obraz i **wydobyć pcapy** wewnątrz niego, **informacje sieciowe (adresy URL, domeny, adresy IP, adresy MAC, maile)** oraz więcej **plików**. Wystarczy tylko:
```
bulk_extractor memory.img -o out_folder
```
Przejdź przez **wszystkie informacje**, które narzędzie zgromadziło (hasła?), **analizuj** **pakiety** (czytaj [**Analiza Pcap**](../pcap-inspection/)), szukaj **dziwnych domen** (domeny związane z **malware** lub **nieistniejące**).

### PhotoRec

Możesz go znaleźć na [https://www.cgsecurity.org/wiki/TestDisk\_Download](https://www.cgsecurity.org/wiki/TestDisk\_Download)

Dostępny jest w wersjach GUI i CLI. Możesz wybrać **typy plików**, które chcesz, aby PhotoRec wyszukał.

![](<../../../.gitbook/assets/image (524).png>)

### binvis

Sprawdź [kod](https://code.google.com/archive/p/binvis/) i [narzędzie na stronie internetowej](https://binvis.io/#/).

#### Funkcje BinVis

* Wizualizator **struktury** aktywny
* Wiele wykresów dla różnych punktów skupienia
* Skupianie się na fragmentach próbki
* **Widzenie ciągów i zasobów**, w plikach PE lub ELF, np.
* Uzyskiwanie **wzorców** do kryptografii plików
* **Wykrywanie** algorytmów pakujących lub kodujących
* **Identyfikacja** steganografii poprzez wzorce
* Wizualne porównywanie binarne

BinVis to świetny **punkt wyjścia do zapoznania się z nieznanym celem** w scenariuszu black-boxing.

## Konkretne narzędzia do odzyskiwania danych

### FindAES

Wyszukuje klucze AES, szukając ich harmonogramów kluczy. Potrafi znaleźć klucze 128, 192 i 256 bitowe, takie jak te używane przez TrueCrypt i BitLocker.

Pobierz [tutaj](https://sourceforge.net/projects/findaes/).

## Narzędzia uzupełniające

Możesz użyć [**viu** ](https://github.com/atanunq/viu), aby zobaczyć obrazy z terminala.\
Możesz użyć narzędzia wiersza poleceń systemu Linux **pdftotext**, aby przekształcić plik pdf na tekst i go przeczytać.

<details>

<summary><strong>Dowiedz się, jak hakować AWS od zera do bohatera z</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Inne sposoby wsparcia HackTricks:

* Jeśli chcesz zobaczyć swoją **firmę reklamowaną w HackTricks** lub **pobrać HackTricks w formacie PDF**, sprawdź [**PLANY SUBSKRYPCYJNE**](https://github.com/sponsors/carlospolop)!
* Kup [**oficjalne gadżety PEASS & HackTricks**](https://peass.creator-spring.com)
* Odkryj [**Rodzinę PEASS**](https://opensea.io/collection/the-peass-family), naszą kolekcję ekskluzywnych [**NFT**](https://opensea.io/collection/the-peass-family)
* **Dołącz do** 💬 [**grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegramowej**](https://t.me/peass) lub **śledź** nas na **Twitterze** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podziel się swoimi sztuczkami hakerskimi, przesyłając PR-y do** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
