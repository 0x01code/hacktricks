# Trucos con archivos ZIP

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* ¿Trabajas en una **empresa de ciberseguridad**? ¿Quieres ver tu **empresa anunciada en HackTricks**? ¿O quieres tener acceso a la **última versión de PEASS o descargar HackTricks en PDF**? ¡Consulta los [**PLANES DE SUSCRIPCIÓN**](https://github.com/sponsors/carlospolop)!
* Descubre [**The PEASS Family**](https://opensea.io/collection/the-peass-family), nuestra colección exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* Obtén el [**merchandising oficial de PEASS y HackTricks**](https://peass.creator-spring.com)
* **Únete al** [**💬**](https://emojipedia.org/speech-balloon/) [**grupo de Discord**](https://discord.gg/hRep4RUj7f) o al [**grupo de Telegram**](https://t.me/peass) o **sígueme** en **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Comparte tus trucos de hacking enviando PRs al** [**repositorio de hacktricks**](https://github.com/carlospolop/hacktricks) **y al** [**repositorio de hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>

Hay varias herramientas de línea de comandos para archivos ZIP que serán útiles conocer.

* `unzip` a menudo muestra información útil sobre por qué un archivo ZIP no se puede descomprimir.
* `zipdetails -v` proporciona información detallada sobre los valores presentes en los diferentes campos del formato.
* `zipinfo` lista información sobre el contenido del archivo ZIP sin extraerlo.
* `zip -F input.zip --out output.zip` y `zip -FF input.zip --out output.zip` intentan reparar un archivo ZIP corrupto.
* [fcrackzip](https://github.com/hyc/fcrackzip) realiza un ataque de fuerza bruta para adivinar la contraseña de un archivo ZIP (para contraseñas de menos de 7 caracteres aproximadamente).

[Especificación del formato de archivo ZIP](https://pkware.cachefly.net/webdocs/casestudies/APPNOTE.TXT)

Una nota importante relacionada con la seguridad sobre los archivos ZIP protegidos con contraseña es que no cifran los nombres de archivo y los tamaños de archivo originales de los archivos comprimidos que contienen, a diferencia de los archivos RAR o 7z protegidos con contraseña.

Otra nota sobre la ruptura de archivos ZIP es que si tienes una copia sin cifrar/descomprimida de cualquiera de los archivos que están comprimidos en el archivo ZIP cifrado, puedes realizar un "ataque de texto plano" y romper el archivo ZIP, como se detalla [aquí](https://www.hackthis.co.uk/articles/known-plaintext-attack-cracking-zip-files) y se explica en [este documento](https://www.cs.auckland.ac.nz/\~mike/zipattacks.pdf). El nuevo esquema para proteger con contraseña los archivos ZIP (con AES-256, en lugar de "ZipCrypto") no tiene esta debilidad.

De: [https://app.gitbook.com/@cpol/s/hacktricks/\~/edit/drafts/-LlM5mCby8ex5pOeV4pJ/forensics/basic-forensics-esp/zips-tricks](http://127.0.0.1:5000/s/-L\_2uGJGU7AVNRcqRvEi/)
