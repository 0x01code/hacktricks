# macOS SIP

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert) के साथ शून्य से नायक तक AWS हैकिंग सीखें</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग प्राप्त करें**](https://peass.creator-spring.com)
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह में शामिल हों**](https://discord.gg/hRep4RUj7f) या [**telegram समूह**](https://t.me/peass) में शामिल हों या मुझे **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) पर **फॉलो करें**.
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग तरकीबें साझा करें.

</details>

## **मूल जानकारी**

**System Integrity Protection (SIP)** macOS में एक सुरक्षा प्रौद्योगिकी है जो अनधिकृत पहुँच से कुछ सिस्टम डायरेक्टरीज़ की रक्षा करती है, यहाँ तक कि रूट यूज़र के लिए भी। यह इन डायरेक्टरीज़ में फाइलों के निर्माण, परिवर्तन, या हटाने सहित संशोधनों को रोकता है। SIP द्वारा संरक्षित मुख्य डायरेक्टरीज़ हैं:

* **/System**
* **/bin**
* **/sbin**
* **/usr**

इन डायरेक्टरीज़ और उनकी सबडायरेक्टरीज़ के लिए सुरक्षा नियम **`/System/Library/Sandbox/rootless.conf`** फाइल में निर्दिष्ट हैं। इस फाइल में, एक तारांकन (\*) के साथ शुरू होने वाले पथ SIP की प्रतिबंधों के अपवाद को दर्शाते हैं।

उदाहरण के लिए, निम्नलिखित कॉन्फ़िगरेशन:
```javascript
/usr
* /usr/libexec/cups
* /usr/local
* /usr/share/man
```
यह दर्शाता है कि **`/usr`** निर्देशिका आम तौर पर SIP द्वारा संरक्षित होती है। हालांकि, तीन उपनिर्देशिकाओं में संशोधन की अनुमति है (`/usr/libexec/cups`, `/usr/local`, और `/usr/share/man`), क्योंकि वे एक अग्रणी तारांकन (\*) के साथ सूचीबद्ध हैं।

SIP द्वारा किसी निर्देशिका या फ़ाइल की सुरक्षा की जांच करने के लिए, आप **`ls -lOd`** कमांड का उपयोग करके **`restricted`** या **`sunlnk`** फ्लैग की उपस्थिति की जांच कर सकते हैं। उदाहरण के लिए:
```bash
ls -lOd /usr/libexec/cups
drwxr-xr-x  11 root  wheel  sunlnk 352 May 13 00:29 /usr/libexec/cups
```
इस मामले में, **`sunlnk`** फ्लैग का तात्पर्य है कि `/usr/libexec/cups` डायरेक्टरी स्वयं **हटाई नहीं जा सकती**, हालांकि इसके अंदर की फाइलें बनाई, संशोधित, या हटाई जा सकती हैं।

दूसरी ओर:
```bash
ls -lOd /usr/libexec
drwxr-xr-x  338 root  wheel  restricted 10816 May 13 00:29 /usr/libexec
```
यहाँ, **`restricted`** फ्लैग दर्शाता है कि `/usr/libexec` डायरेक्टरी SIP द्वारा संरक्षित है। SIP-संरक्षित डायरेक्टरी में, फाइलों को बनाया, संशोधित, या हटाया नहीं जा सकता है।

इसके अलावा, यदि किसी फाइल में **`com.apple.rootless`** विस्तारित **एट्रिब्यूट** होता है, तो वह फाइल भी **SIP द्वारा संरक्षित** होगी।

**SIP अन्य रूट क्रियाओं को भी सीमित करता है** जैसे:

* अविश्वसनीय कर्नेल एक्सटेंशन्स को लोड करना
* Apple-हस्ताक्षरित प्रक्रियाओं के लिए टास्क-पोर्ट्स प्राप्त करना
* NVRAM वेरिएबल्स में संशोधन करना
* कर्नेल डिबगिंग की अनुमति देना

विकल्प nvram वेरिएबल में एक बिटफ्लैग के रूप में रखे जाते हैं (`csr-active-config` Intel पर और `lp-sip0` ARM के लिए बूटेड डिवाइस ट्री से पढ़ा जाता है)। आप फ्लैग्स को XNU सोर्स कोड में `csr.sh` में देख सकते हैं:

<figure><img src="../../../.gitbook/assets/image (720).png" alt=""><figcaption></figcaption></figure>

### SIP स्थिति

आप निम्नलिखित कमांड के साथ जांच सकते हैं कि आपके सिस्टम पर SIP सक्षम है या नहीं:
```bash
csrutil status
```
यदि आपको SIP अक्षम करने की आवश्यकता है, तो आपको अपने कंप्यूटर को रिकवरी मोड में पुनः आरंभ करना होगा (स्टार्टअप के दौरान Command+R दबाकर), फिर निम्नलिखित कमांड को निष्पादित करें:
```bash
csrutil disable
```
यदि आप SIP सक्षम रखना चाहते हैं लेकिन डिबगिंग सुरक्षा को हटाना चाहते हैं, तो आप इसे निम्नलिखित के साथ कर सकते हैं:
```bash
csrutil enable --without debug
```
### अन्य प्रतिबंध

SIP कई अन्य प्रतिबंध भी लगाता है। उदाहरण के लिए, यह **unsigned kernel extensions** (kexts) को लोड करने की अनुमति नहीं देता और macOS सिस्टम प्रोसेसेस की **debugging** को रोकता है। यह dtrace जैसे टूल्स को सिस्टम प्रोसेसेस की जांच करने से भी रोकता है।

[इस टॉक में और SIP जानकारी](https://www.slideshare.net/i0n1c/syscan360-stefan-esser-os-x-el-capitan-sinking-the-ship).

## SIP Bypasses

यदि कोई हमलावर SIP को बायपास करने में सफल होता है, तो वह यह कर सकेगा:

* सभी उपयोगकर्ताओं के मेल, संदेश, Safari इतिहास... पढ़ें
* वेबकैम, माइक्रोफोन या कुछ भी के लिए अनुमतियां प्रदान करें (SIP संरक्षित TCC डेटाबेस पर सीधे लिखकर) - TCC bypass
* Persistence: वह मैलवेयर को SIP संरक्षित स्थान पर सहेज सकता है और toot भी इसे हटा नहीं पाएगा। साथ ही वह MRT के साथ छेड़छाड़ कर सकता है।
* Kernel extensions लोड करने की सुविधा (इसके लिए अन्य कठोर सुरक्षा भी मौजूद हैं)।

### Installer Packages

**Apple के प्रमाणपत्र के साथ हस्ताक्षरित Installer packages** इसकी सुरक्षा को बायपास कर सकते हैं। इसका मतलब है कि मानक डेवलपर्स द्वारा हस्ताक्षरित पैकेज भी ब्लॉक किए जाएंगे यदि वे SIP-संरक्षित निर्देशिकाओं में संशोधन करने का प्रयास करते हैं।

### Inexistent SIP file

एक संभावित छेद यह है कि यदि कोई फ़ाइल **`rootless.conf` में निर्दिष्ट है लेकिन वर्तमान में मौजूद नहीं है**, तो इसे बनाया जा सकता है। मैलवेयर इसका फायदा उठाकर सिस्टम पर **persistence स्थापित** कर सकता है। उदाहरण के लिए, एक दुर्भावनापूर्ण प्रोग्राम `/System/Library/LaunchDaemons` में एक .plist फ़ाइल बना सकता है यदि यह `rootless.conf` में सूचीबद्ध है लेकिन मौजूद नहीं है।

### com.apple.rootless.install.heritable

{% hint style="danger" %}
प्रतिबंध **`com.apple.rootless.install.heritable`** SIP को बायपास करने की अनुमति देता है
{% endhint %}

#### Shrootless

[**इस ब्लॉग पोस्ट के शोधकर्ताओं ने**](https://www.microsoft.com/en-us/security/blog/2021/10/28/microsoft-finds-new-macos-vulnerability-shrootless-that-could-bypass-system-integrity-protection/) macOS के System Integrity Protection (SIP) तंत्र में एक कमजोरी की खोज की, जिसे 'Shrootless' कमजोरी कहा गया। यह कमजोरी **`system_installd`** डेमॉन के आसपास केंद्रित है, जिसमें एक प्रतिबंध, **`com.apple.rootless.install.heritable`**, होता है जो इसके चाइल्ड प्रोसेसेस को SIP के फाइल सिस्टम प्रतिबंधों को बायपास करने की अनुमति देता है।

**`system_installd`** डेमॉन **Apple** द्वारा हस्ताक्षरित पैकेजों को इंस्टॉल करेगा।

शोधकर्ताओं ने पाया कि Apple-हस्ताक्षरित पैकेज (.pkg फ़ाइल) की स्थापना के दौरान, **`system_installd`** पैकेज में शामिल किसी भी **post-install** स्क्रिप्ट को **चलाता है**। ये स्क्रिप्ट डिफ़ॉल्ट शेल, **`zsh`** द्वारा निष्पादित की जाती हैं, जो गैर-इंटरैक्टिव मोड में भी, यदि **`/etc/zshenv`** फ़ाइल मौजूद है, तो इससे स्वचालित रूप से **चलाता है**। इस व्यवहार का हमलावरों द्वारा शोषण किया जा सकता है: एक दुर्भावनापूर्ण `/etc/zshenv` फ़ाइल बनाकर और **`system_installd` के `zsh` को आमंत्रित करने का इंतजार करके**, वे डिवाइस पर मनमाने ऑपरेशन कर सकते हैं।

इसके अलावा, यह पता चला कि **`/etc/zshenv` का उपयोग एक सामान्य हमला तकनीक के रूप में किया जा सकता है**, केवल SIP बायपास के लिए नहीं। प्रत्येक उपयोगकर्ता प्रोफ़ाइल में एक `~/.zshenv` फ़ाइल होती है, जो `/etc/zshenv` की तरह ही व्यवहार करती है लेकिन इसके लिए रूट अनुमतियां की आवश्यकता नहीं होती। यह फ़ाइल `zsh` शुरू होने पर हर बार ट्रिगर होने वाले पर्सिस्टेंस मैकेनिज़्म के रूप में उपयोग की जा सकती है, या एक प्रिविलेज एलिवेशन मैकेनिज़्म के रूप में। यदि एक एडमिन उपयोगकर्ता `sudo -s` या `sudo <command>` का उपयोग करके रूट में एलिवेट करता है, तो `~/.zshenv` फ़ाइल ट्रिगर हो जाएगी, प्रभावी रूप से रूट में एलिवेट हो जाएगी।

#### [**CVE-2022-22583**](https://perception-point.io/blog/technical-analysis-cve-2022-22583/)

[**CVE-2022-22583**](https://perception-point.io/blog/technical-analysis-cve-2022-22583/) में यह पता चला कि वही **`system_installd`** प्रक्रिया अभी भी दुरुपयोग की जा सकती है क्योंकि यह **post-install script को `/tmp` के अंदर SIP द्वारा संरक्षित एक यादृच्छिक नाम वाले फ़ोल्डर के अंदर रख रही थी**। बात यह है कि **`/tmp` खुद SIP द्वारा संरक्षित नहीं है**, इसलिए यह संभव था कि **वर्चुअल इमेज को इस पर माउंट** किया जाए, फिर **installer** वहां पर **post-install script** डालेगा, **वर्चुअल इमेज को अनमाउंट** करेगा, सभी **फ़ोल्डर्स को पुनः बनाएगा** और **post installation** स्क्रिप्ट को **payload** के साथ जोड़ देगा जिसे निष्पादित किया जाना है।

#### [fsck\_cs utility](https://www.theregister.com/2016/03/30/apple_os_x_rootless/)

बायपास ने इस तथ्य का शोषण किया कि **`fsck_cs`** **symbolic links** का अनुसरण करेगा और उसे प्रस्तुत किए गए फाइलसिस्टम को ठीक करने का प्रयास करेगा।

इसलिए, एक हमलावर _`/dev/diskX`_ से `/System/Library/Extensions/AppleKextExcludeList.kext/Contents/Info.plist` की ओर एक symbolic link बना सकता है और पूर्व पर **`fsck_cs`** को आमंत्रित कर सकता है। जैसे ही `Info.plist` फ़ाइल भ्रष्ट हो जाती है, ऑपरेटिंग सिस्टम अब **kernel extension exclusions को नियंत्रित नहीं कर पाएगा**, इस प्रकार SIP को बायपास कर देगा।

{% code overflow="wrap" %}
```bash
ln -s /System/Library/Extensions/AppleKextExcludeList.kext/Contents/Info.plist /dev/diskX
fsck_cs /dev/diskX 1>&-
touch /Library/Extensions/
reboot
```
{% endcode %}

उपरोक्त Info.plist फ़ाइल, जो अब नष्ट हो चुकी है, **SIP द्वारा कुछ कर्नेल एक्सटेंशन्स को व्हाइटलिस्ट करने** और विशेष रूप से **अन्यों को लोड होने से रोकने** के लिए प्रयोग की जाती है। यह सामान्यतः Apple के अपने कर्नेल एक्सटेंशन **`AppleHWAccess.kext`** को ब्लैकलिस्ट करती है, परंतु कॉन्फ़िगरेशन फ़ाइल के नष्ट हो जाने के बाद, हम इसे लोड कर सकते हैं और सिस्टम RAM में जैसे चाहें पढ़ने और लिखने के लिए इसका उपयोग कर सकते हैं।

#### [SIP संरक्षित फ़ोल्डर्स पर माउंट करना](https://www.slideshare.net/i0n1c/syscan360-stefan-esser-os-x-el-capitan-sinking-the-ship)

**SIP संरक्षित फ़ोल्डर्स पर नई फ़ाइल सिस्टम को माउंट करके सुरक्षा को बायपास करना** संभव था।
```bash
mkdir evil
# Add contento to the folder
hdiutil create -srcfolder evil evil.dmg
hdiutil attach -mountpoint /System/Library/Snadbox/ evil.dmg
```
#### [अपग्रेडर बायपास (2016)](https://objective-see.org/blog/blog\_0x14.html)

जब निष्पादित किया जाता है, तो अपग्रेड/इंस्टॉलर एप्लिकेशन (उदाहरण के लिए `Install macOS Sierra.app`) सिस्टम को एक इंस्टॉलर डिस्क इमेज से बूट करने के लिए सेटअप करता है (जो डाउनलोड किए गए एप्लिकेशन के भीतर एम्बेडेड होता है)। यह इंस्टॉलर डिस्क इमेज OS को अपग्रेड करने के लॉजिक को समाहित करता है, उदाहरण के लिए OS X El Capitan से macOS Sierra तक।

अपग्रेड/इंस्टॉलर इमेज (`InstallESD.dmg`) से सिस्टम को बूट करने के लिए, `Install macOS Sierra.app` **`bless`** उपयोगिता का उपयोग करता है (जो एंटाइटलमेंट `com.apple.rootless.install.heritable` को विरासत में प्राप्त करता है):

{% code overflow="wrap" %}
```bash
/usr/sbin/bless -setBoot -folder /Volumes/Macintosh HD/macOS Install Data -bootefi /Volumes/Macintosh HD/macOS Install Data/boot.efi -options config="\macOS Install Data\com.apple.Boot" -label macOS Installer
```
{% endcode %}

इसलिए, यदि एक हमलावर अपग्रेड इमेज (`InstallESD.dmg`) को सिस्टम बूट होने से पहले मॉडिफाई कर सकता है, तो वह SIP को बायपास कर सकता है।

इमेज को मॉडिफाई करने का तरीका यह था कि एक डायनामिक लोडर (dyld) को बदल दिया जाए जो नासमझी से मैलिशस dylib को लोड और एक्जीक्यूट करेगा, जैसे कि **`libBaseIA`** dylib। इसलिए, जब भी इंस्टॉलर एप्लिकेशन को उपयोगकर्ता द्वारा शुरू किया जाता है (अर्थात् सिस्टम को अपग्रेड करने के लिए) हमारा मैलिशस dylib (जिसका नाम libBaseIA.dylib है) भी इंस्टॉलर में लोड और एक्जीक्यूट होगा।

अब 'अंदर' इंस्टॉलर एप्लिकेशन में, हम अपग्रेड प्रक्रिया के इस चरण को नियंत्रित कर सकते हैं। चूंकि इंस्टॉलर इमेज को 'ब्लेस' करेगा, हमें बस इतना करना है कि इसका उपयोग होने से पहले इमेज, **`InstallESD.dmg`**, को सबवर्ट कर दें। यह **`extractBootBits`** मेथड को मेथड स्विजलिंग के साथ हुक करके संभव था।\
मैलिशस कोड को डिस्क इमेज के उपयोग से ठीक पहले एक्जीक्यूट करने के बाद, इसे इन्फेक्ट करने का समय है।

`InstallESD.dmg` के अंदर एक और एम्बेडेड डिस्क इमेज `BaseSystem.dmg` है जो अपग्रेड कोड की 'रूट फाइल-सिस्टम' है। यह संभव था कि `BaseSystem.dmg` में एक डायनामिक लाइब्रेरी इंजेक्ट की जाए ताकि मैलिशस कोड OS-लेवल फाइलों को मॉडिफाई करने में सक्षम प्रोसेस के संदर्भ में चल रहा हो।

#### [systemmigrationd (2023)](https://www.youtube.com/watch?v=zxZesAN-TEk)

[**DEF CON 31**](https://www.youtube.com/watch?v=zxZesAN-TEk) से इस टॉक में दिखाया गया है कि कैसे **`systemmigrationd`** (जो SIP को बायपास कर सकता है) एक **bash** और एक **perl** स्क्रिप्ट को एक्जीक्यूट करता है, जिसे env वेरिएबल्स **`BASH_ENV`** और **`PERL5OPT`** के माध्यम से दुरुपयोग किया जा सकता है।

### **com.apple.rootless.install**

{% hint style="danger" %}
एंटाइटलमेंट **`com.apple.rootless.install`** SIP को बायपास करने की अनुमति देता है
{% endhint %}

[**CVE-2022-26712**](https://jhftss.github.io/CVE-2022-26712-The-POC-For-SIP-Bypass-Is-Even-Tweetable/) से, सिस्टम XPC सर्विस `/System/Library/PrivateFrameworks/ShoveService.framework/Versions/A/XPCServices/SystemShoveService.xpc` में एंटाइटलमेंट **`com.apple.rootless.install`** है, जो प्रोसेस को SIP प्रतिबंधों को बायपास करने की अनुमति देता है। यह **किसी भी सुरक्षा जांच के बिना फाइलों को मूव करने का एक तरीका भी प्रदर्शित करता है।**

## Sealed System Snapshots

Sealed System Snapshots एक फीचर है जिसे Apple ने **macOS Big Sur (macOS 11)** में पेश किया था, जो अतिरिक्त सुरक्षा और सिस्टम स्थिरता प्रदान करने के लिए **System Integrity Protection (SIP)** तंत्र का हिस्सा है। ये मूल रूप से सिस्टम वॉल्यूम के रीड-ओनली संस्करण हैं।

यहाँ एक विस्तृत नज़र है:

1. **Immutable System**: Sealed System Snapshots macOS सिस्टम वॉल्यूम को "immutable" बनाते हैं, यानी कि इसे मॉडिफाई नहीं किया जा सकता। यह सिस्टम में किसी भी अनधिकृत या आकस्मिक परिवर्तनों को रोकता है जो सुरक्षा या सिस्टम स्थिरता को समझौता कर सकते हैं।
2. **System Software Updates**: जब आप macOS अपडेट्स या अपग्रेड्स इंस्टॉल करते हैं, macOS एक नया सिस्टम स्नैपशॉट बनाता है। macOS स्टार्टअप वॉल्यूम फिर **APFS (Apple File System)** का उपयोग करके इस नए स्नैपशॉट पर स्विच करता है। अपडेट्स लागू करने की पूरी प्रक्रिया सुरक्षित और अधिक विश्वसनीय हो जाती है क्योंकि सिस्टम हमेशा अपडेट के दौरान कुछ गलत होने पर पिछले स्नैपशॉट पर वापस जा सकता है।
3. **Data Separation**: macOS Catalina में पेश किए गए डेटा और सिस्टम वॉल्यूम सेपरेशन की अवधारणा के साथ, Sealed System Snapshot फीचर सुनिश्चित करता है कि आपका सारा डेटा और सेटिंग्स एक अलग "**Data**" वॉल्यूम पर संग्रहीत होते हैं। यह अलगाव आपके डेटा को सिस्टम से स्वतंत्र बनाता है, जो सिस्टम अपडेट्स की प्रक्रिया को सरल बनाता है और सिस्टम सुरक्षा को बढ़ाता है।

याद रखें कि ये स्नैपशॉट्स macOS द्वारा स्वचालित रूप से प्रबंधित किए जाते हैं और APFS की स्पेस शेयरिंग क्षमताओं के कारण आपकी डिस्क पर अतिरिक्त स्थान नहीं लेते हैं। यह भी महत्वपूर्ण है कि ये स्नैपशॉट्स **Time Machine स्नैपशॉट्स** से अलग हैं, जो पूरे सिस्टम के यूजर-एक्सेसिबल बैकअप्स हैं।

### Check Snapshots

कमांड **`diskutil apfs list`** APFS वॉल्यूम्स के **विवरणों और उनके लेआउट** को सूचीबद्ध करता है:

<pre><code>+-- Container disk3 966B902E-EDBA-4775-B743-CF97A0556A13
|   ====================================================
|   APFS Container Reference:     disk3
|   Size (Capacity Ceiling):      494384795648 B (494.4 GB)
|   Capacity In Use By Volumes:   219214536704 B (219.2 GB) (44.3% used)
|   Capacity Not Allocated:       275170258944 B (275.2 GB) (55.7% free)
|   |
|   +-&#x3C; Physical Store disk0s2 86D4B7EC-6FA5-4042-93A7-D3766A222EBE
|   |   -----------------------------------------------------------
|   |   APFS Physical Store Disk:   disk0s2
|   |   Size:                       494384795648 B (494.4 GB)
|   |
|   +-> Volume disk3s1 7A27E734-880F-4D91-A703-FB55861D49B7
|   |   ---------------------------------------------------
<strong>|   |   APFS Volume Disk (Role):   disk3s1 (System)
</strong>|   |   Name:                      Macintosh HD (Case-insensitive)
<strong>|   |   Mount Point:               /System/Volumes/Update/mnt1
</strong>|   |   Capacity Consumed:         12819210240 B (12.8 GB)
|   |   Sealed:                    Broken
|   |   FileVault:                 Yes (Unlocked)
|   |   Encrypted:                 No
|   |   |
|   |   Snapshot:                  FAA23E0C-791C-43FF-B0E7-0E1C0810AC61
|   |   Snapshot Disk:             disk3s1s1
<strong>|   |   Snapshot Mount Point:      /
</strong><strong>|   |   Snapshot Sealed:           Yes
</strong>[...]
+-> Volume disk3s5 281959B7-07A1-4940-BDDF-6419360F3327
|   ---------------------------------------------------
|   APFS Volume Disk (Role):   disk3s5 (Data)
|   Name:                      Macintosh HD - Data (Case-insensitive)
<strong>    |   Mount Point:               /System/Volumes/Data
</strong><strong>    |   Capacity Consumed:         412071784448 B (412.1 GB)
</strong>    |   Sealed:                    No
|   FileVault:                 Yes (Unlocked)
</code></pre>

पिछले आउटपुट में यह संभव है कि **यूजर-एक्सेसिबल लोकेशन्स** `/System/Volumes/Data` के तहत माउंट किए गए हैं।

इसके अलावा, **macOS सिस्टम वॉल्यूम स्नैपशॉट** `/` में माउंट किया गया है और यह **सील्ड** है (OS द्वारा क्रिप्टोग्राफिकली साइन किया गया)। इसलिए, अगर SIP बायपास हो जाता है और इसे मॉडिफाई करता है, तो **OS अब बूट नहीं होगा**।

यह भी संभव है कि **सील सक्षम है यह वेरिफाई करने के लिए** चलाएं:
```bash
csrutil authenticated-root status
Authenticated Root status: enabled
```
इसके अलावा, स्नैपशॉट डिस्क को **read-only** के रूप में भी माउंट किया जाता है:
```
mount
/dev/disk3s1s1 on / (apfs, sealed, local, read-only, journaled)
```
<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग प्राप्त करें**](https://peass.creator-spring.com)
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स शेयर करें।

</details>
