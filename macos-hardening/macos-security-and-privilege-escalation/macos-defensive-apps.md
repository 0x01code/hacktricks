# macOS रक्षात्मक ऐप्स

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आप **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड** करना चाहते हैं? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT संग्रह**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स को** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **के माध्यम से PR जमा करके साझा करें।**

</details>

## फ़ायरवॉल

* [**Little Snitch**](https://www.obdev.at/products/littlesnitch/index.html): यह प्रत्येक प्रक्रिया द्वारा किए गए हर कनेक्शन का मॉनिटरिंग करेगा। मोड के आधार पर (चुप्पी से कनेक्शन की अनुमति देना, चुप्पी से कनेक्शन की अनुमति न देना और चेतावनी) यह आपको हर बार जब एक नया कनेक्शन स्थापित होता है तो **एक चेतावनी दिखाएगा**। इसके पास इस सभी जानकारी को देखने के लिए एक बहुत अच्छा GUI भी है।
* [**LuLu**](https://objective-see.org/products/lulu.html): Objective-See फ़ायरवॉल। यह एक मूलभूत फ़ायरवॉल है जो आपको संदिग्ध कनेक्शन के लिए चेतावनी देगा (इसका GUI भी है लेकिन यह Little Snitch के जैसा फैंसी नहीं है)।

## स्थायित्व पता लगाना

* [**KnockKnock**](https://objective-see.org/products/knockknock.html): Objective-See एप्लिकेशन जो वहां खोजेगा जहां **मैलवेयर स्थायित्व बना सकता है** (यह एक वन-शॉट टूल है, न कि एक मॉनिटरिंग सेवा)।
* [**BlockBlock**](https://objective-see.org/products/blockblock.html): KnockKnock की तरह, स्थायित्व उत्पन्न करने वाली प्रक्रियाओं की मॉनिटरिंग करके।

## कीलॉगर खोज

* [**ReiKey**](https://objective-see.org/products/reikey.html): Objective-See एप्लिकेशन जो **कीलॉगर्स** को खोजने के लिए इंस्टॉल करते हैंडल "इवेंट टैप" को खोजेगा।

## रैंसमवेयर खोज

* [**RansomWhere**](https://objective-see.org/products/ransomwhere.html): फ़ाइल एन्क्रिप्शन क्रियाएँ खोजने के लिए Objective-See एप्लिकेशन।

## माइक्रोफ़ोन और वेबकैम खोज

* [**OverSight**](https://objective-see.org/products/oversight.html): Objective-See एप्लिकेशन जो वेबकैम और माइक्रोफ़ोन का उपयोग करने वाले **एप्लिकेशन को खोजने के लिए**।

## प्रक्रिया इंजेक्शन खोज

* [**Shield**](https://theevilbit.github.io/shield/): विभिन्न प्रक्रिया इंजेक्शन की **खोज करने वाला एप्लिकेशन**।

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आप **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड** करना चाहते हैं? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT संग्रह**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**
