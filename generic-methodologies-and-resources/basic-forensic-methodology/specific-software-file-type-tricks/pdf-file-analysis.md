# Analiza PDF fajlova

<details>

<summary><strong>Naučite AWS hakovanje od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili da **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRIJAVU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

<figure><img src="../../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

\
Koristite [**Trickest**](https://trickest.com/?utm_source=hacktricks&utm_medium=text&utm_campaign=ppc&utm_term=trickest&utm_content=pdf-file-analysis) da biste lako kreirali i **automatizovali radne tokove** pokretane najnaprednijim alatima zajednice.\
Pristupite danas:

{% embed url="https://trickest.com/?utm_source=hacktricks&utm_medium=banner&utm_campaign=ppc&utm_content=pdf-file-analysis" %}

**Za dalje detalje pogledajte:** [**https://trailofbits.github.io/ctf/forensics/**](https://trailofbits.github.io/ctf/forensics/)

PDF format je poznat po svojoj složenosti i potencijalu za skrivanje podataka, čineći ga fokusom za CTF forenzičke izazove. Kombinuje elemente običnog teksta sa binarnim objektima, koji mogu biti kompresovani ili enkriptovani, i mogu uključivati skripte u jezicima poput JavaScript-a ili Flash-a. Da biste razumeli strukturu PDF-a, možete se pozvati na Didier Stevens-ov [uvodni materijal](https://blog.didierstevens.com/2008/04/09/quickpost-about-the-physical-and-logical-structure-of-pdf-files/), ili koristiti alate poput tekst editora ili PDF-specifičnog editora poput Origami-ja.

Za detaljno istraživanje ili manipulaciju PDF-ova, dostupni su alati poput [qpdf](https://github.com/qpdf/qpdf) i [Origami](https://github.com/mobmewireless/origami-pdf). Skriveni podaci unutar PDF-ova mogu biti prikriveni u:

* Nevidljivim slojevima
* XMP formatu metapodataka od strane Adobe-a
* Inkrementalnim generacijama
* Tekstu iste boje kao pozadina
* Tekstu iza slika ili preklapajućih slika
* Komentarima koji se ne prikazuju

Za prilagođenu analizu PDF-a, Python biblioteke poput [PeepDF](https://github.com/jesparza/peepdf) mogu se koristiti za izradu prilagođenih skriptova za parsiranje. Nadalje, potencijal PDF-a za skriveno skladištenje podataka je toliko velik da resursi poput NSA vodiča o rizicima i merama zaštite PDF-a, iako više nije smešten na originalnoj lokaciji, i dalje pružaju vredne uvide. [Kopija vodiča](http://www.itsecure.hu/library/file/Biztons%C3%A1gi%20%C3%BAtmutat%C3%B3k/Alkalmaz%C3%A1sok/Hidden%20Data%20and%20Metadata%20in%20Adobe%20PDF%20Files.pdf) i kolekcija [triokova u PDF formatu](https://github.com/corkami/docs/blob/master/PDF/PDF.md) od strane Ange Albertini-a mogu pružiti dodatno čitanje na ovu temu.

<details>

<summary><strong>Naučite AWS hakovanje od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili da **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRIJAVU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
