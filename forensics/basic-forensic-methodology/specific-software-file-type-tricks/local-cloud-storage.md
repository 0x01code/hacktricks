# स्थानीय क्लाउड संग्रह

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल** हों या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का पालन करें**.
* **अपने हैकिंग ट्रिक्स साझा करें** हैकट्रिक्स रेपो (hacktricks repo) और हैकट्रिक्स-क्लाउड रेपो (hacktricks-cloud repo) में पीआर जमा करके।

</details>

<figure><img src="../../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उपयोग करें और दुनिया के **सबसे उन्नत सामुदायिक उपकरणों** द्वारा संचालित **वर्कफ़्लो** को आसानी से बनाएं और स्वचालित करें।\
आज ही पहुंच प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## OneDrive

Windows में, आप OneDrive फ़ोल्डर को `\Users\<username>\AppData\Local\Microsoft\OneDrive` में ढूंढ सकते हैं। और `logs\Personal` के अंदर आपको फ़ाइल `SyncDiagnostics.log` मिल सकती है जिसमें सिंक्रनाइज़ की गई फ़ाइलों के बारे में कुछ दिलचस्प डेटा हो सकता है:

* बाइट में आकार
* निर्माण तिथि
* संशोधन तिथि
* क्लाउड में फ़ाइलों की संख्या
* फ़ोल्डर में फ़ाइलों की संख्या
* **CID**: OneDrive उपयोगकर्ता का अद्वितीय आईडी
* रिपोर्ट उत्पन्न करने का समय
* ओएस के HD का आकार

एक बार जब आप CID ढूंढ़ लेते हैं, तो सिफारिश की जाती है कि आप **इस आईडी को संबंधित फ़ाइलों की खोज करें**। आप OneDrive के साथ सिंक्रनाइज़ की गई फ़ाइलों के नामों जैसे _**\<CID>.ini**_ और _**\<CID>.dat**_ वाली फ़ाइलें ढूंढ़ सकते हैं जो दिलचस्प जानकारी जैसे फ़ाइलों के नाम शामिल कर सकती हैं।

## Google Drive

Windows में, आप मुख्य Google Drive फ़ोल्डर को `\Users\<username>\AppData\Local\Google\Drive\user_default` में ढूंढ सकते हैं।\
इस फ़ोल्डर में एक फ़ाइल होती है जिसका नाम Sync\_log.log होता है, जिसमें खाते का ईमेल पता, फ़ाइल के नाम, टाइमस्टैम्प, फ़ाइलों के MD5 हैश आदि की जानकारी होती है। हटाई गई फ़ाइलें भी उस लॉग फ़ाइल में अपने संबंधित MD5 के साथ दिखाई देती हैं।

फ़ाइल **`Cloud_graph\Cloud_graph.db`** एक sqlite डेटाबेस है जिसमें टेबल **`cloud_graph_entry`** होती है। इस टेबल में आप सिंक्रनाइज़ की गई फ़ाइलों का **नाम**, संशोधित समय, आकार और फ़ाइलों के MD5 checksum पाएंगे।

डेटाबेस **`Sync_config.db`** के टेबल डेटा में खाते का ईमेल पता, साझा किए गए फ़ोल्डरों का पथ और Google Drive संस्करण होता है।

## Dropbox

Dropbox फ़ाइलों को प्रबंधित करने के लिए
```bash
sqlite -k <Obtained Key> config.dbx ".backup config.db" #This decompress the config.dbx and creates a clear text backup in config.db
```
डेटाबेस **`config.dbx`** में निम्नलिखित जानकारी होती है:

* **ईमेल**: उपयोगकर्ता का ईमेल
* **usernamedisplayname**: उपयोगकर्ता का नाम
* **dropbox\_path**: Dropbox फ़ोल्डर का स्थान
* **Host\_id: Hash**: क्लाउड को प्रमाणित करने के लिए उपयोग किया जाने वाला हैश। इसे केवल वेब से रद्द किया जा सकता है।
* **Root\_ns**: उपयोगकर्ता पहचानकर्ता

डेटाबेस **`filecache.db`** में Dropbox के साथ सिंक्रनाइज़ किए गए सभी फ़ाइलों और फ़ोल्डरों के बारे में जानकारी होती है। टेबल `File_journal` सबसे उपयोगी जानकारी वाला है:

* **Server\_path**: सर्वर में फ़ाइल का स्थान (इस स्थान के पहले क्लाइंट के `host_id` से प्रारंभ होता है)।
* **local\_sjid**: फ़ाइल का संस्करण
* **local\_mtime**: संशोधन तिथि
* **local\_ctime**: निर्माण तिथि

इस डेटाबेस में अन्य टेबल में और भी रोचक जानकारी होती है:

* **block\_cache**: Dropbox के सभी फ़ाइलों और फ़ोल्डरों का हैश
* **block\_ref**: टेबल `block_cache` के हैश आईडी को टेबल `file_journal` में फ़ाइल आईडी के साथ संबंधित करता है
* **mount\_table**: Dropbox के साझा फ़ोल्डर
* **deleted\_fields**: Dropbox द्वारा हटाए गए फ़ाइलें
* **date\_added**

<figure><img src="../../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उपयोग करें और आसानी से **वर्कफ़्लो** बनाएं और संचालित करें, जो दुनिया के **सबसे उन्नत** समुदाय उपकरणों द्वारा संचालित होते हैं।\
आज ही पहुंच प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आप **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड** करना चाहते हैं? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह देखें
* [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल** हों या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का पालन करें**।**
* **अपने हैकिंग ट्रिक्स को** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **में पीआर जमा करके अपने हैकिंग ट्रिक्स साझा करें**।

</details>
