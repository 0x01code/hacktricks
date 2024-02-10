# Κατάχρηση Διεργασιών στο macOS

<details>

<summary><strong>Μάθετε το χάκινγκ του AWS από το μηδέν μέχρι τον ήρωα με το</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Άλλοι τρόποι για να υποστηρίξετε το HackTricks:

* Εάν θέλετε να δείτε την **εταιρεία σας να διαφημίζεται στο HackTricks** ή να **κατεβάσετε το HackTricks σε μορφή PDF**, ελέγξτε τα [**ΣΧΕΔΙΑ ΣΥΝΔΡΟΜΗΣ**](https://github.com/sponsors/carlospolop)!
* Αποκτήστε το [**επίσημο PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Ανακαλύψτε [**την Οικογένεια PEASS**](https://opensea.io/collection/the-peass-family), τη συλλογή μας από αποκλειστικά [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Εγγραφείτε στη** 💬 [**ομάδα Discord**](https://discord.gg/hRep4RUj7f) ή στη [**ομάδα telegram**](https://t.me/peass) ή **ακολουθήστε** μας στο **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Μοιραστείτε τα χάκινγκ κόλπα σας υποβάλλοντας PRs στα** [**HackTricks**](https://github.com/carlospolop/hacktricks) και [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) αποθετήρια του github.

</details>

## Κατάχρηση Διεργασιών στο macOS

Το macOS, όπως κάθε άλλο λειτουργικό σύστημα, παρέχει μια ποικιλία μεθόδων και μηχανισμών για την **αλληλεπίδραση, επικοινωνία και κοινή χρήση δεδομένων** μεταξύ των διεργασιών. Αν και αυτές οι τεχνικές είναι απαραίτητες για την αποτελεσματική λειτουργία του συστήματος, μπορούν επίσης να καταχραστούν από κακόβουλους χρήστες για την **εκτέλεση κακόβουλων δραστηριοτήτων**.

### Εισαγωγή Βιβλιοθήκης

Η Εισαγωγή Βιβλιοθήκης είναι μια τεχνική όπου ένας επιτιθέμενος **αναγκάζει μια διεργασία να φορτώσει μια κακόβουλη βιβλιοθήκη**. Μόλις γίνει η εισαγωγή, η βιβλιοθήκη εκτελείται στο πλαίσιο της στόχος διεργασίας, παρέχοντας στον επιτιθέμενο τα ίδια δικαιώματα και πρόσβαση με τη διεργασία.

{% content-ref url="macos-library-injection/" %}
[macos-library-injection](macos-library-injection/)
{% endcontent-ref %}

### Αγκίστρωμα Συνάρτησης

Το Αγκίστρωμα Συνάρτησης περιλαμβάνει την **παρεμβολή κλήσεων συναρτήσεων** ή μηνυμάτων στον κώδικα λογισμικού. Με το αγκίστρωμα συναρτήσεων, ένας επιτιθέμενος μπορεί να **τροποποιήσει τη συμπεριφορά** μιας διεργασίας, να παρατηρήσει ευαίσθητα δεδομένα ή ακόμα και να αποκτήσει έλεγχο επί της ροής εκτέλεσης.

{% content-ref url="../mac-os-architecture/macos-function-hooking.md" %}
[macos-function-hooking.md](../mac-os-architecture/macos-function-hooking.md)
{% endcontent-ref %}

### Επικοινωνία Μεταξύ Διεργασιών

Η Επικοινωνία Μεταξύ Διεργασιών (IPC) αναφέρεται σε διάφορες μεθόδους με τις οποίες ξεχωριστές διεργασίες **μοιράζονται και ανταλλάσσουν δεδομένα**. Αν και η IPC είναι θεμελιώδης για πολλές νόμιμες εφαρμογές, μπορεί επίσης να καταχραστεί για να παραβιάσει την απομόνωση διεργασιών, να διαρρεύσει ευαίσθητες πληροφορίες ή να εκτελέσει μη εξουσιοδοτημένες ενέργειες.

{% content-ref url="../mac-os-architecture/macos-ipc-inter-process-communication/" %}
[macos-ipc-inter-process-communication](../mac-os-architecture/macos-ipc-inter-process-communication/)
{% endcontent-ref %}

### Εισαγωγή Εφαρμογών Electron

Οι εφαρμογές Electron που εκτελούνται με συγκεκριμένες μεταβλητές περιβάλλοντος μπορεί να είναι ευάλωτες στην εισαγωγή διεργασιών:

{% content-ref url="macos-electron-applications-injection.md" %}
[macos-electron-applications-injection.md](macos-electron-applications-injection.md)
{% endcontent-ref %}

### Ακαθάριστο NIB

Τα αρχεία NIB **ορίζουν τα στοιχεία της διεπαφής χρήστη (UI)** και τις αλληλεπιδράσεις τους μέσα σε μια εφαρμογή. Ωστόσο, μπορούν να **εκτελέσουν αυθαίρετες εντολές** και ο Gatekeeper δεν εμποδίζει την εκτέλεση μιας ήδη εκτελούμενης εφαρμογής αν ένα αρχείο NIB έχει τροποποιηθεί. Επομένως, μπορούν να χρησιμοποιηθούν για να κάνουν αυθαίρετα προγράμματα να εκτελέσουν αυθαίρετες εντολές:
```bash
mv /opt/homebrew/bin/python3 /opt/homebrew/bin/python3.old
cat > /opt/homebrew/bin/python3 <<EOF
#!/bin/bash
# Extra hijack code
/opt/homebrew/bin/python3.old "$@"
EOF
chmod +x /opt/homebrew/bin/python3
```
Ακόμη και ο **root** θα εκτελέσει αυτόν τον κώδικα κατά την εκτέλεση της python.
{% endhint %}

## Ανίχνευση

### Shield

[**Shield**](https://theevilbit.github.io/shield/) ([**Github**](https://github.com/theevilbit/Shield)) είναι μια εφαρμογή ανοιχτού κώδικα που μπορεί να **ανιχνεύει και να αποκλείει ενέργειες εισαγωγής διεργασίας**:

* Χρησιμοποιώντας **Περιβαλλοντικές Μεταβλητές**: Θα παρακολουθεί την παρουσία οποιασδήποτε από τις ακόλουθες περιβαλλοντικές μεταβλητές: **`DYLD_INSERT_LIBRARIES`**, **`CFNETWORK_LIBRARY_PATH`**, **`RAWCAMERA_BUNDLE_PATH`** και **`ELECTRON_RUN_AS_NODE`**
* Χρησιμοποιώντας κλήσεις **`task_for_pid`**: Για να βρει όταν μια διεργασία θέλει να πάρει τη **θύρα εργασίας μιας άλλης** που επιτρέπει την εισαγωγή κώδικα στη διεργασία.
* **Παράμετροι εφαρμογών Electron**: Κάποιος μπορεί να χρησιμοποιήσει τις παρακάτω παραμέτρους γραμμής εντολών **`--inspect`**, **`--inspect-brk`** και **`--remote-debugging-port`** για να ξεκινήσει μια εφαρμογή Electron σε λειτουργία αποσφαλμάτωσης και έτσι να εισάγει κώδικα σε αυτήν.
* Χρησιμοποιώντας **συμβολικά συνδέσμους** ή **συνδέσμους σκληρού δίσκου**: Συνήθως το πιο κοινό κατάχρησης είναι να **τοποθετήσουμε ένα σύνδεσμο με τα δικαιώματα του χρήστη μας**, και **να τον κατευθύνουμε προς μια τοποθεσία με υψηλότερα δικαιώματα**. Η ανίχνευση είναι πολύ απλή τόσο για συμβολικούς συνδέσμους όσο και για συνδέσμους σκληρού δίσκου. Εάν η διεργασία που δημιουργεί τον σύνδεσμο έχει **διαφορετικό επίπεδο δικαιωμάτων** από το αρχείο-στόχο, δημιουργούμε μια **ειδοποίηση**. Δυστυχώς, στην περίπτωση των συμβολικών συνδέσμων, δεν είναι δυνατή η απόκλειση, καθώς δεν έχουμε πληροφορίες για τον προορισμό του συνδέσμου πριν τη δημιουργία του. Αυτό είναι ένα περιορισμός του πλαισίου EndpointSecuriy της Apple.

### Κλήσεις που πραγματοποιούνται από άλλες διεργασίες

Στο [**αυτό το blog post**](https://knight.sc/reverse%20engineering/2019/04/15/detecting-task-modifications.html) μπορείτε να βρείτε πώς είναι δυνατόν να χρησιμοποιήσετε τη συνάρτηση **`task_name_for_pid`** για να λάβετε πληροφορίες για άλλες **διεργασίες που εισάγουν κώδικα σε μια διεργασία** και στη συνέχεια να λάβετε πληροφορίες για αυτήν την άλλη διεργασία.

Σημειώστε ότι για να καλέσετε αυτήν τη συνάρτηση πρέπει να είστε **το ίδιο uid** με αυτό που εκτελεί τη διεργασία ή **root** (και επιστρέφει πληροφορίες για τη διεργασία, όχι έναν τρόπο για να εισάγετε κώδικα).

## Αναφορές

* [https://theevilbit.github.io/shield/](https://theevilbit.github.io/shield/)
* [https://medium.com/@metnew/why-electron-apps-cant-store-your-secrets-confidentially-inspect-option-a49950d6d51f](https://medium.com/@metnew/why-electron-apps-cant-store-your-secrets-confidentially-inspect-option-a49950d6d51f)

<details>

<summary><strong>Μάθετε το hacking στο AWS από το μηδέν μέχρι τον ήρωα με το</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Άλλοι τρόποι για να υποστηρίξετε το HackTricks:

* Εάν θέλετε να δείτε την **εταιρεία σας διαφημισμένη στο HackTricks** ή να **κατεβάσετε το HackTricks σε μορφή PDF** ελέγξτε τα [**ΣΧΕΔΙΑ ΣΥΝΔΡΟΜΗΣ**](https://github.com/sponsors/carlospolop)!
* Αποκτήστε το [**επίσημο PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Ανακαλύψτε [**The PEASS Family**](https://opensea.io/collection/the-peass-family), τη συλλογή μας από αποκλειστικά [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Εγγραφείτε στην** 💬 [**ομάδα Discord**](https://discord.gg/hRep4RUj7f) ή στην [**ομάδα telegram**](https://t.me/peass) ή **ακολουθήστε** μας στο **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Μοιραστείτε τα κόλπα σας για το hacking υποβάλλοντας PRs στα** [**HackTricks**](https://github.com/carlospolop/hacktricks) και [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
