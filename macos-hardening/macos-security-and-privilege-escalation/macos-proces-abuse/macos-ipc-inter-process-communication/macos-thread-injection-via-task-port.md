# Injection de thread macOS via le port de tâche

<details>

<summary><strong>Apprenez le hacking AWS de zéro à héros avec</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Autres moyens de soutenir HackTricks :

* Si vous souhaitez voir votre **entreprise annoncée dans HackTricks** ou **télécharger HackTricks en PDF**, consultez les [**PLANS D'ABONNEMENT**](https://github.com/sponsors/carlospolop)!
* Obtenez le [**merchandising officiel PEASS & HackTricks**](https://peass.creator-spring.com)
* Découvrez [**La Famille PEASS**](https://opensea.io/collection/the-peass-family), notre collection d'[**NFTs**](https://opensea.io/collection/the-peass-family) exclusifs
* **Rejoignez le** 💬 [**groupe Discord**](https://discord.gg/hRep4RUj7f) ou le [**groupe telegram**](https://t.me/peass) ou **suivez** moi sur **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **Partagez vos astuces de hacking en soumettant des PR aux dépôts github** [**HackTricks**](https://github.com/carlospolop/hacktricks) et [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>

## Code

* [https://github.com/bazad/threadexec](https://github.com/bazad/threadexec)
* [https://gist.github.com/knightsc/bd6dfeccb02b77eb6409db5601dcef36](https://gist.github.com/knightsc/bd6dfeccb02b77eb6409db5601dcef36)


## 1. Détournement de thread

Initialement, la fonction **`task_threads()`** est invoquée sur le port de tâche pour obtenir une liste de threads de la tâche distante. Un thread est sélectionné pour le détournement. Cette approche diffère des méthodes conventionnelles d'injection de code car la création d'un nouveau thread distant est interdite en raison de la nouvelle atténuation bloquant `thread_create_running()`.

Pour contrôler le thread, **`thread_suspend()`** est appelé, arrêtant son exécution.

Les seules opérations autorisées sur le thread distant impliquent de **l'arrêter** et de **le démarrer**, de **récupérer** et de **modifier** ses valeurs de registre. Les appels de fonction à distance sont initiés en configurant les registres `x0` à `x7` pour les **arguments**, en configurant **`pc`** pour cibler la fonction souhaitée, et en activant le thread. Assurer que le thread ne plante pas après le retour nécessite la détection du retour.

Une stratégie consiste à **enregistrer un gestionnaire d'exceptions** pour le thread distant en utilisant `thread_set_exception_ports()`, en réglant le registre `lr` sur une adresse invalide avant l'appel de fonction. Cela déclenche une exception après l'exécution de la fonction, envoyant un message au port d'exception, permettant l'inspection de l'état du thread pour récupérer la valeur de retour. Alternativement, comme adopté de l'exploit triple\_fetch d'Ian Beer, `lr` est configuré pour boucler indéfiniment. Les registres du thread sont ensuite surveillés en continu jusqu'à ce que **`pc` pointe vers cette instruction**.

## 2. Ports Mach pour la communication

La phase suivante implique l'établissement de ports Mach pour faciliter la communication avec le thread distant. Ces ports sont essentiels pour transférer des droits d'envoi et de réception arbitraires entre les tâches.

Pour une communication bidirectionnelle, deux droits de réception Mach sont créés : un dans la tâche locale et l'autre dans la tâche distante. Ensuite, un droit d'envoi pour chaque port est transféré à la tâche correspondante, permettant l'échange de messages.

En se concentrant sur le port local, le droit de réception est détenu par la tâche locale. Le port est créé avec `mach_port_allocate()`. Le défi réside dans le transfert d'un droit d'envoi à ce port dans la tâche distante.

Une stratégie consiste à utiliser `thread_set_special_port()` pour placer un droit d'envoi au port local dans le `THREAD_KERNEL_PORT` du thread distant. Ensuite, le thread distant est instruit d'appeler `mach_thread_self()` pour récupérer le droit d'envoi.

Pour le port distant, le processus est essentiellement inversé. Le thread distant est dirigé pour générer un port Mach via `mach_reply_port()` (car `mach_port_allocate()` est inadapté en raison de son mécanisme de retour). Après la création du port, `mach_port_insert_right()` est invoqué dans le thread distant pour établir un droit d'envoi. Ce droit est ensuite stocké dans le noyau en utilisant `thread_set_special_port()`. De retour dans la tâche locale, `thread_get_special_port()` est utilisé sur le thread distant pour acquérir un droit d'envoi au port Mach nouvellement alloué dans la tâche distante.

L'achèvement de ces étapes aboutit à l'établissement de ports Mach, jetant les bases pour une communication bidirectionnelle.

## 3. Primitives de lecture/écriture de mémoire de base

Dans cette section, l'accent est mis sur l'utilisation de la primitive d'exécution pour établir des primitives de base de lecture et d'écriture de mémoire. Ces premières étapes sont cruciales pour obtenir plus de contrôle sur le processus distant, bien que les primitives à ce stade ne servent pas à grand-chose. Bientôt, elles seront améliorées pour des versions plus avancées.

### Lecture et écriture de mémoire à l'aide de la primitive d'exécution

L'objectif est d'effectuer la lecture et l'écriture de mémoire en utilisant des fonctions spécifiques. Pour lire la mémoire, des fonctions ressemblant à la structure suivante sont utilisées :
```c
uint64_t read_func(uint64_t *address) {
return *address;
}
```
Et pour l'écriture en mémoire, des fonctions similaires à cette structure sont utilisées :
```c
void write_func(uint64_t *address, uint64_t value) {
*address = value;
}
```
Ces fonctions correspondent aux instructions d'assemblage données :
```
_read_func:
ldr x0, [x0]
ret
_write_func:
str x1, [x0]
ret
```
### Identification de fonctions appropriées

Un scan des bibliothèques communes a révélé des candidats appropriés pour ces opérations :

1. **Lecture de mémoire :**
La fonction `property_getName()` de la [bibliothèque runtime Objective-C](https://opensource.apple.com/source/objc4/objc4-723/runtime/objc-runtime-new.mm.auto.html) est identifiée comme une fonction appropriée pour la lecture de mémoire. La fonction est décrite ci-dessous :

```c
const char *property_getName(objc_property_t prop) {
return prop->name;
}
```

Cette fonction agit efficacement comme la `read_func` en retournant le premier champ de `objc_property_t`.

2. **Écriture de mémoire :**
Trouver une fonction préconstruite pour l'écriture de mémoire est plus difficile. Cependant, la fonction `_xpc_int64_set_value()` de libxpc est un candidat approprié avec le désassemblage suivant :
```
__xpc_int64_set_value:
str x1, [x0, #0x18]
ret
```
Pour effectuer une écriture 64 bits à une adresse spécifique, l'appel distant est structuré comme suit :
```c
_xpc_int64_set_value(address - 0x18, value)
```
Avec ces primitives établies, la scène est prête pour la création de mémoire partagée, marquant une progression significative dans le contrôle du processus distant.

## 4. Configuration de la Mémoire Partagée

L'objectif est d'établir une mémoire partagée entre les tâches locales et distantes, simplifiant le transfert de données et facilitant l'appel de fonctions avec plusieurs arguments. L'approche consiste à utiliser `libxpc` et son type d'objet `OS_xpc_shmem`, qui est construit sur les entrées de mémoire Mach.

### Vue d'ensemble du Processus :

1. **Allocation de Mémoire** :
- Allouer la mémoire pour le partage en utilisant `mach_vm_allocate()`.
- Utiliser `xpc_shmem_create()` pour créer un objet `OS_xpc_shmem` pour la région de mémoire allouée. Cette fonction gérera la création de l'entrée de mémoire Mach et stockera le droit d'envoi Mach à l'offset `0x18` de l'objet `OS_xpc_shmem`.

2. **Création de Mémoire Partagée dans le Processus Distant** :
- Allouer de la mémoire pour l'objet `OS_xpc_shmem` dans le processus distant avec un appel distant à `malloc()`.
- Copier le contenu de l'objet `OS_xpc_shmem` local dans le processus distant. Cependant, cette copie initiale aura des noms d'entrée de mémoire Mach incorrects à l'offset `0x18`.

3. **Correction de l'Entrée de Mémoire Mach** :
- Utiliser la méthode `thread_set_special_port()` pour insérer un droit d'envoi pour l'entrée de mémoire Mach dans la tâche distante.
- Corriger le champ d'entrée de mémoire Mach à l'offset `0x18` en le réécrivant avec le nom de l'entrée de mémoire distante.

4. **Finalisation de la Configuration de la Mémoire Partagée** :
- Valider l'objet `OS_xpc_shmem` distant.
- Établir la cartographie de la mémoire partagée avec un appel distant à `xpc_shmem_remote()`.

En suivant ces étapes, la mémoire partagée entre les tâches locales et distantes sera efficacement mise en place, permettant des transferts de données simples et l'exécution de fonctions nécessitant plusieurs arguments.

## Extraits de Code Supplémentaires

Pour l'allocation de mémoire et la création d'objet de mémoire partagée :
```c
mach_vm_allocate();
xpc_shmem_create();
```
Pour créer et corriger l'objet de mémoire partagée dans le processus distant :
```c
malloc(); // for allocating memory remotely
thread_set_special_port(); // for inserting send right
```
Gérez correctement les détails des ports Mach et les noms d'entrée de mémoire pour garantir que la configuration de la mémoire partagée fonctionne correctement.

## 5. Atteindre un contrôle total

Une fois que nous avons établi avec succès une mémoire partagée et obtenu des capacités d'exécution arbitraires, nous avons essentiellement obtenu un contrôle total sur le processus cible. Les fonctionnalités clés permettant ce contrôle sont :

1. **Opérations de mémoire arbitraires** :
- Effectuer des lectures de mémoire arbitraires en invoquant `memcpy()` pour copier des données de la région partagée.
- Exécuter des écritures de mémoire arbitraires en utilisant `memcpy()` pour transférer des données vers la région partagée.

2. **Gestion des appels de fonctions avec plusieurs arguments** :
- Pour les fonctions nécessitant plus de 8 arguments, organiser les arguments supplémentaires sur la pile conformément à la convention d'appel.

3. **Transfert de port Mach** :
- Transférer des ports Mach entre des tâches via des messages Mach via des ports précédemment établis.

4. **Transfert de descripteur de fichier** :
- Transférer des descripteurs de fichiers entre des processus en utilisant des fileports, une technique mise en évidence par Ian Beer dans `triple_fetch`.

Ce contrôle complet est encapsulé dans la bibliothèque [threadexec](https://github.com/bazad/threadexec), fournissant une mise en œuvre détaillée et une API conviviale pour interagir avec le processus victime.

## Considérations importantes :

- Assurez-vous d'utiliser correctement `memcpy()` pour les opérations de lecture/écriture de mémoire afin de maintenir la stabilité du système et l'intégrité des données.
- Lors du transfert de ports Mach ou de descripteurs de fichiers, suivez les protocoles appropriés et gérez les ressources de manière responsable pour éviter les fuites ou l'accès non intentionnel.

En adhérant à ces directives et en utilisant la bibliothèque `threadexec`, on peut gérer et interagir efficacement avec les processus à un niveau granulaire, en obtenant un contrôle total sur le processus cible.

# Références
* https://bazad.github.io/2018/10/bypassing-platform-binary-task-threads/

<details>

<summary><strong>Apprenez le hacking AWS de zéro à héros avec</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Autres moyens de soutenir HackTricks :

* Si vous souhaitez voir votre **entreprise annoncée dans HackTricks** ou **télécharger HackTricks en PDF**, consultez les [**PLANS D'ABONNEMENT**](https://github.com/sponsors/carlospolop)!
* Obtenez le [**merchandising officiel PEASS & HackTricks**](https://peass.creator-spring.com)
* Découvrez [**La famille PEASS**](https://opensea.io/collection/the-peass-family), notre collection d'[**NFTs**](https://opensea.io/collection/the-peass-family) exclusifs
* **Rejoignez le** 💬 [**groupe Discord**](https://discord.gg/hRep4RUj7f) ou le [**groupe Telegram**](https://t.me/peass) ou **suivez** moi sur **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **Partagez vos astuces de hacking en soumettant des PR aux dépôts github** [**HackTricks**](https://github.com/carlospolop/hacktricks) et [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
