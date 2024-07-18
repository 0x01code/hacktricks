# macOS Apps - Інспектування, налагодження та Fuzzing

{% hint style="success" %}
Вивчайте та практикуйте хакінг AWS: <img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**Школа хакінгу HackTricks AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Вивчайте та практикуйте хакінг GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**Школа хакінгу HackTricks GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Підтримайте HackTricks</summary>

* Перевірте [**плани підписки**](https://github.com/sponsors/carlospolop)!
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Поширюйте хакінг-прийоми, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) у GitHub.

</details>
{% endhint %}

### [WhiteIntel](https://whiteintel.io)

<figure><img src="../../../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) - це пошуковик, який працює на **темному веб-сайті** та пропонує **безкоштовні** функції для перевірки, чи були **компанія або її клієнти** **компрометовані** **шкідливими програмами-викрадачами**.

Основною метою WhiteIntel є боротьба з захопленням облікових записів та атаками вимагання викупу, що виникають внаслідок шкідливих програм, які крадуть інформацію.

Ви можете перевірити їх веб-сайт та спробувати їхній двигун **безкоштовно** за посиланням:

{% embed url="https://whiteintel.io" %}

***

## Статичний аналіз

### otool & objdump & nm
```bash
otool -L /bin/ls #List dynamically linked libraries
otool -tv /bin/ps #Decompile application
```
{% code overflow="wrap" %}
```bash
objdump -m --dylibs-used /bin/ls #List dynamically linked libraries
objdump -m -h /bin/ls # Get headers information
objdump -m --syms /bin/ls # Check if the symbol table exists to get function names
objdump -m --full-contents /bin/ls # Dump every section
objdump -d /bin/ls # Dissasemble the binary
objdump --disassemble-symbols=_hello --x86-asm-syntax=intel toolsdemo #Disassemble a function using intel flavour
```
{% endcode %}
```bash
nm -m ./tccd # List of symbols
```
### jtool2 & Disarm

