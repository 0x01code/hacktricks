# FZ - 125kHz RFID

<details>

<summary><strong>AWS hacklemeyi sıfırdan kahramana öğrenin</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Kırmızı Takım Uzmanı)</strong></a><strong> ile!</strong></summary>

HackTricks'ı desteklemenin diğer yolları:

* **Şirketinizi HackTricks'te reklamınızı görmek istiyorsanız** veya **HackTricks'i PDF olarak indirmek istiyorsanız** [**ABONELİK PLANLARINI**](https://github.com/sponsors/carlospolop) kontrol edin!
* [**Resmi PEASS & HackTricks ürünlerini**](https://peass.creator-spring.com) edinin
* [**PEASS Ailesi'ni**](https://opensea.io/collection/the-peass-family) keşfedin, özel [**NFT'lerimiz**](https://opensea.io/collection/the-peass-family) koleksiyonumuz
* **Katılın** 💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) veya bizi **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)** takip edin.**
* **Hacking püf noktalarınızı göndererek HackTricks** [**ve HackTricks Cloud**](https://github.com/carlospolop/hacktricks) github depolarına PR gönderin.

</details>

## Giriş

125kHz etiketlerin nasıl çalıştığı hakkında daha fazla bilgi için şuna bakın:

{% content-ref url="../pentesting-rfid.md" %}
[pentesting-rfid.md](../pentesting-rfid.md)
{% endcontent-ref %}

## Eylemler

Bu tür etiketler hakkında daha fazla bilgi için [**bu girişe**](../pentesting-rfid.md#low-frequency-rfid-tags-125khz) bakın.

### Oku

Kart bilgilerini **okumaya** çalışır. Sonra onları **taklit** edebilir.

{% hint style="warning" %}
Bazı interkomlar, anahtar kopyalamayı önlemeye çalışarak okumadan önce yazma komutu gönderir. Yazma başarılı olursa, o etiket sahte olarak kabul edilir. Flipper RFID taklit ettiğinde, okuyucunun orijinalinden ayırt etme şansı olmadığından, bu tür sorunlar ortaya çıkmaz.
{% endhint %}

### El ile Ekle

Flipper Zero'da **manuel olarak verileri belirterek sahte kartlar oluşturabilir** ve ardından bunları taklit edebilirsiniz.

#### Kartlardaki Kimlikler

Bazı durumlarda, bir kart aldığınızda, kartın kimliğini (veya bir kısmını) kartın üzerinde yazılı olarak bulabilirsiniz.

* **EM Marin**

Örneğin, bu EM-Marin kartında fiziksel kartta **son 3'ten 5 baytı açıkça okumak mümkündür**.\
Diğer 2'si karttan okuyamıyorsanız, brute-force yöntemiyle bulunabilir.

<figure><img src="../../../.gitbook/assets/image (101).png" alt=""><figcaption></figcaption></figure>

* **HID**

Aynı durum bu HID kartında da geçerlidir, burada sadece 3 bayttan 2'si kartta yazılı olarak bulunabilir

<figure><img src="../../../.gitbook/assets/image (1011).png" alt=""><figcaption></figcaption></figure>

### Taklit Et/Yaz

Bir kartı **kopyaladıktan** veya **manuel olarak** kimliği **girdikten** sonra, Flipper Zero ile bunu **taklit etmek** veya gerçek bir karta **yazmak** mümkündür.

## Referanslar

* [https://blog.flipperzero.one/rfid/](https://blog.flipperzero.one/rfid/)

<details>

<summary><strong>AWS hacklemeyi sıfırdan kahramana öğrenin</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Kırmızı Takım Uzmanı)</strong></a><strong> ile!</strong></summary>

HackTricks'ı desteklemenin diğer yolları:

* **Şirketinizi HackTricks'te reklamınızı görmek istiyorsanız** veya **HackTricks'i PDF olarak indirmek istiyorsanız** [**ABONELİK PLANLARINI**](https://github.com/sponsors/carlospolop) kontrol edin!
* [**Resmi PEASS & HackTricks ürünlerini**](https://peass.creator-spring.com) edinin
* [**PEASS Ailesi'ni**](https://opensea.io/collection/the-peass-family) keşfedin, özel [**NFT'lerimiz**](https://opensea.io/collection/the-peass-family) koleksiyonumuz
* **Katılın** 💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) veya bizi **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)** takip edin.**
* **Hacking püf noktalarınızı göndererek HackTricks** [**ve HackTricks Cloud**](https://github.com/carlospolop/hacktricks) github depolarına PR gönderin.

</details>
