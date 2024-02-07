# मेमोरी डंप विश्लेषण

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ हैकट्रिक्स क्लाउड ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 ट्विटर 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ ट्विच 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 यूट्यूब 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप चाहते हैं कि आपकी **कंपनी हैकट्रिक्स में विज्ञापित हो**? या क्या आप **PEASS के नवीनतम संस्करण या हैकट्रिक्स को पीडीएफ में डाउनलोड** करना चाहते हैं? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**द पीएस फैमिली**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**एनएफटी**](https://opensea.io/collection/the-peass-family) संग्रह देखें
* [**आधिकारिक PEASS और हैकट्रिक्स स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **ट्विटर** **🐦**[**@carlospolopm**](https://twitter.com/hacktricks_live)** का** **पालन** करें।
* **हैकिंग ट्रिक्स साझा करें, [हैकट्रिक्स रेपो](https://github.com/carlospolop/hacktricks) और [हैकट्रिक्स-क्लाउड रेपो](https://github.com/carlospolop/hacktricks-cloud)** में पीआर जमा करके।

</details>

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

[**RootedCON**](https://www.rootedcon.com/) **स्पेन** में सबसे महत्वपूर्ण साइबर सुरक्षा घटना है और **यूरोप** में सबसे महत्वपूर्ण में से एक है। **तकनीकी ज्ञान को बढ़ावा देने** के मिशन के साथ, यह सम्मेलन प्रौद्योगिकी और साइबर सुरक्षा विशेषज्ञों के लिए एक उफान मिलने का समारोह है।

{% embed url="https://www.rootedcon.com/" %}

## प्रारंभ

पीकैप के अंदर **मैलवेयर** खोजना शुरू करें। [**मैलवेयर विश्लेषण**](../malware-analysis.md) में उल्लिखित **उपकरणों** का उपयोग करें।

## [Volatility](../../../generic-methodologies-and-resources/basic-forensic-methodology/memory-dump-analysis/volatility-cheatsheet.md)

**Volatility मेमोरी डंप विश्लेषण के लिए मुख्य ओपन-सोर्स फ्रेमवर्क है**। यह Python टूल बाह्य स्रोतों या VMware VM से डंप का विश्लेषण करता है, जिसमें डंप के ओएस प्रोफ़ाइल के आधार पर प्रक्रियाएँ और पासवर्ड जैसे डेटा की पहचान करता है। यह प्लगइन के साथ विस्तारणयोग्य है, जिससे यह फोरेंसिक जांचों के लिए अत्यधिक विविध है।

**[यहाँ एक चीटशीट पाएं](../../../generic-methodologies-and-resources/basic-forensic-methodology/memory-dump-analysis/volatility-cheatsheet.md)**

## मिनी डंप क्रैश रिपोर्ट

जब डंप छोटा होता है (कुछ KB, शायद कुछ MB) तो यह शायद एक मिनी डंप क्रैश रिपोर्ट होता है और न कि एक मेमोरी डंप।

![](<../../../.gitbook/assets/image (216).png>)

अगर आपके पास विजुअल स्टूडियो स्थापित है, तो आप इस फ़ाइल को खोल सकते हैं और प्रक्रिया का नाम, वास्तुकला, अपवाद जानकारी और मॉड्यूल जैसी कुछ मूल जानकारी जोड़ सकते हैं:

![](<../../../.gitbook/assets/image (217).png>)

आप अपवाद को भी लोड कर सकते हैं और डीकॉम्पाइल की गई निर्देशिकाएँ देख सकते हैं

![](<../../../.gitbook/assets/image (219).png>)

![](<../../../.gitbook/assets/image (218) (1).png>)

वैसे, विजुअल स्टूडियो डंप की गहराई का विश्लेषण करने के लिए सर्वश्रेष्ठ उपकरण नहीं है।

आपको इसे **आईडीए** या **राडारे** का उपयोग करके गहराई में जांचने की चाहिए।

​

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

[**RootedCON**](https://www.rootedcon.com/) **स्पेन** में सबसे महत्वपूर्ण साइबर सुरक्षा घटना है और **यूरोप** में सबसे महत्वपूर्ण में से एक है। **तकनीकी ज्ञान को बढ़ावा देने** के मिशन के साथ, यह सम्मेलन प्रौद्योगिकी और साइबर सुरक्षा विशेषज्ञों के लिए एक उफान मिलने का समारोह है।

{% embed url="https://www.rootedcon.com/" %}

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ हैकट्रिक्स क्लाउड ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 ट्विटर 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ ट्विच 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 यूट्यूब 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप चाहते हैं कि आपकी **कंपनी हैकट्रिक्स में विज्ञापित हो**? या क्या आप **PEASS के नवीनतम संस्करण या हैकट्रिक्स को पीडीएफ में डाउनलोड** करना चाहते हैं? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**द पीएस फैमिली**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**एनएफटी**](https://opensea.io/collection/the-peass-family) संग्रह देखें
* [**आधिकारिक PEASS और हैकट्रिक्स स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **ट्विटर** **🐦**[**@carlospolopm**](https://twitter.com/hacktricks_live)** का** **पालन** करें।
* **हैकिंग ट्रिक्स साझा करें, [हैकट्रिक्स रेपो](https://github.com/carlospolop/hacktricks) और [हैकट्रिक्स-क्लाउड रेपो](https://github.com/carlospolop/hacktricks-cloud)** में पीआर जमा करके।

</details>
