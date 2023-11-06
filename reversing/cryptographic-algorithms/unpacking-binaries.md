<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ हैकट्रिक्स क्लाउड ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 ट्विटर 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ ट्विच 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 यूट्यूब 🎥</strong></a></summary>

- क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **हैकट्रिक्स में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करना चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**द पीएस फैमिली**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**एनएफटी**](https://opensea.io/collection/the-peass-family) संग्रह

- प्राप्त करें [**आधिकारिक पीएस और हैकट्रिक्स स्वैग**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **ट्विटर** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)** का पालन करें**.

- **हैकिंग ट्रिक्स साझा करें** हैकट्रिक्स रेपो (https://github.com/carlospolop/hacktricks) और हैकट्रिक्स-क्लाउड रेपो (https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके।

</details>


# पैक किए गए बाइनरी की पहचान करना

* **स्ट्रिंग की कमी**: आमतौर पर पैक किए गए बाइनरी में कोई भी स्ट्रिंग नहीं होती है
* बहुत सारी **अनुपयोगी स्ट्रिंगें**: इसके अलावा, जब कोई मैलवेयर किसी वाणिज्यिक पैकर का उपयोग करता है, तो अनुपयोगी स्ट्रिंगें बहुत सारी मिलना सामान्य होता है। यदि ये स्ट्रिंगें मौजूद होती हैं, तो यह यह नहीं मतलब है कि बाइनरी पैक नहीं है।
* आप इसके अलावा कुछ टूल का उपयोग करके बाइनरी को पैक करने के लिए कौन सा पैकर उपयोग किया गया है यह जानने के लिए कर सकते हैं:
* [PEiD](http://www.softpedia.com/get/Programming/Packers-Crypters-Protectors/PEiD-updated.shtml)
* [Exeinfo PE](http://www.softpedia.com/get/Programming/Packers-Crypters-Protectors/ExEinfo-PE.shtml)
* [Language 2000](http://farrokhi.net/language/)

# मूल अनुशंसाएं

* **शुरू करें** IDA में पैक किए गए बाइनरी का **नीचे से विश्लेषण करना और ऊपर जाना**। अनपैकर्स उनपैक कोड के बाहर निकलते हैं, इसलिए यह असंभाव है कि अनपैकर प्रारंभ में अनपैक कोड को निष्पादित करने के लिए निष्पादित कोड को पास करेगा।
* **JMP** या **CALL** को **रजिस्टर** या **मेमोरी क्षेत्रों** के लिए खोजें। यहां तक कि **विधियों को धकेलने और एक पता दिशा को कॉल करने के बाद `retn` को कॉल करने** वाले **कार्यों को खोजें**, क्योंकि उस मामले में फ़ंक्शन की वापसी स्टैक पर पुश किए गए पते को कॉल करने से पहले कॉल हो सकती है।
* `VirtualAlloc` पर **ब्रेकपॉइंट** लगाएं क्योंकि यह प्रोग्राम को अनपैक कोड लिखने के लिए मेमोरी में स्थान आवंटित करता है। "यूज़र कोड तक चलें" या फ़ंक्शन को निष्पादित करने के बाद EAX में मान प्राप्त करने के लिए F8 का उपयोग करें और "**डंप में उस पते का पालन करें**"। आप कभी नहीं जान सकते कि वहां वहां उनपैक कोड सहेजा जाएगा जहां यह जाना जाएगा।
* **`VirtualAlloc`** के साथ मान "**40
