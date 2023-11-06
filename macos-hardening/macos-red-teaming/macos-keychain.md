# macOS Keychain

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप एक **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में शामिल हों या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को।**

</details>

## मुख्य Keychains

* **उपयोगकर्ता Keychain** (`~/Library/Keychains/login.keycahin-db`), जिसका उपयोग **उपयोगकर्ता-विशिष्ट प्रमाणपत्र**, इंटरनेट पासवर्ड, उपयोगकर्ता द्वारा उत्पन्न प्रमाणपत्र, नेटवर्क पासवर्ड और उपयोगकर्ता द्वारा उत्पन्न सार्वजनिक/निजी कुंजी जैसे **उपयोगकर्ता-विशिष्ट प्रमाणपत्र** को संग्रहित करने के लिए किया जाता है।
* **सिस्टम Keychain** (`/Library/Keychains/System.keychain`), जिसमें **सिस्टम-व्यापक प्रमाणपत्र**, जैसे WiFi पासवर्ड, सिस्टम रूट प्रमाणपत्र, सिस्टम निजी कुंजी और सिस्टम एप्लिकेशन पासवर्ड संग्रहित होते हैं।

### पासवर्ड Keychain Access

ये फ़ाइलें, जबकि उनमें कोई आपूर्ति सुरक्षा नहीं है और **डाउनलोड की जा सकती हैं**, एन्क्रिप्टेड होती हैं और **उपयोगकर्ता के प्लेनटेक्स्ट पासवर्ड की आवश्यकता होती है** उन्हें डिक्रिप्ट करने के लिए। डिक्रिप्शन के लिए [**Chainbreaker**](https://github.com/n0fate/chainbreaker) जैसा उपकरण उपयोग किया जा सकता है।

## Keychain Entries Protections

### ACLs

Keychain में प्रत्येक प्रविष्टि को **पहुँच नियंत्रण सूची (ACLs)** द्वारा नियंत्रित किया जाता है, जो प्रविष्टि पर विभिन्न कार्रवाइयों को करने की अनुमति देती है, जैसे:

* **ACLAuhtorizationExportClear**: धारक को गुप्त ज्ञान का स्पष्ट पाठ प्राप्त करने की अनुमति देता है।
* **ACLAuhtorizationExportWrapped**: धारक को दिए गए एक अन्य पासवर्ड के साथ एन्क्रिप्टेड स्पष्ट पाठ प्राप्त करने की अनुमति देता है।
* **ACLAuhtorizationAny**: धारक को किसी भी कार्रवाई करने की अनुमति देता है।

ACLs के साथ-साथ एक **विश्वसनीय एप्लिकेशनों की सूची** भी होती है जो इन कार्रवाइयों को प्रदान किए बिना कर सकती है। इसमें शामिल हो सकता है:

* &#x20;**N`il`** (कोई अधिकृतता की आवश्यकता नहीं है, **सभी विश्वसनीय हैं**)
* एक **खाली** सूची (**कोई नहीं विश्वसनीय है**)
* विशिष्ट **एप्लिकेशनों** की **सूची**।

इसके अलावा प्रविष्टि में **`ACLAuthorizationPartitionID`** भी हो सकता है, जिसका उपयोग **teamid, apple,** और **cdhash** की पहचान करने के लिए किया जाता है।

* यदि **teamid** निर्दिष्ट है, तो प्रविष्टि मान **तालिका** के साथ **पहुँचने** के लिए उपयोग की जाने वाली एप्लिकेशन को **एक ही teamid** होनी चाहिए।
* यदि **apple** निर्दिष्ट है, तो ऐप को **Apple** द्वारा **साइन किया** जाना चाहिए।
* यदि **cdhash** निर्दिष्ट है, तो ऐप को विशिष्ट **cdhash** होना चाहिए।

### एक Keychain Entry बनाना

जब एक **नई प्रविष्टि** को **`Keychain Access.app`** का उपयोग करके बनाया जाता है, निम्नलिखित नियम लागू होते हैं:

* सभी ऐप्स एन्क्रिप्ट कर सकते हैं।
* **कोई ऐप** निर्यात/डिक्रिप्ट कर सकता है (उपयोगकर्ता को प्रश्न नहीं पूछता)।
* सभी ऐप अखंडता जांच देख सकते हैं।
* कोई ऐप
```bash
# Dump all metadata and decrypted secrets (a lot of pop-ups)
security dump-keychain -a -d

# Find generic password for the "Slack" account and print the secrets
security find-generic-password -a "Slack" -g

# Change the specified entrys PartitionID entry
security set-generic-password-parition-list -s "test service" -a "test acount" -S
```
### एपीआई

{% hint style="success" %}
**कीचेन एनुमरेशन और डंपिंग** जो **प्रॉम्प्ट नहीं उत्पन्न करेगा** उसके लिए उपकरण [**LockSmith**](https://github.com/its-a-feature/LockSmith) का उपयोग किया जा सकता है।
{% endhint %}

प्रत्येक कीचेन प्रविष्टि के बारे में **जानकारी** की सूची और प्राप्त करें:

* एपीआई **`SecItemCopyMatching`** प्रत्येक प्रविष्टि के बारे में जानकारी देती है और इसका उपयोग करते समय कुछ विशेषताएं आप सेट कर सकते हैं:
* **`kSecReturnData`**: यदि सत्य है, तो यह डेटा को डिक्रिप्ट करने का प्रयास करेगा (पोप-अप से बचने के लिए इसे असत्य सेट करें)
* **`kSecReturnRef`**: कीचेन आइटम के संदर्भ को भी प्राप्त करें (यदि बाद में आप देखते हैं कि आप पोप-अप के बिना डिक्रिप्ट कर सकते हैं तो सत्य सेट करें)
* **`kSecReturnAttributes`**: प्रविष्टियों के बारे में मेटाडेटा प्राप्त करें
* **`kSecMatchLimit`**: वापसी करने के लिए कितने परिणाम
* **`kSecClass`**: किस प्रकार की कीचेन प्रविष्टि

प्रत्येक प्रविष्टि के **ACL** प्राप्त करें:

* एपीआई **`SecAccessCopyACLList`** के साथ आप **कीचेन आइटम के लिए ACL** प्राप्त कर सकते हैं, और यह एक ACL की सूची (जैसे `ACLAuhtorizationExportClear` और पहले उल्लिखित अन्य) लौटाएगा जहां प्रत्येक सूची में होता है:
* विवरण
* **विश्वसनीय एप्लिकेशन सूची**। इसमें हो सकता है:
* एक ऐप: /Applications/Slack.app
* एक बाइनरी: /usr/libexec/airportd
* एक समूह: group://AirPort

डेटा निर्यात करें:

* एपीआई **`SecKeychainItemCopyContent`** प्लेनटेक्स्ट प्राप्त करता है
* एपीआई **`SecItemExport`** कुंजी और प्रमाणपत्रों को निर्यात करता है लेकिन सामग्री को एन्क्रिप्टेड निर्यात करने के लिए पासवर्ड सेट करना पड़ सकता है

और ये हैं **एक सीक्रेट को प्रॉम्प्ट के बिना निर्यात करने** के लिए **आवश्यकताएं**:

* यदि **1+ विश्वसनीय** ऐप्स सूचीबद्ध हैं:
* उचित **अधिकारीकरण** की आवश्यकता होती है (**`Nil`**, या सीक्रेट जानकारी तक पहुंच के लिए अनुमति देने की अनुमति वाली ऐप्स की सूची का हिस्सा होना चाहिए)
* कोड साइनेचर को मेल खाने के लिए **PartitionID** के साथ मेल खाने की आवश्यकता होती है
* कोड साइनेचर को विश्वसनीय ऐप के कोड साइनेचर के साथ मेल खाने की आवश्यकता होती है (या सही KeychainAccessGroup के सदस्य होना चाहिए)
* यदि **सभी ऐप्लिकेशन विश्वसनीय** हैं:
* उचित **अधिकारीकरण** की आवश्यकता होती है
* कोड साइनेचर को मेल खाने के लिए **PartitionID** के साथ मेल खाने की आवश्यकता होती है
* यदि **कोई PartitionID नहीं** है, तो यह आवश्यक नहीं है

{% hint style="danger" %}
इसलिए, यदि **1 ऐप्लिकेशन दर्ज है**, तो आपको **उस ऐप्लिकेशन में कोड इंजेक्शन करने** की आवश्यकता होती है।

यदि **पार्टीशनआईडी** में **एप्पल** निर्दिष्ट किया गया है, तो आप इसे **`osascript`** के साथ एक्सेस कर सकते हैं, इसलिए कुछ भी जो पार्टीशनआईडी में एप्पल के साथ सभी ऐप्लिकेशनों का विश्वास कर रहा है। **`Python`** भी इसके लिए उपयोग किया जा सकता है।
{% endhint %}

### दो अतिरिक्त विशेषताएं

* **अदृश्य**: यह एक बूलियन फ़्लैग है जो एंट्री को **UI** कीचेन ऐप से **छिपाने** के लिए होता है
* **सामान्य**: यह **मेटाडेटा** संग्रहीत करने के लिए होता है (इसलिए यह एन्क्रिप्टेड नहीं होता है)
* माइक्रोसॉफ्ट सभी संवेदनशील अंत बिंदु तक पहुंच के लिए सभी रिफ़्रेश टोकन को सादा पाठ में संग्रहीत कर रहा था।

## संदर्भ

* [**#OBTS v5.0: "Lock Picking the macOS Keychain" - Cody Thomas**](https://www.youtube.com/watch?v=jKE1ZW33JpY)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आप **PEASS के नवीनतम संस्करण या HackTricks को PDF म
