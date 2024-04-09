# संवेदनशील माउंट्स

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सदस्यता योजनाएं देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, HackTricks और HackTricks Cloud** github रेपो में PR जमा करके।

</details>

<figure><img src="../../../../.gitbook/assets/WebSec_1500x400_10fps_21sn_lightoptimized_v2.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}

`/proc` और `/sys` का अनुचित नेमस्पेस आइसोलेशन के बिना उजागर होना, हमें बड़े सुरक्षा जोखिमों का सामना करना पड़ता है, जिसमें हमले की सतह विस्तार और सूचना फासवणी शामिल है। ये निर्देशिकाएँ संवेदनशील फ़ाइलें शामिल करती हैं जो, अगर गलत रूप से कॉन्फ़िगर की गई हैं या अनधिकृत उपयोगकर्ता द्वारा पहुंची जाती हैं, तो कंटेनर से बाहर निकलने, होस्ट में संशोधन करने या आगे के हमलों की सहायता करने वाली जानकारी प्रदान कर सकती हैं। उदाहरण के लिए, `-v /proc:/host/proc` को गलती से माउंट करना AppArmor सुरक्षा को छोड़ सकता है क्योंकि इसका पथ-आधारित प्रकृति है, `/host/proc` को सुरक्षित नहीं छोड़ता।

**आप प्रत्येक संभावित दुरुपयोग का और विवरण पा सकते हैं** [**https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts**](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts)**.**

## procfs दुरुपयोग

### `/proc/sys`

यह निर्देशिका कर्नेल चर वेरिएबल को संशोधित करने की अनुमति देती है, सामान्यत: `sysctl(2)` के माध्यम से, और कई चिंहित उपनिर्देशिकाएँ शामिल हैं:

#### **`/proc/sys/kernel/core_pattern`**

