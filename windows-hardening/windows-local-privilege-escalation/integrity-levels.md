# Integritätsstufen

{% hint style="success" %}
Lerne & übe AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Lerne & übe GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Unterstütze HackTricks</summary>

* Überprüfe die [**Abonnementpläne**](https://github.com/sponsors/carlospolop)!
* **Tritt der** 💬 [**Discord-Gruppe**](https://discord.gg/hRep4RUj7f) oder der [**Telegram-Gruppe**](https://t.me/peass) bei oder **folge** uns auf **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Teile Hacking-Tricks, indem du PRs zu den** [**HackTricks**](https://github.com/carlospolop/hacktricks) und [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) GitHub-Repos einreichst.

</details>
{% endhint %}

### [WhiteIntel](https://whiteintel.io)

<figure><img src="../../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) ist eine von **Dark-Web** betriebenen Suchmaschine, die **kostenlose** Funktionen bietet, um zu überprüfen, ob ein Unternehmen oder dessen Kunden durch **Stealer-Malware** **kompromittiert** wurden.

Das Hauptziel von WhiteIntel ist es, Kontoübernahmen und Ransomware-Angriffe zu bekämpfen, die durch informationsstehlende Malware verursacht werden.

Du kannst ihre Website besuchen und ihre Engine **kostenlos** ausprobieren unter:

{% embed url="https://whiteintel.io" %}

***

## Integritätsstufen

In Windows Vista und späteren Versionen haben alle geschützten Elemente ein **Integritätsstufen**-Tag. Diese Konfiguration weist den meisten Dateien und Registrierungsschlüsseln eine "mittlere" Integritätsstufe zu, mit Ausnahme bestimmter Ordner und Dateien, auf die Internet Explorer 7 mit einer niedrigen Integritätsstufe schreiben kann. Das Standardverhalten ist, dass Prozesse, die von Standardbenutzern initiiert werden, eine mittlere Integritätsstufe haben, während Dienste typischerweise auf einer Systemintegritätsstufe arbeiten. Ein Hochintegritätslabel schützt das Wurzelverzeichnis.

Eine wichtige Regel ist, dass Objekte nicht von Prozessen mit einer niedrigeren Integritätsstufe als der des Objekts modifiziert werden können. Die Integritätsstufen sind:

* **Untrusted**: Diese Stufe ist für Prozesse mit anonymen Anmeldungen. %%%Beispiel: Chrome%%%
* **Low**: Hauptsächlich für Internetinteraktionen, insbesondere im geschützten Modus von Internet Explorer, der betroffene Dateien und Prozesse sowie bestimmte Ordner wie den **Temporären Internetordner** betrifft. Prozesse mit niedriger Integrität unterliegen erheblichen Einschränkungen, einschließlich des fehlenden Zugriffs auf die Registrierung und eingeschränktem Zugriff auf das Benutzerprofil.
* **Medium**: Die Standardstufe für die meisten Aktivitäten, die Standardbenutzern und Objekten ohne spezifische Integritätsstufen zugewiesen wird. Selbst Mitglieder der Administratorgruppe arbeiten standardmäßig auf dieser Stufe.
* **High**: Reserviert für Administratoren, die es ihnen ermöglichen, Objekte mit niedrigeren Integritätsstufen zu modifizieren, einschließlich solcher auf der hohen Stufe selbst.
* **System**: Die höchste Betriebsstufe für den Windows-Kernel und die Kernservices, die selbst für Administratoren unerreichbar ist und den Schutz wichtiger Systemfunktionen gewährleistet.
* **Installer**: Eine einzigartige Stufe, die über allen anderen steht und es Objekten auf dieser Stufe ermöglicht, jedes andere Objekt zu deinstallieren.

Du kannst die Integritätsstufe eines Prozesses mit **Process Explorer** von **Sysinternals** abrufen, indem du die **Eigenschaften** des Prozesses aufrufst und die Registerkarte "**Sicherheit**" ansiehst:

![](<../../.gitbook/assets/image (824).png>)

Du kannst auch deine **aktuelle Integritätsstufe** mit `whoami /groups` abrufen.

![](<../../.gitbook/assets/image (325).png>)

### Integritätsstufen im Dateisystem

Ein Objekt im Dateisystem kann eine **Mindestanforderung an die Integritätsstufe** benötigen, und wenn ein Prozess diese Integritätsstufe nicht hat, kann er nicht mit ihm interagieren.\
Zum Beispiel, lass uns **eine reguläre Datei von einer regulären Benutzerkonsole erstellen und die Berechtigungen überprüfen**:
```
echo asd >asd.txt
icacls asd.txt
asd.txt BUILTIN\Administrators:(I)(F)
DESKTOP-IDJHTKP\user:(I)(F)
NT AUTHORITY\SYSTEM:(I)(F)
NT AUTHORITY\INTERACTIVE:(I)(M,DC)
NT AUTHORITY\SERVICE:(I)(M,DC)
NT AUTHORITY\BATCH:(I)(M,DC)
```
Jetzt weisen wir der Datei ein minimales Integritätslevel von **Hoch** zu. Dies **muss von einer Konsole** ausgeführt werden, die als **Administrator** läuft, da eine **reguläre Konsole** im Integritätslevel Mittel läuft und **nicht** berechtigt ist, einem Objekt das hohe Integritätslevel zuzuweisen:
```
icacls asd.txt /setintegritylevel(oi)(ci) High
processed file: asd.txt
Successfully processed 1 files; Failed processing 0 files

C:\Users\Public>icacls asd.txt
asd.txt BUILTIN\Administrators:(I)(F)
DESKTOP-IDJHTKP\user:(I)(F)
NT AUTHORITY\SYSTEM:(I)(F)
NT AUTHORITY\INTERACTIVE:(I)(M,DC)
NT AUTHORITY\SERVICE:(I)(M,DC)
NT AUTHORITY\BATCH:(I)(M,DC)
Mandatory Label\High Mandatory Level:(NW)
```
Dies ist der Punkt, an dem es interessant wird. Sie können sehen, dass der Benutzer `DESKTOP-IDJHTKP\user` **VOLLSTÄNDIGE Berechtigungen** über die Datei hat (tatsächlich war dies der Benutzer, der die Datei erstellt hat), jedoch wird er aufgrund des implementierten minimalen Integritätsniveaus die Datei nicht mehr ändern können, es sei denn, er läuft innerhalb eines hohen Integritätsniveaus (beachten Sie, dass er sie lesen kann):
```
echo 1234 > asd.txt
Access is denied.

del asd.txt
C:\Users\Public\asd.txt
Access is denied.
```
{% hint style="info" %}
**Daher, wenn eine Datei ein minimales Integritätsniveau hat, müssen Sie mindestens auf diesem Integritätsniveau arbeiten, um sie zu ändern.**
{% endhint %}

### Integritätsniveaus in Binaries

Ich habe eine Kopie von `cmd.exe` in `C:\Windows\System32\cmd-low.exe` erstellt und ihr ein **Integritätsniveau von niedrig aus einer Administratorkonsole zugewiesen:**
```
icacls C:\Windows\System32\cmd-low.exe
C:\Windows\System32\cmd-low.exe NT AUTHORITY\SYSTEM:(I)(F)
BUILTIN\Administrators:(I)(F)
BUILTIN\Users:(I)(RX)
APPLICATION PACKAGE AUTHORITY\ALL APPLICATION PACKAGES:(I)(RX)
APPLICATION PACKAGE AUTHORITY\ALL RESTRICTED APP PACKAGES:(I)(RX)
Mandatory Label\Low Mandatory Level:(NW)
```
Jetzt, wenn ich `cmd-low.exe` ausführe, wird es **unter einem niedrigen Integritätslevel** anstelle eines mittleren ausgeführt:

![](<../../.gitbook/assets/image (313).png>)

Für Neugierige, wenn Sie einem Binärprogramm ein hohes Integritätslevel zuweisen (`icacls C:\Windows\System32\cmd-high.exe /setintegritylevel high`), wird es nicht automatisch mit hohem Integritätslevel ausgeführt (wenn Sie es von einem mittleren Integritätslevel aus aufrufen -- standardmäßig -- wird es unter einem mittleren Integritätslevel ausgeführt).

### Integritätslevel in Prozessen

Nicht alle Dateien und Ordner haben ein minimales Integritätslevel, **aber alle Prozesse laufen unter einem Integritätslevel**. Und ähnlich wie beim Dateisystem, **wenn ein Prozess in einen anderen Prozess schreiben möchte, muss er mindestens das gleiche Integritätslevel haben**. Das bedeutet, dass ein Prozess mit niedrigem Integritätslevel keinen Handle mit vollem Zugriff auf einen Prozess mit mittlerem Integritätslevel öffnen kann.

Aufgrund der in diesem und im vorherigen Abschnitt kommentierten Einschränkungen ist es aus sicherheitstechnischer Sicht immer **empfohlen, einen Prozess im niedrigsten möglichen Integritätslevel auszuführen**.

### [WhiteIntel](https://whiteintel.io)

<figure><img src="../../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) ist eine **Dark-Web**-unterstützte Suchmaschine, die **kostenlose** Funktionen bietet, um zu überprüfen, ob ein Unternehmen oder dessen Kunden von **Stealer-Malware** **kompromittiert** wurden.

Ihr Hauptziel von WhiteIntel ist es, Kontoübernahmen und Ransomware-Angriffe zu bekämpfen, die aus informationsstehlender Malware resultieren.

Sie können ihre Website besuchen und ihre Engine **kostenlos** ausprobieren unter:

{% embed url="https://whiteintel.io" %}

{% hint style="success" %}
Lernen & üben Sie AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Lernen & üben Sie GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Unterstützen Sie HackTricks</summary>

* Überprüfen Sie die [**Abonnementpläne**](https://github.com/sponsors/carlospolop)!
* **Treten Sie der** 💬 [**Discord-Gruppe**](https://discord.gg/hRep4RUj7f) oder der [**Telegram-Gruppe**](https://t.me/peass) bei oder **folgen** Sie uns auf **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Teilen Sie Hacking-Tricks, indem Sie PRs an die** [**HackTricks**](https://github.com/carlospolop/hacktricks) und [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) GitHub-Repos senden.

</details>
{% endhint %}
