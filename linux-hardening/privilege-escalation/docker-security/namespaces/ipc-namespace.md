# IPC Namespace

<details>

<summary><strong>शून्य से नायक तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) को **फॉलो करें**.
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें.

</details>

## मूल जानकारी

IPC (Inter-Process Communication) namespace एक Linux kernel सुविधा है जो System V IPC ऑब्जेक्ट्स जैसे कि मैसेज क्यूज़, शेयर्ड मेमोरी सेगमेंट्स, और सेमाफोर्स का **अलगाव** प्रदान करती है। यह अलगाव सुनिश्चित करता है कि **विभिन्न IPC namespaces में प्रक्रियाएं एक दूसरे के IPC ऑब्जेक्ट्स को सीधे एक्सेस या मॉडिफाई नहीं कर सकतीं**, जिससे प्रक्रिया समूहों के बीच एक अतिरिक्त सुरक्षा और गोपनीयता की परत प्रदान होती है।

### यह कैसे काम करता है:

1. जब एक नया IPC namespace बनाया जाता है, तो यह एक **पूरी तरह से अलग System V IPC ऑब्जेक्ट्स के सेट के साथ शुरू होता है**। इसका मतलब है कि नए IPC namespace में चल रही प्रक्रियाएं अन्य namespaces या होस्ट सिस्टम में IPC ऑब्जेक्ट्स को डिफ़ॉल्ट रूप से एक्सेस या हस्तक्षेप नहीं कर सकतीं।
2. एक namespace के भीतर बनाए गए IPC ऑब्जेक्ट्स केवल उसी namespace की प्रक्रियाओं के लिए दिखाई देते हैं और **केवल उन्हीं तक पहुंच योग्य होते हैं**। प्रत्येक IPC ऑब्जेक्ट की पहचान उसके namespace के भीतर एक अद्वितीय कुंजी द्वारा की जाती है। हालांकि कुंजी विभिन्न namespaces में समान हो सकती है, ऑब्जेक्ट्स स्वयं अलग होते हैं और namespaces के पार एक्सेस नहीं किए जा सकते।
3. प्रक्रियाएं `setns()` सिस्टम कॉल का उपयोग करके या `unshare()` या `clone()` सिस्टम कॉल्स के साथ `CLONE_NEWIPC` फ्लैग का उपयोग करके नए namespaces बना सकती हैं या namespaces के बीच में जा सकती हैं। जब एक प्रक्रिया एक नए namespace में जाती है या एक बनाती है, तो वह उस namespace से जुड़े IPC ऑब्जेक्ट्स का उपयोग करना शुरू कर देगी।

## प्रयोगशाला:

### विभिन्न Namespaces बनाएं

#### CLI
```bash
sudo unshare -i [--mount-proc] /bin/bash
```
`/proc` फाइलसिस्टम का एक नया इंस्टेंस माउंट करके, यदि आप `--mount-proc` पैरामीटर का उपयोग करते हैं, तो आप सुनिश्चित करते हैं कि नया माउंट नेमस्पेस उस नेमस्पेस के लिए विशिष्ट प्रोसेस जानकारी का **सटीक और अलग दृश्य** रखता है।

<details>

<summary>एरर: bash: fork: Cannot allocate memory</summary>

`unshare` को `-f` विकल्प के बिना निष्पादित करने पर, एक एरर का सामना करना पड़ता है, जो लिनक्स नए PID (प्रोसेस ID) नेमस्पेस को कैसे हैंडल करता है, उसके कारण होता है। मुख्य विवरण और समाधान नीचे दिए गए हैं:

