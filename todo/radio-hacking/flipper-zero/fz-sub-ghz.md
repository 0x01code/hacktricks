# FZ - सब-जीएचजी

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित करना** चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को पीडीएफ में डाउनलोड करने** की पहुंच चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT संग्रह**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल हों** या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का पालन करें**.
* **अपने हैकिंग ट्रिक्स को** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में पीआर जमा करके** अपना योगदान दें।

</details>

<figure><img src="/.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

वे संदेशों को खोजें जो सबसे अधिक मायने रखते हैं ताकि आप उन्हें तेजी से ठीक कर सकें। Intruder आपकी हमला सतह का ट्रैक करता है, प्रोएक्टिव धमकी स्कैन चलाता है, आपकी पूरी टेक स्टैक, एपीआई से वेब ऐप्स और क्लाउड सिस्टम तक, सभी मुद्दों को खोजता है। [**इसे मुफ्त में प्रयास करें**](https://www.intruder.io/?utm\_source=referral\&utm\_campaign=hacktricks) आज।

{% embed url="https://www.intruder.io/?utm_campaign=hacktricks&utm_source=referral" %}

***

## परिचय <a href="#kfpn7" id="kfpn7"></a>

Flipper Zero में एक बिल्ट-इन मॉड्यूल है जो **300-928 मेगाहर्ट्ज़** की रेंज में रेडियो फ्रीक्वेंसी को प्राप्त और प्रेषित कर सकता है, जो रिमोट कंट्रोल को पढ़, सहेज और नकल कर सकता है। ये कंट्रोल गेट, बैरियर, रेडियो लॉक, रिमोट कंट्रोल स्विच, वायरलेस डोरबेल, स्मार्ट लाइट्स और अधिक के साथ इंटरैक्शन के लिए उपयोग किए जाते हैं। Flipper Zero आपकी सुरक्षा को जांचने में मदद कर सकता है।

<figure><img src="../../../.gitbook/assets/image (3) (2) (1).png" alt=""><figcaption></figcaption></figure>

## सब-जीएचजी हार्डवेयर <a href="#kfpn7" id="kfpn7"></a>

Flipper Zero में एक बिल्ट-इन सब-1 जीएचजी मॉड्यूल है, जो [﻿](https://www.st.com/en/nfc/st25r3916.html#overview)﻿[CC1101 चिप](https://www.ti.com/lit/ds/symlink/cc1101.pdf) पर आधारित है और एक रेडियो एंटीना है (अधिकतम दूरी 50 मीटर है)। CC1101 चिप और एंटीना दोनों 300-348 मेगाहर्ट्ज़, 387-464 मेगाहर्ट्ज़ और 779-928 मेगाहर्ट्ज़ बैंड में कार्य करने के लिए डिज़ाइन किए गए हैं।

<figure><img src="../../../.gitbook/assets/image (1) (8) (1).png" alt=""><figcaption></figcaption></figure>

## कार्रवाई

### फ्रीक्वेंसी विश्लेषक

{% hint style="info" %}
कैसे पता करें कि रिमोट कौन सी फ्रीक्वेंसी का उपयोग कर रहा है
{% endhint %}

विश्लेषण करते समय, Flipper Zero फ्रीक्वेंसी कॉन्फ़िगरेशन में उपलब्ध सभी फ्रीक्वेंसी पर सिग्नल स्ट्रेंग्थ (RSSI) स्कैन कर रहा है। Flipper Zero सबसे अधिक RSSI मान वाली फ्रीक्वेंसी को प्रदर्शित करता है, जिसमें सिग्नल स्ट्रेंग्थ -90 [dBm](https://en.wikipedia.org/wiki/DBm) से अधिक होती है।

रिमोट की फ्रीक्वेंसी निर्धारित करने के लिए, निम्नलिखित कार्रवाई करें:

1. रिमोट कंट्रोल को Flipper Zero के बाएं बहुत करीब रखें।
2. **मुख्य मेनू** **→ सब-जीएचजी** पर जाएं।
3. **फ्रीक्वेंसी विश्लेषक** का चयन करें, फिर वह बटन द
### ब्रूट-फोर्स

यदि आप गेराज द्वार द्वारा उपयोग किए जाने वाले प्रोटोकॉल को जानते हैं, तो आप फ्लिपर जीरो के साथ सभी कोड उत्पन्न कर सकते हैं और उन्हें भेज सकते हैं। यह एक उदाहरण है जो आमतौर पर उपयोग होने वाले गेराजों के सामान्य प्रकार का समर्थन करता है: [**https://github.com/tobiabocchi/flipperzero-bruteforce**](https://github.com/tobiabocchi/flipperzero-bruteforce)\*\*\*\*

### मैन्युअल रूप से जोड़ें

{% hint style="info" %}
प्रोटोकॉल की एक विन्यासित सूची से संकेत जोड़ें
{% endhint %}

#### [समर्थित प्रोटोकॉल](https://docs.flipperzero.one/sub-ghz/add-new-remote) की सूची <a href="#3iglu" id="3iglu"></a>

| Princeton\_433 (सभी स्थिर कोड प्रणालियों के साथ काम करता है) | 433.92 | स्थिर   |
| --------------------------------------------------------------- | ------ | ------- |
| Nice Flo 12bit\_433                                             | 433.92 | स्थिर   |
| Nice Flo 24bit\_433                                             | 433.92 | स्थिर   |
| CAME 12bit\_433                                                 | 433.92 | स्थिर   |
| CAME 24bit\_433                                                 | 433.92 | स्थिर   |
| Linear\_300                                                     | 300.00 | स्थिर   |
| CAME TWEE                                                       | 433.92 | स्थिर   |
| Gate TX\_433                                                    | 433.92 | स्थिर   |
| DoorHan\_315                                                    | 315.00 | गतिशील |
| DoorHan\_433                                                    | 433.92 | गतिशील |
| LiftMaster\_315                                                 | 315.00 | गतिशील |
| LiftMaster\_390                                                 | 390.00 | गतिशील |
| Security+2.0\_310                                               | 310.00 | गतिशील |
| Security+2.0\_315                                               | 315.00 | गतिशील |
| Security+2.0\_390                                               | 390.00 | गतिशील |

### समर्थित सब-जीएच विक्रेता

[https://docs.flipperzero.one/sub-ghz/supported-vendors](https://docs.flipperzero.one/sub-ghz/supported-vendors) में सूची देखें

### क्षेत्र द्वारा समर्थित आवृत्तियाँ

[https://docs.flipperzero.one/sub-ghz/frequencies](https://docs.flipperzero.one/sub-ghz/frequencies) में सूची देखें

### परीक्षण

{% hint style="info" %}
सहेजी गई आवृत्तियों के dBm प्राप्त करें
{% endhint %}

## संदर्भ

* [https://docs.flipperzero.one/sub-ghz](https://docs.flipperzero.one/sub-ghz)

<figure><img src="/.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

वे संवेदनशीलता खोजें जो सबसे महत्वपूर्ण हैं ताकि आप उन्हें तेजी से ठीक कर सकें। Intruder आपकी हमला प्रविष्टि को ट्रैक करता है, प्रोएक्टिव धारणा स्कैन चलाता है, आपकी पूरी टेक स्टैक, एपीआई से वेब ऐप्स और क्लाउड सिस्टम तक, सभी मुद्दों को खोजता है। [**इसे मुफ्त में प्रयास करें**](https://www.intruder.io/?utm\_source=referral\&utm\_campaign=hacktricks) आज।

{% embed url="https://www.intruder.io/?utm_campaign=hacktricks&utm_source=referral" %}


<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की अनुमति चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल हों** या मुझे **ट्विटर** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का** **अनुसरण करें।**
* **अपने हैकिंग ट्रिक्स साझा करें, PRs के माध्यम से** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को सबमिट करके।**

</details>
