# चेकलिस्ट - लिनक्स प्रिविलेज एस्केलेशन

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप एक **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एकल [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह।
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल** हों या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का** अनुसरण करें।**
* **अपने हैकिंग ट्रिक्स को** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में पीआर जमा करके अपना योगदान दें।**

</details>

<figure><img src="../.gitbook/assets/image (7) (2).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (1) (3) (1).png" alt=""><figcaption></figcaption></figure>

**HackenProof सभी क्रिप्टो बग बाउंटी का घर है।**

**देरी के बिना पुरस्कार प्राप्त करें**\
HackenProof बाउंटी केवल तब शुरू होती है जब उनके ग्राहक इनाम बजट जमा करते हैं। आपको इनाम उस बग को सत्यापित करने के बाद मिलेगा।

**वेब3 पेंटेस्टिंग में अनुभव प्राप्त करें**\
ब्लॉकचेन प्रोटोकॉल और स्मार्ट कॉन्ट्रैक्ट्स नई इंटरनेट हैं! उनके उभरते दिनों में वेब3 सुरक्षा को मास्टर करें।

**वेब3 हैकर लीजेंड बनें**\
प्रत्येक सत्यापित बग के साथ प्रतिष्ठा अंक प्राप्त करें और साप्ताहिक लीडरबोर्ड के शीर्ष पर विजयी बनें।

[**HackenProof पर साइन अप करें**](https://hackenproof.com/register) और अपने हैक्स से कमाई करें!

{% embed url="https://hackenproof.com/register" %}

### **लिनक्स स्थानीय प्रिविलेज एस्केलेशन वेक्टर्स के लिए सबसे अच्छा टूल:** [**LinPEAS**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS)

### [सिस्टम सूचना](privilege-escalation/#system-information)

* [ ] **ओएस सूचना प्राप्त करें**
* [ ] [**PATH**](privilege-escalation/#path) की जांच करें, कोई **लिखने योग्य फ़ोल्डर** है?
* [ ] [**एनवी चर**](privilege-escalation/#env-info) की जांच करें, कोई संवेदनशील विवरण है?
* [ ] स्क्रिप्ट का उपयोग करके [**कर्नल एक्सप्लॉइट्स**](privilege-escalation/#kernel-exploits) खोजें (DirtyCow?)
* [ ] [**सुडो संस्करण** की जांच करें](privilege-escalation/#sudo-version)
* [ ] [**Dmesg** हस्ताक्षर सत्यापन विफल](privilege-escalation/#dmesg-signature-verification-failed)
* [ ] अधिक सिस्टम इनुमरेशन ([तारीख, सिस्टम आँकड़े, सीपीयू जानकारी, प्रिंटर](privilege-escalation/#more-system-enumeration))
* [ ] [अधिक संरक्षणों की जांच](privilege-escalation/#enumerate-possible-defenses)

### [ड्राइव](privilege-escalation/#drives)

* [ ] माउंट किए गए ड्राइवों की सूची बनाएं
* [ ] कोई **अनमाउंटेड ड्राइव** है?
* [ ] fstab में कोई क्रेडेंशियल हैं?

### [**स्थापित सॉफ़्टवेयर**](privilege-escalation/#installed-software)

* [ ] [**उपयोगी सॉफ़्टवेयर**](privilege-escalation/#useful-software) की जांच करें
* [ ] [**चपेट में आने वाला सॉफ़्टवेयर**](privilege-escalation/#vulnerable-software-installed) की जांच करें

### [प्रक्रियाएँ](privilege-escalation/#processes)

* [ ] क्या कोई **अज्ञात सॉफ़्टवेयर चल रहा** है?
* [ ] क्या कोई सॉफ़्टवेयर **अपेक्षित से अधिक अधिकारों के साथ चल रहा** है?
* [ ] चल रही प्रक्रियाओं के **एक्सप्लॉइट्स** की खोज करें (विशेष रूप से चल रहे संस्करण के लिए)।
* [ ] क्या
### [टाइमर](privilege-escalation/#timers)

* [ ] कोई **लिखने योग्य टाइमर** है?

### [सॉकेट](privilege-escalation/#sockets)

* [ ] कोई **लिखने योग्य .socket** फ़ाइल है?
* [ ] क्या आप **किसी सॉकेट के साथ संवाद कर सकते** हैं?
* [ ] **दिलचस्प जानकारी वाले** **HTTP सॉकेट** हैं?

### [डी-बस](privilege-escalation/#d-bus)

* [ ] क्या आप **किसी डी-बस के साथ संवाद कर सकते** हैं?

### [नेटवर्क](privilege-escalation/#network)

* [ ] जानने के लिए नेटवर्क को जांचें कि आप कहां हैं
* [ ] क्या आपके पास मशीन के अंदर एक शेल प्राप्त करने से पहले **पहले तक पहुंच नहीं हो सकी** खुले पोर्ट हैं?
* [ ] क्या आप `tcpdump` का उपयोग करके ट्रैफिक **स्निफ़ कर सकते** हैं?

### [उपयोगकर्ता](privilege-escalation/#users)

* [ ] सामान्य उपयोगकर्ता / समूहों की **गणना**
* [ ] क्या आपके पास एक **बहुत बड़ा UID** है? क्या **मशीन** **आपत्तियों** के लिए **विकल्पशील** है?
* [ ] क्या आप [**एक समूह के कारण विशेषाधिकारों को बढ़ा सकते**](privilege-escalation/interesting-groups-linux-pe/) हैं जिसमें आप शामिल हैं?
* [ ] **क्लिपबोर्ड** डेटा?
* [ ] पासवर्ड नीति?
* [ ] पहले से पता चले पासवर्ड का प्रयोग करने की कोशिश करें **हर** संभावित **उपयोगकर्ता** के साथ लॉगिन करने के लिए. बिना पासवर्ड के भी लॉगिन करने की कोशिश करें.

### [लिखने योग्य पाथ](privilege-escalation/#writable-path-abuses)

* [ ] यदि आपके पास **पाथ में कुछ फ़ोल्डर पर लिखने की अनुमति** है तो आपको विशेषाधिकारों को बढ़ा सकते हैं

### [SUDO और SUID कमांड](privilege-escalation/#sudo-and-suid)

* [ ] क्या आप **sudo के साथ किसी भी कमांड को निष्पादित** कर सकते हैं? क्या आप इसका उपयोग करके कुछ भी रूट के रूप में पढ़ सकते, लिख सकते या निष्पादित कर सकते हैं? ([**GTFOBins**](https://gtfobins.github.io))
* [ ] क्या कोई **उपयोग करने योग्य SUID बाइनरी** है? ([**GTFOBins**](https://gtfobins.github.io))
* [ ] क्या [**सूडो कमांड्स पाथ द्वारा सीमित** हैं? क्या आप प्रतिबंधों को **दौर कर सकते** हैं](privilege-escalation/#sudo-execution-bypassing-paths)?
* [ ] [**सूडो/SUID बाइनरी बिना कमांड पाथ की निर्देशित**](privilege-escalation/#sudo-command-suid-binary-without-command-path)?
* [ ] [**कमांड पाथ के साथ SUID बाइनरी निर्देशित**](privilege-escalation/#suid-binary-with-command-path)? दौर करें
* [ ] [**LD\_PRELOAD वल्न**](privilege-escalation/#ld\_preload)
* [ ] लिखने योग्य फ़ोल्डर से [**SUID बाइनरी में .so लाइब्रेरी की कमी**](privilege-escalation/#suid-binary-so-injection)?
* [ ] [**SUDO टोकन उपलब्ध**](privilege-escalation/#reusing-sudo-tokens)? [**क्या आप एक SUDO टोकन बना सकते हैं**](privilege-escalation/#var-run-sudo-ts-less-than-username-greater-than)?
* [ ] क्या आप [**sudoers फ़ाइल पढ़ सकते या संशोधित कर सकते हैं**](privilege-escalation/#etc-sudoers-etc-sudoers-d)?
* [ ] क्या आप [**/etc/ld.so.conf.d/ को संशोधित कर सकते हैं**](privilege-escalation/#etc-ld-so-conf-d)?
* [**OpenBSD DOAS**](privilege-escalation/#doas) कमांड

### [क्षमताएँ](privilege-escalation/#capabilities)

* [ ] क्या किसी बाइनरी में कोई **अप्रत्याशित क्षमता** है?

### [ACLs](privilege-escalation/#acls)

* [ ] क्या किसी फ़ाइल में कोई **अप्रत्याशित ACL** है?

### [खुली शैल सत्र](privilege-escalation/#open-shell-sessions)

* [ ] **स्क्रीन**
* [ ] **tmux**

### [SSH](privilege-escalation/#ssh)

* [ ] **डेबियन** [**OpenSSL Predictable PRNG - CVE-2008-0166**](privilege-escalation/#debian-openssl-predictable-prng-cve-2008-0166)
* [ ] [**SSH दिलचस्प कॉन्फ़िगरेशन मान**](privilege-escalation/#ssh-interesting-configuration-values)

### [दिलचस्प फ़ाइलें](privilege-escalation/#interesting-files)

* [ ] **प्रोफ़ाइल फ़ाइलें** - संवेदनशील डेटा पढ़ें? विशेषाधिकार के लिए लिखें?
* [ ] **पासवर्ड/शैडो फ़ा
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**।**
* **अपने हैकिंग ट्रिक्स को साझा करें द्वारा PRs सबमिट करके** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud)।

</details>
