# Системні розширення macOS

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію рекламовану в HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Дізнайтеся про [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіть** свої хакінг-прийоми, надсилайте PR до **HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github репозиторіїв.

</details>

## Системні розширення / Фреймворк безпеки кінцевої точки

На відміну від Ядерних розширень, **Системні розширення працюють в просторі користувача** замість простору ядра, що зменшує ризик аварії системи через несправність розширення.

<figure><img src="../../../.gitbook/assets/image (1) (3) (1) (1).png" alt="https://knight.sc/images/system-extension-internals-1.png"><figcaption></figcaption></figure>

Існує три типи системних розширень: Розширення **DriverKit**, Розширення **Network** та Розширення **Безпеки кінцевої точки**.

### **Розширення DriverKit**

DriverKit є заміною для ядерних розширень, які **забезпечують підтримку апаратного забезпечення**. Воно дозволяє драйверам пристроїв (наприклад, USB, Serial, NIC та HID драйверам) працювати в просторі користувача, а не в просторі ядра. Фреймворк DriverKit включає **версії класів I/O Kit для простору користувача**, і ядро пересилає звичайні події I/O Kit в простір користувача, пропонуючи безпечне середовище для роботи цих драйверів.

### **Розширення Network**

Розширення Network надають можливість налаштування мережевих поведінок. Існує кілька типів Розширень Network:

* **App Proxy**: Використовується для створення клієнта VPN, який реалізує протокол VPN, орієнтований на потік. Це означає, що він обробляє мережевий трафік на основі з'єднань (або потоків), а не окремих пакетів.
* **Packet Tunnel**: Використовується для створен цлієнта VPN, який реалізує протокол VPN, орієнтований на пакети. Це означає, що він обробляє мережевий трафік на основі окремих пакетів.
* **Filter Data**: Використовується для фільтрації мережевих "потоків". Він може відстежувати або змінювати мережеві дані на рівні потоку.
* **Filter Packet**: Використовується для фільтрації окремих мережевих пакетів. Він може відстежувати або змінювати мережеві дані на рівні пакету.
* **DNS Proxy**: Використовується для створення власного постачальника DNS. Він може використовуватися для відстеження або зміни запитів та відповідей DNS.

## Фреймворк безпекennoсті кінцевої точки

Endpoint Security - це фреймворк, наданий Apple в macOS, який надає набір API для системної безпеки. Він призначений для використання **вендорами безпеки та розробниками **для створення продуктів, які можуть відстежувати та контролювати діяльність системи** для виявлення та захисту від зловмисної діяльності.

Цей фреймворк надає **колекцію API для відстеження та контролю діяльності системи**, таких як виконання процесів, події файлової системи, мережеві та ядерні події.

Основа цього фреймворку реалізована в ядрі, як Ядерне розширення (KEXT), розташоване в **`/System/Library/Extensions/EndpointSecurity.kext`**. Це KEXT складається з кількох ключових компонентів:

* **EndpointSecurityDriver**: Він діє як "точка входу" для ядерного розширення. Це основна точка взаємодії між ОС та фреймворком безпеки кінцевої точки.
* **EndpointSecurityEventManager**: Цей компонент відповідає за реалізацію ядерних гуків. Ядерні гуки дозволяють фреймворку відстежувати системні події, перехоплюючи системні виклики.
* **EndpointSecurityClientManager**: Цей компонент керує комунікацією з клієнтами простору користувача, відстежує, які клієнти підключені та потребують отримувати сповіщення про події.
 і **EndpointSecurityMessageManager**: Він відправляє повідомлення та сповіщення про події клієнтам простору користувача.

Події, які може відстежу **фреймворк безпеки кінцевої точки**, поділяються на категорії:

* Події файлів
* Події процесів
* Події сокетів
* Ядерні події (такі як завантаження/вивантаження ядерного розширення або відкриття пристрою I/O Kit)

### Архітектура фреймворку безпеки кіexнвої точки

<figure><img src="../../../.gitbook/assets/image (3) (8).png" alt="https://www.youtube.com/watch?v=jaVkpM1UqOs"><figcaption></figcaption></figure>

**Комунікація простору користувача** з фреймворком безпеки кінцевої точки відбувається через клас IOUserClient. Використовуються два різні підкласи, залежно від типу викликача:

* **EndpointSecurityDriverClient**: Це вимагає дозволу `com.apple.private.endpoint-security.manager`, який має лише процес системи `endpointsecurityd`.
* **EndpointSecurityExternalClient**: Це вимагає дозволу `com.apple.developer.endpoint-security.client`. Це зазвичай використовується стороннім програмним забезпеченням безпеки, яке потребує взаємодії з фреймворком безпеки кінцевої точки.

Розширення безпеки кінцевої точки: **`libEndpointSecurity.dylib`** - це бібліотека С, яку використовують системні розширення для взаємодії з ядром. Ця бібліотека використовує I/O Kit (`IOKit`) для взаємодії з Ядерним р Fameами theами theуинс ** ** вап рsnесsnедs Endpoint ** Endpoint libредs ** theуин ки`).esies events) ** Endpoint lib theуun шstoурsнббі theууин киiniumрingsад ** Endpoint lib Endpoint lib theу).es).ingен theousп foringодs theуин ** Endpoint relevant foringедsments example the. Endpoints the. endpoint you the us ** Endpoint analysis foringед ** me Endpoint URL").sments example the. Endpoint checks a the. Endpointsments. **`libEndpointSecurity.dylib`** is the C library that system extensions use to communicate with the kernel. This library uses the I/O Kit (`...`. This library uses the I/O Kit (`IOKit`) to communicate with the Endpoint Security KEXT.

**`endpointsecurityd`** is a key system daemon involved in managing and launching endpoint security...`. This library uses the I/O Kit (`IOKit`) to communicate with the Endpoint Security KEXT.

**`endpointsecurityd`** is a key system daemon involved in managing and launching endpoint security system extensions, particularly during the early boot process...
**`endpointsecurityd`** is a key system daemon involved in managing and launching endpoint security system extensions, particularly during the early boot process. **Only system extensions** marked with **`NSEndpointSecurityEarlyBoot`** in their `Info.plist` file receive this early boot treatment.

Another system daemon, **`sysextd`**, **validates system extensions** and moves them into the proper system locations. It    ...`. This library uses the I/O Kit (`IOKit`) to communicate with the Endpoint Security KEXT.

**`endpointsecurityd`** is a key system daemon involved in managing and launching endpoint security system extensions, particularly during the early boot process...
**`endpointsecurityd`** is a key system daemon involved in managing and launching endpoint security system extensions, particularly during the early boot process. **Only system extensions** marked with **`NSEndpointSecurityEarlyBoot`** in their `Info.plist` file receive this early boot treatment.

Another system daemon, **`sysextd`**, **validates system extensions** and moves them into the proper system locations. It then asks the relevant daemon to load the extension. The **`SystemExtensions.framework`** is responsible for activating and deactivating system extensions.

## Обхід ESF

ESF використовується безпековими інструментами, які спробують виявити червоного командира, тому будь-яка інформація про те, як цього уникнути, звучить цікаво.

### CVE-2021-30965

Справа в тому, що додатку безпеки потрібно мати **повний доступ до диска**. Тому, якщо зловмисник може це видалити, він може запобігти запуску програмного забезпечення:
```bash
tccutil reset All
```
Для **додаткової інформації** про цей обхід та пов'язані з ним перевірте виступ [#OBTS v5.0: "The Achilles Heel of EndpointSecurity" - Fitzl Csaba](https://www.youtube.com/watch?v=lQO7tvNCoTI)

В кінці це було виправлено, надавши новий дозвіл **`kTCCServiceEndpointSecurityClient`** додатку безпеки, керованому **`tccd`**, щоб `tccutil` не очищав його дозволи, запобігаючи його запуску.

## Посилання

* [**OBTS v3.0: "Endpoint Security & Insecurity" - Scott Knight**](https://www.youtube.com/watch?v=jaVkpM1UqOs)
* [**https://knight.sc/reverse%20engineering/2019/08/24/system-extension-internals.html**](https://knight.sc/reverse%20engineering/2019/08/24/system-extension-internals.html)

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію рекламовану в HackTricks** або **завантажити HackTricks у PDF** Перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приє2021-10-14єднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв GitHub.

</details>
