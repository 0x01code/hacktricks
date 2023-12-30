# पिकल रिक

## पिकल रिक

<details>

<summary><strong>शून्य से हीरो तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS रेड टीम एक्सपर्ट)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **अपनी हैकिंग ट्रिक्स साझा करें, HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके.

</details>

![](../../.gitbook/assets/picklerick.gif)

यह मशीन आसान श्रेणी में थी और यह वास्तव में आसान थी।

## एन्युमरेशन

मैंने मेरे टूल [**Legion**](https://github.com/carlospolop/legion) का उपयोग करके मशीन का **एन्युमरेशन शुरू किया**:

![](<../../.gitbook/assets/image (79) (2).png>)

जैसा कि आप देख सकते हैं 2 पोर्ट्स खुले हैं: 80 (**HTTP**) और 22 (**SSH**)

इसलिए, मैंने HTTP सेवा का एन्युमरेशन करने के लिए legion लॉन्च किया:

![](<../../.gitbook/assets/image (234).png>)

ध्यान दें कि छवि में आप देख सकते हैं कि `robots.txt` में स्ट्रिंग `Wubbalubbadubdub` है

कुछ सेकंड्स के बाद मैंने जांच की कि `disearch` ने पहले ही क्या खोजा है:

![](<../../.gitbook/assets/image (235).png>)

![](<../../.gitbook/assets/image (236).png>)

और जैसा कि आप अंतिम छवि में देख सकते हैं एक **लॉगिन** पेज की खोज की गई थी।

रूट पेज के सोर्स कोड की जांच करने पर, एक यूजरनेम मिला: `R1ckRul3s`

![](<../../.gitbook/assets/image (237) (1).png>)

इसलिए, आप लॉगिन पेज पर `R1ckRul3s:Wubbalubbadubdub` क्रेडेंशियल्स का उपयोग करके लॉगिन कर सकते हैं

## यूजर

उन क्रेडेंशियल्स का उपयोग करके आप एक पोर्टल तक पहुंचेंगे जहां आप कमांड्स निष्पादित कर सकते हैं:

![](<../../.gitbook/assets/image (241).png>)

कुछ कमांड्स जैसे कि cat अनुमति नहीं हैं लेकिन आप पहला इंग्रेडिएंट (फ्लैग) पढ़ सकते हैं, उदाहरण के लिए grep का उपयोग करके:

![](<../../.gitbook/assets/image (242).png>)

फिर मैंने उपयोग किया:

![](<../../.gitbook/assets/image (243) (1).png>)

रिवर्स शेल प्राप्त करने के लिए:

![](<../../.gitbook/assets/image (239) (1).png>)

**दूसरा इंग्रेडिएंट** `/home/rick` में पाया जा सकता है

![](<../../.gitbook/assets/image (240).png>)

## रूट

यूजर **www-data कुछ भी सुडो के रूप में निष्पादित कर सकता है**:

![](<../../.gitbook/assets/image (238).png>)

<details>

<summary><strong>शून्य से हीरो तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS रेड टीम एक्सपर्ट)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **अपनी हैकिंग ट्रिक्स साझा करें, HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके.

</details>
