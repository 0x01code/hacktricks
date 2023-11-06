# macOS Gatekeeper / Quarantine / XProtect

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप एक **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने की अनुमति** चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का** **अनुसरण** करें।**
* **अपने हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud)
*
* .

</details>

## Gatekeeper

**Gatekeeper** एक सुरक्षा सुविधा है जो मैक ऑपरेटिंग सिस्टम के लिए विकसित की गई है, जिसका उद्देश्य है सुनिश्चित करना कि उपयोगकर्ता उनके सिस्टम पर केवल **विश्वसनीय सॉफ़्टवेयर चलाएं**। यह कार्य करता है **सॉफ़्टवेयर की प्रमाणित** करके जो उपयोगकर्ता डाउनलोड करता है और खोलने की कोशिश करता है **ऐप स्टोर के बाहर स्रोतों** से, जैसे एक ऐप, एक प्लगइन या एक स्थापना पैकेज।

Gatekeeper की मुख्य यांत्रिकी उसकी **सत्यापन** प्रक्रिया में होती है। यह जांचता है कि डाउनलोड की गई सॉफ़्टवेयर **पहचाने गए डेवलपर द्वारा साइन की गई है**, सुनिश्चित करते हुए कि सॉफ़्टवेयर की प्रामाणिकता है। इसके अलावा, यह यह सुनिश्चित करता है कि सॉफ़्टवेयर **Apple द्वारा नोटराइज़ नहीं की गई है**, जिससे पता चलता है कि इसमें ज्ञात खतरनाक सामग्री नहीं है और नोटराइज़ के बाद संशोधित नहीं किया गया है।

इसके अतिरिक्त, Gatekeeper उपयोगकर्ता नियंत्रण और सुरक्षा को मजबूत करता है द्वारा **उपयोगकर्ताओं को मंजूरी देने के लिए प्रोत्साहित करने** के द्वारा डाउनलोड की गई सॉफ़्टवेयर को पहली बार खोलने के लिए। यह सुरक्षा उपाय उपयोगकर्ताओं को अनजाने में हानिकारक एक्जीक्यूटेबल कोड को चलाने से रोकने में मदद करती है, जिसे वे एक अहानिकारक डेटा फ़ाइल के रूप में गलती से समझ सकते हैं।

### एप्लिकेशन हस्ताक्षर

एप्लिकेशन हस्ताक्षर, जिसे कोड हस्ताक्षर भी कहा जाता है, Apple की सुरक्षा बुनियाद का एक महत्वपूर्ण घटक है। इसका उपयोग सॉफ़्टवेयर लेखक (डेवलपर) की पहचान **सत्यापित करने** और सुनिश्चित करने के लिए किया जाता है कि कोड को अंतिम बार हस्ताक्षर किए जाने के बाद संशोधित नहीं किया गया है।

यहां इसका काम कैसे करता है:

1. **एप्लिकेशन को हस्ताक्षर करना:** जब एक डेवलपर अपना एप्लिकेशन वितरित करने के लिए तैयार होता है, वह एक निजी कुंजी का उपयोग करके एप्लिकेशन को **हस्ताक्षर करता है**। यह निजी कुंजी उस प्रमाण पत्र से संबंधित होती है जिसे Apple डेवलपर प्रोग्राम में डेवलपर को जारी करता है। हस्ताक्षर प्रक्रिया में, एप्लिकेशन के सभी हिस्सों का एक क्रिप्टोग्राफिक हैश बनाया जाता है और इस हैश को डेवलपर की निजी कुंजी के साथ एन्क्रिप्ट किया जाता है।
2. **एप्लिकेशन का वितरण:** हस्ताक्षरित एप्लिकेशन तब उपयोगकर्ताओं के साथ वितरित किया जाता है जो डेवलपर के प्रमाण पत्र के साथ आता है, जिसमें संबंधित सार्वजनिक कुंजी होती है।
3. **एप्लिकेशन की सत्यापना:** जब एक उपयोगकर
```bash
# Get signer
codesign -vv -d /bin/ls 2>&1 | grep -E "Authority|TeamIdentifier"

# Check if the app’s contents have been modified
codesign --verify --verbose /Applications/Safari.app

# Get entitlements from the binary
codesign -d --entitlements :- /System/Applications/Automator.app # Check the TCC perms

# Check if the signature is valid
spctl --assess --verbose /Applications/Safari.app

# Sign a binary
codesign -s <cert-name-keychain> toolsdemo
```
### नोटराइज़ेशन

