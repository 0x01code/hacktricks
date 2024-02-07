# Seccomp

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) और **मुझे** **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** द्वारा **पीआर्स** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PR जमा करके।

</details>

## मूल जानकारी

**Seccomp**, जिसका मतलब है Secure Computing mode, **Linux kernel की एक सुरक्षा सुविधा है जो सिस्टम कॉल्स को फ़िल्टर करने के लिए डिज़ाइन की गई है**। यह प्रक्रियाओं को सिस्टम कॉल्स के सीमित सेट (`exit()`, `sigreturn()`, `read()`, और `write()`) में प्रतिबंधित करता है। अगर कोई प्रक्रिया कोई और कॉल करने की कोशिश करती है, तो कर्नेल द्वारा SIGKILL या SIGSYS का उपयोग करके उसे समाप्त कर दिया जाता है। यह तंतुओं को वर्चुअलाइज़ नहीं करता है लेकिन प्रक्रिया को उनसे अलग कर देता है।

Seccomp को सक्रिय करने के दो तरीके हैं: `prctl(2)` सिस्टम कॉल के साथ `PR_SET_SECCOMP` के माध्यम से, या Linux kernels 3.17 और ऊपर के लिए, `seccomp(2)` सिस्टम कॉल के माध्यम से। Seccomp को `/proc/self/seccomp` में लिखकर सक्रिय करने की पुरानी विधि को `prctl()` के पक्ष में अनुप्रयोग किया गया है।

एक वृद्धि, **seccomp-bpf**, बर्कले पैकेट फ़िल्टर (BPF) नियमों का उपयोग करके एक अनुकूलनीय नीति के साथ सिस्टम कॉल्स को फ़िल्टर करने की क्षमता जोड़ता है। यह विस्तार सॉफ़्टवेयर द्वारा उपयोग किया जाता है जैसे कि OpenSSH, vsftpd, और Chrome/Chromium ब्राउज़र्स Chrome OS और Linux पर लचीले और कुशल syscall फ़िल्टरिंग के लिए, अब असमर्थित systrace के लिए एक वैकल्पिक पेशकश करता है।

### **मूल/सख्त मोड**

इस मोड में Seccomp **केवल सिस्टम कॉल्स को अनुमति देता है** `exit()`, `sigreturn()`, `read()` और `write()` को पहले से खोले गए फ़ाइल डिस्क्रिप्टर्स के लिए। अगर कोई अन्य सिस्टम कॉल की कोशिश करता है, तो प्रक्रिया को SIGKILL का उपयोग करके मार दिया जाता है

{% code title="seccomp_strict.c" %}
```c
#include <fcntl.h>
#include <stdio.h>
#include <unistd.h>
#include <string.h>
#include <linux/seccomp.h>
#include <sys/prctl.h>

//From https://sysdig.com/blog/selinux-seccomp-falco-technical-discussion/
//gcc seccomp_strict.c -o seccomp_strict

int main(int argc, char **argv)
{
int output = open("output.txt", O_WRONLY);
const char *val = "test";

//enables strict seccomp mode
printf("Calling prctl() to set seccomp strict mode...\n");
prctl(PR_SET_SECCOMP, SECCOMP_MODE_STRICT);

//This is allowed as the file was already opened
printf("Writing to an already open file...\n");
write(output, val, strlen(val)+1);

//This isn't allowed
printf("Trying to open file for reading...\n");
int input = open("output.txt", O_RDONLY);

printf("You will not see this message--the process will be killed first\n");
}
```
### Seccomp-bpf

यह मोड **कॉन्फ़िगरेबल नीति का उपयोग करके सिस्टम कॉल की फ़िल्टरिंग** को बर्कले पैकेट फ़िल्टर नियमों का उपयोग करके संभावित बनाता है।
```c
#include <seccomp.h>
#include <unistd.h>
#include <stdio.h>
#include <errno.h>

//https://security.stackexchange.com/questions/168452/how-is-sandboxing-implemented/175373
//gcc seccomp_bpf.c -o seccomp_bpf -lseccomp

void main(void) {
/* initialize the libseccomp context */
scmp_filter_ctx ctx = seccomp_init(SCMP_ACT_KILL);

/* allow exiting */
printf("Adding rule : Allow exit_group\n");
seccomp_rule_add(ctx, SCMP_ACT_ALLOW, SCMP_SYS(exit_group), 0);

/* allow getting the current pid */
//printf("Adding rule : Allow getpid\n");
//seccomp_rule_add(ctx, SCMP_ACT_ALLOW, SCMP_SYS(getpid), 0);

printf("Adding rule : Deny getpid\n");
seccomp_rule_add(ctx, SCMP_ACT_ERRNO(EBADF), SCMP_SYS(getpid), 0);
/* allow changing data segment size, as required by glibc */
printf("Adding rule : Allow brk\n");
seccomp_rule_add(ctx, SCMP_ACT_ALLOW, SCMP_SYS(brk), 0);

/* allow writing up to 512 bytes to fd 1 */
printf("Adding rule : Allow write upto 512 bytes to FD 1\n");
seccomp_rule_add(ctx, SCMP_ACT_ALLOW, SCMP_SYS(write), 2,
SCMP_A0(SCMP_CMP_EQ, 1),
SCMP_A2(SCMP_CMP_LE, 512));

/* if writing to any other fd, return -EBADF */
printf("Adding rule : Deny write to any FD except 1 \n");
seccomp_rule_add(ctx, SCMP_ACT_ERRNO(EBADF), SCMP_SYS(write), 1,
SCMP_A0(SCMP_CMP_NE, 1));

/* load and enforce the filters */
printf("Load rules and enforce \n");
seccomp_load(ctx);
seccomp_release(ctx);
//Get the getpid is denied, a weird number will be returned like
//this process is -9
printf("this process is %d\n", getpid());
}
```
{% endcode %}

