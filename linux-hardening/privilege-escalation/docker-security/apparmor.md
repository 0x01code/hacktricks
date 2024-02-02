# AppArmor

<details>

<summary><strong>शून्य से नायक तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) में या **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) को **फॉलो करें**.
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें.

</details>

## मूल जानकारी

**AppArmor** एक कर्नेल वृद्धि है जो **प्रोग्राम्स** को **सीमित** संसाधनों के सेट तक सीमित करती है **प्रति-प्रोग्राम प्रोफाइल्स** के साथ. प्रोफाइल्स नेटवर्क एक्सेस, रॉ सॉकेट एक्सेस, और मिलान पथों पर फाइलों को पढ़ने, लिखने या निष्पादित करने की अनुमति जैसी **क्षमताओं** को **अनुमति** दे सकते हैं.

यह एक Mandatory Access Control या **MAC** है जो **एक्सेस कंट्रोल** गुणों को **प्रोग्राम्स के लिए बांधता है बजाय उपयोगकर्ताओं के**.\
AppArmor संयम कर्नेल में लोड किए गए **प्रोफाइल्स के माध्यम से प्रदान किया जाता है**, आमतौर पर बूट पर.\
AppArmor प्रोफाइल्स **दो मोड्स** में हो सकते हैं:

* **Enforcement**: प्रवर्तन मोड में लोड किए गए प्रोफाइल्स प्रोफाइल में परिभाषित नीति के **प्रवर्तन के साथ-साथ नीति उल्लंघन प्रयासों की रिपोर्टिंग** का परिणाम होगा (या तो syslog या auditd के माध्यम से).
* **Complain**: शिकायत मोड में प्रोफाइल्स नीति को लागू नहीं करेंगे लेकिन इसके बजाय नीति **उल्लंघन** प्रयासों की **रिपोर्ट** करेंगे.

AppArmor लिनक्स पर कुछ अन्य MAC सिस्टम्स से भिन्न है: यह **पथ-आधारित** है, यह प्रवर्तन और शिकायत मोड प्रोफाइल्स का मिश्रण करने की अनुमति देता है, यह विकास को आसान बनाने के लिए शामिल फाइलों का उपयोग करता है, और इसकी अन्य लोकप्रिय MAC सिस्टम्स की तुलना में प्रवेश की बहुत कम बाधा है.

### AppArmor के भाग

* **कर्नेल मॉड्यूल**: वास्तविक काम करता है
* **नीतियाँ**: व्यवहार और संयम को परिभाषित करती हैं
* **पार्सर**: कर्नेल में नीतियों को लोड करता है
* **यूटिलिटीज**: apparmor के साथ इंटरैक्ट करने के लिए यूजरमोड प्रोग्राम्स

### प्रोफाइल्स पथ

