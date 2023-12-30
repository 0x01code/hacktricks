# स्थानीय क्लाउड स्टोरेज

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) या **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) को **फॉलो करें**.
* **अपनी हैकिंग ट्रिक्स साझा करें** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके.

</details>

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks) का उपयोग करके आसानी से **वर्कफ्लोज़ को बिल्ड और ऑटोमेट** करें जो दुनिया के **सबसे उन्नत** समुदाय टूल्स द्वारा संचालित होते हैं.\
आज ही एक्सेस प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## OneDrive

Windows में, आप OneDrive फोल्डर को `\Users\<username>\AppData\Local\Microsoft\OneDrive` में पा सकते हैं। और `logs\Personal` के अंदर `SyncDiagnostics.log` फाइल मिल सकती है जिसमें सिंक्रोनाइज़ की गई फाइलों के बारे में कुछ दिलचस्प डेटा होता है:

* बाइट्स में आकार
* निर्माण तिथि
* संशोधन तिथि
* क्लाउड में फाइलों की संख्या
* फोल्डर में फाइलों की संख्या
* **CID**: OneDrive उपयोगकर्ता की अद्वितीय ID
* रिपोर्ट जनरेशन समय
* OS के HD का आकार

CID मिल जाने के बाद, इस ID को शामिल करने वाली फाइलों की **खोज करने की सिफारिश की जाती है**। आपको _**\<CID>.ini**_ और _**\<CID>.dat**_ नाम की फाइलें मिल सकती हैं जिनमें OneDrive के साथ सिंक्रोनाइज़ की गई फाइलों के नाम जैसी दिलचस्प जानकारी हो सकती है।

## Google Drive

Windows में, आप मुख्य Google Drive फोल्डर को `\Users\<username>\AppData\Local\Google\Drive\user_default` में पा सकते हैं\
इस फोल्डर में Sync\_log.log नामक एक फाइल होती है जिसमें खाते का ईमेल पता, फाइलों के नाम, टाइमस्टैम्प, फाइलों के MD5 हैशेज आदि की जानकारी होती है। यहां तक कि हटाई गई फाइलें भी उस लॉग फाइल में उनके संबंधित MD5 के साथ दिखाई देती हैं।

**`Cloud_graph\Cloud_graph.db`** एक sqlite डेटाबेस है जिसमें **`cloud_graph_entry`** नामक टेबल होती है। इस टेबल में आप **सिंक्रोनाइज़** **फाइलों** का **नाम**, संशोधित समय, आकार, और फाइलों का MD5 चेकसम पा सकते हैं।

डेटाबेस **`Sync_config.db`** की टेबल डेटा में खाते का ईमेल पता, साझा फोल्डरों का पथ और Google Drive का संस्करण होता है।

## Dropbox

Dropbox **SQLite डेटाबेस** का उपयोग करके फाइलों का प्रबंधन करता है। इस\
आप डेटाबेस को निम्नलिखित फोल्डरों में पा सकते हैं:

* `\Users\<username>\AppData\Local\Dropbox`
* `\Users\<username>\AppData\Local\Dropbox\Instance1`
* `\Users\<username>\AppData\Roaming\Dropbox`

और मुख्य डेटाबेस हैं:

* Sigstore.dbx
* Filecache.dbx
* Deleted.dbx
* Config.dbx

