# AD प्रमाणपत्र

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके।

</details>

## परिचय

### प्रमाणपत्र के घटक

- प्रमाणपत्र का **विषय** इसके मालिक को दर्शाता है।
- एक **सार्वजनिक कुंजी** एक निजी रखी गई कुंजी के साथ जुड़ी होती है ताकि प्रमाणपत्र को उसके सही मालिक से जोड़ा जा सके।
- **वैधता अवधि**, **NotBefore** और **NotAfter** तिथियों द्वारा परिभाषित, प्रमाणपत्र की प्रभावी अवधि को चिह्नित करती है।
- एक अद्वितीय **क्रमांक**, प्रमाणपत्र प्राधिकरण (CA) द्वारा प्रदान किया गया, प्रत्येक प्रमाणपत्र की पहचान करता है।
- **जारीकर्ता** CA को संदर्भित करता है जिसने प्रमाणपत्र जारी किया है।
- **SubjectAlternativeName** विषय के लिए अतिरिक्त नामों की अनुमति देता है, पहचान की लचीलाता को बढ़ाता है।
- **मौलिक सीमाएं** प्रमाणपत्र के लिए एक CA या अंत इकाई के लिए है और उपयोग सीमाएं परिभाषित करती हैं।
- **विस्तारित कुंजिउज (EKUs)** वस्तु पहचानकर्ताओं (OIDs) के माध्यम से कोड साइनिंग या ईमेल एन्क्रिप्शन जैसे विशेष उद्देश्यों को प्रमाणपत्र की विशिष्टताएँ निर्धारित करती हैं।
- **हस्ताक्षर एल्गोरिथ्म** प्रमाणपत्र के हस्ताक्षर करने के लिए विधि को निर्धारित करता है।
- **हस्ताक्षर** जो जारीकर्ता की निजी कुंजी के साथ बनाया गया है, प्रमाणपत्र की प्रामाणिकता की गारंटी देता है।

### विशेष विचार

- **विषय वैकल्पिक नाम (SANs)** प्रमाणपत्र की अनुपयोगिता को कई पहचानों के लिए विस्तारित करते हैं, जो कई डोमेन्स वाले सर्वरों के लिए महत्वपूर्ण है। सुरक्षित जारीकरण प्रक्रियाएँ आवश्यक हैं ताकि हमलावर विशेषण करने वाले SAN विनिर्माण को बचाया जा सके।

### सर्टिफिकेट प्राधिकरण (CAs) एक्टिव डायरेक्टरी (AD) में

AD CS AD वन में CA प्रमाणपत्रों को मान्यता प्रदान करता है निर्धारित कंटेनर के माध्यम से, प्रत्येक अद्वितीय भूमिका को सेवित करता है:

- **प्रमाणपत्र प्राधिकरण** कंटेनर विश्वसनीय मूल CA प्रमाणपत्रों को रखता है।
- **नामांकन सेवाएं** कंटेनर विस्तारप्रद CA और उनके प्रमाणपत्र नमूने की विवरण प्रदान करता है।
- **NTAuthCertificates** ऑब्जेक्ट AD प्रमाणीकरण के लिए अधिकृत CA प्रमाणपत्रों को शामिल करता है।
- **AIA (Authority Information Access)** कंटेनर मध्यवर्ती और क्रॉस CA प्रमाणपत्रों के साथ प्रमाणपत्र श्रृंखला मान्यता प्रदान करने में सहायक होता है।

### प्रमाणपत्र प्राप्ति: क्लाइंट प्रमाणपत्र अनुरोध प्रवाह

1. अनुरोध प्रक्रिया क्लाइंट्स द्वारा एक एंटरप्राइज CA खोजने से शुरू होती है।
2. एक CSR बनाया जाता है, जिसमें एक सार्वजनिक कुंजी और अन्य विवरण शामिल होते हैं, एक सार्वजनिक-निजी कुंजी जोड़ने के बाद।
3. CA उपलब्ध प्रमाणपत्र नमूनों के खिलाफ CSR का मूल्यांकन करता है, नमूने की अनुमतियों के आधार पर प्रमाणपत्र जारी करता है।
4. मंजूरी प्राप्त होने पर, CA अपनी निजी कुंजी के साथ प्रमाणपत्र को हस्ताक्षरित करता है और इसे क्लाइंट को वापस भेजता है।

