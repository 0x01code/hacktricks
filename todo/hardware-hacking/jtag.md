<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप चाहते हैं कि आपकी **कंपनी HackTricks में विज्ञापित हो** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** पर **फॉलो** करें 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें, HackTricks** को PRs जमा करके](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>


# JTAGenum

[**JTAGenum** ](https://github.com/cyphunk/JTAGenum)एक टूल है जिसका उपयोग Raspberry PI या Arduino के साथ किया जा सकता है ताकि एक अज्ञात चिप से JTAG पिन्स की कोशिश की जा सके।\
**Arduino** में, **2 से 11 पिन को 10 पिन्स से जोड़ें जो संभावित रूप से JTAG के हो सकते हैं**। Arduino में कार्यक्रम लोड करें और यह सभी पिन्स को ब्रूटफोर्स करने के लिए प्रयास करेगा ताकि पता लगाया जा सके कि क्या कोई पिन JTAG का है और प्रत्येक कौन सा है।\
**Raspberry PI** में आप केवल **1 से 6 पिन्स का उपयोग कर सकते हैं** (6 पिन्स, इसलिए आप हर संभावित JTAG पिन का परीक्षण करने में धीमे हो जाएंगे)।

## Arduino

Arduino में, केबल जोड़ने के बाद (पिन 2 से 11 को JTAG पिन्स और Arduino GND को बेसबोर्ड GND से जोड़ें), **Arduino में JTAGenum प्रोग्राम लोड करें** और सीरियल मॉनिटर में एक **`h`** (सहायता के लिए कमांड) भेजें और आपको सहायता दिखनी चाहिए:

![](<../../.gitbook/assets/image (643).png>)

![](<../../.gitbook/assets/image (650).png>)

**"कोई लाइन एंडिंग" और 115200baud** कॉन्फ़िगर करें।\
स्कैनिंग शुरू करने के लिए कमांड s भेजें:

![](<../../.gitbook/assets/image (651) (1) (1) (1).png>)

अगर आप JTAG से संपर्क कर रहे हैं, तो आपको एक या कई **"FOUND!" से शुरू होने वाली लाइनें** मिलेंगी जो JTAG के पिन्स को दर्शाएंगी।

</details>
