# macOS बंडल्स

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी कंपनी का विज्ञापन **HackTricks** में देखना चाहते हैं या **HackTricks को PDF में डाउनलोड** करना चाहते हैं तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) का खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos।

</details>

## मैकओएस बंडल्स

मैकओएस में बंडल्स एप्लिकेशन, लाइब्रेरी और अन्य आवश्यक फ़ाइलों के लिए एक कंटेनर के रूप में काम करते हैं, जिन्हें फाइंडर में एकल वस्तु के रूप में प्रदर्शित किया जाता है, जैसे परिचित `*.app` फ़ाइलें। सबसे आम बंडल `.app` बंडल है, हालांकि `.framework`, `.systemextension`, और `.kext` जैसे अन्य प्रकार भी प्रसारित हैं।

### बंडल के महत्वपूर्ण घटक

एक बंडल के भीतर, विशेष रूप से `<application>.app/Contents/` निर्देशिका के भीतर, कई महत्वपूर्ण संसाधन संग्रहीत होते हैं:

* **\_CodeSignature**: इस निर्देशिका में एप्लिकेशन की अविश्वसनीयता की जांच के लिए महत्वपूर्ण कोड-साइनिंग विवरण संग्रहीत है। आप ऑपनस्ल डीजीएसटी -बाइनरी -शा1 /एप्लिकेशन/सफारी.एप्प/कंटेंट्स/रिसोर्सेज/एसेट्स.कार | ऑपनस्ल बेस64 %%% के जैसे कमांड का उपयोग करके कोड-साइनिंग जानकारी की जांच कर सकते हैं।
* **MacOS**: एप्लिकेशन का कार्यात्मक बाइनरी संग्रहीत है जो उपयोगकर्ता संवाद पर चलता है।
* **Resources**: एप्लिकेशन के उपयोगकर्ता इंटरफ़ेस संघटनों के लिए एक भंडार है जिसमें छवियाँ, दस्तावेज़ और इंटरफ़ेस विवरण (निब/xib फ़ाइलें) शामिल हैं।
* **Info.plist**: एप्लिकेशन का मुख्य कॉन्फ़िगरेशन फ़ाइल के रूप में काम करता है, जिसे प्रणाली को एप्लिकेशन को सही ढंग से पहचानने और संवाद करने के लिए महत्वपूर्ण माना जाता है।

#### Info.plist में महत्वपूर्ण कुंजी

`Info.plist` फ़ाइल एप्लिकेशन कॉन्फ़िगरेशन के लिए एक मूलभूत है, जिसमें CFBundleExecutable जैसी कुंजियाँ शामिल हैं:

* **CFBundleExecutable**: `Contents/MacOS` निर्देशिका में स्थित मुख्य कार्यात्मक फ़ाइल का नाम निर्दिष्ट करता है।
* **CFBundleIdentifier**: एप्लिकेशन के लिए एक वैश्विक पहचानकर्ता प्रदान करता है, जिसे मैकओएस द्वारा एप्लिकेशन प्रबंधन के लिए व्यापक रूप से उपयोग किया जाता है।
* **LSMinimumSystemVersion**: एप्लिकेशन को चलाने के लिए आवश्यक मैकओएस का न्यूनतम संस्करण दर्शाता है।

### बंडल की खोज

`Safari.app` जैसे बंडल की सामग्री की खोज करने के लिए निम्नलिखित कमांड का उपयोग किया जा सकता है: `bash ls -lR /Applications/Safari.app/Contents`

यह खोज डायरेक्टरी जैसे `_CodeSignature`, `MacOS`, `Resources`, और फ़ाइलें जैसे `Info.plist` को प्रकट करती है, प्रत्येक एक अनुपात से एप्लिकेशन की सुरक्षा से लेकर उपयोगकर्ता इंटरफ़ेस और परिचालन पैरामीटर की परिभाषा तक के लिए एक अद्वितीय उद्देश्य सेवा करती है।

#### अतिरिक्त बंडल डायरेक्टरियाँ

सामान्य डायरेक्टरियों के अलावा, बंडल्स में निम्नलिखित भी शामिल हो सकते हैं:

* **Frameworks**: एप्लिकेशन द्वारा उपयोग किए जाने वाले बंडल फ्रेमवर्क संग्रहीत है। फ्रेमवर्क डायनामिक लाइब्स की तरह होते हैं जिनमें अतिरिक्त संसाधन होते हैं।
* **PlugIns**: एप्लिकेशन की क्षमताएँ बढ़ाने वाले प्लगइन और एक्सटेंशन के लिए एक निर्देशिका।
* **XPCServices**: एप्लिकेशन द्वारा प्रक्रिया के बाहर संचार के लिए उपयोग किए जाने वाले XPC सेवाएँ रखता है।

यह संरचना सुनिश्चित करती है कि सभी आवश्यक घटक बंडल के भीतर समाहित हों, जो एक मॉड्यूलर और सुरक्षित एप्लिकेशन वातावरण को सुविधाजनक बनाती है।

अधिक विस्तृत जानकारी के लिए `Info.plist` कुंजियों और उनके अर्थों पर, Apple डेवलपर दस्तावेज़ीकरण व्यापक संसाधन प्रदान करता है: [Apple Info.plist Key Reference](https://developer.apple.com/library/archive/documentation/General/Reference/InfoPlistKeyReference/Introduction/Introduction.html).

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी कंपनी का विज्ञापन **HackTricks** में देखना चाहते हैं या **HackTricks को PDF में डाउनलोड** करना चाहते हैं तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) का खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos।

</details>
