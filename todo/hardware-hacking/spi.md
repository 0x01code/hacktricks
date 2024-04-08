# SPI

<details>

<summary><strong>Impara l'hacking AWS da zero a eroe con</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (Esperto Red Team AWS di HackTricks)</strong></a><strong>!</strong></summary>

Altri modi per supportare HackTricks:

* Se vuoi vedere la tua **azienda pubblicizzata su HackTricks** o **scaricare HackTricks in PDF** Controlla i [**PIANI DI ABBONAMENTO**](https://github.com/sponsors/carlospolop)!
* Ottieni il [**merchandising ufficiale di PEASS & HackTricks**](https://peass.creator-spring.com)
* Scopri [**La Famiglia PEASS**](https://opensea.io/collection/the-peass-family), la nostra collezione di [**NFT esclusivi**](https://opensea.io/collection/the-peass-family)
* **Unisciti al** 💬 [**gruppo Discord**](https://discord.gg/hRep4RUj7f) o al [**gruppo telegram**](https://t.me/peass) o **seguici** su **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Condividi i tuoi trucchi di hacking inviando PR a** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repos di github.

</details>

## Informazioni di Base

SPI (Serial Peripheral Interface) è un Protocollo di Comunicazione Seriale Sincrona utilizzato nei sistemi embedded per la comunicazione a breve distanza tra IC (Circuiti Integrati). Il Protocollo di Comunicazione SPI fa uso dell'architettura master-slave che è orchestrata dal Segnale di Clock e Chip Select. Un'architettura master-slave è composta da un master (di solito un microprocessore) che gestisce periferiche esterne come EEPROM, sensori, dispositivi di controllo, ecc. che sono considerati come schiavi.

Più schiavi possono essere collegati a un master ma i schiavi non possono comunicare tra loro. I schiavi sono amministrati da due pin, clock e chip select. Poiché SPI è un protocollo di comunicazione sincrono, i pin di input e output seguono i segnali di clock. Il chip select è utilizzato dal master per selezionare uno schiavo e interagire con esso. Quando il chip select è alto, il dispositivo schiavo non è selezionato mentre quando è basso, il chip è stato selezionato e il master interagirà con lo schiavo.

Il MOSI (Master Out, Slave In) e il MISO (Master In, Slave Out) sono responsabili dell'invio e della ricezione dei dati. I dati vengono inviati al dispositivo schiavo attraverso il pin MOSI mentre il chip select è mantenuto basso. I dati di input contengono istruzioni, indirizzi di memoria o dati secondo il datasheet del fornitore del dispositivo schiavo. Al ricevimento di un input valido, il pin MISO è responsabile della trasmissione dei dati al master. I dati di output vengono inviati esattamente al ciclo di clock successivo dopo la fine dell'input. Il pin MISO trasmette dati fino a quando i dati vengono completamente trasmessi o il master imposta il pin chip select alto (in tal caso, lo schiavo smetterebbe di trasmettere e il master non ascolterebbe dopo quel ciclo di clock).

## Dumping Firmware da EEPROM

Il dumping del firmware può essere utile per analizzare il firmware e trovare vulnerabilità al suo interno. Molte volte, il firmware non è disponibile su internet o è irrilevante a causa di variazioni di fattori come il numero di modello, la versione, ecc. Pertanto, estrarre il firmware direttamente dal dispositivo fisico può essere utile per essere specifici durante la ricerca di minacce.

Ottenere una Console Seriale può essere utile, ma spesso accade che i file siano in sola lettura. Questo limita l'analisi per vari motivi. Ad esempio, strumenti necessari per inviare e ricevere pacchetti potrebbero non essere presenti nel firmware. Quindi estrarre i binari per eseguirne l'ingegneria inversa potrebbe non essere fattibile. Pertanto, avere l'intero firmware dumpato sul sistema ed estrarre i binari per l'analisi può essere molto utile.

Inoltre, durante il red reaming e l'ottenimento dell'accesso fisico ai dispositivi, il dumping del firmware può aiutare a modificare i file o iniettare file dannosi e quindi riflasharli nella memoria, il che potrebbe essere utile per impiantare un backdoor nel dispositivo. Pertanto, ci sono numerose possibilità che possono essere sbloccate con il dumping del firmware.

### Programmatore e Lettore di EEPROM CH341A

Questo dispositivo è uno strumento economico per il dumping dei firmware dalle EEPROM e anche per riflasharli con file firmware. Questa è stata una scelta popolare per lavorare con i chip BIOS dei computer (che sono solo EEPROM). Questo dispositivo si collega tramite USB e ha bisogno di strumenti minimi per iniziare. Inoltre, di solito svolge il compito rapidamente, quindi può essere utile anche nell'accesso ai dispositivi fisici.

<img src="../../.gitbook/assets/board_image_ch341a.jpg" alt="disegno" larghezza="400" align="center"/>

Collega la memoria EEPROM al Programmatore CH341a e collega il dispositivo al computer. Nel caso in cui il dispositivo non venga rilevato, prova a installare i driver nel computer. Assicurati anche che la EEPROM sia collegata nella giusta orientazione (di solito, posiziona il Pin VCC in orientamento inverso al connettore USB) altrimenti, il software non sarà in grado di rilevare il chip. Fai riferimento al diagramma se necessario:

<img src="../../.gitbook/assets/connect_wires_ch341a.jpg" alt="disegno" larghezza="350"/>

<img src="../../.gitbook/assets/eeprom_plugged_ch341a.jpg" alt="disegno" larghezza="350"/>

Infine, utilizza software come flashrom, G-Flash (GUI), ecc. per il dumping del firmware. G-Flash è un tool GUI minimale veloce e rileva automaticamente la EEPROM. Questo può essere utile se il firmware deve essere estratto rapidamente, senza dover fare molte modifiche alla documentazione.

<img src="../../.gitbook/assets/connected_status_ch341a.jpg" alt="disegno" larghezza="350"/>

Dopo aver dumpato il firmware, l'analisi può essere fatta sui file binari. Strumenti come strings, hexdump, xxd, binwalk, ecc. possono essere utilizzati per estrarre molte informazioni sul firmware così come sull'intero sistema di file.

Per estrarre i contenuti dal firmware, può essere utilizzato binwalk. Binwalk analizza le firme esadecimali e identifica i file nel file binario ed è in grado di estrarli.
```
binwalk -e <filename>
```
Il <filename> può essere .bin o .rom a seconda degli strumenti e delle configurazioni utilizzati.

{% hint style="danger" %} Si noti che l'estrazione del firmware è un processo delicato e richiede molta pazienza. Qualsiasi manomissione può potenzialmente corrompere il firmware o addirittura cancellarlo completamente e rendere il dispositivo inutilizzabile. Si consiglia di studiare il dispositivo specifico prima di tentare di estrarre il firmware. {% endhint %}

### Bus Pirate + flashrom

![](<../../.gitbook/assets/image (907).png>)

Si noti che anche se il PINOUT del Bus Pirate indica i pin per **MOSI** e **MISO** da collegare a SPI, alcuni SPI possono indicare i pin come DI e DO. **MOSI -> DI, MISO -> DO**

![](<../../.gitbook/assets/image (357).png>)

In Windows o Linux è possibile utilizzare il programma [**`flashrom`**](https://www.flashrom.org/Flashrom) per eseguire il dump del contenuto della memoria flash eseguendo qualcosa del genere:
```bash
# In this command we are indicating:
# -VV Verbose
# -c <chip> The chip (if you know it better, if not, don'tindicate it and the program might be able to find it)
# -p <programmer> In this case how to contact th chip via the Bus Pirate
# -r <file> Image to save in the filesystem
flashrom -VV -c "W25Q64.V" -p buspirate_spi:dev=COM3 -r flash_content.img
```
<details>

<summary><strong>Impara l'hacking AWS da zero a eroe con</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (Esperto Red Team AWS di HackTricks)</strong></a><strong>!</strong></summary>

Altri modi per supportare HackTricks:

* Se vuoi vedere la tua **azienda pubblicizzata su HackTricks** o **scaricare HackTricks in PDF** Controlla i [**PIANI DI ABBONAMENTO**](https://github.com/sponsors/carlospolop)!
* Ottieni il [**merchandising ufficiale di PEASS & HackTricks**](https://peass.creator-spring.com)
* Scopri [**La Famiglia PEASS**](https://opensea.io/collection/the-peass-family), la nostra collezione di esclusive [**NFT**](https://opensea.io/collection/the-peass-family)
* **Unisciti al** 💬 [**gruppo Discord**](https://discord.gg/hRep4RUj7f) o al [**gruppo telegram**](https://t.me/peass) o **seguici** su **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Condividi i tuoi trucchi di hacking inviando PR ai** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repos di github.

</details>
