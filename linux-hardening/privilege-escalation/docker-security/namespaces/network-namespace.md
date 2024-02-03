# नेटवर्क नेमस्पेस

<details>

<summary><strong>Learn AWS hacking from zero to hero with</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) को **फॉलो** करें.
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग ट्रिक्स शेयर करें.

</details>

## मूल जानकारी

नेटवर्क नेमस्पेस एक लिनक्स कर्नेल फीचर है जो नेटवर्क स्टैक का अलगाव प्रदान करता है, जिससे **प्रत्येक नेटवर्क नेमस्पेस का अपना स्वतंत्र नेटवर्क कॉन्फ़िगरेशन** हो सकता है, इंटरफेस, IP पते, रूटिंग टेबल्स, और फ़ायरवॉल नियम। यह अलगाव विभिन्न परिदृश्यों में उपयोगी है, जैसे कंटेनराइजेशन, जहां प्रत्येक कंटेनर का अपना नेटवर्क कॉन्फ़िगरेशन होना चाहिए, अन्य कंटेनरों और होस्ट सिस्टम से स्वतंत्र।

### यह कैसे काम करता है:

1. जब एक नया नेटवर्क नेमस्पेस बनाया जाता है, तो यह एक **पूरी तरह से अलग नेटवर्क स्टैक** के साथ शुरू होता है, जिसमें **कोई नेटवर्क इंटरफेस नहीं होते** सिवाय लूपबैक इंटरफेस (lo) के। इसका मतलब है कि नए नेटवर्क नेमस्पेस में चल रही प्रक्रियाएं डिफ़ॉल्ट रूप से अन्य नेमस्पेस या होस्ट सिस्टम में प्रक्रियाओं के साथ संवाद नहीं कर सकती हैं।
2. **वर्चुअल नेटवर्क इंटरफेस**, जैसे कि veth जोड़े, बनाए जा सकते हैं और नेटवर्क नेमस्पेस के बीच में स्थानांतरित किए जा सकते हैं। इससे नेमस्पेस के बीच या नेमस्पेस और होस्ट सिस्टम के बीच नेटवर्क कनेक्टिविटी स्थापित करना संभव होता है। उदाहरण के लिए, veth जोड़े का एक छोर किसी कंटेनर के नेटवर्क नेमस्पेस में रखा जा सकता है, और दूसरा छोर होस्ट नेमस्पेस में एक **ब्रिज** या अन्य नेटवर्क इंटरफेस से जुड़ा हो सकता है, जो कंटेनर को नेटवर्क कनेक्टिविटी प्रदान करता है।
3. एक नेमस्पेस के भीतर नेटवर्क इंटरफेस अपने **अपने IP पते, रूटिंग टेबल्स, और फ़ायरवॉल नियम** हो सकते हैं, अन्य नेमस्पेस से स्वतंत्र। इससे विभिन्न नेटवर्क नेमस्पेस में प्रक्रियाएं अलग-अलग नेटवर्क कॉन्फ़िगरेशन के साथ काम कर सकती हैं और ऐसे चल सकती हैं जैसे वे अलग-अलग नेटवर्क्ड सिस्टम्स पर चल रही हों।
4. प्रक्रियाएं `setns()` सिस्टम कॉल का उपयोग करके नेमस्पेस के बीच में जा सकती हैं, या `unshare()` या `clone()` सिस्टम कॉल्स का उपयोग करके `CLONE_NEWNET` फ्लैग के साथ नए नेमस्पेस बना सकती हैं। जब कोई प्रक्रिया नए नेमस्पेस में जाती है या एक बनाती है, तो वह उस नेमस्पेस से जुड़े नेटवर्क कॉन्फ़िगरेशन और इंटरफेस का उपयोग करना शुरू कर देगी।

## प्रयोगशाला:

### विभिन्न नेमस्पेस बनाएं

#### CLI
```bash
sudo unshare -n [--mount-proc] /bin/bash
# Run ifconfig or ip -a
```
`/proc` फाइलसिस्टम का एक नया उदाहरण माउंट करके, यदि आप `--mount-proc` पैरामीटर का उपयोग करते हैं, तो आप सुनिश्चित करते हैं कि नया माउंट नेमस्पेस उस नेमस्पेस के लिए विशिष्ट **प्रक्रिया जानकारी का सटीक और अलग दृश्य** रखता है।

<details>

<summary>एरर: bash: fork: Cannot allocate memory</summary>

