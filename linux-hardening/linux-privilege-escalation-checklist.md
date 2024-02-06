# चेकलिस्ट - लिनक्स प्रिविलेज एस्कलेशन

<details>

<summary><strong>शून्य से हीरो तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* **हमारे साथ जुड़ें** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में शामिल हों या हमें **ट्विटर** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें** द्वारा PRs सबमिट करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

<figure><img src="../.gitbook/assets/image (7) (2).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1) (3) (1).png" alt=""><figcaption></figcaption></figure>

[**HackenProof Discord**](https://discord.com/invite/N3FrSbmwdy) सर्वर में शामिल होकर अनुभवी हैकर्स और बग बाउंटी हंटर्स के साथ संवाद करें!

**हैकिंग इंसाइट्स**\
हैकिंग के रोमांच और चुनौतियों में डूबने वाली सामग्री के साथ जुड़ें

**रियल-टाइम हैक न्यूज़**\
रियल-टाइम न्यूज़ और अंतर्दृष्टि के माध्यम से तेजी से बदलती हैकिंग दुनिया के साथ अद्यतन रहें

**नवीनतम घोषणाएं**\
नवीनतम बग बाउंटी लॉन्च और महत्वपूर्ण प्लेटफॉर्म अपडेट्स के साथ सूचित रहें

**हमारे साथ जुड़ें** [**डिस्कॉर्ड**](https://discord.com/invite/N3FrSbmwdy) और आज ही शीर्ष हैकर्स के साथ सहयोग करना शुरू करें!

### **लिनक्स स्थानीय प्रिविलेज एस्केलेशन वेक्टर्स खोजने के लिए सर्वश्रेष्ठ उपकरण:** [**LinPEAS**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS)

### [सिस्टम सूचना](privilege-escalation/#system-information)

* [ ] **ओएस सूचना** प्राप्त करें
* [ ] [**PATH**](privilege-escalation/#path) की जांच करें, कोई **लिखने योग्य फ़ोल्डर** है?
* [ ] [**एनवी वेरिएबल्स**](privilege-escalation/#env-info) की जांच करें, कोई संवेदनशील विवरण?
* [ ] [**कर्नेल एक्सप्लॉइट्स**](privilege-escalation/#kernel-exploits) की खोज **स्क्रिप्ट्स का उपयोग करके** (DirtyCow?)
* [ ] **जांचें** कि [**सुडो संस्करण** विकल्पनशील है](privilege-escalation/#sudo-version)
* [ ] [**Dmesg** हस्ताक्षर सत्यापन विफल](privilege-escalation/#dmesg-signature-verification-failed)
* [ ] अधिक सिस्टम इनम ([तारीख, सिस्टम स्टैट्स, सीपीयू जानकारी, प्रिंटर](privilege-escalation/#more-system-enumeration))
* [ ] [और रक्षाओं की जांच करें](privilege-escalation/#enumerate-possible-defenses)

### [ड्राइव्स](privilege-escalation/#drives)

* [ ] माउंट किए गए **ड्राइव्स** की सूची
* [ ] कोई **अमाउंट किए गए ड्राइव** है?
* [ ] क्या fstab में कोई क्रेडेंशियल है?

### [**स्थापित सॉफ्टवेयर**](privilege-escalation/#installed-software)

* [ ] [**उपयोगी सॉफ्टवेयर**](privilege-escalation/#useful-software) **स्थापित** की जांच करें
* [ ] [**विकल्पनशील सॉफ्टवेयर**](privilege-escalation/#vulnerable-software-installed) **स्थापित** की जांच करें

### [प्रोसेसेस](privilege-escalation/#processes)

* [ ] क्या कोई **अज्ञात सॉफ्टवेयर चल रहा है**?
* [ ] क्या कोई सॉफ्टवेयर **अधिक विशेषाधिकारों के साथ चल रहा है** जितना कि इसे होना चाहिए?
* [ ] चल रहे प्रक्रियाओं के **एक्सप्लॉइट्स की खोज** करें (विशेष रूप से चल रहे संस्करण के)
* [ ] क्या आप किसी चल रहे प्रक्रिया के **बाइनरी को संशोधित** कर सकते हैं?
* [ ] प्रक्रियाओं को **मॉनिटर** करें और जांचें कि क्या कोई दिलचस्प प्रक्रिया नियमित रूप से चल रही है।
* [ ] क्या आप किसी दिलचस्प **प्रक्रिया मेमोरी** (जहां पासवर्ड सहेजा जा सकता है) को **पढ़ सकते हैं**?

### [निर्धारित/Cron जॉब्स?](privilege-escalation/#scheduled-jobs)

* [ ] क्या [**PATH** ](privilege-escalation/#cron-path)को कोई क्रॉन द्वारा संशोधित किया जा रहा है और आप इसमें **लिख सकते हैं**?
* [ ] किसी क्रॉन जॉब में कोई [**वाइल्डकार्ड** ](privilege-escalation/#cron-using-a-script-with-a-wildcard-wildcard-injection)है?
* [ ] क्या कोई [**संशोधन योग्य स्क्रिप्ट** ](privilege-escalation/#cron-script-overwriting-and-symlink)को **चलाया जा रहा है** या **संशोधन योग्य फ़ोल्डर** में है?
* [ ] क्या आपने पाया है कि कुछ **स्क्रिप्ट** को या बहुत **नियमित रूप से चलाया जा रहा है** है? (हर 1, 2 या 5 मिनट में)

### [सेवाएं](privilege-escalation/#services)

* [ ] कोई **लिखने योग्य .सेवा** फ़ाइल है?
* [ ] कोई **लिखने योग्य बाइनरी** एक **सेवा** द्वारा **चलाया जा रहा है**?
* [ ] क्या **systemd PATH** में कोई **लिखने योग्य फ़ोल्डर** है?

### [टाइमर](privilege-escalation/#timers)

* [ ] कोई **लिखने योग्य टाइमर** है?

### [सॉकेट्स](privilege-escalation/#sockets)

* [ ] कोई **लिखने योग्य .सॉकेट** फ़ाइल है?
* [ ] क्या आप किसी सॉकेट के साथ **संवाद कर सकते हैं**?
* [ ] **दिलचस्प जानकारी** वाले **HTTP सॉकेट्स**?

### [डी-बस](privilege-escalation/#d-bus)

* [ ] क्या आप किसी D-Bus के साथ **संवाद कर सकते हैं**?

### [नेटवर्क](privilege-escalation/#network)

* [ ] पता लगाने के लिए नेटवर्क की जांच करें
* [ ] क्या आपने मशीन के अंदर एक शैल मिलने से पहले **पहुंच नहीं पा रहे थे** खोले गए पोर्ट्स?
* [ ] `tcpdump` का उपयोग करके **ट्रैफिक स्निफ़** कर सकते हैं?

### [उपयोगकर्ता](privilege-escalation/#users)

* [ ] सामान्य उपयोगकर्ता/समूह **जांचना**
* [ ] क्या आपके पास **बहुत बड़ा यूआईडी** है? क्या **मशीन** **विकल्पनशील** है?
* [ ] क्या आप [**एक समूह के कारण विशेषाधिकारों को बढ़ा सकते हैं**](privilege-escalation/interesting-groups-linux-pe/) जिसमें आप शामिल हैं?
* [ ] **क्लिपबोर्ड** डेटा?
* [ ] पासवर्ड नीति?
* [ ] पिछले में पता लगाए गए **हर ज्ञात पासवर्ड का उपयोग करने की कोशिश करें** लॉगिन करने के लिए **हर** संभावित **उपयोगकर्ता** के साथ। पासवर्ड के बिना भी लॉगिन करने की कोशिश करें।

### [लिखने योग्य PATH](privilege-escalation/#writable-path-abuses)

* [ ] यदि आपके पास **PATH में किसी फ़ोल्डर पर लेखन अधिकार** है तो आप प्रिव
