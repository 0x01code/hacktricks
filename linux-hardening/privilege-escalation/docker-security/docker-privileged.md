# Docker --privileged

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**official PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **Join the** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord group**](https://discord.gg/hRep4RUj7f) या [**telegram group**](https://t.me/peass) में शामिल हों या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स को [hacktricks repo](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud repo](https://github.com/carlospolop/hacktricks-cloud) में PR जमा करके साझा करें।**

</details>

## क्या प्रभावित होता है

जब आप एक privileged container को चलाते हैं, तो आप निम्नलिखित सुरक्षा उपायों को अक्षम कर रहे हैं:

### Mount /dev

एक privileged container में, सभी **उपकरण `/dev/` में पहुंच सकते हैं**। इसलिए आप **माउंट करके** होस्ट के डिस्क से **बाहर निकल** सकते हैं।

{% tabs %}
{% tab title="डिफ़ॉल्ट container के अंदर" %}
```bash
# docker run --rm -it alpine sh
ls /dev
console  fd       mqueue   ptmx     random   stderr   stdout   urandom
core     full     null     pts      shm      stdin    tty      zero
```
{% endtab %}

{% tab title="विशेषाधिकारित कंटेनर के अंदर" %}
```bash
# docker run --rm --privileged -it alpine sh
ls /dev
cachefiles       mapper           port             shm              tty24            tty44            tty7
console          mem              psaux            stderr           tty25            tty45            tty8
core             mqueue           ptmx             stdin            tty26            tty46            tty9
cpu              nbd0             pts              stdout           tty27            tty47            ttyS0
[...]
```
{% endtab %}
{% endtabs %}

### कर्नल फ़ाइल सिस्टम को सिर्फ़ पढ़ने योग्य बनाएं

कर्नल फ़ाइल सिस्टम एक तरीका प्रदान करते हैं जिसके द्वारा **प्रक्रिया कर्नल के चलने के तरीके को बदल सकती है।** डिफ़ॉल्ट रूप से, हम **चाहते हैं कि कंटेनर प्रक्रियाएँ कर्नल को संशोधित न करें**, इसलिए हम कंटेनर के भीतर कर्नल फ़ाइल सिस्टम को केवल पढ़ने योग्य रूप में माउंट करते हैं।

{% tabs %}
{% tab title="डिफ़ॉल्ट कंटेनर के भीतर" %}
```bash
# docker run --rm -it alpine sh
mount | grep '(ro'
sysfs on /sys type sysfs (ro,nosuid,nodev,noexec,relatime)
cpuset on /sys/fs/cgroup/cpuset type cgroup (ro,nosuid,nodev,noexec,relatime,cpuset)
cpu on /sys/fs/cgroup/cpu type cgroup (ro,nosuid,nodev,noexec,relatime,cpu)
cpuacct on /sys/fs/cgroup/cpuacct type cgroup (ro,nosuid,nodev,noexec,relatime,cpuacct)
```
{% endtab %}

{% tab title="विशेषाधिकारित कंटेनर के अंदर" %}
```bash
# docker run --rm --privileged -it alpine sh
mount  | grep '(ro'
```
{% endtab %}
{% endtabs %}

### कर्नल फ़ाइल सिस्टम को मास्क करना

**/proc** फ़ाइल सिस्टम नेमस्पेस-अवगुणी है, और कुछ लिखने की अनुमति दी जा सकती है, इसलिए हम इसे रीड-ओनली माउंट नहीं करते हैं। हालांकि, /proc फ़ाइल सिस्टम के विशेष निर्देशिकाओं को **लिखने से सुरक्षित रखना** चाहिए, और कुछ स्थितियों में, **पढ़ने से भी सुरक्षित रखना** चाहिए। इन मामलों में, कंटेनर इंजन खतरनाक निर्देशिकाओं पर **tmpfs** फ़ाइल सिस्टम को माउंट करते हैं, जिससे कंटेनर के अंदर के प्रक्रियाएँ उन्हें उपयोग नहीं कर सकती हैं।

{% hint style="info" %}
**tmpfs** एक फ़ाइल सिस्टम है जो सभी फ़ाइलों को वर्चुअल मेमोरी में संग्रहित करता है। tmpfs आपके हार्ड ड्राइव पर कोई फ़ाइल नहीं बनाता है। इसलिए अगर आप एक tmpfs फ़ाइल सिस्टम को अनमाउंट करते हैं, तो उसमें रहने वाली सभी फ़ाइलें हमेशा के लिए खो जाती हैं।
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

{% tab title="विशेषाधिकारित कंटेनर के अंदर" %}
```bash
# docker run --rm --privileged -it alpine sh
mount  | grep /proc.*tmpfs
```
{% endtab %}
{% endtabs %}

### लिनक्स क्षमताएं

कंटेनर इंजन डिफ़ॉल्ट रूप से कंटेनर के अंदर क्या हो रहा है को नियंत्रित करने के लिए **सीमित संख्या की क्षमताएं** के साथ कंटेनर को लॉन्च करते हैं। **विशेषाधिकृत** क्षमताएं सभी **क्षमताओं** को उपयोग कर सकती हैं। क्षमताओं के बारे में जानने के लिए पढ़ें:

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

{% tab title="विशेषाधिकारित कंटेनर के अंदर" %}
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

आप `--privileged` मोड में नहीं चला कर `--cap-add` और `--cap-drop` फ्लैग का उपयोग करके कंटेनर के लिए उपलब्ध क्षमताओं को मानिपुरेट कर सकते हैं।

### सेकॉम्प

**सेकॉम्प** कंटेनर द्वारा कॉल किए जा सकने वाले **सिसकॉल** को सीमित करने के लिए उपयोगी है। डॉकर कंटेनर चलाते समय एक डिफ़ॉल्ट सेकॉम्प प्रोफ़ाइल सक्षम होता है, लेकिन विशेषाधिकार मोड में यह अक्षम हो जाता है। सेकॉम्प के बारे में अधिक जानें:

{% content-ref url="seccomp.md" %}
[seccomp.md](seccomp.md)
{% endcontent-ref %}

{% tabs %}
{% tab title="डिफ़ॉल्ट कंटेनर के अंदर" %}
```bash
# docker run --rm -it alpine sh
grep Seccomp /proc/1/status
Seccomp:	2
Seccomp_filters:	1
```
{% endtab %}

{% tab title="विशेषाधिकारित कंटेनर के अंदर" %}
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
ध्यान दें कि जब **Docker** (या अन्य CRIs) को **Kubernetes** क्लस्टर में उपयोग किया जाता है, तो **seccomp फ़िल्टर डिफ़ॉल्ट रूप से अक्षम हो जाता है**।

### AppArmor

**AppArmor** एक कर्नल एन्हांसमेंट है जो **कंटेनर** को **सीमित** संसाधनों के साथ **प्रति-प्रोग्राम प्रोफ़ाइल** में बाधित करने के लिए होता है। जब आप `--privileged` फ़्लैग के साथ चलाते हैं, तो यह सुरक्षा अक्षम हो जाती है।

{% content-ref url="apparmor.md" %}
[apparmor.md](apparmor.md)
{% endcontent-ref %}
```bash
# You can manually disable seccomp in docker with
--security-opt apparmor=unconfined
```
### SELinux

जब आप `--privileged` फ्लैग के साथ चलाते हैं, तो **SELinux लेबल अक्षम हो जाते हैं**, और कंटेनर **उस लेबल के साथ चलता है जिसके साथ कंटेनर इंजन ने चलाया था**। यह लेबल आमतौर पर `unconfined` होता है और **कंटेनर इंजन के लेबल के पूर्ण उपयोग की अनुमति होती है**। रूटलेस मोड में, कंटेनर `container_runtime_t` के साथ चलता है। रूट मोड में, यह `spc_t` के साथ चलता है।

{% content-ref url="../selinux.md" %}
[selinux.md](../selinux.md)
{% endcontent-ref %}
```bash
# You can manually disable selinux in docker with
--security-opt label:disable
```
## क्या प्रभावित नहीं होता है

### नेमस्पेस

नेमस्पेस `--privileged` फ़्लैग से **प्रभावित नहीं होते हैं**। हालांकि, ये सुरक्षा प्रतिबंध सक्षम नहीं होते हैं, उदाहरण के लिए वे सिस्टम पर सभी प्रक्रियाओं या होस्ट नेटवर्क को नहीं देखते हैं। उपयोगकर्ता नेमस्पेस को अक्षम कर सकते हैं इस्तेमाल करके **`--pid=host`, `--net=host`, `--ipc=host`, `--uts=host`** कंटेनर इंजन फ़्लैग्स का उपयोग करके।

{% tabs %}
{% tab title="डिफ़ॉल्ट विशेषाधिकृत कंटेनर के अंदर" %}
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
{% endtab %}
{% endtabs %}

### उपयोगकर्ता नेमस्पेस

कंटेनर इंजन डिफ़ॉल्ट रूप से उपयोगकर्ता नेमस्पेस का उपयोग नहीं करते हैं। हालांकि, रूटलेस कंटेनर हमेशा इसका उपयोग करते हैं ताकि वे फ़ाइल सिस्टम को माउंट कर सकें और एक से अधिक UID का उपयोग कर सकें। रूटलेस मामले में, उपयोगकर्ता नेमस्पेस को अक्षम नहीं किया जा सकता है; रूटलेस कंटेनर चलाने के लिए इसकी आवश्यकता होती है। उपयोगकर्ता नेमस्पेस कुछ विशेषाधिकारों को रोकता है और महत्वपूर्ण सुरक्षा जोड़ता है।

## संदर्भ

* [https://www.redhat.com/sysadmin/privileged-flag-container-engines](https://www.redhat.com/sysadmin/privileged-flag-container-engines)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने की अनुमति** चाहिए? [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* प्राप्त करें [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)** का** अनुसरण करें।
* **अपने हैकिंग ट्रिक्स साझा करें, [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके।**

</details>
