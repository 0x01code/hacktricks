# Integriteitsvlakke

<details>

<summary><strong>Leer AWS-hacking vanaf nul tot held met</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Ander maniere om HackTricks te ondersteun:

* As jy wil sien dat jou **maatskappy geadverteer word in HackTricks** of **HackTricks aflaai in PDF-formaat** Kontroleer die [**INSKRYWINGSPLANNE**](https://github.com/sponsors/carlospolop)!
* Kry die [**amptelike PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Ontdek [**Die PEASS-familie**](https://opensea.io/collection/the-peass-family), ons versameling eksklusiewe [**NFT's**](https://opensea.io/collection/the-peass-family)
* **Sluit aan by die** 💬 [**Discord-groep**](https://discord.gg/hRep4RUj7f) of die [**telegram-groep**](https://t.me/peass) of **volg** ons op **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Deel jou haktruuks deur PR's in te dien by die** [**HackTricks**](https://github.com/carlospolop/hacktricks) en [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github-opslag.

</details>

## Integriteitsvlakke

In Windows Vista en latere weergawes het alle beskermde items 'n **integriteitsvlak**-etiket. Hierdie opstelling ken meestal 'n "medium" integriteitsvlak toe aan lêers en registerleë, behalwe vir sekere lêers en lêers waarop Internet Explorer 7 kan skryf op 'n lae integriteitsvlak. Die verstekgedrag is dat prosesse wat deur standaardgebruikers geïnisieer word, 'n medium integriteitsvlak het, terwyl dienste tipies op 'n stelselintegriteitsvlak werk. 'n Hoë-integriteitsmerk beskerm die hoofgids.

'n Belangrike reël is dat voorwerpe nie deur prosesse met 'n laer integriteitsvlak as die voorwerp se vlak gewysig kan word nie. Die integriteitsvlakke is:

* **Onbetroubaar**: Hierdie vlak is vir prosesse met anonieme aanmeldings. %%%Voorbeeld: Chrome%%%
* **Laag**: Hoofsaaklik vir internetinteraksies, veral in Internet Explorer se Beskermde Modus, wat geassosieerde lêers en prosesse beïnvloed, en sekere lêers soos die **Tydelike Internet-lêer**. Prosesse met 'n lae integriteit staar aansienlike beperkings in die gesig, insluitend geen register skryftoegang en beperkte gebruikersprofiel skryftoegang.
* **Medium**: Die verstekvlak vir die meeste aktiwiteite, toegewys aan standaardgebruikers en voorwerpe sonder spesifieke integriteitsvlakke. Selfs lede van die Administrateursgroep werk standaard op hierdie vlak.
* **Hoog**: Gereserveer vir administrateurs, wat hulle in staat stel om voorwerpe op laer integriteitsvlakke te wysig, insluitend dié op die hoë vlak self.
* **Stelsel**: Die hoogste operasionele vlak vir die Windows-kernel en kerndienste, buite bereik selfs vir administrateurs, wat verseker dat belangrike stelselfunksies beskerm word.
* **Installer**: 'n Unieke vlak wat bokant alle ander staan, wat voorwerpe op hierdie vlak in staat stel om enige ander voorwerp te deïnstalleer.

Jy kan die integriteitsvlak van 'n proses kry deur **Process Explorer** van **Sysinternals** te gebruik, deur die **eienskappe** van die proses te benader en die "**Sekuriteit**" -tabblad te besigtig:

![](<../../.gitbook/assets/image (821).png>)

Jy kan ook jou **huidige integriteitsvlak** kry deur `whoami /groups` te gebruik

![](<../../.gitbook/assets/image (322).png>)

### Integriteitsvlakke in Lêersisteem

'n Voorwerp binne die lêersisteem mag 'n **minimum integriteitsvlakvereiste** hê en as 'n proses nie hierdie integriteitsproses het nie, sal dit nie daarmee kan interaksie hê nie.\
Byvoorbeeld, laat ons **'n gewone lêer vanuit 'n gewone gebruikerskonsole skep en die regte nagaan**:
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
Nou, laat ons 'n minimum integriteitsvlak van **Hoog** aan die lêer toeken. Dit **moet gedoen word vanaf 'n konsole** wat as **administrateur** hardloop, aangesien 'n **gewone konsole** in Medium Integriteitsvlak sal hardloop en **nie toegelaat sal word** om 'n Hoë Integriteitsvlak aan 'n objek toe te ken nie:
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
Hier is waar dinge interessant raak. Jy kan sien dat die gebruiker `DESKTOP-IDJHTKP\user` **VOLLE voorregte** oor die lêer het (inderdaad, hierdie gebruiker het die lêer geskep), maar as gevolg van die minimum integriteitsvlak wat geïmplementeer is, sal hy nie die lêer kan wysig nie tensy hy binne 'n Hoë Integriteitsvlak hardloop (let wel dat hy dit steeds kan lees):
```
echo 1234 > asd.txt
Access is denied.

del asd.txt
C:\Users\Public\asd.txt
Access is denied.
```
{% hint style="info" %}
**Daarom, wanneer 'n lêer 'n minimumintegriteitsvlak het, moet jy ten minste op daardie integriteitsvlak hardloop om dit te kan wysig.**
{% endhint %}

### Integriteitsvlakke in Binêre lêers

Ek het 'n kopie van `cmd.exe` gemaak in `C:\Windows\System32\cmd-laag.exe` en dit 'n **integriteitsvlak van laag vanuit 'n administrateurskonsole ingestel:**
```
icacls C:\Windows\System32\cmd-low.exe
C:\Windows\System32\cmd-low.exe NT AUTHORITY\SYSTEM:(I)(F)
BUILTIN\Administrators:(I)(F)
BUILTIN\Users:(I)(RX)
APPLICATION PACKAGE AUTHORITY\ALL APPLICATION PACKAGES:(I)(RX)
APPLICATION PACKAGE AUTHORITY\ALL RESTRICTED APP PACKAGES:(I)(RX)
Mandatory Label\Low Mandatory Level:(NW)
```
Nou, wanneer ek `cmd-laag.exe` hardloop, sal dit **onder 'n lae-integriteitsvlak** hardloop in plaas van 'n medium een:

![](<../../.gitbook/assets/image (310).png>)

Vir nuuskierige mense, as jy 'n hoë integriteitsvlak toeken aan 'n binêre lêer (`icacls C:\Windows\System32\cmd-hoog.exe /setintegritylevel hoog`), sal dit nie outomaties met 'n hoë integriteitsvlak hardloop nie (as jy dit vanaf 'n medium integriteitsvlak aanroep - standaard - sal dit onder 'n medium integriteitsvlak hardloop).

### Integriteitsvlakke in Prosesse

Nie alle lêers en vouers het 'n minimum integriteitsvlak nie, **maar alle prosesse hardloop onder 'n integriteitsvlak**. En soortgelyk aan wat met die lêerstelsel gebeur het, **as 'n proses binne 'n ander proses wil skryf, moet dit ten minste dieselfde integriteitsvlak hê**. Dit beteken dat 'n proses met 'n lae integriteitsvlak nie 'n handvatsel met volle toegang tot 'n proses met 'n medium integriteitsvlak kan oopmaak nie.

As gevolg van die beperkings wat in hierdie en die vorige afdeling genoem is, word dit altyd **aanbeveel om 'n proses in die laerste moontlike integriteitsvlak te hardloop** vir 'n veiligheidsoogpunt.

<details>

<summary><strong>Leer AWS-hacking vanaf nul tot held met</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Ander maniere om HackTricks te ondersteun:

* As jy wil sien dat jou **maatskappy geadverteer word in HackTricks** of **HackTricks aflaai in PDF-formaat** Kyk na die [**INSKRYWINGSPLANNE**](https://github.com/sponsors/carlospolop)!
* Kry die [**amptelike PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Ontdek [**Die PEASS-familie**](https://opensea.io/collection/the-peass-family), ons versameling eksklusiewe [**NFT's**](https://opensea.io/collection/the-peass-family)
* **Sluit aan by die** 💬 [**Discord-groep**](https://discord.gg/hRep4RUj7f) of die [**telegram-groep**](https://t.me/peass) of **volg** ons op **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Deel jou haktruuks deur PR's in te dien by die** [**HackTricks**](https://github.com/carlospolop/hacktricks) en [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github-opslag.

</details>
