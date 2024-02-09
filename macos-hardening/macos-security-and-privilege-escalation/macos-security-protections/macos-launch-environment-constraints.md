# macOS Launch/Environment Constraints & Trust Cache

<details>

<summary><strong>जीरो से हीरो तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित देखना चाहते हैं**? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का एक्सेस** चाहिए? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) की जाँच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह।
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें।
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) **डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या **टेलीग्राम समूह** या **मुझे** **ट्विटर** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में**।
*
* .

</details>

## मूल जानकारी

macOS में लॉन्च विवादों को **कैसे, कौन और कहाँ से प्रक्रिया प्रारंभ की जा सकती है**, इसे नियंत्रित करने के लिए पेश किया गया था। macOS Ventura में प्रारंभ किए गए इन्हें **प्रणाली बाइनरी को विभिन्न विवाद श्रेणियों में वर्गीकृत करने** के लिए एक ढांचा प्रदान करते हैं, जो **विश्वास भंडार** में सिस्टम बाइनरी और उनके संबंधित हैश को सूचीबद्ध करती है​। ये विवाद सिस्टम के हर एक क्रियात्मक बाइनरी तक फैलते हैं, जिसमें **किसी विशेष बाइनरी को लॉन्च करने के लिए आवश्यकताएं** की विवरणित सेट होती हैं। ये नियम उस बाइनरी को संतुष्ट करने के लिए आवश्यकताएं शामिल करते हैं, जिन्हें बाइनरी को पूरा करना होगा, माता-पिता विवाद जो इसके माता-प
```
Category 1:
Self Constraint: (on-authorized-authapfs-volume || on-system-volume) && launch-type == 1 && validation-category == 1
Parent Constraint: is-init-proc
```
* `(on-authorized-authapfs-volume || on-system-volume)`: सिस्टम या Cryptexes वॉल्यूम में होना चाहिए।
* `launch-type == 1`: सिस्टम सेवा होनी चाहिए (LaunchDaemons में plist)।
* `validation-category == 1`: ऑपरेटिंग सिस्टम एक्जीक्यूटेबल।
* `is-init-proc`: Launchd

### LC श्रेणियों को उलटाना

