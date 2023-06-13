# Numéro de série macOS

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* Travaillez-vous dans une **entreprise de cybersécurité** ? Voulez-vous voir votre **entreprise annoncée dans HackTricks** ? ou voulez-vous avoir accès à la **dernière version de PEASS ou télécharger HackTricks en PDF** ? Consultez les [**PLANS D'ABONNEMENT**](https://github.com/sponsors/carlospolop) !
* Découvrez [**The PEASS Family**](https://opensea.io/collection/the-peass-family), notre collection exclusive de [**NFTs**](https://opensea.io/collection/the-peass-family)
* Obtenez le [**swag officiel PEASS & HackTricks**](https://peass.creator-spring.com)
* **Rejoignez le** [**💬**](https://emojipedia.org/speech-balloon/) [**groupe Discord**](https://discord.gg/hRep4RUj7f) ou le [**groupe telegram**](https://t.me/peass) ou **suivez** moi sur **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Partagez vos astuces de piratage en soumettant des PR au** [**repo hacktricks**](https://github.com/carlospolop/hacktricks) **et au** [**repo hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>

Les appareils Apple fabriqués après 2010 ont généralement des numéros de série alphanumériques de **12 caractères**, les **trois premiers chiffres représentant le lieu de fabrication**, les **deux suivants indiquant l'année** et la **semaine** de fabrication, les **trois chiffres suivants fournissant un identifiant unique**, et les **quatre derniers chiffres représentant le numéro de modèle**.

Exemple de numéro de série : **C02L13ECF8J2**

### **3 - Lieux de fabrication**

| Code           | Usine                                        |
| -------------- | -------------------------------------------- |
| FC             | Fountain Colorado, USA                       |
| F              | Fremont, Californie, USA                     |
| XA, XB, QP, G8 | USA                                          |
| RN             | Mexique                                      |
| CK             | Cork, Irlande                                 |
| VM             | Foxconn, Pardubice, République tchèque       |
| SG, E          | Singapour                                    |
| MB             | Malaisie                                     |
| PT, CY         | Corée                                        |
| EE, QT, UV     | Taïwan                                       |
| FK, F1, F2     | Foxconn - Zhengzhou, Chine                   |
| W8             | Shanghai Chine                               |
| DL, DM         | Foxconn - Chine                              |
| DN             | Foxconn, Chengdu, Chine                      |
| YM, 7J         | Hon Hai/Foxconn, Chine                       |
| 1C, 4H, WQ, F7 | Chine                                        |
| C0             | Tech Com - Filiale de Quanta Computer, Chine |
| C3             | Foxxcon, Shenzhen, Chine                     |
| C7             | Pentagone, Changhai, Chine                   |
| RM             | Remis à neuf/remanufacturé                   |

### 1 - Année de fabrication

| Code | Sortie               |
| ---- | -------------------- |
| C    | 2010/2020 (1er sem.) |
| D    | 2010/2020 (2ème sem.)|
| F    | 2011/2021 (1er sem.) |
| G    | 2011/2021 (2ème sem.)|
| H    | 2012/... (1er sem.)  |
| J    | 2012 (2ème sem.)     |
| K    | 2013 (1er sem.)      |
| L    | 2013 (2ème sem.)     |
| M    | 2014 (1er sem.)      |
| N    | 2014 (2ème sem.)     |
| P    | 2015 (1er sem.)      |
| Q    | 2015 (2ème sem.)     |
| R    | 2016 (1er sem.)      |
| S    | 2016 (2ème sem.)     |
| T    | 2017 (1er sem.)      |
| V    | 2017 (2ème sem.)     |
| W    | 2018 (1er sem.)      |
| X    | 2018 (2ème sem.)     |
| Y    | 2019 (1er sem.)      |
| Z    | 2019 (2ème sem.)     |

### 1 - Semaine de fabrication

Le cinquième caractère représente la semaine de fabrication de l'appareil. Il y a 28 caractères possibles à cet endroit : **les chiffres de 1 à 9 sont utilisés pour représenter les semaines de la première à la neuvième**, et les **caractères C à Y**, **à l'exception** des voyelles A, E, I, O et U, et de la lettre S, représentent les **dixième à vingt-septième semaines**. Pour les appareils fabriqués au **deuxième semestre de l'année, ajoutez 26** au nombre représenté par le cinquième caractère du numéro de série. Par exemple, un produit dont les quatrième et cinquième chiffres sont "JH" a été fabriqué dans la 40ème semaine de 2012.

### 3 - Code unique

Les trois chiffres suivants sont un code d'identification qui **sert à différencier chaque appareil Apple du même modèle** qui est fabriqué dans le même lieu et pendant la même semaine de la même année, en veillant à ce que chaque appareil ait un numéro de série différent.

### 4 - Numéro de série

Les quatre derniers chiffres du numéro de série représentent le **modèle du produit**.

### Référence

{% embed url="https://beetstech.com/blog/decode-meaning-behind-apple-serial-number" %}

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* Travaillez-vous dans une **entreprise de cybersécurité** ? Voulez-vous voir votre **entreprise annoncée dans HackTricks** ? ou voulez-vous avoir accès à la **dernière version de PEASS ou télécharger HackTricks en PDF** ? Consultez les [**PLANS D'ABONNEMENT**](https://github.com/sponsors/carlospolop) !
* Découvrez [**The PEASS Family**](https://opensea.io/collection/the-peass-family), notre collection exclusive de [**NFTs**](https://opensea.io/collection/the-peass-family)
* Obtenez le [**swag officiel PEASS & HackTricks**](https://peass.creator-spring.com)
* **Rejoignez le** [**💬**](https://emojipedia.org/speech-balloon/) [**groupe Discord**](https://discord.gg/hRep4RUj7f) ou le [**groupe telegram**](https://t.me/peass) ou **suivez** moi sur **Twitter** [**🐦**
