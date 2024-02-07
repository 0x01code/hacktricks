# Extraction de la mémoire macOS

<details>

<summary><strong>Apprenez le piratage AWS de zéro à héros avec</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (Expert en équipe rouge AWS de HackTricks)</strong></a><strong>!</strong></summary>

Autres façons de soutenir HackTricks :

* Si vous souhaitez voir votre **entreprise annoncée dans HackTricks** ou **télécharger HackTricks en PDF**, consultez les [**PLANS D'ABONNEMENT**](https://github.com/sponsors/carlospolop) !
* Obtenez le [**swag officiel PEASS & HackTricks**](https://peass.creator-spring.com)
* Découvrez [**La famille PEASS**](https://opensea.io/collection/the-peass-family), notre collection exclusive de [**NFT**](https://opensea.io/collection/the-peass-family)
* **Rejoignez le** 💬 [**groupe Discord**](https://discord.gg/hRep4RUj7f) ou le [**groupe Telegram**](https://t.me/peass) ou **suivez** moi sur **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **Partagez vos astuces de piratage en soumettant des PR aux** [**HackTricks**](https://github.com/carlospolop/hacktricks) et [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) dépôts GitHub.

</details>

## Artefacts de mémoire

### Fichiers d'échange

Les fichiers d'échange, tels que `/private/var/vm/swapfile0`, servent de **caches lorsque la mémoire physique est pleine**. Lorsqu'il n'y a plus d'espace en mémoire physique, ses données sont transférées vers un fichier d'échange, puis ramenées en mémoire physique au besoin. Plusieurs fichiers d'échange peuvent être présents, avec des noms tels que swapfile0, swapfile1, et ainsi de suite.

### Image d'hibernation

Le fichier situé à `/private/var/vm/sleepimage` est crucial pendant le **mode d'hibernation**. **Les données de la mémoire sont stockées dans ce fichier lorsque macOS hiberne**. Lorsque l'ordinateur se réveille, le système récupère les données de la mémoire de ce fichier, permettant à l'utilisateur de reprendre là où il s'était arrêté.

Il convient de noter que sur les systèmes MacOS modernes, ce fichier est généralement chiffré pour des raisons de sécurité, rendant la récupération difficile.

* Pour vérifier si le chiffrement est activé pour le sleepimage, la commande `sysctl vm.swapusage` peut être exécutée. Cela montrera si le fichier est chiffré.

### Journaux de pression mémoire

Un autre fichier important lié à la mémoire dans les systèmes MacOS est le **journal de pression mémoire**. Ces journaux se trouvent dans `/var/log` et contiennent des informations détaillées sur l'utilisation de la mémoire du système et les événements de pression. Ils peuvent être particulièrement utiles pour diagnostiquer des problèmes liés à la mémoire ou comprendre comment le système gère la mémoire au fil du temps.

## Extraction de la mémoire avec osxpmem

Pour extraire la mémoire d'une machine macOS, vous pouvez utiliser [**osxpmem**](https://github.com/google/rekall/releases/download/v1.5.1/osxpmem-2.1.post4.zip).

**Remarque** : Les instructions suivantes ne fonctionneront que pour les Mac avec architecture Intel. Cet outil est désormais archivé et la dernière version date de 2017. Le binaire téléchargé en suivant les instructions ci-dessous cible les puces Intel car Apple Silicon n'existait pas en 2017. Il est peut-être possible de compiler le binaire pour l'architecture arm64, mais vous devrez essayer par vous-même.
```bash
#Dump raw format
sudo osxpmem.app/osxpmem --format raw -o /tmp/dump_mem

#Dump aff4 format
sudo osxpmem.app/osxpmem -o /tmp/dump_mem.aff4
```
Si vous rencontrez cette erreur : `osxpmem.app/MacPmem.kext failed to load - (libkern/kext) authentication failure (file ownership/permissions); check the system/kernel logs for errors or try kextutil(8)` Vous pouvez la corriger en effectuant :
```bash
sudo cp -r osxpmem.app/MacPmem.kext "/tmp/"
sudo kextutil "/tmp/MacPmem.kext"
#Allow the kext in "Security & Privacy --> General"
sudo osxpmem.app/osxpmem --format raw -o /tmp/dump_mem
```
**D'autres erreurs** peuvent être corrigées en **autorisant le chargement du kext** dans "Sécurité et confidentialité --> Général", il suffit de **l'autoriser**.

Vous pouvez également utiliser ce **oneliner** pour télécharger l'application, charger le kext et effectuer le dumping de la mémoire :

{% code overflow="wrap" %}
```bash
sudo su
cd /tmp; wget https://github.com/google/rekall/releases/download/v1.5.1/osxpmem-2.1.post4.zip; unzip osxpmem-2.1.post4.zip; chown -R root:wheel osxpmem.app/MacPmem.kext; kextload osxpmem.app/MacPmem.kext; osxpmem.app/osxpmem --format raw -o /tmp/dump_mem
```
{% endcode %}

<details>

<summary><strong>Apprenez le piratage AWS de zéro à héros avec</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (Expert Red Team AWS de HackTricks)</strong></a><strong>!</strong></summary>

Autres façons de soutenir HackTricks :

* Si vous souhaitez voir votre **entreprise annoncée dans HackTricks** ou **télécharger HackTricks en PDF**, consultez les [**PLANS D'ABONNEMENT**](https://github.com/sponsors/carlospolop) !
* Obtenez le [**swag officiel PEASS & HackTricks**](https://peass.creator-spring.com)
* Découvrez [**La famille PEASS**](https://opensea.io/collection/the-peass-family), notre collection exclusive de [**NFT**](https://opensea.io/collection/the-peass-family)
* **Rejoignez le** 💬 [**groupe Discord**](https://discord.gg/hRep4RUj7f) ou le [**groupe Telegram**](https://t.me/peass) ou **suivez** moi sur **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **Partagez vos astuces de piratage en soumettant des PR aux** [**HackTricks**](https://github.com/carlospolop/hacktricks) et [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) dépôts GitHub.

</details>
