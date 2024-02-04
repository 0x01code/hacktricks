# प्रमाणपत्र

<details>

<summary><strong>शून्य से हीरो तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप चाहते हैं कि आपकी **कंपनी HackTricks में विज्ञापित हो** या **HackTricks को PDF में डाउनलोड करें** तो [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारा संग्रह [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)** का पालन करें।**
* **अपने हैकिंग ट्रिक्स साझा करें, HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos को PR जमा करके।

</details>

<figure><img src="../.gitbook/assets/image (3) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उपयोग करें और दुनिया के **सबसे उन्नत समुदाय उपकरणों** द्वारा संचालित **कार्यप्रवाहों** को आसानी से निर्मित और स्वचालित करें।\
आज ही पहुंचें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## प्रमाणपत्र क्या है

रहस्यमयता में, एक **सार्वजनिक कुंजी प्रमाणपत्र,** जिसे **डिजिटल प्रमाणपत्र** या **पहचान प्रमाणपत्र** भी कहा जाता है, एक इलेक्ट्रॉनिक दस्तावेज है जिसका उपयोग किसी सार्वजनिक कुंजी के स्वामित्व को साबित करने के लिए किया जाता है। प्रमाणपत्र में कुंजी के बारे में जानकारी, उसके मालिक की पहचान के बारे में जानकारी (जिसे विषय कहा जाता है), और एक ऐसे एक संस्था का डिजिटल हस्ताक्षर शामिल है जिसने प्रमाणपत्र की सामग्री की पुष्टि की है (जिसे जारीकर्ता कहा जाता है)। यदि हस्ताक्षर मान्य है, और प्रमाणपत्र की सामग्री को जांचने वाला सॉफ़्टवेयर जारीकर्ता पर विश्वास करता है, तो वह कुंजी का उपयोग प्रमाणपत्र के विषय के साथ सुरक्षित रूप से संचार करने के लिए कर सकता है।

एक सामान्य [सार्वजनिक-कुंजी प्रणाली](https://en.wikipedia.org/wiki/Public-key\_infrastructure) (PKI) योजना में, प्रमाणपत्र जारीकर्ता एक [प्रमाणपत्र प्राधिकरण](https://en.wikipedia.org/wiki/Certificate\_authority) (CA) होता है, जो आम तौर पर ग्राहकों से प्रमाणपत्र जारी करने के लिए शुल्क लेती है। उसके विपरीत, एक [विश्व विश्वास](https://en.wikipedia.org/wiki/Web\_of\_trust) योजना में, व्यक्ति सीधे एक-दूसरे की कुंजियों पर हस्ताक्षर करते हैं, जो एक सार्वजनिक कुंजी प्रमाणपत्र के लिए एक समान कार्य करता है।

सार्वजनिक कुंजी प्रमाणपत्रों के लिए सबसे सामान्य प्रारूप को [X.509](https://en.wikipedia.org/wiki/X.509) द्वारा परिभाषित किया गया है। क्योंकि X.509 बहुत सामान्य है, इस प्रारूप को कुछ उपयोग मामलों के लिए परिभाषित प्रोफ़ाइल द्वारा और अधिक संकुचित किया गया है, जैसे कि [सार्वजनिक कुंजी प्रणाली (X.509)](https://en.wikipedia.org/wiki/PKIX) जैसा कि RFC 5280 में परिभाषित है।

## x509 सामान्य क्षेत्र

* **संस्करण संख्या:** x509 प्रारूप का संस्करण।
* **क्रमांक:** CA के सिस्टम में प्रमाणपत्र को अद्वितीय रूप से पहचानने के लिए उपयोग किया जाता है। विशेष रूप से यह प्रमाणपत्र की रद्दीकरण जानकारी को ट्रैक करने के लिए उपयोग किया जाता है।
* **विषय:** एक ऐसा एंटिटी जिसका प्रमाणपत्र है: एक मशीन, एक व्यक्ति, या एक संगठन।
* **सामान्य नाम:** प्रमाणपत्र से प्रभावित डोमेन। 1 या अधिक हो सकते हैं और वाइल्डकार्ड शामिल कर सकते हैं।
* **देश (C):** देश
* **विभाजित नाम (DN):** पूरा विषय: `C=US, ST=California, L=San Francisco, O=Example, Inc., CN=shared.global.example.net`
* **स्थान (L):** स्थान
* **संगठन (O):** संगठन का नाम
* **संगठनिक इकाई (OU):** संगठन का विभाजन (जैसे "मानव संसाधन").
* **राज्य या प्रांत (ST, S या P):** राज्य या प्रांत के नामों की सूची
* **जारीकर्ता:** जानकारी की पुष्टि करने वाला एंटिटी और प्रमाणपत्र पर हस्ताक्षर किया है।
* **सामान्य नाम (CN):** प्रमाणपत्र प्राधिकरण का नाम
* **देश (C):** प्रमाणपत्र प्राधिकरण का देश
* **विभाजित नाम (DN):** प्रमाणपत्र प्राधिकरण का विभाजित नाम
* **स्थान (L):** संगठन को पाया जा सकने वाला स्थान।
* **संगठन (O):** संगठन का नाम
* **संगठनिक इकाई (OU):** संगठन का विभाजन (जैसे "मानव संसाधन").
* **शुरू से पहले:** प्रमाणपत्र को मान्य रहने की सबसे पहली समय और तारीख। सामान्यत: प्रमाणपत्र जारी किए जाने से कुछ घंटे या दिन पहले सेट किया जाता है, समय की असंगति समस्याओं से बचने के लिए।
* **बाद में नहीं:** प्रमाणपत्र को अब और मान्य नहीं होने की समय और तारीख।
* **सार्वजनिक कुंजी:** प्रमाणपत्र विषय की सार्वजनिक कुंजी। (यह CA द्वारा हस्ताक्षित किया जाने वाला मुख्य हिस्सा है)
* **सार्वजनिक कुंजी एल्गोरिदम:** सार्वजनिक कुंजी उत्पन्न करने के लिए उपयोग किया गया एल्गोरिदम। जैसे RSA।
* **सार्वजनिक कुंजी कर्व:** एलिप्टिक कर्व सार्वजनिक कुंजी एल्गोरिदम द्वारा उपयोग किया गया कर्व (यदि लागू हो)। जैसे nistp521।
* **सार्वजनिक कुंजी घन:** बिट में सार्वजनिक कुंजी स्थान का आकार। जैसे 2048।
* **हस्ताक्षर एल्गोरिदम:** सार्वजनिक कुंजी प्रमाणपत्र को हस्ताक्षित करने के लिए उपयोग किया गया एल्गोरिदम।
* **हस्ताक्षर:** प्रमाणपत्र शरीर का एक हस्ताक्षर जारीकर्ता की निजी कुंजी द्वारा।
* **x509v3 एक्सटेंशन्स**
* **कुंजी उपयोग:** प्रमाणपत्र की सार्वजनिक कुंजी के मान्य उपयोग। सामान्य मान शामिल हैं डिजिटल हस्ताक्षर मान्यता, कुंजी एन्सिफरमेंट, और प्रमाणपत्र हस्ताक्षर।
* एक वेब प्रमाणपत्र में यह एक _X509v3 एक्सटेंशन_ के रूप में प्रकट होग
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
**PEM को P7B में रूपांतरित करें**

**ध्यान दें:** PKCS#7 या P7B प्रारूप Base64 ASCII प्रारूप में संग्रहीत होता है और इसका फ़ाइल एक्सटेंशन .p7b या .p7c होता है। P7B फ़ाइल में केवल प्रमाणपत्र और श्रृंखला प्रमाणपत्र (इंटरमीडिएट सीए) होते हैं, न कि निजी कुंजी। P7B फ़ाइलों को समर्थित करने वाले सबसे सामान्य प्लेटफ़ॉर्म माइक्रोसॉफ्ट विंडोज और जावा टॉमकैट हैं।
```
openssl crl2pkcs7 -nocrl -certfile certificatename.pem -out certificatename.p7b -certfile CACert.cer
```
**PKCS7 को PEM में बदलें**
```
openssl pkcs7 -print_certs -in certificatename.p7b -out certificatename.pem
```
**PFX से PEM में रूपांतरित करें**

**ध्यान दें:** PKCS#12 या PFX प्रारूप एक बाइनरी प्रारूप है जिसमें सर्वर प्रमाणपत्र, इंटरमीडिएट प्रमाणपत्र और निजी कुंजी को एक एन्क्रिप्टेबल फ़ाइल में संग्रहीत करने के लिए है। PFX फ़ाइलों के एक्सटेंशन जैसे .pfx और .p12 होते हैं। PFX फ़ाइलें आम तौर पर Windows मशीनों पर प्रमाणपत्र और निजी कुंजी को आयात और निर्यात करने के लिए प्रयोग की जाती हैं।
```
openssl pkcs12 -in certificatename.pfx -out certificatename.pem
```
**PFX को PKCS#8 में बदलें**\
**ध्यान दें:** इसके लिए 2 कमांड की आवश्यकता है

**1- PFX को PEM में बदलें**
```
openssl pkcs12 -in certificatename.pfx -nocerts -nodes -out certificatename.pem
```
**2- PEM को PKCS8 में बदलें**
```
openSSL pkcs8 -in certificatename.pem -topk8 -nocrypt -out certificatename.pk8
```
**P7B को PFX में बदलें**\
**ध्यान दें:** इसके लिए 2 कमांड की आवश्यकता है

1- **P7B को CER में बदलें**
```
openssl pkcs7 -print_certs -in certificatename.p7b -out certificatename.cer
```
**2- CER और निजी कुंजी को PFX में बदलें**
```
openssl pkcs12 -export -in certificatename.cer -inkey privateKey.key -out certificatename.pfx -certfile  cacert.cer
```
<figure><img src="../.gitbook/assets/image (3) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उपयोग करें और आसानी से **वर्ल्ड के सबसे उन्नत समुदाय उपकरणों** द्वारा संचालित **ऑटोमेट वर्कफ़्लो** बनाएं।\
आज ही पहुंचें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><strong>जीरो से हीरो तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) और हमें **ट्विटर** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके।

</details>
