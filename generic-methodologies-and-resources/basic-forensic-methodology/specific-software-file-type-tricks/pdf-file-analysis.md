# PDF फ़ाइल विश्लेषण

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **PDF में HackTricks डाउनलोड करना चाहते हैं** तो [**सदस्यता योजनाएं देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** पर **फ़ॉलो** करें 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

<figure><img src="../../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm_source=hacktricks&utm_medium=text&utm_campaign=ppc&utm_term=trickest&utm_content=pdf-file-analysis) का उपयोग करें और दुनिया के **सबसे उन्नत समुदाय उपकरणों** द्वारा संचालित **ऑटोमेट वर्कफ़्लो** आसानी से बनाएं।\
आज ही पहुंच प्राप्त करें:

{% embed url="https://trickest.com/?utm_source=hacktricks&utm_medium=banner&utm_campaign=ppc&utm_content=pdf-file-analysis" %}

**अधिक विवरण के लिए देखें:** [**https://trailofbits.github.io/ctf/forensics/**](https://trailofbits.github.io/ctf/forensics/)

PDF प्रारूप को उसकी जटिलता और डेटा छुपाने की संभावना के लिए जाना जाता है, जिससे यह CTF फोरेंसिक्स चुनौतियों का केंद्रीय बिंदु बन जाता है। यह सादा-पाठ तत्वों को बाइनरी ऑब्जेक्ट्स के साथ मिलाता है, जो संकुचित या एन्क्रिप्ट किया गया हो सकता है, और जिसमें जावास्क्रिप्ट या फ़्लैश जैसी भाषाओं में स्क्रिप्ट शामिल हो सकता है। PDF संरचना को समझने के लिए, किसी भी पाठ संपादक या ऑरिगामी जैसे PDF-विशेषिकृत संपादक का उपयोग किया जा सकता है।

PDF की गहराई में अन्वेषण या परिवर्तन के लिए, [qpdf](https://github.com/qpdf/qpdf) और [Origami](https://github.com/mobmewireless/origami-pdf) जैसे उपकरण उपलब्ध हैं। PDF के भीतर छिपी डेटा को निम्नलिखित में छिपाया जा सकता है:

* अदृश्य परतें
* एडोबे द्वारा XMP मेटाडेटा प्रारूप
* इंक्रीमेंट जनरेशन
* पृष्ठभूमि के समान रंग के पाठ
* छवियों के पीछे या ओवरलैपिंग छवियों के पीछे पाठ
* गैर-प्रदर्शित टिप्पणियाँ

कस्टम PDF विश्लेषण के लिए, [PeepDF](https://github.com/jesparza/peepdf) जैसी Python पुस्तकालय का उपयोग किया जा सकता है ताकि विशेष विश्लेषण स्क्रिप्ट बनाया जा सके। इसके अतिरिक्त, PDF के छिपे डेटा भंडारण की संभावना इतनी विशाल है कि NSA गाइड पर PDF जोखिम और उपाय, जो अब अपने मूल स्थान पर होस्ट नहीं है, फिर भी मूल्यवान अवलोकन प्रदान करते हैं। एक [गाइड की प्रति](http://www.itsecure.hu/library/file/Biztons%C3%A1gi%20%C3%BAtmutat%C3%B3k/Alkalmaz%C3%A1sok/Hidden%20Data%20and%20Metadata%20in%20Adobe%20PDF%20Files.pdf) और Ange Albertini द्वारा [PDF प्रारूप ट्रिक्स](https://github.com/corkami/docs/blob/master/PDF/PDF.md) का संग्रह विषय पर अधिक पठन प्रदान कर सकते हैं।

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **PDF में HackTricks डाउनलोड करना चाहते हैं** तो [**सदस्यता योजनाएं देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** पर **फ़ॉलो** करें 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>
