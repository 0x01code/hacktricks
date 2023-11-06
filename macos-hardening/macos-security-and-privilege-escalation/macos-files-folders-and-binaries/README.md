# macOS फ़ाइलें, फ़ोल्डर, बाइनरी और मेमोरी

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें और PR जमा करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **में सबमिट करके**।

</details>

## फ़ाइल व्यवस्था लेआउट

* **/Applications**: स्थापित ऐप्स यहां होने चाहिए। सभी उपयोगकर्ता उन तक पहुंच सकेंगे।
* **/bin**: कमांड लाइन बाइनरी
* **/cores**: यदि मौजूद है, तो यह कोर डंप संग्रहित करने के लिए उपयोग किया जाता है।
* **/dev**: सब कुछ एक फ़ाइल के रूप में व्यवहार किया जाता है, इसलिए आपको यहां हार्डवेयर उपकरण देख सकते हैं।
* **/etc**: कॉन्फ़िगरेशन फ़ाइलें
* **/Library**: यहां वरीयताओं, कैश और लॉग के संबंधित कई उपनिर्देशिकाएं और फ़ाइलें मौजूद हो सकती हैं। एक पुस्तकालय फ़ोल्डर मूल और प्रत्येक उपयोगकर्ता के निर्देशिका में मौजूद होता है।
* **/private**: अनदोक्यूमेंटेड है लेकिन उल्लिखित फ़ोल्डरों की बहुत सारी प्रतीकात्मक लिंक यहां रखी जाती हैं।
* **/sbin**: महत्वपूर्ण सिस्टम बाइनरी (प्रशासन से संबंधित)
* **/System**: OS X को चलाने के लिए फ़ाइल। यहां आपको अधिकांशतः केवल Apple की विशेष फ़ाइलें मिलेंगी (तीसरे पक्षी नहीं)।
* **/tmp**: फ़ाइलें 3 दिनों के बाद हटा दी जाती हैं (यह /private/tmp के लिए एक सॉफ़्ट लिंक है)
* **/Users**: उपयोगकर्ताओं के लिए होम निर्देशिका।
* **/usr**: कॉन्फ़िग और सिस्टम बाइनरी
* **/var**: लॉग फ़ाइलें
* **/Volumes**: माउंट किए गए ड्राइव यहां दिखाई देंगे।
* **/.vol**: `stat a.txt` चलाने पर आपको कुछ इस तरह का कुछ प्राप्त होगा `16777223 7545753 -rw-r--r-- 1 username wheel ...` जहां पहला नंबर फ़ाइल मौजूद वॉल्यूम का आईडी नंबर है और दूसरा एक इनोड नंबर है। आप उस जानकारी के साथ /.vol/ के माध्यम से इस फ़ाइल की सामग्री तक पहुंच सकते हैं `cat /.vol/16777223/7545753` चलाकर।

### ऐप्लिकेशन फ़ोल्डर

* **सिस्टम ऐप्लिकेशनें** `/System/Applications` के तहत स्थित होती हैं
* **स्थापित** ऐप्लिकेशनें आमतौर पर `/Applications` या `~/Applications` में स्थापित होती हैं
* **ऐप्लिकेशन डेटा** रूट के रूप में चल रही ऐप्लिकेशन के लिए `/Library/Application Support` में और उपयोगकर्ता के रूप में चल रही ऐप्लिकेशन के लिए `~/Library/Application Support` में पाया जा सकता है।
* **तृतीय-पक्ष ऐप्स के डेमन** जो **रूट के रूप में चलने की आवश्यकता होती है** आमतौर पर `/Library/PrivilegedHelper
### macOS बंडल

मूल रूप से, एक बंडल फ़ाइल सिस्टम के भीतर एक **निर्देशिका संरचना** है। दिलचस्प बात यह है कि इस निर्देशिका को डिफ़ॉल्ट रूप में फ़ाइंडर में एकल वस्तु की तरह दिखाया जाता है (जैसे `.app`)।

{% content-ref url="macos-bundles.md" %}
[macos-bundles.md](macos-bundles.md)
{% endcontent-ref %}

## Dyld साझा कैश

macOS (और iOS) पर सभी सिस्टम साझा पुस्तकालयें, जैसे फ़्रेमवर्क और dylib, **एक ही फ़ाइल में संयुक्त की जाती हैं**, जिसे **dyld साझा कैश** कहा जाता है। यह प्रदर्शन में सुधार करता है, क्योंकि कोड तेजी से लोड किया जा सकता है।

dyld साझा कैश के समान रूप में, कर्नल और कर्नल एक्सटेंशन भी एक कर्नल कैश में कंपाइल किए जाते हैं, जो बूट समय लोड होता है।

एकल फ़ाइल dylib साझा कैश से पुस्तकालयों को निकालने के लिए बाइनरी [dyld\_shared\_cache\_util](https://www.mbsplugins.de/files/dyld\_shared\_cache\_util-dyld-733.8.zip) का उपयोग किया जा सकता था, जो अब काम नहीं कर सकता है, लेकिन आप [**dyldextractor**](https://github.com/arandomdev/dyldextractor) का भी उपयोग कर सकते हैं: 

{% code overflow="wrap" %}
```bash
# dyld_shared_cache_util
dyld_shared_cache_util -extract ~/shared_cache/ /System/Volumes/Preboot/Cryptexes/OS/System/Library/dyld/dyld_shared_cache_arm64e

