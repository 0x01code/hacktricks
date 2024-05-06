# Certyfikaty

<details>

<summary><strong>Dowiedz się, jak hakować AWS od zera do bohatera z</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Inne sposoby wsparcia HackTricks:

* Jeśli chcesz zobaczyć swoją **firmę reklamowaną w HackTricks** lub **pobrać HackTricks w formacie PDF**, sprawdź [**PLANY SUBSKRYPCYJNE**](https://github.com/sponsors/carlospolop)!
* Zdobądź [**oficjalne gadżety PEASS & HackTricks**](https://peass.creator-spring.com)
* Odkryj [**Rodzinę PEASS**](https://opensea.io/collection/the-peass-family), naszą kolekcję ekskluzywnych [**NFT**](https://opensea.io/collection/the-peass-family)
* **Dołącz do** 💬 [**grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegramowej**](https://t.me/peass) lub **śledź** nas na **Twitterze** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podziel się swoimi sztuczkami hakerskimi, przesyłając PR-y do** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) na GitHubie.

</details>

<figure><img src="../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

\
Użyj [**Trickest**](https://trickest.com/?utm_source=hacktricks&utm_medium=text&utm_campaign=ppc&utm_term=trickest&utm_content=certificates), aby łatwo budować i **automatyzować przepływy pracy** zasilane przez najbardziej zaawansowane narzędzia społecznościowe na świecie.\
Otrzymaj dostęp już dziś:

{% embed url="https://trickest.com/?utm_source=hacktricks&utm_medium=banner&utm_campaign=ppc&utm_content=certificates" %}

## Co to jest Certyfikat

**Certyfikat klucza publicznego** to cyfrowe ID używane w kryptografii do potwierdzenia, że ktoś jest właścicielem klucza publicznego. Zawiera szczegóły klucza, tożsamość właściciela (podmiot) i cyfrowy podpis od zaufanego organu (wydawcy). Jeśli oprogramowanie ufa wydawcy i podpis jest ważny, możliwa jest bezpieczna komunikacja z właścicielem klucza.

Certyfikaty są głównie wydawane przez [organizacje certyfikujące](https://en.wikipedia.org/wiki/Certificate\_authority) (CAs) w konfiguracji [infrastruktury klucza publicznego](https://en.wikipedia.org/wiki/Public-key\_infrastructure) (PKI). Inną metodą jest [sieć zaufania](https://en.wikipedia.org/wiki/Web\_of\_trust), gdzie użytkownicy bezpośrednio weryfikują klucze innych. Powszechnym formatem certyfikatów jest [X.509](https://en.wikipedia.org/wiki/X.509), który może być dostosowany do konkretnych potrzeb, zgodnie z RFC 5280.

## Wspólne Pola x509

### **Wspólne Pola w Certyfikatach x509**

W certyfikatach x509 kilka **pól** odgrywa kluczową rolę w zapewnieniu ważności i bezpieczeństwa certyfikatu. Oto ich rozbudowa:

* **Numer Wersji** oznacza wersję formatu x509.
* **Numer Seryjny** jednoznacznie identyfikuje certyfikat w systemie Organizacji Certyfikującej (CA), głównie do śledzenia unieważnień.
* Pole **Podmiotu** reprezentuje właściciela certyfikatu, który może być maszyną, osobą fizyczną lub organizacją. Zawiera szczegółowe identyfikatory, takie jak:
* **Nazwa Powszechna (CN)**: Domeny objęte certyfikatem.
* **Kraj (C)**, **Miejscowość (L)**, **Stan lub Prowincja (ST, S, lub P)**, **Organizacja (O)** i **Jednostka Organizacyjna (OU)** dostarczają szczegółów geograficznych i organizacyjnych.
* **Nazwa Wyróżniająca (DN)** zawiera pełną identyfikację podmiotu.
* **Wydawca** określa, kto zweryfikował i podpisał certyfikat, zawierając podobne podpola jak Podmiot dla CA.
* **Okres Ważności** jest oznaczony znacznikami **Nie Przed** i **Nie Po**, zapewniając, że certyfikat nie jest używany przed lub po określonej dacie.
* Sekcja **Klucza Publicznego**, kluczowa dla bezpieczeństwa certyfikatu, określa algorytm, rozmiar i inne techniczne szczegóły klucza publicznego.
* **Rozszerzenia x509v3** zwiększają funkcjonalność certyfikatu, określając **Użycie Klucza**, **Rozszerzone Użycie Klucza**, **Alternatywną Nazwę Podmiotu** i inne właściwości, aby dostroić zastosowanie certyfikatu.

#### **Użycie Klucza i Rozszerzenia**

* **Użycie Klucza** identyfikuje zastosowania kryptograficzne klucza publicznego, takie jak podpisywanie cyfrowe lub szyfrowanie klucza.
* **Rozszerzone Użycie Klucza** dalszo zawęża przypadki użycia certyfikatu, np. do uwierzytelniania serwera TLS.
* **Alternatywna Nazwa Podmiotu** i **Podstawowy Ograniczenie** definiują dodatkowe nazwy hostów objęte certyfikatem i czy jest to certyfikat CA czy jednostki końcowej.
* Identyfikatory, takie jak **Identyfikator Klucza Podmiotu** i **Identyfikator Klucza Władzy**, zapewniają unikalność i możliwość śledzenia kluczy.
* **Dostęp do Informacji o Władzy** i **Punkty Dystrybucji Listy Unieważnień (CRL)** zapewniają ścieżki do weryfikacji wydającego CA i sprawdzenia statusu unieważnienia certyfikatu.
* **CT Precertificate SCTs** oferują dzienniki transparentności, kluczowe dla zaufania publicznego do certyfikatu.
```python
# Example of accessing and using x509 certificate fields programmatically:
from cryptography import x509
from cryptography.hazmat.backends import default_backend

# Load an x509 certificate (assuming cert.pem is a certificate file)
with open("cert.pem", "rb") as file:
cert_data = file.read()
certificate = x509.load_pem_x509_certificate(cert_data, default_backend())

# Accessing fields
serial_number = certificate.serial_number
issuer = certificate.issuer
subject = certificate.subject
public_key = certificate.public_key()

print(f"Serial Number: {serial_number}")
print(f"Issuer: {issuer}")
print(f"Subject: {subject}")
print(f"Public Key: {public_key}")
```
### **Różnica między punktami dystrybucji OCSP a listami CRL**

**OCSP** (**RFC 2560**) polega na współpracy klienta i respondera w celu sprawdzenia, czy certyfikat klucza publicznego został unieważniony, bez konieczności pobierania pełnej **CRL**. Ta metoda jest bardziej wydajna niż tradycyjna **CRL**, która zawiera listę unieważnionych numerów seryjnych certyfikatów, ale wymaga pobrania potencjalnie dużego pliku. Listy CRL mogą zawierać do 512 wpisów. Więcej szczegółów dostępnych jest [tutaj](https://www.arubanetworks.com/techdocs/ArubaOS%206\_3\_1\_Web\_Help/Content/ArubaFrameStyles/CertRevocation/About\_OCSP\_and\_CRL.htm).

### **Co to jest Transparentność Certyfikatów**

Transparentność Certyfikatów pomaga zwalczać zagrożenia związane z certyfikatami, zapewniając, że wydanie i istnienie certyfikatów SSL są widoczne dla właścicieli domen, CA i użytkowników. Jej cele to:

* Zapobieganie wydawaniu certyfikatów SSL dla domeny bez wiedzy właściciela domeny przez CA.
* Ustanowienie otwartego systemu audytu do śledzenia certyfikatów wydanych błędnie lub złośliwie.
* Ochrona użytkowników przed fałszywymi certyfikatami.

#### **Rejestry Certyfikatów**

Rejestry certyfikatów są publicznie audytowalnymi, tylko do odczytu rekordami certyfikatów, utrzymywanymi przez usługi sieciowe. Te rejestry dostarczają dowodów kryptograficznych do celów audytu. Zarówno organy wydające certyfikaty, jak i publiczność, mogą przesyłać certyfikaty do tych rejestrów lub zapytywać o weryfikację. Chociaż dokładna liczba serwerów rejestru nie jest ustalona, spodziewa się, że będzie ich mniej niż tysiąc na całym świecie. Te serwery mogą być niezależnie zarządzane przez CA, dostawców usług internetowych lub dowolną zainteresowaną jednostkę.

#### **Zapytanie**

Aby sprawdzić rejestry Transparentności Certyfikatów dla dowolnej domeny, odwiedź [https://crt.sh/](https://crt.sh).

## **Formaty**

### **Format PEM**

* Najczęściej używany format dla certyfikatów.
* Wymaga oddzielnych plików dla certyfikatów i kluczy prywatnych, zakodowanych w Base64 ASCII.
* Powszechne rozszerzenia: .cer, .crt, .pem, .key.
* Głównie używany przez serwery Apache i podobne.

### **Format DER**

* Format binarny certyfikatów.
* Brak instrukcji "BEGIN/END CERTIFICATE" znalezionych w plikach PEM.
* Powszechne rozszerzenia: .cer, .der.
* Często używany w platformach Java.

### **Format P7B/PKCS#7**

* Przechowywany w Base64 ASCII, z rozszerzeniami .p7b lub .p7c.
* Zawiera tylko certyfikaty i łańcuchy certyfikatów, pomijając klucz prywatny.
* Obsługiwany przez systemy Microsoft Windows i Java Tomcat.

### **Format PFX/P12/PKCS#12**

* Format binarny, który łączy certyfikaty serwera, certyfikaty pośrednie i klucze prywatne w jednym pliku.
* Rozszerzenia: .pfx, .p12.
* Głównie używany w systemie Windows do importu i eksportu certyfikatów.

### **Konwersje Formatów**

**Konwersje PEM** są istotne dla kompatybilności:

* **x509 do PEM**
```bash
openssl x509 -in certificatename.cer -outform PEM -out certificatename.pem
```
* **PEM na DER**
```bash
openssl x509 -outform der -in certificatename.pem -out certificatename.der
```
* **DER to PEM**
```bash
openssl x509 -inform der -in certificatename.der -out certificatename.pem
```
* **PEM na P7B**
```bash
openssl crl2pkcs7 -nocrl -certfile certificatename.pem -out certificatename.p7b -certfile CACert.cer
```
* **PKCS7 to PEM**
```bash
openssl pkcs7 -print_certs -in certificatename.p7b -out certificatename.pem
```
**Konwersje PFX** są kluczowe dla zarządzania certyfikatami w systemie Windows:

* **PFX na PEM**
```bash
openssl pkcs12 -in certificatename.pfx -out certificatename.pem
```
* **PFX do PKCS#8** wymaga dwóch kroków:
1. Konwertuj PFX do PEM
```bash
openssl pkcs12 -in certificatename.pfx -nocerts -nodes -out certificatename.pem
```
2. Konwertowanie PEM na PKCS8
```bash
openSSL pkcs8 -in certificatename.pem -topk8 -nocrypt -out certificatename.pk8
```
* **P7B do PFX** wymaga również dwóch poleceń:
1. Konwertuj P7B do CER
```bash
openssl pkcs7 -print_certs -in certificatename.p7b -out certificatename.cer
```
2. Konwertowanie plików CER i klucza prywatnego na PFX
```bash
openssl pkcs12 -export -in certificatename.cer -inkey privateKey.key -out certificatename.pfx -certfile cacert.cer
```
***

<figure><img src="../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

\
Użyj [**Trickest**](https://trickest.com/?utm_source=hacktricks&utm_medium=text&utm_campaign=ppc&utm_term=trickest&utm_content=certificates), aby łatwo budować i **automatyzować przepływy pracy** z wykorzystaniem najbardziej zaawansowanych narzędzi społecznościowych na świecie.\
Otrzymaj dostęp już dziś:

{% embed url="https://trickest.com/?utm_source=hacktricks&utm_medium=banner&utm_campaign=ppc&utm_content=certificates" %}

<details>

<summary><strong>Dowiedz się, jak hakować AWS od zera do bohatera z</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Inne sposoby wsparcia HackTricks:

* Jeśli chcesz zobaczyć swoją **firmę reklamowaną w HackTricks** lub **pobrać HackTricks w formacie PDF**, sprawdź [**PLANY SUBSKRYPCYJNE**](https://github.com/sponsors/carlospolop)!
* Zdobądź [**oficjalne gadżety PEASS & HackTricks**](https://peass.creator-spring.com)
* Odkryj [**Rodzinę PEASS**](https://opensea.io/collection/the-peass-family), naszą kolekcję ekskluzywnych [**NFT**](https://opensea.io/collection/the-peass-family)
* **Dołącz do** 💬 [**grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegramowej**](https://t.me/peass) lub **śledź** nas na **Twitterze** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podziel się swoimi sztuczkami hakerskimi, przesyłając PR-y do** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
