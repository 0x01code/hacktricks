# Wifi Pcap Analizi

<details>

<summary><strong>Sıfırdan kahraman olmaya kadar AWS hackleme öğrenin</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Kırmızı Takım Uzmanı)</strong></a><strong>!</strong></summary>

HackTricks'i desteklemenin diğer yolları:

* **Şirketinizi HackTricks'te reklamını görmek istiyorsanız** veya **HackTricks'i PDF olarak indirmek istiyorsanız** [**ABONELİK PLANLARI**]'na göz atın (https://github.com/sponsors/carlospolop)!
* [**Resmi PEASS & HackTricks ürünlerini**](https://peass.creator-spring.com) edinin
* [**The PEASS Ailesi**]'ni (https://opensea.io/collection/the-peass-family) keşfedin, özel [**NFT'ler**]'imiz (https://opensea.io/collection/the-peass-family) koleksiyonumuzu
* **Katılın** 💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) veya bizi **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**'da takip edin.**
* **Hacking püf noktalarınızı paylaşarak PR'lar göndererek** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github depolarına.

</details>

## BSSID'leri Kontrol Edin

WireShark kullanarak Wifi trafiğinin ağırlıklı olduğu bir yakalama aldığınızda, yakalamadaki tüm SSID'leri incelemeye başlayabilirsiniz _Wireless --> WLAN Traffic_:

![](<../../../.gitbook/assets/image (103).png>)

![](<../../../.gitbook/assets/image (489).png>)

### Kaba Kuvvet

Bu ekranın sütunlarından biri, **pcap içinde herhangi bir kimlik doğrulamasının bulunup bulunmadığını** gösterir. Eğer durum buysa, `aircrack-ng` kullanarak bunu kaba kuvvet saldırısıyla deneyebilirsiniz:
```bash
aircrack-ng -w pwds-file.txt -b <BSSID> file.pcap
```
## Veri Paketlerinde / Yan Kanalda

Eğer **verilerin bir Wifi ağı beacons'larında sızdırıldığını** şüpheleniyorsanız, ağın beacons'larını aşağıdaki gibi bir filtre kullanarak kontrol edebilirsiniz: `wlan contains <AĞINADI>`, veya `wlan.ssid == "AĞINADI"` filtrelenmiş paketler içinde şüpheli dizeler arayın.

## Bir Wifi Ağındaki Bilinmeyen MAC Adreslerini Bulma

Aşağıdaki bağlantı **bir Wifi Ağı içinde veri gönderen makineleri** bulmak için faydalı olacaktır:

* `((wlan.ta == e8:de:27:16:70:c9) && !(wlan.fc == 0x8000)) && !(wlan.fc.type_subtype == 0x0005) && !(wlan.fc.type_subtype ==0x0004) && !(wlan.addr==ff:ff:ff:ff:ff:ff) && wlan.fc.type==2`

Eğer zaten **MAC adreslerini biliyorsanız, çıktıdan onları çıkarabilirsiniz** ve şöyle bir kontrol ekleyebilirsiniz: `&& !(wlan.addr==5c:51:88:31:a0:3b)`

Bir kez ağ içinde iletişim kuran **bilinmeyen MAC** adreslerini tespit ettikten sonra, şu gibi **filtreler** kullanabilirsiniz: `wlan.addr==<MAC adresi> && (ftp || http || ssh || telnet)` trafiğini filtrelemek için. Ftp/http/ssh/telnet filtrelerinin trafiği şifrelediyseniz faydalı olduğunu unutmayın.

## Trafik Şifrelemesi

Düzenle --> Tercihler --> Protokoller --> IEEE 802.11--> Düzenle

![](<../../../.gitbook/assets/image (496).png>)

<details>

<summary><strong>Sıfırdan kahraman olmak için AWS hackleme öğrenin</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks'i desteklemenin diğer yolları:

* **Şirketinizi HackTricks'te reklamını görmek istiyorsanız** veya **HackTricks'i PDF olarak indirmek istiyorsanız** [**ABONELİK PLANLARINI**](https://github.com/sponsors/carlospolop) kontrol edin!
* [**Resmi PEASS & HackTricks ürünlerini**](https://peass.creator-spring.com) edinin
* [**The PEASS Family'yi**](https://opensea.io/collection/the-peass-family) keşfedin, özel [**NFT'lerimiz**](https://opensea.io/collection/the-peass-family) koleksiyonumuzu
* **💬 [Discord grubuna](https://discord.gg/hRep4RUj7f) veya [telegram grubuna](https://t.me/peass) katılın veya** bizi **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**'da takip edin.**
* **Hacking püf noktalarınızı paylaşarak PR göndererek HackTricks** ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github depolarına katkıda bulunun.

</details>
