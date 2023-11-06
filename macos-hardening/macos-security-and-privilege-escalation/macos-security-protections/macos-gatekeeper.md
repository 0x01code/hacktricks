# macOS Gatekeeper / Quarantine / XProtect

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप एक **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का** **अनुसरण** करें।**
* **अपने हैकिंग ट्रिक्स को** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में पीआर जमा करके अपना योगदान दें**
*
* .

</details>

## Gatekeeper

**Gatekeeper** एक सुरक्षा सुविधा है जो मैक ऑपरेटिंग सिस्टम के लिए विकसित की गई है, जो सुनिश्चित करती है कि उपयोगकर्ता उनके सिस्टम पर केवल **विश्वसनीय सॉफ़्टवेयर चलाएं**। यह उपयोगकर्ता द्वारा डाउनलोड किया गया सॉफ़्टवेयर की **मान्यता की जांच** करके काम करता है और उसे खोलने का प्रयास करता है, जो **ऐप, प्लगइन या इंस्टॉलर पैकेज** जैसे स्रोतों के बाहर से हो सकता है।

Gatekeeper की मुख्य तंत्र उसकी **सत्यापन** प्रक्रिया में होता है। यह जांचता है कि डाउनलोड किया गया सॉफ़्टवेयर **पहचाने गए डेवलपर द्वारा साइन किया गया है**, जो सॉफ़्टवेयर की प्रामाणिकता सुनिश्चित करता है। इसके अलावा, यह यह सुनिश्चित करता है कि सॉफ़्टवेयर **Apple द्वारा नोटराइज़ की गई है**, जिससे पता चलता है कि इसमें ज्ञात खतरनाक सामग्री नहीं है और नोटराइज़ के बाद संशोधित नहीं किया गया है।

इसके अलावा, Gatekeeper उपयोगकर्ता नियंत्रण और सुरक्षा को मजबूत करता है द्वारा **उपयोगकर्ताओं को मान्यता देने के लिए प्रारंभ में डाउनलोड किए गए सॉफ़्टवेयर की मंजूरी देने के लिए प्रोम्प्ट करता है**। यह सुरक्षा उपाय उपयोगकर्ताओं को अनजाने में हानिकारक एक्सीक्यूटेबल कोड को चलाने से रोकने में मदद करती है, जिसे वे एक अहानिकारक डेटा फ़ाइल के रूप में गलती से समझ सकते हैं।

### एप्लिकेशन हस्ताक्षर

एप्लिकेशन हस्ताक्षर, जिसे कोड हस्ताक्षर भी कहा जाता है, Apple की सुरक्षा बुनियाद का एक महत्वपूर्ण घटक है। इसका उपयोग सॉफ़्टवेयर लेखक (डेवलपर) की पहचान **सत्यापित करने** और सुनिश्चित करने के लिए किया जाता है कि कोड को अंतिम बार हस्ताक्षर किए जाने के बाद संशोधित नहीं किया गया है।

यहां इसका काम कैसे करता है:

1. **एप्लिकेशन को हस्ताक्षर करना:** जब एक डेवलपर अपना एप्लिकेशन वितरित करने के लिए तैयार होता है, वह एक निजी कुंजी का उपयोग करके एप्लिकेशन को **हस्ताक्षर करता है**। यह निजी कुंजी उस प्रमाणपत्र से संबंधित होती है जिसे Apple डेवलपर प्रोग्राम में डेवलपर को प्रदान करता है जब वह इसमें नामांकित होता है। हस्ताक्षर करने की प्रक्रिया में, एप्लिकेशन के सभी हिस्सों का एक क्रिप्टोग्राफिक हैश बनाया जाता है और इस हैश को डेवलपर की निजी कुंजी के साथ एन्क्रिप्ट किया जाता है।
2. **एप्लिकेशन का वितरण:** हस्ताक्षरित एप्लिकेशन तब उपयोगकर्ताओं के साथ वितरित किया जाता है जबकि डेवलपर का प्रमाणपत्र, जिसमें संबंधित सार्वजन
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

