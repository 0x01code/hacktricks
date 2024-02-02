# नेटवर्क नेमस्पेस

<details>

<summary><strong>Learn AWS hacking from zero to hero with</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) को **फॉलो** करें.
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स शेयर करें.

</details>

## मूल जानकारी

नेटवर्क नेमस्पेस एक लिनक्स कर्नेल फीचर है जो नेटवर्क स्टैक का अलगाव प्रदान करता है, जिससे **प्रत्येक नेटवर्क नेमस्पेस का अपना स्वतंत्र नेटवर्क कॉन्फ़िगरेशन** हो सकता है, इंटरफेस, IP पते, रूटिंग टेबल्स, और फ़ायरवॉल नियम। यह अलगाव विभिन्न परिदृश्यों में उपयोगी है, जैसे कंटेनराइजेशन, जहां प्रत्येक कंटेनर का अपना नेटवर्क कॉन्फ़िगरेशन होना चाहिए, अन्य कंटेनरों और होस्ट सिस्टम से स्वतंत्र।

### यह कैसे काम करता है:

1. जब एक नया नेटवर्क नेमस्पेस बनाया जाता है, तो यह एक **पूरी तरह से अलग नेटवर्क स्टैक** के साथ शुरू होता है, जिसमें **कोई नेटवर्क इंटरफेस नहीं होते** सिवाय लूपबैक इंटरफेस (lo) के। इसका मतलब है कि नए नेटवर्क नेमस्पेस में चल रही प्रक्रियाएं अन्य नेमस्पेस या होस्ट सिस्टम की प्रक्रियाओं के साथ संवाद नहीं कर सकती हैं।
2. **वर्चुअल नेटवर्क इंटरफेस**, जैसे कि veth जोड़े, बनाए जा सकते हैं और नेटवर्क नेमस्पेस के बीच में स्थानांतरित किए जा सकते हैं। इससे नेमस्पेस के बीच या नेमस्पेस और होस्ट सिस्टम के बीच नेटवर्क कनेक्टिविटी स्थापित करना संभव होता है। उदाहरण के लिए, veth जोड़े का एक छोर किसी कंटेनर के नेटवर्क नेमस्पेस में रखा जा सकता है, और दूसरा छोर होस्ट नेमस्पेस में एक **ब्रिज** या अन्य नेटवर्क इंटरफेस से जुड़ा हो सकता है, जो कंटेनर को नेटवर्क कनेक्टिविटी प्रदान करता है।
3. एक नेमस्पेस के भीतर नेटवर्क इंटरफेस अपने **खुद के IP पते, रूटिंग टेबल्स, और फ़ायरवॉल नियम** रख सकते हैं, अन्य नेमस्पेस से स्वतंत्र। इससे अलग-अलग नेटवर्क नेमस्पेस में प्रक्रियाएं विभिन्न नेटवर्क कॉन्फ़िगरेशन के साथ काम कर सकती हैं और ऐसे चल सकती हैं जैसे वे अलग-अलग नेटवर्क सिस्टम्स पर चल रही हों।
4. प्रक्रियाएं `setns()` सिस्टम कॉल का उपयोग करके नेमस्पेस के बीच में जा सकती हैं, या `unshare()` या `clone()` सिस्टम कॉल्स का उपयोग करके `CLONE_NEWNET` फ्लैग के साथ नए नेमस्पेस बना सकती हैं। जब कोई प्रक्रिया नए नेमस्पेस में जाती है या एक बनाती है, तो वह उस नेमस्पेस से जुड़े नेटवर्क कॉन्फ़िगरेशन और इंटरफेस का उपयोग करना शुरू कर देगी।

## प्रयोगशाला:

### विभिन्न नेमस्पेस बनाएं

#### CLI
```bash
sudo unshare -n [--mount-proc] /bin/bash
# Run ifconfig or ip -a
```
`--mount-proc` पैरामीटर का उपयोग करके नए `/proc` फाइलसिस्टम को माउंट करने से आप सुनिश्चित करते हैं कि नया माउंट नेमस्पेस उस नेमस्पेस की **विशिष्ट प्रक्रिया जानकारी का सटीक और अलग दृश्य प्रदान करता है**।

<details>

<summary>एरर: bash: fork: Cannot allocate memory</summary>

