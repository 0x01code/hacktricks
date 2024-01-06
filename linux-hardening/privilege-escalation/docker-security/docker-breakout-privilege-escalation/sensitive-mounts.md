<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग प्राप्त करें**](https://peass.creator-spring.com)
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स शेयर करें।

</details>


(_**यह जानकारी यहाँ से ली गई थी**_ [_**https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts**_](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts))

नेमस्पेस सपोर्ट की कमी के कारण, `/proc` और `/sys` का उजागर होना महत्वपूर्ण हमले की सतह और सूचना लीक का स्रोत प्रदान करता है। `procfs` और `sysfs` के अनेक फाइलें कंटेनर एस्केप, होस्ट मॉडिफिकेशन या बेसिक सूचना लीक के लिए जोखिम प्रदान करती हैं, जो अन्य हमलों को सुविधाजनक बना सकती हैं।

इन तकनीकों का दुरुपयोग करने के लिए केवल `-v /proc:/host/proc` जैसी कुछ गलत कॉन्फ़िगर करना पर्याप्त हो सकता है क्योंकि **AppArmor `/host/proc` की रक्षा नहीं करता क्योंकि AppArmor पथ आधारित है**

# procfs

## /proc/sys

`/proc/sys` आमतौर पर कर्नेल वेरिएबल्स को संशोधित करने की अनुमति देता है, जिसे अक्सर `sysctl(2)` के माध्यम से नियंत्रित किया जाता है।

### /proc/sys/kernel/core\_pattern

[/proc/sys/kernel/core\_pattern](https://man7.org/linux/man-pages/man5/core.5.html) एक प्रोग्राम को परिभाषित करता है जो कोर-फाइल जनरेशन (आमतौर पर एक प्रोग्राम क्रैश) पर निष्पादित होता है और यदि इस फाइल का पहला अक्षर पाइप प्रतीक `|` है, तो कोर फाइल को मानक इनपुट के रूप में पास किया जाता है। यह प्रोग्राम रूट यूजर द्वारा चलाया जाता है और कमांड लाइन आर्ग्युमेंट्स के 128 बाइट्स तक की अनुमति देता है। यह किसी भी क्रैश और कोर फाइल जनरेशन के दौरान (जिसे दुर्भावनापूर्ण क्रियाओं के दौरान सरलता से त्यागा जा सकता है) कंटेनर होस्ट के भीतर तुच्छ कोड निष्पादन की अनुमति देगा।
```bash
[ -w /proc/sys/kernel/core_pattern ] && echo Yes #For testing
cd /proc/sys/kernel
echo "|$overlay/shell.sh" > core_pattern
sleep 5 && ./crash &
```
### /proc/sys/kernel/modprobe

[/proc/sys/kernel/modprobe](https://man7.org/linux/man-pages/man5/proc.5.html) में कर्नेल मॉड्यूल लोडर का पथ होता है, जिसे कर्नेल मॉड्यूल लोड करते समय बुलाया जाता है, जैसे कि [modprobe](https://man7.org/linux/man-pages/man8/modprobe.8.html) कमांड के माध्यम से। कोड निष्पादन प्राप्त किया जा सकता है किसी भी क्रिया को अंजाम देकर जो कर्नेल को कर्नेल मॉड्यूल लोड करने का प्रयास करने के लिए उत्प्रेरित करेगी (जैसे कि क्रिप्टो-API का उपयोग करके वर्तमान में अनलोडेड क्रिप्टो-मॉड्यूल लोड करना, या ifconfig का उपयोग करके एक नेटवर्किंग मॉड्यूल लोड करना जो वर्तमान में इस्तेमाल नहीं किया जा रहा हो)।
```bash
# Check if you can directly access modprobe
ls -l `cat /proc/sys/kernel/modprobe`
```
### /proc/sys/vm/panic_on_oom

[/proc/sys/vm/panic_on_oom](https://man7.org/linux/man-pages/man5/proc.5.html) एक वैश्विक ध्वज है जो निर्धारित करता है कि कर्नेल तब पैनिक होगा जब एक Out of Memory (OOM) स्थिति हिट होती है (बजाय OOM किलर को आमंत्रित करने के)। यह एक Denial of Service (DoS) हमले से अधिक है बजाय कंटेनर एस्केप के, लेकिन यह एक क्षमता को उजागर करता है जो केवल होस्ट के लिए उपलब्ध होनी चाहिए।

### /proc/sys/fs

[/proc/sys/fs](https://man7.org/linux/man-pages/man5/proc.5.html) निर्देशिका फाइल सिस्टम के विभिन्न पहलुओं से संबंधित विकल्पों और जानकारी का एक सरणी होती है, जिसमें कोटा, फाइल हैंडल, इनोड, और डेंट्री जानकारी शामिल है। इस निर्देशिका को लिखने की पहुंच होस्ट के खिलाफ विभिन्न Denial-of-Service हमलों को सक्षम करेगी।

### /proc/sys/fs/binfmt_misc

[/proc/sys/fs/binfmt_misc](https://man7.org/linux/man-pages/man5/proc.5.html) विविध बाइनरी प्रारूपों को निष्पादित करने की अनुमति देता है, जिसका आमतौर पर यह मतलब होता है कि विभिन्न **interpreters को गैर-मूल बाइनरी** प्रारूपों के लिए उनके मैजिक नंबर के आधार पर पंजीकृत किया जा सकता है (जैसे कि Java)। आप कर्नेल को एक बाइनरी निष्पादित करने के लिए हैंडलर्स के रूप में पंजीकृत कर सकते हैं।\
आप एक एक्सप्लॉइट [https://github.com/toffan/binfmt_misc](https://github.com/toffan/binfmt_misc) में पा सकते हैं: _Poor man's rootkit, leverage_ [_binfmt_misc_](https://github.com/torvalds/linux/raw/master/Documentation/admin-guide/binfmt-misc.rst)_'s_ [_credentials_](https://github.com/torvalds/linux/blame/3bdb5971ffc6e87362787c770353eb3e54b7af30/Documentation/binfmt_misc.txt#L62) _विकल्प का उपयोग करके किसी भी suid बाइनरी के माध्यम से प्रिविलेज एस्केलेट करने के लिए (और एक रूट शेल प्राप्त करने के लिए) यदि `/proc/sys/fs/binfmt_misc/register` लिखने योग्य है।_

इस तकनीक की अधिक गहराई से व्याख्या के लिए देखें [https://www.youtube.com/watch?v=WBC7hhgMvQQ](https://www.youtube.com/watch?v=WBC7hhgMvQQ)

## /proc/config.gz

[/proc/config.gz](https://man7.org/linux/man-pages/man5/proc.5.html) `CONFIG_IKCONFIG_PROC` सेटिंग्स के आधार पर, यह चल रहे कर्नेल के लिए कर्नेल कॉन्फ़िगरेशन विकल्पों का एक संपीड़ित संस्करण प्रदर्शित करता है। यह एक समझौता या दुर्भावनापूर्ण कंटेनर को कर्नेल में सक्षम असुरक्षित क्षेत्रों को आसानी से खोजने और लक्षित करने की अनुमति दे सकता है।

## /proc/sysrq-trigger

`Sysrq` एक पुरानी तंत्र है जिसे एक विशेष `SysRq` कीबोर्ड संयोजन के माध्यम से आमंत्रित किया जा सकता है। यह सिस्टम का तत्काल पुनरारंभ, `sync(2)` का जारी करना, सभी फाइल सिस्टम्स को पढ़ने के लिए केवल-पढ़ने के रूप में रीमाउंट करना, कर्नेल डिबगर्स को आमंत्रित करना, और अन्य संचालन करने की अनुमति दे सकता है।

यदि अतिथि ठीक से अलग नहीं किया गया है, तो वह [sysrq](https://www.kernel.org/doc/html/v4.11/admin-guide/sysrq.html) आदेशों को ट्रिगर कर सकता है `/proc/sysrq-trigger` फाइल में अक्षर लिखकर।
```bash
# Reboot the host
echo b > /proc/sysrq-trigger
```
## /proc/kmsg

[/proc/kmsg](https://man7.org/linux/man-pages/man5/proc.5.html) से कर्नेल रिंग बफर संदेशों का पता चलता है जिसे आमतौर पर `dmesg` के माध्यम से एक्सेस किया जाता है। इस जानकारी का खुलासा कर्नेल एक्सप्लॉइट्स में मदद कर सकता है, कर्नेल पते की लीक को ट्रिगर कर सकता है (जिसका उपयोग कर्नेल एड्रेस स्पेस लेआउट रैंडमाइजेशन (KASLR) को हराने में मदद कर सकता है), और कर्नेल, हार्डवेयर, ब्लॉक किए गए पैकेट्स और अन्य सिस्टम विवरणों के बारे में सामान्य जानकारी का स्रोत हो सकता है।

## /proc/kallsyms

[/proc/kallsyms](https://man7.org/linux/man-pages/man5/proc.5.html) में कर्नेल निर्यात प्रतीकों और उनके पते के स्थानों की सूची होती है जो डायनामिक और लोडेबल मॉड्यूल्स के लिए होती है। इसमें भौतिक मेमोरी में कर्नेल की छवि का स्थान भी शामिल है, जो कर्नेल एक्सप्लॉइट विकास के लिए सहायक है। इन स्थानों से, कर्नेल का बेस पता या ऑफसेट पाया जा सकता है, जिसका उपयोग कर्नेल एड्रेस स्पेस लेआउट रैंडमाइजेशन (KASLR) को पराजित करने में किया जा सकता है।

`kptr_restrict` को `1` या `2` पर सेट किए गए सिस्टम्स के लिए, यह फाइल मौजूद होगी लेकिन कोई पता जानकारी प्रदान नहीं करेगी (हालांकि प्रतीकों की सूची मेमोरी में उनके क्रम के समान है)।

## /proc/\[pid]/mem

[/proc/\[pid\]/mem](https://man7.org/linux/man-pages/man5/proc.5.html) कर्नेल मेमोरी डिवाइस `/dev/mem` के इंटरफेस को उजागर करता है। जबकि PID नेमस्पेस कुछ हमलों से सुरक्षा प्रदान कर सकता है इस `procfs` वेक्टर के माध्यम से, यह क्षेत्र ऐतिहासिक रूप से कमजोर रहा है, फिर सुरक्षित माना गया और फिर से विशेषाधिकार वृद्धि के लिए [कमजोर](https://git.zx2c4.com/CVE-2012-0056/about/) पाया गया।

## /proc/kcore

[/proc/kcore](https://man7.org/linux/man-pages/man5/proc.5.html) सिस्टम की भौतिक मेमोरी का प्रतिनिधित्व करता है और यह एक ELF कोर फॉर्मेट में होता है (जो आमतौर पर कोर डंप फाइलों में पाया जाता है)। यह उस मेमोरी में लिखने की अनुमति नहीं देता है। इस फाइल को पढ़ने की क्षमता (विशेषाधिकार प्राप्त उपयोगकर्ताओं तक सीमित) मेजबान सिस्टम और अन्य कंटेनरों से मेमोरी सामग्री को लीक कर सकती है।

बड़े रिपोर्ट किए गए फाइल आकार का प्रतिनिधित्व आर्किटेक्चर के लिए भौतिक रूप से पता लगाने योग्य मेमोरी की अधिकतम मात्रा का होता है, और इसे पढ़ते समय समस्याएं पैदा कर सकता है (या सॉफ्टवेयर की नाजुकता के आधार पर क्रैश हो सकता है)।

[Dumping /proc/kcore in 2019](https://schlafwandler.github.io/posts/dumping-/proc/kcore/)

## /proc/kmem

`/proc/kmem` [/dev/kmem](https://man7.org/linux/man-pages/man4/kmem.4.html) के लिए एक वैकल्पिक इंटरफेस है (जिसकी सीधी पहुंच को cgroup डिवाइस व्हाइटलिस्ट द्वारा अवरुद्ध किया गया है), जो कर्नेल वर्चुअल मेमोरी का प्रतिनिधित्व करने वाली एक कैरेक्टर डिवाइस फाइल है। यह पढ़ने और लिखने दोनों की अनुमति देता है, जिससे कर्नेल मेमोरी का सीधा संशोधन संभव है।

## /proc/mem

`/proc/mem` [/dev/mem](https://man7.org/linux/man-pages/man4/kmem.4.html) के लिए एक वैकल्पिक इंटरफेस है (जिसकी सीधी पहुंच को cgroup डिवाइस व्हाइटलिस्ट द्वारा अवरुद्ध किया गया है), जो सिस्टम की भौतिक मेमोरी का प्रतिनिधित्व करने वाली एक कैरेक्टर डिवाइस फाइल है। यह पढ़ने और लिखने दोनों की अनुमति देता है, जिससे सभी मेमोरी का संशोधन संभव है। (यह `kmem` की तुलना में थोड़ी अधिक सूक्ष्मता की मांग करता है, क्योंकि वर्चुअल पतों को पहले भौतिक पतों में हल करना पड़ता है)।

## /proc/sched\_debug

`/proc/sched_debug` एक विशेष फाइल है जो पूरे सिस्टम के लिए प्रोसेस शेड्यूलिंग जानकारी वापस करती है। इस जानकारी में सभी नेमस्पेसों से प्रोसेस नाम और प्रोसेस आईडी के अलावा प्रोसेस cgroup पहचानकर्ता शामिल होते हैं। यह प्रभावी रूप से PID नेमस्पेस सुरक्षा को दरकिनार करता है और यह अन्य/विश्व पठनीय है, इसलिए इसका उपयोग अनाधिकृत कंटेनरों में भी किया जा सकता है।

## /proc/\[pid]/mountinfo

[/proc/\[pid\]/mountinfo](https://man7.org/linux/man-pages/man5/proc.5.html) में प्रोसेस के माउंट नेमस्पेस में माउंट पॉइंट्स के बारे में जानकारी होती है। यह कंटेनर `rootfs` या इमेज का स्थान उजागर करता है।

# sysfs

## /sys/kernel/uevent\_helper

`uevents` वे घटनाएं हैं जो कर्नेल द्वारा ट्रिगर की जाती हैं जब कोई डिवाइस जोड़ा या हटाया जाता है। विशेष रूप से, `uevent_helper` का पथ `/sys/kernel/uevent_helper` में लिखकर संशोधित किया जा सकता है। फिर, जब एक `uevent` ट्रिगर किया जाता है (जो `/sys/class/mem/null/uevent` जैसी फाइलों में लिखकर उपयोगकर्ता भूमि से भी किया जा सकता है), दुर्भावनापूर्ण `uevent_helper` निष्पादित होता है।
```bash
# Creates a payload
cat "#!/bin/sh" > /evil-helper
cat "ps > /output" >> /evil-helper
chmod +x /evil-helper
# Finds path of OverlayFS mount for container
# Unless the configuration explicitly exposes the mount point of the host filesystem
# see https://ajxchapman.github.io/containers/2020/11/19/privileged-container-escape.html
host_path=`sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab`
# Sets uevent_helper to /path/payload
echo "$host_path/evil-helper" > /sys/kernel/uevent_helper
# Triggers a uevent
echo change > /sys/class/mem/null/uevent
# or else
# echo /sbin/poweroff > /sys/kernel/uevent_helper
# Reads the output
cat /output
```
## /sys/class/thermal

ACPI और विभिन्न हार्डवेयर सेटिंग्स तक पहुँच जो तापमान नियंत्रण के लिए होती है, आमतौर पर लैपटॉप्स या गेमिंग मदरबोर्ड्स में पाई जाती है। यह कंटेनर होस्ट के खिलाफ DoS हमलों की अनुमति दे सकता है, जिससे भौतिक क्षति भी हो सकती है।

## /sys/kernel/vmcoreinfo

यह फाइल कर्नेल पते को लीक कर सकती है जिसका उपयोग KASLR को पराजित करने के लिए किया जा सकता है।

## /sys/kernel/security

`/sys/kernel/security` में `securityfs` इंटरफेस माउंट किया गया है, जो Linux Security Modules की कॉन्फ़िगरेशन की अनुमति देता है। यह [AppArmor policies](https://gitlab.com/apparmor/apparmor/-/wikis/Kernel_interfaces#securityfs-syskernelsecurityapparmor) की कॉन्फ़िगरेशन की अनुमति देता है, और इसलिए इस तक पहुँच से कंटेनर अपनी MAC सिस्टम को अक्षम कर सकता है।

## /sys/firmware/efi/vars

`/sys/firmware/efi/vars` NVRAM में EFI वेरिएबल्स के साथ इंटरैक्ट करने के लिए इंटरफेस प्रदान करता है। जबकि यह अधिकांश सर्वरों के लिए प्रासंगिक नहीं होता है, EFI अधिक से अधिक लोकप्रिय हो रहा है। अनुमति की कमजोरियों ने कुछ लैपटॉप्स को ईंट जैसा बना दिया है।

## /sys/firmware/efi/efivars

`/sys/firmware/efi/efivars` UEFI बूट आर्ग्युमेंट्स के लिए इस्तेमाल होने वाले NVRAM में लिखने के लिए एक इंटरफेस प्रदान करता है। इन्हें संशोधित करने से होस्ट मशीन अनबूटेबल हो सकती है।

## /sys/kernel/debug

`debugfs` एक "कोई नियम नहीं" इंटरफेस प्रदान करता है जिसके द्वारा कर्नेल (या कर्नेल मॉड्यूल्स) यूजरलैंड के लिए सुलभ डिबगिंग इंटरफेस बना सकते हैं। इसमें अतीत में कई सुरक्षा मुद्दे रहे हैं, और फाइलसिस्टम के पीछे के "कोई नियम नहीं" दिशानिर्देश अक्सर सुरक्षा प्रतिबंधों के साथ टकराव में रहे हैं।

# संदर्भ

* [Understanding and Hardening Linux Containers](https://research.nccgroup.com/wp-content/uploads/2020/07/ncc_group_understanding_hardening_linux_containers-1-1.pdf)
* [Abusing Privileged and Unprivileged Linux Containers](https://www.nccgroup.com/globalassets/our-research/us/whitepapers/2016/june/container_whitepaper.pdf)


<details>

<summary><strong>Learn AWS hacking from zero to hero with</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks को अन्य तरीकों से समर्थन दें:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें।
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह।
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) पर **फॉलो** करें।
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>
