# Dosya/Veri Oyma ve Kurtarma Araçları

<details>

<summary><strong>Sıfırdan kahraman olmak için AWS hackleme öğrenin</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Kırmızı Takım Uzmanı)</strong></a><strong>!</strong></summary>

HackTricks'ı desteklemenin diğer yolları:

* **Şirketinizi HackTricks'te reklamını görmek istiyorsanız** veya **HackTricks'i PDF olarak indirmek istiyorsanız** [**ABONELİK PLANLARI**]'na göz atın (https://github.com/sponsors/carlospolop)!
* [**Resmi PEASS & HackTricks ürünlerini**](https://peass.creator-spring.com) edinin
* [**The PEASS Family**]'yi keşfedin (https://opensea.io/collection/the-peass-family), özel [**NFT'lerimiz**](https://opensea.io/collection/the-peass-family) koleksiyonumuz
* **Katılın** 💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) veya bizi **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**'da takip edin.**
* **Hacking püf noktalarınızı paylaşarak PR'lar göndererek** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github depolarına katkıda bulunun.

</details>

**Try Hard Güvenlik Grubu**

<figure><img src="../.gitbook/assets/telegram-cloud-document-1-5159108904864449420.jpg" alt=""><figcaption></figcaption></figure>

{% embed url="https://discord.gg/tryhardsecurity" %}

***

## Oyma ve Kurtarma Araçları

Daha fazla araç için [https://github.com/Claudio-C/awesome-datarecovery](https://github.com/Claudio-C/awesome-datarecovery)

### Otopsi

Görüntülerden dosyaları çıkarmak için en yaygın kullanılan araç [**Otopsi**](https://www.autopsy.com/download/)'dir. İndirin, kurun ve dosyayı içeri alarak "gizli" dosyaları bulun. Otopsi'nin disk görüntüleri ve diğer türdeki görüntüleri desteklemek üzere yapıldığını unutmayın, ancak basit dosyaları desteklemez.

### Binwalk <a href="#binwalk" id="binwalk"></a>

**Binwalk**, gömülü içerik bulmak için ikili dosyaları analiz etmek için bir araçtır. `apt` aracılığıyla yüklenebilir ve kaynağı [GitHub](https://github.com/ReFirmLabs/binwalk)'da bulunabilir.

**Kullanışlı komutlar**:
```bash
sudo apt install binwalk #Insllation
binwalk file #Displays the embedded data in the given file
binwalk -e file #Displays and extracts some files from the given file
binwalk --dd ".*" file #Displays and extracts all files from the given file
```
### Foremost

Gizli dosyaları bulmak için başka bir yaygın araç **foremost**'tir. Foremost'un yapılandırma dosyasını `/etc/foremost.conf` içinde bulabilirsiniz. Belirli dosyaları aramak istiyorsanız, onları yorum satırından çıkarın. Hiçbir şeyi yorum satırından çıkarmazsanız, foremost varsayılan olarak yapılandırılmış dosya türlerini arayacaktır.
```bash
sudo apt-get install foremost
foremost -v -i file.img -o output
#Discovered files will appear inside the folder "output"
```
### **Scalpel**

**Scalpel**, başka bir araçtır ve bir dosyanın içine gömülü olan dosyaları bulmak ve çıkarmak için kullanılabilir. Bu durumda, çıkarmak istediğiniz dosya türlerini yapılandırma dosyasından (_/etc/scalpel/scalpel.conf_) yorum satırından çıkarmalısınız.
```bash
sudo apt-get install scalpel
scalpel file.img -o output
```
### Bulk Extractor

Bu araç Kali içinde gelir ancak burada da bulabilirsiniz: [https://github.com/simsong/bulk\_extractor](https://github.com/simsong/bulk\_extractor)

Bu araç bir görüntüyü tarayabilir ve içindeki **pcap'leri**, **ağ bilgilerini (URL'ler, alan adları, IP'ler, MAC'ler, e-postaları)** ve daha fazla **dosyayı çıkarabilir**. Yapmanız gereken tek şey:
```
bulk_extractor memory.img -o out_folder
```
### PhotoRec

[PhotoRec](https://www.cgsecurity.org/wiki/TestDisk_Download) bulabilirsiniz.

GUI ve CLI sürümleriyle gelir. PhotoRec'in aramasını istediğiniz **dosya türlerini** seçebilirsiniz.

![](<../../../.gitbook/assets/image (524).png>)

### binvis

Kodu [buradan](https://code.google.com/archive/p/binvis/) ve [web sayfa aracını](https://binvis.io/#/) kontrol edin.

#### BinVis'in Özellikleri

- Görsel ve etkin **yapı görüntüleyici**
- Farklı odak noktaları için birden fazla çizim
- Bir örneğin bölümlerine odaklanma
- PE veya ELF yürütülebilir dosyalarda **dizileri ve kaynakları** görmek
- Dosyalarda kriptoanaliz için **desenler** elde etme
- Paketleyici veya kodlayıcı algoritmaları **tespit etme**
- Desenler aracılığıyla **Steganografiyi tanımlama**
- **Görsel** ikili farklılaştırma

BinVis, siyah kutu senaryosunda bilinmeyen bir hedefle tanışmak için harika bir **başlangıç noktasıdır**.

## Özel Veri Kazıma Araçları

### FindAES

AES anahtarlarını arayarak anahtar programlarını arar. TrueCrypt ve BitLocker gibi 128, 192 ve 256 bit anahtarlar gibi anahtarları bulabilir.

[İndirin buradan](https://sourceforge.net/projects/findaes/).

## Tamamlayıcı araçlar

Resimleri terminalden görmek için [**viu**](https://github.com/atanunq/viu)'yu kullanabilirsiniz.\
Bir pdf'i metne dönüştürmek ve okumak için linux komut satırı aracı **pdftotext**'i kullanabilirsiniz.

**Try Hard Security Group**

<figure><img src="../.gitbook/assets/telegram-cloud-document-1-5159108904864449420.jpg" alt=""><figcaption></figcaption></figure>

{% embed url="https://discord.gg/tryhardsecurity" %}

<details>

<summary><strong>Sıfırdan kahraman olmak için AWS hackleme öğrenin</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks'i desteklemenin diğer yolları:

* **Şirketinizi HackTricks'te reklamını görmek istiyorsanız** veya **HackTricks'i PDF olarak indirmek istiyorsanız** [**ABONELİK PLANLARINI**](https://github.com/sponsors/carlospolop) kontrol edin!
* [**Resmi PEASS & HackTricks ürünlerini**](https://peass.creator-spring.com) edinin
* Özel [**NFT'lerimiz olan**](https://opensea.io/collection/the-peass-family) [**The PEASS Family**](https://opensea.io/collection/the-peass-family) koleksiyonumuzu keşfedin
* 💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) katılın veya bizi **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)'da takip edin.
* **Hacking püf noktalarınızı göndererek HackTricks ve HackTricks Cloud** github depolarına PR göndererek paylaşın.

</details>
