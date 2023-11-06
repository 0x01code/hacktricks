<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित करना** चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने** की पहुंच चाहिए? [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)

- प्राप्त करें [**official PEASS & HackTricks swag**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**telegram समूह**](https://t.me/peass) में या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपने हैकिंग ट्रिक्स को [hacktricks repo](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud repo](https://github.com/carlospolop/hacktricks-cloud) में PR जमा करके साझा करें।**

</details>


(_**यह जानकारी**_ [_**https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts**_](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts) **से ली गई है_)

नेमस्पेस समर्थन की कमी के कारण, `/proc` और `/sys` का प्रकटीकरण एक महत्वपूर्ण हमले का स्रोत और जानकारी विस्फोट का स्रोत प्रदान करता है। `procfs` और `sysfs` के कई फ़ाइलें कंटेनर छूटने, होस्ट संशोधन या मूलभूत जानकारी विस्फोट के लिए एक जोखिम प्रदान करती हैं, जो अन्य हमलों को सुविधाजनक बना सकती हैं।

इन तकनीकों का दुरुपयोग करने के लिए शायद बस ऐसी कोई गलत-कॉन्फ़िगर करने की आवश्यकता होगी जैसे `-v /proc:/host/proc` क्योंकि **AppArmor** `/host/proc` को सुरक्षित नहीं रखता है क्योंकि **AppArmor** पाथ पर आधारित होता है।

# procfs

## /proc/sys

`/proc/sys` आमतौर पर कर्नल चरवरीयों को संशोधित करने की अनुमति देता है, जो अक्सर `sysctl(2)` के माध्यम से नियंत्रित होते हैं।

### /proc/sys/kernel/core\_pattern

[/proc/sys/kernel/core\_pattern](https://man7.org/linux/man-pages/man5/core.5.html) एक प्रोग्राम को परिभाषित करता है जो कोर-फ़ाइल उत्पन्न होने पर (आमतौर पर कोई प्रोग्राम क्रैश) चलाया जाता है और यदि इस फ़ाइल का पहला वर्ण पाइप प्रतीक `|` है तो कोर फ़ाइल को मानक इनपुट के रूप में पास किया जाता है। यह प्रोग्राम रूट उपयोगकर्ता द्वारा चलाया जाता है और इसमें 128 बाइट तक के कमांड लाइन तर्क संभव हैं। यह किसी भी क्रैश और कोर फ़ाइल उत्पन्न होने के द्वारा कंटेनर होस्ट में सरल कोड निष्पादन की अनुमति देगा (जो कि कई खतरनाक कार्रवाईयों के दौरान सरलता से छोड़ दिया जा सकता है)।
```bash
[ -w /proc/sys/kernel/core_pattern ] && echo Yes #For testing
cd /proc/sys/kernel
echo "|$overlay/shell.sh" > core_pattern
sleep 5 && ./crash &
```
### /proc/sys/kernel/modprobe

[/proc/sys/kernel/modprobe](https://man7.org/linux/man-pages/man5/proc.5.html) में कर्नल मॉड्यूल लोडर का पथ होता है, जो कर्नल मॉड्यूल को लोड करने के लिए उपयोग होता है, जैसे [modprobe](https://man7.org/linux/man-pages/man8/modprobe.8.html) कमांड के माध्यम से. कोड क्रियाओं को निष्पादित करके कोड का निष्पादन किया जा सकता है जब कर्नल को कर्नल मॉड्यूल लोड करने का प्रयास करने की कोशिश की जाती है (जैसे कि क्रिप्टो-API का उपयोग करके एक वर्तमान मूल्यांकन नहीं करने वाले क्रिप्टो-मॉड्यूल को लोड करने के लिए, या नेटवर्किंग मॉड्यूल को लोड करने के लिए ifconfig का उपयोग करने के लिए जो वर्तमान में उपयोग नहीं हो रहा है).
```bash
# Check if you can directly access modprobe
ls -l `cat /proc/sys/kernel/modprobe`
```
### /proc/sys/vm/panic\_on\_oom

[/proc/sys/vm/panic\_on\_oom](https://man7.org/linux/man-pages/man5/proc.5.html) एक वैश्विक ध्वज है जो निर्धारित करता है कि कर्नल क्या करेगा जब एक आउट ऑफ मेमोरी (OOM) स्थिति होती है (ऑउट ऑफ मेमोरी किलर को बुलाने की बजाय पैनिक करेगा)। यह एक डिनायल ऑफ सर्विस (DoS) हमला है जो केवल होस्ट के लिए होने चाहिए।

### /proc/sys/fs

[/proc/sys/fs](https://man7.org/linux/man-pages/man5/proc.5.html) निर्देशिका विभिन्न फ़ाइल सिस्टम के विभिन्न पहलुओं, सहित कोटा, फ़ाइल हैंडल, इनोड और डेंट्री सूचना के बारे में विकल्प और जानकारी का एक एरे है। इस निर्देशिका में लिखने की अनुमति देने से, होस्ट के खिलाफ विभिन्न डिनायल-ऑफ-सर्विस हमलों की अनुमति होगी।

### /proc/sys/fs/binfmt\_misc

[/proc/sys/fs/binfmt\_misc](https://man7.org/linux/man-pages/man5/proc.5.html) विभिन्न **इंटरप्रेटर्स को गैर-मूलभूत बाइनरी** प्रारूपों के लिए पंजीकृत करने की अनुमति देता है, जो आमतौर पर उनके मैजिक नंबर पर आधारित होते हैं (जैसे जावा)। आप कर्नल को एक बाइनरी को पंजीकृत करके चला सकते हैं।
आप [https://github.com/toffan/binfmt\_misc](https://github.com/toffan/binfmt\_misc) में एक अपशिष्ट पुरुष का उपयोग कर सकते हैं: _Poor man's rootkit, leverage_ [_binfmt\_misc_](https://github.com/torvalds/linux/raw/master/Documentation/admin-guide/binfmt-misc.rst)_'s_ [_credentials_](https://github.com/torvalds/linux/blame/3bdb5971ffc6e87362787c770353eb3e54b7af30/Documentation/binfmt\_misc.txt#L62) _option to escalate privilege through any suid binary (and to get a root shell) if `/proc/sys/fs/binfmt_misc/register` is writeable._

इस तकनीक की और गहराई से समझने के लिए [https://www.youtube.com/watch?v=WBC7hhgMvQQ](https://www.youtube.com/watch?v=WBC7hhgMvQQ) देखें

## /proc/config.gz

[/proc/config.gz](https://man7.org/linux/man-pages/man5/proc.5.html) `CONFIG_IKCONFIG_PROC` सेटिंग्स पर निर्भर करता है, यह चल रहे कर्नल के कॉन्फ़िगरेशन विकल्पों के एक संकुचित संस्करण को उजागर करता है। यह एक संक्षेपित रूप में एक कंप्रोमाइज़ या दुष्ट विन्यास को आसानी से खोजने और लक्ष्यित करने की अनुमति दे सकता है जो कर्नल में सक्षम किए गए हों।

## /proc/sysrq-trigger

`Sysrq` एक पुरानी तंत्र है जिसे एक विशेष `SysRq` कुंजी संयोजन के माध्यम से आह्वानित किया जा सकता है। इससे सिस्टम को तुरंत रिबूट करने, `sync(2)` का इस्यू करने, सभी फ़ाइल सिस्टम को केवल पठने योग्य रूप में पुनर्मूल्यांकित करने, कर्नल डीबगर्स को आह्वानित करने और अन्य संचालनों की अनुमति हो सकती है।

यदि अतिथि को ठीक से अलग नहीं किया गया है, तो वह `/proc/sysrq-trigger` फ़ाइल में अक्षर लिखकर [sysrq](https://www.kernel.org/doc/html/v4.11/admin-guide/sysrq.html) कमांड को ट्रिगर कर सकता है।
```bash
# Reboot the host
echo b > /proc/sysrq-trigger
```
## /proc/kmsg

[/proc/kmsg](https://man7.org/linux/man-pages/man5/proc.5.html) कर्नल रिंग बफर संदेशों को उज्ज्वल कर सकता है जो सामान्यतः `dmesg` के माध्यम से पहुंचे जाते हैं। इस सूचना के प्रकट होने से कर्नल अधिकार उत्पन्न हो सकते हैं, कर्नल पता रिसाव (जो कर्नल पता स्थान व्यावसायिक विकास में मदद कर सकता है) को हराने में मदद कर सकते हैं, और कर्नल, हार्डवेयर, अवरुद्ध पैकेट और अन्य सिस्टम विवरणों के बारे में सामान्य सूचना विस्तार का स्रोत हो सकती है।

## /proc/kallsyms

[/proc/kallsyms](https://man7.org/linux/man-pages/man5/proc.5.html) डाइनामिक और लोडयोग्य मॉड्यूलों के लिए कर्नल निर्यातित प्रतीक और उनके पता स्थानों की सूची को संबोधित करता है। इसमें कर्नल की छवि का स्थान भी शामिल होता है जो कर्नल अधिकार विकास के लिए मददगार होता है। इन स्थानों से, कर्नल का आधार पता या ऑफसेट ढूंढा जा सकता है, जो कर्नल पता स्थान व्यावसायिक विकास (KASLR) को पार करने के लिए उपयोग किया जा सकता है।

`kptr_restrict` को `1` या `2` पर सेट करने वाले सिस्टमों के लिए, यह फ़ाइल मौजूद होगी लेकिन कोई पता सूचना प्रदान नहीं करेगी (हालांकि संकेतों की क्रमबद्धता मेमोरी में क्रमबद्धता के समान होगी)।

## /proc/\[pid]/mem

[/proc/\[pid\]/mem](https://man7.org/linux/man-pages/man5/proc.5.html) कर्नल मेमोरी उपकरण `/dev/mem` के अंतर्गत इंटरफेस प्रकट करता है। यद्यपि PID नेमस्पेस कुछ हमलों से सुरक्षित कर सकता है जैसे कि यह `procfs` वेक्टर के माध्यम से, यह क्षेत्र ऐतिहासिक रूप से संकटग्रस्त रहा है, फिर सुरक्षित माना गया और फिर से [संकटग्रस्त](https://git.zx2c4.com/CVE-2012-0056/about/) प्राधिकरण के लिए।

## /proc/kcore

[/proc/kcore](https://man7.org/linux/man-pages/man5/proc.5.html) सिस्टम की भौतिक मेमोरी को प्रतिष्ठान देता है और एक ELF कोर प्रारूप में होता है (सामान्यतः कोर डंप फ़ाइलों में पाया जाता है)। इसमें उस मेमोरी को लिखने की अनुमति नहीं होती है। इस फ़ाइल को पढ़ने की क्षमता (विशेषाधिकारी उपयोगकर्ताओं के लिए सीमित होती है) मेमोरी सामग्री को मेमोरी से लीक कर सकती है जो होस्ट सिस्टम और अन्य कंटेनरों से हो सकती है।

बड़ी रिपोर्ट की गई फ़ाइल आकार वास्तविकता में व्यावसायिक यादृच्छिकता के आधार पर इसे पढ़ने (या सॉफ़्टवेयर की कमजोरी के आधार पर क्रैश) कर सकती है।

[2019 में /proc/kcore को डंप करना](https://schlafwandler.github.io/posts/dumping-/proc/kcore/)

## /proc/kmem

`/proc/kmem` [/dev/kmem](https://man7.org/linux/man-pages/man4/kmem.4.html) के लिए एक वैकल्पिक इंटरफेस है (जिसका सीग्रुप उपकरण सफेद सूची द्वारा अवरुद्ध होता है), जो कर्नल वर्चुअल मेमोरी को प्रतिष्ठान करने वाली एक वर्ण उपकरण फ़ाइल है। यह पढ़ने और लिखने दोनों की अनुमति देता है, जिससे कर्नल मेमोरी को सीधे संशोधित किया जा सकता है।

## /proc/mem

`/proc/mem` [/dev/mem](https://man7.org/linux/man-pages/man4/kmem.4.html) के लिए एक वैकल्पिक इंटरफेस है (जिसका सीग्रुप उपकरण सफेद सूची द्वारा अवरुद्ध होता है), जो सिस्टम की भौतिक मेमोरी को प्रतिष्ठान करने वाली एक वर्ण उपकरण फ़ाइल है। यह पढ़ने और लिखने दोनों की अनुमति देता है, जिससे सभी मेमोरी को संशोधित किया जा सकता है। (यह `kmem` से थोड़ी अधिक कुशलता की आवश्यकता होती है, क्योंकि वर्चुअल पते को पहले भौतिक पतों में रिज़ॉल्व किया जाना चाहिए)।

## /proc/sched\_debug

`/proc/sched_debug` एक विशेष फ़ाइल है जो पूरे सिस्टम के लिए प्रक्रिया अनुसूचना जानकारी वापस करती है। इस सूचना में प्रक्रिया नाम और प्रक्रिया ID सभी नेमस्पेस से और प्रक्रिया सीग्रुप पहचानकर्ताओं सहित शामिल होते हैं। यह प्रभावी रूप से PID नेमस्पेस संरक्षण को दूर करता है और अनुप्रयोगित कंटेनरों में भी उपयोग किया जा सकता है।

## /proc/\[pid]/mountinfo

[/proc/\[pid\]/mountinfo](https://man7.org/linux/man-pages/man5/proc.5.html) प्रक्रिया के माउंट नेमस्पेस में माउंट प्वाइंट के बारे में जानकारी प्रदान करता है। यह कंटेनर `rootfs` या इमेज का स्थान प्रकट करता है।

# sysfs

## /sys/kernel/uevent\_helper

`uevents` कर्नल द्वारा एक उपकरण को जोड़ा या हटाए जाने पर उत्पन्न होने वाले घटनाओं हैं। विशेष रूप से, `uevent_helper` के लिए पथ `/sys/kernel/uevent_helper` लिखकर सं
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

लैपटॉप या गेमिंग मदरबोर्ड में मिलने वाले एसीपीआई और विभिन्न हार्डवेयर सेटिंग्स का उपयोग तापमान नियंत्रण के लिए किया जाता है। यह कंटेनर होस्ट के खिलाफ डीओएस हमलों की अनुमति देता है, जो शारीरिक क्षति तक भी पहुंच सकता है।

## /sys/kernel/vmcoreinfo

यह फ़ाइल कर्नल पतों को लीक कर सकती है जो KASLR को परास्त करने के लिए उपयोग किए जा सकते हैं।

## /sys/kernel/security

`/sys/kernel/security` में `securityfs` इंटरफ़ेस माउंट होता है, जो लिनक्स सुरक्षा मॉड्यूल के कॉन्फ़िगरेशन की अनुमति देता है। इसके माध्यम से [AppArmor नीतियों](https://gitlab.com/apparmor/apparmor/-/wikis/Kernel\_interfaces#securityfs-syskernelsecurityapparmor) को कॉन्फ़िगर किया जा सकता है, और इसलिए इसका उपयोग करके कंटेनर अपनी MAC सिस्टम को अक्षम कर सकता है।

## /sys/firmware/efi/vars

`/sys/firmware/efi/vars` NVRAM में EFI चरणों के साथ इंटरैक्ट करने के लिए इंटरफ़ेस प्रदान करता है। यह आमतौर पर अधिकांश सर्वरों के लिए महत्वपूर्ण नहीं होता है, लेकिन EFI बढ़ती हुई प्रसिद्ध हो रही है। अनुमति कमजोरियों के कारण कुछ ब्रिक्ड लैपटॉप्स भी हुए हैं।

## /sys/firmware/efi/efivars

`/sys/firmware/efi/efivars` UEFI बूट तर्कों के लिए उपयोग होने वाले NVRAM में लिखने के लिए एक इंटरफ़ेस प्रदान करता है। इन्हें संशोधित करने से होस्ट मशीन बूट नहीं हो सकती है।

## /sys/kernel/debug

`debugfs` कर्नल (या कर्नल मॉड्यूल) द्वारा एक "कोई नियम नहीं" इंटरफ़ेस प्रदान करता है, जिसके माध्यम से यूजरलैंड तक पहुंचने योग्य डीबग इंटरफ़ेस बनाए जा सकते हैं। इसने पिछले में कई सुरक्षा समस्याओं का सामना किया है, और फ़ाइलसिस्टम के पीछे "कोई नियम" दिशानिर्देशों ने अक्सर सुरक्षा प्रतिबंधों के साथ टकराव किया है।

# संदर्भ

* [लिनक्स कंटेनर को समझें और हार्डन करें](https://research.nccgroup.com/wp-content/uploads/2020/07/ncc\_group\_understanding\_hardening\_linux\_containers-1-1.pdf)
* [विशेषाधिकार और अनिश्चित लिनक्स कंटेनर](https://www.nccgroup.com/globalassets/our-research/us/whitepapers/2016/june/container\_whitepaper.pdf)


<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **हैकट्रिक्स** में विज्ञापित करना चाहते हैं? या क्या आप **PEASS के नवीनतम संस्करण का उपयोग करना चाहते हैं या HackTricks को पीडीएफ़ में डाउनलोड करना चाहते हैं**? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**द पीएस फैमिली**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**एनएफटी**](https://opensea.io/collection/the-peass-family) संग्रह

- प्राप्त करें [**आधिकारिक पीएस और हैकट्रिक्स स्वैग**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)** का** अनुसरण करें।**

- **अपने हैकिंग ट्रिक्स साझा करें, हैकट्रिक्स रेपो और हैकट्रिक्स-क्लाउड रेपो में पीआर जमा करके**।

</details>
