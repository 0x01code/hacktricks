# FZ - NFC

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित देखना चाहते हैं**? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का एक्सेस चाहिए**? [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) की जाँच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह।
* [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें।
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **Twitter** पर **फॉलो** करें 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)**।**
* **अपने हैकिंग ट्रिक्स साझा करें** PRs को **hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **के माध्यम से।

</details>

<figure><img src="/.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

वे सुरक्षा गंभीरता को ध्यान में रखते हैं ताकि आप उन्हें तेजी से ठीक कर सकें। Intruder आपकी हमले की सतह का ट्रैक करता है, प्रोएक्टिव धारणा स्कैन चलाता है, आपकी पूरी तकनीकी स्टैक, API से वेब ऐप्स और क्लाउड सिस्टम तक मुद्दे खोजता है। [**इसे मुफ्त में ट्राई करें**](https://www.intruder.io/?utm\_source=referral\&utm\_campaign=hacktricks) आज।

{% embed url="https://www.intruder.io/?utm_campaign=hacktricks&utm_source=referral" %}

***

## परिचय <a href="#9wrzi" id="9wrzi"></a>

RFID और NFC के बारे में जानकारी के लिए निम्नलिखित पेज देखें:

{% content-ref url="../../../radio-hacking/pentesting-rfid.md" %}
[pentesting-rfid.md](../../../radio-hacking/pentesting-rfid.md)
{% endcontent-ref %}

## समर्थित NFC कार्ड <a href="#9wrzi" id="9wrzi"></a>

{% hint style="danger" %}
NFC कार्डों के अलावा Flipper Zero **उच्च आवृत्ति कार्ड** को समर्थन करता है जैसे कई **Mifare** Classic और Ultralight और **NTAG**।
{% endhint %}

नए प्रकार के NFC कार्ड समर्थित किए जाएंगे। Flipper Zero निम्नलिखित **NFC कार्ड प्रकार A** (ISO 14443A) का समर्थन करता है:

* ﻿**बैंक कार्ड (EMV)** — केवल UID, SAK, और ATQA पढ़ सकता है बिना सहेजे।
* ﻿**अज्ञात कार्ड** — (UID, SAK, ATQA) पढ़ सकता है और एक UID को अनुकरण कर सकता है।

**NFC कार्ड प्रकार B, प्रकार F, और प्रकार V** के लिए, Flipper Zero एक UID पढ़ सकता है बिना इसे सहेजे।

### NFC कार्ड प्रकार A <a href="#uvusf" id="uvusf"></a>

#### बैंक कार्ड (EMV) <a href="#kzmrp" id="kzmrp"></a>

Flipper Zero केवल एक UID, SAK, ATQA, और बैंक कार्ड पर स्टोर किए गए डेटा को **सहेजे बिना** पढ़ सकता है।

बैंक कार्ड पढ़ने की स्क्रीन बैंक कार्ड के लिए, Flipper Zero केवल डेटा पढ़ सकता है **बिना सहेजे और अनुकरण करने**।

<figure><img src="https://cdn.flipperzero.one/Monosnap_Miro_2022-08-17_12-26-31.png?auto=format&#x26;ixlib=react-9.1.1&#x26;h=916&#x26;w=2662" alt=""><figcaption></figcaption></figure>

#### अज्ञात कार्ड <a href="#37eo8" id="37eo8"></a>

जब Flipper Zero **NFC कार्ड के प्रकार का निर्धारण करने में असमर्थ** होता है, तो केवल **UID, SAK, और ATQA** पढ़ा और सहेजा जा सकता है।

अज्ञात कार्ड पढ़ने की स्क्रीन अज्ञात NFC कार्ड के लिए, Flipper Zero केवल एक UID को अनुकरण कर सकता है।

<figure><img src="https://cdn.flipperzero.one/Monosnap_Miro_2022-08-17_12-27-53.png?auto=format&#x26;ixlib=react-9.1.1&#x26;h=932&#x26;w=2634" alt=""><figcaption></figcaption></figure>

### NFC कार्ड प्रकार B, F, और V <a href="#wyg51" id="wyg51"></a>

**NFC कार्ड प्रकार B, F, और V** के लिए, Flipper Zero केवल एक UID पढ़ सकता है और इसे सहेजे बिना दिखा सकता है।

<figure><img src="https://archbee.imgix.net/3StCFqarJkJQZV-7N79yY/zBU55Fyj50TFO4U7S-OXH_screenshot-2022-08-12-at-182540.png?auto=format&#x26;ixlib=react-9.1.1&#x26;h=1080&#x26;w=2704" alt=""><figcaption></figcaption></figure>

## क्रियाएँ

NFC के बारे में एक परिचय के लिए [**इस पेज**](../../../radio-hacking/pentesting-rfid.md#high-frequency-rfid-tags-13.56-mhz) को पढ़ें।

### पढ़ें

Flipper Zero **NFC कार्ड पढ़ सकता है**, हालांकि, यह **उन सभी प्रोटोकॉल को समझने में असमर्थ है** जो ISO 14443 पर आधारित हैं। हालांकि, क्योंकि **UID एक निम्न स्तरीय गुण** है, आपको एक स्थिति में पाने की स्थिति में पाएंगे जब **UID पहले से पढ़ा गया है, लेकिन उच्च स्तरीय डेटा स्थानांतरण प्रोटोकॉल अज्ञात है**। आप प्राचीन पठकों के लिए UID का उपयोग करते हुए Flipper का उपयोग करके UID पढ़, अनुकरण और मैन्युअल इनपुट कर सकते हैं।

#### UID पढ़ना बनाम अंदर के डेटा पढ़ना <a href="#reading-the-uid-vs-reading-the-data-inside" id="reading-the-uid-vs-reading-the-data-inside"></a>

<figure><img src="../../../.gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>

Flipper में, 13.56 मेगाहर्ट्ज टैग पढ़ने को दो भागों में विभाजित किया जा सकता है:

* **निम्न स्तरीय पढ़ना** — केवल UID, SAK, और ATQA पढ़ता है। Flipper इस डेटा से उच्च स्तरीय प्रोटोकॉल को अनुमानित करने का प्रयास करता है। इसमें आप 100% निश्चित नहीं हो सकते, क्योंकि यह कुछ कारकों पर आधारित केवल एक अनुमान है।
* **उच्च स्तरीय पढ़ना** — विशिष्ट उच्च स्तरीय प्रोटोकॉल का उपयोग करके कार्ड की मेमोरी से डेटा पढ़ता है। यह Mifare Ultralight पर डेटा पढ़ना, Mifare Classic से सेक्टर पढ़ना, या PayPass/Apple Pay से कार्ड की विशेषताएँ पढ़ना होगा।

### विशिष्ट पढ़ें

यदि Flipper Zero निम्न स्तरीय डेटा से कार्ड के प्रकार को खोजने में सक्षम नहीं है, तो `अतिरिक्त क्रियाएँ` में आप `विशिष्ट कार्ड प्रकार पढ़ें` चुन सकते हैं और **मैन्युअल** **रूप से इंडिकेट कर सकते हैं कि आप किस प्रकार का कार्ड पढ़ना चाहेंगे**।

#### EMV बैंक कार्ड (PayPass, payWave, Apple Pay, Google Pay) <a href="#emv-bank-cards-paypass-paywave-apple-pay-google-pay" id="emv-bank-cards-paypass-paywave-apple-pay-google-pay"></a>

UID को केवल पढ़ने के अलावा, आप एक बैंक कार्ड से बहुत अधिक डेटा निकाल सकते हैं। यह संभावना है कि आप **पूरा कार्ड नंबर** (कार्ड के सामने के 16 अंक), **वैधता तिथि**, और कुछ मामलों की **सबसे हाल की लेन-देन की सूची** के साथ **मालिक का नाम** भी प्राप्त कर सकते हैं।\
हालांकि, इस तरीके से **CVV पढ़ना संभव नहीं है** (कार्ड के पीछे के 3 अंक)। इसके अलावा **बैंक कार्ड पुनरावृत्ति हमलों से सुरक्षित हैं**, इसलिए इसे Flipper के साथ कॉपी करने के बाद उसे अनुकरण करने का प्रयास करके कुछ खरीदने के लिए काम नही
