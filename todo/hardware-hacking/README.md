<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी HackTricks में विज्ञापित हो** या **HackTricks को PDF में डाउनलोड करें** तो [**सदस्यता योजनाएं देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)** पर **फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** द्वारा PRs सबमिट करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>


#

# JTAG

JTAG को बाउंडरी स्कैन करने की अनुमति देता है। बाउंडरी स्कैन निश्चित सर्किटरी का विश्लेषण करता है, जिसमें हर पिन के लिए एम्बेडेड बाउंडरी-स्कैन सेल और रजिस्टर शामिल हैं।

JTAG मानक निम्नलिखित के लिए **विशिष्ट कमांड्स** को परिभाषित करता है:

* **BYPASS** आपको अन्य चिप के माध्यम से जाने के बिना एक विशिष्ट चिप का परीक्षण करने की अनुमति देता है।
* **SAMPLE/PRELOAD** जब यह अपने सामान्य कार्य करने के मोड में होता है, तो उस डेटा का एक नमूना लेता है जो उसमें दाख़िल हो रहा है और निकल रहा है।
* **EXTEST** पिन स्थित
