# Docker Socket का दुरुपयोग करके Privilege Escalation

<details>

<summary><strong>Learn AWS hacking from zero to hero with</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स शेयर करें.

</details>

कुछ अवसरों पर आपके पास केवल **docker socket तक पहुंच** होती है और आप इसका उपयोग **privileges बढ़ाने** के लिए करना चाहते हैं। कुछ क्रियाएं बहुत संदिग्ध हो सकती हैं और आप उनसे बचना चाह सकते हैं, इसलिए यहां आपको विभिन्न फ्लैग्स मिलेंगे जो privileges बढ़ाने के लिए उपयोगी हो सकते हैं:

### माउंट के माध्यम से

आप **filesystem** के विभिन्न भागों को रूट के रूप में चल रहे कंटेनर में **माउंट** कर सकते हैं और उन्हें **एक्सेस** कर सकते हैं।\
आप कंटेनर के अंदर privileges बढ़ाने के लिए माउंट का भी **दुरुपयोग** कर सकते हैं।

* **`-v /:/host`** -> होस्ट filesystem को कंटेनर में माउंट करें ताकि आप होस्ट filesystem को **पढ़ सकें।**
* यदि आप कंटेनर में होते हुए भी होस्ट की तरह महसूस करना चाहते हैं तो आप निम्नलिखित फ्लैग्स का उपयोग करके अन्य रक्षा तंत्रों को अक्षम कर सकते हैं:
* `--privileged`
* `--cap-add=ALL`
* `--security-opt apparmor=unconfined`
* `--security-opt seccomp=unconfined`
* `-security-opt label:disable`
* `--pid=host`
* `--userns=host`
* `--uts=host`
* `--cgroupns=host`
* \*\*`--device=/dev/sda1 --cap-add=SYS_ADMIN --security-opt apparmor=unconfined` \*\* -> यह पिछली विधि के समान है, लेकिन यहां हम **डिवाइस डिस्क को माउंट** कर रहे हैं। फिर, कंटेनर के अंदर `mount /dev/sda1 /mnt` चलाएं और आप `/mnt` में **होस्ट filesystem** को **एक्सेस** कर सकते हैं
* माउंट करने के लिए `</dev/sda1>` डिवाइस खोजने के लिए होस्ट में `fdisk -l` चलाएं
* **`-v /tmp:/host`** -> यदि किसी कारण से आप **केवल कुछ निर्देशिका** को होस्ट से माउंट कर सकते हैं और आपके पास होस्ट के अंदर एक्सेस है। इसे माउंट करें और माउंट की गई निर्देशिका में **suid** के साथ एक **`/bin/bash`** बनाएं ताकि आप इसे होस्ट से चला सकें और रूट तक एस्केलेट कर सकें।

{% hint style="info" %}
ध्यान दें कि शायद आप `/tmp` फोल्डर को माउंट नहीं कर सकते हैं लेकिन आप एक **अलग लिखने योग्य फोल्डर** को माउंट कर सकते हैं। लिखने योग्य निर्देशिकाओं को खोजने के लिए आप उपयोग कर सकते हैं: `find / -writable -type d 2>/dev/null`

**ध्यान दें कि लिनक्स मशीन की सभी निर्देशिकाएं suid बिट का समर्थन नहीं करेंगी!** suid बिट का समर्थन करने वाली निर्देशिकाओं की जांच करने के लिए `mount | grep -v "nosuid"` चलाएं। उदाहरण के लिए आमतौर पर `/dev/shm`, `/run`, `/proc`, `/sys/fs/cgroup` और `/var/lib/lxcfs` suid बिट का समर्थन नहीं करते हैं।

यह भी ध्यान दें कि यदि आप **`/etc`** या किसी अन्य फोल्डर को माउंट कर सकते हैं जिसमें कॉन्फ़िगरेशन फाइलें होती हैं, तो आप उन्हें डॉकर कंटेनर से रूट के रूप में बदल सकते हैं ताकि होस्ट में उनका **दुरुपयोग** करके privileges बढ़ा सकें (शायद `/etc/shadow` में संशोधन करके)
{% endhint %}

### कंटेनर से बचना

* **`--privileged`** -> इस फ्लैग के साथ आप [कंटेनर से सभी अलगाव को हटा देते हैं](docker-privileged.md#what-affects)। रूट के रूप में privileged कंटेनर से बचने की तकनीकों की जांच करें [docker-breakout-privilege-escalation/#automatic-enumeration-and-escape](docker-breakout-privilege-escalation/#automatic-enumeration-and-escape)।
* **`--cap-add=<CAPABILITY/ALL> [--security-opt apparmor=unconfined] [--security-opt seccomp=unconfined] [-security-opt label:disable]`** -> [capabilities का दुरुपयोग करके एस्केलेट करने](../linux-capabilities.md) के लिए, **कंटेनर को वह क्षमता प्रदान करें** और अन्य सुरक्षा विधियों को अक्षम करें जो एक्सप्लॉइट को काम करने से रोक सकते हैं।

### Curl

इस पृष्ठ पर हमने docker फ्लैग्स का उपयोग करके privileges बढ़ाने के तरीकों पर चर्चा की है, आप इन तरीकों का दुरुपयोग करने के **तरीके curl** कमांड का उपयोग करके पृष्ठ में पा सकते हैं:

{% content-ref url="authz-and-authn-docker-access-authorization-plugin.md" %}
[authz-and-authn-docker-access-authorization-plugin.md](authz-and-authn-docker-access-authorization-plugin.md)
{% endcontent-ref %}

<details>

<summary><strong>Learn AWS hacking from zero to hero with</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स शेयर करें.

</details>