यदि सॉफ़्टवेयर इस जांच को **सफलतापूर्वक पार करता है** और कोई चिंता नहीं उठाता है, तो नोटरी सेवा एक नोटराइज़ेशन टिकट उत्पन्न करती है। इसके बाद, डेवलपर को अपने सॉफ़्टवेयर के साथ इस टिकट को **संलग्न करने की आवश्यकता होती है**, जिसे 'स्टैपलिंग' कहा जाता है। इसके अलावा, नोटराइज़ेशन टिकट भी ऑनलाइन प्रकाशित किया जाता है जहां गेटकीपर, एप्पल की सुरक्षा प्रौद्योगिकी, इसे एक्सेस कर सकता है।

उपयोगकर्ता के पहले सॉफ़्टवेयर के स्थापना या निष्पादन पर, नोटराइज़ेशन टिकट की मौजूदगी - चाहे एक्जीक्यूटेबल के साथ संलग्न की गई हो या ऑनलाइन मिली हो - गेटकीपर को सूचित करती है कि सॉफ़्टवेयर को एप्पल द्वारा नोटराइज़ किया गया है। इस परिणामस्वरूप, गेटकीपर प्रारंभिक लॉन्च डायलॉग में एक वर्णनात्मक संदेश प्रदर्शित करता है, जिसमें बताया जाता है कि सॉफ़्टवेयर को एप्पल द्वारा खतरनाक सामग्री की जांच की गई है। इस प्रक्रिया से उपयोगकर्ता अपने सिस्टम पर स्थापित या चलाए जाने वाले सॉफ़्टवेयर की सुरक्षा में विश्वास बढ़ाता है।

### गेटकीपर की गणना करना

गेटकीपर एक सुरक्षा घटक है जो अविश्वसनीय ऐप्स को निष्पादित होने से रोकते हैं और यह भी कई सुरक्षा घटकों को शामिल है।

गेटकीपर की **स्थिति** को देखना संभव है:
```bash
# Check the status
spctl --status
```
{% hint style="danger" %}
ध्यान दें कि GateKeeper हस्ताक्षर जांच केवल **क्वारंटीन विशेषता वाले फ़ाइलों** के लिए होती है, हर फ़ाइल के लिए नहीं।
{% endhint %}

GateKeeper यह जांचेगा कि क्या एक बाइनरी को preferences और हस्ताक्षर के अनुसार चलाया जा सकता है:

<figure><img src="../../../.gitbook/assets/image (678).png" alt=""><figcaption></figcaption></figure>

इस कॉन्फ़िगरेशन को रखने वाला डेटाबेस **`/var/db/SystemPolicy`** में स्थित है। आप इस डेटाबेस को रूट के रूप में चेक कर सकते हैं:
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
यदि आप उस सेटिंग को ऐप स्टोर पर बदलते हैं, तो "**नोटराइज्ड डेवलपर आईडी" नियम गायब हो जाएंगे**।

यहां **GKE** प्रकार के हजारों नियम भी हैं:
```bash
SELECT requirement,allow,disabled,label from authority where label = 'GKE' limit 5;
cdhash H"b40281d347dc574ae0850682f0fd1173aa2d0a39"|1|0|GKE
cdhash H"5fd63f5342ac0c7c0774ebcbecaf8787367c480f"|1|0|GKE
cdhash H"4317047eefac8125ce4d44cab0eb7b1dff29d19a"|1|0|GKE
cdhash H"0a71962e7a32f0c2b41ddb1fb8403f3420e1d861"|1|0|GKE
cdhash H"8d0d90ff23c3071211646c4c9c607cdb601cb18f"|1|0|GKE
```
ये हैश हैं जो **`/var/db/SystemPolicyConfiguration/gke.bundle/Contents/Resources/gke.auth`, `/var/db/gke.bundle/Contents/Resources/gk.db`** और **`/var/db/gkopaque.bundle/Contents/Resources/gkopaque.db`** से आते हैं।

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

