# FZ - NFC

<details>

<summary><strong>जीरो से हीरो तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a> <strong>के साथ!</strong></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित देखना चाहते हैं**? या आप **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) की जाँच करें!
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर फॉलो करें 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**।**
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **को**।

</details>

## परिचय <a href="#id-9wrzi" id="id-9wrzi"></a>

RFID और NFC के बारे में जानकारी के लिए निम्नलिखित पृष्ठ की जाँच करें:

{% content-ref url="../pentesting-rfid.md" %}
[pentesting-rfid.md](../pentesting-rfid.md)
{% endcontent-ref %}

## समर्थित NFC कार्ड <a href="#id-9wrzi" id="id-9wrzi"></a>

{% hint style="danger" %}
NFC कार्ड के अलावा, फ्लिपर जीरो **उच्च आवृत्ति कार्ड के अन्य प्रकार** का समर्थन करता है जैसे कई **Mifare** Classic और Ultralight और **NTAG**।
{% endhint %}

नए प्रकार के NFC कार्ड समर्थित कार्डों की सूची में जोड़े जाएंगे। फ्लिपर जीरो निम्नलिखित **NFC कार्ड प्रकार A** (ISO 14443A) का समर्थन करता है:

* ﻿**बैंक कार्ड (EMV)** — केवल UID, SAK, और ATQA पढ़ सकता है बिना सहेजे।
* ﻿**अज्ञात कार्ड** — (UID, SAK, ATQA) पढ़ सकता है और एक UID को अनुकरण कर सकता है।

**NFC कार्ड प्रकार B, प्रकार F, और प्रकार V** के लिए, फ्लिपर जीरो एक UID पढ़ सकता है बिना इसे सहेजे।

### NFC कार्ड प्रकार A <a href="#uvusf" id="uvusf"></a>

#### बैंक कार्ड (EMV) <a href="#kzmrp" id="kzmrp"></a>

फ्लिपर जीरो केवल एक UID, SAK, ATQA, और बैंक कार्ड पर स्टोर किए गए डेटा को **सहेजे बिना** पढ़ सकता है।

बैंक कार्ड पढ़ने की स्क्रीन बैंक कार्ड के लिए, फ्लिपर जीरो केवल डेटा पढ़ सकता है **बिना सहेजे और अनुकरण किए**।

<figure><img src="https://cdn.flipperzero.one/Monosnap_Miro_2022-08-17_12-26-31.png?auto=format&#x26;ixlib=react-9.1.1&#x26;h=916&#x26;w=2662" alt=""><figcaption></figcaption></figure>

#### अज्ञात कार्ड <a href="#id-37eo8" id="id-37eo8"></a>

जब फ्लिपर जीरो **NFC कार्ड के प्रकार का निर्धारण करने में असमर्थ** होता है, तो केवल **UID, SAK, और ATQA** पढ़ और सहेज सकता है।

अज्ञात कार्ड पढ़ने की स्क्रीन अज्ञात NFC कार्ड के लिए, फ्लिपर जीरो केवल एक UID अनुकरण कर सकता है।

<figure><img src="https://cdn.flipperzero.one/Monosnap_Miro_2022-08-17_12-27-53.png?auto=format&#x26;ixlib=react-9.1.1&#x26;h=932&#x26;w=2634" alt=""><figcaption></figcaption></figure>

### NFC कार्ड प्रकार B, F, और V <a href="#wyg51" id="wyg51"></a>

**NFC कार्ड प्रकार B, F, और V** के लिए, फ्लिपर जीरो केवल एक UID पढ़ सकता है और इसे सहेजे बिना दिखा सकता है।

<figure><img src="https://archbee.imgix.net/3StCFqarJkJQZV-7N79yY/zBU55Fyj50TFO4U7S-OXH_screenshot-2022-08-12-at-182540.png?auto=format&#x26;ixlib=react-9.1.1&#x26;h=1080&#x26;w=2704" alt=""><figcaption></figcaption></figure>

## क्रियाएँ

