# Noyau macOS & Extensions Système

<details>

<summary><strong>Apprenez le piratage AWS de zéro à héros avec</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Autres moyens de soutenir HackTricks :

* Si vous souhaitez voir votre **entreprise annoncée dans HackTricks** ou **télécharger HackTricks en PDF**, consultez les [**PLANS D'ABONNEMENT**](https://github.com/sponsors/carlospolop)!
* Obtenez le [**merchandising officiel PEASS & HackTricks**](https://peass.creator-spring.com)
* Découvrez [**La Famille PEASS**](https://opensea.io/collection/the-peass-family), notre collection d'[**NFTs exclusifs**](https://opensea.io/collection/the-peass-family)
* **Rejoignez le** 💬 [**groupe Discord**](https://discord.gg/hRep4RUj7f) ou le [**groupe Telegram**](https://t.me/peass) ou **suivez** moi sur **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **Partagez vos astuces de piratage en soumettant des PR aux dépôts github** [**HackTricks**](https://github.com/carlospolop/hacktricks) et [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>

## Noyau XNU

Le **cœur de macOS est XNU**, qui signifie "X is Not Unix". Ce noyau est principalement composé du **micro-noyau Mach** (qui sera discuté plus tard), **et** des éléments du Berkeley Software Distribution (**BSD**). XNU fournit également une plateforme pour **les pilotes de noyau via un système appelé I/O Kit**. Le noyau XNU fait partie du projet open source Darwin, ce qui signifie que **son code source est librement accessible**.

Du point de vue d'un chercheur en sécurité ou d'un développeur Unix, **macOS** peut sembler assez **similaire** à un système **FreeBSD** avec une interface graphique élégante et une multitude d'applications personnalisées. La plupart des applications développées pour BSD se compileront et fonctionneront sur macOS sans nécessiter de modifications, car les outils en ligne de commande familiers aux utilisateurs Unix sont tous présents dans macOS. Cependant, parce que le noyau XNU intègre Mach, il existe des différences significatives entre un système de type Unix traditionnel et macOS, et ces différences peuvent causer des problèmes potentiels ou offrir des avantages uniques.

Version open source de XNU : [https://opensource.apple.com/source/xnu/](https://opensource.apple.com/source/xnu/)

### Mach

Mach est un **micro-noyau** conçu pour être **compatible UNIX**. L'un de ses principes de conception clés était de **minimiser** la quantité de **code** s'exécutant dans l'espace **noyau** et de permettre à de nombreuses fonctions typiques du noyau, telles que le système de fichiers, le réseau et l'E/S, de **s'exécuter en tant que tâches au niveau utilisateur**.

Dans XNU, Mach est **responsable de nombreuses opérations de bas niveau critiques** qu'un noyau gère généralement, telles que la planification des processeurs, le multitâche et la gestion de la mémoire virtuelle.

### BSD

Le **noyau XNU** **intègre** également une quantité importante de code dérivé du projet **FreeBSD**. Ce code **s'exécute dans le noyau avec Mach**, dans le même espace d'adressage. Cependant, le code FreeBSD au sein de XNU peut différer substantiellement du code FreeBSD original car des modifications étaient nécessaires pour assurer sa compatibilité avec Mach. FreeBSD contribue à de nombreuses opérations du noyau, y compris :

* Gestion des processus
* Gestion des signaux
* Mécanismes de sécurité de base, y compris la gestion des utilisateurs et des groupes
* Infrastructure des appels système
* Pile TCP/IP et sockets
* Pare-feu et filtrage de paquets

Comprendre l'interaction entre BSD et Mach peut être complexe, en raison de leurs cadres conceptuels différents. Par exemple, BSD utilise des processus comme unité d'exécution fondamentale, tandis que Mach fonctionne sur la base de threads. Cette divergence est réconciliée dans XNU en **associant chaque processus BSD à une tâche Mach** qui contient exactement un thread Mach. Lorsque l'appel système fork() de BSD est utilisé, le code BSD dans le noyau utilise des fonctions Mach pour créer une tâche et une structure de thread.

De plus, **Mach et BSD maintiennent chacun des modèles de sécurité différents** : le modèle de sécurité de **Mach** est basé sur les **droits de port**, tandis que le modèle de sécurité de BSD fonctionne sur la base de la **propriété des processus**. Les disparités entre ces deux modèles ont parfois entraîné des vulnérabilités d'élévation de privilèges locales. Outre les appels système typiques, il existe également des **pièges Mach qui permettent aux programmes en espace utilisateur d'interagir avec le noyau**. Ces différents éléments forment ensemble l'architecture hybride et multifacette du noyau macOS.

### I/O Kit - Pilotes

I/O Kit est le framework de **pilotes de périphériques**, open-source et orienté objet, dans le noyau XNU et est responsable de l'ajout et de la gestion des **pilotes de périphériques chargés dynamiquement**. Ces pilotes permettent d'ajouter du code modulaire au noyau de manière dynamique pour une utilisation avec différents matériels, par exemple.

{% content-ref url="macos-iokit.md" %}
[macos-iokit.md](macos-iokit.md)
{% endcontent-ref %}

### IPC - Communication Inter-Processus

{% content-ref url="macos-ipc-inter-process-communication/" %}
[macos-ipc-inter-process-communication](macos-ipc-inter-process-communication/)
{% endcontent-ref %}

### Kernelcache

Le **kernelcache** est une **version précompilée et préliée du noyau XNU**, avec des **pilotes** essentiels et des **extensions de noyau**. Il est stocké dans un format **compressé** et est décompressé en mémoire pendant le processus de démarrage. Le kernelcache facilite un **temps de démarrage plus rapide** en ayant une version prête à l'exécution du noyau et des pilotes cruciaux disponibles, réduisant le temps et les ressources qui seraient autrement dépensés pour charger et lier dynamiquement ces composants au moment du démarrage.

Dans iOS, il est situé dans **`/System/Library/Caches/com.apple.kernelcaches/kernelcache`** dans macOS, vous pouvez le trouver avec **`find / -name kernelcache 2>/dev/null`**

#### IMG4

Le format de fichier IMG4 est un format conteneur utilisé par Apple dans ses appareils iOS et macOS pour **stocker et vérifier de manière sécurisée les composants du firmware** (comme le **kernelcache**). Le format IMG4 comprend un en-tête et plusieurs balises qui encapsulent différentes pièces de données, y compris la charge utile réelle (comme un noyau ou un chargeur de démarrage), une signature et un ensemble de propriétés de manifeste. Le format prend en charge la vérification cryptographique, permettant à l'appareil de confirmer l'authenticité et l'intégrité du composant du firmware avant de l'exécuter.

Il est généralement composé des composants suivants :

* **Charge utile (IM4P)** :
* Souvent compressée (LZFSE4, LZSS, …)
* Optionnellement chiffrée
* **Manifeste (IM4M)** :
* Contient la Signature
* Dictionnaire supplémentaire Clé/Valeur
* **Informations de restauration (IM4R)** :
* Également connu sous le nom de APNonce
* Empêche la répétition de certaines mises à jour
* OPTIONNEL : Généralement, cela n'est pas trouvé

Décompresser le Kernelcache :
```bash
# pyimg4 (https://github.com/m1stadev/PyIMG4)
pyimg4 im4p extract -i kernelcache.release.iphone14 -o kernelcache.release.iphone14.e

# img4tool (https://github.com/tihmstar/img4tool
img4tool -e kernelcache.release.iphone14 -o kernelcache.release.iphone14.e
```
#### Symboles du Kernelcache

Parfois, Apple publie un **kernelcache** avec des **symboles**. Vous pouvez télécharger certains firmwares avec symboles en suivant les liens sur [https://theapplewiki.com](https://theapplewiki.com/).

### IPSW

Ce sont des **firmwares** Apple que vous pouvez télécharger depuis [**https://ipsw.me/**](https://ipsw.me/). Parmi d'autres fichiers, il contiendra le **kernelcache**.\
Pour **extraire** les fichiers, vous pouvez simplement les **décompresser**.

Après avoir extrait le firmware, vous obtiendrez un fichier tel que : **`kernelcache.release.iphone14`**. Il est au format **IMG4**, vous pouvez extraire les informations intéressantes avec :

* [**pyimg4**](https://github.com/m1stadev/PyIMG4)

{% code overflow="wrap" %}
```bash
pyimg4 im4p extract -i kernelcache.release.iphone14 -o kernelcache.release.iphone14.e
```
{% endcode %}

* [**img4tool**](https://github.com/tihmstar/img4tool)
```bash
img4tool -e kernelcache.release.iphone14 -o kernelcache.release.iphone14.e
```
Vous pouvez vérifier les symboles dans le kernelcache extrait avec : **`nm -a kernelcache.release.iphone14.e | wc -l`**

Avec cela, nous pouvons maintenant **extraire toutes les extensions** ou **celle qui vous intéresse :**
```bash
# List all extensions
kextex -l kernelcache.release.iphone14.e
## Extract com.apple.security.sandbox
kextex -e com.apple.security.sandbox kernelcache.release.iphone14.e

# Extract all
kextex_all kernelcache.release.iphone14.e

# Check the extension for symbols
nm -a binaries/com.apple.security.sandbox | wc -l
```
## Extensions du noyau macOS

macOS est **extrêmement restrictif pour charger les Extensions du Noyau** (.kext) en raison des privilèges élevés avec lesquels le code s'exécutera. En fait, par défaut, c'est pratiquement impossible (à moins qu'une méthode de contournement ne soit trouvée).

{% content-ref url="macos-kernel-extensions.md" %}
[macos-kernel-extensions.md](macos-kernel-extensions.md)
{% endcontent-ref %}

### Extensions Système macOS

Au lieu d'utiliser les Extensions du Noyau, macOS a créé les Extensions Système, qui offrent des API au niveau utilisateur pour interagir avec le noyau. De cette façon, les développeurs peuvent éviter d'utiliser des extensions du noyau.

{% content-ref url="macos-system-extensions.md" %}
[macos-system-extensions.md](macos-system-extensions.md)
{% endcontent-ref %}

## Références

* [**The Mac Hacker's Handbook**](https://www.amazon.com/-/es/Charlie-Miller-ebook-dp-B004U7MUMU/dp/B004U7MUMU/ref=mt\_other?\_encoding=UTF8\&me=\&qid=)
* [**https://taomm.org/vol1/analysis.html**](https://taomm.org/vol1/analysis.html)

<details>

<summary><strong>Apprenez le piratage AWS de zéro à héros avec</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Autres moyens de soutenir HackTricks :

* Si vous souhaitez voir votre **entreprise annoncée dans HackTricks** ou **télécharger HackTricks en PDF**, consultez les [**PLANS D'ABONNEMENT**](https://github.com/sponsors/carlospolop)!
* Obtenez le [**merchandising officiel PEASS & HackTricks**](https://peass.creator-spring.com)
* Découvrez [**La Famille PEASS**](https://opensea.io/collection/the-peass-family), notre collection d'[**NFTs**](https://opensea.io/collection/the-peass-family) exclusifs
* **Rejoignez le** 💬 [**groupe Discord**](https://discord.gg/hRep4RUj7f) ou le [**groupe telegram**](https://t.me/peass) ou **suivez** moi sur **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **Partagez vos astuces de piratage en soumettant des PR aux dépôts github** [**HackTricks**](https://github.com/carlospolop/hacktricks) et [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
