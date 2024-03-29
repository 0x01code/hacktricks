# Зловживання установниками macOS

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити свою **компанію в рекламі на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) **і** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **репозиторіїв на GitHub**.

</details>

## Основна інформація про Pkg

Установочний пакет macOS (також відомий як файл `.pkg`) - це формат файлу, який використовується macOS для **розподілу програмного забезпечення**. Ці файли схожі на **коробку, яка містить все, що потрібно для встановлення та правильної роботи програми**.

Сам файл пакету є архівом, який містить **ієрархію файлів та каталогів, які будуть встановлені на цільовий** комп'ютер. Він також може включати **сценарії** для виконання завдань до та після встановлення, наприклад, налаштування файлів конфігурації або очищення старих версій програмного забезпечення.

### Ієрархія

<figure><img src="../../../.gitbook/assets/Pasted Graphic.png" alt="https://www.youtube.com/watch?v=iASSG0_zobQ"><figcaption></figcaption></figure>

* **Distribution (xml)**: Налаштування (назва, текст вітання...) та сценарії/перевірки встановлення
* **PackageInfo (xml)**: Інформація, вимоги до встановлення, місце встановлення, шляхи до сценаріїв для запуску
* **Bill of materials (bom)**: Список файлів для встановлення, оновлення або видалення з правами доступу до файлів
* **Payload (CPIO архів зі стисненням gzip)**: Файли для встановлення в `install-location` з PackageInfo
* **Scripts (CPIO архів зі стисненням gzip)**: Сценарії до та після встановлення та інші ресурси, розпаковані в тимчасовий каталог для виконання.
```bash
# Tool to directly get the files inside a package
pkgutil —expand "/path/to/package.pkg" "/path/to/out/dir"

# Get the files ina. more manual way
mkdir -p "/path/to/out/dir"
cd "/path/to/out/dir"
xar -xf "/path/to/package.pkg"

# Decompress also the CPIO gzip compressed ones
cat Scripts | gzip -dc | cpio -i
cpio -i < Scripts
```
Для візуалізації вмісту інсталятора без розпакування його вручну ви також можете використовувати безкоштовний інструмент [**Suspicious Package**](https://mothersruin.com/software/SuspiciousPackage/).

## Основна інформація про DMG

Файли DMG, або образи диска Apple, є форматом файлу, який використовується macOS від Apple для образів дисків. Файл DMG суттєво є **монтованим образом диска** (він містить власну файлову систему), який містить сирі блочні дані, які зазвичай стиснуті та іноді зашифровані. Коли ви відкриваєте файл DMG, macOS **монтує його, ніби це фізичний диск**, що дозволяє вам отримати доступ до його вмісту.

### Ієрархія

<figure><img src="../../../.gitbook/assets/image (12) (2).png" alt=""><figcaption></figcaption></figure>

Ієрархія файлу DMG може бути різною в залежності від вмісту. Однак для DMG-файлів програм зазвичай слідує ця структура:

* Верхній рівень: Це корінь образу диска. Він часто містить програму та, можливо, посилання на папку Applications.
* Додаток (.app): Це справжня програма. У macOS програма зазвичай є пакетом, який містить багато окремих файлів та папок, що складають програму.
* Посилання на додатки: Це ярлик до папки Applications у macOS. Мета полягає в тому, щоб зробити встановлення програми легким. Ви можете перетягнути файл .app на цей ярлик, щоб встановити додаток.

## Підвищення привілеїв через зловживання pkg

### Виконання з публічних каталогів

Якщо перед або після скрипт встановлення, наприклад, виконується з **`/var/tmp/Installerutil`**, і зловмисник може контролювати цей скрипт, він може підвищити привілеї при кожному його виконанні. Або інший схожий приклад:

<figure><img src="../../../.gitbook/assets/Pasted Graphic 5.png" alt="https://www.youtube.com/watch?v=iASSG0_zobQ"><figcaption></figcaption></figure>

### AuthorizationExecuteWithPrivileges

Це [публічна функція](https://developer.apple.com/documentation/security/1540038-authorizationexecutewithprivileg), яку викликають кілька інсталяторів та оновлень для **виконання чогось як root**. Ця функція приймає **шлях** до **файлу**, який **виконується** як параметр, однак, якщо зловмисник може **змінити** цей файл, він зможе **зловживати** його виконанням з правами root для **підвищення привілеїв**.
```bash
# Breakpoint in the function to check wich file is loaded
(lldb) b AuthorizationExecuteWithPrivileges
# You could also check FS events to find this missconfig
```
### Виконання шляхом монтування

Якщо програмний інсталятор записує дані в `/tmp/fixedname/bla/bla`, можливо **створити монтировання** над `/tmp/fixedname` без власників, щоб **змінювати будь-який файл під час установки** для зловживання процесом установки.

Прикладом цього є **CVE-2021-26089**, який зміг **перезаписати періодичний скрипт**, щоб отримати виконання як root. Для отримання додаткової інформації перегляньте виступ: [**OBTS v4.0: "Mount(ain) of Bugs" - Csaba Fitzl**](https://www.youtube.com/watch?v=jSYPazD4VcE)

## pkg як шкідливе ПЗ

### Порожній навантаження

Можливо просто створити файл **`.pkg`** з **перед та після-інсталяційними скриптами** без будь-якого навантаження.

### JS у файлі розподілу xml

Можливо додати теги **`<script>`** у файл розподілу xml пакета, і цей код буде виконаний, і він може **виконувати команди** за допомогою **`system.run`**:

<figure><img src="../../../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

## Посилання

* [**DEF CON 27 - Unpacking Pkgs A Look Inside Macos Installer Packages And Common Security Flaws**](https://www.youtube.com/watch?v=iASSG0\_zobQ)
* [**OBTS v4.0: "The Wild World of macOS Installers" - Tony Lambert**](https://www.youtube.com/watch?v=Eow5uNHtmIg)
