<details>

<summary><strong>AWS hacklemeyi sıfırdan kahramana öğrenin</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Kırmızı Takım Uzmanı)</strong></a><strong> ile</strong>!</summary>

HackTricks'ı desteklemenin diğer yolları:

* **Şirketinizi HackTricks'te reklamını görmek istiyorsanız** veya **HackTricks'i PDF olarak indirmek istiyorsanız** [**ABONELİK PLANLARI**](https://github.com/sponsors/carlospolop)'na göz atın!
* [**Resmi PEASS & HackTricks ürünlerini**](https://peass.creator-spring.com) edinin
* [**PEASS Ailesi'ni**](https://opensea.io/collection/the-peass-family) keşfedin, özel [**NFT'lerimiz**](https://opensea.io/collection/the-peass-family) koleksiyonumuz
* **💬 [**Discord grubumuza**](https://discord.gg/hRep4RUj7f) veya [**telegram grubumuza**](https://t.me/peass) katılın veya bizi **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)'da **takip edin**.
* **Hacking püf noktalarınızı paylaşarak PR'lar göndererek** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github depolarına katkıda bulunun.

</details>


# Temel Bilgiler

SPI (Serial Peripheral Interface), gömülü sistemlerde kullanılan kısa mesafe iletişimi için kullanılan Senkron Seri İletişim Protokolüdür. SPI İletişim Protokolü, saat ve Çip Seçim Sinyali tarafından orkestrasyonu yapılan master-slave mimarisinden yararlanır. Bir master-slave mimarisi, genellikle bir mikroişlemci olan bir ana bilgisayarın EEPROM, sensörler, kontrol cihazları vb. gibi harici bileşenleri yönettiği ve köle olarak kabul edilen cihazlardan oluşur.

Bir ana bilgisayara birden fazla köle bağlanabilir ancak köleler birbirleriyle iletişim kuramaz. Köleler, saat ve çip seçim olmak üzere iki pin tarafından yönetilir. SPI, senkron bir iletişim protokolü olduğundan, giriş ve çıkış pinleri saat sinyallerini takip eder. Ana bilgisayar, bir köleyi seçmek ve onunla etkileşimde bulunmak için çip seçimini kullanır. Çip seçim yüksek olduğunda, köle cihaz seçilmezken, düşük olduğunda çip seçilmiş olur ve ana bilgisayar köle ile etkileşimde bulunur.

MOSI (Master Out, Slave In) ve MISO (Master In, Slave Out) veri gönderme ve alma işlemlerinden sorumludur. Veri, MOSI pini aracılığıyla köle cihaza gönderilirken çip seçimi düşük tutulur. Giriş verisi, köle cihazın veri sayfasına göre talimatlar, bellek adresleri veya veriler içerir. Geçerli bir girişte, MISO pini verileri ana bilgisayara iletmekten sorumludur. Çıkış verisi, giriş bittikten hemen sonra bir sonraki saat döngüsünde gönderilir. MISO pinleri, veri tamamen iletilene kadar veya ana bilgisayar çip seçim pini yüksek olduğunda (bu durumda, köle iletimi durdurur ve ana bilgisayar o saat döngüsünden sonra dinlemez).

# Flashı Dök

## Bus Pirate + flashrom

![](<../../.gitbook/assets/image (201).png>)

Unutmayın ki, Pirate Bus'un PINOUT'unun **MOSI** ve **MISO** pinlerini SPI'ye bağlamak için göstermesine rağmen, bazı SPI'ler DI ve DO olarak pinleri gösterebilir. **MOSI -> DI, MISO -> DO**

![](<../../.gitbook/assets/image (648) (1) (1).png>)

Windows veya Linux'ta, flash belleğin içeriğini dökmek için [**`flashrom`**](https://www.flashrom.org/Flashrom) programını çalıştırarak şunun gibi bir şey yapabilirsiniz:
```bash
# In this command we are indicating:
# -VV Verbose
# -c <chip> The chip (if you know it better, if not, don'tindicate it and the program might be able to find it)
# -p <programmer> In this case how to contact th chip via the Bus Pirate
# -r <file> Image to save in the filesystem
flashrom -VV -c "W25Q64.V" -p buspirate_spi:dev=COM3 -r flash_content.img
```
<details>

<summary><strong>Sıfırdan kahraman olmaya kadar AWS hacklemeyi öğrenin</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks'ı desteklemenin diğer yolları:

* **Şirketinizi HackTricks'te reklamını görmek istiyorsanız** veya **HackTricks'i PDF olarak indirmek istiyorsanız** [**ABONELİK PLANLARI**]'na göz atın (https://github.com/sponsors/carlospolop)!
* [**Resmi PEASS & HackTricks ürünlerini**](https://peass.creator-spring.com) edinin
* [**The PEASS Family**]'yi (https://opensea.io/collection/the-peass-family) keşfedin, özel [**NFT'lerimiz**](https://opensea.io/collection/the-peass-family) koleksiyonumuz
* **Katılın** 💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) veya bizi **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)** takip edin.**
* **Hacking püf noktalarınızı paylaşarak PR göndererek** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github depolarına.

</details>
