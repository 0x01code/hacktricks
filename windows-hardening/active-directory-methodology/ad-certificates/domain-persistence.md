# एडी सीएस डोमेन स्थिरता

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें और PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos को अपडेट करें।

</details>

**यह [https://www.specterops.io/assets/resources/Certified\_Pre-Owned.pdf](https://www.specterops.io/assets/resources/Certified\_Pre-Owned.pdf) में साझा की गई डोमेन स्थिरता तकनीकों का सारांश है**। अधिक विवरण के लिए इसे देखें।

## चोरी गई सीए प्रमाणपत्रों के साथ प्रमाणपत्र बनाना - DPERSIST1

एक प्रमाणपत्र को CA प्रमाणपत्र मानने के लिए आप कैसे कह सकते हैं?

यह तय किया जा सकता है कि एक प्रमाणपत्र एक CA प्रमाणपत्र है अगर कई शर्तें पूरी होती हैं:

- प्रमाणपत्र CA सर्वर पर संग्रहीत है, जिसकी निजी कुंजी को मशीन के DPAPI द्वारा सुरक्षित किया गया है, या यदि ऑपरेटिंग सिस्टम इसे समर्थन करता है तो TPM/HSM जैसे हार्डवेयर द्वारा।
- प्रमाणपत्र के इश्यूर और सब्जेक्ट फ़ील्ड CA के विभाजन नाम से मेल खाते हैं।
- "CA संस्करण" एक्सटेंशन केवल CA प्रमाणपत्रों में मौजूद है।
- प्रमाणपत्र में Extended Key Usage (EKU) फ़ील्ड गायब है।

इस प्रमाणपत्र की निजी कुंजी को निकालने के लिए, CA सर्वर पर `certsrv.msc` उपकरण बिल्ट-इन GUI के माध्यम से समर्थित विधि है। फिर भी, यह प्रमाणपत्र सिस्टम में संग्रहीत अन्य प्रमाणपत्रों से अलग नहीं है; इसलिए, निकालने के लिए [THEFT2 तकनीक](certificate-theft.md#user-certificate-theft-via-dpapi-theft2) की तकनीक लागू की जा सकती है।

इस प्रमाणपत्र और निजी कुंजी को निम्नलिखित कमांड के साथ Certipy का उपयोग करके प्राप्त किया जा सकता है:
```bash
certipy ca 'corp.local/administrator@ca.corp.local' -hashes :123123.. -backup
```
जब CA प्रमाणपत्र और इसकी निजी कुंजी को `.pfx` प्रारूप में प्राप्त किया जाता है, तो [ForgeCert](https://github.com/GhostPack/ForgeCert) जैसे उपकरण का उपयोग करके मान्य प्रमाणपत्र उत्पन्न किए जा सकते हैं:
```bash
# Generating a new certificate with ForgeCert
ForgeCert.exe --CaCertPath ca.pfx --CaCertPassword Password123! --Subject "CN=User" --SubjectAltName localadmin@theshire.local --NewCertPath localadmin.pfx --NewCertPassword Password123!

# Generating a new certificate with certipy
certipy forge -ca-pfx CORP-DC-CA.pfx -upn administrator@corp.local -subject 'CN=Administrator,CN=Users,DC=CORP,DC=LOCAL'

# Authenticating using the new certificate with Rubeus
Rubeus.exe asktgt /user:localdomain /certificate:C:\ForgeCert\localadmin.pfx /password:Password123!

# Authenticating using the new certificate with certipy
certipy auth -pfx administrator_forged.pfx -dc-ip 172.16.126.128
```
{% hint style="warning" %}
उस उपयोगकर्ता को प्रमाणपत्र जालीकरण के लिए लक्षित किया जाना चाहिए जो सक्रिय हो और सक्षम हो Active Directory में प्रमाणीकरण के लिए सफलता प्राप्त करने के लिए। krbtgt जैसे विशेष खातों के लिए प्रमाणपत्र जालीकरण करना अप्रभावी है।
{% endhint %}

यह जालीकृत प्रमाणपत्र **वैध** होगा जब तक निर्दिष्ट समाप्ति तिथि तक होगा और **मूल सीए प्रमाणपत्र वैध** होगा (सामान्यत: 5 से **10+ वर्ष** तक)। यह **मशीनों** के लिए भी वैध है, इसलिए **S4U2Self** के साथ मिलाकर, एक हमलावर किसी भी डोमेन मशीन पर **स्थिरता बनाए रख सकता है** जब तक सीए प्रमाणपत्र वैध है।\
इसके अतिरिक्त, इस विधि से **उत्पन्न प्रमाणपत्रों** को **रद्द नहीं किया जा सकता** क्योंकि सीए उनके बारे में जागरूक नहीं है।

## धोखाधड़ी सीए प्रमाणपत्रों पर विश्वास - DPERSIST2

`NTAuthCertificates` ऑब्जेक्ट को एक या एक से अधिक **सीए प्रमाणपत्रों** को शामिल करने के लिए परिभाषित किया गया है जिसे एक्टिव डायरेक्टरी (एडी) का उपयोग करता है। डोमेन कंट्रोलर द्वारा सत्यापन प्रक्रिया में शामिल है `NTAuthCertificates` ऑब्जेक्ट को चेक करना जो प्रमाणीकरण के आईशूअर फ़ील्ड में निर्दिष्ट सीए से मेल खाता है। यदि कोई मिलता है तो प्रमाणीकरण जारी रहता है।

एक आत्म-साइन किया गया सीए प्रमाणपत्र एक हमलावर द्वारा `NTAuthCertificates` ऑब्जेक्ट में जोड़ा जा सकता है, प्रदान किया गया कि उनके पास इस एडी ऑब्ज
