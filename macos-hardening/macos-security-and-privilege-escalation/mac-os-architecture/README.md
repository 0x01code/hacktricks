# macOS कर्नेल और सिस्टम एक्सटेंशन्स

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert) के साथ</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) का खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मेरा** **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** का पालन करें**.
* **अपने हैकिंग ट्रिक्स साझा करें** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos को PR जमा करके।

</details>

## XNU कर्नेल

**macOS का मूल है XNU**, जिसका मतलब है "X is Not Unix". यह कर्नेल मूल रूप से **Mach माइक्रोकर्नल** (जिस पर बाद में चर्चा की जाएगी), **और** बर्कले सॉफ्टवेयर डिस्ट्रीब्यूशन (**BSD**) से तत्वों से मिलकर बना है। XNU भी **I/O Kit नामक सिस्टम के माध्यम से कर्नेल ड्राइवर्स के लिए एक मंच प्रदान करता है**। XNU कर्नेल Darwin ओपन सोर्स प्रोजेक्ट का हिस्सा है, जिसका मतलब है कि **इसका सोर्स कोड मुफ्त रूप से उपलब्ध है**।

एक सुरक्षा शोधक या यूनिक्स डेवलपर के दृष्टिकोण से, **macOS** एक **FreeBSD** सिस्टम के साथ काफी **समान** महसूस हो सकता है जिसमें एक शानदार GUI और कई कस्टम एप्लिकेशन्स हैं। BSD के लिए विकसित अधिकांश एप्लिकेशन्स को बिना संशोधन के macOS पर कंपाइल और चलाया जा सकता है, क्योंकि Unix उपयोगकर्ताओं के लिए परिचित कमांड-लाइन टूल्स सभी macOS में मौजूद हैं। हालांकि, XNU कर्नेल में Mach शामिल होने के कारण, एक पारंपरिक यूनिक्स-जैसे सिस्टम और macOS के बीच कुछ महत्वपूर्ण अंतर हैं, और ये अंतर संभावित मुद्दों का कारण बन सकते हैं या अद्वितीय लाभ प्रदान कर सकते हैं।

