# Захист macOS

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію рекламовану на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Дізнайтеся про [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>

## Gatekeeper

Gatekeeper зазвичай використовується для посилання на комбінацію **Quarantine + Gatekeeper + XProtect**, 3 модулі безпеки macOS, які спробують **запобігти виконанню потенційно шкідливого програмного забезпечення, завантаженого користувачами**.

Додаткова інформація:

{% content-ref url="macos-gatekeeper.md" %}
[macos-gatekeeper.md](macos-gatekeeper.md)
{% endcontent-ref %}

## Обмеження процесів

### SIP - Захист системної цілісності

{% content-ref url="macos-sip.md" %}
[macos-sip.md](macos-sip.md)
{% endcontent-ref %}

### Пісочниця

Пісочниця macOS **обмежує додатки**, що працюють у пісочниці, до **дій, дозволених в профілі пісочниці**, з яким працює додаток. Це допомагає забезпечити, що **додаток буде отримувати доступ лише до очікуваних ресурсів**.

{% content-ref url="macos-sandbox/" %}
[macos-sandbox](macos-sandbox/)
{% endcontent-ref %}

### TCC - **Прозорість, Згода та Контроль**

**TCC (Прозорість, Згода та Контроль)** - це система безпеки. Вона призначена для **управління дозволами** додатків, зокрема шляхом регулювання їх доступу до чутливих функцій. Це включає елементи, такі як **сервіси місцезнаходження, контакти, фотографії, мікрофон, камера, доступність та повний доступ до диска**. TCC забезпечує, що додатки можуть отримати доступ до цих функцій лише після отримання явної згоди користувача, тим самим зміцнюючи конфіденційність та контроль над особистими даними.

{% content-ref url="macos-tcc/" %}
[macos-tcc](macos-tcc/)
{% endcontent-ref %}

### Обмеження запуску/середовища та кеш довіри

Обмеження запуску в macOS - це функція безпеки для **регулювання ініціювання процесу**, визначаючи, **хто може запустити** процес, **як**, і **звідки**. Введені в macOS Ventura, вони категоризують системні бінарники в межах **кешу довіри**. Кожний виконуваний бінарний файл має встановлені **правила** для його **запуску**, включаючи **сам**, **батьківські** та **відповідальні** обмеження. Розширені до сторонніх додатків як **Обмеження середовища** в macOS Sonoma, ці функції допомагають пом'якшити можливі експлуатації системи, керуючи умовами запуску процесу.

{% content-ref url="macos-launch-environment-constraints.md" %}
[macos-launch-environment-constraints.md](macos-launch-environment-constraints.md)
{% endcontent-ref %}

## MRT - Інструмент видалення шкідливого програмного забезпечення

Інструмент видалення шкідливого програмного забезпечення (MRT) є ще однією частиною інфраструктури безпеки macOS. Як із назви випливає, основна функція MRT - **видалення відомого шкідливого програмного забезпечення з інфікованих систем**.

Після виявлення шкідливого програмного забезпечення на Mac (чи то за допомогою XProtect, чи іншими засобами), MRT може бути використаний для автоматичного **видалення шкідливого програмного забезпечення**. MRT працює непомітно в фоновому режимі і зазвичай запускається кожного разу, коли система оновлюється або коли завантажується нове визначення шкідливого програмного забезпечення (здається, що правила, за якими MRT виявляє шкідливе програмне забезпечення, знаходяться всередині бінарного файлу).

Хоча як XProtect, так і MRT є частинами заходів безпеки macOS, вони виконують різні функції:

* **XProtect** є запобіжним інструментом. Він **перевіряє файли при їх завантаженні** (через певні додатки), і якщо він виявляє будь-які відомі типи шкідливого програмного забезпечення, він **запобігає відкриттю файлу**, тим самим запобігаючи інфікуванню вашої системи в першу чергу.
* **MRT**, з іншого боку, є **реактивним інструментом**. Він працює після виявлення шкідливого програмного забезпечення на системі з метою видалення образливого програмного забезпечення для очищення системи.

Додаток MRT розташований в **`/Library/Apple/System/Library/CoreServices/MRT.app`**

## Керування фоновими завданнями

**macOS** тепер **повідомляє** кожного разу, коли інструмент використовує відомий **спосіб постійного виконання коду** (такий як елементи входу, демони...), щоб користувач краще **знав, яке програмне забезпечення постійно виконується**.

<figure><img src="../../../.gitbook/assets/image (711).png" alt=""><figcaption></figcaption></figure>

Це відбувається за допомогою **демона**, розташованого в `/System/Library/PrivateFrameworks/BackgroundTaskManagement.framework/Versions/A/Resources/backgroundtaskmanagementd` та **агента** в `/System/Library/PrivateFrameworks/BackgroundTaskManagement.framework/Support/BackgroundTaskManagementAgent.app`

Спосіб, як **`backgroundtaskmanagementd`** знає, що щось встановлено в постійну папку, полягає в тому, що він отримує **FSEvents** та створює деякі **обробники** для них.

Крім того, є файл plist, який містить **відомі додатки**, які часто виконуються постійно, підтримувані Apple, розташований за адресою: `/System/Library/PrivateFrameworks/BackgroundTaskManagement.framework/Versions/A/Resources/attributions.plist`
```json
[...]
"us.zoom.ZoomDaemon" => {
"AssociatedBundleIdentifiers" => [
0 => "us.zoom.xos"
]
"Attribution" => "Zoom"
"Program" => "/Library/PrivilegedHelperTools/us.zoom.ZoomDaemon"
"ProgramArguments" => [
0 => "/Library/PrivilegedHelperTools/us.zoom.ZoomDaemon"
]
"TeamIdentifier" => "BJ4HAAB9B3"
}
[...]
```
### Перелік

Можливо **перелічити всі** налаштовані фонові елементи, які працюють за допомогою інструменту командного рядка Apple:
```bash
# The tool will always ask for the users password
sfltool dumpbtm
```
Крім того, цю інформацію також можна вивести за допомогою [**DumpBTM**](https://github.com/objective-see/DumpBTM).
```bash
# You need to grant the Terminal Full Disk Access for this to work
chmod +x dumpBTM
xattr -rc dumpBTM # Remove quarantine attr
./dumpBTM
```
Ця інформація зберігається в **`/private/var/db/com.apple.backgroundtaskmanagement/BackgroundItems-v4.btm`** і для терміналу потрібен FDA.

### Втручання в BTM

Коли знайдено нову постійність, відбувається подія типу **`ES_EVENT_TYPE_NOTIFY_BTM_LAUNCH_ITEM_ADD`**. Таким чином, будь-який спосіб **запобігти** відправці цієї **події** або **агенту сповіщати** користувача допоможе зловмиснику _**обійти**_ BTM.

* **Скидання бази даних**: Виконання наступної команди скине базу даних (повинна буде перебудована з нуля), однак, з якоїсь причини після виконання цього **нові постійність не будуть сповіщені, поки систему не перезавантажено**.
* Потрібен **root**.
```bash
# Reset the database
sfltool resettbtm
```
* **Зупиніть агента**: Можливо надіслати сигнал зупинки агенту, щоб він **не сповіщав користувача**, коли виявляються нові виявлення.
```bash
# Get PID
pgrep BackgroundTaskManagementAgent
1011

# Stop it
kill -SIGSTOP 1011

# Check it's stopped (a T means it's stopped)
ps -o state 1011
T
```
* **Помилка**: Якщо **процес, що створив постійність, швидко завершується після цього**, демон спробує **отримати інформацію** про нього, **зазнає невдачу**, і **не зможе відправити подію**, що вказує на те, що нова річ стає постійною.

Посилання та **більше інформації про BTM**:

* [https://youtu.be/9hjUmT031tc?t=26481](https://youtu.be/9hjUmT031tc?t=26481)
* [https://www.patreon.com/posts/new-developer-77420730?l=fr](https://www.patreon.com/posts/new-developer-77420730?l=fr)
* [https://support.apple.com/en-gb/guide/deployment/depdca572563/web](https://support.apple.com/en-gb/guide/deployment/depdca572563/web)

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити свою **компанію рекламовану в HackTricks** або **завантажити HackTricks у PDF** Перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв.

</details>
