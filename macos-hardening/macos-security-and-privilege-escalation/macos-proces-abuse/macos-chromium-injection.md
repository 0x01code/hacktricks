# macOS Chromium Injection

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

दूसरे तरीके HackTricks का समर्थन करने के लिए:

* अगर आप अपनी **कंपनी को HackTricks में विज्ञापित करना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)** पर **फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके।

</details>

## मूल जानकारी

Google Chrome, Microsoft Edge, Brave और अन्य च्रोमियम-आधारित ब्राउज़र। ये ब्राउज़र च्रोमियम मुक्त स्रोत परियोजना पर निर्मित हैं, जिसका मतलब है कि उनके पास एक समान आधार है और इसलिए उनमें समान कार्यक्षमताएँ और डेवलपर विकल्प होते हैं।

#### `--load-extension` ध्वज

`--load-extension` ध्वज का उपयोग जब चालू किया जाता है एक च्रोमियम-आधारित ब्राउज़र को कमांड लाइन या स्क्रिप्ट से। यह ध्वज **ब्राउज़र में एक या एक से अधिक एक्सटेंशन को स्वचालित रूप से लोड करने की अनुमति देता है**।

#### `--use-fake-ui-for-media-stream` ध्वज

`--use-fake-ui-for-media-stream` ध्वज एक और कमांड लाइन विकल्प है जो चालू करने के लिए उपयोग किया जा सकता है च्रोमियम-आधारित ब्राउज़र। यह ध्वज **सामान्य उपयोगकर्ता प्रॉम्प्ट को छलना करने के लिए डिज़ाइन किया गया है जो कैमरा और माइक्रोफोन से मीडिया स्ट्रीम तक पहुँचने की अनुमति के लिए पूछता है**। जब यह ध्वज उपयोग किया जाता है, तो ब्राउज़र स्वचालित रूप से किसी भी वेबसाइट या एप्लिकेशन को अनुमति प्रदान करता है जो कैमरा या माइक्रोफोन तक पहुँचने का अनुरोध करता है।

### उपकरण

* [https://github.com/breakpointHQ/snoop](https://github.com/breakpointHQ/snoop)
* [https://github.com/breakpointHQ/VOODOO](https://github.com/breakpointHQ/VOODOO)

### उदाहरण
```bash
# Intercept traffic
voodoo intercept -b chrome
```
## संदर्भ

* [https://twitter.com/RonMasas/status/1758106347222995007](https://twitter.com/RonMasas/status/1758106347222995007)

<details>

<summary><strong>जीरो से हीरो तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** द्वारा PRs सबमिट करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>
