# Rozszerzenia systemowe macOS

<details>

<summary><strong>Dowiedz się, jak hakować AWS od zera do bohatera z</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Inne sposoby wsparcia HackTricks:

* Jeśli chcesz zobaczyć swoją **firmę reklamowaną w HackTricks** lub **pobrać HackTricks w formacie PDF**, sprawdź [**PLANY SUBSKRYPCYJNE**](https://github.com/sponsors/carlospolop)!
* Zdobądź [**oficjalne gadżety PEASS & HackTricks**](https://peass.creator-spring.com)
* Odkryj [**Rodzinę PEASS**](https://opensea.io/collection/the-peass-family), naszą kolekcję ekskluzywnych [**NFT**](https://opensea.io/collection/the-peass-family)
* **Dołącz do** 💬 [**grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegramowej**](https://t.me/peass) lub **śledź** nas na **Twitterze** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podziel się swoimi sztuczkami hakerskimi, przesyłając PR-y do** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repozytoriów na GitHubie.

</details>

## Rozszerzenia systemowe / Framework bezpieczeństwa końcowego

W przeciwieństwie do Rozszerzeń jądra, **Rozszerzenia systemowe działają w przestrzeni użytkownika** zamiast w przestrzeni jądra, co zmniejsza ryzyko awarii systemu spowodowanej nieprawidłowym działaniem rozszerzenia.

<figure><img src="../../../.gitbook/assets/image (603).png" alt="https://knight.sc/images/system-extension-internals-1.png"><figcaption></figcaption></figure>

Istnieją trzy rodzaje rozszerzeń systemowych: Rozszerzenia **DriverKit**, Rozszerzenia **Sieciowe** i Rozszerzenia **Bezpieczeństwa Końcowego**.

### **Rozszerzenia DriverKit**

DriverKit to zastępstwo dla rozszerzeń jądra, które **zapewnia obsługę sprzętu**. Pozwala sterownikom urządzeń (takim jak USB, szeregowe, NIC i HID) działać w przestrzeni użytkownika zamiast w przestrzeni jądra. Framework DriverKit zawiera **wersje przestrzeni użytkownika pewnych klas I/O Kit**, a jądro przekazuje normalne zdarzenia I/O Kit do przestrzeni użytkownika, oferując bezpieczniejsze środowisko dla tych sterowników.

### **Rozszerzenia Sieciowe**

Rozszerzenia Sieciowe umożliwiają dostosowanie zachowań sieciowych. Istnieje kilka rodzajów Rozszerzeń Sieciowych:

* **Proxy Aplikacji**: Służy do tworzenia klienta VPN, który implementuje protokół VPN oparty na przepływach, co oznacza, że obsługuje ruch sieciowy na podstawie połączeń (lub przepływów) zamiast pojedynczych pakietów.
* **Tunel Pakietowy**: Służy do tworzenia klienta VPN, który implementuje protokół VPN oparty na pakietach, co oznacza, że obsługuje ruch sieciowy na podstawie pojedynczych pakietów.
* **Filtr Danych**: Służy do filtrowania "przepływów" sieciowych. Może monitorować lub modyfikować dane sieciowe na poziomie przepływu.
* **Filtr Pakietów**: Służy do filtrowania pojedynczych pakietów sieciowych. Może monitorować lub modyfikować dane sieciowe na poziomie pakietu.
* **Proxy DNS**: Służy do tworzenia niestandardowego dostawcy DNS. Może być używany do monitorowania lub modyfikowania żądań i odpowiedzi DNS.

## Framework Bezpieczeństwa Końcowego

Bezpieczeństwo Końcowe to framework dostarczany przez Apple w macOS, który zapewnia zestaw interfejsów API do bezpieczeństwa systemu. Jest przeznaczony do użytku przez **dostawców bezpieczeństwa i deweloperów do budowania produktów, które mogą monitorować i kontrolować aktywność systemu** w celu identyfikacji i ochrony przed działaniami złośliwymi.

Ten framework zapewnia **zbiór interfejsów API do monitorowania i kontrolowania aktywności systemu**, takich jak wykonania procesów, zdarzenia systemu plików, zdarzenia sieciowe i jądra.

Rdzeń tego frameworka jest zaimplementowany w jądrze jako Rozszerzenie Jądra (KEXT) znajdujące się w **`/System/Library/Extensions/EndpointSecurity.kext`**. To KEXT składa się z kilku kluczowych komponentów:

* **EndpointSecurityDriver**: Działa jako "punkt wejścia" dla rozszerzenia jądra. Jest głównym punktem interakcji między systemem operacyjnym a frameworkiem Bezpieczeństwa Końcowego.
* **EndpointSecurityEventManager**: Odpowiada za implementację haków jądra. Haki jądra pozwalają frameworkowi monitorować zdarzenia systemowe poprzez przechwytywanie wywołań systemowych.
* **EndpointSecurityClientManager**: Zarządza komunikacją z klientami przestrzeni użytkownika, śledząc, które klienty są podłączone i potrzebują otrzymywać powiadomienia o zdarzeniach.
* **EndpointSecurityMessageManager**: Wysyła wiadomości i powiadomienia o zdarzeniach do klientów przestrzeni użytkownika.

Zdarzenia, które framework Bezpieczeństwa Końcowego może monitorować, są kategoryzowane jako:

* Zdarzenia plików
* Zdarzenia procesów
* Zdarzenia gniazd
* Zdarzenia jądra (takie jak ładowanie/wyładowanie rozszerzenia jądra lub otwieranie urządzenia I/O Kit)

### Architektura Frameworka Bezpieczeństwa Końcowego

<figure><img src="../../../.gitbook/assets/image (1065).png" alt="https://www.youtube.com/watch?v=jaVkpM1UqOs"><figcaption></figcaption></figure>

**Komunikacja przestrzeni użytkownika** z frameworkiem Bezpieczeństwa Końcowego odbywa się za pośrednictwem klasy IOUserClient. Używane są dwie różne podklasy, w zależności od rodzaju wywołującego:

* **EndpointSecurityDriverClient**: Wymaga uprawnienia `com.apple.private.endpoint-security.manager`, które posiada tylko proces systemowy `endpointsecurityd`.
* **EndpointSecurityExternalClient**: Wymaga uprawnienia `com.apple.developer.endpoint-security.client`. Typowo byłoby to używane przez oprogramowanie zewnętrznych dostawców bezpieczeństwa, które musi współdziałać z frameworkiem Bezpieczeństwa Końcowego.

Rozszerzenia Bezpieczeństwa Końcowego:**`libEndpointSecurity.dylib`** to biblioteka C, którą używają rozszerzenia systemowe do komunikacji z jądrem. Ta biblioteka korzysta z I/O Kit (`IOKit`) do komunikacji z Rozszerzeniem Bezpieczeństwa Końcowego KEXT.

**`endpointsecurityd`** to kluczowy demon systemowy zaangażowany w zarządzanie i uruchamianie rozszerzeń systemowych bezpieczeństwa końcowego, zwłaszcza podczas wczesnego procesu uruchamiania. **Tylko rozszerzenia systemowe** oznaczone jako **`NSEndpointSecurityEarlyBoot`** w swoim pliku `Info.plist` otrzymują to wczesne uruchomienie.

Inny demon systemowy, **`sysextd`**, **waliduje rozszerzenia systemowe** i przenosi je do odpowiednich lokalizacji systemowych. Następnie prosi odpowiedniego demona o załadowanie rozszerzenia. **`SystemExtensions.framework`** jest odpowiedzialny za aktywowanie i dezaktywowanie rozszerzeń systemowych.

## Omijanie ESF

ESF jest używany przez narzędzia bezpieczeństwa, które będą próbować wykryć red teamera, więc jakiekolwiek informacje na temat tego, jak to można ominąć, brzmią interesująco.

### CVE-2021-30965

Rzecz w tym, że aplikacja bezpieczeństwa musi mieć **Uprawnienia Pełnego Dostępu do Dysku**. Więc jeśli atakujący mógłby to usunąć, mógłby uniemożliwić działanie oprogramowania:
```bash
tccutil reset All
```
Dla **więcej informacji** na temat tego bypassu i powiązanych sprawdź prezentację [#OBTS v5.0: "The Achilles Heel of EndpointSecurity" - Fitzl Csaba](https://www.youtube.com/watch?v=lQO7tvNCoTI)

Na końcu problem został rozwiązany poprzez nadanie nowego uprawnienia **`kTCCServiceEndpointSecurityClient`** aplikacji zabezpieczeń zarządzanej przez **`tccd`**, dzięki czemu `tccutil` nie będzie czyścić jej uprawnień, co uniemożliwi jej uruchomienie.

## Referencje

* [**OBTS v3.0: "Endpoint Security & Insecurity" - Scott Knight**](https://www.youtube.com/watch?v=jaVkpM1UqOs)
* [**https://knight.sc/reverse%20engineering/2019/08/24/system-extension-internals.html**](https://knight.sc/reverse%20engineering/2019/08/24/system-extension-internals.html)

<details>

<summary><strong>Naucz się hakować AWS od zera do bohatera z</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Inne sposoby wsparcia HackTricks:

* Jeśli chcesz zobaczyć swoją **firmę reklamowaną w HackTricks** lub **pobrać HackTricks w formacie PDF**, sprawdź [**PLANY SUBSKRYPCYJNE**](https://github.com/sponsors/carlospolop)!
* Kup [**oficjalne gadżety PEASS & HackTricks**](https://peass.creator-spring.com)
* Odkryj [**Rodzinę PEASS**](https://opensea.io/collection/the-peass-family), naszą kolekcję ekskluzywnych [**NFT**](https://opensea.io/collection/the-peass-family)
* **Dołącz do** 💬 [**grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegram**](https://t.me/peass) lub **śledź** nas na **Twitterze** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podziel się swoimi sztuczkami hakerskimi, przesyłając PR-y do** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
