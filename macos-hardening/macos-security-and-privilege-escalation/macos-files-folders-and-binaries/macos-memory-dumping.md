# macOS मेमोरी डंपिंग

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स शेयर करें।

</details>

## मेमोरी आर्टिफैक्ट्स

### स्वैप फाइल्स

* **`/private/var/vm/swapfile0`**: यह फाइल **कैश के रूप में इस्तेमाल होती है जब भौतिक मेमोरी भर जाती है**। भौतिक मेमोरी में डेटा को स्वैपफाइल में धकेला जाता है और फिर जरूरत पड़ने पर भौतिक मेमोरी में वापस स्वैप किया जाता है। यहां एक से अधिक फाइलें मौजूद हो सकती हैं। उदाहरण के लिए, आप swapfile0, swapfile1, आदि देख सकते हैं।
*   **`/private/var/vm/sleepimage`**: जब OS X **हाइबरनेशन** में जाता है, **मेमोरी में संग्रहित डेटा को sleepimage फाइल में डाल दिया जाता है**। जब उपयोगकर्ता वापस आता है और कंप्यूटर को जगाता है, मेमोरी को sleepimage से पुनर्स्थापित किया जाता है और उपयोगकर्ता वहीं से शुरू कर सकता है जहां वह छोड़ा था।

आधुनिक MacOS सिस्टम्स में डिफ़ॉल्ट रूप से यह फाइल एन्क्रिप्टेड होगी, इसलिए यह पुनर्प्राप्त नहीं हो सकती है।

* हालांकि, इस फाइल का एन्क्रिप्शन अक्षम किया जा सकता है। `sysctl vm.swapusage` का आउटपुट चेक करें।

### osxpmem के साथ मेमोरी डंपिंग

MacOS मशीन में मेमोरी डंप करने के लिए आप [**osxpmem**](https://github.com/google/rekall/releases/download/v1.5.1/osxpmem-2.1.post4.zip) का उपयोग कर सकते हैं।

**नोट**: निम्नलिखित निर्देश केवल Intel आर्किटेक्चर वाले Macs के लिए काम करेंगे। यह टूल अब आर्काइव्ड है और अंतिम रिलीज 2017 में हुई थी। नीचे दिए गए निर्देशों का उपयोग करके डाउनलोड किया गया बाइनरी Intel चिप्स के लिए लक्षित है क्योंकि 2017 में Apple Silicon मौजूद नहीं था। arm64 आर्किटेक्चर के लिए बाइनरी को कंपाइल करना संभव हो सकता है लेकिन आपको खुद ही कोशिश करनी होगी।
```bash
#Dump raw format
sudo osxpmem.app/osxpmem --format raw -o /tmp/dump_mem

#Dump aff4 format
sudo osxpmem.app/osxpmem -o /tmp/dump_mem.aff4
```
यदि आपको यह त्रुटि मिलती है: `osxpmem.app/MacPmem.kext failed to load - (libkern/kext) authentication failure (file ownership/permissions); check the system/kernel logs for errors or try kextutil(8)` आप इसे ठीक कर सकते हैं:
```bash
sudo cp -r osxpmem.app/MacPmem.kext "/tmp/"
sudo kextutil "/tmp/MacPmem.kext"
#Allow the kext in "Security & Privacy --> General"
sudo osxpmem.app/osxpmem --format raw -o /tmp/dump_mem
```
**अन्य त्रुटियां** को "Security & Privacy --> General" में **kext को लोड करने की अनुमति देकर** ठीक किया जा सकता है, बस इसे **अनुमति दें**।

आप इस **oneliner** का उपयोग करके एप्लिकेशन डाउनलोड करने, kext को लोड करने और मेमोरी डंप करने के लिए भी कर सकते हैं:

{% code overflow="wrap" %}
```bash
sudo su
cd /tmp; wget https://github.com/google/rekall/releases/download/v1.5.1/osxpmem-2.1.post4.zip; unzip osxpmem-2.1.post4.zip; chown -R root:wheel osxpmem.app/MacPmem.kext; kextload osxpmem.app/MacPmem.kext; osxpmem.app/osxpmem --format raw -o /tmp/dump_mem
```
<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग प्राप्त करें**](https://peass.creator-spring.com)
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग ट्रिक्स शेयर करें।

</details>
