# Docker --privileged

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी का हैकट्रिक्स में विज्ञापित करना चाहते हैं**? या क्या आप **PEASS के नवीनतम संस्करण देखना चाहते हैं या हैकट्रिक्स को पीडीएफ में डाउनलोड करना चाहते हैं**? [**सब्सक्रिप्शन प्लान**](https://github.com/sponsors/carlospolop) की जाँच करें!
* [**दी पीएस फैमिली**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**एनएफटी**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक पीएस और हैकट्रिक्स स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **ट्विटर** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)** पर** **फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, [हैकट्रिक्स रेपो](https://github.com/carlospolop/hacktricks) और [हैकट्रिक्स-क्लाउड रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके**।

</details>

## क्या प्रभावित होता है

जब आप एक कंटेनर को --privileged के रूप में चलाते हैं, तो आप निम्नलिखित सुरक्षा सुरक्षा को अक्षम कर देते हैं:

### /dev माउंट करें

एक --privileged कंटेनर में, सभी **डिवाइस /dev/ में पहुंच सकते हैं**। इसलिए आप **माउंट करके** होस्ट के डिस्क से **बाहर निकल** सकते हैं।

{% tabs %}
{% tab title="डिफ़ॉल्ट कंटेनर के अंदर" %}
```bash
# docker run --rm -it alpine sh
ls /dev
console  fd       mqueue   ptmx     random   stderr   stdout   urandom
core     full     null     pts      shm      stdin    tty      zero
```
{% endtab %}

{% tab title="प्रिविलेज्ड कंटेनर के अंदर" %}
```bash
# docker run --rm --privileged -it alpine sh
ls /dev
cachefiles       mapper           port             shm              tty24            tty44            tty7
console          mem              psaux            stderr           tty25            tty45            tty8
core             mqueue           ptmx             stdin            tty26            tty46            tty9
cpu              nbd0             pts              stdout           tty27            tty47            ttyS0
[...]
```
### केवल पढ़ने योग्य कर्नेल फ़ाइल सिस्टम

कर्नेल फ़ाइल सिस्टम प्रक्रिया के लिए कर्नेल के व्यवहार को संशोधित करने के लिए एक तंत्र प्रदान करते हैं। हालांकि, जब यह कंटेनर प्रक्रियाएँ होती हैं, तो हम चाहते हैं कि उन्हें कर्नेल में कोई भी परिवर्तन न करने दिया जाए। इसलिए, हम कर्नेल फ़ाइल सिस्टम को कंटेनर के भीतर **केवल पढ़ने योग्य** रूप में माउंट करते हैं, यह सुनिश्चित करते हैं कि कंटेनर प्रक्रियाएँ कर्नेल को संशोधित नहीं कर सकतीं।
```bash
# docker run --rm -it alpine sh
mount | grep '(ro'
sysfs on /sys type sysfs (ro,nosuid,nodev,noexec,relatime)
cpuset on /sys/fs/cgroup/cpuset type cgroup (ro,nosuid,nodev,noexec,relatime,cpuset)
cpu on /sys/fs/cgroup/cpu type cgroup (ro,nosuid,nodev,noexec,relatime,cpu)
cpuacct on /sys/fs/cgroup/cpuacct type cgroup (ro,nosuid,nodev,noexec,relatime,cpuacct)
```
{% endtab %}

{% tab title="प्रिविलेज्ड कंटेनर के अंदर" %}
```bash
# docker run --rm --privileged -it alpine sh
mount  | grep '(ro'
```
### कर्नेल फ़ाइल सिस्टम को मास्क करना

**/proc** फ़ाइल सिस्टम को चयनित रूप से लिखने योग्य है, लेकिन सुरक्षा के लिए, कुछ हिस्से लिखने और पढ़ने की पहुंच को रोकने के लिए उन्हें **tmpfs** से ढक दिया गया है, यह सुनिश्चित करता है कि कंटेनर प्रक्रियाएँ संवेदनशील क्षेत्रों तक पहुंच नहीं पा सकतीं।

{% hint style="info" %}
**tmpfs** एक फ़ाइल सिस्टम है जो सभी फ़ाइलों को वर्चुअल मेमोरी में स्टोर करता है। tmpfs आपके हार्ड ड्राइव पर कोई फ़ाइल नहीं बनाता है। इसलिए अगर आप एक tmpfs फ़ाइल सिस्टम को अनमाउंट करते हैं, तो उसमें रहने वाली सभी फ़ाइलें हमेशा के लिए खो जाती हैं।
{% endhint %}

{% tabs %}
{% tab title="डिफ़ॉल्ट कंटेनर के अंदर" %}
```bash
# docker run --rm -it alpine sh
mount  | grep /proc.*tmpfs
tmpfs on /proc/acpi type tmpfs (ro,relatime)
tmpfs on /proc/kcore type tmpfs (rw,nosuid,size=65536k,mode=755)
tmpfs on /proc/keys type tmpfs (rw,nosuid,size=65536k,mode=755)
```
{% endtab %}

{% tab title="प्रिविलेज्ड कंटेनर के अंदर" %}
```bash
# docker run --rm --privileged -it alpine sh
mount  | grep /proc.*tmpfs
```
### लिनक्स क्षमताएँ

कंटेनर इंजन डिफ़ॉल्ट रूप से कंटेनर के अंदर क्या हो रहा है को नियंत्रित करने के लिए **कुछ सीमित संख्या** की **क्षमताएँ** के साथ कंटेनर को लॉन्च करते हैं। **विशेषाधिकृत** वाले में **सभी** **क्षमताएँ** एक्सेस करने में सक्षम होती हैं। क्षमताओं के बारे में जानने के लिए पढ़ें:

{% content-ref url="../linux-capabilities.md" %}
[linux-capabilities.md](../linux-capabilities.md)
{% endcontent-ref %}

{% tabs %}
{% tab title="डिफ़ॉल्ट कंटेनर के अंदर" %}
```bash
# docker run --rm -it alpine sh
apk add -U libcap; capsh --print
[...]
Current: cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap=eip
Bounding set =cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap
[...]
```
{% endtab %}

{% tab title="प्रबलीकृत कंटेनर के अंदर" %}
```bash
# docker run --rm --privileged -it alpine sh
apk add -U libcap; capsh --print
[...]
Current: =eip cap_perfmon,cap_bpf,cap_checkpoint_restore-eip
Bounding set =cap_chown,cap_dac_override,cap_dac_read_search,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_linux_immutable,cap_net_bind_service,cap_net_broadcast,cap_net_admin,cap_net_raw,cap_ipc_lock,cap_ipc_owner,cap_sys_module,cap_sys_rawio,cap_sys_chroot,cap_sys_ptrace,cap_sys_pacct,cap_sys_admin,cap_sys_boot,cap_sys_nice,cap_sys_resource,cap_sys_time,cap_sys_tty_config,cap_mknod,cap_lease,cap_audit_write,cap_audit_control,cap_setfcap,cap_mac_override,cap_mac_admin,cap_syslog,cap_wake_alarm,cap_block_suspend,cap_audit_read
[...]
```
{% endtab %}
{% endtabs %}

आप कंटेनर में `--privileged` मोड में नहीं चलाते हुए `--cap-add` और `--cap-drop` फ्लैग का उपयोग करके कंटेनर के लिए उपलब्ध क्षमताओं को मानवीय कर सकते हैं।

### Seccomp

**Seccomp** कंटेनर द्वारा कॉल किए जा सकने वाले **syscalls** को **सीमित** करने के लिए उपयोगी है। डॉकर कंटेनर चलाने पर एक डिफ़ॉल्ट सेकॉम्प प्रोफ़ाइल सक्षम होता है, लेकिन विशेषाधिकारित मोड में यह अक्षम हो जाता है। Seccomp के बारे में अधिक जानकारी यहाँ देखें:

{% content-ref url="seccomp.md" %}
[seccomp.md](seccomp.md)
{% endcontent-ref %}
```bash
# docker run --rm -it alpine sh
grep Seccomp /proc/1/status
Seccomp:	2
Seccomp_filters:	1
```
{% endtab %}

{% tab title="प्रिविलेज्ड कंटेनर के अंदर" %}
```bash
# docker run --rm --privileged -it alpine sh
grep Seccomp /proc/1/status
Seccomp:	0
Seccomp_filters:	0
```
{% endtab %}
{% endtabs %}
```bash
# You can manually disable seccomp in docker with
--security-opt seccomp=unconfined
```
ध्यान दें कि जब **Kubernetes** क्लस्टर में Docker (या अन्य CRIs) का उपयोग किया जाता है, तो **seccomp फ़िल्टर** डिफ़ॉल्ट रूप से अक्षम हो जाता है।

### AppArmor

**AppArmor** एक कर्नेल एन्हांसमेंट है जो **कंटेनर** को **सीमित** संसाधनों के साथ **प्रोग्राम प्रोफ़ाइल** के लिए सीमित करने के लिए है। जब आप `--privileged` फ़्लैग के साथ चलाते हैं, तो यह सुरक्षा अक्षम हो जाती है।

{% content-ref url="apparmor.md" %}
[apparmor.md](apparmor.md)
{% endcontent-ref %}
```bash
# You can manually disable seccomp in docker with
--security-opt apparmor=unconfined
```
### SELinux

कंटेनर को `--privileged` फ्लैग के साथ चलाने से **SELinux लेबल** निष्क्रिय हो जाते हैं, जिससे यह कंटेनर इंजन का लेबल विरासत में प्राप्त करता है, सामान्यत: `unconfined`, पूर्ण पहुंच प्रदान करते हुए। रूटलेस मोड में, यह `container_runtime_t` का उपयोग करता है, जबकि रूट मोड में, `spc_t` लागू होता है।

{% content-ref url="../selinux.md" %}
[selinux.md](../selinux.md)
{% endcontent-ref %}
```bash
# You can manually disable selinux in docker with
--security-opt label:disable
```
## क्या प्रभावित नहीं होता

### नेमस्पेस

नेमस्पेस `--privileged` फ्लैग द्वारा प्रभावित **नहीं होते** हैं। यहां तक कि जब उन्हें सुरक्षा प्रतिबंध सक्षम नहीं किया गया है, तो वे **सिस्टम पर सभी प्रक्रियाएँ या मुख्य नेटवर्क को नहीं देखते**। उपयोगकर्ता व्यक्तिगत नेमस्पेस को निष्क्रिय कर सकते हैं जैसे कि **`--pid=host`, `--net=host`, `--ipc=host`, `--uts=host`** कंटेनर इंजन फ्लैग का उपयोग करके।

{% tabs %}
{% tab title="डिफ़ॉल्ट प्रबंधित कंटेनर के अंदर" %}
```bash
# docker run --rm --privileged -it alpine sh
ps -ef
PID   USER     TIME  COMMAND
1 root      0:00 sh
18 root      0:00 ps -ef
```
{% endtab %}

{% tab title="अंदर --pid=host कंटेनर" %}
```bash
# docker run --rm --privileged --pid=host -it alpine sh
ps -ef
PID   USER     TIME  COMMAND
1 root      0:03 /sbin/init
2 root      0:00 [kthreadd]
3 root      0:00 [rcu_gp]ount | grep /proc.*tmpfs
[...]
```
### उपयोगकर्ता नेमस्पेस

**डिफ़ॉल्ट रूप से, कंटेनर इंजन यूजर नेमस्पेस का उपयोग नहीं करते हैं, केवल रूटलेस कंटेनर के लिए**, जो फ़ाइल सिस्टम माउंटिंग और एक से अधिक यूआईडी का उपयोग करने के लिए इसकी आवश्यकता होती है। यूजर नेमस्पेस, जो रूटलेस कंटेनर के लिए अनिवार्य हैं, को अक्षम नहीं किया जा सकता और विशेष रूप से सुरक्षा को बढ़ावा देते हैं द्वारा विशेषाधिकारों को प्रतिबंधित करके।

## संदर्भ

* [https://www.redhat.com/sysadmin/privileged-flag-container-engines](https://www.redhat.com/sysadmin/privileged-flag-container-engines)
