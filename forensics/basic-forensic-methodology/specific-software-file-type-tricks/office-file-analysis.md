# कार्यालय फ़ाइल विश्लेषण

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सदस्यता योजनाएँ देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks) github repos में PRs सबमिट करके।

</details>

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks) का उपयोग करें और दुनिया के **सबसे उन्नत समुदाय उपकरणों** द्वारा संचालित **कार्यप्रवाहों** को आसानी से निर्माण और स्वचालित करें।\
आज ही पहुंचें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

**अधिक विवरण के लिए [https://trailofbits.github.io/ctf/forensics/](https://trailofbits.github.io/ctf/forensics/) देखें**

माइक्रोसॉफ्ट ने कई कार्यालय दस्तावेज़ प्रारूप बनाए हैं, जिनमें दो मुख्य प्रकार हैं **OLE प्रारूप** (जैसे RTF, DOC, XLS, PPT) और **Office Open XML (OOXML) प्रारूप** (जैसे DOCX, XLSX, PPTX)। इन प्रारूपों में मैक्रो शामिल हो सकते हैं, जिन्हें फिशिंग और मैलवेयर के लिए लक्ष्य बनाया जा सकता है। OOXML फ़ाइलें ज़िप कंटेनर के रूप में संरचित होती हैं, जिससे अनज़िप करके फ़ाइल और फ़ोल्डर व्यवस्था और XML फ़ाइल की सामग्री का पता लगाया जा सकता है।

OOXML फ़ाइल संरचनाओं की खोज के लिए दस्तावेज़ को अनज़िप करने के लिए एक कमांड और आउटपुट संरचना दी गई है। इन फ़ाइलों में डेटा छुपाने के तकनीकों का विवरण दस्तावेज़ किया गया है, जिससे CTF चुनौतियों में डेटा छुपाने में जारी नवाचार का संकेत मिलता है।

विश्लेषण के लिए, **oletools** और **OfficeDissector** OLE और OOXML दस्तावेज़ों की जांच के लिए व्यापक उपकरण संग्रह प्रदान करते हैं। ये उपकरण निहित मैक्रो की पहचान और विश्लेषण में मदद करते हैं, जो अक्सर मैलवेयर वितरण के लिए वेक्टर के रूप में काम करते हैं, आम तौर पर अतिरिक्त हानिकारक पेयलोड को डाउनलोड और क्रियान्वित करने के लिए। VBA मैक्रो का विश्लेषण माइक्रोसॉफ्ट ऑफ़िस के बिना लिब्रे ऑफ़िस का उपयोग करके किया जा सकता है, जिसमें ब्रेकपॉइंट्स और वॉच वेरिएबल्स के साथ डीबगिंग की अनुमति होती है।

**oletools** की स्थापना और उपयोग सरल है, जिसमें pip के माध्यम से स्थापना के लिए आदेश प्रदान किए गए हैं और दस्तावेज़ से मैक्रो निकालने के लिए। मैक्रो का स्वचालित क्रियान्वयन `AutoOpen`, `AutoExec`, या `Document_Open` जैसे फ़ंक्शन्स द्वारा प्रेरित किया जाता है।
```bash
sudo pip3 install -U oletools
olevba -c /path/to/document #Extract macros
```
<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उपयोग करें और आसानी से **दुनिया के सबसे उन्नत समुदाय उपकरणों** द्वारा संचालित **कार्यप्रवाह** बनाएं और स्वचालित करें।\
आज ही पहुंचें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सदस्यता योजनाएं देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें** और **HackTricks** और **HackTricks Cloud** github repos में PR जमा करके।

</details>
