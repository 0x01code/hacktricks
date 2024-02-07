# AppArmor

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) और **मुझे** **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks) github repos में PRs सबमिट करके।

</details>

## मूल जानकारी

AppArmor एक **कर्नेल एन्हांसमेंट है जो प्रोग्रामों के लिए प्रोफाइल के माध्यम से उपलब्ध संसाधनों को प्रतिबंधित करने के लिए डिज़ाइन किया गया है**, अभिक्रियात्मक रूप से अनिवार्य पहुंच नियंत्रण (MAC) को कार्यान्वित करता है जिसमें पहुंच नियंत्रण गुण यूज़र के बजाय सीधे प्रोग्रामों से जुड़े होते हैं। यह सिस्टम **कर्नेल में प्रोफाइल लोड करके** काम करता है, आम तौर पर बूट के दौरान, और ये प्रोफाइल निर्दिष्ट करते हैं कि एक प्रोग्राम किस संसाधन तक पहुंच सकता है, जैसे नेटवर्क कनेक्शन, रॉ सॉकेट पहुंच, और फ़ाइल अनुमतियाँ।

AppArmor प्रोफाइल के लिए दो प्रचलित मोड हैं:

- **निषेध मोड**: यह मोड प्रोफाइल में परिभाषित नीतियों को सक्रिय रूप से लागू करता है, जो इन नीतियों का उल्लंघन करने वाली क्रियाओं को रोकता है और syslog या auditd जैसे सिस्टम के माध्यम से इन नीतियों का उल्लंघन करने की कोशिशों को लॉग करता है।
- **शिकायत मोड**: निषेध मोड की तरह, शिकायत मोड उन क्रियाओं को नहीं रोकता जो प्रोफाइल की नीतियों के खिलाफ जाती हैं। इसके बजाय, यह इन प्रयासों को नीति उल्लंघन के रूप में लॉग करता है बिना प्रतिबंधों को लागू करने।

### AppArmor के घटक

- **कर्नेल मॉड्यूल**: नीतियों के प्रवर्तन के लिए जिम्मेदार।
- **नीतियाँ**: प्रोग्राम व्यवहार और संसाधन पहुंच के लिए नियम और प्रतिबंध निर्धारित करती हैं।
- **पार्सर**: नीतियों को कर्नेल में प्रवर्तन या रिपोर्टिंग के लिए लोड करता है।
- **उपयोगकर्ता**: ये उपयोगकर्ता मोड के कार्यक्रम हैं जो AppArmor के साथ इंटरैक्ट करने और प्रबंधन के लिए एक इंटरफेस प्रदान करते हैं।

### प्रोफाइल पथ

