# macOS बंडल्स

<details>

<summary><strong> AWS हैकिंग सीखें शून्य से लेकर हीरो तक </strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग प्राप्त करें**](https://peass.creator-spring.com)
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) को **फॉलो करें**.
* **अपनी हैकिंग ट्रिक्स साझा करें, HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके.

</details>

## मूल जानकारी

मूल रूप से, एक बंडल फाइल सिस्टम के भीतर एक **डायरेक्टरी संरचना** है। दिलचस्प बात यह है कि डिफ़ॉल्ट रूप से यह डायरेक्टरी **Finder में एकल ऑब्जेक्ट की तरह दिखती है**.&#x20;

हम जिस **सामान्य** बंडल से अक्सर मिलेंगे वह है **`.app` बंडल**, लेकिन कई अन्य एक्जीक्यूटेबल्स भी बंडल्स के रूप में पैक किए जाते हैं, जैसे कि **`.framework`** और **`.systemextension`** या **`.kext`**.

एक बंडल के भीतर समाहित संसाधनों के प्रकार में एप्लिकेशन, लाइब्रेरीज, इमेजेज, डॉक्यूमेंटेशन, हेडर फाइल्स, आदि शामिल हो सकते हैं। ये सभी फाइलें `<application>.app/Contents/` के अंदर होती हैं।
```bash
ls -lR /Applications/Safari.app/Contents
```
* `Contents/_CodeSignature` -> एप्लिकेशन के **कोड-साइनिंग जानकारी** शामिल होती है (जैसे कि हैशेज आदि)।
* `openssl dgst -binary -sha1 /Applications/Safari.app/Contents/Resources/Assets.car | openssl base64`
* `Contents/MacOS` -> **एप्लिकेशन का बाइनरी** शामिल होता है (जो यूजर द्वारा एप्लिकेशन आइकन पर डबल-क्लिक करने पर निष्पादित होता है)।
* `Contents/Resources` -> **एप्लिकेशन के UI तत्व** शामिल होते हैं, जैसे कि इमेजेज, दस्तावेज़, और nib/xib फाइलें (जो विभिन्न यूजर इंटरफेस का वर्णन करती हैं)।
* `Contents/Info.plist` -> एप्लिकेशन की मुख्य "**कॉन्फ़िगरेशन फ़ाइल।**" Apple का कहना है कि "सिस्टम इस फ़ाइल की उपस्थिति पर निर्भर करता है ताकि \[एप्लिकेशन] और किसी भी संबंधित फ़ाइलों के बारे में प्रासंगिक जानकारी की पहचान कर सके"।
* **Plist** **फ़ाइलें** कॉन्फ़िगरेशन जानकारी शामिल करती हैं। आप plist कीज़ के अर्थ के बारे में जानकारी [https://developer.apple.com/library/archive/documentation/General/Reference/InfoPlistKeyReference/Introduction/Introduction.html](https://developer.apple.com/library/archive/documentation/General/Reference/InfoPlistKeyReference/Introduction/Introduction.html) में पा सकते हैं।
*   एप्लिकेशन का विश्लेषण करते समय रुचि के जोड़े हो सकते हैं:\\

* **CFBundleExecutable**

**एप्लिकेशन के बाइनरी का नाम** शामिल करता है (Contents/MacOS में पाया जाता है)।

* **CFBundleIdentifier**

एप्लिकेशन के बंडल आइडेंटिफ़ायर को शामिल करता है (अक्सर सिस्टम द्वारा एप्लिकेशन की **वैश्विक** **पहचान** करने के लिए इस्तेमाल किया जाता है)।

* **LSMinimumSystemVersion**

**macOS के सबसे पुराने** **संस्करण** को शामिल करता है जिसके साथ एप्लिकेशन संगत है।

<details>

<summary><strong>Learn AWS hacking from zero to hero with</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Other ways to support HackTricks:

* If you want to see your **company advertised in HackTricks** or **download HackTricks in PDF** Check the [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Get the [**official PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Discover [**The PEASS Family**](https://opensea.io/collection/the-peass-family), our collection of exclusive [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Join the** 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) or the [**telegram group**](https://t.me/peass) or **follow** me on **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **Share your hacking tricks by submitting PRs to the** [**HackTricks**](https://github.com/carlospolop/hacktricks) and [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
