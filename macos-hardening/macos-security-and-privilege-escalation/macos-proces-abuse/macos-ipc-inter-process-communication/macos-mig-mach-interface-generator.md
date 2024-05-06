# macOS MIG - Mach Interface Generator

<details>

<summary><strong>Erlernen Sie AWS-Hacking von Grund auf mit</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Andere Möglichkeiten, HackTricks zu unterstützen:

* Wenn Sie Ihr **Unternehmen in HackTricks beworben sehen möchten** oder **HackTricks als PDF herunterladen möchten**, überprüfen Sie die [**ABONNEMENTPLÄNE**](https://github.com/sponsors/carlospolop)!
* Holen Sie sich das [**offizielle PEASS & HackTricks-Merchandise**](https://peass.creator-spring.com)
* Entdecken Sie [**The PEASS Family**](https://opensea.io/collection/the-peass-family), unsere Sammlung exklusiver [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Treten Sie der** 💬 [**Discord-Gruppe**](https://discord.gg/hRep4RUj7f) oder der [**Telegram-Gruppe**](https://t.me/peass) bei oder **folgen** Sie uns auf **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Teilen Sie Ihre Hacking-Tricks, indem Sie PRs an die** [**HackTricks**](https://github.com/carlospolop/hacktricks) und [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) GitHub-Repositories senden.

</details>

## Grundlegende Informationen

MIG wurde erstellt, um den Prozess der Mach IPC-Codeerstellung zu **vereinfachen**. Es generiert im Wesentlichen den benötigten Code, damit Server und Client gemäß einer bestimmten Definition kommunizieren können. Selbst wenn der generierte Code hässlich ist, muss ein Entwickler ihn nur importieren und sein Code wird viel einfacher sein als zuvor.

Die Definition wird in der Interface Definition Language (IDL) unter Verwendung der Erweiterung `.defs` angegeben.

Diese Definitionen haben 5 Abschnitte:

* **Subsystem-Deklaration**: Das Schlüsselwort `subsystem` wird verwendet, um den **Namen** und die **ID** anzugeben. Es ist auch möglich, es als **`KernelServer`** zu kennzeichnen, wenn der Server im Kernel ausgeführt werden soll.
* **Einschlüsse und Importe**: MIG verwendet den C-Präprozessor, sodass Importe verwendet werden können. Außerdem ist es möglich, `uimport` und `simport` für benutzer- oder servergenerierten Code zu verwenden.
* **Typdeklarationen**: Es ist möglich, Datentypen zu definieren, obwohl normalerweise `mach_types.defs` und `std_types.defs` importiert werden. Für benutzerdefinierte Typen kann eine bestimmte Syntax verwendet werden:
* \[i`n/out]tran`: Funktion, die von einer eingehenden oder zu einer ausgehenden Nachricht übersetzt werden muss
* `c[user/server]type`: Zuordnung zu einem anderen C-Typ.
* `destructor`: Diese Funktion aufrufen, wenn der Typ freigegeben wird.
* **Operationen**: Dies sind die Definitionen der RPC-Methoden. Es gibt 5 verschiedene Typen:
* `routine`: Erwartet Antwort
* `simpleroutine`: Erwartet keine Antwort
* `procedure`: Erwartet Antwort
* `simpleprocedure`: Erwartet keine Antwort
* `function`: Erwartet Antwort

### Beispiel

Erstellen Sie eine Definitiondatei, in diesem Fall mit einer sehr einfachen Funktion:

{% code title="myipc.defs" %}
```cpp
subsystem myipc 500; // Arbitrary name and id

userprefix USERPREF;        // Prefix for created functions in the client
serverprefix SERVERPREF;    // Prefix for created functions in the server

#include <mach/mach_types.defs>
#include <mach/std_types.defs>

simpleroutine Subtract(
server_port :  mach_port_t;
n1          :  uint32_t;
n2          :  uint32_t);
```
{% endcode %}

Beachten Sie, dass das erste **Argument der Port ist, an den gebunden wird** und MIG wird **automatisch den Antwortport verwalten** (es sei denn, `mig_get_reply_port()` wird im Clientcode aufgerufen). Darüber hinaus werden die **IDs der Operationen** **sequenziell** sein, beginnend mit der angegebenen Subsystem-ID (sofern eine Operation veraltet ist, wird sie gelöscht und `skip` wird verwendet, um ihre ID weiterhin zu verwenden).

Verwenden Sie nun MIG, um den Server- und Clientcode zu generieren, die in der Lage sein werden, miteinander zu kommunizieren, um die Subtract-Funktion aufzurufen:
```bash
mig -header myipcUser.h -sheader myipcServer.h myipc.defs
```
Es werden mehrere neue Dateien im aktuellen Verzeichnis erstellt.

{% hint style="success" %}
Ein komplexeres Beispiel finden Sie in Ihrem System mit: `mdfind mach_port.defs`\
Und Sie können es aus dem gleichen Ordner wie die Datei kompilieren mit: `mig -DLIBSYSCALL_INTERFACE mach_ports.defs`
{% endhint %}

In den Dateien **`myipcServer.c`** und **`myipcServer.h`** finden Sie die Deklaration und Definition der Struktur **`SERVERPREFmyipc_subsystem`**, die im Wesentlichen die Funktion definiert, die basierend auf der empfangenen Nachrichten-ID aufgerufen werden soll (wir haben eine Startnummer von 500 angegeben):

{% tabs %}
{% tab title="myipcServer.c" %}
```c
/* Description of this subsystem, for use in direct RPC */
const struct SERVERPREFmyipc_subsystem SERVERPREFmyipc_subsystem = {
myipc_server_routine,
500, // start ID
501, // end ID
(mach_msg_size_t)sizeof(union __ReplyUnion__SERVERPREFmyipc_subsystem),
(vm_address_t)0,
{
{ (mig_impl_routine_t) 0,
// Function to call
(mig_stub_routine_t) _XSubtract, 3, 0, (routine_arg_descriptor_t)0, (mach_msg_size_t)sizeof(__Reply__Subtract_t)},
}
};
```
{% endtab %}

{% tab title="myipcServer.h" %} 

### macOS MIG (Mach Interface Generator)

MIG (Mach Interface Generator) is a tool used to define inter-process communication on macOS systems. It generates client and server-side code for message-based IPC. By understanding and manipulating MIG interfaces, an attacker can abuse inter-process communication mechanisms to escalate privileges or perform other malicious actions.

#### Example:

```c
#include <mach/mach.h>
#include <servers/bootstrap.h>
#include "myipcServer.h"

kern_return_t myipc_server(mach_msg_header_t *InHeadP, mach_msg_header_t *OutHeadP);
```

In the example above, `myipc_server` is a MIG server function that processes incoming messages. Understanding how MIG-generated code works is crucial for analyzing and exploiting macOS IPC mechanisms. 

### macOS MIG (Mach Interface Generator)

MIG (Mach Interface Generator) ist ein Tool, das zur Definition der Interprozesskommunikation auf macOS-Systemen verwendet wird. Es generiert Client- und Server-seitigen Code für nachrichtenbasierte IPC. Indem ein Angreifer MIG-Schnittstellen versteht und manipuliert, kann er Interprozesskommunikationsmechanismen missbrauchen, um Berechtigungen zu eskalieren oder andere bösartige Aktionen auszuführen.

#### Beispiel:

```c
#include <mach/mach.h>
#include <servers/bootstrap.h>
#include "myipcServer.h"

kern_return_t myipc_server(mach_msg_header_t *InHeadP, mach_msg_header_t *OutHeadP);
```

Im obigen Beispiel ist `myipc_server` eine MIG-Serverfunktion, die eingehende Nachrichten verarbeitet. Das Verständnis, wie der von MIG generierte Code funktioniert, ist entscheidend für die Analyse und Ausnutzung von macOS-IPC-Mechanismen. 

{% endtab %}
```c
/* Description of this subsystem, for use in direct RPC */
extern const struct SERVERPREFmyipc_subsystem {
mig_server_routine_t	server;	/* Server routine */
mach_msg_id_t	start;	/* Min routine number */
mach_msg_id_t	end;	/* Max routine number + 1 */
unsigned int	maxsize;	/* Max msg size */
vm_address_t	reserved;	/* Reserved */
struct routine_descriptor	/* Array of routine descriptors */
routine[1];
} SERVERPREFmyipc_subsystem;
```
{% endtab %}
{% endtabs %}

Basierend auf der vorherigen Struktur wird die Funktion **`myipc_server_routine`** die **Nachrichten-ID** erhalten und die entsprechende Funktion zurückgeben, die aufgerufen werden soll:
```c
mig_external mig_routine_t myipc_server_routine
(mach_msg_header_t *InHeadP)
{
int msgh_id;

msgh_id = InHeadP->msgh_id - 500;

if ((msgh_id > 0) || (msgh_id < 0))
return 0;

return SERVERPREFmyipc_subsystem.routine[msgh_id].stub_routine;
}
```
In diesem Beispiel haben wir nur 1 Funktion in den Definitionen definiert, aber wenn wir mehr Funktionen definiert hätten, wären sie innerhalb des Arrays von **`SERVERPREFmyipc_subsystem`** und die erste Funktion wäre der ID **500** zugewiesen worden, die zweite der ID **501**...

Wenn erwartet wurde, dass die Funktion eine **Antwort** sendet, würde auch die Funktion `mig_internal kern_return_t __MIG_check__Reply__<name>` existieren.

Tatsächlich ist es möglich, diese Beziehung in der Struktur **`subsystem_to_name_map_myipc`** aus **`myipcServer.h`** (**`subsystem_to_name_map_***`** in anderen Dateien) zu identifizieren:
```c
#ifndef subsystem_to_name_map_myipc
#define subsystem_to_name_map_myipc \
{ "Subtract", 500 }
#endif
```
Schließlich wird eine weitere wichtige Funktion zur Funktionsweise des Servers **`myipc_server`** sein, die tatsächlich die mit der empfangenen ID verbundene Funktion aufrufen wird:

<pre class="language-c"><code class="lang-c">mig_external boolean_t myipc_server
(mach_msg_header_t *InHeadP, mach_msg_header_t *OutHeadP)
{
/*
* typedef struct {
* 	mach_msg_header_t Head;
* 	NDR_record_t NDR;
* 	kern_return_t RetCode;
* } mig_reply_error_t;
*/

mig_routine_t routine;

OutHeadP->msgh_bits = MACH_MSGH_BITS(MACH_MSGH_BITS_REPLY(InHeadP->msgh_bits), 0);
OutHeadP->msgh_remote_port = InHeadP->msgh_reply_port;
/* Mindestgröße: routine() wird sie aktualisieren, wenn sie sich unterscheidet */
OutHeadP->msgh_size = (mach_msg_size_t)sizeof(mig_reply_error_t);
OutHeadP->msgh_local_port = MACH_PORT_NULL;
OutHeadP->msgh_id = InHeadP->msgh_id + 100;
OutHeadP->msgh_reserved = 0;

if ((InHeadP->msgh_id > 500) || (InHeadP->msgh_id &#x3C; 500) ||
<strong>	    ((routine = SERVERPREFmyipc_subsystem.routine[InHeadP->msgh_id - 500].stub_routine) == 0)) {
</strong>		((mig_reply_error_t *)OutHeadP)->NDR = NDR_record;
((mig_reply_error_t *)OutHeadP)->RetCode = MIG_BAD_ID;
return FALSE;
}
<strong>	(*routine) (InHeadP, OutHeadP);
</strong>	return TRUE;
}
</code></pre>

Überprüfen Sie die zuvor hervorgehobenen Zeilen, die auf die Funktion zugreifen, die anhand der ID aufgerufen werden soll.

Der folgende Code erstellt einen einfachen **Server** und **Client**, bei dem der Client die Funktionen des Servers subtrahieren kann:

{% tabs %}
{% tab title="myipc_server.c" %}
```c
// gcc myipc_server.c myipcServer.c -o myipc_server

#include <stdio.h>
#include <mach/mach.h>
#include <servers/bootstrap.h>
#include "myipcServer.h"

kern_return_t SERVERPREFSubtract(mach_port_t server_port, uint32_t n1, uint32_t n2)
{
printf("Received: %d - %d = %d\n", n1, n2, n1 - n2);
return KERN_SUCCESS;
}

int main() {

mach_port_t port;
kern_return_t kr;

// Register the mach service
kr = bootstrap_check_in(bootstrap_port, "xyz.hacktricks.mig", &port);
if (kr != KERN_SUCCESS) {
printf("bootstrap_check_in() failed with code 0x%x\n", kr);
return 1;
}

// myipc_server is the function that handles incoming messages (check previous exlpanation)
mach_msg_server(myipc_server, sizeof(union __RequestUnion__SERVERPREFmyipc_subsystem), port, MACH_MSG_TIMEOUT_NONE);
}
```
{% endtab %}

{% tab title="myipc_client.c" %} 

### macOS IPC - Inter-Process Communication

#### macOS MIG - Mach Interface Generator

MIG is a tool used to define inter-process communication on macOS systems. It generates client-side and server-side code for message-based communication between processes. By defining the messages and data structures in a .defs file, MIG creates the necessary C code for handling these messages. This allows processes to communicate with each other securely and efficiently.

To use MIG, you need to define the messages and data structures in a .defs file and run the MIG compiler to generate the C code. The generated code can then be included in your client and server applications to enable inter-process communication using the defined message formats.

MIG simplifies the process of implementing inter-process communication on macOS systems by handling the low-level details of message passing and data serialization. It abstracts the complexity of the Mach messaging system, making it easier for developers to create secure communication channels between processes.

By leveraging MIG for inter-process communication, developers can ensure that their applications can securely exchange data and messages with other processes running on the same system. This is essential for building robust and secure macOS applications that interact with each other seamlessly. 

{% endtab %}
```c
// gcc myipc_client.c myipcUser.c -o myipc_client

#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

#include <mach/mach.h>
#include <servers/bootstrap.h>
#include "myipcUser.h"

int main() {

// Lookup the receiver port using the bootstrap server.
mach_port_t port;
kern_return_t kr = bootstrap_look_up(bootstrap_port, "xyz.hacktricks.mig", &port);
if (kr != KERN_SUCCESS) {
printf("bootstrap_look_up() failed with code 0x%x\n", kr);
return 1;
}
printf("Port right name %d\n", port);
USERPREFSubtract(port, 40, 2);
}
```
{% endtab %}
{% endtabs %}

### Der NDR\_record

Der NDR\_record wird von `libsystem_kernel.dylib` exportiert und ist eine Struktur, die es MIG ermöglicht, Daten so zu transformieren, dass sie unabhängig vom System sind, auf dem sie verwendet werden, da MIG dafür gedacht war, zwischen verschiedenen Systemen verwendet zu werden (und nicht nur auf derselben Maschine).

Dies ist interessant, weil wenn `_NDR_record` als Abhängigkeit in einem Binärdatei gefunden wird (`jtool2 -S <binary> | grep NDR` oder `nm`), bedeutet dies, dass die Binärdatei ein MIG-Client oder -Server ist.

Darüber hinaus haben **MIG-Server** die Dispatch-Tabelle in `__DATA.__const` (oder in `__CONST.__constdata` im macOS-Kernel und `__DATA_CONST.__const` in anderen \*OS-Kerneln). Dies kann mit **`jtool2`** ausgelesen werden.

Und **MIG-Clients** verwenden den `__NDR_record`, um mit `__mach_msg` an die Server zu senden.

## Binäranalyse

### jtool

Da viele Binärdateien jetzt MIG verwenden, um Mach-Ports freizulegen, ist es interessant zu wissen, wie man **identifiziert, dass MIG verwendet wurde** und die **Funktionen, die MIG mit jeder Nachrichten-ID ausführt**.

[**jtool2**](../../macos-apps-inspecting-debugging-and-fuzzing/#jtool2) kann MIG-Informationen aus einer Mach-O-Binärdatei analysieren, indem es die Nachrichten-ID angibt und die auszuführende Funktion identifiziert:
```bash
jtool2 -d __DATA.__const myipc_server | grep MIG
```
Darüber hinaus sind MIG-Funktionen nur Wrapper der tatsächlichen aufgerufenen Funktion, was bedeutet, dass Sie durch Disassemblierung und Grepping nach BL möglicherweise die tatsächliche aufgerufene Funktion finden können:
```bash
jtool2 -d __DATA.__const myipc_server | grep BL
```
### Assembly

Es wurde zuvor erwähnt, dass die Funktion, die sich um **den Aufruf der richtigen Funktion je nach empfangener Nachrichten-ID kümmern wird**, `myipc_server` war. Sie haben jedoch normalerweise nicht die Symbole der Binärdatei (keine Funktionsnamen), daher ist es interessant zu **überprüfen, wie sie dekompiliert aussieht**, da sie immer sehr ähnlich sein wird (der Code dieser Funktion ist unabhängig von den freigegebenen Funktionen):

{% tabs %}
{% tab title="myipc_server dekompiliert 1" %}
<pre class="language-c"><code class="lang-c">int _myipc_server(int arg0, int arg1) {
var_10 = arg0;
var_18 = arg1;
// Initiale Anweisungen zum Finden der richtigen Funktionszeiger
*(int32_t *)var_18 = *(int32_t *)var_10 &#x26; 0x1f;
*(int32_t *)(var_18 + 0x8) = *(int32_t *)(var_10 + 0x8);
*(int32_t *)(var_18 + 0x4) = 0x24;
*(int32_t *)(var_18 + 0xc) = 0x0;
*(int32_t *)(var_18 + 0x14) = *(int32_t *)(var_10 + 0x14) + 0x64;
*(int32_t *)(var_18 + 0x10) = 0x0;
if (*(int32_t *)(var_10 + 0x14) &#x3C;= 0x1f4 &#x26;&#x26; *(int32_t *)(var_10 + 0x14) >= 0x1f4) {
rax = *(int32_t *)(var_10 + 0x14);
// Aufruf von sign_extend_64, der helfen kann, diese Funktion zu identifizieren
// Dies speichert in rax den Zeiger auf den Aufruf, der aufgerufen werden muss
// Überprüfen Sie die Verwendung der Adresse 0x100004040 (Funktionsadressenarray)
// 0x1f4 = 500 (die Start-ID)
<strong>            rax = *(sign_extend_64(rax - 0x1f4) * 0x28 + 0x100004040);
</strong>            var_20 = rax;
// If - else, das if gibt false zurück, während das else die richtige Funktion aufruft und true zurückgibt
<strong>            if (rax == 0x0) {
</strong>                    *(var_18 + 0x18) = **_NDR_record;
*(int32_t *)(var_18 + 0x20) = 0xfffffffffffffed1;
var_4 = 0x0;
}
else {
// Berechnete Adresse, die die richtige Funktion mit 2 Argumenten aufruft
<strong>                    (var_20)(var_10, var_18);
</strong>                    var_4 = 0x1;
}
}
else {
*(var_18 + 0x18) = **_NDR_record;
*(int32_t *)(var_18 + 0x20) = 0xfffffffffffffed1;
var_4 = 0x0;
}
rax = var_4;
return rax;
}
</code></pre>
{% endtab %}

{% tab title="myipc_server dekompiliert 2" %}
Dies ist dieselbe Funktion dekompiliert in einer anderen Hopper Free-Version:

<pre class="language-c"><code class="lang-c">int _myipc_server(int arg0, int arg1) {
r31 = r31 - 0x40;
saved_fp = r29;
stack[-8] = r30;
var_10 = arg0;
var_18 = arg1;
// Initiale Anweisungen zum Finden der richtigen Funktionszeiger
*(int32_t *)var_18 = *(int32_t *)var_10 &#x26; 0x1f | 0x0;
*(int32_t *)(var_18 + 0x8) = *(int32_t *)(var_10 + 0x8);
*(int32_t *)(var_18 + 0x4) = 0x24;
*(int32_t *)(var_18 + 0xc) = 0x0;
*(int32_t *)(var_18 + 0x14) = *(int32_t *)(var_10 + 0x14) + 0x64;
*(int32_t *)(var_18 + 0x10) = 0x0;
r8 = *(int32_t *)(var_10 + 0x14);
r8 = r8 - 0x1f4;
if (r8 > 0x0) {
if (CPU_FLAGS &#x26; G) {
r8 = 0x1;
}
}
if ((r8 &#x26; 0x1) == 0x0) {
r8 = *(int32_t *)(var_10 + 0x14);
r8 = r8 - 0x1f4;
if (r8 &#x3C; 0x0) {
if (CPU_FLAGS &#x26; L) {
r8 = 0x1;
}
}
if ((r8 &#x26; 0x1) == 0x0) {
r8 = *(int32_t *)(var_10 + 0x14);
// 0x1f4 = 500 (die Start-ID)
<strong>                    r8 = r8 - 0x1f4;
</strong>                    asm { smaddl     x8, w8, w9, x10 };
r8 = *(r8 + 0x8);
var_20 = r8;
r8 = r8 - 0x0;
if (r8 != 0x0) {
if (CPU_FLAGS &#x26; NE) {
r8 = 0x1;
}
}
// Dasselbe If-Else wie in der vorherigen Version
// Überprüfen Sie die Verwendung der Adresse 0x100004040 (Funktionsadressenarray)
<strong>                    if ((r8 &#x26; 0x1) == 0x0) {
</strong><strong>                            *(var_18 + 0x18) = **0x100004000;
</strong>                            *(int32_t *)(var_18 + 0x20) = 0xfffffed1;
var_4 = 0x0;
}
else {
// Aufruf an die berechnete Adresse, an der die Funktion sein sollte
<strong>                            (var_20)(var_10, var_18);
</strong>                            var_4 = 0x1;
}
}
else {
*(var_18 + 0x18) = **0x100004000;
*(int32_t *)(var_18 + 0x20) = 0xfffffed1;
var_4 = 0x0;
}
}
else {
*(var_18 + 0x18) = **0x100004000;
*(int32_t *)(var_18 + 0x20) = 0xfffffed1;
var_4 = 0x0;
}
r0 = var_4;
return r0;
}

</code></pre>
{% endtab %}
{% endtabs %}

Tatsächlich finden Sie im Funktionsaufruf **`0x100004000`** das Array der **`routine_descriptor`**-Strukturen. Das erste Element der Struktur ist die **Adresse**, an der die **Funktion** implementiert ist, und die **Struktur nimmt 0x28 Bytes ein**, sodass Sie alle 0x28 Bytes (beginnend ab Byte 0) 8 Bytes erhalten und das wird die **Adresse der Funktion** sein, die aufgerufen wird:

<figure><img src="../../../../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (36).png" alt=""><figcaption></figcaption></figure>

Diese Daten können [**mit diesem Hopper-Skript**](https://github.com/knightsc/hopper/blob/master/scripts/MIG%20Detect.py) extrahiert werden.
### Debug

Der von MIG generierte Code ruft auch `kernel_debug` auf, um Protokolle über Operationen beim Eintritt und Austritt zu generieren. Es ist möglich, sie mit **`trace`** oder **`kdv`** zu überprüfen: `kdv all | grep MIG`

## Referenzen

* [\*OS Internals, Band I, Benutzermodus, Jonathan Levin](https://www.amazon.com/MacOS-iOS-Internals-User-Mode/dp/099105556X)

<details>

<summary><strong>Erlernen Sie AWS-Hacking von Null auf Heldenniveau mit</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Andere Möglichkeiten, HackTricks zu unterstützen:

* Wenn Sie Ihr **Unternehmen in HackTricks beworben sehen möchten** oder **HackTricks im PDF-Format herunterladen möchten**, überprüfen Sie die [**ABONNEMENTPLÄNE**](https://github.com/sponsors/carlospolop)!
* Holen Sie sich das [**offizielle PEASS & HackTricks-Merch**](https://peass.creator-spring.com)
* Entdecken Sie [**The PEASS Family**](https://opensea.io/collection/the-peass-family), unsere Sammlung exklusiver [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Treten Sie der** 💬 [**Discord-Gruppe**](https://discord.gg/hRep4RUj7f) oder der [**Telegram-Gruppe**](https://t.me/peass) bei oder **folgen** Sie uns auf **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Teilen Sie Ihre Hacking-Tricks, indem Sie PRs an die** [**HackTricks**](https://github.com/carlospolop/hacktricks) und [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) GitHub-Repositories einreichen.

</details>
