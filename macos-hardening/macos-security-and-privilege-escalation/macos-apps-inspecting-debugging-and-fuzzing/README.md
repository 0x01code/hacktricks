# macOS ऐप्स - निरीक्षण, डिबगिंग और फज़िंग

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS रेड टीम एक्सपर्ट)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **HackTricks** के [**github रेपोज़**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें.

</details>

## स्टैटिक विश्लेषण

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
{% endcode %}

### jtool2

यह टूल **codesign**, **otool**, और **objdump** के लिए एक **प्रतिस्थापन** के रूप में उपयोग किया जा सकता है, और कुछ अतिरिक्त सुविधाएँ प्रदान करता है। [**यहाँ डाउनलोड करें**](http://www.newosxbook.com/tools/jtool.html) या `brew` के साथ इंस्टॉल करें।
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
**`Codesign`** का उपयोग **macOS** में किया जा सकता है जबकि **`ldid`** का उपयोग **iOS** में किया जा सकता है।
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
### SuspiciousPackage

[**SuspiciousPackage**](https://mothersruin.com/software/SuspiciousPackage/get.html) एक उपकरण है जो **.pkg** फाइलों (इंस्टॉलर्स) का निरीक्षण करने में सहायक है और इसे इंस्टॉल करने से पहले देख सकते हैं कि अंदर क्या है।\
इन इंस्टॉलर्स में `preinstall` और `postinstall` बैश स्क्रिप्ट्स होती हैं जिनका दुरुपयोग मैलवेयर लेखक आमतौर पर **मैलवेयर** को **बनाए** **रखने** के लिए करते हैं।

### hdiutil

यह उपकरण Apple डिस्क इमेजेज (**.dmg**) फाइलों को **माउंट** करने की अनुमति देता है ताकि उन्हें कुछ भी चलाने से पहले निरीक्षण किया जा सके:
```bash
hdiutil attach ~/Downloads/Firefox\ 58.0.2.dmg
```
यह `/Volumes` में माउंट किया जाएगा

### Objective-C

#### मेटाडेटा

{% hint style="danger" %}
ध्यान दें कि Objective-C में लिखे गए प्रोग्राम [Mach-O binaries](../macos-files-folders-and-binaries/universal-binaries-and-mach-o-format.md) में **संकलित** होने पर भी अपने क्लास घोषणाओं को **बनाए** **रखते** हैं। ऐसी क्लास घोषणाएँ **शामिल** करती हैं:
{% endhint %}

* क्लास का नाम
* क्लास के तरीके
* क्लास के इंस्टेंस वेरिएबल्स

आप इस जानकारी को [**class-dump**](https://github.com/nygard/class-dump) का उपयोग करके प्राप्त कर सकते हैं:
```bash
class-dump Kindle.app
```
#### फंक्शन कॉलिंग

जब एक फंक्शन को एक बाइनरी में कॉल किया जाता है जो ऑब्जेक्टिव-सी का उपयोग करता है, तो संकलित कोड उस फंक्शन को कॉल करने के बजाय, यह **`objc_msgSend`** को कॉल करेगा। जो अंतिम फंक्शन को कॉल करेगा:

![](<../../../.gitbook/assets/image (560).png>)

इस फंक्शन की अपेक्षित पैरामीटर्स ये हैं:

* पहला पैरामीटर (**self**) "एक पॉइंटर है जो **क्लास के इंस्टेंस को इंगित करता है जिसे मैसेज प्राप्त करना है**"। या अधिक सरल शब्दों में, यह वह ऑब्जेक्ट है जिस पर मेथड को इन्वोक किया जा रहा है। अगर मेथड एक क्लास मेथड है, तो यह क्लास ऑब्जेक्ट का एक इंस्टेंस होगा (पूरे के रूप में), जबकि एक इंस्टेंस मेथड के लिए, self एक इंस्टेंशिएटेड इंस्टेंस को इंगित करेगा।
* दूसरा पैरामीटर, (**op**), "वह सेलेक्टर है जो मैसेज को हैंडल करने वाले मेथड का है"। फिर से, अधिक सरल शब्दों में, यह सिर्फ **मेथड का नाम है।**
* शेष पैरामीटर्स वे **मान हैं जो मेथड द्वारा आवश्यक हैं** (op)।

| **आर्ग्युमेंट**      | **रजिस्टर**                                                    | **(के लिए) objc\_msgSend**                                |
| ----------------- | --------------------------------------------------------------- | ------------------------------------------------------ |
| **पहला आर्ग्युमेंट**  | **rdi**                                                         | **self: ऑब्जेक्ट जिस पर मेथड को इन्वोक किया जा रहा है** |
| **दूसरा आर्ग्युमेंट**  | **rsi**                                                         | **op: मेथड का नाम**                                     |
| **तीसरा आर्ग्युमेंट**  | **rdx**                                                         | **मेथड का पहला आर्ग्युमेंट**                           |
| **चौथा आर्ग्युमेंट**  | **rcx**                                                         | **मेथड का दूसरा आर्ग्युमेंट**                           |
| **पांचवां आर्ग्युमेंट**  | **r8**                                                          | **मेथड का तीसरा आर्ग्युमेंट**                           |
| **छठा आर्ग्युमेंट**  | **r9**                                                          | **मेथड का चौथा आर्ग्युमेंट**                           |
| **सातवां+ आर्ग्युमेंट** | <p><strong>rsp+</strong><br><strong>(स्टैक पर)</strong></p> | **मेथड का पांचवां+ आर्ग्युमेंट**                        |

### Swift

Swift बाइनरीज के साथ, चूंकि ऑब्जेक्टिव-सी संगतता है, कभी-कभी आप [class-dump](https://github.com/nygard/class-dump/) का उपयोग करके डिक्लेरेशन्स निकाल सकते हैं लेकिन हमेशा नहीं।

**`jtool -l`** या **`otool -l`** कमांड लाइन्स के साथ यह संभव है कि कई सेक्शन्स मिल सकते हैं जो **`__swift5`** प्रीफिक्स से शुरू होते हैं:
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
इन खंडों में संग्रहीत जानकारी के बारे में आप [**इस ब्लॉग पोस्ट में और जानकारी पा सकते हैं**](https://knight.sc/reverse%20engineering/2019/07/17/swift-metadata.html).

इसके अलावा, **Swift बाइनरीज में सिंबल्स हो सकते हैं** (उदाहरण के लिए लाइब्रेरीज को अपने फंक्शन्स को कॉल करने के लिए सिंबल्स स्टोर करने की जरूरत होती है). **सिंबल्स आमतौर पर फंक्शन नाम की जानकारी और अट्रिब्यूट को एक बदसूरत तरीके से रखते हैं**, इसलिए वे बहुत उपयोगी होते हैं और "**डिमैंगलर्स**" होते हैं जो मूल नाम प्राप्त कर सकते हैं:
```bash
# Ghidra plugin
https://github.com/ghidraninja/ghidra_scripts/blob/master/swift_demangler.py

# Swift cli
swift demangle
```
### पैक्ड बाइनरीज़

* उच्च एंट्रोपी की जांच करें
* स्ट्रिंग्स की जांच करें (यदि समझने योग्य स्ट्रिंग्स लगभग नहीं हैं, पैक्ड)
* MacOS के लिए UPX पैकर एक सेक्शन उत्पन्न करता है जिसे "\_\_XHDR" कहा जाता है

## डायनामिक विश्लेषण

{% hint style="warning" %}
ध्यान दें कि बाइनरीज़ को डीबग करने के लिए, **SIP को अक्षम करना होगा** (`csrutil disable` या `csrutil enable --without debug`) या बाइनरीज़ को एक अस्थायी फोल्डर में कॉपी करें और **हस्ताक्षर को हटाएं** `codesign --remove-signature <binary-path>` या बाइनरी की डीबगिंग की अनुमति दें (आप [यह स्क्रिप्ट](https://gist.github.com/carlospolop/a66b8d72bb8f43913c4b5ae45672578b) का उपयोग कर सकते हैं)
{% endhint %}

{% hint style="warning" %}
ध्यान दें कि **सिस्टम बाइनरीज़ को इंस्ट्रूमेंट करने के लिए**, (जैसे कि `cloudconfigurationd`) macOS पर, **SIP को अक्षम करना होगा** (केवल हस्ताक्षर को हटाना काम नहीं करेगा)।
{% endhint %}

### यूनिफाइड लॉग्स

MacOS बहुत सारे लॉग्स उत्पन्न करता है जो एक एप्लिकेशन चलाते समय **यह समझने में** बहुत उपयोगी हो सकते हैं **कि वह क्या कर रहा है**।

इसके अलावा, कुछ लॉग्स में `<private>` टैग होता है जो कुछ **उपयोगकर्ता** या **कंप्यूटर** **पहचान योग्य** जानकारी को **छिपाने** के लिए होता है। हालांकि, इस जानकारी को प्रकट करने के लिए **एक प्रमाणपत्र स्थापित करना संभव है**। [**यहां**](https://superuser.com/questions/1532031/how-to-show-private-data-in-macos-unified-log) से व्याख्याओं का पालन करें।

### Hopper

#### बाएँ पैनल

Hopper के बाएँ पैनल में बाइनरी के प्रतीकों (**Labels**), प्रक्रियाओं और फंक्शनों की सूची (**Proc**) और स्ट्रिंग्स (**Str**) देखी जा सकती हैं। ये सभी स्ट्रिंग्स नहीं हैं लेकिन Mac-O फाइल के कई हिस्सों में परिभाषित वे हैं (जैसे _cstring या_ `objc_methname`)।

#### मध्य पैनल

मध्य पैनल में आप **डिसासेंबल्ड कोड** देख सकते हैं। और आप इसे **कच्चे** डिसासेंबल, **ग्राफ** के रूप में, **डीकंपाइल्ड** और **बाइनरी** के रूप में देख सकते हैं, संबंधित आइकन पर क्लिक करके:

<figure><img src="../../../.gitbook/assets/image (2) (6).png" alt=""><figcaption></figcaption></figure>

कोड ऑब्जेक्ट पर राइट क्लिक करके आप उस ऑब्जेक्ट के **संदर्भों को देख सकते हैं** या उसका नाम बदल सकते हैं (यह डीकंपाइल्ड प्स्यूडोकोड में काम नहीं करता):

<figure><img src="../../../.gitbook/assets/image (1) (1) (2).png" alt=""><figcaption></figcaption></figure>

इसके अलावा, **मध्य नीचे आप पायथन कमांड्स लिख सकते हैं**।

#### दाएँ पैनल

दाएँ पैनल में आप जैसे **नेविगेशन हिस्ट्री** (ताकि आप जान सकें कि आप वर्तमान स्थिति में कैसे पहुंचे), **कॉल ग्राफ** जहां आप देख सकते हैं सभी **फंक्शन्स जो इस फंक्शन को कॉल करते हैं** और सभी फंक्शन्स जो **इस फंक्शन को कॉल करते हैं**, और **लोकल वेरिएबल्स** की जानकारी देख सकते हैं।

### dtrace

यह उपयोगकर्ताओं को एप्लिकेशन्स तक एक अत्यंत **निम्न स्तर** पर पहुंच प्रदान करता है और उपयोगकर्ताओं को **प्रोग्राम्स** को **ट्रेस** करने और यहां तक कि उनके निष्पादन प्रवाह को बदलने की क्षमता प्रदान करता है। Dtrace **प्रोब्स** का उपयोग करता है जो **कर्नेल के चारों ओर रखे जाते हैं** और स्थानों पर होते हैं जैसे कि सिस्टम कॉल्स की शुरुआत और अंत।

DTrace **`dtrace_probe_create`** फंक्शन का उपयोग करके प्रत्येक सिस्टम कॉल के लिए एक प्रोब बनाता है। ये प्रोब्स प्रत्येक सिस्टम कॉल के **प्रवेश और निकास बिंदु पर** चालू किए जा सकते हैं। DTrace के साथ इंटरैक्शन /dev/dtrace के माध्यम से होता है जो केवल रूट उपयोगकर्ता के लिए उपलब्ध है।

{% hint style="success" %}
SIP सुरक्षा को पूरी तरह से अक्षम किए बिना Dtrace को सक्षम करने के लिए आप रिकवरी मोड पर निम्नलिखित कमांड निष्पादित कर सकते हैं: `csrutil enable --without dtrace`

आप **`dtrace`** या **`dtruss`** बाइनरीज़ भी कर सकते हैं जो **आपने कंपाइल की हैं**।
{% endhint %}

dtrace के उपलब्ध प्रोब्स को प्राप्त किया जा सकता है:
```bash
dtrace -l | head
ID   PROVIDER            MODULE                          FUNCTION NAME
1     dtrace                                                     BEGIN
2     dtrace                                                     END
3     dtrace                                                     ERROR
43    profile                                                     profile-97
44    profile                                                     profile-199
```
प्रोब नाम में चार भाग होते हैं: प्रोवाइडर, मॉड्यूल, फंक्शन, और नाम (`fbt:mach_kernel:ptrace:entry`). यदि आप किसी भाग का नाम नहीं बताते हैं, तो Dtrace उस भाग को वाइल्डकार्ड के रूप में लागू करेगा।

DTrace को कॉन्फ़िगर करने के लिए प्रोब्स को सक्रिय करने और उनके ट्रिगर होने पर क्या क्रियाएं करनी हैं, इसे निर्दिष्ट करने के लिए हमें D भाषा का उपयोग करना होगा।

अधिक विस्तृत व्याख्या और अधिक उदाहरण [https://illumos.org/books/dtrace/chp-intro.html](https://illumos.org/books/dtrace/chp-intro.html) पर पाए जा सकते हैं।

#### उदाहरण

**DTrace स्क्रिप्ट्स उपलब्ध** होने की सूची के लिए `man -k dtrace` चलाएं। उदाहरण: `sudo dtruss -n binary`

* पंक्ति
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

आप इसका उपयोग **SIP सक्रिय** होने के साथ भी कर सकते हैं
```bash
ktrace trace -s -S -t c -c ls | grep "ls("
```
### ProcessMonitor

[**ProcessMonitor**](https://objective-see.com/products/utilities.html#ProcessMonitor) एक बहुत उपयोगी उपकरण है जो जांच करता है कि कोई प्रक्रिया कौन से प्रक्रिया संबंधित क्रियाएं कर रही है (उदाहरण के लिए, यह मॉनिटर करता है कि कोई प्रक्रिया कौन से नए प्रक्रियाएं बना रही है)।

### SpriteTree

[**SpriteTree**](https://themittenmac.com/tools/) एक उपकरण है जो प्रक्रियाओं के बीच संबंधों को प्रिंट करता है।\
आपको अपने मैक को **`sudo eslogger fork exec rename create > cap.json`** जैसे कमांड से मॉनिटर करना होगा (इसे चलाने वाले टर्मिनल को FDA की आवश्यकता होती है)। और फिर आप इस उपकरण में json लोड कर सकते हैं ताकि सभी संबंधों को देख सकें:

<figure><img src="../../../.gitbook/assets/image (710).png" alt="" width="375"><figcaption></figcaption></figure>

### FileMonitor

[**FileMonitor**](https://objective-see.com/products/utilities.html#FileMonitor) फाइल इवेंट्स (जैसे कि निर्माण, संशोधन, और हटाने) को मॉनिटर करने की अनुमति देता है, और ऐसी घटनाओं के बारे में विस्तृत जानकारी प्रदान करता है।

### Crescendo

[**Crescendo**](https://github.com/SuprHackerSteve/Crescendo) एक GUI उपकरण है जिसका रूप और अनुभव Windows उपयोगकर्ता Microsoft Sysinternal के _Procmon_ से परिचित हो सकते हैं। यह आपको सभी प्रकार की घटनाओं की रिकॉर्डिंग शुरू और रोकने, उन्हें श्रेणियों (फाइल, प्रक्रिया, नेटवर्क, आदि) द्वारा फ़िल्टर करने और रिकॉर्ड की गई घटनाओं को json फाइल के रूप में सहेजने की अनुमति देता है।

### Apple Instruments

[**Apple Instruments**](https://developer.apple.com/library/archive/documentation/Performance/Conceptual/CellularBestPractices/Appendix/Appendix.html) Xcode के डेवलपर टूल्स का हिस्सा हैं – इनका उपयोग एप्लिकेशन प्रदर्शन की निगरानी, मेमोरी लीक की पहचान, और फाइलसिस्टम गतिविधि को ट्रैक करने के लिए किया जाता है।

![](<../../../.gitbook/assets/image (15).png>)

### fs\_usage

प्रक्रियाओं द्वारा किए गए क्रियाओं का अनुसरण करने की अनुमति देता है:
```bash
fs_usage -w -f filesys ls #This tracks filesystem actions of proccess names containing ls
fs_usage -w -f network curl #This tracks network actions
```
### TaskExplorer

[**Taskexplorer**](https://objective-see.com/products/taskexplorer.html) बाइनरी द्वारा प्रयुक्त **लाइब्रेरीज**, उपयोग में आने वाली **फाइल्स** और **नेटवर्क** कनेक्शन्स को देखने के लिए उपयोगी है।\
यह बाइनरी प्रोसेसेस को **virustotal** के खिलाफ जांचता है और बाइनरी के बारे में जानकारी दिखाता है।

## PT\_DENY\_ATTACH <a href="#page-title" id="page-title"></a>

[**इस ब्लॉग पोस्ट**](https://knight.sc/debugging/2019/06/03/debugging-apple-binaries-that-use-pt-deny-attach.html) में आप एक उदाहरण पा सकते हैं कि कैसे **चल रहे डेमन** को **डीबग** करना है जिसने **`PT_DENY_ATTACH`** का उपयोग किया है ताकि डीबगिंग को रोका जा सके भले ही SIP अक्षम हो।

### lldb

**lldb** **macOS** बाइनरी **डीबगिंग** के लिए मानक उपकरण है।
```bash
lldb ./malware.bin
lldb -p 1122
lldb -n malware.bin
lldb -n malware.bin --waitfor
```
आप **`.lldbinit`** नामक फाइल बनाकर अपने होम फोल्डर में निम्नलिखित पंक्ति के साथ lldb का उपयोग करते समय intel flavour सेट कर सकते हैं:
```bash
settings set target.x86-disassembly-flavor intel
```
{% hint style="warning" %}
lldb के अंदर, प्रोसेस को `process save-core` के साथ डंप करें।
{% endhint %}

<table data-header-hidden><thead><tr><th width="225"></th><th></th></tr></thead><tbody><tr><td><strong>(lldb) कमांड</strong></td><td><strong>विवरण</strong></td></tr><tr><td><strong>run (r)</strong></td><td>निष्पादन शुरू करना, जो एक ब्रेकपॉइंट तक पहुंचने या प्रोसेस समाप्त होने तक अनवरत जारी रहेगा।</td></tr><tr><td><strong>continue (c)</strong></td><td>डीबग किए गए प्रोसेस का निष्पादन जारी रखें।</td></tr><tr><td><strong>nexti (n / ni)</strong></td><td>अगला निर्देश निष्पादित करें। यह कमांड फंक्शन कॉल्स को छोड़ देगा।</td></tr><tr><td><strong>stepi (s / si)</strong></td><td>अगला निर्देश निष्पादित करें। nexti कमांड के विपरीत, यह कमांड फंक्शन कॉल्स में कदम रखेगा।</td></tr><tr><td><strong>finish (f)</strong></td><td>वर्तमान फंक्शन ("फ्रेम") में शेष निर्देशों को निष्पादित करें और रुकें।</td></tr><tr><td><strong>control + c</strong></td><td>निष्पादन रोकें। यदि प्रोसेस को run (r) या continue (c) किया गया है, तो यह प्रोसेस को वर्तमान में निष्पादित हो रहे स्थान पर रोक देगा।</td></tr><tr><td><strong>breakpoint (b)</strong></td><td><p>b main #कोई भी फंक्शन जिसका नाम main हो</p><p>b &#x3C;binname>`main #बिन का मुख्य फंक्शन</p><p>b set -n main --shlib &#x3C;lib_name> #इंगित किए गए बिन का मुख्य फंक्शन</p><p>b -[NSDictionary objectForKey:]</p><p>b -a 0x0000000100004bd9</p><p>br l #ब्रेकपॉइंट सूची</p><p>br e/dis &#x3C;num> #ब्रेकपॉइंट सक्षम/अक्षम करें</p><p>breakpoint delete &#x3C;num></p></td></tr><tr><td><strong>help</strong></td><td><p>help breakpoint #breakpoint कमांड की मदद प्राप्त करें</p><p>help memory write #मेमोरी में लिखने के लिए मदद प्राप्त करें</p></td></tr><tr><td><strong>reg</strong></td><td><p>reg read</p><p>reg read $rax</p><p>reg read $rax --format &#x3C;<a href="https://lldb.llvm.org/use/variable.html#type-format">format</a>></p><p>reg write $rip 0x100035cc0</p></td></tr><tr><td><strong>x/s &#x3C;reg/memory address></strong></td><td>मेमोरी को एक नल-टर्मिनेटेड स्ट्रिंग के रूप में प्रदर्शित करें।</td></tr><tr><td><strong>x/i &#x3C;reg/memory address></strong></td><td>मेमोरी को असेंबली निर्देश के रूप में प्रदर्शित करें।</td></tr><tr><td><strong>x/b &#x3C;reg/memory address></strong></td><td>मेमोरी को बाइट के रूप में प्रदर्शित करें।</td></tr><tr><td><strong>print object (po)</strong></td><td><p>यह पैरामीटर द्वारा संदर्भित ऑब्जेक्ट को प्रिंट करेगा</p><p>po $raw</p><p><code>{</code></p><p><code>dnsChanger = {</code></p><p><code>"affiliate" = "";</code></p><p><code>"blacklist_dns" = ();</code></p><p>ध्यान दें कि Apple के अधिकांश Objective-C APIs या मेथड्स ऑब्जेक्ट्स लौटाते हैं, और इसलिए उन्हें "print object" (po) कमांड के माध्यम से प्रदर्शित किया जाना चाहिए। यदि po सार्थक आउटपुट नहीं देता है तो <code>x/b</code> का उपयोग करें</p></td></tr><tr><td><strong>memory</strong></td><td>memory read 0x000....<br>memory read $x0+0xf2a<br>memory write 0x100600000 -s 4 0x41414141 #उस पते में AAAA लिखें<br>memory write -f s $rip+0x11f+7 "AAAA" #पते में AAAA लिखें</td></tr><tr><td><strong>disassembly</strong></td><td><p>dis #वर्तमान फंक्शन को डिसासेंबल करें</p><p>dis -n &#x3C;funcname> #फंक्शन को डिसासेंबल करें</p><p>dis -n &#x3C;funcname> -b &#x3C;basename> #फंक्शन को डिसासेंबल करें<br>dis -c 6 #6 लाइनों को डिसासेंबल करें<br>dis -c 0x100003764 -e 0x100003768 # एक पते से दूसरे पते तक<br>dis -p -c 4 # वर्तमान पते से डिसासेंबलिंग शुरू करें</p></td></tr><tr><td><strong>parray</strong></td><td>parray 3 (char **)$x1 # x1 रजिस्टर में 3 घटकों की एरे की जांच करें</td></tr></tbody></table>

{% hint style="info" %}
**`objc_sendMsg`** फंक्शन को कॉल करते समय, **rsi** रजिस्टर **मेथड के नाम** को एक नल-टर्मिनेटेड ("C") स्ट्रिंग के रूप में रखता है। नाम को lldb के माध्यम से प्रिंट करने के लिए करें:

`(lldb) x/s $rsi: 0x1000f1576: "startMiningWithPort:password:coreCount:slowMemory:currency:"`

`(lldb) print (char*)$rsi:`\
`(char *) $1 = 0x00000001000f1576 "startMiningWithPort:password:coreCount:slowMemory:currency:"`

`(lldb) reg read $rsi: rsi = 0x00000001000f1576 "startMiningWithPort:password:coreCount:slowMemory:currency:"`
{% endhint %}

### एंटी-डायनामिक विश्लेषण

#### VM पता लगाना

* कमांड **`sysctl hw.model`** "Mac" लौटाता है जब **होस्ट MacOS होता है** लेकिन VM होने पर कुछ अलग लौटाता है।
* **`hw.logicalcpu`** और **`hw.physicalcpu`** के मानों के साथ खेलकर कुछ मैलवेयर VM होने का पता लगाने की कोशिश करते हैं।
* कुछ मैलवेयर MAC पते (00:50:56) के आधार पर **VMware** होने का भी **पता लगा** सकते हैं।
* यह भी संभव है कि एक साधारण कोड के साथ **प्रोसेस के डीबग होने का पता लगाया जा सके**:
* `if(P_TRACED == (info.kp_proc.p_flag & P_TRACED)){ //प्रोसेस डीबग हो रहा है }`
* यह **`ptrace`** सिस्टम कॉल को भी **`PT_DENY_ATTACH`** फ्लैग के साथ आमंत्रित कर सकता है। यह एक डीबगर को जुड़ने और ट्रेसिंग से **रोकता** है।
* आप जांच सकते हैं कि क्या **`sysctl`** या **`ptrace`** फंक्शन को **आयात** किया जा रहा है (लेकिन मैलवेयर इसे गतिशील रूप से आयात कर सकता है)
* इस लेख में जैसा कि नोट किया गया है, “[Defeating Anti-Debug Techniques: macOS ptrace variants](https://alexomara.com/blog/defeating-anti-debug-techniques-macos-ptrace-variants/)” :\
“_संदेश Process # exited with **status = 45 (0x0000002d)** आमतौर पर यह संकेत देता है कि डीबग लक्ष्य **PT\_DENY\_ATTACH** का उपयोग कर रहा है_”

## फज़िंग

### [ReportCrash](https://ss64.com/osx/reportcrash.html)

ReportCrash **क्रैश हो रहे प्रोसेसों का विश्लेषण करता है और एक क्रैश रिपोर्ट को डिस्क पर सहेजता है**। एक क्रैश रिपोर्ट में ऐसी जानकारी होती है जो एक डेवलपर को क्रैश के कारण का निदान करने में **मदद कर सकती है**।\
एप्लिकेशनों और अन्य प्रोसेसों के लिए जो प्रति-उपयोगकर्ता launchd संदर्भ में चल रहे हैं, ReportCrash एक LaunchAgent के रूप में चलता है और उपयोगकर्ता के `~/Library/Logs/DiagnosticReports/` में क्रैश रिपोर्ट सहेजता है।\
डेमन्स, अन्य प्रोसेस जो सिस्टम launchd संदर्भ में चल रहे हैं और अन्य विशेषाधिकार प्राप्त प्रोसेसों के लिए, ReportCrash एक LaunchDaemon के रूप में चलता है और सिस्टम के `/Library/Logs/DiagnosticReports` में क्रैश रिपोर्ट सहेजता है।

यदि आप क्रैश रिपोर्ट्स के Apple को भेजे जाने के बारे में चिंतित हैं तो आप उन्हें अक्षम कर सकते हैं। यदि नहीं, तो क्रैश रिपोर्ट्स यह **पता लगाने में उपयोगी हो सकती हैं कि एक सर्वर कैसे क्रैश हुआ**।
```bash
#To disable crash reporting:
launchctl unload -w /System/Library/LaunchAgents/com.apple.ReportCrash.plist
sudo launchctl unload -w /System/Library/LaunchDaemons/com.apple.ReportCrash.Root.plist

#To re-enable crash reporting:
launchctl load -w /System/Library/LaunchAgents/com.apple.ReportCrash.plist
sudo launchctl load -w /System/Library/LaunchDaemons/com.apple.ReportCrash.Root.plist
```
### स्लीप

MacOS में fuzzing करते समय यह सुनिश्चित करना महत्वपूर्ण है कि Mac स्लीप मोड में न जाए:

* systemsetup -setsleep Never
* pmset, System Preferences
* [KeepingYouAwake](https://github.com/newmarcel/KeepingYouAwake)

#### SSH डिस्कनेक्ट

यदि आप SSH कनेक्शन के माध्यम से fuzzing कर रहे हैं, तो यह सुनिश्चित करना महत्वपूर्ण है कि सेशन डिस्कनेक्ट न हो। इसलिए sshd_config फाइल को निम्नलिखित के साथ बदलें:

* TCPKeepAlive Yes
* ClientAliveInterval 0
* ClientAliveCountMax 0
```bash
sudo launchctl unload /System/Library/LaunchDaemons/ssh.plist
sudo launchctl load -w /System/Library/LaunchDaemons/ssh.plist
```
### आंतरिक हैंडलर्स

**निम्नलिखित पृष्ठ की जाँच करें** यह जानने के लिए कि आप कैसे पता लगा सकते हैं कि कौन सा ऐप निर्दिष्ट स्कीम या प्रोटोकॉल को **संभालने का जिम्मेदार है:**

{% content-ref url="../macos-file-extension-apps.md" %}
[macos-file-extension-apps.md](../macos-file-extension-apps.md)
{% endcontent-ref %}

### नेटवर्क प्रक्रियाओं का गणना

यह जानना दिलचस्प है कि कौन सी प्रक्रियाएं नेटवर्क डेटा का प्रबंधन कर रही हैं:
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
### फज़र्स

#### [AFL++](https://github.com/AFLplusplus/AFLplusplus)

CLI टूल्स के लिए काम करता है

#### [Litefuzz](https://github.com/sec-tools/litefuzz)

यह macOS GUI टूल्स के साथ "**बस काम करता है**". ध्यान दें कि कुछ macOS ऐप्स में कुछ विशेष आवश्यकताएँ होती हैं जैसे कि अद्वितीय फ़ाइल नाम, सही एक्सटेंशन, फ़ाइलों को सैंडबॉक्स से पढ़ने की ज़रूरत (`~/Library/Containers/com.apple.Safari/Data`)...

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
### और MacOS Fuzzing की जानकारी

* [https://www.youtube.com/watch?v=T5xfL9tEg44](https://www.youtube.com/watch?v=T5xfL9tEg44)
* [https://github.com/bnagy/slides/blob/master/OSXScale.pdf](https://github.com/bnagy/slides/blob/master/OSXScale.pdf)
* [https://github.com/bnagy/francis/tree/master/exploitaben](https://github.com/bnagy/francis/tree/master/exploitaben)
* [https://github.com/ant4g0nist/crashwrangler](https://github.com/ant4g0nist/crashwrangler)

## संदर्भ

* [**OS X Incident Response: Scripting and Analysis**](https://www.amazon.com/OS-Incident-Response-Scripting-Analysis-ebook/dp/B01FHOHHVS)
* [**https://www.youtube.com/watch?v=T5xfL9tEg44**](https://www.youtube.com/watch?v=T5xfL9tEg44)
* [**https://taomm.org/vol1/analysis.html**](https://taomm.org/vol1/analysis.html)

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग प्राप्त करें**](https://peass.creator-spring.com)
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में शामिल हों या [**telegram group**](https://t.me/peass) या **Twitter** 🐦 पर **मुझे फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें.

</details>
