# macOS फाइल्स, फोल्डर्स, बाइनरीज़ और मेमोरी

<details>

<summary><strong>शून्य से हीरो तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) **का अनुसरण करें**.
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें.

</details>

## फाइल हायरार्की लेआउट

* **/Applications**: इंस्टॉल किए गए ऐप्स यहाँ होने चाहिए। सभी यूजर्स इन्हें एक्सेस कर सकेंगे।
* **/bin**: कमांड लाइन बाइनरीज़
* **/cores**: अगर मौजूद है, तो यह कोर डंप्स स्टोर करने के लिए इस्तेमाल होता है
* **/dev**: सब कुछ फाइल के रूप में माना जाता है इसलिए आपको हार्डवेयर डिवाइसेस यहाँ स्टोर होते दिख सकते हैं।
* **/etc**: कॉन्फ़िगरेशन फाइल्स
* **/Library**: यहाँ बहुत सारी सबडायरेक्टरीज़ और फाइल्स प्रेफरेंसेस, कैशेस और लॉग्स से संबंधित मिल सकती हैं। एक Library फोल्डर रूट में और प्रत्येक यूजर की डायरेक्टरी में मौजूद होता है।
* **/private**: अनडॉक्यूमेंटेड लेकिन बहुत सारे उल्लेखित फोल्डर्स प्राइवेट डायरेक्टरी के सिम्बॉलिक लिंक्स होते हैं।
* **/sbin**: एसेंशियल सिस्टम बाइनरीज़ (प्रशासन से संबंधित)
* **/System**: OS X को चलाने के लिए फाइल। यहाँ ज्यादातर केवल Apple विशिष्ट फाइल्स मिलनी चाहिए (तीसरे पक्ष की नहीं)।
* **/tmp**: फाइल्स 3 दिनों के बाद डिलीट हो जाती हैं (यह /private/tmp का सॉफ्ट लिंक है)
* **/Users**: यूजर्स के लिए होम डायरेक्टरी।
* **/usr**: कॉन्फ़िग और सिस्टम बाइनरीज़
* **/var**: लॉग फाइल्स
* **/Volumes**: माउंटेड ड्राइव्स यहाँ दिखाई देंगी।
* **/.vol**: `stat a.txt` चलाने पर आपको कुछ ऐसा मिलता है `16777223 7545753 -rw-r--r-- 1 username wheel ...` जहाँ पहला नंबर उस वॉल्यूम का आईडी नंबर होता है जहाँ फाइल मौजूद है और दूसरा इनोड नंबर होता है। आप इस जानकारी के साथ /.vol/ के माध्यम से इस फाइल की सामग्री तक पहुँच सकते हैं `cat /.vol/16777223/7545753`

### एप्लिकेशन्स फोल्डर्स

* **सिस्टम एप्लिकेशन्स** `/System/Applications` के अंतर्गत स्थित होते हैं।
* **इंस्टॉल किए गए** एप्लिकेशन्स आमतौर पर `/Applications` या `~/Applications` में इंस्टॉल किए जाते हैं।
* **एप्लिकेशन डेटा** रूट के रूप में चल रहे एप्लिकेशन्स के लिए `/Library/Application Support` में और यूजर के रूप में चल रहे एप्लिकेशन्स के लिए `~/Library/Application Support` में मिल सकता है।
* तीसरे पक्ष के एप्लिकेशन्स **daemons** जिन्हें **रूट के रूप में चलने की आवश्यकता होती है** आमतौर पर `/Library/PrivilegedHelperTools/` में स्थित होते हैं।
* **Sandboxed** एप्स `~/Library/Containers` फोल्डर में मैप किए जाते हैं। प्रत्येक एप्लिकेशन का एक फोल्डर होता है जो एप्लिकेशन के बंडल ID (`com.apple.Safari`) के अनुसार नामित होता है।
* **कर्नेल** `/System/Library/Kernels/kernel` में स्थित होता है।
* **Apple के कर्नेल एक्सटेंशन्स** `/System/Library/Extensions` में स्थित होते हैं।
* **तीसरे पक्ष के कर्नेल एक्सटेंशन्स** `/Library/Extensions` में संग्रहीत होते हैं।

### संवेदनशील जानकारी वाली फाइल्स

MacOS पासवर्ड्स जैसी जानकारी कई जगहों पर स्टोर करता है:

