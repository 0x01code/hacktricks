# macOS कर्नेल और सिस्टम एक्सटेंशन्स

<details>

<summary><strong>AWS हैकिंग सीखें शुरुआत से लेकर एक्सपर्ट तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) में या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) **का अनुसरण करें**.
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें.

</details>

## XNU कर्नेल

**macOS का मूल XNU है**, जिसका अर्थ है "X is Not Unix". यह कर्नेल मूल रूप से **Mach माइक्रोकर्नेल** (बाद में चर्चा की जाएगी) और Berkeley Software Distribution (**BSD**) के तत्वों से बना है। XNU **कर्नेल ड्राइवर्स के लिए एक प्लेटफॉर्म भी प्रदान करता है जिसे I/O Kit कहा जाता है**। XNU कर्नेल Darwin ओपन सोर्स प्रोजेक्ट का हिस्सा है, जिसका मतलब है **इसका सोर्स कोड स्वतंत्र रूप से सुलभ है**।

सुरक्षा शोधकर्ता या Unix डेवलपर के दृष्टिकोण से, **macOS** काफी **समान** महसूस हो सकता है **FreeBSD** सिस्टम के साथ एक सुंदर GUI और कस्टम एप्लिकेशन्स के साथ। BSD के लिए विकसित अधिकांश एप्लिकेशन्स macOS पर बिना किसी संशोधन के कंपाइल और चलेंगे, क्योंकि Unix उपयोगकर्ताओं के लिए परिचित कमांड-लाइन टूल्स macOS में सभी मौजूद हैं। हालांकि, चूंकि XNU कर्नेल में Mach शामिल है, इसलिए पारंपरिक Unix-जैसे सिस्टम और macOS के बीच कुछ महत्वपूर्ण अंतर हैं, और ये अंतर संभावित समस्याएं पैदा कर सकते हैं या अनूठे लाभ प्रदान कर सकते हैं।