यह **जांचना संभव है कि गेटकीपर द्वारा किसी ऐप की अनुमति होगी**:
```bash
spctl --assess -v /Applications/App.app
```
गेटकीपर में नए नियम जोड़ना संभव है ताकि कुछ ऐप्स को निष्पादित करने की अनुमति मिल सके:
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

एक एप्लिकेशन या फ़ाइल को डाउनलोड करने पर, macOS के विशेष एप्लिकेशन जैसे वेब ब्राउज़र या ईमेल क्लाइंट डाउनलोड की गई फ़ाइल के लिए एक विस्तारित फ़ाइल गुणांक, जिसे "क्वारंटाइन फ़्लैग" के रूप में जाना जाता है, जोड़ते हैं। यह गुणांक फ़ाइल को एक असुरक्षित स्रोत (इंटरनेट) से आने वाला और संभावित जोखिम ले जाने वाला चिह्नित करने के रूप में सुरक्षा उपाय के रूप में कार्य करता है। हालांकि, सभी एप्लिकेशन इस गुणांक को जोड़ते नहीं हैं, उदाहरण के लिए, सामान्य बिटटोरेंट क्लाइंट सॉफ़्टवेयर आमतौर पर इस प्रक्रिया को छोड़ देता है।

**क्वारंटाइन फ़्लैग की अनुपस्थिति** (कुछ बिटटोरेंट क्लाइंट के माध्यम से डाउनलोड की गई फ़ाइलों के साथ), जब उपयोगकर्ता फ़ाइल को निष्पादित करने का प्रयास करता है, macOS के गेटकीपर सुरक्षा सुविधा को संकेत देती है।

जहां **क्वारंटाइन फ़्लैग मौजूद नहीं है** (कुछ बिटटोरेंट क्लाइंट के माध्यम से डाउनलोड की गई फ़ाइलों के साथ), गेटकीपर की **जांचें नहीं की जा सकती हैं**। इसलिए, उपयोगकर्ताओं को असुरक्षित या अज्ञात स्रोतों से डाउनलोड की गई फ़ाइलों को खोलने पर सतर्कता बरतनी चाहिए।

{% hint style="info" %}
कोड हस्ताक्षरों की वैधता की जांच करना एक **संसाधन-उपभोगी** प्रक्रिया है जिसमें कोड और उसके सभी बंडल रिसोर्सेज़ के क्रिप्टोग्राफिक हैश उत्पन्न करने शामिल होते हैं। इसके अलावा, प्रमाणपत्र की वैधता की जांच में उसे जारी किए जाने के बाद रद्द किया गया है यह देखने के लिए ऑनलाइन जांच करने की आवश्यकता होती है। इन कारणों से, पूर्ण कोड हस्ताक्षर और नोटराइज़ेशन जांच को **हर बार जब एक ऐप लॉन्च होता है, चलाना असंभव है**।

इसलिए, ये जांचें **केवल जब क्वारंटाइन गुणांक वाले ऐप्स को निष्पादित किया जाता हैं**।
{% endhint %}

{% hint style="warning" %}
यह गुणांक फ़ाइल बनाने / डाउनलोड करने वाले एप्लिकेशन द्वारा **सेट किया जाना चाहिए**।

