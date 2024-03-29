# Зловживання токенами

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Працюєте в **кібербезпеці компанії**? Хочете, щоб ваша **компанія рекламувалася на HackTricks**? або хочете мати доступ до **останньої версії PEASS або завантажити HackTricks у PDF**? Перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* **Приєднуйтесь до** [**💬**](https://emojipedia.org/speech-balloon/) [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за мною на **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до [репозиторію hacktricks](https://github.com/carlospolop/hacktricks) та [репозиторію hacktricks-cloud](https://github.com/carlospolop/hacktricks-cloud)**.

</details>

## Токени

Якщо ви **не знаєте, що таке токени доступу Windows**, прочитайте цю сторінку перед продовженням:

{% content-ref url="../access-tokens.md" %}
[access-tokens.md](../access-tokens.md)
{% endcontent-ref %}

**Можливо, ви зможете підвищити привілеї, зловживаючи токенами, які вже маєте**

### SeImpersonatePrivilege

Це привілегія, яку має будь-який процес, що дозволяє імітацію (але не створення) будь-якого токена, за умови, що можна отримати до нього доступ. Привілегований токен можна отримати від служби Windows (DCOM), спонукавши її виконати аутентифікацію NTLM проти експлойту, що подальше дозволяє виконати процес з привілеями SYSTEM. Цю уразливість можна використовувати за допомогою різних інструментів, таких як [juicy-potato](https://github.com/ohpe/juicy-potato), [RogueWinRM](https://github.com/antonioCoco/RogueWinRM) (який вимагає вимкнення winrm), [SweetPotato](https://github.com/CCob/SweetPotato) та [PrintSpoofer](https://github.com/itm4n/PrintSpoofer).

{% content-ref url="../roguepotato-and-printspoofer.md" %}
[roguepotato-and-printspoofer.md](../roguepotato-and-printspoofer.md)
{% endcontent-ref %}

{% content-ref url="../juicypotato.md" %}
[juicypotato.md](../juicypotato.md)
{% endcontent-ref %}

### SeAssignPrimaryPrivilege

Це дуже схоже на **SeImpersonatePrivilege**, воно використовуватиме **той самий метод** для отримання привілегованого токена.\
Потім ця привілегія дозволяє **призначити первинний токен** новому/призупиненому процесу. З привілегованим токеном імітації можна похідно створити первинний токен (DuplicateTokenEx).\
З цим токеном можна створити **новий процес** за допомогою 'CreateProcessAsUser' або створити процес призупинено та **встановити токен** (загалом, ви не можете змінювати первинний токен запущеного процесу).

### SeTcbPrivilege

Якщо у вас увімкнено цей токен, ви можете використовувати **KERB\_S4U\_LOGON**, щоб отримати **токен імітації** для будь-якого іншого користувача без знання облікових даних, **додати довільну групу** (адміністратори) до токена, встановити **рівень цілісності** токена на "**середній**" та призначити цей токен **поточному потоці** (SetThreadToken).

### SeBackupPrivilege

Система змушується **надавати всім доступ на читання** до будь-якого файлу (обмежено на операції читання) за допомогою цієї привілегії. Вона використовується для **читання хешів паролів локальних облікових записів адміністратора** з реєстру, після чого можна використовувати інструменти, такі як "**psexec**" або "**wmicexec**" з хешем (техніка Pass-the-Hash). Однак ця техніка не працює у двох випадках: коли обліковий запис локального адміністратора вимкнено або коли існує політика, яка забороняє адміністративні права для локальних адміністраторів, які підключаються віддалено.\
Ви можете **зловживати цією привілегією** за допомогою:

* [https://github.com/Hackplayers/PsCabesha-tools/blob/master/Privesc/Acl-FullControl.ps1](https://github.com/Hackplayers/PsCabesha-tools/blob/master/Privesc/Acl-FullControl.ps1)
* [https://github.com/giuliano108/SeBackupPrivilege/tree/master/SeBackupPrivilegeCmdLets/bin/Debug](https://github.com/giuliano108/SeBackupPrivilege/tree/master/SeBackupPrivilegeCmdLets/bin/Debug)
* слідуючи **IppSec** в [https://www.youtube.com/watch?v=IfCysW0Od8w\&t=2610\&ab\_channel=IppSec](https://www.youtube.com/watch?v=IfCysW0Od8w\&t=2610\&ab\_channel=IppSec)
* Або як пояснено в розділі **підвищення привілеїв з операторами резервного копіювання**:

{% content-ref url="../../active-directory-methodology/privileged-groups-and-token-privileges.md" %}
[privileged-groups-and-token-privileges.md](../../active-directory-methodology/privileged-groups-and-token-privileges.md)
{% endcontent-ref %}

### SeRestorePrivilege

Ця привілегія надає дозвіл на **запис доступу** до будь-якого системного файлу, незалежно від списку керування доступом (ACL) файлу. Вона відкриває безліч можливостей для підвищення привілеїв, включаючи можливість **змінювати служби**, виконувати DLL Hijacking та встановлювати **відлагоджувачі** через параметри виконання файлу зображення серед інших різних технік.

### SeCreateTokenPrivilege

SeCreateTokenPrivilege є потужним дозволом, особливо корисним, коли користувач має можливість імітувати токени, але також у відсутності SeImpersonatePrivilege. Ця можливість залежить від здатності імітувати токен, який представляє того ж користувача і чиїй рівень цілісності не перевищує рівня цілісності поточного процесу.

**Ключові моменти:**
- **Імітація без SeImpersonatePrivilege:** Можливо використовувати SeCreateTokenPrivilege для підвищення привілеїв за певних умов імітації токенів.
- **Умови для імітації токенів:** Успішна імітація вимагає, щоб цільовий токен належав тому ж користувачеві та мав рівень цілісності, який менше або дорівнює рівню цілісності процесу, який намагається імітувати.
- **Створення та зміна токенів імітації:** Користувачі можуть створювати токен імітації та покращувати його, додавши ідентифікатор безпеки привілегованої групи.

### SeLoadDriverPrivilege

Ця привілегія дозволяє **завантажувати та вивантажувати драйвери пристроїв** з створенням запису реєстру зі специфічними значеннями для `ImagePath` та `Type`. Оскільки прямий доступ до запису `HKLM` (HKEY_LOCAL_MACHINE) обмежений, замість цього слід використовувати `HKCU` (HKEY_CURRENT_USER). Однак для того, щоб зробити `HKCU` впізнаваним ядром для конфігурації драйвера, слід дотримуватися певного шляху.

Цей шлях - `\Registry\User\<RID>\System\CurrentControlSet\Services\DriverName`, де `<RID>` є Відносним Ідентифікатором поточного користувача. У межах `HKCU` слід створити цей весь шлях та встановити два значення:
- `ImagePath`, який є шляхом до виконавчого файлу
- `Type`, зі значенням `SERVICE_KERNEL_DRIVER` (`0x00000001`).

**Кроки для виконання:**
1. Отримати доступ до `HKCU` замість `HKLM` через обмежений доступ на запис.
2. Створити шлях `\Registry\User\<RID>\System\CurrentControlSet\Services\DriverName` всередині `HKCU`, де `<RID>` представляє Відносний Ідентифікатор поточного користувача.
3. Встановити `ImagePath` до шляху виконання бінарного файлу.
4. Призначити `Type` як `SERVICE_KERNEL_DRIVER` (`0x00000001`).
```python
# Example Python code to set the registry values
import winreg as reg

# Define the path and values
path = r'Software\YourPath\System\CurrentControlSet\Services\DriverName' # Adjust 'YourPath' as needed
key = reg.OpenKey(reg.HKEY_CURRENT_USER, path, 0, reg.KEY_WRITE)
reg.SetValueEx(key, "ImagePath", 0, reg.REG_SZ, "path_to_binary")
reg.SetValueEx(key, "Type", 0, reg.REG_DWORD, 0x00000001)
reg.CloseKey(key)
```
Додаткові способи зловживання цим привілеєм за посиланням [https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/privileged-accounts-and-token-privileges#seloaddriverprivilege](https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/privileged-accounts-and-token-privileges#seloaddriverprivilege)

### SeTakeOwnershipPrivilege

Це схоже на **SeRestorePrivilege**. Його основна функція дозволяє процесу **припустити власність об'єкта**, обійшовши вимогу до явного доступу за допомогою надання прав доступу WRITE_OWNER. Процес полягає в спочатку забезпеченні власності потрібного ключа реєстру для запису, а потім зміні DACL для можливості виконання запису.
```bash
takeown /f 'C:\some\file.txt' #Now the file is owned by you
icacls 'C:\some\file.txt' /grant <your_username>:F #Now you have full access
# Use this with files that might contain credentials such as
%WINDIR%\repair\sam
%WINDIR%\repair\system
%WINDIR%\repair\software
%WINDIR%\repair\security
%WINDIR%\system32\config\security.sav
%WINDIR%\system32\config\software.sav
%WINDIR%\system32\config\system.sav
%WINDIR%\system32\config\SecEvent.Evt
%WINDIR%\system32\config\default.sav
c:\inetpub\wwwwroot\web.config
```
### SeDebugPrivilege

Ця привілея дозволяє **налагоджувати інші процеси**, включаючи читання та запис у пам'ять. З цією привілеєю можна використовувати різні стратегії для впровадження в пам'ять, здатні ухилятися від більшості антивірусів та рішень для запобігання вторгнення на хост.

#### Вивантаження пам'яті

Ви можете використовувати [ProcDump](https://docs.microsoft.com/en-us/sysinternals/downloads/procdump) з [SysInternals Suite](https://docs.microsoft.com/en-us/sysinternals/downloads/sysinternals-suite), щоб **захопити пам'ять процесу**. Зокрема, це може стосуватися процесу **Local Security Authority Subsystem Service ([LSASS](https://en.wikipedia.org/wiki/Local_Security_Authority_Subsystem_Service))**, який відповідає за зберігання облікових даних користувача після успішного входу користувача до системи.

Після цього ви можете завантажити це вивантаження в mimikatz, щоб отримати паролі:
```
mimikatz.exe
mimikatz # log
mimikatz # sekurlsa::minidump lsass.dmp
mimikatz # sekurlsa::logonpasswords
```
#### RCE

Якщо ви хочете отримати оболонку `NT SYSTEM`, ви можете використовувати:

* ****[**SeDebugPrivilege-Exploit (C++)**](https://github.com/bruno-1337/SeDebugPrivilege-Exploit)****
* ****[**SeDebugPrivilegePoC (C#)**](https://github.com/daem0nc0re/PrivFu/tree/main/PrivilegedOperations/SeDebugPrivilegePoC)****
* ****[**psgetsys.ps1 (Powershell Script)**](https://raw.githubusercontent.com/decoder-it/psgetsystem/master/psgetsys.ps1)****
```powershell
# Get the PID of a process running as NT SYSTEM
import-module psgetsys.ps1; [MyProcess]::CreateProcessFromParent(<system_pid>,<command_to_execute>)
```
## Перевірка привілеїв
```
whoami /priv
```
**Токени, які з'являються як вимкнені**, можуть бути увімкнені, ви можете фактично зловживати _Увімкненими_ та _Вимкненими_ токенами.

### Увімкнути всі токени

Якщо у вас вимкнені токени, ви можете використати скрипт [**EnableAllTokenPrivs.ps1**](https://raw.githubusercontent.com/fashionproof/EnableAllTokenPrivs/master/EnableAllTokenPrivs.ps1), щоб увімкнути всі токени:
```powershell
.\EnableAllTokenPrivs.ps1
whoami /priv
```
Або **скрипт** вбудований у цей [**пост**](https://www.leeholmes.com/adjusting-token-privileges-in-powershell/).

## Таблиця

Повний шпаргалка привілеїв токенів за посиланням [https://github.com/gtworek/Priv2Admin](https://github.com/gtworek/Priv2Admin), нижче наведено лише прямі способи використання привілеїв для отримання сеансу адміністратора або читання чутливих файлів.

| Привілегія                | Вплив       | Інструмент              | Шлях виконання                                                                                                                                                                                                                                                                                                                                     | Примітки                                                                                                                                                                                                                                                                                                                      |
| -------------------------- | ----------- | ----------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **`SeAssignPrimaryToken`** | _**Адмін**_ | Інструмент стороннього розробника | _"Це дозволить користувачеві імітувати токени та підвищувати привілеї до nt системи за допомогою таких інструментів, як potato.exe, rottenpotato.exe та juicypotato.exe"_                                                                                                                                                                                                      | Дякую [Aurélien Chalot](https://twitter.com/Defte\_) за оновлення. Спробую переформулювати це на щось більш схоже на рецепт незабаром.                                                                                                                                                                                        |
| **`SeBackup`**             | **Загроза** | _**Вбудовані команди**_ | Читання чутливих файлів за допомогою `robocopy /b`                                                                                                                                                                                                                                                                                                             | <p>- Може бути цікавіше, якщо ви зможете прочитати %WINDIR%\MEMORY.DMP<br><br>- <code>SeBackupPrivilege</code> (і robocopy) не допомагає, коли мова йде про відкриття файлів.<br><br>- Robocopy потребує як SeBackup, так і SeRestore для роботи з параметром /b.</p>                                                                      |
| **`SeCreateToken`**        | _**Адмін**_ | Інструмент стороннього розробника | Створення довільного токену, включаючи права локального адміністратора за допомогою `NtCreateToken`.                                                                                                                                                                                                                                                                          |                                                                                                                                                                                                                                                                                                                                |
| **`SeDebug`**              | _**Адмін**_ | **PowerShell**          | Подвоїти токен `lsass.exe`.                                                                                                                                                                                                                                                                                                                   | Скрипт можна знайти на [FuzzySecurity](https://github.com/FuzzySecurity/PowerShell-Suite/blob/master/Conjure-LSASS.ps1)                                                                                                                                                                                                         |
| **`SeLoadDriver`**         | _**Адмін**_ | Інструмент стороннього розробника | <p>1. Завантажте помилковий драйвер ядра, такий як <code>szkg64.sys</code><br>2. Використовуйте вразливість драйвера<br><br>Альтернативно, привілегія може бути використана для вивантаження драйверів, пов'язаних з безпекою, за допомогою вбудованої команди <code>ftlMC</code>. наприклад: <code>fltMC sysmondrv</code></p>                                                                           | <p>1. Вразливість <code>szkg64</code> перерахована як <a href="https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-15732">CVE-2018-15732</a><br>2. Код використання <code>szkg64</code> був створений <a href="https://twitter.com/parvezghh">Parvez Anwar</a></p> |
| **`SeRestore`**            | _**Адмін**_ | **PowerShell**          | <p>1. Запустіть PowerShell/ISE з привілеєм SeRestore присутнім.<br>2. Увімкніть привілегію за допомогою <a href="https://github.com/gtworek/PSBits/blob/master/Misc/EnableSeRestorePrivilege.ps1">Enable-SeRestorePrivilege</a>).<br>3. Перейменуйте utilman.exe на utilman.old<br>4. Перейменуйте cmd.exe на utilman.exe<br>5. Заблокуйте консоль і натисніть Win+U</p> | <p>Атаку може виявити деяке антивірусне програмне забезпечення.</p><p>Альтернативний метод полягає в заміні службових бінарних файлів, збережених у "Program Files", використовуючи ту ж саму привілегію</p>                                                                                                                                                            |
| **`SeTakeOwnership`**      | _**Адмін**_ | _**Вбудовані команди**_ | <p>1. <code>takeown.exe /f "%windir%\system32"</code><br>2. <code>icalcs.exe "%windir%\system32" /grant "%username%":F</code><br>3. Перейменуйте cmd.exe на utilman.exe<br>4. Заблокуйте консоль і натисніть Win+U</p>                                                                                                                                       | <p>Атаку може виявити деяке антивірусне програмне забезпечення.</p><p>Альтернативний метод полягає в заміні службових бінарних файлів, збережених у "Program Files", використовуючи ту ж саму привілегію.</p>                                                                                                                                                           |
| **`SeTcb`**                | _**Адмін**_ | Інструмент стороннього розробника | <p>Маніпулювати токенами, щоб включити права локального адміністратора. Може знадобитися SeImpersonate.</p><p>Підлягає перевірці.</p>                                                                                                                                                                                                                                     |                                                                                                                                                                                                                                                                                                                                |

## Посилання

* Подивіться цю таблицю, що визначає токени Windows: [https://github.com/gtworek/Priv2Admin](https://github.com/gtworek/Priv2Admin)
* Ознайомтеся з [**цією статтею**](https://github.com/hatRiot/token-priv/blob/master/abusing\_token\_eop\_1.0.txt) про підвищення привілеїв за допомогою токенів.
