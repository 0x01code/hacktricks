# macOS-Bundles

<details>

<summary><strong>Lernen Sie AWS-Hacking von Null auf Held mit</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Andere Möglichkeiten, HackTricks zu unterstützen:

* Wenn Sie Ihr **Unternehmen in HackTricks bewerben möchten** oder **HackTricks als PDF herunterladen möchten**, überprüfen Sie die [**ABONNEMENTPLÄNE**](https://github.com/sponsors/carlospolop)!
* Holen Sie sich das [**offizielle PEASS & HackTricks-Merchandise**](https://peass.creator-spring.com)
* Entdecken Sie [**The PEASS Family**](https://opensea.io/collection/the-peass-family), unsere Sammlung exklusiver [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Treten Sie der** 💬 [**Discord-Gruppe**](https://discord.gg/hRep4RUj7f) oder der [**Telegramm-Gruppe**](https://t.me/peass) bei oder **folgen** Sie uns auf **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Teilen Sie Ihre Hacking-Tricks, indem Sie PRs an die** [**HackTricks**](https://github.com/carlospolop/hacktricks) und [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) GitHub-Repositories senden.

</details>

## Grundlegende Informationen

Bundles in macOS dienen als Container für verschiedene Ressourcen, einschließlich Anwendungen, Bibliotheken und anderen erforderlichen Dateien, sodass sie in Finder als einzelne Objekte angezeigt werden, wie z. B. die vertrauten `*.app`-Dateien. Das am häufigsten verwendete Bundle ist das `.app`-Bundle, aber auch andere Typen wie `.framework`, `.systemextension` und `.kext` sind weit verbreitet.

### Wesentliche Komponenten eines Bundles

Innerhalb eines Bundles, insbesondere im Verzeichnis `<Anwendung>.app/Contents/`, befinden sich verschiedene wichtige Ressourcen:

- **_CodeSignature**: In diesem Verzeichnis werden Code-Signaturdetails gespeichert, die für die Überprüfung der Integrität der Anwendung wichtig sind. Sie können die Code-Signaturinformationen mit Befehlen wie folgt überprüfen:
%%%bash
openssl dgst -binary -sha1 /Applications/Safari.app/Contents/Resources/Assets.car | openssl base64
%%%
- **MacOS**: Enthält die ausführbare Binärdatei der Anwendung, die bei der Benutzerinteraktion ausgeführt wird.
- **Resources**: Ein Repository für die Benutzeroberflächenkomponenten der Anwendung, einschließlich Bilder, Dokumente und Schnittstellenbeschreibungen (nib/xib-Dateien).
- **Info.plist**: Dient als Hauptkonfigurationsdatei der Anwendung und ist für das System entscheidend, um die Anwendung entsprechend zu erkennen und mit ihr zu interagieren.

#### Wichtige Schlüssel in Info.plist

Die Datei `Info.plist` ist ein Eckpfeiler für die Anwendungskonfiguration und enthält Schlüssel wie:

- **CFBundleExecutable**: Gibt den Namen der Hauptausführungsdatei an, die sich im Verzeichnis `Contents/MacOS` befindet.
- **CFBundleIdentifier**: Bietet eine globale Kennung für die Anwendung, die von macOS umfangreich für die Anwendungsverwaltung verwendet wird.
- **LSMinimumSystemVersion**: Gibt die Mindestversion von macOS an, die für die Ausführung der Anwendung erforderlich ist.

### Erkunden von Bundles

Um den Inhalt eines Bundles wie `Safari.app` zu erkunden, kann der folgende Befehl verwendet werden:
%%%bash
ls -lR /Applications/Safari.app/Contents
%%%

Diese Erkundung zeigt Verzeichnisse wie `_CodeSignature`, `MacOS`, `Resources` und Dateien wie `Info.plist`, von denen jeder eine einzigartige Funktion von der Sicherung der Anwendung bis zur Definition ihrer Benutzeroberfläche und Betriebsparameter erfüllt.

#### Zusätzliche Bundle-Verzeichnisse

Neben den gängigen Verzeichnissen können Bundles auch Folgendes enthalten:

- **Frameworks**: Enthält gebündelte Frameworks, die von der Anwendung verwendet werden.
- **PlugIns**: Ein Verzeichnis für Plug-Ins und Erweiterungen, die die Fähigkeiten der Anwendung erweitern.
- **XPCServices**: Enthält von der Anwendung für die Kommunikation außerhalb des Prozesses verwendete XPC-Dienste.

Diese Struktur gewährleistet, dass alle erforderlichen Komponenten im Bundle eingeschlossen sind und eine modulare und sichere Anwendungsumgebung ermöglichen.

Für weitere detaillierte Informationen zu `Info.plist`-Schlüsseln und deren Bedeutung bietet die Apple-Entwicklerdokumentation umfangreiche Ressourcen: [Apple Info.plist Key Reference](https://developer.apple.com/library/archive/documentation/General/Reference/InfoPlistKeyReference/Introduction/Introduction.html).

<details>

<summary><strong>Lernen Sie AWS-Hacking von Null auf Held mit</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Andere Möglichkeiten, HackTricks zu unterstützen:

* Wenn Sie Ihr **Unternehmen in HackTricks bewerben möchten** oder **HackTricks als PDF herunterladen möchten**, überprüfen Sie die [**ABONNEMENTPLÄNE**](https://github.com/sponsors/carlospolop)!
* Holen Sie sich das [**offizielle PEASS & HackTricks-Merchandise**](https://peass.creator-spring.com)
* Entdecken Sie [**The PEASS Family**](https://opensea.io/collection/the-peass-family), unsere Sammlung exklusiver [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Treten Sie der** 💬 [**Discord-Gruppe**](https://discord.gg/hRep4RUj7f) oder der [**Telegramm-Gruppe**](https://t.me/peass) bei oder **folgen** Sie uns auf **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Teilen Sie Ihre Hacking-Tricks, indem Sie PRs an die** [**HackTricks**](https://github.com/carlospolop/hacktricks) und [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) GitHub-Repositories senden.

</details>
