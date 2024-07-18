# macOS मेमोरी डंपिंग

{% hint style="success" %}
AWS हैकिंग सीखें और प्रैक्टिस करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण AWS रेड टीम एक्सपर्ट (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और प्रैक्टिस करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण GCP रेड टीम एक्सपर्ट (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks का समर्थन करें</summary>

* [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) और **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** को** **फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, हैकट्रिक्स** [**HackTricks**](https://github.com/carlospolop/hacktricks) **और** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **github रेपो में PR जमा करके।**

</details>
{% endhint %}

### [WhiteIntel](https://whiteintel.io)

<figure><img src="../../../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) एक **डार्क-वेब** पर आधारित खोज इंजन है जो **मुफ्त** सुविधाएं प्रदान करता है ताकि यह जांच सकें कि कोई कंपनी या उसके ग्राहकों को **स्टीलर मैलवेयर** द्वारा **कंप्रोमाइज** किया गया है या नहीं।

व्हाइटइंटेल का मुख्य उद्देश्य खाता हथियाने और रैंसमवेयर हमलों का मुकाबला करना है जो जानकारी चोरी करने वाले मैलवेयर से होते हैं।

आप उनकी वेबसाइट चेक कर सकते हैं और **मुफ्त** में उनका इंजन प्रयास कर सकते हैं:

{% embed url="https://whiteintel.io" %}

***

## मेमोरी आर्टिफैक्ट्स

### स्वैप फ़ाइलें

स्वैप फ़ाइलें, जैसे `/private/var/vm/swapfile0`, **जब फिजिकल मेमोरी भर जाती है तो कैश के रूप में काम करती हैं**। जब फिजिकल मेमोरी में और जगह नहीं होती, तो उसका डेटा एक स्वैप फ़ाइल में स्थानांतरित किया जाता है और फिर जैसे ही आवश्यक होता है, यह डेटा फिजिकल मेमोरी में वापस लाया जाता है। एक से अधिक स्वैप फ़ाइलें मौजूद हो सकती हैं, जिनके नाम होते हैं swapfile0, swapfile1, और इसी तरह।

### हाइबर्नेट इमेज

`/private/var/vm/sleepimage` पर स्थित फ़ाइल हाइबर्नेशन मोड के दौरान महत्वपूर्ण है। **जब OS X हाइबर्नेट होता है तो मेमोरी से डेटा इस फ़ाइल में स्टोर किया जाता है**। कंप्यूटर को जागने पर, सिस्टम इस फ़ाइल से मेमोरी डेटा पुनः प्राप्त करता है, जिससे उपयोगकर्ता जहाँ से छोड़े थे वहाँ से जारी रख सकता है।

यह ध्यान देने योग्य है कि आधुनिक MacOS सिस्टमों पर, सुरक्षा कारणों से यह फ़ाइल आम तौर पर एन्क्रिप्टेड होती है, जिससे पुनर्प्राप्ति कठिन होती है।

* यदि स्लीपइमेज के लिए एन्क्रिप्शन सक्षम है या नहीं यह जांचने के लिए, कमांड `sysctl vm.swapusage` चलाई जा सकती है। यह दिखाएगी कि क्या फ़ाइल एन्क्रिप्टेड है या नहीं।

### मेमोरी दबाव लॉग

MacOS सिस्टम में एक और महत्वपूर्ण मेमोरी संबंधित फ़ाइल है **मेमोरी दबाव लॉग**। ये लॉग `/var/log` में स्थित होते हैं और सिस्टम के मेमोरी उपयोग और दबाव घटनाओं के बारे में विस्तृत जानकारी शामिल करते हैं। ये मेमोरी संबंधित मुद्दों का निदान करने या समय के साथ सिस्टम कैसे मेमोरी प्रबंधित करता है को समझने के लिए विशेष रूप से उपयोगी हो सकते हैं।

## osxpmem के साथ मेमोरी डंपिंग

MacOS मशीन में मेमोरी डंप करने के लिए आप [**osxpmem**](https://github.com/google/rekall/releases/download/v1.5.1/osxpmem-2.1.post4.zip) का उपयोग कर सकते हैं।

**नोट**: निम्नलिखित निर्देश केवल इंटेल आर्किटेक्चर वाले Macs के लिए काम करेंगे। यह टूल अब संग्रहीत है और अंतिम रिलीज 2017 में हुआ था। नीचे दिए गए निर्देशों का उपयोग करके डाउनलोड किया गया बाइनरी इंटेल चिप्स को लक्षित करता है क्योंकि 2017 में Apple Silicon मौजूद नहीं था। यह संभव है कि arm64 आर्किटेक्चर के लिए बाइनरी कंपाइल करना संभव हो, लेकिन आपको खुद कोशिश करनी पड़ेगी।
```bash
#Dump raw format
sudo osxpmem.app/osxpmem --format raw -o /tmp/dump_mem

#Dump aff4 format
sudo osxpmem.app/osxpmem -o /tmp/dump_mem.aff4
```
यदि आपको यह त्रुटि मिलती है: `osxpmem.app/MacPmem.kext failed to load - (libkern/kext) authentication failure (file ownership/permissions); check the system/kernel logs for errors or try kextutil(8)` तो आप इसे निम्नलिखित करके ठीक कर सकते हैं:
```bash
sudo cp -r osxpmem.app/MacPmem.kext "/tmp/"
sudo kextutil "/tmp/MacPmem.kext"
#Allow the kext in "Security & Privacy --> General"
sudo osxpmem.app/osxpmem --format raw -o /tmp/dump_mem
```
**अन्य त्रुटियाँ** को "सुरक्षा और गोपनीयता --> सामान्य" में **kext को लोड करने की अनुमति देने** से ठीक किया जा सकता है, बस **अनुमति** दें।

आप इस **वनलाइनर** का उपयोग करके एप्लिकेशन डाउनलोड कर सकते हैं, kext लोड कर सकते हैं और मेमोरी डंप कर सकते हैं:
```bash
sudo su
cd /tmp; wget https://github.com/google/rekall/releases/download/v1.5.1/osxpmem-2.1.post4.zip; unzip osxpmem-2.1.post4.zip; chown -R root:wheel osxpmem.app/MacPmem.kext; kextload osxpmem.app/MacPmem.kext; osxpmem.app/osxpmem --format raw -o /tmp/dump_mem
```
{% endcode %}

### [WhiteIntel](https://whiteintel.io)

<figure><img src="../../../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) एक **डार्क-वेब** पर आधारित खोज इंजन है जो **मुफ्त** सुविधाएं प्रदान करता है ताकि जांच सकें कि कोई कंपनी या उसके ग्राहकों को **स्टीलर मैलवेयर्स** द्वारा **क्षति** पहुंचाई गई है या नहीं।

WhiteIntel का मुख्य उद्देश्य खाता हस्तांतरण और जानकारी चुराने वाले मैलवेयर से होने वाले रैंसमवेयर हमलों का मुकाबला करना है।

आप उनकी वेबसाइट चेक कर सकते हैं और **मुफ्त** में उनका इंजन ट्राय कर सकते हैं:

{% embed url="https://whiteintel.io" %}

{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण AWS रेड टीम एक्सपर्ट (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण GCP रेड टीम एक्सपर्ट (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) जांचें!
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** पर **फॉलो** करें 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **हैकिंग ट्रिक्स साझा करें PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में।

</details>
{% endhint %}
