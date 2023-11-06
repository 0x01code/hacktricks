# डिस्ट्रोलेस को हथियार बनाना

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ हैकट्रिक्स क्लाउड ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 ट्विटर 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ ट्विच 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 यूट्यूब 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **हैकट्रिक्स में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने की उपलब्धता** चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**द पीएस फैमिली**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल** हों या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **हैकिंग ट्रिक्स साझा करें और PR जमा करें** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **के माध्यम से**.

</details>

## डिस्ट्रोलेस क्या है

डिस्ट्रोलेस कंटेनर एक प्रकार का कंटेनर है जिसमें केवल उस विशेष एप्लिकेशन को चलाने के लिए आवश्यक आवश्यकताएं होती हैं, बिना किसी अतिरिक्त सॉफ़्टवेयर या टूल के जो आवश्यक नहीं होते। ये कंटेनर इतने **हल्के** और **सुरक्षित** होने का उद्देश्य रखते हैं, और वे किसी भी अनावश्यक घटकों को हटाकर **हमले के सतह को कम से कम** करने का प्रयास करते हैं।

डिस्ट्रोलेस कंटेनर अक्सर **प्रोडक्शन वातावरणों** में उपयोग किए जाते हैं जहां सुरक्षा और विश्वसनीयता महत्वपूर्ण होती हैं।

कुछ **डिस्ट्रोलेस कंटेनर** के **उदाहरण** हैं:

* **Google** द्वारा प्रदान किया गया: [https://console.cloud.google.com/gcr/images/distroless/GLOBAL](https://console.cloud.google.com/gcr/images/distroless/GLOBAL)
* **Chainguard** द्वारा प्रदान किया गया: [https://github.com/chainguard-images/images/tree/main/images](https://github.com/chainguard-images/images/tree/main/images)

## डिस्ट्रोलेस को हथियार बनाना

डिस्ट्रोलेस कंटेनर को हथियार बनाने का उद्देश्य है कि **सीमाओं** के बावजूद (सिस्टम में सामान्य बाइनरी की कमी) और कंटेनर में सामान्य रूप से पाए जाने वाली सुरक्षा सुरक्षाओं (डिवाइस `/dev/shm` में रीड-ओनली या नो-एक्जीक्यूट) के बावजूद भी **विभिन्न बाइनरी और पेलोड को क्रियान्वित** किया जा सके।

### मेमोरी के माध्यम से

2023 के किसी बिंदु पर आ रहा है...

### मौजूदा बाइनरी के माध्यम से

#### openssl

****[**इस पोस्ट में,**](https://www.form3.tech/engineering/content/exploiting-distroless-images) बताया गया है कि बाइनरी **`openssl`** इन कंटेनर में अक्सर पाया जाता है, संभवतः इसलिए कि इसे कंटेनर के अंदर चलाने वाले सॉफ़्टवेयर की **आवश्यकता** होती है।

**`openssl`** बाइनरी का दुरुपयोग करके **विभिन्न चीजें क्रियान्वित** की जा सकती हैं।
