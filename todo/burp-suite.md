<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili da **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRIJATELJSTVO**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>


# Osnovni Payload-ovi

* **Jednostavna Lista:** Samo lista koja sadrži unos u svakom redu
* **Runtime Fajl:** Lista čitana u vreme izvršavanja (ne učitana u memoriju). Za podršku velikim listama.
* **Modifikacija slučaja:** Primenite neke promene na listu stringova (Bez promene, u mala slova, u VELIKA SLOVA, u Pravo ime - Prvo veliko slovo i ostalo malim slovima-, u Pravo Ime - Prvo veliko slovo i ostalo ostaje isto-.
* **Brojevi:** Generišite brojeve od X do Y koristeći korak Z ili nasumično.
* **Brute Forcer:** Set karaktera, minimalna i maksimalna dužina.

[https://github.com/0xC01DF00D/Collabfiltrator](https://github.com/0xC01DF00D/Collabfiltrator) : Payload za izvršavanje komandi i preuzimanje izlaza putem DNS zahteva ka burpcollab.

{% embed url="https://medium.com/@ArtsSEC/burp-suite-exporter-462531be24e" %}

[https://github.com/h3xstream/http-script-generator](https://github.com/h3xstream/http-script-generator)
