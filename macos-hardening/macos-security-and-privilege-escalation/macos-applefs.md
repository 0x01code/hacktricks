# macOS AppleFS

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**official PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**telegram समूह**](https://t.me/peass) या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें और PRs सबमिट करें** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को**

</details>

## Apple Propietary File System (APFS)

APFS, यानी Apple File System, एक आधुनिक फ़ाइल सिस्टम है जिसे Apple Inc. ने विकसित किया है और इसका उद्देश्य पुराने Hierarchical File System Plus (HFS+) को बदलना था जिसमें **बेहतर प्रदर्शन, सुरक्षा और कुशलता** को महत्व दिया गया है।

APFS की कुछ महत्वपूर्ण विशेषताएं शामिल हैं:

1. **स्थान साझाकरण**: APFS एक ही शारीरिक उपकरण पर आधारित मुक्त संग्रह को **साझा करने की अनुमति देता है**। इससे स्थान का अधिक उपयोग किया जा सकता है क्योंकि संग्रह डाइनामिक रूप से बढ़ और घट सकते हैं बिना मैनुअल आकार बदलने या पुनर्विभाजन की आवश्यकता के।
1. इसका मतलब है, फ़ाइल डिस्क में पारंपरिक विभाजनों की तुलना में, **APFS में विभिन्न विभाजन (संग्रह) सभी डिस्क स्थान को साझा करते हैं**, जबकि एक सामान्य विभाजन में आमतौर पर एक निश्चित आकार होता था।
2. **स्नैपशॉट**: APFS **स्नैपशॉट बनाने का समर्थन करता है**, जो फ़ाइल सिस्टम के समय-सीमा में **केवल पढ़ने योग्य** उदाहरण हैं। स्नैपशॉट सुरक्षित बैकअप और सिस्टम को आसानी से वापस लाने की सुविधा प्रदान करते हैं, क्योंकि इन्हें कम से कम अतिरिक्त संग्रह का उपयोग करते हैं और त्वरित रूप से बनाए जा सकते हैं या पूर्ववत किए जा सकते हैं।
3. **क्लोन**: APFS **फ़ाइल या निर्देशिका क्लोन बना सकता है जो मूल संग्रह के साथ समान संग्रह को साझा करते हैं** जब तक कि क्लोन या मूल फ़ाइल संशोधित नहीं होती है। यह सुविधा फ़ाइल या निर्देशिका की प्रतिलिपि बनाने के लिए संग्रह स्थान को दोहराने का एक कुशल तरीका प्रदान करती है।
4. **एन्क्रिप्शन**: APFS **पूरे डिस्क एन्क्रिप्शन का समर्थन** करता है साथ ही फ़ाइल और निर्देशिका एन्क्रिप्शन, विभिन्न उपयोग मामलों में डेटा सुरक्षा को बढ़ाता है।
5. **क्रैश सुरक्षा**: APFS एक **कॉपी-ऑन-राइट मेटाडेटा योजना का उपयोग करता है जो फ़ाइल सिस्टम की संगतता सुनिश्चित करता है**, यहां तक कि अचानक बिजली की आपातकालीन गुमनामी या सिस्टम क्रैश के मामलों में भी, डेटा के करप्शन का जोखिम कम हो जाता है।

समग्र रूप से, APFS Apple उपकरणों के लिए एक अधिक आधुनिक, लचीला और कुशल फ़ाइल सिस्टम प्रदान करता है, जिसमें प्रदर्शन, विश्वसनीयता और सुरक्षा में सुधार का ध्यान दिया गया है।
```bash
diskutil list # Get overview of the APFS volumes
```
## Firmlinks

`Data` वॉल्यूम **`/System/Volumes/Data`** में माउंट होता है (आप `diskutil apfs list` के साथ इसे जांच सकते हैं।)

फर्मलिंक्स की सूची **`/usr/share/firmlinks`** फ़ाइल में मिल सकती है।
```bash
cat /usr/share/firmlinks
/AppleInternal	AppleInternal
/Applications	Applications
/Library	Library
[...]
```
**बाएं** में, **सिस्टम वॉल्यूम** पर निर्देशिका पथ है, और **दाएं** में, यह निर्देशिका पथ है जहां यह **डेटा वॉल्यूम** पर मैप होता है। इसलिए, `/library` --> `/system/Volumes/data/library`
