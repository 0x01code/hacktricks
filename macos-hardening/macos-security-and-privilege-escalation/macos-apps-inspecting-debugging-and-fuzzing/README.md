# Applications macOS - Inspection, débogage et Fuzzing

<details>

<summary><strong>Apprenez le piratage AWS de zéro à héros avec</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (Expert en équipe rouge AWS de HackTricks)</strong></a><strong>!</strong></summary>

Autres façons de soutenir HackTricks :

* Si vous souhaitez voir votre **entreprise annoncée dans HackTricks** ou **télécharger HackTricks en PDF**, consultez les [**PLANS D'ABONNEMENT**](https://github.com/sponsors/carlospolop) !
* Obtenez le [**swag officiel PEASS & HackTricks**](https://peass.creator-spring.com)
* Découvrez [**La famille PEASS**](https://opensea.io/collection/the-peass-family), notre collection exclusive de [**NFT**](https://opensea.io/collection/the-peass-family)
* **Rejoignez le** 💬 [**groupe Discord**](https://discord.gg/hRep4RUj7f) ou le [**groupe Telegram**](https://t.me/peass) ou **suivez-nous** sur **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Partagez vos astuces de piratage en soumettant des PR aux** [**HackTricks**](https://github.com/carlospolop/hacktricks) et [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) dépôts GitHub.

</details>

### [WhiteIntel](https://whiteintel.io)

<figure><img src="../../../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) est un moteur de recherche alimenté par le **dark web** qui offre des fonctionnalités **gratuites** pour vérifier si une entreprise ou ses clients ont été **compromis** par des **logiciels malveillants voleurs**.

Le but principal de WhiteIntel est de lutter contre les prises de contrôle de compte et les attaques de ransomware résultant de logiciels malveillants volant des informations.

Vous pouvez consulter leur site Web et essayer leur moteur **gratuitement** sur :

{% embed url="https://whiteintel.io" %}

***

## Analyse statique

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

L'outil peut être utilisé comme un **remplacement** pour **codesign**, **otool**, et **objdump**, et offre quelques fonctionnalités supplémentaires. [**Téléchargez-le ici**](http://www.newosxbook.com/tools/jtool.html) ou installez-le avec `brew`.
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
**`Codesign`** peut être trouvé dans **macOS** tandis que **`ldid`** peut être trouvé dans **iOS**
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
### SuspiciousPackage

[**SuspiciousPackage**](https://mothersruin.com/software/SuspiciousPackage/get.html) est un outil utile pour inspecter les fichiers **.pkg** (installateurs) et voir ce qu'ils contiennent avant de les installer.\
Ces installateurs ont des scripts bash `preinstall` et `postinstall` que les auteurs de logiciels malveillants utilisent généralement pour **persister** **le** **logiciel malveillant**.

### hdiutil

Cet outil permet de **monter** des images disque Apple (**.dmg**) pour les inspecter avant d'exécuter quoi que ce soit:
```bash
hdiutil attach ~/Downloads/Firefox\ 58.0.2.dmg
```
Il sera monté dans `/Volumes`

### Objective-C

#### Métadonnées

{% hint style="danger" %}
Notez que les programmes écrits en Objective-C **conservent** leurs déclarations de classe **lorsqu'ils sont** **compilés** en [binaires Mach-O](../macos-files-folders-and-binaries/universal-binaries-and-mach-o-format.md). Ces déclarations de classe **incluent** le nom et le type de :
{% endhint %}

* La classe
* Les méthodes de classe
* Les variables d'instance de classe

Vous pouvez obtenir ces informations en utilisant [**class-dump**](https://github.com/nygard/class-dump) :
```bash
class-dump Kindle.app
```
#### Appel de fonction

Lorsqu'une fonction est appelée dans un binaire qui utilise Objective-C, le code compilé au lieu d'appeler cette fonction, appellera **`objc_msgSend`**. Qui appellera la fonction finale :

![](<../../../.gitbook/assets/image (305).png>)

Les paramètres que cette fonction attend sont :

- Le premier paramètre (**self**) est "un pointeur qui pointe vers l'**instance de la classe qui doit recevoir le message**". Ou plus simplement, c'est l'objet sur lequel la méthode est invoquée. Si la méthode est une méthode de classe, il s'agira d'une instance de l'objet de la classe (dans son ensemble), tandis que pour une méthode d'instance, self pointera vers une instance instanciée de la classe en tant qu'objet.
- Le deuxième paramètre, (**op**), est "le sélecteur de la méthode qui gère le message". Encore une fois, plus simplement, il s'agit simplement du **nom de la méthode**.
- Les paramètres restants sont toutes les **valeurs requises par la méthode** (op).

Voyez comment **obtenir ces informations facilement avec `lldb` en ARM64** sur cette page :

{% content-ref url="arm64-basic-assembly.md" %}
[arm64-basic-assembly.md](arm64-basic-assembly.md)
{% endcontent-ref %}

x64 :

| **Argument**      | **Registre**                                                   | **(pour) objc\_msgSend**                                |
| ----------------- | -------------------------------------------------------------- | ------------------------------------------------------ |
| **1er argument**  | **rdi**                                                        | **self : objet sur lequel la méthode est invoquée**    |
| **2e argument**   | **rsi**                                                        | **op : nom de la méthode**                             |
| **3e argument**   | **rdx**                                                        | **1er argument de la méthode**                         |
| **4e argument**   | **rcx**                                                        | **2e argument de la méthode**                          |
| **5e argument**   | **r8**                                                         | **3e argument de la méthode**                          |
| **6e argument**   | **r9**                                                         | **4e argument de la méthode**                          |
| **7e+ argument**  | <p><strong>rsp+</strong><br><strong>(sur la pile)</strong></p> | **5e+ argument de la méthode**                         |

### Dynadump

[**Dynadump**](https://github.com/DerekSelander/dynadump) est un outil pour obtenir des Objc-Classes à partir de dylibs.

### Swift

Avec les binaires Swift, étant donné qu'il y a une compatibilité Objective-C, parfois vous pouvez extraire des déclarations en utilisant [class-dump](https://github.com/nygard/class-dump/) mais pas toujours.

Avec les lignes de commande **`jtool -l`** ou **`otool -l`** il est possible de trouver plusieurs sections qui commencent par le préfixe **`__swift5`** :
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
Vous pouvez trouver plus d'informations sur les [**informations stockées dans cette section dans cet article de blog**](https://knight.sc/reverse%20engineering/2019/07/17/swift-metadata.html).

De plus, **les binaires Swift peuvent avoir des symboles** (par exemple, les bibliothèques doivent stocker des symboles pour que leurs fonctions puissent être appelées). Les **symboles ont généralement des informations sur le nom de la fonction** et les attributs de manière peu lisible, donc ils sont très utiles et il existe des "**désassembleurs**" qui peuvent obtenir le nom original:
```bash
# Ghidra plugin
https://github.com/ghidraninja/ghidra_scripts/blob/master/swift_demangler.py

# Swift cli
swift demangle
```
### Binaires packagés

* Vérifiez l'entropie élevée
* Vérifiez les chaînes (s'il n'y a presque aucune chaîne compréhensible, c'est empaqueté)
* Le packer UPX pour MacOS génère une section appelée "\_\_XHDR"

## Analyse dynamique

{% hint style="warning" %}
Notez que pour déboguer les binaires, **SIP doit être désactivé** (`csrutil disable` ou `csrutil enable --without debug`) ou copier les binaires dans un dossier temporaire et **supprimer la signature** avec `codesign --remove-signature <chemin-binaire>` ou autoriser le débogage du binaire (vous pouvez utiliser [ce script](https://gist.github.com/carlospolop/a66b8d72bb8f43913c4b5ae45672578b))
{% endhint %}

{% hint style="warning" %}
Notez que pour **instrumenter les binaires système**, (comme `cloudconfigurationd`) sur macOS, **SIP doit être désactivé** (simplement supprimer la signature ne fonctionnera pas).
{% endhint %}

### APIs

macOS expose quelques APIs intéressantes qui fournissent des informations sur les processus :

* `proc_info` : C'est le principal qui donne beaucoup d'informations sur chaque processus. Vous devez être root pour obtenir des informations sur d'autres processus mais vous n'avez pas besoin d'entitlements spéciaux ou de ports mach.
* `libsysmon.dylib` : Il permet d'obtenir des informations sur les processus via des fonctions exposées par XPC, cependant, il est nécessaire d'avoir l'entitlement `com.apple.sysmond.client`.

### Stackshot & microstackshots

**Stackshotting** est une technique utilisée pour capturer l'état des processus, y compris les piles d'appels de tous les threads en cours d'exécution. Cela est particulièrement utile pour le débogage, l'analyse des performances et la compréhension du comportement du système à un moment donné. Sur iOS et macOS, le stackshotting peut être effectué à l'aide de plusieurs outils et méthodes comme les outils **`sample`** et **`spindump`**.

### Sysdiagnose

Cet outil (`/usr/bini/ysdiagnose`) collecte essentiellement de nombreuses informations de votre ordinateur en exécutant des dizaines de commandes différentes telles que `ps`, `zprint`...

Il doit être exécuté en tant que **root** et le démon `/usr/libexec/sysdiagnosed` a des entitlements très intéressants tels que `com.apple.system-task-ports` et `get-task-allow`.

Son plist est situé dans `/System/Library/LaunchDaemons/com.apple.sysdiagnose.plist` qui déclare 3 MachServices :

* `com.apple.sysdiagnose.CacheDelete` : Supprime les anciennes archives dans /var/rmp
* `com.apple.sysdiagnose.kernel.ipc` : Port spécial 23 (noyau)
* `com.apple.sysdiagnose.service.xpc` : Interface en mode utilisateur via la classe `Libsysdiagnose` Obj-C. Trois arguments dans un dictionnaire peuvent être passés (`compress`, `display`, `run`)

### Journaux unifiés

MacOS génère de nombreux journaux qui peuvent être très utiles lors de l'exécution d'une application pour comprendre **ce qu'elle fait**.

De plus, il existe des journaux qui contiendront la balise `<private>` pour **masquer** certaines informations **identifiables par l'utilisateur** ou **l'ordinateur**. Cependant, il est possible d'**installer un certificat pour divulguer ces informations**. Suivez les explications à partir de [**ici**](https://superuser.com/questions/1532031/how-to-show-private-data-in-macos-unified-log).

### Hopper

#### Panneau de gauche

Dans le panneau de gauche de Hopper, il est possible de voir les symboles (**Labels**) du binaire, la liste des procédures et fonctions (**Proc**) et les chaînes (**Str**). Ce ne sont pas toutes les chaînes mais celles définies dans plusieurs parties du fichier Mac-O (comme _cstring ou_ `objc_methname`).

#### Panneau central

Dans le panneau central, vous pouvez voir le **code désassemblé**. Et vous pouvez le voir en désassemblage **brut**, en **graphique**, en **décompilé** et en **binaire** en cliquant sur l'icône respective :

<figure><img src="../../../.gitbook/assets/image (343).png" alt=""><figcaption></figcaption></figure>

En cliquant avec le bouton droit sur un objet de code, vous pouvez voir les **références vers/depuis cet objet** ou même changer son nom (cela ne fonctionne pas dans le pseudocode décompilé) :

<figure><img src="../../../.gitbook/assets/image (1117).png" alt=""><figcaption></figcaption></figure>

De plus, dans le **milieu en bas, vous pouvez écrire des commandes python**.

#### Panneau de droite

Dans le panneau de droite, vous pouvez voir des informations intéressantes telles que l'**historique de navigation** (pour savoir comment vous êtes arrivé à la situation actuelle), le **graphique d'appel** où vous pouvez voir toutes les **fonctions qui appellent cette fonction** et toutes les fonctions que **cette fonction appelle**, et des informations sur les **variables locales**.

### dtrace

Il permet aux utilisateurs d'accéder aux applications à un niveau extrêmement **bas** et offre un moyen aux utilisateurs de **tracer** les **programmes** et même de modifier leur flux d'exécution. Dtrace utilise des **sondes** qui sont **placées dans tout le noyau** et se trouvent à des emplacements tels que le début et la fin des appels système.

DTrace utilise la fonction **`dtrace_probe_create`** pour créer une sonde pour chaque appel système. Ces sondes peuvent être déclenchées au **point d'entrée et de sortie de chaque appel système**. L'interaction avec DTrace se fait via /dev/dtrace qui n'est disponible que pour l'utilisateur root.

{% hint style="success" %}
Pour activer Dtrace sans désactiver complètement la protection SIP, vous pouvez exécuter en mode de récupération : `csrutil enable --without dtrace`

Vous pouvez également **`dtrace`** ou **`dtruss`** des binaires que **vous avez compilés**.
{% endhint %}

Les sondes disponibles de dtrace peuvent être obtenues avec :
```bash
dtrace -l | head
ID   PROVIDER            MODULE                          FUNCTION NAME
1     dtrace                                                     BEGIN
2     dtrace                                                     END
3     dtrace                                                     ERROR
43    profile                                                     profile-97
44    profile                                                     profile-199
```
Le nom de la sonde se compose de quatre parties : le fournisseur, le module, la fonction et le nom (`fbt:mach_kernel:ptrace:entry`). Si vous ne spécifiez pas une partie du nom, Dtrace appliquera cette partie comme un joker.

Pour configurer DTrace afin d'activer les sondes et de spécifier les actions à effectuer lorsqu'elles se déclenchent, nous devrons utiliser le langage D.

Une explication plus détaillée et plus d'exemples peuvent être trouvés dans [https://illumos.org/books/dtrace/chp-intro.html](https://illumos.org/books/dtrace/chp-intro.html)

#### Exemples

Exécutez `man -k dtrace` pour lister les **scripts DTrace disponibles**. Exemple : `sudo dtruss -n binary`

* En ligne
```bash
#Count the number of syscalls of each running process
sudo dtrace -n 'syscall:::entry {@[execname] = count()}'
```
* script
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
```bash
dtruss -c ls #Get syscalls of ls
dtruss -c -p 1000 #get syscalls of PID 1000
```
### kdebug

Il s'agit d'une facilité de traçage du noyau. Les codes documentés peuvent être trouvés dans **`/usr/share/misc/trace.codes`**.

Des outils comme `latency`, `sc_usage`, `fs_usage` et `trace` l'utilisent en interne.

Pour interagir avec `kdebug`, `sysctl` est utilisé sur l'espace de noms `kern.kdebug` et les MIB à utiliser peuvent être trouvés dans `sys/sysctl.h` ayant les fonctions implémentées dans `bsd/kern/kdebug.c`.

Pour interagir avec kdebug avec un client personnalisé, voici généralement les étapes à suivre :

* Supprimer les paramètres existants avec KERN\_KDSETREMOVE
* Définir la trace avec KERN\_KDSETBUF et KERN\_KDSETUP
* Utiliser KERN\_KDGETBUF pour obtenir le nombre d'entrées de tampon
* Récupérer son propre client de la trace avec KERN\_KDPINDEX
* Activer le traçage avec KERN\_KDENABLE
* Lire le tampon en appelant KERN\_KDREADTR
* Pour faire correspondre chaque thread avec son processus, appeler KERN\_KDTHRMAP.

Pour obtenir ces informations, il est possible d'utiliser l'outil Apple **`trace`** ou l'outil personnalisé [kDebugView (kdv)](https://newosxbook.com/tools/kdv.html)**.**

**Notez que Kdebug n'est disponible que pour un client à la fois.** Ainsi, un seul outil alimenté par k-debug peut être exécuté à la fois.

### ktrace

Les API `ktrace_*` proviennent de `libktrace.dylib` qui enveloppent celles de `Kdebug`. Ensuite, un client peut simplement appeler `ktrace_session_create` et `ktrace_events_[single/class]` pour définir des rappels sur des codes spécifiques, puis le démarrer avec `ktrace_start`.

Vous pouvez utiliser celui-ci même avec **SIP activé**

Vous pouvez utiliser en tant que clients l'utilitaire `ktrace`:
```bash
ktrace trace -s -S -t c -c ls | grep "ls("
```
Ou `tailspin`.

### kperf

Cela est utilisé pour effectuer un profilage au niveau du noyau et est construit en utilisant des appels `Kdebug`.

En gros, la variable globale `kernel_debug_active` est vérifiée et si elle est définie, elle appelle `kperf_kdebug_handler` avec le code `Kdebug` et l'adresse du cadre du noyau appelant. Si le code `Kdebug` correspond à l'un sélectionné, il obtient les "actions" configurées sous forme de bitmap (vérifiez `osfmk/kperf/action.h` pour les options).

Kperf dispose également d'une table MIB sysctl : (en tant que root) `sysctl kperf`. Ces codes peuvent être trouvés dans `osfmk/kperf/kperfbsd.c`.

De plus, un sous-ensemble de la fonctionnalité de Kperf réside dans `kpc`, qui fournit des informations sur les compteurs de performance de la machine.

### ProcessMonitor

[**ProcessMonitor**](https://objective-see.com/products/utilities.html#ProcessMonitor) est un outil très utile pour vérifier les actions liées aux processus qu'un processus effectue (par exemple, surveiller les nouveaux processus qu'un processus crée).

### SpriteTree

[**SpriteTree**](https://themittenmac.com/tools/) est un outil qui affiche les relations entre les processus.\
Vous devez surveiller votre Mac avec une commande comme **`sudo eslogger fork exec rename create > cap.json`** (le terminal lançant cela nécessite FDA). Ensuite, vous pouvez charger le json dans cet outil pour visualiser toutes les relations :

<figure><img src="../../../.gitbook/assets/image (1182).png" alt="" width="375"><figcaption></figcaption></figure>

### FileMonitor

[**FileMonitor**](https://objective-see.com/products/utilities.html#FileMonitor) permet de surveiller les événements de fichiers (comme les créations, modifications et suppressions) en fournissant des informations détaillées sur ces événements.

### Crescendo

[**Crescendo**](https://github.com/SuprHackerSteve/Crescendo) est un outil GUI avec l'apparence que les utilisateurs de Windows peuvent connaître de _Procmon_ de Microsoft Sysinternal. Cet outil permet de démarrer et d'arrêter l'enregistrement de différents types d'événements, de filtrer ces événements par catégories telles que fichier, processus, réseau, etc., et offre la fonctionnalité de sauvegarder les événements enregistrés au format json.

### Apple Instruments

[**Apple Instruments**](https://developer.apple.com/library/archive/documentation/Performance/Conceptual/CellularBestPractices/Appendix/Appendix.html) font partie des outils de développement Xcode - utilisés pour surveiller les performances des applications, identifier les fuites de mémoire et suivre l'activité du système de fichiers.

![](<../../../.gitbook/assets/image (1138).png>)

### fs\_usage

Permet de suivre les actions effectuées par les processus :
```bash
fs_usage -w -f filesys ls #This tracks filesystem actions of proccess names containing ls
fs_usage -w -f network curl #This tracks network actions
```
### TaskExplorer

[**Taskexplorer**](https://objective-see.com/products/taskexplorer.html) est utile pour voir les **bibliothèques** utilisées par un binaire, les **fichiers** qu'il utilise et les **connexions réseau**.\
Il vérifie également les processus binaires avec **virustotal** et affiche des informations sur le binaire.

## PT\_DENY\_ATTACH <a href="#page-title" id="page-title"></a>

Dans [**cet article de blog**](https://knight.sc/debugging/2019/06/03/debugging-apple-binaries-that-use-pt-deny-attach.html), vous pouvez trouver un exemple sur la façon de **déboguer un démon en cours d'exécution** qui utilise **`PT_DENY_ATTACH`** pour empêcher le débogage même si SIP était désactivé.

### lldb

**lldb** est l'outil de **débogage** binaire **macOS** de facto.
```bash
lldb ./malware.bin
lldb -p 1122
lldb -n malware.bin
lldb -n malware.bin --waitfor
```
Vous pouvez définir le style Intel lors de l'utilisation de lldb en créant un fichier appelé **`.lldbinit`** dans votre dossier personnel avec la ligne suivante :
```bash
settings set target.x86-disassembly-flavor intel
```
{% hint style="warning" %}
À l'intérieur de lldb, dump un processus avec `process save-core`
{% endhint %}

<table data-header-hidden><thead><tr><th width="225"></th><th></th></tr></thead><tbody><tr><td><strong>(lldb) Commande</strong></td><td><strong>Description</strong></td></tr><tr><td><strong>run (r)</strong></td><td>Démarrer l'exécution, qui se poursuivra jusqu'à ce qu'un point d'arrêt soit atteint ou que le processus se termine.</td></tr><tr><td><strong>continue (c)</strong></td><td>Continuer l'exécution du processus en cours de débogage.</td></tr><tr><td><strong>nexti (n / ni)</strong></td><td>Exécuter l'instruction suivante. Cette commande sautera les appels de fonction.</td></tr><tr><td><strong>stepi (s / si)</strong></td><td>Exécuter l'instruction suivante. Contrairement à la commande nexti, cette commande entrera dans les appels de fonction.</td></tr><tr><td><strong>finish (f)</strong></td><td>Exécuter le reste des instructions dans la fonction actuelle ("frame") et s'arrêter.</td></tr><tr><td><strong>control + c</strong></td><td>Mettre en pause l'exécution. Si le processus a été exécuté (r) ou continué (c), cela provoquera l'arrêt du processus ... où qu'il soit en train d'exécuter.</td></tr><tr><td><strong>breakpoint (b)</strong></td><td><p>b main #Toute fonction appelée main</p><p>b &#x3C;nom_du_binaire>`main #Fonction principale du binaire</p><p>b set -n main --shlib &#x3C;nom_lib> #Fonction principale du binaire indiqué</p><p>b -[NSDictionary objectForKey:]</p><p>b -a 0x0000000100004bd9</p><p>br l #Liste des points d'arrêt</p><p>br e/dis &#x3C;num> #Activer/Désactiver le point d'arrêt</p><p>breakpoint delete &#x3C;num></p></td></tr><tr><td><strong>help</strong></td><td><p>help breakpoint #Obtenir de l'aide sur la commande de point d'arrêt</p><p>help memory write #Obtenir de l'aide pour écrire dans la mémoire</p></td></tr><tr><td><strong>reg</strong></td><td><p>reg read</p><p>reg read $rax</p><p>reg read $rax --format &#x3C;<a href="https://lldb.llvm.org/use/variable.html#type-format">format</a>></p><p>reg write $rip 0x100035cc0</p></td></tr><tr><td><strong>x/s &#x3C;adresse_reg/mémoire></strong></td><td>Afficher la mémoire sous forme de chaîne terminée par un caractère nul.</td></tr><tr><td><strong>x/i &#x3C;adresse_reg/mémoire></strong></td><td>Afficher la mémoire sous forme d'instruction d'assemblage.</td></tr><tr><td><strong>x/b &#x3C;adresse_reg/mémoire></strong></td><td>Afficher la mémoire sous forme d'octet.</td></tr><tr><td><strong>print object (po)</strong></td><td><p>Cela affichera l'objet référencé par le paramètre</p><p>po $raw</p><p><code>{</code></p><p><code>dnsChanger = {</code></p><p><code>"affiliate" = "";</code></p><p><code>"blacklist_dns" = ();</code></p><p>Remarquez que la plupart des API ou méthodes Objective-C d'Apple renvoient des objets et doivent donc être affichées via la commande "print object" (po). Si po ne produit pas de sortie significative, utilisez <code>x/b</code></p></td></tr><tr><td><strong>memory</strong></td><td>memory read 0x000....<br>memory read $x0+0xf2a<br>memory write 0x100600000 -s 4 0x41414141 #Écrire AAAA à cette adresse<br>memory write -f s $rip+0x11f+7 "AAAA" #Écrire AAAA à l'adresse</td></tr><tr><td><strong>disassembly</strong></td><td><p>dis #Désassembler la fonction actuelle</p><p>dis -n &#x3C;nom_de_la_fonction> #Désassembler la fonction</p><p>dis -n &#x3C;nom_de_la_fonction> -b &#x3C;nom_de_base> #Désassembler la fonction<br>dis -c 6 #Désassembler 6 lignes<br>dis -c 0x100003764 -e 0x100003768 # De une adresse à l'autre<br>dis -p -c 4 # Commencer à l'adresse actuelle à désassembler</p></td></tr><tr><td><strong>parray</strong></td><td>parray 3 (char **)$x1 # Vérifier le tableau de 3 composants dans le registre x1</td></tr></tbody></table>

{% hint style="info" %}
Lors de l'appel de la fonction **`objc_sendMsg`**, le registre **rsi** contient le **nom de la méthode** sous forme de chaîne terminée par un caractère nul ("C"). Pour afficher le nom via lldb, faites :

`(lldb) x/s $rsi: 0x1000f1576: "startMiningWithPort:password:coreCount:slowMemory:currency:"`

`(lldb) print (char*)$rsi:`\
`(char *) $1 = 0x00000001000f1576 "startMiningWithPort:password:coreCount:slowMemory:currency:"`

`(lldb) reg read $rsi: rsi = 0x00000001000f1576 "startMiningWithPort:password:coreCount:slowMemory:currency:"`
{% endhint %}

### Analyse dynamique anti

#### Détection de VM

* La commande **`sysctl hw.model`** renvoie "Mac" lorsque l'**hôte est un MacOS** mais quelque chose de différent lorsqu'il s'agit d'une VM.
* En jouant avec les valeurs de **`hw.logicalcpu`** et **`hw.physicalcpu`**, certains malwares tentent de détecter s'il s'agit d'une VM.
* Certains malwares peuvent également **détecter** si la machine est basée sur **VMware** en fonction de l'adresse MAC (00:50:56).
* Il est également possible de savoir si un processus est en cours de débogage avec un code simple tel que :
* `if(P_TRACED == (info.kp_proc.p_flag & P_TRACED)){ //processus en cours de débogage }`
* Il est également possible d'invoquer l'appel système **`ptrace`** avec le drapeau **`PT_DENY_ATTACH`**. Cela **empêche** un déb**u**ggeur de s'attacher et de tracer.
* Vous pouvez vérifier si la fonction **`sysctl`** ou **`ptrace`** est **importée** (mais le malware pourrait l'importer dynamiquement)
* Comme indiqué dans cet article, “[Déjouer les techniques anti-debug : variantes de ptrace sur macOS](https://alexomara.com/blog/defeating-anti-debug-techniques-macos-ptrace-variants/)” :\
"_Le message Processus # a quitté avec un **statut = 45 (0x0000002d)** est généralement un signe révélateur que la cible de débogage utilise **PT\_DENY\_ATTACH**_"
## Vidages mémoire

Les vidages mémoire sont créés si :

- `kern.coredump` sysctl est défini sur 1 (par défaut)
- Si le processus n'était pas suid/sgid ou si `kern.sugid_coredump` est à 1 (par défaut, c'est 0)
- La limite `AS_CORE` autorise l'opération. Il est possible de supprimer la création de vidages mémoire en appelant `ulimit -c 0` et de les réactiver avec `ulimit -c unlimited`.

Dans ces cas, les vidages mémoire sont générés selon `kern.corefile` sysctl et sont généralement stockés dans `/cores/core/.%P`.

## Fuzzing

### [ReportCrash](https://ss64.com/osx/reportcrash.html)

ReportCrash **analyse les processus qui plantent et enregistre un rapport de plantage sur le disque**. Un rapport de plantage contient des informations qui peuvent **aider un développeur à diagnostiquer** la cause d'un plantage.\
Pour les applications et autres processus **fonctionnant dans le contexte de lancement par utilisateur**, ReportCrash s'exécute en tant que LaunchAgent et enregistre les rapports de plantage dans `~/Library/Logs/DiagnosticReports/` de l'utilisateur.\
Pour les démons, les autres processus **fonctionnant dans le contexte de lancement système** et les autres processus privilégiés, ReportCrash s'exécute en tant que LaunchDaemon et enregistre les rapports de plantage dans `/Library/Logs/DiagnosticReports` du système.

Si vous êtes préoccupé par les rapports de plantage **envoyés à Apple**, vous pouvez les désactiver. Sinon, les rapports de plantage peuvent être utiles pour **déterminer comment un serveur a planté**.
```bash
#To disable crash reporting:
launchctl unload -w /System/Library/LaunchAgents/com.apple.ReportCrash.plist
sudo launchctl unload -w /System/Library/LaunchDaemons/com.apple.ReportCrash.Root.plist

#To re-enable crash reporting:
launchctl load -w /System/Library/LaunchAgents/com.apple.ReportCrash.plist
sudo launchctl load -w /System/Library/LaunchDaemons/com.apple.ReportCrash.Root.plist
```
### Sommeil

Lors du fuzzing dans un MacOS, il est important de ne pas permettre au Mac de se mettre en veille :

* systemsetup -setsleep Never
* pmset, Préférences Système
* [KeepingYouAwake](https://github.com/newmarcel/KeepingYouAwake)

#### Déconnexion SSH

Si vous effectuez du fuzzing via une connexion SSH, il est important de s'assurer que la session ne se termine pas. Modifiez donc le fichier sshd\_config avec :

* TCPKeepAlive Yes
* ClientAliveInterval 0
* ClientAliveCountMax 0
```bash
sudo launchctl unload /System/Library/LaunchDaemons/ssh.plist
sudo launchctl load -w /System/Library/LaunchDaemons/ssh.plist
```
### Gestionnaires internes

**Consultez la page suivante** pour découvrir comment vous pouvez trouver quelle application est responsable de **la gestion du schéma ou du protocole spécifié :**

{% content-ref url="../macos-file-extension-apps.md" %}
[macos-file-extension-apps.md](../macos-file-extension-apps.md)
{% endcontent-ref %}

### Énumération des processus réseau
```bash
dtrace -n 'syscall::recv*:entry { printf("-> %s (pid=%d)", execname, pid); }' >> recv.log
#wait some time
sort -u recv.log > procs.txt
cat procs.txt
```
Ou utilisez `netstat` ou `lsof`

### Libgmalloc

<figure><img src="../../../.gitbook/assets/Pasted Graphic 14.png" alt=""><figcaption></figcaption></figure>

{% code overflow="wrap" %}
```bash
lldb -o "target create `which some-binary`" -o "settings set target.env-vars DYLD_INSERT_LIBRARIES=/usr/lib/libgmalloc.dylib" -o "run arg1 arg2" -o "bt" -o "reg read" -o "dis -s \$pc-32 -c 24 -m -F intel" -o "quit"
```
{% endcode %}

### Fuzzers

#### [AFL++](https://github.com/AFLplusplus/AFLplusplus)

Fonctionne pour les outils en ligne de commande

#### [Litefuzz](https://github.com/sec-tools/litefuzz)

Il "**fonctionne simplement"** avec les outils GUI macOS. Notez que certaines applications macOS ont des exigences spécifiques comme des noms de fichiers uniques, la bonne extension, la nécessité de lire les fichiers à partir du bac à sable (`~/Library/Containers/com.apple.Safari/Data`)...

Quelques exemples:

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

### Plus d'informations sur le fuzzing MacOS

* [https://www.youtube.com/watch?v=T5xfL9tEg44](https://www.youtube.com/watch?v=T5xfL9tEg44)
* [https://github.com/bnagy/slides/blob/master/OSXScale.pdf](https://github.com/bnagy/slides/blob/master/OSXScale.pdf)
* [https://github.com/bnagy/francis/tree/master/exploitaben](https://github.com/bnagy/francis/tree/master/exploitaben)
* [https://github.com/ant4g0nist/crashwrangler](https://github.com/ant4g0nist/crashwrangler)

## Références

* [**OS X Incident Response: Scripting and Analysis**](https://www.amazon.com/OS-Incident-Response-Scripting-Analysis-ebook/dp/B01FHOHHVS)
* [**https://www.youtube.com/watch?v=T5xfL9tEg44**](https://www.youtube.com/watch?v=T5xfL9tEg44)
* [**https://taomm.org/vol1/analysis.html**](https://taomm.org/vol1/analysis.html)
* [**The Art of Mac Malware: The Guide to Analyzing Malicious Software**](https://taomm.org/)

### [WhiteIntel](https://whiteintel.io)

<figure><img src="../../../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) est un moteur de recherche alimenté par le **dark web** qui offre des fonctionnalités **gratuites** pour vérifier si une entreprise ou ses clients ont été **compromis** par des **logiciels malveillants voleurs**.

Le but principal de WhiteIntel est de lutter contre les prises de contrôle de compte et les attaques de ransomware résultant de logiciels malveillants volant des informations.

Vous pouvez consulter leur site Web et essayer leur moteur **gratuitement** sur :

{% embed url="https://whiteintel.io" %}

<details>

<summary><strong>Apprenez le piratage AWS de zéro à héros avec</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Autres façons de soutenir HackTricks :

* Si vous souhaitez voir votre **entreprise annoncée dans HackTricks** ou **télécharger HackTricks en PDF**, consultez les [**PLANS D'ABONNEMENT**](https://github.com/sponsors/carlospolop) !
* Obtenez le [**swag officiel PEASS & HackTricks**](https://peass.creator-spring.com)
* Découvrez [**The PEASS Family**](https://opensea.io/collection/the-peass-family), notre collection exclusive de [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Rejoignez le** 💬 [**groupe Discord**](https://discord.gg/hRep4RUj7f) ou le [**groupe Telegram**](https://t.me/peass) ou **suivez** nous sur **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Partagez vos astuces de piratage en soumettant des PR aux** [**HackTricks**](https://github.com/carlospolop/hacktricks) et [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
