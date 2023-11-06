# macOS सिस्टम एक्सटेंशन्स

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks क्लाउड ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 ट्विटर 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ ट्विच 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 यूट्यूब 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **हैकट्रिक्स में विज्ञापित करना** चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने की अनुमति** चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एकस्क्लूसिव [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल हों** या मुझे **ट्विटर** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का** **अनुसरण करें**.
* **हैकिंग ट्रिक्स साझा करें** हैकट्रिक्स रेपो (https://github.com/carlospolop/hacktricks) और हैकट्रिक्स-क्लाउड रेपो (https://github.com/carlospolop/hacktricks-cloud) में पीआर सबमिट करके।

</details>

## सिस्टम एक्सटेंशन्स / एंडपॉइंट सुरक्षा फ्रेमवर्क

कर्नल एक्सटेंशन के विपरीत, **सिस्टम एक्सटेंशन्स कर्नल स्थान के बजाय उपयोगकर्ता स्थान में चलते हैं**, जिससे एक्सटेंशन खराबी के कारण सिस्टम क्रैश का खतरा कम होता है।

<figure><img src="../../../.gitbook/assets/image (1) (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

सिस्टम एक्सटेंशन के तीन प्रकार होते हैं: **ड्राइवरकिट** एक्सटेंशन, **नेटवर्क** एक्सटेंशन और **एंडपॉइंट सुरक्षा** एक्सटेंशन।

### **ड्राइवरकिट एक्सटेंशन्स**

ड्राइवरकिट कर्नल एक्सटेंशन के स्थानांतरण के लिए हैंडलर सप्लाई करते हैं। यह उपकरण ड्राइवर (जैसे USB, सीरियल, NIC और HID ड्राइवर) को कर्नल स्थान के बजाय उपयोगकर्ता स्थान में चलाने की अनुमति देता है। ड्राइवरकिट फ्रेमवर्क में कुछ I/O किट कक्षाओं के उपयोगकर्ता स्थान संस्करण शामिल होते हैं, और कर्नल नियमित I/O किट घटनाओं को उपयोगकर्ता स्थान के लिए आगे भेजता है, जिससे इन ड्राइवरों को चलाने के लिए एक सुरक्षित माहौल प्रदान किया जाता है।

### **नेटवर्क एक्सटेंशन्स**

नेटवर्क एक्सटेंशन्स नेटवर्क व्यवहार को अनुकूलित करने की क्षमता प्रदान करते हैं। कई प्रकार के नेटवर्क एक्सटेंशन्स होते हैं:

* **ऐप प्रॉक्सी**: इसका उपयोग एक फ्लो-आधारित, कस्टम वीपीएन प्रोटोकॉल को लागू करने वाले वीपीएन क्लाइंट को बनाने के लिए किया जाता है। इसका मतलब है कि यह नेटवर्क ट्रैफिक को व्यक्तिगत पैकेटों के बजाय कनेक्शन (या फ्लो) के आधार पर हैंडल करता है।
* **पैकेट टनल**: इसका उपयोग एक पैकेट-आधारित, कस्टम वीपीएन प्रोटोकॉल को लागू करने वाले वीपीएन क्लाइंट को बनाने के लिए किया जाता है। इसका मतलब है कि यह नेटवर्क ट्रैफिक को व्यक्तिगत पैकेटों के आधार पर हैंडल करता है।
* **फ़िल्टर डेटा**: इसका उपयोग नेटवर्क "फ्लो" को फ़िल्टर करने के लिए किया जाता है। यह फ्लो स्तर पर नेटवर्क डेटा का मॉनिटर या संशोधन कर सकता है।
* **फ़िल्टर पैकेट**: इसका उपयोग व्यक्तिगत नेटव
## ESF को उमारवाव

ESF का उपयोग सुरक्षा उपकरणों द्वारा किया जाता है जो एक रेड टीमर का पता लगाने का प्रयास करेंगे, इसलिए ऐसी कोई भी जानकारी जो इसे टाल सकती हो, दिखाई देती है।

### CVE-2021-30965

बात यह है कि सुरक्षा एप्लिकेशन को **पूर्ण डिस्क पहुंच अनुमतियाँ** होनी चाहिए। इसलिए यदि कोई हमलावर उसे हटा सकता है, तो वह सॉफ़्टवेयर को चलाने से रोक सकता है:
```bash
tccutil reset All
```
इस बाइपास और संबंधित बाइपास के बारे में **अधिक जानकारी** के लिए टॉक देखें [#OBTS v5.0: "The Achilles Heel of EndpointSecurity" - Fitzl Csaba](https://www.youtube.com/watch?v=lQO7tvNCoTI)

अंत में, इसे ठीक कर दिया गया था जब **`tccd`** द्वारा प्रबंधित सुरक्षा ऐप को नई अनुमति **`kTCCServiceEndpointSecurityClient`** दी गई थी, इससे `tccutil` इसकी अनुमतियों को साफ़ करने से रोक देता है जो इसे चलाने से रोकता है।

## संदर्भ

* [**OBTS v3.0: "Endpoint Security & Insecurity" - Scott Knight**](https://www.youtube.com/watch?v=jaVkpM1UqOs)
* [**https://knight.sc/reverse%20engineering/2019/08/24/system-extension-internals.html**](https://knight.sc/reverse%20engineering/2019/08/24/system-extension-internals.html)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने की अनुमति** चाहिए? [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का पालन करें।**
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को।**

</details>
