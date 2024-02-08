# macOS कर्नेल एक्सटेंशन्स

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ हैकट्रिक्स क्लाउड ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 ट्विटर 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ ट्विच 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 यूट्यूब 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम कर रहे हैं? क्या आप अपनी **कंपनी को हैकट्रिक्स में जारी करना चाहते हैं**? या आप **PEASS की नवीनतम संस्करण देखना चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**सदस्यता की योजनाएं**](https://github.com/sponsors/carlospolop) देखें!
* [**द पीएस फैमिली**](https://opensea.io/collection/the-peass-family) की खास [**NFTs**](https://opensea.io/collection/the-peass-family) का खुलासा करें
* [**PEASS और HackTricks की आधिकारिक स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* **डिस्कॉर्ड समूह** या **टेलीग्राम समूह** में **शामिल हों** या **मुझे** **ट्विटर** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live) **पर फॉलो करें**.
* **हैकिंग ट्रिक्स को साझा करें** [**हैकट्रिक्स रेपो**](https://github.com/carlospolop/hacktricks) **और** [**हैकट्रिक्स-क्लाउड रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में पीआर भेजकर**.

</details>

## मूलभूत जानकारी

कर्नेल एक्सटेंशन्स (Kexts) **`.kext`** एक्सटेंशन वाले **पैकेज** होते हैं जो **मेन macOS कर्नेल स्पेस में सीधे लोड** किए जाते हैं, मुख्य ऑपरेटिंग सिस्टम को अतिरिक्त कार्यक्षमता प्रदान करते हैं।

### आवश्यकताएं

स्वाभाविक रूप से, यह इतना शक्तिशाली है कि **कर्नेल एक्सटेंशन लोड करना कठिन** है। एक कर्नेल एक्सटेंशन को लोड करने के लिए ये हैं **आवश्यकताएं**:

* **रिकवरी मोड में प्रवेश** करते समय, कर्नेल **एक्सटेंशन्स को लोड करने की अनुमति** होनी चाहिए:
* कर्नेल एक्सटेंशन को **कर्नेल कोड साइनिंग सर्टिफिकेट** के साथ **साइन किया जाना चाहिए**, जो केवल **एप्पल द्वारा प्रदान** किया जा सकता है। जो कंपनी और कारणों की विस्तृत समीक्षा करेगा कि यह क्यों आवश्यक है।
* कर्नेल एक्सटेंशन को **नोटराइज़** भी किया जाना चाहिए, जिसे एप्पल मैलवेयर के लिए जांच सकेगा।
* फिर, **रूट** उपयोगकर्ता है जो कर्नेल एक्सटेंशन को **लोड कर सकता है** और पैकेज के अंदर के फ़ाइलें **रूट के नाम से होनी चाहिए**।
* अपलोड प्रक्रिया के दौरान, पैकेज को एक **सुरक्षित गैर-रूट स्थान** में तैयार किया जाना चाहिए: `/Library/StagedExtensions` (`com.apple.rootless.storage.KernelExtensionManagement` अनुमति की आवश्यकता है)।
* अंततः, इसे लोड करने का प्रयास करते समय, उपयोगकर्ता को [**पुष्टि अनुरोध मिलेगा**](https://developer.apple.com/library/archive/technotes/tn2459/\_index.html) और यदि स्वीकृत होता है, कंप्यूटर को इसे लोड करने के लिए **रीस्टार्ट** करना होगा।

### लोडिंग प्रक्रिया

कैटालिना में यह ऐसा था: यह दिलचस्प है कि **सत्यापन** प्रक्रिया **यूज़रलैंड** में होती है। हालांकि, केवल **`com.apple.private.security.kext-management`** अनुमति वाले एप्लिकेशन ही **कर्नेल से एक्सटेंशन लोड करने का अनुरोध कर सकते हैं**: `kextcache`, `kextload`, `kextutil`, `kextd`, `syspolicyd`

1. **`kextutil`** cli **एक्सटेंशन लोड करने के लिए सत्यापन** प्रक्रिया शुरू करता है
* यह **`kextd`** से **Mach सेवा** का उपयोग करके बातचीत करेगा।
2. **`kextd`** कई चीजें जैसे **सिग्नेचर** जैसी चीजें जांचेगा
* यह **`syspolicyd`** से बातचीत करेगा कि क्या एक्सटेंशन **लोड किया जा सकता है**।
3. **`syspolicyd`** उपयोगकर्ता को **प्रॉम्प्ट** करेगा अगर एक्सटेंशन पहले से लोड नहीं हुआ है।
* **`syspolicyd`** परिणाम को **`kextd`** को रिपोर्ट करेगा
4. **`kextd`** अंततः कर्नेल को एक्सटेंशन **लोड करने के लिए कह सकेगा**

यदि **`kextd`** उपलब्ध नहीं है, तो **`kextutil`** एक ही जांच कर सकता है।

## संदर्भ

* [https://www.makeuseof.com/how-to-enable-third-party-kernel-extensions-apple-silicon-mac/](https://www.makeuseof.com/how-to-enable-third-party-kernel-extensions-apple-silicon-mac/)
* [https://www.youtube.com/watch?v=hGKOskSiaQo](https://www.youtube.com/watch?v=hGKOskSiaQo)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ हैकट्रिक्स क्लाउड ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 ट्विटर 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ ट्विच 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 यूट्यूब 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम कर रहे हैं? क्या आप अपनी **कंपनी को हैकट्रिक्स में जारी करना चाहते हैं**? या आप **PEASS की नवीनतम संस्करण देखना चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**सदस्यता की योजनाएं**](https://github.com/sponsors/carlospolop) देखें!
* [**द पीएस फैमिली**](https://opensea.io/collection/the-peass-family) की खास [**NFTs**](https://opensea.io/collection/the-peass-family) का खुलासा करें
* [**PEASS और HackTricks की आधिकारिक स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* **डिस्कॉर्ड समूह** या **टेलीग्राम समूह** में **शामिल हों** या **मुझे** **ट्विटर** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live) **पर फॉलो करें**.
* **हैकिंग ट्रिक्स को साझा करें** [**हैकट्रिक्स रेपो**](https://github.com/carlospolop/hacktricks) **और** [**हैकट्रिक्स-क्लाउड रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में पीआर भेजकर**.

</details>
