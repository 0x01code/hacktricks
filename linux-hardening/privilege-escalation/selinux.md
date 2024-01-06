<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) को **फॉलो करें**.
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स शेयर करें.

</details>


# SELinux in Containers

[SELinux](https://www.redhat.com/en/blog/latest-container-exploit-runc-can-be-blocked-selinux) एक **लेबलिंग सिस्टम** है। हर **प्रोसेस** और हर **फाइल सिस्टम ऑब्जेक्ट** का एक **लेबल** होता है। SELinux नीतियाँ नियम निर्धारित करती हैं कि एक **प्रोसेस लेबल क्या कर सकता है अन्य सभी लेबल्स के साथ** सिस्टम पर।

कंटेनर इंजन **कंटेनर प्रोसेसेस को एक अकेले सीमित SELinux लेबल के साथ लॉन्च करते हैं**, आमतौर पर `container_t`, और फिर कंटेनर के अंदर को `container_file_t` लेबल के साथ सेट करते हैं। SELinux नीति नियम मूल रूप से कहते हैं कि **`container_t` प्रोसेसेस केवल `container_file_t` लेबल वाली फाइलों को पढ़/लिख/निष्पादित कर सकते हैं**। यदि कोई कंटेनर प्रोसेस कंटेनर से बाहर निकलता है और होस्ट पर सामग्री लिखने का प्रयास करता है, तो लिनक्स कर्नेल पहुँच को अस्वीकार कर देता है और केवल कंटेनर प्रोसेस को `container_file_t` लेबल वाली सामग्री पर लिखने की अनुमति देता है।
```shell
$ podman run -d fedora sleep 100
d4194babf6b877c7100e79de92cd6717166f7302113018686cea650ea40bd7cb
$ podman top -l label
LABEL
system_u:system_r:container_t:s0:c647,c780
```
# SELinux उपयोगकर्ता

सामान्य Linux उपयोगकर्ताओं के अलावा SELinux उपयोगकर्ता भी होते हैं। SELinux उपयोगकर्ता एक SELinux नीति का हिस्सा होते हैं। प्रत्येक Linux उपयोगकर्ता को नीति के अनुसार एक SELinux उपयोगकर्ता से मैप किया जाता है। इससे Linux उपयोगकर्ताओं को SELinux उपयोगकर्ताओं पर लगाए गए प्रतिबंधों और सुरक्षा नियमों और तंत्रों को विरासत में मिलता है।


<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सदस्यता योजनाओं**](https://github.com/sponsors/carlospolop) की जाँच करें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) में या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) **का अनुसरण करें**।
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग तरकीबें साझा करें।

</details>
