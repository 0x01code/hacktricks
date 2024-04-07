# macOS Kütüphane Enjeksiyonu

<details>

<summary><strong>Sıfırdan kahraman olacak şekilde AWS hacklemeyi öğrenin</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Kırmızı Takım Uzmanı)</strong></a><strong>!</strong></summary>

HackTricks'ı desteklemenin diğer yolları:

* **Şirketinizi HackTricks'te reklamını görmek istiyorsanız** veya **HackTricks'i PDF olarak indirmek istiyorsanız** [**ABONELİK PLANLARI**](https://github.com/sponsors/carlospolop)'na göz atın!
* [**Resmi PEASS & HackTricks ürünlerini**](https://peass.creator-spring.com) edinin
* [**PEASS Ailesi'ni**](https://opensea.io/collection/the-peass-family) keşfedin, özel [**NFT'lerimiz**](https://opensea.io/collection/the-peass-family) koleksiyonumuz
* **Katılın** 💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) veya bizi **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**'da takip edin.**
* **Hacking püf noktalarınızı göndererek HackTricks ve HackTricks Cloud** github depolarına PR göndererek paylaşın.

</details>

{% hint style="danger" %}
**dyld'ın kodu açık kaynaklıdır** ve [https://opensource.apple.com/source/dyld/](https://opensource.apple.com/source/dyld/) adresinde bulunabilir ve **dyld-852.2.tar.gz gibi bir URL kullanarak** bir **tar** indirilebilir.
{% endhint %}

## **DYLD\_INSERT\_LIBRARIES**

Bu, [**Linux'taki LD\_PRELOAD'a benzer**](../../../../linux-hardening/privilege-escalation/#ld\_preload). Bir sürecin belirli bir kütüphaneyi yüklemek için çalıştırılacağını belirtmesine izin verir (env var etkinse)

Bu teknik aynı zamanda her yüklenen uygulamanın bir "Info.plist" adlı bir plist dosyasına sahip olduğu ve `LSEnvironmental` adlı bir anahtar kullanarak **çevresel değişkenlerin atanmasına izin veren** bir **ASEP tekniği olarak da kullanılabilir**.

{% hint style="info" %}
2012'den beri **Apple, `DYLD_INSERT_LIBRARIES`'in gücünü büyük ölçüde azaltmıştır**.

Koda gidin ve **`src/dyld.cpp`'yi kontrol edin**. **`pruneEnvironmentVariables`** işlevinde **`DYLD_*`** değişkenlerinin kaldırıldığını görebilirsiniz.

**`processRestricted`** işlevinde kısıtlamanın nedeni belirlenir. Bu kodu kontrol ettiğinizde nedenlerin şunlar olduğunu görebilirsiniz:

* İkili dosya `setuid/setgid`
* Macho ikili dosyasında `__RESTRICT/__restrict` bölümünün varlığı.
* Yazılımın [`com.apple.security.cs.allow-dyld-environment-variables`](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_allow-dyld-environment-variables) ayrıcalığı olmadan ayrıcalıkları var
* Bir ikilinin ayrıcalıklarını şu şekilde kontrol edin: `codesign -dv --entitlements :- </path/to/bin>`

Daha güncel sürümlerde bu mantığı **`configureProcessRestrictions`** işlevinin ikinci kısmında bulabilirsiniz. Ancak, daha yeni sürümlerde yürütülen şey, **fonksiyonun başlangıç kontrolleridir** (iOS veya simülasyonla ilgili olanları kaldırabilirsiniz çünkü bunlar macOS'ta kullanılmayacaktır.
{% endhint %}

### Kütüphane Doğrulaması

İkili dosya **`DYLD_INSERT_LIBRARIES`** ortam değişkenini kullanmaya izin verirse bile, ikili dosya kütüphanenin imzasını kontrol ederse özel bir kütüphane yüklemeyecektir.

Özel bir kütüphaneyi yüklemek için ikili dosyanın aşağıdaki ayrıcalıklardan birine sahip olması gerekir:

* [`com.apple.security.cs.disable-library-validation`](../../macos-security-protections/macos-dangerous-entitlements.md#com.apple.security.cs.disable-library-validation)
* [`com.apple.private.security.clear-library-validation`](../../macos-security-protections/macos-dangerous-entitlements.md#com.apple.private.security.clear-library-validation)

veya ikili dosyanın **sertleştirilmiş çalışma zamanı bayrağı** veya **kütüphane doğrulama bayrağı** olmamalıdır.

Bir ikilinin **sertleştirilmiş çalışma zamanı** olup olmadığını `codesign --display --verbose <bin>` ile kontrol edebilirsiniz ve **`CodeDirectory`** içindeki bayrak çalışma zamanını kontrol edebilirsiniz: **`CodeDirectory v=20500 size=767 flags=0x10000(runtime) hashes=13+7 location=embedded`**

Ayrıca, bir kütüphanenin yüklenip yüklenmediğini **aynı sertifikayla imzalanmışsa** kontrol edebilirsiniz.

Bunu (kötüye kullanma) nasıl yapacağınızı ve kısıtlamaları kontrol edin:

{% content-ref url="macos-dyld-hijacking-and-dyld_insert_libraries.md" %}
[macos-dyld-hijacking-and-dyld\_insert\_libraries.md](macos-dyld-hijacking-and-dyld\_insert\_libraries.md)
{% endcontent-ref %}

## Dylib Kaçırma

{% hint style="danger" %}
**Önceki Kütüphane Doğrulama kısıtlamalarının** Dylib kaçırma saldırılarını gerçekleştirmek için de geçerli olduğunu unutmayın.
{% endhint %}

Windows'ta olduğu gibi, MacOS'ta da **dylib'leri kaçırabilir** ve **uygulamaları** **keyfi kod** **çalıştırmaya zorlayabilirsiniz** (aslında bir düzenli kullanıcıdan bu mümkün olmayabilir çünkü bir `.app` paketi içine yazmak ve bir kütüphaneyi kaçırmak için bir TCC iznine ihtiyacınız olabilir).\
Ancak, **MacOS** uygulamalarının kütüphaneleri yükleme şekli **Windows'tan daha kısıtlıdır**. Bu, **kötü amaçlı yazılım** geliştiricilerinin bu tekniği **gizlilik** için kullanabileceği ancak **bu yöntemi ayrıcalıkları yükseltmek için kötüye kullanma olasılığının çok daha düşük olduğu anlamına gelir**.

Öncelikle, **MacOS ikili dosyalarının kütüphaneleri yüklemek için tam yol belirttiğini** daha sık bulmanız daha olasıdır. İkinci olarak, **MacOS'un asla** kütüphaneleri **$PATH** klasörlerinde aramadığını unutmayın.

Bu işlevselliğe ilişkin **ana** kod parçası `ImageLoader.cpp` içindeki **`ImageLoader::recursiveLoadLibraries`** işlevindedir.

Bir macho ikili dosyanın yüklemek için kullanabileceği **4 farklı başlık Komutu** vardır:

* **`LC_LOAD_DYLIB`** komutu bir dylib yüklemek için yaygın bir komuttur.
* **`LC_LOAD_WEAK_DYLIB`** komutu öncekiyle aynı şekilde çalışır, ancak dylib bulunamazsa, herhangi bir hata olmadan devam edilir.
* **`LC_REEXPORT_DYLIB`** komutu sembolleri farklı bir kütüphaneden proxy (veya yeniden ihraç) eder.
* **`LC_LOAD_UPWARD_DYLIB`** komutu birbirlerine bağımlı iki kütüphane olduğunda kullanılır (buna _yukarı bağımlılık_ denir).

Ancak, **2 tür dylib kaçırma** vardır:

* **Zayıf bağlantılı kütüphanelerin eksik olması**: Bu, uygulamanın **LC\_LOAD\_WEAK\_DYLIB** ile yapılandırılmış olmayan bir kütüphaneyi yüklemeye çalışacağı anlamına gelir. Sonra, **saldırgan bir dylib'i beklenen yere yerleştirirse yüklenecektir**.
* Bağlantının "zayıf" olduğu gerçeği, uygulamanın kütüphanenin bulunamaması durumunda çalışmaya devam edeceği anlamına gelir.
* Bu işle ilgili **kod** `ImageLoaderMachO.cpp` dosyasındaki `ImageLoaderMachO::doGetDependentLibraries` işlevindedir, burada `lib->required` yalnızca `LC_LOAD_WEAK_DYLIB` doğru olduğunda `false` olur.
* **Zayıf bağlantılı kütüphaneleri** aşağıdaki gibi ikililerde bulun: (kütüphane kaçırma kütüphaneleri oluşturma örneğine daha sonra bakacaksınız):
* ```bash
otool -l </path/to/bin> | grep LC_LOAD_WEAK_DYLIB -A 5 cmd LC_LOAD_WEAK_DYLIB
cmdsize 56
name /var/tmp/lib/libUtl.1.dylib (offset 24)
time stamp 2 Wed Jun 21 12:23:31 1969
current version 1.0.0
compatibility version 1.0.0
```
* **@rpath ile yapılandırılmış**: Mach-O ikili dosyaları **LC_RPATH** ve **LC_LOAD_DYLIB** komutlarına sahip olabilir. Bu komutların **değerlerine** bağlı olarak, kütüphaneler **farklı dizinlerden** yüklenecektir.
* **`LC_RPATH`** ikilinin kütüphaneleri yüklemek için kullandığı bazı klasörlerin yolunu içerir.
* **`LC_LOAD_DYLIB`** belirli kütüphaneleri yüklemek için yol içerir. Bu yollar **`@rpath`** içerebilir, bu değerler **`LC_RPATH`** içindeki değerlerle **değiştirilecektir**. Eğer **`LC_RPATH`** içinde birden fazla yol varsa, her biri yüklenen kütüphaneyi aramak için kullanılacaktır. Örnek:
* Eğer **`LC_LOAD_DYLIB`** `@rpath/library.dylib` içeriyorsa ve **`LC_RPATH`** `/application/app.app/Contents/Framework/v1/` ve `/application/app.app/Contents/Framework/v2/` içeriyorsa. Her iki klasör de `library.dylib`'i yüklemek için kullanılacaktır. Eğer kütüphane `[...]/v1/` içinde bulunmuyorsa ve saldırgan onu oraya yerleştirebilirse, kütüphanenin yüklenmesini `[...]/v2/` içindeki kütüphanenin yüklenmesini ele geçirmek için **`LC_LOAD_DYLIB`** içindeki yol sırası takip edilir.
* **Rpath yollarını ve kütüphaneleri** bulmak için: `otool -l </path/to/binary> | grep -E "LC_RPATH|LC_LOAD_DYLIB" -A 5`

{% hint style="info" %}
**`@executable_path`**: Ana yürütülebilir dosyanın bulunduğu dizinin **yolu**dur.

**`@loader_path`**: Yük komutunu içeren **Mach-O ikili**'yi içeren dizinin **yoludur**.

* Bir yürütülebilir dosyada kullanıldığında, **`@loader_path`** etkili bir şekilde **`@executable_path`** ile aynıdır.
* Bir **dylib**'de kullanıldığında, **`@loader_path`** dylib'in yolunu verir.
{% endhint %}

Bu işlevselliği **istismar ederek ayrıcalıkları yükseltmenin** yolu, nadir durumlarda **kök** tarafından yürütülen bir **uygulamanın**, saldırganın yazma izinlerine sahip olduğu bir klasördeki bir **kütüphaneyi aradığı** durumdur.

{% hint style="success" %}
Uygulamalardaki **eksik kütüphaneleri bulmak** için güzel bir **tarama aracı** [**Dylib Hijack Scanner**](https://objective-see.com/products/dhs.html) veya bir [**CLI sürümü**](https://github.com/pandazheng/DylibHijack) bulunabilir.\
Bu teknik hakkında teknik detayları içeren güzel bir **rapor** [**burada**](https://www.virusbulletin.com/virusbulletin/2015/03/dylib-hijacking-os-x) bulunabilir.
{% endhint %}

**Örnek**

{% content-ref url="macos-dyld-hijacking-and-dyld_insert_libraries.md" %}
[macos-dyld-hijacking-and-dyld\_insert\_libraries.md](macos-dyld-hijacking-and-dyld\_insert\_libraries.md)
{% endcontent-ref %}

## Dlopen Hijacking

{% hint style="danger" %}
**Önceki Kütüphane Doğrulama** kısıtlamalarının **Dlopen** saldırılarını gerçekleştirmek için de geçerli olduğunu unutmayın.
{% endhint %}

**`man dlopen`**'dan:

* Yol **eğik çizgi karakteri içermiyorsa** (yani sadece bir yaprak adı ise), **dlopen() arama yapacaktır**. Eğer başlangıçta **`$DYLD_LIBRARY_PATH`** ayarlandıysa, dyld önce **o dizine bakacaktır**. Sonra, çağıran mach-o dosyası veya ana yürütülebilir dosya bir **`LC_RPATH`** belirtiyorsa, dyld **bu dizinlere bakacaktır**. Sonra, işlem kısıtlanmamışsa, dyld **mevcut çalışma dizininde arayacaktır**. Son olarak, eski ikililer için dyld bazı yedekler deneyecektir. Eğer başlangıçta **`$DYLD_FALLBACK_LIBRARY_PATH`** ayarlandıysa, dyld **bu dizinlerde arayacaktır**, aksi takdirde dyld **`/usr/local/lib/`**'de (işlem kısıtlanmamışsa) ve ardından **`/usr/lib/`**'de arayacaktır. (Bu bilgi **`man dlopen`**'dan alınmıştır).
1. `$DYLD_LIBRARY_PATH`
2. `LC_RPATH`
3. `CWD` (kısıtlanmamışsa)
4. `$DYLD_FALLBACK_LIBRARY_PATH`
5. `/usr/local/lib/` (kısıtlanmamışsa)
6. `/usr/lib/`

{% hint style="danger" %}
İsimde eğik çizgi yoksa, bir hijacking yapmanın 2 yolu olacaktır:

* Eğer herhangi bir **`LC_RPATH`** **yazılabilirse** (ancak imza kontrol edilir, bu nedenle bunun için ikilinin kısıtlanmamış olması gerekir)
* Eğer ikili **kısıtlanmamışsa** ve ardından CWD'den bir şey yüklemek mümkün olabilir (veya belirtilen çevre değişkenlerinden birini kötüye kullanmak)
{% endhint %}

* Yol **bir çerçeve yolu gibi görünüyorsa** (örneğin `/stuff/foo.framework/foo`), başlangıçta **`$DYLD_FRAMEWORK_PATH`** ayarlandıysa, dyld önce **bu dizine bakacaktır** çerçeve kısmi yol için (örneğin `foo.framework/foo`). Sonra, dyld **verilen yolu deneyecektir** (ilişkisel yollar için mevcut çalışma dizinini kullanarak). Son olarak, eski ikililer için dyld bazı yedekler deneyecektir. Eğer başlangıçta **`$DYLD_FALLBACK_FRAMEWORK_PATH`** ayarlandıysa, dyld **bu dizinleri arayacaktır**. Aksi takdirde, dyld **`/Library/Frameworks`**'de (macOS'ta işlem kısıtlanmamışsa), ardından **`/System/Library/Frameworks`**'de arayacaktır.
1. `$DYLD_FRAMEWORK_PATH`
2. verilen yol (ilişkisel yollar için mevcut çalışma dizinini kullanarak kısıtlanmamışsa)
3. `$DYLD_FALLBACK_FRAMEWORK_PATH`
4. `/Library/Frameworks` (kısıtlanmamışsa)
5. `/System/Library/Frameworks`

{% hint style="danger" %}
Bir çerçeve yolu ise, onu ele geçirmenin yolu şöyle olacaktır:

* Eğer işlem **kısıtlanmamışsa**, CWD'den **ilişkisel yol** kötüye kullanılabilir ve belirtilmese de, belgelerde işlemin kısıtlı olup olmadığı belirtilmediğinden DYLD\_\* çevre değişkenlerinin kısıtlı olup olmadığı belirtilmemiştir.
{% endhint %}

* Yol **eğik çizgi içeriyorsa ancak bir çerçeve yolu değilse** (yani bir dylib için tam yol veya kısmi yol), dlopen() önce (ayarlandıysa) **`$DYLD_LIBRARY_PATH`**'de (yolun yaprak kısmı ile) bakacaktır. Sonra, dyld **verilen yolu deneyecektir** (ilişkisel yollar için mevcut çalışma dizinini kullanarak (ancak sadece kısıtlanmamış işlemler için)). Son olarak, eski ikililer için dyld bazı yedekler deneyecektir. Eğer başlangıçta **`$DYLD_FALLBACK_LIBRARY_PATH`** ayarlandıysa, dyld **bu dizinlerde arayacaktır**, aksi takdirde dyld **`/usr/local/lib/`**'de (işlem kısıtlanmamışsa) ve ardından **`/usr/lib/`**'de arayacaktır.
1. `$DYLD_LIBRARY_PATH`
2. verilen yol (ilişkisel yollar için mevcut çalışma dizinini kullanarak kısıtlanmamışsa)
3. `$DYLD_FALLBACK_LIBRARY_PATH`
4. `/usr/local/lib/` (kısıtlanmamışsa)
5. `/usr/lib/`

{% hint style="danger" %}
İsimde eğik çizgiler varsa ve bir çerçeve değilse, onu ele geçirmenin yolu şöyle olacaktır:

* Eğer ikili **kısıtlanmamışsa** ve ardından CWD'den veya `/usr/local/lib`'den bir şey yüklemek mümkün olabilir (veya belirtilmese de, belgelerde işlemin kısıtlı olup olmadığı belirtilmediğinden birini kötüye kullanmak)
{% endhint %}

{% hint style="info" %}
Not: **Dlopen aramasını kontrol etmek** için **yapılandırma dosyaları yoktur**.

Not: Ana yürütülebilir dosya **set\[ug]id ikili veya ayrıcalıklarla kod imzalanmışsa**, o zaman **tüm çevre değişkenleri yok sayılır** ve yalnızca tam yol kullanılabilir (daha ayrıntılı bilgi için [DYLD\_INSERT\_LIBRARIES kısıtlamalarını kontrol edin](macos-dyld-hijacking-and-dyld\_insert\_libraries.md#check-dyld\_insert\_librery-restrictions)).

Not: Apple platformları, 32 bit ve 64 bit kütüphaneleri birleştirmek için "evrensel" dosyalar kullanır. Bu, **ayrı 32 bit ve 64 bit arama yollarının olmadığı anlamına gelir**.

Not: Apple platformlarında çoğu OS dylib'leri **dyld önbelleğine** birleştirilir ve diskte mevcut değildir. Bu nedenle, bir OS dylib'in var olup olmadığını ön izlemek için **`stat()`** çağırmak **çalışmaz**. Bununla birlikte, **`dlopen()`** uyumlu bir mach-o dosyası bulmak için **`dlopen_preflight()`** aynı adımları kullanır.
{% endhint %}

**Yolları Kontrol Et**

Aşağıdaki kod ile tüm seçenekleri kontrol edelim:
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
```
Eğer derlersen ve çalıştırırsan, **her kütüphane nerede başarısız bir şekilde arandığını görebilirsin**. Ayrıca, **FS loglarını filtreleyebilirsin**:
```bash
sudo fs_usage | grep "dlopentest"
```
## Göreceli Yol Kaçırma

Eğer bir **ayrıcalıklı ikili/uygulama** (örneğin SUID veya güçlü yetkilere sahip bazı ikili dosyalar) **bir göreceli yol kütüphanesini yüklüyorsa** (örneğin `@executable_path` veya `@loader_path` kullanarak) ve **Kütüphane Doğrulaması devre dışı bırakılmışsa**, saldırganın ikili dosyayı, yüklenen göreceli yol kütüphanesini değiştirebileceği ve sürece kod enjekte etmek için kötüye kullanabileceği bir konuma taşıması mümkün olabilir.

## `DYLD_*` ve `LD_LIBRARY_PATH` çevresel değişkenlerini Temizle

`dyld-dyld-832.7.1/src/dyld2.cpp` dosyasında **`pruneEnvironmentVariables`** işlevini bulmak mümkündür, bu işlev **`DYLD_` ile başlayan** ve **`LD_LIBRARY_PATH=`** olan herhangi bir çevresel değişkeni kaldıracaktır.

Ayrıca, **suid** ve **sgid** ikili dosyalar için özellikle **`DYLD_FALLBACK_FRAMEWORK_PATH`** ve **`DYLD_FALLBACK_LIBRARY_PATH`** çevresel değişkenlerini **null** olarak ayarlayacaktır.

Bu işlev, OSX hedefleniyorsa aynı dosyanın **`_main`** işlevinden şu şekilde çağrılır:
```cpp
#if TARGET_OS_OSX
if ( !gLinkContext.allowEnvVarsPrint && !gLinkContext.allowEnvVarsPath && !gLinkContext.allowEnvVarsSharedCache ) {
pruneEnvironmentVariables(envp, &apple);
```
ve bu boolean bayrakları kod içinde aynı dosyada ayarlanır:
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
Bu temelde, eğer ikili dosya **suid** veya **sgid** ise, başlıkta bir **RESTRICT** segmenti bulunuyorsa veya **CS\_RESTRICT** bayrağı ile imzalanmışsa, o zaman **`!gLinkContext.allowEnvVarsPrint && !gLinkContext.allowEnvVarsPath && !gLinkContext.allowEnvVarsSharedCache`** doğru olacak ve çevre değişkenleri budanacaktır.

CS\_REQUIRE\_LV doğruysa, değişkenler budanmayacak ancak kütüphane doğrulaması, bunların orijinal ikili dosya ile aynı sertifikayı kullandığını kontrol edecektir.

## Kısıtlamaları Kontrol Et

### SUID & SGID
```bash
# Make it owned by root and suid
sudo chown root hello
sudo chmod +s hello
# Insert the library
DYLD_INSERT_LIBRARIES=inject.dylib ./hello

# Remove suid
sudo chmod -s hello
```
### Bölüm `__RESTRICT` ile segment `__restrict`
```bash
gcc -sectcreate __RESTRICT __restrict /dev/null hello.c -o hello-restrict
DYLD_INSERT_LIBRARIES=inject.dylib ./hello-restrict
```
### Güçlendirilmiş çalışma zamanı

Anahtarlıkta yeni bir sertifika oluşturun ve bunu ikili dosyayı imzalamak için kullanın:

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
Not edin ki, bayraklarla imzalanmış ikili dosyalar olsa bile **`0x0(none)`** bayrağıyla, yürütüldüğünde **`CS_RESTRICT`** bayrağını dinamik olarak alabilir ve bu nedenle bu teknik onlarda çalışmayacaktır.

Bu bayrağın bir proc'a sahip olup olmadığını kontrol edebilirsiniz (buradan [**csops burada**](https://github.com/axelexic/CSOps)):
```bash
csops -status <pid>
```
ve ardından bayrağın 0x800 etkin olup olmadığını kontrol edin.
{% endhint %}

## Referanslar

* [https://theevilbit.github.io/posts/dyld\_insert\_libraries\_dylib\_injection\_in\_macos\_osx\_deep\_dive/](https://theevilbit.github.io/posts/dyld\_insert\_libraries\_dylib\_injection\_in\_macos\_osx\_deep\_dive/)

<details>

<summary><strong>Sıfırdan kahraman olmaya kadar AWS hackleme öğrenin</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks'i desteklemenin diğer yolları:

* **Şirketinizi HackTricks'te reklamını görmek istiyorsanız** veya **HackTricks'i PDF olarak indirmek istiyorsanız** [**ABONELİK PLANLARI**](https://github.com/sponsors/carlospolop)'na göz atın!
* [**Resmi PEASS & HackTricks ürünlerini keşfedin**](https://peass.creator-spring.com)
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) koleksiyonumuzu keşfedin, özel [**NFT'lerimiz**](https://opensea.io/collection/the-peass-family) arasında
* **💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) katılın veya bizi **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)** takip edin.**
* **Hacking püf noktalarınızı paylaşarak PR'ler göndererek** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github depolarına katkıda bulunun.

</details>