## Docker में Seccomp

**Seccomp-bpf** को **Docker** द्वारा समर्थित किया गया है ताकि **सिस-कॉल्स** को प्रतिबंधित किया जा सके और सतह क्षेत्र को कम किया जा सके। आप यहाँ [https://docs.docker.com/engine/security/seccomp/](https://docs.docker.com/engine/security/seccomp/) पर **डिफ़ॉल्ट रूप से अवरोधित सिस-कॉल्स** देख सकते हैं और **डिफ़ॉल्ट seccomp प्रोफ़ाइल** यहाँ मिल सकता है [https://github.com/moby/moby/blob/master/profiles/seccomp/default.json](https://github.com/moby/moby/blob/master/profiles/seccomp/default.json)।\
आप एक डॉकर कंटेनर को एक **विभिन्न seccomp** नीति के साथ चला सकते हैं:
```bash
docker run --rm \
-it \
--security-opt seccomp=/path/to/seccomp/profile.json \
hello-world
```
यदि आप उदाहरण के लिए किसी कंटेनर से `uname` जैसे कुछ **syscall** को **निषेधित** करना चाहते हैं तो आप [https://github.com/moby/moby/blob/master/profiles/seccomp/default.json](https://github.com/moby/moby/blob/master/profiles/seccomp/default.json) से डिफ़ॉल्ट प्रोफ़ाइल डाउनलोड कर सकते हैं और **सिर्फ सूची से `uname` स्ट्रिंग को हटा दें**।\
यदि आप सुनिश्चित करना चाहते हैं कि **किसी बाइनरी कंटेनर के अंदर काम न करे** तो आप strace का उपयोग करके बाइनरी द्वारा उपयोग किए जा रहे syscalls की सूची बना सकते हैं और फिर उन्हें निषेधित कर सकते हैं।\
निम्नलिखित उदाहरण में `uname` के **syscalls** खोजे गए हैं:
```bash
docker run -it --security-opt seccomp=default.json modified-ubuntu strace uname
```
{% hint style="info" %}
यदि आप **Docker** का उपयोग **केवल एक एप्लिकेशन लॉन्च करने** के लिए कर रहे हैं, तो आप इसे **`strace`** के साथ **प्रोफ़ाइल** कर सकते हैं और **उसे केवल वह syscalls अनुमति दें** जो आवश्यक हैं।
{% endhint %}

### उदाहरण Seccomp नीति

[यहाँ से उदाहरण](https://sreeninet.wordpress.com/2016/03/06/docker-security-part-2docker-engine/)

Seccomp विशेषता को विस्तारित करने के लिए, चलो नीचे दिए गए "chmod" सिस्टम कॉल को अक्षम करने वाली एक Seccomp प्रोफ़ाइल बनाते हैं।
```json
{
"defaultAction": "SCMP_ACT_ALLOW",
"syscalls": [
{
"name": "chmod",
"action": "SCMP_ACT_ERRNO"
}
]
}
```
उपर दिए गए प्रोफ़ाइल में, हमने डिफ़ॉल्ट क्रिया को "अनुमति" पर सेट किया है और "chmod" को अक्षम करने के लिए एक ब्लैकलिस्ट बनाई है। अधिक सुरक्षित होने के लिए, हम डिफ़ॉल्ट क्रिया को ड्रॉप पर सेट कर सकते हैं और सिस्टम कॉल को विवेकपूर्ण रूप से सक्षम करने के लिए एक व्हाइटलिस्ट बना सकते हैं।\
निम्नलिखित आउटपुट "chmod" कॉल को त्रुटि दिखाता है क्योंकि इसे सेक्कॉम्प प्रोफ़ाइल में अक्षम किया गया है।
```bash
$ docker run --rm -it --security-opt seccomp:/home/smakam14/seccomp/profile.json busybox chmod 400 /etc/hosts
chmod: /etc/hosts: Operation not permitted
```
निम्नलिखित आउटपुट "docker inspect" को प्रोफ़ाइल दिखाता है:
```json
"SecurityOpt": [
"seccomp:{\"defaultAction\":\"SCMP_ACT_ALLOW\",\"syscalls\":[{\"name\":\"chmod\",\"action\":\"SCMP_ACT_ERRNO\"}]}"
],
```
### Docker में इसे निष्क्रिय करें

एक कंटेनर लॉन्च करें जिसमें फ्लैग है: **`--security-opt seccomp=unconfined`**

Kubernetes 1.19 के रूप में, **सभी पॉड्स के लिए डिफ़ॉल्ट रूप से सेककॉम्प सक्षम है**। हालांकि, पॉड्स पर लागू डिफ़ॉल्ट सेककॉम्प प्रोफ़ाइल "**RuntimeDefault**" है, जो **कंटेनर रनटाइम द्वारा प्रदान किया गया है** (जैसे, Docker, containerd)। "RuntimeDefault" प्रोफ़ाइल अधिकांश सिस्टम कॉल को अनुमति देता है जबकि कुछ कॉल्स को ब्लॉक करता है जो खतरनाक माने गए हैं या सामान्य रूप से कंटेनर्स द्वारा आवश्यक नहीं माने जाते हैं।
