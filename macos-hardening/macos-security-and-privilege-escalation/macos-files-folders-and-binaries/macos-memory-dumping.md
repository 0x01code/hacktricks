# macOS मेमोरी डंपिंग

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **फॉलो** करें मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को।**

</details>

## मेमोरी आर्टिफैक्ट्स

### स्वैप फ़ाइलें

* **`/private/var/vm/swapfile0`**: यह फ़ाइल **जब फिजिकल मेमोरी भर जाती है तो कैश के रूप में उपयोग होती है**। फिजिकल मेमोरी में डेटा स्वैपफ़ाइल में डाला जाएगा और फिर यदि यह फिर से आवश्यक हो तो फिजिकल मेमोरी में स्वैप वापस लाया जाएगा। इसमें एक से अधिक फ़ाइल हो सकती हैं। उदाहरण के लिए, आप स्वैपफ़ाइल0, स्वैपफ़ाइल1, और इत्यादि देख सकते हैं।
*   **`/private/var/vm/sleepimage`**: जब OS X **निद्रावस्था में जाता है**, **मेमोरी में संग्रहीत डेटा स्लीपइमेज फ़ाइल में रखा जाता है**। जब उपयोगकर्ता वापस आता है और कंप्यूटर को जगाता है, मेमोरी स्लीपइमेज से पुनर्स्थापित की जाती है और उपयोगकर्ता वहीं से जहां से वह छोड़ा था जारी रख सकता है।

आधुनिक MacOS सिस्टम में इस फ़ाइल को डिफ़ॉल्ट रूप से एन्क्रिप्ट किया जाएगा, इसलिए इसे पुनर्प्राप्त करना संभव नहीं हो सकता है।

* हालांकि, इस फ़ाइल की एन्क्रिप्शन अक्षम की जा सकती है। `sysctl vm.swapusage` की जांच करें।

### osxpmem के साथ मेमोरी डंपिंग

MacOS मशीन में मेमोरी डंप करने के लिए आप [**osxpmem**](https://github.com/google/rekall/releases/download/v1.5.1/osxpmem-2.1.post4.zip) का उपयोग कर सकते हैं।

**नोट**: निम्नलिखित निर्देश केवल इंटेल आर्किटेक्चर वाले Mac के लिए काम करेंगे। यह टूल अब संग्रहीत है और अंतिम रिलीज 2017 में हुआ था। निर्देशों का उपयोग करके डाउनलोड किए गए बाइनरी का लक्ष्य 2017 में Apple Silicon के चिप्स को लक्षित करता है। यह संभव है कि आप arm64 आर्किटेक्चर के लिए बाइनरी को कंपाइल कर सकते हैं, लेकिन आपको खुद कोशिश करनी होगी।
```bash
#Dump raw format
sudo osxpmem.app/osxpmem --format raw -o /tmp/dump_mem

#Dump aff4 format
sudo osxpmem.app/osxpmem -o /tmp/dump_mem.aff4
```
यदि आपको यह त्रुटि मिलती है: `osxpmem.app/MacPmem.kext failed to load - (libkern/kext) authentication failure (file ownership/permissions); check the system/kernel logs for errors or try kextutil(8)` तो आप इसे ठीक कर सकते हैं:
```bash
sudo cp -r osxpmem.app/MacPmem.kext "/tmp/"
sudo kextutil "/tmp/MacPmem.kext"
#Allow the kext in "Security & Privacy --> General"
sudo osxpmem.app/osxpmem --format raw -o /tmp/dump_mem
```
**अन्य त्रुटियों** को "सुरक्षा और गोपनीयता --> सामान्य" में **kext को लोड करने की अनुमति देने** से ठीक किया जा सकता है, बस इसे **अनुमति दें**।

आप इस **वनलाइनर** का उपयोग करके एप्लिकेशन को डाउनलोड कर सकते हैं, kext को लोड कर सकते हैं और मेमोरी को डंप कर सकते हैं:

{% code overflow="wrap" %}
```bash
sudo su
cd /tmp; wget https://github.com/google/rekall/releases/download/v1.5.1/osxpmem-2.1.post4.zip; unzip osxpmem-2.1.post4.zip; chown -R root:wheel osxpmem.app/MacPmem.kext; kextload osxpmem.app/MacPmem.kext; osxpmem.app/osxpmem --format raw -o /tmp/dump_mem
```
{% endcode %}

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड** करने का उपयोग करना है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **या** मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें और PRs सबमिट करें** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को।**

</details>
