# Артефакти браузера

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію в рекламі на HackTricks** або **завантажити HackTricks у PDF** Перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Поділіться своїми хакерськими трюками, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв GitHub.

</details>

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Використовуйте [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) для легкої побудови та **автоматизації робочих процесів** за допомогою найбільш **продвинутих** інструментів спільноти у світі.\
Отримайте доступ сьогодні:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## Артефакти браузера <a href="#id-3def" id="id-3def"></a>

Артефакти браузера включають різні типи даних, збережених веб-браузерами, такі як історія навігації, закладки та кеш-дані. Ці артефакти зберігаються в конкретних папках в операційній системі, відрізняючись за місцем та назвою в різних браузерах, але загалом зберігають схожі типи даних.

Ось краткий огляд найпоширеніших артефактів браузера:

* **Історія навігації**: Відстежує відвідування користувачем веб-сайтів, корисно для виявлення відвідування шкідливих сайтів.
* **Дані автозаповнення**: Пропозиції на основі частих пошуків, надаючи уявлення, коли поєднано з історією навігації.
* **Закладки**: Сайти, збережені користувачем для швидкого доступу.
* **Розширення та додатки**: Розширення браузера або додатки, встановлені користувачем.
* **Кеш**: Зберігає веб-контент (наприклад, зображення, файли JavaScript) для покращення часу завантаження веб-сайтів, цінне для судового аналізу.
* **Логіни**: Збережені облікові дані для входу.
* **Фавікони**: Іконки, пов'язані з веб-сайтами, які з'являються во вкладках та закладках, корисні для додаткової інформації про відвідування користувача.
* **Сесії браузера**: Дані, пов'язані з відкритими сесіями браузера.
* **Завантаження**: Записи про файли, завантажені через браузер.
* **Дані форми**: Інформація, введена в веб-форми, збережена для майбутніх пропозицій автозаповнення.
* **Мініатюри**: Зображення попереднього перегляду веб-сайтів.
* **Custom Dictionary.txt**: Слова, додані користувачем до словника браузера.

## Firefox

Firefox організовує дані користувача в межах профілів, збережених в конкретних місцях в залежності від операційної системи:

* **Linux**: `~/.mozilla/firefox/`
* **MacOS**: `/Users/$USER/Library/Application Support/Firefox/Profiles/`
* **Windows**: `%userprofile%\AppData\Roaming\Mozilla\Firefox\Profiles\`

Файл `profiles.ini` в цих каталогах містить список профілів користувача. Дані кожного профілю зберігаються в папці, названій змінною `Path` в `profiles.ini`, розташованій в тому ж каталозі, що і сам `profiles.ini`. Якщо папка профілю відсутня, її можуть бути видалено.

У кожній папці профілю можна знайти кілька важливих файлів:

* **places.sqlite**: Зберігає історію, закладки та завантаження. Інструменти, такі як [BrowsingHistoryView](https://www.nirsoft.net/utils/browsing\_history\_view.html) на Windows, можуть отримати доступ до даних історії.
* Використовуйте конкретні SQL-запити для вилучення інформації про історію та завантаження.
* **bookmarkbackups**: Містить резервні копії закладок.
* **formhistory.sqlite**: Зберігає дані веб-форми.
* **handlers.json**: Управляє обробниками протоколів.
* **persdict.dat**: Слова користувача власного словника.
* **addons.json** та **extensions.sqlite**: Інформація про встановлені додатки та розширення.
* **cookies.sqlite**: Зберігання куки, з [MZCookiesView](https://www.nirsoft.net/utils/mzcv.html) доступним для перегляду на Windows.
* **cache2/entries** або **startupCache**: Дані кешу, доступні через інструменти, такі як [MozillaCacheView](https://www.nirsoft.net/utils/mozilla\_cache\_viewer.html).
* **favicons.sqlite**: Зберігає фавікони.
* **prefs.js**: Налаштування та вподобання користувача.
* **downloads.sqlite**: Стара база даних завантажень, тепер інтегрована в places.sqlite.
* **thumbnails**: Мініатюри веб-сайтів.
* **logins.json**: Зашифрована інформація для входу.
* **key4.db** або **key3.db**: Зберігає ключі шифрування для захисту чутливої інформації.

Додатково, перевірку налаштувань анти-фішингу браузера можна виконати, шукаючи записи `browser.safebrowsing` в `prefs.js`, що вказує, чи ввімкнені або вимкнені функції безпечного перегляду.

Для спроби розшифрувати головний пароль, ви можете використовувати [https://github.com/unode/firefox\_decrypt](https://github.com/unode/firefox\_decrypt)\
З наступним скриптом та викликом ви можете вказати файл паролю для перебору:
```bash
#!/bin/bash