एप्पल की नोटराइज़ेशन प्रक्रिया उपयोगकर्ताओं को संभावित नुकसानदायक सॉफ़्टवेयर से सुरक्षित रखने के लिए एक अतिरिक्त सुरक्षा प्रणाली के रूप में काम करती है। इसमें, **डेवलपर अपने एप्लिकेशन को एप्पल की नोटरी सेवा के द्वारा जांच के लिए सबमिट करता है**, जिसे एप रिव्यू से गलती से न मिला जाए। यह सेवा एक **स्वचालित प्रणाली** है जो सबमिट किए गए सॉफ़्टवेयर को **खतरनाक सामग्री** और कोड-साइनिंग के संबंध में किसी भी संभावित समस्या की जांच करती है।

यदि सॉफ़्टवेयर इस जांच को **सफलतापूर्वक पारित** करता है और कोई चिंता नहीं उठाता है, तो नोटरी सेवा एक नोटराइज़ेशन टिकट उत्पन्न करती है। डेवलपर को फिर से अपने सॉफ़्टवेयर के साथ इस टिकट को **जोड़ना आवश्यक होता है**, जिसे 'स्टैपलिंग' कहा जाता है। इसके अलावा, नोटराइज़ेशन टिकट भी ऑनलाइन प्रकाशित किया जाता है जहां गेटकीपर, एप्पल की सुरक्षा प्रौद्योगिकी, इसे एक्सेस कर सकता है।

उपयोगकर्ता के पहले सॉफ़्टवेयर के स्थापना या निष्पादन पर, नोटराइज़ेशन टिकट की मौजूदगी - चाहे एक्जीक्यूटेबल के साथ स्टैपल की गई हो या ऑनलाइन मिली हो - गेटकीपर को सूचित करती है कि सॉफ़्टवेयर को एप्पल द्वारा नोटराइज़ किया गया है। इस परिणामस्वरूप, गेटकीपर प्रारंभिक लॉन्च डायलॉग में एक वर्णनात्मक संदेश प्रदर्शित करता है, जिसमें बताया जाता है कि सॉफ़्टवेयर को एप्पल द्वारा खतरनाक सामग्री की जांच की गई है। इस प्रक्रिया से उपयोगकर्ता अपने सिस्टम पर स्थापित या चलाए जाने वाले सॉफ़्टवेयर की सुरक्षा में अधिक विश्वास रखते हैं।

### गेटकीपर की गणना करना

गेटकीपर एक सुरक्षा घटक है जो अविश्वसनीय ऐप्स को निष्पादित होने से रोकता है और इसके साथ ही यह कई सुरक्षा घटकों को भी है।

गेटकीपर की **स्थिति** को देखने के लिए निम्न का उपयोग किया जा सकता है:
```bash
# Check the status
spctl --status
```
{% hint style="danger" %}
ध्यान दें कि GateKeeper हस्ताक्षर जांच केवल **क्वारंटीन विशेषता वाले फ़ाइलों** के लिए होती है, हर फ़ाइल के लिए नहीं।
{% endhint %}

GateKeeper यह जांचेगा कि क्या एक बाइनरी को **प्राथमिकताएं और हस्ताक्षर** के अनुसार चलाया जा सकता है:

<figure><img src="../../../.gitbook/assets/image (678).png" alt=""><figcaption></figcaption></figure>

