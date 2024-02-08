# AD प्रमाणपत्र

<details>

<summary><strong>शून्य से हीरो तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) और **मुझे** **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** का** **अनुसरण** करें।
* **अपने हैकिंग ट्रिक्स साझा करें** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos को PR जमा करके।

</details>

## परिचय

### प्रमाणपत्र के घटक

- प्रमाणपत्र का **विषय** इसके मालिक को दर्शाता है।
- एक **सार्वजनिक कुंजी** एक निजी रखी गई कुंजी के साथ जुड़ी होती है ताकि प्रमाणपत्र को उसके सही मालिक से जोड़ा जा सके।
- **वैधता अवधि**, **NotBefore** और **NotAfter** तिथियों द्वारा परिभाषित, प्रमाणपत्र की प्रभावी अवधि को चिह्नित करती है।
- प्रमाणपत्र को पहचानने के लिए प्रमाणपत्र प्राधिकरण (CA) द्वारा प्रदान किया गया एक अद्वितीय **क्रमांक**।
- **जारीकर्ता** CA को संदर्भित करता है जिसने प्रमाणपत्र जारी किया है।
- **SubjectAlternativeName** विषय के लिए अतिरिक्त नामों को समर्थित करता है, पहचान की लचीलाता को बढ़ाता है।
- **मौलिक सीमाएं** प्रमाणपत्र के लिए एक CA या एक अंत इकाई के लिए है और उपयोग सीमाएं परिभाषित करती हैं।
- **विस्तारित कुंजी उपयोग (EKUs)** प्रमाणपत्र के विशिष्ट उद्देश्यों को व्याख्यात करती हैं, जैसे कोड साइनिंग या ईमेल एन्क्रिप्शन, ऑब्ज
```powershell
# Example of requesting a certificate using PowerShell
Get-Certificate -Template "User" -CertStoreLocation "cert:\\CurrentUser\\My"
```
## प्रमाणपत्र प्रमाणीकरण

एक्टिव डायरेक्टरी (AD) प्रमाणपत्र प्रमाणीकरण का समर्थन करती है, मुख्य रूप से **केरबेरोस** और **सुरक्षित चैनल (Schannel)** प्रोटोकॉल का उपयोग करती है।

### केरबेरोस प्रमाणीकरण प्रक्रिया

केरबेरोस प्रमाणीकरण प्रक्रिया में, एक उपयोगकर्ता का टिकट ग्रांटिंग टिकट (TGT) के लिए अनुरोध उपयोगकर्ता के प्रमाणपत्र की **निजी कुंजी** का उपयोग करके हस्ताक्षरित किया जाता है। यह अनुरोध डोमेन कंट्रोलर द्वारा कई मान्यताएँ प्राप्त करता है, जिसमें प्रमाणपत्र की **मान्यता**, **पथ**, और **रोकथाम स्थिति** शामिल हैं। मान्यताएँ इसमें शामिल हैं कि प्रमाणपत्र एक विश्वसनीय स्रोत से आता है और **NTAUTH प्रमाणपत्र स्टोर** में जारीकर्ता की उपस्थिति की पुष्टि करना। सफल मान्यताएँ TGT का जारी होने में समाप्त होती हैं। **`NTAuthCertificates`** ऑब्जेक्ट एडी में, जिसे यहाँ पाया जाता है:
```bash
CN=NTAuthCertificates,CN=Public Key Services,CN=Services,CN=Configuration,DC=<domain>,DC=<com>
```
### प्रमाणपत्र प्रमाणीकरण के लिए विश्वास स्थापित करने के लिए महत्वपूर्ण है।

### सुरक्षित चैनल (Schannel) प्रमाणीकरण

Schannel सुरक्षित TLS/SSL कनेक्शन को सुविधित करता है, जहाँ हैंडशेक के दौरान, ग्राहक एक प्रमाणपत्र प्रस्तुत करता है जो, यदि सफलतापूर्वक मान्यता प्राप्त होता है, पहुंच की अधिकारीकरण करता है। प्रमाणपत्र का एडी खाते से मैपिंग **S4U2Self** फ़ंक्शन या प्रमाणपत्र का **विषय वैकल्पिक नाम (SAN)**, इनमें से अन्य विधियों को शामिल कर सकता है।

### एडी सर्टिफिकेट सेवाएं गणना

एडी के प्रमाणपत्र सेवाएं LDAP क्वेरी के माध्यम से गणना की जा सकती है, जो **एंटरप्राइज प्रमाणपत्र प्राधिकरण (CAs)** और उनके विन्यास के बारे में जानकारी प्रकट करती है। इसे किसी भी डोमेन प्रमाणीकृत उपयोगकर्ता द्वारा विशेष अधिकारों के बिना एक्सेस किया जा सकता है। इसमें **[Certify](https://github.com/GhostPack/Certify)** और **[Certipy](https://github.com/ly4k/Certipy)** जैसे उपकरण एडी सीएस वातावरण में गणना और कमजोरी मूल्यांकन के लिए उपयोग किए जाते हैं।

इन उपकरणों का उपयोग करने के लिए आदेश:
```bash
# Enumerate trusted root CA certificates and Enterprise CAs with Certify
Certify.exe cas
# Identify vulnerable certificate templates with Certify
Certify.exe find /vulnerable

# Use Certipy for enumeration and identifying vulnerable templates
certipy find -vulnerable -u john@corp.local -p Passw0rd -dc-ip 172.16.126.128

# Enumerate Enterprise CAs and certificate templates with certutil
certutil.exe -TCAInfo
certutil -v -dstemplate
```
## संदर्भ

* [https://www.specterops.io/assets/resources/Certified\_Pre-Owned.pdf](https://www.specterops.io/assets/resources/Certified\_Pre-Owned.pdf)
* [https://comodosslstore.com/blog/what-is-ssl-tls-client-authentication-how-does-it-work.html](https://comodosslstore.com/blog/what-is-ssl-tls-client-authentication-how-does-it-work.html)

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** द्वारा PRs सबमिट करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>