Apparmor प्रोफाइल आम तौर पर _**/etc/apparmor.d/**_ में सहेजे जाते हैं।\
`sudo aa-status` के साथ आप उन बाइनरी को सूचीबद्ध कर सकते हैं जिन्हें किसी प्रोफाइल द्वारा प्रतिबंधित किया गया है। यदि आप प्रत्येक सूचीबद्ध बाइनरी के पथ के लिए एक डॉट के बजाय "/" को बदल सकते हैं तो आपको उल्लिखित फ़ोल्डर के अंदर के apparmor प्रोफ़ाइल का नाम प्राप्त होगा।

उदाहरण के लिए, _/usr/bin/man_ के लिए एक **apparmor** प्रोफाइल _/etc/apparmor.d/usr.bin.man_ में स्थित होगा।

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
## एक प्रोफ़ाइल बनाना

* प्रभावित एक्जीक्यूटेबल को दर्शाने के लिए, **पूर्ण मार्ग और वाइल्डकार्ड** (फ़ाइल ग्लॉबिंग के लिए) की अनुमति है।
* **फ़ाइलों** पर एक्सेस नियंत्रण को दर्शाने के लिए निम्नलिखित **एक्सेस नियंत्रण** का उपयोग किया जा सकता है:
* **r** (पढ़ना)
* **w** (लिखना)
* **m** (मेमोरी मैप के रूप में एक्जीक्यूटेबल)
* **k** (फ़ाइल लॉकिंग)
* **l** (हार्ड लिंक्स बनाना)
* **ix** (नए प्रोग्राम के साथ एक और प्रोग्राम को नियमित करने के लिए)
* **Px** (पर्यावरण को साफ करने के बाद एक और प्रोफ़ाइल के तहत निष्पादित करें)
* **Cx** (पर्यावरण को साफ करने के बाद एक बच्चा प्रोफ़ाइल के तहत निष्पादित करें)
* **Ux** (पर्यावरण को साफ करने के बाद अनियंत्रित निष्पादित करें)
* **प्रोफ़ाइल में चर निर्धारित किए जा सकते हैं और प्रोफ़ाइल के बाहर से प्रबंधित किए जा सकते हैं। उदाहरण के लिए: @{PROC} और @{HOME} (प्रोफ़ाइल फ़ाइल में #include \<tunables/global> जोड़ें)
* **अनुमति नियमों को अधिलेखित करने के लिए निषेध नियम समर्थित हैं**।

### aa-genprof

एक प्रोफ़ाइल बनाना शुरू करने के लिए आपकी मदद करने के लिए apparmor का उपयोग किया जा सकता है। **एक्शन्स द्वारा किए गए कार्यों की जाँच करने के लिए apparmor की मदद ले सकते हैं और फिर आपको यह निर्णय लेने की अनुमति देता है कि आप किस कार्रवाई को अनुमति देना चाहते हैं या नकारना चाहते हैं**।\
आपको बस निम्नलिखित को चलाना है:
```bash
sudo aa-genprof /path/to/binary
```
फिर, एक विभिन्न कंसोल में उन सभी क्रियाएँ करें जो बाइनरी सामान्य रूप से करेगा:
```bash
/path/to/binary -a dosomething
```
फिर, पहली कंसोल में "**s**" दबाएं और फिर रिकॉर्डेड क्रियाएँ इंडिकेट करें कि आप क्या अनदेखा करना चाहते हैं, अनुमति देना चाहते हैं, या कुछ और। जब आप समाप्त हो जाएं तो "**f**" दबाएं और नया प्रोफ़ाइल _/etc/apparmor.d/path.to.binary_ में बनाया जाएगा

{% hint style="info" %}
एरो की कुंजियों का उपयोग करके आप चुन सकते हैं कि आप क्या अनुमति देना/अस्वीकार करना/कुछ और करना चाहते हैं
{% endhint %}

### aa-easyprof

आप एक ऐपार्मर प्रोफ़ाइल का टेम्पलेट भी बना सकते हैं एक बाइनरी का:
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
ध्यान दें कि एक बनाए गए प्रोफ़ाइल में डिफ़ॉल्ट रूप से कुछ भी अनुमति नहीं है, इसलिए सब कुछ निषेधित है। उदाहरण के लिए, बाइनरी को `/etc/passwd r,` पढ़ने की अनुमति देने के लिए आपको `/etc/passwd r,` जैसी लाइनें जोड़नी होगी।
{% endhint %}

आप फिर नए प्रोफ़ाइल को **लागू** कर सकते हैं
```bash
sudo apparmor_parser -a /etc/apparmor.d/path.to.binary
```
### लॉग्स से प्रोफ़ाइल को संशोधित करना

निम्नलिखित टूल लॉग्स को पढ़ेगा और उपयोगकर्ता से पूछेगा कि क्या वह कुछ पता चली प्रतिबंधित क्रियाओं को अनुमति देना चाहता है:
```bash
sudo aa-logprof
```
{% hint style="info" %}
एरो कुंजी का उपयोग करके आप चुन सकते हैं कि आप क्या अनुमति देना/विरोध/या कुछ भी करना चाहते हैं।
{% endhint %}

### प्रोफ़ाइल प्रबंधन
```bash
#Main profile management commands
apparmor_parser -a /etc/apparmor.d/profile.name #Load a new profile in enforce mode
apparmor_parser -C /etc/apparmor.d/profile.name #Load a new profile in complain mode
apparmor_parser -r /etc/apparmor.d/profile.name #Replace existing profile
apparmor_parser -R /etc/apparmor.d/profile.name #Remove profile
```
## लॉग

**AUDIT** और **DENIED** लॉग का उदाहरण _/var/log/audit/audit.log_ से एक्जीक्यूटेबल **`service_bin`** का:
```bash
type=AVC msg=audit(1610061880.392:286): apparmor="AUDIT" operation="getattr" profile="/bin/rcat" name="/dev/pts/1" pid=954 comm="service_bin" requested_mask="r" fsuid=1000 ouid=1000
type=AVC msg=audit(1610061880.392:287): apparmor="DENIED" operation="open" profile="/bin/rcat" name="/etc/hosts" pid=954 comm="service_bin" requested_mask="r" denied_mask="r" fsuid=1000 ouid=0
```
आप इस जानकारी को निम्नलिखित का उपयोग करके प्राप्त कर सकते हैं:
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
## डॉकर में Apparmor

नोट करें कि डॉकर का प्रोफ़ाइल **docker-profile** डिफ़ॉल्ट रूप से लोड होता है:
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
By default **Apparmor docker-default profile** is generated from [https://github.com/moby/moby/tree/master/profiles/apparmor](https://github.com/moby/moby/tree/master/profiles/apparmor)

**docker-default profile Summary**:

* **सभी नेटवर्किंग** तक **पहुंच**
* **कोई क्षमता** परिभाषित नहीं है (हालांकि, कुछ क्षमताएँ मूल बेस नियम शामिल करने से आएगी जैसे #include \<abstractions/base>)
* किसी भी **/proc** फ़ाइल में **लेखन** की **अनुमति नहीं** है
* **/proc** और /**sys** के अन्य **उपनिर्देशिकाएँ**/**फ़ाइलें** पढ़ने/लिखने/लॉक/लिंक/क्रियान्वयन पहुंच को **इनकार किया गया है**
* **माउंट** की **अनुमति नहीं** है
* **Ptrace** केवल उस प्रक्रिया पर चलाया जा सकता है जो **एक ही apparmor प्रोफ़ाइल द्वारा सीमित** है

