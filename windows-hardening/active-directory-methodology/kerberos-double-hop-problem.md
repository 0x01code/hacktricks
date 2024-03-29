# Проблема подвійного переходу Kerberos

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Ви працюєте в **кібербезпеці компанії**? Хочете побачити **рекламу вашої компанії на HackTricks**? або хочете мати доступ до **останньої версії PEASS або завантажити HackTricks у PDF**? Перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* Отримайте [**офіційний PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Приєднуйтесь до** [**💬**](https://emojipedia.org/speech-balloon/) [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за мною на **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**репозиторію hacktricks**](https://github.com/carlospolop/hacktricks) **та** [**репозиторію hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>

## Вступ

Проблема "подвійного переходу" Kerberos виникає, коли атакуючий намагається використовувати **аутентифікацію Kerberos через два** **переходи**, наприклад, використовуючи **PowerShell**/**WinRM**.

Коли відбувається **аутентифікація** через **Kerberos**, **підтвердження** **не кешуються в пам'яті**. Тому, якщо ви запустите mimikatz, ви **не знайдете підтвердження** користувача на машині, навіть якщо він запускає процеси.

Це тому, що при підключенні за допомогою Kerberos відбуваються такі кроки:

1. Користувач1 надає підтвердження, і **контролер домену** повертає Kerberos **TGT** користувачу1.
2. Користувач1 використовує **TGT**, щоб запросити **квиток сервісу** для **підключення** до Сервер1.
3. Користувач1 **підключається** до **Сервер1** і надає **квиток сервісу**.
4. **Сервер1** **не має** **підтверджень** користувача1 в кеші або **TGT** користувача1. Тому, коли Користувач1 з Сервер1 намагається увійти на другий сервер, він **не може аутентифікуватися**.

### Неконтрольоване делегування

Якщо **неконтрольоване делегування** увімкнено на ПК, цього не станеться, оскільки **Сервер** отримає **TGT** кожного користувача, який до нього звертається. Крім того, якщо використовується неконтрольоване делегування, ви, ймовірно, зможете **компрометувати Контролер домену** з нього.\
[**Додаткова інформація на сторінці неконтрольованого делегування**](unconstrained-delegation.md).

### CredSSP

Ще один спосіб уникнути цю проблему, який [**відомий як небезпечний**](https://docs.microsoft.com/en-us/powershell/module/microsoft.wsman.management/enable-wsmancredssp?view=powershell-7), це **Постачальник підтримки безпеки облікових даних**. Від Microsoft:

> Аутентифікація CredSSP делегує облікові дані користувача з локального комп'ютера на віддалений комп'ютер. Ця практика збільшує ризик безпеки віддаленої операції. Якщо віддалений комп'ютер скомпрометований, коли облікові дані передаються йому, облікові дані можуть бути використані для управління мережевою сесією.

Дуже рекомендується **вимкнути CredSSP** на виробничих системах, в чутливих мережах та подібних середовищах через проблеми безпеки. Щоб визначити, чи **CredSSP** увімкнено, можна виконати команду `Get-WSManCredSSP`. Ця команда дозволяє **перевірити статус CredSSP** і навіть виконати її віддалено, якщо увімкнено **WinRM**.
```powershell
Invoke-Command -ComputerName bizintel -Credential ta\redsuit -ScriptBlock {
Get-WSManCredSSP
}
```
## Обхідні шляхи

### Виклик команди

Для вирішення проблеми подвійного переходу запропонований метод, що включає в себе вкладений `Invoke-Command`. Це не вирішує проблему безпосередньо, але пропонує обхідний шлях без необхідності спеціальних конфігурацій. Підхід дозволяє виконати команду (`hostname`) на вторинному сервері через команду PowerShell, виконану з початкової атакуючої машини або через раніше встановлену PS-сесію з першим сервером. Ось як це робиться:
```powershell
$cred = Get-Credential ta\redsuit
Invoke-Command -ComputerName bizintel -Credential $cred -ScriptBlock {
Invoke-Command -ComputerName secdev -Credential $cred -ScriptBlock {hostname}
}
```
Альтернативою є встановлення PS-сесії з першим сервером та запуск `Invoke-Command` за допомогою `$cred`, що рекомендується для централізації завдань.

### Зареєструвати конфігурацію PSSession

Рішення для обходу проблеми подвійного переходу полягає використання `Register-PSSessionConfiguration` з `Enter-PSSession`. Цей метод вимагає іншого підходу, ніж `evil-winrm`, і дозволяє створити сесію, яка не страждає від обмеження подвійного переходу.
```powershell
Register-PSSessionConfiguration -Name doublehopsess -RunAsCredential domain_name\username
Restart-Service WinRM
Enter-PSSession -ConfigurationName doublehopsess -ComputerName <pc_name> -Credential domain_name\username
klist
```
### Перенаправлення портів

Для локальних адміністраторів на проміжній цілі, перенаправлення портів дозволяє відправляти запити на кінцевий сервер. Використовуючи `netsh`, можна додати правило для перенаправлення портів, разом з правилом брандмауера Windows для дозволу перенаправленого порту.
```bash
netsh interface portproxy add v4tov4 listenport=5446 listenaddress=10.35.8.17 connectport=5985 connectaddress=10.35.8.23
netsh advfirewall firewall add rule name=fwd dir=in action=allow protocol=TCP localport=5446
```
#### winrs.exe

`winrs.exe` може бути використаний для пересилання запитів WinRM, потенційно як менш виявний варіант, якщо вас турбує моніторинг PowerShell. Наведена нижче команда демонструє його використання:
```bash
winrs -r:http://bizintel:5446 -u:ta\redsuit -p:2600leet hostname
```
### OpenSSH

Встановлення OpenSSH на першому сервері дозволяє обійти проблему подвійного переходу, особливо корисно для сценаріїв з jump box. Цей метод вимагає встановлення та налаштування OpenSSH для Windows через командний рядок. При налаштуванні для аутентифікації за паролем це дозволяє проміжному серверу отримати TGT від імені користувача.

#### Кроки встановлення OpenSSH

1. Завантажте та перемістіть останній архів релізу OpenSSH на цільовий сервер.
2. Розпакуйте та запустіть скрипт `Install-sshd.ps1`.
3. Додайте правило брандмауера для відкриття порту 22 та перевірте, що служби SSH працюють.

Для вирішення помилок `Connection reset` може знадобитися оновлення дозволів для надання всім доступу на читання та виконання в каталозі OpenSSH.
```bash
icacls.exe "C:\Users\redsuit\Documents\ssh\OpenSSH-Win64" /grant Everyone:RX /T
```
## Посилання

* [https://techcommunity.microsoft.com/t5/ask-the-directory-services-team/understanding-kerberos-double-hop/ba-p/395463?lightbox-message-images-395463=102145i720503211E78AC20](https://techcommunity.microsoft.com/t5/ask-the-directory-services-team/understanding-kerberos-double-hop/ba-p/395463?lightbox-message-images-395463=102145i720503211E78AC20)
* [https://posts.slayerlabs.com/double-hop/](https://posts.slayerlabs.com/double-hop/)
* [https://learn.microsoft.com/en-gb/archive/blogs/sergey\_babkins\_blog/another-solution-to-multi-hop-powershell-remoting](https://learn.microsoft.com/en-gb/archive/blogs/sergey\_babkins\_blog/another-solution-to-multi-hop-powershell-remoting)
* [https://4sysops.com/archives/solve-the-powershell-multi-hop-problem-without-using-credssp/](https://4sysops.com/archives/solve-the-powershell-multi-hop-problem-without-using-credssp/)

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Ви працюєте в **кібербезпеці компанії**? Хочете побачити вашу **компанію рекламовану на HackTricks**? або ви хочете мати доступ до **останньої версії PEASS або завантажити HackTricks у PDF**? Перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Дізнайтеся про [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* Отримайте [**офіційний PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Приєднуйтесь до** [**💬**](https://emojipedia.org/speech-balloon/) [**групи Discord**](https://discord.gg/hRep4RUj7f) або групи [**telegram**](https://t.me/peass) або **слідкуйте** за мною на **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**репозиторію hacktricks**](https://github.com/carlospolop/hacktricks) **і** [**репозиторію hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
