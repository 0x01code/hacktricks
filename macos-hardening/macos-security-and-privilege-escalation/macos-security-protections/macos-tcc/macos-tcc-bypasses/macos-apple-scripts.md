# macOS Apple Scripts

<details>

<summary><strong>Learn AWS hacking from zero to hero with</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) को **फॉलो करें**.
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स शेयर करें।

</details>

## Apple Scripts

यह एक स्क्रिप्टिंग भाषा है जिसका उपयोग **रिमोट प्रोसेसेस के साथ इंटरैक्ट करते हुए** कार्य स्वचालन के लिए किया जाता है। यह बहुत आसान बनाता है **अन्य प्रोसेसेस से कुछ क्रियाएं करने के लिए कहना**। **मैलवेयर** इन सुविधाओं का दुरुपयोग कर सकता है अन्य प्रोसेसेस द्वारा निर्यात किए गए फंक्शन्स का दुरुपयोग करने के लिए।\
उदाहरण के लिए, मैलवेयर **ब्राउज़र में खुले पेजों में मनमाना JS कोड इंजेक्ट कर सकता है**। या उपयोगकर्ता को अनुरोधित कुछ अनुमति स्वीकृतियों पर **ऑटो क्लिक** कर सकता है;
```applescript
tell window 1 of process "SecurityAgent"
click button "Always Allow" of group 1
end tell
```
यहाँ कुछ उदाहरण दिए गए हैं: [https://github.com/abbeycode/AppleScripts](https://github.com/abbeycode/AppleScripts)\
AppleScripts का उपयोग करके मैलवेयर के बारे में अधिक जानकारी [**यहाँ**](https://www.sentinelone.com/blog/how-offensive-actors-use-applescript-for-attacking-macos/) पाएं।

Apple स्क्रिप्ट्स को आसानी से "**संकलित**" किया जा सकता है। इन संस्करणों को `osadecompile` के साथ आसानी से "**डीकंपाइल**" किया जा सकता है।

हालांकि, इन स्क्रिप्ट्स को "**केवल पढ़ने के लिए**" के रूप में भी **निर्यात** किया जा सकता है ( "Export..." विकल्प के माध्यम से):

<figure><img src="https://github.com/carlospolop/hacktricks/raw/master/.gitbook/assets/image%20(556).png" alt=""><figcaption></figcaption></figure>
```
file mal.scpt
mal.scpt: AppleScript compiled
```
```markdown
और इस मामले में सामग्री को `osadecompile` का उपयोग करके भी डीकंपाइल नहीं किया जा सकता

हालांकि, कुछ ऐसे टूल्स अभी भी हैं जिनका उपयोग इस प्रकार के एक्जीक्यूटेबल्स को समझने के लिए किया जा सकता है, [**इस शोध को अधिक जानकारी के लिए पढ़ें**](https://labs.sentinelone.com/fade-dead-adventures-in-reversing-malicious-run-only-applescripts/)). टूल [**applescript-disassembler**](https://github.com/Jinmo/applescript-disassembler) के साथ [**aevt\_decompile**](https://github.com/SentineLabs/aevt\_decompile) स्क्रिप्ट को समझने में बहुत उपयोगी होगा।

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) को **फॉलो** करें।
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>
```
