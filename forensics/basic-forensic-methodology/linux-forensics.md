# लिनक्स फोरेंसिक्स

<figure><img src="../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उपयोग करके आसानी से वर्कफ़्लो बनाएं और **सबसे उन्नत समुदाय उपकरणों** द्वारा कार्यप्रवाहों को स्वचालित करें।\
आज ही पहुंच प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **हैकट्रिक्स में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल हों** या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का पालन करें**.
* **अपने हैकिंग ट्रिक्स साझा करें,** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **में PR जमा करके**।

</details>

## प्रारंभिक जानकारी इकट्ठा करना

### मूलभूत जानकारी

सबसे पहले, यह सिफारिश की जाती है कि आपके पास कुछ **USB** होना चाहिए जिसमें **अच्छी तरह से ज्ञात बाइनरी और पुस्तकालय** हों (आप सिर्फ यूबंटू ले सकते हैं और फ़ोल्डर _/bin_, _/sbin_, _/lib,_ और _/lib64_ की प्रतिलिपि बना सकते हैं), फिर USB को माउंट करें, और उन बाइनरीज़ का उपयोग करने के लिए env variables को संशोधित करें:
```bash
export PATH=/mnt/usb/bin:/mnt/usb/sbin
export LD_LIBRARY_PATH=/mnt/usb/lib:/mnt/usb/lib64
```
जब आपने सिस्टम को अच्छे और ज्ञात बाइनरीज़ का उपयोग करने के लिए कॉन्फ़िगर कर लिया है, तो आप **कुछ मूलभूत जानकारी निकालना शुरू** कर सकते हैं:
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

मूल जानकारी प्राप्त करते समय आपको निम्नलिखित चीजों की जांच करनी चाहिए:

* **रूट प्रक्रियाएं** आमतौर पर कम PIDS के साथ चलती हैं, इसलिए यदि आपको एक बड़ी PID के साथ एक रूट प्रक्रिया मिलती है तो आप संदेह कर सकते हैं
* `/etc/passwd` में शेल के बिना उपयोगकर्ताओं के **पंजीकृत लॉगिन** की जांच करें
* शेल के बिना उपयोगकर्ताओं के लिए `/etc/shadow` में **पासवर्ड हैश** की जांच करें

### मेमोरी डंप