NFC के बारे में एक परिचय के लिए [**इस पृष्ठ**](../pentesting-rfid.md#high-frequency-rfid-tags-13.56-mhz) को पढ़ें।

### पढ़ें

फ्लिपर जीरो **NFC कार्ड पढ़ सकता है**, हालांकि, यह **ISO 14443 पर आधारित सभी प्रोटोकॉल को समझ नहीं सकता**। हालांकि, क्योंकि **UID एक निचले स्तर की विशेषता है**, आपको कभी-कभी ऐसे स्थिति में पाया जा सकता है जब **UID पहले से पढ़ा गया है, लेकिन उच्च स्तरीय डेटा स्थानांतरण प्रोटोकॉल अज्ञात है**। आप प्राचीन पठकों के लिए UID का उपयोग करके Flipper का उपयोग करके UID को पढ़, अनुकरण कर सकते हैं और मैन्युअल रूप से इनपुट कर सकते हैं।

#### UID पढ़ना बनाम अंदर के डेटा पढ़ना <a href="#reading-the-uid-vs-reading-the-data-inside" id="reading-the-uid-vs-reading-the-data-inside"></a>

<figure><img src="../../../.gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>

फ्लिपर में, 13.56 मेगाहर्ट्ज टैग पढ़ने को दो भागों में विभाजित किया जा सकता है:

* **निचले स्तर पढ़ें** — केवल UID, SAK, और ATQA पढ़ता है। फ्लिपर इस डेटा से उच्च स्तरीय प्रोटोकॉल को अनुमानित करने का प्रयास करता है। इसमें आप 100% निश्चित नहीं हो सकते, क्योंकि यह कुछ तत्वों पर आधारित केवल एक अनुमान है।
* **उच्च स्तर पढ़ें** — विशेष उच्च स्तरीय प्रोटोकॉल का उपयोग करके कार्ड की मेमोरी से डेटा पढ़ता है। यह Mifare Ultralight पर डेटा पढ़ना, Mifare Classic से सेक्टर पढ़ना, या PayPass/Apple Pay से कार्ड की विशेषताएँ पढ़ना होगा।

### विशेष पढ़ें

यदि फ्लिपर जीरो निचले स्तर के डेटा से कार्ड के प्रकार को खोजने में सक्षम नहीं है, तो `अतिरिक्त क्रियाएँ` में आप `विशेष कार्ड प्रकार पढ़ें` चुन सकते हैं और **मैन्युअल** **रूप से** **उस प्रकार का कार्ड निर्दिष्ट कर सकते हैं** जिसे आप पढ़ना चाहें।

#### EMV बैंक कार्ड (PayPass, payWave, Apple Pay, Google Pay) <a href="#emv-bank-cards-paypass-paywave-apple-pay-google-pay" id="emv-bank-cards-paypass-paywave-apple-pay-google-pay"></a>

UID को केवल पढ़ने के अलावा, आप एक बैंक कार्ड से बहुत अधिक डेटा निकाल सकते हैं। यह संभव है **पूरा कार्ड नंबर** (कार्ड के सामने के 16 अंक), **वैधता तिथि**, और कुछ मामलों में मालिक का नाम\*\* साथ में **हाल के सभी लेन-देनों की सूची**।\
हालांकि, इस तरीके से **CVV पढ़ा नहीं जा सकता** (कार्ड के पीछे के 3 अंक)। इसके अलावा **बैंक कार्ड पुनरावृत्ति हमलों से सुरक्षित हैं**, इसलिए इसे फ्लिपर के साथ कॉपी करने के बाद और फिर कुछ खरीदने के लिए इसे अनुकरण करने का प्रयास करने पर काम नहीं करेगा।

## संदर्भ

* [https://blog.flipperzero.one/rfid/](https://blog.flipperzero.one/rfid/)

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a> <strong>के साथ!</strong></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी का हैकट्रिक्स में विज्ञापित करना चाहते हैं**? या क्या आप **PEASS के नवीनतम संस्करण देखना चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह।
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें।
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **ट्विटर** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)\*\* पर\*\* **फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud)।

</details>
