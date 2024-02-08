# macOS सीरियल नंबर

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert)</strong> के साथ जीरो से हीरो तक AWS हैकिंग सीखें!</summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप अपनी कंपनी का विज्ञापन HackTricks में देखना चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live) पर **फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** द्वारा PRs सबमिट करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में।

</details>


## मूल जानकारी

2010 के बाद के Apple डिवाइसों के सीरियल नंबर 12 अल्फान्यूमेरिक वर्णों से बने होते हैं, प्रत्येक सेगमेंट विशिष्ट जानकारी को सूचित करता है:

- **पहले 3 वर्ण**: **निर्माण स्थान** को इंडिकेट करते हैं।
- **4 और 5 वर्ण**: **निर्माण के साल और सप्ताह** को दर्शाते हैं।
- **6 से 8 वर्ण**: प्रत्येक डिवाइस के लिए **एक अद्वितीय पहचानकर्ता** के रूप में काम करते हैं।
- **अंतिम 4 वर्ण**: डिवाइस का **मॉडल नंबर** निर्धारित करते हैं।

उदाहरण के लिए, सीरियल नंबर **C02L13ECF8J2** इस संरचना का पालन करता है।

### **निर्माण स्थान (पहले 3 वर्ण)**
कुछ कोड विशिष्ट कारखानों को प्रतिनिधित करते हैं:
- **FC, F, XA/XB/QP/G8**: संयुक्त राज्य अमेरिका में विभिन्न स्थान।
- **RN**: मेक्सिको।
- **CK**: कॉर्क, आयरलैंड।
- **VM**: फॉक्सकॉन, चेक गणराज्य।
- **SG/E**: सिंगापुर।
- **MB**: मलेशिया।
- **PT/CY**: कोरिया।
- **EE/QT/UV**: ताइवान।
- **FK/F1/F2, W8, DL/DM, DN, YM/7J, 1C/4H/WQ/F7**: चीन के विभिन्न स्थान।
- **C0, C3, C7**: चीन के विशिष्ट शहर।
- **RM**: नवीनीकृत डिवाइस।

### **निर्माण का साल (4 वर्ण)**
यह वर्ण 'C' (2010 के पहले अर्धवर्ष को प्रतिनिधित करता है) से 'Z' (2019 के दूसरे अर्धवर्ष को) तक विभिन्न अक्षरों के साथ विभिन्न अर्धवर्ष की अवधियों को दर्शाता है।

### **निर्माण का सप्ताह (5 वर्ण)**
अंक 1-9 सप्ताह 1-9 का प्रतिनिधित करते हैं। अक्षर C-Y (स्वरों और 'S' को छोड़कर) सप्ताह 10-27 को प्रतिनिधित करते हैं। वर्ष के दूसरे अर्धवर्ष के लिए, इस संख्या में 26 जोड़ी जाती है।

### **अद्वितीय पहचानकर्ता (वर्ण 6 से 8)**
ये तीन अंक सुनिश्चित करते हैं कि प्रत्येक डिवाइस, यदि वह समान मॉडल और बैच का हो, एक विशिष्ट सीरियल नंबर है।

### **मॉडल नंबर (अंतिम 4 वर्ण)**
ये अंक डिवाइस का विशिष्ट मॉडल पहचानते हैं।

### संदर्भ

* [https://beetstech.com/blog/decode-meaning-behind-apple-serial-number](https://beetstech.com/blog/decode-meaning-behind-apple-serial-number)

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert)</strong> के साथ जीरो से हीरो तक AWS हैकिंग सीखें!</summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप अपनी कंपनी का विज्ञापन HackTricks में देखना चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live) पर **फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** द्वारा PRs सबमिट करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में।

</details>
