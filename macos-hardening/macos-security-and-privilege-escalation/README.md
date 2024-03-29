# Безпека та підвищення привілеїв в macOS

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію рекламовану на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Дізнайтеся про [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв GitHub.

</details>

<figure><img src="../../.gitbook/assets/image (1) (3) (1).png" alt=""><figcaption></figcaption></figure>

Приєднуйтесь до сервера [**HackenProof Discord**](https://discord.com/invite/N3FrSbmwdy), щоб спілкуватися з досвідченими хакерами та мисливцями за багами!

**Інсайти щодо Хакінгу**\
Взаємодійте з контентом, який досліджує захоплення та виклики хакінгу

**Новини про Хакінг у Реальному Часі**\
Будьте в курсі швидкозмінного світу хакінгу завдяки новинам та інсайтам у реальному часі

**Останні Оголошення**\
Будьте в курсі нових баг-баунті, які запускаються, та важливих оновлень платформи

**Приєднуйтесь до нас на** [**Discord**](https://discord.com/invite/N3FrSbmwdy) та почніть співпрацювати з топовими хакерами вже сьогодні!

## Основи MacOS

Якщо ви не знайомі з macOS, вам слід почати вивчати основи macOS:

* Спеціальні файли та дозволи macOS:

{% content-ref url="macos-files-folders-and-binaries/" %}
[macos-files-folders-and-binaries](macos-files-folders-and-binaries/)
{% endcontent-ref %}

* Загальні користувачі macOS

{% content-ref url="macos-users.md" %}
[macos-users.md](macos-users.md)
{% endcontent-ref %}

* **AppleFS**

{% content-ref url="macos-applefs.md" %}
[macos-applefs.md](macos-applefs.md)
{% endcontent-ref %}

* Архітектура ядра

{% content-ref url="mac-os-architecture/" %}
[mac-os-architecture](mac-os-architecture/)
{% endcontent-ref %}

* Загальні мережеві служби та протоколи macOS

{% content-ref url="macos-protocols.md" %}
[macos-protocols.md](macos-protocols.md)
{% endcontent-ref %}

* **Opensource** macOS: [https://opensource.apple.com/](https://opensource.apple.com/)
* Для завантаження `tar.gz` змініть URL, такий як [https://opensource.apple.com/**source**/dyld/](https://opensource.apple.com/source/dyld/), на [https://opensource.apple.com/**tarballs**/dyld/**dyld-852.2.tar.gz**](https://opensource.apple.com/tarballs/dyld/dyld-852.2.tar.gz)

### MacOS MDM

У компаніях **системи macOS ймовірно будуть керуватися MDM**. Тому з погляду атакуючого важливо знати **як це працює**:

{% content-ref url="../macos-red-teaming/macos-mdm/" %}
[macos-mdm](../macos-red-teaming/macos-mdm/)
{% endcontent-ref %}

### MacOS - Інспектування, Налагодження та Fuzzing

{% content-ref url="macos-apps-inspecting-debugging-and-fuzzing/" %}
[macos-apps-inspecting-debugging-and-fuzzing](macos-apps-inspecting-debugging-and-fuzzing/)
{% endcontent-ref %}

## Захист в MacOS

{% content-ref url="macos-security-protections/" %}
[macos-security-protections](macos-security-protections/)
{% endcontent-ref %}

## Поверхня Атаки

### Дозволи на Файли

Якщо **процес, який працює в якості root, записує** файл, яким може керувати користувач, користувач може скористатися цим для **підвищення привілеїв**.\
Це може статися в таких ситуаціях:

* Файл, який вже був створений користувачем (належить користувачеві)
* Файл може бути записаний користувачем через групу
* Файл знаходиться всередині каталогу, який належить користувачеві (користувач може створити файл)
* Файл знаходиться всередині каталогу, який належить root, але користувач має права на запис через групу (користувач може створити файл)

Можливість **створення файлу**, який буде **використовуватися root**, дозволяє користувачеві **використовувати його вміст** або навіть створювати **символічні посилання/жорсткі посилання**, щоб вказати його в інше місце.

Для цього типу уразливостей не забувайте **перевіряти вразливі встановлювачі `.pkg`**:

{% content-ref url="macos-files-folders-and-binaries/macos-installers-abuse.md" %}
[macos-installers-abuse.md](macos-files-folders-and-binaries/macos-installers-abuse.md)
{% endcontent-ref %}

### Розширення Файлів та Обробники URL-схем додатків

Дивні додатки, зареєстровані за допомогою розширень файлів, можуть бути використані, і різні програми можуть бути зареєстровані для відкриття конкретних протоколів

{% content-ref url="macos-file-extension-apps.md" %}
[macos-file-extension-apps.md](macos-file-extension-apps.md)
{% endcontent-ref %}

## Підвищення Привілеїв TCC / SIP в macOS

У macOS **додатки та бінарні файли можуть мати дозволи** на доступ до папок або налаштувань, що робить їх більш привілейованими, ніж інші.

Отже, атакуючому, який хоче успішно скомпрометувати машину macOS, потрібно **підвищити свої привілеї TCC** (або навіть **обійти SIP**, залежно від його потреб).

Ці привілеї зазвичай надаються у формі **привілеїв**, з якими додаток підписаний, або додаток може запитати деякі доступи, і після **затвердження користувачем** їх можна знайти в **базах даних TCC**. Інший спосіб, яким процес може отримати ці привілеї, - це бути **дитиною процесу** з цими **привілеями**, оскільки вони зазвичай **успадковуються**.

Перейдіть за цими посиланнями, щоб знайти різні способи [**підвищення привілеїв в TCC**](macos-security-protections/macos-tcc/#tcc-privesc-and-bypasses), для [**обходу TCC**](macos-security-protections/macos-tcc/macos-tcc-bypasses/) та як у минулому [**було обійдено SIP**](macos-security-protections/macos-sip.md#sip-bypasses).

## Традиційне Підвищення Привілеїв в macOS

Звісно, з погляду червоних команд вам також слід зацікавитися підвищенням до root. Перевірте наступний пост для деяких підказок:

{% content-ref url="macos-privilege-escalation.md" %}
[macos-privilege-escalation.md](macos-privilege-escalation.md)
{% endcontent-ref %}
## Посилання

* [**OS X Incident Response: Scripting and Analysis**](https://www.amazon.com/OS-Incident-Response-Scripting-Analysis-ebook/dp/B01FHOHHVS)
* [**https://taomm.org/vol1/analysis.html**](https://taomm.org/vol1/analysis.html)
* [**https://github.com/NicolasGrimonpont/Cheatsheet**](https://github.com/NicolasGrimonpont/Cheatsheet)
* [**https://assets.sentinelone.com/c/sentinal-one-mac-os-?x=FvGtLJ**](https://assets.sentinelone.com/c/sentinal-one-mac-os-?x=FvGtLJ)
* [**https://www.youtube.com/watch?v=vMGiplQtjTY**](https://www.youtube.com/watch?v=vMGiplQtjTY)

<figure><img src="../../.gitbook/assets/image (1) (3) (1).png" alt=""><figcaption></figcaption></figure>

Приєднуйтесь до сервера [**HackenProof Discord**](https://discord.com/invite/N3FrSbmwdy), щоб спілкуватися з досвідченими хакерами та мисливцями за багами!

**Інсайти щодо Хакінгу**\
Взаємодійте з контентом, який досліджує захоплення та виклики хакінгу

**Новини про Хакінг у Реальному Часі**\
Будьте в курсі швидкозмінного світу хакінгу завдяки новинам та інсайтам у реальному часі

**Останні Оголошення**\
Будьте в курсі найновіших запусків баг баунті та важливих оновлень платформи

**Приєднуйтесь до нас на** [**Discord**](https://discord.com/invite/N3FrSbmwdy) та почніть співпрацювати з топовими хакерами вже сьогодні!

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити свою **компанію рекламовану в HackTricks** або **завантажити HackTricks у PDF** Перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github репозиторіїв.

</details>
