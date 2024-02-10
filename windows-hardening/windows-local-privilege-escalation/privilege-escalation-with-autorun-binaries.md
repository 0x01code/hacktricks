# Επέκταση Προνομίων με τα Autoruns

<details>

<summary><strong>Μάθετε το χάκινγκ στο AWS από το μηδέν μέχρι τον ήρωα με το</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Άλλοι τρόποι για να υποστηρίξετε το HackTricks:

* Αν θέλετε να δείτε την **εταιρεία σας να διαφημίζεται στο HackTricks** ή να **κατεβάσετε το HackTricks σε μορφή PDF**, ελέγξτε τα [**ΣΧΕΔΙΑ ΣΥΝΔΡΟΜΗΣ**](https://github.com/sponsors/carlospolop)!
* Αποκτήστε το [**επίσημο PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Ανακαλύψτε [**την Οικογένεια PEASS**](https://opensea.io/collection/the-peass-family), τη συλλογή μας από αποκλειστικά [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Εγγραφείτε στη** 💬 [**ομάδα Discord**](https://discord.gg/hRep4RUj7f) ή στη [**ομάδα telegram**](https://t.me/peass) ή **ακολουθήστε** μας στο **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Μοιραστείτε τα χάκινγκ κόλπα σας υποβάλλοντας PRs στα** [**HackTricks**](https://github.com/carlospolop/hacktricks) και [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) αποθετήρια του github.

</details>

<img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt="" data-size="original">

Αν ενδιαφέρεστε για μια **καριέρα στο χάκινγκ** και για να χακεύσετε το αχακέυτο - **προσλαμβάνουμε!** (_απαιτείται άπταιστη γραπτή και προφορική γνώση της πολωνικής γλώσσας_).

{% embed url="https://www.stmcyber.com/careers" %}

## WMIC

Το **Wmic** μπορεί να χρησιμοποιηθεί για να εκτελέσει προγράμματα κατά την **εκκίνηση**. Δείτε ποια δυαδικά αρχεία έχουν προγραμματιστεί να εκτελούνται κατά την εκκίνηση με:
```bash
wmic startup get caption,command 2>nul & ^
Get-CimInstance Win32_StartupCommand | select Name, command, Location, User | fl
```
## Προγραμματισμένες εργασίες

Οι **εργασίες** μπορούν να προγραμματιστούν να τρέξουν με **συγκεκριμένη συχνότητα**. Δείτε ποια δυαδικά αρχεία έχουν προγραμματιστεί να τρέξουν με:
```bash
schtasks /query /fo TABLE /nh | findstr /v /i "disable deshab"
schtasks /query /fo LIST 2>nul | findstr TaskName
schtasks /query /fo LIST /v > schtasks.txt; cat schtask.txt | grep "SYSTEM\|Task To Run" | grep -B 1 SYSTEM
Get-ScheduledTask | where {$_.TaskPath -notlike "\Microsoft*"} | ft TaskName,TaskPath,State

#Schtask to give admin access
#You can also write that content on a bat file that is being executed by a scheduled task
schtasks /Create /RU "SYSTEM" /SC ONLOGON /TN "SchedPE" /TR "cmd /c net localgroup administrators user /add"
```
## Φάκελοι

Όλα τα δυαδικά αρχεία που βρίσκονται στους φακέλους **Startup θα εκτελούνται κατά την εκκίνηση**. Οι κοινοί φάκελοι εκκίνησης είναι αυτοί που αναφέρονται παρακάτω, αλλά ο φάκελος εκκίνησης καθορίζεται στο μητρώο. [Διαβάστε αυτό για να μάθετε πού.](privilege-escalation-with-autorun-binaries.md#startup-path)
```bash
dir /b "C:\Documents and Settings\All Users\Start Menu\Programs\Startup" 2>nul
dir /b "C:\Documents and Settings\%username%\Start Menu\Programs\Startup" 2>nul
dir /b "%programdata%\Microsoft\Windows\Start Menu\Programs\Startup" 2>nul
dir /b "%appdata%\Microsoft\Windows\Start Menu\Programs\Startup" 2>nul
Get-ChildItem "C:\Users\All Users\Start Menu\Programs\Startup"
Get-ChildItem "C:\Users\$env:USERNAME\Start Menu\Programs\Startup"
```
## Καταχώριση

{% hint style="info" %}
[Σημείωση από εδώ](https://answers.microsoft.com/en-us/windows/forum/all/delete-registry-key/d425ae37-9dcc-4867-b49c-723dcd15147f): Η καταχώριση **Wow6432Node** υποδεικνύει ότι χρησιμοποιείτε μια 64-μπιτη έκδοση των Windows. Το λειτουργικό σύστημα χρησιμοποιεί αυτό το κλειδί για να εμφανίσει μια ξεχωριστή προβολή του HKEY\_LOCAL\_MACHINE\SOFTWARE για 32-μπιτες εφαρμογές που εκτελούνται σε 64-μπιτες εκδόσεις των Windows.
{% endhint %}

### Εκτέλεση

Κοινώς γνωστές καταχωρίσεις AutoRun:

* `HKLM\Software\Microsoft\Windows\CurrentVersion\Run`
* `HKLM\Software\Microsoft\Windows\CurrentVersion\RunOnce`
* `HKLM\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Run`
* `HKLM\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\RunOnce`
* `HKCU\Software\Microsoft\Windows\CurrentVersion\Run`
* `HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce`
* `HKCU\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Run`
* `HKCU\Software\Wow6432Npde\Microsoft\Windows\CurrentVersion\RunOnce`
* `HKLM\Software\Microsoft\Windows NT\CurrentVersion\Terminal Server\Install\Software\Microsoft\Windows\CurrentVersion\Run`
* `HKLM\Software\Microsoft\Windows NT\CurrentVersion\Terminal Server\Install\Software\Microsoft\Windows\CurrentVersion\Runonce`
* `HKLM\Software\Microsoft\Windows NT\CurrentVersion\Terminal Server\Install\Software\Microsoft\Windows\CurrentVersion\RunonceEx`

Οι καταχωρίσεις του μητρώου που είναι γνωστές ως **Run** και **RunOnce** σχεδιάστηκαν για να εκτελούν αυτόματα προγράμματα κάθε φορά που ο χρήστης συνδέεται στο σύστημα. Η γραμμή εντολών που αντιστοιχεί στην τιμή δεδομένων του κλειδιού περιορίζεται σε 260 χαρακτήρες ή λιγότερο.

**Εκτέλεση υπηρεσιών** (μπορεί να ελέγχει την αυτόματη εκκίνηση των υπηρεσιών κατά την εκκίνηση):

* `HKLM\Software\Microsoft\Windows\CurrentVersion\RunServicesOnce`
* `HKCU\Software\Microsoft\Windows\CurrentVersion\RunServicesOnce`
* `HKLM\Software\Microsoft\Windows\CurrentVersion\RunServices`
* `HKCU\Software\Microsoft\Windows\CurrentVersion\RunServices`
* `HKLM\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\RunServicesOnce`
* `HKCU\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\RunServicesOnce`
* `HKLM\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\RunServices`
* `HKCU\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\RunServices`

**RunOnceEx:**

* `HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\RunOnceEx`
* `HKEY_LOCAL_MACHINE\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\RunOnceEx`

Στα Windows Vista και σε νεότερες εκδόσεις, οι καταχωρίσεις των μητρώων **Run** και **RunOnce** δεν δημιουργούνται αυτόματα. Οι καταχωρίσεις σε αυτά τα κλειδιά μπορούν είτε να εκκινήσουν προγράμματα απευθείας είτε να τα καθορίσουν ως εξαρτήσεις. Για παράδειγμα, για να φορτώσετε ένα αρχείο DLL κατά την είσοδο, μπορείτε να χρησιμοποιήσετε το κλειδί μητρώου **RunOnceEx** μαζί με ένα κλειδί "Depend". Αυτό επιδεικνύεται προσθέτοντας μια καταχώριση στο μητρώο για να εκτελεί το "C:\\temp\\evil.dll" κατά την εκκίνηση του συστήματος:
```
reg add HKLM\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\RunOnceEx\\0001\\Depend /v 1 /d "C:\\temp\\evil.dll"
```
{% hint style="info" %}
**Εκμετάλλευση 1**: Εάν μπορείτε να γράψετε μέσα σε οποιοδήποτε από τα αναφερόμενα κλειδιά του μητρώου μέσα στο **HKLM**, μπορείτε να αναβαθμίσετε τα δικαιώματα όταν συνδεθεί ένας διαφορετικός χρήστης.
{% endhint %}

{% hint style="info" %}
**Εκμετάλλευση 2**: Εάν μπορείτε να αντικαταστήσετε οποιοδήποτε από τα εκτελέσιμα αρχεία που αναφέρονται σε οποιοδήποτε κλειδί του μητρώου μέσα στο **HKLM**, μπορείτε να τροποποιήσετε αυτό το εκτελέσιμο αρχείο με ένα παρασκηνιακό πρόγραμμα όταν συνδεθεί ένας διαφορετικός χρήστης και να αναβαθμίσετε τα δικαιώματα.
{% endhint %}
```bash
#CMD
reg query HKLM\Software\Microsoft\Windows\CurrentVersion\Run
reg query HKLM\Software\Microsoft\Windows\CurrentVersion\RunOnce
reg query HKLM\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Run
reg query HKLM\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\RunOnce
reg query HKCU\Software\Microsoft\Windows\CurrentVersion\Run
reg query HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce
reg query HKCU\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Run
reg query HKCU\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\RunOnce
reg query HKLM\Software\Microsoft\Windows NT\CurrentVersion\Terminal Server\Install\Software\Microsoft\Windows\CurrentVersion\Run
reg query HKLM\Software\Microsoft\Windows NT\CurrentVersion\Terminal Server\Install\Software\Microsoft\Windows\CurrentVersion\RunOnce
reg query HKLM\Software\Microsoft\Windows NT\CurrentVersion\Terminal Server\Install\Software\Microsoft\Windows\CurrentVersion\RunE

reg query HKLM\Software\Microsoft\Windows\CurrentVersion\RunServicesOnce
reg query HKCU\Software\Microsoft\Windows\CurrentVersion\RunServicesOnce
reg query HKLM\Software\Microsoft\Windows\CurrentVersion\RunServices
reg query HKCU\Software\Microsoft\Windows\CurrentVersion\RunServices
reg query HKLM\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\RunServicesOnce
reg query HKCU\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\RunServicesOnce
reg query HKLM\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\RunServices
reg query HKCU\Software\Wow5432Node\Microsoft\Windows\CurrentVersion\RunServices

reg query HKLM\Software\Microsoft\Windows\RunOnceEx
reg query HKLM\Software\Wow6432Node\Microsoft\Windows\RunOnceEx
reg query HKCU\Software\Microsoft\Windows\RunOnceEx
reg query HKCU\Software\Wow6432Node\Microsoft\Windows\RunOnceEx

#PowerShell
Get-ItemProperty -Path 'Registry::HKLM\Software\Microsoft\Windows\CurrentVersion\Run'
Get-ItemProperty -Path 'Registry::HKLM\Software\Microsoft\Windows\CurrentVersion\RunOnce'
Get-ItemProperty -Path 'Registry::HKLM\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Run'
Get-ItemProperty -Path 'Registry::HKLM\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\RunOnce'
Get-ItemProperty -Path 'Registry::HKCU\Software\Microsoft\Windows\CurrentVersion\Run'
Get-ItemProperty -Path 'Registry::HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce'
Get-ItemProperty -Path 'Registry::HKCU\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Run'
Get-ItemProperty -Path 'Registry::HKCU\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\RunOnce'
Get-ItemProperty -Path 'Registry::HKLM\Software\Microsoft\Windows NT\CurrentVersion\Terminal Server\Install\Software\Microsoft\Windows\CurrentVersion\Run'
Get-ItemProperty -Path 'Registry::HKLM\Software\Microsoft\Windows NT\CurrentVersion\Terminal Server\Install\Software\Microsoft\Windows\CurrentVersion\RunOnce'
Get-ItemProperty -Path 'Registry::HKLM\Software\Microsoft\Windows NT\CurrentVersion\Terminal Server\Install\Software\Microsoft\Windows\CurrentVersion\RunE'

Get-ItemProperty -Path 'Registry::HKLM\Software\Microsoft\Windows\CurrentVersion\RunServicesOnce'
Get-ItemProperty -Path 'Registry::HKCU\Software\Microsoft\Windows\CurrentVersion\RunServicesOnce'
Get-ItemProperty -Path 'Registry::HKLM\Software\Microsoft\Windows\CurrentVersion\RunServices'
Get-ItemProperty -Path 'Registry::HKCU\Software\Microsoft\Windows\CurrentVersion\RunServices'
Get-ItemProperty -Path 'Registry::HKLM\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\RunServicesOnce'
Get-ItemProperty -Path 'Registry::HKCU\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\RunServicesOnce'
Get-ItemProperty -Path 'Registry::HKLM\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\RunServices'
Get-ItemProperty -Path 'Registry::HKCU\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\RunServices'

Get-ItemProperty -Path 'Registry::HKLM\Software\Microsoft\Windows\RunOnceEx'
Get-ItemProperty -Path 'Registry::HKLM\Software\Wow6432Node\Microsoft\Windows\RunOnceEx'
Get-ItemProperty -Path 'Registry::HKCU\Software\Microsoft\Windows\RunOnceEx'
Get-ItemProperty -Path 'Registry::HKCU\Software\Wow6432Node\Microsoft\Windows\RunOnceEx'
```
### Διαδρομή Εκκίνησης

* `HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\User Shell Folders`
* `HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders`
* `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders`
* `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\User Shell Folders`

Οι συντομεύσεις που τοποθετούνται στον φάκελο **Startup** θα εκτελούν αυτόματα υπηρεσίες ή εφαρμογές κατά την είσοδο του χρήστη ή την επανεκκίνηση του συστήματος. Η τοποθεσία του φακέλου **Startup** καθορίζεται στο μητρώο για τις εμβέλειες του **Local Machine** και του **Current User**. Αυτό σημαίνει ότι οποιαδήποτε συντόμευση προστίθεται σε αυτές τις συγκεκριμένες τοποθεσίες του **Startup** θα εξασφαλίσει ότι η συνδεδεμένη υπηρεσία ή πρόγραμμα θα ξεκινήσει μετά τη διαδικασία εισόδου ή επανεκκίνησης, καθιστώντας το μια απλή μέθοδο για τον προγραμματισμό της αυτόματης εκτέλεσης προγραμμάτων.

{% hint style="info" %}
Εάν μπορείτε να αντικαταστήσετε οποιονδήποτε \[User] Shell Folder κάτω από το **HKLM**, θα μπορείτε να τον κατευθύνετε προς έναν φάκελο που ελέγχετε εσείς και να τοποθετήσετε ένα backdoor που θα εκτελείται κάθε φορά που ένας χρήστης συνδέεται στο σύστημα, ανεβάζοντας τα δικαιώματα.
{% endhint %}
```bash
reg query "HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\User Shell Folders" /v "Common Startup"
reg query "HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders" /v "Common Startup"
reg query "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders" /v "Common Startup"
reg query "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\User Shell Folders" /v "Common Startup"

Get-ItemProperty -Path 'Registry::HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\User Shell Folders' -Name "Common Startup"
Get-ItemProperty -Path 'Registry::HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders' -Name "Common Startup"
Get-ItemProperty -Path 'Registry::HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders' -Name "Common Startup"
Get-ItemProperty -Path 'Registry::HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\User Shell Folders' -Name "Common Startup"
```
### Κλειδιά Winlogon

`HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon`

Συνήθως, το κλειδί **Userinit** είναι ρυθμισμένο στο **userinit.exe**. Ωστόσο, εάν αυτό το κλειδί τροποποιηθεί, το καθορισμένο εκτελέσιμο αρχείο θα εκτελεστεί επίσης από το **Winlogon** κατά την είσοδο του χρήστη. Αντίστοιχα, το κλειδί **Shell** προορίζεται να δείχνει στο **explorer.exe**, το προεπιλεγμένο περιβάλλον εργασίας για τα Windows.
```bash
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v "Userinit"
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v "Shell"
Get-ItemProperty -Path 'Registry::HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon' -Name "Userinit"
Get-ItemProperty -Path 'Registry::HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon' -Name "Shell"
```
{% hint style="info" %}
Εάν μπορείτε να αντικαταστήσετε την τιμή του μητρώου ή το δυαδικό αρχείο, θα μπορέσετε να αναβαθμίσετε τα δικαιώματα.
{% endhint %}

### Ρυθμίσεις πολιτικής

* `HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer`
* `HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer`

Ελέγξτε το κλειδί **Run**.
```bash
reg query "HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" /v "Run"
reg query "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" /v "Run"
Get-ItemProperty -Path 'Registry::HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer' -Name "Run"
Get-ItemProperty -Path 'Registry::HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer' -Name "Run"
```
### AlternateShell

### Αλλαγή της Εντολής Ασφαλούς Λειτουργίας με Εντολή Εναλλακτικού Κέλυφους

Στο Μητρώο των Windows κάτω από `HKLM\SYSTEM\CurrentControlSet\Control\SafeBoot`, υπάρχει μια προεπιλεγμένη τιμή με την ονομασία **`AlternateShell`** που είναι ρυθμισμένη σε `cmd.exe`. Αυτό σημαίνει ότι όταν επιλέγετε την "Ασφαλή Λειτουργία με Εντολή Εναλλακτικού Κέλυφους" κατά την εκκίνηση (πατώντας F8), χρησιμοποιείται το `cmd.exe`. Ωστόσο, είναι δυνατό να ρυθμίσετε τον υπολογιστή σας να ξεκινά αυτόματα σε αυτήν τη λειτουργία χωρίς να χρειάζεται να πατήσετε F8 και να την επιλέξετε χειροκίνητα.

Βήματα για τη δημιουργία μιας επιλογής εκκίνησης για αυτόματη έναρξη στην "Ασφαλή Λειτουργία με Εντολή Εναλλακτικού Κέλυφους":

1. Αλλάξτε τα χαρακτηριστικά του αρχείου `boot.ini` για να αφαιρέσετε τις σημαίες "μόνο για ανάγνωση", "σύστημα" και "κρυφό": `attrib c:\boot.ini -r -s -h`
2. Ανοίξτε το `boot.ini` για επεξεργασία.
3. Εισαγάγετε μια γραμμή όπως: `multi(0)disk(0)rdisk(0)partition(1)\WINDOWS="Microsoft Windows XP Professional" /fastdetect /SAFEBOOT:MINIMAL(ALTERNATESHELL)`
4. Αποθηκεύστε τις αλλαγές στο `boot.ini`.
5. Επαναφέρετε τα αρχικά χαρακτηριστικά του αρχείου: `attrib c:\boot.ini +r +s +h`

- **Εκμετάλλευση 1:** Η αλλαγή του κλειδιού μητρώου **AlternateShell** επιτρέπει την προσαρμογή του προσαρμοσμένου κέλυφους εντολών, πιθανώς για μη εξουσιοδοτημένη πρόσβαση.
- **Εκμετάλλευση 2 (Δικαιώματα Εγγραφής στο PATH):** Έχοντας δικαιώματα εγγραφής σε οποιοδήποτε μέρος της μεταβλητής συστήματος **PATH**, ειδικά πριν το `C:\Windows\system32`, σας επιτρέπει να εκτελέσετε ένα προσαρμοσμένο `cmd.exe`, το οποίο μπορεί να είναι μια πίσω πόρτα εάν το σύστημα ξεκινήσει σε Ασφαλή Λειτουργία.
- **Εκμετάλλευση 3 (Δικαιώματα Εγγραφής στο PATH και boot.ini):** Η εγγραφή στο αρχείο `boot.ini` επιτρέπει την αυτόματη έναρξη σε Ασφαλή Λειτουργία, διευκολύνοντας τη μη εξουσιοδοτημένη πρόσβαση κατά την επόμενη επανεκκίνηση.

Για να ελέγξετε την τρέχουσα ρύθμιση του **AlternateShell**, χρησιμοποιήστε αυτές τις εντολές:
```bash
reg query HKLM\SYSTEM\CurrentControlSet\Control\SafeBoot /v AlternateShell
Get-ItemProperty -Path 'Registry::HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SafeBoot' -Name 'AlternateShell'
```
### Εγκατεστημένος Συστατικό

Το Active Setup είναι μια λειτουργία στα Windows που **εκκινεί πριν ο περιβάλλον εργασίας της επιφάνειας εργασίας φορτωθεί πλήρως**. Δίνει προτεραιότητα στην εκτέλεση ορισμένων εντολών, οι οποίες πρέπει να ολοκληρωθούν πριν συνεχιστεί η σύνδεση του χρήστη. Αυτή η διαδικασία πραγματοποιείται ακόμα και πριν ενεργοποιηθούν άλλες καταχωρήσεις εκκίνησης, όπως αυτές στις ενότητες του μητρώου Run ή RunOnce.

Το Active Setup διαχειρίζεται μέσω των ακόλουθων κλειδιών του μητρώου:

- `HKLM\SOFTWARE\Microsoft\Active Setup\Installed Components`
- `HKLM\SOFTWARE\Wow6432Node\Microsoft\Active Setup\Installed Components`
- `HKCU\SOFTWARE\Microsoft\Active Setup\Installed Components`
- `HKCU\SOFTWARE\Wow6432Node\Microsoft\Active Setup\Installed Components`

Μέσα σε αυτά τα κλειδιά, υπάρχουν διάφορα υποκλειδιά, το καθένα αντιστοιχεί σε ένα συγκεκριμένο συστατικό. Οι τιμές των κλειδιών που είναι ιδιαίτερα ενδιαφέρουσες περιλαμβάνουν:

- **IsInstalled:**
- Το `0` υποδηλώνει ότι η εντολή του συστατικού δεν θα εκτελεστεί.
- Το `1` σημαίνει ότι η εντολή θα εκτελεστεί μία φορά για κάθε χρήστη, που είναι η προεπιλεγμένη συμπεριφορά αν η τιμή `IsInstalled` λείπει.
- **StubPath:** Ορίζει την εντολή που θα εκτελεστεί από το Active Setup. Μπορεί να είναι οποιαδήποτε έγκυρη γραμμή εντολών, όπως η εκκίνηση του `notepad`.

**Σημειώσεις Ασφαλείας:**

- Η τροποποίηση ή η εγγραφή σε ένα κλειδί όπου η τιμή του **`IsInstalled`** έχει οριστεί σε `"1"` με ένα συγκεκριμένο **`StubPath`** μπορεί να οδηγήσει σε μη εξουσιοδοτημένη εκτέλεση εντολών, πιθανώς για ανέλιξη προνομιούχων δικαιωμάτων.
- Η τροποποίηση του δυαδικού αρχείου που αναφέρεται σε οποιαδήποτε τιμή **`StubPath`** μπορεί επίσης να επιτύχει ανέλιξη προνομιούχων δικαιωμάτων, εφόσον υπάρχουν επαρκή δικαιώματα.

Για να επιθεωρήσετε τις ρυθμίσεις **`StubPath`** σε συστατικά του Active Setup, μπορείτε να χρησιμοποιήσετε τις παρακάτω εντολές:
```bash
reg query "HKLM\SOFTWARE\Microsoft\Active Setup\Installed Components" /s /v StubPath
reg query "HKCU\SOFTWARE\Microsoft\Active Setup\Installed Components" /s /v StubPath
reg query "HKLM\SOFTWARE\Wow6432Node\Microsoft\Active Setup\Installed Components" /s /v StubPath
reg query "HKCU\SOFTWARE\Wow6432Node\Microsoft\Active Setup\Installed Components" /s /v StubPath
```
### Αντικείμενα Βοηθού Προγράμματος Περιήγησης

### Επισκόπηση των Αντικειμένων Βοηθού Προγράμματος Περιήγησης (BHOs)

Τα Αντικείμενα Βοηθού Προγράμματος Περιήγησης (BHOs) είναι DLL πρόσθετα που προσθέτουν επιπλέον λειτουργίες στον Internet Explorer της Microsoft. Φορτώνονται στον Internet Explorer και τον Windows Explorer κάθε φορά που ξεκινούν. Ωστόσο, η εκτέλεσή τους μπορεί να αποκλειστεί θέτοντας το κλειδί **NoExplorer** σε 1, εμποδίζοντας τη φόρτωσή τους με περιπτώσεις του Windows Explorer.

Τα BHOs είναι συμβατά με τα Windows 10 μέσω του Internet Explorer 11, αλλά δεν υποστηρίζονται στο Microsoft Edge, το προεπιλεγμένο πρόγραμμα περιήγησης σε νεότερες εκδόσεις των Windows.

Για να εξερευνήσετε τα BHOs που έχουν καταχωρηθεί σε ένα σύστημα, μπορείτε να επιθεωρήσετε τα ακόλουθα κλειδιά του μητρώου:

- `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Browser Helper Objects`
- `HKLM\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Explorer\Browser Helper Objects`

Κάθε BHO αντιπροσωπεύεται από το **CLSID** του στο μητρώο, λειτουργώντας ως μοναδικό αναγνωριστικό. Λεπτομερείς πληροφορίες για κάθε CLSID μπορούν να βρεθούν στο `HKLM\SOFTWARE\Classes\CLSID\{<CLSID>}`.

Για την ανάκτηση των BHOs από το μητρώο, μπορούν να χρησιμοποιηθούν οι παρακάτω εντολές:
```bash
reg query "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Browser Helper Objects" /s
reg query "HKLM\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Explorer\Browser Helper Objects" /s
```
### Επεκτάσεις Internet Explorer

* `HKLM\Software\Microsoft\Internet Explorer\Extensions`
* `HKLM\Software\Wow6432Node\Microsoft\Internet Explorer\Extensions`

Σημειώστε ότι το μητρώο θα περιέχει 1 νέο μητρώο για κάθε dll και θα αναπαριστάται από το **CLSID**. Μπορείτε να βρείτε τις πληροφορίες του CLSID στο `HKLM\SOFTWARE\Classes\CLSID\{<CLSID>}`

### Οδηγοί γραμματοσειράς

* `HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Font Drivers`
* `HKLM\SOFTWARE\WOW6432Node\Microsoft\Windows NT\CurrentVersion\Font Drivers`
```bash
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Font Drivers"
reg query "HKLM\SOFTWARE\Wow6432Node\Microsoft\Windows NT\CurrentVersion\Font Drivers"
Get-ItemProperty -Path 'Registry::HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Font Drivers'
Get-ItemProperty -Path 'Registry::HKLM\SOFTWARE\Wow6432Node\Microsoft\Windows NT\CurrentVersion\Font Drivers'
```
### Άνοιγμα Εντολής

* `HKLM\SOFTWARE\Classes\htmlfile\shell\open\command`
* `HKLM\SOFTWARE\Wow6432Node\Classes\htmlfile\shell\open\command`
```bash
reg query "HKLM\SOFTWARE\Classes\htmlfile\shell\open\command" /v ""
reg query "HKLM\SOFTWARE\Wow6432Node\Classes\htmlfile\shell\open\command" /v ""
Get-ItemProperty -Path 'Registry::HKLM\SOFTWARE\Classes\htmlfile\shell\open\command' -Name ""
Get-ItemProperty -Path 'Registry::HKLM\SOFTWARE\Wow6432Node\Classes\htmlfile\shell\open\command' -Name ""
```
### Επιλογές Εκτέλεσης Αρχείων Εικόνας

The Image File Execution Options (IFEO) is a Windows feature that allows developers to specify additional debugging options for a specific executable. However, this feature can also be abused by attackers to escalate privileges on a compromised system.

Οι Επιλογές Εκτέλεσης Αρχείων Εικόνας (Image File Execution Options - IFEO) είναι μια δυνατότητα των Windows που επιτρέπει στους προγραμματιστές να καθορίσουν επιπλέον επιλογές αποσφαλμάτωσης για ένα συγκεκριμένο εκτελέσιμο αρχείο. Ωστόσο, αυτή η δυνατότητα μπορεί επίσης να καταχραστεί από επιτιθέμενους για να αναβαθμίσουν τα δικαιώματα σε ένα παραβιασμένο σύστημα.
```
HKLM\Software\Microsoft\Windows NT\CurrentVersion\Image File Execution Options
HKLM\Software\Microsoft\Wow6432Node\Windows NT\CurrentVersion\Image File Execution Options
```
## SysInternals

Σημειώστε ότι όλες οι ιστοσελίδες όπου μπορείτε να βρείτε αυτόματες εκτελέσεις έχουν ήδη αναζητηθεί από το [winpeas.exe](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS/winPEASexe). Ωστόσο, για μια πιο ολοκληρωμένη λίστα αρχείων που εκτελούνται αυτόματα, μπορείτε να χρησιμοποιήσετε το [autoruns](https://docs.microsoft.com/en-us/sysinternals/downloads/autoruns) από το SysInternals:
```
autorunsc.exe -m -nobanner -a * -ct /accepteula
```
## Περισσότερα

**Βρείτε περισσότερα Autoruns όπως καταχωρητές στο [https://www.microsoftpressstore.com/articles/article.aspx?p=2762082\&seqNum=2](https://www.microsoftpressstore.com/articles/article.aspx?p=2762082\&seqNum=2)**

## Αναφορές

* [https://resources.infosecinstitute.com/common-malware-persistence-mechanisms/#gref](https://resources.infosecinstitute.com/common-malware-persistence-mechanisms/#gref)
* [https://attack.mitre.org/techniques/T1547/001/](https://attack.mitre.org/techniques/T1547/001/)
* [https://www.microsoftpressstore.com/articles/article.aspx?p=2762082\&seqNum=2](https://www.microsoftpressstore.com/articles/article.aspx?p=2762082\&seqNum=2)
* [https://www.itprotoday.com/cloud-computing/how-can-i-add-boot-option-starts-alternate-shell](https://www.itprotoday.com/cloud-computing/how-can-i-add-boot-option-starts-alternate-shell)

<img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt="" data-size="original">

Εάν ενδιαφέρεστε για μια **καριέρα στο χάκινγκ** και θέλετε να χακεύετε το αδύνατο - **προσλαμβάνουμε!** (_απαιτείται άπταιστη γραπτή και προφορική γνώση της πολωνικής γλώσσας_).

{% embed url="https://www.stmcyber.com/careers" %}

<details>

<summary><strong>Μάθετε το χάκινγκ του AWS από το μηδέν μέχρι τον ήρωα με το</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Άλλοι τρόποι υποστήριξης του HackTricks:

* Εάν θέλετε να δείτε την **εταιρεία σας να διαφημίζεται στο HackTricks** ή να **κατεβάσετε το HackTricks σε μορφή PDF** ελέγξτε τα [**ΣΧΕΔΙΑ ΣΥΝΔΡΟΜΗΣ**](https://github.com/sponsors/carlospolop)!
* Αποκτήστε το [**επίσημο PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Ανακαλύψτε [**The PEASS Family**](https://opensea.io/collection/the-peass-family), τη συλλογή μας από αποκλειστικά [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Εγγραφείτε στη** 💬 [**ομάδα Discord**](https://discord.gg/hRep4RUj7f) ή στη [**ομάδα telegram**](https://t.me/peass) ή **ακολουθήστε** μας στο **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Μοιραστείτε τα χάκινγκ κόλπα σας υποβάλλοντας PRs στα** [**HackTricks**](https://github.com/carlospolop/hacktricks) και [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) αποθετήρια του github.

</details>
