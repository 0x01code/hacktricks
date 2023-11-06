<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने की अनुमति** चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह

- [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में PR जमा करके साझा करें**.

</details>


# मूल रेखा

मूल रेखा में किसी सिस्टम के निशानीबद्ध भागों की एक स्नैपशॉट लेने से मिलती है, जिसे भविष्य की स्थिति के साथ तुलना करने के लिए उपयोग किया जाता है ताकि परिवर्तनों को हाइलाइट किया जा सके।

उदाहरण के लिए, आप फ़ाइल सिस्टम की प्रत्येक फ़ाइल के हैश की गणना करके और संग्रहीत करके कर सकते हैं ताकि आप पता लगा सकें कि कौन सी फ़ाइलें संशोधित की गई हैं।\
इसे उपयोग करके आप उपयोगकर्ता खाते, चल रहे प्रक्रियाएँ, चल रहे सेवाएँ और किसी भी अन्य चीज़ की भी स्नातक बना सकते हैं, जो बहुत या पूरी तरह से बदलना चाहिए।

## फ़ाइल अखंडता मॉनिटरिंग

फ़ाइल अखंडता मॉनिटरिंग सबसे शक्तिशाली तकनीकों में से एक है जो आईटी इंफ्रास्ट्रक्चर और व्यापार डेटा को जाने-अनजाने खतरों से सुरक्षित करने के लिए उपयोग की जाती है।\
इसका उद्देश्य है एक **मूल रेखा उत्पन्न करना** जिसमें आप सभी फ़ाइलों की जांच करना चाहते हैं और फिर नियमित अंतराल पर उन फ़ाइलों की जांच करना जिसमें **परिवर्तन** हो सकता है (सामग्री, गुण, मेटाडेटा आदि में)।

1\. **मूल रेखा तुलना**, जिसमें एक या एक से अधिक फ़ाइल गुणों को कैप्चर या गणना किया जाएगा और एक मूल रेखा के रूप में संग्रहीत किया जाएगा जिसकी तुलना में भविष्य में की जा सकती है। यह फ़ाइल के समय और तारीख के रूप में भी सरल हो सकता है, हालांकि, क्योंकि यह डेटा आसानी से जाली बनाया जा सकता है, इसलिए आमतौर पर एक अधिक विश्वसनीय दृष्टिकोण का उपयोग किया जाता है। इसमें नियमित अंतराल पर एक मॉनिटर की गणना के लिए एक फ़ाइल के लिए औद्योगिक चेकसम का मूल्यांकन करना शामिल हो सकता है, (जैसे MD5 या SHA-2 हैश एल्गोरिदम का उपयोग करके) और फिर पहले से गणना किए गए चेकसम के साथ परिणाम की तुलना करना।

2\. **रीयल-टाइम परिवर्तन सूचना**, जो आमतौर पर ऑपरेटिंग सिस्टम के कर्नल के रूप में अंतर्निहित या एक विस्तार के रूप में लागू की जाती है जो एक फ़ाइल तक पहुंच या संशोधित होने पर झंडा लगाएगा।

## उपकरण

* [https://github.com/topics/file-integrity-monitoring](https://github.com/topics/file-integrity-monitoring)
* [https://www.solarwinds.com/security-event-manager/use-cases/file-integrity-monitoring-software](https://www.solarwinds.com/security-event-manager/use-cases/file-integrity-monitoring-software)

# स
