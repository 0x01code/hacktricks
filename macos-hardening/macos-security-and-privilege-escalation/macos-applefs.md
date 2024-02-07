# macOS AppleFS

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप चाहते हैं कि आपकी **कंपनी HackTricks में विज्ञापित हो** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें** द्वारा PRs सबमिट करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

## Apple Propietary File System (APFS)

**Apple File System (APFS)** एक आधुनिक फ़ाइल सिस्टम है जो Hierarchical File System Plus (HFS+) को बदलने के लिए डिज़ाइन किया गया है। इसका विकास **बेहतर प्रदर्शन, सुरक्षा, और कुशलता** की आवश्यकता से हुआ था।

APFS की कुछ प्रमुख विशेषताएँ शामिल हैं:

1. **Space Sharing**: APFS एक ही फिजिकल उपकरण पर मौजूद नि:शुल्क स्टोरेज को **एक ही भूतकालिक वॉल्यूम के साथ साझा करने की अनुमति देता है**। यह वॉल्यूम डायनामिक रूप से बढ़ और घट सकते हैं बिना मैनुअल आकार बदलाव या पुनर्विभाजन की आवश्यकता के।
1. इसका मतलब है, तुलनात्मक रूप से **कि APFS में विभिन्न विभाजन (वॉल्यूम) सभी डिस्क स्थान को साझा करते हैं**, जबकि एक साधारण विभाजन में सामान्यत: एक निर्धारित आकार होता था।
2. **Snapshots**: APFS **स्नैपशॉट बनाने** का समर्थन करता है, जो फ़ाइल सिस्टम के समय-सीमा में **केवल पढ़ने योग्य** उदाहरण होते हैं। स्नैपशॉट एक्सपर्ट बैकअप और आसान सिस्टम रोलबैक को संभव बनाते हैं, क्योंकि वे कम अतिरिक्त स्टोरेज का उपभोग करते हैं और त्वरित रूप से बनाए जा सकते हैं या पुनर्वर्तित किए जा सकते हैं।
3. **Clones**: APFS **फ़ाइल या निर्देशिका क्लोन बना सकता है जो मूल के साथ समान स्टोरेज को साझा करते हैं** जब तक या तो क्लोन या मूल फ़ाइल संशोधित नहीं होती। यह सुविधा फ़ाइल या निर्देशिकाओं की प्रतिलिपि बनाने का एक कुशल तरीका प्रदान करती है बिना स्टोरेज स्थान को दोहराने की आवश्यकता के।
4. **Encryption**: APFS **पूरे-डिस्क एन्क्रिप्शन** का समर्थन करता है साथ ही प्रति-फ़ाइल और प्रति-निर्देशिका एन्क्रिप्शन, विभिन्न उपयोग मामलों में डेटा सुरक्षा को बढ़ावा देता है।
5. **Crash Protection**: APFS एक **कॉपी-ऑन-राइट मेटाडेटा योजना का उपयोग करता है जो फ़ाइल सिस्टम संरचना की संरचना सुनिश्चित करती है** यहाँ तक कि अचानक बिजली की चोट या सिस्टम क्रैश के मामलों में भी, डेटा करप्शन के जोखिम को कम करते हैं।

सम्ग्र, APFS Apple उपकरणों के लिए एक और आधुनिक, लचीला, और कुशल फ़ाइल सिस्टम प्रदान करता है, जिसका मुख्य ध्यान प्रदर्शन, विश्वसनीयता, और सुरक्षा पर है।
```bash
diskutil list # Get overview of the APFS volumes
```
## Firmlinks

`Data` वॉल्यूम **`/System/Volumes/Data`** में माउंट किया गया है (आप `diskutil apfs list` के साथ इसे चेक कर सकते हैं).

फर्मलिंक्स की सूची **`/usr/share/firmlinks`** फ़ाइल में मिल सकती है।
```bash
cat /usr/share/firmlinks
/AppleInternal	AppleInternal
/Applications	Applications
/Library	Library
[...]
```
**बाएं** में, **सिस्टम वॉल्यूम** पर निर्देशिका पथ है, और **दाएं** में, उस निर्देशिका पथ को दिखाता है जहां यह **डेटा वॉल्यूम** पर मैप होता है। तो, `/library` --> `/system/Volumes/data/library`
