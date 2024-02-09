# मेमोरी डंप विश्लेषण

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert) के साथ जीरो से हीरो तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आप **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड** करना चाहते हैं? [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) की जाँच करें!
* हमारे [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें
* [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **Twitter** पर **फॉलो** करें 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **हैकिंग ट्रिक्स साझा करें** [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके।

</details>

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

[**RootedCON**](https://www.rootedcon.com/) **स्पेन** में सबसे महत्वपूर्ण साइबर सुरक्षा घटना है और **यूरोप** में सबसे महत्वपूर्ण में से एक है। **तकनीकी ज्ञान को बढ़ावा देने** के मिशन के साथ, यह कांग्रेस प्रौद्योगिकी और साइबर सुरक्षा विशेषज्ञों के लिए एक उफान मिलने का समारोह है।

{% embed url="https://www.rootedcon.com/" %}

## शुरुआत

पीकैप के अंदर **मैलवेयर** खोजना शुरू करें। [**मैलवेयर विश्लेषण**](../malware-analysis.md) में उल्लिखित **उपकरणों** का उपयोग करें।

## [Volatility](../../../generic-methodologies-and-resources/basic-forensic-methodology/memory-dump-analysis/volatility-cheatsheet.md)

**Volatility मेमोरी डंप विश्लेषण के लिए मुख्य ओपन-सोर्स फ्रेमवर्क है**। यह Python टूल बाह्य स्रोतों या VMware VMs से डंप का विश्लेषण करता है, जिसमें डंप के ओएस प्रोफ़ाइल के आधार पर प्रक्रियाएँ और पासवर्ड जैसे डेटा की पहचान करता है। यह प्लगइन के साथ विस्तारणयोग्य है, जिससे यह फोरेंसिक जांचों के लिए बहुत उपयुक्त है।

**[यहाँ एक cheatsheet पाएं](../../../generic-methodologies-and-resources/basic-forensic-methodology/memory-dump-analysis/volatility-cheatsheet.md)**

## मिनी डंप क्रैश रिपोर्ट

जब डंप छोटा होता है (कुछ KB, शायद कुछ MB) तो यह शायद एक मिनी डंप क्रैश रिपोर्ट होता है और मेमोरी डंप नहीं।

![](<../../../.gitbook/assets/image (216).png>)

अगर आपके पास विजुअल स्टूडियो इंस्टॉल किया है, तो आप इस फ़ाइल को खोल सकते हैं और कुछ मूल जानकारी जैसे प्रक्रिया का नाम, वास्तुकला, असामान्य जानकारी और मॉड्यूल जो कार्यान्वित हो रहे हैं, बाँध सकते हैं:

![](<../../../.gitbook/assets/image (217).png>)

आप असामान्यता भी लोड कर सकते हैं और डीकंपाइल की गई निर्देशिकाएँ देख सकते हैं

![](<../../../.gitbook/assets/image (219).png>)

![](<../../../.gitbook/assets/image (218) (1).png>)

वैसे, विजुअल स्टूडियो डंप की गहराई का विश्लेषण करने के लिए सर्वश्रेष्ठ उपकरण नहीं है।

आपको इसे **IDA** या **Radare** का उपयोग करके गहराई से जांचने की चाहिए।

​

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

[**RootedCON**](https://www.rootedcon.com/) **स्पेन** में सबसे महत्वपूर्ण साइबर सुरक्षा घटना है और **यूरोप** में सबसे महत्वपूर्ण में से एक है। **तकनीकी ज्ञान को बढ़ावा देने** के मिशन के साथ, यह कांग्रेस प्रौद्योगिकी और साइबर सुरक्षा विशेषज्ञों के लिए एक उफान मिलने का समारोह है।

{% embed url="https://www.rootedcon.com/" %}

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert) के साथ जीरो से हीरो तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आप **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड** करना चाहते हैं? [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) की जाँच करें!
* हमारे [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें
* [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **Twitter** पर **फॉलो** करें 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **हैकिंग ट्रिक्स साझा करें** [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके।

</details>
