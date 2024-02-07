# macOS सीरियल नंबर

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** द्वारा PRs सबमिट करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>


## मूल जानकारी

2010 के बाद के Apple उपकरणों के सीरियल नंबर में **12 अक्षरों का अक्षरात्मक संख्या** होता है, प्रत्येक सेगमेंट विशिष्ट जानकारी प्रस्तुत करता है:

- **पहले 3 अक्षर**: **निर्माण स्थान** को दर्शाते हैं।
- **अक्षर 4 और 5**: **निर्माण के वर्ष और सप्ताह** को दर्शाते हैं।
- **अक्षर 6 से 8**: प्रत्येक उपकरण के लिए **एक अद्वितीय पहचानकर्ता** के रूप में काम करते हैं।
- **अंतिम 4 अक्षर**: उपकरण का **मॉडल नंबर** निर्दिष्ट करते हैं।

उदाहरण के लिए, सीरियल नंबर **C02L13ECF8J2** इस संरचना का पालन करता है।

### **निर्माण स्थान (पहले 3 अक्षर)**
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
- **RM**: पुनर्निर्मित उपकरण।

### **निर्माण का वर्ष (4 वां अक्षर)**
यह अक्षर 'C' (2010 के पहले अर्धवर्ष को प्रतिनिधित करता है) से 'Z' (2019 के दूसरे अर्धवर्ष को) तक विभिन्न अक्षरों के साथ विभिन्न अर्धवर्ष की अवधियों को दर्शाता है।

### **निर्माण का सप्ताह (5 वां अक्षर)**
अंक 1-9 सप्ताह 1-9 का प्रतिनिधित करते हैं। अक्षर C-Y (स्वरों और 'S' को छोड़कर) सप्ताह 10-27 को प्रतिनिधित करते हैं। वर्ष के दूसरे अर्धवर्ष के लिए, इस संख्या में 26 जोड़ा जाता है।

### **अद्वितीय पहचानकर्ता (अक्षर 6 से 8)**
ये तीन अंक सुनिश्चित करते हैं कि प्रत्येक उपकरण, यदि वही मॉडल और बैच हो, एक विशिष्ट सीरियल नंबर है।

### **मॉडल नंबर (अंतिम 4 अक्षर)**
ये अंक उपकरण का विशिष्ट मॉडल पहचानते हैं।

### संदर्भ

* [https://beetstech.com/blog/decode-meaning-behind-apple-serial-number](https://beetstech.com/blog/decode-meaning-behind-apple-serial-number)

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** द्वारा PRs सबमिट करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
