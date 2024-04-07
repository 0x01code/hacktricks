# macOS Ubacivanje biblioteke

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite svoju **kompaniju reklamiranu na HackTricks-u** ili da **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRIJAVU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**Porodicu PEASS**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

{% hint style="danger" %}
Kod **dyld je otvorenog koda** i može se pronaći na [https://opensource.apple.com/source/dyld/](https://opensource.apple.com/source/dyld/) i može se preuzeti kao tar korišćenjem **URL-a kao što je** [https://opensource.apple.com/tarballs/dyld/dyld-852.2.tar.gz](https://opensource.apple.com/tarballs/dyld/dyld-852.2.tar.gz)
{% endhint %}

## **DYLD\_INSERT\_LIBRARIES**

Ovo je slično kao [**LD\_PRELOAD na Linux-u**](../../../../linux-hardening/privilege-escalation/#ld\_preload). Omogućava da se naznači proces koji će se pokrenuti kako bi učitao određenu biblioteku sa putanje (ako je env var omogućen)

Ova tehnika takođe može biti **korišćena kao ASEP tehnika** jer svaka instalirana aplikacija ima plist nazvan "Info.plist" koji omogućava **dodeljivanje okružnih promenljivih** korišćenjem ključa nazvanog `LSEnvironmental`.

{% hint style="info" %}
Od 2012. godine **Apple je drastično smanjio moć** **`DYLD_INSERT_LIBRARIES`**.

Idite na kod i **proverite `src/dyld.cpp`**. U funkciji **`pruneEnvironmentVariables`** možete videti da su **`DYLD_*`** promenljive uklonjene.

U funkciji **`processRestricted`** postavljen je razlog ograničenja. Proverom tog koda možete videti da su razlozi:

* Binarni fajl je `setuid/setgid`
* Postojanje `__RESTRICT/__restrict` sekcije u macho binarnom fajlu.
* Softver ima privilegije (ojačano izvršavanje) bez [`com.apple.security.cs.allow-dyld-environment-variables`](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_allow-dyld-environment-variables) privilegije
* Proverite **privilegije** binarnog fajla sa: `codesign -dv --entitlements :- </path/to/bin>`

U ažuriranim verzijama ovu logiku možete pronaći u drugom delu funkcije **`configureProcessRestrictions`.** Međutim, ono što se izvršava u novijim verzijama su **početne provere funkcije** (možete ukloniti if-ove koji se odnose na iOS ili simulaciju jer se neće koristiti u macOS.
{% endhint %}

### Provera biblioteke

Čak i ako binarni fajl dozvoljava korišćenje **`DYLD_INSERT_LIBRARIES`** env promenljive, ako binarni fajl proverava potpis biblioteke da bi je učitao, neće učitati prilagođenu biblioteku.

Da biste učitali prilagođenu biblioteku, binarni fajl mora imati **jednu od sledećih privilegija**:

* [`com.apple.security.cs.disable-library-validation`](../../macos-security-protections/macos-dangerous-entitlements.md#com.apple.security.cs.disable-library-validation)
* [`com.apple.private.security.clear-library-validation`](../../macos-security-protections/macos-dangerous-entitlements.md#com.apple.private.security.clear-library-validation)

ili binarni fajl **ne sme** imati **ojačanu izvršnu oznaku** ili **oznaku provere biblioteke**.

Možete proveriti da li binarni fajl ima **ojačanu izvršnu oznaku** sa `codesign --display --verbose <bin>` proveravajući oznaku izvršne oznake u **`CodeDirectory`** kao: **`CodeDirectory v=20500 size=767 flags=0x10000(runtime) hashes=13+7 location=embedded`**

Takođe možete učitati biblioteku ako je **potpisana istim sertifikatom kao binarni fajl**.

Pronađite primer kako (zlo)upotrebiti ovo i proverite ograničenja u:

{% content-ref url="macos-dyld-hijacking-and-dyld_insert_libraries.md" %}
[macos-dyld-hijacking-and-dyld\_insert\_libraries.md](macos-dyld-hijacking-and-dyld\_insert\_libraries.md)
{% endcontent-ref %}

## Dylib Preuzimanje

{% hint style="danger" %}
Zapamtite da **prethodna ograničenja provere biblioteke takođe važe** za izvođenje napada Dylib preuzimanja.
{% endhint %}

Kao i u Windows-u, u MacOS-u takođe možete **preuzeti dylib** da biste omogućili **aplikacijama** da **izvrše** **proizvoljni** **kod** (dobro, zapravo od strane običnog korisnika ovo možda ne bi bilo moguće jer bi vam možda bila potrebna TCC dozvola da biste pisali unutar `.app` paketa i preuzeli biblioteku).\
Međutim, način na koji **MacOS** aplikacije **učitavaju** biblioteke je **više ograničen** nego u Windows-u. To znači da **maliciozni** razvijači i dalje mogu koristiti ovu tehniku za **skrivanje**, ali verovatnoća da će moći **zloupotrebiti ovo za eskalaciju privilegija je mnogo manja**.

Prvo, **češće je** pronaći da **MacOS binarni fajlovi pokazuju punu putanju** do biblioteka koje se učitavaju. I drugo, **MacOS nikada ne traži** u fasciklama **$PATH** za biblioteke.

**Glavni** deo **koda** koji se odnosi na ovu funkcionalnost je u **`ImageLoader::recursiveLoadLibraries`** u `ImageLoader.cpp`.

Postoje **4 različite komande zaglavlja** koje macho binarni fajl može koristiti za učitavanje biblioteka:

* Komanda **`LC_LOAD_DYLIB`** je uobičajena komanda za učitavanje dylib-a.
* Komanda **`LC_LOAD_WEAK_DYLIB`** radi kao prethodna, ali ako dylib nije pronađen, izvršenje se nastavlja bez greške.
* Komanda **`LC_REEXPORT_DYLIB`** proksi (ili re-izvozi) simbole iz druge biblioteke.
* Komanda **`LC_LOAD_UPWARD_DYLIB`** se koristi kada dve biblioteke zavise jedna od druge (ovo se naziva _upward dependency_).

Međutim, postoje **2 vrste dylib preuzimanja**:

* **Nedostajuće slabe povezane biblioteke**: To znači da će aplikacija pokušati učitati biblioteku koja ne postoji konfigurisana sa **LC\_LOAD\_WEAK\_DYLIB**. Zatim, **ako napadač postavi dylib tamo gde se očekuje, biće učitan**.
* Činjenica da je veza "slaba" znači da će aplikacija nastaviti sa radom čak i ako biblioteka nije pronađena.
* **Kod povezan** sa ovim je u funkciji `ImageLoaderMachO::doGetDependentLibraries` u `ImageLoaderMachO.cpp` gde je `lib->required` samo `false` kada je `LC_LOAD_WEAK_DYLIB` tačno.
* **Pronađite slabe povezane biblioteke** u binarnim fajlovima (kasnije imate primer kako kreirati biblioteke za preuzimanje):
* ```bash
otool -l </path/to/bin> | grep LC_LOAD_WEAK_DYLIB -A 5 cmd LC_LOAD_WEAK_DYLIB
cmdsize 56
name /var/tmp/lib/libUtl.1.dylib (offset 24)
time stamp 2 Wed Jun 21 12:23:31 1969
current version 1.0.0
compatibility version 1.0.0
```
* **Konfigurisano sa @rpath**: Mach-O binarni fajlovi mogu imati komande **`LC_RPATH`** i **`LC_LOAD_DYLIB`**. Na osnovu **vrednosti** ovih komandi, **biblioteke** će biti **učitane** iz **različitih direktorijuma**.
* **`LC_RPATH`** sadrži putanje nekih fascikli koje se koriste za učitavanje biblioteka od strane binarnog fajla.
* **`LC_LOAD_DYLIB`** sadrži putanju do određenih biblioteka za učitavanje. Ove putanje mogu sadržati **`@rpath`**, koji će biti **zamenjen** vrednostima u **`LC_RPATH`**. Ako postoji više putanja u **`LC_RPATH`** svaka će biti korišćena za pretragu biblioteke za učitavanje. Primer:
* Ako **`LC_LOAD_DYLIB`** sadrži `@rpath/library.dylib` i **`LC_RPATH`** sadrži `/application/app.app/Contents/Framework/v1/` i `/application/app.app/Contents/Framework/v2/`. Obje fascikle će se koristiti za učitavanje `library.dylib`**.** Ako biblioteka ne postoji u `[...]/v1/` i napadač može da je postavi tamo da preuzme učitavanje biblioteke u `[...]/v2/` jer se prate redosled putanja u **`LC_LOAD_DYLIB`**.
* **Pronađite rpath putanje i biblioteke** u binarnim fajlovima sa: `otool -l </putanja/do/binarnog>` | grep -E "LC_RPATH|LC_LOAD_DYLIB" -A 5

{% hint style="info" %}
**`@executable_path`**: Je **putanja** do direktorijuma koji sadrži **glavni izvršni fajl**.

**`@loader_path`**: Je **putanja** do **direktorijuma** koji sadrži **Mach-O binarni fajl** koji sadrži komandu za učitavanje.

* Kada se koristi u izvršnom fajlu, **`@loader_path`** je efektivno **isti** kao **`@executable_path`**.
* Kada se koristi u **dylib-u**, **`@loader_path`** daje **putanju** do **dylib-a**.
{% endhint %}

Način za **eskalciju privilegija** zloupotrebom ove funkcionalnosti bi bio u retkom slučaju kada **aplikacija** koju izvršava **root** traži neku **biblioteku u nekom folderu gde napadač ima dozvole za pisanje.**

{% hint style="success" %}
Dobar **skener** za pronalaženje **nedostajućih biblioteka** u aplikacijama je [**Dylib Hijack Scanner**](https://objective-see.com/products/dhs.html) ili [**CLI verzija**](https://github.com/pandazheng/DylibHijack).\
Dobar **izveštaj sa tehničkim detaljima** o ovoj tehnici može se pronaći [**ovde**](https://www.virusbulletin.com/virusbulletin/2015/03/dylib-hijacking-os-x).
{% endhint %}

**Primer**

{% content-ref url="macos-dyld-hijacking-and-dyld_insert_libraries.md" %}
[macos-dyld-hijacking-and-dyld\_insert\_libraries.md](macos-dyld-hijacking-and-dyld\_insert\_libraries.md)
{% endcontent-ref %}

## Dlopen Hijacking

{% hint style="danger" %}
Zapamtite da se **prethodna ograničenja validacije biblioteke takođe primenjuju** kako bi se izvele napadi Dlopen hijacking-a.
{% endhint %}

Iz **`man dlopen`**:

* Kada putanja **ne sadrži karakter kosine** (tj. samo je list ime), **dlopen() će vršiti pretragu**. Ako je **`$DYLD_LIBRARY_PATH`** postavljen pri pokretanju, dyld će prvo **tražiti u tom direktorijumu**. Zatim, ako pozivajući mach-o fajl ili glavni izvršni fajl specificira **`LC_RPATH`**, tada će dyld **tražiti u tim** direktorijumima. Zatim, ako je proces **neograničen**, dyld će tražiti u **trenutnom radnom direktorijumu**. Na kraju, za stare binarne fajlove, dyld će pokušati neke rezervne opcije. Ako je **`$DYLD_FALLBACK_LIBRARY_PATH`** postavljen pri pokretanju, dyld će tražiti u **tim direktorijumima**, inače, dyld će tražiti u **`/usr/local/lib/`** (ako je proces neograničen), a zatim u **`/usr/lib/`** (ove informacije su preuzete iz **`man dlopen`**).
1. `$DYLD_LIBRARY_PATH`
2. `LC_RPATH`
3. `CWD`(ako je neograničen)
4. `$DYLD_FALLBACK_LIBRARY_PATH`
5. `/usr/local/lib/` (ako je neograničen)
6. `/usr/lib/`

{% hint style="danger" %}
Ako nema kosina u imenu, postoji 2 načina za izvršenje hakovanja:

* Ako je bilo koji **`LC_RPATH`** **upisiv** (ali se proverava potpis, tako da za ovo takođe treba da binarni fajl bude neograničen)
* Ako je binarni fajl **neograničen** i tada je moguće učitati nešto iz trenutnog radnog direktorijuma (ili zloupotreba jedne od pomenutih env promenljivih)
{% endhint %}

* Kada putanja **izgleda kao putanja framework-a** (npr. `/stuff/foo.framework/foo`), ako je **`$DYLD_FRAMEWORK_PATH`** postavljen pri pokretanju, dyld će prvo tražiti u tom direktorijumu za **delimičnu putanju framework-a** (npr. `foo.framework/foo`). Zatim, dyld će pokušati **nabavljenu putanju onakvu kakva je** (koristeći trenutni radni direktorijum za relativne putanje). Na kraju, za stare binarne fajlove, dyld će pokušati neke rezervne opcije. Ako je **`$DYLD_FALLBACK_FRAMEWORK_PATH`** postavljen pri pokretanju, dyld će tražiti te direktorijume. Inače, tražiće **`/Library/Frameworks`** (na macOS-u ako je proces neograničen), zatim **`/System/Library/Frameworks`**.
1. `$DYLD_FRAMEWORK_PATH`
2. nabavljena putanja (koristeći trenutni radni direktorijum za relativne putanje ako je neograničen)
3. `$DYLD_FALLBACK_FRAMEWORK_PATH`
4. `/Library/Frameworks` (ako je neograničen)
5. `/System/Library/Frameworks`

{% hint style="danger" %}
Ako je putanja framework-a, način za hakovanje bi bio:

* Ako je proces **neograničen**, zloupotreba **relativne putanje iz trenutnog radnog direktorijuma** pomenutih env promenljivih (čak i ako nije rečeno u dokumentaciji da li su DYLD\_\* env promenljive uklonjene ako je proces ograničen)
{% endhint %}

* Kada putanja **sadrži kosinu ali nije putanja framework-a** (tj. puna putanja ili delimična putanja do dylib-a), dlopen() prvo traži (ako je postavljeno) u **`$DYLD_LIBRARY_PATH`** (sa delom lista iz putanje ). Zatim, dyld **pokušava nabavljenu putanju** (koristeći trenutni radni direktorijum za relativne putanje (ali samo za neograničene procese)). Na kraju, za stare binarne fajlove, dyld će pokušati rezervne opcije. Ako je **`$DYLD_FALLBACK_LIBRARY_PATH`** postavljen pri pokretanju, dyld će tražiti u tim direktorijumima, inače, dyld će tražiti u **`/usr/local/lib/`** (ako je proces neograničen), a zatim u **`/usr/lib/`**.
1. `$DYLD_LIBRARY_PATH`
2. nabavljena putanja (koristeći trenutni radni direktorijum za relativne putanje ako je neograničen)
3. `$DYLD_FALLBACK_LIBRARY_PATH`
4. `/usr/local/lib/` (ako je neograničen)
5. `/usr/lib/`

{% hint style="danger" %}
Ako ima kosina u imenu i nije putanja framework-a, način za hakovanje bi bio:

* Ako je binarni fajl **neograničen** i tada je moguće učitati nešto iz trenutnog radnog direktorijuma ili `/usr/local/lib` (ili zloupotreba jedne od pomenutih env promenljivih)
{% endhint %}

{% hint style="info" %}
Napomena: Ne postoje **konfiguracioni fajlovi za kontrolu dlopen pretrage**.

Napomena: Ako je glavni izvršni fajl **set\[ug\]id binarni fajl ili potpisan sa privilegijama**, tada se **sve env promenljive ignorišu**, i može se koristiti samo puna putanja ([proverite ograničenja DYLD\_INSERT\_LIBRARIES](macos-dyld-hijacking-and-dyld\_insert\_libraries.md#check-dyld\_insert\_librery-restrictions) za detaljnije informacije)

Napomena: Apple platforme koriste "univerzalne" fajlove za kombinovanje 32-bitnih i 64-bitnih biblioteka. To znači da ne postoje **posebne putanje za pretragu 32-bitnih i 64-bitnih biblioteka**.

Napomena: Na Apple platformama većina OS dylib-ova je **kombinovana u dyld keš** i ne postoje na disku. Stoga, pozivanje **`stat()`** da bi se proverilo da li OS dylib postoji **neće raditi**. Međutim, **`dlopen_preflight()`** koristi iste korake kao i **`dlopen()`** za pronalaženje kompatibilnog mach-o fajla.
{% endhint %}

**Proverite putanje**

Proverimo sve opcije sa sledećim kodom:
```c
// gcc dlopentest.c -o dlopentest -Wl,-rpath,/tmp/test
#include <dlfcn.h>
#include <stdio.h>

int main(void)
{
void* handle;

fprintf("--- No slash ---\n");
handle = dlopen("just_name_dlopentest.dylib",1);
if (!handle) {
fprintf(stderr, "Error loading: %s\n\n\n", dlerror());
}

fprintf("--- Relative framework ---\n");
handle = dlopen("a/framework/rel_framework_dlopentest.dylib",1);
if (!handle) {
fprintf(stderr, "Error loading: %s\n\n\n", dlerror());
}

fprintf("--- Abs framework ---\n");
handle = dlopen("/a/abs/framework/abs_framework_dlopentest.dylib",1);
if (!handle) {
fprintf(stderr, "Error loading: %s\n\n\n", dlerror());
}

fprintf("--- Relative Path ---\n");
handle = dlopen("a/folder/rel_folder_dlopentest.dylib",1);
if (!handle) {
fprintf(stderr, "Error loading: %s\n\n\n", dlerror());
}

fprintf("--- Abs Path ---\n");
handle = dlopen("/a/abs/folder/abs_folder_dlopentest.dylib",1);
if (!handle) {
fprintf(stderr, "Error loading: %s\n\n\n", dlerror());
}

return 0;
}
```
Ako ga kompajlirate i izvršite, možete videti **gde je svaka biblioteka neuspešno tražena**. Takođe, možete **filtrirati FS zapise**:
```bash
sudo fs_usage | grep "dlopentest"
```
## Hijacking relativnih putanja

Ako je **privilegovani binarni program** (kao što je SUID ili neki binarni program sa moćnim dozvolama) **učitava biblioteku relativne putanje** (na primer koristeći `@executable_path` ili `@loader_path`) i ima **onemogućenu proveru biblioteke**, moguće je premestiti binarni program na lokaciju gde napadač može **izmeniti biblioteku učitanu relativnom putanjom**, i iskoristiti je za ubacivanje koda u proces.

## Uklanjanje `DYLD_*` i `LD_LIBRARY_PATH` env promenljivih

U datoteci `dyld-dyld-832.7.1/src/dyld2.cpp` moguće je pronaći funkciju **`pruneEnvironmentVariables`**, koja će ukloniti bilo koju env promenljivu koja **počinje sa `DYLD_`** i **`LD_LIBRARY_PATH=`**.

Takođe će postaviti na **null** specifično env promenljive **`DYLD_FALLBACK_FRAMEWORK_PATH`** i **`DYLD_FALLBACK_LIBRARY_PATH`** za **suid** i **sgid** binarne programe.

Ova funkcija se poziva iz funkcije **`_main`** iste datoteke ako je ciljani operativni sistem OSX na sledeći način:
```cpp
#if TARGET_OS_OSX
if ( !gLinkContext.allowEnvVarsPrint && !gLinkContext.allowEnvVarsPath && !gLinkContext.allowEnvVarsSharedCache ) {
pruneEnvironmentVariables(envp, &apple);
```
i ovi boolean flagovi se postavljaju u istom fajlu u kodu:
```cpp
#if TARGET_OS_OSX
// support chrooting from old kernel
bool isRestricted = false;
bool libraryValidation = false;
// any processes with setuid or setgid bit set or with __RESTRICT segment is restricted
if ( issetugid() || hasRestrictedSegment(mainExecutableMH) ) {
isRestricted = true;
}
bool usingSIP = (csr_check(CSR_ALLOW_TASK_FOR_PID) != 0);
uint32_t flags;
if ( csops(0, CS_OPS_STATUS, &flags, sizeof(flags)) != -1 ) {
// On OS X CS_RESTRICT means the program was signed with entitlements
if ( ((flags & CS_RESTRICT) == CS_RESTRICT) && usingSIP ) {
isRestricted = true;
}
// Library Validation loosens searching but requires everything to be code signed
if ( flags & CS_REQUIRE_LV ) {
isRestricted = false;
libraryValidation = true;
}
}
gLinkContext.allowAtPaths                = !isRestricted;
gLinkContext.allowEnvVarsPrint           = !isRestricted;
gLinkContext.allowEnvVarsPath            = !isRestricted;
gLinkContext.allowEnvVarsSharedCache     = !libraryValidation || !usingSIP;
gLinkContext.allowClassicFallbackPaths   = !isRestricted;
gLinkContext.allowInsertFailures         = false;
gLinkContext.allowInterposing         	 = true;
```
Ovo zapravo znači da ako je binarni fajl **suid** ili **sgid**, ili ima **RESTRICT** segment u zaglavljima ili je potpisan sa **CS\_RESTRICT** zastavicom, tada je **`!gLinkContext.allowEnvVarsPrint && !gLinkContext.allowEnvVarsPath && !gLinkContext.allowEnvVarsSharedCache`** tačno i okolina varijable su uklonjene.

Imajte na umu da ako je CS\_REQUIRE\_LV tačno, tada varijable neće biti uklonjene, ali će provera validacije biblioteke proveriti da li koriste isti sertifikat kao originalni binarni fajl.

## Provera Restrikcija

### SUID & SGID
```bash
# Make it owned by root and suid
sudo chown root hello
sudo chmod +s hello
# Insert the library
DYLD_INSERT_LIBRARIES=inject.dylib ./hello

# Remove suid
sudo chmod -s hello
```
### Odeljak `__RESTRICT` sa segmentom `__restrict`
```bash
gcc -sectcreate __RESTRICT __restrict /dev/null hello.c -o hello-restrict
DYLD_INSERT_LIBRARIES=inject.dylib ./hello-restrict
```
### Ojačano izvršavanje

Kreirajte novi sertifikat u Keychain-u i koristite ga da potpišete binarni fajl:

{% code overflow="wrap" %}
```bash
# Apply runtime proetction
codesign -s <cert-name> --option=runtime ./hello
DYLD_INSERT_LIBRARIES=inject.dylib ./hello #Library won't be injected

# Apply library validation
codesign -f -s <cert-name> --option=library ./hello
DYLD_INSERT_LIBRARIES=inject.dylib ./hello-signed #Will throw an error because signature of binary and library aren't signed by same cert (signs must be from a valid Apple-signed developer certificate)

# Sign it
## If the signature is from an unverified developer the injection will still work
## If it's from a verified developer, it won't
codesign -f -s <cert-name> inject.dylib
DYLD_INSERT_LIBRARIES=inject.dylib ./hello-signed

# Apply CS_RESTRICT protection
codesign -f -s <cert-name> --option=restrict hello-signed
DYLD_INSERT_LIBRARIES=inject.dylib ./hello-signed # Won't work
```
{% endcode %}

{% hint style="danger" %}
Imajte na umu da čak i ako postoje binarni fajlovi potpisani sa zastavicom **`0x0(none)`**, mogu dobiti dinamički zastavicu **`CS_RESTRICT`** prilikom izvršavanja i stoga ova tehnika neće raditi na njima.

Možete proveriti da li proc ima ovu zastavicu sa (preuzmite [**ovde csops**](https://github.com/axelexic/CSOps)):
```bash
csops -status <pid>
```
## Reference

* [https://theevilbit.github.io/posts/dyld\_insert\_libraries\_dylib\_injection\_in\_macos\_osx\_deep\_dive/](https://theevilbit.github.io/posts/dyld\_insert\_libraries\_dylib\_injection\_in\_macos\_osx\_deep\_dive/)

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**PLANOVE ZA PRIJAVU**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitteru** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
