# PDF फ़ाइल विश्लेषण

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **PDF में HackTricks डाउनलोड करना चाहते हैं** तो [**सदस्यता योजनाएँ देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** पर **फ़ॉलो** करें 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks) का उपयोग करें और दुनिया के **सबसे उन्नत समुदाय उपकरणों** द्वारा संचालित **वर्कफ़्लो** को आसानी से बनाएं और स्वचालित करें।\
आज ही पहुंचें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

स्रोत: [https://trailofbits.github.io/ctf/forensics/](https://trailofbits.github.io/ctf/forensics/)

PDF एक अत्यधिक जटिल दस्तावेज़ फ़ाइल प्रारूप है, जिसमें पर्याप्त ट्रिक्स और छुपाने की जगहें हैं [जिनके बारे में वर्षों तक लिखा जा सकता है](https://www.sultanik.com/pocorgtfo/)। यह CTF फोरेंसिक्स चुनौतियों के लिए भी लोकप्रिय बनाता है। NSA ने 2008 में इन छुपाने की जगहों पर एक मार्गदर्शिका लिखी थी जिसका शीर्षक था "Hidden Data and Metadata in Adobe PDF Files: Publication Risks and Countermeasures." यह अब अपने मूल URL पर उपलब्ध नहीं है, लेकिन आप [यहाँ एक प्रति पा सकते हैं](http://www.itsecure.hu/library/file/Biztons%C3%A1gi%20%C3%BAtmutat%C3%B3k/Alkalmaz%C3%A1sok/Hidden%20Data%20and%20Metadata%20in%20Adobe%20PDF%20Files.pdf)। Ange Albertini ने GitHub पर [PDF फ़ाइल प्रारूप ट्रिक्स](https://github.com/corkami/docs/blob/master/PDF/PDF.md) की एक विकि भी बनाई है।

PDF प्रारूप आंशिक रूप से सादा-टेक्स्ट है, जैसे HTML, लेकिन इसमें कई बाइनरी "ऑब्जेक्ट्स" भी होते हैं। Didier Stevens ने प्रारूप के बारे में [अच्छी प्रारंभिक सामग्री](https://blog.didierstevens.com/2008/04/09/quickpost-about-the-physical-and-logical-structure-of-pdf-files/) लिखी है। बाइनरी ऑब्ज
