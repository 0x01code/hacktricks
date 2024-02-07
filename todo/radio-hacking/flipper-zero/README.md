# Flipper Zero

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी का हैकट्रिक्स में विज्ञापित करना चाहते हैं**? या क्या आप **PEASS के नवीनतम संस्करण देखना चाहते हैं या हैकट्रिक्स को पीडीएफ में डाउनलोड करना चाहते हैं**? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**द पीएस फैमिली**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**एनएफटी**](https://opensea.io/collection/the-peass-family) संग्रह।
* [**आधिकारिक पीएस और हैकट्रिक्स स्वैग**](https://peass.creator-spring.com) प्राप्त करें।
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **ट्विटर पर फॉलो** **करें** **🐦**[**@carlospolopm**](https://twitter.com/hacktricks_live)**।**
* **हैकिंग ट्रिक्स साझा करें, हैकट्रिक्स रेपो** (https://github.com/carlospolop/hacktricks) **और** [**हैकट्रिक्स-क्लाउड रेपो**](https://github.com/carlospolop/hacktricks-cloud) **को PR जमा करके**।

</details>

<figure><img src="/.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

वे सुरक्षित करने के लिए सबसे महत्वपूर्ण सुरक्षा गड़बड़ियों को खोजें ताकि आप उन्हें तेजी से ठीक कर सकें। इंट्रूडर आपके हमले की सतह का ट्रैक करता है, प्रोएक्टिव खतरा स्कैन चलाता है, एपीआई से वेब ऐप्स और क्लाउड सिस्टम जैसे आपके पूरे टेक स्टैक में मुद्दे खोजता है। [**आज ही मुफ्त में इसे ट्राई करें**](https://www.intruder.io/?utm\_source=referral\&utm\_campaign=hacktricks)।

{% embed url="https://www.intruder.io/?utm_campaign=hacktricks&utm_source=referral" %}

***

[**Flipper Zero**](https://flipperzero.one/) के साथ आप:

* **रेडियो तार की सुनें/कैप्चर/रीप्ले करें:** [**सब-जीएचजी**](fz-sub-ghz.md)****
* **NFC कार्ड पढ़ें/कैप्चर/इम्युलेट करें:** [**NFC**](fz-nfc.md)****
* **125kHz टैग पढ़ें/कैप्चर/इम्युलेट करें:** [**125kHz आरआईडीएफ**](fz-125khz-rfid.md)****
* **इन्फ्रारेड सिग्नल पढ़ें/कैप्चर/भेजें:** [**इन्फ्रारेड**](fz-infrared.md)****
* **iButtons पढ़ें/कैप्चर/इम्युलेट करें:** [**iButton**](../ibutton.md)****
* **इसे बैड USB के रूप में उपयोग करें**
* **इसे सुरक्षा कुंजी (U2F) के रूप में उपयोग करें**
* **स्नेक खेलें**

**अन्य फ्लिपर जीरो संसाधन** [**https://github.com/djsime1/awesome-flipperzero**](https://github.com/djsime1/awesome-flipperzero)****

<figure><img src="/.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

वे सुरक्षित करने के लिए सबसे महत्वपूर्ण सुरक्षा गड़बड़ियों को खोजें ताकि आप उन्हें तेजी से ठीक कर सकें। इंट्रूडर आपके हमले की सतह का ट्रैक करता है, प्रोएक्टिव खतरा स्कैन चलाता है, एपीआई से वेब ऐप्स और क्लाउड सिस्टम जैसे आपके पूरे टेक स्टैक में मुद्दे खोजता है। [**आज ही मुफ्त में इसे ट्राई करें**](https://www.intruder.io/?utm\_source=referral\&utm\_campaign=hacktricks)।

{% embed url="https://www.intruder.io/?utm_campaign=hacktricks&utm_source=referral" %}


<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी का हैकट्रिक्स में विज्ञापित करना चाहते हैं**? या क्या आप **PEASS के नवीनतम संस्करण देखना चाहते हैं या हैकट्रिक्स को पीडीएफ में डाउनलोड करना चाहते हैं**? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**द पीएस फैमिली**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**एनएफटी**](https://opensea.io/collection/the-peass-family) संग्रह।
* [**आधिकारिक पीएस और हैकट्रिक्स स्वैग**](https://peass.creator-spring.com) प्राप्त करें।
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **ट्विटर पर फॉलो** **करें** **🐦**[**@carlospolopm**](https://twitter.com/hacktricks_live)**।**
* **हैकिंग ट्रिक्स साझा करें, हैकट्रिक्स रेपो** (https://github.com/carlospolop/hacktricks) **और** [**हैकट्रिक्स-क्लाउड रेपो**](https://github.com/carlospolop/hacktricks-cloud) **को PR जमा करके**।

</details>