चल रहे सिस्टम की मेमोरी प्राप्त करने के लिए, [**LiME**](https://github.com/504ensicsLabs/LiME) का उपयोग करना सिफारिश किया जाता है।\
इसे **कंपाइल** करने के लिए, आपको पीडीआईएस का उपयोग करना होगा जो पीडीआईएस यात्री मशीन का उपयोग कर रही है।

{% hint style="info" %}
ध्यान दें कि आप **LiME या किसी अन्य चीज़ को** पीडीआईएस यात्री मशीन में स्थापित नहीं कर सकते क्योंकि इससे कई बदलाव हो जाएंगे
{% endhint %}

तो, यदि आपके पास एक एकसंदर्भ रूप में उबंटू का संस्करण है, तो आप `apt-get install lime-forensics-dkms` का उपयोग कर सकते हैं\
अन्य मामलों में, आपको [**LiME**](https://github.com/504ensicsLabs/LiME) को github से डाउनलोड करना होगा और सही कर्नल हेडर के साथ इसे कंपाइल करना होगा। पीडीआईएस यात्री मशीन के **ठीक कर्नल हेडर प्राप्त करने** के लिए, आप बस इस निर्देशिका `/lib/modules/<kernel version>` को अपनी मशीन पर **कॉपी करें**, और फिर उनका उपयोग करके LiME को **कंपाइल** करें:
```bash
make -C /lib/modules/<kernel version>/build M=$PWD
sudo insmod lime.ko "path=/home/sansforensics/Desktop/mem_dump.bin format=lime"
```
LiME 3 **फॉर्मेट** का समर्थन करता है:

* रॉ (हर सेगमेंट को एक साथ जोड़ा गया)
* पैडेड (रॉ के समान, लेकिन दायीं बिट्स में शून्य होता है)
* लाइम (मेटाडेटा के साथ अनुशंसित फॉर्मेट)

LiME का उपयोग करके आप **डंप को नेटवर्क के माध्यम से भेज सकते** हैं इसे सिस्टम पर संग्रहीत करने की बजाय कुछ इस तरह से करें: `path=tcp:4444`

### डिस्क इमेजिंग

#### बंद करना

सबसे पहले, आपको **सिस्टम को बंद करने की आवश्यकता** होगी। यह हमेशा एक विकल्प नहीं होता क्योंकि कभी-कभी सिस्टम एक उत्पादन सर्वर होता है जिसे कंपनी को बंद नहीं करने की आर्थिक संभावना होती है।\
सिस्टम को बंद करने के **2 तरीके** होते हैं, एक **सामान्य बंद करना** और एक **"प्लग निकालें" बंद करना**। पहला वाला तरीका **प्रक्रियाओं को सामान्य रूप से समाप्त करने** और **फ़ाइल सिस्टम** को **समकालीन** करने की अनुमति देगा, लेकिन यह भी संभावित **मैलवेयर** को **सबूत नष्ट** करने की अनुमति देगा। "प्लग निकालें" तरीका **कुछ जानकारी की हानि** ले सकता है (ज्यादातर जानकारी नष्ट नहीं होगी क्योंकि हमने पहले ही मेमोरी की छवि ले ली है) और **मैलवेयर को इसके बारे में कुछ करने का कोई मौका नहीं होगा**। इसलिए, यदि आपको लगता है कि एक **मैलवेयर** हो सकता है, तो सिस्टम पर **`sync`** **कमांड** को निष्पादित करें और प्लग निकालें।

#### डिस्क की छवि लेना

महत्वपूर्ण है कि **केस से संबंधित किसी भी चीज़ को कनेक्ट करने से पहले**, आपको यह सुनिश्चित करना होगा कि यह **केवल रीड ओनली** के रूप में माउंट होगा ताकि कोई भी जानकारी संशोधित न हो।
```bash
#Create a raw copy of the disk
dd if=<subject device> of=<image file> bs=512

#Raw copy with hashes along the way (more secure as it checks hashes while it's copying the data)
dcfldd if=<subject device> of=<image file> bs=512 hash=<algorithm> hashwindow=<chunk size> hashlog=<hash file>
dcfldd if=/dev/sdc of=/media/usb/pc.image hash=sha256 hashwindow=1M hashlog=/media/usb/pc.hashes
```
### डिस्क इमेज पूर्व-विश्लेषण

कोई अधिक डेटा के साथ डिस्क इमेज को इमेजिंग करें।
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
<figure><img src="../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उपयोग करके आसानी से वर्कफ़्लो बनाएं और संचालित करें, जो दुनिया के सबसे उन्नत समुदाय उपकरणों द्वारा संचालित होते हैं।\
आज ही पहुंच प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## ज्ञात मैलवेयर की खोज करें

### संशोधित सिस्टम फ़ाइलें

कुछ लिनक्स सिस्टमों में एक सुविधा होती है जो बहुत सारे स्थापित कंपोनेंटों की प्रामाणिकता की जांच करने की सुविधा प्रदान करती है, जिससे असामान्य या अप्रासंगिक फ़ाइलें पहचानने का एक प्रभावी तरीका होता है। उदाहरण के लिए, लिनक्स पर `rpm -Va` का उपयोग रेडहैट पैकेज प्रबंधक का उपयोग करके स्थापित सभी पैकेजों की प्रामाणिकता की जांच करने के लिए किया जाता है।
```bash
#RedHat
rpm -Va
#Debian
dpkg --verify
debsums | grep -v "OK$" #apt-get install debsums
```
### मैलवेयर/रूटकिट डिटेक्टर्स

मैलवेयर को खोजने में सहायक होने वाले टूल्स के बारे में जानने के लिए निम्नलिखित पेज को पढ़ें:

{% content-ref url="malware-analysis.md" %}
[malware-analysis.md](malware-analysis.md)
{% endcontent-ref %}

## स्थापित कार्यक्रमों की खोज

### पैकेज प्रबंधक

डेबियन-आधारित सिस्टमों पर, _**/var/ lib/dpkg/status**_ फ़ाइल में स्थापित पैकेज के बारे में विवरण होते हैं और _**/var/log/dpkg.log**_ फ़ाइल में जब कोई पैकेज स्थापित होता है तो जानकारी दर्ज की जाती है।\
रेडहैट और संबंधित लिनक्स वितरणों पर **`rpm -qa --root=/ mntpath/var/lib/rpm`** कमांड सिस्टम पर एक आरपीएम डेटाबेस की सामग्री की सूची बनाएगी।
```bash
#Debian
cat /var/lib/dpkg/status | grep -E "Package:|Status:"
cat /var/log/dpkg.log | grep installed
#RedHat
rpm -qa --root=/ mntpath/var/lib/rpm
```
### अन्य

**उपरोक्त कमांडों द्वारा सभी स्थापित कार्यक्रमों की सूची नहीं दी जाएगी** क्योंकि कुछ ऐप्लिकेशन कुछ सिस्टमों के लिए पैकेज के रूप में उपलब्ध नहीं होते हैं और स्रोत कोड से स्थापित किए जाने की आवश्यकता होती है। इसलिए, _**/usr/local**_ और _**/opt**_ जैसे स्थानों की समीक्षा अन्य ऐप्लिकेशनों को खोल सकती है जो स्रोत कोड से संकलित और स्थापित किए गए हैं।
```bash
ls /opt /usr/local
```
एक और अच्छा विचार है **जांचें** कि **$PATH** के भीतर **सामान्य फ़ोल्डर** में किसी **स्थापित पैकेज से संबंधित नहीं** हैं बाइनरी को।
```bash
#Both lines are going to print the executables in /sbin non related to installed packages
#Debian
find /sbin/ -exec dpkg -S {} \; | grep "no path found"
#RedHat
find /sbin/ –exec rpm -qf {} \; | grep "is not"
```
<figure><img src="../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उपयोग करके आसानी से वर्कफ़्लो बनाएं और विश्व के सबसे उन्नत समुदाय उपकरणों द्वारा कार्यों को स्वचालित करें।\
आज ही पहुंच प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## हटाए गए चल रहे बाइनरी को पुनर्प्राप्त करें

![](<../../.gitbook/assets/image (641).png>)

## ऑटोस्टार्ट स्थानों की जांच करें

### निर्धारित कार्यों की सूची
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

मैलवेयर के रूप में नए, अनधिकृत सेवा के रूप में अपने आप को अंदर घुसाना बहुत सामान्य है। लिनक्स में कई स्क्रिप्ट होते हैं जो कंप्यूटर बूट होते समय सेवाएं शुरू करने के लिए उपयोग किए जाते हैं। प्रारंभिक स्टार्टअप स्क्रिप्ट _**/etc/inittab**_ अन्य स्क्रिप्टों को बुलाता है जैसे कि rc.sysinit और _**/etc/rc.d/**_ निर्देशिका के तहत विभिन्न स्टार्टअप स्क्रिप्ट, या पुराने संस्करणों में _**/etc/rc.boot/**_। अन्य लिनक्स के संस्करणों पर, जैसे डेबियन, स्टार्टअप स्क्रिप्ट _**/etc/init.d/**_ निर्देशिका में संग्रहीत होते हैं। इसके अलावा, कुछ सामान्य सेवाएं _**/etc/inetd.conf**_ या _**/etc/xinetd/**_ में सक्षम की जाती हैं, लिनक्स के संस्करण पर निर्भर करता है। डिजिटल जांचकर्ताओं को इन सभी स्टार्टअप स्क्रिप्टों की अनौपचारिक प्रविष्टियों के लिए जांच करनी चाहिए।

* _**/etc/inittab**_
* _**/etc/rc.d/**_
* _**/etc/rc.boot/**_
* _**/etc/init.d/**_
* _**/etc/inetd.conf**_
* _**/etc/xinetd/**_
* _**/etc/systemd/system**_
* _**/etc/systemd/system/multi-user.target.wants/**_

### कर्नल मॉड्यूल

लिनक्स सिस्टम पर, मैलवेयर पैकेज के लिए कर्नल मॉड्यूल आमतौर पर रूटकिट घटक के रूप में उपयोग होते हैं। कर्नल मॉड्यूल सिस्टम बूट होते समय `/lib/modules/'uname -r'` और `/etc/modprobe.d` निर्देशिकाओं में कॉन्फ़िगरेशन जानकारी के आधार पर लोड होते हैं, और `/etc/modprobe` या `/etc/modprobe.conf` फ़ाइल में। इन क्षेत्रों की जांच करनी चाहिए जो मैलवेयर से संबंधित हो सकते हैं।

### अन्य आटोस्टार्ट स्थान

लिनक्स उपयोगकर्ता सिस्टम में एक निर्धारित समय पर एक्सेक्यूटेबल चालू करने के लिए लिनक्स कई कॉन्फ़िगरेशन फ़ाइलों का उपयोग करता है जो मैलवेयर के संकेत शामिल हो सकते हैं।

* _**/etc/profile.d/\***_ , _**/etc/profile**_ , _**/etc/bash.bashrc**_ किसी भी उपयोगकर्ता खाते में लॉग इन होने पर निष्पादित होते हैं।
* _**∼/.bashrc**_ , _**∼/.bash\_profile**_ , _**\~/.profile**_ , _**∼/.config/autostart**_ विशेष उपयोगकर्ता लॉग इन होने पर निष्पादित होते हैं।
* _**/etc/rc.local**_ यह पारंपरिक रूप से सभी साधारित सिस्टम सेवाओं के बाद निष्पादित होता है, एक मल्टीयूजर रनलेवल में स्विच करने की प्रक्रिया के अंत में।

## लॉग जांचें

संक्रमित सिस्टम पर उपलब्ध सभी लॉग फ़ाइलों में खोजें कि क्या कोई खराब निष्पादन और संबंधित गतिविधियों के निशान हैं, जैसे कि एक नई सेवा की सृजना।

### प्योर लॉग

सिस्टम और सुरक्षा लॉग में रिकॉर्ड किए गए **लॉगिन** घटनाएं, नेटवर्क के माध्यम से लॉगिन, इसका पता लगा सकते हैं कि किसी खाते के माध्यम से एक संक्रमित सिस्टम में **मैलवेयर** या **अतिचारी उपयोगकर्ता** ने एक निश्चित समय पर पहुंच हासिल की है। मैलवेयर संक्रमण के समय के आसपास की अन्य घटनाएं सिस्टम लॉग में कैप्चर की जा सकती हैं, जिसमें एक **नई सेवा** की **सृजना** या घटनाओं की जानकारी शामिल हो सकती है।\
दिलचस्प सिस्टम लॉगिन:

* **/var/log/syslog** (डेबियन) या **/var/log/messages** (रेडहैट)
* सिस्टम के बारे में सामान्य संदेश और जानकारी दिखाता है। यह वैश्विक सिस्टम के लिए सभी गतिविधि का डेटा लॉग है।
* **/var/log/auth.log** (डेबियन) या **/var/log/secure** (रेडहैट)
* सफल या असफल लॉगिन, और प्रमाणीकरण प्रक्रियाओं के लिए प्रमाणीकरण लॉग रखता है। संग्रहण सिस्टम प्रकार पर निर्भर करता है।
* `cat /var/log/auth.log | grep -iE "session opened for|accepted password|new session|not in sudoers"`
* **/var/log/boot.log**: स्टार्टअप सं
### एप्लिकेशन के निशान

* **SSH**: कंप्रोमाइज़ हुए सिस्टम से SSH का उपयोग करके किए गए सिस्टम के साथ कनेक्शन के परिणामस्वरूप प्रत्येक उपयोगकर्ता खाते के लिए फ़ाइलों में प्रविष्टियाँ बनाई जाती हैं (_**∼/.ssh/authorized\_keys**_ और _**∼/.ssh/known\_keys**_)। ये प्रविष्टियाँ दूरस्थ होस्टों के होस्टनाम या आईपी ​​पते का पता लगा सकती हैं।
* **Gnome डेस्कटॉप**: उपयोगकर्ता खातों में _**∼/.recently-used.xbel**_ फ़ाइल हो सकती है जिसमें Gnome डेस्कटॉप पर चल रहे एप्लिकेशन का उपयोग करके हाल ही में एक्सेस की गई फ़ाइलों के बारे में जानकारी होती है।
* **VIM**: उपयोगकर्ता खातों में _**∼/.viminfo**_ फ़ाइल हो सकती है जिसमें VIM के उपयोग के बारे में विवरण होते हैं, जिसमें खोज स्ट्रिंग इतिहास और vim का उपयोग करके खोली गई फ़ाइलों के लिए पथ शामिल होते हैं।
* **Open Office**: हाल की फ़ाइलें।
* **MySQL**: उपयोगकर्ता खातों में _**∼/.mysql\_history**_ फ़ाइल हो सकती है जिसमें MySQL का उपयोग करके निष्पादित क्वेरी होती हैं।
* **Less**: उपयोगकर्ता खातों में _**∼/.lesshst**_ फ़ाइल हो सकती है जिसमें less के उपयोग के बारे में विवरण होते हैं, जिसमें खोज स्ट्रिंग इतिहास और less के माध्यम से निष्पादित शेल कमांड्स शामिल होते हैं।

### USB लॉग

[**usbrip**](https://github.com/snovvcrash/usbrip) एक छोटा सा सॉफ़्टवेयर है जो प्युथन 3 में लिखा गया है और लिनक्स लॉग फ़ाइलों (`/var/log/syslog*` या `/var/log/messages*` जिस पर वितरण निर्भर करता है) को USB घटना इतिहास तालिकाओं का निर्माण करने के लिए पार्स करता है।

यह दिलचस्प होता है कि **कौन से USB का उपयोग किया गया है** और यह और अधिक उपयोगी होगा अगर आपके पास एक अधिकृत USB सूची हो जिसमें "उल्लंघन घटनाएं" (उस सूची में नहीं शामिल USB का उपयोग) का पता लगाने के लिए हो।

### स्थापना
```
pip3 install usbrip
usbrip ids download #Download USB ID database
```
### उदाहरण

```markdown
#### Example 1: Collecting System Information

To collect system information in Linux, you can use the following commands:

- `uname -a`: Displays the kernel version, hostname, and other system information.
- `cat /etc/issue`: Shows the distribution and version of Linux.
- `cat /etc/passwd`: Lists all the user accounts on the system.
- `cat /etc/group`: Displays the groups on the system.
- `cat /etc/shadow`: Shows the encrypted passwords of user accounts.

#### Example 2: Analyzing Log Files

To analyze log files in Linux, you can use the following commands:

- `cat /var/log/auth.log`: Displays authentication-related logs.
- `cat /var/log/syslog`: Shows system-wide logs.
- `cat /var/log/apache2/access.log`: Lists Apache access logs.
- `cat /var/log/apache2/error.log`: Displays Apache error logs.
- `cat /var/log/secure`: Shows security-related logs.

#### Example 3: Examining Network Connections

To examine network connections in Linux, you can use the following commands:

- `netstat -tuln`: Lists all the listening ports.
- `netstat -antp`: Displays all the active network connections.
- `lsof -i`: Shows the processes that are using network connections.
- `tcpdump -i eth0`: Captures network traffic on the specified interface.
- `iptables -L`: Lists the firewall rules.

#### Example 4: Recovering Deleted Files

To recover deleted files in Linux, you can use the following commands:

- `extundelete /dev/sda1 --restore-all`: Recovers all deleted files on the specified partition.
- `photorec /dev/sda1`: Recovers various file types, including documents, photos, and videos.
- `foremost -t all -i /dev/sda1 -o /recovery`: Recovers files based on their headers and footers.
- `scalpel /dev/sda1 -o /recovery`: Carves files based on their file signatures.
- `testdisk /dev/sda1`: Recovers lost partitions and repairs damaged file systems.
```

```html
#### उदाहरण 1: सिस्टम सूचना का संग्रहण

लिनक्स में सिस्टम सूचना को संग्रहित करने के लिए, आप निम्नलिखित कमांड का उपयोग कर सकते हैं:

- `uname -a`: कर्नल संस्करण, होस्टनाम और अन्य सिस्टम सूचना प्रदर्शित करता है।
- `cat /etc/issue`: लिनक्स का वितरण और संस्करण दिखाता है।
- `cat /etc/passwd`: सिस्टम पर सभी उपयोगकर्ता खाते की सूची प्रदर्शित करता है।
- `cat /etc/group`: सिस्टम पर समूहों को प्रदर्शित करता है।
- `cat /etc/shadow`: उपयोगकर्ता खातों के एन्क्रिप्टेड पासवर्ड प्रदर्शित करता है।

#### उदाहरण 2: लॉग फ़ाइलों का विश्लेषण

लिनक्स में लॉग फ़ाइलों का विश्लेषण करने के लिए, आप निम्नलिखित कमांड का उपयोग कर सकते हैं:

- `cat /var/log/auth.log`: प्रमाणीकरण संबंधित लॉग प्रदर्शित करता है।
- `cat /var/log/syslog`: सिस्टम-व्यापक लॉग प्रदर्शित करता है।
- `cat /var/log/apache2/access.log`: एपाच एक्सेस लॉग प्रदर्शित करता है।
- `cat /var/log/apache2/error.log`: एपाच त्रुटि लॉग प्रदर्शित करता है।
- `cat /var/log/secure`: सुरक्षा संबंधित लॉग प्रदर्शित करता है।

#### उदाहरण 3: नेटवर्क कनेक्शन की जांच

लिनक्स में नेटवर्क कनेक्शन की जांच करने के लिए, आप निम्नलिखित कमांड का उपयोग कर सकते हैं:

- `netstat -tuln`: सभी सुनने वाले पोर्टों की सूची प्रदर्शित करता है।
- `netstat -antp`: सभी सक्रिय नेटवर्क कनेक्शन प्रदर्शित करता है।
- `lsof -i`: नेटवर्क कनेक्शन का उपयोग कर रहे प्रक्रियाओं को प्रदर्शित करता है।
- `tcpdump -i eth0`: निर्दिष्ट इंटरफ़ेस पर नेटवर्क ट्रैफ़िक को कैप्चर करता है।
- `iptables -L`: फ़ायरवॉल नियमों की सूची प्रदर्शित करता है।

#### उदाहरण 4: हटाए गए फ़ाइलों की पुनर्प्राप्ति

लिनक्स में हटाए गए फ़ाइलों की पुनर्प्राप्ति करने के लिए, आप निम्नलिखित कमांड का उपयोग कर सकते हैं:

- `extundelete /dev/sda1 --restore-all`: निर्दिष्ट पार्टीशन पर सभी हटाए गए फ़ाइलों की पुनर्प्राप्ति करता है।
- `photorec /dev/sda1`: दस्तावेज़, फ़ोटो और वीडियो जैसे विभिन्न फ़ाइल प्रकारों की पुनर्प्राप्ति करता है।
- `foremost -t all -i /dev/sda1 -o /recovery`: हेडर और फ़ुटर के आधार पर फ़ाइलों की पुनर्प्राप्ति करता है।
- `scalpel /dev/sda1 -o /recovery`: फ़ाइल हस्ताक्षर के आधार पर फ़ाइलों को अलग करता है।
- `testdisk /dev/sda1`: खोए गए पार्टीशनों की पुनर्प्राप्ति करता है और क्षतिग्रस्त फ़ाइल सिस्टम को मरम्मत करता है।
```

```html
#### उदाहरण 1: सिस्टम सूचना का संग्रहण

लिनक्स में सिस्टम सूचना को संग्रहित करने के लिए, आप निम्नलिखित कमांड का उपयोग कर सकते हैं:

- `uname -a`: कर्नल संस्करण, होस्टनाम और अन्य सिस्टम सूचना प्रदर्शित करता है।
- `cat /etc/issue`: लिनक्स का वितरण और संस्करण दिखाता है।
- `cat /etc/passwd`: सिस्टम पर सभी उपयोगकर्ता खाते की सूची प्रदर्शित करता है।
- `cat /etc/group`: सिस्टम पर समूहों को प्रदर्शित करता है।
- `cat /etc/shadow`: उपयोगकर्ता खातों के एन्क्रिप्टेड पासवर्ड प्रदर्शित करता है।

#### उदाहरण 2: लॉग फ़ाइलों का विश्लेषण

लिनक्स में लॉग फ़ाइलों का विश्लेषण करने के लिए, आप निम्नलिखित कमांड का उपयोग कर सकते हैं:

- `cat /var/log/auth.log`: प्रमाणीकरण संबंधित लॉग प्रदर्शित करता है।
- `cat /var/log/syslog`: सिस्टम-व्यापक लॉग प्रदर्शित करता है।
- `cat /var/log/apache2/access.log`: एपाच एक्सेस लॉग प्रदर्शित करता है।
- `cat /var/log/apache2/error.log`: एपाच त्रुटि लॉग प्रदर्शित करता है।
- `cat /var/log/secure`: सुरक्षा संबंधित लॉग प्रदर्शित करता है।

#### उदाहरण 3: नेटवर्क कनेक्शन की जांच

लिनक्स में नेटवर्क कनेक्शन की जांच करने के लिए, आप निम्नलिखित कमांड का उपयोग कर सकते हैं:

- `netstat -tuln`: सभी सुनने वाले पोर्टों की सूची प्रदर्शित करता है।
- `netstat -antp`: सभी सक्रिय नेटवर्क कनेक्शन प्रदर्शित करता है।
- `lsof -i`: नेटवर्क कनेक्शन का उपयोग कर रहे प्रक्रियाओं को प्रदर्शित करता है।
- `tcpdump -i eth0`: निर्दिष्ट इंटरफ़ेस पर नेटवर्क ट्रैफ़िक को कैप्चर करता है।
- `iptables -L`: फ़ायरवॉल नियमों की सूची प्रदर्शित करता है।

#### उदाहरण 4: हटाए गए फ़ाइलों की पुनर्प्राप्ति

लिनक्स में हटाए गए फ़ाइलों की पुनर्प्राप्ति करने के लिए, आप निम्नलिखित कमांड का उपयोग कर सकते हैं:

- `extundelete /dev/sda1 --restore-all`: निर्दिष्ट पार्टीशन पर सभी हटाए गए फ़ाइलों की पुनर्प्राप्ति करता है।
- `photorec /dev/sda1`: दस्तावेज़, फ़ोटो और वीडियो जैसे विभिन्न फ़ाइल प्रकारों की पुनर्प्राप्ति करता है।
- `foremost -t all -i /dev/sda1 -o /recovery`: हेडर और फ़ुटर के आधार पर फ़ाइलों की पुनर्प्राप्ति करता है।
- `scalpel /dev/sda1 -o /recovery`: फ़ाइल हस्ताक्षर के आधार पर फ़ाइलों को अलग करता ह
```
usbrip events history #Get USB history of your curent linux machine
usbrip events history --pid 0002 --vid 0e0f --user kali #Search by pid OR vid OR user
#Search for vid and/or pid
usbrip ids download #Downlaod database
usbrip ids search --pid 0002 --vid 0e0f #Search for pid AND vid
```
अधिक उदाहरण और जानकारी गिटहब में मौजूद है: [https://github.com/snovvcrash/usbrip](https://github.com/snovvcrash/usbrip)

<figure><img src="../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उपयोग करके आसानी से वर्कफ़्लो बनाएं और संचालित करें, जो दुनिया के सबसे उन्नत समुदाय उपकरणों द्वारा संचालित होते हैं।\
आज ही पहुंच प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## उपयोगकर्ता खातों और लॉगऑन गतिविधियों की समीक्षा करें

अनैतिक नाम या खाते की जांच करें जो अनुमति द्वारा नहीं बनाए गए हो सकते हैं और जिन्हें ज्ञात अनधिकृत घटनाओं के करीब बनाया गया हो सकता है। साथ ही, संभावित सुडो ब्रूट-फ़ोर्स हमलों की जांच करें।\
इसके अलावा, उपयोगकर्ताओं को दिए गए अप्रत्याशित विशेषाधिकारों की जांच करें, जैसे _**/etc/sudoers**_ और _**/etc/groups**_।\
अंत में, कोई पासवर्ड या आसानी से अनुमान लगाए जाने वाले पासवर्ड के साथ खातों की जांच करें।

## फ़ाइल सिस्टम की जांच करें

फ़ाइल सिस्टम डेटा संरचनाएं एक मैलवेयर घटना से संबंधित बहुत सारी **जानकारी** प्रदान कर सकती हैं, जिसमें घटनाओं का **समय** और मैलवेयर की **वास्तविक सामग्री** शामिल हो सकती है।\
मैलवेयर अब फ़ाइल सिस्टम विश्लेषण को रोकने के लिए डिज़ाइन किया जा रहा है। कुछ मैलवेयर अवैध फ़ाइलों पर तारीख-समय चिह्नों को संशोधित करते हैं ताकि उन्हें टाइमलाइन विश्लेषण के साथ ढूंढ़ना कठिन हो। अन्य दुष्ट कोड केवल कुछ जानकारी को मेमोरी में संग्रहित करने के लिए डिज़ाइन किए जाते हैं ताकि फ़ाइल सिस्टम में डेटा की मात्रा को कम किया जा सके।\
इस तरह की एंटी-फ़ोरेंसिक तकनीकों का सामना करने के लिए, फ़ाइल सिस्टम तारीख-समय चिह्नों के टाइमलाइन विश्लेषण और मैलवेयर के सामान्य स्थानों में संग्रहित फ़ाइलों की जांच करने के लिए **सतर्कता से ध्यान देना** आवश्यक है।

* **ऑटोप्सी** का उपयोग करके आप संदिग्ध गतिविधि का पता लगाने के लिए घटनाओं का टाइमलाइन देख सकते हैं। आप सीधे **स्लूथ किट** के `mactime` फ़ीचर का उपयोग भी कर सकते हैं।
* **$PATH** में अप्रत्याशित स्क्रिप्ट्स की जांच करें (कुछ sh या php स्क्रिप्ट्स हो सकते हैं?)
* `/dev` में फ़ाइलें पहले विशेष फ़ाइलें थीं, आप यहां मैलवेयर से संबंधित गैर-विशेष फ़ाइलें भी पा सकते हैं।
* असामान्य या **छिपी हुई फ़ाइलें** और **निर्देशिकाएं** जैसे " .. " (डॉट डॉट स्पेस) या " ..^G " (डॉट डॉट कंट्रोल-जी)
* सिस्टम पर /bin/bash की setuid प्रतियां `find / -user root -perm -04000 –print`
* बड़ी संख्या में हटाई गई फ़ाइलों के **डिलीटेड इनोड के दिनांक-समय चिह्नों की समीक्षा** करें, जो किसी रूटकिट या ट्रोजनाइज़्ड सेवा के स्थापना जैसी दुष्ट गतिविधि की संकेत हो सकती है।
* क्योंकि इनोड्स अगले उपलब्ध आधार पर आवंटित किए जाते हैं, इसलिए सिस्टम पर एक ही समय पर रखे गए दुष्ट फ़ाइलों को संख्याबद्ध इनोड्स के रूप में सौंपा जा सकता है। इसलिए, जब मैलवेयर का एक घटक पता लगाया जाता है, तो पड़ोसी इनोड्स की जांच करना उपयोगी हो सकता है।
* _/bin_ या _/sbin_ जैसे निर्देशिकाओं की भी जांच करें क्योंकि नई या संशोधित फ़ाइलों की **संशोधित और या बदली गई समय** दिलचस्प हो सकती है।
* एक निर्देशिका के फ़ाइलों और फ़ोल्डरों को निर्माण तिथि के आधार पर देखना दिखाई देता है बजाय वर्णानुक्रमिक रूप से, जिससे पता चलता है कि कौन सी फ़ाइलें या फ़ोल्डर नवीनतम हैं (आमतौर पर अंतिम वाले होते हैं)।

आप `ls -laR --sort=time /bin` का उपयोग करके एक फ़ोल्डर की सबसे हाल की फ़ाइलें देख सकते हैं।\
आप `ls -lai /bin |sort -n` का उपयोग करके एक फ़ोल्डर के भीतर की फ़ाइलों के इनोड्स की जांच कर सकते हैं।

{% hint style="info" %}
ध्यान दें कि एक **हमलावर** फ़ाइल को **समय** को **संशोधित** करके **फ़ाइलें वास्तविक** दिखा सकता है, लेकिन वह **इन
```bash
git diff --no-index --diff-filter=A _openwrt1.extracted/squashfs-root/ _openwrt2.extracted/squashfs-root/
```
#### संशोधित सामग्री खोजें

To find modified content in Linux, you can use the `find` command along with the `-mtime` option. The `-mtime` option allows you to specify a time range to search for modified files.

For example, to find files modified within the last 24 hours, you can use the following command:

```bash
find /path/to/directory -mtime 0
```

This will display a list of files that have been modified within the last 24 hours in the specified directory.

You can also use the `-mmin` option to search for files modified within a specific number of minutes. For example, to find files modified within the last 30 minutes, you can use the following command:

```bash
find /path/to/directory -mmin -30
```

This will display a list of files that have been modified within the last 30 minutes in the specified directory.

By using the `find` command with the appropriate options, you can easily identify modified content in Linux for forensic analysis.
```bash
git diff --no-index --diff-filter=M _openwrt1.extracted/squashfs-root/ _openwrt2.extracted/squashfs-root/ | grep -E "^\+" | grep -v "Installed-Time"
```
#### हटाए गए फ़ाइलें खोजें

To find deleted files on a Linux system, you can use various techniques:

1. **File Recovery Tools**: Use tools like `extundelete`, `foremost`, or `scalpel` to recover deleted files from the file system.

2. **File System Analysis**: Analyze the file system metadata to identify deleted files. Tools like `fls` and `icat` from The Sleuth Kit can be used for this purpose.

3. **Carving**: Use file carving techniques to search for deleted files based on their file signatures. Tools like `photorec` and `testdisk` can be helpful in this process.

4. **Memory Analysis**: Analyze the system's memory to identify deleted files that might still be present in memory. Tools like `Volatility` can be used for this purpose.

Remember to perform these actions on a forensic image or a copy of the system to avoid modifying the original evidence.
```bash
git diff --no-index --diff-filter=A _openwrt1.extracted/squashfs-root/ _openwrt2.extracted/squashfs-root/
```
#### अन्य फ़िल्टर

**`-diff-filter=[(A|C|D|M|R|T|U|X|B)…​[*]]`**

केवल वे फ़ाइलें चुनें जो जोड़ी गई (`A`), कॉपी की गई (`C`), हटाई गई (`D`), संशोधित (`M`), नाम बदली गई (`R`), और उनके प्रकार (यानी नियमित फ़ाइल, सिंबलिंक, सबमॉड्यूल, ... ) में परिवर्तन हुआ (`T`), असंगठित (`U`), अज्ञात (`X`), या उनके संयोजन को तोड़ दिया गया (`B`)। इस फ़िल्टर अक्षरों का कोई भी संयोजन (इसमें कोई भी नहीं) उपयोग किया जा सकता है। जब संयोजन में इस संयोजन में अन्य मापदंडों के अनुरूप कोई फ़ाइल मौजूद होती है, तो सभी पथ चयनित होते हैं; यदि कोई ऐसी फ़ाइल नहीं है जो अन्य मापदंडों के अनुरूप हो, तो कुछ भी चयनित नहीं होता है।

इसके अलावा, **ये अपरकेस अक्षर बाहर करने के लिए डाउनकेस किए जा सकते हैं**। उदा। `--diff-filter=ad` जोड़ी गई और हटाई गई पथों को छोड़ता है।

ध्यान दें कि सभी अंतर विशेषणों में सभी प्रकार शामिल नहीं हो सकते हैं। उदाहरण के लिए, इंडेक्स से कार्य वृक्षा तक के अंतर में कभी भी जोड़ी गई प्रविष्टियाँ नहीं हो सकती हैं (क्योंकि अंतर में शामिल पथों का सेट इंडेक्स द्वारा सीमित होता है)। उसी तरह, कॉपी और नाम बदली गई प्रविष्टियाँ उपलब्ध नहीं हो सकती हैं यदि उन प्रकारों के लिए पहचान अक्षम है।

## संदर्भ

* [https://cdn.ttgtmedia.com/rms/security/Malware%20Forensics%20Field%20Guide%20for%20Linux%20Systems\_Ch3.pdf](https://cdn.ttgtmedia.com/rms/security/Malware%20Forensics%20Field%20Guide%20for%20Linux%20Systems\_Ch3.pdf)
* [https://www.plesk.com/blog/featured/linux-logs-explained/](https://www.plesk.com/blog/featured/linux-logs-explained/)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **हैकट्रिक्स में विज्ञापित** देखना चाहते हैं? या क्या आप **PEASS के नवीनतम संस्करण का उपयोग करना चाहते हैं या HackTricks को पीडीएफ़ में डाउनलोड करना चाहते हैं**? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**

**अपने हैकिंग ट्रिक्स को** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में पीआर जमा करके अपने हैकिंग ट्रिक्स साझा करें।**

</details>

<figure><img src="../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उपयोग करें और आसानी से बनाएं और **स्वचालित कार्यप्रवाह** बनाएं जो दुनिया के **सबसे उन्नत** समुदाय उपकरणों द्वारा संचालित होता है।\
आज ही पहुंच प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}