हालांकि, सैंडबॉक्स किए गए फ़ाइलों के लिए इस गुणांक को उनकी हर फ़ाइल पर सेट किया जाएगा। और सैंडबॉक्स वाले ऐप्स इसे खुद सेट कर सकते हैं, या **Info.plist** में [**LSFileQuarantineEnabled**](https://developer.apple.com/documentation/bundleresources/information\_property\_list/lsfilequarantineenabled?language=objc) कुंजी निर्दिष्ट कर सकते हैं जिससे सिस्टम `com.apple.quarantine` विस्तारित गुणांक को फ़ाइलों पर सेट करेगा।
{% endhint %}

इसकी स्थिति की **जांच और सक्षम / अक्षम** (रूट की आवश्यकता होती है) करना संभव है:
```bash
spctl --status
assessments enabled

spctl --enable
spctl --disable
#You can also allow nee identifies to execute code using the binary "spctl"
```
आप निम्नलिखित के साथ भी **जांच सकते हैं कि क्या एक फ़ाइल में quarantine विस्तारित गुणधर्म है** :
```bash
xattr portada.png
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
और उस गुण को इस प्रकार **हटाएं**:
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

क्वारंटाइन जानकारी भी **`~/Library/Preferences/com.apple.LaunchServices.QuarantineEventsV2`** में संचालित लॉन्च सेवाओं द्वारा प्रबंधित एक केंद्रीय डेटाबेस में संग्रहीत होती है।

### XProtect

XProtect मैकओएस में एक निर्मित **एंटी-मैलवेयर** सुविधा है। XProtect **जाने माने मैलवेयर और असुरक्षित फ़ाइल प्रकारों के ख़िलाफ़ जब भी कोई एप्लिकेशन पहली बार चालू की जाती है या संशोधित की जाती है, तो उसे अपने डेटाबेस के ख़िलाफ़ जांचता है**। जब आप कुछ ऐप्स के माध्यम से फ़ाइल डाउनलोड करते हैं, जैसे Safari, Mail या Messages, तो XProtect फ़ाइल को स्वचालित रूप से स्कैन करता है। यदि इसमें उसके डेटाबेस में कोई जाने माने मैलवेयर मिलता है, तो XProtect फ़ाइल को **चलाने से रोकेगा** और आपको ख़तरे के बारे में सूचित करेगा।

XProtect डेटाबेस को Apple द्वारा नई मैलवेयर परिभाषाओं के साथ **नियमित रूप से अपडेट** किया जाता है, और ये अपडेट आपके मैक पर स्वचालित रूप से डाउनलोड और स्थापित हो जाते हैं। इससे यह सुनिश्चित होता है कि XProtect हमेशा नवीनतम ज्ञात ख़तरों के साथ अद्यतित रहता है।

हालांकि, यह ध्यान देने योग्य है कि **XProtect एक पूर्ण विशेषताओं वाला एंटीवायरस समाधान नहीं है**। यह केवल एक निश्चित सूची के ज्ञात ख़तरों की जांच करता है और अधिकांश एंटीवायरस सॉफ़्टवेयर की तरह ऑन-एक्सेस स्कैनिंग नहीं करता है।

आप नवीनतम XProtect अपडेट के बारे में जानकारी प्राप्त कर सकते हैं:

{% code overflow="wrap" %}
```bash
system_profiler SPInstallHistoryDataType 2>/dev/null | grep -A 4 "XProtectPlistConfigData" | tail -n 5
```
{% endcode %}

XProtect **/Library/Apple/System/Library/CoreServices/XProtect.bundle** पर SIP सुरक्षित स्थान पर स्थित है और बंडल के भीतर आप XProtect का उपयोग करने वाली जानकारी पा सकते हैं:

* **`XProtect.bundle/Contents/Resources/LegacyEntitlementAllowlist.plist`**: इन cdhashes के साथ कोड को पुरानी अधिकारिता का उपयोग करने की अनुमति देता है।
* **`XProtect.bundle/Contents/Resources/XProtect.meta.plist`**: BundleID और TeamID के माध्यम से लोड करने के लिए अस्वीकृत किए जाने वाले प्लगइन और एक्सटेंशन की सूची या न्यूनतम संस्करण की संकेत करता है।
* **`XProtect.bundle/Contents/Resources/XProtect.yara`**: मैलवेयर का पता लगाने के लिए Yara नियम।
* **`XProtect.bundle/Contents/Resources/gk.db`**: अवरुद्ध अनुप्रयोगों और TeamID के हैश के साथ SQLite3 डेटाबेस।

ध्यान दें कि XProtect से संबंधित एक और ऐप है **`/Library/Apple/System/Library/CoreServices/XProtect.app`** जो Gatekeeper प्रक्रिया के साथ संलग्न नहीं है।

## Gatekeeper Bypasses

Gatekeeper को बाईपास करने का कोई भी तरीका (उपयोगकर्ता को कुछ डाउनलोड करने और गेटकीपर द्वारा इसे निषेधित करने पर इसे क्रियान्वित करने में सफल होना) macOS में एक सुरक्षा दुर्बलता के रूप में मानी जाती है। निम्नलिखित कुछ CVEs उन तकनीकों को दर्ज करते हैं जिनके माध्यम से पहले Gatekeeper को बाईपास करने की अनुमति थी:

### [CVE-2021-1810](https://labs.withsecure.com/publications/the-discovery-of-cve-2021-1810)

**Archive Utility** द्वारा निकाले जाने पर, **886** अक्षरों से लंबे फ़ाइल **परिवार्य विशेषाधिकार को अनुग्रहित नहीं कर पाती**, जिसके कारण इन फ़ाइलों के लिए **Gatekeeper को बाईपास करना संभव होता है**।

अधिक जानकारी के लिए [**मूल रिपोर्ट**](https://labs.withsecure.com/publications/the-discovery-of-cve-2021-1810) की जांच करें।

### [CVE-2021-30990](https://ronmasas.com/posts/bypass-macos-gatekeeper)

जब एक एप्लिकेशन **Automator** के साथ बनाई जाती है, इसके क्रियान्वयन के बारे में जानकारी `application.app/Contents/document.wflow` में होती है न कि एक्ज़िक्यूटेबल में। एक्ज़िक्यूटेबल केवल एक साधारण Automator बाइनरी है जिसे **Automator Application Stub** कहा जाता है।

इसलिए, आप `application.app/Contents/MacOS/Automator\ Application\ Stub` को **एक संकेतिक लिंक के साथ अन्य सिस्टम के Automator Application Stub की ओर पहुंचने के लिए पॉइंट कर सकते हैं** और यह `document.wflow` (आपका स्क्रिप्ट) के भीतर होने वाली क्रिया को **Gatekeeper को ट्रिगर न करते हुए क्रियान्वित करेगा** क्योंकि वास्तविक एक्ज़िक्यूटेबल के पास अवरोधन xattr नहीं होता है।

उम्मीदित स्थान का उदाहरण: `/System/Library/CoreServices/Automator\ Application\ Stub.app/Contents/MacOS/Automator\ Application\ Stub`

अधिक जानकारी के लिए [**मूल रिपोर्ट**](https://ronmasas.com/posts/bypass-macos-gatekeeper) की जांच करें।

### [CVE-2022-22616](https://www.jamf.com/blog/jamf-threat-labs-safari-vuln-gatekeeper-bypass/)

इस बाईपास में एक zip फ़ाइल बनाई गई थी जिसमें संकेतित रूप से `application.app/Contents` से शुरू होकर संपीड़ित करना शुरू होता है बजाय `application.app` से। इसलिए, **सभी फ़ाइलों परिवार्य विशेषाधिकार** `application.app/Contents` से लागू किए गए थे लेकिन **`application.app` पर नहीं**, जिसे Gatekeeper जांच रहा था, इसलिए Gatekeeper को बाईपास कर दिया गया क्योंकि जब `application.app` ट्रिगर हुआ तो इसमें **परिवार्य विशेषाधिकार विशेषता नहीं थी**।

अधिक जानकारी के लिए [**मूल रिपोर्ट**](https://www.jamf.com/blog/jamf-threat-labs-safari-vuln-gatekeeper-bypass/) की जांच करें।
```bash
zip -r test.app/Contents test.zip
```
अधिक जानकारी के लिए [**मूल रिपोर्ट**](https://www.jamf.com/blog/jamf-threat-labs-safari-vuln-gatekeeper-bypass/) देखें।

### [CVE-2022-32910](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2022-32910)

हालांकि तत्व अलग हों, इस कमजोरी के उपयोग का तरीका पिछले वाले के बहुत समान है। इस मामले में हम **`application.app/Contents`** से एक Apple Archive उत्पन्न करेंगे, ताकि **`application.app` को quarantine attr नहीं मिलेगा** जब **Archive Utility** द्वारा डीकंप्रेस किया जाए।
```bash
aa archive -d test.app/Contents -o test.app.aar
```
अधिक जानकारी के लिए [**मूल रिपोर्ट**](https://www.jamf.com/blog/jamf-threat-labs-macos-archive-utility-vulnerability/) देखें।

### [CVE-2022-42821](https://www.microsoft.com/en-us/security/blog/2022/12/19/gatekeepers-achilles-heel-unearthing-a-macos-vulnerability/)

ACL **`writeextattr`** का उपयोग करके किसी भी व्यक्ति को एक फ़ाइल में एक गुण लिखने से रोका जा सकता है:
```bash
touch /tmp/no-attr
chmod +a "everyone deny writeextattr" /tmp/no-attr
xattr -w attrname vale /tmp/no-attr
xattr: [Errno 13] Permission denied: '/tmp/no-attr'
```
इसके अलावा, **AppleDouble** फ़ाइल प्रारूप एक फ़ाइल की ACEs के साथ एक कॉपी बनाता है।

[**स्रोत कोड**](https://opensource.apple.com/source/Libc/Libc-391/darwin/copyfile.c.auto.html) में देखा जा सकता है कि **`com.apple.acl.text`** नामक xattr में संग्रहीत ACL पाठ प्रतिष्ठान के रूप में सेट किया जाएगा। इसलिए, यदि आपने एक ऐप्लिकेशन को एक zip फ़ाइल में **AppleDouble** फ़ाइल प्रारूप के साथ संपीड़ित किया है जिसमें एक ACL है जो अन्य xattr को इसमें लिखने से रोकता है... तो quarantine xattr ऐप्लिकेशन में सेट नहीं हुआ होगा:
```bash
chmod +a "everyone deny write,writeattr,writeextattr" /tmp/test
ditto -c -k test test.zip
python3 -m http.server
# Download the zip from the browser and decompress it, the file shuold be without a wuarantine xattr
```
अधिक जानकारी के लिए [**मूल रिपोर्ट**](https://www.microsoft.com/en-us/security/blog/2022/12/19/gatekeepers-achilles-heel-unearthing-a-macos-vulnerability/) देखें।

### [CVE-2023-27943](https://blog.f-secure.com/discovery-of-gatekeeper-bypass-cve-2023-27943/)

यह पाया गया कि **Google Chrome डाउनलोड की गई फ़ाइलों में quarantine विशेषता सेट नहीं कर रहा था** क्योंकि कुछ macOS आंतरिक समस्याओं के कारण।

### [CVE-2023-27951](https://redcanary.com/blog/gatekeeper-bypass-vulnerabilities/)

AppleDouble फ़ाइल प्रारूप एक अलग फ़ाइल में फ़ाइल की विशेषताओं को संग्रहीत करते हैं, जो `._` से शुरू होती है, यह मदद करता है **macOS मशीनों के बीच फ़ाइल विशेषताओं की प्रतिलिपि बनाने में**। हालांकि, ध्यान दिया गया कि AppleDouble फ़ाइल को डीकंप्रेस करने के बाद, `._` से शुरू होने वाली फ़ाइल को **quarantine विशेषता नहीं दी जा रही थी**।

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

गेटकीपर को उमारने के लिए एक ऐसी फ़ाइल बनाने की क्षमता होने के कारण, यह **संभव था कि गेटकीपर को दूर करें**। यह ट्रिक थी कि एपलडबल नाम संवेदनशीलता के आदेश का उपयोग करके एक DMG फ़ाइल एप्लिकेशन बनाना था (इसे `._` से शुरू करें) और इस छिपी हुई फ़ाइल के बिना संवेदनशीलता गुणधर्म वाली दिखने वाली फ़ाइल को एक सिंबॉलिक लिंक के रूप में बनाएं।\
जब **dmg फ़ाइल को निष्पादित किया जाता है**, क्योंकि इसमें संवेदनशीलता गुणधर्म नहीं होता है, इसलिए यह **गेटकीपर को दूर करेगा**।
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

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड** करने का उपयोग करना है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को।**

</details>
