# मेमोरी डंप विश्लेषण

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने** की उपलब्धता चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह देखें
* [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल हों** या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)** का** अनुसरण करें।
* **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके अपना योगदान दें।**

</details>

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

[**RootedCON**](https://www.rootedcon.com/) स्पेन में सबसे महत्वपूर्ण साइबर सुरक्षा इवेंट है और यूरोप में सबसे महत्वपूर्ण में से एक है। तकनीकी ज्ञान को बढ़ावा देने की मिशन के साथ, यह कांग्रेस प्रौद्योगिकी और साइबर सुरक्षा विशेषज्ञों के लिए एक उबलता हुआ मिलन स्थल है।

{% embed url="https://www.rootedcon.com/" %}

## प्रारंभ

पीकैप में **मैलवेयर** खोजना शुरू करें। [**मैलवेयर विश्लेषण**](../malware-analysis.md) में उल्लिखित **उपकरण** का उपयोग करें।

## [Volatility](../../../generic-methodologies-and-resources/basic-forensic-methodology/memory-dump-analysis/volatility-cheatsheet.md)

मेमोरी डंप विश्लेषण के लिए प्रीमियर ओपन-सोर्स फ्रेमवर्क है [Volatility](../../../generic-methodologies-and-resources/basic-forensic-methodology/memory-dump-analysis/volatility-cheatsheet.md)। Volatility एक Python स्क्रिप्ट है जो बाहरी उपकरण (या वीएमवेयर मेमोरी इमेज जो वीएम को रोककर इकट्ठा किया गया है) के साथ इकट्ठा किए गए मेमोरी डंप को पार्स करने के लिए है। तो, मेमोरी डंप फ़ाइल और प्रासंगिक "प्रोफ़ाइल" (जिससे डंप इकट्ठा किया गया था वह ओएस), Volatility डेटा में संरचनाएँ पहचानना शुरू कर सकता है: चल रहे प्रक्रियाएँ, पासवर्ड, आदि। यह भी विभिन्न प्लगइन का उपयोग करके विभिन्न प्रकार के आर्टिफैक्ट निकालने के लिए विस्तारयोग्य है।\
स्रोत: [https://trailofbits.github.io/ctf/forensics/](https://trailofbits.github.io/ctf/forensics/)

## मिनी डंप क्रैश रिपोर्ट

जब डंप छोटा होता है (कुछ KB, शायद कुछ MB) तो यह शायद एक मिनी डंप क्रैश रिपोर्ट होता है और न कि मेमोरी डंप।

![](<../../../.gitbook/assets/image (216).png>)

यदि आपके पास Visual Studio स्थापित है, तो आप इस फ़ाइल को खोल सकते हैं और कुछ मूलभूत जानकारी जैसे प्रक्रिया का नाम, आर्किटेक्चर, अपवाद जानकारी और मॉड्यूल्स को बाइंड कर सकते हैं:

![](<../../../.gitbook/assets/image (217).png>)

आप अपवाद को भी लोड कर सकते हैं और
* **अपनी हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके साझा करें।**
