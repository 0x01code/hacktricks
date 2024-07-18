# Fuga dai KIOSKs

{% hint style="success" %}
Impara e pratica l'Hacking su AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Impara e pratica l'Hacking su GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Sostieni HackTricks</summary>

* Controlla i [**piani di abbonamento**](https://github.com/sponsors/carlospolop)!
* **Unisciti al** 💬 [**gruppo Discord**](https://discord.gg/hRep4RUj7f) o al [**gruppo telegram**](https://t.me/peass) o **seguici** su **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Condividi trucchi di hacking inviando PR a** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repos di github.

</details>
{% endhint %}

#### [WhiteIntel](https://whiteintel.io)

<figure><img src="../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) è un motore di ricerca alimentato dal **dark web** che offre funzionalità **gratuite** per verificare se un'azienda o i suoi clienti sono stati **compromessi** da **malware ruba-informazioni**.

Il loro obiettivo principale è contrastare le violazioni di account e gli attacchi ransomware derivanti da malware che rubano informazioni.

Puoi controllare il loro sito web e provare il loro motore **gratuitamente** su:

{% embed url="https://whiteintel.io" %}

---

## Controlla il dispositivo fisico

|   Componente   | Azione                                                               |
| ------------- | -------------------------------------------------------------------- |
| Pulsante di accensione  | Spegnere e riaccendere il dispositivo potrebbe esporre la schermata di avvio      |
| Cavo di alimentazione   | Verifica se il dispositivo si riavvia quando l'alimentazione viene interrotta brevemente   |
| Porte USB     | Collega una tastiera fisica con più scorciatoie                        |
| Ethernet      | La scansione di rete o lo sniffing potrebbero consentire ulteriori sfruttamenti             |


## Controlla le possibili azioni all'interno dell'applicazione GUI

I **Dialoghi Comuni** sono quelle opzioni di **salvataggio di un file**, **apertura di un file**, selezione di un carattere, di un colore... La maggior parte di essi **offrirà una funzionalità completa di Esplora risorse**. Ciò significa che potrai accedere alle funzionalità di Esplora risorse se puoi accedere a queste opzioni:

* Chiudi/Chiudi come
* Apri/Apri con
* Stampa
* Esporta/Importa
* Cerca
* Scansiona

Dovresti verificare se puoi:

* Modificare o creare nuovi file
* Creare collegamenti simbolici
* Accedere ad aree restritte
* Eseguire altre app

### Esecuzione di Comandi

Forse **utilizzando l'opzione `Apri con`** puoi aprire/eseguire qualche tipo di shell.

#### Windows

Ad esempio _cmd.exe, command.com, Powershell/Powershell ISE, mmc.exe, at.exe, taskschd.msc..._ trova più binari che possono essere utilizzati per eseguire comandi (e compiere azioni inaspettate) qui: [https://lolbas-project.github.io/](https://lolbas-project.github.io)

#### \*NIX \_\_

_bash, sh, zsh..._ Più qui: [https://gtfobins.github.io/](https://gtfobins.github.io)

## Windows

### Eludere le restrizioni del percorso

* **Variabili d'ambiente**: Ci sono molte variabili d'ambiente che puntano a un determinato percorso
* **Altri protocolli**: _about:, data:, ftp:, file:, mailto:, news:, res:, telnet:, view-source:_
* **Collegamenti simbolici**
* **Scorciatoie**: CTRL+N (apri nuova sessione), CTRL+R (Esegui Comandi), CTRL+SHIFT+ESC (Task Manager), Windows+E (apri esplora risorse), CTRL-B, CTRL-I (Preferiti), CTRL-H (Cronologia), CTRL-L, CTRL-O (File/Apri Dialogo), CTRL-P (Dialogo di Stampa), CTRL-S (Salva come)
* Menu amministrativo nascosto: CTRL-ALT-F8, CTRL-ESC-F9
* **URI della Shell**: _shell:Strumenti Amministrativi, shell:Libreria Documenti, shell:Biblioteche, shell:Profili Utente, shell:Personale, shell:Cartella Ricerca, shell:Sistemashell:Cartelle Posta di Rete, shell:Invia a, shell:Profili Utenti, shell:Strumenti Amministrativi Comuni, shell:Risorse del Computer, shell:Cartella Internet_
* **Percorsi UNC**: Percorsi per connettersi a cartelle condivise. Dovresti provare a connetterti al C$ della macchina locale ("\\\127.0.0.1\c$\Windows\System32")
* **Altri percorsi UNC:**

| UNC                       | UNC            | UNC                  |
| ------------------------- | -------------- | -------------------- |
| %ALLUSERSPROFILE%         | %APPDATA%      | %CommonProgramFiles% |
| %COMMONPROGRAMFILES(x86)% | %COMPUTERNAME% | %COMSPEC%            |
| %HOMEDRIVE%               | %HOMEPATH%     | %LOCALAPPDATA%       |
| %LOGONSERVER%             | %PATH%         | %PATHEXT%            |
| %ProgramData%             | %ProgramFiles% | %ProgramFiles(x86)%  |
| %PROMPT%                  | %PSModulePath% | %Public%             |
| %SYSTEMDRIVE%             | %SYSTEMROOT%   | %TEMP%               |
| %TMP%                     | %USERDOMAIN%   | %USERNAME%           |
| %USERPROFILE%             | %WINDIR%       |                      | 

### Scarica i Tuoi Binari

Console: [https://sourceforge.net/projects/console/](https://sourceforge.net/projects/console/)\
Esplora risorse: [https://sourceforge.net/projects/explorerplus/files/Explorer%2B%2B/](https://sourceforge.net/projects/explorerplus/files/Explorer%2B%2B/)\
Editor del Registro di sistema: [https://sourceforge.net/projects/uberregedit/](https://sourceforge.net/projects/uberregedit/)

### Accesso al filesystem dal browser

| PERCORSO                | PERCORSO              | PERCORSO               | PERCORSO                |
| ------------------- | ----------------- | ------------------ | ------------------- |
| File:/C:/windows    | File:/C:/windows/ | File:/C:/windows\\ | File:/C:\windows    |
| File:/C:\windows\\  | File:/C:\windows/ | File://C:/windows  | File://C:/windows/  |
| File://C:/windows\\ | File://C:\windows | File://C:\windows/ | File://C:\windows\\ |
| C:/windows          | C:/windows/       | C:/windows\\       | C:\windows          |
| C:\windows\\        | C:\windows/       | %WINDIR%           | %TMP%               |
| %TEMP%              | %SYSTEMDRIVE%     | %SYSTEMROOT%       | %APPDATA%           |
| %HOMEDRIVE%         | %HOMESHARE        |                    | <p><br></p>         |
### Scorciatoie

* Sticky Keys – Premi SHIFT 5 volte
* Mouse Keys – SHIFT+ALT+NUMLOCK
* Alto Contrasto – SHIFT+ALT+PRINTSCN
* Toggle Keys – Tieni premuto NUMLOCK per 5 secondi
* Filter Keys – Tieni premuto il tasto destro SHIFT per 12 secondi
* WINDOWS+F1 – Ricerca Windows
* WINDOWS+D – Mostra Desktop
* WINDOWS+E – Avvia Esplora risorse di Windows
* WINDOWS+R – Esegui
* WINDOWS+U – Centro facilità di accesso
* WINDOWS+F – Cerca
* SHIFT+F10 – Menu contestuale
* CTRL+SHIFT+ESC – Task Manager
* CTRL+ALT+DEL – Schermata di avvio nelle versioni più recenti di Windows
* F1 – Aiuto F3 – Cerca
* F6 – Barra degli indirizzi
* F11 – Attiva/disattiva schermo intero in Internet Explorer
* CTRL+H – Cronologia di Internet Explorer
* CTRL+T – Internet Explorer – Nuova scheda
* CTRL+N – Internet Explorer – Nuova pagina
* CTRL+O – Apri File
* CTRL+S – Salva CTRL+N – Nuovo RDP / Citrix

### Swipe

* Scorri dal lato sinistro a quello destro per vedere tutte le finestre aperte, minimizzando l'applicazione KIOSK e accedendo direttamente a tutto il sistema operativo;
* Scorri dal lato destro a sinistra per aprire il Centro notifiche, minimizzando l'applicazione KIOSK e accedendo direttamente a tutto il sistema operativo;
* Scorri dal bordo superiore per rendere visibile la barra del titolo di un'app aperta in modalità schermo intero;
* Scorri verso l'alto dal basso per mostrare la barra delle applicazioni in un'app a schermo intero.

### Trucchi per Internet Explorer

#### 'Barra degli strumenti delle immagini'

È una barra degli strumenti che appare in alto a sinistra dell'immagine quando viene cliccata. Sarai in grado di Salvare, Stampare, Inviare per posta, Aprire "Le mie immagini" in Esplora risorse. Il Kiosk deve utilizzare Internet Explorer.

#### Protocollo Shell

Digita questi URL per ottenere una visualizzazione di Esplora risorse:

* `shell:Strumenti di amministrazione`
* `shell:Libreria documenti`
* `shell:Biblioteche`
* `shell:Profili utente`
* `shell:Personale`
* `shell:Cartella home di ricerca`
* `shell:Cartelle posta di rete`
* `shell:Invia a`
* `shell:Profili utente`
* `shell:Strumenti di amministrazione comuni`
* `shell:Cartella computer`
* `shell:Cartella Internet`
* `Shell:Profilo`
* `Shell:Programmi`
* `Shell:Sistema`
* `Shell:Pannello di controllo`
* `Shell:Windows`
* `shell:::{21EC2020-3AEA-1069-A2DD-08002B30309D}` --> Pannello di controllo
* `shell:::{20D04FE0-3AEA-1069-A2D8-08002B30309D}` --> Il mio computer
* `shell:::{{208D2C60-3AEA-1069-A2D7-08002B30309D}}` --> Le mie posizioni di rete
* `shell:::{871C5380-42A0-1069-A2EA-08002B30309D}` --> Internet Explorer

### Mostra le estensioni dei file

Controlla questa pagina per ulteriori informazioni: [https://www.howtohaven.com/system/show-file-extensions-in-windows-explorer.shtml](https://www.howtohaven.com/system/show-file-extensions-in-windows-explorer.shtml)

## Trucchi per i browser

Backup delle versioni di iKat:

[http://swin.es/k/](http://swin.es/k/)\
[http://www.ikat.kronicd.net/](http://www.ikat.kronicd.net)\\

Crea una finestra di dialogo comune utilizzando JavaScript e accedi all'esplora risorse: `document.write('<input/type=file>')`\
Fonte: https://medium.com/@Rend\_/give-me-a-browser-ill-give-you-a-shell-de19811defa0

## iPad

### Gesture e pulsanti

* Scorri verso l'alto con quattro (o cinque) dita / Doppio tocco sul pulsante Home: Per visualizzare la vista multitasking e cambiare app
* Scorri da una parte all'altra con quattro o cinque dita: Per passare all'app successiva/precedente
* Pizzica lo schermo con cinque dita / Tocca il pulsante Home / Scorri verso l'alto con 1 dito dal basso dello schermo in modo rapido verso l'alto: Per accedere alla Home
* Scorri con un dito dal basso dello schermo solo 1-2 pollici (lentamente): Comparirà il dock
* Scorri verso il basso dall'alto del display con 1 dito: Per visualizzare le notifiche
* Scorri verso il basso con 1 dito nell'angolo in alto a destra dello schermo: Per vedere il centro di controllo di iPad Pro
* Scorri con 1 dito dal lato sinistro dello schermo 1-2 pollici: Per visualizzare la vista di oggi
* Scorri rapidamente con 1 dito dal centro dello schermo verso destra o sinistra: Per passare all'app successiva/precedente
* Premi e tieni premuto il pulsante Accensione/Spegnimento nell'angolo superiore destro dell'iPad + Sposta il cursore Spegni tutto il modo a destra: Per spegnere
* Premi il pulsante Accensione/Spegnimento nell'angolo superiore destro dell'iPad e il pulsante Home per alcuni secondi: Per forzare uno spegnimento
* Premi il pulsante Accensione/Spegnimento nell'angolo superiore destro dell'iPad e il pulsante Home rapidamente: Per fare uno screenshot che comparirà nell'angolo in basso a sinistra del display. Premi entrambi i pulsanti contemporaneamente molto brevemente, se li tieni premuti per alcuni secondi verrà eseguito uno spegnimento forzato.

### Scorciatoie

Dovresti avere una tastiera per iPad o un adattatore per tastiera USB. Qui verranno mostrate solo le scorciatoie che potrebbero aiutare a uscire dall'applicazione.

| Tasto | Nome         |
| --- | ------------ |
| ⌘   | Comando      |
| ⌥   | Opzione (Alt) |
| ⇧   | Maiusc        |
| ↩   | Invio       |
| ⇥   | Tab          |
| ^   | Controllo      |
| ←   | Freccia sinistra   |
| →   | Freccia destra  |
| ↑   | Freccia su     |
| ↓   | Freccia giù   |

#### Scorciatoie di sistema

Queste scorciatoie sono per le impostazioni visive e audio, a seconda dell'uso dell'iPad.

| Scorciatoia | Azione                                                                         |
| -------- | ------------------------------------------------------------------------------ |
| F1       | Abbassa luminosità schermo                                                                    |
| F2       | Aumenta luminosità schermo                                                                |
| F7       | Torna indietro di una canzone                                                                  |
| F8       | Riproduci/metti in pausa                                                                     |
| F9       | Salta canzone                                                                      |
| F10      | Silenzia                                                                           |
| F11      | Diminuisci volume                                                                |
| F12      | Aumenta volume                                                                |
| ⌘ Spazio  | Visualizza un elenco di lingue disponibili; per sceglierne una, tocca di nuovo il tasto spazio. |

#### Navigazione iPad

| Scorciatoia                                           | Azione                                                  |
| -------------------------------------------------- | ------------------------------------------------------- |
| ⌘H                                                 | Vai alla Home                                              |
| ⌘⇧H (Command-Shift-H)                              | Vai alla Home                                              |
| ⌘ (Spazio)                                          | Apri Spotlight                                          |
| ⌘⇥ (Command-Tab)                                   | Elenca le ultime dieci app utilizzate                                 |
| ⌘\~                                                | Vai all'ultima app                                       |
| ⌘⇧3 (Command-Shift-3)                              | Screenshot (compare in basso a sinistra per salvare o agire su di esso) |
| ⌘⇧4                                                | Screenshot e aprilo nell'editor                    |
| Premi e tieni premuto ⌘                                   | Elenco delle scorciatoie disponibili per l'app                 |
| ⌘⌥D (Command-Option/Alt-D)                         | Mostra il dock                                      |
| ^⌥H (Control-Option-H)                             | Pulsante Home                                             |
| ^⌥H H (Control-Option-H-H)                         | Mostra la barra multitasking                                      |
| ^⌥I (Control-Option-i)                             | Selettore di elementi                                            |
| Escape                                             | Pulsante Indietro                                             |
| → (Freccia destra)                                    | Prossimo elemento                                               |
| ← (Freccia sinistra)                                     | Elemento precedente                                           |
| ↑↓ (Freccia su, Freccia giù)                          | Tocca contemporaneamente l'elemento selezionato                        |
| ⌥ ↓ (Opzione-Freccia giù)                            | Scorri verso il basso                                             |
| ⌥↑ (Opzione-Freccia su)                               | Scorri verso l'alto                                               |
| ⌥← o ⌥→ (Opzione-Freccia sinistra o Opzione-Freccia destra) | Scorri a sinistra o a destra                                    |
| ^⌥S (Control-Option-S)                             | Attiva o disattiva la lettura del testo VoiceOver                         |
| ⌘⇧⇥ (Command-Shift-Tab)                            | Passa all'app precedente                              |
| ⌘⇥ (Command-Tab)                                   | Torna all'app originale                         |
| ←+→, poi Opzione + ← o Opzione+→                   | Naviga attraverso il Dock                                   |
#### Scorciatoie Safari

| Scorciatoia              | Azione                                           |
| ------------------------ | ------------------------------------------------ |
| ⌘L (Command-L)           | Apri posizione                                   |
| ⌘T                       | Apri una nuova scheda                            |
| ⌘W                       | Chiudi la scheda corrente                        |
| ⌘R                       | Aggiorna la scheda corrente                      |
| ⌘.                       | Interrompi il caricamento della scheda corrente   |
| ^⇥                       | Passa alla scheda successiva                     |
| ^⇧⇥ (Control-Shift-Tab)  | Passa alla scheda precedente                     |
| ⌘L                       | Seleziona l'input di testo/campo URL per modificarlo |
| ⌘⇧T (Command-Shift-T)    | Apre l'ultima scheda chiusa (può essere utilizzato più volte) |
| ⌘\[                      | Torna indietro di una pagina nella cronologia di navigazione |
| ⌘]                       | Vai avanti di una pagina nella cronologia di navigazione |
| ⌘⇧R                      | Attiva la modalità Lettore                        |

#### Scorciatoie Mail

| Scorciatoia                | Azione                       |
| -------------------------- | ---------------------------- |
| ⌘L                        | Apri posizione               |
| ⌘T                        | Apri una nuova scheda        |
| ⌘W                        | Chiudi la scheda corrente    |
| ⌘R                        | Aggiorna la scheda corrente  |
| ⌘.                        | Interrompi il caricamento della scheda corrente |
| ⌘⌥F (Command-Option/Alt-F) | Cerca nella tua casella di posta |

## Riferimenti

* [https://www.macworld.com/article/2975857/6-only-for-ipad-gestures-you-need-to-know.html](https://www.macworld.com/article/2975857/6-only-for-ipad-gestures-you-need-to-know.html)
* [https://www.tomsguide.com/us/ipad-shortcuts,news-18205.html](https://www.tomsguide.com/us/ipad-shortcuts,news-18205.html)
* [https://thesweetsetup.com/best-ipad-keyboard-shortcuts/](https://thesweetsetup.com/best-ipad-keyboard-shortcuts/)
* [http://www.iphonehacks.com/2018/03/ipad-keyboard-shortcuts.html](http://www.iphonehacks.com/2018/03/ipad-keyboard-shortcuts.html)

#### [WhiteIntel](https://whiteintel.io)

<figure><img src="../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) è un motore di ricerca alimentato dal **dark web** che offre funzionalità **gratuite** per verificare se un'azienda o i suoi clienti sono stati **compromessi** da **malware ruba-informazioni**.

Il loro obiettivo principale è combattere le violazioni degli account e gli attacchi ransomware derivanti da malware che ruba informazioni.

Puoi visitare il loro sito web e provare il loro motore **gratuitamente** su:

{% embed url="https://whiteintel.io" %}

{% hint style="success" %}
Impara e pratica l'Hacking su AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Impara e pratica l'Hacking su GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Supporta HackTricks</summary>

* Controlla i [**piani di abbonamento**](https://github.com/sponsors/carlospolop)!
* **Unisciti al** 💬 [**gruppo Discord**](https://discord.gg/hRep4RUj7f) o al [**gruppo telegram**](https://t.me/peass) o **seguici** su **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Condividi trucchi di hacking inviando PR ai** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repos di Github.

</details>
{% endhint %}
