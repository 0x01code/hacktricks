# लिनक्स पर्यावरण चर

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एकल [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल हों** या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **हैकिंग ट्रिक्स साझा करें और PRs सबमिट करें** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को**।

</details>

## वैश्विक चर

वैश्विक चर **बालक प्रक्रियाओं** द्वारा विरासत में लिए जाएंगे।

आप अपनी वर्तमान सत्र के लिए एक वैश्विक चर बना सकते हैं:
```bash
export MYGLOBAL="hello world"
echo $MYGLOBAL #Prints: hello world
```
यह चर आपके वर्तमान सत्र और इसके चाइल्ड प्रक्रियाओं द्वारा पहुंचने योग्य होगा।

आप इसे हटा सकते हैं:
```bash
unset MYGLOBAL
```
## स्थानीय चर

**स्थानीय चर** केवल **वर्तमान शैल/स्क्रिप्ट** द्वारा **पहुंचे जा सकते हैं**।
```bash
LOCAL="my local"
echo $LOCAL
unset LOCAL
```
## वर्तमान चरों की सूची

To list the current environment variables in Linux, you can use the following command:

```bash
echo $PATH
```

This command will display the value of the `PATH` variable, which contains a list of directories where the system looks for executable files.

To list all the environment variables, you can use the `env` command:

```bash
env
```

This command will display a list of all the environment variables and their values.
```bash
set
env
printenv
cat /proc/$$/environ
cat /proc/`python -c "import os; print(os.getppid())"`/environ
```
## स्थायी पर्यावरण चर

#### **प्रत्येक उपयोगकर्ता के व्यवहार को प्रभावित करने वाले फ़ाइलें:**

* _**/etc/bash.bashrc**_: यह फ़ाइल हर बार पढ़ी जाती है जब एक इंटरैक्टिव शेल शुरू होता है (सामान्य टर्मिनल) और इसमें निर्दिष्ट सभी कमांडों को निष्पादित किया जाता है।
* _**/etc/profile और /etc/profile.d/\***_**:** यह फ़ाइल हर बार एक उपयोगकर्ता लॉगिन करने पर पढ़ी जाती है। इसलिए इसमें निष्पादित सभी कमांड केवल उपयोगकर्ता लॉगिन करने के समय एक बार ही निष्पादित होंगे।
*   \*\*उदाहरण: \*\*

`/etc/profile.d/somescript.sh`

```bash
#!/bin/bash
TEST=$(cat /var/somefile)
export $TEST
```

#### **केवल एक विशिष्ट उपयोगकर्ता के व्यवहार को प्रभावित करने वाली फ़ाइलें:**

* _**\~/.bashrc**_: यह फ़ाइल _/etc/bash.bashrc_ फ़ाइल की तरह काम करती है, लेकिन यह केवल एक विशिष्ट उपयोगकर्ता के लिए ही निष्पादित होती है। यदि आप अपने लिए एक पर्यावरण बनाना चाहते हैं, तो अपने होम निर्देशिका में इस फ़ाइल को संशोधित या बना सकते हैं।
* _**\~/.profile, \~/.bash\_profile, \~/.bash\_login**_: ये फ़ाइलें _/etc/profile_ की तरह ही हैं। इसके निष्पादन का तरीका में अंतर होता है। यह फ़ाइल केवल तब निष्पादित होती है जब उपयोगकर्ता, जिसके होम निर्देशिका में यह फ़ाइल मौजूद होती है, लॉगिन करता है।

**यहां से निकाला गया है:** [**यहां**](https://codeburst.io/linux-environment-variables-53cea0245dc9) **और** [**यहां**](https://www.gnu.org/software/bash/manual/html\_node/Bash-Startup-Files.html)

## सामान्य चर

से: [https://geek-university.com/linux/common-environment-variables/](https://geek-university.com/linux/common-environment-variables/)

* **DISPLAY** - **X** द्वारा उपयोग किया जाने वाला प्रदर्शन। इस चर को आमतौर पर **:0.0** पर सेट किया जाता है, जो मानवीय कंप्यूटर पर पहला प्रदर्शन को दर्शाता है।
* **EDITOR** - उपयोगकर्ता की पसंदीदा पाठ संपादक।
* **HISTFILESIZE** - इतिहास फ़ाइल में शामिल लाइनों की अधिकतम संख्या।
* **HISTSIZE** - उपयोगकर्ता अपने सत्र को समाप्त करते समय इतिहास फ़ाइल में जोड़ी गई पंक्तियों की संख्या
* **HOME** - आपका होम निर्देशिका।
* **HOSTNAME** - कंप्यूटर का होस्टनाम।
* **LANG** - आपकी वर्तमान भाषा।
* **MAIL** - उपयोगकर्ता के मेल स्पूल का स्थान। आमतौर पर **/var/spool/mail/USER**।
* **MANPATH** - मैनुअल पेज खोजने के लिए निर्देशिकाओं की सूची।
* **OSTYPE** - ऑपरेटिंग सिस्टम का प्रकार।
* **PS1** - बैश में डिफ़ॉल्ट प्रॉम्प्ट।
* **PATH** - सभी निर्देशिकाओं का पथ संग्रहीत करता है जिनमें बाइनरी फ़ाइलें होती हैं, जिन्हें आप फ़ाइल के नाम को निर्दिष्ट करके निष्पादित करना चाहते हैं और न कि संबंधित या पूर्ण पथ के द्वारा।
* **PWD** - वर्तमान कार्य निर्देशिका।
* **SHELL** - वर्तमान कमांड शेल का पथ (उदाहरण के लिए, **/bin/bash**)।
* **TERM** - वर्तमान टर्मिनल प्रकार (उदाहरण के लिए, **xterm**)।
* **TZ** - आपका समय क्षेत्र।
* **USER** - आपका वर्तमान उपयोगकर्ता नाम।

## हैकिंग के लिए रोचक चर

### **HISTFILESIZE**

इस चर की **मान को 0** पर बदलें, ताकि जब आप **अपना सत्र समाप्त** करें, तो इतिहास फ़ाइल (\~/.bash\_history) **हटा दी जाएगी**।
```bash
export HISTFILESIZE=0
```
### **HISTSIZE**

इस चर की मान को 0 परिवर्तित करें, ताकि जब आप अपनी सत्र समाप्त करें, कोई भी कमांड **इतिहास फ़ाइल** (\~/.bash\_history) में जोड़ी न जाए।
```bash
export HISTSIZE=0
```
### http\_proxy & https\_proxy

यहां घोषित **प्रॉक्सी** का उपयोग प्रक्रियाएं **http या https** के माध्यम से इंटरनेट से कनेक्ट होने के लिए करेंगी।
```bash
export http_proxy="http://10.10.10.10:8080"
export https_proxy="http://10.10.10.10:8080"
```
### SSL\_CERT\_FILE और SSL\_CERT\_DIR

प्रक्रियाएं **इन env variables** में निर्दिष्ट प्रमाणपत्रों पर विश्वास करेंगी।
```bash
export SSL_CERT_FILE=/path/to/ca-bundle.pem
export SSL_CERT_DIR=/path/to/ca-certificates
```
### PS1

अपने प्रॉम्प्ट की दिखावट को बदलें।

मैंने [**इसे**](https://gist.github.com/carlospolop/43f7cd50f3deea972439af3222b68808) बनाया है (दूसरे पर आधारित, कोड पढ़ें)।

रूट:

![](<../.gitbook/assets/image (87).png>)

सामान्य उपयोगकर्ता:

![](<../.gitbook/assets/image (88).png>)

एक, दो और तीन बैकग्राउंड जॉब्स:

![](<../.gitbook/assets/image (89).png>)

एक बैकग्राउंड जॉब, एक रोकी गई और अंतिम कमांड सही तरीके से पूरा नहीं हुआ:

![](<../.gitbook/assets/image (90).png>)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह।
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें और PRs सबमिट करें** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को**।

</details>
