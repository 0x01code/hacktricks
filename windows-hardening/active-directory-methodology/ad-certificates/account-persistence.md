# AD CS खाता स्थायित्व

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert) के साथ शून्य से नायक तक AWS हैकिंग सीखें</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग तरकीबें साझा करें।

</details>

## प्रमाणपत्रों के माध्यम से सक्रिय उपयोगकर्ता क्रेडेंशियल चोरी – PERSIST1

यदि उपयोगकर्ता को डोमेन प्रमाणीकरण की अनुमति देने वाला प्रमाणपत्र अनुरोध करने की अनुमति है, तो हमलावर इसे **अनुरोध** कर सकता है और **चुरा** सकता है ताकि **स्थायित्व** **बनाए** **रखें**।

**`User`** टेम्पलेट इसकी अनुमति देता है और यह **डिफ़ॉल्ट** रूप से आता है। हालांकि, यह निष्क्रिय हो सकता है। इसलिए, [**Certify**](https://github.com/GhostPack/Certify) आपको मान्य प्रमाणपत्रों को खोजने में मदद करता है जिससे स्थायित्व बना रहे:
```
Certify.exe find /clientauth
```
ध्यान दें कि जब तक प्रमाणपत्र **मान्य** है, तब तक उस प्रमाणपत्र का उपयोग उस उपयोगकर्ता के रूप में प्रमाणीकरण के लिए किया जा सकता है, भले ही उपयोगकर्ता अपना **पासवर्ड** **बदल** दे।

**GUI** से `certmgr.msc` का उपयोग करके या कमांड-लाइन से `certreq.exe` के साथ प्रमाणपत्र का अनुरोध किया जा सकता है।

[**Certify**](https://github.com/GhostPack/Certify) का उपयोग करते हुए आप चला सकते हैं:
```
Certify.exe request /ca:CA-SERVER\CA-NAME /template:TEMPLATE-NAME
```
परिणाम एक **प्रमाणपत्र** + **निजी कुंजी** `.pem` प्रारूपित पाठ का ब्लॉक होगा
```bash
openssl pkcs12 -in cert.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out cert.pfx
```
**उस प्रमाणपत्र का उपयोग करने के लिए**, कोई व्यक्ति `.pfx` को एक लक्ष्य पर **अपलोड** कर सकता है और [**Rubeus**](https://github.com/GhostPack/Rubeus) के साथ **TGT का अनुरोध करने के लिए उपयोग कर सकता है** पंजीकृत उपयोगकर्ता के लिए, जब तक प्रमाणपत्र मान्य है (डिफ़ॉल्ट जीवनकाल 1 वर्ष है):
```bash
Rubeus.exe asktgt /user:harmj0y /certificate:C:\Temp\cert.pfx /password:CertPass!
```
{% hint style="warning" %}
[**THEFT5**](certificate-theft.md#ntlm-credential-theft-via-pkinit-theft5) अनुभाग में बताई गई तकनीक के साथ मिलकर, हमलावर **खाते का NTLM हैश स्थायी रूप से प्राप्त कर सकता है**, जिसका उपयोग हमलावर **pass-the-hash** या **crack** के माध्यम से **plaintext** **पासवर्ड** प्राप्त करने के लिए कर सकता है। \
यह **लंबी अवधि की क्रेडेंशियल चोरी** की एक वैकल्पिक विधि है जो **LSASS को स्पर्श नहीं करती** और यह **गैर-उन्नत संदर्भ** से संभव है।
{% endhint %}

## प्रमाणपत्रों के माध्यम से मशीन स्थायित्व - PERSIST2

यदि किसी प्रमाणपत्र टेम्पलेट ने **Domain Computers** को नामांकन सिद्धांतों के रूप में अनुमति दी हो, तो हमलावर **समझौता किए गए सिस्टम के मशीन खाते का नामांकन कर सकता है**। डिफ़ॉल्ट **`Machine`** टेम्पलेट इन सभी विशेषताओं से मेल खाता है।

यदि कोई **हमलावर समझौता किए गए सिस्टम पर विशेषाधिकार बढ़ाता है**, तो हमलावर मशीन खातों को नामांकन विशेषाधिकार प्रदान करने वाले प्रमाणपत्र टेम्पलेट्स में **SYSTEM** खाते का उपयोग करके नामांकन कर सकता है (अधिक जानकारी [**THEFT3**](certificate-theft.md#machine-certificate-theft-via-dpapi-theft3) में)।

आप [**Certify**](https://github.com/GhostPack/Certify) का उपयोग करके मशीन खाते के लिए प्रमाणपत्र एकत्र कर सकते हैं, जो स्वचालित रूप से SYSTEM में उन्नत हो जाता है:
```bash
Certify.exe request /ca:dc.theshire.local/theshire-DC-CA /template:Machine /machine
```
ध्यान दें कि मशीन अकाउंट सर्टिफिकेट तक पहुंच के साथ, हमलावर **Kerberos को प्रमाणित** कर सकता है मशीन अकाउंट के रूप में। **S4U2Self** का उपयोग करके, हमलावर फिर किसी भी उपयोगकर्ता के रूप में होस्ट पर किसी भी सेवा के लिए **Kerberos सेवा टिकट प्राप्त** कर सकता है (उदाहरण के लिए, CIFS, HTTP, RPCSS, आदि)।

अंततः, यह हमलावर को मशीन पर्सिस्टेंस की विधि देता है।

## सर्टिफिकेट नवीकरण के माध्यम से अकाउंट पर्सिस्टेंस - PERSIST3

सर्टिफिकेट टेम्प्लेट्स में एक **Validity Period** होता है जो यह निर्धारित करता है कि जारी किया गया सर्टिफिकेट कितने समय तक उपयोग किया जा सकता है, साथ ही एक **Renewal period** (आमतौर पर 6 सप्ताह) भी होता है। यह एक **समय की खिड़की** है **इससे पहले** कि सर्टिफिकेट **समाप्त हो** जाए जहां एक **अकाउंट इसे नवीनीकृत** कर सकता है जारी करने वाले सर्टिफिकेट प्राधिकरण से।

यदि हमलावर चोरी या दुर्भावनापूर्ण नामांकन के माध्यम से डोमेन प्रमाणीकरण के लिए सक्षम सर्टिफिकेट को समझौता करता है, तो हमलावर **सर्टिफिकेट की वैधता अवधि के दौरान AD को प्रमाणित** कर सकता है। हालांकि, हमलावर **समाप्ति से पहले सर्टिफिकेट को नवीनीकृत** कर सकता है। यह एक **विस्तारित पर्सिस्टेंस** के रूप में काम कर सकता है जो **अतिरिक्त टिकट** नामांकनों को अनुरोध किए जाने से **रोकता है**, जो **CA सर्वर पर ही आर्टिफैक्ट्स छोड़ सकता है**।

<details>

<summary><strong>Learn AWS hacking from zero to hero with</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें।
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह।
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) **का पालन करें**।
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>
