# PsExec/Winexec/ScExec

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos को PRs सबमिट करके।

</details>

## ये कैसे काम करते हैं

नीचे दिए गए चरणों में प्रक्रिया को दर्शाया गया है, जिससे सेवा बाइनरी को SMB के माध्यम से लक्षित मशीन पर रिमोट निष्पादन प्राप्त किया जा सकता है:

1. **SMB के माध्यम से ADMIN$ सेयर पर सेवा बाइनरी की प्रतिलिपि** की जाती है।
2. **रिमोट मशीन पर एक सेवा का निर्माण** बाइनरी की ओर इशारा करके किया जाता है।
3. सेवा को **रिमोट रूप से शुरू** किया जाता है।
4. बाहर निकलने पर, सेवा **रोकी जाती है, और बाइनरी हटाई जाती है**।

### **PsExec को मैन्युअल रूप से निष्पादित करने की प्रक्रिया**

यह माना जाता है कि एक निष्पादनीय पेलोड (msfvenom द्वारा बनाई गई और एंटीवायरस की पहचान से बचने के लिए Veil का उपयोग करके अव्यावरणित की गई), 'met8888.exe' नामक एक मीटरप्रेटर रिवर्स_http पेलोड को प्रस्तुत करती है, तो निम्नलिखित चरण लिए जाते हैं:

- **बाइनरी की प्रतिलिपि**: एक्जीक्यूटेबल को कमांड प्रॉम्प्ट से ADMIN$ सेयर में कॉपी किया जाता है, हालांकि यह किसी भी फाइल सिस्टम पर कहीं भी रखा जा सकता है ताकि यह छुपा रहे।

- **सेवा बनाना**: विंडोज `sc` कमांड का उपयोग करके, जो विंडोज सेवाओं का जांच, निर्माण और हटाने की अनुमति देता है, एक सेवा नामक "मीटरप्रेटर" नामक सेवा बनाई जाती है जो अपलोड की गई बाइनरी की ओर इशारा करती है।

- **सेवा शुरू करना**: अंतिम चरण में सेवा शुरू करना शामिल है, जिससे बाइनरी एक वास्तविक सेवा बाइनरी न होने और अपेक्षित प्रतिक्रिया को वापस न करने के कारण "समय समाप्त" त्रुटि हो सकती है। यह त्रुटि अमूल्य है क्योंकि मुख्य लक्ष्य बाइनरी का निष्पादन है।

मेटास्प्लॉइट लिस्टनर का अवलोकन करेगा कि सत्र सफलतापूर्वक प्रारंभ हो गया है।

[`sc` कमांड के बारे में अधिक जानें](https://technet.microsoft.com/en-us/library/bb490995.aspx)।

[https://blog.ropnop.com/using-credentials-to-own-windows-boxes-part-2-psexec-and-services/](https://blog.ropnop.com/using-credentials-to-own-windows-boxes-part-2-psexec-and-services/) में अधिक विस्तृत चरण खोजें।

**आप Windows Sysinternals बाइनरी PsExec.exe का भी उपयोग कर सकते हैं:**

![](<../../.gitbook/assets/image (165).png>)

आप [**SharpLateral**](https://github.com/mertdas/SharpLateral) का भी उपयोग कर सकते हैं:

{% code overflow="wrap" %}
```
SharpLateral.exe redexec HOSTNAME C:\\Users\\Administrator\\Desktop\\malware.exe.exe malware.exe ServiceName
```
{% endcode %}

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

दूसरे तरीके HackTricks का समर्थन करने के लिए:

* अगर आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें** द्वारा PRs सबमिट करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>
