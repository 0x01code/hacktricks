# FZ - iButton

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी कंपनी का विज्ञापन **HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, **The PEASS Family** की खोज करें
* **जुड़ें** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) से या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

## परिचय

आईबटन क्या है इसके बारे में अधिक जानकारी के लिए देखें:

{% content-ref url="../ibutton.md" %}
[ibutton.md](../ibutton.md)
{% endcontent-ref %}

## डिज़ाइन

निम्नलिखित छवि का **नीला** हिस्सा यह दिखाता है कि आपको **वास्तविक आईबटन को कैसे रखना है** ताकि फ्लिपर इसे **पढ़ सके।** **हरा** हिस्सा यह दिखाता है कि आपको **फ्लिपर जीरो को रीडर के साथ स्पर्श करना है** ताकि आप एक आईबटन को **सही ढंग से उत्पन्न कर सकें।**

<figure><img src="../../../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

## क्रियाएँ

### पढ़ें

पढ़ने मोड में फ्लिपर आईबटन को छूने के लिए प्रतीक्षा कर रहा है और तीन प्रकार की कुंजियों को पचान सकता है: **डलास, साइफ्रल, और मेटाकॉम**। फ्लिपर खुद ही कुंजी के प्रोटोकॉल का पता लगाएगा। कुंजी प्रोटोकॉल का नाम ID नंबर के ऊपर स्क्रीन पर प्रदर्शित किया जाएगा।

### मैन्युअल रूप से जोड़ें

**मैन्युअल रूप से** एक आईबटन को जोड़ना संभव है: **डलास, साइफ्रल, और मेटाकॉम**

### **उत्पन्न करें**

सहेजे गए आईबटनों को **उत्पन्न** करना संभव है (पढ़ें या मैन्युअल रूप से जोड़े गए)।

{% hint style="info" %}
यदि आप फ्लिपर जीरो की अपेक्षित संपर्क को रीडर से छू नहीं पा रहे हैं तो आप **बाह्य GPIO का उपयोग कर सकते हैं:**
{% endhint %}

<figure><img src="../../../.gitbook/assets/image (24) (1).png" alt=""><figcaption></figcaption></figure>

## संदर्भ

* [https://blog.flipperzero.one/taming-ibutton/](https://blog.flipperzero.one/taming-ibutton/)

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी कंपनी का विज्ञापन **HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, **The PEASS Family** की खोज करें
* **जुड़ें** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) से या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>