यदि आप पिछली लाइन `-f` के बिना चलाते हैं, तो आपको वह एरर मिलेगा।\
यह एरर इसलिए होता है क्योंकि नए नेमस्पेस में PID 1 प्रक्रिया समाप्त हो जाती है।

bash चलने के बाद, bash कई नए सब-प्रोसेस फोर्क करेगा कुछ काम करने के लिए। यदि आप unshare को `-f` के बिना चलाते हैं, तो bash का समान pid होगा जैसा कि वर्तमान "unshare" प्रक्रिया का है। वर्तमान "unshare" प्रक्रिया unshare सिस्टमकॉल को कॉल करती है, एक नया pid नेमस्पेस बनाती है, लेकिन वर्तमान "unshare" प्रक्रिया नए pid नेमस्पेस में नहीं होती है। यह लिनक्स कर्नेल का वांछित व्यवहार है: प्रक्रिया A एक नया नेमस्पेस बनाती है, प्रक्रिया A स्वयं नए नेमस्पेस में नहीं डाली जाती है, केवल प्रक्रिया A की सब-प्रोसेस ही नए नेमस्पेस में डाली जाती हैं। इसलिए जब आप चलाते हैं:
```
unshare -p /bin/bash
```
unshare प्रक्रिया /bin/bash को exec करेगी, और /bin/bash कई सब-प्रोसेस फोर्क करेगा, पहला सब-प्रोसेस बैश का नए नेमस्पेस का PID 1 बन जाएगा, और सबप्रोसेस अपना काम पूरा करने के बाद बाहर निकल जाएगा। इसलिए नए नेमस्पेस का PID 1 बाहर निकल जाता है।

PID 1 प्रक्रिया का एक विशेष कार्य होता है: यह सभी अनाथ प्रक्रियाओं का माता-पिता प्रक्रिया बनना चाहिए। अगर रूट नेमस्पेस में PID 1 प्रक्रिया बाहर निकल जाती है, तो कर्नेल पैनिक हो जाएगा। अगर एक सब नेमस्पेस में PID 1 प्रक्रिया बाहर निकल जाती है, तो लिनक्स कर्नेल disable\_pid\_allocation फंक्शन को कॉल करेगा, जो उस नेमस्पेस में PIDNS\_HASH\_ADDING फ्लैग को साफ कर देगा। जब लिनक्स कर्नेल एक नई प्रक्रिया बनाता है, कर्नेल alloc\_pid फंक्शन को कॉल करेगा ताकि एक नेमस्पेस में PID आवंटित कर सके, और अगर PIDNS\_HASH\_ADDING फ्लैग सेट नहीं है, तो alloc\_pid फंक्शन -ENOMEM त्रुटि लौटाएगा। इसीलिए आपको "Cannot allocate memory" त्रुटि मिली।

आप इस समस्या को '-f' विकल्प का उपयोग करके हल कर सकते हैं:
```
unshare -fp /bin/bash
```
यदि आप '-f' विकल्प के साथ unshare चलाते हैं, तो unshare नया pid namespace बनाने के बाद एक नई प्रक्रिया को fork करेगा। और नई प्रक्रिया में /bin/bash चलाएगा। नई प्रक्रिया नए pid namespace की pid 1 होगी। फिर bash कई उप-प्रक्रियाओं को कुछ कार्य करने के लिए fork करेगा। चूंकि bash स्वयं नए pid namespace की pid 1 है, इसकी उप-प्रक्रियाएं बिना किसी समस्या के बाहर निकल सकती हैं।

[https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory](https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory) से कॉपी किया गया

</details>

#### Docker
```bash
docker run -ti --name ubuntu1 -v /usr:/ubuntu1 ubuntu bash
# Run ifconfig or ip -a
```
### &#x20;जांचें कि आपकी प्रक्रिया किस namespace में है
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
### नेटवर्क नेमस्पेस के अंदर प्रवेश करें
```bash
nsenter -n TARGET_PID --pid /bin/bash
```
```markdown
आप किसी अन्य प्रक्रिया के **namespace में केवल तभी प्रवेश कर सकते हैं जब आप root हों**। और आप बिना डिस्क्रिप्टर के अन्य namespace में **प्रवेश नहीं** कर सकते (जैसे `/proc/self/ns/net`).

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert) के साथ AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) को **फॉलो** करें।
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग ट्रिक्स शेयर करें।

</details>
```
