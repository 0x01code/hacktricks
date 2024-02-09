# iButton

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप अपनी कंपनी का **HackTricks में विज्ञापन देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

## परिचय

iButton एक इलेक्ट्रॉनिक पहचान कुंजी के लिए एक सामान्य नाम है जो एक **सिक्के के आकार के धातु कंटेनर** में पैक किया गया है। इसे डैलस टच मेमोरी या संपर्क मेमोरी भी कहा जाता है। यह अक्सर "मैग्नेटिक" कुंजी के रूप में गलती से संदर्भित किया जाता है, लेकिन इसमें **कुछ भी मैग्नेटिक नहीं** है। वास्तव में, एक पूर्णकालिक **माइक्रोचिप** जो एक डिजिटल प्रोटोकॉल पर काम कर रहा है, इसके भीतर छिपा है।

<figure><img src="../../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

### iButton क्या है? <a href="#what-is-ibutton" id="what-is-ibutton"></a>

सामान्यत: iButton में कुंजी और पठक का भौतिक रूप - एक गोल सिक्का जिसमें दो संपर्क हैं, का अर्थ है। इसके लिए जो फ्रेम है, उसके चारों ओर कई विभिन्न परिवर्तन हैं, सबसे सामान्य प्लास्टिक होल्डर से लेकर अंगूठी, पेंडेंट आदि।

<figure><img src="../../.gitbook/assets/image (23) (2).png" alt=""><figcaption></figcaption></figure>

जब कुंजी पठक तक पहुंचती है, तो **संपर्क स्पर्श में आते हैं** और कुंजी को अपना आईडी **प्रेषित** करने की शक्ति मिलती है। कभी-कभी कुंजी **तुरंत पठन नहीं** होती क्योंकि **एक इंटरकॉम का संपर्क PSD अधिक** होता है जितना होना चाहिए। इस स्थिति में, आपको पठक को पठक के दीवारों में से एक पर दबाना होगा।

<figure><img src="../../.gitbook/assets/image (21) (2).png" alt=""><figcaption></figcaption></figure>

### **1-Wire प्रोटोकॉल** <a href="#1-wire-protocol" id="1-wire-protocol"></a>

डैलस कुंजियाँ 1-वायर प्रोटोकॉल का उपयोग करके डेटा आदान-प्रदान करती हैं। डेटा स्थानांतरण के लिए केवल एक संपर्क (!!) होता है, दोनों दिशाओं में, मास्टर से स्लेव और उल्टा। 1-वायर प्रोटोकॉल मास्टर-स्लेव मॉडल के अनुसार काम करता है। इस टोपोलॉजी में, मास्टर हमेशा संचार प्रारंभ करता है और स्लेव उसके निर्देशों का पालन करता है।

जब कुंजी (स्लेव) इंटरकॉम (मास्टर) से संपर्क करती है, तो कुंजी के भीतर छिपा चिप चालू हो जाता है, इंटरकॉम द्वारा शक्ति प्राप्त करके, और कुंजी को प्रारंभ किया जाता है। इसके बाद इंटरकॉम कुंजी आईडी का अनुरोध करता है। अगले, हम इस प्रक्रिया को अधिक विस्तार से देखेंगे।

फ्लिपर मास्टर और स्लेव मोड में दोनों काम कर सकता है। कुंजी पठन मोड में, फ्लिपर एक पाठक के रूप में काम करता है यह कहना है कि यह मास्टर के रूप में काम करता है। और कुंजी उपन्यास मोड में, फ्लिपर एक कुंजी बनने का दिखावा करता है, यह स्लेव मोड में है।

### डैलस, साइफ्रल और मेटाकॉम कुंजियाँ

इन कुंजियों के काम के बारे में जानकारी के लिए पृष्ठ [https://blog.flipperzero.one/taming-ibutton/](https://blog.flipperzero.one/taming-ibutton/) देखें

### हमले

iButton को Flipper Zero के साथ हमला किया जा सकता है:

{% content-ref url="flipper-zero/fz-ibutton.md" %}
[fz-ibutton.md](flipper-zero/fz-ibutton.md)
{% endcontent-ref %}

## संदर्भ

* [https://blog.flipperzero.one/taming-ibutton/](https://blog.flipperzero.one/taming-ibutton/)
