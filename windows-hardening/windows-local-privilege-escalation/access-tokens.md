# Токени доступу

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Ви працюєте в **кібербезпеці компанії**? Хочете побачити вашу **компанію рекламовану на HackTricks**? або хочете мати доступ до **останньої версії PEASS або завантажити HackTricks у PDF**? Перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* **Приєднуйтесь до** [**💬**](https://emojipedia.org/speech-balloon/) [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за мною на **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**репозиторію hacktricks**](https://github.com/carlospolop/hacktricks) **та** [**репозиторію hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>

## Токени доступу

Кожен **користувач, який увійшов** до системи, **має токен доступу з інформацією про безпеку** для цієї сесії входу. Система створює токен доступу, коли користувач увійшов. **Кожен процес, виконаний** від імені користувача, **має копію токена доступу**. Токен ідентифікує користувача, групи користувача та привілеї користувача. Токен також містить SID входу (Ідентифікатор безпеки), який ідентифікує поточну сесію входу.

Ви можете побачити цю інформацію, виконавши `whoami /all`
```
whoami /all

USER INFORMATION
----------------

User Name             SID
===================== ============================================
desktop-rgfrdxl\cpolo S-1-5-21-3359511372-53430657-2078432294-1001


GROUP INFORMATION
-----------------

Group Name                                                    Type             SID                                                                                                           Attributes
============================================================= ================ ============================================================================================================= ==================================================
Mandatory Label\Medium Mandatory Level                        Label            S-1-16-8192
Everyone                                                      Well-known group S-1-1-0                                                                                                       Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Local account and member of Administrators group Well-known group S-1-5-114                                                                                                     Group used for deny only
BUILTIN\Administrators                                        Alias            S-1-5-32-544                                                                                                  Group used for deny only
BUILTIN\Users                                                 Alias            S-1-5-32-545                                                                                                  Mandatory group, Enabled by default, Enabled group
BUILTIN\Performance Log Users                                 Alias            S-1-5-32-559                                                                                                  Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\INTERACTIVE                                      Well-known group S-1-5-4                                                                                                       Mandatory group, Enabled by default, Enabled group
CONSOLE LOGON                                                 Well-known group S-1-2-1                                                                                                       Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users                              Well-known group S-1-5-11                                                                                                      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization                                Well-known group S-1-5-15                                                                                                      Mandatory group, Enabled by default, Enabled group
MicrosoftAccount\cpolop@outlook.com                           User             S-1-11-96-3623454863-58364-18864-2661722203-1597581903-3158937479-2778085403-3651782251-2842230462-2314292098 Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Local account                                    Well-known group S-1-5-113                                                                                                     Mandatory group, Enabled by default, Enabled group
LOCAL                                                         Well-known group S-1-2-0                                                                                                       Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Cloud Account Authentication                     Well-known group S-1-5-64-36                                                                                                   Mandatory group, Enabled by default, Enabled group


PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                          State
============================= ==================================== ========
SeShutdownPrivilege           Shut down the system                 Disabled
SeChangeNotifyPrivilege       Bypass traverse checking             Enabled
SeUndockPrivilege             Remove computer from docking station Disabled
SeIncreaseWorkingSetPrivilege Increase a process working set       Disabled
SeTimeZonePrivilege           Change the time zone                 Disabled
```
або використовуючи _Process Explorer_ від Sysinternals (виберіть процес та перейдіть на вкладку "Безпека"):

![](<../../.gitbook/assets/image (321).png>)

### Локальний адміністратор

Коли входить локальний адміністратор, **створюються два токени доступу**: один з правами адміністратора та інший зі звичайними правами. **За замовчуванням**, коли цей користувач запускає процес, використовується той з **звичайними** (не адміністраторськими) **правами**. Коли цей користувач намагається **виконати** що-небудь **як адміністратор** (наприклад, "Виконати в якості адміністратора"), буде використано **UAC**, щоб запитати дозвіл.\
Якщо ви хочете [**дізнатися більше про UAC, прочитайте цю сторінку**](../authentication-credentials-uac-and-efs.md#uac)**.**

### Імітація облікових даних користувача

Якщо у вас є **дійсні облікові дані будь-якого іншого користувача**, ви можете **створити** нову **сесію входу** з цими обліковими даними:
```
runas /user:domain\username cmd.exe
```
**Токен доступу** також має **посилання** на сеанси входу в **LSASS**, це корисно, якщо процесу потрібен доступ до деяких об'єктів мережі.\
Ви можете запустити процес, який **використовує різні облікові дані для доступу до мережевих служб**, використовуючи:
```
runas /user:domain\username /netonly cmd.exe
```
### Типи токенів

Існує два типи доступних токенів:

* **Основний токен**: Він служить як представлення облікових даних безпеки процесу. Створення та асоціація основних токенів з процесами - це дії, які вимагають підвищених привілеїв, підкреслюючи принцип розділення привілеїв. Зазвичай, за створення токену відповідає служба аутентифікації, тоді як його асоціацією з оболонкою операційної системи користувача займається служба входу в систему. Важливо зауважити, що процеси успадковують основний токен свого батьківського процесу при створенні.

* **Токен імперсонації**: Надає можливість серверній програмі тимчасово приймати ідентичність клієнта для доступу до захищених об'єктів. Цей механізм розглядається на чотирьох рівнях операцій:
- **Анонімний**: Надає серверу доступ, схожий на доступ невідомого користувача.
- **Ідентифікація**: Дозволяє серверу перевірити ідентичність клієнта без використання її для доступу до об'єктів.
- **Імперсонація**: Дозволяє серверу працювати в ідентичності клієнта.
- **Делегування**: Схоже на Імперсонацію, але включає можливість розширити це припущення ідентичності до віддалених систем, з якими взаємодіє сервер, забезпечуючи збереження облікових даних.

#### Імітація токенів

Використовуючи модуль _**incognito**_ у metasploit, якщо у вас достатньо привілеїв, ви можете легко **переглядати** та **імітувати** інші **токени**. Це може бути корисним для виконання **дій, ніби ви були іншим користувачем**. Ви також можете **підвищити привілеї** за допомогою цієї техніки.

### Привілеї токенів

Дізнайтеся, які **привілеї токенів можна використовувати для підвищення привілеїв:**

{% content-ref url="privilege-escalation-abusing-tokens/" %}
[privilege-escalation-abusing-tokens](privilege-escalation-abusing-tokens/)
{% endcontent-ref %}

Подивіться [**всі можливі привілеї токенів та деякі визначення на цій зовнішній сторінці**](https://github.com/gtworek/Priv2Admin).

## Посилання

Дізнайтеся більше про токени в цих посібниках: [https://medium.com/@seemant.bisht24/understanding-and-abusing-process-tokens-part-i-ee51671f2cfa](https://medium.com/@seemant.bisht24/understanding-and-abusing-process-tokens-part-i-ee51671f2cfa) та [https://medium.com/@seemant.bisht24/understanding-and-abusing-access-tokens-part-ii-b9069f432962](https://medium.com/@seemant.bisht24/understanding-and-abusing-access-tokens-part-ii-b9069f432962)
