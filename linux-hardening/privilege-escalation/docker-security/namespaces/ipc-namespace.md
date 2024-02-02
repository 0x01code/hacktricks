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

IPC (Inter-Process Communication) namespace एक Linux kernel फीचर है जो System V IPC ऑब्जेक्ट्स, जैसे कि मैसेज क्यूज़, शेयर्ड मेमोरी सेगमेंट्स, और सेमाफोर्स का **अलगाव** प्रदान करता है। यह अलगाव सुनिश्चित करता है कि **विभिन्न IPC namespaces में प्रोसेस एक दूसरे के IPC ऑब्जेक्ट्स को सीधे एक्सेस या मॉडिफाई नहीं कर सकते**, जिससे प्रोसेस समूहों के बीच एक अतिरिक्त सुरक्षा और गोपनीयता की परत प्रदान होती है।

### यह कैसे काम करता है:

1. जब एक नया IPC namespace बनाया जाता है, तो यह एक **पूरी तरह से अलग System V IPC ऑब्जेक्ट्स के सेट के साथ शुरू होता है**। इसका मतलब है कि नए IPC namespace में चल रहे प्रोसेस अन्य namespaces या होस्ट सिस्टम में IPC ऑब्जेक्ट्स को डिफ़ॉल्ट रूप से एक्सेस या हस्तक्षेप नहीं कर सकते।
2. एक namespace के भीतर बनाए गए IPC ऑब्जेक्ट्स केवल उसी namespace के प्रोसेस के लिए दिखाई देते हैं और **केवल उन्हीं तक पहुँच योग्य होते हैं**। प्रत्येक IPC ऑब्जेक्ट की पहचान उसके namespace के भीतर एक अद्वितीय कुंजी द्वारा की जाती है। हालांकि कुंजी विभिन्न namespaces में समान हो सकती है, ऑब्जेक्ट्स स्वयं अलग होते हैं और namespaces के पार एक्सेस नहीं किए जा सकते।
3. प्रोसेस `setns()` सिस्टम कॉल का उपयोग करके या `unshare()` या `clone()` सिस्टम कॉल्स के साथ `CLONE_NEWIPC` फ्लैग का उपयोग करके नए namespaces बना सकते हैं या namespaces के बीच में जा सकते हैं। जब एक प्रोसेस एक नए namespace में जाता है या एक बनाता है, तो वह उस namespace से जुड़े IPC ऑब्जेक्ट्स का उपयोग करना शुरू कर देगा।

## प्रयोगशाला:

### विभिन्न Namespaces बनाएं

#### CLI
```bash
sudo unshare -i [--mount-proc] /bin/bash
```
`--mount-proc` पैरामीटर का उपयोग करके नए `/proc` फाइलसिस्टम को माउंट करने से आप सुनिश्चित करते हैं कि नए माउंट नेमस्पेस में **उस नेमस्पेस के विशिष्ट प्रक्रिया जानकारी का सटीक और अलग दृश्य हो**।

<details>

<summary>एरर: bash: fork: Cannot allocate memory</summary>

यदि आप पिछली लाइन `-f` के बिना चलाते हैं तो आपको वह एरर मिलेगा।\
यह एरर इसलिए होता है क्योंकि PID 1 प्रक्रिया नए नेमस्पेस में बाहर निकल जाती है।

bash चलने के बाद, bash कई नए सब-प्रोसेस फोर्क करेगा कुछ काम करने के लिए। यदि आप unshare को -f के बिना चलाते हैं, तो bash का समान pid होगा जैसा कि वर्तमान "unshare" प्रक्रिया का है। वर्तमान "unshare" प्रक्रिया unshare सिस्टमकॉल को कॉल करती है, एक नया pid नेमस्पेस बनाती है, लेकिन वर्तमान "unshare" प्रक्रिया नए pid नेमस्पेस में नहीं होती है। यह लिनक्स कर्नेल का वांछित व्यवहार है: प्रक्रिया A एक नया नेमस्पेस बनाती है, प्रक्रिया A स्वयं नए नेमस्पेस में नहीं डाली जाती है, केवल प्रक्रिया A की सब-प्रोसेस ही नए नेमस्पेस में डाली जाती हैं। इसलिए जब आप चलाते हैं:
```
unshare -p /bin/bash
```
```markdown
unshare प्रक्रिया /bin/bash को exec करेगी, और /bin/bash कई सब-प्रोसेस फोर्क करेगा, पहला सब-प्रोसेस बैश का नए नेमस्पेस का PID 1 बन जाएगा, और सबप्रोसेस अपना काम पूरा करने के बाद बाहर निकल जाएगा। इसलिए नए नेमस्पेस का PID 1 बाहर निकल जाता है।

PID 1 प्रक्रिया का एक विशेष कार्य होता है: यह सभी अनाथ प्रक्रियाओं का माता-पिता प्रक्रिया बन जाना चाहिए। अगर रूट नेमस्पेस में PID 1 प्रक्रिया बाहर निकल जाती है, तो कर्नेल पैनिक हो जाएगा। अगर एक सब नेमस्पेस में PID 1 प्रक्रिया बाहर निकल जाती है, तो लिनक्स कर्नेल disable\_pid\_allocation फंक्शन को कॉल करेगा, जो उस नेमस्पेस में PIDNS\_HASH\_ADDING फ्लैग को साफ कर देगा। जब लिनक्स कर्नेल एक नई प्रक्रिया बनाता है, कर्नेल alloc\_pid फंक्शन को कॉल करेगा ताकि एक नेमस्पेस में PID आवंटित कर सके, और अगर PIDNS\_HASH\_ADDING फ्लैग सेट नहीं है, तो alloc\_pid फंक्शन एक -ENOMEM त्रुटि लौटाएगा। इसीलिए आपको "Cannot allocate memory" त्रुटि मिली।

आप इस समस्या को '-f' विकल्प का उपयोग करके हल कर सकते हैं:
```
```
unshare -fp /bin/bash
```
यदि आप '-f' विकल्प के साथ unshare चलाते हैं, तो unshare नया pid namespace बनाने के बाद एक नई प्रक्रिया को fork करेगा। और नई प्रक्रिया में /bin/bash चलाएगा। नई प्रक्रिया नए pid namespace की pid 1 होगी। फिर bash कई सब-प्रक्रियाओं को कुछ काम करने के लिए fork करेगा। चूंकि bash स्वयं नए pid namespace की pid 1 है, इसकी सब-प्रक्रियाएं बिना किसी समस्या के बाहर निकल सकती हैं।

[https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory](https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory) से कॉपी किया गया

</details>

#### Docker
```bash
docker run -ti --name ubuntu1 -v /usr:/ubuntu1 ubuntu bash
```
### &#x20;जांचें कि आपकी प्रक्रिया किस नेमस्पेस में है
```bash
ls -l /proc/self/ns/ipc
lrwxrwxrwx 1 root root 0 Apr  4 20:37 /proc/self/ns/ipc -> 'ipc:[4026531839]'
```
### सभी IPC namespaces खोजें

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
इसके अलावा, आप केवल **अगर आप root हैं तो दूसरे process namespace में प्रवेश कर सकते हैं**। और आप **नहीं** **प्रवेश** कर सकते हैं दूसरे namespace में **बिना एक descriptor के** जो उसकी ओर इशारा करता हो (जैसे `/proc/self/ns/net`).

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
<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग प्राप्त करें**](https://peass.creator-spring.com)
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग ट्रिक्स शेयर करें।

</details>
