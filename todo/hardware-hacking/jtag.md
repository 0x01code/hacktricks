<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स शेयर करें।

</details>


# JTAGenum

[**JTAGenum**](https://github.com/cyphunk/JTAGenum) एक उपकरण है जिसका उपयोग Raspberry PI या Arduino के साथ किया जा सकता है ताकि अज्ञात चिप से JTAG पिन्स का पता लगाया जा सके।\
**Arduino** में, **2 से 11 तक के पिन्स को JTAG के 10 पिन्स से जोड़ें**। Arduino में प्रोग्राम लोड करें और यह सभी पिन्स को ब्रूटफोर्स करके यह पता लगाने की कोशिश करेगा कि कौन से पिन्स JTAG के हैं और प्रत्येक कौन सा है।\
**Raspberry PI** में आप केवल **1 से 6 तक के पिन्स का उपयोग कर सकते हैं** (6 पिन्स, इसलिए आप प्रत्येक संभावित JTAG पिन का परीक्षण धीरे-धीरे करेंगे)।

## Arduino

Arduino में, केबल्स को जोड़ने के बाद (पिन 2 से 11 तक JTAG पिन्स और Arduino GND को बेसबोर्ड GND से), **Arduino में JTAGenum प्रोग्राम लोड करें** और Serial Monitor में **`h`** भेजें (मदद के लिए कमांड) और आपको मदद दिखाई देगी:

![](<../../.gitbook/assets/image (643).png>)

![](<../../.gitbook/assets/image (650).png>)

**"No line ending" और 115200baud** को कॉन्फ़िगर करें।\
स्कैनिंग शुरू करने के लिए s कमांड भेजें:

![](<../../.gitbook/assets/image (651) (1) (1) (1).png>)

यदि आप JTAG से संपर्क कर रहे हैं, तो आपको एक या कई **लाइनें FOUND! से शुरू होती हुई** मिलेंगी जो JTAG के पिन्स को इंगित करती हैं।


<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स शेयर करें।

</details>
