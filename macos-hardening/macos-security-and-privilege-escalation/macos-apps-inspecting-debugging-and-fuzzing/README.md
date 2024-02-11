# macOS-toepassings - Inspekteer, foutopsporing en Fuzzing

<details>

<summary><strong>Leer AWS-hacking vanaf nul tot held met</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Ander maniere om HackTricks te ondersteun:

* As jy wil sien dat jou **maatskappy geadverteer word in HackTricks** of **HackTricks aflaai in PDF-formaat**, kyk na die [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Kry die [**amptelike PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Ontdek [**The PEASS Family**](https://opensea.io/collection/the-peass-family), ons versameling eksklusiewe [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Sluit aan by die** 💬 [**Discord-groep**](https://discord.gg/hRep4RUj7f) of die [**telegram-groep**](https://t.me/peass) of **volg** ons op **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Deel jou hacktruuks deur PR's in te dien by die** [**HackTricks**](https://github.com/carlospolop/hacktricks) en [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github-repos.

</details>

## Statische Analise

### otool
```bash
otool -L /bin/ls #List dynamically linked libraries
otool -tv /bin/ps #Decompile application
```
### objdump

{% code overflow="wrap" %}
```bash
objdump -m --dylibs-used /bin/ls #List dynamically linked libraries
objdump -m -h /bin/ls # Get headers information
objdump -m --syms /bin/ls # Check if the symbol table exists to get function names
objdump -m --full-contents /bin/ls # Dump every section
objdump -d /bin/ls # Dissasemble the binary
objdump --disassemble-symbols=_hello --x86-asm-syntax=intel toolsdemo #Disassemble a function using intel flavour
```
{% endcode %}

### jtool2

Die instrument kan gebruik word as 'n **vervanging** vir **codesign**, **otool**, en **objdump**, en bied 'n paar ekstra kenmerke. [**Laai dit hier af**](http://www.newosxbook.com/tools/jtool.html) of installeer dit met `brew`.
```bash
# Install
brew install --cask jtool2

jtool2 -l /bin/ls # Get commands (headers)
jtool2 -L /bin/ls # Get libraries
jtool2 -S /bin/ls # Get symbol info
jtool2 -d /bin/ls # Dump binary
jtool2 -D /bin/ls # Decompile binary

# Get signature information
ARCH=x86_64 jtool2 --sig /System/Applications/Automator.app/Contents/MacOS/Automator

# Get MIG information
jtool2 -d __DATA.__const myipc_server | grep MIG
```
### Codesign / ldid

{% hint style="danger" %}
**`Codesign`** kan gevind word in **macOS** terwyl **`ldid`** gevind kan word in **iOS**
{% endhint %}
```bash
# Get signer
codesign -vv -d /bin/ls 2>&1 | grep -E "Authority|TeamIdentifier"

# Check if the app’s contents have been modified
codesign --verify --verbose /Applications/Safari.app

# Get entitlements from the binary
codesign -d --entitlements :- /System/Applications/Automator.app # Check the TCC perms

# Check if the signature is valid
spctl --assess --verbose /Applications/Safari.app

# Sign a binary
codesign -s <cert-name-keychain> toolsdemo

# Get signature info
ldid -h <binary>

# Get entitlements
ldid -e <binary>

# Change entilements
## /tmp/entl.xml is a XML file with the new entitlements to add
ldid -S/tmp/entl.xml <binary>
```
### VerdagtePakket

[**VerdagtePakket**](https://mothersruin.com/software/SuspiciousPackage/get.html) is 'n nuttige hulpmiddel om **.pkg** lêers (installeerders) te ondersoek en te sien wat binne-in is voordat dit geïnstalleer word.\
Hierdie installeerders het `preinstall` en `postinstall` bash-skripte wat malware-skrywers gewoonlik misbruik om die malware te **volhard**.

### hdiutil

Hierdie hulpmiddel maak dit moontlik om Apple-diskbeeldlêers (**.dmg**) te **monteer** om dit te ondersoek voordat enige iets uitgevoer word:
```bash
hdiutil attach ~/Downloads/Firefox\ 58.0.2.dmg
```
Dit sal in `/Volumes` gemonteer word.

### Objective-C

#### Metadata

{% hint style="danger" %}
Let daarop dat programme wat in Objective-C geskryf is, hul klassedeclarasies behou wanneer hulle gekompileer word in [Mach-O-binêre](../macos-files-folders-and-binaries/universal-binaries-and-mach-o-format.md). Sulke klassedeclarasies sluit die naam en tipe van die volgende in:
{% endhint %}

* Die klas
* Die klasmetodes
* Die klasinstansie-variables

Jy kan hierdie inligting kry deur [**class-dump**](https://github.com/nygard/class-dump) te gebruik:
```bash
class-dump Kindle.app
```
Let wel, hierdie name kan versteek word om die omkeer van die binêre kode moeiliker te maak.

#### Funksie-oproep

Wanneer 'n funksie in 'n binêre kode geroep word wat Objective-C gebruik, sal die gekompileerde kode in plaas daarvan die funksie **`objc_msgSend`** oproep. Dit sal die finale funksie oproep:

![](<../../../.gitbook/assets/image (560).png>)

Die parameters wat hierdie funksie verwag, is:

* Die eerste parameter (**self**) is " 'n wyser wat wys na die **instansie van die klas wat die boodskap moet ontvang** ". Of eenvoudig gestel, dit is die objek waarop die metode aangeroep word. As die metode 'n klasmetode is, sal dit 'n instansie van die klasobjek (as geheel) wees, terwyl dit vir 'n instansiemetode sal wys na 'n geïnstantieerde instansie van die klas as 'n objek.
* Die tweede parameter (**op**) is "die selekteerder van die metode wat die boodskap hanteer". Weereens, eenvoudig gestel, is dit net die **naam van die metode**.
* Die oorblywende parameters is enige **waardes wat deur die metode vereis word** (op).

| **Argument**      | **Register**                                                    | **(vir) objc\_msgSend**                                |
| ----------------- | --------------------------------------------------------------- | ------------------------------------------------------ |
| **1ste argument** | **rdi**                                                         | **self: objek waarop die metode aangeroep word**       |
| **2de argument**  | **rsi**                                                         | **op: naam van die metode**                            |
| **3de argument**  | **rdx**                                                         | **1ste argument van die metode**                       |
| **4de argument**  | **rcx**                                                         | **2de argument van die metode**                        |
| **5de argument**  | **r8**                                                          | **3de argument van die metode**                        |
| **6de argument**  | **r9**                                                          | **4de argument van die metode**                        |
| **7de+ argument** | <p><strong>rsp+</strong><br><strong>(op die stapel)</strong></p> | **5de+ argument van die metode**                       |

### Swift

Met Swift-binêre lêers, aangesien daar Objective-C-verenigbaarheid is, kan jy soms verklarings onttrek deur [class-dump](https://github.com/nygard/class-dump/) te gebruik, maar nie altyd nie.

Met die **`jtool -l`** of **`otool -l`** opdraglyne is dit moontlik om verskeie afdelings te vind wat begin met die voorvoegsel **`__swift5`**:
```bash
jtool2 -l /Applications/Stocks.app/Contents/MacOS/Stocks
LC 00: LC_SEGMENT_64              Mem: 0x000000000-0x100000000    __PAGEZERO
LC 01: LC_SEGMENT_64              Mem: 0x100000000-0x100028000    __TEXT
[...]
Mem: 0x100026630-0x100026d54        __TEXT.__swift5_typeref
Mem: 0x100026d60-0x100027061        __TEXT.__swift5_reflstr
Mem: 0x100027064-0x1000274cc        __TEXT.__swift5_fieldmd
Mem: 0x1000274cc-0x100027608        __TEXT.__swift5_capture
[...]
```
Jy kan verdere inligting oor die **inligting wat in hierdie afdeling gestoor word in hierdie blogpos** vind (https://knight.sc/reverse%20engineering/2019/07/17/swift-metadata.html).

Verder, **Swift binaêre lêers kan simbole hê** (byvoorbeeld biblioteke moet simbole stoor sodat sy funksies geroep kan word). Die **simbole het gewoonlik die inligting oor die funksienaam** en attr op 'n lelike manier, so hulle is baie nuttig en daar is "**demanglers"** wat die oorspronklike naam kan kry:
```bash
# Ghidra plugin
https://github.com/ghidraninja/ghidra_scripts/blob/master/swift_demangler.py

# Swift cli
swift demangle
```
### Gepakte binaire lêers

* Kontroleer vir hoë entropie
* Kontroleer die strings (as daar byna geen verstaanbare string is nie, is dit gepak)
* Die UPX-pakker vir MacOS genereer 'n afdeling genaamd "\_\_XHDR"

## Dinamiese Analise

{% hint style="warning" %}
Let daarop dat om binaire lêers te ontleed, **SIP gedeaktiveer moet word** (`csrutil disable` of `csrutil enable --without debug`) of om die binaire lêers na 'n tydelike vouer te kopieer en die handtekening met `codesign --remove-signature <binary-pad>` te verwyder of om die ontleed van die binaire lêer toe te laat (jy kan [hierdie skripsie](https://gist.github.com/carlospolop/a66b8d72bb8f43913c4b5ae45672578b) gebruik)
{% endhint %}

{% hint style="warning" %}
Let daarop dat om **sisteem-binaire lêers** (soos `cloudconfigurationd`) op macOS te **instrumenteer**, moet SIP gedeaktiveer word (net die handtekening verwyder sal nie werk nie).
{% endhint %}

### Vereenigde Logboeke

MacOS genereer baie logboeke wat baie nuttig kan wees wanneer 'n toepassing uitgevoer word om te probeer verstaan **wat dit doen**.

Daar is ook logboeke wat die etiket `<private>` bevat om sekere **gebruikers-** of **rekenaaridentifiseerbare** inligting te **versteek**. Dit is egter moontlik om 'n sertifikaat te installeer om hierdie inligting bekend te maak. Volg die verduidelikings vanaf [**hierdie skakel**](https://superuser.com/questions/1532031/how-to-show-private-data-in-macos-unified-log).

### Hopper

#### Linker paneel

In die linker paneel van Hopper is dit moontlik om die simbole (**Etikette**) van die binaire lêer, die lys van prosedures en funksies (**Proc**) en die strings (**Str**) te sien. Dit is nie al die strings nie, maar diegene wat in verskeie dele van die Mac-O-lêer gedefinieer is (soos _cstring of_ `objc_methname`).

#### Middelste paneel

In die middelste paneel kan jy die **ontleedde kode** sien. En jy kan dit sien as 'n **rof** ontleed, as 'n **grafiek**, as **ontsommel** en as **binêr** deur op die betrokke ikoon te klik:

<figure><img src="../../../.gitbook/assets/image (2) (6).png" alt=""><figcaption></figcaption></figure>

Deur met die rechtermuisknop op 'n kode-object te klik, kan jy **verwysings na/van daardie objek** sien of selfs sy naam verander (dit werk nie in ontsommelde pseudokode nie):

<figure><img src="../../../.gitbook/assets/image (1) (1) (2).png" alt=""><figcaption></figcaption></figure>

Verder kan jy in die **middelste onderste deel Python-opdragte skryf**.

#### Regter paneel

In die regter paneel kan jy interessante inligting sien soos die **navigasiegeskiedenis** (sodat jy weet hoe jy by die huidige situasie uitgekom het), die **oproepgrafiek** waar jy al die **funksies kan sien wat hierdie funksie oproep** en al die funksies wat **hierdie funksie oproep**, en inligting oor **plaaslike veranderlikes**.

### dtrace

Dit stel gebruikers in staat om toepassings op 'n uiters **lae vlak** te benader en bied 'n manier vir gebruikers om **programme te volg** en selfs hul uitvoeringsvloei te **verander**. Dtrace gebruik **sondes** wat **regdeur die kernel geplaas** word en op plekke soos die begin en einde van stelseloproepe is.

DTrace gebruik die **`dtrace_probe_create`**-funksie om 'n sonde vir elke stelseloproep te skep. Hierdie sonde kan by die **intree- en uittreepunt van elke stelseloproep** geaktiveer word. Die interaksie met DTrace vind plaas deur middel van /dev/dtrace wat slegs beskikbaar is vir die root-gebruiker.

{% hint style="success" %}
Om Dtrace te aktiveer sonder om SIP-beskerming heeltemal te deaktiveer, kan jy in herstelmodus uitvoer: `csrutil enable --without dtrace`

Jy kan ook **`dtrace`** of **`dtruss`** binaire lêers wat **jy self saamgestel het**, gebruik.
{% endhint %}

Die beskikbare sonde van dtrace kan verkry word met:
```bash
dtrace -l | head
ID   PROVIDER            MODULE                          FUNCTION NAME
1     dtrace                                                     BEGIN
2     dtrace                                                     END
3     dtrace                                                     ERROR
43    profile                                                     profile-97
44    profile                                                     profile-199
```
Die ondersoeknaam bestaan uit vier dele: die verskaffer, module, funksie en naam (`fbt:mach_kernel:ptrace:entry`). As jy nie 'n deel van die naam spesifiseer nie, sal Dtrace daardie deel as 'n wildcard toepas.

Om DTrace te konfigureer om ondersoeke te aktiveer en om te spesifiseer watter aksies uitgevoer moet word wanneer hulle aktiveer, sal ons die D-taal moet gebruik.

'n Meer gedetailleerde verduideliking en meer voorbeelde kan gevind word in [https://illumos.org/books/dtrace/chp-intro.html](https://illumos.org/books/dtrace/chp-intro.html)

#### Voorbeelde

Voer `man -k dtrace` uit om die **DTrace-skrips beskikbaar** te lys. Voorbeeld: `sudo dtruss -n binary`

* Op lyn
```bash
#Count the number of syscalls of each running process
sudo dtrace -n 'syscall:::entry {@[execname] = count()}'
```
# Skripsie

Hierdie is 'n skripsie oor die inspekteer, foutopsporing en fuzzing van macOS-toepassings. Hierdie tegnieke kan gebruik word om die sekuriteit van macOS-toepassings te verbeter en voorregverhoging te bereik.

## Inspekteer

By die inspekteer van 'n macOS-toepassing, kan jy die toepassing se bronkode, afhanklikhede en ander relevante inligting ontleed. Hier is 'n paar tegnieke wat jy kan gebruik:

### 1. Disassembling

Deur die toepassing te disassembleer, kan jy die masjienkode ontleed en die programlogika verstaan. Jy kan hulpmiddels soos `Hopper Disassembler` of `IDA Pro` gebruik om hierdie taak uit te voer.

### 2. Dynamic Analysis

Deur die toepassing dinamies te analiseer, kan jy sy gedrag tydens uitvoering bestudeer. Jy kan hulpmiddels soos `lldb` of `gdb` gebruik om die toepassing te ontleed en te monitor terwyl dit uitgevoer word.

### 3. Reverse Engineering

Deur die toepassing te herontwerp, kan jy die oorspronklike bronkode herstel. Jy kan hulpmiddels soos `class-dump` of `Hopper Disassembler` gebruik om die toepassing se klassedefinisies en metodes te ontleed.

## Foutopsporing

Foutopsporing is 'n belangrike tegniek om probleme in 'n toepassing te identifiseer en op te los. Hier is 'n paar tegnieke wat jy kan gebruik:

### 1. Logging

Deur logboeke in die toepassing in te skakel, kan jy nuttige inligting oor die toepassing se uitvoering versamel. Jy kan die `NSLog`-funksie gebruik om logboekinskrywings te skep en die `Console`-toepassing gebruik om die logboeke te monitor.

### 2. Breakpoints

Deur breekpunte in die toepassing te plaas, kan jy die uitvoering stop en die toestand van die toepassing ondersoek. Jy kan hulpmiddels soos `lldb` of `gdb` gebruik om breekpunte te plaas en die toepassing te ontleed terwyl dit uitgevoer word.

### 3. Memory Analysis

Deur die geheue van die toepassing te analiseer, kan jy probleme soos geheuelekke of ongeldige geheueverwysings identifiseer. Jy kan hulpmiddels soos `Instruments` of `Valgrind` gebruik om hierdie analise uit te voer.

## Fuzzing

Fuzzing is 'n tegniek wat gebruik word om die toepassing te toets deur ongeldige of lukrake insette te voorsien. Hier is 'n paar tegnieke wat jy kan gebruik:

### 1. Input Mutation

Deur die insette van die toepassing te muteer, kan jy verskillende scenario's simuleer en potensiële probleme identifiseer. Jy kan hulpmiddels soos `AFL` of `Radamsa` gebruik om die insette te muteer.

### 2. Protocol Fuzzing

Deur die kommunikasieprotokolle van die toepassing te fuzz, kan jy die toepassing se reaksie op ongeldige of onverwagte protokolopdragte toets. Jy kan hulpmiddels soos `Sulley` of `Peach` gebruik om hierdie tipe fuzzing uit te voer.

### 3. File Fuzzing

Deur lukrake of ongeldige lêers as insette te gebruik, kan jy die toepassing se hantering van lêers toets. Jy kan hulpmiddels soos `zzuf` of `Atheris` gebruik om hierdie tipe fuzzing uit te voer.

Met hierdie inspekteer-, foutopsporing- en fuzzingtegnieke kan jy die sekuriteit van macOS-toepassings verbeter en voorregverhoging bereik. Dit is belangrik om hierdie tegnieke verantwoordelik en eties te gebruik.
```bash
syscall:::entry
/pid == $1/
{
}

#Log every syscall of a PID
sudo dtrace -s script.d 1234
```

```bash
syscall::open:entry
{
printf("%s(%s)", probefunc, copyinstr(arg0));
}
syscall::close:entry
{
printf("%s(%d)\n", probefunc, arg0);
}

#Log files opened and closed by a process
sudo dtrace -s b.d -c "cat /etc/hosts"
```

```bash
syscall:::entry
{
;
}
syscall:::return
{
printf("=%d\n", arg1);
}

#Log sys calls with values
sudo dtrace -s syscalls_info.d -c "cat /etc/hosts"
```
### dtruss

`dtruss` is a command-line tool available on macOS that allows you to trace and inspect system calls made by an application. It can be used for debugging and analyzing the behavior of macOS applications.

To use `dtruss`, you need to run it with the target application as an argument. It will then display a list of system calls made by the application, along with their arguments and return values. This can be useful for understanding how an application interacts with the operating system and identifying any potential security vulnerabilities or performance issues.

Here is an example of how to use `dtruss`:

```bash
$ sudo dtruss -f -p <PID>
```

In this command, `-f` specifies that `dtruss` should follow child processes, and `-p <PID>` specifies the process ID of the target application. Running `dtruss` with these options will display a live stream of system calls made by the application.

Keep in mind that `dtruss` requires root privileges to run, so you may need to use `sudo` to execute it. Additionally, it is important to only use `dtruss` on applications that you have permission to inspect, as it can potentially expose sensitive information.

Overall, `dtruss` is a powerful tool for inspecting and debugging macOS applications. By tracing system calls, you can gain valuable insights into an application's behavior and identify potential security issues.
```bash
dtruss -c ls #Get syscalls of ls
dtruss -c -p 1000 #get syscalls of PID 1000
```
### ktrace

Jy kan dit selfs gebruik met **SIP geaktiveer**
```bash
ktrace trace -s -S -t c -c ls | grep "ls("
```
### ProcessMonitor

[**ProcessMonitor**](https://objective-see.com/products/utilities.html#ProcessMonitor) is 'n baie nuttige instrument om die prosesverwante aksies wat 'n proses uitvoer te monitor (byvoorbeeld, om te monitor watter nuwe prosesse 'n proses skep).

### SpriteTree

[**SpriteTree**](https://themittenmac.com/tools/) is 'n instrument wat die verhoudings tussen prosesse afdruk.\
Jy moet jou Mac monitor met 'n bevel soos **`sudo eslogger fork exec rename create > cap.json`** (die terminal wat dit lanceer, vereis FDA). En dan kan jy die json in hierdie instrument laai om al die verhoudings te sien:

<figure><img src="../../../.gitbook/assets/image (710).png" alt="" width="375"><figcaption></figcaption></figure>

### FileMonitor

[**FileMonitor**](https://objective-see.com/products/utilities.html#FileMonitor) maak dit moontlik om lêer-gebeure (soos skepping, wysigings en verwyderings) te monitor en bied gedetailleerde inligting oor sulke gebeure.

### Crescendo

[**Crescendo**](https://github.com/SuprHackerSteve/Crescendo) is 'n GUI-instrument met die voorkoms en gevoel wat Windows-gebruikers mag ken van Microsoft Sysinternal se _Procmon_. Hierdie instrument maak dit moontlik om die opname van verskillende tipes gebeure te begin en te stop, maak dit moontlik om hierdie gebeure te filter volgens kategorieë soos lêer, proses, netwerk, ens., en bied die funksionaliteit om die opgeneemde gebeure in 'n json-formaat te stoor.

### Apple Instruments

[**Apple Instruments**](https://developer.apple.com/library/archive/documentation/Performance/Conceptual/CellularBestPractices/Appendix/Appendix.html) is deel van Xcode se Ontwikkelaarshulpmiddels - dit word gebruik om programprestasie te monitor, geheuelekasies te identifiseer en lêersisteemaktiwiteit te volg.

![](<../../../.gitbook/assets/image (15).png>)

### fs\_usage

Maak dit moontlik om aksies wat deur prosesse uitgevoer word, te volg:
```bash
fs_usage -w -f filesys ls #This tracks filesystem actions of proccess names containing ls
fs_usage -w -f network curl #This tracks network actions
```
### TaskExplorer

[**Taskexplorer**](https://objective-see.com/products/taskexplorer.html) is nuttig om die **biblioteke** wat deur 'n binêre lêer gebruik word, die **lêers** wat dit gebruik en die **netwerk**-verbindings te sien.\
Dit kontroleer ook die binêre prosesse teen **virustotal** en wys inligting oor die binêre lêer.

## PT\_DENY\_ATTACH <a href="#page-title" id="page-title"></a>

In [**hierdie blogpos**](https://knight.sc/debugging/2019/06/03/debugging-apple-binaries-that-use-pt-deny-attach.html) kan jy 'n voorbeeld vind oor hoe om 'n lopende daemon te **debug** wat **`PT_DENY_ATTACH`** gebruik om te voorkom dat dit gedebug word, selfs as SIP gedeaktiveer is.

### lldb

**lldb** is die de **facto-hulpmiddel** vir **macOS** binêre **debugging**.
```bash
lldb ./malware.bin
lldb -p 1122
lldb -n malware.bin
lldb -n malware.bin --waitfor
```
Jy kan die intel-flavour instel wanneer jy lldb gebruik deur 'n lêer genaamd **`.lldbinit`** in jou tuisgids te skep met die volgende lyn:
```bash
settings set target.x86-disassembly-flavor intel
```
{% hint style="warning" %}
Binne lldb, dump 'n proses met `process save-core`
{% endhint %}

<table data-header-hidden><thead><tr><th width="225"></th><th></th></tr></thead><tbody><tr><td><strong>(lldb) Opdrag</strong></td><td><strong>Beskrywing</strong></td></tr><tr><td><strong>run (r)</strong></td><td>Begin uitvoering, wat ononderbroke sal voortgaan totdat 'n breekpunt getref word of die proses beëindig word.</td></tr><tr><td><strong>continue (c)</strong></td><td>Gaan voort met die uitvoering van die gedebugde proses.</td></tr><tr><td><strong>nexti (n / ni)</strong></td><td>Voer die volgende instruksie uit. Hierdie opdrag sal oorslaan oor funksie-oproepe.</td></tr><tr><td><strong>stepi (s / si)</strong></td><td>Voer die volgende instruksie uit. In teenstelling met die nexti-opdrag, sal hierdie opdrag in funksie-oproepe stap.</td></tr><tr><td><strong>finish (f)</strong></td><td>Voer die res van die instruksies in die huidige funksie ("raam") uit en hou op.</td></tr><tr><td><strong>control + c</strong></td><td>Onderbreek uitvoering. As die proses uitgevoer (r) of voortgesit (c) is, sal dit die proses laat staan ​​... waar dit tans uitgevoer word.</td></tr><tr><td><strong>breakpoint (b)</strong></td><td><p>b main #Enige funksie genaamd main</p><p>b &#x3C;binname>`main #Hooffunksie van die bin</p><p>b set -n main --shlib &#x3C;lib_name> #Hooffunksie van die aangeduide bin</p><p>b -[NSDictionary objectForKey:]</p><p>b -a 0x0000000100004bd9</p><p>br l #Breekpuntlys</p><p>br e/dis &#x3C;num> #Aktiveer/Deaktiveer breekpunt</p><p>breakpoint delete &#x3C;num></p></td></tr><tr><td><strong>help</strong></td><td><p>help breakpoint #Kry hulp van breekpunt-opdrag</p><p>help memory write #Kry hulp om in die geheue te skryf</p></td></tr><tr><td><strong>reg</strong></td><td><p>reg read</p><p>reg read $rax</p><p>reg read $rax --format &#x3C;<a href="https://lldb.llvm.org/use/variable.html#type-format">formaat</a>></p><p>reg write $rip 0x100035cc0</p></td></tr><tr><td><strong>x/s &#x3C;reg/geheue-adres></strong></td><td>Vertoon die geheue as 'n null-geëindigde string.</td></tr><tr><td><strong>x/i &#x3C;reg/geheue-adres></strong></td><td>Vertoon die geheue as 'n samestellingsinstruksie.</td></tr><tr><td><strong>x/b &#x3C;reg/geheue-adres></strong></td><td>Vertoon die geheue as 'n byte.</td></tr><tr><td><strong>print object (po)</strong></td><td><p>Dit sal die voorwerp wat deur die parameter verwys word, druk</p><p>po $raw</p><p><code>{</code></p><p><code>dnsChanger = {</code></p><p><code>"affiliate" = "";</code></p><p><code>"blacklist_dns" = ();</code></p><p>Merk op dat die meeste van Apple se Objective-C API's of metodes voorwerpe teruggee, en dus vertoon moet word deur middel van die "print object" (po) opdrag. As po nie 'n betekenisvolle uitset lewer nie, gebruik <code>x/b</code></p></td></tr><tr><td><strong>memory</strong></td><td>memory read 0x000....<br>memory read $x0+0xf2a<br>memory write 0x100600000 -s 4 0x41414141 #Skryf AAAA in daardie adres<br>memory write -f s $rip+0x11f+7 "AAAA" #Skryf AAAA in die adres</td></tr><tr><td><strong>disassembly</strong></td><td><p>dis #Ontas huidige funksie</p><p>dis -n &#x3C;funcname> #Ontas funksie</p><p>dis -n &#x3C;funcname> -b &#x3C;basename> #Ontas funksie<br>dis -c 6 #Ontas 6 lyne<br>dis -c 0x100003764 -e 0x100003768 # Van die een adres tot die ander<br>dis -p -c 4 # Begin by huidige adres ontleed</p></td></tr><tr><td><strong>parray</strong></td><td>parray 3 (char **)$x1 #Kontroleer reeks van 3 komponente in x1-reg</td></tr></tbody></table>

{% hint style="info" %}
Wanneer die **`objc_sendMsg`**-funksie geroep word, hou die **rsi**-register die **naam van die metode** as 'n null-geëindigde ("C") string. Om die naam via lldb af te druk, doen die volgende:

`(lldb) x/s $rsi: 0x1000f1576: "startMiningWithPort:password:coreCount:slowMemory:currency:"`

`(lldb) print (char*)$rsi:`\
`(char *) $1 = 0x00000001000f1576 "startMiningWithPort:password:coreCount:slowMemory:currency:"`

`(lldb) reg read $rsi: rsi = 0x00000001000f1576 "startMiningWithPort:password:coreCount:slowMemory:currency:"`
{% endhint %}

### Anti-Dinamiese Analise

#### VM-opsporing

* Die opdrag **`sysctl hw.model`** gee "Mac" terug as die **gasheer 'n MacOS** is, maar iets anders as dit 'n VM is.
* Deur te speel met die waardes van **`hw.logicalcpu`** en **`hw.physicalcpu`** probeer sommige kwaadwillige programme om te bepaal of dit 'n VM is.
* Sommige kwaadwillige programme kan ook **vasstel of die masjien VMware-gebaseer** is op grond van die MAC-adres (00:50:56).
* Dit is ook moontlik om vas te stel of 'n proses gedebugeer word met 'n eenvoudige kode soos:
* `if(P_TRACED == (info.kp_proc.p_flag & P_TRACED)){ //proses word gedebugeer }`
* Dit kan ook die **`ptrace`**-sisteemaanroep met die **`PT_DENY_ATTACH`**-vlag aanroep. Dit **voorkom** dat 'n deb**u**gger kan koppel en naspeur.
* Jy kan nagaan of die **`sysctl`**- of **`ptrace`**-funksie **ingevoer** word (maar die kwaadwillige program kan dit dinamies invoer)
* Soos opgemerk in hierdie uiteensetting, “[Defeating Anti-Debug Techniques: macOS ptrace variants](https://alexomara.com/blog/defeating-anti-debug-techniques-macos-ptrace-variants/)” :\
"_Die boodskap Process # exited with **status = 45 (0x0000002d)** is gewoonlik 'n duidelike teken dat die doelwit van die debuut **PT\_DENY\_ATTACH** gebruik_"
## Fuzzing

### [ReportCrash](https://ss64.com/osx/reportcrash.html)

ReportCrash **ontleed afbrekende prosesse en stoor 'n afbrekingsverslag op die skyf**. 'n Afbrekingsverslag bevat inligting wat 'n ontwikkelaar kan help om die oorsaak van 'n afbreking te diagnoseer.\
Vir toepassings en ander prosesse **wat in die per-gebruiker launchd konteks loop**, loop ReportCrash as 'n LaunchAgent en stoor afbrekingsverslae in die gebruiker se `~/Library/Logs/DiagnosticReports/`\
Vir daemons, ander prosesse **wat in die stelsel launchd konteks loop** en ander bevoorregte prosesse, loop ReportCrash as 'n LaunchDaemon en stoor afbrekingsverslae in die stelsel se `/Library/Logs/DiagnosticReports`

As jy bekommerd is oor afbrekingsverslae **wat na Apple gestuur word**, kan jy dit deaktiveer. As nie, kan afbrekingsverslae nuttig wees om **uit te vind hoe 'n bediener afgebreek het**.
```bash
#To disable crash reporting:
launchctl unload -w /System/Library/LaunchAgents/com.apple.ReportCrash.plist
sudo launchctl unload -w /System/Library/LaunchDaemons/com.apple.ReportCrash.Root.plist

#To re-enable crash reporting:
launchctl load -w /System/Library/LaunchAgents/com.apple.ReportCrash.plist
sudo launchctl load -w /System/Library/LaunchDaemons/com.apple.ReportCrash.Root.plist
```
### Slaap

Terwyl jy in 'n MacOS fuzz, is dit belangrik om te verseker dat die Mac nie gaan slaap nie:

* systemsetup -setsleep Never
* pmset, Sisteemvoorkeure
* [KeepingYouAwake](https://github.com/newmarcel/KeepingYouAwake)

#### SSH Ontkoppel

As jy fuzz via 'n SSH-verbinding, is dit belangrik om seker te maak dat die sessie nie gaan afloop nie. Verander dus die sshd\_config-lêer met:

* TCPKeepAlive Yes
* ClientAliveInterval 0
* ClientAliveCountMax 0
```bash
sudo launchctl unload /System/Library/LaunchDaemons/ssh.plist
sudo launchctl load -w /System/Library/LaunchDaemons/ssh.plist
```
### Interne Handlers

**Kyk na die volgende bladsy** om uit te vind watter toepassing verantwoordelik is vir **die hanteer van die gespesifiseerde skema of protokol:**

{% content-ref url="../macos-file-extension-apps.md" %}
[macos-file-extension-apps.md](../macos-file-extension-apps.md)
{% endcontent-ref %}

### Enumerating Netwerkprosesse

Dit is interessant om prosesse te vind wat netwerkdata bestuur:
```bash
dtrace -n 'syscall::recv*:entry { printf("-> %s (pid=%d)", execname, pid); }' >> recv.log
#wait some time
sort -u recv.log > procs.txt
cat procs.txt
```
Of gebruik `netstat` of `lsof`

### Libgmalloc

<figure><img src="../../../.gitbook/assets/Pasted Graphic 14.png" alt=""><figcaption></figcaption></figure>

{% code overflow="wrap" %}
```bash
lldb -o "target create `which some-binary`" -o "settings set target.env-vars DYLD_INSERT_LIBRARIES=/usr/lib/libgmalloc.dylib" -o "run arg1 arg2" -o "bt" -o "reg read" -o "dis -s \$pc-32 -c 24 -m -F intel" -o "quit"
```
{% endcode %}

### Fuzzers

#### [AFL++](https://github.com/AFLplusplus/AFLplusplus)

Werk vir CLI-hulpmiddels

#### [Litefuzz](https://github.com/sec-tools/litefuzz)

Dit "**werk net"** met macOS GUI-hulpmiddels. Merk op dat sommige macOS-programme spesifieke vereistes het, soos unieke lêernaam, die regte uitbreiding, die lees van lêers uit die sandput (`~/Library/Containers/com.apple.Safari/Data`)...

Voorbeelde:

{% code overflow="wrap" %}
```bash
# iBooks
litefuzz -l -c "/System/Applications/Books.app/Contents/MacOS/Books FUZZ" -i files/epub -o crashes/ibooks -t /Users/test/Library/Containers/com.apple.iBooksX/Data/tmp -x 10 -n 100000 -ez

# -l : Local
# -c : cmdline with FUZZ word (if not stdin is used)
# -i : input directory or file
# -o : Dir to output crashes
# -t : Dir to output runtime fuzzing artifacts
# -x : Tmeout for the run (default is 1)
# -n : Num of fuzzing iterations (default is 1)
# -e : enable second round fuzzing where any crashes found are reused as inputs
# -z : enable malloc debug helpers

# Font Book
litefuzz -l -c "/System/Applications/Font Book.app/Contents/MacOS/Font Book FUZZ" -i input/fonts -o crashes/font-book -x 2 -n 500000 -ez

# smbutil (using pcap capture)
litefuzz -lk -c "smbutil view smb://localhost:4455" -a tcp://localhost:4455 -i input/mac-smb-resp -p -n 100000 -z

# screensharingd (using pcap capture)
litefuzz -s -a tcp://localhost:5900 -i input/screenshared-session --reportcrash screensharingd -p -n 100000
```
{% endcode %}

### Meer Fuzzing MacOS-inligting

* [https://www.youtube.com/watch?v=T5xfL9tEg44](https://www.youtube.com/watch?v=T5xfL9tEg44)
* [https://github.com/bnagy/slides/blob/master/OSXScale.pdf](https://github.com/bnagy/slides/blob/master/OSXScale.pdf)
* [https://github.com/bnagy/francis/tree/master/exploitaben](https://github.com/bnagy/francis/tree/master/exploitaben)
* [https://github.com/ant4g0nist/crashwrangler](https://github.com/ant4g0nist/crashwrangler)

## Verwysings

* [**OS X Incident Response: Scripting and Analysis**](https://www.amazon.com/OS-Incident-Response-Scripting-Analysis-ebook/dp/B01FHOHHVS)
* [**https://www.youtube.com/watch?v=T5xfL9tEg44**](https://www.youtube.com/watch?v=T5xfL9tEg44)
* [**https://taomm.org/vol1/analysis.html**](https://taomm.org/vol1/analysis.html)
* [**The Art of Mac Malware: The Guide to Analyzing Malicious Software**](https://taomm.org/)

<details>

<summary><strong>Leer AWS-hacking van nul tot held met</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Ander maniere om HackTricks te ondersteun:

* As jy jou **maatskappy in HackTricks wil adverteer** of **HackTricks in PDF wil aflaai**, kyk na die [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Kry die [**amptelike PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Ontdek [**The PEASS Family**](https://opensea.io/collection/the-peass-family), ons versameling eksklusiewe [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Sluit aan by die** 💬 [**Discord-groep**](https://discord.gg/hRep4RUj7f) of die [**telegram-groep**](https://t.me/peass) of **volg** ons op **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Deel jou haktruuks deur PR's in te dien by die** [**HackTricks**](https://github.com/carlospolop/hacktricks) en [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github-repos.

</details>
