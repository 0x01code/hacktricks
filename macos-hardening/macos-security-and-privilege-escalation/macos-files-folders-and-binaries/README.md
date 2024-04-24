# Archivos, Carpetas, Binarios y Memoria de macOS

<details>

<summary><strong>Aprende a hackear AWS desde cero hasta convertirte en un experto con</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Otras formas de apoyar a HackTricks:

* Si deseas ver tu **empresa anunciada en HackTricks** o **descargar HackTricks en PDF** ¡Consulta los [**PLANES DE SUSCRIPCIÓN**](https://github.com/sponsors/carlospolop)!
* Obtén el [**oficial PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Descubre [**The PEASS Family**](https://opensea.io/collection/the-peass-family), nuestra colección exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Únete al** 💬 [**grupo de Discord**](https://discord.gg/hRep4RUj7f) o al [**grupo de telegram**](https://t.me/peass) o **síguenos** en **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Comparte tus trucos de hacking enviando PRs a los repositorios de** [**HackTricks**](https://github.com/carlospolop/hacktricks) y [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>

## Estructura jerárquica de archivos

* **/Applications**: Las aplicaciones instaladas deberían estar aquí. Todos los usuarios podrán acceder a ellas.
* **/bin**: Binarios de línea de comandos
* **/cores**: Si existe, se utiliza para almacenar volcados de núcleo
* **/dev**: Todo se trata como un archivo, por lo que es posible ver dispositivos de hardware almacenados aquí.
* **/etc**: Archivos de configuración
* **/Library**: Se pueden encontrar muchos subdirectorios y archivos relacionados con preferencias, cachés y registros. Existe una carpeta Library en la raíz y en el directorio de cada usuario.
* **/private**: No documentado, pero muchos de los directorios mencionados son enlaces simbólicos al directorio privado.
* **/sbin**: Binarios esenciales del sistema (relacionados con la administración)
* **/System**: Archivo para hacer funcionar OS X. Deberías encontrar principalmente archivos específicos de Apple aquí (no de terceros).
* **/tmp**: Los archivos se eliminan después de 3 días (es un enlace simbólico a /private/tmp)
* **/Users**: Directorio de inicio para los usuarios.
* **/usr**: Configuración y binarios del sistema
* **/var**: Archivos de registro
* **/Volumes**: Las unidades montadas aparecerán aquí.
* **/.vol**: Al ejecutar `stat a.txt` se obtiene algo como `16777223 7545753 -rw-r--r-- 1 nombredeusuario wheel ...` donde el primer número es el número de identificación del volumen donde se encuentra el archivo y el segundo es el número de inodo. Puedes acceder al contenido de este archivo a través de /.vol/ con esa información ejecutando `cat /.vol/16777223/7545753`

### Carpetas de Aplicaciones

* Las **aplicaciones del sistema** se encuentran en `/System/Applications`
* Las aplicaciones **instaladas** suelen estar en `/Applications` o en `~/Applications`
* Los **datos de la aplicación** se pueden encontrar en `/Library/Application Support` para las aplicaciones que se ejecutan como root y en `~/Library/Application Support` para las aplicaciones que se ejecutan como el usuario.
* Los **daemons** de aplicaciones de terceros que **necesitan ejecutarse como root** suelen estar ubicados en `/Library/PrivilegedHelperTools/`
* Las aplicaciones **sandboxed** se mapean en la carpeta `~/Library/Containers`. Cada aplicación tiene una carpeta con el nombre del ID de paquete de la aplicación (`com.apple.Safari`).
* El **núcleo** se encuentra en `/System/Library/Kernels/kernel`
* Las **extensiones de kernel de Apple** se encuentran en `/System/Library/Extensions`
* Las **extensiones de kernel de terceros** se almacenan en `/Library/Extensions`

### Archivos con Información Sensible

macOS almacena información como contraseñas en varios lugares:

{% content-ref url="macos-sensitive-locations.md" %}
[macos-sensitive-locations.md](macos-sensitive-locations.md)
{% endcontent-ref %}

### Instaladores pkg Vulnerables

{% content-ref url="macos-installers-abuse.md" %}
[macos-installers-abuse.md](macos-installers-abuse.md)
{% endcontent-ref %}

## Extensiones Específicas de OS X

* **`.dmg`**: Los archivos de imagen de disco de Apple son muy frecuentes para instaladores.
* **`.kext`**: Debe seguir una estructura específica y es la versión de OS X de un controlador (es un paquete).
* **`.plist`**: También conocido como lista de propiedades, almacena información en formato XML o binario.
* Puede ser XML o binario. Los binarios se pueden leer con:
* `defaults read config.plist`
* `/usr/libexec/PlistBuddy -c print config.plsit`
* `plutil -p ~/Library/Preferences/com.apple.screensaver.plist`
* `plutil -convert xml1 ~/Library/Preferences/com.apple.screensaver.plist -o -`
* `plutil -convert json ~/Library/Preferences/com.apple.screensaver.plist -o -`
* **`.app`**: Aplicaciones de Apple que siguen una estructura de directorio (es un paquete).
* **`.dylib`**: Bibliotecas dinámicas (como los archivos DLL de Windows)
* **`.pkg`**: Son iguales que xar (formato de archivo extensible). El comando installer se puede usar para instalar el contenido de estos archivos.
* **`.DS_Store`**: Este archivo está en cada directorio, guarda los atributos y personalizaciones del directorio.
* **`.Spotlight-V100`**: Esta carpeta aparece en el directorio raíz de cada volumen en el sistema.
* **`.metadata_never_index`**: Si este archivo está en la raíz de un volumen, Spotlight no indexará ese volumen.
* **`.noindex`**: Los archivos y carpetas con esta extensión no serán indexados por Spotlight.
* **`.sdef`**: Archivos dentro de paquetes que especifican cómo es posible interactuar con la aplicación desde un AppleScript.

### Paquetes de macOS

Un paquete es un **directorio** que **parece un objeto en Finder** (un ejemplo de paquete son los archivos `*.app`).

{% content-ref url="macos-bundles.md" %}
[macos-bundles.md](macos-bundles.md)
{% endcontent-ref %}

## Caché Compartida de Dyld

En macOS (y iOS) todas las bibliotecas compartidas del sistema, como frameworks y dylibs, se **combinan en un solo archivo**, llamado la **caché compartida de dyld**. Esto mejora el rendimiento, ya que el código se puede cargar más rápido.

Al igual que la caché compartida de dyld, el núcleo y las extensiones del núcleo también se compilan en una caché de núcleo, que se carga en el arranque.

Para extraer las bibliotecas del archivo único de la caché compartida de dylib, era posible utilizar el binario [dyld\_shared\_cache\_util](https://www.mbsplugins.de/files/dyld\_shared\_cache\_util-dyld-733.8.zip) que puede que no funcione en la actualidad, pero también puedes usar [**dyldextractor**](https://github.com/arandomdev/dyldextractor):

{% code overflow="wrap" %}
```bash
# dyld_shared_cache_util
dyld_shared_cache_util -extract ~/shared_cache/ /System/Volumes/Preboot/Cryptexes/OS/System/Library/dyld/dyld_shared_cache_arm64e

# dyldextractor
dyldex -l [dyld_shared_cache_path] # List libraries
dyldex_all [dyld_shared_cache_path] # Extract all
# More options inside the readme
```
{% endcode %}

En versiones antiguas es posible encontrar la **caché compartida** en **`/System/Library/dyld/`**.

En iOS puedes encontrarlas en **`/System/Library/Caches/com.apple.dyld/`**.

{% hint style="success" %}
Ten en cuenta que incluso si la herramienta `dyld_shared_cache_util` no funciona, puedes pasar el **binario dyld compartido a Hopper** y Hopper podrá identificar todas las bibliotecas y permitirte **seleccionar cuál** quieres investigar:
{% endhint %}

<figure><img src="../../../.gitbook/assets/image (1149).png" alt="" width="563"><figcaption></figcaption></figure>

## Permisos Especiales de Archivos

### Permisos de Carpeta

En una **carpeta**, **leer** permite **listarla**, **escribir** permite **borrar** y **escribir** archivos en ella, y **ejecutar** permite **atravesar** el directorio. Por lo tanto, por ejemplo, un usuario con **permiso de lectura sobre un archivo** dentro de un directorio donde no tiene permiso de **ejecución no podrá leer** el archivo.

### Modificadores de Bandera

Existen algunas banderas que se pueden establecer en los archivos y que harán que el archivo se comporte de manera diferente. Puedes **verificar las banderas** de los archivos dentro de un directorio con `ls -lO /ruta/directorio`

* **`uchg`**: Conocida como bandera de **uchange** evitará que se realice cualquier acción de cambio o eliminación del **archivo**. Para establecerla haz: `chflags uchg archivo.txt`
* El usuario root podría **quitar la bandera** y modificar el archivo
* **`restricted`**: Esta bandera hace que el archivo esté **protegido por SIP** (no puedes agregar esta bandera a un archivo).
* **`Bit pegajoso`**: Si un directorio tiene el bit pegajoso, **solo** el **propietario de los directorios o root pueden renombrar o eliminar** archivos. Normalmente esto se establece en el directorio /tmp para evitar que los usuarios normales eliminen o muevan archivos de otros usuarios.

Todas las banderas se pueden encontrar en el archivo `sys/stat.h` (encuéntralo usando `mdfind stat.h | grep stat.h`) y son:

* `UF_SETTABLE` 0x0000ffff: Máscara de banderas cambiables por el propietario.
* `UF_NODUMP` 0x00000001: No hacer volcado del archivo.
* `UF_IMMUTABLE` 0x00000002: El archivo no puede ser cambiado.
* `UF_APPEND` 0x00000004: Los escritos en el archivo solo pueden ser añadidos.
* `UF_OPAQUE` 0x00000008: El directorio es opaco con respecto a la unión.
* `UF_COMPRESSED` 0x00000020: El archivo está comprimido (algunos sistemas de archivos).
* `UF_TRACKED` 0x00000040: No hay notificaciones para eliminaciones/renombrados para archivos con esto establecido.
* `UF_DATAVAULT` 0x00000080: Se requiere autorización para lectura y escritura.
* `UF_HIDDEN` 0x00008000: Indica que este elemento no debe mostrarse en una interfaz gráfica.
* `SF_SUPPORTED` 0x009f0000: Máscara de banderas soportadas por el superusuario.
* `SF_SETTABLE` 0x3fff0000: Máscara de banderas cambiables por el superusuario.
* `SF_SYNTHETIC` 0xc0000000: Máscara de banderas sintéticas de solo lectura del sistema.
* `SF_ARCHIVED` 0x00010000: El archivo está archivado.
* `SF_IMMUTABLE` 0x00020000: El archivo no puede ser cambiado.
* `SF_APPEND` 0x00040000: Los escritos en el archivo solo pueden ser añadidos.
* `SF_RESTRICTED` 0x00080000: Se requiere autorización para escritura.
* `SF_NOUNLINK` 0x00100000: El elemento no puede ser eliminado, renombrado o montado.
* `SF_FIRMLINK` 0x00800000: El archivo es un firmlink.
* `SF_DATALESS` 0x40000000: El archivo es un objeto sin datos.

### **ACLs de Archivos**

Los **ACLs** de archivos contienen **ACE** (Entradas de Control de Acceso) donde se pueden asignar permisos más **granulares** a diferentes usuarios.

Es posible otorgar a una **carpeta** estos permisos: `listar`, `buscar`, `añadir_archivo`, `añadir_subdirectorio`, `eliminar_hijo`, `eliminar_hijo`.\
Y a un **archivo**: `leer`, `escribir`, `añadir`, `ejecutar`.

Cuando el archivo contiene ACLs, verás un **"+" al listar los permisos como en**:
```bash
ls -ld Movies
drwx------+   7 username  staff     224 15 Apr 19:42 Movies
```
Puedes **leer los ACLs** del archivo con:
```bash
ls -lde Movies
drwx------+ 7 username  staff  224 15 Apr 19:42 Movies
0: group:everyone deny delete
```
Puedes encontrar **todos los archivos con ACLs** con (esto es muuuy lento):
```bash
ls -RAle / 2>/dev/null | grep -E -B1 "\d: "
```
### Atributos Extendidos

Los atributos extendidos tienen un nombre y un valor deseado, y se pueden ver usando `ls -@` y manipular usando el comando `xattr`. Algunos atributos extendidos comunes son:

* `com.apple.resourceFork`: Compatibilidad con la bifurcación de recursos. También visible como `filename/..namedfork/rsrc`
* `com.apple.quarantine`: MacOS: Mecanismo de cuarentena de Gatekeeper (III/6)
* `metadata:*`: MacOS: varios metadatos, como `_backup_excludeItem`, o `kMD*`
* `com.apple.lastuseddate` (#PS): Fecha de último uso del archivo
* `com.apple.FinderInfo`: MacOS: Información del Finder (por ejemplo, etiquetas de color)
* `com.apple.TextEncoding`: Especifica la codificación de texto de archivos de texto ASCII
* `com.apple.logd.metadata`: Utilizado por logd en archivos en `/var/db/diagnostics`
* `com.apple.genstore.*`: Almacenamiento generacional (`/.DocumentRevisions-V100` en la raíz del sistema de archivos)
* `com.apple.rootless`: MacOS: Utilizado por Protección de Integridad del Sistema para etiquetar archivos (III/10)
* `com.apple.uuidb.boot-uuid`: Marcas de logd de épocas de arranque con UUID único
* `com.apple.decmpfs`: MacOS: Compresión de archivos transparente (II/7)
* `com.apple.cprotect`: \*OS: Datos de cifrado por archivo (III/11)
* `com.apple.installd.*`: \*OS: Metadatos utilizados por installd, por ejemplo, `installType`, `uniqueInstallID`

### Bifurcaciones de Recursos | ADS de macOS

Esta es una forma de obtener **Flujos de Datos Alternativos en las máquinas MacOS**. Puedes guardar contenido dentro de un atributo extendido llamado **com.apple.ResourceFork** dentro de un archivo guardándolo en **file/..namedfork/rsrc**.
```bash
echo "Hello" > a.txt
echo "Hello Mac ADS" > a.txt/..namedfork/rsrc

xattr -l a.txt #Read extended attributes
com.apple.ResourceFork: Hello Mac ADS

ls -l a.txt #The file length is still q
-rw-r--r--@ 1 username  wheel  6 17 Jul 01:15 a.txt
```
Puedes **encontrar todos los archivos que contienen este atributo extendido** con:

{% code overflow="wrap" %}
```bash
find / -type f -exec ls -ld {} \; 2>/dev/null | grep -E "[x\-]@ " | awk '{printf $9; printf "\n"}' | xargs -I {} xattr -lv {} | grep "com.apple.ResourceFork"
```
{% endcode %}

### decmpfs

El atributo extendido `com.apple.decmpfs` indica que el archivo está almacenado encriptado, `ls -l` reportará un **tamaño de 0** y los datos comprimidos están dentro de este atributo. Cada vez que se accede al archivo, se desencriptará en memoria.

Este atributo se puede ver con `ls -lO` indicado como comprimido porque los archivos comprimidos también están etiquetados con la bandera `UF_COMPRESSED`. Si se elimina un archivo comprimido con esta bandera usando `chflags nocompressed </path/to/file>`, el sistema no sabrá que el archivo estaba comprimido y, por lo tanto, no podrá descomprimirlo y acceder a los datos (pensará que en realidad está vacío).

La herramienta afscexpand se puede utilizar para forzar la descompresión de un archivo.

## **Binarios universales y** Formato Mach-o

Los binarios de Mac OS generalmente se compilan como **binarios universales**. Un **binario universal** puede **soportar múltiples arquitecturas en el mismo archivo**.

{% content-ref url="universal-binaries-and-mach-o-format.md" %}
[universal-binaries-and-mach-o-format.md](universal-binaries-and-mach-o-format.md)
{% endcontent-ref %}

## Volcado de memoria de macOS

{% content-ref url="macos-memory-dumping.md" %}
[macos-memory-dumping.md](macos-memory-dumping.md)
{% endcontent-ref %}

## Archivos de Categoría de Riesgo en Mac OS

El directorio `/System/Library/CoreServices/CoreTypes.bundle/Contents/Resources/System` es donde se almacena información sobre el **riesgo asociado con diferentes extensiones de archivo**. Este directorio categoriza los archivos en varios niveles de riesgo, influyendo en cómo Safari maneja estos archivos al descargarlos. Las categorías son las siguientes:

* **LSRiskCategorySafe**: Los archivos en esta categoría se consideran **completamente seguros**. Safari abrirá automáticamente estos archivos después de ser descargados.
* **LSRiskCategoryNeutral**: Estos archivos no vienen con advertencias y **no se abren automáticamente** en Safari.
* **LSRiskCategoryUnsafeExecutable**: Los archivos en esta categoría **generan una advertencia** que indica que el archivo es una aplicación. Esto sirve como una medida de seguridad para alertar al usuario.
* **LSRiskCategoryMayContainUnsafeExecutable**: Esta categoría es para archivos, como archivos comprimidos, que podrían contener un ejecutable. Safari **generará una advertencia** a menos que pueda verificar que todo el contenido es seguro o neutral.

## Archivos de registro

* **`$HOME/Library/Preferences/com.apple.LaunchServices.QuarantineEventsV2`**: Contiene información sobre archivos descargados, como la URL desde donde se descargaron.
* **`/var/log/system.log`**: Registro principal de los sistemas OSX. com.apple.syslogd.plist es responsable de la ejecución del registro del sistema (puedes verificar si está desactivado buscando "com.apple.syslogd" en `launchctl list`.
* **`/private/var/log/asl/*.asl`**: Estos son los Registros del Sistema de Apple que pueden contener información interesante.
* **`$HOME/Library/Preferences/com.apple.recentitems.plist`**: Almacena archivos y aplicaciones accedidos recientemente a través de "Finder".
* **`$HOME/Library/Preferences/com.apple.loginitems.plsit`**: Almacena elementos para iniciar al arrancar el sistema.
* **`$HOME/Library/Logs/DiskUtility.log`**: Archivo de registro para la aplicación DiskUtility (información sobre unidades, incluidas las USB).
* **`/Library/Preferences/SystemConfiguration/com.apple.airport.preferences.plist`**: Datos sobre puntos de acceso inalámbricos.
* **`/private/var/db/launchd.db/com.apple.launchd/overrides.plist`**: Lista de demonios desactivados.
