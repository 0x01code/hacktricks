# FZ - 125kHz RFID

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने** की उपलब्धता चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल** हों या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **हैकिंग ट्रिक्स को शेयर करें और PRs सबमिट करें** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **में।**

</details>

## परिचय

125kHz टैग काम करने के बारे में अधिक जानकारी के लिए यह देखें:

{% content-ref url="../../../radio-hacking/pentesting-rfid.md" %}
[pentesting-rfid.md](../../../radio-hacking/pentesting-rfid.md)
{% endcontent-ref %}

## कार्रवाई

इन प्रकार के टैग के बारे में अधिक जानकारी के लिए [**इस परिचय**](../../../radio-hacking/pentesting-rfid.md#low-frequency-rfid-tags-125khz) को पढ़ें।

### पढ़ें

कार्ड जानकारी को **पढ़ने** का प्रयास करें। फिर इसे **उत्पन्न** कर सकते हैं।

{% hint style="warning" %}
ध्यान दें कि कुछ इंटरकॉम्स अपने आप को कुंजी की प्रतिलिपि से सुरक्षित करने का प्रयास करते हैं जो पढ़ने से पहले एक लिखित आदेश भेजकर करते हैं। यदि लिखित आदेश सफल होता है, तो वह टैग नकली माना जाता है। जब Flipper RFID की अनुकरण करता है, तो पाठक को मूल से इसे अलग नहीं करने का कोई तरीका नहीं होता है, इसलिए ऐसी कोई समस्या नहीं होती है।
{% endhint %}

### मैन्युअल रूप से जोड़ें

आप **Flipper Zero में फ़र्ज़ी कार्ड बना सकते हैं और डेटा को मैन्युअल रूप से दर्ज करके उसे उत्पन्न** कर सकते हैं।

#### कार्ड पर आईडी

कभी-कभी, जब आप कार्ड प्राप्त करते हैं, तो आपको कार्ड में लिखी गई आईडी (या भाग) मिलती है।

* **EM Marin**

उदाहरण के लिए, इस EM-Marin कार्ड में शारीरिक कार्ड में **पढ़ा जा सकता है** कि पिछले 5 बाइट में से 3 बाइट **स्पष्ट में पढ़ा जा सकता है**।\
अगर आप कार्ड से उन्हें पढ़ नहीं सकते हैं, तो आप उन्हें ब्रूट-फ़ोर्स कर सकते हैं।

<figure><img src="../../../.gitbook/assets/image (30).png" alt=""><figcaption></figcaption></figure>

* **HID**

इस HID कार्ड में भी ऐसा ही होता है जहां केवल 3 बाइट में से 2 बाइट कार्ड में मुद्रित मिलते हैं

<figure><img src="../../../.gitbook/assets/image (15) (3).png" alt=""><figcaption></figcaption></figure>

### उत्पन्न/लिखें

कार्ड की **नकल बनाने** या आईडी **मैन्युअल रूप से दर्ज करने** के बाद, इसे Flipper Zero के साथ **उत्पन्न** किया जा सकता है या इसे एक वास्तविक कार्ड में **लिखा** जा सकता है।

## संदर्भ

* [https://blog.flipperzero.one/rfid/](https://blog.flipperzero.one/rfid/)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनल
