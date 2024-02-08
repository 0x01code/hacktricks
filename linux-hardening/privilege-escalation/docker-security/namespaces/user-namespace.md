# उपयोगकर्ता नेमस्पेस

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह **The PEASS Family** की खोज करें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें** द्वारा PRs सबमिट करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos को।

</details>

## मूल जानकारी

एक उपयोगकर्ता नेमस्पेस एक लिनक्स कर्नेल सुविधा है जो **उपयोगकर्ता और समूह आईडी मैपिंग की अलगाव प्रदान करती है**, जिससे प्रत्येक उपयोगकर्ता नेमस्पेस के पास अपना **उपयोगकर्ता और समूह आईडी सेट** होता है। यह अलगाव प्रक्रियाओं को सक्षम करता है जो विभिन्न उपयोगकर्ता नेमस्पेस में चल रही होती हैं, ताकि वे **विभिन्न विशेषाधिकार और स्वामित्व** रख सकें, भले ही वे संख्यात्मक रूप से एक ही उपयोगकर्ता और समूह आईडी साझा करते हों।

उपयोगकर्ता नेमस्पेस विशेष रूप से कंटेनरीकरण में उपयोगी है, जहां प्रत्येक कंटेनर को अपने विशिष्ट उपयोगकर्ता और समूह आईडी सेट होनी चाहिए, जिससे कंटेनर और मेज़बान सिस्टम के बीच बेहतर सुरक्षा और अलगाव हो सके।

### काम कैसे करता है:

1. जब एक नया उपयोगकर्ता नेमस्पेस बनाया जाता है, तो यह **उपयोगकर्ता और समूह आईडी मैपिंग का खाली सेट के साथ शुरू होता है**। इसका मतलब है कि नए उपयोगकर्ता नेमस्पेस में चल रही किसी भी प्रक्रिया को **आरंभ में नेमस्पेस के बाहर कोई विशेषाधिकार नहीं होता है**।
2. आईडी मैपिंग नए नेमस्पेस में उपयोगकर्ता और समूह आईडी के बीच और माता (या मेज़बान) नेमस्पेस में वहां की उपयोगकर्ता और समूह आईडी के बीच स्थापित की जा सकती है। यह **नए नेमस्पेस में प्रक्रियाओं को उपयोगकर्ता और समूह आईडी के अनुसार विशेषाधिकार और स्वामित्व देने की अनुमति देता है**। हालांकि, आईडी मैपिंग को विशेष सीमाओं और आईडी के उपसमूहों पर प्रतिबंधित किया जा सकता है, जिससे नए नेमस्पेस में प्रक्रियाओं को दी जाने वाली विशेषाधिकारों पर नियंत्रण करने की सुविधा होती है।
3. एक उपयोगकर्ता नेमस्पेस के भीतर, **प्रक्रियाओं को नेमस्पेस के भीतर कार्यों के लिए पूर्ण रूप से रूट विशेषाधिकार (UID 0) हो सकते हैं**, जबकि उनके पास नेमस्पेस के बाहर सीमित विशेषाधिकार हो सकते हैं। यह उन्हें **मेज़बान सिस्टम पर पूर्ण रूट विशेषाधिकार न होने के बावजूद अपने नेमस्पेस के भीतर रूट जैसी क्षमताओं के साथ कंटेनर चलाने की अनुमति देता है**।
4. प्रक्रियाएँ `setns()` सिस्टम कॉल का उपयोग करके नेमस्पेस के बीच ले जा सकती हैं या `unshare()` या `clone()` सिस्टम कॉल का उपयोग करके `CLONE_NEWUSER` ध्वज के साथ नए नेमस्पेस बना सकती हैं। जब कोई प्रक्रिया नए नेमस्पेस में जाती है या एक नया नेमस्पेस बनाती है, तो वह उस नेमस्पेस से संबंधित उपयोगकर्ता और समूह आईडी मैपिंग का उपयोग करने लगेगी।

## लैब:

### विभिन्न नेमस्पेस बनाएँ

#### CLI
```bash
sudo unshare -U [--mount-proc] /bin/bash
```
यदि आप `--mount-proc` पैरामीटर का उपयोग करते हैं तो `/proc` फ़ाइल सिस्टम का एक नया इंस्टेंस माउंट करके, आप सुनिश्चित करते हैं कि नया माउंट नेमस्पेस के पास उस नेमस्पेस के लिए विशिष्ट प्रक्रिया सूचना का सटीक और अलग दृश्य है।
```bash
docker run -ti --name ubuntu1 -v /usr:/ubuntu1 ubuntu bash
```
उपयोगकर्ता नेमस्पेस का उपयोग करने के लिए, डॉकर डेमन को **`--userns-remap=default`** के साथ शुरू किया जाना चाहिए (यूबंटू 14.04 में, इसे `/etc/default/docker` को संशोधित करके और फिर `sudo service docker restart` को क्रियान्वित करके किया जा सकता है)