इस समाकृति को रखने वाला डेटाबेस **`/var/db/SystemPolicy`** में स्थित है। आप इस डेटाबेस की जांच रूट के रूप में कर सकते हैं:
```bash
# Open database
sqlite3 /var/db/SystemPolicy

# Get allowed rules
SELECT requirement,allow,disabled,label from authority where label != 'GKE' and disabled=0;
requirement|allow|disabled|label
anchor apple generic and certificate 1[subject.CN] = "Apple Software Update Certification Authority"|1|0|Apple Installer
anchor apple|1|0|Apple System
anchor apple generic and certificate leaf[field.1.2.840.113635.100.6.1.9] exists|1|0|Mac App Store
anchor apple generic and certificate 1[field.1.2.840.113635.100.6.2.6] exists and (certificate leaf[field.1.2.840.113635.100.6.1.14] or certificate leaf[field.1.2.840.113635.100.6.1.13]) and notarized|1|0|Notarized Developer ID
[...]
```
नोट करें कि पहला नियम "**ऐप स्टोर**" में समाप्त हुआ था और दूसरा नियम "**डेवलपर आईडी**" में समाप्त हुआ था और पिछले इमेज में यह **ऐप स्टोर और पहचाने गए डेवलपरों से ऐप्स को चलाने के लिए सक्षम किया गया था**।
यदि आप उस सेटिंग को ऐप स्टोर पर संशोधित करते हैं, तो "**नोटराइज्ड डेवलपर आईडी" नियम गायब हो जाएंगे**।

यहां **GKE** प्रकार के हजारों नियम भी हैं:
```bash
SELECT requirement,allow,disabled,label from authority where label = 'GKE' limit 5;
cdhash H"b40281d347dc574ae0850682f0fd1173aa2d0a39"|1|0|GKE
cdhash H"5fd63f5342ac0c7c0774ebcbecaf8787367c480f"|1|0|GKE
cdhash H"4317047eefac8125ce4d44cab0eb7b1dff29d19a"|1|0|GKE
cdhash H"0a71962e7a32f0c2b41ddb1fb8403f3420e1d861"|1|0|GKE
cdhash H"8d0d90ff23c3071211646c4c9c607cdb601cb18f"|1|0|GKE
```
ये हैश **`/var/db/SystemPolicyConfiguration/gke.bundle/Contents/Resources/gke.auth`, `/var/db/gke.bundle/Contents/Resources/gk.db`** और **`/var/db/gkopaque.bundle/Contents/Resources/gkopaque.db`** से आते हैं।

**`spctl`** के विकल्प **`--master-disable`** और **`--global-disable`** इन हस्ताक्षर जांचों को पूरी तरह से **अक्षम** कर देंगे:
```bash
# Disable GateKeeper
spctl --global-disable
spctl --master-disable

# Enable it
spctl --global-enable
spctl --master-enable
```
पूरी तरह से सक्षम करने पर, एक नया विकल्प दिखाई देगा:

<figure><img src="../../../.gitbook/assets/image (679).png" alt=""><figcaption></figcaption></figure>

यह **जांचना संभव है कि क्या गेटकीपर द्वारा एक ऐप को अनुमति दी जाएगी** के साथ:
```bash
spctl --assess -v /Applications/App.app
```
गेटकीपर में नए नियम जोड़ना संभव है ताकि कुछ ऐप्स को निष्पादित किया जा सके:
```bash
# Check if allowed - nop
spctl --assess -v /Applications/App.app
/Applications/App.app: rejected
source=no usable signature

# Add a label and allow this label in GateKeeper
sudo spctl --add --label "whitelist" /Applications/App.app
sudo spctl --enable --label "whitelist"

# Check again - yep
spctl --assess -v /Applications/App.app
/Applications/App.app: accepted
```
### क्वारंटाइन फ़ाइलें

एक एप्लिकेशन या फ़ाइल को डाउनलोड करने के बाद, macOS के विशेष एप्लिकेशन जैसे वेब ब्राउज़र या ईमेल क्लाइंट डाउनलोड की गई फ़ाइल के लिए एक विस्तारित फ़ाइल गुणांक जो कि "क्वारंटाइन झंडा" के रूप में जाना जाता है, जोड़ते हैं। यह गुणांक फ़ाइल को एक असुरक्षित स्रोत (इंटरनेट) से आने वाला और संभावित जोखिम ले जाने वाला चिह्नित करने के रूप में सुरक्षा उपाय के रूप में कार्य करता है। हालांकि, सभी एप्लिकेशन इस गुणांक को जोड़ते नहीं हैं, उदाहरण के लिए, सामान्य बिटटोरेंट क्लाइंट सॉफ़्टवेयर आमतौर पर इस प्रक्रिया को छोड़ देता है।

**क्वारंटाइन झंडा मौजूद होने पर, जब उपयोगकर्ता फ़ाइल को निष्पादित करने का प्रयास करता है, macOS के गेटकीपर सुरक्षा सुविधा को संकेत मिलता है।**

