<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks क्लाउड ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 ट्विटर 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ ट्विच 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 यूट्यूब 🎥</strong></a></summary>

- क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS परिवार**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)

- प्राप्त करें [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर सबमिट करके साझा करें।**

</details>


# मूलभूत जानकारी

**Seccomp** या सुरक्षित गणना मोड, संक्षेप में, लिनक्स कर्नल की एक सुविधा है जो **सिस्टम कॉल फ़िल्टर** के रूप में कार्य कर सकती है।\
Seccomp के 2 मोड होते हैं।

**seccomp** (सुरक्षित गणना मोड का संक्षेप रूप) लिनक्स कर्नल में एक कंप्यूटर सुरक्षा सुविधा है। seccomp एक प्रक्रिया को "सुरक्षित" स्थिति में एक-तरफ़ा परिवर्तन करने की अनुमति देता है जहां **यह कोई सिस्टम कॉल नहीं कर सकता**, केवल `exit()`, `sigreturn()`, `read()` और `write()` को **पहले से खोले गए** फ़ाइल डिस्क्रिप्टर पर। यदि इसके अलावा कोई अन्य सिस्टम कॉल की कोशिश की जाए, तो कर्नल प्रक्रिया को SIGKILL या SIGSYS के साथ **समाप्त** कर देगा। इस प्रकार में, यह सिस्टम के संसाधनों को वर्चुअलाइज़ नहीं करता है, बल्कि प्रक्रिया को पूरी तरह से उनसे अलग करता है।

seccomp मोड को `prctl(2)` सिस्टम कॉल का उपयोग करके `PR_SET_SECCOMP` तर्क का उपयोग करके सक्षम किया जाता है, या (लिनक्स कर्नल 3.17 के बाद से) `seccomp(2)` सिस्टम कॉल का उपयोग करके। seccomp मोड को पहले एक फ़ाइल, `/proc/self/seccomp`, में लिखकर सक्षम किया जाता था, लेकिन इस विधि को `prctl()` के पक्ष में हटा दिया गया है। कुछ कर्नल संस्करणों में, seccomp `RDTSC` x86 इंस्ट्रक्शन को अक्षरशः अक्षम कर देता है, जो पावर-ऑन के बाद बिते हुए प्रोसेसर साइकिलों की संख्या वापस करता है, जिसे उच्च-सटीक समयगणना के लिए उपयोग किया जाता है।

**seccomp-bpf** seccomp का एक विस्तार है जो बर्कले पैकेट फ़िल्टर नियमों का उपयोग करके सिस्टम कॉल की फ़िल्टरिंग की अनुमति देता है। इसका उपयोग ओपनएसएसएच और वीएसएफटीपीडी द्वारा किया जाता है, साथ ही क्रोम ओएस और लिनक्स पर गूगल क्रोम/क्रोमियम वेब ब्राउज़र। (इस दृष्टि में seccomp-bpf पुराने सिस्ट्रेस के लिए समान कार्यक्षमता प्राप्त करता है, लेकिन और लचीलापन और उच्च प्रदर्शन के साथ—जो लगता है कि लिनक्स के लिए अब और समर्थित नहीं है।)

## **मूल/सख्त मोड**

इस मोड में **Seccomp केवल syscalls की अनुमति देता है** `exit()`, `sigreturn()`, `read()` और `write()` को पहले से खोले गए फ़ाइल डिस्क्रिप्टर पर। यदि कोई अन्य syscall की कोशिश की जाए, तो प्रक्रिया को SIGKILL के उपयोग से मार दिया जाता है

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
## Seccomp-bpf

यह मोड एक विन्यासयोग्य नीति का उपयोग करके बर्कले पैकेट फ़िल्टर नियमों का उपयोग करके सिस्टम कॉल के फ़िल्टरिंग को संभव बनाता है।

{% code title="seccomp_bpf.c" %}
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

# Docker में Seccomp

**Seccomp-bpf** को **Docker** द्वारा समर्थित किया जाता है ताकि संग्रह को कम करके कंटेनर से **syscalls **को प्रतिबंधित किया जा सके। आप डिफ़ॉल्ट रूप से **ब्लॉक किए गए syscalls **को [https://docs.docker.com/engine/security/seccomp/](https://docs.docker.com/engine/security/seccomp/) में और डिफ़ॉल्ट seccomp प्रोफ़ाइल को यहां पा सकते हैं [https://github.com/moby/moby/blob/master/profiles/seccomp/default.json](https://github.com/moby/moby/blob/master/profiles/seccomp/default.json)।\
आप एक डॉकर कंटेनर को **अलग seccomp** नीति के साथ चला सकते हैं:
```bash
docker run --rm \
-it \
--security-opt seccomp=/path/to/seccomp/profile.json \
hello-world
```
यदि आप उदाहरण के लिए किसी कंटेनर को `uname` जैसे कुछ `syscall` का निषेध करना चाहते हैं, तो आप [https://github.com/moby/moby/blob/master/profiles/seccomp/default.json](https://github.com/moby/moby/blob/master/profiles/seccomp/default.json) से डिफ़ॉल्ट प्रोफ़ाइल डाउनलोड कर सकते हैं और बस उस सूची से `uname` स्ट्रिंग को हटा दें।\
यदि आप सुनिश्चित करना चाहते हैं कि **किसी बाइनरी कंटेनर के अंदर काम नहीं करती है**, तो आप strace का उपयोग करके बाइनरी के द्वारा उपयोग किए जा रहे syscalls की सूची बना सकते हैं और फिर उन्हें निषेधित कर सकते हैं।\
निम्नलिखित उदाहरण में `uname` के **syscalls** का पता चलता है:
```bash
docker run -it --security-opt seccomp=default.json modified-ubuntu strace uname
```
{% hint style="info" %}
यदि आप केवल एक एप्लिकेशन चलाने के लिए Docker का उपयोग कर रहे हैं, तो आप इसे **`strace`** के साथ **प्रोफ़ाइल** कर सकते हैं और उसे उन syscalls की अनुमति दें जो इसे आवश्यक होती हैं।
{% endhint %}

## Docker में इसे निष्क्रिय करें

फ़्लैग के साथ एक कंटेनर चलाएं: **`--security-opt seccomp=unconfined`**


<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने की अनुमति** चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह

- प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपने हैकिंग ट्रिक्स साझा करें, [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके**।

</details>
