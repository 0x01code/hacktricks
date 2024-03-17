# स्थानीय क्लाउड स्टोरेज

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप चाहते हैं कि आपकी **कंपनी HackTricks में विज्ञापित हो** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** द्वारा PRs सबमिट करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उपयोग करें और दुनिया के **सबसे उन्नत समुदाय उपकरणों** द्वारा संचालित **वर्कफ़्लो** को आसानी से बनाएं और स्वचालित करें।\
आज ही पहुंचें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## OneDrive

Windows में, आप OneDrive फ़ोल्डर को `\Users\<username>\AppData\Local\Microsoft\OneDrive` में पा सकते हैं। और `logs\Personal` के अंदर फ़ाइल `SyncDiagnostics.log` में कुछ दिलचस्प डेटा पाया जा सकता है जो सिंक्रनाइज़ की गई फ़ाइलों के बारे में है:

* बाइट में आकार
* निर्माण तिथि
* संशोधन तिथि
* क्लाउड में फ़ाइलों की संख्या
* फ़ोल्डर में फ़ाइलों की संख्या
* **CID**: OneDrive उपयोगकर्ता की अद्वितीय पहचान
* रिपोर्ट जनरेशन समय
* ओएस के HD का आकार

जैसे ही आप CID पाएंगे, सिफ़र इस आईडी को शामिल फ़ाइलें खोजने की सिफारिश करता है। आपको _**\<CID>.ini**_ और _**\<CID>.dat**_ नाम की फ़ाइलें मिल सकती हैं जो OneDrive के साथ सिंक्रनाइज़ की गई फ़ाइलों के नाम जैसी दिलचस्प जानकारी शामिल कर सकती हैं।

## Google Drive

Windows में, आप मुख्य Google Drive फ़ोल्डर को `\Users\<username>\AppData\Local\Google\Drive\user_default` में पा सकते हैं\
इस फ़ोल्डर में एक फ़ाइल होती है जिसका नाम है Sync\_log.log जिसमें खाते का ईमेल पता, फ़ाइलों के नाम, समय चिह्न, फ़ाइलों के MD5 हैश आदि जैसी जानकारी होती है। हटाई गई फ़ाइलें भी उस लॉग फ़ाइल में उनके संबंधित MD5 के साथ दिखाई देती हैं।

फ़ाइल **`Cloud_graph\Cloud_graph.db`** एक sqlite डेटाबेस है जिसमें तालिका **`cloud_graph_entry`** होती है। इस तालिका में आप सिंक्रनाइज़ की गई **फ़ाइलों** का **नाम**, संशोधित समय, आकार, और फ़ाइलों का MD5 checksum पा सकते हैं।

डेटाबेस **`Sync_config.db`** की तालिका डेटा में खाते का ईमेल पता, साझा किए गए फ़ोल्डरों का पथ और Google Drive संस्करण होता है।

## Dropbox

Dropbox फ़ाइलों को प्रबंधित करने के लिए **SQLite डेटाबेस** का उपयोग करता है। इसमें\
आप फोल्डर में डेटाबेस पा सकते हैं:

* `\Users\<username>\AppData\Local\Dropbox`
* `\Users\<username>\AppData\Local\Dropbox\Instance1`
* `\Users\<username>\AppData\Roaming\Dropbox`

और मुख्य डेटाबेस हैं:

* Sigstore.dbx
* Filecache.dbx
* Deleted.dbx
* Config.dbx

