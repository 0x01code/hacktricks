# प्रमाणपत्र

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) **का अनुसरण करें**.
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग तकनीकें साझा करें.

</details>

<figure><img src="../.gitbook/assets/image (3) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks) का उपयोग करके आसानी से **वर्कफ्लोज़ को बिल्ड और ऑटोमेट** करें जो दुनिया के **सबसे उन्नत** समुदाय टूल्स द्वारा संचालित होते हैं.\
आज ही एक्सेस प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## प्रमाणपत्र क्या है

क्रिप्टोग्राफी में, **पब्लिक की प्रमाणपत्र,** जिसे **डिजिटल प्रमाणपत्र** या **पहचान प्रमाणपत्र** भी कहा जाता है, एक इलेक्ट्रॉनिक दस्तावेज होता है जिसका उपयोग पब्लिक की के स्वामित्व को साबित करने के लिए किया जाता है। प्रमाणपत्र में की के बारे में जानकारी, उसके मालिक (जिसे विषय कहा जाता है) की पहचान के बारे में जानकारी, और एक इकाई के डिजिटल हस्ताक्षर शामिल होते हैं जिसने प्रमाणपत्र की सामग्री की पुष्टि की है (जिसे जारीकर्ता कहा जाता है)। यदि हस्ताक्षर मान्य है, और प्रमाणपत्र की जांच करने वाले सॉफ्टवेयर को जारीकर्ता पर भरोसा है, तो वह उस की का उपयोग प्रमाणपत्र के विषय के साथ सुरक्षित रूप से संवाद करने के लिए कर सकता है।

