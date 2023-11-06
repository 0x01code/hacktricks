# macOS Office सैंडबॉक्स बाईपास

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks क्लाउड ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 ट्विटर 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ ट्विच 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 यूट्यूब 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने की आवश्यकता** है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एकल [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल** हों या मुझे **ट्विटर** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का** **अनुसरण** करें।**
* **अपने हैकिंग ट्रिक्स को** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में पीआर जमा करके अपना योगदान दें।**

</details>

### शब्द सैंडबॉक्स बाईपास लॉन्च एजेंट के माध्यम से

एप्लिकेशन एक **कस्टम सैंडबॉक्स** का उपयोग करता है जिसमें **`com.apple.security.temporary-exception.sbpl`** इंटाइटलमेंट का उपयोग किया जाता है और यह कस्टम सैंडबॉक्स `~$` से शुरू होने वाले फ़ाइल नाम के साथ कहीं भी फ़ाइलें लिखने की अनुमति देता है: `(require-any (require-all (vnode-type REGULAR-FILE) (regex #"(^|/)~$[^/]+$")))`

इसलिए, बाईपास करना इतना आसान था जितना कि **`plist`** लॉन्च एजेंट को `~/Library/LaunchAgents/~$escape.plist` में लिखना।

[**यहां मूल रिपोर्ट देखें**](https://www.mdsec.co.uk/2018/08/escaping-the-sandbox-microsoft-office-on-macos/).

### वर्ड सैंडबॉक्स बाईपास लॉगिन आइटम और ज़िप के माध्यम से

(ध्यान दें कि पहले बाईपास से, वर्ड अनियंत्रित फ़ाइलें लिख सकता है जिनका नाम `~$` से शुरू होता है, हालांकि पिछले दुर्घटना के पैच के बाद `/Library/Application Scripts` या `/Library/LaunchAgents` में लिखना संभव नहीं था)।

यह पता चला कि सैंडबॉक्स के भीतर से एक **लॉगिन आइटम** (जब उपयोगकर्ता लॉग इन करता है तो चलाए जाने वाले ऐप्स) बनाना संभव है। हालांकि, ये ऐप्स **केवल तब चलेंगे जब** वे **नोटराइज़्ड** होंगे और इसमें **आर्ग्यूमेंट जोड़ना संभव नहीं है** (इसलिए आप बस **`बैश`** का उपयोग करके एक रिवर्स शेल नहीं चला सकते हैं)।

पिछले सैंडबॉक्स बाईपास से, माइक्रोसॉफ्ट ने `~/Library/LaunchAgents` में फ़ाइलें लिखने के विकल्प को अक्षम कर दिया था। हालांकि, यह पता चला कि यदि आप एक **ज़िप फ़ाइल को लॉगिन आइटम** के रूप में डालते हैं तो `Archive Utility` इसे अपने मौजूदा स्थान पर **अनज़िप** कर देगा। इसलिए, चूंकि डिफ़ॉल्ट रूप से `~/Library` के `LaunchAgents` फ़ोल्डर को नहीं बनाया जाता है, इसलिए यह संभव था कि `LaunchAgents/~$escape.plist` में एक प्लिस्ट ज़िप करें और जब इसे
* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आप **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड** करना चाहते हैं? [**सदस्यता योजनाओं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT संग्रह**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल हों** या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें, PRs को** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में सबमिट करके**.