आपके पास इसके बारे में अधिक जानकारी [**यहाँ**](https://theevilbit.github.io/posts/launch\_constraints\_deep\_dive/#reversing-constraints) में है, लेकिन मूल रूप से, वे **AMFI (AppleMobileFileIntegrity)** में परिभाषित हैं, इसलिए आपको **कर्नेल डेवलपमेंट किट** डाउनलोड करने की आवश्यकता है ताकि आप **KEXT** प्राप्त कर सकें। **`kConstraintCategory`** से शुरू होने वाले प्रतीकचिह्न **रोचक** होते हैं। उन्हें निकालकर आपको एक DER (ASN.1) एन्कोडेड स्ट्रीम मिलेगा जिसे आपको [ASN.1 डिकोडर](https://holtstrom.com/michael/tools/asn1decoder.php) या पायथन-एएसएन.1 पुस्तकालय और इसके `dump.py` स्क्रिप्ट, [andrivet/python-asn1](https://github.com/andrivet/python-asn1/tree/master) के साथ डिकोड करने की आवश्यकता होगी जो आपको एक अधिक समझने योग्य स्ट्रिंग देगा।

## पर्यावरण प्रतिबंध

ये तीसरे पक्ष के एप्लिकेशन में विन्यासित लॉन्च प्रतिबंध हैं। डेवलपर अपने एप्लिकेशन में पहुंच को सीमित करने के लिए **तथ्य** और **
```bash
codesign -d -vvvv app.app
```
## विश्वास कैश

**macOS** में कुछ विश्वास कैश हैं:

- **`/System/Volumes/Preboot/*/boot/*/usr/standalone/firmware/FUD/BaseSystemTrustCache.img4`**
- **`/System/Volumes/Preboot/*/boot/*/usr/standalone/firmware/FUD/StaticTrustCache.img4`**
- **`/System/Library/Security/OSLaunchPolicyData`**

और iOS में ऐसा लगता है कि यह **`/usr/standalone/firmware/FUD/StaticTrustCache.img4`** में है।

{% hint style="warning" %}
Apple Silicon उपकरणों पर चल रहे macOS में, यदि एक Apple द्वारा हस्ताक्षरित बाइनरी विश्वास कैश में नहीं है, तो AMFI इसे लोड करने से इनकार करेगा।
{% endhint %}

### विश्वास कैशों की गणना

पिछले विश्वास कैश फ़ाइलें **IMG4** और **IM4P** प्रारूप में हैं, जिसमें IM4P एक IMG4 प्रारूप का भाग है।

आप [**pyimg4**](https://github.com/m1stadev/PyIMG4) का उपयोग डेटाबेस के पेलोड को निकालने के लिए कर सकते हैं:
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

(एक और विकल्प यह हो सकता है कि आप उपकरण [**img4tool**](https://github.com/tihmstar/img4tool) का उपयोग करें, जो M1 में भी चलेगा यदि रिलीज पुराना है और x86\_64 के लिए यदि आप इसे सही स्थानों पर स्थापित करते हैं).

अब आप उपकरण [**trustcache**](https://github.com/CRKatri/trustcache) का उपयोग करके सूचना को एक पठनीय प्रारूप में प्राप्त कर सकते हैं:
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
विश्वास कैश निम्नलिखित संरचना का पालन करता है, इसलिए **LC श्रेणी 4 वां स्तंभ है**।
```c
struct trust_cache_entry2 {
uint8_t cdhash[CS_CDHASH_LEN];
uint8_t hash_type;
uint8_t flags;
uint8_t constraintCategory;
uint8_t reserved0;
} __attribute__((__packed__));
```
तो, आप डेटा निकालने के लिए [**इस तरह का स्क्रिप्ट**](https://gist.github.com/xpn/66dc3597acd48a4c31f5f77c3cc62f30) उपयोग कर सकते हैं।

उस डेटा से आप **लॉन्च विवाद मूल्य `0`** वाले ऐप्स की जांच कर सकते हैं, जो वे हैं जो प्रतिबंधित नहीं हैं ([**यहाँ जांचें**](https://gist.github.com/LinusHenze/4cd5d7ef057a144cda7234e2c247c056) कि प्रत्येक मूल्य क्या है)।

## हमले से रोकथाम

लॉन्च विवाद कई पुराने हमलों को रोकने में सहायक होता है **यह सुनिश्चित करके कि प्रक्रिया अप्रत्याशित स्थितियों में नहीं चलाई जाएगी:** उदाहरण के लिए अप्रत्याशित स्थानों से या अप्रत्याशित माता प्रक्रिया द्वारा आमंत्रित किया जाने से (अगर केवल launchd को इसे लॉन्च करना चाहिए)

इसके अतिरिक्त, लॉन्च विवाद भी **डाउनग्रेड हमलों को रोकता है।**

हालांकि, यह **सामान्य XPC** दुरुपयोग, **Electron** कोड इंजेक्शन या पुस्तकालय सत्यापन के बिना **dylib इंजेक्शन** को नहीं रोकता (जब तक पुस्तकालयों को लोड करने वाले टीम आईडी पता हो)।

### XPC डेमन सुरक्षा

सोनोमा रिलीज में, एक महत्वपूर्ण बिंदु है डेमन XPC सेवा की **जिम्मेदारी कॉन्फ़िगरेशन**। XPC सेवा अपने आप के लिए ज़िम्मेदार है, जिसके विपरीत जुड़ने वाले क्लाइंट जिम्मेदार होते हैं। यह फीडबैक रिपोर्ट FB13206884 में दर्ज है। यह सेटअप दोषपूर्ण लग सकता है, क्योंकि यह निश्चित इंटरैक्शन को XPC सेवा के साथ संभव बनाता है:

- **XPC सेवा को लॉन्च करना**: यदि यह एक बग माना जाता है, तो यह सेटअप अटैकर को XPC सेवा को प्रारंभ करने की अनुमति नहीं देता।
- **सक्रिय सेवा से कनेक्ट करना**: यदि XPC सेवा पहले से चल रही है (संभावना अपने मूल एप्लिकेशन द्वारा सक्रिय की गई), तो इसे कनेक्ट करने के कोई बाधाएँ नहीं हैं।

XPC सेवा पर प्रतिबंधों को लागू करना हमलों के लिए **संभावित हमलों के लिए खिड़की को संक्षेपित करने** में फायदेमंद हो सकता है, लेकिन यह मुख्य चिंता का समाधान नहीं करता है। XPC सेवा की सुरक्षा सुनिश्चित करने के लिए मुख्य रूप से **कनेक्टिंग क्लाइंट को प्रभावी ढंग से सत्यापित करना** आवश्यक है। यह सेवा की सुरक्षा को मजबूत करने का एकमात्र तरीका रहता है। इसके अलावा, यह उल्लेखनीय जिम्मेदारी कॉन्फ़िगरेशन वर्तमान में संचालित है, जो निर्धारित डिज़ाइन के साथ मेल नहीं खा सकता है।

### इलेक्ट्रॉन सुरक्षा

यदि ऐप्लिकेशन को **लॉन्च सर्विस द्वारा खोलना आवश्यक है** (माता की प्रतिबंधों में)। यह **`open`** (जो एनवी वेरिएबल सेट कर सकता है) या **लॉन्च सेविसेस एपीआई** (जिसमें एनवी वेरिएबल निर्दिष्ट किए जा सकते हैं) का उपयोग करके प्राप्त किया जा सकता है।

## संदर्भ

* [https://youtu.be/f1HA5QhLQ7Y?t=24146](https://youtu.be/f1HA5QhLQ7Y?t=24146)
* [https://theevilbit.github.io/posts/launch\_constraints\_deep\_dive/](https://theevilbit.github.io/posts/launch\_constraints\_deep\_dive/)
* [https://eclecticlight.co/2023/06/13/why-wont-a-system-app-or-command-tool-run-launch-constraints-and-trust-caches/](https://eclecticlight.co/2023/06/13/why-wont-a-system-app-or-command-tool-run-launch-constraints-and-trust-caches/)
* [https://developer.apple.com/videos/play/wwdc2023/10266/](https://developer.apple.com/videos/play/wwdc2023/10266/)

<details>

<summary><strong>जीरो से हीरो तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी का हैकट्रिक्स में विज्ञापन देखना चाहते हैं**? या क्या आप **PEASS के नवीनतम संस्करण का उपयोग करना चाहते हैं या हैकट्रिक्स को पीडीएफ में डाउनलोड करना चाहते हैं**? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**द पीएएस फैमिली**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**एनएफटीएस**](https://opensea.io/collection/the-peass-family) संग्रह
* प्राप्त करें [**आधिकारिक पीएएस और हैकट्रिक्स स्वैग**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर फॉलो करें 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**।**
* **हैकिंग ट्रिक्स साझा करें, हैकट्रिक्स रेपो** में पीआर जमा करके और **हैकट्रिक्स-क्लाउड रेपो** में पीआर जमा करके।
*
* .

</details>
