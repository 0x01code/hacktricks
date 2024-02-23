# Abuso de Procesos en macOS

<details>

<summary><strong>Aprende hacking en AWS desde cero hasta experto con</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Otras formas de apoyar a HackTricks:

* Si deseas ver tu **empresa anunciada en HackTricks** o **descargar HackTricks en PDF** ¡Consulta los [**PLANES DE SUSCRIPCIÓN**](https://github.com/sponsors/carlospolop)!
* Obtén el [**oficial PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Descubre [**La Familia PEASS**](https://opensea.io/collection/the-peass-family), nuestra colección exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Únete al** 💬 [**grupo de Discord**](https://discord.gg/hRep4RUj7f) o al [**grupo de telegram**](https://t.me/peass) o **síguenos** en **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Comparte tus trucos de hacking enviando PRs a los repositorios de** [**HackTricks**](https://github.com/carlospolop/hacktricks) y [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>

## Abuso de Procesos en macOS

macOS, al igual que cualquier otro sistema operativo, proporciona una variedad de métodos y mecanismos para que los **procesos interactúen, se comuniquen y compartan datos**. Si bien estas técnicas son esenciales para el funcionamiento eficiente del sistema, también pueden ser abusadas por actores malintencionados para **realizar actividades maliciosas**.

### Inyección de Bibliotecas

La Inyección de Bibliotecas es una técnica en la que un atacante **obliga a un proceso a cargar una biblioteca maliciosa**. Una vez inyectada, la biblioteca se ejecuta en el contexto del proceso objetivo, proporcionando al atacante los mismos permisos y acceso que el proceso.

{% content-ref url="macos-library-injection/" %}
[macos-library-injection](macos-library-injection/)
{% endcontent-ref %}

### Enganche de Funciones

El Enganche de Funciones implica **interceptar llamadas de funciones** o mensajes dentro de un código de software. Al enganchar funciones, un atacante puede **modificar el comportamiento** de un proceso, observar datos sensibles o incluso obtener control sobre el flujo de ejecución.

{% content-ref url="../mac-os-architecture/macos-function-hooking.md" %}
[macos-function-hooking.md](../mac-os-architecture/macos-function-hooking.md)
{% endcontent-ref %}

### Comunicación entre Procesos

La Comunicación entre Procesos (IPC) se refiere a diferentes métodos mediante los cuales procesos separados **comparten e intercambian datos**. Si bien el IPC es fundamental para muchas aplicaciones legítimas, también puede ser mal utilizado para subvertir el aislamiento de procesos, filtrar información sensible o realizar acciones no autorizadas.

{% content-ref url="../mac-os-architecture/macos-ipc-inter-process-communication/" %}
[macos-ipc-inter-process-communication](../mac-os-architecture/macos-ipc-inter-process-communication/)
{% endcontent-ref %}

### Inyección de Aplicaciones Electron

Las aplicaciones Electron ejecutadas con variables de entorno específicas podrían ser vulnerables a la inyección de procesos:

{% content-ref url="macos-electron-applications-injection.md" %}
[macos-electron-applications-injection.md](macos-electron-applications-injection.md)
{% endcontent-ref %}

### Inyección en Chromium

Es posible utilizar las banderas `--load-extension` y `--use-fake-ui-for-media-stream` para realizar un **ataque de hombre en el navegador** que permita robar pulsaciones de teclas, tráfico, cookies, inyectar scripts en páginas, entre otros:

{% content-ref url="macos-chromium-injection.md" %}
[macos-chromium-injection.md](macos-chromium-injection.md)
{% endcontent-ref %}

### NIB Sucio

Los archivos NIB **definen elementos de interfaz de usuario (UI)** y sus interacciones dentro de una aplicación. Sin embargo, pueden **ejecutar comandos arbitrarios** y **Gatekeeper no impide** que una aplicación ya ejecutada vuelva a ejecutarse si se modifica un **archivo NIB**. Por lo tanto, podrían usarse para hacer que programas arbitrarios ejecuten comandos arbitrarios:

{% content-ref url="macos-dirty-nib.md" %}
[macos-dirty-nib.md](macos-dirty-nib.md)
{% endcontent-ref %}

### Inyección en Aplicaciones Java

Es posible abusar de ciertas capacidades de Java (como la variable de entorno **`_JAVA_OPTS`**) para hacer que una aplicación Java ejecute **código/comandos arbitrarios**.

{% content-ref url="macos-java-apps-injection.md" %}
[macos-java-apps-injection.md](macos-java-apps-injection.md)
{% endcontent-ref %}

### Inyección en Aplicaciones .Net

Es posible inyectar código en aplicaciones .Net **abusando de la funcionalidad de depuración de .Net** (no protegida por las protecciones de macOS como el endurecimiento en tiempo de ejecución).

{% content-ref url="macos-.net-applications-injection.md" %}
[macos-.net-applications-injection.md](macos-.net-applications-injection.md)
{% endcontent-ref %}

### Inyección en Perl

Consulta diferentes opciones para hacer que un script de Perl ejecute código arbitrario en:

{% content-ref url="macos-perl-applications-injection.md" %}
[macos-perl-applications-injection.md](macos-perl-applications-injection.md)
{% endcontent-ref %}

### Inyección en Ruby

También es posible abusar de las variables de entorno de Ruby para hacer que scripts arbitrarios ejecuten código arbitrario:

{% content-ref url="macos-ruby-applications-injection.md" %}
[macos-ruby-applications-injection.md](macos-ruby-applications-injection.md)
{% endcontent-ref %}

### Inyección en Python

Si se establece la variable de entorno **`PYTHONINSPECT`**, el proceso de Python ingresará a una interfaz de línea de comandos de Python una vez que haya terminado. También es posible usar **`PYTHONSTARTUP`** para indicar un script de Python que se ejecute al inicio de una sesión interactiva.\
Sin embargo, ten en cuenta que el script de **`PYTHONSTARTUP`** no se ejecutará cuando **`PYTHONINSPECT`** cree la sesión interactiva.

Otras variables de entorno como **`PYTHONPATH`** y **`PYTHONHOME`** también podrían ser útiles para hacer que un comando de Python ejecute código arbitrario.

Ten en cuenta que los ejecutables compilados con **`pyinstaller`** no utilizarán estas variables de entorno incluso si se ejecutan utilizando un Python integrado.

{% hint style="danger" %}
En general, no pude encontrar una forma de hacer que Python ejecute código arbitrario abusando de las variables de entorno.\
Sin embargo, la mayoría de las personas instalan Python usando **Hombrew**, que instalará Python en una **ubicación escribible** para el usuario administrador predeterminado. Puedes secuestrarlo con algo como:
```bash
mv /opt/homebrew/bin/python3 /opt/homebrew/bin/python3.old
cat > /opt/homebrew/bin/python3 <<EOF
#!/bin/bash
# Extra hijack code
/opt/homebrew/bin/python3.old "$@"
EOF
chmod +x /opt/homebrew/bin/python3
```
Incluso **root** ejecutará este código al ejecutar python.

## Detección

### Shield

[**Shield**](https://theevilbit.github.io/shield/) ([**Github**](https://github.com/theevilbit/Shield)) es una aplicación de código abierto que puede **detectar y bloquear acciones de inyección de procesos**:

* Usando **Variables de Entorno**: Monitorizará la presencia de cualquiera de las siguientes variables de entorno: **`DYLD_INSERT_LIBRARIES`**, **`CFNETWORK_LIBRARY_PATH`**, **`RAWCAMERA_BUNDLE_PATH`** y **`ELECTRON_RUN_AS_NODE`**
* Usando llamadas a **`task_for_pid`**: Para encontrar cuando un proceso quiere obtener el **puerto de tarea de otro** lo que permite inyectar código en el proceso.
* Parámetros de aplicaciones **Electron**: Alguien puede usar los argumentos de línea de comandos **`--inspect`**, **`--inspect-brk`** y **`--remote-debugging-port`** para iniciar una aplicación Electron en modo de depuración, y así inyectar código en ella.
* Usando **enlaces simbólicos** o **enlaces duros**: Típicamente el abuso más común es **colocar un enlace con nuestros privilegios de usuario**, y **apuntarlo a una ubicación de mayor privilegio**. La detección es muy simple tanto para enlaces duros como para enlaces simbólicos. Si el proceso que crea el enlace tiene un **nivel de privilegio diferente** al archivo de destino, creamos una **alerta**. Desafortunadamente, en el caso de los enlaces simbólicos, no es posible bloquear, ya que no tenemos información sobre el destino del enlace antes de la creación. Esta es una limitación del framework de EndpointSecuriy de Apple.

### Llamadas realizadas por otros procesos

En [**esta publicación de blog**](https://knight.sc/reverse%20engineering/2019/04/15/detecting-task-modifications.html) puedes encontrar cómo es posible utilizar la función **`task_name_for_pid`** para obtener información sobre otros **procesos que inyectan código en un proceso** y luego obtener información sobre ese otro proceso.

Ten en cuenta que para llamar a esa función necesitas tener el **mismo uid** que el que ejecuta el proceso o ser **root** (y devuelve información sobre el proceso, no una forma de inyectar código).

## Referencias

* [https://theevilbit.github.io/shield/](https://theevilbit.github.io/shield/)
* [https://medium.com/@metnew/why-electron-apps-cant-store-your-secrets-confidentially-inspect-option-a49950d6d51f](https://medium.com/@metnew/why-electron-apps-cant-store-your-secrets-confidentially-inspect-option-a49950d6d51f)

<details>

<summary><strong>Aprende hacking en AWS desde cero hasta experto con</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Otras formas de apoyar a HackTricks:

* Si quieres ver tu **empresa anunciada en HackTricks** o **descargar HackTricks en PDF** ¡Consulta los [**PLANES DE SUSCRIPCIÓN**](https://github.com/sponsors/carlospolop)!
* Obtén el [**swag oficial de PEASS & HackTricks**](https://peass.creator-spring.com)
* Descubre [**The PEASS Family**](https://opensea.io/collection/the-peass-family), nuestra colección exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Únete al** 💬 [**grupo de Discord**](https://discord.gg/hRep4RUj7f) o al [**grupo de telegram**](https://t.me/peass) o **síguenos** en **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Comparte tus trucos de hacking enviando PRs a los repositorios de** [**HackTricks**](https://github.com/carlospolop/hacktricks) y [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github.

</details>
