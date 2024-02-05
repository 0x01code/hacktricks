<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप चाहते हैं कि आपकी **कंपनी HackTricks में विज्ञापित हो** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** पर **फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos पर PR जमा करके।

</details>


`/proc` और `/sys` की अनुचित नेमस्पेस आइसोलेशन के बिना उजागर होना, हमें बड़े सुरक्षा जोखिमों का सामना करना पड़ता है, जैसे हमले की सतह विस्तार और सूचना उजागरी। ये निर्देशिकाएँ संवेदनशील फ़ाइलों को समेत हैं जो, अगर गलत रूप से कॉन्फ़िगर किए गए या अनधिकृत उपयोगकर्ता द्वारा एक्सेस किए गए, कंटेनर छूट, होस्ट में संशोधन, या आगे होने वाले हमलों की सहायता प्रदान कर सकती हैं। उदाहरण के लिए, `-v /proc:/host/proc` को गलती से माउंट करना AppArmor सुरक्षा को छोड़ सकता है क्योंकि इसकी पथ-आधारित प्रकृति के कारण `/host/proc` सुरक्षित नहीं रहता।

आप [https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts) में प्रत्येक संभावित दुरुपयोग के अधिक विवरण पा सकते हैं।

# procfs दुरुपयोग

## `/proc/sys`
यह निर्देशिका कर्नेल चरित्रियों को संशोधित करने की अनुमति देती है, सामान्यत: `sysctl(2)` के माध्यम से, और कई चिंहित उपनिर्देशिकाएँ शामिल हैं:

### **`/proc/sys/kernel/core_pattern`**
- [core(5)](https://man7.org/linux/man-pages/man5/core.5.html) में वर्णित।
- पहले 128 बाइट के तर्क के साथ कोर-फ़ाइल उत्पन्न होने पर कार्य को परिभाषित करने की अनुमति देता है। यदि फ़ाइल पाइप `|` के साथ शुरू होती है, तो यह कोड निष्पादन में ले जा सकता है।
- **परीक्षण और दुरुपयोग उदाहरण**:
```bash
[ -w /proc/sys/kernel/core_pattern ] && echo Yes # लेखन पहुंच की परीक्षा
cd /proc/sys/kernel
echo "|$overlay/shell.sh" > core_pattern # कस्टम हैंडलर सेट करें
sleep 5 && ./crash & # हैंडलर को ट्रिगर करें
```

### **`/proc/sys/kernel/modprobe`**
- [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html) में विस्तार से वर्णित।
- कर्नेल मॉड्यूल लोड करने के लिए आमंत्रित करने वाले कर्नेल मॉड्यूल लोडर का पथ शामिल है।
- **पहुंच की जांच उदाहरण**:
```bash
ls -l $(cat /proc/sys/kernel/modprobe) # modprobe की पहुंच की जांच करें
```

### **`/proc/sys/vm/panic_on_oom`**
- [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html) में संदर्भित।
- एक वैश्विक ध्वनि जो कर्नेल पैनिक या OOM किलर को आमंत्रित करती है जब एक OOM स्थिति घटित होती है।

### **`/proc/sys/fs`**
- [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html) के अनुसार, फ़ाइल सिस्टम के बारे में विकल्प और जानकारी शामिल है।
- लेखन पहुंच मेज़बान के खिलाफ विभिन्न विभाजन-सेवा हमले सक्षम कर सकती है।

### **`/proc/sys/fs/binfmt_misc`**
- अपने मैजिक नंबर के आधार पर गैर-मूलभूत बाइनरी प्रारूपों के लिए अनुप्रयोक्ताओं को पंजीकृत करने की अनुमति देता है।
- `/proc/sys/fs/binfmt_misc/register` लिखने योग्य है तो विशेषाधिकार या रूट शैली एक्सेस हो सकता है।
- संबंधित दुरुपयोग और स्पष्टीकरण:
- [binfmt_misc के माध्यम से गरीब आदमी का रूटकिट](https://github.com/toffan/binfmt_misc)
- विस्तृत ट्यूटोरियल: [वीडियो लिंक](https://www.youtube.com/watch?v=WBC7hhgMvQQ)

## `/proc` में अन्य

### **`/proc/config.gz`**
- `CONFIG_IKCONFIG_PROC` सक्षम होने पर कर्नेल कॉन्फ़िगरेशन को उजागर कर सकता है।
- चल रहे कर्नेल में दोषों की पहचान के लिए उपयोगी।

### **`/proc/sysrq-trigger`**
- Sysrq कमांड को आमंत्रित करने की अनुमति देता है, स्थानांतरण के लिए स्थानांतरण करने के लिए या अन्य महत्वपूर्ण कार्रवाई करने के लिए।
- **होस्ट को पुनरारंभ करने का उदाहरण**:
```bash
echo b > /proc/sysrq-trigger # होस्ट को पुनरारंभ करता है
```

### **`/proc/kmsg`**
- कर्नेल रिंग बफर संदेशों को उजागर करता है।
- कर्नेल दुरुपयोग, पते लीक, और संवेदनशील सिस्टम जानकारी प्रदान कर सकता है।

### **`/proc/kallsyms`**
- कर्नेल निर्यातित चिन्हों और उनके पतों की सूची देता है।
- कर्नेल दुरुपयोग विकास के लिए अत्यधिक महत्वपूर्ण, विशेष रूप से KASLR को पार करने के लिए।
- पता सूचना `kptr_restrict` को `1` या `2` पर सेट करके प्रतिबंधित है।
- [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html) में विवरण।

### **`/proc/[pid]/mem`**
- कर्नेल मेमोरी डिवाइस `/dev/mem` के साथ संवाद करता है।
- ऐतिहासिक रूप से विशेषाधिकार उन्नति हमलों के लिए वंशानुगत रहा है।
- [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html) पर अधिक।

### **`/proc/kcore`**
- एलएफ कोर प्रारूप में सिस्टम की भौतिक मेमोरी को प्रस्तुत करता है।
- पढ़ने से मेज़बान सिस्टम और अन्य कंटेनरों की मेमोरी सामग्री उजागर हो सकती है।
- बड़े फ़ाइल आकार से पढ़ने में समस्याएँ या सॉफ़्टवेयर क्रैश की ओर ले जा सकती हैं।
- [2019 में /proc/kcore को डंप करना](https://schlafwandler.github.io/posts/dumping-/proc/kcore/) में विस्तृत उपयोग।

### **`/proc/kmem`**
- कर्नेल वर्चुअल मेमोरी को प्रस्तुत करने वाले `/dev/kmem` के लिए वैकल्पिक इंटरफेस।
- पढ़ने और लिखने की अनुमति देता है, इसलिए कर्नेल मेमोरी का सीधा संशोधन करने की अनुमति है।

### **`/proc/mem`**
- विकल्पिक इंटरफेस `/dev/mem` के लिए, भौतिक मेमोरी को प्रस्तुत करता है।
- पढ़ने और लिखने की अनुमति देता है, सभी मेमोरी का संशोधन वर्चुअल से भौतिक पते तक पहुंचने की आवश्यकता है।

### **`/proc/sched_debug`**
- प्रक्रिया अनुस