Apparmor प्रोफाइल्स आमतौर पर _**/etc/apparmor.d/**_ में सहेजे जाते हैं\
`sudo aa-status` के साथ आप उन बाइनरीज की सूची बना पाएंगे जो किसी प्रोफाइल द्वारा प्रतिबंधित हैं. यदि आप प्रत्येक सूचीबद्ध बाइनरी के पथ के चर "/" को एक बिंदु से बदल सकते हैं और आपको उल्लिखित फोल्डर के अंदर apparmor प्रोफाइल का नाम मिल जाएगा.

उदाहरण के लिए, _/usr/bin/man_ के लिए एक **apparmor** प्रोफाइल _/etc/apparmor.d/usr.bin.man_ में स्थित होगी

### कमांड्स
```bash
aa-status     #check the current status
aa-enforce    #set profile to enforce mode (from disable or complain)
aa-complain   #set profile to complain mode (from diable or enforcement)
apparmor_parser #to load/reload an altered policy
aa-genprof    #generate a new profile
aa-logprof    #used to change the policy when the binary/program is changed
aa-mergeprof  #used to merge the policies
```
## प्रोफाइल बनाना

* प्रभावित कार्यक्रम को इंगित करने के लिए, **पूर्ण पथ और वाइल्डकार्ड्स** का उपयोग किया जा सकता है (फाइल ग्लोबिंग के लिए) फाइलों को निर्दिष्ट करने के लिए।
* बाइनरी के द्वारा **फाइलों** पर होने वाली पहुँच को इंगित करने के लिए निम्नलिखित **पहुँच नियंत्रण** का उपयोग किया जा सकता है:
* **r** (पढ़ना)
* **w** (लिखना)
* **m** (मेमोरी मैप के रूप में निष्पादनयोग्य)
* **k** (फाइल लॉकिंग)
* **l** (हार्ड लिंक्स का निर्माण)
* **ix** (दूसरे कार्यक्रम को निष्पादित करने के लिए, नए कार्यक्रम के नीति को विरासत में लेने के साथ)
* **Px** (वातावरण को साफ करने के बाद दूसरे प्रोफाइल के तहत निष्पादित करना)
* **Cx** (वातावरण को साफ करने के बाद एक बच्चे के प्रोफाइल के तहत निष्पादित करना)
* **Ux** (वातावरण को साफ करने के बाद अनियंत्रित रूप से निष्पादित करना)
* **वेरिएबल्स** को प्रोफाइल में परिभाषित किया जा सकता है और प्रोफाइल के बाहर से हेरफेर किया जा सकता है। उदाहरण के लिए: @{PROC} और @{HOME} (प्रोफाइल फाइल में #include \<tunables/global> जोड़ें)
* **अनुमति नियमों को ओवरराइड करने के लिए इनकार नियमों का समर्थन किया जाता है**।

### aa-genprof

प्रोफाइल बनाना शुरू करने के लिए आसानी से apparmor आपकी मदद कर सकता है। यह संभव है कि **apparmor द्वारा एक बाइनरी द्वारा किए गए कार्यों का निरीक्षण किया जाए और फिर आप तय करें कि कौन से कार्यों को आप अनुमति देना चाहते हैं या इनकार करना चाहते हैं**।\
आपको बस चलाना होगा:
```bash
sudo aa-genprof /path/to/binary
```
फिर, एक अलग कंसोल में वे सभी क्रियाएं करें जो आमतौर पर बाइनरी करती है:
```bash
/path/to/binary -a dosomething
```
तब, पहले कंसोल में "**s**" दबाएं और फिर रिकॉर्ड किए गए क्रियाओं में इंगित करें कि आप अनदेखा करना चाहते हैं, अनुमति देना चाहते हैं, या जो भी करना चाहते हैं। जब आप समाप्त कर लें तो "**f**" दबाएं और नया प्रोफाइल _/etc/apparmor.d/path.to.binary_ में बनाया जाएगा।

{% hint style="info" %}
तीर कुंजियों का उपयोग करके आप चुन सकते हैं कि आप क्या अनुमति देना/अस्वीकार करना/जो भी करना चाहते हैं।
{% endhint %}

### aa-easyprof

आप एक बाइनरी के एप्पार्मोर प्रोफाइल का एक टेम्पलेट भी बना सकते हैं:
```bash
sudo aa-easyprof /path/to/binary
# vim:syntax=apparmor
# AppArmor policy for binary
# ###AUTHOR###
# ###COPYRIGHT###
# ###COMMENT###

#include <tunables/global>

# No template variables specified

"/path/to/binary" {
#include <abstractions/base>

# No abstractions specified

# No policy groups specified

# No read paths specified

# No write paths specified
}
```
{% hint style="info" %}
ध्यान दें कि डिफ़ॉल्ट रूप से बनाई गई प्रोफ़ाइल में कुछ भी अनुमति नहीं है, इसलिए सब कुछ अस्वीकृत है। आपको `/etc/passwd r,` जैसी पंक्तियाँ जोड़नी होंगी ताकि बाइनरी `/etc/passwd` को पढ़ने की अनुमति हो, उदाहरण के लिए।
{% endhint %}

आप तब नई प्रोफ़ाइल को **लागू** कर सकते हैं
```bash
sudo apparmor_parser -a /etc/apparmor.d/path.to.binary
```
### लॉग्स से प्रोफाइल में संशोधन

निम्नलिखित उपकरण लॉग्स को पढ़ेगा और उपयोगकर्ता से पूछेगा कि क्या वह कुछ पता चले निषिद्ध क्रियाओं को अनुमति देना चाहता है:
```bash
sudo aa-logprof
```
{% hint style="info" %}
तीर कुंजियों का उपयोग करके आप जो चाहें उसे अनुमति देने/अस्वीकार करने/जो भी करना चाहें उसे चुन सकते हैं
{% endhint %}

### प्रोफाइल प्रबंधन
```bash
#Main profile management commands
apparmor_parser -a /etc/apparmor.d/profile.name #Load a new profile in enforce mode
apparmor_parser -C /etc/apparmor.d/profile.name #Load a new profile in complain mode
apparmor_parser -r /etc/apparmor.d/profile.name #Replace existing profile
apparmor_parser -R /etc/apparmor.d/profile.name #Remove profile
```
## लॉग्स

_`/var/log/audit/audit.log`_ से **AUDIT** और **DENIED** लॉग्स का उदाहरण, जो कि **`service_bin`** नामक एक्जीक्यूटेबल के हैं:
```bash
type=AVC msg=audit(1610061880.392:286): apparmor="AUDIT" operation="getattr" profile="/bin/rcat" name="/dev/pts/1" pid=954 comm="service_bin" requested_mask="r" fsuid=1000 ouid=1000
type=AVC msg=audit(1610061880.392:287): apparmor="DENIED" operation="open" profile="/bin/rcat" name="/etc/hosts" pid=954 comm="service_bin" requested_mask="r" denied_mask="r" fsuid=1000 ouid=0
```
आप इस जानकारी को इस्तेमाल करके भी प्राप्त कर सकते हैं:
```bash
sudo aa-notify -s 1 -v
Profile: /bin/service_bin
Operation: open
Name: /etc/passwd
Denied: r
Logfile: /var/log/audit/audit.log

Profile: /bin/service_bin
Operation: open
Name: /etc/hosts
Denied: r
Logfile: /var/log/audit/audit.log

AppArmor denials: 2 (since Wed Jan  6 23:51:08 2021)
For more information, please see: https://wiki.ubuntu.com/DebuggingApparmor
```
## Docker में Apparmor

ध्यान दें कि कैसे **docker-profile** नामक प्रोफाइल को डॉकर द्वारा डिफ़ॉल्ट रूप से लोड किया जाता है:
```bash
sudo aa-status
apparmor module is loaded.
50 profiles are loaded.
13 profiles are in enforce mode.
/sbin/dhclient
/usr/bin/lxc-start
/usr/lib/NetworkManager/nm-dhcp-client.action
/usr/lib/NetworkManager/nm-dhcp-helper
/usr/lib/chromium-browser/chromium-browser//browser_java
/usr/lib/chromium-browser/chromium-browser//browser_openjdk
/usr/lib/chromium-browser/chromium-browser//sanitized_helper
/usr/lib/connman/scripts/dhclient-script
docker-default
```
डिफ़ॉल्ट रूप से **Apparmor docker-default profile** यहाँ से उत्पन्न होता है: [https://github.com/moby/moby/tree/master/profiles/apparmor](https://github.com/moby/moby/tree/master/profiles/apparmor)

**docker-default profile सारांश**:

* सभी **नेटवर्किंग** के लिए **पहुँच**
* कोई भी **क्षमता** परिभाषित नहीं है (हालांकि, कुछ क्षमताएँ बेसिक बेस नियमों को शामिल करने से आएंगी जैसे कि #include \<abstractions/base> )
* किसी भी **/proc** फ़ाइल में **लिखना** **अनुमति नहीं** है
* /**proc** और /**sys** के अन्य **उपनिर्देशिकाएँ**/**फ़ाइलें** पढ़ने/लिखने/लॉक/लिंक/निष्पादन पहुँच से **वंचित** हैं
* **माउंट** करना **अनुमति नहीं** है
* **Ptrace** केवल उस प्रक्रिया पर चलाया जा सकता है जो **समान apparmor profile** द्वारा सीमित है

