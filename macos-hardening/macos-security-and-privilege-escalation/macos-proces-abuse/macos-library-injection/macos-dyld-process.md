# macOS Dyld प्रक्रिया

<details>

<summary><strong>जीरो से हीरो तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप अपनी कंपनी का विज्ञापन **HackTricks** में देखना चाहते हैं या **HackTricks को PDF में डाउनलोड** करना चाहते हैं तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह **The PEASS Family** की खोज करें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PR जमा करके।

</details>

## मूल जानकारी

Mach-o बाइनरी का वास्तविक **प्रवेश बिंदु** डायनामिक लिंक किया जाता है, जो `LC_LOAD_DYLINKER` में परिभाषित होता है और सामान्यत: `/usr/lib/dyld` होता है।

यह लिंकर सभी क्रियान्वित पुस्तकालयों को ढूंढ़ने, मेमोरी में मैप करने और सभी गुदाजी पुस्तकालयों को लिंक करने की आवश्यकता होगी। इस प्रक्रिया के बाद ही, बाइनरी का प्रवेश-बिंदु कार्यान्वित होगा।

बेशक, **`dyld`** किसी भी आवश्यकता का नहीं है (यह सिस्कॉल और libSystem उद्धरण का उपयोग करता है)।

{% hint style="danger" %}
यदि इस लिंकर में कोई भी सुरक्षा दोष होता है, क्योंकि यह किसी भी बाइनरी को कार्यान्वित करने से पहले (यहां तक कि उच्च विशेषाधिकार वाले भी) कार्यान्वित हो रहा है, तो **विशेषाधिकारों को उन्नत करना** संभव होगा।
{% endhint %}

### प्रवाह

Dyld को **`dyldboostrap::start`** द्वारा लोड किया जाएगा, जो चीजों को भी लोड करेगा जैसे **स्टैक कैनरी**। यह इसलिए है क्योंकि इस कार्य को इसके **`apple`** तर्क सूची में इस और अन्य **संवेदनशील** **मान** मिलेंगे।

**`dyls::_main()`** dyld का प्रवेश-बिंदु है और इसका पहला कार्य है `configureProcessRestrictions()` चलाना, जो सामान्यत: **`DYLD_*`** पर्यावरण मानों को प्रतिबंधित करता है जो निम्नलिखित में स्पष्ट किया गया है:

{% content-ref url="./" %}
[.](./)
{% endcontent-ref %}

फिर, यह dyld साझा कैश को मैप करेगा जो सभी महत्वपूर्ण सिस्टम पुस्तकालयों को पूर्व-लिंक करता है और फिर यह बाइनरी पर निर्भर करता है और जब तक सभी आवश्यक पुस्तकालय लोड हो जाते हैं, जारी रहता है। इसलिए:

1. यह `DYLD_INSERT_LIBRARIES` के साथ डाली गई पुस्तकालयों को लोड करना शुरू करता है (यदि अनुमति है)
2. फिर साझा कैश वाले
3. फिर आयातित वाले
4. फिर आगे आवश्यक पुस्तकालयों को आयात करना जारी रखें

एक बार जब सभी लोड हो जाते हैं, तो इन पुस्तकालयों के **आरंभक** चलाए जाते हैं। ये **`__attribute__((constructor))`** का उपयोग करके कोड किए गए हैं जो `LC_ROUTINES[_64]` में परिभाषित हैं (अब पुराने हो गए हैं) या `S_MOD_INIT_FUNC_POINTERS` के साथ झंडीत किए गए खंड में पॉइंटर द्वारा।

अंतकर्ता **`__attribute__((destructor))`** के साथ कोड किए गए हैं और एक खंड में स्थित हैं जिसे `S_MOD_TERM_FUNC_POINTERS` झंडीत किया गया है (**`__DATA.__mod_term_func`**).

### स्टब्स

