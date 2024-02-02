# Espacio de nombres de montaje

<details>

<summary><strong>Aprende hacking en AWS de cero a héroe con</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Otras formas de apoyar a HackTricks:

* Si quieres ver a tu **empresa anunciada en HackTricks** o **descargar HackTricks en PDF**, consulta los [**PLANES DE SUSCRIPCIÓN**](https://github.com/sponsors/carlospolop)!
* Consigue el [**merchandising oficial de PEASS & HackTricks**](https://peass.creator-spring.com)
* Descubre [**La Familia PEASS**](https://opensea.io/collection/the-peass-family), nuestra colección de [**NFTs**](https://opensea.io/collection/the-peass-family) exclusivos
* **Únete al** 💬 [**grupo de Discord**](https://discord.gg/hRep4RUj7f) o al [**grupo de telegram**](https://t.me/peass) o **sígueme** en **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **Comparte tus trucos de hacking enviando PRs a los repositorios de github de** [**HackTricks**](https://github.com/carlospolop/hacktricks) y [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>

## Información Básica

Un espacio de nombres de montaje es una característica del kernel de Linux que proporciona aislamiento de los puntos de montaje del sistema de archivos vistos por un grupo de procesos. Cada espacio de nombres de montaje tiene su propio conjunto de puntos de montaje del sistema de archivos, y **los cambios en los puntos de montaje en un espacio de nombres no afectan a otros espacios de nombres**. Esto significa que los procesos que se ejecutan en diferentes espacios de nombres de montaje pueden tener diferentes vistas de la jerarquía del sistema de archivos.

Los espacios de nombres de montaje son particularmente útiles en la contenerización, donde cada contenedor debe tener su propio sistema de archivos y configuración, aislado de otros contenedores y del sistema anfitrión.

### Cómo funciona:

1. Cuando se crea un nuevo espacio de nombres de montaje, se inicializa con una **copia de los puntos de montaje de su espacio de nombres padre**. Esto significa que, en el momento de la creación, el nuevo espacio de nombres comparte la misma vista del sistema de archivos que su padre. Sin embargo, cualquier cambio posterior en los puntos de montaje dentro del espacio de nombres no afectará al padre ni a otros espacios de nombres.
2. Cuando un proceso modifica un punto de montaje dentro de su espacio de nombres, como montar o desmontar un sistema de archivos, el **cambio es local a ese espacio de nombres** y no afecta a otros espacios de nombres. Esto permite que cada espacio de nombres tenga su propia jerarquía de sistema de archivos independiente.
3. Los procesos pueden moverse entre espacios de nombres utilizando la llamada al sistema `setns()`, o crear nuevos espacios de nombres utilizando las llamadas al sistema `unshare()` o `clone()` con la bandera `CLONE_NEWNS`. Cuando un proceso se mueve a un nuevo espacio de nombres o crea uno, comenzará a usar los puntos de montaje asociados con ese espacio de nombres.
4. **Los descriptores de archivos e inodos se comparten entre espacios de nombres**, lo que significa que si un proceso en un espacio de nombres tiene un descriptor de archivo abierto apuntando a un archivo, puede **pasar ese descriptor de archivo** a un proceso en otro espacio de nombres, y **ambos procesos accederán al mismo archivo**. Sin embargo, la ruta del archivo puede no ser la misma en ambos espacios de nombres debido a diferencias en los puntos de montaje.

## Laboratorio:

### Crear diferentes Espacios de Nombres

#### CLI
```bash
sudo unshare -m [--mount-proc] /bin/bash
```
Al montar una nueva instancia del sistema de archivos `/proc` si usas el parámetro `--mount-proc`, te aseguras de que el nuevo espacio de nombres de montaje tenga una **vista precisa y aislada de la información de procesos específica de ese espacio de nombres**.

<details>

<summary>Error: bash: fork: No se puede asignar memoria</summary>

Si ejecutas la línea anterior sin `-f`, obtendrás ese error.\
El error es causado porque el proceso PID 1 sale en el nuevo espacio de nombres.

Después de que bash comience a ejecutarse, bash generará varios subprocesos nuevos para hacer algunas cosas. Si ejecutas unshare sin -f, bash tendrá el mismo pid que el proceso "unshare" actual. El proceso "unshare" actual llama al systemcall unshare, crea un nuevo espacio de nombres pid, pero el proceso "unshare" actual no está en el nuevo espacio de nombres pid. Es el comportamiento deseado del kernel de Linux: el proceso A crea un nuevo espacio de nombres, el propio proceso A no se colocará en el nuevo espacio de nombres, solo los subprocesos del proceso A se colocarán en el nuevo espacio de nombres. Entonces, cuando ejecutas:
</details>
```
unshare -p /bin/bash
```
El proceso unshare ejecutará /bin/bash, y /bin/bash generará varios subprocesos, el primer subproceso de bash se convertirá en el PID 1 del nuevo espacio de nombres, y el subproceso saldrá después de completar su trabajo. Entonces, el PID 1 del nuevo espacio de nombres sale.

El proceso PID 1 tiene una función especial: debe convertirse en el proceso padre de todos los procesos huérfanos. Si el proceso PID 1 en el espacio de nombres raíz sale, el kernel entrará en pánico. Si el proceso PID 1 en un subespacio de nombres sale, el kernel de Linux llamará a la función disable_pid_allocation, que limpiará la bandera PIDNS_HASH_ADDING en ese espacio de nombres. Cuando el kernel de Linux crea un nuevo proceso, el kernel llamará a la función alloc_pid para asignar un PID en un espacio de nombres, y si la bandera PIDNS_HASH_ADDING no está establecida, la función alloc_pid devolverá un error -ENOMEM. Por eso recibiste el error "Cannot allocate memory".

Puedes resolver este problema utilizando la opción '-f':
```
unshare -fp /bin/bash
```
```markdown
Si ejecutas unshare con la opción '-f', unshare bifurcará un nuevo proceso después de crear el nuevo espacio de nombres de pid. Y ejecutará /bin/bash en el nuevo proceso. El nuevo proceso será el pid 1 del nuevo espacio de nombres de pid. Luego, bash también bifurcará varios subprocesos para realizar algunas tareas. Como bash es el pid 1 del nuevo espacio de nombres de pid, sus subprocesos pueden salir sin ningún problema.

Copiado de [https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory](https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory)

</details>

#### Docker
```
```bash
docker run -ti --name ubuntu1 -v /usr:/ubuntu1 ubuntu bash
```
### &#x20;Verifica en qué espacio de nombres está tu proceso
```bash
ls -l /proc/self/ns/mnt
lrwxrwxrwx 1 root root 0 Apr  4 20:30 /proc/self/ns/mnt -> 'mnt:[4026531841]'
```
### Encuentra todos los espacios de nombres de montaje

{% code overflow="wrap" %}
```bash
sudo find /proc -maxdepth 3 -type l -name mnt -exec readlink {} \; 2>/dev/null | sort -u
# Find the processes with an specific namespace
sudo find /proc -maxdepth 3 -type l -name mnt -exec ls -l  {} \; 2>/dev/null | grep <ns-number>
```
### Entrar dentro de un espacio de nombres de montaje
```bash
nsenter -m TARGET_PID --pid /bin/bash
```
También, solo puedes **entrar en el espacio de nombres de otro proceso si eres root**. Y **no puedes** **entrar** en otro espacio de nombres **sin un descriptor** que apunte a él (como `/proc/self/ns/mnt`).

Debido a que los nuevos montajes solo son accesibles dentro del espacio de nombres, es posible que un espacio de nombres contenga información sensible que solo puede ser accesible desde él.

### Montar algo
```bash
# Generate new mount ns
unshare -m /bin/bash
mkdir /tmp/mount_ns_example
mount -t tmpfs tmpfs /tmp/mount_ns_example
mount | grep tmpfs # "tmpfs on /tmp/mount_ns_example"
echo test > /tmp/mount_ns_example/test
ls /tmp/mount_ns_example/test # Exists

# From the host
mount | grep tmpfs # Cannot see "tmpfs on /tmp/mount_ns_example"
ls /tmp/mount_ns_example/test # Doesn't exist
```
<details>

<summary><strong>Aprende hacking en AWS de cero a héroe con</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Otras formas de apoyar a HackTricks:

* Si quieres ver a tu **empresa anunciada en HackTricks** o **descargar HackTricks en PDF**, consulta los [**PLANES DE SUSCRIPCIÓN**](https://github.com/sponsors/carlospolop)!
* Consigue el [**merchandising oficial de PEASS & HackTricks**](https://peass.creator-spring.com)
* Descubre [**La Familia PEASS**](https://opensea.io/collection/the-peass-family), nuestra colección de [**NFTs**](https://opensea.io/collection/the-peass-family) exclusivos
* **Únete al** 💬 [**grupo de Discord**](https://discord.gg/hRep4RUj7f) o al [**grupo de telegram**](https://t.me/peass) o **sigue** a **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **Comparte tus trucos de hacking enviando PRs a los repositorios de github** [**HackTricks**](https://github.com/carlospolop/hacktricks) y [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
