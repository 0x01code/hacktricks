# डायमंड टिकट

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, HackTricks और HackTricks Cloud** github repos में PRs सबमिट करके।

</details>

## डायमंड टिकट

**जैसे कि एक सोने की टिकट**, एक डायमंड टिकट एक TGT है जिसका उपयोग **किसी भी सेवा तक किसी भी उपयोगकर्ता के रूप में पहुंचने** के लिए किया जा सकता है। एक सोने की टिकट पूरी तरह से ऑफलाइन बनाई जाती है, उस डोमेन के krbtgt हैश के साथ एन्क्रिप्ट की जाती है, और फिर उसे उपयोग के लिए एक लॉगऑन सत्र में पारित किया जाता है। क्योंकि डोमेन कंट्रोलर TGTs को ट्रैक नहीं करते हैं जिन्हें यह (या वे) वास्तव में जारी किया गया है, वे खुशी-खुशी TGTs को स्वीकार करेंगे जो उसके अपने krbtgt हैश के साथ एन्क्रिप्ट किए गए हों।

सोने की टिकट का उपयोग पहचानने के लिए दो सामान्य तकनीक हैं:

* उन TGS-REQs की खोज करें जिनके पास कोई संबंधित AS-REQ नहीं है।
* उन TGTs की खोज करें जिनके पास मिमीकैट्ज की डिफ़ॉल्ट 10 वर्ष की जीवनकाल है जैसे मूर्ख मूल्य हैं।

एक **डायमंड टिकट** एक DC द्वारा जारी किए गए वैध TGT के क्षेत्रों को **संशोधित करके बनाया जाता है**। इसे **एक TGT का अनुरोध करके**, डोमेन के krbtgt हैश के साथ **डिक्रिप्ट** करके, टिकट के वांछित क्षेत्रों को **संशोधित करके**, फिर इसे **पुनः एन्क्रिप्ट** करके प्राप्त किया जाता है। यह एक सोने की टिकट के दो पूर्ववर्ती दोषों को **दूर करता है** क्योंकि:

* TGS-REQs के पहले AS-REQ होगा।
* TGT एक DC द्वारा जारी किया गया था जिसका मतलब है कि यह डोमेन के केरबेरोस नीति से सभी सही विवरण रखेगा। यद्यपि इन्हें सोने की टिकट में सटीक रूप से जाली बनाया जा सकता है, यह अधिक जटिल है और गलतियों के लिए खुला है।
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
<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

दूसरे तरीके HackTricks का समर्थन करने के लिए:

* अगर आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** पर फॉलो करें 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>
