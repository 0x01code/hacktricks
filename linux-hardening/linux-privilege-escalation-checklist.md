# Чек-лист - Підвищення привілеїв в Linux

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію в рекламі на HackTricks** або **завантажити HackTricks у PDF-форматі**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв GitHub.

</details>

<figure><img src="../../.gitbook/assets/image (1) (3) (1).png" alt=""><figcaption></figcaption></figure>

Приєднуйтесь до [**HackenProof Discord**](https://discord.com/invite/N3FrSbmwdy), щоб спілкуватися з досвідченими хакерами та мисливцями за багами!

**Інсайти щодо хакінгу**\
Взаємодійте з контентом, який досліджує захоплення та виклики хакінгу

**Новини про хакінг у реальному часі**\
Будьте в курсі швидкозмінного світу хакінгу завдяки новинам та інсайтам у реальному часі

**Останні оголошення**\
Будьте в курсі нових баг-баунті, які запускаються, та важливих оновлень платформи

**Приєднуйтесь до нас на** [**Discord**](https://discord.com/invite/N3FrSbmwdy) та почніть співпрацювати з найкращими хакерами вже сьогодні!

### **Найкращий інструмент для пошуку векторів підвищення привілеїв в Linux:** [**LinPEAS**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS)

### [Інформація про систему](privilege-escalation/#system-information)

* [ ] Отримати **інформацію про ОС**
* [ ] Перевірити [**ШЛЯХ**](privilege-escalation/#path), будь-яка **записувальна тека**?
* [ ] Перевірити [**змінні середовища**](privilege-escalation/#env-info), будь-які чутливі дані?
* [ ] Шукати [**експлойти ядра**](privilege-escalation/#kernel-exploits) **за допомогою скриптів** (DirtyCow?)
* [ ] **Перевірити**, чи є [**уразлива версія sudo**](privilege-escalation/#sudo-version)
* [ ] [**Dmesg** перевірка підпису не вдалася](privilege-escalation/#dmesg-signature-verification-failed)
* [ ] Додаткова системна енумерація ([дата, статистика системи, інформація про процесор, принтери](privilege-escalation/#more-system-enumeration))
* [ ] [Енумерація додаткових захистів](privilege-escalation/#enumerate-possible-defenses)

### [Диски](privilege-escalation/#drives)

* [ ] **Список підключених** дисків
* [ ] Чи є **непідключений диск**?
* [ ] Чи є **дані авторизації в fstab**?

### [**Встановлене програмне забезпечення**](privilege-escalation/#installed-software)

* [ ] **Перевірити наявність** [**корисного програмного забезпечення**](privilege-escalation/#useful-software) **встановленого**
* [ ] **Перевірити наявність** [**уразливого програмного забезпечення**](privilege-escalation/#vulnerable-software-installed) **встановленого**

### [Процеси](privilege-escalation/#processes)

* [ ] Чи працює яке-небудь **невідоме програмне забезпечення**?
* [ ] Чи працює яке-небудь програмне забезпечення з **більшими привілеями**, ніж має?
* [ ] Шукати **експлойти запущених процесів** (особливо версію, яка працює).
* [ ] Чи можете ви **змінити бінарний файл** будь-якого запущеного процесу?
* [ ] **Моніторинг процесів** та перевірка, чи працює який-небудь цікавий процес часто.
* [ ] Чи можете ви **читати** деяку цікаву **пам'ять процесу** (де можуть бути збережені паролі)?

### [Заплановані/Cron завдання?](privilege-escalation/#scheduled-jobs)

* [ ] Чи **ШЛЯХ** ](privilege-escalation/#cron-path)модифікується якимось кроном і ви можете **писати** в нього?
* [ ] Якісь [**маски** ](privilege-escalation/#cron-using-a-script-with-a-wildcard-wildcard-injection)у cron завданні?
* [ ] Деякий [**модифікований скрипт** ](privilege-escalation/#cron-script-overwriting-and-symlink)виконується або знаходиться в **модифікованій теки**?
* [ ] Чи виявлено, що деякі **скрипти** можуть бути або **виконуються дуже часто**](privilege-escalation/#frequent-cron-jobs)? (кожні 1, 2 або 5 хвилин)

### [Служби](privilege-escalation/#services)

* [ ] Чи є **записувальний файл .service**?
* [ ] Чи є **записувальний бінарний файл**, який виконується **службою**?
* [ ] Чи є **записувальна тека в шляху systemd**?

### [Таймери](privilege-escalation/#timers)

* [ ] Чи є **записувальний таймер**?

### [Сокети](privilege-escalation/#sockets)

* [ ] Чи є **записувальний файл .socket**?
* [ ] Чи можете ви **спілкуватися з будь-яким сокетом**?
* [ ] **HTTP сокети** з цікавою інформацією?

### [D-Bus](privilege-escalation/#d-bus)

* [ ] Чи можете ви **спілкуватися з будь-яким D-Bus**?

### [Мережа](privilege-escalation/#network)

* [ ] Енумерувати мережу, щоб знати, де ви знаходитесь
* [ ] **Відкриті порти, до яких ви не могли отримати доступ до отримання оболонки всередині машини?
* [ ] Чи можете ви **прослуховувати трафік** за допомогою `tcpdump`?

### [Користувачі](privilege-escalation/#users)

* [ ] Загальна **енумерація користувачів/груп**
* [ ] Чи у вас є **дуже великий UID**? Чи є **машини** **вразливими**?
* [ ] Чи можете ви [**підвищити привілеї завдяки групі**](privilege-escalation/interesting-groups-linux-pe/), до якої ви належите?
* [ ] **Дані буфера обміну**?
* [ ] Політика паролю?
* [ ] Спробуйте **використати** кожен **відомий пароль**, який ви вже виявили раніше, щоб увійти **з кожним** можливим **користувачем**. Спробуйте також увійти без пароля.

### [Записувальний ШЛЯХ](privilege-escalation/#writable-path-abuses)

* [ ] Якщо у вас є **права на запис у деякій теки в ШЛЯХ**, ви можете підвищити привілеї

### [SUDO та SUID команди](privilege-escalation/#sudo-and-suid)

* [ ] Чи можете ви виконати **будь-яку команду з sudo**? Чи можете ви використовувати її для ЧИТАННЯ, ЗАПИСУ або ВИКОНАННЯ чого-небудь як root? ([**GTFOBins**](https://gtfobins.github.io))
* [ ] Чи є **експлойтований SUID бінарний файл**? ([**GTFOBins**](https://gtfobins.github.io))
* [ ] Чи [**обмежені команди sudo** **шляхом**? чи можете ви **обійти** обмеження](privilege-escalation/#sudo-execution-bypassing-paths)?
* [ ] [**Sudo/SUID бінарний файл без вказаного шляху**](privilege-escalation/#sudo-command-suid-binary-without-command-path)?
* [ ] [**SUID бінарний файл з вказаним шляхом**](privilege-escalation/#suid-binary-with-command-path)? Обхід
* [ ] [**Уразливість LD\_PRELOAD**](privilege-escalation/#ld\_preload)
* [ ] [**Відсутність .so бібліотеки в SUID бінарному файлі**](privilege-escalation/#suid-binary-so-injection) з записувальної теки?
* [ ] [**Доступні токени SUDO**](privilege-escalation/#reusing-sudo-tokens)? [**Чи можете створити токен SUDO**](privilege-escalation/#var-run-sudo-ts-less-than-username-greater-than)?
* [ ] Чи можете [**читати або змінювати файли sudoers**](privilege-escalation/#etc-sudoers-etc-sudoers-d)?
* [ ] Чи можете [**змінювати /etc/ld.so.conf.d/**](privilege-escalation/#etc-ld-so-conf-d)?
* [ ] [**OpenBSD DOAS**](privilege-escalation/#doas) команда
### [Можливості](privilege-escalation/#capabilities)

* [ ] Чи має який-небудь виконуваний файл **неочікувані можливості**?

### [ACLs](privilege-escalation/#acls)

* [ ] Чи має який-небудь файл **неочікувані ACL**?

### [Відкриті сесії оболонки](privilege-escalation/#open-shell-sessions)

* [ ] **screen**
* [ ] **tmux**

### [SSH](privilege-escalation/#ssh)

* [ ] **Debian** [**OpenSSL Predictable PRNG - CVE-2008-0166**](privilege-escalation/#debian-openssl-predictable-prng-cve-2008-0166)
* [ ] [**Цікаві значення конфігурації SSH**](privilege-escalation/#ssh-interesting-configuration-values)

### [Цікаві файли](privilege-escalation/#interesting-files)

* [ ] **Файли профілю** - Чи можна прочитати чутливі дані? Записати для підвищення привілеїв?
* [ ] **Файли passwd/shadow** - Чи можна прочитати чутливі дані? Записати для підвищення привілеїв?
* [ ] **Перевірте загально цікаві теки** на наявність чутливих даних
* [ ] **Дивні місця/Файли, які належать**, до яких ви маєте доступ або можна змінити виконувані файли
* [ ] **Змінено** за останні хвилини
* [ ] **Файли баз даних Sqlite**
* [ ] **Приховані файли**
* [ ] **Скрипти/Виконувані файли в PATH**
* [ ] **Веб-файли** (паролі?)
* [ ] **Резервні копії**?
* [ ] **Відомі файли, що містять паролі**: Використовуйте **Linpeas** та **LaZagne**
* [ ] **Загальний пошук**

### [**Файли для запису**](privilege-escalation/#writable-files)

* [ ] **Змінити бібліотеку Python** для виконання довільних команд?
* [ ] Чи можна **змінити файли журналу**? Використовуйте експлойт **Logtotten**
* [ ] Чи можна **змінити /etc/sysconfig/network-scripts/**? Експлойт для Centos/Redhat
* [ ] Чи можна [**записувати в файли ini, int.d, systemd або rc.d**](privilege-escalation/#init-init-d-systemd-and-rc-d)?

### [**Інші трюки**](privilege-escalation/#other-tricks)

* [ ] Чи можна [**зловживати NFS для підвищення привілеїв**](privilege-escalation/#nfs-privilege-escalation)?
* [ ] Чи потрібно [**вибратися з обмеженої оболонки**](privilege-escalation/#escaping-from-restricted-shells)?