{% content-ref url="macos-sensitive-locations.md" %}
[macos-sensitive-locations.md](macos-sensitive-locations.md)
{% endcontent-ref %}

### कमजोर pkg इंस्टॉलर्स

{% content-ref url="macos-installers-abuse.md" %}
[macos-installers-abuse.md](macos-installers-abuse.md)
{% endcontent-ref %}

## OS X विशिष्ट एक्सटेंशन्स

* **`.dmg`**: Apple Disk Image फाइल्स इंस्टॉलर्स के लिए बहुत आम हैं।
* **`.kext`**: इसे विशिष्ट संरचना का पालन करना चाहिए और यह OS X का ड्राइवर संस्करण है। (यह एक बंडल है)
* **`.plist`**: जिसे प्रॉपर्टी लिस्ट भी कहा जाता है, XML या बाइनरी प्रारूप में जानकारी स्टोर करता है।
* XML या बाइनरी हो सकता है। बाइनरी वाले को पढ़ा जा सकता है:
* `defaults read config.plist`
* `/usr/libexec/PlistBuddy -c print config.plsit`
* `plutil -p ~/Library/Preferences/com.apple.screensaver.plist`
* `plutil -convert xml1 ~/Library/Preferences/com.apple.screensaver.plist -o -`
* `plutil -convert json ~/Library/Preferences/com.apple.screensaver.plist -o -`
* **`.app`**: Apple एप्लिकेशन्स जो डायरेक्टरी संरचना का पालन करते हैं (यह एक बंडल है)।
* **`.dylib`**: डायनामिक लाइब्रेरीज़ (विंडोज DLL फाइल्स की तरह)
* **`.pkg`**: xar (eXtensible Archive format) के समान हैं। इंस्टॉलर कमांड का इस्तेमाल इन फाइल्स की सामग्री को इंस्टॉल करने के लिए किया जा सकता है।
* **`.DS_Store`**: यह फाइल प्रत्येक डायरेक्टरी में होती है, यह डायरेक्टरी के गुणों और कस्टमाइजेशन को सेव करती है।
* **`.Spotlight-V100`**: यह फोल्डर सिस्टम पर हर वॉल्यूम की रूट डायरेक्टरी पर दिखाई देता है।
* **`.metadata_never_index`**: अगर यह फाइल किसी वॉल्यूम की रूट पर है तो Spotlight उस वॉल्यूम को इंडेक्स नहीं करेगा।
* **`.noindex`**: इस एक्सटेंशन वाली फाइल्स और फोल्डर्स को Spotlight द्वारा इंडेक्स नहीं किया जाएगा।

### macOS बंडल्स

मूल रूप से, एक बंडल फाइल सिस्टम के भीतर एक **डायरेक्टरी संरचना** है। दिलचस्प बात यह है कि डिफ़ॉल्ट रूप से यह डायरेक्टरी Finder में एक एकल ऑब्जेक्ट की तरह दिखती है (जैसे `.app`).&#x20;

{% content-ref url="macos-bundles.md" %}
[macos-bundles.md](macos-bundles.md)
{% endcontent-ref %}

## Dyld Shared Cache

macOS (और iOS) पर सभी सिस्टम शेयर्ड लाइब्रेरीज़, जैसे कि frameworks और dylibs, एक एकल फाइल में **संयुक्त किए जाते हैं**, जिसे **dyld shared cache** कहा जाता है। यह प्रदर्शन में सुधार करता है, क्योंकि कोड को तेजी से लोड किया जा सकता है।

dyld shared cache के समान, कर्नेल और कर्नेल एक
```bash
# dyld_shared_cache_util
dyld_shared_cache_util -extract ~/shared_cache/ /System/Volumes/Preboot/Cryptexes/OS/System/Library/dyld/dyld_shared_cache_arm64e

# dyldextractor
dyldex -l [dyld_shared_cache_path] # List libraries
dyldex_all [dyld_shared_cache_path] # Extract all
# More options inside the readme
```
{% endcode %}

पुराने संस्करणों में आप **`/System/Library/dyld/`** में **साझा कैश** पा सकते हैं।

iOS में आप उन्हें **`/System/Library/Caches/com.apple.dyld/`** में पा सकते हैं।

{% hint style="success" %}
ध्यान दें कि यदि `dyld_shared_cache_util` उपकरण काम नहीं करता है, तो आप **साझा dyld बाइनरी को Hopper को पास कर सकते हैं** और Hopper सभी लाइब्रेरीज की पहचान कर सकता है और आपको **चुनने देगा कि** आप किसकी जांच करना चाहते हैं:
{% endhint %}

