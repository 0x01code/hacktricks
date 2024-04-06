# FZ - Sub-GHz

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a> <strong>के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **जुड़ें** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)\*\* पर फॉलो\*\* करें।
* **अपने हैकिंग ट्रिक्स साझा करें** द्वारा PRs सबमिट करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

**Try Hard सुरक्षा समूह**

<figure><img src="https://github.com/carlospolop/hacktricks/blob/in/todo/radio-hacking/.gitbook/assets/telegram-cloud-document-1-5159108904864449420.jpg" alt=""><figcaption></figcaption></figure>

{% embed url="https://discord.gg/tryhardsecurity" %}

***

## परिचय <a href="#kfpn7" id="kfpn7"></a>

फ्लिपर जीरो **300-928 मेगाहर्ट्ज की श्रेणी में रेडियो फ्रीक्वेंसी प्राप्त और प्रेषित कर सकता है** अपने बिल्ट-इन मॉड्यूल के साथ, जो रिमोट कंट्रोल को पढ़, सहेज और अनुकरण कर सकता है। ये नियंत्रण द्वारों, बैरियर्स, रेडियो ताले, रिमोट कंट्रोल स्विच, वायरलेस डोरबेल्स, स्मार्ट लाइट्स और अधिक के साथ बातचीत के लिए उपयोग किए जाते हैं। फ्लिपर जीरो आपकी मदद कर सकता है जानने में कि क्या आपकी सुरक्षा को क्षति पहुंची है।

<figure><img src="../../../.gitbook/assets/image (3) (2) (1).png" alt=""><figcaption></figcaption></figure>

## सब-गीगाहर्ट्ज हार्डवेयर <a href="#kfpn7" id="kfpn7"></a>