एक बार जब आप **एक डॉकर कंटेनर चलाते हैं** तो आपको निम्नलिखित आउटपुट दिखाई देना चाहिए:
```bash
1 processes are in enforce mode.
docker-default (825)
```
नोट करें कि **एपार्मर** डिफ़ॉल्ट रूप से कंटेनर को दी गई **क्षमताएँ भी रोक देगा**। उदाहरण के लिए, यह **/proc के अंदर लिखने की अनुमति को रोक सकता है, भले ही SYS\_ADMIN क्षमता दी गई हो** क्योंकि डॉकर एपार्मर प्रोफ़ाइल डिफ़ॉल्ट रूप से इस पहुंच को इनकार करता है:
```bash
docker run -it --cap-add SYS_ADMIN --security-opt seccomp=unconfined ubuntu /bin/bash
echo "" > /proc/stat
sh: 1: cannot create /proc/stat: Permission denied
```
आपको **apparmor को अक्षम** करना होगा ताकि इसकी प्रतिबंधिताओं को अनदेखा किया जा सके:
```bash
docker run -it --cap-add SYS_ADMIN --security-opt seccomp=unconfined --security-opt apparmor=unconfined ubuntu /bin/bash
```
ध्यान दें कि डिफ़ॉल्ट रूप से **AppArmor** भी **कंटेनर को माउंट करने से रोकेगा** भी SYS\_ADMIN क्षमता के साथ।

ध्यान दें कि आप **कैपेबिलिटी** को डॉकर कंटेनर में **जोड़ सकते/हटा सकते** हैं (यह **AppArmor** और **Seccomp** जैसी सुरक्षा विधियों द्वारा अभी भी प्रतिबंधित होगा):