1. **समस्या की व्याख्या**:
- लिनक्स कर्नेल एक प्रोसेस को `unshare` सिस्टम कॉल का उपयोग करके नए नेमस्पेस बनाने की अनुमति देता है। हालांकि, नए PID नेमस्पेस के निर्माण की पहल करने वाला प्रोसेस (जिसे "unshare" प्रोसेस कहा जाता है) नए नेमस्पेस में प्रवेश नहीं करता; केवल उसके चाइल्ड प्रोसेस ही करते हैं।
- `%unshare -p /bin/bash%` चलाने से `/bin/bash` `unshare` के समान प्रोसेस में शुरू होता है। नतीजतन, `/bin/bash` और उसके चाइल्ड प्रोसेस मूल PID नेमस्पेस में होते हैं।
- नए नेमस्पेस में `/bin/bash` का पहला चाइल्ड प्रोसेस PID 1 बन जाता है। जब यह प्रोसेस बाहर निकलता है, तो यदि कोई अन्य प्रोसेस नहीं हैं, तो नेमस्पेस की सफाई ट्रिगर होती है, क्योंकि PID 1 का अनाथ प्रोसेस को गोद लेने का विशेष रोल होता है। लिनक्स कर्नेल तब उस नेमस्पेस में PID आवंटन को अक्षम कर देगा।

2. **परिणाम**:
- नए नेमस्पेस में PID 1 के बाहर निकलने से `PIDNS_HASH_ADDING` फ्लैग की सफाई हो जाती है। इससे `alloc_pid` फंक्शन नए प्रोसेस बनाते समय नए PID को आवंटित करने में विफल हो जाता है, जिससे "Cannot allocate memory" एरर होता है।

3. **समाधान**:
- इस समस्या का समाधान `unshare` के साथ `-f` विकल्प का उपयोग करके किया जा सकता है। यह विकल्प `unshare` को नए PID नेमस्पेस बनाने के बाद एक नया प्रोसेस फोर्क करने के लिए बनाता है।
- `%unshare -fp /bin/bash%` निष्पादित करने से सुनिश्चित होता है कि `unshare` कमांड स्वयं नए नेमस्पेस में PID 1 बन जाता है। `/bin/bash` और उसके चाइल्ड प्रोसेस तब इस नए नेमस्पेस के भीतर सुरक्षित रूप से समाहित होते हैं, जिससे PID 1 के समय से पहले बाहर निकलने से बचा जाता है और सामान्य PID आवंटन की अनुमति देता है।

`unshare` को `-f` फ्लैग के साथ चलाने से सुनिश्चित होता है कि नया PID नेमस्पेस सही ढंग से बनाए रखा जाता है, जिससे `/bin/bash` और उसके उप-प्रोसेस बिना मेमोरी आवंटन एरर का सामना किए बिना काम कर सकते हैं।

</details>

#### Docker
```bash
docker run -ti --name ubuntu1 -v /usr:/ubuntu1 ubuntu bash
```
### जांचें कि आपकी प्रक्रिया किस नेमस्पेस में है
```bash
ls -l /proc/self/ns/ipc
lrwxrwxrwx 1 root root 0 Apr  4 20:37 /proc/self/ns/ipc -> 'ipc:[4026531839]'
```
### सभी IPC namespaces का पता लगाएं

{% code overflow="wrap" %}
```bash
sudo find /proc -maxdepth 3 -type l -name ipc -exec readlink {} \; 2>/dev/null | sort -u
# Find the processes with an specific namespace
sudo find /proc -maxdepth 3 -type l -name ipc -exec ls -l  {} \; 2>/dev/null | grep <ns-number>
```
### IPC नेमस्पेस के अंदर प्रवेश करें
```bash
nsenter -i TARGET_PID --pid /bin/bash
```
इसके अलावा, आप किसी अन्य प्रक्रिया के **namespace में केवल तभी प्रवेश कर सकते हैं जब आप root हों**। और आप बिना किसी डिस्क्रिप्टर के अन्य namespace में **प्रवेश नहीं** कर सकते जो इसकी ओर इशारा करता हो (जैसे `/proc/self/ns/net`).

### IPC object बनाएं
```bash
# Container
sudo unshare -i /bin/bash
ipcmk -M 100
Shared memory id: 0
ipcs -m

------ Shared Memory Segments --------
key        shmid      owner      perms      bytes      nattch     status
0x2fba9021 0          root       644        100        0

# From the host
ipcs -m # Nothing is seen
```
# संदर्भ
* [https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory](https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory)

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert) के साथ शून्य से नायक तक AWS हैकिंग सीखें</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग तरकीबें साझा करें।

</details>
