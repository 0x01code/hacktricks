<details>

<summary><strong>AWS हैकिंग सीखें शून्य से नायक तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **अपनी हैकिंग ट्रिक्स साझा करें, HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके.

</details>

# Firmware Integrity

**कस्टम फर्मवेयर और/या संकलित बाइनरीज को अखंडता या हस्ताक्षर सत्यापन दोषों का लाभ उठाने के लिए अपलोड किया जा सकता है**। बैकडोर बाइंड शेल संकलन के लिए निम्नलिखित चरणों का पालन किया जा सकता है:

1. फर्मवेयर को firmware-mod-kit (FMK) का उपयोग करके निकाला जा सकता है।
2. लक्षित फर्मवेयर आर्किटेक्चर और एंडियननेस की पहचान की जानी चाहिए।
3. Buildroot या अन्य उपयुक्त तरीकों का उपयोग करके क्रॉस कंपाइलर बनाया जा सकता है।
4. क्रॉस कंपाइलर का उपयोग करके बैकडोर बनाया जा सकता है।
5. बैकडोर को निकाले गए फर्मवेयर के /usr/bin डायरेक्टरी में कॉपी किया जा सकता है।
6. उपयुक्त QEMU बाइनरी को निकाले गए फर्मवेयर rootfs में कॉपी किया जा सकता है।
7. बैकडोर को chroot और QEMU का उपयोग करके अनुकरण किया जा सकता है।
8. बैकडोर को netcat के माध्यम से पहुँचा जा सकता है।
9. QEMU बाइनरी को निकाले गए फर्मवेयर rootfs से हटाया जाना चाहिए।
10. संशोधित फर्मवेयर को FMK का उपयोग करके पुनः पैक किया जा सकता है।
11. बैकडोरड फर्मवेयर का परीक्षण किया जा सकता है, इसे firmware analysis toolkit (FAT) के साथ अनुकरण करके और लक्षित बैकडोर IP और पोर्ट पर netcat का उपयोग करके कनेक्ट करके।

यदि डायनामिक विश्लेषण, बूटलोडर मैनिपुलेशन, या हार्डवेयर सुरक्षा परीक्षण के माध्यम से पहले ही रूट शेल प्राप्त कर लिया गया है, तो पूर्व-संकलित दुर्भावनापूर्ण बाइनरीज जैसे कि इम्प्लांट्स या रिवर्स शेल्स को निष्पादित किया जा सकता है। Metasploit फ्रेमवर्क और 'msfvenom' जैसे स्वचालित पेलोड/इम्प्लांट टूल्स का उपयोग निम्नलिखित चरणों के साथ किया जा सकता है:

1. लक्षित फर्मवेयर आर्किटेक्चर और एंडियननेस की पहचान की जानी चाहिए।
2. Msfvenom का उपयोग करके लक्षित पेलोड, हमलावर होस्ट IP, सुनने वाले पोर्ट नंबर, फाइलटाइप, आर्किटेक्चर, प्लेटफॉर्म, और आउटपुट फाइल को निर्दिष्ट किया जा सकता है।
3. पेलोड को समझौता किए गए डिवाइस पर स्थानांतरित किया जा सकता है और सुनिश्चित किया जा सकता है कि इसमें निष्पादन अनुमतियाँ हैं।
4. Metasploit को आने वाले अनुरोधों को संभालने के लिए तैयार किया जा सकता है, msfconsole शुरू करके और पेलोड के अनुसार सेटिंग्स को कॉन्फ़िगर करके।
5. मीटरप्रीटर रिवर्स शेल को समझौता किए गए डिवाइस पर निष्पादित किया जा सकता है।
6. मीटरप्रीटर सत्रों को देखा जा सकता है क्योंकि वे खुलते हैं।
7. पोस्ट-एक्सप्लॉइटेशन गतिविधियाँ की जा सकती हैं।

यदि संभव हो, तो स्टार्टअप स्क्रिप्ट्स के भीतर की कमजोरियों का लाभ उठाकर रिबूट्स के पार डिवाइस तक स्थायी पहुँच प्राप्त की जा सकती है। ये कमजोरियाँ तब उत्पन्न होती हैं जब स्टार्टअप स्क्रिप्ट्स [प्रतीकात्मक लिंक](https://www.chromium.org/chromium-os/chromiumos-design-docs/hardening-against-malicious-stateful-data) करती हैं, या उन पर निर्भर करती हैं जो कोड अविश्वसनीय माउंटेड स्थानों में स्थित होते हैं जैसे कि SD कार्ड और फ्लैश वॉल्यूम जो रूट फाइल सिस्टम्स के बाहर डेटा स्टोर करने के लिए उपयोग किए जाते हैं।

# संदर्भ
* अधिक जानकारी के लिए [https://scriptingxss.gitbook.io/firmware-security-testing-methodology/](https://scriptingxss.gitbook.io/firmware-security-testing-methodology/) देखें

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से नायक तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **अपनी हैकिंग ट्रिक्स साझा करें, HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके.

</details>
