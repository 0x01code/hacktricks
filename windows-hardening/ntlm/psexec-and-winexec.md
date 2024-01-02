# PsExec/Winexec/ScExec

<details>

<summary><strong>शून्य से नायक तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) को **फॉलो** करें.
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें.

</details>

## वे कैसे काम करते हैं

1. SMB के माध्यम से ADMIN$ शेयर पर एक सेवा बाइनरी की प्रतिलिपि बनाएं
2. बाइनरी को इंगित करते हुए दूरस्थ मशीन पर एक सेवा बनाएं
3. दूरस्थ रूप से सेवा शुरू करें
4. निकास होने पर, सेवा को रोकें और बाइनरी को हटा दें

## **मैन्युअली PsExec'ing**

पहले मान लेते हैं कि हमारे पास एक पेलोड एक्जीक्यूटेबल है जिसे हमने msfvenom के साथ जनरेट किया है और Veil के साथ छिपाया है (ताकि AV इसे फ्लैग न करे)। इस मामले में, मैंने एक meterpreter reverse_http पेलोड बनाया और इसे 'met8888.exe' कहा

**बाइनरी की प्रतिलिपि बनाएं**। हमारे "jarrieta" कमांड प्रॉम्प्ट से, सिर्फ बाइनरी को ADMIN$ में कॉपी करें। वास्तव में, इसे फाइल सिस्टम पर कहीं भी छिपाया जा सकता है।

**सेवा बनाएं**। Windows `sc` कमांड का उपयोग Windows सेवाओं को पूछताछ करने, बनाने, हटाने आदि के लिए किया जाता है और इसे दूरस्थ रूप से उपयोग किया जा सकता है। इसके बारे में और पढ़ें [यहाँ](https://technet.microsoft.com/en-us/library/bb490995.aspx)। हमारे कमांड प्रॉम्प्ट से, हम दूरस्थ रूप से "meterpreter" नामक एक सेवा बनाएंगे जो हमारे अपलोड की गई बाइनरी को इंगित करती है:

**सेवा शुरू करें**। अंतिम चरण सेवा शुरू करना और बाइनरी को निष्पादित करना है। _नोट:_ जब सेवा शुरू होती है तो यह "समय-सीमा" पर होगी और एक त्रुटि उत्पन्न करेगी। यह इसलिए है क्योंकि हमारी meterpreter बाइनरी एक वास्तविक सेवा बाइनरी नहीं है और अपेक्षित प्रतिक्रिया कोड वापस नहीं करेगी। यह ठीक है क्योंकि हमें बस इसे एक बार निष्पादित करने की आवश्यकता है:

यदि हम हमारे Metasploit लिसनर को देखें, तो हम देखेंगे कि सत्र खोला गया है।

**सेवा साफ करें।**

यहाँ से निकाला गया: [https://blog.ropnop.com/using-credentials-to-own-windows-boxes-part-2-psexec-and-services/](https://blog.ropnop.com/using-credentials-to-own-windows-boxes-part-2-psexec-and-services/)

**आप Windows Sysinternals बाइनरी PsExec.exe का भी उपयोग कर सकते हैं:**

आप [**SharpLateral**](https://github.com/mertdas/SharpLateral) का भी उपयोग कर सकते हैं:

{% code overflow="wrap" %}
```
SharpLateral.exe redexec HOSTNAME C:\\Users\\Administrator\\Desktop\\malware.exe.exe malware.exe ServiceName
```
<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** 🐦 पर **मुझे फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>
