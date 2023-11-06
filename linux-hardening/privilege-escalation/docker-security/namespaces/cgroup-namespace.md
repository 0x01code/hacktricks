# सीग्रुप नेमस्पेस

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **फॉलो** करें मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को**

</details>

## मूलभूत जानकारी

सीग्रुप नेमस्पेस एक लिनक्स कर्नल सुविधा है जो **नेमस्पेस के भीतर चल रहे प्रक्रियाओं के लिए सीग्रुप हाइरार्की की अलगाव प्रदान करती है**। सीग्रुप, **कंट्रोल समूहों** के लिए संक्षिप्त रूप से हैं, एक कर्नल सुविधा है जो प्रक्रियाओं को व्यवस्थित करने और **सिस्टम संसाधनों पर सीमाएं लागू करने** जैसे सीपीयू, मेमोरी और आई/ओ के लिए व्यवस्था करने की अनुमति देती है।

हालांकि सीग्रुप नेमस्पेस पिछले चर्चित नेमस्पेस (पीआईडी, माउंट, नेटवर्क, आदि) की तरह एक अलग नेमस्पेस प्रकार नहीं हैं, वे नेमस्पेस अलगाव की अवधारणा से संबंधित हैं। **सीग्रुप नेमस्पेस सीग्रुप हाइरार्की के दृश्य को वर्चुअलाइज़ करते हैं**, इसका मतलब है कि सीग्रुप नेमस्पेस में चल रही प्रक्रियाएं हाइरार्की के बारे में अन्य नेमस्पेस या होस्ट के अन्य नेमस्पेस की तुलना में एक अलग दृश्य रखती हैं।

### काम कैसे करता है:

1. जब एक नया सीग्रुप नेमस्पेस बनाया जाता है, **यह उस सीग्रुप हाइरार्की के आधार पर एक दृश्य के साथ शुरू होता है जो नई प्रक्रिया के नेमस्पेस को बनाने वाली प्रक्रिया के सीग्रुप के नीचे सीग्रुप उपवृक्ति तक सीमित होती है**। इसका मतलब है कि सीग्रुप नेमस्पेस में चल रही प्रक्रियाएं केवल संपूर्ण सीग्रुप हाइरार्की का एक उपसेट देखेंगी, जो नई प्रक्रिया के सीग्रुप के नीचे सीग्रुप उपवृक्ति तक सीमित होती है।
2. सीग्रुप नेमस्पेस के भीतर की प्रक्रियाएं **अपने खुद के सीग्रुप को हाइरार्की का मूल बनाती हैं**। इसका मतलब है कि नेमस्पेस के भीतर की प्रक्रियाओं के दृष्टिकोण से, उनका खुद का सीग्रुप मूल के रूप में प्रदर्शित होता है, और वे अपने उपवृक्ति के बाहर के सीग्रुप को नहीं देख सकते या पहुंच सकते हैं।
3. सीग्रुप नेमस्पेस सीधे संसाधनों की अलगाव प्रदान नहीं करते हैं; **वे केवल सीग्रुप हाइरार्की दृश्य का अलगाव प्रदान करते हैं**। **संसाधन नियंत्रण और अलगाव अभी भी सीग्रुप** उपप्रणालियों (जैसे सीपीयू, मेमोरी, आदि) **द्वारा प्रवर्तित किए जाते हैं**।

सीग्रुप के बारे में अधिक जानकारी के लिए देखें:

{% content-ref url="../cgroups.md" %}
[cgroups.md](../cgroups.md)
{% endcontent-ref %}

## प्रयोगशाला:

### विभिन्न नेमस्पेस बनाएँ

#### CLI
```bash
sudo unshare -C [--mount-proc] /bin/bash
```
यदि आप `--mount-proc` पैरामीटर का उपयोग करते हैं, तो `/proc` फ़ाइल सिस्टम के एक नए इंस्टेंस को माउंट करके आप सुनिश्चित करते हैं कि नया माउंट नेमस्पेस उस नेमस्पेस के लिए प्रक्रिया सूचना का सटीक और अलगाववित दृश्य होता है।

<details>

<summary>त्रुटि: bash: fork: मेमोरी आवंटित नहीं कर सकता</summary>

यदि आप `-f` के बिना पिछली पंक्ति को चलाते हैं, तो आप उस त्रुटि को प्राप्त करेंगे।\
यह त्रुटि नए नेमस्पेस में प्रोसेस 1 की प्रक्रिया के बाहर निकलने के कारण होती है।

जब बैश चलने लगता है, तो बैश कई नई सब-प्रोसेस को फोर्क करके कुछ काम करने के लिए बनाता है। यदि आप -f के बिना अनशेयर चलाते हैं, तो बैश का पिडी वर्तमान "अनशेयर" प्रक्रिया के समान होगा। वर्तमान "अनशेयर" प्रक्रिया अनशेयर सिस्टम कॉल करती है, एक नया पिडी नेमस्पेस बनाती है, लेकिन वर्तमान "अनशेयर" प्रक्रिया नए पिडी नेमस्पेस में नहीं है। यह लिनक्स कर्नल का वांछित व्यवहार है: प्रक्रिया एक नया नेमस्पेस बनाती है, प्रक्रिया खुद नए नेमस्पेस में नहीं रखी जाएगी, केवल प्रक्रिया के सब-प्रोसेस नए नेमस्पेस में रखे जाएंगे। इसलिए जब आप चलाते हैं:
```
unshare -p /bin/bash
```
अनशेयर प्रक्रिया /बिन/बैश को एक्जेक्यूट करेगी, और /बिन/बैश कई सब-प्रक्रियाएं फोर्क करेगा, बैश की पहली सब-प्रक्रिया नए नेमस्पेस का पीआईडी 1 बन जाएगी, और सब-प्रक्रिया अपना काम पूरा करने के बाद बंद हो जाएगी। इसलिए नए नेमस्पेस का पीआईडी 1 बंद हो जाता है।