फ्लिपर जीरो में एक बिल्ट-इन सब-1 गीगाहर्ट्ज मॉड्यूल है जो [﻿](https://www.st.com/en/nfc/st25r3916.html#overview)﻿[CC1101 चिप](https://www.ti.com/lit/ds/symlink/cc1101.pdf) पर आधारित है और एक रेडियो एंटीना है (अधिकतम दायरा 50 मीटर है)। CC1101 चिप और एंटीना दोनों 300-348 मेगाहर्ट्ज, 387-464 मेगाहर्ट्ज और 779-928 मेगाहर्ट्ज बैंड्स में काम करने के लिए डिज़ाइन किए गए हैं।

<figure><img src="../../../.gitbook/assets/image (1) (8) (1).png" alt=""><figcaption></figcaption></figure>

## क्रियाएँ

### फ्रीक्वेंसी विश्लेषक

{% hint style="info" %}
कैसे पता करें कि रिमोट किस फ्रीक्वेंसी का उपयोग कर रहा है
{% endhint %}

विश्लेषण करते समय, फ्लिपर जीरो फ्रीक्वेंसी कॉन्फ़िगरेशन में उपलब्ध सभी फ्रीक्वेंसियों पर सिग्नल शक्ति (RSSI) स्कैन कर रहा है। फ्लिपर जीरो सबसे उच्च RSSI मान वाली फ्रीक्वेंसी को प्रदर्शित करता है, जिसमें सिग्नल की शक्ति -90 [dBm](https://en.wikipedia.org/wiki/DBm) से अधिक है।

रिमोट की फ्रीक्वेंसी निर्धारित करने के लिए, निम्नलिखित करें:

1. रिमोट कंट्रोल को बहुत करीब फ्लिपर जीरो के बाएं में रखें।
2. **मुख्य मेनू** **→ सब-गीगाहर्ट्ज** पर जाएं।
3. **फ्रीक्वेंसी विश्लेषक** चुनें, फिर उस रिमोट कंट्रोल के बटन पर दबाएं जिसे आप विश्लेषण करना चाहते हैं।
4. स्क्रीन पर फ्रीक्वेंसी मान की समीक्षा करें।

### पढ़ें

{% hint style="info" %}
उपयोग की गई फ्रीक्वेंसी के बारे में जानकारी खोजें (एक और तरीका जिससे पता चलता है कि कौन सी फ्रीक्वेंसी का उपयोग किया गया है)
{% endhint %}

**पढ़ें** विकल्प **निर्धारित फ्रीक्वेंसी पर सुनवाई करता है** जिसे डिफ़ॉल्ट रूप से 433.92 AM में किया जाता है। जब **कुछ मिलता है** जब पढ़ा जाता है, तो स्क्रीन में **जानकारी दी जाती है**। यह जानकारी भविष्य में सिग्नल की प्रतिलिपि बनाने के लिए उपयोग की जा सकती है।

पढ़ते समय, **बाएं बटन** दबाकर और **कॉन्फ़िगर करें**।\
इस समय इसमें **4 मॉड्यूलेशन** (AM270, AM650, FM328 और FM476) हैं, और **कई महत्वपूर्ण फ्रीक्वेंसियाँ** संग्रहीत हैं:

<figure><img src="../../../.gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure>

आप **जिसे भी देखना चाहते हैं** उसे सेट कर सकते हैं, हालांकि, यदि आपको यह नहीं पता कि रिमोट द्वारा कौन सी फ्रीक्वेंसी का उपयोग किया गया हो सकता है, **हॉपिंग को ऑन करें** (डिफ़ॉल्ट रूप से ऑफ), और फ्लिपर उसे पकड़ने और आपको उस जानकारी को देने के लिए कई बार बटन दबाए जब तक तकनीक आपको आवश्यक फ्रीक्वेंसी सेट करने की जानकारी न दे।

{% hint style="danger" %}
फ्रीक्वेंसियों के बीच स्विच करने में कुछ समय लगता है, इसलिए स्विचिंग के समय प्रेषित सिग्नल छूट सकते हैं। बेहतर सिग्नल प्राप्ति के लिए, फ्रीक्वेंसी विश्लेषक द्वारा निर्धारित एक स्थिर फ्रीक्वेंसी सेट करें।
{% endhint %}

### **रॉ रीड**

{% hint style="info" %}
निर्धारित फ्रीक्वेंसी में एक सिग्नल चुराएं (और पुनः प्रसारित करें)
{% endhint %}

**रीड रॉ** विकल्प **सुनवाई की गई सिग्नल** रिकॉर्ड करता है जो सुनने वाली फ्रीक्वेंसी में भेजा गया है। इसका उपयोग सिग्नल चुराने और उसे दोहराने के लिए किया जा सकता है।

डिफ़ॉल्ट रूप से **रीड रॉ भी 433.92 में AM650 में है**, लेकिन यदि पढ़ने के दौरान आपको यह पता चलता है कि आपको दिखाई गई सिग्नल किसी **अलग फ्रीक्वेंसी/मॉड्यूलेशन में है, तो आप उसे भी संशोधित कर सकते हैं** बाएं दबाकर (रीड रॉ विकल्प के अंदर होते हुए)।

### ब्रूट-फोर्स

यदि आप गेराज दरवाजे द्वारा उपयोग किए जाने वाले प्रोटोकॉल को जानते हैं, तो आप **सभी कोड उत्पन्न कर सकते हैं और उन्हें फ्लिपर जीरो के साथ भेज सकते हैं**। यह एक उदाहरण है जो सामान्य रूप से गेराजों के सामान्य प्रकारों का समर्थन करता है: [**https://github.com/tobiabocchi/flipperzero-bruteforce**](https://github.com/tobiabocchi/flipperzero-bruteforce)

### मैन्युअल रूप से जोड़ें

{% hint style="info" %}
निर्धारित प्रोटोकॉलों की सूची से सिग्नल जोड़ें
{% endhint %}

#### \[समर्थित प्रोटोकॉ

### समर्थित सब-जीएचजेड विक्रेताओं

[https://docs.flipperzero.one/sub-ghz/supported-vendors](https://docs.flipperzero.one/sub-ghz/supported-vendors) में सूची देखें

### क्षेत्र द्वारा समर्थित फ्रीक्वेंसी

[https://docs.flipperzero.one/sub-ghz/frequencies](https://docs.flipperzero.one/sub-ghz/frequencies) में सूची देखें

### परीक्षण

{% hint style="info" %}
सहेजी गई फ्रीक्वेंसियों के डीबीएम प्राप्त करें
{% endhint %}

## संदर्भ

* [https://docs.flipperzero.one/sub-ghz](https://docs.flipperzero.one/sub-ghz)

**Try Hard सुरक्षा समूह**

<figure><img src="https://github.com/carlospolop/hacktricks/blob/in/todo/radio-hacking/.gitbook/assets/telegram-cloud-document-1-5159108904864449420.jpg" alt=""><figcaption></figcaption></figure>

{% embed url="https://discord.gg/tryhardsecurity" %}

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a> <strong>के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी कंपनी का विज्ञापन HackTricks में देखना चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **जुड़ें** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)\*\* पर फॉलो\*\* करें।
* **अपने हैकिंग ट्रिक्स साझा करें, HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके।

</details>