#./brute.sh top-passwords.txt 2>/dev/null | grep -A2 -B2 "chrome:"
passfile=$1
while read pass; do
echo "Trying $pass"
echo "$pass" | python firefox_decrypt.py
done < $passfile
```
## Google Chrome

Google Chrome зберігає профілі користувачів у конкретних місцях в залежності від операційної системи:

* **Linux**: `~/.config/google-chrome/`
* **Windows**: `C:\Users\XXX\AppData\Local\Google\Chrome\User Data\`
* **MacOS**: `/Users/$USER/Library/Application Support/Google/Chrome/`

У цих каталогах більшість даних користувача можна знайти у папках **Default/** або **ChromeDefaultData/**. Наступні файли містять значущі дані:

* **Історія**: Містить URL-адреси, завантаження та ключові слова пошуку. На Windows можна використовувати [ChromeHistoryView](https://www.nirsoft.net/utils/chrome\_history\_view.html) для перегляду історії. Стовпець "Transition Type" має різні значення, включаючи кліки користувача на посилання, введені URL-адреси, відправлення форм та перезавантаження сторінок.
* **Cookies**: Зберігає файли cookie. Для перевірки доступний [ChromeCookiesView](https://www.nirsoft.net/utils/chrome\_cookies\_view.html).
* **Кеш**: Містить кешовані дані. Для перевірки користувачі Windows можуть використовувати [ChromeCacheView](https://www.nirsoft.net/utils/chrome\_cache\_view.html).
* **Закладки**: Закладки користувача.
* **Web Data**: Містить історію форм.
* **Favicons**: Зберігає значки веб-сайтів.
* **Login Data**: Включає дані для входу, такі як імена користувачів та паролі.
* **Поточна сесія**/**Поточні вкладки**: Дані про поточну сесію перегляду та відкриті вкладки.
* **Остання сесія**/**Останні вкладки**: Інформація про активні сайти під час останньої сесії перед закриттям Chrome.
* **Розширення**: Каталоги для розширень та додатків браузера.
* **Мініатюри**: Зберігає мініатюри веб-сайтів.
* **Preferences**: Файл, що містить багато інформації, включаючи налаштування для плагінів, розширень, спливаючих вікон, сповіщень та іншого.
* **Вбудований анти-фішинг браузера**: Щоб перевірити, чи увімкнено анти-фішинг та захист від шкідливих програм, виконайте `grep 'safebrowsing' ~/Library/Application Support/Google/Chrome/Default/Preferences`. Шукайте `{"enabled: true,"}` у виводі.

## **Відновлення даних з бази даних SQLite**

Як можна побачити в попередніх розділах, як Chrome, так і Firefox використовують бази даних **SQLite** для зберігання даних. Можливо **відновити видалені записи за допомогою інструменту** [**sqlparse**](https://github.com/padfoot999/sqlparse) **або** [**sqlparse\_gui**](https://github.com/mdegrazia/SQLite-Deleted-Records-Parser/releases).

## **Internet Explorer 11**

Internet Explorer 11 керує своїми даними та метаданими в різних місцях, що допомагає в розділенні збережених інформації та відповідних деталей для легкого доступу та управління.

### Зберігання метаданих

Метадані для Internet Explorer зберігаються в `%userprofile%\Appdata\Local\Microsoft\Windows\WebCache\WebcacheVX.data` (з VX, яке може бути V01, V16 або V24). Крім цього, файл `V01.log` може показувати розбіжності часу модифікації з `WebcacheVX.data`, що вказує на необхідність виправлення за допомогою `esentutl /r V01 /d`. Ці метадані, розміщені в базі даних ESE, можна відновити та перевірити за допомогою інструментів, таких як photorec та [ESEDatabaseView](https://www.nirsoft.net/utils/ese\_database\_view.html). У таблиці **Containers** можна визначити конкретні таблиці або контейнери, де зберігається кожний сегмент даних, включаючи деталі кешу для інших інструментів Microsoft, таких як Skype.

### Перевірка кешу

Інструмент [IECacheView](https://www.nirsoft.net/utils/ie\_cache\_viewer.html) дозволяє перевіряти кеш, вимагаючи розташування папки для видобутку даних кешу. Метадані для кешу включають ім'я файлу, каталог, кількість доступів, початковий URL та мітки часу, що вказують на час створення, доступу, модифікації та закінчення кешу.

### Управління файлами cookie

Файли cookie можна дослідити за допомогою [IECookiesView](https://www.nirsoft.net/utils/iecookies.html), а метадані включають імена, URL-адреси, кількість доступів та різні часові деталі. Постійні файли cookie зберігаються в `%userprofile%\Appdata\Roaming\Microsoft\Windows\Cookies`, а сеансові файли cookie знаходяться в пам'яті.

### Деталі завантажень

Метадані завантажень доступні за допомогою [ESEDatabaseView](https://www.nirsoft.net/utils/ese\_database\_view.html), і конкретні контейнери містять дані, такі як URL-адреса, тип файлу та місце завантаження. Фізичні файли можна знайти в `%userprofile%\Appdata\Roaming\Microsoft\Windows\IEDownloadHistory`.

### Історія перегляду

Для перегляду історії перегляду можна використовувати [BrowsingHistoryView](https://www.nirsoft.net/utils/browsing\_history\_view.html), вимагаючи розташування видобутих файлів історії та налаштування для Internet Explorer. Тут метадані включають часи модифікації та доступу, а також кількість доступів. Файли історії розташовані в `%userprofile%\Appdata\Local\Microsoft\Windows\History`.

### Введені URL-адреси

Введені URL-адреси та час їх використання зберігаються в реєстрі під `NTUSER.DAT` за адресою `Software\Microsoft\InternetExplorer\TypedURLs` та `Software\Microsoft\InternetExplorer\TypedURLsTime`, відстежуючи останні 50 URL-адрес, введених користувачем, та їх часи останнього введення.

## Microsoft Edge

Microsoft Edge зберігає дані користувачів у `%userprofile%\Appdata\Local\Packages`. Шляхи для різних типів даних такі:

* **Шлях профілю**: `C:\Users\XX\AppData\Local\Packages\Microsoft.MicrosoftEdge_XXX\AC`
* **Історія, Cookies та Завантаження**: `C:\Users\XX\AppData\Local\Microsoft\Windows\WebCache\WebCacheV01.dat`
* **Налаштування, Закладки та Список для читання**: `C:\Users\XX\AppData\Local\Packages\Microsoft.MicrosoftEdge_XXX\AC\MicrosoftEdge\User\Default\DataStore\Data\nouser1\XXX\DBStore\spartan.edb`
* **Кеш**: `C:\Users\XXX\AppData\Local\Packages\Microsoft.MicrosoftEdge_XXX\AC#!XXX\MicrosoftEdge\Cache`
* **Останні активні сесії**: `C:\Users\XX\AppData\Local\Packages\Microsoft.MicrosoftEdge_XXX\AC\MicrosoftEdge\User\Default\Recovery\Active`

