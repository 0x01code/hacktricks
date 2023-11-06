# उपयोगकर्ता नेमस्पेस

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित करना** चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करना चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल हों** या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)** का पालन करें**.
* **अपने हैकिंग ट्रिक्स को** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में पीआर जमा करके अपना योगदान दें।**

</details>

## मूलभूत जानकारी

उपयोगकर्ता नेमस्पेस एक Linux कर्नल सुविधा है जो **उपयोगकर्ता और समूह आईडी मैपिंग की अलगाव प्रदान करती है**, जिससे प्रत्येक उपयोगकर्ता नेमस्पेस के पास अपने **उपयोगकर्ता और समूह आईडी का अपना सेट** होता है। यह अलगाव प्रक्रियाओं को संचालित करने के लिए अनुमति देता है जो विभिन्न उपयोगकर्ता नेमस्पेस में चल रही होती हैं, भले ही वे संख्यात्मक रूप से उपयोगकर्ता और समूह आईडी साझा करती हों।

उपयोगकर्ता नेमस्पेस कंटेनरीकरण में विशेष रूप से उपयोगी होते हैं, जहां प्रत्येक कंटेनर के पास अपने निर्देशित उपयोगकर्ता और समूह आईडी का अपना सेट होना चाहिए, जिससे कंटेनर और होस्ट सिस्टम के बीच बेहतर सुरक्षा और अलगाव संभव होता है।

### काम कैसे करता है:

1. जब एक नया उपयोगकर्ता नेमस्पेस बनाया जाता है, तो इसके पास **उपयोगकर्ता और समूह आईडी मैपिंग का खाली सेट शुरू होता है**। इसका मतलब है कि नए उपयोगकर्ता नेमस्पेस में चल रही किसी भी प्रक्रिया के पास **नेमस्पेस के बाहर कोई विशेषाधिकार नहीं होते हैं**।
2. नए नेमस्पेस में उपयोगकर्ता और समूह आईडी के बीच आईडी मैपिंग स्थापित की जा सकती हैं, जो मूल (या होस्ट) नेमस्पेस में उपयोगकर्ता और समूह आईडी के बीच होती हैं। इससे नए नेमस्पेस में चल रही प्रक्रियाओं को मूल नेमस्पेस में उपयोगकर्ता और समूह आईडी के अनुसार विशेषाधिकार और स्वामित्व हो सकता है। हालांकि, आईडी मैपिंग को विशेष सीमाओं और आईडी के उपसमूहों पर प्रतिबंधित किया जा सकता है, जिससे नए नेमस्पेस में चल रही प्रक्रियाओं को प्राप्त विशेषाधिकारों पर नियंत्रण रखने की सुविधा होती है।
3. एक उपयोगकर्ता नेमस्पेस के भीतर, **प्रक्रियाओं को पूर्ण रूप से रूट विशेषाधिकार (UID 0) हो सकते हैं** नेमस्पेस के अंदर के आपरेशन के लिए**, जबकि उनके पास नेमस्पेस के बाहर सीमित विशेषाधिकार होते हैं। इससे कंटेनर अपने नेमस्पेस में रूट जैसी क्षमताओं के साथ चल सकते हैं, लेकिन होस्ट सिस्टम पर पूर्ण रूप से रूट विशेषाधिकार नहीं होते हैं।
4. प्रक्रियाएं `setns()` सिस्टम कॉल का उपयोग करके नेमस्पेस के बीच ले जा सकती हैं या `unshare()` या `clone()` सिस्टम कॉल का उपयोग करके नए नेमस्पेस बना सकती हैं जिनमें `CLONE_NEWUSER` फ़्लैग होता है। जब कोई प्रक्रिया नए नेमस्पेस में जाती है या एक नया नेमस्पेस बनाती है, तो वह उस नेमस्पेस के साथ संबंधित उपयोगकर्ता और समूह आईडी मैपिंग का उपयोग करना शुरू कर देती है।

## प्रयोगशाला:

### विभिन्न नेमस्पेस बनाएँ

#### CLI
```bash
sudo unshare -U [--mount-proc] /bin/bash
```
यदि आप `--mount-proc` पैरामीटर का उपयोग करते हैं, तो `/proc` फ़ाइल सिस्टम के एक नए इंस्टेंस को माउंट करके आप सुनिश्चित करते हैं कि नया माउंट नेमस्पेस उस नेमस्पेस के लिए प्रक्रिया सूचना का सटीक और अलगाववित दृश्य होता है।