<figure><img src="../../../.gitbook/assets/image (680).png" alt="" width="563"><figcaption></figcaption></figure>

## विशेष फाइल अनुमतियाँ

### फ़ोल्डर अनुमतियाँ

एक **फ़ोल्डर** में, **पढ़ने** की अनुमति से **सूचीबद्ध करने** की अनुमति मिलती है, **लिखने** की अनुमति से **हटाने** और फ़ोल्डर में फाइलों को **लिखने** की अनुमति मिलती है, और **निष्पादित** करने की अनुमति से डायरेक्टरी को **पार करने** की अनुमति मिलती है। इसलिए, उदाहरण के लिए, एक उपयोगकर्ता जिसके पास फ़ाइल के ऊपर **पढ़ने की अनुमति** है लेकिन वह डायरेक्टरी में **निष्पादित की अनुमति नहीं है**, वह फ़ाइल को **पढ़ नहीं पाएगा**।

### फ्लैग संशोधक

कुछ फ्लैग होते हैं जो फाइलों में सेट किए जा सकते हैं जिससे फाइल अलग तरह से व्यवहार करेगी। आप डायरेक्टरी के अंदर फाइलों के फ्लैग्स की **जांच** `ls -lO /path/directory` के साथ कर सकते हैं

* **`uchg`**: **uchange** फ्लैग के रूप में जाना जाता है जो **किसी भी कार्रवाई को रोकेगा** जो फाइल को बदलने या हटाने से संबंधित है। इसे सेट करने के लिए करें: `chflags uchg file.txt`
* रूट उपयोगकर्ता फ्लैग को **हटा सकता है** और फाइल को संशोधित कर सकता है
* **`restricted`**: यह फ्लैग फाइल को SIP द्वारा **संरक्षित** करता है (आप इस फ्लैग को फाइल में नहीं जोड़ सकते)।
* **`Sticky bit`**: यदि किसी डायरेक्टरी में स्टिकी बिट होता है, तो **केवल** डायरेक्टरी का मालिक या रूट ही फाइलों का नाम बदल सकता है या उन्हें हटा सकता है। आमतौर पर यह /tmp डायरेक्टरी पर सेट किया जाता है ताकि सामान्य उपयोगकर्ता अन्य उपयोगकर्ताओं की फाइलों को हटाने या स्थानांतरित करने से रोक सकें।

### **फाइल ACLs**

फाइल **ACLs** में **ACE** (एक्सेस कंट्रोल एंट्रीज) होते हैं जहां अलग-अलग उपयोगकर्ताओं को अधिक **विस्तृत अनुमतियाँ** दी जा सकती हैं।

एक **डायरेक्टरी** को ये अनुमतियाँ दी जा सकती हैं: `list`, `search`, `add_file`, `add_subdirectory`, `delete_child`, `delete_child`.\
और एक **फाइल** को: `read`, `write`, `append`, `execute`.

जब फाइल में ACLs होते हैं तो आप **अनुमतियों की सूची में "+" पाएंगे जैसे कि**:
```bash
ls -ld Movies
drwx------+   7 username  staff     224 15 Apr 19:42 Movies
```
आप फ़ाइल के **ACLs पढ़ सकते हैं** इसके साथ:
```bash
ls -lde Movies
drwx------+ 7 username  staff  224 15 Apr 19:42 Movies
0: group:everyone deny delete
```
आप **सभी फाइलें जिनमें ACLs हैं** इसके साथ खोज सकते हैं (यह बहुत धीमा है):
```bash
ls -RAle / 2>/dev/null | grep -E -B1 "\d: "
```
### रिसोर्स फोर्क्स | macOS ADS

यह **macOS मशीनों में अल्टरनेट डेटा स्ट्रीम्स प्राप्त करने** का एक तरीका है। आप **com.apple.ResourceFork** नामक एक्सटेंडेड एट्रिब्यूट के अंदर **file/..namedfork/rsrc** में सेव करके फाइल के अंदर सामग्री सेव कर सकते हैं।
```bash
echo "Hello" > a.txt
echo "Hello Mac ADS" > a.txt/..namedfork/rsrc

xattr -l a.txt #Read extended attributes
com.apple.ResourceFork: Hello Mac ADS

ls -l a.txt #The file length is still q
-rw-r--r--@ 1 username  wheel  6 17 Jul 01:15 a.txt
```
आप इस विस्तारित विशेषता वाली सभी फाइलों को **इसके साथ खोज सकते हैं**:

