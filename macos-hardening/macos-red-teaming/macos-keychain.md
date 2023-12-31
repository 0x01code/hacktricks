# macOS Keychain

<details>

<summary><strong>Learn AWS hacking from zero to hero with</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **follow** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>

## मुख्य Keychains

* **User Keychain** (`~/Library/Keychains/login.keycahin-db`), जिसका उपयोग **उपयोगकर्ता-विशिष्ट प्रमाण-पत्र** जैसे एप्लिकेशन पासवर्ड, इंटरनेट पासवर्ड, उपयोगकर्ता-निर्मित प्रमाणपत्र, नेटवर्क पासवर्ड, और उपयोगकर्ता-निर्मित सार्वजनिक/निजी कुंजियों को संग्रहीत करने के लिए किया जाता है।
* **System Keychain** (`/Library/Keychains/System.keychain`), जो **सिस्टम-व्यापी प्रमाण-पत्र** जैसे WiFi पासवर्ड, सिस्टम रूट प्रमाणपत्र, सिस्टम निजी कुंजियाँ, और सिस्टम एप्लिकेशन पासवर्ड संग्रहीत करता है।

### Password Keychain Access

ये फाइलें, जबकि इनमें स्वाभाविक सुरक्षा नहीं होती और इन्हें **डाउनलोड** किया जा सकता है, एन्क्रिप्टेड होती हैं और इन्हें डिक्रिप्ट करने के लिए **उपयोगकर्ता के प्लेनटेक्स्ट पासवर्ड की आवश्यकता होती है**। डिक्रिप्शन के लिए [**Chainbreaker**](https://github.com/n0fate/chainbreaker) जैसे टूल का उपयोग किया जा सकता है।

## Keychain Entries Protections

### ACLs

Keychain में प्रत्येक प्रविष्टि **Access Control Lists (ACLs)** द्वारा शासित होती है, जो यह निर्धारित करती हैं कि कौन किस प्रकार की क्रियाएं कर सकता है, जिसमें शामिल हैं:

* **ACLAuhtorizationExportClear**: प्रधारक को गुप्त संदेश का स्पष्ट पाठ प्राप्त करने की अनुमति देता है।
* **ACLAuhtorizationExportWrapped**: प्रधारक को दूसरे प्रदान किए गए पासवर्ड के साथ एन्क्रिप्टेड स्पष्ट पाठ प्राप्त करने की अनुमति देता है।
* **ACLAuhtorizationAny**: प्रधारक को कोई भी क्रिया करने की अनुमति देता है।

ACLs के साथ एक **विश्वसनीय एप्लिकेशनों की सूची** भी होती है जो बिना प्रॉम्प्ट के ये क्रियाएं कर सकते हैं। यह हो सकता है:

* &#x20;**N`il`** (कोई अधिकार आवश्यक नहीं, **सभी विश्वसनीय हैं**)
* एक **खाली** सूची (**कोई भी** विश्वसनीय नहीं है)
* विशिष्ट **एप्लिकेशनों** की **सूची**।

साथ ही प्रविष्टि में **`ACLAuthorizationPartitionID`,** कुंजी भी हो सकती है, जिसका उपयोग **teamid, apple,** और **cdhash** की पहचान करने के लिए किया जाता है।

* यदि **teamid** निर्दिष्ट है, तो प्रविष्टि मूल्य तक **बिना प्रॉम्प्ट के पहुँचने** के लिए उपयोग किए गए एप्लिकेशन के पास **समान teamid** होना चाहिए।
* यदि **apple** निर्दिष्ट है, तो एप्लिकेशन को **Apple** द्वारा **साइन** किया जाना चाहिए।
* यदि **cdhash** इंगित किया गया है, तो **एप्लिकेशन** के पास विशिष्ट **cdhash** होना चाहिए।

### Keychain Entry बनाना

जब **नई** **प्रविष्टि** **`Keychain Access.app`** का उपयोग करके बनाई जाती है, तो निम्नलिखित नियम लागू होते हैं:

* सभी एप्लिकेशन एन्क्रिप्ट कर सकते हैं।
* **कोई भी एप्लिकेशन** निर्यात/डिक्रिप्ट नहीं कर सकता (उपयोगकर्ता को प्रॉम्प्ट किए बिना)।
* सभी एप्लिकेशन अखंडता जांच देख सकते हैं।
* कोई भी एप्लिकेशन ACLs बदल नहीं सकता।
* **partitionID** को **`apple`** पर सेट किया जाता है।

जब कोई **एप्लिकेशन keychain में प्रविष्टि बनाता है**, तो नियम थोड़े अलग होते हैं:

* सभी एप्लिकेशन एन्क्रिप्ट कर सकते हैं।
* केवल **प्रविष्टि बनाने वाला एप्लिकेशन** (या विशेष रूप से जोड़े गए अन्य एप्लिकेशन) निर्यात/डिक्रिप्ट कर सकते हैं (उपयोगकर्ता को प्रॉम्प्ट किए बिना)।
* सभी एप्लिकेशन अखंडता जांच देख सकते हैं।
* कोई भी एप्लिकेशन ACLs बदल नहीं सकता।
* **partitionID** को **`teamid:[teamID here]`** पर सेट किया जाता है।

## Keychain तक पहुँच

### `security`
```bash
# Dump all metadata and decrypted secrets (a lot of pop-ups)
security dump-keychain -a -d

# Find generic password for the "Slack" account and print the secrets
security find-generic-password -a "Slack" -g

# Change the specified entrys PartitionID entry
security set-generic-password-parition-list -s "test service" -a "test acount" -S
```
### APIs

{% hint style="success" %}
**keychain enumeration और dumping** जो **prompt नहीं उत्पन्न करेगा** उसे [**LockSmith**](https://github.com/its-a-feature/LockSmith) टूल के साथ किया जा सकता है।
{% endhint %}

प्रत्येक keychain प्रविष्टि के बारे में **जानकारी** प्राप्त करें:

* API **`SecItemCopyMatching`** प्रत्येक प्रविष्टि के बारे में जानकारी देता है और इसका उपयोग करते समय आप कुछ विशेषताएँ सेट कर सकते हैं:
* **`kSecReturnData`**: यदि सच है, तो यह डेटा को डिक्रिप्ट करने की कोशिश करेगा (पॉप-अप से बचने के लिए false पर सेट करें)
* **`kSecReturnRef`**: keychain आइटम का संदर्भ भी प्राप्त करें (यदि बाद में आप देखते हैं कि आप पॉप-अप के बिना डिक्रिप्ट कर सकते हैं तो सच में सेट करें)
* **`kSecReturnAttributes`**: प्रविष्टियों के बारे में मेटाडेटा प्राप्त करें
* **`kSecMatchLimit`**: कितने परिणाम लौटाने हैं
* **`kSecClass`**: keychain प्रविष्टि का किस प्रकार है

प्रत्येक प्रविष्टि के **ACLs** प्राप्त करें:

* API **`SecAccessCopyACLList`** के साथ आप **keychain आइटम के लिए ACL** प्राप्त कर सकते हैं, और यह ACLs की एक सूची लौटाएगा (जैसे `ACLAuhtorizationExportClear` और पहले उल्लिखित अन्य) जहाँ प्रत्येक सूची में होगा:
* विवरण
* **Trusted Application List**। यह हो सकता है:
* एक ऐप: /Applications/Slack.app
* एक बाइनरी: /usr/libexec/airportd
* एक समूह: group://AirPort

डेटा निर्यात करें:

* API **`SecKeychainItemCopyContent`** प्लेनटेक्स्ट प्राप्त करता है
* API **`SecItemExport`** कुंजियों और प्रमाणपत्रों को निर्यात करता है लेकिन सामग्री को एन्क्रिप्टेड निर्यात करने के लिए पासवर्ड सेट करना पड़ सकता है

और ये **आवश्यकताएँ** हैं जो **बिना prompt के एक रहस्य को निर्यात करने के लिए**:

* यदि **1+ trusted** ऐप्स सूचीबद्ध हैं:
* उचित **authorizations** की आवश्यकता होती है (**`Nil`**, या रहस्य जानकारी तक पहुँचने के लिए अनुमति सूची के ऐप्स का **भाग** होना)
* कोड हस्ताक्षर का **PartitionID** से मेल होना चाहिए
* कोड हस्ताक्षर का एक **trusted app** के साथ मेल होना चाहिए (या सही KeychainAccessGroup का सदस्य होना चाहिए)
* यदि **सभी अनुप्रयोगों को विश्वास है**:
* उचित **authorizations** की आवश्यकता होती है
* कोड हस्ताक्षर का **PartitionID** से मेल होना चाहिए
* यदि **कोई PartitionID नहीं है**, तो यह आवश्यक नहीं है

{% hint style="danger" %}
इसलिए, यदि **1 अनुप्रयोग सूचीबद्ध है**, तो आपको उस अनुप्रयोग में कोड **इंजेक्ट करना होगा**।

यदि **apple** को **partitionID** में इंगित किया गया है, तो आप इसे **`osascript`** के साथ पहुँच सकते हैं, इसलिए कुछ भी जो सभी अनुप्रयोगों को apple के साथ partitionID में विश्वास करता है। **`Python`** भी इसके लिए उपयोग किया जा सकता है।
{% endhint %}

### दो अतिरिक्त विशेषताएँ

* **Invisible**: यह एक बूलियन फ्लैग है जो प्रविष्टि को **UI** Keychain ऐप से **छिपाने** के लिए है
* **General**: यह **मेटाडेटा** स्टोर करने के लिए है (इसलिए यह ENCRYPTED नहीं है)
* Microsoft संवेदनशील एंडपॉइंट तक पहुँचने के लिए सभी रिफ्रेश टोकन को सादे पाठ में स्टोर कर रहा था।

## संदर्भ

* [**#OBTS v5.0: "Lock Picking the macOS Keychain" - Cody Thomas**](https://www.youtube.com/watch?v=jKE1ZW33JpY)

<details>

<summary><strong>Learn AWS hacking from zero to hero with</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं या **HackTricks को PDF में डाउनलोड** करना चाहते हैं तो [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) देखें!
* [**official PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा [**NFTs**](https://opensea.io/collection/the-peass-family) का विशेष संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** का पालन करें**।
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>
