# Analiza Wifi Pcap

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite svoju **kompaniju reklamiranu na HackTricks-u** ili da **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRIJAVU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**Porodicu PEASS**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikova slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Provera BSSID-ova

Kada primite snimak čiji je glavni saobraćaj Wifi koristeći WireShark, možete početi istraživati sve SSID-ove snimka sa _Wireless --> WLAN Traffic_:

![](<../../../.gitbook/assets/image (106).png>)

![](<../../../.gitbook/assets/image (492).png>)

### Brute Force

Jedna od kolona na tom ekranu pokazuje da li je **pronađena bilo kakva autentifikacija unutar pcap-a**. Ukoliko je to slučaj, možete pokušati da je probijete koristeći `aircrack-ng`:
```bash
aircrack-ng -w pwds-file.txt -b <BSSID> file.pcap
```
## Podaci u Bitovima / Bočni Kanal

Ako sumnjate da **podaci cure unutar bitova Wi-Fi mreže** možete proveriti bitove mreže koristeći filter poput sledećeg: `wlan contains <IMEmreže>`, ili `wlan.ssid == "IMEmreže"` pretražite filtrirane pakete za sumnjive niske.

## Pronalaženje Nepoznatih MAC Adresa u Wi-Fi Mreži

Sledeći link će biti koristan za pronalaženje **mašina koje šalju podatke unutar Wi-Fi Mreže**:

* `((wlan.ta == e8:de:27:16:70:c9) && !(wlan.fc == 0x8000)) && !(wlan.fc.type_subtype == 0x0005) && !(wlan.fc.type_subtype ==0x0004) && !(wlan.addr==ff:ff:ff:ff:ff:ff) && wlan.fc.type==2`

Ako već znate **MAC adrese, možete ih ukloniti iz rezultata** dodavanjem provera poput ove: `&& !(wlan.addr==5c:51:88:31:a0:3b)`

Kada otkrijete **nepoznate MAC** adrese koje komuniciraju unutar mreže, možete koristiti **filtere** poput sledećeg: `wlan.addr==<MAC adresa> && (ftp || http || ssh || telnet)` da filtrirate njen saobraćaj. Imajte na umu da su ftp/http/ssh/telnet filteri korisni ako ste dešifrovali saobraćaj.

## Dekriptovanje Saobraćaja

Edit --> Preferences --> Protocols --> IEEE 802.11--> Edit

![](<../../../.gitbook/assets/image (499).png>)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite svoju **kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** Proverite [**PLANOVE ZA PRIJAVU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