### प्रमाणपत्र नमूने

AD में परिभाषित, ये नमूने प्रमाणपत्र जारी करने के लिए सेटिंग्स और अनुमतियों को आउटलाइन करते हैं, जिसमें परमिटेड EKUs और प्रमाणपत्र सेवाओं के लिए प्रवेश या संशोधन के अधिकार शामिल हैं, प्रमाणपत्र सेवाओं का प्रबंधन करने के लिए महत्वपूर्ण है।

## प्रमाणपत्र नामांकन

प्रमाणपत्रों के लिए नामांकन प्रक्रिया एक प्रशासक द्वारा प्रारंभ किया जाता है जो **एक प्रमाणपत्र नमूना बनाता है**, जिसे एक एंटरप्राइज सर्टिफिकेट प्राधिकरण (CA) द्वारा **प्रकाशित** किया जाता है। यह नमूना क्लाइंट नामांकन के लिए उपलब्ध हो जाता है, जिसे एक Active Directory ऑब्ज
```powershell
# Example of requesting a certificate using PowerShell
Get-Certificate -Template "User" -CertStoreLocation "cert:\\CurrentUser\\My"
```
## प्रमाणपत्र प्रमाणीकरण

एक्टिव डायरेक्टरी (AD) प्रमाणपत्र प्रमाणीकरण का समर्थन करती है, मुख्य रूप से **केरबेरोस** और **सुरक्षित चैनल (Schannel)** प्रोटोकॉल का उपयोग करती है।

### केरबेरोस प्रमाणीकरण प्रक्रिया

केरबेरोस प्रमाणीकरण प्रक्रिया में, एक उपयोगकर्ता का टिकट ग्रांटिंग टिकट (TGT) के लिए अनुरोध उपयोगकर्ता के प्रमाणपत्र की **निजी कुंजी** का उपयोग करके हस्ताक्षरित किया जाता है। यह अनुरोध डोमेन कंट्रोलर द्वारा कई मान्यताओं से गुजरता है, जिसमें प्रमाणपत्र की **मान्यता**, **पथ**, और **रोकथाम स्थिति** शामिल हैं। मान्यताएँ इसमें शामिल हैं कि प्रमाणपत्र एक विश्वसनीय स्रोत से आता है और **NTAUTH प्रमाणपत्र स्टोर** में जारीकर्ता की उपस्थिति की पुष्टि करना। सफल मान्यताएँ एक TGT के जारी होने में समाप्त होती हैं। **`NTAuthCertificates`** ऑब्जेक्ट एडी में, यहाँ पाया जाता है:
```bash
CN=NTAuthCertificates,CN=Public Key Services,CN=Services,CN=Configuration,DC=<domain>,DC=<com>
```
### प्रमाणपत्र प्रमाणीकरण के लिए विश्वास स्थापित करने के लिए महत्वपूर्ण है।

### सुरक्षित चैनल (Schannel) प्रमाणीकरण

Schannel सुरक्षित TLS/SSL कनेक्शन को सुविधाजनक बनाता है, जहां हैंडशेक के दौरान क्लाइंट एक प्रमाणपत्र प्रस्तुत करता है जो, यदि सफलतापूर्वक मान्यता प्राप्त होता है, पहुंच की अधिकारी करता है। प्रमाणपत्र को एडी खाते से मैप करना **Kerberos’s S4U2Self** फ़ंक्शन या प्रमाणपत्र का **Subject Alternative Name (SAN)**, इनमें से अन्य विधियों को शामिल कर सकता है।

### एडी सर्टिफिकेट सेवा गणना

एडी के प्रमाणपत्र सेवाएं LDAP क्वेरी के माध्यम से गणित की जा सकती हैं, जो **एंटरप्राइज सर्टिफिकेट अथॉरिटीज (CAs)** और उनके विन्यास के बारे में जानकारी प्रकट करती है। इसे किसी भी डोमेन-प्रमाणीकृत उपयोगकर्ता द्वारा विशेष अधिकारों के बिना पहुंचा जा सकता है। एडी सीएस परिवेश में गणना और कमजोरी मूल्यांकन के लिए उपकरण जैसे **[Certify](https://github.com/GhostPack/Certify)** और **[Certipy](https://github.com/ly4k/Certipy)** का उपयोग किया जाता है।

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

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>
