# UTS नेमस्पेस

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करना चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल हों** या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)** का पालन करें**.
* **अपने हैकिंग ट्रिक्स को** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में पीआर जमा करके** अपना योगदान दें।

</details>

## मूलभूत जानकारी

एक UTS (UNIX Time-Sharing System) नेमस्पेस एक लिनक्स कर्नल सुविधा है जो दो सिस्टम पहचानकर्ताओं की **अलगाव** प्रदान करती है: **होस्टनेम** और **NIS** (नेटवर्क सूचना सेवा) डोमेन नाम। यह अलगाव प्रत्येक UTS नेमस्पेस को अपने **स्वतंत्र होस्टनेम और NIS डोमेन नाम** के साथ प्रदान करता है, जो खासकर कंटेनरीकरण स्थितियों में उपयोगी होता है जहां प्रत्येक कंटेनर एक अलग सिस्टम के रूप में प्रदर्शित होना चाहिए।

### काम करने का तरीका:

1. जब एक नया UTS नेमस्पेस बनाया जाता है, तो यह अपने मूल नेमस्पेस से **होस्टनेम और NIS डोमेन नाम की प्रतिलिपि के साथ शुरू होता है**। इसका मतलब है कि, नई नेमस्पेस को निर्माण के समय, नया नेमस्पेस **अपने मूल नेमस्पेस के साथ समान पहचानकर्ताओं का साझा करता है**। हालांकि, नेमस्पेस के भीतर होस्टनेम या NIS डोमेन नाम में किसी भी बादलाव का प्रभाव अन्य नेमस्पेस पर नहीं होगा।
2. UTS नेमस्पेस के भीतर कार्यप्रणाली **`sethostname()` और `setdomainname()` सिस्टम कॉल** का उपयोग करके होस्टनेम और NIS डोमेन नाम को बदल सकती है। ये बदलाव नेमस्पेस के लिए स्थानिक होते हैं और अन्य नेमस्पेस या होस्ट सिस्टम पर प्रभाव नहीं डालते हैं।
3. कार्यप्रणाली `setns()` का उपयोग करके नेमस्पेस के बीच ले जा सकती है या `unshare()` या `clone()` सिस्टम कॉल का उपयोग करके नए नेमस्पेस बना सकती है। जब कोई प्रक्रिया नए नेमस्पेस में जाती है या एक नया नेमस्पेस बनाती है, तो वह उस नेमस्पेस के साथ जुड़े होस्टनेम और NIS डोमेन नाम का उपयोग करना शुरू कर देगी।

## प्रयोगशाला:

### विभिन्न नेमस्पेस बनाएँ

#### CLI
```bash
sudo unshare -u [--mount-proc] /bin/bash
```
यदि आप `--mount-proc` पैरामीटर का उपयोग करते हैं, तो `/proc` फ़ाइल सिस्टम के एक नए इंस्टेंस को माउंट करके आप सुनिश्चित करते हैं कि नया माउंट नेमस्पेस उस नेमस्पेस के लिए प्रक्रिया सूचना का सटीक और अलगाववित दृश्य होता है।

<details>

<summary>त्रुटि: bash: fork: मेमोरी आवंटित नहीं कर सकता</summary>

यदि आप `-f` के बिना पिछली पंक्ति को चलाते हैं, तो आप उस त्रुटि को प्राप्त करेंगे।\
यह त्रुटि नए नेमस्पेस में प्रोसेस 1 की प्रक्रिया के बाहर निकलने के कारण होती है।

