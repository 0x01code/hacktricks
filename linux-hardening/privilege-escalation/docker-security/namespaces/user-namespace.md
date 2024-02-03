# यूजर नेमस्पेस

<details>

<summary><strong>शून्य से नायक तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS रेड टीम एक्सपर्ट)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**टेलीग्राम समूह**](https://t.me/peass) में या **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) को **फॉलो करें**.
* **अपनी हैकिंग ट्रिक्स को HackTricks** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपोज में PRs सबमिट करके **शेयर करें**.

</details>

## मूल जानकारी

यूजर नेमस्पेस एक लिनक्स कर्नेल फीचर है जो **यूजर और ग्रुप ID मैपिंग्स का अलगाव प्रदान करता है**, जिससे प्रत्येक यूजर नेमस्पेस के पास अपना **स्वयं का यूजर और ग्रुप IDs का सेट** हो सकता है। यह अलगाव प्रक्रियाओं को विभिन्न यूजर नेमस्पेस में चलाने की अनुमति देता है **विभिन्न विशेषाधिकार और स्वामित्व के साथ**, भले ही वे संख्यात्मक रूप से समान यूजर और ग्रुप IDs साझा करें।

यूजर नेमस्पेस विशेष रूप से कंटेनराइजेशन में उपयोगी हैं, जहां प्रत्येक कंटेनर के पास अपना स्वतंत्र यूजर और ग्रुप IDs का सेट होना चाहिए, जिससे कंटेनरों और होस्ट सिस्टम के बीच बेहतर सुरक्षा और अलगाव हो सके।

### यह कैसे काम करता है:

1. जब एक नया यूजर नेमस्पेस बनाया जाता है, तो यह **यूजर और ग्रुप ID मैपिंग्स के खाली सेट के साथ शुरू होता है**। इसका मतलब है कि नए यूजर नेमस्पेस में चलने वाली कोई भी प्रक्रिया **प्रारंभ में नेमस्पेस के बाहर कोई विशेषाधिकार नहीं रखेगी**।
2. ID मैपिंग्स को नए नेमस्पेस में यूजर और ग्रुप IDs के बीच और माता-पिता (या होस्ट) नेमस्पेस में उनके बीच स्थापित किया जा सकता है। यह **नए नेमस्पेस में प्रक्रियाओं को माता-पिता नेमस्पेस में यूजर और ग्रुप IDs के अनुरूप विशेषाधिकार और स्वामित्व प्रदान करता है**। हालांकि, ID मैपिंग्स को विशिष्ट रेंजों और IDs के उपसमूहों तक सीमित किया जा सकता है, जिससे नए नेमस्पेस में प्रक्रियाओं को दिए गए विशेषाधिकारों पर सूक्ष्म नियंत्रण हो सकता है।
3. एक यूजर नेमस्पेस के भीतर, **प्रक्रियाएं नेमस्पेस के अंदर के कार्यों के लिए पूर्ण रूट विशेषाधिकार (UID 0) रख सकती हैं**, जबकि नेमस्पेस के बाहर सीमित विशेषाधिकार रखती हैं। यह **कंटेनरों को अपने नेमस्पेस के भीतर रूट-जैसी क्षमताओं के साथ चलाने की अनुमति देता है बिना होस्ट सिस्टम पर पूर्ण रूट विशेषाधिकार के**।
4. प्रक्रियाएं `setns()` सिस्टम कॉल का उपयोग करके या `unshare()` या `clone()` सिस्टम कॉल का उपयोग करके `CLONE_NEWUSER` फ्लैग के साथ नए नेमस्पेस बना सकती हैं या नेमस्पेस के बीच जा सकती हैं। जब एक प्रक्रिया एक नए नेमस्पेस में जाती है या एक बनाती है, तो वह उस नेमस्पेस से जुड़ी यूजर और ग्रुप ID मैपिंग्स का उपयोग करना शुरू कर देगी।

## प्रयोगशाला:

### विभिन्न नेमस्पेस बनाएं

#### CLI
```bash
sudo unshare -U [--mount-proc] /bin/bash
```
`/proc` फाइलसिस्टम का एक नया इंस्टेंस माउंट करके, यदि आप `--mount-proc` पैरामीटर का उपयोग करते हैं, तो आप सुनिश्चित करते हैं कि नया माउंट नेमस्पेस उस नेमस्पेस के लिए विशिष्ट प्रक्रिया जानकारी का **सटीक और अलग दृश्य** रखता है।

<details>

<summary>एरर: bash: fork: Cannot allocate memory</summary>

`unshare` को `-f` विकल्प के बिना निष्पादित करने पर, नए PID (प्रोसेस ID) नेमस्पेस के निर्माण को लिनक्स कैसे संभालता है, उसके कारण एक एरर का सामना करना पड़ता है। मुख्य विवरण और समाधान नीचे दिए गए हैं:

1. **समस्या की व्याख्या**:
- लिनक्स कर्नेल एक प्रक्रिया को `unshare` सिस्टम कॉल का उपयोग करके नए नेमस्पेस बनाने की अनुमति देता है। हालांकि, नए PID नेमस्पेस के निर्माण की पहल करने वाली प्रक्रिया (जिसे "unshare" प्रक्रिया कहा जाता है) नए नेमस्पेस में प्रवेश नहीं करती है; केवल उसकी चाइल्ड प्रोसेसेस ही करती हैं।
- `%unshare -p /bin/bash%` चलाने से `/bin/bash` `unshare` के समान प्रक्रिया में शुरू होता है। नतीजतन, `/bin/bash` और उसकी चाइल्ड प्रोसेसेस मूल PID नेमस्पेस में होती हैं।
- नए नेमस्पेस में `/bin/bash` की पहली चाइल्ड प्रोसेस PID 1 बन जाती है। जब यह प्रोसेस बाहर निकलती है, तो यदि कोई अन्य प्रोसेस नहीं हैं, तो नेमस्पेस की सफाई ट्रिगर होती है, क्योंकि PID 1 का अनाथ प्रोसेसेस को गोद लेने का विशेष रोल होता है। लिनक्स कर्नेल तब उस नेमस्पेस में PID आवंटन को अक्षम कर देगा।

2. **परिणाम**:
- नए नेमस्पेस में PID 1 के बाहर निकलने से `PIDNS_HASH_ADDING` फ्लैग की सफाई हो जाती है। इससे `alloc_pid` फंक्शन नई प्रोसेस बनाते समय नए PID को आवंटित करने में विफल हो जाता है, जिससे "Cannot allocate memory" एरर उत्पन्न होता है।

3. **समाधान**:
- इस समस्या को `-f` विकल्प के साथ `unshare` का उपयोग करके हल किया जा सकता है। यह विकल्प `unshare` को नए PID नेमस्पेस बनाने के बाद एक नई प्रोसेस को फोर्क करने के लिए बनाता है।
- `%unshare -fp /bin/bash%` निष्पादित करने से सुनिश्चित होता है कि `unshare` कमांड स्वयं नए नेमस्पेस में PID 1 बन जाता है। `/bin/bash` और उसकी चाइल्ड प्रोसेसेस तब इस नए नेमस्पेस के भीतर सुरक्षित रूप से समाहित होती हैं, जिससे PID 1 के समय से पहले बाहर निकलने को रोका जा सकता है और सामान्य PID आवंटन की अनुमति दी जा सकती है।

`unshare` को `-f` फ्लैग के साथ चलाने से सुनिश्चित होता है कि नया PID नेमस्पेस सही ढंग से बनाए रखा जाता है, जिससे `/bin/bash` और उसकी सब-प्रोसेसेस बिना मेमोरी आवंटन एरर का सामना किए बिना काम कर सकती हैं।

</details>

#### Docker
```bash
docker run -ti --name ubuntu1 -v /usr:/ubuntu1 ubuntu bash
```
यूजर नेमस्पेस का उपयोग करने के लिए, Docker डेमॉन को **`--userns-remap=default`** के साथ शुरू करना आवश्यक है (उबुंटू 14.04 में, इसे `/etc/default/docker` में संशोधित करके और फिर `sudo service docker restart` को निष्पादित करके किया जा सकता है)

### &#x20;जांचें कि आपकी प्रक्रिया किस नेमस्पेस में है
```bash
ls -l /proc/self/ns/user
lrwxrwxrwx 1 root root 0 Apr  4 20:57 /proc/self/ns/user -> 'user:[4026531837]'
```
डॉकर कंटेनर से यूजर मैप की जांच करना संभव है:
```bash
cat /proc/self/uid_map
0          0 4294967295  --> Root is root in host
0     231072      65536  --> Root is 231072 userid in host
```
या होस्ट से:
```bash
cat /proc/<pid>/uid_map
```
### सभी User namespaces खोजें

{% code overflow="wrap" %}
```bash
sudo find /proc -maxdepth 3 -type l -name user -exec readlink {} \; 2>/dev/null | sort -u
# Find the processes with an specific namespace
sudo find /proc -maxdepth 3 -type l -name user -exec ls -l  {} \; 2>/dev/null | grep <ns-number>
```
{% endcode %}

### यूजर नेमस्पेस के अंदर प्रवेश करें
```bash
nsenter -U TARGET_PID --pid /bin/bash
```
इसके अलावा, आप किसी अन्य प्रक्रिया के **namespace में केवल तभी प्रवेश कर सकते हैं जब आप root हों**। और आप बिना किसी डिस्क्रिप्टर के अन्य namespace में **प्रवेश नहीं** कर सकते हैं (जैसे कि `/proc/self/ns/user`).

### नया User namespace बनाएं (मैपिंग्स के साथ)

{% code overflow="wrap" %}
```bash
unshare -U [--map-user=<uid>|<name>] [--map-group=<gid>|<name>] [--map-root-user] [--map-current-user]
```
Since there is no English text provided outside of the markdown syntax to translate, I cannot provide a translation. If you provide the relevant English text, I can then translate it into Hindi for you.
```bash
# Container
sudo unshare -U /bin/bash
nobody@ip-172-31-28-169:/home/ubuntu$ #Check how the user is nobody

# From the host
ps -ef | grep bash # The user inside the host is still root, not nobody
root       27756   27755  0 21:11 pts/10   00:00:00 /bin/bash
```
### क्षमताओं की पुनः प्राप्ति

यूजर नेमस्पेस के मामले में, **जब एक नया यूजर नेमस्पेस बनाया जाता है, तो उस नेमस्पेस में प्रवेश करने वाली प्रक्रिया को उस नेमस्पेस के भीतर पूर्ण सेट क्षमताएं प्रदान की जाती हैं**। ये क्षमताएं प्रक्रिया को विशेषाधिकार प्राप्त संचालन करने की अनुमति देती हैं जैसे कि **फाइलसिस्टम्स को माउंट करना**, डिवाइसेस बनाना, या फाइलों के स्वामित्व को बदलना, लेकिन **केवल अपने यूजर नेमस्पेस के संदर्भ में**।

उदाहरण के लिए, जब आपके पास एक यूजर नेमस्पेस के भीतर `CAP_SYS_ADMIN` क्षमता होती है, तो आप उन संचालनों को कर सकते हैं जिनके लिए आमतौर पर यह क्षमता आवश्यक होती है, जैसे कि फाइलसिस्टम्स को माउंट करना, लेकिन केवल अपने यूजर नेमस्पेस के संदर्भ में। आपके द्वारा इस क्षमता के साथ किए गए कोई भी संचालन होस्ट सिस्टम या अन्य नेमस्पेस पर प्रभाव नहीं डालेंगे।

{% hint style="warning" %}
इसलिए, यहां तक कि अगर एक नए यूजर नेमस्पेस के अंदर एक नई प्रक्रिया प्राप्त करने से **आपको सभी क्षमताएं वापस मिल जाती हैं** (CapEff: 000001ffffffffff), आप वास्तव में **केवल उन्हीं क्षमताओं का उपयोग कर सकते हैं जो नेमस्पेस से संबंधित हैं** (उदाहरण के लिए माउंट) लेकिन हर एक का नहीं। इसलिए, अकेले यह पर्याप्त नहीं है एक Docker कंटेनर से बचने के लिए।
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
# संदर्भ
* [https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory](https://stackoverflow.com/questions/44666700/unshare-pid-bin-bash-fork-cannot-allocate-memory)

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert) के साथ शून्य से नायक तक AWS हैकिंग सीखें</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सदस्यता योजनाएँ**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें.

</details>
