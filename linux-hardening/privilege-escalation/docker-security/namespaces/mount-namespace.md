# माउंट नेमस्पेस

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें.

</details>

## मूल जानकारी

माउंट नेमस्पेस एक Linux कर्नेल फीचर है जो प्रक्रियाओं के एक समूह द्वारा देखे गए फाइल सिस्टम माउंट पॉइंट्स का अलगाव प्रदान करता है। प्रत्येक माउंट नेमस्पेस का अपना सेट ऑफ फाइल सिस्टम माउंट पॉइंट्स होता है, और **एक नेमस्पेस में माउंट पॉइंट्स में परिवर्तन अन्य नेमस्पेसेस को प्रभावित नहीं करते**। इसका मतलब है कि विभिन्न माउंट नेमस्पेसेस में चल रही प्रक्रियाएं फाइल सिस्टम की हायरार्की के विभिन्न दृश्य देख सकती हैं।

माउंट नेमस्पेसेस कंटेनराइजेशन में विशेष रूप से उपयोगी होते हैं, जहां प्रत्येक कंटेनर का अपना फाइल सिस्टम और कॉन्फ़िगरेशन होना चाहिए, अन्य कंटेनरों और होस्ट सिस्टम से अलग।

### यह कैसे काम करता है:

1. जब एक नया माउंट नेमस्पेस बनाया जाता है, तो यह अपने पेरेंट नेमस्पेस से माउंट पॉइंट्स की एक **प्रतिलिपि के साथ आरंभ किया जाता है**। इसका मतलब है कि, निर्माण के समय, नया नेमस्पेस अपने पेरेंट के समान फाइल सिस्टम का दृश्य साझा करता है। हालांकि, नेमस्पेस के भीतर माउंट पॉइंट्स में किसी भी बाद के परिवर्तन से पेरेंट या अन्य नेमस्पेसेस प्रभावित नहीं होंगे।
2. जब एक प्रक्रिया अपने नेमस्पेस के भीतर एक माउंट पॉइंट को संशोधित करती है, जैसे कि एक फाइल सिस्टम को माउंट करना या अनमाउंट करना, तो **परिवर्तन उस नेमस्पेस के लिए स्थानीय होता है** और अन्य नेमस्पेसेस को प्रभावित नहीं करता। इससे प्रत्येक नेमस्पेस का अपना स्वतंत्र फाइल सिस्टम हायरार्की हो सकता है।
3. प्रक्रियाएं `setns()` सिस्टम कॉल का उपयोग करके नेमस्पेसेस के बीच में जा सकती हैं, या `unshare()` या `clone()` सिस्टम कॉल्स का उपयोग करके `CLONE_NEWNS` फ्लैग के साथ नए नेमस्पेसेस बना सकती हैं। जब एक प्रक्रिया एक नए नेमस्पेस में जाती है या एक बनाती है, तो वह उस नेमस्पेस से जुड़े माउंट पॉइंट्स का उपयोग करना शुरू कर देगी।
4. **फाइल डिस्क्रिप्टर्स और इनोड्स नेमस्पेसेस के आर-पार साझा किए जाते हैं**, इसका मतलब है कि यदि एक नेमस्पेस में एक प्रक्रिया के पास एक फाइल की ओर इशारा करता हुआ एक खुला फाइल डिस्क्रिप्टर है, तो वह **उस फाइल डिस्क्रिप्टर को** दूसरे नेमस्पेस में एक प्रक्रिया को **पास कर सकती है**, और **दोनों प्रक्रियाएं उसी फाइल तक पहुंच प्राप्त करेंगी**। हालांकि, दोनों नेमस्पेसेस में माउंट पॉइंट्स में अंतर के कारण फाइल का पथ समान नहीं हो सकता है।

## प्रयोगशाला:

### विभिन्न नेमस्पेसेस बनाएं

#### CLI
```bash
sudo unshare -m [--mount-proc] /bin/bash
```
`--mount-proc` पैरामीटर का उपयोग करके एक नए `/proc` फाइलसिस्टम की इंस्टेंस को माउंट करने से, आप सुनिश्चित करते हैं कि नया माउंट नेमस्पेस **उस नेमस्पेस के लिए विशिष्ट प्रक्रिया जानकारी का सटीक और अलग दृश्य** प्रदान करता है।

<details>

<summary>एरर: bash: fork: Cannot allocate memory</summary>

यदि आप पिछली लाइन `-f` के बिना चलाते हैं तो आपको वह एरर मिलेगा।\
यह एरर इसलिए होता है क्योंकि नए नेमस्पेस में PID 1 प्रक्रिया समाप्त हो जाती है।

