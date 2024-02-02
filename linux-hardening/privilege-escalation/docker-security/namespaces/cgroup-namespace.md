# CGroup Namespace

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग प्राप्त करें**](https://peass.creator-spring.com)
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) में या **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) को **फॉलो करें**.
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें.

</details>

## मूल जानकारी

Cgroup namespace एक Linux kernel फीचर है जो **प्रोसेसेस के लिए cgroup हायरार्की के आइसोलेशन प्रदान करता है** जो एक namespace के भीतर चल रहे होते हैं। Cgroups, जिसे **कंट्रोल ग्रुप्स** के नाम से भी जाना जाता है, एक kernel फीचर है जो प्रोसेसेस को हायरार्किकल ग्रुप्स में व्यवस्थित करने और CPU, मेमोरी, और I/O जैसे **सिस्टम संसाधनों पर सीमाएं लागू करने** की अनुमति देता है।

जबकि cgroup namespaces अन्य जिन namespace प्रकारों के बारे में हमने पहले चर्चा की थी (PID, mount, network, आदि) की तरह एक अलग namespace प्रकार नहीं हैं, वे namespace आइसोलेशन की अवधारणा से संबंधित हैं। **Cgroup namespaces cgroup हायरार्की के दृश्य को वर्चुअलाइज करते हैं**, ताकि cgroup namespace के भीतर चल रहे प्रोसेसेस को हायरार्की का एक अलग दृश्य मिलता है जबकि होस्ट या अन्य namespaces में चल रहे प्रोसेसेस की तुलना में।

### यह कैसे काम करता है:

1. जब एक नया cgroup namespace बनाया जाता है, **यह cgroup हायरार्की के दृश्य के साथ शुरू होता है जो बनाने वाले प्रोसेस के cgroup पर आधारित होता है**। इसका मतलब है कि नए cgroup namespace में चल रहे प्रोसेसेस केवल cgroup हायरार्की का एक उपसमूह देखेंगे, जो बनाने वाले प्रोसेस के cgroup के रूट पर स्थित cgroup उपवृक्ष तक सीमित होता है।
2. Cgroup namespace के भीतर के प्रोसेसेस **अपने cgroup को हायरार्की के रूट के रूप में देखेंगे**। इसका मतलब है कि, namespace के भीतर प्रोसेसेस के दृष्टिकोण से, उनका अपना cgroup रूट के रूप में प्रतीत होता है, और वे अपने उपवृक्ष के बाहर के cgroups को देख या एक्सेस नहीं कर सकते।
3. Cgroup namespaces सीधे संसाधनों के आइसोलेशन प्रदान नहीं करते हैं; **वे केवल cgroup हायरार्की दृश्य के आइसोलेशन प्रदान करते हैं**। **संसाधन नियंत्रण और आइसोलेशन अभी भी cgroup** उपसिस्टम्स (जैसे कि cpu, मेमोरी, आदि) द्वारा लागू किए जाते हैं।

CGroups के बारे में अधिक जानकारी के लिए देखें:

{% content-ref url="../cgroups.md" %}
[cgroups.md](../cgroups.md)
{% endcontent-ref %}

## प्रयोगशाला:

### विभिन्न Namespaces बनाएं

#### CLI
```bash
sudo unshare -C [--mount-proc] /bin/bash
```
`--mount-proc` पैरामीटर का उपयोग करके एक नए `/proc` फाइलसिस्टम को माउंट करने से आप सुनिश्चित करते हैं कि नए माउंट नेमस्पेस में उस नेमस्पेस के विशिष्ट प्रक्रिया जानकारी का **सटीक और अलग दृश्य** होता है।

<details>

<summary>एरर: bash: fork: Cannot allocate memory</summary>

यदि आप पिछली लाइन `-f` के बिना चलाते हैं, तो आपको वह एरर मिलेगा।\
यह एरर इसलिए होता है क्योंकि नए नेमस्पेस में PID 1 प्रक्रिया समाप्त हो जाती है।

