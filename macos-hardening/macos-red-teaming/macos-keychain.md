# Llavero de macOS

{% hint style="success" %}
Aprende y practica Hacking en AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Aprende y practica Hacking en GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Apoya a HackTricks</summary>

* ¡Revisa los [**planes de suscripción**](https://github.com/sponsors/carlospolop)!
* **Únete al** 💬 [**grupo de Discord**](https://discord.gg/hRep4RUj7f) o al [**grupo de telegram**](https://t.me/peass) o **síguenos** en **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Comparte trucos de hacking enviando PRs a los repositorios de** [**HackTricks**](https://github.com/carlospolop/hacktricks) y [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
{% endhint %}

### [WhiteIntel](https://whiteintel.io)

<figure><img src="../../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) es un motor de búsqueda alimentado por la **dark web** que ofrece funcionalidades **gratuitas** para verificar si una empresa o sus clientes han sido **comprometidos** por **malwares robadores**.

El objetivo principal de WhiteIntel es combatir tomas de cuentas y ataques de ransomware resultantes de malwares que roban información.

Puedes visitar su sitio web y probar su motor de forma **gratuita** en:

{% embed url="https://whiteintel.io" %}

***

## Llaveros Principales

* El **Llavero de Usuario** (`~/Library/Keychains/login.keycahin-db`), que se utiliza para almacenar **credenciales específicas del usuario** como contraseñas de aplicaciones, contraseñas de internet, certificados generados por el usuario, contraseñas de red y claves públicas/privadas generadas por el usuario.
* El **Llavero del Sistema** (`/Library/Keychains/System.keychain`), que almacena **credenciales de todo el sistema** como contraseñas de WiFi, certificados raíz del sistema, claves privadas del sistema y contraseñas de aplicaciones del sistema.

### Acceso al Llavero de Contraseñas

Estos archivos, aunque no tienen protección inherente y pueden ser **descargados**, están encriptados y requieren la **contraseña en texto plano del usuario para ser desencriptados**. Una herramienta como [**Chainbreaker**](https://github.com/n0fate/chainbreaker) podría ser utilizada para la desencriptación.

## Protecciones de las Entradas del Llavero

### Listas de Control de Acceso (ACLs)

Cada entrada en el llavero está gobernada por **Listas de Control de Acceso (ACLs)** que dictan quién puede realizar varias acciones en la entrada del llavero, incluyendo:

* **ACLAuhtorizationExportClear**: Permite al titular obtener el texto claro del secreto.
* **ACLAuhtorizationExportWrapped**: Permite al titular obtener el texto claro encriptado con otra contraseña proporcionada.
* **ACLAuhtorizationAny**: Permite al titular realizar cualquier acción.

Las ACLs están acompañadas por una **lista de aplicaciones de confianza** que pueden realizar estas acciones sin solicitar permiso. Esto podría ser:

* **N`il`** (no se requiere autorización, **todos son de confianza**)
* Una lista **vacía** (nadie es de confianza)
* **Lista** de **aplicaciones** específicas.

Además, la entrada podría contener la clave **`ACLAuthorizationPartitionID`,** que se utiliza para identificar el **teamid, apple,** y **cdhash.**

* Si se especifica el **teamid**, entonces para **acceder al valor de la entrada** sin una **solicitud**, la aplicación utilizada debe tener el **mismo teamid**.
* Si se especifica el **apple**, entonces la aplicación debe estar **firmada** por **Apple**.
* Si se indica el **cdhash**, entonces la aplicación debe tener el **cdhash** específico.

### Creación de una Entrada en el Llavero

Cuando se crea una **nueva** **entrada** utilizando **`Keychain Access.app`**, se aplican las siguientes reglas:

* Todas las aplicaciones pueden encriptar.
* **Ninguna aplicación** puede exportar/desencriptar (sin solicitar al usuario).
* Todas las aplicaciones pueden ver la comprobación de integridad.
* Ninguna aplicación puede cambiar las ACLs.
* El **partitionID** se establece en **`apple`**.

Cuando una **aplicación crea una entrada en el llavero**, las reglas son ligeramente diferentes:

* Todas las aplicaciones pueden encriptar.
* Solo la **aplicación creadora** (o cualquier otra aplicación añadida explícitamente) puede exportar/desencriptar (sin solicitar al usuario).
* Todas las aplicaciones pueden ver la comprobación de integridad.
* Ninguna aplicación puede cambiar las ACLs.
* El **partitionID** se establece en **`teamid:[teamID aquí]`**.

## Accediendo al Llavero

### `security`
```bash
# Dump all metadata and decrypted secrets (a lot of pop-ups)
security dump-keychain -a -d

# Find generic password for the "Slack" account and print the secrets
security find-generic-password -a "Slack" -g

# Change the specified entrys PartitionID entry
security set-generic-password-parition-list -s "test service" -a "test acount" -S
```
### APIs

{% hint style="success" %}
La **enumeración y volcado de contraseñas** del llavero que **no generará un aviso** se puede hacer con la herramienta [**LockSmith**](https://github.com/its-a-feature/LockSmith)
{% endhint %}

Listar y obtener **información** sobre cada entrada del llavero:

* La API **`SecItemCopyMatching`** proporciona información sobre cada entrada y hay algunos atributos que se pueden configurar al usarla:
* **`kSecReturnData`**: Si es verdadero, intentará descifrar los datos (establecer en falso para evitar posibles ventanas emergentes)
* **`kSecReturnRef`**: Obtener también la referencia al elemento del llavero (establecer en verdadero en caso de que luego veas que puedes descifrar sin ventana emergente)
* **`kSecReturnAttributes`**: Obtener metadatos sobre las entradas
* **`kSecMatchLimit`**: Cuántos resultados devolver
* **`kSecClass`**: Qué tipo de entrada del llavero

Obtener **ACLs** de cada entrada:

* Con la API **`SecAccessCopyACLList`** puedes obtener el **ACL del elemento del llavero**, y devolverá una lista de ACLs (como `ACLAuhtorizationExportClear` y los otros mencionados anteriormente) donde cada lista tiene:
* Descripción
* **Lista de aplicaciones de confianza**. Esto podría ser:
* Una aplicación: /Applications/Slack.app
* Un binario: /usr/libexec/airportd
* Un grupo: group://AirPort

Exportar los datos:

* La API **`SecKeychainItemCopyContent`** obtiene el texto plano
* La API **`SecItemExport`** exporta las claves y certificados pero podría ser necesario establecer contraseñas para exportar el contenido cifrado

Y estos son los **requisitos** para poder **exportar un secreto sin un aviso**:

* Si hay **1 o más aplicaciones de confianza** listadas:
* Necesita las **autorizaciones** apropiadas (**`Nil`**, o ser **parte** de la lista permitida de aplicaciones en la autorización para acceder a la información secreta)
* Necesita que la firma del código coincida con **PartitionID**
* Necesita que la firma del código coincida con la de una **aplicación de confianza** (o ser miembro del grupo KeychainAccessGroup correcto)
* Si **todas las aplicaciones son de confianza**:
* Necesita las **autorizaciones** apropiadas
* Necesita que la firma del código coincida con **PartitionID**
* Si no hay **PartitionID**, entonces esto no es necesario

{% hint style="danger" %}
Por lo tanto, si hay **1 aplicación listada**, necesitas **inyectar código en esa aplicación**.

Si se indica **apple** en el **PartitionID**, podrías acceder con **`osascript`** a cualquier cosa que confíe en todas las aplicaciones con apple en el PartitionID. **`Python`** también podría ser usado para esto.
{% endhint %}

### Dos atributos adicionales

* **Invisible**: Es un indicador booleano para **ocultar** la entrada de la aplicación del llavero **UI**
* **General**: Es para almacenar **metadatos** (por lo que NO ESTÁ CIFRADO)
* Microsoft estaba almacenando en texto plano todos los tokens de actualización para acceder a puntos finales sensibles.

## Referencias

* [**#OBTS v5.0: "Lock Picking the macOS Keychain" - Cody Thomas**](https://www.youtube.com/watch?v=jKE1ZW33JpY)

### [WhiteIntel](https://whiteintel.io)

<figure><img src="../../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) es un motor de búsqueda alimentado por la **dark web** que ofrece funcionalidades **gratuitas** para verificar si una empresa o sus clientes han sido **comprometidos** por **malwares robadores**.

El objetivo principal de WhiteIntel es combatir las tomas de cuentas y los ataques de ransomware resultantes de malwares que roban información.

Puedes visitar su sitio web y probar su motor de búsqueda de forma **gratuita** en:

{% embed url="https://whiteintel.io" %}

{% hint style="success" %}
Aprende y practica Hacking en AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Aprende y practica Hacking en GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Apoya a HackTricks</summary>

* ¡Consulta los [**planes de suscripción**](https://github.com/sponsors/carlospolop)!
* **Únete al** 💬 [**grupo de Discord**](https://discord.gg/hRep4RUj7f) o al [**grupo de telegram**](https://t.me/peass) o **síguenos** en **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Comparte trucos de hacking enviando PRs a los repositorios de** [**HackTricks**](https://github.com/carlospolop/hacktricks) y [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
{% endhint %}
