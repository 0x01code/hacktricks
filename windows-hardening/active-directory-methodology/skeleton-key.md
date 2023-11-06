# स्केलेटन की

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ हैकट्रिक्स क्लाउड ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 ट्विटर 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ ट्विच 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 यूट्यूब 🎥</strong></a></summary>

- क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **हैकट्रिक्स में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**द पीएस फैमिली**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)

- प्राप्त करें [**आधिकारिक पीएस और हैकट्रिक्स स्वैग**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **ट्विटर** पर **फॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **हैकिंग ट्रिक्स साझा करें और PRs सबमिट करें [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में।**

</details>

## **स्केलेटन की**

**स्रोत:** [**https://blog.stealthbits.com/unlocking-all-the-doors-to-active-directory-with-the-skeleton-key-attack/**](https://blog.stealthbits.com/unlocking-all-the-doors-to-active-directory-with-the-skeleton-key-attack/)

एक्टिव डायरेक्टरी खातों को संक्रमित करने के लिए कई तरीके हैं जिन्हें हमलावर उच्चतम अधिकार प्राप्त करने और स्थायित्व बनाने के लिए उपयोग कर सकते हैं जब वे अपने डोमेन में स्थापित हो गए हों। स्केलेटन की एक विशेष रूप से डायरेक्टरी डोमेनों के लिए निर्दिष्ट किया गया है ताकि किसी भी खाते को हथिया सकना चिंताजनक आसान हो जाए। यह मैलवेयर खुद को LSASS में इंजेक्ट करता है और एक मास्टर पासवर्ड बनाता है जो डोमेन में किसी भी खाते के लिए काम करेगा। मौजूदा पासवर्ड भी काम करेंगे, इसलिए इस हमले का पता लगाना बहुत मुश्किल होता है जब तक आप जानते हों कि इसकी तलाश क्या है।

जैसा कि अपेक्षित था, यह उन कई हमलों में से एक है जिसे [Mimikatz](https://github.com/gentilkiwi/mimikatz) का उपयोग करके पैकेज किया जाता है और बहुत ही आसानी से किया जा सकता है। चलो देखते हैं कि यह कैसे काम करता है।

### स्केलेटन की हमले के लिए आवश्यकताएं

इस हमले को करने के लिए, **हमलावर को डोमेन व्यवस्थापक अधिकार** होने चाहिए। इस हमले को पूरी तरह से संपूर्ण करने के लिए इसे **प्रत्येक डोमेन नियंत्रक पर किया जाना चाहिए**, लेकिन केवल एक डोमेन नियंत्रक को लक्ष्य बनाना भी प्रभावी हो सकता है। **डोमेन नियंत्रक को रिबूट** करने से यह मैलवेयर हट जाएगा और हमलावर द्वारा पुनः लागू किया जाना होगा।

### स्केलेटन की हमला करना

हमला करना बहुत सरल है। इसके लिए केवल निम्नलिखित **कमांड को प्रत्येक डोमेन नियंत्रक पर चलाना** होगा: `misc::skeleton`। इसके बाद, आप Mimikatz के डिफ़ॉल्ट पासवर्ड के साथ किसी भी उपयोगकर्ता के रूप में प्रमाणित कर सकते हैं।

![Mimikatz के साथ misc::skeleton का उपयोग करके एक स्केलेटन की को डोमेन नियंत्रक में इ
- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**।**

- **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके साझा करें।**
