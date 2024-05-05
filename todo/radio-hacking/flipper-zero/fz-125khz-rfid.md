# FZ - 125kHz RFID

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** Proverite [**PLANOVE ZA PRIJAVU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

<figure><img src="https://pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}

## Uvod

Za više informacija o tome kako 125kHz oznake funkcionišu pogledajte:

{% content-ref url="../pentesting-rfid.md" %}
[pentesting-rfid.md](../pentesting-rfid.md)
{% endcontent-ref %}

## Akcije

Za više informacija o ovim tipovima oznaka [**pročitajte ovaj uvod**](../pentesting-rfid.md#low-frequency-rfid-tags-125khz).

### Čitanje

Pokušava da **pročita** informacije sa kartice. Zatim ih može **emulirati**.

{% hint style="warning" %}
Imajte na umu da neki interfoni pokušavaju da se zaštite od dupliciranja ključeva slanjem komande za pisanje pre čitanja. Ako pisanje uspe, ta oznaka se smatra lažnom. Kada Flipper emulira RFID, čitač nema načina da je razlikuje od originalne, tako da se takvi problemi ne javljaju.
{% endhint %}

### Dodavanje Ručno

Možete kreirati **lažne kartice u Flipper Zero-u navodeći podatke** ručno, a zatim ih emulirati.

#### ID-ovi na karticama

Ponekad, kada dobijete karticu, pronaći ćete ID (ili deo) napisan na kartici.

* **EM Marin**

Na primer, na ovoj EM-Marin kartici na fizičkoj kartici je moguće **pročitati poslednja 3 od 5 bajtova jasno**.\
Preostala 2 se mogu probati silom ako ih ne možete pročitati sa kartice.

<figure><img src="../../../.gitbook/assets/image (104).png" alt=""><figcaption></figcaption></figure>

* **HID**

Isto se dešava na ovoj HID kartici gde su samo 2 od 3 bajta vidljiva na kartici

<figure><img src="../../../.gitbook/assets/image (1014).png" alt=""><figcaption></figcaption></figure>

### Emuliranje/Pisanje

Nakon **kopiranja** kartice ili **ručnog unosa** ID-a, moguće je **emulirati** ga sa Flipper Zero-om ili ga **zapisati** na pravu karticu.

## Reference

* [https://blog.flipperzero.one/rfid/](https://blog.flipperzero.one/rfid/)

<figure><img src="https://pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** Proverite [**PLANOVE ZA PRIJAVU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
