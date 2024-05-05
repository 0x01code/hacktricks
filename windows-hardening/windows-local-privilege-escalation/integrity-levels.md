# Nivoi integriteta

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite svoju **kompaniju reklamiranu na HackTricks-u** ili da **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRETPLATU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**Porodicu PEASS**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

### [WhiteIntel](https://whiteintel.io)

<figure><img src="../../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) je pretraživač pokretan **dark-web-om** koji nudi **besplatne** funkcionalnosti za proveru da li je kompanija ili njeni korisnici **kompromitovani** od strane **malvera za krađu podataka**.

Primarni cilj WhiteIntel-a je borba protiv preuzimanja naloga i napada ransomware-a koji proizilaze iz malvera za krađu informacija.

Možete posetiti njihovu veb stranicu i isprobati njihovu mašinu za **besplatno** na:

{% embed url="https://whiteintel.io" %}

***

## Nivoi integriteta

U Windows Vista i kasnijim verzijama, svi zaštićeni elementi dolaze sa oznakom **nivoa integriteta**. Ova postavka uglavnom dodeljuje "srednji" nivo integriteta fajlovima i ključevima registra, osim određenih foldera i fajlova na koje Internet Explorer 7 može da piše na niskom nivou integriteta. Podrazumevano ponašanje je da procesi pokrenuti od strane standardnih korisnika imaju srednji nivo integriteta, dok usluge obično funkcionišu na nivou sistema integriteta. Etiketa visokog integriteta štiti korenski direktorijum.

Ključno pravilo je da objekte ne mogu menjati procesi sa nižim nivoom integriteta od nivoa objekta. Nivoi integriteta su:

* **Nepoverljivo**: Ovaj nivo je za procese sa anonimnim prijavama. %%%Primer: Chrome%%%
* **Nizak**: Glavno za interakcije na internetu, posebno u Internet Explorer-ovom Protected Mode-u, utičući na povezane fajlove i procese, i određene foldere poput **Privremenog internet foldera**. Procesi niskog integriteta suočavaju se sa značajnim ograničenjima, uključujući nedostatak pristupa pisanju u registar i ograničen pristup profilu korisnika.
* **Srednji**: Podrazumevani nivo za većinu aktivnosti, dodeljen standardnim korisnicima i objektima bez specifičnih nivoa integriteta. Čak i članovi grupe Administratori funkcionišu na ovom nivou podrazumevano.
* **Visok**: Rezervisan za administratore, omogućavajući im da menjaju objekte na nižim nivoima integriteta, uključujući one na visokom nivou samom.
* **Sistem**: Najviši operativni nivo za Windows kernel i osnovne usluge, nedostupan čak i za administratore, obezbeđujući zaštitu vitalnih sistemskih funkcija.
* **Instalater**: Jedinstveni nivo koji stoji iznad svih ostalih, omogućavajući objektima na ovom nivou da deinstaliraju bilo koji drugi objekat.

Možete dobiti nivo integriteta procesa koristeći **Process Explorer** iz **Sysinternals-a**, pristupajući **svojstvima** procesa i pregledajući "**Bezbednost**" karticu:

![](<../../.gitbook/assets/image (824).png>)

Takođe možete dobiti svoj **trenutni nivo integriteta** koristeći `whoami /groups`

![](<../../.gitbook/assets/image (325).png>)

### Nivoi integriteta u fajl-sistemu

