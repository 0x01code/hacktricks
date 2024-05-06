# कार्यालय फ़ाइल विश्लेषण

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन **The PEASS Family** की खोज करें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

<figure><img src="../../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm_source=hacktricks&utm_medium=text&utm_campaign=ppc&utm_content=office-file-analysis) का उपयोग करें और **दुनिया के सबसे उन्नत समुदाय उपकरणों द्वारा संचालित** **कार्यप्रणालियों** को आसानी से निर्माण करें।\
आज ही पहुंचें:

{% embed url="https://trickest.com/?utm_source=hacktricks&utm_medium=banner&utm_campaign=ppc&utm_content=office-file-analysis" %}

अधिक जानकारी के लिए [https://trailofbits.github.io/ctf/forensics/](https://trailofbits.github.io/ctf/forensics/) देखें। यह केवल एक सारांश है:

माइक्रोसॉफ्ट ने कई ऑफिस दस्तावेज़ प्रारूप बनाए हैं, जिनमें दो मुख्य प्रकार हैं **OLE प्रारूप** (जैसे RTF, DOC, XLS, PPT) और **Office Open XML (OOXML) प्रारूप** (जैसे DOCX, XLSX, PPTX)। इन प्रारूपों में मैक्रो शामिल हो सकते हैं, जिन्हें फिशिंग और मैलवेयर के लिए लक्ष्य बनाया जा सकता है। OOXML फ़ाइलें ज़िप कंटेनर के रूप में संरचित होती हैं, जिससे अनजिपिंग के माध्यम से जांच की जा सकती है, फ़ाइल और फ़ोल्डर व्यवस्था और XML फ़ाइल सामग्री को प्रकट करते हैं।

OOXML फ़ाइल संरचनाओं की खोज के लिए एक दस्तावेज़ को अनजिप करने के लिए और आउटपुट संरचना दी गई है। इन फ़ाइलों में डेटा छुपाने के तकनीकों का विवरण दस्तावेज़ किया गया है, जो सीटीएफ चुनौतियों में डेटा छुपाने में जारी नवाचार को दर्शाता है।

विश्लेषण के लिए, **oletools** और **OfficeDissector** OLE और OOXML दस्तावेज़ों की जांच के लिए व्यापक उपकरण सेट प्रदान करते हैं। ये उपकरण सम्मिलित मैक्रो की पहचान और विश्लेषण में मदद करते हैं, जो आम तौर पर मैलवेयर वितरण के लिए वेक्टर के रूप में काम करते हैं, जिसमें सामान्यत: अतिरिक्त हानिकारक पेलोड डाउनलोड और क्रियान्वित करने की जानकारी होती है। VBA मैक्रो का विश्लेषण माइक्रोसॉफ्ट ऑफिस के बिना लिब्रे ऑफिस का उपयोग करके किया जा सकता है, जो ब्रेकपॉइंट्स और वॉच वेरिएबल्स के साथ डीबगिंग की अनुमति देता है।

**oletools** की स्थापना और उपयोग सरल है, जिसमें पाइप के माध्यम से स्थापना और दस्तावेज़ से मैक्रो निकालने के लिए दिए गए कमांड हैं। मैक्रो का स्वचालित क्रियान्वित होना `AutoOpen`, `AutoExec`, या `Document_Open` जैसे फ़ंक्शन्स द्वारा ट्रिगर किया जाता है।
```bash
sudo pip3 install -U oletools
olevba -c /path/to/document #Extract macros
```
<figure><img src="../../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm_source=hacktricks&utm_medium=text&utm_campaign=ppc&utm_content=office-file-analysis) का उपयोग करें और आसानी से **ऑटोमेट वर्कफ़्लो** बनाएं जो दुनिया के **सबसे उन्नत** समुदाय उपकरणों द्वारा संचालित हैं।\
आज ही पहुंचें:

{% embed url="https://trickest.com/?utm_source=hacktricks&utm_medium=banner&utm_campaign=ppc&utm_content=office-file-analysis" %}

<details>

<summary><strong>जीरो से हीरो तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी कंपनी को **HackTricks में विज्ञापित करना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फ़ॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें और** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos पर PRs सबमिट करके।

</details>
