# macOS MIG - Mach Interface Generator

<details>

<summary><strong>Naucz się hakować AWS od zera do bohatera z</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Inne sposoby wsparcia HackTricks:

* Jeśli chcesz zobaczyć swoją **firmę reklamowaną w HackTricks** lub **pobrać HackTricks w formacie PDF**, sprawdź [**PLAN SUBSKRYPCYJNY**](https://github.com/sponsors/carlospolop)!
* Zdobądź [**oficjalne gadżety PEASS & HackTricks**](https://peass.creator-spring.com)
* Odkryj [**Rodzinę PEASS**](https://opensea.io/collection/the-peass-family), naszą kolekcję ekskluzywnych [**NFT**](https://opensea.io/collection/the-peass-family)
* **Dołącz do** 💬 [**grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy telegramowej**](https://t.me/peass) lub **śledź** nas na **Twitterze** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Podziel się swoimi sztuczkami hakerskimi, przesyłając PR-y do** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>

MIG został stworzony, aby **uproszczać proces tworzenia kodu Mach IPC**. W zasadzie **generuje wymagany kod** dla serwera i klienta w celu komunikacji z określoną definicją. Nawet jeśli wygenerowany kod jest brzydki, programista będzie musiał go tylko zaimportować, a jego kod będzie znacznie prostszy niż wcześniej.

### Przykład

Utwórz plik definicji, w tym przypadku z bardzo prostą funkcją:

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

Teraz użyj mig, aby wygenerować kod serwera i klienta, które będą w stanie komunikować się między sobą, aby wywołać funkcję Odejmij:
```bash
mig -header myipcUser.h -sheader myipcServer.h myipc.defs
```
W bieżącym katalogu zostanie utworzonych kilka nowych plików.

W plikach **`myipcServer.c`** i **`myipcServer.h`** znajduje się deklaracja i definicja struktury **`SERVERPREFmyipc_subsystem`**, która definiuje funkcję do wywołania na podstawie otrzymanego identyfikatora wiadomości (ustawiliśmy początkową liczbę na 500):

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
{% tab title="myipcServer.h" %}

```c
#ifndef myipcServer_h
#define myipcServer_h

#include <stdio.h>
#include <stdlib.h>
#include <mach/mach.h>
#include <mach/mach_error.h>
#include <servers/bootstrap.h>
#include <mach/mach_traps.h>
#include <mach/mach_types.h>
#include <mach/mach_init.h>
#include <mach/mach_port.h>
#include <mach/mach_interface.h>
#include <mach/mach_vm.h>
#include <mach/mach_voucher_types.h>
#include <mach/mach_voucher.h>
#include <mach/mach_time.h>
#include <mach/mach_host.h>
#include <mach/mach_host_priv.h>
#include <mach/mach_host_server.h>
#include <mach/mach_host_user.h>
#include <mach/mach_host_reboot.h>
#include <mach/mach_host_special_ports.h>
#include <mach/mach_host_info.h>
#include <mach/mach_host_notify.h>
#include <mach/mach_host_security.h>
#include <mach/mach_host_policy.h>
#include <mach/mach_host_qos.h>
#include <mach/mach_host_ledger.h>
#include <mach/mach_host_statistics.h>
#include <mach/mach_host_vm_info.h>
#include <mach/mach_host_vm_priv.h>
#include <mach/mach_host_vm_ext.h>
#include <mach/mach_host_vm_prot.h>
#include <mach/mach_host_vm_behavior.h>
#include <mach/mach_host_vm_region.h>
#include <mach/mach_host_vm_wire.h>
#include <mach/mach_host_vm_purgable.h>
#include <mach/mach_host_vm_info_internal.h>
#include <mach/mach_host_vm_info_external.h>
#include <mach/mach_host_vm_info_shared.h>
#include <mach/mach_host_vm_info_compressed.h>
#include <mach/mach_host_vm_info_region.h>
#include <mach/mach_host_vm_info_region_internal.h>
#include <mach/mach_host_vm_info_region_external.h>
#include <mach/mach_host_vm_info_region_shared.h>
#include <mach/mach_host_vm_info_region_compressed.h>
#include <mach/mach_host_vm_info_region_purgable.h>
#include <mach/mach_host_vm_info_region_purgable_internal.h>
#include <mach/mach_host_vm_info_region_purgable_external.h>
#include <mach/mach_host_vm_info_region_purgable_shared.h>
#include <mach/mach_host_vm_info_region_purgable_compressed.h>
#include <mach/mach_host_vm_info_region_purgable_wired.h>
#include <mach/mach_host_vm_info_region_purgable_unwired.h>
#include <mach/mach_host_vm_info_region_purgable_volatile.h>
#include <mach/mach_host_vm_info_region_purgable_nonvolatile.h>
#include <mach/mach_host_vm_info_region_purgable_empty.h>
#include <mach/mach_host_vm_info_region_purgable_killed.h>
#include <mach/mach_host_vm_info_region_purgable_zf.h>
#include <mach/mach_host_vm_info_region_purgable_reusable.h>
#include <mach/mach_host_vm_info_region_purgable_nonreusable.h>
#include <mach/mach_host_vm_info_region_purgable_dontdump.h>
#include <mach/mach_host_vm_info_region_purgable_nolock.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_internal.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_external.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_shared.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_compressed.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_wired.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_unwired.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_volatile.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_nonvolatile.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_empty.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_killed.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_zf.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_reusable.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_nonreusable.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_dontdump.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_nolock.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_nolock_internal.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_nolock_external.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_nolock_shared.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_nolock_compressed.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_nolock_wired.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_nolock_unwired.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_nolock_volatile.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_nolock_nonvolatile.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_nolock_empty.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_nolock_killed.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_nolock_zf.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_nolock_reusable.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_nolock_nonreusable.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_nolock_dontdump.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_nolock_nolock.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_nolock_nolock_internal.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_nolock_nolock_external.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_nolock_nolock_shared.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_nolock_nolock_compressed.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_nolock_nolock_wired.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_nolock_nolock_unwired.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_nolock_nolock_volatile.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_nolock_nolock_nonvolatile.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_nolock_nolock_empty.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_nolock_nolock_killed.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_nolock_nolock_zf.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_nolock_nolock_reusable.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_nolock_nolock_nonreusable.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_nolock_nolock_dontdump.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_nolock_nolock_nolock.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_nolock_nolock_nolock_internal.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_nolock_nolock_nolock_external.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_nolock_nolock_nolock_shared.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_nolock_nolock_nolock_compressed.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_nolock_nolock_nolock_wired.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_nolock_nolock_nolock_unwired.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_nolock_nolock_nolock_volatile.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_nolock_nolock_nolock_nonvolatile.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_nolock_nolock_nolock_empty.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_nolock_nolock_nolock_killed.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_nolock_nolock_nolock_zf.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_nolock_nolock_nolock_reusable.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_nolock_nolock_nolock_nonreusable.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_nolock_nolock_nolock_dontdump.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_nolock_nolock_nolock_nolock.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_nolock_nolock_nolock_nolock_internal.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_nolock_nolock_nolock_nolock_external.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_nolock_nolock_nolock_nolock_shared.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_nolock_nolock_nolock_nolock_compressed.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_nolock_nolock_nolock_nolock_wired.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_nolock_nolock_nolock_nolock_unwired.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_nolock_nolock_nolock_nolock_volatile.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_nolock_nolock_nolock_nolock_nonvolatile.h>
#include <mach/mach_host_vm_info_region_purgable_nolock_nolock_nolock
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

Na podstawie poprzedniej struktury funkcja **`myipc_server_routine`** otrzyma **ID wiadomości** i zwróci odpowiednią funkcję do wywołania:
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
W tym przykładzie zdefiniowaliśmy tylko 1 funkcję w definicjach, ale gdybyśmy zdefiniowali więcej funkcji, byłyby one umieszczone w tablicy **`SERVERPREFmyipc_subsystem`**, a pierwsza z nich zostałaby przypisana do ID **500**, druga do ID **501**...

Tak naprawdę można zidentyfikować tę relację w strukturze **`subsystem_to_name_map_myipc`** z pliku **`myipcServer.h`**:
```c
#ifndef subsystem_to_name_map_myipc
#define subsystem_to_name_map_myipc \
{ "Subtract", 500 }
#endif
```
Wreszcie, kolejną ważną funkcją, która sprawi, że serwer będzie działać, będzie **`myipc_server`**, która właściwie **wywoła funkcję** związaną z otrzymanym identyfikatorem:

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
/* Minimal size: routine() will update it if different */
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

Sprawdź wcześniej wyróżnione linie, które odnoszą się do funkcji, które mają być wywołane na podstawie identyfikatora.

Poniżej znajduje się kod do utworzenia prostego **serwera** i **klienta**, gdzie klient może wywołać funkcje Odejmowanie na serwerze:

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
{% tab title="myipc_client.c" %}

```c
#include <stdio.h>
#include <stdlib.h>
#include <mach/mach.h>
#include <mach/message.h>
#include <servers/bootstrap.h>

#define SERVER_NAME "com.example.myipc_server"

int main() {
    kern_return_t kr;
    mach_port_t server_port;
    char message[256] = "Hello, server!";
    mach_msg_header_t *msg = (mach_msg_header_t *)message;
    
    // Look up the server port
    kr = bootstrap_look_up(bootstrap_port, SERVER_NAME, &server_port);
    if (kr != KERN_SUCCESS) {
        printf("Failed to look up server port: %s\n", mach_error_string(kr));
        exit(1);
    }
    
    // Set up the message header
    msg->msgh_bits = MACH_MSGH_BITS(MACH_MSG_TYPE_COPY_SEND, 0);
    msg->msgh_size = sizeof(message);
    msg->msgh_remote_port = server_port;
    msg->msgh_local_port = MACH_PORT_NULL;
    msg->msgh_reserved = 0;
    msg->msgh_id = 0;
    
    // Send the message
    kr = mach_msg(msg, MACH_SEND_MSG, msg->msgh_size, 0, MACH_PORT_NULL, MACH_MSG_TIMEOUT_NONE, MACH_PORT_NULL);
    if (kr != KERN_SUCCESS) {
        printf("Failed to send message: %s\n", mach_error_string(kr));
        exit(1);
    }
    
    printf("Message sent to server!\n");
    
    return 0;
}
```

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

### Analiza binarna

Ponieważ wiele plików binarnych teraz używa MIG do eksponowania portów mach, interesujące jest wiedzieć, jak **zidentyfikować, że został użyty MIG** oraz **funkcje, które MIG wykonuje** przy każdym identyfikatorze wiadomości.

[**jtool2**](../../macos-apps-inspecting-debugging-and-fuzzing/#jtool2) może analizować informacje MIG z pliku binarnego Mach-O, wskazując identyfikator wiadomości i identyfikując funkcję do wykonania:
```bash
jtool2 -d __DATA.__const myipc_server | grep MIG
```
Wcześniej wspomniano, że funkcja, która będzie odpowiedzialna za **wywołanie odpowiedniej funkcji w zależności od otrzymanego identyfikatora wiadomości**, to `myipc_server`. Jednak zazwyczaj nie będziesz mieć symboli binarnych (brak nazw funkcji), więc interesujące jest **sprawdzenie, jak wygląda zdekompilowana wersja**, ponieważ zawsze będzie bardzo podobna (kod tej funkcji jest niezależny od funkcji wystawionych):

{% tabs %}
{% tab title="myipc_server zdekompilowany 1" %}
<pre class="language-c"><code class="lang-c">int _myipc_server(int arg0, int arg1) {
var_10 = arg0;
var_18 = arg1;
// Instrukcje początkowe do znalezienia odpowiednich wskaźników funkcji
*(int32_t *)var_18 = *(int32_t *)var_10 &#x26; 0x1f;
*(int32_t *)(var_18 + 0x8) = *(int32_t *)(var_10 + 0x8);
*(int32_t *)(var_18 + 0x4) = 0x24;
*(int32_t *)(var_18 + 0xc) = 0x0;
*(int32_t *)(var_18 + 0x14) = *(int32_t *)(var_10 + 0x14) + 0x64;
*(int32_t *)(var_18 + 0x10) = 0x0;
if (*(int32_t *)(var_10 + 0x14) &#x3C;= 0x1f4 &#x26;&#x26; *(int32_t *)(var_10 + 0x14) >= 0x1f4) {
rax = *(int32_t *)(var_10 + 0x14);
// Wywołanie sign_extend_64, które może pomóc zidentyfikować tę funkcję
// To zapisuje w rax wskaźnik do wywołania, które trzeba wywołać
// Sprawdź użycie adresu 0x100004040 (tablica adresów funkcji)
// 0x1f4 = 500 (początkowy identyfikator)
<strong>            rax = *(sign_extend_64(rax - 0x1f4) * 0x28 + 0x100004040);
</strong>            var_20 = rax;
// If - else, jeśli if zwraca false, a else wywołuje odpowiednią funkcję i zwraca true
<strong>            if (rax == 0x0) {
</strong>                    *(var_18 + 0x18) = **_NDR_record;
*(int32_t *)(var_18 + 0x20) = 0xfffffffffffffed1;
var_4 = 0x0;
}
else {
// Obliczony adres, który wywołuje odpowiednią funkcję z 2 argumentami
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

{% tab title="myipc_server zdekompilowany 2" %}
To jest ta sama funkcja zdekompilowana w innej wersji Hopper free:

<pre class="language-c"><code class="lang-c">int _myipc_server(int arg0, int arg1) {
r31 = r31 - 0x40;
saved_fp = r29;
stack[-8] = r30;
var_10 = arg0;
var_18 = arg1;
// Instrukcje początkowe do znalezienia odpowiednich wskaźników funkcji
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
// 0x1f4 = 500 (początkowy identyfikator)
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
// To samo if else co w poprzedniej wersji
// Sprawdź użycie adresu 0x100004040 (tablica adresów funkcji)
<strong>                    if ((r8 &#x26; 0x1) == 0x0) {
</strong><strong>                            *(var_18 + 0x18) = **0x100004000;
</strong>                            *(int32_t *)(var_18 + 0x20) = 0xfffffed1;
var_4 = 0x0;
}
else {
// Wywołanie obliczonego adresu, gdzie powinna znajdować się funkcja
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

W rzeczywistości, jeśli przejdziesz do funkcji **`0x100004000`**, znajdziesz tablicę struktur **`routine_descriptor`**. Pierwszy element struktury to **adres**, gdzie jest zaimplementowana **funkcja**, a **struktura zajmuje 0x28 bajtów**, więc co 0x28 bajtów (zaczynając od bajtu 0) można uzyskać 8 bajtów, które będą **adresem funkcji**, która zostanie wywołana:

<figure><img src="../../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Te dane można wyodrębnić [**korzystając z tego skryptu Hoppera**](https://github.com/knightsc/hopper
* **Dołącz do** 💬 [**grupy Discord**](https://discord.gg/hRep4RUj7f) lub [**grupy Telegram**](https://t.me/peass) lub **śledź** nas na **Twitterze** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Podziel się swoimi sztuczkami hakerskimi, przesyłając PR-y do repozytoriów** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) na GitHubie.

</details>
