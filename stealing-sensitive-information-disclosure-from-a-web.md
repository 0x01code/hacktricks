# वेब से संवेदनशील जानकारी चोरी करना

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ हैकट्रिक्स क्लाउड ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 ट्विटर 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ ट्विच 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 यूट्यूब 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **हैकट्रिक्स में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**द पीएस फैमिली**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT संग्रह**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **फॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स को** [**हैकट्रिक्स रेपो**](https://github.com/carlospolop/hacktricks) **और** [**हैकट्रिक्स-क्लाउड रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में पीआर जमा करके साझा करें।**

</details>

यदि किसी बिंदु पर आपको एक **वेब पृष्ठ मिलता है जो आपके सत्र पर आधारित संवेदनशील जानकारी प्रस्तुत करता है**: शायद यह कुकीज़ को प्रतिबिंबित कर रहा हो, या प्रिंट कर रहा हो या सीसी विवरण या किसी अन्य संवेदनशील जानकारी को, आप इसे चोरी करने का प्रयास कर सकते हैं।\
यहां मैं आपको इसे प्राप्त करने के मुख्य तरीके प्रस्तुत करता हूँ:

* [**CORS बाईपास**](pentesting-web/cors-bypass.md): यदि आप CORS हैडर को बाईपास कर सकते हैं तो आप एक दुष्ट पृष्ठ के लिए Ajax अनुरोध करके जानकारी चोरी कर सकते हैं।
* [**XSS**](pentesting-web/xss-cross-site-scripting/): यदि आप पृष्ठ पर XSS संरचनात्मकता की कमजोरी पाते हैं तो आप इसका दुरुपयोग करके जानकारी चोरी कर सकते हैं।
* [**डैंगलिंग मार्कअप**](pentesting-web/dangling-markup-html-scriptless-injection/): यदि आप XSS टैग नहीं डाल सकते हैं तो आप अन्य साधारित HTML टैग का उपयोग करके भी जानकारी चोरी कर सकते हैं।
* [**क्लिकजैकिंग**](pentesting-web/clickjacking.md): यदि इस हमले के खिलाफ कोई सुरक्षा नहीं है, तो आप उपयोगकर्ता को धोखा देकर संवेदनशील डेटा को भेजने में सक्षम हो सकते हैं (एक उदाहरण [यहां](https://medium.com/bugbountywriteup/apache-example-servlet-leads-to-61a2720cac20)).

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ हैकट्रिक्स क्लाउड ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 ट्विटर 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ ट्विच 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 यूट्यूब 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **हैकट्रिक्स में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**द पीएस फैमिली**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT संग्रह**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **फॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स को** [**हैकट्रिक्स रेपो**](https://github.com/carlospolop/hacktricks) **और** [**हैकट्रिक्स-क्लाउड रेपो**](https://github.com/carlospolop/hacktricks