एक विशिष्ट [पब्लिक-की इंफ्रास्ट्रक्चर](https://en.wikipedia.org/wiki/Public-key_infrastructure) (PKI) योजना में, प्रमाणपत्र जारीकर्ता एक [प्रमाणपत्र प्राधिकरण](https://en.wikipedia.org/wiki/Certificate_authority) (CA) होता है, आमतौर पर एक कंपनी जो ग्राहकों को उनके लिए प्रमाणपत्र जारी करने के लिए शुल्क लेती है। इसके विपरीत, एक [विश्वास के जाल](https://en.wikipedia.org/wiki/Web_of_trust) योजना में, व्यक्ति सीधे एक दूसरे की कीज़ पर हस्ताक्षर करते हैं, एक प्रारूप में जो पब्लिक की प्रमाणपत्र के समान कार्य करता है।

पब्लिक की प्रमाणपत्रों के लिए सबसे आम प्रारूप [X.509](https://en.wikipedia.org/wiki/X.509) द्वारा परिभाषित है। क्योंकि X.509 बहुत सामान्य है, प्रारूप को विशेष उपयोग के मामलों के लिए परिभाषित प्रोफाइलों द्वारा और अधिक सीमित किया जाता है, जैसे कि [पब्लिक की इंफ्रास्ट्रक्चर (X.509)](https://en.wikipedia.org/wiki/PKIX) जो RFC 5280 में परिभाषित है।

## x509 सामान्य फील्ड्स

* **संस्करण संख्या:** x509 प्रारूप का संस्करण।
* **क्रम संख्या**: CA की प्रणालियों के भीतर प्रमाणपत्र की अद्वितीय पहचान के लिए उपयोग की जाती है। विशेष रूप से यह प्रतिस्थापन जानकारी को ट्रैक करने के लिए उपयोग की जाती है।
* **विषय**: प्रमाणपत्र किसके लिए है: एक मशीन, एक व्यक्ति, या एक संगठन।
* **सामान्य नाम**: प्रमाणपत्र द्वारा प्रभावित डोमेन। 1 या अधिक हो सकते हैं और इसमें वाइल्डकार्ड्स शामिल हो सकते हैं।
* **देश (C)**: देश
* **प्रतिष्ठित नाम (DN)**: पूरा विषय: `C=US, ST=California, L=San Francisco, O=Example, Inc., CN=shared.global.example.net`
* **स्थानीयता (L)**: स्थानीय स्थान
* **संगठन (O)**: संगठन का नाम
* **संगठनात्मक इकाई (OU)**: एक संगठन का विभाजन (जैसे "मानव संसाधन").
* **राज्य या प्रांत (ST, S या P)**: राज्य या प्रांत के नामों की सूची
* **जारीकर्ता**: जिस इकाई ने जानकारी की पुष्टि की और प्रमाणपत्र पर हस्ताक्षर किए।
* **सामान्य नाम (CN)**: प्रमाणपत्र प्राधिकरण का नाम
* **देश (C)**: प्रमाणपत्र प्राधिकरण का देश
* **प्रतिष्ठित नाम (DN)**: प्रमाणपत्र प्राधिकरण का प्रतिष्ठित नाम
* **स्थानीयता (L)**: स्थानीय स्थान जहां संगठन पाया जा सकता है।
* **संगठन (O)**: संगठन का नाम
* **संगठनात्मक इकाई (OU)**: एक संगठन का विभाजन (जैसे "मानव संसाधन").
* **नॉट बिफोर**: प्रमाणपत्र के मान्य होने का सबसे पहला समय और तारीख। आमतौर पर यह प्रमाणपत्र जारी किए जाने के क्षण से कुछ घंटे या दिन पहले सेट किया जाता है, ताकि [घड़ी की विसंगति](https://en.wikipedia.org/wiki/Clock_skew#On_a_network) समस्याओं से बचा जा सके।
* **नॉट आफ्टर**: समय और तारीख जिसके बाद प्रमाणपत्र अब मान्य नहीं है।
* **पब्लिक की**: प्रमाणपत्र विषय की एक पब्लिक की। (यह मुख्य भागों में से एक है क्योंकि यह CA द्वारा हस्ताक्षरित होता है)
* **पब्लिक की एल्गोरिथ्म**: पब्लिक की उत्पन्न करने के लिए उपयोग की गई एल्गोरिथ्म। जैसे RSA।
* **पब्लिक की कर्व**: एलिप्टिक कर्व पब्लिक की एल्गोरिथ्म द्वारा उपयोग किया गया कर्व (यदि लागू हो)। जैसे nistp521।
* **पब्लिक की एक्सपोनेंट**: पब्लिक की डेराइव करने के लिए उपयोग किया गया एक्सपोनेंट (यदि लागू हो)। जैसे 65537।
* **पब्लिक की साइज**: बिट्स में पब्लिक की स्पेस का आकार। जैसे 2048।
* **सिग्नेचर एल्गोरिथ्म**: पब्लिक की प्रमाणपत्र को हस्ताक्षर करने के लिए उपयोग की गई एल्गोरिथ्म।
* **सिग्नेचर**: जारीकर्ता की प्राइवेट की द्वारा प्रमाणपत्र शरीर का एक हस्ताक्षर।
* **x509v3 एक्सटेंशन्स**
* **की उपयोग**: प्रमाणपत्र की पब्लिक की के क्रिप्टोग्राफिक उपयोगों की वैधता। सामान्य मानों में डिजिटल हस्ताक्षर मान्यता, की एन्क्रिप्टमेंट, और प्रमाणपत्र हस्त
```
openssl x509 -in certificatename.cer -outform PEM -out certificatename.pem
```
#### **PEM को DER में बदलें**
```
openssl x509 -outform der -in certificatename.pem -out certificatename.der
```
**DER को PEM में बदलें**
```
openssl x509 -inform der -in certificatename.der -out certificatename.pem
```
**PEM को P7B में बदलें**

**ध्यान दें:** PKCS#7 या P7B प्रारूप Base64 ASCII प्रारूप में संग्रहीत होता है और इसका फ़ाइल एक्सटेंशन .p7b या .p7c होता है। P7B फ़ाइल में केवल प्रमाणपत्र और चेन प्रमाणपत्र (Intermediate CAs) होते हैं, निजी कुंजी नहीं होती। P7B फ़ाइलों को समर्थन करने वाले सबसे सामान्य प्लेटफ़ॉर्म Microsoft Windows और Java Tomcat हैं।
```
openssl crl2pkcs7 -nocrl -certfile certificatename.pem -out certificatename.p7b -certfile CACert.cer
```
**PKCS7 को PEM में परिवर्तित करें**
```
openssl pkcs7 -print_certs -in certificatename.p7b -out certificatename.pem
```
**pfx को PEM में बदलें**

**ध्यान दें:** PKCS#12 या PFX प्रारूप एक बाइनरी प्रारूप है जो सर्वर प्रमाणपत्र, मध्यवर्ती प्रमाणपत्र, और निजी कुंजी को एक एन्क्रिप्ट करने योग्य फाइल में संग्रहित करने के लिए है। PFX फाइलों के आमतौर पर एक्सटेंशन जैसे .pfx और .p12 होते हैं। PFX फाइलें आमतौर पर Windows मशीनों पर प्रमाणपत्र और निजी कुंजियों को आयात और निर्यात करने के लिए उपयोग की जाती हैं।
```
openssl pkcs12 -in certificatename.pfx -out certificatename.pem
```
**PFX को PKCS#8 में बदलें**\
**ध्यान दें:** इसके लिए 2 आदेशों की आवश्यकता है

**1- PFX को PEM में बदलें**
```
openssl pkcs12 -in certificatename.pfx -nocerts -nodes -out certificatename.pem
```
**2- PEM को PKCS8 में परिवर्तित करें**
```
openSSL pkcs8 -in certificatename.pem -topk8 -nocrypt -out certificatename.pk8
```
**P7B को PFX में बदलें**\
**ध्यान दें:** इसके लिए 2 आदेशों की आवश्यकता है

1- **P7B को CER में बदलें**
```
openssl pkcs7 -print_certs -in certificatename.p7b -out certificatename.cer
```
**2- CER और Private Key को PFX में परिवर्तित करें**
```
openssl pkcs12 -export -in certificatename.cer -inkey privateKey.key -out certificatename.pfx -certfile  cacert.cer
```
<figure><img src="../.gitbook/assets/image (3) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उपयोग करके आसानी से **वर्कफ्लोज़ को ऑटोमेट** करें जो दुनिया के **सबसे उन्नत** समुदाय टूल्स द्वारा संचालित होते हैं।\
आज ही एक्सेस प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert) के साथ AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपोज़ में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>