<details>

<summary>त्रुटि: bash: fork: मेमोरी आवंटित नहीं कर सकता</summary>

यदि आप `-f` के बिना पिछली पंक्ति को चलाते हैं, तो आपको वह त्रुटि मिलेगी।\
यह त्रुटि नए नेमस्पेस में प्रोसेस 1 का बंद हो जाने से होती है।

जब बैश चलने लगता है, तो बैश कई नए सब-प्रोसेस बनाने के लिए फोर्क करेगा। यदि आप -f के बिना अनशेयर चलाते हैं, तो बैश का पिडी वर्तमान "अनशेयर" प्रक्रिया के समान होगा। वर्तमान "अनशेयर" प्रक्रिया अनशेयर सिस्टम कॉल करती है, एक नया पिडी नेमस्पेस बनाती है, लेकिन वर्तमान "अनशेयर" प्रक्रिया नए पिडी नेमस्पेस में नहीं है। यह लिनक्स कर्नल का वांछित व्यवहार है: प्रक्रिया A एक नया नेमस्पेस बनाती है, प्रक्रिया A खुद नए नेमस्पेस में नहीं डाली जाएगी, केवल प्रक्रिया A के सब-प्रोसेस नए नेमस्पेस में डाले जाएंगे। इसलिए जब आप चलाते हैं:
```
unshare -p /bin/bash
```
अनशेयर प्रक्रिया /बिन/बैश को एक्जेक्यूट करेगी, और /बिन/बैश कई सब-प्रक्रियाएं फोर्क करेगा, बैश की पहली सब-प्रक्रिया नए नेमस्पेस का पीआईडी 1 बन जाएगी, और सब-प्रक्रिया अपना काम पूरा करने के बाद बंद हो जाएगी। इसलिए नए नेमस्पेस का पीआईडी 1 बंद हो जाता है।

पीआईडी 1 प्रक्रिया का एक विशेष कार्य होता है: यह सभी अनाथ प्रक्रियाओं के माता-पिता प्रक्रिया बनना चाहिए। अगर रूट नेमस्पेस में पीआईडी 1 प्रक्रिया बंद हो जाती है, तो कर्नल पैनिक हो जाएगा। अगर उप-नेमस्पेस में पीआईडी 1 प्रक्रिया बंद हो जाती है, तो लिनक्स कर्नल disable\_pid\_allocation फंक्शन को कॉल करेगा, जो उस नेमस्पेस में PIDNS\_HASH\_ADDING फ्लैग को साफ करेगा। जब लिनक्स कर्नल एक नई प्रक्रिया बनाता है, तो कर्नल नेमस्पेस में एक पीआईडी आवंटित करने के लिए alloc\_pid फंक्शन को कॉल करेगा, और अगर PIDNS\_HASH\_ADDING फ्लैग सेट नहीं है, तो alloc\_pid फंक्शन -ENOMEM त्रुटि लौटाएगा। इसीलिए आपको "Cannot allocate memory" त्रुटि मिली है।

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
```
उपयोगकर्ता नेमस्पेस का उपयोग करने के लिए, डॉकर डीमन को **`--userns-remap=default`** के साथ शुरू किया जाना चाहिए। (यूबंटू 14.04 में, इसे `/etc/default/docker` को संशोधित करके और फिर `sudo service docker restart` को निष्पादित करके किया जा सकता है)

### &#x20;जांचें कि आपका प्रक्रिया किस नेमस्पेस में है
```bash
ls -l /proc/self/ns/user
lrwxrwxrwx 1 root root 0 Apr  4 20:57 /proc/self/ns/user -> 'user:[4026531837]'
```
यह संभव है कि आप डॉकर कंटेनर से उपयोगकर्ता मानचित्र की जांच कर सकते हैं:
```bash
cat /proc/self/uid_map
0          0 4294967295  --> Root is root in host
0     231072      65536  --> Root is 231072 userid in host
```
या होस्ट से:
```bash
cat /proc/<pid>/uid_map
```
### सभी उपयोगकर्ता नेमस्पेस ढूंढें

{% code overflow="wrap" %}
```bash
sudo find /proc -maxdepth 3 -type l -name user -exec readlink {} \; 2>/dev/null | sort -u
# Find the processes with an specific namespace
sudo find /proc -maxdepth 3 -type l -name user -exec ls -l  {} \; 2>/dev/null | grep <ns-number>
```
{% endcode %}

### एक उपयोगकर्ता नेमस्पेस में प्रवेश करें

{% code-tabs %}
{% code-tabs-item title="उपयोगकर्ता नेमस्पेस में प्रवेश करें" %}
```bash
unshare -r /bin/bash
```
{% endcode-tabs-item %}
{% endcode-tabs %}

उपयोगकर्ता नेमस्पेस में प्रवेश करने के लिए `unshare -r /bin/bash` कमांड का उपयोग करें।
```bash
nsenter -U TARGET_PID --pid /bin/bash
```
इसके अलावा, आप केवल रूट होने पर ही **दूसरे प्रक्रिया नेमस्पेस में प्रवेश कर सकते हैं**। और आप **बिना एक डिस्क्रिप्टर** के अन्य नेमस्पेस में **प्रवेश नहीं** कर सकते हैं (जैसे `/proc/self/ns/user` की तरह)।

### नए उपयोगकर्ता नेमस्पेस (मैपिंग के साथ) बनाएं

{% code overflow="wrap" %}
```bash
unshare -U [--map-user=<uid>|<name>] [--map-group=<gid>|<name>] [--map-root-user] [--map-current-user]
```
{% endcode %}
```bash
# Container
sudo unshare -U /bin/bash
nobody@ip-172-31-28-169:/home/ubuntu$ #Check how the user is nobody

