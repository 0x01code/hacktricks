# macOS Uygulamaları - İnceleme, hata ayıklama ve Fuzzing

<details>

<summary><strong>AWS hackleme konusunda sıfırdan kahramana dönüşmek için</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Kırmızı Takım Uzmanı)</strong></a><strong>'ı öğrenin!</strong></summary>

HackTricks'i desteklemenin diğer yolları:

* Şirketinizi HackTricks'te **reklamını görmek** veya HackTricks'i **PDF olarak indirmek** için [**ABONELİK PLANLARINI**](https://github.com/sponsors/carlospolop) kontrol edin!
* [**Resmi PEASS & HackTricks ürünlerini**](https://peass.creator-spring.com) edinin
* [**The PEASS Ailesi'ni**](https://opensea.io/collection/the-peass-family) keşfedin, özel [**NFT'lerimiz**](https://opensea.io/collection/the-peass-family) koleksiyonumuz
* 💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) **katılın** veya **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)'u **takip edin**.
* **Hacking hilelerinizi** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github depolarına **PR göndererek paylaşın**.

</details>

## Statik Analiz

### otool
```bash
otool -L /bin/ls #List dynamically linked libraries
otool -tv /bin/ps #Decompile application
```
### objdump

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

### jtool2

Bu araç, **codesign**, **otool** ve **objdump** için bir **yerine geçme** olarak kullanılabilir ve birkaç ek özellik sunar. [**Buradan indirebilirsiniz**](http://www.newosxbook.com/tools/jtool.html) veya `brew` ile kurabilirsiniz.
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
### Codesign / ldid

{% hint style="danger" %}
**`Codesign`**, macOS'ta bulunurken **`ldid`**, iOS'ta bulunur.
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

[**SuspiciousPackage**](https://mothersruin.com/software/SuspiciousPackage/get.html), kurmadan önce içeriğini görmek için **.pkg** dosyalarını (kurulum dosyaları) incelemek için kullanışlı bir araçtır.\
Bu kurulum dosyaları, kötü amaçlı yazılım yazarlarının genellikle kötü amaçlı yazılımı **sürdürmek** için kötüye kullandığı `preinstall` ve `postinstall` bash komut dosyalarına sahiptir.

### hdiutil

Bu araç, Apple disk görüntülerini (**.dmg**) çalıştırmadan önce incelemek için **mount** etmeyi sağlar:
```bash
hdiutil attach ~/Downloads/Firefox\ 58.0.2.dmg
```
### Objective-C

#### Metadata

{% hint style="danger" %}
Objective-C ile yazılan programlar, [Mach-O ikili dosyalarına](../macos-files-folders-and-binaries/universal-binaries-and-mach-o-format.md) derlendiklerinde sınıf bildirimlerini **saklar**. Bu sınıf bildirimleri aşağıdaki bilgileri içerir:
{% endhint %}

* Sınıfın adı
* Sınıfın metodları
* Sınıfın örnek değişkenleri

Bu bilgilere [**class-dump**](https://github.com/nygard/class-dump) kullanarak erişebilirsiniz:
```bash
class-dump Kindle.app
```
#### Fonksiyon çağırma

Objective-C kullanan bir ikili dosyada bir fonksiyon çağırıldığında, derlenmiş kod bu fonksiyonu çağırmak yerine **`objc_msgSend`**'i çağırır. Bu da son fonksiyonu çağırır:

![](<../../../.gitbook/assets/image (560).png>)

Bu fonksiyonun beklediği parametreler şunlardır:

* İlk parametre (**self**), "mesajı alan sınıfın örneğine işaret eden bir işaretçi"dir. Daha basit bir ifadeyle, yöntemin çağrıldığı nesnedir. Eğer yöntem bir sınıf yöntemi ise, bu bir sınıf nesnesinin bir örneği olacaktır, bir örnek yöntem için ise self, bir nesne olarak sınıfın bir örneğine işaret edecektir.
* İkinci parametre (**op**), "mesajı işleyen yöntemin seçicisi"dir. Yine daha basit bir ifadeyle, bu sadece yöntemin adıdır.
* Geri kalan parametreler, yöntem tarafından gereken herhangi bir değerdir (op).

| **Argüman**       | **Kayıt**                                                       | **(için) objc\_msgSend**                              |
| ----------------- | --------------------------------------------------------------- | ------------------------------------------------------ |
| **1. argüman**    | **rdi**                                                         | **self: yöntemin çağrıldığı nesne**                    |
| **2. argüman**    | **rsi**                                                         | **op: yöntemin adı**                                  |
| **3. argüman**    | **rdx**                                                         | **yönteme gönderilen 1. argüman**                      |
| **4. argüman**    | **rcx**                                                         | **yönteme gönderilen 2. argüman**                      |
| **5. argüman**    | **r8**                                                          | **yönteme gönderilen 3. argüman**                      |
| **6. argüman**    | **r9**                                                          | **yönteme gönderilen 4. argüman**                      |
| **7. ve sonrası** | <p><strong>rsp+</strong><br><strong>(yığında)</strong></p>     | **yönteme gönderilen 5. ve sonrası argümanlar**         |

### Swift

Swift ikili dosyalarıyla, Objective-C uyumluluğu olduğu için bazen [class-dump](https://github.com/nygard/class-dump/) kullanarak bildirimleri çıkarabilirsiniz, ancak her zaman mümkün olmayabilir.

**`jtool -l`** veya **`otool -l`** komut satırlarıyla **`__swift5`** önekiyle başlayan birkaç bölüm bulunabilir.
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
Bu bölümde depolanan bilgiler hakkında daha fazla bilgiye [bu blog yazısında](https://knight.sc/reverse%20engineering/2019/07/17/swift-metadata.html) ulaşabilirsiniz.

Ayrıca, **Swift ikili dosyalarında semboller olabilir** (örneğin, kütüphaneler sembolleri depolamalıdır, böylece işlevlerine çağrı yapılabilir). Semboller genellikle işlev adı ve öznitelik hakkında bilgi içerir ve karmaşık bir şekilde temsil edilir, bu yüzden oldukça faydalıdırlar ve orijinal adı alabilen "**demanglers**" bulunmaktadır.
```bash
# Ghidra plugin
https://github.com/ghidraninja/ghidra_scripts/blob/master/swift_demangler.py

# Swift cli
swift demangle
```
### Paketlenmiş ikili dosyalar

* Yüksek entropi kontrol edin
* Dizeleri kontrol edin (anlaşılabilir bir dize neredeyse yoksa, paketlenmiş)
* MacOS için UPX paketleyici, "\_\_XHDR" adında bir bölüm oluşturur

## Dinamik Analiz

{% hint style="warning" %}
Not olarak, ikili dosyaları hata ayıklamak için **SIP devre dışı bırakılmalıdır** (`csrutil disable` veya `csrutil enable --without debug`) veya ikili dosyaları geçici bir klasöre kopyalayıp `codesign --remove-signature <binary-path>` ile imzayı kaldırmak veya ikili dosyanın hata ayıklanmasına izin vermek (bu betiği kullanabilirsiniz: [bu betik](https://gist.github.com/carlospolop/a66b8d72bb8f43913c4b5ae45672578b))
{% endhint %}

{% hint style="warning" %}
Not olarak, macOS'ta **cloudconfigurationd** gibi sistem ikili dosyalarını **enstrümantal hale getirmek** için SIP devre dışı bırakılmalıdır (yalnızca imzayı kaldırmak işe yaramaz).
{% endhint %}

### Birleşik Günlükler

MacOS, bir uygulama çalıştırılırken ne yaptığını anlamaya çalışırken çok faydalı olabilecek birçok günlük oluşturur.

Ayrıca, bazı günlükler, bazı **kullanıcıya** veya **bilgisayara** **tanımlanabilir** bilgileri **gizlemek** için `<private>` etiketini içerecektir. Bununla birlikte, bu bilgileri açıklamak için bir sertifika **yüklenebilir**. [**Buradaki**](https://superuser.com/questions/1532031/how-to-show-private-data-in-macos-unified-log) açıklamaları takip edin.

### Hopper

#### Sol panel

Hopper'ın sol panelinde ikili dosyanın sembolleri (**Etiketler**), prosedürlerin ve fonksiyonların listesi (**Proc**) ve dizeler (**Str**) görülebilir. Bunlar tüm dizeler değildir, ancak Mac-O dosyasının çeşitli bölümlerinde tanımlananlar (_cstring veya_ `objc_methname` gibi).

#### Orta panel

Orta panelde **ayrıştırılmış kodu** görebilirsiniz. Ve bunu **ham** bir şekilde, **grafik** olarak, **decompile** olarak ve **ikili** olarak görebilirsiniz, ilgili simgeye tıklayarak:

<figure><img src="../../../.gitbook/assets/image (2) (6).png" alt=""><figcaption></figcaption></figure>

Kod nesnesine sağ tıklayarak, **o nesneye yapılan başvuruları** görebilir veya hatta adını değiştirebilirsiniz (bu, decompile edilmiş sahte kodda çalışmaz):

<figure><img src="../../../.gitbook/assets/image (1) (1) (2).png" alt=""><figcaption></figcaption></figure>

Ayrıca, **orta aşağıda python komutları yazabilirsiniz**.

#### Sağ panel

Sağ panelde, **gezinme geçmişi** (bu şekilde mevcut duruma nasıl geldiğinizi bilirsiniz), bu işlevi **çağıran tüm işlevleri** ve bu işlevin **çağırdığı tüm işlevleri** görebileceğiniz **çağrı grafiği** ve **yerel değişkenler** bilgisi gibi ilginç bilgileri görebilirsiniz.

### dtrace

Kullanıcılara uygulamalara son derece **düşük seviyede erişim** sağlar ve kullanıcılara programları **izlemelerine** ve hatta **yürütme akışlarını değiştirmelerine** olanak tanır. Dtrace, çekirdeğin her yerine yerleştirilen **probeleri** kullanır ve sistem çağrılarının başlangıcı ve sonu gibi konumlarda bulunur.

DTrace, her sistem çağrısı için bir prob oluşturmak için **`dtrace_probe_create`** işlevini kullanır. Bu probler, her sistem çağrısının giriş ve çıkış noktasında tetiklenebilir. DTrace ile etkileşim, yalnızca kök kullanıcısı için kullanılabilen /dev/dtrace üzerinden gerçekleşir.

{% hint style="success" %}
SIP korumasını tamamen devre dışı bırakmadan Dtrace'i etkinleştirmek için kurtarma modunda şunu çalıştırabilirsiniz: `csrutil enable --without dtrace`

Ayrıca, **derlediğiniz** **`dtrace`** veya **`dtruss`** ikili dosyalarını kullanabilirsiniz.
{% endhint %}

dtrace'in mevcut probeleri şu şekilde elde edilebilir:
```bash
dtrace -l | head
ID   PROVIDER            MODULE                          FUNCTION NAME
1     dtrace                                                     BEGIN
2     dtrace                                                     END
3     dtrace                                                     ERROR
43    profile                                                     profile-97
44    profile                                                     profile-199
```
Sondalama adı dört bölümden oluşur: sağlayıcı, modül, işlev ve ad (`fbt:mach_kernel:ptrace:entry`). Adın bazı bölümlerini belirtmezseniz, Dtrace bu bölümü joker karakter olarak uygular.

DTrace'i yapılandırmak ve tetiklendiklerinde ne tür eylemler gerçekleştireceğini belirtmek için D dilini kullanmamız gerekecek.

Daha detaylı bir açıklama ve daha fazla örnek [https://illumos.org/books/dtrace/chp-intro.html](https://illumos.org/books/dtrace/chp-intro.html) adresinde bulunabilir.

#### Örnekler

**Kullanılabilir DTrace komut dosyalarını** listelemek için `man -k dtrace` komutunu çalıştırın. Örnek: `sudo dtruss -n binary`

* Satırda
```bash
#Count the number of syscalls of each running process
sudo dtrace -n 'syscall:::entry {@[execname] = count()}'
```
# macOS Uygulamalarını İnceleme, Hata Ayıklama ve Fuzzing

Bu bölümde, macOS işletim sisteminde uygulamaları incelemek, hata ayıklamak ve fuzzing yapmak için bazı teknikler hakkında bilgi bulacaksınız.

## Uygulama İnceleme

Uygulamaları incelemek için çeşitli araçlar ve yöntemler vardır. İşte bazıları:

- **otool**: Bu araç, bir uygulamanın içindeki paylaşılan kütüphaneleri ve bağımlılıkları listeler.
- **strings**: Bu araç, bir uygulamanın içindeki metin dizelerini görüntüler.
- **class-dump**: Bu araç, bir uygulamanın içindeki sınıfları ve yöntemleri görüntüler.
- **Hopper Disassembler**: Bu ticari bir araçtır ve bir uygulamanın içindeki makine kodunu analiz etmek için kullanılır.

## Hata Ayıklama

Hata ayıklama, bir uygulamanın çalışma zamanında hataları bulmak ve düzeltmek için kullanılan bir tekniktir. macOS'ta hata ayıklama yapmak için aşağıdaki araçları kullanabilirsiniz:

- **lldb**: Bu, macOS'ta yerleşik bir hata ayıklama aracıdır. Bir uygulamayı lldb ile başlatarak, adım adım kodu izleyebilir, değişken değerlerini kontrol edebilir ve hataları tespit edebilirsiniz.
- **GDB**: Bu, GNU Hata Ayıklama Aracı'dır ve macOS'ta da kullanılabilir. lldb'ye benzer şekilde çalışır ve bir uygulamayı hata ayıklamak için kullanılabilir.

## Fuzzing

Fuzzing, bir uygulamanın girişine rastgele veya özel olarak oluşturulmuş verileri besleyerek hataları tespit etmek için kullanılan bir tekniktir. macOS'ta fuzzing yapmak için aşağıdaki araçları kullanabilirsiniz:

- **AFL**: Bu, American Fuzzy Lop'un kısaltmasıdır ve birçok platformda kullanılabilen bir fuzzing aracıdır. AFL, uygulamaları otomatik olarak test etmek için giriş verilerini değiştirir ve hataları tespit eder.
- **honggfuzz**: Bu, honggfuzz adlı bir fuzzing aracıdır ve macOS'ta da kullanılabilir. AFL gibi çalışır ve uygulamaları otomatik olarak test etmek için giriş verilerini değiştirir.

Bu araçlar ve teknikler, macOS uygulamalarını incelemek, hata ayıklamak ve fuzzing yapmak için başlangıç noktası olabilir. Daha fazla bilgi için ilgili araçların belgelerini ve kaynaklarını inceleyebilirsiniz.
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

`dtruss` is a command-line tool available on macOS that allows you to inspect and debug applications by tracing system calls and signals. It provides a way to monitor the interactions between an application and the operating system, which can be useful for understanding how an application works or identifying potential vulnerabilities.

To use `dtruss`, you simply need to run it followed by the command or application you want to trace. For example, to trace the `ls` command, you would run:

```
dtruss ls
```

This will display a list of system calls and signals that are being executed by the `ls` command. Each line of output includes the system call or signal name, the arguments passed to the call, and the return value.

By analyzing the output of `dtruss`, you can gain insights into how an application interacts with the operating system and identify any abnormal behavior or potential security issues. It can be particularly useful for identifying privilege escalation vulnerabilities, as it allows you to see if an application is making unauthorized system calls or accessing sensitive resources.

Keep in mind that `dtruss` requires root privileges to trace system calls from other processes. You can use the `sudo` command to run `dtruss` with elevated privileges:

```
sudo dtruss <command>
```

Overall, `dtruss` is a powerful tool for inspecting and debugging applications on macOS, providing valuable insights into their behavior and potential security vulnerabilities.
```bash
dtruss -c ls #Get syscalls of ls
dtruss -c -p 1000 #get syscalls of PID 1000
```
### ktrace

Bu, **SIP etkinleştirilmiş olsa bile** kullanabileceğiniz bir yöntemdir.
```bash
ktrace trace -s -S -t c -c ls | grep "ls("
```
### ProcessMonitor

[**ProcessMonitor**](https://objective-see.com/products/utilities.html#ProcessMonitor), bir işlemin gerçekleştirdiği işlemlerle ilgili bilgileri kontrol etmek için çok kullanışlı bir araçtır (örneğin, bir işlemin hangi yeni işlemleri oluşturduğunu izlemek).

### SpriteTree

[**SpriteTree**](https://themittenmac.com/tools/), işlemler arasındaki ilişkileri yazdıran bir araçtır.\
Mac'inizi **`sudo eslogger fork exec rename create > cap.json`** gibi bir komutla izlemeniz gerekmektedir (bu komutu çalıştıran terminalin FDA'ya ihtiyacı vardır). Ardından bu araca json dosyasını yükleyerek tüm ilişkileri görebilirsiniz:

<figure><img src="../../../.gitbook/assets/image (710).png" alt="" width="375"><figcaption></figcaption></figure>

### FileMonitor

[**FileMonitor**](https://objective-see.com/products/utilities.html#FileMonitor), dosya olaylarını (oluşturma, değiştirme ve silme gibi) izlemeyi sağlar ve bu olaylar hakkında detaylı bilgi sağlar.

### Crescendo

[**Crescendo**](https://github.com/SuprHackerSteve/Crescendo), Microsoft Sysinternal's _Procmon_ aracından Windows kullanıcılarının aşina olabileceği bir görünüm ve hisse sahip olan bir GUI aracıdır. Bu araç, çeşitli olay türlerinin kaydedilip durdurulmasına izin verir, bu olayları dosya, işlem, ağ vb. kategorilere göre filtrelemeye olanak sağlar ve kaydedilen olayları json formatında kaydetme işlevselliği sunar.

### Apple Instruments

[**Apple Instruments**](https://developer.apple.com/library/archive/documentation/Performance/Conceptual/CellularBestPractices/Appendix/Appendix.html), Xcode'un Geliştirici araçlarının bir parçasıdır - uygulama performansını izlemek, bellek sızıntılarını tanımlamak ve dosya sistemi etkinliğini takip etmek için kullanılır.

![](<../../../.gitbook/assets/image (15).png>)

### fs\_usage

İşlemler tarafından gerçekleştirilen işlemleri takip etmeyi sağlar:
```bash
fs_usage -w -f filesys ls #This tracks filesystem actions of proccess names containing ls
fs_usage -w -f network curl #This tracks network actions
```
### TaskExplorer

[**Taskexplorer**](https://objective-see.com/products/taskexplorer.html), bir ikili dosyanın kullandığı **kütüphaneleri**, kullandığı **dosyaları** ve **ağ** bağlantılarını görmek için kullanışlıdır.\
Ayrıca, ikili işlemleri **virustotal**'a karşı kontrol eder ve ikili hakkında bilgi gösterir.

## PT\_DENY\_ATTACH <a href="#page-title" id="page-title"></a>

[**Bu blog yazısında**](https://knight.sc/debugging/2019/06/03/debugging-apple-binaries-that-use-pt-deny-attach.html), SIP devre dışı bırakılmış olsa bile hata ayıklamayı önlemek için **`PT_DENY_ATTACH`** kullanan çalışan bir daemon'ı nasıl hata ayıklayacağınız hakkında bir örnek bulabilirsiniz.

### lldb

**lldb**, macOS ikili dosyalarını hata ayıklama için kullanılan de facto araçtır.
```bash
lldb ./malware.bin
lldb -p 1122
lldb -n malware.bin
lldb -n malware.bin --waitfor
```
`lldb` kullanırken intel lezzetini ayarlayabilirsiniz. Ev klasörünüzde **`.lldbinit`** adında bir dosya oluşturarak aşağıdaki satırı ekleyin:
```bash
settings set target.x86-disassembly-flavor intel
```
{% hint style="warning" %}
lldb içinde `process save-core` komutunu kullanarak bir işlemi dökün.
{% endhint %}

<table data-header-hidden><thead><tr><th width="225"></th><th></th></tr></thead><tbody><tr><td><strong>(lldb) Komut</strong></td><td><strong>Açıklama</strong></td></tr><tr><td><strong>run (r)</strong></td><td>Çalıştırmayı başlatır ve işlem bir kesme noktasına ulaşana veya işlem sona erene kadar kesintisiz devam eder.</td></tr><tr><td><strong>continue (c)</strong></td><td>Hata ayıklanan işlemin yürütmesini devam ettirir.</td></tr><tr><td><strong>nexti (n / ni)</strong></td><td>Sonraki talimatı yürütür. Bu komut, işlev çağrılarını atlar.</td></tr><tr><td><strong>stepi (s / si)</strong></td><td>Sonraki talimatı yürütür. nexti komutunun aksine, bu komut işlev çağrılarına adım atar.</td></tr><tr><td><strong>finish (f)</strong></td><td>Geçerli işlevin geri kalan talimatlarını ("çerçeve") yürütür ve durur.</td></tr><tr><td><strong>control + c</strong></td><td>Yürütme duraklatılır. Eğer işlem çalıştırıldıysa (r) veya devam ettirildiyse (c), bu işlemi mevcut olarak yürütülen yerde durdurur.</td></tr><tr><td><strong>breakpoint (b)</strong></td><td><p>b main #Herhangi bir main fonksiyonu çağrılır</p><p>b &#x3C;binname>`main #Belirtilen binin ana fonksiyonu</p><p>b set -n main --shlib &#x3C;lib_name> #Belirtilen binin ana fonksiyonu</p><p>b -[NSDictionary objectForKey:]</p><p>b -a 0x0000000100004bd9</p><p>br l #Kesme noktası listesi</p><p>br e/dis &#x3C;num> #Kesme noktasını etkinleştir/devre dışı bırak</p><p>breakpoint delete &#x3C;num></p></td></tr><tr><td><strong>help</strong></td><td><p>help breakpoint #Kesme noktası komutunun yardımını al</p><p>help memory write #Belleğe yazmak için yardım al</p></td></tr><tr><td><strong>reg</strong></td><td><p>reg read</p><p>reg read $rax</p><p>reg read $rax --format &#x3C;<a href="https://lldb.llvm.org/use/variable.html#type-format">format</a>></p><p>reg write $rip 0x100035cc0</p></td></tr><tr><td><strong>x/s &#x3C;reg/memory address></strong></td><td>Belleği null-ile sonlanmış bir dize olarak görüntüler.</td></tr><tr><td><strong>x/i &#x3C;reg/memory address></strong></td><td>Belleği derleme talimatı olarak görüntüler.</td></tr><tr><td><strong>x/b &#x3C;reg/memory address></strong></td><td>Belleği bayt olarak görüntüler.</td></tr><tr><td><strong>print object (po)</strong></td><td><p>Bu, parametre tarafından referans alınan nesneyi yazdırır</p><p>po $raw</p><p><code>{</code></p><p><code>dnsChanger = {</code></p><p><code>"affiliate" = "";</code></p><p><code>"blacklist_dns" = ();</code></p><p>Apple'ın Objective-C API'lerinin çoğu veya yöntemleri nesneler döndürür ve bu nedenle "print object" (po) komutuyla görüntülenmelidir. Eğer po anlamlı bir çıktı üretmezse <code>x/b</code> kullanın</p></td></tr><tr><td><strong>memory</strong></td><td>memory read 0x000....<br>memory read $x0+0xf2a<br>memory write 0x100600000 -s 4 0x41414141 #Bu adrese AAAA yazın<br>memory write -f s $rip+0x11f+7 "AAAA" #Bu adrese AAAA yazın</td></tr><tr><td><strong>disassembly</strong></td><td><p>dis #Geçerli işlevi derler</p><p>dis -n &#x3C;funcname> #Fonksiyonu derler</p><p>dis -n &#x3C;funcname> -b &#x3C;basename> #Fonksiyonu derler<br>dis -c 6 #6 satırı derler<br>dis -c 0x100003764 -e 0x100003768 #Bir adresten diğerine kadar derler<br>dis -p -c 4 #Mevcut adreste derlemeye başlar</p></td></tr><tr><td><strong>parray</strong></td><td>parray 3 (char **)$x1 #x1 reg içindeki 3 bileşenli diziyi kontrol edin</td></tr></tbody></table>

{% hint style="info" %}
**`objc_sendMsg`** fonksiyonu çağrıldığında, **rsi** kaydedici, bir null-ile sonlanmış ("C") dize olarak **yöntem adını** tutar. lldb ile adı yazdırmak için:

`(lldb) x/s $rsi: 0x1000f1576: "startMiningWithPort:password:coreCount:slowMemory:currency:"`

`(lldb) print (char*)$rsi:`\
`(char *) $1 = 0x00000001000f1576 "startMiningWithPort:password:coreCount:slowMemory:currency:"`

`(lldb) reg read $rsi: rsi = 0x00000001000f1576 "startMiningWithPort:password:coreCount:slowMemory:currency:"`
{% endhint %}

### Anti-Dinamik Analiz

#### VM tespiti

* **`sysctl hw.model`** komutu, **ana bilgisayar MacOS ise** "Mac" döndürür, ancak bir VM ise farklı bir değer döndürür.
* Bazı kötü amaçlı yazılımlar, **`hw.logicalcpu`** ve **`hw.physicalcpu`** değerlerini oynayarak bir VM olup olmadığını tespit etmeye çalışır.
* Bazı kötü amaçlı yazılımlar, MAC adresine (00:50:56) dayanarak **makinenin VMware olup olmadığını** da tespit edebilir.
* Bir işlemin hata ayıklanıp ayıklanmadığını basit bir kodla tespit etmek de mümkündür:
* `if(P_TRACED == (info.kp_proc.p_flag & P_TRACED)){ //hata ayıklanan işlem }`
* Ayrıca, **`ptrace`** sistem çağrısını **`PT_DENY_ATTACH`** bayrağıyla çağırabilirsiniz. Bu, bir hata ayıklayıcının eklenmesini ve izlenmesini engeller.
* **`sysctl`** veya **`ptrace`** işlevinin **ithal edilip edilmediğini** kontrol edebilirsiniz (ancak kötü amaçlı yazılım bunu dinamik olarak da içe aktarabilir)
* Bu yazıda belirtildiği gibi, "[Defeating Anti-Debug Techniques: macOS ptrace variants](https://alexomara.com/blog/defeating-anti-debug-techniques-macos-ptrace-variants/)" :\
"_Process # exited with **status = 45 (0x0000002d)**_ genellikle hata ayıklama hedefinin **PT\_DENY\_ATTACH** kullandığının bir işareti olabilir"
## Fuzzing

### [ReportCrash](https://ss64.com/osx/reportcrash.html)

ReportCrash, **çöken işlemleri analiz eder ve bir çökme raporunu diske kaydeder**. Bir çökme raporu, bir çökmenin nedenini teşhis etmeye yardımcı olabilecek bilgiler içerir.\
Kullanıcı başına launchd bağlamında çalışan uygulamalar ve diğer işlemler için, ReportCrash, bir LaunchAgent olarak çalışır ve çökme raporlarını kullanıcının `~/Library/Logs/DiagnosticReports/` dizinine kaydeder.\
Daemonlar, sistem launchd bağlamında çalışan diğer işlemler ve diğer ayrıcalıklı işlemler için, ReportCrash bir LaunchDaemon olarak çalışır ve çökme raporlarını sistemdeki `/Library/Logs/DiagnosticReports` dizinine kaydeder.

Eğer çökme raporlarının **Apple'a gönderilmesinden endişe duyuyorsanız**, bunları devre dışı bırakabilirsiniz. Aksi takdirde, çökme raporları bir sunucunun nasıl çöktüğünü **anlamak için faydalı olabilir**.
```bash
#To disable crash reporting:
launchctl unload -w /System/Library/LaunchAgents/com.apple.ReportCrash.plist
sudo launchctl unload -w /System/Library/LaunchDaemons/com.apple.ReportCrash.Root.plist

#To re-enable crash reporting:
launchctl load -w /System/Library/LaunchAgents/com.apple.ReportCrash.plist
sudo launchctl load -w /System/Library/LaunchDaemons/com.apple.ReportCrash.Root.plist
```
### Uyku

MacOS'ta fuzzing yaparken Mac'in uyumasına izin vermemek önemlidir:

* systemsetup -setsleep Never
* pmset, Sistem Tercihleri
* [KeepingYouAwake](https://github.com/newmarcel/KeepingYouAwake)

#### SSH Bağlantısı Kesilmesi

SSH bağlantısı üzerinden fuzzing yapıyorsanız, oturumun sona ermemesi için aşağıdaki adımları izleyin:

* TCPKeepAlive Yes
* ClientAliveInterval 0
* ClientAliveCountMax 0
```bash
sudo launchctl unload /System/Library/LaunchDaemons/ssh.plist
sudo launchctl load -w /System/Library/LaunchDaemons/ssh.plist
```
### Dahili İşleyiciler

**Aşağıdaki sayfaya** göz atarak, **belirli bir şema veya protokolü işleme sorumluluğuna sahip olan uygulamayı** nasıl bulabileceğinizi öğrenebilirsiniz:

{% content-ref url="../macos-file-extension-apps.md" %}
[macos-file-extension-apps.md](../macos-file-extension-apps.md)
{% endcontent-ref %}

### Ağ İşlemlerini Sıralama

Bu, ağ verilerini yöneten işlemleri bulmak için ilginç bir yöntemdir:
```bash
dtrace -n 'syscall::recv*:entry { printf("-> %s (pid=%d)", execname, pid); }' >> recv.log
#wait some time
sort -u recv.log > procs.txt
cat procs.txt
```
Veya `netstat` veya `lsof` kullanın

### Libgmalloc

<figure><img src="../../../.gitbook/assets/Pasted Graphic 14.png" alt=""><figcaption></figcaption></figure>

{% code overflow="wrap" %}
```bash
lldb -o "target create `which some-binary`" -o "settings set target.env-vars DYLD_INSERT_LIBRARIES=/usr/lib/libgmalloc.dylib" -o "run arg1 arg2" -o "bt" -o "reg read" -o "dis -s \$pc-32 -c 24 -m -F intel" -o "quit"
```
{% endcode %}

### Fuzzerlar

#### [AFL++](https://github.com/AFLplusplus/AFLplusplus)

CLI araçları için çalışır.

#### [Litefuzz](https://github.com/sec-tools/litefuzz)

MacOS GUI araçlarıyla "**sadece çalışır"**. Bazı macOS uygulamalarının benzersiz dosya adları, doğru uzantılar, dosyaların kum havuzundan (`~/Library/Containers/com.apple.Safari/Data`) okunması gibi belirli gereksinimleri vardır.

Bazı örnekler:

{% code overflow="wrap" %}
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

### Daha Fazla Fuzzing MacOS Bilgisi

* [https://www.youtube.com/watch?v=T5xfL9tEg44](https://www.youtube.com/watch?v=T5xfL9tEg44)
* [https://github.com/bnagy/slides/blob/master/OSXScale.pdf](https://github.com/bnagy/slides/blob/master/OSXScale.pdf)
* [https://github.com/bnagy/francis/tree/master/exploitaben](https://github.com/bnagy/francis/tree/master/exploitaben)
* [https://github.com/ant4g0nist/crashwrangler](https://github.com/ant4g0nist/crashwrangler)

## Referanslar

* [**OS X Incident Response: Scripting and Analysis**](https://www.amazon.com/OS-Incident-Response-Scripting-Analysis-ebook/dp/B01FHOHHVS)
* [**https://www.youtube.com/watch?v=T5xfL9tEg44**](https://www.youtube.com/watch?v=T5xfL9tEg44)
* [**https://taomm.org/vol1/analysis.html**](https://taomm.org/vol1/analysis.html)
* [**The Art of Mac Malware: The Guide to Analyzing Malicious Software**](https://taomm.org/)

<details>

<summary><strong>AWS hackleme konusunda sıfırdan kahramana dönüşmek için</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>'ı öğrenin!</strong></summary>

HackTricks'ı desteklemenin diğer yolları:

* Şirketinizi HackTricks'te **reklamınızı görmek** veya **HackTricks'i PDF olarak indirmek** için [**ABONELİK PLANLARI**](https://github.com/sponsors/carlospolop)'na göz atın!
* [**Resmi PEASS & HackTricks ürünlerini**](https://peass.creator-spring.com) edinin
* Özel [**NFT'lerden**](https://opensea.io/collection/the-peass-family) oluşan koleksiyonumuz [**The PEASS Family**](https://opensea.io/collection/the-peass-family)'i keşfedin
* 💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) **katılın** veya **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)'u **takip edin**.
* **Hacking hilelerinizi** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github reposuna **PR göndererek** paylaşın.

</details>
