# macOS AppleFS

<details>

<summary><strong>Μάθετε το χάκινγκ του AWS από το μηδέν μέχρι τον ήρωα με το</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Άλλοι τρόποι για να υποστηρίξετε το HackTricks:

* Αν θέλετε να δείτε την **εταιρεία σας να διαφημίζεται στο HackTricks** ή να **κατεβάσετε το HackTricks σε μορφή PDF** ελέγξτε τα [**ΣΧΕΔΙΑ ΣΥΝΔΡΟΜΗΣ**](https://github.com/sponsors/carlospolop)!
* Αποκτήστε το [**επίσημο PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Ανακαλύψτε [**The PEASS Family**](https://opensea.io/collection/the-peass-family), τη συλλογή μας από αποκλειστικά [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Εγγραφείτε στη** 💬 [**ομάδα Discord**](https://discord.gg/hRep4RUj7f) ή στη [**ομάδα telegram**](https://t.me/peass) ή **ακολουθήστε** μας στο **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Μοιραστείτε τα χάκινγκ κόλπα σας υποβάλλοντας PRs στα** [**HackTricks**](https://github.com/carlospolop/hacktricks) και [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) αποθετήρια του github.

</details>

## Apple Propietary File System (APFS)

Το **Apple File System (APFS)** είναι ένα σύγχρονο σύστημα αρχείων που σχεδιάστηκε για να αντικαταστήσει το Hierarchical File System Plus (HFS+). Η ανάπτυξή του οδηγήθηκε από την ανάγκη για **βελτιωμένη απόδοση, ασφάλεια και αποδοτικότητα**.

Ορισμένα σημαντικά χαρακτηριστικά του APFS περιλαμβάνουν:

1. **Κοινή χρήση χώρου**: Το APFS επιτρέπει σε πολλούς τόμους να **μοιράζονται τον ίδιο ελεύθερο αποθηκευτικό χώρο** σε ένα μόνο φυσικό συσκευή. Αυτό επιτρέπει πιο αποδοτική χρήση του χώρου καθώς οι τόμοι μπορούν να αυξομειώνονται δυναμικά χωρίς την ανάγκη για χειροκίνητη αλλαγή μεγέθους ή ανακατανομής.
1. Αυτό σημαίνει, σε σύγκριση με τις παραδοσιακές διαμερίσεις σε αρχεία δίσκων, **ότι στο APFS διάφορες διαμερίσεις (τόμοι) μοιράζονται όλο τον χώρο του δίσκου**, ενώ μια κανονική διαμέριση είχε συνήθως ένα σταθερό μέγεθος.
2. **Snapshots**: Το APFS υποστηρίζει **τη δημιουργία αντιγράφων ασφαλείας**, τα οποία είναι **μόνο για ανάγνωση**, στιγμιότυπων του συστήματος αρχείων. Τα στιγμιότυπα επιτρέπουν αποδοτικά αντίγραφα ασφαλείας και εύκολη αναστροφή του συστήματος, καθώς καταναλώνουν ελάχιστο επιπλέον αποθηκευτικό χώρο και μπορούν να δημιουργηθούν ή να αναστραφούν γρήγορα.
3. **Κλώνοι**: Το APFS μπορεί να **δημιουργήσει κλώνους αρχείων ή καταλόγων που μοιράζονται την ίδια αποθήκευση** με το αρχικό αρχείο μέχρι ο κλώνος ή το αρχικό αρχείο να τροποποιηθεί. Αυτή η δυνατότητα παρέχει έναν αποδοτικό τρόπο δημιουργίας αντιγράφων αρχείων ή καταλόγων χωρίς να διπλασιάζεται ο χώρος αποθήκευσης.
4. **Κρυπτογράφηση**: Το APFS **υποστηρίζει φυσικά την κρυπτογράφηση ολόκληρου του δίσκου**, καθώς και την κρυπτογράφηση ανά αρχείο και ανά κατάλογο, ενισχύοντας την ασφάλεια των δεδομένων σε διάφορες περιπτώσεις χρήσης.
5. **Προστασία από απροσδόκητα σφάλματα**: Το APFS χρησιμοποιεί ένα **σχήμα μεταδεδομένων αντιγραφής-κατά-εγγραφή που εξασφαλίζει τη συνέπεια του συστήματος αρχείων** ακόμη και σε περιπτώσεις απότομης απώλειας ισχύος ή κατάρρευσης του συστήματος, μειώνοντας τον κίνδυνο διάβρωσης των δεδομένων.

Συνολικά, το APFS προσφέρει ένα πιο σύγχρονο, ευέλικτο και αποδοτικό σύστημα αρχείων για τις συσκευές της Apple, με έμφαση στη βελτιωμένη απόδοση, αξιοπιστία και ασφάλεια.
```bash
diskutil list # Get overview of the APFS volumes
```
## Firmlinks

Ο τόμος `Data` είναι προσαρτημένος στο **`/System/Volumes/Data`** (μπορείτε να το ελέγξετε με την εντολή `diskutil apfs list`).

Η λίστα των firmlinks μπορεί να βρεθεί στο αρχείο **`/usr/share/firmlinks`**.
```bash
cat /usr/share/firmlinks
/AppleInternal	AppleInternal
/Applications	Applications
/Library	Library
[...]
```
Στα **αριστερά**, υπάρχει η διαδρομή καταλόγου στο **Όγκο του Συστήματος**, και στα **δεξιά**, η διαδρομή καταλόγου όπου αντιστοιχεί στο **Όγκο Δεδομένων**. Έτσι, `/library` --> `/system/Volumes/data/library`

<details>

<summary><strong>Μάθετε το χάκινγκ του AWS από το μηδέν μέχρι τον ήρωα με το</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Άλλοι τρόποι για να υποστηρίξετε το HackTricks:

* Εάν θέλετε να δείτε την **εταιρεία σας να διαφημίζεται στο HackTricks** ή να **κατεβάσετε το HackTricks σε μορφή PDF**, ελέγξτε τα [**ΠΑΚΕΤΑ ΣΥΝΔΡΟΜΗΣ**](https://github.com/sponsors/carlospolop)!
* Αποκτήστε το [**επίσημο PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Ανακαλύψτε [**την Οικογένεια PEASS**](https://opensea.io/collection/the-peass-family), τη συλλογή μας από αποκλειστικά [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Εγγραφείτε στη** 💬 [**ομάδα Discord**](https://discord.gg/hRep4RUj7f) ή στην [**ομάδα telegram**](https://t.me/peass) ή **ακολουθήστε** μας στο **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Μοιραστείτε τα χάκινγκ κόλπα σας υποβάλλοντας PRs στα** [**HackTricks**](https://github.com/carlospolop/hacktricks) και [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) αποθετήρια του github.

</details>
