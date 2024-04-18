<details>

<summary><strong>Impara l'hacking AWS da zero a eroe con</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (Esperto Red Team AWS di HackTricks)</strong></a><strong>!</strong></summary>

Altri modi per supportare HackTricks:

* Se vuoi vedere la tua **azienda pubblicizzata su HackTricks** o **scaricare HackTricks in PDF** Controlla i [**PIANI DI ABBONAMENTO**](https://github.com/sponsors/carlospolop)!
* Ottieni il [**merchandising ufficiale di PEASS & HackTricks**](https://peass.creator-spring.com)
* Scopri [**La Famiglia PEASS**](https://opensea.io/collection/the-peass-family), la nostra collezione di [**NFT esclusivi**](https://opensea.io/collection/the-peass-family)
* **Unisciti al** 💬 [**gruppo Discord**](https://discord.gg/hRep4RUj7f) o al [**gruppo telegram**](https://t.me/peass) o **seguici** su **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Condividi i tuoi trucchi di hacking inviando PR a** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repos di github.

</details>

## WhiteIntel

<figure><img src=".gitbook/assets/image (1224).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) è un motore di ricerca alimentato dal **dark web** che offre funzionalità **gratuite** per verificare se un'azienda o i suoi clienti sono stati **compromessi** da **malware ruba-informazioni**.

Il loro obiettivo principale di WhiteIntel è combattere le violazioni degli account e gli attacchi ransomware derivanti da malware che rubano informazioni.

Puoi visitare il loro sito web e provare il loro motore gratuitamente su:

{% embed url="https://whiteintel.io" %}

---

# Riassunto dell'attacco

Immagina un server che **firma** alcuni **dati** **aggiungendo** un **segreto** a dei dati noti in chiaro e quindi hashando quei dati. Se conosci:

* **La lunghezza del segreto** (questo può essere anche forzato da un determinato intervallo di lunghezza)
* **I dati in chiaro**
* **L'algoritmo (e che è vulnerabile a questo attacco)**
* **Il padding è noto**
* Di solito ne viene utilizzato uno predefinito, quindi se sono soddisfatte le altre 3 condizioni, anche questo lo è
* Il padding varia a seconda della lunghezza del segreto+dati, ecco perché è necessaria la lunghezza del segreto

Allora, è possibile per un **attaccante** **aggiungere** **dati** e **generare** una firma valida per i **dati precedenti + dati aggiunti**.

## Come?

Fondamentalmente gli algoritmi vulnerabili generano gli hash prima **hashando un blocco di dati**, e poi, **dal** **hash creato precedentemente** (stato), **aggiungono il blocco successivo di dati** e lo **hashano**.

Immagina che il segreto sia "segreto" e i dati siano "dati", l'MD5 di "segretodati" è 6036708eba0d11f6ef52ad44e8b74d5b.\
Se un attaccante vuole aggiungere la stringa "aggiungi" può:

* Generare un MD5 di 64 "A"
* Cambiare lo stato dell'hash inizializzato precedentemente in 6036708eba0d11f6ef52ad44e8b74d5b
* Aggiungere la stringa "aggiungi"
* Concludere l'hash e l'hash risultante sarà un **valido per "segreto" + "dati" + "padding" + "aggiungi"**

## **Strumento**

{% embed url="https://github.com/iagox86/hash_extender" %}

## Riferimenti

Puoi trovare questo attacco ben spiegato in [https://blog.skullsecurity.org/2012/everything-you-need-to-know-about-hash-length-extension-attacks](https://blog.skullsecurity.org/2012/everything-you-need-to-know-about-hash-length-extension-attacks)

## WhiteIntel

<figure><img src=".gitbook/assets/image (1224).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) è un motore di ricerca alimentato dal **dark web** che offre funzionalità **gratuite** per verificare se un'azienda o i suoi clienti sono stati **compromessi** da **malware ruba-informazioni**.

Il loro obiettivo principale di WhiteIntel è combattere le violazioni degli account e gli attacchi ransomware derivanti da malware che rubano informazioni.

Puoi visitare il loro sito web e provare il loro motore gratuitamente su:

{% embed url="https://whiteintel.io" %}

<details>
