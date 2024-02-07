# समय नेमस्पेस

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** द्वारा PRs सबमिट करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

## मूल जानकारी

Linux में समय नेमस्पेस को सिस्टम मोनोटोनिक और बूट-टाइम घड़ी के लिए नेमस्पेस ऑफसेट की अनुमति देता है। यह आम तौर पर Linux कंटेनर में तारीख/समय बदलने और चेकपॉइंट या स्नैपशॉट से पुनर्स्थापित होने के बाद घड़ी को समायोजित करने के लिए उपयोग किया जाता है।

## लैब:

### विभिन्न नेमस्पेस बनाएँ

#### CLI
```bash
sudo unshare -T [--mount-proc] /bin/bash
```
यदि आप `--mount-proc` पैरामीटर का उपयोग करते हैं और `/proc` फ़ाइल सिस्टम का एक नया इंस्टेंस माउंट करते हैं, तो आप सुनिश्चित करते हैं कि नया माउंट नेमस्पेस के पास उस नेमस्पेस के लिए विशिष्ट प्रक्रिया सूचना का सटीक और अलग दृश्य है।
```bash
docker run -ti --name ubuntu1 -v /usr:/ubuntu1 ubuntu bash
```
### &#x20;जांचें कि आपकी प्रक्रिया किस नेमस्पेस में है
```bash
ls -l /proc/self/ns/time
lrwxrwxrwx 1 root root 0 Apr  4 21:16 /proc/self/ns/time -> 'time:[4026531834]'
```
### सभी समय नेमस्पेस खोजें

{% code overflow="wrap" %}
```bash
sudo find /proc -maxdepth 3 -type l -name time -exec readlink {} \; 2>/dev/null | sort -u
# Find the processes with an specific namespace
sudo find /proc -maxdepth 3 -type l -name time -exec ls -l  {} \; 2>/dev/null | grep <ns-number>
```
### एक समय नेमस्पेस के अंदर जाएं

{% endcode %}
```bash
nsenter -T TARGET_PID --pid /bin/bash
```
आप **केवल रूट यदि आप अन्य प्रक्रिया नेमस्पेस में प्रवेश कर सकते हैं**। और आप **बिना एक डिस्क्रिप्टर के** (जैसे `/proc/self/ns/net`) **अन्य नेमस्पेस में प्रवेश नहीं कर सकते**।
