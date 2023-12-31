# CGroups

<details>

<summary><strong>Learn AWS hacking from zero to hero with</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

अन्य तरीके HackTricks का समर्थन करने के लिए:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग प्राप्त करें**](https://peass.creator-spring.com)
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) को **फॉलो करें**.
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स शेयर करें.

</details>

## मूल जानकारी

**Linux control groups**, जिन्हें cgroups के नाम से भी जाना जाता है, एक Linux kernel feature है जो आपको प्रक्रियाओं के संग्रह के लिए **सीमित करने**, पुलिसिंग, और **सिस्टम संसाधनों** को प्राथमिकता देने की अनुमति देता है। Cgroups प्रक्रियाओं के समूहों के **संसाधन उपयोग को प्रबंधित और अलग करने** का एक तरीका प्रदान करते हैं (CPU, मेमोरी, डिस्क I/O, नेटवर्क, आदि)। यह कई उद्देश्यों के लिए उपयोगी हो सकता है, जैसे कि प्रक्रियाओं के विशेष समूह के लिए उपलब्ध संसाधनों को सीमित करना, कुछ प्रकार के कार्यभारों को अन्य से अलग करना, या विभिन्न प्रक्रियाओं के समूहों के बीच सिस्टम संसाधनों के उपयोग को प्राथमिकता देना।

cgroups के **दो संस्करण** हैं, 1 और 2, और दोनों वर्तमान में उपयोग में हैं और एक सिस्टम पर एक साथ कॉन्फ़िगर किए जा सकते हैं। cgroups संस्करण 1 और **संस्करण 2** के बीच सबसे **महत्वपूर्ण अंतर** यह है कि बाद वाले ने cgroups के लिए एक नया हायरार्किकल संगठन पेश किया, जहां समूहों को माता-पिता-बच्चे के संबंधों के साथ एक **वृक्ष-सदृश संरचना** में व्यवस्थित किया जा सकता है। यह विभिन्न प्रक्रियाओं के समूहों के बीच संसाधनों के आवंटन पर अधिक लचीला और सूक्ष्म नियंत्रण करने की अनुमति देता है।

नए हायरार्किकल संगठन के अलावा, cgroups संस्करण 2 ने **कई अन्य परिवर्तन और सुधार भी पेश किए**, जैसे कि **नए संसाधन नियंत्रकों** के लिए समर्थन, पुराने अनुप्रयोगों के लिए बेहतर समर्थन, और सुधारित प्रदर्शन।

कुल मिलाकर, cgroups **संस्करण 2 में संस्करण 1 की तुलना में अधिक सुविधाएँ और बेहतर प्रदर्शन है**, लेकिन पुराने सिस्टमों के साथ संगतता की चिंता होने पर अंतिम अभी भी कुछ परिदृश्यों में उपयोग किया जा सकता है।

आप /proc/\<pid> में उसकी cgroup फ़ाइल को देखकर किसी भी प्रक्रिया के लिए v1 और v2 cgroups की सूची बना सकते हैं। आप इस कमांड के साथ अपने शेल के cgroups को देखना शुरू कर सकते हैं:
```shell-session
$ cat /proc/self/cgroup
12:rdma:/
11:net_cls,net_prio:/
10:perf_event:/
9:cpuset:/
8:cpu,cpuacct:/user.slice
7:blkio:/user.slice
6:memory:/user.slice 5:pids:/user.slice/user-1000.slice/session-2.scope 4:devices:/user.slice
3:freezer:/
2:hugetlb:/testcgroup
1:name=systemd:/user.slice/user-1000.slice/session-2.scope
0::/user.slice/user-1000.slice/session-2.scope
```
यदि आपके सिस्टम पर **आउटपुट काफी छोटा है**, तो चिंतित न हों; इसका मतलब यह है कि आपके पास शायद केवल **cgroups v2 हैं**। यहाँ प्रत्येक आउटपुट लाइन एक अलग cgroup को दर्शाती है और प्रत्येक लाइन एक नंबर से शुरू होती है। इसे पढ़ने के कुछ संकेत यहाँ दिए गए हैं:

