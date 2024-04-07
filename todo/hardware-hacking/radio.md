# Radio

<details>

<summary><strong>Apprenez le piratage AWS de zéro à héros avec</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Autres façons de soutenir HackTricks :

* Si vous souhaitez voir votre **entreprise annoncée dans HackTricks** ou **télécharger HackTricks en PDF**, consultez les [**PLANS D'ABONNEMENT**](https://github.com/sponsors/carlospolop) !
* Obtenez le [**swag officiel PEASS & HackTricks**](https://peass.creator-spring.com)
* Découvrez [**La famille PEASS**](https://opensea.io/collection/the-peass-family), notre collection exclusive de [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Rejoignez le** 💬 [**groupe Discord**](https://discord.gg/hRep4RUj7f) ou le [**groupe Telegram**](https://t.me/peass) ou **suivez-nous** sur **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Partagez vos astuces de piratage en soumettant des PR aux** [**HackTricks**](https://github.com/carlospolop/hacktricks) et [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) dépôts GitHub.

</details>

## SigDigger

[**SigDigger** ](https://github.com/BatchDrake/SigDigger)est un analyseur de signal numérique gratuit pour GNU/Linux et macOS, conçu pour extraire des informations de signaux radio inconnus. Il prend en charge une variété de périphériques SDR via SoapySDR, permet une démodulation ajustable des signaux FSK, PSK et ASK, décode la vidéo analogique, analyse les signaux en rafales et écoute les canaux vocaux analogiques (le tout en temps réel).

### Configuration de base

Après l'installation, il y a quelques éléments que vous pourriez envisager de configurer.\
Dans les paramètres (le deuxième bouton d'onglet), vous pouvez sélectionner le **périphérique SDR** ou **sélectionner un fichier** à lire et quelle fréquence syntoniser et le taux d'échantillonnage (recommandé jusqu'à 2,56 Msps si votre PC le prend en charge)\\

![](<../../.gitbook/assets/image (242).png>)

Dans le comportement de l'interface graphique, il est recommandé d'activer quelques éléments si votre PC le prend en charge :

![](<../../.gitbook/assets/image (469).png>)

{% hint style="info" %}
Si vous réalisez que votre PC ne capture pas les choses, essayez de désactiver OpenGL et de réduire le taux d'échantillonnage.
{% endhint %}

### Utilisations

* Juste pour **capturer un signal pendant un certain temps et l'analyser**, maintenez simplement le bouton "Appuyer pour capturer" aussi longtemps que nécessaire.

![](<../../.gitbook/assets/image (957).png>)

* Le **Syntoniseur** de SigDigger aide à **capturer de meilleurs signaux** (mais peut aussi les dégrader). Idéalement, commencez par 0 et continuez à **l'augmenter jusqu'à** ce que vous trouviez que le **bruit** introduit est **plus important** que **l'amélioration du signal** dont vous avez besoin).

![](<../../.gitbook/assets/image (1096).png>)

### Synchronisation avec le canal radio

Avec [**SigDigger** ](https://github.com/BatchDrake/SigDigger)synchronisez-vous avec le canal que vous souhaitez écouter, configurez l'option "Aperçu audio de la bande de base", configurez la bande passante pour obtenir toutes les informations envoyées, puis réglez le Syntoniseur au niveau avant que le bruit ne commence vraiment à augmenter :

![](<../../.gitbook/assets/image (582).png>)

## Astuces intéressantes

* Lorsqu'un appareil envoie des rafales d'informations, généralement la **première partie sera un préambule** donc vous **n'avez pas besoin de vous inquiéter** si vous **ne trouvez pas d'informations** là-dedans **ou s'il y a des erreurs**.
* Dans les trames d'informations, vous devriez normalement **trouver différentes trames bien alignées entre elles** :

![](<../../.gitbook/assets/image (1073).png>)

![](<../../.gitbook/assets/image (594).png>)

* **Après avoir récupéré les bits, vous devrez peut-être les traiter d'une certaine manière**. Par exemple, dans la codification Manchester, un haut + bas sera un 1 ou 0 et un bas + haut sera l'autre. Ainsi, les paires de 1 et 0 (hauts et bas) seront un vrai 1 ou un vrai 0.
* Même si un signal utilise la codification Manchester (il est impossible de trouver plus de deux 0 ou 1 consécutifs), vous pourriez **trouver plusieurs 1 ou 0 ensemble dans le préambule**!

### Découverte du type de modulation avec IQ

Il existe 3 façons de stocker des informations dans les signaux : en modulant l'**amplitude**, la **fréquence** ou la **phase**.\
Si vous vérifiez un signal, il existe différentes façons d'essayer de comprendre ce qui est utilisé pour stocker les informations (trouvez plus de façons ci-dessous), mais une bonne méthode est de vérifier le graphique IQ.

![](<../../.gitbook/assets/image (785).png>)

* **Détection de l'AM** : Si dans le graphique IQ apparaissent par exemple **2 cercles** (probablement un à 0 et l'autre à une amplitude différente), cela pourrait signifier qu'il s'agit d'un signal AM. Cela est dû au fait que dans le graphique IQ, la distance entre le 0 et le cercle est l'amplitude du signal, il est donc facile de visualiser différentes amplitudes utilisées.
* **Détection de la PM** : Comme dans l'image précédente, si vous trouvez de petits cercles non liés entre eux, cela signifie probablement qu'une modulation de phase est utilisée. Cela est dû au fait que dans le graphique IQ, l'angle entre le point et le 0,0 est la phase du signal, ce qui signifie que 4 phases différentes sont utilisées.
* Notez que si l'information est cachée dans le fait qu'une phase est modifiée et non dans la phase elle-même, vous ne verrez pas clairement des phases différentes différenciées.
* **Détection de la FM** : L'IQ n'a pas de champ pour identifier les fréquences (la distance au centre est l'amplitude et l'angle est la phase).\
Par conséquent, pour identifier la FM, vous devriez **voir essentiellement un cercle** dans ce graphique.\
De plus, une fréquence différente est "représentée" dans le graphique IQ par une **accélération de vitesse à travers le cercle** (donc dans SysDigger en sélectionnant le signal, le graphique IQ est peuplé, si vous trouvez une accélération ou un changement de direction dans le cercle créé, cela pourrait signifier que c'est de la FM) :

## Exemple d'AM

{% file src="../../.gitbook/assets/sigdigger_20220308_165547Z_2560000_433500000_float32_iq.raw" %}

### Découverte de l'AM

#### Vérification de l'enveloppe

En vérifiant les informations AM avec [**SigDigger** ](https://github.com/BatchDrake/SigDigger)et en regardant simplement l'**enveloppe**, vous pouvez voir différents niveaux d'amplitude clairs. Le signal utilisé envoie des impulsions avec des informations en AM, voici à quoi ressemble une impulsion :

![](<../../.gitbook/assets/image (587).png>)

Et voici à quoi ressemble une partie du symbole avec la forme d'onde :

![](<../../.gitbook/assets/image (731).png>)

#### Vérification de l'histogramme

Vous pouvez **sélectionner tout le signal** où se trouvent les informations, sélectionner le mode **Amplitude** et **Sélection** et cliquer sur **Histogramme**. Vous pouvez observer que seuls 2 niveaux clairs sont trouvés

![](<../../.gitbook/assets/image (261).png>)

Par exemple, si vous sélectionnez la Fréquence au lieu de l'Amplitude dans ce signal AM, vous ne trouvez qu'une seule fréquence (aucune information modulée en fréquence n'utilise qu'une seule fréquence).

![](<../../.gitbook/assets/image (729).png>)

Si vous trouvez beaucoup de fréquences, potentiellement ce ne sera pas une FM, probablement la fréquence du signal a simplement été modifiée en raison du canal.
#### Avec IQ

Dans cet exemple, vous pouvez voir qu'il y a **un grand cercle** mais aussi **beaucoup de points au centre.**

![](<../../.gitbook/assets/image (219).png>)

### Obtenir le débit de symboles

#### Avec un symbole

Sélectionnez le plus petit symbole que vous pouvez trouver (pour être sûr qu'il s'agit bien de 1) et vérifiez la "Fréquence de sélection". Dans ce cas, elle serait de 1,013 kHz (donc 1 kHz).

![](<../../.gitbook/assets/image (75).png>)

#### Avec un groupe de symboles

Vous pouvez également indiquer le nombre de symboles que vous allez sélectionner et SigDigger calculera la fréquence d'1 symbole (plus il y a de symboles sélectionnés, probablement mieux c'est). Dans ce scénario, j'ai sélectionné 10 symboles et la "Fréquence de sélection" est de 1,004 kHz :

![](<../../.gitbook/assets/image (1005).png>)

### Obtenir les bits

Ayant trouvé qu'il s'agit d'un signal **modulé en AM** et connaissant le **débit de symboles** (et sachant que dans ce cas, quelque chose en hausse signifie 1 et quelque chose en baisse signifie 0), il est très facile d'**obtenir les bits** encodés dans le signal. Ainsi, sélectionnez le signal avec les informations et configurez l'échantillonnage et la décision, puis appuyez sur "échantillon" (vérifiez que l'**Amplitude** est sélectionnée, que le **débit de symboles découvert** est configuré et que la **récupération de l'horloge Gadner** est sélectionnée) :

![](<../../.gitbook/assets/image (962).png>)

* **Synchroniser avec les intervalles de sélection** signifie que si vous avez précédemment sélectionné des intervalles pour trouver le débit de symboles, ce débit de symboles sera utilisé.
* **Manuel** signifie que le débit de symboles indiqué va être utilisé.
* Dans la **Sélection d'intervalles fixes**, vous indiquez le nombre d'intervalles qui doivent être sélectionnés et il calcule le débit de symboles à partir de cela.
* La **récupération de l'horloge Gadner** est généralement la meilleure option, mais vous devez quand même indiquer un débit de symboles approximatif.

En appuyant sur "échantillon", cela apparaît :

![](<../../.gitbook/assets/image (641).png>)

Maintenant, pour faire comprendre à SigDigger **où se situe la plage** du niveau portant des informations, vous devez cliquer sur le **niveau le plus bas** et maintenir cliqué jusqu'au plus grand niveau :

![](<../../.gitbook/assets/image (436).png>)

S'il y avait par exemple **4 niveaux d'amplitude différents**, vous auriez dû configurer les **Bits par symbole à 2** et sélectionner du plus petit au plus grand.

Enfin, en **augmentant** le **zoom** et en **changeant la taille de la ligne**, vous pouvez voir les bits (et vous pouvez tout sélectionner et copier pour obtenir tous les bits) :

![](<../../.gitbook/assets/image (273).png>)

Si le signal a plus d'1 bit par symbole (par exemple 2), SigDigger n'a **aucun moyen de savoir quel symbole est** 00, 01, 10, 11, donc il utilisera différentes **échelles de gris** pour représenter chacun (et si vous copiez les bits, il utilisera des **nombres de 0 à 3**, que vous devrez traiter).

De plus, l'utilisation de **codifications** telles que **Manchester**, et **haut + bas** peut être **1 ou 0** et un bas + haut peut être un 1 ou 0. Dans ces cas, vous devez **traiter les hauts (1) et les bas (0)** obtenus pour substituer les paires de 01 ou 10 en tant que 0 ou 1.

## Exemple de FM

{% file src="../../.gitbook/assets/sigdigger_20220308_170858Z_2560000_433500000_float32_iq.raw" %}

### Découverte de FM

#### Vérification des fréquences et de la forme d'onde

Exemple de signal envoyant des informations modulées en FM :

![](<../../.gitbook/assets/image (722).png>)

Dans l'image précédente, vous pouvez observer assez clairement que **2 fréquences sont utilisées**, mais si vous **observez** la **forme d'onde**, vous pourriez **ne pas être en mesure d'identifier correctement les 2 fréquences différentes** :

![](<../../.gitbook/assets/image (714).png>)

Cela est dû au fait que j'ai capturé le signal dans les deux fréquences, donc l'une est approximativement l'opposée de l'autre :

![](<../../.gitbook/assets/image (939).png>)

Si la fréquence synchronisée est **plus proche d'une fréquence que de l'autre**, vous pouvez facilement voir les 2 fréquences différentes :

![](<../../.gitbook/assets/image (419).png>)

![](<../../.gitbook/assets/image (485).png>)

#### Vérification de l'histogramme

En vérifiant l'histogramme de fréquence du signal avec des informations, vous pouvez facilement voir 2 signaux différents :

![](<../../.gitbook/assets/image (868).png>)

Dans ce cas, si vous vérifiez l'**histogramme d'amplitude**, vous ne trouverez **qu'une seule amplitude**, donc ce **ne peut pas être de l'AM** (si vous trouvez beaucoup d'amplitudes, c'est peut-être parce que le signal a perdu de la puissance le long du canal) :

![](<../../.gitbook/assets/image (814).png>)

Et voici l'histogramme de phase (qui montre très clairement que le signal n'est pas modulé en phase) :

![](<../../.gitbook/assets/image (993).png>)

#### Avec IQ

IQ n'a pas de champ pour identifier les fréquences (la distance au centre est l'amplitude et l'angle est la phase).\
Par conséquent, pour identifier la FM, vous devriez **voir essentiellement un cercle** dans ce graphique.\
De plus, une fréquence différente est "représentée" par le graphique IQ par une **accélération de vitesse à travers le cercle** (donc dans SysDigger en sélectionnant le signal, le graphique IQ est rempli, si vous trouvez une accélération ou un changement de direction dans le cercle créé, cela pourrait signifier que c'est de la FM) :

![](<../../.gitbook/assets/image (78).png>)

### Obtenir le débit de symboles

Vous pouvez utiliser la **même technique que celle utilisée dans l'exemple de l'AM** pour obtenir le débit de symboles une fois que vous avez trouvé les fréquences portant des symboles.

### Obtenir les bits

Vous pouvez utiliser la **même technique que celle utilisée dans l'exemple de l'AM** pour obtenir les bits une fois que vous avez **trouvé que le signal est modulé en fréquence** et le **débit de symboles**.
