<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію рекламовану на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Дізнайтеся про [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв.

</details>


# Перевірка можливих дій у GUI-додатку

**Загальні діалоги** - це опції **збереження файлу**, **відкриття файлу**, вибір шрифту, кольору... Більшість з них **пропонуватимуть повну функціональність провідника**. Це означає, що ви зможете отримати доступ до функцій провідника, якщо зможете отримати доступ до цих опцій:

* Закрити/Закрити як
* Відкрити/Відкрити за
* Друк
* Експорт/Імпорт
* Пошук
* Сканування

Ви повинні перевірити, чи можете ви:

* Змінювати або створювати нові файли
* Створювати символічні посилання
* Отримати доступ до обмежених областей
* Виконувати інші додатки

## Виконання команд

Можливо, **використовуючи опцію `Відкрити за`** ви можете відкрити/виконати якийсь вид оболонки.

### Windows

Наприклад _cmd.exe, command.com, Powershell/Powershell ISE, mmc.exe, at.exe, taskschd.msc..._ знайдіть більше виконуваних файлів, які можна використовувати для виконання команд (та виконання неочікуваних дій) тут: [https://lolbas-project.github.io/](https://lolbas-project.github.io)

### \*NIX __

_bash, sh, zsh..._ Більше тут: [https://gtfobins.github.io/](https://gtfobins.github.io)

# Windows

## Обхід обмежень шляху

* **Змінні середовища**: Є багато змінних середовища, які вказують на деякий шлях
* **Інші протоколи**: _about:, data:, ftp:, file:, mailto:, news:, res:, telnet:, view-source:_
* **Символічні посилання**
* **Ярлики**: CTRL+N (відкрити нову сесію), CTRL+R (Виконати команди), CTRL+SHIFT+ESC (Диспетчер завдань),  Windows+E (відкрити провідник), CTRL-B, CTRL-I (Вибране), CTRL-H (Історія), CTRL-L, CTRL-O (Діалог Відкриття файлу), CTRL-P (Діалог Друку), CTRL-S (Зберегти як)
* Приховане адміністративне меню: CTRL-ALT-F8, CTRL-ESC-F9
* **URI оболонки**: _shell:Administrative Tools, shell:DocumentsLibrary, shell:Librariesshell:UserProfiles, shell:Personal, shell:SearchHomeFolder, shell:Systemshell:NetworkPlacesFolder, shell:SendTo, shell:UsersProfiles, shell:Common Administrative Tools, shell:MyComputerFolder, shell:InternetFolder_
* **UNC-шляхи**: Шляхи для підключення до спільних папок. Ви повинні спробувати підключитися до C$ локальної машини ("\\\127.0.0.1\c$\Windows\System32")
* **Більше UNC-шляхів:**

| UNC                       | UNC            | UNC                  |
| ------------------------- | -------------- | -------------------- |
| %ALLUSERSPROFILE%         | %APPDATA%      | %CommonProgramFiles% |
| %COMMONPROGRAMFILES(x86)% | %COMPUTERNAME% | %COMSPEC%            |
| %HOMEDRIVE%               | %HOMEPATH%     | %LOCALAPPDATA%       |
| %LOGONSERVER%             | %PATH%         | %PATHEXT%            |
| %ProgramData%             | %ProgramFiles% | %ProgramFiles(x86)%  |
| %PROMPT%                  | %PSModulePath% | %Public%             |
| %SYSTEMDRIVE%             | %SYSTEMROOT%   | %TEMP%               |
| %TMP%                     | %USERDOMAIN%   | %USERNAME%           |
| %USERPROFILE%             | %WINDIR%       |                      |

## Завантажте ваші виконувані файли

Консоль: [https://sourceforge.net/projects/console/](https://sourceforge.net/projects/console/)\
Провідник: [https://sourceforge.net/projects/explorerplus/files/Explorer%2B%2B/](https://sourceforge.net/projects/explorerplus/files/Explorer%2B%2B/)\
Редактор реєстру: [https://sourceforge.net/projects/uberregedit/](https://sourceforge.net/projects/uberregedit/)

## Доступ до файлової системи з браузера

| ШЛЯХ                | ШЛЯХ              | ШЛЯХ               | ШЛЯХ                |
| ------------------- | ----------------- | ------------------ | ------------------- |
| File:/C:/windows    | File:/C:/windows/ | File:/C:/windows\\ | File:/C:\windows    |
| File:/C:\windows\\  | File:/C:\windows/ | File://C:/windows  | File://C:/windows/  |
| File://C:/windows\\ | File://C:\windows | File://C:\windows/ | File://C:\windows\\ |
| C:/windows          | C:/windows/       | C:/windows\\       | C:\windows          |
| C:\windows\\        | C:\windows/       | %WINDIR%           | %TMP%               |
| %TEMP%              | %SYSTEMDRIVE%     | %SYSTEMROOT%       | %APPDATA%           |
| %HOMEDRIVE%         | %HOMESHARE        |                    | <p><br></p>         |

## Ярлики

* Sticky Keys – Натисніть SHIFT 5 разів
* Mouse Keys – SHIFT+ALT+NUMLOCK
* High Contrast – SHIFT+ALT+PRINTSCN
* Toggle Keys – Утримуйте NUMLOCK протягом 5 секунд
* Filter Keys – Утримуйте правий SHIFT протягом 12 секунд
* WINDOWS+F1 – Пошук в Windows
* WINDOWS+D – Показати робочий стіл
* WINDOWS+E – Запустити Провідник Windows
* WINDOWS+R – Виконати
* WINDOWS+U – Центр спрощення доступу
* WINDOWS+F – Пошук
* SHIFT+F10 – Контекстне меню
* CTRL+SHIFT+ESC – Диспетчер завдань
* CTRL+ALT+DEL – Екран привітання на новіших версіях Windows
* F1 – Довідка F3 – Пошук
* F6 – Панель адреси
* F11 – Перемикання повноекранного режиму в Internet Explorer
* CTRL+H – Історія Internet Explorer
* CTRL+T – Internet Explorer – Нова вкладка
* CTRL+N – Internet Explorer – Нова сторінка
* CTRL+O – Відкрити файл
* CTRL+S – Зберегти CTRL+N – Новий RDP / Citrix
## Проведення пальцем

* Проведіть пальцем зліва направо, щоб побачити всі відкриті вікна, зменшити додаток KIOSK та отримати прямий доступ до всієї операційної системи;
* Проведіть пальцем зправа наліво, щоб відкрити Центр дій, зменшити додаток KIOSK та отримати прямий доступ до всієї операційної системи;
* Проведіть пальцем зверху вниз, щоб зробити видимою панель заголовка для додатка, відкритого в повноекранному режимі;
* Проведіть пальцем знизу вгору, щоб показати панель завдань у додатку на весь екран.

## Прийоми для Internet Explorer

### 'Панель зображення'

Це панель інструментів, яка з'являється у верхньому лівому куті зображення при його кліку. Ви зможете зберегти, надрукувати, надіслати поштою, відкрити "Мої зображення" в Провіднику. Кіоск повинен використовувати Internet Explorer.

### Протокол оболонки

Введіть ці URL-адреси, щоб отримати перегляд Провідника:

* `shell:Administrative Tools`
* `shell:DocumentsLibrary`
* `shell:Libraries`
* `shell:UserProfiles`
* `shell:Personal`
* `shell:SearchHomeFolder`
* `shell:NetworkPlacesFolder`
* `shell:SendTo`
* `shell:UserProfiles`
* `shell:Common Administrative Tools`
* `shell:MyComputerFolder`
* `shell:InternetFolder`
* `Shell:Profile`
* `Shell:ProgramFiles`
* `Shell:System`
* `Shell:ControlPanelFolder`
* `Shell:Windows`
* `shell:::{21EC2020-3AEA-1069-A2DD-08002B30309D}` --> Панель керування
* `shell:::{20D04FE0-3AEA-1069-A2D8-08002B30309D}` --> Мій комп'ютер
* `shell:::{{208D2C60-3AEA-1069-A2D7-08002B30309D}}` --> Мої мережеві місця
* `shell:::{871C5380-42A0-1069-A2EA-08002B30309D}` --> Internet Explorer

## Показ розширень файлів

Перевірте цю сторінку для отримання додаткової інформації: [https://www.howtohaven.com/system/show-file-extensions-in-windows-explorer.shtml](https://www.howtohaven.com/system/show-file-extensions-in-windows-explorer.shtml)

# Прийоми для браузерів

Резервне копіювання версій iKat:

[http://swin.es/k/](http://swin.es/k/)\
[http://www.ikat.kronicd.net/](http://www.ikat.kronicd.net)\

Створіть загальний діалог за допомогою JavaScript та отримайте доступ до Провідника файлів: `document.write('<input/type=file>')`
Джерело: https://medium.com/@Rend_/give-me-a-browser-ill-give-you-a-shell-de19811defa0

# iPad

## Жести та кнопки

* Проведіть вгору чотирма (або п'ятьма) пальцями / Подвійне натискання кнопки "Додому": Щоб переглянути перегляд багатьох завдань та змінити додаток

* Проведіть одним чи іншим способом чотирма або п'ятьма пальцями: Щоб перейти до наступного/попереднього додатка

* Зжимайте екран п'ятьма пальцями / Торкніться кнопки "Додому" / Проведіть пальцем вгору одним пальцем знизу екрана в швидкому русі вгору: Для доступу до Дому

* Проведіть одним пальцем знизу екрана всього 1-2 дюйми (повільно): З'явиться док-станція

* Проведіть пальцем вниз зверху дисплея одним пальцем: Щоб переглянути свої сповіщення

* Проведіть пальцем вниз одним пальцем у верхньому правому куті екрана: Щоб побачити центр керування iPad Pro

* Проведіть одним пальцем зліва від екрана 1-2 дюйми: Щоб побачити перегляд сьогодні

* Швидко проведіть одним пальцем з центру екрана вправо або вліво: Щоб перейти до наступного/попереднього додатка

* Натисніть і утримуйте кнопку Увімкнення/Вимкнення/Сон у верхньому правому куті iPad + Перемістіть повзунок "вимкнути" до кінця праворуч: Щоб вимкнути живлення

* Натисніть кнопку Увімкнення/Вимкнення/Сон у верхньому правому куті iPad та кнопку "Додому" протягом кількох секунд: Щоб примусово вимкнути живлення

* Натисніть кнопку Увімкнення/Вимкнення/Сон у верхньому правому куті iPad та кнопку "Додому" швидко: Щоб зробити знімок екрана, який з'явиться в нижньому лівому куті дисплея. Натисніть обидві кнопки одночасно дуже коротко, якщо ви утримуєте їх кілька секунд, буде виконано примусове вимкнення. 

## Швидкі клавіші

Вам потрібна клавіатура для iPad або адаптер USB клавіатури. Тут показані лише швидкі клавіші, які можуть допомогти вийти з додатка.

| Клавіша | Назва         |
| --- | ------------ |
| ⌘   | Команда      |
| ⌥   | Опція (Alt) |
| ⇧   | Shift        |
| ↩   | Повернення       |
| ⇥   | Таб          |
| ^   | Контроль      |
| ←   | Ліва стрілка   |
| →   | Права стрілка  |
| ↑   | Вгору     |
| ↓   | Вниз   |

### Системні швидкі клавіші

Ці швидкі клавіші призначені для візуальних налаштувань та звукових налаштувань, залежно від використання iPad.

| Швидка клавіша | Дія                                                                         |
| -------- | ------------------------------------------------------------------------------ |
| F1       | Затемнити екран                                                                    |
| F2       | Підсвітити екран                                                                |
| F7       | Повернутися на одну пісню                                                                  |
| F8       | Відтворення/пауза                                                                     |
| F9       | Пропустити пісню                                                                      |
| F10      | Без звуку                                                                           |
| F11      | Зменшити гучність                                                                |
| F12      | Збільшити гучність                                                                |
| ⌘ Пробіл  | Показати список доступних мов; щоб вибрати одну, ще раз натисніть пробіл. |

### Навігація iPad

| Швидка клавіша                                           | Дія                                                  |
| -------------------------------------------------- | ------------------------------------------------------- |
| ⌘H                                                 | Перейти на Головний екран                                              |
| ⌘⇧H (Command-Shift-H)                              | Перейти на Головний екран                                              |
| ⌘ (Space)                                          | Відкрити Spotlight                                          |
| ⌘⇥ (Command-Tab)                                   | Список останніх десяти використаних додатків                                 |
| ⌘\~                                                | Перейти до останнього додатка                                       |
| ⌘⇧3 (Command-Shift-3)                              | Зробити знімок екрана (з'являється в нижньому лівому куті для збереження або дій над ним) |
| ⌘⇧4                                                | Зробити знімок екрана та відкрити його в редакторі                    |
| Натисніть і утримуйте ⌘                                   | Список доступних швидких клавіш для додатка                 |
| ⌘⌥D (Command-Option/Alt-D)                         | Викликати док-станцію                                      |
| ^⌥H (Control-Option-H)                             | Кнопка "Додому"                                             |
| ^⌥H H (Control-Option-H-H)                         | Показати панель багатозадачності                                      |
| ^⌥I (Control-Option-i)                             | Вибір елемента                                            |
| Escape                                             | Кнопка "Назад"                                             |
| → (Права стрілка)                                    | Наступний елемент                                               |
| ← (Ліва стрілка)                                     | Попередній елемент                                           |
| ↑↓ (Стрілка вгору, стрілка вниз)                          | Одночасно натисніть вибраний елемент                        |
| ⌥ ↓ (Опція-стрілка вниз)                            | Прокрутити вниз                                             |
| ⌥↑ (Опція-стрілка вгору)                               | Прокрутити вгору                                               |
| ⌥← або ⌥→ (Опція-стрілка вліво або Опція-стрілка вправо) | Прокрутити вліво або вправо                                    |
| ^⌥S (Control-Option-S)                             | Увімкнути або вимкнути голосове оголошення                         |
| ⌘⇧⇥ (Command-Shift-Tab)                            | Переключитися на попередній додаток                              |
| ⌘⇥ (Command-Tab)                                   | Повернутися до початкового додатка                         |
| ←+→, потім Опція + ← або Опція+→                   | Навігація через Док                                   |
### Гарячі клавіші Safari

| Гаряча клавіша           | Дія                                              |
| ----------------------- | ------------------------------------------------ |
| ⌘L (Command-L)          | Відкрити рядок адреси                            |
| ⌘T                      | Відкрити нову вкладку                             |
| ⌘W                      | Закрити поточну вкладку                          |
| ⌘R                      | Оновити поточну вкладку                          |
| ⌘.                      | Зупинити завантаження поточної вкладки           |
| ^⇥                      | Перейти на наступну вкладку                       |
| ^⇧⇥ (Control-Shift-Tab) | Перейти на попередню вкладку                     |
| ⌘L                      | Вибрати текстове поле введення/URL для зміни    |
| ⌘⇧T (Command-Shift-T)   | Відкрити останню закриту вкладку (можна використовувати кілька разів) |
| ⌘\[                     | Повернутися на одну сторінку у вашій історії перегляду |
| ⌘]                      | Перейти на одну сторінку вперед у вашій історії перегляду |
| ⌘⇧R                     | Активувати режим читання                          |

### Гарячі клавіші пошти

| Гаряча клавіша          | Дія                           |
| ----------------------- | ---------------------------- |
| ⌘L                      | Відкрити рядок адреси         |
| ⌘T                      | Відкрити нову вкладку         |
| ⌘W                      | Закрити поточну вкладку      |
| ⌘R                      | Оновити поточну вкладку      |
| ⌘.                      | Зупинити завантаження поточної вкладки |
| ⌘⌥F (Command-Option/Alt-F) | Пошук у вашій поштовій скриньці |

# Посилання

* [https://www.macworld.com/article/2975857/6-only-for-ipad-gestures-you-need-to-know.html](https://www.macworld.com/article/2975857/6-only-for-ipad-gestures-you-need-to-know.html)
* [https://www.tomsguide.com/us/ipad-shortcuts,news-18205.html](https://www.tomsguide.com/us/ipad-shortcuts,news-18205.html)
* [https://thesweetsetup.com/best-ipad-keyboard-shortcuts/](https://thesweetsetup.com/best-ipad-keyboard-shortcuts/)
* [http://www.iphonehacks.com/2018/03/ipad-keyboard-shortcuts.html](http://www.iphonehacks.com/2018/03/ipad-keyboard-shortcuts.html)


<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити **рекламу вашої компанії на HackTricks** або **завантажити HackTricks у PDF-форматі**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами в **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>
