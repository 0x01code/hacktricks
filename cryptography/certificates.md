# Certificats

<details>

<summary><strong>Apprenez le piratage AWS de zéro à héros avec</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (Expert en équipe rouge AWS de HackTricks)</strong></a><strong>!</strong></summary>

Autres façons de soutenir HackTricks :

* Si vous souhaitez voir votre **entreprise annoncée dans HackTricks** ou **télécharger HackTricks en PDF**, consultez les [**PLANS D'ABONNEMENT**](https://github.com/sponsors/carlospolop) !
* Obtenez le [**swag officiel PEASS & HackTricks**](https://peass.creator-spring.com)
* Découvrez [**La famille PEASS**](https://opensea.io/collection/the-peass-family), notre collection exclusive de [**NFT**](https://opensea.io/collection/the-peass-family)
* **Rejoignez le** 💬 [**groupe Discord**](https://discord.gg/hRep4RUj7f) ou le [**groupe Telegram**](https://t.me/peass) ou **suivez-nous** sur **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Partagez vos astuces de piratage en soumettant des PR aux** [**HackTricks**](https://github.com/carlospolop/hacktricks) et [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) dépôts GitHub.

</details>

<figure><img src="../.gitbook/assets/image (3) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Utilisez [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) pour créer et **automatiser des workflows** facilement grâce aux outils communautaires les plus avancés au monde.\
Accédez dès aujourd'hui :

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## Qu'est-ce qu'un certificat

En cryptographie, un **certificat de clé publique**, également connu sous le nom de **certificat numérique** ou **certificat d'identité**, est un document électronique utilisé pour prouver la propriété d'une clé publique. Le certificat contient des informations sur la clé, des informations sur l'identité de son propriétaire (appelé le sujet), et la signature numérique d'une entité qui a vérifié le contenu du certificat (appelé l'émetteur). Si la signature est valide et que le logiciel examinant le certificat fait confiance à l'émetteur, il peut utiliser cette clé pour communiquer de manière sécurisée avec le sujet du certificat.

Dans un schéma typique d'**infrastructure à clé publique** ([PKI](https://en.wikipedia.org/wiki/Public-key\_infrastructure)), l'émetteur du certificat est une **autorité de certification** ([CA](https://en.wikipedia.org/wiki/Certificate\_authority)), généralement une entreprise qui facture ses clients pour émettre des certificats pour eux. En revanche, dans un schéma de **toile de confiance** ([web of trust](https://en.wikipedia.org/wiki/Web\_of\_trust)), les individus signent directement les clés des autres, dans un format qui remplit une fonction similaire à un certificat de clé publique.

Le format le plus courant pour les certificats de clé publique est défini par [X.509](https://en.wikipedia.org/wiki/X.509). Comme X.509 est très général, le format est en outre contraint par des profils définis pour certains cas d'utilisation, tels que l'**Infrastructure à clé publique (X.509)** comme défini dans la RFC 5280.

## Champs communs x509

* **Numéro de version** : Version du format x509.
* **Numéro de série** : Utilisé pour identifier de manière unique le certificat dans les systèmes d'une CA. En particulier, cela est utilisé pour suivre les informations de révocation.
* **Sujet** : L'entité à laquelle appartient un certificat : une machine, un individu ou une organisation.
* **Nom commun** : Domaines affectés par le certificat. Peut être 1 ou plus et peut contenir des caractères génériques.
* **Pays (C)** : Pays
* **Nom distinctif (DN)** : L'ensemble du sujet : `C=US, ST=California, L=San Francisco, O=Example, Inc., CN=shared.global.example.net`
* **Localité (L)** : Lieu local
* **Organisation (O)** : Nom de l'organisation
* **Unité organisationnelle (OU)** : Division d'une organisation (comme "Ressources humaines").
* **État ou province (ST, S ou P)** : Liste des noms d'état ou de province
* **Émetteur** : L'entité qui a vérifié les informations et signé le certificat.
* **Nom commun (CN)** : Nom de l'autorité de certification
* **Pays (C)** : Pays de l'autorité de certification
* **Nom distinctif (DN)** : Nom distinctif de l'autorité de certification
* **Localité (L)** : Lieu local où l'organisation peut être trouvée.
* **Organisation (O)** : Nom de l'organisation
* **Unité organisationnelle (OU)** : Division d'une organisation (comme "Ressources humaines").
* **Non avant** : La date et l'heure les plus précoces à partir desquelles le certificat est valide. Généralement défini quelques heures ou jours avant le moment où le certificat a été émis, pour éviter les problèmes de [dérive d'horloge](https://fr.wikipedia.org/wiki/Dérive\_d'horloge).
* **Non après** : La date et l'heure après lesquelles le certificat n'est plus valide.
* **Clé publique** : Une clé publique appartenant au sujet du certificat. (C'est l'une des parties principales car c'est ce qui est signé par la CA)
* **Algorithme de clé publique** : Algorithme utilisé pour générer la clé publique. Comme RSA.
* **Courbe de clé publique** : La courbe utilisée par l'algorithme de clé publique à courbe elliptique (si applicable). Comme nistp521.
* **Exposant de clé publique** : Exposant utilisé pour dériver la clé publique (si applicable). Comme 65537.
* **Taille de clé publique** : La taille de l'espace de clé publique en bits. Comme 2048.
* **Algorithme de signature** : L'algorithme utilisé pour signer le certificat de clé publique.
* **Signature** : Une signature du corps du certificat par la clé privée de l'émetteur.
* **Extensions x509v3**
* **Utilisation de la clé** : Les utilisations cryptographiques valides de la clé publique du certificat. Les valeurs courantes incluent la validation de la signature numérique, le chiffrement de clé et la signature de certificat.
* Dans un certificat Web, cela apparaîtra comme une _extension X509v3_ et aura la valeur `Signature numérique`
* **Utilisation étendue de la clé** : Les applications dans lesquelles le certificat peut être utilisé. Les valeurs courantes incluent l'authentification du serveur TLS, la protection des e-mails et la signature de code.
* Dans un certificat Web, cela apparaîtra comme une _extension X509v3_ et aura la valeur `Authentification du serveur Web TLS`
* **Nom alternatif du sujet** : Permet aux utilisateurs de spécifier des **noms d'hôtes** supplémentaires pour un seul **certificat SSL**. L'utilisation de l'extension SAN est une pratique standard pour les certificats SSL, et elle est en passe de remplacer l'utilisation du **nom** commun.
* **Contrainte de base** : Cette extension décrit si le certificat est un certificat de CA ou un certificat d'entité finale. Un certificat de CA est quelque chose qui signe des certificats d'autres entités et un certificat d'entité finale est le certificat utilisé dans une page Web par exemple (la dernière partie de la chaîne).
* **Identifiant de clé du sujet** (SKI) : Cette extension déclare un **identifiant unique pour la clé publique** dans le certificat. Il est requis sur tous les certificats de CA. Les CA propagent leur propre SKI vers l'extension **Identifiant de clé de l'émetteur** (AKI) sur les certificats émis. C'est le hachage de la clé publique du sujet.
* **Identifiant de clé de l'autorité** : Il contient un identifiant de clé qui est dérivé de la clé publique dans le certificat de l'émetteur. C'est le hachage de la clé publique de l'émetteur.
* **Accès aux informations de l'autorité** (AIA) : Cette extension contient au plus deux types d'informations :
* Informations sur **comment obtenir l'émetteur de ce certificat** (méthode d'accès à l'émetteur de CA)
* Adresse du **répondeur OCSP à partir duquel la révocation de ce certificat** peut être vérifiée (méthode d'accès OCSP).
* **Points de distribution de la liste de révocation** (CRL) : Cette extension identifie l'emplacement de la liste de révocation à partir de laquelle la révocation de ce certificat peut être vérifiée. L'application qui traite le certificat peut obtenir l'emplacement de la CRL à partir de cette extension, télécharger la CRL, puis vérifier la révocation de ce certificat.
* **CT SCTs de précertificat** : Journaux de transparence des certificats concernant le certificat

### Différence entre OCSP et Points de distribution de la liste de révocation (CRL)

**OCSP** (RFC 2560) est un protocole standard qui se compose d'un **client OCSP et d'un répondeur OCSP**. Ce protocole **détermine l'état de révocation d'un certificat numérique public** **sans** avoir à **télécharger** l'**ensemble de la CRL**.\
**CRL** est la **méthode traditionnelle** de vérification de la validité du certificat. Une **CRL fournit une liste de numéros de série de certificat** qui ont été révoqués ou ne sont plus valides. Les CRL permettent au vérificateur de vérifier l'état de révocation du certificat présenté tout en le vérifiant. Les CRL sont limitées à 512 entrées.\
De [ici](https://www.arubanetworks.com/techdocs/ArubaOS%206\_3\_1\_Web\_Help/Content/ArubaFrameStyles/CertRevocation/About\_OCSP\_and\_CRL.htm).

### Qu'est-ce que la transparence des certificats

La transparence des certificats vise à remédier aux menaces basées sur les certificats en **rendant l'émission et l'existence des certificats SSL ouvertes à l'examen par les propriétaires de domaines, les AC et les utilisateurs de domaines**. Plus précisément, la transparence des certificats a trois objectifs principaux :

* Rendre impossible (ou du moins très difficile) à une AC d'**émettre un certificat SSL pour un domaine sans que le propriétaire** de ce domaine ne le voie.
* Fournir un **système d'audit et de surveillance ouvert** permettant à tout propriétaire de domaine ou AC de déterminer si des certificats ont été émis par erreur ou de manière malveillante.
* **Protéger les utilisateurs** (autant que possible) contre les certificats émis par erreur ou de manière malveillante.

#### **Journaux de certificats**

Les journaux de certificats sont des services réseau simples qui conservent des **enregistrements de certificats cryptographiquement assurés, vérifiables publiquement, en ajout seulement**. **N'importe qui peut soumettre des certificats à un journal**, bien que les autorités de certification soient susceptibles d'être les principaux soumissionnaires. De même, n'importe qui peut interroger un journal pour obtenir une preuve cryptographique, qui peut être utilisée pour vérifier que le journal fonctionne correctement ou vérifier qu'un certificat particulier a été enregistré. Le nombre de serveurs de journaux n'a pas besoin d'être important (disons, beaucoup moins d'un millier dans le monde entier), et chacun pourrait être exploité indépendamment par une AC, un FAI ou toute autre partie intéressée.

#### Requête

Vous pouvez interroger les journaux de transparence des certificats de n'importe quel domaine sur [https://crt.sh/](https://crt.sh).

## Formats

Il existe différents formats pouvant être utilisés pour stocker un certificat.

#### **Format PEM**

* C'est le format le plus courant utilisé pour les certificats
* La plupart des serveurs (par exemple Apache) s'attendent à ce que les certificats et la clé privée soient dans des fichiers séparés\
\- Habituellement, ils sont des fichiers ASCII encodés en Base64\
\- Les extensions utilisées pour les certificats PEM sont .cer, .crt, .pem, .key\
\- Apache et des serveurs similaires utilisent des certificats au format PEM

#### **Format DER**

* Le format DER est la forme binaire du certificat
* Tous les types de certificats et de clés privées peuvent être encodés au format DER
* Les certificats formatés en DER ne contiennent pas les déclarations "BEGIN CERTIFICATE/END CERTIFICATE"
* Les certificats formatés en DER utilisent le plus souvent les extensions ‘.cer’ et '.der'
* DER est généralement utilisé dans les plates-formes Java

#### **Format P7B/PKCS#7**

* Le format PKCS#7 ou P7B est stocké au format ASCII Base64 et a une extension de fichier .p7b ou .p7c
* Un fichier P7B contient uniquement des certificats et des certificats de chaîne (AC intermédiaires), pas la clé privée
* Les plates-formes les plus courantes qui prennent en charge les fichiers P7B sont Microsoft Windows et Java Tomcat

#### **Format PFX/P12/PKCS#12**

* Le format PKCS#12 ou PFX/P12 est un format binaire pour stocker le certificat du serveur, les certificats intermédiaires et la clé privée dans un fichier chiffrable
* Ces fichiers ont généralement des extensions telles que .pfx et .p12
* Ils sont généralement utilisés sur les machines Windows pour importer et exporter des certificats et des clés privées

### Conversions de formats

**Convertir x509 en PEM**
```
openssl x509 -in certificatename.cer -outform PEM -out certificatename.pem
```
#### **Convertir PEM en DER**
```
openssl x509 -outform der -in certificatename.pem -out certificatename.der
```
**Convertir DER en PEM**
```
openssl x509 -inform der -in certificatename.der -out certificatename.pem
```
**Convertir PEM en P7B**

**Remarque :** Le format PKCS#7 ou P7B est stocké au format ASCII Base64 et a une extension de fichier .p7b ou .p7c. Un fichier P7B ne contient que des certificats et des certificats de chaîne (CA intermédiaires), pas la clé privée. Les plateformes les plus courantes prenant en charge les fichiers P7B sont Microsoft Windows et Java Tomcat.
```
openssl crl2pkcs7 -nocrl -certfile certificatename.pem -out certificatename.p7b -certfile CACert.cer
```
**Convertir PKCS7 en PEM**
```
openssl pkcs7 -print_certs -in certificatename.p7b -out certificatename.pem
```
**Convertir pfx en PEM**

**Remarque :** Le format PKCS#12 ou PFX est un format binaire pour stocker le certificat du serveur, les certificats intermédiaires et la clé privée dans un fichier chiffrable. Les fichiers PFX ont généralement des extensions telles que .pfx et .p12. Les fichiers PFX sont généralement utilisés sur les machines Windows pour importer et exporter des certificats et des clés privées.
```
openssl pkcs12 -in certificatename.pfx -out certificatename.pem
```
**Convertir PFX en PKCS#8**\
**Remarque :** Cela nécessite 2 commandes

**1- Convertir PFX en PEM**
```
openssl pkcs12 -in certificatename.pfx -nocerts -nodes -out certificatename.pem
```
**2- Convertir PEM en PKCS8**
```
openSSL pkcs8 -in certificatename.pem -topk8 -nocrypt -out certificatename.pk8
```
**Convertir P7B en PFX**\
**Remarque :** Cela nécessite 2 commandes

1- **Convertir P7B en CER**
```
openssl pkcs7 -print_certs -in certificatename.p7b -out certificatename.cer
```
**2- Convertir un certificat CER et une clé privée en PFX**
```
openssl pkcs12 -export -in certificatename.cer -inkey privateKey.key -out certificatename.pfx -certfile  cacert.cer
```
<figure><img src="../.gitbook/assets/image (3) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Utilisez [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) pour construire facilement et **automatiser des workflows** alimentés par les outils communautaires les plus avancés au monde.\
Accédez-y aujourd'hui :

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><strong>Apprenez le piratage AWS de zéro à héros avec</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (Expert de l'équipe rouge AWS de HackTricks)</strong></a><strong>!</strong></summary>

Autres façons de soutenir HackTricks :

* Si vous souhaitez voir votre **entreprise annoncée dans HackTricks** ou **télécharger HackTricks en PDF**, consultez les [**PLANS D'ABONNEMENT**](https://github.com/sponsors/carlospolop) !
* Obtenez le [**swag officiel PEASS & HackTricks**](https://peass.creator-spring.com)
* Découvrez [**La famille PEASS**](https://opensea.io/collection/the-peass-family), notre collection exclusive de [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Rejoignez le** 💬 [**groupe Discord**](https://discord.gg/hRep4RUj7f) ou le [**groupe Telegram**](https://t.me/peass) ou **suivez-nous** sur **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Partagez vos astuces de piratage en soumettant des PR aux** [**HackTricks**](https://github.com/carlospolop/hacktricks) et [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) dépôts GitHub.

</details>
