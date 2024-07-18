# macOS Installers Missbrauch

{% hint style="success" %}
Lernen Sie und üben Sie AWS-Hacking: <img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Lernen Sie und üben Sie GCP-Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Unterstützen Sie HackTricks</summary>

* Überprüfen Sie die [**Abonnementpläne**](https://github.com/sponsors/carlospolop)!
* **Treten Sie der** 💬 [**Discord-Gruppe**](https://discord.gg/hRep4RUj7f) oder der [**Telegram-Gruppe**](https://t.me/peass) bei oder **folgen** Sie uns auf **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Teilen Sie Hacking-Tricks, indem Sie PRs an die** [**HackTricks**](https://github.com/carlospolop/hacktricks) und [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) Github-Repositorys senden.

</details>
{% endhint %}

## Pkg Grundlegende Informationen

Ein macOS **Installationspaket** (auch bekannt als `.pkg`-Datei) ist ein Dateiformat, das von macOS verwendet wird, um **Software zu verteilen**. Diese Dateien sind wie eine **Box, die alles enthält, was eine Software** benötigt, um korrekt installiert und ausgeführt zu werden.

Die Paketdatei selbst ist ein Archiv, das eine **Hierarchie von Dateien und Verzeichnissen enthält, die auf dem Zielcomputer installiert werden**. Es kann auch **Skripte** enthalten, um Aufgaben vor und nach der Installation auszuführen, wie z. B. das Einrichten von Konfigurationsdateien oder das Bereinigen alter Versionen der Software.

### Hierarchie

<figure><img src="../../../.gitbook/assets/Pasted Graphic.png" alt="https://www.youtube.com/watch?v=iASSG0_zobQ"><figcaption></figcaption></figure>

* **Distribution (xml)**: Anpassungen (Titel, Willkommensnachricht...) und Skript/Installationsprüfungen
* **PackageInfo (xml)**: Informationen, Installationsanforderungen, Installationsort, Pfade zu auszuführenden Skripten
* **Materialliste (bom)**: Liste der zu installierenden, zu aktualisierenden oder zu entfernenden Dateien mit Dateiberechtigungen
* **Payload (CPIO-Archiv gzip-komprimiert)**: Dateien, die im `install-location` aus PackageInfo installiert werden sollen
* **Skripte (CPIO-Archiv gzip-komprimiert)**: Vor- und Nachinstallations-Skripte und weitere Ressourcen, die in ein temporäres Verzeichnis extrahiert werden, um ausgeführt zu werden.

### Dekomprimieren
```bash
# Tool to directly get the files inside a package
pkgutil —expand "/path/to/package.pkg" "/path/to/out/dir"

# Get the files ina. more manual way
mkdir -p "/path/to/out/dir"
cd "/path/to/out/dir"
xar -xf "/path/to/package.pkg"

# Decompress also the CPIO gzip compressed ones
cat Scripts | gzip -dc | cpio -i
cpio -i < Scripts
```
Um den Inhalt des Installers ohne manuelles Entpacken zu visualisieren, können Sie auch das kostenlose Tool [**Suspicious Package**](https://mothersruin.com/software/SuspiciousPackage/) verwenden.

## DMG Grundinformationen

DMG-Dateien oder Apple Disk Images sind ein Dateiformat, das von Apples macOS für Festplattenabbilder verwendet wird. Eine DMG-Datei ist im Wesentlichen ein **einbindbares Festplattenabbild** (es enthält sein eigenes Dateisystem), das rohe Blockdaten enthält, die typischerweise komprimiert und manchmal verschlüsselt sind. Wenn Sie eine DMG-Datei öffnen, **bindet macOS sie ein, als ob es sich um eine physische Festplatte handeln würde**, was es Ihnen ermöglicht, auf deren Inhalt zuzugreifen.

{% hint style="danger" %}
Beachten Sie, dass **`.dmg`**-Installer **so viele Formate** unterstützen, dass in der Vergangenheit einige von ihnen mit Schwachstellen missbraucht wurden, um **Kernelcodeausführung** zu erlangen.
{% endhint %}

### Hierarchie

<figure><img src="../../../.gitbook/assets/image (225).png" alt=""><figcaption></figcaption></figure>

Die Hierarchie einer DMG-Datei kann je nach Inhalt unterschiedlich sein. Für Anwendungs-DMGs folgt sie jedoch in der Regel dieser Struktur:

* Top-Level: Dies ist die Wurzel des Festplattenabbilds. Es enthält oft die Anwendung und möglicherweise einen Link zum Ordner "Programme".
* Anwendung (.app): Dies ist die tatsächliche Anwendung. In macOS ist eine Anwendung in der Regel ein Paket, das viele einzelne Dateien und Ordner enthält, die die Anwendung ausmachen.
* Anwendungslink: Dies ist eine Verknüpfung zum Ordner "Programme" in macOS. Der Zweck davon ist es, es Ihnen leicht zu machen, die Anwendung zu installieren. Sie können die .app-Datei zu dieser Verknüpfung ziehen, um die App zu installieren.

## Privilege Escalation durch pkg-Missbrauch

### Ausführung aus öffentlichen Verzeichnissen

Wenn beispielsweise ein Vor- oder Nachinstallations-Skript aus **`/var/tmp/Installerutil`** ausgeführt wird und ein Angreifer dieses Skript kontrollieren könnte, könnte er Berechtigungen eskalieren, wann immer es ausgeführt wird. Oder ein ähnliches Beispiel:

<figure><img src="../../../.gitbook/assets/Pasted Graphic 5.png" alt="https://www.youtube.com/watch?v=iASSG0_zobQ"><figcaption><p><a href="https://www.youtube.com/watch?v=kCXhIYtODBg">https://www.youtube.com/watch?v=kCXhIYtODBg</a></p></figcaption></figure>

### AuthorizationExecuteWithPrivileges

Dies ist eine [öffentliche Funktion](https://developer.apple.com/documentation/security/1540038-authorizationexecutewithprivileg), die von mehreren Installationsprogrammen und Updatern aufgerufen wird, um **etwas als Root auszuführen**. Diese Funktion akzeptiert den **Pfad** der **Datei**, die als Parameter **ausgeführt** werden soll. Wenn ein Angreifer jedoch diese Datei **modifizieren** könnte, könnte er deren Ausführung mit Root-Rechten **missbrauchen**, um Berechtigungen zu **eskaliere**n.
```bash
# Breakpoint in the function to check wich file is loaded
(lldb) b AuthorizationExecuteWithPrivileges
# You could also check FS events to find this missconfig
```
### Ausführung durch Einhängen

Wenn ein Installer nach `/tmp/fixedname/bla/bla` schreibt, ist es möglich, **ein Mount über** `/tmp/fixedname` ohne Besitzer zu erstellen, sodass Sie **während der Installation jede Datei ändern** können, um den Installationsprozess zu missbrauchen.

Ein Beispiel hierfür ist **CVE-2021-26089**, bei dem es gelungen ist, ein periodisches Skript zu überschreiben, um die Ausführung als Root zu erhalten. Für weitere Informationen schauen Sie sich den Vortrag an: [**OBTS v4.0: "Mount(ain) of Bugs" - Csaba Fitzl**](https://www.youtube.com/watch?v=jSYPazD4VcE)

## pkg als Malware

### Leerer Payload

Es ist möglich, einfach eine **`.pkg`**-Datei mit **Vor- und Nachinstallations-Skripten** ohne Payload zu generieren.

### JS in Distribution-XML

Es ist möglich, **`<script>`**-Tags in der **Distribution-XML**-Datei des Pakets hinzuzufügen, und dieser Code wird ausgeführt, um Befehle mithilfe von **`system.run`** auszuführen:

<figure><img src="../../../.gitbook/assets/image (1043).png" alt=""><figcaption></figcaption></figure>

## Referenzen

* [**DEF CON 27 - Entpacken von Pkgs Ein Blick in Macos-Installationspakete und häufige Sicherheitslücken**](https://www.youtube.com/watch?v=iASSG0\_zobQ)
* [**OBTS v4.0: "Die wilde Welt der macOS-Installer" - Tony Lambert**](https://www.youtube.com/watch?v=Eow5uNHtmIg)
* [**DEF CON 27 - Entpacken von Pkgs Ein Blick in MacOS-Installationspakete**](https://www.youtube.com/watch?v=kCXhIYtODBg)
