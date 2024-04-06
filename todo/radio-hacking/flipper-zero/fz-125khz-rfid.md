# FZ - 125kHz RFID

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a> <strong>के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी HackTricks में विज्ञापित हो** या **HackTricks को PDF में डाउनलोड** करें तो [**सदस्यता योजनाएं देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)\*\* पर फॉलो\*\* करें।
* **हैकिंग ट्रिक्स साझा करें** द्वारा PRs सबमिट करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>

## परिचय

125kHz टैग काम कैसे करते हैं के बारे में अधिक जानकारी के लिए देखें:

{% content-ref url="../pentesting-rfid.md" %}
[pentesting-rfid.md](../pentesting-rfid.md)
{% endcontent-ref %}

## क्रियाएँ

इन प्रकार के टैग्स के बारे में अधिक जानकारी के लिए [**इस परिचय**](../pentesting-rfid.md#low-frequency-rfid-tags-125khz) को पढ़ें।

### पढ़ें

कार्ड जानकारी **पढ़ने** का प्रयास करता है। फिर इसे **उत्पादित** कर सकता है।

{% hint style="warning" %}
ध्यान दें कि कुछ इंटरकॉम्स खुद को कुंजी की नकल से सुरक्षित रखने की कोशिश करते हैं जिसके लिए पढ़ने से पहले एक लेखन कमांड भेजते हैं। यदि लेखन सफल होता है, तो उस टैग को नकली माना जाता है। जब Flipper RFID का उत्पादन करता है, तो पढ़ने वाले को मूल से इसे भिन्न करने का कोई तरीका नहीं होता है, इसलिए ऐसी कोई समस्याएँ नहीं होती हैं।
{% endhint %}

### मैन्युअल रूप से जोड़ें

आप **Flipper Zero में डेटा दर्शाते हुए नकली कार्ड बना सकते हैं** और फिर इसे उत्पादित कर सकते हैं।

#### कार्ड पर आईडी

कभी-कभी, जब आप एक कार्ड प्राप्त करते हैं, तो आपको उसमें लिखी आईडी (या हिस्सा) मिलता है।

* **EM Marin**

उदाहरण के लिए इस EM-Marin कार्ड में शारीरिक कार्ड में **पांच बाइट में से पिछले 3 बाइट को स्पष्ट रूप से पढ़ा जा सकता है**।\
अगर आप कार्ड से उन्हें पढ़ नहीं सकते हैं तो उनमें से 2 को ब्रूट-फोर्स किया जा सकता है।

<figure><img src="../../../.gitbook/assets/image (30).png" alt=""><figcaption></figcaption></figure>

* **HID**

इस HID कार्ड में भी ऐसा ही होता है जहाँ केवल 3 बाइट में से 2 बाइट कार्ड में मुद्रित पाए जा सकते हैं।

<figure><img src="../../../.gitbook/assets/image (15) (3).png" alt=""><figcaption></figcaption></figure>

### उत्पादित/लिखें

कार्ड की नकल बनाने या आईडी **मैन्युअल रूप से दर्ज** करने के बाद Flipper Zero के साथ इसे **उत्पादित** करना या वास्तविक कार्ड में **लिखना** संभव है।

## संदर्भ

* [https://blog.flipperzero.one/rfid/](https://blog.flipperzero.one/rfid/)

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a> <strong>के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी HackTricks में विज्ञापित हो** या **HackTricks को PDF में डाउनलोड** करें तो [**सदस्यता योजनाएं देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)\*\* पर फॉलो\*\* करें।
* **हैकिंग ट्रिक्स साझा करें** द्वारा PRs सबमिट करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
