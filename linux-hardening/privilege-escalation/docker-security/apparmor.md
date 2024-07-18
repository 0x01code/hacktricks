# AppArmor

{% hint style="success" %}
Вивчайте та практикуйте взлом AWS: <img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**Школа взлому AWS для Червоної Команди (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте взлом GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**Школа взлому GCP для Червоної Команди (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримайте HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Поширюйте хакерські трюки, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>
{% endhint %}

### [WhiteIntel](https://whiteintel.io)

<figure><img src="../../../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) - це пошуковик, який працює на **темному вебі** та пропонує **безкоштовні** функціональності для перевірки, чи були **компанія або її клієнти скомпрометовані** **шкідливими програмами-крадіями**.

Основною метою WhiteIntel є боротьба з захопленням облікових записів та атаками вимагання викупу, що виникають внаслідок шкідливих програм, які крадуть інформацію.

Ви можете перевірити їх веб-сайт та спробувати їхній двигун **безкоштовно** за посиланням:

{% embed url="https://whiteintel.io" %}

***

## Основна інформація

AppArmor - це **покращення ядра, призначене для обмеження ресурсів, доступних програмам через профілі для кожної програми**, ефективно реалізуючи обов'язковий контроль доступу (MAC), пов'язуючи атрибути контролю доступу безпосередньо з програмами, а не з користувачами. Ця система працює шляхом **завантаження профілів в ядро**, зазвичай під час завантаження, і ці профілі визначають, до яких ресурсів може отримати доступ програма, таких як мережеві підключення, доступ до сокетів та дозволи на файли.

Існують два режими роботи для профілів AppArmor:

* **Режим виконання**: Цей режим активно застосовує політики, визначені в межах профілю, блокуючи дії, які порушують ці політики, та реєструючи будь-які спроби їх порушення через системи, такі як syslog або auditd.
* **Режим скарг**: На відміну від режиму виконання, режим скарг не блокує дії, які суперечать політикам профілю. Замість цього він реєструє ці спроби як порушення політики без застосування обмежень.

### Компоненти AppArmor

* **Модуль ядра**: Відповідає за виконання політик.
* **Політики**: Визначають правила та обмеження для поведінки програм та доступу до ресурсів.
* **Парсер**: Завантажує політики в ядро для виконання або звітування.
* **Утиліти**: Це програми режиму користувача, які надають інтерфейс для взаємодії та управління AppArmor.

### Шляхи до профілів

Профілі AppArmor зазвичай зберігаються в _**/etc/apparmor.d/**_\
За допомогою `sudo aa-status` ви зможете переглянути перелік виконуваних файлів, які обмежені яким-небудь профілем. Якщо ви заміните символ "/" на крапку у шляху кожного вказаного виконуваного файлу, ви отримаєте назву профілю apparmor всередині зазначеної папки.

Наприклад, профіль **apparmor** для _/usr/bin/man_ буде розташований в _/etc/apparmor.d/usr.bin.man_.

### Команди
```bash
aa-status     #check the current status
aa-enforce    #set profile to enforce mode (from disable or complain)
aa-complain   #set profile to complain mode (from diable or enforcement)
apparmor_parser #to load/reload an altered policy
aa-genprof    #generate a new profile
aa-logprof    #used to change the policy when the binary/program is changed
aa-mergeprof  #used to merge the policies
```
## Створення профілю

* Для вказання затронутої виконувальної програми дозволяються **абсолютні шляхи та маски** (для глобального вибору файлів).
* Для вказання доступу, який виконувальний файл матиме до **файлів**, можна використовувати наступні **контролі доступу**:
* **r** (читання)
* **w** (запис)
* **m** (відображення пам'яті як виконуваний)
* **k** (блокування файлів)
* **l** (створення жорстких посилань)
* **ix** (виконання іншої програми з новою програмою, яка успадковує політику)
* **Px** (виконання під іншим профілем після очищення середовища)
* **Cx** (виконання під дочірнім профілем після очищення середовища)
* **Ux** (виконання без обмежень після очищення середовища)
* **Змінні** можуть бути визначені в профілях та можуть бути змінені ззовні профілю. Наприклад: @{PROC} та @{HOME} (додайте #include \<tunables/global> до файлу профілю)
* **Правила відмови підтримуються для перевизначення правил дозволу**.

### aa-genprof

Для початку створення профілю вам може допомогти **apparmor**. Можливо, **apparmor перевіряє дії, виконані виконувальною програмою, а потім дозволяє вам вирішити, які дії ви хочете дозволити або відмовити**.\
Вам просто потрібно виконати:
```bash
sudo aa-genprof /path/to/binary
```
Потім, у іншій консолі виконайте всі дії, які зазвичай виконує виконуваний файл:
```bash
/path/to/binary -a dosomething
```
Потім, у першій консолі натисніть "**s**", а потім у записаних діях вкажіть, чи хочете ви ігнорувати, дозволяти чи щось інше. Коли закінчите, натисніть "**f**", і новий профіль буде створено в _/etc/apparmor.d/path.to.binary_

{% hint style="info" %}
Використовуючи стрілки, ви можете вибрати, що хочете дозволити/заборонити/щось інше
{% endhint %}

### aa-easyprof

Ви також можете створити шаблон профілю apparmor для бінарного файлу за допомогою:
```bash
sudo aa-easyprof /path/to/binary
# vim:syntax=apparmor
# AppArmor policy for binary
# ###AUTHOR###
# ###COPYRIGHT###
# ###COMMENT###

#include <tunables/global>

# No template variables specified

"/path/to/binary" {
#include <abstractions/base>

# No abstractions specified

# No policy groups specified

# No read paths specified

# No write paths specified
}
```
{% hint style="info" %}
Зауважте, що за замовчуванням у створеному профілі нічого не дозволено, тому все відхилено. Вам потрібно додати рядки, наприклад, `/etc/passwd r,` щоб дозволити бінарний читати `/etc/passwd`, наприклад.
{% endhint %}

Потім ви можете **заставити** новий профіль застосовуватися за допомогою
```bash
sudo apparmor_parser -a /etc/apparmor.d/path.to.binary
```
### Зміна профілю з журналів

Наступний інструмент буде читати журнали та запитувати користувача, чи він хоче дозволити деякі виявлені заборонені дії:
```bash
sudo aa-logprof
```
{% hint style="info" %}
За допомогою стрілок ви можете вибрати, що ви хочете дозволити/заборонити/чи ще щось
{% endhint %}

### Керування профілем
```bash
#Main profile management commands
apparmor_parser -a /etc/apparmor.d/profile.name #Load a new profile in enforce mode
apparmor_parser -C /etc/apparmor.d/profile.name #Load a new profile in complain mode
apparmor_parser -r /etc/apparmor.d/profile.name #Replace existing profile
apparmor_parser -R /etc/apparmor.d/profile.name #Remove profile
```
## Логи

Приклад **AUDIT** та **DENIED** логів з _/var/log/audit/audit.log_ виконуваного файлу **`service_bin`**:
```bash
type=AVC msg=audit(1610061880.392:286): apparmor="AUDIT" operation="getattr" profile="/bin/rcat" name="/dev/pts/1" pid=954 comm="service_bin" requested_mask="r" fsuid=1000 ouid=1000
type=AVC msg=audit(1610061880.392:287): apparmor="DENIED" operation="open" profile="/bin/rcat" name="/etc/hosts" pid=954 comm="service_bin" requested_mask="r" denied_mask="r" fsuid=1000 ouid=0
```
Ви також можете отримати цю інформацію, використовуючи:
```bash
sudo aa-notify -s 1 -v
Profile: /bin/service_bin
Operation: open
Name: /etc/passwd
Denied: r
Logfile: /var/log/audit/audit.log

Profile: /bin/service_bin
Operation: open
Name: /etc/hosts
Denied: r
Logfile: /var/log/audit/audit.log

AppArmor denials: 2 (since Wed Jan  6 23:51:08 2021)
For more information, please see: https://wiki.ubuntu.com/DebuggingApparmor
```
## Apparmor в Docker

Зверніть увагу, що профіль **docker-profile** docker за замовчуванням завантажується:
```bash
sudo aa-status
apparmor module is loaded.
50 profiles are loaded.
13 profiles are in enforce mode.
/sbin/dhclient
/usr/bin/lxc-start
/usr/lib/NetworkManager/nm-dhcp-client.action
/usr/lib/NetworkManager/nm-dhcp-helper
/usr/lib/chromium-browser/chromium-browser//browser_java
/usr/lib/chromium-browser/chromium-browser//browser_openjdk
/usr/lib/chromium-browser/chromium-browser//sanitized_helper
/usr/lib/connman/scripts/dhclient-script
docker-default
```
За замовчуванням профіль **Apparmor docker-default** генерується з [https://github.com/moby/moby/tree/master/profiles/apparmor](https://github.com/moby/moby/tree/master/profiles/apparmor)

**Загальна інформація про профіль docker-default**:

* **Доступ** до всієї **мережі**
* **Жодна можливість** не визначена (Однак деякі можливості будуть включені за допомогою базових правил, наприклад #include \<abstractions/base>)
* **Запис** в будь-який файл **/proc** **не дозволено**
* Інші **підкаталоги**/**файли** /**proc** та /**sys** мають **заборонений** доступ на читання/запис/блокування/посилання/виконання
* **Монтування** **не дозволено**
* **Ptrace** може бути запущений лише на процесі, який обмежений **тим самим профілем apparmor**

Після того, як ви **запустите контейнер docker**, ви повинні побачити наступний вивід:
```bash
1 processes are in enforce mode.
docker-default (825)
```
Зауважте, що **apparmor навіть заблокує привілеї capabilities**, надані контейнеру за замовчуванням. Наприклад, він може **заблокувати дозвіл на запис всередині /proc навіть якщо SYS\_ADMIN capability надано**, оскільки за замовчуванням профіль apparmor docker відмовляє в цьому доступі:
```bash
docker run -it --cap-add SYS_ADMIN --security-opt seccomp=unconfined ubuntu /bin/bash
echo "" > /proc/stat
sh: 1: cannot create /proc/stat: Permission denied
```
Вам потрібно **вимкнути apparmor**, щоб обійти його обмеження:
```bash
docker run -it --cap-add SYS_ADMIN --security-opt seccomp=unconfined --security-opt apparmor=unconfined ubuntu /bin/bash
```
Зауважте, що за замовчуванням **AppArmor** також **заборонить контейнеру монтувати** теки зсередини навіть з можливістю SYS\_ADMIN.

Зверніть увагу, що ви можете **додавати/видаляти** **можливості** до контейнера Docker (це все ще буде обмежено методами захисту, такими як **AppArmor** та **Seccomp**):

* `--cap-add=SYS_ADMIN` додати можливість `SYS_ADMIN`
* `--cap-add=ALL` додати всі можливості
* `--cap-drop=ALL --cap-add=SYS_PTRACE` відмінити всі можливості та додати лише `SYS_PTRACE`

{% hint style="info" %}
Зазвичай, коли ви **виявляєте**, що у вас є **привілейована можливість** доступна **всередині** контейнера **docker**, **але** деяка частина **експлойту не працює**, це може бути через те, що **apparmor docker буде це запобігати**.
{% endhint %}

### Приклад

(Приклад з [**тут**](https://sreeninet.wordpress.com/2016/03/06/docker-security-part-2docker-engine/))

Щоб проілюструвати функціональність AppArmor, я створив новий профіль Docker "mydocker" з наступним доданим рядком:
```
deny /etc/* w,   # deny write for all files directly in /etc (not in a subdir)
```
Щоб активувати профіль, нам потрібно виконати наступне:
```
sudo apparmor_parser -r -W mydocker
```
Для переліку профілів ми можемо виконати наступну команду. Нижче наведена команда, яка перелічує мій новий профіль AppArmor.
```
$ sudo apparmor_status  | grep mydocker
mydocker
```
Як показано нижче, ми отримуємо помилку при спробі змінити "/etc/", оскільки профіль AppArmor перешкоджає доступу на запис до "/etc".
```
$ docker run --rm -it --security-opt apparmor:mydocker -v ~/haproxy:/localhost busybox chmod 400 /etc/hostname
chmod: /etc/hostname: Permission denied
```
### Обхід захисту Docker AppArmor1

Ви можете знайти, який **профіль apparmor використовується контейнером**, використовуючи:
```bash
docker inspect 9d622d73a614 | grep lowpriv
"AppArmorProfile": "lowpriv",
"apparmor=lowpriv"
```
Потім ви можете запустити наступний рядок, щоб **знайти точний профіль, який використовується**:
```bash
find /etc/apparmor.d/ -name "*lowpriv*" -maxdepth 1 2>/dev/null
```
### Обхід захисту Docker AppArmor

**AppArmor базується на шляхах**, це означає, що навіть якщо він може **захищати** файли всередині каталогу, наприклад **`/proc`**, якщо ви можете **налаштувати спосіб запуску контейнера**, ви можете **підмонтувати** каталог proc хоста всередині **`/host/proc`**, і він **більше не буде захищений AppArmor**.

### Обхід AppArmor через Shebang

У [**цьому багу**](https://bugs.launchpad.net/apparmor/+bug/1911431) ви можете побачити приклад того, як **навіть якщо ви запобігаєте запуску perl з певними ресурсами**, якщо ви просто створите оболонку **вказуючи** у першому рядку **`#!/usr/bin/perl`**, і **виконаєте файл безпосередньо**, ви зможете виконати все, що завгодно. Наприклад:
```perl
echo '#!/usr/bin/perl
use POSIX qw(strftime);
use POSIX qw(setuid);
POSIX::setuid(0);
exec "/bin/sh"' > /tmp/test.pl
chmod +x /tmp/test.pl
/tmp/test.pl
```
### [WhiteIntel](https://whiteintel.io)

<figure><img src="../../../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) - це пошуковий двигун, який працює на **темному вебі** і пропонує **безкоштовні** функціональні можливості для перевірки, чи були **компанія або її клієнти скомпрометовані** **викрадачами шкідливих програм**.

Основна мета WhiteIntel - боротьба з захопленням облікових записів та атаками вірусів-вимагачів, що виникають внаслідок шкідливих програм, які крадуть інформацію.

Ви можете перевірити їх веб-сайт та спробувати їх двигун **безкоштовно** за посиланням:

{% embed url="https://whiteintel.io" %}

{% hint style="success" %}
Вивчайте та практикуйте взлом AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**Навчання HackTricks AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте взлом GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**Навчання HackTricks GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримайте HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Поширюйте хакерські трюки, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>
{% endhint %}