जब **क्वारंटाइन झंडा मौजूद नहीं होता है** (बिटटोरेंट क्लाइंट के माध्यम से डाउनलोड की गई फ़ाइलों के साथ), गेटकीपर की **जांचें नहीं की जा सकती हैं**। इसलिए, उपयोगकर्ताओं को असुरक्षित या अज्ञात स्रोतों से डाउनलोड की गई फ़ाइलें खोलने पर सतर्क रहना चाहिए।

{% hint style="info" %}
कोड हस्ताक्षरों की मान्यता की **जांच** एक **संसाधन-प्रवर्धनात्मक** प्रक्रिया है जिसमें कोड और उसके सभी बंडल रिसोर्सेज़ के क्रिप्टोग्राफ़िक हैश उत्पन्न करने शामिल होते हैं। इसके अलावा, प्रमाणपत्र की मान्यता की जांच में शामिल होता है कि इसे जारी किए जाने के बाद रद्द किया गया है या नहीं, जिसके लिए ऑनलाइन जांच की जाती है। इन कारणों से, पूर्ण कोड हस्ताक्षर और नोटराइज़ेशन जांच को **हर बार जब एक ऐप लॉन्च होता है, चलाना असंभव हो जाता है**।

इसलिए, ये जांचें **केवल क्वारंटाइन गुणांक वाले ऐप्स को निष्पादित करते समय ही चलाई जाती हैं**।
{% endhint %}

{% hint style="warning" %}
यह गुणांक फ़ाइल को बनाने / डाउनलोड करने वाले एप्लिकेशन द्वारा **सेट किया जाना चाहिए**।