# dyldextractor
dyldex -l [dyld_shared_cache_path] # List libraries
dyldex_all [dyld_shared_cache_path] # Extract all
# More options inside the readme
```
{% endcode %}

पुराने संस्करणों में आप **`/System/Library/dyld/`** में **साझा कैश** ढूंढ़ सकते हैं।

iOS में आप उन्हें **`/System/Library/Caches/com.apple.dyld/`** में ढूंढ़ सकते हैं।

{% hint style="success" %}
ध्यान दें कि यदि `dyld_shared_cache_util` उपकरण काम नहीं करता है, तो आप **साझा dyld बाइनरी को Hopper को पास** कर सकते हैं और Hopper सभी पुस्तकालयों की पहचान कर सकता है और आपको **जांचने के लिए कौन सा** चुनने देगा:
{% endhint %}

<figure><img src="../../../.gitbook/assets/image (680).png" alt="" width="563"><figcaption></figcaption></figure>

## विशेष फ़ाइल अनुमतियाँ

### फ़ोल्डर अनुमतियाँ

एक **फ़ोल्डर** में, **पढ़ने** की अनुमति उसे **सूचीबद्ध करने** की अनुमति देती है, **लिखने** की अनुमति उसे फ़ाइलें **हटाने** और **लिखने** की अनुमति देती है, और **चलाने** की अनुमति उसे निर्देशिका में **चलने** की अनुमति देती है। इसलिए, उदाहरण के लिए, एक उपयोगकर्ता को एक फ़ाइल पर **पढ़ने की अनुमति** होती है जो उसे निर्देशिका में है जहां उसे **चलाने की** अनुमति नहीं है, तो वह फ़ाइल **पढ़ने में सक्षम नहीं होगा**।

### फ़्लैग संशोधक

कुछ फ़्लैग हो सकते हैं जो फ़ाइल को अलग ढंग से व्यवहार करने के लिए सेट कर सकते हैं। आप `ls -lO /path/directory` के साथ एक निर्देशिका में फ़ाइलों के फ़्लैग की जांच कर सकते हैं

* **`uchg`**: यह ज्ञात है कि **uchange** फ़्लैग फ़ाइल को **किसी भी कार्रवाई से बदलने या हटाने से रोकेगा**। इसे सेट करने के लिए: `chflags uchg file.txt`
* रूट उपयोगकर्ता फ़्लैग को **हटा सकता है** और फ़ाइल को संशोधित कर सकता है
* **`restricted`**: यह फ़्लैग फ़ाइल को **SIP द्वारा संरक्षित** करता है (आप इस फ़्लैग को फ़ाइल में नहीं जोड़ सकते हैं)।
* **`Sticky bit`**: यदि एक निर्देशिका में sticky bit होता है, **केवल** निर्देशिका के मालिक या रूट उपयोगकर्ता फ़ाइलों को नाम बदल सकते हैं या हटा सकते हैं। आमतौर पर यह /tmp निर्देशिका पर सेट किया जाता है ताकि साधारण उपयोगकर्ता अन्य उपयोगकर्ताओं की फ़ाइलें हटाने या हटाने से रोक सकें।

### **फ़ाइल ACLs**

फ़ाइल **ACLs** में **ACE** (पहुँच नियंत्रण प्रविष्टियाँ) होती हैं जहां विभिन्न उपयोगकर्ताओं को अधिक **विस्तृत अनुमतियाँ** प्राप्त कराई जा सकती हैं।

एक **निर्देशिका** को इन अनुमतियों को प्रदान करना संभव है: `list`, `search`, `add_file`, `add_subdirectory`, `delete_child`, `delete_child`।
और एक **फ़ाइल** को: `read`, `write`, `append`, `execute`।

जब फ़ाइल में ACLs होती हैं तो आप **अनुमतियों की सूचीबद्ध करते समय "+" ढंग से पाएंगे जैसे कि**:
```bash
ls -ld Movies
drwx------+   7 username  staff     224 15 Apr 19:42 Movies
```
आप निम्नलिखित कमांड के साथ फ़ाइल की ACL पढ़ सकते हैं:
```bash
ls -lde Movies
drwx------+ 7 username  staff  224 15 Apr 19:42 Movies
0: group:everyone deny delete
```
आप (यह बहुत धीमा होगा) के साथ **ACLs वाले सभी फ़ाइलों को** ढूंढ सकते हैं:
```bash
ls -RAle / 2>/dev/null | grep -E -B1 "\d: "
```
### संसाधन फोर्क्स | macOS ADS

यह मैकओएस मशीनों में **वैकल्पिक डेटा स्ट्रीम्स** प्राप्त करने का एक तरीका है। आप एक फ़ाइल के अंदर सामग्री को **com.apple.ResourceFork** नामक एक्सटेंडेड एट्रिब्यूट में सहेज सकते हैं जिसे **file/..namedfork/rsrc** में सहेजकर किया जाता है।
```bash
echo "Hello" > a.txt
echo "Hello Mac ADS" > a.txt/..namedfork/rsrc

