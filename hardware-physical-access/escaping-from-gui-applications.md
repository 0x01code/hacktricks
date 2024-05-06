# Escapando de los KIOSKs

<details>

<summary><strong>Aprende hacking en AWS desde cero hasta experto con</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Otras formas de apoyar a HackTricks:

* Si deseas ver tu **empresa anunciada en HackTricks** o **descargar HackTricks en PDF** ¡Consulta los [**PLANES DE SUSCRIPCIÓN**](https://github.com/sponsors/carlospolop)!
* Obtén el [**oficial PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Descubre [**The PEASS Family**](https://opensea.io/collection/the-peass-family), nuestra colección exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Únete al** 💬 [**grupo de Discord**](https://discord.gg/hRep4RUj7f) o al [**grupo de telegram**](https://t.me/peass) o **síguenos** en **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Comparte tus trucos de hacking enviando PRs a los repositorios de** [**HackTricks**](https://github.com/carlospolop/hacktricks) y [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>

#### [WhiteIntel](https://whiteintel.io)

<figure><img src="../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) es un motor de búsqueda alimentado por la **dark web** que ofrece funcionalidades **gratuitas** para verificar si una empresa o sus clientes han sido **comprometidos** por **malwares de robo**.

El objetivo principal de WhiteIntel es combatir los secuestros de cuentas y los ataques de ransomware resultantes de malwares que roban información.

Puedes visitar su sitio web y probar su motor de forma **gratuita** en:

{% embed url="https://whiteintel.io" %}

---

## Verificar dispositivo físico

|   Componente   | Acción                                                               |
| ------------- | -------------------------------------------------------------------- |
| Botón de encendido  | Apagar y encender el dispositivo nuevamente puede exponer la pantalla de inicio      |
| Cable de alimentación   | Verificar si el dispositivo se reinicia cuando se corta brevemente la energía   |
| Puertos USB     | Conectar un teclado físico con más accesos directos                        |
| Ethernet      | Un escaneo de red o sniffing puede permitir una mayor explotación             |


## Verificar posibles acciones dentro de la aplicación GUI

Los **Diálogos comunes** son esas opciones de **guardar un archivo**, **abrir un archivo**, seleccionar una fuente, un color... La mayoría de ellos **ofrecerán una funcionalidad completa de Explorador**. Esto significa que podrás acceder a funcionalidades de Explorador si puedes acceder a estas opciones:

* Cerrar/Cerrar como
* Abrir/Abrir con
* Imprimir
* Exportar/Importar
* Buscar
* Escanear

Deberías verificar si puedes:

* Modificar o crear nuevos archivos
* Crear enlaces simbólicos
* Acceder a áreas restringidas
* Ejecutar otras aplicaciones

### Ejecución de comandos

Tal vez **usando la opción `Abrir con`** puedas abrir/ejecutar algún tipo de shell.

#### Windows

Por ejemplo _cmd.exe, command.com, Powershell/Powershell ISE, mmc.exe, at.exe, taskschd.msc..._ encuentra más binarios que se pueden usar para ejecutar comandos (y realizar acciones inesperadas) aquí: [https://lolbas-project.github.io/](https://lolbas-project.github.io)

#### \*NIX \_\_

_bash, sh, zsh..._ Más aquí: [https://gtfobins.github.io/](https://gtfobins.github.io)

## Windows

### Saltando restricciones de ruta

* **Variables de entorno**: Hay muchas variables de entorno que apuntan a alguna ruta
* **Otros protocolos**: _about:, data:, ftp:, file:, mailto:, news:, res:, telnet:, view-source:_
* **Enlaces simbólicos**
* **Accesos directos**: CTRL+N (abrir nueva sesión), CTRL+R (Ejecutar comandos), CTRL+SHIFT+ESC (Administrador de tareas), Windows+E (abrir explorador), CTRL-B, CTRL-I (Favoritos), CTRL-H (Historial), CTRL-L, CTRL-O (Diálogo de Archivo/Abrir), CTRL-P (Diálogo de Impresión), CTRL-S (Guardar como)
* Menú administrativo oculto: CTRL-ALT-F8, CTRL-ESC-F9
* **URIs de shell**: _shell:Herramientas Administrativas, shell:Bibliotecas de Documentos, shell:Bibliotecas, shell:Perfiles de Usuario, shell:Personal, shell:Carpeta de Búsqueda, shell:Sistemas, shell:Carpetas de Red, shell:Enviar a, shell:Perfiles de Usuarios, shell:Herramientas Administrativas Comunes, shell:Mi PC, shell:Carpeta de Internet_
* **Rutas UNC**: Rutas para conectarse a carpetas compartidas. Deberías intentar conectarte al C$ de la máquina local ("\\\127.0.0.1\c$\Windows\System32")
* **Más rutas UNC:**

| UNC                       | UNC            | UNC                  |
| ------------------------- | -------------- | -------------------- |
| %ALLUSERSPROFILE%         | %APPDATA%      | %CommonProgramFiles% |
| %COMMONPROGRAMFILES(x86)% | %COMPUTERNAME% | %COMSPEC%            |
| %HOMEDRIVE%               | %HOMEPATH%     | %LOCALAPPDATA%       |
| %LOGONSERVER%             | %PATH%         | %PATHEXT%            |
| %ProgramData%             | %ProgramFiles% | %ProgramFiles(x86)%  |
| %PROMPT%                  | %PSModulePath% | %Public%             |
| %SYSTEMDRIVE%             | %SYSTEMROOT%   | %TEMP%               |
| %TMP%                     | %USERDOMAIN%   | %USERNAME%           |
| %USERPROFILE%             | %WINDIR%       |                      |

### Descarga tus Binarios

Consola: [https://sourceforge.net/projects/console/](https://sourceforge.net/projects/console/)\
Explorador: [https://sourceforge.net/projects/explorerplus/files/Explorer%2B%2B/](https://sourceforge.net/projects/explorerplus/files/Explorer%2B%2B/)\
Editor de registro: [https://sourceforge.net/projects/uberregedit/](https://sourceforge.net/projects/uberregedit/)

### Accediendo al sistema de archivos desde el navegador

| RUTA                | RUTA              | RUTA               | RUTA                |
| ------------------- | ----------------- | ------------------ | ------------------- |
| File:/C:/windows    | File:/C:/windows/ | File:/C:/windows\\ | File:/C:\windows    |
| File:/C:\windows\\  | File:/C:\windows/ | File://C:/windows  | File://C:/windows/  |
| File://C:/windows\\ | File://C:\windows | File://C:\windows/ | File://C:\windows\\ |
| C:/windows          | C:/windows/       | C:/windows\\       | C:\windows          |
| C:\windows\\        | C:\windows/       | %WINDIR%           | %TMP%               |
| %TEMP%              | %SYSTEMDRIVE%     | %SYSTEMROOT%       | %APPDATA%           |
| %HOMEDRIVE%         | %HOMESHARE        |                    | <p><br></p>         |
### Atajos

* Sticky Keys – Presiona SHIFT 5 veces
* Mouse Keys – SHIFT+ALT+BLOQ NUM
* Alto Contraste – SHIFT+ALT+IMPR PANT
* Teclas de Alternancia – Mantén presionado BLOQ NUM por 5 segundos
* Teclas de Filtro – Mantén presionado el SHIFT derecho por 12 segundos
* WINDOWS+F1 – Búsqueda de Windows
* WINDOWS+D – Mostrar Escritorio
* WINDOWS+E – Abrir Explorador de Windows
* WINDOWS+R – Ejecutar
* WINDOWS+U – Centro de Accesibilidad
* WINDOWS+F – Búsqueda
* SHIFT+F10 – Menú Contextual
* CTRL+SHIFT+ESC – Administrador de Tareas
* CTRL+ALT+DEL – Pantalla de inicio en versiones más nuevas de Windows
* F1 – Ayuda F3 – Búsqueda
* F6 – Barra de direcciones
* F11 – Alternar pantalla completa en Internet Explorer
* CTRL+H – Historial de Internet Explorer
* CTRL+T – Internet Explorer – Nueva pestaña
* CTRL+N – Internet Explorer – Nueva página
* CTRL+O – Abrir Archivo
* CTRL+S – Guardar CTRL+N – Nueva RDP / Citrix

### Deslizamientos

* Desliza desde el lado izquierdo hacia la derecha para ver todas las ventanas abiertas, minimizando la aplicación KIOSK y accediendo directamente a todo el sistema operativo;
* Desliza desde el lado derecho hacia la izquierda para abrir el Centro de Acción, minimizando la aplicación KIOSK y accediendo directamente a todo el sistema operativo;
* Desliza desde el borde superior para hacer visible la barra de título de una aplicación abierta en modo de pantalla completa;
* Desliza hacia arriba desde la parte inferior para mostrar la barra de tareas en una aplicación de pantalla completa.

### Trucos de Internet Explorer

#### 'Barra de Imagen'

Es una barra de herramientas que aparece en la parte superior izquierda de la imagen cuando se hace clic en ella. Podrás Guardar, Imprimir, Enviar por correo electrónico, Abrir "Mis imágenes" en el Explorador. El Kiosk debe estar utilizando Internet Explorer.

#### Protocolo Shell

Escribe estas URL para obtener una vista de Explorador:

* `shell:Herramientas Administrativas`
* `shell:BibliotecaDocumentos`
* `shell:Bibliotecas`
* `shell:PerfilesUsuarios`
* `shell:Personal`
* `shell:CarpetaInicioBusqueda`
* `shell:CarpetaLugaresRed`
* `shell:EnviarA`
* `shell:PerfilesUsuarios`
* `shell:HerramientasAdministrativasComunes`
* `shell:MiPC`
* `shell:Internet`
* `Shell:Perfil`
* `Shell:ArchivosPrograma`
* `Shell:Sistema`
* `Shell:PanelControl`
* `Shell:Windows`
* `shell:::{21EC2020-3AEA-1069-A2DD-08002B30309D}` --> Panel de Control
* `shell:::{20D04FE0-3AEA-1069-A2D8-08002B30309D}` --> Mi PC
* `shell:::{{208D2C60-3AEA-1069-A2D7-08002B30309D}}` --> Mis sitios de red
* `shell:::{871C5380-42A0-1069-A2EA-08002B30309D}` --> Internet Explorer

### Mostrar Extensiones de Archivos

Consulta esta página para más información: [https://www.howtohaven.com/system/show-file-extensions-in-windows-explorer.shtml](https://www.howtohaven.com/system/show-file-extensions-in-windows-explorer.shtml)

## Trucos de Navegadores

Respaldo de versiones iKat:

[http://swin.es/k/](http://swin.es/k/)\
[http://www.ikat.kronicd.net/](http://www.ikat.kronicd.net)\\

Crear un cuadro de diálogo común usando JavaScript y acceder al explorador de archivos: `document.write('<input/type=file>')`\
Fuente: https://medium.com/@Rend\_/give-me-a-browser-ill-give-you-a-shell-de19811defa0

## iPad

### Gestos y botones

* Desliza hacia arriba con cuatro (o cinco) dedos / Doble toque en el botón de inicio: Para ver la vista de multitarea y cambiar de aplicación
* Desliza de un lado a otro con cuatro o cinco dedos: Para cambiar a la siguiente/anterior aplicación
* Pellizca la pantalla con cinco dedos / Toca el botón de inicio / Desliza hacia arriba con 1 dedo desde la parte inferior de la pantalla en un movimiento rápido hacia arriba: Para acceder a Inicio
* Desliza un dedo desde la parte inferior de la pantalla solo 1-2 pulgadas (lento): Aparecerá el dock
* Desliza hacia abajo desde la parte superior de la pantalla con 1 dedo: Para ver tus notificaciones
* Desliza hacia abajo con 1 dedo en la esquina superior derecha de la pantalla: Para ver el centro de control del iPad Pro
* Desliza 1 dedo desde el lado izquierdo de la pantalla 1-2 pulgadas: Para ver la vista de Hoy
* Desliza rápidamente 1 dedo desde el centro de la pantalla hacia la derecha o izquierda: Para cambiar a la siguiente/anterior aplicación
* Presiona y mantén presionado el botón de Encendido/Apagado en la esquina superior derecha del iPad + Mueve el deslizador de Apagar completamente hacia la derecha: Para apagar
* Presiona el botón de Encendido/Apagado en la esquina superior derecha del iPad y el botón de Inicio durante unos segundos: Para forzar un apagado
* Presiona el botón de Encendido/Apagado en la esquina superior derecha del iPad y el botón de Inicio rápidamente: Para tomar una captura de pantalla que aparecerá en la parte inferior izquierda de la pantalla. Presiona ambos botones al mismo tiempo brevemente, si los mantienes presionados unos segundos se realizará un apagado forzado.

### Atajos

Debes tener un teclado para iPad o un adaptador de teclado USB. Aquí se mostrarán solo los atajos que podrían ayudar a escapar de la aplicación.

| Tecla | Nombre         |
| --- | ------------ |
| ⌘   | Comando      |
| ⌥   | Opción (Alt) |
| ⇧   | Mayús        |
| ↩   | Retorno       |
| ⇥   | Tabulador          |
| ^   | Control      |
| ←   | Flecha Izquierda   |
| →   | Flecha Derecha  |
| ↑   | Flecha Arriba     |
| ↓   | Flecha Abajo   |

#### Atajos del sistema

Estos atajos son para la configuración visual y de sonido, dependiendo del uso del iPad.

| Atajo | Acción                                                                         |
| -------- | ------------------------------------------------------------------------------ |
| F1       | Atenuar pantalla                                                                    |
| F2       | Aumentar brillo de pantalla                                                                |
| F7       | Retroceder una canción                                                                  |
| F8       | Reproducir/pausar                                                                     |
| F9       | Saltar canción                                                                      |
| F10      | Silenciar                                                                           |
| F11      | Disminuir volumen                                                                |
| F12      | Aumentar volumen                                                                |
| ⌘ Espacio  | Mostrar una lista de idiomas disponibles; para elegir uno, toca de nuevo la barra espaciadora. |

#### Navegación en iPad

| Atajo                                           | Acción                                                  |
| -------------------------------------------------- | ------------------------------------------------------- |
| ⌘H                                                 | Ir a Inicio                                              |
| ⌘⇧H (Comando-Mayús-H)                              | Ir a Inicio                                              |
| ⌘ (Espacio)                                          | Abrir Spotlight                                          |
| ⌘⇥ (Comando-Tabulador)                                   | Listar las últimas diez aplicaciones usadas                                 |
| ⌘\~                                                | Ir a la última aplicación                                       |
| ⌘⇧3 (Comando-Mayús-3)                              | Captura de pantalla (aparece en la esquina inferior izquierda para guardar o actuar sobre ella) |
| ⌘⇧4                                                | Captura de pantalla y ábrela en el editor                    |
| Presiona y mantén ⌘                                   | Lista de atajos disponibles para la aplicación                 |
| ⌘⌥D (Comando-Opción/Alt-D)                         | Muestra el dock                                      |
| ^⌥H (Control-Opción-H)                             | Botón de Inicio                                             |
| ^⌥H H (Control-Opción-H-H)                         | Mostrar barra de multitarea                                      |
| ^⌥I (Control-Opción-i)                             | Selector de elementos                                            |
| Escape                                             | Botón de retroceso                                             |
| → (Flecha derecha)                                    | Siguiente elemento                                               |
| ← (Flecha izquierda)                                     | Elemento anterior                                           |
| ↑↓ (Flecha arriba, Flecha abajo)                          | Toca simultáneamente el elemento seleccionado                        |
| ⌥ ↓ (Opción-Flecha abajo)                            | Desplazarse hacia abajo                                             |
| ⌥↑ (Opción-Flecha arriba)                               | Desplazarse hacia arriba                                               |
| ⌥← o ⌥→ (Opción-Flecha izquierda o Opción-Flecha derecha) | Desplazarse hacia la izquierda o derecha                                    |
| ^⌥S (Control-Opción-S)                             | Activar o desactivar el habla de VoiceOver                         |
| ⌘⇧⇥ (Comando-Mayús-Tabulador)                            | Cambiar a la aplicación anterior                              |
| ⌘⇥ (Comando-Tabulador)                                   | Volver a la aplicación original                         |
| ←+→, luego Opción + ← o Opción+→                   | Navegar a través del Dock                                   |
#### Atajos de Safari

| Atajo                   | Acción                                           |
| ----------------------- | ------------------------------------------------ |
| ⌘L (Comando-L)          | Abrir ubicación                                  |
| ⌘T                      | Abrir una nueva pestaña                          |
| ⌘W                      | Cerrar la pestaña actual                         |
| ⌘R                      | Actualizar la pestaña actual                    |
| ⌘.                      | Detener la carga de la pestaña actual            |
| ^⇥                      | Cambiar a la siguiente pestaña                   |
| ^⇧⇥ (Control-Mayús-Tab) | Moverse a la pestaña anterior                    |
| ⌘L                      | Seleccionar el campo de entrada de texto/URL para modificarlo |
| ⌘⇧T (Comando-Mayús-T)   | Abrir la última pestaña cerrada (se puede usar varias veces) |
| ⌘\[                     | Retroceder una página en tu historial de navegación |
| ⌘]                      | Avanzar una página en tu historial de navegación |
| ⌘⇧R                     | Activar el Modo Lector                            |

#### Atajos de Correo

| Atajo                   | Acción                       |
| ----------------------- | ---------------------------- |
| ⌘L                      | Abrir ubicación              |
| ⌘T                      | Abrir una nueva pestaña      |
| ⌘W                      | Cerrar la pestaña actual     |
| ⌘R                      | Actualizar la pestaña actual |
| ⌘.                      | Detener la carga de la pestaña actual |
| ⌘⌥F (Comando-Opción/Alt-F) | Buscar en tu buzón de correo  |

## Referencias

* [https://www.macworld.com/article/2975857/6-only-for-ipad-gestures-you-need-to-know.html](https://www.macworld.com/article/2975857/6-only-for-ipad-gestures-you-need-to-know.html)
* [https://www.tomsguide.com/us/ipad-shortcuts,news-18205.html](https://www.tomsguide.com/us/ipad-shortcuts,news-18205.html)
* [https://thesweetsetup.com/best-ipad-keyboard-shortcuts/](https://thesweetsetup.com/best-ipad-keyboard-shortcuts/)
* [http://www.iphonehacks.com/2018/03/ipad-keyboard-shortcuts.html](http://www.iphonehacks.com/2018/03/ipad-keyboard-shortcuts.html)

#### [WhiteIntel](https://whiteintel.io)

<figure><img src="../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) es un motor de búsqueda alimentado por la **dark web** que ofrece funcionalidades **gratuitas** para verificar si una empresa o sus clientes han sido **comprometidos** por **malwares robadores**.

El objetivo principal de WhiteIntel es combatir tomas de cuentas y ataques de ransomware resultantes de malwares que roban información.

Puedes visitar su sitio web y probar su motor de búsqueda de forma **gratuita** en:

{% embed url="https://whiteintel.io" %}

<details>

<summary><strong>Aprende a hackear AWS desde cero hasta experto con</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Otras formas de apoyar a HackTricks:

* Si deseas ver tu **empresa anunciada en HackTricks** o **descargar HackTricks en PDF** ¡Consulta los [**PLANES DE SUSCRIPCIÓN**](https://github.com/sponsors/carlospolop)!
* Obtén el [**oficial PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Descubre [**The PEASS Family**](https://opensea.io/collection/the-peass-family), nuestra colección exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Únete al** 💬 [**grupo de Discord**](https://discord.gg/hRep4RUj7f) o al [**grupo de telegram**](https://t.me/peass) o **síguenos** en **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Comparte tus trucos de hacking enviando PRs a los repositorios de** [**HackTricks**](https://github.com/carlospolop/hacktricks) y [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
