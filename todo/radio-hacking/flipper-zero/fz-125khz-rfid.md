# FZ - 125kHz RFID

<details>

<summary><strong>शून्य से नायक तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **अपनी हैकिंग ट्रिक्स साझा करें, HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके.

</details>

## परिचय

125kHz टैग्स कैसे काम करते हैं, इसके बारे में अधिक जानकारी के लिए देखें:

{% content-ref url="../../../radio-hacking/pentesting-rfid.md" %}
[pentesting-rfid.md](../../../radio-hacking/pentesting-rfid.md)
{% endcontent-ref %}

## क्रियाएँ

इन प्रकार के टैग्स के बारे में अधिक जानकारी के लिए [**यह परिचय पढ़ें**](../../../radio-hacking/pentesting-rfid.md#low-frequency-rfid-tags-125khz).

### पढ़ें

कार्ड की जानकारी **पढ़ने** का प्रयास करता है। फिर इसे **अनुकरण** कर सकता है।

{% hint style="warning" %}
ध्यान दें कि कुछ इंटरकॉम्स कुंजी डुप्लिकेशन से बचाव के लिए पढ़ने से पहले एक लिखने का कमांड भेजते हैं। यदि लिखना सफल होता है, तो उस टैग को नकली माना जाता है। जब Flipper RFID का अनुकरण करता है तो पाठक के लिए इसे मूल से अलग करने का कोई तरीका नहीं होता, इसलिए ऐसी कोई समस्या नहीं होती।
{% endhint %}

### मैन्युअली जोड़ें

आप Flipper Zero में **नकली कार्ड्स बना सकते हैं जिसमें आप मैन्युअली डेटा इंगित करते हैं** और फिर इसे अनुकरण कर सकते हैं।

#### कार्ड्स पर IDs

कभी-कभी, जब आपको एक कार्ड मिलता है तो आपको कार्ड पर लिखी गई ID (या उसका हिस्सा) दिखाई देगी।

* **EM Marin**

उदाहरण के लिए इस EM-Marin कार्ड में भौतिक कार्ड पर संभव है कि आप **पिछले 3 में से 5 बाइट्स को स्पष्ट रूप से पढ़ सकें**।\
अन्य 2 को ब्रूट-फोर्स किया जा सकता है यदि आप उन्हें कार्ड से पढ़ नहीं सकते।

<figure><img src="../../../.gitbook/assets/image (30).png" alt=""><figcaption></figcaption></figure>

* **HID**

इस HID कार्ड में भी ऐसा ही होता है जहां कार्ड पर छपे हुए केवल 2 में से 3 बाइट्स पाए जा सकते हैं

<figure><img src="../../../.gitbook/assets/image (15) (3).png" alt=""><figcaption></figcaption></figure>

### अनुकरण/लिखें

कार्ड की **नकल** करने के बाद या ID **मैन्युअली दर्ज** करने के बाद इसे Flipper Zero के साथ **अनुकरण** किया जा सकता है या एक वास्तविक कार्ड में **लिखा** जा सकता है।

## संदर्भ

* [https://blog.flipperzero.one/rfid/](https://blog.flipperzero.one/rfid/)

<details>

<summary><strong>शून्य से नायक तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **अपनी हैकिंग ट्रिक्स साझा करें, HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके.

</details>
