<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, HackTricks** (https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके।

</details>


यदि आपके पास एक USB कनेक्शन का pcap है जिसमें कई बार अवरुद्धियाँ हैं तो शायद यह एक USB कीबोर्ड कनेक्शन हो।

एक वायरशार्क फ़िल्टर जैसे यह उपयोगी हो सकता है: `usb.transfer_type == 0x01 and frame.len == 35 and !(usb.capdata == 00:00:00:00:00:00:00:00)`

यह महत्वपूर्ण हो सकता है कि "02" से शुरू होने वाला डेटा शिफ्ट का उपयोग करके दबाया गया है।

आप इसे विश्लेषित करने के लिए और अधिक जानकारी पढ़ सकते हैं और कुछ स्क्रिप्ट्स भी पा सकते हैं:

* [https://medium.com/@ali.bawazeeer/kaizen-ctf-2018-reverse-engineer-usb-keystrok-from-pcap-file-2412351679f4](https://medium.com/@ali.bawazeeer/kaizen-ctf-2018-reverse-engineer-usb-keystrok-from-pcap-file-2412351679f4)
* [https://github.com/tanc7/HacktheBox\_Deadly\_Arthropod\_Writeup](https://github.com/tanc7/HacktheBox_Deadly_Arthropod_Writeup)

</details>