XNU का ओपन सोर्स संस्करण: [https://opensource.apple.com/source/xnu/](https://opensource.apple.com/source/xnu/)

### Mach

Mach एक **माइक्रोकर्नेल** है जिसे **UNIX-संगत** होने के लिए डिजाइन किया गया है। इसके मुख्य डिजाइन सिद्धांतों में से एक था **कर्नेल** स्थान में चलने वाले **कोड** की मात्रा को **न्यूनतम** करना और इसके बजाय फाइल सिस्टम, नेटवर्किंग, और I/O जैसे कई पारंपरिक कर्नेल कार्यों को **यूजर-लेवल टास्क्स के रूप में चलाने की अनुमति देना**।

XNU में, Mach **कई महत्वपूर्ण निम्न-स्तरीय संचालनों के लिए जिम्मेदार है** जो आमतौर पर एक कर्नेल संभालता है, जैसे कि प्रोसेसर शेड्यूलिंग, मल्टीटास्किंग, और वर्चुअल मेमोरी प्रबंधन।

### BSD

XNU **कर्नेल** भी **FreeBSD** प्रोजेक्ट से प्राप्त कोड की एक महत्वपूर्ण मात्रा को **शामिल करता है**। यह कोड **Mach के साथ कर्नेल के भाग के रूप में चलता है**, एक ही एड्रेस स्पेस में। हालांकि, XNU में FreeBSD कोड मूल FreeBSD कोड से काफी अलग हो सकता है क्योंकि Mach के साथ इसकी संगतता सुनिश्चित करने के लिए संशोधन आवश्यक थे। FreeBSD कई कर्नेल संचालनों में योगदान देता है जिसमें शामिल हैं:

* प्रोसेस प्रबंधन
* सिग्नल हैंडलिंग
* मूल सुरक्षा तंत्र, जिसमें यूजर और ग्रुप प्रबंधन शामिल है
* सिस्टम कॉल इंफ्रास्ट्रक्चर
* TCP/IP स्टैक और सॉकेट्स
* फ़ायरवॉल और पैकेट फ़िल्टरिंग

BSD और Mach के बीच इंटरैक्शन को समझना जटिल हो सकता है, उनके अलग-अलग संकल्पनात्मक ढांचे के कारण। उदाहरण के लिए, BSD प्रक्रियाओं का उपयोग अपनी मौलिक क्रियान्वयन इकाई के रूप में करता है, जबकि Mach धागों के आधार पर काम करता है। XNU में इस विसंगति को **प्रत्येक BSD प्रोसेस को एक Mach कार्य से जोड़कर** सुलझाया जाता है जिसमें ठीक एक Mach धागा होता है। जब BSD का fork() सिस्टम कॉल का उपयोग किया जाता है, तो कर्नेल के भीतर BSD कोड Mach कार्यों का उपयोग करके एक कार्य और एक धागा संरचना बनाता है।

इसके अलावा, **Mach और BSD प्रत्येक अलग सुरक्षा मॉडल बनाए रखते हैं**: **Mach** का सुरक्षा मॉडल **पोर्ट अधिकारों** पर आधारित है, जबकि BSD का सुरक्षा मॉडल **प्रोसेस स्वामित्व** पर काम करता है। इन दो मॉडलों के बीच अंतर कभी-कभी स्थानीय विशेषाधिकार-वृद्धि संवेदनशीलताओं में परिणाम करते हैं। सामान्य सिस्टम कॉल्स के अलावा, वहाँ भी **Mach जाल हैं जो यूजर-स्पेस प्रोग्राम्स को कर्नेल के साथ इंटरैक्ट करने की अनुमति देते हैं**। ये विभिन्न तत्व मिलकर macOS कर्नेल की बहुमुखी, हाइब्रिड वास्तुकला का निर्माण करते हैं।

### I/O Kit - ड्राइवर्स

I/O Kit XNU कर्नेल में ओपन-सोर्स, ऑब्जेक्ट-ओरिएंटेड, **डिवाइस-ड्राइवर फ्रेमवर्क** है और यह **गतिशील रूप से लोड किए गए डिवाइस ड्राइवर्स** के जोड़ने और प्रबंधन के लिए जिम्मेदार है। ये ड्राइवर्स विभिन्न हार्डवेयर के साथ उपयोग के लिए कर्नेल में गतिशील रूप से मॉड्यूलर कोड जोड़ने की अनुमति देते हैं, उदाहरण के लिए।

{% content-ref url="macos-iokit.md" %}
[macos-iokit.md](macos-iokit.md)
{% endcontent-ref %}

### IPC - इंटर प्रोसेस कम्युनिकेशन

{% content-ref url="macos-ipc-inter-process-communication/" %}
[macos-ipc-inter-process-communication](macos-ipc-inter-process-communication/)
{% endcontent-ref %}

### कर्नेलकैश

**कर्नेलकैश** XNU कर्नेल का एक **पूर्व-संकलित और पूर्व-लिंक किया गया संस्करण** है, जिसमें आवश्यक डिवाइस **ड्राइवर्स** और **कर्नेल एक्सटेंशन्स** शामिल हैं। यह एक **संपीड़ित** प्रारूप में संग्रहीत होता है और बूट-अप प्रक्रिया के दौरान मेमोरी में डिकंप्रेस हो जाता है। कर्नेलकैश बूट समय को **तेजी से बनाने में मदद करता है** क्योंकि इसमें कर्नेल और महत्वपूर्ण ड्राइवर्स का तैयार
```bash
# pyimg4 (https://github.com/m1stadev/PyIMG4)
pyimg4 im4p extract -i kernelcache.release.iphone14 -o kernelcache.release.iphone14.e

# img4tool (https://github.com/tihmstar/img4tool
img4tool -e kernelcache.release.iphone14 -o kernelcache.release.iphone14.e
```
#### कर्नेलकैश सिंबल्स

कभी-कभी Apple **कर्नेलकैश** को **सिंबल्स** के साथ जारी करता है। आप [https://theapplewiki.com](https://theapplewiki.com/) पर दिए गए लिंक्स का अनुसरण करके कुछ फर्मवेयर्स को सिंबल्स के साथ डाउनलोड कर सकते हैं।

### IPSW

ये Apple के **फर्मवेयर्स** हैं जिन्हें आप [**https://ipsw.me/**](https://ipsw.me/) से डाउनलोड कर सकते हैं। अन्य फाइलों के बीच इसमें **कर्नेलकैश** भी शामिल होगा।\
फाइलों को **निकालने** के लिए आप बस इसे **अनजिप** कर सकते हैं।

फर्मवेयर निकालने के बाद आपको ऐसी फाइल मिलेगी: **`kernelcache.release.iphone14`**। यह **IMG4** फॉर्मेट में है, आप इससे दिलचस्प जानकारी निकाल सकते हैं:

* [**pyimg4**](https://github.com/m1stadev/PyIMG4)

{% code overflow="wrap" %}
```bash
pyimg4 im4p extract -i kernelcache.release.iphone14 -o kernelcache.release.iphone14.e
```
{% endcode %}

* [**img4tool**](https://github.com/tihmstar/img4tool)
```bash
img4tool -e kernelcache.release.iphone14 -o kernelcache.release.iphone14.e
```
आप निम्नलिखित कमांड के साथ kernelcache में सिंबल्स की जांच कर सकते हैं: **`nm -a kernelcache.release.iphone14.e | wc -l`**

इसके साथ हम अब **सभी एक्सटेंशन्स निकाल सकते हैं** या **वह एक्सटेंशन जिसमें आपकी रुचि है:**
```bash
# List all extensions
kextex -l kernelcache.release.iphone14.e
## Extract com.apple.security.sandbox
kextex -e com.apple.security.sandbox kernelcache.release.iphone14.e

# Extract all
kextex_all kernelcache.release.iphone14.e

# Check the extension for symbols
nm -a binaries/com.apple.security.sandbox | wc -l
```
## macOS कर्नेल एक्सटेंशन्स

macOS **कर्नेल एक्सटेंशन्स** (.kext) को लोड करने के लिए अत्यधिक प्रतिबंधित है क्योंकि उस कोड को उच्च विशेषाधिकारों के साथ चलाया जाएगा। वास्तव में, डिफ़ॉल्ट रूप से यह लगभग असंभव है (जब तक कि एक बाईपास नहीं मिल जाता है)।

{% content-ref url="macos-kernel-extensions.md" %}
[macos-kernel-extensions.md](macos-kernel-extensions.md)
{% endcontent-ref %}

### macOS सिस्टम एक्सटेंशन्स

कर्नेल एक्सटेंशन्स का उपयोग करने के बजाय macOS ने सिस्टम एक्सटेंशन्स बनाए, जो कर्नेल के साथ इंटरैक्ट करने के लिए यूजर लेवल APIs प्रदान करते हैं। इस तरह, डेवलपर्स कर्नेल एक्सटेंशन्स का उपयोग करने से बच सकते हैं।

{% content-ref url="macos-system-extensions.md" %}
[macos-system-extensions.md](macos-system-extensions.md)
{% endcontent-ref %}

## संदर्भ

* [**The Mac Hacker's Handbook**](https://www.amazon.com/-/es/Charlie-Miller-ebook-dp-B004U7MUMU/dp/B004U7MUMU/ref=mt\_other?\_encoding=UTF8\&me=\&qid=)
* [**https://taomm.org/vol1/analysis.html**](https://taomm.org/vol1/analysis.html)

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert) के साथ AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स शेयर करें।

</details>
