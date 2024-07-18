# Зловживання установниками macOS

{% hint style="success" %}
Вивчайте та вправляйтеся в хакінгу AWS: <img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**Навчання AWS Red Team Expert (ARTE) від HackTricks**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та вправляйтеся в хакінгу GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**Навчання GCP Red Team Expert (GRTE) від HackTricks**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримайте HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Поширюйте хакерські трюки, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>
{% endhint %}

## Основна інформація про Pkg

Установочний пакет macOS (також відомий як файл `.pkg`) - це формат файлу, який використовується macOS для **розподілу програмного забезпечення**. Ці файли схожі на **коробку, яка містить все необхідне для встановлення та правильної роботи програми**.

Сам файл пакету є архівом, який містить **ієрархію файлів та каталогів, які будуть встановлені на цільовий** комп'ютер. Він також може включати **сценарії** для виконання завдань до та після встановлення, наприклад, налаштування файлів конфігурації або очищення старих версій програмного забезпечення.

### Ієрархія

<figure><img src="../../../.gitbook/assets/Pasted Graphic.png" alt="https://www.youtube.com/watch?v=iASSG0_zobQ"><figcaption></figcaption></figure>

* **Distribution (xml)**: Налаштування (назва, текст вітання...) та сценарії/перевірки встановлення
* **PackageInfo (xml)**: Інформація, вимоги до встановлення, місце встановлення, шляхи до сценаріїв для виконання
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
Для візуалізації вмісту установника без ручного розпакування ви також можете використовувати безкоштовний інструмент [**Suspicious Package**](https://mothersruin.com/software/SuspiciousPackage/).

## Основна інформація про DMG

Файли DMG, або образи диска Apple, - це формат файлу, який використовується macOS від Apple для образів дисків. Файл DMG суттєво є **монтуємим образом диска** (він містить власну файлову систему), який містить сирі блочні дані, які зазвичай стиснуті, а іноді зашифровані. Коли ви відкриваєте файл DMG, macOS **монтує його, ніби це фізичний диск**, що дозволяє вам отримати доступ до його вмісту.

{% hint style="danger" %}
Зверніть увагу, що установники **`.dmg`** підтримують **так багато форматів**, що у минулому деякі з них, що містять вразливості, були використані для отримання **виконання коду ядра**.
{% endhint %}

### Ієрархія

<figure><img src="../../../.gitbook/assets/image (225).png" alt=""><figcaption></figcaption></figure>

Ієрархія файлу DMG може бути різною в залежності від вмісту. Однак для DMG-файлів програм зазвичай слідує така структура:

* Верхній рівень: Це корінь образу диска. Він часто містить програму та, можливо, посилання на папку Applications.
* Додаток (.app): Це справжня програма. В macOS програма зазвичай є пакетом, який містить багато окремих файлів та папок, що складають програму.
* Посилання на додатки: Це ярлик до папки Applications в macOS. Мета полягає в тому, щоб зробити встановлення програми легким. Ви можете перетягнути файл .app на цей ярлик, щоб встановити додаток.

## Підвищення привілеїв через зловживання pkg

### Виконання з публічних каталогів

Якщо передустановочний або післяустановочний скрипт, наприклад, виконується з **`/var/tmp/Installerutil`**, і зловмисник може контролювати цей скрипт, він може підвищити привілеї кожного разу, коли він виконується. Або інший схожий приклад:

<figure><img src="../../../.gitbook/assets/Pasted Graphic 5.png" alt="https://www.youtube.com/watch?v=iASSG0_zobQ"><figcaption><p><a href="https://www.youtube.com/watch?v=kCXhIYtODBg">https://www.youtube.com/watch?v=kCXhIYtODBg</a></p></figcaption></figure>

### AuthorizationExecuteWithPrivileges

Це [публічна функція](https://developer.apple.com/documentation/security/1540038-authorizationexecutewithprivileg), яку викликають кілька установників та оновлювачів для **виконання чогось як root**. Ця функція приймає **шлях** до **файлу**, який **виконується** як параметр, однак, якщо зловмисник може **змінити** цей файл, він зможе **зловживати** його виконанням з правами root для **підвищення привілеїв**.
```bash
# Breakpoint in the function to check wich file is loaded
(lldb) b AuthorizationExecuteWithPrivileges
# You could also check FS events to find this missconfig
```
### Виконання шляхом монтування

Якщо програмний інсталятор записує дані до `/tmp/fixedname/bla/bla`, можливо **створити монтування** над `/tmp/fixedname` без власників, тому ви зможете **змінювати будь-який файл під час установки** для зловживання процесом установки.

Прикладом цього є **CVE-2021-26089**, який зміг **перезаписати періодичний скрипт**, щоб отримати виконання як root. Для отримання додаткової інформації перегляньте доповідь: [**OBTS v4.0: "Mount(ain) of Bugs" - Csaba Fitzl**](https://www.youtube.com/watch?v=jSYPazD4VcE)

## pkg як шкідливе ПЗ

### Порожній навантаження

Можливо просто створити файл **`.pkg`** з **перед та після-інсталяційними скриптами** без будь-якого навантаження.

### JS у файлі Distribution xml

Можливо додати теги **`<script>`** у файл **distribution xml** пакету, і цей код буде виконаний, і він може **виконувати команди** за допомогою **`system.run`**:

<figure><img src="../../../.gitbook/assets/image (1043).png" alt=""><figcaption></figcaption></figure>

## Посилання

* [**DEF CON 27 - Unpacking Pkgs A Look Inside Macos Installer Packages And Common Security Flaws**](https://www.youtube.com/watch?v=iASSG0\_zobQ)
* [**OBTS v4.0: "The Wild World of macOS Installers" - Tony Lambert**](https://www.youtube.com/watch?v=Eow5uNHtmIg)
* [**DEF CON 27 - Unpacking Pkgs A Look Inside MacOS Installer Packages**](https://www.youtube.com/watch?v=kCXhIYtODBg)
