# Лінукс Форензіка

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Використовуйте [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) для легкої побудови та **автоматизації робочих процесів**, які працюють на найбільш **продвинутих** інструментах спільноти.\
Отримайте доступ сьогодні:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити свою **компанію в рекламі на HackTricks** або **завантажити HackTricks у PDF** Перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи телеграм**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв GitHub.

</details>

## Початковий Збір Інформації

### Основна Інформація

По-перше, рекомендується мати **USB** з **відомими хорошими бінарниками та бібліотеками на ньому** (ви можете просто взяти Ubuntu та скопіювати папки _/bin_, _/sbin_, _/lib,_ та _/lib64_), потім підключити USB, та змінити змінні середовища для використання цих бінарників:
```bash
export PATH=/mnt/usb/bin:/mnt/usb/sbin
export LD_LIBRARY_PATH=/mnt/usb/lib:/mnt/usb/lib64
```
Після того, як ви налаштували систему для використання надійних та відомих бінарних файлів, ви можете почати **витягувати деяку базову інформацію**:
```bash
date #Date and time (Clock may be skewed, Might be at a different timezone)
uname -a #OS info
ifconfig -a || ip a #Network interfaces (promiscuous mode?)
ps -ef #Running processes
netstat -anp #Proccess and ports
lsof -V #Open files
netstat -rn; route #Routing table
df; mount #Free space and mounted devices
free #Meam and swap space
w #Who is connected
last -Faiwx #Logins
lsmod #What is loaded
cat /etc/passwd #Unexpected data?
cat /etc/shadow #Unexpected data?
find /directory -type f -mtime -1 -print #Find modified files during the last minute in the directory
```
#### Підозріла інформація

Під час отримання базової інформації варто перевірити на дивні речі, такі як:

* **Процеси root** зазвичай працюють з низькими PID, тому якщо ви знаходите процес root з великим PID, ви можете підозрювати
* Перевірте **зареєстровані входи** користувачів без оболонки в `/etc/passwd`
* Перевірте **хеші паролів** в `/etc/shadow` для користувачів без оболонки

### Дамп пам'яті

Для отримання пам'яті робочої системи рекомендується використовувати [**LiME**](https://github.com/504ensicsLabs/LiME).\
Для **компіляції** його вам потрібно використовувати **таке ж ядро**, яке використовує машина-жертва.

{% hint style="info" %}
Пам'ятайте, що ви **не можете встановлювати LiME або будь-що інше** на машину-жертву, оскільки це призведе до кількох змін в ній
{% endhint %}

