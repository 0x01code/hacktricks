# macOS सीरियल नंबर

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **अपनी हैकिंग ट्रिक्स साझा करें, HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके.

</details>

2010 के बाद निर्मित Apple उपकरणों में आमतौर पर **12-अक्षर अल्फ़ान्यूमेरिक** सीरियल नंबर होते हैं, जिसमें **पहले तीन अंक निर्माण स्थान को दर्शाते हैं**, उसके बाद के **दो** अंक **वर्ष** और **सप्ताह** के निर्माण को इंगित करते हैं, अगले **तीन** अंक एक **अद्वितीय** **पहचानकर्ता** प्रदान करते हैं, और **अंतिम** **चार** अंक **मॉडल नंबर** को दर्शाते हैं।

सीरियल नंबर उदाहरण: **C02L13ECF8J2**

### **3 - निर्माण स्थान**

| कोड           | फैक्टरी                                      |
| -------------- | -------------------------------------------- |
| FC             | Fountain Colorado, USA                       |
| F              | Fremont, California, USA                     |
| XA, XB, QP, G8 | USA                                          |
| RN             | Mexico                                       |
| CK             | Cork, Ireland                                |
| VM             | Foxconn, Pardubice, Czech Republic           |
| SG, E          | Singapore                                    |
| MB             | Malaysia                                     |
| PT, CY         | Korea                                        |
| EE, QT, UV     | Taiwan                                       |
| FK, F1, F2     | Foxconn – Zhengzhou, China                   |
| W8             | Shanghai China                               |
| DL, DM         | Foxconn – China                              |
| DN             | Foxconn, Chengdu, China                      |
| YM, 7J         | Hon Hai/Foxconn, China                       |
| 1C, 4H, WQ, F7 | China                                        |
| C0             | Tech Com – Quanta Computer Subsidiary, China |
| C3             | Foxxcon, Shenzhen, China                     |
| C7             | Pentragon, Changhai, China                   |
| RM             | Refurbished/remanufactured                   |

### 1 - निर्माण का वर्ष

| कोड | रिलीज़              |
| ---- | -------------------- |
| C    | 2010/2020 (पहली छमाही) |
| D    | 2010/2020 (दूसरी छमाही) |
| F    | 2011/2021 (पहली छमाही) |
| G    | 2011/2021 (दूसरी छमाही) |
| H    | 2012/... (पहली छमाही)  |
| J    | 2012 (दूसरी छमाही)      |
| K    | 2013 (पहली छमाही)      |
| L    | 2013 (दूसरी छमाही)      |
| M    | 2014 (पहली छमाही)      |
| N    | 2014 (दूसरी छमाही)      |
| P    | 2015 (पहली छमाही)      |
| Q    | 2015 (दूसरी छमाही)      |
| R    | 2016 (पहली छमाही)      |
| S    | 2016 (दूसरी छमाही)      |
| T    | 2017 (पहली छमाही)      |
| V    | 2017 (दूसरी छमाही)      |
| W    | 2018 (पहली छमाही)      |
| X    | 2018 (दूसरी छमाही)      |
| Y    | 2019 (पहली छमाही)      |
| Z    | 2019 (दूसरी छमाही)      |

### 1 - निर्माण का सप्ताह

पांचवां अक्षर उस सप्ताह को दर्शाता है जिसमें उपकरण का निर्माण किया गया था। इस स्थान पर 28 संभावित अक्षर हैं: **अंक 1-9 का उपयोग पहले से नौवें सप्ताह को दर्शाने के लिए किया जाता है**, और **अक्षर C से Y**, स्वर A, E, I, O, और U, और अक्षर S को **छोड़कर**, **दसवें से सत्ताईसवें सप्ताह को दर्शाते हैं**। यदि उपकरण **वर्ष की दूसरी छमाही में निर्मित है, तो पांचवें अक्षर द्वारा दर्शाए गए नंबर में 26 जोड़ें**। उदाहरण के लिए, एक उत्पाद जिसके सीरियल नंबर के चौथे और पांचवें अंक “JH” हैं, उसका निर्माण 2012 के 40वें सप्ताह में हुआ था।

### 3 - अद्वितीय कोड

अगले तीन अंक एक पहचानकर्ता कोड हैं जो **प्रत्येक Apple उपकरण को अलग करने के लिए कार्य करता है** जो एक ही मॉडल का है और एक ही स्थान पर और एक ही वर्ष के एक ही सप्ताह में निर्मित होता है, यह सुनिश्चित करता है कि प्रत्येक उपकरण का एक अलग सीरियल नंबर हो।

### 4 - सीरियल नंबर

सीरियल नंबर के अंतिम चार अंक **उत्पाद के मॉडल** को दर्शाते हैं।

### संदर्भ

{% embed url="https://beetstech.com/blog/decode-meaning-behind-apple-serial-number" %}

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **अपनी हैकिंग ट्रिक्स साझा करें, HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके.

</details>
