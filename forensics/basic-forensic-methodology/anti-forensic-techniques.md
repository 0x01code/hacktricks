<details>

<summary><strong>Aprende hacking en AWS de cero a héroe con</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (Experto en Red Team de AWS de HackTricks)</strong></a><strong>!</strong></summary>

Otras formas de apoyar a HackTricks:

* Si deseas ver tu **empresa anunciada en HackTricks** o **descargar HackTricks en PDF** Consulta los [**PLANES DE SUSCRIPCIÓN**](https://github.com/sponsors/carlospolop)!
* Obtén el [**swag oficial de PEASS & HackTricks**](https://peass.creator-spring.com)
* Descubre [**La Familia PEASS**](https://opensea.io/collection/the-peass-family), nuestra colección exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Únete al** 💬 [**grupo de Discord**](https://discord.gg/hRep4RUj7f) o al [**grupo de telegram**](https://t.me/peass) o **síguenos** en **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Comparte tus trucos de hacking enviando PRs a** [**HackTricks**](https://github.com/carlospolop/hacktricks) y [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repositorios de github.

</details>


# Marcas de tiempo

Un atacante puede estar interesado en **cambiar las marcas de tiempo de los archivos** para evitar ser detectado.\
Es posible encontrar las marcas de tiempo dentro del MFT en los atributos `$STANDARD_INFORMATION` __ y __ `$FILE_NAME`.

Ambos atributos tienen 4 marcas de tiempo: **Modificación**, **acceso**, **creación** y **modificación del registro MFT** (MACE o MACB).

**El explorador de Windows** y otras herramientas muestran la información de **`$STANDARD_INFORMATION`**.

## TimeStomp - Herramienta Anti-forense

Esta herramienta **modifica** la información de la marca de tiempo dentro de **`$STANDARD_INFORMATION`** **pero** **no** la información dentro de **`$FILE_NAME`**. Por lo tanto, es posible **identificar** **actividad** **sospechosa**.

## Usnjrnl

El **Diario USN** (Diario de Número de Secuencia de Actualización), o Diario de Cambios, es una característica del sistema de archivos Windows NT (NTFS) que **mantiene un registro de los cambios realizados en el volumen**.\
Es posible utilizar la herramienta [**UsnJrnl2Csv**](https://github.com/jschicht/UsnJrnl2Csv) para buscar modificaciones en este registro.

![](<../../.gitbook/assets/image (449).png>)

La imagen anterior es la **salida** mostrada por la **herramienta** donde se puede observar que se realizaron algunos **cambios al archivo**.

## $LogFile

Todos los cambios de metadatos en un sistema de archivos se registran para garantizar la recuperación consistente de las estructuras críticas del sistema de archivos después de un fallo del sistema. Esto se llama [registro anticipado](https://en.wikipedia.org/wiki/Write-ahead\_logging).\
Los metadatos registrados se almacenan en un archivo llamado “**$LogFile**”, que se encuentra en un directorio raíz de un sistema de archivos NTFS.\
Es posible utilizar herramientas como [LogFileParser](https://github.com/jschicht/LogFileParser) para analizar este archivo y encontrar cambios.

![](<../../.gitbook/assets/image (450).png>)

Nuevamente, en la salida de la herramienta es posible ver que se realizaron **algunos cambios**.

Utilizando la misma herramienta es posible identificar a **qué hora se modificaron las marcas de tiempo**:

![](<../../.gitbook/assets/image (451).png>)

* CTIME: Hora de creación del archivo
* ATIME: Hora de modificación del archivo
* MTIME: Modificación del registro MFT del archivo
* RTIME: Hora de acceso al archivo

## Comparación de `$STANDARD_INFORMATION` y `$FILE_NAME`

Otra forma de identificar archivos modificados sospechosos sería comparar la hora en ambos atributos en busca de **diferencias**.

## Nanosegundos

Las marcas de tiempo de **NTFS** tienen una **precisión** de **100 nanosegundos**. Entonces, encontrar archivos con marcas de tiempo como 2010-10-10 10:10:**00.000:0000 es muy sospechoso**.

## SetMace - Herramienta Anti-forense

Esta herramienta puede modificar ambos atributos `$STARNDAR_INFORMATION` y `$FILE_NAME`. Sin embargo, a partir de Windows Vista, es necesario tener un sistema operativo en vivo para modificar esta información.

# Ocultación de datos

NTFS utiliza un clúster y el tamaño mínimo de información. Esto significa que si un archivo ocupa un clúster y medio, el **medio restante nunca se utilizará** hasta que se elimine el archivo. Entonces, es posible **ocultar datos en este espacio de desecho**.

Existen herramientas como slacker que permiten ocultar datos en este espacio "oculto". Sin embargo, un análisis del `$logfile` y `$usnjrnl` puede mostrar que se agregaron datos:

![](<../../.gitbook/assets/image (452).png>)

Entonces, es posible recuperar el espacio de desecho utilizando herramientas como FTK Imager. Ten en cuenta que este tipo de herramienta puede guardar el contenido de forma obstruida o incluso encriptada.

# UsbKill

Esta es una herramienta que **apagará la computadora si se detecta algún cambio en los puertos USB**.\
Una forma de descubrir esto sería inspeccionar los procesos en ejecución y **revisar cada script de Python en ejecución**.

# Distribuciones de Linux en vivo

Estas distribuciones se **ejecutan dentro de la memoria RAM**. La única forma de detectarlas es **en caso de que el sistema de archivos NTFS esté montado con permisos de escritura**. Si está montado solo con permisos de lectura, no será posible detectar la intrusión.

# Eliminación segura

[https://github.com/Claudio-C/awesome-data-sanitization](https://github.com/Claudio-C/awesome-data-sanitization)

# Configuración de Windows

Es posible deshabilitar varios métodos de registro de Windows para dificultar mucho la investigación forense.

## Deshabilitar Marcas de Tiempo - UserAssist

Esta es una clave del registro que mantiene las fechas y horas en que se ejecutó cada ejecutable por el usuario.

Deshabilitar UserAssist requiere dos pasos:

1. Establecer dos claves del registro, `HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Advanced\Start_TrackProgs` y `HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Advanced\Start_TrackEnabled`, ambos en cero para indicar que queremos deshabilitar UserAssist.
2. Limpiar los subárboles del registro que se parecen a `HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist\<hash>`.

## Deshabilitar Marcas de Tiempo - Prefetch

Esto guardará información sobre las aplicaciones ejecutadas con el objetivo de mejorar el rendimiento del sistema Windows. Sin embargo, esto también puede ser útil para prácticas forenses.

* Ejecutar `regedit`
* Seleccionar la ruta de archivo `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SessionManager\Memory Management\PrefetchParameters`
* Hacer clic derecho en tanto `EnablePrefetcher` como `EnableSuperfetch`
* Seleccionar Modificar en cada uno de estos para cambiar el valor de 1 (o 3) a 0
* Reiniciar

## Deshabilitar Marcas de Tiempo - Hora de Último Acceso

Cada vez que se abre una carpeta desde un volumen NTFS en un servidor Windows NT, el sistema toma tiempo para **actualizar un campo de marca de tiempo en cada carpeta listada**, llamado la hora de último acceso. En un volumen NTFS muy utilizado, esto puede afectar el rendimiento.

1. Abrir el Editor del Registro (Regedit.exe).
2. Navegar a `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\FileSystem`.
3. Buscar `NtfsDisableLastAccessUpdate`. Si no existe, agregar este DWORD y establecer su valor en 1, lo que deshabilitará el proceso.
4. Cerrar el Editor del Registro y reiniciar el servidor.

## Eliminar Historial de USB

Todas las **Entradas de Dispositivos USB** se almacenan en el Registro de Windows bajo la clave del registro **USBSTOR** que contiene subclaves que se crean cada vez que conectas un Dispositivo USB a tu PC o Laptop. Puedes encontrar esta clave aquí H`KEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Enum\USBSTOR`. **Al eliminar esto** eliminarás el historial de USB.\
También puedes usar la herramienta [**USBDeview**](https://www.nirsoft.net/utils/usb\_devices\_view.html) para asegurarte de haberlos eliminado (y para eliminarlos).

Otro archivo que guarda información sobre los USB es el archivo `setupapi.dev.log` dentro de `C:\Windows\INF`. Este también debería ser eliminado.

## Deshabilitar Copias de Sombra

**Listar** las copias de sombra con `vssadmin list shadowstorage`\
**Eliminar** ejecutando `vssadmin delete shadow`

También puedes eliminarlas a través de la GUI siguiendo los pasos propuestos en [https://www.ubackup.com/windows-10/how-to-delete-shadow-copies-windows-10-5740.html](https://www.ubackup.com/windows-10/how-to-delete-shadow-copies-windows-10-5740.html)

Para deshabilitar las copias de sombra:

1. Ir al botón de inicio de Windows y escribir "servicios" en el cuadro de búsqueda de texto; abrir el programa Servicios.
2. Localizar "Copia de Sombra de Volumen" en la lista, resaltarlo y luego hacer clic derecho > Propiedades.
3. Desde el menú desplegable "Tipo de inicio", seleccionar Deshabilitado, y luego hacer clic en Aplicar y Aceptar.

![](<../../.gitbook/assets/image (453).png>)

También es posible modificar la configuración de qué archivos se van a copiar en la copia de sombra en el registro `HKLM\SYSTEM\CurrentControlSet\Control\BackupRestore\FilesNotToSnapshot`

## Sobrescribir archivos eliminados

* Puedes usar una **herramienta de Windows**: `cipher /w:C` Esto indicará a cipher que elimine cualquier dato del espacio de disco no utilizado disponible dentro de la unidad C.
* También puedes usar herramientas como [**Eraser**](https://eraser.heidi.ie)

## Eliminar registros de eventos de Windows

* Windows + R --> eventvwr.msc --> Expandir "Registros de Windows" --> Hacer clic derecho en cada categoría y seleccionar "Borrar registro"
* `for /F "tokens=*" %1 in ('wevtutil.exe el') DO wevtutil.exe cl "%1"`
* `Get-EventLog -LogName * | ForEach { Clear-EventLog $_.Log }`

## Deshabilitar registros de eventos de Windows

* `reg add 'HKLM\SYSTEM\CurrentControlSet\Services\eventlog' /v Start /t REG_DWORD /d 4 /f`
* Dentro de la sección de servicios, deshabilitar el servicio "Registro de eventos de Windows"
* `WEvtUtil.exec clear-log` o `WEvtUtil.exe cl`

## Deshabilitar $UsnJrnl

* `fsutil usn deletejournal /d c:`

</details>
