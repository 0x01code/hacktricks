# euid, ruid, suid

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)** का** **अनुसरण** करें।**
* **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके साझा करें।**

</details>

**यह पोस्ट** [**https://0xdf.gitlab.io/2022/05/31/setuid-rabbithole.html#testing-on-jail**](https://0xdf.gitlab.io/2022/05/31/setuid-rabbithole.html#testing-on-jail) **से कॉपी की गई है**

## **`*uid`**

* **`ruid`**: यह प्रक्रिया का **वास्तविक उपयोगकर्ता आईडी** है जो प्रक्रिया को शुरू करता है।
* **`euid`**: यह **प्रभावी उपयोगकर्ता आईडी** है, जिसे प्रक्रिया के **प्रिविलेज का निर्धारण करने के लिए सिस्टम देखता है**। अधिकांश मामलों में, `euid` `ruid` के समान होगा, लेकिन SetUID बाइनरी इसका उदाहरण है जहां वे अलग होते हैं। जब एक **SetUID** बाइनरी शुरू होता है, तो **`euid` फ़ाइल के मालिक के लिए सेट होता है**, जिससे इन बाइनरी को कार्य करने की अनुमति मिलती है।
* `suid`: यह **सहेजी गई उपयोगकर्ता आईडी** है, जब एक विशेषाधिकारी प्रक्रिया (अधिकांश मामलों में रूट के रूप में चल रही होती है) को **प्रिविलेज को कम करने** के लिए चाहिए, लेकिन फिर **प्रिविलेज स्थिति में वापस आने** की आवश्यकता होती है।

{% hint style="info" %}
अगर एक **गैर-रूट प्रक्रिया** अपना **`euid` बदलना चाहती है**, तो वह केवल इसे **वर्तमान मानों** के रूप में **`ruid`**, **`euid`**, या **`suid`** में **सेट** कर सकती है।
{% endhint %}

## set\*uid

पहली नजर में, यह आसान है कि सिस्टम कॉल **`setuid`** `ruid` को सेट करेगा। वास्तव में, एक विशेषाधिकारी प्रक्रिया के लिए, यह करता है। लेकिन सामान्य मामले में, यह वास्तव में **`euid` को सेट करता है**। [मैन पेज](https://man7.org/linux/man-pages/man2/setuid.2.html) से:

> setuid() **कॉल करने वाली प्रक्रिया की प्रभावी उपयोगकर्ता आईडी सेट करता है**। यदि कॉल करने वाली प्रक्रिया विशेषाधिकारी है (अधिक निश्चित रूप से: यदि प्रक्रिया में CAP\_SETUID क्षमता है उसके उपयोगकर्ता नेमस्पेस में), तो वास्तविक UID और सहेजी गई सेट-उपयोगकर्ता-आईडी भी सेट हो जाती हैं।

इसलिए, जब आप `setuid(0)` के रूप में रूट के रूप में चल रहे होते हैं, तो यह सभी आईडी को रूट में सेट करता है, और
### **सिस्टम**

`सिस्टम` एक [पूरी तरह से अलग तरीके](https://man7.org/linux/man-pages/man3/system.3.html) से एक नया प्रक्रिया शुरू करने के लिए है। जहां `execve` एक ही प्रक्रिया के स्तर पर काम करता है, **`सिस्टम` `fork` का उपयोग करके एक बच्चा प्रक्रिया बनाता है** और फिर उस बच्चा प्रक्रिया में `execl` का उपयोग करके निष्पादित करता है:

> ```
> execl("/bin/sh", "sh", "-c", command, (char *) NULL);
> ```

`execl` केवल `execve` के चारों ओर एक लपेट है जो स्ट्रिंग तर्कों को `argv` एरे में रूपांतरित करता है और `execve` को बुलाता है। यह महत्वपूर्ण है कि **`सिस्टम` कमांड को बुलाने के लिए `श` का उपयोग करता है**।

### sh और bash SUID <a href="#sh-and-bash-suid" id="sh-and-bash-suid"></a>

**`बैश`** में एक **`-p विकल्प`** होता है, जिसे [मैन पेज](https://linux.die.net/man/1/bash) इस तरह से वर्णित करता है:

> _विशेषाधिकारित_ मोड को सक्षम करें। इस मोड में, **$ENV** और **$BASH\_ENV** फ़ाइलें प्रोसेस की स्थिति में नहीं होती हैं, शेल फ़ंक्शन वातानुकूलन से प्राप्त नहीं होती हैं, और यदि वे पर्याप्तता में हों, तो **SHELLOPTS**, **BASHOPTS**, **CDPATH**, और **GLOBIGNORE** चरित्रित करने वाले चरित्रों को अनदेखा कर दिया जाता है। यदि शेल वास्तविक उपयोगकर्ता (समूह) आईडी के समान नहीं होती है और **-p विकल्प नहीं दिया गया हो**, तो ये कार्रवाई ली जाती है और **प्रभावी उपयोगकर्ता आईडी को वास्तविक उपयोगकर्ता आईडी पर सेट किया जाता है**। यदि **-p** विकल्प **प्रारंभ में दिया जाता है**, तो **प्रभावी उपयोगकर्ता आईडी रीसेट नहीं होती है**। इस विकल्प को बंद करने से प्रभावी उपयोगकर्ता और समूह आईडी वास्तविक उपयोगकर्ता और समूह आईडी पर सेट किए जाते हैं।

संक्षेप में, `-p` के बिना, जब बैश चलाया जाता है, तो `euid` को `ruid` पर सेट किया जाता है। **`-p` इसे रोकता है**।

**`श`** शेल **इस तरह की कोई सुविधा नहीं है**। [मैन पेज](https://man7.org/linux/man-pages/man1/sh.1p.html) में "उपयोगकर्ता आईडी" के बारे में उल्लेख नहीं है, केवल `-i` विकल्प के साथ, जो कहता है:

> \-i यह निर्दिष्ट करें कि शेल इंटरैक्टिव है; नीचे देखें। यदि वास्तविक प्रक्रिया की वास्तविक उपयोगकर्ता आईडी वास्तविक उपयोगकर्ता आईडी के बराबर नहीं होती है या यदि वास्तविक समूह आईडी वास्तविक समूह आईडी के बराबर नहीं होती है, तो कॉल करने वाले प्रक्रिया के लिए -i विकल्प निर्दिष्ट करने को एक त्रुटि मान सकता है।

## परीक्षण

### setuid / system <a href="#setuid--system" id="setuid--system"></a>

इस पृष्ठभूमि के साथ, मैं इस कोड को ले और जेल (HTB) पर क्या होता है, उसे विवरण में देखूंगा:
```c
#define _GNU_SOURCE
#include <stdlib.h>
#include <unistd.h>

int main(void) {
setuid(1000);
system("id");
return 0;
}
```
यह प्रोग्राम NFS पर जेल में कंपाइल किया गया है और SetUID के रूप में सेट किया गया है:
```bash
oxdf@hacky$ gcc a.c -o /mnt/nfsshare/a;
...[snip]...
oxdf@hacky$ chmod 4755 /mnt/nfsshare/a
```
जब रूट के रूप में, मैं इस फ़ाइल को देख सकता हूँ:
```
[root@localhost nfsshare]# ls -l a
-rwsr-xr-x. 1 frank frank 16736 May 30 04:58 a
```
जब मैं इसे nobody के रूप में चलाता हूँ, `id` nobody के रूप में चलता है:
```bash
bash-4.2$ $ ./a
uid=99(nobody) gid=99(nobody) groups=99(nobody) context=system_u:system_r:unconfined_service_t:s0
```
यह प्रोग्राम `ruid` के साथ शुरू होता है जिसका मान 99 (कोई नहीं) है और `euid` के साथ जिसका मान 1000 (फ्रैंक) है। जब यह `setuid` कॉल तक पहुंचता है, तो उनी ही मानों को सेट किया जाता है।

फिर `system` कॉल होता है, और मैं 99 के `uid` को देखने की उम्मीद करूंगा, लेकिन एक `euid` के साथ भी 1000 को देखने की उम्मीद होगी। फिर एक नहीं होता है? समस्या यह है कि इस वितरण में **`sh` को `bash` से संकेतित किया जाता है**:
```
$ ls -l /bin/sh
lrwxrwxrwx. 1 root root 4 Jun 25  2017 /bin/sh -> bash
```
इसलिए `system` `/bin/sh sh -c id` को कॉल करता है, जो कि वास्तव में `/bin/bash bash -c id` है। जब `bash` को बिना `-p` के कॉल किया जाता है, तो यह 99 का `ruid` और 1000 का `euid` देखता है, और `euid` को 99 पर सेट कर देता है।

### setreuid / system <a href="#setreuid--system" id="setreuid--system"></a>

इस सिद्धांत को परीक्षण करने के लिए, मैं `setuid` को `setreuid` से बदलने की कोशिश करूंगा:
```c
#define _GNU_SOURCE
#include <stdlib.h>
#include <unistd.h>

int main(void) {
setreuid(1000, 1000);
system("id");
return 0;
}
```
कंपाइल और अनुमतियाँ:
```
oxdf@hacky$ gcc b.c -o /mnt/nfsshare/b; chmod 4755 /mnt/nfsshare/b
```
अब जेल में, अब `id` 1000 का uid लौटाता है:
```
bash-4.2$ $ ./b
uid=1000(frank) gid=99(nobody) groups=99(nobody) context=system_u:system_r:unconfined_service_t:s0
```
`setreuid` कॉल `ruid` और `euid` दोनों को 1000 पर सेट करता है, इसलिए जब `system` ने `bash` को कॉल किया, वे मिल गए और चीजें फ्रैंक के रूप में जारी रहीं।

### setuid / execve <a href="#setuid--execve" id="setuid--execve"></a>

यदि मेरा उपरोक्त समझ सही है, तो मुझे uids को गलत करने की चिंता नहीं करनी चाहिए, बल्कि `execve` को कॉल करना चाहिए, क्योंकि यह मौजूदा IDs को ले जाएगा। यह काम करेगा, लेकिन यहां छाप हैं। उदाहरण के लिए, सामान्य कोड इस तरह दिख सकता है:
```c
#define _GNU_SOURCE
#include <stdlib.h>
#include <unistd.h>

int main(void) {
setuid(1000);
execve("/usr/bin/id", NULL, NULL);
return 0;
}
```
यदि मैं पर्यावरण को छोड़ दूँ (सरलता के लिए मैं NULL पास कर रहा हूँ), तो मुझे `id` पर पूरा पथ चाहिए होगा। यह काम करता है, और मुझे वही मिलता है जो मैं उम्मीद कर रहा हूँ:
```
bash-4.2$ $ ./c
uid=99(nobody) gid=99(nobody) euid=1000(frank) groups=99(nobody) context=system_u:system_r:unconfined_service_t:s0
```
`[r]uid` 99 है, लेकिन `euid` 1000 है।

इससे शेल प्राप्त करने की कोशिश करने पर, मुझे सतर्क रहना होगा। उदाहरण के लिए, बस `bash` को कॉल करना:
```c
#define _GNU_SOURCE
#include <stdlib.h>
#include <unistd.h>

int main(void) {
setuid(1000);
execve("/bin/bash", NULL, NULL);
return 0;
}
```
मैं उसे कंपाइल करूंगा और इसे SetUID के रूप में सेट कर दूंगा:
```
oxdf@hacky$ gcc d.c -o /mnt/nfsshare/d
oxdf@hacky$ chmod 4755 /mnt/nfsshare/d
```
फिर भी, यह सभी को वापस नोबॉडी के रूप में लौटाएगा:
```
bash-4.2$ $ ./d
bash-4.2$ $ id
uid=99(nobody) gid=99(nobody) groups=99(nobody) context=system_u:system_r:unconfined_service_t:s0
```
यदि यह `setuid(0)` होता, तो यह ठीक काम करता (मान लें कि प्रक्रिया को उसे करने की अनुमति होती है), क्योंकि फिर यह तीनों आईडी को 0 पर बदल देता है। लेकिन एक गैर-रूट उपयोगकर्ता के रूप में, यह केवल `euid` को 1000 (जो पहले से ही था) सेट करता है, और फिर `sh` को कॉल करता है। लेकिन जेल पर `bash` `sh` होता है। और जब `bash` `ruid` 99 और `euid` 1000 के साथ शुरू होता है, तो यह `euid` को फिर से 99 पर छोड़ देगा।

इसे ठीक करने के लिए, मैं `bash -p` कॉल करूंगा:
```c
#define _GNU_SOURCE
#include <stdlib.h>
#include <unistd.h>

int main(void) {
char *const paramList[10] = {"/bin/bash", "-p", NULL};
setuid(1000);
execve(paramList[0], paramList, NULL);
return 0;
}
```
इस बार `euid` मौजूद है:
```
bash-4.2$ $ ./e
bash-4.2$ $ id
uid=99(nobody) gid=99(nobody) euid=1000(frank) groups=99(nobody) context=system_u:system_r:unconfined_service_t:s0
```
या मैं `setuid` के बजाय `setreuid` या `setresuid` को कॉल कर सकता हूँ।

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह।
* प्राप्त करें [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें, [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके।**

</details>
