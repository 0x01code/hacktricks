# AD CS खाता स्थिरता

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड** करने का उपयोग करना है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)

- प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **फॉलो** करें मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके साझा करें।**

</details>

## प्रमुख उपयोगकर्ता क्रेडेंशियल चोरी प्रमाणपत्र के माध्यम से - PERSIST1

यदि उपयोगकर्ता को डोमेन प्रमाणीकरण की अनुमति होती है, तो हमलावर इसे **अनुरोध** और **चोरी** कर सकता है ताकि **स्थिरता** बनाए रख सके।

**`User`** टेम्पलेट इसे अनुमति देता है और यह **डिफ़ॉल्ट** रूप में आता है। हालांकि, यह अक्षम हो सकता है। इसलिए, [**Certify**](https://github.com/GhostPack/Certify) आपको मान्य प्रमाणपत्र खोजने की अनुमति देता है ताकि स्थिरता बनाए रख सकें:
```
Certify.exe find /clientauth
```
नोट करें कि **प्रमाणपत्र प्रमाणीकरण के लिए उपयोग किया जा सकता है** जब तक प्रमाणपत्र **मान्य हो**, **यहां तक कि** उपयोगकर्ता **अपना** **पासवर्ड बदल दे**।

**GUI** से `certmgr.msc` का उपयोग करके या `certreq.exe` के माध्यम से कोई भी प्रमाणपत्र अनुरोध किया जा सकता है।

[**Certify**](https://github.com/GhostPack/Certify) का उपयोग करके आप निम्नलिखित को चला सकते हैं:
```
Certify.exe request /ca:CA-SERVER\CA-NAME /template:TEMPLATE-NAME
```
परिणाम एक **प्रमाणपत्र** + **निजी कुंजी** `.pem` प्रारूपित पाठ का ब्लॉक होगा।
```bash
openssl pkcs12 -in cert.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out cert.pfx
```
उस प्रमाणपत्र का उपयोग करने के लिए, व्यक्ति उसे टारगेट पर अपलोड कर सकता है और [Rubeus](https://github.com/GhostPack/Rubeus) के साथ इस्तेमाल कर सकता है ताकि पंजीकृत उपयोगकर्ता के लिए एक TGT का अनुरोध कर सके, जब तक प्रमाणपत्र मान्य है (डिफ़ॉल्ट अवधि 1 वर्ष है):
```bash
Rubeus.exe asktgt /user:harmj0y /certificate:C:\Temp\cert.pfx /password:CertPass!
```
{% hint style="warning" %}
इस तकनीक को [**THEFT5**](certificate-theft.md#ntlm-credential-theft-via-pkinit-theft5) अनुभाग में बताए गए तकनीक के साथ मिलाकर, हमलावर एकाउंट के **NTLM हैश को प्राप्त** कर सकता है, जिसे हमलावर **पास-द-हैश** या **क्रैक** करने के लिए उपयोग कर सकता है और **प्लेनटेक्स्ट पासवर्ड** प्राप्त कर सकता है। \
यह एक **लंबे समय तक क्रेडेंशियल चोरी** का वैकल्पिक तरीका है जो **LSASS** को छूने की आवश्यकता नहीं होती है और इसे **नॉन-उच्चीकृत संदर्भ** से संभव है।
{% endhint %}

## प्रमाणपत्रों के माध्यम से मशीन स्थायित्व - PERSIST2

यदि कोई प्रमाणपत्र टेम्पलेट **Domain Computers** को नामांकन सिद्धांत के रूप में अनुमति देता है, तो हमलावर **कंप्यूटर खाते को नामांकित** कर सकता है। डिफ़ॉल्ट **`Machine`** टेम्पलेट इन सभी विशेषताओं के साथ मेल खाता है।

यदि हमलावर किसी सिस्टम पर व्यवस्थापक अधिकार प्राप्त करता है, तो हमलावर **SYSTEM** खाता का उपयोग प्रमाणपत्र टेम्पलेट में नामांकन अधिकार प्रदान करने के लिए कर सकता है (अधिक जानकारी [**THEFT3**](certificate-theft.md#machine-certificate-theft-via-dpapi-theft3) में मिलेगी)।

आप [**Certify**](https://github.com/GhostPack/Certify) का उपयोग कर सकते हैं ताकि मशीन खाते के लिए प्रमाणपत्र एकत्र कर सकें और स्वचालित रूप से SYSTEM में उन्नति कर सकें:
```bash
Certify.exe request /ca:dc.theshire.local/theshire-DC-CA /template:Machine /machine
```
नोट करें कि मशीन खाता प्रमाणपत्र के साथ पहुंच होने पर, हमलावर को फिर मशीन खाते के रूप में **Kerberos में प्रमाणित** करने की सुविधा होती है। **S4U2Self** का उपयोग करके, हमलावर फिर किसी भी सेवा के लिए **Kerberos सेवा टिकट प्राप्त** कर सकता है (जैसे CIFS, HTTP, RPCSS, आदि) के रूप में किसी भी उपयोगकर्ता के रूप में।

अंततः, इससे हमलावर को एक मशीन स्थायित्व विधि मिलती है।

## प्रमाणपत्र नवीनीकरण के माध्यम से खाता स्थायित्व - PERSIST3

प्रमाणपत्र टेम्पलेट में एक **मान्यता अवधि** होती है जो निर्गत प्रमाणपत्र का कितना समय तक उपयोग किया जा सकता है, साथ ही एक **नवीनीकरण अवधि** (आमतौर पर 6 सप्ताह) होती है। यह प्रमाणपत्र **समाप्त होने से पहले** का एक विंडो है जहां से खाता प्रमाणपत्र प्राप्त कर सकता है।

यदि हमलावर किसी प्रमाणपत्र को संप्रदाय प्रमाणीकरण के लिए चोरी या दुर्भाग्यपूर्ण पंजीकरण के माध्यम से हमला करता है, तो हमलावर प्रमाणपत्र की मान्यता अवधि के दौरान **AD में प्रमाणित** हो सकता है। हालांकि, हमलावर **समाप्ति से पहले प्रमाणपत्र को नवीनीकरण** कर सकता है। यह एक **लंबी अवधि की स्थायित्व** दृष्टिकोण के रूप में कार्य कर सकता है जो **अतिरिक्त टिकट** पंजीकरण की अनुरोधनाओं को रोक सकता है, जो CA सर्वर पर **आर्टिफैक्ट** छोड़ सकता है।

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड** करने की सुविधा चाहिए? [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह

- प्राप्त करें [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके साझा करें।**

</details>
