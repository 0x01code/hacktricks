# AD प्रमाणपत्र

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से नायक तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग प्राप्त करें**](https://peass.creator-spring.com)
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) **का पालन करें**.
* **HackTricks के लिए अपनी हैकिंग ट्रिक्स साझा करें PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में.

</details>

## मूल जानकारी

### प्रमाणपत्र के भाग

* **Subject** - प्रमाणपत्र का मालिक।
* **Public Key** - Subject को अलग से संग्रहीत प्राइवेट की से जोड़ता है।
* **NotBefore और NotAfter तिथियां** - प्रमाणपत्र की वैधता की अवधि निर्धारित करती हैं।
* **Serial Number** - CA द्वारा असाइन किया गया प्रमाणपत्र के लिए एक पहचानकर्ता।
* **Issuer** - यह दर्शाता है कि प्रमाणपत्र किसने जारी किया है (आमतौर पर एक CA)।
* **SubjectAlternativeName** - Subject के एक या अधिक वैकल्पिक नामों को परिभाषित करता है। (_नीचे देखें_)
* **Basic Constraints** - यह पहचानता है कि प्रमाणपत्र एक CA है या एक अंतिम इकाई, और प्रमाणपत्र का उपयोग करते समय कोई प्रतिबंध हैं या नहीं।
* **Extended Key Usages (EKUs)** - Object identifiers (OIDs) जो वर्णन करते हैं **प्रमाणपत्र का उपयोग कैसे किया जाएगा**। Microsoft शब्दावली में इसे Enhanced Key Usage भी कहा जाता है। आम EKU OIDs में शामिल हैं:
* Code Signing (OID 1.3.6.1.5.5.7.3.3) - प्रमाणपत्र एक्जीक्यूटेबल कोड के हस्ताक्षर के लिए है।
* Encrypting File System (OID 1.3.6.1.4.1.311.10.3.4) - प्रमाणपत्र फाइल सिस्टम को एन्क्रिप्ट करने के लिए है।
* Secure Email (1.3.6.1.5.5.7.3.4) - प्रमाणपत्र ईमेल को एन्क्रिप्ट करने के लिए है।
* Client Authentication (OID 1.3.6.1.5.5.7.3.2) - प्रमाणपत्र दूसरे सर्वर के लिए प्रमाणीकरण के लिए है (उदाहरण के लिए, AD के लिए)।
* Smart Card Logon (OID 1.3.6.1.4.1.311.20.2.2) - प्रमाणपत्र स्मार्ट कार्ड प्रमाणीकरण में उपयोग के लिए है।
* Server Authentication (OID 1.3.6.1.5.5.7.3.1) - प्रमाणपत्र सर्वरों की पहचान के लिए है (उदाहरण के लिए, HTTPS प्रमाणपत्र)।
* **Signature Algorithm** - प्रमाणपत्र को हस्ताक्षर करने के लिए उपयोग किए गए एल्गोरिदम को निर्दिष्ट करता है।
* **Signature** - प्रमाणपत्रों के शरीर का हस्ताक्षर जो जारीकर्ता की (उदाहरण के लिए, एक CA की) प्राइवेट की का उपयोग करके बनाया गया है।

#### Subject Alternative Names

**Subject Alternative Name** (SAN) एक X.509v3 एक्सटेंशन है। यह **अतिरिक्त पहचानों** को **प्रमाणपत्र** से जोड़ने की अनुमति देता है। उदाहरण के लिए, यदि एक वेब सर्वर **कई डोमेनों के लिए सामग्री** होस्ट करता है, तो **प्रत्येक** लागू **डोमेन** को **SAN** में **शामिल किया जा सकता है** ताकि वेब सर्वर को केवल एक HTTPS प्रमाणपत्र की आवश्यकता हो।

डिफ़ॉल्ट रूप से, प्रमाणपत्र-आधारित प्रमाणीकरण के दौरान, AD प्रमाणपत्रों को उपयोगकर्ता खातों से मैप करता है जो SAN में निर्दिष्ट एक UPN के आधार पर होता है। यदि एक हमलावर **एक अनियंत्रित SAN को निर्दिष्ट कर सकता है** जब एक प्रमाणपत्र का अनुरोध करते हैं जिसमें एक **EKU क्लाइंट प्रमाणीकरण को सक्षम करता है**, और CA एक प्रमाणपत्र बनाता है और हमलावर द्वारा प्रदान किए गए SAN का उपयोग करके हस्ताक्षर करता है, तो **हमलावर डोमेन में किसी भी उपयोगकर्ता बन सकता है**।

### CAs