Тому, якщо у вас є ідентична версія Ubuntu, ви можете використовувати `apt-get install lime-forensics-dkms`\
У інших випадках вам потрібно завантажити [**LiME**](https://github.com/504ensicsLabs/LiME) з github та скомпілювати його з правильними заголовками ядра. Для **отримання точних заголовків ядра** машини-жертви ви можете просто **скопіювати каталог** `/lib/modules/<версія ядра>` на свою машину, а потім **скомпілювати** LiME, використовуючи їх:
```bash
make -C /lib/modules/<kernel version>/build M=$PWD
sudo insmod lime.ko "path=/home/sansforensics/Desktop/mem_dump.bin format=lime"
```
LiME підтримує 3 **формати**:

* Raw (кожний сегмент об'єднаний разом)
* Padded (такий самий, як raw, але з нулями в правих бітах)
* Lime (рекомендований формат з метаданими)

LiME також може бути використаний для **відправлення дампу через мережу** замість зберігання його в системі, використовуючи щось на зразок: `path=tcp:4444`

### Диск Imaging

#### Вимкнення

По-перше, вам потрібно **вимкнути систему**. Це не завжди можливо, оскільки деякі системи є серверами виробничого призначення, які компанія не може собі дозволити вимкнути.\
Є **2 способи** вимкнення системи, **звичайне вимкнення** та **вимкнення "від'єднати штепсель"**. Перше дозволить **процесам завершити роботу як зазвичай** та **синхронізувати файлову систему**, але також дозволить можливому **шкідливому програмному забезпеченню** **знищити докази**. Підхід "від'єднати штепсель" може призвести до **втрати деякої інформації** (багато інформації не буде втрачено, оскільки ми вже взяли зображення пам'яті) та **шкідливе програмне забезпечення не матиме можливості** щодо цього нічого зробити. Тому, якщо ви **підозрюєте**, що може бути **шкідливе програмне забезпечення**, просто виконайте **команду `sync`** на системі та від'єднайте штепсель.

#### Взяття зображення диска

Важливо зауважити, що **перед підключенням вашого комп'ютера до чого-небудь, що стосується справи**, вам потрібно бути впевненим, що він буде **підключений як тільки для читання**, щоб уникнути зміни будь-якої інформації.
```bash
#Create a raw copy of the disk
dd if=<subject device> of=<image file> bs=512

#Raw copy with hashes along the way (more secure as it checks hashes while it's copying the data)
dcfldd if=<subject device> of=<image file> bs=512 hash=<algorithm> hashwindow=<chunk size> hashlog=<hash file>
dcfldd if=/dev/sdc of=/media/usb/pc.image hash=sha256 hashwindow=1M hashlog=/media/usb/pc.hashes
```
### Попередній аналіз образу диска

Створення образу диска без додаткових даних.
```bash
#Find out if it's a disk image using "file" command
file disk.img
disk.img: Linux rev 1.0 ext4 filesystem data, UUID=59e7a736-9c90-4fab-ae35-1d6a28e5de27 (extents) (64bit) (large files) (huge files)

#Check which type of disk image it's
img_stat -t evidence.img
raw
#You can list supported types with
img_stat -i list
Supported image format types:
raw (Single or split raw file (dd))
aff (Advanced Forensic Format)
afd (AFF Multiple File)
afm (AFF with external metadata)
afflib (All AFFLIB image formats (including beta ones))
ewf (Expert Witness Format (EnCase))

#Data of the image
fsstat -i raw -f ext4 disk.img
FILE SYSTEM INFORMATION
--------------------------------------------
File System Type: Ext4
Volume Name:
Volume ID: 162850f203fd75afab4f1e4736a7e776

Last Written at: 2020-02-06 06:22:48 (UTC)
Last Checked at: 2020-02-06 06:15:09 (UTC)

Last Mounted at: 2020-02-06 06:15:18 (UTC)
Unmounted properly
Last mounted on: /mnt/disk0

Source OS: Linux
[...]

#ls inside the image
fls -i raw -f ext4 disk.img
d/d 11: lost+found
d/d 12: Documents
d/d 8193:       folder1
d/d 8194:       folder2
V/V 65537:      $OrphanFiles

#ls inside folder
fls -i raw -f ext4 disk.img 12
r/r 16: secret.txt

#cat file inside image
icat -i raw -f ext4 disk.img 16
ThisisTheMasterSecret
```
<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Використовуйте [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) для легкої побудови та **автоматизації робочих процесів**, що працюють на найбільш **продвинутих** інструментах спільноти у світі.\
Отримайте доступ сьогодні:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## Пошук відомого шкідливого програмного забезпечення

### Змінені системні файли

Linux пропонує інструменти для забезпечення цілісності компонентів системи, що є важливим для виявлення потенційно проблемних файлів.

* **Системи на основі RedHat**: Використовуйте `rpm -Va` для комплексної перевірки.
* **Системи на основі Debian**: `dpkg --verify` для початкової перевірки, а потім `debsums | grep -v "OK$"` (після встановлення `debsums` за допомогою `apt-get install debsums`) для ідентифікації будь-яких проблем.

### Виявлення шкідливого програмного забезпечення/кітових програм

Прочитайте наступну сторінку, щоб дізнатися про інструменти, які можуть бути корисними для пошуку шкідливого програмного забезпечення:

{% content-ref url="malware-analysis.md" %}
[malware-analysis.md](malware-analysis.md)
{% endcontent-ref %}

## Пошук встановлених програм

Для ефективного пошуку встановлених програм як на системах Debian, так і на RedHat, розгляньте можливість використання системних журналів та баз даних разом із ручними перевірками в загальних каталогах.

* Для Debian перевірте _**`/var/lib/dpkg/status`**_ та _**`/var/log/dpkg.log`**_ для отримання деталей про встановлені пакети, використовуючи `grep` для фільтрації конкретної інформації.
* Користувачі RedHat можуть запитувати базу даних RPM за допомогою `rpm -qa --root=/mntpath/var/lib/rpm`, щоб перелічити встановлені пакети.

Щоб виявити програмне забезпечення, встановлене вручну або поза цими менеджерами пакунків, дослідіть каталоги, такі як _**`/usr/local`**_, _**`/opt`**_, _**`/usr/sbin`**_, _**`/usr/bin`**_, _**`/bin`**_ та _**`/sbin`**_. Поєднайте переліки каталогів з системними командами, щоб ідентифікувати виконувані файли, які не пов'язані з відомими пакетами, покращуючи пошук всіх встановлених програм.
```bash
# Debian package and log details
cat /var/lib/dpkg/status | grep -E "Package:|Status:"
cat /var/log/dpkg.log | grep installed
# RedHat RPM database query
rpm -qa --root=/mntpath/var/lib/rpm
# Listing directories for manual installations
ls /usr/sbin /usr/bin /bin /sbin
# Identifying non-package executables (Debian)
find /sbin/ -exec dpkg -S {} \; | grep "no path found"
# Identifying non-package executables (RedHat)
find /sbin/ –exec rpm -qf {} \; | grep "is not"
# Find exacuable files
find / -type f -executable | grep <something>
```
<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Використовуйте [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks), щоб легко створювати та **автоматизувати робочі процеси**, які працюють на найбільш **продвинутих** інструментах спільноти у світі.\
Отримайте доступ сьогодні:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## Відновлення видалених виконуваних файлів

Уявіть процес, який був виконаний з /tmp/exec та видалений. Є можливість його видобути
```bash
cd /proc/3746/ #PID with the exec file deleted
head -1 maps #Get address of the file. It was 08048000-08049000
dd if=mem bs=1 skip=08048000 count=1000 of=/tmp/exec2 #Recorver it
```
## Перевірка місць автозапуску

### Заплановані завдання
```bash
cat /var/spool/cron/crontabs/*  \
/var/spool/cron/atjobs \
/var/spool/anacron \
/etc/cron* \
/etc/at* \
/etc/anacrontab \
/etc/incron.d/* \
/var/spool/incron/* \

#MacOS
ls -l /usr/lib/cron/tabs/ /Library/LaunchAgents/ /Library/LaunchDaemons/ ~/Library/LaunchAgents/
```
### Сервіси

Шляхи, де може бути встановлений шкідливий ПЗ як сервіс:

* **/etc/inittab**: Викликає ініціалізаційні скрипти, наприклад rc.sysinit, спрямовуючи далі до скриптів запуску.
* **/etc/rc.d/** та **/etc/rc.boot/**: Містять скрипти для запуску сервісів, останній з яких знаходиться в старіших версіях Linux.
* **/etc/init.d/**: Використовується в певних версіях Linux, наприклад Debian, для зберігання скриптів запуску.
* Сервіси також можуть бути активовані через **/etc/inetd.conf** або **/etc/xinetd/**, залежно від варіанту Linux.
* **/etc/systemd/system**: Каталог для системних та скриптів керування сервісами.
* **/etc/systemd/system/multi-user.target.wants/**: Містить посилання на сервіси, які повинні бути запущені в режимі багатокористувацького рівня.
* **/usr/local/etc/rc.d/**: Для власних або сторонніх сервісів.
* **\~/.config/autostart/**: Для автоматичного запуску програм, специфічних для користувача, що може бути прихованим місцем для шкідливого ПЗ, спрямованого на користувача.
* **/lib/systemd/system/**: Системні файли одиниць за замовчуванням, надані встановленими пакетами.

### Модулі ядра

Модулі ядра Linux, часто використовувані шкідливим ПЗ як компоненти rootkit, завантажуються при завантаженні системи. Критичні для цих модулів каталоги та файли включають:

* **/lib/modules/$(uname -r)**: Містить модулі для версії ядра, що працює.
* **/etc/modprobe.d**: Містить файли конфігурації для керування завантаженням модулів.
* **/etc/modprobe** та **/etc/modprobe.conf**: Файли для глобальних налаштувань модулів.

### Інші місця автозапуску

Linux використовує різні файли для автоматичного виконання програм при вході користувача, що потенційно можуть приховувати шкідливе ПЗ:

* **/etc/profile.d/**\*, **/etc/profile**, та **/etc/bash.bashrc**: Виконуються при будь-якому вході користувача.
* **\~/.bashrc**, **\~/.bash\_profile**, **\~/.profile**, та **\~/.config/autostart**: Файли, специфічні для користувача, які запускаються при їх вході.
* **/etc/rc.local**: Запускається після того, як всі системні служби стартують, позначаючи завершення переходу до середовища багатокористувацького рівня.

## Аналіз журналів

Системи Linux відстежують дії користувачів та події системи через різні файли журналів. Ці журнали є ключовими для ідентифікації несанкціонованого доступу, інфікування шкідливим ПЗ та інших інцидентів безпеки. Основні файли журналів включають:

* **/var/log/syslog** (Debian) або **/var/log/messages** (RedHat): Записують повідомлення та дії на рівні системи.
* **/var/log/auth.log** (Debian) або **/var/log/secure** (RedHat): Фіксують спроби аутентифікації, успішні та невдачні входи.
* Використовуйте `grep -iE "session opened for|accepted password|new session|not in sudoers" /var/log/auth.log` для фільтрації відповідних подій аутентифікації.
* **/var/log/boot.log**: Містить повідомлення про запуск системи.
* **/var/log/maillog** або **/var/log/mail.log**: Журнали дій поштового сервера, корисні для відстеження поштових сервісів.
* **/var/log/kern.log**: Зберігає повідомлення ядра, включаючи помилки та попередження.
* **/var/log/dmesg**: Містить повідомлення драйвера пристрою.
* **/var/log/faillog**: Фіксує невдачі входу, допомагаючи в розслідуванні порушень безпеки.
* **/var/log/cron**: Журнали виконання завдань cron.
* **/var/log/daemon.log**: Відстежує діяльність фонових служб.
* **/var/log/btmp**: Документує невдачі входу.
* **/var/log/httpd/**: Містить журнали помилок та доступу Apache HTTPD.
* **/var/log/mysqld.log** або **/var/log/mysql.log**: Журнали дій бази даних MySQL.
* **/var/log/xferlog**: Фіксує передачі файлів FTP.
* **/var/log/**: Завжди перевіряйте наявність неочікуваних журналів тут.

{% hint style="info" %}
Системні журнали та підсистеми аудиту Linux можуть бути вимкнені або видалені під час вторгнення або інциденту з шкідливим ПЗ. Оскільки журнали в системах Linux, як правило, містять деяку з найкориснішою інформацією про зловмисну діяльність, зловмисники регулярно їх видаляють. Тому при аналізі наявних файлів журналів важливо шукати прогалини або неузгоджені записи, які можуть свідчити про видалення або втручання.
{% endhint %}

**Linux зберігає історію команд для кожного користувача**, зберігається в:

* \~/.bash\_history
* \~/.zsh\_history
* \~/.zsh\_sessions/\*
* \~/.python\_history
* \~/.\*\_history

Крім того, команда `last -Faiwx` надає список входів користувачів. Перевірте його на невідомі або неочікувані входи.

Перевірте файли, які можуть надати додаткові привілеї:

* Перегляньте `/etc/sudoers` на непередбачені привілеї користувача, які можуть бути надані.
* Перегляньте `/etc/sudoers.d/` на непередбачені привілеї користувача, які можуть бути надані.
* Дослідіть `/etc/groups`, щоб виявити незвичайні членства в групах або дозволи.
* Дослідіть `/etc/passwd`, щоб виявити незвичайні членства в групах або дозволи.

Деякі програми також генерують власні журнали:

* **SSH**: Перевірте _\~/.ssh/authorized\_keys_ та _\~/.ssh/known\_hosts_ на несанкціоновані віддалені підключення.
* **Gnome Desktop**: Перегляньте _\~/.recently-used.xbel_ для нещодавно відкритих файлів через додатки Gnome.
* **Firefox/Chrome**: Перевірте історію браузера та завантаження в _\~/.mozilla/firefox_ або _\~/.config/google-chrome_ на підозрілі дії.
* **VIM**: Перегляньте _\~/.viminfo_ для деталей використання, таких як шляхи до файлів та історія пошуку.
* **Open Office**: Перевірте нещодавній доступ до документів, що може вказувати на компрометовані файли.
* **FTP/SFTP**: Перегляньте журнали в _\~/.ftp\_history_ або _\~/.sftp\_history_ для передач файлів, які можуть бути несанкціонованими.
* **MySQL**: Дослідіть _\~/.mysql\_history_ для виконаних запитів MySQL, що може розкрити несанкціоновану діяльність бази даних.
* **Less**: Проаналізуйте _\~/.lesshst_ для історії використання, включаючи переглянуті файли та виконані команди.
* **Git**: Перегляньте _\~/.gitconfig_ та проект _.git/logs_ для змін у репозиторіях.

### Журнали USB

[**usbrip**](https://github.com/snovvcrash/usbrip) - це невеликий програмний засіб, написаний на чистому Python 3, який аналізує файли журналів Linux (`/var/log/syslog*` або `/var/log/messages*` в залежності від дистрибутиву) для побудови таблиць історії подій USB.

Цікаво **знати всі USB, які були використані**, і це буде корисно, якщо у вас є авторизований список USB для пошуку "порушень" (використання USB, які не входять до цього списку).

### Встановлення
```bash
pip3 install usbrip
usbrip ids download #Download USB ID database
```
### Приклади
```bash
usbrip events history #Get USB history of your curent linux machine
usbrip events history --pid 0002 --vid 0e0f --user kali #Search by pid OR vid OR user
#Search for vid and/or pid
usbrip ids download #Downlaod database
usbrip ids search --pid 0002 --vid 0e0f #Search for pid AND vid
```
Більше прикладів та інформації у GitHub: [https://github.com/snovvcrash/usbrip](https://github.com/snovvcrash/usbrip)

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Використовуйте [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) для легкої побудови та **автоматизації робочих процесів**, які працюють за допомогою найбільш **продвинутих** інструментів у спільноті.\
Отримайте доступ сьогодні:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## Перегляд облікових записів користувачів та активностей входу в систему

Перевірте файли _**/etc/passwd**_, _**/etc/shadow**_ та **журнали безпеки** на наявність незвичайних імен або облікових записів, створених або використаних поруч з відомими несанкціонованими подіями. Також перевірте можливі атаки на sudo методом перебору паролів.\
Крім того, перевірте файли, такі як _**/etc/sudoers**_ та _**/etc/groups**_ на непередбачені привілеї, надані користувачам.\
Нарешті, шукайте облікові записи без **паролів** або з **легко вгадуваними** паролями.

## Аналіз файлової системи

### Аналіз структур файлової системи при розслідуванні випадків шкідливого програмного забезпечення

При розслідуванні випадків шкідливого програмного забезпечення структура файлової системи є важливим джерелом інформації, що розкриває як послідовність подій, так і вміст шкідливого програмного забезпечення. Однак автори шкідливого програмного забезпечення розробляють техніки для ускладнення цього аналізу, такі як зміна часів штампів файлів або уникання файлової системи для зберігання даних.

Для протидії цим анти-форензичним методам важливо:

* **Провести ретельний аналіз часової шкали** за допомогою інструментів, таких як **Autopsy** для візуалізації часових шкал подій або `mactime` від **Sleuth Kit** для детальних даних часової шкали.
* **Дослідити неочікувані скрипти** у $PATH системи, які можуть містити скрипти оболонки або PHP, використовані зловмисниками.
* **Перевірте `/dev` на атипові файли**, оскільки традиційно тут містяться спеціальні файли, але можуть містити файли, пов'язані з шкідливим програмним забезпеченням.
* **Шукайте приховані файли або каталоги** з назвами, такими як ".. " (крапка крапка пробіл) або "..^G" (крапка крапка керування-G), які можуть приховувати зловмисний вміст.
* **Визначте файли setuid root** за допомогою команди: `find / -user root -perm -04000 -print` Це знаходить файли з підвищеними дозволами, які можуть бути використані зловмисниками.
* **Перевірте часи видалення** в таблицях inode, щоб виявити масове видалення файлів, що може вказувати на наявність rootkit або троянців.
* **Огляньте послідовні inode** для поруч знаходячихся зловмисних файлів після ідентифікації одного, оскільки вони можуть бути розміщені разом.
* **Перевірте загальні бінарні каталоги** (_/bin_, _/sbin_) на недавно змінені файли, оскільки їх може бути змінено шкідливим програмним забезпеченням.
````bash
# List recent files in a directory:
ls -laR --sort=time /bin```

# Sort files in a directory by inode:
ls -lai /bin | sort -n```
````
{% hint style="info" %}
Зверніть увагу, що **зловмисник** може **змінити** **час**, щоб файли виглядали **легітимними**, але він **не може** змінити **інод**. Якщо ви помітили, що **файл** вказує на те, що він був створений і змінений у **один і той же час**, що й решта файлів у тій самій папці, але **інод** **неочікувано більший**, то **часи змінені** для цього файлу.
{% endhint %}

## Порівняння файлів різних версій файлової системи

### Зведення порівняння версій файлової системи

Для порівняння версій файлової системи та визначення змін використовуємо спрощені команди `git diff`:

* **Для пошуку нових файлів** порівняйте дві теки:
```bash
git diff --no-index --diff-filter=A path/to/old_version/ path/to/new_version/
```
* **Для зміненого вмісту**, перерахуйте зміни, ігноруючи конкретні рядки:
```bash
git diff --no-index --diff-filter=M path/to/old_version/ path/to/new_version/ | grep -E "^\+" | grep -v "Installed-Time"
```
* **Для виявлення видалених файлів**:
```bash
git diff --no-index --diff-filter=D path/to/old_version/ path/to/new_version/
```
* **Опції фільтрації** (`--diff-filter`) допомагають звузити вибір до конкретних змін, таких як додані (`A`), видалені (`D`) або змінені (`M`) файли.
* `A`: Додані файли
* `C`: Скопійовані файли
* `D`: Видалені файли
* `M`: Змінені файли
* `R`: Перейменовані файли
* `T`: Зміни типу (наприклад, файл на символьне посилання)
* `U`: Несполучені файли
* `X`: Невідомі файли
* `B`: Пошкоджені файли

## Посилання

* [https://cdn.ttgtmedia.com/rms/security/Malware%20Forensics%20Field%20Guide%20for%20Linux%20Systems\_Ch3.pdf](https://cdn.ttgtmedia.com/rms/security/Malware%20Forensics%20Field%20Guide%20for%20Linux%20Systems\_Ch3.pdf)
* [https://www.plesk.com/blog/featured/linux-logs-explained/](https://www.plesk.com/blog/featured/linux-logs-explained/)
* [https://git-scm.com/docs/git-diff#Documentation/git-diff.txt---diff-filterACDMRTUXB82308203](https://git-scm.com/docs/git-diff#Documentation/git-diff.txt---diff-filterACDMRTUXB82308203)
* **Книга: Malware Forensics Field Guide for Linux Systems: Digital Forensics Field Guides**
