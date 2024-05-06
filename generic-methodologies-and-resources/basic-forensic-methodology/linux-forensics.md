# लिनक्स फोरेंसिक्स

<figure><img src="../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm_source=hacktricks&utm_medium=text&utm_campaign=ppc&utm_content=linux-forensics) का उपयोग करें और आसानी से **सबसे उन्नत समुदाय उपकरणों** द्वारा संचालित **कार्यप्रवाह** बनाएं और स्वचालित करें।\
आज ही पहुंचें:

{% embed url="https://trickest.com/?utm_source=hacktricks&utm_medium=banner&utm_campaign=ppc&utm_content=linux-forensics" %}

<details>

<summary><strong>जानें जीरो से हीरो तक AWS हैकिंग को</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> सीखें!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी कंपनी का **विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) और हमें **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

## प्रारंभिक जानकारी एकत्र करना

### मौलिक जानकारी

सबसे पहले, सुझाव दिया जाता है कि कुछ **USB** के साथ होना चाहिए जिसमें **अच्छे जाने माने बाइनरी और लाइब्रेरी** हों (आप बस उबंटू ले सकते हैं और फोल्डर _/bin_, _/sbin_, _/lib,_ और _/lib64_ कॉपी कर सकते हैं), फिर USB को माउंट करें, और env variables को उन बाइनरी का उपयोग करने के लिए संशोधित करें:
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
#### संदेहास्पद जानकारी

मूल जानकारी प्राप्त करते समय आपको ऐसी अजीब चीजों की जांच करनी चाहिए जैसे:

* **रूट प्रक्रियाएँ** आम तौर पर कम PIDS के साथ चलती हैं, इसलिए अगर आप एक बड़ी PID के साथ एक रूट प्रक्रिया पाते हैं तो आप संदेह कर सकते हैं
* `/etc/passwd` में शैली बिना उपयोगकर्ताओं की **रजिस्टर्ड लॉगिन** की जांच करें
* `/etc/shadow` में **पासवर्ड हैश** की जांच करें जिनके पास एक शैल नहीं है

### मेमोरी डंप

