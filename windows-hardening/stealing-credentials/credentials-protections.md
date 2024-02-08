# Windows Credentials Protections

## Credentials Protections

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

दूसरे तरीके HackTricks का समर्थन करने के लिए:

* अगर आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos को PRs सबमिट करके।

</details>

## WDigest

[WDigest](https://technet.microsoft.com/pt-pt/library/cc778868(v=ws.10).aspx?f=255&MSPPError=-2147217396) प्रोटोकॉल, जो Windows XP के साथ पेश किया गया था, HTTP प्रोटोकॉल के माध्यम से प्रमाणीकरण के लिए डिज़ाइन किया गया है और **Windows XP से Windows 8.0 और Windows Server 2003 से Windows Server 2012 तक** डिफ़ॉल्ट रूप से सक्षम है। इस डिफ़ॉल्ट सेटिंग से **LSASS में प्लेन-टेक्स्ट पासवर्ड स्टोरेज** होती है। हमलावर Mimikatz का उपयोग करके **इन क्रेडेंशियल्स को निकाल सकता है** जो निम्नलिखित को निष्पादित करके:
```bash
sekurlsa::wdigest
```
इस सुविधा को बंद या चालू करने के लिए, _**HKEY\_LOCAL\_MACHINE\System\CurrentControlSet\Control\SecurityProviders\WDigest**_ के भीतर _**UseLogonCredential**_ और _**Negotiate**_ रजिस्ट्री कुंजी को "1" पर सेट किया जाना चाहिए। यदि ये कुंजी **अनुपस्थित है या "0" पर सेट है**, तो WDigest **अक्षम हो जाता है**:
```bash
reg query HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest /v UseLogonCredential
```
## LSA सुरक्षा

**Windows 8.1** से शुरू करके, Microsoft ने LSA की सुरक्षा को **अविश्वसनीय प्रक्रियाओं द्वारा अनधिकृत मेमोरी पठन या कोड इंजेक्शन को ब्लॉक करने** में सुरक्षा को बढ़ा दिया। यह उन आदेशों के सामान्य कार्य को रोकता है जैसे `mimikatz.exe sekurlsa:logonpasswords`। इस **बढ़ी हुई सुरक्षा को सक्षम करने** के लिए, _**HKEY\_LOCAL\_MACHINE\SYSTEM\CurrentControlSet\Control\LSA**_ में _**RunAsPPL**_ मान को 1 पर समायोजित किया जाना चाहिए:
```
reg query HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\LSA /v RunAsPPL
```
### बायपास

इस सुरक्षा को बायपास करना संभव है जिसके लिए Mimikatz ड्राइवर mimidrv.sys का उपयोग किया जा सकता है:

![](../../.gitbook/assets/mimidrv.png)

## क्रेडेंशियल गार्ड

**क्रेडेंशियल गार्ड**, एक विशेषता जो केवल **Windows 10 (एंटरप्राइज और एजुकेशन संस्करण)** के लिए है, **वर्चुअल सुरक्षित मोड (VSM)** और **वर्चुअलाइजेशन बेस्ड सिक्योरिटी (VBS)** का उपयोग करके मशीन क्रेडेंशियल की सुरक्षा को बढ़ाता है। यह CPU वर्चुअलाइजेशन एक्सटेंशन का उपयोग करता है ताकि मुख्य ऑपरेटिंग सिस्टम की पहुंच से दूर, सुरक्षित मेमोरी स्थान में कुंजीय प्रक्रियाओं को अलग किया जा सके। यह अलगाव सुनिश्चित करता है कि कर्नेल भी VSM मेमोरी तक पहुंच नहीं पा सकता, जिससे **पास-द-हैश** जैसे हमलों से क्रेडेंशियल सुरक्षित रहते हैं। **लोकल सुरक्षा प्राधिकरण (LSA)** इस सुरक्षित वातावरण में एक ट्रस्टलेट के रूप में काम करता है, जबकि मुख्य ऑपरेटिंग सिस्टम में **LSASS** प्रक्रिया केवल VSM के LSA के साथ संवाद करती है।

डिफ़ॉल्ट रूप से, **क्रेडेंशियल गार्ड** सक्रिय नहीं है और संगठन में मैन्युअल सक्रियण की आवश्यकता होती है। यह **Mimikatz** जैसे उपकरणों के खिलाफ सुरक्षा को बढ़ाने के लिए महत्वपूर्ण है, जो क्रेडेंशियल निकालने की क्षमता में बाधित होते हैं। हालांकि, कस्टम **सुरक्षा समर्थन प्रदाताओं (SSP)** को जोड़कर क्रेडेंशियल को स्पष्ट पाठ में पकड़ने के लिए विकल्पों का शोध किया जा सकता है।

**क्रेडेंशियल गार्ड** के सक्रियण स्थिति की सत्यापन के लिए, रजिस्ट्री कुंजी **_HKLM\System\CurrentControlSet\Control\LSA_** के तहत **_LsaCfgFlags_** को जांचा जा सकता है। "**1**" का मान सक्रियण को **UEFI लॉक** के साथ दर्शाता है, "**2**" लॉक के बिना, और "**0**" इसे सक्रिय नहीं करता है। यह रजिस्ट्री जांच, जो एक मजबूत संकेतक है, क्रेडेंशियल गार्ड को सक्रिय करने के लिए एकमात्र कदम नहीं है। इस सुविधा को सक्रिय करने के लिए विस्तृत मार्गदर्शन और एक PowerShell स्क्रिप्ट ऑनलाइन उपलब्ध हैं।
```powershell
reg query HKLM\System\CurrentControlSet\Control\LSA /v LsaCfgFlags
```
विस्तृत समझ और निर्देशों के लिए **Windows 10** में **Credential Guard** को सक्षम करने और **Windows 11 Enterprise और Education (संस्करण 22H2)** के संगत सिस्टम में स्वचालित करने के लिए [Microsoft की दस्तावेज़ी](https://docs.microsoft.com/en-us/windows/security/identity-protection/credential-guard/credential-guard-manage) पर जाएं।

Credential कैप्चर के लिए कस्टम SSPs को लागू करने के विवरण [इस गाइड](../active-directory-methodology/custom-ssp.md) में दिए गए हैं।

## RDP RestrictedAdmin Mode

**Windows 8.1 और Windows Server 2012 R2** ने कई नए सुरक्षा सुविधाएँ पेश की, जिसमें से एक है **_RDP के लिए Restricted Admin mode_**। यह मोड **[पास द हैश](https://blog.ahasayen.com/pass-the-hash/)** हमलों से जुड़ी जोखिमों को कम करके सुरक्षा को बढ़ाने के लिए डिज़ाइन किया गया था।

पारंपरिक रूप से, RDP के माध्यम से दूरस्थ कंप्यूटर से कनेक्ट करते समय, आपके credentials लक्षित मशीन पर स्टोर किए जाते हैं। यह खासतौर पर उच्च विशेषाधिकारों वाले खातों का उपयोग करते समय एक महत्वपूर्ण सुरक्षा जोखिम पैदा करता है। हालांकि, **Restricted Admin mode** के परिचय के साथ, यह जोखिम काफी कम हो जाता है।

**Restricted Admin mode** का उपयोग करके RDP कनेक्शन प्रारंभ करते समय, आपके credentials को स्टोर क
```bash
reg query "HKEY_LOCAL_MACHINE\SOFTWARE\MICROSOFT\WINDOWS NT\CURRENTVERSION\WINLOGON" /v CACHEDLOGONSCOUNT
```
इन कैश किए गए क्रेडेंशियल्स तक पहुंच को कड़ी से नियंत्रित किया गया है, केवल **SYSTEM** खाते को इन्हें देखने की आवश्यक अनुमतियाँ हैं। इस जानकारी तक पहुंचने की आवश्यकता होने पर प्रशासकों को SYSTEM उपयोगकर्ता विशेषाधिकारों के साथ इसे देखना होगा। क्रेडेंशियल्स स्टोर किए गए हैं: `HKEY_LOCAL_MACHINE\SECURITY\Cache`

**Mimikatz** का उपयोग कमांड `lsadump::cache` का उपयोग करके इन कैश किए गए क्रेडेंशियल्स को निकालने के लिए किया जा सकता है।

अधिक जानकारी के लिए, मूल [स्रोत](http://juggernaut.wikidot.com/cached-credentials) व्यापक जानकारी प्रदान करता है।

## सुरक्षित उपयोगकर्ता

**सुरक्षित उपयोगकर्ता समूह** में सदस्यता कई सुरक्षा वृद्धि लाता है, उपयोगकर्ताओं के खिलाफ क्रेडेंशियल चोरी और दुरुपयोग के खिलाफ उच्च स्तर की सुरक्षा सुनिश्चित करता है:

- **क्रेडेंशियल डेलीगेशन (CredSSP)**: यदि **डिफ़ॉल्ट क्रेडेंशियल डेलीगेशन की अनुमति देने वाली** समूह नीति सेटिंग सक्षम है, तो सुरक्षित उपयोगकर्ताओं के सादर क्रेडेंशियल कैश नहीं किए जाएंगे।
- **Windows Digest**: **Windows 8.1 और Windows Server 2012 R2** से शुरू करके, सिस्टम सुरक्षित उपयोगकर्ताओं के सादर क्रेडेंशियल कैश नहीं करेगा, चाहे Windows Digest स्थिति हो या न हो।
- **NTLM**: सिस्टम सुरक्षित उपयोगकर्ताओं के सादर क्रेडेंशियल या NT एक-तरफा कार्यों (NTOWF) कैश नहीं करेगा।
- **Kerberos**: सुरक्षित उपयोगकर्ताओं के लिए, Kerberos प्रमाणीकरण **DES** या **RC4 कुंजियाँ** नहीं उत्पन्न करेगा, न ही सादर क्रेडेंशियल या दीर्घकालिक कुंजियाँ टिकट-प्रदान टिकट (TGT) प्राप्ति के बाद से अधिक कैश करेगा।
- **ऑफलाइन साइन-इन**: सुरक्षित उपयोगकर्ताओं के लिए साइन-इन या अनलॉक पर एक कैश निर्माता नहीं बनाया जाएगा, जिसका मतलब है कि इन खातों के लिए ऑफलाइन साइन-इन का समर्थन नहीं है।

ये सुरक्षा उपाय तत्कालिक रूप से एक उपयोगकर्ता, जो **सुरक्षित उपयोगकर्ता समूह** का सदस्य है, उपकरण में साइन इन करता है, तो सुनिश्चित करते हैं कि विभिन्न क्रेडेंशियल कम्प्रोमाइज के विभिन्न तरीकों के खिलाफ सुरक्षा उपाय स्थापित हैं।

अधिक विस्तृत जानकारी के लिए, आधिकारिक [दस्तावेज़](https://docs.microsoft.com/en-us/windows-server/security/credentials-protection-and-management/protected-users-security-group) पर संपर्क करें।
