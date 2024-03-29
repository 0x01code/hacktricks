<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію рекламовану на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакінг-прийомами, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв GitHub.

</details>


**Оригінальний пост** [**https://itm4n.github.io/windows-registry-rpceptmapper-eop/**](https://itm4n.github.io/windows-registry-rpceptmapper-eop/)

## Огляд

Було виявлено, що поточний користувач може записувати два ключі реєстру:

- **`HKLM\SYSTEM\CurrentControlSet\Services\Dnscache`**
- **`HKLM\SYSTEM\CurrentControlSet\Services\RpcEptMapper`**

Було запропоновано перевірити дозволи служби **RpcEptMapper** за допомогою **графічного інтерфейсу regedit**, зокрема вкладки **Ефективні дозволи** вікна **Розширені параметри безпеки**. Цей підхід дозволяє оцінити надані дозволи конкретним користувачам або групам без необхідності окремого розгляду кожного запису контролю доступу (ACE).

На знімку екрана показано дозволи, надані користувачеві з низькими привілеями, серед яких варто відзначити дозвіл **Створення підключа**, також відомий як **AppendData/AddSubdirectory**, що відповідає результатам скрипта.

Було відзначено неможливість безпосередньо змінювати певні значення, але можливість створення нових підключів. Було показано приклад спроби змінити значення **ImagePath**, що призвело до повідомлення про відмову в доступі.

Незважаючи на ці обмеження, було виявлено можливість підвищення привілеїв через можливість використання підключа **Performance** в структурі реєстру служби **RpcEptMapper**, який за замовчуванням відсутній. Це може дозволити реєстрацію DLL та моніторинг продуктивності.

Була використана документація про підключ **Performance** та його використання для моніторингу продуктивності, що призвело до розробки концепції DLL. Ця DLL, що демонструє реалізацію функцій **OpenPerfData**, **CollectPerfData** та **ClosePerfData**, була протестована за допомогою **rundll32**, підтверджуючи її успішну роботу.

Метою було змусити службу **RPC Endpoint Mapper** завантажити створену DLL продуктивності. Спостереження показали, що виконання запитів класу WMI, пов'язаних з даними продуктивності через PowerShell, призвело до створення файлу журналу, що дозволило виконання довільного коду в контексті **LOCAL SYSTEM**, надаючи підвищені привілеї.

Було підкреслено постійність та потенційні наслідки цієї уразливості, вказуючи на її важливість для стратегій післяексплуатаційних дій, бічного руху та ухилення від антивірусних/EDR систем.

Хоча уразливість спочатку була розкрита ненавмисно через скрипт, було підкреслено, що її експлуатація обмежена застарілими версіями Windows (наприклад, **Windows 7 / Server 2008 R2**) та потребує локального доступу.

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію рекламовану на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакінг-прийомами, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв GitHub.

</details>
