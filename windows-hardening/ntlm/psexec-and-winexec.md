# PsExec/Winexec/ScExec

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे**? या क्या आप **PEASS के नवीनतम संस्करण तक पहुँचना चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) या **Twitter** पर मुझे **फॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपनी हैकिंग ट्रिक्स साझा करें, hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **में PRs सबमिट करके.**

</details>

## वे कैसे काम करते हैं

1. SMB के माध्यम से ADMIN$ शेयर पर एक सेवा बाइनरी की प्रतिलिपि बनाएं
2. बाइनरी की ओर इशारा करते हुए दूरस्थ मशीन पर एक सेवा बनाएं
3. दूरस्थ रूप से सेवा शुरू करें
4. निकास के बाद, सेवा को रोकें और बाइनरी को हटा दें

## **मैन्युअली PsExec'ing**

पहले मान लेते हैं कि हमारे पास एक पेलोड एक्जीक्यूटेबल है जिसे हमने msfvenom के साथ जनरेट किया है और Veil के साथ छिपाया है (ताकि AV इसे फ्लैग न करे). इस मामले में, मैंने एक meterpreter reverse_http पेलोड बनाया और इसे 'met8888.exe' कहा

**बाइनरी की प्रतिलिपि बनाएं**. हमारे "jarrieta" कमांड प्रॉम्प्ट से, सिर्फ बाइनरी को ADMIN$ में कॉपी करें. वास्तव में, इसे फाइलसिस्टम पर कहीं भी कॉपी किया जा सकता है और छिपाया जा सकता है.

![](../../.gitbook/assets/copy\_binary\_admin.png)

**एक सेवा बनाएं**. Windows `sc` कमांड का उपयोग Windows सेवाओं को पूछताछ, बनाने, हटाने आदि के लिए किया जाता है और इसे दूरस्थ रूप से उपयोग किया जा सकता है. इसके बारे में और पढ़ें [यहाँ](https://technet.microsoft.com/en-us/library/bb490995.aspx). हमारे कमांड प्रॉम्प्ट से, हम दूरस्थ रूप से "meterpreter" नामक एक सेवा बनाएंगे जो हमारे अपलोड की गई बाइनरी की ओर इशारा करती है:

![](../../.gitbook/assets/sc\_create.png)

**सेवा शुरू करें**. अंतिम चरण सेवा शुरू करना और बाइनरी को निष्पादित करना है. _नोट:_ जब सेवा शुरू होती है तो यह "समय-सीमा" पर जाएगी और एक त्रुटि उत्पन्न करेगी. यह इसलिए है क्योंकि हमारी meterpreter बाइनरी एक वास्तविक सेवा बाइनरी नहीं है और अपेक्षित प्रतिक्रिया कोड वापस नहीं करेगी. यह ठीक है क्योंकि हमें बस इसे एक बार निष्पादित करने के लिए चाहिए:

![](../../.gitbook/assets/sc\_start\_error.png)

यदि हम हमारे Metasploit लिसनर को देखें, तो हम देखेंगे कि सत्र खोला गया है.

**सेवा को साफ करें.**

![](../../.gitbook/assets/sc\_delete.png)

यहाँ से निकाला गया: [https://blog.ropnop.com/using-credentials-to-own-windows-boxes-part-2-psexec-and-services/](https://blog.ropnop.com/using-credentials-to-own-windows-boxes-part-2-psexec-and-services/)

**आप Windows Sysinternals बाइनरी PsExec.exe का भी उपयोग कर सकते हैं:**

![](<../../.gitbook/assets/image (165).png>)

आप [**SharpLateral**](https://github.com/mertdas/SharpLateral) का भी उपयोग कर सकते हैं:

{% code overflow="wrap" %}
```
SharpLateral.exe redexec HOSTNAME C:\\Users\\Administrator\\Desktop\\malware.exe.exe malware.exe ServiceName
```
<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे**? या क्या आप **PEASS के नवीनतम संस्करण तक पहुँच चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**सदस्यता योजनाओं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) में या **Twitter** पर मुझे **फॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **हैकिंग ट्रिक्स साझा करें, [**hacktricks repo**](https://github.com/carlospolop/hacktricks) और [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके.**

</details>