* **नंबर 2–12 cgroups v1 के लिए हैं**। इनके **controllers** नंबर के बगल में सूचीबद्ध हैं।
* **नंबर 1** भी **संस्करण 1 के लिए है**, लेकिन इसमें कोई controller नहीं है। यह cgroup केवल **प्रबंधन उद्देश्यों** के लिए है (इस मामले में, systemd ने इसे कॉन्फ़िगर किया है)।
* अंतिम लाइन, **नंबर 0**, **cgroups v2 के लिए है**। यहाँ कोई controllers दिखाई नहीं दे रहे हैं। एक सिस्टम पर जहाँ cgroups v1 नहीं है, यह एकमात्र आउटपुट लाइन होगी।
* **नाम संरचनात्मक हैं और फ़ाइल पथों के हिस्सों की तरह दिखते हैं**। इस उदाहरण में आप देख सकते हैं कि कुछ cgroups का नाम /user.slice है और अन्य /user.slice/user-1000.slice/session-2.scope हैं।
* /testcgroup नाम बनाया गया था ताकि यह दिखाया जा सके कि cgroups v1 में, एक प्रक्रिया के लिए cgroups पूरी तरह से स्वतंत्र हो सकते हैं।
* **user.slice के अंतर्गत नाम** जिनमें session शामिल हैं, वे login sessions हैं, जिन्हें systemd द्वारा असाइन किया गया है। जब आप एक shell के cgroups को देखेंगे तो आप उन्हें देखेंगे। आपकी **सिस्टम सेवाओं** के लिए **cgroups** **system.slice के अंतर्गत** होंगे।

### cgroups देखना

Cgroups आमतौर पर **फ़ाइल सिस्टम के माध्यम से पहुँचे जाते हैं**। यह पारंपरिक Unix सिस्टम कॉल इंटरफ़ेस के विपरीत है जो कर्नेल के साथ इंटरैक्ट करने के लिए होता है।\
एक shell के cgroup सेटअप को एक्सप्लोर करने के लिए, आप `/proc/self/cgroup` फ़ाइल में देख सकते हैं ताकि shell के cgroup का पता लगा सकें, और फिर `/sys/fs/cgroup` (या `/sys/fs/cgroup/unified`) डायरेक्टरी में नेविगेट करें और cgroup के समान नाम वाली **डायरेक्टरी की तलाश करें**। इस डायरेक्टरी में जाकर आसपास देखने से आप cgroup के विभिन्न **सेटिंग्स और संसाधन उपयोग की जानकारी देख सकेंगे**।

<figure><img src="../../../.gitbook/assets/image (10) (2) (2).png" alt=""><figcaption></figcaption></figure>

यहाँ जो कई फ़ाइलें हो सकती हैं, उनमें से **मुख्य cgroup इंटरफ़ेस फ़ाइलें `cgroup` से शुरू होती हैं**। `cgroup.procs` पर नज़र डालें (cat का उपयोग करना ठीक है), जो cgroup में प्रक्रियाओं की सूची देती है। एक समान फ़ाइल, `cgroup.threads`, थ्रेड्स को भी शामिल करती है।

<figure><img src="../../../.gitbook/assets/image (1) (1) (5).png" alt=""><figcaption></figcaption></figure>

ज्यादातर shells के लिए इस्तेमाल किए जाने वाले cgroups में ये दो controllers होते हैं, जो **मेमोरी की मात्रा** और cgroup में **प्रक्रियाओं की कुल संख्या** को नियंत्रित कर सकते हैं। किसी controller के साथ इंटरैक्ट करने के लिए, उन **फ़ाइलों की तलाश करें जो controller प्रीफ़िक्स से मेल खाती हैं**। उदाहरण के लिए, यदि आप cgroup में चल रहे थ्रेड्स की संख्या देखना चाहते हैं, तो pids.current को देखें:

