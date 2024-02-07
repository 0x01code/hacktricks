<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** का** **अनुसरण** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PR जमा करके।

</details>


# कंटेनर में SELinux

[रेडहैट डॉक्स से परिचय और उदाहरण](https://www.redhat.com/sysadmin/privileged-flag-container-engines)

[SELinux](https://www.redhat.com/en/blog/latest-container-exploit-runc-can-be-blocked-selinux) एक **लेबलिंग सिस्टम** है। प्रत्येक **प्रक्रिया** और प्रत्येक **फ़ाइल** सिस्टम ऑब्जेक्ट का एक **लेबल** होता है। SELinux नीतियाँ प्रक्रिया लेबल के बारे में नियम परिभाषित करती हैं कि सिस्टम पर अन्य सभी लेबल के साथ प्रक्रिया लेबल को क्या करने की अनुमति है।

कंटेनर इंजन **कंटेनर प्रक्रियाएँ एक एकल सीमित SELinux लेबल के साथ लॉन्च करते हैं**, आम तौर पर `container_t`, और फिर कंटेनर को `container_file_t` लेबल वाले कंटेनर के अंदर सेट करते हैं। SELinux नीति नियम बुनियादी रूप से कहते हैं कि **`container_t` प्रक्रियाएँ केवल `container_file_t` लेबल वाली फ़ाइलों को पढ़ सकती हैं/लिख सकती हैं/क्रियान्वित कर सकती हैं**। यदि कोई कंटेनर प्रक्रिया कंटेनर से बाहर निकलती है और होस्ट पर सामग्री में लिखने का प्रयास करती है, तो लिनक्स कर्नेल एक्सेस नकारता है और केवल कंटेनर प्रक्रिया को `container_file_t` लेबल वाली सामग्री में लिखने की अनुमति देता है।
```shell
$ podman run -d fedora sleep 100
d4194babf6b877c7100e79de92cd6717166f7302113018686cea650ea40bd7cb
$ podman top -l label
LABEL
system_u:system_r:container_t:s0:c647,c780
```
# SELinux उपयोगकर्ता

सामान्य Linux उपयोगकर्ताओं के अतिरिक्त SELinux उपयोगकर्ता होते हैं। SELinux उपयोगकर्ता SELinux नीति का हिस्सा होते हैं। प्रत्येक Linux उपयोगकर्ता नीति के हिस्से के रूप में एक SELinux उपयोगकर्ता से मैप किया जाता है। यह लिनक्स उपयोगकर्ताओं को SELinux उपयोगकर्ताओं पर लगाए गए प्रतिबंध और सुरक्षा नियम और तंत्रों को विरासत में लेने देता है।
