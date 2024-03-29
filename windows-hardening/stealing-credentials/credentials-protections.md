# Захист облікових даних Windows

## Захист облікових даних

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити **рекламу вашої компанії на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв GitHub.

</details>

## WDigest

Протокол [WDigest](https://technet.microsoft.com/pt-pt/library/cc778868(v=ws.10).aspx?f=255&MSPPError=-2147217396), введений з Windows XP, призначений для аутентифікації через протокол HTTP і **увімкнений за замовчуванням на Windows XP до Windows 8.0 та Windows Server 2003 до Windows Server 2012**. Це налаштування за замовчуванням призводить до **зберігання паролів у відкритому вигляді в LSASS** (Local Security Authority Subsystem Service). Атакувач може використовувати Mimikatz для **вилучення цих облікових даних**, виконавши:
```bash
sekurlsa::wdigest
```
Для **вимкнення або увімкнення цієї функції** ключі реєстру _**UseLogonCredential**_ та _**Negotiate**_ в межах _**HKEY\_LOCAL\_MACHINE\System\CurrentControlSet\Control\SecurityProviders\WDigest**_ повинні бути встановлені на "1". Якщо ці ключі **відсутні або встановлені на "0"**, WDigest **вимкнений**:
```bash
reg query HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest /v UseLogonCredential
```
## Захист LSA

Починаючи з **Windows 8.1**, Microsoft покращила безпеку LSA, щоб **блокувати несанкціоновані читання пам'яті або впровадження коду ненадійними процесами**. Це покращення ускладнює типове функціонування команд, таких як `mimikatz.exe sekurlsa:logonpasswords`. Для **увімкнення цього покращеного захисту**, значення _**RunAsPPL**_ в _**HKEY\_LOCAL\_MACHINE\SYSTEM\CurrentControlSet\Control\LSA**_ повинно бути налаштоване на 1:
```
reg query HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\LSA /v RunAsPPL
```
### Обхід

Можливо обійти цю захист за допомогою драйвера Mimikatz mimidrv.sys:

![](../../.gitbook/assets/mimidrv.png)

## Захист облікових даних

**Захист облікових даних**, функція, що доступна лише в **Windows 10 (Enterprise та Education виданнях)**, підвищує безпеку облікових даних машини за допомогою **віртуального захищеного режиму (VSM)** та **віртуалізованої безпеки (VBS)**. Вона використовує розширення віртуалізації процесора для ізоляції ключових процесів у захищеному просторі пам'яті, поза досягненням основної операційної системи. Ця ізоляція забезпечує, що навіть ядро не може отримати доступ до пам'яті в VSM, ефективно захищаючи облікові дані від атак, таких як **передача хешу**. **Місцевий орган безпеки (LSA)** працює в цій безпечній середовищі як довірений, тоді як процес **LSASS** в основній ОС виступає лише як комунікатор з LSA в VSM.

За замовчуванням **Захист облікових даних** не активний і вимагає ручної активації в організації. Це критично для підвищення безпеки проти інструментів, таких як **Mimikatz**, які обмежені у здатності видобувати облікові дані. Однак вразливості все ще можуть бути використані через додавання власних **постачальників підтримки безпеки (SSP)** для захоплення облікових даних у відкритому вигляді під час спроб входу.

Для перевірки статусу активації **Захисту облікових даних** можна перевірити ключ реєстру **_LsaCfgFlags_** у **_HKLM\System\CurrentControlSet\Control\LSA_**. Значення "**1**" вказує на активацію з **UEFI блокуванням**, "**2**" без блокування, а "**0**" позначає, що він не активований. Ця перевірка реєстру, хоча й є сильним показником, не єдиний крок для активації Захисту облікових даних. Детальні вказівки та сценарій PowerShell для активації цієї функції доступні в Інтернеті.
```powershell
reg query HKLM\System\CurrentControlSet\Control\LSA /v LsaCfgFlags
```
Для зрозуміння та інструкцій з увімкнення **Credential Guard** в Windows 10 та його автоматичного активації в сумісних системах **Windows 11 Enterprise та Education (версія 22H2)**, відвідайте [документацію Microsoft](https://docs.microsoft.com/en-us/windows/security/identity-protection/credential-guard/credential-guard-manage).

Додаткові відомості щодо впровадження власних SSP для захоплення облікових даних надані в [цьому керівництві](../active-directory-methodology/custom-ssp.md).


## Режим обмеженого адміністратора RDP

**Windows 8.1 та Windows Server 2012 R2** ввели кілька нових функцій безпеки, включаючи **_Режим обмеженого адміністратора для RDP_**. Цей режим був розроблений для підвищення безпеки шляхом зменшення ризиків, пов'язаних з атаками **[передача хешу](https://blog.ahasayen.com/pass-the-hash/)**.

Традиційно, при підключенні до віддаленого комп'ютера через RDP, ваші облікові дані зберігаються на цільовій машині. Це створює значний ризик для безпеки, особливо при використанні облікових записів з підвищеними привілеями. Однак з введенням **_Режиму обмеженого адміністратора_**, цей ризик значно зменшується.

При ініціюванні підключення RDP за допомогою команди **mstsc.exe /RestrictedAdmin**, аутентифікація на віддаленому комп'ютері виконується без зберігання ваших облікових даних на ньому. Цей підхід гарантує, що в разі зараження шкідливим ПЗ або якщо зловмисник отримає доступ до віддаленого сервера, ваші облікові дані не будуть компрометовані, оскільки вони не зберігаються на сервері.

Важливо зауважити, що в **Режимі обмеженого адміністратора** спроби доступу до мережевих ресурсів з сеансу RDP не використовуватимуть ваші особисті облікові дані; замість цього використовується **ідентифікатор машини**.

Ця функція є значним кроком уперед у забезпеченні безпеки підключень віддаленого робочого столу та захисту від розголошення конфіденційної інформації у разі порушення безпеки.

![](../../.gitbook/assets/ram.png)

Для отримання докладнішої інформації відвідайте [цей ресурс](https://blog.ahasayen.com/restricted-admin-mode-for-rdp/).


## Кешовані облікові дані

Windows захищає **доменні облікові дані** за допомогою **Місцевої служби безпеки (LSA)**, підтримуючи процеси входу за допомогою безпечних протоколів, таких як **Kerberos** та **NTLM**. Ключовою функцією Windows є можливість кешування **останніх десяти входів у домен**, щоб забезпечити можливість користувачам отримати доступ до своїх комп'ютерів навіть у випадку, якщо **контролер домену вийшов з ладу**—це велике плюс для користувачів ноутбуків, які часто перебувають поза мережею своєї компанії.

Кількість кешованих входів можна налаштувати за допомогою конкретного **реєстрового ключа або політики групи**. Для перегляду або зміни цього параметра використовується наступна команда:
```bash
reg query "HKEY_LOCAL_MACHINE\SOFTWARE\MICROSOFT\WINDOWS NT\CURRENTVERSION\WINLOGON" /v CACHEDLOGONSCOUNT
```
Доступ до цих кешованих облікових даних суворо контролюється, лише обліковий запис **SYSTEM** має необхідні дозволи для їх перегляду. Адміністраторам, які потребують доступу до цієї інформації, слід це робити з привілеями користувача SYSTEM. Облікові дані зберігаються за адресою: `HKEY_LOCAL_MACHINE\SECURITY\Cache`

**Mimikatz** може бути використаний для вилучення цих кешованих облікових даних за допомогою команди `lsadump::cache`.

Для отримання додаткових відомостей, оригінальне [джерело](http://juggernaut.wikidot.com/cached-credentials) надає вичерпну інформацію.


## Захищені користувачі

Членство в групі **Захищені користувачі** вводить кілька покращень з точки зору безпеки для користувачів, забезпечуючи вищий рівень захисту від крадіжки та зловживання обліковими даними:

- **Делегування облікових даних (CredSSP)**: Навіть якщо параметр політики групи для **Дозволити делегування облікових даних за замовчуванням** увімкнено, облікові дані Захищених користувачів не будуть кешуватися у вигляді звичайного тексту.
- **Windows Digest**: Починаючи з **Windows 8.1 та Windows Server 2012 R2**, система не буде кешувати облікові дані Захищених користувачів у вигляді звичайного тексту, незалежно від статусу Windows Digest.
- **NTLM**: Система не буде кешувати облікові дані Захищених користувачів у вигляді звичайного тексту або NT односторонніх функцій (NTOWF).
- **Kerberos**: Для Захищених користувачів аутентифікація Kerberos не буде генерувати ключі **DES** або **RC4**, а також не буде кешувати облікові дані у вигляді звичайного тексту або довгострокові ключі поза початковим отриманням квитка для отримання квитка (TGT).
- **Офлайн увімкнення**: Для Захищених користувачів не буде створено кешованого перевірника при увімкненні або розблокуванні, що означає, що офлайн увімкнення не підтримується для цих облікових записів.

Ці заходи захисту активуються в момент, коли користувач, який є членом групи **Захищені користувачі**, увійшов до пристрою. Це забезпечує наявність критичних заходів безпеки для захисту від різних методів компрометації облікових даних.

Для отримання докладної інформації, перегляньте офіційну [документацію](https://docs.microsoft.com/en-us/windows-server/security/credentials-protection-and-management/protected-users-security-group).

**Таблиця з** [**документації**](https://docs.microsoft.com/en-us/windows-server/identity/ad-ds/plan/security-best-practices/appendix-c--protected-accounts-and-groups-in-active-directory)**.**

| Windows Server 2003 RTM | Windows Server 2003 SP1+ | <p>Windows Server 2012,<br>Windows Server 2008 R2,<br>Windows Server 2008</p> | Windows Server 2016          |
| ----------------------- | ------------------------ | ----------------------------------------------------------------------------- | ---------------------------- |
| Account Operators       | Account Operators        | Account Operators                                                             | Account Operators            |
| Administrator           | Administrator            | Administrator                                                                 | Administrator                |
| Administrators          | Administrators           | Administrators                                                                | Administrators               |
| Backup Operators        | Backup Operators         | Backup Operators                                                              | Backup Operators             |
| Cert Publishers         |                          |                                                                               |                              |
| Domain Admins           | Domain Admins            | Domain Admins                                                                 | Domain Admins                |
| Domain Controllers      | Domain Controllers       | Domain Controllers                                                            | Domain Controllers           |
| Enterprise Admins       | Enterprise Admins        | Enterprise Admins                                                             | Enterprise Admins            |
|                         |                          |                                                                               | Enterprise Key Admins        |
|                         |                          |                                                                               | Key Admins                   |
| Krbtgt                  | Krbtgt                   | Krbtgt                                                                        | Krbtgt                       |
| Print Operators         | Print Operators          | Print Operators                                                               | Print Operators              |
|                         |                          | Read-only Domain Controllers                                                  | Read-only Domain Controllers |
| Replicator              | Replicator               | Replicator                                                                    | Replicator                   |
| Schema Admins           | Schema Admins            | Schema Admins                                                                 | Schema Admins                |
| Server Operators        | Server Operators         | Server Operators                                                              | Server Operators             |
