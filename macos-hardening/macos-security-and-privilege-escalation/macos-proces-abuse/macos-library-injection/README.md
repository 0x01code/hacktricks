# macOS Library Injection

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити вашу **компанію в рекламі на HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний PEASS & HackTricks мерч**](https://peass.creator-spring.com)
* Дізнайтеся про [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв на GitHub.

</details>

{% hint style="danger" %}
Код **dyld є відкритим джерелом** і можна знайти за посиланням [https://opensource.apple.com/source/dyld/](https://opensource.apple.com/source/dyld/) та завантажити у вигляді tar за допомогою **URL, такого як** [https://opensource.apple.com/tarballs/dyld/dyld-852.2.tar.gz](https://opensource.apple.com/tarballs/dyld/dyld-852.2.tar.gz)
{% endhint %}

## **DYLD\_INSERT\_LIBRARIES**

Це схоже на [**LD\_PRELOAD на Linux**](../../../../linux-hardening/privilege-escalation/#ld\_preload). Це дозволяє вказати процесу, що буде запущено для завантаження певної бібліотеки з шляху (якщо змінна середовища ввімкнена).

Цю техніку також можна **використовувати як техніку ASEP**, оскільки кожна встановлена програма має файл plist під назвою "Info.plist", який дозволяє **призначати змінні середовища** за допомогою ключа під назвою `LSEnvironmental`.

{% hint style="info" %}
Починаючи з 2012 року **Apple драматично зменшила потужність** **`DYLD_INSERT_LIBRARIES`**.

Перейдіть до коду та **перевірте `src/dyld.cpp`**. У функції **`pruneEnvironmentVariables`** ви можете побачити, що змінні **`DYLD_*`** видаляються.

У функції **`processRestricted`** встановлюється причина обмеження. Перевіривши цей код, ви побачите, що причини такі:

* Бінарний файл має `setuid/setgid`
* Існування розділу `__RESTRICT/__restrict` у бінарному файлі macho.
* Програмне забезпечення має entitlements (захищений режим) без entitlement [`com.apple.security.cs.allow-dyld-environment-variables`](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_allow-dyld-environment-variables)
* Перевірте **entitlements** бінарного файлу за допомогою: `codesign -dv --entitlements :- </path/to/bin>`

У більш оновлених версіях ви можете знайти цю логіку у другій частині функції **`configureProcessRestrictions`.** Однак, у новіших версіях виконуються **початкові перевірки функції** (ви можете видалити умови, що стосуються iOS або симуляції, оскільки вони не будуть використовуватися в macOS.
{% endhint %}

### Перевірка бібліотек

Навіть якщо бінарний файл дозволяє використовувати змінну середовища **`DYLD_INSERT_LIBRARIES`**, якщо бінарний файл перевіряє підпис бібліотеки для її завантаження, він не завантажить власну бібліотеку.

Для завантаження власної бібліотеки бінарний файл повинен мати **один з наступних entitlements**:

* [`com.apple.security.cs.disable-library-validation`](../../macos-security-protections/macos-dangerous-entitlements.md#com.apple.security.cs.disable-library-validation)
* [`com.apple.private.security.clear-library-validation`](../../macos-security-protections/macos-dangerous-entitlements.md#com.apple.private.security.clear-library-validation)

або бінарний файл **не повинен** мати прапорець **hardened runtime** або прапорець **library validation**.

Ви можете перевірити, чи має бінарний файл **hardened runtime** за допомогою `codesign --display --verbose <bin>` перевіряючи прапорець runtime в **`CodeDirectory`** наприклад: **`CodeDirectory v=20500 size=767 flags=0x10000(runtime) hashes=13+7 location=embedded`**

Ви також можете завантажити бібліотеку, якщо вона **підписана тим самим сертифікатом, що й бінарний файл**.

Знайдіть приклад того, як (зловживати) використовувати це та перевірте обмеження в:

{% content-ref url="macos-dyld-hijacking-and-dyld_insert_libraries.md" %}
[macos-dyld-hijacking-and-dyld\_insert\_libraries.md](macos-dyld-hijacking-and-dyld\_insert\_libraries.md)
{% endcontent-ref %}

## Перехоплення Dylib

{% hint style="danger" %}
Пам'ятайте, що **попередні обмеження перевірки бібліотеки також застосовуються** для виконання атак перехоплення Dylib.
{% endhint %}

Як і в Windows, в MacOS ви також можете **перехоплювати dylibs**, щоб зробити, щоб **додатки виконували** **довільний** **код** (ну, фактично від звичайного користувача це може бути неможливо, оскільки вам може знадобитися дозвіл TCC для запису всередині пакета `.app` та перехоплення бібліотеки).\
Однак, спосіб, яким **додатки MacOS** **завантажують** **бібліотеки**, **більш обмежений**, ніж в Windows. Це означає, що розробники **шкідливого програмного забезпечення все ще можуть використовувати цю техніку для** **приховування**, але ймовірність **зловживання цим для підвищення привілеїв набагато нижча**.

По-перше, **частіше зустрічається**, що **бінарні файли MacOS вказують повний шлях** до бібліотек для завантаження. І по-друге, **MacOS ніколи не шукає** в папках **$PATH** бібліотеки.

**Основна** частина **коду**, що стосується цієї функціональності, знаходиться в **`ImageLoader::recursiveLoadLibraries`** в `ImageLoader.cpp`.

Існують **4 різні команди заголовка**, які може використовувати бінарний файл macho для завантаження бібліотек:

* Команда **`LC_LOAD_DYLIB`** - це загальна команда для завантаження dylib.
* Команда **`LC_LOAD_WEAK_DYLIB`** працює так само, як попередня, але якщо dylib не знайдено, виконання продовжується без будь-якої помилки.
* Команда **`LC_REEXPORT_DYLIB`** - це проксі (або повторне експортування) символів з іншої бібліотеки.
* Команда **`LC_LOAD_UPWARD_DYLIB`** використовується, коли дві бібліотеки залежать одна від одної (це називається _upward dependency_).

Однак існують **2 типи перехоплення dylib**:

* **Відсутні слабкі зв'язані бібліотеки**: Це означає, що додаток спробує завантажити бібліотеку, якої не існує, налаштовану за допомогою **LC\_LOAD\_WEAK\_DYLIB**. Потім, **якщо зловмисник розмістить dylib там, де вона очікується, вона буде завантажена**.
* Те, що посилання є "слабким", означає, що додаток продовжить працювати навіть якщо бібліотека не знайдена.
* **Код, що стосується цього**, знаходиться в функції `ImageLoaderMachO::doGetDependentLibraries` файлу `ImageLoaderMachO.cpp`, де `lib->required` є лише `false`, коли `LC_LOAD_WEAK_DYLIB` є true.
* **Знайдіть слабкі зв'язані бібліотеки** у бінарних файлах (пізніше ви побачите приклад створення бібліотек для перехоплення):
* ```bash
  ```

otool -l \</path/to/bin> | grep LC\_LOAD\_WEAK\_DYLIB -A 5 cmd LC\_LOAD\_WEAK\_DYLIB cmdsize 56 name /var/tmp/lib/libUtl.1.dylib (offset 24) time stamp 2 Wed Jun 21 12:23:31 1969 current version 1.0.0 compatibility version 1.0.0

````
* **Налаштовано за допомогою @rpath**: Бінарні файли Mach-O можуть містити команди **`LC_RPATH`** та **`LC_LOAD_DYLIB`**. На основі **значень** цих команд бібліотеки будуть **завантажені** з **різних каталогів**.
* **`LC_RPATH`** містить шляхи до деяких папок, які використовуються для завантаження бібліотек бінарним файлом.
* **`LC_LOAD_DYLIB`** містить шлях до конкретних бібліотек для завантаження. Ці шляхи можуть містити **`@rpath`**, який буде **замінений** значеннями в **`LC_RPATH`**. Якщо в **`LC_RPATH`** є кілька шляхів, кожен з них буде використовуватися для пошуку бібліотеки для завантаження. Приклад:
* Якщо **`LC_LOAD_DYLIB`** містить `@rpath/library.dylib` і **`LC_RPATH`** містить `/application/app.app/Contents/Framework/v1/` та `/application/app.app/Contents/Framework/v2/`. Обидва каталоги будуть використовуватися для завантаження `library.dylib`**.** Якщо бібліотека не існує в `[...]/v1/` і зловмисник може помістити її туди, щоб перехопити завантаження бібліотеки в `[...]/v2/`, оскільки дотримується порядок шляхів у **`LC_LOAD_DYLIB`**.
* **Знайдіть шляхи rpath та бібліотеки** в бінарних файлах за допомогою: `otool -l </path/to/binary> | grep -E "LC_RPATH|LC_LOAD_DYLIB" -A 5`

<div data-gb-custom-block data-tag="hint" data-style='info'>

**`@executable_path`**: Це **шлях** до каталогу, що містить **виконуваний файл**.

**`@loader_path`**: Це **шлях** до **каталогу**, що містить **Mach-O binary**, який містить команду завантаження.

* Коли використовується в виконуваному файлі, **`@loader_path`** фактично є **таким самим**, як **`@executable_path`**.
* Коли використовується в **dylib**, **`@loader_path`** вказує **шлях** до **dylib**.

</div>

Спосіб **підвищення привілеїв** за допомогою цієї функціональності полягає в тому, що в рідкісному випадку **застосунок**, який виконується **від** **root**, шукає деяку **бібліотеку в деякому каталозі, де зловмисник має права на запис.**

<div data-gb-custom-block data-tag="hint" data-style='success'>

Чудовий **сканер** для пошуку **відсутніх бібліотек** в застосунках - це [**Dylib Hijack Scanner**](https://objective-see.com/products/dhs.html) або [**CLI версія**](https://github.com/pandazheng/DylibHijack).\
Чудовий **звіт з технічними деталями** про цю техніку можна знайти [**тут**](https://www.virusbulletin.com/virusbulletin/2015/03/dylib-hijacking-os-x).

</div>

**Приклад**

<div data-gb-custom-block data-tag="content-ref" data-url='../../macos-dyld-hijacking-and-dyld_insert_libraries.md'>

[macos-dyld-hijacking-and-dyld\_insert\_libraries.md](../../macos-dyld-hijacking-and-dyld\_insert\_libraries.md)

</div>

## Підвищення привілеїв Dlopen

<div data-gb-custom-block data-tag="hint" data-style='danger'>

Пам'ятайте, що **попередні обмеження перевірки бібліотек також застосовуються** для виконання атак Dlopen hijacking.

</div>

З **`man dlopen`**:

* Коли шлях **не містить символа косої риски** (тобто це лише ім'я листка), **dlopen() буде робити пошук**. Якщо **`$DYLD_LIBRARY_PATH`** був встановлений при запуску, dyld спочатку **подивиться в тому каталозі**. Далі, якщо викликаючий файл mach-o або виконуваний файл вказують **`LC_RPATH`**, то dyld буде **дивитися в цих** каталогах. Далі, якщо процес **необмежений**, dyld буде шукати в **поточному робочому каталозі**. Нарешті, для старих бінарників dyld спробує деякі резервні варіанти. Якщо **`$DYLD_FALLBACK_LIBRARY_PATH`** був встановлений при запуску, dyld буде шукати в **цих каталогах**, інакше dyld подивиться в **`/usr/local/lib/`** (якщо процес необмежений), а потім в **`/usr/lib/`** (цю інформацію було взято з **`man dlopen`**).
1. `$DYLD_LIBRARY_PATH`
2. `LC_RPATH`
3. `CWD`(якщо необмежений)
4. `$DYLD_FALLBACK_LIBRARY_PATH`
5. `/usr/local/lib/` (якщо необмежений)
6. `/usr/lib/`

<div data-gb-custom-block data-tag="hint" data-style='danger'>

Якщо в імені немає косої риски, є 2 способи здійснити перехоплення:

* Якщо будь-який **`LC_RPATH`** є **записуваним** (але перевіряється підпис, тому для цього також потрібно, щоб бінарний файл був необмеженим)
* Якщо бінарний файл **необмежений**, тоді можливо завантажити щось з CWD (або зловживання однією з зазначених змінних середовища)

</div>

* Коли шлях **виглядає як шлях до фреймворку** (наприклад, `/stuff/foo.framework/foo`), якщо **`$DYLD_FRAMEWORK_PATH`** був встановлений при запуску, dyld спочатку подивиться в цьому каталозі для **часткового шляху фреймворку** (наприклад, `foo.framework/foo`). Далі dyld спробує **поданого шляху як є** (використовуючи поточний робочий каталог для відносних шляхів). Нарешті, для старих бінарників dyld спробує деякі резервні варіанти. Якщо **`$DYLD_FALLBACK_FRAMEWORK_PATH`** був встановлений при запуску, dyld буде шукати в цих каталогах. В іншому випадку він буде шукати в **`/Library/Frameworks`** (на macOS, якщо процес необмежений), а потім в **`/System/Library/Frameworks`**.
1. `$DYLD_FRAMEWORK_PATH`
2. поданий шлях (використовуючи поточний робочий каталог для відносних шляхів, якщо необмежений)
3. `$DYLD_FALLBACK_FRAMEWORK_PATH`
4. `/Library/Frameworks` (якщо необмежений)
5. `/System/Library/Frameworks`

<div data-gb-custom-block data-tag="hint" data-style='danger'>

Якщо шлях до фреймворку, спосіб перехоплення буде:

* Якщо процес **необмежений**, зловживання **відносним шляхом від CWD** згаданими змінними середовища (навіть якщо в документації не сказано, що якщо процес обмежений, змінні середовища DYLD\_\* видаляються)

</div>

* Коли шлях **містить косу риску, але не є шляхом до фреймворку** (тобто повний шлях або частковий шлях до dylib), dlopen() спочатку подивиться (якщо встановлено) в **`$DYLD_LIBRARY_PATH`** (з листковою частиною від шляху). Далі dyld **спробує поданий шлях** (використовуючи поточний робочий каталог для відносних шляхів (але лише для необмежених процесів)). Нарешті, для старих бінарників dyld спробує резервні варіанти. Якщо **`$DYLD_FALLBACK_LIBRARY_PATH`** був встановлений при запуску, dyld буде шукати в цих каталогах, інакше dyld подивиться в **`/usr/local/lib/`** (якщо процес необмежений), а потім в **`/usr/lib/`**.
1. `$DYLD_LIBRARY_PATH`
2. поданий шлях (використовуючи поточний робочий каталог для відносних шляхів, якщо необмежений)
3. `$DYLD_FALLBACK_LIBRARY_PATH`
4. `/usr/local/lib/` (якщо необмежений)
5. `/usr/lib/`

<div data-gb-custom-block data-tag="hint" data-style='danger'>

Якщо в імені є косі риски і це не шлях до фреймворку, спосіб перехоплення буде:

* Якщо бінарний файл **необмежений**, тоді можливо завантажити щось з CWD або `/usr/local/lib` (або зловживання однією з зазначених змінних середовища)

</div>

<div data-gb-custom-block data-tag="hint" data-style='info'>

Примітка: Немає **файлів конфігурації для керування пошуком dlopen**.

Примітка: Якщо виконуваний файл є **set\[ug\]id binary або підписаний з entitlements**, тоді **всі змінні середовища ігноруються**, і можна використовувати лише повний шлях ([перевірте обмеження DYLD\_INSERT\_LIBRARIES](../../macos-dyld-hijacking-and-dyld\_insert\_libraries.md#check-dyld\_insert\_librery-restrictions) для більш детальної інформації)

Примітка: На платформах Apple використовуються "універсальні" файли для поєднання бібліотек 32-біт і 64-біт. Це означає, що **окремих шляхів пошуку для 32-біт та 64-біт бібліотек немає**.

Примітка: На платформах Apple більшість OS dylibs **об'єднані в кеш dyld** і не існують на диску. Тому виклик **`stat()`** для попереднього перегляду того, чи існує OS dylib, **не працюватиме**. Однак **`dlopen_preflight()`** використовує ті самі кроки, що й **`dlopen()`**, для пошуку сумісного файлу mach-o.

</div>

**Перевірте шляхи**

Давайте перевіримо всі варіанти за допомогою наступного коду:
```c
// gcc dlopentest.c -o dlopentest -Wl,-rpath,/tmp/test
#include <dlfcn.h>
#include <stdio.h>

int main(void)
{
void* handle;

fprintf("--- No slash ---\n");
handle = dlopen("just_name_dlopentest.dylib",1);
if (!handle) {
fprintf(stderr, "Error loading: %s\n\n\n", dlerror());
}

fprintf("--- Relative framework ---\n");
handle = dlopen("a/framework/rel_framework_dlopentest.dylib",1);
if (!handle) {
fprintf(stderr, "Error loading: %s\n\n\n", dlerror());
}

fprintf("--- Abs framework ---\n");
handle = dlopen("/a/abs/framework/abs_framework_dlopentest.dylib",1);
if (!handle) {
fprintf(stderr, "Error loading: %s\n\n\n", dlerror());
}

fprintf("--- Relative Path ---\n");
handle = dlopen("a/folder/rel_folder_dlopentest.dylib",1);
if (!handle) {
fprintf(stderr, "Error loading: %s\n\n\n", dlerror());
}

fprintf("--- Abs Path ---\n");
handle = dlopen("/a/abs/folder/abs_folder_dlopentest.dylib",1);
if (!handle) {
fprintf(stderr, "Error loading: %s\n\n\n", dlerror());
}

return 0;
}
````

Якщо ви скомпілюєте та виконаєте це, ви можете побачити **де безуспішно шукалася кожна бібліотека**. Крім того, ви можете **фільтрувати журнали файлової системи**:

```bash
sudo fs_usage | grep "dlopentest"
```

## Використання відносних шляхів для викрадення

Якщо **привілейований бінарний файл/додаток** (наприклад, SUID або деякий бінарний файл з потужними entitlements) **завантажує бібліотеку за відносним шляхом** (наприклад, використовуючи `@executable_path` або `@loader_path`) і має **вимкнену перевірку бібліотек**, можливо перемістити бінарний файл до місця, де атакуючий може **змінити завантажену бібліотеку за відносним шляхом**, і використати це для впровадження коду в процес.

## Обрізання змінних середовища `DYLD_*` та `LD_LIBRARY_PATH`

У файлі `dyld-dyld-832.7.1/src/dyld2.cpp` можна знайти функцію **`pruneEnvironmentVariables`**, яка видалить будь-яку змінну середовища, яка **починається з `DYLD_`** та **`LD_LIBRARY_PATH=`**.

Також буде встановлено на **null** конкретно змінні середовища **`DYLD_FALLBACK_FRAMEWORK_PATH`** та **`DYLD_FALLBACK_LIBRARY_PATH`** для бінарних файлів з правами **suid** та **sgid**.

Ця функція викликається з функції **`_main`** того ж файлу, якщо цільова платформа - OSX, таким чином:

```cpp
#if TARGET_OS_OSX
if ( !gLinkContext.allowEnvVarsPrint && !gLinkContext.allowEnvVarsPath && !gLinkContext.allowEnvVarsSharedCache ) {
pruneEnvironmentVariables(envp, &apple);
```

і ці булеві прапорці встановлені в тому ж файлі у коді:

```cpp
#if TARGET_OS_OSX
// support chrooting from old kernel
bool isRestricted = false;
bool libraryValidation = false;
// any processes with setuid or setgid bit set or with __RESTRICT segment is restricted
if ( issetugid() || hasRestrictedSegment(mainExecutableMH) ) {
isRestricted = true;
}
bool usingSIP = (csr_check(CSR_ALLOW_TASK_FOR_PID) != 0);
uint32_t flags;
if ( csops(0, CS_OPS_STATUS, &flags, sizeof(flags)) != -1 ) {
// On OS X CS_RESTRICT means the program was signed with entitlements
if ( ((flags & CS_RESTRICT) == CS_RESTRICT) && usingSIP ) {
isRestricted = true;
}
// Library Validation loosens searching but requires everything to be code signed
if ( flags & CS_REQUIRE_LV ) {
isRestricted = false;
libraryValidation = true;
}
}
gLinkContext.allowAtPaths                = !isRestricted;
gLinkContext.allowEnvVarsPrint           = !isRestricted;
gLinkContext.allowEnvVarsPath            = !isRestricted;
gLinkContext.allowEnvVarsSharedCache     = !libraryValidation || !usingSIP;
gLinkContext.allowClassicFallbackPaths   = !isRestricted;
gLinkContext.allowInsertFailures         = false;
gLinkContext.allowInterposing         	 = true;
```

Що в основному означає, що якщо бінарний файл має **suid** або **sgid**, або має сегмент **RESTRICT** у заголовках або був підписаний з прапорцем **CS\_RESTRICT**, тоді **`!gLinkContext.allowEnvVarsPrint && !gLinkContext.allowEnvVarsPath && !gLinkContext.allowEnvVarsSharedCache`** є істинним, і змінні середовища обрізаються.

Зверніть увагу, що якщо **CS\_REQUIRE\_LV** є істинним, тоді змінні не будуть обрізані, але перевірка валідації бібліотеки перевірить, що вони використовують той самий сертифікат, що й оригінальний бінарний файл.

## Перевірка обмежень

### SUID та SGID

```bash
# Make it owned by root and suid
sudo chown root hello
sudo chmod +s hello
# Insert the library
DYLD_INSERT_LIBRARIES=inject.dylib ./hello

# Remove suid
sudo chmod -s hello
```

### Розділ `__RESTRICT` з сегментом `__restrict`

```bash
gcc -sectcreate __RESTRICT __restrict /dev/null hello.c -o hello-restrict
DYLD_INSERT_LIBRARIES=inject.dylib ./hello-restrict
```

### Захищений режим виконання

Створіть новий сертифікат у Keychain та використовуйте його для підпису бінарного файлу:

{% code overflow="wrap" %}
```bash
# Apply runtime proetction
codesign -s <cert-name> --option=runtime ./hello
DYLD_INSERT_LIBRARIES=inject.dylib ./hello #Library won't be injected

# Apply library validation
codesign -f -s <cert-name> --option=library ./hello
DYLD_INSERT_LIBRARIES=inject.dylib ./hello-signed #Will throw an error because signature of binary and library aren't signed by same cert (signs must be from a valid Apple-signed developer certificate)

# Sign it
## If the signature is from an unverified developer the injection will still work
## If it's from a verified developer, it won't
codesign -f -s <cert-name> inject.dylib
DYLD_INSERT_LIBRARIES=inject.dylib ./hello-signed

# Apply CS_RESTRICT protection
codesign -f -s <cert-name> --option=restrict hello-signed
DYLD_INSERT_LIBRARIES=inject.dylib ./hello-signed # Won't work
```
{% endcode %}

{% hint style="danger" %}
Зверніть увагу, що навіть якщо є бінарні файли, підписані флагами **`0x0(none)`**, вони можуть динамічно отримати прапор **`CS_RESTRICT`** при виконанні, тому ця техніка в них не працюватиме.

Ви можете перевірити, чи має цей прапор процес за допомогою (отримати [**csops тут**](https://github.com/axelexic/CSOps)):

```bash
csops -status <pid>
```

і потім перевірте, чи увімкнено прапорець 0x800.
{% endhint %}

## References

* [https://theevilbit.github.io/posts/dyld\_insert\_libraries\_dylib\_injection\_in\_macos\_osx\_deep\_dive/](https://theevilbit.github.io/posts/dyld\_insert\_libraries\_dylib\_injection\_in\_macos\_osx\_deep\_dive/)

<details>

<summary><strong>Вивчайте хакінг AWS від нуля до героя з</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Інші способи підтримки HackTricks:

* Якщо ви хочете побачити **рекламу вашої компанії в HackTricks** або **завантажити HackTricks у форматі PDF**, перевірте [**ПЛАНИ ПІДПИСКИ**](https://github.com/sponsors/carlospolop)!
* Отримайте [**офіційний мерч PEASS & HackTricks**](https://peass.creator-spring.com)
* Відкрийте для себе [**Сім'ю PEASS**](https://opensea.io/collection/the-peass-family), нашу колекцію ексклюзивних [**NFT**](https://opensea.io/collection/the-peass-family)
* **Приєднуйтесь до** 💬 [**групи Discord**](https://discord.gg/hRep4RUj7f) або [**групи Telegram**](https://t.me/peass) або **слідкуйте** за нами на **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Поділіться своїми хакерськими трюками, надсилайте PR до** [**HackTricks**](https://github.com/carlospolop/hacktricks) та [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) репозиторіїв.

</details>
