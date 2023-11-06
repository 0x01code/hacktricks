# डायमंड टिकट

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ हैकट्रिक्स क्लाउड ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 ट्विटर 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ ट्विच 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 यूट्यूब 🎥</strong></a></summary>

- क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **हैकट्रिक्स में विज्ञापित करना** चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करना चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**द पीएस फैमिली**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)

- प्राप्त करें [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **फॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपने हैकिंग ट्रिक्स को [हैकट्रिक्स रेपो](https://github.com/carlospolop/hacktricks) और [हैकट्रिक्स-क्लाउड रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके साझा करें।**

</details>

## डायमंड टिकट

**स्वर्ण टिकट की तरह**, डायमंड टिकट एक TGT है जिसका उपयोग किसी भी सेवा में किसी भी उपयोगकर्ता के रूप में करने के लिए किया जा सकता है। स्वर्ण टिकट पूरी तरह से ऑफलाइन बनाया जाता है, इसे उस डोमेन के krbtgt हैश के साथ एन्क्रिप्ट किया जाता है, और फिर इसे उपयोग के लिए एक लॉगऑन सत्र में पास किया जाता है। क्योंकि डोमेन कंट्रोलर TGT को ट्रैक नहीं करते हैं जिन्हें वह (या वे) वैध रूप से जारी किए हैं, वे खुशी-खुशी TGT को स्वीकार करेंगे जो उसके खुद के krbtgt हैश के साथ एन्क्रिप्ट किया गया है।

स्वर्ण टिकट का उपयोग खोजने के लिए दो सामान्य तकनीकें हैं:

* उसके पास कोई संबंधित AS-REQ न होने वाले TGS-REQ खोजें।
* उसके पास मिमीकेट्ज़ की डिफ़ॉल्ट 10 वर्ष की अवधि जैसे मूर्ख मान्यताएँ रखने वाले TGT खोजें।

एक **डायमंड टिकट** एक DC द्वारा जारी किए गए वैध TGT के क्षेत्रों को संशोधित करके बनाया जाता है। इसे एक TGT का अनुरोध करके प्राप्त किया जाता है, डोमेन के krbtgt हैश के साथ इसे डिक्रिप्ट किया जाता है, टिकट के वांछित क्षेत्रों को संशोधित किया जाता है, और फिर इसे पुनः एन्क्रिप्ट किया जाता है। यह एक स्वर्ण टिकट की दो पहले उल्लिखित कमियों को पार करता है क्योंकि:

* TGS-REQ के पहले AS-REQ होंगे।
* TGT एक DC द्वारा जारी किया गया था, जिसका मतलब है कि यह डोमेन के केरबेरोस नीति के सभी सही विवरण रखेगा। हालांकि, ये स्वर्ण टिकट में सटीकता से जाली बनाई जा सकती हैं, यह अधिक जटिल होता है और गलतियों के लिए खुला होता है।
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
\






<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करना चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)

- प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या **Twitter** पर **फॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके साझा करें।**

</details>
