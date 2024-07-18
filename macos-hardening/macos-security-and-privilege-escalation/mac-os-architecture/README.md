# macOS कर्नेल और सिस्टम एक्सटेंशन्स

{% hint style="success" %}
AWS हैकिंग सीखें और अभ्यास करें:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण AWS रेड टीम एक्सपर्ट (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP हैकिंग सीखें और अभ्यास करें: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks प्रशिक्षण GCP रेड टीम एक्सपर्ट (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>हैकट्रिक्स का समर्थन करें</summary>

* [**सदस्यता योजनाएँ**](https://github.com/sponsors/carlospolop) की जाँच करें!
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) और **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर हमें** **फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में।

</details>
{% endhint %}

## XNU कर्नेल

**macOS का मूल भाग XNU** है, जिसका मतलब है "X is Not Unix". यह कर्नेल मूल रूप से **Mach माइक्रोकर्नल** (जिस पर बाद में चर्चा की जाएगी), **और** बर्कले सॉफ्टवेयर डिस्ट्रीब्यूशन (**बीएसडी**) से तत्वों से मिलकर बना है। XNU भी **I/O किट** नामक एक सिस्टम के माध्यम से **कर्नेल ड्राइवर्स के लिए एक मंच प्रदान करता है**। XNU कर्नेल डार्विन ओपन सोर्स प्रोजेक्ट का हिस्सा है, जिसका मतलब है **इसका सोर्स कोड मुफ्त रूप से उपलब्ध है**।

एक सुरक्षा शोधक या यूनिक्स डेवलपर के दृष्टिकोण से, **macOS** एक **फ्रीबीएसडी** सिस्टम के साथ काफी **समान** महसूस हो सकता है जिसमें एक शानदार जीयूआई और कई कस्टम एप्लिकेशन्स हैं। बीएसडी के लिए विकसित की गई अधिकांश एप्लिकेशन्स को बिना संशोधन के macOS पर कंपाइल और चलाया जा सकता है, क्योंकि यूनिक्स उपयोगकर्ताओं के लिए परिचित कमांड-लाइन टूल्स सभी macOS में मौजूद हैं। हालांकि, क्योंकि XNU कर्नेल में Mach शामिल है, एक पारंपरिक यूनिक्स-जैसे सिस्टम और macOS के बीच कुछ महत्वपूर्ण अंतर हैं, और ये अंतर संभावित मुद्दों का कारण बन सकते हैं या अद्वितीय लाभ प्रदान कर सकते हैं।

XNU का ओपन सोर्स संस्करण: [https://opensource.apple.com/source/xnu/](https://opensource.apple.com/source/xnu/)

### Mach

Mach एक **माइक्रोकर्नल** है जो **यूनिक्स-संगत** बनाया गया है। इसका एक मुख्य डिज़ाइन सिद्धांत था कि **कर्नेल स्पेस में चल रहे कोड** की मात्रा को **कम करना** और बजाय इसके कि कई पारंपरिक कर्नेल कार्य, जैसे फ़ाइल सिस्टम, नेटवर्किंग, और आई/ओ, **उपयोगकर्ता स्तर के टास्क के रूप में चलने देना**।

XNU में, Mach **कर्नेल के लिए कई महत्वपूर्ण लो-लेवल ऑपरेशन्स की जिम्मेदारी लेता है**, जैसे प्रोसेसर शेड्यूलिंग, मल्टीटास्किंग, और वर्चुअल मेमोरी मैनेजमेंट।

### BSD

XNU **कर्नेल** में भी **बहुत सारा कोड** शामिल है जो **फ्रीबीएसडी** प्रोजेक्ट से लिया गया है। यह कोड **कर्नेल का हिस्सा होता है जिसमें Mach के साथ चलता है**, एक ही पता स्थान में। हालांकि, XNU के भीतर फ्रीबीएसडी कोड मूल फ्रीबीएसडी कोड से काफी भिन्न हो सकता है क्योंकि Mach के साथ संगतता सुनिश्चित करने के लिए संशोधन की आवश्यकता थी। फ्रीबीएसडी कई कर्नेल ऑपरेशन्स में योगदान करता है जिसमें शामिल हैं:

* प्रोसेस प्रबंधन
* सिग्नल हैंडलिंग
* मूल सुरक्षा तंत्र, उपयोगकर्ता और समूह प्रबंधन सहित
* सिस्टम कॉल इंफ्रास्ट्रक्चर
* TCP/IP स्टैक और सॉकेट्स
* फ़ायरवॉल और पैकेट फ़िल्टरिंग

BSD और Mach के बीच संवाद को समझना कठिन हो सकता है, उनके विभिन्न विचारात्मक ढांचे के कारण। उदाहरण के लिए, BSD प्रक्रियाएँ अपनी मौलिक क्रियान्वयन इकाई के रूप में उपयोग करता है, जबकि Mach धागों पर आधारित काम करता है। यह अंतर XNU में सुलझाया जाता है **प्रत्येक BSD प्रक्रिया को एक Mach टास्क से जोड़ने** के द्वारा जिसमें एक Mach धाग होता है। जब BSD का fork() सिस्टम कॉल किया जाता है, कर्नेल के भीतर BSD कोड Mach फ़ंक्शन का उपयोग करता है एक टास्क और एक धाग संरचना बनाने के लिए।

इसके अतिरिक्त, **Mach और BSD दोनों अलग-अलग सुरक्षा मॉडल बनाए रखते हैं**: **Mach** का सुरक्षा मॉडल **पोर्ट राइट्स** पर आधारित है, जबकि BSD का सुरक्षा मॉडल **प्रक्रिया स्वामित्व** पर आधारित है। इन दो मॉडलों के बीच अंतरों ने कभी-कभी स्थानीय प्रिविलेज-एस्केलेशन संरचनाओं में परिणाम दिया है। सामान्य सिस्टम कॉल्स के अलावा, उपयोगकर्ता-स्पेस प्रोग्राम्स को कर्नेल के साथ बातचीत करने की अनुमति देने वाले **Mach ट्रैप्स** भी हैं। ये विभिन्न तत्व मिलकर macOS कर्नेल की बहुपक्षीय, मिश्रित वास्तुकला बनाते हैं।

### I/O किट - ड्राइवर्स

I/O किट एक ओपन-सोर्स, ऑब्जेक्ट-ओरिएंटेड **डिवाइस-ड्राइवर फ्रेमवर्क** है XNU कर्नेल में, **डायनामिकली लोडेड डिवाइस ड्राइवर्स** को संभालता है। यह कर्नेल में मॉड्यूलर कोड को तुरंत जोड़ने की अनुमति देता है, विभिन्न हार्डवेयर का समर्थन करता है।

{% content-ref url="macos-iokit.md" %}
[macos-iokit.md](macos-iokit.md)
{% endcontent-ref %}

### IPC - इंटर प्रोसेस कम्यूनिकेशन

{% content-ref url="../macos-proces-abuse/macos-ipc-inter-process-communication/" %}
[macos-ipc-inter-process-communication](../macos-proces-abuse/macos-ipc-inter-process-communication/)
{% endcontent-ref %}

### कर्नेलकैश

**कर्नेलकैश** एक **पूर्व-कंपाइल्ड और पूर्व-लिंक्ड संस्करण** है XNU कर्नेल का, महत्वपूर्ण डिवाइस **ड्राइवर्स** और **कर्नेल एक्सटेंशन्स** के साथ। यह एक **संपीड़ित** प्रारूप में संग्रहीत होता है और बूट-अप प्रक्रिया के दौरान मेमोरी में डीकंप्रेस होता है। कर्नेलकैश एक तैयार-से-चलाने योग्य संस्करण के साथ और महत्वपूर्ण ड्राइवर्स उपलब्ध होने से **बू
```bash
# pyimg4 (https://github.com/m1stadev/PyIMG4)
pyimg4 im4p extract -i kernelcache.release.iphone14 -o kernelcache.release.iphone14.e

# img4tool (https://github.com/tihmstar/img4tool
img4tool -e kernelcache.release.iphone14 -o kernelcache.release.iphone14.e
```
#### कर्नेलकैश प्रतीक

कभी-कभी Apple **कर्नेलकैश** के साथ **प्रतीक** जारी करता है। आप कुछ फर्मवेयर्स को प्रतीक के साथ डाउनलोड कर सकते हैं [https://theapplewiki.com](https://theapplewiki.com/) पर दिए गए लिंकों का पालन करके।

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

इसके साथ हम अब **सभी एक्सटेंशन्स** या **जिसमें आप रुचि रखते हैं, उसे निकाल सकते हैं:**
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

macOS **कर्नेल एक्सटेंशन्स (.kext) लोड करने के लिए अत्यधिक प्रतिबंधक है** क्योंकि उच्च विशेषाधिकारों के कारण कोड चलेगा। वास्तव में, डिफ़ॉल्ट रूप से यह लगभग असंभव है (जब तक कोई बायपास नहीं मिल जाता है)।

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
