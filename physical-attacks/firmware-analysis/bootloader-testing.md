<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने की उपलब्धता** चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT संग्रह**](https://opensea.io/collection/the-peass-family)

- [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में PR जमा करके साझा करें**.

</details>


Copied from [https://scriptingxss.gitbook.io/firmware-security-testing-methodology/](https://scriptingxss.gitbook.io/firmware-security-testing-methodology/)

जब डिवाइस की शुरुआत और बूटलोडर्स जैसे U-boot को संशोधित किया जाता है, तो निम्नलिखित कोशिश की जानी चाहिए:

* बूटलोडर्स इंटरप्रेटर शैल का उपयोग करने के लिए "0", स्पेस या अन्य पहचाने गए "मैजिक कोड" दबाएं।
* विन्यासों को संशोधित करें और बूट तर्कों के अंत में '`init=/bin/sh`' जैसे शैल आदेश को निष्पादित करने के लिए।
* `#printenv`
* `#setenv bootargs=console=ttyS0,115200 mem=63M root=/dev/mtdblock3 mtdparts=sflash:<partitiionInfo> rootfstype=<fstype> hasEeprom=0 5srst=0 init=/bin/sh`
* `#saveenv`
* `#boot`
* अपने कार्यस्थल से नेटवर्क के माध्यम से छवियों को लोड करने के लिए एक tftp सर्वर सेटअप करें। सुनिश्चित करें कि डिवाइस के पास नेटवर्क पहुंच है।
* `#setenv ipaddr 192.168.2.2 #डिवाइस का स्थानीय आईपी`
* `#setenv serverip 192.168.2.1 #tftp सर्वर आईपी`
* `#saveenv`
* `#reset`
* `#ping 192.168.2.1 #नेटवर्क पहुंच की उपलब्धता की जांच करें`
* `#tftp ${loadaddr} uImage-3.6.35 #loadaddr दो तर्क लेता है: फ़ाइल को लोड करने के लिए पता और TFTP सर्वर पर छवि का नाम`
* उबूट-इमेज लिखने के लिए `ubootwrite.py` का उपयोग करें और रूट प्राप्त करने के लिए संशोधित फर्मवेयर को पुश करें
* वर्बोज़ लॉगिंग
* विचित्र कर्नल लोड करना
* अविश्वसनीय स्रोतों से बूट करना
* \*सतर्कता बरतें: एक पिन को ग्राउंड से जोड़ें, डिवाइस बूट अप सीक्वेंस को देखें, कर्नल डीकंप्रेस होने से पहले, एसपीआई फ़्लैश चिप पर एक डेटा पिन (DO) पर ग्राउंड करें/जोड़ें
* \*सतर्कता बरतें: एक पिन को ग्राउंड से जोड़ें, डिवाइस बूट अप सीक्वेंस को देखें, कर्नल डीकंप्रेस होने से पहले, एसपीआई फ़्लैश चिप के 8 और 9 नंबर पिन को जोड़ें/जोड़ें, जब U-boot UBI इमेज को डीकंप्रेस करता है
* \*पिन को जोड़ने से पहले NAND फ़्लैश चिप के डेटाशीट की समीक्षा करें
* एक रोग DHCP सर्वर को दुरुपयोगी पैरामीटर के साथ कॉन्फ़िगर करें जो एक डिवाइस को PXE बूट के दौरान सेवन करने के लिए इनपुट होंगे
* Metasploit के (MSF) DHCP अतिरिक्त सर्वर का उपयोग करें और 'FILENAME' पैरामीटर को आदेश संशोधन आदेशों जैसे 'a";/bin/sh;#' के
