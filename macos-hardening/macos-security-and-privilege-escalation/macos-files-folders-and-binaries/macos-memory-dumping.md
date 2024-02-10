# Αποσπάσματα Μνήμης macOS

<details>

<summary><strong>Μάθετε το χάκινγκ του AWS από το μηδέν μέχρι τον ήρωα με το</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Άλλοι τρόποι υποστήριξης του HackTricks:

* Εάν θέλετε να δείτε την **εταιρεία σας να διαφημίζεται στο HackTricks** ή να **κατεβάσετε το HackTricks σε μορφή PDF** ελέγξτε τα [**ΣΧΕΔΙΑ ΣΥΝΔΡΟΜΗΣ**](https://github.com/sponsors/carlospolop)!
* Αποκτήστε το [**επίσημο PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Ανακαλύψτε [**την Οικογένεια PEASS**](https://opensea.io/collection/the-peass-family), τη συλλογή μας από αποκλειστικά [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Εγγραφείτε στη** 💬 [**ομάδα Discord**](https://discord.gg/hRep4RUj7f) ή στη [**ομάδα telegram**](https://t.me/peass) ή **ακολουθήστε** μας στο **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Μοιραστείτε τα χάκινγκ κόλπα σας υποβάλλοντας PRs στα** [**HackTricks**](https://github.com/carlospolop/hacktricks) και [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) αποθετήρια του github.

</details>

## Αρχεία Μνήμης

### Αρχεία Swap

Τα αρχεία swap, όπως το `/private/var/vm/swapfile0`, λειτουργούν ως **μνήμες cache όταν η φυσική μνήμη είναι γεμάτη**. Όταν δεν υπάρχει πλέον χώρος στη φυσική μνήμη, τα δεδομένα μεταφέρονται σε ένα αρχείο swap και στη συνέχεια επαναφέρονται στη φυσική μνήμη όταν απαιτείται. Μπορεί να υπάρχουν πολλά αρχεία swap, με ονόματα όπως swapfile0, swapfile1 και ούτω καθεξής.

### Εικόνα Hibernate

Το αρχείο που βρίσκεται στο `/private/var/vm/sleepimage` είναι κρίσιμο κατά την **κατάσταση αδράνειας**. **Τα δεδομένα από τη μνήμη αποθηκεύονται σε αυτό το αρχείο όταν το OS X μπαίνει σε κατάσταση αδράνειας**. Κατά την αφύπνιση του υπολογιστή, το σύστημα ανακτά τα δεδομένα μνήμης από αυτό το αρχείο, επιτρέποντας στον χρήστη να συνεχίσει από εκεί που σταμάτησε.

Αξίζει να σημειωθεί ότι σε σύγχρονα συστήματα MacOS, αυτό το αρχείο είναι συνήθως κρυπτογραφημένο για λόγους ασφαλείας, καθιστώντας την ανάκτηση δύσκολη.

* Για να ελέγξετε εάν η κρυπτογράφηση είναι ενεργοποιημένη για το sleepimage, μπορείτε να εκτελέσετε την εντολή `sysctl vm.swapusage`. Αυτό θα εμφανίσει εάν το αρχείο είναι κρυπτογραφημένο.

### Αρχεία Καταγραφής Πίεσης Μνήμης

Ένα άλλο σημαντικό αρχείο που σχετίζεται με τη μνήμη στα συστήματα MacOS είναι το **αρχείο καταγραφής πίεσης μνήμης**. Αυτά τα αρχεία καταγραφής βρίσκονται στο `/var/log` και περιέχουν λεπτομερείς πληροφορίες σχετικά με τη χρήση της μνήμης του συστήματος και τα γεγονότα πίεσης. Μπορούν να είναι ιδιαίτερα χρήσιμα για τη διάγνωση θεμάτων που σχετίζονται με τη μνήμη ή για την κατανόηση του τρόπου διαχείρισης της μνήμης από το σύστημα με την πάροδο του χρόνου.

## Αποστολή μνήμης με το osxpmem

Για να αποστείλετε τη μνήμη σε μια μηχανή MacOS, μπορείτε να χρησιμοποιήσετε το [**osxpmem**](https://github.com/google/rekall/releases/download/v1.5.1/osxpmem-2.1.post4.zip).

**Σημείωση**: Οι παρακάτω οδηγίες θα λειτουργήσουν μόνο για Mac με αρχιτεκτονική Intel. Αυτό το εργαλείο είναι τώρα αρχειοθετημένο και η τελευταία έκδοση ήταν το 2017. Το δυαδικό που λήφθηκε χρησιμοποιώντας τις παρακάτω οδηγίες απευθύνεται σε επεξεργαστές Intel καθώς το Apple Silicon δεν υπήρχε το 2017. Μπορεί να είναι δυνατή η μεταγλώττιση του δυαδικού για την αρχιτεκτονική arm64, αλλά θα πρέπει να το δοκιμάσετε μόνοι σας.
```bash
#Dump raw format
sudo osxpmem.app/osxpmem --format raw -o /tmp/dump_mem

#Dump aff4 format
sudo osxpmem.app/osxpmem -o /tmp/dump_mem.aff4
```
Εάν αντιμετωπίσετε αυτό το σφάλμα: `osxpmem.app/MacPmem.kext απέτυχε να φορτωθεί - (libkern/kext) αποτυχία πιστοποίησης (κυριότητα/δικαιώματα αρχείου); ελέγξτε τα αρχεία καταγραφής του συστήματος/πυρήνα για σφάλματα ή δοκιμάστε το kextutil(8)` Μπορείτε να το διορθώσετε κάνοντας:
```bash
sudo cp -r osxpmem.app/MacPmem.kext "/tmp/"
sudo kextutil "/tmp/MacPmem.kext"
#Allow the kext in "Security & Privacy --> General"
sudo osxpmem.app/osxpmem --format raw -o /tmp/dump_mem
```
**Άλλα σφάλματα** μπορεί να διορθωθούν **επιτρέποντας τη φόρτωση του kext** στις "Ασφάλεια & Προσωπικά Δεδομένα --> Γενικά", απλά **επιτρέψτε** το.

Μπορείτε επίσης να χρησιμοποιήσετε αυτό το **oneliner** για να κατεβάσετε την εφαρμογή, να φορτώσετε το kext και να κατεβάσετε τη μνήμη:

{% code overflow="wrap" %}
```bash
sudo su
cd /tmp; wget https://github.com/google/rekall/releases/download/v1.5.1/osxpmem-2.1.post4.zip; unzip osxpmem-2.1.post4.zip; chown -R root:wheel osxpmem.app/MacPmem.kext; kextload osxpmem.app/MacPmem.kext; osxpmem.app/osxpmem --format raw -o /tmp/dump_mem
```
{% endcode %}

<details>

<summary><strong>Μάθετε το χάκινγκ στο AWS από το μηδέν μέχρι τον ήρωα με το</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Άλλοι τρόποι για να υποστηρίξετε το HackTricks:

* Εάν θέλετε να δείτε την **εταιρεία σας να διαφημίζεται στο HackTricks** ή να **κατεβάσετε το HackTricks σε μορφή PDF** ελέγξτε τα [**ΣΧΕΔΙΑ ΣΥΝΔΡΟΜΗΣ**](https://github.com/sponsors/carlospolop)!
* Αποκτήστε το [**επίσημο PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Ανακαλύψτε [**την Οικογένεια PEASS**](https://opensea.io/collection/the-peass-family), τη συλλογή μας από αποκλειστικά [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Εγγραφείτε στη** 💬 [**ομάδα Discord**](https://discord.gg/hRep4RUj7f) ή στη [**ομάδα telegram**](https://t.me/peass) ή **ακολουθήστε** μας στο **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Μοιραστείτε τα χάκινγκ κόλπα σας υποβάλλοντας PRs στα** [**HackTricks**](https://github.com/carlospolop/hacktricks) και [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) αποθετήρια του github.

</details>
