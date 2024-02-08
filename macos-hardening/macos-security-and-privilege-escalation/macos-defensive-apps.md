# macOS रक्षात्मक ऐप्स

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सदस्यता योजनाएं देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारा संग्रह [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** द्वारा PRs सबमिट करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>

## फ़ायरवॉल

* [**Little Snitch**](https://www.obdev.at/products/littlesnitch/index.html): यह प्रत्येक प्रक्रिया द्वारा किए गए प्रत्येक कनेक्शन का मॉनिटर करेगा। मोड के आधार पर (चुप अनुमति कनेक्शन, चुप इनकार कनेक्शन और अलर्ट) यह हर बार एक नया कनेक्शन स्थापित होने पर **आपको एक अलर्ट दिखाएगा**। इसमें इस सभी जानकारी को देखने के लिए एक बहुत अच्छा GUI भी है।
* [**LuLu**](https://objective-see.org/products/lulu.html): Objective-See फ़ायरवॉल। यह एक मौलिक फ़ायरवॉल है जो संदेहपूर्ण कनेक्शन के लिए आपको अलर्ट करेगा (इसमें एक GUI है लेकिन यह Little Snitch की तरह फैंसी नहीं है)।

## स्थिरता पता लगाना

* [**KnockKnock**](https://objective-see.org/products/knockknock.html): Objective-See एप्लिकेशन जो ढूंढेगा कि **मैलवेयर कहाँ स्थिर हो सकता है** (यह एक वन-शॉट टूल है, न कि एक मॉनिटरिंग सेवा)।
* [**BlockBlock**](https://objective-see.org/products/blockblock.html): KnockKnock की तरह, स्थिरता उत्पन्न करने वाली प्रक्रियाओं को मॉनिटर करके।

## कीलॉगर्स पता लगाना

* [**ReiKey**](https://objective-see.org/products/reikey.html): Objective-See एप्लिकेशन जो **कीलॉगर्स** को खोजने में मदद करेगा जो कीबोर्ड "इवेंट टैप्स" को स्थापित करते हैं।

## रैंसमवेयर पता लगाना

* [**RansomWhere**](https://objective-see.org/products/ransomwhere.html): Objective-See एप्लिकेशन जो **फ़ाइल एन्क्रिप्शन** क्रियाओं को पहचानने में मदद करेगा।

## माइक और वेबकैम पता लगाना

* [**OverSight**](https://objective-see.org/products/oversight.html): Objective-See एप्लिकेशन जो **ऐप्लिकेशन को पहचानने में मदद करेगा जो वेबकैम और माइक का उपयोग करना शुरू करता है।**

## प्रक्रिया इंजेक्शन पता लगाना

* [**Shield**](https://theevilbit.github.io/shield/): विभिन्न प्रक्रिया इंजेक्शन की **पहचान करता है**।
