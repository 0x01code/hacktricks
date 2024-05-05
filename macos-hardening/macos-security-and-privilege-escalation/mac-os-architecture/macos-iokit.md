# macOS IOKit

<details>

<summary><strong>AWS hacklemeyi sıfırdan ileri seviyeye öğrenin</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> ile!</strong></summary>

* **Bir **cybersecurity şirketinde mi çalışıyorsunuz? **Şirketinizi HackTricks'te** görmek ister misiniz? Ya da **PEASS'ın en son sürümüne erişmek veya HackTricks'i PDF olarak indirmek** ister misiniz? [**ABONELİK PLANLARINI**](https://github.com/sponsors/carlospolop) kontrol edin!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) koleksiyonumuzu keşfedin, özel [**NFT'ler**](https://opensea.io/collection/the-peass-family)
* [**Resmi PEASS ve HackTricks ürünlerini alın**](https://peass.creator-spring.com)
* **Discord** [**💬**](https://emojipedia.org/speech-balloon/) **grubuna katılın** veya [**telegram grubuna**](https://t.me/peass) veya **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live) **takip edin**.
* **Hacking püf noktalarınızı paylaşın, PR göndererek** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **ve** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **ile.**

</details>

## Temel Bilgiler

I/O Kit, XNU çekirdeğindeki açık kaynaklı, nesne yönelimli **aygıt sürücü çerçevesidir**, **dinamik olarak yüklenen aygıt sürücülerini** işler. Çeşitli donanımı destekleyen çekirdeğe modüler kodun anında eklenmesine izin verir.

IOKit sürücüleri temelde **çekirdekten fonksiyonlar ihraç eder**. Bu fonksiyon parametre **türleri önceden tanımlanmıştır** ve doğrulanır. Dahası, XPC gibi, IOKit sadece **Mach mesajlarının üzerinde başka bir katmandır**.

**IOKit XNU çekirdek kodu**, Apple tarafından [https://github.com/apple-oss-distributions/xnu/tree/main/iokit](https://github.com/apple-oss-distributions/xnu/tree/main/iokit) adresinde açık kaynak olarak yayınlanmıştır. Ayrıca, kullanıcı alanı IOKit bileşenleri de açık kaynaktır [https://github.com/opensource-apple/IOKitUser](https://github.com/opensource-apple/IOKitUser).

Ancak, **hiçbir IOKit sürücüsü** açık kaynak değildir. Neyse ki, zaman zaman bir sürücünün sürümü, onu hata ayıklamayı kolaylaştıran sembollerle gelebilir. [**Firmware'den sürücü uzantılarını nasıl alacağınızı buradan öğrenin**](./#ipsw)**.**

**C++** dilinde yazılmıştır. Demangled C++ sembollerini alabilirsiniz:
```bash
# Get demangled symbols
nm -C com.apple.driver.AppleJPEGDriver

# Demangled symbols from stdin
c++filt
__ZN16IOUserClient202222dispatchExternalMethodEjP31IOExternalMethodArgumentsOpaquePK28IOExternalMethodDispatch2022mP8OSObjectPv
IOUserClient2022::dispatchExternalMethod(unsigned int, IOExternalMethodArgumentsOpaque*, IOExternalMethodDispatch2022 const*, unsigned long, OSObject*, void*)
```
{% hint style="danger" %}
IOKit **açık fonksiyonları**, bir istemcinin bir işlevi çağırmaya çalıştığında **ek güvenlik kontrolleri** gerçekleştirebilir ancak uygulamalar genellikle **IOKit fonksiyonlarıyla etkileşime girebilecekleri** **kum havuzu** tarafından **sınırlanır**.
{% endhint %}

## Sürücüler

macOS'ta şurada bulunurlar:

* **`/System/Library/Extensions`**
* OS X işletim sistemi içine yerleştirilmiş KEXT dosyaları.
* **`/Library/Extensions`**
* 3. taraf yazılım tarafından yüklenen KEXT dosyaları

iOS'ta şurada bulunurlar:

* **`/System/Library/Extensions`**
```bash
#Use kextstat to print the loaded drivers
kextstat
Executing: /usr/bin/kmutil showloaded
No variant specified, falling back to release
Index Refs Address            Size       Wired      Name (Version) UUID <Linked Against>
1  142 0                  0          0          com.apple.kpi.bsd (20.5.0) 52A1E876-863E-38E3-AC80-09BBAB13B752 <>
2   11 0                  0          0          com.apple.kpi.dsep (20.5.0) 52A1E876-863E-38E3-AC80-09BBAB13B752 <>
3  170 0                  0          0          com.apple.kpi.iokit (20.5.0) 52A1E876-863E-38E3-AC80-09BBAB13B752 <>
4    0 0                  0          0          com.apple.kpi.kasan (20.5.0) 52A1E876-863E-38E3-AC80-09BBAB13B752 <>
5  175 0                  0          0          com.apple.kpi.libkern (20.5.0) 52A1E876-863E-38E3-AC80-09BBAB13B752 <>
6  154 0                  0          0          com.apple.kpi.mach (20.5.0) 52A1E876-863E-38E3-AC80-09BBAB13B752 <>
7   88 0                  0          0          com.apple.kpi.private (20.5.0) 52A1E876-863E-38E3-AC80-09BBAB13B752 <>
8  106 0                  0          0          com.apple.kpi.unsupported (20.5.0) 52A1E876-863E-38E3-AC80-09BBAB13B752 <>
9    2 0xffffff8003317000 0xe000     0xe000     com.apple.kec.Libm (1) 6C1342CC-1D74-3D0F-BC43-97D5AD38200A <5>
10   12 0xffffff8003544000 0x92000    0x92000    com.apple.kec.corecrypto (11.1) F5F1255F-6552-3CF4-A9DB-D60EFDEB4A9A <8 7 6 5 3 1>
```
Sayı 9'a kadar listelenen sürücüler **0 adresinde yüklenir**. Bu, bunların gerçek sürücüler olmadığı anlamına gelir ve **çekilemezler**.

Belirli uzantıları bulmak için şunu kullanabilirsiniz:
```bash
kextfind -bundle-id com.apple.iokit.IOReportFamily #Search by full bundle-id
kextfind -bundle-id -substring IOR #Search by substring in bundle-id
```
Kernel uzantılarını yüklemek ve kaldırmak için şunları yapın:
```bash
kextload com.apple.iokit.IOReportFamily
kextunload com.apple.iokit.IOReportFamily
```
## IORegistry

**IORegistry**, macOS ve iOS'taki IOKit çerçevesinin kritik bir parçasıdır ve sistem donanım konfigürasyonunu ve durumunu temsil etmek için bir veritabanı olarak hizmet verir. Tüm donanım ve sürücüleri temsil eden nesnelerin hiyerarşik bir koleksiyonudur ve birbirleriyle olan ilişkilerini gösterir.

IORegistry'yi **`ioreg`** komutunu kullanarak konsoldan inceleyebilirsiniz (özellikle iOS için kullanışlıdır).
```bash
ioreg -l #List all
ioreg -w 0 #Not cut lines
ioreg -p <plane> #Check other plane
```
**`IORegistryExplorer`**'ı [**https://developer.apple.com/download/all/**](https://developer.apple.com/download/all/) adresinden **Xcode Ek Araçları**'ndan indirebilir ve **grafiksel** arayüz aracılığıyla **macOS IORegistry**'yi inceleyebilirsiniz.

<figure><img src="../../../.gitbook/assets/image (1167).png" alt="" width="563"><figcaption></figcaption></figure>

IORegistryExplorer'da "planes" (düzlemler), IORegistry'deki farklı nesneler arasındaki ilişkileri düzenlemek ve göstermek için kullanılır. Her düzlem, sistem donanımının ve sürücü yapılandırmasının belirli bir görünümünü veya belirli bir ilişki türünü temsil eder. İşte IORegistryExplorer'da karşılaşabileceğiniz bazı yaygın düzlemler:

1. **IOService Düzlemi**: Bu en genel düzlemdir, sürücüleri ve nub'ları (sürücüler arasındaki iletişim kanalları) temsil eden hizmet nesnelerini gösterir. Bu nesneler arasındaki sağlayıcı-müşteri ilişkilerini gösterir.
2. **IODeviceTree Düzlemi**: Bu düzlem, cihazların sistemdeki bağlantılarını temsil eder. USB veya PCI gibi otobüsler aracılığıyla bağlanan cihazların hiyerarşisini görselleştirmek için sıkça kullanılır.
3. **IOPower Düzlemi**: Nesneleri ve ilişkilerini güç yönetimi açısından gösterir. Diğer nesnelerin güç durumunu etkileyen nesneleri gösterebilir, güçle ilgili sorunları gidermek için faydalıdır.
4. **IOUSB Düzlemi**: Özellikle USB cihazlarına ve ilişkilerine odaklanır, USB hub'larının ve bağlı cihazların hiyerarşisini gösterir.
5. **IOAudio Düzlemi**: Bu düzlem, ses cihazlarını ve sistem içindeki ilişkilerini temsil etmek içindir.
6. ...

## Sürücü İletişim Kod Örneği

Aşağıdaki kod, `"YourServiceNameHere"` adlı IOKit hizmetine bağlanır ve seçici 0 içindeki işlevi çağırır. Bunun için:

* İlk olarak **`IOServiceMatching`** ve **`IOServiceGetMatchingServices`**'i çağırarak hizmeti alır.
* Ardından **`IOServiceOpen`** çağırarak bir bağlantı kurar.
* Ve son olarak **`IOConnectCallScalarMethod`** ile seçici 0'ı (seçici, çağırmak istediğiniz işlevin atandığı numaradır) belirterek bir işlevi çağırır.
```objectivec
#import <Foundation/Foundation.h>
#import <IOKit/IOKitLib.h>

int main(int argc, const char * argv[]) {
@autoreleasepool {
// Get a reference to the service using its name
CFMutableDictionaryRef matchingDict = IOServiceMatching("YourServiceNameHere");
if (matchingDict == NULL) {
NSLog(@"Failed to create matching dictionary");
return -1;
}

// Obtain an iterator over all matching services
io_iterator_t iter;
kern_return_t kr = IOServiceGetMatchingServices(kIOMasterPortDefault, matchingDict, &iter);
if (kr != KERN_SUCCESS) {
NSLog(@"Failed to get matching services");
return -1;
}

// Get a reference to the first service (assuming it exists)
io_service_t service = IOIteratorNext(iter);
if (!service) {
NSLog(@"No matching service found");
IOObjectRelease(iter);
return -1;
}

// Open a connection to the service
io_connect_t connect;
kr = IOServiceOpen(service, mach_task_self(), 0, &connect);
if (kr != KERN_SUCCESS) {
NSLog(@"Failed to open service");
IOObjectRelease(service);
IOObjectRelease(iter);
return -1;
}

// Call a method on the service
// Assume the method has a selector of 0, and takes no arguments
kr = IOConnectCallScalarMethod(connect, 0, NULL, 0, NULL, NULL);
if (kr != KERN_SUCCESS) {
NSLog(@"Failed to call method");
}

// Cleanup
IOServiceClose(connect);
IOObjectRelease(service);
IOObjectRelease(iter);
}
return 0;
}
```
**IOConnectCallScalarMethod** gibi IOKit işlevlerini çağırmak için kullanılabilecek **IOConnectCallMethod**, **IOConnectCallStructMethod** gibi **diğer** işlevler bulunmaktadır.

## Sürücü giriş noktasını tersine çevirme

Bunları örneğin bir [**firmware görüntüsünden (ipsw)**](./#ipsw) elde edebilirsiniz. Daha sonra, favori dekompilerinize yükleyin.

Doğru işlevi çağıran çağrıyı alan ve doğru işlevi çağıran sürücü işlevi olan **externalMethod** işlevini dekompilasyona başlayabilirsiniz:

<figure><img src="../../../.gitbook/assets/image (1168).png" alt="" width="315"><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (1169).png" alt=""><figcaption></figcaption></figure>

Bu korkunç çağrı, şunu ifade eder:

{% code overflow="wrap" %}
```cpp
IOUserClient2022::dispatchExternalMethod(unsigned int, IOExternalMethodArgumentsOpaque*, IOExternalMethodDispatch2022 const*, unsigned long, OSObject*, void*)
```
{% endcode %}

Önceki tanımda **`self`** parametresinin eksik olduğuna dikkat edin, doğru tanım şu şekilde olmalıdır:

{% code overflow="wrap" %}
```cpp
IOUserClient2022::dispatchExternalMethod(self, unsigned int, IOExternalMethodArgumentsOpaque*, IOExternalMethodDispatch2022 const*, unsigned long, OSObject*, void*)
```
{% endcode %}

Aslında, gerçek tanımı [https://github.com/apple-oss-distributions/xnu/blob/1031c584a5e37aff177559b9f69dbd3c8c3fd30a/iokit/Kernel/IOUserClient.cpp#L6388](https://github.com/apple-oss-distributions/xnu/blob/1031c584a5e37aff177559b9f69dbd3c8c3fd30a/iokit/Kernel/IOUserClient.cpp#L6388) adresinde bulabilirsiniz:
```cpp
IOUserClient2022::dispatchExternalMethod(uint32_t selector, IOExternalMethodArgumentsOpaque *arguments,
const IOExternalMethodDispatch2022 dispatchArray[], size_t dispatchArrayCount,
OSObject * target, void * reference)
```
Bu bilgiyle Ctrl+Right -> `Düzenle fonksiyon imzası` yeniden yazılabilir ve bilinen tipler ayarlanabilir:

<figure><img src="../../../.gitbook/assets/image (1174).png" alt=""><figcaption></figcaption></figure>

Yeni decompile edilmiş kod şu şekilde görünecek:

<figure><img src="../../../.gitbook/assets/image (1175).png" alt=""><figcaption></figcaption></figure>

Bir sonraki adım için **`IOExternalMethodDispatch2022`** yapısının tanımlanmış olması gerekiyor. [https://github.com/apple-oss-distributions/xnu/blob/1031c584a5e37aff177559b9f69dbd3c8c3fd30a/iokit/IOKit/IOUserClient.h#L168-L176](https://github.com/apple-oss-distributions/xnu/blob/1031c584a5e37aff177559b9f69dbd3c8c3fd30a/iokit/IOKit/IOUserClient.h#L168-L176) adresinde açık kaynaklıdır, şu şekilde tanımlayabilirsiniz:

<figure><img src="../../../.gitbook/assets/image (1170).png" alt=""><figcaption></figcaption></figure>

Şimdi, `(IOExternalMethodDispatch2022 *)&sIOExternalMethodArray` takip ederek birçok veri görebilirsiniz:

<figure><img src="../../../.gitbook/assets/image (1176).png" alt="" width="563"><figcaption></figcaption></figure>

Veri Türünü **`IOExternalMethodDispatch2022:`** olarak değiştirin:

<figure><img src="../../../.gitbook/assets/image (1177).png" alt="" width="375"><figcaption></figcaption></figure>

değişiklikten sonra:

<figure><img src="../../../.gitbook/assets/image (1179).png" alt="" width="563"><figcaption></figcaption></figure>

Ve şimdi, içinde **7 elemanın bir dizisi** olduğunu biliyoruz (son decompile edilmiş kodu kontrol edin), 7 elemanlık bir dizi oluşturmak için tıklayın:

<figure><img src="../../../.gitbook/assets/image (1180).png" alt="" width="563"><figcaption></figcaption></figure>

Dizi oluşturulduktan sonra tüm ihraç edilen fonksiyonları görebilirsiniz:

<figure><img src="../../../.gitbook/assets/image (1181).png" alt=""><figcaption></figcaption></figure>

{% hint style="success" %}
Hatırlarsanız, kullanıcı alanından bir **ihraç edilen** fonksiyonu **çağırmak** için fonksiyonun adını değil, **seçici numarasını** çağırmamız gerekir. Burada, seçici **0**'ın **`initializeDecoder`** fonksiyonu, seçici **1**'in **`startDecoder`** fonksiyonu, seçici **2**'nin **`initializeEncoder`** fonksiyonu olduğunu görebilirsiniz...
{% endhint %}
