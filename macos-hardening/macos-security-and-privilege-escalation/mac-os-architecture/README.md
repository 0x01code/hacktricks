# Architettura del macOS Kernel & delle Estensioni di Sistema

{% hint style="success" %}
Impara e pratica l'Hacking AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Impara e pratica l'Hacking GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Sostieni HackTricks</summary>

* Controlla i [**piani di abbonamento**](https://github.com/sponsors/carlospolop)!
* **Unisciti al** 💬 [**gruppo Discord**](https://discord.gg/hRep4RUj7f) o al [**gruppo telegram**](https://t.me/peass) o **seguici** su **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Condividi trucchi di hacking inviando PR a** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repos di Github.

</details>
{% endhint %}

## Kernel XNU

Il **cuore di macOS è XNU**, che sta per "X is Not Unix". Questo kernel è fondamentalmente composto dal **microkernel Mach** (di cui si parlerà in seguito), **e** elementi della Berkeley Software Distribution (**BSD**). XNU fornisce anche una piattaforma per **driver di kernel tramite un sistema chiamato I/O Kit**. Il kernel XNU fa parte del progetto open source Darwin, il che significa che **il suo codice sorgente è liberamente accessibile**.

Dal punto di vista di un ricercatore di sicurezza o di uno sviluppatore Unix, **macOS** può sembrare piuttosto **simile** a un sistema **FreeBSD** con un'interfaccia grafica elegante e una serie di applicazioni personalizzate. La maggior parte delle applicazioni sviluppate per BSD si compilerà ed eseguirà su macOS senza necessità di modifiche, poiché gli strumenti a riga di comando familiari agli utenti Unix sono tutti presenti in macOS. Tuttavia, poiché il kernel XNU incorpora Mach, ci sono alcune differenze significative tra un sistema simile a Unix tradizionale e macOS, e queste differenze potrebbero causare potenziali problemi o fornire vantaggi unici.

Versione open source di XNU: [https://opensource.apple.com/source/xnu/](https://opensource.apple.com/source/xnu/)

### Mach

Mach è un **microkernel** progettato per essere **compatibile con UNIX**. Uno dei suoi principi di progettazione chiave era **minimizzare** la quantità di **codice** in esecuzione nello **spazio del kernel** e invece consentire a molte funzioni tipiche del kernel, come il sistema di file, la rete e l'I/O, di **eseguirsi come attività a livello utente**.

In XNU, Mach è **responsabile di molte delle operazioni critiche a basso livello** che un kernel gestisce tipicamente, come la pianificazione del processore, il multitasking e la gestione della memoria virtuale.

### BSD

Il **kernel** XNU **incorpora anche** una quantità significativa di codice derivato dal progetto **FreeBSD**. Questo codice **viene eseguito come parte del kernel insieme a Mach**, nello stesso spazio degli indirizzi. Tuttavia, il codice FreeBSD all'interno di XNU potrebbe differire sostanzialmente dal codice FreeBSD originale perché sono state apportate modifiche per garantire la sua compatibilità con Mach. FreeBSD contribuisce a molte operazioni del kernel, tra cui:

* Gestione dei processi
* Gestione dei segnali
* Meccanismi di sicurezza di base, inclusa la gestione degli utenti e dei gruppi
* Infrastruttura delle chiamate di sistema
* Stack TCP/IP e socket
* Firewall e filtraggio dei pacchetti

Comprendere l'interazione tra BSD e Mach può essere complesso, a causa dei loro diversi quadri concettuali. Ad esempio, BSD utilizza i processi come sua unità di esecuzione fondamentale, mentre Mach opera in base ai thread. Questa discrepanza viene conciliata in XNU **associando ogni processo BSD con un'attività Mach** che contiene esattamente un thread Mach. Quando viene utilizzata la chiamata di sistema fork() di BSD, il codice BSD all'interno del kernel utilizza le funzioni Mach per creare una struttura di attività e di thread.

Inoltre, **Mach e BSD mantengono modelli di sicurezza diversi**: il modello di sicurezza di **Mach** si basa sui **diritti di porta**, mentre il modello di sicurezza di BSD opera in base alla **proprietà del processo**. Le disparità tra questi due modelli hanno occasionalmente causato vulnerabilità di escalation dei privilegi locali. Oltre alle tipiche chiamate di sistema, ci sono anche **trappole Mach che consentono ai programmi dello spazio utente di interagire con il kernel**. Questi diversi elementi insieme formano l'architettura ibrida e sfaccettata del kernel macOS.

### I/O Kit - Driver

L'I/O Kit è un framework **driver di dispositivo orientato agli oggetti** open source nel kernel XNU, gestisce **driver di dispositivo caricati dinamicamente**. Consente di aggiungere codice modulare al kernel al volo, supportando hardware diversificato.

{% content-ref url="macos-iokit.md" %}
[macos-iokit.md](macos-iokit.md)
{% endcontent-ref %}

### IPC - Comunicazione tra Processi

{% content-ref url="../macos-proces-abuse/macos-ipc-inter-process-communication/" %}
[macos-ipc-inter-process-communication](../macos-proces-abuse/macos-ipc-inter-process-communication/)
{% endcontent-ref %}

### Kernelcache

Il **kernelcache** è una versione **pre-compilata e pre-linkata del kernel XNU**, insieme a driver di dispositivo essenziali ed estensioni di kernel. È memorizzato in un formato **compresso** e viene decompresso in memoria durante il processo di avvio. Il kernelcache facilita un **avvio più veloce** avendo disponibile una versione pronta all'uso del kernel e dei driver cruciali, riducendo il tempo e le risorse che altrimenti sarebbero impiegate nel caricare dinamicamente e collegare questi componenti al momento dell'avvio.

In iOS si trova in **`/System/Library/Caches/com.apple.kernelcaches/kernelcache`** in macOS puoi trovarlo con **`find / -name kernelcache 2>/dev/null`** o **`mdfind kernelcache | grep kernelcache`**

È possibile eseguire **`kextstat`** per controllare le estensioni di kernel caricate.

#### IMG4

Il formato file IMG4 è un formato di contenitore utilizzato da Apple nei suoi dispositivi iOS e macOS per **memorizzare e verificare in modo sicuro** componenti firmware (come il **kernelcache**). Il formato IMG4 include un'intestazione e diversi tag che racchiudono diverse parti di dati, inclusi il payload effettivo (come un kernel o un bootloader), una firma e un insieme di proprietà del manifesto. Il formato supporta la verifica crittografica, consentendo al dispositivo di confermare l'autenticità e l'integrità del componente firmware prima di eseguirlo.

Solitamente è composto dai seguenti componenti:

* **Payload (IM4P)**:
* Spesso compresso (LZFSE4, LZSS, ...)
* Opzionalmente criptato
* **Manifesto (IM4M)**:
* Contiene Firma
* Dizionario Chiave/Valore aggiuntivo
* **Informazioni di Ripristino (IM4R)**:
* Noti anche come APNonce
* Impedisce il replay di alcuni aggiornamenti
* FACOLTATIVO: Di solito non viene trovato

Decomprimere il Kernelcache:
```bash
# pyimg4 (https://github.com/m1stadev/PyIMG4)
pyimg4 im4p extract -i kernelcache.release.iphone14 -o kernelcache.release.iphone14.e

# img4tool (https://github.com/tihmstar/img4tool
img4tool -e kernelcache.release.iphone14 -o kernelcache.release.iphone14.e
```
#### Simboli del Kernelcache

A volte Apple rilascia il **kernelcache** con i **simboli**. Puoi scaricare alcuni firmware con i simboli seguendo i link su [https://theapplewiki.com](https://theapplewiki.com/).

### IPSW

Questi sono **firmware** Apple che puoi scaricare da [**https://ipsw.me/**](https://ipsw.me/). Tra gli altri file conterrà il **kernelcache**.\
Per **estrarre** i file puoi semplicemente **scompattarlo**.

Dopo aver estratto il firmware otterrai un file come: **`kernelcache.release.iphone14`**. È in formato **IMG4**, puoi estrarre le informazioni interessanti con:

* [**pyimg4**](https://github.com/m1stadev/PyIMG4)

{% code overflow="wrap" %}
```bash
pyimg4 im4p extract -i kernelcache.release.iphone14 -o kernelcache.release.iphone14.e
```
{% endcode %}

* [**img4tool**](https://github.com/tihmstar/img4tool)
```bash
img4tool -e kernelcache.release.iphone14 -o kernelcache.release.iphone14.e
```
Puoi controllare i simboli estratti dal kernelcache con: **`nm -a kernelcache.release.iphone14.e | wc -l`**

Con questo possiamo ora **estraire tutte le estensioni** o quella **che ti interessa:**
```bash
# List all extensions
kextex -l kernelcache.release.iphone14.e
## Extract com.apple.security.sandbox
kextex -e com.apple.security.sandbox kernelcache.release.iphone14.e

# Extract all
kextex_all kernelcache.release.iphone14.e

# Check the extension for symbols
nm -a binaries/com.apple.security.sandbox | wc -l
```
## Estensioni del kernel di macOS

macOS è **estremamente restrittivo nel caricare le estensioni del kernel** (.kext) a causa dei privilegi elevati con cui il codice verrà eseguito. Attualmente, per impostazione predefinita è praticamente impossibile (a meno che non venga trovato un bypass).

{% content-ref url="macos-kernel-extensions.md" %}
[macos-kernel-extensions.md](macos-kernel-extensions.md)
{% endcontent-ref %}

### Estensioni di sistema di macOS

Invece di utilizzare le Estensioni del Kernel, macOS ha creato le Estensioni di Sistema, che offrono API a livello utente per interagire con il kernel. In questo modo, gli sviluppatori possono evitare di utilizzare le estensioni del kernel.

{% content-ref url="macos-system-extensions.md" %}
[macos-system-extensions.md](macos-system-extensions.md)
{% endcontent-ref %}

## Riferimenti

* [**The Mac Hacker's Handbook**](https://www.amazon.com/-/es/Charlie-Miller-ebook-dp-B004U7MUMU/dp/B004U7MUMU/ref=mt\_other?\_encoding=UTF8\&me=\&qid=)
* [**https://taomm.org/vol1/analysis.html**](https://taomm.org/vol1/analysis.html)

{% hint style="success" %}
Impara e pratica l'Hacking su AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Impara e pratica l'Hacking su GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Supporta HackTricks</summary>

* Controlla i [**piani di abbonamento**](https://github.com/sponsors/carlospolop)!
* **Unisciti al** 💬 [**gruppo Discord**](https://discord.gg/hRep4RUj7f) o al [**gruppo telegram**](https://t.me/peass) o **seguici** su **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Condividi trucchi di hacking inviando PR ai** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repository di GitHub.

</details>
{% endhint %}
