# एंड्रॉइड फोरेंसिक्स

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ हैकट्रिक्स क्लाउड ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 ट्विटर 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ ट्विच 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 यूट्यूब 🎥</strong></a></summary>

- क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को हैकट्रिक्स में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)

- प्राप्त करें [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **फॉलो** करें मुझे **ट्विटर** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपने हैकिंग ट्रिक्स को [हैकट्रिक्स रेपो](https://github.com/carlospolop/hacktricks) और [हैकट्रिक्स-क्लाउड रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके साझा करें**.

</details>

## लॉक डिवाइस

एक एंड्रॉइड डिवाइस से डेटा निकालने के लिए उसे अनलॉक करना होगा। यदि यह लॉक है, तो आप कर सकते हैं:

* जांचें कि डिवाइस में USB के माध्यम से डीबगिंग सक्षम है या नहीं।
* [स्मज अटैक](https://www.usenix.org/legacy/event/woot10/tech/full\_papers/Aviv.pdf) के लिए संभावित जांचें
* [ब्रूट-फोर्स](https://www.cultofmac.com/316532/this-brute-force-device-can-crack-any-iphones-pin-code/) के साथ प्रयास करें

## डेटा अधिग्रहण

[adb का उपयोग करके एंड्रॉइड बैकअप बनाएं](mobile-pentesting/android-app-pentesting/adb-commands.md#backup) और इसे [एंड्रॉइड बैकअप एक्सट्रैक्टर](https://sourceforge.net/projects/adbextractor/) का उपयोग करके निकालें: `java -jar abe.jar unpack file.backup file.tar`

### यदि रूट एक्सेस या जेटैग इंटरफेस के लिए शारीरिक कनेक्शन है

* `cat /proc/partitions` (फ्लैश मेमोरी के पथ की खोज करें, आमतौर पर पहला प्रविष्टि _mmcblk0_ होता है और पूरी फ्लैश मेमोरी के लिए होता है)।
* `df /data` (सिस्टम के ब्लॉक साइज़ का पता लगाएं)।
* dd if=/dev/block/mmcblk0 of=/sdcard/blk0.img bs=4096 (इसे ब्लॉक साइज़ से एकत्रित जानकारी के साथ चलाएं)।

### मेमोरी

Linux Memory Extractor (LiME) का उपयोग करके रैम जानकारी निकालें। यह एक कर्नल एक्सटेंशन है जिसे adb के माध्यम से लोड किया जाना चाहिए।
