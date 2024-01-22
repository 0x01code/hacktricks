# Abus de processus macOS

<details>

<summary><strong>Apprenez le hacking AWS de zéro à héros avec</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Autres moyens de soutenir HackTricks :

* Si vous souhaitez voir votre **entreprise annoncée dans HackTricks** ou **télécharger HackTricks en PDF**, consultez les [**PLANS D'ABONNEMENT**](https://github.com/sponsors/carlospolop)!
* Obtenez le [**merchandising officiel PEASS & HackTricks**](https://peass.creator-spring.com)
* Découvrez [**La Famille PEASS**](https://opensea.io/collection/the-peass-family), notre collection d'[**NFTs**](https://opensea.io/collection/the-peass-family) exclusifs
* **Rejoignez le** 💬 [**groupe Discord**](https://discord.gg/hRep4RUj7f) ou le [**groupe Telegram**](https://t.me/peass) ou **suivez** moi sur **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **Partagez vos astuces de hacking en soumettant des PR aux dépôts github** [**HackTricks**](https://github.com/carlospolop/hacktricks) et [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>

## Abus de processus MacOS

MacOS, comme tout autre système d'exploitation, offre une variété de méthodes et de mécanismes pour que les **processus interagissent, communiquent et partagent des données**. Bien que ces techniques soient essentielles pour le fonctionnement efficace du système, elles peuvent également être détournées par des acteurs malveillants pour **effectuer des activités malveillantes**.

### Injection de bibliothèque

L'injection de bibliothèque est une technique par laquelle un attaquant **force un processus à charger une bibliothèque malveillante**. Une fois injectée, la bibliothèque s'exécute dans le contexte du processus cible, donnant à l'attaquant les mêmes permissions et accès que le processus.

{% content-ref url="macos-library-injection/" %}
[injection-de-bibliotheque-macos](macos-library-injection/)
{% endcontent-ref %}

### Accrochage de fonction

L'accrochage de fonction implique **l'interception d'appels de fonctions** ou de messages au sein d'un code logiciel. En accrochant des fonctions, un attaquant peut **modifier le comportement** d'un processus, observer des données sensibles ou même prendre le contrôle du flux d'exécution.

{% content-ref url="../mac-os-architecture/macos-function-hooking.md" %}
[accrochage-de-fonction-macos.md](../mac-os-architecture/macos-function-hooking.md)
{% endcontent-ref %}

### Communication inter-processus

La communication inter-processus (IPC) fait référence à différentes méthodes par lesquelles des processus séparés **partagent et échangent des données**. Bien que l'IPC soit fondamental pour de nombreuses applications légitimes, elle peut également être détournée pour subvertir l'isolation des processus, fuiter des informations sensibles ou effectuer des actions non autorisées.

{% content-ref url="../mac-os-architecture/macos-ipc-inter-process-communication/" %}
[communication-inter-processus-macos](../mac-os-architecture/macos-ipc-inter-process-communication/)
{% endcontent-ref %}

### Injection dans les applications Electron

Les applications Electron exécutées avec des variables d'environnement spécifiques pourraient être vulnérables à l'injection de processus :

{% content-ref url="macos-electron-applications-injection.md" %}
[injection-dans-applications-electron-macos.md](macos-electron-applications-injection.md)
{% endcontent-ref %}

### NIB sale

Les fichiers NIB **définissent les éléments de l'interface utilisateur (UI)** et leurs interactions au sein d'une application. Cependant, ils peuvent **exécuter des commandes arbitraires** et **Gatekeeper n'empêche pas** une application déjà exécutée d'être exécutée si un **fichier NIB est modifié**. Par conséquent, ils pourraient être utilisés pour faire exécuter des commandes arbitraires par des programmes arbitraires :

{% content-ref url="macos-dirty-nib.md" %}
[nib-sale-macos.md](macos-dirty-nib.md)
{% endcontent-ref %}

### Injection dans les applications Java

Il est possible d'abuser de certaines capacités Java (comme la variable d'environnement **`_JAVA_OPTS`**) pour faire exécuter **du code/commandes arbitraires** par une application Java.

{% content-ref url="macos-java-apps-injection.md" %}
[injection-dans-applications-java-macos.md](macos-java-apps-injection.md)
{% endcontent-ref %}

### Injection dans les applications .Net

Il est possible d'injecter du code dans des applications .Net en **abusant de la fonctionnalité de débogage .Net** (non protégée par les protections macOS telles que le durcissement à l'exécution).

{% content-ref url="macos-.net-applications-injection.md" %}
[injection-dans-applications-net-macos.md](macos-.net-applications-injection.md)
{% endcontent-ref %}

### Injection Perl

Vérifiez différentes options pour faire exécuter du code arbitraire par un script Perl dans :

{% content-ref url="macos-perl-applications-injection.md" %}
[injection-dans-applications-perl-macos.md](macos-perl-applications-injection.md)
{% endcontent-ref %}

### Injection Ruby

Il est également possible d'abuser des variables d'environnement Ruby pour faire exécuter du code arbitraire par des scripts arbitraires :

{% content-ref url="macos-ruby-applications-injection.md" %}
[injection-dans-applications-ruby-macos.md](macos-ruby-applications-injection.md)
{% endcontent-ref %}

### Injection Python

Si la variable d'environnement **`PYTHONINSPECT`** est définie, le processus Python passera dans une interface de ligne de commande Python une fois terminé. Il est également possible d'utiliser **`PYTHONSTARTUP`** pour indiquer un script Python à exécuter au début d'une session interactive.\
Cependant, notez que le script **`PYTHONSTARTUP`** ne sera pas exécuté lorsque **`PYTHONINSPECT`** crée la session interactive.

D'autres variables d'environnement telles que **`PYTHONPATH`** et **`PYTHONHOME`** pourraient également être utiles pour faire exécuter du code arbitraire par une commande Python.

Notez que les exécutables compilés avec **`pyinstaller`** n'utiliseront pas ces variables d'environnement même s'ils fonctionnent avec un Python intégré.

{% hint style="danger" %}
Dans l'ensemble, je n'ai pas trouvé de moyen de faire exécuter du code arbitraire par Python en abusant des variables d'environnement.\
Cependant, la plupart des gens installent Python via **Homebrew**, qui installera Python dans un **emplacement accessible en écriture** pour l'utilisateur admin par défaut. Vous pouvez le détourner avec quelque chose comme :
```bash
mv /opt/homebrew/bin/python3 /opt/homebrew/bin/python3.old
cat > /opt/homebrew/bin/python3 <<EOF
#!/bin/bash
# Extra hijack code
/opt/homebrew/bin/python3.old "$@"
EOF
chmod +x /opt/homebrew/bin/python3
```
Même **root** exécutera ce code lors de l'exécution de python.
{% endhint %}

## Détection

### Shield

[**Shield**](https://theevilbit.github.io/shield/) ([**Github**](https://github.com/theevilbit/Shield)) est une application open source qui peut **détecter et bloquer les actions d'injection de processus** :

* Utilisation de **Variables d'Environnement** : Il surveillera la présence de l'une des variables d'environnement suivantes : **`DYLD_INSERT_LIBRARIES`**, **`CFNETWORK_LIBRARY_PATH`**, **`RAWCAMERA_BUNDLE_PATH`** et **`ELECTRON_RUN_AS_NODE`**
* Utilisation des appels **`task_for_pid`** : Pour trouver quand un processus veut obtenir le **port de tâche d'un autre** ce qui permet d'injecter du code dans le processus.
* **Paramètres des applications Electron** : Quelqu'un peut utiliser les arguments de ligne de commande **`--inspect`**, **`--inspect-brk`** et **`--remote-debugging-port`** pour démarrer une application Electron en mode débogage, et ainsi injecter du code.
* Utilisation de **symlinks** ou **hardlinks** : Typiquement, l'abus le plus courant est de **placer un lien avec nos privilèges utilisateur**, et de le **pointer vers un emplacement de privilège supérieur**. La détection est très simple pour les hardlinks et les symlinks. Si le processus créant le lien a un **niveau de privilège différent** de celui du fichier cible, nous créons une **alerte**. Malheureusement, dans le cas des symlinks, le blocage n'est pas possible, car nous n'avons pas d'informations sur la destination du lien avant sa création. C'est une limitation du framework EndpointSecuriy d'Apple.

### Appels effectués par d'autres processus

Dans [**ce billet de blog**](https://knight.sc/reverse%20engineering/2019/04/15/detecting-task-modifications.html), vous pouvez trouver comment il est possible d'utiliser la fonction **`task_name_for_pid`** pour obtenir des informations sur d'autres **processus injectant du code dans un processus** et ensuite obtenir des informations sur cet autre processus.

Notez que pour appeler cette fonction, vous devez être **le même uid** que celui qui exécute le processus ou **root** (et cela retourne des informations sur le processus, pas un moyen d'injecter du code).

## Références

* [https://theevilbit.github.io/shield/](https://theevilbit.github.io/shield/)
* [https://medium.com/@metnew/why-electron-apps-cant-store-your-secrets-confidentially-inspect-option-a49950d6d51f](https://medium.com/@metnew/why-electron-apps-cant-store-your-secrets-confidentially-inspect-option-a49950d6d51f)

<details>

<summary><strong>Apprenez le hacking AWS de zéro à héros avec</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Autres moyens de soutenir HackTricks :

* Si vous souhaitez voir votre **entreprise annoncée dans HackTricks** ou **télécharger HackTricks en PDF**, consultez les [**PLANS D'ABONNEMENT**](https://github.com/sponsors/carlospolop)!
* Obtenez le [**merchandising officiel PEASS & HackTricks**](https://peass.creator-spring.com)
* Découvrez [**La Famille PEASS**](https://opensea.io/collection/the-peass-family), notre collection d'[**NFTs**](https://opensea.io/collection/the-peass-family) exclusifs
* **Rejoignez le** 💬 [**groupe Discord**](https://discord.gg/hRep4RUj7f) ou le [**groupe telegram**](https://t.me/peass) ou **suivez** moi sur **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **Partagez vos astuces de hacking en soumettant des PR aux dépôts github** [**HackTricks**](https://github.com/carlospolop/hacktricks) et [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
