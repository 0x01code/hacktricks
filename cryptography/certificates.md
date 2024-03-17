# प्रमाणपत्र

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

<figure><img src="../.gitbook/assets/image (3) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उपयोग करें और दुनिया के **सबसे उन्नत समुदाय उपकरणों** द्वारा संचालित **वर्कफ़्लो** आसानी से बनाएं और स्वचालित करें।\
आज ही पहुंच प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## प्रमाणपत्र क्या है

**पब्लिक की प्रमाणपत्र** एक डिजिटल आईडी है जिसका उपयोग गणित में किसी को पब्लिक की प्रमाणपत्र होने का सबूत देने के लिए किया जाता है। इसमें कुंजी के विवरण, मालिक की पहचान (विषय), और एक विश्वसनीय प्राधिकारी से डिजिटल हस्ताक्षर शामिल है। यदि सॉफ़्टवेयर प्राधिकारी पर विश्वास करता है और हस्ताक्षर मान्य है, तो कुंजी के मालिक के साथ सुरक्षित संचार संभव है।

प्रमाणपत्रों को अधिकांशत: [प्रमाणपत्र प्राधिकारियों](https://en.wikipedia.org/wiki/Certificate\_authority) (CAs) द्वारा [सार्वजनिक-कुंजी प्रणाली](https://en.wikipedia.org/wiki/Public-key\_infrastructure) (PKI) सेटअप में जारी किया जाता है। एक और विधि है [विश्व विश्वास](https://en.wikipedia.org/wiki/Web\_of\_trust), जहां उपयोगकर्ता सीधे एक-दूसरे की कुंजियों की पुष्टि करते हैं। प्रमाणपत्रों के लिए सामान्य प्रारूप है [X.509](https://en.wikipedia.org/wiki/X.509), जो RFC 5280 में विस्तारित आवश्यकताओं के रूप में सारांशित किया जा सकता है।

## x509 सामान्य क्षेत्र

### **x509 प्रमाणपत्रों में सामान्य क्षेत्र**

x509 प्रमाणपत्रों में कई **क्षेत्र** प्रमाणपत्र की मान्यता और सुरक्षा सुनिश्चित करने में महत्वपूर्ण भूमिका निभाते हैं। यहाँ इन क्षेत्रों का विवरण है:

* **संस्करण संख्या** x509 प्रारूप के संस्करण को दर्शाती है।
* **क्रम संख्या** प्रमाणपत्र को प्रमाणपत्र प्राधिकारी (CA) सिस्टम के भीतर विशेष रूप से पुनर्निर्धारण ट्रैकिंग के लिए अद्वितीय रूप से पहचानती है।
* **विषय** क्षेत्र प्रमाणपत्र के मालिक को प्रतिनिधित्व करता है, जो कि एक मशीन, एक व्यक्ति, या एक संगठन हो सकता है। इसमें विस्तृत पहचान शामिल है जैसे:
* **सामान्य नाम (CN)**: प्रमाणपत्र द्वारा कवर किए गए डोमेन।
* **देश (C)**, **स्थान (L)**, **राज्य या प्रांत (ST, S, या P)**, **संगठन (O)**, और **संगठनिक इकाई (OU)** भौगोलिक और संगठनात्मक विवरण प्रदान करते हैं।
* **विभाजित नाम (DN)** पूर्ण विषय पहचान को आवरित करता है।
* **जारीकर्ता** वह व्यक्ति या संगठन है जिसने प्रमाणपत्र की पुष्टि की और साइन किया है, CA के लिए विषय के लिए समान उपक्षेत्रों को समेत।
* **वैधता अवधि** **Not Before** और **Not After** टाइमस्टैम्प्स द्वारा चिह्नित होती है, जिससे प्रमाणपत्र का उपयोग किसी निश्चित तारीख से पहले या बाद में नहीं किया जाता है।
* **सार्वजनिक कुंजी** खंड, प्रमाणपत्र की सुरक्षा के लिए महत्वपूर्ण है, जो सार्वजनिक कुंजी के एल्गोरिथ्म, आकार, और अन्य तकनीकी विवरण निर्दिष्ट करता है।
* **x509v3 एक्सटेंशन्स** प्रमाणपत्र की कार्यक्षमता को बढ़ाते हैं, **Key Usage**, **Extended Key Usage**, **Subject Alternative Name**, और प्रमाणपत्र के अन्य गुणों को निर्दिष्ट करने के लिए गुणवत्ता को समायोजित करने के लिए।

#### **कुंजी उपयोग और एक्सटेंशन्स**

* **कुंजी उपयोग** पब्लिक कुंजी के गणितीय अनुप्रयोगों की पहचान करता है, जैसे डिजिटल हस्ताक्षर या कुंजी एन्सिफरमेंट।
* **विस्तारित कुंजी उपयोग** प्रमाणपत्र के उपयोग मामलों को और अधिक संक्षेपित करता है, जैसे TLS सर्वर प्रमाणीकरण के लिए।
* **विषय वैकल्पिक नाम** और **मौलिक सीमा** प्रमाणपत्र द्वारा कवर किए गए अतिरिक्त होस्ट नामों को परिभाषित करते हैं और यह निर्धारित करते हैं कि यह एक सीए या अंत-इकाई प्रमाणपत्र है।
* **विषय कुंजी पहचानकर्ता** और **प्राधिकार कुंजी पहचानकर्ता** कुंजियों की अद्वितीयता और प्रत्यायन की खोजी सुनिश्चित करते हैं।
* **प्राधिकार सूचना पहुंच** और **CRL वितरण बिंदु** प्रमाणपत्र की प्रमाणपत्र प्राधिकारी की पुष्टि करने के लिए मार्ग प्रदान करते हैं और प्रमाणपत्र पुनर्निर्धारण स्थिति की जांच करने के लिए।
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

**OCSP** (**RFC 2560**) एक क्लाइंट और एक प्रतिक्रियावादी को साथ में काम करने के लिए शामिल करता है ताकि एक डिजिटल सार्वजनिक-कुंजी प्रमाणपत्र को रद्द किया गया है या नहीं जांच सकें, पूरी **CRL** को डाउनलोड करने की आवश्यकता के बिना। यह विधि पारंपरिक **CRL** से अधिक दक्ष है, जो रद्द प्रमाणपत्र सीरियल नंबरों की सूची प्रदान करता है लेकिन एक संभावित बड़ी फ़ाइल को डाउनलोड करने की आवश्यकता होती है। CRLs में तकरीबन 512 एंट्री शामिल की जा सकती हैं। अधिक विवरण यहाँ उपलब्ध हैं [here](https://www.arubanetworks.com/techdocs/ArubaOS%206\_3\_1\_Web\_Help/Content/ArubaFrameStyles/CertRevocation/About\_OCSP\_and\_CRL.htm).

### **सर्टिफिकेट पारदर्शिता क्या है**

सर्टिफिकेट पारदर्शिता सर्टिफिकेट संबंधित खतरों का मुकाबला करने में मदद करती है जिसके द्वारा सुनिश्चित किया जाता है कि SSL सर्टिफिकेट की जारीगी और मौजूदगी डोमेन मालिकों, सीए, और उपयोगकर्ताओं को दिखाई देती है। इसके उद्देश्य हैं:

* सीए को डोमेन के लिए SSL सर्टिफिकेट जारी करने से रोकना जिसके लिए डोमेन के मालिक की जानकारी के बिना हो।
* गलती से या दुरुपयोग से जारी किए गए सर्टिफिकेटों को ट्रैक करने के लिए एक खुला लेखांकन प्रणाली स्थापित करना।
* उपयोगकर्ताओं को धोखाधड़ी सर्टिफिकेटों के खिलाफ सुरक्षित रखना।

#### **सर्टिफिकेट लॉग्स**

सर्टिफिकेट लॉग्स सार्वजनिक लेखनीय, अपेंड-केवल प्रमाणपत्रों के रिकॉर्ड हैं, जो नेटवर्क सेवाओं द्वारा बनाए गए हैं। ये लॉग्स लेखांकन के उद्देश्यों के लिए औदारिक सबूत प्रदान करते हैं। जारी करने वाले प्राधिकरण और सार्वजनिक इन सर्टिफिकेटों को इन लॉग्स में सबमिट कर सकते हैं या सत्यापन के लिए उन्हें क्वेरी कर सकते हैं। जबकि लॉग सर्वरों की सटीक संख्या निश्चित नहीं है, यह वैश्विक रूप से हजार से कम की उम्मीद है। ये सर्वर सीए, आईएसपी, या किसी भी इच्छुक एंटिटी द्वारा स्वतंत्र रूप से प्रबंधित किए जा सकते हैं।

#### **क्वेरी**

किसी भी डोमेन के लिए सर्टिफिकेट पारदर्शिता लॉग्स की जांच करने के लिए, [https://crt.sh/](https://crt.sh) पर जाएं।

## **फॉर्मेट**

### **PEM फॉर्मेट**

* सर्टिफिकेटों के लिए सबसे अधिक प्रयोग किया जाने वाला फॉर्मेट।
* सर्टिफिकेटों और निजी कुंजियों के लिए अलग फ़ाइलें आवश्यक होती हैं, जो बेस64 एस्की में एन्कोड किए गए होते हैं।
* सामान्य एक्सटेंशन: .cer, .crt, .pem, .key.
* मुख्य रूप से एपाचे और समान सर्वरों द्वारा प्रयोग किया जाता है।

### **DER फॉर्मेट**

* सर्टिफिकेटों का एक बाइनरी फॉर्मेट।
* PEM फ़ाइलों में पाए जाने वाले "बिगिन/एंड सर्टिफिकेट" बयानों की कमी है।
* सामान्य एक्सटेंशन: .cer, .der.
* जावा प्लेटफ़ॉर्म के साथ अक्सर प्रयोग किया जाता है।

### **P7B/PKCS#7 फॉर्मेट**

* बेस64 एस्की में स्टोर किया गया है, जिसमें .p7b या .p7c एक्सटेंशन होती है।
* केवल सर्टिफिकेट और श्रृंखला सर्टिफिकेट को समेत करता है, निजी कुंजी को छोड़कर।
* माइक्रोसॉफ्ट विंडोज और जावा टॉमकैट द्वारा समर्थित।

### **PFX/P12/PKCS#12 फॉर्मेट**

* एक बाइनरी फॉर्मेट जो सर्वर सर्टिफिकेट, इंटरमीडिएट सर्टिफिकेट, और निजी कुंजियों को एक फ़ाइल में बंधता है।
* एक्सटेंशन: .pfx, .p12.
* मुख्य रूप से सर्टिफिकेट आयात और निर्यात के लिए विंडोज पर प्रयोग किया जाता है।

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
**PFX कन्वर्शन** Windows पर certificates को manage करने के लिए महत्वपूर्ण हैं:

* **PFX से PEM**
```bash
openssl pkcs12 -in certificatename.pfx -out certificatename.pem
```
* **PFX से PKCS#8** में दो कदम शामिल हैं:
1. PFX को PEM में रूपांतरित करें
```bash
openssl pkcs12 -in certificatename.pfx -nocerts -nodes -out certificatename.pem
```
2. PEM को PKCS8 में बदलें
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

<figure><img src="../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उपयोग करें और आसानी से **दुनिया के सबसे उन्नत समुदाय उपकरणों** द्वारा संचालित **कार्यप्रवाहों** को निर्मित करें।\
आज ही पहुंचें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><strong>शून्य से हीरो तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सदस्यता योजनाएं देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके।

</details>
