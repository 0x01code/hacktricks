# macOS MIG - Mach Interface Generator

<details>

<summary><strong>Erfahren Sie AWS-Hacking von Null auf Held mit</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Andere Möglichkeiten, HackTricks zu unterstützen:

* Wenn Sie Ihr **Unternehmen in HackTricks beworben sehen möchten** oder **HackTricks in PDF herunterladen möchten**, überprüfen Sie die [**ABONNEMENTPLÄNE**](https://github.com/sponsors/carlospolop)!
* Holen Sie sich das [**offizielle PEASS & HackTricks-Merchandise**](https://peass.creator-spring.com)
* Entdecken Sie [**The PEASS Family**](https://opensea.io/collection/the-peass-family), unsere Sammlung exklusiver [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Treten Sie der** 💬 [**Discord-Gruppe**](https://discord.gg/hRep4RUj7f) oder der [**Telegram-Gruppe**](https://t.me/peass) bei oder **folgen** Sie uns auf **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Teilen Sie Ihre Hacking-Tricks, indem Sie PRs an die** [**HackTricks**](https://github.com/carlospolop/hacktricks) und [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) GitHub-Repositories einreichen.

</details>

## Grundlegende Informationen

MIG wurde erstellt, um den Prozess der Mach IPC-Codeerstellung zu **vereinfachen**. Es generiert im Wesentlichen den benötigten Code, damit Server und Client gemäß einer bestimmten Definition kommunizieren können. Selbst wenn der generierte Code hässlich ist, muss ein Entwickler ihn nur importieren, und sein Code wird viel einfacher sein als zuvor.

Die Definition wird in der Interface Definition Language (IDL) unter Verwendung der Erweiterung `.defs` angegeben.

Diese Definitionen haben 5 Abschnitte:

* **Subsystemdeklaration**: Das Schlüsselwort `subsystem` wird verwendet, um den **Namen** und die **ID** anzugeben. Es ist auch möglich, es als **`KernelServer`** zu kennzeichnen, wenn der Server im Kernel ausgeführt werden soll.
* **Einschlüsse und Importe**: MIG verwendet den C-Präprozessor, sodass er Importe verwenden kann. Außerdem ist es möglich, `uimport` und `simport` für benutzer- oder servergenerierten Code zu verwenden.
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

Beachten Sie, dass das erste **Argument der Port ist, an den gebunden wird** und MIG wird **automatisch den Antwortport verwalten** (es sei denn, `mig_get_reply_port()` wird im Clientcode aufgerufen). Darüber hinaus werden die **IDs der Operationen** **sequenziell** sein und mit der angegebenen Subsystem-ID beginnen (sofern eine Operation veraltet ist, wird sie gelöscht und `skip` wird verwendet, um weiterhin ihre ID zu verwenden).

Verwenden Sie nun MIG, um den Server- und Clientcode zu generieren, die in der Lage sein werden, miteinander zu kommunizieren, um die Subtract-Funktion aufzurufen:
```bash
mig -header myipcUser.h -sheader myipcServer.h myipc.defs
```
Es werden mehrere neue Dateien im aktuellen Verzeichnis erstellt.

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

MIG (Mach Interface Generator) is a tool used to define inter-process communication on macOS systems. It generates client and server-side code for message-based IPC. By defining the messages and data structures in a .defs file, MIG creates the necessary code to handle the communication between processes.

#### Example:

```c
routine my_ipc_server_routine {
    mach_msg_header_t Head;
    mach_msg_type_t Type;
    int data;
} InData;

routine my_ipc_server_routine {
    mach_msg_header_t Head;
    mach_msg_type_t Type;
    int data;
} OutData;
```

In the example above, `my_ipc_server_routine` is defined with input and output data structures. This code will be used by MIG to generate the server-side code for handling the IPC messages.

MIG is a powerful tool that can be abused by attackers to escalate privileges on macOS systems. By understanding how MIG works and how it generates code, attackers can find vulnerabilities and exploit them to gain higher privileges. It is essential for macOS system administrators to be aware of the risks associated with MIG and take necessary steps to secure their systems against potential attacks. 

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
In diesem Beispiel haben wir nur 1 Funktion in den Definitionen definiert, aber wenn wir mehr Funktionen definiert hätten, wären sie innerhalb des Arrays von **`SERVERPREFmyipc_subsystem`** gewesen und die erste wäre der ID **500** zugewiesen worden, die zweite der ID **501**...

Tatsächlich ist es möglich, diese Beziehung in der Struktur **`subsystem_to_name_map_myipc`** aus **`myipcServer.h`** zu identifizieren:
```c
#ifndef subsystem_to_name_map_myipc
#define subsystem_to_name_map_myipc \
{ "Subtract", 500 }
#endif
```
Schließlich wird eine weitere wichtige Funktion zur Funktionsweise des Servers **`myipc_server`** sein, die tatsächlich die mit der empfangenen ID verbundene Funktion aufruft:

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

Im Folgenden ist der Code zum Erstellen eines einfachen **Servers** und **Clients**, bei dem der Client die Funktionen vom Server abziehen kann:

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

### macOS IPC: Interprozesskommunikation

#### macOS MIG: Mach-Schnittstellengenerator

Mach Interface Generator (MIG) ist ein Dienstprogramm, das in macOS zum Erstellen von IPC-Mechanismen verwendet wird. MIG generiert C-Dateien, die für die Kommunikation zwischen Prozessen über spezielle IPC-Ports erforderlich sind. Es ist wichtig zu verstehen, wie MIG funktioniert, um IPC-Schwachstellen in macOS-Anwendungen zu identifizieren und auszunutzen. 

Um MIG zu verwenden, müssen Sie die MIG-Spezifikationen verstehen, die die IPC-Schnittstellen definieren. Durch Analysieren dieser Spezifikationen können Schwachstellen in der IPC-Implementierung identifiziert werden, die möglicherweise für Privilegienescalation oder andere Angriffe ausgenutzt werden können. 

Ein gründliches Verständnis von MIG und der von ihm generierten IPC-Implementierung ist entscheidend für die Sicherheitsbewertung von macOS-Anwendungen und die Identifizierung potenzieller Angriffsvektoren. 

```c
#include <mach/mach.h>
#include <stdio.h>

int main() {
    mach_port_t server_port;
    kern_return_t kr;

    kr = task_for_pid(mach_task_self(), getpid(), &server_port);
    if (kr != KERN_SUCCESS) {
        printf("Failed to get task port for PID\n");
        return 1;
    }

    // Use the server port for further IPC communication
    return 0;
}
```

In diesem Beispiel wird gezeigt, wie ein Prozess den MIG-Generierungscode verwenden kann, um einen Serverport für die IPC-Kommunikation zu erhalten. Dies ist ein grundlegendes Konzept für die Interprozesskommunikation in macOS mithilfe von MIG. 

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
### Binäranalyse

Da viele Binärdateien jetzt MIG verwenden, um Mach-Ports freizulegen, ist es interessant zu wissen, wie man **identifiziert, dass MIG verwendet wurde** und die **Funktionen, die MIG mit jeder Nachrichten-ID ausführt**.

[**jtool2**](../../macos-apps-inspecting-debugging-and-fuzzing/#jtool2) kann MIG-Informationen aus einer Mach-O-Binärdatei analysieren, um die Nachrichten-ID anzuzeigen und die auszuführende Funktion zu identifizieren:
```bash
jtool2 -d __DATA.__const myipc_server | grep MIG
```
Es wurde zuvor erwähnt, dass die Funktion, die sich um **das Aufrufen der richtigen Funktion je nach empfangener Nachrichten-ID kümmert**, `myipc_server` war. Normalerweise haben Sie jedoch nicht die Symbole der Binärdatei (keine Funktionsnamen), daher ist es interessant zu **überprüfen, wie der dekompilierte Code aussieht**, da er immer sehr ähnlich sein wird (der Code dieser Funktion ist unabhängig von den freigegebenen Funktionen):

{% tabs %}
{% tab title="Dekompilierter myipc_server 1" %}
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

{% tab title="Dekompilierter myipc_server 2" %}
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
// Aufruf der berechneten Adresse, an der die Funktion sein sollte
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

<details>

<summary><strong>Erlernen Sie AWS-Hacking von Grund auf mit</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Andere Möglichkeiten, HackTricks zu unterstützen:

* Wenn Sie Ihr **Unternehmen in HackTricks beworben sehen** oder **HackTricks im PDF-Format herunterladen** möchten, überprüfen Sie die [**ABONNEMENTPLÄNE**](https://github.com/sponsors/carlospolop)!
* Holen Sie sich das [**offizielle PEASS & HackTricks-Merch**](https://peass.creator-spring.com)
* Entdecken Sie [**The PEASS Family**](https://opensea.io/collection/the-peass-family), unsere Sammlung exklusiver [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Treten Sie der** 💬 [**Discord-Gruppe**](https://discord.gg/hRep4RUj7f) oder der [**Telegram-Gruppe**](https://t.me/peass) bei oder **folgen** Sie uns auf **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Teilen Sie Ihre Hacking-Tricks, indem Sie Pull Requests an die** [**HackTricks**](https://github.com/carlospolop/hacktricks) **und** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **GitHub-Repositorys senden.**

</details>
