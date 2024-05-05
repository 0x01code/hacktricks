# Napad produžetka dužine heša

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite svoju **kompaniju reklamiranu na HackTricks-u** ili da **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRIJAVU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks suvenir**](https://peass.creator-spring.com)
* Otkrijte [**Porodičnu PEASS**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

#### [WhiteIntel](https://whiteintel.io)

<figure><img src="../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) je pretraživač pokretan **dark web-om** koji nudi **besplatne** funkcionalnosti za proveru da li je kompanija ili njeni korisnici **kompromitovani** od strane **malvera za krađu**.

Primarni cilj WhiteIntel-a je borba protiv preuzimanja naloga i napada ransomware-a koji proizilaze iz malvera za krađu informacija.

Možete posetiti njihovu veb lokaciju i isprobati njihovu mašinu za **besplatno** na:

{% embed url="https://whiteintel.io" %}

***

## Rezime napada

Zamislite server koji **potpisuje** neke **podatke** tako što **dodaje** **tajnu vrednost** nekim poznatim čistim tekstualnim podacima, a zatim hešira te podatke. Ako znate:

* **Dužinu tajne** (ovo takođe može biti probijeno iz datog opsega dužine)
* **Čiste tekstualne podatke**
* **Algoritam (i ranjiv je na ovaj napad)**
* **Padding je poznat**
* Obično se koristi podrazumevani, pa ako su ispunjeni i ostali zahtevi, i ovaj je
* Padding varira u zavisnosti od dužine tajne+podataka, zbog čega je potrebna dužina tajne

Onda je moguće da **napadač** **doda** **podatke** i **generiše** validan **potpis** za **prethodne podatke + dodate podatke**.

### Kako?

Osnovno, ranjivi algoritmi generišu hešove tako što prvo **heširaju blok podataka**, a zatim, **iz** **prethodno** kreiranog **heša** (stanja), dodaju **sledeći blok podataka** i **heširaju ga**.

Zatim, zamislite da je tajna "tajna" i podaci su "podaci", MD5 od "tajnapodaci" je 6036708eba0d11f6ef52ad44e8b74d5b.\
Ako napadač želi da doda string "dodatak" može:

* Generisati MD5 od 64 "A"
* Promeniti stanje prethodno inicijalizovanog heša u 6036708eba0d11f6ef52ad44e8b74d5b
* Dodati string "dodatak"
* Završiti heš i rezultujući heš će biti **validan za "tajna" + "podaci" + "padding" + "dodatak"**

### **Alat**

{% embed url="https://github.com/iagox86/hash_extender" %}

### Reference

Ovaj napad možete pronaći dobro objašnjen na [https://blog.skullsecurity.org/2012/everything-you-need-to-know-about-hash-length-extension-attacks](https://blog.skullsecurity.org/2012/everything-you-need-to-know-about-hash-length-extension-attacks)

#### [WhiteIntel](https://whiteintel.io)

<figure><img src="../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) je pretraživač pokretan **dark web-om** koji nudi **besplatne** funkcionalnosti za proveru da li je kompanija ili njeni korisnici **kompromitovani** od strane **malvera za krađu**.

Primarni cilj WhiteIntel-a je borba protiv preuzimanja naloga i napada ransomware-a koji proizilaze iz malvera za krađu informacija.

Možete posetiti njihovu veb lokaciju i isprobati njihovu mašinu za **besplatno** na:

{% embed url="https://whiteintel.io" %}

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite svoju **kompaniju reklamiranu na HackTricks-u** ili da **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRIJAVU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks suvenir**](https://peass.creator-spring.com)
* Otkrijte [**Porodičnu PEASS**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