<figure><img src="../../../.gitbook/assets/image (3) (5).png" alt=""><figcaption></figcaption></figure>

**max का मूल्य यह दर्शाता है कि इस cgroup की कोई विशेष सीमा नहीं है**, लेकिन चूंकि cgroups संरचनात्मक होते हैं, इसलिए उपनिर्देशिका श्रृंखला में नीचे कोई cgroup इसे सीमित कर सकता है।

### cgroups को संचालित करना और बनाना

किसी प्रक्रिया को cgroup में डालने के लिए, **उसके PID को रूट के रूप में उसकी `cgroup.procs` फ़ाइल में लिखें:**
```shell-session
# echo pid > cgroup.procs
```
यह है कि cgroups में कितने परिवर्तन काम करते हैं। उदाहरण के लिए, यदि आप **cgroup के अधिकतम PIDs की संख्या सीमित करना चाहते हैं** (कहें, 3,000 PIDs तक), तो इसे निम्नानुसार करें:
```shell-session
# echo 3000 > pids.max
```
**cgroups बनाना थोड़ा कठिन है**। तकनीकी रूप से, यह cgroup ट्री में कहीं एक उपनिर्देशिका बनाने जितना आसान है; जब आप ऐसा करते हैं, तो कर्नेल स्वचालित रूप से इंटरफ़ेस फ़ाइलें बना देता है। अगर किसी cgroup में कोई प्रक्रियाएं नहीं हैं, तो आप इंटरफ़ेस फ़ाइलों के होते हुए भी rmdir के साथ cgroup को हटा सकते हैं। जो आपको उलझा सकते हैं वे cgroups के नियम हैं, जिनमें शामिल हैं:

* आप **केवल बाहरी-स्तर ("पत्ती") cgroups में प्रक्रियाएं डाल सकते हैं**। उदाहरण के लिए, अगर आपके पास /my-cgroup और /my-cgroup/my-subgroup नामक cgroups हैं, तो आप /my-cgroup में प्रक्रियाएं नहीं डाल सकते, लेकिन /my-cgroup/my-subgroup में डालना ठीक है। (एक अपवाद है अगर cgroups में कोई कंट्रोलर्स नहीं हैं, लेकिन आइए और न खोदें।)
* एक cgroup में **उसके माता-पिता cgroup में न होने वाला कंट्रोलर नहीं हो सकता**।
* आपको बच्चे cgroups के लिए स्पष्ट रूप से **कंट्रोलर्स निर्दिष्ट करने होंगे**। आप यह `cgroup.subtree_control` फ़ाइल के माध्यम से करते हैं; उदाहरण के लिए, अगर आप चाहते हैं कि एक बच्चे cgroup में cpu और pids कंट्रोलर्स हों, तो इस फ़ाइल में +cpu +pids लिखें।

इन नियमों का एक अपवाद **रूट cgroup** है जो पदानुक्रम के नीचे पाया जाता है। आप इस cgroup में **प्रक्रियाएं रख सकते हैं**। ऐसा करने का एक कारण यह हो सकता है कि आप एक प्रक्रिया को systemd के नियंत्रण से अलग करना चाहते हैं।

कोई भी कंट्रोलर्स सक्षम न होने पर भी, आप एक cgroup के CPU उपयोग को उसकी cpu.stat फ़ाइल देखकर जान सकते हैं:

<figure><img src="../../../.gitbook/assets/image (2) (6) (3).png" alt=""><figcaption></figcaption></figure>

चूंकि यह cgroup के पूरे जीवनकाल के दौरान संचित CPU उपयोग है, आप देख सकते हैं कि एक सेवा प्रोसेसर समय का उपभोग कैसे करती है भले ही यह कई उपप्रक्रियाएं उत्पन्न करती है जो अंततः समाप्त हो जाती हैं।

<details>

<summary><strong>Learn AWS hacking from zero to hero with</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का अन्य तरीकों से समर्थन करें:

* अगर आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें।
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह।
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) **का पालन करें**।
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>
