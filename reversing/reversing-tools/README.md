<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію рекламовану на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Дізнайтеся про [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакінговими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв GitHub.

</details>

# Посібник з декомпіляції Wasm та компіляції Wat

У світі **WebAssembly** інструменти для **декомпіляції** та **компіляції** є важливими для розробників. Цей посібник представляє деякі онлайн-ресурси та програмне забезпечення для роботи з файлами **Wasm (бінарний WebAssembly)** та **Wat (текстовий WebAssembly)**.

## Онлайн-інструменти

- Для **декомпіляції** Wasm у Wat, корисний інструмент доступний на [демонстрації wasm2wat від Wabt](https://webassembly.github.io/wabt/demo/wasm2wat/index.html).
- Для **компіляції** Wat назад у Wasm, служить [демонстрація wat2wasm від Wabt](https://webassembly.github.io/wabt/demo/wat2wasm/).
- Інша опція декомпіляції може бути знайдена на [web-wasmdec](https://wwwg.github.io/web-wasmdec/).

## Програмні рішення

- Для більш надійного рішення, [JEB від PNF Software](https://www.pnfsoftware.com/jeb/demo) пропонує широкі можливості.
- Проект з відкритим кодом [wasmdec](https://github.com/wwwg/wasmdec) також доступний для завдань декомпіляції.

# Ресурси для декомпіляції .Net

Декомпіляцію .Net збірок можна виконати за допомогою таких інструментів, як:

- [ILSpy](https://github.com/icsharpcode/ILSpy), який також пропонує [плагін для Visual Studio Code](https://github.com/icsharpcode/ilspy-vscode), що дозволяє використання на різних платформах.
- Для завдань, пов'язаних з **декомпіляцією**, **зміною** та **рекомпіляцією**, високо рекомендується [dnSpy](https://github.com/0xd4d/dnSpy/releases). Вибір методу правою кнопкою миші та вибір **Змінити метод** дозволяє вносити зміни в код.
- [dotPeek від JetBrains](https://www.jetbrains.com/es-es/decompiler/) є ще однією альтернативою для декомпіляції .Net збірок.

## Покращення відладки та журналювання з DNSpy

### Журналювання DNSpy
Для журналювання інформації у файл за допомогою DNSpy, включіть наступний фрагмент коду .Net:

%%%cpp
using System.IO;
path = "C:\\inetpub\\temp\\MyTest2.txt";
File.AppendAllText(path, "Пароль: " + password + "\n");
%%%

### Відлагодження DNSpy
Для ефективної відладки з DNSpy рекомендується послідовність кроків для налаштування **атрибутів Assembly** для відлагодження, забезпечуючи, що оптимізації, які можуть заважати відлагодженню, вимкнені. Цей процес включає зміну налаштувань `DebuggableAttribute`, рекомпіляцію збірки та збереження змін.

Крім того, для відлагодження додатка .Net, запущеного **IIS**, виконання `iisreset /noforce` перезапускає IIS. Для приєднання DNSpy до процесу IIS для відлагодження, посібник інструктує вибрати процес **w3wp.exe** у DNSpy та розпочати сеанс відлагодження.

Для комплексного перегляду завантажених модулів під час відлагодження рекомендується отримати доступ до вікна **Модулів** в DNSpy, а потім відкрити всі модулі та відсортувати збірки для зручнішої навігації та відлагодження.

Цей посібник узагальнює суть декомпіляції WebAssembly та .Net, пропонуючи шлях для розробників для легкості виконання цих завдань.

## **Декомпілятор Java**
Для декомпіляції Java байткоду ці інструменти можуть бути дуже корисними:
- [jadx](https://github.com/skylot/jadx)
- [JD-GUI](https://github.com/java-decompiler/jd-gui/releases)

## **Відлагодження DLLs**
### Використання IDA
- **Rundll32** завантажується з конкретних шляхів для 64-бітних та 32-бітних версій.
- **Windbg** вибирається як відлагоджувач з опцією призупинення завантаження/вивантаження бібліотек.
- Параметри виконання включають шлях до DLL та назву функції. Ця настройка призупиняє виконання при завантаженні кожної DLL.

### Використання x64dbg/x32dbg
- Подібно до IDA, **rundll32** завантажується з модифікаціями командного рядка для вказівки DLL та функції.
- Налаштування змінюються для зупинки на вході в DLL, що дозволяє встановлювати точку зупинки на бажаній точці входу в DLL.

### Зображення
- Точки зупинки виконання та конфігурації показані за допомогою знімків екрану.

## **ARM & MIPS**
- Для емуляції, [arm_now](https://github.com/nongiach/arm_now) є корисним ресурсом.

## **Шеллкоди**
### Техніки відлагодження
- **Blobrunner** та **jmp2it** - це інструменти для виділення шеллкодів у пам'яті та їх відлагодження з використанням Ida або x64dbg.
- Blobrunner [релізи](https://github.com/OALabs/BlobRunner/releases/tag/v0.0.5)
- jmp2it [скомпільована версія](https://github.com/adamkramer/jmp2it/releases/)
- **Cutter** пропонує емуляцію та інспекцію шеллкоду з графічним інтерфейсом користувача, підкреслюючи відмінності у роботі з файлом шеллкоду порівняно з безпосереднім шеллкодом.

### Деобфускація та аналіз
- **scdbg** надає відомості про функції шеллкоду та можливості деобфускації.
%%%bash
scdbg.exe -f shellcode # Базова інформація
scdbg.exe -f shellcode -r # Звіт про аналіз
scdbg.exe -f shellcode -i -r # Інтерактивні гачки
scdbg.exe -f shellcode -d # Виведення розкодованого шеллкоду
scdbg.exe -f shellcode /findsc # Пошук початкового зміщення
scdbg.exe -f shellcode /foff 0x0000004D # Виконання з зміщення
%%%

- **CyberChef** для розбору шеллкоду: [Рецепт CyberChef](https://gchq.github.io/CyberChef/#recipe=To_Hex%28'Space',0%29Disassemble_x86%28'32','Full%20x86%20architecture',16,0,true,true%29)

## **Movfuscator**
- Обфускатор, який замінює всі інструкції на `mov`.
- Корисні ресурси включають [пояснення на YouTube](https://www.youtube.com/watch?v=2VF_wPkiBJY) та [слайди PDF](https://github.com/xoreaxeaxeax/movfuscator/blob/master/slides/domas_2015_the_movfuscator.pdf).
- **demovfuscator** може розшифрувати обфускацію movfuscator, вимагаючи залежностей, таких як `libcapstone-dev` та `libz3-dev`, та встановлення [keystone](https://github.com/keystone-engine/keystone/blob/master/docs/COMPILE-NIX.md).
## **Delphi**
- Для бінарних файлів Delphi рекомендується використовувати [IDR](https://github.com/crypto2011/IDR).


# Курси

* [https://github.com/0xZ0F/Z0FCourse\_ReverseEngineering](https://github.com/0xZ0F/Z0FCourse_ReverseEngineering)
* [https://github.com/malrev/ABD](https://github.com/malrev/ABD) \(Деобфускація бінарних файлів\)



<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію в рекламі на HackTricks** або **завантажити HackTricks у PDF-форматі**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний мерч PEASS & HackTricks**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) **і** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **репозиторіїв на GitHub**.

</details>
