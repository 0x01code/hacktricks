# प्रमाणपत्र

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल** हों या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का पालन करें।**
* **हैकिंग ट्रिक्स साझा करें और** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **में PR जमा करके** अपने हैकिंग ट्रिक्स साझा करें।

</details>

<figure><img src="../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उपयोग करें और आसानी से बनाएं और **स्वचालित कार्यप्रवाह** को चलाएं जो दुनिया के **सबसे उन्नत** सामुदायिक उपकरणों द्वारा संचालित होता है।\
आज ही पहुंच प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## प्रमाणपत्र क्या है

क्रिप्टोग्राफी में, एक **सार्वजनिक कुंजी प्रमाणपत्र**, जिसे **डिजिटल प्रमाणपत्र** या **पहचान प्रमाणपत्र** भी कहा जाता है, एक इलेक्ट्रॉनिक दस्तावेज़ है जिसका उपयोग सार्वजनिक कुंजी के स्वामित्व को साबित करने के लिए किया जाता है। प्रमाणपत्र में कुंजी के बारे में जानकारी, उसके मालिक की पहचान के बारे में जानकारी (जिसे विषय कहा जाता है), और प्रमाणपत्र की सामग्री को सत्यापित करने वाले एक संगठन के द्वारा दिए गए डिजिटल हस्ताक्षर शामिल होते हैं (जिसे जारक कहा जाता है)। यदि हस्ताक्षर मान्य है और प्रमाणपत्र की सामग्री को सत्यापित करने वाला सॉफ़्टवेयर प्रमाणित करता है, तो वह प्रमाणपत्र के विषय के साथ सुरक्षित रूप से संचार करने के लिए उस कुंजी का उपयोग कर सकता है।

