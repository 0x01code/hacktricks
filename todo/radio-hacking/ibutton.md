# iButton

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से नायक तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **अपनी हैकिंग ट्रिक्स साझा करें, HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके.

</details>

## परिचय

iButton एक इलेक्ट्रॉनिक पहचान कुंजी का सामान्य नाम है जो **सिक्के के आकार के धातु के कंटेनर** में पैक की गई है। इसे **Dallas Touch** Memory या संपर्क मेमोरी भी कहा जाता है। हालांकि इसे अक्सर गलती से “मैग्नेटिक” कुंजी के रूप में संदर्भित किया जाता है, इसमें **कुछ भी मैग्नेटिक नहीं** है। वास्तव में, इसके अंदर एक पूर्ण **माइक्रोचिप** छिपी हुई है जो एक डिजिटल प्रोटोकॉल पर काम करती है।

<figure><img src="../../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

### iButton क्या है? <a href="#what-is-ibutton" id="what-is-ibutton"></a>

आमतौर पर, iButton का तात्पर्य कुंजी और पाठक के भौतिक रूप से होता है - दो संपर्कों वाला एक गोल सिक्का। इसके चारों ओर के फ्रेम के लिए, सबसे आम प्लास्टिक होल्डर से लेकर अंगूठियों, पेंडेंट्स आदि तक कई प्रकार के विविधताएं हैं।

<figure><img src="../../.gitbook/assets/image (23) (2).png" alt=""><figcaption></figcaption></figure>

जब कुंजी पाठक तक पहुँचती है, तो **संपर्क छूने लगते हैं** और कुंजी अपनी ID **प्रसारित** करने के लिए सक्रिय हो जाती है। कभी-कभी कुंजी को तुरंत **नहीं पढ़ा** जाता क्योंकि **इंटरकॉम का संपर्क PSD बड़ा** होता है जितना कि होना चाहिए। इसलिए कुंजी और पाठक के बाहरी आकार छू नहीं पाते। अगर ऐसा है, तो आपको पाठक की दीवारों में से एक पर कुंजी दबानी होगी।

<figure><img src="../../.gitbook/assets/image (21) (2).png" alt=""><figcaption></figcaption></figure>

### **1-Wire प्रोटोकॉल** <a href="#1-wire-protocol" id="1-wire-protocol"></a>

Dallas कुंजियाँ 1-wire प्रोटोकॉल का उपयोग करके डेटा का आदान-प्रदान करती हैं। केवल एक संपर्क डेटा ट्रांसफर के लिए (!!) दोनों दिशाओं में, मास्टर से स्लेव और इसके विपरीत। 1-wire प्रोटोकॉल मास्टर-स्लेव मॉडल के अनुसार काम करता है। इस टोपोलॉजी में, मास्टर हमेशा संचार शुरू करता है और स्लेव उसके निर्देशों का पालन करता है।

जब कुंजी (स्लेव) इंटरकॉम (मास्टर) से संपर्क करती है, तो कुंजी के अंदर की चिप चालू हो जाती है, इंटरकॉम द्वारा संचालित, और कुंजी को प्रारंभ किया जाता है। उसके बाद इंटरकॉम कुंजी ID का अनुरोध करता है। आगे हम इस प्रक्रिया को और अधिक विस्तार से देखेंगे।

Flipper मास्टर और स्लेव दोनों मोड में काम कर सकता है। कुंजी पढ़ने के मोड में, Flipper एक पाठक के रूप में काम करता है यानी यह मास्टर के रूप में काम करता है। और कुंजी अनुकरण मोड में, फ्लिपर खुद को एक कुंजी होने का नाटक करता है, यह स्लेव मोड में है।

### Dallas, Cyfral & Metakom कुंजियाँ

इन कुंजियों के काम करने के बारे में जानकारी के लिए पेज देखें [https://blog.flipperzero.one/taming-ibutton/](https://blog.flipperzero.one/taming-ibutton/)

### हमले

iButtons पर Flipper Zero के साथ हमला किया जा सकता है:

{% content-ref url="flipper-zero/fz-ibutton.md" %}
[fz-ibutton.md](flipper-zero/fz-ibutton.md)
{% endcontent-ref %}

## संदर्भ

* [https://blog.flipperzero.one/taming-ibutton/](https://blog.flipperzero.one/taming-ibutton/)

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से नायक तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **अपनी हैकिंग ट्रिक्स साझा करें, HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके.

</details>