एक बार जब आप **डॉकर कंटेनर चलाते हैं** तो आपको निम्नलिखित आउटपुट दिखाई देना चाहिए:
```bash
1 processes are in enforce mode.
docker-default (825)
```
ध्यान दें कि **apparmor डिफ़ॉल्ट रूप से कंटेनर को दी गई capabilities विशेषाधिकारों को भी अवरुद्ध कर देगा**। उदाहरण के लिए, यह **/proc के अंदर लिखने की अनुमति को अवरुद्ध करने में सक्षम होगा भले ही SYS\_ADMIN क्षमता प्रदान की गई हो** क्योंकि डिफ़ॉल्ट रूप से docker apparmor प्रोफ़ाइल इस पहुँच को अस्वीकार करता है:
```bash
docker run -it --cap-add SYS_ADMIN --security-opt seccomp=unconfined ubuntu /bin/bash
echo "" > /proc/stat
sh: 1: cannot create /proc/stat: Permission denied
```
आपको इसकी पाबंदियों को बायपास करने के लिए **apparmor को निष्क्रिय करना होगा**:
```bash
docker run -it --cap-add SYS_ADMIN --security-opt seccomp=unconfined --security-opt apparmor=unconfined ubuntu /bin/bash
```
ध्यान दें कि डिफ़ॉल्ट रूप से **AppArmor** कंटेनर को अंदर से फोल्डर्स माउंट करने से भी **रोकेगा** यहां तक कि SYS\_ADMIN क्षमता के साथ भी।

