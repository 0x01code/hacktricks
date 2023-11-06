# कार्यालय फ़ाइल विश्लेषण

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड** करने का उपयोग करना है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल हों** या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का पालन करें**.
* **अपने हैकिंग ट्रिक्स को** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **में PR जमा करके अपने हैकिंग ट्रिक्स साझा करें।**

</details>

<figure><img src="../../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उपयोग करें और आसानी से बनाएं और **स्वचालित कार्यप्रवाह** बनाएं जो दुनिया के **सबसे उन्नत** समुदाय उपकरणों द्वारा संचालित होता है।\
आज ही पहुंच प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## परिचय

माइक्रोसॉफ्ट ने **दर्जनों कार्यालय दस्तावेज़ फ़ाइल प्रारूप** बनाए हैं, जिनमें से कई फ़िशिंग हमलों और मैलवेयर के वितरण के लिए लोकप्रिय हैं क्योंकि उनकी क्षमता होती है **मैक्रो** (VBA स्क्रिप्ट) को शामिल करने की।

व्यापक रूप से, कार्यालय फ़ाइल प्रारूप की दो पीढ़ियां हैं: **OLE प्रारूप** (फ़ाइल एक्सटेंशन जैसे RTF, DOC, XLS, PPT) और "**ऑफ़िस ओपन एमएल**" प्रारूप (फ़ाइल एक्सटेंशन जिसमें DOCX, XLSX, PPTX शामिल हैं)। **दोनों** प्रारूप संरचित, संयुक्त फ़ाइल बाइनरी प्रारूप हैं जो **लिंक्ड या एम्बेडेड सामग्री** (ऑब्जेक्ट्स) को सक्षम करते हैं। OOXML फ़ाइल ज़िप फ़ाइल कंटेनर होती हैं, इसका मतलब है कि छिपी हुई डेटा की जांच करने का सबसे आसान तरीका सीधे डॉक्यूमेंट को `अनज़िप` करना है:
```
$ unzip example.docx
Archive:  example.docx
inflating: [Content_Types].xml
inflating: _rels/.rels
inflating: word/_rels/document.xml.rels
inflating: word/document.xml
inflating: word/theme/theme1.xml
extracting: docProps/thumbnail.jpeg
inflating: word/comments.xml
inflating: word/settings.xml
inflating: word/fontTable.xml
inflating: word/styles.xml
inflating: word/stylesWithEffects.xml
inflating: docProps/app.xml
inflating: docProps/core.xml
inflating: word/webSettings.xml
inflating: word/numbering.xml
$ tree
.
├── [Content_Types].xml
├── _rels
├── docProps
│   ├── app.xml
│   ├── core.xml
│   └── thumbnail.jpeg
└── word
├── _rels
│   └── document.xml.rels
├── comments.xml
├── document.xml
├── fontTable.xml
├── numbering.xml
├── settings.xml
├── styles.xml
├── stylesWithEffects.xml
├── theme
│   └── theme1.xml
└── webSettings.xml
```
जैसा कि आप देख सकते हैं, कुछ संरचना फ़ाइल और फ़ोल्डर हाइरार्की द्वारा बनाई जाती है। शेष भाग XML फ़ाइलों में निर्दिष्ट किया जाता है। [_OOXML फ़ाइल प्रारूप के लिए नए स्टेगनोग्राफिक तकनीकें_, 2011](http://download.springer.com/static/pdf/713/chp%3A10.1007%2F978-3-642-23300-5\_27.pdf?originUrl=http%3A%2F%2Flink.springer.com%2Fchapter%2F10.1007%2F978-3-642-23300-5\_27\&token2=exp=1497911340\~acl=%2Fstatic%2Fpdf%2F713%2Fchp%25253A10.1007%25252F978-3-642-23300-5\_27.pdf%3ForiginUrl%3Dhttp%253A%252F%252Flink.springer.com%252Fchapter%252F10.1007%252F978-3-642-23300-5\_27\*\~hmac=aca7e2655354b656ca7d699e8e68ceb19a95bcf64e1ac67354d8bca04146fd3d) डेटा छिपाने की तकनीकों के बारे में कुछ विचारों का विवरण देता है, लेकिन सीटीएफ चुनौती लेखक हमेशा नए विचार लाते रहेंगे।

एक बार फिर, OLE और OOXML दस्तावेज़ों के परीक्षण और विश्लेषण के लिए एक Python टूलसेट मौजूद है: [oletools](http://www.decalage.info/python/oletools)। विशेष रूप से OOXML दस्तावेज़ों के लिए, [OfficeDissector](https://www.officedissector.com) एक बहुत ही शक्तिशाली विश्लेषण ढांचा (और Python पुस्तकालय) है। इसमें एक [उपयोग के लिए त्वरित गाइड](https://github.com/grierforensics/officedissector/blob/master/doc/html/\_sources/txt/ANALYZING\_OOXML.txt) भी शामिल है।

कभी-कभी चुनौती यह होती है कि छिपे स्थिर डेटा को खोजने के बजाय एक VBA मैक्रो का विश्लेषण करके उसके व्यवहार को निर्धारित किया जाए। यह एक अधिक वास्तविक स्थिति है और जो कि क्षेत्र में विश्लेषक हर दिन करते हैं। पहले उल्लिखित विश्लेषक उपकरण यह दिखा सकते हैं कि क्या एक मैक्रो मौजूद है, और शायद आपके लिए इसे निकाल सकते हैं। विंडोज पर एक ऑफिस दस्तावेज़ में एक साधारण VBA मैक्रो, %TEMP% में एक PowerShell स्क्रिप्ट डाउनलोड करेगा और इसे निष्पादित करने का प्रयास करेगा, जिसके बाद आपके पास अब एक PowerShell स्क्रिप्ट विश्लेषण का कार्य हो गया है। लेकिन खतरनाक VBA मैक्रो अक्सर संयोजन किए जाते हैं क्योंकि VBA [आमतौर पर केवल कोड निष्पादन के लिए उपयोग किया जाता है](https://www.lastline.com/labsblog/party-like-its-1999-comeback-of-vba-malware-downloaders-part-3/)। जहां आपको एक जटिल VBA मैक्रो को समझने की आवश्यकता होती है, या यदि मैक्रो अस्पष्ट है और उसमें एक अनपैकर रूटीन है, तो आपको इसे डीबग करने के लिए माइक्रोसॉफ्ट ऑफिस का लाइसेंस नहीं चाहिए। आप [Libre Office](http://libreoffice.org) का उपयोग कर सकते हैं: [इसका इंटरफ़ेस](http://www.debugpoint.com/2014/09/debugging-libreoffice-macro-basic-using-breakpoint-and-watch/) किसी भी व्यक्ति को परियोजना को डीबग करने का अनुभव होगा; आप ब्रेकपॉइंट सेट कर सकते हैं और वॉच वेरिएबल बना सकते हैं और उनके मान को पकड़ सकते हैं जब वे अनपैक हो चुके हों लेकिन जो कुछ भी पेलोड व्यवहार निष्पादित हो चुका हो। आप एक विशेष दस्तावेज़ का मैक्रो भी कमांड लाइन से शुरू कर सकते हैं:
```
$ soffice path/to/test.docx macro://./standard.module1.mymacro
```
## [oletools](https://github.com/decalage2/oletools)

oletools एक संग्रह है जो ओएलई (OLE) फ़ाइल फ़ॉरमेट के विभिन्न प्रकार के दुष्प्रभावित फ़ाइलों के विश्लेषण के लिए उपयोगी है। यह टूल्स ओएलई फ़ाइलों की जांच करने और उनमें छिपी जासूसी या अनुमति नियंत्रण को खोजने में मदद करता है। इसके अलावा, यह फ़ाइलों के भीतर वायरस, मैलवेयर और अन्य खतरनाक संलग्नकों की खोज करने में भी सक्षम है।

यह टूल्स विभिन्न ओएलई फ़ाइल फ़ॉरमेटों के लिए समर्थित है, जैसे DOC, PPT, XLS, PUB, VSD, MSG, और अन्य। इसके साथ, यह विभिन्न विश्लेषण तकनीकों का उपयोग करता है, जैसे कि फ़ाइल के भीतर के ऑब्जेक्ट्स, मैक्रो, और वीबीए मैक्रो की जांच करना।

यह टूल्स एक कमांड लाइन इंटरफ़ेस के साथ आता है, जिससे आप फ़ाइलों को विश्लेषण कर सकते हैं और उपयोगकर्ता द्वारा परिभाषित नियमों के आधार पर चेक कर सकते हैं। इसके अलावा, यह एक ग्राफिकल उपयोगकर्ता इंटरफ़ेस (GUI) भी प्रदान करता है जो उपयोगकर्ताओं को आसानी से फ़ाइलों का विश्लेषण करने और उन्हें संशोधित करने की अनुमति देता है।
```bash
sudo pip3 install -U oletools
olevba -c /path/to/document #Extract macros
```
## स्वचालित निष्पादन

`AutoOpen`, `AutoExec` या `Document_Open` जैसे मैक्रो फंक्शन स्वचालित रूप से निष्पादित होंगे।

## संदर्भ

* [https://trailofbits.github.io/ctf/forensics/](https://trailofbits.github.io/ctf/forensics/)

<figure><img src="../../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उपयोग करें ताकि आसानी से वर्कफ़्लो बना सकें और दुनिया के सबसे उन्नत समुदाय उपकरणों द्वारा कार्यों को स्वचालित कर सकें।\
आज ही पहुंच प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप चाहते हैं कि आपकी **कंपनी HackTricks में विज्ञापित** हो? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की अनुमति चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT संग्रह**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल हों** या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का पालन करें**।**
* **अपने हैकिंग ट्रिक्स को** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **में पीआर जमा करके अपने हैकिंग ट्रिक्स साझा करें**।

</details>
