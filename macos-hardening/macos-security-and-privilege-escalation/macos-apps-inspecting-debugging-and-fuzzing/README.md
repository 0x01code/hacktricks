# macOS Apps - निरीक्षण, डीबगिंग और फजिंग

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके।

</details>

## स्थैतिक विश्लेषण

### otool
```bash
otool -L /bin/ls #List dynamically linked libraries
otool -tv /bin/ps #Decompile application
```
### objdump

{% code overflow="wrap" %}
```bash
objdump -m --dylibs-used /bin/ls #List dynamically linked libraries
objdump -m -h /bin/ls # Get headers information
objdump -m --syms /bin/ls # Check if the symbol table exists to get function names
objdump -m --full-contents /bin/ls # Dump every section
objdump -d /bin/ls # Dissasemble the binary
objdump --disassemble-symbols=_hello --x86-asm-syntax=intel toolsdemo #Disassemble a function using intel flavour
```
### jtool2

यह टूल **codesign**, **otool**, और **objdump** के लिए एक **प्रतिस्थापन** के रूप में उपयोग किया जा सकता है, और कुछ अतिरिक्त सुविधाएँ प्रदान करता है। [**यहाँ से डाउनलोड करें**](http://www.newosxbook.com/tools/jtool.html) या `brew` के साथ इंस्टॉल करें।
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
### Codesign / ldid

{% hint style="danger" %}
**`Codesign`** मैकओएस में पाया जा सकता है जबकि **`ldid`** आईओएस में पाया जा सकता है।
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

[**संदेहास्पद पैकेज**](https://mothersruin.com/software/SuspiciousPackage/get.html) एक उपकरण है जो **.pkg** फ़ाइलें (इंस्टॉलर) की जांच करने के लिए उपयोगी है और इसे इंस्टॉल करने से पहले भीतर क्या है देखने में मदद करता है।\
इन इंस्टॉलर में `preinstall` और `postinstall` बैश स्क्रिप्ट होते हैं जिन्हें मैलवेयर लेखक सामान्यत: दुरुपयोग करते हैं **मैलवेयर** को **स्थायी** बनाने के लिए।

### hdiutil

यह उपकरण एप्पल डिस्क इमेज (**.dmg**) फ़ाइलों को **माउंट** करने की अनुमति देता है ताकि इन्हें चलाने से पहले जांच सकें:
```bash
hdiutil attach ~/Downloads/Firefox\ 58.0.2.dmg
```
यह `/Volumes` में माउंट किया जाएगा।

### Objective-C

#### मेटाडेटा

{% hint style="danger" %}
ध्यान दें कि Objective-C में लिखे गए कार्यक्रम [Mach-O binaries](../macos-files-folders-and-binaries/universal-binaries-and-mach-o-format.md) में कंपाइल किए जाते समय अपनी क्लास घोषणाएं **रखते हैं**। ऐसी क्लास घोषणाएं में नाम और प्रकार शामिल होते हैं:
{% endhint %}

* कक्षा
* कक्षा के मेथड
* कक्षा के इंस्टेंस वेरिएबल्स

आप [**class-dump**](https://github.com/nygard/class-dump) का उपयोग करके इस जानकारी को प्राप्त कर सकते हैं:
```bash
class-dump Kindle.app
```
#### कार्य को बुलाना

जब एक फ़ंक्शन को कॉल किया जाता है एक बाइनरी में जो objective-C का उपयोग करता है, कॉम्पाइल कोड उस फ़ंक्शन को बुलाने की बजाय, **`objc_msgSend`** को बुलाएगा। जो अंतिम फ़ंक्शन को बुलाएगा:

![](<../../../.gitbook/assets/image (560).png>)

इस फ़ंक्शन की उम्मीद करती है:

* पहला पैरामीटर (**self**) "एक पॉइंटर है जो **संदेश प्राप्त करने वाली कक्षा के उदाहरण की ओर पॉइंट करता है**"। या और सरल शब्दों में कहें, यह वस्तु है जिस पर विधि को आमंत्रित किया जा रहा है। यदि विधि एक कक्षा विधि है, तो यह कक्षा वस्तु का एक उदाहरण होगा (सम्पूर्ण रूप से), जबकि एक उदाहरण विधि के लिए, self कक्षा के एक उदाहरण को एक वस्तु के रूप में पॉइंट करेगा।
* दूसरा पैरामीटर, (**op**), "संदेश का संभालन करने वाली विधि का चयनकर्ता" है। फिर सरल शब्दों में कहें, यह बस **विधि का नाम है**।
* शेष पैरामीटर वह कोई **मान हैं जो विधि के द्वारा आवश्यक हैं** (op)।

| **विवाद**        | **रजिस्टर**                                                    | **(के लिए) objc\_msgSend**                                |
| ----------------- | --------------------------------------------------------------- | ------------------------------------------------------ |
| **1 वां विवाद**  | **rdi**                                                         | **self: वस्तु जिस पर विधि को आमंत्रित किया जा रहा है** |
| **2 वां विवाद**  | **rsi**                                                         | **op: विधि का नाम**                             |
| **3 वां विवाद**  | **rdx**                                                         | **विधि के लिए 1 वां विवाद**                         |
| **4 वां विवाद**  | **rcx**                                                         | **विधि के लिए 2 वां विवाद**                         |
| **5 वां विवाद**  | **r8**                                                          | **विधि के लिए 3 वां विवाद**                         |
| **6 वां विवाद**  | **r9**                                                          | **विधि के लिए 4 वां विवाद**                         |
| **7+ विवाद** | <p><strong>rsp+</strong><br><strong>(स्टैक पर)</strong></p> | **विधि के लिए 5+ विवाद**                        |

### स्विफ्ट

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
आप इस [**ब्लॉग पोस्ट में इन सेक्शन में संग्रहित जानकारी के बारे में अधिक जानकारी पा सकते हैं**](https://knight.sc/reverse%20engineering/2019/07/17/swift-metadata.html)।

इसके अतिरिक्त, **स्विफ्ट बाइनरी में प्रतीक हो सकते हैं** (उदाहरण के लिए पुस्तकालयों को प्रतीक संग्रहित करने की आवश्यकता होती है ताकि उनके कार्य को बुलाया जा सके)। **प्रतीकों में सामान्यत: फ़ंक्शन के नाम और एट्रिब्यूट की जानकारी होती है और यह एक बेहद अद्भुत तरीके से होती है, इसलिए वे बहुत उपयोगी होते हैं और वहाँ "**डीमैंगलर्स"** होते हैं जो मूल नाम प्राप्त कर सकते हैं:
```bash
# Ghidra plugin
https://github.com/ghidraninja/ghidra_scripts/blob/master/swift_demangler.py

# Swift cli
swift demangle
```
### पैक किए गए बाइनरी

* उच्च एंट्रोपी की जांच करें
* स्ट्रिंग्स की जांच करें (क्या कोई समझने योग्य स्ट्रिंग है, पैक किया गया है)
* MacOS के लिए UPX पैकर एक धारा उत्पन्न करता है जिसे "\_\_XHDR" कहा जाता है

## गतिशील विश्लेषण

{% hint style="warning" %}
ध्यान दें कि बाइनरी को डीबग करने के लिए **SIP को अक्षम करना आवश्यक है** (`csrutil disable` या `csrutil enable --without debug`) या बाइनरी को एक अस्थायी फ़ोल्डर में कॉपी करना और `codesign --remove-signature <binary-path>` के साथ **हस्ताक्षर को हटाना** या बाइनरी का डीबगिंग अनुमति देना (आप [इस स्क्रिप्ट](https://gist.github.com/carlospolop/a66b8d72bb8f43913c4b5ae45672578b) का उपयोग कर सकते हैं)
{% endhint %}

{% hint style="warning" %}
ध्यान दें कि **सिस्टम बाइनरी** (जैसे `cloudconfigurationd`) को **इंस्ट्रुमेंट** करने के लिए macOS पर, **SIP को अक्षम किया जाना चाहिए** (हस्ताक्षर को बस हटाना काम नहीं करेगा)।
{% endhint %}

### एकीकृत लॉग

MacOS बहुत सारे लॉग उत्पन्न करता है जो एक एप्लिकेशन चलाने पर **यह समझने में मददगार हो सकते हैं कि वह क्या कर रहा है**।

इसके अतिरिक्त, कुछ लॉग होंगे जिनमें `<private>` टैग होगा ताकि कुछ **उपयोगकर्ता** या **कंप्यूटर संदर्भीय** **पहचाननीय** जानकारी **छुपी** हो। हालांकि, इस जानकारी को **खोलने के लिए एक प्रमाणपत्र स्थापित किया जा सकता है**। [**यहाँ**](https://superuser.com/questions/1532031/how-to-show-private-data-in-macos-unified-log) से विवरणों का पालन करें।

### हॉपर

#### बाएं पैनल

हॉपर के बाएं पैनल में आप बाइनरी के प्रतीक (**लेबल**), प्रक्रियाओं और कार्यों (**प्रोसेड्यूर्स**) की सूची और स्ट्रिंग्स (**स्ट्र**) देख सकते हैं। ये सभी स्ट्रिंग्स नहीं हैं लेकिन वे वहाँ परिभाषित हैं जहाँ Mac-O फ़ाइल के कई हिस्सों में (_cstring या_ `objc_methname` जैसे)।

#### मध्य पैनल

मध्य पैनल में आप **डिसएंबल कोड** देख सकते हैं। और आप इसे **रॉ**, **ग्राफ**, **डीकंपाइल्ड** और **बाइनरी** रूप में देख सकते हैं जिसके लिए आपको संबंधित आइकन पर क्लिक करना होगा:

<figure><img src="../../../.gitbook/assets/image (2) (6).png" alt=""><figcaption></figcaption></figure>

किसी कोड ऑब्जेक्ट पर राइट क्लिक करके आप **उस ऑब्ज
```bash
dtrace -l | head
ID   PROVIDER            MODULE                          FUNCTION NAME
1     dtrace                                                     BEGIN
2     dtrace                                                     END
3     dtrace                                                     ERROR
43    profile                                                     profile-97
44    profile                                                     profile-199
```
प्रोब नाम चार भागों से मिलकर बनता है: प्रदाता, मॉड्यूल, कार्य, और नाम (`fbt:mach_kernel:ptrace:entry`). अगर आप नाम के किसी भाग को निर्दिष्ट नहीं करते हैं, तो Dtrace उस भाग को वाइल्डकार्ड के रूप में लागू करेगा।

प्रोब्स को सक्रिय करने और जब वे फायर होते हैं तो क्या क्रियाएँ करनी हैं, इसे कॉन्फ़िगर करने के लिए हमें डी भाषा का उपयोग करना होगा।

एक और विस्तृत व्याख्या और अधिक उदाहरण [https://illumos.org/books/dtrace/chp-intro.html](https://illumos.org/books/dtrace/chp-intro.html) में देखा जा सकता है।

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
### ktrace

आप इसे **SIP सक्रिय** होने पर भी उपयोग कर सकते हैं
```bash
ktrace trace -s -S -t c -c ls | grep "ls("
```
### ProcessMonitor

[**ProcessMonitor**](https://objective-see.com/products/utilities.html#ProcessMonitor) एक बहुत ही उपयोगी उपकरण है जो यह जांचने के लिए उपयोगी है कि कोई प्रक्रिया किस प्रकार की क्रियाएँ कर रही है (उदाहरण के लिए, देखें कि कोई प्रक्रिया कौन से नए प्रक्रियाएँ बना रही है)।

### SpriteTree

[**SpriteTree**](https://themittenmac.com/tools/) एक उपकरण है जो प्रक्रियाओं के बीच संबंधों को प्रिंट करता है।\
आपको अपने मैक को एक ऐसे कमांड के साथ मॉनिटर करने की आवश्यकता है जैसे **`sudo eslogger fork exec rename create > cap.json`** (इसे लॉन्च करने के लिए टर्मिनल में FDA की आवश्यकता है)। और फिर आप इस उपकरण में json लोड कर सकते हैं ताकि आप सभी संबंधों को देख सकें:

<figure><img src="../../../.gitbook/assets/image (710).png" alt="" width="375"><figcaption></figcaption></figure>

### FileMonitor

[**FileMonitor**](https://objective-see.com/products/utilities.html#FileMonitor) फ़ाइल घटनाओं (जैसे निर्माण, संशोधन और हटाने) का मॉनिटर करने की अनुमति देता है जो इन घटनाओं के बारे में विस्तृत जानकारी प्रदान करता है।

### Crescendo

[**Crescendo**](https://github.com/SuprHackerSteve/Crescendo) एक GUI उपकरण है जिसका लुक और फ़ील Windows उपयोगकर्ता जान सकते हैं Microsoft Sysinternal’s _Procmon_ से। यह उपकरण विभिन्न घटना प्रकारों को शुरू और रोकने की अनुमति देता है, इन घटनाओं को फ़ाइल, प्रक्रिया, नेटवर्क आदि जैसे श्रेणियों द्वारा फ़िल्टर करने की अनुमति देता है, और जो घटनाएँ जोड़ी गई हैं उन्हें json प्रारूप में सहेजने की क्षमता प्रदान करता है।

### Apple Instruments

[**Apple Instruments**](https://developer.apple.com/library/archive/documentation/Performance/Conceptual/CellularBestPractices/Appendix/Appendix.html) Xcode के डेवलपर उपकरणों का हिस्सा है - जो अनुप्रयोग प्रदर्शन को मॉनिटर करने, मेमोरी लीक्स की पहचान करने और फ़ाइल सिस्टम गतिविधि को ट्रैक करने के लिए उपयोग किया जाता है।

![](<../../../.gitbook/assets/image (15).png>) 

### fs\_usage

प्रक्रियाओं द्वारा किए गए कार्रवाई का पालन करने की अनुमति देता है:
```bash
fs_usage -w -f filesys ls #This tracks filesystem actions of proccess names containing ls
fs_usage -w -f network curl #This tracks network actions
```
### TaskExplorer

[**Taskexplorer**](https://objective-see.com/products/taskexplorer.html) किसी बाइनरी द्वारा उपयोग की जाने वाली **लाइब्रेरी**, उसके द्वारा उपयोग की जा रही **फ़ाइलें** और **नेटवर्क** कनेक्शन देखने के लिए उपयोगी है।\
यह बाइनरी प्रक्रियाओं को **virustotal** के खिलाफ जांचता है और बाइनरी के बारे में जानकारी दिखाता है।

## PT\_DENY\_ATTACH <a href="#page-title" id="page-title"></a>

[**इस ब्लॉग पोस्ट**](https://knight.sc/debugging/2019/06/03/debugging-apple-binaries-that-use-pt-deny-attach.html) में आप एक उदाहरण पा सकते हैं कि कैसे **`PT_DENY_ATTACH`** का उपयोग करके डीबगिंग को रोकने वाले चल रहे डेमन को डीबग करें भले ही SIP अक्षम हो।

### lldb

**lldb** **macOS** बाइनरी **डीबगिंग** के लिए डी फैक्टो टूल है।
```bash
lldb ./malware.bin
lldb -p 1122
lldb -n malware.bin
lldb -n malware.bin --waitfor
```
जब आप lldb का उपयोग कर रहे होते हैं, तो आप अपने होम फ़ोल्डर में निम्नलिखित लाइन के साथ फ़ाइल बना कर intel फ्लेवर सेट कर सकते हैं:
```bash
settings set target.x86-disassembly-flavor intel
```
{% hint style="warning" %}
ल्लडबी में, `process save-core` के साथ एक प्रक्रिया को डंप करें।
{% endhint %}

<table data-header-hidden><thead><tr><th width="225"></th><th></th></tr></thead><tbody><tr><td><strong>(lldb) कमांड</strong></td><td><strong>विवरण</strong></td></tr><tr><td><strong>run (r)</strong></td><td>एक्सीक्यूशन शुरू करें, जो एक ब्रेकपॉइंट हिट होने या प्रक्रिया समाप्त होने तक जारी रहेगा।</td></tr><tr><td><strong>continue (c)</strong></td><td>डीबग की गई प्रक्रिया का एक्सीक्यूशन जारी रखें।</td></tr><tr><td><strong>nexti (n / ni)</strong></td><td>अगले इंस्ट्रक्शन को एक्सीक्यूट करें। यह कमांड फ़ंक्शन कॉल को छोड़ देगा।</td></tr><tr><td><strong>stepi (s / si)</strong></td><td>अगले इंस्ट्रक्शन को एक्सीक्यूट करें। nexti कमांड की तरह, यह कमांड फ़ंक्शन कॉल में स्टेप करेगा।</td></tr><tr><td><strong>finish (f)</strong></td><td>वर्तमान फ़ंक्शन ("फ़्रेम") में शेष इंस्ट्रक्शनों को एक्सीक्यूट करें और रोकें।</td></tr><tr><td><strong>control + c</strong></td><td>एक्सीक्यूशन को रोकें। यदि प्रक्रिया को चलाया गया है (r) या जारी रखा गया है (c), तो यह प्रक्रिया को वर्तमान में कहीं भी रोक देगा।</td></tr><tr><td><strong>breakpoint (b)</strong></td><td><p>b main #कोई फंक्शन जिसका नाम main है</p><p>b &#x3C;binname>`main #बिन का मुख्य फंक्शन</p><p>b set -n main --shlib &#x3C;lib_name> #निर्दिष्ट बिन का मुख्य फंक्शन</p><p>b -[NSDictionary objectForKey:]</p><p>b -a 0x0000000100004bd9</p><p>br l #ब्रेकपॉइंट सूची</p><p>br e/dis &#x3C;num> #ब्रेकपॉइंट सक्षम/अक्षम करें</p><p>breakpoint delete &#x3C;num></p></td></tr><tr><td><strong>help</strong></td><td><p>help breakpoint #ब्रेकपॉइंट कमांड की मदद प्राप्त करें</p><p>help memory write #मेमोरी में लिखने की मदद प्राप्त करें</p></td></tr><tr><td><strong>reg</strong></td><td><p>reg read</p><p>reg read $rax</p><p>reg read $rax --format &#x3C;<a href="https://lldb.llvm.org/use/variable.html#type-format">format</a>></p><p>reg write $rip 0x100035cc0</p></td></tr><tr><td><strong>x/s &#x3C;reg/memory address></strong></td><td>मेमोरी को एक नल-समाप्त ("सी") स्ट्रिंग के रूप में प्रदर्शित करें।</td></tr><tr><td><strong>x/i &#x3C;reg/memory address></strong></td><td>मेमोरी को एसेम्बली इंस्ट्रक्शन के रूप में प्रदर्शित करें।</td></tr><tr><td><strong>x/b &#x3C;reg/memory address></strong></td><td>मेमोरी को बाइट के रूप में प्रदर्शित करें।</td></tr><tr><td><strong>print object (po)</strong></td><td><p>यह पैरामीटर द्वारा संदर्भित ऑब्जेक्ट प्रिंट करेगा</p><p>po $raw</p><p><code>{</code></p><p><code>dnsChanger = {</code></p><p><code>"affiliate" = "";</code></p><p><code>"blacklist_dns" = ();</code></p><p>ध्यान दें कि अधिकांश Apple के Objective-C API या मेथड्स ऑब्जेक्ट वापस करते हैं, और इसलिए "प्रिंट ऑब्जेक्ट" (po) कमांड के माध्यम से प्रदर्शित किए जाना चाहिए। यदि po से कोई अर्थपूर्ण आउटपुट नहीं निकालता है, तो <code>x/b</code> का उपयोग करें</p></td></tr><tr><td><strong>memory</strong></td><td>memory read 0x000....<br>memory read $x0+0xf2a<br>memory write 0x100600000 -s 4 0x41414141 #उस पते में AAAA लिखें<br>memory write -f s $rip+0x11f+7 "AAAA" #उस पते में AAAA लिखें</td></tr><tr><td><strong>disassembly</strong></td><td><p>dis #वर्तमान फ़ंक्शन को डिसैसेंबल करें</p><p>dis -n &#x3C;funcname> #फ़ंक्शन को डिसैसेंबल करें</p><p>dis -n &#x3C;funcname> -b &#x3C;basename> #फ़ंक्शन को डिसैसेंबल करें<br>dis -c 6 #6 लाइनों को डिसैसेंबल करें<br>dis -c 0x100003764 -e 0x100003768 #एक से दूसरे तक<br>dis -p -c 4 #वर्तमान पते से डिसैसेंबल करना शुरू करें</p></td></tr><tr><td><strong>parray</strong></td><td>parray 3 (char **)$x1 #x1 रजिस्टर में 3 घटकों की एरे की जाँच करें</td></tr></tbody></table>

{% hint style="info" %}
**`objc_sendMsg`** फ़ंक्शन को कॉल करते समय, **rsi** रजिस्टर मेथड का नाम एक नल-समाप्त (“सी”) स्ट्रिंग के रूप में रखता है। lldb के माध्यम से नाम प्रिंट करने के लिए:

`(lldb) x/s $rsi: 0x1000f1576: "startMiningWithPort:password:coreCount:slowMemory:currency:"`

`(lldb) print (char*)$rsi:`\
`(char *) $1 = 0x00000001000f1576 "startMiningWithPort:password:coreCount:slowMemory:currency:"`

`(lldb) reg read $rsi: rsi = 0x00000001000f1576 "startMiningWithPort:password:coreCount:slowMemory:currency:"`
{% endhint %}

### एंटी-डायनामिक विश्लेषण

#### VM पता लगाना

* **`sysctl hw.model`** कमांड "Mac" लौटाता है जब **होस्ट एक MacOS** है लेकिन कुछ अलग जब यह एक VM है।
* **`hw.logicalcpu`** और **`hw.physicalcpu`** के मानों के साथ खेलने से कुछ मैलवेयर्स यह जांचने की कोशिश करते हैं कि यह एक VM है या नहीं।
* कुछ मैलवेयर्स यह भी **पता लगा सकते हैं** कि मशीन **VMware** पर आधारित है (00:50:56)।
* यह भी संभव है कि एक प्रक्रिया की **डीबगिंग हो रही है** या नहीं एक सरल कोड के साथ जैसे:
* `if(P_TRACED == (info.kp_proc.p_flag & P_TRACED)){ //प्रक्रिया डीबग हो रही है }`
* यह भी **`ptrace`** सिस्टम कॉल को **`PT_DENY_ATTACH`** फ्लैग के साथ आवंटित कर सकता है। यह एक डेबगर को जोड़ने और ट्रेसिंग करने से **रोकता है**।
* आप यह जांच सकते हैं कि क्या **`sysctl`** या **`ptrace`** फ़ंक्शन **आयात** किया जा रहा है (लेकिन मैलवेयर इसे गतिशील रूप से आयात कर सकता है)
* इस लेख में दर्ज किया गया है, “[एंटी-डीबग तकनीकों को पराजित करना: macOS ptrace variants](https://alexomara.com/blog/defeating-anti-debug-techniques-macos-ptrace-variants/)” :\
“_संदेश प्रक्रिया # के साथ बाहर निकल गई जिसका **स्थिति = 45 (0x0000002d)** है आम तौर पर एक संकेत है कि डीबग लक्ष्य **PT\_DENY\_ATTACH** का उपयोग कर रहा है_”
## फज़िंग

### [ReportCrash](https://ss64.com/osx/reportcrash.html)

ReportCrash **क्रैश होने वाली प्रक्रियाओं का विश्लेषण करता है और डिस्क पर एक क्रैश रिपोर्ट सहेजता है**। एक क्रैश रिपोर्ट में जानकारी होती है जो एक ड्रवाइवर के कारण का निदान करने में मदद कर सकती है।\
उपयोगकर्ता लॉन्चडी संदर्भ में चल रहे एप्लिकेशन और अन्य प्रक्रियाओं के लिए, ReportCrash एक लॉन्चएजेंट के रूप में चलता है और उपयोगकर्ता के `~/Library/Logs/DiagnosticReports/` में क्रैश रिपोर्ट सहेजता है।\
डेमन्स, अन्य प्रक्रियाएं **सिस्टम लॉन्चडी संदर्भ में चल रहे** और अन्य विशेषाधिकारी प्रक्रियाएं के लिए, ReportCrash एक लॉन्चडेमन के रूप में चलता है और सिस्टम के `/Library/Logs/DiagnosticReports` में क्रैश रिपोर्ट सहेजता है।

यदि आपको चिंता है कि क्रैश रिपोर्ट **एप्पल को भेजे जा रहे** हैं तो आप उन्हें अक्षम कर सकते हैं। अगर नहीं, क्रैश रिपोर्ट किस प्रकार से एक सर्वर
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
* pmset, System Preferences
* [KeepingYouAwake](https://github.com/newmarcel/KeepingYouAwake)

#### SSH डिस्कनेक्ट

यदि आप SSH कनेक्शन के माध्यम से फज़िंग कर रहे हैं तो सुनिश्चित करें कि सत्र दिन भर नहीं जा रहा है। इसलिए sshd\_config फ़ाइल को निम्नलिखित से बदलें:

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

यह दिलचस्प है कि नेटवर्क डेटा का प्रबंधन कर रही प्रक्रियाएं खोजना:
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

यह macOS GUI टूल्स के साथ "**बस काम करता है"**। ध्यान दें कि कुछ macOS ऐप्स के कुछ विशेष आवश्यकताएं हो सकती हैं जैसे अद्वितीय फ़ाइल नाम, सही एक्सटेंशन, सैंडबॉक्स से फ़ाइलें पढ़ने की आवश्यकता (`~/Library/Containers/com.apple.Safari/Data`)...

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

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी HackTricks में विज्ञापित हो** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>
