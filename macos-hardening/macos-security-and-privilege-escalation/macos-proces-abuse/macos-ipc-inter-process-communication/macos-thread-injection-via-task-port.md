# Injection de thread macOS via le port de tâche

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* Travaillez-vous dans une **entreprise de cybersécurité** ? Voulez-vous voir votre **entreprise annoncée dans HackTricks** ? ou voulez-vous avoir accès à la **dernière version de PEASS ou télécharger HackTricks en PDF** ? Consultez les [**PLANS D'ABONNEMENT**](https://github.com/sponsors/carlospolop) !
* Découvrez [**The PEASS Family**](https://opensea.io/collection/the-peass-family), notre collection exclusive de [**NFT**](https://opensea.io/collection/the-peass-family)
* Obtenez le [**swag officiel PEASS & HackTricks**](https://peass.creator-spring.com)
* **Rejoignez le** [**💬**](https://emojipedia.org/speech-balloon/) [**groupe Discord**](https://discord.gg/hRep4RUj7f) ou le [**groupe Telegram**](https://t.me/peass) ou **suivez** moi sur **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Partagez vos astuces de piratage en soumettant des PR au** [**repo hacktricks**](https://github.com/carlospolop/hacktricks) **et au** [**repo hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>

Ce post a été copié depuis [https://bazad.github.io/2018/10/bypassing-platform-binary-task-threads/](https://bazad.github.io/2018/10/bypassing-platform-binary-task-threads/) (qui contient plus d'informations)

### Code

* [https://github.com/bazad/threadexec](https://github.com/bazad/threadexec)
* [https://gist.github.com/knightsc/bd6dfeccb02b77eb6409db5601dcef36](https://gist.github.com/knightsc/bd6dfeccb02b77eb6409db5601dcef36)

### 1. Détournement de thread

La première chose que nous faisons est d'appeler **`task_threads()`** sur le port de tâche pour obtenir une liste de threads dans la tâche distante, puis choisir l'un d'entre eux à détourner. Contrairement aux frameworks d'injection de code traditionnels, nous **ne pouvons pas créer un nouveau thread distant** car `thread_create_running()` sera bloqué par la nouvelle atténuation.

Ensuite, nous pouvons appeler **`thread_suspend()`** pour arrêter l'exécution du thread.

À ce stade, le seul contrôle utile que nous avons sur le thread distant est de l'**arrêter**, de le **démarrer**, d'**obtenir** ses **valeurs de registre** et de **définir** ses **valeurs de registre**. Ainsi, nous pouvons **initier un appel de fonction distant** en définissant les registres `x0` à `x7` dans le thread distant sur les **arguments**, en définissant **`pc`** sur la fonction que nous voulons exécuter, et en démarrant le thread. À ce stade, nous devons détecter le retour et nous assurer que le thread ne plante pas.

Il existe plusieurs façons de procéder. Une façon serait de **enregistrer un gestionnaire d'exception** pour le thread distant en utilisant `thread_set_exception_ports()` et de définir le registre d'adresse de retour, `lr`, sur une adresse invalide avant d'appeler la fonction ; de cette façon, après l'exécution de la fonction, une exception serait générée et un message serait envoyé à notre port d'exception, à partir duquel nous pouvons inspecter l'état du thread pour récupérer la valeur de retour. Cependant, pour simplifier, j'ai copié la stratégie utilisée dans l'exploit triple\_fetch de Ian Beer, qui consistait à **définir `lr` sur l'adresse d'une instruction qui bouclerait indéfiniment** et à vérifier régulièrement les registres du thread jusqu'à ce que **`pc` pointe vers cette instruction**.

### 2. Ports Mach pour la communication

L'étape suivante consiste à **créer des ports Mach** par lesquels nous pouvons communiquer avec le thread distant. Ces ports Mach seront utiles plus tard pour faciliter le transfert de droits d'envoi et de réception arbitraires entre les tâches.

Pour établir une communication bidirectionnelle, nous devrons créer deux droits de réception Mach : un dans la **tâche locale et un dans la tâche distante**. Ensuite, nous devrons **transférer un droit d'envoi** à chaque port **vers l'autre tâche**. Cela donnera à chaque tâche un moyen d'envoyer un message qui peut être reçu par l'autre.

Concentrons-nous d'abord sur la configuration du port local, c'est-à-dire le port pour lequel la tâche locale détient le droit de réception. Nous pouvons créer le port Mach comme n'importe quel autre, en appelant `mach_port_allocate()`. Le truc consiste à obtenir un droit d'envoi vers ce port dans la tâche distante.

Un truc pratique que nous pouvons utiliser pour copier un droit d'envoi de la tâche actuelle dans une tâche distante en n'utilisant qu'une primitive d'exécution de base est de stocker un **droit d'envoi vers notre port local dans le port spécial `THREAD_KERNEL_PORT` du thread distant** en utilisant `thread_set_special_port()` ; ensuite, nous pouvons faire en sorte que le thread distant appelle `mach_thread_self()` pour récupérer le droit d'envoi.

Ensuite, nous allons configurer le port distant, qui est pratiquement l'inverse de ce que nous venons de faire. Nous pouvons faire en sorte que le **thread distant alloue un port Mach en appelant `mach_reply_port()`** ; nous ne pouvons pas utiliser `mach_port_allocate()` car ce dernier renvoie le nom du port alloué en mémoire et nous n'avons pas encore de primitive de lecture. Une fois que nous avons un port, nous pouvons créer un droit d'envoi en appelant `mach_port_insert_right()` dans le thread distant. Ensuite, nous pouvons stocker le port dans le noyau en appelant `thread_set_special_port()`. Enfin, dans la tâche locale, nous pouvons récupérer le port en appelant `thread_get_special_port()` sur le thread distant, **ce qui nous donne un droit d'envoi vers le port Mach récemment alloué dans la tâche distante**.

À ce stade, nous avons créé les ports Mach que nous utiliserons pour la communication bidirectionnelle.
### 3. Lecture/écriture basique de la mémoire <a href="#step-3-basic-memory-readwrite" id="step-3-basic-memory-readwrite"></a>

Maintenant, nous allons utiliser la primitive d'exécution pour créer des primitives de lecture et d'écriture basiques de la mémoire. Ces primitives ne seront pas utilisées pour grand-chose (nous passerons bientôt à des primitives beaucoup plus puissantes), mais elles constituent une étape clé pour nous aider à étendre notre contrôle sur le processus distant.

Pour lire et écrire dans la mémoire en utilisant notre primitive d'exécution, nous rechercherons des fonctions comme celles-ci :
```c
uint64_t read_func(uint64_t *address) {
return *address;
}
void write_func(uint64_t *address, uint64_t value) {
*address = value;
}
```
Ils pourraient correspondre à l'assemblage suivant :
```
_read_func:
ldr     x0, [x0]
ret
_write_func:
str     x1, [x0]
ret
```
Une analyse rapide de certaines bibliothèques courantes a révélé de bons candidats. Pour lire la mémoire, nous pouvons utiliser la fonction `property_getName()` de la [bibliothèque Objective-C runtime](https://opensource.apple.com/source/objc4/objc4-723/runtime/objc-runtime-new.mm.auto.html) :
```c
const char *property_getName(objc_property_t prop)
{
return prop->name;
}
```
Il s'avère que `prop` est le premier champ de `objc_property_t`, donc cela correspond directement à la fonction hypothétique `read_func` ci-dessus. Nous devons simplement effectuer un appel de fonction à distance avec le premier argument étant l'adresse que nous voulons lire, et la valeur de retour sera les données à cette adresse.

Trouver une fonction pré-faite pour écrire en mémoire est légèrement plus difficile, mais il existe encore de bonnes options sans effets secondaires indésirables. Dans libxpc, la fonction `_xpc_int64_set_value()` a le désassemblage suivant :
```
__xpc_int64_set_value:
str     x1, [x0, #0x18]
ret
```
Ainsi, pour effectuer une écriture de 64 bits à l'adresse `adresse`, nous pouvons effectuer l'appel à distance :
```c
_xpc_int64_set_value(address - 0x18, value)
```
Avec ces primitives en main, nous sommes prêts à créer une mémoire partagée.

### 4. Mémoire partagée

Notre prochaine étape consiste à créer une mémoire partagée entre la tâche distante et locale. Cela nous permettra de transférer plus facilement des données entre les processus : avec une région de mémoire partagée, la lecture et l'écriture arbitraire de mémoire sont aussi simples qu'un appel distant à `memcpy()`. De plus, avoir une région de mémoire partagée nous permettra de configurer facilement une pile afin de pouvoir appeler des fonctions avec plus de 8 arguments.

Pour simplifier les choses, nous pouvons réutiliser les fonctionnalités de mémoire partagée de libxpc. Libxpc fournit un type d'objet XPC, `OS_xpc_shmem`, qui permet d'établir des régions de mémoire partagée via XPC. En inversant libxpc, nous déterminons que `OS_xpc_shmem` est basé sur des entrées de mémoire Mach, qui sont des ports Mach représentant une région de mémoire virtuelle. Et comme nous avons déjà montré comment envoyer des ports Mach à la tâche distante, nous pouvons l'utiliser pour configurer facilement notre propre mémoire partagée.

Premièrement, nous devons allouer la mémoire que nous partagerons en utilisant `mach_vm_allocate()`. Nous devons utiliser `mach_vm_allocate()` afin de pouvoir utiliser `xpc_shmem_create()` pour créer un objet `OS_xpc_shmem` pour la région. `xpc_shmem_create()` se chargera de créer l'entrée de mémoire Mach pour nous et stockera le droit d'envoi Mach vers l'entrée de mémoire dans l'objet `OS_xpc_shmem` opaque à l'offset `0x18`.

Une fois que nous avons le port de l'entrée de mémoire, nous créerons un objet `OS_xpc_shmem` dans le processus distant représentant la même région de mémoire, ce qui nous permettra d'appeler `xpc_shmem_map()` pour établir la mise en correspondance de la mémoire partagée. Tout d'abord, nous effectuons un appel distant à `malloc()` pour allouer de la mémoire pour l'objet `OS_xpc_shmem` et utilisons notre primitive d'écriture de base pour copier le contenu de l'objet `OS_xpc_shmem` local. Malheureusement, l'objet résultant n'est pas tout à fait correct : son champ d'entrée de mémoire Mach à l'offset `0x18` contient le nom de la tâche locale pour l'entrée de mémoire, et non le nom de la tâche distante. Pour corriger cela, nous utilisons l'astuce `thread_set_special_port()` pour insérer un droit d'envoi vers l'entrée de mémoire Mach dans la tâche distante, puis nous écrasons le champ `0x18` avec le nom de l'entrée de mémoire distante. À ce stade, l'objet `OS_xpc_shmem` distant est valide et la mise en correspondance de la mémoire peut être établie avec un appel distant à `xpc_shmem_remote()`.

### 5. Contrôle total <a href="#step-5-full-control" id="step-5-full-control"></a>

Avec une mémoire partagée à une adresse connue et une primitive d'exécution arbitraire, nous avons pratiquement terminé. Les lectures et écritures arbitraires de mémoire sont implémentées en appelant `memcpy()` vers et depuis la région partagée, respectivement. Les appels de fonctions avec plus de 8 arguments sont effectués en disposant les arguments supplémentaires au-delà des 8 premiers sur la pile selon la convention d'appel. Le transfert de ports Mach arbitraires entre les tâches peut être effectué en envoyant des messages Mach sur les ports établis précédemment. Nous pouvons même transférer des descripteurs de fichiers entre les processus en utilisant des fileports (un grand merci à Ian Beer pour avoir démontré cette technique dans triple\_fetch!).

En résumé, nous avons maintenant un contrôle total et facile sur le processus cible. Vous pouvez consulter l'implémentation complète et l'API exposée dans la bibliothèque [threadexec](https://github.com/bazad/threadexec).

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* Travaillez-vous dans une **entreprise de cybersécurité** ? Voulez-vous voir votre **entreprise annoncée dans HackTricks** ? Ou voulez-vous avoir accès à la **dernière version de PEASS ou télécharger HackTricks en PDF** ? Consultez les [**PLANS D'ABONNEMENT**](https://github.com/sponsors/carlospolop) !
* Découvrez [**The PEASS Family**](https://opensea.io/collection/the-peass-family), notre collection d'[**NFTs**](https://opensea.io/collection/the-peass-family) exclusifs.
* Obtenez le [**swag officiel PEASS & HackTricks**](https://peass.creator-spring.com).
* **Rejoignez le** [**💬**](https://emojipedia.org/speech-balloon/) [**groupe Discord**](https://discord.gg/hRep4RUj7f) ou le [**groupe Telegram**](https://t.me/peass) ou **suivez** moi sur **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Partagez vos astuces de piratage en soumettant des PR au** [**repo hacktricks**](https://github.com/carlospolop/hacktricks) **et au** [**repo hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
