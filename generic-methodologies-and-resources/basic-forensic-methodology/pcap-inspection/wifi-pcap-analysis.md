# Wifi Pcap Analise

<details>

<summary><strong>Leer AWS hakwerk vanaf nul tot held met</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Ander maniere om HackTricks te ondersteun:

* As jy wil sien dat jou **maatskappy geadverteer word in HackTricks** of **HackTricks aflaai in PDF-formaat** Kyk na die [**INSKRYWINGSPLANNE**](https://github.com/sponsors/carlospolop)!
* Kry die [**amptelike PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Ontdek [**Die PEASS Familie**](https://opensea.io/collection/the-peass-family), ons versameling van eksklusiewe [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Sluit aan by die** 💬 [**Discord groep**](https://discord.gg/hRep4RUj7f) of die [**telegram groep**](https://t.me/peass) of **volg** ons op **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Deel jou haktruuks deur PRs in te dien by die** [**HackTricks**](https://github.com/carlospolop/hacktricks) en [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github-opslag.

</details>

## Kontroleer BSSIDs

Wanneer jy 'n vangs ontvang waarvan die hoofverkeer Wifi gebruik met WireShark, kan jy begin om al die SSIDs van die vangs te ondersoek met _Wireless --> WLAN Traffic_:

![](<../../../.gitbook/assets/image (106).png>)

![](<../../../.gitbook/assets/image (492).png>)

### Brute Force

Een van die kolomme van daardie skerm dui aan of **enige verifikasie binne die pcap gevind is**. As dit die geval is, kan jy probeer om dit te Brute force met behulp van `aircrack-ng`:
```bash
aircrack-ng -w pwds-file.txt -b <BSSID> file.pcap
```
## Data in Beacons / Side Channel

Indien jy vermoed dat **data uitlek binne die beacons van 'n Wifi-netwerk**, kan jy die beacons van die netwerk ondersoek deur 'n filter soos die volgende te gebruik: `wlan contains <NAMEofNETWORK>`, of `wlan.ssid == "NAMEofNETWORK"` soek binne die gefiltreerde pakkette vir verdagte strings.

## Vind Onbekende MAC-adresse in 'n Wifi-netwerk

Die volgende skakel sal nuttig wees om die **toestelle wat data stuur binne 'n Wifi-netwerk** te vind:

* `((wlan.ta == e8:de:27:16:70:c9) && !(wlan.fc == 0x8000)) && !(wlan.fc.type_subtype == 0x0005) && !(wlan.fc.type_subtype ==0x0004) && !(wlan.addr==ff:ff:ff:ff:ff:ff) && wlan.fc.type==2`

As jy reeds die **MAC-adresse ken, kan jy hulle uit die uitset verwyder** deur kontroles by te voeg soos hierdie: `&& !(wlan.addr==5c:51:88:31:a0:3b)`

Sodra jy **onbekende MAC-adresse** wat binne die netwerk kommunikeer, opgespoor het, kan jy **filters** gebruik soos die volgende: `wlan.addr==<MAC-adres> && (ftp || http || ssh || telnet)` om sy verkeer te filtreer. Let daarop dat ftp/http/ssh/telnet-filters nuttig is as jy die verkeer ontsluit het.

## Ontsleutel Verkeer

Edit --> Voorkeure --> Protokolle --> IEEE 802.11--> Edit

![](<../../../.gitbook/assets/image (499).png>)

<details>

<summary><strong>Leer AWS-hacking vanaf nul tot held met</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Ander maniere om HackTricks te ondersteun:

* As jy wil sien dat jou **maatskappy geadverteer word in HackTricks** of **HackTricks aflaai in PDF-formaat** Kontroleer die [**INSKRYWINGSPLANNE**](https://github.com/sponsors/carlospolop)!
* Kry die [**amptelike PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Ontdek [**Die PEASS-familie**](https://opensea.io/collection/the-peass-family), ons versameling eksklusiewe [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Sluit aan by die** 💬 [**Discord-groep**](https://discord.gg/hRep4RUj7f) of die [**telegram-groep**](https://t.me/peass) of **volg** ons op **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Deel jou haktruuks deur PR's in te dien by die** [**HackTricks**](https://github.com/carlospolop/hacktricks) en [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github-opslag.

</details>
