# Пропуски пісочниці macOS Office

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (Експерт з червоної команди HackTricks AWS)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію в рекламі HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами в **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв GitHub.

</details>

### Пропуск пісочниці Word через запуск агентів

Додаток використовує **власну пісочницю** з дозволом **`com.apple.security.temporary-exception.sbpl`**, і ця власна пісочниця дозволяє записувати файли будь-де, поки ім'я файлу починається з `~$`: `(require-any (require-all (vnode-type REGULAR-FILE) (regex #"(^|/)~$[^/]+$")))`

Отже, втеча була настільки простою, як **запис `plist`** LaunchAgent у `~/Library/LaunchAgents/~$escape.plist`.

Перевірте [**оригінальний звіт тут**](https://www.mdsec.co.uk/2018/08/escaping-the-sandbox-microsoft-office-on-macos/).

### Пропуск пісочниці Word через елементи входу та zip

Пам'ятайте, що після першої втечі Word може записувати довільні файли, ім'я яких починається з `~$`, хоча після попередньої уразливості вже не було можливості записувати в `/Library/Application Scripts` або в `/Library/LaunchAgents`.

Було виявлено, що з пісочниці можна створити **Елемент входу** (додатки, які виконуватимуться при вході користувача). Однак ці додатки **не виконаються**, якщо вони **не підписані** і **неможливо додати аргументи** (тому ви не можете просто запустити зворотний shell, використовуючи **`bash`**).

Після попереднього пропуску пісочниці Microsoft відключив можливість записувати файли в `~/Library/LaunchAgents`. Однак було виявлено, що якщо помістити **zip-файл як Елемент входу**, `Archive Utility` просто **розпакує** його у поточному місці. Оскільки за замовчуванням папка `LaunchAgents` з `~/Library` не створюється, було можливо **заархівувати plist у `LaunchAgents/~$escape.plist`** і **розмістити** zip-файл у **`~/Library`**, тому після розпакування він дістанеться до місця постійного збереження.

Перевірте [**оригінальний звіт тут**](https://objective-see.org/blog/blog\_0x4B.html).

### Пропуск пісочниці Word через елементи входу та .zshenv

(Пам'ятайте, що після першої втечі Word може записувати довільні файли, ім'я яких починається з `~$`).

Однак попередня техніка мала обмеження: якщо папка **`~/Library/LaunchAgents`** існує через те, що інші програми створили її, вона не працюватиме. Тому для цього було виявлено інший ланцюжок Елементів входу.

Атакуючий може створити файли **`.bash_profile`** та **`.zshenv`** з навантаженням для виконання, а потім заархівувати їх і **записати zip-файл у папку користувача жертви**: **`~/~$escape.zip`**.

Потім додайте zip-файл до **Елементів входу**, а потім до додатку **`Terminal`**. Коли користувач знову увійде в систему, zip-файл буде розпакований у файли користувача, перезаписуючи **`.bash_profile`** та **`.zshenv`**, і, отже, термінал виконає один з цих файлів (залежно від того, чи використовується bash чи zsh).

Перевірте [**оригінальний звіт тут**](https://desi-jarvis.medium.com/office365-macos-sandbox-escape-fcce4fa4123c).

### Пропуск пісочниці Word за допомогою Open та змінних середовища

З процесів у пісочниці все ще можна викликати інші процеси за допомогою утиліти **`open`**. Більше того, ці процеси будуть працювати **в межах власної пісочниці**.

Було виявлено, що утиліта open має опцію **`--env`** для запуску програми з **конкретними змінними середовища**. Тому було можливо створити файл **`.zshenv` у папці** **всередині** **пісочниці** і використовувати `open` з `--env`, встановлюючи змінну **`HOME`** на цю папку, відкриваючи додаток `Terminal`, який виконає файл `.zshenv` (з якоїсь причини також було потрібно встановити змінну `__OSINSTALL_ENVIROMENT`).

Перевірте [**оригінальний звіт тут**](https://perception-point.io/blog/technical-analysis-of-cve-2021-30864/).

### Пропуск пісочниці Word за допомогою Open та stdin

Утиліта **`open`** також підтримувала параметр **`--stdin`** (і після попереднього пропуску вже не було можливо використовувати `--env`).

Справа в тому, що навіть якщо **`python`** був підписаний Apple, він **не виконає** сценарій з атрибутом **`quarantine`**. Однак було можливо передати йому сценарій з stdin, тому він не перевірятиме, чи був він у карантині чи ні:&#x20;

1. Скиньте файл **`~$exploit.py`** з довільними командами Python.
2. Запустіть _open_ **`–stdin='~$exploit.py' -a Python`**, який запускає додаток Python з нашим файлом, що служить йому в якості стандартного вводу. Python з радістю виконує наш код, і оскільки він є дочірнім процесом _launchd_, він не обмежений правилами пісочниці Word.

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (Експерт з червоної команди HackTricks AWS)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію в рекламі HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами в **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв GitHub.

</details>
