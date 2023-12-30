# Office फ़ाइल विश्लेषण

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) **का अनुसरण करें.**
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग ट्रिक्स शेयर करें।

</details>

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उपयोग करके आसानी से **वर्कफ्लोज़ को बिल्ड और ऑटोमेट** करें जो दुनिया के **सबसे उन्नत** समुदाय टूल्स द्वारा संचालित होते हैं।\
आज ही एक्सेस प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## परिचय

Microsoft ने **कई दर्जन ऑफिस दस्तावेज़ फ़ाइल प्रारूप** बनाए हैं, जिनमें से कई फ़िशिंग हमलों और मैलवेयर के वितरण के लिए लोकप्रिय हैं क्योंकि इनमें मैक्रोज़ (VBA स्क्रिप्ट्स) **शामिल करने की क्षमता** होती है।

मोटे तौर पर, Office फ़ाइल प्रारूप की दो पीढ़ियाँ हैं: **OLE प्रारूप** (RTF, DOC, XLS, PPT जैसे फ़ाइल एक्सटेंशन), और "**Office Open XML**" प्रारूप (DOCX, XLSX, PPTX जैसे फ़ाइल एक्सटेंशन शामिल हैं)। **दोनों** प्रारूप संरचित, कंपाउंड फ़ाइल बाइनरी प्रारूप हैं जो **लिंक्ड या एम्बेडेड सामग्री** (ऑब्जेक्ट्स) को सक्षम करते हैं। OOXML फ़ाइलें ज़िप फ़ाइल कंटेनर होती हैं, जिसका मतलब है कि छिपे हुए डेटा की जाँच करने का एक सबसे आसान तरीका बस दस्तावेज़ को `unzip` करना है:
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
```markdown
जैसा कि आप देख सकते हैं, कुछ संरचना फ़ाइल और फ़ोल्डर की हायरार्की द्वारा बनाई गई है। शेष XML फ़ाइलों के अंदर निर्दिष्ट है। [_New Steganographic Techniques for the OOXML File Format_, 2011](http://download.springer.com/static/pdf/713/chp%3A10.1007%2F978-3-642-23300-5\_27.pdf?originUrl=http%3A%2F%2Flink.springer.com%2Fchapter%2F10.1007%2F978-3-642-23300-5\_27\&token2=exp=1497911340\~acl=%2Fstatic%2Fpdf%2F713%2Fchp%25253A10.1007%25252F978-3-642-23300-5\_27.pdf%3ForiginUrl%3Dhttp%253A%252F%252Flink.springer.com%252Fchapter%252F10.1007%252F978-3-642-23300-5\_27\*\~hmac=aca7e2655354b656ca7d699e8e68ceb19a95bcf64e1ac67354d8bca04146fd3d) में डेटा छिपाने की तकनीकों के कुछ विचार विस्तार से बताए गए हैं, लेकिन CTF चैलेंज लेखक हमेशा नए विचारों के साथ आते रहेंगे।

एक बार फिर, OLE और OOXML दस्तावेज़ों की जांच और **विश्लेषण** के लिए एक Python टूलसेट मौजूद है: [oletools](http://www.decalage.info/python/oletools)। विशेष रूप से OOXML दस्तावेज़ों के लिए, [OfficeDissector](https://www.officedissector.com) एक बहुत शक्तिशाली विश्लेषण फ्रेमवर्क (और Python लाइब्रेरी) है। इसमें इसके उपयोग की [त्वरित गाइड](https://github.com/grierforensics/officedissector/blob/master/doc/html/\_sources/txt/ANALYZING\_OOXML.txt) भी शामिल है।

कभी-कभी चुनौती छिपे हुए स्थिर डेटा को खोजने की नहीं होती, बल्कि **VBA मैक्रो का विश्लेषण** करने की होती है ताकि उसके व्यवहार का पता लगाया जा सके। यह एक अधिक यथार्थवादी परिदृश्य है और एक ऐसा है जिसे फील्ड में विश्लेषक प्रतिदिन करते हैं। उल्लिखित डिसेक्टर टूल्स यह संकेत दे सकते हैं कि क्या मैक्रो मौजूद है, और शायद इसे आपके लिए निकाल भी सकते हैं। विंडोज पर एक Office दस्तावेज़ में एक विशिष्ट VBA मैक्रो, %TEMP% में एक PowerShell स्क्रिप्ट डाउनलोड करेगा और इसे निष्पादित करने का प्रयास करेगा, जिस स्थिति में अब आपके पास PowerShell स्क्रिप्ट विश्लेषण का कार्य भी होता है। लेकिन दुर्भावनापूर्ण VBA मैक्रो शायद ही कभी जटिल होते हैं क्योंकि VBA [आमतौर पर कोड निष्पादन को बूटस्ट्रैप करने के लिए एक जंपिंग-ऑफ प्लेटफॉर्म के रूप में ही इस्तेमाल किया जाता है](https://www.lastline.com/labsblog/party-like-its-1999-comeback-of-vba-malware-downloaders-part-3/)। जिस स्थिति में आपको एक जटिल VBA मैक्रो को समझने की आवश्यकता हो, या अगर मैक्रो अस्पष्ट है और इसमें एक अनपैकर रूटीन है, तो इसे डिबग करने के लिए आपको Microsoft Office का लाइसेंस रखने की आवश्यकता नहीं है। आप [Libre Office](http://libreoffice.org) का उपयोग कर सकते हैं: [इसका इंटरफेस](http://www.debugpoint.com/2014/09/debugging-libreoffice-macro-basic-using-breakpoint-and-watch/) किसी के लिए भी परिचित होगा जिसने किसी प्रोग्राम को डिबग किया है; आप ब्रेकपॉइंट सेट कर सकते हैं और वॉच वेरिएबल्स बना सकते हैं और उन मूल्यों को कैप्चर कर सकते हैं जो अनपैक हो चुके हैं लेकिन जो कुछ भी पेलोड व्यवहार निष्पादित होने से पहले हैं। आप एक विशिष्ट दस्तावेज़ के मैक्रो को कमांड लाइन से भी शुरू कर सकते हैं:
```
```
$ soffice path/to/test.docx macro://./standard.module1.mymacro
```
## [oletools](https://github.com/decalage2/oletools)
```bash
sudo pip3 install -U oletools
olevba -c /path/to/document #Extract macros
```
## स्वचालित निष्पादन

मैक्रो फ़ंक्शन जैसे कि `AutoOpen`, `AutoExec` या `Document_Open` **स्वचालित रूप से** **निष्पादित** होंगे।

## संदर्भ

* [https://trailofbits.github.io/ctf/forensics/](https://trailofbits.github.io/ctf/forensics/)

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks) का उपयोग करके आसानी से और **स्वचालित वर्कफ़्लोज़** बनाएं जो दुनिया के **सबसे उन्नत** समुदाय उपकरणों द्वारा संचालित होते हैं।\
आज ही पहुंच प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><strong>Learn AWS hacking from zero to hero with</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) में शामिल हों या मुझे **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) **का अनुसरण करें**।
* **अपनी हैकिंग तरकीबें साझा करें, HackTricks** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके।

</details>
