<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने की अनुमति** चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)

- प्राप्त करें [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में PR जमा करके साझा करें।**

</details>


#

# JTAG

JTAG एक बाउंडरी स्कैन को करने की अनुमति देता है। बाउंडरी स्कैन निम्नलिखित प्रत्येक पिन के लिए सम्मिलित बाउंडरी-स्कैन सेल और रजिस्टर सहित निश्चित सर्किट्री का विश्लेषण करता है।

JTAG मानक निम्नलिखित विशेष आदेशों को परिभाषित करता है जो बाउंडरी स्कैन का आयोजन करने के लिए होते हैं, जिनमें निम्नलिखित शामिल हैं:

* **BYPASS** आपको अन्य चिप्स के माध्यम से जाने के बिना एक विशिष्ट चिप का परीक्षण करने की अनुमति देता है।
* **SAMPLE/PRELOAD** यह उस समय डेटा का एक सैंपल लेता है जब यह अपने सामान्य कार्यान्वयन मोड में होता है।
* **EXTEST** पिन स्थितियों को सेट और पढ़ता है।

इसके अलावा, यह अन्य आदेशों का समर्थन भी कर सकता है जैसे:

* उपकरण की पहचान के लिए **IDCODE**
* उपकरण के आंतरिक परीक्षण के लिए **INTEST**

आप JTAGulator जैसे उपकरण का उपयोग करते हैं तो आपको इन निर्देशों के साथ संपर्क कर सकते हैं।

## परीक्षण पहुंच पोर्ट

बाउंडरी स्कैन में चार-तार टेस्ट पहुंच (**TAP**) की जांच शामिल होती है, जो एक सामान्य उद्देश्य पोर्ट है जो एक कंपोनेंट में बने JTAG परीक्षण समर्थन के लिए पहुंच प्रदान करता है। TAP निम्नलिखित पांच सिग्नल का उपयोग करता है:

* परीक्षण घड़ी की इनपुट (**TCK**) TCK एक **घड़ी** है जो निर्धारित करती है कि TAP नियंत्रक कितनी बार एकल क्रिया करेगा (अर्थात्, स्थिति मशीन में अगले स्थिति में छलांग लगाएगा)।
* परीक्षण मोड चयन (**TMS**) इनपुट TMS निर्धारित करता है कि **सीमित स्थिति मशीन** को कंट्रोल करेगा। प्रत्येक घड़ी की ध्वनि पर यह जांचता है कि उपकरण के JTAG TAP नियंत्रक ने TMS पिन पर वोल्टेज की जांच की है। यदि वोल्टेज निश्चित थ्रेशोल्ड से कम है, तो सिग्नल को कम माना जाता है और 0 के रूप में व्याख्या की जाती है, जबकि वोल्टेज निश्चित थ्रेशोल्ड से अधिक होने पर सिग्नल को उच्च माना जाता है और 1 के
- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**।**

- **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके साझा करें।**
