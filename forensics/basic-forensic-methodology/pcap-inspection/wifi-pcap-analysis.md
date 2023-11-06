<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)

- प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **फॉलो** करें मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके साझा करें।**

</details>


# BSSIDs जांचें

जब आप वायरशार्क का उपयोग करके वाईफ़ाई का पकड़ जो प्रमुख ट्रैफ़िक है, प्राप्त करते हैं, तो आप _Wireless --> WLAN Traffic_ के साथ सभी पकड़ के SSID का जांच करना शुरू कर सकते हैं:

![](<../../../.gitbook/assets/image (424).png>)

![](<../../../.gitbook/assets/image (425).png>)

## ब्रूट फ़ोर्स

उस स्क्रीन की एक कॉलम में यह दिखाता है कि **पीकैप में कोई प्रमाणीकरण मिला है या नहीं**। यदि ऐसा है तो आप `aircrack-ng` का उपयोग करके इसे ब्रूट फ़ोर्स करने का प्रयास कर सकते हैं:
```bash
aircrack-ng -w pwds-file.txt -b <BSSID> file.pcap
```
# बीकन्स / साइड चैनल में डेटा

यदि आप संदेह करते हैं कि **एक Wifi नेटवर्क के बीकन्स में डेटा लीक हो रहा है**, तो आप निम्नलिखित फ़िल्टर का उपयोग करके नेटवर्क के बीकन्स की जांच कर सकते हैं: `wlan contains <नेटवर्क का नाम>`, या `wlan.ssid == "नेटवर्क का नाम"` आप संदेहास्पद स्ट्रिंग के लिए फ़िल्टर किए गए पैकेट में खोज करें।

# एक Wifi नेटवर्क में अज्ञात MAC पते खोजें

निम्नलिखित लिंक अज्ञात MAC पते खोजने में उपयोगी होगा:

* `((wlan.ta == e8:de:27:16:70:c9) && !(wlan.fc == 0x8000)) && !(wlan.fc.type_subtype == 0x0005) && !(wlan.fc.type_subtype ==0x0004) && !(wlan.addr==ff:ff:ff:ff:ff:ff) && wlan.fc.type==2`

यदि आप पहले से ही **MAC पते जानते हैं तो आप उन्हें आउटपुट से हटा सकते हैं** इस तरह के चेक्स जोड़कर: `&& !(wlan.addr==5c:51:88:31:a0:3b)`

नेटवर्क के अंदर आपके द्वारा पहचाने गए **अज्ञात MAC पतों** को खोजने के लिए आप निम्नलिखित फ़िल्टर का उपयोग कर सकते हैं: `wlan.addr==<MAC पता> && (ftp || http || ssh || telnet)` इस बात का ध्यान दें कि ftp/http/ssh/telnet फ़िल्टर उपयोगी होते हैं अगर आपने ट्रैफ़िक को डिक्रिप्ट कर लिया है।

# ट्रैफ़िक को डिक्रिप्ट करें

संपादित करें -> वरीयताएँ -> प्रोटोकॉल -> IEEE 802.11 -> संपादित

![](<../../../.gitbook/assets/image (426).png>)





<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करना चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFT संग्रह**](https://opensea.io/collection/the-peass-family)

- प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपने हैकिंग ट्रिक्स साझा करें, [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके**।

</details>
