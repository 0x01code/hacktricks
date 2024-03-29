# Атака Skeleton Key

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (Експерт з червоної команди HackTricks AWS)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію в рекламі HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв.

</details>

Атака **Skeleton Key** - це складна техніка, яка дозволяє зловмисникам **обійти аутентифікацію Active Directory**, **впроваджуючи майстер-пароль** на контролер домену. Це дозволяє зловмиснику **аутентифікуватися як будь-який користувач** без їх пароля, ефективно **надаючи їм необмежений доступ** до домену.

Це можна зробити за допомогою [Mimikatz](https://github.com/gentilkiwi/mimikatz). Для виконання цієї атаки **необхідні права адміністратора домену**, і зловмиснику потрібно атакувати кожен контролер домену, щоб забезпечити всебічне порушення. Однак ефект атаки тимчасовий, оскільки **перезапуск контролера домену знищує шкідливе ПЗ**, що вимагає повторної реалізації для постійного доступу.

**Виконання атаки** вимагає однієї команди: `misc::skeleton`.

## Заходи запобігання

Стратегії запобігання таким атакам включають моніторинг конкретних ідентифікаторів подій, які вказують на встановлення служб або використання чутливих привілеїв. Зокрема, пошук події System Event ID 7045 або Security Event ID 4673 може розкрити підозрілі дії. Крім того, запуск `lsass.exe` як захищеного процесу може значно ускладнити зусилля зловмисників, оскільки для цього їм потрібно використовувати драйвер режиму ядра, що підвищує складність атаки.

Ось команди PowerShell для підвищення заходів безпеки:

- Для виявлення встановлення підозрілих служб використовуйте: `Get-WinEvent -FilterHashtable @{Logname='System';ID=7045} | ?{$_.message -like "*Kernel Mode Driver*"}`

- Зокрема, для виявлення драйвера Mimikatz можна використати наступну команду: `Get-WinEvent -FilterHashtable @{Logname='System';ID=7045} | ?{$_.message -like "*Kernel Mode Driver*" -and $_.message -like "*mimidrv*"}`

- Для зміцнення `lsass.exe` рекомендується ввести його як захищений процес: `New-ItemProperty HKLM:\SYSTEM\CurrentControlSet\Control\Lsa -Name RunAsPPL -Value 1 -Verbose`

Перевірка після перезавантаження системи є важливою для того, щоб переконатися, що захисні заходи були успішно застосовані. Це можливо за допомогою: `Get-WinEvent -FilterHashtable @{Logname='System';ID=12} | ?{$_.message -like "*protected process*`

## Посилання
* [https://blog.netwrix.com/2022/11/29/skeleton-key-attack-active-directory/](https://blog.netwrix.com/2022/11/29/skeleton-key-attack-active-directory/)
