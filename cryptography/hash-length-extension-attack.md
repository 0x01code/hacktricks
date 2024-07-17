<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप चाहते हैं कि **आपकी कंपनी HackTricks में विज्ञापित हो** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** पर फॉलो करें 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में

</details>


# हमले का सारांश

एक सर्वर की कलर टेक्स्ट डेटा के साथ कुछ गुप्त से जोड़कर उस डेटा का हैश बनाता है। यदि आप जानते हैं:

* **गुप्त की लंबाई** (यह एक दी गई लंबाई सीमा से भी bruteforced किया जा सकता है)
* **स्पष्ट पाठ डेटा**
* **एल्गोरिथ्म (और यह हमले के लिए वंर्नरबल है)**
* **पैडिंग ज्ञात है**
* आम तौर पर एक डिफ़ॉल्ट प्रयोग होता है, इसलिए अगर अन्य 3 आवश्यकताएं पूरी होती हैं, तो यह भी होता है
* पैडिंग गुप्त+डेटा की लंबाई पर निर्भर होता है, इसलिए गुप्त की लंबाई की आवश्यकता है

तो, एक **हमलावर** के लिए संभव है कि वह **डेटा** को **जोड़ सकता है** और **पिछले डेटा + जोड़े गए डेटा** के लिए एक मान्य **सिग्नेचर** उत्पन्न कर सकता है।

## कैसे?

मूल रूप से वंर्नरबल एल्गोरिथ्म पहले **डेटा ब्लॉक का हैश बनाते हैं**, और फिर, **पहले** बनाए गए **हैश** (स्थिति) से, वे **अगले डेटा ब्लॉक को जोड़ते हैं** और **उसे हैश करते हैं**।

फिर, यहां सोचें कि गुप्त "गुप्त" है और डेटा "डेटा" है, "गुप्तडेटा" का MD5 6036708eba0d11f6ef52ad44e8b74d5b है।\
यदि एक हमलावर "जोड़ना" स्ट्रिंग जोड़ना चाहता है तो वह:

* 64 "A" का MD5 उत्पन्न करें
* पहले इनिशियलाइज़ हैश की स्थिति को 6036708eba0d11f6ef52ad44e8b74d5b पर बदलें
* स्ट्रिंग "जोड़ना" जोड़ें
* हैश समाप्त करें और परिणामी हैश "गुप्त" + "डेटा" + "पैडिंग" + "जोड़ना" के लिए **मान्य होगा**

## **उपकरण**

{% embed url="https://github.com/iagox86/hash_extender" %}

## संदर्भ

आप इस हमले को [https://blog.skullsecurity.org/2012/everything-you-need-to-know-about-hash-length-extension-attacks](https://blog.skullsecurity.org/2012/everything-you-need-to-know-about-hash-length-extension-attacks) में अच्छी तरह से समझ सकते हैं


<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप चाहते हैं कि **आपकी कंपनी HackTricks में विज्ञापित हो** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** पर फॉलो करें 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में

</details>
