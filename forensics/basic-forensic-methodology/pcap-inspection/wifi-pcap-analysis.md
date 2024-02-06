<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **Twitter** पर **फॉलो** करें 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें, HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके।

</details>


# BSSIDs की जाँच करें

जब आप वायरशार्क का उपयोग करके एक कैप्चर प्राप्त करते हैं जिसका मुख्य ट्रैफिक Wifi है, तो आप _Wireless --> WLAN Traffic_ के साथ कैप्चर के सभी SSID की जाँच शुरू कर सकते हैं:

![](<../../../.gitbook/assets/image (424).png>)

![](<../../../.gitbook/assets/image (425).png>)

## ब्रूट फोर्स

उस स्क्रीन की एक कॉलम में यह दिखाता है कि **क्या कैप में कोई प्रमाणीकरण मिला है**। यदि ऐसा है तो आप `aircrack-ng` का उपयोग करके इसे Brute force करने की कोशिश कर सकते हैं:
```bash
aircrack-ng -w pwds-file.txt -b <BSSID> file.pcap
```
# उदाहरण के लिए यह PSK (पूर्व साझा कुंजी) को सुरक्षित करने वाले WPA पासफ्रेज़ को पुनः प्राप्त करेगा, जो बाद में ट्रैफिक को डिक्रिप्ट करने के लिए आवश्यक होगा।

# बीकन्स / साइड चैनल में डेटा

यदि आप संदेह कर रहे हैं कि **एक WiFi नेटवर्क के बीकन्स में डेटा लीक हो रहा है** तो आप नेटवर्क के बीकन्स की जांच कर सकते हैं उस फ़िल्टर का उपयोग करके जैसे: `wlan contains <NAMEofNETWORK>`, या `wlan.ssid == "NAMEofNETWORK"` आप संदेशित पैकेट्स के भीतर संदेहास्पद स्ट्रिंग के लिए खोज करें।

# एक WiFi नेटवर्क में अज्ञात MAC पते खोजें

निम्नलिखित लिंक उपयोगी होगा **एक WiFi नेटवर्क के अंदर डेटा भेजने वाली मशीनों को खोजने के लिए**:

* `((wlan.ta == e8:de:27:16:70:c9) && !(wlan.fc == 0x8000)) && !(wlan.fc.type_subtype == 0x0005) && !(wlan.fc.type_subtype ==0x0004) && !(wlan.addr==ff:ff:ff:ff:ff:ff) && wlan.fc.type==2`

यदि आप **MAC पते पहले से जानते हैं तो आप उन्हें आउटपुट से हटा सकते हैं** इस जैसे चेक्स जोड़कर: `&& !(wlan.addr==5c:51:88:31:a0:3b)`

एक बार जब आपने नेटवर्क के अंदर **अज्ञात MAC पते** को पहचान लिया है जो भेजने के लिए आप **फ़िल्टर** जैसे इस जैसे उपयोग कर सकते हैं: `wlan.addr==<MAC address> && (ftp || http || ssh || telnet)` इसका ट्रैफिक फ़िल्टर करने के लिए। ध्यान दें कि ftp/http/ssh/telnet फ़िल्टर उपयोगी हैं अगर आपने ट्रैफिक को डिक्रिप्ट कर लिया है।

# ट्रैफिक को डिक्रिप्ट करें

संपादित करें --> वरीयताएँ --> प्रोटोकॉल --> IEEE 802.11--> संपादित

![](<../../../.gitbook/assets/image (426).png>)