Objekat unutar fajl-sistema može zahtevati **minimalni zahtev za nivoom integriteta** i ako proces nema ovaj integritet, neće moći da interaguje sa njim.\
Na primer, dozvolimo **kreiranje običnog fajla iz konzole običnog korisnika i proverimo dozvole**:
```
echo asd >asd.txt
icacls asd.txt
asd.txt BUILTIN\Administrators:(I)(F)
DESKTOP-IDJHTKP\user:(I)(F)
NT AUTHORITY\SYSTEM:(I)(F)
NT AUTHORITY\INTERACTIVE:(I)(M,DC)
NT AUTHORITY\SERVICE:(I)(M,DC)
NT AUTHORITY\BATCH:(I)(M,DC)
```
Sada dodelimo minimalni nivo integriteta **Visok** datoteci. Ovo **mora biti urađeno iz konzole** koja se izvršava kao **administrator**, jer će **obična konzola** biti pokrenuta na srednjem nivou integriteta i **neće biti dozvoljeno** dodeljivanje Visokog nivoa integriteta objektu:
```
icacls asd.txt /setintegritylevel(oi)(ci) High
processed file: asd.txt
Successfully processed 1 files; Failed processing 0 files

C:\Users\Public>icacls asd.txt
asd.txt BUILTIN\Administrators:(I)(F)
DESKTOP-IDJHTKP\user:(I)(F)
NT AUTHORITY\SYSTEM:(I)(F)
NT AUTHORITY\INTERACTIVE:(I)(M,DC)
NT AUTHORITY\SERVICE:(I)(M,DC)
NT AUTHORITY\BATCH:(I)(M,DC)
Mandatory Label\High Mandatory Level:(NW)
```
Ovde stvari postaju interesantne. Možete videti da korisnik `DESKTOP-IDJHTKP\user` ima **PUNE privilegije** nad datotekom (zaista je ovaj korisnik kreirao datoteku), međutim, zbog implementiranog minimalnog nivoa integriteta neće moći više da menja datoteku osim ako radi unutar Nivoa visokog integriteta (napomena: moći će da je čita):
```
echo 1234 > asd.txt
Access is denied.

del asd.txt
C:\Users\Public\asd.txt
Access is denied.
```
{% hint style="info" %}
**Dakle, kada datoteka ima minimalni nivo integriteta, da biste je izmenili, morate je pokrenuti barem na tom nivou integriteta.**
{% endhint %}

### Nivoi Integriteta u Binarnim Datotekama

Napravio sam kopiju `cmd.exe` u `C:\Windows\System32\cmd-low.exe` i postavio sam joj **nivo integriteta na niski iz administratorske konzole:**
```
icacls C:\Windows\System32\cmd-low.exe
C:\Windows\System32\cmd-low.exe NT AUTHORITY\SYSTEM:(I)(F)
BUILTIN\Administrators:(I)(F)
BUILTIN\Users:(I)(RX)
APPLICATION PACKAGE AUTHORITY\ALL APPLICATION PACKAGES:(I)(RX)
APPLICATION PACKAGE AUTHORITY\ALL RESTRICTED APP PACKAGES:(I)(RX)
Mandatory Label\Low Mandatory Level:(NW)
```
Sada, kada pokrenem `cmd-low.exe`, **pokrenuće se sa niskim nivoom integriteta** umesto srednjeg:

![](<../../.gitbook/assets/image (313).png>)

Za radoznale ljude, ako dodelite visok nivo integriteta binarnom fajlu (`icacls C:\Windows\System32\cmd-high.exe /setintegritylevel high`), neće se automatski pokrenuti sa visokim nivoom integriteta (ako ga pozovete sa srednjeg nivoa integriteta --podrazumevano-- pokrenuće se sa srednjim nivoom integriteta).

### Nivoi Integriteta u Procesima

Nisu svi fajlovi i folderi sa minimalnim nivoom integriteta, **ali svi procesi se izvršavaju pod određenim nivoom integriteta**. I slično kao što se desilo sa fajl-sistemom, **ako proces želi da piše unutar drugog procesa, mora imati barem isti nivo integriteta**. To znači da proces sa niskim nivoom integriteta ne može otvoriti ručku sa punim pristupom procesu sa srednjim nivoom integriteta.

Zbog ograničenja navedenih u ovoj i prethodnoj sekciji, sa aspekta bezbednosti, uvek je **preporučljivo pokrenuti proces sa što nižim nivoom integriteta**.

### [WhiteIntel](https://whiteintel.io)

<figure><img src="../../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) je pretraživač pokretan **dark-webom** koji nudi **besplatne** funkcionalnosti za proveru da li je kompanija ili njeni korisnici **napadnuti** od strane **malvera koji kradu informacije**.

Primarni cilj WhiteIntela je borba protiv preuzimanja naloga i napada ransomware-a koji proizilaze iz malvera koji kradu informacije.

Možete posetiti njihovu veb lokaciju i isprobati njihov pretraživač **besplatno** na:

{% embed url="https://whiteintel.io" %}

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite svoju **kompaniju reklamiranu na HackTricks-u** ili da **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA ČLANSTVO**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikova slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