ध्यान दें कि आप डॉकर कंटेनर में **क्षमताओं को जोड़/हटा** सकते हैं (यह अभी भी **AppArmor** और **Seccomp** जैसे सुरक्षा तरीकों द्वारा सीमित होगा):

* `--cap-add=SYS_ADMIN` `SYS_ADMIN` क्षमता दें
* `--cap-add=ALL` सभी क्षमताएँ दें
* `--cap-drop=ALL --cap-add=SYS_PTRACE` सभी क्षमताएँ हटाएं और केवल `SYS_PTRACE` दें

{% hint style="info" %}
आमतौर पर, जब आप **पाते हैं** कि आपके पास एक **विशेषाधिकार प्राप्त क्षमता** उपलब्ध है **अंदर** एक **डॉकर** कंटेनर में **लेकिन** कुछ हिस्सा **exploit काम नहीं कर रहा है**, यह इसलिए होगा क्योंकि डॉकर **apparmor इसे रोक रहा होगा**।
{% endhint %}

### उदाहरण

([**यहाँ**](https://sreeninet.wordpress.com/2016/03/06/docker-security-part-2docker-engine/) से उदाहरण)

AppArmor कार्यक्षमता को समझाने के लिए, मैंने एक नया डॉकर प्रोफ़ाइल "mydocker" बनाया जिसमें निम्नलिखित पंक्ति जोड़ी गई:
```
deny /etc/* w,   # deny write for all files directly in /etc (not in a subdir)
```
प्रोफ़ाइल को सक्रिय करने के लिए, हमें निम्नलिखित करना होगा:
```
sudo apparmor_parser -r -W mydocker
```
प्रोफाइल्स की सूची बनाने के लिए, हम निम्नलिखित कमांड कर सकते हैं। नीचे दिया गया कमांड मेरे नए AppArmor प्रोफाइल की सूची दिखा रहा है।
```
$ sudo apparmor_status  | grep mydocker
mydocker
```
नीचे दिखाए गए अनुसार, जब हम “/etc/” में परिवर्तन करने का प्रयास करते हैं, तो हमें त्रुटि मिलती है क्योंकि AppArmor प्रोफाइल “/etc” में लिखने की अनुमति को रोक रहा है।
```
$ docker run --rm -it --security-opt apparmor:mydocker -v ~/haproxy:/localhost busybox chmod 400 /etc/hostname
chmod: /etc/hostname: Permission denied
```
### AppArmor Docker Bypass1

आप यह पता लगा सकते हैं कि कौन सी **apparmor profile एक container को चला रही है** इसका उपयोग करके:
```bash
docker inspect 9d622d73a614 | grep lowpriv
"AppArmorProfile": "lowpriv",
"apparmor=lowpriv"
```
फिर, आप निम्नलिखित पंक्ति को चलाकर **उपयोग में लाई जा रही सटीक प्रोफ़ाइल का पता लगा सकते हैं**:
```bash
find /etc/apparmor.d/ -name "*lowpriv*" -maxdepth 1 2>/dev/null
```
### AppArmor Docker Bypass2 का उल्लंघन

**AppArmor पथ-आधारित है**, इसका मतलब है कि यदि यह **`/proc`** जैसी डायरेक्टरी के अंदर की फाइलों की **रक्षा** कर रहा हो, लेकिन यदि आप **कंटेनर को कैसे चलाया जाएगा इसे कॉन्फ़िगर** कर सकते हैं, तो आप होस्ट की proc डायरेक्टरी को **`/host/proc`** के अंदर **माउंट** कर सकते हैं और यह **AppArmor द्वारा अब सुरक्षित नहीं रहेगा**।

### AppArmor Shebang Bypass का उल्लंघन

[**इस बग**](https://bugs.launchpad.net/apparmor/+bug/1911431) में आप एक उदाहरण देख सकते हैं कि कैसे **यहां तक कि यदि आप perl को कुछ संसाधनों के साथ चलाने से रोक रहे हैं**, अगर आप बस एक शेल स्क्रिप्ट बनाते हैं जिसमें पहली पंक्ति में **`#!/usr/bin/perl`** निर्दिष्ट करते हैं और आप **फाइल को सीधे निष्पादित करते हैं**, तो आप जो चाहें वह निष्पादित कर पाएंगे। उदाहरण के लिए:
```perl
echo '#!/usr/bin/perl
use POSIX qw(strftime);
use POSIX qw(setuid);
POSIX::setuid(0);
exec "/bin/sh"' > /tmp/test.pl
chmod +x /tmp/test.pl
/tmp/test.pl
```
<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग ट्रिक्स शेयर करें।

</details>
