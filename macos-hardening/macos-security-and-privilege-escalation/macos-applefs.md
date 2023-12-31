# macOS AppleFS

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग प्राप्त करें**](https://peass.creator-spring.com)
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) को **फॉलो करें**.
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स शेयर करें.

</details>

## Apple Propietary File System (APFS)

APFS, या Apple File System, Apple Inc. द्वारा विकसित एक आधुनिक फाइल सिस्टम है जिसे पुराने Hierarchical File System Plus (HFS+) को बदलने के लिए डिजाइन किया गया था, जिसमें **प्रदर्शन, सुरक्षा, और कार्यक्षमता में सुधार** पर जोर दिया गया है।

APFS की कुछ उल्लेखनीय विशेषताएं हैं:

1. **Space Sharing**: APFS एकल भौतिक डिवाइस पर **समान अंतर्निहित मुक्त स्टोरेज को साझा करने की अनुमति देता है**। यह अधिक कुशल स्थान उपयोग को सक्षम बनाता है क्योंकि वॉल्यूम गतिशील रूप से बढ़ और सिकुड़ सकते हैं बिना मैन्युअल रीसाइजिंग या रिपार्टिशनिंग की आवश्यकता के।
2. इसका मतलब है, पारंपरिक पार्टिशन की तुलना में फाइल डिस्क में, **APFS में विभिन्न पार्टिशन (वॉल्यूम) सभी डिस्क स्पेस को साझा करते हैं**, जबकि एक नियमित पार्टिशन आमतौर पर एक निश्चित आकार का होता था।
3. **Snapshots**: APFS **स्नैपशॉट्स बनाने का समर्थन करता है**, जो **केवल-पढ़ने के लिए**, समय-बिंदु के उदाहरण होते हैं फाइल सिस्टम के। स्नैपशॉट्स कुशल बैकअप और आसान सिस्टम रोलबैक को सक्षम बनाते हैं, क्योंकि वे न्यूनतम अतिरिक्त स्टोरेज का उपभोग करते हैं और जल्दी से बनाए या वापस लाए जा सकते हैं।
4. **Clones**: APFS **फाइल या डायरेक्टरी क्लोन बना सकता है जो समान स्टोरेज को साझा करते हैं** जैसे मूल तब तक जब तक क्लोन या मूल फाइल में संशोधन नहीं किया जाता। यह सुविधा फाइलों या डायरेक्टरीज की प्रतियां बनाने का एक कुशल तरीका प्रदान करती है बिना स्टोरेज स्थान को दोहराए।
5. **Encryption**: APFS **पूर्ण-डिस्क एन्क्रिप्शन का स्वाभाविक रूप से समर्थन करता है** साथ ही प्रति-फाइल और प्रति-डायरेक्टरी एन्क्रिप्शन, विभिन्न उपयोग के मामलों में डेटा सुरक्षा को बढ़ाता है।
6. **Crash Protection**: APFS एक **कॉपी-ऑन-राइट मेटाडेटा योजना का उपयोग करता है जो अचानक बिजली की हानि या सिस्टम क्रैश के मामलों में भी फाइल सिस्टम की स्थिरता सुनिश्चित करता है**, डेटा भ्रष्टाचार के जोखिम को कम करता है।

कुल मिलाकर, APFS Apple उपकरणों के लिए एक अधिक आधुनिक, लचीला, और कुशल फाइल सिस्टम प्रदान करता है, जिसमें प्रदर्शन, विश्वसनीयता, और सुरक्षा में सुधार पर ध्यान केंद्रित है।
```bash
diskutil list # Get overview of the APFS volumes
```
## फर्मलिंक्स

`Data` वॉल्यूम **`/System/Volumes/Data`** में माउंट किया गया है (आप इसे `diskutil apfs list` के साथ जांच सकते हैं)।

फर्मलिंक्स की सूची **`/usr/share/firmlinks`** फाइल में पाई जा सकती है।
```bash
cat /usr/share/firmlinks
/AppleInternal	AppleInternal
/Applications	Applications
/Library	Library
[...]
```
<details>

<summary><strong>शून्य से नायक तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग ट्रिक्स शेयर करें।

</details>