एक साधारण [सार्वजनिक-कुंजी बुनियादी](https://en.wikipedia.org/wiki/Public-key\_infrastructure) (PKI) योजना में, प्रमाणपत्र जारक एक [प्रमाणपत्र प्राधिकरण](https://en.wikipedia.org/wiki/Certificate\_authority) (CA) होता है, आमतौर पर एक कंपनी जो ग्राहकों को प्रमाणपत्र जारी करने के लिए शुल्क लेती है। इसके विपरीत, एक [विश्व विश्वास](https://en.wikipedia.org/wiki/Web\_of\_trust) योजना में, व्यक्तियों एक-दूसरे की कुंजी के साथ साइन करते हैं, एक प्रमाणपत्र के लिए एक समान कार्य करने वाले प्रारूप में।

सार्वजनिक कुंजी प्रमाणपत्रों के लिए सबसे सामान्य प्रारूप [X.509](https://en.wikipedia.org/wiki/X.509) द्वारा परिभाषित किया जाता है। क्योंकि X.509 बहुत सामान्य है, इस प्रारूप को निश्चित उपयोग मामलों के लिए परिभाषित प्रोफ़ाइल द्वारा और बाहरी बाध्यताओं द्वारा ब
* **विषय कुंजी पहचानकर्ता** (SKI): यह एक्सटेंशन प्रमाणपत्र में सार्वजनिक कुंजी के लिए एक अद्वितीय पहचानकर्ता घोषित करता है। इसे सभी सीए प्रमाणपत्रों पर आवश्यक होता है। सीए अपनी स्वयं की एसकेआई को जारी प्रमाणपत्रों पर जारीकर्ता कुंजी पहचानकर्ता (एएके) एक्सटेंशन में प्रसारित करते हैं। यह विषय सार्वजनिक कुंजी का हैश है।
* **अधिकार कुंजी पहचानकर्ता**: इसमें एक कुंजी पहचानकर्ता होता है जो प्रमाणपत्र में जारीकर्ता की सार्वजनिक कुंजी से प्राप्त होता है। यह जारीकर्ता सार्वजनिक कुंजी का हैश होता है।
* **अधिकार सूचना पहुंच** (AIA): यह एक्सटेंशन निम्नलिखित दो प्रकार की जानकारी को संग्रहित करता है:
* इस प्रमाणपत्र के जारीकर्ता को कैसे प्राप्त करें (सीए जारक पहुंच विधि)
* इस प्रमाणपत्र की रद्दी की जांच की जा सकती है वहां से जहां से इस प्रमाणपत्र की रद्दी की जा सकती है (ओसीएसपी पहुंच विधि)।
* **सीआरएल वितरण बिंदु**: यह एक्सटेंशन इस प्रमाणपत्र की रद्दी की जांच की जा सकने वाली सीआरएल के स्थान की पहचान करता है। प्रमाणपत्र को प्रसंस्करण करने वाला एप्लिकेशन इस एक्सटेंशन से सीआरएल के स्थान को प्राप्त कर सकता है, सीआरएल को डाउनलोड करके और फिर इस प्रमाणपत्र की रद्दी की जांच कर सकता है।
* **सीटी प्रीसर्टिफिकेट एससीटी** : प्रमाणपत्र के संबंध में प्रमाणपत्र पारदर्शिता के लॉग

### OCSP और CRL वितरण बिंदु के बीच अंतर

**OCSP** (RFC 2560) एक मानक प्रोटोकॉल है जिसमें एक **OCSP क्लाइंट और एक OCSP उत्तरदाता** होता है। यह प्रोटोकॉल एक दिए गए डिजिटल सार्वजनिक-कुंजी प्रमाणपत्र की रद्दी स्थिति **पूरे CRL को डाउनलोड किए बिना** निर्धारित करता है।\
**CRL** प्रमाणपत्र की मान्यता की जांच करने का **पारंपरिक तरीका** है। **CRL एक प्रमाणपत्र अनुक्रम संख्याओं की सूची प्रदान करता है** जिनकी रद्दी की गई है या अमान्य हो गई है। CRL को प्रमाणित करते समय सत्यापक को प्रस्तुत प्रमाणपत्र की रद्दी स्थिति की जांच करने की अनुमति देते हैं। CRL की संख्या 512 प्रविष्टियों तक सीमित होती है।\
[यहां से](https://www.arubanetworks.com/techdocs/ArubaOS%206\_3\_1\_Web\_Help/Content/ArubaFrameStyles/CertRevocation/About\_OCSP\_and\_CRL.htm)।

### प्रमाणपत्र पारदर्शिता क्या है

प्रमाणपत्र पारदर्शिता उद्देश्य से प्रमाणपत्र पर आधारित खतरों को दूर करने का प्रयास करता है जिसमें डोमेन मालिकों, सीएएस और डोमेन उपयोगकर्ताओं को प्रमाणपत्र के जारी होने और मौजूद होने की जांच करने की सुविधा होती है। विशेष रूप से, प्रमाणपत्र पारदर्शिता के तीन मुख्य उद्देश्य हैं:

* एक ऐसे सीए को असंभव बनाना (या कम से कम बहुत मुश्किल) जो एक डोमेन के लिए एक SSL प्रमाणपत्र जारी कर सकता है बिना उस डोमेन के मालिक को उस प्रमाणपत्र के द्वारा दिखाई देने की सुविधा के बिना।
* किसी भी डोमेन मालिक या सीए को यह निर्धारित करने की सुविधा प्रदान करना कि क्या प्रमाणपत्र गलती से या दुर्भाग्यपूर्ण रूप से जारी किए गए हैं।
* उपयोगकर्ताओं की सुरक्षा (जितना संभव हो सके) को गलती से या दुर्भाग्यपूर्ण रूप से जारी किए गए प्रमाणपत्रों से बचाना।

#### प्रमाणपत्र लॉग

प्रमाणपत्र लॉग साधारणतया प्रमाणपत्रों के **क्रिप्टोग्राफिक आश्वासित, सार्वजनिक निरीक्षण योग्य, अपेंड-केवल रिकॉर्ड्स** को बनाए रखने वाली सर्विसेज होती हैं। **कोई भी व्यक्ति एक लॉग में प्रमाणपत्र सबमिट कर सकता है**, हालांकि प्रमाणपत्र प्राधिकरणों को संभावित रूप से सबमिट करने वाले होंगे। उसी तरह, कोई भी व्यक्ति एक लॉग के लिए एक क्रिप्टोग्राफिक प्रमाण प्राप्त कर सकता है, जिसका उपयोग करके यह सत्यापित किया जा सकता है कि लॉग सही ढंग से काम कर रहा है या एक विशेष प्रमाणपत्र को लॉग किया गया है। लॉग सर्वरों की संख्य
```
openssl x509 -in certificatename.cer -outform PEM -out certificatename.pem
```
#### **PEM को DER में कनवर्ट करें**

To convert a PEM (Privacy-Enhanced Mail) certificate file to DER (Distinguished Encoding Rules) format, you can use the OpenSSL command-line tool.

PEM एक प्राइवेसी-एन्हांस्ड मेल (Privacy-Enhanced Mail) प्रमाणपत्र फ़ाइल है और DER (Distinguished Encoding Rules) प्रारूप में कनवर्ट करने के लिए, आप OpenSSL कमांड-लाइन उपकरण का उपयोग कर सकते हैं।

```plaintext
openssl x509 -outform der -in certificate.pem -out certificate.der
```

This command will convert the certificate.pem file from PEM format to DER format and save it as certificate.der.

यह कमांड certificate.pem फ़ाइल को PEM प्रारूप से DER प्रारूप में कनवर्ट करेगा और इसे certificate.der के रूप में सहेजेगा।
```
openssl x509 -outform der -in certificatename.pem -out certificatename.der
```
**DER को PEM में कनवर्ट करें**

To convert a DER-encoded certificate to PEM format, you can use the OpenSSL command-line tool. Here's the command you can use:

```
openssl x509 -inform der -in certificate.der -out certificate.pem
```

Replace `certificate.der` with the path to your DER-encoded certificate file, and `certificate.pem` with the desired output file name for the PEM-encoded certificate.

DER-कोडेड प्रमाणपत्र को PEM प्रारूप में कनवर्ट करने के लिए, आप OpenSSL कमांड-लाइन टूल का उपयोग कर सकते हैं। यहां आपका उपयोग करने का कमांड है:

```
openssl x509 -inform der -in certificate.der -out certificate.pem
```

`certificate.der` को अपने DER-कोडेड प्रमाणपत्र फ़ाइल के पथ से बदलें, और `certificate.pem` को PEM-कोडेड प्रमाणपत्र के लिए वांछित आउटपुट फ़ाइल का नाम बदलें।
```
openssl x509 -inform der -in certificatename.der -out certificatename.pem
```
**PEM को P7B में कनवर्ट करें**

**नोट:** PKCS#7 या P7B प्रारूप Base64 ASCII प्रारूप में संग्रहीत किया जाता है और इसका फ़ाइल एक्सटेंशन .p7b या .p7c होता है। P7B फ़ाइल में केवल प्रमाणपत्र और चेन प्रमाणपत्र (इंटरमीडिएट सीए) होते हैं, न कि निजी कुंजी। P7B फ़ाइल को समर्थित करने वाले सबसे सामान्य प्लेटफ़ॉर्म माइक्रोसॉफ़्ट विंडोज और जावा टॉमकैट हैं।
```
openssl crl2pkcs7 -nocrl -certfile certificatename.pem -out certificatename.p7b -certfile CACert.cer
```
**PKCS7 को PEM में कनवर्ट करें**

PKCS7 फॉर्मेट में एक डिजिटल सर्टिफिकेट या एन्क्रिप्टेड डेटा हो सकता है। PKCS7 फॉर्मेट को PEM (Privacy Enhanced Mail) फॉर्मेट में कनवर्ट करने के लिए निम्नलिखित चरणों का पालन करें:

1. पहले, PKCS7 फ़ाइल को खोलें और उसे Base64 डेकोड करें।
2. डेकोड किए गए डेटा को PEM फॉर्मेट में लिखें।
3. PEM फ़ाइल को सहेजें और उसे उपयोग करें।

यदि आप OpenSSL का उपयोग कर रहे हैं, तो निम्नलिखित कमांड का उपयोग करके PKCS7 फ़ाइल को PEM में कनवर्ट कर सकते हैं:

```plaintext
openssl pkcs7 -print_certs -in input.p7b -out output.pem
```

यह कमांड PKCS7 फ़ाइल `input.p7b` को PEM फ़ाइल `output.pem` में कनवर्ट करेगी।
```
openssl pkcs7 -print_certs -in certificatename.p7b -out certificatename.pem
```
**PFX को PEM में कनवर्ट करें**

**नोट:** PKCS#12 या PFX प्रारूप सर्वर प्रमाणपत्र, इंटरमीडिएट प्रमाणपत्र और निजी कुंजी को एक एन्क्रिप्ट करने योग्य फ़ाइल में संग्रहीत करने के लिए एक बाइनरी प्रारूप है। PFX फ़ाइलों के आम रूप से .pfx और .p12 जैसे एक्सटेंशन होते हैं। PFX फ़ाइलें आमतौर पर विंडोज मशीनों पर प्रमाणपत्र और निजी कुंजी को आयात और निर्यात करने के लिए उपयोग की जाती हैं।
```
openssl pkcs12 -in certificatename.pfx -out certificatename.pem
```
**PFX को PKCS#8 में बदलें**\
**नोट:** इसके लिए 2 कमांड की आवश्यकता होती है

**1- PFX को PEM में बदलें**
```
openssl pkcs12 -in certificatename.pfx -nocerts -nodes -out certificatename.pem
```
**2- PEM को PKCS8 में कनवर्ट करें**

To convert a PEM file to PKCS8 format, you can use the OpenSSL command-line tool. Here's how you can do it:

```plaintext
openssl pkcs8 -topk8 -inform PEM -outform PEM -in private_key.pem -out private_key_pkcs8.pem -nocrypt
```

In the above command, replace `private_key.pem` with the path to your PEM file, and `private_key_pkcs8.pem` with the desired output file name for the PKCS8 format.

This command will convert the private key in the PEM file to PKCS8 format and save it in the specified output file. The `-nocrypt` option ensures that the private key is not encrypted with a passphrase.

After executing the command, you will have the private key in PKCS8 format, ready to be used for various cryptographic operations.
```
openSSL pkcs8 -in certificatename.pem -topk8 -nocrypt -out certificatename.pk8
```
**P7B को PFX में बदलें**\
**नोट:** इसके लिए 2 कमांड की आवश्यकता होती है

1- **P7B को CER में बदलें**
```
openssl pkcs7 -print_certs -in certificatename.p7b -out certificatename.cer
```
**2- CER और निजी कुंजी को PFX में परिवर्तित करें**

यदि आपके पास एक CER प्रमाणपत्र और निजी कुंजी है और आप इन्हें PFX प्रारूप में परिवर्तित करना चाहते हैं, तो आप निम्नलिखित चरणों का पालन कर सकते हैं:

1. सबसे पहले, एक पाठ संपादक में एक नया फ़ाइल खोलें और उसे `.txt` या किसी अन्य उपयुक्त नाम से सहेजें।

2. अपनी CER प्रमाणपत्र फ़ाइल को खोलें और सभी सामग्री को पाठ संपादक में कॉपी करें।

3. पाठ संपादक में जाएं और नया पंक्ति शुरू करें। इस पंक्ति में, निम्न लाइन को पेस्ट करें:

   ```
   -----BEGIN CERTIFICATE-----
   ```

4. अब, CER प्रमाणपत्र की सामग्री को पेस्ट करें।

5. एक और नया पंक्ति शुरू करें और निम्न लाइन को पेस्ट करें:

   ```
   -----END CERTIFICATE-----
   ```

6. अब, अपनी निजी कुंजी फ़ाइल को खोलें और सभी सामग्री को पाठ संपादक में कॉपी करें।

7. पाठ संपादक में जाएं और नया पंक्ति शुरू करें। इस पंक्ति में, निम्न लाइन को पेस्ट करें:

   ```
   -----BEGIN PRIVATE KEY-----
   ```

8. अब, निजी कुंजी की सामग्री को पेस्ट करें।

9. एक और नया पंक्ति शुरू करें और निम्न लाइन को पेस्ट करें:

   ```
   -----END PRIVATE KEY-----
   ```

10. अब, फ़ाइल को सहेजें और उसे `.pfx` या किसी अन्य उपयुक्त नाम से सहेजें।

इसके बाद, आपके पास CER प्रमाणपत्र और निजी कुंजी का PFX प्रारूप उपलब्ध होगा।
```
openssl pkcs12 -export -in certificatename.cer -inkey privateKey.key -out certificatename.pfx -certfile  cacert.cer
```
<figure><img src="../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उपयोग करके आसानी से बनाएं और **स्वचालित कार्यप्रवाह** बनाएं जो दुनिया के **सबसे उन्नत** समुदाय उपकरणों द्वारा संचालित होता है।\
आज ही पहुंच प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एकल [**NFT संग्रह**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) को PR जमा करके।

</details>
