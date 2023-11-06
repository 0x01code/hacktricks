# पिकल रिक

## पिकल रिक

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ हैकट्रिक्स क्लाउड ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 ट्विटर 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ ट्विच 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 यूट्यूब 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को हैकट्रिक्स में विज्ञापित करना चाहते हैं**? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT संग्रह**](https://opensea.io/collection/the-peass-family) देखें
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल हों** या मुझे **ट्विटर** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का** **अनुसरण** करें।**
* **हैकिंग ट्रिक्स को शेयर करें और PRs सबमिट करें** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **को।**

</details>

![](../../.gitbook/assets/picklerick.gif)

यह मशीन आसान श्रेणी में थी और यह काफी आसान थी।

## जांच

मैंने अपने उपकरण [**लेजियन**](https://github.com/carlospolop/legion) का उपयोग करके मशीन की जांच शुरू की:

![](<../../.gitbook/assets/image (79) (2).png>)

जैसा कि आप देख सकते हैं, 2 पोर्ट खुले हैं: 80 (**HTTP**) और 22 (**SSH**)

इसलिए, मैंने HTTP सेवा की जांच के लिए लेजियन चलाया:

![](<../../.gitbook/assets/image (234).png>)

चित्र में देखें कि `robots.txt` में तार शामिल है `Wubbalubbadubdub`

कुछ सेकंड के बाद मैंने देखा कि `disearch` ने पहले से ही क्या खोज निकाली है:

![](<../../.gitbook/assets/image (235).png>)

![](<../../.gitbook/assets/image (236).png>)

और जैसा कि आप अंतिम चित्र में देख सकते हैं, एक **लॉगिन** पृष्ठ खोजा गया था।

रूट पृष्ठ के स्रोत को जांचते हुए, एक उपयोगकर्ता खोजा गया है: `R1ckRul3s`

![](<../../.gitbook/assets/image (237) (1).png>)

इसलिए, आप `R1ckRul3s:Wubbalubbadubdub` क्रेडेंशियल का उपयोग करके लॉगिन पृष्ठ पर लॉगिन कर सकते हैं।

## उपयोगकर्ता

उन क्रेडेंशियल का उपयोग करके आप एक पोर्टल तक पहुंचेंगे जहां आप कमांड चला सकते हैं:

![](<../../.gitbook/assets/image (241).png>)

कुछ कमांड जैसे cat अनुमति नहीं देते हैं लेकिन आप पहले इंग्रीडिएंट (फ़्लैग) को उदाहरण के लिए grep का उपयोग करके पढ़ सकते हैं:

![](<../../.gitbook/assets/image (242).png>)

फिर मैंने इस्तेमाल किया:

![](<../../.gitbook/assets/image (243) (1).png>)

एक रिवर्स शेल प्राप्त करने के लिए:

![](<../../.gitbook/assets/image (239) (1).png>)

**दूसरा इंग्रीडिएंट** `/home/rick` में पाया जा सकता है

![](<../../.gitbook/assets/image (240).png>)

## रूट

उपयोगकर्ता **www-data सुडो के रूप में कुछ भी चला सकता है**:

![](<../../.gitbook/assets/image (238).png>)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ हैकट्रिक्स क्लाउड ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 ट्विटर 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ ट्विच 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 यूट्यूब 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को हैकट्रिक्स में विज्ञापित करना चाहते हैं**? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The