".dbx" एक्सटेंशन यह दिखाता है कि **डेटाबेस** **एन्क्रिप्टेड** हैं। Dropbox **DPAPI** का उपयोग करता है ([https://docs.microsoft.com/en-us/previous-versions/ms995355(v=msdn.10)?redirectedfrom=MSDN](https://docs.microsoft.com/en-us/previous-versions/ms995355\(v=msdn.10\)?redirectedfrom=MSDN))

Dropbox द्वारा उपयोग की जाने वाली एन्क्रिप्शन को बेहतर समझने के लिए आप [https://blog.digital-forensics.it/2017/04/brush-up-on-dropbox-dbx-decryption.html](https://blog.digital-forensics.it/2017/04/brush-up-on-dropbox-dbx-decryption.html) पढ़ सकते हैं।

हालांकि, मुख्य जानकारी यह है:

* **एंट्रोपी**: d114a55212655f74bd772e37e64aee9b
* **सॉल्ट**: 0D638C092E8B82FC452883F95F355B8E
* **एल्गोरिथ्म**: PBKDF2
* **इटरेशन्स**: 1066

उस जानकारी के अलावा, डेटाबेस को डिक्रिप्ट करने के लिए आपको अभी भी चाहिए होता है:

* **एन्क्रिप्टेड DPAPI कुंजी**: आप इसे रजिस्ट्री में `NTUSER.DAT\Software\Dropbox\ks\client` के अंदर मिल सकती है (इस डेटा को बाइनरी के रूप में निर्यात करें)
* **`SYSTEM`** और **`SECURITY`** हाइव्स
* **DPAPI मास्टर कुंजी**: जो `\Users\<username>\AppData\Roaming\Microsoft\Protect` में पाई जा सकती हैं
* Windows उपयोगकर्ता का **उपयोगकर्ता नाम** और **पासवर्ड**

फिर आप उपकरण [**DataProtectionDecryptor**](https://nirsoft.net/utils/dpapi\_data\_decryptor.html)** का उपयोग कर सकते हैं:**

![](<../../../.gitbook/assets/image (448).png>)

यदि सब कुछ अपेक्षित रूप से होता है, तो उपकरण आपको संकेत देगा कि आपको मूल को वापस प्राप्त करने के लिए कौन सी प्राथमिक कुंजी का उपयोग करना है। मूल को पुनः प्राप्त करने के लिए, बस इस [cyber\_chef रसीपी](https://gchq.github.io/CyberChef/#recipe=Derive\_PBKDF2\_key\(%7B'option':'Hex','string':'98FD6A76ECB87DE8DAB4623123402167'%7D,128,1066,'SHA1',%7B'option':'Hex','string':'0D638C092E8B82FC452883F95F355B8E'%7D\) का उपयोग करें और प्राथमिक कुंजी को "पासफ़्रेज" के रूप में इस रसीपी के अंदर डालें।

परिणामी हेक्स वह अंतिम कुंजी है जिसका उपयोग डेटाबेस को एन्क्रिप्ट करने के लिए किया जाता है जिसे निम्नलिखित के साथ डिक्रिप्ट किया जा सकता है:
```bash
sqlite -k <Obtained Key> config.dbx ".backup config.db" #This decompress the config.dbx and creates a clear text backup in config.db
```
**`config.dbx`** डेटाबेस में निम्नलिखित जानकारी होती है:

* **ईमेल**: उपयोगकर्ता का ईमेल
* **usernamedisplayname**: उपयोगकर्ता का नाम
* **dropbox\_path**: ड्रॉपबॉक्स फ़ोल्डर का स्थान
* **Host\_id: Hash**: क्लाउड को प्रमाणित करने के लिए उपयोग किया जाने वाला हैश। इसे केवल वेब से रद्द किया जा सकता है।
* **Root\_ns**: उपयोगकर्ता पहचानकर्ता

**`filecache.db`** डेटाबेस में ड्रॉपबॉक्स के साथ सिंक्रनाइज़ किए गए सभी फ़ाइलों और फ़ोल्डर्स के बारे में जानकारी होती है। तालिका `File_journal` सबसे उपयोगी जानकारी वाला है:

* **Server\_path**: सर्वर के अंदर फ़ाइल का स्थान (यह पथ क्लाइंट के `host_id` से पूर्ववत है)।
* **local\_sjid**: फ़ाइल का संस्करण
* **local\_mtime**: संशोधन तिथि
* **local\_ctime**: निर्माण तिथि

इस डेटाबेस के अन्य तालिकाएँ और भी दिलचस्प जानकारी रखती हैं:

* **block\_cache**: ड्रॉपबॉक्स की सभी फ़ाइलों और फ़ोल्डर्स का हैश
* **block\_ref**: तालिका `block_cache` के हैश आईडी को तालिका `file_journal` में फ़ाइल आईडी के साथ संबंधित करता है।
* **mount\_table**: ड्रॉपबॉक्स के साझा फ़ोल्डर
* **deleted\_fields**: ड्रॉपबॉक्स ने हटाई गई फ़ाइलें
* **date\_added**

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks) का उपयोग करें और आसानी से **वर्कफ़्लो** बनाएं और **स्वचालित** करें जो दुनिया के **सबसे उन्नत** समुदाय उपकरणों द्वारा संचालित है।\
आज ही पहुंच प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी कंपनी का विज्ञापन **HackTricks** में देखना चाहते हैं या **PDF** में **HackTricks** डाउनलोड करना चाहते हैं तो [**सब्सक्रिप्शन प्लान**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)** पर फ़ॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** हैकिंग ट्रिक्स को पीआर के माध्यम से **HackTricks** और **HackTricks Cloud** github रेपो में सबमिट करके।

</details>
