<details>

<summary><strong>Sıfırdan kahraman olmaya kadar AWS hackleme</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Kırmızı Takım Uzmanı)</strong></a><strong> öğrenin!</strong></summary>

HackTricks'ı desteklemenin diğer yolları:

* **Şirketinizi HackTricks'te reklamını görmek istiyorsanız** veya **HackTricks'i PDF olarak indirmek istiyorsanız** [**ABONELİK PLANLARI**](https://github.com/sponsors/carlospolop)'na göz atın!
* [**Resmi PEASS & HackTricks ürünlerini**](https://peass.creator-spring.com) edinin
* [**PEASS Ailesi'ni**](https://opensea.io/collection/the-peass-family) keşfedin, özel [**NFT'lerimiz**](https://opensea.io/collection/the-peass-family) koleksiyonumuz
* **Katılın** 💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) veya bizi **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)** takip edin.**
* **Hacking püf noktalarınızı paylaşarak** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github depolarına PR göndererek destek olun.

</details>

<figure><img src="https://pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}


# Zaman Damgaları

Bir saldırgan, **dosyaların zaman damgalarını değiştirmek** isteyebilir ve tespit edilmekten kaçınabilir.\
Zaman damgalarını MFT içindeki `$STANDARD_INFORMATION` ve `$FILE_NAME` özniteliklerinde bulmak mümkündür.

Her iki öznitelik de 4 zaman damgasına sahiptir: **Değiştirme**, **erişim**, **oluşturma** ve **MFT kayıt değiştirme** (MACE veya MACB).

**Windows gezgini** ve diğer araçlar, bilgileri **`$STANDARD_INFORMATION`** özniteliğinden gösterir.

## TimeStomp - Anti-forensik Aracı

Bu araç, zaman damgası bilgilerini **`$STANDARD_INFORMATION`** içinde **değiştirir** **ancak** **`$FILE_NAME`** içindeki bilgileri **değiştirmez**. Bu nedenle, **şüpheli aktiviteyi tanımlamak mümkündür**.

## Usnjrnl

**USN Günlüğü** (Güncelleme Sıra Numarası Günlüğü), NTFS'nin (Windows NT dosya sistemi) bir özelliğidir ve hacim değişikliklerini takip eder. [**UsnJrnl2Csv**](https://github.com/jschicht/UsnJrnl2Csv) aracı, bu değişikliklerin incelenmesine olanak tanır.

![](<../../.gitbook/assets/image (449).png>)

Önceki görüntü, **araç** tarafından gösterilen **çıktıdır** ve dosyaya bazı **değişikliklerin yapıldığı** görülebilir.

## $LogFile

Bir dosya sistemindeki **tüm meta veri değişiklikleri**, [önceden yazma günlüğü](https://en.wikipedia.org/wiki/Write-ahead_logging) olarak bilinen bir süreçte kaydedilir. Kaydedilen meta veriler, NTFS dosya sisteminin kök dizininde bulunan `**$LogFile**` adlı bir dosyada tutulur. [LogFileParser](https://github.com/jschicht/LogFileParser) gibi araçlar, bu dosyayı ayrıştırmak ve değişiklikleri tanımlamak için kullanılabilir.

![](<../../.gitbook/assets/image (450).png>)

Yine, aracın çıktısında **bazı değişikliklerin yapıldığı** görülebilir.

Aynı araç kullanılarak **zaman damgalarının ne zaman değiştirildiğini** belirlemek mümkündür:

![](<../../.gitbook/assets/image (451).png>)

* CTIME: Dosyanın oluşturma zamanı
* ATIME: Dosyanın değiştirme zamanı
* MTIME: Dosyanın MFT kayıt değiştirme zamanı
* RTIME: Dosyanın erişim zamanı

## `$STANDARD_INFORMATION` ve `$FILE_NAME` karşılaştırması

Şüpheli değiştirilmiş dosyaları tanımlamanın başka bir yolu, her iki öznitelikteki zamanı karşılaştırarak **uyumsuzlukları** aramaktır.

## Nanosaniye

**NTFS** zaman damgalarının **100 nanosaniye** hassasiyeti vardır. Dolayısıyla, 2010-10-10 10:10:**00.000:0000 gibi zaman damgalarına sahip dosyalar bulmak çok **şüphelidir**.

## SetMace - Anti-forensik Aracı

Bu araç, hem `$STARNDAR_INFORMATION` hem de `$FILE_NAME` özniteliklerini değiştirebilir. Ancak, Windows Vista'dan itibaren, bu bilgileri değiştirmek için canlı bir işletim sistemi gereklidir.

# Veri Gizleme

NTFS, bir küme ve minimum bilgi boyutu kullanır. Bu, bir dosyanın bir küme ve yarım küme kullanması durumunda, dosya silinene kadar **kalan yarımın asla kullanılmayacağı** anlamına gelir. Bu nedenle, bu "gizli" alanda veri **gizlemek mümkündür**.

Bu "gizli" alanda veri gizlemeyi sağlayan slacker gibi araçlar vardır. Ancak, `$logfile` ve `$usnjrnl` analizi, bazı verilerin eklendiğini gösterebilir:

![](<../../.gitbook/assets/image (452).png>)

Dolayısıyla, FTK Imager gibi araçlar kullanılarak bu yarım alanı kurtarmak mümkündür. Bu tür bir aracın içeriği şifreli veya hatta şifrelenmiş olarak kaydedebileceğini unutmayın.

# UsbKill

Bu, **USB bağlantı noktalarında herhangi bir değişiklik algılandığında bilgisayarı kapatacak bir araçtır**.\
Bunu keşfetmenin bir yolu, çalışan işlemleri incelemek ve **çalışan her python betiğini gözden geçirmektir**.

# Canlı Linux Dağıtımları

Bu dağıtımlar **RAM bellek içinde yürütülür**. Bunları tespit etmenin tek yolu, NTFS dosya sisteminin yazma izinleriyle bağlandığı durumlardır. Salt okuma izinleriyle bağlandığında, sızma algılanamaz.

# Güvenli Silme

[https://github.com/Claudio-C/awesome-data-sanitization](https://github.com/Claudio-C/awesome-data-sanitization)

# Windows Yapılandırması

Forensik incelemeyi zorlaştırmak için çeşitli Windows günlükleme yöntemlerini devre dışı bırakmak mümkündür.

## Zaman Damgalarını Devre Dışı Bırakma - UserAssist

Bu, her bir yürütülebilir dosyanın kullanıcı tarafından çalıştırıldığı tarih ve saatleri tutan bir kayıt defteri anahtarıdır.

UserAssist'in devre dışı bırakılması için iki adım gereklidir:

1. UserAssist'in devre dışı bırakılmasını istediğimizi belirtmek için `HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Advanced\Start_TrackProgs` ve `HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Advanced\Start_TrackEnabled` adlı iki kayıt defteri anahtarı değerini sıfıra ayarlayın.
2. `HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist\<hash>` gibi görünen kayıt defteri alt ağaçlarını temizleyin.

## Zaman Damgalarını Devre Dışı Bırakma - Prefetch

Bu, Windows sisteminin performansını artırmak amacıyla çalıştırılan uygulamalar hakkında bilgi saklar. Ancak, bu aynı zamanda adli bilişim uygulamaları için de faydalı olabilir.

* `regedit`i çalıştırın
* Dosya yolunu seçin `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SessionManager\Memory Management\PrefetchParameters`
* Hem `EnablePrefetcher` hem de `EnableSuperfetch` üzerinde sağ tıklayın
* Her birini değiştirmek için Değiştir'i seçerek değeri 1'den (veya 3'ten) 0'a değiştirin
* Yeniden başlatın

## Zaman Damgalarını Devre Dışı Bırakma - Son Erişim Zamanı

Bir NTFS birimindeki bir klasör açıldığında, sistem her listelenen klasörde **bir zaman damgası alanını günceller** ve buna son erişim zamanı denir. Yoğun kullanılan bir NTFS biriminde, bu performansı etkileyebilir.

1. Kayıt Defteri Düzenleyici'yi (Regedit.exe) açın.
2. `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\FileSystem`'e göz atın.
3. `NtfsDisableLastAccessUpdate`'i arayın. Var olmazsa, bu DWORD'u ekleyin ve değerini 1 olarak ayarlayın, bu işlemi devre dışı bırakacaktır.
4. Kayıt Defteri Düzenleyici'ni kapatın ve sunucuyu yeniden başlatın.
## USB Geçmişini Silme

Tüm **USB Aygıt Girişleri**, PC'nize veya Dizüstü Bilgisayarınıza bir USB Aygıtı takıldığında oluşturulan alt anahtarlar içeren **USBSTOR** kaydı altında Windows Kayıt Defterinde saklanır. Bu anahtarı burada bulabilirsiniz: `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Enum\USBSTOR`. **Bunu silerek** USB geçmişini silebilirsiniz.\
Ayrıca [**USBDeview**](https://www.nirsoft.net/utils/usb_devices_view.html) aracını kullanarak bunları sildiğinizden emin olabilirsiniz (ve silmek için).

USB'ler hakkında bilgi saklayan başka bir dosya, `C:\Windows\INF` içindeki `setupapi.dev.log` dosyasıdır. Bu da silinmelidir.

## Gölge Kopyalarını Devre Dışı Bırakma

`vssadmin list shadowstorage` komutu ile gölge kopyaları **listele**\
Onları silmek için `vssadmin delete shadow` komutunu çalıştırın

Ayrıca [https://www.ubackup.com/windows-10/how-to-delete-shadow-copies-windows-10-5740.html](https://www.ubackup.com/windows-10/how-to-delete-shadow-copies-windows-10-5740.html) adresinde önerilen adımları takip ederek GUI üzerinden de silebilirsiniz.

Gölge kopyalarını devre dışı bırakmak için [buradan adımları](https://support.waters.com/KB_Inf/Other/WKB15560_How_to_disable_Volume_Shadow_Copy_Service_VSS_in_Windows) izleyin:

1. Windows başlat düğmesine tıkladıktan sonra metin arama kutusuna "services" yazarak Hizmetler programını açın.
2. Listeden "Volume Shadow Copy" bulun, seçin ve ardından sağ tıklayarak Özelliklere erişin.
3. "Başlangıç türü" açılır menüsünden Devre Dışı seçin ve ardından Değişiklikleri uygulamak ve Tamam'a tıklayarak değişikliği onaylayın.

Ayrıca hangi dosyaların gölge kopyasının alınacağının yapılandırmasını kayıt defterinde `HKLM\SYSTEM\CurrentControlSet\Control\BackupRestore\FilesNotToSnapshot` değiştirme olasılığı vardır.

## Silinen Dosyaları Üzerine Yazma

* `cipher /w:C` komutunu kullanarak **Windows aracını** kullanabilirsiniz: Bu, C sürücüsündeki kullanılmayan disk alanından verileri kaldırması için cipher'a işaret eder.
* [**Eraser**](https://eraser.heidi.ie) gibi araçları da kullanabilirsiniz

## Windows Olay Günlüklerini Silme

* Windows + R --> eventvwr.msc --> "Windows Günlükleri"ni genişletin --> Her kategoriye sağ tıklayın ve "Günlüğü Temizle" seçeneğini seçin
* `for /F "tokens=*" %1 in ('wevtutil.exe el') DO wevtutil.exe cl "%1"`
* `Get-EventLog -LogName * | ForEach { Clear-EventLog $_.Log }`

## Windows Olay Günlüklerini Devre Dışı Bırakma

* `reg add 'HKLM\SYSTEM\CurrentControlSet\Services\eventlog' /v Start /t REG_DWORD /d 4 /f`
* Hizmetler bölümünde "Windows Olay Günlüğü" hizmetini devre dışı bırakın
* `WEvtUtil.exec clear-log` veya `WEvtUtil.exe cl`

## $UsnJrnl'yi Devre Dışı Bırakma

* `fsutil usn deletejournal /d c:`

<figure><img src="https://pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}
