# सेकॉम्प

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ हैकट्रिक्स क्लाउड ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 ट्विटर 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ ट्विच 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 यूट्यूब 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को हैकट्रिक्स में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें और PR जमा करके** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **को फ़ॉलो करें।**

</details>

## मूलभूत जानकारी

**सेकॉम्प** या सुरक्षित कंप्यूटिंग मोड, संक्षेप में, लिनक्स कर्नल की एक सुविधा है जो एक **सिस्टेम कॉल फ़िल्टर** के रूप में कार्य कर सकती है।\
सेकॉम्प के 2 मोड होते हैं।

**सेकॉम्प** (सुरक्षित कंप्यूटिंग मोड के लिए संक्षेप में) लिनक्स कर्नल में एक कंप्यूटर सुरक्षा सुविधा है। सेकॉम्प एक प्रक्रिया को एक "सुरक्षित" स्थिति में बदलने की अनुमति देता है जहां **यह कोई सिस्टम कॉल नहीं कर सकती** है, केवल `exit()`, `sigreturn()`, `read()` और `write()` को **पहले से खोले गए** फ़ाइल डिस्क्रिप्टर पर। यदि इसके अलावा कोई अन्य सिस्टम कॉल की कोशिश की जाती है, तो कर्नल प्रक्रिया को SIGKILL या SIGSYS के साथ **समाप्त** कर देगा। इस प्रकार में, यह सिस्टम के संसाधनों को वर्चुअलाइज़ नहीं करता है, बल्कि प्रक्रिया को पूरी तरह से उनसे अलग करता है।

सेकॉम्प मोड को `prctl(2)` सिस्टम कॉल का उपयोग करके `PR_SET_SECCOMP` तर्क का उपयोग करके सक्षम किया जाता है, या (लिनक्स कर्नल 3.17 के बाद से) `seccomp(2)` सिस्टम कॉल का उपयोग करके। सेकॉम्प मोड को पहले एक फ़ाइल, `/proc/self/seccomp` में लिखकर सक्षम किया जाता था, लेकिन इस विधि को `prctl()` के पक्ष में हटा दिया गया। कुछ कर्नल संस्करणों में, सेकॉम्प `RDTSC` x86 इंस्ट्रक्शन को अक्षम कर देता है, जो पावर-ऑन के बाद बिते हुए प्रोसेसर साइकिलों की संख्या वापस करता है, जिसे उच्च-सटीक समय मापन के लिए उपयोग किया जाता है।

**सेकॉम्प-बीपीएफ** सेकॉम्प का एक विस्तार है जो बर्कले पैकेट फ़िल्टर नियम का उपयोग करके सिस्टम कॉल की फ़िल्टरिंग की अनुमति देता है। इसका उपयोग ओपनएसएसएच और वीएसएफटीपीडी द्वारा किया जाता है, साथ ही क्रोम ओएस और लिनक्स पर गूगल क्रोम/क्रोमियम वेब ब्राउज़र। (इस दृष्टि में सेकॉम्प-बीपीएफ पुराने सिस्ट्रेस के समान कार्यक्षमता प्राप्त करता है, लेकिन इसमें और लचीलापन और उच्च प्रदर्शन होता है, लिनक्स के लिए यह अब बदल गया लगता है।)

### **मूल/सख्त मोड**

इस मोड में सेकॉम्प **केवल सिस्टम कॉल की अनुमति देता है** `exit()`, `sigreturn()`, `read()` और `write()` को पहले से खोले गए फ़ाइल डिस्क्रिप्टर पर। यदि कोई अन्य सिस्टम कॉल की जाती है, तो प्रक्रिया को SIGKILL का उपयोग करके मार दिया जाता है।

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
{% endcode %}

### Seccomp-bpf

यह मोड एक विन्यासयोग्य नीति का उपयोग करके बर्कले पैकेट फ़िल्टर नियमों का उपयोग करके सिस्टम कॉल के फ़िल्टरिंग की अनुमति देता है।

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

## Docker में Seccomp

