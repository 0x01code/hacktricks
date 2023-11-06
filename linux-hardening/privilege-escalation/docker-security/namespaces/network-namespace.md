# नेटवर्क नेमस्पेस

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करना चाहिए? [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**official PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**telegram समूह**](https://t.me/peass) या **फॉलो** करें मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को**

</details>

## मूलभूत जानकारी

नेटवर्क नेमस्पेस एक लिनक्स कर्नल सुविधा है जो नेटवर्क स्टैक की अलगाव प्रदान करती है, जिससे **प्रत्येक नेटवर्क नेमस्पेस को अपनी स्वतंत्र नेटवर्क कॉन्फ़िगरेशन**, इंटरफ़ेस, IP पते, रूटिंग टेबल और फ़ायरवॉल नियम हो सकते हैं। यह अलगाव विभिन्न स्थितियों में उपयोगी होता है, जैसे कि कंटेनरीकरण, जहां प्रत्येक कंटेनर को अपनी नेटवर्क कॉन्फ़िगरेशन होनी चाहिए, अन्य कंटेनर और होस्ट सिस्टम के अलगाव में।

### काम कैसे करता है:

1. जब एक नया नेटवर्क नेमस्पेस बनाया जाता है, तो इसके पास एक **पूरी तरह से अलगाव वाला नेटवर्क स्टैक** होता है, **कोई नेटवर्क इंटरफ़ेस नहीं** होता है केवल लूपबैक इंटरफ़ेस (lo) के लिए। इसका मतलब है कि नए नेटवर्क नेमस्पेस में चल रहे प्रक्रियाएँ डिफ़ॉल्ट रूप से अन्य नेमस्पेस या होस्ट सिस्टम के प्रक्रियाओं के साथ संवाद करने में असमर्थ होती हैं।
2. **वर्चुअल नेटवर्क इंटरफ़ेस**, जैसे कि veth पेयर, बनाए जा सकते हैं और नेटवर्क नेमस्पेस के बीच या नेमस्पेस और होस्ट सिस्टम के बीच नेटवर्क कनेक्टिविटी स्थापित करने के लिए ले जाए जा सकते हैं। उदाहरण के लिए, veth पेयर का एक अंत कंटेनर के नेटवर्क नेमस्पेस में रखा जा सकता है, और दूसरा अंत होस्ट नेमस्पेस में एक **ब्रिज** या दूसरे नेटवर्क इंटरफ़ेस से जुड़ा हो सकता है, जो कंटेनर को नेटवर्क कनेक्टिविटी प्रदान करता है।
3. नेमस्पेस के भीतर नेटवर्क इंटरफ़ेस के पास अपने **खुद के IP पते, रूटिंग टेबल और फ़ायरवॉल नियम** हो सकते हैं, अन्य नेमस्पेस के अलगाव में। इससे अलग नेटवर्क नेमस्पेस में चल रही प्रक्रियाएँ अलग-अलग नेटवर्क कॉन्फ़िगरेशन रख सकती हैं और ऐसा लग सकता है कि वे अलग-अलग नेटवर्क सिस्टम पर चल रही हैं।
4. प्रक्रियाएँ `setns()` सिस्टम कॉल का उपयोग करके नेमस्पेस के बीच जा सकती हैं, या `unshare()` या `clone()` सिस्टम कॉल का उपयोग करके नए नेमस्पेस बना सकती हैं `CLONE_NEWNET` फ़्लैग के साथ। जब कोई प्रक्रिया नए नेमस्पेस में जाती है या एक नया नेमस्पेस बनाती है, तो वह उस नेमस्पेस के साथ संबंधित नेटवर्क कॉन्फ़िगरेशन और इंटरफ़ेस का उपयोग करना शुरू कर देगी।

## प्रयोगशाला:

### विभिन्न नेमस्पेस बनाएँ

#### CLI
```bash
sudo unshare -n [--mount-proc] /bin/bash
# Run ifconfig or ip -a
```
यदि आप `--mount-proc` पैरामीटर का उपयोग करते हैं, तो `/proc` फ़ाइल सिस्टम के एक नए इंस्टेंस को माउंट करके आप सुनिश्चित करते हैं कि नया माउंट नेमस्पेस उस नेमस्पेस के लिए प्रक्रिया सूचना का सटीक और अलगाववित दृश्य होता है।

<details>

<summary>त्रुटि: bash: fork: मेमोरी आवंटित नहीं कर सकता</summary>

यदि आप `-f` के बिना पिछली पंक्ति को चलाते हैं, तो आप उस त्रुटि को प्राप्त करेंगे।\
यह त्रुटि नए नेमस्पेस में प्रोसेस 1 की प्रक्रिया के बाहर निकलने के कारण होती है।

जब बैश चलने लगता है, तो बैश कई नई सब-प्रोसेस को फोर्क करके कुछ काम करने के लिए बनाता है। यदि आप -f के बिना अनशेयर चलाते हैं, तो बैश का पिडी वर्तमान "अनशेयर" प्रक्रिया के समान होगा। वर्तमान "अनशेयर" प्रक्रिया अनशेयर सिस्टम कॉल करती है, एक नया पिडी नेमस्पेस बनाती है, लेकिन वर्तमान "अनशेयर" प्रक्रिया नए पिडी नेमस्पेस में नहीं है। यह लिनक्स कर्नल का वांछित व्यवहार है: प्रक्रिया एक नया नेमस्पेस बनाती है, प्रक्रिया खुद नए नेमस्पेस में नहीं रखी जाएगी, केवल प्रक्रिया के सब-प्रोसेस ही नए नेमस्पेस में रखे जाएंगे। इसलिए जब आप चलाते हैं:
```
unshare -p /bin/bash
```
अनशेयर प्रक्रिया /बिन/बैश को एक्जेक्यूट करेगी, और /बिन/बैश कई सब-प्रक्रियाएं फोर्क करेगा, बैश की पहली सब-प्रक्रिया नए नेमस्पेस का पीआईडी 1 बन जाएगी, और सब-प्रक्रिया अपना काम पूरा करने के बाद बंद हो जाएगी। इसलिए नए नेमस्पेस का पीआईडी 1 बंद हो जाता है।

पीआईडी 1 प्रक्रिया का एक विशेष कार्य होता है: यह सभी अनाथ प्रक्रियाओं के माता-पिता प्रक्रिया बनना चाहिए। अगर रूट नेमस्पेस में पीआईडी 1 प्रक्रिया बंद हो जाती है, तो कर्नल पैनिक हो जाता है। अगर एक सब नेमस्पेस में पीआईडी 1 प्रक्रिया बंद हो जाती है, तो लिनक्स कर्नल disable\_pid\_allocation फ़ंक्शन को कॉल करेगा, जो उस नेमस्पेस में PIDNS\_HASH\_ADDING फ़्लैग को साफ़ करेगा। जब लिनक्स कर्नल एक नई प्रक्रिया बनाता है, तो कर्नल एक नेमस्पेस में एक पीआईडी आवंटित करने के लिए alloc\_pid फ़ंक्शन को कॉल करेगा, और अगर PIDNS\_HASH\_ADDING फ़्लैग सेट नहीं है, तो alloc\_pid फ़ंक्शन -ENOMEM त्रुटि लौटाएगा। इसीलिए आपको "Cannot allocate memory" त्रुटि मिली है।

आप '-f' विकल्प का उपयोग करके इस समस्या को हल कर सकते हैं:
```
unshare -fp /bin/bash
```
यदि आप '-f' विकल्प के साथ unshare चलाते हैं, तो unshare नए pid नेमस्पेस बनाने के बाद एक नई प्रक्रिया फोर्क करेगा। और नई प्रक्रिया में /bin/bash चलाएगा। नई प्रक्रिया नए pid नेमस्पेस का pid 1 होगी। फिर बैश भी कुछ काम करने के लिए कई उप-प्रक्रियाएं फोर्क करेगा। बैश खुद नए pid नेमस्पेस का pid 1 होने के कारण, इसकी उप-प्रक्रियाएं किसी भी समस्या के बिना बंद हो सकती हैं।

[https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory](https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory)

</details>

#### डॉकर
```bash
docker run -ti --name ubuntu1 -v /usr:/ubuntu1 ubuntu bash
# Run ifconfig or ip -a
```
### &#x20;जांचें कि आपकी प्रक्रिया किस नेमस्पेस में है

To check which namespace your process is in, you can use the following command:

```bash
ls -l /proc/$$/ns/net
```

This command will display the network namespace of your process.
```bash
ls -l /proc/self/ns/net
lrwxrwxrwx 1 root root 0 Apr  4 20:30 /proc/self/ns/net -> 'net:[4026531840]'
```
### सभी नेटवर्क नेमस्पेस ढूंढें

{% code overflow="wrap" %}
```bash
sudo find /proc -maxdepth 3 -type l -name net -exec readlink {} \; 2>/dev/null | sort -u | grep "net:"
# Find the processes with an specific namespace
sudo find /proc -maxdepth 3 -type l -name net -exec ls -l  {} \; 2>/dev/null | grep <ns-number>
```
{% code %}

### नेटवर्क नेमस्पेस के अंदर दाखिल हों

{% endcode %}
```bash
nsenter -n TARGET_PID --pid /bin/bash
```
इसके अलावा, आप केवल **रूट उपयोगकर्ता होने पर ही किसी अन्य प्रक्रिया नेमस्पेस में प्रवेश कर सकते हैं**। और आप **उसके बिना किसी अन्य नेमस्पेस में प्रवेश नहीं कर सकते** हैं जिसके लिए एक डिस्क्रिप्टर हो (जैसे `/proc/self/ns/net`)।

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण तक पहुंच या HackTricks को PDF में डाउनलोड करने की आवश्यकता है**? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें और PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को**।

</details>
