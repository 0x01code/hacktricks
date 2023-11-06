# IPC नेमस्पेस

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **फॉलो** करें मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को**

</details>

## मूलभूत जानकारी

IPC (Inter-Process Communication) नेमस्पेस एक लिनक्स कर्नल सुविधा है जो सिस्टम V IPC ऑब्जेक्ट्स, जैसे संदेश कतारें, साझा मेमोरी सेगमेंट्स और सेमाफोर्स, की **अलगाव** प्रदान करती है। यह अलगाव सुनिश्चित करता है कि **अलग IPC नेमस्पेस में कार्यरत प्रक्रियाएं सीधे रूप से एक-दूसरे के IPC ऑब्जेक्ट्स तक पहुंच या उन्हें संशोधित नहीं कर सकती हैं**, प्रक्रिया समूहों के बीच एक अतिरिक्त सुरक्षा और गोपनीयता की स्तर प्रदान करते हुए।

### काम कैसे करता है:

1. जब एक नया IPC नेमस्पेस बनाया जाता है, तो इसके पास एक **पूरी तरह से अलगावशील सेट का सिस्टम V IPC ऑब्जेक्ट्स** होता है। इसका मतलब है कि नए IPC नेमस्पेस में चल रही प्रक्रियाएं डिफ़ॉल्ट रूप से अन्य नेमस्पेस या होस्ट सिस्टम के IPC ऑब्जेक्ट्स तक पहुंच या उनमें हस्तक्षेप करने के लिए पहुंच नहीं हो सकती हैं।
2. नेमस्पेस के भीतर बनाए गए IPC ऑब्जेक्ट्स केवल उस नेमस्पेस के भीतर की प्रक्रियाओं के लिए दृश्यमान और **पहुंचने योग्य** होते हैं। प्रत्येक IPC ऑब्जेक्ट अपने नेमस्पेस के भीतर एक अद्वितीय कुंजी द्वारा पहचाना जाता है। हालांकि कुंजी अलग-अलग नेमस्पेस में एक ही हो सकती है, ऑब्जेक्ट्स स्वयं अलगावशील होते हैं और नेमस्पेस के बीच पहुंच नहीं हो सकते हैं।
3. प्रक्रियाएं `setns()` सिस्टम कॉल का उपयोग करके नेमस्पेस के बीच जा सकती हैं या `unshare()` या `clone()` सिस्टम कॉल का उपयोग करके नए नेमस्पेस बना सकती हैं जिसमें `CLONE_NEWIPC` फ़्लैग होता है। जब कोई प्रक्रिया नए नेमस्पेस में जाती है या एक नया नेमस्पेस बनाती है, तो वह उस नेमस्पेस से जुड़े IPC ऑब्जेक्ट्स का उपयोग करना शुरू कर देती है।

## प्रयोगशाला:

### विभिन्न नेमस्पेस बनाएँ

#### CLI
```bash
sudo unshare -i [--mount-proc] /bin/bash
```
यदि आप `--mount-proc` पैरामीटर का उपयोग करते हैं, तो `/proc` फ़ाइल सिस्टम के एक नए इंस्टेंस को माउंट करके आप सुनिश्चित करते हैं कि नया माउंट नेमस्पेस उस नेमस्पेस के लिए प्रक्रिया सूचना का सटीक और अलगाववित दृश्य होता है।

<details>

<summary>त्रुटि: bash: fork: मेमोरी आवंटित नहीं कर सकता</summary>

यदि आप `-f` के बिना पिछली पंक्ति को चलाते हैं, तो आपको यह त्रुटि मिलेगी।\
यह त्रुटि नए नेमस्पेस में प्रोसेस 1 की प्रक्रिया के बाहर निकलने के कारण होती है।

जब बैश चलने लगता है, तो बैश कई नई सब-प्रोसेस को फोर्क करके कुछ काम करने के लिए बनाता है। यदि आप -f के बिना अनशेयर चलाते हैं, तो बैश का पिडी वर्तमान "अनशेयर" प्रक्रिया के साथ समान होगा। वर्तमान "अनशेयर" प्रक्रिया अनशेयर सिस्टम कॉल करती है, एक नया पिडी नेमस्पेस बनाती है, लेकिन वर्तमान "अनशेयर" प्रक्रिया नए पिडी नेमस्पेस में नहीं है। यह लिनक्स कर्नल का वांछित व्यवहार है: प्रक्रिया एक नया नेमस्पेस बनाती है, प्रक्रिया खुद नए नेमस्पेस में नहीं रखी जाएगी, केवल प्रक्रिया के उप-प्रक्रियाएं नए नेमस्पेस में रखी जाएंगी। इसलिए जब आप चलाते हैं:
```
unshare -p /bin/bash
```
अनशेयर प्रक्रिया /बिन/बैश को एक्जेक्यूट करेगी, और /बिन/बैश कई सब-प्रक्रियाएं फोर्क करेगा, बैश की पहली सब-प्रक्रिया नए नेमस्पेस का पीआईडी 1 बन जाएगी, और सब-प्रक्रिया अपना काम पूरा करने के बाद बंद हो जाएगी। इसलिए नए नेमस्पेस का पीआईडी 1 बंद हो जाता है।

