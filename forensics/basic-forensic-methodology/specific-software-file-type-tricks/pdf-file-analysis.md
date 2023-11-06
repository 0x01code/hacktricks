# PDF फ़ाइल विश्लेषण

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड** करने की उपलब्धता चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल हों** या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का पालन करें**.
* **अपने हैकिंग ट्रिक्स को** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **के माध्यम से PR जमा करके साझा करें**.

</details>

<figure><img src="../../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उपयोग करें और आसानी से **वर्कफ़्लो बनाएं और स्वचालित करें** जो दुनिया के **सबसे उन्नत समुदाय उपकरणों** द्वारा संचालित होते हैं।\
आज ही पहुंच प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

स्रोत: [https://trailofbits.github.io/ctf/forensics/](https://trailofbits.github.io/ctf/forensics/)

PDF एक अत्यंत जटिल दस्तावेज़ फ़ाइल प्रारूप है, जिसमें काफी ट्रिक्स और छिपाने की जगहें होती हैं [जिनके बारे में वर्षों तक लिखा जा सकता है](https://www.sultanik.com/pocorgtfo/)। इसलिए इसे CTF फोरेंसिक्स चुनौतियों के लिए भी लोकप्रिय बनाता है। NSA ने 2008 में इन छिपाने की जगहों पर एक गाइड लिखी थी जिसका शीर्षक था "Hidden Data and Metadata in Adobe PDF Files: Publication Risks and Countermeasures"। यह अपने मूल URL पर अब उपलब्ध नहीं है, लेकिन आप [यहां एक प्रतिलिपि पा सकते हैं](http://www.itsecure.hu/library/file/Biztons%C3%A1gi%20%C3%BAtmutat%C3%B3k/Alkalmaz%C3%A1sok/Hidden%20Data%20and%20Metadata%20in%20Adobe%20PDF%20Files.pdf)। एंज अल्बर्टिनी ने भी GitHub पर [PDF फ़ाइल प्रारूप ट्रिक्स](https://github.com/corkami/docs/blob/master/PDF/PDF.md) की एक विकि रखी है।

PDF प्रारूप आंशिक रूप से सादा-पाठ है, जैसे HTML, लेकिन इसमें कई बाइनरी "ऑब्जेक्ट" होते हैं। दिदिये स्टीवेंस ने इस प्रारूप के बारे में [अच्छी प्रारंभिक सामग्री](https://blog.didierstevens.com/2008/04/09/quickpost-about-the-physical-and-logical-structure-of-pdf-files/) लिखी है। बाइनरी ऑब्जेक्ट्स संपीड़ित या यहां तक कि एन्क्रिप्टेड डेटा हो सकते हैं, और इसमें जावास्क्रिप्ट या फ़्लैश जैसी स्क्रिप्टिंग भाषाओं में सामग्री भी शामिल हो सकती है। एक PDF की संरचना प्रदर्शित करने के लिए, आप इसे एक पाठ संपादक के साथ ब्राउज़ कर सकते हैं या इसे Origami जैसे PDF-ज्ञानी फ़ाइल-प्रारूप संपादक के साथ खोल सकते हैं।

[qpdf](https://github.com/qpdf/qpdf) एक ऐसा उपकरण है जिसका उपयोग PDF की जांच और इससे सूचना का परिवर्तन या निकालने के लिए किया जा सकता है। एक और उपकरण है जो Ruby में एक फ़्रेमवर्क है,
