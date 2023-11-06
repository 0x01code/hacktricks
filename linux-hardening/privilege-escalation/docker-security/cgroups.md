# सीग्रुप्स

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **फॉलो** करें मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को।**

</details>

## मूलभूत जानकारी

**लिनक्स कंट्रोल समूह**, जिन्हें सीग्रुप्स के रूप में भी जाना जाता है, एक लिनक्स कर्नल सुविधा है जो आपको **सिस्टम संसाधनों को सीमित**, पुलिस और प्राथमिकता देने की अनुमति देती है। सीग्रुप्स एक तरीका प्रदान करते हैं **समूहों के प्रक्रियाओं के संसाधन उपयोग** (सीपीयू, मेमोरी, डिस्क आईओ, नेटवर्क, आदि) को प्रबंधित और अलग करने का। इसका उपयोग कई उद्देश्यों के लिए किया जा सकता है, जैसे कि किसी विशेष समूह के प्रक्रियाओं के लिए उपलब्ध संसाधनों की सीमा लगाना, अन्यों से निर्दिष्ट प्रकार के वर्कलोड को अलग करना, या सिस्टम संसाधनों के उपयोग को विभिन्न समूहों के बीच प्राथमिकता देना।

सीग्रुप्स के दो संस्करण हैं, 1 और 2, और दोनों वर्तमान में उपयोग में हैं और एक सिस्टम पर समय-समय पर कॉन्फ़िगर किए जा सकते हैं। सीग्रुप्स संस्करण 1 और **संस्करण 2** के बीच सबसे **महत्वपूर्ण अंतर** यह है कि इसने सीग्रुप्स के लिए एक नया पेड़-जैसा संगठन पेश किया है, जहां समूहों को **एक पेड़-जैसी संरचना** में व्यवस्थित किया जा सकता है जिसमें माता-पिता-बाल संबंध होते हैं। इससे विभिन्न समूहों के बीच संसाधनों के आवंटन पर एक और लचीला और विस्तृत नियंत्रण होता है।

नई पेड़-जैसी संगठन के अलावा, सीग्रुप्स संस्करण 2 ने भी **कई अन्य बदलाव और सुधार** पेश किए हैं, जैसे कि नए संसाधन नियंत्रकों का समर्थन, पुराने अनुप्रयोगों के लिए बेहतर समर्थन, और बेहतर प्रदर्शन।

समग्र, सीग्रुप्स **संस्करण 2 अधिक सुविधाएं और बेहतर प्रदर्शन** प्रदान करता है संस्करण 1 की तुलना में, लेकिन जहां पुराने सिस्टमों के संगतता की चिंता होती है, वहां अभी भी संस्करण 1 का उपयोग किया जा सकता है।

आप किसी प्रक्रिया के लिए v1 और v2 सीग्रुप्स की सूची उसके cgroup फ़ाइल में /proc/\<pid> देखकर कर सकते हैं। आप इस कमांड के साथ अपने शैल के सीग्रुप्स पर नज़र डालकर शुरू कर सकते हैं:
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
अगर आपके सिस्टम पर **आउटपुट काफी कम है**, तो चिंता न करें; यह बस इसका मतलब है कि आपके पास **केवल cgroups v2** हैं। यहां हर पंक्ति एक नंबर से शुरू होती है और यह एक अलग-अलग cgroup होती है। इसे पढ़ने के लिए यहां कुछ संकेत हैं:

* **2 से 12 तक के नंबर cgroups v1 के लिए हैं**। इनके साथ नंबर के पश्चात नियंत्रक दिए गए होते हैं।
* **नंबर 1** भी **वर्जन 1** के लिए है, लेकिन इसमें कोई नियंत्रक नहीं होता है। यह cgroup **प्रबंधन के उद्देश्यों** के लिए होती है (इस मामले में, systemd ने इसे कॉन्फ़िगर किया है)।
* अंतिम पंक्ति, **नंबर 0**, cgroups v2 के लिए है। यहां कोई नियंत्रक दिखाई नहीं देता है। cgroups v1 वाले सिस्टम पर यह आउटपुट की एकमात्र पंक्ति होगी।
* **नाम हियरार्कियल होते हैं और फ़ाइल पथ के हिस्से की तरह दिखते हैं**। इस उदाहरण में आप देख सकते हैं कि कुछ cgroups का नाम /user.slice है और कुछ /user.slice/user-1000.slice/session-2.scope है।
* /testcgroup नाम उसको दिखाने के लिए बनाया गया था कि cgroups v1 में, प्रक्रिया के लिए cgroups पूरी तरह से अलग हो सकते हैं।
* session शामिल होने वाले user.slice के नाम लॉगिन सत्र होते हैं, जिन्हें systemd द्वारा निर्धारित किया जाता है। आप जब एक शैल के cgroups को देख रहे होंगे तो आप उन्हें देखेंगे। आपके **सिस्टम सेवाओं** के **cgroups system.slice के तहत होंगे**।

### cgroups देखना

सामान्यतः cgroups को **फ़ाइल सिस्टम के माध्यम से एक्सेस किया जाता है**। यह यूनिक्स सिस्टम कॉल इंटरफ़ेस के विपरीत है जिसका उपयोग करने के लिए कर्नल के साथ इंटरैक्ट करने के लिए किया जाता है।\
एक शैल के cgroup की जांच करने के लिए, आप `/proc/self/cgroup` फ़ाइल में देख सकते हैं और फिर `/sys/fs/cgroup` (या `/sys/fs/cgroup/unified`) निर्देशिका में जाकर उसी नाम के एक **निर्देशिका की तलाश करें** जैसा कि cgroup का नाम है। इस निर्देशिका में जाकर चेक करने से आपको cgroup के विभिन्न **सेटिंग्स और संसाधन उपयोग जानकारी** देखने की अनुमति मिलेगी।