जब बैश चलने लगता है, तो बैश कई नई सब-प्रोसेस को कुछ करने के लिए फोर्क करेगा। यदि आप -f के बिना अनशेयर चलाते हैं, तो बैश का पिडी वर्तमान "अनशेयर" प्रक्रिया के समान होगा। वर्तमान "अनशेयर" प्रक्रिया अनशेयर सिस्टम कॉल करती है, एक नया पिडी नेमस्पेस बनाती है, लेकिन वर्तमान "अनशेयर" प्रक्रिया नए पिडी नेमस्पेस में नहीं है। यह लिनक्स कर्नल का वांछित व्यवहार है: प्रक्रिया एक नया नेमस्पेस बनाती है, प्रक्रिया खुद नए नेमस्पेस में नहीं रखी जाएगी, केवल प्रक्रिया के सब-प्रोसेस ही नए नेमस्पेस में रखे जाएंगे। इसलिए जब आप चलाते हैं:
```
unshare -p /bin/bash
```
अनशेयर प्रक्रिया /बिन/बैश को एक्जेक्यूट करेगी, और /बिन/बैश कई सब-प्रक्रियाएं फोर्क करेगा, बैश की पहली सब-प्रक्रिया नए नेमस्पेस का पीआईडी 1 बन जाएगी, और सब-प्रक्रिया अपना काम पूरा करने के बाद बंद हो जाएगी। इसलिए नए नेमस्पेस का पीआईडी 1 बंद हो जाता है।

पीआईडी 1 प्रक्रिया का एक विशेष कार्य होता है: यह सभी अनाथ प्रक्रियाओं के माता-पिता प्रक्रिया बनना चाहिए। अगर रूट नेमस्पेस में पीआईडी 1 प्रक्रिया बंद हो जाती है, तो कर्नल पैनिक हो जाता है। अगर एक सब-नेमस्पेस में पीआईडी 1 प्रक्रिया बंद हो जाती है, तो लिनक्स कर्नल disable\_pid\_allocation फंक्शन को कॉल करेगा, जो उस नेमस्पेस में PIDNS\_HASH\_ADDING फ्लैग को साफ करेगा। जब लिनक्स कर्नल एक नई प्रक्रिया बनाता है, तो कर्नल एक नेमस्पेस में एक पीआईडी आवंटित करने के लिए alloc\_pid फंक्शन को कॉल करेगा, और अगर PIDNS\_HASH\_ADDING फ्लैग सेट नहीं है, तो alloc\_pid फंक्शन -ENOMEM त्रुटि लौटाएगा। इसीलिए आपको "Cannot allocate memory" त्रुटि मिली है।

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
$ cat /proc/$$/ns/uts
```

आपकी प्रक्रिया किस नेमस्पेस में है यह जानने के लिए, निम्नलिखित कमांड का उपयोग करें:

```bash
$ cat /proc/$$/ns/uts
```
```bash
ls -l /proc/self/ns/uts
lrwxrwxrwx 1 root root 0 Apr  4 20:49 /proc/self/ns/uts -> 'uts:[4026531838]'
```
### सभी UTS नेमस्पेस ढूंढ़ें

{% code overflow="wrap" %}
```bash
sudo find /proc -maxdepth 3 -type l -name uts -exec readlink {} \; 2>/dev/null | sort -u
# Find the processes with an specific namespace
sudo find /proc -maxdepth 3 -type l -name uts -exec ls -l  {} \; 2>/dev/null | grep <ns-number>
```
{% code %}

### एक UTS नेमस्पेस में अंदर जाएं

{% endcode %}
```bash
nsenter -u TARGET_PID --pid /bin/bash
```
इसके अलावा, आप केवल रूट यूजर होने की स्थिति में ही **दूसरे प्रक्रिया नेमस्पेस में प्रवेश कर सकते हैं**। और आप **बिना उसके एक डिस्क्रिप्टर के** (जैसे `/proc/self/ns/uts`) **उस नेमस्पेस में प्रवेश नहीं कर सकते हैं**।

### होस्टनेम बदलें
```bash
unshare -u /bin/bash
hostname newhostname # Hostname won't be changed inside the host UTS ns
```
<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करना चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **या** मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें और PRs सबमिट करें** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को।**

</details>
