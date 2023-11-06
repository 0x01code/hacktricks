# Injection de bibliothèque macOS

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* Travaillez-vous dans une **entreprise de cybersécurité** ? Voulez-vous voir votre **entreprise annoncée dans HackTricks** ? ou voulez-vous avoir accès à la **dernière version de PEASS ou télécharger HackTricks en PDF** ? Consultez les [**PLANS D'ABONNEMENT**](https://github.com/sponsors/carlospolop) !
* Découvrez [**The PEASS Family**](https://opensea.io/collection/the-peass-family), notre collection exclusive de [**NFT**](https://opensea.io/collection/the-peass-family)
* Obtenez le [**swag officiel PEASS & HackTricks**](https://peass.creator-spring.com)
* **Rejoignez le** [**💬**](https://emojipedia.org/speech-balloon/) [**groupe Discord**](https://discord.gg/hRep4RUj7f) ou le [**groupe Telegram**](https://t.me/peass) ou **suivez** moi sur **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Partagez vos astuces de piratage en soumettant des PR au** [**repo hacktricks**](https://github.com/carlospolop/hacktricks) **et au** [**repo hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>

{% hint style="danger" %}
Le code de **dyld est open source** et peut être trouvé sur [https://opensource.apple.com/source/dyld/](https://opensource.apple.com/source/dyld/) et peut être téléchargé sous forme de tar en utilisant une **URL telle que** [https://opensource.apple.com/tarballs/dyld/dyld-852.2.tar.gz](https://opensource.apple.com/tarballs/dyld/dyld-852.2.tar.gz)
{% endhint %}

## **DYLD\_INSERT\_LIBRARIES**

> Ceci est une liste de bibliothèques dynamiques séparées par des deux-points à charger avant celles spécifiées dans le programme. Cela vous permet de tester de nouveaux modules de bibliothèques partagées dynamiques existantes utilisées dans des images de l'espace de noms plat en chargeant une bibliothèque partagée dynamique temporaire contenant uniquement les nouveaux modules. Notez que cela n'a aucun effet sur les images construites avec un espace de noms à deux niveaux utilisant une bibliothèque partagée dynamique, sauf si DYLD\_FORCE\_FLAT\_NAMESPACE est également utilisé.

C'est comme le [**LD\_PRELOAD sur Linux**](../../../../linux-hardening/privilege-escalation#ld\_preload).

Cette technique peut également être **utilisée comme technique ASEP** car chaque application installée a un fichier plist appelé "Info.plist" qui permet l'**attribution de variables d'environnement** en utilisant une clé appelée `LSEnvironmental`.

{% hint style="info" %}
Depuis 2012, **Apple a considérablement réduit la puissance** de **`DYLD_INSERT_LIBRARIES`**.

Allez dans le code et **vérifiez `src/dyld.cpp`**. Dans la fonction **`pruneEnvironmentVariables`**, vous pouvez voir que les variables **`DYLD_*`** sont supprimées.

Dans la fonction **`processRestricted`**, la raison de la restriction est définie. En vérifiant ce code, vous pouvez voir que les raisons sont :

* Le binaire est `setuid/setgid`
* Existence de la section `__RESTRICT/__restrict` dans le binaire macho.
* Le logiciel a des attributs (runtime renforcé) sans l'attribut [`com.apple.security.cs.allow-dyld-environment-variables`](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_allow-dyld-environment-variables)
* Vérifiez les **attributs** d'un binaire avec : `codesign -dv --entitlements :- </path/to/bin>`

Dans les versions plus récentes, vous pouvez trouver cette logique dans la deuxième partie de la fonction **`configureProcessRestrictions`.** Cependant, ce qui est exécuté dans les versions plus récentes, ce sont les **vérifications initiales de la fonction** (vous pouvez supprimer les ifs liés à iOS ou à la simulation car ils ne seront pas utilisés dans macOS.
{% endhint %}

### Validation de la bibliothèque

Même si le binaire permet d'utiliser la variable d'environnement **`DYLD_INSERT_LIBRARIES`**, si le binaire vérifie la signature de la bibliothèque à charger, il ne chargera pas une bibliothèque personnalisée.

Pour charger une bibliothèque personnalisée, le binaire doit avoir **l'un des attributs suivants** :

* &#x20;[`com.apple.security.cs.disable-library-validation`](../../macos-security-protections/macos-dangerous-entitlements.md#com.apple.security.cs.disable-library-validation)
* [`com.apple.private.security.clear-library-validation`](../../macos-security-protections/macos-dangerous-entitlements.md#com.apple.private.security.clear-library-validation)

ou le binaire **ne doit pas** avoir le **flag runtime renforcé** ou le **flag de validation de la bibliothèque**.

Vous pouvez vérifier si un binaire a le **runtime renforcé** avec `codesign --display --verbose <bin>` en vérifiant le flag runtime dans **`CodeDirectory`** comme : **`CodeDirectory v=20500 size=767 flags=0x10000(runtime) hashes=13+7 location=embedded`**

Vous pouvez également charger une bibliothèque si elle est **signée avec le même certificat que le binaire**.

Trouvez un exemple sur la façon d'utiliser (abuser) cela et vérifiez les restrictions dans :

{% content-ref url="../../macos-dyld-hijacking-and-dyld_insert_libraries.md" %}
[macos-dyld-hijacking-and-dyld\_insert\_libraries.md](../../macos-dyld-hijacking-and-dyld\_insert\_libraries.md)
{% endcontent-ref %}
## Dylib Hijacking

{% hint style="danger" %}
N'oubliez pas que les restrictions de **Validation de bibliothèque précédentes s'appliquent également** pour effectuer des attaques de détournement de dylib.
{% endhint %}

Comme dans Windows, sur MacOS, vous pouvez également **détourner les dylibs** pour faire **exécuter du code arbitraire** par les **applications**.\
Cependant, la façon dont les applications **MacOS** chargent les bibliothèques est **plus restreinte** que sur Windows. Cela implique que les développeurs de logiciels malveillants peuvent toujours utiliser cette technique pour **se camoufler**, mais la probabilité de pouvoir **abuser de cela pour escalader les privilèges est beaucoup plus faible**.

Tout d'abord, il est **plus courant** de trouver que les **binaires MacOS indiquent le chemin complet** des bibliothèques à charger. Et deuxièmement, **MacOS ne recherche jamais** dans les dossiers du **$PATH** pour les bibliothèques.

La **partie principale** du **code** liée à cette fonctionnalité se trouve dans **`ImageLoader::recursiveLoadLibraries`** dans `ImageLoader.cpp`.

Il existe **4 commandes d'en-tête différentes** qu'un binaire macho peut utiliser pour charger des bibliothèques :

* La commande **`LC_LOAD_DYLIB`** est la commande courante pour charger un dylib.
* La commande **`LC_LOAD_WEAK_DYLIB`** fonctionne comme la précédente, mais si le dylib n'est pas trouvé, l'exécution se poursuit sans aucune erreur.
* La commande **`LC_REEXPORT_DYLIB`** permet de faire proxy (ou réexporter) les symboles d'une bibliothèque différente.
* La commande **`LC_LOAD_UPWARD_DYLIB`** est utilisée lorsque deux bibliothèques dépendent l'une de l'autre (on parle de dépendance ascendante).

Cependant, il existe **2 types de détournement de dylib** :

* **Bibliothèques liées faibles manquantes** : Cela signifie que l'application va essayer de charger une bibliothèque qui n'existe pas, configurée avec **LC\_LOAD\_WEAK\_DYLIB**. Ensuite, **si un attaquant place un dylib là où il est attendu, il sera chargé**.
* Le fait que le lien soit "faible" signifie que l'application continuera de s'exécuter même si la bibliothèque n'est pas trouvée.
* Le **code associé** à cela se trouve dans la fonction `ImageLoaderMachO::doGetDependentLibraries` de `ImageLoaderMachO.cpp`, où `lib->required` est seulement `false` lorsque `LC_LOAD_WEAK_DYLIB` est vrai.
* **Trouver des bibliothèques liées faibles** dans les binaires avec (vous avez ensuite un exemple de création de bibliothèques de détournement) :
* ```bash
otool -l </chemin/vers/binaire> | grep LC_LOAD_WEAK_DYLIB -A 5 cmd LC_LOAD_WEAK_DYLIB
cmdsize 56
name /var/tmp/lib/libUtl.1.dylib (offset 24)
time stamp 2 Wed Jun 21 12:23:31 1969
current version 1.0.0
compatibility version 1.0.0
```
* **Configuré avec @rpath** : Les binaires Mach-O peuvent avoir les commandes **`LC_RPATH`** et **`LC_LOAD_DYLIB`**. En fonction des **valeurs** de ces commandes, les **bibliothèques** vont être **chargées** à partir de **différents répertoires**.
* **`LC_RPATH`** contient les chemins de certains dossiers utilisés pour charger les bibliothèques par le binaire.
* **`LC_LOAD_DYLIB`** contient le chemin des bibliothèques spécifiques à charger. Ces chemins peuvent contenir **`@rpath`**, qui sera **remplacé** par les valeurs dans **`LC_RPATH`**. S'il y a plusieurs chemins dans **`LC_RPATH`**, chacun sera utilisé pour rechercher la bibliothèque à charger. Exemple :
* Si **`LC_LOAD_DYLIB`** contient `@rpath/library.dylib` et **`LC_RPATH`** contient `/application/app.app/Contents/Framework/v1/` et `/application/app.app/Contents/Framework/v2/`. Les deux dossiers vont être utilisés pour charger `library.dylib`**.** Si la bibliothèque n'existe pas dans `[...]/v1/` et que l'attaquant peut la placer là pour détourner le chargement de la bibliothèque dans `[...]/v2/`, car l'ordre des chemins dans **`LC_LOAD_DYLIB`** est suivi.
* **Trouver les chemins rpath et les bibliothèques** dans les binaires avec : `otool -l </chemin/vers/binaire> | grep -E "LC_RPATH|LC_LOAD_DYLIB" -A 5`

{% hint style="info" %}
**`@executable_path`** : Est le **chemin** vers le répertoire contenant le **fichier exécutable principal**.

**`@loader_path`** : Est le **chemin** vers le **répertoire** contenant le **binaire Mach-O** qui contient la commande de chargement.

* Lorsqu'il est utilisé dans un exécutable, **`@loader_path`** est effectivement le **même** que **`@executable_path`**.
* Lorsqu'il est utilisé dans un **dylib**, **`@loader_path`** donne le **chemin** vers le **dylib**.
{% endhint %}

La façon d'**escalader les privilèges** en abusant de cette fonctionnalité serait dans le cas rare où une **application** exécutée **par** **root** recherche une **bibliothèque** dans un **dossier** où l'attaquant a des **permissions d'écriture**.

{% hint style="success" %}
Un **scanner** pratique pour trouver des **bibliothèques manquantes** dans les applications est [**Dylib Hijack Scanner**](https://objective-see.com/products/dhs.html) ou une [**version CLI**](https://github.com/pandazheng/DylibHijack).\
Un **rapport avec des détails techniques** intéressants sur cette technique peut être trouvé [**ici**](https://www.virusbulletin.com/virusbulletin/2015/03/dylib-hijacking-os-x).
{% endhint %}

**Exemple**

{% content-ref url="../../macos-dyld-hijacking-and-dyld_insert_libraries.md" %}
[macos-dyld-hijacking-and-dyld\_insert\_libraries.md](../../macos-dyld-hijacking-and-dyld\_insert\_libraries.md)
{% endcontent-ref %}

## Dlopen Hijacking

{% hint style="danger" %}
N'oubliez pas que les restrictions de **Validation de bibliothèque précédentes s'appliquent également** pour effectuer des attaques de détournement de Dlopen.
{% endhint %}

D'après **`man dlopen`** :

* Lorsque le chemin **ne contient pas de caractère slash** (c'est-à-dire qu'il s'agit simplement d'un nom de feuille), **dlopen() effectuera une recherche**. Si **`$DYLD_LIBRARY_PATH`** était défini au lancement, dyld cherchera d'abord dans ce répertoire. Ensuite, si le fichier mach-o appelant ou l'exécutable principal spécifie un **`LC_RPATH`**, alors dyld cherchera dans ces répertoires. Ensuite, si le processus est **non restreint**, dyld recherchera dans le **répertoire de travail actuel**. Enfin, pour les anciens binaires, dyld essaiera quelques solutions de repli. Si **`$DYLD_FALLBACK_LIBRARY_PATH`** était défini au lancement, dyld cherchera dans **ces répertoires**, sinon, dyld cherchera dans **`/usr/local/lib/`** (si le processus est non restreint), puis dans **`/usr/lib/`** (ces informations ont été extraites de **`man dlopen`**).
1. `$DYLD_LIBRARY_PATH`
2. `LC_RPATH`
3. `CWD` (si non restreint)
4. `$DYLD_FALLBACK_LIBRARY_PATH`
5. `/usr/local/lib/` (si non restreint)
6. `/usr/lib/`

{% hint style="danger" %}
S'il n'y a pas de slash dans le nom, il y aurait 2 façons de faire un détournement :

* Si un **`LC_RPATH`** est **modifiable** (mais la signature est vérifiée, donc pour cela, vous avez également besoin que le binaire soit non restreint)
* Si le binaire est **non restreint**, il est alors possible de charger quelque chose depuis le CWD (ou en abusant de l'une des variables d'environnement mentionnées)
{% endhint %}
* Lorsque le chemin ressemble à un chemin de framework (par exemple, `/stuff/foo.framework/foo`), si `$DYLD_FRAMEWORK_PATH` a été défini au lancement, dyld cherchera d'abord dans ce répertoire le chemin partiel du framework (par exemple, `foo.framework/foo`). Ensuite, dyld essaiera le chemin fourni tel quel (en utilisant le répertoire de travail actuel pour les chemins relatifs). Enfin, pour les anciens binaires, dyld essaiera quelques solutions de repli. Si `$DYLD_FALLBACK_FRAMEWORK_PATH` a été défini au lancement, dyld recherchera dans ces répertoires. Sinon, il recherchera dans `/Library/Frameworks` (sur macOS si le processus n'est pas restreint), puis dans `/System/Library/Frameworks`.
1. `$DYLD_FRAMEWORK_PATH`
2. chemin fourni (en utilisant le répertoire de travail actuel pour les chemins relatifs si le processus n'est pas restreint)
3. `$DYLD_FALLBACK_FRAMEWORK_PATH`
4. `/Library/Frameworks` (si le processus n'est pas restreint)
5. `/System/Library/Frameworks`

{% hint style="danger" %}
Si le chemin est un chemin de framework, la façon de le pirater serait :

* Si le processus n'est pas restreint, en abusant du chemin relatif à partir du CWD et des variables d'environnement mentionnées (même si les documents ne précisent pas si le processus est restreint, les variables d'environnement DYLD\_\* sont supprimées)
{% endhint %}

* Lorsque le chemin contient un slash mais n'est pas un chemin de framework (c'est-à-dire un chemin complet ou un chemin partiel vers une dylib), dlopen() cherche d'abord (si défini) dans `$DYLD_LIBRARY_PATH` (avec la partie terminale du chemin). Ensuite, dyld essaie le chemin fourni (en utilisant le répertoire de travail actuel pour les chemins relatifs, mais uniquement pour les processus non restreints). Enfin, pour les anciens binaires, dyld essaiera des solutions de repli. Si `$DYLD_FALLBACK_LIBRARY_PATH` a été défini au lancement, dyld recherchera dans ces répertoires, sinon, dyld cherchera dans `/usr/local/lib/` (si le processus n'est pas restreint), puis dans `/usr/lib/`.
1. `$DYLD_LIBRARY_PATH`
2. chemin fourni (en utilisant le répertoire de travail actuel pour les chemins relatifs si le processus n'est pas restreint)
3. `$DYLD_FALLBACK_LIBRARY_PATH`
4. `/usr/local/lib/` (si le processus n'est pas restreint)
5. `/usr/lib/`

{% hint style="danger" %}
Si le nom contient des slashes et n'est pas un framework, la façon de le pirater serait :

* Si le binaire n'est pas restreint, il est possible de charger quelque chose à partir du CWD ou de `/usr/local/lib` (ou d'abuser de l'une des variables d'environnement mentionnées)
{% endhint %}

{% hint style="info" %}
Remarque : Il n'y a **aucun** fichier de configuration pour **contrôler la recherche de dlopen**.

Remarque : Si l'exécutable principal est un binaire **set\[ug]id ou signé avec des entitlements**, alors **toutes les variables d'environnement sont ignorées**, et seul un chemin complet peut être utilisé (consultez les restrictions de DYLD\_INSERT\_LIBRARIES pour plus d'informations détaillées)

Remarque : Les plates-formes Apple utilisent des fichiers "universels" pour combiner les bibliothèques 32 bits et 64 bits. Cela signifie qu'il n'y a **pas de chemins de recherche séparés pour les bibliothèques 32 bits et 64 bits**.

Remarque : Sur les plates-formes Apple, la plupart des dylibs du système d'exploitation sont **combinées dans le cache dyld** et n'existent pas sur le disque. Par conséquent, l'appel à **`stat()`** pour vérifier si une dylib du système d'exploitation existe **ne fonctionnera pas**. Cependant, **`dlopen_preflight()`** utilise les mêmes étapes que **`dlopen()`** pour trouver un fichier mach-o compatible.
{% endhint %}

**Vérifier les chemins**

Vérifions toutes les options avec le code suivant :
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
Si vous le compilez et l'exécutez, vous pouvez voir **où chaque bibliothèque a été recherchée sans succès**. De plus, vous pouvez **filtrer les journaux du système de fichiers** :
```bash
sudo fs_usage | grep "dlopentest"
```
## Hijacking de chemin relatif

Si un **binaire/application privilégié** (comme un SUID ou un binaire avec des autorisations puissantes) charge une bibliothèque de chemin relatif (par exemple en utilisant `@executable_path` ou `@loader_path`) et que la **validation de la bibliothèque est désactivée**, il pourrait être possible de déplacer le binaire vers un emplacement où l'attaquant pourrait **modifier la bibliothèque chargée par le chemin relatif** et l'utiliser pour injecter du code dans le processus.

## Supprimer les variables d'environnement `DYLD_*` et `LD_LIBRARY_PATH`

Dans le fichier `dyld-dyld-832.7.1/src/dyld2.cpp`, il est possible de trouver la fonction **`pruneEnvironmentVariables`**, qui supprimera toute variable d'environnement qui **commence par `DYLD_`** et **`LD_LIBRARY_PATH=`**.

Elle définira également spécifiquement les variables d'environnement **`DYLD_FALLBACK_FRAMEWORK_PATH`** et **`DYLD_FALLBACK_LIBRARY_PATH`** sur **null** pour les binaires **suid** et **sgid**.

Cette fonction est appelée depuis la fonction **`_main`** du même fichier lorsqu'on cible OSX de la manière suivante:
```cpp
#if TARGET_OS_OSX
if ( !gLinkContext.allowEnvVarsPrint && !gLinkContext.allowEnvVarsPath && !gLinkContext.allowEnvVarsSharedCache ) {
pruneEnvironmentVariables(envp, &apple);
```
et ces indicateurs booléens sont définis dans le même fichier dans le code :
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
Ce qui signifie essentiellement que si le binaire est **suid** ou **sgid**, ou s'il a un segment **RESTRICT** dans les en-têtes ou s'il a été signé avec le drapeau **CS\_RESTRICT**, alors **`!gLinkContext.allowEnvVarsPrint && !gLinkContext.allowEnvVarsPath && !gLinkContext.allowEnvVarsSharedCache`** est vrai et les variables d'environnement sont élaguées.

Notez que si CS\_REQUIRE\_LV est vrai, alors les variables ne seront pas élaguées mais la validation de la bibliothèque vérifiera qu'elles utilisent le même certificat que le binaire d'origine.

## Vérification des restrictions

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
### Section `__RESTRICT` avec le segment `__restrict`

La section `__RESTRICT` est une section spéciale dans le segment `__restrict` du binaire macOS. Cette section est utilisée pour restreindre l'accès à certaines fonctionnalités sensibles du système d'exploitation. Elle est conçue pour empêcher les processus non autorisés d'interférer avec ces fonctionnalités.

Lorsqu'un processus tente d'accéder à une fonctionnalité restreinte, le système d'exploitation vérifie si le processus a les privilèges nécessaires pour y accéder. Si ce n'est pas le cas, le processus est bloqué et une erreur est renvoyée.

La section `__RESTRICT` est utilisée pour renforcer la sécurité du système d'exploitation en limitant les privilèges des processus et en empêchant les attaques de privilège d'escalade. Elle joue un rôle essentiel dans la protection des fonctionnalités sensibles du système d'exploitation contre les abus et les exploitations malveillantes.

Il est important de noter que la section `__RESTRICT` ne peut être modifiée que par des processus ayant les privilèges nécessaires. Cela garantit que seuls les processus autorisés peuvent accéder aux fonctionnalités restreintes du système d'exploitation.

En résumé, la section `__RESTRICT` avec le segment `__restrict` est une mesure de sécurité essentielle dans macOS pour restreindre l'accès aux fonctionnalités sensibles du système d'exploitation et prévenir les attaques de privilège d'escalade.
```bash
gcc -sectcreate __RESTRICT __restrict /dev/null hello.c -o hello-restrict
DYLD_INSERT_LIBRARIES=inject.dylib ./hello-restrict
```
### Runtime sécurisé

Créez un nouveau certificat dans le trousseau d'accès et utilisez-le pour signer le binaire :

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
Notez que même s'il existe des binaires signés avec le drapeau **`0x0(none)`**, ils peuvent obtenir dynamiquement le drapeau **`CS_RESTRICT`** lorsqu'ils sont exécutés et donc cette technique ne fonctionnera pas sur eux.

Vous pouvez vérifier si un processus a ce drapeau avec (obtenez [**csops ici**](https://github.com/axelexic/CSOps)):&#x20;
```bash
csops -status <pid>
```
et vérifiez ensuite si le drapeau 0x800 est activé.
{% endhint %}

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* Travaillez-vous dans une **entreprise de cybersécurité** ? Voulez-vous voir votre **entreprise annoncée dans HackTricks** ? ou voulez-vous avoir accès à la **dernière version de PEASS ou télécharger HackTricks en PDF** ? Consultez les [**PLANS D'ABONNEMENT**](https://github.com/sponsors/carlospolop) !
* Découvrez [**La famille PEASS**](https://opensea.io/collection/the-peass-family), notre collection exclusive de [**NFT**](https://opensea.io/collection/the-peass-family)
* Obtenez le [**swag officiel PEASS & HackTricks**](https://peass.creator-spring.com)
* **Rejoignez le** [**💬**](https://emojipedia.org/speech-balloon/) [**groupe Discord**](https://discord.gg/hRep4RUj7f) ou le [**groupe Telegram**](https://t.me/peass) ou **suivez** moi sur **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Partagez vos astuces de piratage en soumettant des PR au** [**repo hacktricks**](https://github.com/carlospolop/hacktricks) **et au** [**repo hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
