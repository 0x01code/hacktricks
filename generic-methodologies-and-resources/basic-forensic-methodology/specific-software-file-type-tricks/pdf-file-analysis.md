# PDF Dosyası Analizi

<details>

<summary><strong>Sıfırdan kahramana AWS hackleme öğrenin</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks'ı desteklemenin diğer yolları:

* **Şirketinizi HackTricks'te reklamını görmek istiyorsanız** veya **HackTricks'i PDF olarak indirmek istiyorsanız** [**ABONELİK PLANLARINI**](https://github.com/sponsors/carlospolop) kontrol edin!
* [**Resmi PEASS & HackTricks ürünlerini alın**](https://peass.creator-spring.com)
* [**The PEASS Ailesi'ni**](https://opensea.io/collection/the-peass-family) keşfedin, özel [**NFT'lerimiz**](https://opensea.io/collection/the-peass-family) koleksiyonumuz
* **Katılın** 💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) veya bizi **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**'da takip edin.**
* **Hacking püf noktalarınızı göndererek HackTricks** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github depolarına PR gönderin.

</details>

<figure><img src="../../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) kullanarak dünyanın **en gelişmiş topluluk araçları** tarafından desteklenen **iş akışlarını kolayca oluşturun ve otomatikleştirin**.\
Bugün Erişim Alın:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

**Daha fazla ayrıntı için kontrol edin:** [**https://trailofbits.github.io/ctf/forensics/**](https://trailofbits.github.io/ctf/forensics/)

PDF formatı, verileri gizleme potansiyeli ve karmaşıklığı ile bilinir, bu da onu CTF adli bilişim zorluklarının odak noktası haline getirir. Basit metin öğelerini sıkıştırılmış veya şifrelenmiş olabilecek ikili nesnelerle birleştirir ve JavaScript veya Flash gibi dillerde betikler içerebilir. PDF yapısını anlamak için Didier Stevens'ın [giriş materyallerine](https://blog.didierstevens.com/2008/04/09/quickpost-about-the-physical-and-logical-structure-of-pdf-files/) başvurulabilir veya bir metin düzenleyici veya Origami gibi PDF'ye özgü bir düzenleyici gibi araçlar kullanılabilir.

PDF'lerin derinlemesine keşfi veya manipülasyonu için [qpdf](https://github.com/qpdf/qpdf) ve [Origami](https://github.com/mobmewireless/origami-pdf) gibi araçlar mevcuttur. PDF'lerdeki gizli veriler şunlarda gizlenebilir:

* Görünmez katmanlar
* Adobe'nin XMP meta veri formatı
* Artımlı nesiller
* Arka planla aynı renkteki metin
* Resimlerin arkasındaki veya resimlerin üzerine binen metin
* Görüntülenmeyen yorumlar

Özel PDF analizi için [PeepDF](https://github.com/jesparza/peepdf) gibi Python kütüphaneleri, özel ayrıştırma betikleri oluşturmak için kullanılabilir. Ayrıca, PDF'nin gizli veri depolama potansiyeli o kadar geniştir ki, NSA'nın PDF riskleri ve karşı önlemler hakkındaki kılavuzu gibi kaynaklar, orijinal konumunda artık barındırılmıyor olsa da hala değerli içgörüler sunmaktadır. Bir [kılavuz kopyası](http://www.itsecure.hu/library/file/Biztons%C3%A1gi%20%C3%BAtmutat%C3%B3k/Alkalmaz%C3%A1sok/Hidden%20Data%20and%20Metadata%20in%20Adobe%20PDF%20Files.pdf) ve Ange Albertini'nin PDF formatı hakkında daha fazla okuma sağlayan bir [PDF formatı püf noktaları koleksiyonu](https://github.com/corkami/docs/blob/master/PDF/PDF.md) sunabilir. 

<details>

<summary><strong>Sıfırdan kahramana AWS hackleme öğrenin</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks'ı desteklemenin diğer yolları:

* **Şirketinizi HackTricks'te reklamını görmek istiyorsanız** veya **HackTricks'i PDF olarak indirmek istiyorsanız** [**ABONELİK PLANLARINI**](https://github.com/sponsors/carlospolop) kontrol edin!
* [**Resmi PEASS & HackTricks ürünlerini alın**](https://peass.creator-spring.com)
* [**The PEASS Ailesi'ni**](https://opensea.io/collection/the-peass-family) keşfedin, özel [**NFT'lerimiz**](https://opensea.io/collection/the-peass-family) koleksiyonumuz
* **Katılın** 💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) veya bizi **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**'da takip edin.**
* **Hacking püf noktalarınızı göndererek HackTricks** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github depolarına PR gönderin.

</details>
