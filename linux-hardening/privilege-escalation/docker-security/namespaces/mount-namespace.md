# माउंट नेमस्पेस

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** द्वारा PRs सबमिट करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

## मूल जानकारी

माउंट नेमस्पेस एक लिनक्स कर्नेल सुविधा है जो एक समूह के प्रक्रियाओं द्वारा देखे गए फ़ाइल सिस्टम माउंट पॉइंट का अलगीकरण प्रदान करती है। प्रत्येक माउंट नेमस्पेस के अपने फ़ाइल सिस्टम माउंट पॉइंट का सेट होता है, और **एक नेमस्पेस में माउंट पॉइंट्स में परिवर्तन अन्य नेमस्पेस को प्रभावित नहीं करते**। इसका मतलब है कि विभिन्न माउंट नेमस्पेस में चल रही प्रक्रियाएं फ़ाइल सिस्टम हायरार्की का विभिन्न दृश्य रख सकती हैं।

माउंट नेमस्पेस कंटेनरीकरण में विशेष रूप से उपयुक्त है, जहाँ प्रत्येक कंटेनर को अपना खुद का फ़ाइल सिस्टम और विन्यास होना चाहिए, अन्य कंटेनर और होस्ट सिस्टम से अलग।

### कैसे काम करता है:

1. जब एक नया माउंट नेमस्पेस बनाया जाता है, तो इसे उसके माता-पिता नेमस्पेस से माउंट पॉइंट्स की **कॉपी के साथ आरंभ किया जाता है**। इसका मतलब है कि, निर्माण के समय, नया नेमस्पेस फ़ाइल सिस्टम का एक समान दृश्य साझा करता है जैसे कि उसके माता-पिता। हालांकि, नेमस्पेस के भीतर माउंट पॉइंट्स में किए गए किसी भी भविष्यवाणी परिवर्तन माता-पिता या अन्य नेमस्पेस को प्रभावित नहीं करेगा।
2. जब कोई प्रक्रिया अपने नेमस्पेस में माउंट पॉइंट को संशोधित करती है, जैसे कि फ़ाइल सिस्टम को माउंट करना या अनमाउंट करना, तो **परिवर्तन उस नेमस्पेस के लिए स्थानीय होता है** और अन्य नेमस्पेस पर प्रभावित नहीं होता। यह हर नेमस्पेस को अपनी स्वतंत्र फ़ाइल सिस्टम हायरार्की देने की अनुमति देता है।
3. प्रक्रियाएं `setns()` सिस्टम कॉल का उपयोग करके नेमस्पेस के बीच ले जा सकती हैं, या `unshare()` या `clone()` सिस्टम कॉल का उपयोग करके `CLONE_NEWNS` फ़्लैग के साथ नए नेमस्पेस बना सकती हैं। जब कोई प्रक्रिया नए नेमस्पेस में जाती है या एक नया बनाती है, तो वह उस नेमस्पेस से संबंधित माउंट पॉइंट्स का उपयोग करना शुरू करेगी।
4. **फ़ाइल डिस्क्रिप्टर और इनोड्स नेमस्पेस के बीच साझा किए जाते हैं**, इसका मतलब है कि यदि एक प्रक्रिया एक नेमस्पेस में एक फ़ाइल को दिखाने वाला खुला फ़ाइल डिस्क्रिप्टर रखती है, तो वह प्रक्रिया दूसरे नेमस्पेस में एक प्रक्रिया को **उस फ़ाइल डिस्क्रिप्टर को पास** कर सकती है, और **दोनों प्रक्रियाएं वही फ़ाइल एक्सेस करेंगी**। हालांकि, माउंट पॉइंट्स में अंतर के कारण दोनों नेमस्पेस में फ़ाइल का पथ समान नहीं हो सकता।

## लैब:

### विभिन्न नेमस्पेस बनाएँ

#### CLI
```bash
sudo unshare -m [--mount-proc] /bin/bash
```
यदि आप `--mount-proc` पैरामीटर का उपयोग करके `/proc` फ़ाइल सिस्टम का एक नया इंस्टेंस माउंट करते हैं, तो आप सुनिश्चित करते हैं कि नया माउंट नेमस्पेस उस नेमस्पेस के लिए प्रक्रिया सूचना का सटीक और अलगजीकृत दृश्य रखता है।
```bash
docker run -ti --name ubuntu1 -v /usr:/ubuntu1 ubuntu bash
```
### &#x20;जांचें कि आपकी प्रक्रिया किस नेमस्पेस में है
```bash
ls -l /proc/self/ns/mnt
lrwxrwxrwx 1 root root 0 Apr  4 20:30 /proc/self/ns/mnt -> 'mnt:[4026531841]'
```
### सभी माउंट नेमस्पेस खोजें

{% code overflow="wrap" %}
```bash
sudo find /proc -maxdepth 3 -type l -name mnt -exec readlink {} \; 2>/dev/null | sort -u
# Find the processes with an specific namespace
sudo find /proc -maxdepth 3 -type l -name mnt -exec ls -l  {} \; 2>/dev/null | grep <ns-number>
```
{% endcode %}

### माउंट नेमस्पेस के अंदर जाएं
```bash
nsenter -m TARGET_PID --pid /bin/bash
```
आप केवल **रूट होने पर** **दूसरे प्रक्रिया नेमस्पेस में प्रवेश** कर सकते हैं। और आप **उसमें प्रवेश नहीं** कर सकते **बिना एक डिस्क्रिप्टर** के जो इसे पॉइंट करता है (जैसे `/proc/self/ns/mnt`).

क्योंकि नए माउंट्स केवल नेमस्पेस के भीतर एक्सेस करने योग्य होते हैं, इससे संभव है कि एक नेमस्पेस में संवेदनशील जानकारी हो जो केवल उससे ही एक्सेस की जा सकती है।

### कुछ माउंट करें
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
## संदर्भ
* [https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory](https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory)


<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** हैकट्रिक्स और हैकट्रिक्स क्लाउड github रेपो में PRs सबमिट करके।

</details>
