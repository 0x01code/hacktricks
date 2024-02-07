# macOS बंडल्स

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी कंपनी का विज्ञापन **HackTricks** में देखना चाहते हैं या **HackTricks को PDF में डाउनलोड** करना चाहते हैं तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** ट्विटर पर **फॉलो** करें 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

## मैकओएस बंडल्स

मैकओएस में बंडल्स एप्लिकेशन, लाइब्रेरीज़, और अन्य आवश्यक फ़ाइलों को समेत करने वाले संसाधनों के एक संग्रह के रूप में काम करते हैं, जिन्हें फाइंडर में एकल वस्तुओं के रूप में प्रदर्शित किया जाता है, जैसे परिचित `*.app` फ़ाइलें। सबसे आम बंडल `.app` बंडल है, हालांकि अन्य प्रकार जैसे `.framework`, `.systemextension`, और `.kext` भी प्रसारित हैं।

### बंडल के महत्वपूर्ण घटक

बंडल के भीतर, विशेष रूप से `<application>.app/Contents/` निर्देशिका के भीतर, कई महत्वपूर्ण संसाधन संग्रहित होते हैं:

- **_CodeSignature**: इस निर्देशिका में कोड-साइनिंग विवरण संग्रहित होते हैं जो एप्लिकेशन की अखंडता की पुष्टि के लिए आवश्यक होते हैं। आप इस्तेमाल करके कोड-साइनिंग जानकारी की जांच कर सकते हैं:
%%%bash
openssl dgst -binary -sha1 /Applications/Safari.app/Contents/Resources/Assets.car | openssl base64
%%%
- **MacOS**: एप्लिकेशन के एक्सीक्यूटेबल बाइनरी को संचालित करने वाला निर्देशिका।
- **Resources**: एप्लिकेशन के उपयोगकर्ता इंटरफ़ेस संघटन के लिए एक भंडार है जिसमें छवियाँ, दस्तावेज़, और इंटरफ़ेस विवरण (nib/xib फ़ाइलें) शामिल हैं।
- **Info.plist**: एप्लिकेशन का मुख्य कॉन्फ़िगरेशन फ़ाइल के रूप में काम करता है, जिसे सिस्टम को एप्लिकेशन को सही ढंग से पहचानने और इंटरैक्ट करने के लिए महत्वपूर्ण माना जाता है।

#### Info.plist में महत्वपूर्ण कुंजी

`Info.plist` फ़ाइल एप्लिकेशन कॉन्फ़िगरेशन के लिए एक मूलभूत है, जिसमें CFBundleExecutable, CFBundleIdentifier, LSMinimumSystemVersion जैसी कुंजियाँ शामिल हैं।

### बंडल की खोज

`Safari.app` जैसे बंडल की सामग्री की खोज करने के लिए निम्नलिखित कमांड का उपयोग किया जा सकता है:
%%%bash
ls -lR /Applications/Safari.app/Contents
%%%

यह खोज डायरेक्टरी जैसे `_CodeSignature`, `MacOS`, `Resources`, और फ़ाइलें जैसे `Info.plist` को प्रकट करती है, जो प्रत्येक एक अलग उद्देश्य से सुरक्षित एप्लिकेशन से लेकर उसके उपयोगकर्ता इंटरफ़ेस और परिचालन पैरामीटरों को परिभाषित करने में मदद करती है।

#### अतिरिक्त बंडल डायरेक्टरी

सामान्य डायरेक्टरियों के अलावा, बंडल्स में निम्नलिखित भी शामिल हो सकते हैं:

- **Frameworks**: एप्लिकेशन द्वारा उपयोग किए जाने वाले बंडल फ़्रेमवर्क्स को संग्रहित करता है।
- **PlugIns**: एक डायरेक्टरी जो एप्लिकेशन की क्षमताएँ बढ़ाने वाले प्लगइन और एक्सटेंशन के लिए है।
- **XPCServices**: एप्लिकेशन द्वारा प्रक्रिया के बाहर संचार के लिए उपयोग किए जाने वाले XPC सेवाएँ को धारित करता है।

यह संरचना सुनिश्चित करती है कि सभी आवश्यक घटक बंडल के भीतर समाहित हों, जो एक मॉड्यूलर और सुरक्षित एप्लिकेशन वातावरण को सुविधाजनक बनाती है।

अधिक विस्तृत जानकारी के लिए `Info.plist` कुंजियों और उनके अर्थों पर, Apple डेवलपर दस्तावेज़ीकरण व्यापक संसाधन प्रदान करता है: [Apple Info.plist Key Reference](https://developer.apple.com/library/archive/documentation/General/Reference/InfoPlistKeyReference/Introduction/Introduction.html).

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी कंपनी का विज्ञापन **HackTricks** में देखना चाहते हैं या **HackTricks को PDF में डाउनलोड** करना चाहते हैं तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** ट्विटर पर **फॉलो** करें 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>
