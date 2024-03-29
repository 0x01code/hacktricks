# Сертифікати

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію в рекламі на HackTricks** або **завантажити HackTricks у PDF** Перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв GitHub.

</details>

<figure><img src="../.gitbook/assets/image (3) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Використовуйте [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) для легкої побудови та **автоматизації робочих процесів** за допомогою найбільш **продвинутих** інструментів спільноти у світі.\
Отримайте доступ сьогодні:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## Що таке сертифікат

**Сертифікат публічного ключа** - це цифровий ідентифікатор, який використовується в криптографії для підтвердження того, що хтось володіє публічним ключем. Він містить деталі ключа, ідентичність власника (суб'єкт) та цифровий підпис від довіреної авторитетності (видавця). Якщо програмне забезпечення довіряє видавцю та підпис є дійсним, можлива безпечна комунікація з власником ключа.

Сертифікати в основному видавати [сертифікатними органами](https://en.wikipedia.org/wiki/Certificate\_authority) (CAs) в налаштуванні [інфраструктури з відкритим ключем](https://en.wikipedia.org/wiki/Public-key\_infrastructure) (PKI). Іншим методом є [мережа довіри](https://en.wikipedia.org/wiki/Web\_of\_trust), де користувачі безпосередньо перевіряють ключі один одного. Загальним форматом для сертифікатів є [X.509](https://en.wikipedia.org/wiki/X.509), який може бути адаптований для конкретних потреб, як описано в RFC 5280.

## Загальні поля x509

### **Загальні поля в сертифікатах x509**

У сертифікатах x509 кілька **полів** відіграють важливу роль у забезпеченні валідності та безпеки сертифіката. Ось розбір цих полів:

* **Номер версії** вказує версію формату x509.
* **Серійний номер** унікально ідентифікує сертифікат у системі Сертифікаційного органу (CA), головним чином для відстеження відкликання.
* Поле **Суб'єкт** представляє власника сертифіката, яким може бути машина, фізична особа або організація. Воно включає детальну ідентифікацію, таку як:
* **Загальне ім'я (CN)**: Домени, які охоплюються сертифікатом.
* **Країна (C)**, **Місцевість (L)**, **Штат або провінція (ST, S, або P)**, **Організація (O)** та **Організаційна одиниця (OU)** надають географічні та організаційні відомості.
* **Виділений ім'я (DN)** укладає повну ідентифікацію суб'єкта.
* **Видавець** вказує, хто перевірив та підписав сертифікат, включаючи схожі підполя, як у Суб'єкта для CA.
* **Період валідності** позначений мітками **Не раніше** та **Не пізніше**, що гарантує, що сертифікат не використовується до або після певної дати.
* Розділ **Публічний ключ**, важливий для безпеки сертифіката, вказує алгоритм, розмір та інші технічні деталі публічного ключа.
* **Розширення x509v3** покращують функціональність сертифіката, вказуючи **Використання ключа**, **Розширене використання ключа**, **Альтернативне ім'я суб'єкта** та інші властивості для налаштування застосування сертифіката.

#### **Використання ключа та розширення**

* **Використання ключа** ідентифікує криптографічні застосування публічного ключа, такі як цифровий підпис або шифрування ключа.
* **Розширене використання ключа** додатково уточнює випадки використання сертифіката, наприклад, для аутентифікації сервера TLS.
* **Альтернативне ім'я суб'єкта** та **Основний обмеження** визначають додаткові імена хостів, які охоплюються сертифікатом, та чи це сертифікат CA чи кінцевого суб'єкта, відповідно.
* Ідентифікатори, такі як **Ідентифікатор ключа суб'єкта** та **Ідентифікатор ключа авторитету**, забезпечують унікальність та відстежуваність ключів.
* **Доступ до інформації про авторитет** та **Точки розподілу списків відкликання** надають шляхи для перевірки видавця CA та перевірки статусу відкликання сертифіката.
* **CT Precertificate SCTs** пропонують журнали прозорості, важливі для громадської довіри до сертифіката.
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
### **Різниця між точками розподілу OCSP та CRL**

**OCSP** (**RFC 2560**) передбачає співпрацю клієнта та відповідача для перевірки скасування цифрового сертифіката з відкритим ключем без необхідності завантажувати повний **CRL**. Цей метод ефективніший, ніж традиційний **CRL**, який надає список скасованих серійних номерів сертифікатів, але вимагає завантаження потенційно великого файлу. CRL може містити до 512 записів. Додаткові відомості доступні [тут](https://www.arubanetworks.com/techdocs/ArubaOS%206\_3\_1\_Web\_Help/Content/ArubaFrameStyles/CertRevocation/About\_OCSP\_and\_CRL.htm).

### **Що таке Transparent Certificate Transparency**

Transparent Certificate Transparency допомагає боротися з загрозами, пов'язаними з сертифікатами, забезпечуючи видимість видачі та існування SSL-сертифікатів власникам доменів, CAs та користувачам. Його цілі:

* Запобігання CAs видавати SSL-сертифікати для домену без відома власника домену.
* Створення відкритої системи аудиту для відстеження помилково виданих або зловживаних сертифікатів.
* Захист користувачів від шахрайських сертифікатів.

#### **Логи сертифікатів**

Логи сертифікатів є публічно перевіряемими, додаванням записів про сертифікати, які підтримуються мережевими службами. Ці логи надають криптографічні докази для аудиту. Як видачі організації, так і громадськість можуть надсилати сертифікати в ці логи або запитувати їх для перевірки. Хоча точна кількість серверів логу не фіксується, очікується, що їх буде менше тисячі глобально. Ці сервери можуть бути незалежно керовані CAs, постачальниками послуг Інтернету або будь-якою зацікавленою стороною.

#### **Запит**

Щоб дослідити логи Transparent Certificate Transparency для будь-якого домену, відвідайте [https://crt.sh/](https://crt.sh).

## **Формати**

### **Формат PEM**

* Найбільш поширений формат для сертифікатів.
* Вимагає окремих файлів для сертифікатів та приватних ключів, закодованих у Base64 ASCII.
* Загальні розширення: .cer, .crt, .pem, .key.
* В основному використовується Apache та подібними серверами.

### **Формат DER**

* Бінарний формат сертифікатів.
* Відсутні "ПОЧАТОК/КІНЕЦЬ СЕРТИФІКАТА" у заявках PEM.
* Загальні розширення: .cer, .der.
* Часто використовується з платформами Java.

### **Формат P7B/PKCS#7**

* Зберігається у Base64 ASCII, з розширеннями .p7b або .p7c.
* Містить лише сертифікати та ланцюжкові сертифікати, за винятком приватного ключа.
* Підтримується Microsoft Windows та Java Tomcat.

### **Формат PFX/P12/PKCS#12**

* Бінарний формат, який упаковує серверні сертифікати, проміжні сертифікати та приватні ключі в один файл.
* Розширення: .pfx, .p12.
* Головним чином використовується в Windows для імпорту та експорту сертифікатів.

### **Конвертація форматів**

**Конвертації PEM** є важливими для сумісності:

* **x509 до PEM**
```bash
openssl x509 -in certificatename.cer -outform PEM -out certificatename.pem
```
* **PEM у DER**
```bash
openssl x509 -outform der -in certificatename.pem -out certificatename.der
```
* **DER у PEM**
```bash
openssl x509 -inform der -in certificatename.der -out certificatename.pem
```
* **PEM у P7B**
```bash
openssl crl2pkcs7 -nocrl -certfile certificatename.pem -out certificatename.p7b -certfile CACert.cer
```
* **PKCS7 у PEM**
```bash
openssl pkcs7 -print_certs -in certificatename.p7b -out certificatename.pem
```
**Конвертація PFX** є важливою для керування сертифікатами в Windows:

* **PFX в PEM**
```bash
openssl pkcs12 -in certificatename.pfx -out certificatename.pem
```
* **PFX до PKCS#8** включає два кроки:
1. Конвертувати PFX у PEM
```bash
openssl pkcs12 -in certificatename.pfx -nocerts -nodes -out certificatename.pem
```
2. Конвертувати PEM у PKCS8
```bash
openSSL pkcs8 -in certificatename.pem -topk8 -nocrypt -out certificatename.pk8
```
* **P7B в PFX** також потребує дві команди:
1. Конвертувати P7B в CER
```bash
openssl pkcs7 -print_certs -in certificatename.p7b -out certificatename.cer
```
2. Конвертувати CER та приватний ключ в PFX
```bash
openssl pkcs12 -export -in certificatename.cer -inkey privateKey.key -out certificatename.pfx -certfile cacert.cer
```
***

<figure><img src="../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Використовуйте [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) для легкої побудови та **автоматизації робочих процесів**, які працюють за допомогою найбільш **продвинутих** інструментів у спільноті.\
Отримайте доступ сьогодні:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити свою **компанію рекламовану на HackTricks** або **завантажити HackTricks у PDF-форматі**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) **та** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **репозиторіїв на GitHub**.

</details>
