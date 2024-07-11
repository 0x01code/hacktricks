# macOS Apps - जांच, डीबगिंग और फजिंग

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में** देखना चाहते हैं या **PDF में HackTricks डाउनलोड** करना चाहते हैं तो [**सब्सक्रिप्शन प्लान**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें और PRs सबमिट करें** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

### [WhiteIntel](https://whiteintel.io)

<figure><img src="../../../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) एक **डार्क-वेब** पर आधारित खोज इंजन है जो **स्टीलर मैलवेयर** द्वारा कंपनी या उसके ग्राहकों को **कंप्रोमाइज** होने की मुफ्त सुविधाएं प्रदान करता है।

WhiteIntel का मुख्य उद्देश्य खाता हासिल करने और जानकारी चोरी मैलवेयर से होने वाले रैंसमवेयर हमलों का मुकाबला करना है।

आप उनकी वेबसाइट चेक कर सकते हैं और उनका इंजन **मुफ्त** में आज़मा सकते हैं:

{% embed url="https://whiteintel.io" %}

***

## स्थैतिक विश्लेषण

### otool & objdump & nm
```bash
otool -L /bin/ls #List dynamically linked libraries
otool -tv /bin/ps #Decompile application
```
{% code overflow="wrap" %}
```bash
objdump -m --dylibs-used /bin/ls #List dynamically linked libraries
objdump -m -h /bin/ls # Get headers information
objdump -m --syms /bin/ls # Check if the symbol table exists to get function names
objdump -m --full-contents /bin/ls # Dump every section
objdump -d /bin/ls # Dissasemble the binary
objdump --disassemble-symbols=_hello --x86-asm-syntax=intel toolsdemo #Disassemble a function using intel flavour
```
{% endcode %}
```bash
nm -m ./tccd # List of symbols
```
### jtool2 और Disarm

आप यहाँ से [**disarm डाउनलोड कर सकते हैं**](https://newosxbook.com/tools/disarm.html)।
```bash
ARCH=arm64e disarm -c -i -I --signature /path/bin # Get bin info and signature
ARCH=arm64e disarm -c -l /path/bin # Get binary sections
ARCH=arm64e disarm -c -L /path/bin # Get binary commands (dependencies included)
ARCH=arm64e disarm -c -S /path/bin # Get symbols (func names, strings...)
ARCH=arm64e disarm -c -d /path/bin # Get disasembled
jtool2 -d __DATA.__const myipc_server | grep MIG # Get MIG info
```
आप यहाँ [**jtool2 डाउनलोड कर सकते हैं**](http://www.newosxbook.com/tools/jtool.html) या `brew` के साथ इंस्टॉल कर सकते हैं।
```bash
# Install
brew install --cask jtool2

jtool2 -l /bin/ls # Get commands (headers)
jtool2 -L /bin/ls # Get libraries
jtool2 -S /bin/ls # Get symbol info
jtool2 -d /bin/ls # Dump binary
jtool2 -D /bin/ls # Decompile binary

# Get signature information
ARCH=x86_64 jtool2 --sig /System/Applications/Automator.app/Contents/MacOS/Automator

# Get MIG information
jtool2 -d __DATA.__const myipc_server | grep MIG
```
{% hint style="danger" %}
**jtool को disarm की प्राथमिकता दी गई है**
{% endhint %}

### Codesign / ldid

{% hint style="success" %}
**`Codesign`** macOS में पाया जा सकता है जबकि **`ldid`** iOS में पाया जा सकता है
{% endhint %}
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

# Get signature info
ldid -h <binary>

# Get entitlements
ldid -e <binary>

# Change entilements
## /tmp/entl.xml is a XML file with the new entitlements to add
ldid -S/tmp/entl.xml <binary>
```
### संदेहास्पद पैकेज

[**संदेहास्पद पैकेज**](https://mothersruin.com/software/SuspiciousPackage/get.html) एक उपकरण है जो **.pkg** फ़ाइलें (इंस्टॉलर) की जांच करने के लिए उपयोगी है और इसे इंस्टॉल करने से पहले भीतर क्या है देखने के लिए।\
इन इंस्टॉलर में `preinstall` और `postinstall` बैश स्क्रिप्ट होते हैं जिन्हें मैलवेयर लेखक सामान्यत: दुरुपयोग करते हैं **मैलवेयर** को **स्थायी** बनाने के लिए।

### hdiutil

यह उपकरण एप्पल डिस्क इमेज (**.dmg**) फ़ाइलों को **माउंट** करने की अनुमति देता है ताकि इन्हें कुछ भी चलाने से पहले जांच सकें:
```bash
hdiutil attach ~/Downloads/Firefox\ 58.0.2.dmg
```
यह `/Volumes` में माउंट किया जाएगा।

### पैक किए गए बाइनरी

* उच्च एंट्रोपी की जांच करें
* स्ट्रिंग की जांच करें (क्या कोई समझने योग्य स्ट्रिंग है, पैक किया गया है)
* MacOS के लिए UPX पैकर एक खंड उत्पन्न करता है जिसे "\_\_XHDR" कहा जाता है

## स्थैतिक Objective-C विश्लेषण

### मेटाडेटा

{% hint style="danger" %}
ध्यान दें कि Objective-C में लिखे गए कार्यक्रम [Mach-O binaries](../macos-files-folders-and-binaries/universal-binaries-and-mach-o-format.md) में कॉम्पाइल होने पर अपनी कक्षा घोषणाएं बनाए रखते हैं। ऐसी कक्षा घोषणाएं नाम और प्रकार को शामिल करती हैं:
{% endhint %}

* परिभाषित इंटरफेस
* इंटरफेस मेथड्स
* इंटरफेस इंस्टेंस वेरिएबल्स
* परिभाषित प्रोटोकॉल्स

ध्यान दें कि इन नामों को अस्पष्ट किया जा सकता है ताकि बाइनरी का पलटाव करना कठिन हो।

### फ़ंक्शन कॉलिंग

जब एक फ़ंक्शन को एक बाइनरी में बुलाया जाता है जो Objective-C का उपयोग करता है, तो कॉम्पाइल कोड उस फ़ंक्शन को बुलाने की बजाय, **`objc_msgSend`** को बुलाएगा। जो अंतिम फ़ंक्शन को बुलाएगा:

![](<../../../.gitbook/assets/image (305).png>)

इस फ़ंक्शन की उम्मीद की जाने वाली पैरामीटर हैं:

* पहला पैरामीटर (**self**) "एक पॉइंटर है जो संदेश प्राप्त करने वाली कक्षा के उदाहरण की ओर पोइंट करता है"। या और सरल शब्दों में कहें, यह वस्तु है जिस पर विधि को आमंत्रित किया जा रहा है। यदि विधि एक कक्षा विधि है, तो यह कक्षा ऑब्जेक्ट का एक उदाहरण होगा (समूचे रूप में), जबकि एक उदाहरण विधि के लिए, self कक्षा के एक उदाहरण को एक ऑब्जेक्ट के रूप में पॉइंट करेगा।
* दूसरा पैरामीटर, (**op**), "संदेश का संभालन करने वाली विधि का सेलेक्टर" है। फिर भी, और सरल शब्दों में कहें, यह बस विधि का **नाम है**।
* शेष पैरामीटर वह सभी **मान हैं जो विधि द्वारा आवश्यक हैं** (op)।

देखें कैसे **इस जानकारी को `lldb` में ARM64 में आसानी से प्राप्त करें** इस पृष्ठ में:

{% content-ref url="arm64-basic-assembly.md" %}
[arm64-basic-assembly.md](arm64-basic-assembly.md)
{% endcontent-ref %}

x64:

| **विधि**         | **रजिस्टर**                                                    | **(के लिए) objc\_msgSend**                                |
| ----------------- | --------------------------------------------------------------- | ------------------------------------------------------ |
| **1वां पैरामीटर**  | **rdi**                                                         | **self: वस्तु जिस पर विधि को आमंत्रित किया जा रहा है** |
| **2वां पैरामीटर**  | **rsi**                                                         | **op: विधि का नाम**                             |
| **3वां पैरामीटर**  | **rdx**                                                         | **विधि के लिए 1वां पैरामीटर**                         |
| **4वां पैरामीटर**  | **rcx**                                                         | **विधि के लिए 2वां पैरामीटर**                         |
| **5वां पैरामीटर**  | **r8**                                                          | **विधि के लिए 3वां पैरामीटर**                         |
| **6वां पैरामीटर**  | **r9**                                                          | **विधि के लिए 4वां पैरामीटर**                         |
| **7वां+ पैरामीटर** | <p><strong>rsp+</strong><br><strong>(स्टैक पर)</strong></p> | **विधि के लिए 5वां+ पैरामीटर**                        |

### ObjectiveC मेटाडेटा डंप करें

### Dynadump

[**Dynadump**](https://github.com/DerekSelander/dynadump) एक टूल है जो Objective-C बाइनरी क्लास-डंप करने के लिए है। गिटहब डायलिब्स को निर्दिष्ट करता है लेकिन यह एक्जीक्यूटेबल्स के साथ भी काम करता है।
```bash
./dynadump dump /path/to/bin
```
जब यह लेख लिखा गया था, यह **वर्तमान में सबसे अच्छा काम करने वाला** है।

#### नियमित उपकरण
```bash
nm --dyldinfo-only /path/to/bin
otool -ov /path/to/bin
objdump --macho --objc-meta-data /path/to/bin
```
#### class-dump

[**class-dump**](https://github.com/nygard/class-dump/) वह मूल उपकरण है जो ObjetiveC स्वरूपित कोड में कक्षाओं, श्रेणियों और प्रोटोकॉल के लिए घोषणाएँ उत्पन्न करता है।

यह पुराना और अनुरक्षित है, इसलिए यह संभावना है कि यह सही ढंग से काम नहीं करेगा।

#### ICDump

[**iCDump**](https://github.com/romainthomas/iCDump) एक आधुनिक और क्रॉस-प्लेटफ़ॉर्म Objective-C क्लास डंप है। मौजूदा उपकरणों की तुलना में, iCDump एप्पल पारिस्थितिकी से स्वतंत्र रूप से चल सकता है और यह पायथन बाइंडिंग्स को उजागर करता है।
```python
import icdump
metadata = icdump.objc.parse("/path/to/bin")

print(metadata.to_decl())
```
## स्थैतिक स्विफ्ट विश्लेषण

स्विफ्ट बाइनरी के साथ, क्योंकि यहाँ Objective-C संगतता है, कभी-कभी आप [class-dump](https://github.com/nygard/class-dump/) का उपयोग करके घोषणाएँ निकाल सकते हैं लेकिन हमेशा नहीं।

**`jtool -l`** या **`otool -l`** कमांड लाइन के साथ कई खंड खोजना संभव है जो **`__swift5`** उपसर्ग से शुरू होते हैं:
```bash
jtool2 -l /Applications/Stocks.app/Contents/MacOS/Stocks
LC 00: LC_SEGMENT_64              Mem: 0x000000000-0x100000000    __PAGEZERO
LC 01: LC_SEGMENT_64              Mem: 0x100000000-0x100028000    __TEXT
[...]
Mem: 0x100026630-0x100026d54        __TEXT.__swift5_typeref
Mem: 0x100026d60-0x100027061        __TEXT.__swift5_reflstr
Mem: 0x100027064-0x1000274cc        __TEXT.__swift5_fieldmd
Mem: 0x1000274cc-0x100027608        __TEXT.__swift5_capture
[...]
```
आप इस [**ब्लॉग पोस्ट में इन सेक्शन में संग्रहित जानकारी के बारे में**](https://knight.sc/reverse%20engineering/2019/07/17/swift-metadata.html) अधिक जानकारी पा सकते हैं।

इसके अतिरिक्त, **स्विफ्ट बाइनरी में प्रतीक हो सकते हैं** (उदाहरण के लिए पुस्तकालयों को प्रतीक संग्रहित करने की आवश्यकता होती है ताकि उनके कार्य को बुलाया जा सके)। **प्रतीकों में सामान्यत: फ़ंक्शन के नाम और एट्रिब्यूट की जानकारी होती है** और एक बेहद अवर्णनीय तरीके में, इसलिए वे बहुत उपयोगी होते हैं और वहाँ "**डीमैंगलर्स"** होते हैं जो मूल नाम प्राप्त कर सकते हैं:
```bash
# Ghidra plugin
https://github.com/ghidraninja/ghidra_scripts/blob/master/swift_demangler.py

# Swift cli
swift demangle
```
## गतिशील विश्लेषण

{% hint style="warning" %}
ध्यान दें कि बाइनरी को डीबग करने के लिए, **SIP को अक्षम करना आवश्यक है** (`csrutil disable` या `csrutil enable --without debug`) या बाइनरी को एक अस्थायी फ़ोल्डर में कॉपी करना और `codesign --remove-signature <binary-path>` के साथ **हस्ताक्षर हटाना** या बाइनरी का डीबगिंग अनुमति देना (आप [इस स्क्रिप्ट](https://gist.github.com/carlospolop/a66b8d72bb8f43913c4b5ae45672578b) का उपयोग कर सकते हैं)
{% endhint %}

{% hint style="warning" %}
ध्यान दें कि **सिस्टम बाइनरी को इंस्ट्रुमेंट करने** के लिए (जैसे `cloudconfigurationd`) macOS पर, **SIP को अक्षम किया जाना चाहिए** (हस्ताक्षर को बस हटाना काम नहीं करेगा)।
{% endhint %}

### एपीआई

macOS कुछ दिलचस्प एपीआई प्रकट करता है जो प्रक्रियाओं के बारे में जानकारी प्रदान करते हैं:

* `proc_info`: यह प्रत्येक प्रक्रिया के बारे में बहुत सारी जानकारी प्रदान करने वाला मुख्य एपीआई है। आपको अन्य प्रक्रियाओं की जानकारी प्राप्त करने के लिए रूट होना चाहिए लेकिन आपको विशेष entitlements या mach ports की आवश्यकता नहीं है।
* `libsysmon.dylib`: यह XPC उद्घाटित फ़ंक्शन के माध्यम से प्रक्रियाओं के बारे में जानकारी प्राप्त करने की अनुमति देता है, हालांकि, इसे `com.apple.sysmond.client` इंटाइटलमेंट होना चाहिए।

### स्टैकशॉट और माइक्रोस्टैकशॉट

**स्टैकशॉटिंग** एक तकनीक है जिसका उपयोग प्रक्रियाओं की स्थिति को कैप्चर करने के लिए किया जाता है, सभी चल रहे धागों के कॉल स्टैक्स सहित। यह डीबगिंग, प्रदर्शन विश्लेषण और विशेष समय पर सिस्टम के व्यवहार को समझने के लिए विशेष रूप से उपयोगी है। iOS और macOS पर, स्टैकशॉटिंग कई उपकरणों और विधियों का उपयोग करके किया जा सकता है जैसे कि उपकरण **`sample`** और **`spindump`**।

### सिसडायग्नोस

यह उपकरण (`/usr/bini/ysdiagnose`) आपके कंप्यूटर से बहुत सारी जानकारी एकत्र करता है और `ps`, `zprint` जैसे अनेक विभिन्न कमांडों को निष्पादित करता है।

इसे **रूट** के रूप में चलाया जाना चाहिए और डेमन `/usr/libexec/sysdiagnosed` के पास `com.apple.system-task-ports` और `get-task-allow` जैसे बहुत दिलचस्प entitlements हैं।

इसका plist `/System/Library/LaunchDaemons/com.apple.sysdiagnose.plist` में स्थित है जो 3 MachServices घोषित करता है:

* `com.apple.sysdiagnose.CacheDelete`: /var/rmp में पुराने आर्काइव को हटाता है
* `com.apple.sysdiagnose.kernel.ipc`: विशेष पोर्ट 23 (कर्नेल)
* `com.apple.sysdiagnose.service.xpc`: `Libsysdiagnose` Obj-C कक्लास के माध्यम से उपयोगकर्ता मोड इंटरफेस। एक डिक्शनरी में तीन तर्क पारित किए जा सकते हैं (`compress`, `display`, `run`)

### एकीकृत लॉग

MacOS बहुत सारे लॉग उत्पन्न करता है जो एक एप्लिकेशन को चलाने के समय **वह क्या कर रहा है** समझने में मददगार हो सकते हैं।

इसके अतिरिक्त, कुछ लॉग होंगे जिनमें `<private>` टैग होगा ताकि कुछ **उपयोगकर्ता** या **कंप्यूटर** **पहचाननीय** जानकारी **छुपी** रहे। हालांकि, इस जानकारी को **खोलने के लिए एक प्रमाणपत्र स्थापित करना संभव है**। [**यहाँ से**](https://superuser.com/questions/1532031/how-to-show-private-data-in-macos-unified-log) विवरणों का पालन करें।

### हॉपर

#### बाएं पैनल

हॉपर के बाएं पैनल में आप बाइनरी के प्रतीक (**लेबल**), प्रक्रियाओं और फ़ंक्शनों की सूची (**प्रोसी**) और स्ट्रिंग्स (**स्ट्र**) देख सकते हैं। ये सभी स्ट्रिंग्स नहीं हैं लेकिन वे मैक-ओ फ़ाइल के कई हिस्सों में परिभाषित हैं (जैसे _cstring या_ `objc_methname`).

#### मध्य पैनल

मध्य पैनल में आप **डिसएंबल्ड कोड** देख सकते हैं। और आप इसे **रॉ** डिसएंबल, **ग्राफ**, **डीकॉम्पाइल्ड** और **बाइनरी** के रूप में देख सकते हैं जिसके लिए आपको संबंधित आइकन पर क्लिक करना होगा:

<figure><img src="../../../.gitbook/assets/image (343).png" alt=""><figcaption></figcaption></figure>

किसी कोड ऑब्जेक्ट पर राइट क्लिक करके आप **उस ऑब्जेक्ट से/के लिए संदर्भ** देख सकते हैं या उसका नाम बदल सकते हैं (यह डिकॉम्पाइल्ड प्यूडोकोड में काम नहीं करता):

<figure><img src="../../../.gitbook/assets/image (1117).png" alt=""><figcaption></figcaption></figure>

इसके अतिरिक्त, **मध्य नीचे आप पायथन कमांड लिख सकते हैं**।

#### दाएं पैनल

दाएं पैनल में आप **नेविगेशन हिस्ट्री** (ताकि आप जानें कि आप वर्तमान स्थिति तक कैसे पहुंचे), **कॉल ग्राफ** जहां आप सभी **फ़ंक्शन देख सकते हैं जो इस फ़ंक्शन को कॉल करते हैं** और सभी फ़ंक्शन जो **यह फ़ंक्शन कॉल करते हैं**, और **स्थानीय चर** जानकारी देख सकते हैं।

### dtrace

यह उपयोगकर्ताओं को एक अत्यधिक **निचले स्तर** पर एप्लिकेशन तक पहुंच प्रदान करता है और उपयोगकर्ताओं को **प्रोग्राम की ट्रेस** करने और उनके निष्पादन धारण को बदलने का एक तरीका प्रदान करता है। Dtrace उपयोगकर्ताओं के लिए **प्रोब्स** का उपयोग करता है जो कर्नेल के भीतर विभिन्न स्थानों पर रखे गए हैं जैसे सिस्टम कॉल के आरंभ और समापन पर।

DTrace उपयोगकर्ता के लिए प्रत्येक सिस्टम कॉल के प्रवेश और निकासी बिंदु में एक प्रोब बनाने के लिए **`dtrace_probe_create`** फ़ंक्शन का उपयोग करता है। इन प्रोब्स को प्रत्येक सिस्टम कॉल के प्रवेश और निकासी बिंदु में फायर किया जा सकता है। DTrace के साथ बातचीत /dev/dtrace के माध्यम से होती है जो केवल रूट उपयोगकर्ता के लिए ही उपलब्ध है।

{% hint style="success" %}
SIP सुरक्षा को पूरी तरह से अक्षम किए बिना Dtrace को सक्षम करने के लिए आप पुनर्प्राप्ति मोड पर निम्नलिखित को कार्यान्वित कर सकते हैं: `csrutil enable --without dtrace`

आप उन बाइनरी को भी **`dtrace`** या **`dtruss`** कर सकते हैं जो **आपने कंपाइल किए हों**।
{% endhint %}

dtrace के उपलब्ध प्रोब्स को प्राप्त करने के लिए:
```bash
dtrace -l | head
ID   PROVIDER            MODULE                          FUNCTION NAME
1     dtrace                                                     BEGIN
2     dtrace                                                     END
3     dtrace                                                     ERROR
43    profile                                                     profile-97
44    profile                                                     profile-199
```
प्रोब नाम चार भागों से मिलकर बनता है: प्रदाता, मॉड्यूल, कार्य और नाम (`fbt:mach_kernel:ptrace:entry`). यदि आप नाम के किसी भाग को निर्दिष्ट नहीं करते हैं, तो Dtrace उस भाग को वाइल्डकार्ड के रूप में लागू करेगा।

प्रोब्स को सक्रिय करने और जब वे फायर होते हैं तो क्या क्रियाएँ करनी हैं, इसे कॉन्फ़िगर करने के लिए हमें डी भाषा का उपयोग करना होगा।

एक और विस्तृत व्याख्या और अधिक उदाहरण [https://illumos.org/books/dtrace/chp-intro.html](https://illumos.org/books/dtrace/chp-intro.html) में देखी जा सकती है।

#### उदाहरण

`man -k dtrace` चलाकर **उपलब्ध DTrace स्क्रिप्ट** की सूची देखें। उदाहरण: `sudo dtruss -n binary`

* लाइन में
```bash
#Count the number of syscalls of each running process
sudo dtrace -n 'syscall:::entry {@[execname] = count()}'
```
* स्क्रिप्ट
```bash
syscall:::entry
/pid == $1/
{
}

#Log every syscall of a PID
sudo dtrace -s script.d 1234
```

```bash
syscall::open:entry
{
printf("%s(%s)", probefunc, copyinstr(arg0));
}
syscall::close:entry
{
printf("%s(%d)\n", probefunc, arg0);
}

#Log files opened and closed by a process
sudo dtrace -s b.d -c "cat /etc/hosts"
```

```bash
syscall:::entry
{
;
}
syscall:::return
{
printf("=%d\n", arg1);
}

#Log sys calls with values
sudo dtrace -s syscalls_info.d -c "cat /etc/hosts"
```
### dtruss
```bash
dtruss -c ls #Get syscalls of ls
dtruss -c -p 1000 #get syscalls of PID 1000
```
### kdebug

यह एक कर्णल ट्रेसिंग सुविधा है। दस्तावेज़ कोड **`/usr/share/misc/trace.codes`** में मिल सकते हैं।

`latency`, `sc_usage`, `fs_usage` और `trace` जैसे उपकरण इसका आंतरिक उपयोग करते हैं।

`kdebug` के साथ इंटरफेस बनाने के लिए `sysctl` को `kern.kdebug` नेमस्पेस के ऊपर उपयोग किया जाता है और उपयोग करने वाले MIBs `sys/sysctl.h` में मिल सकते हैं जिनके कार्यों का अमल `bsd/kern/kdebug.c` में किया गया है।

कस्टम क्लाइंट के साथ kdebug के साथ इंटरैक्ट करने के लिए आम तौर पर निम्नलिखित कदम होते हैं:

* मौजूदा सेटिंग को KERN\_KDSETREMOVE के साथ हटाएं
* KERN\_KDSETBUF और KERN\_KDSETUP के साथ ट्रेस सेट करें
* बफर एंट्री की संख्या प्राप्त करने के लिए KERN\_KDGETBUF का उपयोग करें
* KERN\_KDPINDEX के साथ अपने क्लाइंट को ट्रेस से बाहर लें
* KERN\_KDENABLE के साथ ट्रेसिंग सक्षम करें
* KERN\_KDREADTR को कॉल करके बफर पढ़ें
* प्रत्येक थ्रेड को उसके प्रक्रिया के साथ मिलाने के लिए KERN\_KDTHRMAP को कॉल करें।

इस जानकारी को प्राप्त करने के लिए यह संभव है कि आप Apple उपकरण **`trace`** या कस्टम उपकरण [kDebugView (kdv)](https://newosxbook.com/tools/kdv.html)** का उपयोग करें।**

**ध्यान दें कि Kdebug केवल 1 ग्राहक के लिए ही उपलब्ध है।** इसलिए एक समय में केवल एक k-debug पावर उपकरण को चलाया जा सकता है।

### ktrace

`ktrace_*` API `libktrace.dylib` से आते हैं जो `Kdebug` के उनको ढकने के लिए हैं। फिर, एक क्लाइंट बस `ktrace_session_create` और `ktrace_events_[single/class]` को कॉल कर सकता है विशिष्ट कोड पर कॉलबैक सेट करने के लिए और फिर इसे `ktrace_start` के साथ शुरू कर सकता है।

आप इसे **SIP सक्रिय** होने पर भी उपयोग कर सकते हैं।

आप उपयोगकर्ताओं के रूप में उपयोग कर सकते हैं उपयोगिता `ktrace`:
```bash
ktrace trace -s -S -t c -c ls | grep "ls("
```
या `tailspin`.

### kperf

यह कर्नेल स्तर की प्रोफाइलिंग करने के लिए उपयोग किया जाता है और यह `Kdebug` कॉलआउट का उपयोग करके निर्मित किया गया है।

मूल रूप से, वैश्विक चर `kernel_debug_active` की जांच की जाती है और जब यह सेट होती है तो `Kdebug` कोड और कर्नेल फ्रेम कॉल करने का पता लगाने के लिए `kperf_kdebug_handler` को कॉल करता है। यदि `Kdebug` कोड चयनित किसी को मिलता है तो यह "क्रियाएँ" को बिटमैप के रूप में कॉन्फ़िगर किया जाता है (विकल्पों के लिए `osfmk/kperf/action.h` की जांच करें)।

Kperf में एक sysctl MIB तालिका भी है: (रूट के रूप में) `sysctl kperf`। ये कोड `osfmk/kperf/kperfbsd.c` में पाए जा सकते हैं।

इसके अतिरिक्त, Kperfs की एक उपसमूहता `kpc` में है, जो मशीन प्रदर्शन काउंटर के बारे में जानकारी प्रदान करता है।

### ProcessMonitor

[**ProcessMonitor**](https://objective-see.com/products/utilities.html#ProcessMonitor) एक बहुत ही उपयोगी उपकरण है जो यह जांचने के लिए उपयुक्त है कि एक प्रक्रिया किस प्रकार की क्रियाएँ कर रही है (उदाहरण के लिए, जांचने के लिए कि प्रक्रिया कौन सी नई प्रक्रियाएँ बना रही है)।

### SpriteTree

[**SpriteTree**](https://themittenmac.com/tools/) एक उपकरण है जो प्रक्रियाओं के बीच संबंधों को प्रिंट करता है।\
आपको अपने मैक को एक ऐसे आदेश के साथ मॉनिटर करने की आवश्यकता है जैसे **`sudo eslogger fork exec rename create > cap.json`** (इसे लॉन्च करने के लिए टर्मिनल में FDA की आवश्यकता है)। और फिर आप इस उपकरण में जोड़ सकते हैं जो सभी संबंधों को देखने के लिए जासन लोड कर सकते हैं:

<figure><img src="../../../.gitbook/assets/image (1182).png" alt="" width="375"><figcaption></figcaption></figure>

### FileMonitor

[**FileMonitor**](https://objective-see.com/products/utilities.html#FileMonitor) फ़ाइल घटनाओं को मॉनिटर करने की अनुमति देता है (जैसे निर्माण, संशोधन और हटाने) इन घटनाओं के बारे में विस्तृत जानकारी प्रदान करता है।

### Crescendo

[**Crescendo**](https://github.com/SuprHackerSteve/Crescendo) एक GUI उपकरण है जिसमें Microsoft Sysinternal’s _Procmon_ से जानकारी रखने वाले Windows उपयोगकर्ताओं को जानकारी हो सकती है। यह उपकरण विभिन्न घटना प्रकारों को शुरू और बंद करने की अनुमति देता है, इन घटनाओं को फ़ाइल, प्रक्रिया, नेटवर्क आदि जैसे श्रेणियों द्वारा फ़िल्टर करने की अनुमति देता है, और जानकारी को एक json प्रारूप में सहेजने की क्षमता प्रदान करता है।

### Apple Instruments

[**Apple Instruments**](https://developer.apple.com/library/archive/documentation/Performance/Conceptual/CellularBestPractices/Appendix/Appendix.html) Xcode के डेवलपर उपकरणों का हिस्सा है - जो अनुप्रयोग प्रदर्शन को मॉनिटर करने, मेमोरी लीक्स की पहचान करने और फ़ाइल सिस्टम गतिविधि को ट्रैक करने के लिए उपयोग किया जाता है।

![](<../../../.gitbook/assets/image (1138).png>)
```bash
fs_usage -w -f filesys ls #This tracks filesystem actions of proccess names containing ls
fs_usage -w -f network curl #This tracks network actions
```
### TaskExplorer

[**Taskexplorer**](https://objective-see.com/products/taskexplorer.html) किसी बाइनरी द्वारा उपयोग की जाने वाली **लाइब्रेरी**, उसके द्वारा उपयोग की जा रही **फ़ाइलें** और **नेटवर्क** कनेक्शन देखने के लिए उपयोगी है।\
यह बाइनरी प्रक्रियाओं को **virustotal** के खिलाफ जांचता है और बाइनरी के बारे में जानकारी दिखाता है।

## PT\_DENY\_ATTACH <a href="#page-title" id="page-title"></a>

[**इस ब्लॉग पोस्ट**](https://knight.sc/debugging/2019/06/03/debugging-apple-binaries-that-use-pt-deny-attach.html) में आपको एक उदाहरण मिलेगा कि कैसे **`PT_DENY_ATTACH`** का उपयोग करके डीमन को डीबग करें जो SIP निषेधित होने के बावजूद भी डीबगिंग को रोकता है।

### lldb

**lldb** **macOS** बाइनरी **डीबगिंग** के लिए डी फैक्टो टूल है।
```bash
lldb ./malware.bin
lldb -p 1122
lldb -n malware.bin
lldb -n malware.bin --waitfor
```
जब आप lldb का उपयोग कर रहे हो तो आप intel फ्लेवर सेट कर सकते हैं अपने होम फ़ोल्डर में निम्नलिखित लाइन के साथ फ़ाइल बनाकर **`.lldbinit`** कहा जाता है:
```bash
settings set target.x86-disassembly-flavor intel
```
{% hint style="warning" %}
ल्लडबी के अंदर, `process save-core` के साथ एक प्रक्रिया को डंप करें।
{% endhint %}

<table data-header-hidden><thead><tr><th width="225"></th><th></th></tr></thead><tbody><tr><td><strong>(lldb) कमांड</strong></td><td><strong>विवरण</strong></td></tr><tr><td><strong>run (r)</strong></td><td>एक्सीक्यूशन शुरू करना, जो एक ब्रेकपॉइंट हिट होने या प्रक्रिया समाप्त होने तक जारी रहेगा।</td></tr><tr><td><strong>continue (c)</strong></td><td>डीबग की गई प्रक्रिया का जारी रखना।</td></tr><tr><td><strong>nexti (n / ni)</strong></td><td>अगले इंस्ट्रक्शन को निष्पादित करें। यह कमांड फ़ंक्शन कॉल को छोड़ देगा।</td></tr><tr><td><strong>stepi (s / si)</strong></td><td>अगले इंस्ट्रक्शन को निष्पादित करें। nexti कमांड की तरह, यह कमांड फ़ंक्शन कॉल में कदम रखेगा।</td></tr><tr><td><strong>finish (f)</strong></td><td>वर्तमान फ़ंक्शन ("फ़्रेम") में शेष इंस्ट्रक्शनों को निष्पादित करें और रोकें।</td></tr><tr><td><strong>control + c</strong></td><td>एक्सीक्यूशन को रोकें। अगर प्रक्रिया को चलाया गया है (r) या जारी रखा गया है (c), तो यह प्रक्रिया को वर्तमान में कहीं भी ठहरा देगा।</td></tr><tr><td><strong>breakpoint (b)</strong></td><td><p>b main #कोई फ़ंक्शन जिसे मेन कहा गया है</p><p>b &#x3C;binname>`main #बिन का मुख्य फ़ंक्शन</p><p>b set -n main --shlib &#x3C;lib_name> #निर्दिष्ट बिन का मुख्य फ़ंक्शन</p><p>b -[NSDictionary objectForKey:]</p><p>b -a 0x0000000100004bd9</p><p>br l #ब्रेकपॉइंट सूची</p><p>br e/dis &#x3C;num> #ब्रेकपॉइंट सक्षम/अक्षम करें</p><p>breakpoint delete &#x3C;num></p></td></tr><tr><td><strong>help</strong></td><td><p>help breakpoint #ब्रेकपॉइंट कमांड की मदद प्राप्त करें</p><p>help memory write #मेमोरी में लिखने की मदद प्राप्त करें</p></td></tr><tr><td><strong>reg</strong></td><td><p>reg read</p><p>reg read $rax</p><p>reg read $rax --format &#x3C;<a href="https://lldb.llvm.org/use/variable.html#type-format">format</a>></p><p>reg write $rip 0x100035cc0</p></td></tr><tr><td><strong>x/s &#x3C;reg/memory address></strong></td><td>मेमोरी को एक नल-समाप्त ("सी") स्ट्रिंग के रूप में प्रदर्शित करें।</td></tr><tr><td><strong>x/i &#x3C;reg/memory address></strong></td><td>मेमोरी को एसेम्बली इंस्ट्रक्शन के रूप में प्रदर्शित करें।</td></tr><tr><td><strong>x/b &#x3C;reg/memory address></strong></td><td>मेमोरी को बाइट के रूप में प्रदर्शित करें।</td></tr><tr><td><strong>print object (po)</strong></td><td><p>यह पैरामीटर द्वारा संदर्भित ऑब्जेक्ट प्रिंट करेगा</p><p>po $raw</p><p><code>{</code></p><p><code>dnsChanger = {</code></p><p><code>"affiliate" = "";</code></p><p><code>"blacklist_dns" = ();</code></p><p>ध्यान दें कि अधिकांश Apple के Objective-C API या मेथड्स ऑब्जेक्ट वापस करते हैं, और इसलिए "प्रिंट ऑब्जेक्ट" (po) कमांड के माध्यम से प्रदर्शित किए जाना चाहिए। यदि po मानवीय आउटपुट नहीं प्रदान करता है तो <code>x/b</code> का उपयोग करें</p></td></tr><tr><td><strong>memory</strong></td><td>memory read 0x000....<br>memory read $x0+0xf2a<br>memory write 0x100600000 -s 4 0x41414141 #उस पते में AAAA लिखें<br>memory write -f s $rip+0x11f+7 "AAAA" #उस पते में AAAA लिखें</td></tr><tr><td><strong>disassembly</strong></td><td><p>dis #वर्तमान फ़ंक्शन को डिसैसेंबल करें</p><p>dis -n &#x3C;funcname> #फ़ंक्शन को डिसैसेंबल करें</p><p>dis -n &#x3C;funcname> -b &#x3C;basename> #फ़ंक्शन को डिसैसेंबल करें<br>dis -c 6 #6 लाइनों को डिसैसेंबल करें<br>dis -c 0x100003764 -e 0x100003768 #एक से दूसरे तक<br>dis -p -c 4 #वर्तमान पते से डिसैसेंबल करना शुरू करें</p></td></tr><tr><td><strong>parray</strong></td><td>parray 3 (char **)$x1 # x1 रजिस्टर में 3 घटकों के एरे की जाँच करें</td></tr></tbody></table>

{% hint style="info" %}
**`objc_sendMsg`** फ़ंक्शन को कॉल करते समय, **rsi** रजिस्टर मेथड का नाम एक नल-समाप्त (“सी”) स्ट्रिंग के रूप में रखता है। lldb के माध्यम से नाम प्रिंट करने के लिए:

`(lldb) x/s $rsi: 0x1000f1576: "startMiningWithPort:password:coreCount:slowMemory:currency:"`

`(lldb) print (char*)$rsi:`\
`(char *) $1 = 0x00000001000f1576 "startMiningWithPort:password:coreCount:slowMemory:currency:"`

`(lldb) reg read $rsi: rsi = 0x00000001000f1576 "startMiningWithPort:password:coreCount:slowMemory:currency:"`
{% endhint %}

### एंटी-डायनामिक विश्लेषण

#### वीएम पता लगाना

* कमांड **`sysctl hw.model`** "Mac" लौटाता है जब **होस्ट एक MacOS** होता है लेकिन कुछ अलग होता है जब यह एक वीएम होता है।
* **`hw.logicalcpu`** और **`hw.physicalcpu`** के मानों के साथ खेलने से कुछ मैलवेयर यह जांचने की कोशिश करते हैं कि यह एक वीएम है या नहीं।
* कुछ मैलवेयर यह भी **पता लगा सकते हैं** कि मशीन **VMware** पर आधारित है MAC पते (00:50:56) के आधार पर।
* यह भी संभव है कि एक प्रक्रिया की डीबगिंग हो रही है या नहीं इसे एक सरल कोड के साथ खोजना:
* `if(P_TRACED == (info.kp_proc.p_flag & P_TRACED)){ //प्रक्रिया जिसे डीबग किया जा रहा है }`
* यह भी **`ptrace`** सिस्टम कॉल को **`PT_DENY_ATTACH`** फ़्लैग के साथ आमंत्रित कर सकता है। यह एक डेबगर को जोड़ने और ट्रेसिंग करने से रोकता है।
* आप यह जांच सकते हैं कि क्या **`sysctl`** या **`ptrace`** फ़ंक्शन को **आयात** किया जा रहा है (लेकिन मैलवेयर इसे गतिशील रूप से आयात कर सकता है)
* जैसा कि इस लेखन में दर्शाया गया है, “[एंटी-डीबग तकनीकों को पराजित करना: macOS ptrace variants](https://alexomara.com/blog/defeating-anti-debug-techniques-macos-ptrace-variants/)” :\
“_संदेश प्रक्रिया # के साथ बाहर निकल गई **स्थिति = 45 (0x0000002d)** आम तौर पर एक संकेत है कि डीबग लक्ष्य **PT\_DENY\_ATTACH** का उपयोग कर रहा है_”
## कोर डंप्स

कोर डंप्स बनाए जाते हैं अगर:

- `kern.coredump` सिसक्टल 1 पर सेट किया गया है (डिफ़ॉल्ट)
- अगर प्रक्रिया suid/sgid नहीं थी या `kern.sugid_coredump` 1 है (डिफ़ॉल्ट में 0)
- `AS_CORE` सीमा ऑपरेशन की अनुमति देती है। कोर डंप्स बनाने को `ulimit -c 0` बुलाकर दबा सकते हैं और `ulimit -c unlimited` से उन्हें पुनः सक्रिय कर सकते हैं।

इन मामलों में कोर डंप्स `kern.corefile` सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस्टेम सिस
```bash
#To disable crash reporting:
launchctl unload -w /System/Library/LaunchAgents/com.apple.ReportCrash.plist
sudo launchctl unload -w /System/Library/LaunchDaemons/com.apple.ReportCrash.Root.plist

#To re-enable crash reporting:
launchctl load -w /System/Library/LaunchAgents/com.apple.ReportCrash.plist
sudo launchctl load -w /System/Library/LaunchDaemons/com.apple.ReportCrash.Root.plist
```
### नींद

MacOS में फज़िंग करते समय Mac को सोने नहीं देने का महत्वपूर्ण है:

* systemsetup -setsleep Never
* pmset, सिस्टम प्राथमिकताएँ
* [KeepingYouAwake](https://github.com/newmarcel/KeepingYouAwake)

#### SSH डिस्कनेक्ट

यदि आप SSH कनेक्शन के माध्यम से फज़िंग कर रहे हैं तो सुनिश्चित करना महत्वपूर्ण है कि सत्र दिन भर न जाए। इसलिए sshd\_config फ़ाइल को निम्नलिखित से बदलें:

* TCPKeepAlive Yes
* ClientAliveInterval 0
* ClientAliveCountMax 0
```bash
sudo launchctl unload /System/Library/LaunchDaemons/ssh.plist
sudo launchctl load -w /System/Library/LaunchDaemons/ssh.plist
```
### आंतरिक हैंडलर

**निम्नलिखित पृष्ठ की जाँच करें** ताकि आप पता लगा सकें कि **निर्दिष्ट स्कीम या प्रोटोकॉल का हैंडलिंग कौन संभाल रहा है:**

{% content-ref url="../macos-file-extension-apps.md" %}
[macos-file-extension-apps.md](../macos-file-extension-apps.md)
{% endcontent-ref %}

### नेटवर्क प्रक्रियाओं की गणना

यह दिलचस्प है कि नेटवर्क डेटा को प्रबंधित कर रही प्रक्रियाएं खोजना:
```bash
dtrace -n 'syscall::recv*:entry { printf("-> %s (pid=%d)", execname, pid); }' >> recv.log
#wait some time
sort -u recv.log > procs.txt
cat procs.txt
```
या `netstat` या `lsof` का उपयोग करें

### Libgmalloc

<figure><img src="../../../.gitbook/assets/Pasted Graphic 14.png" alt=""><figcaption></figcaption></figure>

{% code overflow="wrap" %}
```bash
lldb -o "target create `which some-binary`" -o "settings set target.env-vars DYLD_INSERT_LIBRARIES=/usr/lib/libgmalloc.dylib" -o "run arg1 arg2" -o "bt" -o "reg read" -o "dis -s \$pc-32 -c 24 -m -F intel" -o "quit"
```
{% endcode %}

### फज़र्स

#### [AFL++](https://github.com/AFLplusplus/AFLplusplus)

CLI टूल्स के लिए काम करता है

#### [Litefuzz](https://github.com/sec-tools/litefuzz)

यह macOS GUI टूल्स के साथ "**बस काम करता है"**। ध्यान दें कि कुछ macOS ऐप्स के कुछ विशेष आवश्यकताएं हो सकती हैं जैसे अद्वितीय फ़ाइल नाम, सही एक्सटेंशन, सैंडबॉक्स से फ़ाइलों को पढ़ने की आवश्यकता (`~/Library/Containers/com.apple.Safari/Data`)...

कुछ उदाहरण:

{% code overflow="wrap" %}
```bash
# iBooks
litefuzz -l -c "/System/Applications/Books.app/Contents/MacOS/Books FUZZ" -i files/epub -o crashes/ibooks -t /Users/test/Library/Containers/com.apple.iBooksX/Data/tmp -x 10 -n 100000 -ez

# -l : Local
# -c : cmdline with FUZZ word (if not stdin is used)
# -i : input directory or file
# -o : Dir to output crashes
# -t : Dir to output runtime fuzzing artifacts
# -x : Tmeout for the run (default is 1)
# -n : Num of fuzzing iterations (default is 1)
# -e : enable second round fuzzing where any crashes found are reused as inputs
# -z : enable malloc debug helpers

# Font Book
litefuzz -l -c "/System/Applications/Font Book.app/Contents/MacOS/Font Book FUZZ" -i input/fonts -o crashes/font-book -x 2 -n 500000 -ez

# smbutil (using pcap capture)
litefuzz -lk -c "smbutil view smb://localhost:4455" -a tcp://localhost:4455 -i input/mac-smb-resp -p -n 100000 -z

# screensharingd (using pcap capture)
litefuzz -s -a tcp://localhost:5900 -i input/screenshared-session --reportcrash screensharingd -p -n 100000
```
{% endcode %}

### अधिक Fuzzing MacOS जानकारी

* [https://www.youtube.com/watch?v=T5xfL9tEg44](https://www.youtube.com/watch?v=T5xfL9tEg44)
* [https://github.com/bnagy/slides/blob/master/OSXScale.pdf](https://github.com/bnagy/slides/blob/master/OSXScale.pdf)
* [https://github.com/bnagy/francis/tree/master/exploitaben](https://github.com/bnagy/francis/tree/master/exploitaben)
* [https://github.com/ant4g0nist/crashwrangler](https://github.com/ant4g0nist/crashwrangler)

## संदर्भ

* [**OS X Incident Response: Scripting and Analysis**](https://www.amazon.com/OS-Incident-Response-Scripting-Analysis-ebook/dp/B01FHOHHVS)
* [**https://www.youtube.com/watch?v=T5xfL9tEg44**](https://www.youtube.com/watch?v=T5xfL9tEg44)
* [**https://taomm.org/vol1/analysis.html**](https://taomm.org/vol1/analysis.html)
* [**The Art of Mac Malware: The Guide to Analyzing Malicious Software**](https://taomm.org/)

### [WhiteIntel](https://whiteintel.io)

<figure><img src="../../../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) एक **डार्क-वेब** से प्रेरित खोज इंजन है जो **नि:शुल्क** सुविधाएं प्रदान करता है ताकि जांच सकें कि कोई कंपनी या उसके ग्राहकों को **स्टीलर मैलवेयर** द्वारा **कंप्रोमाइज** किया गया है।

WhiteIntel का मुख्य उद्देश्य खाता हाथ में लेने और रैंसमवेयर हमलों से लड़ना है जो जानकारी चोरी करने वाले मैलवेयर से होते हैं।

आप उनकी वेबसाइट चेक कर सकते हैं और **नि:शुल्क** इंजन का प्रयास कर सकते हैं:

{% embed url="https://whiteintel.io" %}

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी कंपनी का विज्ञापन देखना चाहते हैं HackTricks में या HackTricks को PDF में डाउनलोड करना चाहते हैं तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **जुड़ें** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos को PRs सबमिट करके।

</details>
