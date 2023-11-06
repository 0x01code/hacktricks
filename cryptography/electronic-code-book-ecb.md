<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks क्लाउड ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 ट्विटर 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ ट्विच 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 यूट्यूब 🎥</strong></a></summary>

- क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह।

- प्राप्त करें [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com)।

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**।**

- **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके साझा करें।**

</details>


# ECB

(ECB) इलेक्ट्रॉनिक कोड बुक - सममित्र एन्क्रिप्शन योजना जो **प्रत्येक ब्लॉक के स्पष्ट पाठ को** **साइपर पाठ के ब्लॉक से** **बदल देती है**। यह सबसे **सरल** एन्क्रिप्शन योजना है। मुख्य विचार यह है कि स्पष्ट पाठ को **N बिट के ब्लॉकों में विभाजित** (इनपुट डेटा के ब्लॉक का आकार, एन्क्रिप्शन एल्गोरिदम पर निर्भर करता है) और फिर केवल कुंजी का उपयोग करके प्रत्येक स्पष्ट पाठ के ब्लॉक को एन्क्रिप्ट (डिक्रिप्ट) करें।

![](https://upload.wikimedia.org/wikipedia/commons/thumb/e/e6/ECB_decryption.svg/601px-ECB_decryption.svg.png)

ECB का उपयोग करने से कई सुरक्षा संबंधित प्रभाव हो सकते हैं:

* **एन्क्रिप्टेड संदेश के ब्लॉक हटा सकते हैं**
* **एन्क्रिप्टेड संदेश के ब्लॉक को चला सकते हैं**

# संकट की पहचान

सोचिए आप कई बार एक एप्लिकेशन में लॉगिन करते हैं और आप **हमेशा एक ही कुकी प्राप्त** करते हैं। इसलिए, एप्लिकेशन की कुकी **`<उपयोगकर्ता नाम>|<पासवर्ड>`** होती है।\
फिर, आप दो नए उपयोगकर्ताओं को बनाते हैं, जिनमें से दोनों का **एक ही लंबा पासवर्ड** होता है और **लगभग** **एक ही** **उपयोगकर्ता नाम** होता है।\
आपको पता चलता है कि **8B के ब्लॉक** जहां **दोनों उपयोगकर्ताओं की जानकारी** समान होती है, **बराबर** होते हैं। तब आपको यह लगता है कि इसका कारण **ECB का उपयोग हो रहा है**।

निम्नलिखित उदाहरण में देखें। ध्यान दें कि इन **2 डिकोडेड कुकीज़** में कई बार ब्लॉक **`\x23U\xE45K\xCB\x21\xC8`** होता है।
```
\x23U\xE45K\xCB\x21\xC8\x23U\xE45K\xCB\x21\xC8\x04\xB6\xE1H\xD1\x1E \xB6\x23U\xE45K\xCB\x21\xC8\x23U\xE45K\xCB\x21\xC8+=\xD4F\xF7\x99\xD9\xA9

\x23U\xE45K\xCB\x21\xC8\x23U\xE45K\xCB\x21\xC8\x04\xB6\xE1H\xD1\x1E \xB6\x23U\xE45K\xCB\x21\xC8\x23U\xE45K\xCB\x21\xC8+=\xD4F\xF7\x99\xD9\xA9
```
यह इसलिए है क्योंकि उन कुकीज़ के **उपयोगकर्ता नाम और पासवर्ड में कई बार अक्षर "a"** शामिल थे (उदाहरण के लिए)। **अलग** ब्लॉक वे हैं जिनमें **कम से कम 1 अलग अक्षर** था (शायद विभाजक "|" या उपयोगकर्ता नाम में कुछ आवश्यक अंतर हो सकता है)।

अब, हमलावर को बस यह खोजना होगा कि प्रारूप `<उपयोगकर्ता नाम><विभाजक><पासवर्ड>` है या `<पासवर्ड><विभाजक><उपयोगकर्ता नाम>` है। इसके लिए, वह बस **कई उपयोगकर्ता नाम** उत्पन्न कर सकता है **जिनमें समान और लंबे उपयोगकर्ता नाम और पासवर्ड** हों ताकि वह प्रारूप और विभाजक की लंबाई खोज सके:

| उपयोगकर्ता नाम की लंबाई: | पासवर्ड की लंबाई: | उपयोगकर्ता नाम + पासवर्ड की लंबाई: | कुकी की लंबाई (डिकोडिंग के बाद): |
| ---------------- | ---------------- | ------------------------- | --------------------------------- |
| 2                | 2                | 4                         | 8                                 |
| 3                | 3                | 6                         | 8                                 |
| 3                | 4                | 7                         | 8                                 |
| 4                | 4                | 8                         | 16                                |
| 7                | 7                | 14                        | 16                                |

# सुरक्षा की कमजोरी का उपयोग

## पूरे ब्लॉक हटाना

कुकी के प्रारूप (`<उपयोगकर्ता नाम>|<पासवर्ड>`) को जानते हुए, उपयोगकर्ता नाम `admin` की अनुकरण करने के लिए `aaaaaaaaadmin` नामक एक नया उपयोगकर्ता बनाएं और कुकी प्राप्त करें और इसे डिकोड करें:
```
\x23U\xE45K\xCB\x21\xC8\xE0Vd8oE\x123\aO\x43T\x32\xD5U\xD4
```
हम पहले से बनाए गए पैटर्न `\x23U\xE45K\xCB\x21\xC8` को देख सकते हैं जो केवल `a` से मिलता है।
फिर, आप पहले 8B के ब्लॉक को हटा सकते हैं और आपको उपयोगकर्ता नाम `admin` के लिए एक मान्य कुकी मिलेगी:
```
\xE0Vd8oE\x123\aO\x43T\x32\xD5U\xD4
```
## ब्लॉकों को हटाएं

बहुत सारे डेटाबेस में `WHERE username='admin';` या `WHERE username='admin    ';` की खोज करने में कोई अंतर नहीं होता है। _(अतिरिक्त जगहों का ध्यान दें)_

इसलिए, उपयोगकर्ता `admin` की अनुकरण करने का एक और तरीका है:

* एक उपयोगकर्ता नाम उत्पन्न करें जो इस शर्त को पूरा करता है: `len(<username>) + len(<delimiter) % len(block)`. `8B` ब्लॉक आकार के साथ आप `username       ` नामक उपयोगकर्ता उत्पन्न कर सकते हैं, जबकि विभाजक `|` होगा, चंक `<username><delimiter>` 2 ब्लॉक्स को 8B का उत्पन्न करेगा।
* फिर, हमें अनुकरण करने के लिए उपयोगकर्ता नाम और अंतरिक्ष को भरने वाला पासवर्ड उत्पन्न करें, जैसे: `admin   `

इस उपयोगकर्ता की कुकी 3 ब्लॉकों से मिलकर बनी होगी: पहले 2 ब्लॉक उपयोगकर्ता नाम + विभाजक के ब्लॉक होंगे और तीसरा ब्लॉक पासवर्ड का होगा (जो उपयोगकर्ता नाम का अनुकरण कर रहा है): `username       |admin   `

** फिर, बस पहले ब्लॉक को आखिरी ब्लॉक के साथ बदल दें और हम उपयोगकर्ता `admin` का अनुकरण कर रहे होंगे: `admin          |username`**

# संदर्भ

* [http://cryptowiki.net/index.php?title=Electronic_Code_Book\_(ECB)](http://cryptowiki.net/index.php?title=Electronic_Code_Book_\(ECB\))


<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आप **PEASS के नवीनतम संस्करण का उपयोग करना चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह।

- प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपने हैकिंग ट्रिक्स साझा करें, [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके।**

</details>
