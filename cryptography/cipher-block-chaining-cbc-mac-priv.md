<details>

<summary><strong>Apprenez le piratage AWS de zéro à héros avec</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (Expert de l'équipe rouge AWS de HackTricks)</strong></a><strong>!</strong></summary>

Autres façons de soutenir HackTricks :

* Si vous souhaitez voir votre **entreprise annoncée dans HackTricks** ou **télécharger HackTricks en PDF**, consultez les [**PLANS D'ABONNEMENT**](https://github.com/sponsors/carlospolop) !
* Obtenez le [**swag officiel PEASS & HackTricks**](https://peass.creator-spring.com)
* Découvrez [**La famille PEASS**](https://opensea.io/collection/the-peass-family), notre collection exclusive de [**NFT**](https://opensea.io/collection/the-peass-family)
* **Rejoignez le** 💬 [**groupe Discord**](https://discord.gg/hRep4RUj7f) ou le [**groupe Telegram**](https://t.me/peass) ou **suivez-nous** sur **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Partagez vos astuces de piratage en soumettant des PR aux** référentiels [**HackTricks**](https://github.com/carlospolop/hacktricks) et [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>


# CBC

Si le **cookie** est **seulement** le **nom d'utilisateur** (ou la première partie du cookie est le nom d'utilisateur) et que vous souhaitez vous faire passer pour l'utilisateur "**admin**". Ensuite, vous pouvez créer le nom d'utilisateur **"bdmin"** et **bruteforcer** le **premier octet** du cookie.

# CBC-MAC

En cryptographie, un **code d'authentification de message par chiffrement en mode de chaînage de blocs** (**CBC-MAC**) est une technique pour construire un code d'authentification de message à partir d'un chiffrement par blocs. Le message est chiffré avec un algorithme de chiffrement par blocs en mode CBC pour créer une **chaîne de blocs de telle sorte que chaque bloc dépende du chiffrement correct du bloc précédent**. Cette interdépendance garantit qu'un **changement** de **n'importe quel** bit du texte en clair provoquera un **changement** du **dernier bloc chiffré** d'une manière qui ne peut être prédite ou contrée sans connaître la clé du chiffrement par blocs.

Pour calculer le CBC-MAC du message m, on chiffre m en mode CBC avec un vecteur d'initialisation nul et on garde le dernier bloc. La figure suivante esquisse le calcul du CBC-MAC d'un message comprenant des blocs ![m\_{1}\\|m\_{2}\\|\cdots \\|m\_{x}](https://wikimedia.org/api/rest\_v1/media/math/render/svg/bbafe7330a5e40a04f01cc776c9d94fe914b17f5) en utilisant une clé secrète k et un chiffrement par blocs E :

![CBC-MAC structure (en).svg](https://upload.wikimedia.org/wikipedia/commons/thumb/b/bf/CBC-MAC\_structure\_\(en\).svg/570px-CBC-MAC\_structure\_\(en\).svg.png)

# Vulnérabilité

Avec le CBC-MAC, l'**IV utilisé est généralement 0**.\
C'est un problème car 2 messages connus (`m1` et `m2`) généreront indépendamment 2 signatures (`s1` et `s2`). Ainsi :

* `E(m1 XOR 0) = s1`
* `E(m2 XOR 0) = s2`

Ensuite, un message composé de m1 et m2 concaténés (m3) générera 2 signatures (s31 et s32) :

* `E(m1 XOR 0) = s31 = s1`
* `E(m2 XOR s1) = s32`

**Ce qui est possible à calculer sans connaître la clé du chiffrement.**

Imaginez que vous chiffrez le nom **Administrateur** en blocs de **8 octets** :

* `Administ`
* `rator\00\00\00`

Vous pouvez créer un nom d'utilisateur appelé **Administ** (m1) et récupérer la signature (s1).\
Ensuite, vous pouvez créer un nom d'utilisateur appelé le résultat de `rator\00\00\00 XOR s1`. Cela générera `E(m2 XOR s1 XOR 0)` qui est s32.\
maintenant, vous pouvez utiliser s32 comme signature du nom complet **Administrateur**.

### Résumé

1. Obtenez la signature du nom d'utilisateur **Administ** (m1) qui est s1
2. Obtenez la signature du nom d'utilisateur **rator\x00\x00\x00 XOR s1 XOR 0** qui est s32**.**
3. Définissez le cookie sur s32 et il sera un cookie valide pour l'utilisateur **Administrateur**.

# Contrôle de l'attaque IV

Si vous pouvez contrôler l'IV utilisé, l'attaque pourrait être très facile.\
Si les cookies ne sont que le nom d'utilisateur chiffré, pour vous faire passer pour l'utilisateur "**administrateur**", vous pouvez créer l'utilisateur "**Administrateur**" et obtenir son cookie.\
Maintenant, si vous pouvez contrôler l'IV, vous pouvez changer le premier octet de l'IV de sorte que **IV\[0] XOR "A" == IV'\[0] XOR "a"** et régénérer le cookie pour l'utilisateur **Administrateur**. Ce cookie sera valide pour **usurper** l'utilisateur **administrateur** avec l'IV initial.

# Références

Plus d'informations sur [https://en.wikipedia.org/wiki/CBC-MAC](https://en.wikipedia.org/wiki/CBC-MAC)


<details>

<summary><strong>Apprenez le piratage AWS de zéro à héros avec</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (Expert de l'équipe rouge AWS de HackTricks)</strong></a><strong>!</strong></summary>

Autres façons de soutenir HackTricks :

* Si vous souhaitez voir votre **entreprise annoncée dans HackTricks** ou **télécharger HackTricks en PDF**, consultez les [**PLANS D'ABONNEMENT**](https://github.com/sponsors/carlospolop) !
* Obtenez le [**swag officiel PEASS & HackTricks**](https://peass.creator-spring.com)
* Découvrez [**La famille PEASS**](https://opensea.io/collection/the-peass-family), notre collection exclusive de [**NFT**](https://opensea.io/collection/the-peass-family)
* **Rejoignez le** 💬 [**groupe Discord**](https://discord.gg/hRep4RUj7f) ou le [**groupe Telegram**](https://t.me/peass) ou **suivez-nous** sur **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Partagez vos astuces de piratage en soumettant des PR aux** référentiels [**HackTricks**](https://github.com/carlospolop/hacktricks) et [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