पीआईडी 1 प्रक्रिया का एक विशेष कार्य होता है: यह सभी अनाथ प्रक्रियाओं के माता-पिता प्रक्रिया बनना चाहिए। अगर रूट नेमस्पेस में पीआईडी 1 प्रक्रिया बंद हो जाती है, तो कर्नल पैनिक हो जाता है। अगर एक सब नेमस्पेस में पीआईडी 1 प्रक्रिया बंद हो जाती है, तो लिनक्स कर्नल disable\_pid\_allocation फंक्शन को कॉल करेगा, जो उस नेमस्पेस में PIDNS\_HASH\_ADDING फ्लैग को साफ करेगा। जब लिनक्स कर्नल एक नई प्रक्रिया बनाता है, तो कर्नल नेमस्पेस में एक पीआईडी आवंटित करने के लिए alloc\_pid फंक्शन को कॉल करेगा, और अगर PIDNS\_HASH\_ADDING फ्लैग सेट नहीं है, तो alloc\_pid फंक्शन -ENOMEM त्रुटि लौटाएगा। इसीलिए आपको "Cannot allocate memory" त्रुटि मिली है।

आप '-f' विकल्प का उपयोग करके इस समस्या को हल कर सकते हैं:
```
unshare -fp /bin/bash
```
यदि आप '-f' विकल्प के साथ unshare चलाते हैं, तो unshare नए pid नेमस्पेस बनाने के बाद एक नई प्रक्रिया फोर्क करेगा। और नई प्रक्रिया में /bin/bash चलाएगा। नई प्रक्रिया नए pid नेमस्पेस का pid 1 होगी। फिर बैश भी कुछ कार्यों के लिए कई उप-प्रक्रियाएं फोर्क करेगा। बैश खुद नए pid नेमस्पेस का pid 1 होने के कारण, इसकी उप-प्रक्रियाएं किसी भी समस्या के बिना बंद हो सकती हैं।

[https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory](https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory)

</details>

#### डॉकर
```bash
docker run -ti --name ubuntu1 -v /usr:/ubuntu1 ubuntu bash
```
### &#x20;जांचें कि आपकी प्रक्रिया किस नेमस्पेस में है

To check which namespace your process is in, use the following command:

```bash
ls -l /proc/$$/ns/ipc
```

आपकी प्रक्रिया किस नेमस्पेस में है यह जानने के लिए, निम्नलिखित कमांड का उपयोग करें:

```bash
ls -l /proc/$$/ns/ipc
```
```bash
ls -l /proc/self/ns/ipc
lrwxrwxrwx 1 root root 0 Apr  4 20:37 /proc/self/ns/ipc -> 'ipc:[4026531839]'
```
### सभी IPC नेमस्पेस ढूंढें

{% code overflow="wrap" %}
```bash
sudo find /proc -maxdepth 3 -type l -name ipc -exec readlink {} \; 2>/dev/null | sort -u
# Find the processes with an specific namespace
sudo find /proc -maxdepth 3 -type l -name ipc -exec ls -l  {} \; 2>/dev/null | grep <ns-number>
```
{% code %}

### IPC नेमस्पेस में अंदर जाएं

{% endcode %}
```bash
nsenter -i TARGET_PID --pid /bin/bash
```
इसके अलावा, आप केवल रूट यूजर होने की स्थिति में ही दूसरे प्रक्रिया नेमस्पेस में प्रवेश कर सकते हैं। और आप बिना उसके एक डिस्क्रिप्टर के (जैसे `/proc/self/ns/net`) इसके दूसरे नेमस्पेस में प्रवेश नहीं कर सकते हैं।

### IPC ऑब्जेक्ट बनाएँ
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

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित करना चाहते हैं**? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करना है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT संग्रह**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को।**

</details>
