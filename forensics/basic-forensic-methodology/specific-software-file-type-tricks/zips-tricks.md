# ZIPs tricks

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

**कमांड लाइन उपकरण** जो **ज़िप फ़ाइलें** प्रबंधित करने के लिए आवश्यक हैं जो ज़िप फ़ाइलें निदान, मरम्मत और क्रैकिंग के लिए हैं। यहाँ कुछ मुख्य उपयोगी उपकरण हैं:

- **`unzip`**: बताता है कि ज़िप फ़ाइल को क्यों डीकंप्रेस नहीं किया जा सकता है।
- **`zipdetails -v`**: ज़िप फ़ाइल प्रारूप क्षेत्रों का विस्तृत विश्लेषण प्रदान करता है।
- **`zipinfo`**: ज़िप फ़ाइल की सामग्री की सूची बनाता है बिना उन्हें निकाले।
- **`zip -F input.zip --out output.zip`** और **`zip -FF input.zip --out output.zip`**: क्षतिग्रस्त ज़िप फ़ाइलों को मरम्मत करने की कोशिश करें।
- **[fcrackzip](https://github.com/hyc/fcrackzip)**: ज़िप पासवर्ड की ब्रूट-फोर्स क्रैकिंग के लिए एक उपकरण, जो लगभग 7 अक्षरों तक के पासवर्ड के लिए प्रभावी है।

[ज़िप फ़ाइल प्रारूप विनिर्देशिका](https://pkware.cachefly.net/webdocs/casestudies/APPNOTE.TXT) ज़िप फ़ाइलों के संरचना और मानकों पर विस्तृत विवरण प्रदान करती है।

महत्वपूर्ण नोट: पासवर्ड से सुरक्षित ज़िप फ़ाइलें **फ़ाइल नाम या फ़ाइल का आकार नहीं एन्क्रिप्ट** करती हैं, जो एक सुरक्षा दोष है जिसे RAR या 7z फ़ाइलें नहीं साझा करती हैं जो इस जानकारी को एन्क्रिप्ट करती हैं। इसके अतिरिक्त, पुरानी ZipCrypto विधि से एन्क्रिप्ट की गई ज़िप फ़ाइलें एक **प्लेनटेक्स्ट हमले** के लिए संवेदनशील हैं यदि किसी अनएन्क्रिप्टेड कॉपी की उपलब्ध है। एक उपलब्ध फ़ाइल का ज्ञात सामग्री का उपयोग करके ज़िप का पासवर्ड क्रैक करने के लिए यह हमला किया जा सकता है, जो [HackThis के लेख](https://www.hackthis.co.uk/articles/known-plaintext-attack-cracking-zip-files) में विस्तार से वर्णित है और [इस शैक्षिक पेपर](https://www.cs.auckland.ac.nz/\~mike/zipattacks.pdf) में और विस्तार से समझाया गया है। हालांकि, **AES-256** एन्क्रिप्शन के साथ सुरक्षित की गई ज़िप फ़ाइलें इस प्लेनटेक्स्ट हमले से मुक्त हैं, संवेदनशील डेटा के लिए सुरक्षित एन्क्रिप्शन विध
