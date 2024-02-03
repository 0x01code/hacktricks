# समय नामस्थान

<details>

<summary><strong> AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) में या **Twitter** 🐦 पर **मुझे फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपोज़ में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>

## मूल जानकारी

समय नामस्थान सिस्टम मोनोटोनिक और बूट-टाइम घड़ियों के लिए प्रति-नामस्थान ऑफसेट की अनुमति देता है। समय नामस्थान लिनक्स कंटेनरों के उपयोग के लिए उपयुक्त है ताकि कंटेनर के भीतर तारीख/समय बदली जा सके और चेकपॉइंट/स्नैपशॉट से पुनर्स्थापना के बाद कंटेनर के भीतर घड़ियों को समायोजित किया जा सके।

## प्रयोगशाला:

### विभिन्न नामस्थान बनाएँ

#### CLI
```bash
sudo unshare -T [--mount-proc] /bin/bash
```
`/proc` फाइलसिस्टम का एक नया इंस्टेंस माउंट करके, यदि आप `--mount-proc` पैरामीटर का उपयोग करते हैं, तो आप सुनिश्चित करते हैं कि नया माउंट नेमस्पेस उस नेमस्पेस के लिए विशिष्ट प्रोसेस जानकारी का **सटीक और अलग दृश्य** प्रदान करता है।

<details>

<summary>एरर: bash: fork: Cannot allocate memory</summary>

`unshare` को `-f` विकल्प के बिना निष्पादित करने पर, एक एरर का सामना करना पड़ता है, जो लिनक्स नए PID (प्रोसेस ID) नेमस्पेस को कैसे हैंडल करता है, उसके कारण होता है। मुख्य विवरण और समाधान नीचे दिए गए हैं:

1. **समस्या की व्याख्या**:
- लिनक्स कर्नेल एक प्रोसेस को `unshare` सिस्टम कॉल का उपयोग करके नए नेमस्पेस बनाने की अनुमति देता है। हालांकि, नए PID नेमस्पेस के निर्माण की पहल करने वाला प्रोसेस (जिसे "unshare" प्रोसेस कहा जाता है) नए नेमस्पेस में प्रवेश नहीं करता; केवल उसके चाइल्ड प्रोसेस ही करते हैं।
- `%unshare -p /bin/bash%` चलाने से `/bin/bash` `unshare` के समान प्रोसेस में शुरू होता है। नतीजतन, `/bin/bash` और उसके चाइल्ड प्रोसेस मूल PID नेमस्पेस में होते हैं।
- नए नेमस्पेस में `/bin/bash` का पहला चाइल्ड प्रोसेस PID 1 बन जाता है। जब यह प्रोसेस बाहर निकलता है, तो यदि कोई अन्य प्रोसेस नहीं हैं, तो नेमस्पेस की सफाई ट्रिगर होती है, क्योंकि PID 1 का अनाथ प्रोसेस को गोद लेने का विशेष रोल होता है। लिनक्स कर्नेल तब उस नेमस्पेस में PID आवंटन को अक्षम कर देगा।

2. **परिणाम**:
- नए नेमस्पेस में PID 1 के बाहर निकलने से `PIDNS_HASH_ADDING` फ्लैग की सफाई हो जाती है। इससे `alloc_pid` फंक्शन नए प्रोसेस बनाते समय नए PID को आवंटित करने में असफल हो जाता है, जिससे "Cannot allocate memory" एरर होता है।

3. **समाधान**:
- इस समस्या को `-f` विकल्प के साथ `unshare` का उपयोग करके हल किया जा सकता है। यह विकल्प `unshare` को नए PID नेमस्पेस बनाने के बाद एक नए प्रोसेस को फोर्क करने के लिए बनाता है।
- `%unshare -fp /bin/bash%` निष्पादित करने से सुनिश्चित होता है कि `unshare` कमांड स्वयं नए नेमस्पेस में PID 1 बन जाता है। `/bin/bash` और उसके चाइल्ड प्रोसेस तब इस नए नेमस्पेस के भीतर सुरक्षित रूप से समाहित होते हैं, जिससे PID 1 के समय से पहले बाहर निकलने को रोका जाता है और सामान्य PID आवंटन की अनुमति दी जाती है।

`unshare` को `-f` फ्लैग के साथ चलाने से सुनिश्चित होता है कि नया PID नेमस्पेस सही ढंग से बनाए रखा जाता है, जिससे `/bin/bash` और उसके उप-प्रोसेस बिना मेमोरी आवंटन एरर का सामना किए बिना काम कर सकते हैं।

</details>

#### Docker
```bash
docker run -ti --name ubuntu1 -v /usr:/ubuntu1 ubuntu bash
```
### &#x20;जांचें कि आपकी प्रक्रिया किस namespace में है
```bash
ls -l /proc/self/ns/time
lrwxrwxrwx 1 root root 0 Apr  4 21:16 /proc/self/ns/time -> 'time:[4026531834]'
```
### सभी Time namespaces खोजें

{% code overflow="wrap" %}
```bash
sudo find /proc -maxdepth 3 -type l -name time -exec readlink {} \; 2>/dev/null | sort -u
# Find the processes with an specific namespace
sudo find /proc -maxdepth 3 -type l -name time -exec ls -l  {} \; 2>/dev/null | grep <ns-number>
```
{% endcode %}

### टाइम नेमस्पेस के अंदर प्रवेश करें
```bash
nsenter -T TARGET_PID --pid /bin/bash
```
```markdown
आप किसी अन्य प्रक्रिया के **namespace में केवल तभी प्रवेश कर सकते हैं जब आप root हों**। और आप किसी अन्य namespace में **प्रवेश नहीं कर सकते** **बिना एक descriptor के** जो उसकी ओर इशारा करता हो (जैसे `/proc/self/ns/net`).

# संदर्भ
* [https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory](https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory)

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert) के साथ शून्य से नायक तक AWS हैकिंग सीखें</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) में या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) **का अनुसरण करें**।
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>
```
