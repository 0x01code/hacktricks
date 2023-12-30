# वेब से संवेदनशील जानकारी की चोरी

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert) के साथ शून्य से नायक तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में शामिल हों या [**telegram group**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>

यदि कभी आपको एक **वेब पेज मिलता है जो आपके सत्र के आधार पर आपको संवेदनशील जानकारी प्रस्तुत करता है**: शायद यह कुकीज़ को प्रतिबिंबित कर रहा है, या प्रिंटिंग या CC विवरण या कोई अन्य संवेदनशील जानकारी, आप इसे चुराने का प्रयास कर सकते हैं।\
यहाँ मैं आपको मुख्य तरीके प्रस्तुत करता हूँ जिनसे आप इसे हासिल करने का प्रयास कर सकते हैं:

* [**CORS bypass**](pentesting-web/cors-bypass.md): यदि आप CORS हेडर्स को बायपास कर सकते हैं तो आप एक दुर्भावनापूर्ण पेज के लिए Ajax अनुरोध करके जानकारी चुरा सकते हैं।
* [**XSS**](pentesting-web/xss-cross-site-scripting/): यदि आप पेज पर XSS भेद्यता पाते हैं तो आप इसका दुरुपयोग करके जानकारी चुरा सकते हैं।
* [**Danging Markup**](pentesting-web/dangling-markup-html-scriptless-injection/): यदि आप XSS टैग्स इंजेक्ट नहीं कर सकते हैं तो आप अन्य नियमित HTML टैग्स का उपयोग करके जानकारी चुरा सकते हैं।
* [**Clickjaking**](pentesting-web/clickjacking.md): यदि इस हमले के खिलाफ कोई सुरक्षा नहीं है, तो आप उपयोगकर्ता को धोखा देकर संवेदनशील डेटा आपको भेजने के लिए मजबूर कर सकते हैं (एक उदाहरण [यहाँ](https://medium.com/bugbountywriteup/apache-example-servlet-leads-to-61a2720cac20))।

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert) के साथ शून्य से नायक तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में शामिल हों या [**telegram group**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>
