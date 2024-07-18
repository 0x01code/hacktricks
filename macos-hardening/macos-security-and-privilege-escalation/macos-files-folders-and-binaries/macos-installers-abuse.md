# Zloupotreba macOS instalatera

{% hint style="success" %}
Naučite i vežbajte hakovanje AWS-a:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Obuka AWS Crveni Tim Stručnjak (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Naučite i vežbajte hakovanje GCP-a: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Obuka GCP Crveni Tim Stručnjak (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Podržite HackTricks</summary>

* Proverite [**planove pretplate**](https://github.com/sponsors/carlospolop)!
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Podelite hakovanje trikova slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
{% endhint %}

## Osnovne informacije o Pkg-u

macOS **instalacioni paket** (poznat i kao `.pkg` fajl) je format fajla koji koristi macOS za **distribuciju softvera**. Ovi fajlovi su poput **kutije koja sadrži sve što je potrebno da bi se komad softvera** instalirao i pokrenuo ispravno.

Sam paket fajl je arhiva koja drži **hijerarhiju fajlova i direktorijuma koji će biti instalirani na ciljnom** računaru. Takođe može uključivati **skripte** za obavljanje zadataka pre i posle instalacije, poput postavljanja konfiguracionih fajlova ili čišćenja starih verzija softvera.

### Hijerarhija

<figure><img src="../../../.gitbook/assets/Pasted Graphic.png" alt="https://www.youtube.com/watch?v=iASSG0_zobQ"><figcaption></figcaption></figure>

* **Distribucija (xml)**: Prilagođavanja (naslov, tekst dobrodošlice...) i skripte/provere instalacije
* **PackageInfo (xml)**: Informacije, zahtevi za instalaciju, lokacija instalacije, putanje do skripti za pokretanje
* **Bilans materijala (bom)**: Lista fajlova za instalaciju, ažuriranje ili uklanjanje sa dozvolama za fajlove
* **Payload (CPIO arhiva gzip kompresovana)**: Fajlovi za instalaciju na `install-location` iz PackageInfo
* **Skripte (CPIO arhiva gzip kompresovana)**: Pre i post instalacione skripte i dodatni resursi izdvojeni u privremeni direktorijum za izvršenje.

### Dekompresija
```bash
# Tool to directly get the files inside a package
pkgutil —expand "/path/to/package.pkg" "/path/to/out/dir"

# Get the files ina. more manual way
mkdir -p "/path/to/out/dir"
cd "/path/to/out/dir"
xar -xf "/path/to/package.pkg"

# Decompress also the CPIO gzip compressed ones
cat Scripts | gzip -dc | cpio -i
cpio -i < Scripts
```
## Osnovne informacije o DMG-u

DMG datoteke, ili Apple Disk Images, su format datoteka koji koristi macOS kompanije Apple za disk slike. DMG datoteka je suštinski **montabilna disk slika** (sadrži sopstveni fajl sistem) koja sadrži sirove blok podatke obično kompresovane i ponekad enkriptovane. Kada otvorite DMG datoteku, macOS je **montira kao da je fizički disk**, omogućavajući vam pristup njenom sadržaju.

{% hint style="danger" %}
Imajte na umu da **`.dmg`** instalateri podržavaju **toliko formata** da su u prošlosti neki od njih koji su sadržali ranjivosti zloupotrebljeni kako bi se dobio **izvršni kod kernela**.
{% endhint %}

### Hijerarhija

<figure><img src="../../../.gitbook/assets/image (225).png" alt=""><figcaption></figcaption></figure>

Hijerarhija DMG datoteke može biti različita u zavisnosti od sadržaja. Međutim, za aplikativne DMG-ove, obično prati ovu strukturu:

- Gornji nivo: Ovo je koren disk slike. Često sadrži aplikaciju i možda link ka fascikli Aplikacije.
- Aplikacija (.app): Ovo je stvarna aplikacija. U macOS-u, aplikacija je obično paket koji sadrži mnogo pojedinačnih fajlova i fascikli koje čine aplikaciju.
- Link ka Aplikacijama: Ovo je prečica do fascikle Aplikacije u macOS-u. Svrha ovoga je da vam olakša instalaciju aplikacije. Možete prevući .app fajl na ovu prečicu da instalirate aplikaciju.

## Eskalacija privilegija putem zloupotrebe pkg-a

### Izvršavanje iz javnih direktorijuma

Ako se skript za pre ili post instalaciju na primer izvršava iz **`/var/tmp/Installerutil`**, i napadač može kontrolisati tu skriptu kako bi eskalirao privilegije svaki put kada se izvrši. Ili još jedan sličan primer:

<figure><img src="../../../.gitbook/assets/Pasted Graphic 5.png" alt="https://www.youtube.com/watch?v=iASSG0_zobQ"><figcaption><p><a href="https://www.youtube.com/watch?v=kCXhIYtODBg">https://www.youtube.com/watch?v=kCXhIYtODBg</a></p></figcaption></figure>

### AuthorizationExecuteWithPrivileges

Ovo je [javna funkcija](https://developer.apple.com/documentation/security/1540038-authorizationexecutewithprivileg) koju će nekoliko instalatera i ažuriranja pozvati da **izvrše nešto kao root**. Ova funkcija prihvata **putanju** **fajla** koji se **izvršava** kao parametar, međutim, ako napadač može **modifikovati** ovaj fajl, biće u mogućnosti da **zloupotrebi** njegovo izvršavanje sa root privilegijama kako bi **eskaliro privilegije**.
```bash
# Breakpoint in the function to check wich file is loaded
(lldb) b AuthorizationExecuteWithPrivileges
# You could also check FS events to find this missconfig
```
### Izvršenje putem montiranja

Ako instalater piše u `/tmp/fixedname/bla/bla`, moguće je **napraviti montiranje** preko `/tmp/fixedname` bez vlasnika tako da možete **izmeniti bilo koji fajl tokom instalacije** kako biste zloupotrebili proces instalacije.

Primer za ovo je **CVE-2021-26089** koji je uspeo da **prepiše periodični skript** kako bi dobio izvršenje kao root. Za više informacija pogledajte ovaj talk: [**OBTS v4.0: "Mount(ain) of Bugs" - Csaba Fitzl**](https://www.youtube.com/watch?v=jSYPazD4VcE)

## pkg kao zlonamerni softver

### Prazan Payload

Moguće je jednostavno generisati **`.pkg`** fajl sa **pre i post-install skriptama** bez ikakvog payload-a.

### JS u Distribution xml

Moguće je dodati **`<script>`** tagove u **distribution xml** fajl paketa i taj kod će biti izvršen i može **izvršiti komande** koristeći **`system.run`**:

<figure><img src="../../../.gitbook/assets/image (1043).png" alt=""><figcaption></figcaption></figure>

## Reference

* [**DEF CON 27 - Unpacking Pkgs A Look Inside Macos Installer Packages And Common Security Flaws**](https://www.youtube.com/watch?v=iASSG0\_zobQ)
* [**OBTS v4.0: "The Wild World of macOS Installers" - Tony Lambert**](https://www.youtube.com/watch?v=Eow5uNHtmIg)
* [**DEF CON 27 - Unpacking Pkgs A Look Inside MacOS Installer Packages**](https://www.youtube.com/watch?v=kCXhIYtODBg)