Ви можете [**завантажити disarm звідси**](https://newosxbook.com/tools/disarm.html).
```bash
ARCH=arm64e disarm -c -i -I --signature /path/bin # Get bin info and signature
ARCH=arm64e disarm -c -l /path/bin # Get binary sections
ARCH=arm64e disarm -c -L /path/bin # Get binary commands (dependencies included)
ARCH=arm64e disarm -c -S /path/bin # Get symbols (func names, strings...)
ARCH=arm64e disarm -c -d /path/bin # Get disasembled
jtool2 -d __DATA.__const myipc_server | grep MIG # Get MIG info
```
Ви можете [**завантажити jtool2 тут**](http://www.newosxbook.com/tools/jtool.html) або встановити його за допомогою `brew`.
```bash
# Install
brew install --cask jtool2

jtool2 -l /bin/ls # Get commands (headers)
jtool2 -L /bin/ls # Get libraries
jtool2 -S /bin/ls # Get symbol info
jtool2 -d /bin/ls # Dump binary
jtool2 -D /bin/ls # Decompile binary

# Get signature information
ARCH=x86_64 jtool2 --sig /System/Applications/Automator.app/Contents/MacOS/Automator

# Get MIG information
jtool2 -d __DATA.__const myipc_server | grep MIG
```
{% hint style="danger" %}
**jtool застарів на користь disarm**
{% endhint %}

### Codesign / ldid

{% hint style="success" %}
**`Codesign`** можна знайти в **macOS**, тоді як **`ldid`** можна знайти в **iOS**
{% endhint %}
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

# Get signature info
ldid -h <binary>

# Get entitlements
ldid -e <binary>

# Change entilements
## /tmp/entl.xml is a XML file with the new entitlements to add
ldid -S/tmp/entl.xml <binary>
```
### SuspiciousPackage

[**SuspiciousPackage**](https://mothersruin.com/software/SuspiciousPackage/get.html) - це інструмент, корисний для інспектування файлів **.pkg** (інсталяторів) та перегляду їх вмісту перед встановленням.\
Ці інсталятори мають сценарії `preinstall` та `postinstall` bash, які зловживають зловмисники для **постійного** **розповсюдження** **зловмисного** **програмного** **забезпечення**.

### hdiutil

Цей інструмент дозволяє **монтувати** образи дисків Apple (**.dmg**) для їх інспектування перед запуском чого-небудь:
```bash
hdiutil attach ~/Downloads/Firefox\ 58.0.2.dmg
```
Це буде змонтовано в `/Volumes`

### Упаковані бінарні файли

* Перевірка високої ентропії
* Перевірка рядків (чи є майже незрозумілий рядок, упакований)
* Упаковувач UPX для MacOS генерує розділ під назвою "\_\_XHDR"

## Статичний аналіз Objective-C

### Метадані

{% hint style="danger" %}
Зверніть увагу, що програми, написані на Objective-C, **зберігають** свої оголошення класів **під час** **компіляції** у [бінарних файлах Mach-O](../macos-files-folders-and-binaries/universal-binaries-and-mach-o-format.md). Такі оголошення класів **включають** ім'я та тип:
{% endhint %}

* Визначені інтерфейси
* Методи інтерфейсу
* Змінні екземпляра інтерфейсу
* Визначені протоколи

Зверніть увагу, що ці імена можуть бути затемнені, щоб ускладнити розбір бінарного коду.

### Виклик функцій

Коли функція викликається в бінарному файлі, який використовує Objective-C, скомпільований код замість виклику цієї функції буде викликати **`objc_msgSend`**. Це викличе остаточну функцію:

![](<../../../.gitbook/assets/image (305).png>)

Параметри, які очікує ця функція, це:

* Перший параметр (**self**) - "вказівник, який вказує на **екземпляр класу, який має отримати повідомлення**". Іншими словами, це об'єкт, на якому викликається метод. Якщо метод є методом класу, це буде екземпляр об'єкта класу (в цілому), тоді як для методу екземпляра self вказуватиме на створений екземпляр класу як об'єкт.
* Другий параметр (**op**) - "селектор методу, який обробляє повідомлення". Це просто **ім'я методу**.
* Решта параметрів - будь-які **значення, які потрібні для методу** (op).

Дивіться, як **легко отримати цю інформацію за допомогою `lldb` в ARM64** на цій сторінці:

{% content-ref url="arm64-basic-assembly.md" %}
[arm64-basic-assembly.md](arm64-basic-assembly.md)
{% endcontent-ref %}

x64:

| **Аргумент**      | **Регістр**                                                    | **(для) objc\_msgSend**                                |
| ----------------- | --------------------------------------------------------------- | ------------------------------------------------------ |
| **1-й аргумент**  | **rdi**                                                         | **self: об'єкт, на якому викликається метод**         |
| **2-й аргумент**  | **rsi**                                                         | **op: ім'я методу**                                   |
| **3-й аргумент**  | **rdx**                                                         | **1-й аргумент методу**                               |
| **4-й аргумент**  | **rcx**                                                         | **2-й аргумент методу**                               |
| **5-й аргумент**  | **r8**                                                          | **3-й аргумент методу**                               |
| **6-й аргумент**  | **r9**                                                          | **4-й аргумент методу**                               |
| **7+ аргумент**   | <p><strong>rsp+</strong><br><strong>(на стеку)</strong></p>   | **5+ аргумент методу**                                |

### Вивантаження метаданих ObjectiveC

### Dynadump

[**Dynadump**](https://github.com/DerekSelander/dynadump) - це інструмент для розбору класів бінарних файлів Objective-C. У репозиторії GitHub зазначено dylibs, але це також працює з виконуваними файлами.
```bash
./dynadump dump /path/to/bin
```
На момент написання, це **зараз працює краще за все**.

#### Звичайні інструменти
```bash
nm --dyldinfo-only /path/to/bin
otool -ov /path/to/bin
objdump --macho --objc-meta-data /path/to/bin
```
#### class-dump

[**class-dump**](https://github.com/nygard/class-dump/) - це оригінальний інструмент для генерації декларацій для класів, категорій та протоколів у форматованому коді ObjetiveC.

Це старий і не підтримуваний інструмент, тому ймовірно, що він не працюватиме належним чином.

#### ICDump

[**iCDump**](https://github.com/romainthomas/iCDump) - це сучасний та крос-платформений дамп класів Objective-C. Порівняно з існуючими інструментами, iCDump може працювати незалежно від екосистеми Apple та надає зв'язки з Python.
```python
import icdump
metadata = icdump.objc.parse("/path/to/bin")

print(metadata.to_decl())
```
## Статичний аналіз Swift

З Swift бінарними файлами, оскільки є сумісність з Objective-C, іноді можна видобути оголошення, використовуючи [class-dump](https://github.com/nygard/class-dump/), але не завжди.

За допомогою командних рядків **`jtool -l`** або **`otool -l`** можна знайти кілька розділів, які починаються з префікса **`__swift5`**:
```bash
jtool2 -l /Applications/Stocks.app/Contents/MacOS/Stocks
LC 00: LC_SEGMENT_64              Mem: 0x000000000-0x100000000    __PAGEZERO
LC 01: LC_SEGMENT_64              Mem: 0x100000000-0x100028000    __TEXT
[...]
Mem: 0x100026630-0x100026d54        __TEXT.__swift5_typeref
Mem: 0x100026d60-0x100027061        __TEXT.__swift5_reflstr
Mem: 0x100027064-0x1000274cc        __TEXT.__swift5_fieldmd
Mem: 0x1000274cc-0x100027608        __TEXT.__swift5_capture
[...]
```
Ви можете знайти додаткову інформацію про **інформацію, збережену в цих розділах, у цьому дописі блогу**. 

Більше того, **бінарні файли Swift можуть містити символи** (наприклад, бібліотеки повинні зберігати символи, щоб їх функції можна було викликати). **Символи зазвичай містять інформацію про назву функції** та атрибути у некрасивому вигляді, тому вони дуже корисні, і є "**деманглери"**, які можуть отримати початкову назву:
```bash
# Ghidra plugin
https://github.com/ghidraninja/ghidra_scripts/blob/master/swift_demangler.py

# Swift cli
swift demangle
```
## Динамічний аналіз

{% hint style="warning" %}
Зверніть увагу, що для налагодження бінарних файлів **SIP повинен бути вимкнений** (`csrutil disable` або `csrutil enable --without debug`) або скопіювати бінарні файли в тимчасову теку та **видалити підпис** за допомогою `codesign --remove-signature <шлях-до-бінарного-файлу>` або дозволити налагодження бінарного файлу (можна скористатися [цим скриптом](https://gist.github.com/carlospolop/a66b8d72bb8f43913c4b5ae45672578b))
{% endhint %}

{% hint style="warning" %}
Зверніть увагу, що для **інструментування системних бінарних файлів** (таких як `cloudconfigurationd`) на macOS, **SIP повинен бути вимкнений** (просто видалення підпису не працюватиме).
{% endhint %}

### APIs

macOS надає деякі цікаві API, які надають інформацію про процеси:

* `proc_info`: Це основний, що надає багато інформації про кожен процес. Вам потрібно мати права root, щоб отримати інформацію про інші процеси, але вам не потрібні спеціальні дозволи або порти mach.
* `libsysmon.dylib`: Це дозволяє отримувати інформацію про процеси через викладені функції XPC, однак для цього потрібно мати дозвіл `com.apple.sysmond.client`.

### Stackshot та microstackshots

**Stackshotting** - це техніка, яка використовується для захоплення стану процесів, включаючи стеки викликів усіх запущених потоків. Це особливо корисно для налагодження, аналізу продуктивності та розуміння поведінки системи в конкретний момент часу. На iOS та macOS stackshotting можна виконати за допомогою кількох інструментів та методів, таких як інструменти **`sample`** та **`spindump`**.

### Sysdiagnose

Цей інструмент (`/usr/bini/ysdiagnose`) в основному збирає багато інформації з вашого комп'ютера, виконуючи десятки різних команд, таких як `ps`, `zprint`...

Він повинен бути запущений як **root**, а демон `/usr/libexec/sysdiagnosed` має дуже цікаві дозволи, такі як `com.apple.system-task-ports` та `get-task-allow`.

Його plist розташований в `/System/Library/LaunchDaemons/com.apple.sysdiagnose.plist`, який оголошує 3 MachServices:

* `com.apple.sysdiagnose.CacheDelete`: Видаляє старі архіви в /var/rmp
* `com.apple.sysdiagnose.kernel.ipc`: Спеціальний порт 23 (ядро)
* `com.apple.sysdiagnose.service.xpc`: Інтерфейс режиму користувача через клас `Libsysdiagnose` Obj-C. Три аргументи в словнику можуть бути передані (`compress`, `display`, `run`)

### Об'єднані журнали

macOS генерує багато журналів, які можуть бути дуже корисними при запуску додатка для розуміння **що він робить**.

Більше того, є деякі журнали, які міститимуть тег `<private>`, щоб **приховати** деяку **ідентифіковану** інформацію про **користувача** або **комп'ютер**. Однак можна **встановити сертифікат для розкриття цієї інформації**. Дотримуйтесь пояснень з [**тут**](https://superuser.com/questions/1532031/how-to-show-private-data-in-macos-unified-log).

### Hopper

#### Ліва панель

У лівій панелі hopper можна побачити символи (**Мітки**) бінарного файлу, список процедур та функцій (**Proc**) та рядки (**Str**). Це не всі рядки, але ті, що визначені в кількох частинах файлу Mac-O (наприклад _cstring або_ `objc_methname`).

#### Середня панель

У середній панелі ви можете побачити **розібраний код**. І ви можете побачити його у **сирому** розібранні, як **граф**, як **декомпільований** та як **бінарний**, натиснувши на відповідний значок:

<figure><img src="../../../.gitbook/assets/image (343).png" alt=""><figcaption></figcaption></figure>

Клацнувши правою кнопкою миші на об'єкті коду, ви можете побачити **посилання на/від цього об'єкту** або навіть змінити його ім'я (це не працює в декомпільованому псевдокоді):

<figure><img src="../../../.gitbook/assets/image (1117).png" alt=""><figcaption></figcaption></figure>

Більше того, у **середній нижній частині ви можете вводити команди python**.

#### Права панель

У правій панелі ви можете побачити цікаву інформацію, таку як **історію навігації** (щоб знати, як ви потрапили в поточну ситуацію), **граф викликів**, де ви можете побачити всі **функції, які викликають цю функцію**, і всі функції, які **ця функція викликає**, та інформацію про **локальні змінні**.

### dtrace

Це дозволяє користувачам отримати доступ до додатків на дуже **низькому рівні** та надає спосіб користувачам **слідкувати** за **програмами** та навіть змінювати їхній хід виконання. Dtrace використовує **проби**, які розміщені по всьому ядру і знаходяться в таких місцях, як початок та кінець системних викликів.

DTrace використовує функцію **`dtrace_probe_create`** для створення проби для кожного системного виклику. Ці проби можуть бути викликані в **точці входу та виходу кожного системного виклику**. Взаємодія з DTrace відбувається через /dev/dtrace, який доступний лише для користувача root.

{% hint style="success" %}
Щоб увімкнути Dtrace без повного вимкнення захисту SIP, ви можете виконати у режимі відновлення: `csrutil enable --without dtrace`

Ви також можете **`dtrace`** або **`dtruss`** бінарні файли, **які ви скомпілювали**.
{% endhint %}

Доступні проби dtrace можна отримати за допомогою:
```bash
dtrace -l | head
ID   PROVIDER            MODULE                          FUNCTION NAME
1     dtrace                                                     BEGIN
2     dtrace                                                     END
3     dtrace                                                     ERROR
43    profile                                                     profile-97
44    profile                                                     profile-199
```
Назва зонду складається з чотирьох частин: постачальник, модуль, функція та назва (`fbt:mach_kernel:ptrace:entry`). Якщо ви не вказуєте деяку частину назви, Dtrace застосує цю частину як шаблон.

Для налаштування DTrace для активації зондів та вказівки дій, які слід виконати при їх спрацюванні, нам знадобиться використовувати мову D.

Докладніше пояснення та більше прикладів можна знайти за посиланням [https://illumos.org/books/dtrace/chp-intro.html](https://illumos.org/books/dtrace/chp-intro.html)

#### Приклади

Запустіть `man -k dtrace`, щоб переглянути **доступні сценарії DTrace**. Приклад: `sudo dtruss -n binary`

* У рядку
```bash
#Count the number of syscalls of each running process
sudo dtrace -n 'syscall:::entry {@[execname] = count()}'
```
* сценарій
```bash
syscall:::entry
/pid == $1/
{
}

#Log every syscall of a PID
sudo dtrace -s script.d 1234
```

```bash
syscall::open:entry
{
printf("%s(%s)", probefunc, copyinstr(arg0));
}
syscall::close:entry
{
printf("%s(%d)\n", probefunc, arg0);
}

#Log files opened and closed by a process
sudo dtrace -s b.d -c "cat /etc/hosts"
```

```bash
syscall:::entry
{
;
}
syscall:::return
{
printf("=%d\n", arg1);
}

#Log sys calls with values
sudo dtrace -s syscalls_info.d -c "cat /etc/hosts"
```
### dtruss
```bash
dtruss -c ls #Get syscalls of ls
dtruss -c -p 1000 #get syscalls of PID 1000
```
### kdebug

Це засіб відстеження ядра. Документовані коди можна знайти в **`/usr/share/misc/trace.codes`**.

Інструменти, такі як `latency`, `sc_usage`, `fs_usage` та `trace`, використовують його внутрішньо.

Для взаємодії з `kdebug` використовується `sysctl` через простір імен `kern.kdebug`, а MIB-об'єкти для використання можна знайти в `sys/sysctl.h`, де функції реалізовані в `bsd/kern/kdebug.c`.

Для взаємодії з kdebug з власним клієнтом зазвичай виконуються такі кроки:

* Видалити існуючі налаштування за допомогою KERN\_KDSETREMOVE
* Встановити слідкування за допомогою KERN\_KDSETBUF та KERN\_KDSETUP
* Використовуйте KERN\_KDGETBUF, щоб отримати кількість записів у буфері
* Отримайте власного клієнта зі слідкування за допомогою KERN\_KDPINDEX
* Увімкніть слідкування за допомогою KERN\_KDENABLE
* Прочитайте буфер, викликаючи KERN\_KDREADTR
* Для відповідності кожного потоку його процесу викличте KERN\_KDTHRMAP.

Для отримання цієї інформації можна використовувати інструмент Apple **`trace`** або власний інструмент [kDebugView (kdv)](https://newosxbook.com/tools/kdv.html)**.**

**Зауважте, що Kdebug доступний лише для 1 клієнта одночасно.** Таким чином, одночасно може виконуватися лише один інструмент, який працює на основі k-debug.

### ktrace

API `ktrace_*` походять з `libktrace.dylib`, яка обгортає ті, що використовуються в `Kdebug`. Після цього клієнт може просто викликати `ktrace_session_create` та `ktrace_events_[single/class]`, щоб встановити зворотні виклики на конкретні коди, а потім запустити його за допомогою `ktrace_start`.

Ви можете використовувати це навіть з **SIP активованим**

Ви можете використовувати як клієнти утиліту `ktrace`:
```bash
ktrace trace -s -S -t c -c ls | grep "ls("
```
Або `tailspin`.

### kperf

Це використовується для профілювання на рівні ядра і побудоване за допомогою викликів `Kdebug`.

Основна змінна `kernel_debug_active` перевіряється, і якщо вона встановлена, викликається `kperf_kdebug_handler` з кодом `Kdebug` та адресою виклику ядра. Якщо код `Kdebug` відповідає вибраному, отримує "дії", налаштовані як бітова карта (перевірте `osfmk/kperf/action.h` для варіантів).

У Kperf також є таблиця MIB sysctl: (як root) `sysctl kperf`. Цей код можна знайти в `osfmk/kperf/kperfbsd.c`.

Крім того, підмножина функціоналу Kperf знаходиться в `kpc`, яка надає інформацію про лічильники продуктивності машини.

### ProcessMonitor

[**ProcessMonitor**](https://objective-see.com/products/utilities.html#ProcessMonitor) - це дуже корисний інструмент для перевірки дій, пов'язаних з процесами, які виконує процес (наприклад, відстеження нових процесів, які створює процес).

### SpriteTree

[**SpriteTree**](https://themittenmac.com/tools/) - це інструмент для відображення взаємозв'язків між процесами.\
Вам потрібно відстежувати свій Mac за допомогою команди, подібної **`sudo eslogger fork exec rename create > cap.json`** (термінал, що запускає це, потребує FDA). Потім ви можете завантажити json у цей інструмент, щоб переглянути всі взаємозв'язки:

<figure><img src="../../../.gitbook/assets/image (1182).png" alt="" width="375"><figcaption></figcaption></figure>

### FileMonitor

[**FileMonitor**](https://objective-see.com/products/utilities.html#FileMonitor) дозволяє відстежувати події файлів (такі як створення, зміни та видалення), надаючи докладну інформацію про такі події.

### Crescendo

[**Crescendo**](https://github.com/SuprHackerSteve/Crescendo) - це графічний інструмент з виглядом, який користувачі Windows можуть знати з _Procmon_ від Microsoft Sysinternal. Цей інструмент дозволяє почати та зупинити запис різних типів подій, фільтрувати ці події за категоріями, такими як файл, процес, мережа і т. д., і надає функціонал для збереження записаних подій у форматі json.

### Apple Instruments

[**Apple Instruments**](https://developer.apple.com/library/archive/documentation/Performance/Conceptual/CellularBestPractices/Appendix/Appendix.html) є частиною інструментів розробника Xcode, які використовуються для моніторингу продуктивності додатків, виявлення витоків пам'яті та відстеження активності файлової системи.

![](<../../../.gitbook/assets/image (1138).png>)

### fs\_usage

Дозволяє відстежувати дії, виконані процесами:
```bash
fs_usage -w -f filesys ls #This tracks filesystem actions of proccess names containing ls
fs_usage -w -f network curl #This tracks network actions
```
### TaskExplorer

[**Taskexplorer**](https://objective-see.com/products/taskexplorer.html) корисний для перегляду **бібліотек**, які використовує бінарний файл, **файлів**, які він використовує, та **мережевих** з'єднань.\
Також перевіряє процеси бінарного файлу на предмет **virustotal** та показує інформацію про бінарний файл.

## PT\_DENY\_ATTACH <a href="#page-title" id="page-title"></a>

У [**цьому дописі блогу**](https://knight.sc/debugging/2019/06/03/debugging-apple-binaries-that-use-pt-deny-attach.html) ви можете знайти приклад того, як **налагодити працюючий демон**, який використовував **`PT_DENY_ATTACH`** для запобігання налагодженню навіть якщо SIP був вимкнений.

### lldb

**lldb** - це фактичний інструмент для **налагодження** бінарних файлів **macOS**.
```bash
lldb ./malware.bin
lldb -p 1122
lldb -n malware.bin
lldb -n malware.bin --waitfor
```
Ви можете встановити intel flavour при використанні lldb, створивши файл під назвою **`.lldbinit`** у вашій домашній папці з наступним рядком:
```bash
settings set target.x86-disassembly-flavor intel
```
{% hint style="warning" %}
Усередині lldb виведіть процес за допомогою `process save-core`
{% endhint %}

<table data-header-hidden><thead><tr><th width="225"></th><th></th></tr></thead><tbody><tr><td><strong>(lldb) Команда</strong></td><td><strong>Опис</strong></td></tr><tr><td><strong>run (r)</strong></td><td>Початок виконання, яке триватиме безперервно до того, як буде досягнуто точку зупинки або процес завершиться.</td></tr><tr><td><strong>continue (c)</strong></td><td>Продовжити виконання процесу, який відлагоджується.</td></tr><tr><td><strong>nexti (n / ni)</strong></td><td>Виконати наступну інструкцію. Ця команда пропустить виклики функцій.</td></tr><tr><td><strong>stepi (s / si)</strong></td><td>Виконати наступну інструкцію. На відміну від команди nexti, ця команда увійде в виклики функцій.</td></tr><tr><td><strong>finish (f)</strong></td><td>Виконати решту інструкцій у поточній функції ("фрейм") та зупинити виконання.</td></tr><tr><td><strong>control + c</strong></td><td>Призупинити виконання. Якщо процес був запущений (r) або продовжений (c), це призведе до зупинки процесу ... де б він зараз виконувався.</td></tr><tr><td><strong>breakpoint (b)</strong></td><td><p>b main #Будь-яка функція з ім'ям main</p><p>b &#x3C;binname>`main #Головна функція бінарного файлу</p><p>b set -n main --shlib &#x3C;lib_name> #Головна функція вказаного бінарного файлу</p><p>b -[NSDictionary objectForKey:]</p><p>b -a 0x0000000100004bd9</p><p>br l #Список точок зупинки</p><p>br e/dis &#x3C;num> #Увімкнути/вимкнути точку зупинки</p><p>breakpoint delete &#x3C;num></p></td></tr><tr><td><strong>help</strong></td><td><p>help breakpoint #Отримати довідку про команду точки зупинки</p><p>help memory write #Отримати довідку щодо запису в пам'ять</p></td></tr><tr><td><strong>reg</strong></td><td><p>reg read</p><p>reg read $rax</p><p>reg read $rax --format &#x3C;<a href="https://lldb.llvm.org/use/variable.html#type-format">format</a>></p><p>reg write $rip 0x100035cc0</p></td></tr><tr><td><strong>x/s &#x3C;reg/memory address</strong></td><td>Показати пам'ять як рядок з завершенням нульового символу.</td></tr><tr><td><strong>x/i &#x3C;reg/memory address</strong></td><td>Показати пам'ять як машинну інструкцію.</td></tr><tr><td><strong>x/b &#x3C;reg/memory address</strong></td><td>Показати пам'ять як байт.</td></tr><tr><td><strong>print object (po)</strong></td><td><p>Це виведе об'єкт, на який посилається параметр</p><p>po $raw</p><p><code>{</code></p><p><code>dnsChanger = {</code></p><p><code>"affiliate" = "";</code></p><p><code>"blacklist_dns" = ();</code></p><p>Зверніть увагу, що більшість API-інтерфейсів або методів Objective-C від Apple повертають об'єкти, тому їх слід відображати за допомогою команди "print object" (po). Якщо po не виводить змістовний результат, використовуйте <code>x/b</code></p></td></tr><tr><td><strong>memory</strong></td><td>memory read 0x000....<br>memory read $x0+0xf2a<br>memory write 0x100600000 -s 4 0x41414141 #Записати AAAA за цією адресою<br>memory write -f s $rip+0x11f+7 "AAAA" #Записати AAAA за адресою</td></tr><tr><td><strong>disassembly</strong></td><td><p>dis #Розібрати поточну функцію</p><p>dis -n &#x3C;funcname> #Розібрати функцію</p><p>dis -n &#x3C;funcname> -b &#x3C;basename> #Розібрати функцію<br>dis -c 6 #Розібрати 6 рядків<br>dis -c 0x100003764 -e 0x100003768 # Від однієї адреси до іншої<br>dis -p -c 4 # Почати розібрати з поточної адреси</p></td></tr><tr><td><strong>parray</strong></td><td>parray 3 (char **)$x1 # Перевірити масив з 3 компонентами в регістрі x1</td></tr></tbody></table>

{% hint style="info" %}
При виклику функції **`objc_sendMsg`**, регістр **rsi** містить **ім'я методу** як рядок з завершенням нульового символу ("C"). Щоб вивести ім'я через lldb, виконайте:

`(lldb) x/s $rsi: 0x1000f1576: "startMiningWithPort:password:coreCount:slowMemory:currency:"`

`(lldb) print (char*)$rsi:`\
`(char *) $1 = 0x00000001000f1576 "startMiningWithPort:password:coreCount:slowMemory:currency:"`

`(lldb) reg read $rsi: rsi = 0x00000001000f1576 "startMiningWithPort:password:coreCount:slowMemory:currency:"`
{% endhint %}

### Протидія динамічному аналізу

#### Виявлення віртуальних машин

* Команда **`sysctl hw.model`** повертає "Mac", коли **хост - MacOS**, але щось інше, коли це віртуальна машина.
* Граючи зі значеннями **`hw.logicalcpu`** та **`hw.physicalcpu`**, деякі шкідливі програми намагаються виявити, чи це віртуальна машина.
* Деякі шкідливі програми також можуть **виявити**, чи машина базується на **VMware** за MAC-адресою (00:50:56).
* Також можна виявити, чи процес **відлагоджується** за допомогою простого коду, наприклад:
* `if(P_TRACED == (info.kp_proc.p_flag & P_TRACED)){ //процес відлагоджується }`
* Також можна викликати системний виклик **`ptrace`** з прапорцем **`PT_DENY_ATTACH`**. Це **запобігає** приєднанню та відстеженню відлагоджувача.
* Можна перевірити, чи функція **`sysctl`** або **`ptrace`** імпортується (але шкідлива програма може імпортувати її динамічно)
* Як вказано в цьому огляді, “[Перемога над анти-відлагоджувальними техніками: варіанти macOS ptrace](https://alexomara.com/blog/defeating-anti-debug-techniques-macos-ptrace-variants/)” :\
“_Повідомлення Процес # завершився з **статусом = 45 (0x0000002d)** зазвичай є вказівкою на те, що ціль відлагоджування використовує **PT\_DENY\_ATTACH**_”
## Ядерні дампи

Ядерні дампи створюються, якщо:

- `kern.coredump` sysctl встановлено на 1 (за замовчуванням)
- Якщо процес не був suid/sgid або `kern.sugid_coredump` дорівнює 1 (за замовчуванням 0)
- Ліміт `AS_CORE` дозволяє операцію. Можливо приглушити створення ядерних дампів, викликавши `ulimit -c 0`, і знову їх увімкнути за допомогою `ulimit -c unlimited`.

У цих випадках ядерні дампи генеруються відповідно до `kern.corefile` sysctl і зазвичай зберігаються в `/cores/core/.%P`.

## Fuzzing

### [ReportCrash](https://ss64.com/osx/reportcrash.html)

ReportCrash **аналізує процеси, що призводять до збоїв, і зберігає звіт про збій на диск**. Звіт про збій містить інформацію, яка може **допомогти розробнику діагностувати** причину збою.\
Для додатків та інших процесів, що **працюють в контексті запуску користувача**, ReportCrash працює як LaunchAgent і зберігає звіти про збої в `~/Library/Logs/DiagnosticReports/` користувача\
Для демонів, інших процесів, що **працюють в контексті запуску системи** та інших привілейованих процесів, ReportCrash працює як LaunchDaemon і зберігає звіти про збої в `/Library/Logs/DiagnosticReports`

Якщо вас турбують звіти про збої, **які надсилаються в Apple**, ви можете їх вимкнути. Якщо ні, звіти про збої можуть бути корисними для **визначення причини збою сервера**.
```bash
#To disable crash reporting:
launchctl unload -w /System/Library/LaunchAgents/com.apple.ReportCrash.plist
sudo launchctl unload -w /System/Library/LaunchDaemons/com.apple.ReportCrash.Root.plist

#To re-enable crash reporting:
launchctl load -w /System/Library/LaunchAgents/com.apple.ReportCrash.plist
sudo launchctl load -w /System/Library/LaunchDaemons/com.apple.ReportCrash.Root.plist
```
### Сон

Під час розгортання в MacOS важливо не дозволяти Mac спати:

* systemsetup -setsleep Never
* pmset, System Preferences
* [KeepingYouAwake](https://github.com/newmarcel/KeepingYouAwake)

#### Відключення SSH

Якщо ви розгортаєте через SSH-з'єднання, важливо переконатися, що сеанс не завершиться. Тому змініть файл sshd\_config наступним чином:

* TCPKeepAlive Yes
* ClientAliveInterval 0
* ClientAliveCountMax 0
```bash
sudo launchctl unload /System/Library/LaunchDaemons/ssh.plist
sudo launchctl load -w /System/Library/LaunchDaemons/ssh.plist
```
### Внутрішні обробники

**Перевірте наступну сторінку**, щоб дізнатися, яка програма відповідає за **обробку вказаної схеми або протоколу:**

{% content-ref url="../macos-file-extension-apps.md" %}
[macos-file-extension-apps.md](../macos-file-extension-apps.md)
{% endcontent-ref %}

### Перелік мережевих процесів

Це цікаво виявити процеси, які керують мережевими даними:
```bash
dtrace -n 'syscall::recv*:entry { printf("-> %s (pid=%d)", execname, pid); }' >> recv.log
#wait some time
sort -u recv.log > procs.txt
cat procs.txt
```
Або скористайтеся `netstat` або `lsof`

### Libgmalloc

<figure><img src="../../../.gitbook/assets/Pasted Graphic 14.png" alt=""><figcaption></figcaption></figure>

{% code overflow="wrap" %}
```bash
lldb -o "target create `which some-binary`" -o "settings set target.env-vars DYLD_INSERT_LIBRARIES=/usr/lib/libgmalloc.dylib" -o "run arg1 arg2" -o "bt" -o "reg read" -o "dis -s \$pc-32 -c 24 -m -F intel" -o "quit"
```
{% endcode %}

### Fuzzers

#### [AFL++](https://github.com/AFLplusplus/AFLplusplus)

Працює для інструментів командного рядка

#### [Litefuzz](https://github.com/sec-tools/litefuzz)

Він "**просто працює"** з інструментами GUI macOS. Зверніть увагу, що деякі додатки macOS мають певні вимоги, такі як унікальні імена файлів, правильне розширення, потрібно читати файли з пісочниці (`~/Library/Containers/com.apple.Safari/Data`)...

Деякі приклади:
```bash
# iBooks
litefuzz -l -c "/System/Applications/Books.app/Contents/MacOS/Books FUZZ" -i files/epub -o crashes/ibooks -t /Users/test/Library/Containers/com.apple.iBooksX/Data/tmp -x 10 -n 100000 -ez

# -l : Local
# -c : cmdline with FUZZ word (if not stdin is used)
# -i : input directory or file
# -o : Dir to output crashes
# -t : Dir to output runtime fuzzing artifacts
# -x : Tmeout for the run (default is 1)
# -n : Num of fuzzing iterations (default is 1)
# -e : enable second round fuzzing where any crashes found are reused as inputs
# -z : enable malloc debug helpers

# Font Book
litefuzz -l -c "/System/Applications/Font Book.app/Contents/MacOS/Font Book FUZZ" -i input/fonts -o crashes/font-book -x 2 -n 500000 -ez

# smbutil (using pcap capture)
litefuzz -lk -c "smbutil view smb://localhost:4455" -a tcp://localhost:4455 -i input/mac-smb-resp -p -n 100000 -z

# screensharingd (using pcap capture)
litefuzz -s -a tcp://localhost:5900 -i input/screenshared-session --reportcrash screensharingd -p -n 100000
```
{% endcode %}

### Додаткова інформація про Fuzzing MacOS

* [https://www.youtube.com/watch?v=T5xfL9tEg44](https://www.youtube.com/watch?v=T5xfL9tEg44)
* [https://github.com/bnagy/slides/blob/master/OSXScale.pdf](https://github.com/bnagy/slides/blob/master/OSXScale.pdf)
* [https://github.com/bnagy/francis/tree/master/exploitaben](https://github.com/bnagy/francis/tree/master/exploitaben)
* [https://github.com/ant4g0nist/crashwrangler](https://github.com/ant4g0nist/crashwrangler)

## Посилання

* [**OS X Incident Response: Scripting and Analysis**](https://www.amazon.com/OS-Incident-Response-Scripting-Analysis-ebook/dp/B01FHOHHVS)
* [**https://www.youtube.com/watch?v=T5xfL9tEg44**](https://www.youtube.com/watch?v=T5xfL9tEg44)
* [**https://taomm.org/vol1/analysis.html**](https://taomm.org/vol1/analysis.html)
* [**The Art of Mac Malware: The Guide to Analyzing Malicious Software**](https://taomm.org/)

### [WhiteIntel](https://whiteintel.io)

<figure><img src="../../../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) - це пошуковий двигун, який працює на **темному веб-сайті** і надає **безкоштовні** функціональні можливості для перевірки, чи були **компанія або її клієнти** **пошкоджені** **шкідливими програмами для крадіжки інформації**.

Основною метою WhiteIntel є боротьба з захопленням облікових записів та атаками вимагання викупу, що виникають внаслідок шкідливих програм для крадіжки інформації.

Ви можете перевірити їх веб-сайт та спробувати їх двигун **безкоштовно** за посиланням:

{% embed url="https://whiteintel.io" %}

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
