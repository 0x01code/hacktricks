<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग ट्रिक्स शेयर करें।

</details>


# हमले का सारांश

कल्पना कीजिए एक सर्वर जो कुछ **डेटा** को **साइन** कर रहा है **सीक्रेट** को कुछ ज्ञात स्पष्ट पाठ डेटा के साथ **जोड़कर** और फिर उस डेटा को हैशिंग कर रहा है। यदि आप जानते हैं:

* **सीक्रेट की लंबाई** (इसे एक निश्चित लंबाई सीमा से भी ब्रूटफोर्स किया जा सकता है)
* **स्पष्ट पाठ डेटा**
* **एल्गोरिथम (और यह इस हमले के लिए संवेदनशील है)**
* **पैडिंग ज्ञात है**
* आमतौर पर एक डिफ़ॉल्ट वाला इस्तेमाल किया जाता है, इसलिए अगर अन्य 3 आवश्यकताएँ पूरी होती हैं, तो यह भी होता है
* पैडिंग सीक्रेट+डेटा की लंबाई के आधार पर बदलती है, इसलिए सीक्रेट की लंबाई की आवश्यकता होती है

तब, यह संभव है कि एक **हमलावर** **डेटा** को **जोड़े** और **पिछले डेटा + जोड़े गए डेटा** के लिए एक वैध **हस्ताक्षर** **उत्पन्न** करे।

## कैसे?

मूल रूप से संवेदनशील एल्गोरिथम डेटा के एक ब्लॉक को **हैशिंग** करके हैशेज उत्पन्न करते हैं, और फिर, **पहले से बनाए गए** हैश (स्टेट) से, वे **अगले ब्लॉक डेटा को जोड़ते हैं** और **इसे हैश करते हैं**।

तब, कल्पना कीजिए कि सीक्रेट "secret" है और डेटा "data" है, "secretdata" का MD5 6036708eba0d11f6ef52ad44e8b74d5b है।\
यदि एक हमलावर "append" स्ट्रिंग जोड़ना चाहता है तो वह:

* 64 "A" का एक MD5 उत्पन्न कर सकता है
* पहले से इनिशियलाइज्ड हैश की स्टेट को 6036708eba0d11f6ef52ad44e8b74d5b में बदल सकता है
* स्ट्रिंग "append" जोड़ सकता है
* हैश को समाप्त कर सकता है और परिणामी हैश "secret" + "data" + "padding" + "append" के लिए एक **वैध होगा**

## **टूल**

{% embed url="https://github.com/iagox86/hash_extender" %}

# संदर्भ

आप इस हमले को अच्छी तरह से समझाया हुआ [https://blog.skullsecurity.org/2012/everything-you-need-to-know-about-hash-length-extension-attacks](https://blog.skullsecurity.org/2012/everything-you-need-to-know-about-hash-length-extension-attacks) में पा सकते हैं।


<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग ट्रिक्स शेयर करें।

</details>
