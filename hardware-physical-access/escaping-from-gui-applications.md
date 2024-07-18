# Втеча з КІОСКів

{% hint style="success" %}
Вивчайте та практикуйте хакінг AWS: <img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**Навчання HackTricks AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте хакінг GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**Навчання HackTricks GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримайте HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Поширюйте хакерські трюки, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв GitHub.

</details>
{% endhint %}

#### [WhiteIntel](https://whiteintel.io)

<figure><img src="../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) - це пошуковик, який працює на **темному веб-просторі** та пропонує **безкоштовні** функціональні можливості для перевірки того, чи були **компанія або її клієнти скомпрометовані** **зловмисними програмами-крадіями**.

Основною метою WhiteIntel є боротьба з захопленням облікових записів та атаками вірусів-вимагачів, що виникають внаслідок викрадення інформації.

Ви можете перевірити їх веб-сайт та спробувати їхній двигун **безкоштовно** за посиланням:

{% embed url="https://whiteintel.io" %}

---

## Перевірка фізичного пристрою

|   Компонент   | Дія                                                               |
| ------------- | -------------------------------------------------------------------- |
| Кнопка живлення  | Вимкнення та увімкнення пристрою може викрити початковий екран      |
| Живлення   | Перевірте, чи пристрій перезавантажується при короткочасному відключенні живлення   |
| USB порти     | Підключіть фізичну клавіатуру з більшою кількістю скорочень                        |
| Ethernet      | Сканування мережі або перехоплення може дозволити подальшу експлуатацію             |


## Перевірка можливих дій всередині GUI-застосунку

**Загальні діалоги** - це опції **збереження файлу**, **відкриття файлу**, вибір шрифту, кольору... Більшість з них **пропонуватимуть повну функціональність Провідника**. Це означає, що ви зможете отримати доступ до функцій Провідника, якщо зможете отримати доступ до цих опцій:

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
* Виконувати інші програми

### Виконання команд

Можливо, **використовуючи опцію `Відкрити за`** ви можете відкрити/виконати який-небудь тип оболонки.

#### Windows

