# माउंट नेमस्पेस

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करना चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **फॉलो** करें मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को।**

</details>

## मूलभूत जानकारी

माउंट नेमस्पेस एक लिनक्स कर्नल सुविधा है जो प्रक्रिया समूह द्वारा देखे जाने वाले फ़ाइल सिस्टम माउंट पॉइंट की अलगाव प्रदान करती है। प्रत्येक माउंट नेमस्पेस के पास अपने फ़ाइल सिस्टम माउंट पॉइंट का एक सेट होता है, और **एक नेमस्पेस में माउंट पॉइंट में हुए परिवर्तन अन्य नेमस्पेस पर प्रभाव नहीं डालते**। इसका मतलब है कि अलग-अलग माउंट नेमस्पेस में चल रही प्रक्रियाएँ फ़ाइल सिस्टम के व्यवस्था के अलग-अलग दृष्टिकोण रख सकती हैं।

माउंट नेमस्पेस कंटेनरीकरण में विशेष रूप से उपयोगी होते हैं, जहां प्रत्येक कंटेनर के पास अपना खुद का फ़ाइल सिस्टम और कॉन्फ़िगरेशन होना चाहिए, अन्य कंटेनर और होस्ट सिस्टम से अलग।

### कैसे काम करता है:

1. जब एक नया माउंट नेमस्पेस बनाया जाता है, तो इसे अपने मूल नेमस्पेस के माउंट पॉइंट की **प्रतिलिपि के साथ प्रारंभिक।** इसका मतलब है कि नया नेमस्पेस निर्माण के समय, नया नेमस्पेस फ़ाइल सिस्टम के साथ अपने मूल का समान दृश्य साझा करता है। हालांकि, नेमस्पेस के भीतर माउंट पॉइंट में किसी भी भविष्यवाणी के परिवर्तन पर मूल या अन्य नेमस्पेस पर प्रभाव नहीं पड़ता है।
2. जब प्रक्रिया अपने नेमस्पेस के भीतर माउंट पॉइंट को संशोधित करती है, जैसे कि फ़ाइल सिस्टम को माउंट या अनमाउंट करना, तो **परिवर्तन उस नेमस्पेस के लिए स्थानीय होता है** और अन्य नेमस्पेस पर प्रभाव नहीं पड़ता है। इससे प्रत्येक नेमस्पेस के पास अपना स्वतंत्र फ़ाइल सिस्टम व्यवस्था हो सकती है।
3. प्रक्रियाएँ `setns()` सिस्टम कॉल का उपयोग करके नेमस्पेस के बीच चल सकती हैं, या `unshare()` या `clone()` सिस्टम कॉल का उपयोग करके नए नेमस्पेस बना सकती हैं `CLONE_NEWNS` ध्वज के साथ। जब प्रक्रिया एक नए नेमस्पेस में जाती है या एक नया नेमस्पेस बनाती है, तो वह उस नेमस्पेस के साथ जुड़े माउंट पॉइंट का उपयोग करना शुरू कर देगी।
4. **फ़ाइल डेस्क्रिप्टर और इनोड नेमस्पेस के बीच साझा किए जाते हैं**, इसका मतलब है कि अगर एक नेमस्पेस में एक प्रक्रिया के पास एक खुला फ़ाइल डेस्क्रिप्टर है जो एक फ़ाइल को इंगित कर रहा है, तो वह प्रक्रिया दूसरे नेमस्पेस में एक प्रक्रिया को उस फ़ाइल डेस्क्रिप्टर को पास कर सकती है, और **दोनों प्रक्रियाएँ उसी फ़ाइल तक पहुंचेंगी**। हालांकि, माउंट पॉइंट में अंतर के कारण दोनों नेमस्पेस में फ़ाइल का पथ समान नहीं हो सकता है।

## प्रयोगशाला:

### विभिन्न नेमस्पेस बनाएँ

#### CLI
```bash
sudo unshare -m [--mount-proc] /bin/bash
```
यदि आप `--mount-proc` पैरामीटर का उपयोग करते हैं, तो `/proc` फ़ाइल सिस्टम के एक नए इंस्टेंस को माउंट करके आप सुनिश्चित करते हैं कि नया माउंट नेमस्पेस उस नेमस्पेस के लिए प्रक्रिया सूचना का सटीक और अलगाववित दृश्य होता है।

<details>

<summary>त्रुटि: bash: fork: मेमोरी आवंटित नहीं कर सकता</summary>

