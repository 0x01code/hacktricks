<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग प्राप्त करें**](https://peass.creator-spring.com)
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) **का अनुसरण करें**.
* **अपनी हैकिंग ट्रिक्स साझा करें, HackTricks** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके.

</details>


# ECB

(ECB) इलेक्ट्रॉनिक कोड बुक - सममितीय एन्क्रिप्शन योजना जो **प्रत्येक ब्लॉक को स्पष्ट पाठ** के बदले में **सिफरटेक्स्ट के ब्लॉक से बदल देती है**. यह सबसे **सरल** एन्क्रिप्शन योजना है. मुख्य विचार यह है कि स्पष्ट पाठ को **N बिट्स के ब्लॉक्स में विभाजित करें** (इनपुट डेटा के ब्लॉक के आकार, एन्क्रिप्शन एल्गोरिदम पर निर्भर करता है) और फिर प्रत्येक स्पष्ट पाठ के ब्लॉक को एकमात्र कुंजी का उपयोग करके एन्क्रिप्ट (डिक्रिप्ट) करें.

![](https://upload.wikimedia.org/wikipedia/commons/thumb/e/e6/ECB_decryption.svg/601px-ECB_decryption.svg.png)

ECB का उपयोग करने से कई सुरक्षा प्रभाव पड़ते हैं:

* **एन्क्रिप्टेड संदेश से ब्लॉक्स को हटाया जा सकता है**
* **एन्क्रिप्टेड संदेश से ब्लॉक्स को इधर-उधर किया जा सकता है**

# कमजोरी का पता लगाना

कल्पना कीजिए कि आप कई बार एक एप्लिकेशन में लॉगिन करते हैं और आपको **हमेशा वही कुकी मिलती है**. यह इसलिए है क्योंकि एप्लिकेशन की कुकी **`<username>|<password>`** है.\
फिर, आप दो नए उपयोगकर्ता बनाते हैं, दोनों के पास **समान लंबा पासवर्ड** और **लगभग** **समान** **उपयोगकर्ता नाम** होता है.\
आप पाते हैं कि **8B के ब्लॉक्स** जहां **दोनों उपयोगकर्ताओं की जानकारी** समान है, वे **बराबर हैं**. फिर, आप कल्पना करते हैं कि यह शायद इसलिए है क्योंकि **ECB का उपयोग किया जा रहा है**.

निम्नलिखित उदाहरण की तरह. देखें कैसे ये **2 डिकोडेड कुकीज़** में कई बार ब्लॉक **`\x23U\xE45K\xCB\x21\xC8`** होता है.
```
\x23U\xE45K\xCB\x21\xC8\x23U\xE45K\xCB\x21\xC8\x04\xB6\xE1H\xD1\x1E \xB6\x23U\xE45K\xCB\x21\xC8\x23U\xE45K\xCB\x21\xC8+=\xD4F\xF7\x99\xD9\xA9

\x23U\xE45K\xCB\x21\xC8\x23U\xE45K\xCB\x21\xC8\x04\xB6\xE1H\xD1\x1E \xB6\x23U\xE45K\xCB\x21\xC8\x23U\xE45K\xCB\x21\xC8+=\xD4F\xF7\x99\xD9\xA9
```
यह इसलिए है क्योंकि **उन कुकीज़ के यूजरनेम और पासवर्ड में कई बार "a" अक्षर शामिल था** (उदाहरण के लिए)। जो **ब्लॉक्स** **अलग** हैं, वे ब्लॉक्स हैं जिनमें **कम से कम 1 अलग अक्षर** शामिल था (शायद डिलिमिटर "|" या यूजरनेम में कुछ जरूरी अंतर)।

अब, हमलावर को केवल यह पता लगाना है कि फॉर्मेट `<username><delimiter><password>` है या `<password><delimiter><username>`। इसके लिए, वह केवल **कई यूजरनेम बना सकता है** जिनमें **समान और लंबे यूजरनेम और पासवर्ड होते हैं जब तक कि वह फॉर्मेट और डिलिमिटर की लंबाई का पता न लगा ले:**

| Username length: | Password length: | Username+Password length: | Cookie's length (after decoding): |
| ---------------- | ---------------- | ------------------------- | --------------------------------- |
| 2                | 2                | 4                         | 8                                 |
| 3                | 3                | 6                         | 8                                 |
| 3                | 4                | 7                         | 8                                 |
| 4                | 4                | 8                         | 16                                |
| 7                | 7                | 14                        | 16                                |

# कमजोरी का शोषण

## पूरे ब्लॉक्स को हटाना

कुकी के फॉर्मेट को जानते हुए (`<username>|<password>`), यूजरनेम `admin` का प्रतिरूपण करने के लिए एक नया यूजर `aaaaaaaaadmin` बनाएं और कुकी प्राप्त करें और उसे डिकोड करें:
```
\x23U\xE45K\xCB\x21\xC8\xE0Vd8oE\x123\aO\x43T\x32\xD5U\xD4
```
हम पैटर्न `\x23U\xE45K\xCB\x21\xC8` देख सकते हैं जो पहले उस यूजरनेम के साथ बनाया गया था जिसमें केवल `a` था।\
फिर, आप पहले ब्लॉक के 8B को हटा सकते हैं और आपको यूजरनेम `admin` के लिए एक वैध कुकी मिल जाएगी:
```
\xE0Vd8oE\x123\aO\x43T\x32\xD5U\xD4
```
## ब्लॉक्स को स्थानांतरित करना

कई डेटाबेस में `WHERE username='admin';` के लिए खोजना या `WHERE username='admin    ';` के लिए खोजना समान होता है _(अतिरिक्त स्पेसेस का ध्यान दें)_

इसलिए, `admin` उपयोगकर्ता की नकल करने का एक और तरीका होगा:

* एक उपयोगकर्ता नाम उत्पन्न करें जिसके लिए: `len(<username>) + len(<delimiter) % len(block)`. अगर ब्लॉक का आकार `8B` है तो आप उपयोगकर्ता नाम बना सकते हैं: `username       `, और डिलिमिटर `|` के साथ `<username><delimiter>` चंक 8Bs के 2 ब्लॉक्स उत्पन्न करेगा।
* फिर, एक पासवर्ड उत्पन्न करें जो उस उपयोगकर्ता नाम को भरने के लिए एक सटीक संख्या में ब्लॉक्स को भरेगा जिसे हम नकल करना चाहते हैं और स्पेसेस, जैसे: `admin   `

इस उपयोगकर्ता का कुकी 3 ब्लॉक्स से बना होगा: पहले 2 ब्लॉक्स उपयोगकर्ता नाम + डिलिमिटर के हैं और तीसरा पासवर्ड का है (जो उपयोगकर्ता नाम की नकल कर रहा है): `username       |admin   `

** फिर, बस पहले ब्लॉक को आखिरी ब्लॉक के साथ बदल दें और आप `admin` उपयोगकर्ता की नकल कर रहे होंगे: `admin          |username`**

# संदर्भ

* [http://cryptowiki.net/index.php?title=Electronic_Code_Book\_(ECB)](http://cryptowiki.net/index.php?title=Electronic_Code_Book_\(ECB\))


<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert) के साथ AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स शेयर करें।

</details>