### &#x20;जांचें कि आपकी प्रक्रिया किस नेमस्पेस में है
```bash
ls -l /proc/self/ns/user
lrwxrwxrwx 1 root root 0 Apr  4 20:57 /proc/self/ns/user -> 'user:[4026531837]'
```
आप डॉकर कंटेनर से उपयोगकर्ता मानचित्र की जांच कर सकते हैं:
```bash
cat /proc/self/uid_map
0          0 4294967295  --> Root is root in host
0     231072      65536  --> Root is 231072 userid in host
```
या मेज़बान से:
```bash
cat /proc/<pid>/uid_map
```
### सभी उपयोगकर्ता नेमस्पेस खोजें

{% code overflow="wrap" %}
```bash
sudo find /proc -maxdepth 3 -type l -name user -exec readlink {} \; 2>/dev/null | sort -u
# Find the processes with an specific namespace
sudo find /proc -maxdepth 3 -type l -name user -exec ls -l  {} \; 2>/dev/null | grep <ns-number>
```
{% endcode %}

### एक उपयोगकर्ता नेमस्पेस के अंदर प्रवेश करें
```bash
nsenter -U TARGET_PID --pid /bin/bash
```
आप केवल **रूट होने पर** **दूसरे प्रक्रिया नेमस्पेस में प्रवेश** कर सकते हैं। और आप **उसमें प्रवेश नहीं** कर सकते **बिना एक डिस्क्रिप्टर** के जो इसे पॉइंट करता है (जैसे `/proc/self/ns/user`)।

### नया उपयोगकर्ता नेमस्पेस बनाएं (मैपिंग के साथ)

{% code overflow="wrap" %}
```bash
unshare -U [--map-user=<uid>|<name>] [--map-group=<gid>|<name>] [--map-root-user] [--map-current-user]
```
{% endcode %}
```bash
# Container
sudo unshare -U /bin/bash
nobody@ip-172-31-28-169:/home/ubuntu$ #Check how the user is nobody

# From the host
ps -ef | grep bash # The user inside the host is still root, not nobody
root       27756   27755  0 21:11 pts/10   00:00:00 /bin/bash
```
### क्षमताओं को पुनर्प्राप्त करना

उपयोगकर्ता नेमस्पेस के मामले में, **जब एक नया उपयोगकर्ता नेमस्पेस बनाया जाता है, तो नेमस्पेस में प्रवेश करने वाले प्रक्रिया को उस नेमस्पेस के भीतर पूरी सेट की क्षमताएं प्रदान की जाती है**। ये क्षमताएं प्रक्रिया को विशेषाधिकारित ऑपरेशन करने की अनुमति देती हैं जैसे **फाइल सिस्टम माउंट करना**, डिवाइस बनाना, या फ़ाइलों की स्वामित्व परिवर्तित करना, लेकिन **केवल अपने उपयोगकर्ता नेमस्पेस के संदर्भ में**।

उदाहरण के लिए, जब आपके पास उपयोगकर्ता नेमस्पेस के भीतर `CAP_SYS_ADMIN` क्षमता होती है, तो आप उस क्षमता की आवश्यकता होने पर ऑपरेशन कर सकते हैं, जैसे कि फाइल सिस्टम माउंट करना, लेकिन केवल अपने उपयोगकर्ता नेमस्पेस के संदर्भ में। इस क्षमता के साथ आप जो भी ऑपरेशन करते हैं, वे मुख्य सिस्टम या अन्य नेमस्पेस पर प्रभाव नहीं डालेंगे।

{% hint style="warning" %}
इसलिए, यदि एक नए प्रक्रिया को एक नए उपयोगकर्ता नेमस्पेस के भीतर लाने से **आपको सभी क्षमताएं वापस मिल जाएंगी** (CapEff: 000001ffffffffff), तो आप वास्तव में **केवल उन्हें उपयोग कर सकते हैं जो नेमस्पेस से संबंधित हैं** (उदाहरण के लिए माउंट) लेकिन सभी नहीं। इसलिए, यह अपने आप में एक डॉकर कंटेनर से बाहर निकलने के लिए पर्याप्त नहीं है।
{% endhint %}
```bash
# There are the syscalls that are filtered after changing User namespace with:
unshare -UmCpf  bash

Probando: 0x067 . . . Error
Probando: 0x070 . . . Error
Probando: 0x074 . . . Error
Probando: 0x09b . . . Error
Probando: 0x0a3 . . . Error
Probando: 0x0a4 . . . Error
Probando: 0x0a7 . . . Error
Probando: 0x0a8 . . . Error
Probando: 0x0aa . . . Error
Probando: 0x0ab . . . Error
Probando: 0x0af . . . Error
Probando: 0x0b0 . . . Error
Probando: 0x0f6 . . . Error
Probando: 0x12c . . . Error
Probando: 0x130 . . . Error
Probando: 0x139 . . . Error
Probando: 0x140 . . . Error
Probando: 0x141 . . . Error
Probando: 0x143 . . . Error
```
## संदर्भ
* [https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory](https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory)

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)** पर **फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके।

</details>