सभी macOS बाइनरी गतिशील रूप से लिंक होती हैं। इसलिए, वे विभिन्न मशीनों और संदर्भ में सही कोड पर जाने में मदद करने वाले कुछ स्टब्स खंड शामिल होते हैं। यह dyld है जो बाइनरी को निष्क्रिय नहीं करने वाले पतों को हल करने की आवश्यकता है (कम से कम गुदाजी वाले).

बाइनरी में कुछ स्टब्स खंड:

* **`__TEXT.__[auth_]stubs`**: `__DATA` खंड से पॉइंटर
* **`__TEXT.__stub_helper`**: डायनामिक लिंकिंग को आमंत्रित करने वाला छोटा कोड
* **`__DATA.__[auth_]got`**: ग्लोबल ऑफसेट टेबल (आयातित फ़ंक्शनों के पतों के पते, जब वे हल होते हैं, (यह लोड समय के दौरान बाउंड होता है क्योंकि यह झंडीत है झंडी `S_NON_LAZY_SYMBOL_POINTERS` के साथ`)
* **`__DATA.__nl_symbol_ptr`**: नॉन-लेज़ी सिम्बल पॉइंटर (यह लोड समय के दौरान बाउंड होते हैं क्योंकि यह झंडीत है झंडी `S_NON_LAZY_SYMBOL_POINTERS` के साथ`)
* **`__DATA.__la_symbol_ptr`**: आलसी सिम्बल पॉइंटर (पहली पहुंच पर बाउंड होते हैं)

{% hint style="warning" %}
ध्यान दें कि "auth\_" प्रीफ़िक्स के साथ पॉइंटर एक प्रक्रिया एन्क्रिप्शन कुंजी का उपयोग कर रहे हैं इसे सुरक्षित रखने के लिए (PAC)। इसके अलावा, यह संभावित है कि आर्म64 निर्देश `BLRA[A/B]` का उपयोग करें पॉइंटर की पुष्टि करने से पहले। और RETA\[A/B\] का उपयोग एक RET पते के बजाय किया जा सकता है।\
वास्तव में, **`__TEXT.__auth_stubs`** में कोड **`bl`** की बजाय **`braa`** का उपयोग करेगा अनुरोधित फ़ंक्शन को आधारित करने के लिए।

इसके अलावा ध्यान दें कि वर्तमान dyld संस्करण सब कुछ **गुदाजी के रूप में लोड** करते हैं।
{% endhint %}

### आलसी सिम्बल्स खोजें
```c
//gcc load.c -o load
#include <stdio.h>
int main (int argc, char **argv, char **envp, char **apple)
{
printf("Hi\n");
}
```
रोचक डिसएसेम्बली भाग:
```armasm
; objdump -d ./load
100003f7c: 90000000    	adrp	x0, 0x100003000 <_main+0x1c>
100003f80: 913e9000    	add	x0, x0, #4004
100003f84: 94000005    	bl	0x100003f98 <_printf+0x100003f98>
```
यह संभव है कि जंप करके printf को कॉल करने जा रहा है **`__TEXT.__stubs`**:
```bash
objdump --section-headers ./load

./load:	file format mach-o arm64

Sections:
Idx Name          Size     VMA              Type
0 __text        00000038 0000000100003f60 TEXT
1 __stubs       0000000c 0000000100003f98 TEXT
2 __cstring     00000004 0000000100003fa4 DATA
3 __unwind_info 00000058 0000000100003fa8 DATA
4 __got         00000008 0000000100004000 DATA
```
जब **`__stubs`** सेक्शन को disassemble किया जाता है:
```bash
objdump -d --section=__stubs ./load

./load:	file format mach-o arm64

Disassembly of section __TEXT,__stubs:

0000000100003f98 <__stubs>:
100003f98: b0000010    	adrp	x16, 0x100004000 <__stubs+0x4>
100003f9c: f9400210    	ldr	x16, [x16]
100003fa0: d61f0200    	br	x16
```
आप देख सकते हैं कि हम **GOT के पते पर जा रहे हैं**, जो इस मामले में लेजी नहीं है और printf फ़ंक्शन का पता रखेगा।

अन्य स्थितियों में GOT पर सीधे जाने की बजाय, यह **`__DATA.__la_symbol_ptr`** पर जा सकता है जो एक मान लोड करेगा जो फ़ंक्शन का प्रतिनिधित्व करता है जिसे यह लोड करने की कोशिश कर रहा है, फिर **`__TEXT.__stub_helper`** पर जाएगा जो **`__DATA.__nl_symbol_ptr`** पर जाएगा जिसमें **`dyld_stub_binder`** का पता होता है जो फ़ंक्शन की संख्या और एक पते के रूप में पैरामीटर लेता है।
यह अंतिम फ़ंक्शन, खोजी गई फ़ंक्शन के पते को ढूंढने के बाद उसे **`__TEXT.__stub_helper`** में संबंधित स्थान में लिखता है ताकि भविष्य में लुकअप न करना पड़े।

{% hint style="success" %}
हालांकि ध्यान दें कि वर्तमान dyld संस्करण सभी चीजें लोड करते हैं जैसा कि लेजी नहीं है।
{% endhint %}

#### Dyld ऑपकोड

अंततः, **`dyld_stub_binder`** को निर्दिष्ट फ़ंक्शन को खोजने और उसे सही पते में लिखने की आवश्यकता है ताकि इसे फिर से खोजने की आवश्यकता न हो। इसे करने के लिए यह dyld के भीतर ऑपकोड (एक सीमित स्थिति मशीन) का उपयोग करता है।

## apple\[] argument vector

macOS में मुख्य फ़ंक्शन वास्तव में 3 के बजाय 4 प्रारूप में प्राप्त करता है। चौथा apple कहलाता है और प्रत्येक प्रविष्टि `key=value` के रूप में है। उदाहरण के लिए:
```c
// gcc apple.c -o apple
#include <stdio.h>
int main (int argc, char **argv, char **envp, char **apple)
{
for (int i=0; apple[i]; i++)
printf("%d: %s\n", i, apple[i])
}
```
### macOS Dyld Process

#### macOS लाइब्रेरी इंजेक्शन

Dyld एक प्रक्रिया है जो macOS में लोड होने वाले सभी लाइब्रेरी को संभालता है। Dyld प्रक्रिया को अद्यतन करने के लिए एक अद्वितीय तरीका है जिसे लाइब्रेरी इंजेक्शन कहा जाता है। यह तकनीक प्रक्रिया में अनधिकृत लाइब्रेरी को डालने की अनुमति देती है, जिससे एक हमलावर अनुप्रयोग को नुकसान पहुंचा सकता है।
```
0: executable_path=./a
1:
2:
3:
4: ptr_munge=
5: main_stack=
6: executable_file=0x1a01000012,0x5105b6a
7: dyld_file=0x1a01000012,0xfffffff0009834a
8: executable_cdhash=757a1b08ab1a79c50a66610f3adbca86dfd3199b
9: executable_boothash=f32448504e788a2c5935e372d22b7b18372aa5aa
10: arm64e_abi=os
11: th_port=
```
{% hint style="success" %}
जब ये मान मुख्य फ़ंक्शन तक पहुँचते हैं, तो संवेदनशील जानकारी पहले से ही उनसे हटा दी गई होती है या फिर यह एक डेटा लीक हो सकता है।
{% endhint %}

मुख्य में पहुँचने से पहले इन दिलचस्प मानों को देखना संभव है डीबगिंग के साथ:

<pre><code>lldb ./apple

<strong>(lldb) target create "./a"
</strong>Current executable set to '/tmp/a' (arm64).
(lldb) process launch -s
[..]

<strong>(lldb) mem read $sp
</strong>0x16fdff510: 00 00 00 00 01 00 00 00 01 00 00 00 00 00 00 00  ................
0x16fdff520: d8 f6 df 6f 01 00 00 00 00 00 00 00 00 00 00 00  ...o............

<strong>(lldb) x/55s 0x016fdff6d8
</strong>[...]
0x16fdffd6a: "TERM_PROGRAM=WarpTerminal"
0x16fdffd84: "WARP_USE_SSH_WRAPPER=1"
0x16fdffd9b: "WARP_IS_LOCAL_SHELL_SESSION=1"
0x16fdffdb9: "SDKROOT=/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX14.4.sdk"
0x16fdffe24: "NVM_DIR=/Users/carlospolop/.nvm"
0x16fdffe44: "CONDA_CHANGEPS1=false"
0x16fdffe5a: ""
0x16fdffe5b: ""
0x16fdffe5c: ""
0x16fdffe5d: ""
0x16fdffe5e: ""
0x16fdffe5f: ""
0x16fdffe60: "pfz=0xffeaf0000"
0x16fdffe70: "stack_guard=0x8af2b510e6b800b5"
0x16fdffe8f: "malloc_entropy=0xf2349fbdea53f1e4,0x3fd85d7dcf817101"
0x16fdffec4: "ptr_munge=0x983e2eebd2f3e746"
0x16fdffee1: "main_stack=0x16fe00000,0x7fc000,0x16be00000,0x4000000"
0x16fdfff17: "executable_file=0x1a01000012,0x5105b6a"
0x16fdfff3e: "dyld_file=0x1a01000012,0xfffffff0009834a"
0x16fdfff67: "executable_cdhash=757a1b08ab1a79c50a66610f3adbca86dfd3199b"
0x16fdfffa2: "executable_boothash=f32448504e788a2c5935e372d22b7b18372aa5aa"
0x16fdfffdf: "arm64e_abi=os"
0x16fdfffed: "th_port=0x103"
0x16fdffffb: ""
</code></pre>

## dyld\_all\_image\_infos

यह एक संरचना है जो dyld द्वारा निर्यात की गई है जिसमें dyld की स्थिति के बारे में जानकारी होती है जो [**स्रोत कोड**](https://opensource.apple.com/source/dyld/dyld-852.2/include/mach-o/dyld\_images.h.auto.html) में पाई जा सकती है जैसे संस्करण, dyld\_image\_info एरे के लिए प्वाइंटर, dyld\_image\_notifier के लिए, यदि प्रोसेस साझा कैश से अलग है, यदि libSystem आरंभक को कॉल किया गया था, dyld के अपने Mach हेडर के लिए प्वाइंटर, dyld संस्करण स्ट्रिंग के प्वाइंटर...

## dyld env variables

### debug dyld

दिलचस्प एनवायरनमेंट वेरिएबल जो समझने में मदद करते हैं कि dyld क्या कर रहा है:

* **DYLD\_PRINT\_LIBRARIES**

प्रत्येक लाइब्रेरी की जांच करें जो लोड हो रही है:
```
DYLD_PRINT_LIBRARIES=1 ./apple
dyld[19948]: <9F848759-9AB8-3BD2-96A1-C069DC1FFD43> /private/tmp/a
dyld[19948]: <F0A54B2D-8751-35F1-A3CF-F1A02F842211> /usr/lib/libSystem.B.dylib
dyld[19948]: <C683623C-1FF6-3133-9E28-28672FDBA4D3> /usr/lib/system/libcache.dylib
dyld[19948]: <BFDF8F55-D3DC-3A92-B8A1-8EF165A56F1B> /usr/lib/system/libcommonCrypto.dylib
dyld[19948]: <B29A99B2-7ADE-3371-A774-B690BEC3C406> /usr/lib/system/libcompiler_rt.dylib
dyld[19948]: <65612C42-C5E4-3821-B71D-DDE620FB014C> /usr/lib/system/libcopyfile.dylib
dyld[19948]: <B3AC12C0-8ED6-35A2-86C6-0BFA55BFF333> /usr/lib/system/libcorecrypto.dylib
dyld[19948]: <8790BA20-19EC-3A36-8975-E34382D9747C> /usr/lib/system/libdispatch.dylib
dyld[19948]: <4BB77515-DBA8-3EDF-9AF7-3C9EAE959EA6> /usr/lib/system/libdyld.dylib
dyld[19948]: <F7CE9486-FFF5-3CB8-B26F-75811EF4283A> /usr/lib/system/libkeymgr.dylib
dyld[19948]: <1A7038EC-EE49-35AE-8A3C-C311083795FB> /usr/lib/system/libmacho.dylib
[...]
```
* **DYLD\_PRINT\_SEGMENTS**

परीक्षण करें कि प्रत्येक पुस्तकालय कैसे लोड होती है:
```
DYLD_PRINT_SEGMENTS=1 ./apple
dyld[21147]: re-using existing shared cache (/System/Volumes/Preboot/Cryptexes/OS/System/Library/dyld/dyld_shared_cache_arm64e):
dyld[21147]:         0x181944000->0x1D5D4BFFF init=5, max=5 __TEXT
dyld[21147]:         0x1D5D4C000->0x1D5EC3FFF init=1, max=3 __DATA_CONST
dyld[21147]:         0x1D7EC4000->0x1D8E23FFF init=3, max=3 __DATA
dyld[21147]:         0x1D8E24000->0x1DCEBFFFF init=3, max=3 __AUTH
dyld[21147]:         0x1DCEC0000->0x1E22BFFFF init=1, max=3 __AUTH_CONST
dyld[21147]:         0x1E42C0000->0x1E5457FFF init=1, max=1 __LINKEDIT
dyld[21147]:         0x1E5458000->0x22D173FFF init=5, max=5 __TEXT
dyld[21147]:         0x22D174000->0x22D9E3FFF init=1, max=3 __DATA_CONST
dyld[21147]:         0x22F9E4000->0x230F87FFF init=3, max=3 __DATA
dyld[21147]:         0x230F88000->0x234EC3FFF init=3, max=3 __AUTH
dyld[21147]:         0x234EC4000->0x237573FFF init=1, max=3 __AUTH_CONST
dyld[21147]:         0x239574000->0x270BE3FFF init=1, max=1 __LINKEDIT
dyld[21147]: Kernel mapped /private/tmp/a
dyld[21147]:     __PAGEZERO (...) 0x000000904000->0x000101208000
dyld[21147]:         __TEXT (r.x) 0x000100904000->0x000100908000
dyld[21147]:   __DATA_CONST (rw.) 0x000100908000->0x00010090C000
dyld[21147]:     __LINKEDIT (r..) 0x00010090C000->0x000100910000
dyld[21147]: Using mapping in dyld cache for /usr/lib/libSystem.B.dylib
dyld[21147]:         __TEXT (r.x) 0x00018E59D000->0x00018E59F000
dyld[21147]:   __DATA_CONST (rw.) 0x0001D5DFDB98->0x0001D5DFDBA8
dyld[21147]:   __AUTH_CONST (rw.) 0x0001DDE015A8->0x0001DDE01878
dyld[21147]:         __AUTH (rw.) 0x0001D9688650->0x0001D9688658
dyld[21147]:         __DATA (rw.) 0x0001D808AD60->0x0001D808AD68
dyld[21147]:     __LINKEDIT (r..) 0x000239574000->0x000270BE4000
dyld[21147]: Using mapping in dyld cache for /usr/lib/system/libcache.dylib
dyld[21147]:         __TEXT (r.x) 0x00018E597000->0x00018E59D000
dyld[21147]:   __DATA_CONST (rw.) 0x0001D5DFDAF0->0x0001D5DFDB98
dyld[21147]:   __AUTH_CONST (rw.) 0x0001DDE014D0->0x0001DDE015A8
dyld[21147]:     __LINKEDIT (r..) 0x000239574000->0x000270BE4000
[...]
```
* **DYLD\_PRINT\_INITIALIZERS**

प्रिंट करें जब प्रत्येक लाइब्रेरी इनिशियलाइज़र चल रहा है:
```
DYLD_PRINT_INITIALIZERS=1 ./apple
dyld[21623]: running initializer 0x18e59e5c0 in /usr/lib/libSystem.B.dylib
[...]
```
### अन्य

* `DYLD_BIND_AT_LAUNCH`: आलसी बाइंडिंग को गैर-आलसी के साथ हल किया जाता है
* `DYLD_DISABLE_PREFETCH`: \_\_DATA और \_\_LINKEDIT सामग्री का पूर्वाभिलेखन अक्षम करें
* `DYLD_FORCE_FLAT_NAMESPACE`: एक स्तरीय बाइंडिंग
* `DYLD_[FRAMEWORK/LIBRARY]_PATH | DYLD_FALLBACK_[FRAMEWORK/LIBRARY]_PATH | DYLD_VERSIONED_[FRAMEWORK/LIBRARY]_PATH`: संकलन पथ
* `DYLD_INSERT_LIBRARIES`: एक विशिष्ट पुस्तकालय लोड करें
* `DYLD_PRINT_TO_FILE`: एक फ़ाइल में dyld डीबग लिखें
* `DYLD_PRINT_APIS`: libdyld API कॉल प्रिंट करें
* `DYLD_PRINT_APIS_APP`: मुख्य द्वारा किए गए libdyld API कॉल प्रिंट करें
* `DYLD_PRINT_BINDINGS`: बाइंड होने पर प्रतीक प्रिंट करें
* `DYLD_WEAK_BINDINGS`: बाइंड होने पर केवल कमजोर प्रतीक प्रिंट करें
* `DYLD_PRINT_CODE_SIGNATURES`: कोड हस्ताक्षर पंजीकरण कार्यों को प्रिंट करें
* `DYLD_PRINT_DOFS`: लोड होने पर D-ट्रेस ऑब्जेक्ट फ़ॉर्मेट खंड प्रिंट करें
* `DYLD_PRINT_ENV`: dyld द्वारा देखे गए env प्रिंट करें
* `DYLD_PRINT_INTERPOSTING`: इंटरपोस्टिंग कार्यों को प्रिंट करें
* `DYLD_PRINT_LIBRARIES`: लोड की गई पुस्तकालयें प्रिंट करें
* `DYLD_PRINT_OPTS`: लोड विकल्प प्रिंट करें
* `DYLD_REBASING`: प्रतीक रीबेसिंग कार्यों को प्रिंट करें
* `DYLD_RPATHS`: @rpath के विस्तार को प्रिंट करें
* `DYLD_PRINT_SEGMENTS`: Mach-O सेगमेंट के मैपिंग प्रिंट करें
* `DYLD_PRINT_STATISTICS`: समय सांख्यिकी प्रिंट करें
* `DYLD_PRINT_STATISTICS_DETAILS`: विस्तृत समय सांख्यिकी प्रिंट करें
* `DYLD_PRINT_WARNINGS`: चेतावनी संदेश प्रिंट करें
* `DYLD_SHARED_CACHE_DIR`: साझा पुस्तकालय कैश के लिए पथ
* `DYLD_SHARED_REGION`: "उपयोग", "निजी", "टालें"
* `DYLD_USE_CLOSURES`: बंद करने की अनुमति दें

ऐसा कुछ और भी ढूंढना संभव है:
```bash
strings /usr/lib/dyld | grep "^DYLD_" | sort -u
```
या dyld परियोजना को [https://opensource.apple.com/tarballs/dyld/dyld-852.2.tar.gz](https://opensource.apple.com/tarballs/dyld/dyld-852.2.tar.gz) से डाउनलोड करके फ़ोल्डर के अंदर चला सकते हैं:
```bash
find . -type f | xargs grep strcmp| grep key,\ \" | cut -d'"' -f2 | sort -u
```
## संदर्भ

* [**\*OS Internals, Volume I: User Mode. By Jonathan Levin**](https://www.amazon.com/MacOS-iOS-Internals-User-Mode/dp/099105556X)

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह **The PEASS Family** की खोज करें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>
