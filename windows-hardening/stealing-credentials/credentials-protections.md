# Windows Credentials Protections

## Credentials Protections

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप चाहते हैं कि आपकी **कंपनी HackTricks में विज्ञापित हो**? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें और PRs सबमिट करें** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को**.

</details>

## WDigest

[WDigest](https://technet.microsoft.com/pt-pt/library/cc778868\(v=ws.10\).aspx?f=255\&MSPPError=-2147217396) प्रोटोकॉल Windows XP में प्रस्तुत किया गया था और इसे प्रमाणीकरण के लिए HTTP प्रोटोकॉल के साथ उपयोग करने के लिए डिज़ाइन किया गया था। माइक्रोसॉफ्ट ने इस प्रोटोकॉल को **विभिन्न संस्करणों के Windows में डिफ़ॉल्ट रूप से सक्षम किया** है (Windows XP - Windows 8.0 और Windows Server 2003 - Windows Server 2012) जिसका मतलब है कि **सादा-पाठ पासवर्ड LSASS में संग्रहीत होते हैं** (स्थानीय सुरक्षा प्राधिकरण उपस्थिति उपकरण सेवा)। **Mimikatz** LSASS के साथ संवाद कर सकता है जिससे हमलावर निम्नलिखित कमांड के माध्यम से **इन प्रमाणिकरण जानकारी को प्राप्त कर सकता है**:
```
sekurlsa::wdigest
```
यह व्यवहार **निष्क्रिय/सक्रिय** किया जा सकता है, _**UseLogonCredential**_ और _**Negotiate**_ की मान को _**HKEY\_LOCAL\_MACHINE\System\CurrentControlSet\Control\SecurityProviders\WDigest**_ में **1** सेट करके।\
यदि ये रजिस्ट्री कुंजी **मौजूद नहीं हैं** या मान **"0"** है, तो WDigest **निष्क्रिय** हो जाएगा।
```
reg query HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest /v UseLogonCredential
```
## LSA सुरक्षा

माइक्रोसॉफ्ट ने **Windows 8.1 और बाद में** LSA की अतिरिक्त सुरक्षा प्रदान की है ताकि अविश्वसनीय प्रक्रियाओं को इसकी मेमोरी को **पढ़ने** या कोड इंजेक्शन करने की अनुमति न मिले। इससे आमतौर पर `mimikatz.exe sekurlsa:logonpasswords` काम नहीं करेगा।\
इस सुरक्षा को **सक्रिय करने** के लिए, आपको _**HKEY\_LOCAL\_MACHINE\SYSTEM\CurrentControlSet\Control\LSA**_ में _**RunAsPPL**_ मान को 1 सेट करना होगा।
```
reg query HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\LSA /v RunAsPPL
```
### बाईपास

Mimikatz ड्राइवर mimidrv.sys का उपयोग करके इस सुरक्षा को बाईपास किया जा सकता है:

![](../../.gitbook/assets/mimidrv.png)

## क्रेडेंशियल गार्ड

**क्रेडेंशियल गार्ड** विंडोज 10 (एंटरप्राइज और एजुकेशन संस्करण) में एक नई सुविधा है जो आपके क्रेडेंशियल को हैश पास की तरह के खतरों से सुरक्षित रखने में मदद करती है। यह एक प्रौद्योगिकी है जिसे वर्चुअलाइज़ेशन CPU के वर्चुअलाइज़ेशन एक्सटेंशन का उपयोग करके काम में लाती है (लेकिन यह वास्तविक वर्चुअल मशीन नहीं है) और **मेमोरी के क्षेत्रों की सुरक्षा** प्रदान करती है (आप इसे वर्चुअलाइज़ेशन आधारित सुरक्षा या VBS के रूप में सुन सकते हैं)। VSM विशेष "बबल" बनाता है जिसमें कुंजी **प्रक्रियाएं** होती हैं जो सामान्य **ऑपरेटिंग सिस्टम** प्रक्रियाओं से **अलग** होती हैं, यहां तक कि कर्नल और **केवल विश्वसनीय प्रक्रियाएं ही प्रक्रियाओं** (जिन्हें **ट्रस्टलेट्स** कहा जाता है) के साथ संवाद कर सकती हैं। इसका मतलब है कि मुख्य ऑपरेटिंग सिस्टम में एक प्रक्रिया VSM की मेमोरी को पढ़ नहीं सकती है, यहां तक कि कर्नल प्रक्रियाएं भी नहीं। **स्थानीय सुरक्षा प्राधिकरण (LSA) विश्वसनीय प्रक्रियाओं** में से एक है जो VSM में होती है, इसके अलावा मुख्य **LSASS** प्रक्रिया भी मुख्य ऑपरेटिंग सिस्टम में चलती है ताकि मौजूदा प्रक्रियाओं के साथ समर्थन सुनिश्चित करें, लेकिन वास्तव में यह केवल एक प्रॉक्सी या स्टब के रूप में काम करती है जो VSM में संवाद करने के लिए होती है और इसे सुनिश्चित करती है कि वास्तविक क्रेडेंशियल VSM में चल रही हैं और इसलिए हमले से सुरक्षित हैं। विंडोज 10 के लिए, क्रेडेंशियल गार्ड को चालू करना और अपने संगठन में डिप्लॉय करना आवश्यक है क्योंकि यह **डिफ़ॉल्ट रूप से सक्षम नहीं होता है**।
[https://www.itprotoday.com/windows-10/what-credential-guard](https://www.itprotoday.com/windows-10/what-credential-guard) से अधिक जानकारी और Credential Guard को सक्षम करने के लिए एक PS1 स्क्रिप्ट [यहां मिल सकती है](https://docs.microsoft.com/en-us/windows/security/identity-protection/credential-guard/credential-guard-manage)। हालांकि, विंडोज 11 एंटरप्राइज, संस्करण 22H2 और विंडोज 11 एजुकेशन, संस्करण 22H2 में शुरू होते हुए, संगत सिस्टमों में विंडोज डिफेंडर क्रेडेंशियल गार्ड [डिफ़ॉल्ट रूप से सक्षम होता है](https://learn.microsoft.com/en-us/windows/security/identity-protection/credential-guard/credential-guard-manage#Default%20Enablement)।

इस मामले में **Mimikatz कुछ नहीं कर सकता** है और LSASS से हैश निकाल सकता है। लेकिन आप हमेशा अपने **कस्टम SSP** को जोड़ सकते हैं और जब एक उपयोगकर्ता साफ-पाठ में लॉगिन करने का प्रयास करता है तो **क्रेडेंशियल** को **कैप्चर** कर सकते हैं।\
अधिक जानकारी [**SSP और इसे कैसे करें यहां**](../active-directory-methodology/custom-ssp.md) मिल सकती है।

क्रेडेंशियल गार्ड को **विभिन्न तरीकों से सक्षम किया जा सकता है**। रजिस्ट्री का उपयोग करके यह जांचने के लिए कि क्या यह सक्षम हुआ है, आप _**HKLM\System\CurrentControlSet\Control\LSA**_ में कुंजी _**LsaCfgFlags**_ के मान की जांच कर सकते हैं। यदि मान **"1"** है, तो यह UEFI लॉक के साथ सक्रिय है, यदि **"2"** है, तो यह लॉक के बिना सक्रिय है और यदि **"0"** है, तो यह सक्षम नहीं है।\
यह **क्रेडेंशियल गार्ड को सक्षम करने के लिए पर्याप्त नहीं है** (लेकिन यह एक मजबूत संकेतक है)।\
अधिक जानकारी और Credential Guard को सक्षम करने के लिए एक PS1 स्क्रिप्ट [यहां मिल सकती है](https://docs.microsoft.com/en-us/windows/security/identity-protection/credential-guard/credential-guard-manage)।
```
reg query HKLM\System\CurrentControlSet\Control\LSA /v LsaCfgFlags
```
## RDP RestrictedAdmin Mode

Windows 8.1 और Windows Server 2012 R2 के साथ, नए सुरक्षा सुविधाएं पेश की गईं। उनमें से एक सुरक्षा सुविधा है _Restricted Admin mode for RDP_। यह नई सुरक्षा सुविधा [pass the hash](https://blog.ahasayen.com/pass-the-hash/) हमलों के जोखिम को कम करने के लिए पेश की गई है।

RDP का उपयोग करके एक दूरस्थ कंप्यूटर से कनेक्ट होने पर, आपके क्रेडेंशियल्स उस दूरस्थ कंप्यूटर पर संग्रहीत होते हैं जिसमें आप RDP कर रहे हैं। आमतौर पर आप दूरस्थ सर्वरों से कनेक्ट होने के लिए एक शक्तिशाली खाता का उपयोग कर रहे होते हैं, और इन सभी कंप्यूटरों पर आपके क्रेडेंशियल्स संग्रहीत होना वास्तव में एक सुरक्षा खतरा है।

_आरडीपी के लिए Restricted Admin mode_ का उपयोग करके, जब आप एक दूरस्थ कंप्यूटर से कनेक्ट होने के लिए आरडीपी का उपयोग करते हैं, तो आप दूरस्थ कंप्यूटर में प्रमाणित हो जाएंगे, लेकिन **आपके क्रेडेंशियल्स उस दूरस्थ कंप्यूटर पर संग्रहीत नहीं होंगे**, जैसा कि पहले होता था। इसका मतलब है कि अगर उस दूरस्थ सर्वर पर मैलवेयर या यहां तक कि एक दुष्ट उपयोगकर्ता सक्रिय है, तो आपके क्रेडेंशियल्स मैलवेयर के हमले के लिए उपलब्ध नहीं होंगे।

ध्यान दें कि जब आपके क्रेडेंशियल्स RDP सत्र पर सहेजे नहीं जा रहे होते हैं और आप **नेटवर्क संसाधनों तक पहुंच करने का प्रयास करते हैं**, तो आपके क्रेडेंशियल्स का उपयोग नहीं होगा। **मशीन पहचान** का उपयोग होगा।

![](../../.gitbook/assets/ram.png)

[यहां से](https://blog.ahasayen.com/restricted-admin-mode-for-rdp/)।

## Cached Credentials

**डोमेन क्रेडेंशियल्स** ऑपरेटिंग सिस्टम के घटकों द्वारा उपयोग किए जाते हैं और **स्थानीय सुरक्षा प्राधिकरण** (LSA) द्वारा **प्रमाणित** किए जाते हैं। आमतौर पर, डोमेन क्रेडेंशियल्स उस समय एक उपयोगकर्ता के लिए स्थापित किए जाते हैं जब एक पंजीकृत सुरक्षा पैकेज उपयोगकर्ता के लॉगऑन डेटा को प्रमाणित करता है। यह पंजीकृत सुरक्षा पैकेज आमतौर पर **Kerberos** प्रोटोकॉल या **NTLM** हो सकता है।

**Windows अंतिम दस डोमेन लॉगिन क्रेडेंशियल्स को संग्रहीत करता है यदि डोमेन कंट्रोलर ऑफ़लाइन हो जाता है**। यदि डोमेन कंट्रोलर ऑफ़लाइन हो जाता है, तो उपयोगकर्ता **अपने कंप्यूटर में लॉगिन करने के लिए अभी भी सक्षम होगा**। यह सुविधा मुख्य रूप से लैपटॉप उपयोगकर्ताओं के लिए है जो नियमित रूप से अपनी कंपनी के डोमेन में लॉगिन नहीं करते हैं। कंप्यूटर द्वारा संग्रहीत क्रेडेंशियल्स की संख्या निम्नलिखित **रजिस्ट्री कुंजी या समूह नीति** के माध्यम से नियंत्रित की जा सकती है:
```bash
reg query "HKEY_LOCAL_MACHINE\SOFTWARE\MICROSOFT\WINDOWS NT\CURRENTVERSION\WINLOGON" /v CACHEDLOGONSCOUNT
```
प्रमाणपत्र साधारित उपयोगकर्ताओं, यहां तक कि प्रशासक खातों से भी, से छिपे हुए हैं। **SYSTEM** उपयोगकर्ता ही एकमात्र उपयोगकर्ता है जिसके **विशेषाधिकार** होते हैं इन **प्रमाणपत्रों** को **देखने** के लिए। रजिस्ट्री में इन प्रमाणपत्रों को देखने के लिए एक प्रशासक को रजिस्ट्री तक पहुंचने के लिए SYSTEM उपयोगकर्ता के रूप में पहुंच होनी चाहिए।\
कैश किए गए प्रमाणपत्र रजिस्ट्री में निम्नलिखित रजिस्ट्री स्थान पर संग्रहीत होते हैं:
```
HKEY_LOCAL_MACHINE\SECURITY\Cache
```
**Mimikatz से निकालना**: `lsadump::cache`\
[यहाँ से](http://juggernaut.wikidot.com/cached-credentials).

## सुरक्षित उपयोगकर्ता

जब साइन इन किए गए उपयोगकर्ता सुरक्षित उपयोगकर्ता समूह का सदस्य होता है, निम्नलिखित सुरक्षा उपयोग की जाती है:

* क्रेडेंशियल डिलीगेशन (CredSSP) उपयोगकर्ता के सादा पाठ क्रेडेंशियल को कैश नहीं करेगा, यहाँ तक कि **अनुमति देने की डिफ़ॉल्ट क्रेडेंशियल को डिलीगेट करने की ग्रुप नीति सेटिंग सक्षम हो**।
* Windows 8.1 और Windows Server 2012 R2 के साथ शुरू होकर, Windows Digest उपयोगकर्ता के सादा पाठ क्रेडेंशियल को कैश नहीं करेगा, यहाँ तक कि Windows Digest सक्षम होने पर भी।
* **NTLM** उपयोगकर्ता के **सादा पाठ क्रेडेंशियल** या NT **एक तरफ़ा कार्य** (NTOWF) को **कैश नहीं करेगा**।
* **Kerberos** अब **DES** या **RC4 कुंजी** नहीं बनाएगा। इसके अलावा, यह **उपयोगकर्ता के सादा पाठ** क्रेडेंशियल या लंबे समय तक की कुंजी को प्रारंभिक TGT प्राप्त करने के बाद कैश नहीं करेगा।
* साइन-इन या अनलॉक करने पर **कैश्ड सत्यापक नहीं बनाया जाता है**, इसलिए ऑफ़लाइन साइन-इन अब समर्थित नहीं है।

जब उपयोगकर्ता खाता सुरक्षित उपयोगकर्ता समूह में जोड़ा जाता है, सुरक्षा सुरक्षा उपयोग करना शुरू होगी जब उपयोगकर्ता उपकरण में साइन इन करता है। **यहाँ से** [**यहाँ**](https://docs.microsoft.com/en-us/windows-server/security/credentials-protection-and-management/protected-users-security-group)**।**

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

**यहाँ से तालिका** [**यहाँ**](https://docs.microsoft.com/en-us/windows-server/identity/ad-ds/plan/security-best-practices/appendix-c--protected-accounts-and-groups-in-active-directory)**।**

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने की अनुमति** चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को**।

</details>
