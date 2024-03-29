# Файли, теки, бінарні файли та пам'ять macOS

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію в рекламі HackTricks** або **завантажити HackTricks у PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв GitHub.

</details>

## Структура ієрархії файлів

* **/Applications**: Встановлені додатки повинні бути тут. Усі користувачі зможуть до них отримати доступ.
* **/bin**: Бінарні файли командного рядка
* **/cores**: Якщо існує, використовується для зберігання дампів ядра
* **/dev**: Все трактується як файл, тому ви можете побачити тут збережені пристрої апаратного забезпечення.
* **/etc**: Файли конфігурації
* **/Library**: Тут можна знайти багато підкаталогів та файлів, пов'язаних з налаштуваннями, кешами та журналами. Папка Library існує в корені та в кожному каталозі користувача.
* **/private**: Недокументовано, але багато згаданих тек є символічними посиланнями на приватну теку.
* **/sbin**: Основні системні бінарні файли (пов'язані з адмініструванням)
* **/System**: Файл для запуску OS X. Тут ви повинні знайти в основному лише файли, специфічні для Apple (не сторонні).
* **/tmp**: Файли видаляються через 3 дні (це символічне посилання на /private/tmp)
* **/Users**: Домашня тека для користувачів.
* **/usr**: Конфігураційні та системні бінарні файли
* **/var**: Файли журналів
* **/Volumes**: Підключені диски з'являться тут.
* **/.vol**: Запускаючи `stat a.txt`, ви отримуєте щось на зразок `16777223 7545753 -rw-r--r-- 1 username wheel ...`, де перше число - це ідентифікаційний номер тома, де існує файл, а друге - номер іноду. Ви можете отримати доступ до вмісту цього файлу через /.vol/ з цією інформацією, запустивши `cat /.vol/16777223/7545753`

### Теки додатків

* **Системні додатки** розташовані в `/System/Applications`
* **Встановлені** додатки зазвичай встановлюються в `/Applications` або в `~/Applications`
* **Дані додатків** можна знайти в `/Library/Application Support` для додатків, які працюють як root, та `~/Library/Application Support` для додатків, які працюють як користувач.
* Демони **сторонніх додатків**, які **потребують запуску як root**, зазвичай розташовані в `/Library/PrivilegedHelperTools/`
* **Додатки в пісочниці** відображаються в теку `~/Library/Containers`. Кожен додаток має теку з назвою, що відповідає ідентифікатору пакета додатка (`com.apple.Safari`).
* **Ядро** розташоване в `/System/Library/Kernels/kernel`
* **Розширення ядра Apple** розташовані в `/System/Library/Extensions`
* **Розширення ядра сторонніх виробників** зберігаються в `/Library/Extensions`

### Файли з чутливою інформацією

macOS зберігає інформацію, таку як паролі, в кількох місцях:

{% content-ref url="macos-sensitive-locations.md" %}
[macos-sensitive-locations.md](macos-sensitive-locations.md)
{% endcontent-ref %}

### Вразливі інсталятори pkg

{% content-ref url="macos-installers-abuse.md" %}
[macos-installers-abuse.md](macos-installers-abuse.md)
{% endcontent-ref %}

## Особливі розширення OS X

* **`.dmg`**: Файли образів дисків Apple дуже поширені для програм-інсталяторів.
* **`.kext`**: Вони повинні мати певну структуру і є версією драйвера для OS X. (це пакет)
* **`.plist`**: Також відомий як property list, зберігає інформацію у форматі XML або бінарному форматі.
* Може бути XML або бінарний. Бінарні можна прочитати за допомогою:
* `defaults read config.plist`
* `/usr/libexec/PlistBuddy -c print config.plsit`
* `plutil -p ~/Library/Preferences/com.apple.screensaver.plist`
* `plutil -convert xml1 ~/Library/Preferences/com.apple.screensaver.plist -o -`
* `plutil -convert json ~/Library/Preferences/com.apple.screensaver.plist -o -`
* **`.app`**: Додатки Apple, які слідують структурі теки (це пакет).
* **`.dylib`**: Динамічні бібліотеки (подібні до файлів DLL у Windows)
* **`.pkg`**: Це те саме, що й xar (eXtensible Archive format). Команда installer може використовуватися для встановлення вмісту цих файлів.
* **`.DS_Store`**: Цей файл є в кожній текі, він зберігає атрибути та налаштування теки.
* **`.Spotlight-V100`**: Ця тека з'являється в кореневій текі кожного тому в системі.
* **`.metadata_never_index`**: Якщо цей файл знаходиться в корені тому, Spotlight не індексуватиме цей том.
* **`.noindex`**: Файли та теки з цим розширенням не будуть індексуватися Spotlight.

### Пакети macOS

Пакет - це **тека**, яка **виглядає як об'єкт у Finder** (приклад пакетів - файли `*.app`).

{% content-ref url="macos-bundles.md" %}
[macos-bundles.md](macos-bundles.md)
{% endcontent-ref %}

## Кеш спільних бібліотек Dyld

На macOS (і iOS) всі системні спільні бібліотеки, такі як фреймворки та dylib, **об'єднані в один файл**, який називається **кешем спільних бібліотек dyld**. Це покращує продуктивність, оскільки код може завантажуватися швидше.

Аналогічно до кешу спільних бібліотек dyld, ядро та розширення ядра також компілюються в кеш ядра, який завантажується під час завантаження.

Для вилучення бібліотек з одного файлу кешу спільних бібліотек dylib можна було використовувати бінарний файл [dyld\_shared\_cache\_util](https://www.mbsplugins.de/files/dyld\_shared\_cache\_util-dyld-733.8.zip), який можливо зараз не працює, але ви також можете використовувати [**dyldextractor**](https://github.com/arandomdev/dyldextractor):

{% code overflow="wrap" %}
```bash
# dyld_shared_cache_util
dyld_shared_cache_util -extract ~/shared_cache/ /System/Volumes/Preboot/Cryptexes/OS/System/Library/dyld/dyld_shared_cache_arm64e

# dyldextractor
dyldex -l [dyld_shared_cache_path] # List libraries
dyldex_all [dyld_shared_cache_path] # Extract all
# More options inside the readme
```
{% endcode %}

У старіших версіях ви можете знайти **спільний кеш** в **`/System/Library/dyld/`**.

У iOS ви можете знайти їх в **`/System/Library/Caches/com.apple.dyld/`**.

{% hint style="success" %}
Зверніть увагу, що навіть якщо інструмент `dyld_shared_cache_util` не працює, ви можете передати **спільний dyld бінарний файл в Hopper** і Hopper зможе ідентифікувати всі бібліотеки та дозволить вам **вибрати, яку саме** ви хочете дослідити:
{% endhint %}

<figure><img src="../../../.gitbook/assets/image (680).png" alt="" width="563"><figcaption></figcaption></figure>

## Спеціальні дозволи на файли

### Дозволи на теки

У **теці**, **читання** дозволяє **переглядати її**, **запис** дозволяє **видаляти** та **записувати** файли в ній, а **виконання** дозволяє **переміщатися** по каталозі. Таким чином, наприклад, користувач з **дозволом на читання файлу** всередині каталогу, де він **не має дозволу на виконання**, **не зможе прочитати** файл.

### Модифікатори прапорців

Є деякі прапорці, які можуть бути встановлені в файлах, що змінюють поведінку файлу. Ви можете **перевірити прапорці** файлів всередині каталогу за допомогою `ls -lO /шлях/до/каталогу`

* **`uchg`**: Відомий як прапорець **uchange**, запобігає будь-яким діям зміни або видалення **файлу**. Щоб встановити його, виконайте: `chflags uchg file.txt`
* Користувач root може **видалити прапорець** та змінити файл
* **`restricted`**: Цей прапорець робить файл **захищеним за допомогою SIP** (ви не можете додати цей прапорець до файлу).
* **`Sticky bit`**: Якщо в каталозі встановлено прапорець Sticky bit, **тільки** власник каталогу або root можуть **перейменовувати або видаляти** файли. Зазвичай це встановлено в каталозі /tmp, щоб запобігти звичайним користувачам видаленню або переміщенню файлів інших користувачів.

Усі прапорці можна знайти в файлі `sys/stat.h` (знайдіть його за допомогою `mdfind stat.h | grep stat.h`) і це:

* `UF_SETTABLE` 0x0000ffff: Маска прапорців, які може змінити власник.
* `UF_NODUMP` 0x00000001: Не викидати файл.
* `UF_IMMUTABLE` 0x00000002: Файл не може бути змінений.
* `UF_APPEND` 0x00000004: Записи в файл можуть бути лише додані.
* `UF_OPAQUE` 0x00000008: Каталог є непрозоримим щодо об'єднання.
* `UF_COMPRESSED` 0x00000020: Файл стиснутий (деякі файлові системи).
* `UF_TRACKED` 0x00000040: Немає сповіщень про видалення/перейменування для файлів з цим набором.
* `UF_DATAVAULT` 0x00000080: Потрібно дозвіл для читання та запису.
* `UF_HIDDEN` 0x00008000: Підказка, що цей елемент не повинен відображатися в графічному інтерфейсі.
* `SF_SUPPORTED` 0x009f0000: Маска прапорців, які підтримує суперкористувач.
* `SF_SETTABLE` 0x3fff0000: Маска прапорців, які може змінити суперкористувач.
* `SF_SYNTHETIC` 0xc0000000: Маска системних тільки для читання синтетичних прапорців.
* `SF_ARCHIVED` 0x00010000: Файл заархівований.
* `SF_IMMUTABLE` 0x00020000: Файл не може бути змінений.
* `SF_APPEND` 0x00040000: Записи в файл можуть бути лише додані.
* `SF_RESTRICTED` 0x00080000: Потрібен дозвіл для запису.
* `SF_NOUNLINK` 0x00100000: Елемент не може бути видалений, перейменований або змонтований.
* `SF_FIRMLINK` 0x00800000: Файл є посиланням на програму.
* `SF_DATALESS` 0x40000000: Файл є об'єктом без даних.

### **ACL файлів**

ACL файлів містять **ACE** (Access Control Entries), де можна призначити більш **деталізовані дозволи** різним користувачам.

Можливо надати ці дозволи **каталогу**: `list`, `search`, `add_file`, `add_subdirectory`, `delete_child`, `delete_child`.\
А для **файлу**: `read`, `write`, `append`, `execute`.

Коли файл містить ACL, ви побачите **"+" при переліку дозволів, як у**:
```bash
ls -ld Movies
drwx------+   7 username  staff     224 15 Apr 19:42 Movies
```
Ви можете **читати ACLs** файлу за допомогою:
```bash
ls -lde Movies
drwx------+ 7 username  staff  224 15 Apr 19:42 Movies
0: group:everyone deny delete
```
Ви можете знайти **всі файли з ACL** за допомогою (це дуже повільно):
```bash
ls -RAle / 2>/dev/null | grep -E -B1 "\d: "
```
### Розширені атрибути

Розширені атрибути мають назву та бажане значення, і можуть бути переглянуті за допомогою `ls -@` та змінені за допомогою команди `xattr`. Деякі поширені розширені атрибути:

* `com.apple.resourceFork`: Сумісність ресурсного виливу. Також видимий як `filename/..namedfork/rsrc`
* `com.apple.quarantine`: MacOS: Механізм карантину Gatekeeper (III/6)
* `metadata:*`: MacOS: різноманітні метадані, такі як `_backup_excludeItem`, або `kMD*`
* `com.apple.lastuseddate` (#PS): Дата останнього використання файлу
* `com.apple.FinderInfo`: MacOS: Інформація Finder (наприклад, кольорові мітки)
* `com.apple.TextEncoding`: Вказує кодування тексту файлів ASCII
* `com.apple.logd.metadata`: Використовується logd на файлах у `/var/db/diagnostics`
* `com.apple.genstore.*`: Генераційне сховище (`/.DocumentRevisions-V100` в корені файлової системи)
* `com.apple.rootless`: MacOS: Використовується захистом цілісності системи для позначення файлу (III/10)
* `com.apple.uuidb.boot-uuid`: Позначки logd для епох завантаження з унікальним UUID
* `com.apple.decmpfs`: MacOS: Прозоре стиснення файлів (II/7)
* `com.apple.cprotect`: \*OS: Дані про шифрування для кожного файлу (III/11)
* `com.apple.installd.*`: \*OS: Метадані, використовувані installd, наприклад, `installType`, `uniqueInstallID`

### Ресурсні виливи | macOS ADS

Це спосіб отримання **Альтернативних потоків даних в MacOS**. Ви можете зберегти вміст всередині розширеного атрибуту з назвою **com.apple.ResourceFork** всередині файлу, зберігаючи його у **file/..namedfork/rsrc**.
```bash
echo "Hello" > a.txt
echo "Hello Mac ADS" > a.txt/..namedfork/rsrc

xattr -l a.txt #Read extended attributes
com.apple.ResourceFork: Hello Mac ADS

ls -l a.txt #The file length is still q
-rw-r--r--@ 1 username  wheel  6 17 Jul 01:15 a.txt
```
Ви можете **знайти всі файли, що містять цей розширений атрибут**, за допомогою:

{% code overflow="wrap" %}
```bash
find / -type f -exec ls -ld {} \; 2>/dev/null | grep -E "[x\-]@ " | awk '{printf $9; printf "\n"}' | xargs -I {} xattr -lv {} | grep "com.apple.ResourceFork"
```
### decmpfs

Розширений атрибут `com.apple.decmpfs` вказує, що файл зберігається у зашифрованому вигляді, `ls -l` повідомить про **розмір 0**, а стислі дані знаходяться у цьому атрибуті. Кожного разу, коли файл доступний, він буде розшифрований у пам'яті.

Цей атрибут можна побачити за допомогою `ls -lO`, вказаний як стиснутий, оскільки стиснені файли також позначаються прапорцем `UF_COMPRESSED`. Якщо стиснутий файл видалити цей прапорець за допомогою `chflags nocompressed </шлях/до/файлу>`, система не буде знати, що файл був стиснутий, і тому не зможе розпакувати та отримати доступ до даних (вона буде вважати, що файл насправді порожній).

Інструмент afscexpand можна використовувати для примусового розпакування файлу.

## **Універсальні бінарники та** Формат Mach-o

Бінарники Mac OS зазвичай компілюються як **універсальні бінарники**. **Універсальний бінарник** може **підтримувати кілька архітектур у одному файлі**.

{% content-ref url="universal-binaries-and-mach-o-format.md" %}
[universal-binaries-and-mach-o-format.md](universal-binaries-and-mach-o-format.md)
{% endcontent-ref %}

## Витяг пам'яті macOS

{% content-ref url="macos-memory-dumping.md" %}
[macos-memory-dumping.md](macos-memory-dumping.md)
{% endcontent-ref %}

## Файли категорії ризику Mac OS

Каталог `/System/Library/CoreServices/CoreTypes.bundle/Contents/Resources/System` - це місце, де зберігається інформація про **ризики, пов'язані з різними розширеннями файлів**. Цей каталог категоризує файли на різні рівні ризику, що впливає на те, як Safari обробляє ці файли під час завантаження. Категорії наступні:

* **LSRiskCategorySafe**: Файли цієї категорії вважаються **повністю безпечними**. Safari автоматично відкриє ці файли після їх завантаження.
* **LSRiskCategoryNeutral**: Ці файли не супроводжуються жодними попередженнями і **не відкриваються автоматично** Safari.
* **LSRiskCategoryUnsafeExecutable**: Файли цієї категорії **спричиняють попередження**, що файл є додатком. Це слугує як захисний захід для попередження користувача.
* **LSRiskCategoryMayContainUnsafeExecutable**: Ця категорія призначена для файлів, таких як архіви, які можуть містити виконуваний файл. Safari спричинить **попередження**, якщо вона не може перевірити, що всі вміст безпечні або нейтральні.

## Файли журналів

* **`$HOME/Library/Preferences/com.apple.LaunchServices.QuarantineEventsV2`**: Містить інформацію про завантажені файли, таку як URL, звідки вони були завантажені.
* **`/var/log/system.log`**: Основний журнал систем OSX. com.apple.syslogd.plist відповідає за виконання системного журналювання (можна перевірити, чи воно вимкнене, шукаючи "com.apple.syslogd" в `launchctl list`.
* **`/private/var/log/asl/*.asl`**: Це журнали системи Apple, які можуть містити цікаву інформацію.
* **`$HOME/Library/Preferences/com.apple.recentitems.plist`**: Зберігає нещодавно відкриті файли та програми через "Finder".
* **`$HOME/Library/Preferences/com.apple.loginitems.plsit`**: Зберігає елементи для запуску при запуску системи.
* **`$HOME/Library/Logs/DiskUtility.log`**: Файл журналу для програми DiskUtility (інформація про диски, включаючи USB-накопичувачі).
* **`/Library/Preferences/SystemConfiguration/com.apple.airport.preferences.plist`**: Дані про бездротові точки доступу.
* **`/private/var/db/launchd.db/com.apple.launchd/overrides.plist`**: Список демонів, які вимкнені.

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію рекламовану в HackTricks** або **завантажити HackTricks у PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Дізнайтеся про [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами в **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github репозиторіїв.

</details>
