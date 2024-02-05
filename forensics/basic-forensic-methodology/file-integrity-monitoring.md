<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप चाहते हैं कि **आपकी कंपनी HackTricks में विज्ञापित हो** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** पर **फॉलो** करें 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें** और PRs सबमिट करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>


# बेसलाइन

एक बेसलाइन में एक सिस्टम के कुछ हिस्सों का एक स्नैपशॉट लेना शामिल है ताकि इसे भविष्य की स्थिति के साथ तुलना करने के लिए परिवर्तनों को हाइलाइट किया जा सके।

उदाहरण के लिए, आप फाइल सिस्टम की प्रत्येक फ़ाइल का हैश की गणना करके और स्टोर करके उसे संशोधित किए गए फ़ाइलें पता लगाने के लिए संचित कर सकते हैं।\
यह उपयोगकर्ता खाते बनाए गए, प्रक्रियाएँ चल रही हैं, सेवाएँ चल रही हैं और किसी भी अन्य चीज़ के साथ भी किया जा सकता है जो बहुत अधिक परिवर्तित नहीं होना चाहिए, या बिल्कुल नहीं।

## फ़ाइल अखंडता मॉनिटरिंग

फ़ाइल अखंडता मॉनिटरिंग एक ऐसा शक्तिशाली तकनीक है जो कई प्रकार के जाने और अज्ञात खतरों के खिलाफ IT ढांचे और व्यावसायिक डेटा को सुरक्षित रखने के लिए उपयोग किया जाता है।\
उद्देश्य एक **बेसलाइन उत्पन्न करना** है जिसमें आप जिन फ़ाइलों को मॉनिटर करना चाहते हैं उन सभी फ़ाइलों का एक **बेसलाइन** बनाना और फिर **नियमित अंतराल** पर उन फ़ाइलों की **संभावित परिवर्तनों** की जांच करना है (सामग्री, गुणवत्ता, मेटाडेटा आदि में).

1\. **बेसलाइन तुलना,** जिसमें एक या एक से अधिक फ़ाइल गुणवत्ताएँ कैप्चर या गणित की जाएंगी और एक बेसलाइन के रूप में स्टोर की जाएंगी जिसे भविष्य में तुलना के लिए तुलना की जा सकती है। यह फ़ाइल के समय और तारीख के रूप में साधारण हो सकता है, हालांकि, क्योंकि यह डेटा आसानी से धोखा दे सकता है, इसके लिए आमतौर पर एक अधिक विश्वसनीय दृष्टिकोण का उपयोग किया जाता है। इसमें नियमित अंतराल पर मॉनिटर की गई फ़ाइल के लिए एक और अद्वितीय चेकसम का मूल्यांकन करना शामिल हो सकता है, (जैसे MD5 या SHA-2 हैशिंग एल्गोरिथ्म का उपयोग करके) और फिर पिछले गणना किए गए चेकसम के साथ परिणाम की तुलना करना।

2\. **रियल-टाइम परिवर्तन सूचना,** जो आमतौर पर ऑपरेटिंग सिस्टम के कर्नल के भीतर या एक विस्तार के रूप में कार्यान्वित किया जाता है जो जब एक फ़ाइल तक पहुंचा जाता है या संशोधित किया जाता है तो झंडा दिखाएगा।

## उपकरण

* [https://github.com/topics/file-integrity-monitoring](https://github.com/topics/file-integrity-monitoring)
* [https://www.solarwinds.com/security-event-manager/use-cases/file-integrity-monitoring-software](https://www.solarwinds.com/security-event-manager/use-cases/file-integrity-monitoring-software)

# संदर्भ

* [https://cybersecurity.att.com/blogs/security-essentials/what-is-file-integrity-monitoring-and-why-you-need-it](https://cybersecurity.att.com/blogs/security-essentials/what-is-file-integrity-monitoring-and-why-you-need-it)


<details>
