# FZ - NFC

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**official PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**telegram समूह**](https://t.me/peass) में **या** मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को PR जमा करके।**

</details>

<figure><img src="/.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

वे संवेदनशीलता खोजें जो सबसे महत्वपूर्ण हैं ताकि आप उन्हें तेजी से ठीक कर सकें। Intruder आपकी हमला सतह का ट्रैक करता है, प्रोएक्टिव धमकी स्कैन चलाता है, आपकी पूरी टेक स्टैक, एपीआई से वेब ऐप्स और क्लाउड सिस्टम तक, मुद्दों को खोजता है। [**इसे मुफ़्त में ट्राय करें**](https://www.intruder.io/?utm\_source=referral\&utm\_campaign=hacktricks) आज।

{% embed url="https://www.intruder.io/?utm_campaign=hacktricks&utm_source=referral" %}

***

## परिचय <a href="#9wrzi" id="9wrzi"></a>

RFID और NFC के बारे में जानकारी के लिए निम्नलिखित पृष्ठ की जांच करें:

{% content-ref url="../../../radio-hacking/pentesting-rfid.md" %}
[pentesting-rfid.md](../../../radio-hacking/pentesting-rfid.md)
{% endcontent-ref %}

## समर्थित NFC कार्ड <a href="#9wrzi" id="9wrzi"></a>

{% hint style="danger" %}
NFC कार्ड के अलावा Flipper Zero **कई अन्य प्रकार के High-frequency कार्ड** को भी समर्थित करता है जैसे कि कई **Mifare** Classic और Ultralight और **NTAG**।
{% endhint %}

नए प्रकार के NFC कार्ड समर्थित कार्डों की सूची में जोड़े जाएंगे। Flipper Zero निम्नलिखित **NFC कार्ड प्रकार A** (ISO 14443A) का समर्थन करता है:

* ﻿**बैंक कार्ड (EMV)** - केवल UID, SAK, और ATQA को सहेजे बिना पढ़ें।
* ﻿**अज्ञात कार्ड** - (UID, SAK, ATQA) पढ़ें और एक UID की नकल करें।

**NFC कार्ड प्रकार B, प्रकार F, और प्रकार V** के लिए, Flipper Zero एक UID को सहेजे बिना पढ़ सकता है।

### NFC कार्ड प्रकार A <a href="#uvusf" id="uvusf"></a>

#### बैंक कार्ड (EMV) <a href="#kzmrp" id="kzmrp"></a>

Flipper Zero केवल एक UID, SAK, ATQA, और बैंक कार्ड पर संग्रहित डेटा को **सहेजे बिना** पढ़ सकता है।

बैंक कार्ड पढ़ने की स्क्रीनबैंक कार्ड के लिए, Flipper Zero केवल डेटा को पढ़ सकता है **सहेजे और नकल नहीं करता**।

<figure><img src="https://cdn.flipperzero.one/Monosnap_Miro_2022-08-17_12-26-31.png?auto=format&#x26;ixlib=react-9.1.1&#x26;h=916&#x26;w=2662" alt=""><figcaption></figcaption></figure>

#### अज्ञात कार्ड <a href="#37eo8" id="37eo8"></a>

जब Flipper Zero **NFC कार्ड के प्रकार को निर्धारित करने में असमर्थ होता है**, तो केवल **UID, SAK, और ATQA** को **पढ़ा और सहेजा** जा सकता है।

अज्ञात कार्ड पढ़ने की स्क्रीनअज्ञात NFC कार्ड के लिए, Flipper Zero केवल एक UID की नकल कर सकता है।

<figure><img src="https://cdn.flipperzero.one/Monosnap_Miro_2022-08-17_12-27-53.png?auto=format&#x26;ixlib=react-9.1.1&#x26;h=932&#x26;w=2634" alt=""><figcaption></figcaption></figure>

### NFC कार्ड प्रकार B, F, और V <a href="#wyg51" id="wyg51"></a>

**NFC कार्ड प्रकार B, F, और V** के लिए, Flipper Zero केवल एक UID को पढ़ सकता है और प्रदर्शित कर सकता है, इसे सहेजे बिना।

<figure><img src="https://archbee.imgix.net/3StCFqarJkJQZV-7N79yY/zBU55Fyj50TFO4U7S-OXH_screenshot-2022-08-12-at-182540.png?auto=format&#x26;ixlib=react-9.1.1&#x26;h=1080&#x26;w
#### UID पढ़ना बनाम अंदर के डेटा को पढ़ना <a href="#reading-the-uid-vs-reading-the-data-inside" id="reading-the-uid-vs-reading-the-data-inside"></a>

<figure><img src="../../../.gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>

फ्लिपर में, 13.56 मेगाहर्ट्ज़ टैग्स को दो हिस्सों में विभाजित किया जा सकता है:

* **लो-स्तर पढ़ें** - केवल UID, SAK और ATQA पढ़ता है। फ्लिपर इस डेटा के आधार पर हाई-स्तर प्रोटोकॉल को अनुमानित करने की कोशिश करता है। इसमें आप 100% निश्चित नहीं हो सकते क्योंकि यह केवल कुछ कारकों पर आधारित एक अनुमान है।
* **हाई-स्तर पढ़ें** - एक विशिष्ट हाई-स्तर प्रोटोकॉल का उपयोग करके कार्ड की मेमोरी से डेटा पढ़ता है। इसमें Mifare Ultralight पर डेटा पढ़ना, Mifare Classic से सेक्टर पढ़ना या PayPass/Apple Pay से कार्ड की विशेषताओं को पढ़ना शामिल होगा।

### विशिष्ट रीड

यदि फ्लिपर जीरो लो स्तरीय डेटा से कार्ड के प्रकार को खोजने में सक्षम नहीं है, तो `अतिरिक्त कार्रवाई` में आप `विशिष्ट कार्ड प्रकार पढ़ें` का चयन कर सकते हैं और **मैन्युअल** **रूप से कार्ड के प्रकार की घोषणा कर सकते हैं**।

#### EMV बैंक कार्ड (PayPass, payWave, Apple Pay, Google Pay) <a href="#emv-bank-cards-paypass-paywave-apple-pay-google-pay" id="emv-bank-cards-paypass-paywave-apple-pay-google-pay"></a>

UID को पढ़ने के अलावा, आप एक बैंक कार्ड से बहुत सारा डेटा निकाल सकते हैं। आप **पूरी कार्ड नंबर** (कार्ड के सामने के 16 अंक), **मान्यता की तारीख**, और कुछ मामलों में यहां तक कि **मालिक का नाम** भी निकाल सकते हैं साथ ही **नवीनतम लेनदेनों की सूची**।\
हालांकि, आप इस तरीके से CVV पढ़ नहीं सकते हैं (कार्ड के पीछे के 3 अंक)। इसके अलावा, **बैंक कार्ड replay हमलों से सुरक्षित होते हैं**, इसलिए उसे फ्लिपर के साथ कॉपी करके उसे नकली बनाने का प्रयास करके कुछ खरीदने के लिए काम नहीं करेगा।

## संदर्भ

* [https://blog.flipperzero.one/rfid/](https://blog.flipperzero.one/rfid/)

<figure><img src="/.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

वे संदर्भ ढूंढें जो सबसे अधिक मायने रखते हैं ताकि आप उन्हें तेजी से ठीक कर सकें। Intruder आपकी हमला सतह का ट्रैक करता है, प्रोएक्टिव ध्रुवीकरण स्कैन चलाता है, आपकी पूरी टेक स्टैक, एपीआई से वेब ऐप्स और क्लाउड सिस्टम तक, मुद्दों की खोज करता है। [**इसे मुफ्त में प्रयास करें**](https://www.intruder.io/?utm\_source=referral\&utm\_campaign=hacktricks) आज।

{% embed url="https://www.intruder.io/?utm_campaign=hacktricks&utm_source=referral" %}


<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण देखने** या HackTricks को **PDF में डाउनलोड करने** की अनुमति चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)** का पालन करें**।**
* **अपने हैकिंग ट्रिक्स साझा करें** हैकट्रिक्स रेपो में पीआर जमा करके और हैकट्रिक्स-क्लाउड रेपो में पीआर जमा करके।

</details>
