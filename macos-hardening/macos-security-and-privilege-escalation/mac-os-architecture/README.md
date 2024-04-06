# macOS Kernel & System Extensions

<details>

<summary><strong>Dowiedz się, jak hakować AWS od zera do bohatera z</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Inne sposoby wsparcia HackTricks:

* Jeśli chcesz zobaczyć swoją **firmę reklamowaną w HackTricks** lub **pobrać HackTricks w formacie PDF**, sprawdź [**PLANY SUBSKRYPCYJNE**](https://github.com/sponsors/carlospolop)!
* Zdobądź [**oficjalne gadżety PEASS & HackTricks**](https://peass.creator-spring.com)
* Odkryj [**Rodzinę PEASS**](https://opensea.io/collection/the-peass-family), naszą kolekcję ekskluzywnych [**NFT**](https://opensea.io/collection/the-peass-family)
* **Dołącz do** 💬 [**grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegramowej**](https://t.me/peass) albo **śledź** nas na **Twitterze** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podziel się swoimi sztuczkami hakerskimi, przesyłając PR-y do** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) na GitHubie.

</details>

## Jądro XNU

**Rdzeniem macOS jest XNU**, co oznacza "X is Not Unix". To jądro składa się z **mikrojądra Mach** (o którym będzie mowa później), **oraz** elementów z dystrybucji oprogramowania Berkeley Software Distribution (**BSD**). XNU zapewnia również platformę dla **sterowników jądra poprzez system o nazwie I/O Kit**. Jądro XNU jest częścią projektu open source Darwin, co oznacza, że **jego kod źródłowy jest dostępny bezpłatnie**.

Z perspektywy badacza bezpieczeństwa lub dewelopera Unixa, **macOS** może wydawać się dość **podobne** do systemu **FreeBSD** z eleganckim interfejsem graficznym i szeregiem niestandardowych aplikacji. Większość aplikacji opracowanych dla BSD skompiluje się i uruchomi na macOS bez konieczności modyfikacji, ponieważ narzędzia wiersza poleceń znane użytkownikom Unixa są obecne w macOS. Jednakże, ponieważ jądro XNU zawiera Mach, istnieją istotne różnice między tradycyjnym systemem przypominającym Unixa a macOS, co może powodować potencjalne problemy lub zapewniać unikalne korzyści.

Open source wersja XNU: [https://opensource.apple.com/source/xnu/](https://opensource.apple.com/source/xnu/)

### Mach

Mach to **mikrojądro** zaprojektowane do bycia **zgodnym z UNIX-em**. Jedną z jego kluczowych zasad projektowych było **minimalizowanie** ilości **kodu** działającego w przestrzeni **jądra** i zamiast tego umożliwienie wielu typowych funkcji jądra, takich jak system plików, sieć i wejście/wyjście, aby **działały jako zadania na poziomie użytkownika**.

W XNU, Mach jest **odpowiedzialny za wiele krytycznych operacji na niskim poziomie**, które typowo obsługuje jądro, takie jak planowanie procesora, wielozadaniowość i zarządzanie pamięcią wirtualną.

### BSD

Jądro XNU **również zawiera** znaczną ilość kodu pochodzącego z projektu **FreeBSD**. Ten kod **działa jako część jądra razem z Machem**, w tej samej przestrzeni adresowej. Jednak kod FreeBSD w XNU może znacząco różnić się od oryginalnego kodu FreeBSD, ponieważ konieczne były modyfikacje, aby zapewnić jego kompatybilność z Mach. FreeBSD przyczynia się do wielu operacji jądra, w tym:

* Zarządzanie procesami
* Obsługa sygnałów
* Podstawowe mechanizmy bezpieczeństwa, w tym zarządzanie użytkownikami i grupami
* Infrastruktura wywołań systemowych
* Stos TCP/IP i gniazda
* Zapora sieciowa i filtrowanie pakietów

Zrozumienie interakcji między BSD a Mach może być skomplikowane ze względu na ich różne ramy koncepcyjne. Na przykład BSD używa procesów jako swojej fundamentalnej jednostki wykonawczej, podczas gdy Mach działa na podstawie wątków. Ta rozbieżność jest pogodzona w XNU poprzez **powiązanie każdego procesu BSD z zadaniem Mach**, które zawiera dokładnie jeden wątek Macha. Gdy używane jest wywołanie systemowe fork() BSD, kod BSD w jądrze używa funkcji Macha do utworzenia struktury zadania i wątku.

Ponadto, **Mach i BSD utrzymują różne modele bezpieczeństwa**: **model bezpieczeństwa Macha** opiera się na **prawach portów**, podczas gdy model bezpieczeństwa BSD działa na podstawie **własności procesu**. Różnice między tymi dwoma modelami czasami prowadziły do podatności na eskalację uprawnień lokalnych. Oprócz typowych wywołań systemowych, istnieją również **pułapki Macha, które pozwalają programom przestrzeni użytkownika na interakcję z jądrem**. Te różne elementy razem tworzą wielowymiarową, hybrydową architekturę jądra macOS.

### I/O Kit - Sterowniki

I/O Kit to otwarty, obiektowy **framework sterowników urządzeń** w jądrze XNU, obsługujący **dynamicznie ładowane sterowniki urządzeń**. Umożliwia dodawanie modułowego kodu do jądra w locie, obsługując różnorodny sprzęt.

{% content-ref url="macos-iokit.md" %}
[macos-iokit.md](macos-iokit.md)
{% endcontent-ref %}

### IPC - Komunikacja Międzyprocesowa

{% content-ref url="../macos-proces-abuse/macos-ipc-inter-process-communication/" %}
[macos-ipc-inter-process-communication](../macos-proces-abuse/macos-ipc-inter-process-communication/)
{% endcontent-ref %}

### Kernelcache

**Kernelcache** to **przedskompilowana i przedpołączona wersja jądra XNU**, wraz z niezbędnymi **sterownikami urządzeń** i **rozszerzeniami jądra**. Jest przechowywany w formacie **skompresowanym** i jest dekompresowany do pamięci podczas procesu uruchamiania systemu. Kernelcache ułatwia **szybsze uruchamianie** poprzez posiadanie gotowej do uruchomienia wersji jądra i istotnych sterowników, zmniejszając czas i zasoby, które w przeciwnym razie zostałyby wykorzystane na dynamiczne ładowanie i łączenie tych komponentów podczas uruchamiania systemu.

W iOS znajduje się w **`/System/Library/Caches/com.apple.kernelcaches/kernelcache`**, a w macOS można go znaleźć za pomocą **`find / -name kernelcache 2>/dev/null`** lub **`mdfind kernelcache | grep kernelcache`**

Można uruchomić **`kextstat`** aby sprawdzić załadowane rozszerzenia jądra.

#### IMG4

Format pliku IMG4 to format kontenera używany przez Apple w swoich urządzeniach iOS i macOS do bezpiecznego **przechowywania i weryfikacji komponentów oprogramowania** (takich jak **kernelcache**). Format IMG4 zawiera nagłówek i kilka tagów, które zawierają różne części danych, w tym rzeczywistą ładunek (jak jądro lub bootloader), sygnaturę oraz zestaw właściwości manifestu. Format obsługuje weryfikację kryptograficzną, pozwalając urządzeniu potwierdzić autentyczność i integralność komponentu oprogramowania przed jego wykonaniem.

Zazwyczaj składa się z następujących składników:

* **Ładunek (IM4P)**:
* Często skompresowany (LZFSE4, LZSS, …)
* Opcjonalnie zaszyfrowany
* **Manifest (IM4M)**:
* Zawiera sygnaturę
* Dodatkowy słownik Klucz/Wartość
* **Informacje o przywracaniu (IM4R)**:
* Znane również jako APNonce
* Zapobiega odtwarzaniu niektórych aktualizacji
* OPCJONALNIE: Zazwyczaj tego nie ma

Dekompresuj Kernelcache:

```bash
# pyimg4 (https://github.com/m1stadev/PyIMG4)
pyimg4 im4p extract -i kernelcache.release.iphone14 -o kernelcache.release.iphone14.e

# img4tool (https://github.com/tihmstar/img4tool
img4tool -e kernelcache.release.iphone14 -o kernelcache.release.iphone14.e
```

#### Symbole jądra

Czasami Apple wydaje **kernelcache** z **symbolami**. Możesz pobrać niektóre oprogramowania z symbolami, przechodząc do linków na [https://theapplewiki.com](https://theapplewiki.com/).

### IPSW

To są oprogramowania Apple, które można pobrać z [**https://ipsw.me/**](https://ipsw.me/). Oprócz innych plików zawiera **kernelcache**.\
Aby **wyodrębnić** pliki, wystarczy je po prostu **rozpakować**.

Po wyodrębnieniu oprogramowania otrzymasz plik o nazwie: **`kernelcache.release.iphone14`**. Jest w formacie **IMG4**, ciekawe informacje można wyodrębnić za pomocą:

* [**pyimg4**](https://github.com/m1stadev/PyIMG4)

{% code overflow="wrap" %}
```bash
pyimg4 im4p extract -i kernelcache.release.iphone14 -o kernelcache.release.iphone14.e
```
{% endcode %}

* [**img4tool**](https://github.com/tihmstar/img4tool)

```bash
img4tool -e kernelcache.release.iphone14 -o kernelcache.release.iphone14.e
```

Możesz sprawdzić wyodrębnione symbole jądra za pomocą: **`nm -a kernelcache.release.iphone14.e | wc -l`**

Dzięki temu możemy teraz **wyodrębnić wszystkie rozszerzenia** lub **to, które cię interesuje:**

```bash
# List all extensions
kextex -l kernelcache.release.iphone14.e
## Extract com.apple.security.sandbox
kextex -e com.apple.security.sandbox kernelcache.release.iphone14.e

# Extract all
kextex_all kernelcache.release.iphone14.e

# Check the extension for symbols
nm -a binaries/com.apple.security.sandbox | wc -l
```

## Rozszerzenia jądra macOS

macOS jest **bardzo restrykcyjny w kwestii ładowania rozszerzeń jądra** (.kext) ze względu na wysokie uprawnienia, z którymi kod będzie uruchamiany. Faktycznie, domyślnie jest to praktycznie niemożliwe (chyba że zostanie znalezione obejście).

{% content-ref url="macos-kernel-extensions.md" %}
[macos-kernel-extensions.md](macos-kernel-extensions.md)
{% endcontent-ref %}

### Rozszerzenia systemowe macOS

Zamiast korzystać z Rozszerzeń Jądra, macOS stworzył Rozszerzenia Systemowe, które oferują interakcję z jądrem za pomocą interfejsów API na poziomie użytkownika. W ten sposób programiści mogą unikać korzystania z rozszerzeń jądra.

{% content-ref url="macos-system-extensions.md" %}
[macos-system-extensions.md](macos-system-extensions.md)
{% endcontent-ref %}

## Odnośniki

* [**The Mac Hacker's Handbook**](https://www.amazon.com/-/es/Charlie-Miller-ebook-dp-B004U7MUMU/dp/B004U7MUMU/ref=mt\_other?\_encoding=UTF8\&me=\&qid=)
* [**https://taomm.org/vol1/analysis.html**](https://taomm.org/vol1/analysis.html)

<details>

<summary><strong>Naucz się hakować AWS od zera do bohatera z</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Inne sposoby wsparcia HackTricks:

* Jeśli chcesz zobaczyć swoją **firmę reklamowaną w HackTricks** lub **pobrać HackTricks w formacie PDF**, sprawdź [**PLANY SUBSKRYPCYJNE**](https://github.com/sponsors/carlospolop)!
* Zdobądź [**oficjalne gadżety PEASS & HackTricks**](https://peass.creator-spring.com)
* Odkryj [**Rodzinę PEASS**](https://opensea.io/collection/the-peass-family), naszą kolekcję ekskluzywnych [**NFT**](https://opensea.io/collection/the-peass-family)
* **Dołącz do** 💬 [**grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegramowej**](https://t.me/peass) lub **śledź** nas na **Twitterze** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podziel się swoimi sztuczkami hakerskimi, przesyłając PR-y do** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
