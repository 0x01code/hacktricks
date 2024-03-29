# Універсальні бінарні файли macOS та формат Mach-O

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію рекламовану на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) **та** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **репозиторіїв на GitHub**.

</details>

## Основна інформація

Бінарні файли Mac OS зазвичай компілюються як **універсальні бінарні файли**. **Універсальний бінарний файл** може **підтримувати кілька архітектур у одному файлі**.

Ці бінарні файли відповідають **структурі Mach-O**, яка в основному складається з:

* Заголовка
* Команд завантаження
* Даних

![https://alexdremov.me/content/images/2022/10/6XLCD.gif](<../../../.gitbook/assets/image (559).png>)

## Fat Header

Шукайте файл за допомогою: `mdfind fat.h | grep -i mach-o | grep -E "fat.h$"`

<pre class="language-c"><code class="lang-c"><strong>#define FAT_MAGIC	0xcafebabe
</strong><strong>#define FAT_CIGAM	0xbebafeca	/* NXSwapLong(FAT_MAGIC) */
</strong>
struct fat_header {
<strong>	uint32_t	magic;		/* FAT_MAGIC or FAT_MAGIC_64 */
</strong><strong>	uint32_t	nfat_arch;	/* number of structs that follow */
</strong>};

struct fat_arch {
cpu_type_t	cputype;	/* cpu specifier (int) */
cpu_subtype_t	cpusubtype;	/* machine specifier (int) */
uint32_t	offset;		/* file offset to this object file */
uint32_t	size;		/* size of this object file */
uint32_t	align;		/* alignment as a power of 2 */
};
</code></pre>

Заголовок містить **магічні** байти, за якими слідує **кількість** **архітектур**, які містить файл (`nfat_arch`), і кожна архітектура матиме структуру `fat_arch`.

Перевірте це за допомогою:

<pre class="language-shell-session"><code class="lang-shell-session">% file /bin/ls
/bin/ls: Mach-O universal binary with 2 architectures: [x86_64:Mach-O 64-bit executable x86_64] [arm64e:Mach-O 64-bit executable arm64e]
/bin/ls (for architecture x86_64):	Mach-O 64-bit executable x86_64
/bin/ls (for architecture arm64e):	Mach-O 64-bit executable arm64e

% otool -f -v /bin/ls
Fat headers
fat_magic FAT_MAGIC
<strong>nfat_arch 2
</strong><strong>architecture x86_64
</strong>    cputype CPU_TYPE_X86_64
cpusubtype CPU_SUBTYPE_X86_64_ALL
capabilities 0x0
<strong>    offset 16384
</strong><strong>    size 72896
</strong>    align 2^14 (16384)
<strong>architecture arm64e
</strong>    cputype CPU_TYPE_ARM64
cpusubtype CPU_SUBTYPE_ARM64E
capabilities PTR_AUTH_VERSION USERSPACE 0
<strong>    offset 98304
</strong><strong>    size 88816
</strong>    align 2^14 (16384)
</code></pre>

або використовуючи інструмент [Mach-O View](https://sourceforge.net/projects/machoview/):

<figure><img src="../../../.gitbook/assets/image (5) (1) (1) (3) (1).png" alt=""><figcaption></figcaption></figure>

Як ви, можливо, мислите, зазвичай універсальний бінарний файл, скомпільований для 2 архітектур, **подвоює розмір** того, що скомпільовано лише для 1 архітектури.

## **Заголовок Mach-O**

Заголовок містить базову інформацію про файл, таку як магічні байти для ідентифікації його як файлу Mach-O та інформацію про цільову архітектуру. Ви можете знайти його за допомогою: `mdfind loader.h | grep -i mach-o | grep -E "loader.h$"`
```c
#define	MH_MAGIC	0xfeedface	/* the mach magic number */
#define MH_CIGAM	0xcefaedfe	/* NXSwapInt(MH_MAGIC) */
struct mach_header {
uint32_t	magic;		/* mach magic number identifier */
cpu_type_t	cputype;	/* cpu specifier (e.g. I386) */
cpu_subtype_t	cpusubtype;	/* machine specifier */
uint32_t	filetype;	/* type of file (usage and alignment for the file) */
uint32_t	ncmds;		/* number of load commands */
uint32_t	sizeofcmds;	/* the size of all the load commands */
uint32_t	flags;		/* flags */
};

#define MH_MAGIC_64 0xfeedfacf /* the 64-bit mach magic number */
#define MH_CIGAM_64 0xcffaedfe /* NXSwapInt(MH_MAGIC_64) */
struct mach_header_64 {
uint32_t	magic;		/* mach magic number identifier */
int32_t		cputype;	/* cpu specifier */
int32_t		cpusubtype;	/* machine specifier */
uint32_t	filetype;	/* type of file */
uint32_t	ncmds;		/* number of load commands */
uint32_t	sizeofcmds;	/* the size of all the load commands */
uint32_t	flags;		/* flags */
uint32_t	reserved;	/* reserved */
};
```
**Типи файлів**:

* MH\_EXECUTE (0x2): Стандартний виконавчий файл Mach-O
* MH\_DYLIB (0x6): Динамічна зв'язана бібліотека Mach-O (тобто .dylib)
* MH\_BUNDLE (0x8): Пакунок Mach-O (тобто .bundle)
```bash
# Checking the mac header of a binary
otool -arch arm64e -hv /bin/ls
Mach header
magic  cputype cpusubtype  caps    filetype ncmds sizeofcmds      flags
MH_MAGIC_64    ARM64          E USR00     EXECUTE    19       1728   NOUNDEFS DYLDLINK TWOLEVEL PIE
```
Або використовуючи [Mach-O View](https://sourceforge.net/projects/machoview/):

<figure><img src="../../../.gitbook/assets/image (4) (1) (4).png" alt=""><figcaption></figcaption></figure>

## **Команди завантаження Mach-O**

**Макет файлу в пам'яті** вказаний тут, деталізуючи **розташування таблиці символів**, контекст основного потоку під час початку виконання та необхідні **спільні бібліотеки**. Інструкції надаються динамічному завантажувачу **(dyld)** щодо процесу завантаження бінарного файлу в пам'ять.

Використовує структуру **load\_command**, визначену в зазначеному **`loader.h`**:
```objectivec
struct load_command {
uint32_t cmd;           /* type of load command */
uint32_t cmdsize;       /* total size of command in bytes */
};
```
Існує близько **50 різних типів команд завантаження**, які система обробляє по-різному. Найпоширеніші з них: `LC_SEGMENT_64`, `LC_LOAD_DYLINKER`, `LC_MAIN`, `LC_LOAD_DYLIB` та `LC_CODE_SIGNATURE`.

### **LC\_SEGMENT/LC\_SEGMENT\_64**

{% hint style="success" %}
Фактично, цей тип команди завантаження визначає, **як завантажувати сегменти \_\_TEXT** (виконавчий код) **та \_\_DATA** (дані для процесу) **згідно з вказаними зміщеннями в розділі Дані** при виконанні бінарного файлу.
{% endhint %}

Ці команди **визначають сегменти**, які **відображаються** в **віртуальному просторі пам'яті** процесу при його виконанні.

Існують **різні типи** сегментів, такі як сегмент **\_\_TEXT**, який містить виконавчий код програми, та сегмент **\_\_DATA**, який містить дані, використовувані процесом. Ці **сегменти розташовані в розділі даних** файлу Mach-O.

**Кожен сегмент** може бути поділений на кілька **секцій**. Структура **команди завантаження** містить **інформацію** про **ці секції** у відповідному сегменті.

У заголовку спочатку ви знаходите **заголовок сегмента**:

<pre class="language-c"><code class="lang-c">struct segment_command_64 { /* для 64-бітних архітектур */
uint32_t	cmd;		/* LC_SEGMENT_64 */
uint32_t	cmdsize;	/* включає розмір структур section_64 */
char		segname[16];	/* назва сегмента */
uint64_t	vmaddr;		/* адреса пам'яті цього сегмента */
uint64_t	vmsize;		/* розмір пам'яті цього сегмента */
uint64_t	fileoff;	/* зміщення файлу цього сегмента */
uint64_t	filesize;	/* кількість для відображення з файлу */
int32_t		maxprot;	/* максимальний захист VM */
int32_t		initprot;	/* початковий захист VM */
<strong>	uint32_t	nsects;		/* кількість секцій у сегменті */
</strong>	uint32_t	flags;		/* прапорці */
};
</code></pre>

Приклад заголовка сегмента:

<figure><img src="../../../.gitbook/assets/image (2) (2) (1) (1).png" alt=""><figcaption></figcaption></figure>

Цей заголовок визначає **кількість секцій, заголовки яких з'являються після** нього:
```c
struct section_64 { /* for 64-bit architectures */
char		sectname[16];	/* name of this section */
char		segname[16];	/* segment this section goes in */
uint64_t	addr;		/* memory address of this section */
uint64_t	size;		/* size in bytes of this section */
uint32_t	offset;		/* file offset of this section */
uint32_t	align;		/* section alignment (power of 2) */
uint32_t	reloff;		/* file offset of relocation entries */
uint32_t	nreloc;		/* number of relocation entries */
uint32_t	flags;		/* flags (section type and attributes)*/
uint32_t	reserved1;	/* reserved (for offset or index) */
uint32_t	reserved2;	/* reserved (for count or sizeof) */
uint32_t	reserved3;	/* reserved */
};
```
Приклад **заголовка розділу**:

<figure><img src="../../../.gitbook/assets/image (6) (2).png" alt=""><figcaption></figcaption></figure>

Якщо ви **додаєте** **зміщення розділу** (0x37DC) + **зміщення**, де **починається архітектура**, у цьому випадку `0x18000` --> `0x37DC + 0x18000 = 0x1B7DC`

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Також можна отримати **інформацію про заголовки** з **командного рядка** за допомогою:
```bash
otool -lv /bin/ls
```
Загальні сегменти, завантажені цією командою:

* **`__PAGEZERO`:** Це вказує ядру **відобразити** **адресу нуль**, щоб її **не можна було читати, записувати або виконувати**. Змінні maxprot та minprot у структурі встановлені на нуль, щоб показати, що на цій сторінці **немає прав на читання-запис-виконання**.
* Це виділення важливе для **пом'якшення вразливостей нульових вказівників**.
* **`__TEXT`**: Містить **виконуваний** **код** з правами на **читання** та **виконання** (без можливості запису)**.** Загальні розділи цього сегмента:
* `__text`: Скомпільований бінарний код
* `__const`: Константні дані
* `__cstring`: Рядкові константи
* `__stubs` та `__stubs_helper`: Використовуються під час процесу завантаження динамічних бібліотек
* **`__DATA`**: Містить дані, які можна **читати** та **писати** (без можливості виконання)**.**
* `__data`: Глобальні змінні (які були ініціалізовані)
* `__bss`: Статичні змінні (які не були ініціалізовані)
* `__objc_*` (\_\_objc\_classlist, \_\_objc\_protolist, тощо): Інформація, використовувана в середовищі виконання Objective-C
* **`__LINKEDIT`**: Містить інформацію для лінкера (dyld), таку як "запис, рядок та записи таблиці перенесення."
* **`__OBJC`**: Містить інформацію, використовувану в середовищі виконання Objective-C. Хоча цю інформацію також можна знайти в сегменті \_\_DATA, у різних розділах \_\_objc\_\*.

### **`LC_MAIN`**

Містить точку входу в атрибуті **entryoff**. Під час завантаження **dyld** просто **додає** це значення до (в пам'яті) **бази бінарного файлу**, а потім **переходить** до цієї інструкції для початку виконання коду бінарного файлу.

### **LC\_CODE\_SIGNATURE**

Містить інформацію про **підпис коду файлу Mach-O**. Він містить лише **зсув**, який **вказує** на **блоб підпису**. Зазвичай це знаходиться в самому кінці файлу.\
Однак деяку інформацію про цей розділ можна знайти в [**цьому блозі**](https://davedelong.com/blog/2018/01/10/reading-your-own-entitlements/) та цьому [**gists**](https://gist.github.com/carlospolop/ef26f8eb9fafd4bc22e69e1a32b81da4).

### **LC\_LOAD\_DYLINKER**

Містить **шлях до виконавчого файлу динамічного лінкера**, який відображає спільні бібліотеки в адресний простір процесу. **Значення завжди встановлене на `/usr/lib/dyld`**. Важливо зауважити, що в macOS відображення dylib відбувається в **режимі користувача**, а не в режимі ядра.

### **`LC_LOAD_DYLIB`**

Ця команда завантаження описує залежність **динамічної** **бібліотеки**, яка **вказує** **завантажувачу** (dyld) **завантажити та зв'язати цю бібліотеку**. Існує команда завантаження LC\_LOAD\_DYLIB **для кожної бібліотеки**, яку потребує бінарний файл Mach-O.

* Ця команда завантаження є структурою типу **`dylib_command`** (яка містить структуру dylib, що описує фактичну залежну динамічну бібліотеку):
```objectivec
struct dylib_command {
uint32_t        cmd;            /* LC_LOAD_{,WEAK_}DYLIB */
uint32_t        cmdsize;        /* includes pathname string */
struct dylib    dylib;          /* the library identification */
};

struct dylib {
union lc_str  name;                 /* library's path name */
uint32_t timestamp;                 /* library's build time stamp */
uint32_t current_version;           /* library's current version number */
uint32_t compatibility_version;     /* library's compatibility vers number*/
};
```
![](<../../../.gitbook/assets/image (558).png>)

Ви також можете отримати цю інформацію з командного рядка за допомогою:
```bash
otool -L /bin/ls
/bin/ls:
/usr/lib/libutil.dylib (compatibility version 1.0.0, current version 1.0.0)
/usr/lib/libncurses.5.4.dylib (compatibility version 5.4.0, current version 5.4.0)
/usr/lib/libSystem.B.dylib (compatibility version 1.0.0, current version 1319.0.0)
```
Деякі потенційно пов'язані з шкідливим ПЗ бібліотеки:

* **DiskArbitration**: Моніторинг USB-накопичувачів
* **AVFoundation:** Захоплення аудіо та відео
* **CoreWLAN**: Сканування Wifi.

{% hint style="info" %}
Mach-O бінарний файл може містити один або **більше конструкторів**, які будуть **виконані перед** адресою, вказаною в **LC\_MAIN**.\
Зміщення будь-яких конструкторів зберігаються в розділі **\_\_mod\_init\_func** сегмента **\_\_DATA\_CONST**.
{% endhint %}

## **Дані Mach-O**

У основі файлу знаходиться область даних, яка складається з кількох сегментів, які визначені в області команд завантаження. **У кожному сегменті можуть міститися різні секції даних**, причому кожна секція **містить код або дані**, що специфічні для типу.

{% hint style="success" %}
Дані - це в основному частина, що містить всю **інформацію**, яка завантажується командами завантаження **LC\_SEGMENTS\_64**
{% endhint %}

![https://www.oreilly.com/api/v2/epubs/9781785883378/files/graphics/B05055\_02\_38.jpg](<../../../.gitbook/assets/image (507) (3).png>)

Це включає:

* **Таблиця функцій:** Яка містить інформацію про функції програми.
* **Таблиця символів**: Яка містить інформацію про зовнішню функцію, використану бінарним файлом
* Також може містити внутрішні функції, назви змінних та інше.

Для перевірки цього можна використовувати інструмент [**Mach-O View**](https://sourceforge.net/projects/machoview/):

<figure><img src="../../../.gitbook/assets/image (2) (1) (4).png" alt=""><figcaption></figcaption></figure>

Або з командного рядка:
```bash
size -m /bin/ls
```
<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію рекламовану на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>
