<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)

- प्राप्त करें [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके साझा करें।**

</details>


# CBC

यदि **कुकी** केवल **उपयोगकर्ता नाम** है (या कुकी का पहला हिस्सा उपयोगकर्ता नाम है) और आप उपयोगकर्ता नाम "**एडमिन**" का अनुकरण करना चाहते हैं। तो, आप उपयोगकर्ता नाम **"बीडमिन"** बना सकते हैं और कुकी के **पहले बाइट** को **ब्रूटफ़ोर्स** कर सकते हैं।

# CBC-MAC

क्रिप्टोग्राफी में, एक **साइफर ब्लॉक चेनिंग संदेश प्रमाणीकरण कोड** (**CBC-MAC**) एक तकनीक है जिसका उपयोग एक ब्लॉक साइफर से संदेश प्रमाणीकरण कोड बनाने के लिए किया जाता है। संदेश को सीबीसी मोड में किसी ब्लॉक साइफर एल्गोरिदम के साथ एन्क्रिप्ट किया जाता है ताकि प्रत्येक ब्लॉक पिछले ब्लॉक के सही एन्क्रिप्शन पर निर्भर हो। इस आपसी आश्रितता से सुनिश्चित होता है कि किसी भी प्लेनटेक्स्ट बिट के किसी भी परिवर्तन से अंतिम एन्क्रिप्टेड ब्लॉक में एक परिवर्तन होगा जिसे बिना ब्लॉक साइफर की कुंजी जाने के बिना पूर्वानुमान या विरोध नहीं किया जा सकता है।

संदेश m का CBC-MAC निर्णय करने के लिए, व्यक्ति एक शून्य प्रारंभिक वेक्टर के साथ सीबीसी मोड में m को एन्क्रिप्ट करता है और अंतिम ब्लॉक को रखता है। निम्नलिखित चित्र में एक गुप्त कुंजी k और ब्लॉक साइफर E का उपयोग करके ब्लॉकों![m\_{1}\\|m\_{2}\\|\cdots \\|m\_{x}](https://wikimedia.org/api/rest\_v1/media/math/render/svg/bbafe7330a5e40a04f01cc776c9d94fe914b17f5) से बने संदेश के CBC-MAC की गणना की गई है:

![CBC-MAC structure (en).svg](https://upload.wikimedia.org/wikipedia/commons/thumb/b/bf/CBC-MAC\_structure\_\(en\).svg/570px-CBC-MAC\_structure\_\(en\).svg.png)

# संकटग्रस्तता

CBC-MAC के साथ आमतौर पर **इस्तेमाल किया जाने वाला IV 0 होता है**।\
यह एक समस्या है क्योंकि 2 ज्ञात संदेश (`m1` और `m2`) स्वतंत्र रूप से 2 हस्ताक्षर (`s1` और `s2`) उत्पन्न करेंगे। इसलिए:

* `E(m1 XOR 0) = s1`
* `E(m2 XOR 0) = s2`

फिर m1 और m2 को जोड़कर बनाए गए संदेश (m3) 2 हस्ताक्षर (s31 और s32) उत्पन्न करेगा:

* `E(m1 XOR 0) = s31 = s1`
* `E(m2 XOR s1) = s32`

**जिसे एन्क्रिप्शन की कुंजी जाने के बिना गणना किया जा सकता है।**

सोचिए आप 8 बाइट ब्लॉक में नाम **व
- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**।**

- **अपने हैकिंग ट्रिक्स को [हैकट्रिक्स रेपो](https://github.com/carlospolop/hacktricks) और [हैकट्रिक्स-क्लाउड रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके साझा करें।**
