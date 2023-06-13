# Verificación de Conexión de XPC en macOS

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* ¿Trabajas en una **empresa de ciberseguridad**? ¿Quieres ver tu **empresa anunciada en HackTricks**? ¿O quieres tener acceso a la **última versión de PEASS o descargar HackTricks en PDF**? ¡Consulta los [**PLANES DE SUSCRIPCIÓN**](https://github.com/sponsors/carlospolop)!
* Descubre [**The PEASS Family**](https://opensea.io/collection/the-peass-family), nuestra colección exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* Obtén el [**swag oficial de PEASS y HackTricks**](https://peass.creator-spring.com)
* **Únete al** [**💬**](https://emojipedia.org/speech-balloon/) [**grupo de Discord**](https://discord.gg/hRep4RUj7f) o al [**grupo de telegram**](https://t.me/peass) o **sígueme** en **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Comparte tus trucos de hacking enviando PR al** [**repositorio de hacktricks**](https://github.com/carlospolop/hacktricks) **y al** [**repositorio de hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>

## Verificación de Conexión de XPC

Cuando se establece una conexión a un servicio XPC, el servidor verificará si la conexión está permitida. Estas son las verificaciones que normalmente realiza:

1. Verificar si el **proceso de conexión está firmado con un certificado firmado por Apple** (solo otorgado por Apple).
   * Si esto **no se verifica**, un atacante podría crear un **certificado falso** para coincidir con cualquier otra verificación.
2. Verificar si el proceso de conexión está firmado con el **certificado de la organización** (verificación de ID de equipo).
   * Si esto **no se verifica**, **cualquier certificado de desarrollador** de Apple se puede usar para firmar y conectarse al servicio.
3. Verificar si el proceso de conexión **contiene un ID de paquete adecuado**.
4. Verificar si el proceso de conexión tiene un **número de versión de software adecuado**.
   * Si esto **no se verifica**, se podría usar un cliente antiguo e inseguro, vulnerable a la inyección de procesos, para conectarse al servicio XPC incluso con las otras verificaciones en su lugar.
5. Verificar si el proceso de conexión tiene un **permiso** que le permita conectarse al servicio. Esto es aplicable para binarios de Apple.
6. La **verificación** debe estar **basada** en el **token de auditoría del cliente conectado** en lugar de su **ID de proceso (PID)**, ya que lo primero evita los ataques de reutilización de PID.
   * Los desarrolladores rara vez usan la llamada de API de token de auditoría ya que es **privada**, por lo que Apple podría **cambiarla** en cualquier momento. Además, el uso de API privadas no está permitido en las aplicaciones de la Mac App Store.

Para obtener más información sobre la verificación de ataques de reutilización de PID:

{% content-ref url="macos-pid-reuse.md" %}
[macos-pid-reuse.md](macos-pid-reuse.md)
{% endcontent-ref %}

### Trustcache - Prevención de Ataques de Degradación

Trustcache es un método defensivo introducido en las máquinas Apple Silicon que almacena una base de datos de CDHSAH de binarios de Apple para que solo se puedan ejecutar binarios no modificados permitidos. Lo que evita la ejecución de versiones anteriores.

### Ejemplos de Código

El servidor implementará esta **verificación** en una función llamada **`shouldAcceptNewConnection`**.

{% code overflow="wrap" %}
```objectivec
- (BOOL)listener:(NSXPCListener *)listener shouldAcceptNewConnection:(NSXPCConnection *)newConnection {
    //Check connection
    return YES;
}
```
{% endcode %}

El objeto NSXPCConnection tiene una propiedad **privada** llamada **`auditToken`** (la que debería ser utilizada pero podría cambiar) y una propiedad **pública** llamada **`processIdentifier`** (la que no debería ser utilizada).

El proceso de conexión podría ser verificado con algo como: 

{% code overflow="wrap" %}
```objectivec
[...]
SecRequirementRef requirementRef = NULL;
NSString requirementString = @"anchor apple generic and identifier \"xyz.hacktricks.service\" and certificate leaf [subject.CN] = \"TEAMID\" and info [CFBundleShortVersionString] >= \"1.0\"";
/* Check:
- Signed by a cert signed by Apple
- Check the bundle ID
- Check the TEAMID of the signing cert
- Check the version used
*/

// Check the requirements
SecRequirementCreateWithString(requirementString, kSecCSDefaultFlags, &requirementRef);
SecCodeCheckValidity(code, kSecCSDefaultFlags, requirementRef);
```
Si un desarrollador no quiere verificar la versión del cliente, al menos podría verificar que el cliente no sea vulnerable a la inyección de procesos: 

{% code overflow="wrap" %}
```objectivec
[...]
CFDictionaryRef csInfo = NULL;
SecCodeCopySigningInformation(code, kSecCSDynamicInformation, &csInfo);
uint32_t csFlags = [((__bridge NSDictionary *)csInfo)[(__bridge NSString *)kSecCodeInfoStatus] intValue];
const uint32_t cs_hard = 0x100;        // don't load invalid page. 
const uint32_t cs_kill = 0x200;        // Kill process if page is invalid
const uint32_t cs_restrict = 0x800;    // Prevent debugging
const uint32_t cs_require_lv = 0x2000; // Library Validation
const uint32_t cs_runtime = 0x10000;   // hardened runtime
if ((csFlags & (cs_hard | cs_require_lv)) {
    return Yes; // Accept connection
}
```
{% endcode %}

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* ¿Trabajas en una **empresa de ciberseguridad**? ¿Quieres ver tu **empresa anunciada en HackTricks**? ¿O quieres tener acceso a la **última versión de PEASS o descargar HackTricks en PDF**? ¡Revisa los [**PLANES DE SUSCRIPCIÓN**](https://github.com/sponsors/carlospolop)!
* Descubre [**The PEASS Family**](https://opensea.io/collection/the-peass-family), nuestra colección exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* Obtén el [**swag oficial de PEASS y HackTricks**](https://peass.creator-spring.com)
* **Únete al** [**💬**](https://emojipedia.org/speech-balloon/) **grupo de Discord** o al [**grupo de telegram**](https://t.me/peass) o **sígueme en** **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Comparte tus trucos de hacking enviando PRs al** [**repositorio de hacktricks**](https://github.com/carlospolop/hacktricks) **y al** [**repositorio de hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
