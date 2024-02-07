# macOS फ़ाइलें, फ़ोल्डर, बाइनरी और मेमोरी

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी कंपनी का विज्ञापन HackTricks में देखना चाहते हैं या **HackTricks को PDF में डाउनलोड** करना चाहते हैं तो [**सब्सक्रिप्शन प्लान**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मेरा** ट्विटर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** का पालन करें।**
* **HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PR जमा करके अपने हैकिंग ट्रिक्स साझा करें।

</details>

## फ़ाइल व्यवस्था लेआउट

* **/Applications**: स्थापित ऐप्स यहाँ होने चाहिए। सभी उपयोगकर्ता उन्हें एक्सेस कर सकेंगे।
* **/bin**: कमांड लाइन बाइनरी
* **/cores**: अगर मौजूद है, तो यह कोर डंप स्टोर करने के लिए उपयोग किया जाता है
* **/dev**: सब कुछ एक फ़ाइल के रूप में देखा जाता है इसलिए आप हार्डवेयर डिवाइस यहाँ स्टोर किए गए देख सकते हैं।
* **/etc**: कॉन्फ़िगरेशन फ़ाइलें
* **/Library**: यहाँ प्राथमिकताएँ, कैश और लॉग्स से संबंधित कई उपनिर्देशिकाएँ और फ़ाइलें पाई जा सकती हैं। एक लाइब्रेरी फ़ोल्डर मूल और प्रत्येक उपयोगकर्ता के निर्देशिका में मौजूद है।
* **/private**: अनसंदर्भित लेकिन उल्लिखित फ़ोल्डरों की बहुत सारी प्रतीकात्मक लिंक्स हैं जो निजी डायरेक्टरी के लिए हैं।
* **/sbin**: महत्वपूर्ण सिस्टम बाइनरी (प्रशासन से संबंधित)
* **/System**: OS X को चलाने के लिए फ़ाइल। आपको यहाँ अधिकांशत: केवल Apple विशिष्ट फ़ाइलें मिलेंगी (तीसरे पक्ष नहीं)।
* **/tmp**: फ़ाइलें 3 दिनों के बाद हटा दी जाती हैं (यह /private/tmp के लिए एक सॉफ़्ट लिंक है)
* **/Users**: उपयोगकर्ताओं के लिए होम डायरेक्टरी।
* **/usr**: कॉन्फ़िग और सिस्टम बाइनरी
* **/var**: लॉग फ़ाइलें
* **/Volumes**: माउंट की गई ड्राइव्स यहाँ दिखाई देंगी।
* **/.vol**: `stat a.txt` चलाने पर आपको `16777223 7545753 -rw-r--r-- 1 username wheel ...` जैसी कुछ प्राप्त होगी जहाँ पहला नंबर वह वॉल्यूम का आईडी नंबर है जहाँ फ़ाइल मौजूद है और दूसरा इनोड नंबर है। आप इस फ़ाइल की सामग्री तक पहुंच सकते हैं /.vol/ के माध्यम से उस जानकारी के साथ चलाने `cat /.vol/16777223/7545753`

### एप्लिकेशन फ़ोल्डर

* **सिस्टम एप्लिकेशन** `/System/Applications` के तहत स्थित हैं
* **स्थापित** एप्लिकेशन आम तौर पर `/Applications` या `~/Applications` में स्थापित होते हैं
* **एप्लिकेशन डेटा** `/Library/Application Support` में पाया जा सकता है जो रूट के रूप में चल रहे एप्लिकेशनों के लिए है और `~/Library/Application Support` उपयोगकर्ता के रूप में चल रहे एप्लिकेशनों के लिए है।
* **संदर्भित** एप्स को `~/Library/Containers` फ़ोल्डर में मैप किया जाता है। प्रत्येक एप्लिकेशन के पास एक फ़ोल्डर होता है जिसका नाम एप्लिकेशन के बंडल ID के अनुसार होता है (`com.apple.Safari`)।
* **कर्नेल** `/System/Library/Kernels/kernel` में स्थित है
* **Apple के कर्नेल एक्सटेंशन** `/System/Library/Extensions` में स्थित हैं
* **तीसरे पक्ष कर्नेल एक्सटेंशन** `/Library/Extensions` में स्टोर किए जाते हैं

### संवेदनशील जानकारी वाली फ़ाइलें

MacOS जैसी जगहों पर पासवर्ड जैसी जानकारी स्टोर करता है:

{% content-ref url="macos-sensitive-locations.md" %}
[macos-sensitive-locations.md](macos-sensitive-locations.md)
{% endcontent-ref %}

### वंलरेबल pkg इंस्टॉलर्स

{% content-ref url="macos-installers-abuse.md" %}
[macos-installers-abuse.md](macos-installers-abuse.md)
{% endcontent-ref %}

## OS X विशेष एक्सटेंशन्स

* **`.dmg`**: Apple Disk Image फ़ाइलें इंस्टॉलर्स के लिए बहुत आम हैं।
* **`.kext`**: यह एक विशिष्ट संरचना का पालन करना चाहिए और यह ड्राइवर का OS X संस्करण है। (यह एक बंडल है)
* **`.plist`**: XML या बाइनरी स्वरूप में जानकारी स्टोर करता है।
* XML या बाइनरी हो सकता है। बाइनरी वाले को इस प्रकार से पढ़ा जा सकता है:
* `defaults read config.plist`
* `/usr/libexec/PlistBuddy -c print config.plsit`
* `plutil -p ~/Library/Preferences/com.apple.screensaver.plist`
* `plutil -convert xml1 ~/Library/Preferences/com.apple.screensaver.plist -o -`
* `plutil -convert json ~/Library/Preferences/com.apple.screensaver.plist -o -`
* **`.app`**: एप्पल एप्लिकेशन जो डायरेक्टरी संरचना का पालन करते हैं (यह एक बंडल है)।
* **`.dylib`**: डायनामिक लाइब्रेरीज (जैसे Windows DLL फ़ाइलें)
* **`.pkg`**: xar (eXtensible Archive format) के समान हैं। इन फ़ाइलों के सामग्री को इंस्टॉल करने के लिए इंस्टॉलर कमांड का उपयोग किया जा सकता है।
* **`.DS_Store`**: हर डायरेक्टरी पर यह फ़ाइल होती है, यह डायरेक्टरी की विशेषताएँ और कस्टमाइजेशन को सहेजती है।
* **`.Spotlight-V100`**: यह फ़ोल्डर प्रत्येक वॉल्यूम के रूट डायरेक्टरी पर दिखाई देता है।
* **`.metadata_never_index`**: यदि यह फ़ाइल किसी वॉल्यूम के रूट पर है तो Spotlight उस वॉल्यूम को इंडेक्स नहीं करेगा।
* **`.noindex`**: इस एक्सटेंशन वाली फ़ाइलें और फ़ोल्डर Spotlight द्वारा इंडेक्स नहीं की जाएंगी।

### macOS बंडल्स

एक बंडल एक **डायरेक्टरी** है जो **फाइंडर में एक ऑब्जेक्ट की तरह दिखती है** (एक बंडल उदाहरण हैं `*.app` फ़ाइलें)।

{% content-ref url="macos-bundles.md" %}
[macos-bundles.md](macos-bundles.md)
{% endcontent-ref %}

## Dyld Shared Cache

macOS (और iOS) पर सभी सिस्टम साझा लाइब्रेरी, जैसे फ़्रेमवर्क और dylibs, को **एक ही फ़ाइल में** मिलाकर, जिसे **dyld साझा कैश** कहा जाता है। यह प्रदर्शन में सुधार करता है, क्योंकि कोड तेजी से लोड किया जा सकता है।

dyld साझा कैश के समान, कर्नेल और कर्नेल एक्सटेंशन भी एक कर्नेल कैश में कॉम्पाइल किए जाते हैं, जो बूट समय पर लोड किया जाता है।

एकल फ़ाइल dylib साझा कैश से पुस्तकालयों को निकालने के लिए संभव था जिसे बाइनरी [
```bash
# dyld_shared_cache_util
dyld_shared_cache_util -extract ~/shared_cache/ /System/Volumes/Preboot/Cryptexes/OS/System/Library/dyld/dyld_shared_cache_arm64e

# dyldextractor
dyldex -l [dyld_shared_cache_path] # List libraries
dyldex_all [dyld_shared_cache_path] # Extract all
# More options inside the readme
```
{% endcode %}

बुनियादी संस्करणों में आप **`/System/Library/dyld/`** में **`विभाजित कैश`** पा सकते हैं।

iOS में आप उन्हें **`/System/Library/Caches/com.apple.dyld/`** में पा सकते हैं।

{% hint style="success" %}
ध्यान दें कि यदि `dyld_shared_cache_util` उपकरण काम नहीं करता है, तो आप **साझा dyld बाइनरी को Hopper को पारित कर सकते हैं** और Hopper सभी पुस्तकालयों की पहचान करने में सक्षम होगा और आपको **जिसे आप जांचना चाहते हैं, उसे चुनने देगा**:
{% endhint %}

<figure><img src="../../../.gitbook/assets/image (680).png" alt="" width="563"><figcaption></figcaption></figure>

## विशेष फ़ाइल अनुमतियाँ

### फ़ोल्डर अनुमतियाँ

**फ़ोल्डर** में, **पढ़ने** से इसे **सूचीत करने** की अनुमति होती है, **लिखने** से इसे **हटाने** और फ़ाइलों पर **लिखने** की अनुमति होती है, और **क्रियान्वयन** से निर्देशिका **चालू करने** की अनुमति होती है। इसलिए, उदाहरण के लिए, एक उपयोगकर्ता को **फ़ाइल पर पढ़ने की अनुमति** होने पर भी उस निर्देशिका में जिस पर उसके **क्रियान्वयन** की अनुमति नहीं है, **फ़ाइल पढ़ने में सक्षम नहीं होगा**।

### ध्वज संशोधक

कुछ ध्वज हैं जो फ़ाइलों में सेट किए जा सकते हैं जो फ़ाइल का व्यवहार अलग बना देंगे। आप `ls -lO /पथ/निर्देशिका` के साथ निर्देशिका में फ़ाइलों के ध्वजों की जांच कर सकते हैं।

* **`uchg`**: **uchange** ध्वज के रूप में जाना जाता है जो किसी भी क्रिया को **बदलने या हटाने से रोकेगा**। इसे सेट करने के लिए: `chflags uchg file.txt`
* रूट उपयोगकर्ता ध्वज को **हटा सकता है** और फ़ाइल को संशोधित कर सकता है
* **`प्रतिबंधित`**: यह ध्वज फ़ाइल को **SIP द्वारा संरक्षित** बना देगा (आप इस ध्वज को फ़ाइल में नहीं जोड़ सकते)।
* **`Sticky bit`**: यदि एक निर्देशिका में sticky bit है, **केवल** निर्देशिका के मालिक या रूट **फ़ाइलों को नाम बदल सकते हैं या हटा सकते हैं**। सामान्यत: यह /tmp निर्देशिका पर सेट किया जाता है ताकि साधारण उपयोगकर्ता अन्य उपयोगकर्ताओं की फ़ाइलों को हटाने या हटाने से रोक सकें।

### **फ़ाइल ACLs**

फ़ाइल **ACLs** में **ACE** (पहुंच नियंत्रण प्रविष्टियाँ) होती हैं जहां विभिन्न उपयोगकर्ताओं को विभिन्न अनुमतियाँ सौंपी जा सकती हैं।

एक **निर्देशिका** को ये अनुमतियाँ प्रदान करना संभव है: `सूची`, `खोज`, `फ़ाइल जोड़ें`, `उपनिर्देशिका जोड़ें`, `बच्चे को हटाएं`, `बच्चे को हटाएं`।\
और एक **फ़ाइल** को: `पढ़ें`, `लिखें`, `जोड़ें`, `क्रियान्वित करें`।

जब फ़ाइल में ACLs होती हैं तो आपको **अनुमतियों को सूचीत करते समय "+" मिलेगा जैसे कि**:
```bash
ls -ld Movies
drwx------+   7 username  staff     224 15 Apr 19:42 Movies
```
आप निम्नलिखित के साथ **फ़ाइल की ACLs पढ़ सकते हैं:**
```bash
ls -lde Movies
drwx------+ 7 username  staff  224 15 Apr 19:42 Movies
0: group:everyone deny delete
```
आप **सभी फ़ाइलें एसीएल के साथ** इसके साथ पा सकते हैं (यह बहुत ही धीमा है):
```bash
ls -RAle / 2>/dev/null | grep -E -B1 "\d: "
```
### संसाधन फोर्क्स | macOS ADS

यह **मैकओएस मशीनों में विकल्प डेटा स्ट्रीम्स प्राप्त करने का एक तरीका है।** आप एक फ़ाइल के अंदर **com.apple.ResourceFork** नामक विस्तारित विशेषता में सामग्री सहेज सकते हैं इसे **file/..namedfork/rsrc** में सहेजकर।
```bash
echo "Hello" > a.txt
echo "Hello Mac ADS" > a.txt/..namedfork/rsrc

xattr -l a.txt #Read extended attributes
com.apple.ResourceFork: Hello Mac ADS

ls -l a.txt #The file length is still q
-rw-r--r--@ 1 username  wheel  6 17 Jul 01:15 a.txt
```
आप इस विस्तृत विशेषता को धारित सभी फ़ाइलें निम्नलिखित के साथ पा सकते हैं:

{% code overflow="wrap" %}
```bash
find / -type f -exec ls -ld {} \; 2>/dev/null | grep -E "[x\-]@ " | awk '{printf $9; printf "\n"}' | xargs -I {} xattr -lv {} | grep "com.apple.ResourceFork"
```
{% endcode %}

## **यूनिवर्सल बाइनरीज और** Mach-o फॉर्मेट

मैक ओएस बाइनरीज आम तौर पर **यूनिवर्सल बाइनरीज** के रूप में कंपाइल किए जाते हैं। एक **यूनिवर्सल बाइनरी** में **एक ही फ़ाइल में कई आर्किटेक्चर का समर्थन कर सकता है**।

{% content-ref url="universal-binaries-and-mach-o-format.md" %}
[universal-binaries-and-mach-o-format.md](universal-binaries-and-mach-o-format.md)
{% endcontent-ref %}

## macOS मेमोरी डंपिंग

{% content-ref url="macos-memory-dumping.md" %}
[macos-memory-dumping.md](macos-memory-dumping.md)
{% endcontent-ref %}

## जोखिम श्रेणी फ़ाइलें Mac OS

डायरेक्टरी `/System/Library/CoreServices/CoreTypes.bundle/Contents/Resources/System` यहाँ जानकारी होती है कि **विभिन्न फ़ाइल एक्सटेंशन के साथ जुड़े जोखिम के साथ कितना संबंध है**। इस डायरेक्टरी में फ़ाइलों को विभिन्न जोखिम स्तरों में वर्गीकृत किया जाता है, जो सफारी को डाउनलोड के बाद इन फ़ाइलों का संचालन कैसे करना है पर प्रभाव डालता है। इन श्रेणियों में निम्नलिखित शामिल हैं:

- **LSRiskCategorySafe**: इस श्रेणी में फ़ाइलें **पूरी तरह सुरक्षित** मानी जाती हैं। सफारी इन फ़ाइलों को स्वचालित रूप से डाउनलोड के बाद खोलेगा।
- **LSRiskCategoryNeutral**: इन फ़ाइलों के साथ कोई चेतावनी नहीं होती और सफारी द्वारा **स्वचालित रूप से नहीं खोली जाती** है।
- **LSRiskCategoryUnsafeExecutable**: इस श्रेणी की फ़ाइलें **एक चेतावनी प्रेरित करती हैं** जिसमें इसका संवर्गन एक एप्लिकेशन है। यह उपयोगकर्ता को सूचित करने के लिए एक सुरक्षा उपाय के रूप में काम करता है।
- **LSRiskCategoryMayContainUnsafeExecutable**: यह श्रेणी फ़ाइलों के लिए है, जैसे कि आर्काइव, जो एक एक्सीक्यूटेबल शामिल कर सकते हैं। सफारी **एक चेतावनी प्रेरित करेगा** जब तक यह सत्यापित नहीं कर सकता कि सभी सामग्री सुरक्षित या न्यूट्रल है।

## लॉग फ़ाइलें

* **`$HOME/Library/Preferences/com.apple.LaunchServices.QuarantineEventsV2`**: डाउनलोड की गई फ़ाइलों के बारे में जानकारी रखता है, जैसे कि उन्हें कहाँ से डाउनलोड किया गया था।
* **`/var/log/system.log`**: OSX सिस्टम का मुख्य लॉग। com.apple.syslogd.plist सिस्टम लॉगिंग का निष्पादन जिम्मेदार है (आप "launchctl list" में "com.apple.syslogd" की खोज करके देख सकते हैं कि क्या यह निष्क्रिय है या नहीं)।
* **`/private/var/log/asl/*.asl`**: ये एप्पल सिस्टम लॉग हैं जो दिलचस्प जानकारी रख सकते हैं।
* **`$HOME/Library/Preferences/com.apple.recentitems.plist`**: "फाइंडर" के माध्यम से हाल ही में एक्सेस की गई फ़ाइलें और एप्लिकेशन स्टोर करता है।
* **`$HOME/Library/Preferences/com.apple.loginitems.plsit`**: सिस्टम स्टार्टअप पर लॉन्च करने के लिए आइटम स्टोर करता है।
* **`$HOME/Library/Logs/DiskUtility.log`**: DiskUtility ऐप के लिए लॉग फ़ाइल (ड्राइव्स के बारे में जानकारी, इसमें USB भी शामिल हैं)।
* **`/Library/Preferences/SystemConfiguration/com.apple.airport.preferences.plist`**: वायरलेस एक्सेस पॉइंट्स के बारे में डेटा।
* **`/private/var/db/launchd.db/com.apple.launchd/overrides.plist`**: निष्क्रिय किए गए डेमन्स की सूची।
