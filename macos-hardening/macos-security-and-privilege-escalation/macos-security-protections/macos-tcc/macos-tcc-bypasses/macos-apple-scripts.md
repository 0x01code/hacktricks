# macOS एप्पल स्क्रिप्ट्स

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, HackTricks और HackTricks Cloud** github रेपो में PR जमा करके।

</details>

## एप्पल स्क्रिप्ट्स

यह एक स्क्रिप्टिंग भाषा है जिसका उपयोग कार्य स्वचालन के लिए **दूरस्थ प्रक्रियाओं के साथ बातचीत** करने के लिए किया जाता है। यह किसी अन्य प्रक्रियाएँ कुछ कार्रवाई करने के लिए कहना बहुत आसान बना देता है। **मैलवेयर** इन सुविधाओं का दुरुपयोग कर सकता है ताकि अन्य प्रक्रियाएँ द्वारा निर्यात की गई कार्यों का दुरुपयोग करें।\
उदाहरण के लिए, एक मैलवेयर **ब्राउज़र खुली पृष्ठों में विचित्र JS कोड इंजेक्ट** कर सकता है। या उपयोगकर्ता से अनुमति मांगने वाली कुछ अनुमति को **स्वचालित रूप से क्लिक** कर सकता है।
```applescript
tell window 1 of process "SecurityAgent"
click button "Always Allow" of group 1
end tell
```
यहाँ आपको कुछ उदाहरण मिलेंगे: [https://github.com/abbeycode/AppleScripts](https://github.com/abbeycode/AppleScripts)\
एप्पलस्क्रिप्ट का उपयोग करके मैलवेयर के बारे में अधिक जानकारी [**यहाँ**](https://www.sentinelone.com/blog/how-offensive-actors-use-applescript-for-attacking-macos/) मिलेगी।

एप्पल स्क्रिप्ट आसानी से "**कॉम्पाइल**" किए जा सकते हैं। इन संस्करणों को `osadecompile` के साथ आसानी से "**डीकॉम्पाइल**" किया जा सकता है।

हालांकि, ये स्क्रिप्ट भी **"केवल पढ़ने के लिए"** के रूप में निर्यात किए जा सकते हैं (विकल्प "निर्यात..." के माध्यम से):

<figure><img src="https://github.com/carlospolop/hacktricks/raw/master/.gitbook/assets/image%20(556).png" alt=""><figcaption></figcaption></figure>
```
file mal.scpt
mal.scpt: AppleScript compiled
```
और इस मामले में सामग्री को `osadecompile` के साथ भी डिकंपाइल नहीं किया जा सकता

हालांकि, इस प्रकार के एक्जीक्यूटेबल्स को समझने के लिए कुछ उपकरण अभी भी हैं, [**अधिक जानकारी के लिए इस रिसर्च को पढ़ें**](https://labs.sentinelone.com/fade-dead-adventures-in-reversing-malicious-run-only-applescripts/)). उपकरण [**applescript-disassembler**](https://github.com/Jinmo/applescript-disassembler) के साथ [**aevt\_decompile**](https://github.com/SentineLabs/aevt\_decompile) काफी उपयोगी होगा ताकि स्क्रिप्ट काम कैसे करता है, समझने में।
