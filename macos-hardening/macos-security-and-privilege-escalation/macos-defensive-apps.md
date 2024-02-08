# Aplicaciones defensivas de macOS

<details>

<summary><strong>Aprende a hackear AWS desde cero hasta convertirte en un experto con</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (Experto en Red Team de AWS de HackTricks)</strong></a><strong>!</strong></summary>

Otras formas de apoyar a HackTricks:

* Si deseas ver tu **empresa anunciada en HackTricks** o **descargar HackTricks en PDF**, ¡Consulta los [**PLANES DE SUSCRIPCIÓN**](https://github.com/sponsors/carlospolop)!
* Obtén la [**merchandising oficial de PEASS & HackTricks**](https://peass.creator-spring.com)
* Descubre [**La Familia PEASS**](https://opensea.io/collection/the-peass-family), nuestra colección de [**NFTs**](https://opensea.io/collection/the-peass-family) exclusivos
* **Únete al** 💬 [**grupo de Discord**](https://discord.gg/hRep4RUj7f) o al [**grupo de telegram**](https://t.me/peass) o **síguenos** en **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Comparte tus trucos de hacking enviando PRs a los repositorios de** [**HackTricks**](https://github.com/carlospolop/hacktricks) y [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>

## Firewalls

* [**Little Snitch**](https://www.obdev.at/products/littlesnitch/index.html): Monitoreará cada conexión realizada por cada proceso. Dependiendo del modo (permitir conexiones silenciosamente, denegar conexiones silenciosamente y alertar), te **mostrará una alerta** cada vez que se establezca una nueva conexión. También tiene una interfaz gráfica muy agradable para ver toda esta información.
* [**LuLu**](https://objective-see.org/products/lulu.html): Firewall de Objective-See. Este es un firewall básico que te alertará sobre conexiones sospechosas (tiene una GUI pero no es tan elegante como la de Little Snitch).

## Detección de persistencia

* [**KnockKnock**](https://objective-see.org/products/knockknock.html): Aplicación de Objective-See que buscará en varias ubicaciones donde **el malware podría estar persistiendo** (es una herramienta de un solo uso, no un servicio de monitoreo).
* [**BlockBlock**](https://objective-see.org/products/blockblock.html): Similar a KnockKnock al monitorear procesos que generan persistencia.

## Detección de keyloggers

* [**ReiKey**](https://objective-see.org/products/reikey.html): Aplicación de Objective-See para encontrar **keyloggers** que instalan "event taps" de teclado.

## Detección de ransomware

* [**RansomWhere**](https://objective-see.org/products/ransomwhere.html): Aplicación de Objective-See para detectar acciones de **encriptación de archivos**.

## Detección de micrófono y cámara web

* [**OverSight**](https://objective-see.org/products/oversight.html): Aplicación de Objective-See para detectar **aplicaciones que comienzan a usar la cámara web y el micrófono**.

## Detección de inyección de procesos

* [**Shield**](https://theevilbit.github.io/shield/): Aplicación que **detecta diferentes técnicas de inyección de procesos**.
