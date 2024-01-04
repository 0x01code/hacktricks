# macOS Gatekeeper / Quarantine / XProtect

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **cybersecurity company** में काम करते हैं? क्या आप चाहते हैं कि आपकी **company का विज्ञापन HackTricks में दिखाई दे**? या क्या आप **PEASS के नवीनतम संस्करण तक पहुँचना चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) देखें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* [**official PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* **Join the** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord group**](https://discord.gg/hRep4RUj7f) या [**telegram group**](https://t.me/peass) या **Twitter पर** मुझे **follow** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपनी hacking tricks साझा करें PRs जमा करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud)
*
* .

</details>

## Gatekeeper

**Gatekeeper** एक सुरक्षा सुविधा है जो Mac ऑपरेटिंग सिस्टम्स के लिए विकसित की गई है, जिसका उद्देश्य यह सुनिश्चित करना है कि उपयोगकर्ता अपने सिस्टम पर केवल **विश्वसनीय सॉफ्टवेयर चलाएं**। यह **सॉफ्टवेयर का मान्यता प्राप्त करने** के द्वारा काम करता है जिसे उपयोगकर्ता डाउनलोड करता है और App Store के **बाहरी स्रोतों से** खोलने का प्रयास करता है, जैसे कि एक ऐप, प्लग-इन, या इंस्टॉलर पैकेज।

Gatekeeper की मुख्य तंत्र इसकी **सत्यापन** प्रक्रिया में निहित है। यह जांचता है कि डाउनलोड किया गया सॉफ्टवेयर **मान्यता प्राप्त डेवलपर द्वारा हस्ताक्षरित है** या नहीं, जिससे सॉफ्टवेयर की प्रामाणिकता सुनिश्चित होती है। इसके अलावा, यह यह भी सुनिश्चित करता है कि सॉफ्टवेयर **Apple द्वारा नोटराइज्ड है**, जिससे पुष्टि होती है कि इसमें ज्ञात मैलवेयर सामग्री नहीं है और नोटराइजेशन के बाद इसमें कोई छेड़छाड़ नहीं की गई है।

इसके अतिरिक्त, Gatekeeper उपयोगकर्ता नियंत्रण और सुरक्षा को मजबूत करता है **उपयोगकर्ताओं को पहली बार डाउनलोड किए गए सॉफ्टवेयर को खोलने की अनुमति देने के लिए प्रेरित करके**। यह सुरक्षा उपाय उपयोगकर्ताओं को अनजाने में संभावित रूप से हानिकारक निष्पादन योग्य कोड चलाने से रोकता है जिसे वे हानिरहित डेटा फ़ाइल के लिए गलती कर सकते हैं।

### एप्लिकेशन हस्ताक्षर

एप्लिकेशन हस्ताक्षर, जिन्हें कोड हस्ताक्षर भी कहा जाता है, Apple की सुरक्षा अवसंरचना का एक महत्वपूर्ण घटक हैं। इनका उपयोग **सॉफ्टवेयर लेखक की पहचान की पुष्टि करने** (डेवलपर) और यह सुनिश्चित करने के लिए किया जाता है कि कोड को अंतिम बार हस्ताक्षरित करने के बाद से छेड़छाड़ नहीं की गई है।

यहाँ यह कैसे काम करता है:

1. **एप्लिकेशन को हस्ताक्षरित करना:** जब एक डेवलपर अपने एप्लिकेशन को वितरित करने के लिए तैयार होता है, वे **एक निजी कुंजी का उपयोग करके एप्लिकेशन को हस्ताक्षरित करते हैं**। यह निजी कुंजी एक **प्रमाणपत्र से जुड़ी होती है जो Apple डेवलपर को जारी करता है** जब वे Apple Developer Program में नामांकन करते हैं। हस्ताक्षर प्रक्रिया में ऐप के सभी भागों का क्रिप्टोग्राफिक हैश बनाना और इस हैश को डेवलपर की निजी कुंजी से एन्क्रिप्ट करना शामिल है।
2. **एप्लिकेशन का वितरण:** हस्ताक्षरित एप्लिकेशन को फिर डेवलपर के प्रमाणपत्र के साथ उपयोगकर्ताओं को वितरित किया जाता है, जिसमें संबंधित सार्वजनिक कुंजी होती है।
3. **एप्लिकेशन की पुष्टि करना:** जब एक उपयोगकर्ता एप्लिकेशन डाउनलोड करता है और इसे चलाने का प्रयास करता है, उनका Mac ऑपरेटिंग सिस्टम डेवलपर के प्रमाणपत्र से सार्वजनिक कुंजी का उपयोग करके हैश को डिक्रिप्ट करता है। फिर यह एप्लिकेशन की वर्तमान स्थिति के आधार पर हैश की पुनः गणना करता है और इसे डिक्रिप्ट किए गए हैश के साथ तुलना करता है। अगर वे मेल खाते हैं, इसका मतलब है **एप्लिकेशन में कोई बदलाव नहीं किया गया है** डेवलपर द्वारा हस्ताक्षरित होने के बाद, और सिस्टम एप्लिकेशन को चलाने की अनुमति देता है।

एप्लिकेशन हस्ताक्षर Apple के Gatekeeper तकनीक का एक अनिवार्य हिस्सा हैं। जब एक उपयोगकर्ता इंटरनेट से डाउनलोड किए गए एप्लिकेशन को खोलने का प्रयास करता है, Gatekeeper एप्लिकेशन हस्ताक्षर की पुष्टि करता है। अगर यह Apple द्वारा जारी किए गए प्रमाणपत्र के साथ हस्ताक्षरित है और कोड में कोई छेड़छाड़ नहीं की गई है, तो Gatekeeper एप्लिकेशन को चलाने की अनुमति देता है। अन्यथा, यह एप्लिकेशन को ब्लॉक करता है और उपयोगकर्ता को सतर्क करता है।

macOS Catalina से शुरू करके, **Gatekeeper यह भी जांचता है कि एप्लिकेशन को Apple द्वारा नोटराइज्ड किया गया है** या नहीं, जिससे सुरक्षा की एक अतिरिक्त परत जोड़ी जाती है। नोटराइजेशन प्रक्रिया एप्लिकेशन को ज्ञात सुरक्षा मुद्दों और मैलवेयर कोड के लिए जांचती है, और अगर ये जांच पास हो जाती हैं, तो Apple एप्लिकेशन में एक टिकट जोड़ता है जिसे Gatekeeper सत्यापित कर सकता है।

#### हस्ताक्षरों की जांच करें

जब आप किसी **malware sample** की जांच कर रहे हों, तो आपको हमेशा **हस्ताक्षर की जांच करनी चाहिए** क्योंकि **डेवलपर** जिसने इसे हस्ताक्षरित किया हो, वह पहले से ही **malware से संबंधित** हो सकता है।
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
### नोटरीकरण

Apple की नोटरीकरण प्रक्रिया उपयोगकर्ताओं को संभावित रूप से हानिकारक सॉफ्टवेयर से बचाने के लिए एक अतिरिक्त सुरक्षा उपाय के रूप में काम करती है। इसमें **डेवलपर द्वारा उनके एप्लिकेशन की जांच के लिए जमा करना** शामिल है **Apple की नोटरी सेवा** द्वारा, जिसे App Review से भ्रमित नहीं होना चाहिए। यह सेवा एक **स्वचालित प्रणाली** है जो जमा किए गए सॉफ्टवेयर की **मैलिशस कंटेंट** की उपस्थिति और कोड-साइनिंग के साथ किसी भी संभावित समस्याओं की जांच करती है।

यदि सॉफ्टवेयर इस निरीक्षण को बिना किसी चिंता के **पास** करता है, तो नोटरी सेवा एक नोटरीकरण टिकट उत्पन्न करती है। फिर डेवलपर को इस टिकट को उनके सॉफ्टवेयर से **जोड़ना** आवश्यक होता है, जिसे 'स्टेपलिंग' कहा जाता है। इसके अलावा, नोटरीकरण टिकट को ऑनलाइन भी प्रकाशित किया जाता है जहां Gatekeeper, Apple की सुरक्षा प्रौद्योगिकी, इसे एक्सेस कर सकती है।

उपयोगकर्ता द्वारा पहली बार सॉफ्टवेयर की स्थापना या निष्पादन के समय, नोटरीकरण टिकट की उपस्थिति - चाहे वह एक्जीक्यूटेबल से स्टेपल की गई हो या ऑनलाइन पाई गई हो - **Gatekeeper को सूचित करती है कि सॉफ्टवेयर को Apple द्वारा नोटरीकरण किया गया है**। परिणामस्वरूप, Gatekeeper प्रारंभिक लॉन्च डायलॉग में एक वर्णनात्मक संदेश प्रदर्शित करता है, जो दर्शाता है कि सॉफ्टवेयर Apple द्वारा मैलिशस कंटेंट के लिए जांचा गया है। इस प्रक्रिया से उपयोगकर्ताओं का उनके सिस्टम पर स्थापित या चलाए जाने वाले सॉफ्टवेयर की सुरक्षा में विश्वास बढ़ता है।

### GateKeeper की गणना करना

GateKeeper दोनों है, **कई सुरक्षा घटक** जो अविश्वसनीय एप्स को निष्पादित होने से रोकते हैं और यह भी **घटकों में से एक है**।

GateKeeper की **स्थिति** देखना संभव है:
```bash
# Check the status
spctl --status
```
{% hint style="danger" %}
ध्यान दें कि GateKeeper हस्ताक्षर जांच केवल **Quarantine विशेषता वाली फाइलों** पर की जाती है, हर फाइल पर नहीं।
{% endhint %}

GateKeeper यह जांचेगा कि **प्राथमिकताओं और हस्ताक्षर** के अनुसार एक बाइनरी को निष्पादित किया जा सकता है:

<figure><img src="../../../.gitbook/assets/image (678).png" alt=""><figcaption></figcaption></figure>

इस कॉन्फ़िगरेशन को रखने वाला डेटाबेस **`/var/db/SystemPolicy`** में स्थित है। आप इस डेटाबेस को रूट के साथ जांच सकते हैं:
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
ध्यान दें कि पहला नियम "**App Store**" पर समाप्त हुआ और दूसरा "**Developer ID**" पर और पिछली छवि में यह **App Store और पहचाने गए डेवलपर्स से ऐप्स को निष्पादित करने के लिए सक्षम था**।
यदि आप उस सेटिंग को App Store में **संशोधित** करते हैं, तो "**Notarized Developer ID" नियम गायब हो जाएंगे**।

**type GKE** के भी हजारों नियम हैं:
```bash
SELECT requirement,allow,disabled,label from authority where label = 'GKE' limit 5;
cdhash H"b40281d347dc574ae0850682f0fd1173aa2d0a39"|1|0|GKE
cdhash H"5fd63f5342ac0c7c0774ebcbecaf8787367c480f"|1|0|GKE
cdhash H"4317047eefac8125ce4d44cab0eb7b1dff29d19a"|1|0|GKE
cdhash H"0a71962e7a32f0c2b41ddb1fb8403f3420e1d861"|1|0|GKE
cdhash H"8d0d90ff23c3071211646c4c9c607cdb601cb18f"|1|0|GKE
```
ये हैशेज **`/var/db/SystemPolicyConfiguration/gke.bundle/Contents/Resources/gke.auth`, `/var/db/gke.bundle/Contents/Resources/gk.db`** और **`/var/db/gkopaque.bundle/Contents/Resources/gkopaque.db`** से आते हैं।

या आप पिछली जानकारी को इस प्रकार सूचीबद्ध कर सकते हैं:
```bash
sudo spctl --list
```
विकल्प **`--master-disable`** और **`--global-disable`** का उपयोग **`spctl`** में इन हस्ताक्षर जांचों को पूरी तरह से **अक्षम** कर देगा:
```bash
# Disable GateKeeper
spctl --global-disable
spctl --master-disable

# Enable it
spctl --global-enable
spctl --master-enable
```
पूरी तरह से सक्षम होने पर, एक नया विकल्प दिखाई देगा:

<figure><img src="../../../.gitbook/assets/image (679).png" alt=""><figcaption></figcaption></figure>

**यह जांचना संभव है कि कोई ऐप GateKeeper द्वारा अनुमति दी जाएगी या नहीं** :
```bash
spctl --assess -v /Applications/App.app
```
GateKeeper में नए नियम जोड़कर कुछ ऐप्स के निष्पादन की अनुमति दी जा सकती है:
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
### क्वारंटाइन फाइल्स

**डाउनलोडिंग** के दौरान किसी एप्लिकेशन या फाइल को, macOS के विशिष्ट **एप्लिकेशन्स** जैसे वेब ब्राउजर्स या ईमेल क्लाइंट्स, डाउनलोड की गई फाइल पर एक विस्तारित फाइल एट्रिब्यूट **जोड़ते हैं**, जिसे आमतौर पर "**क्वारंटाइन फ्लैग**" के नाम से जाना जाता है। यह एट्रिब्यूट एक सुरक्षा उपाय के रूप में काम करता है जो फाइल को एक अविश्वसनीय स्रोत (इंटरनेट) से आने वाली और संभावित जोखिम वाली के रूप में **चिह्नित करता है**। हालांकि, सभी एप्लिकेशन्स यह एट्रिब्यूट नहीं जोड़ते हैं, उदाहरण के लिए, आम BitTorrent क्लाइंट सॉफ्टवेयर आमतौर पर इस प्रक्रिया को बायपास करते हैं।

**क्वारंटाइन फ्लैग की उपस्थिति macOS के Gatekeeper सुरक्षा फीचर को संकेत देती है जब उपयोगकर्ता फाइल को निष्पादित करने का प्रयास करता है**।

जिस स्थिति में **क्वारंटाइन फ्लैग मौजूद नहीं होता है** (जैसे कि कुछ BitTorrent क्लाइंट्स के माध्यम से डाउनलोड की गई फाइलों के साथ), Gatekeeper के **चेक्स प्रदर्शित नहीं किए जा सकते हैं**। इसलिए, उपयोगकर्ताओं को कम सुरक्षित या अज्ञात स्रोतों से डाउनलोड की गई फाइलों को खोलते समय सावधानी बरतनी चाहिए।

{% hint style="info" %}
कोड हस्ताक्षरों की **वैधता की जांच** एक **संसाधन-गहन** प्रक्रिया है जिसमें कोड और उसके सभी बंडल किए गए संसाधनों के क्रिप्टोग्राफिक **हैशेज** उत्पन्न करना शामिल है। इसके अलावा, प्रमाणपत्र की वैधता की जांच में Apple के सर्वरों पर एक **ऑनलाइन जांच** करना शामिल है ताकि यह देखा जा सके कि क्या इसे जारी किए जाने के बाद रद्द कर दिया गया है। इन कारणों से, पूर्ण कोड हस्ताक्षर और नोटराइजेशन जांच को हर बार एप्लिकेशन लॉन्च होने पर चलाना **अव्यावहारिक है**।

इसलिए, ये जांचें **केवल उन एप्लिकेशन्स को निष्पादित करते समय चलाई जाती हैं जिनमें क्वारंटाइन एट्रिब्यूट होता है।**
{% endhint %}

{% hint style="warning" %}
यह एट्रिब्यूट **फाइल बनाने/डाउनलोड करने वाले एप्लिकेशन द्वारा सेट किया जाना चाहिए**।

हालांकि, सैंडबॉक्स की गई फाइलों में यह एट्रिब्यूट हर फाइल को बनाने पर सेट किया जाएगा। और नॉन सैंडबॉक्स एप्लिकेशन्स इसे स्वयं सेट कर सकते हैं, या **Info.plist** में [**LSFileQuarantineEnabled**](https://developer.apple.com/documentation/bundleresources/information_property_list/lsfilequarantineenabled?language=objc) कुंजी को निर्दिष्ट कर सकते हैं जिससे सिस्टम बनाई गई फाइलों पर `com.apple.quarantine` विस्तारित एट्रिब्यूट सेट करेगा,
{% endhint %}

इसकी स्थिति की **जांच करने और सक्षम/अक्षम** करने के लिए संभव है (रूट आवश्यक) के साथ:
```bash
spctl --status
assessments enabled

spctl --enable
spctl --disable
#You can also allow nee identifies to execute code using the binary "spctl"
```
आप यह भी **पता लगा सकते हैं कि किसी फ़ाइल में quarantine विस्तारित विशेषता है या नहीं** इसके साथ:
```bash
xattr file.png
com.apple.macl
com.apple.quarantine
```
**मान** की जांच करें **विस्तारित** **गुणों** की और पता लगाएं कि किस ऐप ने quarantine attr लिखा है:
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
वास्तव में एक प्रक्रिया "उसके द्वारा बनाई गई फाइलों पर क्वारंटाइन फ्लैग्स सेट कर सकती है" (मैंने बनाई गई फाइल में USER\_APPROVED फ्लैग लागू करने की कोशिश की लेकिन यह इसे लागू नहीं करेगा):

<details>

<summary>सोर्स कोड लागू क्वारंटाइन फ्लैग्स</summary>
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
<details>

और उस विशेषता को **हटाएं** इसके साथ:
```bash
xattr -d com.apple.quarantine portada.png
#You can also remove this attribute from every file with
find . -iname '*' -print0 | xargs -0 xattr -d com.apple.quarantine
```
मैकोज़ गेटकीपर के साथ सभी क्वारंटीन की गई फाइलों को इस प्रकार खोजें:

{% code overflow="wrap" %}
```bash
find / -exec ls -ld {} \; 2>/dev/null | grep -E "[x\-]@ " | awk '{printf $9; printf "\n"}' | xargs -I {} xattr -lv {} | grep "com.apple.quarantine"
```
{% endcode %}

क्वारंटाइन जानकारी **`~/Library/Preferences/com.apple.LaunchServices.QuarantineEventsV2`** में स्थित एक केंद्रीय डेटाबेस में भी संग्रहीत की जाती है जिसे LaunchServices प्रबंधित करता है।

#### **Quarantine.kext**

कर्नेल एक्सटेंशन केवल **सिस्टम पर कर्नेल कैश के माध्यम से** उपलब्ध है; हालांकि, आप **https://developer.apple.com/** से **Kernel Debug Kit** डाउनलोड कर सकते हैं, जिसमें एक्सटेंशन का सिम्बोलिकेटेड संस्करण होगा।

### XProtect

XProtect macOS में एक निर्मित **एंटी-मैलवेयर** सुविधा है। XProtect **किसी भी एप्लिकेशन को जब पहली बार लॉन्च किया जाता है या संशोधित किया जाता है तो अपने डेटाबेस** के ज्ञात मैलवेयर और असुरक्षित फाइल प्रकारों के खिलाफ जांचता है। जब आप Safari, Mail, या Messages जैसे कुछ एप्स के माध्यम से फाइल डाउनलोड करते हैं, XProtect स्वचालित रूप से फाइल को स्कैन करता है। यदि यह अपने डेटाबेस में किसी भी ज्ञात मैलवेयर से मेल खाता है, तो XProtect **फाइल को चलने से रोक देगा** और आपको खतरे के बारे में सचेत करेगा।

XProtect डेटाबेस को Apple द्वारा नए मैलवेयर परिभाषाओं के साथ **नियमित रूप से अपडेट** किया जाता है, और ये अपडेट स्वचालित रूप से आपके Mac पर डाउनलोड और स्थापित किए जाते हैं। इससे सुनिश्चित होता है कि XProtect हमेशा नवीनतम ज्ञात खतरों के साथ अद्यतन रहे।

हालांकि, यह ध्यान देने योग्य है कि **XProtect एक पूर्ण-विशेषता वाला एंटीवायरस समाधान नहीं है**। यह केवल ज्ञात खतरों की एक विशिष्ट सूची के लिए जांच करता है और अधिकांश एंटीवायरस सॉफ्टवेयर की तरह ऑन-एक्सेस स्कैनिंग नहीं करता है।

आप नवीनतम XProtect अपडेट के बारे में जानकारी प्राप्त कर सकते हैं:

{% code overflow="wrap" %}
```bash
system_profiler SPInstallHistoryDataType 2>/dev/null | grep -A 4 "XProtectPlistConfigData" | tail -n 5
```
{% endcode %}

XProtect **/Library/Apple/System/Library/CoreServices/XProtect.bundle** स्थित है और इस बंडल के अंदर आप XProtect द्वारा उपयोग की जाने वाली जानकारी पा सकते हैं:

* **`XProtect.bundle/Contents/Resources/LegacyEntitlementAllowlist.plist`**: इन cdhashes के साथ कोड को पुराने entitlements का उपयोग करने की अनुमति देता है।
* **`XProtect.bundle/Contents/Resources/XProtect.meta.plist`**: प्लगइन्स और एक्सटेंशन्स की सूची जिन्हें BundleID और TeamID के माध्यम से या न्यूनतम संस्करण का संकेत देते हुए लोड करने की अनुमति नहीं है।
* **`XProtect.bundle/Contents/Resources/XProtect.yara`**: मैलवेयर का पता लगाने के लिए Yara नियम।
* **`XProtect.bundle/Contents/Resources/gk.db`**: अवरुद्ध एप्लिकेशन्स और TeamIDs के हैशेस के साथ SQLite3 डेटाबेस।

ध्यान दें कि **`/Library/Apple/System/Library/CoreServices/XProtect.app`** में एक और ऐप है जो XProtect से संबंधित है लेकिन Gatekeeper प्रक्रिया में शामिल नहीं है।

### Not Gatekeeper

{% hint style="danger" %}
ध्यान दें कि Gatekeeper **हर बार नहीं चलता** जब आप एक एप्लिकेशन को निष्पादित करते हैं, केवल _**AppleMobileFileIntegrity**_ (AMFI) ही **एक्जीक्यूटेबल कोड सिग्नेचर्स को सत्यापित करेगा** जब आप एक ऐसे एप्लिकेशन को निष्पादित करते हैं जिसे पहले ही Gatekeeper द्वारा सत्यापित किया जा चुका है।
{% endhint %}

इसलिए, पहले यह संभव था कि एक एप्लिकेशन को Gatekeeper के साथ कैश करने के लिए निष्पादित किया जाए, फिर **एप्लिकेशन की नॉन-एक्जीक्यूटेबल फाइलों को संशोधित करें** (जैसे कि Electron asar या NIB फाइलें) और यदि कोई अन्य सुरक्षा उपाय नहीं थे, तो एप्लिकेशन **निष्पादित** होता था **मैलिशस** जोड़ों के साथ।

हालांकि, अब यह संभव नहीं है क्योंकि macOS **एप्लिकेशन बंडलों के अंदर फाइलों को संशोधित करने से रोकता है**। इसलिए, यदि आप [Dirty NIB](../macos-proces-abuse/macos-dirty-nib.md) हमले की कोशिश करते हैं, तो आप पाएंगे कि इसका दुरुपयोग करना अब संभव नहीं है क्योंकि Gatekeeper के साथ एप्लिकेशन को कैश करने के बाद, आप बंडल को संशोधित नहीं कर पाएंगे। और यदि आप उदाहरण के लिए Contents निर्देशिका का नाम NotCon में बदलते हैं (जैसा कि एक्सप्लॉइट में इंगित किया गया है), और फिर एप्लिकेशन के मुख्य बाइनरी को Gatekeeper के साथ कैश करने के लिए निष्पादित करते हैं, तो यह एक त्रुटि को ट्रिगर करेगा और निष्पादित नहीं होगा।

## Gatekeeper Bypasses

Gatekeeper को बायपास करने का कोई भी तरीका (उपयोगकर्ता को कुछ डाउनलोड करने और निष्पादित करने के लिए प्रबंधित करना जब Gatekeeper को इसे अस्वीकार करना चाहिए) macOS में एक सुरक्षा दोष माना जाता है। ये कुछ CVE हैं जो अतीत में Gatekeeper को बायपास करने की तकनीकों को सौंपे गए थे:

### [CVE-2021-1810](https://labs.withsecure.com/publications/the-discovery-of-cve-2021-1810)

**Archive Utility** द्वारा निकाले गए फाइलों के **पथ जो 886** वर्णों से अधिक लंबे होते हैं, वे com.apple.quarantine विस्तारित विशेषता को विरासत में प्राप्त करने में विफल हो जाते हैं, जिससे उन फाइलों के लिए Gatekeeper को **बायपास करना संभव हो जाता है**।

अधिक जानकारी के लिए [**मूल रिपोर्ट**](https://labs.withsecure.com/publications/the-discovery-of-cve-2021-1810) देखें।

### [CVE-2021-30990](https://ronmasas.com/posts/bypass-macos-gatekeeper)

जब एक एप्लिकेशन **Automator** के साथ बनाया जाता है, तो इसे निष्पादित करने के लिए जरूरी जानकारी `application.app/Contents/document.wflow` के अंदर होती है, न कि एक्जीक्यूटेबल में। एक्जीक्यूटेबल सिर्फ एक सामान्य Automator बाइनरी होती है जिसे **Automator Application Stub** कहा जाता है।

इसलिए, आप `application.app/Contents/MacOS/Automator\ Application\ Stub` को सिस्टम के अंदर एक अन्य Automator Application Stub की ओर एक सिम्बोलिक लिंक के साथ **पॉइंट कर सकते हैं** और यह `document.wflow` के अंदर क्या है (आपकी स्क्रिप्ट) **बिना Gatekeeper को ट्रिगर किए** निष्पादित करेगा क्योंकि वास्तविक एक्जीक्यूटेबल में quarantine xattr नहीं होता है।&#x20;

उम्मीद की जाने वाली स्थान का उदाहरण: `/System/Library/CoreServices/Automator\ Application\ Stub.app/Contents/MacOS/Automator\ Application\ Stub`

अधिक जानकारी के लिए [**मूल रिपोर्ट**](https://ronmasas.com/posts/bypass-macos-gatekeeper) देखें।

### [CVE-2022-22616](https://www.jamf.com/blog/jamf-threat-labs-safari-vuln-gatekeeper-bypass/)

इस बायपास में एक zip फाइल `application.app/Contents` से शुरू होकर संपीड़ित की गई थी, न कि `application.app` से। इसलिए, **quarantine attr** सभी **फाइलों पर `application.app/Contents`** पर लागू किया गया था लेकिन **`application.app` पर नहीं**, जिसे Gatekeeper जांच रहा था, इसलिए Gatekeeper को बायपास किया गया क्योंकि जब `application.app` को ट्रिगर किया गया तो उसमें **quarantine विशेषता नहीं थी।**
```bash
zip -r test.app/Contents test.zip
```
अधिक जानकारी के लिए [**मूल रिपोर्ट**](https://www.jamf.com/blog/jamf-threat-labs-safari-vuln-gatekeeper-bypass/) देखें।

### [CVE-2022-32910](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2022-32910)

भले ही घटक अलग हों, इस भेद्यता का शोषण पिछले एक के समान ही है। इस मामले में हम **`application.app/Contents`** से एक Apple Archive बनाएंगे ताकि **`application.app`** को **Archive Utility** द्वारा डिकंप्रेस किए जाने पर quarantine attr न मिले।
```bash
aa archive -d test.app/Contents -o test.app.aar
```
अधिक जानकारी के लिए [**मूल रिपोर्ट**](https://www.jamf.com/blog/jamf-threat-labs-macos-archive-utility-vulnerability/) देखें।

### [CVE-2022-42821](https://www.microsoft.com/en-us/security/blog/2022/12/19/gatekeepers-achilles-heel-unearthing-a-macos-vulnerability/)

ACL **`writeextattr`** का उपयोग किसी फाइल में एक विशेषता लिखने से किसी को भी रोकने के लिए किया जा सकता है:
```bash
touch /tmp/no-attr
chmod +a "everyone deny writeextattr" /tmp/no-attr
xattr -w attrname vale /tmp/no-attr
xattr: [Errno 13] Permission denied: '/tmp/no-attr'
```
इसके अलावा, **AppleDouble** फाइल प्रारूप एक फाइल की प्रतिलिपि उसके ACEs सहित बनाता है।

[**सोर्स कोड**](https://opensource.apple.com/source/Libc/Libc-391/darwin/copyfile.c.auto.html) में यह देखा जा सकता है कि xattr **`com.apple.acl.text`** में संग्रहीत ACL पाठ प्रतिनिधित्व, डिकम्प्रेस्ड फाइल में ACL के रूप में सेट किया जाएगा। इसलिए, यदि आपने एक एप्लिकेशन को **AppleDouble** फाइल प्रारूप में एक zip फाइल में संपीड़ित किया है जिसमें एक ACL है जो अन्य xattrs को उस पर लिखने से रोकता है... तो एप्लिकेशन में quarantine xattr सेट नहीं किया गया था:

{% code overflow="wrap" %}
```bash
chmod +a "everyone deny write,writeattr,writeextattr" /tmp/test
ditto -c -k test test.zip
python3 -m http.server
# Download the zip from the browser and decompress it, the file should be without a quarantine xattr
```
{% endcode %}

अधिक जानकारी के लिए [**मूल रिपोर्ट**](https://www.microsoft.com/en-us/security/blog/2022/12/19/gatekeepers-achilles-heel-unearthing-a-macos-vulnerability/) देखें।

ध्यान दें कि इसका शोषण AppleArchives के साथ भी किया जा सकता है:
```bash
mkdir app
touch app/test
chmod +a "everyone deny write,writeattr,writeextattr" app/test
aa archive -d app -o test.aar
```
### [CVE-2023-27943](https://blog.f-secure.com/discovery-of-gatekeeper-bypass-cve-2023-27943/)

यह पता चला कि **Google Chrome डाउनलोड की गई फाइलों को quarantine attribute सेट नहीं कर रहा था** क्योंकि macOS की कुछ आंतरिक समस्याओं के कारण।

### [CVE-2023-27951](https://redcanary.com/blog/gatekeeper-bypass-vulnerabilities/)

AppleDouble फाइल फॉर्मेट्स एक अलग फाइल में फाइल के गुणों को संग्रहीत करते हैं जो `._` से शुरू होती है, यह **macOS मशीनों के बीच** फाइल गुणों की प्रतिलिपि बनाने में मदद करता है। हालांकि, यह देखा गया कि AppleDouble फाइल को डिकंप्रेस करने के बाद, `._` से शुरू होने वाली फाइल को quarantine attribute नहीं दिया गया था।

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

क्वारंटाइन एट्रिब्यूट सेट न होने वाली फाइल बनाने में सक्षम होने से, **Gatekeeper को बायपास करना संभव था।** चाल यह थी कि **AppleDouble नाम संविधान का उपयोग करके DMG फाइल एप्लिकेशन बनाएं** (इसे `._` से शुरू करें) और एक **दृश्यमान फाइल को इस छिपी** फाइल के लिए सिम लिंक के रूप में बनाएं जिसमें क्वारंटाइन एट्रिब्यूट नहीं है।\
जब **dmg फाइल निष्पादित की जाती है**, चूंकि इसमें क्वारंटाइन एट्रिब्यूट नहीं होता, यह **Gatekeeper को बायपास कर देगी।**
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
### क्वारंटाइन xattr रोकें

यदि ".app" बंडल में क्वारंटाइन xattr जोड़ा नहीं गया है, तो इसे निष्पादित करते समय **Gatekeeper सक्रिय नहीं होगा**।

<details>

<summary><strong>शून्य से लेकर हीरो तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) **का अनुसरण करें**।
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>
