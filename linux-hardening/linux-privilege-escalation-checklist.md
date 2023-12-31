# चेकलिस्ट - लिनक्स प्रिविलेज एस्केलेशन

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ हैकट्रिक्स क्लाउड ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 ट्विटर 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ ट्विच 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 यूट्यूब 🎥</strong></a></summary>

* क्या आप **साइबरसिक्योरिटी कंपनी** में काम करते हैं? क्या आप चाहते हैं कि आपकी **कंपनी का विज्ञापन हैकट्रिक्स में दिखाई दे**? या क्या आप **PEASS के नवीनतम संस्करण तक पहुँच चाहते हैं या हैकट्रिक्स को PDF में डाउनलोड करना चाहते हैं**? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* [**आधिकारिक PEASS & हैकट्रिक्स स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* **[**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) में शामिल हों** या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे ट्विटर पर फॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपनी हैकिंग ट्रिक्स साझा करें, [**हैकट्रिक्स रेपो**](https://github.com/carlospolop/hacktricks) और [**हैकट्रिक्स-क्लाउड रेपो**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके।​**

</details>

<figure><img src="../.gitbook/assets/image (7) (2).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1) (3) (1).png" alt=""><figcaption></figcaption></figure>

[**HackenProof Discord**](https://discord.com/invite/N3FrSbmwdy) सर्वर में शामिल हों और अनुभवी हैकर्स और बग बाउंटी हंटर्स के साथ संवाद करें!

**हैकिंग इनसाइट्स**\
हैकिंग के रोमांच और चुनौतियों में गहराई से जाने वाली सामग्री के साथ जुड़ें

**रियल-टाइम हैक न्यूज**\
रियल-टाइम न्यूज और इनसाइट्स के माध्यम से तेजी से बदलती हैकिंग दुनिया के साथ अप-टू-डेट रहें

**नवीनतम घोषणाएँ**\
नवीनतम बग बाउंटीज लॉन्चिंग और महत्वपूर्ण प्लेटफॉर्म अपडेट्स के साथ सूचित रहें

**[**डिस्कॉर्ड**](https://discord.com/invite/N3FrSbmwdy) पर हमसे जुड़ें** और आज ही शीर्ष हैकर्स के साथ सहयोग करना शुरू करें!

### **लिनक्स लोकल प्रिविलेज एस्केलेशन वेक्टर्स की तलाश के लिए सर्वोत्तम उपकरण:** [**LinPEAS**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS)

### [सिस्टम जानकारी](privilege-escalation/#system-information)

* [ ] **OS जानकारी** प्राप्त करें
* [ ] [**PATH**](privilege-escalation/#path) की जाँच करें, कोई **लिखने योग्य फोल्डर**?
* [ ] [**env वेरिएबल्स**](privilege-escalation/#env-info) की जाँच करें, कोई संवेदनशील विवरण?
* [ ] [**कर्नेल एक्सप्लॉइट्स**](privilege-escalation/#kernel-exploits) की खोज **स्क्रिप्ट्स का उपयोग करके** (DirtyCow?)
* [ ] [**sudo संस्करण** की जाँच करें यदि वह संवेदनशील है](privilege-escalation/#sudo-version)
* [ ] [**Dmesg** हस्ताक्षर सत्यापन विफल](privilege-escalation/#dmesg-signature-verification-failed)
* [ ] अधिक सिस्टम enum ([तारीख, सिस्टम सांख्यिकी, सीपीयू जानकारी, प्रिंटर्स](privilege-escalation/#more-system-enumeration))
* [ ] [अधिक रक्षा की गणना करें](privilege-escalation/#enumerate-possible-defenses)

### [ड्राइव्स](privilege-escalation/#drives)

* [ ] **सूचीबद्ध माउंटेड** ड्राइव्स
* [ ] **कोई अनमाउंटेड ड्राइव?**
* [ ] **fstab में कोई क्रेड्स?**

### [**इंस्टॉल किया गया सॉफ्टवेयर**](privilege-escalation/#installed-software)

* [ ] **इंस्टॉल किए गए**[ **उपयोगी सॉफ्टवेयर**](privilege-escalation/#useful-software) की जाँच करें
* [ ] **इंस्टॉल किए गए** [**संवेदनशील सॉफ्टवेयर**](privilege-escalation/#vulnerable-software-installed) की जाँच करें

### [प्रक्रियाएँ](privilege-escalation/#processes)

* [ ] क्या कोई **अज्ञात सॉफ्टवेयर चल रहा है**?
* [ ] क्या कोई सॉफ्टवेयर **अधिक विशेषाधिकारों के साथ चल रहा है** जितना उसे होना चाहिए?
* [ ] चल रही प्रक्रियाओं के **एक्सप्लॉइट्स की खोज करें** (विशेष रूप से चल रहे संस्करण)।
* [ ] क्या आप किसी चल रही प्रक्रिया के **बाइनरी को संशोधित** कर सकते हैं?
* [ ] **प्रक्रियाओं की निगरानी करें** और जाँच करें कि क्या कोई दिलचस्प प्रक्रिया बार-बार चल रही है।
* [ ] क्या आप किसी दिलचस्प **प्रक्रिया की मेमोरी पढ़** सकते हैं (जहाँ पासवर्ड सहेजे जा सकते हैं)?

### [निर्धारित/क्रॉन जॉब्स?](privilege-escalation/#scheduled-jobs)

* [ ] क्या [**PATH** ](privilege-escalation/#cron-path)किसी क्रॉन द्वारा संशोधित किया जा रहा है और आप उसमें **लिख** सकते हैं?
* [ ] किसी क्रॉन जॉब में कोई [**वाइल्डकार्ड** ](privilege-escalation/#cron-using-a-script-with-a-wildcard-wildcard-injection)?
* [ ] कोई [**संशोधनीय स्क्रिप्ट** ](privilege-escalation/#cron-script-overwriting-and-symlink) **निष्पादित** की जा रही है या **संशोधनीय फोल्डर** के अंदर है?
* [ ] क्या आपने पता लगाया है कि कोई **स्क्रिप्ट** [**बहुत बार निष्पादित**](privilege-escalation/#frequent-cron-jobs) की जा सकती है या की जा रही है? (हर 1, 2 या 5 मिनट में)

### [सेवाएँ](privilege-escalation/#services)

* [ ] कोई **लिखने योग्य .service** फ़ाइल?
* [ ] कोई **लिखने योग्य बाइनरी** जो किसी **सेवा** द्वारा निष्पादित की जा रही है?
* [ ] कोई **लिखने योग्य फोल्डर systemd PATH में**?

### [टाइमर्स](privilege-escalation/#timers)

* [ ] कोई **लिखने योग्य टाइमर**?

### [सॉकेट्स](privilege-escalation/#sockets)

* [ ] कोई **लिखने योग्य .socket** फ़ाइल?
* [ ] क्या आप **किसी सॉकेट के साथ संवाद कर सकते हैं**?
* [ ] **HTTP सॉकेट्स** में दिलचस्प जानकारी?

### [D-Bus](privilege-escalation/#d-bus)

* [ ] क्या आप **किसी D-Bus के साथ संवाद कर सकते हैं**?

### [नेटवर्क](privilege-escalation/#network)

* [ ] नेटवर्क की गणना करें ताकि आप जान सकें कि आप कहाँ हैं
* [ ] **खुले पोर्ट्स जिन्हें आप मशीन के अंदर शेल प्राप्त करने से पहले एक्सेस नहीं कर सकते थे**?
* [ ] क्या आप `tcpdump` का उपयोग करके **ट्रैफिक स्निफ** कर सकते हैं?

### [उपयोगकर्ता](privilege-escalation/#users)

* [ ] सामान्य उपयोगकर्ता/समूहों की **गणना**
* [ ] क्या आपके पास एक **बहुत बड़ी UID** है? क्या **मशीन** **संवेदनशील** है?
* [ ] क्या आप [**एक समूह के कारण विशेषाधिकार बढ़ा सकते हैं**](privilege-escalation/interesting-groups-linux-pe/) जिसके आप सदस्य हैं?
* [ ] **क्लिपबोर्ड
