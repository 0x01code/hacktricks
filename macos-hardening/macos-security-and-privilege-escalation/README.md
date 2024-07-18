# Sécurité et élévation de privilèges sur macOS

{% hint style="success" %}
Apprenez et pratiquez le piratage AWS :<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**Formation HackTricks AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Apprenez et pratiquez le piratage GCP : <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**Formation HackTricks GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Soutenez HackTricks</summary>

* Consultez les [**plans d'abonnement**](https://github.com/sponsors/carlospolop)!
* **Rejoignez le** 💬 [**groupe Discord**](https://discord.gg/hRep4RUj7f) ou le [**groupe Telegram**](https://t.me/peass) ou **suivez-nous** sur **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Partagez des astuces de piratage en soumettant des PR aux** [**HackTricks**](https://github.com/carlospolop/hacktricks) et [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) dépôts GitHub.

</details>
{% endhint %}

<figure><img src="../../.gitbook/assets/image (380).png" alt=""><figcaption></figcaption></figure>

Rejoignez le serveur [**HackenProof Discord**](https://discord.com/invite/N3FrSbmwdy) pour communiquer avec des pirates expérimentés et des chasseurs de primes en bugs !

**Perspectives de piratage**\
Engagez-vous avec du contenu qui explore le frisson et les défis du piratage

**Actualités de piratage en temps réel**\
Restez informé du monde du piratage en temps réel grâce aux actualités et aux informations

**Dernières annonces**\
Restez informé des dernières primes de bugs lancées et des mises à jour cruciales de la plateforme

**Rejoignez-nous sur** [**Discord**](https://discord.com/invite/N3FrSbmwdy) et commencez à collaborer avec les meilleurs pirates dès aujourd'hui !

## Base de MacOS

Si vous n'êtes pas familier avec macOS, vous devriez commencer par apprendre les bases de macOS :

* Fichiers et autorisations spéciaux de macOS :

{% content-ref url="macos-files-folders-and-binaries/" %}
[macos-files-folders-and-binaries](macos-files-folders-and-binaries/)
{% endcontent-ref %}

* Utilisateurs courants de macOS

{% content-ref url="macos-users.md" %}
[macos-users.md](macos-users.md)
{% endcontent-ref %}

* **AppleFS**

{% content-ref url="macos-applefs.md" %}
[macos-applefs.md](macos-applefs.md)
{% endcontent-ref %}

* L'**architecture** du **noyau**

{% content-ref url="mac-os-architecture/" %}
[mac-os-architecture](mac-os-architecture/)
{% endcontent-ref %}

* Services et protocoles réseau **courants** de macOS

{% content-ref url="macos-protocols.md" %}
[macos-protocols.md](macos-protocols.md)
{% endcontent-ref %}

* macOS **Open Source** : [https://opensource.apple.com/](https://opensource.apple.com/)
* Pour télécharger un `tar.gz`, modifiez une URL telle que [https://opensource.apple.com/**source**/dyld/](https://opensource.apple.com/source/dyld/) en [https://opensource.apple.com/**tarballs**/dyld/**dyld-852.2.tar.gz**](https://opensource.apple.com/tarballs/dyld/dyld-852.2.tar.gz)

### MDM de MacOS

Dans les entreprises, les systèmes macOS sont très probablement gérés avec un MDM. Par conséquent, du point de vue d'un attaquant, il est intéressant de savoir **comment cela fonctionne** :

{% content-ref url="../macos-red-teaming/macos-mdm/" %}
[macos-mdm](../macos-red-teaming/macos-mdm/)
{% endcontent-ref %}

### MacOS - Inspection, Débogage et Fuzzing

{% content-ref url="macos-apps-inspecting-debugging-and-fuzzing/" %}
[macos-apps-inspecting-debugging-and-fuzzing](macos-apps-inspecting-debugging-and-fuzzing/)
{% endcontent-ref %}

## Protections de sécurité de MacOS

{% content-ref url="macos-security-protections/" %}
[macos-security-protections](macos-security-protections/)
{% endcontent-ref %}

## Surface d'attaque

### Autorisations de fichiers

Si un **processus s'exécutant en tant que root écrit** un fichier pouvant être contrôlé par un utilisateur, l'utilisateur pourrait en abuser pour **élever ses privilèges**.\
Cela pourrait se produire dans les situations suivantes :

* Le fichier utilisé a déjà été créé par un utilisateur (appartenant à l'utilisateur)
* Le fichier utilisé est accessible en écriture par l'utilisateur en raison d'un groupe
* Le fichier utilisé est à l'intérieur d'un répertoire appartenant à l'utilisateur (l'utilisateur pourrait créer le fichier)
* Le fichier utilisé est à l'intérieur d'un répertoire appartenant à root mais l'utilisateur a un accès en écriture dessus en raison d'un groupe (l'utilisateur pourrait créer le fichier)

Pouvoir **créer un fichier** qui va être **utilisé par root**, permet à un utilisateur de **profiter de son contenu** ou même de créer des **liens symboliques/hardlinks** pour le rediriger vers un autre endroit.

Pour ce type de vulnérabilités, n'oubliez pas de **vérifier les installateurs `.pkg` vulnérables** :

{% content-ref url="macos-files-folders-and-binaries/macos-installers-abuse.md" %}
[macos-installers-abuse.md](macos-files-folders-and-binaries/macos-installers-abuse.md)
{% endcontent-ref %}

### Extension de fichier et gestionnaires d'applications de schéma d'URL

Des applications étranges enregistrées par des extensions de fichier pourraient être abusées et différentes applications peuvent être enregistrées pour ouvrir des protocoles spécifiques

{% content-ref url="macos-file-extension-apps.md" %}
[macos-file-extension-apps.md](macos-file-extension-apps.md)
{% endcontent-ref %}

## Élévation de privilèges TCC / SIP de macOS

Dans macOS, les **applications et binaires peuvent avoir des autorisations** pour accéder à des dossiers ou des paramètres qui les rendent plus privilégiés que d'autres.

Par conséquent, un attaquant qui souhaite compromettre avec succès une machine macOS devra **élever ses privilèges TCC** (ou même **contourner SIP**, selon ses besoins).

Ces privilèges sont généralement accordés sous forme de **droits** avec lesquels l'application est signée, ou l'application peut demander certains accès et après que l'utilisateur les a **approuvés**, ils peuvent être trouvés dans les **bases de données TCC**. Une autre façon pour un processus d'obtenir ces privilèges est d'être un **enfant d'un processus** avec ces **privilèges** car ils sont généralement **hérités**.

Suivez ces liens pour trouver différentes façons d'**élever les privilèges dans TCC**](macos-security-protections/macos-tcc/#tcc-privesc-and-bypasses), de [**contourner TCC**](macos-security-protections/macos-tcc/macos-tcc-bypasses/) et comment dans le passé [**SIP a été contourné**](macos-security-protections/macos-sip.md#sip-bypasses).

## Élévation de privilèges traditionnelle de macOS

Bien sûr, du point de vue des équipes rouges, vous devriez également être intéressé par l'élévation des privilèges à root. Consultez le post suivant pour quelques indices :

{% content-ref url="macos-privilege-escalation.md" %}
[macos-privilege-escalation.md](macos-privilege-escalation.md)
{% endcontent-ref %}
## Références

* [**OS X Incident Response: Scripting and Analysis**](https://www.amazon.com/OS-Incident-Response-Scripting-Analysis-ebook/dp/B01FHOHHVS)
* [**https://taomm.org/vol1/analysis.html**](https://taomm.org/vol1/analysis.html)
* [**https://github.com/NicolasGrimonpont/Cheatsheet**](https://github.com/NicolasGrimonpont/Cheatsheet)
* [**https://assets.sentinelone.com/c/sentinal-one-mac-os-?x=FvGtLJ**](https://assets.sentinelone.com/c/sentinal-one-mac-os-?x=FvGtLJ)
* [**https://www.youtube.com/watch?v=vMGiplQtjTY**](https://www.youtube.com/watch?v=vMGiplQtjTY)

<figure><img src="../../.gitbook/assets/image (380).png" alt=""><figcaption></figcaption></figure>

Rejoignez le serveur [**HackenProof Discord**](https://discord.com/invite/N3FrSbmwdy) pour communiquer avec des hackers expérimentés et des chasseurs de bugs !

**Perspectives sur le Hacking**\
Engagez-vous avec du contenu qui explore le frisson et les défis du hacking

**Actualités de Hacking en Temps Réel**\
Restez à jour avec le monde du hacking en constante évolution grâce aux actualités et aux informations en temps réel

**Dernières Annonces**\
Restez informé des dernières primes de bugs lancées et des mises à jour cruciales de la plateforme

**Rejoignez-nous sur** [**Discord**](https://discord.com/invite/N3FrSbmwdy) et commencez à collaborer avec les meilleurs hackers dès aujourd'hui!

{% hint style="success" %}
Apprenez et pratiquez le Hacking AWS :<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**Formation HackTricks AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Apprenez et pratiquez le Hacking GCP : <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**Formation HackTricks GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Soutenez HackTricks</summary>

* Consultez les [**plans d'abonnement**](https://github.com/sponsors/carlospolop)!
* **Rejoignez le** 💬 [**groupe Discord**](https://discord.gg/hRep4RUj7f) ou le [**groupe Telegram**](https://t.me/peass) ou **suivez-nous** sur **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Partagez des astuces de hacking en soumettant des PR aux** [**HackTricks**](https://github.com/carlospolop/hacktricks) et [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}