bash चलने के बाद, bash कई नए उप-प्रक्रियाओं को कुछ करने के लिए फोर्क करेगा। यदि आप बिना -f के unshare चलाते हैं, तो bash का समान pid वर्तमान "unshare" प्रक्रिया के रूप में होगा। वर्तमान "unshare" प्रक्रिया unshare सिस्टमकॉल को कॉल करती है, एक नया pid नेमस्पेस बनाती है, लेकिन वर्तमान "unshare" प्रक्रिया नए pid नेमस्पेस में नहीं होती है। यह लिनक्स कर्नेल का वांछित व्यवहार है: प्रक्रिया A एक नया नेमस्पेस बनाती है, प्रक्रिया A स्वयं नए नेमस्पेस में नहीं डाली जाती है, केवल प्रक्रिया A की उप-प्रक्रियाएं नए नेमस्पेस में डाली जाती हैं। इसलिए जब आप चलाते हैं:
```
unshare -p /bin/bash
```
unshare प्रक्रिया /bin/bash को exec करेगी, और /bin/bash कई सब-प्रोसेस उत्पन्न करेगा, पहला सब-प्रोसेस बैश का नए नेमस्पेस का PID 1 बन जाएगा, और सबप्रोसेस अपना काम पूरा करने के बाद बाहर निकल जाएगा। इसलिए नए नेमस्पेस का PID 1 बाहर निकल जाता है।

PID 1 प्रक्रिया का एक विशेष कार्य होता है: यह सभी अनाथ प्रक्रियाओं का माता-पिता प्रक्रिया बनना चाहिए। यदि रूट नेमस्पेस में PID 1 प्रक्रिया बाहर निकल जाती है, तो कर्नेल पैनिक हो जाएगा। यदि एक सब नेमस्पेस में PID 1 प्रक्रिया बाहर निकल जाती है, तो लिनक्स कर्नेल disable_pid_allocation फंक्शन को कॉल करेगा, जो उस नेमस्पेस में PIDNS_HASH_ADDING फ्लैग को साफ कर देगा। जब लिनक्स कर्नेल एक नई प्रक्रिया बनाता है, कर्नेल alloc_pid फंक्शन को कॉल करेगा ताकि एक नेमस्पेस में PID आवंटित कर सके, और यदि PIDNS_HASH_ADDING फ्लैग सेट नहीं है, तो alloc_pid फंक्शन -ENOMEM त्रुटि लौटाएगा। इसीलिए आपको "Cannot allocate memory" त्रुटि मिली।

आप इस समस्या को '-f' विकल्प का उपयोग करके हल कर सकते हैं:
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
### जांचें कि आपकी प्रक्रिया किस नेमस्पेस में है
```bash
ls -l /proc/self/ns/cgroup
lrwxrwxrwx 1 root root 0 Apr  4 21:19 /proc/self/ns/cgroup -> 'cgroup:[4026531835]'
```
### सभी CGroup नेमस्पेस का पता लगाएं

{% code overflow="wrap" %}
```bash
sudo find /proc -maxdepth 3 -type l -name cgroup -exec readlink {} \; 2>/dev/null | sort -u
# Find the processes with an specific namespace
sudo find /proc -maxdepth 3 -type l -name cgroup -exec ls -l  {} \; 2>/dev/null | grep <ns-number>
```
{% endcode %}

### CGroup नेमस्पेस के अंदर प्रवेश करें
```bash
nsenter -C TARGET_PID --pid /bin/bash
```
```markdown
आप किसी अन्य प्रक्रिया के नेमस्पेस में केवल **रूट होने पर ही प्रवेश कर सकते हैं**। और आप बिना डिस्क्रिप्टर के अन्य नेमस्पेस में **प्रवेश नहीं** कर सकते (जैसे `/proc/self/ns/cgroup`).

<details>

<summary><strong>शून्य से नायक तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) को **फॉलो करें**।
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>
```
