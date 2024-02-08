# UTS नेमस्पेस

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें** द्वारा PRs सबमिट करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

## मूल जानकारी

एक UTS (UNIX Time-Sharing System) नेमस्पेस एक लिनक्स कर्नेल सुविधा है जो दो सिस्टम पहचानकर्ताओं को अलग करने की प्रदान करती है: **होस्टनाम** और **NIS** (नेटवर्क सूचना सेवा) डोमेन नाम। यह अलगाव उत्पन्न करता है कि प्रत्येक UTS नेमस्पेस के पास अपना **स्वतंत्र होस्टनाम और NIS डोमेन नाम** हो, जो विशेष रूप से उपकरणीकरण स्थितियों में उपयोगी है जहां प्रत्येक कंटेनर एक अलग सिस्टम के रूप में प्रकट होना चाहिए।

### यह कैसे काम करता है:

1. जब एक नया UTS नेमस्पेस बनाया जाता है, तो यह अपने **मूल नेमस्पेस से होस्टनाम और NIS डोमेन नाम की प्रतिलिपि के साथ शुरू होता है**। इसका मतलब है कि, निर्माण के समय, नया नेमस्पेस **अपने मूल सिद्धांतों को साझा करता है जैसे कि इसके मूल**। हालांकि, नेमस्पेस के भीतर होस्टनाम या NIS डोमेन नाम में कोई भी भविष्यवाणी परिवर्तन अन्य नेमस्पेस पर प्रभाव नहीं डालेगा।
2. UTS नेमस्पेस के भीतर प्रक्रियाएं **`sethostname()` और `setdomainname()` सिस्टम कॉल का उपयोग करके होस्टनाम और NIS डोमेन नाम को बदल सकती हैं**। ये परिवर्तन नेमस्पेस के लिए स्थानीय हैं और अन्य नेमस्पेस या मुख्य सिस्टम पर प्रभाव नहीं डालते।
3. प्रक्रियाएं `setns()` सिस्टम कॉल का उपयोग करके नेमस्पेस के बीच ले जा सकती हैं या `unshare()` या `clone()` सिस्टम कॉल का उपयोग करके `CLONE_NEWUTS` ध्वज के साथ नए नेमस्पेस बना सकती हैं। जब कोई प्रक्रिया नए नेमस्पेस में जाती है या एक नया नेमस्पेस बनाती है, तो वह उस नेमस्पेस से जुड़े होस्टनाम और NIS डोमेन नाम का उपयोग करना शुरू करेगी।

## लैब:

### विभिन्न नेमस्पेस बनाएं

#### CLI
```bash
sudo unshare -u [--mount-proc] /bin/bash
```
यदि आप `--mount-proc` पैरामीटर का उपयोग करते हैं तो `/proc` फ़ाइल सिस्टम का एक नया इंस्टेंस माउंट करके, आप सुनिश्चित करते हैं कि नया माउंट नेमस्पेस के लिए प्रक्रिया सूचना का सटीक और अलगजा दृश्य है।
```bash
docker run -ti --name ubuntu1 -v /usr:/ubuntu1 ubuntu bash
```
### &#x20;जांचें कि आपकी प्रक्रिया किस नेमस्पेस में है
```bash
ls -l /proc/self/ns/uts
lrwxrwxrwx 1 root root 0 Apr  4 20:49 /proc/self/ns/uts -> 'uts:[4026531838]'
```
### सभी UTS नेमस्पेस खोजें

{% code overflow="wrap" %}
```bash
sudo find /proc -maxdepth 3 -type l -name uts -exec readlink {} \; 2>/dev/null | sort -u
# Find the processes with an specific namespace
sudo find /proc -maxdepth 3 -type l -name uts -exec ls -l  {} \; 2>/dev/null | grep <ns-number>
```
{% endcode %}

### एक UTS नेमस्पेस के अंदर जाएं
```bash
nsenter -u TARGET_PID --pid /bin/bash
```
आप केवल **रूट होने पर** **दूसरे प्रक्रिया नेमस्पेस में प्रवेश** कर सकते हैं। और आप **उसमें प्रवेश नहीं** कर सकते **बिना एक डिस्क्रिप्टर** के जो इसे पॉइंट करता है (जैसे `/proc/self/ns/uts`).

### होस्टनाम बदलें
```bash
unshare -u /bin/bash
hostname newhostname # Hostname won't be changed inside the host UTS ns
```
## संदर्भ
* [https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory](https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory)

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** द्वारा PRs सबमिट करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>
