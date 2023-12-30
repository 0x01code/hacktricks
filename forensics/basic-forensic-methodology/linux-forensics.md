# लिनक्स फोरेंसिक्स

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उपयोग करके आसानी से **वर्कफ्लोज़ को ऑटोमेट** करें जो दुनिया के **सबसे उन्नत** समुदाय टूल्स द्वारा संचालित होते हैं।\
आज ही एक्सेस प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert) के साथ शून्य से हीरो तक AWS हैकिंग सीखें!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) या [**telegram group**](https://t.me/peass) में **शामिल हों** या मुझे **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) पर **फॉलो करें**।
* **HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपोज़ में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>

## प्रारंभिक सूचना संग्रहण

### मूलभूत जानकारी

सबसे पहले, यह सिफारिश की जाती है कि आपके पास कुछ **USB** हो जिसमें **अच्छे ज्ञात बाइनरीज़ और लाइब्रेरीज़ हों** (आप बस उबंटू प्राप्त कर सकते हैं और फोल्डर्स _/bin_, _/sbin_, _/lib,_ और _/lib64_ की प्रतिलिपि बना सकते हैं), फिर USB को माउंट करें, और उन बाइनरीज़ का उपयोग करने के लिए पर्यावरण वेरिएबल्स में परिवर्तन करें:
```bash
export PATH=/mnt/usb/bin:/mnt/usb/sbin
export LD_LIBRARY_PATH=/mnt/usb/lib:/mnt/usb/lib64
```
एक बार जब आपने सिस्टम को अच्छे और ज्ञात बाइनरीज का उपयोग करने के लिए कॉन्फ़िगर कर लिया है, तो आप **कुछ मूलभूत जानकारी निकालना** शुरू कर सकते हैं:
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
#### संदिग्ध जानकारी

बुनियादी जानकारी प्राप्त करते समय आपको अजीब चीजों की जांच करनी चाहिए जैसे:

* **Root प्रक्रियाएं** आमतौर पर कम PIDS के साथ चलती हैं, इसलिए यदि आपको बड़े PID के साथ एक root प्रक्रिया मिलती है तो आपको संदेह हो सकता है
* `/etc/passwd` में बिना शेल के उपयोगकर्ताओं की **पंजीकृत लॉगिन** की जांच करें
* बिना शेल के उपयोगकर्ताओं के लिए `/etc/shadow` में **पासवर्ड हैशेज** की जांच करें

### मेमोरी डंप

चल रहे सिस्टम की मेमोरी प्राप्त करने के लिए, [**LiME**](https://github.com/504ensicsLabs/LiME) का उपयोग करना सुझावित है।\
इसे **कंपाइल** करने के लिए, आपको पीड़ित मशीन के **समान कर्नेल** का उपयोग करना होगा।

{% hint style="info" %}
याद रखें कि आप पीड़ित मशीन में LiME या कोई अन्य चीज **इंस्टॉल नहीं कर सकते** क्योंकि इससे उसमें कई बदलाव होंगे।
{% endhint %}

इसलिए, यदि आपके पास Ubuntu का समान संस्करण है तो आप `apt-get install lime-forensics-dkms` का उपयोग कर सकते हैं।\
अन्य मामलों में, आपको github से [**LiME**](https://github.com/504ensicsLabs/LiME) डाउनलोड करना होगा और सही कर्नेल हेडर्स के साथ इसे कंपाइल करना होगा। पीड़ित मशीन के **बिल्कुल सही कर्नेल हेडर्स** प्राप्त करने के लिए, आप बस **डायरेक्टरी को कॉपी कर सकते हैं** `/lib/modules/<kernel version>` अपनी मशीन में, और फिर उनका उपयोग करके LiME को **कंपाइल** करें:
```bash
make -C /lib/modules/<kernel version>/build M=$PWD
sudo insmod lime.ko "path=/home/sansforensics/Desktop/mem_dump.bin format=lime"
```
LiME 3 **प्रारूपों** का समर्थन करता है:

* Raw (प्रत्येक सेगमेंट एक साथ जोड़ा गया)
* Padded (रॉ की तरह, लेकिन दाईं ओर शून्य के साथ)
* Lime (मेटाडेटा के साथ अनुशंसित प्रारूप)

LiME का उपयोग **नेटवर्क के माध्यम से डंप भेजने** के लिए भी किया जा सकता है, सिस्टम पर संग्रहीत करने के बजाय इस तरह का उपयोग करके: `path=tcp:4444`

### डिस्क इमेजिंग

#### शट डाउन करना

सबसे पहले, आपको **सिस्टम को शट डाउन** करने की आवश्यकता होगी। यह हमेशा एक विकल्प नहीं होता है क्योंकि कभी-कभी सिस्टम एक प्रोडक्शन सर्वर होता है जिसे कंपनी शट डाउन करने का खर्च नहीं उठा सकती।\
सिस्टम को शट डाउन करने के **2 तरीके** हैं, एक **सामान्य शट डाउन** और एक **"प्लग खींचना" शट डाउन**। पहला तरीका **प्रक्रियाओं को सामान्य रूप से समाप्त** करने और **फाइलसिस्टम** को **सिंक्रनाइज़** करने की अनुमति देगा, लेकिन यह संभावित **मैलवेयर** को भी **सबूत नष्ट** करने की अनुमति देगा। "प्लग खींचना" दृष्टिकोण में कुछ **सूचना हानि** हो सकती है (ज्यादा सूचना खोने वाली नहीं है क्योंकि हमने पहले ही मेमोरी की एक इमेज ले ली है) और **मैलवेयर को कुछ भी करने का कोई अवसर नहीं** होगा। इसलिए, अगर आप **संदेह** करते हैं कि वहां **मैलवेयर** हो सकता है, तो सिस्टम पर **`sync`** **कमांड** निष्पादित करें और प्लग खींच लें।

#### डिस्क की इमेज लेना

यह ध्यान देना महत्वपूर्ण है कि **केस से संबंधित किसी भी चीज़ से अपने कंप्यूटर को जोड़ने से पहले**, आपको यह सुनिश्चित करना होगा कि यह **केवल पढ़ने के लिए माउंट** किया जाएगा ताकि किसी भी सूचना को संशोधित न किया जा सके।
```bash
#Create a raw copy of the disk
dd if=<subject device> of=<image file> bs=512

#Raw copy with hashes along the way (more secure as it checks hashes while it's copying the data)
dcfldd if=<subject device> of=<image file> bs=512 hash=<algorithm> hashwindow=<chunk size> hashlog=<hash file>
dcfldd if=/dev/sdc of=/media/usb/pc.image hash=sha256 hashwindow=1M hashlog=/media/usb/pc.hashes
```
### डिस्क इमेज प्री-विश्लेषण

डिस्क इमेज की इमेजिंग जिसमें कोई और डेटा नहीं होता।
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
<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उपयोग करके दुनिया के **सबसे उन्नत** समुदाय उपकरणों द्वारा संचालित **वर्कफ्लो को आसानी से बनाएं और स्वचालित करें**।\
आज ही पहुंच प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## ज्ञात Malware की खोज

### संशोधित सिस्टम फाइलें

कुछ Linux सिस्टम्स में कई स्थापित घटकों की **सत्यता की जांच करने की सुविधा** होती है, जो असामान्य या अनुचित स्थान पर मौजूद फाइलों की पहचान करने का प्रभावी तरीका प्रदान करती है। उदाहरण के लिए, Linux पर `rpm -Va` का उपयोग करके RedHat Package Manager के माध्यम से स्थापित सभी पैकेजों की सत्यता की जांच की जाती है।
```bash
#RedHat
rpm -Va
#Debian
dpkg --verify
debsums | grep -v "OK$" #apt-get install debsums
```
### मैलवेयर/रूटकिट डिटेक्टर्स

मैलवेयर खोजने में उपयोगी हो सकने वाले टूल्स के बारे में जानने के लिए निम्नलिखित पृष्ठ पढ़ें:

{% content-ref url="malware-analysis.md" %}
[malware-analysis.md](malware-analysis.md)
{% endcontent-ref %}

## स्थापित प्रोग्राम्स की खोज

### पैकेज मैनेजर

Debian-आधारित सिस्टम्स पर, _**/var/lib/dpkg/status**_ फाइल में स्थापित पैकेजों के बारे में विवरण होते हैं और _**/var/log/dpkg.log**_ फाइल में एक पैकेज स्थापित होने पर जानकारी दर्ज की जाती है।\
RedHat और संबंधित Linux वितरणों पर **`rpm -qa --root=/mntpath/var/lib/rpm`** कमांड एक सिस्टम पर RPM डेटाबेस की सामग्री की सूची देगा।
```bash
#Debian
cat /var/lib/dpkg/status | grep -E "Package:|Status:"
cat /var/log/dpkg.log | grep installed
#RedHat
rpm -qa --root=/ mntpath/var/lib/rpm
```
### अन्य

**उपरोक्त आदेशों द्वारा सभी स्थापित कार्यक्रम सूचीबद्ध नहीं किए जाएंगे** क्योंकि कुछ एप्लिकेशन कुछ सिस्टमों के लिए पैकेज के रूप में उपलब्ध नहीं होते हैं और इन्हें स्रोत से स्थापित करना पड़ता है। इसलिए, _**/usr/local**_ और _**/opt**_ जैसे स्थानों की समीक्षा से स्रोत कोड से संकलित और स्थापित किए गए अन्य एप्लिकेशन का पता चल सकता है।
```bash
ls /opt /usr/local
```
एक और अच्छा विचार है **सामान्य फोल्डर्स** की **जांच** करना **$PATH** के अंदर **इंस्टॉल किए गए पैकेजों से संबंधित नहीं** बाइनरीज के लिए:
```bash
#Both lines are going to print the executables in /sbin non related to installed packages
#Debian
find /sbin/ -exec dpkg -S {} \; | grep "no path found"
#RedHat
find /sbin/ –exec rpm -qf {} \; | grep "is not"
```
```markdown
<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks) का उपयोग करके आसानी से **वर्कफ्लोज़ को ऑटोमेट** करें जो दुनिया के **सबसे उन्नत** समुदाय टूल्स द्वारा संचालित होते हैं।
आज ही एक्सेस प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## हटाए गए चल रहे बाइनरीज को पुनः प्राप्त करें

![](<../../.gitbook/assets/image (641).png>)

## ऑटोस्टार्ट स्थानों का निरीक्षण करें

### निर्धारित कार्य
```
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

मैलवेयर द्वारा खुद को एक नई, अनधिकृत सेवा के रूप में स्थापित करना बेहद आम है। लिनक्स में कई स्क्रिप्ट्स होती हैं जो कंप्यूटर बूट होने पर सेवाओं को शुरू करने के लिए इस्तेमाल की जाती हैं। इनिशियलाइजेशन स्टार्टअप स्क्रिप्ट _**/etc/inittab**_ अन्य स्क्रिप्ट्स जैसे कि rc.sysinit और _**/etc/rc.d/**_ निर्देशिका के विभिन्न स्टार्टअप स्क्रिप्ट्स, या कुछ पुराने संस्करणों में _**/etc/rc.boot/**_ को कॉल करती है। लिनक्स के अन्य संस्करणों में, जैसे कि डेबियन, स्टार्टअप स्क्रिप्ट्स _**/etc/init.d/**_ निर्देशिका में संग्रहीत होती हैं। इसके अलावा, कुछ सामान्य सेवाएं _**/etc/inetd.conf**_ या _**/etc/xinetd/**_ में सक्षम की जाती हैं, जो लिनक्स के संस्करण पर निर्भर करती हैं। डिजिटल जांचकर्ताओं को प्रत्येक इन स्टार्टअप स्क्रिप्ट्स की जांच असामान्य प्रविष्टियों के लिए करनी चाहिए।

* _**/etc/inittab**_
* _**/etc/rc.d/**_
* _**/etc/rc.boot/**_
* _**/etc/init.d/**_
* _**/etc/inetd.conf**_
* _**/etc/xinetd/**_
* _**/etc/systemd/system**_
* _**/etc/systemd/system/multi-user.target.wants/**_

### कर्नेल मॉड्यूल

लिनक्स सिस्टम्स पर, कर्नेल मॉड्यूल्स आमतौर पर मैलवेयर पैकेजों के रूटकिट घटकों के रूप में इस्तेमाल किए जाते हैं। कर्नेल मॉड्यूल्स `/lib/modules/'uname -r'` और `/etc/modprobe.d` निर्देशिकाओं में कॉन्फ़िगरेशन जानकारी के आधार पर सिस्टम बूट होने पर लोड किए जाते हैं, और `/etc/modprobe` या `/etc/modprobe.conf` फाइल। इन क्षेत्रों की जांच मैलवेयर से संबंधित आइटमों के लिए की जानी चाहिए।

### अन्य ऑटोस्टार्ट स्थान

कई कॉन्फ़िगरेशन फाइलें हैं जिनका उपयोग लिनक्स करता है जब एक उपयोगकर्ता सिस्टम में लॉग इन करता है, जिसमें मैलवेयर के निशान हो सकते हैं।

* _**/etc/profile.d/\***_ , _**/etc/profile**_ , _**/etc/bash.bashrc**_ किसी भी उपयोगकर्ता खाते के लॉग इन होने पर निष्पादित किए जाते हैं।
* _**∼/.bashrc**_ , _**∼/.bash\_profile**_ , _**\~/.profile**_ , _**∼/.config/autostart**_ विशिष्ट उपयोगकर्ता के लॉग इन होने पर निष्पादित किए जाते हैं।
* _**/etc/rc.local**_ परंपरागत रूप से सभी सामान्य सिस्टम सेवाओं के शुरू होने के बाद, मल्टीयूजर रनलेवल में स्विच करने की प्रक्रिया के अंत में निष्पादित किया जाता है।

## लॉग्स की जांच करें

समझौता किए गए सिस्टम पर उपलब्ध सभी लॉग फाइलों में मैलिशस निष्पादन और संबंधित गतिविधियों जैसे कि एक नई सेवा के निर्माण के निशानों की तलाश करें।

### शुद्ध लॉग्स

**लॉगिन** घटनाएं जो सिस्टम और सुरक्षा लॉग्स में दर्ज की गई हैं, जिसमें नेटवर्क के माध्यम से लॉगिन शामिल हैं, यह प्रकट कर सकती हैं कि **मैलवेयर** या एक **घुसपैठिया** एक निश्चित समय पर एक दिए गए खाते के माध्यम से समझौता किए गए सिस्टम तक पहुंच गया। अन्य घटनाएं जो मैलवेयर संक्रमण के समय के आसपास हो सकती हैं, सिस्टम लॉग्स में कैप्चर की जा सकती हैं, जिसमें एक घटना के समय के आसपास एक **नई** **सेवा** या नए खातों का **निर्माण** शामिल है।\
रोचक सिस्टम लॉगिन्स:

* **/var/log/syslog** (डेबियन) या **/var/log/messages** (रेडहैट)
* सिस्टम के बारे में सामान्य संदेश और जानकारी दिखाता है। यह वैश्विक सिस्टम की सभी गतिविधियों का डेटा लॉग है।
* **/var/log/auth.log** (डेबियन) या **/var/log/secure** (रेडहैट)
* सफल या असफल लॉगिन्स के लिए प्रमाणीकरण लॉग्स रखते हैं, और प्रमाणीकरण प्रक्रियाओं का भंडारण करते हैं। सिस्टम के प्रकार पर निर्भर करता है।
* `cat /var/log/auth.log | grep -iE "session opened for|accepted password|new session|not in sudoers"`
* **/var/log/boot.log**: स्टार्ट-अप संदेश और बूट जानकारी।
* **/var/log/maillog** या **var/log/mail.log:** मेल सर्वर लॉग्स के लिए है, जो आपके सर्वर पर चल रही पोस्टफिक्स, smtpd, या ईमेल-संबंधित सेवाओं की जानकारी के लिए उपयोगी है।
* **/var/log/kern.log**: कर्नेल लॉग्स और चेतावनी जानकारी रखता है। कर्नेल गतिविधि लॉग्स (जैसे कि dmesg, kern.log, klog) यह दिखा सकते हैं कि एक विशेष सेवा बार-बार क्रैश हुई, जो संकेत दे सकती है कि एक अस्थिर ट्रोजनाइज्ड संस्करण स्थापित किया गया था।
* **/var/log/dmesg**: डिवाइस ड्राइवर संदेशों के लिए एक भंडार। **dmesg** का उपयोग करके इस फाइल में संदेश देखें।
* **/var/log/faillog:** असफल लॉगिन्स की जानकारी रिकॉर्ड करता है। इसलिए, लॉगिन क्रेडेंशियल हैक्स और ब्रूट-फोर्स हमलों जैसे संभावित सुरक्षा उल्लंघनों की जांच के लिए उपयोगी है।
* **/var/log/cron**: क्रॉन्ड-संबंधित संदेशों (क्रॉन जॉब्स) का रिकॉर्ड रखता है। जैसे जब क्रॉन डेमॉन ने एक जॉब शुरू किया।
* **/var/log/daemon.log:** चल रही पृष्ठभूमि सेवाओं का ट्रैक रखता है लेकिन उन्हें ग्राफिकली प्रस्तुत नहीं करता है।
* **/var/log/btmp**: सभी असफल लॉगिन प्रयासों का नोट रखता है।
* **/var/log/httpd/**: एक निर्देशिका जिसमें अपाचे httpd डेमॉन की error\_log और access\_log फाइलें होती हैं। हर गलती जो httpd का सामना करती है, **error\_log** फाइल में रखी जाती है। सिस्टम-संबंधित गलतियों और अन्य समस्याओं के बारे में सोचें। **access\_log** HTTP के माध्यम से आने वाले सभी अनुरोधों को लॉग करता है।
* **/var/log/mysqld.log** या **/var/log/mysql.log**: MySQL लॉग फाइल जो हर डीबग, विफलता और सफलता संदेश को रिकॉर्ड करती है, जिसमें MySQL डेमॉन mysqld के शुरू होने, बंद होने और पुनः शुरू होने शामिल हैं। सिस्टम निर्देशिका पर निर्णय लेता है। रेडहैट, सेंटोस, फेडोरा, और अन्य रेडहैट-आधारित सिस्टम /var/log/mariadb/mariadb.log का उपयोग करते हैं। हालांकि, डेबियन/उबुंटू /var/log/mysql/error.log निर्देशिका का उपयोग करते हैं।
* **/var/log/xferlog**: FTP
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
गिटहब पर और उदाहरण और जानकारी: [https://github.com/snovvcrash/usbrip](https://github.com/snovvcrash/usbrip)

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks) का उपयोग करके आसानी से और **ऑटोमेट वर्कफ्लोज** बनाएं जो दुनिया के **सबसे उन्नत** समुदाय टूल्स द्वारा संचालित होते हैं।\
आज ही एक्सेस प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## उपयोगकर्ता खातों और लॉगऑन गतिविधियों की समीक्षा करें

_**/etc/passwd**_, _**/etc/shadow**_ और **सुरक्षा लॉग्स** की जांच करें असामान्य नामों या खातों के लिए जो अनधिकृत घटनाओं के निकट समय में बनाए गए या इस्तेमाल किए गए हों। साथ ही, संभावित sudo ब्रूट-फोर्स हमलों की जांच करें।\
इसके अलावा, _**/etc/sudoers**_ और _**/etc/groups**_ जैसी फाइलों की जांच करें उपयोगकर्ताओं को दिए गए अप्रत्याशित विशेषाधिकारों के लिए।\
अंत में, **कोई पासवर्ड नहीं** या **आसानी से अनुमान लगाए गए** पासवर्ड वाले खातों की तलाश करें।

## फाइल सिस्टम की जांच करें

फाइल सिस्टम डेटा संरचनाएं **मालवेयर** घटना से संबंधित महत्वपूर्ण मात्रा में **जानकारी** प्रदान कर सकती हैं, जिसमें घटनाओं की **समय** और मालवेयर की वास्तविक **सामग्री** शामिल है।\
**मालवेयर** को फाइल सिस्टम विश्लेषण को **बाधित करने** के लिए बढ़ते हुए डिजाइन किया जा रहा है। कुछ मालवेयर समयरेखा विश्लेषण के साथ उन्हें खोजना कठिन बनाने के लिए दुर्भावनापूर्ण फाइलों पर दिनांक-समय स्टाम्प्स को बदल देते हैं। अन्य दुर्भावनापूर्ण कोड फाइल सिस्टम में संग्रहीत डेटा की मात्रा को कम करने के लिए केवल मेमोरी में कुछ जानकारी संग्रहीत करने के लिए डिजाइन किए गए हैं।\
ऐसी एंटी-फोरेंसिक तकनीकों से निपटने के लिए, फाइल सिस्टम दिनांक-समय स्टाम्प्स के **समयरेखा विश्लेषण** पर और मालवेयर पाए जा सकने वाले सामान्य स्थानों में संग्रहीत फाइलों पर **सावधानीपूर्वक ध्यान देना** आवश्यक है।

* **ऑटोप्सी** का उपयोग करके आप घटनाओं की समयरेखा देख सकते हैं जो संदिग्ध गतिविधि की खोज में उपयोगी हो सकती है। आप सीधे **Sleuth Kit** से `mactime` फीचर का भी उपयोग कर सकते हैं।
* **$PATH** के अंदर **अप्रत्याशित स्क्रिप्ट्स** की जांच करें (शायद कुछ sh या php स्क्रिप्ट्स?)
* `/dev` में फाइलें विशेष फाइलें होती थीं, आपको यहां मालवेयर से संबंधित गैर-विशेष फाइलें मिल सकती हैं।
* असामान्य या **छिपी हुई फाइलों** और **डायरेक्टरीज** की तलाश करें, जैसे कि “.. ” (डॉट डॉट स्पेस) या “..^G ” (डॉट डॉट कंट्रोल-जी)
* सिस्टम पर /bin/bash की Setuid प्रतियां `find / -user root -perm -04000 –print`
* बड़ी संख्या में फाइलों के नष्ट होने के समय के दिनांक-समय स्टाम्प्स की समीक्षा करें, जो रूटकिट या ट्रोजनाइज्ड सेवा की स्थापना जैसी दुर्भावनापूर्ण गतिविधि का संकेत दे सकती है।
* चूंकि inodes को अगले उपलब्ध आधार पर आवंटित किया जाता है, इसलिए लगभग एक ही समय में सिस्टम पर रखी गई **दुर्भावनापूर्ण फाइलों को लगातार inodes आवंटित किए जा सकते हैं**। इसलिए, मालवेयर के एक घटक का पता लगाने के बाद, पड़ोसी inodes की जांच करना उत्पादक हो सकता है।
* _/bin_ या _/sbin_ जैसी डायरेक्टरीज की भी जांच करें क्योंकि नई या संशोधित फाइलों का **संशोधित और या बदला हुआ समय** दिलचस्प हो सकता है।
* एक डायरेक्टरी की फाइलों और फोल्डरों को वर्णानुक्रम के बजाय **निर्माण तिथि द्वारा क्रमित** देखना दिलचस्प होता है ताकि देखा जा सके कि कौन सी फाइलें या फोल्डर हाल ही में हैं (आमतौर पर अंतिम वाले)।

आप एक फोल्डर की सबसे हाल की फाइलों की जांच `ls -laR --sort=time /bin` का उपयोग करके कर सकते हैं\
आप एक फोल्डर के अंदर फाइलों के inodes की जांच `ls -lai /bin |sort -n` का उपयोग करके कर सकते हैं

{% hint style="info" %}
ध्यान दें कि एक **हमलावर** **समय** को **संशोधित** कर सकता है ताकि **फाइलें** **वैध प्रतीत** हों, लेकिन वह **inode** को संशोधित नहीं कर सकता। यदि आप पाते हैं कि एक **फाइल** बताती है कि यह उसी समय बनाई और संशोधित की गई थी जैसे उसी फोल्डर में बाकी फाइलें, लेकिन **inode** **अप्रत्याशित रूप से बड़ा** है, तो उस **फाइल के टाइमस्टैम्प्स संशोधित किए गए थे**।
{% endhint %}

## विभिन्न फाइल सिस्टम संस्करणों की फाइलों की तुलना करें

#### जोड़ी गई फाइलें खोजें
```bash
git diff --no-index --diff-filter=A _openwrt1.extracted/squashfs-root/ _openwrt2.extracted/squashfs-root/
```
#### संशोधित सामग्री का पता लगाएं
```bash
git diff --no-index --diff-filter=M _openwrt1.extracted/squashfs-root/ _openwrt2.extracted/squashfs-root/ | grep -E "^\+" | grep -v "Installed-Time"
```
#### हटाए गए फाइलों को खोजें
```bash
git diff --no-index --diff-filter=A _openwrt1.extracted/squashfs-root/ _openwrt2.extracted/squashfs-root/
```
#### अन्य फिल्टर्स

**`-diff-filter=[(A|C|D|M|R|T|U|X|B)…​[*]]`**

केवल उन फाइलों का चयन करें जो जोड़ी गई हैं (`A`), कॉपी की गई हैं (`C`), हटाई गई हैं (`D`), संशोधित की गई हैं (`M`), नाम बदले गए हैं (`R`), और जिनका प्रकार (यानी नियमित फाइल, सिमलिंक, सबमॉड्यूल, …​) बदला गया है (`T`), जो अनमर्ज्ड हैं (`U`), अज्ञात हैं (`X`), या जिनकी जोड़ी टूटी हुई है (`B`). फिल्टर अक्षरों का कोई भी संयोजन (शून्य सहित) इस्तेमाल किया जा सकता है। जब `*` (सभी-या-कोई नहीं) संयोजन में जोड़ा जाता है, तो सभी पथ चुने जाते हैं अगर कोई भी फाइल तुलना में अन्य मानदंडों से मेल खाती है; अगर कोई भी फाइल अन्य मानदंडों से मेल नहीं खाती है, तो कुछ भी चुना नहीं जाता है।

इसके अलावा, **इन अपर-केस अक्षरों को निचले केस में बदलकर बाहर किया जा सकता है**। उदाहरण के लिए, `--diff-filter=ad` जोड़े गए और हटाए गए पथों को बाहर करता है।

ध्यान दें कि सभी डिफ्स में सभी प्रकार की सुविधाएँ नहीं हो सकती हैं। उदाहरण के लिए, इंडेक्स से वर्किंग ट्री तक के डिफ्स में कभी भी जोड़े गए प्रविष्टियाँ नहीं हो सकती हैं (क्योंकि डिफ में शामिल पथों का सेट इंडेक्स में क्या है इससे सीमित होता है)। इसी तरह, कॉपी किए गए और नाम बदले गए प्रविष्टियाँ नहीं आ सकती हैं अगर उन प्रकारों का पता लगाना अक्षम किया गया हो।

## संदर्भ

* [https://cdn.ttgtmedia.com/rms/security/Malware%20Forensics%20Field%20Guide%20for%20Linux%20Systems\_Ch3.pdf](https://cdn.ttgtmedia.com/rms/security/Malware%20Forensics%20Field%20Guide%20for%20Linux%20Systems\_Ch3.pdf)
* [https://www.plesk.com/blog/featured/linux-logs-explained/](https://www.plesk.com/blog/featured/linux-logs-explained/)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

क्या आप **साइबरसिक्योरिटी कंपनी** में काम करते हैं? क्या आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे**? या क्या आप **PEASS के नवीनतम संस्करण तक पहुँच चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!

* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* **जुड़ें** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**telegram समूह**](https://t.me/peass) से या **Twitter** पर मुझे **फॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**

**अपनी हैकिंग ट्रिक्स साझा करें [**hacktricks repo**](https://github.com/carlospolop/hacktricks) में PRs सबमिट करके और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud).

</details>

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उपयोग करके आसानी से **वर्कफ्लोज़ को बिल्ड और ऑटोमेट करें** जो दुनिया के **सबसे उन्नत** समुदाय टूल्स द्वारा संचालित होते हैं।\
आज ही पहुँच प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}
