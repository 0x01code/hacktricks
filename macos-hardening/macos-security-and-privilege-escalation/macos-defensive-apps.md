# macOS डिफेंसिव ऐप्स

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert) के साथ शून्य से हीरो तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपोज़ में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>

## फ़ायरवॉल्स

* [**Little Snitch**](https://www.obdev.at/products/littlesnitch/index.html): यह हर प्रोसेस द्वारा किए गए हर कनेक्शन की निगरानी करेगा। मोड के आधार पर (साइलेंट अलाउ कनेक्शन्स, साइलेंट डिनाई कनेक्शन और अलर्ट) यह आपको एक अलर्ट **दिखाएगा** जब भी एक नया कनेक्शन स्थापित होता है। इसमें इस सभी जानकारी को देखने के लिए एक बहुत अच्छा GUI भी है।
* [**LuLu**](https://objective-see.org/products/lulu.html): Objective-See फ़ायरवॉल। यह एक बेसिक फ़ायरवॉल है जो आपको संदिग्ध कनेक्शन्स के लिए अलर्ट करेगा (इसमें GUI है लेकिन यह Little Snitch के जितना फैंसी नहीं है)।

## पर्सिस्टेंस डिटेक्शन

* [**KnockKnock**](https://objective-see.org/products/knockknock.html): Objective-See एप्लिकेशन जो कई लोकेशन्स में खोज करेगा जहां **मैलवेयर पर्सिस्ट हो सकता है** (यह एक वन-शॉट टूल है, मॉनिटरिंग सर्विस नहीं)।
* [**BlockBlock**](https://objective-see.org/products/blockblock.html): KnockKnock की तरह पर्सिस्टेंस जनरेट करने वाले प्रोसेसेस की मॉनिटरिंग करके।

## कीलॉगर्स डिटेक्शन

* [**ReiKey**](https://objective-see.org/products/reikey.html): Objective-See एप्लिकेशन जो **कीलॉगर्स** का पता लगाएगा जो कीबोर्ड "इवेंट टैप्स" स्थापित करते हैं।

## रैन्समवेयर डिटेक्शन

* [**RansomWhere**](https://objective-see.org/products/ransomwhere.html): Objective-See एप्लिकेशन जो **फ़ाइल एन्क्रिप्शन** क्रियाओं का पता लगाएगा।

## माइक & वेबकैम डिटेक्शन

* [**OverSight**](https://objective-see.org/products/oversight.html): Objective-See एप्लिकेशन जो पता लगाएगा **एप्लिकेशन जो वेबकैम और माइक का उपयोग शुरू करते हैं।**

## प्रोसेस इंजेक्शन डिटेक्शन

* [**Shield**](https://theevilbit.github.io/shield/): एप्लिकेशन जो **विभिन्न प्रोसेस इंजेक्शन** तकनीकों का पता लगाता है।

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert) के साथ शून्य से हीरो तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपोज़ में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>