जब bash चलना शुरू करता है, तो bash कई नए सब-प्रोसेस बनाता है कुछ काम करने के लिए। यदि आप unshare को `-f` के बिना चलाते हैं, तो bash का समान pid होगा जैसा कि वर्तमान "unshare" प्रक्रिया का है। वर्तमान "unshare" प्रक्रिया unshare सिस्टमकॉल को कॉल करती है, एक नया pid नेमस्पेस बनाती है, लेकिन वर्तमान "unshare" प्रक्रिया नए pid नेमस्पेस में नहीं होती है। यह लिनक्स कर्नेल का वांछित व्यवहार है: प्रक्रिया A एक नया नेमस्पेस बनाती है, प्रक्रिया A स्वयं नए नेमस्पेस में नहीं डाली जाती है, केवल प्रक्रिया A की सब-प्रोसेस ही नए नेमस्पेस में डाली जाती हैं। इसलिए जब आप चलाते हैं:
```
unshare -p /bin/bash
```
unshare प्रक्रिया /bin/bash को exec करेगी, और /bin/bash कई सब-प्रोसेस फोर्क करेगा, पहला सब-प्रोसेस बैश का नए नेमस्पेस का PID 1 बन जाएगा, और सबप्रोसेस अपना काम पूरा करने के बाद एक्जिट हो जाएगा। इसलिए नए नेमस्पेस का PID 1 एक्जिट हो जाता है।

PID 1 प्रक्रिया का एक विशेष कार्य होता है: यह सभी अनाथ प्रक्रियाओं का माता-पिता प्रक्रिया बनना चाहिए। अगर रूट नेमस्पेस में PID 1 प्रक्रिया एक्जिट हो जाती है, तो कर्नेल पैनिक हो जाएगा। अगर एक सब नेमस्पेस में PID 1 प्रक्रिया एक्जिट हो जाती है, तो लिनक्स कर्नेल disable_pid_allocation फंक्शन को कॉल करेगा, जो उस नेमस्पेस में PIDNS_HASH_ADDING फ्लैग को साफ कर देगा। जब लिनक्स कर्नेल एक नई प्रक्रिया बनाता है, कर्नेल alloc_pid फंक्शन को कॉल करेगा ताकि एक नेमस्पेस में एक PID आवंटित कर सके, और अगर PIDNS_HASH_ADDING फ्लैग सेट नहीं है, तो alloc_pid फंक्शन -ENOMEM एरर लौटाएगा। इसीलिए आपको "Cannot allocate memory" एरर मिला।

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
ls -l /proc/self/ns/mnt
lrwxrwxrwx 1 root root 0 Apr  4 20:30 /proc/self/ns/mnt -> 'mnt:[4026531841]'
```
### सभी Mount namespaces का पता लगाएं

{% code overflow="wrap" %}
```bash
sudo find /proc -maxdepth 3 -type l -name mnt -exec readlink {} \; 2>/dev/null | sort -u
# Find the processes with an specific namespace
sudo find /proc -maxdepth 3 -type l -name mnt -exec ls -l  {} \; 2>/dev/null | grep <ns-number>
```
{% endcode %}

### माउंट नेमस्पेस के अंदर प्रवेश करें
```bash
nsenter -m TARGET_PID --pid /bin/bash
```
यह भी ध्यान रखें कि आप किसी अन्य प्रक्रिया के **namespace में केवल तभी प्रवेश कर सकते हैं जब आप root हों**। और आप किसी अन्य namespace में **बिना डिस्क्रिप्टर के प्रवेश नहीं कर सकते** जो उसकी ओर इशारा करता हो (जैसे `/proc/self/ns/mnt`).

चूंकि नए mounts केवल namespace के भीतर ही सुलभ होते हैं, इसलिए संभव है कि एक namespace में संवेदनशील जानकारी हो जो केवल उसी से सुलभ हो सकती है।

### कुछ mount करें
```bash
# Generate new mount ns
unshare -m /bin/bash
mkdir /tmp/mount_ns_example
mount -t tmpfs tmpfs /tmp/mount_ns_example
mount | grep tmpfs # "tmpfs on /tmp/mount_ns_example"
echo test > /tmp/mount_ns_example/test
ls /tmp/mount_ns_example/test # Exists

# From the host
mount | grep tmpfs # Cannot see "tmpfs on /tmp/mount_ns_example"
ls /tmp/mount_ns_example/test # Doesn't exist
```
<details>

<summary><strong>शून्य से नायक तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग प्राप्त करें**](https://peass.creator-spring.com)
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह में शामिल हों**](https://discord.gg/hRep4RUj7f) या [**telegram समूह**](https://t.me/peass) या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>
