# लिनक्स परिवेश चर

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके।

</details>

**Try Hard सुरक्षा समूह**

<figure><img src="../.gitbook/assets/telegram-cloud-document-1-5159108904864449420.jpg" alt=""><figcaption></figcaption></figure>

{% embed url="https://discord.gg/tryhardsecurity" %}

***

## वैश्विक चर

वैश्विक चर **बच्चा प्रक्रियाओं** द्वारा विरासत में मिलेगा।

आप निम्नलिखित करके अपने वर्तमान सत्र के लिए एक वैश्विक चर बना सकते हैं:
```bash
export MYGLOBAL="hello world"
echo $MYGLOBAL #Prints: hello world
```
यह चर प्रवेशयों और इसके चाइल्ड प्रक्रियाओं द्वारा पहुंचने योग्य होगा।

आप निम्नलिखित करके एक चर को **हटा** सकते हैं:
```bash
unset MYGLOBAL
```
## स्थानीय चर

**स्थानीय चर** केवल **वर्तमान शैल / स्क्रिप्ट** द्वारा **पहुंचे जा सकते** हैं।
```bash
LOCAL="my local"
echo $LOCAL
unset LOCAL
```
## वर्तमान चरों की सूची
```bash
set
env
printenv
cat /proc/$$/environ
cat /proc/`python -c "import os; print(os.getppid())"`/environ
```
## सामान्य चर

From: [https://geek-university.com/linux/common-environment-variables/](https://geek-university.com/linux/common-environment-variables/)

* **DISPLAY** – **X** द्वारा उपयोग किया गया प्रदर्शन। इस चर को आम तौर पर **:0.0** पर सेट किया जाता है, जिसका मतलब है कि वर्तमान कंप्यूटर पर पहला प्रदर्शन है।
* **EDITOR** – उपयोगकर्ता का पसंदीदा पाठ संपादक।
* **HISTFILESIZE** – इतिहास फ़ाइल में शामिल सवालों की अधिकतम संख्या।
* **HISTSIZE** – उपयोगकर्ता अपने सत्र को समाप्त करने पर इतिहास फ़ाइल में जोड़ी गई पंक्तियों की संख्या
* **HOME** – आपका होम निर्देशिका।
* **HOSTNAME** – कंप्यूटर का होस्टनाम।
* **LANG** – आपकी वर्तमान भाषा।
* **MAIL** – उपयोगकर्ता के मेल स्पूल का स्थान। सामान्यत: **/var/spool/mail/USER**।
* **MANPATH** – मैनुअल पेज खोजने के लिए निर्देशिकाओं की सूची।
* **OSTYPE** – ऑपरेटिंग सिस्टम का प्रकार।
* **PS1** – बैश में डिफ़ॉल्ट प्रॉम्प्ट।
* **PATH** – सभी निर्देशिकाओं का पथ संग्रहित करता है जिसमें बाइनरी फ़ाइलें हैं जिन्हें आप केवल फ़ाइल के नाम को निर्दिष्ट करके नहीं बल्कि सापेक्ष या पूर्ण पथ द्वारा क्रियान्वित करना चाहते हैं।
* **PWD** – वर्तमान काम कर रहा निर्देशिका।
* **SHELL** – वर्तमान कमांड शैल का पथ (उदाहरण के लिए, **/bin/bash**।)
* **TERM** – वर्तमान टर्मिनल प्रकार (उदाहरण के लिए, **xterm**।)
* **TZ** – आपका समय क्षेत्र।
* **USER** – आपका वर्तमान उपयोगकर्ता नाम।

## हैकिंग के लिए दिलचस्प चर

### **HISTFILESIZE**

**इस चर की मान को 0 पर बदलें**, ताकि जब आप **अपने सत्र को समाप्त** करते हैं, तो **इतिहास फ़ाइल** (\~/.bash\_history) **हटा दी जाएगी**।
```bash
export HISTFILESIZE=0
```
### **HISTSIZE**

इस चर की **मान को 0 परिवर्तित** करें, ताकि जब आप **अपनी सत्र समाप्त** करें, कोई भी कमांड **इतिहास फ़ाइल** (\~/.bash\_history) में जोड़ी न जाए।
```bash
export HISTSIZE=0
```
### http\_proxy & https\_proxy

यह प्रक्रियाएं **प्रॉक्सी** का उपयोग करेंगी जो **http या https** के माध्यम से इंटरनेट से कनेक्ट करने के लिए यहाँ घोषित किया गया है।
```bash
export http_proxy="http://10.10.10.10:8080"
export https_proxy="http://10.10.10.10:8080"
```
### SSL\_CERT\_FILE & SSL\_CERT\_DIR

प्रक्रियाएं **इन env variables** में उक्त प्रमाणपत्रों का विश्वास करेंगी।
```bash
export SSL_CERT_FILE=/path/to/ca-bundle.pem
export SSL_CERT_DIR=/path/to/ca-certificates
```
### PS1

अपने प्रॉम्प्ट की दिखावट को बदलें।

[**यह एक उदाहरण है**](https://gist.github.com/carlospolop/43f7cd50f3deea972439af3222b68808)

रूट:

![](<../.gitbook/assets/image (87).png>)

सामान्य उपयोगकर्ता:

![](<../.gitbook/assets/image (88).png>)

एक, दो और तीन पृष्ठभूमि पर चल रहे कार्य:

![](<../.gitbook/assets/image (89).png>)

एक पृष्ठभूमि पर कार्य, एक रोका गया और अंतिम कमांड सही ढंग से समाप्त नहीं हुआ:

![](<../.gitbook/assets/image (90).png>)

**Try Hard Security Group**

<figure><img src="../.gitbook/assets/telegram-cloud-document-1-5159108904864449420.jpg" alt=""><figcaption></figcaption></figure>

{% embed url="https://discord.gg/tryhardsecurity" %}

<details>

<summary><strong>शून्य से हीरो तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी कंपनी का विज्ञापन HackTricks में देखना चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live) पर **फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks) github repos में PRs सबमिट करके।

</details>
