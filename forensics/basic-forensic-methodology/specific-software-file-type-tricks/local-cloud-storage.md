# Almacenamiento local en la nube

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* ¿Trabajas en una **empresa de ciberseguridad**? ¿Quieres ver tu **empresa anunciada en HackTricks**? ¿O quieres tener acceso a la **última versión de PEASS o descargar HackTricks en PDF**? ¡Consulta los [**PLANES DE SUSCRIPCIÓN**](https://github.com/sponsors/carlospolop)!
* Descubre [**The PEASS Family**](https://opensea.io/collection/the-peass-family), nuestra colección de exclusivos [**NFTs**](https://opensea.io/collection/the-peass-family)
* Obtén el [**oficial PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Únete al** [**💬**](https://emojipedia.org/speech-balloon/) [**grupo de Discord**](https://discord.gg/hRep4RUj7f) o al [**grupo de telegram**](https://t.me/peass) o **sígueme** en **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Comparte tus trucos de hacking enviando PR al** [**repositorio de hacktricks**](https://github.com/carlospolop/hacktricks) **y al** [**repositorio de hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>

![](<../.gitbook/assets/image (9) (1) (2).png>)

\
Usa [**Trickest**](https://trickest.io/) para construir y **automatizar flujos de trabajo** con las herramientas de la comunidad más avanzadas del mundo.\
Obtén acceso hoy:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## OneDrive

En Windows, puedes encontrar la carpeta de OneDrive en `\Users\<username>\AppData\Local\Microsoft\OneDrive`. Y dentro de `logs\Personal` es posible encontrar el archivo `SyncDiagnostics.log` que contiene algunos datos interesantes sobre los archivos sincronizados:

* Tamaño en bytes
* Fecha de creación
* Fecha de modificación
* Número de archivos en la nube
* Número de archivos en la carpeta
* **CID**: ID único del usuario de OneDrive
* Hora de generación del informe
* Tamaño del disco duro del sistema operativo

Una vez que hayas encontrado el CID, se recomienda **buscar archivos que contengan este ID**. Es posible que puedas encontrar archivos con el nombre: _**\<CID>.ini**_ y _**\<CID>.dat**_ que pueden contener información interesante como los nombres de los archivos sincronizados con OneDrive.

## Google Drive

En Windows, puedes encontrar la carpeta principal de Google Drive en `\Users\<username>\AppData\Local\Google\Drive\user_default`\
Esta carpeta contiene un archivo llamado Sync\_log.log con información como la dirección de correo electrónico de la cuenta, nombres de archivo, marcas de tiempo, hashes MD5 de los archivos, etc. Incluso los archivos eliminados aparecen en ese archivo de registro con su correspondiente MD5.

El archivo **`Cloud_graph\Cloud_graph.db`** es una base de datos sqlite que contiene la tabla **`cloud_graph_entry`**. En esta tabla puedes encontrar el **nombre** de los **archivos sincronizados**, la hora de modificación, el tamaño y el checksum MD5 de los archivos.

Los datos de la tabla de la base de datos **`Sync_config.db`** contienen la dirección de correo electrónico de la cuenta, la ruta de las carpetas compartidas y la versión de Google Drive.

## Dropbox

Dropbox utiliza **bases de datos SQLite** para gestionar los archivos. En esto\
Puedes encontrar las bases de datos en las carpetas:

* `\Users\<username>\AppData\Local\Dropbox`
* `\Users\<username>\AppData\Local\Dropbox\Instance1`
* `\Users\<username>\AppData\Roaming\Dropbox`

Y las bases de datos principales son:

* Sigstore.dbx
* Filecache.dbx
* Deleted.dbx
* Config.dbx

La extensión ".dbx" significa que las **bases de datos** están **encriptadas**. Dropbox utiliza **DPAPI** ([https://docs.microsoft.com/en-us/previous-versions/ms995355(v=msdn.10)?redirectedfrom=MSDN](https://docs.microsoft.com/en-us/previous-versions/ms995355\(v=msdn.10\)?redirectedfrom=MSDN))

Para entender mejor la encriptación que utiliza Dropbox, puedes leer [https://blog.digital-forensics.it/2017/04/brush-up-on-dropbox-dbx-decryption.html](https://blog.digital-forensics.it/2017/04/brush-up-on-dropbox-dbx-decryption.html).

Sin embargo, la información principal es:

* **Entropía**: d114a55212655f74bd772e37e64aee9b
* **Salt**: 0D638C092E8B82FC452883F95F355B8E
* **Algoritmo**: PBKDF2
* **Iteraciones**: 1066

Aparte de esa información, para descifrar las bases de datos todavía necesitas:

* La **clave DPAPI encriptada**: Puedes encontrarla en el registro dentro de `NTUSER.DAT\Software\Dropbox\ks\client` (exporta estos datos como binarios)
* Los archivos **`SYSTEM`** y **`SECURITY`**
* Las **claves maestras DPAPI**: Que se pueden encontrar en `\Users\<username>\AppData\Roaming\Microsoft\Protect`
* El **nombre de usuario** y la **contraseña** del usuario de Windows

Luego puedes usar la herramienta [**DataProtectionDecryptor**](https://nirsoft.net/utils/dpapi\_data\_decryptor.html)**:**

![](<../../../.gitbook/assets/image (448).png>)

Si todo va según lo esperado, la herramienta indicará la **clave primaria** que necesitas **usar para recuperar la original**. Para recuperar la original, simplemente usa esta [receta de cyber\_chef](https://gchq.github.io/CyberChef/#recipe=Derive\_PBKDF2\_key\(%7B'option':'Hex','string':'98FD6A76ECB87DE8DAB4623123402167'%7D,128,1066,'SHA1',%7B'option':'Hex','string':'0D638C092E8B82FC452883F95F355B8E'%7D\)) poniendo la clave primaria como la "contraseña" dentro de la receta.

El resultado en hexadecimal es la clave final utilizada para encriptar las bases de datos que se pueden descifrar con:
```bash
sqlite -k <Obtained Key> config.dbx ".backup config.db" #This decompress the config.dbx and creates a clear text backup in config.db
```
La base de datos **`config.dbx`** contiene:

* **Email**: El correo electrónico del usuario.
* **usernamedisplayname**: El nombre del usuario.
* **dropbox\_path**: La ruta donde se encuentra la carpeta de Dropbox.
* **Host\_id: Hash** utilizado para autenticarse en la nube. Esto solo se puede revocar desde la web.
* **Root\_ns**: Identificador de usuario.

La base de datos **`filecache.db`** contiene información sobre todos los archivos y carpetas sincronizados con Dropbox. La tabla `File_journal` es la que contiene más información útil:

* **Server\_path**: La ruta donde se encuentra el archivo dentro del servidor (esta ruta está precedida por el `host_id` del cliente).
* **local\_sjid**: Versión del archivo.
* **local\_mtime**: Fecha de modificación.
* **local\_ctime**: Fecha de creación.

Otras tablas dentro de esta base de datos contienen información más interesante:

* **block\_cache**: Hash de todos los archivos y carpetas de Dropbox.
* **block\_ref**: Relaciona el ID de hash de la tabla `block_cache` con el ID de archivo en la tabla `file_journal`.
* **mount\_table**: Comparte carpetas de Dropbox.
* **deleted\_fields**: Archivos eliminados de Dropbox.
* **date\_added**

![](<../.gitbook/assets/image (9) (1) (2).png>)

\
Utiliza [**Trickest**](https://trickest.io/) para construir y automatizar fácilmente flujos de trabajo impulsados por las herramientas de la comunidad más avanzadas del mundo.\
Obtén acceso hoy mismo:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* ¿Trabajas en una **empresa de ciberseguridad**? ¿Quieres ver tu **empresa anunciada en HackTricks**? ¿O quieres tener acceso a la **última versión de PEASS o descargar HackTricks en PDF**? ¡Consulta los [**PLANES DE SUSCRIPCIÓN**](https://github.com/sponsors/carlospolop)!
* Descubre [**The PEASS Family**](https://opensea.io/collection/the-peass-family), nuestra colección exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family).
* Obtén el [**swag oficial de PEASS y HackTricks**](https://peass.creator-spring.com).
* **Únete al** [**💬**](https://emojipedia.org/speech-balloon/) [**grupo de Discord**](https://discord.gg/hRep4RUj7f) o al [**grupo de telegram**](https://t.me/peass) o **sígueme** en **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Comparte tus trucos de hacking enviando PRs al** [**repositorio de hacktricks**](https://github.com/carlospolop/hacktricks) **y al** [**repositorio de hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
