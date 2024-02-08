# macOS Launch/Environment Constraints & Trust Cache

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं**? या क्या आप **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने की इच्छा रखते हैं**? [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) की जाँच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह।
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें।
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मेरा** **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का** **पालन** करें।
* **हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में**।
*
* .

</details>

## मूल जानकारी

macOS में लॉन्च विवादों का उद्देश्य है **प्रक्रिया को कैसे, किसके द्वारा और कहाँ प्रारंभ किया जा सकता है** को नियंत्रित करके सुरक्षा को बढ़ावा देना। macOS Ventura में प्रारंभ किए गए इन विवादों ने **प्रत्येक सिस्टम बाइनरी को विभिन्न विवाद श्रेणियों में वर्गीकृत करने** के लिए एक ढांचा प्रदान किया, जो **विश्वास भंडार** में सिस्टम बाइनरी और उनके संबंधित हैश शामिल करने वाली सूची है। ये विवाद सिस्टम के हर एक निष्पादन बाइनरी तक फैले हुए हैं, जिसमें एक विशेष बाइनरी को लॉन्च करने के लिए आवश्यकताएं व्यक्त करने वाले नियम शामिल हैं। ये नियम उस बाइनरी को विशेष बाइनरी को लॉन्च करने के लिए आवश्यकताएं व्यक्त करने वाले नियमों का विस्तार करते हैं। ये नियम उस बाइनरी को विशेष बाइनरी को लॉन्च करने के लिए आवश्यकताएं व्यक्त करने वाले नियमों का विस्तार करते हैं। ये नियम उस बाइनरी को विशेष बाइनरी को लॉन्च करने के लिए आवश्यकताएं व्यक्त करने वाले नियमों का विस्तार करते हैं।

यह तंत्र तीसरे पक्ष के ऐप्स तक **पर्यावरण विवादों** के माध्यम से फैलता है, जो macOS Sonoma से प्रारंभ होता है, जिसके द्वारा डेवलपर्स अपने ऐप्स को सुरक्षित रखने के लिए **पर्यावरण विवादों के लिए एक सेट की तरह की कुंजियों और मानों को निर्दिष्ट कर सकते हैं**।

आप **लॉन्च पर्यावरण और पुस्तकालय विवादों** को विवाद शब्दकोशों में परिभाषित करते हैं जिन्हें आप या तो **`launchd` संपत्ति सूची फ़ाइलों** में सहेजते हैं, या **कोड साइनिंग में उपयोग करने वाली अलग संपत्ति सूची फ़ाइलों** में।

4 प्रकार के विवाद हैं:

* **स्वयं विवाद**: **चल रहे** बाइनरी पर लागू विवाद।
* **माता प्रक्रिया**: **प्रक्रिया के माता** पर लागू विवाद (उदाहरण के लिए **`launchd`** एक XP सेवा चला रहा है)
* **जिम्मेदार विवाद**: **सेवा को कॉल करने वाली प्रक्रिया** पर लागू विवाद XPC संचार में
* **पुस्तकालय लोड विवाद**: लोड किया जा सकने वाले कोड का विवरण विस्तार से वर्णन करने के लिए पुस्तकालय लोड विवाद का उपयोग करें

तो जब एक प्रक्रिया को दूसरी प्रक्रिया को लॉन्च करने की कोशिश करती है — `execve(_:_:_:)` या `posix_spawn(_:_:_:_:_:_:)` को कॉल करके — ऑपरेटिंग सिस्टम यह जांचता है कि **एक्जीक्यूटेबल** फ़ाइल अपनी **अपनी स्वयं विवाद** को **पूरा करता है**। यह भी जांचता है कि **माता प्रक्रिया** का एक्जीक्यूटेबल **अपने विवाद को पूरा करता है**, और कि **जिम्मेदार प्रक्रिया** का एक्जीक्यूटेबल **अपने विवाद को पूरा करता है**। यदि इन लॉन्च विवादों में से कोई भी पूरा नहीं होता है, तो ऑपरेटिंग सिस्टम प्रोग्राम को नहीं चलाता।

