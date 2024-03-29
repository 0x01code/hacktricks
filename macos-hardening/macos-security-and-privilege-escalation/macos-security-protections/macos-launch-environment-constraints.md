# Обмеження запуску/середовища macOS та кеш довіри

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Ви працюєте в **кібербезпецівій компанії**? Хочете побачити **рекламу вашої компанії на HackTricks**? або хочете мати доступ до **останньої версії PEASS або завантажити HackTricks у форматі PDF**? Перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Дізнайтеся про [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* **Приєднуйтесь до** [**💬**](https://emojipedia.org/speech-balloon/) [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за мною на **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**репозиторію hacktricks**](https://github.com/carlospolop/hacktricks) **та** [**репозиторію hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud)
*
* .

</details>

## Основна інформація

Обмеження запуску в macOS були введені для підвищення безпеки шляхом **регулювання того, як, хто і звідки може бути ініційований процес**. Запроваджені в macOS Ventura, вони надають фреймворк, який категоризує **кожний системний бінарний файл у різні категорії обмежень**, які визначені в **кеші довіри**, список, що містить системні бінарні файли та їх відповідні хеші. Ці обмеження поширюються на кожний виконуваний бінарний файл у системі, включаючи набір **правил**, які визначають вимоги для **запуску певного бінарного файлу**. Правила охоплюють власні обмеження, які повинен задовольняти бінарний файл, обмеження батьків, які повинні бути виконані батьківським процесом, та обмеження відповідальності, які повинні дотримуватися іншими відповідними сутностями.

Механізм поширюється на сторонні додатки через **Обмеження середовища**, починаючи з macOS Sonoma, що дозволяє розробникам захищати свої додатки, вказуючи **набір ключів та значень для обмежень середовища**.

Ви визначаєте **обмеження запуску та бібліотек** в словниках обмежень, які ви зберігаєте в **файлах властивостей `launchd`**, або в **окремих файлах властивостей**, які ви використовуєте при підписанні коду.

Існує 4 типи обмежень:

* **Власні обмеження**: Обмеження, які застосовуються до **виконуваного** бінарного файлу.
* **Батьківський процес**: Обмеження, які застосовуються до **батька процесу** (наприклад **`launchd`**, який запускає службу XP)
* **Обмеження відповідальності**: Обмеження, які застосовуються до **процесу, який викликає службу** у комунікації XPC
* **Обмеження завантаження бібліотек**: Використовуйте обмеження завантаження бібліотек для селективного опису коду, який може бути завантажений

Таким чином, коли процес намагається запустити інший процес — викликаючи `execve(_:_:_:)` або `posix_spawn(_:_:_:_:_:_:)` — операційна система перевіряє, що **виконуваний** файл **задовольняє** своє **власне обмеження**. Вона також перевіряє, що **виконуваний файл батьківського процесу задовольняє обмеження батьківського процесу**, і що **виконуваний файл відповідального процесу задовольняє обмеження відповідального процесу**. Якщо будь-які з цих обмежень запуску не виконуються, операційна система не запускає програму.

Якщо при завантаженні бібліотеки будь-яка частина **обмеження бібліотеки не виконується**, ваш процес **не завантажує** бібліотеку.

## Категорії LC

LC складається з **фактів** та **логічних операцій** (і, або..), які комбінують факти.

[**Факти, які може використовувати LC, задокументовані**](https://developer.apple.com/documentation/security/defining\_launch\_environment\_and\_library\_constraints). Наприклад:

* is-init-proc: Булеве значення, яке вказує, чи має виконуваний файл бути процесом ініціалізації операційної системи (`launchd`).
* is-sip-protected: Булеве значення, яке вказує, чи має виконуваний файл бути файлом, захищеним Системою Цілісності (SIP).
* `on-authorized-authapfs-volume:` Булеве значення, яке вказує, чи операційна система завантажила виконуваний файл з авторизованого, аутентифікованого тому APFS.
* `on-authorized-authapfs-volume`: Булеве значення, яке вказує, чи операційна система завантажила виконуваний файл з авторизованого, аутентифікованого тому APFS.
* Cryptexes volume
* `on-system-volume:` Булеве значення, яке вказує, чи операційна система завантажила виконуваний файл з поточного завантаженого тому системи.
* Всередині /System...
* ...

Коли Apple бінарний файл підписаний, він **призначає його категорії LC** всередині **кешу довіри**.

* **16 категорій LC для iOS** були [**розгорнуті та задокументовані тут**](https://gist.github.com/LinusHenze/4cd5d7ef057a144cda7234e2c247c056).
* Поточні **категорії LC (macOS 14** - Somona) були розгорнуті, і їх [**опис можна знайти тут**](https://gist.github.com/theevilbit/a6fef1e0397425a334d064f7b6e1be53).

Наприклад, Категорія 1:
```
Category 1:
Self Constraint: (on-authorized-authapfs-volume || on-system-volume) && launch-type == 1 && validation-category == 1
Parent Constraint: is-init-proc
```
* `(on-authorized-authapfs-volume || on-system-volume)`: Повинно бути у томі System або Cryptexes.
* `launch-type == 1`: Повинно бути системним сервісом (plist у LaunchDaemons).
* `validation-category == 1`: Виконуваний файл операційної системи.
* `is-init-proc`: Launchd

### Реверсінг LC Категорій

Ви можете отримати більше інформації [**тут**](https://theevilbit.github.io/posts/launch\_constraints\_deep\_dive/#reversing-constraints), але в основному вони визначені в **AMFI (AppleMobileFileIntegrity)**, тому вам потрібно завантажити Набір розробки ядра, щоб отримати **KEXT**. Символи, що починаються з **`kConstraintCategory`**, є **цікавими**. Видобуваючи їх, ви отримаєте закодований потік DER (ASN.1), який потрібно розкодувати за допомогою [ASN.1 Decoder](https://holtstrom.com/michael/tools/asn1decoder.php) або бібліотеки python-asn1 та її сценарію `dump.py`, [andrivet/python-asn1](https://github.com/andrivet/python-asn1/tree/master), яка надасть вам більш зрозумілий рядок.

## Обмеження середовища

Це обмеження запуску, налаштоване в **додатках сторонніх розробників**. Розробник може вибрати **факти** та **логічні операнди для використання** у своєму додатку для обмеження доступу до нього.

Можливо перерахувати Обмеження середовища додатка за допомогою:
```bash
codesign -d -vvvv app.app
```
## Кеш довіри

У **macOS** є кілька кешів довіри:

* **`/System/Volumes/Preboot/*/boot/*/usr/standalone/firmware/FUD/BaseSystemTrustCache.img4`**
* **`/System/Volumes/Preboot/*/boot/*/usr/standalone/firmware/FUD/StaticTrustCache.img4`**
* **`/System/Library/Security/OSLaunchPolicyData`**

А в iOS виглядає, що він знаходиться в **`/usr/standalone/firmware/FUD/StaticTrustCache.img4`**.

{% hint style="warning" %}
На macOS, що працює на пристроях Apple Silicon, якщо підписаний Apple бінарний файл не знаходиться в кеші довіри, AMFI відмовиться його завантажувати.
{% endhint %}

### Перелік кешів довіри

Попередні файли кешу довіри мають формат **IMG4** та **IM4P**, де IM4P - це розділ навантаження формату IMG4.

Ви можете використовувати [**pyimg4**](https://github.com/m1stadev/PyIMG4), щоб видобути навантаження баз даних:

{% code overflow="wrap" %}
```bash
# Installation
python3 -m pip install pyimg4

# Extract payloads data
cp /System/Volumes/Preboot/*/boot/*/usr/standalone/firmware/FUD/BaseSystemTrustCache.img4 /tmp
pyimg4 img4 extract -i /tmp/BaseSystemTrustCache.img4 -p /tmp/BaseSystemTrustCache.im4p
pyimg4 im4p extract -i /tmp/BaseSystemTrustCache.im4p -o /tmp/BaseSystemTrustCache.data

cp /System/Volumes/Preboot/*/boot/*/usr/standalone/firmware/FUD/StaticTrustCache.img4 /tmp
pyimg4 img4 extract -i /tmp/StaticTrustCache.img4 -p /tmp/StaticTrustCache.im4p
pyimg4 im4p extract -i /tmp/StaticTrustCache.im4p -o /tmp/StaticTrustCache.data

pyimg4 im4p extract -i /System/Library/Security/OSLaunchPolicyData -o /tmp/OSLaunchPolicyData.data
```
{% endcode %}

(Іншою опцією може бути використання інструменту [**img4tool**](https://github.com/tihmstar/img4tool), який буде працювати навіть в M1, навіть якщо версія застаріла, і для x86\_64, якщо ви встановите його в відповідні місця).

Тепер ви можете використовувати інструмент [**trustcache**](https://github.com/CRKatri/trustcache), щоб отримати інформацію у зручному форматі:
```bash
# Install
wget https://github.com/CRKatri/trustcache/releases/download/v2.0/trustcache_macos_arm64
sudo mv ./trustcache_macos_arm64 /usr/local/bin/trustcache
xattr -rc /usr/local/bin/trustcache
chmod +x /usr/local/bin/trustcache

# Run
trustcache info /tmp/OSLaunchPolicyData.data | head
trustcache info /tmp/StaticTrustCache.data | head
trustcache info /tmp/BaseSystemTrustCache.data | head

version = 2
uuid = 35EB5284-FD1E-4A5A-9EFB-4F79402BA6C0
entry count = 969
0065fc3204c9f0765049b82022e4aa5b44f3a9c8 [none] [2] [1]
00aab02b28f99a5da9b267910177c09a9bf488a2 [none] [2] [1]
0186a480beeee93050c6c4699520706729b63eff [none] [2] [2]
0191be4c08426793ff3658ee59138e70441fc98a [none] [2] [3]
01b57a71112235fc6241194058cea5c2c7be3eb1 [none] [2] [2]
01e6934cb8833314ea29640c3f633d740fc187f2 [none] [2] [2]
020bf8c388deaef2740d98223f3d2238b08bab56 [none] [2] [3]
```
Довірний кеш має наступну структуру, тому **категорія LC є четвертим стовпцем**
```c
struct trust_cache_entry2 {
uint8_t cdhash[CS_CDHASH_LEN];
uint8_t hash_type;
uint8_t flags;
uint8_t constraintCategory;
uint8_t reserved0;
} __attribute__((__packed__));
```
Потім ви можете використати скрипт, такий як [**цей**](https://gist.github.com/xpn/66dc3597acd48a4c31f5f77c3cc62f30), щоб витягти дані.

З цих даних ви можете перевірити додатки з **значенням обмежень запуску `0`**, які не обмежені ([**перевірте тут**](https://gist.github.com/LinusHenze/4cd5d7ef057a144cda7234e2c247c056) для кожного значення).

## Заходи проти атак

Обмеження запуску б врегулювало кілька старих атак, **переконуючись, що процес не буде виконаний в неочікуваних умовах:** наприклад, з неочікуваних місць або викликаний неочікуваним батьківським процесом (якщо лише launchd має запускати його).

Більше того, обмеження запуску також **перешкоджає атакам на зниження рівня.**

Однак вони **не захищають від поширених зловживань XPC**, **внедрень коду Electron** або **внедрень dylib** без перевірки бібліотек (якщо не відомі ідентифікатори команд, які можуть завантажувати бібліотеки).

### Захист від служб XPC Daemon

У випуску Sonoma помітним пунктом є **конфігурація відповідальності служби XPC демона**. Служба XPC відповідає за себе, на відміну від клієнта, який підключається. Це документовано в звіті про зворотній зв'язок FB13206884. Ця настройка може здатися недосконалою, оскільки вона дозволяє певні взаємодії зі службою XPC:

- **Запуск служби XPC**: Якщо припустити, що це помилка, ця настройка не дозволяє ініціювати службу XPC через код атакувальника.
- **Підключення до активної служби**: Якщо служба XPC вже працює (можливо, активована її початковим додатком), немає перешкод для підключення до неї.

Хоча впровадження обмежень на службу XPC може бути корисним, **звужуючи вікно для потенційних атак**, це не вирішує основну проблему. Забезпечення безпеки служби XPC фундаментально вимагає **ефективної перевірки підключеного клієнта**. Це залишається єдиним методом зміцнення безпеки служби. Також варто зазначити, що згадана конфігурація відповідальності наразі працює, що може не відповідати задуманому дизайну.


### Захист від Electron

Навіть якщо вимагається, щоб додаток був **відкритий за допомогою LaunchService** (в обмеженнях батьків). Це можна досягти за допомогою **`open`** (який може встановлювати змінні середовища) або використовуючи **API служб запуску** (де можна вказати змінні середовища).

## Посилання

* [https://youtu.be/f1HA5QhLQ7Y?t=24146](https://youtu.be/f1HA5QhLQ7Y?t=24146)
* [https://theevilbit.github.io/posts/launch\_constraints\_deep\_dive/](https://theevilbit.github.io/posts/launch\_constraints\_deep\_dive/)
* [https://eclecticlight.co/2023/06/13/why-wont-a-system-app-or-command-tool-run-launch-constraints-and-trust-caches/](https://eclecticlight.co/2023/06/13/why-wont-a-system-app-or-command-tool-run-launch-constraints-and-trust-caches/)
* [https://developer.apple.com/videos/play/wwdc2023/10266/](https://developer.apple.com/videos/play/wwdc2023/10266/)

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Ви працюєте в **кібербезпеці**? Хочете побачити вашу **компанію в рекламі на HackTricks**? або хочете мати доступ до **останньої версії PEASS або завантажити HackTricks у PDF**? Перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Дізнайтеся про [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* **Приєднуйтесь до** [**💬**](https://emojipedia.org/speech-balloon/) [**групи Discord**](https://discord.gg/hRep4RUj7f) або групи в **телеграмі** або **слідкуйте** за мною в **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**репозиторію hacktricks**](https://github.com/carlospolop/hacktricks) **та** [**репозиторію hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud)
*
* .

</details>
