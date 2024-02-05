# लिनक्स फोरेंसिक्स

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उपयोग करें और आसानी से **समुदाय के सबसे उन्नत उपकरणों** द्वारा संचालित **कार्यप्रवाह** बनाएं और स्वचालित करें।\
आज ही पहुंचें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><strong>शून्य से हीरो तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी कंपनी का **विज्ञापन HackTricks में** देखना चाहते हैं या **HackTricks को PDF में डाउनलोड** करना चाहते हैं, तो [**सदस्यता योजनाएं देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) और हमें **ट्विटर** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके।

</details>

## प्रारंभिक जानकारी एकत्र करना

### मौलिक जानकारी

सबसे पहले, सुझाव दिया जाता है कि कुछ **USB** के साथ होना चाहिए जिसमें **अच्छे जाने माने बाइनरी और लाइब्रेरी** हों (आप बस उबंटू ले सकते हैं और फोल्डर _/bin_, _/sbin_, _/lib,_ और _/lib64_ कॉपी कर सकते हैं), फिर USB को माउंट करें, और एनवायरनमेंट वेरिएबल्स को उन बाइनरीज का उपयोग करने के लिए संशोधित करें:
```bash
export PATH=/mnt/usb/bin:/mnt/usb/sbin
export LD_LIBRARY_PATH=/mnt/usb/lib:/mnt/usb/lib64
```
जब आपने सिस्टम को अच्छे और जाने-माने binaries का उपयोग करने के लिए कॉन्फ़िगर कर लिया है, तो आप **कुछ मौलिक जानकारी निकालना** शुरू कर सकते हैं:
```bash
date #Date and time (Clock may be skewed, Might be at a different timezone)
uname -a #OS info
ifconfig -a || ip a #Network interfaces (promiscuous mode?)
ps -ef #Running processes
netstat -anp #Proccess and ports
lsof -V #Open files
netstat -rn; route #Routing table
df; mount #Free space and mounted devices
free #Meam and swap space
w #Who is connected
last -Faiwx #Logins
lsmod #What is loaded
cat /etc/passwd #Unexpected data?
cat /etc/shadow #Unexpected data?
find /directory -type f -mtime -1 -print #Find modified files during the last minute in the directory
```
#### संदेहजनक जानकारी

मूल जानकारी प्राप्त करते समय आपको ऐसी अजीब चीजों की जांच करनी चाहिए जैसे:

* **रूट प्रक्रियाएँ** आम तौर पर कम PIDS के साथ चलती हैं, इसलिए अगर आप एक बड़ी PID के साथ एक रूट प्रक्रिया पाते हैं तो आप संदेह कर सकते हैं
* `/etc/passwd` में शैली बिना उपयोगकर्ताओं की **रजिस्टर्ड लॉगिन** की जांच करें
* `/etc/shadow` में **पासवर्ड हैश** की जांच करें जिन उपयोगकर्ताओं के पास एक शैल नहीं है

### मेमोरी डंप

