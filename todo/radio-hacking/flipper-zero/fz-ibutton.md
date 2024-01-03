# FZ - iButton

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से नायक तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपोज में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>

## परिचय

iButton क्या है, इसके बारे में अधिक जानकारी के लिए देखें:

{% content-ref url="../ibutton.md" %}
[ibutton.md](../ibutton.md)
{% endcontent-ref %}

## डिजाइन

निम्नलिखित छवि का **नीला** भाग यह दर्शाता है कि आपको असली iButton को कैसे **रखना चाहिए** ताकि Flipper इसे **पढ़ सके।** **हरा** भाग यह दर्शाता है कि आपको Flipper zero को रीडर से कैसे **स्पर्श करना चाहिए** ताकि iButton का **सही ढंग से अनुकरण** किया जा सके।

<figure><img src="../../../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

## क्रियाएँ

### पढ़ें

Read Mode में Flipper iButton की कुंजी को स्पर्श करने की प्रतीक्षा कर रहा होता है और यह तीन प्रकार की कुंजियों: **Dallas, Cyfral, और Metakom** को पचा सकता है। Flipper **स्वयं कुंजी के प्रकार का पता लगा लेगा**। कुंजी प्रोटोकॉल का नाम ID नंबर के ऊपर स्क्रीन पर प्रदर्शित होगा।

### मैन्युअली जोड़ें

आप मैन्युअली एक iButton जोड़ सकते हैं जिसका प्रकार हो: **Dallas, Cyfral, और Metakom**

### **अनुकरण करें**

सहेजे गए iButtons (पढ़े गए या मैन्युअली जोड़े गए) का **अनुकरण** करना संभव है।

{% hint style="info" %}
यदि आप Flipper Zero के अपेक्षित संपर्कों को रीडर से स्पर्श नहीं करा पा रहे हैं तो आप **बाहरी GPIO का उपयोग कर सकते हैं:**
{% endhint %}

<figure><img src="../../../.gitbook/assets/image (24) (1).png" alt=""><figcaption></figcaption></figure>

## संदर्भ

* [https://blog.flipperzero.one/taming-ibutton/](https://blog.flipperzero.one/taming-ibutton/)

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से नायक तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपोज में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>
