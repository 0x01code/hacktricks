# macOS Apple Scripts

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **फॉलो** करें मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को।**

</details>

## Apple Scripts

यह एक स्क्रिप्टिंग भाषा है जिसका उपयोग कार्य स्वचालन के लिए किया जाता है **दूरस्थ प्रक्रियाओं के साथ बातचीत करने** के लिए। इससे **अन्य प्रक्रियाओं को कुछ कार्रवाई करने के लिए कहना बहुत आसान हो जाता है**। **मैलवेयर** इन सुविधाओं का दुरुपयोग करके अन्य प्रक्रियाओं द्वारा निर्यात की गई फ़ंक्शन का दुरुपयोग कर सकता है।\
उदाहरण के लिए, एक मैलवेयर ब्राउज़र खोले गए पृष्ठों में **विचित्र JS कोड संचोदित कर सकता है**। या उपयोगकर्ता से मांगी गई कुछ अनुमतियों को **ऑटो क्लिक** कर सकता है।
```applescript
tell window 1 of process "SecurityAgent"
click button "Always Allow" of group 1
end tell
```
यहां आपको कुछ उदाहरण मिलेंगे: [https://github.com/abbeycode/AppleScripts](https://github.com/abbeycode/AppleScripts)\
एपल स्क्रिप्ट का उपयोग करके मैलवेयर के बारे में अधिक जानकारी [**यहां**](https://www.sentinelone.com/blog/how-offensive-actors-use-applescript-for-attacking-macos/) मिलेगी।

एपल स्क्रिप्ट आसानी से "**कंपाइल**" किए जा सकते हैं। इन संस्करणों को `osadecompile` के साथ आसानी से "**डीकंपाइल**" किया जा सकता है।

हालांकि, ये स्क्रिप्ट भी **"केवल पढ़ने योग्य"** रूप में निर्यात किए जा सकते हैं (विकल्प "निर्यात करें..." के माध्यम से):

<figure><img src="https://github.com/carlospolop/hacktricks/raw/master/.gitbook/assets/image%20(556).png" alt=""><figcaption></figcaption></figure>
```
file mal.scpt
mal.scpt: AppleScript compiled
```
और इस मामले में `osadecompile` के साथ भी सामग्री को डीकंपाइल नहीं किया जा सकता है।

हालांकि, इस प्रकार के एक्सीक्यूटेबल को समझने के लिए अभी भी कुछ उपकरण हैं, [**अधिक जानकारी के लिए इस शोध को पढ़ें**](https://labs.sentinelone.com/fade-dead-adventures-in-reversing-malicious-run-only-applescripts/)। उपकरण [**applescript-disassembler**](https://github.com/Jinmo/applescript-disassembler) के साथ [**aevt\_decompile**](https://github.com/SentineLabs/aevt\_decompile) एक्सीक्यूटेबल को समझने में बहुत उपयोगी होंगे।

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने की आवश्यकता** है? [**सदस्यता योजनाओं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह।
* प्राप्त करें [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को फॉलो करके।**

</details>