जब `unshare` को `-f` विकल्प के बिना निष्पादित किया जाता है, तो लिनक्स नए PID (प्रोसेस ID) नेमस्पेस को संभालने के तरीके के कारण एक एरर का सामना किया जाता है। मुख्य विवरण और समाधान नीचे दिए गए हैं:

1. **समस्या की व्याख्या**:
- लिनक्स कर्नेल एक प्रक्रिया को `unshare` सिस्टम कॉल का उपयोग करके नए नेमस्पेस बनाने की अनुमति देता है। हालांकि, नए PID नेमस्पेस के निर्माण की पहल करने वाली प्रक्रिया (जिसे "unshare" प्रक्रिया कहा जाता है) नए नेमस्पेस में प्रवेश नहीं करती है; केवल उसकी चाइल्ड प्रोसेसेस ही करती हैं।
- `%unshare -p /bin/bash%` चलाने से `/bin/bash` `unshare` के समान प्रक्रिया में शुरू होता है। नतीजतन, `/bin/bash` और उसकी चाइल्ड प्रोसेसेस मूल PID नेमस्पेस में होती हैं।
- नए नेमस्पेस में `/bin/bash` की पहली चाइल्ड प्रोसेस PID 1 बन जाती है। जब यह प्रोसेस बाहर निकलती है, तो यदि कोई अन्य प्रोसेस नहीं हैं, तो यह नेमस्पेस की सफाई को ट्रिगर करती है, क्योंकि PID 1 की अनाथ प्रोसेसेस को गोद लेने की विशेष भूमिका होती है। लिनक्स कर्नेल तब उस नेमस्पेस में PID आवंटन को अक्षम कर देगा।

2. **परिणाम**:
- नए नेमस्पेस में PID 1 के बाहर निकलने से `PIDNS_HASH_ADDING` फ्लैग की सफाई हो जाती है। इससे `alloc_pid` फंक्शन नई प्रोसेस बनाते समय नए PID को आवंटित करने में विफल हो जाता है, जिससे "Cannot allocate memory" एरर होता है।

3. **समाधान**:
- इस समस्या को `-f` विकल्प के साथ `unshare` का उपयोग करके हल किया जा सकता है। यह विकल्प `unshare` को नए PID नेमस्पेस बनाने के बाद एक नई प्रक्रिया को फोर्क करने के लिए बनाता है।
- `%unshare -fp /bin/bash%` निष्पादित करने से सुनिश्चित होता है कि `unshare` कमांड स्वयं नए नेमस्पेस में PID 1 बन जाता है। `/bin/bash` और उसकी चाइल्ड प्रोसेसेस तब इस नए नेमस्पेस के भीतर सुरक्षित रूप से समाहित होती हैं, जिससे PID 1 के समय से पहले बाहर निकलने को रोका जा सकता है और सामान्य PID आवंटन की अनुमति दी जा सकती है।

`unshare` को `-f` फ्लैग के साथ चलाने से सुनिश्चित होता है कि नया PID नेमस्पेस सही ढंग से बनाए रखा जाता है, जिससे `/bin/bash` और उसकी सब-प्रोसेसेस बिना मेमोरी आवंटन एरर का सामना किए बिना काम कर सकती हैं।

</details>

#### Docker
```bash
docker run -ti --name ubuntu1 -v /usr:/ubuntu1 ubuntu bash
# Run ifconfig or ip -a
```
### &#x20;जांचें कि आपकी प्रक्रिया किस नेमस्पेस में है
```bash
ls -l /proc/self/ns/net
lrwxrwxrwx 1 root root 0 Apr  4 20:30 /proc/self/ns/net -> 'net:[4026531840]'
```
### सभी नेटवर्क नेमस्पेस खोजें

{% code overflow="wrap" %}
```bash
sudo find /proc -maxdepth 3 -type l -name net -exec readlink {} \; 2>/dev/null | sort -u | grep "net:"
# Find the processes with an specific namespace
sudo find /proc -maxdepth 3 -type l -name net -exec ls -l  {} \; 2>/dev/null | grep <ns-number>
```
{% endcode %}

### नेटवर्क नेमस्पेस के अंदर प्रवेश करें
```bash
nsenter -n TARGET_PID --pid /bin/bash
```
आप किसी अन्य प्रक्रिया के **namespace में केवल तभी प्रवेश कर सकते हैं जब आप root हों**। और आप बिना डिस्क्रिप्टर के अन्य namespace में **प्रवेश नहीं** कर सकते (जैसे `/proc/self/ns/net`).

# संदर्भ
* [https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory](https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory)

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert) के साथ शून्य से नायक तक AWS हैकिंग सीखें</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) को **फॉलो करें**।
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>
