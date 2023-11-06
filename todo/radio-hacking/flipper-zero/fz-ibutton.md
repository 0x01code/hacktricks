# FZ - iButton

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT संग्रह**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल हों** या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **हैकिंग ट्रिक्स साझा करें और PR जमा करें** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को**

</details>

## परिचय

आईबटन क्या है इसके बारे में अधिक जानकारी के लिए देखें:

{% content-ref url="../ibutton.md" %}
[ibutton.md](../ibutton.md)
{% endcontent-ref %}

## डिज़ाइन

निम्नलिखित छवि का **नीला** हिस्सा यह दिखाता है कि आपको **वास्तविक आईबटन को** कैसे रखना होगा ताकि फ्लिपर इसे **पढ़ सके।** **हरी** भाग यह दिखाता है कि आपको फ्लिपर जीरो को **इंटरनल आईबटन की तरह सही ढंग से छूना** होगा।

<figure><img src="../../../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

## क्रियाएँ

### पढ़ें

पढ़ने के मोड में फ्लिपर आईबटन की प्रतीक्षा कर रहा है और तीन प्रकार की कुंजियों को पारित कर सकता है: **डैलास, साइफ्रल और मेटाकॉम**। फ्लिपर खुद ही कुंजी के प्रोटोकॉल का पता लगा लेगा। कुंजी प्रोटोकॉल का नाम ID नंबर के ऊपर स्क्रीन पर प्रदर्शित होगा।

### मैन्युअल रूप से जोड़ें

**मैन्युअल रूप से** आईबटन को जोड़ना संभव है: **डैलास, साइफ्रल और मेटाकॉम**

### **उत्पन्न करें**

सहेजे गए आईबटन (पढ़ें या मैन्युअल रूप से जोड़े गए) को **उत्पन्न करना संभव है।**

{% hint style="info" %}
यदि आप फ्लिपर जीरो के अपेक्षित संपर्कों को छूने में सक्षम नहीं हो सकते हैं, तो आप **बाहरी GPIO का उपयोग कर सकते हैं:**
{% endhint %}

<figure><img src="../../../.gitbook/assets/image (24) (1).png" alt=""><figcaption></figcaption></figure>

## संदर्भ

* [https://blog.flipperzero.one/taming-ibutton/](https://blog.flipperzero.one/taming-ibutton/)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT संग्रह**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल हों** या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **हैकिंग ट्रिक्स साझा करें और PR जमा करें** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को**

</details>
