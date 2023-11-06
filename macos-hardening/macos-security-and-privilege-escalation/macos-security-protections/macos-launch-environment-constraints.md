# macOS लॉन्च/वातावरण प्रतिबंध

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करना चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एकल [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल हों** या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का** **अनुसरण** करें।**
* **अपने हैकिंग ट्रिक्स को** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में PR जमा करके साझा करें**
*
* .

</details>

## मूलभूत जानकारी

macOS में लॉन्च प्रतिबंधों को लागू करने का उद्देश्य है **प्रक्रिया को कैसे, कौन और कहां से प्रारंभ किया जा सकता है** की सुरक्षा को बढ़ाना। macOS Ventura में प्रारंभिक किए गए इन प्रतिबंधों ने **प्रत्येक सिस्टम बाइनरी को अलग प्रतिबंध श्रेणियों में वर्गीकृत** किया है, जो **विश्वास कैश** में परिभाषित हैं, जिसमें सिस्टम बाइनरी और उनके संबंधित हैश शामिल होते हैं। ये प्रतिबंध सिस्टम के हर एक्जीक्यूटेबल बाइनरी तक फैलते हैं, जिसमें किसी विशेष बाइनरी को लॉन्च करने के लिए आवश्यकताएं निर्धारित करने वाले **नियम** शामिल होते हैं। ये नियम स्वयं प्रतिबंधों को संतुष्ट करने के लिए होते हैं, जो एक बाइनरी को पूरा करना होता है, माता प्रक्रिया द्वारा पूरा करने के लिए आवश्यकताएं और अन्य संबंधित संस्थाओं द्वारा पालन करने के लिए आवश्यकताएं।

इस तंत्र को macOS Sonoma से शुरू होकर तीसरे पक्ष ऐप्स तक विस्तारित किया गया है, जो डेवलपर्स को उनके ऐप्स की सुरक्षा को सुरक्षित करने के लिए एक **पर्यावरण प्रतिबंधों के लिए कुंजी और मान निर्दिष्ट करने की** सुविधा प्रदान करता है।

आप **लॉन्च पर्यावरण और पुस्तकालय प्रतिबंधों** को प्रतिबंध शब्दकोश में वर्णित नियमों में परिभाषित करते हैं, जिन्हें आप या तो **`launchd` प्रॉपर्टी सूची फ़ाइलों** में सहेजते हैं, या कोड साइनिंग में उपयोग करने वाली **अलग प्रॉपर्टी सूची** फ़ाइलों में।

4 प्रकार के प्रतिबंध होते हैं:

* **स्वयं प्रतिबंध**: **चल रहे** बाइनरी पर लागू प्रतिबंध।
* **माता प्रक्रिया**: **प्रक्रिया के माता** पर लागू प्रतिबंध (उदाहरण के लिए **`launchd`** एक XP सेवा चला रहा है)
* **जिम्मेदार प्रतिबंध**: **सेवा को कॉल करने वाली प्रक्रिया** पर लागू प्रतिबंध
* **पुस्तकालय लोड प्रतिबंध**: पुस्तकालय लोड प्रतिबंध का उपयोग करके लोड किए जाने वाले कोड का विवरण देने के लिए प्रयोग करें

तो जब एक प्रक्रिया दूसरी प्रक्रिया को लॉन्च करने का प्रयास करती है - `execve(_:_:_:)` या `posix_spawn(_:_:_:_:_:_:)` को कॉल करके - ऑपरेटिंग सिस्टम यह जांचता है कि **एक्जीक्यूटेबल** फ़ाइल अपने **स्वयं प्रतिबंध** को पूरा करती है। यह भी जांचता है कि **माता प्रक्रिया** का एक्जीक्यूटेबल **प्रक्रिया के माता प्रतिबंध** को पूरा करता है, और कि **जिम्मेदार प्रक्रिया** का
```
Category 1:
Self Constraint: (on-authorized-authapfs-volume || on-system-volume) && launch-type == 1 && validation-category == 1
Parent Constraint: is-init-proc
```
* `(on-authorized-authapfs-volume || on-system-volume)`: सिस्टम या क्रिपटेक्स वॉल्यूम में होना चाहिए।
* `launch-type == 1`: सिस्टम सेवा होना चाहिए (लॉन्चडेमन्स में प्लिस्ट)।
* &#x20; `validation-category == 1`: ऑपरेटिंग सिस्टम का एक्सीक्यूटेबल होना चाहिए।
* `is-init-proc`: लॉन्च्ड

### LC श्रेणियों को उलटा करना

आपके पास इसके बारे में अधिक जानकारी [**यहां**](https://theevilbit.github.io/posts/launch\_constraints\_deep\_dive/#reversing-constraints) है, लेकिन मूल रूप से वे **AMFI (AppleMobileFileIntegrity)** में परिभाषित होते हैं, इसलिए आपको **कर्नल डेवलपमेंट किट** डाउनलोड करने की आवश्यकता होगी ताकि आप **KEXT** प्राप्त कर सकें। **`kConstraintCategory`** से शुरू होने वाले प्रतीक्षापूर्वक हैं। उन्हें निकालकर आपको एक DER (ASN.1) कोडेड स्ट्रीम मिलेगी जिसे आपको [ASN.1 डिकोडर](https://holtstrom.com/michael/tools/asn1decoder.php) या python-asn1 लाइब्रेरी और इसके `dump.py` स्क्रिप्ट, [andrivet/python-asn1](https://github.com/andrivet/python-asn1/tree/master) के साथ डिकोड करने की आवश्यकता होगी, जो आपको एक अधिक समझने योग्य स्ट्रिंग देगा।

## लॉन्च पर्यावरण की बाधाएँ

ये तीसरे पक्ष के अनुप्रयोगों में कॉन्फ़िगर की गई लॉन्च पर्यावरण बाधाएँ हैं। डेवलपर अपने अनुप्रयोग में एक्सेस को सीमित करने के लिए अपनी अनुप्रयोग में **तथ्य** और **
```bash
codesign -d -vvvv app.app
```
## विश्वास कैश

**macOS** में कुछ विश्वास कैश होते हैं:

* **`/System/Volumes/Preboot/*/boot/*/usr/standalone/firmware/FUD/BaseSystemTrustCache.img4`**
* **`/System/Volumes/Preboot/*/boot/*/usr/standalone/firmware/FUD/StaticTrustCache.img4`**
* **`/System/Library/Security/OSLaunchPolicyData`**

और iOS में ऐसा दिखता है कि यह **`/usr/standalone/firmware/FUD/StaticTrustCache.img4`** में होता है।

### विश्वास कैशों की गणना

पिछले विश्वास कैश फ़ाइलें **IMG4** और **IM4P** प्रारूप में होती हैं, जहां IM4P एक IMG4 प्रारूप का पेलोड अनुभाग होता है।

आप [**pyimg4**](https://github.com/m1stadev/PyIMG4) का उपयोग डेटाबेस के पेलोड को निकालने के लिए कर सकते हैं:

{% code overflow="wrap" %}
```bash
# Installation
python3 -m pip install pyimg4

# Extract payloads data
cp /System/Volumes/Preboot/*/boot/*/usr/standalone/firmware/FUD/BaseSystemTrustCache.img4 /tmp
pyimg4 img4 extract -i /tmp/BaseSystemTrustCache.img4 -p /tmp/BaseSystemTrustCache.im4p
pyimg4 im4p extract -i /tmp/BaseSystemTrustCache.im4p -o /tmp/BaseSystemTrustCache.data

cp /System/Volumes/Preboot/*/boot/*/usr/standalone/firmware/FUD/StaticTrustCache.img4 /tmp
pyimg4 img4 extract -i /tmp/StaticTrustCache.img4 -p /tmp/StaticTrustCache.im4p
pyimg4 im4p extract -i /tmp/StaticTrustCache.im4p -o /tmp/StaticTrustCache.data

pyimg4 im4p extract -i /System/Library/Security/OSLaunchPolicyData -o /tmp/OSLaunchPolicyData.data
```
{% endcode %}

(एक विकल्प यह हो सकता है कि आप टूल [**img4tool**](https://github.com/tihmstar/img4tool) का उपयोग करें, जो M1 में चलेगा, यदि रिलीज पुराना है और x86\_64 के लिए यदि आप इसे सही स्थानों पर स्थापित करते हैं).

अब आप टूल [**trustcache**](https://github.com/CRKatri/trustcache) का उपयोग करके पठनीय प्रारूप में जानकारी प्राप्त कर सकते हैं:
```bash
# Install
wget https://github.com/CRKatri/trustcache/releases/download/v2.0/trustcache_macos_arm64
sudo mv ./trustcache_macos_arm64 /usr/local/bin/trustcache
xattr -rc /usr/local/bin/trustcache
chmod +x /usr/local/bin/trustcache

# Run
trustcache info /tmp/OSLaunchPolicyData.data | head
trustcache info /tmp/StaticTrustCache.data | head
trustcache info /tmp/BaseSystemTrustCache.data | head

version = 2
uuid = 35EB5284-FD1E-4A5A-9EFB-4F79402BA6C0
entry count = 969
0065fc3204c9f0765049b82022e4aa5b44f3a9c8 [none] [2] [1]
00aab02b28f99a5da9b267910177c09a9bf488a2 [none] [2] [1]
0186a480beeee93050c6c4699520706729b63eff [none] [2] [2]
0191be4c08426793ff3658ee59138e70441fc98a [none] [2] [3]
01b57a71112235fc6241194058cea5c2c7be3eb1 [none] [2] [2]
01e6934cb8833314ea29640c3f633d740fc187f2 [none] [2] [2]
020bf8c388deaef2740d98223f3d2238b08bab56 [none] [2] [3]
```
विश्वास कैश का निम्नलिखित संरचना का पालन किया जाता है, इसलिए **LC श्रेणी 4 वें स्तंभ** होती है।
```c
struct trust_cache_entry2 {
uint8_t cdhash[CS_CDHASH_LEN];
uint8_t hash_type;
uint8_t flags;
uint8_t constraintCategory;
uint8_t reserved0;
} __attribute__((__packed__));
```
तो, आप डेटा निकालने के लिए [**इस तरह का स्क्रिप्ट**](https://gist.github.com/xpn/66dc3597acd48a4c31f5f77c3cc62f30) का उपयोग कर सकते हैं।

उस डेटा से आप **`0` के लॉन्च सीमाओं के साथ ऐप्स की जांच कर सकते हैं**, जो ऐसे होते हैं जो सीमित नहीं होते हैं ([**यहां जांचें**](https://gist.github.com/LinusHenze/4cd5d7ef057a144cda7234e2c247c056) कि प्रत्येक मान क्या है)।

## हमला निवारण

लॉन्च कंस्ट्रेंट्स ने कई पुराने हमलों को निवारण किया है जो **यह सुनिश्चित करते हैं कि प्रक्रिया अप्रत्याशित स्थितियों में नहीं चलाई जाएगी:** उदाहरण के लिए अप्रत्याशित स्थानों से या अप्रत्याशित माता-पिता प्रक्रिया द्वारा आह्वानित होने से (यदि केवल लॉन्चडी ही इसे लॉन्च करना चाहिए था)

इसके अलावा, लॉन्च कंस्ट्रेंट्स ने **डाउनग्रेड हमलों को भी निवारण किया है**।

हालांकि, यह **सामान्य XPC** दुरुपयोग, **इलेक्ट्रॉन** कोड इंजेक्शन या लाइब्रेरी सत्यापन के बिना **डायलिब इंजेक्शन** को निवारण नहीं करता है (जब तक लाइब्रेरी लोड करने वाले टीम आईडी पता नहीं हो)।

### XPC डेमन सुरक्षा

इस लेख लिखने के समय (सोनोमा रिलीज) में **डेमन XPC सेवा के लिए जिम्मेदार प्रक्रिया** जुड़े ग्राहक की बजाय खुद डेमन XPC सेवा होती है। (जमा किया गया FB: FB13206884)। एक सेकंड के लिए मान लेते हैं कि यह एक बग है, हम फिर भी **हमारे हमले को चालू करने में सक्षम नहीं होंगे**, लेकिन यदि यह **पहले से ही सक्रिय है** (शायद क्योंकि मूल ऐप ने इसे आह्वानित किया था), तो हमें इससे **जुड़ने से कुछ नहीं रोक सकता** है। इसलिए, यदि सीमा सेट करना एक अच्छा विचार हो सकता है, और यह हमले का समय-सीमा सीमित करेगा, लेकिन यह मुख्य समस्या को हल नहीं करता है, और हमारी XPC सेवा को अच्छी तरह से जुड़े ग्राहक की प्रमाणित करनी चाहिए। यह अभी भी इसी तरह काम करने का एकमात्र तरीका है। इसके अलावा, शुरुआत में उल्लिखित किया गया है कि यह अभी भी इसी तरह काम नहीं करता है।

### इलेक्ट्रॉन सुरक्षा

यदि इसकी आवश्यकता होती है कि ऐप्लिकेशन को **लॉन्च सर्विस द्वारा खोलना** होता है (माता-पिता की सीमाओं में)। इसे **`open`** का उपयोग करके (जो env variables सेट कर सकता है) या **लॉन्च सर्विसेज एपीआई** का उपयोग करके (जहां env variables दर्शाए जा सकते हैं) प्राप्त किया जा सकता है।

## संदर्भ

* [https://youtu.be/f1HA5QhLQ7Y?t=24146](https://youtu.be/f1HA5QhLQ7Y?t=24146)
* [https://theevilbit.github.io/posts/launch\_constraints\_deep\_dive/](https://theevilbit.github.io/posts/launch\_constraints\_deep\_dive/)
* [https://eclecticlight.co/2023/06/13/why-wont-a-system-app-or-command-tool-run-launch-constraints-and-trust-caches/](https://eclecticlight.co/2023/06/13/why-wont-a-system-app-or-command-tool-run-launch-constraints-and-trust-caches/)
* [https://developer.apple.com/videos/play/wwdc2023/10266/](https://developer.apple.com/videos/play/wwdc2023/10266/)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **हैकट्रिक्स में विज्ञापित** देखना चाहते हैं? या क्या आप **PEASS के नवीनतम संस्करण का उपयोग करना चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* प्राप्त करें [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **में**
*
* .

</details>
