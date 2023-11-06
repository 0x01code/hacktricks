<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)

- प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **फॉलो** करें मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपने हैकिंग ट्रिक्स साझा करें, [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके।**

</details>


# कंटेनर में SELinux

[SELinux](https://www.redhat.com/en/blog/latest-container-exploit-runc-can-be-blocked-selinux) एक **लेबलिंग** **सिस्टम** है। हर **प्रक्रिया** और हर **फ़ाइल** सिस्टम ऑब्जेक्ट का एक **लेबल** होता है। SELinux नीतियाँ निर्धारित करती हैं कि एक **प्रक्रिया लेबल सभी अन्य लेबलों के साथ क्या कर सकती है**।

कंटेनर इंजन्स **कंटेनर प्रक्रियाओं को एकमात्र सीमित SELinux लेबल** के साथ लॉन्च करते हैं, आमतौर पर `container_t`, और फिर कंटेनर को `container_file_t` लेबल देते हैं। SELinux नीति नियमों का मतलब है कि **`container_t` प्रक्रियाएं केवल `container_file_t` लेबल वाली फ़ाइलों को पढ़ सकती/लिख सकती/चला सकती हैं**। यदि कोई कंटेनर प्रक्रिया कंटेनर से बाहर निकलती है और होस्ट पर सामग्री में लिखने का प्रयास करती है, तो लिनक्स कर्नल उपयोग अस्वीकार करता है और केवल कंटेनर प्रक्रिया को `container_file_t` लेबल वाली सामग्री में लिखने की अनुमति देता है।
```shell
$ podman run -d fedora sleep 100
d4194babf6b877c7100e79de92cd6717166f7302113018686cea650ea40bd7cb
$ podman top -l label
LABEL
system_u:system_r:container_t:s0:c647,c780
```
# SELinux उपयोगकर्ता

नियमित Linux उपयोगकर्ताओं के अलावा SELinux उपयोगकर्ताएँ होती हैं। SELinux उपयोगकर्ताएँ SELinux नीति का हिस्सा होती हैं। पॉलिसी के हिस्सा के रूप में हर Linux उपयोगकर्ता को एक SELinux उपयोगकर्ता से मैप किया जाता है। इससे Linux उपयोगकर्ताओं को SELinux उपयोगकर्ताओं पर लगाए गए प्रतिबंध और सुरक्षा नियम और यंत्रों को अनुग्रहित करने की अनुमति मिलती है।
