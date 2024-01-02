# Windows प्रमाणीकरण सुरक्षा

## प्रमाणीकरण सुरक्षा

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) **का पालन करें.**
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग तरकीबें साझा करें।

</details>

## WDigest

[WDigest](https://technet.microsoft.com/pt-pt/library/cc778868\(v=ws.10\).aspx?f=255\&MSPPError=-2147217396) प्रोटोकॉल को Windows XP में पेश किया गया था और इसे HTTP प्रोटोकॉल के साथ प्रमाणीकरण के लिए डिजाइन किया गया था। Microsoft ने इस प्रोटोकॉल को **डिफ़ॉल्ट रूप से कई संस्करणों में सक्षम किया है** (Windows XP — Windows 8.0 और Windows Server 2003 — Windows Server 2012) जिसका अर्थ है कि **प्लेन-टेक्स्ट पासवर्ड LSASS में संग्रहीत किए जाते हैं** (Local Security Authority Subsystem Service). **Mimikatz** LSASS के साथ इंटरैक्ट कर सकता है जिससे हमलावर निम्नलिखित कमांड के माध्यम से इन प्रमाणीकरणों को **पुनः प्राप्त कर सकता है**:
```
sekurlsa::wdigest
```
यह व्यवहार **निष्क्रिय/सक्रिय किया जा सकता है 1 सेट करके** _**UseLogonCredential**_ और _**Negotiate**_ के मान को _**HKEY\_LOCAL\_MACHINE\System\CurrentControlSet\Control\SecurityProviders\WDigest**_ में।\
यदि ये रजिस्ट्री कुंजियाँ **मौजूद नहीं हैं** या मान **"0"** है, तो WDigest **निष्क्रिय** हो जाएगा।
```
reg query HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest /v UseLogonCredential
```
## LSA सुरक्षा

Microsoft ने **Windows 8.1 और बाद के संस्करणों** में LSA के लिए अतिरिक्त सुरक्षा प्रदान की है ताकि अविश्वसनीय प्रक्रियाएं इसकी मेमोरी को **पढ़ने** या कोड इंजेक्ट करने से **रोक** सकें। इससे सामान्य `mimikatz.exe sekurlsa:logonpasswords` का सही ढंग से काम करना रुक जाएगा।\
इस सुरक्षा को **सक्रिय करने** के लिए आपको _**RunAsPPL**_ का मान _**HKEY\_LOCAL\_MACHINE\SYSTEM\CurrentControlSet\Control\LSA**_ में 1 सेट करना होगा।
```
reg query HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\LSA /v RunAsPPL
```
### बायपास

Mimikatz ड्राइवर mimidrv.sys का उपयोग करके इस सुरक्षा को बायपास करना संभव है:

![](../../.gitbook/assets/mimidrv.png)

## Credential Guard

**Credential Guard** विंडोज 10 (एंटरप्राइज और एजुकेशन संस्करण) में एक नई सुविधा है जो आपके मशीन पर क्रेडेंशियल्स की सुरक्षा करती है, जैसे कि पास द हैश से। यह Virtual Secure Mode (VSM) नामक तकनीक के माध्यम से काम करता है जो CPU के वर्चुअलाइजेशन एक्सटेंशन का उपयोग करता है (लेकिन यह एक वास्तविक वर्चुअल मशीन नहीं है) ताकि **मेमोरी के क्षेत्रों की सुरक्षा प्रदान की जा सके** (आप इसे Virtualization Based Security या VBS के रूप में सुन सकते हैं)। VSM मुख्य **ऑपरेटिंग सिस्टम** की प्रक्रियाओं से अलग एक "बबल" बनाता है, यहां तक कि कर्नेल भी और **केवल विशिष्ट विश्वसनीय प्रक्रियाएं ही VSM में प्रक्रियाओं** (जिन्हें **trustlets** कहा जाता है) से संवाद कर सकती हैं। इसका मतलब है कि मुख्य OS की कोई भी प्रक्रिया VSM की मेमोरी को पढ़ नहीं सकती है, यहां तक कि कर्नेल प्रक्रियाएं भी। **Local Security Authority (LSA) VSM में trustlets में से एक है** इसके अलावा मुख्य OS में चलने वाली सामान्य **LSASS** प्रक्रिया भी है जो मौजूदा प्रक्रियाओं के साथ समर्थन सुनिश्चित करने के लिए है लेकिन वास्तव में यह VSM में संस्करण के साथ संवाद करने के लिए केवल एक प्रॉक्सी या स्टब के रूप में कार्य करता है जिससे वास्तविक क्रेडेंशियल्स VSM के संस्करण पर चलते हैं और इसलिए हमले से सुरक्षित रहते हैं। विंडोज 10 के लिए, Credential Guard को चालू करना होगा और आपके संगठन में तैनात किया जाना चाहिए क्योंकि यह **डिफ़ॉल्ट रूप से सक्षम नहीं है।**
[https://www.itprotoday.com/windows-10/what-credential-guard](https://www.itprotoday.com/windows-10/what-credential-guard) से। Credential Guard को सक्षम करने के लिए और अधिक जानकारी और PS1 स्क्रिप्ट [यहां पाई जा सकती है](https://docs.microsoft.com/en-us/windows/security/identity-protection/credential-guard/credential-guard-manage)। हालांकि, Windows 11 Enterprise, संस्करण 22H2 और Windows 11 Education, संस्करण 22H2 में, संगत सिस्टमों में Windows Defender Credential Guard [डिफ़ॉल्ट रूप से चालू होता है](https://learn.microsoft.com/en-us/windows/security/identity-protection/credential-guard/credential-guard-manage#Default%20Enablement)।

इस मामले में **Mimikatz LSASS से हैशेस निकालने के लिए बहुत कुछ नहीं कर सकता**। लेकिन आप हमेशा अपना **कस्टम SSP** जोड़ सकते हैं और **क्रेडेंशियल्स कैप्चर कर सकते हैं** जब एक उपयोगकर्ता **क्लियर-टेक्स्ट** में लॉगिन करने की कोशिश करता है।\
[**SSP और यह कैसे करें यहां अधिक जानकारी**](../active-directory-methodology/custom-ssp.md)।

Credentials Guard को **विभिन्न तरीकों से सक्षम किया जा सकता है**। यह जांचने के लिए कि यह रजिस्ट्री का उपयोग करके सक्षम किया गया था या नहीं, आप _**HKLM\System\CurrentControlSet\Control\LSA**_ में कुंजी _**LsaCfgFlags**_ के मान की जांच कर सकते हैं। अगर मान **"1"** है तो यह UEFI लॉक के साथ सक्रिय है, अगर **"2"** है तो लॉक के बिना सक्रिय है और अगर **"0"** है तो यह सक्षम नहीं है।\
यह **Credentials Guard को सक्षम करने के लिए पर्याप्त नहीं है** (लेकिन यह एक मजबूत संकेतक है)।\
Credential Guard को सक्षम करने के लिए और अधिक जानकारी और PS1 स्क्रिप्ट [यहां पाई जा सकती है](https://docs.microsoft.com/en-us/windows/security/identity-protection/credential-guard/credential-guard-manage)।
```
reg query HKLM\System\CurrentControlSet\Control\LSA /v LsaCfgFlags
```
## RDP RestrictedAdmin Mode

Windows 8.1 और Windows Server 2012 R2 के साथ, नए सुरक्षा फीचर्स पेश किए गए थे। इन सुरक्षा फीचर्स में से एक है _RDP के लिए Restricted Admin mode_। यह नया सुरक्षा फीचर [pass the hash](https://blog.ahasayen.com/pass-the-hash/) हमलों के जोखिम को कम करने के लिए पेश किया गया है।

जब आप RDP का उपयोग करके किसी दूरस्थ कंप्यूटर से जुड़ते हैं, तो आपकी प्रमाणिकता उस दूरस्थ कंप्यूटर पर संग्रहीत हो जाती है जिसमें आप RDP कर रहे होते हैं। आमतौर पर आप दूरस्थ सर्वरों से जुड़ने के लिए एक शक्तिशाली खाते का उपयोग करते हैं, और आपकी प्रमाणिकता का इन सभी कंप्यूटरों पर संग्रहीत होना वास्तव में एक सुरक्षा खतरा है।

_RDP के लिए Restricted Admin mode_ का उपयोग करते हुए, जब आप कमांड **mstsc.exe /RestrictedAdmin** का उपयोग करके किसी दूरस्थ कंप्यूटर से जुड़ते हैं, तो आप उस दूरस्थ कंप्यूटर के प्रति प्रमाणित होंगे, लेकिन **आपकी प्रमाणिकता उस दूरस्थ कंप्यूटर पर संग्रहीत नहीं की जाएगी**, जैसा कि पहले होता था। इसका मतलब है कि अगर कोई मैलवेयर या यहां तक कि कोई दुर्भावनापूर्ण उपयोगकर्ता उस दूरस्थ सर्वर पर सक्रिय है, तो आपकी प्रमाणिकता उस दूरस्थ डेस्कटॉप सर्वर पर मैलवेयर के हमले के लिए उपलब्ध नहीं होगी।

ध्यान दें कि चूंकि आपकी प्रमाणिकता RDP सत्र पर संग्रहीत नहीं की जा रही है, इसलिए अगर आप **नेटवर्क संसाधनों तक पहुंचने का प्रयास करते हैं** तो आपकी प्रमाणिकता का उपयोग नहीं किया जाएगा। **इसके बजाय मशीन की पहचान का उपयोग किया जाएगा**।

![](../../.gitbook/assets/ram.png)

[यहाँ](https://blog.ahasayen.com/restricted-admin-mode-for-rdp/) से।

## Cached Credentials

**डोमेन प्रमाणिकता** का उपयोग ऑपरेटिंग सिस्टम के घटकों द्वारा किया जाता है और इसे **Local** **Security Authority** (LSA) द्वारा **प्रमाणित** किया जाता है। आमतौर पर, डोमेन प्रमाणिकता एक उपयोगकर्ता के लिए स्थापित की जाती है जब एक पंजीकृत सुरक्षा पैकेज उपयोगकर्ता के लॉगऑन डेटा को प्रमाणित करता है। यह पंजीकृत सुरक्षा पैकेज **Kerberos** प्रोटोकॉल या **NTLM** हो सकता है।

**Windows डोमेन कंट्रोलर ऑफलाइन होने की स्थिति में अंतिम दस डोमेन लॉगिन प्रमाणिकता को संग्रहीत करता है**। अगर डोमेन कंट्रोलर ऑफलाइन हो जाता है, तो एक उपयोगकर्ता **अभी भी अपने कंप्यूटर में लॉग इन कर पाएगा**। यह फीचर मुख्य रूप से लैपटॉप उपयोगकर्ताओं के लिए है जो नियमित रूप से अपनी कंपनी के डोमेन में लॉग इन नहीं करते हैं। कंप्यूटर द्वारा संग्रहीत प्रमाणिकता की संख्या को निम्नलिखित **रजिस्ट्री कुंजी द्वारा, या ग्रुप पॉलिसी के माध्यम से** नियंत्रित किया जा सकता है:
```bash
reg query "HKEY_LOCAL_MACHINE\SOFTWARE\MICROSOFT\WINDOWS NT\CURRENTVERSION\WINLOGON" /v CACHEDLOGONSCOUNT
```
क्रेडेंशियल्स सामान्य उपयोगकर्ताओं से छिपे होते हैं, यहां तक कि एडमिनिस्ट्रेटर खातों से भी। **SYSTEM** उपयोगकर्ता ही एकमात्र उपयोगकर्ता है जिसके पास इन **क्रेडेंशियल्स** को **देखने** के **विशेषाधिकार** होते हैं। रजिस्ट्री में इन क्रेडेंशियल्स को देखने के लिए एक एडमिनिस्ट्रेटर को SYSTEM उपयोगकर्ता के रूप में रजिस्ट्री तक पहुँचना आवश्यक है।\
कैश्ड क्रेडेंशियल्स निम्नलिखित रजिस्ट्री स्थान पर संग्रहीत किए जाते हैं:
```
HKEY_LOCAL_MACHINE\SECURITY\Cache
```
**Mimikatz से निकालना**: `lsadump::cache`\
[यहाँ](http://juggernaut.wikidot.com/cached-credentials) से।

## सुरक्षित उपयोगकर्ता

जब साइन इन किया गया उपयोगकर्ता सुरक्षित उपयोगकर्ता समूह का सदस्य होता है तो निम्नलिखित सुरक्षा लागू होती है:

* क्रेडेंशियल प्रतिनिधित्व (CredSSP) उपयोगकर्ता के सादा पाठ क्रेडेंशियल्स को कैश नहीं करेगा, भले ही **Allow delegating default credentials** समूह नीति सेटिंग सक्षम हो।
* Windows 8.1 और Windows Server 2012 R2 के शुरू होने के साथ, Windows Digest उपयोगकर्ता के सादा पाठ क्रेडेंशियल्स को कैश नहीं करेगा, भले ही Windows Digest सक्षम हो।
* **NTLM** उपयोगकर्ता के **सादा पाठ क्रेडेंशियल्स** या NT **एक-तरफा फंक्शन** (NTOWF) को **कैश नहीं करेगा**।
* **Kerberos** अब **DES** या **RC4 कुंजियाँ** नहीं बनाएगा। साथ ही यह उपयोगकर्ता के सादा पाठ क्रेडेंशियल्स या दीर्घकालिक कुंजियों को प्रारंभिक TGT प्राप्त होने के बाद कैश नहीं करेगा।
* **साइन-इन या अनलॉक के समय कैश्ड सत्यापनकर्ता नहीं बनाया जाता है**, इसलिए ऑफलाइन साइन-इन अब समर्थित नहीं है।

उपयोगकर्ता खाते को सुरक्षित उपयोगकर्ता समूह में जोड़ने के बाद, सुरक्षा तब शुरू होगी जब उपयोगकर्ता डिवाइस में साइन इन करेगा। [**यहाँ**](https://docs.microsoft.com/en-us/windows-server/security/credentials-protection-and-management/protected-users-security-group) से।

| Windows Server 2003 RTM | Windows Server 2003 SP1+ | <p>Windows Server 2012,<br>Windows Server 2008 R2,<br>Windows Server 2008</p> | Windows Server 2016          |
| ----------------------- | ------------------------ | ----------------------------------------------------------------------------- | ---------------------------- |
| Account Operators       | Account Operators        | Account Operators                                                             | Account Operators            |
| Administrator           | Administrator            | Administrator                                                                 | Administrator                |
| Administrators          | Administrators           | Administrators                                                                | Administrators               |
| Backup Operators        | Backup Operators         | Backup Operators                                                              | Backup Operators             |
| Cert Publishers         |                          |                                                                               |                              |
| Domain Admins           | Domain Admins            | Domain Admins                                                                 | Domain Admins                |
| Domain Controllers      | Domain Controllers       | Domain Controllers                                                            | Domain Controllers           |
| Enterprise Admins       | Enterprise Admins        | Enterprise Admins                                                             | Enterprise Admins            |
|                         |                          |                                                                               | Enterprise Key Admins        |
|                         |                          |                                                                               | Key Admins                   |
| Krbtgt                  | Krbtgt                   | Krbtgt                                                                        | Krbtgt                       |
| Print Operators         | Print Operators          | Print Operators                                                               | Print Operators              |
|                         |                          | Read-only Domain Controllers                                                  | Read-only Domain Controllers |
| Replicator              | Replicator               | Replicator                                                                    | Replicator                   |
| Schema Admins           | Schema Admins            | Schema Admins                                                                 | Schema Admins                |
| Server Operators        | Server Operators         | Server Operators                                                              | Server Operators             |

**तालिका [**यहाँ**](https://docs.microsoft.com/en-us/windows-server/identity/ad-ds/plan/security-best-practices/appendix-c--protected-accounts-and-groups-in-active-directory) से।**

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert) के साथ AWS हैकिंग सीखें शून्य से नायक तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सदस्यता योजनाएँ**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) में शामिल हों या मुझे **Twitter** 🐦 पर **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **HackTricks** को अपनी हैकिंग ट्रिक्स साझा करके PRs जमा करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में योगदान दें।

</details>
