# संवेदनशील माउंट्स

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी कंपनी का विज्ञापन **HackTricks** में देखना चाहते हैं या **HackTricks को PDF में डाउनलोड** करना चाहते हैं तो [**सब्सक्रिप्शन प्लान**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारा संग्रह [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, हैकट्रिक्स** और **हैकट्रिक्स क्लाउड** गिटहब रेपो में पीआर जमा करके।

</details>

<figure><img src="../../../..https://pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}

`/proc` और `/sys` की अनुचित नेमस्पेस आइसोलेशन के बिना उद्घाटन सुरक्षा जोखिम बढ़ाता है, जिसमें हमले की सतह विस्तार और सूचना फासला शामिल है। यदि ये निर्धारित या अनधिकृत उपयोगकर्ता द्वारा पहुंचे जाते हैं, तो ये निर्धारित फ़ाइलें कंटेनर छूट, होस्ट संशोधन, या आगे के हमलों की सहायता करने वाली जानकारी प्रदान कर सकती हैं। उदाहरण के लिए, गलती से `-v /proc:/host/proc` माउंट करना AppArmor सुरक्षा को छोड़कर `/host/proc` को सुरक्षित छोड़ सकता है।

**आप प्रत्येक संभावित दुर्बलता का विस्तार से विवरण यहाँ पा सकते हैं** [**https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts**](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts)**.**

## procfs दुर्बलताएँ

### `/proc/sys`

यह निर्देशिका कर्नेल चरित्रियों को संशोधित करने की अनुमति देती है, सामान्यत: `sysctl(2)` के माध्यम से, और कई चिंहित उपनिर्देशिकाएं शामिल हैं:

#### **`/proc/sys/kernel/core_pattern`**

* [core(5)](https://man7.org/linux/man-pages/man5/core.5.html) में वर्णित।
* पहले 128 बाइट के साथ कोर-फ़ाइल उत्पन्न करने पर कार्य को परिभाषित करने की अनुमति देता है। यदि फ़ाइल पाइप `|` के साथ शुरू होती है, तो यह कोड निष्पादन में ले जा सकता है।
*   **परीक्षण और शोध उदाहरण**:

```bash
[ -w /proc/sys/kernel/core_pattern ] && echo Yes # लेखन पहुंच का परीक्षण करें
cd /proc/sys/kernel
echo "|$overlay/shell.sh" > core_pattern # कस्टम हैंडलर सेट करें
sleep 5 && ./crash & # हैंडलर को ट्रिगर करें
```

#### **`/proc/sys/kernel/modprobe`**

* [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html) में विस्तार से वर्णित।
* कर्नेल मॉड्यूल लोडर के पथ को समाहित करता है, जो कर्नेल मॉड्यूल लोड करने के लिए आह्वानित किया जाता है।
*   **पहुंच की जांच उदाहरण**:

```bash
ls -l $(cat /proc/sys/kernel/modprobe) # modprobe की पहुंच की जांच करें
```

#### **`/proc/sys/vm/panic_on_oom`**

* [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html) में संदर्भित।
* एक वैश्विक ध्वनि जो कर्नेल पैनिक या OOM किलर को आम शर्त पर कैसे आह्वानित करता है।

#### **`/proc/sys/fs`**

* [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html) के अनुसार, फ़ाइल सिस्टम के बारे में विकल्प और जानकारी शामिल है।
* लेखन पहुंच होस्ट के खिलाफ विभिन्न विभाजन-सेवा हमलों को सक्षम कर सकती है।

#### **`/proc/sys/fs/binfmt_misc`**

* अपने मैजिक नंबर के आधार पर गैर-मूलभूत बाइनरी प्रारूपों के लिए अनुप्रयोगकर्ताओं को पंजीकृत करने की अनुमति देता है।
* `/proc/sys/fs/binfmt_misc/register` लिखने योग्य है तो विशेषाधिकार या रूट शैली पहुंच देने की संभावना है।
* संबंधित उत्पीड़न और स्पष्टीकरण:
* [binfmt\_misc के माध्यम से गरीब आदमी का रूटकिट](https://github.com/toffan/binfmt\_misc)
* विस्तृत ट्यूटोरियल: [वीडियो लिंक](https://www.youtube.com/watch?v=WBC7hhgMvQQ)

### `/proc` में अन्य

#### **`/proc/config.gz`**

* यदि `CONFIG_IKCONFIG_PROC` सक्षम है, तो कर्नेल कॉन्फ़िगरेशन को उजागर कर सकता है।
* चल रहे कर्नेल में दुर्बलताओं की पहचान करने के लिए उपयोगी।

#### **`/proc/sysrq-trigger`**

* Sysrq कमांड को आह्वानित करने की अनुमति देता है, जो तत्काल सिस्टम पुनरारंभ या अन्य महत्वपूर्ण क्रियाएँ कर सकता है।
*   **होस्ट को पुनरारंभ करने का उदाहरण**:

```bash
echo b > /proc/sysrq-trigger # होस्ट को पुनरारंभ करता है
```

#### **`/proc/kmsg`**

* कर्नेल रिंग बफर संदेशों को उजागर करता है।
* कर्नेल उत्पीड़नों में सहायक हो सकता है, पते लीक कर सकता है, और संवेदनशील सिस्टम जानकारी प्रदान कर सकता है।

#### **`/proc/kallsyms`**

* कर्नेल निर्यातित चिन्हों और उनके पतों की सूची देता है।
* कर्नेल उत्पीड़न विकास के लिए आवश्यक है, विशेषतः KASLR को पार करने के लिए।
* पता सूचना `kptr_restrict` को `1` या `2` पर सेट करके प्रतिबंधित है।
* [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html) में विवरण।

#### **`/proc/[pid]/mem`**

* कर्नेल मेमोरी डिवाइस `/dev/mem` के साथ संवाद करता है।
* ऐतिहासिक रूप से दुर्बलता उन्नति हमलों के लिए विकल्प है।
* [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html) पर अधिक।

#### **`/proc/kcore`**

* एलएफ कोर प्रारूप में सिस्टम की भौतिक मेमोरी को प्रतिनिधित करता है।
* पढ़ने से होस्ट सिस्टम और अन्य कंटेनरों की मेमोरी सामग्री लीक हो सकती है।
* बड़े फ़ाइल आकार पढ़ने में समस्याएँ या सॉफ़्टवेयर क्रैश कर सकती हैं।
* [2019 में /proc/kcore को डंप करने का उपयोग](https://schlafwandler.github.io/posts/dumping-/proc/kcore/) में विस्तृत उपयोग।

#### **`/proc/kmem`**

* `/dev/kmem` के लिए वैकल्पिक इंटरफेस, कर्नेल वर्चुअल मेमोरी को प्रतिनिधित करता है।
* पढ़ने और लिखने की अनुमति देता है, इसलिए कर्नेल मेमोरी का सीधा संशोधन करने की संभावना है।

#### **`/proc/mem`**

* `/dev/mem` के लिए वैकल्पिक इंटरफेस, भौतिक मेमोरी को प्रतिनिधित करता है।
* पढ़ने और लिखने की अनुमति देता है, सभी मेमोरी का संशोधन वर्चुअल से भौतिक पते तक पहुंचना आवश्यक है
#### **`/sys/class/thermal`**

* तापमान सेटिंग को नियंत्रित करता है, संभावित डीओएस हमले या भौतिक क्षति का कारण बन सकता है।

#### **`/sys/kernel/vmcoreinfo`**

* कर्नेल पतों को लीक करता है, केएएसएलआर को कमजोर कर सकता है।

#### **`/sys/kernel/security`**

* `securityfs` इंटरफेस को होस्ट की तरह कॉन्फ़िगर करने की अनुमति देता है, जैसे AppArmor जैसे लिनक्स सुरक्षा मॉड्यूल।
* पहुंच एक कंटेनर को अपनी एमएसी प्रणाली को अक्षम करने की अनुमति दे सकती है।

#### **`/sys/firmware/efi/vars` और `/sys/firmware/efi/efivars`**

* NVRAM में EFI वेरिएबल्स के साथ इंटरफेस उजागर करता है।
* गलत कॉन्फ़िगरेशन या शोषण ब्रिक्ड लैपटॉप या अनबूटेबल होस्ट मशीन की ओर ले जा सकता है।

#### **`/sys/kernel/debug`**

* `debugfs` कर्नेल के लिए "कोई नियम" डीबग इंटरफेस प्रदान करता है।
* इसकी असीमित प्रकृति के कारण सुरक्षा समस्याओं का इतिहास है।

### संदर्भ

* [https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts)
* [Understanding and Hardening Linux Containers](https://research.nccgroup.com/wp-content/uploads/2020/07/ncc\_group\_understanding\_hardening\_linux\_containers-1-1.pdf)
* [Abusing Privileged and Unprivileged Linux Containers](https://www.nccgroup.com/globalassets/our-research/us/whitepapers/2016/june/container\_whitepaper.pdf)

<figure><img src="../../../..https://pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}

<details>

<summary><strong>Learn AWS hacking from zero to hero with</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Other ways to support HackTricks:

* If you want to see your **company advertised in HackTricks** or **download HackTricks in PDF** Check the [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Get the [**official PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Discover [**The PEASS Family**](https://opensea.io/collection/the-peass-family), our collection of exclusive [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Join the** 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) or the [**telegram group**](https://t.me/peass) or **follow** us on **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Share your hacking tricks by submitting PRs to the** [**HackTricks**](https://github.com/carlospolop/hacktricks) and [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
