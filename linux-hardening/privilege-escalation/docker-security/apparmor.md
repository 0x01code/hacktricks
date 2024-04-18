# AppArmor

<details>

<summary><strong>Impara l'hacking AWS da zero a ero con</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (Esperto Red Team AWS di HackTricks)</strong></a><strong>!</strong></summary>

Altri modi per supportare HackTricks:

* Se vuoi vedere la tua **azienda pubblicizzata su HackTricks** o **scaricare HackTricks in PDF** Controlla i [**PIANI DI ABBONAMENTO**](https://github.com/sponsors/carlospolop)!
* Ottieni il [**merchandising ufficiale PEASS & HackTricks**](https://peass.creator-spring.com)
* Scopri [**La Famiglia PEASS**](https://opensea.io/collection/the-peass-family), la nostra collezione di [**NFT esclusivi**](https://opensea.io/collection/the-peass-family)
* **Unisciti al** 💬 [**gruppo Discord**](https://discord.gg/hRep4RUj7f) o al [**gruppo telegram**](https://t.me/peass) o **seguici** su **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Condividi i tuoi trucchi di hacking inviando PR a** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repos di github.

</details>

### [WhiteIntel](https://whiteintel.io)

<figure><img src="/.gitbook/assets/image (1224).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) è un motore di ricerca alimentato dal **dark web** che offre funzionalità **gratuite** per verificare se un'azienda o i suoi clienti sono stati **compromessi** da **malware ruba informazioni**.

Il loro obiettivo principale di WhiteIntel è combattere i takeover degli account e gli attacchi ransomware derivanti da malware che rubano informazioni.

Puoi visitare il loro sito web e provare il loro motore **gratuitamente** su:

{% embed url="https://whiteintel.io" %}

---

## Informazioni di Base

AppArmor è un **miglioramento del kernel progettato per limitare le risorse disponibili ai programmi attraverso profili per programma**, implementando efficacemente il Controllo di Accesso Obbligatorio (MAC) legando gli attributi di controllo degli accessi direttamente ai programmi anziché agli utenti. Questo sistema opera **caricando i profili nel kernel**, di solito durante l'avvio, e questi profili indicano a quali risorse un programma può accedere, come connessioni di rete, accesso a socket grezzi e autorizzazioni sui file.

Ci sono due modalità operative per i profili di AppArmor:

- **Modalità di Applicazione**: Questa modalità applica attivamente le politiche definite all'interno del profilo, bloccando azioni che violano tali politiche e registrando qualsiasi tentativo di violarle attraverso sistemi come syslog o auditd.
- **Modalità di Lamentele**: A differenza della modalità di applicazione, la modalità di lamentele non blocca le azioni che vanno contro le politiche del profilo. Invece, registra questi tentativi come violazioni di politica senza imporre restrizioni.

### Componenti di AppArmor

- **Modulo Kernel**: Responsabile dell'applicazione delle politiche.
- **Politiche**: Specificano le regole e le restrizioni per il comportamento del programma e l'accesso alle risorse.
- **Parser**: Carica le politiche nel kernel per l'applicazione o la segnalazione.
- **Utilità**: Questi sono programmi in modalità utente che forniscono un'interfaccia per interagire e gestire AppArmor.

### Percorso dei Profili

I profili di AppArmor di solito sono salvati in _**/etc/apparmor.d/**_\
Con `sudo aa-status` sarai in grado di elencare i binari che sono limitati da qualche profilo. Se sostituisci il carattere "/" con un punto nel percorso di ciascun binario elencato, otterrai il nome del profilo di apparmor all'interno della cartella menzionata.

Ad esempio, un profilo **apparmor** per _/usr/bin/man_ sarà situato in _/etc/apparmor.d/usr.bin.man_

### Comandi
```bash
aa-status     #check the current status
aa-enforce    #set profile to enforce mode (from disable or complain)
aa-complain   #set profile to complain mode (from diable or enforcement)
apparmor_parser #to load/reload an altered policy
aa-genprof    #generate a new profile
aa-logprof    #used to change the policy when the binary/program is changed
aa-mergeprof  #used to merge the policies
```
## Creazione di un profilo

* Per indicare l'eseguibile interessato, sono consentiti **percorsi assoluti e caratteri jolly** (per la selezione dei file) per specificare i file.
* Per indicare l'accesso che il binario avrà sui **file**, possono essere utilizzati i seguenti **controlli di accesso**:
* **r** (lettura)
* **w** (scrittura)
* **m** (mappatura in memoria come eseguibile)
* **k** (blocco file)
* **l** (creazione di collegamenti rigidi)
* **ix** (per eseguire un altro programma con il nuovo programma che eredita la policy)
* **Px** (eseguire sotto un altro profilo, dopo aver pulito l'ambiente)
* **Cx** (eseguire sotto un profilo figlio, dopo aver pulito l'ambiente)
* **Ux** (eseguire senza vincoli, dopo aver pulito l'ambiente)
* Le **variabili** possono essere definite nei profili e possono essere manipolate dall'esterno del profilo. Ad esempio: @{PROC} e @{HOME} (aggiungere #include \<tunables/global> al file del profilo)
* Le **regole di negazione sono supportate per sovrascrivere le regole di autorizzazione**.

### aa-genprof

Per iniziare facilmente a creare un profilo, apparmor può aiutarti. È possibile fare in modo che **apparmor ispezioni le azioni eseguite da un binario e poi ti permetta di decidere quali azioni desideri consentire o negare**.\
Basta eseguire:
```bash
sudo aa-genprof /path/to/binary
```
Quindi, in una console diversa esegui tutte le azioni che di solito eseguirebbe il binario:
```bash
/path/to/binary -a dosomething
```
Quindi, nella prima console premi "**s**" e poi nelle azioni registrate indica se vuoi ignorare, consentire, o altro. Quando hai finito premi "**f**" e il nuovo profilo verrà creato in _/etc/apparmor.d/percorso.per.binario_

{% hint style="info" %}
Utilizzando i tasti freccia puoi selezionare cosa desideri consentire/negare/altro
{% endhint %}

### aa-easyprof

Puoi anche creare un modello di un profilo apparmor di un binario con:
```bash
sudo aa-easyprof /path/to/binary
# vim:syntax=apparmor
# AppArmor policy for binary
# ###AUTHOR###
# ###COPYRIGHT###
# ###COMMENT###

#include <tunables/global>

# No template variables specified

"/path/to/binary" {
#include <abstractions/base>

# No abstractions specified

# No policy groups specified

# No read paths specified

# No write paths specified
}
```
{% hint style="info" %}
Si noti che per impostazione predefinita in un profilo creato nulla è consentito, quindi tutto è negato. Sarà necessario aggiungere righe come `/etc/passwd r,` per consentire alla binaria di leggere `/etc/passwd`, ad esempio.
{% endhint %}

Puoi quindi **applicare** il nuovo profilo con
```bash
sudo apparmor_parser -a /etc/apparmor.d/path.to.binary
```
### Modifica di un profilo dai log

Il seguente strumento leggerà i log e chiederà all'utente se desidera permettere alcune delle azioni proibite rilevate:
```bash
sudo aa-logprof
```
{% hint style="info" %}
Utilizzando i tasti freccia è possibile selezionare ciò che si desidera consentire/negare/o qualsiasi altra azione
{% endhint %}

### Gestione di un Profilo
```bash
#Main profile management commands
apparmor_parser -a /etc/apparmor.d/profile.name #Load a new profile in enforce mode
apparmor_parser -C /etc/apparmor.d/profile.name #Load a new profile in complain mode
apparmor_parser -r /etc/apparmor.d/profile.name #Replace existing profile
apparmor_parser -R /etc/apparmor.d/profile.name #Remove profile
```
## Registri

Esempio di registri **AUDIT** e **DENIED** da _/var/log/audit/audit.log_ dell'eseguibile **`service_bin`**:
```bash
type=AVC msg=audit(1610061880.392:286): apparmor="AUDIT" operation="getattr" profile="/bin/rcat" name="/dev/pts/1" pid=954 comm="service_bin" requested_mask="r" fsuid=1000 ouid=1000
type=AVC msg=audit(1610061880.392:287): apparmor="DENIED" operation="open" profile="/bin/rcat" name="/etc/hosts" pid=954 comm="service_bin" requested_mask="r" denied_mask="r" fsuid=1000 ouid=0
```
Puoi ottenere queste informazioni anche utilizzando:
```bash
sudo aa-notify -s 1 -v
Profile: /bin/service_bin
Operation: open
Name: /etc/passwd
Denied: r
Logfile: /var/log/audit/audit.log

Profile: /bin/service_bin
Operation: open
Name: /etc/hosts
Denied: r
Logfile: /var/log/audit/audit.log

AppArmor denials: 2 (since Wed Jan  6 23:51:08 2021)
For more information, please see: https://wiki.ubuntu.com/DebuggingApparmor
```
## Apparmor in Docker

Nota come il profilo **docker-profile** di docker venga caricato per impostazione predefinita:
```bash
sudo aa-status
apparmor module is loaded.
50 profiles are loaded.
13 profiles are in enforce mode.
/sbin/dhclient
/usr/bin/lxc-start
/usr/lib/NetworkManager/nm-dhcp-client.action
/usr/lib/NetworkManager/nm-dhcp-helper
/usr/lib/chromium-browser/chromium-browser//browser_java
/usr/lib/chromium-browser/chromium-browser//browser_openjdk
/usr/lib/chromium-browser/chromium-browser//sanitized_helper
/usr/lib/connman/scripts/dhclient-script
docker-default
```
Di default il profilo **Apparmor docker-default** è generato da [https://github.com/moby/moby/tree/master/profiles/apparmor](https://github.com/moby/moby/tree/master/profiles/apparmor)

**docker-default profile Summary**:

* **Accesso** a tutta la **rete**
* **Nessuna capacità** è definita (Tuttavia, alcune capacità verranno incluse dalle regole di base di base, ad es. #include \<abstractions/base>)
* **Scrittura** su qualsiasi file **/proc** non è **permessa**
* Altre **sottodirectory**/**file** di /**proc** e /**sys** hanno accesso **negato** in lettura/scrittura/blocco/link/esecuzione
* **Montaggio** non è **permesso**
* **Ptrace** può essere eseguito solo su un processo confinato dallo **stesso profilo apparmor**

Una volta che **esegui un container docker** dovresti vedere il seguente output:
```bash
1 processes are in enforce mode.
docker-default (825)
```
Nota che **apparmor bloccherà persino i privilegi delle capabilities** concessi al container per impostazione predefinita. Ad esempio, sarà in grado di **bloccare il permesso di scrittura all'interno di /proc anche se la capability SYS\_ADMIN è stata concessa** perché per impostazione predefinita il profilo apparmor di docker nega questo accesso:
```bash
docker run -it --cap-add SYS_ADMIN --security-opt seccomp=unconfined ubuntu /bin/bash
echo "" > /proc/stat
sh: 1: cannot create /proc/stat: Permission denied
```
Devi **disabilitare apparmor** per eludere le sue restrizioni:
```bash
docker run -it --cap-add SYS_ADMIN --security-opt seccomp=unconfined --security-opt apparmor=unconfined ubuntu /bin/bash
```
Nota che per impostazione predefinita **AppArmor** vieterà anche al container di montare cartelle dall'interno anche con la capacità SYS\_ADMIN.

Nota che puoi **aggiungere/rimuovere** **capacità** al container Docker (questo sarà comunque limitato da metodi di protezione come **AppArmor** e **Seccomp**):

* `--cap-add=SYS_ADMIN` conferisce la capacità `SYS_ADMIN`
* `--cap-add=ALL` conferisce tutte le capacità
* `--cap-drop=ALL --cap-add=SYS_PTRACE` elimina tutte le capacità e conferisce solo `SYS_PTRACE`

{% hint style="info" %}
Di solito, quando **scopri** di avere una **capacità privilegiata** disponibile **all'interno** di un **container docker** ma una parte dell'**exploit non funziona**, ciò sarà perché **AppArmor di Docker lo sta impedendo**.
{% endhint %}

### Esempio

(Esempio da [**qui**](https://sreeninet.wordpress.com/2016/03/06/docker-security-part-2docker-engine/))

Per illustrare la funzionalità di AppArmor, ho creato un nuovo profilo Docker "mydocker" con la seguente riga aggiunta:
```
deny /etc/* w,   # deny write for all files directly in /etc (not in a subdir)
```
Per attivare il profilo, dobbiamo fare quanto segue:
```
sudo apparmor_parser -r -W mydocker
```
Per elencare i profili, possiamo eseguire il seguente comando. Il comando qui sotto elenca il mio nuovo profilo AppArmor.
```
$ sudo apparmor_status  | grep mydocker
mydocker
```
Come mostrato di seguito, otteniamo un errore quando cerchiamo di modificare "/etc/" poiché il profilo di AppArmor impedisce l'accesso in scrittura a "/etc".
```
$ docker run --rm -it --security-opt apparmor:mydocker -v ~/haproxy:/localhost busybox chmod 400 /etc/hostname
chmod: /etc/hostname: Permission denied
```
### AppArmor Docker Bypass1

È possibile trovare quale **profilo apparmor sta eseguendo un contenitore** utilizzando:
```bash
docker inspect 9d622d73a614 | grep lowpriv
"AppArmorProfile": "lowpriv",
"apparmor=lowpriv"
```
Quindi, puoi eseguire la seguente riga per **trovare il profilo esatto in uso**:
```bash
find /etc/apparmor.d/ -name "*lowpriv*" -maxdepth 1 2>/dev/null
```
### Bypass di AppArmor Docker

AppArmor è basato sui percorsi, ciò significa che anche se potrebbe proteggere i file all'interno di una directory come `/proc`, se puoi configurare come verrà eseguito il container, potresti montare la directory proc dell'host all'interno di `/host/proc` e non sarà più protetta da AppArmor.

### Bypass di AppArmor Shebang

In [questo bug](https://bugs.launchpad.net/apparmor/+bug/1911431) puoi vedere un esempio di come, anche se stai impedendo a perl di essere eseguito con determinate risorse, se crei uno script shell specificando nella prima riga `#!/usr/bin/perl` e esegui direttamente il file, sarai in grado di eseguire ciò che desideri. Esempio:
```perl
echo '#!/usr/bin/perl
use POSIX qw(strftime);
use POSIX qw(setuid);
POSIX::setuid(0);
exec "/bin/sh"' > /tmp/test.pl
chmod +x /tmp/test.pl
/tmp/test.pl
```
### [WhiteIntel](https://whiteintel.io)

<figure><img src="/.gitbook/assets/image (1224).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) è un motore di ricerca alimentato dal **dark web** che offre funzionalità **gratuite** per verificare se un'azienda o i suoi clienti sono stati **compromessi** da **malware ruba-informazioni**.

Il loro obiettivo principale è combattere i takeover di account e gli attacchi ransomware derivanti da malware che rubano informazioni.

Puoi visitare il loro sito web e provare il loro motore gratuitamente su:

{% embed url="https://whiteintel.io" %}

<details>

<summary><strong>Impara l'hacking di AWS da zero a eroe con</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Altri modi per supportare HackTricks:

* Se desideri vedere la tua **azienda pubblicizzata su HackTricks** o **scaricare HackTricks in PDF** Controlla i [**PIANI DI ABBONAMENTO**](https://github.com/sponsors/carlospolop)!
* Ottieni il [**merchandising ufficiale di PEASS & HackTricks**](https://peass.creator-spring.com)
* Scopri [**The PEASS Family**](https://opensea.io/collection/the-peass-family), la nostra collezione di esclusive [**NFT**](https://opensea.io/collection/the-peass-family)
* **Unisciti al** 💬 [**gruppo Discord**](https://discord.gg/hRep4RUj7f) o al [**gruppo telegram**](https://t.me/peass) o **seguici** su **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Condividi i tuoi trucchi di hacking inviando PR ai** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repository di Github.

</details>
