# macOS Fajlovi, Folderi, Binarni fajlovi i Memorija

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** Proverite [**PLANOVE ZA PRIJAVU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**Porodicu PEASS**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili **pratite** nas na **Twitteru** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Poređenje hijerarhije fajlova

* **/Applications**: Instalirane aplikacije treba da budu ovde. Svi korisnici će imati pristup njima.
* **/bin**: Binarni fajlovi komandne linije
* **/cores**: Ako postoji, koristi se za čuvanje core dump-ova
* **/dev**: Sve je tretirano kao fajl pa možete videti hardverske uređaje ovde.
* **/etc**: Konfiguracioni fajlovi
* **/Library**: Mnogo poddirektorijuma i fajlova vezanih za postavke, keš i logove se mogu naći ovde. Postoji Library folder u root-u i u svakom korisničkom direktorijumu.
* **/private**: Nedokumentovano ali mnogi pomenuti folderi su simboličke veze ka privatnom direktorijumu.
* **/sbin**: Bitni sistemski binarni fajlovi (vezani za administraciju)
* **/System**: Fajl za pokretanje OS X-a. Trebalo bi da ovde uglavnom pronađete samo Apple specifične fajlove (ne treće strane).
* **/tmp**: Fajlovi se brišu posle 3 dana (to je soft link ka /private/tmp)
* **/Users**: Matični direktorijum za korisnike.
* **/usr**: Konfiguracioni i sistemski binarni fajlovi
* **/var**: Log fajlovi
* **/Volumes**: Montirani drajvovi će se pojaviti ovde.
* **/.vol**: Pokretanjem `stat a.txt` dobijate nešto poput `16777223 7545753 -rw-r--r-- 1 username wheel ...` gde je prvi broj ID broj volumena gde fajl postoji, a drugi je inode broj. Možete pristupiti sadržaju ovog fajla kroz /.vol/ sa tim informacijama pokretanjem `cat /.vol/16777223/7545753`

### Folderi Aplikacija

* **Sistemski aplikacije** se nalaze pod `/System/Applications`
* **Instalirane** aplikacije obično se instaliraju u `/Applications` ili u `~/Applications`
* **Podaci aplikacije** se mogu naći u `/Library/Application Support` za aplikacije koje se izvršavaju kao root i `~/Library/Application Support` za aplikacije koje se izvršavaju kao korisnik.
* **Demoni** trećih strana aplikacija koji **mora da se izvršavaju kao root** obično se nalaze u `/Library/PrivilegedHelperTools/`
* **Aplikacije u pesku** su mapirane u folder `~/Library/Containers`. Svaka aplikacija ima folder nazvan prema ID-u paketa aplikacije (`com.apple.Safari`).
* **Kernel** se nalazi u `/System/Library/Kernels/kernel`
* **Apple-ove kernel ekstenzije** se nalaze u `/System/Library/Extensions`
* **Ekstenzije kernela trećih strana** se čuvaju u `/Library/Extensions`

### Fajlovi sa Osetljivim Informacijama

macOS čuva informacije poput lozinki na nekoliko mesta:

{% content-ref url="macos-sensitive-locations.md" %}
[macos-sensitive-locations.md](macos-sensitive-locations.md)
{% endcontent-ref %}

### Ranjivi pkg instalateri

{% content-ref url="macos-installers-abuse.md" %}
[macos-installers-abuse.md](macos-installers-abuse.md)
{% endcontent-ref %}

## OS X Specifične Ekstenzije

* **`.dmg`**: Fajlovi Apple Disk Image su veoma česti za instalatere.
* **`.kext`**: Mora pratiti specifičnu strukturu i to je OS X verzija drajvera. (to je paket)
* **`.plist`**: Takođe poznat kao property list čuva informacije u XML ili binarnom formatu.
* Može biti XML ili binarni. Binarni se mogu pročitati sa:
* `defaults read config.plist`
* `/usr/libexec/PlistBuddy -c print config.plsit`
* `plutil -p ~/Library/Preferences/com.apple.screensaver.plist`
* `plutil -convert xml1 ~/Library/Preferences/com.apple.screensaver.plist -o -`
* `plutil -convert json ~/Library/Preferences/com.apple.screensaver.plist -o -`
* **`.app`**: Apple aplikacije koje prate strukturu direktorijuma (To je paket).
* **`.dylib`**: Dinamičke biblioteke (kao Windows DLL fajlovi)
* **`.pkg`**: Isti su kao xar (eXtensible Archive format). Komanda installer se može koristiti za instalaciju sadržaja ovih fajlova.
* **`.DS_Store`**: Ovaj fajl se nalazi u svakom direktorijumu, čuva atribute i prilagođavanja direktorijuma.
* **`.Spotlight-V100`**: Ovaj folder se pojavljuje u korenskom direktorijumu svakog volumena na sistemu.
* **`.metadata_never_index`**: Ako se ovaj fajl nalazi u korenu volumena, Spotlight neće indeksirati taj volumen.
* **`.noindex`**: Fajlovi i folderi sa ovom ekstenzijom neće biti indeksirani od strane Spotlight-a.
* **`.sdef`**: Fajlovi unutar paketa koji specificiraju kako je moguće interagovati sa aplikacijom putem AppleScript-a.

### macOS Paketi

Paket je **direktorijum** koji **izgleda kao objekat u Finder-u** (primer paketa su fajlovi `*.app`).

{% content-ref url="macos-bundles.md" %}
[macos-bundles.md](macos-bundles.md)
{% endcontent-ref %}

## Dyld Deljeni Keš Biblioteka (SLC)

Na macOS-u (i iOS-u) svi deljeni sistemski fajlovi, poput okvira i dylib-ova, **kombinovani su u jedan fajl**, nazvan **dyld deljeni keš biblioteka**. Ovo poboljšava performanse, jer se kod može učitati brže.

Ovo se nalazi na macOS-u u `/System/Volumes/Preboot/Cryptexes/OS/System/Library/dyld/` i u starijim verzijama možda možete pronaći **deljeni keš** u **`/System/Library/dyld/`**.\
Na iOS-u ih možete pronaći u **`/System/Library/Caches/com.apple.dyld/`**.

Slično dyld deljenom kešu, kernel i kernel ekstenzije takođe su kompilovane u keš kernela, koji se učitava prilikom pokretanja.

Da biste izvukli biblioteke iz jednog fajla dylib deljenog keša, bilo je moguće koristiti binarni [dyld\_shared\_cache\_util](https://www.mbsplugins.de/files/dyld\_shared\_cache\_util-dyld-733.8.zip) koji možda ne radi više, ali možete koristiti i [**dyldextractor**](https://github.com/arandomdev/dyldextractor):

{% code overflow="wrap" %}
```bash
# dyld_shared_cache_util
dyld_shared_cache_util -extract ~/shared_cache/ /System/Volumes/Preboot/Cryptexes/OS/System/Library/dyld/dyld_shared_cache_arm64e

# dyldextractor
dyldex -l [dyld_shared_cache_path] # List libraries
dyldex_all [dyld_shared_cache_path] # Extract all
# More options inside the readme
```
{% endcode %}

{% hint style="success" %}
Imajte na umu da čak i ako alat `dyld_shared_cache_util` ne radi, možete proslediti **deljeni dyld binarni fajl Hopper-u** i Hopper će moći da identifikuje sve biblioteke i omogući vam da **izaberete koju** želite da istražite:
{% endhint %}

<figure><img src="../../../.gitbook/assets/image (1149).png" alt="" width="563"><figcaption></figcaption></figure>

Neki ekstraktori neće raditi jer su dylibs prelinkovani sa hardkodiranim adresama i zbog toga mogu skakati na nepoznate adrese.

{% hint style="success" %}
Takođe je moguće preuzeti Deljeni bibliotečki keš drugih \*OS uređaja na macOS-u korišćenjem emulatora u Xcode-u. Biće preuzeti unutar: ls `$HOME/Library/Developer/Xcode/<*>OS\ DeviceSupport/<version>/Symbols/System/Library/Caches/com.apple.dyld/`, kao:`$HOME/Library/Developer/Xcode/iOS\ DeviceSupport/14.1\ (18A8395)/Symbols/System/Library/Caches/com.apple.dyld/dyld_shared_cache_arm64`
{% endhint %}

### Mapiranje SLC

**`dyld`** koristi sistemski poziv **`shared_region_check_np`** da zna da li je SLC mapiran (što vraća adresu) i **`shared_region_map_and_slide_np`** da mapira SLC.

Imajte na umu da čak i ako je SLC kliznuo pri prvom korišćenju, svi **procesi** koriste **isti primerak**, što **eliminiše ASLR** zaštitu ako je napadač bio u mogućnosti da pokrene procese u sistemu. Ovo je zapravo iskorišćeno u prošlosti i popravljeno sa deljenim regionom pagera.

Grane bazena su male Mach-O dylibs koje stvaraju male prostore između mapa slika čineći nemogućim interponovanje funkcija.

### Poništavanje SLC-ova

Korišćenjem okruženjskih promenljivih:

* **`DYLD_DHARED_REGION=private DYLD_SHARED_CACHE_DIR=</path/dir> DYLD_SHARED_CACHE_DONT_VALIDATE=1`** -> Ovo će omogućiti učitavanje novog deljenog bibliotečkog keša
* **`DYLD_SHARED_CACHE_DIR=avoid`** i ručno zamenite biblioteke simboličkim linkovima ka deljenom kešu sa pravim (morate ih izdvojiti)

## Posebne Dozvole za Fajlove

### Dozvole za Foldere

U **folderu**, **čitanje** omogućava da ga **listate**, **pisanje** omogućava da ga **obrišete** i **pišete** fajlove u njemu, a **izvršavanje** omogućava da **pretražujete** direktorijum. Dakle, na primer, korisnik sa **dozvolom za čitanje nad fajlom** unutar direktorijuma gde **nema dozvolu za izvršavanje** **neće moći da pročita** fajl.

### Modifikatori zastavica

Postoje neke zastavice koje se mogu postaviti u fajlovima koje će fajl ponašati drugačije. Možete **proveriti zastavice** fajlova unutar direktorijuma sa `ls -lO /path/directory`

* **`uchg`**: Poznat kao **uchange** flag će **sprečiti bilo koju akciju** menjanja ili brisanja **fajla**. Da biste ga postavili uradite: `chflags uchg file.txt`
* Korisnik root može **ukloniti zastavicu** i izmeniti fajl
* **`restricted`**: Ova zastavica čini da fajl bude **zaštićen SIP-om** (ne možete dodati ovu zastavicu fajlu).
* **`Sticky bit`**: Ako je direktorijum sa sticky bitom, **samo** vlasnik direktorijuma ili root mogu preimenovati ili obrisati fajlove. Tipično se postavlja na /tmp direktorijum da bi se sprečilo obične korisnike da brišu ili premeštaju fajlove drugih korisnika.

Sve zastavice se mogu naći u fajlu `sys/stat.h` (pronađite ga koristeći `mdfind stat.h | grep stat.h`) i to su:

* `UF_SETTABLE` 0x0000ffff: Maska promenljivih zastavica vlasnika.
* `UF_NODUMP` 0x00000001: Ne sme se dumpovati fajl.
* `UF_IMMUTABLE` 0x00000002: Fajl se ne sme menjati.
* `UF_APPEND` 0x00000004: Pisanja u fajl mogu samo dodavati.
* `UF_OPAQUE` 0x00000008: Direktorijum je neproziran u odnosu na uniju.
* `UF_COMPRESSED` 0x00000020: Fajl je kompresovan (neki fajl-sistemi).
* `UF_TRACKED` 0x00000040: Nema obaveštenja za brisanja/preimenovanja za fajlove sa ovim setom.
* `UF_DATAVAULT` 0x00000080: Potrebno je ovlašćenje za čitanje i pisanje.
* `UF_HIDDEN` 0x00008000: Nagoveštaj da ovaj predmet ne bi trebalo da se prikazuje u GUI-ju.
* `SF_SUPPORTED` 0x009f0000: Maska superuser podržanih zastavica.
* `SF_SETTABLE` 0x3fff0000: Maska superuser promenljivih zastavica.
* `SF_SYNTHETIC` 0xc0000000: Maska sistema samo za čitanje sintetičkih zastavica.
* `SF_ARCHIVED` 0x00010000: Fajl je arhiviran.
* `SF_IMMUTABLE` 0x00020000: Fajl se ne sme menjati.
* `SF_APPEND` 0x00040000: Pisanja u fajl mogu samo dodavati.
* `SF_RESTRICTED` 0x00080000: Potrebno je ovlašćenje za pisanje.
* `SF_NOUNLINK` 0x00100000: Stavka se ne sme ukloniti, preimenovati ili montirati.
* `SF_FIRMLINK` 0x00800000: Fajl je čvrsta veza.
* `SF_DATALESS` 0x40000000: Fajl je objekat bez podataka.

### **Fajl ACLs**

Fajl **ACLs** sadrže **ACE** (Access Control Entries) gde se mogu dodeliti više **granularnih dozvola** različitim korisnicima.

Moguće je dodeliti **direktorijumu** ove dozvole: `list`, `search`, `add_file`, `add_subdirectory`, `delete_child`, `delete_child`.\
I fajlu: `read`, `write`, `append`, `execute`.

Kada fajl sadrži ACLs, **naći ćete "+" prilikom listanja dozvola kao u**:
```bash
ls -ld Movies
drwx------+   7 username  staff     224 15 Apr 19:42 Movies
```
Možete **pročitati ACL-ove** datoteke pomoću:
```bash
ls -lde Movies
drwx------+ 7 username  staff  224 15 Apr 19:42 Movies
0: group:everyone deny delete
```
Možete pronaći **sve datoteke sa ACL-ovima** sa (ovo je veeoma sporo):
```bash
ls -RAle / 2>/dev/null | grep -E -B1 "\d: "
```
### Prošireni atributi

Prošireni atributi imaju ime i željenu vrednost, mogu se videti korišćenjem `ls -@` i manipulisati korišćenjem `xattr` komande. Neki uobičajeni prošireni atributi su:

* `com.apple.resourceFork`: Kompatibilnost resursnih viljuški. Takođe vidljivo kao `filename/..namedfork/rsrc`
* `com.apple.quarantine`: MacOS: Mehanizam karantina Gatekeeper-a (III/6)
* `metadata:*`: MacOS: razni metapodaci, kao što su `_backup_excludeItem`, ili `kMD*`
* `com.apple.lastuseddate` (#PS): Datum poslednje upotrebe datoteke
* `com.apple.FinderInfo`: MacOS: Informacije Finder-a (npr. boja oznaka)
* `com.apple.TextEncoding`: Određuje tekstualno kodiranje ASCII tekstualnih datoteka
* `com.apple.logd.metadata`: Korišćeno od strane logd-a na datotekama u `/var/db/diagnostics`
* `com.apple.genstore.*`: Generacijsko skladištenje (`/.DocumentRevisions-V100` u korenu fajl sistema)
* `com.apple.rootless`: MacOS: Korišćeno od strane Sistema zaštite integriteta za obeležavanje datoteke (III/10)
* `com.apple.uuidb.boot-uuid`: Oznake logd-a za epohe pokretanja sa jedinstvenim UUID-om
* `com.apple.decmpfs`: MacOS: Transparentna kompresija fajlova (II/7)
* `com.apple.cprotect`: \*OS: Podaci o šifrovanju po datoteci (III/11)
* `com.apple.installd.*`: \*OS: Metapodaci korišćeni od strane installd-a, npr., `installType`, `uniqueInstallID`

### Resursne viljuške | macOS ADS

Ovo je način da se dobiju **Alternativni podaci u toku u MacOS** mašinama. Možete sačuvati sadržaj unutar proširenog atributa nazvanog **com.apple.ResourceFork** unutar datoteke tako što ćete ga sačuvati u **file/..namedfork/rsrc**.
```bash
echo "Hello" > a.txt
echo "Hello Mac ADS" > a.txt/..namedfork/rsrc

xattr -l a.txt #Read extended attributes
com.apple.ResourceFork: Hello Mac ADS

ls -l a.txt #The file length is still q
-rw-r--r--@ 1 username  wheel  6 17 Jul 01:15 a.txt
```
Možete **pronaći sve datoteke koje sadrže ovaj prošireni atribut** sa:
```bash
find / -type f -exec ls -ld {} \; 2>/dev/null | grep -E "[x\-]@ " | awk '{printf $9; printf "\n"}' | xargs -I {} xattr -lv {} | grep "com.apple.ResourceFork"
```
{% endcode %}

### decmpfs

Prošireni atribut `com.apple.decmpfs` ukazuje da je datoteka pohranjena šifrovano, `ls -l` će prijaviti **veličinu 0** i komprimirani podaci su unutar ovog atributa. Svaki put kada se datoteka pristupi, biće dešifrovana u memoriji.

Ovaj atribut može se videti sa `ls -lO` označen kao komprimiran jer su komprimirane datoteke takođe označene zastavicom `UF_COMPRESSED`. Ako se komprimirana datoteka ukloni sa ovom zastavicom `chflags nocompressed </putanja/do/datoteke>`, sistem neće znati da je datoteka bila komprimirana i stoga neće moći da dekomprimuje i pristupi podacima (misliće da je zapravo prazna).

Alat afscexpand može se koristiti da prisili dekompresiju datoteke.

## **Univerzalne binarne datoteke &** Mach-o Format

Mac OS binarne datoteke obično su kompajlirane kao **univerzalne binarne datoteke**. **Univerzalna binarna datoteka** može **podržavati više arhitektura u istoj datoteci**.

{% content-ref url="universal-binaries-and-mach-o-format.md" %}
[universal-binaries-and-mach-o-format.md](universal-binaries-and-mach-o-format.md)
{% endcontent-ref %}

## Dumpovanje memorije macOS

{% content-ref url="macos-memory-dumping.md" %}
[macos-memory-dumping.md](macos-memory-dumping.md)
{% endcontent-ref %}

## Kategorija rizika datoteka Mac OS

Direktorijum `/System/Library/CoreServices/CoreTypes.bundle/Contents/Resources/System` je mesto gde su smeštene informacije o **riziku povezanom sa različitim ekstenzijama datoteka**. Ovaj direktorijum kategorizuje datoteke u različite nivoe rizika, utičući na to kako Safari obrađuje ove datoteke prilikom preuzimanja. Kategorije su:

* **LSRiskCategorySafe**: Datoteke u ovoj kategoriji se smatraju **potpuno sigurnim**. Safari će automatski otvoriti ove datoteke nakon što ih preuzme.
* **LSRiskCategoryNeutral**: Ove datoteke ne dolaze sa upozorenjima i **ne otvaraju se automatski** u Safariju.
* **LSRiskCategoryUnsafeExecutable**: Datoteke u ovoj kategoriji **pokreću upozorenje** koje ukazuje da je datoteka aplikacija. Ovo služi kao sigurnosna mera da upozori korisnika.
* **LSRiskCategoryMayContainUnsafeExecutable**: Ova kategorija je za datoteke, poput arhiva, koje mogu sadržati izvršnu datoteku. Safari će **pokrenuti upozorenje** osim ako može da potvrdi da su svi sadržaji sigurni ili neutralni.

## Log datoteke

* **`$HOME/Library/Preferences/com.apple.LaunchServices.QuarantineEventsV2`**: Sadrži informacije o preuzetim datotekama, poput URL adrese sa koje su preuzete.
* **`/var/log/system.log`**: Glavni log sistemima OSX. com.apple.syslogd.plist je odgovoran za izvršavanje sistemskog logovanja (možete proveriti da li je onemogućen tražeći "com.apple.syslogd" u `launchctl list`.
* **`/private/var/log/asl/*.asl`**: Ovo su Apple sistemski logovi koji mogu sadržati zanimljive informacije.
* **`$HOME/Library/Preferences/com.apple.recentitems.plist`**: Čuva nedavno pristupljene datoteke i aplikacije putem "Finder"-a.
* **`$HOME/Library/Preferences/com.apple.loginitems.plsit`**: Čuva stavke koje se pokreću prilikom pokretanja sistema.
* **`$HOME/Library/Logs/DiskUtility.log`**: Log datoteka za DiskUtility aplikaciju (informacije o drajvovima, uključujući USB-ove).
* **`/Library/Preferences/SystemConfiguration/com.apple.airport.preferences.plist`**: Podaci o bežičnim pristupnim tačkama.
* **`/private/var/db/launchd.db/com.apple.launchd/overrides.plist`**: Lista deaktiviranih demona.