Наприклад _cmd.exe, command.com, Powershell/Powershell ISE, mmc.exe, at.exe, taskschd.msc..._ знайдіть більше виконуваних файлів, які можна використовувати для виконання команд (та виконання неочікуваних дій) тут: [https://lolbas-project.github.io/](https://lolbas-project.github.io)

#### \*NIX \_\_

_bash, sh, zsh..._ Більше тут: [https://gtfobins.github.io/](https://gtfobins.github.io)

## Windows

### Обхід обмежень шляху

* **Змінні середовища**: Є багато змінних середовища, які вказують на деякий шлях
* **Інші протоколи**: _about:, data:, ftp:, file:, mailto:, news:, res:, telnet:, view-source:_
* **Символічні посилання**
* **Ярлики**: CTRL+N (відкрити нову сесію), CTRL+R (Виконати команди), CTRL+SHIFT+ESC (Диспетчер завдань), Windows+E (відкрити провідник), CTRL-B, CTRL-I (Обрані), CTRL-H (Історія), CTRL-L, CTRL-O (Діалог Відкриття файлу), CTRL-P (Діалог Друку), CTRL-S (Зберегти як)
* Приховане адміністративне меню: CTRL-ALT-F8, CTRL-ESC-F9
* **URI оболонки**: _shell:Administrative Tools, shell:DocumentsLibrary, shell:Librariesshell:UserProfiles, shell:Personal, shell:SearchHomeFolder, shell:Systemshell:NetworkPlacesFolder, shell:SendTo, shell:UsersProfiles, shell:Common Administrative Tools, shell:MyComputerFolder, shell:InternetFolder_
* **UNC-шляхи**: Шляхи для підключення до спільних папок. Ви повинні спробувати підключитися до C$ локальної машини ("\\\127.0.0.1\c$\Windows\System32")
* **Ще UNC-шляхи:**

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

### Завантажте свої виконувані файли

Консоль: [https://sourceforge.net/projects/console/](https://sourceforge.net/projects/console/)\
Провідник: [https://sourceforge.net/projects/explorerplus/files/Explorer%2B%2B/](https://sourceforge.net/projects/explorerplus/files/Explorer%2B%2B/)\
Редактор реєстру: [https://sourceforge.net/projects/uberregedit/](https://sourceforge.net/projects/uberregedit/)

### Доступ до файлової системи з браузера

| ШЛЯХ                | ШЛЯХ              | ШЛЯХ               | ШЛЯХ                |
| ------------------- | ----------------- | ------------------ | ------------------- |
| File:/C:/windows    | File:/C:/windows/ | File:/C:/windows\\ | File:/C:\windows    |
| File:/C:\windows\\  | File:/C:\windows/ | File://C:/windows  | File://C:/windows/  |
| File://C:/windows\\ | File://C:\windows | File://C:\windows/ | File://C:\windows\\ |
| C:/windows          | C:/windows/       | C:/windows\\       | C:\windows          |
| C:\windows\\        | C:\windows/       | %WINDIR%           | %TMP%               |
| %TEMP%              | %SYSTEMDRIVE%     | %SYSTEMROOT%       | %APPDATA%           |
| %HOMEDRIVE%         | %HOMESHARE        |                    | <p><br></p>         |
### Гарячі клавіші

* Sticky Keys – Натисніть SHIFT 5 разів
* Mouse Keys – SHIFT+ALT+NUMLOCK
* Високий контраст – SHIFT+ALT+PRINTSCN
* Клавіші перемикання – Утримуйте NUMLOCK протягом 5 секунд
* Клавіші фільтрації – Утримуйте правий SHIFT протягом 12 секунд
* WINDOWS+F1 – Пошук у Windows
* WINDOWS+D – Показати робочий стіл
* WINDOWS+E – Запустити Провідник Windows
* WINDOWS+R – Виконати
* WINDOWS+U – Центр спрощення доступу
* WINDOWS+F – Пошук
* SHIFT+F10 – Контекстне меню
* CTRL+SHIFT+ESC – Диспетчер завдань
* CTRL+ALT+DEL – Екран запуску на новіших версіях Windows
* F1 – Довідка F3 – Пошук
* F6 – Панель адреси
* F11 – Перемикання на повноекранний режим у Internet Explorer
* CTRL+H – Історія Internet Explorer
* CTRL+T – Internet Explorer – Нова вкладка
* CTRL+N – Internet Explorer – Нова сторінка
* CTRL+O – Відкрити файл
* CTRL+S – Зберегти CTRL+N – Новий RDP / Citrix

### Проведення

* Проведіть від лівого краю до правого, щоб побачити всі відкриті вікна, мінімізуючи додаток KIOSK та отримуючи прямий доступ до всієї ОС;
* Проведіть від правого краю до лівого, щоб відкрити Центр дій, мінімізуючи додаток KIOSK та отримуючи прямий доступ до всієї ОС;
* Проведіть зверху, щоб зробити видимою панель заголовка для додатка, відкритого в повноекранному режимі;
* Проведіть вгору знизу, щоб показати панель завдань у додатку на повному екрані.

### Хитрощі Internet Explorer

#### 'Панель зображення'

Це панель інструментів, яка з'являється вгорі ліворуч зображення при його кліку. Ви зможете Зберегти, Роздрукувати, Надіслати поштою, Відкрити "Мої зображення" в Провіднику. Kiosk повинен використовувати Internet Explorer.

#### Протокол оболонки

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

### Показ розширень файлів

Перевірте цю сторінку для отримання додаткової інформації: [https://www.howtohaven.com/system/show-file-extensions-in-windows-explorer.shtml](https://www.howtohaven.com/system/show-file-extensions-in-windows-explorer.shtml)

## Хитрощі браузерів

Резервні копії версій iKat:

[http://swin.es/k/](http://swin.es/k/)\
[http://www.ikat.kronicd.net/](http://www.ikat.kronicd.net)\\

Створіть загальний діалог за допомогою JavaScript та отримайте доступ до провідника файлів: `document.write('<input/type=file>')`\
Джерело: https://medium.com/@Rend\_/give-me-a-browser-ill-give-you-a-shell-de19811defa0

## iPad

### Жести та кнопки

* Проведіть вгору чотирма (або п'ятьма) пальцями / Подвійне натискання кнопки "Додому": Щоб переглянути перегляд багатьох завдань та змінити додаток
* Проведіть одним чи іншим способом чотирма або п'ятьма пальцями: Щоб перейти до наступного/попереднього додатка
* Зжимайте екран п'ятьма пальцями / Торкніться кнопки "Додому" / Проведіть вгору одним пальцем знизу екрана швидким рухом вгору: Для доступу до дому
* Проведіть одним пальцем знизу екрана всього 1-2 дюйми (повільно): Появиться док-станція
* Проведіть пальцем вниз зверху дисплея одним пальцем: Щоб переглянути свої сповіщення
* Проведіть пальцем вниз одним пальцем у верхньому правому куті екрана: Щоб побачити центр керування iPad Pro
* Проведіть одним пальцем зліва від екрана 1-2 дюйми: Щоб побачити перегляд сьогодні
* Проведіть швидко одним пальцем з центру екрана вправо або вліво: Щоб перейти до наступного/попереднього додатка
* Натисніть і утримуйте кнопку Увімкнення/Вимкнення/Сон у верхньому правому куті iPad + Перемістіть повзунок "вимкнути" аж до кінця праворуч: Для вимкнення живлення
* Натисніть кнопку Увімкнення/Вимкнення/Сон у верхньому правому куті iPad та кнопку "Додому" протягом кількох секунд: Для примусового вимкнення
* Натисніть кнопку Увімкнення/Вимкнення/Сон у верхньому правому куті iPad та кнопку "Додому" швидко: Для зроблення знімка екрана, який з'явиться в нижньому лівому куті дисплея. Натисніть обидві кнопки одночасно дуже коротко, якщо ви утримуєте їх кілька секунд, буде виконано примусове вимкнення.

### Гарячі клавіші

Вам потрібна клавіатура для iPad або адаптер USB клавіатури. Тут показані лише гарячі клавіші, які можуть допомогти вийти з додатка.

| Клавіша | Назва         |
| --- | ------------ |
| ⌘   | Команда      |
| ⌥   | Опція (Alt) |
| ⇧   | Shift        |
| ↩   | Повернення       |
| ⇥   | Вкладка          |
| ^   | Контроль      |
| ←   | Вліво   |
| →   | Вправо  |
| ↑   | Вгору     |
| ↓   | Вниз   |

#### Гарячі клавіші системи

Ці гарячі клавіші призначені для візуальних налаштувань та звукових налаштувань, залежно від використання iPad.

| Гаряча клавіша | Дія                                                                         |
| -------- | ------------------------------------------------------------------------------ |
| F1       | Затемнити екран                                                                    |
| F2       | Підсвітити екран                                                                |
| F7       | Назад на одну пісню                                                                  |
| F8       | Відтворення/пауза                                                                     |
| F9       | Пропустити пісню                                                                      |
| F10      | Без звуку                                                                           |
| F11      | Зменшити гучність                                                                |
| F12      | Збільшити гучність                                                                |
| ⌘ Space  | Показати список доступних мов; щоб вибрати одну, ще раз натисніть пробіл. |

#### Навігація iPad

| Гаряча клавіша                                           | Дія                                                  |
| -------------------------------------------------- | ------------------------------------------------------- |
| ⌘H                                                 | Перейти на домашню сторінку                                              |
| ⌘⇧H (Command-Shift-H)                              | Перейти на домашню сторінку                                              |
| ⌘ (Space)                                          | Відкрити Spotlight                                          |
| ⌘⇥ (Command-Tab)                                   | Список останніх десяти використаних додатків                                 |
| ⌘\~                                                | Перейти до останнього додатка                                       |
| ⌘⇧3 (Command-Shift-3)                              | Знімок екрана (з'являється в нижньому лівому куті для збереження або дій над ним) |
| ⌘⇧4                                                | Знімок екрана та відкриття його в редакторі                    |
| Натисніть і утримуйте ⌘                                   | Список доступних скорочених клавіш для додатка                 |
| ⌘⌥D (Command-Option/Alt-D)                         | Виклик док-станції                                      |
| ^⌥H (Control-Option-H)                             | Кнопка "Додому"                                             |
| ^⌥H H (Control-Option-H-H)                         | Показати панель багатозадачності                                      |
| ^⌥I (Control-Option-i)                             | Вибір елемента                                            |
| Escape                                             | Кнопка "Назад"                                             |
| → (Стрілка вправо)                                    | Наступний елемент                                               |
| ← (Стрілка вліво)                                     | Попередній елемент                                           |
| ↑↓ (Стрілка вгору, Стрілка вниз)                          | Одночасно натисніть вибраний елемент                        |
| ⌥ ↓ (Опція-Стрілка вниз)                            | Прокрутити вниз                                             |
| ⌥↑ (Опція-Стрілка вгору)                               | Прокрутити вгору                                             |
| ⌥← або ⌥→ (Опція-Стрілка вліво або Опція-Стрілка вправо) | Прокрутити вліво або вправо                                    |
| ^⌥S (Control-Option-S)                             | Увімкнути або вимкнути голосовий вихід VoiceOver                         |
| ⌘⇧⇥ (Command-Shift-Tab)                            | Переключитися на попередній додаток                              |
| ⌘⇥ (Command-Tab)                                   | Повернутися до початкового додатка                         |
| ←+→, потім Опція + ← або Опція+→                   | Навігація через док-станцію                                   |
#### Гарячі клавіші Safari

| Клавіші                | Дія                                           |
| ----------------------- | --------------------------------------------- |
| ⌘L (Command-L)          | Відкрити рядок адреси                         |
| ⌘T                      | Відкрити нову вкладку                          |
| ⌘W                      | Закрити поточну вкладку                        |
| ⌘R                      | Оновити поточну вкладку                        |
| ⌘.                      | Зупинити завантаження поточної вкладки        |
| ^⇥                      | Перемкнутися на наступну вкладку               |
| ^⇧⇥ (Control-Shift-Tab) | Переміститися на попередню вкладку            |
| ⌘L                      | Вибрати поле введення тексту/URL для зміни    |
| ⌘⇧T (Command-Shift-T)   | Відкрити останню закриту вкладку (можна використовувати кілька разів) |
| ⌘\[                     | Повернутися на одну сторінку у вашій історії перегляду |
| ⌘]                      | Перейти на одну сторінку вперед у вашій історії перегляду |
| ⌘⇧R                     | Активувати режим читання                       |

#### Гарячі клавіші пошти

| Клавіші                   | Дія                           |
| -------------------------- | ---------------------------- |
| ⌘L                         | Відкрити рядок адреси         |
| ⌘T                         | Відкрити нову вкладку         |
| ⌘W                         | Закрити поточну вкладку      |
| ⌘R                         | Оновити поточну вкладку      |
| ⌘.                         | Зупинити завантаження поточної вкладки |
| ⌘⌥F (Command-Option/Alt-F) | Пошук у вашій поштовій скриньці |

## Посилання

* [https://www.macworld.com/article/2975857/6-only-for-ipad-gestures-you-need-to-know.html](https://www.macworld.com/article/2975857/6-only-for-ipad-gestures-you-need-to-know.html)
* [https://www.tomsguide.com/us/ipad-shortcuts,news-18205.html](https://www.tomsguide.com/us/ipad-shortcuts,news-18205.html)
* [https://thesweetsetup.com/best-ipad-keyboard-shortcuts/](https://thesweetsetup.com/best-ipad-keyboard-shortcuts/)
* [http://www.iphonehacks.com/2018/03/ipad-keyboard-shortcuts.html](http://www.iphonehacks.com/2018/03/ipad-keyboard-shortcuts.html)

#### [WhiteIntel](https://whiteintel.io)

<figure><img src="../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) - це пошуковий двигун, який працює на **темному веб-сайті** і пропонує **безкоштовні** функціональні можливості для перевірки, чи були **компанія або її клієнти** **пошкоджені** **викрадачами злочинців**.

Основна мета WhiteIntel - це боротьба з захопленням облікових записів та атаками вимагання викупу, що виникають внаслідок шкідливого програмного забезпечення, яке краде інформацію.

Ви можете перевірити їх веб-сайт та спробувати їх двигун **безкоштовно** за посиланням:

{% embed url="https://whiteintel.io" %}

{% hint style="success" %}
Вивчайте та практикуйте взлом AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**Навчання HackTricks AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте взлом GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**Навчання HackTricks GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримайте HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами в **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Поділіться хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) **та** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **репозиторіїв на GitHub.**

</details>
{% endhint %}