* [core(5)](https://man7.org/linux/man-pages/man5/core.5.html) में वर्णित।
* कोर-फ़ाइल उत्पन्न होने पर पहले 128 बाइट को तार्किक रूप से निर्धारित करने की अनुमति देता है। यदि फ़ाइल पाइप `|` के साथ शुरू होती है, तो यह कोड निष्पादन में ले जा सकता है।
*   **परीक्षण और दुरुपयोग उदाहरण**:

```bash
[ -w /proc/sys/kernel/core_pattern ] && echo Yes # लेखन पहुंच का परीक्षण करें
cd /proc/sys/kernel
echo "|$overlay/shell.sh" > core_pattern # कस्टम हैंडलर सेट करें
sleep 5 && ./crash & # हैंडलर को ट्रिगर करें
```

#### **`/proc/sys/kernel/modprobe`**

* [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html) में विस्तार से वर्णित।
* कर्नेल मॉड्यूल लोड करने के लिए आमंत्रित करने वाले कर्नेल मॉड्यूल लोडर का पथ शामिल है।
*   **पहुंच की जांच उदाहरण**:

```bash
ls -l $(cat /proc/sys/kernel/modprobe) # modprobe तक पहुंच की जांच करें
```

#### **`/proc/sys/vm/panic_on_oom`**

* [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html) में संदर्भित।
* एक वैश्विक ध्वनि जो कर्नेल पैनिक या OOM किलर को आमंत्रित करती है जब OOM स्थिति घटित होती है।

#### **`/proc/sys/fs`**

* [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html) के अनुसार, फ़ाइल सिस्टम के बारे में विकल्प और जानकारी शामिल है।
* लेखन पहुंच से मेज़बान के खिलाफ विभिन्न सेवा अस्वीकृति हमले सक्षम हो सकते हैं।

#### **`/proc/sys/fs/binfmt_misc`**

* अपने मैजिक नंबर के आधार पर गैर-मूल बाइनरी प्रारूपों के लिए अनुप्रयोक्ताओं को पंजीकृत करने की अनुमति देता है।
* `/proc/sys/fs/binfmt_misc/register` लिखने योग्य होने पर विशेषाधिकार उन्नति या रूट शैली पहुंच देने की ओर ले जा सकता है।
* संबंधित दुरुपयोग और स्पष्टीकरण:
* [binfmt\_misc के माध्यम से गरीब आदमी का रूटकिट](https://github.com/toffan/binfmt\_misc)
* विस्तृत ट्यूटोरियल: [वीडियो लिंक](https://www.youtube.com/watch?v=WBC7hhgMvQQ)

### `/proc` में अन्य

#### **`/proc/config.gz`**

* यदि `CONFIG_IKCONFIG_PROC` सक्षम है, तो कर्नेल कॉन्फ़िगरेशन का पता लगा सकता है।
* चल रहे कर्नेल में दोषों की पहचान करने के लिए उपयोगी।

#### **`/proc/sysrq-trigger`**

* Sysrq कमांड को आमंत्रित करने की अनुमति देता है, जो तत्काल सिस्टम पुनरारंभ या अन्य महत्वपूर्ण क्रियाएँ का कारण बन सकता है।
*   **मेज़बान को पुनरारंभ करने का उदाहरण**:

```bash
echo b > /proc/sysrq-trigger # मेज़बान को पुनरारंभ करता है
```

#### **`/proc/kmsg`**

* कर्नेल रिंग बफर संदेशों को उजागर करता है।
* कर्नेल दोषों, पते फासवणियों में सहायक हो सकता है और संवेदनशील सिस्टम जानकारी प्रदान कर सकता है।

#### **`/proc/kallsyms`**

* कर्नेल निर्यातित चिन्हों और उनके पतों की सूची देता है।
* कर्नेल दोष विकास के लिए आवश्यक है, विशेषतः KASLR को पार करने के लिए।
* पता सूचना `kptr_restrict` को `1` या `2` पर सेट करके प्रतिबंधित है।
* [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html) में विवरण।

#### **`/proc/[pid]/mem`**

* कर्नेल मेमोरी डिवाइस `/dev/mem` के साथ संवाद करता है।
* ऐतिहासिक रूप से विशेषाधिकार उन्नति हमलों के लिए विकल्प है।
* [proc(5)](https://man7.org/linux/man-pages/man5/proc.5.html) पर अधिक।

#### **`/proc/kcore`**

* ईएलएफ कोर प्रारूप में सिस्टम की भौतिक मेमोरी को प्रतिनिधित करता है।
* पढ़ने से मेज़बान सिस्टम और अन्य कंटेनरों की मेमोरी सामग्री लीक हो सकती है।
* बड़े फ़ाइल आकार से पढ़ने में समस्याएँ या सॉफ़्टवेयर क्रैश हो सकता है।
* [2019 में /proc/kcore को डंप करने का उपयोग](https://schlafwandler.github.io/posts/dumping-/proc/kcore/) में विस्तृत उपयोग।

#### **`/proc/kmem`**

* `/dev/kmem` के लिए वैकल्पिक इंटरफेस, कर्नेल वर्चुअल मेमोरी का प्रतिनिधित करता है।
* पढ़ने और लिखने की अनुमति देता है, इसलिए कर्नेल मेमोरी का सीधा संशोधन करने की अनुमति देता है।

#### **`/proc/mem`**

* `/dev/mem` के लिए वैकल्पिक इंटरफेस, भौतिक मेमोरी का प्रतिनिधित करता ह
#### **`/sys/class/thermal`**

* तापमान सेटिंग को नियंत्रित करता है, संभावित तौर पर DoS हमले या भौतिक क्षति का कारण बन सकता है।

#### **`/sys/kernel/vmcoreinfo`**

* कर्नेल पतों को लीक कर सकता है, केएएसएलआर को कमजोर कर सकता है।

#### **`/sys/kernel/security`**

* `securityfs` इंटरफेस को होस्ट की तरह कॉन्फ़िगर करने की अनुमति देता है, जैसे AppArmor जैसे लिनक्स सुरक्षा मॉड्यूल।
* पहुंच एक कंटेनर को अपनी MAC सिस्टम को अक्षम करने की अनुमति दे सकती है।

#### **`/sys/firmware/efi/vars` और `/sys/firmware/efi/efivars`**

* NVRAM में EFI वेरिएबल्स के साथ इंटरैक्ट करने के लिए इंटरफेस उजागर करता है।
* गलत कॉन्फ़िगरेशन या शोषण लैपटॉप को ब्रिक करने या अनबूटेबल होस्ट मशीन की ओर ले जा सकता है।

#### **`/sys/kernel/debug`**

* `debugfs` कर्नेल के लिए "कोई नियम" डीबगिंग इंटरफेस प्रदान करता है।
* इसकी असीमित प्रकृति के कारण सुरक्षा समस्याओं का इतिहास है।

### संदर्भ

* [https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts)
* [Understanding and Hardening Linux Containers](https://research.nccgroup.com/wp-content/uploads/2020/07/ncc\_group\_understanding\_hardening\_linux\_containers-1-1.pdf)
* [Abusing Privileged and Unprivileged Linux Containers](https://www.nccgroup.com/globalassets/our-research/us/whitepapers/2016/june/container\_whitepaper.pdf)

<figure><img src="../../../../.gitbook/assets/WebSec_1500x400_10fps_21sn_lightoptimized_v2.gif" alt=""><figcaption></figcaption></figure>

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
