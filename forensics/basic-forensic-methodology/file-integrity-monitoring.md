<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **अपनी हैकिंग ट्रिक्स साझा करें PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में.

</details>


# बेसलाइन

बेसलाइन का मतलब है किसी सिस्टम के कुछ हिस्सों का स्नैपशॉट लेना ताकि **भविष्य की स्थिति के साथ तुलना करके परिवर्तनों को उजागर किया जा सके**.

उदाहरण के लिए, आप फाइलसिस्टम की प्रत्येक फाइल का हैश कैलकुलेट करके स्टोर कर सकते हैं ताकि पता चल सके कि कौन सी फाइलें संशोधित की गई हैं.\
यह उपयोगकर्ता खातों के साथ भी किया जा सकता है जो बनाए गए हैं, चल रही प्रक्रियाएं, चल रही सेवाएं और कोई भी अन्य चीज जो ज्यादा नहीं बदलनी चाहिए, या बिल्कुल भी नहीं.

## फाइल इंटेग्रिटी मॉनिटरिंग

फाइल इंटेग्रिटी मॉनिटरिंग IT इंफ्रास्ट्रक्चर और व्यापार डेटा को विभिन्न प्रकार के ज्ञात और अज्ञात खतरों से सुरक्षित करने के लिए प्रयोग की जाने वाली सबसे शक्तिशाली तकनीकों में से एक है.\
लक्ष्य है उन सभी फाइलों का एक **बेसलाइन उत्पन्न करना** जिन्हें आप मॉनिटर करना चाहते हैं और फिर **समय-समय पर** उन फाइलों की **जांच** करना संभावित **परिवर्तनों** के लिए (सामग्री, गुण, मेटाडेटा, आदि में).

1\. **बेसलाइन तुलना,** जिसमें एक या अधिक फाइल गुणों को कैप्चर या कैलकुलेट किया जाएगा और बेसलाइन के रूप में संग्रहीत किया जाएगा जिसे भविष्य में तुलना की जा सकती है. यह फाइल की समय और तारीख जितना सरल हो सकता है, हालांकि, चूंकि यह डेटा आसानी से स्पूफ किया जा सकता है, इसलिए आमतौर पर एक अधिक विश्वसनीय दृष्टिकोण प्रयोग किया जाता है. इसमें निगरानी की जा रही फाइल के लिए क्रिप्टोग्राफिक चेकसम का आवधिक मूल्यांकन शामिल हो सकता है, (उदाहरण के लिए MD5 या SHA-2 हैशिंग एल्गोरिदम का उपयोग करके) और फिर पहले से कैलकुलेट किए गए चेकसम के साथ परिणाम की तुलना करना.

2\. **रियल-टाइम परिवर्तन सूचना**, जो आमतौर पर ऑपरेटिंग सिस्टम के कर्नेल के भीतर या एक्सटेंशन के रूप में लागू की जाती है जो यह संकेत देगी जब कोई फाइल एक्सेस या संशोधित की जाती है.

## उपकरण

* [https://github.com/topics/file-integrity-monitoring](https://github.com/topics/file-integrity-monitoring)
* [https://www.solarwinds.com/security-event-manager/use-cases/file-integrity-monitoring-software](https://www.solarwinds.com/security-event-manager/use-cases/file-integrity-monitoring-software)

# संदर्भ

* [https://cybersecurity.att.com/blogs/security-essentials/what-is-file-integrity-monitoring-and-why-you-need-it](https://cybersecurity.att.com/blogs/security-essentials/what-is-file-integrity-monitoring-and-why-you-need-it)


<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **अपनी हैकिंग ट्रिक्स साझा करें PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में.

</details>