यदि किसी पुस्तकालय को लोड करते समय किसी भी हिस्से का **पुस्तकालय विवाद सत्य नहीं है**, तो आपकी प्रक्रिया **पुस्तकालय को लोड नहीं** करती है।

## LC श्रेणियाँ

एलसी एक **तथ्य** और **तार्किक क्रियाएँ** (और, या..) द्वारा संयोजित होता है जो तथ्यों को संयोजित करता है।

[**एलसी जो एलसी एक उपयोग कर सकता है वह दस्तावेज़ीकृत है**](https://developer.apple.com/documentation/security/defining\_launch\_environment\_and\_library\_constraints)। उदाहरण के लिए:

* is-init-proc: एक बूलियन मान जो दर्शाता है कि एक्जीक्यूटेबल ऑपरेटिंग सिस्टम की प्रारंभिकीकरण प्रक्रिया (`launchd`) होना चाहिए।
* is-sip-protected: एक बूलियन मान जो दर्शाता है कि एक्जीक्यूटेबल को सिस्टम अखंडता सुरक्षा (SIP) द्वारा संरक्षित फ़ाइल होना चाहिए।
* `on-authorized-authapfs-volume:` एक बूलियन मान जो दर्शाता है कि ऑपरेटिंग सिस्टम ने एक अधिकृत, प्रमाणित APFS वॉल्यूम से एक्जीक्यूटेबल लोड किया है।
* `on-authorized-authapfs-volume`: एक बूलियन मान जो दर्शाता है कि ऑपरेटिंग सिस्टम ने एक अधिकृत, प्रमाणित APFS वॉल्यूम से एक्जीक्यूटेबल लोड किया है।
* Cryptexes volume
* `on-system-volume:` एक बूलियन मान जो दर्शाता है कि ऑपरेटिंग सिस्टम ने वर्तमान में बूट किए गए सिस्टम वॉल्यूम से एक्जीक्यूटेबल लोड किया है।
* Inside /System...
* ...

जब एक Apple बाइनरी को साइन किया जाता है तो यह उसे **एलसी श्रेणी में निर्धारित करता है** विश्वास भंडार के अंदर।

* **iOS 16 LC categories** [**यहाँ पर उलटे और दस्तावेज़ीकृत किए गए हैं**](https://gist.github.com/LinusHenze/4cd5d7ef057a144cda7234e2c247c056)।
* वर्तमान **LC categories (macOS 14** - Somona) को उलटा गया है और उनकी [**विवरण यहाँ पाए जा सकते हैं**](https://gist.github.com/theevilbit/a6fef1e0397425a334d064f7b6e1be53)।

उदाहरण के लिए श्रेणी 1 है:
```
Category 1:
Self Constraint: (on-authorized-authapfs-volume || on-system-volume) && launch-type == 1 && validation-category == 1
Parent Constraint: is-init-proc
```
* `(on-authorized-authapfs-volume || on-system-volume)`: सिस्टम या Cryptexes वॉल्यूम में होना चाहिए।
* `launch-type == 1`: सिस्टम सेवा होनी चाहिए (LaunchDaemons में plist).
* `validation-category == 1`: ऑपरेटिंग सिस्टम एक्जीक्यूटेबल।
* `is-init-proc`: Launchd

### LC श्रेणियों को उलटाना

आपके पास इसके बारे में अधिक जानकारी [**यहाँ**](https://theevilbit.github.io/posts/launch\_constraints\_deep\_dive/#reversing-constraints) में है, लेकिन मूल रूप से, वे **AMFI (AppleMobileFileIntegrity)** में परिभाषित हैं, इसलिए आपको **कर्नेल डेवलपमेंट किट** डाउनलोड करने की आवश्यकता है ताकि आप **KEXT** प्राप्त कर सकें। **`kConstraintCategory`** से शुरू होने वाले प्रतीकचिह्न **रोचक** हैं। उन्हें निकालकर आपको एक DER (ASN.1) एन्कोडेड स्ट्रीम मिलेगा जिसे आपको [ASN.1 डिकोडर](https://holtstrom.com/michael/tools/asn1decoder.php) या पायथन-एएसएन.1 पुस्तकालय और इसके `dump.py` स्क्रिप्ट, [andrivet/python-asn1](https://github.com/andrivet/python-asn1/tree/master) के साथ डिकोड करने की आवश्यकता होगी जो आपको एक अधिक समझने योग्य स्ट्रिंग देगा।

## पर्यावरण प्रतिबंध

ये तीसरे पक्ष के एप्लिकेशन में विन्यासित लॉन्च प्रतिबंध हैं। डेवलपर अपने एप्लिकेशन में पहुंच को सीमित करने के लिए **तथ्य** और **
```bash
codesign -d -vvvv app.app
```
## विश्वास कैश

**macOS** में कुछ विश्वास कैश होती हैं:

- **`/System/Volumes/Preboot/*/boot/*/usr/standalone/firmware/FUD/BaseSystemTrustCache.img4`**
- **`/System/Volumes/Preboot/*/boot/*/usr/standalone/firmware/FUD/StaticTrustCache.img4`**
- **`/System/Library/Security/OSLaunchPolicyData`**

और iOS में ऐसा लगता है कि यह **`/usr/standalone/firmware/FUD/StaticTrustCache.img4`** में है।

{% hint style="warning" %}
Apple Silicon उपकरणों पर चल रहे macOS में, यदि किसी Apple द्वारा हस्ताक्षरित बाइनरी विश्वास कैश में नहीं है, तो AMFI इसे लोड करने से इनकार करेगा।
{% endhint %}

### विश्वास कैशों की गणना

पिछले विश्वास कैश फ़ाइलें **IMG4** और **IM4P** प्रारूप में होती हैं, जिसमें IM4P एक IMG4 प्रारूप का पेलोड खंड होता है।

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

अब आप उपकरण [**trustcache**](https://github.com/CRKatri/trustcache) का उपयोग करके सूचना को एक पठनीय स्वरूप में प्राप्त कर सकते हैं:
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
Then, you could use a script such as [**यह एक**](https://gist.github.com/xpn/66dc3597acd48a4c31f5f77c3cc62f30) to extract data.

From that data you can check the Apps with a **launch constraints value of `0`** , which are the ones that aren't constrained ([**यहाँ जांचें**](https://gist.github.com/LinusHenze/4cd5d7ef057a144cda7234e2c247c056) for what each value is).

## हमले को रोकथाम

लॉन्च कंस्ट्रेंट्स ने कई पुराने हमलों को रोक दिया होता है **यह सुनिश्चित करके कि प्रक्रिया अप्रत्याशित स्थितियों में नहीं चलाई जाएगी:** उदाहरण के लिए अप्रत्याशित स्थानों से या अप्रत्याशित माता प्रक्रिया द्वारा आमंत्रित किया जाने पर (अगर केवल launchd को इसे लॉन्च करना चाहिए)

इसके अतिरिक्त, लॉन्च कंस्ट्रेंट्स ने भी **डाउनग्रेड हमलों को रोका।**

हालांकि, ये **सामान्य XPC** दुरुपयोग, **इलेक्ट्रॉन** कोड इंजेक्शन या पुस्तकालय सत्यापन के बिना **डायलिब इंजेक्शन** को नहीं रोकते (जब तक पुस्तकालयों को लोड करने वाले टीम आईडी पता हो)

### XPC डेमन सुरक्षा

सोनोमा रिलीज में, डेमन XPC सेवा की **जिम्मेदारी कॉन्फ़िगरेशन** एक महत्वपूर्ण बिंदु है। XPC सेवा अपने आप के लिए ज़िम्मेदार है, जिसके विपरीत जुड़ने वाले क्लाइंट जिम्मेदार होते हैं। यह फीडबैक रिपोर्ट FB13206884 में दर्ज है। यह सेटअप दोषपूर्ण लग सकता है, क्योंकि यह XPC सेवा के साथ कुछ इंटरैक्शन की अनुमति देता है:

- **XPC सेवा को लॉन्च करना**: यदि यह एक बग माना जाता है, तो यह सेटअप अटैकर को XPC सेवा को आरंभ करने की अनुमति नहीं देता।
- **सक्रिय सेवा से कनेक्ट करना**: यदि XPC सेवा पहले से चल रही है (संभावना अपने मूल एप्लिकेशन द्वारा सक्रिय की गई हो), तो इसे कनेक्ट करने के लिए कोई बाधा नहीं है।

XPC सेवा पर प्रतिबंध लगाना **संभावित हमलों के लिए खिड़की को संक्षेपित करने** के द्वारा फायदेमंद हो सकता है, लेकिन यह मुख्य चिंता का समाधान नहीं करता है। XPC सेवा की सुरक्षा सुनिश्चित करने के लिए मुख्य रूप से **कनेक्टिंग क्लाइंट को प्रभावी ढंग से सत्यापित करना** आवश्यक है। यह सेवा की सुरक्षा को मजबूत करने का एकमात्र तरीका रहता है। इसके अलावा, यह उल्लिखित जिम्मेदारी कॉन्फ़िगरेशन वर्तमान में संचालित है, जो निर्धारित डिज़ाइन के साथ मेल नहीं खा सकता है।


### इलेक्ट्रॉन सुरक्षा

यदि ऐप्लिकेशन को **लॉन्चसर्विस द्वारा खोलना आवश्यक है** (माता की प्रतिबंधों में)। यह **`open`** (जो एनवी वेरिएबल सेट कर सकता है) या **लॉन्च सेवाएं एपीआई** (जिसमें एनवी वेरिएबल निर्दिष्ट किए जा सकते हैं) का उपयोग करके प्राप्त किया जा सकता है।

## संदर्भ

* [https://youtu.be/f1HA5QhLQ7Y?t=24146](https://youtu.be/f1HA5QhLQ7Y?t=24146)
* [https://theevilbit.github.io/posts/launch\_constraints\_deep\_dive/](https://theevilbit.github.io/posts/launch\_constraints\_deep\_dive/)
* [https://eclecticlight.co/2023/06/13/why-wont-a-system-app-or-command-tool-run-launch-constraints-and-trust-caches/](https://eclecticlight.co/2023/06/13/why-wont-a-system-app-or-command-tool-run-launch-constraints-and-trust-caches/)
* [https://developer.apple.com/videos/play/wwdc2023/10266/](https://developer.apple.com/videos/play/wwdc2023/10266/)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप चाहते हैं कि आपकी **कंपनी को हैकट्रिक्स में विज्ञापित** किया जाए? या क्या आप **PEASS के नवीनतम संस्करण को देखना चाहते हैं या हैकट्रिक्स को पीडीएफ में डाउनलोड करना चाहते हैं**? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**द पीएएस फैमिली**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**एनएफटीएस**](https://opensea.io/collection/the-peass-family) संग्रह
* प्राप्त करें [**आधिकारिक पीएएस और हैकट्रिक्स स्वैग**](https://peass.creator-spring.com)
* **जुड़ें** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें, हैकट्रिक्स रेपो** में पीआर जमा करके [**हैकट्रिक्स रेपो**](https://github.com/carlospolop/hacktricks) **और** [**हैकट्रिक्स-क्लाउड रेपो**](https://github.com/carlospolop/hacktricks-cloud)
*
* .

</details>
