# macOS कर्नेल एक्सटेंशन्स

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert) के साथ</strong></a><strong>!</strong></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम कर रहे हैं? क्या आप अपनी **कंपनी को हैकट्रिक्स में विज्ञापित करना चाहते हैं**? या आपको **PEASS का नवीनतम संस्करण देखना है या HackTricks को PDF में डाउनलोड करना है**? [**सदस्यता योजनाएं देखें**](https://github.com/sponsors/carlospolop)!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खास [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह का खुलासा करें
* [**PEASS और HackTricks की आधिकारिक स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* **डिस्कॉर्ड समूह** में **शामिल हों** या [**टेलीग्राम समूह**](https://t.me/peass) में शामिल हों या **मुझे** **ट्विटर** पर फॉलो करें 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)।
* **हैकिंग ट्रिक्स को साझा करें** [**हैकट्रिक्स रेपो**](https://github.com/carlospolop/hacktricks) **और** [**हैकट्रिक्स-क्लाउड रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में पीआर भेजकर**।

</details>

## मूल जानकारी

कर्नेल एक्सटेंशन्स (Kexts) **पैकेज** होते हैं जिनका **`.kext`** एक्सटेंशन होता है और जो **मेन macOS कर्नेल स्पेस में सीधे लोड होते हैं**, मुख्य ऑपरेटिंग सिस्टम को अतिरिक्त कार्यक्षमता प्रदान करते हैं।

### आवश्यकताएं

स्वाभाविक रूप से, यह इतना शक्तिशाली है कि **कर्नेल एक्सटेंशन लोड करना कठिन** है। एक कर्नेल एक्सटेंशन को लोड करने के लिए ये हैं **आवश्यकताएं**:

* **रिकवरी मोड में प्रवेश** करते समय, कर्नेल **एक्सटेंशन्स को लोड करने की अनुमति** होनी चाहिए:
* कर्नेल एक्सटेंशन को **कर्नेल कोड साइनिंग सर्टिफिकेट** के साथ **साइन किया जाना चाहिए**, जो केवल **एप्पल द्वारा प्रदान** किया जा सकता है। जो कंपनी और क्यों इसकी आवश्यकता है, उसे विस्तार से समीक्षा करेगा।
* कर्नेल एक्सटेंशन को **नोटराइज़** भी किया जाना चाहिए, जिसे एप्पल मैलवेयर के लिए जांच सकेगा।
* फिर, **रूट** उपयोगकर्ता है जो कर्नेल एक्सटेंशन को **लोड कर सकता है** और पैकेज के अंदर के फ़ाइलें **रूट के होनी चाहिए**।
* अपलोड प्रक्रिया के दौरान, पैकेज को एक **सुरक्षित गैर-रूट स्थान** में तैयार किया जाना चाहिए: `/Library/StagedExtensions` (की अनुमति `com.apple.rootless.storage.KernelExtensionManagement` की आवश्यकता है)।
* अंततः, इसे लोड करने का प्रयास करते समय, उपयोगकर्ता को [**पुष्टि अनुरोध मिलेगा**](https://developer.apple.com/library/archive/technotes/tn2459/\_index.html) और अगर स्वीकृति मिलती है, तो कंप्यूटर को इसे लोड करने के लिए **रीस्टार्ट** करना होगा।

### लोडिंग प्रक्रिया

कैटालिना में यह ऐसा था: यह दिलचस्प है कि **सत्यापन** प्रक्रिया **यूज़रलैंड** में होती है। हालांकि, केवल ऐप्लिकेशन जिनके पास **`com.apple.private.security.kext-management`** अनुमति है, केवल **कर्नेल को एक्सटेंशन लोड करने का अनुरोध कर सकते हैं**: `kextcache`, `kextload`, `kextutil`, `kextd`, `syspolicyd`

1. **`kextutil`** cli **एक्सटेंशन लोड करने के लिए सत्यापन** प्रक्रिया शुरू करेगा
* यह **`kextd`** से बातचीत करेगा जिसे एक **Mach सेवा** का उपयोग करके भेजा जाएगा।
2. **`kextd`** कई चीजें जैसे **सिग्नेचर** की जांच करेगा
* यह **`syspolicyd`** से बातचीत करेगा ताकि एक्सटेंशन को **लोड किया जा सके**।
3. **`syspolicyd`** उपयोगकर्ता को **प्रॉम्प्ट** करेगा अगर एक्सटेंशन पहले से लोड नहीं हुआ है।
* **`syspolicyd`** परिणाम को **`kextd`** को रिपोर्ट करेगा
4. **`kextd`** अंततः कर्नेल को एक्सटेंशन **लोड करने के लिए कह सकेगा**

अगर **`kextd`** उपलब्ध नहीं है, तो **`kextutil`** एक ही जांच कर सकता है।

## संदर्भ

* [https://www.makeuseof.com/how-to-enable-third-party-kernel-extensions-apple-silicon-mac/](https://www.makeuseof.com/how-to-enable-third-party-kernel-extensions-apple-silicon-mac/)
* [https://www.youtube.com/watch?v=hGKOskSiaQo](https://www.youtube.com/watch?v=hGKOskSiaQo)

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम कर रहे हैं? क्या आप अपनी **कंपनी को हैकट्रिक्स में विज्ञापित करना चाहते हैं**? या आपको **PEASS का नवीनतम संस्करण देखना है या HackTricks को PDF में डाउनलोड करना है**? [**सदस्यता योजनाएं देखें**](https://github.com/sponsors/carlospolop)!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खास [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह का खुलासा करें
* [**PEASS और HackTricks की आधिकारिक स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* **डिस्कॉर्ड समूह** में **शामिल हों** या [**टेलीग्राम समूह**](https://t.me/peass) में शामिल हों या **मुझे** **ट्विटर** पर फॉलो करें 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)।
* **हैकिंग ट्रिक्स को साझा करें** [**हैकट्रिक्स रेपो**](https://github.com/carlospolop/hacktricks) **और** [**हैकट्रिक्स-क्लाउड रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में पीआर भेजकर**।

</details>