चल रहे सिस्टम की मेमोरी प्राप्त करने के लिए, [**LiME**](https://github.com/504ensicsLabs/LiME) का उपयोग करना सुझावित है।\
इसे **कंपाइल** करने के लिए, आपको पीडीआई की विक्टिम मशीन का **वही कर्नेल** उपयोग करना होगा।

{% hint style="info" %}
ध्यान रखें कि आप **LiME या किसी अन्य चीज़ को** विक्टिम मशीन में इंस्टॉल नहीं कर सकते क्योंकि इससे इसमें कई बदलाव हो जाएंगे
{% endhint %}

तो, यदि आपके पास एक वैशिष्ट्यपूर्ण संस्करण का यूबंटू है तो आप `apt-get install lime-forensics-dkms` का उपयोग कर सकते हैं\
अन्य मामलों में, आपको [**LiME**](https://github.com/504ensicsLabs/LiME) को github से डाउनलोड करना होगा और सही कर्नेल हेडर्स के साथ इसे कंपाइल करना होगा। विक्टिम मशीन के **सटीक कर्नेल हेडर्स** प्राप्त करने के लिए, आप बस अपनी मशीन पर निर्देशिका `/lib/modules/<कर्नेल संस्करण>` की **कॉपी** कर सकते हैं, और फिर उन्हें उपयोग करके LiME को **कंपाइल** कर सकते हैं:
```bash
make -C /lib/modules/<kernel version>/build M=$PWD
sudo insmod lime.ko "path=/home/sansforensics/Desktop/mem_dump.bin format=lime"
```
LiME समर्थित करता है 3 **फ़ॉर्मेट**:

* Raw (हर सेगमेंट को एक साथ जोड़ा गया)
* Padded (रॉ के समान, लेकिन दाईं बिट्स में शून्य हैं)
* Lime (मेटाडेटा के साथ अनुशंसित फ़ॉर्मेट)

LiME का उपयोग **नेटवर्क के माध्यम से डंप भेजने** के लिए भी किया जा सकता है इसे सिस्टम पर स्टोर करने की बजाय कुछ इस प्रकार का उपयोग करके: `path=tcp:4444`

### डिस्क इमेजिंग

#### बंद करना

सबसे पहले, आपको **सिस्टम को बंद करने** की आवश्यकता होगी। यह हमेशा एक विकल्प नहीं है क्योंकि कई बार सिस्टम एक उत्पादन सर्वर होगा जिसे कंपनी को बंद करने की सामर्थ्य नहीं होगी।\
सिस्टम को बंद करने के **2 तरीके** हैं, एक **सामान्य बंद करना** और एक **"प्लग निकालना" बंद करना**। पहला तरीका **प्रक्रियाओं को सामान्य रूप से समाप्त करने** और **फ़ाइल सिस्टम** को **समकालीन** होने देगा, लेकिन यह भी संभावित **मैलवेयर** को **सबूत नष्ट** करने की अनुमति देगा। "प्लग निकालना" दृष्टिकोण **कुछ जानकारी की हानि** ले सकता है (ज्यादातर जानकारी नष्ट नहीं होने वाली है क्योंकि हमने पहले ही मेमोरी का इमेज ले लिया है) और **मैलवेयर को कुछ करने का कोई मौका नहीं** होगा। इसलिए, यदि आपको लगता है कि वहाँ **मैलवेयर** हो सकता है, तो सिस्टम पर **`sync`** **कमांड** को चलाएं और प्लग निकालें।

#### डिस्क का इमेज लेना

यह महत्वपूर्ण है कि **मामले से संबंधित कुछ भी कनेक्ट करने से पहले**, आपको यह सुनिश्चित करना होगा कि यह **केवल पढ़ने योग्य रूप में माउंट** होगा ताकि कोई भी जानकारी में कोई परिवर्तन न हो।
```bash
#Create a raw copy of the disk
dd if=<subject device> of=<image file> bs=512

#Raw copy with hashes along the way (more secure as it checks hashes while it's copying the data)
dcfldd if=<subject device> of=<image file> bs=512 hash=<algorithm> hashwindow=<chunk size> hashlog=<hash file>
dcfldd if=/dev/sdc of=/media/usb/pc.image hash=sha256 hashwindow=1M hashlog=/media/usb/pc.hashes
```
### डिस्क इमेज पूर्व विश्लेषण

किसी भी अधिक डेटा के साथ डिस्क इमेज को इमेजिंग करें।
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
<figure><img src="../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm_source=hacktricks&utm_medium=text&utm_campaign=ppc&utm_content=linux-forensics) का उपयोग करें और आसानी से **ऑटोमेट वर्कफ़्लो** बनाएं जो दुनिया के **सबसे उन्नत** समुदाय उपकरणों द्वारा संचालित हैं।\
आज ही पहुंचें:

{% embed url="https://trickest.com/?utm_source=hacktricks&utm_medium=banner&utm_campaign=ppc&utm_content=linux-forensics" %}

## ज्ञात मैलवेयर की खोज

### संशोधित सिस्टम फ़ाइलें

Linux सिस्टम घटकों की अखंडता सुनिश्चित करने के लिए उपकरण प्रदान करता है, जो संभावित समस्यात्मक फ़ाइलों को पहचानने में महत्वपूर्ण हैं।

* **RedHat आधारित सिस्टम**: व्यापक जांच के लिए `rpm -Va` का उपयोग करें।
* **Debian आधारित सिस्टम**: प्रारंभिक सत्यापन के लिए `dpkg --verify` का उपयोग करें, और फिर `debsums | grep -v "OK$"` ( `apt-get install debsums` के साथ `debsums` को स्थापित करने के बाद) का उपयोग करें किसी भी समस्याओं की पहचान के लिए।

### मैलवेयर/रूटकिट डिटेक्टर्स

मैलवेयर खोजने में सहायक हो सकने वाले उपकरणों के बारे में जानने के लिए निम्नलिखित पृष्ठ पढ़ें:

{% content-ref url="malware-analysis.md" %}
[malware-analysis.md](malware-analysis.md)
{% endcontent-ref %}

## स्थापित कार्यक्रमों की खोज

Debian और RedHat सिस्टमों पर स्थापित कार्यक्रमों की प्रभावी खोज करने के लिए, सामान्य निरीक्षण के साथ सिस्टम लॉग और डेटाबेस का सहारा लेना ध्यान में रखें।

* Debian के लिए, _**`/var/lib/dpkg/status`**_ और _**`/var/log/dpkg.log`**_ की जांच करें पैकेज स्थापनाओं के बारे में विवरण प्राप्त करने के लिए, `grep` का उपयोग करें विशिष्ट जानकारी के लिए।
* RedHat उपयोगकर्ता RPM डेटाबेस को `rpm -qa --root=/mntpath/var/lib/rpm` के साथ क्वेरी कर सकते हैं इंस्टॉल किए गए पैकेजों की सूची के लिए।

इन पैकेज प्रबंधकों के बाहर मैन्युअल या स्थापित किए गए सॉफ़्टवेयर को खोजने के लिए, _**`/usr/local`**_, _**`/opt`**_, _**`/usr/sbin`**_, _**`/usr/bin`**_, _**`/bin`**_, और _**`/sbin`**_ जैसे निर्देशिकाओं का अन्वेषण करें। सिस्टम-विशेष आदेशों के साथ निर्देशिका सूचियों को मिलाकर पहचानें जिनमें जाने-माने पैकेजों से संबंधित नहीं हैं, सभी स्थापित कार्यक्रमों की खोज को बढ़ावा दें।
```bash
# Debian package and log details
cat /var/lib/dpkg/status | grep -E "Package:|Status:"
cat /var/log/dpkg.log | grep installed
# RedHat RPM database query
rpm -qa --root=/mntpath/var/lib/rpm
# Listing directories for manual installations
ls /usr/sbin /usr/bin /bin /sbin
# Identifying non-package executables (Debian)
find /sbin/ -exec dpkg -S {} \; | grep "no path found"
# Identifying non-package executables (RedHat)
find /sbin/ –exec rpm -qf {} \; | grep "is not"
# Find exacuable files
find / -type f -executable | grep <something>
```
<figure><img src="../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm_source=hacktricks&utm_medium=text&utm_campaign=ppc&utm_content=linux-forensics) का उपयोग करें और आसानी से **वर्ल्ड के सबसे उन्नत** समुदाय उपकरणों द्वारा संचालित **कार्यप्रवाह** बनाएं और स्वचालित करें।\
आज ही पहुंचें:

{% embed url="https://trickest.com/?utm_source=hacktricks&utm_medium=banner&utm_campaign=ppc&utm_content=linux-forensics" %}

## हटाए गए चल रहे बाइनरी को पुनः प्राप्त करें

कल्पना करें एक प्रक्रिया जो /tmp/exec से निष्पादित की गई थी और फिर हटा दी गई थी। इसे निकालना संभव है
```bash
cd /proc/3746/ #PID with the exec file deleted
head -1 maps #Get address of the file. It was 08048000-08049000
dd if=mem bs=1 skip=08048000 count=1000 of=/tmp/exec2 #Recorver it
```
## ऑटोस्टार्ट स्थानों की जांच

### निर्धारित कार्यों
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

मालवेयर को सेवा के रूप में स्थापित किया जा सकने वाले पथ:

* **/etc/inittab**: rc.sysinit जैसे प्रारंभीकरण स्क्रिप्ट को बुलाता है, जो आगे स्टार्टअप स्क्रिप्ट्स की ओर निर्देशित करता है।
* **/etc/rc.d/** और **/etc/rc.boot/**: सेवा स्टार्टअप के लिए स्क्रिप्ट्स को रखते हैं, जिन्हें पुराने Linux संस्करणों में पाया जाता है।
* **/etc/init.d/**: Debian जैसे कुछ Linux संस्करणों में स्टार्टअप स्क्रिप्ट्स को स्टोर करने के लिए उपयोग किया जाता है।
* सेवाएं **/etc/inetd.conf** या **/etc/xinetd/** के माध्यम से भी सक्रिय की जा सकती हैं, जो Linux प्रकार पर निर्भर करता है।
* **/etc/systemd/system**: सिस्टम और सेवा प्रबंधक स्क्रिप्ट के लिए एक निर्देशिका।
* **/etc/systemd/system/multi-user.target.wants/**: एक मल्टी-यूजर रनलेवल में शुरू किए जाने वाली सेवाओं के लिंक्स को रखता है।
* **/usr/local/etc/rc.d/**: कस्टम या थर्ड-पार्टी सेवाओं के लिए।
* **\~/.config/autostart/**: उपयोगकर्ता-विशिष्ट स्वचालित स्टार्टअप एप्लिकेशन के लिए, जो उपयोगकर्ता-लक्षित मालवेयर के लिए एक छुपाने की जगह हो सकता है।
* **/lib/systemd/system/**: सिस्टम-व्यापी डिफ़ॉल्ट यूनिट फ़ाइलें इंस्टॉल किए गए पैकेज्स द्वारा प्रदान की गई हैं।

### कर्नेल मॉड्यूल

लिनक्स कर्नेल मॉड्यूल, जो अक्सर मालवेयर द्वारा रूटकिट घटक के रूप में उपयोग किए जाते हैं, सिस्टम बूट पर लोड किए जाते हैं। इन मॉड्यूल के लिए महत्वपूर्ण निर्देशिकाएं और फ़ाइलें शामिल हैं:

* **/lib/modules/$(uname -r)**: चल रहे कर्नेल संस्करण के लिए मॉड्यूल्स को रखता है।
* **/etc/modprobe.d**: मॉड्यूल लोडिंग को नियंत्रित करने के लिए कॉन्फ़िगरेशन फ़ाइलें शामिल हैं।
* **/etc/modprobe** और **/etc/modprobe.conf**: वैश्विक मॉड्यूल सेटिंग्स के लिए फ़ाइलें।

### अन्य स्वचालित स्टार्टअर्ट स्थान

लिनक्स उपयोगकर्ता लॉगिन पर स्वचालित रूप से कार्यों को निष्पादित करने के लिए विभिन्न फ़ाइलों का उपयोग करता है, जो संभावित रूप से मालवेयर को छुपाने के लिए हो सकते हैं:

* **/etc/profile.d/**\*, **/etc/profile**, और **/etc/bash.bashrc**: किसी भी उपयोगकर्ता लॉगिन के लिए निष्पादित किया जाता है।
* **\~/.bashrc**, **\~/.bash\_profile**, **\~/.profile**, और **\~/.config/autostart**: उपयोगकर्ता-विशिष्ट फ़ाइलें जो उनके लॉगिन पर चलती हैं।
* **/etc/rc.local**: सभी सिस्टम सेवाएं शुरू होने के बाद चलाया जाता है, जो मल्टीयूजर परिवेश की ओर संकेत करता है।

## लॉग की जांच

लिनक्स सिस्टम विभिन्न लॉग फ़ाइलों के माध्यम से उपयोगकर्ता गतिविधियों और सिस्टम घटनाओं का ट्रैक करते हैं। ये लॉग अनधिकृत पहुंच, मालवेयर संक्रमण, और अन्य सुरक्षा घटनाओं की पहचान के लिए महत्वपूर्ण हैं। मुख्य लॉग फ़ाइलें शामिल हैं:

* **/var/log/syslog** (Debian) या **/var/log/messages** (RedHat): सिस्टम-व्यापी संदेश और गतिविधियों को कैप्चर करते हैं।
* **/var/log/auth.log** (Debian) या **/var/log/secure** (RedHat): प्रमाणीकरण प्रयासों, सफल और असफल लॉगिन को रिकॉर्ड करते हैं।
* `grep -iE "session opened for|accepted password|new session|not in sudoers" /var/log/auth.log` का उपयोग संबंधित प्रमाणीकरण घटनाओं को फ़िल्टर करने के लिए करें।
* **/var/log/boot.log**: सिस्टम स्टार्टअप संदेश शामिल करता है।
* **/var/log/maillog** या **/var/log/mail.log**: ईमेल सर्वर गतिविधियों को लॉग करता है, जो ईमेल संबंधित सेवाओं का ट्रैकिंग करने के लिए उपयोगी है।
* **/var/log/kern.log**: कर्नेल संदेश स्टोर करता है, जिसमें त्रुटियाँ और चेतावनियाँ शामिल हैं।
* **/var/log/dmesg**: डिवाइस ड्राइवर संदेशों को रखता है।
* **/var/log/faillog**: सुरक्षा उल्लंघन जांच में मदद करने वाले असफल लॉगिन प्रयासों को रिकॉर्ड करता है।
* **/var/log/cron**: क्रॉन जॉब क्रियाओं को लॉग करता है।
* **/var/log/daemon.log**: पिछले सेवा गतिविधियों का ट्रैक करता है।
* **/var/log/btmp**: असफल लॉगिन प्रयासों को दस्तावेज़ करता है।
* **/var/log/httpd/**: Apache HTTPD त्रुटि और एक्सेस लॉग्स को रखता है।
* **/var/log/mysqld.log** या **/var/log/mysql.log**: MySQL डेटाबेस गतिविधियों को लॉग करता है।
* **/var/log/xferlog**: FTP फ़ाइल स्थानांतरण को रिकॉर्ड करता है।
* **/var/log/**: यहाँ अप्रत्याशित लॉग के लिए हमेशा जांच करें।

{% hint style="info" %}
लिनक्स सिस्टम लॉग और ऑडिट सबसिस्टम को एक उत्पीड़न या मालवेयर घटना में अक्षम या हटा दिया जा सकता है। क्योंकि लिनक्स सिस्टम पर लॉग आम तौर पर दुर्भाग्यपूर्ण गतिविधियों के बारे में सबसे उपयोगी जानकारी शामिल करते हैं, अतः अवैध पहुंच या टैम्परिंग का संकेत हो सकने वाले खालियों या अव्यवस्थित प्रविष्टियों की खोज करना महत्वपूर्ण है।
{% endhint %}

**लिनक्स प्रत्येक उपयोगकर्ता के लिए एक कमांड हिस्ट्री बनाए रखता है**, जिसे इसमें स्टोर किया जाता है:

* \~/.bash\_history
* \~/.zsh\_history
* \~/.zsh\_sessions/\*
* \~/.python\_history
* \~/.\*\_history

इसके अतिरिक्त, `last -Faiwx` कमांड उपयोगकर्ता लॉगिन की सूची प्रदान करता है। अज्ञात या अप्रत्याशित लॉगिन के लिए इसे जांचें।

अतिरिक्त अधिक अधिकार प्रदान कर सकने वाली फ़ाइलें जांचें:

* अनपेक्षित उपयोगकर्ता अधिकारों के लिए `/etc/sudoers` की समीक्षा करें जो प्रदान किए गए हो सकते हैं।
* अनपेक्षित उपयोगकर्ता अधिकारों के लिए `/etc/sudoers.d/` की समीक्षा करें जो प्रदान किए गए हो सकते हैं।
* असामान्य समूह सदस्यता या अनुमतियों की पहचान के लिए `/etc/groups` की समीक्षा करें।
* असामान्य समूह सदस्यता या अनुमतियों की पहचान के लिए `/etc/passwd` की समीक्षा करें।

कुछ ऐप्स भी अपने खुद के लॉग उत्पन्न करते हैं:

* **SSH**: अनधिकृत रिमोट कनेक्शन के लिए _\~/.ssh/authorized\_keys_ और _\~/.ssh/known\_hosts_
```bash
pip3 install usbrip
usbrip ids download #Download USB ID database
```
### उदाहरण
```bash
usbrip events history #Get USB history of your curent linux machine
usbrip events history --pid 0002 --vid 0e0f --user kali #Search by pid OR vid OR user
#Search for vid and/or pid
usbrip ids download #Downlaod database
usbrip ids search --pid 0002 --vid 0e0f #Search for pid AND vid
```
अधिक उदाहरण और जानकारी गिथब में: [https://github.com/snovvcrash/usbrip](https://github.com/snovvcrash/usbrip)

<figure><img src="../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm_source=hacktricks&utm_medium=text&utm_campaign=ppc&utm_content=linux-forensics) का उपयोग करें और आसानी से **ऑटोमेट वर्कफ़्लो** बनाएं जो दुनिया के **सबसे उन्नत** समुदाय उपकरणों द्वारा संचालित हैं।\
आज ही पहुंचें:

{% embed url="https://trickest.com/?utm_source=hacktricks&utm_medium=banner&utm_campaign=ppc&utm_content=linux-forensics" %}

## उपयोगकर्ता खातों और लॉगऑन गतिविधियों की समीक्षा

अनूठे नामों या खातों की जांच करें जो अनुमति दी गई घटनाओं के निकट बनाए गए या उपयोग किए गए हों, जो अनाधिकृत हो सकते हैं, _**/etc/passwd**_, _**/etc/shadow**_ और **सुरक्षा लॉग** की जांच करें। संभावित सुडो ब्रूट-फ़ोर्स हमलों की भी जांच करें।\
इसके अतिरिक्त, उपयोगकर्ताओं को दी गई अप्रत्याशित विशेषाधिकारों के लिए फ़ाइलें जैसे _**/etc/sudoers**_ और _**/etc/groups**_ की भी जांच करें।\
अंत में, **कोई पासवर्ड नहीं** वाले खातों या **आसानी से अनुमान लगाए जाने वाले** पासवर्डों की जांच करें।

## फ़ाइल सिस्टम की जांच

### मैलवेयर जांच में फ़ाइल सिस्टम संरचनाओं का विश्लेषण

मैलवेयर घटनाओं की जांच करते समय, फ़ाइल सिस्टम की संरचना एक महत्वपूर्ण सूत्र होती है, जो घटनाओं की क्रमवार्ता और मैलवेयर की सामग्री दिखाती है। हालांकि, मैलवेयर लेखक इस विश्लेषण को बाधित करने के लिए तकनीक विकसित कर रहे हैं, जैसे फ़ाइल टाइमस्टैम्प को संशोधित करना या डेटा भंडारण के लिए फ़ाइल सिस्टम से बचना।

इन एंटी-फोरेंसिक विधियों का विरोध करने के लिए, निम्नलिखित महत्वपूर्ण है:

* **घटना समयरेखा विश्लेषण** का विस्तृत विश्लेषण करें, जैसे कि **Autopsy** जैसे उपकरणों का उपयोग करके घटना समयरेखाओं को दृश्यीकरण करने या **Sleuth Kit's** `mactime` का उपयोग विस्तृत समयरेखा डेटा के लिए।
* तंत्रिका के $PATH में **अप्रत्याशित स्क्रिप्ट** की जांच करें, जिसमें हमलावर द्वारा उपयोग किए जाने वाले शैल या PHP स्क्रिप्ट शामिल हो सकते हैं।
* **अटिपिकल फ़ाइलों** के लिए `/dev` जांचें, क्योंकि इसमें पारंपरिक रूप से विशेष फ़ाइलें होती हैं, लेकिन यह मैलवेयर संबंधित फ़ाइलों को भी रख सकता है।
* ".. " (डॉट डॉट स्पेस) या "..^G" (डॉट डॉट कंट्रोल-जी) जैसे नामों वाली **छिपी हुई फ़ाइलें या निर्देशिकाएं** खोजें, जो कट्टर सामग्री को छुपा सकती हैं।
* `find / -user root -perm -04000 -print` आदेश का उपयोग करके **सेटयूआइड रूट फ़ाइलें** पहचानें। यह उच्च अनुमतियों वाली फ़ाइलों को खोजता है, जिन्हें हमलावरों द्वारा दुरुपयोग किया जा सकता है।
* इनोड तालिकाओं में **मिटाने की समयचिह्न** की समीक्षा करें, जिससे बड़ी संख्या में फ़ाइलों को मिटाया गया हो सकता है, जो रूटकिट या ट्रोजन की मौजूदगी की संकेत कर सकता है।
* एक के बाद एक आने वाली **इनोड** की जांच करें, क्योंकि एक बार एक को पहचानने के बाद ये निकट में रखे गए मैलवेयर संबंधित फ़ाइलों को रख सकते हैं।
* हाल ही में संशोधित फ़ाइलों के लिए सामान्य बाइनरी निर्देशिकाएं (_/bin_, _/sbin_) की जांच करें, क्योंकि ये मैलवेयर द्वारा संशोधित किए जा सकते हैं।
````bash
# List recent files in a directory:
ls -laR --sort=time /bin```

# Sort files in a directory by inode:
ls -lai /bin | sort -n```
````
{% hint style="info" %}
ध्यान दें कि एक **हमलावर** समय **संशोधित** कर सकता है ताकि **फ़ाइलें वास्तविक** लगें, लेकिन वह **inode** को **संशोधित** नहीं कर सकता। अगर आपको लगता है कि एक **फ़ाइल** इस बात का संकेत देती है कि यह उसी फ़ोल्डर में बाकी फ़ाइलों के साथ **एक ही समय** पर बनाई और संशोधित की गई थी, लेकिन **inode** **अपेक्षाकृत बड़ा** है, तो **उस फ़ाइल के timestamps को संशोधित किया गया था**।
{% endhint %}

## विभिन्न फ़ाइल सिस्टम संस्करणों की तुलना करें

### फ़ाइल सिस्टम संस्करण तुलना सारांश

संशोधित संस्करणों की तुलना करने और परिवर्तनों को पहचानने के लिए, हम सरलित `git diff` कमांड का उपयोग करते हैं:

* **नई फ़ाइलें खोजने** के लिए, दो निर्देशिकाओं की तुलना करें:
```bash
git diff --no-index --diff-filter=A path/to/old_version/ path/to/new_version/
```
* **संशोधित सामग्री के लिए**, विशिष्ट पंक्तियों को नजरअंदाज करते हुए परिवर्तनों की सूची बनाएं:
```bash
git diff --no-index --diff-filter=M path/to/old_version/ path/to/new_version/ | grep -E "^\+" | grep -v "Installed-Time"
```
* **हटाए गए फ़ाइलें पता लगाने के लिए**:
```bash
git diff --no-index --diff-filter=D path/to/old_version/ path/to/new_version/
```
* **फ़िल्टर विकल्प** (`--diff-filter`) मदद करते हैं किसी विशेष परिवर्तन जैसे जोड़ी गई (`A`), हटाई गई (`D`), या संशोधित (`M`) फ़ाइलों तक पहुँचने में।
* `A`: जोड़ी गई फ़ाइलें
* `C`: कॉपी की गई फ़ाइलें
* `D`: हटाई गई फ़ाइलें
* `M`: संशोधित फ़ाइलें
* `R`: नामकरण की गई फ़ाइलें
* `T`: प्रकार के परिवर्तन (उदा., फ़ाइल से सिंबलिंक)
* `U`: अमर्जित फ़ाइलें
* `X`: अज्ञात फ़ाइलें
* `B`: टूटी हुई फ़ाइलें

## संदर्भ

* [https://cdn.ttgtmedia.com/rms/security/Malware%20Forensics%20Field%20Guide%20for%20Linux%20Systems\_Ch3.pdf](https://cdn.ttgtmedia.com/rms/security/Malware%20Forensics%20Field%20Guide%20for%20Linux%20Systems\_Ch3.pdf)
* [https://www.plesk.com/blog/featured/linux-logs-explained/](https://www.plesk.com/blog/featured/linux-logs-explained/)
* [https://git-scm.com/docs/git-diff#Documentation/git-diff.txt---diff-filterACDMRTUXB82308203](https://git-scm.com/docs/git-diff#Documentation/git-diff.txt---diff-filterACDMRTUXB82308203)
* **पुस्तक: मैलवेयर फोरेंसिक्स फ़ील्ड गाइड फॉर लिनक्स सिस्टम्स: डिजिटल फोरेंसिक्स फील्ड गाइड्स**

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को हैकट्रिक्स में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या हैकट्रिक्स को पीडीएफ में डाउनलोड** करने का एक्सेस चाहिए? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!

* खोजें [**द पीएस फैमिली**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**एनएफटी**](https://opensea.io/collection/the-peass-family) संग्रह
* प्राप्त करें [**आधिकारिक पीएस और हैकट्रिक्स स्वैग**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर फॉलो करें 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**

**अपने हैकिंग ट्रिक्स साझा करें** [**हैकट्रिक्स रेपो**](https://github.com/carlospolop/hacktricks) **और** [**हैकट्रिक्स-क्लाउड रेपो**](https://github.com/carlospolop/hacktricks-cloud) **को PR जमा करके।**

</details>

<figure><img src="../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm_source=hacktricks&utm_medium=text&utm_campaign=ppc&utm_content=linux-forensics) का उपयोग करें और दुनिया के **सबसे उन्नत समुदाय उपकरणों** द्वारा संचालित **औटोमेटेड वर्कफ़्लो** आसानी से बनाएं।\
आज ही पहुंच प्राप्त करें:

{% embed url="https://trickest.com/?utm_source=hacktricks&utm_medium=banner&utm_campaign=ppc&utm_content=linux-forensics" %}
