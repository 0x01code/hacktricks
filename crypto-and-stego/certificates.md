# प्रमाणपत्र

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सदस्यता योजनाएँ देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>

<figure><img src="../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm_source=hacktricks&utm_medium=text&utm_campaign=ppc&utm_term=trickest&utm_content=certificates) का उपयोग करें और **दुनिया के सबसे उन्नत समुदाय उपकरणों द्वारा संचालित** **कार्यप्रवाह** आसानी से बनाएं और स्वचालित करें।\
आज ही पहुंच प्राप्त करें:

{% embed url="https://trickest.com/?utm_source=hacktricks&utm_medium=banner&utm_campaign=ppc&utm_content=certificates" %}

## प्रमाणपत्र क्या है

**पब्लिक की प्रमाणपत्र** एक डिजिटल आईडी है जिसका उपयोग गणित्र में किसी को पब्लिक की प्रमाणपत्र के मालिक होने का सबूत देने के लिए किया जाता है। इसमें कुंजी के विवरण, मालिक की पहचान (विषय), और एक विश्वसनीय प्राधिकारी से डिजिटल हस्ताक्षर शामिल है। यदि सॉफ़्टवेयर प्राधिकारी पर भरोसा करता है और हस्ताक्षर मान्य है, तो कुंजी के मालिक के साथ सुरक्षित संचार संभव है।

