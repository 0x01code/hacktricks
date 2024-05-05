# macOS SIP

<details>

<summary><strong>Leer AWS-hacking vanaf nul tot held met</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Ander maniere om HackTricks te ondersteun:

* As jy jou **maatskappy geadverteer wil sien in HackTricks** of **HackTricks in PDF wil aflaai** Kyk na die [**INSKRYWINGSPLANNE**](https://github.com/sponsors/carlospolop)!
* Kry die [**amptelike PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Ontdek [**Die PEASS Familie**](https://opensea.io/collection/the-peass-family), ons versameling eksklusiewe [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Sluit aan by die** 💬 [**Discord-groep**](https://discord.gg/hRep4RUj7f) of die [**telegram-groep**](https://t.me/peass) of **volg** ons op **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Deel jou haktruuks deur PR's in te dien by die** [**HackTricks**](https://github.com/carlospolop/hacktricks) en [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github-opslag.

</details>

### [WhiteIntel](https://whiteintel.io)

<figure><img src="../../../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) is 'n **dark-web** aangedrewe soekenjin wat **gratis** funksies bied om te kontroleer of 'n maatskappy of sy kliënte deur **steelmalware** gekompromitteer is.

Die primêre doel van WhiteIntel is om rekening-oorneeminge en lospryse-aanvalle te bekamp wat voortspruit uit inligtingsteelmalware.

Jy kan hul webwerf besoek en hul enjin **gratis** probeer by:

{% embed url="https://whiteintel.io" %}

***

## **Basiese Inligting**

**System Integrity Protection (SIP)** in macOS is 'n meganisme wat ontwerp is om selfs die mees bevoorregte gebruikers te verhoed om ongemagtigde veranderinge aan sleutelstelselvelde te maak. Hierdie kenmerk speel 'n kritieke rol in die handhawing van die integriteit van die stelsel deur aksies soos die byvoeging, wysiging of verwydering van lêers in beskermde areas te beperk. Die primêre velde wat deur SIP beskerm word, sluit in:

* **/System**
* **/bin**
* **/sbin**
* **/usr**

Die reëls wat SIP se gedrag beheer, is gedefinieer in die konfigurasie-lêer wat geleë is by **`/System/Library/Sandbox/rootless.conf`**. Binne hierdie lêer word paaie wat voorafgegaan word deur 'n sterretjie (\*) aangedui as uitsonderings op die andersins streng SIP-beperkings.

Oorweeg die volgende voorbeeld:
```javascript
/usr
* /usr/libexec/cups
* /usr/local
* /usr/share/man
```
Hierdie uittreksel impliseer dat terwyl SIP gewoonlik die **`/usr`**-gids beveilig, daar spesifieke subgidse (`/usr/libexec/cups`, `/usr/local`, en `/usr/share/man`) is waar wysigings toegelaat word, soos aangedui deur die sterretjie (\*) voor hul paaie.

Om te verifieer of 'n gids of lêer deur SIP beskerm word, kan jy die **`ls -lOd`**-bevel gebruik om vir die teenwoordigheid van die **`restricted`**- of **`sunlnk`**-vlag te kyk. Byvoorbeeld:
```bash
ls -lOd /usr/libexec/cups
drwxr-xr-x  11 root  wheel  sunlnk 352 May 13 00:29 /usr/libexec/cups
```
In hierdie geval, dui die **`sunlnk`** vlag aan dat die `/usr/libexec/cups` gids self **nie verwyder kan word nie**, hoewel lêers binne dit geskep, gewysig, of verwyder kan word.

Aan die ander kant:
```bash
ls -lOd /usr/libexec
drwxr-xr-x  338 root  wheel  restricted 10816 May 13 00:29 /usr/libexec
```
Hier dui die **`beperk`** vlag aan dat die `/usr/libexec` gids beskerm word deur SIP. In 'n SIP-beskermde gids kan lêers nie geskep, gewysig, of verwyder word nie.

Verder, as 'n lêer die eienskap **`com.apple.rootless`** uitgebreide **eienskap** bevat, sal daardie lêer ook deur SIP **beskerm word**.

**SIP beperk ook ander root-aksies** soos:

* Laai onbetroubare kernel-uitbreidings
* Kry taak-poorte vir Apple-ondertekende prosesse
* Wysiging van NVRAM-veranderlikes
* Toelaat van kernel foutopsporing

Opsies word in nvram-veranderlike as 'n bietjievlag (`csr-active-config` op Intel en `lp-sip0` word gelees van die geboote Toestelboom vir ARM) gehou. Jy kan die vlae vind in die XNU bronkode in `csr.sh`:

<figure><img src="../../../.gitbook/assets/image (1192).png" alt=""><figcaption></figcaption></figure>

### SIP Status

Jy kan nagaan of SIP op jou stelsel geaktiveer is met die volgende bevel:
```bash
csrutil status
```
Indien jy SIP moet deaktiveer, moet jy jou rekenaar herlaai in herstelmodus (deur Command+R tydens opstart te druk), en dan die volgende bevel uitvoer:
```bash
csrutil disable
```
Indien jy wil voortgaan met SIP geaktiveer maar die ontslag van die foutopsporingsbeskerming wil verwyder, kan jy dit doen met:
```bash
csrutil enable --without debug
```
### Ander Beperkings

* **Verbied die laai van ongetekende kernel-uitbreidings** (kexts), verseker slegs geverifieerde uitbreidings interaksie met die stelselkernel.
* **Voorkom die foutsoektog** van macOS-stelselprosesse, beskerm kernstelselkomponente teen ongemagtigde toegang en wysiging.
* **Belemmer gereedskappe** soos dtrace om stelselprosesse te ondersoek, wat die integriteit van die stelselbedryf verder beskerm.

[**Leer meer oor SIP-inligting in hierdie aanbieding**](https://www.slideshare.net/i0n1c/syscan360-stefan-esser-os-x-el-capitan-sinking-the-ship)**.**

## SIP-Omseilings

Omseiling van SIP stel 'n aanvaller in staat om:

* **Toegang tot Gebruikersdata**: Lees sensitiewe gebruikersdata soos e-pos, boodskappe, en Safari geskiedenis van alle gebruikersrekeninge.
* **TCC-Omseiling**: Manipuleer direk die TCC (Deursigtigheid, Toestemming, en Beheer) databasis om ongemagtigde toegang tot die webkamera, mikrofoon, en ander hulpbronne te verleen.
* **Vestig Volharding**: Plaas kwaadwillige sagteware in SIP-beskermde liggings, wat dit bestand maak teen verwydering, selfs deur wortelpriviliges. Dit sluit ook die potensiaal in om met die Malware Removal Tool (MRT) te knoei.
* **Laai Kernel-Uitbreidings**: Alhoewel daar addisionele beskermings is, vereenvoudig die omseiling van SIP die proses om ongetekende kernel-uitbreidings te laai.

### Installeerderpakkette

**Installeerderpakkette wat met Apple se sertifikaat onderteken is** kan sy beskerming omseil. Dit beteken dat selfs pakkette wat deur standaardontwikkelaars onderteken is, geblokkeer sal word as hulle probeer om SIP-beskermde gids te wysig.

### Nie-bestaande SIP-lêer

Een potensiële leemte is dat as 'n lêer in **`rootless.conf` gespesifiseer word maar tans nie bestaan nie**, dit geskep kan word. Kwaadwillige sagteware kan dit uitbuit om **volharding te vestig** op die stelsel. Byvoorbeeld, 'n skadelike program kan 'n .plist-lêer in `/System/Library/LaunchDaemons` skep as dit in `rootless.conf` gelys word maar nie teenwoordig is nie.

### com.apple.rootless.install.heritable

{% hint style="gevaar" %}
Die toestemming **`com.apple.rootless.install.heritable`** maak dit moontlik om SIP te omseil
{% endhint %}

#### [CVE-2019-8561](https://objective-see.org/blog/blog\_0x42.html) <a href="#cve" id="cve"></a>

Daar is ontdek dat dit moontlik was om **die installeerderpakkie te ruil nadat die stelsel sy kodesignatuur geverifieer het** en dan sou die stelsel die skadelike pakkie installeer in plaas van die oorspronklike. Aangesien hierdie aksies deur **`system_installd`** uitgevoer is, sou dit SIP toelaat om omseil te word.

#### [CVE-2020–9854](https://objective-see.org/blog/blog\_0x4D.html) <a href="#cve-unauthd-chain" id="cve-unauthd-chain"></a>

As 'n pakkie van 'n gemoniteerde beeld of eksterne aandrywing geïnstalleer is, sou die **installeerder** die binêre lêer van **daardie lêerstelsel uitvoer** (eerder as van 'n SIP-beskermde plek), wat **`system_installd`** 'n arbitêre binêre lêer sou laat uitvoer.

#### CVE-2021-30892 - Shrootless

[**Navorsers van hierdie blogpos**](https://www.microsoft.com/en-us/security/blog/2021/10/28/microsoft-finds-new-macos-vulnerability-shrootless-that-could-bypass-system-integrity-protection/) het 'n kwesbaarheid in macOS se Stelselintegriteitsbeskerming (SIP) meganisme ontdek, genoem die 'Shrootless' kwesbaarheid. Hierdie kwesbaarheid draai om die **`system_installd`** daemon, wat 'n toestemming het, **`com.apple.rootless.install.heritable`**, wat enige van sy kinderprosesse toelaat om SIP se lêersisteembeperkings te omseil.

**`system_installd`** daemon sal pakkette installeer wat deur **Apple** onderteken is.

Navorsers het bevind dat tydens die installasie van 'n Apple-ondertekende pakkie (.pkg-lêer), **`system_installd`** enige **na-installeer** skripte wat in die pakkie ingesluit is, **uitvoer**. Hierdie skripte word deur die verstek skul, **`zsh`**, uitgevoer, wat outomaties bevele uitvoer van die **`/etc/zshenv`** lêer, indien dit bestaan, selfs in nie-interaktiewe modus. Hierdie gedrag kan deur aanvallers uitgebuit word: deur 'n skadelike `/etc/zshenv` lêer te skep en te wag vir **`system_installd` om `zsh` aan te roep**, kan hulle arbitêre handelinge op die toestel uitvoer.

Daarbenewens is daar ontdek dat **`/etc/zshenv` as 'n algemene aanvalstegniek gebruik kon word**, nie net vir 'n SIP-omseiling nie. Elke gebruikersprofiel het 'n `~/.zshenv` lêer, wat dieselfde manier as `/etc/zshenv` optree maar nie wortelregte benodig nie. Hierdie lêer kan gebruik word as 'n volhardingsmeganisme, wat elke keer geaktiveer word as `zsh` begin, of as 'n bevoorregtingsverhogingsmeganisme. As 'n admin-gebruiker na wortel verhoog met `sudo -s` of `sudo <opdrag>`, sou die `~/.zshenv` lêer geaktiveer word, wat effektief na wortel verhoog.

#### [**CVE-2022-22583**](https://perception-point.io/blog/technical-analysis-cve-2022-22583/)

In [**CVE-2022-22583**](https://perception-point.io/blog/technical-analysis-cve-2022-22583/) is ontdek dat dieselfde **`system_installd`** proses steeds misbruik kon word omdat dit die **na-installeer skripsie binne 'n ewekansig genoemde vouer beskerm deur SIP binne `/tmp`** geplaas het. Die ding is dat **`/tmp` self nie deur SIP beskerm word nie**, dus was dit moontlik om 'n **virtuele beeld daarop te monteer**, dan sou die **installeerder** die **na-installeer skripsie** daarin sit, die virtuele beeld **aflaai**, al die **vouers herstel**, en die **na-installasie** skripsie met die **lading** om uit te voer, byvoeg.

#### [fsck\_cs nutsprogram](https://www.theregister.com/2016/03/30/apple\_os\_x\_rootless/)

'n Kwesbaarheid is geïdentifiseer waar **`fsck_cs`** mislei is om 'n belangrike lêer te beskadig, as gevolg van sy vermoë om **simboliese skakels** te volg. Spesifiek het aanvallers 'n skakel van _`/dev/diskX`_ na die lêer `/System/Library/Extensions/AppleKextExcludeList.kext/Contents/Info.plist` geskep. Die uitvoering van **`fsck_cs`** op _`/dev/diskX`_ het gelei tot die beskadiging van `Info.plist`. Hierdie lêer se integriteit is noodsaaklik vir die bedryf van die stelsel se SIP (Stelselintegriteitsbeskerming), wat die laai van kernel-uitbreidings beheer. Sodra dit beskadig is, is SIP se vermoë om kernel-uitsluitings te bestuur, gekompromitteer.

Die opdragte om hierdie kwesbaarheid te misbruik is:
```bash
ln -s /System/Library/Extensions/AppleKextExcludeList.kext/Contents/Info.plist /dev/diskX
fsck_cs /dev/diskX 1>&-
touch /Library/Extensions/
reboot
```
Die uitbuiting van hierdie kwesbaarheid het ernstige implikasies. Die `Info.plist` lêer, normaalweg verantwoordelik vir die bestuur van toestemmings vir kernel-uitbreidings, word ondoeltreffend. Dit sluit die onvermoë om sekere uitbreidings op 'n swartlys te plaas, soos `AppleHWAccess.kext`, in. Gevolglik, met die SIP se beheermeganisme buite werking, kan hierdie uitbreiding gelaai word, wat ongemagtigde lees- en skryftoegang tot die stelsel se RAM verleen.

#### [Monteer oor SIP-beskermde lêers](https://www.slideshare.net/i0n1c/syscan360-stefan-esser-os-x-el-capitan-sinking-the-ship)

Dit was moontlik om 'n nuwe lêersisteem oor **SIP-beskermde lêers te monteer om die beskerming te omseil**.
```bash
mkdir evil
# Add contento to the folder
hdiutil create -srcfolder evil evil.dmg
hdiutil attach -mountpoint /System/Library/Snadbox/ evil.dmg
```
#### [Opgraderingsontduiking (2016)](https://objective-see.org/blog/blog\_0x14.html)

Die stelsel is ingestel om te begin vanaf 'n ingeslote installeerderfbeeld binne die `Installeer macOS Sierra.app` om die OS op te gradeer, deur gebruik te maak van die `bless` nutsprogram. Die gebruikte bevel is as volg:
```bash
/usr/sbin/bless -setBoot -folder /Volumes/Macintosh HD/macOS Install Data -bootefi /Volumes/Macintosh HD/macOS Install Data/boot.efi -options config="\macOS Install Data\com.apple.Boot" -label macOS Installer
```
Die sekuriteit van hierdie proses kan gekompromitteer word as 'n aanvaller die opgraderingsbeeld (`InstallESD.dmg`) verander voordat dit begin. Die strategie behels die vervanging van 'n dinamiese laaier (dyld) met 'n skadelike weergawe (`libBaseIA.dylib`). Hierdie vervanging lei daartoe dat die aanvaller se kode uitgevoer word wanneer die installeerder geïnisieer word.

Die aanvaller se kode verkry beheer gedurende die opgraderingsproses deur die stelsel se vertroue in die installeerder uit te buit. Die aanval vorder deur die `InstallESD.dmg` beeld te verander deur middel van metode swizzling, wat spesifiek die `extractBootBits` metode teiken. Dit maak die inspuiting van skadelike kode moontlik voordat die skyfbeeld gebruik word.

Verder, binne die `InstallESD.dmg`, is daar 'n `BaseSystem.dmg`, wat as die opgraderingskode se hooflêersisteem dien. Die inspuiting van 'n dinamiese biblioteek hierin maak dit vir die skadelike kode moontlik om binne 'n proses te werk wat in staat is om OS-vlak lêers te verander, wat die potensiaal vir stelselkompromittering aansienlik verhoog.

#### [systemmigrationd (2023)](https://www.youtube.com/watch?v=zxZesAN-TEk)

In hierdie gesprek van [**DEF CON 31**](https://www.youtube.com/watch?v=zxZesAN-TEk), word getoon hoe **`systemmigrationd`** (wat SIP kan omseil) 'n **bash** en 'n **perl** skriffie uitvoer, wat misbruik kan word via omgewingsveranderlikes **`BASH_ENV`** en **`PERL5OPT`**.

#### CVE-2023-42860 <a href="#cve-a-detailed-look" id="cve-a-detailed-look"></a>

Soos [**gedetailleer in hierdie blogpos**](https://blog.kandji.io/apple-mitigates-vulnerabilities-installer-scripts), het 'n `postinstall` skriffie van `InstallAssistant.pkg` pakkette toegelaat om uitgevoer te word:
```bash
/usr/bin/chflags -h norestricted "${SHARED_SUPPORT_PATH}/SharedSupport.dmg"
```
en dit was moontlik om 'n simboliese skakel te skep in `${SHARED_SUPPORT_PATH}/SharedSupport.dmg` wat 'n gebruiker sou toelaat om **enige lêer te ontsper, SIP-beskerming te omseil**.

### **com.apple.rootless.install**

{% hint style="danger" %}
Die toestemming **`com.apple.rootless.install`** maak dit moontlik om SIP te omseil
{% endhint %}

Die toestemming `com.apple.rootless.install` is bekend om System Integrity Protection (SIP) op macOS te omseil. Dit is veral genoem in verband met [**CVE-2022-26712**](https://jhftss.github.io/CVE-2022-26712-The-POC-For-SIP-Bypass-Is-Even-Tweetable/).

In hierdie spesifieke geval besit die stelsel XPC-diens geleë by `/System/Library/PrivateFrameworks/ShoveService.framework/Versions/A/XPCServices/SystemShoveService.xpc` hierdie toestemming. Dit laat die verwante proses toe om SIP-beperkings te omseil. Verder bied hierdie diens 'n metode wat die beweging van lêers toelaat sonder om enige sekuriteitsmaatreëls af te dwing.

## Verzegelde Stelsel-Snapshots

Verzegelde Stelsel-Snapshots is 'n kenmerk wat deur Apple in **macOS Big Sur (macOS 11)** ingevoer is as deel van sy **System Integrity Protection (SIP)** meganisme om 'n addisionele laag van sekuriteit en stelselstabiliteit te bied. Dit is essensieel leesbare weergawes van die stelselvolume.

Hier is 'n meer gedetailleerde blik:

1. **Onveranderlike Stelsel**: Verzegelde Stelsel-Snapshots maak die macOS stelselvolume "onveranderlik", wat beteken dat dit nie gewysig kan word nie. Dit voorkom enige ongemagtigde of toevallige veranderinge aan die stelsel wat sekuriteit of stelselstabiliteit kan benadeel.
2. **Stelsel Sagteware-opdaterings**: Wanneer jy macOS-opdaterings of opgraderings installeer, skep macOS 'n nuwe stelsel-snapshot. Die macOS-opstartvolume gebruik dan **APFS (Apple File System)** om na hierdie nuwe snapshot oor te skakel. Die hele proses van die toepassing van opdaterings word veiliger en betroubaarder aangesien die stelsel altyd na die vorige snapshot kan terugkeer as iets verkeerd gaan tydens die opdatering.
3. **Data Skeiding**: In samehang met die konsep van Data- en Stelselvolume-skeiding wat in macOS Catalina ingevoer is, verseker die Verzegelde Stelsel-Snapshot-funksie dat al jou data en instellings op 'n aparte "**Data**" volume gestoor word. Hierdie skeiding maak jou data onafhanklik van die stelsel, wat die proses van stelselopdaterings vereenvoudig en stelselsekuriteit verbeter.

Onthou dat hierdie snapshots outomaties bestuur word deur macOS en nie addisionele spasie op jou skyf inneem nie, dankie aan die spasie-deelvermoë van APFS. Dit is ook belangrik om te let dat hierdie snapshots verskil van **Time Machine-snapshots**, wat gebruikerstoeganklike rugsteun van die hele stelsel is.

### Kontroleer Snapshots

Die opdrag **`diskutil apfs list`** lys die **besonderhede van die APFS-volume** en hul uitleg:

<pre><code>+-- Container disk3 966B902E-EDBA-4775-B743-CF97A0556A13
|   ====================================================
|   APFS Container Verwysing:     disk3
|   Grootte (Kapasiteitsplafon):  494384795648 B (494.4 GB)
|   Kapasiteit wat deur volumes gebruik word:   219214536704 B (219.2 GB) (44.3% gebruik)
|   Kapasiteit nie toegewys nie:       275170258944 B (275.2 GB) (55.7% vry)
|   |
|   +-&#x3C; Fisiese Stoor disk0s2 86D4B7EC-6FA5-4042-93A7-D3766A222EBE
|   |   -----------------------------------------------------------
|   |   APFS Fisiese Stoor Disk:   disk0s2
|   |   Grootte:                       494384795648 B (494.4 GB)
|   |
|   +-> Volume disk3s1 7A27E734-880F-4D91-A703-FB55861D49B7
|   |   ---------------------------------------------------
<strong>|   |   APFS Volume Disk (Rol):   disk3s1 (Stelsel)
</strong>|   |   Naam:                      Macintosh HD (Nie-sensitief vir hoofletters)
<strong>|   |   Koppelingspunt:               /System/Volumes/Update/mnt1
</strong>|   |   Kapasiteit Verbruik:         12819210240 B (12.8 GB)
|   |   Verzegel:                    Gebreek
|   |   FileVault:                 Ja (Oopgemaak)
|   |   Versleutel:                 Nee
|   |   |
|   |   Snapshot:                  FAA23E0C-791C-43FF-B0E7-0E1C0810AC61
|   |   Snapshot Disk:             disk3s1s1
<strong>|   |   Snapshot Koppelingspunt:      /
</strong><strong>|   |   Snapshot Verzegel:           Ja
</strong>[...]
+-> Volume disk3s5 281959B7-07A1-4940-BDDF-6419360F3327
|   ---------------------------------------------------
|   APFS Volume Disk (Rol):   disk3s5 (Data)
|   Naam:                      Macintosh HD - Data (Nie-sensitief vir hoofletters)
<strong>    |   Koppelingspunt:               /System/Volumes/Data
</strong><strong>    |   Kapasiteit Verbruik:         412071784448 B (412.1 GB)
</strong>    |   Verzegel:                    Nee
|   FileVault:                 Ja (Oopgemaak)
</code></pre>

In die vorige uitset is dit moontlik om te sien dat **gebruikerstoeganklike liggings** gekoppel is onder `/System/Volumes/Data`.

Verder is die **macOS-stelselvolumesnapshot** gekoppel in `/` en dit is **verzegel** (kriptografies deur die OS onderteken). Dus, as SIP omseil word en dit verander, sal die **OS nie meer opstart nie**.

Dit is ook moontlik om **te verifieer dat die verseël is geaktiveer** deur die volgende uit te voer:
```bash
csrutil authenticated-root status
Authenticated Root status: enabled
```
Boonop, die oomblikopname skyf is ook gemoniteer as **net-lees**:
```bash
mount
/dev/disk3s1s1 on / (apfs, sealed, local, read-only, journaled)
```
### [WhiteIntel](https://whiteintel.io)

<figure><img src="../../../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) is 'n **dark-web** aangedrewe soekenjin wat **gratis** funksies bied om te kontroleer of 'n maatskappy of sy kliënte deur **diefstal malware** **gekompromitteer** is.

Hul primêre doel van WhiteIntel is om rekening-oorneemings en afpersingsaanvalle te beveg wat voortspruit uit inligtingsteel-malware.

Jy kan hul webwerf besoek en hul enjin vir **gratis** probeer by:

{% embed url="https://whiteintel.io" %}

<details>

<summary><strong>Leer AWS hak vanaf nul tot held met</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Ander maniere om HackTricks te ondersteun:

* As jy wil sien dat jou **maatskappy geadverteer word in HackTricks** of **HackTricks aflaai in PDF-formaat** Kyk na die [**INSKRYWINGSPLANNE**](https://github.com/sponsors/carlospolop)!
* Kry die [**amptelike PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Ontdek [**Die PEASS Familie**](https://opensea.io/collection/the-peass-family), ons versameling eksklusiewe [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Sluit aan by die** 💬 [**Discord groep**](https://discord.gg/hRep4RUj7f) of die [**telegram groep**](https://t.me/peass) of **volg** ons op **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Deel jou haktruuks deur PRs in te dien by die** [**HackTricks**](https://github.com/carlospolop/hacktricks) en [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github-opslag.

</details>
