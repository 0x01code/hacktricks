<details>

<summary><strong>Aprende hacking en AWS de cero a héroe con</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (Experto en Equipo Rojo de AWS de HackTricks)</strong></a><strong>!</strong></summary>

Otras formas de apoyar a HackTricks:

* Si deseas ver tu **empresa anunciada en HackTricks** o **descargar HackTricks en PDF** ¡Consulta los [**PLANES DE SUSCRIPCIÓN**](https://github.com/sponsors/carlospolop)!
* Obtén el [**swag oficial de PEASS & HackTricks**](https://peass.creator-spring.com)
* Descubre [**La Familia PEASS**](https://opensea.io/collection/the-peass-family), nuestra colección exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Únete al** 💬 [**grupo de Discord**](https://discord.gg/hRep4RUj7f) o al [**grupo de telegram**](https://t.me/peass) o **síguenos** en **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Comparte tus trucos de hacking enviando PRs a los** [**HackTricks**](https://github.com/carlospolop/hacktricks) y [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repositorios de github.

</details>


# Resumen del ataque

Imagina un servidor que está **firmando** algunos **datos** al **añadir** un **secreto** a algunos datos de texto claro conocidos y luego hasheando esos datos. Si conoces:

* **La longitud del secreto** (esto también se puede obtener por fuerza bruta dentro de un rango de longitudes dado)
* **Los datos de texto claro**
* **El algoritmo (y es vulnerable a este ataque)**
* **El relleno es conocido**
* Por lo general, se usa uno predeterminado, por lo que si se cumplen los otros 3 requisitos, esto también lo está
* El relleno varía dependiendo de la longitud del secreto+datos, por eso se necesita la longitud del secreto

Entonces, es posible para un **atacante** **añadir** **datos** y **generar** una **firma válida** para los **datos anteriores + datos añadidos**.

## ¿Cómo?

Básicamente, los algoritmos vulnerables generan los hashes primero **hasheando un bloque de datos**, y luego, **a partir** del **hash creado previamente** (estado), **añaden el siguiente bloque de datos** y lo **hashean**.

Entonces, imagina que el secreto es "secreto" y los datos son "datos", el MD5 de "secretodata" es 6036708eba0d11f6ef52ad44e8b74d5b.\
Si un atacante quiere añadir la cadena "añadir" puede:

* Generar un MD5 de 64 "A"s
* Cambiar el estado del hash inicializado previamente a 6036708eba0d11f6ef52ad44e8b74d5b
* Añadir la cadena "añadir"
* Finalizar el hash y el hash resultante será uno **válido para "secreto" + "datos" + "relleno" + "añadir"**

## **Herramienta**

{% embed url="https://github.com/iagox86/hash_extender" %}

## Referencias

Puedes encontrar este ataque bien explicado en [https://blog.skullsecurity.org/2012/everything-you-need-to-know-about-hash-length-extension-attacks](https://blog.skullsecurity.org/2012/everything-you-need-to-know-about-hash-length-extension-attacks)


<details>

<summary><strong>Aprende hacking en AWS de cero a héroe con</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (Experto en Equipo Rojo de AWS de HackTricks)</strong></a><strong>!</strong></summary>

Otras formas de apoyar a HackTricks:

* Si deseas ver tu **empresa anunciada en HackTricks** o **descargar HackTricks en PDF** ¡Consulta los [**PLANES DE SUSCRIPCIÓN**](https://github.com/sponsors/carlospolop)!
* Obtén el [**swag oficial de PEASS & HackTricks**](https://peass.creator-spring.com)
* Descubre [**La Familia PEASS**](https://opensea.io/collection/the-peass-family), nuestra colección exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Únete al** 💬 [**grupo de Discord**](https://discord.gg/hRep4RUj7f) o al [**grupo de telegram**](https://t.me/peass) o **síguenos** en **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Comparte tus trucos de hacking enviando PRs a los** [**HackTricks**](https://github.com/carlospolop/hacktricks) y [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repositorios de github.

</details>