xattr -l a.txt #Read extended attributes
com.apple.ResourceFork: Hello Mac ADS

ls -l a.txt #The file length is still q
-rw-r--r--@ 1 username  wheel  6 17 Jul 01:15 a.txt
```
आप इस विस्तृत गुणधर्म को सम्पन्न सभी फ़ाइलों को खोज सकते हैं इसके साथ:

{% code overflow="wrap" %}
```bash
find / -type f -exec ls -ld {} \; 2>/dev/null | grep -E "[x\-]@ " | awk '{printf $9; printf "\n"}' | xargs -I {} xattr -lv {} | grep "com.apple.ResourceFork"
```
{% endcode %}

## **यूनिवर्सल बाइनरीज़ और** Mach-o फॉर्मेट

मैक ओएस बाइनरीज़ आमतौर पर **यूनिवर्सल बाइनरीज़** के रूप में कंपाइल होते हैं। एक **यूनिवर्सल बाइनरी** में **एक ही फ़ाइल में कई आर्किटेक्चर का समर्थन किया जा सकता है**।

{% content-ref url="universal-binaries-and-mach-o-format.md" %}
[universal-binaries-and-mach-o-format.md](universal-binaries-and-mach-o-format.md)
{% endcontent-ref %}

## macOS मेमोरी डंपिंग

{% content-ref url="macos-memory-dumping.md" %}
[macos-memory-dumping.md](macos-memory-dumping.md)
{% endcontent-ref %}

## रिस्क श्रेणी फ़ाइलें Mac OS

फ़ाइलें `/System/Library/CoreServices/CoreTypes.bundle/Contents/Resources/System` में फ़ाइल एक्सटेंशन के आधार पर फ़ाइलों से संबंधित जोखिम शामिल होता है।

संभावित श्रेणियों में निम्नलिखित शामिल हैं:

* **LSRiskCategorySafe**: **पूरी तरह से** **सुरक्षित**; डाउनलोड के बाद सफारी ऑटो-ओपन होगी
* **LSRiskCategoryNeutral**: कोई चेतावनी नहीं, लेकिन **ऑटो-ओपन नहीं होती**
* **LSRiskCategoryUnsafeExecutable**: एक **चेतावनी** "यह फ़ाइल एक एप्लिकेशन है..." को **ट्रिगर करता है**
* **LSRiskCategoryMayContainUnsafeExecutable**: इसका उपयोग कुछ ऐसे आर्काइव के लिए होता है जिसमें एक एक्सेक्यूटेबल होता है। यह **चेतावनी ट्रिगर करता है जब तक सफारी सभी सामग्री सुरक्षित या न्यूट्रल निर्धारित कर सकता है**।

## लॉग फ़ाइलें

* **`$HOME/Library/Preferences/com.apple.LaunchServices.QuarantineEventsV2`**: डाउनलोड की गई फ़ाइलों के बारे में जानकारी शामिल है, जैसे कि वे कहां से डाउनलोड की गई थीं।
* **`/var/log/system.log`**: OSX सिस्टम का मुख्य लॉग। com.apple.syslogd.plist सिस्टम लॉगिंग के निष्पादन के लिए जिम्मेदार है (आप "com.apple.syslogd" को `launchctl list` में खोजकर देख सकते हैं कि क्या यह अक्षम है।
* **`/private/var/log/asl/*.asl`**: ये एप्पल सिस्टम लॉग हैं जिनमें दिलचस्प जानकारी हो सकती है।
* **`$HOME/Library/Preferences/com.apple.recentitems.plist`**: "फ़ाइंडर" के माध्यम से हाल ही में एक्सेस की गई फ़ाइलें और एप्लिकेशन संग्रहीत करता है।
* **`$HOME/Library/Preferences/com.apple.loginitems.plsit`**: सिस्टम स्टार्टअप पर लॉन्च होने वाले आइटम संग्रहीत करता है
* **`$HOME/Library/Logs/DiskUtility.log`**: DiskUtility ऐप के लिए लॉग फ़ाइल (ड्राइव्स के बारे में जानकारी, सहित USB)
* **`/Library/Preferences/SystemConfiguration/com.apple.airport.preferences.plist`**: वायरलेस एक्सेस प्वाइंट के बारे में डेटा।
* **`/private/var/db/launchd.db/com.apple.launchd/overrides.plist`**: निष्क्रिय किए गए डेमन की सूची।

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **हैकट्रिक्स में विज्ञापित** देखना चाहते हैं? या क्या आप **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड** करने का उद्देश्य रखते हैं? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह
* प्राप्त करें [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को**।

</details>
