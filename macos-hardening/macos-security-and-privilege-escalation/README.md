# Sicurezza e Privilege Escalation su macOS

<details>

<summary><strong>Impara l'hacking su AWS da zero a eroe con</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (Esperto Red Team AWS di HackTricks)</strong></a><strong>!</strong></summary>

Altri modi per supportare HackTricks:

* Se vuoi vedere la tua **azienda pubblicizzata su HackTricks** o **scaricare HackTricks in PDF** Controlla i [**PIANI DI ABBONAMENTO**](https://github.com/sponsors/carlospolop)!
* Ottieni il [**merchandising ufficiale di PEASS & HackTricks**](https://peass.creator-spring.com)
* Scopri [**La Famiglia PEASS**](https://opensea.io/collection/the-peass-family), la nostra collezione di [**NFT esclusivi**](https://opensea.io/collection/the-peass-family)
* **Unisciti al** 💬 [**gruppo Discord**](https://discord.gg/hRep4RUj7f) o al [**gruppo telegram**](https://t.me/peass) o **seguici** su **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Condividi i tuoi trucchi di hacking inviando PR a** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repos di github.

</details>

<figure><img src="../../.gitbook/assets/image (380).png" alt=""><figcaption></figcaption></figure>

Unisciti al server [**HackenProof Discord**](https://discord.com/invite/N3FrSbmwdy) per comunicare con hacker esperti e cacciatori di bug bounty!

**Approfondimenti sull'Hacking**\
Coinvolgiti con contenuti che esplorano l'emozione e le sfide dell'hacking

**Notizie sull'Hacking in Tempo Reale**\
Resta aggiornato con il mondo dell'hacking in rapida evoluzione attraverso notizie e approfondimenti in tempo reale

**Ultimi Annunci**\
Resta informato sui nuovi bug bounty in arrivo e sugli aggiornamenti cruciali delle piattaforme

**Unisciti a noi su** [**Discord**](https://discord.com/invite/N3FrSbmwdy) e inizia a collaborare con i migliori hacker oggi stesso!

## Concetti di Base su MacOS

Se non sei familiare con macOS, dovresti iniziare imparando i concetti di base di macOS:

* File e permessi speciali di macOS:

{% content-ref url="macos-files-folders-and-binaries/" %}
[macos-files-folders-and-binaries](macos-files-folders-and-binaries/)
{% endcontent-ref %}

* Utenti comuni di macOS

{% content-ref url="macos-users.md" %}
[macos-users.md](macos-users.md)
{% endcontent-ref %}

* **AppleFS**

{% content-ref url="macos-applefs.md" %}
[macos-applefs.md](macos-applefs.md)
{% endcontent-ref %}

* L'**architettura** del k**ernel**

{% content-ref url="mac-os-architecture/" %}
[mac-os-architecture](mac-os-architecture/)
{% endcontent-ref %}

* Servizi e protocolli di rete n**etwork comuni di macOS**

{% content-ref url="macos-protocols.md" %}
[macos-protocols.md](macos-protocols.md)
{% endcontent-ref %}

* macOS **Open Source**: [https://opensource.apple.com/](https://opensource.apple.com/)
* Per scaricare un `tar.gz` cambia un URL come [https://opensource.apple.com/**source**/dyld/](https://opensource.apple.com/source/dyld/) in [https://opensource.apple.com/**tarballs**/dyld/**dyld-852.2.tar.gz**](https://opensource.apple.com/tarballs/dyld/dyld-852.2.tar.gz)

### MDM su MacOS

Nelle aziende i sistemi **macOS** sono molto probabilmente gestiti con un MDM. Pertanto, dal punto di vista di un attaccante è interessante sapere **come funziona**:

{% content-ref url="../macos-red-teaming/macos-mdm/" %}
[macos-mdm](../macos-red-teaming/macos-mdm/)
{% endcontent-ref %}

### MacOS - Ispezione, Debugging e Fuzzing

{% content-ref url="macos-apps-inspecting-debugging-and-fuzzing/" %}
[macos-apps-inspecting-debugging-and-fuzzing](macos-apps-inspecting-debugging-and-fuzzing/)
{% endcontent-ref %}

## Protezioni di Sicurezza su MacOS

{% content-ref url="macos-security-protections/" %}
[macos-security-protections](macos-security-protections/)
{% endcontent-ref %}

## Superficie di Attacco

### Permessi dei File

Se un **processo in esecuzione come root scrive** un file che può essere controllato da un utente, l'utente potrebbe sfruttarlo per **escalare i privilegi**.\
Ciò potrebbe verificarsi nelle seguenti situazioni:

* Il file utilizzato è stato già creato da un utente (di proprietà dell'utente)
* Il file utilizzato è scrivibile dall'utente a causa di un gruppo
* Il file utilizzato è all'interno di una directory di proprietà dell'utente (l'utente potrebbe creare il file)
* Il file utilizzato è all'interno di una directory di proprietà di root ma l'utente ha accesso in scrittura su di essa a causa di un gruppo (l'utente potrebbe creare il file)

Essere in grado di **creare un file** che verrà **utilizzato da root**, consente a un utente di **sfruttarne il contenuto** o addirittura creare **symlink/hardlink** per farlo puntare altrove.

Per questo tipo di vulnerabilità non dimenticare di **controllare gli installer `.pkg` vulnerabili**:

{% content-ref url="macos-files-folders-and-binaries/macos-installers-abuse.md" %}
[macos-installers-abuse.md](macos-files-folders-and-binaries/macos-installers-abuse.md)
{% endcontent-ref %}

### Estensione del File e Gestori di App degli URL scheme

App strane registrate dalle estensioni dei file potrebbero essere sfruttate e diverse applicazioni possono essere registrate per aprire protocolli specifici

{% content-ref url="macos-file-extension-apps.md" %}
[macos-file-extension-apps.md](macos-file-extension-apps.md)
{% endcontent-ref %}

## Privilege Escalation TCC / SIP su macOS

In macOS le **applicazioni e i binari possono avere permessi** per accedere a cartelle o impostazioni che li rendono più privilegiati di altri.

Pertanto, un attaccante che vuole compromettere con successo una macchina macOS dovrà **escalare i suoi privilegi TCC** (o addirittura **bypassare SIP**, a seconda delle sue esigenze).

Questi privilegi sono solitamente concessi sotto forma di **entitlements** con cui l'applicazione è firmata, oppure l'applicazione potrebbe aver richiesto alcuni accessi e dopo che l'**utente li ha approvati** possono essere trovati nei **database TCC**. Un altro modo in cui un processo può ottenere questi privilegi è essendo un **figlio di un processo** con quei **privilegi** poiché di solito vengono **ereditati**.

Segui questi link per trovare diversi modi per [**escalare i privilegi in TCC**](macos-security-protections/macos-tcc/#tcc-privesc-and-bypasses), per [**bypassare TCC**](macos-security-protections/macos-tcc/macos-tcc-bypasses/) e come in passato è stato [**bypassato SIP**](macos-security-protections/macos-sip.md#sip-bypasses).

## Privilege Escalation Tradizionale su macOS

Naturalmente, dal punto di vista delle red team dovresti essere interessato anche ad escalare a root. Controlla il seguente post per alcuni suggerimenti:

{% content-ref url="macos-privilege-escalation.md" %}
[macos-privilege-escalation.md](macos-privilege-escalation.md)
{% endcontent-ref %}
## Riferimenti

* [**OS X Incident Response: Scripting and Analysis**](https://www.amazon.com/OS-Incident-Response-Scripting-Analysis-ebook/dp/B01FHOHHVS)
* [**https://taomm.org/vol1/analysis.html**](https://taomm.org/vol1/analysis.html)
* [**https://github.com/NicolasGrimonpont/Cheatsheet**](https://github.com/NicolasGrimonpont/Cheatsheet)
* [**https://assets.sentinelone.com/c/sentinal-one-mac-os-?x=FvGtLJ**](https://assets.sentinelone.com/c/sentinal-one-mac-os-?x=FvGtLJ)
* [**https://www.youtube.com/watch?v=vMGiplQtjTY**](https://www.youtube.com/watch?v=vMGiplQtjTY)

<figure><img src="../../.gitbook/assets/image (380).png" alt=""><figcaption></figcaption></figure>

Unisciti al server [**HackenProof Discord**](https://discord.com/invite/N3FrSbmwdy) per comunicare con hacker esperti e cacciatori di bug!

**Approfondimenti sull'Hacking**\
Interagisci con contenuti che esplorano l'emozione e le sfide dell'hacking

**Notizie sull'Hacking in Tempo Reale**\
Resta aggiornato sul mondo dell'hacking frenetico attraverso notizie e approfondimenti in tempo reale

**Ultime Annunci**\
Rimani informato sui nuovi bug bounty in arrivo e sugli aggiornamenti cruciali delle piattaforme

**Unisciti a noi su** [**Discord**](https://discord.com/invite/N3FrSbmwdy) e inizia a collaborare con i migliori hacker oggi!

<details>

<summary><strong>Impara l'hacking su AWS da zero a eroe con</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Altri modi per supportare HackTricks:

* Se desideri vedere la tua **azienda pubblicizzata su HackTricks** o **scaricare HackTricks in PDF** controlla i [**PIANI DI ABBONAMENTO**](https://github.com/sponsors/carlospolop)!
* Ottieni il [**merchandising ufficiale di PEASS & HackTricks**](https://peass.creator-spring.com)
* Scopri [**The PEASS Family**](https://opensea.io/collection/the-peass-family), la nostra collezione esclusiva di [**NFT**](https://opensea.io/collection/the-peass-family)
* **Unisciti al** 💬 [**gruppo Discord**](https://discord.gg/hRep4RUj7f) o al [**gruppo telegram**](https://t.me/peass) o **seguici** su **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Condividi i tuoi trucchi di hacking inviando PR ai** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repos di github.

</details>
