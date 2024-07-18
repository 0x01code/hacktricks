# AppArmor

{% hint style="success" %}
सीखें और प्रैक्टिस करें AWS हैकिंग: <img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण AWS रेड टीम एक्सपर्ट (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
सीखें और प्रैक्टिस करें GCP हैकिंग: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण GCP रेड टीम एक्सपर्ट (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) और **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** को** **फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, हैकट्रिक्स**](https://github.com/carlospolop/hacktricks) **और हैकट्रिक्स क्लाउड**](https://github.com/carlospolop/hacktricks-cloud) **github रेपो में पीआर जमा करके।

</details>
{% endhint %}

### [WhiteIntel](https://whiteintel.io)

<figure><img src="../../../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) एक **डार्क-वेब** पर आधारित खोज इंजन है जो **मुफ्त** सुविधाएं प्रदान करता है ताकि यह जांच सके कि कोई कंपनी या उसके ग्राहकों को **स्टीलर मैलवेयर्स** द्वारा **कंप्रोमाइज** किया गया है या नहीं।

व्हाइटइंटेल का मुख्य उद्देश्य खाता हासिल करने और रैंसमवेयर हमलों का मुकाबला करना है जो जानकारी चोरी करने वाले मैलवेयर से होते हैं।

आप उनकी वेबसाइट चेक कर सकते हैं और **मुफ्त** में उनका इंजन आजमा सकते हैं:

{% embed url="https://whiteintel.io" %}

***

## मूल जानकारी

AppArmor एक **कर्नेल एन्हांसमेंट** है जो कार्यक्रमों के लिए प्रोफाइल के माध्यम से उपलब्ध संसाधनों को प्रतिबंधित करने के लिए डिज़ाइन किया गया है, अनिवार्य एक्सेस नियंत्रण (MAC) को प्रभावी रूप से लागू करके एक्सेस नियंत्रण गुणों को सीधे प्रोग्रामों से युक्त करने के द्वारा। यह सिस्टम **कर्नेल में प्रोफाइल लोड करके** काम करता है, सामान्यत: बूट के दौरान, और ये प्रोफाइल निर्दिष्ट करते हैं कि कोई प्रोग्राम किस संसाधन तक पहुंच सकता है, जैसे नेटवर्क कनेक्शन, रॉ सॉकेट एक्सेस, और फ़ाइल अनुमतियाँ।

AppArmor प्रोफाइल के लिए दो प्रचलित मोड हैं:

* **निषेध मोड**: यह मोड प्रोफाइल में परिभाषित नीतियों को सक्रिय रूप से लागू करता है, जो इन नीतियों का उल्लंघन करने वाली क्रियाओं को रोकता है और syslog या auditd जैसे सिस्टम के माध्यम से इन्हें उल्लंघन करने की कोशिशों को लॉग करता है।
* **शिकायत मोड**: निषेध मोड की तरह, शिकायत मोड उन क्रियाओं को नहीं रोकता जो प्रोफाइल की नीतियों के खिलाफ जाती हैं। इसके बजाय, यह इन प्रयासों को नीतियों के उल्लंघन के रूप में लॉग करता है बिना प्रतिबंधों को लागू करने।

### AppArmor के घटक

* **कर्नेल मॉड्यूल**: नीतियों के प्रवर्तन के लिए जिम्मेदार।
* **नीतियाँ**: कार्यक्रम व्यवहार और संसाधन एक्सेस के लिए नियम और प्रतिबंध निर्धारित करती हैं।
* **पार्सर**: नीतियों को कर्नेल में प्रवर्तन या रिपोर्टिंग के लिए लोड करता है।
* **उपयोगिताएं**: ये उपयोगकर्ता मोड प्रोग्राम्स के साथ बातचीत करने और AppArmor का प्रबंधन करने के लिए एक इंटरफेस प्रदान करने वाले प्रोग्राम हैं।

### प्रोफाइल पथ

Apparmor प्रोफाइल आम तौर पर _**/etc/apparmor.d/**_ में सहेजे जाते हैं।\
`sudo aa-status` के साथ आप उन बाइनरी को सूचीबद्ध कर सकते हैं जिन्हें किसी प्रोफाइल द्वारा प्रतिबंधित किया गया है। यदि आप प्रत्येक सूचीबद्ध बाइनरी के पथ का चार "/" को उसके नाम में डॉट में बदल सकते हैं तो आपको उल्लिखित फ़ोल्डर के अंदर एपार्मर प्रोफाइल का नाम प्राप्त होगा।

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
## प्रोफ़ाइल बनाना

* प्रभावित एक्जीक्यूटेबल को दर्शाने के लिए, **पूर्ण मार्ग और वाइल्डकार्ड** (फ़ाइल ग्लॉबिंग के लिए) की अनुमति है।
* **फ़ाइलों** पर एक्सेस नियंत्रण को दर्शाने के लिए निम्नलिखित **एक्सेस नियंत्रण** का उपयोग किया जा सकता है:
* **r** (पढ़ना)
* **w** (लिखना)
* **m** (मेमोरी मैप के रूप में एक्जीक्यूटेबल)
* **k** (फ़ाइल लॉकिंग)
* **l** (हार्ड लिंक्स बनाना)
* **ix** (नए प्रोग्राम को नीति विरासत में एक्जीक्यूट करने के लिए)
* **Px** (पर्यावरण को साफ करने के बाद एक अन्य प्रोफ़ाइल के तहत एक्जीक्यूट करें)
* **Cx** (पर्यावरण को साफ करने के बाद एक बच्चा प्रोफ़ाइल के तहत एक्जीक्यूट करें)
* **Ux** (पर्यावरण को साफ करने के बाद अनियंत्रित एक्जीक्यूट करें)
* **प्रोफ़ाइल में चर** परिभाषित किए जा सकते हैं और प्रोफ़ाइल के बाहर से उन्हें परिवर्तित किया जा सकता है। उदाहरण के लिए: @{PROC} और @{HOME} (प्रोफ़ाइल फ़ाइल में #include \<tunables/global> जोड़ें)
* **अनुमति नियमों को अधिलेखित करने के लिए निषेध नियम समर्थित हैं**।

### aa-genprof

एक प्रोफ़ाइल बनाना शुरू करने के लिए आपकी मदद करने के लिए apparmor का उपयोग किया जा सकता है। **एक्शन्स को जाँचने के लिए apparmor की सहायता लेकर एक एक्जीक्यूटेबल द्वारा किए गए कार्रवाईयों को जाँचने के बाद आपको यह निर्धारित करने देने की संभावना है कि आप किस कार्रवाई को अनुमति देना चाहते हैं या निषेध करना चाहते हैं**।\
आपको बस यह रन करना है:
```bash
sudo aa-genprof /path/to/binary
```
फिर, एक विभिन्न कंसोल में वह सभी क्रियाएँ करें जो बाइनरी सामान्य रूप से करेगा:
```bash
/path/to/binary -a dosomething
```
तो, पहली कंसोल में "**s**" दबाएं और फिर रिकॉर्डेड क्रियाओं में इंडिकेट करें कि आप क्या अनदेखा करना चाहते हैं, अनुमति देना चाहते हैं, या कुछ और। जब आप समाप्त हो जाएं तो "**f**" दबाएं और नया प्रोफ़ाइल _/etc/apparmor.d/path.to.binary_ में बनाया जाएगा

{% hint style="info" %}
एरो कुंजी का उपयोग करके आप चुन सकते हैं कि आप क्या अनुमति देना/इनकार/कुछ और चाहते हैं
{% endhint %}

### aa-easyprof

आप एक बाइनरी के एक apparmor प्रोफ़ाइल का टेम्प्लेट भी बना सकते हैं:
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
ध्यान दें कि एक बनाए गए प्रोफ़ाइल में डिफ़ॉल्ट रूप से कुछ भी अनुमति नहीं है, इसलिए सब कुछ निषेधित है। उदाहरण के लिए बाइनरी को `/etc/passwd r,` पढ़ने की अनुमति देने के लिए आपको लाइन जोड़नी होगी।
{% endhint %}

आप फिर नए प्रोफ़ाइल को **लागू** कर सकते हैं
```bash
sudo apparmor_parser -a /etc/apparmor.d/path.to.binary
```
### लॉग से प्रोफ़ाइल को संशोधित करना

निम्नलिखित टूल लॉग पढ़ेगा और उपयोगकर्ता से पूछेगा कि क्या वह कुछ प्रतिबंधित क्रियाओं की अनुमति देना चाहता है:
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
आप इस जानकारी को निम्नलिखित का उपयोग करके भी प्राप्त कर सकते हैं:
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

ध्यान दें कि डॉकर का प्रोफ़ाइल **docker-profile** डिफ़ॉल्ट रूप से लोड होता है:
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
* **कोई क्षमता** परिभाषित नहीं है (हालांकि, कुछ क्षमताएँ मूल बेस नियम शामिल करने से आएगी जैसे #include \<abstractions/base> )
* किसी भी **/proc** फ़ाइल में **लेखन** की **अनुमति नहीं** है
* **अन्य उपनिर्देशिकाएँ**/**फ़ाइलें** /**proc** और /**sys** की **पढ़ने/लिखने/लॉक/लिंक/क्रियान्वयन पहुंच** को **इनकार किया गया है**
* **माउंट** की **अनुमति नहीं** है
* **Ptrace** केवल उस प्रक्रिया पर चलाया जा सकता है जो **एक ही apparmor प्रोफ़ाइल द्वारा सीमित** है

एक बार जब आप **एक डॉकर कंटेनर चलाते हैं** तो आपको निम्नलिखित आउटपुट दिखाई देना चाहिए:
```bash
1 processes are in enforce mode.
docker-default (825)
```
नोट करें कि **एपार्मर** डिफ़ॉल्ट रूप से कंटेनर को दी गई **क्षमताएँ भी रोक देगा**। उदाहरण के लिए, यह **/proc के अंदर लिखने की अनुमति को रोक सकता है भले ही SYS\_ADMIN क्षमता दी गई हो** क्योंकि डॉकर एपार्मर प्रोफ़ाइल डिफ़ॉल्ट रूप से इस पहुंच को इनकार करता है:
```bash
docker run -it --cap-add SYS_ADMIN --security-opt seccomp=unconfined ubuntu /bin/bash
echo "" > /proc/stat
sh: 1: cannot create /proc/stat: Permission denied
```
आपको **apparmor को अक्षम** करना होगा ताकि इसकी प्रतिबंधिताओं को अनदेखा किया जा सके:
```bash
docker run -it --cap-add SYS_ADMIN --security-opt seccomp=unconfined --security-opt apparmor=unconfined ubuntu /bin/bash
```
नोट करें कि डिफ़ॉल्ट रूप से **AppArmor** भी **कंटेनर को माउंट करने से रोकेगा** अंदर से भी SYS\_ADMIN क्षमता के साथ।

नोट करें कि आप **क्षमताएँ जोड़ सकते/हटा सकते हैं** डॉकर कंटेनर में (यह अब भी **AppArmor** और **Seccomp** जैसी सुरक्षा विधियों द्वारा प्रतिबंधित रहेगा):

* `--cap-add=SYS_ADMIN` `SYS_ADMIN` क्षमता दें
* `--cap-add=ALL` सभी क्षमताएँ दें
* `--cap-drop=ALL --cap-add=SYS_PTRACE` सभी क्षमताएँ हटाएं और केवल `SYS_PTRACE` दें

{% hint style="info" %}
सामान्यत: जब आपको पाता चलता है कि आपके पास एक **विशेषाधिकार क्षमता** उपलब्ध है **कंटेनर के अंदर** **लेकिन** कुछ हिस्सा **एक्सप्लॉइट काम नहीं कर रहा है**, तो यह इसलिए हो सकता है कि डॉकर **AppArmor इसे रोक रहा होगा**।
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
अद्यतन किए गए AppArmor प्रोफ़ाइल को सूचीबद्ध करने के लिए, हम निम्नलिखित कमांड का उपयोग कर सकते हैं। नीचे दिए गए कमांड मेरे नए AppArmor प्रोफ़ाइल को सूचीबद्ध कर रहा है।
```
$ sudo apparmor_status  | grep mydocker
mydocker
```
जैसा नीचे दिखाया गया है, हमें " /etc/ " बदलने की कोशिश करने पर त्रुटि मिलती है क्योंकि AppArmor प्रोफ़ाइल " /etc/ " में लेखन पहुंचने से रोक रहा है।
```
$ docker run --rm -it --security-opt apparmor:mydocker -v ~/haproxy:/localhost busybox chmod 400 /etc/hostname
chmod: /etc/hostname: Permission denied
```
### AppArmor डॉकर बायपास1

आप निम्नलिखित का उपयोग करके **पता लगा सकते हैं कि कौन सा **apparmor प्रोफ़ाइल** एक कंटेनर चला रहा है**:
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

**AppArmor का मार्ग-आधारित है**, इसका मतलब है कि यदि यह **फ़ाइलें सुरक्षित कर रहा है** तो भी एक निर्देशिका के भीतर फ़ाइलें जैसे **`/proc`** अगर आप **कॉन्टेनर को कैसे चलाया जाएगा वह विन्यासित कर सकते हैं**, तो आप **माउंट** कर सकते हैं होस्ट के proc निर्देशिका को **`/host/proc`** और यह **अब AppArmor द्वारा सुरक्षित नहीं होगा**।

### AppArmor Shebang Bypass

[**इस बग**](https://bugs.launchpad.net/apparmor/+bug/1911431) में आप देख सकते हैं कि **यदि आप किसी विशेष संसाधन के साथ पर्ल को चलाने से रोक रहे हैं**, तो भी यदि आप केवल पहली पंक्ति में **`#!/usr/bin/perl`** निर्दिष्ट करते हैं और आप **फ़ाइल को सीधे निष्पादित करते हैं**, तो आप चाहें जो भी चाहें निष्पादित कर सकते हैं। उदा:
```perl
echo '#!/usr/bin/perl
use POSIX qw(strftime);
use POSIX qw(setuid);
POSIX::setuid(0);
exec "/bin/sh"' > /tmp/test.pl
chmod +x /tmp/test.pl
/tmp/test.pl
```
### [WhiteIntel](https://whiteintel.io)

<figure><img src="../../../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) एक **डार्क-वेब** प्रेरित खोज इंजन है जो **मुफ्त** सुविधाएं प्रदान करता है ताकि जांच सकें कि कोई कंपनी या उसके ग्राहकों को **स्टीलर मैलवेयर्स** द्वारा **क्षति** पहुंचाई गई है या नहीं।

WhiteIntel का मुख्य उद्देश्य खाता हासिल करने और जानकारी चुराने वाले मैलवेयर से होने वाले रैंसमवेयर हमलों का मुकाबला करना है।

आप उनकी वेबसाइट चेक कर सकते हैं और **मुफ्त** में उनका इंजन प्रयास कर सकते हैं:

{% embed url="https://whiteintel.io" %}

{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण AWS रेड टीम एक्सपर्ट (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण GCP रेड टीम एक्सपर्ट (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 पर **फॉलो** करें [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **हैकिंग ट्रिक्स साझा करें PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में।

</details>
{% endhint %}