AD CS चार स्थानों में AD वन द्वारा विश्वास किए गए CA प्रमाणपत्रों को परिभाषित करता है `CN=Public Key Services,CN=Services,CN=Configuration,DC=<domain>,DC=<com>` कंटेनर के नीचे, प्रत्येक उनके उद्देश्य से भिन्न होता है:

* **Certification Authorities** कंटेनर **विश्वसनीय रूट CA प्रमाणपत्रों** को परिभाषित करता है। ये CAs PKI ट्री हायरार्की के **शीर्ष पर हैं** और AD CS वातावरणों में विश्वास का आधार हैं। प्रत्येक CA को कंटेनर के अंदर एक AD ऑब्जेक्ट के रूप में प्रतिनिधित्व किया जाता है जहां **objectClass** को **`certificationAuthority`** पर सेट किया गया है और **`cACertificate`** प्रॉपर्टी में **CA के प्रमाणपत्र** के **बाइट्स** होते हैं। Windows इन CA प्रमाणपत्रों को **प्रत्येक Windows मशीन** पर Trusted Root Certification Authorities प्रमाणपत्र स्टोर में प्रसारित करता है। AD के लिए एक प्रमाणपत्र को **विश्वसनीय** मानने के लिए, प्रमाणपत्र की ट्रस्ट **चेन** अंततः **इस कंटेनर में परिभाषित एक रूट CA के साथ समाप्त** होनी चाहिए।
* **Enrolment Services** कंटेनर प्रत्येक **Enterprise CA** को परिभाषित करता है (यानी, AD CS में बनाए गए CAs जिसमें Enterprise CA भूमिका सक्षम है)। प्रत्येक Enterprise CA के पास निम्नलिखित विशेषताओं के साथ एक AD ऑब्जेक्ट होता है:
* एक **objectClass** विशेषता **`pKIEnrollmentService`** के लिए
* एक **`cACertificate`** विशेषता जिसमें **CA के प्रमाणपत्र के बाइट्स** होते हैं
* एक **`dNSHostName`** प्रॉपर्टी जो **CA के DNS होस्ट को सेट करती है**
* एक **certificateTemplates** फील्ड जो **सक्षम प्रमाणपत्र टेम्पलेट्स** को परिभाषित करती है। प्रमाणपत्र टेम्पलेट्स एक "ब्लूप्रिंट" होते हैं जिसका उपयोग CA प्रमाणपत्र बनाते समय करता है, और इसमें EKUs, नामांकन अनुमतियां, प्रमाणपत्र की समाप्ति, जारी करने की आवश्यकताएं, और क्रिप्टोग्राफी सेटिंग्स जैसी चीजें शामिल होती हैं। हम बाद में प्रमाणपत्र टेम्पलेट्स के बारे में और विस्तार से चर्चा करेंगे।

{% hint style="info" %}
AD वातावरणों में, **क्लाइंट्स Enterprise CAs से प्रमाणपत्र का अनुरोध करने के लिए इंटरैक्ट करते हैं** जो एक प्रमाणपत्र टेम्पलेट में परिभाषित सेटिंग्स के आधार पर होता है। Enterprise CA प्रमाणपत्रों को प्रत्येक Windows मशीन पर Intermediate Certification Authorities प्रमाणपत्र स्टोर में प्रसारित किया जाता है।
{% endhint %}

* **NTAuthCertificates** AD ऑब
```bash
# https://github.com/GhostPack/Certify
Certify.exe cas #enumerate trusted root CA certificates, certificates defined by the NTAuthCertificates object, and various information about Enterprise CAs
Certify.exe find #enumerate certificate templates
Certify.exe find /vulnerable #Enumerate vulenrable certificate templater

# https://github.com/ly4k/Certipy
certipy find -u john@corp.local -p Passw0rd -dc-ip 172.16.126.128
certipy find -vulnerable [-hide-admins] -u john@corp.local -p Passw0rd -dc-ip 172.16.126.128 #Search vulnerable templates

certutil.exe -TCAInfo #enumerate Enterprise CAs
certutil -v -dstemplate #enumerate certificate templates
```
## संदर्भ

* [https://www.specterops.io/assets/resources/Certified\_Pre-Owned.pdf](https://www.specterops.io/assets/resources/Certified\_Pre-Owned.pdf)
* [https://comodosslstore.com/blog/what-is-ssl-tls-client-authentication-how-does-it-work.html](https://comodosslstore.com/blog/what-is-ssl-tls-client-authentication-how-does-it-work.html)

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert) के साथ शून्य से नायक तक AWS हैकिंग सीखें</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सदस्यता योजनाएँ**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा संग्रह विशेष [**NFTs**](https://opensea.io/collection/the-peass-family)
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) में शामिल हों या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) **का अनुसरण करें**.
* **HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें.

</details>
