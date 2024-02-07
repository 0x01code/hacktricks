<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि **आपकी कंपनी HackTricks में विज्ञापित हो** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)** पर **फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>


# मूल्यांकन

एक मूल्यांकन में एक सिस्टम के कुछ हिस्सों का एक स्नैपशॉट लेना शामिल है ताकि **बदलावों को हाइलाइट करने के लिए एक भविष्य की स्थिति के साथ तुलना की जा सके**।

उदाहरण के लिए, आप फ़ाइल सिस्टम की प्रत्येक फ़ाइल का हैश कैलकुलेट और स्टोर कर सकते हैं ताकि आप पता लगा सकें कि कौन सी फ़ाइलें संशोधित की गई थीं।\
इसे उपयोगकर्ता खाते बनाए गए, प्रक्रियाएँ चल रही हैं, सेवाएँ चल रही हैं और किसी भी अन्य चीज़ के साथ भी किया जा सकता है जो बहुत अधिक या पूरी तरह से बदलना नहीं चाहिए।

## फ़ाइल अखंडता मॉनिटरिंग

फ़ाइल अखंडता मॉनिटरिंग (FIM) एक महत्वपूर्ण सुरक्षा तकनीक है जो फ़ाइलों में परिवर्तनों का ट्रैक करके IT वातावरण और डेटा की सुरक्षा करती है। इसमें दो मुख्य कदम शामिल हैं:

1. **मूल्यांकन तुलना:** भविष्य की तुलना के लिए फ़ाइल गुणांक या एन्क्रिप्टेड चेकसम्स (जैसे MD5 या SHA-2) का उपयोग करके एक मूल्यांकन स्थापित करें ताकि संशोधनों का पता लगा सकें।
2. **रियल-टाइम चेंज नोटिफिकेशन:** फ़ाइलों तक पहुंचने या संशोधित किए जाने पर तुरंत अलर्ट प्राप्त करें, आम तौर पर OS कर्नल एक्सटेंशन के माध्यम से।

## उपकरण

* [https://github.com/topics/file-integrity-monitoring](https://github.com/topics/file-integrity-monitoring)
* [https://www.solarwinds.com/security-event-manager/use-cases/file-integrity-monitoring-software](https://www.solarwinds.com/security-event-manager/use-cases/file-integrity-monitoring-software)

# संदर्भ

* [https://cybersecurity.att.com/blogs/security-essentials/what-is-file-integrity-monitoring-and-why-you-need-it](https://cybersecurity.att.com/blogs/security-essentials/what-is-file-integrity-monitoring-and-why-you-need-it)


<details>
