# Anti-Forensic Teknikleri

<details>

<summary><strong>Sıfırdan kahraman olmak için AWS hackleme öğrenin</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Kırmızı Takım Uzmanı)</strong></a><strong>!</strong></summary>

HackTricks'ı desteklemenin diğer yolları:

* **Şirketinizi HackTricks'te reklamını görmek istiyorsanız** veya **HackTricks'i PDF olarak indirmek istiyorsanız** [**ABONELİK PLANLARI**](https://github.com/sponsors/carlospolop)'na göz atın!
* [**Resmi PEASS & HackTricks ürünlerini**](https://peass.creator-spring.com) edinin
* [**PEASS Ailesi'ni**](https://opensea.io/collection/the-peass-family) keşfedin, özel [**NFT'lerimiz**](https://opensea.io/collection/the-peass-family) koleksiyonumuzu
* **Katılın** 💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) veya bizi **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** takip edin**.
* **Hacking püf noktalarınızı paylaşarak PR'lar göndererek** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github depolarına katkıda bulunun.

</details>

## Zaman Damgaları

Bir saldırgan, **dosyaların zaman damgalarını değiştirmek** isteyebilir.\
Zaman damgalarını `$STANDARD_INFORMATION` ve `$FILE_NAME` özniteliklerinde MFT içinde bulmak mümkündür.

Her iki öznitelik de 4 zaman damgasına sahiptir: **Değiştirme**, **erişim**, **oluşturma** ve **MFT kayıt değişikliği** (MACE veya MACB).

**Windows Gezgini** ve diğer araçlar, bilgileri **`$STANDARD_INFORMATION`** özniteliğinden gösterir.

### TimeStomp - Anti-forensic Aracı

Bu araç, **`$STANDARD_INFORMATION`** içindeki zaman damgası bilgilerini **değiştirir** **ancak** **`$FILE_NAME`** içindeki bilgileri **değiştirmez**. Bu nedenle, **şüpheli aktiviteleri tespit etmek mümkündür**.

### Usnjrnl

**USN Journal** (Güncelleme Sıra Numarası Günlüğü), NTFS'in (Windows NT dosya sistemi) bir özelliğidir ve hacim değişikliklerini takip eder. [**UsnJrnl2Csv**](https://github.com/jschicht/UsnJrnl2Csv) aracı, bu değişikliklerin incelenmesine olanak tanır.

![](<../../.gitbook/assets/image (801).png>)

Önceki görüntü, **aracın çıktısı** olup dosyaya bazı **değişikliklerin uygulandığını** gözlemlemek mümkündür.

### $LogFile

Bir dosya sistemindeki **tüm meta veri değişiklikleri**, [önceden yazma günlüğü](https://en.wikipedia.org/wiki/Write-ahead\_logging) olarak bilinen bir süreçte kaydedilir. Kaydedilen meta veriler, NTFS dosya sisteminin kök dizininde bulunan `**$LogFile**` adlı bir dosyada tutulur. [LogFileParser](https://github.com/jschicht/LogFileParser) gibi araçlar, bu dosyayı ayrıştırmak ve değişiklikleri tanımlamak için kullanılabilir.

![](<../../.gitbook/assets/image (137).png>)

Yine, aracın çıktısında **bazı değişikliklerin yapıldığı** görülebilir.

Aynı araç kullanılarak **zaman damgalarının ne zaman değiştirildiği** belirlenebilir:

![](<../../.gitbook/assets/image (1089).png>)

* CTIME: Dosyanın oluşturma zamanı
* ATIME: Dosyanın değiştirme zamanı
* MTIME: Dosyanın MFT kayıt değişikliği
* RTIME: Dosyanın erişim zamanı

### `$STANDARD_INFORMATION` ve `$FILE_NAME` karşılaştırması

Şüpheli değiştirilmiş dosyaları tanımlamanın başka bir yolu, her iki öznitelikteki zamanı karşılaştırarak **uyumsuzlukları** aramaktır.

### Nanosaniyeler

**NTFS** zaman damgalarının **100 nanosaniye hassasiyeti** vardır. Dolayısıyla, 2010-10-10 10:10:**00.000:0000 gibi zaman damgalarına sahip dosyalar bulmak çok şüphelidir**.

### SetMace - Anti-forensic Aracı

Bu araç, hem `$STARNDAR_INFORMATION` hem de `$FILE_NAME` özniteliklerini değiştirebilir. Ancak, Windows Vista'dan itibaren, bu bilgileri değiştirmek için canlı bir işletim sistemi gereklidir.

## Veri Gizleme

NTFS, bir küme ve minimum bilgi boyutu kullanır. Bu, bir dosyanın bir kümeyi ve yarım kümeyi işgal ettiği durumda, **kalan yarım kümeyi dosya silinene kadar asla kullanılmayacağı anlamına gelir**. Bu nedenle, bu "gizli" alanda veri gizlemek mümkündür.

Bu "gizli" alanda veri gizlemeyi sağlayan slacker gibi araçlar bulunmaktadır. Ancak, `$logfile` ve `$usnjrnl` analizi, bazı verilerin eklendiğini gösterebilir:

![](<../../.gitbook/assets/image (1060).png>)

Dolayısıyla, FTK Imager gibi araçlar kullanılarak bu yarım kümeyi kurtarmak mümkündür. Bu tür bir aracın içeriği şifreli veya hatta şifrelenmiş olarak kaydedebileceğini unutmayın.

## UsbKill

Bu, **USB bağlantı noktalarında herhangi bir değişiklik algılandığında bilgisayarı kapatacak bir araçtır**.\
Bunu keşfetmenin bir yolu, çalışan işlemleri incelemek ve **çalışan her python betiğini gözden geçirmektir**.

## Canlı Linux Dağıtımları

Bu dağıtımlar **RAM bellek içinde çalıştırılır**. Bunları tespit etmenin tek yolu, NTFS dosya sisteminin yazma izinleriyle bağlandığı durumlarda mümkündür. Salt okuma izinleriyle bağlandığında, sızma algılanamaz.

## Güvenli Silme

[https://github.com/Claudio-C/awesome-data-sanitization](https://github.com/Claudio-C/awesome-data-sanitization)

## Windows Yapılandırması

Forensik incelemeyi zorlaştırmak için çeşitli Windows günlükleme yöntemlerini devre dışı bırakmak mümkündür.

### Zaman Damgalarını Devre Dışı Bırakma - UserAssist

Bu, her bir yürütülebilir dosyanın kullanıcı tarafından çalıştırıldığı tarih ve saatleri tutan bir kayıt defteri anahtarıdır.

UserAssist'in devre dışı bırakılması için iki adım gereklidir:

1. UserAssist'in devre dışı bırakılmasını istediğimizi belirtmek için `HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Advanced\Start_TrackProgs` ve `HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Advanced\Start_TrackEnabled` olmak üzere iki kayıt defteri anahtarı ayarlayın ve her ikisini de sıfır yapın.
2. `HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist\<hash>` gibi görünen kayıt defteri alt ağaçlarını temizleyin.

### Zaman Damgalarını Devre Dışı Bırakma - Prefetch

Bu, Windows sisteminin performansını iyileştirmek amacıyla yürütülen uygulamalar hakkında bilgi saklar. Ancak, bu aynı zamanda adli bilişim uygulamaları için de faydalı olabilir.

* `regedit`i çalıştırın
* Dosya yolunu seçin `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SessionManager\Memory Management\PrefetchParameters`
* Hem `EnablePrefetcher` hem de `EnableSuperfetch` üzerinde sağ tıklayın
* Her birini değiştirmek için bunlardan her birine tıklayarak Değiştir'i seçin ve değeri 1'den (veya 3'ten) 0'a değiştirin
* Yeniden başlatın

### Zaman Damgalarını Devre Dışı Bırakma - Son Erişim Zamanı

Bir NTFS birimindeki bir klasör bir Windows NT sunucusunda açıldığında, sistem her listelenen klasörde **bir zaman damgası alanını günceller**, bu alana son erişim zamanı denir. Yoğun kullanılan bir NTFS biriminde, bu performansı etkileyebilir.

1. Kayıt Defteri Düzenleyicisi'ni (Regedit.exe) açın.
2. `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\FileSystem`'e göz atın.
3. `NtfsDisableLastAccessUpdate`'i arayın. Var olmazsa, bu DWORD'u ekleyin ve değerini 1 olarak ayarlayın, bu işlemi devre dışı bırakacaktır.
4. Kayıt Defteri Düzenleyici'ni kapatın ve sunucuyu yeniden başlatın.
### USB Geçmişini Silme

Tüm **USB Aygıt Girişleri**, PC'nize veya Dizüstü Bilgisayarınıza bir USB Aygıtı takıldığında oluşturulan alt anahtarlar içeren **USBSTOR** kaydı altında Windows Kayıt Defterinde saklanır. Bu anahtarı burada bulabilirsiniz: `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Enum\USBSTOR`. **Bunu silerek** USB geçmişini silebilirsiniz.\
Ayrıca [**USBDeview**](https://www.nirsoft.net/utils/usb_devices_view.html) aracını kullanarak bunları sildiğinizden emin olabilirsiniz (ve silebilirsiniz).

USB'ler hakkında bilgi saklayan başka bir dosya, `C:\Windows\INF` içindeki `setupapi.dev.log` dosyasıdır. Bu da silinmelidir.

### Gölge Kopyalarını Devre Dışı Bırakma

`vssadmin list shadowstorage` komutu ile gölge kopyaları **listele**\
Onları silmek için `vssadmin delete shadow` komutunu çalıştırın

Ayrıca [https://www.ubackup.com/windows-10/how-to-delete-shadow-copies-windows-10-5740.html](https://www.ubackup.com/windows-10/how-to-delete-shadow-copies-windows-10-5740.html) adresinde önerilen adımları takip ederek GUI üzerinden de silebilirsiniz.

Gölge kopyalarını devre dışı bırakmak için [buradan adımları](https://support.waters.com/KB_Inf/Other/WKB15560_How_to_disable_Volume_Shadow_Copy_Service_VSS_in_Windows) izleyin:

1. Başlat düğmesine tıkladıktan sonra metin arama kutusuna "hizmetler" yazarak Hizmetler programını açın.
2. Listeden "Volume Shadow Copy" bulun, seçin ve ardından sağ tıklayarak Özelliklere erişin.
3. "Başlangıç türü" açılır menüsünden Devre Dışı seçin ve Değişikliği uygulamak ve Tamam'a tıklayarak değişikliği onaylayın.

Ayrıca hangi dosyaların gölge kopyasının alınacağının yapılandırmasını kayıt defterinde `HKLM\SYSTEM\CurrentControlSet\Control\BackupRestore\FilesNotToSnapshot` değiştirme olasılığı vardır.

### Silinen Dosyaları Üzerine Yazma

* `cipher /w:C` komutunu kullanabilirsiniz. Bu, cipher'a C sürücüsü içindeki kullanılmayan disk alanından herhangi bir veriyi kaldırmasını söyler.
* [**Eraser**](https://eraser.heidi.ie) gibi araçları da kullanabilirsiniz

### Windows Olay Günlüklerini Silme

* Windows + R --> eventvwr.msc --> "Windows Günlükleri"ni genişletin --> Her kategoriye sağ tıklayın ve "Günlüğü Temizle" seçeneğini seçin
* `for /F "tokens=*" %1 in ('wevtutil.exe el') DO wevtutil.exe cl "%1"`
* `Get-EventLog -LogName * | ForEach { Clear-EventLog $_.Log }`

### Windows Olay Günlüklerini Devre Dışı Bırakma

* `reg add 'HKLM\SYSTEM\CurrentControlSet\Services\eventlog' /v Start /t REG_DWORD /d 4 /f`
* Hizmetler bölümünde "Windows Olay Günlüğü" hizmetini devre dışı bırakın
* `WEvtUtil.exec clear-log` veya `WEvtUtil.exe cl`

### $UsnJrnl'yi Devre Dışı Bırakma

* `fsutil usn deletejournal /d c:`
