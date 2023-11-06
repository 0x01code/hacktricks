# macOS सीरियल नंबर

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल** हों या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का पालन करें**.
* **अपने हैकिंग ट्रिक्स को** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में पीआर जमा करके अपने हैकिंग ट्रिक्स साझा करें।**

</details>

2010 के बाद निर्मित Apple उपकरणों में आमतौर पर **12-अक्षरीय अल्फान्यूमेरिक** सीरियल नंबर होते हैं, जिसमें **पहले तीन अंक निर्माण स्थान** को प्रतिष्ठानित करते हैं, अगले **दो** वर्ष और सप्ताह को प्रतिष्ठानित करते हैं, अगले **तीन** अंक एक **अद्वितीय पहचानकर्ता** प्रदान करते हैं, और अंतिम **चार** अंक मॉडल नंबर को प्रतिष्ठानित करते हैं।

सीरियल नंबर उदाहरण: **C02L13ECF8J2**

### **3 - निर्माण स्थान**

| कोड | कारख़ाना                                      |
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

| कोड | विमोचन              |
| ---- | -------------------- |
| C    | 2010/2020 (पहला अर्धवर्ष) |
| D    | 2010/2020 (दूसरा अर्धवर्ष) |
| F    | 2011/2021 (पहला अर्धवर्ष) |
| G    | 2011/2021 (दूसरा अर्धवर्ष) |
| H    | 2012/... (पहला अर्धवर्ष)  |
| J    | 2012 (दूसरा अर्धवर्ष)      |
| K    | 2013 (पहला अर्धवर्ष)      |
| L    | 2013 (दूसरा अर्धवर्ष)      |
| M    | 2014 (पहला अर्धवर्ष)      |
| N    | 2014 (दूसरा अर्धवर्ष)      |
| P    | 2015 (पहला अर्धवर्ष)      |
| Q    | 2015 (दूसरा अर्धवर्ष)      |
| R    | 2016 (पहला अर्धवर्ष)      |
| S    | 2016 (दूसरा अर्धवर्ष)      |
| T    | 2017 (पहला अर्धवर्ष)      |
| V    | 2017 (दूसरा अर्धवर्ष)      |
| W    | 2018 (पहला अर्धवर्ष)      |
| X    | 2018 (दूसरा अर्धवर्ष)      |
| Y    | 2019 (पहला अर्धवर्ष)      |
| Z    | 2019 (दूसरा अर्धवर्ष)      |

### 1 - निर्माण का सप्ताह

पांचवा अक्षर उस सप्ताह को प्रतिष्ठानित करता है जिसमें उपकरण निर्मित हुआ था। इस स्थान पर 28 संभव वर्ण हैं: **1 से 9 तक के अंक पहले से नौवें सप्ताह को प्रतिष्ठानित करने के लिए उपयोग किए जाते हैं**, और **वर्ण C से Y तक**, **आवयव A, E, I, O और U और अक्षर S को छोड़कर**, **दसवें से सतावें सप्ताह को प्रतिष्ठानित करते हैं**। वर्ष के **दूसरे अर्धवर्ष में उत्पन्न उपकरणों के लिए, पांचवे अक्षर के द्वारा प्रतिष्ठानित संख्या में 26 जोड़ें**। उदाहरण के लिए, एक उत्पाद जिसका सीरियल नंबर का चौथा और प
### संदर्भ

{% embed url="https://beetstech.com/blog/decode-meaning-behind-apple-serial-number" %}

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT संग्रह**](https://opensea.io/collection/the-peass-family) देखें
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल हों** या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का** **अनुसरण** करें।**
* **अपने हैकिंग ट्रिक्स को** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में पीआर जमा करके अपना योगदान दें।**

</details>
