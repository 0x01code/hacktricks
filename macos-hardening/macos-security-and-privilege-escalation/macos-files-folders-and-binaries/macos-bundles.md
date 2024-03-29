# Бандли macOS

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (Експерт з червоної команди HackTricks AWS)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити **рекламу вашої компанії на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв.

</details>

## Основна інформація

Бандли в macOS служать контейнерами для різноманітних ресурсів, включаючи програми, бібліотеки та інші необхідні файли, що робить їх вигляд схожими на один об'єкт у Finder, такі як знайомі файли `*.app`. Найпоширенішим бандлом є бандл `.app`, хоча інші типи, такі як `.framework`, `.systemextension` та `.kext`, також поширені.

### Основні компоненти бандлу

У бандлі, зокрема у каталозі `<application>.app/Contents/`, розміщено різноманітні важливі ресурси:

* **\_CodeSignature**: Цей каталог зберігає важливі дані підписування коду для перевірки цілісності програми. Ви можете переглянути інформацію про підпис коду за допомогою команд, наприклад: %%%bash openssl dgst -binary -sha1 /Applications/Safari.app/Contents/Resources/Assets.car | openssl base64 %%%
* **MacOS**: Містить виконуваний бінарний файл програми, який запускається при взаємодії з користувачем.
* **Resources**: Сховище компонентів інтерфейсу користувача програми, включаючи зображення, документи та описи інтерфейсу (файли nib/xib).
* **Info.plist**: Діє як основний файл конфігурації програми, важливий для системи для визнання та взаємодії з програмою належним чином.

#### Важливі ключі в Info.plist

Файл `Info.plist` є важливим для конфігурації програми, містить ключі, такі як:

* **CFBundleExecutable**: Вказує назву головного виконуваного файлу, розташованого в каталозі `Contents/MacOS`.
* **CFBundleIdentifier**: Надає глобальний ідентифікатор програми, який широко використовується macOS для управління програмами.
* **LSMinimumSystemVersion**: Вказує мінімальну версію macOS, необхідну для запуску програми.

### Дослідження бандлів

Для дослідження вмісту бандлу, такого як `Safari.app`, можна використати наступну команду: `bash ls -lR /Applications/Safari.app/Contents`

Це дослідження розкриває каталоги, такі як `_CodeSignature`, `MacOS`, `Resources`, та файли, такі як `Info.plist`, кожен з яких виконує унікальну функцію від захисту програми до визначення її інтерфейсу користувача та операційних параметрів.

#### Додаткові каталоги бандлу

Поза загальними каталогами, бандли також можуть містити:

* **Frameworks**: Містить упаковані фреймворки, використовувані програмою. Фреймворки схожі на dylibs з додатковими ресурсами.
* **PlugIns**: Каталог для плагінів та розширень, які покращують можливості програми.
* **XPCServices**: Містить XPC-сервіси, використовувані програмою для міжпроцесної комунікації.

Ця структура забезпечує, що всі необхідні компоненти увібрані в бандл, сприяючи модулярному та безпечному середовищу програми.

Для отримання більш детальної інформації про ключі `Info.plist` та їх значення, документація розробника Apple надає обширні ресурси: [Apple Info.plist Key Reference](https://developer.apple.com/library/archive/documentation/General/Reference/InfoPlistKeyReference/Introduction/Introduction.html).

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (Експерт з червоної команди HackTricks AWS)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити **рекламу вашої компанії на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв.

</details>
