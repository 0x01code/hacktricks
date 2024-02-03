# UTS Namespace

<details>

<summary><strong>शून्य से नायक तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग प्राप्त करें**](https://peass.creator-spring.com)
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) को **फॉलो करें**.
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें.

</details>

## मूल जानकारी

UTS (UNIX Time-Sharing System) namespace एक Linux kernel सुविधा है जो दो सिस्टम पहचानकर्ताओं का **पृथक्करण प्रदान करती है**: **hostname** और **NIS** (Network Information Service) डोमेन नाम. यह पृथक्करण प्रत्येक UTS namespace को अपना **स्वतंत्र hostname और NIS डोमेन नाम** होने की अनुमति देता है, जो कंटेनराइजेशन परिदृश्यों में विशेष रूप से उपयोगी है जहां प्रत्येक कंटेनर को अपने hostname के साथ एक अलग सिस्टम के रूप में प्रतीत होना चाहिए.

### यह कैसे काम करता है:

1. जब एक नया UTS namespace बनाया जाता है, तो यह अपने पैरेंट namespace से **hostname और NIS डोमेन नाम की एक प्रति के साथ शुरू होता है**. इसका मतलब है कि, निर्माण के समय, नया namespace **अपने पैरेंट के समान पहचानकर्ता साझा करता है**. हालांकि, namespace के भीतर hostname या NIS डोमेन नाम में किसी भी बाद के परिवर्तन से अन्य namespaces प्रभावित नहीं होंगे.
2. UTS namespace के भीतर प्रक्रियाएं **hostname और NIS डोमेन नाम बदल सकती हैं** `sethostname()` और `setdomainname()` सिस्टम कॉल्स का उपयोग करके, क्रमशः. ये परिवर्तन namespace के लिए स्थानीय होते हैं और अन्य namespaces या होस्ट सिस्टम को प्रभावित नहीं करते हैं.
3. प्रक्रियाएं `setns()` सिस्टम कॉल का उपयोग करके namespaces के बीच में जा सकती हैं या `unshare()` या `clone()` सिस्टम कॉल्स का उपयोग करके `CLONE_NEWUTS` फ्लैग के साथ नए namespaces बना सकती हैं. जब एक प्रक्रिया एक नए namespace में जाती है या एक बनाती है, तो वह उस namespace से जुड़े hostname और NIS डोमेन नाम का उपयोग करना शुरू कर देगी.

## प्रयोगशाला:

### विभिन्न Namespaces बनाएं

#### CLI
```bash
sudo unshare -u [--mount-proc] /bin/bash
```
यदि आप `--mount-proc` पैरामीटर का उपयोग करके `/proc` फाइलसिस्टम का एक नया इंस्टेंस माउंट करते हैं, तो आप सुनिश्चित करते हैं कि नया माउंट नेमस्पेस उस नेमस्पेस की विशिष्ट प्रक्रिया जानकारी का **सटीक और अलग दृश्य** रखता है।

<details>

<summary>एरर: bash: fork: Cannot allocate memory</summary>

जब `unshare` को `-f` विकल्प के बिना निष्पादित किया जाता है, तो एक एरर का सामना करना पड़ता है जो लिनक्स नए PID (प्रोसेस ID) नेमस्पेस को कैसे संभालता है उसके कारण होता है। मुख्य विवरण और समाधान नीचे दिए गए हैं:

1. **समस्या की व्याख्या**:
- लिनक्स कर्नेल एक प्रक्रिया को `unshare` सिस्टम कॉल का उपयोग करके नए नेमस्पेस बनाने की अनुमति देता है। हालांकि, नए PID नेमस्पेस की सृष्टि करने वाली प्रक्रिया (जिसे "unshare" प्रक्रिया कहा जाता है) नए नेमस्पेस में प्रवेश नहीं करती है; केवल उसकी चाइल्ड प्रोसेस ही करती हैं।
- `%unshare -p /bin/bash%` चलाने से `/bin/bash` `unshare` की समान प्रक्रिया में शुरू होता है। नतीजतन, `/bin/bash` और उसकी चाइल्ड प्रोसेस मूल PID नेमस्पेस में होती हैं।
- नए नेमस्पेस में `/bin/bash` की पहली चाइल्ड प्रोसेस PID 1 बन जाती है। जब यह प्रोसेस बाहर निकलती है, तो यदि कोई अन्य प्रोसेस नहीं हैं, तो यह नेमस्पेस की सफाई को ट्रिगर करती है, क्योंकि PID 1 का अनाथ प्रोसेस को गोद लेने का विशेष रोल होता है। लिनक्स कर्नेल तब उस नेमस्पेस में PID आवंटन को अक्षम कर देगा।

2. **परिणाम**:
- नए नेमस्पेस में PID 1 के बाहर निकलने से `PIDNS_HASH_ADDING` फ्लैग की सफाई हो जाती है। इससे `alloc_pid` फंक्शन नई प्रोसेस बनाते समय नए PID को आवंटित करने में विफल हो जाता है, जिससे "Cannot allocate memory" एरर होता है।

3. **समाधान**:
- इस समस्या को `-f` विकल्प के साथ `unshare` का उपयोग करके हल किया जा सकता है। यह विकल्प `unshare` को नए PID नेमस्पेस बनाने के बाद एक नई प्रोसेस को फोर्क करने के लिए बनाता है।
- `%unshare -fp /bin/bash%` निष्पादित करने से सुनिश्चित होता है कि `unshare` कमांड स्वयं नए नेमस्पेस में PID 1 बन जाता है। `/bin/bash` और उसकी चाइल्ड प्रोसेस तब इस नए नेमस्पेस के भीतर सुरक्षित रूप से समाहित होती हैं, जिससे PID 1 के समय से पहले बाहर निकलने को रोका जाता है और सामान्य PID आवंटन की अनुमति दी जाती है।

`unshare` को `-f` फ्लैग के साथ चलाने से सुनिश्चित होता है कि नया PID नेमस्पेस सही ढंग से बनाए रखा जाता है, जिससे `/bin/bash` और उसकी सब-प्रोसेस मेमोरी आवंटन एरर का सामना किए बिना काम कर सकती हैं।

</details>

#### Docker
```bash
docker run -ti --name ubuntu1 -v /usr:/ubuntu1 ubuntu bash
```
### &#x20;जांचें कि आपकी प्रक्रिया किस नेमस्पेस में है
```bash
ls -l /proc/self/ns/uts
lrwxrwxrwx 1 root root 0 Apr  4 20:49 /proc/self/ns/uts -> 'uts:[4026531838]'
```
### सभी UTS नामस्थान खोजें

{% code overflow="wrap" %}
```bash
sudo find /proc -maxdepth 3 -type l -name uts -exec readlink {} \; 2>/dev/null | sort -u
# Find the processes with an specific namespace
sudo find /proc -maxdepth 3 -type l -name uts -exec ls -l  {} \; 2>/dev/null | grep <ns-number>
```
### UTS नेमस्पेस के अंदर प्रवेश करें
```bash
nsenter -u TARGET_PID --pid /bin/bash
```
इसके अलावा, आप किसी अन्य प्रक्रिया के **namespace में केवल तभी प्रवेश कर सकते हैं जब आप root हों**। और आप बिना किसी डिस्क्रिप्टर के जो इसकी ओर इशारा करता हो (जैसे `/proc/self/ns/uts`), अन्य namespace में **प्रवेश नहीं** कर सकते।

### Change hostname
```bash
unshare -u /bin/bash
hostname newhostname # Hostname won't be changed inside the host UTS ns
```
# संदर्भ
* [https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory](https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory)

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>