## Safari

Дані Safari зберігаються в `/Users/$User/Library/Safari`. Основні файли включають:

* **History.db**: Містить таблиці `history_visits` та `history_items` з URL-адресами та часами відвідувань. Використовуйте `sqlite3` для запитів.
* **Downloads.plist**: Інформація про завантажені файли.
* **Bookmarks.plist**: Зберігає URL-адреси закладок.
* **TopSites.plist**: Найчастіше відвідувані сайти.
* **Extensions.plist**: Список розширень браузера Safari. Використовуйте `plutil` або `pluginkit` для отримання.
* **UserNotificationPermissions.plist**: Домени, яким дозволено надсилати сповіщення. Використовуйте `plutil` для аналізу.
* **LastSession.plist**: Вкладки з останньої сесії. Використовуйте `plutil` для аналізу.
* **Вбудований анти-фішинг браузера**: Перевірте за допомогою `defaults read com.apple.Safari WarnAboutFraudulentWebsites`. Відповідь 1 вказує на активність функції.

## Opera

Дані Opera розташовані в `/Users/$USER/Library/Application Support/com.operasoftware.Opera` та мають формат історії та завантажень, схожий на Chrome.

* **Вбудований анти-фішинг браузера**: Перевірте, чи параметр `fraud_protection_enabled` в файлі Preferences встановлено на `true` за допомогою `grep`.

Ці шляхи та команди є важливими для доступу та розуміння даних перегляду, збережених різними веб-браузерами.

## References

* [https://nasbench.medium.com/web-browsers-forensics-7e99940c579a](https://nasbench.medium.com/web-browsers-forensics-7e99940c579a)
* [https://www.sentinelone.com/labs/macos-incident-response-part-3-system-manipulation/](https://www.sentinelone.com/labs/macos-incident-response-part-3-system-manipulation/)
* [https://books.google.com/books?id=jfMqCgAAQBAJ\&pg=PA128\&lpg=PA128\&dq=%22This+file](https://books.google.com/books?id=jfMqCgAAQBAJ\&pg=PA128\&lpg=PA128\&dq=%22This+file)
* **Книга: OS X Incident Response: Scripting and Analysis By Jaron Bradley стор. 123**

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Використовуйте [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) для легкої побудови та **автоматизації робочих процесів** за допомогою найбільш розвинутих інструментів спільноти у світі.\
Отримайте доступ сьогодні:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>
* Якщо ви хочете побачити **рекламу вашої компанії на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний мерч PEASS & HackTricks**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) **і** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **репозиторіїв на GitHub.**
