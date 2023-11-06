# macOS Apps - निरीक्षण, डीबगिंग और फजिंग

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप एक **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **फॉलो** करें मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को।**

</details>

## स्थिर विश्लेषण

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

यह टूल **codesign**, **otool**, और **objdump** के लिए एक **प्रतिस्थापन** के रूप में उपयोग किया जा सकता है, और कुछ अतिरिक्त सुविधाएं प्रदान करता है। [**यहां डाउनलोड करें**](http://www.newosxbook.com/tools/jtool.html) या `brew` के साथ इंस्टॉल करें।
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
**`Codesign`** को **macOS** में पाया जा सकता है जबकि **`ldid`** को **iOS** में पाया जा सकता है।
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

[**SuspiciousPackage**](https://mothersruin.com/software/SuspiciousPackage/get.html) एक उपकरण है जो **.pkg** फ़ाइलें (इंस्टॉलर) की जांच करने के लिए उपयोगी होता है और इंस्टॉल करने से पहले उसके अंदर क्या है देखने की सुविधा प्रदान करता है।\
इन इंस्टॉलर में `preinstall` और `postinstall` बैश स्क्रिप्ट होते हैं जिन्हें मैलवेयर लेखक आमतौर पर इस्तेमाल करते हैं ताकि मैलवेयर **को** **स्थायी** **रूप** **से** **संचालित** **किया** **जा** **सके**।

### hdiutil

यह उपकरण Apple डिस्क इमेज (**.dmg**) फ़ाइलों को **माउंट** करने की अनुमति देता है ताकि उन्हें कुछ चलाने से पहले जांचा जा सके:
```bash
hdiutil attach ~/Downloads/Firefox\ 58.0.2.dmg
```
यह `/Volumes` में माउंट हो जाएगा।

### Objective-C

#### मेटाडेटा

{% hint style="danger" %}
ध्यान दें कि Objective-C में लिखे गए प्रोग्राम [Mach-O बाइनरी](../macos-files-folders-and-binaries/universal-binaries-and-mach-o-format.md) में कक्षा घोषणाएं रखते हैं। ऐसी कक्षा घोषणाएं निम्नलिखित जानकारी को शामिल करती हैं:
{% endhint %}

* कक्षा
* कक्षा के विधियाँ
* कक्षा के उदाहरण चर

आप [**class-dump**](https://github.com/nygard/class-dump) का उपयोग करके इस जानकारी को प्राप्त कर सकते हैं:
```bash
class-dump Kindle.app
```
ध्यान दें कि इन नामों को अस्पष्ट किया जा सकता है ताकि बाइनरी का पुनर्वर्तन करना कठिन हो जाए।

#### फ़ंक्शन कॉलिंग

जब एक फ़ंक्शन को एक objective-C का उपयोग करने वाले बाइनरी में कॉल किया जाता है, तो कंपाइल किए गए कोड की बजाय वह फ़ंक्शन को **`objc_msgSend`** को कॉल करेगा। जो अंतिम फ़ंक्शन को कॉल करेगा:

![](<../../../.gitbook/assets/image (560).png>)

इस फ़ंक्शन की उम्मीद की जाने वाली पैरामीटर हैं:

* पहला पैरामीटर (**self**) "एक पॉइंटर है जो संदेश प्राप्त करने वाली कक्षा के उदाहरण की ओर पॉइंट करता है"। या और सरल शब्दों में कहें तो, यह वह ऑब्जेक्ट है जिस पर विधि को आमंत्रित किया जा रहा है। यदि विधि एक कक्षा विधि है, तो यह कक्षा ऑब्जेक्ट (संपूर्ण रूप में) का एक उदाहरण होगा, जबकि एक उदाहरण विधि के लिए, self कक्षा के एक निर्मित उदाहरण को ओब्जेक्ट के रूप में पॉइंट करेगा।
* दूसरा पैरामीटर (**op**) "संदेश को संभालने वाली विधि का चयनकर्ता" है। फिर से, और सरल शब्दों में कहें तो, यह बस विधि का **नाम है।**
* शेष पैरामीटर वे कोई **मान हैं जो विधि द्वारा आवश्यक हैं** (op)।

| **वादा**          | **रजिस्टर**                                                    | **(के लिए) objc\_msgSend**                                |
| ----------------- | --------------------------------------------------------------- | ------------------------------------------------------ |
| **1 वादा**  | **rdi**                                                         | **self: वह ऑब्जेक्ट है जिस पर विधि को आमंत्रित किया जा रहा है** |
| **2 वादा**  | **rsi**                                                         | **op: विधि का नाम**                             |
| **3 वादा**  | **rdx**                                                         | **विधि के लिए पहला वादा**                         |
| **4 वादा**  | **rcx**                                                         | **विधि के लिए दूसरा वादा**                         |
| **5 वादा**  | **r8**                                                          | **विधि के लिए तीसरा वादा**                         |
| **6 वादा**  | **r9**                                                          | **विधि के लिए चौथा वादा**                         |
| **7+ वादा** | <p><strong>rsp+</strong><br><strong>(स्टैक पर)</strong></p> | **विधि के लिए पांचवा+ वादा**                        |

### स्विफ्ट

स्विफ्ट बाइनरी के साथ, क्योंकि यहां Objective-C संगतता होती है, कभी-कभी आप [class-dump](https://github.com/nygard/class-dump/) का उपयोग करके घोषणाएँ निकाल सकते हैं लेकिन हमेशा नहीं।

**`jtool -l`** या **`otool -l`** कमांड लाइन के साथ आपको कई खंड मिल सकते हैं जो **`__swift5`** प्रीफ़िक्स के साथ शुरू होते हैं:
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
आप इस [**ब्लॉग पोस्ट में**](https://knight.sc/reverse%20engineering/2019/07/17/swift-metadata.html) इन सेक्शन में संग्रहित जानकारी के बारे में अधिक जानकारी पा सकते हैं।

इसके अलावा, **स्विफ्ट बाइनरी में प्रतीक हो सकते हैं** (उदाहरण के लिए पुस्तकालयों को प्रतीक संग्रहित करने की आवश्यकता होती है ताकि उनके कार्यों को बुलाया जा सके)। **प्रतीकों में आमतौर पर फ़ंक्शन के नाम और विशेषता की जानकारी होती है**, लेकिन वे बहुत ही असुंदर तरीके से होते हैं, इसलिए वे बहुत ही उपयोगी होते हैं और उनके पास "**डीमैंगलर्स"** होते हैं जो मूल नाम प्राप्त कर सकते हैं:
```bash
# Ghidra plugin
https://github.com/ghidraninja/ghidra_scripts/blob/master/swift_demangler.py

# Swift cli
swift demangle
```
### पैक किए गए बाइनरी

* उच्च एंट्रोपी की जांच करें
* स्ट्रिंग की जांच करें (क्या कोई समझने योग्य स्ट्रिंग नहीं है, पैक किया गया है)
* MacOS के लिए UPX पैकर एक धारा उत्पन्न करता है जिसका नाम "\_\_XHDR" होता है

## गतिशील विश्लेषण

{% hint style="warning" %}
ध्यान दें कि बाइनरी को डीबग करने के लिए, **SIP को अक्षम करना आवश्यक है** (`csrutil disable` या `csrutil enable --without debug`) या बाइनरी को एक अस्थायी फ़ोल्डर में कॉपी करके `codesign --remove-signature <binary-path>` के साथ **हस्ताक्षर को हटाएं** या बाइनरी के डीबगिंग की अनुमति दें (आप [इस स्क्रिप्ट](https://gist.github.com/carlospolop/a66b8d72bb8f43913c4b5ae45672578b) का उपयोग कर सकते हैं)
{% endhint %}

{% hint style="warning" %}
ध्यान दें कि macOS पर **सिस्टम बाइनरी** (जैसे कि `cloudconfigurationd`) को **इंस्ट्रुमेंट** करने के लिए, **SIP को अक्षम करना आवश्यक है** (केवल हस्ताक्षर को हटाना काम नहीं करेगा)।
{% endhint %}

### एकीकृत लॉग

MacOS बहुत सारे लॉग उत्पन्न करता है जो एक एप्लिकेशन चलाने पर बहुत उपयोगी हो सकते हैं और आपको समझने में मदद कर सकते हैं कि **यह क्या कर रहा है**।

इसके अलावा, कुछ लॉग होंगे जिनमें टैग `<private>` होगा जो कुछ **उपयोगकर्ता** या **कंप्यूटर** के **पहचाने जाने योग्य** जानकारी को **छिपाने** के लिए होते हैं। हालांकि, इस जानकारी को **खोलने के लिए एक प्रमाणपत्र स्थापित किया जा सकता है**। [**यहां**](https://superuser.com/questions/1532031/how-to-show-private-data-in-macos-unified-log) दिए गए विवरणों का पालन करें।

### हॉपर

#### बाएं पैनल

हॉपर के बाएं पैनल में आप बाइनरी के संकेत (**लेबल**) देख सकते हैं, प्रक्रियाओं और फ़ंक्शन (**प्रोसेस**) की सूची और स्ट्रिंग (**स्ट्रिंग्स**)। ये सभी स्ट्रिंग नहीं हैं लेकिन वे मैक-ओ फ़ाइल के कई हिस्सों में परिभाषित हैं (जैसे _cstring या `objc_methname`)

#### मध्य पैनल

मध्य पैनल में आप **डिसएंबल कोड** देख सकते हैं। और आप इसे **रॉ**, **ग्राफ**, **डिकॉम्पाइल्ड** और **बाइनरी** के रूप में देख सकते हैं जिसके लिए आप संबंधित आइकन पर क्लिक कर सकते हैं:

<figure><img src="../../../.gitbook/assets/image (2) (6).png" alt=""><figcaption></figcaption></figure>

कोड ऑब्जेक्ट पर दायां क्लिक करके आप **उस ऑब्जेक्ट के संदर्भ** देख सकते हैं या उसका नाम बदल सकते हैं (यह डिकॉम्पाइल्ड प्यूडोकोड में काम नहीं करता):

<figure><img src="../../../.gitbook/assets/image (1) (1) (2).png" alt=""><figcaption></figcaption></figure>

इसके अलावा, **मध्य नीचे आप पायथन कमांड लिख सकते हैं**।

#### दाएं पैनल

दाएं पैनल में आप रोचक जानकारी देख सकते हैं जैसे **नेविगेशन इतिहास** (ताकि आप जान सकें कि आप वर्तमान स्थिति तक कैसे पहुंचे), **कॉल ग्राफ** जहां आप सभी **फ़ंक्शन देख सकते हैं जो इस फ़ंक्शन को कॉल करते हैं** और उन सभी फ़ंक्शनों को जो **यह फ़ंक्शन कॉल करता है**, और **स्थानीय चर** जानकारी।

### dtrace

यह उपयोगकर्ताओं को एक अत्यंत **निम्न स्तर पर** एप्लिकेशन तक पहुंच देता है और उपयोगकर्ताओं को **प्रोग्राम का पता लगाने** और उनके निष्पादन फ्लो को **बदलने** का एक तरीका प्रदान करता है। Dtrace उपयोगकर्ताओं को **प्रोब्स** का उपयोग करता है जो **कर्नल के भीतर रखे जाते हैं** और सिस्टम कॉल के आरंभ और समाप्ति जैसे स्थानों पर स्थित होते हैं।

DTrace उपयोगकर्ता प्रत्येक सिस्टम कॉल के लिए एक प्रोब बनाने के लिए **`dtrace_probe_create`** फ़ंक्शन का उपयोग करता है। इन प्रोब्स को प्रत्येक सिस्टम कॉल के **एंट्री और एक्जिट प्वाइंट** में फायर किया जा सकता है। DTrace के साथ इंटरैक्शन /dev/dtrace के माध्यम से होता है जो केवल रूट उपयोगकर्ता के लिए ही उपलब्ध है।

{% hint style="success" %}
SIP सुरक्षा को पूरी तरह से अक्षम किए बिना Dtrace को सक्षम करने के लिए आप पुनर्प्राप्ति मोड पर निम्नलिखित को निष्पादित कर सकते हैं: `csrutil enable --without dtrace`

आप भी **आपने कंपाइल किए गए** बाइनरी को **`dtrace`** या **`dtruss`** कर सकते हैं।
{% endhint %}

dtrace के उपलब्ध प्रोब्स को निम्नलिखित के साथ प्राप्त किया जा सकता है:
```bash
dtrace -l | head
ID   PROVIDER            MODULE                          FUNCTION NAME
1     dtrace                                                     BEGIN
2     dtrace                                                     END
3     dtrace                                                     ERROR
43    profile                                                     profile-97
44    profile                                                     profile-199
```
प्रोब नाम चार भागों से मिलकर बनता है: प्रदाता, मॉड्यूल, फंक्शन और नाम (`fbt:mach_kernel:ptrace:entry`). यदि आप नाम के किसी भाग को निर्दिष्ट नहीं करते हैं, तो Dtrace उस भाग को वाइल्डकार्ड के रूप में लागू करेगा।

DTrace को प्रोब्स को सक्रिय करने और जब वे फायर होते हैं तो क्या कार्रवाई करनी है इसे कॉन्फ़िगर करने के लिए, हमें D भाषा का उपयोग करना होगा।

अधिक विस्तृत व्याख्या और अधिक उदाहरण [https://illumos.org/books/dtrace/chp-intro.html](https://illumos.org/books/dtrace/chp-intro.html) में मिल सकते हैं।

#### उदाहरण

**DTrace स्क्रिप्ट उपलब्ध** करने के लिए `man -k dtrace` चलाएं। उदाहरण: `sudo dtruss -n binary`

* लाइन में
```bash
#Count the number of syscalls of each running process
sudo dtrace -n 'syscall:::entry {@[execname] = count()}'
```
# स्क्रिप्ट

एक स्क्रिप्ट एक आदेश सेट होता है जो कंप्यूटर पर निर्दिष्ट कार्रवाई करता है। यह एक टेक्स्ट फ़ाइल होती है जिसमें एक या एक से अधिक आदेशों की सूची होती है, जिन्हें कंप्यूटर एक पदानुक्रम में पढ़ता है। स्क्रिप्ट भाषाएँ जैसे कि बैश, पायथन, पर्ल, रबी, पॉवरशेल, आदि का उपयोग करके लिखी जा सकती हैं।

स्क्रिप्ट का उपयोग करके, हैकर्स विभिन्न कार्रवाइयों को स्वचालित कर सकते हैं, जैसे कि फ़ाइलों को बनाना, संपादित करना, हटाना, नेटवर्क कमांड्स चलाना, डेटाबेस के साथ संचालन करना, आदि। स्क्रिप्ट लिखने के लिए, आपको उचित संदर्भ और भाषा का चयन करना होगा, और फिर आप आदेशों को लिख सकते हैं जो आपकी आवश्यकताओं को पूरा करेंगे।

एक स्क्रिप्ट को चलाने के लिए, आपको उसे एक्सीक्यूटेबल बनाना होगा। इसके लिए, आपको उसे एक फ़ाइल में सहेजना होगा और उसे एक्सीक्यूटेबल बनाने के लिए उचित अनुमतियों को सेट करना होगा। एक बार जब आपका स्क्रिप्ट एक्सीक्यूटेबल हो जाए, आप उसे चला सकते हैं और उसके द्वारा प्रदर्शित कार्रवाइयों का आनंद ले सकते हैं।

स्क्रिप्ट लिखने के दौरान, ध्यान देने योग्य बातें शामिल हैं:

- स्क्रिप्ट को सुरक्षित रखें और अनधिकृत उपयोग से बचाएं।
- उचित अनुमतियों को सेट करें ताकि केवल आप ही उसे चला सकें।
- अपने स्क्रिप्ट को नियमित रूप से सत्यापित करें और बग्स को ठीक करें।
- स्क्रिप्ट को अपडेट करें और नवीनतम सुरक्षा अपडेट को लागू करें।

स्क्रिप्ट लिखने के लिए अधिक जानकारी के लिए, आप विभिन्न संसाधनों, ट्यूटोरियल्स, और वेबसाइट्स का उपयोग कर सकते हैं जो आपको विभिन्न स्क्रिप्टिंग भाषाओं के बारे में शिक्षा और संसाधन प्रदान करते हैं।
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

dtruss is a command-line tool in macOS that allows you to trace and inspect system calls made by a running process. It can be used for debugging and analyzing the behavior of applications.

To use dtruss, you need to specify the process ID (PID) of the target application. You can find the PID using the `ps` command or by using tools like Activity Monitor.

Once you have the PID, you can run the following command to start tracing the system calls:

```
sudo dtruss -p <PID>
```

This will display a list of system calls made by the application, along with their arguments and return values. You can use this information to understand how the application interacts with the operating system and identify any potential vulnerabilities or issues.

dtruss is a powerful tool for inspecting and debugging macOS applications. However, it should be used responsibly and only on applications that you have permission to analyze.
```bash
dtruss -c ls #Get syscalls of ls
dtruss -c -p 1000 #get syscalls of PID 1000
```
### ktrace

आप इसे **SIP सक्रिय** होने के साथ भी उपयोग कर सकते हैं।
```bash
ktrace trace -s -S -t c -c ls | grep "ls("
```
### ProcessMonitor

[**ProcessMonitor**](https://objective-see.com/products/utilities.html#ProcessMonitor) एक बहुत ही उपयोगी उपकरण है जो प्रक्रिया से संबंधित कार्रवाइयों की जांच करने के लिए है (उदाहरण के लिए, यह जांचता है कि प्रक्रिया कौन सी नई प्रक्रियाएं बना रही है)।

### SpriteTree

[**SpriteTree**](https://themittenmac.com/tools/) एक उपकरण है जो प्रक्रियाओं के बीच संबंधों को प्रिंट करता है।\
आपको अपने मैक को एक ऐसे कमांड के साथ मॉनिटर करना होगा जैसे **`sudo eslogger fork exec rename create > cap.json`** (इसे चलाने के लिए टर्मिनल में FDA की आवश्यकता होती है)। और फिर आप इस उपकरण में json लोड कर सकते हैं ताकि आप सभी संबंधों को देख सकें:

<figure><img src="../../../.gitbook/assets/image (710).png" alt="" width="375"><figcaption></figcaption></figure>

### FileMonitor

[**FileMonitor**](https://objective-see.com/products/utilities.html#FileMonitor) फ़ाइल घटनाओं (जैसे निर्माण, संशोधन और हटाना) का मॉनिटर करने की अनुमति देता है और इन घटनाओं के बारे में विस्तृत जानकारी प्रदान करता है।

### Crescendo

[**Crescendo**](https://github.com/SuprHackerSteve/Crescendo) एक GUI उपकरण है जिसका लुक और फ़ील Windows उपयोगकर्ताओं को Microsoft Sysinternal's _Procmon_ से परिचित हो सकता है। इसकी मदद से आप सभी प्रकार की घटनाओं को रिकॉर्ड करना शुरू और बंद कर सकते हैं, उन्हें श्रेणियों (फ़ाइल, प्रक्रिया, नेटवर्क, आदि) के अनुसार फ़िल्टर कर सकते हैं और रिकॉर्ड की गई घटनाओं को json फ़ाइल के रूप में सहेज सकते हैं।

### Apple Instruments

[**Apple Instruments**](https://developer.apple.com/library/archive/documentation/Performance/Conceptual/CellularBestPractices/Appendix/Appendix.html) Xcode के डेवलपर उपकरणों का हिस्सा है - इसका उपयोग अनुप्रयोग क्षमता की मॉनिटरिंग, मेमोरी लीक की पहचान और फ़ाइलसिस्टम गतिविधि के ट्रैकिंग के लिए किया जाता है।

![](<../../../.gitbook/assets/image (15).png>)

### fs\_usage

प्रक्रियाओं द्वारा किए जाने वाले कार्रवाइयों का पालन करने की अनुमति देता है:
```bash
fs_usage -w -f filesys ls #This tracks filesystem actions of proccess names containing ls
fs_usage -w -f network curl #This tracks network actions
```
### TaskExplorer

[**Taskexplorer**](https://objective-see.com/products/taskexplorer.html) किसी बाइनरी द्वारा उपयोग की जाने वाली **पुस्तकालयों**, उसके द्वारा उपयोग की जाने वाली **फ़ाइलों** और **नेटवर्क** कनेक्शनों को देखने के लिए उपयोगी है।\
यह बाइनरी प्रक्रियाओं को **वायरसटोटल** के खिलाफ जांचता है और बाइनरी के बारे में जानकारी दिखाता है।

## PT\_DENY\_ATTACH <a href="#page-title" id="page-title"></a>

[**इस ब्लॉग पोस्ट**](https://knight.sc/debugging/2019/06/03/debugging-apple-binaries-that-use-pt-deny-attach.html) में आप एक उदाहरण पा सकते हैं कि कैसे **`PT_DENY_ATTACH`** का उपयोग करके डीबगिंग को रोकने के लिए SIP भी अक्षम होने के बावजूद एक चल रहे डेमन को **डीबग** कर सकते हैं।

### lldb

**lldb** मैकओएस बाइनरी **डीबगिंग** के लिए डी फैक्टो टूल है।
```bash
lldb ./malware.bin
lldb -p 1122
lldb -n malware.bin
lldb -n malware.bin --waitfor
```
आप लिनक्स डीबगर टूल lldb का उपयोग करते समय intel फ्लेवर सेट कर सकते हैं। इसके लिए अपने होम फ़ोल्डर में निम्नलिखित पंक्ति के साथ एक फ़ाइल **`.lldbinit`** बना सकते हैं:
```bash
settings set target.x86-disassembly-flavor intel
```
{% hint style="warning" %}
ल्लडब में, `process save-core` के साथ एक प्रक्रिया को डंप करें।
{% endhint %}

<table data-header-hidden><thead><tr><th width="225"></th><th></th></tr></thead><tbody><tr><td><strong>(lldb) कमांड</strong></td><td><strong>विवरण</strong></td></tr><tr><td><strong>run (r)</strong></td><td>एक्सिक्यूशन शुरू करें, जो एक ब्रेकपॉइंट पकड़ने या प्रक्रिया समाप्त होने तक बिना रुके जारी रहेगी।</td></tr><tr><td><strong>continue (c)</strong></td><td>डिबग की गई प्रक्रिया के एक्सिक्यूशन को जारी रखें।</td></tr><tr><td><strong>nexti (n / ni)</strong></td><td>अगले इंस्ट्रक्शन को एक्सिक्यूट करें। यह कमांड फंक्शन कॉल को छोड़ देगा।</td></tr><tr><td><strong>stepi (s / si)</strong></td><td>अगले इंस्ट्रक्शन को एक्सिक्यूट करें। इस कमांड के विपरीत, यह कमांड फंक्शन कॉल में स्टेप करेगा।</td></tr><tr><td><strong>finish (f)</strong></td><td>वर्तमान फंक्शन ("फ्रेम") में शेष इंस्ट्रक्शनों को एक्सिक्यूट करें और रोकें।</td></tr><tr><td><strong>control + c</strong></td><td>एक्सिक्यूशन को रोकें। यदि प्रक्रिया को चलाया गया है (r) या जारी रखा गया है (c), तो यह प्रक्रिया को वर्तमान में कहीं भी रोकेगा।</td></tr><tr><td><strong>breakpoint (b)</strong></td><td><p>b main # कोई फंक्शन जिसका नाम main है</p><p>b &#x3C;binname>`main # बिन का मुख्य फंक्शन</p><p>b set -n main --shlib &#x3C;lib_name> # निर्दिष्ट बिन का मुख्य फंक्शन</p><p>b -[NSDictionary objectForKey:]</p><p>b -a 0x0000000100004bd9</p><p>br l # ब्रेकपॉइंट सूची</p><p>br e/dis &#x3C;num> # ब्रेकपॉइंट सक्षम/अक्षम करें</p><p>breakpoint delete &#x3C;num></p></td></tr><tr><td><strong>help</strong></td><td><p>help breakpoint # ब्रेकपॉइंट कमांड की मदद प्राप्त करें</p><p>help memory write # मेमोरी में लिखने की मदद प्राप्त करें</p></td></tr><tr><td><strong>reg</strong></td><td><p>reg read</p><p>reg read $rax</p><p>reg read $rax --format &#x3C;<a href="https://lldb.llvm.org/use/variable.html#type-format">format</a>></p><p>reg write $rip 0x100035cc0</p></td></tr><tr><td><strong>x/s &#x3C;reg/memory address></strong></td><td>मेमोरी को एक नल-समाप्त ("सी") स्ट्रिंग के रूप में प्रदर्शित करें।</td></tr><tr><td><strong>x/i &#x3C;reg/memory address></strong></td><td>मेमोरी को एसेंबली इंस्ट्रक्शन के रूप में प्रदर्शित करें।</td></tr><tr><td><strong>x/b &#x3C;reg/memory address></strong></td><td>मेमोरी को बाइट के रूप में प्रदर्शित करें।</td></tr><tr><td><strong>print object (po)</strong></td><td><p>इससे पैरामीटर द्वारा संदर्भित ऑब्जेक्ट प्रिंट होगा</p><p>po $raw</p><p><code>{</code></p><p><code>dnsChanger = {</code></p><p><code>"affiliate" = "";</code></p><p><code>"blacklist_dns" = ();</code></p><p>ध्यान दें कि अधिकांश Apple के Objective-C API या मेथड ऑब्जेक्ट वापस करते हैं, और इसलिए "प्रिंट ऑब्जेक्ट" (po) कमांड के माध्यम से प्रदर्शित किए जाने चाहिए। यदि po सार्थक आउटपुट नहीं प्रदर्शित करता है, तो <code>x/b</code> का उपयोग करें</p></td></tr><tr><td><strong>memory</strong></td><td>memory read 0x000....<br>memory read $x0+0xf2a<br>memory write 0x100600000 -s 4 0x41414141 # उस पते में AAAA लिखें<br>memory write -f s $rip+0x11f+7 "AAAA" # उस पते में AAAA लिखें</td></tr><tr><td><strong>disassembly</strong></td><td><p>dis # वर्तमान फंक्शन को डिसेंबल करें</p><p>dis -n &#x3C;funcname> # फंक्शन को डिसेंबल करें</p><p>dis -n &#x3C;funcname> -b &#x3C;basename> # फंक्शन को डिसेंबल करें<br>dis -c 6 # 6 लाइनों को डिसेंबल करें<br>dis -c 0x100003764 -e 0x100003768 # एक से दूसरे तक<br>dis -p -c 4 # वर्तमान पते से डिसेंबल करना शुरू करें</p></td></tr><tr><td><strong>parray</strong></td><td>parray 3 (char **)$x1 # x1 रजिस्टर में 3 घटकों के एरे की जांच करें</td></tr></tbody></table>

{% hint style="info" %}
**`objc_sendMsg`** फंक्शन को कॉल करते समय, **rsi** रजिस्टर में नाम को एक नल-समाप्त ("सी") स्ट्रिंग के रूप म
## फज़िलत

### [रिपोर्टक्रैश](https://ss64.com/osx/reportcrash.html)

रिपोर्टक्रैश **क्रैश होने वाले प्रक्रियाओं का विश्लेषण करता है और क्रैश रिपोर्ट को डिस्क में सहेजता है**। एक क्रैश रिपोर्ट में जानकारी होती है जो एक डिवेलपर को क्रैश के कारण का निदान करने में मदद कर सकती है।\
अनुप्रयोगों और अन्य प्रक्रियाओं के लिए **प्रति-उपयोगकर्ता लॉन्चडी संदर्भ में चल रहे**, रिपोर्टक्रैश एक लॉन्चएजेंट के रूप में चलता है और क्रैश रिपोर्ट्स को उपयोगकर्ता के `~/Library/Logs/DiagnosticReports/` में सहेजता है।\
डेमन, अन्य प्रक्रियाएं **सिस्टम लॉन्चडी संदर्भ में चल रहे** और अन्य विशेषाधिकृत प्रक्रियाओं के लिए, रिपोर्टक्रैश एक लॉन्चडेमन के रूप में चलता है और क्रैश रिपोर्ट्स को सिस्टम के `/Library/Logs/DiagnosticReports` में सहेजता है।

यदि आपको चिंता है कि क्रैश रिपोर्ट्स **एपल को भेजे जा रहे हैं**, तो आप उन्हें अक्षम कर सकते हैं। यदि नहीं, तो क्रैश रिपोर्ट्स सर्वर क्रैश के कारण का पता लगाने में **उपयोगी हो सकते हैं**।
```bash
#To disable crash reporting:
launchctl unload -w /System/Library/LaunchAgents/com.apple.ReportCrash.plist
sudo launchctl unload -w /System/Library/LaunchDaemons/com.apple.ReportCrash.Root.plist

#To re-enable crash reporting:
launchctl load -w /System/Library/LaunchAgents/com.apple.ReportCrash.plist
sudo launchctl load -w /System/Library/LaunchDaemons/com.apple.ReportCrash.Root.plist
```
### नींद

MacOS में फज़िलत करते समय, मैक को सोने नहीं देना महत्वपूर्ण है:

* systemsetup -setsleep Never
* pmset, सिस्टम प्राथमिकताएं
* [KeepingYouAwake](https://github.com/newmarcel/KeepingYouAwake)

#### SSH डिस्कनेक्ट

यदि आप SSH कनेक्शन के माध्यम से फज़िलत कर रहे हैं, तो सुनिश्चित करें कि सत्र खत्म नहीं हो रहा है। इसलिए sshd\_config फ़ाइल को निम्नलिखित से बदलें:

* TCPKeepAlive Yes
* ClientAliveInterval 0
* ClientAliveCountMax 0
```bash
sudo launchctl unload /System/Library/LaunchDaemons/ssh.plist
sudo launchctl load -w /System/Library/LaunchDaemons/ssh.plist
```
### आंतरिक हैंडलर्स

**निम्नलिखित पृष्ठ पर जांच करें** कि आप कैसे पता लगा सकते हैं कि **निर्दिष्ट स्कीम या प्रोटोकॉल का हैंडल करने वाला ऐप कौन है:**

{% content-ref url="../macos-file-extension-apps.md" %}
[macos-file-extension-apps.md](../macos-file-extension-apps.md)
{% endcontent-ref %}

### नेटवर्क प्रक्रियाओं की गणना

यह दिलचस्प है कि नेटवर्क डेटा का प्रबंधन करने वाली प्रक्रियाएं कैसे खोजी जा सकती हैं:
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

### फजर्स

#### [AFL++](https://github.com/AFLplusplus/AFLplusplus)

CLI टूल्स के लिए काम करता है।

#### [Litefuzz](https://github.com/sec-tools/litefuzz)

यह macOS GUI टूल्स के साथ "**बस काम करता है"**। ध्यान दें कि कुछ macOS ऐप्स के कुछ विशेष आवश्यकताएं होती हैं जैसे अद्वितीय फ़ाइल नाम, सही एक्सटेंशन, सैंडबॉक्स से फ़ाइलें पढ़ने की आवश्यकता (`~/Library/Containers/com.apple.Safari/Data`)...

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

### और फज़िलत करने वाली MacOS जानकारी

* [https://www.youtube.com/watch?v=T5xfL9tEg44](https://www.youtube.com/watch?v=T5xfL9tEg44)
* [https://github.com/bnagy/slides/blob/master/OSXScale.pdf](https://github.com/bnagy/slides/blob/master/OSXScale.pdf)
* [https://github.com/bnagy/francis/tree/master/exploitaben](https://github.com/bnagy/francis/tree/master/exploitaben)
* [https://github.com/ant4g0nist/crashwrangler](https://github.com/ant4g0nist/crashwrangler)

## संदर्भ

* [**OS X घटना प्रतिक्रिया: स्क्रिप्टिंग और विश्लेषण**](https://www.amazon.com/OS-Incident-Response-Scripting-Analysis-ebook/dp/B01FHOHHVS)
* [**https://www.youtube.com/watch?v=T5xfL9tEg44**](https://www.youtube.com/watch?v=T5xfL9tEg44)
* [**https://taomm.org/vol1/analysis.html**](https://taomm.org/vol1/analysis.html)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने की अनुमति** चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह
* प्राप्त करें [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को।**

</details>
