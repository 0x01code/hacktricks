Si en algún momento encuentras una **página web que te presenta información sensible basada en tu sesión**: puede que esté reflejando cookies, o imprimiendo detalles de tarjetas de crédito o cualquier otra información sensible, puedes intentar robarla.\
Aquí te presento las principales formas en que puedes intentar lograrlo:

* [**Bypass de CORS**](pentesting-web/cors-bypass.md): Si puedes saltarte las cabeceras CORS, podrás robar la información realizando una solicitud Ajax desde una página maliciosa.
* [**XSS**](pentesting-web/xss-cross-site-scripting/): Si encuentras una vulnerabilidad XSS en la página, es posible que puedas abusar de ella para robar la información.
* [**Dangling Markup**](pentesting-web/dangling-markup-html-scriptless-injection.md): Si no puedes inyectar etiquetas XSS, aún puedes robar la información utilizando otras etiquetas HTML regulares.
* [**Clickjaking**](pentesting-web/clickjacking.md): Si no hay protección contra este ataque, es posible que puedas engañar al usuario para que te envíe los datos sensibles (un ejemplo [aquí](https://medium.com/bugbountywriteup/apache-example-servlet-leads-to-61a2720cac20)).


<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- ¿Trabajas en una **empresa de ciberseguridad**? ¿Quieres ver tu **empresa anunciada en HackTricks**? ¿O quieres tener acceso a la **última versión de PEASS o descargar HackTricks en PDF**? ¡Consulta los [**PLANES DE SUSCRIPCIÓN**](https://github.com/sponsors/carlospolop)!

- Descubre [**The PEASS Family**](https://opensea.io/collection/the-peass-family), nuestra colección exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)

- Obtén la [**oficial PEASS & HackTricks swag**](https://peass.creator-spring.com)

- **Únete al** [**💬**](https://emojipedia.org/speech-balloon/) **grupo de Discord** o al [**grupo de telegram**](https://t.me/peass) o **sígueme en** **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **Comparte tus trucos de hacking enviando PR al [repositorio de hacktricks](https://github.com/carlospolop/hacktricks) y al [repositorio de hacktricks-cloud](https://github.com/carlospolop/hacktricks-cloud)**.

</details>
