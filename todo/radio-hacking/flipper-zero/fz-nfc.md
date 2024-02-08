# FZ - NFC

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ हैकट्रिक्स क्लाउड ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 ट्विटर 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ ट्विच 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 यूट्यूब 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप चाहते हैं कि आपकी **कंपनी हैकट्रिक्स में विज्ञापित हो**? या क्या आप **PEASS के नवीनतम संस्करण का उपयोग करना चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**द पीएएस फैमिली**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**एनएफटी**](https://opensea.io/collection/the-peass-family) संग्रह।
* [**आधिकारिक पीएएस और हैकट्रिक्स स्वैग**](https://peass.creator-spring.com) प्राप्त करें।
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **ट्विटर** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)** पर** **फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**हैकट्रिक्स रेपो**](https://github.com/carlospolop/hacktricks) **और** [**हैकट्रिक्स-क्लाउड रेपो**](https://github.com/carlospolop/hacktricks-cloud)।

</details>

<figure><img src="/.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

वे सुरक्षा गंभीरता को ध्यान में रखते हुए वे सबसे महत्वपूर्ण दुरुपयोग खोजें ताकि आप उन्हें तेजी से ठीक कर सकें। इंट्रूडर आपके हमले क्षेत्र का ट्रैक करता है, प्रोएक्टिव धमकी स्कैन चलाता है, आपकी पूरी तकनीकी स्टैक, API से वेब ऐप्स और क्लाउड सिस्टम तक मुद्दे खोजता है। [**यह नि: शुल्क ट्राय करें**](https://www.intruder.io/?utm\_source=referral\&utm\_campaign=hacktricks) आज।

{% embed url="https://www.intruder.io/?utm_campaign=hacktricks&utm_source=referral" %}

***

## परिचय <a href="#9wrzi" id="9wrzi"></a>

RFID और NFC के बारे में जानकारी के लिए निम्नलिखित पृष्ठ की जांच करें:

{% content-ref url="../../../radio-hacking/pentesting-rfid.md" %}
[pentesting-rfid.md](../../../radio-hacking/pentesting-rfid.md)
{% endcontent-ref %}

## समर्थित NFC कार्ड <a href="#9wrzi" id="9wrzi"></a>

{% hint style="danger" %}
NFC कार्ड के अलावा, फ्लिपर जीरो **उच्च आवृत्ति कार्ड** को समर्थित करता है जैसे कि कई **माइफेयर** क्लासिक और अल्ट्रालाइट और **एनटीएजी**।
{% endhint %}

नए प्रकार के NFC कार्ड समर्थित कार्डों की सूची में जोड़े जाएंगे। फ्लिपर जीरो निम्नलिखित **NFC कार्ड प्रकार A** (ISO 14443A) को समर्थित करता है:

* ﻿**बैंक कार्ड (ईएमवी)** — केवल UID, SAK, और ATQA पढ़ें बिना सहेजे।
* ﻿**अज्ञात कार्ड** — (UID, SAK, ATQA) पढ़ें और एक UID का अनुकरण करें।

**NFC कार्ड प्रकार B, प्रकार F, और प्रकार V** के लिए, फ्लिपर जीरो एक UID पढ़ सकता है बिना इसे सहेजे।

### NFC कार्ड प्रकार A <a href="#uvusf" id="uvusf"></a>

#### बैंक कार्ड (ईएमवी) <a href="#kzmrp" id="kzmrp"></a>

फ्लिपर जीरो केवल एक UID, SAK, ATQA, और बैंक कार्ड पर स्टोर किए गए डेटा को **सहेजे बिना** पढ़ सकता है।

बैंक कार्ड पढ़ने की स्क्रीन बैंक कार्ड के लिए, फ्लिपर जीरो केवल डेटा पढ़ सकता है **बिना सहेजे और इसे अनुकरण करना**।

<figure><img src="https://cdn.flipperzero.one/Monosnap_Miro_2022-08-17_12-26-31.png?auto=format&#x26;ixlib=react-9.1.1&#x26;h=916&#x26;w=2662" alt=""><figcaption></figcaption></figure>

#### अज्ञात कार्ड <a href="#37eo8" id="37eo8"></a>

जब फ्लिपर जीरो **NFC कार्ड के प्रकार का निर्धारण करने में असमर्थ** होता है, तो केवल **UID, SAK, और ATQA** पढ़े और सहेजे जा सकते हैं।

अज्ञात कार्ड पढ़ने की स्क्रीन अज्ञात NFC कार्ड के लिए, फ्लिपर जीरो केवल एक UID अनुकरण कर सकता है।

<figure><img src="https://cdn.flipperzero.one/Monosnap_Miro_2022-08-17_12-27-53.png?auto=format&#x26;ixlib=react-9.1.1&#x26;h=932&#x26;w=2634" alt=""><figcaption></figcaption></figure>

### NFC कार्ड प्रकार B, F, और V <a href="#wyg51" id="wyg51"></a>

**NFC कार्ड प्रकार B, F, और V** के लिए, फ्लिपर जीरो केवल एक UID पढ़ सकता है और इसे सहेजे बिना दिखा सकता है।

<figure><img src="https://archbee.imgix.net/3StCFqarJkJQZV-7N79yY/zBU55Fyj50TFO4U7S-OXH_screenshot-2022-08-12-at-182540.png?auto=format&#x26;ixlib=react-9.1.1&#x26;h=1080&#x26;w=2704" alt=""><figcaption></figcaption></figure>

## क्रियाएँ

NFC के बारे में एक परिचय के लिए [**इस पृष्ठ को पढ़ें**](../../../radio-hacking/pentesting-rfid.md#high-frequency-rfid-tags-13.56-mhz)।

### पढ़ें

फ्लिपर जीरो **NFC कार्ड पढ़ सकता है**, हालांकि, यह **सभी प्रोटोकॉल को समझ नहीं सकता** जो ISO 14443 पर आधारित है। हालांकि, क्योंकि **UID एक निम्न स्तरीय गुण है**, आपको एक स्थिति में पाने की स्थिति में पड़ सकते हैं जब **UID पहले से पढ़ा गया है, लेकिन उच्च स्तरीय डेटा स्थानांतरण प्रोटोकॉल अज्ञात है**। आप प्राचीन पठकों के लिए UID का उपयोग करने के लिए फ्लिपर का उपयोग करके UID पढ़, अनुकरण और मैन्युअल इनपुट कर सकते हैं।

#### UID पढ़ना बनाम अंदर के डेटा पढ़ना <a href="#reading-the-uid-vs-reading-the-data-inside" id="reading-the-uid-vs-reading-the-data-inside"></a>

<figure><img src="../../../.gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>

फ्लिपर में, 13.56 मेगाहर्ट्ज टैग पढ़ने को दो भागों में विभाजित किया जा सकता है:

* **निम्न स्तरीय पढ़ना** — केवल UID, SAK, और ATQA पढ़ता है। फ्लिपर इस डेटा से उच्च स्तरीय प्रोटोकॉल को अनुमानित करने का प्रयास करता है। आप इसके साथ 100% निश्चित नहीं हो सकते, क्योंकि यह कुछ तत्वों पर आधारित केवल एक अनुमान है।
* **उच्च स्तरीय पढ़ना** — विशेष उच्च स्तरीय प्रोटोकॉल का उपयोग करके कार्ड की मेमोरी से डेटा पढ़ता है। यह Mifare Ultralight पर डेटा पढ़ना, Mifare Classic से सेक्टर पढ़ना, या PayPass/Apple Pay से कार्ड की विशेषताएँ पढ़ना होगा।

### विशेष पढ़ें

यदि फ्लिपर जीरो निम्न स्तरीय डेटा से कार्ड के प्रकार को खोजने में सक्षम नहीं है, तो `अतिरिक्त क्रियाएँ` में आप `विशेष कार्ड प्रकार पढ़ें` चुन सकते हैं और **मैन्युअल** **रूप से इंडिकेट कर सकते हैं कि आप किस प्रकार का कार्ड पढ़ना चाहेंगे**।

#### ईएमवी बैंक कार्ड (PayPass, payWave, Apple Pay, Google Pay) <a href="#emv-bank-cards-paypass-paywave-apple-pay-google-pay" id="emv-bank-cards-paypass-paywave-apple-pay-google-pay"></a>

UID को केवल पढ़ने के अलावा, आप ए
