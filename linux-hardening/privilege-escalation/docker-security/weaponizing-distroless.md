# डिस्ट्रोलेस का उपयोग करना

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से नायक तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग ट्रिक्स शेयर करें।

</details>

## डिस्ट्रोलेस क्या है

एक डिस्ट्रोलेस कंटेनर एक प्रकार का कंटेनर है जिसमें **केवल विशिष्ट एप्लिकेशन चलाने के लिए आवश्यक निर्भरताएं होती हैं**, बिना किसी अतिरिक्त सॉफ्टवेयर या टूल्स के जो आवश्यक नहीं हैं। ये कंटेनर **हल्के** और **सुरक्षित** होने के लिए डिजाइन किए गए हैं, और वे अनावश्यक घटकों को हटाकर **हमले की सतह को कम से कम करने** का लक्ष्य रखते हैं।

डिस्ट्रोलेस कंटेनर अक्सर **उत्पादन वातावरण में उपयोग किए जाते हैं जहां सुरक्षा और विश्वसनीयता महत्वपूर्ण होती है**।

डिस्ट्रोलेस कंटेनर्स के कुछ **उदाहरण** हैं:

* **Google** द्वारा प्रदान किया गया: [https://console.cloud.google.com/gcr/images/distroless/GLOBAL](https://console.cloud.google.com/gcr/images/distroless/GLOBAL)
* **Chainguard** द्वारा प्रदान किया गया: [https://github.com/chainguard-images/images/tree/main/images](https://github.com/chainguard-images/images/tree/main/images)

## डिस्ट्रोलेस का उपयोग करना

एक डिस्ट्रोलेस कंटेनर को हथियार बनाने का लक्ष्य यह है कि **डिस्ट्रोलेस** द्वारा निहित सीमाओं (सिस्टम में सामान्य बाइनरीज़ की कमी) के साथ-साथ कंटेनरों में आमतौर पर पाए जाने वाले सुरक्षा उपायों जैसे `/dev/shm` में **रीड-ओनली** या **नो-एक्जीक्यूट** के बावजूद **मनमाने बाइनरीज़ और पेलोड्स को निष्पादित करना संभव हो**।

### मेमोरी के माध्यम से

2023 के किसी समय आ रहा है...

### मौजूदा बाइनरीज़ के माध्यम से

#### openssl

****[**इस पोस्ट में,**](https://www.form3.tech/engineering/content/exploiting-distroless-images) बताया गया है कि बाइनरी **`openssl`** अक्सर इन कंटेनरों में पाई जाती है, संभवतः इसलिए क्योंकि यह कंटेनर के अंदर चलने वाले सॉफ्टवेयर द्वारा **आवश्यक** होती है।

**`openssl`** बाइनरी का दुरुपयोग करके **मनमानी चीजों को निष्पादित करना संभव है**।

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से नायक तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग ट्रिक्स शेयर करें।

</details>