चल रहे सिस्टम की मेमोरी प्राप्त करने के लिए, [**LiME**](https://github.com/504ensicsLabs/LiME) का उपयोग करना सुझावित है।\
इसे **कंपाइल** करने के लिए, आपको पीडीआई का उपयोग करना होगा जो पीडीआई यंत्र का उपयोग कर रहा है।

{% hint style="info" %}
ध्यान रखें कि आप **LiME या किसी अन्य चीज़ को** पीडीआई यंत्र में स्थापित नहीं कर सकते क्योंकि यह कई परिवर्तन कर देगा
{% endhint %}

तो, यदि आपके पास एक अभिन्न संस्करण का यूबंटू है तो आप `apt-get install lime-forensics-dkms` का उपयोग कर सकते हैं\
अन्य मामलों में, आपको [**LiME**](https://github.com/504ensicsLabs/LiME) को github से डाउनलोड करना होगा और सही कर्नेल हेडर्स के साथ इसे कंपाइल करना होगा। पीडीआई यंत्र के सटीक कर्नेल हेडर्स प्राप्त करने के लिए, आप बस अपने यंत्र में निर्देशिका `/lib/modules/<कर्नेल संस्करण>` की **नकल** कर सकते हैं, और फिर उन्हें उपयोग करके LiME को **कंपाइल** कर सकते हैं:
```bash
make -C /lib/modules/<kernel version>/build M=$PWD
sudo insmod lime.ko "path=/home/sansforensics/Desktop/mem_dump.bin format=lime"
```
LiME समर्थित करता है 3 **फॉर्मेट**:

* Raw (प्रत्येक सेगमेंट को एक साथ जोड़ा गया)
* Padded (रॉ के समान, लेकिन दाएं बिट्स में शून्य हैं)
* Lime (मेटाडेटा के साथ अनुशंसित फॉर्मेट)

LiME का उपयोग **नेटवर्क के माध्यम से डंप भेजने** के लिए भी किया जा सकता है इसे सिस्टम पर स्टोर करने की बजाय कुछ इस प्रकार का उपयोग करके: `path=tcp:4444`

### डिस्क इमेजिंग

#### बंद करना

सबसे पहले, आपको **सिस्टम को बंद करने** की आवश्यकता होगी। यह हमेशा एक विकल्प नहीं है क्योंकि कुछ समय सिस्टम एक उत्पादन सर्वर होगा जिसे कंपनी को बंद करने की सामर्थ्य नहीं होगी।\
सिस्टम को बंद करने के **2 तरीके** हैं, एक **सामान्य बंद करना** और एक **"प्लग निकालना" बंद करना**। पहला वाला **प्रक्रियाएँ सामान्य रूप से समाप्त होने** और **फ़ाइल सिस्टम** को **समकालीन** होने देगा, लेकिन यह भी संभावित **मैलवेयर** को **सबूत नष्ट** करने की अनुमति देगा। "प्लग निकालना" दृष्टिकोण **कुछ जानकारी का हानि** ले सकता है (ज्यादातर जानकारी नष्ट नहीं होने वाली है क्योंकि हमने पहले ही मेमोरी का छवि ले ली है) और **मैलवेयर को कुछ करने का कोई मौका नहीं** होगा। इसलिए, यदि आपको लगता है कि वहाँ कोई **मैलवेयर** हो सकता है, तो सिस्टम पर **`sync`** **कमांड** को निष्पादित करें और प्लग निकालें।

#### डिस्क की छवि लेना

यह महत्वपूर्ण है कि **मामले से संबंधित किसी भी चीज़ को अपने कंप्यूटर से कनेक्ट करने से पहले**, आपको यह सुनिश्चित करना होगा कि यह **केवल पढ़ने के लिए माउंट** किया जाएगा ताकि कोई भी जानकारी को संशोधित न करें।
```bash
#Create a raw copy of the disk
dd if=<subject device> of=<image file> bs=512

#Raw copy with hashes along the way (more secure as it checks hashes while it's copying the data)
dcfldd if=<subject device> of=<image file> bs=512 hash=<algorithm> hashwindow=<chunk size> hashlog=<hash file>
dcfldd if=/dev/sdc of=/media/usb/pc.image hash=sha256 hashwindow=1M hashlog=/media/usb/pc.hashes
```
### डिस्क इमेज पूर्व विश्लेषण

कोई अधिक डेटा के साथ डिस्क इमेजिंग।
```bash
#Find out if it's a disk image using "file" command
file disk.img
disk.img: Linux rev 1.0 ext4 filesystem data, UUID=59e7a736-9c90-4fab-ae35-1d6a28e5de27 (extents) (64bit) (large files) (huge files)

#Check which type of disk image it's
img_stat -t evidence.img
raw
#You can list supported types with
img_stat -i list
Supported image format types:
raw (Single or split raw file (dd))
aff (Advanced Forensic Format)
afd (AFF Multiple File)
afm (AFF with external metadata)
afflib (All AFFLIB image formats (including beta ones))
ewf (Expert Witness Format (EnCase))

#Data of the image
fsstat -i raw -f ext4 disk.img
FILE SYSTEM INFORMATION
--------------------------------------------
File System Type: Ext4
Volume Name:
Volume ID: 162850f203fd75afab4f1e4736a7e776

Last Written at: 2020-02-06 06:22:48 (UTC)
Last Checked at: 2020-02-06 06:15:09 (UTC)

Last Mounted at: 2020-02-06 06:15:18 (UTC)
Unmounted properly
Last mounted on: /mnt/disk0

Source OS: Linux
[...]

#ls inside the image
fls -i raw -f ext4 disk.img
d/d 11: lost+found
d/d 12: Documents
d/d 8193:       folder1
d/d 8194:       folder2
V/V 65537:      $OrphanFiles

#ls inside folder
fls -i raw -f ext4 disk.img 12
r/r 16: secret.txt

#cat file inside image
icat -i raw -f ext4 disk.img 16
ThisisTheMasterSecret
```
<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks) का उपयोग करें और **ऑटोमेट वर्कफ़्लो** बनाएं जो दुनिया के **सबसे उन्नत** समुदाय उपकरणों द्वारा संचालित हो।\
आज ही पहुंच प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## ज्ञात मैलवेयर की खोज

### संशोधित सिस्टम फ़ाइलें

कुछ लिनक्स सिस्टमों में बहुत सारे स्थापित घटकों की अखंडता की पुष्टि करने की विशेषता होती है, जो असामान्य या अनुचित फ़ाइलों की पहचान के लिए एक प्रभावी तरीका प्रदान करती है। उदाहरण के लिए, लिनक्स पर `rpm -Va` का उपयोग रेडहैट पैकेज प्रबंधक का उपयोग करके स्थापित सभी पैकेज की पुष्टि करने के लिए डिज़ाइन किया गया है।
```bash
#RedHat
rpm -Va
#Debian
dpkg --verify
debsums | grep -v "OK$" #apt-get install debsums
```
### मैलवेयर/रूटकिट डिटेक्टर्स

निम्नलिखित पृष्ठ को पढ़ें ताकि आप मैलवेयर को खोजने के लिए उपयोगी टूल्स के बारे में जान सकें:

{% content-ref url="malware-analysis.md" %}
[malware-analysis.md](malware-analysis.md)
{% endcontent-ref %}

## स्थापित कार्यक्रमों की खोज

### पैकेज प्रबंधक

डेबियन-आधारित सिस्टमों पर, _**/var/ lib/dpkg/status**_ फ़ाइल में स्थापित पैकेज के विवरण होते हैं और _**/var/log/dpkg.log**_ फ़ाइल में जानकारी रिकॉर्ड होती है जब एक पैकेज स्थापित होता है।\
रेडहैट और संबंधित लिनक्स वितरणों पर **`rpm -qa --root=/ mntpath/var/lib/rpm`** कमांड एक सिस्टम पर एक आरपीएम डेटाबेस की सामग्री की सूची देगी।
```bash
#Debian
cat /var/lib/dpkg/status | grep -E "Package:|Status:"
cat /var/log/dpkg.log | grep installed
#RedHat
rpm -qa --root=/ mntpath/var/lib/rpm
```
### अन्य

**उपर्युक्त कमांड्स द्वारा सभी स्थापित कार्यक्रमों की सूची नहीं होगी** क्योंकि कुछ एप्लिकेशन कुछ विशिष्ट सिस्टमों के लिए पैकेज के रूप में उपलब्ध नहीं होते हैं और स्रोत से स्थापित करना पड़ता है। इसलिए, _**/usr/local**_ और _**/opt**_ जैसी स्थानों की समीक्षा अन्य एप्लिकेशनों को प्रकट कर सकती है जो स्रोत कोड से संकलित और स्थापित किए गए हों।
```bash
ls /opt /usr/local
```
एक और अच्छा विचार है **$PATH** के अंदर **सामान्य फोल्डर** की **जाँच करना** जो **इंस्टॉल किए गए पैकेज** से **संबंधित नहीं** हैं:
```bash
#Both lines are going to print the executables in /sbin non related to installed packages
#Debian
find /sbin/ -exec dpkg -S {} \; | grep "no path found"
#RedHat
find /sbin/ –exec rpm -qf {} \; | grep "is not"
```
<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उपयोग करें और आसानी से **ऑटोमेट वर्कफ़्लो** बनाएं जो दुनिया के **सबसे उन्नत** समुदाय उपकरणों द्वारा संचालित हो।\
आज ही पहुंच प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## हटाए गए चल रहे बाइनरी को पुनः प्राप्त करें

![](<../../.gitbook/assets/image (641).png>)

## ऑटोस्टार्ट स्थानों की जांच करें

### निर्धारित कार्याएँ
```bash
cat /var/spool/cron/crontabs/*  \
/var/spool/cron/atjobs \
/var/spool/anacron \
/etc/cron* \
/etc/at* \
/etc/anacrontab \
/etc/incron.d/* \
/var/spool/incron/* \

#MacOS
ls -l /usr/lib/cron/tabs/ /Library/LaunchAgents/ /Library/LaunchDaemons/ ~/Library/LaunchAgents/
```
### सेवाएं

मैलवेयर के लिए एक नए, अनधिकृत सेवा के रूप में अपने आप को अंकुरित करना बहुत सामान्य है। लिनक्स में कई स्क्रिप्ट हैं जो कंप्यूटर बूट होते समय सेवाएं शुरू करने के लिए उपयोग किए जाते हैं। प्रारंभी स्टार्टअप स्क्रिप्ट _**/etc/inittab**_ अन्य स्क्रिप्टों को बुलाता है जैसे rc.sysinit और _**/etc/rc.d/**_ निर्देशिका के तहत विभिन्न स्टार्टअप स्क्रिप्ट, या कुछ पुराने संस्करणों में _**/etc/rc.boot/**_। अन्य लिनक्स संस्करणों पर, जैसे डेबियन, स्टार्टअप स्क्रिप्ट _**/etc/init.d/**_ निर्देशिका में संग्रहीत होते हैं। साथ ही, कुछ सामान्य सेवाएं _**/etc/inetd.conf**_ या _**/etc/xinetd/**_ में सक्षम की जाती हैं लिनक्स के संस्करण के आधार पर। डिजिटल जांचकर्ताओं को इन स्टार्टअप स्क्रिप्ट की हर अनूठी प्रविष्टियों की जांच करनी चाहिए।

* _**/etc/inittab**_
* _**/etc/rc.d/**_
* _**/etc/rc.boot/**_
* _**/etc/init.d/**_
* _**/etc/inetd.conf**_
* _**/etc/xinetd/**_
* _**/etc/systemd/system**_
* _**/etc/systemd/system/multi-user.target.wants/**_

### कर्नेल मॉड्यूल

लिनक्स सिस्टम पर, कर्नेल मॉड्यूल मैलवेयर पैकेज के लिए रूटकिट घटक के रूप में आमतौर पर उपयोग किए जाते हैं। कर्नेल मॉड्यूल सिस्टम बूट होने पर `/lib/modules/'uname -r'` और `/etc/modprobe.d` निर्देशिकाओं में कॉन्फ़िगरेशन सूचना के आधार पर लोड होते हैं, और `/etc/modprobe` या `/etc/modprobe.conf` फ़ाइल। इन क्षेत्रों की जांच करनी चाहिए जो मैलवेयर से संबंधित हैं।

### अन्य ऑटोस्टार्ट स्थान

लिनक्स कई विन्यास फ़ाइलों का उपयोग करता है जो स्वचालित रूप से एक कार्यक्षमता को लॉग इन करने पर लॉन्च करने के लिए हो सकती हैं जो मैलवेयर के अनुकरण को शामिल कर सकती हैं।

* _**/etc/profile.d/\***_ , _**/etc/profile**_ , _**/etc/bash.bashrc**_ जब कोई भी उपयोगकर्ता खाता लॉग इन करता है तो इन्हें निष्पादित किया जाता है।
* _**∼/.bashrc**_ , _**∼/.bash\_profile**_ , _**\~/.profile**_ , _**∼/.config/autostart**_ जब विशिष्ट उपयोगकर्ता लॉग इन करता है तो इन्हें निष्पादित किया जाता है।
* _**/etc/rc.local**_ इसे पारंपरिक रूप से उन सभी साधारण सिस्टम सेवाओं के बाद निष्पादित किया जाता है, प्रोसेस के अंत में एक मल्टीयूजर रनलेवल में स्विच करने की प्रक्रिया के बाद।

## लॉग की जांच

कंप्रोमाइज़ सिस्टम पर उपलब्ध सभी लॉग फ़ाइलों में दुर्भाग्यपूर्ण निष्पादन और संबंधित गतिविधियों के लिए खोजें जैसे कि एक नई सेवा का निर्माण।

### शुद्ध लॉग

सिस्टम और सुरक्षा लॉग में दर्ज किए गए **लॉगिन** घटनाएं, जिसमें नेटवर्क के माध्यम से लॉगिन, एक निश्चित समय पर किसी खाते के माध्यम से **मैलवेयर** या एक **अतिक्रमणकारी ने पहुंच** प्राप्त किया हो सकता है। एक मैलवेयर संक्रमण के समय के आसपास अन्य घटनाएं सिस्टम लॉग में कैप्चर की जा सकती हैं, जैसे कि एक **नई** **सेवा** का **निर्माण** या घटना के समय नए खातों का निर्माण।\
रोचक सिस्टम लॉगिन:

* **/var/log/syslog** (debian) या **/var/log/messages** (Redhat)
* सिस्टम के बारे में सामान्य संदेश और जानकारी दिखाता है। यह वैश्विक सिस्टम के द्वारा किए गए सभी गतिविधियों का डेटा लॉग है।
* **/var/log/auth.log** (debian) या **/var/log/secure** (Redhat)
* सफल या असफल लॉगिन, और प्रमाणीकरण प्रक्रियाओं के लिए प्रमाणीकरण लॉग रखता है। संग्रहण सिस्टम प्रकार पर निर्भर करता है।
* `cat /var/log/auth.log | grep -iE "session opened for|accepted password|new session|not in sudoers"`
* **/var/log/boot.log**: स्टार्टअप संदेश और बूट जानकारी।
* **/var/log/maillog** या **var/log/mail.log:** मेल सर्वर लॉग के लिए है, पोस्टफिक्स, smtpd, या आपके सर्वर पर चल रही ईमेल संबंधित सेवाओं की जानकारी के लिए उपयुक्त है।
* **/var/log/kern.log**: कर्नेल लॉग और चेतावनी जानकारी रखता है। कर्नेल गतिविधि लॉग (जैसे, dmesg, kern.log, klog) दिखा सकते हैं कि किसी विशेष सेवा का बार-बार क्रैश हो रहा है, जिससे संकेत मिल सकता है कि एक अस्थिर ट्रोजनाइज़ वर्जन स्थापित किया गया है।
* **/var/log/dmesg**: डिवाइस ड्राइवर संदेशों के लिए एक भंडारण स्थान। इस फ़ाइल में संदेश देखने के लिए **dmesg** का उपयोग करें।
* **/var/log/faillog:** असफल लॉगिन के बारे में जानकारी रिकॉर्ड करता है। इसलिए, लॉगिन क्रेडेंशियल हैक्स और ब्रूट-फोर्स हमलों जैसे संभावित सुरक्षा उल्लंघनों की जांच के लिए उपयोगी है।
* **/var/log/cron**: Crond संबंधित संदेशों (क्रॉन जॉब्स) का रिकॉर्ड रखता है। जैसे जब क्रॉन डेमन ने एक नौकरी शुरू की।
* **/var/log/daemon.log:** चल रहे पृष्ठ सेवाओं का ट्रैक रखता है लेकिन उन्हें ग्राफिकल रूप में प्रस्तुत नहीं करता है।
* **/var/log/btmp**: सभी असफल लॉगिन प्रयासों का एक नोट रखता है।
* **/var/log/httpd/**: Apache httpd डेमन के error\_log और access\_log फ़ाइलों को रखने वाला एक निर्देशिका। httpd कोई भी त्रुटि जो आती है, उसे **error\_log** फ़ाइल में रखा जाता है। सिस्टम से संबंधित त्रुटियों की तरह। **access\_log** HTTP के माध्यम से आने वाले सभी अनुरोधों का लॉग रखता है।
* **/var/log/mysqld.log** या **/var/log/mysql.log**: MySQL लॉग फ़ाइल जो हर debug, विफलता और सफलता संदेश को रिकॉर्ड करती है, जिसमें MySQL डेमन mysqld की शुरुआत, बंद करना और पुनरारंभ करना शामिल है। सिस्टम निर्धारित करता है। RedHat, CentOS, Fedora, और अन्य RedHat-आधारित सिस्टम /var/log/mariadb/mariadb.log का उपयोग करते हैं। हालांकि, Debian/Ubuntu /var/log/mysql/error.log निर्देशिका का उपयोग करते हैं।
* **/var/log/xferlog**: FTP फ़ाइल स्थानांतरण सत्रों को रखता है। फ़ाइल नामों और उपयोगकर्ता प्रेरित FTP स्थानांतरण जैसी जानकारी शामिल है।
* **/var/log/\*** : इस निर्देशिका में अप्रत्याशित लॉग के लिए हमेशा जांच करना चाहिए

{% hint style="info"
```
pip3 install usbrip
usbrip ids download #Download USB ID database
```
### उदाहरण
```
usbrip events history #Get USB history of your curent linux machine
usbrip events history --pid 0002 --vid 0e0f --user kali #Search by pid OR vid OR user
#Search for vid and/or pid
usbrip ids download #Downlaod database
usbrip ids search --pid 0002 --vid 0e0f #Search for pid AND vid
```
अधिक उदाहरण और जानकारी गिटहब में: [https://github.com/snovvcrash/usbrip](https://github.com/snovvcrash/usbrip)

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उपयोग करें और **ऑटोमेट वर्कफ़्लो** बनाने के लिए दुनिया के **सबसे उन्नत** समुदाय उपकरणों द्वारा संचालित करें।\
आज ही पहुंचें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## उपयोगकर्ता खातों और लॉगऑन गतिविधियों की समीक्षा

अनूठे नामों या खातों की जांच करें जो अनुमतित घटनाओं के निकट बनाए गए या उपयोग किए गए हों।\
असामान्य नामों या खातों की जांच करें जो अनुमतित घटनाओं के निकट बनाए गए या उपयोग किए गए हों।\
सुदूर विशेषाधिकारों की जांच करें जो उपयोगकर्ताओं को दी गई हों।\
किसी भी खातों की खोज करें जिनमें **कोई पासवर्ड नहीं** है या **आसानी से अनुमान लगाया जा सकने वाला पासवर्ड** है।

## फ़ाइल सिस्टम की जांच

फ़ाइल सिस्टम डेटा संरचनाएँ एक **मैलवेयर** घटना से संबंधित बहुत सारी **जानकारी** प्रदान कर सकती हैं, जैसे **घटनाओं का समय** और **मैलवेयर** की वास्तविक **सामग्री**।\
**मैलवेयर** को **फ़ाइल सिस्टम विश्लेषण** को अवरोधित करने के लिए बढ़ाया जा रहा है। कुछ मैलवेयर दुर्भाग्यपूर्ण फ़ाइलों पर दिनांक-समय छापों को बदलते हैं ताकि उन्हें टाइमलाइन विश्लेषण के साथ खोजना कठिन हो। अन्य दुर्भाग्यपूर्ण कोड डेटा को केवल फ़ाइल सिस्टम में रखने के लिए डिज़ाइन किए गए हैं।\
इस प्रकार के एंटी-फोरेंसिक तकनीकों का सामना करने के लिए, फ़ाइल सिस्टम दिनांक-समय छापों के टाइमलाइन विश्लेषण और मैलवेयर के सामान्य स्थानों में संभावित फ़ाइलों की जांच करना आवश्यक है।

* **ऑटॉप्सी** का उपयोग करके आप संदेहास्पद गतिविधि खोजने के लिए घटनाओं का टाइमलाइन देख सकते हैं। आप **Sleuth Kit** से `mactime` विशेषता का सीधा उपयोग भी कर सकते हैं।
* **$PATH** में **अप्रत्याशित स्क्रिप्ट** की जांच करें (शायद कुछ sh या php स्क्रिप्ट?)
* `/dev` में फ़ाइलें विशेष फ़ाइलें थीं, आप यहाँ मैलवेयर से संबंधित गैर-विशेष फ़ाइलें पा सकते हैं।
* असामान्य या **छिपी हुई फ़ाइलें** और **निर्देशिकाएँ** खोजें, जैसे “.. ” (डॉट डॉट स्पेस) या “..^G ” (डॉट डॉट कंट्रोल-जी)
* सिस्टम पर /bin/bash की setuid प्रतियां `find / -user root -perm -04000 –print`
* बड़ी संख्या में फ़ाइलों के हटाए जाने के लिए हटाए गए **इनोड की दिनांक-समय छापों** की समीक्षा करें, जो एक ही समय पर होने वाली दुर्भाग्यपूर्ण गतिविधि जैसे रूटकिट या ट्रोजनाइज़ सेवा की स्थापना का संकेत दे सकती है।
* क्योंकि इनोड अगली उपलब्धता के आधार पर आवंटित किए जाते हैं, **सिस्टम पर डाली गई दुर्भाग्यपूर्ण फ़ाइलें लगभग एक ही समय पर स्थापित की जा सकती हैं**। इसलिए, जब मैलवेयर का एक घटक पाया जाता है, तो पड़ोसी इनोड की जांच करना उपयोगी हो सकता है।
* नए या संशोधित फ़ाइलों की **संशोधित और या बदली गई समय** की दिनांक-समय छापों की समीक्षा करें जैसे _/bin_ या _/sbin_ जैसे निर्देशिकाओं की।
* एक निर्देशिका की फ़ाइलों और फ़ोल्डर को देखना **निर्माण तिथि** के आधार पर वर्णित होता है बजाय वर्णानुक्रमिक रूप से देखना दिखाई देता है कि कौन सी फ़ाइलें या फ़ोल्डर अधिक हाल की हैं (अंतिम वाले सामान्यतः)।

आप `ls -laR --sort=time /bin` का उपयोग करके एक फ़ोल्डर की सबसे हाल की फ़ाइलें देख सकते हैं।\
आप `ls -lai /bin |sort -n` का उपयोग करके एक फ़ोल्डर में फ़ाइलों के इनोड देख सकते हैं।

{% hint style="info" %}
ध्यान दें कि एक **हमलावर** **फ़ाइलें दिखाने** के लिए **समय** को **संशोधित** कर सकता है, लेकिन वह **इनोड** को **संशोधित** नहीं कर सकता। यदि आपको लगता है कि एक **फ़ाइल** का **इनोड समय** के रूप में बनाया गया है और उसका **इनोड अप्रत्याशित रूप से बड़ा** है, तो उस **फ़ाइल के टाइमस्टैम्प संशोधित किए गए** हो सकते हैं।
{% endhint %}

## विभिन्न फ़ाइल सिस्टम संस्करणों की फ़ाइलों की तुलना करें

#### जोड़ी गई फ़ाइलें
```bash
git diff --no-index --diff-filter=A _openwrt1.extracted/squashfs-root/ _openwrt2.extracted/squashfs-root/
```
#### संशोधित सामग्री खोजें
```bash
git diff --no-index --diff-filter=M _openwrt1.extracted/squashfs-root/ _openwrt2.extracted/squashfs-root/ | grep -E "^\+" | grep -v "Installed-Time"
```
#### हटाए गए फ़ाइलें खोजें
```bash
git diff --no-index --diff-filter=A _openwrt1.extracted/squashfs-root/ _openwrt2.extracted/squashfs-root/
```
#### अन्य फ़िल्टर

**`-diff-filter=[(A|C|D|M|R|T|U|X|B)…​[*]]`**

केवल फ़ाइलों को चुनें जिन्हें जोड़ा गया (`A`), कॉपी किया गया (`C`), हटाया गया (`D`), संशोधित किया गया (`M`), नाम बदला गया (`R`), और उनके प्रकार (जैसे सामान्य फ़ाइल, सिंकलिंक, सबमॉड्यूल, …​) बदल गया (`T`), अमर्जित (`U`), अज्ञात (`X`), या उनके संयोजन टूट गया (`B`). फ़िल्टर वर्णों का कोई भी संयोजन (सहित किसी भी नहीं) उपयोग किया जा सकता है. जब `*` (सभी या कोई नहीं) संयोजन में जोड़ा जाता है, तो तुलना में किसी अन्य मानदंड को मिलने पर सभी पथ चयनित होते हैं; अगर कोई ऐसी फ़ाइल नहीं है जो अन्य मानदंड को मिलती है, तो कुछ भी चयनित नहीं होता है।

इसके अलावा, **ये ऊपरी मामले छोटे अक्षर में बदले जा सकते हैं**। उदाहरण के लिए, `--diff-filter=ad` जोड़े और हटाए गए पथों को निकालता है।

ध्यान दें कि सभी विभेदों में सभी प्रकार शामिल नहीं हो सकते। उदाहरण के लिए, सूची से काम के पेड़ से वर्किंग ट्री की तुलना में विभेद कभी भी जोड़े गए प्रविष्टियों को नहीं रख सकते (क्योंकि विभेद में शामिल पथों का सेट सूची में क्या है उससे सीमित होता है)। उसी तरह, कॉपी और नाम बदले गए प्रविष्टियाँ उस समय प्रकट नहीं हो सकतीं जब उन प्रकारों के लिए पहचान अक्षम हो।

## संदर्भ

* [https://cdn.ttgtmedia.com/rms/security/Malware%20Forensics%20Field%20Guide%20for%20Linux%20Systems\_Ch3.pdf](https://cdn.ttgtmedia.com/rms/security/Malware%20Forensics%20Field%20Guide%20for%20Linux%20Systems\_Ch3.pdf)
* [https://www.plesk.com/blog/featured/linux-logs-explained/](https://www.plesk.com/blog/featured/linux-logs-explained/)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं**? या क्या आप **PEASS के नवीनतम संस्करण का उपयोग करना चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) की जांच करें!

* अपने [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन की
* प्राप्त करें [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**

**अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **को**।

</details>

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उपयोग करें और आसानी से **वर्ल्ड के सबसे उन्नत समुदाय उपकरणों द्वारा संचालित** वर्कफ़्लो बनाएं और स्वचालित करें।\
आज ही पहुंचें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}