# From the host
ps -ef | grep bash # The user inside the host is still root, not nobody
root       27756   27755  0 21:11 pts/10   00:00:00 /bin/bash
```
### क्षमताओं की पुनर्प्राप्ति

यदि उपयोगकर्ता नेमस्पेस के मामले में, तब जब एक नया उपयोगकर्ता नेमस्पेस बनाया जाता है, तो नेमस्पेस में प्रवेश करने वाले प्रक्रिया को उस नेमस्पेस के भीतर पूरी क्षमता सेट प्रदान की जाती है। ये क्षमताएं प्रक्रिया को विशेषाधिकारिक आपरेशन करने की अनुमति देती हैं, जैसे कि फ़ाइल सिस्टम माउंट करना, डिवाइस बनाना, या फ़ाइलों के स्वामित्व में परिवर्तन करना, लेकिन केवल अपने उपयोगकर्ता नेमस्पेस के संदर्भ में।

उदाहरण के लिए, जब आपके पास उपयोगकर्ता नेमस्पेस के भीतर `CAP_SYS_ADMIN` क्षमता होती है, तो आप इस क्षमता की आवश्यकता वाले आपरेशन कर सकते हैं, जैसे कि फ़ाइल सिस्टम माउंट करना, लेकिन केवल अपने उपयोगकर्ता नेमस्पेस के संदर्भ में। इस क्षमता के साथ आप जो भी आपरेशन करते हैं, वह होस्ट सिस्टम या अन्य नेमस्पेस पर प्रभाव नहीं डालेगा।

{% hint style="warning" %}
इसलिए, यदि आप एक नए प्रक्रिया को एक नए उपयोगकर्ता नेमस्पेस में प्राप्त करने के बावजूद आपको सभी क्षमताएं वापस मिल जाएंगी (CapEff: 000001ffffffffff), तो आप वास्तव में केवल नेमस्पेस से संबंधित क्षमताओं (जैसे माउंट) का ही उपयोग कर सकते हैं, लेकिन हर एक क्षमता का नहीं। इसलिए, यह अपने आप में एक डॉकर कंटेनर से बाहर निकलने के लिए पर्याप्त नहीं है।
{% endhint %}
```bash
# There are the syscalls that are filtered after changing User namespace with:
unshare -UmCpf  bash

Probando: 0x067 . . . Error
Probando: 0x070 . . . Error
Probando: 0x074 . . . Error
Probando: 0x09b . . . Error
Probando: 0x0a3 . . . Error
Probando: 0x0a4 . . . Error
Probando: 0x0a7 . . . Error
Probando: 0x0a8 . . . Error
Probando: 0x0aa . . . Error
Probando: 0x0ab . . . Error
Probando: 0x0af . . . Error
Probando: 0x0b0 . . . Error
Probando: 0x0f6 . . . Error
Probando: 0x12c . . . Error
Probando: 0x130 . . . Error
Probando: 0x139 . . . Error
Probando: 0x140 . . . Error
Probando: 0x141 . . . Error
Probando: 0x143 . . . Error
```
<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **हैकट्रिक्स में विज्ञापित करना** चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करना चाहते हैं? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT संग्रह**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स को** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में PR जमा करके साझा करें।**

</details>
