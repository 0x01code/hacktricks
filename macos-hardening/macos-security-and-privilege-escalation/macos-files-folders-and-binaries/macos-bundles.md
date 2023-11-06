# macOS बंडल

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल हों** या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **हैकिंग ट्रिक्स साझा करें और PRs सबमिट करें** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को**.

</details>

## मूलभूत जानकारी

मूल रूप से, एक बंडल फ़ाइल सिस्टम में एक **निर्देशिका संरचना** है। दिलचस्प बात यह है कि, डिफ़ॉल्ट रूप से इस निर्देशिका में **फ़ाइंडर में एकल वस्तु की तरह दिखता है**।&#x20;

हमें आमतौर पर मिलने वाला बंडल **`.app` बंडल** है, लेकिन कई अन्य एक्जीक्यूटेबल भी बंडल के रूप में पैकेज किए जाते हैं, जैसे **`.framework`** और **`.systemextension`** या **`.kext`**।

बंडल में शामिल संसाधनों के प्रकार में एप्लिकेशन, पुस्तकालय, छवियाँ, दस्तावेज़ीकरण, हैडर फ़ाइलें आदि शामिल हो सकती हैं। सभी ये फ़ाइलें `<एप्लिकेशन>.app/Contents/` में होती हैं।
```bash
ls -lR /Applications/Safari.app/Contents
```
* `Contents/_CodeSignature` -> ऐप्लिकेशन के बारे में **कोड-साइनिंग जानकारी** को संग्रहीत करता है (जैसे हैश, आदि)।
* `openssl dgst -binary -sha1 /Applications/Safari.app/Contents/Resources/Assets.car | openssl base64`
* `Contents/MacOS` -> ऐप्लिकेशन का **बाइनरी** संग्रहीत करता है (जो उपयोगकर्ता द्वारा यूआई में ऐप्लिकेशन आइकन पर डबल-क्लिक करने पर निष्पादित होता है)।
* `Contents/Resources` -> ऐप्लिकेशन के **यूआई तत्व** को संग्रहीत करता है, जैसे छवियाँ, दस्तावेज़ और निब/xib फ़ाइलें (जो विभिन्न उपयोगकर्ता इंटरफ़ेस का वर्णन करती हैं)।
* `Contents/Info.plist` -> ऐप्लिकेशन की मुख्य "**कॉन्फ़िगरेशन फ़ाइल**"। Apple नोट करता है कि "सिस्टम इस फ़ाइल की मौजूदगी पर निर्भर करता है ताकि ऐप्लिकेशन और संबंधित फ़ाइलों के बारे में प्रासंगिक जानकारी प्राप्त कर सके"।
* **Plist** **फ़ाइलें** कॉन्फ़िगरेशन जानकारी संग्रहीत करती हैं। आप [https://developer.apple.com/library/archive/documentation/General/Reference/InfoPlistKeyReference/Introduction/Introduction.html](https://developer.apple.com/library/archive/documentation/General/Reference/InfoPlistKeyReference/Introduction/Introduction.html) में प्लिस्ट कुंजीयों के अर्थ के बारे में जानकारी प्राप्त कर सकते हैं।
*   ऐप्लिकेशन का विश्लेषण करते समय रुचि वाले जोड़ों में शामिल हो सकते हैं:\\

* **CFBundleExecutable**

ऐप्लिकेशन के **बाइनरी का नाम** संग्रहीत करता है (Contents/MacOS में पाया जाता है)।

* **CFBundleIdentifier**

ऐप्लिकेशन का बंडल पहचानकर्ता संग्रहीत करता है (अक्सर सिस्टम द्वारा ऐप्लिकेशन को **वैश्विक रूप से पहचानने** के लिए उपयोग किया जाता है)।

* **LSMinimumSystemVersion**

ऐप्लिकेशन के संगत **macOS का सबसे पुराना संस्करण** संग्रहीत करता है।

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आप **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड** करना चाहते हैं? [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह
* प्राप्त करें [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को डाउनलोड करें।**

</details>