यदि आप `-f` के बिना पिछली पंक्ति को चलाते हैं, तो आप उस त्रुटि को प्राप्त करेंगे।\
यह त्रुटि नए नेमस्पेस में प्रोसेस 1 का बंद हो जाने से होती है।

जब बैश चलने लगता है, तो बैश कई नए उप-प्रक्रियाएं बनाने के लिए फोर्क करेगा। यदि आप -f के बिना अनशेयर चलाते हैं, तो बैश का पिडी वर्तमान "अनशेयर" प्रक्रिया के समान होगा। वर्तमान "अनशेयर" प्रक्रिया अनशेयर सिस्टम कॉल करती है, एक नया पिडी नेमस्पेस बनाती है, लेकिन वर्तमान "अनशेयर" प्रक्रिया नए पिडी नेमस्पेस में नहीं है। यह लिनक्स कर्नल का वांछित व्यवहार है: प्रक्रिया एक नया नेमस्पेस बनाती है, प्रक्रिया खुद नए नेमस्पेस में नहीं डाली जाएगी, केवल प्रक्रिया की उप-प्रक्रियाएं नए नेमस्पेस में डाली जाएगी। इसलिए जब आप चलाते हैं:
```
unshare -p /bin/bash
```
अनशेयर प्रक्रिया /बिन/बैश को एक्जेक्यूट करेगी, और /बिन/बैश कई सब-प्रक्रियाएं फोर्क करेगा, बैश की पहली सब-प्रक्रिया नए नेमस्पेस का पीआईडी 1 बन जाएगी, और सब-प्रक्रिया अपना काम पूरा करने के बाद बंद हो जाएगी। इसलिए नए नेमस्पेस का पीआईडी 1 बंद हो जाता है।

पीआईडी 1 प्रक्रिया का एक विशेष कार्य होता है: यह सभी अनाथ प्रक्रियाओं के माता-पिता प्रक्रिया बनना चाहिए। अगर रूट नेमस्पेस में पीआईडी 1 प्रक्रिया बंद हो जाती है, तो कर्नल पैनिक हो जाएगा। अगर एक सब-नेमस्पेस में पीआईडी 1 प्रक्रिया बंद हो जाती है, तो लिनक्स कर्नल disable\_pid\_allocation फंक्शन को कॉल करेगा, जो उस नेमस्पेस में PIDNS\_HASH\_ADDING फ्लैग को साफ करेगा। जब लिनक्स कर्नल एक नई प्रक्रिया बनाता है, तो कर्नल नेमस्पेस में एक पीआईडी आवंटित करने के लिए alloc\_pid फंक्शन को कॉल करेगा, और अगर PIDNS\_HASH\_ADDING फ्लैग सेट नहीं है, तो alloc\_pid फंक्शन -ENOMEM त्रुटि लौटाएगा। इसीलिए आपको "Cannot allocate memory" त्रुटि मिली है।

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
cat /proc/$$/mountinfo | grep "ns"
```

आपकी प्रक्रिया किस नेमस्पेस में है यह जानने के लिए, निम्नलिखित कमांड का उपयोग करें:

```bash
cat /proc/$$/mountinfo | grep "ns"
```
```bash
ls -l /proc/self/ns/mnt
lrwxrwxrwx 1 root root 0 Apr  4 20:30 /proc/self/ns/mnt -> 'mnt:[4026531841]'
```
### सभी माउंट नेमस्पेस ढूंढ़ें

{% code overflow="wrap" %}
```bash
sudo find /proc -maxdepth 3 -type l -name mnt -exec readlink {} \; 2>/dev/null | sort -u
# Find the processes with an specific namespace
sudo find /proc -maxdepth 3 -type l -name mnt -exec ls -l  {} \; 2>/dev/null | grep <ns-number>
```
{% code %}

### माउंट नेमस्पेस के अंदर प्रवेश करें

{% endcode %}
```bash
nsenter -m TARGET_PID --pid /bin/bash
```
इसके अलावा, आप केवल रूट यूजर होने की स्थिति में ही **दूसरे प्रक्रिया नेमस्पेस में प्रवेश कर सकते हैं**। और आप **बिना एक डिस्क्रिप्टर** के (जैसे `/proc/self/ns/mnt`) इसकी ओर नेमस्पेस में **प्रवेश नहीं** कर सकते हैं।

क्योंकि नए माउंट्स केवल नेमस्पेस के भीतर ही पहुंचने योग्य होते हैं, इसलिए एक नेमस्पेस में संवेदनशील जानकारी हो सकती है जिसका केवल उसी से पहुंच किया जा सकता है।

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
<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT संग्रह**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को।**

</details>
