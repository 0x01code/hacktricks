# macOS Απορρόφηση Μνήμης

{% hint style="success" %}
Μάθετε & εξασκηθείτε στο AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**Εκπαίδευση HackTricks AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Μάθετε & εξασκηθείτε στο GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**Εκπαίδευση HackTricks GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Υποστηρίξτε το HackTricks</summary>

* Ελέγξτε τα [**σχέδια συνδρομής**](https://github.com/sponsors/carlospolop)!
* **Εγγραφείτε** στην 💬 [**ομάδα Discord**](https://discord.gg/hRep4RUj7f) ή στην [**ομάδα telegram**](https://t.me/peass) ή **ακολουθήστε** μας στο **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Μοιραστείτε κόλπα χάκερ υποβάλλοντας PRs στα** [**HackTricks**](https://github.com/carlospolop/hacktricks) και [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) αποθετήρια στο GitHub.

</details>
{% endhint %}

### [WhiteIntel](https://whiteintel.io)

<figure><img src="../../../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) είναι μια μηχανή αναζήτησης που τροφοδοτείται από το **dark web** και προσφέρει **δωρεάν** λειτουργίες για να ελέγξετε αν μια εταιρεία ή οι πελάτες της έχουν **διαρρεύσει** από **κλέφτες κακόβουλων λογισμικών**.

Ο κύριος στόχος του WhiteIntel είναι η καταπολέμηση των αναλήψεων λογαριασμών και των επιθέσεων ransomware που προκύπτουν από κακόβουλα λογισμικά που κλέβουν πληροφορίες.

Μπορείτε να ελέγξετε την ιστοσελίδα τους και να δοκιμάσετε τη μηχανή τους δωρεάν στο:

{% embed url="https://whiteintel.io" %}

***

## Αρχεία Αποτυχίας

### Αρχεία Swap

Τα αρχεία swap, όπως το `/private/var/vm/swapfile0`, λειτουργούν ως **μνήμες cache όταν η φυσική μνήμη είναι γεμάτη**. Όταν δεν υπάρχει άλλος χώρος στη φυσική μνήμη, τα δεδομένα μεταφέρονται σε ένα αρχείο swap και στη συνέχεια επαναφέρονται στη φυσική μνήμη όταν χρειάζεται. Μπορεί να υπάρχουν πολλά αρχεία swap, με ονόματα όπως swapfile0, swapfile1 κλπ.

### Εικόνα Αδράνειας

Το αρχείο που βρίσκεται στη διαδρομή `/private/var/vm/sleepimage` είναι κρίσιμο κατά τη διάρκεια της **λειτουργίας αδράνειας**. **Τα δεδομένα από τη μνήμη αποθηκεύονται σε αυτό το αρχείο όταν το OS X μπαίνει σε λειτουργία αδράνειας**. Κατά την αφύπνιση του υπολογιστή, το σύστημα ανακτά τα δεδομένα μνήμης από αυτό το αρχείο, επιτρέποντας στον χρήστη να συνεχίσει από το σημείο που σταμάτησε.

Αξίζει να σημειωθεί ότι σε μοντέρνα συστήματα MacOS, αυτό το αρχείο είναι τυπικά κρυπτογραφημένο για λόγους ασφαλείας, κάτι που δυσκολεύει την ανάκτηση.

* Για να ελέγξετε αν η κρυπτογράφηση είναι ενεργοποιημένη για το sleepimage, μπορείτε να εκτελέσετε την εντολή `sysctl vm.swapusage`. Αυτό θα δείξει αν το αρχείο είναι κρυπτογραφημένο.

### Καταγραφές Πίεσης Μνήμης

Ένα άλλο σημαντικό αρχείο σχετικό με τη μνήμη στα συστήματα MacOS είναι το **αρχείο καταγραφής πίεσης μνήμης**. Αυτά τα αρχεία καταγραφής βρίσκονται στο `/var/log` και περιέχουν λεπτομερείς πληροφορίες σχετικά με τη χρήση μνήμης του συστήματος και τα γεγονότα πίεσης. Μπορούν να είναι ιδιαίτερα χρήσιμα για τη διάγνωση θεμάτων που σχετίζονται με τη μνήμη ή την κατανόηση του τρόπου με τον οποίο το σύστημα διαχειρίζεται τη μνήμη με τον χρόνο.

## Απορρόφηση μνήμης με το osxpmem

Για να απορροφήσετε τη μνήμη σε μια μηχανή MacOS, μπορείτε να χρησιμοποιήσετε το [**osxpmem**](https://github.com/google/rekall/releases/download/v1.5.1/osxpmem-2.1.post4.zip).

**Σημείωση**: Οι παρακάτω οδηγίες θα λειτουργήσουν μόνο για Mac με αρχιτεκτονική Intel. Αυτό το εργαλείο είναι τώρα αρχειοθετημένο και η τελευταία έκδοση ήταν το 2017. Το δυαδικό που λήφθηκε χρησιμοποιώντας τις οδηγίες παρακάτω στοχεύει σε chips Intel καθώς το Apple Silicon δεν υπήρχε το 2017. Μπορεί να είναι δυνατόν να μεταγλωττίσετε το δυαδικό για αρχιτεκτονική arm64 αλλά θα πρέπει να το δοκιμάσετε μόνοι σας.
```bash
#Dump raw format
sudo osxpmem.app/osxpmem --format raw -o /tmp/dump_mem

#Dump aff4 format
sudo osxpmem.app/osxpmem -o /tmp/dump_mem.aff4
```
Εάν βρείτε αυτό το σφάλμα: `osxpmem.app/MacPmem.kext failed to load - (libkern/kext) authentication failure (file ownership/permissions); check the system/kernel logs for errors or try kextutil(8)` Μπορείτε να το διορθώσετε κάνοντας:
```bash
sudo cp -r osxpmem.app/MacPmem.kext "/tmp/"
sudo kextutil "/tmp/MacPmem.kext"
#Allow the kext in "Security & Privacy --> General"
sudo osxpmem.app/osxpmem --format raw -o /tmp/dump_mem
```
**Άλλα σφάλματα** μπορεί να διορθωθούν επιτρέποντας τη φόρτωση του kext στο "Ασφάλεια & Απορρήτου --> Γενικά", απλά **επιτρέψτε** το.

Μπορείτε επίσης να χρησιμοποιήσετε αυτό το **oneliner** για να κατεβάσετε την εφαρμογή, να φορτώσετε το kext και να κάνετε dump τη μνήμη:

{% code overflow="wrap" %}
```bash
sudo su
cd /tmp; wget https://github.com/google/rekall/releases/download/v1.5.1/osxpmem-2.1.post4.zip; unzip osxpmem-2.1.post4.zip; chown -R root:wheel osxpmem.app/MacPmem.kext; kextload osxpmem.app/MacPmem.kext; osxpmem.app/osxpmem --format raw -o /tmp/dump_mem
```
{% endcode %}

### [WhiteIntel](https://whiteintel.io)

<figure><img src="../../../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) είναι ένας μηχανισμός αναζήτησης που τροφοδοτείται από το **dark web** και προσφέρει δωρεάν λειτουργίες για να ελέγξετε αν μια εταιρεία ή οι πελάτες της έχουν **διαρρεύσει** από **κλέφτες κακόβουλων λογισμικών**.

Ο κύριος στόχος του WhiteIntel είναι η καταπολέμηση των αναλήψεων λογαριασμών και των επιθέσεων ransomware που προκύπτουν από κακόβουλα λογισμικά που κλέβουν πληροφορίες.

Μπορείτε να ελέγξετε τον ιστότοπό τους και να δοκιμάσετε τον μηχανισμό τους δωρεάν στη διεύθυνση:

{% embed url="https://whiteintel.io" %}

{% hint style="success" %}
Μάθετε & εξασκηθείτε στο Hacking του AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**Εκπαίδευση HackTricks AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Μάθετε & εξασκηθείτε στο Hacking του GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**Εκπαίδευση HackTricks GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Υποστηρίξτε το HackTricks</summary>

* Ελέγξτε τα [**σχέδια συνδρομής**](https://github.com/sponsors/carlospolop)!
* **Εγγραφείτε** 💬 στην [**ομάδα Discord**](https://discord.gg/hRep4RUj7f) ή στην [**ομάδα telegram**](https://t.me/peass) ή **ακολουθήστε** μας στο **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Μοιραστείτε κόλπα χάκερ υποβάλλοντας PRs στα** [**HackTricks**](https://github.com/carlospolop/hacktricks) και [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) αποθετήρια στο GitHub.

</details>
{% endhint %}
