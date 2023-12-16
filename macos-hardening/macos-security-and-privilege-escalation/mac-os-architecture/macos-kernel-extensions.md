# Extensions du noyau macOS

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* Vous travaillez dans une **entreprise de cybersécurité** ? Vous voulez voir votre **entreprise annoncée sur HackTricks** ? Ou vous voulez accéder à la **dernière version de PEASS ou télécharger HackTricks en PDF** ? Consultez les [**PLANS D'ABONNEMENT**](https://github.com/sponsors/carlospolop) !
* Découvrez [**The PEASS Family**](https://opensea.io/collection/the-peass-family), notre collection exclusive de [**NFTs**](https://opensea.io/collection/the-peass-family)
* Obtenez le [**swag officiel de PEASS et HackTricks**](https://peass.creator-spring.com)
* **Rejoignez le** [**💬**](https://emojipedia.org/speech-balloon/) **groupe Discord** ou le [**groupe Telegram**](https://t.me/peass) ou **suivez-moi** sur **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live).
* **Partagez vos astuces de piratage en envoyant une PR à** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **et** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud).

</details>

## Informations de base

Les extensions du noyau (Kexts) sont des **packages** avec une extension **`.kext`** qui sont **chargés directement dans l'espace du noyau macOS**, fournissant des fonctionnalités supplémentaires au système d'exploitation principal.

### Exigences

Évidemment, c'est si puissant qu'il est **compliqué de charger une extension du noyau**. Voici les **exigences** qu'une extension du noyau doit respecter pour être chargée :

* Lorsque vous **entrez en mode de récupération**, les **extensions du noyau doivent être autorisées** à être chargées :

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* L'extension du noyau doit être **signée avec un certificat de signature de code du noyau**, qui ne peut être **accordé que par Apple**. Qui examinera en détail l'entreprise et les raisons pour lesquelles elle est nécessaire.
* L'extension du noyau doit également être **notarisée**, Apple pourra la vérifier pour les logiciels malveillants.
* Ensuite, l'utilisateur **root** est celui qui peut **charger l'extension du noyau** et les fichiers à l'intérieur du package doivent **appartenir à root**.
* Pendant le processus de chargement, le package doit être préparé dans un **emplacement protégé non root** : `/Library/StagedExtensions` (nécessite l'autorisation `com.apple.rootless.storage.KernelExtensionManagement`).
* Enfin, lors de la tentative de chargement, l'utilisateur recevra une [**demande de confirmation**](https://developer.apple.com/library/archive/technotes/tn2459/\_index.html) et, si elle est acceptée, l'ordinateur doit être **redémarré** pour la charger.

### Processus de chargement

Dans Catalina, c'était comme ça : Il est intéressant de noter que le processus de **vérification** se produit dans **l'espace utilisateur**. Cependant, seules les applications ayant l'autorisation **`com.apple.private.security.kext-management`** peuvent **demander au noyau de charger une extension** : `kextcache`, `kextload`, `kextutil`, `kextd`, `syspolicyd`

1. **`kextutil`** en ligne de commande **démarre** le processus de **vérification** pour charger une extension
* Il communiquera avec **`kextd`** en utilisant un **service Mach**.
2. **`kextd`** vérifiera plusieurs choses, telles que la **signature**
* Il communiquera avec **`syspolicyd`** pour **vérifier** si l'extension peut être **chargée**.
3. **`syspolicyd`** demandera **l'autorisation à l'utilisateur** si l'extension n'a pas été chargée précédemment.
* **`syspolicyd`** rapportera le résultat à **`kextd`**
4. **`kextd`** pourra enfin **demander au noyau de charger** l'extension

Si **`kextd`** n'est pas disponible, **`kextutil`** peut effectuer les mêmes vérifications.

## Références

* [https://www.makeuseof.com/how-to-enable-third-party-kernel-extensions-apple-silicon-mac/](https://www.makeuseof.com/how-to-enable-third-party-kernel-extensions-apple-silicon-mac/)
* [https://www.youtube.com/watch?v=hGKOskSiaQo](https://www.youtube.com/watch?v=hGKOskSiaQo)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* Vous travaillez dans une **entreprise de cybersécurité** ? Vous voulez voir votre **entreprise annoncée sur HackTricks** ? Ou vous voulez accéder à la **dernière version de PEASS ou télécharger HackTricks en PDF** ? Consultez les [**PLANS D'ABONNEMENT**](https://github.com/sponsors/carlospolop) !
* Découvrez [**The PEASS Family**](https://opensea.io/collection/the-peass-family), notre collection exclusive de [**NFTs**](https://opensea.io/collection/the-peass-family)
* Obtenez le [**swag officiel de PEASS et HackTricks**](https://peass.creator-spring.com)
* **Rejoignez le** [**💬**](https://emojipedia.org/speech-balloon/) **groupe Discord** ou le [**groupe Telegram**](https://t.me/peass) ou **suivez-moi** sur **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live).
* **Partagez vos astuces de piratage en envoyant une PR à** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **et** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud).

</details>