* `--cap-add=SYS_ADMIN` `SYS_ADMIN` कैप देना
* `--cap-add=ALL` सभी कैप देना
* `--cap-drop=ALL --cap-add=SYS_PTRACE` सभी कैप हटाना और केवल `SYS_PTRACE` देना

{% hint style="info" %}
सामान्यत: जब आपको एक **विशेषाधिकारिक क्षमता** उपलब्ध होती है **डॉकर** कंटेनर **के भीतर** लेकिन **कुछ हिस्सा** एक **उत्पीड़न काम नहीं कर रहा हो**, तो यह इसलिए हो सकता है कि डॉकर **AppArmor इसे रोक रहा होगा**।
{% endhint %}

### उदाहरण

(उदाहरण [**यहाँ से**](https://sreeninet.wordpress.com/2016/03/06/docker-security-part-2docker-engine/) से)

AppArmor कार्यक्षमता को विस्तारित करने के लिए, मैंने एक नया डॉकर प्रोफ़ाइल "mydocker" बनाया जिसमें निम्नलिखित पंक्ति जोड़ी गई:
```
deny /etc/* w,   # deny write for all files directly in /etc (not in a subdir)
```
इस प्रोफ़ाइल को सक्रिय करने के लिए, हमें निम्नलिखित कार्रवाई करनी होगी:
```
sudo apparmor_parser -r -W mydocker
```
इसके लिए प्रोफाइल सूची देखने के लिए, हम निम्नलिखित कमांड कर सकते हैं। नीचे दिए गए कमांड मेरे नए AppArmor प्रोफाइल को सूचीबद्ध कर रहा है।
```
$ sudo apparmor_status  | grep mydocker
mydocker
```
जैसा नीचे दिखाया गया है, हमें " /etc/ " बदलने की कोशिश करने पर त्रुटि मिलती है क्योंकि AppArmor प्रोफ़ाइल " /etc/ " में लेखन अधिकार को रोक रहा है।
```
$ docker run --rm -it --security-opt apparmor:mydocker -v ~/haproxy:/localhost busybox chmod 400 /etc/hostname
chmod: /etc/hostname: Permission denied
```
### AppArmor Docker Bypass1

आप निम्नलिखित का उपयोग करके **कैसे पता लगाएं कि कौन सा **apparmor प्रोफ़ाइल** एक कंटेनर चला रहा है**:
```bash
docker inspect 9d622d73a614 | grep lowpriv
"AppArmorProfile": "lowpriv",
"apparmor=lowpriv"
```
फिर, आप निम्नलिखित पंक्ति को चला सकते हैं **उपयोग किया जा रहा सटीक प्रोफ़ाइल खोजने के लिए**:
```bash
find /etc/apparmor.d/ -name "*lowpriv*" -maxdepth 1 2>/dev/null
```
### AppArmor Docker Bypass2

**अजीब मामले में आप **apparmor डॉकर प्रोफ़ाइल को संशोधित कर सकते हैं और इसे पुनः लोड कर सकते हैं।** आप प्रतिबंधों को हटा सकते हैं और उन्हें "अनदेखा" कर सकते हैं।

### AppArmor Shebang Bypass

[**इस बग**](https://bugs.launchpad.net/apparmor/+bug/1911431) में आप देख सकते हैं कि **यदि आप किसी विशेष संसाधनों के साथ पर्ल को चलाने से रोक रहे हैं**, तो भी यदि आप केवल पहली पंक्ति में **`#!/usr/bin/perl`** निर्दिष्ट करते हैं और आप **फ़ाइल को सीधे निष्पादित** करते हैं, तो आप जो चाहें वह निष्पादित कर सकते हैं। उदा।:
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

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

दूसरे तरीके HackTricks का समर्थन करने के लिए:

* अगर आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks swag प्राप्त करें**](https://peass.creator-spring.com)
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें** द्वारा PRs सबमिट करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>
