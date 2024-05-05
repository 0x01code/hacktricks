# UART

<details>

<summary><strong>Sıfırdan kahraman olmak için AWS hackleme öğrenin</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Kırmızı Takım Uzmanı)</strong></a><strong> ile</strong></summary>

HackTricks'ı desteklemenin diğer yolları:

* **Şirketinizi HackTricks'te reklamınızı görmek istiyorsanız** veya **HackTricks'i PDF olarak indirmek istiyorsanız** [**ABONELİK PLANLARI**](https://github.com/sponsors/carlospolop)'na göz atın!
* [**Resmi PEASS & HackTricks ürünlerini**](https://peass.creator-spring.com) edinin
* [**PEASS Ailesi'ni**](https://opensea.io/collection/the-peass-family) keşfedin, özel [**NFT'lerimiz**](https://opensea.io/collection/the-peass-family) koleksiyonumuz
* **Katılın** 💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) veya bizi **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)'da **takip edin**.
* **Hacking püf noktalarınızı paylaşarak** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github depolarına PR göndererek.

</details>

### [WhiteIntel](https://whiteintel.io)

<figure><img src="../../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) **dark-web** destekli bir arama motorudur ve şirketin veya müşterilerinin **hırsız kötü amaçlı yazılımlar** tarafından **kompromize edilip edilmediğini** kontrol etmek için **ücretsiz** işlevler sunar.

WhiteIntel'in asıl amacı, bilgi çalan kötü amaçlı yazılımlardan kaynaklanan hesap ele geçirmeleri ve fidye saldırılarıyla mücadele etmektir.

Websitesini ziyaret edebilir ve motorlarını **ücretsiz** deneyebilirsiniz:

{% embed url="https://whiteintel.io" %}

***

## Temel Bilgiler

UART, verileri bileşenler arasında bir bit aynı anda aktaran bir seri iletişim protokolüdür. Buna karşılık, paralel iletişim protokolleri verileri aynı anda birden fazla kanaldan iletilir. Yaygın seri iletişim protokolleri arasında RS-232, I2C, SPI, CAN, Ethernet, HDMI, PCI Express ve USB bulunur.

Genel olarak, UART boşta iken (mantıksal 1 değerinde) hat yüksekte tutulur. Daha sonra, veri transferinin başlangıcını belirtmek için verici, alıcıya bir başlangıç biti gönderir, bu sırada sinyal düşükte (mantıksal 0 değerinde) tutulur. Daha sonra, verici gerçek mesajı içeren beş ila sekiz veri biti gönderir, ardından isteğe bağlı bir çiftlik biti ve bir veya iki durdurma biti (mantıksal 1 değerinde) gönderir, yapılandırmaya bağlı olarak. Hata denetimi için kullanılan çiftlik biti, uygulamada nadiren görülür. Durdurma biti (veya bitleri) iletimin sonunu belirtir.

En yaygın yapılandırmayı 8N1 olarak adlandırıyoruz: sekiz veri biti, çiftlik yok ve bir durdurma biti. Örneğin, 8N1 UART yapılandırmasında karakter C'yi veya ASCII'de 0x43'ü göndermek isteseydik, aşağıdaki bitleri gönderirdik: 0 (başlangıç biti); 0, 1, 0, 0, 0, 0, 1, 1 (ikili 0x43 değeri) ve 0 (dur biti).

![](<../../.gitbook/assets/image (764).png>)

UART ile iletişim kurmak için donanım araçları:

* USB-seri adaptör
* CP2102 veya PL2303 çipli adaptörler
* Bus Pirate, Adafruit FT232H, Shikra veya Attify Badge gibi çok amaçlı araçlar

### UART Portlarını Tanımlama

UART'ın 4 portu vardır: **TX**(Gönder), **RX**(Al), **Vcc**(Gerilim) ve **GND**(Toprak). PCB'de **`TX`** ve **`RX`** harflerinin **yazılı olduğu** 4 port bulabilirsiniz. Ancak işaret yoksa, bir **multimetre** veya bir **mantık analizörü** kullanarak kendiniz bulmanız gerekebilir.

Bir **multimetre** ve cihaz kapalıyken:

* **GND** pini tanımlamak için **Süreklilik Testi** modunu kullanın, arka ucu toprağa yerleştirin ve kırmızı ucu ile test edin, multimetreden bir ses duyana kadar. PCB'de birkaç GND pini bulunabilir, bu nedenle UART'a ait olanı bulmuş olabilirsiniz veya olmayabilirsiniz.
* **VCC portunu** tanımlamak için **DC gerilim modunu** ayarlayın ve 20 V gerilime kadar ayarlayın. Siyah probu toprağa, kırmızı probu pine yerleştirin. Cihazı açın. Multimetre sabit bir 3.3 V veya 5 V gerilim ölçerse, Vcc pini bulmuşsunuz demektir. Başka gerilimler alırsanız, diğer portlarla tekrar deneyin.
* **TX** **portunu** tanımlamak için **DC gerilim modunu** 20 V gerilime kadar ayarlayın, siyah probu toprağa, kırmızı probu pine yerleştirin ve cihazı açın. Gerilimin birkaç saniye boyunca dalgalanıp daha sonra Vcc değerinde sabitlendiğini bulursanız, muhtemelen TX portunu bulmuşsunuzdur. Bu, cihazı açarken bazı hata ayıklama verileri gönderdiği içindir.
* **RX portu**, diğer 3'e en yakın olanı olacaktır, en düşük gerilim dalgalanması ve tüm UART pinlerinin en düşük genel değerine sahiptir.

TX ve RX portlarını karıştırabilirsiniz ve hiçbir şey olmaz, ancak GND ve VCC portlarını karıştırırsanız devreyi yakabilirsiniz.

Bazı hedef cihazlarda üretici tarafından RX veya TX veya hatta her ikisi devre dışı bırakılarak UART portu devre dışı bırakılabilir. Bu durumda, devre kartındaki bağlantıları izlemek ve bazı kırılma noktalarını bulmak faydalı olabilir. Cihazın garantisinin olup olmadığını kontrol etmek, UART'nin algılanmadığını ve devrenin kırıldığını doğrulamak hakkında güçlü bir ipucudur. Cihazın garanti ile gönderilmiş olması durumunda, üretici bazı hata ayıklama arayüzleri (bu durumda UART) bırakır ve bu nedenle UART'yi bağlantıyı keser ve hata ayıklama yaparken tekrar bağlar. Bu kırılma pinleri lehimleme veya jumper tellerle bağlanabilir.

### UART Baud Oranını Tanımlama

Doğru baud oranını tanımlamanın en kolay yolu, **TX pini çıkışını incelemek ve veriyi okumaya çalışmaktır**. Aldığınız veri okunabilir değilse, veri okunabilir hale gelene kadar bir sonraki mümkün baud oranına geçin. Bunu yapmak için bir USB-seri adaptör veya Bus Pirate gibi çok amaçlı bir cihaz kullanabilir ve [baudrate.py](https://github.com/devttys0/baudrate/) gibi bir yardımcı betikle eşleştirebilirsiniz. En yaygın baud oranları 9600, 38400, 19200, 57600 ve 115200'dür.

{% hint style="danger" %}
Bu protokolde bir cihazın TX'sini diğer cihazın RX'ine bağlamanız gerektiğini unutmamak önemlidir!
{% endhint %}

## CP210X UART to TTY Adaptörü

CP210X Çipi, NodeMCU (esp8266 ile) gibi birçok prototip kartında Seri İletişim için kullanılır. Bu adaptörler oldukça ucuzdur ve hedefin UART arabirimine bağlanmak için kullanılabilir. Cihazın 5 pini vardır: 5V, GND, RXD, TXD, 3.3V. Herhangi bir hasarı önlemek için hedefin desteklediği gerilimi bağlamayı unutmayın. Son olarak, Adaptörün RXD pimini hedefin TXD'sine ve Adaptörün TXD pimini hedefin RXD'sine bağlayın.

Adaptör algılanmazsa, CP210X sürücülerinin ana sistemde yüklü olduğundan emin olun. Adaptör algılandığında ve bağlandığında, picocom, minicom veya screen gibi araçlar kullanılabilir.

Linux/MacOS sistemlerine bağlı cihazları listelemek için:
```
ls /dev/
```
UART arayüzü ile temel etkileşim için aşağıdaki komutu kullanın:
```
picocom /dev/<adapter> --baud <baudrate>
```
minicom için, onu yapılandırmak için aşağıdaki komutu kullanın:
```
minicom -s
```
`Serial port setup` seçeneğinde baud hızı ve cihaz adı gibi ayarları yapılandırın.

Yapılandırmadan sonra `minicom` komutunu kullanarak UART Konsolunu başlatın.

## Arduino UNO R3 Üzerinden UART (Çıkarılabilir Atmel 328p Yonga Kartları)

UART Seri USB adaptörleri mevcut değilse, Arduino UNO R3 hızlı bir hile ile kullanılabilir. Arduino UNO R3 genellikle her yerde bulunabilir olduğundan, bu yöntem çok zaman kazandırabilir.

Arduino UNO R3'ün kendisinde kart üzerinde bulunan bir USB'den Seri adaptörü vardır. UART bağlantısı almak için, sadece Arduino'dan Atmel 328p mikrodenetleyici yongasını çıkarın. Bu hile, Arduino UNO R3 varyantlarında (SMD versiyonunda kullanılan) kart üzerine lehimlenmemiş olan Atmel 328p'ye sahip olanlarda çalışır. Arduino'nun RX pini (Dijital Pin 0) ile UART Arayüzünün TX pini ve Arduino'nun TX pini (Dijital Pin 1) ile UART arayüzünün RX pini bağlanır.

Son olarak, UART arayüzüne göre baud hızını ayarlayarak Arduino IDE'yi kullanmanız önerilir.

## Bus Pirate

Bu senaryoda, programın tüm çıktılarını Seri Monitöre gönderen Arduino'nun UART iletişimini izleyeceğiz.
```bash
# Check the modes
UART>m
1. HiZ
2. 1-WIRE
3. UART
4. I2C
5. SPI
6. 2WIRE
7. 3WIRE
8. KEYB
9. LCD
10. PIC
11. DIO
x. exit(without change)

# Select UART
(1)>3
Set serial port speed: (bps)
1. 300
2. 1200
3. 2400
4. 4800
5. 9600
6. 19200
7. 38400
8. 57600
9. 115200
10. BRG raw value

# Select the speed the communication is occurring on (you BF all this until you find readable things)
# Or you could later use the macro (4) to try to find the speed
(1)>5
Data bits and parity:
1. 8, NONE *default
2. 8, EVEN
3. 8, ODD
4. 9, NONE

# From now on pulse enter for default
(1)>
Stop bits:
1. 1 *default
2. 2
(1)>
Receive polarity:
1. Idle 1 *default
2. Idle 0
(1)>
Select output type:
1. Open drain (H=Hi-Z, L=GND)
2. Normal (H=3.3V, L=GND)

(1)>
Clutch disengaged!!!
To finish setup, start up the power supplies with command 'W'
Ready

# Start
UART>W
POWER SUPPLIES ON
Clutch engaged!!!

# Use macro (2) to read the data of the bus (live monitor)
UART>(2)
Raw UART input
Any key to exit
Escritura inicial completada:
AAA Hi Dreg! AAA
waiting a few secs to repeat....
```
## UART Konsolu ile Firmware'in Dump Edilmesi

UART Konsolu, çalışma zamanı ortamındaki temel firmware ile çalışmanın harika bir yolunu sağlar. Ancak UART Konsolu erişimi salt okunur olduğunda birçok kısıtlama getirebilir. Birçok gömülü cihazda, firmware EEPROM'larda depolanır ve geçici belleğe sahip işlemcilerde yürütülür. Dolayısıyla, firmware, üretim sırasında EEPROM içindeki orijinal firmware olduğundan ve herhangi yeni dosyalar geçici bellek nedeniyle kaybolacağından salt okunur tutulur. Bu nedenle, gömülü firmware'lerle çalışırken firmware'in dump edilmesi değerli bir çabadır.

Bunu yapmanın birçok yolu vardır ve SPI bölümü, çeşitli cihazlarla EEPROM'dan firmware'in doğrudan çıkarılma yöntemlerini kapsar. Bununla birlikte, fiziksel cihazlar ve harici etkileşimlerle firmware'in dump edilmesini ilk denemeden önce UART ile yapmayı önerilir.

UART Konsolundan firmware dump etmek, öncelikle bootloader'lara erişim sağlamayı gerektirir. Birçok popüler satıcı, Linux'u yüklemek için bootloader olarak uboot (Universal Bootloader) kullanır. Bu nedenle, uboot'a erişim sağlamak gereklidir.

Bootloader'a erişmek için, UART portunu bilgisayara bağlayın ve herhangi bir Seri Konsol aracını kullanın ve cihazın güç kaynağını bağlı olmaktan çıkarın. Kurulum hazır olduğunda, Enter tuşuna basılı tutun. Son olarak, cihazın güç kaynağını bağlayın ve başlatmasına izin verin.

Bunu yapmak, uboot'un yüklenmesini kesintiye uğratacak ve bir menü sağlayacaktır. Uboot komutlarını anlamak ve bunları listelemek için yardım menüsünü kullanmak önerilir. Bu muhtemelen `help` komutu olabilir. Farklı satıcılar farklı yapılandırmaları kullandıklarından, bunların her birini ayrı ayrı anlamak gereklidir.

Genellikle, firmware'i dump etmek için kullanılan komut:
```
md
```
Bu, "bellek dökümü" anlamına gelir. Bu işlem, belleği (EEPROM İçeriği) ekrana dökecektir. Bellek dökümünü yakalamak için işleme başlamadan önce Seri Konsol çıktısını günlüğe kaydetmeniz önerilir.

Son olarak, günlük dosyasından gereksiz tüm verileri çıkarın ve dosyayı `dosyaadı.rom` olarak depolayın ve içeriği çıkarmak için binwalk kullanın:
```
binwalk -e <filename.rom>
```
Bu, EEPROM'daki olası içerikleri, hex dosyasında bulunan imzalara göre listeleyecektir.

Ancak, kullanılsa bile her zaman uboot'un kilidinin açık olduğu durum olmayabilir. Enter tuşu bir şey yapmıyorsa, Boşluk Tuşu gibi farklı tuşları kontrol edin. Önyükleyicinin kilitli olduğu ve kesintiye uğramadığı durumlarda bu yöntem çalışmayabilir. Uboot'un cihaz için önyükleyici olup olmadığını kontrol etmek için, cihazın önyükleme sırasında UART Konsolu çıktısını kontrol edin. Önyükleme sırasında uboot'un bahsedilip edilmediğini kontrol edebilir.

### [WhiteIntel](https://whiteintel.io)

<figure><img src="../../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io), şirketin veya müşterilerinin **hırsız kötü amaçlı yazılımlar** tarafından **tehlikeye atılıp atılmadığını** kontrol etmek için **ücretsiz** işlevler sunan **dark-web** destekli bir arama motorudur.

WhiteIntel'in asıl amacı, bilgi çalan kötü amaçlı yazılımlardan kaynaklanan hesap ele geçirmeleri ve fidye yazılım saldırılarıyla mücadele etmektir.

Websitesini ziyaret edebilir ve **ücretsiz** olarak motorlarını deneyebilirsiniz:

{% embed url="https://whiteintel.io" %}

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert)</strong> ile sıfırdan kahramana kadar AWS hacklemeyi öğrenin</summary>

HackTricks'ı desteklemenin diğer yolları:

* **Şirketinizi HackTricks'te reklamını görmek istiyorsanız** veya **HackTricks'i PDF olarak indirmek istiyorsanız** [**ABONELİK PLANLARI**](https://github.com/sponsors/carlospolop)'na göz atın!
* [**Resmi PEASS & HackTricks ürünlerini**](https://peass.creator-spring.com) edinin
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)'yi keşfedin, özel [**NFT'lerimiz**](https://opensea.io/collection/the-peass-family) koleksiyonumuzu
* **💬 [Discord grubuna](https://discord.gg/hRep4RUj7f) veya [telegram grubuna](https://t.me/peass) katılın veya** bizi **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)'da takip edin.
* **Hacking püf noktalarınızı paylaşarak PR'lar göndererek** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github depolarına katkıda bulunun.

</details>
