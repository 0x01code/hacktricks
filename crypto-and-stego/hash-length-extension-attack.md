# हैश लेंथ एक्सटेंशन हमला

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** पर **फॉलो** करें 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

#### [WhiteIntel](https://whiteintel.io)

<figure><img src="../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) एक **डार्क-वेब** से प्रेरित खोज इंजन है जो **स्टीलर मैलवेयर** द्वारा कंपनी या उसके ग्राहकों के **कंप्रोमाइज़** होने की **मुफ्त** सुविधाएं प्रदान करता है।

WhiteIntel का मुख्य उद्देश्य है खाता हस्तांतरण और जानकारी चोरी मैलवेयर से होने वाले रैंसमवेयर हमलों का विरोध करना।

आप उनकी वेबसाइट चेक कर सकते हैं और उनके इंजन का मुफ्त परीक्षण कर सकते हैं:

{% embed url="https://whiteintel.io" %}

***

## हमले का सारांश

एक सर्वर की कलर टेक्स्ट डेटा के साथ कुछ **गोपनीय** जोड़कर उस डेटा को हैश कर रहा है। यदि आप जानते हैं:

* **गोपनीय की लंबाई** (यह एक दी गई लंबाई सीमा से भी bruteforced किया जा सकता है)
* **स्पष्ट पाठ डेटा**
* **एल्गोरिथ्म (और यह हमले के लिए वंर्नरबल है)**
* **पैडिंग ज्ञात है**
* आम तौर पर एक डिफ़ॉल्ट प्रयोग होता है, इसलिए अगर अन्य 3 आवश्यकताएं पूरी होती हैं, तो यह भी होता है
* पैडिंग गोपनीय+डेटा की लंबाई के आधार पर भिन्न होता है, इसलिए गोपनीय की लंबाई की आवश्यकता है

तो, एक **हमलावर** के लिए संभव है कि वह **डेटा** को **जोड़ सकता है** और **पिछले डेटा + जोड़ा गया डेटा** के लिए एक मान्य **सिग्नेचर** उत्पन्न कर सकता है।

### कैसे?

मौजूदा एल्गोरिथ्म बुद्धिमानी से हैश उत्पन्न करते हैं पहले से ही एक डेटा ब्लॉक को हैश करके, और फिर, **पिछले** बनाए गए **हैश** (स्थिति) से, वे **अगले डेटा ब्लॉक को जोड़ते हैं** और **हैश करते हैं**।

फिर, यहां यह विचार करें कि गोपनीय "गोपनीय" है और डेटा "डेटा" है, "गोपनीयडेटा" का MD5 6036708eba0d11f6ef52ad44e8b74d5b है।\
यदि एक हमलावर "जोड़ना" स्ट्रिंग जोड़ना चाहता है तो वह:

* 64 "A" का MD5 उत्पन्न करें
* पहले से आरंभित हैश की स्थिति को 6036708eba0d11f6ef52ad44e8b74d5b पर बदलें
* स्ट्रिंग "जोड़ना" जोड़ें
* हैश समाप्त करें और परिणामी हैश "गोपनीय" + "डेटा" + "पैडिंग" + "जोड़ना" के लिए एक **मान्य** होगा**

### **उपकरण**

{% embed url="https://github.com/iagox86/hash_extender" %}

### संदर्भ

आप इस हमले को अच्छी तरह से समझ सकते हैं [https://blog.skullsecurity.org/2012/everything-you-need-to-know-about-hash-length-extension-attacks](https://blog.skullsecurity.org/2012/everything-you-need-to-know-about-hash-length-extension-attacks)

#### [WhiteIntel](https://whiteintel.io)

<figure><img src="../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) एक **डार्क-वेब** से प्रेरित खोज इंजन है जो **स्टीलर मैलवेयर** द्वारा कंपनी या उसके ग्राहकों के **कंप्रोमाइज** होने की **मुफ्त** सुविधाएं प्रदान करता है।

WhiteIntel का मुख्य उद्देश्य है खाता हस्तांतरण और जानकारी चोरी मैलवेयर से होने वाले रैंसमवेयर हमलों का विरोध करना।

आप उनकी वेबसाइट चेक कर सकते हैं और उनके इंजन का मुफ्त परीक्षण कर सकते हैं:

{% embed url="https://whiteintel.io" %}

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** पर **फॉलो** करें 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>
