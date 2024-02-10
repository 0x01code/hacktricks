# Dumping della memoria di macOS

<details>

<summary><strong>Impara l'hacking di AWS da zero a esperto con</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Altri modi per supportare HackTricks:

* Se vuoi vedere la tua **azienda pubblicizzata su HackTricks** o **scaricare HackTricks in PDF**, controlla i [**PACCHETTI DI ABBONAMENTO**](https://github.com/sponsors/carlospolop)!
* Ottieni il [**merchandising ufficiale di PEASS & HackTricks**](https://peass.creator-spring.com)
* Scopri [**The PEASS Family**](https://opensea.io/collection/the-peass-family), la nostra collezione di [**NFT**](https://opensea.io/collection/the-peass-family) esclusivi
* **Unisciti al** 💬 [**gruppo Discord**](https://discord.gg/hRep4RUj7f) o al [**gruppo Telegram**](https://t.me/peass) o **seguici** su **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Condividi i tuoi trucchi di hacking inviando PR ai repository** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github.

</details>

## Artefatti di memoria

### File di swap

I file di swap, come `/private/var/vm/swapfile0`, fungono da **cache quando la memoria fisica è piena**. Quando non c'è più spazio nella memoria fisica, i dati vengono trasferiti in un file di swap e poi riportati nella memoria fisica quando necessario. Potrebbero essere presenti più file di swap, con nomi come swapfile0, swapfile1, e così via.

### Immagine di ibernazione

Il file situato in `/private/var/vm/sleepimage` è fondamentale durante la **modalità di ibernazione**. **I dati dalla memoria vengono memorizzati in questo file quando macOS si iberna**. Al risveglio del computer, il sistema recupera i dati dalla memoria da questo file, consentendo all'utente di riprendere da dove aveva lasciato.

È importante notare che sui sistemi MacOS moderni, questo file è tipicamente crittografato per motivi di sicurezza, rendendo difficile il recupero.

* Per verificare se la crittografia è abilitata per il sleepimage, è possibile eseguire il comando `sysctl vm.swapusage`. Questo mostrerà se il file è crittografato.

### Log della pressione di memoria

Un altro file importante correlato alla memoria nei sistemi MacOS è il **log della pressione di memoria**. Questi log si trovano in `/var/log` e contengono informazioni dettagliate sull'utilizzo della memoria del sistema e sugli eventi di pressione della memoria. Possono essere particolarmente utili per diagnosticare problemi legati alla memoria o per comprendere come il sistema gestisce la memoria nel tempo.

## Dumping della memoria con osxpmem

Per effettuare il dumping della memoria su una macchina MacOS, è possibile utilizzare [**osxpmem**](https://github.com/google/rekall/releases/download/v1.5.1/osxpmem-2.1.post4.zip).

**Nota**: Le seguenti istruzioni funzioneranno solo per i Mac con architettura Intel. Questo strumento è ora archiviato e l'ultima versione risale al 2017. Il binario scaricato utilizzando le istruzioni seguenti è destinato ai chip Intel in quanto Apple Silicon non era disponibile nel 2017. Potrebbe essere possibile compilare il binario per l'architettura arm64, ma dovrai provare da solo.
```bash
#Dump raw format
sudo osxpmem.app/osxpmem --format raw -o /tmp/dump_mem

#Dump aff4 format
sudo osxpmem.app/osxpmem -o /tmp/dump_mem.aff4
```
Se trovi questo errore: `osxpmem.app/MacPmem.kext non è riuscito a caricare - (libkern/kext) autenticazione fallita (proprietà/file di autorizzazioni); controlla i log di sistema/kernel per gli errori o prova kextutil(8)` Puoi risolverlo facendo:
```bash
sudo cp -r osxpmem.app/MacPmem.kext "/tmp/"
sudo kextutil "/tmp/MacPmem.kext"
#Allow the kext in "Security & Privacy --> General"
sudo osxpmem.app/osxpmem --format raw -o /tmp/dump_mem
```
**Altri errori** potrebbero essere risolti **consentendo il caricamento del kext** in "Sicurezza e Privacy --> Generale", basta **consentirlo**.

Puoi anche utilizzare questo **oneliner** per scaricare l'applicazione, caricare il kext e fare il dump della memoria:

{% code overflow="wrap" %}
```bash
sudo su
cd /tmp; wget https://github.com/google/rekall/releases/download/v1.5.1/osxpmem-2.1.post4.zip; unzip osxpmem-2.1.post4.zip; chown -R root:wheel osxpmem.app/MacPmem.kext; kextload osxpmem.app/MacPmem.kext; osxpmem.app/osxpmem --format raw -o /tmp/dump_mem
```
{% endcode %}

<details>

<summary><strong>Impara l'hacking di AWS da zero a esperto con</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Altri modi per supportare HackTricks:

* Se vuoi vedere la tua **azienda pubblicizzata su HackTricks** o **scaricare HackTricks in PDF** Controlla i [**PIANI DI ABBONAMENTO**](https://github.com/sponsors/carlospolop)!
* Ottieni il [**merchandising ufficiale di PEASS & HackTricks**](https://peass.creator-spring.com)
* Scopri [**The PEASS Family**](https://opensea.io/collection/the-peass-family), la nostra collezione di esclusive [**NFT**](https://opensea.io/collection/the-peass-family)
* **Unisciti al** 💬 [**gruppo Discord**](https://discord.gg/hRep4RUj7f) o al [**gruppo Telegram**](https://t.me/peass) o **seguici** su **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Condividi i tuoi trucchi di hacking inviando PR ai repository github di** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
