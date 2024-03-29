# macOS Gatekeeper / Quarantine / XProtect

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Ви працюєте в **кібербезпецівій компанії**? Хочете побачити вашу **компанію в рекламі на HackTricks**? або хочете мати доступ до **останньої версії PEASS або завантажити HackTricks у PDF**? Перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Дізнайтеся про [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* **Приєднуйтесь до** [**💬**](https://emojipedia.org/speech-balloon/) [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за мною на **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Поділіться своїми хакінг-трюками, надсилайте PR до** [**репозиторію hacktricks**](https://github.com/carlospolop/hacktricks) **та** [**репозиторію hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud)
*
* .

</details>

## Gatekeeper

**Gatekeeper** - це функція безпеки, розроблена для операційних систем Mac, яка призначена для того, щоб користувачі **запускали лише довірене програмне забезпечення** на своїх системах. Вона працює, **перевіряючи програмне забезпечення**, яке користувач завантажує та намагається відкрити з **джерел поза App Store**, таких як додаток, плагін або пакет встановлення.

Основний механізм Gatekeeper полягає в його **перевірці**. Він перевіряє, чи програмне забезпечення, яке завантажується, **підписане визнаним розробником**, що гарантує автентичність програмного забезпечення. Крім того, він переконується, що програмне забезпечення **пройшло процедуру нотаріальної перевірки від Apple**, підтверджуючи, що воно не містить відомого шкідливого вмісту і не було змінено після нотаріальної перевірки.

Крім того, Gatekeeper посилює контроль користувача та безпеку, **просить користувачів схвалити відкриття** завантаженого програмного забезпечення вперше. Цей захист допомагає запобігти користувачам ненавмисному запуску потенційно шкідливого виконавчого коду, який вони могли помилково вважати безпечним файлом даних.

### Підписи програм

Підписи програм, також відомі як кодові підписи, є важливою складовою інфраструктури безпеки Apple. Вони використовуються для **перевірки ідентичності автора програмного забезпечення** (розробника) та для забезпечення того, що код не був змінений з моменту останнього підпису.

Ось як це працює:

1. **Підписання програми:** Коли розробник готовий поширити свою програму, він **підписує програму за допомогою приватного ключа**. Цей приватний ключ пов'язаний з **сертифікатом, який Apple видає розробнику** при вступі до програми розробника Apple. Процес підпису включає створення криптографічного хешу всіх частин програми та шифрування цього хешу приватним ключем розробника.
2. **Поширення програми:** Підписана програма потім поширюється користувачам разом з сертифікатом розробника, який містить відповідний публічний ключ.
3. **Перевірка програми:** Коли користувач завантажує та намагається запустити програму, їх операційна система Mac використовує публічний ключ з сертифіката розробника для розшифрування хешу. Потім вона повторно обчислює хеш на основі поточного стану програми та порівнює його з розшифрованим хешем. Якщо вони співпадають, це означає, що **програма не була змінена** з моменту підпису розробника, і система дозволяє програмі запуститися.

Підписи програм є важливою частиною технології Gatekeeper від Apple. Коли користувач намагається **відкрити програму, завантажену з Інтернету**, Gatekeeper перевіряє підпис програми. Якщо вона підписана сертифікатом, виданим Apple відомому розробнику, і код не був змінений, Gatekeeper дозволяє програмі запуститися. В іншому випадку він блокує програму та повідомляє користувача.

Починаючи з macOS Catalina, **Gatekeeper також перевіряє, чи програма пройшла процедуру нотаріальної перевірки** від Apple, додавши додатковий рівень безпеки. Процес нотаріальної перевірки перевіряє програму на відомі проблеми безпеки та шкідливий код, і якщо ці перевірки пройдуть успішно, Apple додає квиток до програми, який може перевірити Gatekeeper.

#### Перевірка підписів

При перевірці деякого **зразка шкідливого програмного забезпечення** ви завжди повинні **перевіряти підпис** бінарного файлу, оскільки **розробник**, який його підписав, може вже бути **пов'язаний** з **шкідливим програмним забезпеченням**.
```bash
# Get signer
codesign -vv -d /bin/ls 2>&1 | grep -E "Authority|TeamIdentifier"

# Check if the app’s contents have been modified
codesign --verify --verbose /Applications/Safari.app

# Get entitlements from the binary
codesign -d --entitlements :- /System/Applications/Automator.app # Check the TCC perms

# Check if the signature is valid
spctl --assess --verbose /Applications/Safari.app

# Sign a binary
codesign -s <cert-name-keychain> toolsdemo
```
### Нотаризація

Процес нотаризації Apple служить додатковим заходом для захисту користувачів від потенційно шкідливого програмного забезпечення. Це включає **розробника, який подає свою заявку на розгляд** до **Служби нотаріату Apple**, яку не слід плутати з оглядом додатків. Ця служба є **автоматизованою системою**, яка аналізує подане програмне забезпечення на наявність **шкідливого вмісту** та будь-яких потенційних проблем з підписуванням коду.

Якщо програмне забезпечення **пройшло** цю перевірку без підняття будь-яких побоїв, Служба нотаріату генерує нотаріальний квиток. Після цього розробнику потрібно **долучити цей квиток до свого програмного забезпечення**, процес відомий як 'шиття'. Крім того, нотаріальний квиток також публікується в Інтернеті, де Gatekeeper, технологія безпеки Apple, може отримати до нього доступ.

При першій установці або запуску програмного забезпечення користувача, наявність нотаріального квитка - чи то пришитого до виконавчого файлу, чи знайденого в Інтернеті - **повідомляє Gatekeeper, що програмне забезпечення було нотаріально засвідчено Apple**. У результаті Gatekeeper відображає описове повідомлення в діалозі першого запуску, вказуючи, що програмне забезпечення пройшло перевірку на наявність шкідливого вмісту від Apple. Цей процес підвищує довіру користувачів до безпеки програмного забезпечення, яке вони встановлюють або запускають на своїх системах.

### Перелік GateKeeper

GateKeeper - це **кілька компонентів безпеки**, які запобігають запуску ненадійних додатків та також **один з компонентів**.

Можливо переглянути **стан** GateKeeper за допомогою:
```bash
# Check the status
spctl --status
```
{% hint style="danger" %}
Зверніть увагу, що перевірки підпису GateKeeper виконуються лише для **файлів з атрибутом Quarantine**, а не для кожного файлу.
{% endhint %}

GateKeeper перевірить, чи згідно з **налаштуваннями та підписом** бінарний файл може бути виконаний:

<figure><img src="../../../.gitbook/assets/image (678).png" alt=""><figcaption></figcaption></figure>

База даних, яка зберігає цю конфігурацію, розташована в **`/var/db/SystemPolicy`**. Ви можете перевірити цю базу даних як root за допомогою:
```bash
# Open database
sqlite3 /var/db/SystemPolicy

# Get allowed rules
SELECT requirement,allow,disabled,label from authority where label != 'GKE' and disabled=0;
requirement|allow|disabled|label
anchor apple generic and certificate 1[subject.CN] = "Apple Software Update Certification Authority"|1|0|Apple Installer
anchor apple|1|0|Apple System
anchor apple generic and certificate leaf[field.1.2.840.113635.100.6.1.9] exists|1|0|Mac App Store
anchor apple generic and certificate 1[field.1.2.840.113635.100.6.2.6] exists and (certificate leaf[field.1.2.840.113635.100.6.1.14] or certificate leaf[field.1.2.840.113635.100.6.1.13]) and notarized|1|0|Notarized Developer ID
[...]
```
Зверніть увагу, як перше правило закінчилося на "**App Store**", а друге на "**Developer ID**", і що на попередньому зображенні було **увімкнено виконання додатків з App Store та ідентифікованих розробників**. Якщо ви **зміните** цей параметр на App Store, то правила "**Notarized Developer ID" зникнуть**.

Також існує тисячі правил типу **GKE**:
```bash
SELECT requirement,allow,disabled,label from authority where label = 'GKE' limit 5;
cdhash H"b40281d347dc574ae0850682f0fd1173aa2d0a39"|1|0|GKE
cdhash H"5fd63f5342ac0c7c0774ebcbecaf8787367c480f"|1|0|GKE
cdhash H"4317047eefac8125ce4d44cab0eb7b1dff29d19a"|1|0|GKE
cdhash H"0a71962e7a32f0c2b41ddb1fb8403f3420e1d861"|1|0|GKE
cdhash H"8d0d90ff23c3071211646c4c9c607cdb601cb18f"|1|0|GKE
```
Це хеші, які походять з **`/var/db/SystemPolicyConfiguration/gke.bundle/Contents/Resources/gke.auth`, `/var/db/gke.bundle/Contents/Resources/gk.db`** та **`/var/db/gkopaque.bundle/Contents/Resources/gkopaque.db`**

Або ви можете перелічити попередню інформацію за допомогою:
```bash
sudo spctl --list
```
Опції **`--master-disable`** та **`--global-disable`** команди **`spctl`** повністю **вимкнуть** перевірку цих підписів:
```bash
# Disable GateKeeper
spctl --global-disable
spctl --master-disable

# Enable it
spctl --global-enable
spctl --master-enable
```
Коли повністю увімкнено, з'явиться нова опція:

<figure><img src="../../../.gitbook/assets/image (679).png" alt=""><figcaption></figcaption></figure>

Можливо **перевірити, чи дозволить GateKeeper використання додатку** за допомогою:
```bash
spctl --assess -v /Applications/App.app
```
Можливо додати нові правила в GateKeeper для дозволу виконання певних додатків за допомогою:
```bash
# Check if allowed - nop
spctl --assess -v /Applications/App.app
/Applications/App.app: rejected
source=no usable signature

# Add a label and allow this label in GateKeeper
sudo spctl --add --label "whitelist" /Applications/App.app
sudo spctl --enable --label "whitelist"

# Check again - yep
spctl --assess -v /Applications/App.app
/Applications/App.app: accepted
```
### Файли карантину

Після **завантаження** додатку або файлу, певні **додатки macOS**, такі як веб-браузери або поштові клієнти, **додають розширений атрибут файлу**, відомий як "**прапор карантину**", до завантаженого файлу. Цей атрибут діє як захисний захід, щоб **позначити файл** як походять з ненадійного джерела (інтернету) і, можливо, нести ризики. Однак не всі додатки додають цей атрибут, наприклад, звичайне програмне забезпечення клієнта BitTorrent зазвичай обходить цей процес.

**Наявність прапорця карантину сигналізує про функцію безпеки Gatekeeper macOS, коли користувач намагається виконати файл**.

У випадку, коли **прапорець карантину відсутній** (як у випадку файлів, завантажених через деякі клієнти BitTorrent), **перевірки Gatekeeper можуть не виконуватися**. Таким чином, користувачам слід бути обережними при відкритті файлів, завантажених з менш безпечних або невідомих джерел.

{% hint style="info" %}
**Перевірка** валідності **підписів коду** - це **ресурсоємний** процес, який включає генерацію криптографічних **хешів** коду та всіх його пакетованих ресурсів. Крім того, перевірка валідності сертифіката передбачає виконання **онлайн-перевірки** на серверах Apple, щоб переконатися, чи був він скасований після видачі. З цих причин повна перевірка підпису коду та його підтвердження є **непрактичною для запуску кожного разу при запуску додатка**.

Тому ці перевірки **виконуються лише при виконанні додатків з атрибутом карантину**.
{% endhint %}

{% hint style="warning" %}
Цей атрибут повинен бути **встановлений додатком, який створює/завантажує** файл.

Однак файли, які знаходяться в пісочниці, матимуть цей атрибут встановленим для кожного файлу, який вони створюють. І непісочні додатки можуть встановити його самостійно або вказати ключ [**LSFileQuarantineEnabled**](https://developer.apple.com/documentation/bundleresources/information\_property\_list/lsfilequarantineenabled?language=objc) в **Info.plist**, що змусить систему встановити розширений атрибут `com.apple.quarantine` на створені файли.
{% endhint %}

Можливо **перевірити його статус та увімкнути/вимкнути** (потрібні права адміністратора) за допомогою:
```bash
spctl --status
assessments enabled

spctl --enable
spctl --disable
#You can also allow nee identifies to execute code using the binary "spctl"
```
Ви також можете **знайти, чи має файл розширений атрибут карантину** за допомогою:
```bash
xattr file.png
com.apple.macl
com.apple.quarantine
```
Перевірте **значення** **розширених** **атрибутів** та дізнайтеся, яка програма записала атрибут карантину за допомогою:
```bash
xattr -l portada.png
com.apple.macl:
00000000  03 00 53 DA 55 1B AE 4C 4E 88 9D CA B7 5C 50 F3  |..S.U..LN.....P.|
00000010  16 94 03 00 27 63 64 97 98 FB 4F 02 84 F3 D0 DB  |....'cd...O.....|
00000020  89 53 C3 FC 03 00 27 63 64 97 98 FB 4F 02 84 F3  |.S....'cd...O...|
00000030  D0 DB 89 53 C3 FC 00 00 00 00 00 00 00 00 00 00  |...S............|
00000040  00 00 00 00 00 00 00 00                          |........|
00000048
com.apple.quarantine: 00C1;607842eb;Brave;F643CD5F-6071-46AB-83AB-390BA944DEC5
# 00c1 -- It has been allowed to eexcute this file (QTN_FLAG_USER_APPROVED = 0x0040)
# 607842eb -- Timestamp
# Brave -- App
# F643CD5F-6071-46AB-83AB-390BA944DEC5 -- UID assigned to the file downloaded
```
Фактично процес може встановлювати прапори карантину для створених ним файлів (я намагався застосувати прапор USER\_APPROVED до створеного файлу, але він не застосовується):

<details>

<summary>Вихідний код застосування прапорів карантину</summary>
```c
#include <stdio.h>
#include <stdlib.h>

enum qtn_flags {
QTN_FLAG_DOWNLOAD = 0x0001,
QTN_FLAG_SANDBOX = 0x0002,
QTN_FLAG_HARD = 0x0004,
QTN_FLAG_USER_APPROVED = 0x0040,
};

#define qtn_proc_alloc _qtn_proc_alloc
#define qtn_proc_apply_to_self _qtn_proc_apply_to_self
#define qtn_proc_free _qtn_proc_free
#define qtn_proc_init _qtn_proc_init
#define qtn_proc_init_with_self _qtn_proc_init_with_self
#define qtn_proc_set_flags _qtn_proc_set_flags
#define qtn_file_alloc _qtn_file_alloc
#define qtn_file_init_with_path _qtn_file_init_with_path
#define qtn_file_free _qtn_file_free
#define qtn_file_apply_to_path _qtn_file_apply_to_path
#define qtn_file_set_flags _qtn_file_set_flags
#define qtn_file_get_flags _qtn_file_get_flags
#define qtn_proc_set_identifier _qtn_proc_set_identifier

typedef struct _qtn_proc *qtn_proc_t;
typedef struct _qtn_file *qtn_file_t;

int qtn_proc_apply_to_self(qtn_proc_t);
void qtn_proc_init(qtn_proc_t);
int qtn_proc_init_with_self(qtn_proc_t);
int qtn_proc_set_flags(qtn_proc_t, uint32_t flags);
qtn_proc_t qtn_proc_alloc();
void qtn_proc_free(qtn_proc_t);
qtn_file_t qtn_file_alloc(void);
void qtn_file_free(qtn_file_t qf);
int qtn_file_set_flags(qtn_file_t qf, uint32_t flags);
uint32_t qtn_file_get_flags(qtn_file_t qf);
int qtn_file_apply_to_path(qtn_file_t qf, const char *path);
int qtn_file_init_with_path(qtn_file_t qf, const char *path);
int qtn_proc_set_identifier(qtn_proc_t qp, const char* bundleid);

int main() {

qtn_proc_t qp = qtn_proc_alloc();
qtn_proc_set_identifier(qp, "xyz.hacktricks.qa");
qtn_proc_set_flags(qp, QTN_FLAG_DOWNLOAD | QTN_FLAG_USER_APPROVED);
qtn_proc_apply_to_self(qp);
qtn_proc_free(qp);

FILE *fp;
fp = fopen("thisisquarantined.txt", "w+");
fprintf(fp, "Hello Quarantine\n");
fclose(fp);

return 0;

}
```
</details>

І **видаліть** цей атрибут за допомогою:
```bash
xattr -d com.apple.quarantine portada.png
#You can also remove this attribute from every file with
find . -iname '*' -print0 | xargs -0 xattr -d com.apple.quarantine
```
І знайдіть всі заражені файли за допомогою:

{% code overflow="wrap" %}
```bash
find / -exec ls -ld {} \; 2>/dev/null | grep -E "[x\-]@ " | awk '{printf $9; printf "\n"}' | xargs -I {} xattr -lv {} | grep "com.apple.quarantine"
```
{% endcode %}

Інформація про карантин також зберігається в центральній базі даних, керованій LaunchServices в **`~/Library/Preferences/com.apple.LaunchServices.QuarantineEventsV2`**.

#### **Quarantine.kext**

Розширення ядра доступне лише через **кеш ядра на системі**; однак ви _можете_ завантажити **Набір засобів для відлагодження ядра з https://developer.apple.com/**, який містить символьну версію розширення.

### XProtect

XProtect - це вбудований **антивірусний** захист в macOS. XProtect **перевіряє будь-яку програму при її першому запуску або зміні на відомі віруси та небезпечні типи файлів у своїй базі даних**. Коли ви завантажуєте файл через певні програми, такі як Safari, Mail або Messages, XProtect автоматично сканує файл. Якщо він відповідає будь-якому відомому вірусу у своїй базі даних, XProtect **заборонить запуск файлу** та попередить вас про загрозу.

База даних XProtect **регулярно оновлюється** Apple новими визначеннями вірусів, і ці оновлення автоматично завантажуються та встановлюються на вашому Mac. Це забезпечує постійне оновлення XProtect з останніми відомими загрозами.

Однак варто зауважити, що **XProtect не є повноцінним антивірусним рішенням**. Він лише перевіряє певний список відомих загроз та не виконує сканування при доступі, як більшість антивірусного програмного забезпечення.

Ви можете отримати інформацію про останнє оновлення XProtect, запустивши:

{% code overflow="wrap" %}
```bash
system_profiler SPInstallHistoryDataType 2>/dev/null | grep -A 4 "XProtectPlistConfigData" | tail -n 5
```
{% endcode %}

XProtect розташований у захищеному місці SIP за шляхом **/Library/Apple/System/Library/CoreServices/XProtect.bundle** і всередині пакету ви можете знайти інформацію, яку використовує XProtect:

* **`XProtect.bundle/Contents/Resources/LegacyEntitlementAllowlist.plist`**: Дозволяє коду з цими cdhashes використовувати застарілі entitlements.
* **`XProtect.bundle/Contents/Resources/XProtect.meta.plist`**: Список плагінів та розширень, які заборонено завантажувати за допомогою BundleID та TeamID або вказуючи мінімальну версію.
* **`XProtect.bundle/Contents/Resources/XProtect.yara`**: Правила Yara для виявлення шкідливих програм.
* **`XProtect.bundle/Contents/Resources/gk.db`**: База даних SQLite3 з хешами заблокованих програм та TeamID.

Зверніть увагу, що є ще одне додаток у **`/Library/Apple/System/Library/CoreServices/XProtect.app`**, пов'язане з XProtect, яке не має відношення до процесу Gatekeeper.

### Не Gatekeeper

{% hint style="danger" %}
Зверніть увагу, що Gatekeeper **не виконується кожного разу**, коли ви запускаєте додаток, лише _**AppleMobileFileIntegrity**_ (AMFI) буде тільки **перевіряти підписи виконуваного коду**, коли ви запускаєте додаток, який вже був виконаний та перевірений Gatekeeper.
{% endhint %}

Отже, раніше було можливо виконати додаток для кешування його з Gatekeeper, потім **змінити не виконувальні файли додатка** (наприклад, файли Electron asar або NIB), і якщо інших захистів не було, додаток виконувався з **шкідливими** додатками.

Однак зараз це неможливо, оскільки macOS **запобігає зміні файлів** всередині пакунків додатків. Таким чином, якщо ви спробуєте атаку [Dirty NIB](../macos-proces-abuse/macos-dirty-nib.md), ви побачите, що це вже не можливо зловживати, оскільки після виконання додатка для кешування з Gatekeeper ви не зможете змінити пакет. І якщо ви, наприклад, зміните назву каталогу Contents на NotCon (як вказано в експлойті), а потім виконаєте основний бінарний файл додатка для кешування з Gatekeeper, він викличе помилку і не буде виконаний.

## Ухилення від Gatekeeper

Будь-який спосіб ухилення від Gatekeeper (змога змусити користувача завантажити щось та виконати це, коли Gatekeeper повинен заборонити це) вважається вразливістю в macOS. Ось деякі CVE, призначені для технік, які дозволяли ухиляти від Gatekeeper у минулому:

### [CVE-2021-1810](https://labs.withsecure.com/publications/the-discovery-of-cve-2021-1810)

Було помічено, що якщо для розпакування використовується **Archive Utility**, файли з **шляхами, що перевищують 886 символів**, не отримують розширеного атрибуту com.apple.quarantine. Ця ситуація ненавмисно дозволяє цим файлам **обійти безпеку Gatekeeper**.

Перевірте [**оригінальний звіт**](https://labs.withsecure.com/publications/the-discovery-of-cve-2021-1810) для отримання додаткової інформації.

### [CVE-2021-30990](https://ronmasas.com/posts/bypass-macos-gatekeeper)

Коли додаток створюється за допомогою **Automator**, інформація про те, що йому потрібно виконати, знаходиться всередині `application.app/Contents/document.wflow`, а не виконуваному файлі. Виконуваний файл - це просто загальний бінарний файл Automator під назвою **Automator Application Stub**.

Отже, ви можете зробити `application.app/Contents/MacOS/Automator\ Application\ Stub` **вказувати символічним посиланням на інший Automator Application Stub всередині системи**, і він виконає те, що знаходиться всередині `document.wflow` (ваш скрипт) **без спрацювання Gatekeeper**, оскільки фактичний виконуваний файл не має атрибуту карантину.&#x20;

Приклад очікуваного розташування: `/System/Library/CoreServices/Automator\ Application\ Stub.app/Contents/MacOS/Automator\ Application\ Stub`

Перевірте [**оригінальний звіт**](https://ronmasas.com/posts/bypass-macos-gatekeeper) для отримання додаткової інформації.

### [CVE-2022-22616](https://www.jamf.com/blog/jamf-threat-labs-safari-vuln-gatekeeper-bypass/)

У цьому ухиленні було створено zip-файл з додатком, який почав стискати з `application.app/Contents` замість `application.app`. Тому **атрибут карантину** був застосований до всіх **файлів з `application.app/Contents`**, але **не до `application.app`**, який перевіряв Gatekeeper, тому Gatekeeper був ухилено, оскільки коли `application.app` був запущений, **він не мав атрибуту карантину.**
```bash
zip -r test.app/Contents test.zip
```
Перевірте [**оригінальний звіт**](https://www.jamf.com/blog/jamf-threat-labs-safari-vuln-gatekeeper-bypass/) для отримання додаткової інформації.

### [CVE-2022-32910](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2022-32910)

Навіть якщо компоненти відрізняються, експлуатація цієї вразливості дуже схожа на попередню. У цьому випадку ми створимо Apple Archive з **`application.app/Contents`**, щоб **`application.app` не отримував атрибут карантину** при розпакуванні за допомогою **Archive Utility**.
```bash
aa archive -d test.app/Contents -o test.app.aar
```
Перевірте [**оригінальний звіт**](https://www.jamf.com/blog/jamf-threat-labs-macos-archive-utility-vulnerability/) для отримання додаткової інформації.

### [CVE-2022-42821](https://www.microsoft.com/en-us/security/blog/2022/12/19/gatekeepers-achilles-heel-unearthing-a-macos-vulnerability/)

ACL **`writeextattr`** може бути використаний для запобігання комусь писати атрибут у файлі:
```bash
touch /tmp/no-attr
chmod +a "everyone deny writeextattr" /tmp/no-attr
xattr -w attrname vale /tmp/no-attr
xattr: [Errno 13] Permission denied: '/tmp/no-attr'
```
Більше того, формат файлу **AppleDouble** копіює файл разом з його ACEs.

У [**вихідному коді**](https://opensource.apple.com/source/Libc/Libc-391/darwin/copyfile.c.auto.html) можна побачити, що текстове представлення ACL, збережене всередині xattr під назвою **`com.apple.acl.text`**, буде встановлено як ACL у розпакованому файлі. Таким чином, якщо ви стиснули додаток у zip-файл з форматом файлу **AppleDouble** з ACL, яке перешкоджає запису інших xattrs до нього... xattr карантину не було встановлено у додаток:

{% code overflow="wrap" %}
```bash
chmod +a "everyone deny write,writeattr,writeextattr" /tmp/test
ditto -c -k test test.zip
python3 -m http.server
# Download the zip from the browser and decompress it, the file should be without a quarantine xattr
```
{% endcode %}

Перевірте [**оригінальний звіт**](https://www.microsoft.com/en-us/security/blog/2022/12/19/gatekeepers-achilles-heel-unearthing-a-macos-vulnerability/) для отримання додаткової інформації.

Зверніть увагу, що це також може бути використано з AppleArchives:
```bash
mkdir app
touch app/test
chmod +a "everyone deny write,writeattr,writeextattr" app/test
aa archive -d app -o test.aar
```
### [CVE-2023-27943](https://blog.f-secure.com/discovery-of-gatekeeper-bypass-cve-2023-27943/)

Було виявлено, що **Google Chrome не встановлював атрибут карантину** для завантажених файлів через деякі внутрішні проблеми macOS.

### [CVE-2023-27951](https://redcanary.com/blog/gatekeeper-bypass-vulnerabilities/)

Формати файлів AppleDouble зберігають атрибути файлу в окремому файлі, який починається з `._`, це допомагає копіювати атрибути файлу **між машинами macOS**. Однак було помічено, що після розпакування файлу AppleDouble, файл, який починається з `._`, **не отримував атрибут карантину**.

{% code overflow="wrap" %}
```bash
mkdir test
echo a > test/a
echo b > test/b
echo ._a > test/._a
aa archive -d test/ -o test.aar

# If you downloaded the resulting test.aar and decompress it, the file test/._a won't have a quarantitne attribute
```
{% endcode %}

Можливість створення файлу без встановленого атрибуту карантину дозволяло **обійти Gatekeeper.** Секрет полягав у **створенні додатку файлу DMG** за допомогою конвенції імен AppleDouble (початок з `._`) та створенні **видимого файлу як символьного посилання на цей прихований** файл без атрибуту карантину.\
Коли **виконується файл dmg**, оскільки він не має атрибуту карантину, він **обходить Gatekeeper**.
```bash
# Create an app bundle with the backdoor an call it app.app

echo "[+] creating disk image with app"
hdiutil create -srcfolder app.app app.dmg

echo "[+] creating directory and files"
mkdir
mkdir -p s/app
cp app.dmg s/app/._app.dmg
ln -s ._app.dmg s/app/app.dmg

echo "[+] compressing files"
aa archive -d s/ -o app.aar
```
### Запобігання карантину xattr

У пакеті ".app", якщо до нього не додано атрибут карантину xattr, при його виконанні **Gatekeeper не буде активований**.
