<details>

<summary><strong>Μάθετε το χάκινγκ του AWS από το μηδέν μέχρι τον ήρωα με το</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Άλλοι τρόποι για να υποστηρίξετε το HackTricks:

* Αν θέλετε να δείτε την **εταιρεία σας να διαφημίζεται στο HackTricks** ή να **κατεβάσετε το HackTricks σε μορφή PDF** ελέγξτε τα [**ΣΧΕΔΙΑ ΣΥΝΔΡΟΜΗΣ**](https://github.com/sponsors/carlospolop)!
* Αποκτήστε το [**επίσημο PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Ανακαλύψτε [**την Οικογένεια PEASS**](https://opensea.io/collection/the-peass-family), τη συλλογή μας από αποκλειστικά [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Εγγραφείτε στη** 💬 [**ομάδα Discord**](https://discord.gg/hRep4RUj7f) ή στη [**ομάδα telegram**](https://t.me/peass) ή **ακολουθήστε** μας στο **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Μοιραστείτε τα χάκινγκ κόλπα σας υποβάλλοντας PRs στα** [**HackTricks**](https://github.com/carlospolop/hacktricks) και [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) αποθετήρια του github.

</details>


Η έκθεση των `/proc` και `/sys` χωρίς κατάλληλη απομόνωση του namespace εισάγει σημαντικούς κινδύνους ασφαλείας, συμπεριλαμβανομένης της αύξησης της επιφάνειας επίθεσης και της αποκάλυψης πληροφοριών. Αυτοί οι κατάλογοι περιέχουν ευαίσθητα αρχεία που, εάν διαμορφωθούν εσφαλμένα ή ανακτηθούν από μη εξουσιοδοτημένο χρήστη, μπορούν να οδηγήσουν σε διαφυγή του δοχείου, τροποποίηση του κεντρικού συστήματος ή παροχή πληροφοριών που βοηθούν σε περαιτέρω επιθέσεις. Για παράδειγμα, η εσφαλμένη σύνδεση `-v /proc:/host/proc` μπορεί να παρακάμψει την προστασία AppArmor λόγω της φύσης του μονοπατιού, αφήνοντας το `/host/proc` ανεπτυγμένο.

**Μπορείτε να βρείτε περαιτέρω λεπτομέρειες για κάθε πιθανή ευπάθεια στο [https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts).**

# Ευπάθειες procfs

## `/proc/sys`
Αυτός ο κατάλογος επιτρέπει την πρόσβαση για τροποποίηση μεταβλητών πυρήνα, συνήθως μέσω της `sysctl(2)`, και περιέχει αρκετούς υποκαταλόγους που αφορούν:

### **`/proc/sys/kernel/core_pattern`**
- Περιγράφεται στο [core(5)](https://man7.org/linux/man-pages/man5/core.5.html).
- Επιτρέπει τον καθορισμό ενός προγράμματος για εκτέλεση κατά τη δημιουργία αρχείου πυρήνα με τα πρώτα 128 byte ως παραμέτρους. Αυτό μπορεί να οδηγήσει σε εκτέλεση κώδικα εάν το αρχείο αρχίζει με μια σωλήνωση `|`.
- **Παράδειγμα Δοκιμής και Εκμετάλλευσης**:
```bash
[ -w /proc/sys/kernel/core_pattern ] && echo Ναι # Δοκιμή πρόσβασης εγγραφής
cd /proc/sys/kernel
echo "|$overlay/shell.sh" > core_pattern # Ορισμός προσαρμοσμένου χειριστή
sleep 5 && ./crash & # Ενεργοποίηση χειριστή
```

### **`/proc/sys/kernel/modprobe`**
- Λεπτομερώς περιγράφεται στο [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html).
- Περιέχει τη διαδρομή προς τον φορτωτή πυρήνα, που καλείται για τη φόρτωση πυρήνα.
- **Παράδειγμα Έλεγχου Πρόσβασης**:
```bash
ls -l $(cat /proc/sys/kernel/modprobe) # Έλεγχος πρόσβασης στο modprobe
```

### **`/proc/sys/vm/panic_on_oom`**
- Αναφέρεται στο [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html).
- Ένα παγκόσμιο σημαία που ελέγχει εάν ο πυρήνας πανικοβάλλεται ή εκτελεί τον OOM killer όταν συμβαίνει μια κατάσταση OOM.

### **`/proc/sys/fs`**
- Σύμφωνα με το [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html), περιέχει επιλογές και πληροφορίες σχετικά με το σύστημα αρχείων.
### **`/sys/class/thermal`**
- Ελέγχει τις ρυθμίσεις θερμοκρασίας, προκαλώντας πιθανώς επιθέσεις DoS ή φυσικές ζημιές.

### **`/sys/kernel/vmcoreinfo`**
- Διαρρέει διευθύνσεις πυρήνα, ενδεχομένως απειλώντας το KASLR.

### **`/sys/kernel/security`**
- Περιέχει τη διεπαφή `securityfs`, επιτρέποντας τη διαμόρφωση των Linux Security Modules όπως το AppArmor.
- Η πρόσβαση μπορεί να επιτρέψει σε έναν container να απενεργοποιήσει το σύστημα του MAC.

### **`/sys/firmware/efi/vars` και `/sys/firmware/efi/efivars`**
- Εκθέτουν διεπαφές για την αλληλεπίδραση με τις μεταβλητές EFI στη μνήμη NVRAM.
- Η εσφαλμένη διαμόρφωση ή η εκμετάλλευση μπορεί να οδηγήσει σε ανενεργό laptop ή μη εκκινήσιμες μηχανές.

### **`/sys/kernel/debug`**
- Το `debugfs` προσφέρει μια διεπαφή αποσφαλμάτωσης "χωρίς κανόνες" στον πυρήνα.
- Υπάρχει ιστορία προβλημάτων ασφάλειας λόγω της απεριόριστης φύσης του.

## Αναφορές
* [https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts)
* [Understanding and Hardening Linux Containers](https://research.nccgroup.com/wp-content/uploads/2020/07/ncc\_group\_understanding\_hardening\_linux\_containers-1-1.pdf)
* [Abusing Privileged and Unprivileged Linux Containers](https://www.nccgroup.com/globalassets/our-research/us/whitepapers/2016/june/container\_whitepaper.pdf)


<details>

<summary><strong>Μάθετε το χάκινγκ του AWS από το μηδέν μέχρι τον ήρωα με το</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Άλλοι τρόποι για να υποστηρίξετε το HackTricks:

* Εάν θέλετε να δείτε την **εταιρεία σας διαφημισμένη στο HackTricks** ή να **κατεβάσετε το HackTricks σε μορφή PDF** ελέγξτε τα [**ΣΧΕΔΙΑ ΣΥΝΔΡΟΜΗΣ**](https://github.com/sponsors/carlospolop)!
* Αποκτήστε το [**επίσημο PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Ανακαλύψτε [**The PEASS Family**](https://opensea.io/collection/the-peass-family), τη συλλογή μας από αποκλειστικά [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Εγγραφείτε στην** 💬 [**ομάδα Discord**](https://discord.gg/hRep4RUj7f) ή στην [**ομάδα telegram**](https://t.me/peass) ή **ακολουθήστε** μας στο **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Μοιραστείτε τα χάκινγκ κόλπα σας υποβάλλοντας PRs στα** [**HackTricks**](https://github.com/carlospolop/hacktricks) και [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
