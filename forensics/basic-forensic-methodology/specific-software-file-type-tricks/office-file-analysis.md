# कार्यालय फ़ाइल विश्लेषण

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) और हमें **ट्विटर** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** द्वारा PRs सबमिट करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks) का उपयोग करें और आसानी से **वर्कफ़्लो** बनाएं और **स्वचालित करें** जो दुनिया के **सबसे उन्नत** समुदाय उपकरणों द्वारा संभाला जाता है।\
आज ही पहुंचें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## परिचय

माइक्रोसॉफ्ट ने **कई कार्यालय दस्तावेज़ फ़ाइल प्रारूप** बनाए हैं, जिनमें से कई फ़ाइल प्रारूप फिशिंग हमलों और मैलवेयर के वितरण के लिए लोकप्रिय हैं क्योंकि इनमें **मैक्रो** (VBA स्क्रिप्ट) शामिल करने की क्षमता होती है।

व्यापक रूप से, कार्यालय फ़ाइल प्रारूप के दो पीढ़ियाँ हैं: **OLE प्रारूप** (फ़ाइल एक्सटेंशन जैसे RTF, DOC, XLS, PPT), और "**ऑफ़िस ओपन एक्सएमएल**" प्रारूप (फ़ाइल एक्सटेंशन जो DOCX, XLSX, PPTX शामिल हैं)। **दोनों** प्रारूप संरचित, संयुक्त फ़ाइल बाइनरी प्रारूप हैं जो **लिंक्ड या एम्बेडेड सामग्री** (ऑब्जेक्ट्स) को सक्षम करते हैं। OOXML फ़ाइल ज़िप फ़ाइल कंटेनर होती हैं, इसका मतलब है कि छिपी डेटा की जांच के लिए सबसे आसान तरीका बस डॉक्यूमेंट को `अनज़िप` करना है:
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
जैसा कि आप देख सकते हैं, कुछ संरचना फ़ाइल और फ़ोल्डर व्यवस्था द्वारा बनाई गई है। शेष भाग XML फ़ाइलों में निर्दिष्ट किया गया है। [_OOXML फ़ाइल प्रारूप के लिए नए स्टेगनोग्राफिक तकनीक_, 2011](http://download.springer.com/static/pdf/713/chp%3A10.1007%2F978-3-642-23300-5\_27.pdf?originUrl=http%3A%2F%2Flink.springer.com%2Fchapter%2F10.1007%2F978-3-642-23300-5\_27\&token2=exp=1497911340\~acl=%2Fstatic%2Fpdf%2F713%2Fchp%25253A10.1007%25252F978-3-642-23300-5\_27.pdf%3ForiginUrl%3Dhttp%253A%252F%252Flink.springer.com%252Fchapter%252F10.1007%252F978-3-642-23300-5\_27\*\~hmac=aca7e2655354b656ca7d699e8e68ceb19a95bcf64e1ac67354d8bca04146fd3d) कुछ डेटा छुपाने की तकनीकों के लिए कुछ विचारों का विवरण देता है, लेकिन CTF चैलेंज लेखक हमेशा नए विचार ला रहे होंगे।

एक बार फिर, OLE और OOXML दस्तावेजों की जांच और **विश्लेषण के लिए एक Python टूलसेट** मौजूद है: [oletools](http://www.decalage.info/python/oletools)। विशेष रूप से OOXML दस्तावेजों के लिए, [OfficeDissector](https://www.officedissector.com) एक बहुत ही शक्तिशाली विश्लेषण ढांचा (और Python पुस्तकालय) है। इसमें एक [उपयोग के लिए त्वरित मार्गदर्शिका](https://github.com/grierforensics/officedissector/blob/master/doc/html/\_sources/txt/ANALYZING\_OOXML.txt) शामिल है।

कभी-कभी चुनौती छिपी स्थिर डेटा खोजने की नहीं होती, बल्कि **एक VBA मैक्रो का विश्लेषण करना** होता है ताकि इसका व्यवहार निर्धारित किया जा सके। यह एक अधिक वास्तविक स्थिति है और जिसे क्षेत्र में विश्लेषक हर दिन करते हैं। उक्त विश्लेषक उपकरण दिखा सकते हैं कि क्या एक मैक्रो मौजूद है, और शायद आपके लिए इसे निकाल सकते हैं। एक सामान्य VBA मैक्रो एक ऑफिस दस्तावेज में, Windows पर, %TEMP% में एक PowerShell स्क्रिप्ट डाउनलोड करेगा और इसे क्रियान्वित करने का प्रयास करेगा, जिसके बाद आपके पास अब एक PowerShell स्क्रिप्ट विश्लेषण कार्य भी है। लेकिन दुर्भाग्यपूर्ण VBA मैक्रो शायद ही जटिल हों क्योंकि VBA [आमतौर पर केवल कोड निष्पादन को बूटस्ट्रैप करने के लिए उपयोग किया जाता है](https://www.lastline.com/labsblog/party-like-its-1999-comeback-of-vba-malware-downloaders-part-3/)। उस स्थिति में जहां आपको एक जटिल VBA मैक्रो को समझने की आवश्यकता है, या यदि मैक्रो अस्पष्ट है और उसमें एक अनपैकर रूटीन है, तो आपको इसे डीबग करने के लिए माइक्रोसॉफ्ट ऑफिस का लाइसेंस नहीं होने की आवश्यकता नहीं है। आप [Libre Office](http://libreoffice.org) का उपयोग कर सकते हैं: [इसका इंटरफेस](http://www.debugpoint.com/2014/09/debugging-libreoffice-macro-basic-using-breakpoint-and-watch/) किसी भी व्यक्ति को परियोजना की डीबग की है; आप ब्रेकपॉइंट सेट कर सकते हैं और वॉच वेरिएबल बना सकते हैं और उन्हें पैक किए जाने के बाद उनके मान को पकड़ सकते हैं लेकिन जब तक कोई पेलोड व्यवहार नहीं होता है। आप एक विशिष्ट दस्तावेज का मैक्रो भी कमांड लाइन से शुरू कर सकते हैं:
```
$ soffice path/to/test.docx macro://./standard.module1.mymacro
```
## [oletools](https://github.com/decalage2/oletools)
```bash
sudo pip3 install -U oletools
olevba -c /path/to/document #Extract macros
```
## स्वचालित क्रियान्वयन

`AutoOpen`, `AutoExec` या `Document_Open` जैसे मैक्रो फ़ंक्शन **स्वचालित रूप से क्रियान्वित** होंगे।

## संदर्भ

* [https://trailofbits.github.io/ctf/forensics/](https://trailofbits.github.io/ctf/forensics/)

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks) का उपयोग करें और दुनिया के **सबसे उन्नत समुदाय उपकरणों** द्वारा संचालित **कार्यप्रणालियों** को आसानी से निर्मित करें और स्वचालित करें।\
आज ही पहुंच प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><strong>शून्य से हीरो तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी कंपनी का विज्ञापन HackTricks में देखना चाहते हैं या **PDF में HackTricks डाउनलोड** करना चाहते हैं तो [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)** पर फ़ॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके।

</details>
