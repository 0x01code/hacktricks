<details>

<summary><strong>Aprende hacking en AWS desde cero hasta experto con</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (Experto en Red Team de AWS de HackTricks)</strong></a><strong>!</strong></summary>

Otras formas de apoyar a HackTricks:

* Si deseas ver tu **empresa anunciada en HackTricks** o **descargar HackTricks en PDF** Consulta los [**PLANES DE SUSCRIPCIÓN**](https://github.com/sponsors/carlospolop)!
* Obtén la [**merchandising oficial de PEASS & HackTricks**](https://peass.creator-spring.com)
* Descubre [**La Familia PEASS**](https://opensea.io/collection/the-peass-family), nuestra colección exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Únete al** 💬 [**grupo de Discord**](https://discord.gg/hRep4RUj7f) o al [**grupo de telegram**](https://t.me/peass) o **síguenos** en **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Comparte tus trucos de hacking enviando PRs a los** [**HackTricks**](https://github.com/carlospolop/hacktricks) y [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repositorios de github.

</details>


# JTAGenum

[**JTAGenum** ](https://github.com/cyphunk/JTAGenum)es una herramienta que se puede utilizar con una Raspberry PI o un Arduino para intentar encontrar pines JTAG de un chip desconocido.\
En el **Arduino**, conecta los **pines del 2 al 11 a 10 pines que potencialmente pertenecen a un JTAG**. Carga el programa en el Arduino y este intentará realizar fuerza bruta en todos los pines para encontrar si alguno pertenece a JTAG y cuál es cada uno.\
En la **Raspberry PI** solo puedes usar **pines del 1 al 6** (6 pines, por lo que irás más lento probando cada pin potencial de JTAG).

## Arduino

En Arduino, después de conectar los cables (pin 2 al 11 a los pines JTAG y Arduino GND a GND de la placa base), **carga el programa JTAGenum en Arduino** y en el Monitor Serie envía un **`h`** (comando para obtener ayuda) y deberías ver la ayuda:

![](<../../.gitbook/assets/image (643).png>)

![](<../../.gitbook/assets/image (650).png>)

Configura **"Sin final de línea" y 115200baud**.\
Envía el comando s para comenzar el escaneo:

![](<../../.gitbook/assets/image (651) (1) (1) (1).png>)

Si estás conectado a un JTAG, encontrarás una o varias **líneas que comienzan con ¡ENCONTRADO!** indicando los pines de JTAG.

</details>
