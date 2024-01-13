<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग प्राप्त करें**](https://peass.creator-spring.com)
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग ट्रिक्स शेयर करें।

</details>


# BSSIDs की जांच करें

जब आपको WireShark का उपयोग करते हुए Wifi का मुख्य ट्रैफिक वाला कैप्चर प्राप्त होता है, तो आप _Wireless --> WLAN Traffic_ के साथ कैप्चर के सभी SSIDs की जांच शुरू कर सकते हैं:

![](<../../../.gitbook/assets/image (424).png>)

![](<../../../.gitbook/assets/image (425).png>)

## Brute Force

उस स्क्रीन के कॉलम में से एक यह इंगित करता है कि क्या **pcap के अंदर कोई प्रमाणीकरण पाया गया था**। यदि ऐसा है तो आप `aircrack-ng` का उपयोग करके इसे Brute force करने का प्रयास कर सकते हैं:
```bash
aircrack-ng -w pwds-file.txt -b <BSSID> file.pcap
```
उदाहरण के लिए यह WPA पासफ्रेज प्राप्त करेगा जो PSK (प्री शेयर्ड-की) की सुरक्षा करता है, जिसकी आवश्यकता बाद में ट्रैफिक को डिक्रिप्ट करने के लिए होगी।

# बीकन्स / साइड चैनल में डेटा

यदि आपको संदेह है कि **वाईफाई नेटवर्क के बीकन्स में डेटा लीक हो रहा है** तो आप नेटवर्क के बीकन्स की जांच निम्नलिखित फिल्टर का उपयोग करके कर सकते हैं: `wlan contains <NAMEofNETWORK>`, या `wlan.ssid == "NAMEofNETWORK"` संदिग्ध स्ट्रिंग्स के लिए फिल्टर किए गए पैकेट्स की खोज करें।

# एक वाईफाई नेटवर्क में अज्ञात MAC पते खोजें

निम्नलिखित लिंक **वाईफाई नेटवर्क के अंदर डेटा भेजने वाली मशीनों** को खोजने के लिए उपयोगी होगा:

* `((wlan.ta == e8:de:27:16:70:c9) && !(wlan.fc == 0x8000)) && !(wlan.fc.type_subtype == 0x0005) && !(wlan.fc.type_subtype ==0x0004) && !(wlan.addr==ff:ff:ff:ff:ff:ff) && wlan.fc.type==2`

यदि आप पहले से ही **MAC पते जानते हैं तो आप उन्हें आउटपुट से हटा सकते हैं** इस तरह के चेक जोड़कर: `&& !(wlan.addr==5c:51:88:31:a0:3b)`

एक बार जब आपने **अज्ञात MAC** पते का पता लगा लिया है जो नेटवर्क के अंदर संवाद कर रहे हैं तो आप इसके ट्रैफिक को फिल्टर करने के लिए निम्नलिखित **फिल्टर्स** का उपयोग कर सकते हैं: `wlan.addr==<MAC address> && (ftp || http || ssh || telnet)`। ध्यान दें कि ftp/http/ssh/telnet फिल्टर्स उपयोगी हैं यदि आपने ट्रैफिक को डिक्रिप्ट किया है।

# ट्रैफिक डिक्रिप्ट करें

Edit --> Preferences --> Protocols --> IEEE 802.11--> Edit

![](<../../../.gitbook/assets/image (426).png>)

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert) के साथ शून्य से हीरो तक AWS हैकिंग सीखें</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं या **HackTricks को PDF में डाउनलोड** करना चाहते हैं तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) या [**telegram group**](https://t.me/peass) में **शामिल हों** या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>
