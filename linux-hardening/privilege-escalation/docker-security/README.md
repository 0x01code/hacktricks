# Seguridad de Docker

![](<../../../.gitbook/assets/image (9) (1) (2).png>)

\
Utilice [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) para construir y **automatizar flujos de trabajo** con las herramientas de la comunidad **más avanzadas** del mundo.\
Obtenga acceso hoy:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* ¿Trabaja en una **empresa de ciberseguridad**? ¿Quiere ver su **empresa anunciada en HackTricks**? ¿O quiere tener acceso a la **última versión de PEASS o descargar HackTricks en PDF**? ¡Consulte los [**PLANES DE SUSCRIPCIÓN**](https://github.com/sponsors/carlospolop)!
* Descubra [**The PEASS Family**](https://opensea.io/collection/the-peass-family), nuestra colección exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* Obtenga el [**swag oficial de PEASS y HackTricks**](https://peass.creator-spring.com)
* **Únase al** [**💬**](https://emojipedia.org/speech-balloon/) [**grupo de Discord**](https://discord.gg/hRep4RUj7f) o al [**grupo de telegram**](https://t.me/peass) o **sígame** en **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Comparta sus trucos de hacking enviando PR al** [**repositorio de hacktricks**](https://github.com/carlospolop/hacktricks) **y al** [**repositorio de hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>

## **Seguridad básica del motor de Docker**

El motor de Docker realiza el trabajo pesado de ejecutar y administrar contenedores. El motor de Docker utiliza características del kernel de Linux como **Namespaces** y **Cgroups** para proporcionar un aislamiento básico entre contenedores. También utiliza características como **la eliminación de capacidades**, **Seccomp**, **SELinux/AppArmor para lograr un mejor aislamiento**.

Finalmente, se puede utilizar un **plugin de autenticación** para **limitar las acciones** que los usuarios pueden realizar.

![](<../../../.gitbook/assets/image (625) (1) (1).png>)

### **Acceso seguro al motor de Docker**

El cliente de Docker puede acceder al motor de Docker **localmente usando un socket Unix o de forma remota usando el mecanismo http**. Para usarlo de forma remota, es necesario utilizar https y **TLS** para garantizar la confidencialidad, integridad y autenticación.

De forma predeterminada, escucha en el socket Unix `unix:///var/`\
`run/docker.sock` y en las distribuciones de Ubuntu, las opciones de inicio de Docker se especifican en `/etc/default/docker`. Para permitir que la API y el cliente de Docker accedan al motor de Docker de forma remota, necesitamos **exponer el demonio de Docker usando un socket http**. Esto se puede hacer mediante:
```bash
DOCKER_OPTS="-D -H unix:///var/run/docker.sock -H
tcp://192.168.56.101:2376" -> add this to /etc/default/docker
Sudo service docker restart -> Restart Docker daemon
```
Exponer el demonio de Docker mediante http no es una buena práctica y es necesario asegurar la conexión mediante https. Hay dos opciones: la primera opción es que el **cliente verifique la identidad del servidor** y la segunda opción es que **tanto el cliente como el servidor se verifiquen mutuamente**. Los certificados establecen la identidad de un servidor. Para un ejemplo de ambas opciones, [**consulte esta página**](https://sreeninet.wordpress.com/2016/03/06/docker-security-part-3engine-access/).

### **Seguridad de la imagen del contenedor**

Las imágenes de los contenedores se almacenan en un repositorio privado o público. A continuación, se presentan las opciones que Docker proporciona para almacenar imágenes de contenedores:

* [Docker hub](https://hub.docker.com) - Este es un servicio de registro público proporcionado por Docker.
* [Docker registry](https://github.com/%20docker/distribution) - Este es un proyecto de código abierto que los usuarios pueden usar para alojar su propio registro.
* [Docker trusted registry](https://www.docker.com/docker-trusted-registry) - Esta es la implementación comercial de Docker del registro de Docker y proporciona autenticación de usuario basada en roles junto con la integración del servicio de directorio LDAP.

### Escaneo de imágenes

Los contenedores pueden tener **vulnerabilidades de seguridad** debido a la imagen base o al software instalado encima de la imagen base. Docker está trabajando en un proyecto llamado **Nautilus** que realiza un escaneo de seguridad de los contenedores y enumera las vulnerabilidades. Nautilus funciona comparando cada capa de la imagen del contenedor con el repositorio de vulnerabilidades para identificar agujeros de seguridad.

Para obtener más [**información, lea esto**](https://docs.docker.com/engine/scan/).

#### Cómo escanear imágenes <a href="#how-to-scan-images" id="how-to-scan-images"></a>

El comando `docker scan` le permite escanear imágenes de Docker existentes utilizando el nombre o ID de la imagen. Por ejemplo, ejecute el siguiente comando para escanear la imagen hello-world:
```bash
docker scan hello-world

Testing hello-world...

Organization:      docker-desktop-test
Package manager:   linux
Project name:      docker-image|hello-world
Docker image:      hello-world
Licenses:          enabled

✓ Tested 0 dependencies for known issues, no vulnerable paths found.

Note that we do not currently have vulnerability data for your image.
```
### Firma de Imágenes de Docker

Las imágenes de contenedor de Docker pueden almacenarse en un registro público o privado. Es necesario **firmar** las imágenes de **contenedor** para poder confirmar que las imágenes no han sido manipuladas. El **editor** de contenido se encarga de **firmar** la imagen del contenedor y de enviarla al registro.\
A continuación se presentan algunos detalles sobre la confianza del contenido de Docker:

* La confianza del contenido de Docker es una implementación del proyecto de código abierto [Notary](https://github.com/docker/notary). El proyecto de código abierto Notary se basa en el proyecto [The Update Framework (TUF)](https://theupdateframework.github.io).
* La confianza del contenido de Docker se habilita con `export DOCKER_CONTENT_TRUST=1`. A partir de la versión 1.10 de Docker, la confianza del contenido **no está habilitada de forma predeterminada**.
* **Cuando** la confianza del contenido está **habilitada**, solo podemos **descargar imágenes firmadas**. Cuando se empuja una imagen, es necesario ingresar la clave de etiquetado.
* Cuando el **editor** **envía** la imagen por **primera vez** usando docker push, es necesario ingresar una **contraseña** para la **clave raíz y la clave de etiquetado**. Las demás claves se generan automáticamente.
* Docker también ha agregado soporte para claves de hardware usando Yubikey y los detalles están disponibles [aquí](https://blog.docker.com/2015/11/docker-content-trust-yubikey/).

A continuación se presenta el **error** que obtenemos cuando **la confianza del contenido está habilitada y la imagen no está firmada**.
```shell-session
$ docker pull smakam/mybusybox
Using default tag: latest
No trust data for latest
```
El siguiente resultado muestra la imagen del contenedor siendo enviada a Docker Hub con la firma habilitada. Como no es la primera vez, se solicita al usuario que ingrese solo la frase de contraseña para la clave del repositorio.
```shell-session
$ docker push smakam/mybusybox:v2
The push refers to a repository [docker.io/smakam/mybusybox]
a7022f99b0cc: Layer already exists 
5f70bf18a086: Layer already exists 
9508eff2c687: Layer already exists 
v2: digest: sha256:8509fa814029e1c1baf7696b36f0b273492b87f59554a33589e1bd6283557fc9 size: 2205
Signing and pushing trust metadata
Enter passphrase for repository key with ID 001986b (docker.io/smakam/mybusybox): 
```
Es necesario almacenar la clave raíz, la clave del repositorio y la frase de contraseña en un lugar seguro. El siguiente comando se puede utilizar para hacer una copia de seguridad de las claves privadas:
```bash
tar -zcvf private_keys_backup.tar.gz ~/.docker/trust/private
```
Cuando cambié el host de Docker, tuve que mover las claves raíz y las claves del repositorio para operar desde el nuevo host.

![](<../../../.gitbook/assets/image (9) (1) (2).png>)

\
Utilice [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) para construir y **automatizar flujos de trabajo** fácilmente con las herramientas comunitarias más avanzadas del mundo.\
Obtenga acceso hoy:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## Características de seguridad de los contenedores

<details>

<summary>Resumen de las características de seguridad de los contenedores</summary>

#### Espacios de nombres

Los espacios de nombres son útiles para aislar un proyecto de los demás, aislando las comunicaciones de los procesos, la red, los montajes... Es útil para aislar el proceso de Docker de otros procesos (e incluso de la carpeta /proc) para que no pueda escapar abusando de otros procesos.

Podría ser posible "escapar" o más exactamente **crear nuevos espacios de nombres** utilizando el binario **`unshare`** (que utiliza la llamada al sistema **`unshare`**). Docker por defecto lo previene, pero Kubernetes no (en el momento de escribir esto).\
De todas formas, esto es útil para crear nuevos espacios de nombres, pero **no para volver a los espacios de nombres predeterminados del host** (a menos que tenga acceso a algún `/proc` dentro de los espacios de nombres del host, donde podría usar **`nsenter`** para entrar en los espacios de nombres del host).

#### CGroups

Esto permite limitar los recursos y no afecta a la seguridad del aislamiento del proceso (excepto para el `release_agent` que podría ser utilizado para escapar).

#### Capabilities Drop

Encuentro que esta es una de las características **más importantes** en cuanto a la seguridad del aislamiento del proceso. Esto se debe a que sin las capacidades, incluso si el proceso se está ejecutando como root, **no podrá realizar algunas acciones privilegiadas** (porque la llamada al sistema **`syscall`** devolverá un error de permiso porque el proceso no tiene las capacidades necesarias).

Estas son las **capacidades restantes** después de que el proceso abandona las demás:

{% code overflow="wrap" %}
```
Current: cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap=ep
```
{% endcode %}

#### Seccomp

Está habilitado por defecto en Docker. Ayuda a **limitar aún más las llamadas al sistema** que el proceso puede realizar.\
El **perfil Seccomp predeterminado de Docker** se puede encontrar en [https://github.com/moby/moby/blob/master/profiles/seccomp/default.json](https://github.com/moby/moby/blob/master/profiles/seccomp/default.json)

#### AppArmor

Docker tiene una plantilla que se puede activar: [https://github.com/moby/moby/tree/master/profiles/apparmor](https://github.com/moby/moby/tree/master/profiles/apparmor)

Esto permitirá reducir las capacidades, llamadas al sistema, acceso a archivos y carpetas...

</details>

### Namespaces

**Namespaces** son una característica del kernel de Linux que **particiona los recursos del kernel** de tal manera que un conjunto de **procesos ve** un conjunto de **recursos** mientras que **otro** conjunto de **procesos** ve un **conjunto diferente** de recursos. La característica funciona teniendo el mismo espacio de nombres para un conjunto de recursos y procesos, pero esos espacios de nombres se refieren a recursos distintos. Los recursos pueden existir en múltiples espacios.

Docker hace uso de los siguientes espacios de nombres del kernel de Linux para lograr el aislamiento del contenedor:

* espacio de nombres pid
* espacio de nombres de montaje
* espacio de nombres de red
* espacio de nombres ipc
* espacio de nombres UTS

Para **más información sobre los espacios de nombres**, consulte la siguiente página:

{% content-ref url="namespaces/" %}
[namespaces](namespaces/)
{% endcontent-ref %}

### cgroups

La característica del kernel de Linux **cgroups** proporciona la capacidad de **restringir recursos como la CPU, la memoria, la E / S, el ancho de banda de la red entre** un conjunto de procesos. Docker permite crear contenedores utilizando la característica cgroup que permite el control de recursos para el contenedor específico.\
A continuación se muestra un contenedor creado con una memoria de espacio de usuario limitada a 500m, una memoria de kernel limitada a 50m, una cuota de CPU de 512, un peso de blkioweight de 400. La cuota de CPU es una proporción que controla el uso de CPU del contenedor. Tiene un valor predeterminado de 1024 y un rango entre 0 y 1024. Si tres contenedores tienen la misma cuota de CPU de 1024, cada contenedor puede tomar hasta el 33% de la CPU en caso de conflicto de recursos de CPU. blkio-weight es una proporción que controla la E / S del contenedor. Tiene un valor predeterminado de 500 y un rango entre 10 y 1000.
```
docker run -it -m 500M --kernel-memory 50M --cpu-shares 512 --blkio-weight 400 --name ubuntu1 ubuntu bash
```
Para obtener el cgroup de un contenedor puedes hacer lo siguiente:
```bash
docker run -dt --rm denial sleep 1234 #Run a large sleep inside a Debian container
ps -ef | grep 1234 #Get info about the sleep process
ls -l /proc/<PID>/ns #Get the Group and the namespaces (some may be uniq to the hosts and some may be shred with it)
```
Para obtener más información, consulte:

{% content-ref url="cgroups.md" %}
[cgroups.md](cgroups.md)
{% endcontent-ref %}

### Capacidades

Las capacidades permiten un **control más fino de las capacidades que se pueden permitir** para el usuario root. Docker utiliza la función de capacidad del kernel de Linux para **limitar las operaciones que se pueden realizar dentro de un contenedor** independientemente del tipo de usuario.

Cuando se ejecuta un contenedor de Docker, el **proceso elimina las capacidades sensibles que el proceso podría usar para escapar del aislamiento**. Esto intenta asegurar que el proceso no pueda realizar acciones sensibles y escapar:

{% content-ref url="../linux-capabilities.md" %}
[linux-capabilities.md](../linux-capabilities.md)
{% endcontent-ref %}

### Seccomp en Docker

Esta es una función de seguridad que permite a Docker **limitar las llamadas al sistema** que se pueden utilizar dentro del contenedor:

{% content-ref url="seccomp.md" %}
[seccomp.md](seccomp.md)
{% endcontent-ref %}

### AppArmor en Docker

**AppArmor** es una mejora del kernel para confinar los **contenedores** a un **conjunto limitado de recursos** con **perfiles por programa**:

{% content-ref url="apparmor.md" %}
[apparmor.md](apparmor.md)
{% endcontent-ref %}

### SELinux en Docker

[SELinux](https://www.redhat.com/en/blog/latest-container-exploit-runc-can-be-blocked-selinux) es un **sistema de etiquetado**. Cada **proceso** y cada **objeto del sistema de archivos** tiene una **etiqueta**. Las políticas de SELinux definen reglas sobre lo que una **etiqueta de proceso puede hacer con todas las demás etiquetas** en el sistema.

Los motores de contenedores lanzan **procesos de contenedor con una sola etiqueta SELinux confinada**, generalmente `container_t`, y luego establecen el contenedor dentro del contenedor para que tenga la etiqueta `container_file_t`. Las reglas de la política de SELinux básicamente dicen que los **procesos `container_t` solo pueden leer/escribir/ejecutar archivos etiquetados como `container_file_t`**.

{% content-ref url="../selinux.md" %}
[selinux.md](../selinux.md)
{% endcontent-ref %}

### AuthZ y AuthN

Un complemento de autorización **aprueba** o **deniega** **solicitudes** al **daemon** de Docker en función tanto del contexto de **autenticación** actual como del contexto de **comando**. El contexto de **autenticación** contiene todos los detalles del **usuario** y el **método de autenticación**. El contexto de **comando** contiene todos los datos de **solicitud** relevantes.

{% content-ref url="authz-and-authn-docker-access-authorization-plugin.md" %}
[authz-and-authn-docker-access-authorization-plugin.md](authz-and-authn-docker-access-authorization-plugin.md)
{% endcontent-ref %}

## Interesantes banderas de Docker

### Bandera --privileged

En la siguiente página, puede aprender **qué implica la bandera `--privileged`**:

{% content-ref url="docker-privileged.md" %}
[docker-privileged.md](docker-privileged.md)
{% endcontent-ref %}

### --security-opt

#### no-new-privileges

Si está ejecutando un contenedor donde un atacante logra obtener acceso como usuario de baja privilegiado. Si tiene un **binario suid mal configurado**, el atacante puede abusar de él y **escalar privilegios dentro** del contenedor. Lo que puede permitirle escapar de él.

Ejecutar el contenedor con la opción **`no-new-privileges`** habilitada **evitará este tipo de escalada de privilegios**.
```
docker run -it --security-opt=no-new-privileges:true nonewpriv
```
#### Otros
```bash
#You can manually add/drop capabilities with
--cap-add
--cap-drop

# You can manually disable seccomp in docker with
--security-opt seccomp=unconfined

# You can manually disable seccomp in docker with
--security-opt apparmor=unconfined

# You can manually disable selinux in docker with
--security-opt label:disable
```
Para más opciones de **`--security-opt`**, consulta: [https://docs.docker.com/engine/reference/run/#security-configuration](https://docs.docker.com/engine/reference/run/#security-configuration)

![](<../../../.gitbook/assets/image (9) (1) (2).png>)

\
Utiliza [**Trickest**](https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks) para construir y **automatizar flujos de trabajo** con las herramientas comunitarias más avanzadas del mundo.\
Obtén acceso hoy mismo:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## Otras consideraciones de seguridad

### Gestión de secretos

En primer lugar, **¡no los incluyas dentro de tu imagen!**

Además, **no utilices variables de entorno** para tu información confidencial. Cualquier persona que pueda ejecutar `docker inspect` o `exec` en el contenedor puede encontrar tu secreto.

Los volúmenes de Docker son mejores. Son la forma recomendada de acceder a tu información confidencial en la documentación de Docker. Puedes **utilizar un volumen como sistema de archivos temporal en memoria**. Los volúmenes eliminan el riesgo de `docker inspect` y de registro. Sin embargo, **los usuarios root aún podrían ver el secreto, al igual que cualquiera que pueda `exec` en el contenedor**.

Incluso **mejor que los volúmenes, utiliza los secretos de Docker**.

Si solo necesitas el **secreto en tu imagen**, puedes utilizar **BuildKit**. BuildKit reduce significativamente el tiempo de construcción y tiene otras características interesantes, incluyendo el soporte de secretos en tiempo de construcción.

Hay tres formas de especificar el backend de BuildKit para que puedas utilizar sus características ahora:

1. Establécelo como una variable de entorno con `export DOCKER_BUILDKIT=1`.
2. Inicia tu comando `build` o `run` con `DOCKER_BUILDKIT=1`.
3. Habilita BuildKit de forma predeterminada. Establece la configuración en /_etc/docker/daemon.json_ en _true_ con: `{ "features": { "buildkit": true } }`. Luego reinicia Docker.
4. Luego puedes utilizar secretos en tiempo de construcción con la bandera `--secret` de la siguiente manera:
```bash
docker build --secret my_key=my_value ,src=path/to/my_secret_file .
```
Donde su archivo especifica sus secretos como pares clave-valor.

Estos secretos se excluyen de la caché de compilación de la imagen y de la imagen final.

Si necesita su **secreto en su contenedor en ejecución**, y no solo al construir su imagen, use **Docker Compose o Kubernetes**.

Con Docker Compose, agregue el par clave-valor de secretos a un servicio y especifique el archivo secreto. Un agradecimiento especial a la respuesta de [Stack Exchange](https://serverfault.com/a/936262/535325) por el consejo de secretos de Docker Compose del que se adapta el siguiente ejemplo.

Ejemplo de docker-compose.yml con secretos:
```yaml
version: "3.7"

services:

  my_service:
    image: centos:7
    entrypoint: "cat /run/secrets/my_secret"
    secrets:
      - my_secret

secrets:
  my_secret:
    file: ./my_secret_file.txt
```
Si estás utilizando Docker Compose, inicia el servicio como de costumbre con `docker-compose up --build my_service`.

Si estás utilizando [Kubernetes](https://kubernetes.io/docs/concepts/configuration/secret/), este tiene soporte para secretos. [Helm-Secrets](https://github.com/futuresimple/helm-secrets) puede ayudar a hacer más fácil la gestión de secretos en K8s. Además, K8s tiene Controles de Acceso Basados en Roles (RBAC) - al igual que Docker Enterprise. RBAC hace que la gestión de los secretos sea más manejable y segura para los equipos.

### gVisor

**gVisor** es un kernel de aplicación, escrito en Go, que implementa una porción sustancial de la superficie del sistema Linux. Incluye un tiempo de ejecución de la [Iniciativa de Contenedor Abierto (OCI)](https://www.opencontainers.org) llamado `runsc` que proporciona un **límite de aislamiento entre la aplicación y el kernel del host**. El tiempo de ejecución `runsc` se integra con Docker y Kubernetes, lo que hace que sea sencillo ejecutar contenedores en sandbox.

{% embed url="https://github.com/google/gvisor" %}

### Kata Containers

**Kata Containers** es una comunidad de código abierto que trabaja para construir un tiempo de ejecución de contenedor seguro con máquinas virtuales ligeras que se sienten y funcionan como contenedores, pero proporcionan una **mayor aislamiento de carga de trabajo utilizando la tecnología de virtualización de hardware como una segunda capa de defensa**.

{% embed url="https://katacontainers.io/" %}

### Consejos resumidos

* **No utilices la bandera `--privileged` ni montes un** [**socket de Docker dentro del contenedor**](https://raesene.github.io/blog/2016/03/06/The-Dangers-Of-Docker.sock/)**.** El socket de Docker permite la creación de contenedores, por lo que es una forma fácil de tomar el control total del host, por ejemplo, ejecutando otro contenedor con la bandera `--privileged`.
* No **ejecutes como root dentro del contenedor. Utiliza un** [**usuario diferente**](https://docs.docker.com/develop/develop-images/dockerfile\_best-practices/#user) **y** [**espacios de nombres de usuario**](https://docs.docker.com/engine/security/userns-remap/)**.** El root en el contenedor es el mismo que en el host a menos que se remapee con espacios de nombres de usuario. Sólo está ligeramente restringido por, principalmente, los espacios de nombres de Linux, las capacidades y los cgroups.
* [**Elimina todas las capacidades**](https://docs.docker.com/engine/reference/run/#runtime-privilege-and-linux-capabilities) **(`--cap-drop=all`) y habilita sólo las que se requieren** (`--cap-add=...`). Muchas cargas de trabajo no necesitan capacidades y añadirlos aumenta el alcance de un posible ataque.
* [**Utiliza la opción de seguridad "no-new-privileges"**](https://raesene.github.io/blog/2019/06/01/docker-capabilities-and-no-new-privs/) para evitar que los procesos obtengan más privilegios, por ejemplo, a través de binarios suid.
* [**Limita los recursos disponibles para el contenedor**](https://docs.docker.com/engine/reference/run/#runtime-constraints-on-resources)**.** Los límites de recursos pueden proteger la máquina de ataques de denegación de servicio.
* **Ajusta los perfiles de** [**seccomp**](https://docs.docker.com/engine/security/seccomp/)**,** [**AppArmor**](https://docs.docker.com/engine/security/apparmor/) **(o SELinux)** para restringir las acciones y las llamadas al sistema disponibles para el contenedor al mínimo requerido.
* **Utiliza** [**imágenes oficiales de Docker**](https://docs.docker.com/docker-hub/official\_images/) **y exige firmas** o construye las tuyas propias basadas en ellas. No heredes ni utilices imágenes [con puertas traseras](https://arstechnica.com/information-technology/2018/06/backdoored-images-downloaded-5-million-times-finally-removed-from-docker-hub/). También guarda las claves raíz, la frase de contraseña en un lugar seguro. Docker tiene planes para gestionar las claves con UCP.
* **Reconstruye regularmente** tus imágenes para **aplicar parches de seguridad al host e imágenes.**
* Gestiona tus **secretos sabiamente** para que sea difícil para el atacante acceder a ellos.
* Si **expones el demonio de Docker, utiliza HTTPS** con autenticación de cliente y servidor.
* En tu Dockerfile, **prefiere COPY en lugar de ADD**. ADD extrae automáticamente archivos comprimidos y puede copiar archivos desde URLs. COPY no tiene estas capacidades. Siempre que sea posible, evita el uso de ADD para no ser susceptible a ataques a través de URLs remotas y archivos Zip.
* Ten **contenedores separados para cada microservicio**
* **No pongas ssh** dentro del contenedor, se puede utilizar "docker exec" para ssh al contenedor.
* Ten **imágenes de contenedor más pequeñas**

## Escape de Docker / Escalada de privilegios

Si estás **dentro de un contenedor de Docker** o tienes acceso a un usuario en el **grupo de Docker**, podrías intentar **escapar y escalar privilegios**:

{% content-ref url="docker-breakout-privilege-escalation/" %}
[docker-breakout-privilege-escalation](docker-breakout-privilege-escalation/)
{% endcontent-ref %}

## Bypass del plugin de autenticación de Docker

Si tienes acceso al socket de Docker o tienes acceso a un usuario en el **grupo de Docker pero tus acciones están siendo limitadas por un plugin de autenticación de Docker**, comprueba si puedes **bypass**:

{% content-ref url="authz-and-authn-docker-access-authorization-plugin.md" %}
[authz-and-authn-docker-access-authorization-plugin.md](authz-and-authn-docker-access-authorization-plugin.md)
{% endcontent-ref %}

## Reforzamiento de Docker

* La herramienta [**docker-bench-security**](https://github.com/docker/docker-bench-security) es un script que comprueba docenas de prácticas comunes de seguridad en la implementación de contenedores de Docker en producción. Las pruebas son todas automatizadas y se basan en el [CIS Docker Benchmark v1.3.1](https://www.cisecurity.org/benchmark/docker/).\
  Necesitas ejecutar la herramienta desde el host que ejecuta Docker o desde un contenedor con suficientes privilegios. Encuentra **cómo ejecutarla en el README:** [**https://github.com/docker/docker-bench-security**](https://github.com/docker/docker-bench-security).

## Referencias

* [https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/](https://blog.trailofbits
