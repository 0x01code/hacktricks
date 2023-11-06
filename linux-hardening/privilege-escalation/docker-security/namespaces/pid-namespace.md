# PID नेमस्पेस

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को**

</details>

## मूलभूत जानकारी

PID (प्रक्रिया आईडेंटिफायर) नेमस्पेस एक विशेषता है जो लिनक्स कर्नल में मौजूद है और प्रक्रिया विभाजन प्रदान करती है जिससे कि एक समूह की प्रक्रियाओं को अन्य नेमस्पेस में मौजूद प्रक्रियाओं से अलग और अद्वितीय प्रक्रिया आईडी (PID) सेट होती है। यह खासकर सुरक्षा और संसाधन प्रबंधन के लिए आवश्यक प्रक्रिया विभाजन में उपयोगी होता है।

जब एक नया PID नेमस्पेस बनाया जाता है, तो उस नेमस्पेस में पहली प्रक्रिया को PID 1 के रूप में सौंपा जाता है। यह प्रक्रिया नए नेमस्पेस का "इनिट" प्रक्रिया बन जाती है और नेमस्पेस के अंदर अन्य प्रक्रियाओं का प्रबंधन करने के लिए जिम्मेदार होती है। नेमस्पेस के अंदर बनाई गई प्रत्येक आगामी प्रक्रिया को उस नेमस्पेस के भीतर एक अद्वितीय PID मिलेगी, और ये PID अन्य नेमस्पेस में मौजूद PIDs से अलग होंगे।

एक PID नेमस्पेस के अंदर की प्रक्रिया के दृष्टिकोण से, वह केवल उसी नेमस्पेस में मौजूद अन्य प्रक्रियाएं देख सकती है। वह अन्य नेमस्पेस में मौजूद प्रक्रियाओं के बारे में जागरूक नहीं होती है, और वह उनके साथ पारंपरिक प्रक्रिया प्रबंधन उपकरणों (जैसे कि `kill`, `wait` आदि) का उपयोग करके उनसे संवाद नहीं कर सकती। यह एक स्तर की अलगाव प्रदान करता है जो प्रक्रियाओं को एक दूसरे के साथ बाधित होने से रोकता है।

### काम कैसे करता है:

1. जब एक नई प्रक्रिया बनाई जाती है (उदाहरण के लिए, `clone()` सिस्टम कॉल का उपयोग करके), तो प्रक्रिया को एक नए या मौजूदा PID नेमस्पेस को सौंपा जा सकता है। **यदि एक नया नेमस्पेस बनाया जाता है, तो प्रक्रिया उस नेमस्पेस की "इनिट" प्रक्रिया बन जाती है**।
2. **कर्नल** नए नेमस्पेस में मौजूद PIDs और मूल नेमस्पेस (यानी नये नेमस्पेस के लिए जिससे नया नेमस्पेस बनाया गया था) में मौजूद PIDs के बीच एक **मैपिंग बनाए रखता है**। यह मैपिंग **कर्नल को जब आवश्यक होता है, जैसे कि अलग-अलग नेमस्पेस में प्रक्रियाओं के बीच संकेत भेजने के समय**, PIDs का अनुवाद करने की अनुमति देता है।
3. **PID नेमस्पेस के अंदर की प्रक्रियाएं केवल उसी नेमस्पेस में मौजूद और उसके साथ संवाद कर सकती हैं**। वे अन्य नेमस्पेस में मौजूद प्रक्रियाओं के बारे में जागरूक नहीं होती हैं, और उनके PIDs उनके नेमस्पेस के भीतर अद्वितीय होते हैं।
4. **PID नेमस्पेस को नष्ट कर दिया जाता है** (उदाहरण के लिए, जब नेमस्पेस की "इनिट" प्रक्रिया समाप्त हो जाती है), **उस नेमस्पेस के अंदर की सभी प्रक्रियाएं समाप्त हो जाती हैं**। इससे सुनिश्चित होता है कि नेमस्पेस के साथ संबंधित सभी संसाधनों को सही ढंग से साफ किया जाता है।

## प्रयोगशाला:

### विभिन्न नेमस्पेस बनाएँ

#### CLI
```bash
sudo unshare -pf --mount-proc /bin/bash
```
<details>

<summary>त्रुटि: bash: fork: मेमोरी आवंटित नहीं कर सका</summary>

यदि आप पिछली पंक्ति को `-f` के बिना चलाते हैं, तो आप यह त्रुटि प्राप्त करेंगे।\
यह त्रुटि नए नेमस्पेस में पीआईडी 1 प्रक्रिया के बाहर निकलने से होती है।