<figure><img src="../../../.gitbook/assets/image (10) (2).png" alt=""><figcaption></figcaption></figure>

यहां मौजूद अनेक फ़ाइलों में, **प्राथमिक cgroup इंटरफ़ेस फ़ाइल cgroup से शुरू होती हैं**। पहले `cgroup.procs` देखें (cat का उपयोग करना ठीक है), जिसमें cgroup में प्रक्रियाएँ सूचीबद्ध होती हैं। एक समान फ़ाइल, `cgroup.threads`, धागों को भी शामिल करती है।

<figure><img src="../../../.gitbook/assets/image (1) (1) (5).png" alt=""><figcaption></figcaption></figure>

शैल के लिए उपयोग किए जाने वाले अधिकांश cgroups में ये दो नियंत्रक होते हैं, जो **मेमोरी की मात्रा** और cgroup में **प्रक्रियाओं की कुल संख्या** को नियंत्रित कर सकते हैं। नियंत्रक के साथ इंटरैक्ट करने के लिए, नियंत्रक प्रिफ़िक्स के मिलते जुलते **फ़ाइलों की तलाश करें**। उदाहरण के लिए, यदि आप cgroup में चल रहे धागों की संख्या देखना चाहते हैं, तो pids.current की जांच करें:

<figure><img src="../../../.gitbook/assets/image (3) (5).png" alt=""><figcaption></figcaption></figure>

**max का मतलब है कि इस cgroup का कोई विशेष सीमा नहीं है**, लेकिन cgroups हायरार्कियल होते हैं, इसलिए एक उपनिर्देशिका चेन में वापस जाने वाला cgroup इसे सीमित कर सकता है।

### cgroups को मानिपुलेट और बनाना

एक प्रक्रिया को एक cgroup में डालने के लिए, **उसके PID को रूट के रूप में उसकी `cgroup.procs` फ़ाइल में लिखें:**
```shell-session
# echo pid > cgroup.procs
```
यहां बताया गया है कि cgroups के लिए बदलाव कैसे काम करते हैं। उदाहरण के लिए, यदि आप **cgroup के मैक्सिमम पीआईडी की सीमा को सीमित करना चाहते हैं** (मान लीजिए, 3,000 पीआईडीएस), तो इसे निम्नलिखित तरीके से करें:
```shell-session
# echo 3000 > pids.max
```
**क्रिएटिंग सीग्रुप्स कठिन हो सकता है**। तकनीकी रूप से, यह एक सबडायरेक्टरी क्रिएट करने के रूप में आसान है जहां-कहीं सीग्रुप ट्री में; जब आप ऐसा करते हैं, कर्नल स्वचालित रूप से इंटरफेस फ़ाइलें बनाता है। यदि किसी सीग्रुप में कोई प्रक्रिया नहीं है, तो आप इंटरफेस फ़ाइलें मौजूद होने के बावजूद rmdir के साथ सीग्रुप को हटा सकते हैं। आपको इन सीग्रुप्स को नियमों के बारे में चिंता कर सकती है, जिनमें शामिल हैं:

* आप **केवल बाहरी स्तर ("पत्ती") सीग्रुप्स में प्रक्रियाएं रख सकते हैं**। उदाहरण के लिए, यदि आपके पास /my-cgroup और /my-cgroup/my-subgroup नामक सीग्रुप्स हैं, तो आप /my-cgroup में प्रक्रियाएं नहीं रख सकते हैं, लेकिन /my-cgroup/my-subgroup ठीक है। (एक छूट है अगर सीग्रुप्स में कोई नियंत्रक नहीं है, लेकिन चलो और गहराई में न जाएं।)
* **किसी सीग्रुप में उसके माता-पिता सीग्रुप में नहीं होने वाले नियंत्रक नहीं हो सकते**।
* आपको बाल सीग्रुप्स के लिए नियंत्रक विशेष रूप से **निर्दिष्ट करना होगा**। आप इसे `cgroup.subtree_control` फ़ाइल के माध्यम से कर सकते हैं; उदाहरण के लिए, यदि आप चाहते हैं कि एक बाल सीग्रुप में cpu और pids नियंत्रक हों, तो इस फ़ाइल में +cpu +pids लिखें।

इन नियमों की एक छूट है जो हियरार्की के नीचे पाए जाने वाले **रूट सीग्रुप** में होती है। आप **इस सीग्रुप में प्रक्रियाएं रख सकते हैं**। इसे करने का एक कारण हो सकता है कि आप एक प्रक्रिया को systemd के नियंत्रण से अलग करना चाहते हैं।

नियंत्रकों को सक्षम न करने के बावजूद, आप इसके cpu.stat फ़ाइल को देखकर एक सीग्रुप का सीपीयू उपयोग देख सकते हैं:

<figure><img src="../../../.gitbook/assets/image (2) (6) (3).png" alt=""><figcaption></figcaption></figure>

क्योंकि यह सीग्रुप के संपूर्ण जीवनकाल के एकत्रित सीपीयू उपयोग है, आप देख सकते हैं कि एक सेवा प्रोसेसर समय का उपयोग कैसे करती है, यदि इसमें कई सबप्रोसेस होते हैं जो अंततः समाप्त हो जाते हैं।

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी में काम करते हैं**? क्या आप चाहते हैं कि आपकी **कंपनी HackTricks में विज्ञापित हो**? या क्या आप **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड** करना चाहते हैं? [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**official PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का** **अनुसरण करें**।**
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को।**

</details>