".dbx" एक्सटेंशन का मतलब है कि **डेटाबेस** **एन्क्रिप्टेड** हैं। Dropbox **DPAPI** का उपयोग करता है ([https://docs.microsoft.com/en-us/previous-versions/ms995355(v=msdn.10)?redirectedfrom=MSDN](https://docs.microsoft.com/en-us/previous-versions/ms995355\(v=msdn.10\)?redirectedfrom=MSDN))

Dropbox द्वारा उपयोग किए गए एन्क्रिप्शन को बेहतर समझने के लिए आप [https://blog.digital-forensics.it/2017/04/brush-up-on-dropbox-dbx-decryption.html](https://blog.digital-forensics.it/2017/04/brush-up-on-dropbox-dbx-decryption.html) पढ़ सकते हैं।

हालांकि, मुख्य जानकारी है:

* **एंट्रोपी**: d114a55212655f74bd772e37e64aee9b
* **साल्ट**: 0D638C092E8B82FC452883F95F355B8E
* **एल्गोरिथम**: PBKDF2
* **इटरेशन्स**: 1066

इसके अलावा, डेटाबेस को डिक्रिप्ट करने के लिए आपको अभी भी चाहिए:

* **एन्क्रिप्टेड DPAPI की**: आप इसे रजिस्ट्री के अंदर `NTUSER.DAT\Software\Dropbox\ks\client` में पा सकते हैं (इस डेटा को बाइनरी के रूप में एक्सपोर्ट करें)
* **`SYSTEM`** और **`SECURITY`** हाइव्स
* **DPAPI मास्टर कीज़**: जो `\Users\<username>\AppData\Roaming\Microsoft\Protect` में मिल सकती हैं
* Windows उपयोगकर्ता का **उपयोगकर्ता नाम** और **पासवर्ड**

फिर आप [**DataProtectionDecryptor**](https://nirsoft.net/utils/dpapi_data_decryptor.html) टूल का उपयोग कर सकते हैं:

![](<../../../.gitbook/assets/image (448).png>)

यदि सब कुछ अपेक्षित रूप से चलता है, तो टूल आपको **प्राइमरी की** इंगित करेगा जिसे आपको मूल को पुनः प्राप्त करने के लिए **उपयोग करना होगा**। मूल को पुनः प्राप्त करने के लिए, बस इस [cyber_chef receipt](https://gchq.github.io/CyberChef/#recipe=Derive_PBKDF2_key(%7B'option':'Hex','string':'98FD6A76ECB87DE8DAB4623123402167'%7D,128,1066,'SHA1',%7B'option':'Hex','string':'0D638C092E8B82FC452883F95F355B8E'%7D)) का उपयोग करें और प्राइमरी की को "passphrase" के रूप में रिसीप्ट में डालें।

परिणामी हेक्स अंतिम कुंजी है जिसका उपयोग डेटाबेस को एन्क्रिप्ट करने के लिए किया गया था जिसे डिक्रिप्ट किया जा सकता है:
```bash
sqlite -k <Obtained Key> config.dbx ".backup config.db" #This decompress the config.dbx and creates a clear text backup in config.db
```
**`config.dbx`** डेटाबेस में निम्नलिखित जानकारी होती है:

* **Email**: उपयोगकर्ता का ईमेल
* **usernamedisplayname**: उपयोगकर्ता का नाम
* **dropbox\_path**: ड्रॉपबॉक्स फोल्डर का पथ
* **Host\_id: Hash** जिसका उपयोग क्लाउड में प्रमाणीकरण के लिए किया जाता है। इसे केवल वेब से ही निरस्त किया जा सकता है।
* **Root\_ns**: उपयोगकर्ता पहचानकर्ता

**`filecache.db`** डेटाबेस में ड्रॉपबॉक्स के साथ सिंक्रोनाइज़ किए गए सभी फाइलों और फोल्डरों की जानकारी होती है। `File_journal` तालिका में अधिक उपयोगी जानकारी होती है:

* **Server\_path**: सर्वर के अंदर फाइल का पथ (इस पथ से पहले क्लाइंट का `host_id` होता है)।
* **local\_sjid**: फाइल का संस्करण
* **local\_mtime**: संशोधन तिथि
* **local\_ctime**: निर्माण तिथि

इस डेटाबेस के अन्य तालिकाओं में और भी रोचक जानकारी होती है:

* **block\_cache**: ड्रॉपबॉक्स की सभी फाइलों और फोल्डरों का हैश
* **block\_ref**: `block_cache` तालिका के हैश ID को `file_journal` तालिका में फाइल ID के साथ संबंधित करता है
* **mount\_table**: ड्रॉपबॉक्स के शेयर फोल्डर
* **deleted\_fields**: ड्रॉपबॉक्स की हटाई गई फाइलें
* **date\_added**

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उपयोग करके आसानी से **workflows को बनाएं और ऑटोमेट करें** जो दुनिया के **सबसे उन्नत** समुदाय उपकरणों द्वारा संचालित होते हैं।\
आज ही पहुंच प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><strong>AWS हैकिंग सीखें शुरुआत से लेकर एक्सपर्ट तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** को फॉलो करें।**
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>
