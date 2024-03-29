# Робота з червоним командуванням macOS

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію в рекламі на HackTricks** або **завантажити HackTricks у PDF** Перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв GitHub.

</details>

## Зловживання MDM

* JAMF Pro: `jamf checkJSSConnection`
* Kandji

Якщо ви змогли **скомпрометувати адміністративні облікові дані** для доступу до платформи управління, ви можете **потенційно скомпрометувати всі комп'ютери**, розповсюджуючи своє шкідливе ПЗ на машинах.

Для червоного командування в середовищах MacOS високо рекомендується мати розуміння того, як працюють MDM:

{% content-ref url="macos-mdm/" %}
[macos-mdm](macos-mdm/)
{% endcontent-ref %}

### Використання MDM як C2

MDM матиме дозвіл на встановлення, запит або видалення профілів, встановлення додатків, створення локальних облікових записів адміністратора, встановлення пароля для прошивки, зміну ключа FileVault...

Для запуску власного MDM вам потрібно **ваш CSR підписаний вендором**, який ви можете спробувати отримати за допомогою [**https://mdmcert.download/**](https://mdmcert.download/). І для запуску власного MDM для пристроїв Apple ви можете використовувати [**MicroMDM**](https://github.com/micromdm/micromdm).

Однак, для встановлення додатка на зареєстрований пристрій, вам все ще потрібно, щоб його підписав рахунок розробника... однак, під час реєстрації в MDM **пристрій додає сертифікат SSL MDM як довірений ЦС**, тому ви тепер можете підписувати все.

Для реєстрації пристрою в MDM вам потрібно встановити файл **`mobileconfig`** як root, який може бути доставлений через файл **pkg** (ви можете стиснути його в zip, і коли завантажуєте з safari, він буде розпакований).

**Агент Mythic Orthrus** використовує цей метод.

### Зловживання JAMF PRO

JAMF може виконувати **власні скрипти** (скрипти, розроблені системним адміністратором), **нативні навантаження** (створення локальних облікових записів, встановлення пароля EFI, моніторинг файлів/процесів...) та **MDM** (конфігурація пристроїв, сертифікати пристроїв...).

#### Самозареєстрація JAMF

Перейдіть на сторінку, таку як `https://<company-name>.jamfcloud.com/enroll/`, щоб перевірити, чи вони мають **ввімкнену самозареєстрацію**. Якщо вони мають, це може **запитати облікові дані для доступу**.

Ви можете використовувати скрипт [**JamfSniper.py**](https://github.com/WithSecureLabs/Jamf-Attack-Toolkit/blob/master/JamfSniper.py) для виконання атаки зі спробою пароля.

Більше того, після знаходження відповідних облікових даних ви можете спробувати перебрати інші імена користувачів за наступною формою:

![](<../../.gitbook/assets/image (7) (1) (1).png>)

#### Аутентифікація пристрою JAMF

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

**`jamf`** бінарний файл містив секрет для відкриття keychain, який на момент відкриття був **розповсюджений** серед всіх і був: **`jk23ucnq91jfu9aj`**.\
Більше того, jamf **зберігається** як **LaunchDaemon** в **`/Library/LaunchAgents/com.jamf.management.agent.plist`**

#### Захоплення пристрою JAMF

URL **JSS** (Jamf Software Server), який використовуватиме **`jamf`**, розташований в **`/Library/Preferences/com.jamfsoftware.jamf.plist`**.\
Цей файл в основному містить URL:

{% code overflow="wrap" %}
```bash
plutil -convert xml1 -o - /Library/Preferences/com.jamfsoftware.jamf.plist

[...]
<key>is_virtual_machine</key>
<false/>
<key>jss_url</key>
<string>https://halbornasd.jamfcloud.com/</string>
<key>last_management_framework_change_id</key>
<integer>4</integer>
[...]
```
Отже, зловмисник може розмістити шкідливий пакет (`pkg`), який **перезапише цей файл** при встановленні, встановивши **URL на прослуховувач Mythic C2 від агента Typhon**, щоб тепер мати можливість використовувати JAMF як C2.
```bash
# After changing the URL you could wait for it to be reloaded or execute:
sudo jamf policy -id 0

# TODO: There is an ID, maybe it's possible to have the real jamf connection and another one to the C2
```
{% endcode %}

#### Підробка JAMF

Для **підробки комунікації** між пристроєм та JMF вам потрібно:

* **UUID** пристрою: `ioreg -d2 -c IOPlatformExpertDevice | awk -F" '/IOPlatformUUID/{print $(NF-1)}'`
* **Ключ JAMF** з: `/Library/Application\ Support/Jamf/JAMF.keychain`, який містить сертифікат пристрою

З цією інформацією **створіть віртуальну машину** з **викраденим** апаратним **UUID** та з **вимкненим SIP**, скиньте **ключ JAMF**, **підключіться** до агента Jamf та вкрадіть його інформацію.

#### Викрадення секретів

<figure><img src="../../.gitbook/assets/image (11).png" alt=""><figcaption><p>a</p></figcaption></figure>

Ви також можете моніторити розташування `/Library/Application Support/Jamf/tmp/` для **власних сценаріїв**, які адміністратори можуть хотіти виконати через Jamf, оскільки вони **розміщуються тут, виконуються та видаляються**. Ці сценарії **можуть містити облікові дані**.

Однак **облікові дані** можуть передаватися через ці сценарії як **параметри**, тому вам потрібно буде моніторити `ps aux | grep -i jamf` (навіть не бути root).

Сценарій [**JamfExplorer.py**](https://github.com/WithSecureLabs/Jamf-Attack-Toolkit/blob/master/JamfExplorer.py) може слухати нові файли, які додаються та нові аргументи процесу.

### Віддалений доступ до macOS

Також про "спеціальні" **мережеві** **протоколи** **MacOS**:

{% content-ref url="../macos-security-and-privilege-escalation/macos-protocols.md" %}
[macos-protocols.md](../macos-security-and-privilege-escalation/macos-protocols.md)
{% endcontent-ref %}

## Active Directory

У деяких випадках ви знайдете, що **комп'ютер MacOS підключений до AD**. У цьому сценарії вам слід спробувати **перелічити** активний каталог, як ви звикли. Знайдіть допомогу на наступних сторінках:

{% content-ref url="../../network-services-pentesting/pentesting-ldap.md" %}
[pentesting-ldap.md](../../network-services-pentesting/pentesting-ldap.md)
{% endcontent-ref %}

{% content-ref url="../../windows-hardening/active-directory-methodology/" %}
[active-directory-methodology](../../windows-hardening/active-directory-methodology/)
{% endcontent-ref %}

{% content-ref url="../../network-services-pentesting/pentesting-kerberos-88/" %}
[pentesting-kerberos-88](../../network-services-pentesting/pentesting-kerberos-88/)
{% endcontent-ref %}

Деякі **локальні інструменти MacOS**, які також можуть вам допомогти, - це `dscl`:
```bash
dscl "/Active Directory/[Domain]/All Domains" ls /
```
Також є деякі інструменти, підготовлені для MacOS для автоматичного переліку AD та гри з kerberos:

* [**Machound**](https://github.com/XMCyber/MacHound): MacHound - це розширення для інструменту аудитування Bloodhound, яке дозволяє збирати та вживати взаємовідносини Active Directory на хостах MacOS.
* [**Bifrost**](https://github.com/its-a-feature/bifrost): Bifrost - це проект на Objective-C, призначений для взаємодії з API Heimdal krb5 на macOS. Метою проекту є покращення тестування безпеки навколо Kerberos на пристроях macOS за допомогою вбудованих API без потреби в будь-якому іншому фреймворку або пакетах на цільовому пристрої.
* [**Orchard**](https://github.com/its-a-feature/Orchard): Інструмент JavaScript для автоматизації (JXA) для переліку Active Directory.
```bash
echo show com.apple.opendirectoryd.ActiveDirectory | scutil
```
### Користувачі

Три типи користувачів MacOS:

* **Локальні користувачі** — Керуються локальним сервісом OpenDirectory, вони не пов'язані з Active Directory.
* **Мережеві користувачі** — Волатильні користувачі Active Directory, які потребують підключення до сервера DC для аутентифікації.
* **Мобільні користувачі** — Користувачі Active Directory з локальним резервним копіюванням своїх облікових даних та файлів.

Локальна інформація про користувачів та групи зберігається в папці _/var/db/dslocal/nodes/Default._\
Наприклад, інформація про користувача з ім'ям _mark_ зберігається в _/var/db/dslocal/nodes/Default/users/mark.plist_, а інформація про групу _admin_ знаходиться в _/var/db/dslocal/nodes/Default/groups/admin.plist_.

Крім використання ребер HasSession та AdminTo, **MacHound додає три нові ребра** до бази даних Bloodhound:

* **CanSSH** - сутність, якій дозволено SSH на хост
* **CanVNC** - сутність, якій дозволено VNC на хост
* **CanAE** - сутність, якій дозволено виконувати скрипти AppleEvent на хості
```bash
#User enumeration
dscl . ls /Users
dscl . read /Users/[username]
dscl "/Active Directory/TEST/All Domains" ls /Users
dscl "/Active Directory/TEST/All Domains" read /Users/[username]
dscacheutil -q user

#Computer enumeration
dscl "/Active Directory/TEST/All Domains" ls /Computers
dscl "/Active Directory/TEST/All Domains" read "/Computers/[compname]$"

#Group enumeration
dscl . ls /Groups
dscl . read "/Groups/[groupname]"
dscl "/Active Directory/TEST/All Domains" ls /Groups
dscl "/Active Directory/TEST/All Domains" read "/Groups/[groupname]"

#Domain Information
dsconfigad -show
```
Додаткова інформація за посиланням [https://its-a-feature.github.io/posts/2018/01/Active-Directory-Discovery-with-a-Mac/](https://its-a-feature.github.io/posts/2018/01/Active-Directory-Discovery-with-a-Mac/)

## Доступ до Ключового сховища

Ключове сховище, ймовірно, містить чутливу інформацію, яка, якщо доступна без виклику запиту, може допомогти в проведенні червоного командного вправи:

{% content-ref url="macos-keychain.md" %}
[macos-keychain.md](macos-keychain.md)
{% endcontent-ref %}

## Зовнішні сервіси

Червоний командний редагування MacOS відрізняється від звичайного червоного командного редагування Windows, оскільки зазвичай **MacOS інтегрований з кількома зовнішніми платформами безпосередньо**. Звичайна конфігурація MacOS передбачає доступ до комп'ютера за допомогою **синхронізованих облікових даних OneLogin та доступ до кількох зовнішніх сервісів** (наприклад, github, aws...) через OneLogin.

## Різноманітні техніки червоного командного редагування

### Safari

Коли файл завантажується в Safari, якщо це "безпечний" файл, він буде **автоматично відкритий**. Так, наприклад, якщо ви **завантажуєте zip**, він буде автоматично розпакований:

<figure><img src="../../.gitbook/assets/image (12) (3).png" alt=""><figcaption></figcaption></figure>

## Посилання

* [**https://www.youtube.com/watch?v=IiMladUbL6E**](https://www.youtube.com/watch?v=IiMladUbL6E)
* [**https://medium.com/xm-cyber/introducing-machound-a-solution-to-macos-active-directory-based-attacks-2a425f0a22b6**](https://medium.com/xm-cyber/introducing-machound-a-solution-to-macos-active-directory-based-attacks-2a425f0a22b6)
* [**https://gist.github.com/its-a-feature/1a34f597fb30985a2742bb16116e74e0**](https://gist.github.com/its-a-feature/1a34f597fb30985a2742bb16116e74e0)
* [**Come to the Dark Side, We Have Apples: Turning macOS Management Evil**](https://www.youtube.com/watch?v=pOQOh07eMxY)
* [**OBTS v3.0: "An Attackers Perspective on Jamf Configurations" - Luke Roberts / Calum Hall**](https://www.youtube.com/watch?v=ju1IYWUv4ZA)
