# PDF फ़ाइल विश्लेषण

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert) के साथ शून्य से नायक तक AWS हैकिंग सीखें</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* अपनी हैकिंग ट्रिक्स साझा करें [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके.

</details>

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks) का उपयोग करके आसानी से **वर्कफ्लोज़ को बिल्ड और ऑटोमेट** करें जो दुनिया के **सबसे उन्नत** समुदाय टूल्स द्वारा संचालित होते हैं.\
आज ही एक्सेस प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

से: [https://trailofbits.github.io/ctf/forensics/](https://trailofbits.github.io/ctf/forensics/)

PDF एक अत्यंत जटिल दस्तावेज़ फ़ाइल प्रारूप है, जिसमें छिपाने के लिए पर्याप्त चालें और स्थान हैं [वर्षों तक लिखने के लिए](https://www.sultanik.com/pocorgtfo/). यह CTF फोरेंसिक चुनौतियों के लिए भी लोकप्रिय है. NSA ने 2008 में "Hidden Data and Metadata in Adobe PDF Files: Publication Risks and Countermeasures" शीर्षक से इन छिपे स्थानों के बारे में एक गाइड लिखी थी. यह अब अपने मूल URL पर उपलब्ध नहीं है, लेकिन आप [यहाँ एक प्रति पा सकते हैं](http://www.itsecure.hu/library/file/Biztons%C3%A1gi%20%C3%BAtmutat%C3%B3k/Alkalmaz%C3%A1sok/Hidden%20Data%20and%20Metadata%20in%20Adobe%20PDF%20Files.pdf). Ange Albertini भी GitHub पर [PDF फ़ाइल प्रारूप चालें](https://github.com/corkami/docs/blob/master/PDF/PDF.md) की एक विकी रखते हैं.

PDF प्रारूप आंशिक रूप से प्लेन-टेक्स्ट है, HTML की तरह, लेकिन सामग्री में कई बाइनरी "ऑब्जेक्ट्स" के साथ. Didier Stevens ने प्रारूप के बारे में [अच्छी प्रारंभिक सामग्री](https://blog.didierstevens.com/2008/04/09/quickpost-about-the-physical-and-logical-structure-of-pdf-files/) लिखी है. बाइनरी ऑब्जेक्ट्स में संपीड़ित या यहां तक कि एन्क्रिप्टेड डेटा हो सकता है, और इसमें JavaScript या Flash जैसी स्क्रिप्टिंग भाषाओं में सामग्री शामिल हो सकती है. PDF की संरचना प्रदर्शित करने के लिए, आप इसे टेक्स्ट एडिटर के साथ ब्राउज़ कर सकते हैं या Origami जैसे PDF-जागरूक फ़ाइल-प्रारूप एडिटर के साथ खोल सकते हैं.

[qpdf](https://github.com/qpdf/qpdf) एक ऐसा टूल है जो PDF का पता लगाने और उससे जानकारी निकालने या परिवर्तित करने के लिए उपयोगी हो सकता है. एक अन्य Ruby में एक फ्रेमवर्क है जिसे [Origami](https://github.com/mobmewireless/origami-pdf) कहा जाता है.

PDF सामग्री का पता लगाते समय छिपे हुए डेटा के लिए कुछ जांचने वाले स्थानों में शामिल हैं:

* गैर-दृश्यमान परतें
* Adobe का मेटाडेटा प्रारूप "XMP"
* PDF की "incremental generation" सुविधा जिसमें पिछला संस्करण संरक्षित होता है लेकिन उपयोगकर्ता को दृश्यमान नहीं होता
* सफेद पृष्ठभूमि पर सफेद टेक्स्ट
* छवियों के पीछे टेक्स्ट
* एक ओवरलैपिंग छवि के पीछे एक छवि
* प्रदर्शित नहीं किए गए टिप्पणियां

PDF फ़ाइल प्रारूप के साथ काम करने के लिए कई Python पैकेज भी हैं, जैसे कि [PeepDF](https://github.com/jesparza/peepdf), जो आपको अपनी पार्सिंग स्क्रिप्ट्स लिखने की अनुमति देते हैं.

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert) के साथ शून्य से नायक तक AWS हैकिंग सीखें</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* अपनी हैकिंग ट्रिक्स साझा करें [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके.

</details>
