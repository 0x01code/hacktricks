# Proxmark 3

<details>

<summary><strong>जीरो से हीरो तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित देखना चाहते हैं**? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का एक्सेस चाहिए**? [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**द पीएस फैमिली**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड ग्रुप**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम ग्रुप**](https://t.me/peass) और **ट्विटर** पर **मेरा पालन करें** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **को**.

</details>

**Try Hard Security Group**

<figure><img src="https://github.com/carlospolop/hacktricks/blob/in/todo/.gitbook/assets/telegram-cloud-document-1-5159108904864449420.jpg" alt=""><figcaption></figcaption></figure>

{% embed url="https://discord.gg/tryhardsecurity" %}

***

## Proxmark3 के साथ RFID सिस्टमों को हमला करना

पहली चीज जो आपको करनी होगी, वह है [**Proxmark3**](https://proxmark.com) और [**सॉफ़्टवेयर और इसकी डिपेंडेंसी को इंस्टॉल करना**](https://github.com/Proxmark/proxmark3/wiki/Kali-Linux)[**s**](https://github.com/Proxmark/proxmark3/wiki/Kali-Linux).

### MIFARE Classic 1KB को हमला करना

इसमें **16 सेक्टर** हैं, प्रत्येक में **4 ब्लॉक** हैं और प्रत्येक ब्लॉक में **16B** होता है। UID सेक्टर 0 ब्लॉक 0 में है (और इसे बदला नहीं जा सकता)।\
प्रत्येक सेक्टर तक पहुंचने के लिए आपको **2 कुंजियाँ** (**A** और **B**) चाहिए होती हैं जो **प्रत्येक सेक्टर के ब्लॉक 3 में** संग्रहित होती हैं (सेक्टर ट्रेलर)। सेक्टर ट्रेलर में भी **एक्सेस बिट्स** संग्रहित होते हैं जो **प्रत्येक ब्लॉक पर पढ़ने और लिखने** की अनुमतियाँ देते हैं जो 2 कुंजियों का उपयोग करते हैं।\
यदि आप पहली को जानते हैं तो पढ़ने की अनुमति देने के लिए 2 कुंजियाँ उपयोगी हैं और यदि आप दूसरी को जानते हैं तो लिखने की अनुमति देने के लिए। (उदाहरण के लिए)।

कई हमले किए जा सकते हैं

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

Proxmark3 अन्य क्रियाएँ करने की अनुमति देता है जैसे कि **इकछुपण** एक **टैग से रीडर संचार** को सुनने के लिए संवेदनशील डेटा ढूंढने का प्रयास करना। इस कार्ड में आप संचार को स्निफ कर सकते हैं और कुंजी का उपयोग कर सकते हैं क्योंकि **रहस्यमय संचालन जो कमजोर हैं** और सादा और चिपर टेक्स्ट को जानकर आप इसे कैलक्यूलेट कर सकते हैं (`mfkey64` टूल)।

### कच्चे कमांड

आईओटी सिस्टम कभी-कभी **गैर-ब्रांडेड या गैर-वाणिज्यिक टैग** का उपयोग करते हैं। इस मामले में, आप Proxmark3 का उपयोग करके टैग्स को अपने लिए **कस्टम कच्चे कमांड भेजने** के लिए कर सकते हैं।

```bash
proxmark3> hf search UID : 80 55 4b 6c ATQA : 00 04
SAK : 08 [2]
TYPE : NXP MIFARE CLASSIC 1k | Plus 2k SL1
proprietary non iso14443-4 card found, RATS not supported
No chinese magic backdoor command detected
Prng detection: WEAK
Valid ISO14443A Tag Found - Quiting Search
```

इस जानकारी के साथ आप कार्ड के बारे में जानकारी खोजने का प्रयास कर सकते हैं और उसके साथ संवाद करने के तरीके के बारे में। Proxmark3 अनुप्रेषित आदेश भेजने की अनुमति देता है जैसे: `hf 14a raw -p -b 7 26`

### स्क्रिप्ट

Proxmark3 सॉफ़्टवेयर के साथ एक पूर्व लोड किया गया **स्वचालन स्क्रिप्ट** सूची आती है जिसका उपयोग आप सरल कार्यों को पूरा करने के लिए कर सकते हैं। पूरी सूची प्राप्त करने के लिए, `script list` कमांड का उपयोग करें। अगले, स्क्रिप्ट के नाम के साथ `script run` कमांड का उपयोग करें:

```
proxmark3> script run mfkeys
```

आप एक स्क्रिप्ट बना सकते हैं ताकि **टैग रीडर** को **फज़ किया** जा सके, तो केवल **एक मान्य कार्ड** के डेटा की प्रतिलिपि बनाएं और एक **Lua स्क्रिप्ट** लिखें जो एक या एक से अधिक **रैंडम बाइट्स** को यादृच्छिक रूप से **रैंडमाइज़** करें और किसी भी अवधि के साथ **रीडर क्रैश** की जांच करें।