प्रमाणपत्रों को अधिकांशत: [प्रमाणपत्र प्राधिकारियों](https://en.wikipedia.org/wiki/Certificate\_authority) (CAs) द्वारा [सार्वजनिक-कुंजी प्रणाली](https://en.wikipedia.org/wiki/Public-key\_infrastructure) (PKI) सेटअप में जारी किया जाता है। एक और विधि है [विश्व विश्वास](https://en.wikipedia.org/wiki/Web\_of\_trust), जहां उपयोगकर्ता सीधे एक-दूसरे की कुंजियों की पुष्टि करते हैं। प्रमाणपत्रों के लिए सामान्य प्रारूप है [X.509](https://en.wikipedia.org/wiki/X.509), जो RFC 5280 में उल्लिखित विशेष आवश्यकताओं के लिए समायोजित किया जा सकता है।

## x509 सामान्य क्षेत्र

### **x509 प्रमाणपत्रों में सामान्य क्षेत्र**

x509 प्रमाणपत्रों में कई **क्षेत्र** प्रमाणपत्र की मान्यता और सुरक्षा सुनिश्चित करने में महत्वपूर्ण भूमिका निभाते हैं। यहाँ इन क्षेत्रों का विवरण है:

* **संस्करण संख्या** x509 प्रारूप के संस्करण को दर्शाती है।
* **क्रमांक** प्रमाणपत्र को संकेतित करता है जो प्रमाणपत्र प्राधिकारी (CA) प्रणाली के भीतर एकदमिक रूप से पहचान करने के लिए मुख्य रूप से रोकथाम ट्रैकिंग के लिए होता है।
* **विषय** क्षेत्र प्रमाणपत्र के मालिक को प्रस्तुत करता है, जो एक मशीन, एक व्यक्ति, या एक संगठन हो सकता है। इसमें विस्तृत पहचान शामिल है जैसे:
* **सामान्य नाम (CN)**: प्रमाणपत्र द्वारा कवर किए गए डोमेन।
* **देश (C)**, **स्थान (L)**, **राज्य या प्रांत (ST, S, या P)**, **संगठन (O)**, और **संगठनिक इकाई (OU)** भौगोलिक और संगठनात्मक विवरण प्रदान करते हैं।
* **विभाजित नाम (DN)** पूर्ण विषय पहचान को आवरित करता है।
* **जारीकर्ता** वह व्यक्ति कौन है जिन्होंने प्रमाणपत्र की पुष्टि की और साइन किया है, सीए के लिए विषय के लिए समान उपक्षेत्रों को समेत।
* **वैधता अवधि** को **Not Before** और **Not After** टाइमस्टैम्प्स द्वारा चिह्नित किया जाता है, जिससे प्रमाणपत्र का उपयोग किसी निश्चित तारीख से पहले या बाद में नहीं किया जाता है।
* **सार्वजनिक कुंजी** खंड, प्रमाणपत्र की सुरक्षा के लिए महत्वपूर्ण है, सार्वजनिक कुंजी के एल्गोरिथ्म, आकार, और अन्य तकनीकी विवरण को निर्दिष्ट करता है।
* **x509v3 एक्सटेंशन्स** प्रमाणपत्र की कार्यक्षमता को बढ़ाते हैं, **Key Usage**, **Extended Key Usage**, **Subject Alternative Name**, और प्रमाणपत्र के अन्य गुणों को निर्दिष्ट करने के लिए गुणवत्ता को समायोजित करने के लिए।

#### **कुंजी उपयोग और एक्सटेंशन्स**

* **कुंजी उपयोग** पब्लिक कुंजी के गणितीय अनुप्रयोगों की पहचान करता है, जैसे डिजिटल हस्ताक्षर या कुंजी एन्सिफरमेंट।
* **विस्तारित कुंजी उपयोग** प्रमाणपत्र के उपयोग मामलों को और अधिक संकुचित करता है, जैसे, TLS सर्वर प्रमाणीकरण के लिए।
* **विषय वैकल्पिक नाम** और **मौलिक सीमा** प्रमाणपत्र द्वारा कवर किए गए अतिरिक्त होस्ट नामों को परिभाषित करते हैं और यह निर्धारित करते हैं कि यह एक सीए या अंत-प्राणी प्रमाणपत्र है।
* **विषय कुंजी पहचानकर्ता** और **प्राधिकार कुंजी पहचानकर्ता** कुंजियों की अद्वितीयता और प्रत्यायन की निश्चितता सुनिश्चित करते हैं।
* **प्राधिकार सूचना पहुंच** और **CRL वितरण बिंदु** प्रमाणपत्र की प्रमाणपत्र जांच करने के लिए मार्ग प्रदान करते हैं और प्रमाणपत्र रोकने की स्थिति की जांच करते हैं।
* **CT Precertificate SCTs** प्रमाणपत्र में सार्वजनिक विश्वास के लिए महत्वपूर्ण ट्रांसपेरेंसी लॉग्स प्रदान करते हैं।
```python
# Example of accessing and using x509 certificate fields programmatically:
from cryptography import x509
from cryptography.hazmat.backends import default_backend

# Load an x509 certificate (assuming cert.pem is a certificate file)
with open("cert.pem", "rb") as file:
cert_data = file.read()
certificate = x509.load_pem_x509_certificate(cert_data, default_backend())

# Accessing fields
serial_number = certificate.serial_number
issuer = certificate.issuer
subject = certificate.subject
public_key = certificate.public_key()

print(f"Serial Number: {serial_number}")
print(f"Issuer: {issuer}")
print(f"Subject: {subject}")
print(f"Public Key: {public_key}")
```
### **OCSP और CRL वितरण बिंदुओं के बीच अंतर**

**OCSP** (**RFC 2560**) एक ग्राहक और एक प्रतिक्रियावादी को साथ में काम करने के लिए शामिल करता है ताकि एक डिजिटल सार्वजनिक-कुंजी प्रमाणपत्र को रद्द किया गया है या नहीं जांच सकें, पूरी **CRL** को डाउनलोड करने की आवश्यकता न हो। यह विधि पारंपरिक **CRL** से अधिक दक्ष है, जो रद्द प्रमाणपत्र सीरियल नंबर की सूची प्रदान करता है लेकिन एक संभावित बड़ी फ़ाइल को डाउनलोड करने की आवश्यकता होती है। CRLs में तकरीबन 512 एंट्रीज शामिल की जा सकती हैं। अधिक विवरण यहाँ उपलब्ध है [here](https://www.arubanetworks.com/techdocs/ArubaOS%206\_3\_1\_Web\_Help/Content/ArubaFrameStyles/CertRevocation/About\_OCSP\_and\_CRL.htm).

### **प्रमाणपत्र पारदर्शिता क्या है**

प्रमाणपत्र पारदर्शिता प्रमाणपत्र संबंधित खतरों का मुकाबला करने में मदद करती है जिसमें SSL प्रमाणपत्रों की प्रक्रिया और मौजूदगी डोमेन मालिकों, सीए, और उपयोगकर्ताओं को दिखाई देती है। इसके उद्देश्य हैं:

* सीए को डोमेन के लिए SSL प्रमाणपत्र जारी करने से रोकना जिसके लिए डोमेन के मालिक की जानकारी की आवश्यकता होती है।
* गलती से या दुर्भाग्यपूर्ण रूप से जारी प्रमाणपत्रों को ट्रैक करने के लिए एक खुला लेखांकन प्रणाली स्थापित करना।
* उपयोगकर्ताओं को धोखाधड़ी से बचाना।

#### **प्रमाणपत्र लॉग**

प्रमाणपत्र लॉग सार्वजनिक लेखनीय, अपेंड-केवल प्रमाणपत्रों के रिकॉर्ड हैं, जो नेटवर्क सेवाओं द्वारा बनाए गए हैं। ये लॉग लेखांकन के उद्देश्यों के लिए औद्योगिक सबूत प्रदान करते हैं। प्रमाणपत्र प्रक्रियाधिकारी और सार्वजनिक दोनों इन लॉगों पर प्रमाणपत्र जमा कर सकते हैं या सत्यापन के लिए उन्हें क्वेरी कर सकते हैं। जबकि लॉग सर्वरों की सटीक संख्या निर्धारित नहीं है, यह वैश्विक रूप से हजार से कम की उम्मीद है। ये सर्वर सीए, आईएसपी, या किसी भी इच्छुक एंटिटी द्वारा स्वतंत्र रूप से प्रबंधित किए जा सकते हैं।

#### **क्वेरी**

किसी भी डोमेन के लिए प्रमाणपत्र पारदर्शिता लॉग की जांच करने के लिए [https://crt.sh/](https://crt.sh) पर जाएं।

## **फॉर्मेट**

### **PEM फॉर्मेट**

* प्रमाणपत्रों के लिए सबसे व्यापक फॉर्मेट।
* प्रमाणपत्रों और निजी कुंजीयों के लिए अलग फ़ाइलें आवश्यक होती हैं, जो बेस64 एस्की में एन्कोड किए गए होते हैं।
* सामान्य एक्सटेंशन: .cer, .crt, .pem, .key।
* मुख्य रूप से एपाचे और समान सर्वरों द्वारा उपयोग किया जाता है।

### **DER फॉर्मेट**

* प्रमाणपत्रों का एक बाइनरी फॉर्मेट।
* PEM फ़ाइलों में पाए जाने वाले "शुरू/समाप्त प्रमाणपत्र" वक्तव्य इसमें नहीं होते हैं।
* सामान्य एक्सटेंशन: .cer, .der।
* अक्सर जावा प्लेटफ़ॉर्म के साथ उपयोग किया जाता है।

### **P7B/PKCS#7 फॉर्मेट**

* बेस64 एस्की में स्टोर किया गया है, जिसमें एक्सटेंशन .p7b या .p7c होता है।
* केवल प्रमाणपत्रों और श्रृंखला प्रमाणपत्रों को शामिल करता है, निजी कुंजी को छोड़कर।
* माइक्रोसॉफ्ट विंडोज और जावा टॉमकैट द्वारा समर्थित।

### **PFX/P12/PKCS#12 फॉर्मेट**

* एक बाइनरी फॉर्मेट जो सर्वर प्रमाणपत्र, इंटरमीडिएट प्रमाणपत्र, और निजी कुंजीयों को एक फ़ाइल में बंधने का कार्य करता है।
* एक्सटेंशन: .pfx, .p12।
* मुख्य रूप से सर्टिफिकेट आयात और निर्यात के लिए विंडोज पर उपयोग किया जाता है।

### **फॉर्मेट कनवर्ट करना**

**PEM कनवर्शन** संगतता के लिए आवश्यक हैं:

* **x509 से PEM**
```bash
openssl x509 -in certificatename.cer -outform PEM -out certificatename.pem
```
* **PEM से DER**
```bash
openssl x509 -outform der -in certificatename.pem -out certificatename.der
```
* **DER से PEM**
```bash
openssl x509 -inform der -in certificatename.der -out certificatename.pem
```
* **PEM से P7B में**
```bash
openssl crl2pkcs7 -nocrl -certfile certificatename.pem -out certificatename.p7b -certfile CACert.cer
```
* **PKCS7 से PEM**
```bash
openssl pkcs7 -print_certs -in certificatename.p7b -out certificatename.pem
```
**PFX कन्वर्शन** Windows पर प्रमाणपत्रों का प्रबंधन करने के लिए महत्वपूर्ण है:

* **PFX से PEM**
```bash
openssl pkcs12 -in certificatename.pfx -out certificatename.pem
```
* **PFX से PKCS#8** में दो कदम शामिल हैं:
1. PFX को PEM में रूपांतरित करें
```bash
openssl pkcs12 -in certificatename.pfx -nocerts -nodes -out certificatename.pem
```
2. PEM को PKCS8 में रूपांतरित करें
```bash
openSSL pkcs8 -in certificatename.pem -topk8 -nocrypt -out certificatename.pk8
```
* **P7B से PFX** के लिए दो कमांड भी आवश्यक हैं:
1. P7B को CER में रूपांतरित करें
```bash
openssl pkcs7 -print_certs -in certificatename.p7b -out certificatename.cer
```
2. CER और निजी कुंजी को PFX में रूपांतरित करें
```bash
openssl pkcs12 -export -in certificatename.cer -inkey privateKey.key -out certificatename.pfx -certfile cacert.cer
```
***

<figure><img src="../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm_source=hacktricks&utm_medium=text&utm_campaign=ppc&utm_term=trickest&utm_content=certificates) का उपयोग करें और आसानी से **ऑटोमेट वर्कफ़्लो** बनाएं जो दुनिया के **सबसे उन्नत** समुदाय उपकरणों द्वारा संचालित हैं।\
आज ही पहुंचें:

{% embed url="https://trickest.com/?utm_source=hacktricks&utm_medium=banner&utm_campaign=ppc&utm_content=certificates" %}

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फ़ॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके।

</details>
