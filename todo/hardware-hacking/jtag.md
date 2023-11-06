<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT संग्रह**](https://opensea.io/collection/the-peass-family)

- [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में PR जमा करके साझा करें**.

</details>


# JTAGenum

[**JTAGenum** ](https://github.com/cyphunk/JTAGenum) एक टूल है जिसे रैस्पबेरी पाई या आर्डुइनो के साथ उपयोग किया जा सकता है ताकि एक अज्ञात चिप से JTAG पिन की कोशिश की जा सके।\
**आर्डुइनो** में, **2 से 11 पिन को JTAG पिन्स के 10 पिन्स से कनेक्ट करें**। आर्डुइनो में प्रोग्राम लोड करें और यह सभी पिन्स को bruteforce करने के लिए प्रयास करेगा ताकि पता लगा सकें कि कौन सा पिन JTAG का है और प्रत्येक पिन का नाम क्या है।\
**रैस्पबेरी पाई** में आप केवल **1 से 6 पिन्स** (6 पिन्स, इसलिए आप हर पोटेंशियल JTAG पिन की परीक्षण करने के लिए धीमे हो जाएंगे) का उपयोग कर सकते हैं।

## आर्डुइनो

आर्डुइनो में, केबल जोड़ने के बाद (पिन 2 से 11 को JTAG पिन्स और आर्डुइनो GND को बेसबोर्ड GND से कनेक्ट करें), **आर्डुइनो में JTAGenum प्रोग्राम लोड करें** और Serial Monitor में **`h`** (सहायता के लिए कमांड) भेजें और आपको सहायता दिखाई देनी चाहिए:

![](<../../.gitbook/assets/image (643).png>)

![](<../../.gitbook/assets/image (650).png>)

**"No line ending" और 115200baud** को कॉन्फ़िगर करें।\
स्कैनिंग शुरू करने के लिए कमांड s भेजें:

![](<../../.gitbook/assets/image (651) (1) (1) (1).png>)

यदि आप JTAG से संपर्क कर रहे हैं, तो आपको एक या एक से अधिक **FOUND!** से शुरू होने वाली पंक्तियाँ मिलेंगी जो JTAG के पिन्स को दर्शाती हैं।


<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT संग्रह**](https://opensea.io/collection/the-peass-family)

- [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में PR जमा करके साझा करें**.

</details>
