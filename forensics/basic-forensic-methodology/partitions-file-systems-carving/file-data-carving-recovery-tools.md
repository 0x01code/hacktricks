# Alatke za izdvajanje podataka i oporavak

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite svoju **kompaniju reklamiranu na HackTricks-u** ili da **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRETPLATU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**Porodicu PEASS**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Alatke za izdvajanje podataka i oporavak

Više alatki na [https://github.com/Claudio-C/awesome-datarecovery](https://github.com/Claudio-C/awesome-datarecovery)

### Autopsy

Najčešće korišćeni alat u forenzici za izdvajanje fajlova iz slika je [**Autopsy**](https://www.autopsy.com/download/). Preuzmite ga, instalirajte ga i pustite da obradi fajl kako bi pronašao "skrivene" fajlove. Imajte na umu da je Autopsy napravljen da podržava disk slike i druge vrste slika, ali ne i obične fajlove.

### Binwalk <a href="#binwalk" id="binwalk"></a>

**Binwalk** je alat za analizu binarnih fajlova radi pronalaženja ugrađenog sadržaja. Može se instalirati putem `apt` i njegov izvorni kod se nalazi na [GitHub-u](https://github.com/ReFirmLabs/binwalk).

**Korisne komande**:
```bash
sudo apt install binwalk #Insllation
binwalk file #Displays the embedded data in the given file
binwalk -e file #Displays and extracts some files from the given file
binwalk --dd ".*" file #Displays and extracts all files from the given file
```
### Foremost

Još jedan čest alat za pronalaženje skrivenih datoteka je **foremost**. Konfiguracionu datoteku za foremost možete pronaći u `/etc/foremost.conf`. Ako želite da pretražujete samo određene datoteke, uklonite komentare iz njih. Ako ne uklonite komentare, foremost će pretraživati prema svojim podrazumevano konfigurisanim tipovima datoteka.
```bash
sudo apt-get install foremost
foremost -v -i file.img -o output
#Discovered files will appear inside the folder "output"
```
### **Skalpel**

**Skalpel** je još jedan alat koji se može koristiti za pronalaženje i izdvajanje **datoteka ugrađenih u datoteku**. U ovom slučaju, moraćete da uklonite komentare iz konfiguracione datoteke (_/etc/scalpel/scalpel.conf_) za vrste datoteka koje želite da izdvojite.
```bash
sudo apt-get install scalpel
scalpel file.img -o output
```
### Bulk Extractor

Ovaj alat dolazi unutar kali distribucije, ali ga možete pronaći ovde: [https://github.com/simsong/bulk\_extractor](https://github.com/simsong/bulk\_extractor)

Ovaj alat može skenirati sliku i **izdvojiti pcaps** unutar nje, **informacije o mreži (URL-ovi, domeni, IP adrese, MAC adrese, mejlovi)** i više **datoteka**. Samo treba da uradite:
```
bulk_extractor memory.img -o out_folder
```
### PhotoRec

Možete ga pronaći na [https://www.cgsecurity.org/wiki/TestDisk\_Download](https://www.cgsecurity.org/wiki/TestDisk\_Download)

Dolazi sa GUI i CLI verzijama. Možete odabrati **tipove datoteka** koje želite da PhotoRec pretražuje.

![](<../../../.gitbook/assets/image (524).png>)

### binvis

Proverite [kod](https://code.google.com/archive/p/binvis/) i [web stranicu alata](https://binvis.io/#/).

#### Funkcije BinVis

* Vizuelni i aktivni **pregled strukture**
* Više grafikona za različite fokusne tačke
* Fokusiranje na delove uzorka
* **Videti niske i resurse**, u PE ili ELF izvršnim datotekama npr.
* Dobijanje **obrazaca** za kriptoanalizu datoteka
* **Otkrivanje** algoritama pakovanja ili enkodiranja
* **Identifikacija** steganografije po obrascima
* **Vizuelno** binarno-diferenciranje

BinVis je odlično **polazište za upoznavanje sa nepoznatim ciljem** u scenariju crne kutije.

## Specifični alati za izvlačenje podataka

### FindAES

Pretražuje AES ključeve tražeći njihove rasporede ključeva. Može pronaći 128, 192 i 256 bitne ključeve, poput onih koje koriste TrueCrypt i BitLocker.

Preuzmite [ovde](https://sourceforge.net/projects/findaes/).

## Komplementarni alati

Možete koristiti [**viu** ](https://github.com/atanunq/viu) da vidite slike iz terminala.\
Možete koristiti linux alat komandne linije **pdftotext** da transformišete pdf u tekst i pročitate ga.
