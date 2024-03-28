# Bundle macOS

<details>

<summary><strong>Impara l'hacking AWS da zero a esperto con</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (Esperto Red Team AWS di HackTricks)</strong></a><strong>!</strong></summary>

Altri modi per supportare HackTricks:

* Se vuoi vedere la tua **azienda pubblicizzata su HackTricks** o **scaricare HackTricks in PDF** Controlla i [**PIANI DI ABBONAMENTO**](https://github.com/sponsors/carlospolop)!
* Ottieni il [**merchandising ufficiale PEASS & HackTricks**](https://peass.creator-spring.com)
* Scopri [**La Famiglia PEASS**](https://opensea.io/collection/the-peass-family), la nostra collezione di [**NFT esclusivi**](https://opensea.io/collection/the-peass-family)
* **Unisciti al** 💬 [**gruppo Discord**](https://discord.gg/hRep4RUj7f) o al [**gruppo telegram**](https://t.me/peass) o **seguici** su **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Condividi i tuoi trucchi di hacking inviando PR ai** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repository di Github.

</details>

## Informazioni di Base

I bundle in macOS fungono da contenitori per una varietà di risorse, tra cui applicazioni, librerie e altri file necessari, facendoli apparire come singoli oggetti in Finder, come i familiari file `*.app`. Il bundle più comunemente incontrato è il bundle `.app`, anche se altri tipi come `.framework`, `.systemextension` e `.kext` sono anche diffusi.

### Componenti Essenziali di un Bundle

All'interno di un bundle, in particolare all'interno della directory `<applicazione>.app/Contents/`, sono presenti una varietà di risorse importanti:

* **\_CodeSignature**: Questa directory memorizza dettagli di firma del codice vitali per verificare l'integrità dell'applicazione. È possibile ispezionare le informazioni sulla firma del codice utilizzando comandi come: %%%bash openssl dgst -binary -sha1 /Applications/Safari.app/Contents/Resources/Assets.car | openssl base64 %%%
* **MacOS**: Contiene il binario eseguibile dell'applicazione che viene eseguito all'interazione dell'utente.
* **Resources**: Un repository per i componenti dell'interfaccia utente dell'applicazione, tra cui immagini, documenti e descrizioni dell'interfaccia (file nib/xib).
* **Info.plist**: Agisce come file di configurazione principale dell'applicazione, cruciale affinché il sistema riconosca e interagisca con l'applicazione in modo appropriato.

#### Chiavi Importanti in Info.plist

Il file `Info.plist` è un pilastro per la configurazione dell'applicazione, contenente chiavi come:

* **CFBundleExecutable**: Specifica il nome del file eseguibile principale situato nella directory `Contents/MacOS`.
* **CFBundleIdentifier**: Fornisce un identificatore globale per l'applicazione, ampiamente utilizzato da macOS per la gestione delle applicazioni.
* **LSMinimumSystemVersion**: Indica la versione minima di macOS richiesta affinché l'applicazione possa essere eseguita.

### Esplorare i Bundle

Per esplorare i contenuti di un bundle, come `Safari.app`, è possibile utilizzare il seguente comando: `bash ls -lR /Applications/Safari.app/Contents`

Questa esplorazione rivela directory come `_CodeSignature`, `MacOS`, `Resources` e file come `Info.plist`, ognuno con uno scopo unico, dalla sicurezza dell'applicazione alla definizione dell'interfaccia utente e dei parametri operativi.

#### Directory Aggiuntive del Bundle

Oltre alle directory comuni, i bundle possono includere anche:

* **Frameworks**: Contiene framework inclusi nell'applicazione. I framework sono come dylibs con risorse aggiuntive.
* **PlugIns**: Una directory per plug-in ed estensioni che migliorano le capacità dell'applicazione.
* **XPCServices**: Contiene servizi XPC utilizzati dall'applicazione per la comunicazione out-of-process.

Questa struttura garantisce che tutti i componenti necessari siano racchiusi nel bundle, facilitando un ambiente di applicazione modulare e sicuro.

Per informazioni più dettagliate sulle chiavi di `Info.plist` e i loro significati, la documentazione degli sviluppatori Apple fornisce risorse estese: [Riferimento Chiave Info.plist Apple](https://developer.apple.com/library/archive/documentation/General/Reference/InfoPlistKeyReference/Introduction/Introduction.html).

<details>

<summary><strong>Impara l'hacking AWS da zero a esperto con</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (Esperto Red Team AWS di HackTricks)</strong></a><strong>!</strong></summary>

Altri modi per supportare HackTricks:

* Se vuoi vedere la tua **azienda pubblicizzata su HackTricks** o **scaricare HackTricks in PDF** Controlla i [**PIANI DI ABBONAMENTO**](https://github.com/sponsors/carlospolop)!
* Ottieni il [**merchandising ufficiale PEASS & HackTricks**](https://peass.creator-spring.com)
* Scopri [**La Famiglia PEASS**](https://opensea.io/collection/the-peass-family), la nostra collezione di [**NFT esclusivi**](https://opensea.io/collection/the-peass-family)
* **Unisciti al** 💬 [**gruppo Discord**](https://discord.gg/hRep4RUj7f) o al [**gruppo telegram**](https://t.me/peass) o **seguici** su **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Condividi i tuoi trucchi di hacking inviando PR ai** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repository di Github.

</details>
