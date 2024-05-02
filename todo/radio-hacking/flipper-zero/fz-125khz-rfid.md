# FZ - 125kHz RFID

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप चाहते हैं कि आपकी **कंपनी HackTricks में विज्ञापित हो** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** पर **फॉलो** करें 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें** और PRs को **जमा करें** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

<figure><img src="https://pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}


## परिचय

125kHz टैग कैसे काम करते हैं के बारे में अधिक जानकारी के लिए देखें:

{% content-ref url="../pentesting-rfid.md" %}
[pentesting-rfid.md](../pentesting-rfid.md)
{% endcontent-ref %}

## क्रियाएँ

इन प्रकार के टैग्स के बारे में अधिक जानकारी के लिए [**इस परिचय**](../pentesting-rfid.md#low-frequency-rfid-tags-125khz) को पढ़ें।

### पढ़ें

कार्ड जानकारी **पढ़ने** का प्रयास करता है। फिर इसे **उत्पादित** कर सकता है।

{% hint style="warning" %}
ध्यान दें कि कुछ इंटरकॉम्स कुंजी की प्रतिलिपि से पहले लिखने के लिए स्वयं को सुरक्षित करने की कोशिश करते हैं। यदि लेखन सफल होता है, तो उस टैग को नकली माना जाता है। जब Flipper RFID का उत्पादन करता है, तो पठक के लिए यह मूल से भिन्न करने का कोई तरीका नहीं है, इसलिए ऐसी कोई समस्याएँ नहीं होती हैं।
{% endhint %}

### मैन्युअल रूप से जोड़ें

आप **Flipper Zero में फर्जी कार्ड बना सकते हैं** जिसमें आप डेटा को मैन्युअल रूप से दर्ज कर सकते हैं और फिर इसे उत्पादित कर सकते हैं।

#### कार्ड पर आईडी

कभी-कभी, जब आप एक कार्ड प्राप्त करते हैं, तो आपको उसमें लिखी आईडी (या हिस्सा) मिलेगा।

* **EM मारिन**

उदाहरण के लिए इस EM-Marin कार्ड में शारीरिक कार्ड में **पांच बाइट में से पिछले 3 बाइट को स्पष्ट रूप से पढ़ा जा सकता है**।\
अगर आप कार्ड से उन्हें पढ़ नहीं सकते हैं तो उन्हें ब्रूट-फोर्स किया जा सकता है।

<figure><img src="../../../.gitbook/assets/image (101).png" alt=""><figcaption></figcaption></figure>

* **HID**

इस HID कार्ड में भी ऐसा ही होता है जहाँ केवल 3 बाइट में से 2 बाइट कार्ड में मुद्रित पाए जा सकते हैं

<figure><img src="../../../.gitbook/assets/image (1011).png" alt=""><figcaption></figcaption></figure>

### उत्पादित/लिखें

किसी कार्ड की **नकल** बनाने के बाद या आईडी **मैन्युअल** रूप से दर्ज करने के बाद, इसे Flipper Zero में **उत्पादित** करना या वास्तविक कार्ड में **लिखना** संभव है।

## संदर्भ

* [https://blog.flipperzero.one/rfid/](https://blog.flipperzero.one/rfid/)

<figure><img src="https://pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}


<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप चाहते हैं कि आपकी **कंपनी HackTricks में विज्ञापित हो** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** पर **फॉलो** करें 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें** और PRs को **जमा करें** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>
