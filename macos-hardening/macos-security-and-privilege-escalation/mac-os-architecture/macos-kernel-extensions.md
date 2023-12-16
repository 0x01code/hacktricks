# macOS कर्नल एक्सटेंशन्स

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ हैकट्रिक्स क्लाउड ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 ट्विटर 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ ट्विच 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 यूट्यूब 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **हैकट्रिक्स में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड** करने का एक्सेस चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) देखें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* [**PEASS और HackTricks की आधिकारिक स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* **डिस्कॉर्ड समूह** या **टेलीग्राम समूह** में **शामिल हों** या **मुझे ट्विटर पर फ़ॉलो करें** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live).
* **हैकिंग ट्रिक्स को PR भेजकर** [**हैकट्रिक्स रेपो**](https://github.com/carlospolop/hacktricks) **और** [**हैकट्रिक्स-क्लाउड रेपो**](https://github.com/carlospolop/hacktricks-cloud) **के माध्यम से अपने हैकिंग ट्रिक्स साझा करें।**

</details>

## मूलभूत जानकारी

कर्नल एक्सटेंशन्स (Kexts) एक **पैकेज** होते हैं जिनका **`.kext`** एक्सटेंशन होता है और जो **मैकओएस कर्नल स्पेस में सीधे लोड होते हैं**, मुख्य ऑपरेटिंग सिस्टम को अतिरिक्त फंक्शनालिटी प्रदान करते हैं।

### आवश्यकताएं

स्वाभाविक रूप से, यह इतना शक्तिशाली है कि कर्नल एक्सटेंशन्स को लोड करना **कठिन होता है**। एक कर्नल एक्सटेंशन को लोड करने के लिए ये हैं उन आवश्यकताएं:

* **रिकवरी मोड में प्रवेश करते समय**, कर्नल **एक्सटेंशन्स को लोड करने की अनुमति देनी चाहिए**:
  
<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* कर्नल एक्सटेंशन को **कर्नल कोड साइनिंग सर्टिफिकेट** के साथ **साइन किया जाना चाहिए**, जो केवल **एप्पल द्वारा प्रदान** किया जा सकता है। जो कंपनी और कारणों की विस्तृत समीक्षा करेगा।
* कर्नल एक्सटेंशन को **नोटराइज़्ड** भी होना चाहिए, जिसे एप्पल मैलवेयर के लिए जांच सकेगा।
* फिर, **रूट** उपयोगकर्ता ही है जो कर्नल एक्सटेंशन को **लोड कर सकता है** और पैकेज के अंदर के फ़ाइलें **रूट के नाम से होनी चाहिए**।
* अपलोड प्रक्रिया के दौरान, पैकेज को एक **सुरक्षित गैर-रूट स्थान** में तैयार किया जाना चाहिए: `/Library/StagedExtensions` (आवश्यकता है `com.apple.rootless.storage.KernelExtensionManagement` अनुदान की)।
* अंत में, इसे लोड करने का प्रयास करने पर, उपयोगकर्ता को [**पुष्टि अनुरोध प्राप्त होगा**](https://developer.apple.com/library/archive/technotes/tn2459/\_index.html) और, यदि स्वीकार किया जाता है, कंप्यूटर को इसे लोड करने के लिए **रीस्टार्ट** करना होगा।

### लोडिंग प्रक्रिया

कैटालिना में ऐसा था: ध्यान देने योग्य है कि **सत्यापन** प्रक्रिया **यूजरलैंड** में होती है। हालांकि, केवल **`com.apple.private.security.kext-management`** अनु
