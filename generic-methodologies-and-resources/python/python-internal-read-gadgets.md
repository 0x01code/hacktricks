# Gadgets de Lecture Interne Python

<details>

<summary><strong>Apprenez le piratage AWS de zéro à héros avec</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (Expert en équipe rouge AWS de HackTricks)</strong></a><strong>!</strong></summary>

Autres façons de soutenir HackTricks :

* Si vous souhaitez voir votre **entreprise annoncée dans HackTricks** ou **télécharger HackTricks en PDF**, consultez les [**PLANS D'ABONNEMENT**](https://github.com/sponsors/carlospolop) !
* Obtenez le [**swag officiel PEASS & HackTricks**](https://peass.creator-spring.com)
* Découvrez [**La Famille PEASS**](https://opensea.io/collection/the-peass-family), notre collection exclusive de [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Rejoignez le** 💬 [**groupe Discord**](https://discord.gg/hRep4RUj7f) ou le [**groupe Telegram**](https://t.me/peass) ou **suivez-nous** sur **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Partagez vos astuces de piratage en soumettant des PR aux** [**HackTricks**](https://github.com/carlospolop/hacktricks) et [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) dépôts GitHub.

</details>

## Informations de Base

Différentes vulnérabilités telles que les [**Chaines de Format Python**](bypass-python-sandboxes/#python-format-string) ou la [**Pollution de Classe**](class-pollution-pythons-prototype-pollution.md) pourraient vous permettre de **lire des données internes Python mais ne vous permettront pas d'exécuter du code**. Par conséquent, un testeur d'intrusion devra tirer le meilleur parti de ces autorisations de lecture pour **obtenir des privilèges sensibles et escalader la vulnérabilité**.

### Flask - Lire la clé secrète

La page principale d'une application Flask aura probablement l'objet global **`app`** où ce **secret est configuré**.
```python
app = Flask(__name__, template_folder='templates')
app.secret_key = '(:secret:)'
```
Dans ce cas, il est possible d'accéder à cet objet en utilisant simplement n'importe quel gadget pour **accéder aux objets globaux** de la [**page de contournement des sandbox Python**](bypass-python-sandboxes/).

Dans le cas où **la vulnérabilité se trouve dans un fichier Python différent**, vous avez besoin d'un gadget pour parcourir les fichiers pour accéder à celui principal afin de **accéder à l'objet global `app.secret_key`** pour changer la clé secrète de Flask et pouvoir [**escalader les privilèges** en connaissant cette clé](../../network-services-pentesting/pentesting-web/flask.md#flask-unsign).

Une charge utile comme celle-ci [de ce writeup](https://ctftime.org/writeup/36082):

{% code overflow="wrap" %}
```python
__init__.__globals__.__loader__.__init__.__globals__.sys.modules.__main__.app.secret_key
```
{% endcode %}

Utilisez cette charge utile pour **changer `app.secret_key`** (le nom dans votre application peut être différent) afin de pouvoir signer de nouveaux cookies flask avec plus de privilèges.

### Werkzeug - machine\_id et node uuid

[**En utilisant cette charge utile de ce writeup**](https://vozec.fr/writeups/tweedle-dum-dee/), vous pourrez accéder à **machine\_id** et à l'**uuid** du nœud, qui sont les **secrets principaux** dont vous avez besoin pour [**générer le code pin Werkzeug**](../../network-services-pentesting/pentesting-web/werkzeug.md) que vous pouvez utiliser pour accéder à la console Python dans `/console` si le **mode de débogage est activé:**
```python
{ua.__class__.__init__.__globals__[t].sys.modules[werkzeug.debug]._machine_id}
{ua.__class__.__init__.__globals__[t].sys.modules[werkzeug.debug].uuid._node}
```
{% hint style="warning" %}
Notez que vous pouvez obtenir le **chemin local des serveurs vers le fichier `app.py`** en générant une **erreur** sur la page web qui vous **donnera le chemin**.
{% endhint %}

Si la vulnérabilité se trouve dans un fichier python différent, vérifiez le truc Flask précédent pour accéder aux objets depuis le fichier python principal.

<details>

<summary><strong>Apprenez le piratage AWS de zéro à héros avec</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (Expert de l'équipe rouge AWS de HackTricks)</strong></a><strong>!</strong></summary>

Autres façons de soutenir HackTricks:

* Si vous souhaitez voir votre **entreprise annoncée dans HackTricks** ou **télécharger HackTricks en PDF**, consultez les [**PLANS D'ABONNEMENT**](https://github.com/sponsors/carlospolop)!
* Obtenez le [**swag officiel PEASS & HackTricks**](https://peass.creator-spring.com)
* Découvrez [**La famille PEASS**](https://opensea.io/collection/the-peass-family), notre collection exclusive de [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Rejoignez le** 💬 [**groupe Discord**](https://discord.gg/hRep4RUj7f) ou le [**groupe Telegram**](https://t.me/peass) ou **suivez-nous** sur **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Partagez vos astuces de piratage en soumettant des PR aux** [**HackTricks**](https://github.com/carlospolop/hacktricks) et [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) dépôts GitHub.

</details>