जब बैश चलने लगता है, तो बैश कई नए उप-प्रक्रियाएं बनाने के लिए फोर्क करेगा। यदि आप -f के बिना अनशेयर चलाते हैं, तो बैश का पीआईडी मौजूदा "अनशेयर" प्रक्रिया के समान होगा। मौजूदा "अनशेयर" प्रक्रिया अनशेयर सिस्टम कॉल को बुलाती है, एक नया पीआईडी नेमस्पेस बनाती है, लेकिन मौजूदा "अनशेयर" प्रक्रिया नए पीआईडी नेमस्पेस में नहीं है। यह लिनक्स कर्नल का वांछित व्यवहार है: प्रक्रिया एक नया नेमस्पेस बनाती है, प्रक्रिया खुद नए नेमस्पेस में नहीं डाली जाएगी, केवल प्रक्रिया की उप-प्रक्रियाएं नए नेमस्पेस में डाली जाएगी। इसलिए जब आप निम्नलिखित को चलाते हैं:
```
unshare -p /bin/bash
```
अनशेयर प्रक्रिया /बिन/बैश को एक्ज़ेक्यूट करेगी, और /बिन/बैश कई सब-प्रक्रियाएं फोर्क करेगा, बैश की पहली सब-प्रक्रिया नए नेमस्पेस का पीआईडी 1 बन जाएगी, और सब-प्रक्रिया अपना काम पूरा करने के बाद बंद हो जाएगी। इसलिए नए नेमस्पेस का पीआईडी 1 बंद हो जाता है।

पीआईडी 1 प्रक्रिया का एक विशेष कार्य होता है: यह सभी अनाथ प्रक्रियाओं के माता-पिता प्रक्रिया बनना चाहिए। अगर रूट नेमस्पेस में पीआईडी 1 प्रक्रिया बंद हो जाती है, तो कर्नल पैनिक हो जाएगा। अगर एक सब-नेमस्पेस में पीआईडी 1 प्रक्रिया बंद हो जाती है, तो लिनक्स कर्नल disable\_pid\_allocation फ़ंक्शन को कॉल करेगा, जो उस नेमस्पेस में PIDNS\_HASH\_ADDING फ़्लैग को साफ़ करेगा। जब लिनक्स कर्नल एक नई प्रक्रिया बनाता है, तो कर्नल एक नेमस्पेस में एक पीआईडी आवंटित करने के लिए alloc\_pid फ़ंक्शन को कॉल करेगा, और अगर PIDNS\_HASH\_ADDING फ़्लैग सेट नहीं है, तो alloc\_pid फ़ंक्शन एक -ENOMEM त्रुटि लौटाएगा। इसीलिए आपको "Cannot allocate memory" त्रुटि मिली है।

आप '-f' विकल्प का उपयोग करके इस समस्या को हल कर सकते हैं:
```
unshare -fp /bin/bash
```
यदि आप '-f' विकल्प के साथ unshare चलाते हैं, तो unshare नया प्रक्रिया बनाएगा जब यह नया pid नेमस्पेस बनाएगा। और नई प्रक्रिया में /bin/bash चलाएगा। नई प्रक्रिया नये pid नेमस्पेस का pid 1 होगी। फिर बैश भी कुछ कार्यों को करने के लिए कई उप-प्रक्रियाएं बनाएगी। बैश खुद नए pid नेमस्पेस का pid 1 होने के कारण, इसकी उप-प्रक्रियाएं किसी भी समस्या के बिना बंद हो सकती हैं।

[https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory](https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory) से कॉपी किया गया है

</details>

नये `/proc` फ़ाइल सिस्टम के एक नये इंस्टेंस को माउंट करके, आप सुनिश्चित करते हैं कि नया माउंट नेमस्पेस उस नेमस्पेस के लिए प्रक्रिया सूचना का सटीक और अलग दृश्य होता है।

#### डॉकर
```bash
docker run -ti --name ubuntu1 -v /usr:/ubuntu1 ubuntu bash
```
### &#x20;जांचें कि आपकी प्रक्रिया किस नेमस्पेस में है

To check which namespace your process is in, use the following command:

```bash
cat /proc/$$/ns/pid
```

This will display the inode number of the PID namespace associated with your process.
```bash
ls -l /proc/self/ns/pid
lrwxrwxrwx 1 root root 0 Apr  3 18:45 /proc/self/ns/pid -> 'pid:[4026532412]'
```
### सभी PID नेमस्पेस ढूंढ़ें

{% code overflow="wrap" %}
```bash
sudo find /proc -maxdepth 3 -type l -name pid -exec readlink {} \; 2>/dev/null | sort -u
```
{% endcode %}

ध्यान दें कि प्रारंभिक (डिफ़ॉल्ट) PID नेमस्पेस से रूट उपयोगकर्ता सभी प्रक्रियाओं को देख सकता है, यहां तक कि नए PID नेमस्पेस में भी, इसलिए हम सभी PID नेमस्पेस को देख सकते हैं।

### एक PID नेमस्पेस के अंदर प्रवेश करें
```bash
nsenter -t TARGET_PID --pid /bin/bash
```
जब आप डिफ़ॉल्ट नेमस्पेस से पीआईडी नेमस्पेस में प्रवेश करते हैं, तो आपको फिर भी सभी प्रक्रियाएँ दिखाई देंगी। और उस पीआईडी एनएस से प्रक्रिया नई बैश पर पीआईडी एनएस को देख सकेगी।

इसके अलावा, आपको दूसरी प्रक्रिया पीआईडी नेमस्पेस में केवल रूट होने की स्थिति में ही प्रवेश कर सकते हैं। और आप बिना उसके एक डिस्क्रिप्टर के (जैसे `/proc/self/ns/pid`) इंटरनेस्पेस में प्रवेश नहीं कर सकते हैं।

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप चाहते हैं कि आपकी **कंपनी HackTricks में विज्ञापित हो**? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को पीडीएफ़ में डाउनलोड करने का एक्सेस** चाहिए? [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* प्राप्त करें [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें और PRs सबमिट करें** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को**.

</details>
