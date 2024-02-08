# macOS मेमोरी डंपिंग

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें और PR जमा करें** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

## मेमोरी आर्टिफैक्ट्स

### स्वैप फ़ाइलें

स्वैप फ़ाइलें, जैसे `/private/var/vm/swapfile0`, **जब फिजिकल मेमोरी भर जाती है तो कैश के रूप में काम करती हैं**। जब फिजिकल मेमोरी में और जगह नहीं होती, तो उसका डेटा एक स्वैप फ़ाइल में स्थानांतरित किया जाता है और फिर आवश्यकतानुसार फिजिकल मेमोरी में वापस लाया जाता है। एक से अधिक स्वैप फ़ाइलें मौजूद हो सकती हैं, जिनके नाम हो सकते हैं swapfile0, swapfile1, आदि।

### हाइबर्नेट इमेज

`/private/var/vm/sleepimage` पर स्थित फ़ाइल **हाइबर्नेशन मोड** के दौरान महत्वपूर्ण है। **OS X हाइबर्नेट होते समय मेमोरी से डेटा इस फ़ाइल में स्टोर किया जाता है**। कंप्यूटर को जागने पर, सिस्टम इस फ़ाइल से मेमोरी डेटा पुनः प्राप्त करता है, जिससे उपयोगकर्ता जहाँ छोड़े थे वहाँ से जारी रख सकता है।

यह ध्यान देने योग्य है कि आधुनिक MacOS सिस्टमों पर, सुरक्षा कारणों से यह फ़ाइल सामान्यत: एन्क्रिप्टेड होती है, जिससे पुनर्प्राप्ति कठिन होती है।

* यदि स्लीपइमेज के लिए एन्क्रिप्शन सक्षम है या नहीं यह जांचने के लिए, कमांड `sysctl vm.swapusage` चलाई जा सकती है। यह दिखाएगा कि क्या फ़ाइल एन्क्रिप्टेड है।

### मेमोरी दबाव लॉग

MacOS सिस्टम में एक और महत्वपूर्ण मेमोरी संबंधित फ़ाइल है **मेमोरी दबाव लॉग**। ये लॉग `/var/log` में स्थित होते हैं और सिस्टम के मेमोरी उपयोग और दबाव घटनाओं के बारे में विस्तृत जानकारी शामिल करते हैं। ये मेमोरी संबंधित मुद्दों का निदान करने या समय के साथ सिस्टम के मेमोरी का प्रबंधन कैसे करता है इसको समझने के लिए विशेष रूप से उपयोगी हो सकते हैं।

## osxpmem के साथ मेमोरी डंपिंग

MacOS मशीन में मेमोरी डंप करने के लिए आप [**osxpmem**](https://github.com/google/rekall/releases/download/v1.5.1/osxpmem-2.1.post4.zip) का उपयोग कर सकते हैं।

**ध्यान दें**: निम्नलिखित निर्देश केवल इंटेल आर्किटेक्चर वाले Macs के लिए काम करेंगे। यह टूल अब संग्रहीत है और अंतिम रिलीज 2017 में हुआ था। नीचे दिए गए निर्देशों का उपयोग करके डाउनलोड किया गया बाइनरी इंटेल चिप्स को लक्षित करता है क्योंकि 2017 में Apple Silicon मौजूद नहीं था। यह संभावना है कि arm64 आर्किटेक्चर के लिए बाइनरी को कंपाइल करना संभव है, लेकिन आपको खुद प्रयास करना होगा।
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

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

दूसरे तरीके HackTricks का समर्थन करने के लिए:

* अगर आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** द्वारा PRs सबमिट करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>
