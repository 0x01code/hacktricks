# macOS Systemerweiterungen

{% hint style="success" %}
Lernen & üben Sie AWS-Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Lernen & üben Sie GCP-Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Unterstützen Sie HackTricks</summary>

* Überprüfen Sie die [**Abonnementpläne**](https://github.com/sponsors/carlospolop)!
* **Treten Sie der** 💬 [**Discord-Gruppe**](https://discord.gg/hRep4RUj7f) oder der [**Telegramm-Gruppe**](https://t.me/peass) bei oder **folgen** Sie uns auf **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Teilen Sie Hacking-Tricks, indem Sie PRs an die** [**HackTricks**](https://github.com/carlospolop/hacktricks) und [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) GitHub-Repositories senden.

</details>
{% endhint %}

## Systemerweiterungen / Endpoint-Sicherheitsframework

Im Gegensatz zu Kernelerweiterungen **laufen Systemerweiterungen im Benutzerbereich** anstelle des Kernelbereichs, was das Risiko eines Systemabsturzes aufgrund einer Erweiterungsfehlfunktion verringert.

<figure><img src="../../../.gitbook/assets/image (606).png" alt="https://knight.sc/images/system-extension-internals-1.png"><figcaption></figcaption></figure>

Es gibt drei Arten von Systemerweiterungen: **DriverKit**-Erweiterungen, **Netzwerk**-Erweiterungen und **Endpoint-Sicherheits**-Erweiterungen.

### **DriverKit-Erweiterungen**

DriverKit ist ein Ersatz für Kernelerweiterungen, der **Hardwareunterstützung bereitstellt**. Es ermöglicht Gerätetreibern (wie USB-, Seriell-, NIC- und HID-Treibern), im Benutzerbereich anstelle des Kernelbereichs ausgeführt zu werden. Das DriverKit-Framework enthält **Benutzerbereichsversionen bestimmter I/O-Kit-Klassen**, und der Kernel leitet normale I/O-Kit-Ereignisse an den Benutzerbereich weiter, was eine sicherere Umgebung für diese Treiber bietet.

### **Netzwerk-Erweiterungen**

Netzwerk-Erweiterungen bieten die Möglichkeit, Netzwerkverhalten anzupassen. Es gibt verschiedene Arten von Netzwerk-Erweiterungen:

* **App-Proxy**: Dies wird verwendet, um einen VPN-Client zu erstellen, der ein flussorientiertes, benutzerdefiniertes VPN-Protokoll implementiert. Dies bedeutet, dass er Netzwerkverkehr basierend auf Verbindungen (oder Flüssen) und nicht auf einzelnen Paketen verarbeitet.
* **Paket-Tunnel**: Dies wird verwendet, um einen VPN-Client zu erstellen, der ein paketorientiertes, benutzerdefiniertes VPN-Protokoll implementiert. Dies bedeutet, dass er Netzwerkverkehr basierend auf einzelnen Paketen verarbeitet.
* **Datenfilter**: Dies wird verwendet, um Netzwerk "Flüsse" zu filtern. Es kann Netzwerkdaten auf Flussebene überwachen oder ändern.
* **Paketfilter**: Dies wird verwendet, um einzelne Netzwerkpakete zu filtern. Es kann Netzwerkdaten auf Paketebene überwachen oder ändern.
* **DNS-Proxy**: Dies wird verwendet, um einen benutzerdefinierten DNS-Anbieter zu erstellen. Es kann verwendet werden, um DNS-Anfragen und -Antworten zu überwachen oder zu ändern.

## Endpoint-Sicherheitsframework

Endpoint Security ist ein von Apple in macOS bereitgestelltes Framework, das eine Reihe von APIs für die Systemsicherheit bietet. Es ist für die Verwendung durch **Sicherheitsanbieter und Entwickler gedacht, um Produkte zu erstellen, die die Systemaktivität überwachen und steuern** und so bösartige Aktivitäten erkennen und schützen können.

Dieses Framework bietet eine **Sammlung von APIs zur Überwachung und Steuerung der Systemaktivität**, wie z. B. Prozessausführungen, Dateisystemereignisse, Netzwerk- und Kernelereignisse.

Der Kern dieses Frameworks ist im Kernel implementiert, als Kernelerweiterung (KEXT) unter **`/System/Library/Extensions/EndpointSecurity.kext`**. Diese KEXT besteht aus mehreren Schlüsselkomponenten:

* **EndpointSecurityDriver**: Dies fungiert als "Einstiegspunkt" für die Kernelerweiterung. Es ist der Hauptinteraktionspunkt zwischen dem Betriebssystem und dem Endpoint-Sicherheitsframework.
* **EndpointSecurityEventManager**: Diese Komponente ist dafür verantwortlich, Kernelhaken zu implementieren. Kernelhaken ermöglichen es dem Framework, Systemereignisse zu überwachen, indem Systemaufrufe abgefangen werden.
* **EndpointSecurityClientManager**: Dies verwaltet die Kommunikation mit Benutzerbereichsclients, um zu verfolgen, welche Clients verbunden sind und Ereignisbenachrichtigungen erhalten müssen.
* **EndpointSecurityMessageManager**: Dies sendet Nachrichten und Ereignisbenachrichtigungen an Benutzerbereichsclients.

Die Ereignisse, die das Endpoint-Sicherheitsframework überwachen kann, sind in folgende Kategorien unterteilt:

* Dateiereignisse
* Prozessereignisse
* Socketereignisse
* Kernelereignisse (wie Laden/Entladen einer Kernelerweiterung oder Öffnen eines I/O-Kit-Geräts)

### Architektur des Endpoint-Sicherheitsframeworks

<figure><img src="../../../.gitbook/assets/image (1068).png" alt="https://www.youtube.com/watch?v=jaVkpM1UqOs"><figcaption></figcaption></figure>

Die **Kommunikation im Benutzerbereich** mit dem Endpoint-Sicherheitsframework erfolgt über die Klasse IOUserClient. Es werden zwei verschiedene Unterklassen verwendet, abhängig vom Typ des Aufrufers:

* **EndpointSecurityDriverClient**: Dies erfordert die Berechtigung `com.apple.private.endpoint-security.manager`, die nur vom Systemprozess `endpointsecurityd` gehalten wird.
* **EndpointSecurityExternalClient**: Dies erfordert die Berechtigung `com.apple.developer.endpoint-security.client`. Dies wird in der Regel von Sicherheitssoftware von Drittanbietern verwendet, die mit dem Endpoint-Sicherheitsframework interagieren muss.

Die Endpoint-Sicherheitserweiterungen:**`libEndpointSecurity.dylib`** ist die C-Bibliothek, die Systemerweiterungen zur Kommunikation mit dem Kernel verwenden. Diese Bibliothek verwendet das I/O-Kit (`IOKit`), um mit der Endpoint-Sicherheits-KEXT zu kommunizieren.

**`endpointsecurityd`** ist ein wichtiger Systemdaemon, der bei der Verwaltung und dem Starten von Endpoint-Sicherheitssystemerweiterungen, insbesondere während des frühen Bootvorgangs, beteiligt ist. **Nur Systemerweiterungen**, die in ihrer `Info.plist`-Datei mit **`NSEndpointSecurityEarlyBoot`** markiert sind, erhalten diese frühe Bootbehandlung.

Ein weiterer Systemdaemon, **`sysextd`**, **validiert Systemerweiterungen** und verschiebt sie an die richtigen Systempositionen. Anschließend fordert er den relevanten Daemon auf, die Erweiterung zu laden. Das **`SystemExtensions.framework`** ist für das Aktivieren und Deaktivieren von Systemerweiterungen verantwortlich.

## Umgehung von ESF

ESF wird von Sicherheitstools verwendet, die versuchen, einen Red Teamer zu erkennen, daher klingt jede Information darüber, wie dies vermieden werden könnte, interessant.

### CVE-2021-30965

Die Sache ist, dass die Sicherheitsanwendung **Vollzugriff auf das Laufwerk benötigt**. Wenn ein Angreifer das entfernen könnte, könnte er verhindern, dass die Software ausgeführt wird:
```bash
tccutil reset All
```
Für **weitere Informationen** zu diesem Umgehungstrick und verwandten Themen, überprüfen Sie den Vortrag [#OBTS v5.0: "Die Achillesferse der Endpunktsicherheit" - Fitzl Csaba](https://www.youtube.com/watch?v=lQO7tvNCoTI)

Am Ende wurde dies behoben, indem die neue Berechtigung **`kTCCServiceEndpointSecurityClient`** der Sicherheits-App, die von **`tccd`** verwaltet wird, zugewiesen wurde, damit `tccutil` ihre Berechtigungen nicht löscht und sie somit ausgeführt werden kann.

## Referenzen

* [**OBTS v3.0: "Endpunktsicherheit & Unsicherheit" - Scott Knight**](https://www.youtube.com/watch?v=jaVkpM1UqOs)
* [**https://knight.sc/reverse%20engineering/2019/08/24/system-extension-internals.html**](https://knight.sc/reverse%20engineering/2019/08/24/system-extension-internals.html)

{% hint style="success" %}
Lernen Sie & üben Sie AWS-Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Lernen Sie & üben Sie GCP-Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Unterstützen Sie HackTricks</summary>

* Überprüfen Sie die [**Abonnementpläne**](https://github.com/sponsors/carlospolop)!
* **Treten Sie der** 💬 [**Discord-Gruppe**](https://discord.gg/hRep4RUj7f) oder der [**Telegram-Gruppe**](https://t.me/peass) bei oder **folgen** Sie uns auf **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Teilen Sie Hacking-Tricks, indem Sie PRs an die** [**HackTricks**](https://github.com/carlospolop/hacktricks) und [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) GitHub-Repositories einreichen.

</details>
{% endhint %}
