# macOS Keychain

<details>

<summary><strong>Naucz się hakować AWS od zera do bohatera z</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Inne sposoby wsparcia HackTricks:

* Jeśli chcesz zobaczyć swoją **firmę reklamowaną w HackTricks** lub **pobrać HackTricks w formacie PDF**, sprawdź [**PLANY SUBSKRYPCYJNE**](https://github.com/sponsors/carlospolop)!
* Kup [**oficjalne gadżety PEASS & HackTricks**](https://peass.creator-spring.com)
* Odkryj [**Rodzinę PEASS**](https://opensea.io/collection/the-peass-family), naszą kolekcję ekskluzywnych [**NFT**](https://opensea.io/collection/the-peass-family)
* **Dołącz do** 💬 [**grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegramowej**](https://t.me/peass) lub **śledź** nas na **Twitterze** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Podziel się swoimi sztuczkami hakerskimi, przesyłając PR-y do** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) na GitHubie.

</details>

### [WhiteIntel](https://whiteintel.io)

<figure><img src="/.gitbook/assets/image (1224).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) to silnik wyszukiwania zasilany **dark webem**, który oferuje **darmowe** funkcje sprawdzania, czy firma lub jej klienci zostali **skompromitowani** przez **złośliwe oprogramowanie kradnące dane**.

Ich głównym celem WhiteIntel jest zwalczanie przejęć kont i ataków ransomware wynikających z złośliwego oprogramowania kradnącego informacje.

Możesz odwiedzić ich stronę i wypróbować ich silnik **za darmo** pod adresem:

{% embed url="https://whiteintel.io" %}

---

## Główne Schowki

* **Schowek Użytkownika** (`~/Library/Keychains/login.keycahin-db`), który służy do przechowywania **danych uwierzytelniających specyficznych dla użytkownika**, takich jak hasła do aplikacji, hasła internetowe, certyfikaty wygenerowane przez użytkownika, hasła sieciowe oraz klucze publiczne/prywatne wygenerowane przez użytkownika.
* **Schowek Systemowy** (`/Library/Keychains/System.keychain`), który przechowuje **dane uwierzytelniające na poziomie systemowym**, takie jak hasła WiFi, certyfikaty root systemu, prywatne klucze systemowe oraz hasła aplikacji systemowych.

### Dostęp do Hasła w Schowku

Te pliki, mimo że nie posiadają wbudowanej ochrony i mogą być **pobrane**, są szyfrowane i wymagają **czystego tekstu hasła użytkownika do odszyfrowania**. Narzędzie takie jak [**Chainbreaker**](https://github.com/n0fate/chainbreaker) może być użyte do odszyfrowania.

## Ochrona Pozycji w Schowku

### Listy Kontroli Dostępu (ACLs)

Każda pozycja w schowku jest regulowana przez **Listy Kontroli Dostępu (ACLs)**, które określają, kto może wykonywać różne czynności na pozycji w schowku, w tym:

* **ACLAuhtorizationExportClear**: Pozwala posiadaczowi uzyskać czysty tekst tajemnicy.
* **ACLAuhtorizationExportWrapped**: Pozwala posiadaczowi uzyskać zaszyfrowany czysty tekst za pomocą innego podanego hasła.
* **ACLAuhtorizationAny**: Pozwala posiadaczowi wykonać dowolną czynność.

ACLs są dodatkowo wspierane przez **listę zaufanych aplikacji**, które mogą wykonywać te czynności bez prośby o zgodę. Mogą to być:

* &#x20;**N`il`** (brak wymaganej autoryzacji, **każdy jest zaufany**)
* Pusta lista (**nikt nie jest zaufany**)
* **Lista** konkretnych **aplikacji**.

Pozycja może również zawierać klucz **`ACLAuthorizationPartitionID`,** który służy do identyfikacji **teamid, apple** i **cdhash.**

* Jeśli określono **teamid**, to aby **uzyskać dostęp do wartości pozycji** bez **prośby**, używana aplikacja musi mieć **ten sam teamid**.
* Jeśli określono **apple**, to aplikacja musi być **podpisana** przez **Apple**.
* Jeśli wskazano **cdhash**, to **aplikacja** musi mieć określony **cdhash**.

### Tworzenie Pozycji w Schowku

Gdy **nowa** **pozycja** jest tworzona za pomocą **`Keychain Access.app`**, obowiązują następujące zasady:

* Wszystkie aplikacje mogą szyfrować.
* **Żadna aplikacja** nie może eksportować/odszyfrować (bez prośby użytkownika).
* Wszystkie aplikacje mogą zobaczyć sprawdzenie integralności.
* Żadna aplikacja nie może zmieniać ACLs.
* **partitionID** jest ustawione na **`apple`**.

Gdy **aplikacja tworzy pozycję w schowku**, zasady są nieco inne:

* Wszystkie aplikacje mogą szyfrować.
* Tylko **tworząca aplikacja** (lub inne aplikacje dodane explicite) mogą eksportować/odszyfrować (bez prośby użytkownika).
* Wszystkie aplikacje mogą zobaczyć sprawdzenie integralności.
* Żadna aplikacja nie może zmieniać ACLs.
* **partitionID** jest ustawione na **`teamid:[teamID tutaj]`**.

## Dostęp do Schowka

### `security`
```bash
# Dump all metadata and decrypted secrets (a lot of pop-ups)
security dump-keychain -a -d

# Find generic password for the "Slack" account and print the secrets
security find-generic-password -a "Slack" -g

# Change the specified entrys PartitionID entry
security set-generic-password-parition-list -s "test service" -a "test acount" -S
```
### Interfejsy programowania aplikacji

{% hint style="success" %}
**Wyliczenie i wyciek** tajemnic z **keychaina**, które **nie generują monitu**, można wykonać za pomocą narzędzia [**LockSmith**](https://github.com/its-a-feature/LockSmith)
{% endhint %}

Lista i uzyskanie **informacji** o każdym wpisie w keychainie:

* Interfejs programowania aplikacji **`SecItemCopyMatching`** udostępnia informacje o każdym wpisie, a podczas korzystania z niego można ustawić kilka atrybutów:
* **`kSecReturnData`**: Jeśli jest ustawione na true, spróbuje zdekodować dane (ustaw na false, aby uniknąć potencjalnych okienek)
* **`kSecReturnRef`**: Uzyskaj również odniesienie do elementu keychaina (ustaw na true, jeśli później zauważysz, że możesz zdekodować bez okienka)
* **`kSecReturnAttributes`**: Uzyskaj metadane dotyczące wpisów
* **`kSecMatchLimit`**: Ile wyników zwrócić
* **`kSecClass`**: Jaki rodzaj wpisu w keychainie

Uzyskaj **ACL** każdego wpisu:

* Za pomocą interfejsu programowania aplikacji **`SecAccessCopyACLList`** można uzyskać **ACL dla elementu keychaina**, który zwróci listę ACL (takich jak `ACLAuhtorizationExportClear` i inne wcześniej wspomniane), gdzie każda lista zawiera:
* Opis
* **Lista zaufanych aplikacji**. Może to być:
* Aplikacja: /Applications/Slack.app
* Binarny plik: /usr/libexec/airportd
* Grupa: group://AirPort

Eksportuj dane:

* Interfejs programowania aplikacji **`SecKeychainItemCopyContent`** pobiera tekst jawny
* Interfejs programowania aplikacji **`SecItemExport`** eksportuje klucze i certyfikaty, ale może być konieczne ustawienie haseł do zaszyfrowania eksportowanych treści

A oto **wymagania**, aby móc **eksportować tajemnicę bez monitu**:

* Jeśli jest **1+ zaufanych** aplikacji wymienionych:
* Potrzebne są odpowiednie **uprawnienia** (**`Nil`**, lub być **częścią** listy dozwolonych aplikacji w autoryzacji dostępu do informacji o tajemnicy)
* Kod musi pasować do **PartitionID**
* Kod musi pasować do tego samego co kod jednej **zaufanej aplikacji** (lub być członkiem odpowiedniej grupy KeychainAccessGroup)
* Jeśli **wszystkie aplikacje są zaufane**:
* Potrzebne są odpowiednie **uprawnienia**
* Kod musi pasować do **PartitionID**
* Jeśli **brak PartitionID**, to nie jest to wymagane

{% hint style="danger" %}
Dlatego jeśli jest **wymieniona 1 aplikacja**, musisz **wstrzyknąć kod w tę aplikację**.

Jeśli jest wskazane **apple** w **partitionID**, można uzyskać do niego dostęp za pomocą **`osascript`**, więc wszystko, co ufa wszystkim aplikacjom z apple w partitionID. Można również użyć **`Pythona`** do tego.
{% endhint %}

### Dwa dodatkowe atrybuty

* **Niewidoczny**: Jest to flaga logiczna do **ukrycia** wpisu z aplikacji **UI** Keychain
* **Ogólny**: Służy do przechowywania **metadanych** (więc NIE JEST SZYFROWANY)
* Microsoft przechowywał w postaci zwykłego tekstu wszystkie tokeny odświeżania do dostępu do wrażliwych punktów końcowych.

## Odnośniki

* [**#OBTS v5.0: "Lock Picking the macOS Keychain" - Cody Thomas**](https://www.youtube.com/watch?v=jKE1ZW33JpY)

### [WhiteIntel](https://whiteintel.io)

<figure><img src="/.gitbook/assets/image (1224).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) to wyszukiwarka zasilana przez **dark web**, która oferuje **darmowe** funkcje do sprawdzenia, czy firma lub jej klienci nie zostali **skompromitowani** przez **złośliwe oprogramowanie kradnące informacje**.

Ich głównym celem jest zwalczanie przejęć kont i ataków ransomware wynikających z złośliwego oprogramowania kradnącego informacje.

Możesz odwiedzić ich stronę internetową i wypróbować ich silnik **za darmo** pod adresem:

{% embed url="https://whiteintel.io" %}

<details>

<summary><strong>Dowiedz się, jak hakować AWS od zera do bohatera z</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Inne sposoby wsparcia HackTricks:

* Jeśli chcesz zobaczyć swoją **firmę reklamowaną w HackTricks** lub **pobrać HackTricks w formacie PDF**, sprawdź [**PLANY SUBSKRYPCYJNE**](https://github.com/sponsors/carlospolop)!
* Kup [**oficjalne gadżety PEASS & HackTricks**](https://peass.creator-spring.com)
* Odkryj [**Rodzinę PEASS**](https://opensea.io/collection/the-peass-family), naszą kolekcję ekskluzywnych [**NFT**](https://opensea.io/collection/the-peass-family)
* **Dołącz do** 💬 [**grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegramowej**](https://t.me/peass) lub **śledź** nas na **Twitterze** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Podziel się swoimi sztuczkami hakerskimi, przesyłając PR do** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