पीआईडी 1 प्रक्रिया का एक विशेष कार्य होता है: यह सभी अनाथ प्रक्रियाओं के माता-पिता प्रक्रिया बनना चाहिए। अगर रूट नेमस्पेस में पीआईडी 1 प्रक्रिया बंद हो जाती है, तो कर्नल पैनिक हो जाएगा। अगर उप-नेमस्पेस में पीआईडी 1 प्रक्रिया बंद हो जाती है, तो लिनक्स कर्नल disable\_pid\_allocation फंक्शन को कॉल करेगा, जो उस नेमस्पेस में PIDNS\_HASH\_ADDING फ्लैग को साफ करेगा। जब लिनक्स कर्नल एक नई प्रक्रिया बनाता है, तो कर्नल एक नेमस्पेस में एक पीआईडी आवंटित करने के लिए alloc\_pid फंक्शन को कॉल करेगा, और अगर PIDNS\_HASH\_ADDING फ्लैग सेट नहीं है, तो alloc\_pid फंक्शन -ENOMEM त्रुटि लौटाएगा। इसीलिए आपको "Cannot allocate memory" त्रुटि मिली है।

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
### &#x20;जांचें कि आपकी प्रक्रिया किस नेमस्पेस में है

To check which namespace your process is in, you can use the following command:

```bash
cat /proc/$$/cgroup
```

आपकी प्रक्रिया किस नेमस्पेस में है यह जानने के लिए, आप निम्नलिखित कमांड का उपयोग कर सकते हैं:

```bash
cat /proc/$$/cgroup
```
```bash
ls -l /proc/self/ns/cgroup
lrwxrwxrwx 1 root root 0 Apr  4 21:19 /proc/self/ns/cgroup -> 'cgroup:[4026531835]'
```
### सभी सीग्रुप नेमस्पेस ढूंढ़ें

{% code overflow="wrap" %}
```bash
sudo find /proc -maxdepth 3 -type l -name cgroup -exec readlink {} \; 2>/dev/null | sort -u
# Find the processes with an specific namespace
sudo find /proc -maxdepth 3 -type l -name cgroup -exec ls -l  {} \; 2>/dev/null | grep <ns-number>
```
{% endcode %}

### एक सीग्रुप नेमस्पेस में दाखिल हों

{% code-tabs %}
{% code-tabs-item title="उदाहरण" %}
```bash
nsenter --target <PID> --mount --uts --ipc --net --pid
```
{% endcode-tabs-item %}
{% endcode-tabs %}

एक सीग्रुप नेमस्पेस में प्रवेश करने के लिए, आप `nsenter` उपकरण का उपयोग कर सकते हैं। यह उपकरण आपको दिए गए प्रक्रिया के भीतर एक नेमस्पेस में प्रवेश करने की अनुमति देता है। आपको निश्चित करने के लिए कि आप एक सीग्रुप नेमस्पेस में प्रवेश कर रहे हैं, आपको `--cgroup` विकल्प का उपयोग करना होगा।

यहां एक उदाहरण है जहां हम `nsenter` का उपयोग करके एक सीग्रुप नेमस्पेस में प्रवेश कर रहे हैं:

```bash
nsenter --target <PID> --mount --uts --ipc --net --pid
```

यहां `<PID>` को उस प्रक्रिया के प्रकार के साथ बदल देना होगा जिसमें आप सीग्रुप नेमस्पेस में प्रवेश करना चाहते हैं। उदाहरण के लिए, यदि आप `PID` 1234 के साथ सीग्रुप नेमस्पेस में प्रवेश करना चाहते हैं, तो आपको निम्नलिखित कमांड का उपयोग करना होगा:

```bash
nsenter --target 1234 --mount --uts --ipc --net --pid
```

इसके बाद, आप सीग्रुप नेमस्पेस में सफलतापूर्वक प्रवेश करेंगे और उस प्रक्रिया के संदर्भ में कमांड चला सकेंगे।
```bash
nsenter -C TARGET_PID --pid /bin/bash
```
इसके अलावा, आप केवल **रूट यूज़र** होने पर ही **दूसरे प्रक्रिया नेमस्पेस में प्रवेश कर सकते हैं**। और आप **उसके बिना** अन्य नेमस्पेस में **प्रवेश नहीं कर सकते** हैं जिसके लिए एक डिस्क्रिप्टर हो (जैसे `/proc/self/ns/cgroup`)।

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह।
* प्राप्त करें [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें और PR जमा करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को**।

</details>
