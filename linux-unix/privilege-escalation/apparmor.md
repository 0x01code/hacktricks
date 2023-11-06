<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करना चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)

- प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपने हैकिंग ट्रिक्स साझा करें, [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके।**

</details>


# मूलभूत जानकारी

**AppArmor** एक कर्नल एन्हांसमेंट है जो **प्रोग्रामों** को **सीमित संसाधनों** के **साथ सीमित** सेट में बाधित करता है। प्रोफ़ाइल्स नेटवर्क एक्सेस, रॉ सॉकेट एक्सेस और मिलते पथों पर फ़ाइलों को पढ़ने, लिखने या निष्पादित करने की अनुमति जैसी **क्षमताओं** को **अनुमति** दे सकते हैं।

यह एक अनिवार्य पहुंच नियंत्रण या **MAC** है जो **उपयोगकर्ताओं के बजाय प्रोग्रामों** को बाधित करने के लिए **पहुंच नियंत्रण** गुणों को **बाधित करता है**।\
AppArmor बाधा आमतौर पर कर्नल में लोड किए जाने वाले प्रोफ़ाइल्स के माध्यम से प्रदान की जाती है।\
AppArmor प्रोफ़ाइल्स दो मोड में हो सकते हैं:

* **निष्पादन**: निष्पादन मोड में लोड किए गए प्रोफ़ाइल्स प्रोफ़ाइल में परिभाषित नीति के **निष्पादन के परिणामस्वरूप होंगे और रिपोर्टिंग** नीति उल्लंघन प्रयासों को (सिसलॉग या ऑडिटडी के माध्यम से) करेंगे।
* **शिकायत**: शिकायत मोड में प्रोफ़ाइल्स **नीति का पालन नहीं करेंगे** बल्कि बजाय इसके **नीति उल्लंघन** प्रयासों की **रिपोर्टिंग** करेंगे।

AppArmor लिनक्स पर कुछ अन्य MAC सिस्टमों से अलग होता है: यह **पाथ-आधारित** है, यह निष्पादन और शिकायत मोड प्रोफ़ाइलों को मिश्रित करने की अनुमति देता है, यह विकास को सुगम बनाने के लिए इंकलूड फ़ाइल का उपयोग करता है, और इसका उपयोग करने के लिए अन्य लोकप्रिय MAC सिस्टमों की तुलना में बहुत कम बाधा होती है।

## AppArmor के भाग

* **कर्नल मॉड्यूल**: वास्तविक कार्य करता है
* **नीतियाँ**: व्यवहार और सीमा की परिभाषा करती हैं
* **पार्सर**: नीतियों को कर्नल में लोड करता है
* **उपयोगकर्ता उपकरण**: apparmor के साथ संवाद करने के लिए यूजरमोड प्रोग्राम

## प्रोफ़ाइल पथ

Apparmor प्रोफ़ाइल आमतौर पर _**/etc/apparmor.d/**_ में सहेजे जाते हैं\
`sudo aa-status` के साथ आप उन बाइनरीज़ की सूची देख सकेंगे जिन्हें किसी प्रोफ़ाइल द्वारा प्रतिबंधित किया जाता है। यदि आप प्रतिबंधित किसी भी बाइनरी के पथ के हर एक चार "/" को बदलकर एक डॉट में बदल सकते हैं तो आप में उल्लिखित फ़ोल्डर में apparmor प्रोफ़ाइल का नाम प्राप्त करेंगे।

उदाहरण के लिए, _/usr/bin/man_ के लिए एक **apparmor** प्रोफ़ाइल _/etc/apparmor.d/usr.bin.man_ में स्थित होगा

## कमांड्स
```bash
aa-status     #check the current status
aa-enforce    #set profile to enforce mode (from disable or complain)
aa-complain   #set profile to complain mode (from diable or enforcement)
apparmor_parser #to load/reload an altered policy
aa-genprof    #generate a new profile
aa-logprof    #used to change the policy when the binary/program is changed
aa-mergeprof  #used to merge the policies
```
# प्रोफ़ाइल बनाना

* प्रभावित एक्ज़ीक्यूटेबल को दर्शाने के लिए, **पूर्ण पथ और वाइल्डकार्ड** (फ़ाइल ग्लॉबिंग के लिए) की अनुमति है।
* **फ़ाइलों** पर बाइनरी के ऊपर एक्सेस को दर्शाने के लिए निम्नलिखित **एक्सेस नियंत्रण** का उपयोग किया जा सकता है:
* **r** (पढ़ना)
* **w** (लिखना)
* **m** (मेमोरी मैप के रूप में एक्सेक्यूटेबल)
* **k** (फ़ाइल लॉकिंग)
* **l** (हार्ड लिंक बनाना)
* **ix** (नई प्रोग्राम को नीति वारिस्त करते हुए एक और प्रोग्राम को एक्सेक्यूट करने के लिए)
* **Px** (एक और प्रोफ़ाइल के तहत एक्सेक्यूट करें, पर्यावरण को साफ़ करने के बाद)
* **Cx** (एक बाल प्रोफ़ाइल के तहत एक्सेक्यूट करें, पर्यावरण को साफ़ करने के बाद)
* **Ux** (एक्सेक्यूट करें अनियंत्रित, पर्यावरण को साफ़ करने के बाद)
* **प्रोफ़ाइल में चर निर्धारित किए जा सकते हैं और प्रोफ़ाइल के बाहर से प्रबंधित किए जा सकते हैं। उदाहरण के लिए: @{PROC} और @{HOME} (प्रोफ़ाइल फ़ाइल में #include \<tunables/global> जोड़ें)
* **अनुमति नियमों को अधिरोहित करने के लिए निषेध नियम समर्थित हैं**।

## aa-genprof

प्रोफ़ाइल बनाना शुरू करने के लिए आपकी मदद करने के लिए एपार्मोर का उपयोग किया जा सकता है। यह संभव है कि **एपार्मोर बाइनरी द्वारा किए गए कार्रवाईयों की जांच करेगा और फिर आपको निर्धारित करने देगा कि आप कौन सी कार्रवाईयों को अनुमति देना चाहते हैं और कौन सी कार्रवाईयों को निषेध करना चाहते हैं**।\
आपको बस यह चलाना होगा:
```bash
sudo aa-genprof /path/to/binary
```
तो, एक अलग कंसोल में वे सभी कार्रवाइयाँ करें जो बाइनरी सामान्यतः करेगा:
```bash
/path/to/binary -a dosomething
```
तो, पहले कंसोल में "**s**" दबाएं और फिर रिकॉर्ड की गई कार्रवाइयों में इंगोर करना, अनुमति देना, या कुछ और करने के लिए दर्ज करें। जब आप समाप्त हो जाएं तो "**f**" दबाएं और नया प्रोफ़ाइल _/etc/apparmor.d/path.to.binary_ में बनाया जाएगा।

{% hint style="info" %}
एरो कीज़ का उपयोग करके आप जो कुछ चाहें अनुमति/अस्वीकार/या कुछ और करना चाहते हैं, उसे चुन सकते हैं।
{% endhint %}

## aa-easyprof

आप एक बाइनरी के एक एपारमोर प्रोफ़ाइल का टेम्पलेट भी बना सकते हैं:
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
ध्यान दें कि एक नई प्रोफ़ाइल में डिफ़ॉल्ट रूप से कुछ भी अनुमति नहीं होती है, इसलिए सब कुछ निषेधित होता है। आपको `/etc/passwd r,` जैसी पंक्तियों को जोड़ने की आवश्यकता होगी, जिससे उदाहरण के लिए `/etc/passwd` बाइनरी को पढ़ने की अनुमति मिले।
{% endhint %}

फिर आप नई प्रोफ़ाइल को **प्रबल** कर सकते हैं
```bash
sudo apparmor_parser -a /etc/apparmor.d/path.to.binary
```
## लॉग्स से प्रोफ़ाइल को संशोधित करना

निम्नलिखित टूल लॉग्स को पढ़ेगा और उपयोगकर्ता से पूछेगा कि क्या वह कुछ प्रतिबंधित क्रियाओं की अनुमति देना चाहता है:
```bash
sudo aa-logprof
```
{% hint style="info" %}
एरो कीज़ का उपयोग करके आप चुन सकते हैं कि आप क्या अनुमति देना / निषेध करना / कुछ भी करना चाहते हैं
{% endhint %}

## प्रोफ़ाइल प्रबंधन
```bash
#Main profile management commands
apparmor_parser -a /etc/apparmor.d/profile.name #Load a new profile in enforce mode
apparmor_parser -C /etc/apparmor.d/profile.name #Load a new profile in complain mode
apparmor_parser -r /etc/apparmor.d/profile.name #Replace existing profile
apparmor_parser -R /etc/apparmor.d/profile.name #Remove profile
```
# लॉग

**`service_bin`** के _/var/log/audit/audit.log_ से **AUDIT** और **DENIED** लॉग का उदाहरण:
```bash
type=AVC msg=audit(1610061880.392:286): apparmor="AUDIT" operation="getattr" profile="/bin/rcat" name="/dev/pts/1" pid=954 comm="service_bin" requested_mask="r" fsuid=1000 ouid=1000
type=AVC msg=audit(1610061880.392:287): apparmor="DENIED" operation="open" profile="/bin/rcat" name="/etc/hosts" pid=954 comm="service_bin" requested_mask="r" denied_mask="r" fsuid=1000 ouid=0
```
आप इस जानकारी को निम्नलिखित तरीके से प्राप्त कर सकते हैं:
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
# डॉकर में Apparmor

ध्यान दें कि डॉकर का प्रोफ़ाइल **डॉकर-प्रोफ़ाइल** डिफ़ॉल्ट रूप से लोड होता है:
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
डिफ़ॉल्ट रूप से **Apparmor डॉकर-डिफ़ॉल्ट प्रोफ़ाइल** [https://github.com/moby/moby/tree/master/profiles/apparmor](https://github.com/moby/moby/tree/master/profiles/apparmor) से उत्पन्न होता है।

**डॉकर-डिफ़ॉल्ट प्रोफ़ाइल सारांश**:

* सभी **नेटवर्किंग** तक **पहुंच**
* कोई **क्षमता** परिभाषित नहीं है (हालांकि, कुछ क्षमताएं मूल बेस नियमों को सम्मिलित करने से आएगी, जैसे #include \<abstractions/base>)
* किसी भी **/proc** फ़ाइल में **लिखने** की अनुमति नहीं है
* अन्य **उपनिर्देशिकाएं**/**फ़ाइलें** /**proc** और /**sys** की पढ़ने/लिखने/लॉक/लिंक/एक्सीक्यूट उपयोग की अनुमति नहीं है
* **माउंट** की अनुमति नहीं है
* **Ptrace** केवल उस प्रक्रिया पर चलाया जा सकता है जो **एक ही apparmor प्रोफ़ाइल** द्वारा सीमित हो

एक बार जब आप **डॉकर कंटेनर चलाते हैं**, तो आपको निम्नलिखित आउटपुट दिखाई देना चाहिए:
```bash
1 processes are in enforce mode.
docker-default (825)
```
नोट करें कि **एपार्मर डिफ़ॉल्ट में कंटेनर को प्राप्त किए गए क्षमता अधिकारों को भी अवरुद्ध कर देगा**। उदाहरण के लिए, यदि SYS_ADMIN क्षमता प्राप्त की जाती है, तो यह अनुमति देने के लिए डॉकर एपार्मर प्रोफ़ाइल इस पहुंच को अस्वीकार करेगा:
```bash
docker run -it --cap-add SYS_ADMIN --security-opt seccomp=unconfined ubuntu /bin/bash
echo "" > /proc/stat
sh: 1: cannot create /proc/stat: Permission denied
```
आपको apparmor को अक्षम करने की आवश्यकता है ताकि इसकी प्रतिबंधों को अनदेखा किया जा सके:
```bash
docker run -it --cap-add SYS_ADMIN --security-opt seccomp=unconfined --security-opt apparmor=unconfined ubuntu /bin/bash
```
नोट करें कि डिफ़ॉल्ट रूप से **AppArmor** भी **कंटेनर को माउंट करने से रोकेगा** भी अंदर से, यहां तक कि SYS_ADMIN क्षमता के साथ भी।

नोट करें कि आप डॉकर कंटेनर में **क्षमताएं जोड़ सकते/हटा सकते** हैं (यह अभी भी **AppArmor** और **Seccomp** जैसे सुरक्षा उपायों द्वारा प्रतिबंधित रहेगा):

* `--cap-add=SYS_ADMIN`_ _`SYS_ADMIN` क्षमता दें
* `--cap-add=ALL`_ _सभी क्षमताएं दें
* `--cap-drop=ALL --cap-add=SYS_PTRACE` सभी क्षमताएं हटाएं और केवल `SYS_PTRACE` दें

{% hint style="info" %}
आमतौर पर, जब आपको पता चलता है कि आपके पास एक **विशेषाधिकारित क्षमता** उपलब्ध है **डॉकर** कंटेनर **के अंदर** लेकिन कुछ हिस्सा **उत्पीड़न काम नहीं कर रहा है**, तो यह इसलिए होगा कि डॉकर **apparmor इसे रोक रहा होगा**।
{% endhint %}

## AppArmor Docker ब्रेकआउट

आप यह जान सकते हैं कि कौन सा **apparmor प्रोफ़ाइल कंटेनर चला रहा है** इस्तेमाल करके:
```bash
docker inspect 9d622d73a614 | grep lowpriv
"AppArmorProfile": "lowpriv",
"apparmor=lowpriv"
```
तब, आप निम्नलिखित पंक्ति को चला सकते हैं ताकि **उपयोग में आ रहे प्रोफ़ाइल को खोजें**:
```bash
find /etc/apparmor.d/ -name "*lowpriv*" -maxdepth 1 2>/dev/null
```
अजीब मामले में आप **एपार्मर डॉकर प्रोफ़ाइल को संशोधित करके और इसे रीलोड करके** उसे बदल सकते हैं। आप प्रतिबंधों को हटा सकते हैं और उन्हें "बाइपास" कर सकते हैं।


<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **हैकट्रिक्स में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)

- प्राप्त करें [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके साझा करें।**

</details>
