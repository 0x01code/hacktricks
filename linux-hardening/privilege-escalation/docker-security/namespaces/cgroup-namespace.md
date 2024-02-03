# CGroup Namespace

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) में या **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) को **फॉलो करें**.
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें.

</details>

## मूल जानकारी

Cgroup namespace एक Linux kernel फीचर है जो प्रोसेसेस के लिए **cgroup hierarchies के अलगाव** प्रदान करता है जो एक namespace के भीतर चल रहे होते हैं। Cgroups, जिसे **control groups** के नाम से भी जाना जाता है, एक kernel फीचर है जो प्रोसेसेस को हायरार्किकल ग्रुप्स में व्यवस्थित करने और CPU, मेमोरी, और I/O जैसे **सिस्टम संसाधनों पर सीमाएं लागू करने** की अनुमति देता है।

जबकि cgroup namespaces अन्य जिन namespaces के बारे में हमने पहले चर्चा की थी (PID, mount, network, आदि) की तरह एक अलग namespace प्रकार नहीं हैं, वे namespace अलगाव की अवधारणा से संबंधित हैं। **Cgroup namespaces cgroup hierarchy के दृश्य को वर्चुअलाइज करते हैं**, ताकि cgroup namespace के भीतर चल रहे प्रोसेसेस को hierarchy का एक अलग दृश्य मिलता है, होस्ट या अन्य namespaces में चल रहे प्रोसेसेस की तुलना में।

### यह कैसे काम करता है:

1. जब एक नया cgroup namespace बनाया जाता है, **यह cgroup hierarchy के दृश्य के साथ शुरू होता है जो बनाने वाले प्रोसेस के cgroup पर आधारित होता है**। इसका मतलब है कि नए cgroup namespace में चल रहे प्रोसेसेस केवल cgroup hierarchy का एक उप-सेट देखेंगे, जो बनाने वाले प्रोसेस के cgroup पर आधारित cgroup उप-वृक्ष तक सीमित होता है।
2. Cgroup namespace के भीतर के प्रोसेसेस **अपने cgroup को hierarchy के मूल के रूप में देखेंगे**। इसका मतलब है कि, namespace के भीतर प्रोसेसेस के दृष्टिकोण से, उनका अपना cgroup मूल के रूप में प्रतीत होता है, और वे अपने उप-वृक्ष के बाहर के cgroups को देख या एक्सेस नहीं कर सकते।
3. Cgroup namespaces सीधे संसाधनों के अलगाव को प्रदान नहीं करते हैं; **वे केवल cgroup hierarchy दृश्य के अलगाव को प्रदान करते हैं**। **संसाधन नियंत्रण और अलगाव अभी भी cgroup** उप-प्रणालियों (जैसे कि cpu, मेमोरी, आदि) द्वारा लागू किए जाते हैं।

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
`/proc` फाइल सिस्टम का एक नया इंस्टेंस माउंट करके, यदि आप `--mount-proc` पैरामीटर का उपयोग करते हैं, तो आप सुनिश्चित करते हैं कि नया माउंट नेमस्पेस उस नेमस्पेस की विशिष्ट प्रक्रिया जानकारी का **सटीक और अलग दृश्य** प्रदान करता है।

<details>

<summary>एरर: bash: fork: Cannot allocate memory</summary>

`unshare` को `-f` विकल्प के बिना निष्पादित करने पर, एक एरर का सामना करना पड़ता है, जो लिनक्स नए PID (प्रोसेस ID) नेमस्पेस को कैसे हैंडल करता है, उसके कारण होता है। मुख्य विवरण और समाधान नीचे दिए गए हैं:

1. **समस्या की व्याख्या**:
- लिनक्स कर्नेल एक प्रक्रिया को `unshare` सिस्टम कॉल का उपयोग करके नए नेमस्पेस बनाने की अनुमति देता है। हालांकि, नया PID नेमस्पेस बनाने की पहल करने वाली प्रक्रिया (जिसे "unshare" प्रक्रिया कहा जाता है) नए नेमस्पेस में प्रवेश नहीं करती; केवल उसकी चाइल्ड प्रोसेस ही करती हैं।
- `%unshare -p /bin/bash%` चलाने से `/bin/bash` `unshare` की समान प्रक्रिया में शुरू होता है। नतीजतन, `/bin/bash` और उसकी चाइल्ड प्रोसेस मूल PID नेमस्पेस में होती हैं।
- नए नेमस्पेस में `/bin/bash` की पहली चाइल्ड प्रोसेस PID 1 बन जाती है। जब यह प्रोसेस बाहर निकलती है, तो यदि कोई अन्य प्रोसेस नहीं हैं, तो नेमस्पेस की सफाई ट्रिगर होती है, क्योंकि PID 1 का अनाथ प्रोसेस को गोद लेने का विशेष रोल होता है। लिनक्स कर्नेल तब उस नेमस्पेस में PID आवंटन को अक्षम कर देगा।

2. **परिणाम**:
- नए नेमस्पेस में PID 1 के बाहर निकलने से `PIDNS_HASH_ADDING` फ्लैग की सफाई हो जाती है। इससे `alloc_pid` फंक्शन नई प्रोसेस बनाते समय नया PID आवंटित करने में विफल हो जाता है, जिससे "Cannot allocate memory" एरर होता है।

3. **समाधान**:
- इस समस्या को `-f` विकल्प के साथ `unshare` का उपयोग करके हल किया जा सकता है। यह विकल्प `unshare` को नया PID नेमस्पेस बनाने के बाद एक नई प्रक्रिया को फोर्क करने के लिए बनाता है।
- `%unshare -fp /bin/bash%` निष्पादित करने से सुनिश्चित होता है कि `unshare` कमांड स्वयं नए नेमस्पेस में PID 1 बन जाता है। `/bin/bash` और उसकी चाइल्ड प्रोसेस इस नए नेमस्पेस के भीतर सुरक्षित रूप से समाहित होती हैं, जिससे PID 1 के समय से पहले बाहर निकलने को रोका जाता है और सामान्य PID आवंटन की अनुमति दी जाती है।

`unshare` को `-f` फ्लैग के साथ चलाने से सुनिश्चित होता है कि नया PID नेमस्पेस सही ढंग से बनाए रखा जाता है, जिससे `/bin/bash` और उसकी सब-प्रोसेस मेमोरी आवंटन एरर का सामना किए बिना काम कर सकती हैं।

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
```
{% endcode %}

### CGroup नेमस्पेस के अंदर प्रवेश करें
```
```bash
nsenter -C TARGET_PID --pid /bin/bash
```
```markdown
आप किसी अन्य प्रक्रिया के **namespace में केवल तभी प्रवेश कर सकते हैं जब आप root हों**। और आप बिना डिस्क्रिप्टर के अन्य namespace में **प्रवेश नहीं** कर सकते जो इसकी ओर इशारा करता हो (जैसे `/proc/self/ns/cgroup`).

# संदर्भ
* [https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory](https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory)

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert) के साथ शून्य से नायक तक AWS हैकिंग सीखें</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) में शामिल हों या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) **का अनुसरण करें**।
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>
```
