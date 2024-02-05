# macOS डर्टी NIB

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी कंपनी का विज्ञापन **HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड** करना चाहते हैं तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** द्वारा PRs सबमिट करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos।

</details>

**तकनीक के बारे में अधिक विवरण के लिए मूल पोस्ट देखें: [https://blog.xpnsec.com/dirtynib/**](https://blog.xpnsec.com/dirtynib/)**। यहां एक सारांश है:

NIB फ़ाइलें, Apple के विकास पारिस्थितिकी का हिस्सा, एप्लिकेशन में **UI तत्वों** और उनके बीच के अंतर्क्रियाएँ परिभाषित करने के लिए हैं। वे विंडोज और बटन्स जैसे सिरीयलाइज़ड ऑब्जेक्ट्स को शामिल करते हैं, और रनटाइम पर लोड किए जाते हैं। इनके लगातार उपयोग के बावजूद, Apple अब अधिक समग्र UI फ्लो विज़ुअलाइज़ेशन के लिए स्टोरीबोर्ड का समर्थन करता है।

### NIB फ़ाइलों के साथ सुरक्षा संबंधित चिंताएँ
**NIB फ़ाइलें सुरक्षा जोखिम** हो सकती हैं। उनमें **विचारहीन कमांड्स को निष्पादित** करने की क्षमता होती है, और एप्लिकेशन में NIB फ़ाइलों में परिवर्तन Gatekeeper को एप्लिकेशन को निष्पादित करने से नहीं रोकते, जो एक महत्वपूर्ण धमाका है।

### डर्टी NIB इंजेक्शन प्रक्रिया
#### NIB फ़ाइल बनाना और सेटअप करना
1. **प्रारंभिक सेटअप**:
- XCode का उपयोग करके एक नई NIB फ़ाइल बनाएं।
- इंटरफेस में एक ऑब्ज
