# Ερευνητική μεθοδολογία Docker

<details>

<summary><strong>Μάθετε το χάκινγκ του AWS από το μηδέν μέχρι τον ήρωα με το</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Άλλοι τρόποι για να υποστηρίξετε το HackTricks:

* Εάν θέλετε να δείτε την **εταιρεία σας να διαφημίζεται στο HackTricks** ή να **κατεβάσετε το HackTricks σε μορφή PDF** ελέγξτε τα [**ΣΧΕΔΙΑ ΣΥΝΔΡΟΜΗΣ**](https://github.com/sponsors/carlospolop)!
* Αποκτήστε το [**επίσημο PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Ανακαλύψτε [**The PEASS Family**](https://opensea.io/collection/the-peass-family), τη συλλογή μας από αποκλειστικά [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Εγγραφείτε στη** 💬 [**ομάδα Discord**](https://discord.gg/hRep4RUj7f) ή στη [**ομάδα telegram**](https://t.me/peass) ή **ακολουθήστε** μας στο **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Μοιραστείτε τα χάκινγκ κόλπα σας υποβάλλοντας PRs στα** [**HackTricks**](https://github.com/carlospolop/hacktricks) και [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) αποθετήρια του github.

</details>

## Τροποποίηση εμπορεύματος

Υπάρχουν υποψίες ότι ένα κάποιο docker container έχει παραβιαστεί:
```bash
docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
cc03e43a052a        lamp-wordpress      "./run.sh"          2 minutes ago       Up 2 minutes        80/tcp              wordpress
```
Μπορείτε εύκολα **να βρείτε τις τροποποιήσεις που έχουν γίνει σε αυτό το container σχετικά με την εικόνα** με:
```bash
docker diff wordpress
C /var
C /var/lib
C /var/lib/mysql
A /var/lib/mysql/ib_logfile0
A /var/lib/mysql/ib_logfile1
A /var/lib/mysql/ibdata1
A /var/lib/mysql/mysql
A /var/lib/mysql/mysql/time_zone_leap_second.MYI
A /var/lib/mysql/mysql/general_log.CSV
...
```
Στην προηγούμενη εντολή, το **C** σημαίνει **Αλλαγή** και το **A,** **Προσθήκη**.\
Εάν ανακαλύψετε ότι ένα ενδιαφέρον αρχείο όπως το `/etc/shadow` έχει τροποποιηθεί, μπορείτε να το κατεβάσετε από τον εκτελέσιμο χώρο για να ελέγξετε για κακόβουλη δραστηριότητα με:
```bash
docker cp wordpress:/etc/shadow.
```
Μπορείτε επίσης **να το συγκρίνετε με το αρχικό** εκτελώντας ένα νέο container και εξάγοντας το αρχείο από αυτό:
```bash
docker run -d lamp-wordpress
docker cp b5d53e8b468e:/etc/shadow original_shadow #Get the file from the newly created container
diff original_shadow shadow
```
Εάν ανακαλύψετε ότι **προστέθηκε ένα ύποπτο αρχείο**, μπορείτε να αποκτήσετε πρόσβαση στον εκτελέσιμο και να το ελέγξετε:
```bash
docker exec -it wordpress bash
```
## Τροποποιήσεις εικόνας

Όταν σας δίνεται μια εξαγόμενη εικόνα docker (πιθανώς σε μορφή `.tar`), μπορείτε να χρησιμοποιήσετε το [**container-diff**](https://github.com/GoogleContainerTools/container-diff/releases) για να **εξάγετε ένα σύνοψη των τροποποιήσεων**:
```bash
docker save <image> > image.tar #Export the image to a .tar file
container-diff analyze -t sizelayer image.tar
container-diff analyze -t history image.tar
container-diff analyze -t metadata image.tar
```
Στη συνέχεια, μπορείτε να **αποσυμπιέσετε** την εικόνα και να **έχετε πρόσβαση στα blobs** για να αναζητήσετε ύποπτα αρχεία που μπορεί να έχετε βρει στο ιστορικό αλλαγών:
```bash
tar -xf image.tar
```
### Βασική Ανάλυση

Μπορείτε να αποκτήσετε **βασικές πληροφορίες** από την εικόνα που εκτελείται:
```bash
docker inspect <image>
```
Μπορείτε επίσης να λάβετε ένα σύνοψη **ιστορικού αλλαγών** με:
```bash
docker history --no-trunc <image>
```
Μπορείτε επίσης να δημιουργήσετε ένα **dockerfile από ένα εικόνα** με την εντολή:
```bash
alias dfimage="docker run -v /var/run/docker.sock:/var/run/docker.sock --rm alpine/dfimage"
dfimage -sV=1.36 madhuakula/k8s-goat-hidden-in-layers>
```
### Dive

Για να βρείτε προστιθέμενα/τροποποιημένα αρχεία σε εικόνες docker, μπορείτε επίσης να χρησιμοποιήσετε το [**dive**](https://github.com/wagoodman/dive) (κατεβάστε το από [**απελευθερώσεις**](https://github.com/wagoodman/dive/releases/tag/v0.10.0)) εργαλείο:
```bash
#First you need to load the image in your docker repo
sudo docker load < image.tar                                                                                                                                                                                                         1 ⨯
Loaded image: flask:latest

#And then open it with dive:
sudo dive flask:latest
```
Αυτό σας επιτρέπει να **περιηγηθείτε στα διάφορα blobs των εικόνων του Docker** και να ελέγξετε ποια αρχεία τροποποιήθηκαν/προστέθηκαν. Το **κόκκινο** σημαίνει προσθήκη και το **κίτρινο** σημαίνει τροποποίηση. Χρησιμοποιήστε το **tab** για να μετακινηθείτε στην άλλη προβολή και το **space** για να αναδιπλώσετε/ανοίξετε φακέλους.

Με το die δεν θα μπορείτε να έχετε πρόσβαση στο περιεχόμενο των διαφορετικών σταδίων της εικόνας. Για να το κάνετε αυτό, θα πρέπει να **αποσυμπιέσετε κάθε επίπεδο και να έχετε πρόσβαση** σε αυτό.\
Μπορείτε να αποσυμπιέσετε όλα τα επίπεδα μιας εικόνας από τον κατάλογο όπου αποσυμπιέστηκε η εικόνα εκτελώντας:
```bash
tar -xf image.tar
for d in `find * -maxdepth 0 -type d`; do cd $d; tar -xf ./layer.tar; cd ..; done
```
## Διαπιστευτήρια από τη μνήμη

Σημειώστε ότι όταν εκτελείτε ένα δοχείο docker μέσα σε έναν κεντρικό υπολογιστή, **μπορείτε να δείτε τις διεργασίες που εκτελούνται στο δοχείο από τον κεντρικό υπολογιστή** απλά εκτελώντας την εντολή `ps -ef`

Για τον λόγο αυτό (ως root) μπορείτε να **αντιγράψετε τη μνήμη των διεργασιών** από τον κεντρικό υπολογιστή και να αναζητήσετε **διαπιστευτήρια** ακριβώς [**όπως στο παρακάτω παράδειγμα**](../../linux-hardening/privilege-escalation/#process-memory).

<details>

<summary><strong>Μάθετε το χάκινγκ στο AWS από το μηδέν μέχρι τον ήρωα με το</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Άλλοι τρόποι για να υποστηρίξετε το HackTricks:

* Εάν θέλετε να δείτε την **εταιρεία σας να διαφημίζεται στο HackTricks** ή να **κατεβάσετε το HackTricks σε μορφή PDF** ελέγξτε τα [**ΣΧΕΔΙΑ ΣΥΝΔΡΟΜΗΣ**](https://github.com/sponsors/carlospolop)!
* Αποκτήστε το [**επίσημο PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Ανακαλύψτε [**The PEASS Family**](https://opensea.io/collection/the-peass-family), τη συλλογή μας από αποκλειστικά [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Εγγραφείτε στη** 💬 [**ομάδα Discord**](https://discord.gg/hRep4RUj7f) ή στη [**ομάδα telegram**](https://t.me/peass) ή **ακολουθήστε** μας στο **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Μοιραστείτε τα χάκινγκ κόλπα σας υποβάλλοντας PRs στα** [**HackTricks**](https://github.com/carlospolop/hacktricks) και [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) αποθετήρια του github.

</details>