हालांकि, सैंडबॉक्स किए गए फ़ाइलों के लिए इस गुणांक को सेट किया जाएगा। और सैंडबॉक्स नहीं किए गए ऐप्स इसे खुद ही सेट कर सकते हैं, या **Info.plist** में [**LSFileQuarantineEnabled**](https://developer.apple.com/documentation/bundleresources/information\_property\_list/lsfilequarantineenabled?language=objc) कुंजी निर्दिष्ट कर सकते हैं जिससे सिस्टम निर्मित फ़ाइलों पर `com.apple.quarantine` विस्तारित गुणांक सेट करता है,
{% endhint %}

इसकी स्थिति की **जांच और सक्षम / अक्षम** (रूट की आवश्यकता होती है) करना संभव है:
```bash
spctl --status
assessments enabled

spctl --enable
spctl --disable
#You can also allow nee identifies to execute code using the binary "spctl"
```
आप निम्नलिखित के साथ भी **जांच सकते हैं कि क्या एक फ़ाइल में quarantine विस्तारित गुणधर्म है**:
```bash
xattr file.png
com.apple.macl
com.apple.quarantine
```
विस्तारित गुणों की मान की जांच करें और इसके साथ उस ऐप का पता लगाएं जिसने क्वारंटीन गुण लिखा है:
```bash
xattr -l portada.png
com.apple.macl:
00000000  03 00 53 DA 55 1B AE 4C 4E 88 9D CA B7 5C 50 F3  |..S.U..LN.....P.|
00000010  16 94 03 00 27 63 64 97 98 FB 4F 02 84 F3 D0 DB  |....'cd...O.....|
00000020  89 53 C3 FC 03 00 27 63 64 97 98 FB 4F 02 84 F3  |.S....'cd...O...|
00000030  D0 DB 89 53 C3 FC 00 00 00 00 00 00 00 00 00 00  |...S............|
00000040  00 00 00 00 00 00 00 00                          |........|
00000048
com.apple.quarantine: 00C1;607842eb;Brave;F643CD5F-6071-46AB-83AB-390BA944DEC5
# 00c1 -- It has been allowed to eexcute this file
# 607842eb -- Timestamp
# Brave -- App
# F643CD5F-6071-46AB-83AB-390BA944DEC5 -- UID assigned to the file downloaded
```
और उस गुण को इस प्रकार से **हटाएं**:
```bash
xattr -d com.apple.quarantine portada.png
#You can also remove this attribute from every file with
find . -iname '*' -print0 | xargs -0 xattr -d com.apple.quarantine
```
और निम्नलिखित के साथ सभी क्वारंटाइन फ़ाइलें ढूंढ़ें:

{% code overflow="wrap" %}
```bash
find / -exec ls -ld {} \; 2>/dev/null | grep -E "[x\-]@ " | awk '{printf $9; printf "\n"}' | xargs -I {} xattr -lv {} | grep "com.apple.quarantine"
```
{% endcode %}

क्वारंटाइन जानकारी भी एक केंद्रीय डेटाबेस में संग्रहीत होती है जिसे लॉन्चसर्विसेज़ द्वारा प्रबंधित किया जाता है **`~/Library/Preferences/com.apple.LaunchServices.QuarantineEventsV2`**.

### XProtect

XProtect मैकओएस में एक अंतरविषय-मैलवेयर सुरक्षा सुविधा है। XProtect **जाने माने मैलवेयर और असुरक्षित फ़ाइल प्रकारों के ख़िलाफ़ जब भी कोई एप्लिकेशन पहली बार चालू की जाती है या संशोधित की जाती है, तो उसे अपने डेटाबेस के ख़िलाफ़ जांचता है**। जब आप कुछ ऐप्स के माध्यम से फ़ाइल डाउनलोड करते हैं, जैसे Safari, Mail या Messages, XProtect फ़ाइल की स्वचालित जांच करता है। यदि इसके डेटाबेस में किसी जाने माने मैलवेयर के साथ मेल ख़ाता है, तो XProtect फ़ाइल को **चलाने से रोकेगा** और आपको ख़तरे के बारे में सूचित करेगा।

XProtect डेटाबेस को Apple द्वारा नई मैलवेयर परिभाषाओं के साथ **नियमित रूप से अपडेट किया जाता है**, और ये अपडेट आपके मैक पर स्वचालित रूप से डाउनलोड और स्थापित होते हैं। इससे यह सुनिश्चित होता है कि XProtect हमेशा नवीनतम ज्ञात ख़तरों के साथ अद्यतित रहता है।

हालांकि, यह ध्यान देने योग्य है कि **XProtect एक पूर्ण सुविधा वाला एंटीवायरस समाधान नहीं है**। यह केवल एक निश्चित सूची के ज्ञात ख़तरों की जांच करता है और अधिकांश एंटीवायरस सॉफ़्टवेयर की तरह ऑन-एक्सेस स्कैनिंग नहीं करता है।

आप नवीनतम XProtect अपडेट के बारे में जानकारी प्राप्त कर सकते हैं:

{% code overflow="wrap" %}
```bash
system_profiler SPInstallHistoryDataType 2>/dev/null | grep -A 4 "XProtectPlistConfigData" | tail -n 5
```
{% endcode %}

XProtect स्थित है। SIP सुरक्षित स्थान पर **/Library/Apple/System/Library/CoreServices/XProtect.bundle** और बंडल के अंदर आप XProtect का उपयोग करने के लिए जानकारी पा सकते हैं:

* **`XProtect.bundle/Contents/Resources/LegacyEntitlementAllowlist.plist`**: इन cdhashes के साथ कोड को पुरानी अधिकारिताओं का उपयोग करने देता है।
* **`XProtect.bundle/Contents/Resources/XProtect.meta.plist`**: BundleID और TeamID के माध्यम से लोड करने के लिए अस्वीकृत किए जाने वाले प्लगइन और एक्सटेंशन की सूची या न्यूनतम संस्करण की संकेत करता है।
* **`XProtect.bundle/Contents/Resources/XProtect.yara`**: मैलवेयर का पता लगाने के लिए Yara नियम।
* **`XProtect.bundle/Contents/Resources/gk.db`**: अवरुद्ध अनुप्रयोगों और TeamID के हैश के साथ SQLite3 डेटाबेस।

ध्यान दें कि XProtect से संबंधित एक और ऐप है **`/Library/Apple/System/Library/CoreServices/XProtect.app`** जो Gatekeeper प्रक्रिया के साथ संलग्न नहीं है।

## Gatekeeper Bypasses

Gatekeeper को बाइपास करने का कोई भी तरीका (उपयोगकर्ता को कुछ डाउनलोड करने और गेटकीपर द्वारा इसे अनुमति नहीं देने पर इसे क्रियान्वित करने में सफल होना) macOS में एक सुरक्षा दुर्बलता के रूप में मानी जाती है। निम्नलिखित कुछ CVEs तक द्वारा बाइपास करने की तकनीकों को अनुमति दी गई थी:

### [CVE-2021-1810](https://labs.withsecure.com/publications/the-discovery-of-cve-2021-1810)

**Archive Utility** द्वारा निकाले जाने पर, **886** वर्णों से लंबे फ़ाइल **प्राप्त करने में विफल रहेंगी** जो com.apple.quarantine विस्तारित गुणधर्म को अनुग्रहित नहीं करेगी, जिससे इन फ़ाइलों को **गेटकीपर को बाइपास करने के लिए** संभव होगा।

अधिक जानकारी के लिए [**मूल रिपोर्ट**](https://labs.withsecure.com/publications/the-discovery-of-cve-2021-1810) की जांच करें।

### [CVE-2021-30990](https://ronmasas.com/posts/bypass-macos-gatekeeper)

जब एक एप्लिकेशन **Automator** के साथ बनाई जाती है, इसके क्रियान्वयन के बारे में जानकारी `application.app/Contents/document.wflow` में होती है न कि क्रियान्वयक। क्रियान्वयक केवल एक साधारण Automator बाइनरी है जिसे **Automator Application Stub** कहा जाता है।

इसलिए, आप `application.app/Contents/MacOS/Automator\ Application\ Stub` को **एक प्रतीकात्मक लिंक के साथ दूसरे सिस्टम के Automator Application Stub की ओर पहुंचा सकते हैं** और यह `document.wflow` (आपका स्क्रिप्ट) के अंदर है क्रियान्वित करेगा **गेटकीपर को ट्रिगर नहीं करता** क्योंकि वास्तविक क्रियान्वयक के पास अवरोध एक्सट्रिब्यूट नहीं है।

उम्मीदित स्थान का उदाहरण: `/System/Library/CoreServices/Automator\ Application\ Stub.app/Contents/MacOS/Automator\ Application\ Stub`

अधिक जानकारी के लिए [**मूल रिपोर्ट**](https://ronmasas.com/posts/bypass-macos-gatekeeper) की जांच करें।

### [CVE-2022-22616](https://www.jamf.com/blog/jamf-threat-labs-safari-vuln-gatekeeper-bypass/)

इस बाइपास में एक ज़िप फ़ाइल एक ऐप्लिकेशन के साथ बनाई गई थी जो `application.app/Contents` से शुरू होकर संपीड़ित होने लगी थी बजाय `application.app` से। इसलिए, **अवरोध गुणधर्म** `application.app/Contents` के सभी **फ़ाइलों पर लागू किया गया था** लेकिन **`application.app` पर नहीं**, जिसे गेटकीपर जांच रहा था, इसलिए गेटकीपर को बाइपास किया गया क्योंकि `application.app` ट्रिगर होने पर उसमें **अवरोध गुणधर्म नहीं था**।

अधिक जानकारी के लिए [**मूल रिपोर्ट**](https://www.jamf.com/blog/jamf-threat-labs-safari-vuln-gatekeeper-bypass/) की जांच करें।
```bash
zip -r test.app/Contents test.zip
```
अधिक जानकारी के लिए [**मूल रिपोर्ट**](https://www.jamf.com/blog/jamf-threat-labs-safari-vuln-gatekeeper-bypass/) देखें।

### [CVE-2022-32910](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2022-32910)

हालांकि तत्व अलग हों, इस कमजोरी का शोधार्थीकरण पिछले वाले के बहुत समान है। इस मामले में हम **`application.app/Contents`** से एक Apple Archive उत्पन्न करेंगे, ताकि **`application.app` को quarantine एट्रिब्यूट न मिले** जब **Archive Utility** द्वारा अनबंधित किया जाए।
```bash
aa archive -d test.app/Contents -o test.app.aar
```
अधिक जानकारी के लिए [**मूल रिपोर्ट**](https://www.jamf.com/blog/jamf-threat-labs-macos-archive-utility-vulnerability/) की जांच करें।

### [CVE-2022-42821](https://www.microsoft.com/en-us/security/blog/2022/12/19/gatekeepers-achilles-heel-unearthing-a-macos-vulnerability/)

ACL **`writeextattr`** का उपयोग करके किसी भी व्यक्ति को एक फ़ाइल में एक गुण लिखने से रोका जा सकता है:
```bash
touch /tmp/no-attr
chmod +a "everyone deny writeextattr" /tmp/no-attr
xattr -w attrname vale /tmp/no-attr
xattr: [Errno 13] Permission denied: '/tmp/no-attr'
```
इसके अलावा, **AppleDouble** फ़ाइल प्रारूप एक फ़ाइल की ACEs के साथ एक फ़ाइल की प्रतिलिपि बनाता है।

[**स्रोत कोड**](https://opensource.apple.com/source/Libc/Libc-391/darwin/copyfile.c.auto.html) में देखा जा सकता है कि **`com.apple.acl.text`** नामक xattr में संग्रहीत ACL पाठ प्रतिष्ठान के रूप में सेट किया जाएगा। इसलिए, यदि आपने एक ऐप्लिकेशन को एक zip फ़ाइल में **AppleDouble** फ़ाइल प्रारूप के साथ संक्षिप्त किया है जिसमें एक ACL है जो इसे अन्य xattr को लिखने से रोकता है... तो quarantine xattr को ऐप्लिकेशन में सेट नहीं किया गया था:
```bash
chmod +a "everyone deny write,writeattr,writeextattr" /tmp/test
ditto -c -k test test.zip
python3 -m http.server
# Download the zip from the browser and decompress it, the file shuold be without a wuarantine xattr
```
अधिक जानकारी के लिए [**मूल रिपोर्ट**](https://www.microsoft.com/en-us/security/blog/2022/12/19/gatekeepers-achilles-heel-unearthing-a-macos-vulnerability/) देखें।

### [CVE-2023-27943](https://blog.f-secure.com/discovery-of-gatekeeper-bypass-cve-2023-27943/)

यह पाया गया कि **Google Chrome डाउनलोड की गई फ़ाइलों को quarantine विशेषता सेट नहीं कर रहा था** क्योंकि कुछ macOS आंतरिक समस्याओं के कारण।

### [CVE-2023-27951](https://redcanary.com/blog/gatekeeper-bypass-vulnerabilities/)

AppleDouble फ़ाइल प्रारूप एक अलग फ़ाइल में फ़ाइल की विशेषताओं को संग्रहीत करते हैं, जो `._` से शुरू होती है, यह मदद करता है **macOS मशीनों के बीच फ़ाइल विशेषताओं की प्रतिलिपि बनाने में**। हालांकि, ध्यान दिया गया कि AppleDouble फ़ाइल को डीकंप्रेस करने के बाद, `._` से शुरू होने वाली फ़ाइल को **quarantine विशेषता नहीं दी गई थी**।

{% code overflow="wrap" %}
```bash
mkdir test
echo a > test/a
echo b > test/b
echo ._a > test/._a
aa archive -d test/ -o test.aar

# If you downloaded the resulting test.aar and decompress it, the file test/._a won't have a quarantitne attribute
```
{% endcode %}

गेटकीपर को अनदेखा करने के लिए एक ऐसा फ़ाइल बनाने की क्षमता होने के कारण, यह **संभव था।** यह चाल थी कि एक **DMG फ़ाइल एप्लिकेशन बनाएं** जो AppleDouble नाम संविधान का उपयोग करता है (इसे `._` से शुरू करें) और इस छिपे हुए फ़ाइल के बिना किसी **दिखाई देने वाली फ़ाइल को संकेत रूप में बनाएं** जिसमें कोई क्वारंटाइन गुण नहीं है।\
जब **dmg फ़ाइल को निष्पादित किया जाता है**, क्योंकि इसमें कोई क्वारंटाइन गुण नहीं होता है, इसलिए यह **गेटकीपर को अनदेखा करेगा**।
```bash
# Create an app bundle with the backdoor an call it app.app

echo "[+] creating disk image with app"
hdiutil create -srcfolder app.app app.dmg

echo "[+] creating directory and files"
mkdir
mkdir -p s/app
cp app.dmg s/app/._app.dmg
ln -s ._app.dmg s/app/app.dmg

echo "[+] compressing files"
aa archive -d s/ -o app.aar
```
<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **हैकट्रिक्स में विज्ञापित करना** चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT संग्रह**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें,** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **में PR जमा करके।**

</details>
