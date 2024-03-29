# Фізичні атаки

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити **рекламу вашої компанії на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв.

</details>

## Відновлення пароля BIOS та безпека системи

Скидання налаштувань **BIOS** можна виконати кількома способами. Більшість материнських плат мають **батарею**, яку, вийнявши протягом **30 хвилин**, скине налаштування BIOS, включаючи пароль. Альтернативно, за допомогою **перемички на материнській платі** можна скинути ці налаштування, з'єднуючи конкретні контакти.

У випадках, коли апаратні налаштування неможливі або непрактичні, **програмні інструменти** пропонують рішення. Запуск системи з **Live CD/USB** з дистрибутивами, такими як **Kali Linux**, надає доступ до інструментів, таких як **_killCmos_** та **_CmosPWD_**, які допоможуть у відновленні пароля BIOS.

У випадках, коли пароль BIOS невідомий, введення його неправильно **три рази** зазвичай призводить до помилкового коду. Цей код можна використовувати на веб-сайтах, наприклад [https://bios-pw.org](https://bios-pw.org), для можливого отримання використовуваного пароля.

### Безпека UEFI

Для сучасних систем, які використовують **UEFI** замість традиційного BIOS, можна використовувати інструмент **chipsec** для аналізу та зміни налаштувань UEFI, включаючи вимкнення **Secure Boot**. Це можна зробити за допомогою наступної команди:

`python chipsec_main.py -module exploits.secure.boot.pk`

### Аналіз RAM та атаки Cold Boot

RAM зберігає дані короткий час після відключення живлення, зазвичай протягом **1-2 хвилин**. Цю стійкість можна продовжити до **10 хвилин**, застосовуючи холодні речовини, такі як рідкий азот. Протягом цього розширеного періоду можна створити **дамп пам'яті** за допомогою інструментів, таких як **dd.exe** та **volatility** для аналізу.

### Атаки на прямий доступ до пам'яті (DMA)

**INCEPTION** - це інструмент, призначений для **фізичного впливу на пам'ять** через DMA, сумісний з інтерфейсами, такими як **FireWire** та **Thunderbolt**. Він дозволяє обійти процедури входу, патчуючи пам'ять для прийняття будь-якого пароля. Однак він неефективний проти систем **Windows 10**.

### Live CD/USB для доступу до системи

Зміна системних бінарних файлів, таких як **_sethc.exe_** або **_Utilman.exe_** на копію **_cmd.exe_**, може забезпечити командний рядок з привілеями системи. Інструменти, такі як **chntpw**, можна використовувати для редагування файлу **SAM** у встановленні Windows, що дозволяє змінювати паролі.

**Kon-Boot** - це інструмент, який спрощує вхід в системи Windows без знання пароля шляхом тимчасової модифікації ядра Windows або UEFI. Додаткову інформацію можна знайти на [https://www.raymond.cc](https://www.raymond.cc/blog/login-to-windows-administrator-and-linux-root-account-without-knowing-or-changing-current-password/).

### Обробка функцій безпеки Windows

#### Ярлики завантаження та відновлення

- **Supr**: Доступ до налаштувань BIOS.
- **F8**: Увійти в режим відновлення.
- Натискання **Shift** після банера Windows може обійти автовход.

#### Пристрої BAD USB

Пристрої, такі як **Rubber Ducky** та **Teensyduino**, служать платформами для створення **поганих USB** пристроїв, здатних виконувати попередньо визначені завдання при підключенні до цільового комп'ютера.

#### Копіювання тіньового образу тому

Привілеї адміністратора дозволяють створювати копії чутливих файлів, включаючи файл **SAM**, за допомогою PowerShell.

### Обхід шифрування BitLocker

Шифрування BitLocker можна обійти, якщо **пароль відновлення** знайдено у файлі дампу пам'яті (**MEMORY.DMP**). Для цього можна використовувати інструменти, такі як **Elcomsoft Forensic Disk Decryptor** або **Passware Kit Forensic**.

### Соціальна інженерія для додавання ключа відновлення

Новий ключ відновлення BitLocker можна додати за допомогою тактик соціальної інженерії, переконуючи користувача виконати команду, яка додає новий ключ відновлення, складений з нулів, тим самим спрощуючи процес розшифрування. 

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити **рекламу вашої компанії на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв.

</details>