**Seccomp-bpf** को **Docker** द्वारा समर्थित किया जाता है ताकि संग्रह से संबंधित **syscalls** को प्रतिबंधित किया जा सके और सतह क्षेत्र को कम किया जा सके। आप डिफ़ॉल्ट में **ब्लॉक किए गए syscalls** को [https://docs.docker.com/engine/security/seccomp/](https://docs.docker.com/engine/security/seccomp/) में और डिफ़ॉल्ट seccomp प्रोफ़ाइल को यहां [https://github.com/moby/moby/blob/master/profiles/seccomp/default.json](https://github.com/moby/moby/blob/master/profiles/seccomp/default.json) में पा सकते हैं।\
आप एक डॉकर कंटेनर को एक **अलग seccomp** नीति के साथ चला सकते हैं:
```bash
docker run --rm \
-it \
--security-opt seccomp=/path/to/seccomp/profile.json \
hello-world
```
यदि आप उदाहरण के लिए किसी कंटेनर को `uname` जैसे कुछ **सिस्टम कॉल** को **मना** करना चाहते हैं, तो आप [https://github.com/moby/moby/blob/master/profiles/seccomp/default.json](https://github.com/moby/moby/blob/master/profiles/seccomp/default.json) से डिफ़ॉल्ट प्रोफ़ाइल डाउनलोड कर सकते हैं और बस **सूची से `uname` स्ट्रिंग को हटा दें**।\
यदि आप सुनिश्चित करना चाहते हैं कि **किसी बाइनरी कंटेनर के अंदर काम नहीं करती है**, तो आप strace का उपयोग करके बाइनरी के द्वारा उपयोग की जा रही सिस्टम कॉल की सूची बना सकते हैं और फिर उन्हें मना कर सकते हैं।\
निम्नलिखित उदाहरण में `uname` की **सिस्टम कॉल** की खोज की जाती है:
```bash
docker run -it --security-opt seccomp=default.json modified-ubuntu strace uname
```
{% hint style="info" %}
यदि आप केवल एक एप्लिकेशन चलाने के लिए Docker का उपयोग कर रहे हैं, तो आप इसे **`strace`** के साथ **प्रोफ़ाइल** कर सकते हैं और केवल उन syscalls को **अनुमति दें** जिनकी आवश्यकता होती है
{% endhint %}

### उदाहरण Seccomp नीति

Seccomp फ़ीचर को दिखाने के लिए, नीचे दिए गए तरीके से "chmod" सिस्टम कॉल को अक्षम करने वाली एक Seccomp प्रोफ़ाइल बनाएं।
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
उपरोक्त प्रोफ़ाइल में, हमने डिफ़ॉल्ट क्रिया को "अनुमति" पर सेट किया है और "chmod" को अक्षम करने के लिए एक ब्लैकलिस्ट बनाई है। अधिक सुरक्षित होने के लिए, हम डिफ़ॉल्ट क्रिया को ड्रॉप पर सेट कर सकते हैं और सिस्टम कॉल को चयनित रूप से सक्षम करने के लिए एक व्हाइटलिस्ट बना सकते हैं।\
निम्नलिखित आउटपुट में "chmod" कॉल त्रुटि दिखा रहा है क्योंकि यह सेकंप प्रोफ़ाइल में अक्षम है।
```bash
$ docker run --rm -it --security-opt seccomp:/home/smakam14/seccomp/profile.json busybox chmod 400 /etc/hosts
chmod: /etc/hosts: Operation not permitted
```
निम्नलिखित आउटपुट "डॉकर इंस्पेक्ट" को प्रदर्शित करता है जो प्रोफ़ाइल को दिखाता है:
```json
"SecurityOpt": [
"seccomp:{\"defaultAction\":\"SCMP_ACT_ALLOW\",\"syscalls\":[{\"name\":\"chmod\",\"action\":\"SCMP_ACT_ERRNO\"}]}"
],
```
### इसे डॉकर में निष्क्रिय करें

फ़्लैग के साथ एक कंटेनर लॉन्च करें: **`--security-opt seccomp=unconfined`**

Kubernetes 1.19 के रूप में, **सभी पॉड्स के लिए सेक्कॉम्प डिफ़ॉल्ट रूप से सक्षम है**। हालांकि, पॉड्स पर लागू डिफ़ॉल्ट सेक्कॉम्प प्रोफ़ाइल "RuntimeDefault" है, जो **कंटेनर रनटाइम द्वारा प्रदान किया जाता है** (जैसे कि Docker, containerd)। "RuntimeDefault" प्रोफ़ाइल कई सिस्टम कॉल को अनुमति देता है जबकि कुछ कॉल्स को अवैध या सामान्यतः कंटेनरों की आवश्यकता नहीं माना जाता है।

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने की उपलब्धता** चाहिए? [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* प्राप्त करें [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **में।**

</details>
