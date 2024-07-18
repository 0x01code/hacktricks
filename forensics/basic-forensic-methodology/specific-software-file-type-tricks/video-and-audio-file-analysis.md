{% hint style="success" %}
AWS Hacking'i öğrenin ve uygulayın:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Eğitim AWS Kırmızı Takım Uzmanı (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP Hacking'i öğrenin ve uygulayın: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Eğitim GCP Kırmızı Takım Uzmanı (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks'i Destekleyin</summary>

* [**Abonelik planlarını**](https://github.com/sponsors/carlospolop) kontrol edin!
* 💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) katılın veya [**telegram grubuna**](https://t.me/peass) katılın veya bizi **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** takip edin.**
* **Hacking püf noktalarını göndererek HackTricks ve HackTricks Cloud** github depolarına PR gönderin.

</details>
{% endhint %}

**Ses ve video dosyası manipülasyonu**, **CTF adli bilişim zorlukları** için temel bir konudur, **steganografi** ve metaveri analizi kullanarak gizli mesajları gizlemek veya ortaya çıkarmak için. **[Mediainfo](https://mediaarea.net/en/MediaInfo)** ve **`exiftool`** gibi araçlar, dosya metaverisini incelemek ve içerik türlerini belirlemek için gereklidir.

Ses zorlukları için, **[Audacity](http://www.audacityteam.org/)**, ses içine kodlanmış metni ortaya çıkarmak için temel olan dalga formlarını görüntüleme ve spektrogramları analiz etme konusunda öne çıkar. Detaylı spektrogram analizi için **[Sonic Visualiser](http://www.sonicvisualiser.org/)** şiddetle tavsiye edilir. **Audacity**, gizli mesajları tespit etmek için parçaları yavaşlatma veya tersine çevirme gibi ses manipülasyonuna izin verir. Ses dosyalarını dönüştürme ve düzenleme konusunda üstün olan bir komut satırı yardımcı programı olan **[Sox](http://sox.sourceforge.net/)** bulunmaktadır.

**En Az Anlamlı Bitler (LSB)** manipülasyonu, ses ve video steganografisinde yaygın bir tekniktir, verileri gizlice gömmek için medya dosyalarının sabit boyutlu parçalarını kullanır. **[Multimon-ng](http://tools.kali.org/wireless-attacks/multimon-ng)**, **DTMF tonları** veya **Morse kodu** olarak gizlenmiş mesajları çözmek için kullanışlıdır.

Video zorlukları genellikle ses ve video akışlarını bir araya getiren konteyner formatlarını içerir. **[FFmpeg](http://ffmpeg.org/)**, bu formatları analiz etmek ve manipüle etmek için başvurulan bir araçtır, içeriği çözümlemek ve oynatmak için uygundur. Geliştiriciler için, **[ffmpy](http://ffmpy.readthedocs.io/en/latest/examples.html)**, FFmpeg'in yeteneklerini Python'a entegre ederek gelişmiş betiksel etkileşimler sağlar.

Bu araçlar yelpazesi, CTF zorluklarında gereken çok yönlülüğü vurgular, katılımcıların ses ve video dosyaları içindeki gizli verileri ortaya çıkarmak için geniş bir analiz ve manipülasyon teknikleri yelpazesi kullanmaları gerekmektedir.

## Referanslar
* [https://trailofbits.github.io/ctf/forensics/](https://trailofbits.github.io/ctf/forensics/)
{% hint style="success" %}
AWS Hacking'i öğrenin ve uygulayın:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Eğitim AWS Kırmızı Takım Uzmanı (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP Hacking'i öğrenin ve uygulayın: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Eğitim GCP Kırmızı Takım Uzmanı (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks'i Destekleyin</summary>

* [**Abonelik planlarını**](https://github.com/sponsors/carlospolop) kontrol edin!
* 💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) katılın veya [**telegram grubuna**](https://t.me/peass) katılın veya bizi **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** takip edin.**
* **Hacking püf noktalarını göndererek HackTricks ve HackTricks Cloud** github depolarına PR gönderin.

</details>
{% endhint %}
