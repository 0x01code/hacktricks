# Python Interne Lese-Gadgets

<details>

<summary><strong>Lernen Sie AWS-Hacking von Null auf Held mit</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Andere Möglichkeiten, HackTricks zu unterstützen:

* Wenn Sie Ihr **Unternehmen in HackTricks bewerben möchten** oder **HackTricks als PDF herunterladen möchten**, überprüfen Sie die [**ABONNEMENTPLÄNE**](https://github.com/sponsors/carlospolop)!
* Holen Sie sich das [**offizielle PEASS & HackTricks-Merchandise**](https://peass.creator-spring.com)
* Entdecken Sie [**The PEASS Family**](https://opensea.io/collection/the-peass-family), unsere Sammlung exklusiver [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Treten Sie der** 💬 [**Discord-Gruppe**](https://discord.gg/hRep4RUj7f) oder der [**Telegram-Gruppe**](https://t.me/peass) bei oder **folgen** Sie uns auf **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Teilen Sie Ihre Hacking-Tricks, indem Sie PRs an die** [**HackTricks**](https://github.com/carlospolop/hacktricks) und [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) Github-Repositories senden.

</details>

## Grundlegende Informationen

Verschiedene Schwachstellen wie [**Python-Formatzeichenketten**](bypass-python-sandboxes/#python-format-string) oder [**Klassenverschmutzung**](class-pollution-pythons-prototype-pollution.md) können es Ihnen ermöglichen, **Python-interne Daten zu lesen, aber keinen Code auszuführen**. Daher muss ein Pentester diese Leseberechtigungen bestmöglich nutzen, um **sensible Berechtigungen zu erhalten und die Schwachstelle zu eskalieren**.

### Flask - Geheimen Schlüssel lesen

Die Hauptseite einer Flask-Anwendung wird wahrscheinlich das **`app`**-globale Objekt enthalten, in dem dieser **geheime Schlüssel konfiguriert ist**.
```python
app = Flask(__name__, template_folder='templates')
app.secret_key = '(:secret:)'
```
In diesem Fall ist es möglich, auf dieses Objekt zuzugreifen, indem man ein beliebiges Gadget verwendet, um auf globale Objekte von der Seite **Bypass Python Sandboxes** zuzugreifen.

Wenn die Schwachstelle in einer anderen Python-Datei liegt, benötigen Sie ein Gadget, um Dateien zu durchsuchen und zur Hauptdatei zu gelangen, um auf das globale Objekt `app.secret_key` zuzugreifen und den Flask-Schlüssel zu ändern. Dadurch können Sie [Berechtigungen eskalieren, indem Sie diesen Schlüssel kennen](../../network-services-pentesting/pentesting-web/flask.md#flask-unsign).

Ein Payload wie dieser [aus diesem Writeup](https://ctftime.org/writeup/36082):

{% code overflow="wrap" %}
```python
__init__.__globals__.__loader__.__init__.__globals__.sys.modules.__main__.app.secret_key
```
{% endcode %}

Verwenden Sie dieses Payload, um `app.secret_key` (der Name in Ihrer App kann unterschiedlich sein) zu ändern, um neue und privilegierte Flask-Cookies signieren zu können.

### Werkzeug - machine\_id und node uuid

[Mit diesem Payload aus diesem Writeup](https://vozec.fr/writeups/tweedle-dum-dee/) können Sie auf die **machine\_id** und den **uuid**-Knoten zugreifen, die die **Hauptgeheimnisse** sind, die Sie benötigen, um den [Werkzeug-Pin zu generieren](../../network-services-pentesting/pentesting-web/werkzeug.md), den Sie verwenden können, um auf die Python-Konsole in `/console` zuzugreifen, wenn der **Debug-Modus aktiviert ist:**
```python
{ua.__class__.__init__.__globals__[t].sys.modules[werkzeug.debug]._machine_id}
{ua.__class__.__init__.__globals__[t].sys.modules[werkzeug.debug].uuid._node}
```
{% hint style="warning" %}
Beachten Sie, dass Sie den **lokalen Pfad des Servers zur `app.py`** erhalten können, indem Sie auf der Webseite einen **Fehler** generieren, der Ihnen den Pfad **anzeigt**.
{% endhint %}

Wenn die Schwachstelle in einer anderen Python-Datei liegt, überprüfen Sie den vorherigen Flask-Trick, um auf die Objekte aus der Haupt-Python-Datei zuzugreifen.

<details>

<summary><strong>Lernen Sie AWS-Hacking von Null auf Held mit</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Andere Möglichkeiten, HackTricks zu unterstützen:

* Wenn Sie Ihr **Unternehmen in HackTricks bewerben möchten** oder **HackTricks als PDF herunterladen** möchten, überprüfen Sie die [**ABONNEMENTPLÄNE**](https://github.com/sponsors/carlospolop)!
* Holen Sie sich das [**offizielle PEASS & HackTricks-Merchandise**](https://peass.creator-spring.com)
* Entdecken Sie [**The PEASS Family**](https://opensea.io/collection/the-peass-family), unsere Sammlung exklusiver [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Treten Sie der** 💬 [**Discord-Gruppe**](https://discord.gg/hRep4RUj7f) oder der [**Telegram-Gruppe**](https://t.me/peass) bei oder **folgen** Sie uns auf **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Teilen Sie Ihre Hacking-Tricks, indem Sie Pull Requests an die** [**HackTricks**](https://github.com/carlospolop/hacktricks) und [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) GitHub-Repositories senden.

</details>
