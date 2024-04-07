# Analyse Forensique Android

<details>

<summary><strong>Apprenez le piratage AWS de zéro à héros avec</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (Expert de l'équipe rouge AWS de HackTricks)</strong></a><strong>!</strong></summary>

Autres façons de soutenir HackTricks :

* Si vous souhaitez voir votre **entreprise annoncée dans HackTricks** ou **télécharger HackTricks en PDF**, consultez les [**PLANS D'ABONNEMENT**](https://github.com/sponsors/carlospolop) !
* Obtenez le [**swag officiel PEASS & HackTricks**](https://peass.creator-spring.com)
* Découvrez [**La famille PEASS**](https://opensea.io/collection/the-peass-family), notre collection exclusive de [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Rejoignez le** 💬 [**groupe Discord**](https://discord.gg/hRep4RUj7f) ou le [**groupe Telegram**](https://t.me/peass) ou **suivez-nous** sur **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Partagez vos astuces de piratage en soumettant des PR aux** [**HackTricks**](https://github.com/carlospolop/hacktricks) et [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) dépôts GitHub.

</details>

## Appareil Verrouillé

Pour commencer à extraire des données d'un appareil Android, il doit être déverrouillé. S'il est verrouillé, vous pouvez :

* Vérifier si le débogage via USB est activé sur l'appareil.
* Vérifier une éventuelle [attaque par empreintes digitales](https://www.usenix.org/legacy/event/woot10/tech/full\_papers/Aviv.pdf)
* Essayer avec une [force brute](https://www.cultofmac.com/316532/this-brute-force-device-can-crack-any-iphones-pin-code/)

## Acquisition de Données

Créez une [sauvegarde Android en utilisant adb](../mobile-pentesting/android-app-pentesting/adb-commands.md#backup) et extrayez-la en utilisant [Android Backup Extractor](https://sourceforge.net/projects/adbextractor/) : `java -jar abe.jar unpack file.backup file.tar`

### Si accès root ou connexion physique à l'interface JTAG

* `cat /proc/partitions` (recherchez le chemin de la mémoire flash, généralement la première entrée est _mmcblk0_ et correspond à toute la mémoire flash).
* `df /data` (Découvrez la taille de bloc du système).
* dd if=/dev/block/mmcblk0 of=/sdcard/blk0.img bs=4096 (exécutez-le avec les informations recueillies sur la taille de bloc).

### Mémoire

Utilisez Linux Memory Extractor (LiME) pour extraire les informations de la RAM. C'est une extension de noyau qui doit être chargée via adb.

<details>

<summary><strong>Apprenez le piratage AWS de zéro à héros avec</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (Expert de l'équipe rouge AWS de HackTricks)</strong></a><strong>!</strong></summary>

Autres façons de soutenir HackTricks :

* Si vous souhaitez voir votre **entreprise annoncée dans HackTricks** ou **télécharger HackTricks en PDF**, consultez les [**PLANS D'ABONNEMENT**](https://github.com/sponsors/carlospolop) !
* Obtenez le [**swag officiel PEASS & HackTricks**](https://peass.creator-spring.com)
* Découvrez [**La famille PEASS**](https://opensea.io/collection/the-peass-family), notre collection exclusive de [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Rejoignez le** 💬 [**groupe Discord**](https://discord.gg/hRep4RUj7f) ou le [**groupe Telegram**](https://t.me/peass) ou **suivez-nous** sur **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Partagez vos astuces de piratage en soumettant des PR aux** [**HackTricks**](https://github.com/carlospolop/hacktricks) et [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) dépôts GitHub.

</details>
