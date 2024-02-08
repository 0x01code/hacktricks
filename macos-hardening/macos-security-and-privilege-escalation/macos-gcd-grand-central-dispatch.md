# macOS GCD - Grand Central Dispatch

<details>

<summary><strong>Aprende hacking en AWS desde cero hasta experto con</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Otras formas de apoyar a HackTricks:

* Si deseas ver tu **empresa anunciada en HackTricks** o **descargar HackTricks en PDF** ¡Consulta los [**PLANES DE SUSCRIPCIÓN**](https://github.com/sponsors/carlospolop)!
* Obtén el [**oficial PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Descubre [**The PEASS Family**](https://opensea.io/collection/the-peass-family), nuestra colección exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Únete al** 💬 [**grupo de Discord**](https://discord.gg/hRep4RUj7f) o al [**grupo de telegram**](https://t.me/peass) o **síguenos** en **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Comparte tus trucos de hacking enviando PRs a los repositorios de** [**HackTricks**](https://github.com/carlospolop/hacktricks) y [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>

## Información Básica

**Grand Central Dispatch (GCD),** también conocido como **libdispatch**, está disponible tanto en macOS como en iOS. Es una tecnología desarrollada por Apple para optimizar el soporte de aplicaciones para la ejecución concurrente (multihilo) en hardware multinúcleo.

**GCD** proporciona y gestiona **colas FIFO** a las que tu aplicación puede **enviar tareas** en forma de **objetos de bloque**. Los bloques enviados a las colas de despacho son **ejecutados en un grupo de hilos** totalmente gestionado por el sistema. GCD crea automáticamente hilos para ejecutar las tareas en las colas de despacho y programa esas tareas para que se ejecuten en los núcleos disponibles.

{% hint style="success" %}
En resumen, para ejecutar código en **paralelo**, los procesos pueden enviar **bloques de código a GCD**, que se encargará de su ejecución. Por lo tanto, los procesos no crean nuevos hilos; **GCD ejecuta el código dado con su propio grupo de hilos**.
{% endhint %}

Esto es muy útil para gestionar la ejecución en paralelo de manera exitosa, reduciendo en gran medida la cantidad de hilos que los procesos crean y optimizando la ejecución en paralelo. Esto es ideal para tareas que requieren **gran paralelismo** (¿fuerza bruta?) o para tareas que no deben bloquear el hilo principal: Por ejemplo, el hilo principal en iOS maneja las interacciones de la interfaz de usuario, por lo que cualquier otra funcionalidad que pueda hacer que la aplicación se bloquee (búsqueda, acceso a la web, lectura de un archivo...) se gestiona de esta manera.

## Objective-C

En Objetive-C existen diferentes funciones para enviar un bloque para ser ejecutado en paralelo:

* [**dispatch\_async**](https://developer.apple.com/documentation/dispatch/1453057-dispatch\_async): Envía un bloque para su ejecución asíncrona en una cola de despacho y retorna inmediatamente.
* [**dispatch\_sync**](https://developer.apple.com/documentation/dispatch/1452870-dispatch\_sync): Envía un objeto de bloque para su ejecución y retorna después de que ese bloque termine de ejecutarse.
* [**dispatch\_once**](https://developer.apple.com/documentation/dispatch/1447169-dispatch\_once): Ejecuta un objeto de bloque solo una vez durante la vida útil de una aplicación.
* [**dispatch\_async\_and\_wait**](https://developer.apple.com/documentation/dispatch/3191901-dispatch\_async\_and\_wait): Envía un elemento de trabajo para su ejecución y retorna solo después de que termine de ejecutarse. A diferencia de [**`dispatch_sync`**](https://developer.apple.com/documentation/dispatch/1452870-dispatch\_sync), esta función respeta todos los atributos de la cola cuando ejecuta el bloque.

Estas funciones esperan estos parámetros: [**`dispatch_queue_t`**](https://developer.apple.com/documentation/dispatch/dispatch\_queue\_t) **`queue,`** [**`dispatch_block_t`**](https://developer.apple.com/documentation/dispatch/dispatch\_block\_t) **`block`**

Esta es la **estructura de un Bloque**:
```c
struct Block {
void *isa; // NSConcreteStackBlock,...
int flags;
int reserved;
void *invoke;
struct BlockDescriptor *descriptor;
// captured variables go here
};
```
Y este es un ejemplo de cómo usar **paralelismo** con **`dispatch_async`**:
```objectivec
#import <Foundation/Foundation.h>

// Define a block
void (^backgroundTask)(void) = ^{
// Code to be executed in the background
for (int i = 0; i < 10; i++) {
NSLog(@"Background task %d", i);
sleep(1);  // Simulate a long-running task
}
};

int main(int argc, const char * argv[]) {
@autoreleasepool {
// Create a dispatch queue
dispatch_queue_t backgroundQueue = dispatch_queue_create("com.example.backgroundQueue", NULL);

// Submit the block to the queue for asynchronous execution
dispatch_async(backgroundQueue, backgroundTask);

// Continue with other work on the main queue or thread
for (int i = 0; i < 10; i++) {
NSLog(@"Main task %d", i);
sleep(1);  // Simulate a long-running task
}
}
return 0;
}
```
## Swift

**`libswiftDispatch`** es una biblioteca que proporciona **enlaces Swift** al marco Grand Central Dispatch (GCD) que originalmente está escrito en C.\
La biblioteca **`libswiftDispatch`** envuelve las API de C GCD en una interfaz más amigable para Swift, lo que facilita y hace más intuitivo para los desarrolladores de Swift trabajar con GCD.

* **`DispatchQueue.global().sync{ ... }`**
* **`DispatchQueue.global().async{ ... }`**
* **`let onceToken = DispatchOnce(); onceToken.perform { ... }`**
* **`async await`**
* **`var (data, response) = await URLSession.shared.data(from: URL(string: "https://api.example.com/getData"))`**

**Ejemplo de código**:
```swift
import Foundation

// Define a closure (the Swift equivalent of a block)
let backgroundTask: () -> Void = {
for i in 0..<10 {
print("Background task \(i)")
sleep(1)  // Simulate a long-running task
}
}

// Entry point
autoreleasepool {
// Create a dispatch queue
let backgroundQueue = DispatchQueue(label: "com.example.backgroundQueue")

// Submit the closure to the queue for asynchronous execution
backgroundQueue.async(execute: backgroundTask)

// Continue with other work on the main queue
for i in 0..<10 {
print("Main task \(i)")
sleep(1)  // Simulate a long-running task
}
}
```
## Frida

El siguiente script de Frida se puede utilizar para **engancharse en varias funciones de `dispatch`** y extraer el nombre de la cola, la traza de ejecución y el bloque: [**https://github.com/seemoo-lab/frida-scripts/blob/main/scripts/libdispatch.js**](https://github.com/seemoo-lab/frida-scripts/blob/main/scripts/libdispatch.js)
```bash
frida -U <prog_name> -l libdispatch.js

dispatch_sync
Calling queue: com.apple.UIKit._UIReusePool.reuseSetAccess
Callback function: 0x19e3a6488 UIKitCore!__26-[_UIReusePool addObject:]_block_invoke
Backtrace:
0x19e3a6460 UIKitCore!-[_UIReusePool addObject:]
0x19e3a5db8 UIKitCore!-[UIGraphicsRenderer _enqueueContextForReuse:]
0x19e3a57fc UIKitCore!+[UIGraphicsRenderer _destroyCGContext:withRenderer:]
[...]
```
## Ghidra

Actualmente, Ghidra no comprende ni la estructura **`dispatch_block_t`** de ObjectiveC, ni la estructura **`swift_dispatch_block`**.

Entonces, si deseas que las entienda, simplemente puedes **declararlas**:

<figure><img src="../../.gitbook/assets/image (688).png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (690).png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (691).png" alt="" width="563"><figcaption></figcaption></figure>

Luego, encuentra un lugar en el código donde se **utilicen**:

{% hint style="success" %}
Ten en cuenta todas las referencias hechas a "block" para entender cómo podrías descubrir que se está utilizando la estructura.
{% endhint %}

<figure><img src="../../.gitbook/assets/image (692).png" alt="" width="563"><figcaption></figcaption></figure>

Haz clic derecho en la variable -> Cambiar tipo de variable y selecciona en este caso **`swift_dispatch_block`**:

<figure><img src="../../.gitbook/assets/image (693).png" alt="" width="563"><figcaption></figcaption></figure>

Ghidra reescribirá automáticamente todo:

<figure><img src="../../.gitbook/assets/image (694).png" alt="" width="563"><figcaption></figcaption></figure>