XNU का ओपन सोर्स संस्करण: [https://opensource.apple.com/source/xnu/](https://opensource.apple.com/source/xnu/)

### Mach

Mach एक **माइक्रोकर्नल** है जो **UNIX-संगत** बनाया गया है। इसका एक मुख्य डिज़ाइन सिद्धांत था कि कर्नेल स्पेस में चल रहे **कोड** की मात्रा को **कम करना** और बजाय इसके कि कई पारंपरिक कर्नेल कार्य, जैसे फ़ाइल सिस्टम, नेटवर्किंग, और I/O, **उपयोगकर्ता स्तर के टास्क के रूप में चलने देना**।

XNU में, Mach **कर्नेल के बहुत से महत्वपूर्ण निम्न स्तरीय कार्यों के लिए जिम्मेदार है**, जैसे प्रोसेसर शेड्यूलिंग, मल्टीटास्किंग, और वर्चुअल मेमोरी प्रबंधन।

### BSD

XNU **कर्नेल** में भी **FreeBSD** परियोजना से लिए गए कोड की एक महत्वपूर्ण मात्रा शामिल है। यह कोड **कर्नेल का हिस्सा होता है जिसमें Mach के साथ चलता है**। हालांकि, XNU के भीतर FreeBSD कोड मूल FreeBSD कोड से काफी भिन्न हो सकता है क्योंकि Mach के साथ इसकी संगतता सुनिश्चित करने के लिए संशोधन की आवश्यकता थी। FreeBSD बहुत सारे कर्नेल कार्यों में योगदान करता है जिसमें शामिल हैं:

* प्रोसेस प्रबंधन
* सिग्नल हैंडलिंग
* मूल सुरक्षा तंत्र, उपयोगकर्ता और समूह प्रबंधन सहित
* सिस्टम कॉल इंफ्रास्ट्रक्चर
* TCP/IP स्टैक और सॉकेट्स
* फ़ायरवॉल और पैकेट फ़िल्टरिंग

BSD और Mach के बीच के इंटरैक्शन को समझना कठिन हो सकता है, उनके विभिन्न विचारात्मक ढांचे के कारण। उदाहरण के लिए, BSD प्रक्रियाएँ अपनी मौलिक क्रियान्वयन इकाई के रूप में उपयोग करता है, जबकि Mach धागों पर आधारित चालू होता है। यह अंतर XNU में सुलझाया जाता है **हर एक BSD प्रक्रिया को एक Mach टास्क से जोड़ने** के द्वारा जिसमें एक Mach धाग होता है। जब BSD का fork() सिस्टम कॉल किया जाता है, कर्नेल के भीतर BSD कोड Mach कार्यों का उपयोग करता है एक टास्क और एक धाग संरचना बनाने के लिए।

इसके अतिरिक्त, **Mach और BSD दोनों अलग सुरक्षा मॉडल बनाए रखते हैं**: **Mach** का सुरक्षा मॉडल **पोर्ट राइट्स** पर आधारित है, जबकि BSD का सुरक्षा मॉडल **प्रक्रिया स्वामित्व** पर आधारित है। इन दो मॉडलों के बीच के अंतरों ने कभी-कभी स्थानीय प्रिविल
```bash
# pyimg4 (https://github.com/m1stadev/PyIMG4)
pyimg4 im4p extract -i kernelcache.release.iphone14 -o kernelcache.release.iphone14.e

# img4tool (https://github.com/tihmstar/img4tool
img4tool -e kernelcache.release.iphone14 -o kernelcache.release.iphone14.e
```
#### कर्नेलकैश सिम्बल्स

कभी-कभी Apple **कर्नेलकैश** के साथ **सिम्बल्स** जारी करता है। आप कुछ फर्मवेयर्स को सिम्बल्स के साथ [https://theapplewiki.com](https://theapplewiki.com/) पर दिए गए लिंक्स का पालन करके डाउनलोड कर सकते हैं।

### IPSW

ये Apple **फर्मवेयर्स** हैं जिन्हें आप [**https://ipsw.me/**](https://ipsw.me/) से डाउनलोड कर सकते हैं। अन्य फ़ाइलों के बीच यह **कर्नेलकैश** शामिल होगा।\
फ़ाइलों को **निकालने** के लिए आप बस इसे **अनज़िप** कर सकते हैं।

फर्मवेयर को निकालने के बाद आपको एक फ़ाइल मिलेगी जैसे: **`kernelcache.release.iphone14`**। यह **IMG4** प्रारूप में है, आप दिलचस्प जानकारी को निकाल सकते हैं:

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
आप सिम्बल के लिए निकाले गए kernelcache की जाँच कर सकते हैं: **`nm -a kernelcache.release.iphone14.e | wc -l`**

इसके साथ हम अब **सभी एक्सटेंशन** या **जिसमें आप रुचि रखते हैं, उसे निकाल सकते हैं:**
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

macOS **कर्नेल एक्सटेंशन्स (.kext) लोड करना** बहुत प्रतिबंधकारी है क्योंकि उच्च विशेषाधिकारों के कारण कोड चलेगा। वास्तव में, डिफ़ॉल्ट रूप से यह लगभग असंभव है (जब तक कोई बायपास नहीं मिल जाता है)।

{% content-ref url="macos-kernel-extensions.md" %}
[macos-kernel-extensions.md](macos-kernel-extensions.md)
{% endcontent-ref %}

### macOS सिस्टम एक्सटेंशन्स

कर्नेल एक्सटेंशन्स का उपयोग करने की बजाय, macOS ने सिस्टम एक्सटेंशन्स बनाए हैं, जो कर्नेल के साथ बातचीत करने के लिए उपयोगकर्ता स्तरीय API प्रदान करता है। इस तरह, डेवलपर्स कर्नेल एक्सटेंशन्स का उपयोग टाल सकते हैं।

{% content-ref url="macos-system-extensions.md" %}
[macos-system-extensions.md](macos-system-extensions.md)
{% endcontent-ref %}

## संदर्भ

* [**The Mac Hacker's Handbook**](https://www.amazon.com/-/es/Charlie-Miller-ebook-dp-B004U7MUMU/dp/B004U7MUMU/ref=mt\_other?\_encoding=UTF8\&me=\&qid=)
* [**https://taomm.org/vol1/analysis.html**](https://taomm.org/vol1/analysis.html)

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी कंपनी का विज्ञापन HackTricks में देखना चाहते हैं या **HackTricks को PDF में डाउनलोड** करना चाहते हैं तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) **पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** द्वारा **पीआर** सबमिट करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो का समर्थन करें।

</details>
