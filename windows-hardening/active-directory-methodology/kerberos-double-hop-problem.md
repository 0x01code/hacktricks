# Kerberos Double-Hop-Problem

<details>

<summary><strong>Erlernen Sie das Hacken von AWS von Grund auf mit</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Arbeiten Sie in einem **Cybersicherheitsunternehmen**? Möchten Sie Ihr **Unternehmen in HackTricks beworben** sehen? Oder möchten Sie Zugriff auf die **neueste Version des PEASS oder HackTricks im PDF-Format** haben? Überprüfen Sie die [**ABONNEMENTPLÄNE**](https://github.com/sponsors/carlospolop)!
* Entdecken Sie [**The PEASS Family**](https://opensea.io/collection/the-peass-family), unsere Sammlung exklusiver [**NFTs**](https://opensea.io/collection/the-peass-family)
* Holen Sie sich das [**offizielle PEASS & HackTricks-Merchandise**](https://peass.creator-spring.com)
* **Treten Sie der** [**💬**](https://emojipedia.org/speech-balloon/) **Discord-Gruppe** bei (https://discord.gg/hRep4RUj7f) oder der **Telegram-Gruppe** oder **folgen** Sie mir auf **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Teilen Sie Ihre Hacking-Tricks, indem Sie PRs an das** [**HackTricks-Repo**](https://github.com/carlospolop/hacktricks) **und das** [**HackTricks-Cloud-Repo**](https://github.com/carlospolop/hacktricks-cloud) **senden**.

</details>

<figure><img src="/.gitbook/assets/WebSec_1500x400_10fps_21sn_lightoptimized_v2.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}

## Einführung

Das Kerberos "Double-Hop"-Problem tritt auf, wenn ein Angreifer versucht, die **Kerberos-Authentifizierung über zwei** **Hops** zu verwenden, beispielsweise mit **PowerShell**/**WinRM**.

Wenn eine **Authentifizierung** über **Kerberos** erfolgt, werden **Anmeldeinformationen** **nicht** im **Speicher zwischengespeichert**. Daher finden Sie bei der Ausführung von Mimikatz **keine Anmeldeinformationen** des Benutzers auf dem Computer, auch wenn er Prozesse ausführt.

Dies liegt daran, dass bei der Verbindung mit Kerberos die folgenden Schritte ausgeführt werden:

1. Benutzer1 gibt Anmeldeinformationen an und der **Domänencontroller** sendet ein Kerberos **TGT** an Benutzer1 zurück.
2. Benutzer1 verwendet das **TGT**, um ein **Service-Ticket** anzufordern, um eine Verbindung mit Server1 herzustellen.
3. Benutzer1 stellt eine Verbindung mit **Server1** her und gibt das **Service-Ticket** an.
4. **Server1** hat die **Anmeldeinformationen** von Benutzer1 oder das **TGT** von Benutzer1 **nicht zwischengespeichert**. Daher kann sich Benutzer1 von Server1 aus nicht bei einem zweiten Server anmelden.

### Unbeschränkte Delegierung

Wenn die **unbeschränkte Delegierung** auf dem PC aktiviert ist, tritt dies nicht auf, da der **Server** ein **TGT** jedes Benutzers erhält, der darauf zugreift. Darüber hinaus können bei Verwendung der unbeschränkten Delegierung wahrscheinlich der **Domänencontroller kompromittiert** werden.\
[**Weitere Informationen auf der Seite zur unbeschränkten Delegierung**](unconstrained-delegation.md).

### CredSSP

Ein weiterer Weg, um dieses Problem zu vermeiden, der [**bemerkenswert unsicher ist**](https://docs.microsoft.com/en-us/powershell/module/microsoft.wsman.management/enable-wsmancredssp?view=powershell-7), ist der **Credential Security Support Provider (CredSSP)**. Von Microsoft:

> Die CredSSP-Authentifizierung delegiert die Benutzeranmeldeinformationen vom lokalen Computer an einen Remote-Computer. Diese Praxis erhöht das Sicherheitsrisiko des Remotevorgangs. Wenn der Remote-Computer kompromittiert ist, können die Anmeldeinformationen, wenn sie an ihn übergeben werden, verwendet werden, um die Netzwerksitzung zu steuern.

Es wird dringend empfohlen, dass **CredSSP** auf Produktivsystemen, sensiblen Netzwerken und ähnlichen Umgebungen aufgrund von Sicherheitsbedenken deaktiviert wird. Um festzustellen, ob **CredSSP** aktiviert ist, kann der Befehl `Get-WSManCredSSP` ausgeführt werden. Dieser Befehl ermöglicht die **Überprüfung des CredSSP-Status** und kann sogar remote ausgeführt werden, sofern **WinRM** aktiviert ist.
```powershell
Invoke-Command -ComputerName bizintel -Credential ta\redsuit -ScriptBlock {
Get-WSManCredSSP
}
```
## Workarounds

### Invoke Command

Um das Double-Hop-Problem anzugehen, wird eine Methode mit einer verschachtelten `Invoke-Command` vorgestellt. Dies löst das Problem nicht direkt, bietet jedoch eine Lösung ohne spezielle Konfigurationen zu benötigen. Der Ansatz ermöglicht die Ausführung eines Befehls (`hostname`) auf einem sekundären Server über einen PowerShell-Befehl, der von einer initialen angreifenden Maschine ausgeführt wird oder über eine zuvor eingerichtete PS-Sitzung mit dem ersten Server. So wird es gemacht:
```powershell
$cred = Get-Credential ta\redsuit
Invoke-Command -ComputerName bizintel -Credential $cred -ScriptBlock {
Invoke-Command -ComputerName secdev -Credential $cred -ScriptBlock {hostname}
}
```
### Registrieren der PSSession-Konfiguration

Eine Lösung zur Umgehung des doppelten Hop-Problems besteht darin, `Register-PSSessionConfiguration` mit `Enter-PSSession` zu verwenden. Diese Methode erfordert einen anderen Ansatz als `evil-winrm` und ermöglicht eine Sitzung, die nicht unter der doppelten Hop-Beschränkung leidet.
```powershell
Register-PSSessionConfiguration -Name doublehopsess -RunAsCredential domain_name\username
Restart-Service WinRM
Enter-PSSession -ConfigurationName doublehopsess -ComputerName <pc_name> -Credential domain_name\username
klist
```
### Portweiterleitung

Für lokale Administratoren auf einem Zwischenziel ermöglicht die Portweiterleitung, Anfragen an einen endgültigen Server zu senden. Mit `netsh` kann eine Regel für die Portweiterleitung hinzugefügt werden, zusammen mit einer Windows-Firewallregel, um den weitergeleiteten Port zuzulassen.
```bash
netsh interface portproxy add v4tov4 listenport=5446 listenaddress=10.35.8.17 connectport=5985 connectaddress=10.35.8.23
netsh advfirewall firewall add rule name=fwd dir=in action=allow protocol=TCP localport=5446
```
#### winrs.exe

`winrs.exe` kann zum Weiterleiten von WinRM-Anfragen verwendet werden, möglicherweise als weniger erkennbare Option, wenn die Überwachung von PowerShell ein Anliegen ist. Der folgende Befehl zeigt dessen Verwendung:
```bash
winrs -r:http://bizintel:5446 -u:ta\redsuit -p:2600leet hostname
```
### OpenSSH

Die Installation von OpenSSH auf dem ersten Server ermöglicht eine Umgehung des Double-Hop-Problems, besonders nützlich für Jump-Box-Szenarien. Diese Methode erfordert die CLI-Installation und Konfiguration von OpenSSH für Windows. Wenn für die Passwortauthentifizierung konfiguriert, ermöglicht dies dem Zwischenserver, ein TGT im Namen des Benutzers zu erhalten.

#### OpenSSH Installationschritte

1. Laden Sie die neueste OpenSSH-Version herunter und verschieben Sie sie auf den Zielserver.
2. Entpacken Sie das Zip-File und führen Sie das `Install-sshd.ps1`-Skript aus.
3. Fügen Sie eine Firewall-Regel hinzu, um Port 22 zu öffnen, und überprüfen Sie, ob die SSH-Dienste ausgeführt werden.

Um `Verbindungsabbruch`-Fehler zu beheben, müssen Berechtigungen möglicherweise aktualisiert werden, um allen Lese- und Ausführungszugriff auf das OpenSSH-Verzeichnis zu ermöglichen.
```bash
icacls.exe "C:\Users\redsuit\Documents\ssh\OpenSSH-Win64" /grant Everyone:RX /T
```
## Referenzen

* [https://techcommunity.microsoft.com/t5/ask-the-directory-services-team/understanding-kerberos-double-hop/ba-p/395463?lightbox-message-images-395463=102145i720503211E78AC20](https://techcommunity.microsoft.com/t5/ask-the-directory-services-team/understanding-kerberos-double-hop/ba-p/395463?lightbox-message-images-395463=102145i720503211E78AC20)
* [https://posts.slayerlabs.com/double-hop/](https://posts.slayerlabs.com/double-hop/)
* [https://learn.microsoft.com/en-gb/archive/blogs/sergey\_babkins\_blog/another-solution-to-multi-hop-powershell-remoting](https://learn.microsoft.com/en-gb/archive/blogs/sergey\_babkins\_blog/another-solution-to-multi-hop-powershell-remoting)
* [https://4sysops.com/archives/solve-the-powershell-multi-hop-problem-without-using-credssp/](https://4sysops.com/archives/solve-the-powershell-multi-hop-problem-without-using-credssp/)

<figure><img src="/.gitbook/assets/WebSec_1500x400_10fps_21sn_lightoptimized_v2.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}

<details>

<summary><strong>Erlernen Sie AWS-Hacking von Null auf Held mit</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Arbeiten Sie in einem **Cybersicherheitsunternehmen**? Möchten Sie Ihr **Unternehmen in HackTricks beworben sehen**? Oder möchten Sie Zugriff auf die **neueste Version des PEASS erhalten oder HackTricks im PDF-Format herunterladen**? Überprüfen Sie die [**ABONNEMENTPLÄNE**](https://github.com/sponsors/carlospolop)!
* Entdecken Sie [**The PEASS Family**](https://opensea.io/collection/the-peass-family), unsere Sammlung exklusiver [**NFTs**](https://opensea.io/collection/the-peass-family)
* Holen Sie sich das [**offizielle PEASS & HackTricks-Merch**](https://peass.creator-spring.com)
* **Treten Sie der** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord-Gruppe**](https://discord.gg/hRep4RUj7f) oder der [**Telegram-Gruppe**](https://t.me/peass) bei oder **folgen** Sie mir auf **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Teilen Sie Ihre Hacking-Tricks, indem Sie PRs an das** [**HackTricks-Repository**](https://github.com/carlospolop/hacktricks) **und das** [**HackTricks-Cloud-Repository**](https://github.com/carlospolop/hacktricks-cloud) **senden**.

</details>