{% code overflow="wrap" %}
```bash
find / -type f -exec ls -ld {} \; 2>/dev/null | grep -E "[x\-]@ " | awk '{printf $9; printf "\n"}' | xargs -I {} xattr -lv {} | grep "com.apple.ResourceFork"
```
{% endcode %}

## **यूनिवर्सल बाइनरीज़ &** Mach-o प्रारूप

Mac OS बाइनरीज़ आमतौर पर **यूनिवर्सल बाइनरीज़** के रूप में संकलित की जाती हैं। एक **यूनिवर्सल बाइनरी** एक ही फ़ाइल में **कई आर्किटेक्चर्स का समर्थन कर सकती है**।

{% content-ref url="universal-binaries-and-mach-o-format.md" %}
[universal-binaries-and-mach-o-format.md](universal-binaries-and-mach-o-format.md)
{% endcontent-ref %}

## macOS मेमोरी डंपिंग

{% content-ref url="macos-memory-dumping.md" %}
[macos-memory-dumping.md](macos-memory-dumping.md)
{% endcontent-ref %}

## जोखिम श्रेणी फ़ाइलें Mac OS

फ़ाइल `/System/Library/CoreServices/CoreTypes.bundle/Contents/Resources/System` में फ़ाइल एक्सटेंशन के आधार पर फ़ाइलों से जुड़े जोखिम शामिल होते हैं।

संभावित श्रेणियाँ निम्नलिखित हैं:

* **LSRiskCategorySafe**: **पूरी तरह से** **सुरक्षित**; Safari डाउनलोड के बाद स्वतः खुलेगा
* **LSRiskCategoryNeutral**: कोई चेतावनी नहीं, लेकिन **स्वतः नहीं खुलेगा**
* **LSRiskCategoryUnsafeExecutable**: **चेतावनी ट्रिगर** करता है “यह फ़ाइल एक एप्लिकेशन है...”
* **LSRiskCategoryMayContainUnsafeExecutable**: यह उन आर्काइव्स के लिए है जिनमें एक एक्ज़ीक्यूटेबल होता है। यह **चेतावनी ट्रिगर करता है जब तक कि Safari सभी सामग्री को सुरक्षित या तटस्थ निर्धारित नहीं कर लेता**।

## लॉग फ़ाइलें

* **`$HOME/Library/Preferences/com.apple.LaunchServices.QuarantineEventsV2`**: डाउनलोड की गई फ़ाइलों के बारे में जानकारी रखता है, जैसे कि वे कहाँ से डाउनलोड की गई थीं।
* **`/var/log/system.log`**: OSX सिस्टम्स का मुख्य लॉग। com.apple.syslogd.plist syslogging के निष्पादन के लिए जिम्मेदार है (आप `launchctl list` में "com.apple.syslogd" को देखकर जांच सकते हैं कि यह निष्क्रिय है या नहीं)।
* **`/private/var/log/asl/*.asl`**: ये Apple सिस्टम लॉग्स हैं जिनमें दिलचस्प जानकारी हो सकती है।
* **`$HOME/Library/Preferences/com.apple.recentitems.plist`**: "Finder" के माध्यम से हाल ही में एक्सेस की गई फ़ाइलों और एप्लिकेशनों को स्टोर करता है।
* **`$HOME/Library/Preferences/com.apple.loginitems.plsit`**: सिस्टम स्टार्टअप पर लॉन्च करने के लिए आइटम्स को स्टोर करता है।
* **`$HOME/Library/Logs/DiskUtility.log`**: DiskUtility ऐप के लिए लॉग फ़ाइल (ड्राइव्स के बारे में जानकारी, यूएसबी सहित)।
* **`/Library/Preferences/SystemConfiguration/com.apple.airport.preferences.plist`**: वायरलेस एक्सेस पॉइंट्स के बारे में डेटा।
* **`/private/var/db/launchd.db/com.apple.launchd/overrides.plist`**: निष्क्रिय किए गए डेमन्स की सूची।

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert) के साथ शून्य से हीरो तक AWS हैकिंग सीखें</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप HackTricks में अपनी **कंपनी का विज्ञापन देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें।
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह।
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में शामिल हों या मुझे **Twitter** 🐦 पर **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>
