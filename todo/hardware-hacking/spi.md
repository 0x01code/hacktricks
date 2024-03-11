<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप चाहते हैं कि आपकी **कंपनी HackTricks में विज्ञापित हो** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** पर फॉलो करें 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें, HackTricks** को PRs जमा करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>


# मूल जानकारी

SPI (सीरियल पेरिफेरल इंटरफेस) एक सिंक्रोनस सीरियल संचार प्रोटोकॉल है जो एम्बेडेड सिस्टम में उपयोग किया जाता है छोटी दूरी की संचार के लिए आईसी (एकीकृत परिपथ) के बीच। SPI संचार प्रोटोकॉल मास्टर-स्लेव वास्तुकला का उपयोग करता है जिसे घड़ी और चिप सिलेक्ट सिग्नल द्वारा निर्देशित किया जाता है। मास्टर-स्लेव वास्तुकला में एक मास्टर (सामान्यत: एक माइक्रोप्रोसेसर) होता है जो बाह्य पेरिफेरल्स को प्रबंधित करता है जैसे EEPROM, सेंसर, नियंत्रण उपकरण आदि जो स्लेव के रूप में माना जाता है।

एक मास्टर के साथ कई स्लेव्स कनेक्ट किए जा सकते हैं लेकिन स्लेव्स एक-दूसरे से संवाद नहीं कर सकते। स्लेव्स को दो पिन्स द्वारा प्रशासित किया जाता है, क्लॉक और चिप सिलेक्ट। SPI एक सिंक्रोनस संचार प्रोटोकॉल होने के कारण, इनपुट और आउटपुट पिन्स क्लॉक सिग्नल का पालन करते हैं। चिप सिलेक्ट का उपयोग मास्टर द्वारा एक स्लेव का चयन करने और उसके साथ संवाद करने के लिए किया जाता है। जब चिप सिलेक्ट ऊंचा होता है, तो स्लेव उपकरण का चयन नहीं होता है जबकि जब यह नीचा होता है, तो चिप का चयन किया गया होता है और मास्टर स्लेव के साथ संवाद करेगा।

MOSI (मास्टर आउट, स्लेव इन) और MISO (मास्टर इन, स्लेव आउट) डेटा भेजने और प्राप्त करने के लिए जिम्मेदार हैं। डेटा MOSI पिन के माध्यम से स्लेव उपकरण को भेजा जाता है जबकि चिप सिलेक्ट नीचे रखा जाता है। इनपुट डेटा में निर्देश, मेमोरी पते या डेटा शामिल होता है जैसा कि स्लेव उपकरण वेंडर के डेटाशीट के अनुसार होता है। एक वैध इनपुट पर, MISO पिन मास्टर को डेटा भेजने के लिए जिम्मेदार होता है। आउटपुट डेटा इनपुट समाप्त होने के बाद अगले क्लॉक साइकिल पर ठीक भेजा जाता है। MISO पिन डेटा भेजता है जब तक डेटा पूरी तरह से ट्रांसमिटर नहीं होता या मास्टर चिप सिलेक्ट पिन को ऊंचा नहीं कर देता (उस मामले में, स्लेव ट्रांसमिट करना बंद कर देगा और मास्टर उस क्लॉक साइकिल के बाद सुनने वाला नहीं होगा)।

# डंप फ्लैश

## बस पाइरेट + फ्लैशरोम

![](<../../.gitbook/assets/image (201).png>)

ध्यान दें कि यदि पाइरेट बस का पिनआउट **MOSI** और **MISO** के लिए पिन दिखाता है तो कुछ SPIs में पिन्स को DI और DO के रूप में दिखाया जा सकता है। **MOSI -> DI, MISO -> DO**

![](<../../.gitbook/assets/image (648) (1) (1).png>)

Windows या Linux में आप [**`flashrom`**](https://www.flashrom.org/Flashrom) प्रोग्राम का उपयोग करके फ्लैश मेमोरी की सामग्री डंप कर सकते हैं कुछ इस प्रकार कुछ चला कर:
```bash
# In this command we are indicating:
# -VV Verbose
# -c <chip> The chip (if you know it better, if not, don'tindicate it and the program might be able to find it)
# -p <programmer> In this case how to contact th chip via the Bus Pirate
# -r <file> Image to save in the filesystem
flashrom -VV -c "W25Q64.V" -p buspirate_spi:dev=COM3 -r flash_content.img
```
<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

दूसरे तरीके HackTricks का समर्थन करने के लिए:

* अगर आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** पर फॉलो करें 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>
