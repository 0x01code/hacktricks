# Analyse de fichiers de bureau

<details>

<summary><strong>Apprenez le piratage AWS de zéro à héros avec</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (Expert en équipe rouge AWS de HackTricks)</strong></a><strong>!</strong></summary>

Autres façons de soutenir HackTricks :

* Si vous souhaitez voir votre **entreprise annoncée dans HackTricks** ou **télécharger HackTricks en PDF**, consultez les [**PLANS D'ABONNEMENT**](https://github.com/sponsors/carlospolop) !
* Obtenez le [**swag officiel PEASS & HackTricks**](https://peass.creator-spring.com)
* Découvrez [**La famille PEASS**](https://opensea.io/collection/the-peass-family), notre collection exclusive de [**NFT**](https://opensea.io/collection/the-peass-family)
* **Rejoignez le** 💬 [**groupe Discord**](https://discord.gg/hRep4RUj7f) ou le [**groupe Telegram**](https://t.me/peass) ou **suivez-nous** sur **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Partagez vos astuces de piratage en soumettant des PR aux** [**HackTricks**](https://github.com/carlospolop/hacktricks) et [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) dépôts GitHub.

</details>

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Utilisez [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) pour construire et **automatiser facilement des flux de travail** alimentés par les outils communautaires les plus avancés au monde.\
Accédez dès aujourd'hui :

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## Introduction

Microsoft a créé **des dizaines de formats de fichiers de documents de bureau**, dont beaucoup sont populaires pour la distribution d'attaques de phishing et de logiciels malveillants en raison de leur capacité à **inclure des macros** (scripts VBA).

De manière générale, il existe deux générations de formats de fichiers Office : les **formats OLE** (extensions de fichier comme RTF, DOC, XLS, PPT) et les formats "**Office Open XML**" (extensions de fichier incluant DOCX, XLSX, PPTX). **Les deux** formats sont des formats binaires de fichiers composés structurés qui **permettent le contenu lié ou intégré** (objets). Les fichiers OOXML sont des conteneurs de fichiers zip, ce qui signifie que l'une des façons les plus simples de vérifier les données cachées est simplement de `dézipper` le document :
```
$ unzip example.docx
Archive:  example.docx
inflating: [Content_Types].xml
inflating: _rels/.rels
inflating: word/_rels/document.xml.rels
inflating: word/document.xml
inflating: word/theme/theme1.xml
extracting: docProps/thumbnail.jpeg
inflating: word/comments.xml
inflating: word/settings.xml
inflating: word/fontTable.xml
inflating: word/styles.xml
inflating: word/stylesWithEffects.xml
inflating: docProps/app.xml
inflating: docProps/core.xml
inflating: word/webSettings.xml
inflating: word/numbering.xml
$ tree
.
├── [Content_Types].xml
├── _rels
├── docProps
│   ├── app.xml
│   ├── core.xml
│   └── thumbnail.jpeg
└── word
├── _rels
│   └── document.xml.rels
├── comments.xml
├── document.xml
├── fontTable.xml
├── numbering.xml
├── settings.xml
├── styles.xml
├── stylesWithEffects.xml
├── theme
│   └── theme1.xml
└── webSettings.xml
```
Comme vous pouvez le voir, une partie de la structure est créée par la hiérarchie des fichiers et dossiers. Le reste est spécifié à l'intérieur des fichiers XML. [_Nouvelles techniques stéganographiques pour le format de fichier OOXML_, 2011](http://download.springer.com/static/pdf/713/chp%3A10.1007%2F978-3-642-23300-5\_27.pdf?originUrl=http%3A%2F%2Flink.springer.com%2Fchapter%2F10.1007%2F978-3-642-23300-5\_27\&token2=exp=1497911340\~acl=%2Fstatic%2Fpdf%2F713%2Fchp%25253A10.1007%25252F978-3-642-23300-5\_27.pdf%3ForiginUrl%3Dhttp%253A%252F%252Flink.springer.com%252Fchapter%252F10.1007%252F978-3-642-23300-5\_27\*\~hmac=aca7e2655354b656ca7d699e8e68ceb19a95bcf64e1ac67354d8bca04146fd3d) détaille certaines idées de techniques de dissimulation de données, mais les auteurs de défis CTF en créeront toujours de nouvelles.

Encore une fois, un ensemble d'outils Python existe pour l'examen et **l'analyse des documents OLE et OOXML** : [oletools](http://www.decalage.info/python/oletools). Pour les documents OOXML en particulier, [OfficeDissector](https://www.officedissector.com) est un cadre d'analyse très puissant (et une bibliothèque Python). Ce dernier inclut un [guide rapide de son utilisation](https://github.com/grierforensics/officedissector/blob/master/doc/html/\_sources/txt/ANALYZING\_OOXML.txt).

Parfois, le défi n'est pas de trouver des données statiques cachées, mais d'**analyser une macro VBA** pour déterminer son comportement. Il s'agit d'un scénario plus réaliste et une tâche que les analystes sur le terrain effectuent tous les jours. Les outils de dissèquement mentionnés peuvent indiquer si une macro est présente, et probablement l'extraire pour vous. Une macro VBA typique dans un document Office, sur Windows, téléchargera un script PowerShell vers %TEMP% et tentera de l'exécuter, auquel cas vous aurez maintenant une tâche d'analyse de script PowerShell. Mais les macros VBA malveillantes sont rarement compliquées car VBA est [généralement utilisé comme une plateforme de lancement pour l'exécution de code](https://www.lastline.com/labsblog/party-like-its-1999-comeback-of-vba-malware-downloaders-part-3/). Dans le cas où vous devez comprendre une macro VBA compliquée, ou si la macro est obfusquée et possède une routine de décompactage, vous n'avez pas besoin de posséder une licence Microsoft Office pour déboguer cela. Vous pouvez utiliser [Libre Office](http://libreoffice.org) : [son interface](http://www.debugpoint.com/2014/09/debugging-libreoffice-macro-basic-using-breakpoint-and-watch/) sera familière à toute personne ayant débogué un programme ; vous pouvez définir des points d'arrêt et créer des variables de surveillance et capturer des valeurs après leur décompactage mais avant que le comportement de la charge utile ne s'exécute. Vous pouvez même démarrer une macro d'un document spécifique à partir d'une ligne de commande :
```
$ soffice path/to/test.docx macro://./standard.module1.mymacro
```
## [oletools](https://github.com/decalage2/oletools)
```bash
sudo pip3 install -U oletools
olevba -c /path/to/document #Extract macros
```
## Exécution automatique

Les fonctions de macro comme `AutoOpen`, `AutoExec` ou `Document_Open` seront **automatiquement exécutées**.

## Références

* [https://trailofbits.github.io/ctf/forensics/](https://trailofbits.github.io/ctf/forensics/)

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Utilisez [**Trickest**](https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks) pour construire facilement et **automatiser des workflows** alimentés par les outils communautaires les plus avancés au monde.\
Accédez dès aujourd'hui :

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><strong>Apprenez le piratage AWS de zéro à héros avec</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Autres façons de soutenir HackTricks :

* Si vous souhaitez voir votre **entreprise annoncée dans HackTricks** ou **télécharger HackTricks en PDF**, consultez les [**PLANS D'ABONNEMENT**](https://github.com/sponsors/carlospolop) !
* Obtenez le [**swag officiel PEASS & HackTricks**](https://peass.creator-spring.com)
* Découvrez [**The PEASS Family**](https://opensea.io/collection/the-peass-family), notre collection exclusive de [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Rejoignez le** 💬 [**groupe Discord**](https://discord.gg/hRep4RUj7f) ou le [**groupe Telegram**](https://t.me/peass) ou **suivez-nous** sur **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Partagez vos astuces de piratage en soumettant des PR aux** [**HackTricks**](https://github.com/carlospolop/hacktricks) et [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
