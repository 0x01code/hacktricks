<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप अपनी कंपनी को **HackTricks में विज्ञापित करना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** का** **अनुसरण** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, HackTricks** को PRs जमा करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

निम्नलिखित चरणों की सिफारिश की जाती है उपकरण स्टार्टअप कॉन्फ़िगरेशन और बूटलोडर्स जैसे U-boot को संशोधित करने के लिए:

1. **बूटलोडर के इंटरप्रीटर शैल में पहुंचें**:
- बूट के दौरान, "0", स्पेस, या अन्य पहचाने गए "जादू कोड" दबाएं ताकि बूटलोडर के इंटरप्रीटर शैल में पहुंचा जा सके।

2. **बूट आर्ग्यूमेंट को संशोधित करें**:
- निम्नलिखित कमांडों को निष्पादित करें ताकि '`init=/bin/sh`' को बूट आर्ग्यूमेंट में जोड़ा जा सके, जिससे शेल कमांड का निष्पादन संभव हो:
%%%
#printenv
#setenv bootargs=console=ttyS0,115200 mem=63M root=/dev/mtdblock3 mtdparts=sflash:<partitiionInfo> rootfstype=<fstype> hasEeprom=0 5srst=0 init=/bin/sh
#saveenv
#boot
%%%

3. **TFTP सर्वर सेटअप**:
- स्थानीय नेटवर्क पर छवियों को लोड करने के लिए एक TFTP सर्वर को कॉन्फ़िगर करें:
%%%
#setenv ipaddr 192.168.2.2 #उपकरण का स्थानीय आईपी
#setenv serverip 192.168.2.1 #TFTP सर्वर आईपी
#saveenv
#reset
#ping 192.168.2.1 #नेटवर्क एक्सेस की जांच करें
#tftp ${loadaddr} uImage-3.6.35 #loadaddr फ़ाइल को लोड करने के लिए पता लेता है और TFTP सर्वर पर छवि का फ़ाइल नाम लेता है
%%%

4. **`ubootwrite.py` का उपयोग करें**:
- U-boot छवि लिखने और रूट एक्सेस प्राप्त करने के लिए संशोधित फर्मवेयर को ढोंगी करने के लिए `ubootwrite.py` का उपयोग करें।

5. **डीबग सुविधाओं की जांच करें**:
- सत्यापित करें कि डीबग सुविधाएं जैसे verbose लॉगिंग, विविध कर्नेल लोड करना, या अविश्वसनीय स्रोत से बूट करने की सक्षमता सक्षम हैं।

6. **सावधानीपूर्वक हार्डवेयर हस्तक्षेप**:
- उपकरण बूट-अप क्रम के दौरान एक पिन को ग्राउंड से कनेक्ट करते समय सतर्क रहें और SPI या NAND फ्लैश चिप्स के साथ बातचीत करते समय, विशेष रूप से कर्नेल डीकंप्रेस होने से पहले, ध्यानपूर्वक रहें। पिन को शॉर्ट करने से पहले NAND फ्लैश चिप की डेटाशीट की सलाह लें।

7. **रोग डीएचपी सर्वर कॉन्फ़िगर करें**:
- एक रोग डीएचपी सर्वर को सेटअप करें जिसमें डिवाइस एक PXE बूट के दौरान अपशिष्ट पैरामीटर ग्रहण करें। Metasploit के (MSF) DHCP ऑक्सिलरी सर्वर जैसे उपकरण का उपयोग करें। 'FILENAME' पैरामीटर को आदेश इन्जेक्शन कमांडों के साथ संशोधित करें जैसे `'a";/bin/sh;#'` डिवाइस स्टार्टअप प्रक्रियाओं के लिए इनपुट मान्यता का परीक्षण करने के लिए।

**ध्यान दें**: उपकरण पिन के साथ भौतिक बातचीत को शुरू करने वाले चरण (*एस्टेरिक्स के साथ चिह्नित) को उपकरण को नुकसान पहुंचाने से बचने के लिए अत्यधिक सावधानी से निकटता देनी चाहिए।


# संदर्भ
* [https://scriptingxss.gitbook.io/firmware-security-testing-methodology/](https://scriptingxss.gitbook.io/firmware-security-testing-methodology/)

</details>
